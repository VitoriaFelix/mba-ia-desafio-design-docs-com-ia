# ADR-004: HMAC-SHA256 e secret por endpoint

- **Status:** Aceita na reunião
- **Decisora técnica de segurança:** Sofia

## Contexto

O webhook transporta dados de pedidos para sistemas fora da infraestrutura e o consumidor precisa verificar origem e integridade do corpo ([09:19]–[09:22]).

## Decisão

Assinar o corpo do request com HMAC-SHA256 e enviar a assinatura em `X-Signature`; usar uma secret única por endpoint e permitir rotação. A secret antiga permanece válida em paralelo por 24 horas. Exigir TLS/HTTPS e rejeitar URL HTTP. Limitar payload a 64 KB. `X-Timestamp` é um header separado para o consumidor avaliar replay; a decisão de assinar o corpo não autentica esse header.

## Alternativas Consideradas

- **Secret global para toda a plataforma:** descartada porque vazamento de uma secret afetaria todos os clientes/endpoints.
- **Truncar payload acima do limite:** descartado; payload acima de 64 KB deve falhar, não ser truncado.

## Consequências

**Positivas:** isolamento de credenciais por endpoint, verificação de integridade e rotação com janela de migração.

**Negativas:** implementação e suporte à rotação ficam mais complexos; secrets precisam ser protegidas em repouso e excluídas de logs; formato exato da assinatura ainda deve ser acordado.
