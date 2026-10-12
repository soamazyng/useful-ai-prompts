# Database Schema Documentation

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

A skill vive em [`SKILL.md`](SKILL.md), o "hub" que o assistente lê primeiro. Ele segue a arquitetura de Progressive Disclosure do repositório:

- **Frontmatter YAML** (`name`, `description`) — usado pelo Claude para decidir, sem abrir o arquivo inteiro, se o pedido envolve documentar schemas, criar diagramas ERD ou escrever documentação de tabelas.
- **Overview** — define o escopo: criar documentação abrangente de schema de banco de dados, incluindo diagramas de entidade-relacionamento (ERD), definições de tabela, índices, constraints e dicionário de dados.
- **When to Use** — gatilhos: documentação de schema de banco, diagramas ERD, criação de dicionário de dados, documentação de relacionamento entre tabelas, documentação de índices e constraints, documentação de migração, especificações de design de banco de dados.
- **Quick Start** — um exemplo mínimo em Markdown de um documento de schema (cabeçalho com versão do banco e data) contendo um diagrama Mermaid `erDiagram` relacionando `users`, `orders`, `order_items`, `products`, `payments`, `addresses`, `payment_methods`, `categories`, `product_images` e `inventory`.
- **Reference Guides** — tabela apontando para os cinco arquivos de aprofundamento em `references/`, carregados sob demanda. Diferente de outras skills da biblioteca, aqui os arquivos de referência são o próprio dicionário de dados de exemplo de um schema de e-commerce, prontos para servir de modelo:
  - [`references/users.md`](references/users.md) — documentação completa da tabela `users` (colunas, tipos, nulidade, defaults, descrições e índices, incluindo campos de 2FA e soft delete).
  - [`references/products.md`](references/products.md) — documentação da tabela `products`.
  - [`references/orders.md`](references/orders.md) — documentação da tabela `orders` (colunas monetárias, status, moeda).
  - [`references/orderitems.md`](references/orderitems.md) — documentação da tabela `order_items`.
  - [`references/enum-types.md`](references/enum-types.md) — tipos enumerados usados no schema (`order_status`, `payment_status`) e estruturas JSONB documentadas (ex.: formato de `shipping_address`, `product_snapshot`).
- **Best Practices** — listas DO/DON'T rápidas para consulta.

Dois arquivos de apoio completam a skill:

- [`scripts/validate-schema.sh`](scripts/validate-schema.sh) — esqueleto de script para validar um arquivo de schema (sintaxe SQL, referências de chave estrangeira, definições de índice, convenções de nomenclatura) antes de documentá-lo.
- [`templates/migration-template.sql`](templates/migration-template.sql) — template de migração SQL com blocos `up`/`down`, útil para manter a documentação sincronizada a cada mudança de schema.

### Fluxo de execução (resumo)

1. **Levantamento**: extrair o schema atual (via introspecção do banco, arquivo de migração, ou DDL fornecido pelo usuário) para garantir que a documentação reflete a realidade, não uma versão desatualizada.
2. **Diagrama ERD**: gerar um diagrama de entidade-relacionamento (ex.: em Mermaid) mostrando todas as tabelas e a cardinalidade de seus relacionamentos.
3. **Documentação por tabela**: para cada tabela, listar colunas com tipo, nulidade, valor default e descrição em linguagem natural, seguindo o mesmo formato usado nos exemplos de `references/`.
4. **Índices e constraints**: documentar cada índice (incluindo índices parciais/condicionais) e constraint (chave estrangeira, unique, check) com o SQL exato que os define.
5. **Tipos especiais**: documentar enums e a estrutura de campos JSONB (formato esperado, campos obrigatórios/opcionais) quando existirem.
6. **Versionamento**: incluir metadados de versão do schema/documento e data da última atualização, para que a documentação evolua junto com as migrações.
7. **Revisão de completude**: conferir que nenhuma tabela, índice, constraint ou enum do schema real ficou de fora da documentação.

## Como usar

### No Claude Code (esta skill)

A skill é carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Preciso documentar o schema do meu banco de e-commerce, incluindo o ERD e o dicionário de dados"

> "Gere a documentação das tabelas orders e order_items, incluindo os enums de status usados"

Também pode ser invocada explicitamente com `/database-schema-documentation` (ou via `Skill` tool com `skill: "database-schema-documentation"`), fornecendo o DDL ou schema a documentar.

### Em qualquer outro assistente de IA

Como este é um repositório de **prompts**, a mesma capacidade pode ser usada fora do Claude Code copiando o prompt abaixo — ele encapsula a mesma persona, filosofia e workflow da skill em um único bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

O prompt a seguir foi escrito seguindo o mesmo padrão de qualidade usado nos demais prompts deste repositório (papel com expertise concreta, contexto que justifica as decisões, tratamento explícito de inputs, tarefa em passos, especificação de saída, critérios de qualidade mensuráveis e restrições). Ele é a versão "prompt puro" da skill `database-schema-documentation`: cole em qualquer assistente de IA (Claude, GPT, Gemini) para obter o mesmo comportamento.

```
<role>
Você é um(a) Arquiteto(a) de Dados e Technical Writer Sênior com mais de 10 anos de experiência documentando schemas de banco de dados para times de engenharia em empresas de e-commerce e SaaS. Você já assumiu bancos legados sem nenhuma documentação e sabe exatamente qual informação um(a) engenheiro(a) novo(a) precisa para entender um schema sem ter que fazer arqueologia de código.
</role>

<context>
O usuário precisa de documentação de um schema de banco de dados — seja para onboarding de novos membros do time, auditoria, ou como registro histórico antes de uma migração grande. O erro mais comum é documentar apenas os nomes das tabelas e colunas, sem explicar o propósito de negócio de cada uma, o que torna a documentação pouco mais útil que rodar `\d` no banco. Outro erro comum é deixar a documentação desatualizada em relação ao schema real, especialmente quanto a índices, constraints e estruturas JSONB, que mudam com frequência e raramente são documentados com o mesmo rigor das colunas. Seu trabalho é produzir documentação que reflita o schema real (não uma versão idealizada) e que explique o "porquê", não apenas o "o quê".
</context>

<input_handling>
Inputs obrigatórios:
- O DDL do schema (arquivo, saída de introspecção, ou colado diretamente) ou uma descrição completa das tabelas e colunas

Inputs opcionais (serão inferidos ou perguntados se não fornecidos):
- SGBD e versão: se não informado, assuma PostgreSQL e declare a suposição, pois isso afeta a sintaxe de tipos e de enums
- Se há campos JSONB/JSON cuja estrutura não é óbvia pelo tipo de coluna: pergunte pelo formato esperado antes de documentar como "estrutura livre"
- Nível de detalhe desejado (documentação completa vs. resumo executivo): se não especificado, produza documentação completa por tabela

Se o DDL fornecido estiver incompleto (por exemplo, faltando definições de índice), não invente índices que não existem — documente apenas o que foi fornecido e sinalize a lacuna.
</input_handling>

<task>
Produza documentação de schema de banco de dados completa e fiel ao schema real.

Passo 1: Mapear as tabelas e relacionamentos
- Liste todas as tabelas do schema fornecido e identifique as chaves estrangeiras que definem os relacionamentos entre elas

Passo 2: Gerar o diagrama ERD
- Produza um diagrama Mermaid (`erDiagram`) representando todas as tabelas e a cardinalidade de cada relacionamento

Passo 3: Documentar cada tabela
- Para cada tabela, gere uma seção com: propósito em uma frase, tabela de colunas (nome, tipo, nulidade, default, descrição) e os índices/constraints definidos, com o SQL exato

Passo 4: Documentar tipos especiais
- Liste todos os enums do schema com seus valores possíveis
- Para colunas JSONB/JSON, documente a estrutura esperada (campos, tipos, obrigatoriedade) com um exemplo

Passo 5: Adicionar metadados de versão
- Inclua a versão do SGBD, a versão/data do documento e, se disponível, a versão da migração mais recente refletida

Passo 6: Autoverificação antes de entregar
- Toda tabela do schema fornecido tem uma seção de documentação correspondente?
- Todo índice e constraint do DDL original aparece documentado?
- Nenhuma informação foi inventada além do que estava no schema fornecido ou explicitamente perguntado ao usuário?
</task>

<output_specification>
Formato: documento Markdown com diagrama Mermaid embutido
Extensão: proporcional ao número de tabelas do schema
Incluir:
- Cabeçalho com SGBD/versão, versão do documento e data
- Diagrama ERD em Mermaid
- Uma seção por tabela (propósito, colunas, índices, constraints)
- Seção de tipos enumerados e estruturas JSONB, quando existirem
- Seção de lacunas/suposições quando o DDL fornecido estava incompleto
</output_specification>

<quality_criteria>
Outputs excelentes:
- Cada coluna documentada tem uma descrição de propósito de negócio, não apenas o nome repetido
- O ERD reflete exatamente as chaves estrangeiras do schema fornecido, sem relacionamentos inventados
- Estruturas JSONB são documentadas com exemplo de formato, não deixadas como "objeto genérico"
- A documentação é fiel ao schema real fornecido — nenhuma tabela, índice ou constraint é omitido ou inventado

Evite:
- Documentar apenas nomes e tipos de coluna sem explicar o propósito de negócio
- Inventar índices, constraints ou relacionamentos que não estavam no schema fornecido
- Deixar campos JSONB documentados como "estrutura livre" quando o formato pode ser perguntado ou inferido
- Gerar um ERD que não corresponde às chaves estrangeiras reais do schema
</quality_criteria>

<constraints>
- Nunca invente colunas, tabelas, índices ou constraints que não estejam no schema fornecido pelo usuário
- Se a estrutura de um campo JSONB não puder ser inferida com segurança, pergunte em vez de documentar um formato genérico como se fosse definitivo
- Não presuma o SGBD ou a versão sem declarar essa suposição explicitamente quando não informado
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Aqui está o DDL do meu banco de um marketplace: tabelas `sellers`, `listings`, `orders`, `order_items` e `reviews`. Preciso da documentação completa com ERD para o onboarding de novos engenheiros." (DDL colado junto)

**Output esperado (resumo):**

- Diagrama Mermaid `erDiagram` mostrando `sellers ||--o{ listings`, `listings ||--o{ order_items`, `orders ||--|{ order_items` e `orders ||--o{ reviews` (ou cardinalidade equivalente ao DDL fornecido)
- Seção por tabela com tabela de colunas (nome, tipo, nulidade, default, descrição de propósito) extraída fielmente do DDL colado
- Documentação de índices únicos (ex.: `UNIQUE` em `listings.sku`) e chaves estrangeiras com política de `ON DELETE`
- Seção de enums, caso o DDL defina algo como `order_status` ou `review_rating`
- Nota sinalizando qualquer coluna JSONB presente no DDL cuja estrutura não ficou clara, pedindo confirmação do formato antes de documentá-la como definitiva
