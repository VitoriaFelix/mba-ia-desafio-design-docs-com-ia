# PRD — Sistema de Webhooks de Notificação de Pedidos

- **Status:** Proposta para revisão
- **Data da reunião de origem:** quinta-feira, data de calendário não informada na transcrição
- **Stakeholders:** Larissa (Tech Lead), Marcos (Product Manager), Bruno (Engenharia de Pedidos), Diego (Plataforma), Sofia (Segurança)

## 1. Resumo e contexto

Clientes B2B precisam receber notificações outbound quando o status dos pedidos muda. Hoje consultam repetidamente `GET /orders`, aumentando custo e latência da integração. A proposta adiciona configuração de endpoints de webhook, entrega assíncrona, assinatura criptográfica e histórico de entregas, sem acoplar a transação de pedido à disponibilidade dos sistemas dos clientes.

## 2. Problema e motivação

Três clientes — Atlas Comercial, MaxDistribuição e Nova Cargo — solicitaram notificações próximas de tempo real. Para esses clientes, menos de 10 segundos é considerado “tempo real”. A Atlas sinalizou risco comercial caso a solução não seja entregue até o fim do trimestre. Fonte principal: transcrição [09:00]–[09:02] Marcos.

## 3. Público-alvo e cenários de uso

- **Clientes B2B integradores:** cadastram endpoint e escolhem quais status desejam receber.
- **Sistemas dos clientes:** validam assinatura, deduplicam por `X-Event-Id` e buscam detalhes do pedido pela API existente quando necessário.
- **Operadores autenticados:** gerenciam configurações e consultam entregas conforme as permissões que a API atual permitir.
- **Administradores:** reprocessam manualmente eventos na dead-letter queue (DLQ), com auditoria de quem executou a ação.

## 4. Objetivos e métricas de sucesso

1. **Latência de notificação:** objetivo de entrega inferior a 10 segundos em condições normais. Polling de 2 segundos foi aceito na reunião; medir p50/p95/p99 desde o commit da transação até a primeira tentativa de envio.
2. **Consistência:** 100% das mudanças de status commitadas que correspondam a pelo menos um filtro ativo devem ter o evento registrado na outbox na mesma transação. Se a inserção falhar, a transação inteira deve sofrer rollback.
3. **Segurança de entrega:** cada requisição deve ser assinada com HMAC-SHA256 usando secret específica do endpoint; URL HTTP deve ser recusada.
4. **Operabilidade:** permitir consultar histórico de entregas e reprocessar manualmente itens na DLQ.
5. **Resiliência:** esgotar a política de cinco tentativas e os intervalos listados antes de mover o evento para DLQ; a contagem exata das chamadas ainda precisa ser validada.

As metas de latência são objetivos observáveis e dependem da disponibilidade do destino e do volume acumulado; a reunião não definiu um SLO de disponibilidade nem uma taxa máxima de falhas.

## 5. Escopo

### Incluído

- Webhooks **outbound** para mudanças de status do pedido.
- CRUD autenticado de configurações: criar, editar, excluir e listar.
- Filtro por lista de status por endpoint, aplicado antes de inserir na outbox.
- Outbox no MySQL existente, escrita na mesma transação que altera status/histórico/estoque.
- Worker Node.js separado, com polling a cada 2 segundos.
- Payload JSON enxuto e snapshot criado no momento da transição.
- Assinatura HMAC-SHA256, secret por endpoint, TLS obrigatório e rotação com período de coexistência de 24 horas.
- Semântica at-least-once com `X-Event-Id`.
- Timeout HTTP de 10 segundos; política de cinco tentativas e backoff definido, com a interpretação da contagem ainda pendente; DLQ persistida e replay manual por ADMIN.
- Histórico de entregas para consulta pelos clientes.
- Prefixo de erros `WEBHOOK_`, padrões atuais de `AppError`, Pino, Zod, middleware centralizado e módulos.
- Revisão de segurança antes do deploy, reservando pelo menos dois dias úteis.

### Fora de escopo

1. **Envio de e-mail ao cliente** após falhas repetidas: adiado para fase futura ([09:37]–[09:38] Larissa/Marcos).
2. **Dashboard visual** para clientes: não será feito nesta fase; serão fornecidos endpoints ([09:39]–[09:40] Larissa/Marcos).
3. **Rate limiting de saída por cliente:** não foi decidido; observar e decidir posteriormente ([09:38]–[09:39] Diego/Larissa).
4. **Arquivamento/limpeza de linhas entregues após cerca de 30 dias:** explicitamente fora desta feature ([09:07]–[09:08] Diego).
5. **Garantia de ordenação global ou escala horizontal com múltiplos workers:** não faz parte da garantia inicial; single-worker é a decisão atual ([09:12]–[09:13] Diego/Larissa).
6. **Exactly-once:** não será garantido; será usada entrega at-least-once com deduplicação pelo consumidor ([09:24]–[09:26] Diego/Sofia).

## 6. Requisitos funcionais

- **PRD-FR-01 — Cadastro:** usuário autenticado pode cadastrar webhook com URL HTTPS e lista de status de interesse; `customer_id` é informado no body ou path, não extraído do JWT; a secret é gerada pela plataforma e devolvida na criação. O escopo de autorização do usuário sobre esse customer permanece pendente.
- **PRD-FR-02 — Edição:** usuário autenticado pode editar a configuração do webhook.
- **PRD-FR-03 — Remoção:** usuário autenticado pode excluir uma configuração.
- **PRD-FR-04 — Listagem:** usuário autenticado pode listar webhooks de um `customer_id` informado no recurso; a autorização para acessar outro customer ainda precisa ser definida.
- **PRD-FR-05 — Filtro:** somente criar registros na outbox quando existir endpoint ativo daquele customer inscrito no status de destino.
- **PRD-FR-06 — Atomicidade:** persistir snapshot do evento na mesma transação de `changeStatus`; falha de inserção provoca rollback.
- **PRD-FR-07 — Entrega assíncrona:** worker separado busca pendências em polling de 2 segundos e faz requisição HTTP.
- **PRD-FR-08 — Assinatura:** incluir `X-Signature` calculado por HMAC-SHA256 sobre o corpo do request, além dos headers definidos no FDD.
- **PRD-FR-09 — Rotação:** permitir solicitar nova secret; a antiga permanece válida por 24 horas e depois expira.
- **PRD-FR-10 — Tentativas:** aplicar a política discutida de cinco tentativas e intervalos de 1 min, 5 min, 30 min, 2 h e 12 h. A reunião não esclarece se são cinco chamadas totais ou cinco retries após a chamada inicial; essa contagem precisa ser validada antes da implementação.
- **PRD-FR-11 — DLQ:** após esgotar as tentativas, persistir evento, motivo e timestamp em tabela separada.
- **PRD-FR-12 — Replay:** ADMIN pode reprocessar manualmente um item da DLQ; registrar quem executou.
- **PRD-FR-13 — Histórico:** expor consulta de entregas com status de sucesso/falha, payload, resposta e tempo de resposta; a reunião cita os últimos 100 registros como necessidade inicial.
- **PRD-FR-14 — Segurança do destino:** recusar URL não HTTPS e payload acima de 64 KB.
- **PRD-FR-15 — Identificação e deduplicação:** incluir UUID de evento em `X-Event-Id`; consumidores devem deduplicar, pois podem ocorrer duplicatas.
- **PRD-FR-16 — Contrato de evento:** payload JSON inclui `event_id`, `event_type`, `timestamp` ISO 8601, `order_id`, `order_number`, `from_status`, `to_status`, `customer_id` e `total_cents`; não incluir `items`.

## 7. Requisitos não funcionais

- **PRD-NFR-01 — Latência:** objetivo inferior a 10 segundos em condições normais; polling de 2 segundos.
- **PRD-NFR-02 — Isolamento:** chamadas HTTP nunca devem ocorrer dentro da transação do pedido.
- **PRD-NFR-03 — Integridade:** evento e mudança de status são atômicos.
- **PRD-NFR-04 — Segurança:** HTTPS obrigatório, HMAC-SHA256, secret exclusiva por endpoint, rotação de 24 h, limite de 64 KB e timeout de 10 s.
- **PRD-NFR-05 — Resiliência:** retry limitado e DLQ auditável.
- **PRD-NFR-06 — Observabilidade proposta:** acompanhar latência e falhas e usar logs estruturados com Pino. Tracing/identificador de correlação é sugestão para revisão, não decisão da reunião; a instrumentação não foi definida.
- **PRD-NFR-07 — Compatibilidade:** seguir stack e padrões já usados pela aplicação; não adicionar broker/infra adicional nesta fase.

## 8. Decisões e trade-offs principais

- Outbox MySQL na transação atual: consistência forte sem infraestrutura adicional; aumenta escrita e exige manutenção da tabela.
- Worker em processo separado com polling de 2 s: simples e resiliente a reinício da API; latência mínima de polling e consulta periódica ao banco.
- At-least-once + `X-Event-Id`: implementação mais simples que exactly-once; consumidor precisa deduplicar.
- HMAC-SHA256 por endpoint: autentica integridade/origem da chamada; exige armazenamento seguro e rotação de secrets.
- Política de cinco tentativas (contagem ainda por validar) + DLQ: tolera indisponibilidades temporárias, mas falhas longas exigem operação de replay.
- Single-worker: ordenação por `created_at` é simples no estágio inicial; limita paralelismo e não fornece garantia global absoluta.

## 9. Dependências

- MySQL e Prisma já usados pela aplicação.
- `OrderService.changeStatus` e `OrderStatusHistory`.
- Autenticação JWT existente e role `ADMIN` para replay.
- Padrões compartilhados: `AppError`, erros HTTP, Zod, middleware centralizado e Pino.
- Clientes precisam validar HMAC e deduplicar eventos.

## 10. Riscos e mitigação

A probabilidade e o impacto abaixo são uma avaliação qualitativa inicial para revisão; não foram estimados nem aprovados na reunião.

| Risco | Probabilidade | Impacto | Mitigação |
|---|---|---|---|
| Endpoint de cliente indisponível por período prolongado | Alta | Médio/alto | Timeout de 10 s, cinco tentativas com backoff e DLQ com replay manual |
| Secret vazada ou assinatura não validada | Média | Alto | Secret por endpoint, HTTPS, rotação com grace period de 24 h, revisão de segurança |
| Crescimento da outbox por acúmulo ou ausência de limpeza | Média | Médio | Índices por status e `created_at`, batch pequeno, métricas; retenção/arquivamento fica para decisão futura |
| Duplicatas no consumidor | Média | Médio | At-least-once explicitado, UUID `X-Event-Id`, documentação de deduplicação |
| Latência acima de 10 s sob carga | Média | Alto | Medir p95/p99, observar tamanho da fila; política de rate limiting permanece em aberto |
| Regressão no fluxo transacional de pedidos | Baixa/média | Alto | Inserção da outbox na mesma transação, testes de rollback e ponta a ponta |

## 11. Critérios de aceitação

- A mudança de status e o evento são commitados juntos ou sofrem rollback juntos.
- Nenhuma chamada HTTP ao cliente é feita pelo caminho síncrono de `changeStatus`.
- Um worker independente inicia e consome pendências a cada 2 segundos.
- Eventos fora dos filtros não são inseridos na outbox.
- Payload respeita o contrato, snapshot e teto de 64 KB.
- Request contém `X-Event-Id`, `X-Signature`, `X-Timestamp`, `X-Webhook-Id` e `Content-Type: application/json`.
- URL `http://` é rejeitada; timeout é 10 segundos.
- Após a contagem de tentativas ser definida, a sequência aprovada é aplicada e a falha final envia o item à DLQ.
- Apenas ADMIN consegue executar replay e a ação é auditada.
- Histórico de entregas apresenta sucesso/falha, payload, response e tempo de resposta.
- Secrets rotacionadas convivem por 24 horas, sem exposição em logs.
- Revisão de segurança de Sofia concluída antes do deploy.

## 12. Estratégia de testes e validação

- Unitários: seleção de eventos, construção de payload, assinatura HMAC, rotação de secret, backoff e transições de estado do evento.
- Integração: inserção de outbox na transação Prisma; confirmar rollback da mudança de status quando a inserção da outbox falhar.
- HTTP: validação de URL HTTPS, limite de 64 KB, timeout de 10 s, headers e status codes.
- Resiliência: falhas temporárias, aplicação da contagem de tentativas validada pela equipe, movimento para DLQ e replay.
- Autorização: CRUD autenticado e replay restrito a ADMIN; auditar ator.
- Ponta a ponta: transição de status → outbox → worker → endpoint receptor simulado → histórico de entrega.
- Carga: medir latência de notificação e acúmulo de pendências; usar resultados para decidir sobre rate limiting.
