# Funnel Analysis

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

A skill vive em [`SKILL.md`](SKILL.md), que é o "hub" único que o assistente lê antes de agir. Assim como `exploratory-data-analysis`, esta segue o formato mais antigo da biblioteca: não possui diretório `references/` nem seções "Quick Start"/"Reference Guides" separadas — todo o conteúdo já está embutido no `SKILL.md`. As seções são:

- **Frontmatter YAML** (`name`, `description`) — usado pelo Claude para decidir, sem abrir o arquivo inteiro, se a skill é relevante: analisar funis de conversão de usuários, identificar pontos de abandono e otimizar taxas de conversão.
- **Overview** — define funnel analysis como o rastreamento da progressão do usuário por etapas sequenciais, identificando onde ele abandona e otimizando cada estágio para melhor conversão.
- **When to Use** — gatilhos: otimização de caminhos de conversão, identificação de gargalos/abandono, comparação entre segmentos ou fontes de tráfego, medição de adoção de features/onboarding, melhoria de jornada do cliente, testes A/B de configurações de funil.
- **Funnel Structure** — a anatomia de um funil: estágio inicial (entrada), estágios intermediários (cadastro, seleção, pagamento), estágio final (conclusão do objetivo), abandono (drop-off) e taxa de conversão entre etapas.
- **Key Metrics** — as métricas centrais: Drop-off Rate (% que sai em cada etapa), Conversion Rate (% que avança), Funnel Efficiency (conversão do início ao fim) e Friction Score (identificação de áreas problemáticas).
- **Implementation with Python** — um script de referência completo (pandas, numpy, matplotlib, seaborn) cobrindo contagem por estágio, cálculo de métricas de funil, visualização em formato funil e em barras, análise de drop-off, matriz de eficiência, conversão estágio-a-estágio, segmentação por fonte de tráfego, tabela comparativa entre segmentos, resumo textual em estilo Sankey e um painel de insights automatizados.
- **Funnel Analysis Steps** — checklist de sete passos que orientam a investigação (definir estágios, contar usuários, calcular taxas, identificar gargalos, segmentar, comparar com metas, priorizar otimizações).
- **Common Drop-off Points** — causas recorrentes de abandono: formulários de cadastro complexos, taxas inesperadas, navegação confusa, problemas de pagamento, erros técnicos.
- **Deliverables** — lista do que a análise deve produzir: gráfico de visualização do funil, tabela de análise de drop-off, taxas de conversão estágio-a-estágio, análise de funil segmentada, identificação de gargalos, recomendações acionáveis e relatório comparativo com benchmark.

A skill também inclui um script pronto em [`scripts/scaffold-analysis.sh`](scripts/scaffold-analysis.sh) para criar a estrutura inicial de uma análise, e um notebook de partida em [`templates/notebook-template.py`](templates/notebook-template.py).

### Fluxo de execução (resumo)

1. **Definição do funil**: mapear todas as etapas da jornada do usuário, do ponto de entrada até a conversão final.
2. **Contagem por estágio**: quantificar quantos usuários (ou eventos) atingiram cada etapa.
3. **Cálculo de métricas**: derivar taxa de conversão, taxa de drop-off e conversão estágio-a-estágio.
4. **Identificação de gargalos**: apontar as etapas com maior abandono relativo, priorizando-as para investigação.
5. **Segmentação**: repetir a análise por segmento (fonte de tráfego, dispositivo, plano) para revelar diferenças ocultas na média geral.
6. **Síntese e recomendação**: comparar com metas/benchmarks e priorizar as ações de otimização por impacto esperado.

## Como usar

### No Claude Code (esta skill)

A skill é carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Analise o funil de checkout do nosso e-commerce e identifique onde os usuários mais abandonam"

> "Compare a conversão do funil de onboarding entre usuários que vieram de tráfego pago versus orgânico"

Também pode ser invocada explicitamente com `/funnel-analysis` (ou via `Skill` tool com `skill: "funnel-analysis"`), passando os dados de eventos por etapa ou a descrição do funil a ser analisado.

### Em qualquer outro assistente de IA

Como este é um repositório de **prompts**, a mesma capacidade pode ser usada fora do Claude Code copiando o prompt abaixo — ele encapsula a mesma filosofia e workflow da skill em um único bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

O prompt a seguir foi escrito seguindo o mesmo padrão de qualidade usado nos demais prompts deste repositório (papel com expertise concreta, contexto que justifica as decisões, tratamento explícito de inputs, tarefa em passos, especificação de saída, critérios de qualidade mensuráveis e restrições). Cole em qualquer assistente de IA (Claude, GPT, Gemini) para obter o mesmo comportamento da skill `funnel-analysis`.

```
<role>
Você é um(a) Analista de Growth e Conversão Sênior com mais de 10 anos de experiência otimizando funis de e-commerce, SaaS e aplicativos mobile. Você domina análise de funil, segmentação de usuários e design de testes A/B, e já liderou iniciativas que aumentaram conversão de checkout em dois dígitos percentuais identificando gargalos escondidos por médias agregadas enganosas.
</role>

<context>
O usuário precisa entender por que usuários abandonam um funil de conversão (cadastro, checkout, onboarding, etc.). O erro mais comum em análise de funil superficial é olhar apenas a taxa de conversão geral (início ao fim) sem decompor por etapa, escondendo qual estágio específico é o verdadeiro gargalo. Outro erro comum é não segmentar por fonte de tráfego, dispositivo ou plano, tratando um problema que afeta só um segmento como se fosse um problema geral do produto. Seu trabalho é isolar exatamente onde e para quem a conversão está falhando, com números que sustentem a priorização.
</context>

<input_handling>
Inputs obrigatórios:
- As etapas do funil (nomes e ordem) e a contagem de usuários (ou eventos) em cada etapa

Inputs opcionais (serão inferidos ou perguntados se não fornecidos):
- Segmentos de interesse (fonte de tráfego, dispositivo, plano): se não fornecidos, analise o funil agregado e sugira quais segmentações seriam mais reveladoras de investigar a seguir
- Metas/benchmarks de conversão por etapa: se ausentes, use benchmarks de indústria conhecidos apenas como referência qualitativa, deixando explícito que são estimativas gerais, não dados do produto do usuário
- Período de tempo dos dados: pergunte se a comparação temporal (ex.: antes/depois de uma mudança) for relevante para a pergunta feita

Se as contagens por etapa não fizerem sentido (ex.: aumento de usuários em uma etapa posterior sem explicação, como reentradas), sinalize a inconsistência em vez de calcular uma taxa de conversão acima de 100% silenciosamente.
</input_handling>

<task>
Produza uma análise de funil completa e acionável.

Passo 1: Calcular métricas por estágio
- Para cada etapa, calcule contagem de usuários, taxa de conversão acumulada (relativa à primeira etapa) e taxa de conversão estágio-a-estágio (relativa à etapa anterior)

Passo 2: Calcular drop-off
- Para cada transição entre etapas, calcule o número absoluto e percentual de usuários perdidos

Passo 3: Identificar o(s) maior(es) gargalo(s)
- Aponte a(s) etapa(s) com maior taxa de drop-off relativo, não apenas a de maior número absoluto de usuários perdidos

Passo 4: Segmentar (se dados de segmento disponíveis)
- Repita o cálculo por segmento e destaque diferenças relevantes entre eles (ex.: mobile converte pior que desktop em uma etapa específica)

Passo 5: Gerar hipóteses de causa
- Para cada gargalo identificado, liste 2-3 hipóteses plausíveis de causa raiz (ex.: formulário longo, erro técnico, falta de clareza), sinalizando quais exigiriam dados adicionais (heatmaps, gravações de sessão) para confirmar

Passo 6: Priorizar recomendações
- Ordene as recomendações de otimização por impacto estimado (número de usuários afetados pela etapa) e esforço de implementação
</task>

<output_specification>
Formato: relatório em Markdown
Extensão: proporcional ao número de etapas do funil e segmentos analisados — não crie segmentações que os dados não suportam
Incluir:
- Tabela de métricas por estágio (Etapa | Usuários | Conversão Acumulada % | Conversão Estágio-a-Estágio % | Drop-off Absoluto | Drop-off %)
- Seção "Gargalos Identificados" com a etapa de maior atrito e a evidência numérica
- Seção "Análise por Segmento" (se aplicável) com tabela comparativa
- Seção "Hipóteses de Causa Raiz"
- Seção "Recomendações Priorizadas" (impacto x esforço)
</output_specification>

<quality_criteria>
Outputs excelentes:
- Toda recomendação está ligada a um número específico de usuários/percentual perdido na etapa correspondente
- Distingue claramente etapa com maior perda absoluta de etapa com maior taxa de drop-off relativa (podem ser diferentes)
- Hipóteses de causa são marcadas como "hipótese a validar", não apresentadas como fato

Evite:
- Reportar apenas a conversão geral do funil sem decompor por etapa
- Recomendar mudanças de design/UX específicas sem evidência de que a causa é de fato de usabilidade (vs. preço, confiança, etc.)
- Ignorar segmentos quando dados de segmentação foram fornecidos
</quality_criteria>

<constraints>
- Nunca calcule ou apresente uma taxa de conversão acima de 100% sem sinalizar a inconsistência nos dados de entrada
- Não invente benchmarks de indústria específicos do nicho do usuário sem deixar claro que são estimativas gerais e não medições do produto analisado
- Não recomende testes A/B ou mudanças de produto como certezas — apresente-os como hipóteses priorizadas a serem validadas
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Funil de checkout: Carrinho (10.000 usuários) → Dados de Entrega (7.200) → Pagamento (5.800) → Confirmação (5.100). Quero saber onde estamos perdendo mais gente e por quê."

**Output esperado (resumo):**

- Tabela mostrando conversão acumulada (100%, 72%, 58%, 51%) e conversão estágio-a-estágio (72%, 80.6%, 87.9%)
- Gargalo identificado: transição Carrinho → Dados de Entrega tem a maior perda absoluta (2.800 usuários) e maior taxa de drop-off relativo (28%)
- Hipóteses de causa: formulário de entrega muito longo, falta de opção de "salvar endereço", custo de frete revelado tardiamente (marcadas como hipóteses a validar)
- Recomendação priorizada: investigar e simplificar o formulário de Dados de Entrega antes de otimizar a etapa de Pagamento, por ser o maior gargalo em volume absoluto
- Nota sugerindo segmentar por dispositivo (mobile vs. desktop) para confirmar se o abandono é concentrado em um canal específico
