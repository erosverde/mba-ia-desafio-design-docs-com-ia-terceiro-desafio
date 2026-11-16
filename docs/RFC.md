# RFC — Webhooks de mudança de status de pedidos

| Campo | Valor |
| --- | --- |
| Autor | Codex, para o desafio Design Docs do MBA |
| Data | 16 de novembro de 2026 |
| Status | Proposta para revisão |
| Revisores | Larissa, Marcos, Bruno, Diego e Sofia |

## Resumo executivo

Propomos webhooks de saída para avisar clientes B2B quando o status de um pedido muda. A API grava o evento elegível na mesma transação MySQL que confirma a mudança de status. Um worker separado consulta a outbox a cada dois segundos e envia notificações assinadas. Falhas temporárias levam a retentativas e falhas terminais ficam disponíveis para replay administrativo.

## Contexto e problema

Atlas Comercial, MaxDistribuição e Nova Cargo consultam periodicamente `GET /orders` para detectar mudanças. Isso aumenta o custo da integração e atrasa a reação do cliente. A expectativa relatada é receber notificações em menos de dez segundos. [Transcrição, 09:00–09:03](../TRANSCRICAO.md)

Hoje, [`OrderService.changeStatus`](../src/modules/orders/order.service.ts#L126) valida a transição e, na mesma transação Prisma, ajusta o estoque quando aplicável, atualiza o pedido e grava o histórico. Publicar em uma operação separada deixaria uma janela em que a mudança poderia ser confirmada sem registro do evento. [Transcrição, 09:03–09:08 e 09:40–09:41](../TRANSCRICAO.md)

## Proposta técnica

1. **Captura atômica.** Registrar na outbox do MySQL um snapshot do evento de mudança para o qual haja inscrição do cliente. A falha de gravação reverte a transação do pedido. A publicação HTTP ocorre em processo separado, após o commit.
2. **Processamento assíncrono.** Um único worker usa o mesmo MySQL e seu próprio Prisma Client. A cada dois segundos, busca um lote pequeno de pendências elegíveis, priorizando os registros mais antigos. A latência de entrega também depende da fila, do tempo de resposta do destino e das retentativas.
3. **Confiabilidade.** Adotar entrega at-least-once com `X-Event-Id` estável para deduplicação pelo destinatário. Repetir falhas transitórias com backoff limitado. A classificação das respostas HTTP fora de 2xx para retry ou DLQ está em revisão. Persistir falhas terminais em uma DLQ separada e oferecer replay manual com papel `ADMIN` e registro de quem o solicitou.
4. **Autenticação da notificação.** Exigir HTTPS e assinar o corpo com HMAC-SHA256. Cada endpoint tem seu próprio segredo e a rotação oferece uma janela de sobreposição de 24 horas.
5. **Operação pela API.** Oferecer cadastro, edição, remoção e listagem de endpoints, filtros por status e histórico de entregas. O JWT atual identifica usuário e papel. A regra de acesso ao cliente informado está em revisão antes de expor as rotas a usuários externos. Reutilizar a estrutura modular existente, schemas Zod, `AppError`, middleware central de erros e logging Pino.

[Transcrição, 09:08–09:36, 09:40–09:45 e 09:48–09:52](../TRANSCRICAO.md)

## Alternativas consideradas

| ID | Alternativa | Trade-off |
| --- | --- | --- |
| RFC-ALT-01 | Enviar HTTP de forma síncrona durante a mudança de status | Evitaria um consumidor separado, mas a latência e a disponibilidade do destinatário passariam a interferir na transação do pedido. [09:03–09:06](../TRANSCRICAO.md) |
| RFC-ALT-02 | Usar Redis Streams | Introduziria um sistema dedicado à fila, com infraestrutura adicional para a equipe operar e necessidade de coordenar a publicação com o commit em MySQL. [09:06–09:08](../TRANSCRICAO.md) |
| RFC-ALT-03 | Usar trigger MySQL para acordar o consumidor | Poderia reagir à escrita no banco, mas ainda precisaria de um mecanismo externo para sinalizar o worker. [09:09–09:10](../TRANSCRICAO.md) |

## Questões em aberto

| ID | Questão a revisar | Origem |
| --- | --- | --- |
| RFC-OPEN-01 | As cinco tentativas incluem o envio inicial? Os cinco intervalos citados implicam seis envios se aplicados após o inicial. Definir a contagem antes de fechar o desenho do worker. | [09:15–09:17 e 09:48](../TRANSCRICAO.md) |
| RFC-OPEN-02 | Quais sinais de volume, falha ou saturação justificariam limitar a taxa de envio por destino? A reunião decidiu observar o comportamento antes de definir um limite. | [09:38–09:39](../TRANSCRICAO.md) |
| RFC-OPEN-03 | Qual estratégia de ordenação por pedido adotar se a carga exigir múltiplos workers? Particionamento por pedido e lock pessimista foram levantados para avaliação futura. | [09:12–09:14](../TRANSCRICAO.md) |
| RFC-OPEN-04 | Qual política de retenção e arquivamento aplicar à outbox? A reunião mencionou cerca de 30 dias para registros entregues e adiou o trabalho de arquivamento. | [09:08](../TRANSCRICAO.md) |
| RFC-OPEN-05 | Como vincular usuários autenticados aos clientes cujos endpoints e históricos podem acessar? A reunião definiu o cliente como parâmetro explícito e o JWT atual identifica usuário e papel. | [09:31–09:37](../TRANSCRICAO.md) · [Código de autenticação](../src/middlewares/auth.middleware.ts) |
| RFC-OPEN-06 | Qual formato permite a sobreposição dos segredos por 24 horas e como tratar nova rotação dentro dessa janela? Avaliar adiar a segunda rotação ou manter vários segredos anteriores válidos. | [09:21–09:22 e 09:46–09:49](../TRANSCRICAO.md) |
| RFC-OPEN-07 | Quais respostas HTTP fora de 2xx devem gerar retry e quais são falhas terminais? A reunião definiu retry após timeout e DLQ após esgotar tentativas. A classificação dos códigos HTTP precisa de revisão. | [09:15–09:19 e 09:42](../TRANSCRICAO.md) |

## Impacto e riscos

| Impacto ou risco | Como a proposta trata |
| --- | --- |
| Carga adicional no MySQL | Limitar as leituras a lotes pequenos, com índices para status e data de criação. A escrita adicional integra a transação do pedido. [09:06–09:09 e 09:40–09:41](../TRANSCRICAO.md) |
| Destino lento ou indisponível | Isolar o HTTP no worker, aplicar timeout de dez segundos e usar backoff e DLQ. A meta de latência depende de condições normais de entrega. [09:15–09:19 e 09:42](../TRANSCRICAO.md) |
| Duplicatas e entregas fora de ordem durante retentativas | Documentar a deduplicação por ID para consumidores. Um worker prioriza os eventos antigos, mas retentativas podem alterar a ordem de chegada. [09:12–09:17 e 09:24–09:26](../TRANSCRICAO.md) |
| Vazamento de segredo | Isolar a credencial por endpoint, oferecer rotação e incluir revisão de segurança antes do deploy. [09:19–09:23 e 09:46–09:49](../TRANSCRICAO.md) |

## Decisões relacionadas

| ADR | Decisão |
| --- | --- |
| [001](adrs/ADR-001-outbox-transacional-mysql.md) | Outbox transacional em MySQL |
| [002](adrs/ADR-002-worker-polling-outbox.md) | Worker separado com polling da outbox |
| [003](adrs/ADR-003-retentativas-com-backoff.md) | Retentativas com backoff |
| [004](adrs/ADR-004-dlq-e-replay-administrativo.md) | DLQ e replay administrativo |
| [005](adrs/ADR-005-assinatura-hmac-por-endpoint.md) | HMAC-SHA256 por endpoint |
| [006](adrs/ADR-006-entrega-at-least-once.md) | Entrega at-least-once e deduplicação |
| [007](adrs/ADR-007-reuso-dos-padroes-do-projeto.md) | Reuso dos padrões do projeto |
| [008](adrs/ADR-008-contrato-do-evento-de-status.md) | Contrato do evento de mudança de status |
