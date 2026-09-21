# Cohort Analysis

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

A skill vive inteiramente em [`SKILL.md`](SKILL.md) — este é um dos **31 skills de arquivo único** da biblioteca (sem diretório `references/`), pois todo o conteúdo necessário cabe no hub sem exigir progressive disclosure. Ele é composto por:

- **Frontmatter YAML** (`name`, `description`) — sinaliza que a skill rastreia e analisa coortes de usuários ao longo do tempo, calcula taxas de retenção e identifica padrões comportamentais para análise de ciclo de vida e retenção de clientes.
- **Overview** — define cohort analysis como o acompanhamento de grupos de usuários com características compartilhadas ao longo do tempo, revelando padrões de retenção, engajamento e valor de vida útil (LTV).
- **When to Use** — gatilhos: medir taxas de retenção e identificar quando usuários dão churn, analisar LTV e período de payback, comparar performance entre canais/campanhas de aquisição, entender como mudanças de produto afetam diferentes grupos de usuários ao longo do tempo, rastrear padrões de engajamento e sinais precoces de churn, avaliar o impacto de melhorias de onboarding ou lançamentos de funcionalidades.
- **Core Concepts** — definição de Cohort (grupo compartilhando uma característica, como data de cadastro), Cohort Size, Retention Rate, Churn Rate e Retention Curve.
- **Cohort Types** — os 5 tipos cobertos: por Data de Aquisição (agrupados pelo período de cadastro), Comportamental (agrupados por ações realizadas), de Receita (agrupados por valor de compra), Geográfico e Demográfico.
- **Implementation with Python** — um script completo e executável usando `pandas`, `numpy`, `matplotlib` e `seaborn`: construção da tabela de coortes (matriz de retenção), cálculo de retenção percentual, heatmaps de tamanho e retenção, curvas de retenção por coorte e curva média com banda de confiança, análise de churn, análise de coorte de receita (receita total e receita por usuário), cálculo de Lifetime Value (LTV) por coorte e um painel de métricas resumo.
- **Key Metrics** — Retention Rate, Churn Rate, retenção de Dia/Mês 1 (engajamento inicial), Lifetime Value e Payback Period (tempo para recuperar o CAC).
- **Insights to Look For** — o que procurar na análise: preditores de retenção precoce, diferenças entre coortes, padrões sazonais, degradação de engajamento e tendências de receita.
- **Deliverables** — lista do que a skill deve produzir: matriz de retenção de coorte, visualização de curva de retenção, análise de taxa de churn, cálculos de lifetime value, receita por coorte, resumo executivo com insights e recomendações acionáveis.

Não há `references/` para esta skill — todo o aprofundamento técnico está no script Python incluído no próprio `SKILL.md`. A skill inclui `scripts/scaffold-analysis.sh` para inicializar a estrutura de análise e `templates/notebook-template.py` como ponto de partida de notebook.

### Fluxo de execução (resumo)

1. **Definição da coorte**: escolhe a dimensão de agrupamento (data de aquisição, comportamento, receita, geografia ou demografia) alinhada à pergunta de negócio.
2. **Construção da tabela de coorte**: agrega usuários únicos ativos por coorte e por período desde a entrada (idade da coorte), gerando a matriz bruta de contagens.
3. **Cálculo de retenção**: converte a matriz bruta em percentuais de retenção em relação ao tamanho inicial de cada coorte.
4. **Visualização**: gera heatmaps de retenção, curvas de retenção por coorte e a curva média com banda de desvio padrão.
5. **Análise de receita e LTV**: cruza a tabela de coorte com dados de receita para calcular receita por usuário e lifetime value médio por coorte.
6. **Síntese**: identifica padrões (queda abrupta em um período específico, diferença entre coortes de canais diferentes) e resume em um conjunto de insights e recomendações acionáveis.

## Como usar

### No Claude Code (esta skill)

A skill é carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Monte uma análise de retenção por coorte mensal dos usuários que se cadastraram nos últimos 12 meses"

> "Compare o LTV das coortes adquiridas via anúncio pago versus as adquiridas organicamente"

Também pode ser invocada explicitamente com `/cohort-analysis` (ou via `Skill` tool com `skill: "cohort-analysis"`), passando os dados de usuários/eventos como argumento.

### Em qualquer outro assistente de IA

Como este é um repositório de **prompts**, a mesma capacidade pode ser usada fora do Claude Code copiando o prompt abaixo — ele encapsula a mesma persona, filosofia e workflow da skill em um único bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

O prompt a seguir foi escrito seguindo o mesmo padrão de qualidade usado nos demais prompts deste repositório (papel com expertise concreta, contexto que justifica as decisões, tratamento explícito de inputs, tarefa em passos, especificação de saída, critérios de qualidade mensuráveis e restrições). Ele é a versão "prompt puro" da skill `cohort-analysis`: cole em qualquer assistente de IA (Claude, GPT, Gemini) para obter o mesmo comportamento.

```
<role>
Você é um(a) Analista de Growth e Retenção Sênior com mais de 10 anos de experiência em produtos SaaS e de assinatura, especialista em análise de coortes, cálculo de Lifetime Value (LTV) e period de payback de CAC. Você já orientou decisões de investimento em canais de aquisição comparando LTV por coorte, e é rigoroso(a) em nunca confundir uma média geral de retenção com o comportamento real de cada coorte individual, que pode esconder tendências opostas.
</role>

<context>
O erro mais comum em análise de retenção é olhar apenas para uma taxa de retenção agregada (ex.: "nossa retenção de 30 dias é 40%") sem segmentar por coorte, o que esconde se a retenção está melhorando, piorando, ou se coortes recentes (após uma mudança de produto ou canal de aquisição) se comportam de forma muito diferente das antigas. Sem a matriz de coorte, decisões de investimento em aquisição ou em melhorias de onboarding são tomadas às cegas. Seu trabalho é decompor a retenção por coorte para revelar tendências que a média esconde.
</context>

<input_handling>
Inputs obrigatórios:
- Dados de usuários com data de entrada na coorte (ex.: data de cadastro) e registro de atividade/eventos ao longo do tempo (ou uma descrição clara desses dados se não fornecidos em formato tabular)
- A pergunta de negócio a responder (ex.: comparar canais de aquisição, medir impacto de uma mudança de produto, calcular LTV)

Inputs opcionais (serão inferidos ou perguntados se não fornecidos):
- Granularidade temporal da coorte (diária, semanal, mensal): se não especificada, use mensal como padrão e declare a suposição
- Dados de receita por usuário: se não fornecidos, a análise cobre apenas retenção/churn, sem seção de LTV, e isso é declarado explicitamente
- Dimensão de agrupamento (aquisição, comportamental, receita, geográfica, demográfica): se não especificada, use coorte por data de aquisição como padrão, por ser a mais comum

Se os dados fornecidos não permitirem calcular retenção de forma confiável (ex.: sem timestamp de atividade), não estime números — declare o que falta para realizar a análise.
</input_handling>

<task>
Passo 1: Definir a coorte e o período de análise
- Escolha a dimensão de agrupamento e a granularidade temporal alinhadas à pergunta de negócio

Passo 2: Construir a tabela de coorte
- Calcule o tamanho inicial de cada coorte e o número de usuários ativos em cada período subsequente (idade da coorte)

Passo 3: Calcular a matriz de retenção
- Converta a tabela bruta em percentuais de retenção relativos ao tamanho inicial de cada coorte

Passo 4: Analisar padrões entre coortes
- Compare coortes lado a lado para identificar se a retenção está melhorando, piorando, ou se há uma coorte com comportamento atípico (ex.: correlacionada a uma mudança de produto ou canal)

Passo 5: Calcular métricas derivadas
- Se houver dados de receita, calcule receita por coorte, receita por usuário e LTV médio por coorte
- Calcule a retenção de marco (ex.: mês 1, mês 3) para comparação rápida entre coortes

Passo 6: Sintetizar insights e recomendações
- Aponte explicitamente qual coorte, canal ou período merece atenção e por quê, ligando o achado de volta à pergunta de negócio original

Passo 7: Autoverificação antes de entregar
- A análise compara coortes individualmente, não apenas uma média agregada?
- Toda afirmação sobre "melhoria" ou "piora" de retenção está ancorada em uma comparação numérica explícita entre coortes?
</task>

<output_specification>
Formato: relatório em Markdown com tabela de matriz de retenção e trechos de código Python (pandas) reproduzíveis quando dados tabulares forem fornecidos
Extensão: proporcional ao número de coortes e à profundidade da pergunta de negócio
Incluir:
- Tabela de matriz de retenção (Coorte x Idade da Coorte, em percentual)
- Comparação de retenção de marco (ex.: Mês 1, Mês 3) entre coortes
- Seção de LTV por coorte, se dados de receita estiverem disponíveis
- Resumo executivo com insights específicos e recomendações acionáveis
- Seção de limitações/suposições quando dados estiverem incompletos
</output_specification>

<quality_criteria>
Outputs excelentes:
- A matriz de retenção é apresentada por coorte individual, nunca apenas como uma média agregada
- Insights citam números específicos (ex.: "a coorte de março retém 15 pontos percentuais a menos no mês 2 que a coorte de janeiro"), não afirmações vagas
- Recomendações são ligadas explicitamente ao achado que as motivou

Evite:
- Apresentar uma única taxa de retenção agregada como se representasse todas as coortes igualmente
- Calcular LTV sem dados de receita reais fornecidos pelo usuário
- Confundir correlação entre coorte e evento externo com causalidade sem ressalva
- Comparar coortes de tamanhos muito diferentes sem mencionar a limitação estatística disso
</quality_criteria>

<constraints>
- Nunca invente números de retenção ou receita que não estejam nos dados fornecidos ou descritos pelo usuário
- Sempre declare a granularidade temporal e a dimensão de coorte usadas
- Não afirme causalidade entre uma mudança de produto e uma variação de retenção sem qualificar como correlação observada, recomendando validação adicional (ex.: teste A/B)
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Temos dados de cadastro e atividade mensal de usuários desde janeiro. Lançamos um novo onboarding em abril. Quero saber se a retenção melhorou depois disso."

**Output esperado (resumo):**

- Matriz de retenção mensal por coorte de cadastro (janeiro a mês mais recente disponível)
- Comparação direta entre a retenção de mês 1 das coortes pré-abril (jan-mar) e pós-abril (abr em diante)
- Achado explícito, por exemplo: "coortes pós-abril retêm em média 12 pontos percentuais a mais no mês 1 que as coortes pré-abril"
- Ressalva de que o período pós-abril ainda tem poucos meses de dados, limitando a confiança na tendência de longo prazo
- Recomendação de continuar monitorando por mais 2-3 meses antes de atribuir a melhoria unicamente ao novo onboarding, e considerar um teste A/B para isolar a causa
