# ADR-005: At-least-once com X-Event-Id

- **Status:** Aceita na reunião
- **Decisores:** Larissa, Diego, Sofia

## Contexto

Uma chamada pode ser processada pelo destinatário mesmo que a resposta se perca, fazendo o emissor tentar novamente. Garantia exactly-once exigiria coordenação dos dois lados ([09:24]–[09:26]).

## Decisão

Adotar entrega at-least-once. Cada evento recebe UUID único quando entra na outbox; o emissor envia o ID no header `X-Event-Id`. O consumidor é responsável por deduplicar por esse identificador. O payload também contém `event_id`.

## Alternativas Consideradas

- **Exactly-once:** descartado por complexidade e necessidade de coordenação entre sistemas.

## Consequências

**Positivas:** semântica de entrega simples e recuperável; identificador estável facilita deduplicação e suporte.

**Negativas:** consumidores precisam implementar deduplicação; duplicatas são esperadas; a reunião não definiu se o replay preserva o ID ou gera um novo; essa semântica precisa ser decidida.
