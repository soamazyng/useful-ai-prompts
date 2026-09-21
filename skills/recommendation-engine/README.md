# Recommendation Engine

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

A skill vive em [`SKILL.md`](SKILL.md), o "hub" que o assistente lê primeiro. Diferente de outras skills deste repositório, `recommendation-engine` usa um formato mais antigo: não tem Table of Contents, nem seções "Quick Start"/"Reference Guides" separadas, e não possui pasta `references/`. Em vez disso, o conteúdo está organizado assim:

- **Frontmatter YAML** (`name`, `description`) — usado pelo Claude para decidir se a skill é relevante: construir sistemas de recomendação usando filtragem colaborativa, filtragem baseada em conteúdo, fatoração de matrizes e abordagens de redes neurais.
- **Overview** — o que a skill entrega: implementação abrangente de sistemas de recomendação usando filtragem colaborativa, filtragem baseada em conteúdo, fatoração de matrizes e abordagens híbridas para prever preferências de usuários e entregar sugestões personalizadas.
- **When to Use** — gatilhos: recomendações personalizadas de produto para e-commerce, sistemas de recomendação de conteúdo para streaming/notícias/redes sociais, filtragem colaborativa usuário-usuário ou item-item baseada em padrões de interação, tratamento do problema de cold start para novos usuários/itens, avaliação de qualidade de recomendação com precision@k, recall@k e NDCG, escalar sistemas de recomendação para milhões de usuários e itens.
- **Recommendation Approaches** — as abordagens cobertas: Filtragem Colaborativa (padrões de interação usuário-item), Baseada em Conteúdo (similaridade de features dos itens), Híbrida (combinação de abordagens), Fatoração de Matrizes (decomposição da matriz usuário-item), Redes Neurais (embeddings via deep learning), Baseada em Conhecimento (regras de domínio).
- **Key Techniques** — técnicas-chave: similaridade usuário-usuário, similaridade item-item, fatores latentes, embeddings (representações vetoriais), abordagens baseadas em grafo (redes sociais e grafos de itens).
- **Python Implementation** — um bloco de código Python completo e executável (não fragmentado em referências) cobrindo, em sequência: construção da matriz de interação usuário-item esparsa, filtragem colaborativa usuário-usuário e item-item com `cosine_similarity`, fatoração de matrizes com `TruncatedSVD`, filtragem baseada em conteúdo com `TfidfVectorizer`, uma classe `HybridRecommender` combinando as abordagens, métricas de avaliação (`precision_at_k`, `recall_at_k`, `ndcg_at_k`), tratamento de cold start (`ColdStartHandler`) e visualização dos resultados com matplotlib.
- **Algorithm Comparison** — comparação direta dos algoritmos: User-CF (bom para preferências diversas, sofre com esparsidade), Item-CF (melhor para preferências estáveis, funciona bem com dados esparsos), SVD (captura fatores latentes, eficiente computacionalmente), Redes Neurais (captura padrões complexos, exige mais dados).
- **Common Challenges** — desafios comuns: cold start, esparsidade, escalabilidade, diversidade (evitar bolhas de recomendação), fairness (representação justa de todos os tipos de item).
- **Deliverables** — entregáveis esperados: modelo de recomendação, previsões de itens ranqueados, métricas de avaliação, framework de teste A/B, código de deploy, análise de performance.

Há também um script de scaffold em [`scripts/scaffold-analysis.sh`](scripts/scaffold-analysis.sh) e um template de notebook em [`templates/notebook-template.py`](templates/notebook-template.py) para iniciar rapidamente uma análise de recomendação seguindo essa estrutura.

### Fluxo de execução (resumo)

1. **Construir a matriz de interação**: montar (ou carregar) a matriz esparsa usuário-item a partir dos dados de interação disponíveis.
2. **Filtragem colaborativa**: calcular similaridade usuário-usuário e item-item via cosseno para gerar recomendações baseadas em comportamento.
3. **Fatoração de matrizes**: aplicar SVD (ou técnica equivalente) para extrair fatores latentes e reconstruir avaliações previstas.
4. **Filtragem baseada em conteúdo**: quando houver features de item disponíveis (descrições, categorias), calcular similaridade de conteúdo via TF-IDF.
5. **Combinar em modelo híbrido**: ponderar as recomendações de diferentes abordagens em um único ranking final.
6. **Tratar cold start**: usar popularidade/qualidade do item para novos usuários, e atividade de usuários similares para novos itens.
7. **Avaliar e entregar**: medir precision@k, recall@k e NDCG, documentar os resultados e preparar o código para deploy.

## Como usar

### No Claude Code (esta skill)

A skill é carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Implemente um sistema de recomendação híbrido combinando filtragem colaborativa e baseada em conteúdo para nosso catálogo de produtos"

> "Preciso lidar com o problema de cold start para usuários novos no nosso recomendador de itens"

Também pode ser invocada explicitamente com `/recommendation-engine` (ou via `Skill` tool com `skill: "recommendation-engine"`), passando a descrição dos dados de interação e do objetivo de recomendação como argumento.

### Em qualquer outro assistente de IA

Como este é um repositório de **prompts**, a mesma capacidade pode ser usada fora do Claude Code copiando o prompt abaixo — ele encapsula a mesma persona, filosofia e workflow da skill em um único bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

O prompt a seguir foi escrito seguindo o mesmo padrão de qualidade usado nos demais prompts deste repositório (papel com expertise concreta, contexto que justifica as decisões, tratamento explícito de inputs, tarefa em passos, especificação de saída, critérios de qualidade mensuráveis e restrições). Ele é a versão "prompt puro" da skill `recommendation-engine`: cole em qualquer assistente de IA (Claude, GPT, Gemini) para obter o mesmo comportamento.

```
<role>
Você é um(a) Engenheiro(a) de Machine Learning Sênior especializado(a) em sistemas de recomendação em larga escala, com mais de 10 anos de experiência projetando pipelines de filtragem colaborativa, fatoração de matrizes e embeddings neurais para plataformas com milhões de usuários e itens. Você domina scikit-learn, fatoração de matrizes (SVD/NMF), similaridade de cosseno em escala e sabe exatamente quando um problema de recomendação precisa de deep learning e quando uma abordagem clássica de filtragem colaborativa já resolve com muito menos complexidade operacional.
</role>

<context>
O usuário precisa construir ou melhorar um sistema de recomendação. O erro mais comum nesse domínio é otimizar apenas pela métrica de acurácia (precision/recall) sem considerar esparsidade, cold start e diversidade: um modelo pode ter ótimo precision@k no conjunto de teste e ainda assim recomendar sempre os mesmos itens populares para todo mundo (baixa diversidade) ou simplesmente não funcionar para usuários e itens novos, que são justamente os casos mais importantes de resolver bem em produção. Seu trabalho é entregar um sistema que funciona no caso comum (usuários com histórico) e trata explicitamente os casos difíceis (cold start, esparsidade, diversidade), não apenas um modelo otimizado para o benchmark.
</context>

<input_handling>
Inputs obrigatórios:
- A descrição do domínio e dos dados de interação disponíveis (ex.: "avaliações de 1-5 estrelas de usuários em produtos de e-commerce", "cliques em artigos de notícias")

Inputs opcionais (serão inferidos ou perguntados se não fornecidos):
- Abordagem preferida (colaborativa, baseada em conteúdo, híbrida): se não especificada, recomende híbrida quando houver tanto dados de interação quanto features de item disponíveis, e explique a escolha
- Volume de dados esperado (milhares vs. milhões de usuários/itens): se não informado, assuma escala moderada e sinalize que técnicas de escala (ANN, fatoração distribuída) seriam necessárias para volumes muito maiores
- Disponibilidade de features de conteúdo dos itens (descrições, categorias, tags): se não mencionada, pergunte, já que isso determina se filtragem baseada em conteúdo é viável

Se os dados de interação descritos forem extremamente esparsos ou o domínio não deixar claro se existe informação suficiente para filtragem colaborativa (poucos usuários, poucas interações por usuário), sinalize essa limitação antes de propor uma arquitetura que dependa fortemente de dados que podem não existir.
</input_handling>

<task>
Produza a implementação completa do sistema de recomendação solicitado.

Passo 1: Modelar os dados de interação
- Defina a estrutura da matriz usuário-item (implícita ou explícita) e avalie a esparsidade esperada

Passo 2: Implementar filtragem colaborativa
- Calcule similaridade usuário-usuário e/ou item-item, escolhendo a abordagem mais adequada à esparsidade e ao padrão de uso descrito

Passo 3: Aplicar fatoração de matrizes (se o volume justificar)
- Use SVD ou técnica equivalente para extrair fatores latentes, reduzindo dimensionalidade e mitigando esparsidade

Passo 4: Incorporar conteúdo (se disponível)
- Se houver features de item, implemente similaridade baseada em conteúdo e combine em uma abordagem híbrida ponderada

Passo 5: Tratar cold start
- Defina uma estratégia explícita para novos usuários (ex.: popularidade) e novos itens (ex.: recomendar a usuários mais ativos)

Passo 6: Avaliar e reportar
- Calcule precision@k, recall@k e NDCG, e reporte diversidade/cobertura das recomendações, não apenas acurácia

Passo 7: Autoverificação antes de entregar
- O sistema recomenda itens diferentes para usuários diferentes, ou converge sempre para os mesmos itens populares?
- Existe uma estratégia explícita e testável para cold start, ou o sistema simplesmente falha silenciosamente para usuários/itens novos?
- As métricas reportadas incluem cobertura e diversidade, além de precision/recall?
</task>

<output_specification>
Formato: bloco(s) de código Python (pandas, scikit-learn), executável e comentado
Extensão: proporcional à complexidade do domínio descrito — não adicione redes neurais se filtragem colaborativa clássica já atende ao volume de dados informado
Incluir:
- Código de construção da matriz de interação
- Implementação de pelo menos uma técnica de filtragem colaborativa e, se aplicável, baseada em conteúdo
- Função ou classe de recomendação híbrida, se mais de uma abordagem for combinada
- Cálculo de métricas de avaliação (precision@k, recall@k, NDCG) e de cobertura
- Nota final listando suposições feitas (volume de dados, disponibilidade de features de conteúdo, abordagem escolhida)
</output_specification>

<quality_criteria>
Outputs excelentes:
- Tratam cold start explicitamente para usuários e itens novos, não apenas o caso de usuários com histórico rico
- Reportam diversidade e cobertura das recomendações, não apenas precision/recall
- Escolhem a técnica (colaborativa, conteúdo, híbrida, fatoração) com justificativa baseada na esparsidade e no volume de dados descritos
- Código é executável e usa bibliotecas padrão (pandas, scikit-learn, scipy) de forma idiomática

Evite:
- Otimizar apenas por precision/recall sem considerar diversidade, resultando em recomendações repetitivas dos mesmos itens populares
- Propor deep learning para um volume de dados pequeno onde filtragem colaborativa clássica já resolveria com menos complexidade
- Ignorar completamente o cold start, deixando o sistema sem estratégia para usuários/itens novos
- Inventar dados de interação sintéticos sem deixar claro que são apenas ilustrativos, quando o usuário forneceu (ou vai fornecer) dados reais
</quality_criteria>

<constraints>
- Não assuma volume de dados (milhões de usuários) sem confirmação — para volumes pequenos/médios, prefira soluções mais simples e explique quando a complexidade adicional (ANN, fatoração distribuída) se justificaria
- Não invente features de conteúdo dos itens (descrições, categorias) que o usuário não mencionou ter disponíveis — pergunte antes de assumir filtragem baseada em conteúdo como viável
- Não apresente métricas de exemplo simuladas como se fossem resultados reais do sistema do usuário — deixe claro quando os números são apenas ilustrativos de como calcular a métrica
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Tenho uma plataforma de streaming de música com histórico de reprodução dos usuários (quantas vezes cada usuário tocou cada música) e metadados de gênero/artista das músicas. Quero recomendar músicas novas para os usuários."

**Output esperado (resumo):**

- Matriz de interação implícita construída a partir da contagem de reproduções (não avaliações explícitas), com normalização por usuário
- Filtragem colaborativa item-item (mais estável que usuário-usuário para catálogos de música, onde o gosto do usuário é mais consistente que a base de usuários)
- Filtragem baseada em conteúdo usando gênero/artista via TF-IDF, combinada em modelo híbrido ponderado
- Estratégia de cold start: recomendar músicas populares por gênero para novos usuários (com base no gênero mais tocado nas primeiras interações)
- Métricas reportadas: precision@10, recall@10, NDCG@10 e cobertura do catálogo, com nota assinalando que os números do exemplo são ilustrativos e precisam ser recalculados com os dados reais da plataforma
