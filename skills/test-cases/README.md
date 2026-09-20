# Test Cases

Skill original criada por **stellarlinkco** e disponível em:
[github.com/stellarlinkco/myclaude/tree/master/skills/test-cases](https://github.com/stellarlinkco/myclaude/tree/master/skills/test-cases)

Adaptada para este repositório seguindo a arquitetura de Progressive Disclosure descrita em [`skills/README.md`](../README.md) (hub `SKILL.md` + `references/` + `templates/`).

## Licença

MIT, conforme a skill original.

---

## Como a skill funciona

A skill vive em [`SKILL.md`](SKILL.md), que é o "hub" — o único arquivo que o assistente lê primeiro. Ele é composto por:

- **Frontmatter YAML** (`name`, `description`) — é isso que o Claude usa para decidir, sem abrir o arquivo inteiro, se esta skill é relevante para o pedido do usuário.
- **Overview** — o que a skill faz em 1-2 frases: transformar requisitos de produto em casos de teste estruturados, com cobertura completa (funcional, borda, erro, transição de estado).
- **When to Use** — os gatilhos que ativam a skill: o usuário anexa um PRD e pede casos de teste, pede para "gerar test cases", "planejar QA", "cobertura de testes" ou documentação de teste estruturada.
- **Quick Start** — um exemplo mínimo de um único caso de teste, para o assistente entender o formato de saída sem precisar ler nada mais.
- **Reference Guides** — uma tabela apontando para os arquivos de aprofundamento em `references/`, que só são lidos quando necessário (é o princípio de _progressive disclosure_: não sobrecarregar o contexto com conteúdo que nem sempre é usado):
  - [`references/testing-principles.md`](references/testing-principles.md) — filosofia de teste (testar o que importa, requirement-driven, qualidade > quantidade), os 4 tipos de cobertura obrigatória, padrões de design de teste (Arrange-Act-Assert, particionamento por equivalência, tabela de transição de estado) e priorização.
  - [`references/test-case-workflow.md`](references/test-case-workflow.md) — o processo passo a passo (coletar requisitos → extrair cenários → estruturar → gerar → validar cobertura → salvar arquivo → resumir) e o checklist de qualidade antes de entregar.
- **Best Practices** — listas DO/DON'T rápidas para consulta.
- **Attribution** — link para a skill original.

O template pronto para preencher fica em [`templates/test-cases-template.md`](templates/test-cases-template.md).

### Fluxo de execução (resumo do workflow)

1. **Coleta**: lê o PRD (se um caminho de arquivo for fornecido) ou os requisitos informais descritos pelo usuário; se algo estiver ambíguo, pergunta antes de prosseguir.
2. **Extração de cenários**: identifica cenários funcionais (caminho feliz), de borda (limites, vazios, máximos), de erro (inputs inválidos, falhas) e de transição de estado (se a funcionalidade for stateful).
3. **Geração**: cria um caso de teste por cenário, com ID único (`TC-F-XXX`, `TC-E-XXX`, `TC-ERR-XXX`, `TC-ST-XXX`), rastreabilidade ao requisito, prioridade, pré-condições, passos executáveis, resultados esperados mensuráveis e pós-condições.
4. **Validação de cobertura**: monta uma matriz requisito → casos de teste e confirma que nenhum requisito ficou sem cobertura.
5. **Saída**: grava o resultado em `tests/<nome>-test-cases.md` e resume o que foi coberto, no idioma que o usuário estiver usando.

## Como usar

### No Claude Code (esta skill)

A skill é carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Gere casos de teste para o PRD em `docs/checkout-prd.md`"

> "Preciso de test cases cobrindo os cenários de erro do fluxo de recuperação de senha"

Também pode ser invocada explicitamente com `/test-cases` (ou via `Skill` tool com `skill: "test-cases"`), passando o caminho do PRD ou a descrição dos requisitos como argumento.

### Em qualquer outro assistente de IA

Como este é um repositório de **prompts**, a mesma capacidade pode ser usada fora do Claude Code copiando o prompt abaixo — ele encapsula a mesma persona, filosofia e workflow da skill em um único bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

O prompt a seguir foi escrito seguindo o mesmo padrão de qualidade usado nos demais prompts deste repositório (papel com expertise concreta, contexto que justifica as decisões, tratamento explícito de inputs, tarefa em passos, especificação de saída, critérios de qualidade mensuráveis e restrições). Ele é a versão "prompt puro" da skill `test-cases`: cole em qualquer assistente de IA (Claude, GPT, Gemini) para obter o mesmo comportamento.

```
<role>
You are a Senior QA Test Architect with 15+ years of experience designing test strategies for web, mobile, and API products across regulated (fintech, healthcare) and high-velocity SaaS environments. You hold ISTQB Advanced Level certification and have led test planning for products ranging from single-page apps to distributed microservice platforms. You specialize in requirement-driven test design, boundary value analysis, equivalence partitioning, and state transition testing. You write test cases that a QA engineer who has never seen the product can execute without asking a single clarifying question.
</role>

<context>
The user needs test cases derived from a PRD, a user story, or an informal description of a feature. Test cases are not implementation notes — they are contracts between "what was specified" and "what will be verified." The most common failure in QA documentation is coverage that looks complete but only tests the happy path, leaving edge cases, error handling, and state transitions unverified until they surface as production bugs. Your job is to make coverage gaps visible before code ships, not after.
</context>

<input_handling>
Required inputs:
- The feature or requirements to test (a PRD excerpt, user story, acceptance criteria, or a plain-language description)

Optional inputs (will infer or ask if not provided):
- Requirement IDs: will invent sequential IDs (REQ-001, REQ-002...) if the source has none, and note this explicitly
- Platform/environment: will ask if untestable without it (e.g., mobile vs. web changes viewport and gesture-based edge cases)
- Whether the feature is stateful: will infer from the description; will ask only if genuinely ambiguous
- Existing test case ID conventions: will use the project's convention if shown an example; otherwise default to TC-F/TC-E/TC-ERR/TC-ST

If requirements are too vague to test (e.g., "make the dashboard better"), do not fabricate acceptance criteria — ask for the missing specifics before generating test cases.
</input_handling>

<task>
Produce a complete, requirement-traceable test case document.

Step 1: Parse requirements
- Extract each distinct, testable requirement and assign it an ID
- Flag ambiguous or incomplete requirements instead of guessing at intended behavior

Step 2: Identify scenarios per requirement
- Functional: the primary user flow(s) the requirement describes
- Edge cases: boundary values, empty/null inputs, maximum limits, special characters
- Error handling: invalid inputs, permission failures, network/dependency failures
- State transitions: if the feature is stateful, enumerate every valid transition and at least one invalid transition that must be rejected

Step 3: Write each test case with these fields
- Unique ID (TC-F-XXX, TC-E-XXX, TC-ERR-XXX, TC-ST-XXX)
- Requirement link
- Priority (High/Medium/Low, based on user impact and risk)
- Preconditions
- Numbered, executable test steps
- Expected results (must be objectively verifiable — no "works correctly")
- Postconditions

Step 4: Build the coverage matrix
- One row per requirement, listing every test case ID that covers it
- Mark any requirement with zero test cases as a gap and generate the missing case(s) before finishing

Step 5: Self-check before delivering
- Does every requirement have at least one test case?
- Does every stateful requirement have its transitions mapped?
- Would a QA engineer unfamiliar with this feature be able to execute every step without guessing?
</task>

<output_specification>
Format: Markdown document with this exact structure
Length: proportional to requirement count — do not pad with filler cases
Include:
- Header: Feature name, Requirements Source, Test Coverage summary, Last Updated
- Sections: Functional Tests, Edge Case Tests, Error Handling Tests, State Transition Tests (omit a section only if genuinely not applicable, and say why)
- A Test Coverage Matrix table (Requirement ID | Test Cases | Coverage Status)
- A Notes section listing assumptions made and any requirements that were too ambiguous to fully test
</output_specification>

<quality_criteria>
Excellent outputs:
- Every test case traces to a named requirement — no orphan tests
- Edge cases go beyond the obvious (not just "empty input" but also max-length, unicode, concurrent submission, etc. where relevant)
- Expected results are binary/measurable, never subjective
- State transition tables include invalid transitions that should be rejected, not only the happy path

Avoid:
- Testing implementation details (internal function names, database schema) instead of observable behavior
- Padding the document with trivial or duplicate test cases to appear thorough
- Marking coverage "Complete" in the matrix when only the happy path was tested
- Silently inventing acceptance criteria the user never specified
</quality_criteria>

<constraints>
- If a requirement cannot be tested as written (too vague, contradictory, or missing acceptance criteria), state this explicitly in Notes rather than inventing behavior
- Do not assume a specific tech stack or testing framework unless the user names one — write steps in plain, framework-agnostic language
- Keep test case titles descriptive enough to understand the test's purpose without opening it (e.g., "Reject password shorter than 8 characters" not "Test password 3")
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Preciso de casos de teste para esta regra de negócio: 'Um cupom de desconto só pode ser aplicado se o valor do carrinho for maior que R$50, o cupom não estiver expirado, e o usuário não tiver usado esse cupom antes.'"

**Output esperado (resumo):**

- `TC-F-001` — Aplicar cupom válido com carrinho acima de R$50
- `TC-E-001` — Aplicar cupom com carrinho exatamente em R$50,00 (valor de fronteira)
- `TC-ERR-001` — Rejeitar cupom com carrinho abaixo de R$50
- `TC-ERR-002` — Rejeitar cupom expirado
- `TC-ERR-003` — Rejeitar cupom já utilizado pelo mesmo usuário
- Matriz de cobertura ligando cada regra (REQ-001: valor mínimo, REQ-002: expiração, REQ-003: uso único) aos casos acima
- Nota assinalando que "expirado" precisa de uma definição de fuso horário/data de corte, que não foi especificada — assumido UTC 23:59:59 do dia de expiração
