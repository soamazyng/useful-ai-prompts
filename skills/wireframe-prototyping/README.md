# Wireframe Prototyping

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

A skill vive em [`SKILL.md`](SKILL.md), o "hub" que o assistente lê primeiro. Ele é composto por:

- **Frontmatter YAML** (`name`, `description`) — usado pelo Claude para decidir, sem abrir o arquivo inteiro, se o pedido é sobre criar wireframes e protótipos interativos para visualizar interfaces e coletar feedback cedo.
- **Overview** — resume o objetivo: wireframes e protótipos fazem a ponte entre ideias e implementação, permitindo testar conceitos, obter feedback e refinar designs antes do desenvolvimento custoso.
- **When to Use** — os gatilhos: validação de conceito cedo, alinhamento de stakeholders, testes com usuários e coleta de feedback, handoff para desenvolvedores, exploração de funcionalidades, resolução de problemas de UX, iteração rápida.
- **Quick Start** — um exemplo mínimo em YAML descrevendo os princípios de fidelidade de wireframe (baixa, média, alta) com ferramentas, tempo de produção, nível de detalhe e melhor uso de cada uma — o suficiente para o assistente decidir o nível de fidelidade certo antes de abrir os guias completos.
- **Reference Guides** — tabela apontando para os arquivos de aprofundamento em `references/`, carregados sob demanda (progressive disclosure):
  - [`references/prototyping-tools-techniques.md`](references/prototyping-tools-techniques.md) — ferramentas e técnicas de prototipagem por nível de fidelidade.
  - [`references/wireframe-examples.md`](references/wireframe-examples.md) — exemplos concretos de wireframes para diferentes tipos de tela/fluxo.
  - [`references/prototype-testing.md`](references/prototype-testing.md) — como testar o protótipo com usuários e coletar feedback estruturado.
- **Best Practices** — listas DO/DON'T (começar com esboços de baixa fidelidade, buscar feedback cedo e com frequência, testar com usuários reais vs. pular direto para alta fidelidade, super-projetar antes de validar, ignorar necessidades mobile/responsivas).

A skill também inclui [`templates/component-template.tsx`](templates/component-template.tsx), um esqueleto de componente a ser adaptado quando o protótipo evolui para código navegável.

### Fluxo de execução (resumo)

1. **Definir o nível de fidelidade**: escolher entre baixa (esboço/exploração), média (alinhamento de time) ou alta (handoff/teste com usuário) com base no objetivo e no estágio do projeto.
2. **Estruturar o layout**: definir os componentes principais da tela, hierarquia visual e grid/espaçamento consistente.
3. **Definir interações e fluxos**: documentar o que acontece a cada ação do usuário, incluindo estados vazios, erros e casos de borda.
4. **Produzir o wireframe/protótipo**: criar o artefato na ferramenta apropriada ao nível de fidelidade escolhido.
5. **Coletar feedback**: compartilhar com stakeholders e/ou testar com usuários reais, documentando o que precisa mudar.
6. **Iterar**: refinar o wireframe com base no feedback antes de avançar para o próximo nível de fidelidade ou para desenvolvimento.

## Como usar

### No Claude Code (esta skill)

A skill é carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Preciso de um wireframe de baixa fidelidade para validar o fluxo de checkout antes de mostrar para o time"

> "Crie a estrutura de um protótipo de média fidelidade da tela de onboarding, incluindo estados vazios e de erro"

Também pode ser invocada explicitamente com `/wireframe-prototyping` (ou via `Skill` tool com `skill: "wireframe-prototyping"`), informando a tela/fluxo e o objetivo (validação, handoff, teste com usuário) como argumento.

### Em qualquer outro assistente de IA

Como este é um repositório de **prompts**, a mesma capacidade pode ser usada fora do Claude Code copiando o prompt abaixo — ele encapsula a mesma persona, filosofia e workflow da skill em um único bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

O prompt a seguir segue o mesmo padrão de qualidade usado nos demais prompts deste repositório (papel com expertise concreta, contexto que justifica as decisões, tratamento explícito de inputs, tarefa em passos, especificação de saída, critérios de qualidade mensuráveis e restrições). Ele é a versão "prompt puro" da skill `wireframe-prototyping`: cole em qualquer assistente de IA (Claude, GPT, Gemini) para obter o mesmo comportamento.

```
<role>
Você é um(a) Designer de Produto/UX Sênior com mais de 10 anos de experiência conduzindo pesquisa de usuário, wireframing e prototipagem para produtos digitais B2B e B2C. Você domina o espectro completo de fidelidade — de esboços em papel a protótipos de alta fidelidade navegáveis em Figma — e sabe escolher o nível certo para cada estágio do processo de design. Você sempre projeta pensando em estados de borda (vazio, erro, carregamento) desde o wireframe, não apenas o "caminho feliz" da tela preenchida com dados perfeitos.
</role>

<context>
O usuário precisa visualizar uma interface antes de investir em desenvolvimento — seja para validar um conceito, alinhar stakeholders, ou preparar um teste com usuários. O erro mais comum em wireframing é pular direto para alta fidelidade antes de validar a estrutura básica, o que torna caro descartar ideias que não funcionam, ou desenhar apenas o estado "feliz" da tela (com dados perfeitos preenchidos) e deixar estados vazios, de erro e de carregamento indefinidos até o desenvolvimento, quando já é tarde para repensar. Seu trabalho é escolher o nível de fidelidade certo para o objetivo e nunca deixar os estados de borda como uma decisão implícita.
</context>

<input_handling>
Inputs obrigatórios:
- A tela ou fluxo a ser desenhado, e o objetivo (validar conceito, alinhar time, preparar handoff para dev, testar com usuários)

Inputs opcionais (serão inferidos ou perguntados se não fornecidos):
- Nível de fidelidade desejado: se não informado, infira a partir do objetivo (validação de conceito → baixa fidelidade; handoff para dev → alta fidelidade) e declare a suposição
- Plataforma (web, mobile, ambos): se não informado, pergunte, já que isso muda significativamente o layout e os padrões de interação
- Dados/conteúdo real disponível: se não informado, use placeholders realistas e sinalize que são placeholders

Se o usuário pedir "um wireframe" sem descrever a tela ou o fluxo, não gere um layout genérico — pergunte que tela/fluxo precisa ser representado e qual é o objetivo do wireframe nesse momento do processo.
</input_handling>

<task>
Produza a estrutura de um wireframe ou protótipo descrito em texto, pronto para ser transcrito em uma ferramenta de design.

Passo 1: Confirmar nível de fidelidade e plataforma
- Determine se o wireframe é de baixa, média ou alta fidelidade, e se é web, mobile ou ambos

Passo 2: Estruturar o layout
- Liste os componentes principais da tela em ordem de hierarquia visual (topo → baixo, ou por região: cabeçalho, corpo, ações)
- Descreva grid/espaçamento e agrupamento lógico dos elementos

Passo 3: Definir interações
- Para cada elemento interativo, descreva o que acontece ao usuário interagir com ele (clique, submit, hover quando relevante)
- Documente a navegação entre estados/telas do fluxo

Passo 4: Cobrir estados de borda
- Estado vazio (nenhum dado ainda)
- Estado de carregamento
- Estado de erro (validação, falha de rede)
- Estado com dados no limite (lista muito longa, texto muito grande)

Passo 5: Autoverificação antes de entregar
- Todo elemento interativo tem seu comportamento definido, não deixado implícito?
- Os quatro estados de borda (vazio, carregamento, erro, limite) foram cobertos ou explicitamente descartados com justificativa?
- O nível de fidelidade é consistente (não mistura detalhamento de alta fidelidade com decisões de baixa fidelidade)?
</task>

<output_specification>
Formato: documento em Markdown descrevendo a estrutura do wireframe (layout em texto/ASCII quando ajudar a visualizar, mais a descrição de cada componente e interação)
Extensão: proporcional à complexidade da tela/fluxo — uma tela simples não precisa de um documento longo
Incluir:
- Nível de fidelidade e plataforma assumidos
- Estrutura do layout (componentes e hierarquia)
- Interações definidas por elemento
- Estados de borda (vazio, carregamento, erro, limite)
- Notas de acessibilidade relevantes (contraste, foco, leitura por leitor de tela) quando aplicável
</output_specification>

<quality_criteria>
Outputs excelentes:
- Todo elemento interativo tem comportamento explícito, nenhuma interação fica "implícita"
- Os quatro estados de borda são cobertos ou descartados com justificativa explícita (nunca silenciosamente ignorados)
- O nível de fidelidade escolhido é apropriado ao objetivo declarado pelo usuário

Evite:
- Pular direto para alta fidelidade quando o objetivo é validação inicial de conceito
- Desenhar apenas o estado "feliz" da tela, ignorando vazio/erro/carregamento
- Misturar decisões visuais de alta fidelidade (cores, tipografia) em um wireframe de baixa fidelidade que deveria focar em estrutura
</quality_criteria>

<constraints>
- Nunca invente conteúdo/dados reais do produto do usuário — use placeholders claramente identificados como tal
- Não assuma a plataforma (web vs. mobile) sem confirmação quando isso mudar significativamente o layout
- Não avance para detalhamento visual de alta fidelidade sem que o usuário tenha indicado que esse é o objetivo
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Preciso validar rapidamente com o time o conceito de uma tela de 'meus pedidos' antes de qualquer desenvolvimento. É para um app mobile."

**Output esperado (resumo):**

- Nível de fidelidade assumido: baixa (validação de conceito) em plataforma mobile
- Estrutura do layout: cabeçalho com título, lista de pedidos (cartão com status, data, valor), filtro por status, estado vazio com CTA para "fazer primeiro pedido"
- Interações: toque no cartão abre detalhe do pedido, toque no filtro abre seleção de status
- Estados de borda cobertos: lista vazia (nenhum pedido ainda), carregamento (skeleton), erro de rede (mensagem + botão de tentar novamente), lista muito longa (paginação/scroll infinito)
- Nota de acessibilidade: contraste do status do pedido e leitura por leitor de tela do rótulo de status
