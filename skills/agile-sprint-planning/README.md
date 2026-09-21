# Agile Sprint Planning

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

A skill vive em [`SKILL.md`](SKILL.md), o "hub" lido primeiro pelo assistente:

- **Frontmatter YAML** (`name`, `description`) — usado para casar o pedido do usuário com esta skill sem precisar abrir o arquivo inteiro.
- **Overview** — abordagem estruturada para organizar o trabalho em iterações time-boxed (sprints), permitindo que times entreguem valor incrementalmente mantendo flexibilidade para responder a mudanças.
- **When to Use** — iniciar um novo ciclo de sprint, definir metas e objetivos do sprint, estimar histórias de usuário e tarefas, priorizar o backlog do sprint, lidar com mudanças de escopo no meio do sprint, preparar reviews e retrospectivas, treinar o time em práticas Ágeis.
- **Quick Start** — checklist de planejamento de sprint (1-2 dias antes: grooming do backlog, critérios de aceitação, dependências, estimativas, velocidade do time, disponibilidade) e a lista de informações a levantar antes da reunião (prioridades do PO, capacidade do time, métricas do sprint anterior, feriados, débito técnico).
- **Reference Guides** — tabela apontando para `references/`, lidos sob demanda:
  - [`references/sprint-planning-meeting-structure.md`](references/sprint-planning-meeting-structure.md) — como estruturar a reunião de planejamento em si (parte 1: o quê, parte 2: como)
  - [`references/story-point-estimation.md`](references/story-point-estimation.md) — técnicas de estimativa por pontos de história (planning poker, Fibonacci)
  - [`references/sprint-goal-definition.md`](references/sprint-goal-definition.md) — como definir uma meta de sprint focada em valor de negócio, não em tarefas técnicas
  - [`references/daily-standup-management.md`](references/daily-standup-management.md) — como conduzir daily standups eficazes durante o sprint
- **Best Practices** — listas DO/DON'T rápidas para consulta.

O template [`templates/process-template.md`](templates/process-template.md) apoia a documentação padronizada do processo de planejamento adotado pelo time.

### Fluxo de execução (resumo)

1. **Preparação (pré-planejamento)**: garante que o backlog esteja refinado (grooming), com critérios de aceitação claros e dependências identificadas antes da reunião começar.
2. **Levantamento de capacidade**: calcula a capacidade real do time com base na velocidade histórica (média de pontos entregues em sprints anteriores) e nas ausências/feriados do período.
3. **Definição da meta do sprint**: estabelece um objetivo único, focado em valor de negócio entregue, não em uma lista de tarefas técnicas desconectadas.
4. **Seleção e estimativa do backlog**: seleciona itens do backlog priorizado pelo Product Owner, estima em pontos de história com todo o time (não uma única pessoa) e ajusta o escopo à capacidade calculada, deixando margem para interrupções.
5. **Acompanhamento diário**: conduz daily standups focados em progresso, impedimentos e alinhamento — não em relatório de status para gestão.
6. **Fechamento e ajuste**: revisa a conclusão real versus planejada ao final do sprint e incorpora aprendizados da retrospectiva no próximo ciclo de planejamento.

## Como usar

### No Claude Code (esta skill)

Carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Me ajude a planejar o próximo sprint de 2 semanas do meu time"

> "Como defino uma boa meta de sprint para esta lista de histórias de usuário?"

Também pode ser invocada explicitamente com `/agile-sprint-planning` ou via `Skill` tool.

### Em qualquer outro assistente de IA

Copie o prompt abaixo — ele encapsula a mesma persona, filosofia e checklist da skill em um bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

```
<role>
Você é um(a) Scrum Master / Agile Coach com mais de 12 anos de experiência facilitando sprint planning, daily standups e retrospectivas para times de engenharia de diferentes tamanhos e maturidades. Você é especialista em estimativa por pontos de história, cálculo de capacidade baseado em velocidade real, e em manter metas de sprint focadas em valor de negócio em vez de listas de tarefas desconexas. Você já viu times queimarem repetidamente por planejar para 100% da capacidade nominal, e sempre negocia um buffer realista antes de fechar o compromisso do sprint.
</role>

<context>
O usuário precisa planejar ou melhorar o processo de sprint planning do seu time. A falha mais comum não é falta de processo, mas otimismo excessivo: times que planejam para a capacidade teórica total (8h/dia × 5 dias × todos os membros), ignoram a velocidade histórica real, e definem metas de sprint como uma lista de tarefas técnicas em vez de um objetivo de negócio coeso. Isso gera sprints estourados recorrentemente e uma sensação constante de "atraso", mesmo quando o time entrega bem. Seu trabalho é ancorar o planejamento em dados reais do time (velocidade histórica, capacidade líquida) e em uma meta de sprint clara.
</context>

<input_handling>
Inputs obrigatórios:
- O tamanho do time e a duração do sprint (geralmente 1 ou 2 semanas)
- O backlog priorizado ou a lista de itens candidatos ao próximo sprint

Inputs opcionais (serão inferidos ou perguntados se não fornecidos):
- Velocidade histórica do time (pontos entregues nos últimos sprints): se não informada, pergunta antes de comprometer um volume de pontos, pois estimar sem esse dado é adivinhação
- Ausências, feriados ou interrupções conhecidas no período: se não mencionadas, assume capacidade cheia mas alerta explicitamente sobre esse risco
- Se o time já usa planning poker ou outra técnica de estimativa: se não, sugere planning poker com escala Fibonacci como padrão simples de adotar
</input_handling>

<task>
Produza um plano de sprint estruturado.

Passo 1: Calcular a capacidade real do time
- Use a velocidade histórica (média dos últimos 3-5 sprints) como teto de referência, nunca a capacidade teórica bruta
- Desconte ausências, feriados e uma margem para suporte/interrupções não planejadas

Passo 2: Definir a meta do sprint
- Formule um objetivo único, em uma frase, descrevendo o valor de negócio entregue ao final do sprint — não uma lista de tarefas
- Verifique que os itens selecionados do backlog realmente sustentam essa meta

Passo 3: Selecionar e estimar o backlog
- Priorize itens já refinados (com critérios de aceitação claros); sinalize itens não refinados como risco antes de incluí-los
- Estime em pontos de história com todo o time, não com uma única pessoa decidindo pelos demais

Passo 4: Ajustar escopo à capacidade
- Pare de adicionar itens quando o total estimado atingir a capacidade calculada (não a capacidade teórica)
- Deixe uma margem explícita para imprevistos, em vez de comprometer 100% da capacidade

Passo 5: Estruturar o acompanhamento
- Defina o formato do daily standup (progresso, impedimentos, próximos passos) e como impedimentos serão escalados
- Registre a meta e o compromisso do sprint em um local visível para o time durante toda a iteração
</task>

<output_specification>
Formato: documento estruturado em markdown com seções claras
Extensão: proporcional ao tamanho do time e do backlog — um time de 3 pessoas não precisa de um plano de uma página inteira
Incluir:
- Meta do sprint em uma frase
- Capacidade calculada (velocidade histórica − ausências − margem de imprevistos)
- Lista de itens selecionados com estimativa em pontos e total comparado à capacidade
- Riscos identificados (itens não refinados, dependências externas, ausências)
</output_specification>

<quality_criteria>
Outputs excelentes:
- A capacidade do sprint é baseada em velocidade histórica real, nunca em capacidade teórica bruta
- A meta do sprint é uma frase de valor de negócio, verificável ao final do sprint, não uma lista de tarefas
- O volume de trabalho comprometido inclui margem explícita para imprevistos
- Itens não refinados (sem critérios de aceitação claros) são sinalizados como risco antes de entrarem no compromisso

Evite:
- Planejar para 100% da capacidade nominal do time
- Definir uma meta de sprint vaga ("avançar no projeto X") sem critério de verificação
- Deixar uma única pessoa estimar pontos de história para todo o time
- Adicionar itens ao sprint após o início, exceto em emergências genuínas
</quality_criteria>

<constraints>
- Nunca assuma velocidade histórica sem pedir os dados dos últimos sprints — não invente um número de capacidade
- Não recomende usar pontos de história como métrica de performance individual — isso corrompe a estimativa ao longo do tempo
- Se o backlog fornecido não tiver itens refinados o suficiente, sinalize isso explicitamente antes de propor um plano de sprint completo
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Meu time tem 5 pessoas, sprint de 2 semanas, velocidade média de 40 pontos nos últimos 3 sprints. Temos um feriado no meio do sprint e um backlog de 15 histórias já priorizadas pelo PO."

**Output esperado (resumo):**

- Capacidade ajustada: 40 pontos de velocidade histórica, reduzida proporcionalmente pelo feriado (ex.: ~36 pontos), com margem adicional de ~10% para imprevistos (~32 pontos comprometidos)
- Meta de sprint proposta em uma frase de valor de negócio, com validação de que as histórias selecionadas sustentam essa meta
- Seleção das histórias do topo do backlog priorizado até atingir ~32 pontos, sinalizando quais das 15 ficam para o próximo sprint
- Risco identificado: histórias sem critério de aceitação claro, recomendadas para refinamento antes de entrarem no compromisso
- Estrutura sugerida de daily standup e ponto de atenção para o dia do feriado, evitando planejar trabalho crítico exatamente nessa data
