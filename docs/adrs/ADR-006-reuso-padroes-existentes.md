# ADR-006: Reutilizar padrões existentes da aplicação

- **Status:** Aceita na reunião
- **Decisores:** Larissa, Bruno, Diego

## Contexto

A codebase organiza domínios em módulos com controller, service, repository, routes e schemas. Erros usam `AppError`, códigos nomeados, Pino, Zod e middleware de erro centralizado ([09:27]–[09:31]).

## Decisão

Criar módulo `src/modules/webhooks` seguindo a organização atual. Usar schemas Zod, erros compatíveis com `AppError` e prefixo `WEBHOOK_`, logger Pino e middleware centralizado. Reutilizar autenticação e autorização `requireRole` para replay de ADMIN.

Referências existentes verificadas: `src/modules/orders/order.service.ts`, `src/modules/orders/order.routes.ts`, `src/app.ts`, `src/shared/errors/index.ts` e `src/config/database.ts`.

## Alternativas Consideradas

Na reunião não foi comparada uma convenção alternativa detalhada; a decisão registrada foi reutilizar os padrões existentes.

## Consequências

**Positivas:** consistência com os outros módulos, menor curva de manutenção e reutilização de tratamento de erros.

**Negativas:** o novo módulo precisa respeitar contratos existentes; integração exige registrar controllers/services no composition root e confirmar o middleware para os novos códigos.
