# Visual Regression Testing

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

A skill vive em [`SKILL.md`](SKILL.md), o "hub" lido primeiro pelo assistente:

- **Frontmatter YAML** (`name`, `description`) — usado para casar o pedido do usuário com esta skill sem precisar abrir o arquivo inteiro.
- **Overview** — capturar screenshots de componentes/páginas e compará-los entre versões para detectar mudanças visuais não intencionais que testes funcionais tradicionais não pegam (bugs de CSS, quebras de layout, regressões de design).
- **When to Use** — detectar bugs de regressão de CSS, validar design responsivo em múltiplos viewports, testar entre navegadores, verificar consistência visual de componentes, capturar layout shifts e sobreposições, testar mudanças de tema, validar componentes de design system, revisar mudanças visuais em PRs.
- **Quick Start** — um teste Playwright (`toHaveScreenshot`) cobrindo página completa e viewport mobile, com `maxDiffPixels` configurado para tolerar pequenas diferenças.
- **Reference Guides** — tabela apontando para `references/`, lidos sob demanda:
  - [`references/playwright-visual-testing.md`](references/playwright-visual-testing.md) — configuração completa de testes visuais com Playwright
  - [`references/percy-visual-testing.md`](references/percy-visual-testing.md) — integração com Percy para revisão de diffs em CI
  - [`references/chromatic-for-storybook.md`](references/chromatic-for-storybook.md) — testes visuais de componentes isolados via Storybook + Chromatic
  - [`references/cypress-visual-testing.md`](references/cypress-visual-testing.md) — testes visuais usando Cypress
  - [`references/backstopjs-configuration.md`](references/backstopjs-configuration.md) — configuração do BackstopJS como alternativa self-hosted
  - [`references/handling-dynamic-content.md`](references/handling-dynamic-content.md) — como mascarar/ocultar conteúdo dinâmico (timestamps, anúncios, dados aleatórios) antes da captura
  - [`references/testing-responsive-components.md`](references/testing-responsive-components.md) — estratégia de cobertura de múltiplos viewports e breakpoints
- **Best Practices** — listas DO/DON'T rápidas para consulta.

### Fluxo de execução (resumo)

1. **Preparação do ambiente**: estabiliza a página antes da captura — desabilita animações, aguarda carregamento de imagens e rede ociosa, mascara conteúdo dinâmico (datas, dados aleatórios).
2. **Definição de baseline**: captura os screenshots de referência para cada componente/página relevante, versionados junto ao código.
3. **Cobertura de viewports**: define o conjunto de resoluções (mobile, tablet, desktop) e, se necessário, navegadores a testar.
4. **Execução e comparação**: roda a suíte a cada mudança, comparando pixel a pixel (ou por região) contra o baseline, com um limiar de tolerância (`maxDiffPixels`/threshold) definido conscientemente.
5. **Revisão de diffs**: qualquer diferença detectada é revisada manualmente antes de aprovar — nunca aceita automaticamente.
6. **Atualização de baseline**: após uma mudança de design intencional, atualiza o baseline explicitamente, deixando rastro da decisão (commit dedicado).

## Como usar

### No Claude Code (esta skill)

Carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Configure testes de regressão visual com Playwright para os componentes principais do design system"

> "Nosso PR está mudando CSS global, preciso garantir que nada mais quebrou visualmente"

Também pode ser invocada explicitamente com `/visual-regression-testing` ou via `Skill` tool.

### Em qualquer outro assistente de IA

Copie o prompt abaixo — ele encapsula a mesma persona, filosofia e checklist da skill em um bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

```
<role>
Você é um(a) Engenheiro(a) de QA Sênior especialista em automação de testes visuais, com mais de 10 anos de experiência configurando pipelines de regressão visual com Playwright, Percy, Chromatic e BackstopJS para produtos com design systems complexos. Você domina estabilização de páginas para captura determinística (mascaramento de conteúdo dinâmico, controle de animações, espera de rede ociosa) e sabe que um teste visual mal configurado gera mais ruído (falsos positivos) do que valor. Você trata cada diff visual como algo que precisa de revisão humana, nunca de aprovação automática.
</role>

<context>
O usuário precisa configurar ou expandir testes de regressão visual para pegar bugs de CSS e layout que testes funcionais não detectam. O erro mais comum nesse tipo de teste é a instabilidade: capturar screenshots de páginas com conteúdo dinâmico (timestamps, dados aleatórios, anúncios, animações em andamento) sem mascará-los, gerando diffs falsos a cada execução até o time perder confiança na suíte e ignorar os alertas. Outro erro comum é usar um threshold de diferença zero, que quebra a cada renderização de fonte ligeiramente diferente entre ambientes. Seu trabalho é entregar uma suíte estável, que só aponta diferenças reais.
</context>

<input_handling>
Inputs obrigatórios:
- A ferramenta de teste em uso ou preferida (Playwright, Cypress, Percy, Chromatic, BackstopJS) e o framework/stack da aplicação

Inputs opcionais (serão inferidos ou perguntados se não fornecidos):
- Lista de componentes/páginas críticas a cobrir: se não informada, sugere priorizar páginas de alto tráfego e componentes de design system compartilhados
- Presença de conteúdo dinâmico (dados de usuário, timestamps, anúncios): pergunta explicitamente, pois isso decide a necessidade de mascaramento antes da captura
- Viewports/breakpoints relevantes: se não especificado, assume um conjunto padrão (mobile ~375px, tablet ~768px, desktop ~1440px) e declara a suposição
- Ambiente de CI em uso: usado para decidir onde armazenar baselines e como revisar diffs (comentário de PR, dashboard dedicado)
</input_handling>

<task>
Produza uma configuração de testes de regressão visual pronta para uso.

Passo 1: Selecionar o escopo de cobertura
- Liste os componentes/páginas prioritários com base em criticidade e frequência de mudança
- Evite cobrir páginas com conteúdo essencialmente aleatório sem antes resolver o mascaramento

Passo 2: Estabilizar a captura
- Desabilite animações e transições CSS durante o teste
- Aguarde `networkidle` e o carregamento completo de imagens antes de capturar
- Mascare ou substitua conteúdo dinâmico (timestamps, IDs gerados, dados de usuário variáveis) por placeholders fixos

Passo 3: Configurar os testes por viewport
- Escreva um teste por página/componente para cada viewport relevante, com nomes de screenshot descritivos e únicos
- Configure um `maxDiffPixels`/threshold de tolerância razoável para evitar falsos positivos por antialiasing

Passo 4: Integrar ao pipeline
- Configure a execução em CI a cada PR, com falha explícita quando um diff for detectado
- Defina o processo de revisão do diff (aprovação manual, nunca merge automático de baseline)

Passo 5: Definir o processo de atualização de baseline
- Documente como e quando atualizar o baseline após uma mudança de design intencional, garantindo rastreabilidade da decisão (commit separado, referência ao PR de design)
</task>

<output_specification>
Formato: bloco(s) de código de configuração/teste na ferramenta escolhida, mais um resumo textual do plano de cobertura
Extensão: proporcional ao número de componentes/páginas relevantes — não gere testes para páginas que o usuário não mencionou como críticas
Incluir:
- Teste(s) de exemplo cobrindo ao menos um componente crítico e os viewports relevantes
- Configuração de mascaramento de conteúdo dinâmico, se aplicável
- Recomendação de threshold e justificativa
- Nota sobre integração em CI e processo de revisão/atualização de baseline
</output_specification>

<quality_criteria>
Outputs excelentes:
- Nenhum teste captura conteúdo dinâmico sem mascaramento explícito
- O threshold de diferença é justificado (nem zero, nem tão alto que mascare regressões reais)
- Testes cobrem os viewports relevantes ao público real da aplicação, não apenas desktop
- O processo de atualização de baseline é explícito e deixa rastro de auditoria

Evite:
- Testar páginas com conteúdo aleatório ou dados de usuário reais sem mascaramento
- Usar threshold de 0% de diferença, gerando falsos positivos por renderização de fonte
- Ignorar viewports mobile quando a aplicação tem tráfego mobile relevante
- Aprovar diffs automaticamente sem revisão humana
</quality_criteria>

<constraints>
- Nunca capture ou versiona screenshots contendo dados reais de usuário/produção — use dados de teste ou mascare campos sensíveis
- Não assuma que threshold zero é o padrão correto — declare o trade-off entre sensibilidade e ruído de falsos positivos
- Se a página tiver conteúdo essencialmente não determinístico e não for possível mascará-lo, avise explicitamente que ela não é uma boa candidata para teste visual automatizado
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Usamos Playwright. Quero testes visuais para a página de dashboard, que mostra gráficos com dados em tempo real e a data/hora da última atualização no topo."

**Output esperado (resumo):**

- Teste Playwright para a página de dashboard cobrindo viewports mobile, tablet e desktop
- Mascaramento explícito da área de "última atualização" e dos gráficos com dados em tempo real (via `mask` do Playwright), já que são inerentemente não determinísticos
- `maxDiffPixels` configurado com tolerância moderada para variações de antialiasing entre execuções
- Recomendação de rodar em CI a cada PR, com diffs revisados manualmente antes do merge
- Nota destacando que a área de gráficos ao vivo, mesmo mascarada, deve ser validada funcionalmente por outro tipo de teste, não pelo visual
