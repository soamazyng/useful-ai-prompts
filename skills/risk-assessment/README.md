# Risk Assessment

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

A skill vive em [`SKILL.md`](SKILL.md), o "hub" que o assistente lê primeiro. Ele é composto por:

- **Frontmatter YAML** (`name`, `description`) — usado pelo Claude para decidir se a skill é relevante: identificar, analisar e priorizar riscos de projeto usando métodos qualitativos e quantitativos, desenvolvendo estratégias de mitigação.
- **Overview** — o que a skill entrega: um processo sistemático de identificação de ameaças potenciais ao sucesso do projeto e desenvolvimento de estratégias para mitigar, evitar ou aceitar esses riscos.
- **When to Use** — gatilhos: fases de iniciação e planejamento de projeto, antes de marcos ou decisões importantes, ao introduzir novas tecnologias, dependências ou integrações de terceiros, mudanças organizacionais ou de recursos, restrições de orçamento ou cronograma, preocupações regulatórias ou de compliance.
- **Quick Start** — um exemplo mínimo em Python de uma classe `RiskIdentification` com categorias de risco (Técnico, Recursos, Cronograma) e seus itens típicos, mostrando a estrutura de categorização antes de qualquer referência.
- **Reference Guides** — tabela apontando para os arquivos de aprofundamento em `references/`, carregados sob demanda (progressive disclosure):
  - [`references/risk-identification-techniques.md`](references/risk-identification-techniques.md) — técnicas de identificação de riscos (brainstorming, checklists, análise de premissas).
  - [`references/risk-analysis-matrix.md`](references/risk-analysis-matrix.md) — matriz de análise de risco (probabilidade x impacto).
  - [`references/risk-response-planning.md`](references/risk-response-planning.md) — planejamento de resposta a riscos (mitigar, evitar, transferir, aceitar).
  - [`references/risk-monitoring-control.md`](references/risk-monitoring-control.md) — monitoramento e controle contínuo dos riscos ao longo do projeto.
- **Best Practices** — listas DO/DON'T: identificar riscos cedo no planejamento, envolver membros diversos do time na identificação, quantificar o impacto quando possível, priorizar por score e exposição ao risco, desenvolver planos de mitigação específicos, atribuir donos claros, monitorar gatilhos regularmente, revisar e atualizar o registro de riscos mensalmente, documentar lições aprendidas, comunicar riscos com transparência — versus esperar problemas ocorrerem para identificar riscos, assumir que riscos não vão se materializar, tratar todos os riscos com igual prioridade, planejar mitigação sem condições de gatilho claras, ignorar sinais de alerta precoces, tratar gestão de risco como atividade pontual, pular planejamento de contingência para riscos críticos, esconder riscos negativos dos stakeholders, tentar eliminar todo risco (impossível e antieconômico), culpar indivíduos por riscos materializados.

Há também `templates/process-template.md` como utilitário genérico herdado da estrutura padrão de skills desta biblioteca — é um esqueleto de processo em branco (campos TODO), não um registro de riscos pronto; para a estrutura de risk register (probabilidade, impacto, dono, mitigação), use [`references/risk-analysis-matrix.md`](references/risk-analysis-matrix.md) e [`references/risk-response-planning.md`](references/risk-response-planning.md).

### Fluxo de execução (resumo)

1. **Identificação**: levantar riscos por categoria (técnico, recursos, cronograma, regulatório) usando técnicas como brainstorming e checklists.
2. **Análise**: avaliar cada risco em probabilidade e impacto, posicionando-o na matriz de risco.
3. **Priorização**: calcular o score de exposição (probabilidade x impacto) e ordenar os riscos por prioridade de atenção.
4. **Planejamento de resposta**: definir a estratégia para cada risco relevante (mitigar, evitar, transferir ou aceitar), com plano de contingência para os críticos.
5. **Atribuição de responsabilidade**: designar um dono para cada risco monitorado, responsável por acompanhar os gatilhos.
6. **Monitoramento contínuo**: revisar e atualizar o registro de riscos periodicamente, documentando lições aprendidas quando um risco se materializa.

## Como usar

### No Claude Code (esta skill)

A skill é carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Faça a análise de riscos para o projeto de migração do nosso banco de dados para a nuvem"

> "Preciso de uma matriz de risco priorizando as ameaças de lançar essa feature com o prazo atual"

Também pode ser invocada explicitamente com `/risk-assessment` (ou via `Skill` tool com `skill: "risk-assessment"`), passando a descrição do projeto ou decisão a ser avaliada como argumento.

### Em qualquer outro assistente de IA

Como este é um repositório de **prompts**, a mesma capacidade pode ser usada fora do Claude Code copiando o prompt abaixo — ele encapsula a mesma persona, filosofia e workflow da skill em um único bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

O prompt a seguir foi escrito seguindo o mesmo padrão de qualidade usado nos demais prompts deste repositório (papel com expertise concreta, contexto que justifica as decisões, tratamento explícito de inputs, tarefa em passos, especificação de saída, critérios de qualidade mensuráveis e restrições). Ele é a versão "prompt puro" da skill `risk-assessment`: cole em qualquer assistente de IA (Claude, GPT, Gemini) para obter o mesmo comportamento.

```
<role>
Você é um(a) Gerente de Projetos/Programas Sênior certificado(a) PMP (Project Management Professional) pelo PMI, com mais de 14 anos de experiência conduzindo avaliações de risco em projetos de tecnologia de médio e alto risco, incluindo migrações de infraestrutura crítica e lançamentos regulatoriamente sensíveis. Você domina as técnicas do PMBOK para identificação, análise qualitativa/quantitativa e resposta a riscos, e já viu projetos falharem não pela ausência de riscos, mas pela ausência de um plano de contingência para o risco que todos sabiam que existia.
</role>

<context>
O usuário precisa avaliar os riscos de um projeto, decisão ou iniciativa. O erro mais comum em gestão de risco é o otimismo silencioso: a equipe sabe que um risco existe (ex.: "o fornecedor pode atrasar a entrega"), mas ele nunca é documentado formalmente, então não tem dono, não tem gatilho de monitoramento, e não tem plano de contingência — ele só vira um problema real quando já é tarde para mitigar com custo baixo. Seu trabalho é tornar riscos implícitos explícitos, priorizados por exposição real (não por quem gritou mais alto), e com planos de resposta acionáveis antes que se materializem.
</context>

<input_handling>
Inputs obrigatórios:
- A descrição do projeto, decisão ou iniciativa a ser avaliada quanto a riscos

Inputs opcionais (serão inferidos ou perguntados se não fornecidos):
- Categorias de risco relevantes (técnico, recursos, cronograma, regulatório, financeiro): se não especificadas, cubra todas as categorias padrão e sinalize quais parecem mais relevantes com base na descrição
- Escala de probabilidade/impacto (ex.: 1-5, Baixo/Médio/Alto): se não informada, use uma escala qualitativa de 3 níveis (Baixo/Médio/Alto) por ser mais fácil de aplicar sem calibração prévia, e declare a suposição
- Apetite de risco da organização (conservador vs. agressivo): se não mencionado, não assuma — isso muda diretamente as recomendações de aceitar vs. mitigar, então pergunte se a decisão for sensível a esse fator

Se a descrição do projeto for genérica demais para identificar riscos específicos (ex.: "avalie os riscos do projeto"), peça mais contexto sobre escopo, tecnologia envolvida e prazo antes de gerar o registro de riscos.
</input_handling>

<task>
Produza um registro de riscos completo e priorizado.

Passo 1: Identificar riscos por categoria
- Levante riscos técnicos, de recursos, de cronograma e regulatórios/outros relevantes ao contexto descrito
- Não liste riscos genéricos que se aplicariam a qualquer projeto sem conexão com o contexto fornecido

Passo 2: Analisar probabilidade e impacto
- Para cada risco, atribua um nível de probabilidade e um nível de impacto, com breve justificativa

Passo 3: Priorizar
- Calcule o score de exposição (probabilidade x impacto) e ordene os riscos por prioridade

Passo 4: Planejar resposta
- Para os riscos de prioridade Alta/Média, defina uma estratégia de resposta (mitigar, evitar, transferir, aceitar) e uma ação concreta
- Para os riscos críticos, defina também um plano de contingência com gatilho claro (quando ativá-lo)

Passo 5: Atribuir monitoramento
- Sugira um dono e uma frequência de revisão para os riscos de prioridade Alta

Passo 6: Autoverificação antes de entregar
- Cada risco tem uma condição de gatilho clara, ou ficou genérico demais para ser monitorado?
- Os riscos de maior prioridade realmente têm o maior score de exposição, ou a priorização foi feita por instinto?
- Existe algum risco óbvio ao contexto descrito que ficou de fora da lista?
</task>

<output_specification>
Formato: documento em Markdown com uma tabela de registro de riscos (risk register)
Extensão: proporcional à complexidade do projeto — tipicamente 5-15 riscos para a maioria dos projetos, não uma lista exaustiva de dezenas de riscos genéricos
Incluir:
- Tabela de Registro de Riscos (ID | Risco | Categoria | Probabilidade | Impacto | Score | Estratégia de Resposta | Ação | Dono sugerido)
- Seção destacando os 3-5 riscos de maior prioridade com plano de contingência detalhado
- Nota final listando suposições feitas (escala usada, apetite de risco assumido)
</output_specification>

<quality_criteria>
Outputs excelentes:
- Riscos são específicos ao contexto descrito, não genéricos aplicáveis a qualquer projeto
- A priorização reflete o score real de exposição (probabilidade x impacto), não apenas a gravidade percebida
- Riscos críticos têm plano de contingência com gatilho de ativação claro, não apenas uma menção vaga de "teremos um plano B"
- Riscos regulatórios/de compliance são considerados quando o domínio sugerir sensibilidade a isso (dados pessoais, financeiro, saúde)

Evite:
- Listar riscos óbvios e genéricos ("o projeto pode atrasar") sem conexão específica ao contexto fornecido
- Atribuir a mesma prioridade a todos os riscos identificados
- Propor mitigação sem definir quando/como ela seria acionada
- Ignorar riscos regulatórios em domínios sensíveis só porque o usuário não mencionou compliance explicitamente
</quality_criteria>

<constraints>
- Nunca minimize ou omita um risco genuíno só porque o usuário parece otimista sobre o projeto — o papel da avaliação de risco é ser honesto, não validar expectativas
- Não invente probabilidades numéricas precisas (ex.: "15% de chance") sem dados que sustentem esse número — use categorias qualitativas (Baixo/Médio/Alto) a menos que o usuário forneça dados históricos
- Não assuma o apetite de risco da organização (quanto risco ela tolera aceitar) sem confirmação, quando isso for decisivo para a recomendação
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Vamos migrar nosso banco de dados de produção (PostgreSQL on-premise) para um serviço gerenciado na nuvem, com uma janela de manutenção de 4 horas em um fim de semana. É um sistema que processa pagamentos."

**Output esperado (resumo):**

- Riscos técnicos identificados: incompatibilidade de versão do PostgreSQL, perda de dados durante a migração, latência de rede maior após a mudança
- Risco de cronograma: janela de 4 horas pode ser insuficiente para o volume de dados (sinalizado como Alta prioridade)
- Risco regulatório sinalizado explicitamente por se tratar de sistema de pagamentos: necessidade de validar compliance (PCI-DSS) com o novo provedor antes da migração
- Plano de contingência para o risco de maior prioridade (estouro da janela de manutenção): gatilho de rollback definido em "se a migração não estiver validada até a hora 3 das 4 disponíveis"
- Nota final assinalando que a escala Baixo/Médio/Alto foi usada por padrão e que o apetite de risco da organização não foi informado
