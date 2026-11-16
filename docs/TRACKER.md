# Tracker de rastreabilidade

As linhas marcadas como proposta citam o requisito ou o padrão que motivou o desenho. As fontes de critérios de aceite são as dos requisitos que cada cenário verifica. Os IDs adicionais identificam trechos sem ID próprio nos documentos.

### Inventário e cálculo da cobertura

O denominador vem da leitura dos documentos de origem. Extraia os IDs explícitos nas tabelas e nos títulos do PRD, do RFC e do FDD e os oito números nos títulos dos ADRs. Para as seções sem ID próprio, some os grupos abaixo. As faixas são inclusivas e os prefixos pertencem ao documento indicado.

| Documento | IDs explícitos | Grupos sem ID próprio no documento de origem | Itens identificados |
| --- | ---: | --- | ---: |
| [PRD](PRD.md) | 22 | Contexto e motivação (CONTEXTO-01), públicos e cenários (CENARIO-01–02), exclusões (ESCOPO-01–05), decisões e trade-offs (TRD-01–02), dependências (DEP-01–02), riscos (RISCO-01–04), estratégia de validação (VALID-01) | 39 |
| [RFC](RFC.md) | 11 | Contexto (CONTEXTO-01), cinco pontos da proposta (PROP-01–05), quatro linhas de impacto e risco (RISCO-01–04) | 21 |
| [FDD](FDD.md) | 13 | Objetivos (OBJ-01), exclusões (ESCOPO-01), modelagem geral e quatro tabelas (DADOS-01–05), resiliência de reserva e replay (RES-01–02), formato do número do pedido (CONTRATO-02-FORMATO), integração (INT-01–03), observabilidade (OBS-01), riscos (RISCO-01–05), implantação e compatibilidade (DEP-01) | 33 |
| [ADRs](adrs/) | 8 | Cada ADR conta como uma decisão, incluindo contexto, alternativas e consequências | 8 |
| **Total** | **54** | **47 grupos** | **101** |

O resumo executivo e as listas de escopo que retomam requisitos já inventariados contam junto desses requisitos. Rotas e exemplos de gerenciamento contam em CONTRATO-01, o contrato de saída em CONTRATO-02 e o histórico em CONTRATO-03. O README descreve o processo com base nos prompts e no histórico do Git e fica fora do inventário técnico.

Para reproduzir a conta, expanda as faixas, acrescente o prefixo do documento e procure cada ID nas tabelas abaixo. Conte cada correspondência uma vez, excluindo linhas auxiliares com sufixo `-CODIGO`. Uma linha cuja fonte principal é código, como `FDD-CONTRATO-02-FORMATO`, conta como item próprio. **Cobertura do inventário: 101 mapeados ÷ 101 identificados (100%).** Incluindo as 20 referências complementares, são 121 linhas de evidência, das quais 100 citam a transcrição (82,6%) e 21 citam código.

## PRD

| ID | Documento | Tipo | Conteúdo (resumo) | Fonte | Localização |
| --- | --- | --- | --- | --- | --- |
| PRD-OBJ-01 | [docs/PRD.md](PRD.md) | Objetivo | Entrega abaixo de 10 s em condições normais. Medição ponta a ponta proposta | TRANSCRICAO | [09:02] Marcos, [09:10] Marcos |
| PRD-OBJ-02 | [docs/PRD.md](PRD.md) | Objetivo | Reduzir polling pelos clientes. Comparação antes e depois proposta | TRANSCRICAO | [09:00] Marcos |
| PRD-FR-01 | [docs/PRD.md](PRD.md) | Requisito funcional | Cadastro autenticado com cliente explícito e segredo na criação | TRANSCRICAO | [09:21] Sofia, [09:31] Marcos, [09:32] Larissa |
| PRD-FR-02 | [docs/PRD.md](PRD.md) | Requisito funcional | Filtrar pelo novo status e pelas inscrições do cliente antes de enfileirar | TRANSCRICAO | [09:33] Marcos, [09:34] Bruno, [09:40] Bruno |
| PRD-FR-03 | [docs/PRD.md](PRD.md) | Requisito funcional | Evento de mudança com UUID estável e snapshot | TRANSCRICAO | [09:25] Diego, [09:43] Diego, [09:52] Larissa |
| PRD-FR-04 | [docs/PRD.md](PRD.md) | Requisito funcional | Rotação pela API com sobreposição de 24 h | TRANSCRICAO | [09:21] Sofia, [09:22] Sofia |
| PRD-FR-05 | [docs/PRD.md](PRD.md) | Requisito funcional | Consultar as últimas 100 tentativas e seus dados de diagnóstico | TRANSCRICAO | [09:34] Marcos |
| PRD-FR-06 | [docs/PRD.md](PRD.md) | Requisito funcional | Backoff limitado e DLQ com payload, motivo e timestamp | TRANSCRICAO | [09:17] Larissa, [09:18] Diego, [09:48] Larissa |
| PRD-FR-07 | [docs/PRD.md](PRD.md) | Requisito funcional | Replay manual por ADMIN, reenvio à outbox e auditoria | TRANSCRICAO | [09:18] Diego, [09:36] Sofia, [09:36] Larissa |
| PRD-FR-08 | [docs/PRD.md](PRD.md) | Requisito funcional | Listar, editar e remover cadastros via API autenticada | TRANSCRICAO | [09:33] Bruno, [09:37] Sofia |
| PRD-NFR-01 | [docs/PRD.md](PRD.md) | Requisito não funcional | Latência abaixo de 10 s no cenário normal | TRANSCRICAO | [09:02] Marcos |
| PRD-NFR-02 | [docs/PRD.md](PRD.md) | Requisito não funcional | Captura atômica e envio assíncrono em processo separado | TRANSCRICAO | [09:06] Diego, [09:11] Diego, [09:40] Bruno |
| PRD-NFR-03 | [docs/PRD.md](PRD.md) | Requisito não funcional | HMAC-SHA256, segredo por endpoint e HTTPS | TRANSCRICAO | [09:22] Sofia, [09:23] Sofia |
| PRD-NFR-04 | [docs/PRD.md](PRD.md) | Restrição | Limite de 64 KB com erro se excedido | TRANSCRICAO | [09:24] Larissa |
| PRD-NFR-05 | [docs/PRD.md](PRD.md) | Requisito não funcional | At-least-once com deduplicação pelo consumidor | TRANSCRICAO | [09:19] Larissa, [09:25] Diego, [09:26] Larissa |
| PRD-NFR-06 | [docs/PRD.md](PRD.md) | Restrição | Timeout HTTP de 10 s conta como falha | TRANSCRICAO | [09:42] Diego |
| PRD-AC-01 | [docs/PRD.md](PRD.md) | Critério de aceite | Exercitar CRUD, filtros e rejeição de HTTP | TRANSCRICAO | [09:23] Sofia, [09:31] Marcos, [09:33] Bruno, [09:33] Marcos |
| PRD-AC-02 | [docs/PRD.md](PRD.md) | Critério de aceite | Entregar SHIPPED no cenário saudável e verificar filtro e rollback | TRANSCRICAO | [09:02] Marcos, [09:33] Marcos, [09:40] Bruno |
| PRD-AC-02-CODIGO | [docs/PRD.md](PRD.md) | Referência de código | Base existente para PRD-AC-02 | CODIGO | [src/modules/orders/order.service.ts](../src/modules/orders/order.service.ts#L126) |
| PRD-AC-03 | [docs/PRD.md](PRD.md) | Critério de aceite | Verificar assinatura, rotação, expiração e limite de tamanho | TRANSCRICAO | [09:22] Sofia, [09:24] Larissa |
| PRD-AC-04 | [docs/PRD.md](PRD.md) | Critério de aceite | Simular timeout, verificar ID estável, retentativas e DLQ | TRANSCRICAO | [09:17] Larissa, [09:18] Diego, [09:25] Diego, [09:42] Diego |
| PRD-AC-05 | [docs/PRD.md](PRD.md) | Critério de aceite | Consultar as 100 tentativas mais recentes | TRANSCRICAO | [09:34] Marcos |
| PRD-AC-06 | [docs/PRD.md](PRD.md) | Critério de aceite | Recusar replay sem ADMIN e auditar o replay autorizado | TRANSCRICAO | [09:18] Diego, [09:36] Sofia, [09:36] Larissa |
| PRD-ESCOPO-01 | [docs/PRD.md](PRD.md) | Exclusão | Webhooks de entrada | TRANSCRICAO | [09:02] Marcos |
| PRD-ESCOPO-02 | [docs/PRD.md](PRD.md) | Exclusão | Email de alerta ou fallback | TRANSCRICAO | [09:37] Larissa |
| PRD-ESCOPO-03 | [docs/PRD.md](PRD.md) | Exclusão | Dashboard visual | TRANSCRICAO | [09:40] Larissa |
| PRD-ESCOPO-04 | [docs/PRD.md](PRD.md) | Exclusão | Rate limiting de saída adiado | TRANSCRICAO | [09:39] Diego, [09:39] Larissa |
| PRD-ESCOPO-05 | [docs/PRD.md](PRD.md) | Exclusão | Arquivamento automático da outbox adiado | TRANSCRICAO | [09:08] Diego |
| PRD-DEP-01 | [docs/PRD.md](PRD.md) | Dependência | Revisão de Sofia antes do deploy com reserva de dois dias úteis e estimativa de três sprints | TRANSCRICAO | [09:46] Sofia, [09:47] Larissa |
| PRD-DEP-02 | [docs/PRD.md](PRD.md) | Dependência | Transações de pedidos e documentação para integradores | TRANSCRICAO | [09:26] Marcos, [09:40] Marcos |
| PRD-DEP-02-CODIGO | [docs/PRD.md](PRD.md) | Referência de código | Base existente para PRD-DEP-02 | CODIGO | [src/modules/orders/order.service.ts](../src/modules/orders/order.service.ts#L126) |
| PRD-RISCO-01 | [docs/PRD.md](PRD.md) | Pendência | Definir autorização por cliente. JWT atual contém usuário e papel | TRANSCRICAO | [09:32] Larissa, [09:37] Sofia |
| PRD-RISCO-01-CODIGO | [docs/PRD.md](PRD.md) | Referência de código | Base existente para PRD-RISCO-01 | CODIGO | [src/middlewares/auth.middleware.ts](../src/middlewares/auth.middleware.ts) |
| PRD-CONTEXTO-01 | [docs/PRD.md](PRD.md) | Contexto | Atlas, MaxDistribuição e Nova Cargo usam polling. A Atlas sinalizou risco de migração e pediu entrega até fim de novembro | TRANSCRICAO | [09:00] Marcos, [09:45] Marcos, [09:47] Marcos |
| PRD-CENARIO-01 | [docs/PRD.md](PRD.md) | Cenário | Integrador filtra por status, recebe evento e consulta detalhes do pedido quando precisar | TRANSCRICAO | [09:33] Marcos, [09:43] Diego |
| PRD-CENARIO-02 | [docs/PRD.md](PRD.md) | Cenário | Operação consulta histórico e administração recupera falhas | TRANSCRICAO | [09:34] Marcos, [09:36] Sofia |
| PRD-TRD-01 | [docs/PRD.md](PRD.md) | Trade-off | Isolar HTTP no worker e reverter transição quando falhar a captura obrigatória | TRANSCRICAO | [09:06] Diego, [09:11] Diego, [09:40] Bruno |
| PRD-TRD-02 | [docs/PRD.md](PRD.md) | Trade-off | Limitar retry, admitir duplicatas e usar replay após falha terminal | TRANSCRICAO | [09:17] Larissa, [09:18] Diego, [09:26] Larissa |
| PRD-RISCO-02 | [docs/PRD.md](PRD.md) | Risco | Atraso por destino indisponível. Timeout, backoff e DLQ | TRANSCRICAO | [09:15] Diego, [09:18] Diego, [09:42] Diego |
| PRD-RISCO-03 | [docs/PRD.md](PRD.md) | Risco | Duplicatas. ID estável e deduplicação no consumidor | TRANSCRICAO | [09:24] Diego, [09:25] Diego, [09:26] Marcos |
| PRD-RISCO-04 | [docs/PRD.md](PRD.md) | Risco | Vazamento de segredo. Isolamento e rotação | TRANSCRICAO | [09:21] Sofia, [09:22] Sofia |
| PRD-VALID-01 | [docs/PRD.md](PRD.md) | Estratégia de validação proposta | Cobrir contrato, entrega, rollback, segurança e recuperação com testes de API e integração | TRANSCRICAO | [09:40] Bruno, [09:46] Larissa, [09:46] Sofia |

## RFC

| ID | Documento | Tipo | Conteúdo (resumo) | Fonte | Localização |
| --- | --- | --- | --- | --- | --- |
| RFC-ALT-01 | [docs/RFC.md](RFC.md) | Trade-off | HTTP síncrono prolongaria a transação e a tornaria dependente do destino | TRANSCRICAO | [09:04] Bruno |
| RFC-ALT-02 | [docs/RFC.md](RFC.md) | Trade-off | Redis Streams acrescentaria infraestrutura e coordenação com o commit | TRANSCRICAO | [09:07] Larissa, [09:07] Diego |
| RFC-ALT-03 | [docs/RFC.md](RFC.md) | Trade-off | Trigger exigiria sinalização externa para acordar o worker | TRANSCRICAO | [09:09] Bruno, [09:09] Diego |
| RFC-OPEN-01 | [docs/RFC.md](RFC.md) | Pendência | Conciliar cinco tentativas com cinco intervalos de backoff | TRANSCRICAO | [09:17] Diego, [09:17] Larissa, [09:48] Larissa |
| RFC-OPEN-02 | [docs/RFC.md](RFC.md) | Pendência | Observar volume e decidir limite de envios por destino | TRANSCRICAO | [09:38] Diego, [09:39] Diego, [09:39] Larissa |
| RFC-OPEN-03 | [docs/RFC.md](RFC.md) | Pendência | Rever ordenação por pedido ao adotar múltiplos workers | TRANSCRICAO | [09:12] Diego, [09:13] Diego, [09:13] Larissa |
| RFC-OPEN-04 | [docs/RFC.md](RFC.md) | Pendência | Definir retenção e arquivamento de entregues | TRANSCRICAO | [09:08] Diego |
| RFC-OPEN-05 | [docs/RFC.md](RFC.md) | Pendência | Definir autorização por cliente | TRANSCRICAO | [09:32] Larissa, [09:37] Sofia |
| RFC-OPEN-05-CODIGO | [docs/RFC.md](RFC.md) | Referência de código | Base existente para RFC-OPEN-05 | CODIGO | [src/middlewares/auth.middleware.ts](../src/middlewares/auth.middleware.ts) |
| RFC-OPEN-06 | [docs/RFC.md](RFC.md) | Pendência | Validar assinatura durante a sobreposição e comportamento de rotações sucessivas em 24 h | TRANSCRICAO | [09:21] Sofia, [09:22] Sofia, [09:46] Sofia |
| RFC-OPEN-07 | [docs/RFC.md](RFC.md) | Pendência | Classificar respostas fora de 2xx para retry ou DLQ. Timeout segue para retry | TRANSCRICAO | [09:15] Diego, [09:18] Diego, [09:42] Diego |
| RFC-OPEN-08 | [docs/RFC.md](RFC.md) | Pendência | Decidir se o backoff deve bloquear eventos posteriores do mesmo pedido e endpoint com um único worker | TRANSCRICAO | [09:12] Diego, [09:13] Larissa, [09:15] Diego |
| RFC-PROP-01 | [docs/RFC.md](RFC.md) | Decisão | Captura atômica, filtro na inserção e snapshot | TRANSCRICAO | [09:34] Bruno, [09:40] Bruno, [09:52] Larissa |
| RFC-PROP-02 | [docs/RFC.md](RFC.md) | Decisão | Worker único separado, polling de 2 s e lotes pequenos | TRANSCRICAO | [09:08] Diego, [09:09] Diego, [09:11] Diego, [09:12] Diego, [09:30] Bruno |
| RFC-PROP-03 | [docs/RFC.md](RFC.md) | Decisão | At-least-once, backoff limitado, DLQ e replay auditado por ADMIN | TRANSCRICAO | [09:17] Larissa, [09:18] Diego, [09:26] Larissa, [09:36] Sofia |
| RFC-PROP-04 | [docs/RFC.md](RFC.md) | Decisão | HTTPS, HMAC e segredo por endpoint com sobreposição de 24 h | TRANSCRICAO | [09:22] Sofia, [09:23] Sofia |
| RFC-PROP-05 | [docs/RFC.md](RFC.md) | Decisão | CRUD, filtros e histórico pela API com reuso dos padrões locais | TRANSCRICAO | [09:30] Larissa, [09:33] Bruno, [09:34] Marcos |
| RFC-RISCO-01 | [docs/RFC.md](RFC.md) | Risco | Carga no MySQL mitigada por lotes pequenos e índices | TRANSCRICAO | [09:07] Bruno, [09:08] Diego |
| RFC-RISCO-02 | [docs/RFC.md](RFC.md) | Risco | Indisponibilidade do destino mitigada com timeout, backoff e DLQ | TRANSCRICAO | [09:15] Diego, [09:18] Diego, [09:42] Diego |
| RFC-RISCO-03 | [docs/RFC.md](RFC.md) | Risco | Duplicatas e risco de inversão na entrega, deduplicação por ID | TRANSCRICAO | [09:12] Diego, [09:13] Larissa, [09:25] Diego |
| RFC-RISCO-04 | [docs/RFC.md](RFC.md) | Risco | Vazamento de segredo, rotação e revisão de segurança | TRANSCRICAO | [09:21] Sofia, [09:22] Diego, [09:46] Sofia |
| RFC-CONTEXTO-01 | [docs/RFC.md](RFC.md) | Contexto | Polling B2B motiva entrega assíncrona com captura na transação existente | TRANSCRICAO | [09:00] Marcos, [09:06] Diego, [09:40] Bruno |

## FDD

| ID | Documento | Tipo | Conteúdo (resumo) | Fonte | Localização |
| --- | --- | --- | --- | --- | --- |
| FDD-FLUXO-01 | [docs/FDD.md](FDD.md) | Fluxo | Usar o mesmo tx para filtrar, serializar e inserir antes do commit. Limite em bytes é proposta | TRANSCRICAO | [09:24] Larissa, [09:34] Bruno, [09:40] Bruno, [09:41] Bruno, [09:52] Larissa |
| FDD-FLUXO-01-CODIGO | [docs/FDD.md](FDD.md) | Referência de código | Base existente para FDD-FLUXO-01 | CODIGO | [src/modules/orders/order.service.ts](../src/modules/orders/order.service.ts#L126) |
| FDD-FLUXO-02 | [docs/FDD.md](FDD.md) | Fluxo | Polling, envio, timeout, histórico, retry e DLQ. Validar destino antes de abrir a tentativa. Cancelamento de pendências por remoção é proposta. Classificação HTTP em revisão | TRANSCRICAO | [09:08] Diego, [09:09] Diego, [09:17] Larissa, [09:18] Diego, [09:34] Marcos, [09:42] Diego |
| FDD-FLUXO-03 | [docs/FDD.md](FDD.md) | Fluxo | Recolocar pendência com auditoria. Preservar ID e vincular replay a uma entrada e a um ciclo são detalhes propostos | TRANSCRICAO | [09:18] Diego, [09:25] Diego, [09:36] Sofia |
| FDD-FLUXO-03-CODIGO | [docs/FDD.md](FDD.md) | Referência de código | Base existente para FDD-FLUXO-03 | CODIGO | [src/middlewares/auth.middleware.ts](../src/middlewares/auth.middleware.ts#L49) |
| FDD-CONTRATO-01 | [docs/FDD.md](FDD.md) | Contrato | API autenticada de CRUD, rotação, histórico e replay. Rotas e respostas não explicitadas na reunião são propostas | TRANSCRICAO | [09:21] Sofia, [09:31] Marcos, [09:32] Larissa, [09:33] Bruno, [09:34] Marcos, [09:35] Diego, [09:36] Larissa |
| FDD-CONTRATO-01-CODIGO | [docs/FDD.md](FDD.md) | Referência de código | Base existente para FDD-CONTRATO-01 | CODIGO | [src/routes/index.ts](../src/routes/index.ts) |
| FDD-CONTRATO-02 | [docs/FDD.md](FDD.md) | Contrato | JSON do evento e headers. Encoding e duas assinaturas para uma rotação são propostas. Rotações sucessivas seguem em revisão | TRANSCRICAO | [09:22] Sofia, [09:43] Diego, [09:44] Diego, [09:44] Sofia, [09:52] Larissa |
| FDD-CONTRATO-03 | [docs/FDD.md](FDD.md) | Contrato proposto | Histórico das últimas 100 tentativas finalizadas, com snapshot, resposta e duração quando registradas e resultados de timeout e recuperação | TRANSCRICAO | [09:34] Marcos, [09:42] Diego, [09:52] Larissa |
| FDD-ERRO-01 | [docs/FDD.md](FDD.md) | Contrato | Códigos WEBHOOK_* propostos e envelope existente de AppError | TRANSCRICAO | [09:28] Bruno, [09:29] Larissa |
| FDD-ERRO-01-CODIGO | [docs/FDD.md](FDD.md) | Referência de código | Base existente para FDD-ERRO-01 | CODIGO | [src/middlewares/error.middleware.ts](../src/middlewares/error.middleware.ts), [src/shared/errors/app-error.ts](../src/shared/errors/app-error.ts) |
| FDD-DADOS-01 | [docs/FDD.md](FDD.md) | Proposta de modelagem | Configuração, outbox, histórico e DLQ. Uma linha por destino, índices e desativação são detalhes de desenho | TRANSCRICAO | [09:08] Diego, [09:18] Diego, [09:21] Bruno, [09:21] Sofia, [09:34] Marcos, [09:51] Larissa, [09:52] Larissa |
| FDD-DADOS-01-CODIGO | [docs/FDD.md](FDD.md) | Referência de código | Base existente para FDD-DADOS-01 | CODIGO | [prisma/schema.prisma](../prisma/schema.prisma) |
| FDD-INT-01 | [docs/FDD.md](FDD.md) | Integração | Captura na transação atual e novos modelos Prisma com OrderStatus | TRANSCRICAO | [09:40] Bruno, [09:41] Bruno |
| FDD-INT-01-CODIGO | [docs/FDD.md](FDD.md) | Referência de código | Base existente para FDD-INT-01 | CODIGO | [src/modules/orders/order.service.ts](../src/modules/orders/order.service.ts#L126), [prisma/schema.prisma](../prisma/schema.prisma) |
| FDD-INT-02 | [docs/FDD.md](FDD.md) | Integração | Composição Express, rotas, autenticação e validação Zod | TRANSCRICAO | [09:27] Bruno, [09:30] Larissa, [09:36] Larissa |
| FDD-INT-02-CODIGO | [docs/FDD.md](FDD.md) | Referência de código | Base existente para FDD-INT-02 | CODIGO | [src/app.ts](../src/app.ts), [src/routes/index.ts](../src/routes/index.ts), [src/middlewares/auth.middleware.ts](../src/middlewares/auth.middleware.ts), [src/middlewares/validate.middleware.ts](../src/middlewares/validate.middleware.ts) |
| FDD-INT-03 | [docs/FDD.md](FDD.md) | Integração | Worker separado com Prisma por processo e logging compartilhado | TRANSCRICAO | [09:11] Larissa, [09:29] Bruno, [09:30] Bruno |
| FDD-INT-03-CODIGO | [docs/FDD.md](FDD.md) | Referência de código | Base existente para FDD-INT-03 | CODIGO | [src/server.ts](../src/server.ts), [src/config/database.ts](../src/config/database.ts), [src/shared/logger/index.ts](../src/shared/logger/index.ts) |
| FDD-OBS-01 | [docs/FDD.md](FDD.md) | Proposta de observabilidade | Medir fila, latência, falhas e DLQ. Correlacionar por IDs e auditar replay com Pino | TRANSCRICAO | [09:25] Diego, [09:29] Bruno, [09:34] Marcos, [09:36] Sofia |
| FDD-OBS-01-CODIGO | [docs/FDD.md](FDD.md) | Referência de código | Base existente para FDD-OBS-01 | CODIGO | [src/shared/logger/index.ts](../src/shared/logger/index.ts) |
| FDD-AC-01 | [docs/FDD.md](FDD.md) | Critério de aceite | Verificar commit e rollback de pedido, histórico, estoque e outbox | TRANSCRICAO | [09:40] Bruno |
| FDD-AC-01-CODIGO | [docs/FDD.md](FDD.md) | Referência de código | Base existente para FDD-AC-01 | CODIGO | [src/modules/orders/order.service.ts](../src/modules/orders/order.service.ts#L126), [tests/orders.test.ts](../tests/orders.test.ts) |
| FDD-AC-02 | [docs/FDD.md](FDD.md) | Critério de aceite | Validar transição, filtro e snapshot imutável | TRANSCRICAO | [09:34] Bruno, [09:52] Larissa |
| FDD-AC-02-CODIGO | [docs/FDD.md](FDD.md) | Referência de código | Base existente para FDD-AC-02 | CODIGO | [src/modules/orders/order.service.ts](../src/modules/orders/order.service.ts#L126) |
| FDD-AC-03 | [docs/FDD.md](FDD.md) | Critério de aceite | Validar polling, timeout, retry e DLQ. Recuperação de reserva é teste do modelo proposto | TRANSCRICAO | [09:09] Diego, [09:17] Larissa, [09:18] Diego, [09:42] Diego |
| FDD-AC-04 | [docs/FDD.md](FDD.md) | Critério de aceite | Simular resposta perdida e medir latência com destino saudável | TRANSCRICAO | [09:02] Marcos, [09:24] Diego, [09:25] Diego, [09:52] Larissa |
| FDD-AC-05 | [docs/FDD.md](FDD.md) | Critério de aceite | Testar HMAC, HTTPS, tamanho e expiração do segredo anterior | TRANSCRICAO | [09:22] Sofia, [09:23] Sofia, [09:24] Larissa |
| FDD-AC-06 | [docs/FDD.md](FDD.md) | Critério de aceite | Testar as 100 tentativas mais recentes, permissões e auditoria de replay | TRANSCRICAO | [09:34] Marcos, [09:36] Sofia, [09:36] Larissa |
| FDD-AC-06-CODIGO | [docs/FDD.md](FDD.md) | Referência de código | Base existente para FDD-AC-06 | CODIGO | [src/middlewares/auth.middleware.ts](../src/middlewares/auth.middleware.ts#L49) |
| FDD-OBJ-01 | [docs/FDD.md](FDD.md) | Objetivo técnico | Captura atômica, latência abaixo de 10 s em condições normais e recuperação assíncrona | TRANSCRICAO | [09:02] Marcos, [09:06] Diego, [09:18] Diego |
| FDD-ESCOPO-01 | [docs/FDD.md](FDD.md) | Exclusão | Arquivamento, email, rate limiting e painel adiados ou fora do escopo | TRANSCRICAO | [09:08] Diego, [09:37] Larissa, [09:39] Larissa, [09:40] Larissa |
| FDD-DADOS-02 | [docs/FDD.md](FDD.md) | Proposta de modelagem | Cadastro por cliente e estado, filtros e segredo anterior com expiração | TRANSCRICAO | [09:21] Bruno, [09:21] Sofia, [09:33] Marcos |
| FDD-DADOS-03 | [docs/FDD.md](FDD.md) | Proposta de modelagem | Outbox por destino. Índices propostos para vencimento e ordem de criação | TRANSCRICAO | [09:08] Diego, [09:09] Diego, [09:17] Larissa, [09:34] Bruno |
| FDD-DADOS-04 | [docs/FDD.md](FDD.md) | Proposta de modelagem | Histórico por destino com estado da tentativa, resposta, duração e snapshot | TRANSCRICAO | [09:34] Marcos, [09:52] Larissa |
| FDD-DADOS-05 | [docs/FDD.md](FDD.md) | Proposta de modelagem | DLQ com payload, motivo, timestamp e auditoria. Entrada única por ciclo é detalhe proposto | TRANSCRICAO | [09:18] Diego, [09:36] Sofia |
| FDD-RES-01 | [docs/FDD.md](FDD.md) | Proposta de resiliência | Reserva sem envio aberto pode expirar sem custo. Tentativa aberta consome orçamento e tem resultado indeterminado na recuperação | TRANSCRICAO | [09:09] Diego, [09:17] Larissa, [09:25] Diego, [09:42] Diego |
| FDD-RES-02 | [docs/FDD.md](FDD.md) | Proposta de resiliência | Replay de uso único por entrada da DLQ. Novo ciclo terminal gera nova entrada e preserva a identidade do evento | TRANSCRICAO | [09:18] Diego, [09:25] Diego, [09:36] Sofia |
| FDD-CONTRATO-02-FORMATO | [docs/FDD.md](FDD.md) | Compatibilidade | Copiar order.orderNumber para order_number. O exemplo usa ORD-000042 conforme o gerador existente | CODIGO | [src/modules/orders/order.service.ts](../src/modules/orders/order.service.ts#L253) |
| FDD-RISCO-01 | [docs/FDD.md](FDD.md) | Risco | Backlog e pressão no MySQL. Limitar lotes e acompanhar idade da fila | TRANSCRICAO | [09:07] Bruno, [09:08] Diego |
| FDD-RISCO-02 | [docs/FDD.md](FDD.md) | Risco | Duplicata e inversão com retry. Preservar ID e rever ordenação se houver múltiplos workers | TRANSCRICAO | [09:12] Diego, [09:13] Larissa, [09:25] Diego |
| FDD-RISCO-03 | [docs/FDD.md](FDD.md) | Pendência | Autorização por cliente e acesso aos dados de resposta | TRANSCRICAO | [09:32] Larissa, [09:34] Marcos, [09:37] Sofia |
| FDD-RISCO-04 | [docs/FDD.md](FDD.md) | Proposta de mitigação | Revisar segredos, HTTPS e validação de destinos antes do deploy | TRANSCRICAO | [09:21] Sofia, [09:23] Sofia, [09:46] Sofia |
| FDD-RISCO-05 | [docs/FDD.md](FDD.md) | Risco | Falha persistente requer DLQ e replay auditado. Destino pode continuar indisponível | TRANSCRICAO | [09:15] Diego, [09:18] Diego, [09:36] Sofia |
| FDD-DEP-01 | [docs/FDD.md](FDD.md) | Proposta de implantação | Aplicar migração antes de ativar captura e iniciar worker separado no MySQL existente | TRANSCRICAO | [09:08] Larissa, [09:11] Diego, [09:30] Bruno, [09:40] Bruno |

## ADRs

| ID | Documento | Tipo | Conteúdo (resumo) | Fonte | Localização |
| --- | --- | --- | --- | --- | --- |
| ADR-001 | [docs/adrs/ADR-001-outbox-transacional-mysql.md](adrs/ADR-001-outbox-transacional-mysql.md) | Decisão | Outbox MySQL na transação, filtro, UUID e snapshot | TRANSCRICAO | [09:06] Diego, [09:08] Larissa, [09:34] Bruno, [09:40] Bruno, [09:51] Larissa, [09:52] Larissa |
| ADR-001-CODIGO | [docs/adrs/ADR-001-outbox-transacional-mysql.md](adrs/ADR-001-outbox-transacional-mysql.md) | Referência de código | Base existente para ADR-001 | CODIGO | [src/modules/orders/order.service.ts](../src/modules/orders/order.service.ts#L126) |
| ADR-002 | [docs/adrs/ADR-002-worker-polling-outbox.md](adrs/ADR-002-worker-polling-outbox.md) | Decisão | Worker único separado, polling de 2 s e prioridade aos registros mais antigos | TRANSCRICAO | [09:09] Diego, [09:11] Diego, [09:12] Diego, [09:13] Larissa, [09:30] Bruno |
| ADR-002-CODIGO | [docs/adrs/ADR-002-worker-polling-outbox.md](adrs/ADR-002-worker-polling-outbox.md) | Referência de código | Base existente para ADR-002 | CODIGO | [src/server.ts](../src/server.ts) |
| ADR-003 | [docs/adrs/ADR-003-retentativas-com-backoff.md](adrs/ADR-003-retentativas-com-backoff.md) | Decisão com pendência | Backoff de 1 min, 5 min, 30 min, 2 h e 12 h. Contagem inicial em revisão | TRANSCRICAO | [09:15] Diego, [09:16] Bruno, [09:17] Larissa, [09:42] Diego, [09:48] Larissa |
| ADR-004 | [docs/adrs/ADR-004-dlq-e-replay-administrativo.md](adrs/ADR-004-dlq-e-replay-administrativo.md) | Decisão | DLQ separada e replay manual, auditado e exclusivo de ADMIN | TRANSCRICAO | [09:17] Larissa, [09:18] Diego, [09:36] Sofia, [09:36] Larissa |
| ADR-004-CODIGO | [docs/adrs/ADR-004-dlq-e-replay-administrativo.md](adrs/ADR-004-dlq-e-replay-administrativo.md) | Referência de código | Base existente para ADR-004 | CODIGO | [src/middlewares/auth.middleware.ts](../src/middlewares/auth.middleware.ts#L49) |
| ADR-005 | [docs/adrs/ADR-005-assinatura-hmac-por-endpoint.md](adrs/ADR-005-assinatura-hmac-por-endpoint.md) | Decisão | HMAC-SHA256 sobre o corpo, segredo por endpoint e rotação com 24 h | TRANSCRICAO | [09:20] Sofia, [09:21] Sofia, [09:22] Sofia, [09:23] Sofia |
| ADR-006 | [docs/adrs/ADR-006-entrega-at-least-once.md](adrs/ADR-006-entrega-at-least-once.md) | Decisão | At-least-once com UUID em X-Event-Id para deduplicação | TRANSCRICAO | [09:24] Diego, [09:25] Diego, [09:26] Larissa |
| ADR-007 | [docs/adrs/ADR-007-reuso-dos-padroes-do-projeto.md](adrs/ADR-007-reuso-dos-padroes-do-projeto.md) | Decisão | Reusar módulos, AppError, Zod, middleware de erros e Pino | TRANSCRICAO | [09:27] Bruno, [09:29] Bruno, [09:30] Larissa |
| ADR-007-CODIGO | [docs/adrs/ADR-007-reuso-dos-padroes-do-projeto.md](adrs/ADR-007-reuso-dos-padroes-do-projeto.md) | Referência de código | Base existente para ADR-007 | CODIGO | [src/shared/errors/app-error.ts](../src/shared/errors/app-error.ts), [src/middlewares/error.middleware.ts](../src/middlewares/error.middleware.ts), [src/shared/logger/index.ts](../src/shared/logger/index.ts) |
| ADR-008 | [docs/adrs/ADR-008-contrato-do-evento-de-status.md](adrs/ADR-008-contrato-do-evento-de-status.md) | Decisão | JSON compacto persistido na captura e headers de envio | TRANSCRICAO | [09:43] Diego, [09:44] Diego, [09:44] Sofia, [09:52] Larissa |
