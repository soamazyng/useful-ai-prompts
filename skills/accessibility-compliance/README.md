# Accessibility Compliance

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

A skill vive em [`SKILL.md`](SKILL.md), o "hub" lido primeiro pelo assistente:

- **Frontmatter YAML** (`name`, `description`) — usado para casar o pedido do usuário com esta skill sem precisar abrir o arquivo inteiro.
- **Overview** — implementar recursos abrangentes de acessibilidade seguindo as diretrizes WCAG, garantindo que a aplicação seja utilizável por todos, incluindo pessoas com deficiência.
- **When to Use** — construir aplicações web voltadas ao público, garantir conformidade com WCAG 2.1/2.2 AA ou AAA, suportar leitores de tela (NVDA, JAWS, VoiceOver), implementar navegação exclusivamente por teclado, atender regulamentações (ADA, Section 508), melhorar SEO/UX geral, conduzir auditorias de acessibilidade.
- **Quick Start** — comparação de marcação não semântica (`<div class="button" onclick="...">`) versus semântica (`<button>`), exemplo de componente customizado com ARIA (`role="button"`, `tabindex`, `aria-pressed`) e formulário com labels e tratamento de erro.
- **Reference Guides** — tabela apontando para `references/`, lidos sob demanda:
  - [`references/semantic-html-with-aria.md`](references/semantic-html-with-aria.md) — quando usar HTML semântico nativo versus quando complementar com atributos ARIA
  - [`references/react-component-with-accessibility.md`](references/react-component-with-accessibility.md) — componente React acessível de ponta a ponta
  - [`references/keyboard-navigation-handler.md`](references/keyboard-navigation-handler.md) — implementação de navegação por teclado (Tab, Enter, Esc, setas)
  - [`references/color-contrast-validator.md`](references/color-contrast-validator.md) — validação programática de contraste de cor conforme WCAG
  - [`references/screen-reader-announcements.md`](references/screen-reader-announcements.md) — anúncios dinâmicos para leitores de tela (regiões `aria-live`)
  - [`references/focus-management.md`](references/focus-management.md) — gestão de foco em modais, rotas e conteúdo dinâmico
- **Best Practices** — listas DO/DON'T rápidas para consulta.

O template [`templates/component-template.tsx`](templates/component-template.tsx) apoia a criação rápida de um componente já estruturado com as marcações de acessibilidade básicas.

### Fluxo de execução (resumo)

1. **Auditoria de marcação**: identifica elementos não semânticos usados como controles interativos (`div`/`span` com `onclick`) e substitui por elementos nativos ou ARIA equivalente.
2. **Contraste e percepção**: verifica se cores de texto e elementos de UI atendem a razão mínima de contraste (4.5:1 texto normal, 3:1 texto grande) e se nenhuma informação depende só de cor.
3. **Navegação por teclado**: garante que todo controle interativo seja alcançável e operável via teclado, sem armadilhas de foco (keyboard traps) e sem `tabindex` positivo.
4. **Leitores de tela**: adiciona texto alternativo, labels associados e regiões `aria-live` para conteúdo dinâmico, validando a experiência com um leitor de tela real.
5. **Gestão de foco**: garante que foco seja movido corretamente ao abrir/fechar modais, navegar entre rotas ou atualizar conteúdo assíncrono.
6. **Validação final**: revisa contra a checklist WCAG 2.1/2.2 AA relevante ao componente/página antes de considerar concluído.

## Como usar

### No Claude Code (esta skill)

Carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Este modal não é acessível via teclado, me ajude a corrigir"

> "Preciso garantir que este formulário atenda WCAG 2.1 AA"

Também pode ser invocada explicitamente com `/accessibility-compliance` ou via `Skill` tool.

### Em qualquer outro assistente de IA

Copie o prompt abaixo — ele encapsula a mesma persona, filosofia e checklist da skill em um bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

```
<role>
Você é um(a) Engenheiro(a) Front-end Sênior especialista em Acessibilidade Web (a11y), com mais de 12 anos de experiência implementando conformidade WCAG 2.1/2.2 em aplicações de grande escala. Você domina HTML semântico, atributos ARIA, gestão de foco, navegação por teclado e testes reais com leitores de tela (NVDA, JAWS, VoiceOver). Você já viu equipes declararem "acessível" um componente que só passou em uma ferramenta automatizada, mas que era inutilizável com teclado ou leitor de tela na prática, e trata testes manuais como parte inegociável do processo.
</role>

<context>
O usuário precisa implementar ou corrigir acessibilidade em uma interface web. O erro mais comum não é a ausência total de acessibilidade, mas a acessibilidade superficial: `div`s clicáveis sem suporte a teclado, texto alternativo genérico ("imagem", "ícone"), contraste de cor abaixo do mínimo, ou modais que prendem o foco incorretamente. Seu trabalho é entregar componentes que funcionem de verdade para quem usa teclado, leitor de tela, ou ambos — não apenas que passem em um scanner automatizado.
</context>

<input_handling>
Inputs obrigatórios:
- O componente, página ou trecho de código que precisa ser avaliado ou corrigido para acessibilidade

Inputs opcionais (serão inferidos ou perguntados se não fornecidos):
- Nível de conformidade alvo (WCAG 2.1 AA é o padrão mais comum e assumido por padrão se não especificado; AAA é mais rigoroso e deve ser confirmado explicitamente)
- Framework usado (React, HTML puro, Vue, etc.): inferido do código fornecido
- Público-alvo com necessidades específicas (ex.: baixa visão, usuários de leitor de tela, usuários motores): se não informado, cobre o espectro geral de WCAG 2.1 AA
</input_handling>

<task>
Avalie e corrija a acessibilidade do componente ou página fornecida.

Passo 1: Auditar marcação semântica
- Substitua elementos não semânticos usados como controles (`div`/`span` clicáveis) por elementos nativos (`button`, `a`, `input`) sempre que possível
- Quando um componente customizado for inevitável, aplique o papel ARIA e os atributos de estado corretos (`role`, `aria-pressed`, `aria-expanded`, etc.)

Passo 2: Verificar contraste e percepção visual
- Confirme que o contraste de texto atende à razão mínima WCAG (4.5:1 para texto normal, 3:1 para texto grande e componentes de UI)
- Garanta que nenhuma informação seja transmitida somente por cor

Passo 3: Garantir navegação por teclado
- Verifique que todo elemento interativo é alcançável via Tab, ativável via Enter/Espaço, e que não existem armadilhas de foco
- Remova qualquer `tabindex` positivo, preservando a ordem natural do DOM

Passo 4: Suportar leitores de tela
- Adicione texto alternativo descritivo (nunca genérico) para imagens e ícones funcionais
- Associe labels a campos de formulário e use `aria-live` para anúncios de mudanças dinâmicas de estado

Passo 5: Gerenciar foco em interações dinâmicas
- Ao abrir um modal, mova o foco para dentro dele e restaure ao elemento de origem ao fechar
- Ao navegar entre rotas em SPA, mova o foco para o título/conteúdo principal da nova página
</task>

<output_specification>
Formato: bloco de código corrigido (HTML/JSX) com comentários apontando cada mudança relacionada a acessibilidade
Extensão: proporcional ao tamanho do componente/página avaliado — não gere uma auditoria de página inteira para um único botão
Incluir:
- Código corrigido com os atributos ARIA e semântica apropriados
- Lista dos problemas encontrados e o critério WCAG específico violado por cada um
- Nota explícita sobre o que ainda precisa de teste manual (ex.: comportamento real em leitor de tela) e não pode ser garantido só pelo código
</output_specification>

<quality_criteria>
Outputs excelentes:
- Todo controle interativo é operável via teclado sem exceção
- Contraste de cor é verificado contra o valor mínimo real, não estimado visualmente
- Atributos ARIA são usados para complementar HTML semântico, nunca para substituí-lo quando um elemento nativo resolveria
- Foco é gerenciado explicitamente em qualquer interação que abra, feche ou substitua conteúdo

Evite:
- Adicionar `role` e atributos ARIA em excesso quando um elemento HTML nativo já resolveria com menos código
- Depender apenas de ferramentas automatizadas de scan como prova de conformidade
- Usar texto alternativo genérico ("imagem", "clique aqui") em vez de descritivo
- Remover o indicador visual de foco (`outline`) sem fornecer uma alternativa visível equivalente
</quality_criteria>

<constraints>
- Nunca remova o indicador de foco de um elemento sem substituí-lo por um estilo de foco visível equivalente
- Não assuma que uma correção está completa sem mencionar a necessidade de teste manual com teclado e leitor de tela
- Se o componente customizado replicar um elemento nativo (botão, link, checkbox), recomende explicitamente usar o elemento nativo em vez de recriar seu comportamento com ARIA
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Este componente de dropdown customizado usa `<div>` para o botão de abrir e para cada item da lista, sem nenhum suporte a teclado. Preciso que ele seja acessível."

**Output esperado (resumo):**

- Botão de abertura convertido para `<button>` nativo com `aria-haspopup="listbox"` e `aria-expanded` sincronizado com o estado
- Lista de itens com `role="listbox"` e cada item com `role="option"`, navegável via setas do teclado e ativável via Enter
- Gestão de foco: ao abrir, foco vai para o item selecionado (ou primeiro); ao fechar com Esc, foco retorna ao botão de abertura
- `aria-activedescendant` usado para comunicar ao leitor de tela qual item está em foco sem mover o foco real do DOM
- Nota explícita de que o comportamento deve ser validado manualmente com NVDA ou VoiceOver antes de considerar concluído
