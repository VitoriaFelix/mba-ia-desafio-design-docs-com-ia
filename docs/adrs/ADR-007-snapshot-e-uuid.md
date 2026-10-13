# ADR-007: UUID e snapshot renderizado na inserção

- **Status:** Aceita na reunião
- **Decisores:** Larissa, Diego, Bruno

## Contexto

A equipe confirmou que os IDs seguem UUID e que o payload deve refletir o estado do pedido no momento em que o status muda, mesmo que o pedido seja alterado depois ([09:51]–[09:52]).

## Decisão

Gerar UUID para o evento e persistir o payload JSON renderizado no momento da inserção da outbox. O worker envia esse snapshot, sem buscar o estado atual do pedido para reconstruir o evento.

## Alternativas Consideradas

- **ID auto-incremental:** descartado em favor do padrão UUID usado no restante da aplicação.
- **Persistir apenas `order_id` e montar payload no envio:** descartado porque uma alteração posterior poderia mudar o conteúdo histórico do evento.

## Consequências

**Positivas:** identidade de evento estável, consistência temporal e payload reproduzível em retry/replay.

**Negativas:** a outbox armazena mais dados; payload deve respeitar teto de 64 KB; mudanças futuras no formato do evento não devem reescrever snapshots já persistidos.
