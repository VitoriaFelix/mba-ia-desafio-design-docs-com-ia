# ADR-003: Retry com backoff e dead-letter queue

- **Status:** Aceita na reunião
- **Decisores:** Larissa, Bruno, Diego

## Contexto

Endpoints de clientes podem ficar indisponíveis. Retry infinito pode manter eventos pendurados para sempre, enquanto três tentativas podem não cobrir uma manutenção de algumas horas ([09:15]–[09:18]).

## Decisão

Aplicar a política de cinco tentativas com os intervalos definidos na reunião: 1 minuto, 5 minutos, 30 minutos, 2 horas e 12 horas. Após esgotar a política, persistir o evento em tabela separada `webhook_dead_letter`, incluindo payload, motivo e timestamp. Reprocessamento é manual por endpoint admin.

**Ponto pendente:** a interpretação exata de “cinco tentativas” (cinco chamadas totais ou a chamada inicial mais cinco retries) deve ser decidida antes de codar. A reunião listou cinco intervalos, mas não esclareceu a contagem das chamadas.

## Alternativas Consideradas

- **Retry infinito:** descartado por risco de manter eventos pendentes indefinidamente.
- **Três tentativas:** descartado por não cobrir indisponibilidades temporárias mais longas.
- **Marcar somente `failed` na outbox:** a reunião preferiu tabela separada para manter a outbox principal limpa e preservar evidência para debug/replay.

## Consequências

**Positivas:** falhas temporárias têm oportunidade de recuperação; falhas permanentes ficam identificadas e podem ser reprocessadas.

**Negativas:** DLQ exige operação e monitoramento; replay pode gerar duplicatas; armazenamento e política de retenção precisam ser definidos.
