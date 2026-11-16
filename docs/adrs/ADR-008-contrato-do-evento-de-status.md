# ADR-008 — Contrato do evento de mudança de status

## Status

Aceito na reunião técnica.

## Contexto

O destinatário precisa identificar o evento e o endpoint e saber qual transição ocorreu, inclusive quando o pedido já avançou para outro status.

## Decisão

Enviar um JSON `order.status_changed` com `event_id`, `event_type`, `timestamp` em ISO 8601, `order_id`, `order_number`, `from_status`, `to_status`, `customer_id` e dados básicos como `total_cents`. Persistir o snapshot conforme o [ADR-001](ADR-001-outbox-transacional-mysql.md). Para consultar itens e outros detalhes, o destinatário pode usar `GET /orders/:id`.

Enviar `Content-Type: application/json`, `X-Event-Id`, `X-Signature`, `X-Timestamp` com o instante do envio e `X-Webhook-Id` com o ID do endpoint. O timestamp no corpo pertence ao evento.

## Alternativas consideradas

- **Enviar o pedido completo.** Pouparia consultas de alguns consumidores, mas aumentaria o payload e acoplaria a notificação à estrutura dos itens.
- **Montar o evento no envio a partir do pedido atual.** Reduziria os dados persistidos, mas poderia refletir uma versão posterior do pedido.

## Consequências

O corpo permanece estável entre tentativas, enquanto o instante de envio pode variar. Consumidores que precisarem de mais detalhes terão de consultar a API, que retorna o estado atual do pedido.

## Fontes

- [Transcrição da reunião técnica](../../TRANSCRICAO.md), 09:43–09:45 e 09:51–09:52.
