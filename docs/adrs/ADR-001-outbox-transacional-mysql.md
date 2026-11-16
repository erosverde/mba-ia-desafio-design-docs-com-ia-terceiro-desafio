# ADR-001 — Outbox transacional em MySQL

## Status

Aceito na reunião técnica.

## Contexto

Clientes B2B precisam receber notificações de mudança de status dos pedidos. Em [`OrderService.changeStatus`](../../src/modules/orders/order.service.ts#L126), o Prisma abre uma transação, lê o pedido, valida a transição, ajusta o estoque quando aplicável, atualiza o status e grava o histórico. A notificação precisa refletir uma transição confirmada. Uma falha entre o commit do pedido e uma publicação separada poderia deixar a mudança sem um evento registrado.

## Decisão

Persistir os eventos elegíveis na tabela `webhook_outbox` do MySQL existente, dentro da mesma transação Prisma que altera o pedido.

- Após gravar o histórico, chamar `publishWebhookEvent(tx, order, fromStatus, toStatus)` dentro do callback de `this.prisma.$transaction`. A função proposta usa o mesmo cliente transacional `tx` para consultar as configurações do cliente e gravar na outbox.
- Aplicar o filtro pelo novo status na inserção. Se nenhum webhook do cliente estiver inscrito nesse status, a transação pode confirmar a mudança sem inserir um evento.
- Gravar o evento `order.status_changed` com UUID e um snapshot do payload montado na inserção. Mudanças posteriores no pedido preservam o conteúdo já registrado.
- Para uma transição elegível, o commit confirma juntos o status, o histórico, os ajustes de estoque aplicáveis e o evento. Se a inserção falhar, propagar o erro para reverter toda a transação. Se a transação sofrer rollback por outro motivo, a linha da outbox também é revertida.

Um worker em processo separado lê os registros confirmados e envia as notificações HTTP. A garantia desta decisão é a persistência atômica do evento elegível. A entrega ao destinatário ocorre de forma assíncrona e pode gerar duplicatas.

## Alternativas consideradas

- **HTTP síncrono em `changeStatus`.** A latência do destinatário prolongaria a transação e a retenção de locks. O destinatário também poderia processar a notificação antes de um rollback no MySQL.
- **Redis Streams.** Um broker dedicado acrescentaria infraestrutura para a equipe operar. Publicar no Redis e confirmar a mudança no MySQL em operações separadas abriria uma janela de inconsistência que exigiria tratamento adicional.

## Consequências

- O evento elegível fica disponível para processamento mesmo se a API cair logo após o commit. Uma indisponibilidade do destinatário afeta o envio assíncrono.
- A consulta de inscrições e a inserção aumentam o trabalho dentro da transação. Se a gravação de um evento elegível falhar, a mudança de status também falha. Esse custo preserva a atomicidade entre a mudança e o registro do evento.
- A tabela acrescenta carga de leitura e armazenamento ao MySQL. A reunião prevê leitura em lotes pequenos e índices para estado e data de criação. O arquivamento de registros entregues fica para uma etapa posterior.

## Fontes

- [Transcrição da reunião técnica](../../TRANSCRICAO.md), 09:03–09:08 para alternativas e decisão, 09:33–09:34 para filtragem, 09:40–09:41 para integração transacional e 09:51–09:52 para UUID e snapshot.
- [Implementação atual de `OrderService.changeStatus`](../../src/modules/orders/order.service.ts#L126).
