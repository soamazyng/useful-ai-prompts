# Time Series Analysis

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

A skill vive em [`SKILL.md`](SKILL.md), o único arquivo que o assistente lê. Assim como `statistical-hypothesis-testing`, esta skill usa o formato mais antigo — sem `Quick Start`/`Reference Guides` separados nem pasta `references/` — concentrando tudo em um único documento denso:

- **Frontmatter YAML** (`name: Time Series Analysis`, `description`) — usado pelo Claude para decidir se o pedido é sobre padrões temporais (tendência, sazonalidade, autocorrelação) e forecasting.
- **Overview** — resume o propósito: examinar dados coletados ao longo do tempo para identificar padrões, tendências e sazonalidade, com foco em forecasting e entendimento de dinâmicas temporais.
- **Core Components** — define o vocabulário central: Tendência, Sazonalidade, Ciclicidade, Estacionariedade e Autocorrelação.
- **Key Techniques** — cataloga as técnicas disponíveis: Decomposição, Differencing (tornar a série estacionária), ARIMA, Exponential Smoothing e SARIMA.
- **Implementation with Python** — um script Python extenso e executável (via `statsmodels`/`pandas`) cobrindo decomposição sazonal, teste de estacionariedade (Augmented Dickey-Fuller), differencing, ACF/PACF, modelo ARIMA com forecast e intervalo de confiança, Exponential Smoothing, médias móveis em múltiplas janelas, teste de causalidade de Granger, força sazonal e forecasts multi-horizonte.
- **Stationarity** — explica a diferença entre série estacionária e não estacionária, e as soluções (differencing, transformação logarítmica, detrending).
- **Model Selection** — orientação de quando usar ARIMA (univariado), SARIMA (com sazonalidade), Exponential Smoothing (mais simples, bom para tendências) ou Prophet (feriados e changepoints).
- **Evaluation Metrics** — MAE, RMSE e MAPE como métricas de avaliação de forecast.
- **Deliverables** — o que a análise final deve conter: gráficos de decomposição, resultados de teste de estacionariedade, gráficos ACF/PACF, modelos ajustados com diagnósticos, forecast com intervalo de confiança e comparação de métricas de acurácia.

A pasta [`scripts/`](scripts/) contém [`scaffold-analysis.sh`](scripts/scaffold-analysis.sh) para estruturar um novo projeto de análise, e [`templates/notebook-template.py`](templates/notebook-template.py) como ponto de partida de notebook/script.

### Fluxo de execução (resumo)

1. **Visualizar a série bruta**: entender visualmente tendência, sazonalidade aparente e outliers antes de qualquer modelagem.
2. **Testar estacionariedade**: rodar o teste Augmented Dickey-Fuller; se não estacionária, aplicar differencing (ou transformação) até estacionarizar.
3. **Decompor a série**: separar em componentes de tendência, sazonalidade e resíduo para entender a estrutura subjacente.
4. **Analisar autocorrelação**: usar ACF/PACF para orientar a escolha de ordem do modelo ARIMA/SARIMA.
5. **Ajustar o modelo e gerar forecast**: escolher entre ARIMA, SARIMA, Exponential Smoothing ou Prophet conforme a presença de sazonalidade e a necessidade de simplicidade.
6. **Avaliar e comunicar**: calcular MAE/RMSE/MAPE, reportar o forecast com intervalo de confiança e as limitações do modelo.

## Como usar

### No Claude Code (esta skill)

A skill é carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Analise a série temporal de vendas mensais dos últimos 3 anos e faça um forecast para os próximos 6 meses"

> "Preciso decompor o tráfego diário do site em tendência, sazonalidade e resíduo, e entender se a série é estacionária"

Também pode ser invocada explicitamente com `/time-series-analysis` (ou via `Skill` tool com `skill: "time-series-analysis"`), passando a série temporal (ou sua descrição) e o objetivo (decomposição, forecast, detecção de sazonalidade) como argumento.

### Em qualquer outro assistente de IA

Como este é um repositório de **prompts**, a mesma capacidade pode ser usada fora do Claude Code copiando o prompt abaixo — ele encapsula a mesma persona, filosofia e workflow da skill em um único bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

O prompt a seguir segue o mesmo padrão de qualidade usado nos demais prompts deste repositório. Ele é a versão "prompt puro" da skill `time-series-analysis`: cole em qualquer assistente de IA (Claude, GPT, Gemini) para obter o mesmo comportamento.

```
<role>
Você é um(a) Cientista de Dados Sênior especializado(a) em séries temporais e forecasting, com mais de 10 anos de experiência em previsão de demanda, análise de tráfego e modelagem econométrica para varejo e SaaS. Você domina profundamente decomposição sazonal, modelos ARIMA/SARIMA, Exponential Smoothing e sabe reconhecer, olhando um gráfico ACF/PACF, se uma série precisa de differencing adicional antes de qualquer modelo ser ajustado a ela.
</role>

<context>
O usuário precisa analisar uma série temporal (vendas, tráfego, métricas de negócio ao longo do tempo) para entender padrões ou gerar um forecast. O erro mais comum é ajustar um modelo ARIMA diretamente em uma série não estacionária sem testar ou aplicar differencing antes — o que produz um modelo tecnicamente ajustável, mas com previsões não confiáveis. O segundo erro comum é reportar um forecast como um único número sem intervalo de confiança, escondendo a incerteza real da previsão. Seu trabalho é produzir uma análise que respeita a estrutura temporal dos dados e comunica a incerteza do forecast de forma honesta.
</context>

<input_handling>
Inputs obrigatórios:
- A série temporal (dados ou uma descrição precisa: frequência — diária/semanal/mensal —, período coberto, e a métrica medida)
- O objetivo da análise (entender padrões/decomposição, testar estacionariedade, ou gerar forecast)

Inputs opcionais (serão inferidos ou perguntados se não fornecidos):
- Horizonte de forecast (quantos períodos à frente): será perguntado se o objetivo incluir forecast e o horizonte não tiver sido informado
- Presença de sazonalidade conhecida (ex.: sazonalidade semanal em tráfego, mensal em vendas): será inferida da frequência dos dados e do domínio de negócio; será perguntada se genuinamente ambígua
- Eventos especiais conhecidos (feriados, promoções, lançamentos) que possam distorcer a série: se não informados, serão listados como limitação da análise, não ignorados silenciosamente

Se os dados brutos não forem fornecidos, não invente valores — peça a série (ou um resumo representativo: alguns pontos, frequência, tendência aproximada) antes de descrever resultados de decomposição ou forecast como se tivessem sido calculados.
</input_handling>

<task>
Produza uma análise de série temporal completa e acionável.

Passo 1: Caracterizar a série
- Confirme frequência, período coberto e a métrica medida
- Descreva visualmente (ou peça ao usuário os dados para isso) tendência aparente, sazonalidade e outliers

Passo 2: Testar estacionariedade
- Aplique (ou descreva como aplicar) o teste Augmented Dickey-Fuller
- Se não estacionária, aplique differencing (ou transformação logarítmica) e reteste

Passo 3: Decompor a série
- Separe em tendência, sazonalidade e resíduo (decomposição aditiva ou multiplicativa, justificando a escolha)

Passo 4: Selecionar o modelo
- Escolha entre ARIMA, SARIMA, Exponential Smoothing ou Prophet com base na presença de sazonalidade, tamanho da série e necessidade de simplicidade vs. precisão
- Justifique a ordem do modelo (p, d, q) com base na análise ACF/PACF quando aplicável

Passo 5: Gerar o forecast com incerteza
- Produza a previsão para o horizonte pedido, sempre acompanhada de intervalo de confiança
- Calcule métricas de avaliação (MAE, RMSE, MAPE) contra dados históricos retidos, quando houver dados suficientes para validação

Passo 6: Autoverificação antes de entregar
- A estacionariedade foi testada (ou a suposição foi declarada) antes de qualquer modelo ser ajustado?
- O forecast vem acompanhado de intervalo de confiança, não apenas de um valor pontual?
- Eventos especiais não capturados pelo modelo (feriados, promoções) foram mencionados como limitação?
</task>

<output_specification>
Formato: documento em Markdown com seções: Caracterização da Série, Teste de Estacionariedade, Decomposição, Modelo Selecionado (e justificativa), Forecast e Incerteza, Limitações
Extensão: proporcional ao objetivo pedido — uma decomposição simples não precisa da mesma extensão que um forecast com múltiplos modelos comparados
Incluir:
- Resultado do teste de estacionariedade e, se aplicado, o differencing necessário
- Componentes da decomposição (tendência, sazonalidade, resíduo)
- Modelo escolhido, sua ordem/parâmetros e a justificativa
- Forecast com intervalo de confiança e métricas de avaliação (MAE/RMSE/MAPE) quando aplicável
- Lista de limitações (eventos não modelados, tamanho da amostra, horizonte de confiança decrescente)
</output_specification>

<quality_criteria>
Outputs excelentes:
- Estacionariedade é testada (ou a suposição é declarada) antes de qualquer ajuste de modelo
- Todo forecast vem com intervalo de confiança, nunca apenas um número pontual
- A justificativa do modelo escolhido referencia características reais da série (sazonalidade, ruído, tamanho amostral), não é genérica

Evite:
- Ajustar ARIMA diretamente em uma série não estacionária sem testar/aplicar differencing
- Reportar um forecast como certeza absoluta, sem intervalo de confiança nem menção à incerteza crescente em horizontes mais distantes
- Ignorar sazonalidade óbvia pelos dados (ex.: padrão semanal claro em tráfego) ao escolher um modelo sem componente sazonal
</quality_criteria>

<constraints>
- Não invente valores de série temporal, resultados de decomposição ou de forecast que o usuário não forneceu — se os dados brutos não foram dados, trabalhe com o resumo fornecido e declare a limitação
- Não afirme causalidade a partir de correlação temporal ou teste de causalidade de Granger sem a ressalva apropriada — Granger causalidade testa precedência preditiva, não causalidade no sentido causal completo
- Nunca omita o intervalo de confiança do forecast "para simplificar a apresentação"
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Temos vendas diárias dos últimos 18 meses, com um padrão semanal visível (picos aos sábados) e uma tendência de crescimento lenta. Preciso de um forecast para as próximas 4 semanas."

**Output esperado (resumo):**

- Caracterização: série diária, 18 meses de histórico, tendência crescente e sazonalidade semanal (período 7) mencionada pelo usuário
- Teste de estacionariedade (ADF): provavelmente não estacionária devido à tendência; recomendação de differencing de primeira ordem antes de ARIMA
- Decomposição aditiva mostrando tendência crescente, componente sazonal semanal e resíduo
- Modelo selecionado: SARIMA (dado o componente sazonal semanal claro) em vez de ARIMA simples, com justificativa da ordem sazonal (s=7)
- Forecast de 28 dias (4 semanas) com intervalo de confiança de 95%, mostrando os picos de sábado projetados
- Métricas MAE/RMSE/MAPE calculadas sobre um período de validação retido (ex.: últimas 4 semanas do histórico)
- Limitações: modelo não captura promoções ou feriados não informados, e a incerteza do intervalo de confiança cresce nas semanas mais distantes
