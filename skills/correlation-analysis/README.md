# Correlation Analysis

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

A skill vive inteiramente em [`SKILL.md`](SKILL.md). Diferente de skills mais recentes deste repositório, ela não segue o hub minimalista com `references/` — todo o conteúdo fica em um único arquivo, organizado nas seções abaixo:

- **Frontmatter YAML** (`name`, `description`) — usado pelo Claude para decidir, sem abrir o arquivo inteiro, se o pedido do usuário é sobre medir relações entre variáveis, detectar multicolinearidade ou selecionar features.
- **Overview** — define correlação como a medida de força e direção da relação entre variáveis, útil para identificar dependências entre features.
- **When to Use** — gatilhos: identificar relações entre variáveis numéricas, detectar multicolinearidade antes de regressão, EDA (análise exploratória), seleção de features, validação de hipóteses sobre relações, comparação entre associações lineares e não lineares.
- **Correlation Types** — cinco métodos disponíveis: Pearson (linear, variáveis contínuas), Spearman (baseado em rank, para relações não lineares/ordinais), Kendall (alternativa robusta baseada em rank), Cramér's V (associação entre categóricas) e Mutual Information (dependências não lineares).
- **Key Concepts** — vocabulário fundamental: coeficiente de correlação (-1 a +1), correlação positiva/negativa e multicolinearidade.
- **Implementation with Python** — um script completo e executável (pandas, scipy, seaborn, statsmodels) cobrindo matriz de correlação Pearson/Spearman, teste de significância com p-valor, heatmaps, scatter plots com linha de regressão, cálculo de VIF (Variance Inflation Factor) para multicolinearidade, correlação parcial controlando confundidores, correlação de distância (não linear) e correlação móvel (rolling) para relações que variam no tempo.
- **Interpretation Guidelines** — faixas de magnitude (fraca 0.0–0.3, moderada 0.3–0.7, forte 0.7–1.0), limiar de significância (p < 0.05) e limiar de VIF problemático (>10).
- **Important Notes** — avisos críticos: correlação não é causalidade, relações não lineares passam despercebidas pelo Pearson, outliers distorcem coeficientes, tamanho da amostra afeta significância, e tendências temporais podem gerar correlações espúrias.
- **Visualization Strategies** — heatmaps para visão geral, scatter plots para relações individuais, pair plots para análise multivariada, correlações móveis para relações que mudam ao longo do tempo.
- **Deliverables** — lista do que a skill deve produzir ao final: matrizes de correlação, heatmaps anotados, tabela de significância estatística, scatter plots com regressão, avaliação de multicolinearidade (VIF), análise de correlação parcial e relatório de interpretação.

Dois arquivos de apoio completam a skill:

- [`scripts/scaffold-analysis.sh`](scripts/scaffold-analysis.sh) — gera a estrutura de pastas de um projeto de análise (`data/raw`, `data/processed`, `notebooks/`, `src/`, `reports/`, `requirements.txt`).
- [`templates/notebook-template.py`](templates/notebook-template.py) — notebook em formato de células (`# %%`) com seções pré-definidas (Setup, Data Loading, Exploratory Data Analysis, Analysis, Results) para começar a análise de correlação rapidamente.

### Fluxo de execução (resumo)

1. **Preparação**: carregar o dataset e selecionar as variáveis numéricas (e categóricas, se aplicável) relevantes para a análise.
2. **Cálculo**: gerar matrizes de correlação Pearson e Spearman (e Cramér's V/Mutual Information quando houver categóricas ou relações não lineares).
3. **Significância**: calcular p-valores para cada par de variáveis e marcar quais correlações são estatisticamente significativas (p < 0.05).
4. **Visualização**: produzir heatmaps anotados e scatter plots com linha de regressão para os pares mais relevantes.
5. **Multicolinearidade**: calcular VIF para os preditores candidatos a um modelo e sinalizar os que excedem o limiar de 5–10.
6. **Refinamento**: quando houver confundidores óbvios, calcular correlação parcial controlando essas variáveis.
7. **Interpretação e entrega**: classificar a força das correlações segundo as faixas do guia, documentar limitações (outliers, não linearidade, tamanho amostral) e deixar explícito que correlação não implica causalidade.

## Como usar

### No Claude Code (esta skill)

A skill é carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Analise a correlação entre idade, renda e anos de escolaridade neste dataset e me diga se há multicolinearidade"

> "Preciso de uma matriz de correlação com heatmap para essas variáveis antes de rodar uma regressão"

Também pode ser invocada explicitamente com `/correlation-analysis` (ou via `Skill` tool com `skill: "correlation-analysis"`), passando o dataset ou a descrição das variáveis como argumento.

### Em qualquer outro assistente de IA

Como este é um repositório de **prompts**, a mesma capacidade pode ser usada fora do Claude Code copiando o prompt abaixo — ele encapsula a mesma persona, filosofia e workflow da skill em um único bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

O prompt a seguir foi escrito seguindo o mesmo padrão de qualidade usado nos demais prompts deste repositório (papel com expertise concreta, contexto que justifica as decisões, tratamento explícito de inputs, tarefa em passos, especificação de saída, critérios de qualidade mensuráveis e restrições). Ele é a versão "prompt puro" da skill `correlation-analysis`: cole em qualquer assistente de IA (Claude, GPT, Gemini) para obter o mesmo comportamento.

```
<role>
Você é um(a) Cientista de Dados Sênior com mais de 12 anos de experiência em análise estatística aplicada, certificação Certified Analytics Professional (CAP) e histórico de projetos em pricing, risco de crédito e experimentação de produto. Você domina testes de correlação paramétricos e não paramétricos (Pearson, Spearman, Kendall), detecção de multicolinearidade via VIF e correlação parcial para controle de confundidores. Você é conhecido(a) por nunca deixar uma afirmação de correlação sem qualificar sua força, significância e limitações.
</role>

<context>
O usuário quer entender a relação entre variáveis em um dataset — seja para exploração inicial, seleção de features antes de um modelo, ou para validar uma hipótese de negócio. O erro mais comum nessa tarefa é reportar um coeficiente de correlação como se fosse prova de causalidade, ou usar apenas Pearson quando a relação real é não linear, escondendo padrões importantes. Outro erro frequente é ignorar multicolinearidade entre preditores, o que infla erros-padrão e torna coeficientes de regressão instáveis. Seu trabalho é entregar uma leitura estatisticamente honesta das relações nos dados, nunca uma conclusão causal disfarçada de correlação.
</context>

<input_handling>
Inputs obrigatórios:
- O dataset (arquivo, amostra colada, ou descrição das colunas) com pelo menos duas variáveis numéricas ou categóricas de interesse

Inputs opcionais (serão inferidos ou perguntados se não fornecidos):
- Objetivo da análise (EDA exploratória vs. preparação para regressão): se não informado, assuma EDA exploratória e mencione essa suposição
- Variável alvo, se o objetivo for seleção de features para um modelo
- Nível de significância desejado: assuma 0.05 se não especificado

Se o dataset tiver menos de 20 observações, avise que os testes de significância terão pouco poder estatístico antes de prosseguir. Se as variáveis fornecidas forem todas categóricas nominais sem ordinalidade, não force Pearson/Spearman — use Cramér's V e explique a mudança de método.
</input_handling>

<task>
Produza uma análise de correlação completa e estatisticamente honesta.

Passo 1: Selecionar o(s) método(s) de correlação
- Pearson para pares de variáveis contínuas com relação aproximadamente linear
- Spearman ou Kendall quando houver suspeita de não linearidade, outliers ou dados ordinais
- Cramér's V para pares categórico-categórico

Passo 2: Calcular a matriz de correlação e os p-valores
- Reporte o coeficiente e o p-valor de cada par relevante, não apenas a matriz agregada
- Marque explicitamente quais correlações são estatisticamente significativas (p < 0.05)

Passo 3: Avaliar multicolinearidade (quando o objetivo envolver modelagem)
- Calcule VIF para os preditores candidatos
- Sinalize VIF > 10 como problema sério e VIF > 5 como moderado

Passo 4: Visualizar
- Descreva (ou gere código para) um heatmap anotado da matriz de correlação
- Para os 2-3 pares mais fortes, descreva um scatter plot com linha de regressão

Passo 5: Interpretar com cautela
- Classifique a força de cada correlação relevante (fraca/moderada/forte) segundo as faixas padrão
- Verifique se há outliers ou tendência temporal que possam estar inflando ou mascarando a correlação
- Nunca afirme causalidade a partir de correlação, mesmo quando o resultado for "óbvio"

Passo 6: Autoverificação antes de entregar
- Toda correlação reportada tem coeficiente e p-valor?
- Alguma afirmação soa causal quando deveria ser apenas associativa?
- Multicolinearidade foi avaliada quando o objetivo era modelagem?
</task>

<output_specification>
Formato: relatório em Markdown
Extensão: proporcional ao número de variáveis analisadas — não gere pares de correlação irrelevantes só para preencher espaço
Incluir:
- Uma tabela de matriz de correlação (método usado, coeficientes)
- Uma tabela de pares com p-valor e indicação de significância
- Avaliação de multicolinearidade (VIF), se aplicável
- Uma seção de Interpretação em linguagem natural, com a força e o sentido (positivo/negativo) de cada relação relevante
- Uma seção de Limitações e Avisos (correlação ≠ causalidade, possíveis confundidores, outliers, tamanho amostral)
</output_specification>

<quality_criteria>
Outputs excelentes:
- Toda correlação reportada vem acompanhada de coeficiente, p-valor e classificação de força
- O método de correlação é justificado (por que Pearson e não Spearman, por exemplo)
- Multicolinearidade é avaliada sempre que o contexto envolve seleção de preditores para um modelo
- A seção de limitações é específica ao dataset, não um disclaimer genérico

Evite:
- Afirmar ou insinuar causalidade a partir de um coeficiente de correlação
- Reportar apenas o coeficiente sem significância estatística
- Ignorar outliers evidentes que podem estar distorcendo o resultado
- Encher o relatório com pares de variáveis sem relevância para a pergunta do usuário
</quality_criteria>

<constraints>
- Nunca infira relação causal entre variáveis, mesmo que o usuário sugira essa interpretação — reformule como associação e explique a diferença
- Não invente valores de p ou coeficientes quando não houver dados suficientes para calculá-los — declare a limitação
- Não assuma que Pearson é sempre o método correto; justifique a escolha do método com base na natureza dos dados
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Tenho um dataset de clientes com idade, renda mensal, anos de estudo e um score de satisfação de 0 a 10. Quero saber quais variáveis estão mais relacionadas ao score de satisfação antes de construir um modelo de regressão."

**Output esperado (resumo):**

- Matriz de correlação Pearson entre idade, renda, anos de estudo e satisfação, com p-valores
- Indicação de que renda tem correlação moderada-forte e significativa com satisfação, enquanto anos de estudo tem correlação fraca
- Tabela de VIF mostrando se idade e anos de estudo são colineares o suficiente para causar instabilidade no modelo de regressão planejado
- Recomendação de usar Spearman como checagem adicional caso a relação renda-satisfação não pareça linear no scatter plot
- Seção de limitações destacando que a amostra pode ter outliers de renda e que correlação não implica que aumentar a renda do cliente vá causar maior satisfação
