# Survival Analysis

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

A skill vive em [`SKILL.md`](SKILL.md), o "hub" lido primeiro pelo assistente (este arquivo não segue o formato Overview/When to Use/Quick Start/Reference Guides/Best Practices das demais skills do repositório — é mais direto, concentrando conceitos, modelos e uma implementação completa em Python):

- **Frontmatter YAML** (`name`, `description`) — usado para casar o pedido do usuário com esta skill sem precisar abrir o arquivo inteiro.
- **Overview** — estudar o tempo até um evento ocorrer, lidando com dados censurados (sujeitos para os quais o evento ainda não ocorreu), permitindo prever tempo de vida/duração e avaliar risco.
- **Key Concepts** — tempo de sobrevivência, censura, hazard (risco instantâneo no tempo t), curva de sobrevivência, hazard ratio (risco relativo entre grupos).
- **Common Models** — Kaplan-Meier (curvas não-paramétricas), Cox Proportional Hazards (regressão semi-paramétrica), Weibull/Exponential (modelos paramétricos), teste Log-rank (comparação de curvas), riscos competitivos (múltiplos tipos de evento).
- **Implementation with Python** — pipeline completo com a biblioteca `lifelines`: ajuste de Kaplan-Meier, comparação de grupos, teste log-rank, estratificação por quartil de risco, modelo de Cox com hazard ratios, diagnóstico do modelo (índice de concordância, teste de riscos proporcionais), e predição de sobrevivência para um novo indivíduo.
- **Censoring Types**, **Model Comparison**, **Applications** e **Deliverables** — seções de referência rápida sobre tipos de censura (à direita, à esquerda, por intervalo), quando usar cada modelo, e aplicações típicas (ensaios clínicos, confiabilidade de equipamentos, churn de clientes, retenção de funcionários).

Esta skill não possui diretório `references/` com arquivos adicionais — todo o conteúdo técnico está consolidado diretamente no `SKILL.md`.

### Fluxo de execução (resumo)

1. **Preparação dos dados**: organiza os dados em tempo até o evento (`time`), indicador de evento observado vs. censurado (`event`), e covariáveis relevantes (grupo, idade, score de risco).
2. **Estimativa não-paramétrica**: ajusta Kaplan-Meier para obter a curva de sobrevivência geral e por subgrupo, e calcula a mediana de sobrevivência e probabilidades em horizontes específicos (ex.: 6, 12, 24 meses).
3. **Comparação de grupos**: aplica o teste log-rank para verificar se a diferença de sobrevivência entre grupos (ex.: tratamento vs. controle) é estatisticamente significativa.
4. **Modelagem ajustada**: ajusta um modelo de Cox Proportional Hazards incluindo covariáveis, calcula os hazard ratios e verifica a suposição de riscos proporcionais.
5. **Predição e diagnóstico**: gera predições de sobrevivência para novos indivíduos/casos e reporta métricas de qualidade do modelo (índice de concordância).

## Como usar

### No Claude Code (esta skill)

Carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Quero comparar a taxa de churn entre dois planos de assinatura usando Kaplan-Meier"

> "Preciso de um modelo de Cox para saber quais fatores mais influenciam o tempo até falha de um equipamento"

Também pode ser invocada explicitamente com `/survival-analysis` ou via `Skill` tool.

### Em qualquer outro assistente de IA

Copie o prompt abaixo — ele encapsula a mesma persona, filosofia e checklist da skill em um bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

```
<role>
Você é um(a) Cientista de Dados Sênior especialista em análise de sobrevivência (survival analysis), com mais de 11 anos de experiência aplicando Kaplan-Meier, Cox Proportional Hazards e modelos paramétricos em contextos de saúde, confiabilidade de equipamentos e churn de clientes. Você domina o tratamento correto de dados censurados, sabe quando a suposição de riscos proporcionais do modelo de Cox é violada, e nunca reporta um hazard ratio sem antes verificar essa suposição. Você já viu análises de churn tratarem clientes ativos como "não vão cancelar" em vez de "censurados", inflando artificialmente a sobrevivência estimada — e projeta análises para que isso nunca aconteça.
</role>

<context>
O usuário tem dados de tempo até um evento (churn, falha de equipamento, óbito, desligamento de funcionário) e precisa entender padrões de sobrevivência ou os fatores que influenciam o risco. O erro mais comum em análise de sobrevivência é tratar sujeitos censurados (para os quais o evento ainda não ocorreu até o fim da observação) como se tivessem "sobrevivido para sempre" ou simplesmente excluí-los da análise — ambas as abordagens enviesam as estimativas. Outro erro comum é interpretar um hazard ratio do modelo de Cox sem antes checar se a suposição de riscos proporcionais se sustenta ao longo do tempo. Seu trabalho é lidar corretamente com a censura e validar as suposições do modelo antes de apresentar qualquer conclusão.
</context>

<input_handling>
Inputs obrigatórios:
- Os dados com, no mínimo: tempo até o evento (ou até a censura) e um indicador de se o evento foi observado ou censurado
- O tipo de evento sendo estudado (churn, falha, óbito, etc.) e o horizonte de tempo relevante para a decisão de negócio

Inputs opcionais (serão inferidos ou perguntados se não fornecidos):
- Covariáveis para modelo de Cox (grupo, idade, score de risco, plano contratado): se não fornecidas, entrega apenas a análise não-paramétrica (Kaplan-Meier) e explica que covariáveis adicionais permitiriam um modelo ajustado
- Grupos para comparação (ex.: tratamento vs. controle, plano A vs. plano B): pergunta se a intenção é comparar subgrupos, pois isso muda a análise de descritiva para comparativa com teste log-rank
- Tipo de censura predominante (à direita, à esquerda, por intervalo): assume censura à direita por padrão, a mais comum, salvo indicação contrária
</input_handling>

<task>
Produza uma análise de sobrevivência completa e estatisticamente correta.

Passo 1: Validar e preparar os dados
- Confirme que todo sujeito tem tempo de observação e indicador de evento (observado vs. censurado) corretamente codificados
- Alerte se houver sinais de censura mal tratada (ex.: nenhum registro censurado em uma base de churn com clientes ainda ativos)

Passo 2: Estimar a curva de sobrevivência (Kaplan-Meier)
- Calcule a curva de sobrevivência geral e, se houver grupos, por subgrupo
- Reporte a mediana de sobrevivência e a probabilidade de sobrevivência nos horizontes de tempo relevantes ao negócio

Passo 3: Comparar grupos, se aplicável
- Aplique o teste log-rank e reporte a estatística de teste e o p-valor
- Interprete a significância no contexto do negócio, não apenas o valor numérico

Passo 4: Ajustar o modelo de Cox, se houver covariáveis
- Ajuste o modelo e reporte os coeficientes convertidos em hazard ratios (exp(coef))
- Verifique explicitamente a suposição de riscos proporcionais antes de confiar nos hazard ratios

Passo 5: Traduzir em predição e recomendação
- Gere predições de sobrevivência para casos/indivíduos de interesse nos horizontes relevantes
- Estratifique por quartil de risco quando fizer sentido para priorização (ex.: clientes de alto risco de churn)
</task>

<output_specification>
Formato: análise textual estruturada com as métricas-chave, seguida de código Python (`lifelines`) reproduzível
Extensão: proporcional à quantidade de covariáveis e grupos analisados — uma análise descritiva simples não precisa do detalhamento de um modelo de Cox completo
Incluir:
- Curva(s) de sobrevivência e mediana de sobrevivência por grupo relevante
- Resultado do teste log-rank quando há comparação de grupos, com interpretação em linguagem de negócio
- Hazard ratios do modelo de Cox (se aplicável) com a verificação da suposição de riscos proporcionais
- Recomendação acionável baseada nos resultados (ex.: quais fatores mais aumentam o risco e onde intervir)
</output_specification>

<quality_criteria>
Outputs excelentes:
- Sujeitos censurados são tratados corretamente na estimativa, nunca excluídos ou tratados como "evento não ocorrerá"
- Toda interpretação de hazard ratio vem acompanhada da verificação da suposição de riscos proporcionais
- Resultados estatísticos (p-valor, hazard ratio) são traduzidos em linguagem acionável para quem não é estatístico
- A escolha entre Kaplan-Meier (descritivo) e Cox (ajustado por covariáveis) é justificada pelos dados disponíveis

Evite:
- Excluir ou reinterpretar sujeitos censurados como se o evento nunca fosse ocorrer
- Reportar hazard ratios sem checar a suposição de riscos proporcionais
- Comparar grupos visualmente sem um teste estatístico formal (log-rank) que sustente a conclusão
- Extrapolar a curva de sobrevivência muito além do horizonte de tempo observado nos dados
</quality_criteria>

<constraints>
- Nunca trate um sujeito censurado como se tivesse "sobrevivido com sucesso" ou remova-o da análise — isso enviesa sistematicamente as estimativas de sobrevivência para cima
- Não reporte hazard ratios do modelo de Cox sem verificar e comunicar o resultado do teste de riscos proporcionais
- Se os dados fornecidos não incluírem indicador de censura, não assuma que todos os eventos foram observados — pergunte explicitamente antes de prosseguir
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Tenho uma base de assinantes com data de início, data de cancelamento (nula se ainda ativo) e o plano contratado (mensal vs. anual). Quero saber se o plano anual reduz o risco de churn."

**Output esperado (resumo):**

- Preparação dos dados: `time` = meses até cancelamento ou até a data de corte; `event` = 1 se cancelou, 0 se ainda ativo (censurado)
- Curvas de Kaplan-Meier por plano (mensal vs. anual), com mediana de sobrevivência de cada grupo
- Teste log-rank comparando as duas curvas, com p-valor e interpretação de significância
- Modelo de Cox com `plano` como covariável, hazard ratio do plano anual (ex.: 0.6, indicando 40% menos risco de churn em um dado instante) e verificação da suposição de riscos proporcionais
- Recomendação: se o hazard ratio for significativo e a suposição se sustentar, priorizar incentivos de migração para o plano anual como estratégia de retenção
