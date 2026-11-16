# ADR-002 — Worker separado com polling da outbox

## Status

Aceito na reunião técnica.

## Contexto

O [ADR-001](ADR-001-outbox-transacional-mysql.md) registra os eventos na outbox do MySQL junto com a mudança de status do pedido. Os clientes consideram aceitável receber a notificação em menos de dez segundos. O envio precisa continuar disponível durante reinícios da API.

## Decisão

Executar uma única instância do worker em um processo Node separado da API. O worker usará o mesmo banco MySQL, a mesma configuração `DATABASE_URL` e sua própria instância de `PrismaClient`. A proposta é criar `src/worker.ts` como ponto de entrada, seguindo o padrão de [`src/server.ts`](../../src/server.ts), com execução via `npm run worker`.

A cada dois segundos, o worker consulta um lote pequeno de eventos pendentes elegíveis e prioriza os mais antigos por `created_at`. A consulta usa os índices de status e data de criação previstos para a outbox. O worker envia as notificações por HTTP e registra o resultado.

## Alternativas consideradas

- **Trigger no MySQL.** A trigger executaria SQL no banco. Acordar um processo externo exigiria um mecanismo adicional de sinalização.
- **Polling no processo da API.** Um reinício da API também interromperia o consumidor. Em um deploy com várias réplicas da API, seria preciso coordenar qual delas executaria o polling.

## Consequências

- O consumidor pode ser iniciado e reiniciado independentemente da API. A operação passa a incluir um processo adicional e sua conexão com o banco.
- O polling gera consultas periódicas mesmo com a outbox vazia. Em condições normais, um evento pode esperar cerca de dois segundos até a próxima consulta. Tempo de fila, duração do HTTP e retentativas também afetam a latência de entrega.
- Um único worker permite priorizar os eventos mais antigos por pedido na primeira tentativa. Retentativas podem fazer uma notificação anterior chegar depois de uma posterior. A ordenação global fica fora da garantia e a adoção de múltiplos workers exigiria rever a estratégia de ordenação por pedido.

## Fontes

- [Transcrição da reunião técnica](../../TRANSCRICAO.md), 09:02 para a latência esperada, 09:08–09:14 para polling, processo e ordenação, 09:15–09:19 para retentativas e 09:29–09:30 para a conexão do worker.
