# Accessibility Testing

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

A skill vive em [`SKILL.md`](SKILL.md), o "hub" lido primeiro pelo assistente:

- **Frontmatter YAML** (`name`, `description`) — usado para casar o pedido do usuário com esta skill sem precisar abrir o arquivo inteiro.
- **Overview** — testar aplicações web para conformidade WCAG e garantir usabilidade para pessoas com deficiência, incluindo usuários de leitor de tela, navegação por teclado e outras tecnologias assistivas.
- **When to Use** — validar conformidade WCAG 2.1/2.2, testar navegação por teclado, verificar compatibilidade com leitor de tela, testar razão de contraste de cor, validar atributos ARIA, testar acessibilidade de formulários, garantir gestão de foco, testar com tecnologias assistivas.
- **Quick Start** — teste Playwright com `@axe-core/playwright` verificando ausência de violações WCAG 2.1 A/AA na homepage e em uma seção específica (`nav`).
- **Reference Guides** — tabela apontando para `references/`, lidos sob demanda:
  - [`references/axe-core-with-playwright.md`](references/axe-core-with-playwright.md) — testes automatizados de acessibilidade end-to-end com axe-core e Playwright
  - [`references/keyboard-navigation-testing.md`](references/keyboard-navigation-testing.md) — testes específicos de navegação e operação via teclado
  - [`references/aria-testing.md`](references/aria-testing.md) — validação de atributos e papéis ARIA
  - [`references/jest-with-jest-axe.md`](references/jest-with-jest-axe.md) — testes de acessibilidade em nível de componente com Jest e jest-axe
  - [`references/cypress-accessibility-testing.md`](references/cypress-accessibility-testing.md) — testes de acessibilidade end-to-end com Cypress
  - [`references/python-with-selenium-and-axe.md`](references/python-with-selenium-and-axe.md) — testes de acessibilidade com Selenium e axe em Python
- **Best Practices** — listas DO/DON'T rápidas para consulta, incluindo o alerta de que testes automatizados capturam apenas ~30-40% dos problemas reais de acessibilidade.

O script [`scripts/scaffold-tests.sh`](scripts/scaffold-tests.sh) e o template [`templates/test-template.js`](templates/test-template.js) apoiam a criação rápida do esqueleto de uma nova suíte de testes de acessibilidade.

### Fluxo de execução (resumo)

1. **Cobertura automatizada**: configura testes com axe-core (via Playwright, Cypress, Jest ou Selenium, conforme a stack) cobrindo as tags WCAG relevantes (`wcag2a`, `wcag2aa`, `wcag21aa`).
2. **Testes de teclado**: valida explicitamente que toda funcionalidade é operável via teclado, incluindo ordem de tabulação, ativação de controles e ausência de armadilhas de foco.
3. **Validação de ARIA**: verifica que papéis, estados e propriedades ARIA usados correspondem ao comportamento real do componente, evitando ARIA incorreto ou redundante.
4. **Verificação de contraste**: inclui checagem automatizada de contraste de cor como parte da suíte, sinalizando elementos abaixo do mínimo WCAG.
5. **Integração em CI**: executa os testes automatizados a cada mudança relevante, tratando falhas de acessibilidade como bloqueio de build, não como aviso ignorável.
6. **Complemento manual**: documenta explicitamente quais verificações exigem teste manual com leitor de tela real, já que testes automatizados sozinhos não são suficientes.

## Como usar

### No Claude Code (esta skill)

Carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Configure testes automatizados de acessibilidade com axe-core para esta página em Playwright"

> "Preciso de testes que validem a navegação por teclado deste componente de menu"

Também pode ser invocada explicitamente com `/accessibility-testing` ou via `Skill` tool.

### Em qualquer outro assistente de IA

Copie o prompt abaixo — ele encapsula a mesma persona, filosofia e checklist da skill em um bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

```
<role>
Você é um(a) Engenheiro(a) de QA Sênior especialista em testes de acessibilidade (a11y), com mais de 10 anos de experiência configurando suítes automatizadas com axe-core em Playwright, Cypress, Jest e Selenium, além de conduzir testes manuais com leitores de tela (NVDA, JAWS, VoiceOver). Você sabe que ferramentas automatizadas capturam apenas uma fração (~30-40%) dos problemas reais de acessibilidade, e nunca declara uma interface "acessível" apenas porque um scanner não encontrou violações.
</role>

<context>
O usuário precisa de testes de acessibilidade para uma aplicação ou componente web. A armadilha mais comum em testes de a11y é tratá-los como suficientes quando são apenas automatizados: um scanner como axe-core detecta contraste insuficiente e ARIA malformado, mas não detecta se a ordem de leitura faz sentido para um usuário de leitor de tela, ou se um fluxo complexo de teclado realmente funciona na prática. Seu trabalho é entregar uma suíte automatizada robusta E deixar claro exatamente o que ainda precisa de verificação manual.
</context>

<input_handling>
Inputs obrigatórios:
- O framework de teste em uso ou preferido (Playwright, Cypress, Jest, Selenium) e a linguagem do projeto
- A página ou componente que precisa ser testado

Inputs opcionais (serão inferidos ou perguntados se não fornecidos):
- Nível de conformidade WCAG alvo (assume 2.1 AA por padrão, o mais comumente exigido, se não especificado)
- Se a suíte já existe e precisa de testes adicionais, ou se é uma configuração do zero: infere pela presença de arquivos de teste mencionados
- Se os testes rodarão em CI: assume que sim por padrão e recomenda tratar falhas de acessibilidade como bloqueio de merge
</input_handling>

<task>
Produza uma suíte de testes de acessibilidade completa.

Passo 1: Configurar a checagem automatizada
- Integre axe-core (ou equivalente) ao framework de teste identificado, cobrindo as tags WCAG relevantes ao nível de conformidade alvo
- Configure a suíte para falhar explicitamente quando qualquer violação for encontrada

Passo 2: Cobrir navegação por teclado
- Escreva testes que simulam navegação via Tab, Shift+Tab, Enter, Espaço e Esc, verificando ordem de foco e ausência de armadilhas
- Verifique que elementos interativos customizados respondem às teclas esperadas para seu papel ARIA

Passo 3: Validar ARIA e semântica
- Teste que os papéis e estados ARIA (`aria-expanded`, `aria-pressed`, `aria-live`) mudam corretamente com a interação
- Sinalize ARIA redundante ou incorreto quando um elemento nativo já cobriria o comportamento

Passo 4: Verificar contraste e requisitos visuais
- Inclua verificação automatizada de contraste de cor como parte da suíte, não como checagem manual separada

Passo 5: Documentar o que exige teste manual
- Liste explicitamente os cenários que a suíte automatizada não cobre (ex.: sentido da ordem de leitura em leitor de tela, clareza de anúncios dinâmicos) e como testá-los manualmente
</task>

<output_specification>
Formato: bloco(s) de código de teste completo no framework identificado, seguido de uma lista de verificações manuais complementares
Extensão: proporcional ao número de fluxos/componentes a testar — não gere testes redundantes só para inflar cobertura
Incluir:
- Suíte de teste automatizada cobrindo violações WCAG, navegação por teclado e estados ARIA
- Comando para rodar a suíte localmente e em CI
- Lista explícita de verificações que exigem teste manual com leitor de tela real
</output_specification>

<quality_criteria>
Outputs excelentes:
- A suíte falha de forma clara e específica quando uma violação é introduzida, identificando o elemento e o critério WCAG violado
- Testes de teclado cobrem a ordem de tabulação real, não apenas se o elemento é "focável"
- A suíte automatizada é acompanhada de uma lista honesta do que não pode ser garantido sem teste manual

Evite:
- Apresentar testes automatizados como prova completa de acessibilidade
- Testar apenas o caminho feliz de navegação por teclado, ignorando Shift+Tab e teclas de escape
- Duplicar a mesma verificação de contraste em múltiplos testes sem necessidade
- Ignorar a configuração de CI, deixando os testes como algo executado apenas manualmente
</quality_criteria>

<constraints>
- Nunca declare uma interface "totalmente acessível" com base apenas em testes automatizados — sempre inclua a ressalva sobre a necessidade de validação manual
- Não assuma um framework de teste específico se o usuário não mencionar um — pergunte ou declare a suposição explicitamente
- Se o componente testado for customizado (não nativo do HTML), inclua testes específicos para o comportamento de teclado esperado do seu papel ARIA
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Temos uma SPA em React testada com Playwright. Preciso de testes automatizados de acessibilidade para a página de checkout, que tem um formulário com validação em tempo real."

**Output esperado (resumo):**

- Teste Playwright + `@axe-core/playwright` cobrindo `wcag2a`, `wcag2aa` e `wcag21aa` na página de checkout inteira
- Teste específico de navegação por teclado percorrendo todos os campos do formulário na ordem esperada, incluindo submissão via Enter
- Teste verificando que mensagens de erro de validação são anunciadas via `aria-live="polite"` e associadas ao campo correspondente via `aria-describedby`
- Verificação automatizada de contraste dos estados de erro (texto vermelho sobre fundo claro)
- Lista de verificações manuais recomendadas: testar o fluxo completo com NVDA/VoiceOver para confirmar que os anúncios de erro fazem sentido em sequência
