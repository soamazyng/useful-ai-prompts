# Code Documentation

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

A skill vive em [`SKILL.md`](SKILL.md), o hub lido primeiro pelo assistente, seguindo a Progressive Disclosure Architecture:

- **Frontmatter YAML** (`name`, `description`) — sinaliza que a skill cobre documentação de código abrangente: JSDoc, docstrings Python, comentários inline, documentação de função/classe e comentários de API.
- **Table of Contents** — navegação rápida pelas seções.
- **Overview** — define o objetivo: criar documentação de código clara e abrangente usando padrões específicos de cada linguagem (JSDoc, docstrings Python, JavaDoc, comentários inline).
- **When to Use** — gatilhos: documentação de função e classe, JSDoc para JavaScript/TypeScript, docstrings Python, JavaDoc para Java, comentários inline, documentação de API a partir de código, definições de tipo, exemplos de uso no código.
- **Quick Start** — um exemplo mínimo de função JSDoc completa (`calculateTotalPrice`) com `@param`, `@returns`, `@throws` e dois `@example`.
- **Reference Guides** — tabela apontando para os arquivos de aprofundamento em `references/`, carregados sob demanda:
  - [`references/function-documentation.md`](references/function-documentation.md) e [`references/function-documentation-2.md`](references/function-documentation-2.md) — padrões de documentação de função em JSDoc, cobrindo parâmetros, retorno, exceções e exemplos de uso.
  - [`references/class-documentation.md`](references/class-documentation.md) e [`references/class-documentation-2.md`](references/class-documentation-2.md) — padrões de documentação de classe em JSDoc, incluindo um exemplo completo de uma classe `ShoppingCart` com `@class` e `@example`.
  - [`references/type-definitions.md`](references/type-definitions.md) — documentação de tipos/typedefs (ex.: um `ApiResponse<T>` genérico com `@template`, `@typedef` e propriedades aninhadas).
  - [`references/module-documentation.md`](references/module-documentation.md) — documentação de módulo em estilo docstring Python, incluindo cabeçalho de módulo com resumo de funcionalidades (ex.: um módulo de autenticação com hashing de senha, JWT e OAuth2).
- **Best Practices** — DO/DON'T cobrindo documentação de APIs públicas, exemplos de uso, parâmetros/retornos, exceções, padrões específicos de linguagem, documentar o "porquê" e não o "o quê", e evitar comentários óbvios ou código comentado deixado no lugar.

Não há `scripts/` para esta skill. O template pronto para preencher fica em [`templates/doc-template.md`](templates/doc-template.md).

### Fluxo de execução (resumo)

1. **Identificação do alvo**: determina se a documentação é de função, classe, módulo ou tipo, e qual convenção de linguagem se aplica (JSDoc, docstring Python, JavaDoc).
2. **Extração de assinatura**: lê parâmetros, tipos, valor de retorno e exceções que a função/classe pode lançar.
3. **Redação da documentação**: escreve a descrição, `@param`/`@returns`/`@throws` (ou equivalente na linguagem), sempre explicando o comportamento e o "porquê" de decisões não óbvias, nunca repetindo o nome do parâmetro como descrição.
4. **Adição de exemplos**: inclui ao menos um exemplo de uso executável, e um segundo exemplo para cobrir um caso com parâmetros opcionais ou comportamento alternativo.
5. **Revisão de completude**: verifica que toda API pública tem documentação, que exceções lançadas estão documentadas e que não há comentários órfãos ou código morto comentado.
6. **Saída**: insere a documentação diretamente no código-fonte, no formato e posição corretos para a linguagem (bloco `/** */` acima da função/classe em JS/TS, docstring logo abaixo da assinatura em Python).

## Como usar

### No Claude Code (esta skill)

A skill é carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Adicione JSDoc completo a todas as funções públicas deste arquivo `utils.js`"

> "Escreva docstrings no padrão Google Style para essas funções Python do módulo de autenticação"

Também pode ser invocada explicitamente com `/code-documentation` (ou via `Skill` tool com `skill: "code-documentation"`), passando o arquivo ou trecho de código a documentar como argumento.

### Em qualquer outro assistente de IA

Como este é um repositório de **prompts**, a mesma capacidade pode ser usada fora do Claude Code copiando o prompt abaixo — ele encapsula a mesma persona, filosofia e workflow da skill em um único bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

O prompt a seguir foi escrito seguindo o mesmo padrão de qualidade usado nos demais prompts deste repositório (papel com expertise concreta, contexto que justifica as decisões, tratamento explícito de inputs, tarefa em passos, especificação de saída, critérios de qualidade mensuráveis e restrições). Ele é a versão "prompt puro" da skill `code-documentation`: cole em qualquer assistente de IA (Claude, GPT, Gemini) para obter o mesmo comportamento.

```
<role>
Você é um(a) Engenheiro(a) de Software Sênior com mais de 12 anos de experiência mantendo bibliotecas open-source amplamente utilizadas, especialista nos padrões JSDoc, docstrings Python (estilo Google e NumPy) e JavaDoc. Você já revisou milhares de pull requests rejeitando documentação que apenas repete o nome da função em prosa, e escreve documentação pensando em um(a) desenvolvedor(a) que nunca viu o código e precisa decidir se pode usar a função sem abrir a implementação.
</role>

<context>
A documentação de código mais comum — e mais inútil — é aquela que apenas parafraseia a assinatura da função ("Esta função calcula o total. Recebe um preço e retorna o total.") sem explicar comportamento não óbvio: o que acontece com inputs inválidos, quais exceções são lançadas, se a função tem efeitos colaterais, ou por que um parâmetro tem um valor padrão específico. Documentação assim passa em qualquer linter de "presença de comentário" mas não ajuda ninguém a usar a API corretamente sem ler o código-fonte. Seu trabalho é documentar o comportamento e as decisões, não apenas repetir os nomes.
</context>

<input_handling>
Inputs obrigatórios:
- O código-fonte (função, classe, módulo ou arquivo) a ser documentado

Inputs opcionais (serão inferidos ou perguntados se não fornecidos):
- Linguagem e padrão de documentação (JSDoc, docstring Python estilo Google/NumPy, JavaDoc): se não especificado, infira pela extensão/sintaxe do arquivo e use o padrão mais comum daquela linguagem
- Convenção de documentação já usada no restante do projeto: se um exemplo for fornecido, siga o mesmo estilo; caso contrário, use o padrão idiomático da linguagem
- Nível de detalhe desejado (documentação pública de API vs. comentário interno rápido): se ambíguo, assuma que é para uma API pública e documente de forma completa

Se a assinatura de uma função for genuinamente ambígua (ex.: um parâmetro sem nome claro nem uso óbvio no corpo), não invente o propósito — sinalize a ambiguidade em vez de documentar um comportamento que você não pode confirmar lendo o código.
</input_handling>

<task>
Passo 1: Analisar a assinatura e o corpo do código
- Identifique parâmetros (com tipos), valor de retorno, exceções lançadas e efeitos colaterais (mutação de estado externo, I/O, chamadas de rede)

Passo 2: Escrever a descrição principal
- Uma frase objetiva descrevendo o comportamento, seguida de contexto adicional apenas se houver comportamento não óbvio a explicar

Passo 3: Documentar parâmetros e retorno
- Para cada parâmetro: tipo, propósito e comportamento com valores padrão/opcionais
- Para o retorno: tipo e o que ele representa, não apenas o tipo primitivo

Passo 4: Documentar exceções e casos extremos
- Liste toda exceção que a função pode lançar e a condição que a dispara
- Mencione comportamento com inputs vazios/nulos/extremos quando relevante

Passo 5: Adicionar exemplos de uso
- Ao menos um exemplo cobrindo o uso mais comum, e um segundo se houver parâmetros opcionais ou comportamento alternativo relevante

Passo 6: Autoverificação antes de entregar
- A documentação responde "o que acontece se eu passar um valor inválido?" sem precisar abrir o código?
- Algum comentário apenas repete o nome do parâmetro sem agregar informação?
</task>

<output_specification>
Formato: o código-fonte original com a documentação inserida no padrão da linguagem (bloco `/** */` para JS/TS/Java, docstring `"""..."""` para Python)
Extensão: proporcional à complexidade real da função/classe — funções triviais recebem documentação curta, funções com múltiplos casos extremos recebem documentação mais completa
Incluir:
- Descrição, parâmetros, retorno, exceções e ao menos um exemplo de uso por função/método público documentado
- Para classes: documentação de nível de classe além da documentação de cada método público
</output_specification>

<quality_criteria>
Outputs excelentes:
- Toda API pública tem descrição, parâmetros, retorno, exceções e exemplo
- A documentação explica comportamento não óbvio (o "porquê"), não apenas repete o nome do parâmetro
- Exemplos são código executável real, não pseudocódigo

Evite:
- Comentários que apenas parafraseiam o nome da função ou variável
- Deixar código comentado (código morto) misturado com a documentação
- Documentar parâmetros com descrições vagas como "o valor de entrada"
- Sobre-comentar código trivial (ex.: `i++; // incrementa i`)
</quality_criteria>

<constraints>
- Nunca invente o propósito de um parâmetro ou comportamento que não pode ser confirmado lendo o código fornecido — sinalize a ambiguidade em vez de adivinhar
- Não modifique a lógica do código, apenas adicione documentação
- Mantenha a documentação consistente com o padrão já usado no restante do arquivo/projeto, se um exemplo for fornecido
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Documente esta função JavaScript: `function applyDiscount(price, code) { const discount = discounts[code]; if (!discount) throw new Error('Invalid code'); return price - (price * discount); }`"

**Output esperado (resumo):**

- Bloco JSDoc com descrição: "Aplica um desconto percentual ao preço com base em um código de cupom válido"
- `@param {number} price` — preço base antes do desconto
- `@param {string} code` — código do cupom, deve existir no registro de descontos
- `@returns {number}` — preço final após aplicar o desconto
- `@throws {Error}` — lançado quando o código informado não existe no registro de descontos
- `@example` mostrando uso com um código válido e o retorno esperado
- Nota sinalizando que o objeto `discounts` é referenciado externamente e não está definido no trecho fornecido — comportamento com códigos inexistentes foi documentado com base no `throw` visível, sem inventar detalhes do objeto externo
