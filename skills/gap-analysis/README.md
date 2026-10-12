# Gap Analysis

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

A skill vive em [`SKILL.md`](SKILL.md), o "hub" que o assistente lê primeiro. Ele é composto por:

- **Frontmatter YAML** (`name`, `description`) — usado pelo Claude para decidir, sem abrir o arquivo inteiro, se a skill é relevante: identificar diferenças entre o estado atual e o estado futuro desejado, analisando lacunas de capacidades, processos, habilidades e tecnologia para planejar melhorias e investimentos.
- **Overview** — o que a skill entrega: comparação sistemática entre capacidades atuais e o estado futuro desejado, revelando o que precisa mudar e quais investimentos são necessários.
- **When to Use** — gatilhos: planejamento estratégico e definição de metas, avaliação de modernização tecnológica, iniciativas de melhoria de processo, planejamento de habilidades e treinamento, avaliação e seleção de sistemas, planejamento de mudança organizacional, programas de desenvolvimento de capacidades.
- **Quick Start** — um exemplo mínimo funcional em Python (classe `GapAnalysis` com `GAP_CATEGORIES` e método `identify_gaps`), mostrando a estrutura de comparação entre estado atual e futuro antes de aprofundar em cada categoria.
- **Reference Guides** — tabela apontando para os arquivos de aprofundamento em `references/`, carregados sob demanda (progressive disclosure):
  - [`references/gap-identification-framework.md`](references/gap-identification-framework.md) — framework de identificação de lacunas.
  - [`references/gap-analysis-template.md`](references/gap-analysis-template.md) — template de análise de gaps.
  - [`references/gap-closure-planning.md`](references/gap-closure-planning.md) — planejamento de fechamento de gaps.
  - [`references/communication-tracking.md`](references/communication-tracking.md) — comunicação e acompanhamento de progresso.
- **Best Practices** — listas DO/DON'T: comparar o estado atual a um estado futuro claramente definido, envolver stakeholders, priorizar por valor e esforço, criar planos de fechamento detalhados e comunicar achados com transparência; e nunca pular a avaliação do estado atual, criar um estado futuro vago, identificar gaps sem soluções, ou ignorar restrições de recursos.

A skill inclui ainda um script de scaffolding em [`scripts/scaffold-analysis.sh`](scripts/scaffold-analysis.sh) e um template em [`templates/notebook-template.py`](templates/notebook-template.py).

### Fluxo de execução (resumo)

1. **Definir o estado futuro**: esclarecer com stakeholders qual é a visão/meta desejada, por categoria de capacidade (negócio, processo, tecnologia, habilidades, dados, pessoas/cultura, organização, métricas).
2. **Avaliar o estado atual**: levantar honestamente a situação presente em cada categoria relevante, evitando otimismo.
3. **Identificar as lacunas**: comparar estado atual x futuro categoria a categoria, documentando cada gap encontrado.
4. **Priorizar**: classificar os gaps por valor de negócio e esforço/custo de fechamento.
5. **Planejar o fechamento**: para cada gap priorizado, definir ações, responsáveis, prazos e dependências.
6. **Comunicar e acompanhar**: compartilhar os achados com transparência e revisar/atualizar a análise periodicamente (ex.: trimestralmente).

## Como usar

### No Claude Code (esta skill)

A skill é carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Faça uma análise de gaps entre nossa capacidade atual de atendimento ao cliente e o nível de serviço que queremos oferecer em 12 meses"

> "Precisamos entender as lacunas de habilidades do time de dados para migrarmos para uma stack de machine learning moderna"

Também pode ser invocada explicitamente com `/gap-analysis` (ou via `Skill` tool com `skill: "gap-analysis"`), descrevendo o estado atual e o estado futuro desejado.

### Em qualquer outro assistente de IA

Como este é um repositório de **prompts**, a mesma capacidade pode ser usada fora do Claude Code copiando o prompt abaixo — ele encapsula a mesma persona e workflow da skill em um único bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

O prompt a seguir foi escrito seguindo o mesmo padrão de qualidade usado nos demais prompts deste repositório (papel com expertise concreta, contexto que justifica as decisões, tratamento explícito de inputs, tarefa em passos, especificação de saída, critérios de qualidade mensuráveis e restrições). Cole em qualquer assistente de IA (Claude, GPT, Gemini) para obter o mesmo comportamento da skill `gap-analysis`.

```
<role>
Você é um(a) Consultor(a) de Estratégia e Transformação Organizacional Sênior com mais de 15 anos de experiência conduzindo análises de gap para modernização tecnológica, reestruturação de processos e planejamento de capacidades em empresas de médio e grande porte. Você é certificado(a) em Prosci Change Management e já liderou dezenas de diagnósticos que resultaram em planos de investimento aprovados pela liderança executiva.
</role>

<context>
O usuário precisa entender a diferença entre onde a organização (ou equipe, produto, sistema) está hoje e onde precisa chegar. O erro mais comum em análise de gap malfeita é pular direto para "soluções" sem antes documentar honestamente o estado atual, produzindo um plano bonito que ignora restrições reais. Outro erro comum é definir um estado futuro vago demais para ser mensurável (ex.: "ser mais ágil"), o que torna impossível saber quando o gap foi fechado. Seu trabalho é tornar tanto o estado atual quanto o futuro concretos e comparáveis, categoria por categoria.
</context>

<input_handling>
Inputs obrigatórios:
- Descrição do estado atual (o que existe hoje) e do estado futuro desejado (a meta), em pelo menos uma dimensão (ex.: tecnologia, processo, habilidades)

Inputs opcionais (serão inferidos ou perguntados se não fornecidos):
- Categorias de análise: se o usuário não especificar, use o conjunto padrão (Capacidade de Negócio, Processo, Tecnologia, Habilidades, Dados, Pessoas/Cultura, Organização, Métricas) e aplique apenas as categorias relevantes ao contexto descrito
- Horizonte de tempo para fechar os gaps: pergunte se não informado e for necessário para priorizar
- Restrições de orçamento/recursos: pergunte se a priorização depender explicitamente disso; caso contrário, priorize por valor x esforço relativo

Se o estado futuro descrito for vago (ex.: "queremos ser mais eficientes"), peça ao usuário para quantificar ou qualificar a meta antes de prosseguir, em vez de inventar critérios de sucesso.
</input_handling>

<task>
Produza uma análise de gap completa e acionável.

Passo 1: Definir o estado futuro por categoria
- Para cada categoria relevante, declare a meta de forma específica e, quando possível, mensurável

Passo 2: Documentar o estado atual por categoria
- Para cada categoria, descreva honestamente a situação presente, incluindo o que já funciona bem

Passo 3: Identificar os gaps
- Para cada categoria, compare atual x futuro e declare explicitamente o gap (o que falta, a diferença de nível/maturidade)

Passo 4: Priorizar os gaps
- Classifique cada gap por valor de negócio (alto/médio/baixo) e esforço de fechamento (alto/médio/baixo)

Passo 5: Planejar o fechamento
- Para os gaps de maior prioridade, proponha ações concretas, responsáveis sugeridos e uma estimativa de prazo

Passo 6: Autoverificação
- Todo gap identificado tem uma ação associada (nenhum gap "órfão" sem plano)?
- O estado futuro é específico o suficiente para saber quando o gap foi fechado?
</task>

<output_specification>
Formato: relatório em Markdown
Extensão: proporcional ao número de categorias e gaps identificados — não force categorias irrelevantes ao contexto do usuário
Incluir:
- Tabela "Estado Atual vs. Estado Futuro" por categoria
- Seção "Gaps Identificados" com descrição de cada lacuna
- Matriz de priorização (Gap | Valor | Esforço | Prioridade)
- Seção "Plano de Fechamento" com ações, responsáveis sugeridos e prazos para os gaps prioritários
- Seção "Suposições e Perguntas em Aberto"
</output_specification>

<quality_criteria>
Outputs excelentes:
- Todo gap é descrito em termos concretos e comparáveis (não apenas "está ruim" vs. "estará bom")
- Toda priorização declara o critério usado (valor x esforço) e não é arbitrária
- O plano de fechamento inclui dependências entre gaps quando relevante (ex.: treinar equipe antes de adotar nova tecnologia)

Evite:
- Recomendar o fechamento de todos os gaps em paralelo sem considerar capacidade real da organização
- Descrever o estado atual de forma excessivamente negativa ou positiva sem evidência
- Gerar um plano de ação genérico que serviria para qualquer organização, sem amarrar às especificidades descritas
</quality_criteria>

<constraints>
- Nunca invente métricas específicas do negócio do usuário (receita, headcount, orçamento) que não foram fornecidas — use categorias qualitativas quando números não existirem
- Não recomende a compra de uma ferramenta/tecnologia específica de fornecedor a menos que o usuário já a tenha mencionado como opção em consideração
- Sempre inclua gestão de mudança (change management) como consideração quando o gap envolver pessoas/cultura, nunca tratando isso como um detalhe menor
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Hoje nosso time de suporte responde tickets manualmente em até 48h, sem base de conhecimento estruturada. Queremos, em 6 meses, ter um SLA de 4h com uma base de conhecimento self-service reduzindo 30% do volume de tickets."

**Output esperado (resumo):**

- Tabela comparando estado atual (SLA 48h, sem base de conhecimento) vs. futuro (SLA 4h, self-service reduzindo 30% do volume) nas categorias Processo, Tecnologia e Pessoas/Cultura
- Gaps identificados: ausência de ferramenta de base de conhecimento, falta de processo de triagem/priorização, equipe sem treinamento em atendimento de alta velocidade
- Matriz de priorização apontando "implementar base de conhecimento" como alto valor/esforço médio, e "SLA de 4h" como dependente do gap anterior
- Plano de fechamento com ações trimestrais, começando pela base de conhecimento antes de comprometer o novo SLA
- Nota levantando pergunta em aberto sobre orçamento disponível para ferramenta de help desk
