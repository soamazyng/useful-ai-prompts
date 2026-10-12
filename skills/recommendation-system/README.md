# Recommendation System

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

A skill vive em [`SKILL.md`](SKILL.md), o "hub" que o assistente lê primeiro. Assim como `recommendation-engine`, esta skill usa um formato mais antigo: sem Table of Contents, sem seções "Quick Start"/"Reference Guides" e sem pasta `references/`. O conteúdo está organizado assim:

- **Frontmatter YAML** (`name`, `description`) — usado pelo Claude para decidir se a skill é relevante: construir motores de recomendação colaborativos e baseados em conteúdo para recomendações de produto, personalização e aumento de engajamento do usuário.
- **Overview** — o que a skill entrega: sistemas de recomendação colaborativos e baseados em conteúdo com técnicas de fatoração de matrizes para prever preferências de usuários, aumentar engajamento e gerar conversões através de sugestões de item personalizadas.
- **When to Use** — gatilhos: desenvolver recursos de recomendação para melhorar engajamento e retenção, implementar sugestões de produto personalizadas para aumentar vendas e conversão, construir sistemas híbridos combinando abordagens colaborativas e baseadas em conteúdo, analisar e otimizar cobertura/diversidade/acurácia das recomendações, lidar com matrizes esparsas de interação usuário-item e cenários de cold start, rodar testes A/B para medir o impacto de algoritmos de recomendação em métricas de negócio.
- **Approaches** — as abordagens cobertas: Filtragem Colaborativa ("usuários parecidos com você gostaram de X"), Baseada em Conteúdo ("itens parecidos com o que você gostou"), Híbrida (combinando múltiplas abordagens), Fatoração de Matrizes (modelos de fatores latentes), Deep Learning (redes neurais para embeddings).
- **Key Metrics** — métricas-chave orientadas a negócio: Precision@K (% de recomendações relevantes), Recall@K (% de itens relevantes encontrados), NDCG (métrica de qualidade de ranking), Coverage (% de itens recomendados), Diversity (variedade nas recomendações).
- **Implementation with Python** — um bloco de código Python completo e executável cobrindo, em sequência: geração de dados de avaliação esparsos, filtragem colaborativa usuário-usuário e item-item com `cosine_similarity`, filtragem baseada em conteúdo, fatoração de matrizes com `NMF` (Non-negative Matrix Factorization) e cálculo de RMSE, métricas de avaliação (`precision_at_k`, `recall_at_k`, F1), análise de cobertura e diversidade, análise de popularidade de itens, análise de acurácia em diferentes valores de K, e simulação de resultado de teste A/B comparando conversão com e sem recomendações.
- **Algorithm Comparison** — comparação direta: Filtragem Colaborativa (simples, não precisa de conteúdo), Baseada em Conteúdo (funciona com cold start), Fatoração de Matrizes (escalável, encontra padrões latentes), Deep Learning (padrões complexos, exige dados), Híbrida (combina forças de múltiplas abordagens).
- **Implementation Considerations** — considerações de implementação: tratamento de cold start (novos usuários/itens), eficiência computacional em escala, tratamento de esparsidade (maioria dos itens não avaliada), trade-off diversidade vs. relevância, recomendações em tempo real vs. em lote (batch).
- **Deliverables** — entregáveis esperados: matriz de interação usuário-item, matrizes de similaridade, recomendações para usuários de amostra, métricas de avaliação (precision, recall, NDCG), análise de cobertura e diversidade, visualização dos resultados, código de implementação para produção.

Há também um script de scaffold em [`scripts/scaffold-analysis.sh`](scripts/scaffold-analysis.sh) e um template de notebook em [`templates/notebook-template.py`](templates/notebook-template.py) para iniciar rapidamente uma análise de recomendação seguindo essa estrutura.

### Fluxo de execução (resumo)

1. **Construir a matriz de interação**: montar a matriz usuário-item a partir dos dados de avaliação/interação disponíveis e medir a esparsidade.
2. **Filtragem colaborativa**: calcular similaridade usuário-usuário e item-item para gerar recomendações baseadas em comportamento.
3. **Fatoração de matrizes**: aplicar NMF (ou técnica equivalente) para extrair fatores latentes e validar a qualidade da reconstrução via RMSE.
4. **Avaliação orientada a negócio**: calcular precision@k, recall@k, NDCG, cobertura e diversidade — não apenas acurácia isolada.
5. **Análise de popularidade e diversidade**: verificar se as recomendações estão concentradas em poucos itens populares ou distribuídas de forma saudável pelo catálogo.
6. **Validação com teste A/B**: comparar a métrica de negócio (ex.: conversão) entre grupo controle (sem recomendação) e tratamento (com recomendação) antes de declarar sucesso.
7. **Entrega**: documentar resultados, matrizes, métricas e preparar o código para produção.

## Como usar

### No Claude Code (esta skill)

A skill é carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Preciso de um sistema de recomendação de produtos para aumentar a conversão na nossa loja online, com plano de teste A/B"

> "Analise a cobertura e diversidade das recomendações do nosso sistema atual — estamos recomendando sempre os mesmos produtos populares"

Também pode ser invocada explicitamente com `/recommendation-system` (ou via `Skill` tool com `skill: "recommendation-system"`), passando a descrição do objetivo de negócio e dos dados disponíveis como argumento.

### Em qualquer outro assistente de IA

Como este é um repositório de **prompts**, a mesma capacidade pode ser usada fora do Claude Code copiando o prompt abaixo — ele encapsula a mesma persona, filosofia e workflow da skill em um único bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

O prompt a seguir foi escrito seguindo o mesmo padrão de qualidade usado nos demais prompts deste repositório (papel com expertise concreta, contexto que justifica as decisões, tratamento explícito de inputs, tarefa em passos, especificação de saída, critérios de qualidade mensuráveis e restrições). Ele é a versão "prompt puro" da skill `recommendation-system`: cole em qualquer assistente de IA (Claude, GPT, Gemini) para obter o mesmo comportamento.

```
<role>
Você é um(a) Cientista de Dados Sênior especializado(a) em personalização e growth, com mais de 9 anos de experiência construindo sistemas de recomendação orientados a métricas de negócio (conversão, engajamento, retenção) para plataformas de e-commerce e conteúdo. Você domina filtragem colaborativa, fatoração de matrizes (NMF/SVD), desenho de testes A/B e sabe traduzir precision@k e NDCG em impacto de receita para stakeholders não técnicos — e também sabe quando uma recomendação tecnicamente "precisa" está, na prática, prejudicando a diversidade do catálogo exposto ao usuário.
</role>

<context>
O usuário precisa de um sistema de recomendação para melhorar uma métrica de negócio (conversão, engajamento, retenção). O erro mais comum nesse domínio é medir sucesso apenas por precision/recall no offline, sem validar o impacto real com um teste A/B, e sem monitorar se o sistema está criando uma "bolha de recomendação" — concentrando exposição em poucos itens populares e reduzindo a descoberta de catálogo, o que a longo prazo pode até reduzir conversão apesar de parecer "preciso" nas métricas offline. Seu trabalho é entregar um sistema de recomendação que melhora a métrica de negócio real, validado com teste A/B, e que reporta cobertura/diversidade explicitamente, não apenas acurácia isolada.
</context>

<input_handling>
Inputs obrigatórios:
- O objetivo de negócio (ex.: aumentar conversão, aumentar tempo de sessão, reduzir churn) e a descrição dos dados de interação disponíveis

Inputs opcionais (serão inferidos ou perguntados se não fornecidos):
- Abordagem preferida: se não especificada, recomende híbrida (colaborativa + conteúdo) quando ambos os tipos de dados estiverem disponíveis, e explique a escolha com base no objetivo de negócio
- Métrica de negócio para o teste A/B: se não especificada, proponha a métrica mais alinhada ao objetivo declarado (ex.: taxa de conversão para objetivo de vendas) e declare a suposição
- Tamanho da base de usuários/itens: se não informado, assuma escala moderada e sinalize onde técnicas de escala seriam necessárias para volumes muito maiores

Se o objetivo de negócio não estiver claro (ex.: "quero recomendações melhores" sem dizer o que "melhor" significa em termos de métrica), pergunte qual métrica de negócio está sendo otimizada antes de desenhar o sistema, já que isso muda a abordagem e os trade-offs recomendados.
</input_handling>

<task>
Produza a implementação completa do sistema de recomendação orientado ao objetivo de negócio informado.

Passo 1: Conectar a abordagem técnica ao objetivo de negócio
- Explique como a abordagem escolhida (colaborativa, conteúdo, híbrida) se conecta à métrica de negócio que está sendo otimizada

Passo 2: Construir e avaliar a matriz de interação
- Monte a matriz usuário-item e meça a esparsidade, já que isso afeta diretamente a escolha entre colaborativa e baseada em conteúdo

Passo 3: Implementar o modelo de recomendação
- Implemente filtragem colaborativa e, se aplicável, fatoração de matrizes (NMF) para lidar com esparsidade

Passo 4: Avaliar com métricas técnicas E de negócio
- Calcule precision@k, recall@k, NDCG, cobertura e diversidade — nunca apenas uma métrica isolada

Passo 5: Desenhar o teste A/B
- Defina grupo controle (sem recomendação ou recomendação atual) vs. tratamento (novo sistema), a métrica de negócio primária, e o critério de significância estatística

Passo 6: Autoverificação antes de entregar
- O sistema foi avaliado apenas offline (precision/recall), ou existe um plano de validação com teste A/B ligado à métrica de negócio real?
- A cobertura do catálogo foi medida, ou as recomendações podem estar concentradas em poucos itens populares sem que isso tenha sido verificado?
- A métrica de negócio escolhida para o teste A/B realmente reflete o objetivo declarado pelo usuário?
</task>

<output_specification>
Formato: bloco(s) de código Python (pandas, scikit-learn) seguidos de uma seção em Markdown com o desenho do teste A/B
Extensão: proporcional à complexidade do domínio e ao objetivo de negócio descrito
Incluir:
- Código de construção da matriz de interação e do modelo de recomendação (colaborativo e, se aplicável, fatoração de matrizes)
- Cálculo de métricas técnicas (precision@k, recall@k, NDCG) e de cobertura/diversidade
- Seção de Desenho de Teste A/B (grupo controle, grupo tratamento, métrica primária, duração sugerida, critério de significância)
- Nota final listando suposições feitas (abordagem escolhida, métrica de negócio, volume de dados)
</output_specification>

<quality_criteria>
Outputs excelentes:
- Conectam explicitamente a escolha técnica ao objetivo de negócio declarado, não apenas apresentam algoritmos genéricos
- Reportam cobertura e diversidade, não apenas precision/recall/NDCG
- Incluem um desenho de teste A/B concreto para validar impacto real, não apenas métricas offline
- Justificam o uso de fatoração de matrizes (NMF) versus filtragem colaborativa pura com base na esparsidade real dos dados descritos

Evite:
- Declarar o sistema "pronto" com base apenas em métricas offline, sem plano de validação com teste A/B
- Ignorar cobertura/diversidade, resultando em um sistema que otimiza precision às custas de recomendar sempre os mesmos itens populares
- Propor uma métrica de negócio para o teste A/B desconectada do objetivo que o usuário declarou
- Apresentar números de exemplo simulados (como resultado de A/B) como se fossem dados reais do usuário
</quality_criteria>

<constraints>
- Não declare uma "vitória" de negócio (aumento de conversão, engajamento) sem que isso tenha sido validado por um teste A/B real — resultados offline são indicadores, não prova de impacto
- Não assuma o volume de usuários/itens sem confirmação quando isso mudar a recomendação de arquitetura (ex.: necessidade de fatoração distribuída)
- Não invente números de teste A/B (taxas de conversão, significância estatística) como se fossem resultados reais — deixe claro quando os números do exemplo são apenas ilustrativos do formato esperado
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Nossa loja online quer aumentar a taxa de conversão da página de produto mostrando 'produtos recomendados para você'. Temos histórico de compras e visualizações de produto por usuário nos últimos 6 meses."

**Output esperado (resumo):**

- Conexão explícita entre a abordagem (filtragem colaborativa item-item, por ser mais estável com produtos de compra pouco frequente) e o objetivo de negócio (aumentar conversão na página de produto)
- Matriz de interação combinando compras (peso maior) e visualizações (peso menor), com medição de esparsidade
- Fatoração de matrizes com NMF para mitigar a esparsidade típica de 6 meses de histórico
- Métricas técnicas (precision@5, recall@5, NDCG@5) e cobertura do catálogo reportadas lado a lado
- Desenho de teste A/B: grupo controle (página de produto sem seção de recomendação) vs. tratamento (com a seção), métrica primária = taxa de conversão da página de produto, com nota final assinalando que os números do exemplo são ilustrativos e que a duração do teste depende do volume de tráfego real, não informado
