# ADR-007 — Reuso dos padrões do projeto

## Status

Aceito na reunião técnica.

## Contexto

O serviço já organiza os domínios em módulos e compartilha validação, tratamento de erros e logging.

## Decisão

Estruturar o novo módulo em `src/modules/webhooks`, seguindo [o módulo de pedidos](../../src/modules/orders), com rotas, controller, service, repository e schemas Zod. Reutilizar [`AppError`](../../src/shared/errors/app-error.ts), o [middleware central de erros](../../src/middlewares/error.middleware.ts) na camada HTTP e o [logger Pino](../../src/shared/logger/index.ts) na API e no worker. Usar o prefixo `WEBHOOK_` nos códigos de erro do módulo. O ponto de entrada do worker segue o [ADR-002](ADR-002-worker-polling-outbox.md).

## Alternativas consideradas

- **Estrutura e infraestrutura próprias para webhooks.** Dariam mais independência ao módulo, mas duplicariam convenções de validação, registro e tratamento de erros que a equipe já usa.

## Consequências

As novas rotas seguem as convenções do restante da API. O worker reutiliza o logger e precisa tratar falhas em seu próprio fluxo, pois o middleware de erros atua nas requisições HTTP da API.

## Fontes

- [Transcrição da reunião técnica](../../TRANSCRICAO.md), 09:27–09:30 e 09:48.
