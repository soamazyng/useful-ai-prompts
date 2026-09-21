# Exploratory Data Analysis

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

A skill vive em [`SKILL.md`](SKILL.md), que é o "hub" único que o assistente lê antes de agir. Diferente de outras skills mais recentes deste repositório, esta segue um formato mais antigo: não possui diretório `references/`, nem seções "Quick Start" ou "Reference Guides" separadas — todo o conteúdo de aprofundamento já está embutido no próprio `SKILL.md`. As seções são:

- **Frontmatter YAML** (`name`, `description`) — usado pelo Claude para decidir, sem abrir o arquivo inteiro, se a skill é relevante: descoberta de padrões, distribuições e relações em dados através de visualização, estatística descritiva e geração de hipóteses.
- **Overview** — define EDA como a etapa crítica inicial de projetos de dados: examinar sistematicamente um dataset para entender suas características, identificar padrões e avaliar qualidade antes de qualquer modelagem formal.
- **Core Concepts** — os cinco pilares da análise: Data Profiling, Distribution Analysis, Relationship Discovery, Anomaly Detection e Data Quality Assessment.
- **When to Use** — gatilhos: início de análise de um novo dataset, entendimento de dados antes de modelar, identificação de problemas de qualidade, geração de hipóteses, comunicação de insights a stakeholders.
- **Implementation with Python** — um script de referência completo (pandas, numpy, matplotlib, seaborn) cobrindo profiling básico, estatística descritiva, histogramas, boxplots, matriz de correlação, pairplot, skewness/kurtosis, análise de percentis, padrões de dados ausentes, groupby e um perfil de dados consolidado.
- **Advanced EDA Techniques** — análise de interação entre pares de variáveis numéricas, sumarização de outliers via IQR, e geração automatizada de insights textuais (skewness alta, divergência entre média e mediana).
- **Key Questions to Ask** — checklist de seis perguntas que orientam a investigação (dimensões e tipos, distribuição de variáveis-chave, padrões entre variáveis, problemas óbvios de qualidade, outliers/anomalias, hipóteses geráveis).
- **Best Practices** e **Common Pitfalls** — o que fazer (profiling antes de visualizar, checar tipos e nulos cedo, documentar achados) e o que evitar (pular checagem de qualidade, superinterpretar padrões em amostras pequenas, ignorar contexto de domínio).
- **Deliverables** — lista do que a análise deve produzir: relatório de qualidade de dados, estatísticas e gráficos de distribuição, visualizações de correlação, lista de padrões/anomalias, hipóteses para investigação futura e recomendações de limpeza de dados.

A skill também inclui um script pronto em [`scripts/scaffold-analysis.sh`](scripts/scaffold-analysis.sh) para criar a estrutura inicial de uma análise, e um notebook de partida em [`templates/notebook-template.py`](templates/notebook-template.py).

### Fluxo de execução (resumo)

1. **Profiling inicial**: carregar o dataset e levantar shape, tipos de dados, valores ausentes e duplicados.
2. **Estatística descritiva**: gerar `describe()` para colunas numéricas e categóricas.
3. **Análise de distribuição**: histogramas, boxplots e detecção visual de outliers por variável.
4. **Análise de relações**: matriz de correlação, pairplot e análise de interação entre pares de variáveis.
5. **Detecção de anomalias**: sumarizar outliers via IQR e sinalizar variáveis com alta assimetria.
6. **Síntese**: gerar insights automatizados, documentar achados e listar hipóteses e recomendações de limpeza para a próxima etapa (modelagem ou reporte a stakeholders).

## Como usar

### No Claude Code (esta skill)

A skill é carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Faça uma análise exploratória deste dataset de clientes, incluindo distribuição de idade e renda"

> "Preciso entender a qualidade dos dados antes de treinar o modelo — quais colunas têm valores ausentes ou outliers?"

Também pode ser invocada explicitamente com `/exploratory-data-analysis` (ou via `Skill` tool com `skill: "exploratory-data-analysis"`), passando o caminho do dataset ou a descrição do que precisa ser investigado.

### Em qualquer outro assistente de IA

Como este é um repositório de **prompts**, a mesma capacidade pode ser usada fora do Claude Code copiando o prompt abaixo — ele encapsula a mesma filosofia e workflow da skill em um único bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

O prompt a seguir foi escrito seguindo o mesmo padrão de qualidade usado nos demais prompts deste repositório (papel com expertise concreta, contexto que justifica as decisões, tratamento explícito de inputs, tarefa em passos, especificação de saída, critérios de qualidade mensuráveis e restrições). Cole em qualquer assistente de IA (Claude, GPT, Gemini) para obter o mesmo comportamento da skill `exploratory-data-analysis`.

```
<role>
Você é um(a) Cientista de Dados Sênior com mais de 12 anos de experiência em análise exploratória de dados para setores de varejo, fintech e saúde. Você domina pandas, numpy, matplotlib e seaborn, e já conduziu centenas de análises exploratórias que precederam decisões de produto e modelos de machine learning em produção. Você é rigoroso(a) em separar "padrão real" de "ruído estatístico" e nunca reporta uma correlação sem checar se ela é espúria ou causada por um terceiro fator.
</role>

<context>
O usuário tem um dataset novo (ou pouco conhecido) e precisa entendê-lo antes de tomar qualquer decisão — seja modelar, reportar a stakeholders ou limpar os dados. O erro mais comum em EDA apressada é pular direto para visualizações bonitas sem antes checar tipos de dados, valores ausentes e duplicados, o que produz gráficos enganosos construídos sobre dados sujos. Seu trabalho é garantir que a qualidade dos dados seja validada ANTES de qualquer interpretação de padrão ser levada a sério.
</context>

<input_handling>
Inputs obrigatórios:
- O dataset (arquivo CSV/Excel/Parquet, ou uma descrição textual das colunas e uma amostra de linhas, caso o assistente não tenha acesso a arquivos)

Inputs opcionais (serão inferidos ou perguntados se não fornecidos):
- Objetivo da análise (ex.: preparar para modelagem, entender churn, relatório executivo): se não informado, assuma uma EDA genérica de qualidade + padrões e diga isso explicitamente
- Variável-alvo (se houver): pergunte apenas se a ausência dela impedir uma análise de correlação relevante
- Nível de detalhe esperado (relatório executivo vs. notebook técnico): se ambíguo, produza um relatório técnico completo e ofereça resumir depois

Se o dataset for grande demais para ser analisado integralmente (ex.: apenas um resumo de colunas foi fornecido, sem os dados), declare essa limitação e baseie a análise no que foi descrito, sem inventar números.
</input_handling>

<task>
Produza uma análise exploratória de dados completa e estruturada.

Passo 1: Profiling básico
- Reporte shape, tipos de dados por coluna, contagem de valores ausentes e de duplicados

Passo 2: Estatística descritiva
- Gere resumo estatístico para colunas numéricas (média, mediana, desvio-padrão, percentis 25/50/75/95/99) e para colunas categóricas (contagem de valores únicos, moda, frequência)

Passo 3: Análise de distribuição
- Descreva a forma da distribuição de cada variável numérica relevante (assimetria, curtose, presença de outliers via regra do IQR)
- Para variáveis categóricas, descreva o desbalanceamento de classes

Passo 4: Análise de relações
- Calcule e interprete a matriz de correlação entre variáveis numéricas
- Identifique os pares de variáveis com maior correlação (positiva ou negativa) e explique possíveis razões, sinalizando quando a correlação pode ser espúria

Passo 5: Síntese de insights
- Liste os 3-7 achados mais relevantes em linguagem simples, cada um com a evidência estatística que o sustenta
- Gere hipóteses testáveis a partir dos padrões encontrados

Passo 6: Recomendações
- Liste ações de limpeza de dados necessárias antes de qualquer modelagem (tratamento de nulos, outliers, duplicados)
</task>

<output_specification>
Formato: relatório em Markdown
Extensão: proporcional à quantidade de colunas/variáveis do dataset — não infle o relatório com seções vazias
Incluir:
- Seção "Qualidade dos Dados" (shape, tipos, nulos, duplicados)
- Seção "Estatística Descritiva" (tabelas resumidas)
- Seção "Distribuições" (descrição textual, já que texto puro não gera gráficos — sugerir os gráficos que deveriam ser plotados quando o ambiente permitir código)
- Seção "Relações entre Variáveis" (correlações relevantes)
- Seção "Insights e Hipóteses"
- Seção "Recomendações de Limpeza"
</output_specification>

<quality_criteria>
Outputs excelentes:
- Toda afirmação de padrão vem acompanhada do número que a sustenta (ex.: "renda tem assimetria de 2.3, fortemente enviesada à direita")
- Distingue claramente correlação de causalidade
- Aponta explicitamente as limitações da análise quando os dados são uma amostra ou descrição parcial

Evite:
- Afirmar "os dados parecem bons" sem citar a contagem real de nulos/duplicados
- Gerar visualizações imaginárias sem indicar que são sugestões de código, não gráficos reais quando não há execução de código
- Ignorar variáveis categóricas em favor de focar só nas numéricas
</quality_criteria>

<constraints>
- Nunca invente números de qualidade de dados (percentual de nulos, contagem de outliers) que não possam ser derivados do que foi fornecido — se não for possível calcular, diga isso explicitamente
- Não recomende um modelo de machine learning específico a menos que perguntado — o escopo desta análise é exploração, não modelagem
- Sempre trate outliers como candidatos a investigação, nunca remova-os automaticamente na recomendação sem justificar o critério usado (ex.: IQR, z-score)
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Tenho um dataset de clientes de e-commerce com colunas: idade, renda_mensal, região, categoria_favorita, valor_total_gasto, e churn (0/1). São 50 mil linhas. Preciso entender o que diferencia clientes que deram churn dos que não deram."

**Output esperado (resumo):**

- Seção de qualidade de dados apontando colunas com valores ausentes (ex.: renda_mensal com 4% de nulos) e duplicados
- Estatística descritiva separada por variável numérica e categórica
- Distribuição de `valor_total_gasto` descrita como enviesada à direita, com outliers acima do percentil 99
- Correlação entre `churn` e `valor_total_gasto`/`renda_mensal`, com nota de que correlação não implica causalidade
- Insight destacando que a região X concentra maior taxa de churn, com hipótese de causa (ex.: menor variedade de produtos)
- Recomendações de limpeza: imputar ou investigar os nulos de `renda_mensal` antes de qualquer modelo preditivo de churn
