# ADR-006 — Entrega at-least-once com X-Event-Id

## Status

Aceito na reunião técnica.

## Contexto

O destinatário pode processar a notificação antes que o worker receba a resposta. Uma nova tentativa após timeout pode repetir o efeito.

## Decisão

Adotar semântica at-least-once e enviar em `X-Event-Id` o UUID atribuído ao evento na inserção na outbox. Reutilizar esse identificador nas retentativas do mesmo evento. O destinatário usa o ID para deduplicar o processamento. A política de retentativas é finita e falhas terminais seguem para a DLQ.

## Alternativas consideradas

- **Exactly-once.** Exigiria coordenar a confirmação do envio com a persistência do efeito no sistema do destinatário, aumentando o acoplamento entre as partes.

## Consequências

O worker pode repetir envios com resultado incerto. O destinatário precisa persistir ou reconhecer IDs já processados. Um destino que continue indisponível pode exigir replay.

## Fontes

- [Transcrição da reunião técnica](../../TRANSCRICAO.md), 09:24–09:26 e 09:48.
