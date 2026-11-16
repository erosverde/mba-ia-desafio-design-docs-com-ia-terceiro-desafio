# Design Docs gerados por IA — Webhooks de pedidos

## Sobre o desafio

No terceiro desafio do MBA Engenharia com IA, transformei a transcrição de uma reunião técnica e o código de um sistema de pedidos em um pacote de documentação para uma nova funcionalidade. A proposta é notificar clientes B2B por webhook quando seus pedidos mudarem de status, com configuração por endpoint, histórico de entregas e replay administrativo.

A entrega é documental. A [transcrição](TRANSCRICAO.md) fundamenta os requisitos e as decisões. O código existente mostra onde a proposta se encaixa, especialmente na transação de [`OrderService.changeStatus`](src/modules/orders/order.service.ts#L126). O [Tracker](docs/TRACKER.md) reúne essas referências.

## Ferramentas de IA utilizadas

- **ChatGPT** · Usei a conversa inicial para analisar o enunciado, a transcrição e a estrutura do projeto, organizar a sequência de trabalho e preparar a primeira versão do ADR de outbox.
- **Codex no aplicativo desktop** · Usei o acesso ao repositório local para conferir os arquivos, retomar o contexto, redigir os documentos em Markdown e validar as referências. Git e PowerShell ajudaram a verificar as alterações, os links, os exemplos JSON e a cobertura dos IDs no tracker.

## Workflow adotado

1. **Ler as fontes.** Identifiquei com a IA os requisitos da reunião e os pontos de integração no código. Dei prioridade à transação de mudança de status para confirmar onde a notificação seria registrada.
2. **Registrar as decisões.** Comecei pela outbox no MySQL, segui com o worker separado e avancei até os oito ADRs. Cada registro inclui contexto, alternativas, consequências e fontes.
3. **Consolidar o desenho.** Solicitei o RFC para reunir as decisões e as questões de revisão. Em seguida, o FDD detalhou os fluxos, o modelo de dados e os contratos. O PRD consolidou o problema dos clientes, o escopo e os critérios de aceite.
4. **Verificar a rastreabilidade.** Pedi o Tracker depois que os documentos tinham IDs estáveis. O Codex verificou a presença dos IDs, a existência dos arquivos citados e os pares de timestamp e falante na transcrição.

As solicitações foram feitas por etapa. A cada documento, o Codex apresentou o arquivo produzido e as pendências para orientar a etapa seguinte.

## Prompts customizados

Prompt usado para iniciar a gravação no repositório local.

```text
Continue o desafio Design Docs do MBA. Confira o estado do repositório e grave o ADR-001 sobre outbox transacionaaal em MySQL em docs/adrs/0001-outbox-transacional-mysql.md, usando TRANSCRICAO.md e OrderService.changeStatus como fontes. Faça apenas alterações na documentação.
```

Prompt usado para consolidar a rastreabilidade.

```text
Vamos ao Tracker. Preencha o docs/TRACKER.md ligando os requisitos e decisões do PRD, RFC, FDD e ADRs aos timestamps da transcrição ou aos trechos do código.
```

## Iterações e ajustes

Houve quatro iterações principais até o pacote documental atual.

1. **Do rascunho à gravação local.** A primeira conversa conseguiu elaborar o ADR-001, mas o rascunho ainda precisava ser salvo. Retomei a tarefa no Codex com a pasta do repositório aberta e indiquei o caminho do arquivo e as duas fontes que deveriam fundamentá-lo.
2. **Da decisão geral às questões de implementação.** Ao detalhar as retentativas, identifiquei com a IA que a reunião registrava cinco tentativas e cinco intervalos. Usar todos os intervalos depois do envio inicial implica seis envios. O ADR-003 e o RFC registram a pendência. O FDD também marca como propostas os detalhes de contrato ainda sujeitos a revisão, como o formato das assinaturas durante a rotação.
3. **Do acompanhamento no chat à evidência no repositório.** Quando pedi o status do desafio, o Codex conferiu os arquivos e recuperou o plano anterior. O Tracker ainda estava vazio. Depois da redação dos documentos, o preenchimento e a verificação do Tracker deixaram 70 itens rastreados na primeira versão, incluindo os 40 IDs explícitos do PRD, RFC e FDD e os oito ADRs.

4. **Revisão contra o enunciado.** O Codex encontrou sete requisitos funcionais no PRD, embora o enunciado exigisse oito, e exemplos de contrato insuficientes no FDD. Separei cadastro de gerenciamento, completei os exemplos e alinhei o nome dos ADRs ao padrão `ADR-NNN`. Os prompts acima preservam a redação original, incluindo o caminho anterior do primeiro ADR.

## Como navegar a entrega

1. [PRD](docs/PRD.md) — problema, público, escopo, requisitos e critérios de aceite.
2. [RFC](docs/RFC.md) — proposta técnica, alternativas e questões em aberto.
3. [ADRs](docs/adrs/) — decisões 001 a 008, da captura transacional ao contrato do evento.
4. [FDD](docs/FDD.md) — fluxos, modelo de dados, contratos e integração com o sistema existente.
5. [Tracker](docs/TRACKER.md) — referências cruzadas para a [transcrição](TRANSCRICAO.md) e o código.

O enunciado original está no histórico do Git, na revisão `f4354f9ad4543b7a36fa0290c127c2137fe1b374`, no arquivo `README.md`.
