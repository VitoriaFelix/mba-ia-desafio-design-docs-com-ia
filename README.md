# Processo de produção dos design docs

## Objetivo e fontes

Este pacote documenta a feature de webhooks outbound para notificar clientes B2B quando o status de um pedido muda. As fontes primárias foram `TRANSCRICAO.md` e o código existente neste fork. Os documentos de proposta fornecidos em `design-docs-webhooks/docs/` serviram como rascunho de conteúdo e foram confrontados com essas fontes antes de integrar.

A entrega altera somente Markdown. O código da aplicação, os testes, o schema Prisma e as configurações não foram modificados.

## Ferramentas usadas

- **ChatGPT (Codex):** leitura e comparação da transcrição, dos documentos propostos e dos arquivos do fork; organização e revisão de consistência e rastreabilidade.
- **PowerShell:** inspeção local de arquivos e estado Git, cópia/adaptação dos documentos e verificações textuais de links e whitespace.

Não usei GitHub/raw URLs como fonte de confirmação nesta revisão; as referências ao código foram conferidas no clone local.

## Workflow

1. Confirmei o caminho do workspace, a presença de `.git` e dos arquivos/pastas exigidos, e o estado inicial do Git.
2. Li os placeholders já existentes e preservei seus caminhos, preenchendo PRD, RFC, FDD e Tracker após revisar o conteúdo proposto.
3. Comparei as referências de integração com `src/modules/orders/order.service.ts`, `src/modules/orders/order.routes.ts`, `src/app.ts`, `src/routes/index.ts`, `src/config/database.ts`, os middlewares, erros, logger e `prisma/schema.prisma`.
4. Registrei sete decisões confirmadas em ADRs e mantive no RFC/FDD as lacunas que a reunião não resolveu.
5. Revisei caminhos e links Markdown e executei `git diff --check`. Não executei testes da aplicação por se tratar de uma entrega documental.

## Prompts customizados

### Extração rastreável da transcrição

```text
Separe decisões confirmadas, requisitos, alternativas descartadas, itens fora de escopo e questões abertas na transcrição. Para cada afirmação, registre timestamp e falante. Não transforme uma sugestão em decisão e não preencha lacunas com práticas genéricas.
```

### Conferência da integração com o código

```text
Confira cada caminho citado diretamente no clone. Descreva só o comportamento observado e separe arquivos existentes de novos arquivos propostos. Não altere src/, prisma/, tests/ ou configurações; proponha apenas documentação.
```

### Revisão cruzada

```text
Compare PRD, RFC, FDD, ADRs e Tracker com a transcrição e o código local. Procure contradições, referências inexistentes, decisões sem fonte e questões abertas tratadas como fechadas. Verifique links relativos e preserve README.md e TRANSCRICAO.md.
```

## Iterações e ajustes

- A primeira pasta aberta no app era um diretório de trabalho sem clone. Após o caminho do fork ser informado, confirmei o clone correto antes de começar a escrever.
- O fork já continha `docs/PRD.md`, `RFC.md`, `FDD.md` e `TRACKER.md` como placeholders. Revisei os documentos propostos antes de preenchê-los.
- As propostas foram confrontadas com os caminhos reais do fork. Mantive `customer_id` e controle de acesso cruzado como ponto a fechar porque o JWT atual contém identidade/role de usuário, mas não vínculo de customer; também deixei aberta a semântica exata das cinco tentativas e detalhes de consulta/retenção de deliveries.
- Ajustei a referência do FDD para indicar que a validação foi feita no código deste fork e criei um índice de ADRs compatível com os nomes efetivamente adotados.

## Navegação

1. [PRD](docs/PRD.md) — problema, público, escopo e critérios de sucesso.
2. [RFC](docs/RFC.md) — arquitetura proposta, alternativas e questões para revisão.
3. [ADRs](docs/adrs/README.md) — decisões arquiteturais confirmadas.
4. [FDD](docs/FDD.md) — fluxos, contratos e integração técnica proposta.
5. [Tracker](docs/TRACKER.md) — rastreabilidade à transcrição e ao código.

## Limites da documentação

Contratos de API, tabelas e arquivos novos no FDD são propostas para implementação, não recursos existentes. Os detalhes não decididos pela reunião devem ser aprovados antes de codificar. A meta de latência inferior a 10 segundos é um objetivo em condições normais, não uma garantia sob indisponibilidade do destino ou acúmulo de fila.
