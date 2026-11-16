# ADR-003 — Retentativas com backoff

## Status

Aceito na reunião técnica.

## Contexto

O destinatário pode ficar indisponível por horas, inclusive durante manutenção planejada. O [worker](ADR-002-worker-polling-outbox.md) precisa tolerar falhas temporárias sem manter eventos indefinidamente pendentes.

## Decisão

Retentar falhas de envio com backoff e encaminhar falhas terminais à [DLQ](ADR-004-dlq-e-replay-administrativo.md). O timeout HTTP de 10 segundos conta como falha de envio.

A reunião registrou cinco tentativas e cinco intervalos, de 1 min, 5 min, 30 min, 2 h e 12 h. Aplicar todos os intervalos após a tentativa inicial resulta em seis envios. A contagem da tentativa inicial precisa ser confirmada antes de fixar o cronograma de implementação.

## Alternativas consideradas

- **Três tentativas.** Reduziriam o trabalho do worker, mas poderiam esgotar a política durante uma manutenção de duas horas.
- **Retentativas indefinidas.** Dariam mais tempo para recuperação, com o custo de manter eventos pendentes para destinos permanentemente indisponíveis.

## Consequências

O backoff distribui as tentativas no tempo e o limite dá um destino operacional às falhas persistentes. Uma notificação pode atrasar por horas ou exigir replay manual após o esgotamento da política.

## Fontes

- [Transcrição da reunião técnica](../../TRANSCRICAO.md), 09:15–09:19, 09:42 e 09:48.
