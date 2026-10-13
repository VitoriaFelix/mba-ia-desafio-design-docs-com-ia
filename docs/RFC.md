# RFC — Sistema de Webhooks de Notificação de Pedidos

- **Autor:** equipe de engenharia (documento preparado a partir da reunião)
- **Status:** Proposta para revisão da equipe
- **Data:** não informada na transcrição; reunião identificada como quinta-feira, 09:00
- **Revisores:** Larissa (Tech Lead), Marcos (Product Manager), Bruno (Engenharia de Pedidos), Diego (Plataforma), Sofia (Segurança)

## TL;DR

Propomos webhooks outbound para mudanças de status de pedidos, usando outbox no MySQL existente, escrita atomicamente com a transação do pedido e processada por worker Node.js separado em polling de 2 segundos. O worker entrega JSON assinado por HMAC-SHA256, com secret por endpoint, timeout de 10 segundos e cinco tentativas com backoff; falhas finais são movidas para DLQ e podem ser reprocessadas por ADMIN. A entrega é at-least-once e usa `X-Event-Id` para deduplicação do consumidor.

## Contexto e problema

Três clientes B2B precisam de notificações em menos de 10 segundos e hoje fazem polling de `GET /orders`, gerando custo e latência ([09:00]–[09:02] Marcos). A transação de mudança de status já atualiza pedido, histórico e estoque; uma chamada HTTP síncrona faria o pedido depender da disponibilidade dos clientes ([09:04]–[09:06] Bruno/Diego).

## Proposta técnica

1. **Captura transacional:** ampliar `OrderService.changeStatus` para inserir um snapshot na tabela de outbox dentro do mesmo `$transaction` que atualiza pedido e histórico. O evento só é inserido se houver endpoint ativo inscrito no status de destino.
2. **Persistência:** MySQL existente, UUID como identificador, índices por estado e data de criação. Payload é renderizado na inserção para preservar o estado do pedido naquele instante.
3. **Processamento:** processo Node.js separado (`src/worker.ts`, novo), com novo `PrismaClient` no mesmo banco. Polling a cada 2 segundos; um worker no estágio inicial.
4. **Entrega segura:** HTTPS obrigatório; HMAC-SHA256 sobre o corpo; secret por endpoint; rotação com 24 horas de coexistência; headers `X-Event-Id`, `X-Signature`, `X-Timestamp`, `X-Webhook-Id` e `Content-Type`.
5. **Resiliência:** timeout de 10 segundos, política discutida de cinco tentativas nos intervalos definidos e DLQ em tabela separada; replay manual em endpoint admin com role ADMIN e log de auditoria. A contagem pode significar cinco chamadas totais ou cinco retries após a inicial, ponto ainda aberto.
6. **Integração com a aplicação:** seguir módulos `src/modules`, schemas Zod, `AppError`, códigos `WEBHOOK_*`, middleware centralizado e Pino. A implementação detalhada fica no [FDD](FDD.md).

## Alternativas consideradas

### Chamada HTTP síncrona em `changeStatus` — descartada

- **Vantagem:** menor complexidade inicial e tentativa imediata.
- **Trade-off que levou ao descarte:** aumenta duração da transação e acopla a alteração de pedido à latência/disponibilidade de cada cliente; não é aceitável que cliente fora do ar provoque rollback de pedido.
- **Fonte:** [09:03]–[09:06] Bruno, Larissa e Diego.

### Redis Streams ou broker externo — descartado nesta fase

- **Trade-off:** adicionaria infraestrutura; o time é pequeno e a outbox no MySQL existente resolve o problema atual sem cluster Redis.
- **Fonte:** [09:06]–[09:08] Diego.

### Trigger do banco para sinalizar o worker — descartada

- **Vantagem:** poderia parecer mais reativa.
- **Trade-off:** trigger MySQL não notifica diretamente um processo externo; polling de 2 segundos satisfaz o objetivo de menos de 10 segundos com menos complexidade.
- **Fonte:** [09:09]–[09:10] Diego e Marcos.

### Retry indefinido ou apenas três tentativas — descartados

- **Retry indefinido:** pode manter eventos pendurados para sempre.
- **Três tentativas:** não cobriria indisponibilidades temporárias de horas.
- **Decisão adotada:** cinco tentativas e DLQ.
- **Fonte:** [09:15]–[09:18] Diego, Bruno e Larissa.

### Exactly-once — descartado

- **Trade-off:** a garantia exactly-once exigiria coordenação entre emissor e receptor e aumentaria muito a complexidade. A reunião rejeitou essa garantia e escolheu at-least-once com UUID para deduplicação do consumidor.
- **Fonte:** [09:24]–[09:26] Diego e Sofia.

## Questões em aberto

1. **Rate limiting por cliente:** qual limite de rajada/vazão deve ser aplicado, e em que condição deve ser implementado? A reunião decidiu apenas observar ([09:38]–[09:39]).
2. **Escala horizontal:** quando for necessário mais de um worker, a equipe deve escolher particionamento por `order_id` ou lock pessimista; por ora single-worker ([09:12]–[09:13]).
3. **Retenção e arquivamento:** linhas entregues podem ser arquivadas após cerca de 30 dias, mas isso foi deixado fora do escopo atual ([09:07]–[09:08]).
4. **Detalhes de API não definidos:** paginação, ordenação e filtros para `GET /webhooks/:id/deliveries`, semântica de desativação/exclusão de configuração e política de autorização por `customer_id` precisam ser fechadas na revisão do FDD. A reunião pediu os endpoints e campos, mas não fechou todos esses detalhes. A origem de `customer_id` foi esclarecida: body ou path, não JWT; o escopo que cada usuário pode administrar permanece aberto.
5. **Observabilidade além dos logs:** tracing e correlação não foram discutidos na reunião; definir se são necessários e, em caso afirmativo, escolher formato e instrumentação.
6. **Segurança da assinatura e replay:** a reunião definiu HMAC-SHA256 sobre o corpo; `X-Timestamp` é um header separado para o consumidor avaliar replay e não está autenticado pela assinatura definida. Se autenticar o timestamp for requisito, isso exige ampliar o contrato e uma decisão de segurança. Também falta decidir se replay preserva `event_id` ou cria novo identificador.

7. **Semântica de retry:** a reunião fixou cinco tentativas e listou cinco intervalos, mas não esclareceu se a chamada inicial entra na contagem nem qual intervalo se aplica a cada chamada; definir antes da implementação ([09:15]–[09:18]).

## Impacto e riscos

- **Pedidos:** a transação passa a incluir uma inserção adicional; não inclui chamada HTTP.
- **Banco:** novas tabelas de configuração, outbox, deliveries e DLQ; crescimento de armazenamento deve ser observado.
- **Operação:** novo processo worker e endpoint administrativo de replay.
- **Segurança:** gerenciamento de secrets e assinatura; revisão da Sofia obrigatória antes do deploy.
- **Clientes:** devem validar HMAC e deduplicar por `X-Event-Id`.
- **Trade-off:** a simplicidade do polling e do single-worker reduz a infraestrutura inicial, mas limita escala e ordenação em cenários futuros.

## Decisões relacionadas

- [ADR-001 — Outbox no MySQL e transação atômica](adrs/ADR-001-outbox-no-mysql.md)
- [ADR-002 — Worker separado com polling](adrs/ADR-002-worker-separado-polling.md)
- [ADR-003 — Retry exponencial e DLQ](adrs/ADR-003-retry-backoff-dlq.md)
- [ADR-004 — HMAC-SHA256 e secret por endpoint](adrs/ADR-004-hmac-secret-por-endpoint.md)
- [ADR-005 — At-least-once e X-Event-Id](adrs/ADR-005-at-least-once-event-id.md)
- [ADR-006 — Reuso dos padrões existentes](adrs/ADR-006-reuso-padroes-existentes.md)
- [ADR-007 — Snapshot do evento e UUID](adrs/ADR-007-snapshot-e-uuid.md)

## Revisão solicitada

Revisores: confirmar a arquitetura proposta, a política de autorização por customer, a semântica de deliveries e a prioridade das questões em aberto. A proposta não autoriza iniciar implementação dos itens explicitamente adiados sem nova decisão.
