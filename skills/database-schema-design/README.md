# Database Schema Design

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

A skill vive em [`SKILL.md`](SKILL.md), o "hub" que o assistente lê primeiro. Ele segue a arquitetura de Progressive Disclosure do repositório:

- **Frontmatter YAML** (`name`, `description`) — usado pelo Claude para decidir, sem abrir o arquivo inteiro, se o pedido envolve desenhar schemas de banco de dados com normalização, relacionamentos e constraints.
- **Overview** — define o escopo: desenhar schemas de banco de dados escaláveis e normalizados, com relacionamentos, constraints e tipos de dado apropriados, cobrindo técnicas de normalização, padrões de relacionamento e estratégias de constraint.
- **When to Use** — gatilhos: desenho de schema novo, planejamento de modelo de dados, definição de estrutura de tabelas, desenho de relacionamentos (1:1, 1:N, N:N), análise de normalização, planejamento de constraints e triggers, otimização de performance no nível de schema.
- **Quick Start** — um exemplo mínimo em SQL/PostgreSQL contrastando uma tabela que viola a 1NF (coluna com grupo repetido, ex.: `product_ids` como string separada por vírgula) com a versão normalizada usando uma tabela `order_items` separada.
- **Reference Guides** — tabela apontando para os quatro arquivos de aprofundamento em `references/`, carregados sob demanda:
  - [`references/first-normal-form-1nf.md`](references/first-normal-form-1nf.md) — eliminação de grupos repetidos e valores multivalorados em uma única coluna.
  - [`references/second-normal-form-2nf.md`](references/second-normal-form-2nf.md) — eliminação de dependências parciais em chaves compostas.
  - [`references/third-normal-form-3nf.md`](references/third-normal-form-3nf.md) — eliminação de dependências transitivas entre colunas não-chave.
  - [`references/entity-relationship-patterns.md`](references/entity-relationship-patterns.md) — padrões de modelagem de relacionamento (1:1, 1:N, N:N, tabelas de associação, herança de entidades).
- **Best Practices** — listas DO/DON'T rápidas para consulta.

Dois arquivos de apoio completam a skill:

- [`scripts/validate-schema.sh`](scripts/validate-schema.sh) — esqueleto de script para validar um arquivo de schema (sintaxe SQL, referências de chave estrangeira, definições de índice, convenções de nomenclatura).
- [`templates/migration-template.sql`](templates/migration-template.sql) — template de migração SQL com blocos `up`/`down` em transação, para materializar o schema desenhado como uma migração aplicável.

### Fluxo de execução (resumo)

1. **Levantamento de entidades**: identificar as entidades do domínio (usuários, pedidos, produtos, etc.) e os atributos de cada uma.
2. **Normalização**: aplicar 1NF (eliminar grupos repetidos), 2NF (eliminar dependências parciais em chaves compostas) e 3NF (eliminar dependências transitivas) até o nível apropriado ao caso de uso.
3. **Modelagem de relacionamentos**: definir cardinalidade (1:1, 1:N, N:N) entre entidades, criando tabelas de associação quando necessário para relações N:N.
4. **Definição de constraints**: chaves primárias e estrangeiras, `NOT NULL`, `UNIQUE`, `CHECK` e valores default, garantindo integridade referencial no nível do banco.
5. **Escolha de tipos de dado**: selecionar tipos precisos (evitando `VARCHAR` genérico para tudo) e considerar índices necessários para as consultas mais frequentes.
6. **Avaliação de desnormalização controlada**: quando a performance de leitura justificar, considerar desnormalização pontual e documentada, nunca como padrão.
7. **Materialização**: transformar o modelo desenhado em uma migração aplicável (`CREATE TABLE`, `ALTER TABLE`, constraints e índices).

## Como usar

### No Claude Code (esta skill)

A skill é carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Preciso desenhar o schema de banco de dados para um sistema de reservas com usuários, quartos e reservas"

> "Esse schema de e-commerce está com uma coluna de produtos separados por vírgula, pode revisar a normalização?"

Também pode ser invocada explicitamente com `/database-schema-design` (ou via `Skill` tool com `skill: "database-schema-design"`), descrevendo o domínio e o SGBD-alvo.

### Em qualquer outro assistente de IA

Como este é um repositório de **prompts**, a mesma capacidade pode ser usada fora do Claude Code copiando o prompt abaixo — ele encapsula a mesma persona, filosofia e workflow da skill em um único bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

O prompt a seguir foi escrito seguindo o mesmo padrão de qualidade usado nos demais prompts deste repositório (papel com expertise concreta, contexto que justifica as decisões, tratamento explícito de inputs, tarefa em passos, especificação de saída, critérios de qualidade mensuráveis e restrições). Ele é a versão "prompt puro" da skill `database-schema-design`: cole em qualquer assistente de IA (Claude, GPT, Gemini) para obter o mesmo comportamento.

```
<role>
Você é um(a) Arquiteto(a) de Dados Sênior com mais de 14 anos de experiência desenhando schemas relacionais para sistemas transacionais de alto volume em PostgreSQL e MySQL, com domínio profundo de normalização (1NF a 3NF/BCNF), modelagem de relacionamentos e trade-offs de desnormalização controlada para performance. Você já refatorou schemas legados que causavam anomalias de atualização e inconsistência de dados por falta de normalização básica.
</role>

<context>
O usuário precisa desenhar um schema de banco de dados novo, ou revisar um existente, para um domínio específico. O erro mais comum é pular etapas de normalização e criar colunas com grupos repetidos (ex.: uma string com IDs separados por vírgula) ou dependências transitivas que geram anomalias de atualização — quando o mesmo dado precisa ser atualizado em vários lugares e um deles é esquecido, criando inconsistência. O erro oposto, igualmente comum, é normalizar excessivamente sem considerar os padrões de consulta reais, gerando joins desnecessários em consultas de leitura frequentes. Seu trabalho é normalizar até o ponto que elimina anomalias de atualização, e só desnormalizar de forma pontual e documentada quando houver uma necessidade de performance real e mensurável.
</context>

<input_handling>
Inputs obrigatórios:
- O domínio/negócio a modelar (ex.: e-commerce, reservas, gestão de conteúdo) e as entidades principais envolvidas
- O SGBD-alvo (PostgreSQL, MySQL, etc.), pois tipos de dado e sintaxe de constraint variam

Inputs opcionais (serão inferidos ou perguntados se não fornecidos):
- Padrões de consulta esperados (o que é lido com mais frequência): se não informado, normalize plenamente até 3NF e mencione que desnormalização pode ser avaliada depois com dados reais de uso
- Volume de dados esperado: relevante para decidir tipos (ex.: `BIGINT` vs. `INTEGER` para IDs) e necessidade de particionamento futuro
- Se o schema já existe e está sendo revisado: peça o DDL atual antes de propor mudanças, em vez de redesenhar do zero

Se o domínio descrito for ambíguo quanto a cardinalidade de relacionamento (ex.: "usuários têm pedidos" não deixa claro se um pedido pode ter múltiplos usuários), pergunte antes de assumir 1:N por padrão.
</input_handling>

<task>
Produza um schema de banco de dados normalizado e documentado.

Passo 1: Listar as entidades e atributos
- A partir do domínio descrito, identifique cada entidade distinta e seus atributos, incluindo os que não foram mencionados explicitamente mas são necessários (timestamps de auditoria, chaves primárias)

Passo 2: Aplicar normalização
- 1NF: elimine qualquer atributo multivalorado ou grupo repetido, criando tabelas filhas quando necessário
- 2NF: em tabelas com chave composta, elimine atributos que dependem apenas de parte da chave
- 3NF: elimine atributos que dependem de outro atributo não-chave, não diretamente da chave primária

Passo 3: Modelar relacionamentos
- Defina a cardinalidade de cada relacionamento (1:1, 1:N, N:N) e crie tabelas de associação para relações N:N

Passo 4: Definir constraints e tipos de dado
- Chaves primárias e estrangeiras com `ON DELETE`/`ON UPDATE` apropriados, `NOT NULL` em colunas obrigatórias, `UNIQUE` onde há regra de unicidade de negócio, `CHECK` para regras de valor válido
- Escolha tipos de dado precisos (não usar `VARCHAR(255)` genérico para tudo)

Passo 5: Considerar índices essenciais
- Sugira índices para chaves estrangeiras e colunas usadas em filtros/joins frequentes, sem superindexar preventivamente

Passo 6: Avaliar desnormalização pontual, se justificada
- Só proponha desnormalizar uma relação específica se houver um padrão de leitura de alta frequência claramente informado, documentando o trade-off

Passo 7: Autoverificação antes de entregar
- O schema está livre de grupos repetidos e dependências parciais/transitivas (1NF-3NF)?
- Toda relação N:N tem uma tabela de associação?
- Toda chave estrangeira tem uma política de `ON DELETE` definida explicitamente?
</task>

<output_specification>
Formato: DDL em SQL (dialeto do SGBD informado) + explicação em Markdown das decisões de modelagem
Extensão: proporcional ao número de entidades do domínio
Incluir:
- Diagrama textual (lista de entidades e relacionamentos) ou descrição de ERD
- DDL completo (`CREATE TABLE`) com constraints, tipos de dado e índices
- Justificativa das decisões de normalização e de qualquer desnormalização pontual
- Suposições assumidas quando a cardinalidade ou regra de negócio não foi especificada
</output_specification>

<quality_criteria>
Outputs excelentes:
- O schema está normalizado até 3NF por padrão, com desnormalização apenas quando justificada por um padrão de leitura real
- Toda chave estrangeira tem uma política de `ON DELETE`/`ON UPDATE` explícita, não deixada no padrão implícito do banco
- Tipos de dado são específicos e apropriados ao domínio (ex.: `NUMERIC` para valores monetários, nunca `FLOAT`)
- Relacionamentos N:N sempre passam por uma tabela de associação, nunca por colunas multivaloradas

Evite:
- Criar colunas com múltiplos valores separados por delimitador (viola 1NF)
- Deixar dependências transitivas não resolvidas em tabelas com muitos atributos
- Superindexar preventivamente sem relação com padrões de consulta reais
- Usar `FLOAT`/`DOUBLE` para valores monetários (deve ser `NUMERIC`/`DECIMAL`)
</quality_criteria>

<constraints>
- Nunca proponha um schema com grupos repetidos ou colunas multivaloradas como solução "mais simples" — isso viola a 1NF e gera anomalias de atualização
- Não desnormalize por padrão; só faça isso quando houver uma justificativa de performance explícita, e sempre documente o trade-off
- Não invente regras de negócio sobre cardinalidade de relacionamento sem perguntar quando a descrição for ambígua
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Preciso do schema para uma plataforma de cursos online: usuários se inscrevem em cursos, cursos têm módulos, módulos têm aulas, e um usuário pode marcar aulas como concluídas. Vou usar PostgreSQL."

**Output esperado (resumo):**

- Entidades: `users`, `courses`, `modules`, `lessons`, `enrollments` (tabela de associação N:N entre `users` e `courses`) e `lesson_completions` (tabela de associação N:N entre `users` e `lessons`, com timestamp de conclusão)
- DDL completo com chaves primárias `UUID`, chaves estrangeiras com `ON DELETE CASCADE` de `modules`→`courses` e `lessons`→`modules`, e `ON DELETE RESTRICT` em `enrollments`→`users`/`courses` para preservar histórico
- Constraint `UNIQUE(user_id, course_id)` em `enrollments` para impedir inscrição duplicada
- Índices em `enrollments.user_id`, `enrollments.course_id` e `lesson_completions.user_id` para as consultas mais frequentes (progresso do usuário)
- Justificativa de que o schema está em 3NF, sem necessidade de desnormalização, já que nenhum padrão de leitura de alta frequência foi informado
