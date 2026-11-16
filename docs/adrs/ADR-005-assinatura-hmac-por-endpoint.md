# ADR-005 — Assinatura HMAC-SHA256 por endpoint

## Status

Aceito na reunião técnica.

## Contexto

O destinatário precisa verificar a origem da notificação e a integridade do corpo enviado. Uma credencial comprometida deve ter alcance limitado.

## Decisão

Assinar o corpo HTTP com HMAC-SHA256 e enviar o resultado em `X-Signature`. Usar um segredo por endpoint e exigir HTTPS. A rotação solicitada via API mantém o segredo anterior válido por 24 horas em paralelo com o novo. O mecanismo de verificação durante a sobreposição deve ser especificado no contrato.

## Alternativas consideradas

- **Um segredo global.** Simplificaria o armazenamento, mas um vazamento comprometeria a verificação de todos os endpoints e exigiria uma rotação coordenada.

## Consequências

O destinatário consegue verificar a assinatura e a rotação permite uma migração gradual. O serviço passa a gerenciar segredos por endpoint e a sobreposição prolonga por até 24 horas a validade da credencial anterior.

## Fontes

- [Transcrição da reunião técnica](../../TRANSCRICAO.md), 09:19–09:23 e 09:48.
