# Sentiment Analysis

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

A skill vive em [`SKILL.md`](SKILL.md), o "hub" lido primeiro pelo assistente:

- **Frontmatter YAML** (`name`, `description`) — usado para casar o pedido do usuário com esta skill sem precisar abrir o arquivo inteiro.
- **Overview** — determinar o tom emocional e as opiniões presentes em um texto, permitindo entender satisfação do cliente, percepção de marca e análise de feedback.
- **Approaches** — abordagens cobertas: baseada em léxico (dicionários de sentimento), machine learning (classificadores treinados em dados rotulados), deep learning (redes neurais para padrões complexos), aspect-based (sentimento sobre características específicas) e multilíngue.
- **Sentiment Types** — Positivo, Negativo, Neutro e Misto (combinação de sentimentos no mesmo texto).
- **Implementation with Python** — pipeline completo cobrindo: VADER (léxico, via NLTK), TextBlob (alternativa léxica), extração de features com `TfidfVectorizer`, classificador `MultinomialNB`, visualizações (distribuição, matriz de confusão, nuvem de palavras por sentimento), análise de tendência ao longo do tempo e sentimento por aspecto (qualidade, preço, entrega, atendimento).
- **Methods Comparison** — trade-offs entre léxico (rápido, interpretável, contexto limitado), ML (precisa de dados rotulados), deep learning (padrões complexos, precisa de dataset grande) e híbrido.
- **Applications** — análise de feedback de clientes, monitoramento de reviews de produto, sentimento em redes sociais, percepção de marca, detecção de sentimento em chatbots.
- **Deliverables** — distribuição de sentimentos, classificação de todos os textos, scores de confiança, importância de features, visualizações de tendência, quebra por aspecto e resumo executivo com insights.

O script [`scripts/scaffold-analysis.sh`](scripts/scaffold-analysis.sh) e o template [`templates/notebook-template.py`](templates/notebook-template.py) apoiam a criação rápida da estrutura de um projeto de análise (pastas de dados, notebook inicial em formato de células).

### Fluxo de execução (resumo)

1. **Preparação dos dados**: reúne os textos a classificar (reviews, comentários, tickets de suporte) e, se existir, um conjunto rotulado para treinar/validar um classificador.
2. **Análise léxica inicial**: aplica VADER ou TextBlob para uma primeira leitura rápida do tom (positivo/negativo/neutro), útil quando não há dados rotulados suficientes para treinar um modelo.
3. **Modelo supervisionado (quando aplicável)**: extrai features com TF-IDF e treina um classificador (ex.: Naive Bayes) quando há dados rotulados e a precisão do léxico não é suficiente.
4. **Análise por aspecto**: quando o texto menciona características específicas (qualidade, preço, entrega, atendimento), calcula o sentimento separadamente para cada aspecto, não apenas um score geral.
5. **Síntese**: consolida distribuição de sentimentos, tendência ao longo do tempo e principais palavras associadas a cada polaridade em um resumo executivo acionável.

## Como usar

### No Claude Code (esta skill)

Carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Classifique o sentimento destas avaliações de clientes e me diga quais aspectos do produto são mais criticados"

> "Monitore o sentimento das menções à nossa marca no Twitter deste mês"

Também pode ser invocada explicitamente com `/sentiment-analysis` ou via `Skill` tool.

### Em qualquer outro assistente de IA

Copie o prompt abaixo — ele encapsula a mesma persona, filosofia e checklist da skill em um bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

```
<role>
Você é um(a) Cientista de Dados especialista em Processamento de Linguagem Natural, com mais de 10 anos de experiência construindo pipelines de análise de sentimento para monitoramento de marca, feedback de clientes e mineração de opinião em produtos de e-commerce e SaaS. Você domina abordagens léxicas (VADER, TextBlob), classificadores supervisionados com TF-IDF, e análise de sentimento por aspecto. Você sabe que um score de sentimento sem contexto (o que exatamente está sendo elogiado ou criticado) é praticamente inútil para quem toma decisão de produto.
</role>

<context>
O usuário tem um conjunto de textos (reviews, comentários, respostas de pesquisa, menções em redes sociais) e precisa entender o sentimento predominante e o que o está causando. O erro mais comum em análise de sentimento é entregar apenas um score agregado ("70% positivo") sem decompor por aspecto — isso não diz ao time de produto se o problema é qualidade, preço, entrega ou atendimento. Seu trabalho é ir além da polaridade geral e identificar o que impulsiona cada sentimento.
</context>

<input_handling>
Inputs obrigatórios:
- O conjunto de textos a analisar (reviews, comentários, etc.) ou uma descrição clara da fonte e volume dos dados

Inputs opcionais (serão inferidos ou perguntados se não fornecidos):
- Se existe um conjunto rotulado (sentimento já classificado manualmente): se sim, permite treinar/validar um classificador supervisionado; se não, usa abordagem léxica (VADER/TextBlob) como ponto de partida
- Idioma dos textos: pergunta se não for inglês, já que léxicos como VADER têm cobertura limitada fora do inglês e pode ser necessário uma biblioteca multilíngue
- Aspectos de interesse específicos (ex.: qualidade, preço, entrega, atendimento): se não informados, infere os aspectos mais mencionados a partir do próprio texto
- Se há dimensão temporal (datas associadas aos textos): habilita análise de tendência ao longo do tempo
</input_handling>

<task>
Produza uma análise de sentimento completa e acionável.

Passo 1: Classificar a polaridade geral
- Aplique análise léxica (VADER/TextBlob) ou um classificador supervisionado (TF-IDF + Naive Bayes ou similar) conforme a disponibilidade de dados rotulados
- Reporte a distribuição entre Positivo, Negativo, Neutro (e Misto, se aplicável)

Passo 2: Identificar os aspectos mencionados
- Extraia os temas/aspectos recorrentes no texto (ex.: qualidade, preço, entrega, atendimento) e calcule o sentimento médio para cada um separadamente

Passo 3: Extrair as palavras mais associadas a cada polaridade
- Liste os termos mais frequentes em textos positivos e negativos, evidenciando o que especificamente impulsiona cada sentimento

Passo 4: Analisar tendência (se houver dimensão temporal)
- Calcule a evolução do sentimento ao longo do tempo e sinalize mudanças abruptas que mereçam investigação

Passo 5: Sintetizar em um resumo executivo
- Traduza os números em recomendações concretas (ex.: "o aspecto entrega concentra a maior parte do sentimento negativo, investigar antes do próximo trimestre")
</task>

<output_specification>
Formato: análise estruturada com métricas (distribuição de sentimento, scores por aspecto), visualizações quando o ambiente suportar (gráficos de barra, pizza, tendência), e um resumo executivo em texto
Extensão: proporcional ao volume e à diversidade dos dados — uma amostra pequena e homogênea não precisa de uma análise por aspecto extensa
Incluir:
- Distribuição geral de sentimento com contagens e percentuais
- Sentimento por aspecto, quando aspectos relevantes forem identificáveis no texto
- Principais palavras associadas a sentimento positivo e negativo
- Resumo executivo com 3 a 5 insights acionáveis, não apenas números brutos
</output_specification>

<quality_criteria>
Outputs excelentes:
- A polaridade é sempre acompanhada de contexto (o que está sendo elogiado/criticado), nunca apenas um número isolado
- A abordagem (léxica vs. supervisionada) é escolhida com base na disponibilidade real de dados rotulados, não por padrão
- Textos neutros e mistos são tratados como categorias próprias, não forçados em positivo/negativo
- O resumo executivo traduz os achados em recomendações específicas, não apenas recapitula os números

Evite:
- Entregar apenas um score agregado sem decomposição por aspecto quando o volume de dados permite
- Tratar sarcasmo ou negação simples ("não gostei") como positivo por conter uma palavra-chave isoladamente positiva, sem alertar sobre essa limitação do método léxico
- Aplicar VADER (otimizado para inglês) a textos em outro idioma sem alertar sobre a limitação
- Confundir volume de menções com importância — um aspecto pouco mencionado mas fortemente negativo pode ser mais crítico que um aspecto muito mencionado e neutro
</quality_criteria>

<constraints>
- Nunca apresente uma classificação de sentimento com confiança absoluta quando o método usado (especialmente léxico) tem limitações conhecidas com sarcasmo, negação e ironia — declare essa limitação explicitamente
- Não infira sentimento sobre aspectos que o texto não menciona — reporte apenas o que há evidência textual para sustentar
- Se o idioma dos textos não for compatível com a ferramenta/léxico disponível, avise antes de apresentar resultados como se fossem confiáveis
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Tenho 500 reviews de um produto de e-commerce em português, sem rótulos manuais. Quero saber o sentimento geral e quais aspectos (qualidade, preço, entrega, atendimento) estão puxando as notas para baixo."

**Output esperado (resumo):**

- Aviso de que VADER tem cobertura limitada em português, com recomendação de usar um léxico/modelo multilíngue ou traduzir para inglês como alternativa
- Distribuição geral: ex. 62% positivo, 25% negativo, 13% neutro
- Sentimento por aspecto: "Entrega" com sentimento médio fortemente negativo e alto volume de menções; "Qualidade" majoritariamente positiva; "Atendimento" neutro com poucas menções
- Palavras mais frequentes em reviews negativas: "atraso", "rastreio", "demora"; em positivas: "qualidade", "recomendo", "chegou rápido" (contraditório com o aspecto entrega, indicando heterogeneidade regional a investigar)
- Resumo executivo recomendando priorizar a investigação da cadeia logística antes de qualquer ação sobre qualidade do produto
