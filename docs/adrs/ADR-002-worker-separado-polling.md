# ADR-002: Worker separado com polling de 2 segundos

- **Status:** Aceita na reunião
- **Decisores:** Larissa, Bruno, Diego, Marcos

## Contexto

O envio não deve ocorrer dentro do processo síncrono de mudança de status. A reunião define um worker separado, no mesmo stack Node.js e banco, com polling a cada 2 segundos ([09:09]–[09:12]).

## Decisão

Criar entry point separado, proposto como `src/worker.ts`, e comando `npm run worker`. A lógica fica no módulo de webhooks. O worker cria seu próprio `PrismaClient` para o mesmo banco e processa pendências em ordem de criação, inicialmente com uma única instância.

## Alternativas Consideradas

- **Worker dentro da API:** descartado porque reinício/encerramento da API também afetaria o worker.
- **Trigger do MySQL para notificar processo externo:** descartado porque trigger não envia notificação nativa a um processo externo.
- **Broker externo:** descartado nesta fase por infra adicional.

## Consequências

**Positivas:** separação de ciclo de vida, simplicidade operacional e latência de polling compatível com objetivo de produto.

**Negativas:** consulta periódica ao banco; latência mínima determinada pelo intervalo; single-worker limita throughput e escala horizontal exige nova decisão sobre ordering/locks.
