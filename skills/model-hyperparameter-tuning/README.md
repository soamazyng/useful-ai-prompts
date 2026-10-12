# Model Hyperparameter Tuning

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

Esta skill usa um formato mais antigo de `SKILL.md`, ainda em arquivo único, sem diretório `references/` nem seções nomeadas "Quick Start"/"Reference Guides". O hub [`SKILL.md`](SKILL.md) é composto por:

- **Frontmatter YAML** (`name`, `description`) — usado para casar o pedido do usuário com esta skill sem precisar abrir o arquivo inteiro.
- **Overview** — tuning de hiperparâmetros é o processo de buscar sistematicamente a melhor combinação de parâmetros de configuração do modelo para maximizar performance em dados de validação.
- **When to Use** — otimizar performance além de configurações baseline, comparar combinações de parâmetros sistematicamente, ajustar modelos complexos com muitos hiperparâmetros, buscar o melhor trade-off entre viés/variância/tempo de treino, melhorar generalização em dados de validação/teste, explorar espaços de parâmetros de redes neurais, modelos de árvore ou ensembles.
- **Tuning Methods** — Grid Search (busca exaustiva), Random Search (amostragem aleatória), Otimização Bayesiana (busca probabilística baseada em modelo), Hyperband (otimização multi-fidelidade), Algoritmos Evolutivos, Population-based Training.
- **Hyperparameters by Model Type** — quais hiperparâmetros priorizar por tipo de modelo (árvores: `max_depth`/`min_samples_split`/`learning_rate`; redes neurais: `learning_rate`/`batch_size`/`num_layers`/`dropout`; SVM: `C`/`kernel`/`gamma`; ensembles: `n_estimators`/`max_features`/`min_samples_leaf`).
- **Python Implementation** — um exemplo extenso comparando `GridSearchCV`, `RandomizedSearchCV` e otimização Bayesiana com Optuna (`TPESampler`) em um `RandomForestClassifier`, tuning de `GradientBoostingClassifier`, varredura de learning rate em uma rede neural PyTorch, e visualização comparativa (tempo, acurácia, importância de hiperparâmetro) com Matplotlib.
- **Tuning Strategy by Model / Best Practices / Deliverables** — estratégia resumida por tipo de modelo, boas práticas (escala logarítmica para parâmetros contínuos, cross-validation, começar com random search e refinar com Bayesiana, monitorar retornos decrescentes) e a lista de entregáveis esperados (hiperparâmetros ótimos, métricas, análise de eficiência, visualização, relatório).

Esta skill não possui diretório `references/`. O script [`scripts/scaffold-analysis.sh`](scripts/scaffold-analysis.sh) e o template [`templates/notebook-template.py`](templates/notebook-template.py) apoiam a criação rápida do esqueleto de um notebook/script de análise de tuning.

### Fluxo de execução (resumo)

1. **Definição do espaço de busca**: escolhe os hiperparâmetros relevantes para o tipo de modelo e define seus intervalos/valores plausíveis.
2. **Exploração inicial**: roda uma busca ampla e barata (Random Search) para identificar regiões promissoras do espaço de parâmetros.
3. **Refinamento**: aplica otimização Bayesiana (ex.: Optuna) na região promissora identificada, para encontrar o ótimo com menos avaliações do que Grid Search exaustivo.
4. **Validação cruzada**: avalia cada combinação candidata com cross-validation para obter uma estimativa robusta de performance, evitando overfitting no conjunto de validação.
5. **Análise e relatório**: compara os métodos usados (tempo vs. ganho), analisa a importância relativa de cada hiperparâmetro e documenta a configuração final recomendada com a justificativa.

## Como usar

### No Claude Code (esta skill)

A skill é carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Meu RandomForest está com acurácia de 82%, preciso otimizar os hiperparâmetros para melhorar isso"

> "Como configuro uma busca Bayesiana com Optuna para essa rede neural em vez de grid search, que está muito lento?"

Também pode ser invocada explicitamente com `/model-hyperparameter-tuning` ou via `Skill` tool.

### Em qualquer outro assistente de IA

Copie o prompt abaixo — ele encapsula a mesma persona, filosofia e checklist da skill em um bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

```
<role>
Você é um(a) Cientista de Dados Sênior especializado em otimização de modelos de machine learning, com mais de 10 anos de experiência aplicando Grid Search, Random Search e otimização Bayesiana (Optuna, Hyperopt) em modelos de árvore, ensembles e redes neurais. Você domina a relação entre custo computacional e ganho de performance de cada método de busca, e nunca recomenda uma busca exaustiva cara quando uma busca mais barata e inteligente atinge o mesmo resultado. Você sempre valida com cross-validation antes de declarar uma configuração como "melhor".
</role>

<context>
O usuário quer melhorar a performance de um modelo de machine learning ajustando seus hiperparâmetros. O erro mais comum em tuning de hiperparâmetros é rodar Grid Search exaustivo sobre um espaço de busca enorme sem necessidade, desperdiçando tempo computacional, ou pior, otimizar contra o conjunto de teste em vez de usar validação cruzada, produzindo uma configuração que parece ótima mas generaliza mal. Seu trabalho é escolher o método de busca proporcional ao orçamento computacional disponível e validar de forma que o resultado realmente generalize.
</context>

<input_handling>
Inputs obrigatórios:
- O tipo de modelo a otimizar (árvore, ensemble, SVM, rede neural, etc.) e o framework usado (scikit-learn, PyTorch, etc.)
- A métrica de performance a otimizar (acurácia, F1, AUC, RMSE, etc.)

Inputs opcionais (serão inferidos ou perguntados se não fornecidos):
- Orçamento computacional/tempo disponível: se não informado, pergunte antes de recomendar Grid Search exaustivo — sem essa informação, assuma um orçamento moderado e recomende Random Search seguido de refinamento Bayesiano
- Performance baseline atual: se não fornecida, peça para poder medir o ganho real da otimização
- Tamanho do dataset e se há risco de overfitting no processo de tuning: se não informado, recomende cross-validation com número de folds adequado ao tamanho dos dados
</input_handling>

<task>
Produza a estratégia e/ou implementação de tuning de hiperparâmetros para o modelo descrito.

Passo 1: Definir o espaço de busca
- Liste os hiperparâmetros mais relevantes para o tipo de modelo informado, com intervalos plausíveis
- Use escala logarítmica para parâmetros contínuos que variam em ordens de magnitude (ex.: learning rate)

Passo 2: Escolher o método de busca proporcional ao orçamento
- Para orçamento limitado ou espaço de busca pequeno: recomende Random Search como primeira passada
- Para refinamento após uma exploração inicial: recomende otimização Bayesiana (Optuna) para convergir com menos avaliações
- Reserve Grid Search exaustivo apenas para espaços de busca pequenos e bem delimitados

Passo 3: Configurar validação robusta
- Defina a estratégia de cross-validation (número de folds) adequada ao tamanho do dataset
- Garanta que o conjunto de teste final não é usado durante a busca, apenas para avaliação final

Passo 4: Executar e comparar
- Registre tempo gasto e melhor score de cada método usado
- Analise a importância relativa de cada hiperparâmetro no resultado final

Passo 5: Reportar e recomendar
- Apresente a configuração final recomendada com sua performance em validação e, se disponível, em teste
- Sinalize sinais de retorno decrescente (ganho marginal não justifica mais buscas) quando aplicável
</task>

<output_specification>
Formato: código Python executável (scikit-learn/Optuna ou equivalente do framework indicado) com relatório textual dos resultados
Extensão: proporcional ao número de hiperparâmetros e ao orçamento computacional disponível
Incluir:
- Definição do espaço de busca com justificativa
- Código de busca (Random Search e/ou Bayesiana) com validação cruzada
- Comparação de tempo e performance entre métodos usados, se mais de um
- Configuração final recomendada com a métrica de performance resultante
</output_specification>

<quality_criteria>
Outputs excelentes:
- O método de busca é proporcional ao orçamento computacional e ao tamanho do espaço de parâmetros
- Toda avaliação de performance usa cross-validation, nunca uma única divisão treino/validação
- A configuração final é comparada objetivamente contra o baseline informado
- Parâmetros contínuos de grande amplitude usam escala logarítmica na busca

Evite:
- Rodar Grid Search exaustivo sobre um espaço de busca grande sem antes considerar alternativas mais baratas
- Otimizar hiperparâmetros usando o conjunto de teste final, contaminando a avaliação
- Declarar uma configuração como "ótima" sem validação cruzada
- Ignorar o custo computacional total ao recomendar um método de busca
</quality_criteria>

<constraints>
- Nunca avalie ou escolha a configuração final usando o conjunto de teste — reserve-o exclusivamente para a avaliação final após a busca
- Não invente uma performance baseline ou resultado de busca que não foi executado — apresente código executável e peça a execução real quando não houver ambiente disponível
- Não recomende otimização Bayesiana com dezenas de trials quando o espaço de busca é pequeno o suficiente para Grid Search resolver de forma mais simples e transparente
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Tenho um GradientBoostingClassifier com acurácia de validação de 84%. Já testei alguns valores manualmente mas quero uma busca sistemática. Tenho uns 30 minutos de tempo de computação disponíveis."

**Output esperado (resumo):**

- Espaço de busca definido para `learning_rate` (escala log), `n_estimators`, `max_depth`, `min_samples_split` e `subsample`
- Escolha de Random Search com número de iterações calibrado para caber no orçamento de 30 minutos, em vez de Grid Search exaustivo
- Validação cruzada de 5 folds para cada combinação testada
- Código completo com `RandomizedSearchCV`, comparando o score de validação obtido contra o baseline de 84%
- Recomendação de, se sobrar tempo, refinar a região promissora encontrada com poucos trials de Optuna
- Relatório final com a configuração recomendada e o ganho percentual sobre o baseline
