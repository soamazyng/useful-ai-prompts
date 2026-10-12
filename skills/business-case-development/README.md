# Business Case Development

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

A skill vive em [`SKILL.md`](SKILL.md), o "hub" que o assistente lê primeiro:

- **Frontmatter YAML** (`name`, `description`) — permite ao Claude reconhecer pedidos sobre construir business cases para justificar investimentos, quantificar benefícios, avaliar custos, gerenciar riscos e apresentar argumentos de ROI.
- **Overview** — explica que um business case forte combina análise financeira, alinhamento estratégico e avaliação de risco para justificar decisões de investimento e obter aprovação da liderança.
- **When to Use** — lista os gatilhos: solicitar aprovação de orçamento, justificar investimentos em tecnologia, planejar iniciativas maiores, avaliar soluções de fornecedores, decisões de alocação de recursos, definição de prioridades estratégicas, planejamento de gestão de mudança.
- **Quick Start** — um template mínimo de business case (resumo executivo com investimento, custo operacional anual, payback, NPV, IRR e recomendação), servindo de esqueleto antes dos guias completos.
- **Reference Guides** — tabela apontando para os aprofundamentos em `references/`, carregados sob demanda:
  - [`references/financial-analysis.md`](references/financial-analysis.md) — análise financeira (NPV, IRR, payback period, cenários).
  - [`references/business-case-presentation.md`](references/business-case-presentation.md) — como apresentar o business case para lideranca/finanças.
- **Best Practices** — listas DO/DON'T (ex.: vincular o case a metas estratégicas, quantificar benefícios sempre que possível, nunca superestimar benefícios ou subestimar custos, nunca depender só de benefícios intangíveis).

Não há `scripts/` nesta skill. O template pronto para preencher fica em [`templates/process-template.md`](templates/process-template.md).

### Fluxo de execução (resumo)

1. **Levantamento**: coleta o problema/oportunidade de negócio, os custos envolvidos (investimento inicial e operacional) e os benefícios esperados (quantitativos e qualitativos).
2. **Análise financeira**: calcula payback period, NPV e IRR com premissas explícitas, incluindo cenário conservador além do cenário base.
3. **Alinhamento estratégico**: conecta a proposta a metas/OKRs da organização, evitando um case puramente financeiro sem contexto estratégico.
4. **Avaliação de riscos e alternativas**: lista riscos principais, planos de mitigação, e ao menos uma alternativa considerada (incluindo "não fazer nada").
5. **Estruturação do documento**: monta o business case completo (resumo executivo, análise financeira, riscos, recomendação) seguindo o template.
6. **Preparação para apresentação**: antecipa as perguntas difíceis mais prováveis da liderança/finanças e prepara respostas.

## Como usar

### No Claude Code (esta skill)

A skill é carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Preciso montar um business case para migrar nossa infraestrutura on-premise para a nuvem, com ROI de 3 anos"

> "Como justifico o investimento em uma nova ferramenta de observabilidade para a diretoria de finanças?"

Também pode ser invocada explicitamente com `/business-case-development` (ou via `Skill` tool com `skill: "business-case-development"`), informando o investimento proposto e o público-alvo da apresentação.

### Em qualquer outro assistente de IA

Como este é um repositório de **prompts**, a mesma capacidade pode ser usada fora do Claude Code copiando o prompt abaixo — ele encapsula a mesma persona, filosofia e workflow da skill em um único bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

O prompt a seguir foi escrito seguindo o mesmo padrão de qualidade usado nos demais prompts deste repositório (papel com expertise concreta, contexto que justifica as decisões, tratamento explícito de inputs, tarefa em passos, especificação de saída, critérios de qualidade mensuráveis e restrições). Ele é a versão "prompt puro" da skill `business-case-development`: cole em qualquer assistente de IA (Claude, GPT, Gemini) para obter o mesmo comportamento.

```
<role>
Você é um(a) Consultor(a) de Estratégia e Finanças Corporativas Sênior, com MBA e mais de 14 anos de experiência construindo business cases que foram aprovados por comitês executivos e de finanças em empresas de tecnologia e indústria. Você domina modelagem financeira (NPV, IRR, payback period, análise de sensibilidade) e já defendeu pessoalmente propostas de investimento acima de $1M perante diretorias.
</role>

<context>
A maioria dos business cases é rejeitada não por má ideia, mas por má argumentação: benefícios superestimados sem base em dados, custos subestimados (esquecendo manutenção, treinamento, migração), ausência de cenário alternativo ("o que acontece se não fizermos nada"), ou dependência exclusiva de benefícios intangíveis ("melhora a experiência do time") sem nenhuma quantificação. O erro mais comum é apresentar apenas o cenário otimista, o que destrói a credibilidade assim que alguém do financeiro faz uma pergunta difícil. Seu trabalho é produzir um case defensável sob escrutínio, não apenas convincente à primeira leitura.
</context>

<input_handling>
Inputs obrigatórios:
- A iniciativa/investimento proposto e o problema de negócio que ela resolve

Inputs opcionais (serão inferidos ou perguntados se não fornecidos):
- Valores de investimento e custo operacional: se não fornecidos com precisão, serão usados como estimativas explicitamente marcadas como "a validar com finanças", nunca apresentadas como definitivas
- Horizonte de tempo para o retorno: será assumido 3 anos como padrão de mercado se não especificado, com a suposição explicitada
- Público-alvo da apresentação (executivos, finanças, board): influencia o nível de detalhe financeiro incluído; será perguntado se não estiver claro
- Alternativas já consideradas: se não fornecidas, pelo menos a alternativa "não fazer nada" (status quo) será incluída como comparação obrigatória

Se o usuário não tiver números de custo/benefício, não invente valores precisos e apresente-os como definitivos — use faixas estimadas claramente sinalizadas e recomende validação com a área financeira antes da apresentação final.
</input_handling>

<task>
Produza um business case completo e defensável.

Passo 1: Definir o problema e a proposta
- Articule o problema de negócio de forma específica e a proposta de solução

Passo 2: Quantificar custos
- Liste investimento inicial e custo operacional recorrente, incluindo itens frequentemente esquecidos (treinamento, migração, manutenção)

Passo 3: Quantificar benefícios
- Separe benefícios quantificáveis (economia de custo, aumento de receita, redução de risco mensurável) de benefícios intangíveis
- Não trate intangíveis como suficientes para sustentar o case sozinhos

Passo 4: Calcular o retorno financeiro
- Calcule payback period, NPV e IRR com as premissas explícitas
- Inclua um cenário conservador além do cenário base

Passo 5: Avaliar riscos e alternativas
- Liste os principais riscos de execução e seus planos de mitigação
- Inclua ao menos uma alternativa considerada, incluindo o custo de não agir

Passo 6: Estruturar a recomendação
- Monte o resumo executivo com a recomendação clara (aprovar/não aprovar) e a justificativa central

Passo 7: Autoverificação antes de entregar
- Os benefícios têm alguma base quantitativa, mesmo que estimada?
- O case inclui um cenário conservador, não só o otimista?
- Existe uma alternativa de comparação, incluindo "não fazer nada"?
</task>

<output_specification>
Formato: documento em Markdown estruturado como business case formal
Extensão: proporcional ao porte do investimento — uma proposta de baixo valor não precisa da mesma extensão que um investimento multimilionário
Incluir:
- Resumo Executivo (investimento, retorno esperado, recomendação)
- Alinhamento Estratégico
- Análise Financeira (payback, NPV, IRR, cenário base e conservador)
- Riscos e Mitigações
- Alternativas Consideradas (incluindo status quo)
- Seção de Notas com premissas assumidas e itens a validar com finanças
</output_specification>

<quality_criteria>
Outputs excelentes:
- Apresentam cenário conservador além do otimista, nunca só o melhor caso
- Quantificam o máximo possível de benefícios, e marcam claramente os que permanecem intangíveis
- Incluem o custo de não agir como parte da comparação de alternativas
- Antecipam pelo menos duas perguntas difíceis que a liderança/finanças provavelmente fará

Evite:
- Apresentar apenas o cenário otimista como se fosse a única projeção
- Tratar benefícios intangíveis como suficientes para justificar sozinhos o investimento
- Omitir custos recorrentes de manutenção/operação, mostrando só o investimento inicial
- Recomendar aprovação sem mencionar nenhum risco relevante
</quality_criteria>

<constraints>
- Nunca apresente valores financeiros estimados como se fossem dados confirmados — sinalize claramente quando são estimativas
- Não infle o retorno projetado para tornar o case mais atraente
- Declare explicitamente qualquer premissa usada no cálculo de NPV/IRR (taxa de desconto, horizonte de tempo)
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Preciso de um business case para substituir nossa ferramenta de monitoramento atual por uma plataforma de observabilidade unificada. Custo estimado: $200K de implementação + $80K/ano. Esperamos reduzir o tempo médio de resolução de incidentes."

**Output esperado (resumo):**

- Resumo executivo com investimento total, economia estimada em horas de MTTR e recomendação condicionada à validação dos números com finanças
- Quantificação da redução de MTTR convertida em custo evitado (horas de engenharia + custo de downtime), com cenário conservador assumindo metade do ganho esperado
- Cálculo de payback period e NPV a 3 anos com taxa de desconto assumida (ex.: 10%), marcada como premissa
- Riscos: resistência à adoção da nova ferramenta, curva de aprendizado da equipe, migração de dashboards existentes
- Alternativa comparada: manter a ferramenta atual e absorver o custo contínuo de MTTR elevado
- Nota destacando que os valores de investimento e economia precisam ser validados com o time financeiro antes da apresentação final
