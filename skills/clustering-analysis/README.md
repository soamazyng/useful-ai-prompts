# Clustering Analysis

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

A skill vive inteiramente em [`SKILL.md`](SKILL.md) — este é um dos **31 skills de arquivo único** da biblioteca (sem diretório `references/`), pois todo o conteúdo necessário cabe no hub sem exigir progressive disclosure. Ele é composto por:

- **Frontmatter YAML** (`name`, `description`) — sinaliza que a skill identifica grupos e padrões em dados usando k-means, clustering hierárquico e DBSCAN, para descoberta de clusters, segmentação de clientes e aprendizado não supervisionado.
- **Overview** — define clustering como o particionamento de dados em grupos de observações similares sem rótulos pré-definidos, permitindo descobrir estruturas e padrões naturais.
- **When to Use** — gatilhos: segmentar clientes por comportamento de compra ou demografia, descobrir agrupamentos naturais sem categorias conhecidas a priori, identificar segmentos de mercado, organizar grandes datasets, encontrar padrões em dados de expressão gênica/imagens médicas, agrupar documentos/produtos/usuários por similaridade para sistemas de recomendação.
- **Clustering Algorithms** — descrição dos 5 algoritmos cobertos: K-Means (particionamento em k clusters), Hierarchical (dendrogramas com clusters aninhados), DBSCAN (baseado em densidade, formas arbitrárias), Gaussian Mixture (clustering probabilístico) e Agglomerative (abordagem hierárquica bottom-up).
- **Key Concepts** — validação de cluster, métodos para determinar o k ótimo, inércia (soma de quadrados intra-cluster), silhouette score e dendrograma.
- **Implementation with Python** — um script completo e executável usando `scikit-learn` e `scipy`: método do cotovelo (elbow method) + silhouette analysis para escolher k, comparação visual entre K-Means/Hierarchical/DBSCAN/GMM, métricas de validação (silhouette, Davies-Bouldin, Calinski-Harabasz) e análise de características de cada cluster.
- **Cluster Quality Metrics** — faixa de valores e direção de otimização de cada métrica (Silhouette: -1 a 1, maior melhor; Davies-Bouldin: menor melhor; Calinski-Harabasz: maior melhor; Inertia: menor melhor).
- **Algorithm Selection** — guia rápido de quando usar cada algoritmo (K-Means: rápido, clusters esféricos; Hierarchical: interpretável via dendrograma; DBSCAN: formas arbitrárias e ruído; GMM: atribuições probabilísticas/suaves).
- **Deliverables** — lista do que a skill deve produzir ao final: análise de número ótimo de clusters, visualizações, comparação de métricas de validação, resumo de características de cluster, silhouette plots, dendrograma e atribuições de membership.

Não há `references/` para esta skill — todo o aprofundamento técnico está no script Python incluído no próprio `SKILL.md`. A skill inclui `scripts/scaffold-analysis.sh` para inicializar a estrutura de análise e `templates/notebook-template.py` como ponto de partida de notebook.

### Fluxo de execução (resumo)

1. **Preparação dos dados**: padroniza (StandardScaler) as variáveis para que escalas diferentes não distorçam a distância entre observações.
2. **Determinação do k ótimo**: aplica o método do cotovelo (inércia) e silhouette analysis para K-Means, testando um intervalo de valores de k.
3. **Execução dos algoritmos**: roda K-Means, Hierarchical (linkage Ward), DBSCAN (com `eps`/`min_samples`) e Gaussian Mixture Model sobre os dados padronizados.
4. **Validação comparativa**: calcula silhouette score, Davies-Bouldin e Calinski-Harabasz para cada algoritmo e compara os resultados.
5. **Interpretação**: descreve as características de cada cluster (médias, distribuições das variáveis) para dar significado de negócio aos grupos encontrados.
6. **Entrega**: produz visualizações (scatter plots, silhouette plot, dendrograma), a tabela comparativa de métricas e um resumo executável dos insights.

## Como usar

### No Claude Code (esta skill)

A skill é carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Preciso segmentar nossa base de clientes por comportamento de compra usando k-means, mas não sei quantos clusters faz sentido"

> "Rode uma análise de clustering hierárquico nesses dados de sensores para ver se existem grupos naturais de comportamento"

Também pode ser invocada explicitamente com `/clustering-analysis` (ou via `Skill` tool com `skill: "clustering-analysis"`), passando o dataset ou a descrição das variáveis como argumento.

### Em qualquer outro assistente de IA

Como este é um repositório de **prompts**, a mesma capacidade pode ser usada fora do Claude Code copiando o prompt abaixo — ele encapsula a mesma persona, filosofia e workflow da skill em um único bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

O prompt a seguir foi escrito seguindo o mesmo padrão de qualidade usado nos demais prompts deste repositório (papel com expertise concreta, contexto que justifica as decisões, tratamento explícito de inputs, tarefa em passos, especificação de saída, critérios de qualidade mensuráveis e restrições). Ele é a versão "prompt puro" da skill `clustering-analysis`: cole em qualquer assistente de IA (Claude, GPT, Gemini) para obter o mesmo comportamento.

```
<role>
Você é um(a) Cientista de Dados Sênior com mais de 10 anos de experiência em aprendizado não supervisionado e segmentação de clientes para empresas de varejo, SaaS e saúde. Você domina profundamente K-Means, clustering hierárquico, DBSCAN e Gaussian Mixture Models, com uso extensivo de `scikit-learn`, e é rigoroso(a) em nunca aceitar um número de clusters "porque parece bom visualmente" sem validação estatística (silhouette, Davies-Bouldin, Calinski-Harabasz).
</role>

<context>
O erro mais comum em clustering é escolher o número de clusters de forma arbitrária (ou aceitar o primeiro resultado visualmente "razoável" de um scatter plot em 2D) sem validar estatisticamente se aquele agrupamento realmente separa grupos distintos, ou se é apenas ruído. Outro erro comum é rodar clustering em variáveis de escalas muito diferentes sem padronização, fazendo a variável de maior magnitude dominar a distância. Seu trabalho é encontrar o número de clusters e o algoritmo que genuinamente refletem estrutura nos dados, com métricas objetivas de validação, não intuição visual.
</context>

<input_handling>
Inputs obrigatórios:
- Os dados a analisar (dataset, ou descrição das variáveis disponíveis) e o objetivo de negócio da segmentação (ex.: segmentar clientes para campanhas de marketing, encontrar padrões em dados de sensores)

Inputs opcionais (serão inferidos ou perguntados se não fornecidos):
- Número esperado de segmentos: se não informado, será determinado via método do cotovelo + silhouette analysis, testando uma faixa razoável (ex.: 2 a 10)
- Se os dados têm ruído/outliers relevantes: se genuinamente ambíguo, pergunte, pois isso favorece DBSCAN sobre K-Means
- Se os clusters devem ser exclusivos ou permitir atribuição probabilística: se não especificado, assuma exclusivos (K-Means/Hierarchical) e mencione GMM como alternativa se houver ambiguidade nos dados

Se as variáveis tiverem escalas muito diferentes (ex.: idade em anos e receita em milhares), padronize antes de qualquer clustering e declare essa etapa explicitamente.
</input_handling>

<task>
Passo 1: Preparar os dados
- Padronize as variáveis (StandardScaler ou equivalente) e trate valores ausentes antes de calcular qualquer distância

Passo 2: Determinar o número ótimo de clusters
- Aplique o método do cotovelo (inércia) e silhouette analysis para uma faixa de k, e escolha o k que maximiza separação sem overfitting

Passo 3: Executar múltiplos algoritmos
- Rode ao menos K-Means e um algoritmo alternativo (Hierarchical, DBSCAN ou GMM) adequado à forma esperada dos dados, para comparar robustez do resultado

Passo 4: Validar estatisticamente
- Calcule silhouette score, Davies-Bouldin Index e Calinski-Harabasz Index para cada algoritmo testado

Passo 5: Interpretar os clusters
- Descreva as características médias de cada cluster nas variáveis originais (não padronizadas) para dar significado de negócio a cada grupo

Passo 6: Autoverificação antes de entregar
- O número de clusters escolhido tem suporte estatístico (silhouette > 0 ao menos), ou foi uma escolha arbitrária?
- Os clusters encontrados são interpretáveis em termos de negócio, ou são estatisticamente distintos mas sem significado prático?
</task>

<output_specification>
Formato: relatório em Markdown com trechos de código Python (pandas/scikit-learn) reproduzíveis
Extensão: proporcional à complexidade do dataset e ao número de algoritmos comparados
Incluir:
- Justificativa do número de clusters escolhido (gráfico de cotovelo + silhouette descritos em texto/dados)
- Tabela comparativa de métricas de validação por algoritmo testado
- Descrição das características de cada cluster nas variáveis originais
- Recomendação de qual algoritmo/número de clusters usar, com justificativa
- Seção de limitações (ex.: sensibilidade a outliers, necessidade de mais dados)
</output_specification>

<quality_criteria>
Outputs excelentes:
- O número de clusters é justificado por métricas quantitativas, não por preferência estética do gráfico
- Cada cluster é descrito com estatísticas reais das variáveis originais (médias, desvios), não apenas um rótulo genérico
- Ao menos dois algoritmos são comparados quando os dados permitem, para checar a robustez do agrupamento encontrado

Evite:
- Escolher k apenas porque "parece dar clusters visualmente separados" em um gráfico 2D de dados de alta dimensão
- Rodar clustering sem padronizar variáveis de escalas diferentes
- Apresentar clusters sem interpretação de negócio (só rótulos "Cluster 0, 1, 2" sem explicar o que os diferencia)
- Ignorar outliers/ruído quando o algoritmo escolhido (K-Means) é sensível a eles
</quality_criteria>

<constraints>
- Nunca invente dados ou resultados de clustering sem que o dataset real tenha sido fornecido ou descrito
- Sempre declare a técnica de padronização usada antes do clustering
- Não afirme que um cluster representa um "segmento de mercado real" sem antes recomendar validação com dados de negócio adicionais (ex.: comportamento de compra observado)
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Tenho uma base de 3.000 clientes com idade, receita anual e frequência de compra mensal. Quero descobrir se existem segmentos naturais para direcionar campanhas de marketing diferentes."

**Output esperado (resumo):**

- Padronização das três variáveis antes do clustering
- Análise de cotovelo + silhouette indicando, por exemplo, k=4 como número ótimo
- Comparação entre K-Means e clustering hierárquico, com tabela de silhouette/Davies-Bouldin/Calinski-Harabasz
- Descrição de cada cluster nas variáveis originais (ex.: "Cluster 1: jovens, receita baixa, alta frequência de compra — possíveis early adopters sensíveis a preço")
- Recomendação de K-Means com k=4 como base para as campanhas, com ressalva sobre revalidar a segmentação a cada trimestre
