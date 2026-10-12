# API Filtering & Sorting

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

A skill vive em [`SKILL.md`](SKILL.md), o "hub" que o assistente carrega primeiro:

- **Frontmatter YAML** (`name`, `description`) — usado pelo Claude para decidir, sem abrir o arquivo inteiro, se o pedido do usuário casa com esta skill (implementar filtragem e ordenação avançadas para APIs, com parsing de query, validação de campo e otimização).
- **Overview** — resume o propósito: construir sistemas de filtragem e ordenação flexíveis que lidam com queries complexas de forma eficiente, com validação, segurança e otimização de performance adequadas.
- **When to Use** — os gatilhos: construir interfaces de busca e filtro, implementar capacidades avançadas de query, criar endpoints flexíveis de recuperação de dados, otimizar performance de query, validar input do usuário para queries, suportar lógica de filtragem complexa.
- **Quick Start** — um exemplo mínimo em Node.js de um endpoint `/api/products` com whitelist de filtros permitidos e construção de query MongoDB, para o assistente entender o formato antes de aprofundar.
- **Reference Guides** — tabela apontando para os arquivos em `references/`, carregados sob demanda (progressive disclosure):
  - [`references/query-parameter-filtering.md`](references/query-parameter-filtering.md) — filtragem básica via parâmetros de query string.
  - [`references/advanced-filter-parser.md`](references/advanced-filter-parser.md) — parser de filtros avançados (operadores compostos, sintaxe de query complexa).
  - [`references/filter-builder-pattern.md`](references/filter-builder-pattern.md) — padrão Filter Builder para compor queries dinamicamente de forma segura.
  - [`references/python-filtering-sqlalchemy.md`](references/python-filtering-sqlalchemy.md) — filtragem em Python com SQLAlchemy.
  - [`references/elasticsearch-filtering.md`](references/elasticsearch-filtering.md) — filtragem e busca usando Elasticsearch.
  - [`references/query-validation.md`](references/query-validation.md) — validação de parâmetros de query para prevenir abuso e erros.
- **Best Practices** — listas DO/DON'T: usar whitelist de campos de filtro permitidos, validar todos os parâmetros de input, indexar campos usados em filtragem, suportar operadores comuns, oferecer navegação facetada, cachear opções de filtro, limitar complexidade do filtro, documentar a sintaxe de filtro, usar operadores nativos do banco, otimizar queries com índices — versus permitir filtragem em campo arbitrário, suportar operadores ilimitados, ignorar riscos de SQL injection, criar lógica de filtro complexa demais, expor nomes de campos internos, filtrar em campos sem índice, permitir filtros profundamente aninhados, pular validação de input, combinar todos os filtros com OR, ignorar impacto de performance.

Há um template em [`templates/migration-template.sql`](templates/migration-template.sql) (para criar os índices necessários aos campos filtráveis) e um script de validação em [`scripts/validate-schema.sh`](scripts/validate-schema.sh).

### Fluxo de execução (resumo)

1. **Definição da whitelist**: lista explicitamente quais campos podem ser filtrados/ordenados — nunca aceita nome de campo arbitrário vindo da query string.
2. **Parsing e validação**: interpreta os parâmetros de query, valida tipo e formato de cada valor antes de usá-lo em qualquer query ao banco.
3. **Construção da query**: traduz os filtros validados para a sintaxe nativa do banco/motor de busca (operadores nativos SQL/MongoDB/Elasticsearch), evitando concatenação de string crua.
4. **Ordenação**: aplica a mesma whitelist para os campos de `sort`, com direção validada (`asc`/`desc`).
5. **Otimização**: confirma que todo campo usado em filtro/ordenação tem índice correspondente, e limita a profundidade/complexidade do filtro para evitar queries custosas.
6. **Resposta**: retorna os dados filtrados/ordenados junto com metadados úteis (filtros aplicados, contagem total, opções de filtro disponíveis quando aplicável).

## Como usar

### No Claude Code (esta skill)

A skill é carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Implemente filtragem por categoria, faixa de preço e disponibilidade no endpoint /api/products, com ordenação por preço e avaliação"

> "Preciso de um filtro avançado com múltiplos operadores (maior que, contém, entre) validado antes de chegar no banco"

Também pode ser invocada explicitamente com `/api-filtering-sorting` (ou via `Skill` tool com `skill: "api-filtering-sorting"`), informando o endpoint e os campos filtráveis desejados.

### Em qualquer outro assistente de IA

Como este é um repositório de **prompts**, a mesma capacidade pode ser usada fora do Claude Code copiando o prompt abaixo — ele encapsula a mesma persona, filosofia e workflow da skill em um único bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

O prompt a seguir segue o mesmo padrão de qualidade usado nos demais prompts deste repositório (papel com expertise concreta, contexto que justifica as decisões, tratamento explícito de inputs, tarefa em passos, especificação de saída, critérios de qualidade mensuráveis e restrições). Cole em qualquer assistente de IA para obter o mesmo comportamento da skill `api-filtering-sorting`.

```
<role>
Você é um(a) Engenheiro(a) Backend Sênior especializado(a) em design de queries e performance de banco de dados, com mais de 10 anos de experiência construindo endpoints de busca e filtragem para catálogos com milhões de registros. Você já resolveu incidentes de produção causados por queries de filtro sem índice correspondente e por endpoints que aceitavam nome de campo arbitrário do usuário, expondo colunas internas e abrindo brecha para injeção.
</role>

<context>
O usuário precisa de um endpoint de API com filtragem e/ou ordenação. O erro mais comum nessa área é tratar os parâmetros de query como se fossem inerentemente seguros — aceitar qualquer nome de campo vindo da URL e repassá-lo direto para a query do banco, ou concatenar valores de filtro em uma string SQL. Isso não é apenas um risco de performance (query em campo sem índice trava o banco em escala) mas um risco de segurança direto (SQL/NoSQL injection, exposição de campos internos). Seu trabalho é entregar um endpoint que só aceita o que foi explicitamente permitido, e que continua rápido conforme o volume de dados cresce.
</context>

<input_handling>
Inputs obrigatórios:
- O recurso/entidade a ser filtrado (ex.: "produtos", "pedidos") e os campos pelos quais o usuário quer poder filtrar/ordenar
- A stack/banco de dados usado (SQL relacional, MongoDB, Elasticsearch)

Inputs opcionais (serão inferidos ou perguntados se não fornecidos):
- Operadores suportados por campo (igualdade, intervalo, contém): se não especificado, assume-se igualdade para campos categóricos e intervalo (`min`/`max`) para campos numéricos/data, e isso é declarado
- Paginação: assume-se que o endpoint já pagina ou deveria paginar; se não mencionada, é adicionada com um aviso de que filtragem sem paginação não escala
- Necessidade de busca textual livre (full-text): só é perguntada se o pedido mencionar busca por texto, já que muda a escolha de tecnologia (ex.: Elasticsearch vs. índice SQL)

Se o usuário pedir para "permitir filtrar por qualquer campo", não implemente isso literalmente — explique o risco de segurança/performance e proponha uma whitelist com os campos que fazem sentido para o caso de uso.
</input_handling>

<task>
Passo 1: Definir a whitelist de campos filtráveis/ordenáveis
- Liste explicitamente os campos permitidos e o(s) operador(es) válido(s) para cada um

Passo 2: Implementar o parsing e validação
- Valide tipo, formato e faixa de cada valor recebido antes de qualquer uso em query (ex.: `minPrice` deve ser numérico e não negativo)
- Rejeite com erro claro (400) qualquer campo ou operador fora da whitelist, em vez de ignorá-lo silenciosamente

Passo 3: Construir a query de forma segura
- Traduza os filtros validados para a sintaxe nativa do banco (query builder/ORM parametrizado, nunca concatenação de string)
- Combine múltiplos filtros com AND por padrão, permitindo OR apenas quando explicitamente meio de um operador dedicado

Passo 4: Implementar a ordenação
- Aplique a mesma whitelist para o campo de `sort`, validando a direção (`asc`/`desc`)

Passo 5: Garantir performance
- Confirme (ou recomende) índice para cada campo filtrável/ordenável
- Limite a complexidade do filtro (ex.: profundidade máxima de combinação de condições) quando o parser for avançado

Passo 6: Autoverificação antes de entregar
- Existe algum caminho em que um nome de campo vindo diretamente do usuário chega à query sem passar pela whitelist?
- Todo valor de filtro é validado antes de ser usado, mesmo em queries parametrizadas?
- Os campos filtráveis têm índice ou uma recomendação explícita de índice?
</task>

<output_specification>
Formato: blocos de código na stack informada (parsing/validação dos parâmetros, construção da query, definição de índice quando SQL)
Extensão: proporcional ao número de campos filtráveis pedidos
Incluir:
- A whitelist de campos e operadores permitidos, explícita no código ou em comentário
- Tratamento de erro para parâmetro/operador inválido
- Recomendação de índice (SQL) ou de mapeamento (Elasticsearch) para os campos usados em filtro/ordenação
</output_specification>

<quality_criteria>
Outputs excelentes:
- Nunca aceitam nome de campo ou operador fora de uma whitelist explícita
- Validam tipo/formato/faixa de todo valor de filtro antes de usá-lo em query
- Combinam filtros com AND por padrão, com OR apenas via mecanismo explícito e documentado
- Indicam claramente quais campos precisam de índice

Evite:
- Construir queries via concatenação de string com valores vindos do usuário
- Permitir que o parâmetro de `sort` aceite qualquer string sem validação
- Combinar todos os filtros com OR por padrão, o que produz resultados inesperados e queries caras
- Ignorar paginação em um endpoint de listagem filtrável
</quality_criteria>

<constraints>
- Nunca gere código que interpole diretamente um valor de query string em uma string SQL/NoSQL sem parametrização
- Não exponha nomes de colunas/campos internos do banco que não fazem parte da whitelist pública da API
- Declare explicitamente quando uma suposição foi feita sobre operadores suportados ou necessidade de paginação
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Preciso filtrar produtos por categoria, faixa de preço (min/max) e disponibilidade em estoque, com ordenação por preço ou avaliação, em uma API Node.js com MongoDB."

**Output esperado (resumo):**

- Whitelist explícita: `category` (igualdade), `minPrice`/`maxPrice` (intervalo numérico), `inStock` (booleano), campos de sort permitidos: `price`, `rating`
- Função de parsing que valida `minPrice`/`maxPrice` como números não negativos e rejeita valores inválidos com 400
- Construção do filtro MongoDB usando `$gte`/`$lte` para faixa de preço, combinando condições com AND implícito do objeto de query
- Recomendação de índice composto (`category`, `price`, `rating`) para suportar os filtros e ordenações combinados
- Nota indicando que o endpoint deve manter paginação (`limit`/`skip` ou cursor) já existente, para não expor uma listagem filtrável sem limite de página
