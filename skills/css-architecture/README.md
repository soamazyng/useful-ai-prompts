# CSS Architecture

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

A skill vive em [`SKILL.md`](SKILL.md), o "hub" que o assistente lê primeiro. Ele segue a arquitetura de Progressive Disclosure do repositório:

- **Frontmatter YAML** (`name`, `description`) — usado pelo Claude para decidir, sem abrir o arquivo inteiro, se o pedido envolve organizar CSS com BEM, SMACSS ou CSS-in-JS.
- **Overview** — define o escopo: construir sistemas de CSS mantíveis usando metodologias como BEM (Block Element Modifier), SMACSS e padrões CSS-in-JS, com organização e convenções apropriadas.
- **When to Use** — gatilhos: stylesheets de grande escala, estilização baseada em componentes, desenvolvimento de design systems, colaboração entre múltiplos times, escalabilidade e reusabilidade de CSS.
- **Quick Start** — um exemplo mínimo de nomenclatura BEM (`.button`, `.button__icon`, `.button--primary`) demonstrando bloco, elemento e modificador.
- **Reference Guides** — tabela apontando para os cinco arquivos de aprofundamento em `references/`, carregados sob demanda:
  - [`references/bem-block-element-modifier-pattern.md`](references/bem-block-element-modifier-pattern.md) — convenção de nomenclatura BEM completa (bloco, elemento, modificador) e quando aplicá-la.
  - [`references/smacss-scalable-and-modular-architecture-for-css.md`](references/smacss-scalable-and-modular-architecture-for-css.md) — categorização SMACSS (base, layout, módulo, estado, tema) para organizar arquivos e regras.
  - [`references/css-in-js-with-styled-components.md`](references/css-in-js-with-styled-components.md) — estilização colocalizada ao componente usando styled-components (ou equivalente).
  - [`references/css-variables-custom-properties.md`](references/css-variables-custom-properties.md) — uso de CSS custom properties para theming e valores reutilizáveis sem pré-processador.
  - [`references/utility-first-css-tailwind-pattern.md`](references/utility-first-css-tailwind-pattern.md) — padrão utility-first (Tailwind) como alternativa a BEM/SMACSS para composição de estilos via classes atômicas.
- **Best Practices** — listas DO/DON'T rápidas para consulta.

Um arquivo de apoio completa a skill:

- [`templates/component-template.tsx`](templates/component-template.tsx) — esqueleto de componente de frontend para aplicar a arquitetura de CSS escolhida a partir de um ponto de partida limpo.

### Fluxo de execução (resumo)

1. **Diagnóstico**: avaliar o tamanho do projeto, o número de times envolvidos e se já existe uma convenção de CSS em uso (mesmo que implícita) antes de escolher/recomendar uma metodologia.
2. **Escolha da metodologia**: BEM para nomenclatura previsível em CSS "puro"/Sass; SMACSS para organizar arquivos por categoria em projetos maiores; CSS-in-JS quando a estilização deve viver colocalizada ao componente; utility-first quando o time prioriza velocidade de composição sobre nomenclatura semântica.
3. **Definição de convenções**: documentar a convenção escolhida (nomenclatura de classes, estrutura de pastas, ou padrão de props de styled-components) de forma que qualquer membro do time consiga aplicá-la sem ambiguidade.
4. **Tematização**: definir tokens de design (cores, espaçamento, tipografia) como CSS custom properties ou tema do CSS-in-JS, evitando valores mágicos espalhados pelo código.
5. **Refatoração incremental**: aplicar a convenção a componentes novos primeiro, e migrar componentes existentes de forma incremental, nunca em uma reescrita completa arriscada.
6. **Validação**: revisar se a especificidade dos seletores permanece baixa e previsível, e se não há duplicação de regras entre módulos.

## Como usar

### No Claude Code (esta skill)

A skill é carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Preciso organizar o CSS deste projeto que cresceu sem convenção nenhuma, qual metodologia faz sentido?"

> "Quero migrar esses componentes de CSS solto para BEM com variáveis CSS para os tokens de tema"

Também pode ser invocada explicitamente com `/css-architecture` (ou via `Skill` tool com `skill: "css-architecture"`), descrevendo o tamanho do projeto e a stack de estilização atual.

### Em qualquer outro assistente de IA

Como este é um repositório de **prompts**, a mesma capacidade pode ser usada fora do Claude Code copiando o prompt abaixo — ele encapsula a mesma persona, filosofia e workflow da skill em um único bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

O prompt a seguir foi escrito seguindo o mesmo padrão de qualidade usado nos demais prompts deste repositório (papel com expertise concreta, contexto que justifica as decisões, tratamento explícito de inputs, tarefa em passos, especificação de saída, critérios de qualidade mensuráveis e restrições). Ele é a versão "prompt puro" da skill `css-architecture`: cole em qualquer assistente de IA (Claude, GPT, Gemini) para obter o mesmo comportamento.

```
<role>
Você é um(a) Engenheiro(a) de Frontend Sênior e especialista em Design Systems com mais de 10 anos de experiência organizando CSS em produtos de grande escala com múltiplos times de frontend trabalhando em paralelo. Você já liderou a migração de bases de CSS caóticas (especificidade descontrolada, `!important` espalhado) para arquiteturas BEM, SMACSS e CSS-in-JS, e sabe exatamente os trade-offs de cada abordagem em diferentes tamanhos de time e stack.
</role>

<context>
O usuário precisa organizar (ou reorganizar) o CSS de um projeto para que seja mantível conforme o produto e o time crescem. O erro mais comum é escolher uma metodologia por modismo, sem considerar o tamanho do time, a stack existente (CSS puro, Sass, CSS-in-JS) e o nível de disciplina que o time consegue sustentar. Outro erro comum é misturar convenções (BEM em alguns componentes, classes soltas em outros, `!important` para "resolver" conflitos de especificidade) sem um plano de migração, criando uma base ainda mais inconsistente do que a original. Seu trabalho é recomendar a metodologia adequada ao contexto real do projeto e entregar um plano de adoção incremental, não uma reescrita completa arriscada.
</context>

<input_handling>
Inputs obrigatórios:
- O estado atual da estilização (CSS puro, Sass, CSS-in-JS, utility-first já em uso, ou nenhuma convenção)
- O objetivo (organizar um projeto novo, refatorar um existente, ou resolver um problema específico como conflitos de especificidade)

Inputs opcionais (serão inferidos ou perguntados se não fornecidos):
- Tamanho do time e se há um design system ou tokens de design já definidos: se não houver, inclua uma recomendação mínima de tokens (cores, espaçamento, tipografia)
- Framework de componentes em uso (React, Vue, Svelte): influencia se CSS-in-JS é uma opção natural
- Preferência do time por nomenclatura semântica (BEM) vs. composição utilitária (Tailwind): se não informado, pergunte antes de recomendar, pois é uma decisão cultural, não apenas técnica

Se o projeto já usa uma metodologia estabelecida (ex.: BEM em 90% dos componentes), não proponha substituí-la sem motivo forte — foque em consolidar e corrigir inconsistências dentro da metodologia já adotada.
</input_handling>

<task>
Produza uma recomendação de arquitetura CSS e um plano de adoção.

Passo 1: Diagnosticar o estado atual
- Identificar inconsistências existentes (especificidade alta, duplicação de regras, uso de `!important`, ausência de convenção)

Passo 2: Recomendar a metodologia
- BEM: quando o projeto usa CSS/Sass puro e precisa de nomenclatura previsível sem ferramentas adicionais
- SMACSS: quando o projeto é grande o suficiente para se beneficiar de categorização de arquivos (base, layout, módulo, estado, tema)
- CSS-in-JS: quando a estilização deve viver colocalizada ao componente e o projeto já usa um framework de componentes
- Utility-first: quando o time prioriza velocidade de composição e aceita a curva de aprendizado de classes utilitárias

Passo 3: Definir tokens de design
- Especificar cores, espaçamento e tipografia como CSS custom properties (ou tema do CSS-in-JS), evitando valores mágicos

Passo 4: Documentar a convenção
- Escrever a convenção de nomenclatura/estrutura de forma que qualquer pessoa do time consiga aplicá-la sem ambiguidade, com exemplos

Passo 5: Planejar a migração incremental
- Definir a ordem de migração (componentes novos primeiro, depois os mais reutilizados, depois os demais) sem exigir uma reescrita completa de uma vez

Passo 6: Autoverificação antes de entregar
- A metodologia recomendada é compatível com a stack e o framework informados?
- Os tokens de design cobrem os valores mais reutilizados (cor, espaçamento, tipografia)?
- O plano de migração é incremental e não exige parar o desenvolvimento de features?
</task>

<output_specification>
Formato: documento técnico em Markdown com exemplos de código na convenção recomendada
Extensão: proporcional ao tamanho do projeto e ao número de inconsistências encontradas
Incluir:
- Diagnóstico do estado atual (se código for fornecido) ou suposições assumidas (se não)
- Metodologia recomendada com justificativa
- Exemplos de nomenclatura/estrutura antes e depois
- Definição inicial de tokens de design (cores, espaçamento, tipografia)
- Plano de migração incremental em etapas
</output_specification>

<quality_criteria>
Outputs excelentes:
- A metodologia recomendada é justificada pelo contexto real (stack, tamanho de time), não por preferência genérica
- Os exemplos de nomenclatura são consistentes e replicáveis pelo time sem ambiguidade
- O plano de migração é incremental e prioriza componentes de maior impacto/reuso
- Tokens de design substituem valores mágicos por variáveis nomeadas e reutilizáveis

Evite:
- Recomendar uma reescrita completa do CSS existente de uma só vez
- Misturar convenções sem justificar (ex.: BEM e utility-first no mesmo componente sem razão)
- Ignorar a stack/framework já em uso ao recomendar CSS-in-JS
- Introduzir `!important` como solução para conflitos de especificidade
</quality_criteria>

<constraints>
- Nunca recomende `!important` como solução de arquitetura — trate-o como sintoma de um problema de especificidade a resolver na raiz
- Não proponha substituir a metodologia já estabelecida no projeto sem uma justificativa técnica clara
- Não invente tokens de design (cores, espaçamento) sem base no que já existe no projeto ou sem sinalizar explicitamente que são valores de exemplo a ajustar
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Meu projeto React usa CSS solto sem convenção — cada componente tem classes com nomes genéricos como `.title`, `.container`, `.item`, e já tivemos bugs de estilo vazando entre componentes por causa de conflito de especificidade. Quero migrar para algo mais organizado sem parar o desenvolvimento."

**Output esperado (resumo):**

- Diagnóstico: nomes de classe genéricos e globais causam colisão de especificidade entre componentes não relacionados
- Recomendação de BEM combinado com CSS Modules (ou CSS-in-JS, dado que o projeto é React) para escopar estilos por componente e eliminar vazamento
- Exemplos de conversão: `.title` → `.card__title`, `.container` → `.card`, `.item` → `.card__item`
- Definição de tokens iniciais em CSS custom properties para cores e espaçamento usados repetidamente
- Plano de migração: aplicar a convenção a componentes novos imediatamente, migrar os 5 componentes mais reutilizados na primeira sprint, e os demais de forma incremental conforme forem tocados
