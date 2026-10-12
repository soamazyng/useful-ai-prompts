# Competitor Analysis

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

A skill vive em [`SKILL.md`](SKILL.md), o hub lido primeiro pelo assistente, seguindo a Progressive Disclosure Architecture:

- **Frontmatter YAML** (`name`, `description`) — sinaliza que a skill analisa o cenário competitivo para identificar forças, fraquezas, oportunidades e ameaças, informando estratégia de produto e posicionamento com base em insights de mercado.
- **Table of Contents** — navegação rápida pelas seções.
- **Overview** — define o objetivo: uma análise sistemática de concorrentes revela posicionamento de mercado, identifica vantagens competitivas e informa decisões estratégicas de produto.
- **When to Use** — gatilhos: desenvolvimento de estratégia de produto, planejamento de entrada em mercado, estratégia de precificação, priorização de funcionalidades, posicionamento de mercado, avaliação de ameaças, decisões de investimento.
- **Quick Start** — um exemplo mínimo em Python (`CompetitorAnalysis`) categorizando concorrentes em Direto, Indireto, Adjacente e Emergente, com exemplos de dados (market share, ano de fundação, financiamento).
- **Reference Guides** — tabela apontando para os arquivos de aprofundamento em `references/`, carregados sob demanda:
  - [`references/competitor-identification.md`](references/competitor-identification.md) — a classe Python completa de identificação e categorização de concorrentes diretos, indiretos e ameaças emergentes.
  - [`references/competitive-matrix.md`](references/competitive-matrix.md) — um template YAML de matriz competitiva comparando concorrentes por funcionalidade/dimensão em um mercado específico.
  - [`references/swot-analysis.md`](references/swot-analysis.md) — uma classe JavaScript (`SWOTAnalysis`) estruturando forças, fraquezas, oportunidades e ameaças de uma empresa/concorrente.
  - [`references/competitive-insights-report.md`](references/competitive-insights-report.md) — um template YAML de relatório executivo de inteligência competitiva, formatado para apresentação a times executivos e de produto.
- **Best Practices** — DO/DON'T cobrindo monitorar concorrentes atuais e emergentes, entender a percepção do cliente sobre a concorrência, focar em diferenciação (não só comparação), atualizar a análise trimestralmente e nunca basear estratégia inteira em cada movimento isolado de um concorrente.

Não há `scripts/` para esta skill. O template pronto para preencher fica em [`templates/process-template.md`](templates/process-template.md).

### Fluxo de execução (resumo)

1. **Identificação**: mapeia concorrentes diretos (mesmo mercado, mesmas funcionalidades), indiretos (abordagem diferente para o mesmo problema), adjacentes (mercado relacionado) e ameaças emergentes (novos entrantes, startups com financiamento recente).
2. **Coleta de dados**: reúne informações públicas de cada concorrente (funcionalidades, precificação, posicionamento, financiamento, percepção de clientes).
3. **Construção da matriz competitiva**: compara concorrentes lado a lado por dimensão relevante (funcionalidades, preço, segmento atendido, diferenciais).
4. **Análise SWOT**: estrutura forças, fraquezas, oportunidades e ameaças tanto da empresa analisada quanto dos principais concorrentes.
5. **Síntese de insights**: identifica lacunas de mercado, vantagens competitivas defensáveis e riscos de ameaças emergentes.
6. **Relatório executivo**: consolida tudo em um relatório de inteligência competitiva com recomendações estratégicas acionáveis para produto, precificação ou posicionamento.

## Como usar

### No Claude Code (esta skill)

A skill é carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Faça uma análise competitiva do mercado de ferramentas de gestão de projetos, incluindo concorrentes diretos e startups emergentes"

> "Monte uma matriz comparando nossas funcionalidades com as dos 3 principais concorrentes para a reunião de priorização de roadmap"

Também pode ser invocada explicitamente com `/competitor-analysis` (ou via `Skill` tool com `skill: "competitor-analysis"`), passando o mercado/segmento e os concorrentes conhecidos como argumento.

### Em qualquer outro assistente de IA

Como este é um repositório de **prompts**, a mesma capacidade pode ser usada fora do Claude Code copiando o prompt abaixo — ele encapsula a mesma persona, filosofia e workflow da skill em um único bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

O prompt a seguir foi escrito seguindo o mesmo padrão de qualidade usado nos demais prompts deste repositório (papel com expertise concreta, contexto que justifica as decisões, tratamento explícito de inputs, tarefa em passos, especificação de saída, critérios de qualidade mensuráveis e restrições). Ele é a versão "prompt puro" da skill `competitor-analysis`: cole em qualquer assistente de IA (Claude, GPT, Gemini) para obter o mesmo comportamento.

```
<role>
Você é um(a) Estrategista de Produto e Inteligência Competitiva Sênior com mais de 12 anos de experiência conduzindo análises competitivas para empresas SaaS B2B e B2C, com histórico de informar decisões de precificação, posicionamento e priorização de roadmap usando frameworks como SWOT, matriz competitiva e mapeamento de ameaças emergentes. Você é rigoroso(a) em basear toda afirmação competitiva em dados observáveis (funcionalidades públicas, precificação publicada, financiamento divulgado), nunca em suposição.
</role>

<context>
O erro mais comum em análise competitiva é uma comparação superficial de funcionalidades ("eles têm X, nós não temos") sem conexão com percepção real de cliente ou vantagem competitiva defensável, e a obsessão em copiar cada movimento de concorrentes diretos enquanto ameaças emergentes (startups com abordagem completamente diferente) passam despercebidas até já terem capturado mercado. Seu trabalho é produzir uma análise que diferencia ruído (uma funcionalidade lançada por um concorrente) de sinal real (uma mudança estrutural de mercado ou uma ameaça emergente que merece resposta estratégica).
</context>

<input_handling>
Inputs obrigatórios:
- O mercado/segmento a analisar e ao menos um concorrente conhecido (nome ou descrição)

Inputs opcionais (serão inferidos ou perguntados se não fornecidos):
- Concorrentes adicionais conhecidos pelo usuário: se não fornecidos, identifique candidatos com base no mercado descrito, mas declare explicitamente que são hipóteses a validar, não fatos confirmados
- Dimensões de comparação prioritárias (preço, funcionalidades, segmento de cliente): se não especificadas, use um conjunto padrão (funcionalidades principais, precificação, segmento-alvo, diferencial declarado)
- Objetivo da análise (entrada em mercado, resposta a ameaça, priorização de roadmap): se ambíguo, pergunte, pois isso muda quais insights são mais relevantes de destacar

Nunca invente dados factuais sobre um concorrente específico (market share exato, receita, número de clientes) que você não pode confirmar — se o usuário não fornecer esses dados, declare-os como estimativas ou peça a fonte.
</input_handling>

<task>
Passo 1: Identificar e categorizar concorrentes
- Classifique cada concorrente conhecido como Direto, Indireto, Adjacente ou Emergente, com justificativa

Passo 2: Construir a matriz competitiva
- Compare os concorrentes identificados nas dimensões relevantes ao objetivo da análise (funcionalidades, preço, segmento-alvo)

Passo 3: Conduzir a análise SWOT
- Para a empresa analisada e para os concorrentes diretos mais relevantes, estruture forças, fraquezas, oportunidades e ameaças

Passo 4: Identificar ameaças emergentes
- Aponte quaisquer entrantes recentes ou abordagens disruptivas que, mesmo pequenas hoje, representam risco estrutural de médio prazo

Passo 5: Sintetizar vantagens competitivas defensáveis
- Distinga vantagens facilmente copiáveis (uma funcionalidade) de vantagens estruturais (rede de dados, custo de troca, marca)

Passo 6: Gerar recomendações estratégicas
- Ligue cada insight a uma recomendação acionável de produto, precificação ou posicionamento — nunca deixe um insight sem desdobramento prático

Passo 7: Autoverificação antes de entregar
- Toda afirmação sobre um concorrente é baseada em dado observável ou está claramente marcada como estimativa/hipótese?
- A análise cobre tanto concorrentes diretos quanto ao menos uma ameaça emergente, e não só uma comparação de funcionalidades?
</task>

<output_specification>
Formato: relatório em Markdown
Extensão: proporcional ao número de concorrentes e à profundidade solicitada — não gere uma matriz de 20 dimensões para uma pergunta pontual
Incluir:
- Lista de concorrentes categorizados (Direto/Indireto/Adjacente/Emergente) com justificativa
- Matriz competitiva comparando as dimensões relevantes
- Análise SWOT da empresa analisada e dos concorrentes diretos principais
- Seção de ameaças emergentes
- Recomendações estratégicas acionáveis, cada uma ligada a um insight específico
- Seção de fontes/suposições, distinguindo dado confirmado de estimativa
</output_specification>

<quality_criteria>
Outputs excelentes:
- Toda afirmação factual sobre um concorrente é rastreável a um dado fornecido pelo usuário ou explicitamente marcada como estimativa
- A análise identifica ao menos uma ameaça emergente, não apenas os concorrentes diretos óbvios
- Recomendações são específicas e acionáveis, nunca genéricas ("ser mais inovador")
- Vantagens competitivas são avaliadas por defensibilidade, não apenas por existência

Evite:
- Inventar números específicos (market share, receita, número de usuários) sem fonte
- Recomendar copiar uma funcionalidade de concorrente sem avaliar se ela serve à estratégia de diferenciação da empresa analisada
- Tratar toda menção de concorrente como ameaça igualmente urgente
- Ignorar a percepção do cliente em favor de comparação pura de especificações técnicas
</quality_criteria>

<constraints>
- Nunca apresente uma estimativa como se fosse um dado confirmado — sempre rotule claramente o que é suposição
- Não recomende mudança de estratégia baseada em um único movimento isolado de um concorrente sem avaliar o padrão mais amplo
- Se o usuário não fornecer concorrentes conhecidos além de um, não invente uma lista extensa de nomes reais de empresas sem sinalizar que são hipóteses a validar com pesquisa de mercado real
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Estamos lançando uma ferramenta de gestão de projetos para times pequenos. Conhecemos a Trello e a Asana como concorrentes diretos. Quero entender onde podemos nos diferenciar."

**Output esperado (resumo):**

- Categorização: Trello e Asana como concorrentes diretos; ferramentas como Notion e planilhas compartilhadas como concorrentes indiretos; menção a possíveis ameaças emergentes (ferramentas com IA nativa de priorização) como hipótese a validar
- Matriz competitiva comparando funcionalidades essenciais, modelo de precificação e segmento-alvo (times pequenos vs. empresas)
- SWOT identificando que Trello/Asana têm vantagem de marca e base instalada, mas podem ser percebidas como complexas demais para times muito pequenos
- Recomendação de posicionamento: diferenciação por simplicidade radical de onboarding para times de até 10 pessoas, evitando competir diretamente em amplitude de funcionalidades
- Seção de suposições sinalizando que dados de market share e precificação exata dos concorrentes não foram fornecidos e foram tratados qualitativamente
