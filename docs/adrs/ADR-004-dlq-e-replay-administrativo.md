# ADR-004 — DLQ persistida e replay administrativo

## Status

Aceito na reunião técnica.

## Contexto

Depois de esgotar as retentativas, a equipe precisa investigar a falha e poder reprocessar o evento.

## Decisão

Gravar falhas terminais na tabela MySQL `webhook_dead_letter`, com payload, motivo e timestamp. O endpoint `POST /admin/webhooks/dead-letter/:id/replay` recoloca o evento na outbox como pendente. Exigir autenticação e papel `ADMIN` com [`requireRole`](../../src/middlewares/auth.middleware.ts#L49) e registrar quem executou o replay para auditoria.

## Alternativas consideradas

- **Manter falhas terminais na outbox com status `failed`.** Evitaria outra tabela, mas manteria na mesma estrutura os registros de trabalho do worker e as evidências usadas em investigação.

## Consequências

A tabela dedicada facilita consultar falhas e recuperar eventos. O custo é manter uma tabela e um fluxo de replay adicionais. O replay depende de ação administrativa e está sujeito à [semântica de duplicatas](ADR-006-entrega-at-least-once.md).

## Fontes

- [Transcrição da reunião técnica](../../TRANSCRICAO.md), 09:17–09:19 e 09:35–09:36.
