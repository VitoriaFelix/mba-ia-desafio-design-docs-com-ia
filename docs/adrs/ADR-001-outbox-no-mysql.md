# ADR-001: Outbox no MySQL e transação atômica

- **Status:** Aceita na reunião
- **Data:** não informada
- **Decisores:** Larissa, Bruno, Diego

## Contexto

`changeStatus` altera pedido, histórico e estoque numa transação. Uma chamada HTTP síncrona poderia bloquear a transação e tornar a mudança dependente do sistema do cliente. Fonte: [09:03]–[09:08].

## Decisão

Usar padrão Outbox no MySQL existente. A inserção do evento ocorre na mesma transação Prisma que altera o status e cria histórico. Falha na outbox provoca rollback. A tabela deve ter índices para status e `created_at`; o worker lê pendências em lotes pequenos.

## Alternativas Consideradas

- **HTTP síncrono:** descartado porque clientes lentos/indisponíveis afetariam a operação de pedido.
- **Redis Streams ou broker externo:** descartado nesta fase por exigir infraestrutura adicional para um time pequeno.

## Consequências

**Positivas:** atomicidade entre estado do pedido e evento; sem broker adicional; falhas de cliente desacopladas da API.

**Negativas:** novas tabelas e escrita adicional na transação; crescimento da outbox exige observação e política futura de retenção; consultas do worker compartilham o banco com a aplicação.
