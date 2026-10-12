# E2E Testing Automation

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

A skill vive em [`SKILL.md`](SKILL.md), o "hub" que o assistente lê primeiro. Ele é composto por:

- **Frontmatter YAML** (`name`, `description`) — usado pelo Claude para decidir se a skill é relevante: construir testes end-to-end automatizados que simulam interações reais de usuário em toda a stack da aplicação (E2E, Selenium, Cypress, Playwright, automação de browser).
- **Overview** — resume o propósito: testes E2E validam jornadas completas de usuário, da UI através de todos os sistemas de backend, garantindo que toda a stack funcione corretamente da perspectiva de quem usa o produto, simulando cliques, digitação, navegação e envio de formulários reais.
- **When to Use** — os gatilhos: testar jornadas críticas de usuário (cadastro, checkout, login), validar fluxos multi-etapa, testar em diferentes browsers e dispositivos, testes de regressão para mudanças de UI, verificar integração frontend-backend, testar interações reais de usuário e smoke testing de deploys.
- **Quick Start** — um teste mínimo em Playwright cobrindo um fluxo de checkout de e-commerce, para o assistente entender o formato de teste esperado antes de aprofundar.
- **Reference Guides** — tabela apontando para os arquivos de aprofundamento em `references/`, carregados sob demanda (progressive disclosure):
  - [`references/playwright-e2e-tests.md`](references/playwright-e2e-tests.md) — testes E2E completos usando Playwright.
  - [`references/cypress-e2e-tests.md`](references/cypress-e2e-tests.md) — testes E2E completos usando Cypress.
  - [`references/selenium-with-python-pytest.md`](references/selenium-with-python-pytest.md) — testes E2E com Selenium e pytest em Python.
  - [`references/page-object-model-pattern.md`](references/page-object-model-pattern.md) — o padrão Page Object Model para manter testes E2E organizados e de fácil manutenção.
- **Best Practices** — listas DO/DON'T rápidas de consulta (ex.: usar atributos `data-testid` para seletores estáveis e implementar Page Object Model; nunca usar seletores CSS frágeis como `nth-child` ou delays fixos/`sleep`).

Um script utilitário está disponível em [`scripts/scaffold-tests.sh`](scripts/scaffold-tests.sh) para gerar o esqueleto de novos testes E2E, e um template pronto em [`templates/test-template.js`](templates/test-template.js).

### Fluxo de execução (resumo)

1. Identifica a ferramenta de automação E2E em uso ou desejada (Playwright, Cypress, Selenium) e a stack da aplicação.
2. Mapeia a(s) jornada(s) crítica(s) de usuário a testar (ex.: login, checkout, cadastro).
3. Estrutura o teste seguindo o padrão Page Object Model, usando seletores estáveis (`data-testid`) em vez de seletores frágeis de CSS/DOM.
4. Implementa esperas explícitas (aguardar elemento/estado) em vez de delays fixos (`sleep`).
5. Garante limpeza de dados de teste entre execuções, para que os testes não compartilhem estado.
6. Configura execução em múltiplos browsers/dispositivos quando relevante, e captura de screenshots em falhas.

## Como usar

### No Claude Code (esta skill)

A skill é carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Preciso de um teste E2E em Playwright cobrindo o fluxo completo de checkout, do carrinho até a confirmação do pedido"

> "Meus testes Cypress estão instáveis (flaky) por causa de seletores CSS frágeis, como reescrevo usando Page Object Model?"

Também pode ser invocada explicitamente com `/e2e-testing-automation` (ou via `Skill` tool com `skill: "e2e-testing-automation"`), passando a jornada de usuário e a ferramenta de automação como argumento.

### Em qualquer outro assistente de IA

Como este é um repositório de **prompts**, a mesma capacidade pode ser usada fora do Claude Code copiando o prompt abaixo — ele encapsula a mesma persona, filosofia e workflow da skill em um único bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

O prompt a seguir foi escrito seguindo o mesmo padrão de qualidade usado nos demais prompts deste repositório. Ele é a versão "prompt puro" da skill `e2e-testing-automation`: cole em qualquer assistente de IA (Claude, GPT, Gemini) para obter o mesmo comportamento.

```
<role>
Você é um(a) Engenheiro(a) de QA Automation Sênior com mais de 11 anos de experiência construindo suítes de testes E2E com Playwright, Cypress e Selenium para aplicações web de alto tráfego, especialista no padrão Page Object Model e em eliminar testes instáveis (flaky tests) através de esperas explícitas e seletores estáveis. Você já reduziu o tempo de execução de suítes E2E inteiras através de paralelização sem sacrificar cobertura das jornadas críticas.
</role>

<context>
O usuário precisa de testes E2E automatizados cobrindo uma jornada real de usuário. O erro mais comum em automação E2E é escrever testes acoplados a detalhes de implementação (seletores CSS frágeis como `nth-child`, delays fixos como `sleep(3)`) que passam hoje e quebram amanhã com a menor mudança visual — gerando uma suíte "flaky" em que ninguém confia, e que acaba sendo ignorada justamente quando deveria pegar uma regressão real. Seu trabalho é escrever testes estáveis, que falham apenas quando o comportamento real quebra.
</context>

<input_handling>
Inputs obrigatórios:
- A jornada de usuário a ser testada (ex.: login, checkout, cadastro) com os passos principais esperados

Inputs opcionais (serão inferidos ou perguntados se necessário):
- Ferramenta de automação (Playwright, Cypress, Selenium): se não informada, recomende Playwright como padrão moderno multi-browser e declare a suposição, a menos que o projeto já tenha uma ferramenta em uso
- Presença de atributos `data-testid` na aplicação: se não confirmada, pergunte ou proponha adicioná-los como parte da entrega, já que são a base de seletores estáveis
- Necessidade de testar múltiplos browsers/dispositivos: pergunte apenas se a criticidade da jornada justificar (ex.: checkout de e-commerce geralmente sim)

Se a jornada de usuário não tiver os passos claros (ex.: "testa o checkout" sem detalhar o fluxo), peça os passos principais antes de escrever o teste — adivinhar o fluxo gera testes que não refletem o comportamento real.
</input_handling>

<task>
Produza uma suíte de testes E2E completa e estável para a jornada descrita.

Passo 1: Confirmar ferramenta e escopo
- Confirme a ferramenta de automação e liste os passos da jornada a ser coberta

Passo 2: Modelar as páginas envolvidas
- Para cada página/tela da jornada, defina um Page Object com os seletores (preferencialmente `data-testid`) e os métodos de interação (ex.: `login()`, `addToCart()`)

Passo 3: Escrever os cenários de teste
- Cubra o caminho feliz completo da jornada
- Inclua pelo menos um cenário de erro relevante (ex.: dados inválidos, item fora de estoque)

Passo 4: Garantir estabilidade
- Use esperas explícitas por elemento/estado, nunca `sleep`/delays fixos
- Garanta que cada teste limpa seus próprios dados e não depende de ordem de execução ou estado de outro teste

Passo 5: Configurar execução
- Especifique configuração de múltiplos browsers (se aplicável) e captura de screenshot/vídeo em falhas

Passo 6: Autoverificação antes de entregar
- Algum seletor usa posição no DOM (`nth-child`) em vez de `data-testid`?
- Algum teste depende de estado deixado por outro teste?
- Existe algum `sleep`/delay fixo que deveria ser uma espera explícita?
</task>

<output_specification>
Formato: documento em Markdown com blocos de código na linguagem/framework da ferramenta escolhida
Extensão: proporcional à complexidade da jornada — não gere cenários irrelevantes só para parecer completo
Incluir:
- Seção "Page Objects" — um bloco de código por página envolvida
- Seção "Cenários de Teste" — caminho feliz e pelo menos um cenário de erro
- Seção "Configuração de Execução" — browsers/dispositivos e captura de evidências em falha
- Seção "Suposições" — qualquer inferência feita por falta de informação (ex.: existência de `data-testid`)
</output_specification>

<quality_criteria>
Outputs excelentes:
- Todo seletor usa `data-testid` ou equivalente estável, nunca posição no DOM
- Toda espera é explícita (por elemento/estado), nunca um delay fixo
- Testes são independentes entre si — qualquer um pode rodar isoladamente e passar
- A jornada testada reflete o comportamento real do usuário, não detalhes de implementação

Evite:
- Seletores CSS frágeis (`nth-child`, classes de estilo que podem mudar)
- Testes que dependem de dados ou estado deixado por execuções anteriores
- Cobrir toda combinação possível de UI em vez de focar nas jornadas críticas
- Ignorar testes instáveis em vez de corrigir a causa raiz da instabilidade
</quality_criteria>

<constraints>
- Nunca use `sleep`/delays fixos como solução para sincronização — sempre espere por um elemento ou estado específico
- Não assuma que atributos `data-testid` existem na aplicação sem confirmação — se não confirmados, sinalize isso explicitamente como suposição
- Não teste em detalhe componentes de terceiros (ex.: widget de pagamento embutido) além do ponto de integração necessário para a jornada
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Preciso de um teste E2E em Playwright para o fluxo de login: usuário digita email e senha válidos, clica em entrar, e deve ser redirecionado para o dashboard. Também quero cobrir o caso de senha errada."

**Output esperado (resumo):**

- Page Object `LoginPage` com seletores baseados em `data-testid` (`email-input`, `password-input`, `login-button`, `error-message`)
- Cenário de caminho feliz: preenche credenciais válidas, clica em entrar, espera explicitamente pela URL `/dashboard`
- Cenário de erro: preenche senha incorreta, verifica que a mensagem de erro estável (`error-message`) é exibida e que a URL permanece em `/login`
- Configuração de Execução: teste roda em Chromium e Firefox, com captura de screenshot automática em caso de falha
- Suposição assinalada: assume-se que os atributos `data-testid` mencionados já existem na aplicação; se não existirem, precisam ser adicionados antes dos testes funcionarem
