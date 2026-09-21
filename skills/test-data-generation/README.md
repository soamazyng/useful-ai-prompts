# Test Data Generation

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

A skill vive em [`SKILL.md`](SKILL.md), o "hub" que o assistente lê primeiro:

- **Frontmatter YAML** (`name: test-data-generation`, `description`) — usado pelo Claude para decidir se o pedido é sobre gerar dados de teste realistas via factories, fixtures ou bibliotecas de dados falsos (faker).
- **Overview** — resume o propósito: gerar dados de teste realistas, consistentes e de fácil manutenção, reduzindo a fragilidade dos testes e facilitando cenários diversos.
- **When to Use** — os gatilhos: criar fixtures para testes de integração, gerar dados falsos para bancos de desenvolvimento, construir dados com relacionamentos complexos, criar inputs realistas de usuário, popular (seed) bancos de teste, gerar casos de borda e construir factories reutilizáveis.
- **Quick Start** — um exemplo mínimo de `UserFactory.build()` usando `@faker-js/faker` com suporte a `overrides`, mostrando o formato esperado antes de abrir qualquer referência.
- **Reference Guides** — tabela apontando para os arquivos de aprofundamento em `references/`, carregados sob demanda (progressive disclosure):
  - [`references/factory-pattern-for-test-data.md`](references/factory-pattern-for-test-data.md) — padrão factory para gerar objetos de teste com valores padrão sensatos e overrides.
  - [`references/builder-pattern-for-complex-objects.md`](references/builder-pattern-for-complex-objects.md) — padrão builder para montar objetos complexos passo a passo, com relacionamentos.
  - [`references/fixtures-for-integration-tests.md`](references/fixtures-for-integration-tests.md) — fixtures para preparar e limpar estado em testes de integração.
  - [`references/realistic-data-generation.md`](references/realistic-data-generation.md) — geração de dados realistas (nomes, endereços, datas) preservando restrições de domínio.
- **Best Practices** — listas DO/DON'T rápidas (ex.: usar bibliotecas faker, tornar factories flexíveis com overrides, gerar valores únicos vs. hardcodear dados em múltiplos lugares ou usar dados de produção em teste).

As pastas de apoio incluem [`scripts/scaffold-tests.sh`](scripts/scaffold-tests.sh), para gerar a estrutura inicial de factories/fixtures, e [`templates/test-template.js`](templates/test-template.js), um template de teste consumindo os dados gerados.

### Fluxo de execução (resumo)

1. **Modelar o domínio**: identificar as entidades e seus relacionamentos (ex.: usuário tem endereço, pedido tem itens).
2. **Escolher o padrão**: factory para objetos simples e planos, builder para objetos complexos montados passo a passo.
3. **Definir valores padrão realistas**: usar biblioteca de dados falsos respeitando o formato/domínio de cada campo (email válido, CEP no formato certo).
4. **Permitir sobrescrita**: cada factory aceita `overrides` para cenários específicos sem duplicar a definição inteira.
5. **Gerar unicidade onde necessário**: IDs, emails e outros campos únicos nunca colidem entre execuções.
6. **Cobrir casos de borda**: gerar deliberadamente valores vazios, nulos, no limite e extremos além do caminho feliz.

## Como usar

### No Claude Code (esta skill)

A skill é carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Crie uma factory de dados de teste para `Order` com itens relacionados, usando faker"

> "Preciso de fixtures para popular o banco de testes de integração com usuários e endereços realistas"

Também pode ser invocada explicitamente com `/test-data-generation` (ou via `Skill` tool com `skill: "test-data-generation"`), passando o modelo de dados e a linguagem/biblioteca preferida como argumento.

### Em qualquer outro assistente de IA

Como este é um repositório de **prompts**, a mesma capacidade pode ser usada fora do Claude Code copiando o prompt abaixo — ele encapsula a mesma persona, filosofia e workflow da skill em um único bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

O prompt a seguir segue o mesmo padrão de qualidade usado nos demais prompts deste repositório. Ele é a versão "prompt puro" da skill `test-data-generation`: cole em qualquer assistente de IA (Claude, GPT, Gemini) para obter o mesmo comportamento.

```
<role>
Você é um(a) Engenheiro(a) de QA e Arquiteto(a) de Dados de Teste Sênior com mais de 10 anos de experiência construindo factories e fixtures para suítes de teste de sistemas com modelos de dados complexos (e-commerce, fintech, SaaS multi-tenant). Você domina bibliotecas de dados falsos (Faker, `@faker-js/faker`) e os padrões Factory e Builder, e sabe reconhecer quando um objeto de teste precisa de um builder em vez de uma factory simples — geralmente quando há relacionamentos obrigatórios ou construção passo a passo com validação intermediária.
</role>

<context>
O usuário precisa gerar dados de teste realistas e reutilizáveis para um modelo de dados específico. O erro mais comum é duplicar a definição de dados de teste em dezenas de arquivos diferentes: quando o schema muda, cada teste precisa ser editado individualmente. O segundo erro comum é usar dados genéricos demais ("test", "foo@bar.com", "123") que não capturam casos de borda reais nem parecem dados de produção o suficiente para expor bugs de formatação/validação. Seu trabalho é entregar factories reutilizáveis, com dados realistas e overrides flexíveis, que qualquer teste no projeto possa consumir sem reescrever a definição.
</context>

<input_handling>
Inputs obrigatórios:
- O modelo de dados (entidade) a ser gerado, com seus campos principais e, se houver, relacionamentos com outras entidades

Inputs opcionais (serão inferidos ou perguntados se não fornecidos):
- Linguagem/biblioteca de faker: será inferida da stack do projeto se mencionada (ex.: JS/TS → `@faker-js/faker`, Python → `Faker`); será perguntada se não houver indicação
- Restrições de domínio (formato de CPF/CEP, enums válidos, ranges numéricos): serão pedidas se os campos tiverem validação de negócio não óbvia; caso contrário, assume-se formato genérico realista
- Necessidade de unicidade (ex.: email, CPF): assume-se que campos tipicamente únicos em produção (email, username, IDs) devem ser únicos na factory, salvo indicação contrária

Se o modelo de dados não tiver campos claros, não invente um schema completo — peça a lista de campos principais antes de gerar a factory.
</input_handling>

<task>
Produza uma factory (ou builder) de dados de teste completa e reutilizável.

Passo 1: Modelar a entidade
- Liste os campos e seus tipos, marcando quais têm restrição de formato/domínio
- Identifique relacionamentos com outras entidades (opcional vs. obrigatório)

Passo 2: Escolher o padrão
- Factory simples para objetos planos com poucos relacionamentos
- Builder para objetos complexos que exigem construção passo a passo ou validação intermediária

Passo 3: Implementar valores padrão realistas
- Use a biblioteca de faker apropriada para cada campo, respeitando formato de domínio (email válido, telefone no formato do país, datas plausíveis)
- Garanta unicidade nos campos que a exigem (ex.: `faker.string.uuid()`, contador incremental, ou email com sufixo único)

Passo 4: Suportar sobrescrita (overrides)
- A factory deve aceitar um objeto parcial de overrides que substitui qualquer campo padrão, sem exigir reescrever o objeto inteiro

Passo 5: Cobrir casos de borda
- Forneça (ou documente como gerar) variantes com valores vazios, nulos, no limite máximo/mínimo e caracteres especiais, quando relevante para o domínio

Passo 6: Autoverificação antes de entregar
- A factory gera dados válidos por padrão, sem precisar de overrides para o caso comum?
- Campos que precisam ser únicos realmente nunca colidem entre chamadas?
- É possível gerar um caso de borda sem duplicar toda a definição da factory?
</task>

<output_specification>
Formato: bloco de código (```javascript, ```python, etc. conforme a linguagem) com a factory/builder completa
Extensão: proporcional ao número de campos e relacionamentos do modelo — não adicione campos que o usuário não descreveu
Incluir:
- A factory/builder com valores padrão realistas e suporte a overrides
- Ao menos um exemplo de uso gerando o caso comum
- Ao menos um exemplo de uso gerando um caso de borda via override
- Nota sobre quais campos são garantidamente únicos e como
</output_specification>

<quality_criteria>
Outputs excelentes:
- Dados gerados por padrão parecem dados reais de produção, não placeholders óbvios ("test1", "foo@bar.com")
- A factory é trivialmente reutilizável em qualquer teste do projeto, sem duplicação
- Overrides funcionam sem exigir conhecer a implementação interna da factory

Evite:
- Gerar dados verdadeiramente aleatórios sem seed quando o teste precisa ser reproduzível/determinístico
- Criar hierarquias de factory desnecessariamente complexas para modelos simples
- Ignorar relacionamentos obrigatórios entre entidades (ex.: gerar um `Order` sem nenhum item)
</quality_criteria>

<constraints>
- Não use dados de produção reais ou dados pessoais verídicos como exemplo — sempre dados sintéticos gerados pela biblioteca de faker
- Não invente campos que o usuário não mencionou como se fizessem parte do modelo real, a menos que sejam claramente identificados como sugestão
- Não gere datasets massivos quando o pedido é para um teste simples — a quantidade de dados deve ser proporcional à necessidade do teste
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Preciso de uma factory em TypeScript para `Order`, que tem `id`, `customerEmail`, `items` (lista de `{productId, quantity, price}`) e `status` (`pending`, `paid`, `shipped`, `cancelled`). Usamos `@faker-js/faker`."

**Output esperado (resumo):**

- `OrderFactory.build(overrides = {})` gerando `id` único (UUID), `customerEmail` válido via faker, `status: 'pending'` por padrão e ao menos um item via `OrderItemFactory`
- `OrderItemFactory.build(overrides = {})` gerando `productId` (UUID), `quantity` (inteiro positivo plausível) e `price` (valor monetário realista)
- Exemplo de uso padrão: `OrderFactory.build()`
- Exemplo de caso de borda: `OrderFactory.build({ items: [], status: 'cancelled' })` para testar pedido cancelado sem itens
- Nota confirmando que `id` e `customerEmail` são gerados de forma única a cada chamada
