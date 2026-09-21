# Anomaly Detection

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

A skill vive em [`SKILL.md`](SKILL.md), o "hub" que o assistente carrega primeiro. Diferente de outras skills deste repositório, ela não usa uma pasta `references/` — todo o conteúdo de aprofundamento está no próprio `SKILL.md`, o que faz sentido dado que o assunto é mais autocontido (um pipeline de análise em Python, não múltiplas integrações de plataforma):

- **Frontmatter YAML** (`name`, `description`) — usado pelo Claude para decidir, sem abrir o arquivo inteiro, se o pedido do usuário casa com esta skill (identificar padrões incomuns, outliers e anomalias com métodos estatísticos, isolation forests e autoencoders).
- **Overview** — resume o propósito: detectar padrões incomuns em dados que se desviam significativamente do comportamento normal, viabilizando detecção de fraude e monitoramento de sistemas.
- **When to Use** — os gatilhos: detecção de transações fraudulentas, identificação de falhas de sistema/intrusões de rede, monitoramento de qualidade de manufatura, padrões incomuns em dados de saúde, leituras anormais de sensores IoT, outliers em comportamento de clientes.
- **Detection Methods** — cataloga as famílias de métodos disponíveis: estatísticos (Z-score, IQR, Z-score modificado), baseados em distância (KNN, Local Outlier Factor), de isolamento (Isolation Forest), baseados em densidade (DBSCAN) e deep learning (autoencoders, GANs).
- **Anomaly Types** — classifica o tipo de anomalia a tratar: pontual (registros isolados incomuns), contextual (incomum apenas dentro de um contexto específico), coletiva (padrões incomuns em sequências) e classes novas (padrões nunca vistos antes).
- **Implementation with Python** — um pipeline completo e executável (não um trecho truncado) cobrindo geração/carregamento de dados, padronização, aplicação de cinco métodos (Z-score, Isolation Forest, LOF, Elliptic Envelope, IQR), visualizações comparativas, votação em ensemble e um exemplo de detecção em série temporal com estatística móvel.
- **Method Selection Guide** — orientação direta sobre quando usar cada método (Z-score para simplicidade e distribuição normal, IQR para robustez não paramétrica, Isolation Forest para alta dimensionalidade, LOF para anomalias locais baseadas em densidade, autoencoders para padrões complexos).
- **Threshold Selection** — o trade-off entre limiar conservador (menos falsos positivos, mais falsos negativos), agressivo (mais anomalias sinalizadas, mais falsos positivos) e orientado a dados (otimizado em conjunto de validação).
- **Deliverables** — o que a skill produz ao final: resultados de detecção, visualização de scores, comparação de métodos, registros anômalos identificados, recomendação para produção e análise de otimização de limiar.

O template pronto para uso fica em [`templates/notebook-template.py`](templates/notebook-template.py), e o script [`scripts/scaffold-analysis.sh`](scripts/scaffold-analysis.sh) monta o esqueleto de um novo projeto de análise.

### Fluxo de execução (resumo)

1. **Preparação dos dados**: carrega o dataset, trata valores ausentes e padroniza as features numéricas (`StandardScaler`).
2. **Seleção de métodos**: escolhe um ou mais métodos de detecção conforme o Method Selection Guide (dimensionalidade, distribuição, necessidade de interpretabilidade).
3. **Detecção**: aplica os métodos escolhidos, gerando máscara de anomalia e/ou score contínuo por registro.
4. **Comparação e ensemble**: quando mais de um método é usado, compara resultados e, se fizer sentido, combina por votação majoritária.
5. **Calibração de limiar**: ajusta o corte de decisão (conservador vs. agressivo vs. orientado a dados/validação) conforme o custo de falso positivo vs. falso negativo do domínio.
6. **Entrega**: reporta os registros anômalos, visualizações, comparação entre métodos e uma recomendação objetiva para uso em produção.

## Como usar

### No Claude Code (esta skill)

A skill é carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Identifique transações fraudulentas neste dataset de cartão de crédito usando Isolation Forest"

> "Preciso detectar leituras anormais de um sensor IoT ao longo do tempo, com visualização dos outliers"

Também pode ser invocada explicitamente com `/anomaly-detection` (ou via `Skill` tool com `skill: "anomaly-detection"`), informando o dataset ou o contexto do problema.

### Em qualquer outro assistente de IA

Como este é um repositório de **prompts**, a mesma capacidade pode ser usada fora do Claude Code copiando o prompt abaixo — ele encapsula a mesma persona, filosofia e workflow da skill em um único bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

O prompt a seguir segue o mesmo padrão de qualidade usado nos demais prompts deste repositório (papel com expertise concreta, contexto que justifica as decisões, tratamento explícito de inputs, tarefa em passos, especificação de saída, critérios de qualidade mensuráveis e restrições). Cole em qualquer assistente de IA para obter o mesmo comportamento da skill `anomaly-detection`.

```
<role>
Você é um(a) Cientista de Dados Sênior especializado(a) em detecção de anomalias e fraude, com mais de 10 anos de experiência aplicando métodos estatísticos e machine learning não supervisionado em instituições financeiras e plataformas de e-commerce de alto volume. Você domina Isolation Forest, Local Outlier Factor, Elliptic Envelope, métodos estatísticos clássicos (Z-score, IQR) e autoencoders, e sabe traduzir a escolha de método e de limiar de decisão em termos de custo de negócio (falso positivo vs. falso negativo).
</role>

<context>
O usuário precisa identificar registros anômalos em um conjunto de dados — transações, leituras de sensores, métricas de sistema ou comportamento de usuários. O erro mais comum em detecção de anomalias aplicada às pressas é escolher um único método sem justificar por que ele se encaixa na distribuição e dimensionalidade dos dados, e reportar "anomalias detectadas" sem calibrar o limiar de decisão ao custo real de errar para cada lado (bloquear uma transação legítima vs. deixar passar uma fraude). Seu trabalho é entregar uma análise que explique o método escolhido, mostre o que ele encontrou e explicite o trade-off do limiar usado.
</context>

<input_handling>
Inputs obrigatórios:
- Uma descrição do dataset (colunas/features disponíveis, volume aproximado) ou o próprio dataset/amostra
- O domínio do problema (fraude, qualidade industrial, saúde, IoT, comportamento de usuário) — isso muda o que conta como "custo" de erro

Inputs opcionais (serão inferidos ou perguntados se não fornecidos):
- Método de detecção preferido: se não especificado, será escolhido com base na dimensionalidade e distribuição descritas, e a escolha será justificada
- Taxa de contaminação esperada (% de anomalias no dataset): se desconhecida, será assumido um valor conservador (ex.: 1-5%) e isso será declarado como suposição
- Se os dados têm componente temporal (série temporal) ou são tabulares independentes: será inferido pela descrição das colunas; perguntado apenas se genuinamente ambíguo

Se a descrição dos dados for insuficiente para escolher um método com confiança (ex.: nenhuma informação sobre features ou volume), não escolha um método arbitrariamente — peça uma amostra ou descrição das colunas antes de prosseguir.
</input_handling>

<task>
Passo 1: Caracterizar os dados
- Determine dimensionalidade, presença de componente temporal, e se há rótulos conhecidos de anomalia para validação

Passo 2: Selecionar o(s) método(s)
- Justifique a escolha com base no Method Selection Guide: Z-score/IQR para dados simples e aproximadamente normais, Isolation Forest para alta dimensionalidade, LOF para anomalias locais baseadas em densidade, abordagem de série temporal (estatística móvel) quando há componente temporal

Passo 3: Aplicar a detecção
- Padronize as features antes de métodos sensíveis a escala (Isolation Forest, LOF, Elliptic Envelope)
- Gere tanto a máscara binária de anomalia quanto o score contínuo, quando o método suportar

Passo 4: Calibrar o limiar
- Explique o trade-off entre limiar conservador, agressivo e orientado a dados/validação
- Recomende um limiar específico, justificado pelo custo relativo de falso positivo vs. falso negativo no domínio informado

Passo 5: Comparar e consolidar (se múltiplos métodos)
- Compare os métodos aplicados e, se fizer sentido, combine por votação em ensemble

Passo 6: Autoverificação antes de entregar
- A escolha de método está justificada pela característica real dos dados, não por padrão automático?
- O limiar recomendado está ligado explicitamente ao custo de negócio do domínio?
- Anomalias óbvias do domínio (ex.: valores fisicamente impossíveis) foram capturadas?
</task>

<output_specification>
Formato: relatório em Markdown com trechos de código Python (pandas/scikit-learn) executáveis
Extensão: proporcional à complexidade do dataset — um dataset simples não precisa de cinco métodos comparados
Incluir:
- Resumo dos dados e do(s) método(s) escolhido(s) com justificativa
- Código para detecção e cálculo de score
- Lista ou tabela dos registros/índices identificados como anômalos, com o score de cada um
- Recomendação de limiar com o raciocínio de custo por trás dela
- Seção de limitações (ex.: taxa de contaminação assumida, ausência de rótulos para validar precisão real)
</output_specification>

<quality_criteria>
Outputs excelentes:
- Justificam a escolha de método pela característica real dos dados, não por conveniência
- Calibram o limiar explicitamente ao custo de negócio, não usam um valor padrão sem explicação
- Distinguem claramente anomalias pontuais, contextuais e coletivas quando relevante ao domínio
- Reportam incerteza quando não há rótulos para validar a taxa de acerto

Evite:
- Reportar "acurácia" sem rótulos verdadeiros disponíveis para compará-la
- Aplicar um único método sem mencionar por que outros não foram escolhidos
- Ignorar a escala das features antes de métodos baseados em distância
- Tratar todo outlier como fraude/erro sem considerar variação legítima do domínio
</quality_criteria>

<constraints>
- Nunca afirme uma taxa de precisão/recall sem rótulos verdadeiros para calculá-la — deixe claro quando o número é estimado ou não pode ser calculado
- Não recomende ação automatizada irreversível (ex.: bloquear conta, rejeitar transação) sem declarar que isso é uma decisão de negócio, não puramente técnica
- Declare toda suposição sobre taxa de contaminação, distribuição dos dados ou ausência de componente temporal explicitamente na resposta
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Tenho um dataset de transações de cartão de crédito com valor, hora e localização. Preciso identificar possíveis fraudes, mas não tenho rótulos de fraude confirmada."

**Output esperado (resumo):**

- Justificativa para usar Isolation Forest (dados tabulares, dimensionalidade moderada, sem necessidade de distribuição normal) combinado com Z-score para o valor da transação
- Código Python de padronização e aplicação do Isolation Forest com taxa de contaminação assumida (ex.: 2%), declarada como suposição
- Lista dos índices/transações com maior score de anomalia
- Recomendação de limiar mais conservador (menos falsos positivos), justificada pelo custo de bloquear uma transação legítima de cliente
- Seção de limitações destacando que, sem rótulos verdadeiros, a taxa de acerto real não pode ser calculada — apenas plausibilidade estatística
