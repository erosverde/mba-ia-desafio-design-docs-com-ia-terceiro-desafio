# FDD — Webhooks de mudança de status de pedidos

**Status** · Proposta para revisão técnica
**Referências** · [RFC](RFC.md), [ADRs](adrs/) e [transcrição](../TRANSCRICAO.md)

## Contexto e objetivos técnicos

Os clientes consultam a API de pedidos para descobrir mudanças de status. O desenho acrescenta notificações de saída com registro atômico, entrega assíncrona, deduplicação pelo consumidor e recuperação operacional de falhas. A expectativa é entregar em menos de 10 segundos em condições normais. A latência observada inclui espera na fila e duração do HTTP. [Transcrição, 09:00–09:10 e 09:24–09:26](../TRANSCRICAO.md)

## Escopo

Inclui configuração de endpoints e filtros, rotação de segredo, captura de transições, worker, histórico das últimas 100 entregas, retentativas e replay administrativo. Email de fallback, dashboard, rate limiting de saída e arquivamento ficam fora desta fase. [Transcrição, 09:08, 09:21–09:22, 09:31–09:40 e 09:48](../TRANSCRICAO.md)

## Modelo de dados proposto

| Tabela | Dados e restrições |
| --- | --- |
| `webhook_endpoints` | UUID, cliente, URL HTTPS, estado ativo, lista de valores `OrderStatus`, segredo atual e anterior protegido, expiração do anterior e timestamps. Índice por cliente e estado. |
| `webhook_outbox` | UUID da linha, UUID do evento, ID do endpoint, ID do pedido, snapshot JSON serializado em UTF-8, status, contador de tentativas, próxima tentativa, expiração da reserva e timestamps. Índice para busca por status e próxima tentativa. Priorizar `created_at` com UUID da linha como desempate. Uma entrega por par evento e endpoint. |
| `webhook_deliveries` | UUID, ID do endpoint, ID do evento, número da tentativa no ciclo, resultado, código e corpo da resposta quando disponíveis, duração em milissegundos, referência ao snapshot e timestamp. Índice por endpoint e timestamp para listar as últimas 100 tentativas. |
| `webhook_dead_letter` | UUID, referência à entrega na outbox, UUID original do evento, endpoint, snapshot, motivo, timestamp da falha e dados de auditoria do replay. |

A granularidade proposta é uma linha de outbox por endpoint elegível, compartilhando o ID do evento entre as entregas da mesma transição. Usar IDs distintos para a linha e para o evento permite acompanhar falhas por destino sem perder a identidade lógica. A remoção proposta desativa o cadastro e preserva as referências históricas. [Outbox e snapshot](adrs/ADR-001-outbox-transacional-mysql.md) · [DLQ](adrs/ADR-004-dlq-e-replay-administrativo.md) · [Identidade do evento](adrs/ADR-006-entrega-at-least-once.md)

## Fluxos

### FDD-FLUXO-01 — Captura transacional

1. Validar a transição em `OrderService.changeStatus`. Ajustar estoque quando aplicável, atualizar o status e gravar o histórico.
2. Dentro do callback de `$transaction`, chamar `publishWebhookEvent(tx, order, fromStatus, toStatus)`. Buscar pelo mesmo `tx` os endpoints ativos do cliente inscritos no novo status.
3. Se houver destinos elegíveis, gerar o UUID do evento, serializar o snapshot com o estado da transição e inserir uma pendência por destino. Validar o limite do corpo em bytes UTF-8. A proposta interpreta 64 KB como 65.536 bytes e propaga o erro antes da inserção.
4. Confirmar a transação após as inserções. Qualquer falha nesta etapa reverte status, histórico, estoque e outbox. Uma transição sem destinos elegíveis segue o fluxo normal.

[ADR-001](adrs/ADR-001-outbox-transacional-mysql.md) · [Transcrição, 09:23–09:24, 09:33–09:34 e 09:40–09:41](../TRANSCRICAO.md)

### FDD-FLUXO-02 — Processamento e retentativas

1. Executar `npm run worker` como processo separado. A cada dois segundos, buscar um lote pequeno de linhas pendentes com próxima tentativa vencida. Processar sequencialmente, priorizando as mais antigas e evitando sobreposição de ciclos.
2. Para cada linha, reservar por atualização condicional de estado e prazo. Encerrar a transação de reserva antes do HTTP. Em reinício, recuperar reservas expiradas. O prazo deve cobrir o timeout HTTP e a margem de persistência. Vincular a finalização à reserva atual para evitar concluir uma tentativa já recuperada.
3. Ler URL e segredos válidos na configuração do endpoint. A proposta encerra pendências de endpoints removidos sem enviar HTTP e registra o motivo. Enviar os bytes armazenados com timeout total de 10 segundos. Para a política inicial proposta, aceitar 2xx como sucesso e tratar timeout, falha de rede e respostas fora de 2xx como falhas. A revisão deve confirmar a classificação de erros permanentes.
4. Em uma transação curta, registrar o resultado no histórico e finalizar a pendência como entregue, reagendar para a próxima tentativa ou gravar a DLQ e marcar falha terminal. Uma falha do processo depois que o destinatário recebeu o corpo e antes dessa transação pode gerar novo envio com o mesmo ID.
5. Manter a sequência de backoff parametrizada. A reunião cita cinco tentativas e intervalos de 1 min, 5 min, 30 min, 2 h e 12 h. Confirmar se são cinco envios totais ou cinco retentativas antes de configurar o último intervalo e fechar os testes dessa política.

[ADR-002](adrs/ADR-002-worker-polling-outbox.md) · [ADR-003](adrs/ADR-003-retentativas-com-backoff.md) · [ADR-006](adrs/ADR-006-entrega-at-least-once.md) · [Transcrição, 09:42](../TRANSCRICAO.md)

### FDD-FLUXO-03 — Replay

Autenticar o operador e exigir `ADMIN`. Em uma transação, validar a existência da DLQ e a elegibilidade do endpoint, reativar a linha terminal da outbox como pendente, reiniciar a contagem do ciclo e registrar identificação do operador e timestamp. A proposta preserva o snapshot e o ID do evento, mantém o histórico anterior e usa atualização condicional para rejeitar replay concorrente da mesma falha. Depois do commit, o worker retoma o fluxo normal. [ADR-004](adrs/ADR-004-dlq-e-replay-administrativo.md)

## Contratos públicos

### FDD-CONTRATO-01 — API de gerenciamento

Os caminhos da tabela são relativos ao prefixo /api/v1 montado pela aplicação. Todos exigem `Authorization: Bearer <token>`. Requisições com corpo usam `Content-Type: application/json`. As rotas de CRUD, rotação e os formatos abaixo são propostos. A transcrição nomeia explicitamente as rotas de histórico e replay. `customer_id` é informado no caminho de criação e listagem. O JWT atual identifica o usuário e o papel, sem vínculo com o cliente. Responder 401 para token ausente ou inválido e 403 para papel insuficiente no replay. A autorização por cliente permanece em revisão antes de expor essas rotas a usuários externos. [Transcrição, 09:21 e 09:31–09:37](../TRANSCRICAO.md) · [Middleware de autenticação](../src/middlewares/auth.middleware.ts)

| Método e caminho | Corpo de exemplo | Resultado |
| --- | --- | --- |
| `POST /customers/:customerId/webhooks` | `{"url":"https://cliente.example/events","statuses":["SHIPPED","DELIVERED"]}` | `201` com cadastro e segredo gerado. |
| `GET /customers/:customerId/webhooks` | — | `200` com a lista de cadastros ativos. |
| `PATCH /webhooks/:id` | `{"statuses":["DELIVERED"]}` | `200` com o cadastro atualizado. URL e filtros são campos editáveis propostos. |
| `DELETE /webhooks/:id` | — | `204` após desativar o cadastro. |
| `POST /webhooks/:id/rotate-secret` | — | `200` com o novo segredo e `previous_secret_expires_at`. |
| `GET /webhooks/:id/deliveries` | — | `200` com até 100 tentativas, da mais recente para a mais antiga. |
| `POST /admin/webhooks/dead-letter/:id/replay` | — | `202` com `{"status":"pending","event_id":"<uuid>"}` após recolocar na outbox. |

Exemplo de resposta de criação. IDs entre sinais de menor e maior são placeholders.

```json
{
  "id": "<webhook-uuid>",
  "customer_id": "<customer-uuid>",
  "url": "https://cliente.example/events",
  "statuses": ["SHIPPED", "DELIVERED"],
  "active": true,
  "secret": "<secret-gerado>"
}
```

As demais leituras de cadastro omitem a credencial. Exemplo de item de histórico, com o snapshot representado pelo campo `payload`.

```json
{
  "event_id": "<event-uuid>",
  "attempt": 1,
  "status": "delivered",
  "payload": {"event_type": "order.status_changed", "to_status": "SHIPPED"},
  "response": {"status_code": 200, "body": "ok"},
  "duration_ms": 180,
  "created_at": "2026-11-16T14:30:03Z"
}
```

Exemplos complementares de resposta para os contratos propostos. A rotação e o replay recebem apenas o ID no caminho e usam corpo vazio.

| Operação | Requisição | Resposta JSON de exemplo |
| --- | --- | --- |
| Alterar filtro | `PATCH /webhooks/<webhook-uuid>` com `{"statuses":["DELIVERED"]}` | `200 {"id":"<webhook-uuid>","customer_id":"<customer-uuid>","url":"https://cliente.example/events","statuses":["DELIVERED"],"active":true}` |
| Rotacionar segredo | `POST /webhooks/<webhook-uuid>/rotate-secret` sem corpo | `200 {"secret":"<novo-segredo>","previous_secret_expires_at":"2026-11-17T14:30:00Z"}` |
| Solicitar replay | `POST /admin/webhooks/dead-letter/<dlq-uuid>/replay` sem corpo | `202 {"status":"pending","event_id":"<event-uuid>"}` |
| Listar cadastros | `GET /customers/<customer-uuid>/webhooks` sem corpo | `200 [{"id":"<webhook-uuid>","customer_id":"<customer-uuid>","url":"https://cliente.example/events","statuses":["DELIVERED"],"active":true}]` |

### FDD-CONTRATO-02 — Notificação de saída

O worker faz `POST` à URL HTTPS configurada. O corpo usa o snapshot do evento e o mesmo `event_id` em todas as tentativas, inclusive replay.

```json
{
  "event_id": "0db9e705-31e5-4fb0-ad34-cad8879df682",
  "event_type": "order.status_changed",
  "timestamp": "2026-11-16T14:30:00Z",
  "order_id": "470a7438-4dc3-4656-9dea-c73d7f0f9213",
  "order_number": "ORD-2026-00042",
  "from_status": "PROCESSING",
  "to_status": "SHIPPED",
  "customer_id": "2b41cb89-56bc-46ae-84ae-f2ffc20129bc",
  "total_cents": 15990
}
```

| Header | Valor |
| --- | --- |
| `Content-Type` | `application/json` |
| `X-Event-Id` | Mesmo UUID do corpo para deduplicação no destino |
| `X-Webhook-Id` | UUID do cadastro de destino |
| `X-Timestamp` | Instante do envio em ISO 8601, formato proposto |
| `X-Signature` | HMAC-SHA256 calculado sobre os bytes exatos do corpo |

Para concretizar a janela de rotação, propõe-se codificar o HMAC em hexadecimal e usar `v1=<assinatura-atual>`, acrescido de `,v1=<assinatura-anterior>` enquanto ambas as credenciais estiverem válidas. O consumidor valida o corpo bruto com seu segredo e comparação em tempo constante. O timestamp de envio é metadado. O HMAC acordado cobre o corpo. Geração, armazenamento dos segredos, formato final da assinatura e validação de destinos passam pela revisão de segurança. [ADR-005](adrs/ADR-005-assinatura-hmac-por-endpoint.md) · [ADR-008](adrs/ADR-008-contrato-do-evento-de-status.md)

### FDD-ERRO-01 — Matriz de erros

Erros usam o envelope `{"error":{"code":"WEBHOOK_INVALID_URL","message":"A URL deve usar HTTPS"}}`. Os códigos específicos abaixo são propostas de implementação. Os middlewares existentes mantêm seus códigos de autenticação, autorização e validação genérica.

| Código | HTTP ou efeito | Condição |
| --- | --- | --- |
| `WEBHOOK_INVALID_URL` | 400 | URL inválida ou sem HTTPS |
| `WEBHOOK_INVALID_STATUSES` | 400 | Filtro contém estado desconhecido ou lista vazia |
| `WEBHOOK_CUSTOMER_NOT_FOUND` | 404 | Cliente inexistente |
| `WEBHOOK_NOT_FOUND` | 404 | Cadastro desconhecido ou removido |
| `WEBHOOK_DEAD_LETTER_NOT_FOUND` | 404 | Falha terminal desconhecida |
| `WEBHOOK_REPLAY_CONFLICT` | 409 | Replay já solicitado ou destino desativado |
| `WEBHOOK_PAYLOAD_TOO_LARGE` | Falha de captura e rollback | Snapshot excede o limite proposto |
| `WEBHOOK_DELIVERY_TIMEOUT` | Falha de tentativa | HTTP ultrapassa 10 segundos |
| `WEBHOOK_DELIVERY_FAILED` | Falha de tentativa | Erro de rede ou resposta fora de 2xx |

[ADR-007](adrs/ADR-007-reuso-dos-padroes-do-projeto.md) · [Middleware de erros](../src/middlewares/error.middleware.ts)

## Integração com o sistema existente

| Arquivo | Integração |
| --- | --- |
| [`src/modules/orders/order.service.ts`](../src/modules/orders/order.service.ts#L126) | Inserir a chamada de captura no callback transacional após gravar o histórico. |
| [`prisma/schema.prisma`](../prisma/schema.prisma) | Acrescentar modelos e índices via migração MySQL. Reaproveitar `OrderStatus` e as referências a cliente e pedido. |
| [`src/app.ts`](../src/app.ts) e [`src/routes/index.ts`](../src/routes/index.ts) | Instanciar o módulo `src/modules/webhooks` e montar suas rotas no roteador existente. |
| [`src/middlewares/auth.middleware.ts`](../src/middlewares/auth.middleware.ts) | Aplicar `authenticate` nas rotas e `requireRole('ADMIN')` no replay. |
| [`src/middlewares/validate.middleware.ts`](../src/middlewares/validate.middleware.ts) | Reutilizar validação Zod de corpo e parâmetros. Mapear erros específicos de domínio para `AppError`. |
| [`src/shared/errors/app-error.ts`](../src/shared/errors/app-error.ts) e [`src/middlewares/error.middleware.ts`](../src/middlewares/error.middleware.ts) | Reutilizar a classe base e a serialização central de erros HTTP. |
| [`src/server.ts`](../src/server.ts) e [`src/config/database.ts`](../src/config/database.ts) | Usar o servidor como referência para o ponto de entrada separado e a fábrica Prisma para a conexão do worker. |
| [`src/shared/logger/index.ts`](../src/shared/logger/index.ts) | Emitir logs estruturados de entrega, falha e replay. |

## Dependências e compatibilidade

A solução usa Node.js, Express, Prisma, MySQL, Zod e Pino existentes, mais a API criptográfica do Node para a assinatura. Implantar as tabelas antes de ativar a captura e iniciar um único worker. As rotas existentes mantêm seus contratos. Uma falha de registro do webhook elegível passa a poder reverter a transição. [ADR-001](adrs/ADR-001-outbox-transacional-mysql.md) · [ADR-002](adrs/ADR-002-worker-polling-outbox.md) · [ADR-007](adrs/ADR-007-reuso-dos-padroes-do-projeto.md)

## Observabilidade

Como instrumentação proposta, medir tamanho e idade da fila, latência entre captura e primeiro envio, duração do HTTP, tentativas por resultado e entradas na DLQ. Correlacionar logs da transição e do worker via `event_id`, `webhook_id`, `order_id` e número da tentativa. O replay inclui o ID do operador autenticado e o ID da DLQ. Omitir segredos, credenciais HTTP, assinaturas e corpos dos logs. Para tracing, correlacionar os registros de captura, reserva, envio, finalização e replay pelos IDs persistidos. Emitir o registro de captura após o commit para distinguir eventos confirmados. O request ID existente identifica a requisição na API. O event ID conecta essa requisição ao processamento no worker.

## Riscos e mitigação

- **Backlog e pressão no MySQL.** Usar lotes pequenos, índices e monitorar o atraso antes de mudar o intervalo ou a concorrência.
- **Duplicata ou inversão de ordem.** Preservar `X-Event-Id` e documentar para o consumidor que retentativas podem alterar a ordem de chegada. Múltiplos workers exigem revisão da estratégia.
- **Acesso indevido a endpoints e histórico.** O JWT atual representa usuário e papel. A regra de vínculo com o cliente e o acesso aos dados de resposta exigem definição antes da exposição externa.
- **Segredos ou destinos inseguros.** Exigir HTTPS, evitar credenciais em logs e incluir na revisão de segurança o armazenamento de segredos, a rotação e o controle dos destinos e redirecionamentos.
- **Falha persistente.** Usar a DLQ como fallback operacional com replay auditado. O destinatário pode continuar indisponível após o replay.

[RFC](RFC.md) · [Transcrição, 09:12–09:26, 09:32–09:39 e 09:46–09:49](../TRANSCRICAO.md)

## Critérios de aceite técnicos

| ID | Verificação |
| --- | --- |
| FDD-AC-01 | Em teste de integração, transição elegível grava status, histórico, estoque aplicável e todas as pendências. Simular erro de inserção e verificar rollback de todos os efeitos. |
| FDD-AC-02 | Transição inválida ou sem inscrição compatível gera zero pendências. Alterar o pedido depois do commit e verificar que o corpo armazenado continua igual. |
| FDD-AC-03 | Em teste com relógio controlado e servidor HTTP de teste, verificar polling, timeout, backoff acordado e registro da falha terminal na DLQ. Simular reinício com reserva vencida e verificar recuperação. |
| FDD-AC-04 | Simular resposta perdida após recebimento. A nova tentativa mantém o ID e o corpo. Medir a entrega abaixo de dez segundos no cenário de fila vazia com destino saudável. |
| FDD-AC-05 | Verificar HMAC sobre o corpo bruto, rejeição de URL sem HTTPS, limite do corpo, aceitação durante rotação e expiração do segredo anterior ao fim de 24 horas. |
| FDD-AC-06 | Exercitar histórico com 101 tentativas e verificar as 100 mais recentes. Testar 401 sem token, 403 para operador comum e 202 com ADMIN, preservação do evento e auditoria do replay. |

Usar a infraestrutura de integração existente em [`tests/setup.ts`](../tests/setup.ts) e [`tests/orders.test.ts`](../tests/orders.test.ts) como referência para os testes de transação. As confirmações de retry, autorização por cliente e contrato final de rotação são pendências de revisão do desenho.
