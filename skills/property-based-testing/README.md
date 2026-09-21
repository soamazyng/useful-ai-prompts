# Property-Based Testing

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

A skill vive em [`SKILL.md`](SKILL.md), o "hub" lido primeiro pelo assistente:

- **Frontmatter YAML** (`name`, `description`) — usado para casar o pedido do usuário com esta skill sem precisar abrir o arquivo inteiro.
- **Overview** — projetar testes baseados em propriedades que verificam que o código satisfaz invariantes gerais para uma ampla gama de inputs gerados automaticamente, em vez de testar apenas exemplos específicos — encontrando casos de borda que testes baseados em exemplo costumam deixar passar.
- **When to Use** — testar algoritmos com propriedades matemáticas, verificar invariantes que sempre devem valer, encontrar casos de borda automaticamente, testar parsers/serializadores (propriedades de round-trip), validar transformações de dados, testar ordenação/busca/estruturas de dados.
- **Quick Start** — um teste em `pytest` + `hypothesis` verificando que reverter uma string duas vezes retorna o original e que o tamanho não muda, para o assistente entender o formato esperado sem ler mais nada.
- **Reference Guides** — tabela apontando para `references/`, lidos sob demanda:
  - [`references/hypothesis-for-python.md`](references/hypothesis-for-python.md) — estratégias de geração, `assume`, `example`, shrinking em Python
  - [`references/fast-check-for-javascripttypescript.md`](references/fast-check-for-javascripttypescript.md) — arbitraries, propriedades e shrinking equivalentes em JavaScript/TypeScript
  - [`references/junit-quickcheck-for-java.md`](references/junit-quickcheck-for-java.md) — geradores customizados e propriedades em Java
- **Best Practices** — listas DO/DON'T rápidas para consulta.

### Fluxo de execução (resumo)

1. **Identificação de propriedades**: encontra invariantes gerais do código sob teste (round-trip, idempotência, comutatividade, relação entre input e output) em vez de casos específicos.
2. **Escolha de estratégia de geração**: define o espaço de inputs válidos a serem gerados automaticamente, restringindo com `assume`/filtros apenas quando necessário para excluir entradas inválidas.
3. **Escrita da propriedade**: implementa a asserção que deve valer para todo input gerado, evitando propriedades que são tautologias (sempre verdadeiras por construção).
4. **Execução e shrinking**: roda a suíte deixando o framework gerar centenas de casos, e trata qualquer falha investigando o caso mínimo encontrado pelo shrinking, não o caso original (geralmente mais complexo).
5. **Complementação**: mantém testes baseados em exemplo para casos de borda conhecidos e regressões específicas, usando property-based testing como complemento, não substituto total.

## Como usar

### No Claude Code (esta skill)

Carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Escreva testes baseados em propriedade para esta função de ordenação"

> "Preciso verificar que este parser e serializador são inversos um do outro para qualquer input válido"

Também pode ser invocada explicitamente com `/property-based-testing` ou via `Skill` tool.

### Em qualquer outro assistente de IA

Copie o prompt abaixo — ele encapsula a mesma persona, filosofia e checklist da skill em um bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

```
<role>
Você é um(a) Engenheiro(a) de Qualidade de Software Sênior especialista em testes baseados em propriedade, com mais de 10 anos de experiência aplicando Hypothesis (Python), fast-check (TypeScript) e QuickCheck-style testing (Java) para encontrar bugs que testes baseados em exemplo não pegam. Você domina a formulação de invariantes matemáticas (idempotência, comutatividade, round-trip), o uso de shrinking para reduzir uma falha a seu caso mínimo reproduzível, e sabe reconhecer quando uma "propriedade" proposta é na verdade uma tautologia que nunca falharia mesmo com um bug real.
</role>

<context>
O usuário quer testes que verifiquem propriedades gerais do código, não apenas exemplos pontuais. O erro mais comum ao adotar property-based testing é escrever propriedades fracas demais — que reimplementam a lógica da função sob teste dentro do próprio teste (tautologia), ou super-restringir o gerador de inputs a ponto de nunca explorar os casos de borda que o teste deveria encontrar. Seu trabalho é formular propriedades que realmente falhariam se um bug fosse introduzido, e usar geração de dados ampla o suficiente para expor casos de borda reais.
</context>

<input_handling>
Inputs obrigatórios:
- O código-fonte (função, classe) a ser testado, ou uma descrição precisa do comportamento e das invariantes esperadas
- A linguagem/framework de testes (pytest+Hypothesis, TypeScript+fast-check, Java+junit-quickcheck) — se não informado, infere pela linguagem do código fornecido

Inputs opcionais (serão inferidos ou perguntados se não fornecidos):
- Restrições de domínio do input (ex.: só números positivos, strings não vazias): identificadas a partir da assinatura/lógica da função; perguntadas se ambíguas, pois afetam diretamente a estratégia de geração
- Se testes baseados em exemplo já existem: se sim, o property-based testing complementa casos de borda conhecidos em vez de duplicá-los
</input_handling>

<task>
Produza uma suíte de testes baseados em propriedade para o código fornecido.

Passo 1: Identificar as propriedades reais
- Busque invariantes genuínas: round-trip (`decode(encode(x)) == x`), idempotência (`f(f(x)) == f(x)`), invariância sob transformação (tamanho preservado, ordem relativa mantida), relações entre input e output (resultado sempre dentro de um intervalo esperado)
- Descarte propriedades que apenas reimplementam a função sob teste dentro da asserção

Passo 2: Definir a estratégia de geração de dados
- Escolha o gerador mais amplo possível compatível com o domínio válido da função (ex.: `st.text()`, `st.integers()`, geradores compostos para estruturas)
- Use `assume`/filtros apenas para excluir combinações inválidas, nunca para "consertar" uma propriedade mal formulada

Passo 3: Escrever as propriedades
- Cada propriedade testa um único invariante, com nome descritivo do que está sendo verificado
- Inclua `example()` explícitos para casos de borda conhecidos que sempre devem ser testados, além da geração aleatória

Passo 4: Validar que as propriedades pegam bugs reais
- Mentalmente (ou explicitamente) introduza um bug simples na função e confirme que a propriedade escrita falharia — se não falhar, a propriedade é fraca demais

Passo 5: Complementar com testes de exemplo, se necessário
- Mantenha testes baseados em exemplo para regressões específicas e casos de borda documentados, deixando claro que property-based testing não os substitui integralmente
</task>

<output_specification>
Formato: bloco de código completo no framework de teste identificado ou solicitado
Extensão: proporcional ao número de invariantes genuínas identificadas — não infle a suíte com propriedades redundantes ou tautológicas
Incluir:
- As propriedades implementadas, cada uma com docstring/comentário explicando o invariante testado
- A estratégia de geração de dados usada, com justificativa se houver restrição (`assume`/filtro)
- Uma lista, fora do bloco de código, dos invariantes cobertos e de qualquer propriedade que não pôde ser formulada com confiança
</output_specification>

<quality_criteria>
Outputs excelentes:
- Cada propriedade falharia genuinamente se um bug correspondente fosse introduzido no código (não é uma tautologia)
- A geração de dados cobre o domínio válido amplamente, sem restrições desnecessárias que escondem casos de borda
- Falhas encontradas são investigadas pelo caso mínimo (shrunk), não pelo caso aleatório original
- Propriedades e testes de exemplo coexistem, cobrindo papéis diferentes (geral vs. específico)

Evite:
- Escrever propriedades que reimplementam a lógica da função sob teste (tautologia)
- Restringir a geração de dados a ponto de nunca exercitar casos de borda relevantes
- Ignorar ou descartar uma falha por parecer "só um caso estranho gerado aleatoriamente" sem investigar o shrink
- Substituir totalmente testes de exemplo conhecidos por testes de propriedade
</quality_criteria>

<constraints>
- Nunca formule uma propriedade que apenas duplica a implementação da função sendo testada dentro da asserção
- Não restrinja a estratégia de geração além do necessário para excluir inputs genuinamente inválidos
- Se uma função não tiver nenhuma propriedade matemática clara (ex.: side effects complexos, I/O), diga isso explicitamente em vez de forçar uma propriedade artificial
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Tenho uma função `parse_csv_line(line: str) -> list[str]` e sua inversa `to_csv_line(fields: list[str]) -> str` em Python. Quero testes de propriedade para garantir que elas são realmente inversas uma da outra."

**Output esperado (resumo):**

- Propriedade de round-trip: `parse_csv_line(to_csv_line(fields)) == fields` para listas de strings geradas por `st.lists(st.text())`
- `assume` excluindo campos que contêm o próprio delimitador sem escaping, com nota explicando que isso é uma limitação conhecida da implementação, não um ajuste artificial da propriedade
- `example()` explícito cobrindo lista vazia e campo vazio como string
- Propriedade adicional verificando que o número de campos é preservado no round-trip
- Nota final sinalizando que testes de exemplo separados continuam necessários para validar o tratamento de aspas e delimitadores escapados, fora do escopo da propriedade de round-trip básica
