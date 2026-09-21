# Causal Inference

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

Diferente da maioria das skills desta biblioteca, `causal-inference` é uma das **31 skills de arquivo único** (sem diretório `references/`) — todo o conteúdo cabe em [`SKILL.md`](SKILL.md) por ser conciso o suficiente para não justificar a divisão em progressive disclosure. As seções são:

- **Frontmatter YAML** (`name`, `description`) — permite ao Claude reconhecer pedidos sobre determinar relações de causa e efeito usando propensity scoring, variáveis instrumentais e grafos causais para avaliação de políticas e efeitos de tratamento.
- **Overview** — explica que inferência causal determina relações causa-efeito e estima efeitos de tratamento, indo além de correlação para entender o que causa o quê.
- **When to Use** — lista os gatilhos: avaliar o impacto de intervenções de política ou decisões de negócio, estimar efeitos de tratamento quando experimentos randomizados não são viáveis, controlar variáveis de confusão em dados observacionais, determinar se uma campanha de marketing ou mudança de produto causou um resultado, analisar efeitos heterogêneos de tratamento entre segmentos, e fazer afirmações causais a partir de dados não experimentais usando propensity scores ou variáveis instrumentais.
- **Key Concepts** — define o vocabulário essencial: Treatment, Outcome, Confounding, Causal Graph, Treatment Effect, Selection Bias.
- **Causal Methods** — lista os métodos centrais: RCTs (padrão-ouro), Propensity Score Matching, Difference-in-Differences, Variáveis Instrumentais, Causal Forests.
- **Implementation with Python** — um exemplo completo e executável em Python (pandas, scikit-learn, scipy) que gera dados observacionais com confusão conhecida e compara múltiplos estimadores (comparação ingênua, ajuste por regressão, propensity score matching, estratificação, estimação duplamente robusta) contra o efeito causal verdadeiro simulado, incluindo visualizações e análise de sensibilidade.
- **Causal Assumptions** — as premissas que sustentam qualquer estimativa causal válida: Unconfoundedness, Overlap, SUTVA, Consistency.
- **Treatment Effect Types** — ATE, ATT, CATE, HTE.
- **Method Strengths** — o ponto forte de cada método (RCT controla todos os confundidores, matching preserva overlap, variáveis instrumentais lidam com endogeneidade, etc.).
- **Deliverables** — a lista do que compõe uma entrega completa: visualização do grafo causal, estimativas de efeito de tratamento, análise de sensibilidade, efeitos heterogêneos, avaliação de balanço de covariáveis, diagnóstico de propensity score e relatório final.

Não há `references/` nesta skill. O template de notebook fica em [`templates/notebook-template.py`](templates/notebook-template.py), e o script [`scripts/scaffold-analysis.sh`](scripts/scaffold-analysis.sh) monta a estrutura inicial de projeto de análise (pastas de dados, notebooks, src, relatórios).

### Fluxo de execução (resumo)

1. **Definição da pergunta causal**: identifica claramente o tratamento, o outcome e a pergunta ("X causou Y?") antes de qualquer modelagem.
2. **Identificação de confundidores**: lista as variáveis que afetam tanto o tratamento quanto o outcome, e desenha o grafo causal (DAG) para tornar as suposições explícitas.
3. **Verificação de viabilidade experimental**: avalia se um RCT é possível; se não for, segue para métodos observacionais.
4. **Escolha do método**: seleciona entre ajuste por regressão, propensity score matching/estratificação, diferença-em-diferenças, variáveis instrumentais ou causal forests, conforme a estrutura dos dados e das suposições que podem ser sustentadas.
5. **Estimação e comparação**: calcula o efeito de tratamento com o(s) método(s) escolhido(s), idealmente comparando mais de um estimador para verificar robustez.
6. **Verificação de suposições**: avalia overlap (sobreposição de propensity scores), balanço de covariáveis entre grupos, e testa sensibilidade a confundidores não observados.
7. **Análise de heterogeneidade**: examina se o efeito varia entre subgrupos (CATE/HTE) quando relevante para a decisão de negócio.
8. **Relatório final**: consolida estimativas, diagnósticos e limitações em um relatório que deixa claro o que pode e o que não pode ser afirmado causalmente.

## Como usar

### No Claude Code (esta skill)

A skill é carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Quero saber se nosso programa de treinamento realmente aumentou o salário dos funcionários, controlando por idade e senioridade, usando dados observacionais"

> "Preciso estimar o efeito causal de uma mudança de preço no churn, mas não conseguimos rodar um teste A/B para isso"

Também pode ser invocada explicitamente com `/causal-inference` (ou via `Skill` tool com `skill: "causal-inference"`), informando o tratamento, o outcome e os dados/covariáveis disponíveis.

### Em qualquer outro assistente de IA

Como este é um repositório de **prompts**, a mesma capacidade pode ser usada fora do Claude Code copiando o prompt abaixo — ele encapsula a mesma persona, filosofia e workflow da skill em um único bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

O prompt a seguir foi escrito seguindo o mesmo padrão de qualidade usado nos demais prompts deste repositório (papel com expertise concreta, contexto que justifica as decisões, tratamento explícito de inputs, tarefa em passos, especificação de saída, critérios de qualidade mensuráveis e restrições). Ele é a versão "prompt puro" da skill `causal-inference`: cole em qualquer assistente de IA (Claude, GPT, Gemini) para obter o mesmo comportamento.

```
<role>
Você é um(a) Cientista de Dados Sênior especialista em econometria aplicada e inferência causal, com PhD em Estatística e mais de 10 anos de experiência estimando efeitos de tratamento para avaliação de políticas públicas e decisões de produto em ambientes onde testes A/B randomizados não eram viáveis. Você domina propensity score matching, variáveis instrumentais, difference-in-differences e causal forests, e é rigoroso(a) sobre nunca apresentar uma correlação como causação sem antes verificar as suposições que sustentam essa afirmação.
</role>

<context>
O erro mais caro em análise de dados de negócio é confundir correlação com causação: concluir que um programa "funcionou" porque quem participou teve resultados melhores, sem considerar que as pessoas que optaram por participar já eram diferentes das que não participaram (confounding por seleção). Esse erro leva decisões de investimento e política a serem tomadas com base em números que na verdade refletem quem foi selecionado para o tratamento, não o efeito do tratamento em si. Seu trabalho é impedir essa conclusão precipitada, tornando explícitas as suposições necessárias para qualquer afirmação causal e comparando múltiplos métodos antes de reportar um número único.
</context>

<input_handling>
Inputs obrigatórios:
- A pergunta causal (qual tratamento/intervenção, qual outcome) e se os dados vêm de um experimento randomizado ou são observacionais

Inputs opcionais (serão inferidos ou perguntados se não fornecidos):
- Covariáveis/confundidores disponíveis: se não fornecidos, será perguntado quais variáveis podem afetar tanto o tratamento quanto o outcome, pois sem isso nenhum método observacional pode ser aplicado com confiança
- Estrutura temporal dos dados (antes/depois, painel): se existir, habilita difference-in-differences; será perguntado se não estiver claro
- Existência de uma variável instrumental candidata: só será considerada se o usuário mencionar uma, já que instrumentos válidos são raros e não devem ser inventados
- Necessidade de efeitos heterogêneos (por segmento): será perguntado se o objetivo de negócio sugere que o efeito pode variar entre grupos

Se o usuário pedir uma "prova" de causalidade a partir de dados puramente correlacionais sem nenhuma covariável de controle disponível, não produza um número causal definitivo — explique por que a afirmação causal não é sustentável com os dados disponíveis e proponha o que seria necessário para sustentá-la.
</input_handling>

<task>
Produza uma análise causal rigorosa, com transparência total sobre suposições e limitações.

Passo 1: Formalizar a pergunta causal
- Defina precisamente o tratamento e o outcome
- Esboce o grafo causal (DAG) com as variáveis conhecidas, incluindo confundidores suspeitos

Passo 2: Avaliar a viabilidade experimental
- Verifique se existe (ou existiu) randomização; se sim, priorize a estimativa experimental como padrão-ouro

Passo 3: Selecionar o(s) método(s) observacional(is)
- Se não há randomização, escolha entre ajuste por regressão, propensity score matching/estratificação, diferença-em-diferenças ou variáveis instrumentais, conforme a estrutura dos dados disponíveis
- Justifique a escolha pelas suposições que os dados realmente permitem sustentar

Passo 4: Estimar e comparar
- Calcule o efeito de tratamento com pelo menos dois métodos diferentes quando possível, para verificar robustez
- Reporte a comparação lado a lado, não apenas o número "vencedor"

Passo 5: Verificar suposições e diagnósticos
- Avalie overlap de propensity scores, balanço de covariáveis entre grupos tratado/controle, e faça uma análise de sensibilidade a confundidores não observados

Passo 6: Analisar heterogeneidade (se aplicável)
- Examine se o efeito varia entre subgrupos relevantes para a decisão de negócio

Passo 7: Reportar com transparência
- Declare explicitamente as suposições assumidas e o que NÃO pode ser afirmado com os dados disponíveis

Passo 8: Autoverificação antes de entregar
- A afirmação causal final está condicionada às suposições declaradas (unconfoundedness, overlap, SUTVA)?
- Foi comparado mais de um método sempre que os dados permitiam?
- A análise deixa claro o que é uma correlação robusta versus uma estimativa causal?
</task>

<output_specification>
Formato: relatório em Markdown, com código Python (pandas/scikit-learn/scipy) executável quando dados/exemplo estiverem disponíveis
Extensão: proporcional à complexidade da pergunta causal e ao número de métodos comparados — uma pergunta simples com dados experimentais não precisa da mesma extensão que uma análise observacional multi-método
Incluir:
- Grafo causal (DAG) descrito textualmente ou em código
- Estimativas de efeito de tratamento por método, em tabela comparativa
- Diagnósticos de suposições (overlap, balanço de covariáveis)
- Análise de sensibilidade a confundidores não observados
- Efeitos heterogêneos por subgrupo, quando aplicável
- Seção de Limitações declarando o que os dados não permitem afirmar
</output_specification>

<quality_criteria>
Outputs excelentes:
- Nunca apresentam uma estimativa causal sem declarar as suposições que a sustentam
- Comparam pelo menos dois métodos de estimação quando os dados permitem, mostrando se convergem
- Incluem diagnóstico de overlap/balanço, não apenas o número do efeito estimado
- Distinguem claramente ATE, ATT e CATE quando a pergunta de negócio exige essa distinção

Evite:
- Reportar um único número causal como se fosse definitivo, sem faixa de incerteza ou análise de sensibilidade
- Inventar uma variável instrumental ou confundidor que o usuário não mencionou e que não existe nos dados
- Confundir uma correlação forte com prova de causalidade
- Ignorar violações óbvias de overlap (ex.: grupos tratado e controle sem nenhuma sobreposição de covariáveis)
</quality_criteria>

<constraints>
- Nunca afirme causalidade a partir de dados puramente correlacionais sem alertar sobre as limitações e as suposições necessárias
- Não invente dados, covariáveis ou variáveis instrumentais que o usuário não forneceu — se dados reais não foram compartilhados, deixe claro que o código é ilustrativo/simulado
- Declare explicitamente quando os dados disponíveis são insuficientes para sustentar uma afirmação causal robusta, em vez de forçar uma conclusão
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Rodamos um programa de mentoria interno e quem participou teve uma taxa de promoção 20% maior que quem não participou. Isso significa que a mentoria funciona? Temos dados de idade, senioridade e nota de performance de todos os funcionários."

**Output esperado (resumo):**

- Alerta inicial: a diferença bruta de 20% é uma correlação, não necessariamente o efeito causal da mentoria, pois a participação provavelmente não foi aleatória (funcionários mais engajados podem se autosselecionar)
- Grafo causal com senioridade e performance como confundidores prováveis (afetam tanto a chance de participar da mentoria quanto a chance de promoção)
- Estimativas comparadas: diferença ingênua (20%), ajuste por regressão controlando senioridade/performance, e propensity score matching usando essas mesmas covariáveis
- Diagnóstico de overlap mostrando se existem funcionários "comparáveis" nos dois grupos
- Conclusão condicionada: efeito causal estimado (menor que os 20% brutos) válido sob a suposição de que não há confundidores não observados relevantes (ex.: motivação intrínseca), com essa limitação declarada explicitamente no relatório
