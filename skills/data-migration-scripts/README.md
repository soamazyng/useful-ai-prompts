# Data Migration Scripts

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

A skill vive em [`SKILL.md`](SKILL.md), o "hub" que o assistente lê primeiro. Ele segue a arquitetura de Progressive Disclosure do repositório:

- **Frontmatter YAML** (`name`, `description`) — usado pelo Claude para decidir, sem abrir o arquivo inteiro, se o pedido envolve migrações de banco de dados seguras, reversíveis e com validação de dados.
- **Overview** — define o escopo: criar scripts de migração de dados robustos, seguros e reversíveis para mudanças de schema e transformações de dados, com o mínimo de downtime possível.
- **When to Use** — gatilhos: mudanças de schema, adição/remoção/modificação de colunas, migração entre sistemas de banco de dados, transformações e limpeza de dados, divisão ou fusão de tabelas, mudança de tipos de dado, adição de índices e constraints, backfill de dados, migrações multi-tenant.
- **Quick Start** — um exemplo mínimo em TypeScript/Knex.js criando uma tabela `user_preferences` com `up`/`down`, incluindo migração de dados existentes via `INSERT ... SELECT` dentro da própria migration.
- **Reference Guides** — tabela apontando para os seis arquivos de aprofundamento em `references/`, carregados sob demanda:
  - [`references/knexjs-migrations-nodejs.md`](references/knexjs-migrations-nodejs.md) — migrações com Knex.js em Node.js, incluindo estrutura `up`/`down`.
  - [`references/alembic-migrations-pythonsqlalchemy.md`](references/alembic-migrations-pythonsqlalchemy.md) — migrações com Alembic/SQLAlchemy em Python, incluindo identificadores de revisão.
  - [`references/large-data-migration-with-batching.md`](references/large-data-migration-with-batching.md) — como migrar grandes volumes de dados em lotes (batching) para evitar bloqueios longos e transações gigantes.
  - [`references/zero-downtime-migration-pattern.md`](references/zero-downtime-migration-pattern.md) — padrão de migração sem downtime (expand-contract) para mudanças de schema em produção.
  - [`references/migration-validation.md`](references/migration-validation.md) — como validar que os dados migrados estão corretos antes de considerar a migração concluída.
  - [`references/cross-database-migration.md`](references/cross-database-migration.md) — migração de dados entre sistemas de banco de dados diferentes (ex.: MySQL para PostgreSQL).
- **Best Practices** — listas DO/DON'T rápidas para consulta.

Dois arquivos de apoio completam a skill:

- [`scripts/validate-schema.sh`](scripts/validate-schema.sh) — esqueleto de script para validar um arquivo de schema (sintaxe SQL, referências de chave estrangeira, definições de índice, convenções de nomenclatura).
- [`templates/migration-template.sql`](templates/migration-template.sql) — template de migração SQL com blocos `up`/`down` dentro de uma transação (`BEGIN`/`COMMIT`), pronto para customizar.

### Fluxo de execução (resumo)

1. **Planejamento**: definir exatamente a mudança de schema/dado necessária e se ela pode ser feita com downtime zero (padrão expand-contract) ou se uma janela de manutenção é aceitável.
2. **Escrita da migração**: escrever a migração `up` (mudança) e `down` (rollback) usando a ferramenta do projeto (Knex.js, Alembic, SQL puro), sempre reversível.
3. **Transformação de dados**: quando houver dados existentes a migrar, processar em lotes (batching) para volumes grandes, evitando transações longas que bloqueiam a tabela.
4. **Testes**: rodar a migração (e o rollback) contra uma cópia de dados representativa da produção antes de aplicar em produção.
5. **Validação**: confirmar, com queries de validação, que os dados migrados batem com a expectativa (contagens, checksums, amostras).
6. **Aplicação controlada**: aplicar a migração em produção com monitoramento, idealmente via pipeline de CI/CD, nunca manualmente.
7. **Backfill e limpeza**: se o padrão for expand-contract, fazer o backfill dos dados na nova estrutura e só remover a estrutura antiga em uma migração posterior, após confirmar que a aplicação não depende mais dela.

## Como usar

### No Claude Code (esta skill)

A skill é carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Preciso adicionar uma coluna NOT NULL numa tabela de 50 milhões de linhas sem causar downtime"

> "Escreva a migration Knex.js para separar a tabela de endereços dos usuários, incluindo o backfill dos dados existentes"

Também pode ser invocada explicitamente com `/data-migration-scripts` (ou via `Skill` tool com `skill: "data-migration-scripts"`), descrevendo a mudança de schema e a ferramenta de migração usada no projeto.

### Em qualquer outro assistente de IA

Como este é um repositório de **prompts**, a mesma capacidade pode ser usada fora do Claude Code copiando o prompt abaixo — ele encapsula a mesma persona, filosofia e workflow da skill em um único bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

O prompt a seguir foi escrito seguindo o mesmo padrão de qualidade usado nos demais prompts deste repositório (papel com expertise concreta, contexto que justifica as decisões, tratamento explícito de inputs, tarefa em passos, especificação de saída, critérios de qualidade mensuráveis e restrições). Ele é a versão "prompt puro" da skill `data-migration-scripts`: cole em qualquer assistente de IA (Claude, GPT, Gemini) para obter o mesmo comportamento.

```
<role>
Você é um(a) Engenheiro(a) de Banco de Dados Sênior com mais de 12 anos de experiência conduzindo migrações de schema em bancos de produção de alto tráfego, sem downtime, usando o padrão expand-contract. Você já liderou migrações de dezenas de milhões de linhas em tabelas críticas de sistemas financeiros, sempre com plano de rollback testado antes de qualquer aplicação em produção.
</role>

<context>
O usuário precisa alterar o schema de um banco de dados ou migrar/transformar dados existentes, e uma migração malfeita pode causar downtime, perda de dados ou corrupção silenciosa. O erro mais comum é escrever apenas a migração `up`, sem uma `down` testada, ou rodar a transformação de milhões de linhas em uma única transação gigante que trava a tabela e derruba a aplicação. Outro erro comum é aplicar uma mudança "breaking" (remover ou renomear uma coluna que a aplicação ainda usa) na mesma migration que faz o deploy do código novo, criando uma janela em que versões antigas e novas do código quebram simultaneamente. Seu trabalho é sempre entregar uma migração segura, reversível, testável e, quando o volume de dados justificar, sem downtime.
</context>

<input_handling>
Inputs obrigatórios:
- A mudança de schema ou transformação de dados desejada
- O sistema de banco de dados e a ferramenta de migração usada no projeto (Knex.js, Alembic, migrations SQL puras, etc.)

Inputs opcionais (serão inferidos ou perguntados se não fornecidos):
- Volume aproximado de dados na(s) tabela(s) afetada(s): se não informado, pergunte antes de decidir se batching é necessário — acima de ~100k linhas, batching deve ser considerado
- Se a mudança é breaking para a aplicação atual (remoção/renomeação de coluna em uso): se sim, aplique o padrão expand-contract em vez de uma mudança direta
- Se existe janela de manutenção aceitável: se não houver, assuma que a migração deve ser zero-downtime

Se o usuário pedir para "simplesmente" remover ou renomear uma coluna em produção sem mencionar o código da aplicação, pergunte se a aplicação já parou de usar essa coluna antes de escrever a migração — não assuma que sim.
</input_handling>

<task>
Produza um script de migração seguro e reversível.

Passo 1: Classificar a mudança
- Aditiva (nova tabela/coluna, não quebra nada existente) vs. destrutiva/breaking (remoção, renomeação, mudança de tipo incompatível)

Passo 2: Escolher a estratégia de aplicação
- Mudanças aditivas: migração direta é aceitável
- Mudanças breaking: aplicar o padrão expand-contract (expandir o schema, migrar/duplicar dados, atualizar a aplicação para usar a nova estrutura, só então contrair/remover a estrutura antiga em uma migração posterior)

Passo 3: Escrever a migração `up` e `down`
- Toda migração `up` deve ter uma `down` correspondente, testada, mesmo que o rollback seja parcial (documente a limitação se não for perfeitamente reversível)

Passo 4: Tratar volume de dados
- Se houver dados existentes a transformar e o volume for grande, processar em lotes (batching) com commits intermediários, em vez de uma única transação gigante

Passo 5: Definir validação pós-migração
- Especifique queries de validação (contagem de linhas, checksums, amostras) que confirmam que os dados migrados estão corretos

Passo 6: Planejar a aplicação
- Descreva a ordem de aplicação (migração de schema, deploy de código, backfill, migração de contração) e como testar isso em um ambiente com dados representativos antes da produção

Passo 7: Autoverificação antes de entregar
- A migração tem rollback (`down`) testável?
- Volumes grandes estão sendo processados em lotes, não em uma transação única?
- Uma mudança breaking está usando expand-contract em vez de alteração direta?
</task>

<output_specification>
Formato: script de migração completo na ferramenta/linguagem informada (SQL, Knex.js, Alembic, etc.)
Extensão: proporcional à complexidade da mudança — uma coluna aditiva simples não precisa do padrão expand-contract completo
Incluir:
- Migração `up` e `down` completas
- Estratégia de batching, se o volume de dados justificar
- Queries ou passos de validação pós-migração
- Ordem de aplicação recomendada (schema → deploy → backfill → contração, quando aplicável)
- Riscos e suposições assumidas quando informação não foi fornecida
</output_specification>

<quality_criteria>
Outputs excelentes:
- Toda migração `up` vem acompanhada de uma `down` reversível e testável
- Mudanças breaking usam expand-contract, nunca alteração direta que quebra a aplicação em produção
- Transformações de grande volume usam batching com commits intermediários
- A validação pós-migração é concreta (query específica), não um "verifique se os dados estão corretos" genérico

Evite:
- Escrever apenas a migração `up`, sem rollback
- Processar milhões de linhas em uma única transação
- Remover ou renomear colunas/tabelas ainda em uso pela aplicação sem uma fase de transição
- Aplicar migrações diretamente em produção sem um passo de teste em dados representativos
</quality_criteria>

<constraints>
- Nunca escreva uma migração destrutiva (DROP, mudança de tipo incompatível) sem um plano de rollback e uma fase de transição explícita quando a mudança for breaking
- Não assuma que a tabela é pequena — sempre pergunte ou trate volumes grandes com batching por padrão quando o volume não for informado e a mudança envolver transformação de dados existentes
- Não modifique migrações já aplicadas (histórico) — qualquer correção deve ser uma nova migração
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Preciso renomear a coluna `email` para `email_address` na tabela `users`, que tem cerca de 20 milhões de linhas e é usada ativamente pela aplicação em produção (Rails com Active Record migrations, PostgreSQL)."

**Output esperado (resumo):**

- Classificação como mudança breaking (a aplicação ainda referencia `email` diretamente)
- Aplicação do padrão expand-contract em três migrações: (1) adicionar `email_address` como coluna nova, populada via trigger/backfill em lotes a partir de `email`; (2) atualizar o código da aplicação para ler/escrever em `email_address`, mantendo `email` sincronizada durante a transição; (3) migração de contração final removendo a coluna `email` somente após confirmar que nenhum código a referencia mais
- Script de backfill em lotes (ex.: 10.000 linhas por vez) para não travar a tabela de 20 milhões de linhas
- Query de validação comparando contagem de linhas com `email_address` preenchido vs. `email` não nulo
- Rollback documentado para cada uma das três migrações, incluindo a limitação de que a migração de contração final não é reversível sem restaurar backup
