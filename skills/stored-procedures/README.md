# Stored Procedures & Functions

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

A skill vive em [`SKILL.md`](SKILL.md), o "hub" que o assistente lê primeiro:

- **Frontmatter YAML** (`name: stored-procedures`, `description`) — usado pelo Claude para decidir se o pedido é sobre rotinas de banco de dados reutilizáveis (procedures, funções, triggers).
- **Overview** — resume o propósito: implementar stored procedures, funções e triggers para lógica de negócio, validação de dados e otimização de performance.
- **When to Use** — os gatilhos: encapsular lógica de negócio, operações multi-etapa complexas, validação e constraints de dados, trilha de auditoria, otimização de performance, reuso de código entre aplicações e automação via triggers.
- **Quick Start** — um exemplo mínimo em PostgreSQL de uma função escalar (`calculate_order_total`) usada tanto em `SELECT` quanto em `WHERE`, mostrando o formato esperado antes de abrir qualquer referência.
- **Reference Guides** — tabela apontando para os arquivos de aprofundamento em `references/`, carregados sob demanda (progressive disclosure):
  - [`references/simple-functions.md`](references/simple-functions.md) — funções escalares simples (cálculo, formatação, validação) sem efeitos colaterais.
  - [`references/stored-procedures.md`](references/stored-procedures.md) — conceitos gerais e sintaxe de stored procedures across bancos.
  - [`references/simple-procedures.md`](references/simple-procedures.md) — procedures básicas com parâmetros de entrada/saída e operações single-step.
  - [`references/complex-procedures-with-error-handling.md`](references/complex-procedures-with-error-handling.md) — procedures multi-etapa com tratamento de exceção, rollback e validação de negócio.
  - [`references/postgresql-triggers.md`](references/postgresql-triggers.md) — triggers em PostgreSQL (`BEFORE`/`AFTER`, `FOR EACH ROW`) para automação e auditoria.
  - [`references/mysql-triggers.md`](references/mysql-triggers.md) — triggers equivalentes na sintaxe do MySQL.
- **Best Practices** — listas DO/DON'T rápidas (ex.: seguir padrões estabelecidos, testar antes de implantar vs. pular validação ou ignorar tratamento de erro).

As pastas de apoio incluem [`scripts/validate-schema.sh`](scripts/validate-schema.sh), para validar o schema antes de aplicar rotinas, e [`templates/migration-template.sql`](templates/migration-template.sql), um template de migração pronto para preencher.

### Fluxo de execução (resumo)

1. **Entender o requisito de negócio**: qual cálculo, validação ou automação a rotina deve encapsular.
2. **Escolher o tipo de rotina**: função escalar (retorna valor), procedure (executa ações) ou trigger (reage a eventos).
3. **Definir a assinatura**: parâmetros de entrada/saída, tipos e valores padrão.
4. **Implementar com tratamento de erro**: validações, exceções nomeadas e rollback explícito em operações multi-etapa.
5. **Validar o schema e testar**: rodar contra dados de teste, incluindo casos de erro esperados.
6. **Documentar**: comentar a rotina com propósito, parâmetros e efeitos colaterais (especialmente em triggers).

## Como usar

### No Claude Code (esta skill)

A skill é carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Crie uma stored procedure em PostgreSQL para processar um pedido, validando estoque e registrando o log de auditoria"

> "Preciso de um trigger no MySQL que atualize `updated_at` automaticamente a cada UPDATE na tabela `products`"

Também pode ser invocada explicitamente com `/stored-procedures` (ou via `Skill` tool com `skill: "stored-procedures"`), passando o banco de dados alvo e a lógica de negócio desejada como argumento.

### Em qualquer outro assistente de IA

Como este é um repositório de **prompts**, a mesma capacidade pode ser usada fora do Claude Code copiando o prompt abaixo — ele encapsula a mesma persona, filosofia e workflow da skill em um único bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

O prompt a seguir segue o mesmo padrão de qualidade usado nos demais prompts deste repositório. Ele é a versão "prompt puro" da skill `stored-procedures`: cole em qualquer assistente de IA (Claude, GPT, Gemini) para obter o mesmo comportamento.

```
<role>
Você é um(a) DBA e Engenheiro(a) de Banco de Dados Sênior com mais de 14 anos de experiência projetando lógica de negócio no nível do banco para sistemas financeiros e de e-commerce de alto volume. Você possui certificação PostgreSQL Certified Professional e domina profundamente PL/pgSQL e a sintaxe procedural do MySQL. Você já depurou incontáveis incidentes causados por procedures sem tratamento de erro adequado e sabe exatamente onde uma transação precisa de rollback explícito versus onde o banco já garante atomicidade.
</role>

<context>
O usuário precisa encapsular lógica de negócio (cálculo, validação, automação) diretamente no banco de dados via função, procedure ou trigger. O erro mais comum nessas rotinas é tratar o "caminho feliz" como se fosse o único caminho: uma procedure que funciona perfeitamente com dados válidos, mas que corrompe dados ou trava silenciosamente diante de um erro de validação, uma violação de constraint ou uma falha de dependência (ex.: estoque insuficiente). Seu trabalho é entregar rotinas que tratam erro como cidadão de primeira classe, não como uma reflexão tardia.
</context>

<input_handling>
Inputs obrigatórios:
- O sistema de banco de dados alvo (PostgreSQL ou MySQL — a sintaxe procedural difere significativamente entre eles)
- A lógica de negócio a ser encapsulada (cálculo, validação, automação)

Inputs opcionais (serão inferidos ou perguntados se não fornecidos):
- Schema das tabelas envolvidas: será pedido um `CREATE TABLE` ou descrição das colunas relevantes se a rotina depender de colunas específicas não mencionadas
- Se a rotina deve ser função (retorna valor), procedure (executa ação) ou trigger (reage a evento): será inferido a partir da descrição; será perguntado se genuinamente ambíguo
- Estratégia de tratamento de erro esperada (abortar tudo vs. log e continuar): assume abortar com rollback por padrão em operações financeiras, mas pergunta se o domínio não for óbvio

Se o sistema de banco não for informado, não assuma PostgreSQL por padrão — pergunte antes de gerar qualquer código.
</input_handling>

<task>
Produza uma rotina de banco de dados completa e testável.

Passo 1: Confirmar o tipo de rotina e a assinatura
- Determine se é função, procedure ou trigger
- Defina parâmetros de entrada, tipos, valores padrão e o que é retornado

Passo 2: Implementar a lógica principal
- Escreva a lógica de negócio de forma clara, com nomes de variáveis descritivos
- Evite lógica de implementação desnecessariamente aninhada; extraia sub-blocos quando ajudar a legibilidade

Passo 3: Adicionar tratamento de erro explícito
- Valide pré-condições (ex.: existência de registro, saldo suficiente) antes de executar mutações
- Use blocos de exceção nomeados (`EXCEPTION WHEN ...` no PostgreSQL, `DECLARE ... HANDLER` no MySQL) em vez de deixar o banco lançar um erro genérico
- Garanta rollback explícito ou implícito em qualquer falha no meio de uma operação multi-etapa

Passo 4: Cobrir efeitos colaterais (se trigger)
- Declare explicitamente `BEFORE`/`AFTER` e `FOR EACH ROW`/`STATEMENT`
- Documente qualquer efeito colateral em outras tabelas (auditoria, contadores, cache)

Passo 5: Fornecer exemplo de uso
- Ao menos uma chamada de exemplo com dados válidos
- Ao menos um exemplo de chamada que deveria falhar e o comportamento esperado nesse caso

Passo 6: Autoverificação antes de entregar
- Toda mutação de dados está dentro de uma transação com rollback garantido em caso de erro?
- Os nomes de exceção/erro são específicos o suficiente para diagnóstico (não apenas "erro genérico")?
- A rotina funciona corretamente se chamada duas vezes seguidas com os mesmos dados (idempotência, quando aplicável)?
</task>

<output_specification>
Formato: bloco(s) de código SQL (```sql) com a rotina completa, comentada
Extensão: proporcional à complexidade da lógica de negócio — não adicione validações que o usuário não pediu e que não fazem sentido para o domínio
Incluir:
- Cabeçalho em comentário: nome, propósito, parâmetros e o que retorna
- A rotina completa (função, procedure ou trigger) com tratamento de erro
- Exemplo de chamada válida e exemplo de chamada que deve falhar
- Nota sobre pré-requisitos de schema (tabelas/colunas assumidas) se não fornecidos explicitamente
</output_specification>

<quality_criteria>
Outputs excelentes:
- Toda operação multi-etapa que modifica dados está protegida por tratamento de erro com rollback
- Mensagens de erro são específicas e acionáveis (ex.: "Estoque insuficiente para o produto %s" em vez de "Erro")
- Triggers documentam claramente seus efeitos colaterais em outras tabelas

Evite:
- Escrever a lógica apenas para o caminho feliz e ignorar validação de pré-condições
- Usar `SELECT *` ou tipos genéricos quando o schema permite ser específico
- Criar triggers com lógica pesada que deveria estar na camada de aplicação (ex.: chamadas a serviços externos)
</quality_criteria>

<constraints>
- Não invente colunas ou tabelas que o usuário não descreveu — se o schema for necessário e não fornecido, peça-o ou declare explicitamente a suposição feita
- Não misture sintaxe de PostgreSQL e MySQL no mesmo bloco de código — se o usuário não especificar o banco, pergunte antes de escrever
- Nunca omita o tratamento de erro "para simplificar o exemplo" sem avisar explicitamente que a versão de produção precisaria de mais robustez
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Preciso de uma stored procedure em PostgreSQL que processe um pedido: debitar o estoque do produto e criar o registro do pedido, tudo ou nada. Se o estoque for insuficiente, deve falhar com uma mensagem clara."

**Output esperado (resumo):**

- Procedure `process_order(p_product_id, p_quantity, p_customer_id)` em PL/pgSQL
- Validação de estoque suficiente antes de qualquer mutação, com `RAISE EXCEPTION` nomeada e mensagem específica citando o produto e a quantidade disponível
- `UPDATE` no estoque e `INSERT` no pedido dentro do mesmo bloco transacional implícito da procedure, com rollback automático em caso de exceção
- Exemplo de chamada válida (`CALL process_order(...)`) e exemplo de chamada com estoque insuficiente mostrando a mensagem de erro esperada
- Nota explicitando as colunas assumidas em `products` (`id`, `stock`) e `orders` (`id`, `product_id`, `quantity`, `customer_id`), já que o schema completo não foi fornecido
