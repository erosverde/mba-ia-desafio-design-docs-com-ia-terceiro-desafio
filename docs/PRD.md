# PRD — Webhooks de notificação de pedidos

**Status** · Proposta para revisão de produto
**Documentos relacionados** · [RFC](RFC.md) · [FDD](FDD.md)

## Resumo e contexto

Oferecer webhooks de saída para que clientes B2B recebam notificações quando um pedido muda para um status que escolheram acompanhar. A primeira versão inclui gerenciamento pela API, histórico de entregas e recuperação administrativa de falhas. [Transcrição, 09:00–09:03 e 09:31–09:37](../TRANSCRICAO.md)

## Problema e motivação

Atlas Comercial, MaxDistribuição e Nova Cargo consultam periodicamente `GET /orders` para detectar mudanças. Esse fluxo encarece a integração e atrasa o processamento no cliente. A Atlas sinalizou risco de migração e pediu a entrega até o fim de novembro. Marcos ficou de confirmar o prazo com os clientes. [Transcrição, 09:00–09:02 e 09:45–09:47](../TRANSCRICAO.md)

## Público-alvo e cenários de uso

- **Integradores B2B.** Configuram um endpoint e assinam apenas os status relevantes. Um cliente pode acompanhar `SHIPPED` e `DELIVERED` e buscar os detalhes do pedido quando receber a notificação. [Transcrição, 09:31–09:34 e 09:43](../TRANSCRICAO.md)
- **Equipes de operação do cliente.** Consultam o histórico de um endpoint para investigar o último envio, a resposta recebida e o tempo de resposta. [Transcrição, 09:34](../TRANSCRICAO.md)
- **Administradores da plataforma.** Reprocessam manualmente falhas terminais e deixam registro de quem solicitou o replay. [Transcrição, 09:35–09:36](../TRANSCRICAO.md)

## Objetivos e métricas de sucesso

| ID | Objetivo | Como medir |
| --- | --- | --- |
| PRD-OBJ-01 | Entregar notificações em menos de 10 segundos quando o destino está saudável e a fila opera normalmente. | Em validação ponta a ponta, medir do commit da transição até o recebimento pelo endpoint de teste. A reunião definiu a expectativa de latência e deixou percentil e volume de referência em aberto. [09:02 e 09:09–09:10](../TRANSCRICAO.md) |
| PRD-OBJ-02 | Reduzir a necessidade de consultas periódicas para detectar mudanças de status. | Como medição proposta, comparar o volume de consultas periódicas dos clientes participantes antes e depois da integração. A reunião deixou a meta quantitativa em aberto. [09:00–09:02](../TRANSCRICAO.md) |

## Escopo

### Incluso

Configuração e rotação de segredo por endpoint, filtros de status, envio assinado de notificações, retentativas, histórico das últimas 100 tentativas e replay administrativo de falhas terminais. [Transcrição, 09:15–09:26, 09:31–09:37 e 09:48](../TRANSCRICAO.md)

### Fora de escopo

- Receber webhooks enviados pelos clientes. [09:02–09:03](../TRANSCRICAO.md)
- Enviar email de alerta ou fallback quando um endpoint falhar. [09:37–09:38](../TRANSCRICAO.md)
- Implementar dashboard visual. [09:39–09:40](../TRANSCRICAO.md)
- Aplicar rate limiting de saída nesta fase. A equipe vai observar o comportamento antes de decidir. [09:38–09:39](../TRANSCRICAO.md)
- Arquivar automaticamente as linhas entregues da outbox. [09:08](../TRANSCRICAO.md)

## Requisitos funcionais

| ID | Requisito | Fonte |
| --- | --- | --- |
| PRD-FR-01 | Permitir que um usuário autenticado cadastre um endpoint pela API, com o cliente informado explicitamente. Gerar um segredo exclusivo e retorná-lo na criação. | [09:21 e 09:31–09:32](../TRANSCRICAO.md) |
| PRD-FR-02 | Permitir uma lista de status por endpoint. Ao confirmar a transição de um pedido, registrar notificações apenas para inscrições do cliente que correspondam ao novo status. | [09:33–09:34 e 09:40–09:41](../TRANSCRICAO.md) |
| PRD-FR-03 | Enviar o evento `order.status_changed` com ID estável, identificação do pedido, status anterior e novo e um snapshot dos dados acordados para a transição. | [09:24–09:26, 09:43–09:45 e 09:51–09:52](../TRANSCRICAO.md) |
| PRD-FR-04 | Permitir rotação de segredo pela API com validade simultânea do segredo atual e do anterior durante 24 horas. | [09:21–09:22](../TRANSCRICAO.md) |
| PRD-FR-05 | Disponibilizar as últimas 100 tentativas por endpoint, com resultado, payload, resposta quando disponível e tempo de resposta. | [09:34](../TRANSCRICAO.md) |
| PRD-FR-06 | Retentar falhas de entrega com backoff limitado e registrar falhas terminais na DLQ com payload, motivo e timestamp. | [09:15–09:19 e 09:48](../TRANSCRICAO.md) |
| PRD-FR-07 | Permitir replay manual apenas para `ADMIN`, recolocar o evento como pendente e registrar quem solicitou a operação. | [09:18–09:19 e 09:35–09:36](../TRANSCRICAO.md) |
| PRD-FR-08 | Permitir listar os endpoints de um cliente, editar a configuração e remover um endpoint pela API autenticada. | [09:33 e 09:36–09:37](../TRANSCRICAO.md) |

## Requisitos não funcionais

| ID | Requisito | Fonte |
| --- | --- | --- |
| PRD-NFR-01 | Manter a latência abaixo de 10 segundos no cenário normal do PRD-OBJ-01. | [09:02 e 09:09–09:10](../TRANSCRICAO.md) |
| PRD-NFR-02 | Registrar a mudança de status e o evento elegível atomicamente. O envio ocorre depois do commit, em processo independente da API. | [09:06–09:11 e 09:40–09:41](../TRANSCRICAO.md) |
| PRD-NFR-03 | Assinar o corpo com HMAC-SHA256, usar segredo por endpoint e aceitar apenas URLs HTTPS. | [09:19–09:23](../TRANSCRICAO.md) |
| PRD-NFR-04 | Limitar o payload a 64 KB e tratar estouro como erro. | [09:23–09:24](../TRANSCRICAO.md) |
| PRD-NFR-05 | Adotar semântica at-least-once com `X-Event-Id` para deduplicação pelo destinatário. Falhas terminais ficam sujeitas ao fluxo de replay. | [09:18–09:19 e 09:24–09:26](../TRANSCRICAO.md) |
| PRD-NFR-06 | Encerrar uma tentativa de envio após 10 segundos sem resposta e tratar como falha. | [09:42](../TRANSCRICAO.md) |

## Decisões e trade-offs principais

- **Outbox e worker separado.** A indisponibilidade do endpoint afeta a entrega assíncrona. Uma falha ao registrar um evento elegível reverte a mudança de status para preservar a consistência. [ADR-001](adrs/ADR-001-outbox-transacional-mysql.md) · [ADR-002](adrs/ADR-002-worker-polling-outbox.md)
- **Retentativas limitadas e replay.** O backoff acomoda indisponibilidades temporárias. Uma falha persistente pode exigir ação administrativa. A contagem do envio inicial ainda precisa ser conciliada com os cinco intervalos registrados. [ADR-003](adrs/ADR-003-retentativas-com-backoff.md) · [ADR-004](adrs/ADR-004-dlq-e-replay-administrativo.md)
- **At-least-once.** O serviço pode reenviar uma notificação. O cliente usa o ID do evento para evitar processamento duplicado. Retentativas podem alterar a ordem de chegada. [ADR-002](adrs/ADR-002-worker-polling-outbox.md) · [ADR-006](adrs/ADR-006-entrega-at-least-once.md)

## Dependências

- Transições válidas e transação de pedidos já existentes no OMS. [Código de `OrderService.changeStatus`](../src/modules/orders/order.service.ts#L126)
- Endpoint HTTPS operado pelo cliente e integração capaz de validar a assinatura e deduplicar eventos. Marcos ficou de documentar o comportamento no portal de desenvolvedores. [Transcrição, 09:19–09:26 e 09:40](../TRANSCRICAO.md)
- Revisão de segurança de Sofia antes do deploy, com pelo menos dois dias úteis reservados. A estimativa inicial da equipe foi de três sprints, incluindo essa revisão. [Transcrição, 09:45–09:49](../TRANSCRICAO.md)

## Riscos e mitigação

As classificações de probabilidade e impacto abaixo são avaliações qualitativas propostas para revisão.

| Risco | Probabilidade | Impacto | Mitigação ou pendência |
| --- | --- | --- | --- |
| Atraso ou perda operacional de notificações durante indisponibilidade do cliente | Média | Alto | Aplicar timeout, backoff, histórico e DLQ com replay manual. [09:15–09:19 e 09:42](../TRANSCRICAO.md) |
| Processamento repetido no cliente | Média | Alto | Preservar o ID entre tentativas e documentar a deduplicação no portal. [09:24–09:26](../TRANSCRICAO.md) |
| Segredo comprometido | Baixa | Alto | Isolar o segredo por endpoint e permitir rotação. A janela mantém o segredo anterior válido por 24 horas. [09:21–09:23](../TRANSCRICAO.md) |
| Acesso aos cadastros ou ao histórico de outro cliente | Média com o desenho atual | Alto | Revisar a autorização por cliente antes da exposição externa. O JWT atual identifica usuário e papel, e o cliente é informado separadamente. [09:31–09:37](../TRANSCRICAO.md) · [FDD](FDD.md) |

## Critérios de aceitação

| ID | Cenário e resultado esperado | Requisitos |
| --- | --- | --- |
| PRD-AC-01 | Um usuário autenticado cadastra um endpoint HTTPS, recebe o segredo na criação, lista o cadastro, altera os filtros e consegue removê-lo. Uma URL HTTP é rejeitada. | FR-01, FR-02, FR-08, NFR-03 |
| PRD-AC-02 | Um endpoint que assina `SHIPPED` recebe o evento quando um pedido do cliente chega a esse status. Em cenário saudável, recebe em menos de 10 segundos. Uma transição para status fora do filtro, uma transição inválida ou uma transação revertida gera zero notificações para o endpoint. | FR-02, FR-03, NFR-01, NFR-02 |
| PRD-AC-03 | O destinatário valida a assinatura do corpo recebido. Durante a rotação, a integração com o segredo anterior continua funcionando por 24 horas. Após esse prazo, a integração usa o novo segredo. Payload acima do limite gera erro. | FR-04, NFR-03, NFR-04 |
| PRD-AC-04 | Após timeout, a entrega segue a política de retentativas definida para implementação e preserva o `X-Event-Id`. Esgotado o limite, a falha aparece na DLQ com motivo e timestamp. | FR-06, NFR-05, NFR-06 |
| PRD-AC-05 | Com mais de 100 tentativas registradas para um endpoint, a consulta retorna as 100 mais recentes com resultado e dados de diagnóstico disponíveis. | FR-05 |
| PRD-AC-06 | Um usuário sem papel `ADMIN` tem o replay recusado. Um administrador recoloca o evento como pendente e a auditoria identifica quem solicitou a operação. | FR-07 |

## Estratégia de testes e validação

- Validar pela API o cadastro, a edição, o filtro, a remoção e as permissões de replay.
- Em ambiente de integração, usar um destinatário controlado para medir a latência, validar as assinaturas e simular timeout, falha persistente, duplicata e recuperação.
- Confirmar em teste de integração que o rollback do pedido também reverte a notificação. Conferir histórico, DLQ e auditoria após os cenários de falha.
- Submeter geração de segredos, assinatura e rotação à revisão de Sofia antes do deploy. Completar os cenários de backoff quando a equipe resolver a contagem de tentativas.

[FDD — critérios técnicos](FDD.md#critérios-de-aceite-técnicos) · [Transcrição, 09:46–09:49](../TRANSCRICAO.md)
