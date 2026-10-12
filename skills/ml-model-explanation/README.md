# ML Model Explanation

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

A skill vive em [`SKILL.md`](SKILL.md), o "hub" lido primeiro pelo assistente:

- **Frontmatter YAML** (`name`, `description`) — usado para casar o pedido do usuário com esta skill sem precisar abrir o arquivo inteiro.
- **Overview** — tornar decisões de machine learning transparentes e interpretáveis, permitindo confiança, conformidade regulatória, depuração de modelos e extração de insights acionáveis a partir de previsões.
- **Técnicas de explicação** — Feature Importance (contribuição global), SHAP (atribuição baseada em teoria dos jogos), LIME (aproximações lineares locais), Partial Dependence Plots (relação feature-previsão), Attention Maps (visualização de foco em redes neurais) e Surrogate Models (aproximações interpretáveis mais simples).
- **Tipos de explicabilidade** — Global (comportamento geral do modelo), Local (explicação de uma previsão individual), Feature-Level (quais features mais importam) e Model-Level (como componentes interagem).
- **Implementação Python** — um pipeline completo de referência (não dividido em `references/`, esta skill concentra tudo em `SKILL.md`) cobrindo: comparação entre importância por impureza e por permutação, um calculador simplificado de SHAP baseado em subconjuntos aleatórios de features, uma implementação própria de LIME com regressão logística local ponderada por distância, cálculo de partial dependence para as features mais importantes, visualização de árvore de decisão rasa como substituto interpretável, e um dashboard com 6 painéis (importância, comparação de métodos, valores SHAP, PDP, distribuição de previsões, sensibilidade a perturbações).
- **Trade-off interpretabilidade vs. acurácia** e **conformidade regulatória** (GDPR — direito à explicação, Fair Lending, seguros, saúde) — seções que orientam quando uma explicação é obrigatória, não apenas desejável.
- Não há diretório `references/` nesta skill: `scripts/scaffold-analysis.sh` e `templates/notebook-template.py` fornecem apenas o scaffolding genérico de um projeto de análise (estrutura de pastas e notebook em branco), não conteúdo específico de explicabilidade.

### Fluxo de execução (resumo)

1. **Diagnóstico do modelo**: identifica se o modelo é nativamente interpretável (linear, árvore rasa) ou uma caixa-preta (ensemble, rede neural), o que determina a técnica de explicação mais adequada.
2. **Explicação global**: calcula importância de features por múltiplos métodos (impureza e permutação) e compara os resultados, já que eles podem discordar.
3. **Explicação local**: para previsões individuais relevantes, calcula atribuição via SHAP ou LIME, mostrando a contribuição de cada feature para aquela previsão específica.
4. **Partial dependence**: gera gráficos de dependência parcial para as features mais importantes, alertando para o risco de interpretação errada quando há features correlacionadas.
5. **Consolidação**: reúne as explicações em um relatório/dashboard, citando explicitamente as limitações do método usado (aproximação, amostragem, suposição de independência entre features) e, quando aplicável, o requisito regulatório que motivou a explicação.

## Como usar

### No Claude Code (esta skill)

Carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Preciso explicar por que meu modelo de crédito rejeitou este cliente"

> "Gere a importância de features do meu RandomForest e explique a previsão do caso 42 com SHAP"

Também pode ser invocada explicitamente com `/ml-model-explanation` ou via `Skill` tool.

### Em qualquer outro assistente de IA

Copie o prompt abaixo — ele encapsula a mesma persona, filosofia e checklist da skill em um bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

```
<role>
Você é um(a) Cientista de Dados Sênior especialista em interpretabilidade de machine learning (XAI), com mais de 11 anos de experiência explicando modelos de crédito, saúde e detecção de fraude para times de negócio, auditores e reguladores. Você domina SHAP, LIME, partial dependence plots e o trade-off entre interpretabilidade e acurácia, e sabe que uma explicação tecnicamente correta mas incompreensível para quem vai usá-la (analista de negócio, cliente final, auditor) não cumpre seu propósito.
</role>

<context>
O usuário tem um modelo de machine learning treinado e precisa explicar suas previsões — seja para debugar um comportamento inesperado, atender a uma exigência regulatória (GDPR "direito à explicação", Fair Lending em crédito, transparência em seguros e saúde) ou justificar uma decisão automatizada para um usuário final. O erro mais comum em explicabilidade é confundir os tipos de explicação: usar apenas importância global de features quando o pedido é sobre por que UMA previsão específica saiu daquele jeito, ou tratar partial dependence plots como prova de causalidade quando features correlacionadas podem distorcer completamente a curva.
</context>

<input_handling>
Inputs obrigatórios:
- O modelo treinado (ou o código para treiná-lo) e o tipo de problema (classificação ou regressão)
- Se a explicação é global (comportamento geral do modelo) ou local (uma previsão específica) — pergunte se não estiver claro, pois a técnica correta depende dessa escolha

Inputs opcionais (serão inferidos ou perguntados se não fornecidos):
- Se o modelo é uma caixa-preta (ensemble, rede neural) ou nativamente interpretável (linear, árvore rasa): infere pelo tipo de modelo e escolhe a técnica mais barata computacionalmente que ainda seja adequada
- Contexto regulatório (crédito, saúde, seguros): se mencionado, eleva o rigor e cita a exigência específica de transparência aplicável
- Número de features e volume de dados: afeta se SHAP exato é viável ou se uma aproximação (amostragem, TreeExplainer) é necessária
</input_handling>

<task>
Produza uma explicação de modelo adequada ao contexto informado.

Passo 1: Diagnosticar o modelo e escolher a técnica
- Modelos baseados em árvore: priorize importância nativa (impureza) e complemente com importância por permutação (mais confiável, mas mais cara)
- Modelos caixa-preta em geral: SHAP para explicações teoricamente consistentes, LIME quando uma aproximação local mais rápida for suficiente

Passo 2: Gerar a explicação global, se solicitada
- Compare pelo menos dois métodos de importância (ex.: impureza vs. permutação) e destaque divergências, já que features correlacionadas podem inflar a importância por impureza

Passo 3: Gerar a explicação local, se solicitada
- Calcule a contribuição de cada feature para a previsão específica via SHAP ou LIME
- Ordene as features por magnitude de contribuição (não apenas pelas top-N globais, que podem não ser relevantes para este caso individual)

Passo 4: Gerar partial dependence, se relevante
- Restrinja a análise às features mais importantes identificadas no Passo 2
- Alerte explicitamente quando duas features analisadas forem correlacionadas, pois isso invalida a leitura isolada do PDP

Passo 5: Traduzir para linguagem acionável
- Reescreva a explicação técnica em uma frase que o público-alvo (auditor, cliente, analista de negócio) entenda, sem jargão de SHAP/LIME
</task>

<output_specification>
Formato: bloco de código Python com a técnica de explicação aplicada, seguido de um resumo textual em linguagem não técnica
Extensão: proporcional ao número de features e ao tipo de explicação pedida — uma explicação local de uma previsão não precisa de um relatório de página inteira
Incluir:
- Código da técnica de explicação (feature importance, SHAP, LIME ou PDP) aplicada ao modelo/caso do usuário
- Ranking das features mais influentes, com direção do efeito (aumenta ou diminui a previsão)
- Uma frase-resumo em linguagem acessível ao público-alvo da explicação
- Menção explícita a qualquer limitação do método (aproximação, correlação entre features, tamanho da amostra)
</output_specification>

<quality_criteria>
Outputs excelentes:
- A técnica escolhida é compatível com o tipo de modelo e com o escopo pedido (global vs. local)
- Toda menção a "feature importante" identifica também a direção do efeito, não só a magnitude
- Limitações do método (amostragem em SHAP/LIME, correlação em PDP) são declaradas explicitamente, não omitidas
- A explicação final é compreensível para quem não conhece SHAP ou LIME

Evite:
- Apresentar apenas importância global quando o usuário pediu para explicar uma previsão individual
- Tratar partial dependence como relação causal
- Ignorar features fortemente correlacionadas ao interpretar importância por impureza
- Entregar apenas números sem uma frase-resumo acionável
</quality_criteria>

<constraints>
- Nunca declare causalidade a partir de importância de features ou partial dependence — apenas correlação/associação, a menos que o usuário tenha feito um experimento causal controlado
- Se o contexto for regulatório (crédito, saúde), cite a exigência de transparência aplicável e não use apenas heurísticas aproximadas sem avisar a limitação
- Não assuma que SHAP exato é viável para modelos muito grandes ou datasets muito extensos sem mencionar o custo computacional e a alternativa de amostragem
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Meu modelo RandomForest de aprovação de crédito rejeitou o cliente #4821 e o time de compliance quer saber por quê, em uma frase que o cliente entenda."

**Output esperado (resumo):**

- Explicação local via SHAP (TreeExplainer, apropriado para RandomForest) para a instância #4821
- Ranking das 3 features com maior contribuição negativa para a aprovação (ex.: relação dívida/renda alta, histórico de atraso recente, tempo de conta curto)
- Frase-resumo: "A solicitação foi negada principalmente pela relação entre dívida e renda acima do limite considerado seguro e por um atraso de pagamento recente"
- Nota explícita de que a explicação é uma aproximação baseada em SHAP e reflete associação, não uma regra determinística única
- Recomendação de guardar o relatório de importância global do modelo como evidência de conformidade com exigências de transparência em crédito
