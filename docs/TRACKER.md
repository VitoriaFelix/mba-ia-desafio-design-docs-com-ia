# Tracker de rastreabilidade

A localização de transcrição usa timestamp e falante como aparecem em `TRANSCRICAO.md`. Caminhos de código foram conferidos neste fork local. Paths listados como “proposto” não são tratados como código já existente.

| ID | Documento | Tipo | Conteúdo (resumo) | Fonte | Localização |
|---|---|---|---|---|---|
| PRD-CONT-01 | docs/PRD.md | Contexto | Clientes B2B precisam de notificações em tempo quase real | TRANSCRICAO | [09:00] Marcos |
| PRD-FR-01 | docs/PRD.md | Requisito funcional | Criar webhook com URL, secret gerada e status inscritos | TRANSCRICAO | [09:31] Marcos |
| PRD-FR-02 | docs/PRD.md | Requisito funcional | Editar configuração via PATCH | TRANSCRICAO | [09:33] Bruno |
| PRD-FR-03 | docs/PRD.md | Requisito funcional | Excluir configuração via DELETE | TRANSCRICAO | [09:33] Bruno |
| PRD-FR-04 | docs/PRD.md | Requisito funcional | Listar webhooks de um customer | TRANSCRICAO | [09:33] Bruno |
| PRD-FR-05 | docs/PRD.md | Requisito funcional | Filtrar eventos na inserção da outbox | TRANSCRICAO | [09:33]–[09:34] Marcos/Bruno |
| PRD-FR-06 | docs/PRD.md | Requisito funcional | Outbox e mudança de status na mesma transação | TRANSCRICAO | [09:40]–[09:42] Bruno/Diego |
| PRD-FR-07 | docs/PRD.md | Requisito funcional | Worker separado com polling a cada 2 s | TRANSCRICAO | [09:09]–[09:11] Diego/Larissa |
| PRD-FR-08 | docs/PRD.md | Requisito funcional | Assinatura HMAC-SHA256 e header X-Signature | TRANSCRICAO | [09:19]–[09:22] Sofia |
| PRD-FR-09 | docs/PRD.md | Requisito funcional | Secret por endpoint e rotação com 24 h | TRANSCRICAO | [09:21]–[09:22] Sofia |
| PRD-FR-10 | docs/PRD.md | Requisito funcional | Política de cinco tentativas e intervalos; contagem de chamadas e mapeamento ainda pendentes | TRANSCRICAO | [09:15]–[09:18] Diego/Larissa |
| PRD-FR-11 | docs/PRD.md | Requisito funcional | DLQ separada com payload, motivo e timestamp | TRANSCRICAO | [09:18] Diego |
| PRD-FR-12 | docs/PRD.md | Requisito funcional | Replay manual de DLQ via ADMIN | TRANSCRICAO | [09:18]–[09:19] Diego/Larissa |
| PRD-FR-13 | docs/PRD.md | Requisito funcional | Histórico de deliveries com payload, resposta e duração | TRANSCRICAO | [09:34] Marcos |
| PRD-FR-14 | docs/PRD.md | Requisito de segurança | HTTPS obrigatório e payload máximo de 64 KB | TRANSCRICAO | [09:23]–[09:24] Sofia/Larissa |
| PRD-FR-15 | docs/PRD.md | Requisito funcional | At-least-once com X-Event-Id | TRANSCRICAO | [09:24]–[09:26] Diego |
| PRD-FR-16 | docs/PRD.md | Contrato | Payload enxuto, sem items | TRANSCRICAO | [09:42]–[09:44] Diego/Bruno |
| PRD-NFR-01 | docs/PRD.md | Requisito não funcional | Objetivo de latência inferior a 10 s | TRANSCRICAO | [09:02] Marcos |
| PRD-NFR-02 | docs/PRD.md | Restrição | Não fazer HTTP síncrono na transação | TRANSCRICAO | [09:04]–[09:06] Bruno/Diego |
| PRD-OOS-01 | docs/PRD.md | Fora de escopo | E-mail de alerta foi adiado | TRANSCRICAO | [09:37]–[09:38] Larissa/Marcos |
| PRD-OOS-02 | docs/PRD.md | Fora de escopo | Dashboard visual não entra nesta fase | TRANSCRICAO | [09:39]–[09:40] Larissa/Marcos |
| PRD-OOS-03 | docs/PRD.md | Questão aberta | Rate limiting deve ser observado e decidido depois | TRANSCRICAO | [09:38]–[09:39] Diego/Larissa |
| PRD-OOS-04 | docs/PRD.md | Fora de escopo | Arquivamento em ~30 dias fora da feature | TRANSCRICAO | [09:07]–[09:08] Diego |
| PRD-OOS-05 | docs/PRD.md | Restrição | Single-worker inicialmente; sem ordering global | TRANSCRICAO | [09:12]–[09:13] Diego/Larissa |
| RFC-ALT-01 | docs/RFC.md | Alternativa descartada | HTTP síncrono descartado por acoplamento e risco à transação | TRANSCRICAO | [09:03]–[09:06] Bruno/Diego |
| RFC-ALT-02 | docs/RFC.md | Alternativa descartada | Redis Streams/broker externo descartado por infra adicional | TRANSCRICAO | [09:06]–[09:08] Diego |
| RFC-ALT-03 | docs/RFC.md | Alternativa descartada | Trigger MySQL não notifica processo externo | TRANSCRICAO | [09:09]–[09:10] Diego |
| RFC-ALT-04 | docs/RFC.md | Alternativa descartada | Retry infinito e três tentativas descartados | TRANSCRICAO | [09:15]–[09:17] Diego/Bruno |
| RFC-ALT-05 | docs/RFC.md | Alternativa descartada | Exactly-once descartado em favor de at-least-once | TRANSCRICAO | [09:24]–[09:26] Diego/Sofia |
| RFC-OPEN-01 | docs/RFC.md | Questão aberta | Rate limiting por cliente não decidido | TRANSCRICAO | [09:38]–[09:39] Diego/Larissa |
| RFC-OPEN-02 | docs/RFC.md | Questão aberta | Escala futura: particionamento ou lock | TRANSCRICAO | [09:12]–[09:13] Diego |
| RFC-OPEN-03 | docs/RFC.md | Questão aberta | Retenção/arquivamento fora de escopo atual | TRANSCRICAO | [09:07]–[09:08] Diego |
| ADR-001 | docs/adrs/ADR-001-outbox-no-mysql.md | Decisão | Outbox MySQL na mesma transação do pedido | TRANSCRICAO | [09:06]–[09:08] Diego/Larissa |
| ADR-002 | docs/adrs/ADR-002-worker-separado-polling.md | Decisão | Worker separado em polling de 2 s | TRANSCRICAO | [09:09]–[09:11] Diego/Larissa |
| ADR-003 | docs/adrs/ADR-003-retry-backoff-dlq.md | Decisão | Política de cinco tentativas, sequência de intervalos e DLQ; semântica da contagem pendente | TRANSCRICAO | [09:15]–[09:19] Diego/Larissa |
| ADR-004 | docs/adrs/ADR-004-hmac-secret-por-endpoint.md | Decisão | HMAC-SHA256, secret por endpoint, rotação 24 h | TRANSCRICAO | [09:19]–[09:22] Sofia |
| ADR-005 | docs/adrs/ADR-005-at-least-once-event-id.md | Decisão | At-least-once e deduplicação com UUID | TRANSCRICAO | [09:24]–[09:26] Diego |
| ADR-006 | docs/adrs/ADR-006-reuso-padroes-existentes.md | Decisão | Reusar padrões do projeto e códigos WEBHOOK_ | TRANSCRICAO | [09:27]–[09:31] Bruno/Larissa |
| ADR-007 | docs/adrs/ADR-007-snapshot-e-uuid.md | Decisão | UUID e snapshot renderizado na inserção | TRANSCRICAO | [09:51]–[09:52] Larissa/Diego/Bruno |
| FDD-FLOW-01 | docs/FDD.md | Fluxo | Outbox usa tx client recebido de changeStatus | TRANSCRICAO | [09:40]–[09:42] Bruno/Diego |
| FDD-SEC-01 | docs/FDD.md | Segurança | URL HTTPS, payload até 64 KB, timeout 10 s | TRANSCRICAO | [09:23]–[09:24] Sofia/Larissa; [09:42] Diego |
| FDD-API-01 | docs/FDD.md | Contrato proposto | CRUD e deliveries sob /api/v1 | TRANSCRICAO | [09:31]–[09:35] Marcos/Bruno |
| FDD-API-02 | docs/FDD.md | Contrato proposto | Rotação de secret exposta por API; path a definir | TRANSCRICAO | [09:21] Sofia |
| FDD-API-03 | docs/FDD.md | Contrato proposto | Replay DLQ com role ADMIN | TRANSCRICAO | [09:35]–[09:36] Sofia/Larissa |
| FDD-ERR-01 | docs/FDD.md | Padrão de erro | Prefixo WEBHOOK_ e AppError existentes | TRANSCRICAO | [09:27]–[09:31] Bruno/Larissa |
| FDD-OBS-01 | docs/FDD.md | Observabilidade | Reuso de Pino e middleware de erro | TRANSCRICAO | [09:29]–[09:31] Bruno/Larissa |
| CODE-01 | docs/FDD.md | Integração com código | changeStatus já usa transação Prisma e atualiza pedido/histórico | CODIGO | src/modules/orders/order.service.ts |
| CODE-02 | docs/FDD.md | Integração com código | Rotas de pedido usam authenticate e validate | CODIGO | src/modules/orders/order.routes.ts |
| CODE-03 | docs/FDD.md | Integração com código | buildControllers compõe serviços e buildApp monta /api/v1 | CODIGO | src/app.ts |
| CODE-04 | docs/FDD.md | Integração com código | PrismaClient compartilhado criado pela configuração de banco | CODIGO | src/config/database.ts |
| CODE-05 | docs/FDD.md | Integração com código | Erros compartilhados exportam AppError e erros HTTP | CODIGO | src/shared/errors/index.ts |
| CODE-06 | docs/FDD.md | Integração com código | Schema usa MySQL, UUIDs e enum OrderStatus | CODIGO | prisma/schema.prisma |
| CODE-07 | docs/FDD.md | Integração com código | Entry point da API inicializa app e shutdown | CODIGO | src/server.ts |
| CODE-08 | docs/FDD.md | Integração com código | API instala middleware de erro centralizado | CODIGO | src/app.ts; src/middlewares/error.middleware.ts |
| CODE-09 | docs/RFC.md; docs/FDD.md | Autorização | JWT autenticado contém id, email e role, sem customer_id ou vínculo de customer | CODIGO | src/middlewares/auth.middleware.ts |
| FDD-OPEN-01 | docs/FDD.md | Questão aberta | Definir contagem de chamadas e associação dos intervalos de retry | TRANSCRICAO | [09:15]–[09:18] Diego/Larissa |
| FDD-OPEN-02 | docs/FDD.md | Questão aberta | Definir formato textual da assinatura | TRANSCRICAO | [09:19]–[09:22] Sofia |
| FDD-OPEN-03 | docs/FDD.md | Questão aberta | Definir paginação e retenção do histórico | TRANSCRICAO | [09:34] Marcos |
| FDD-OPEN-04 | docs/FDD.md | Questão aberta | Definir autorização cruzada por customer | TRANSCRICAO | [09:31]–[09:37] Marcos/Bruno/Sofia |
| RFC-OPEN-04 | docs/RFC.md; docs/FDD.md; docs/adrs/ADR-003-retry-backoff-dlq.md | Questão aberta | Se são cinco chamadas totais ou cinco retries e como os cinco intervalos se associam às chamadas | TRANSCRICAO | [09:15]–[09:18] Diego/Larissa |
| RFC-OPEN-05 | docs/RFC.md; docs/FDD.md; docs/adrs/ADR-004-hmac-secret-por-endpoint.md | Questão de segurança | HMAC foi definido sobre o corpo; autenticação adicional de X-Timestamp exigiria decisão | TRANSCRICAO | [09:22] Sofia; [09:44] Diego |
| RFC-OPEN-06 | docs/RFC.md; docs/FDD.md; docs/adrs/ADR-005-at-least-once-event-id.md | Questão aberta | Definir se replay preserva event_id ou gera outro | TRANSCRICAO | [09:18] Diego; [09:24]–[09:26] Diego/Sofia |
| RFC-OPEN-07 | docs/RFC.md; docs/FDD.md | Questão aberta | Customer_id é informado no body/path; falta definir o escopo de autorização entre customers | TRANSCRICAO | [09:31]–[09:37] Marcos/Larissa/Bruno/Sofia |
| RFC-OPEN-08 | docs/RFC.md; docs/FDD.md | Questão aberta | Fechar paginação, ordenação e filtros do histórico de deliveries | TRANSCRICAO | [09:34] Marcos |
| FDD-OPEN-05 | docs/FDD.md | Questão aberta | Autenticação de X-Timestamp não decorre da assinatura HMAC do corpo; decidir extensão se necessária | TRANSCRICAO | [09:22] Sofia; [09:44] Diego |
