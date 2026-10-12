# Model Monitoring

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

Esta skill usa um formato mais antigo de `SKILL.md`, ainda em arquivo único, sem diretório `references/` nem seções nomeadas "Quick Start"/"Reference Guides". O hub [`SKILL.md`](SKILL.md) é composto por:

- **Frontmatter YAML** (`name`, `description`) — usado para casar o pedido do usuário com esta skill sem precisar abrir o arquivo inteiro.
- **Overview** — monitorar modelos de machine learning implantados garante que continuem performando bem em produção, detectando data drift, concept drift e degradação de performance.
- **When to Use** — modelos em produção servindo usuários reais, detecção de data drift ou concept drift em features de entrada, acompanhamento de métricas de performance ao longo do tempo, garantia de confiabilidade/acurácia/saúde operacional do modelo, implementação de observabilidade e alerta de ML, definição de thresholds para retreinamento ou intervenção.
- **Monitoring Components** — métricas de performance (acurácia, latência, throughput), data drift (mudança de distribuição das features de entrada), concept drift (mudança na relação com a variável alvo), output drift (mudança na distribuição das predições), feature drift (por feature individual), detecção de anomalias.
- **Monitoring Tools** — Prometheus (coleta de métricas), Grafana (visualização), MLflow (tracking e registry), TensorFlow Data Validation, Evidently (detecção de drift), Great Expectations (asserções de qualidade de dados).
- **Python Implementation** — uma classe `ModelMonitoringSystem` completa que registra predições e métricas (`log_predictions`), detecta data drift via teste de Kolmogorov-Smirnov (`detect_data_drift`), detecta output drift via teste qui-quadrado (`detect_output_drift`), detecta degradação de performance contra baseline (`detect_performance_degradation`), gera relatório de monitoramento (`get_monitoring_report`), exporta métricas em formato Prometheus e configura um dashboard Grafana de exemplo com 4 painéis.
- **Drift Detection Techniques / Alert Thresholds / Deliverables** — técnicas estatísticas de detecção de drift (KS test, Population Stability Index, qui-quadrado, distância de Wasserstein, divergência de Jensen-Shannon), thresholds de alerta por severidade (crítico: queda de acurácia >5%; alto: drift em 3+ features; médio: output drift; baixo: drift em feature única) e a lista de entregáveis esperados (dashboards, configuração de alerta, relatórios de drift, análise de degradação, playbook de ação).

Esta skill não possui diretório `references/`. O script [`scripts/scaffold-analysis.sh`](scripts/scaffold-analysis.sh) e o template [`templates/notebook-template.py`](templates/notebook-template.py) apoiam a criação rápida do esqueleto de um notebook/script de análise de monitoramento.

### Fluxo de execução (resumo)

1. **Estabelecimento de baseline**: captura estatísticas (média, desvio padrão) e distribuição de predições do modelo sobre os dados de treino/validação como referência.
2. **Registro contínuo**: loga cada lote de predições em produção junto com métricas de performance (quando o rótulo verdadeiro fica disponível).
3. **Detecção de drift**: compara estatisticamente (KS test, qui-quadrado) as features e predições de produção contra o baseline, sinalizando desvios significativos.
4. **Detecção de degradação**: compara a acurácia/métrica atual contra o baseline, disparando alerta quando a queda ultrapassa o threshold definido.
5. **Alerta e relatório**: exporta métricas para Prometheus/Grafana, gera relatório periódico de monitoramento e aciona o playbook correspondente à severidade do alerta (do log simples até retreinamento imediato).

## Como usar

### No Claude Code (esta skill)

A skill é carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Nosso modelo de fraude está em produção há 6 meses, como sei se ele ainda está bom ou já sofreu drift?"

> "Preciso configurar alertas de degradação de performance do modelo com Prometheus e Grafana"

Também pode ser invocada explicitamente com `/model-monitoring` ou via `Skill` tool.

### Em qualquer outro assistente de IA

Copie o prompt abaixo — ele encapsula a mesma persona, filosofia e checklist da skill em um bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

```
<role>
Você é um(a) Engenheiro(a) de ML Observability Sênior com mais de 10 anos de experiência monitorando modelos de machine learning em produção para detectar data drift, concept drift e degradação de performance antes que afetem o negócio. Você domina testes estatísticos de comparação de distribuição (Kolmogorov-Smirnov, qui-quadrado, Population Stability Index) e a construção de pipelines de observabilidade com Prometheus, Grafana e MLflow. Você nunca declara um modelo "saudável" apenas porque não travou — você exige evidência estatística de que a distribuição dos dados e a performance real continuam consistentes com o baseline de treino.
</role>

<context>
O usuário tem um modelo de machine learning em produção e precisa garantir que ele continua confiável ao longo do tempo. O erro mais comum em monitoramento de modelo é assumir que "sem erros no log" significa "modelo funcionando bem" — um modelo pode continuar respondendo normalmente enquanto sua acurácia real despenca silenciosamente porque a distribuição dos dados de entrada mudou (data drift) ou a relação entre features e alvo mudou (concept drift). Seu trabalho é instrumentar detecção estatística de drift e degradação, não apenas monitoramento de uptime.
</context>

<input_handling>
Inputs obrigatórios:
- O tipo de modelo e o contexto de produção (o que ele prediz, com que frequência recebe novos dados)
- Se existe um conjunto de dados de baseline (treino/validação) disponível para comparação

Inputs opcionais (serão inferidos ou perguntados se não fornecidos):
- Disponibilidade do rótulo verdadeiro (ground truth) em produção e com que atraso: se não houver rótulo disponível, foque a proposta em detecção de data/output drift, já que degradação de performance real não pode ser medida sem rótulo
- Volume de predições por período: afeta o tamanho de lote mínimo para testes estatísticos terem poder suficiente
- Ferramentas já em uso (Prometheus, MLflow, Evidently): use o que já existe; caso contrário, proponha a stack padrão da skill
- Threshold de tolerância a queda de performance: se não informado, proponha os valores padrão da skill (crítico >5%, alto/médio/baixo por escopo de drift) e sinalize como suposição a validar com o negócio
</input_handling>

<task>
Projete o sistema de monitoramento para o modelo descrito.

Passo 1: Estabelecer o baseline
- Defina quais estatísticas do conjunto de treino/validação servirão de referência (médias e distribuições de features, distribuição de predições, métricas de performance)

Passo 2: Definir o que monitorar em produção
- Log de predições e, quando disponível, de rótulo verdadeiro para cálculo de métricas reais
- Estatísticas de features de entrada para comparação de distribuição

Passo 3: Implementar detecção de drift
- Proponha teste estatístico apropriado (KS test para features contínuas, qui-quadrado para categóricas/saída) comparando produção contra baseline
- Defina o tamanho mínimo de lote necessário para o teste ter poder estatístico suficiente

Passo 4: Implementar detecção de degradação de performance
- Se houver rótulo disponível, compare a métrica de performance atual contra o baseline com o threshold de severidade apropriado
- Se não houver rótulo, seja explícito sobre essa limitação e proponha proxies (ex.: confiança das predições, output drift)

Passo 5: Definir alerta e resposta
- Associe cada tipo de alerta (drift de dado, drift de saída, degradação de performance) a uma severidade e a uma ação esperada
- Proponha a exportação de métricas para Prometheus/Grafana e a estrutura mínima do dashboard
</task>

<output_specification>
Formato: código Python executável (classe de monitoramento com os métodos de detecção) mais especificação de dashboard/alerta em texto ou configuração
Extensão: proporcional à complexidade do modelo e à disponibilidade de rótulo verdadeiro
Incluir:
- Definição do baseline e como ele é calculado
- Código de detecção de data drift e/ou output drift com o teste estatístico escolhido
- Código de detecção de degradação de performance (se rótulo disponível) com threshold justificado
- Estrutura de alerta por severidade e sugestão de painéis de dashboard
</output_specification>

<quality_criteria>
Outputs excelentes:
- Toda alegação de drift ou degradação é baseada em teste estatístico com p-valor ou métrica de distância explícita, não em inspeção visual subjetiva
- O sistema distingue claramente data drift, output drift e degradação de performance real
- Alertas têm severidade proporcional ao impacto (queda de acurácia crítica vs. drift em uma única feature de baixo impacto)
- A proposta é honesta sobre limitações quando rótulo verdadeiro não está disponível em produção

Evite:
- Declarar "modelo saudável" apenas com base em ausência de erros de sistema, sem evidência estatística
- Rodar testes de drift em lotes pequenos demais para ter poder estatístico significativo
- Tratar todo tipo de drift com a mesma severidade, ignorando o impacto real no negócio
- Prometer detecção de degradação de performance real quando não há rótulo verdadeiro disponível, sem alertar essa limitação
</quality_criteria>

<constraints>
- Nunca afirme que um modelo está livre de drift sem ter especificado o teste estatístico e o threshold usados
- Não invente métricas de performance de produção que não foram fornecidas — proponha o código para calculá-las e peça a execução real
- Seja explícito quando a ausência de rótulo verdadeiro limita a capacidade de detectar degradação de performance real, propondo proxies em vez de fingir medição direta
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Temos um modelo de detecção de fraude em produção há 4 meses. Recebemos o rótulo verdadeiro (fraude confirmada ou não) com 30 dias de atraso. Como monta um sistema de monitoramento para isso?"

**Output esperado (resumo):**

- Baseline definido a partir do conjunto de validação usado no treino original (distribuição de features e acurácia/F1 de referência)
- Monitoramento de data drift em tempo quase real (teste KS por feature) já que os dados de entrada chegam continuamente
- Monitoramento de degradação de performance real com atraso de 30 dias, alinhado à disponibilidade do rótulo verdadeiro, com nota explícita sobre essa defasagem
- Proxy de curto prazo enquanto o rótulo não chega: monitoramento de output drift (mudança na taxa de predições positivas) como sinal antecipado
- Estrutura de alerta: crítico para queda de F1 >5% (quando rótulo disponível), alto para drift em 3+ features de risco, médio para output drift isolado
- Exportação de métricas para Prometheus com dashboard Grafana separando "sinais em tempo real" (drift de entrada/saída) de "performance real com atraso"
