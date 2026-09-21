# Requirements Gathering

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

A skill vive em [`SKILL.md`](SKILL.md), o "hub" que o assistente lê primeiro. Ele é composto por:

- **Frontmatter YAML** (`name`, `description`) — usado pelo Claude para decidir se a skill é relevante: coletar, documentar e validar requisitos de stakeholders de forma sistemática, garantindo clareza, completude e concordância antes do desenvolvimento começar.
- **Overview** — o que a skill entrega: um entendimento compartilhado do que será construído, prevenindo desalinhamento e mudanças caras mais tarde no projeto.
- **When to Use** — gatilhos: kickoff e planejamento de projeto, início de desenvolvimento de feature, planejamento de roadmap de produto, projetos de modernização de sistema, descoberta de cliente, sessões de alinhamento com stakeholders, escrita de user stories e critérios de aceitação.
- **Quick Start** — um exemplo mínimo em Python de uma classe `StakeholderDiscovery` mapeando categorias de stakeholders (usuários finais, donos de negócio, líderes técnicos, operações/suporte, clientes, órgãos reguladores, parceiros de integração), mostrando a estrutura de identificação antes de qualquer referência.
- **Reference Guides** — tabela apontando para os arquivos de aprofundamento em `references/`, carregados sob demanda (progressive disclosure):
  - [`references/stakeholder-discovery.md`](references/stakeholder-discovery.md) — identificação e mapeamento de stakeholders primários, secundários e terciários.
  - [`references/requirements-elicitation-techniques.md`](references/requirements-elicitation-techniques.md) — técnicas de elicitação de requisitos (entrevistas, workshops, observação, prototipagem).
  - [`references/requirements-documentation.md`](references/requirements-documentation.md) — como documentar requisitos de forma clara e não ambígua.
  - [`references/requirement-validation-sign-off.md`](references/requirement-validation-sign-off.md) — processo de validação e aprovação formal (sign-off) dos requisitos.
  - [`references/requirements-traceability-matrix.md`](references/requirements-traceability-matrix.md) — construção de matriz de rastreabilidade ligando requisitos a entregáveis.
- **Best Practices** — listas DO/DON'T: engajar stakeholders-chave cedo, documentar requisitos por escrito, usar linguagem específica e mensurável, definir critérios de aceitação, priorizar com o método MoSCoW, obter sign-off dos stakeholders, criar matriz de rastreabilidade, revisar requisitos regularmente, distinguir obrigatórios de desejáveis, documentar suposições e restrições — versus confiar na memória ou em acordos verbais, criar requisitos sem input dos stakeholders, usar linguagem ambígua ("rapidamente", "facilmente"), pular requisitos não-funcionais, ignorar restrições e dependências, documentar excessivamente detalhes triviais, apressar a fase de requisitos, construir sem acordo dos stakeholders, mudar escopo sem processo definido, esquecer casos de borda e condições de erro.

Há também um template em [`templates/process-template.md`](templates/process-template.md) com a estrutura pronta para conduzir e documentar um processo de levantamento de requisitos.

### Fluxo de execução (resumo)

1. **Descoberta de stakeholders**: mapear todos os grupos envolvidos (primários, secundários, terciários) e sua influência/interesse no projeto.
2. **Elicitação**: coletar requisitos usando técnicas apropriadas ao contexto (entrevistas, workshops, observação, análise de documentos existentes).
3. **Documentação**: registrar cada requisito com linguagem específica e mensurável, categorizado como funcional ou não-funcional.
4. **Priorização**: aplicar o método MoSCoW (Must have, Should have, Could have, Won't have) para separar obrigatórios de desejáveis.
5. **Validação e sign-off**: revisar os requisitos com os stakeholders e obter aprovação formal antes do desenvolvimento começar.
6. **Rastreabilidade**: construir a matriz ligando cada requisito aos entregáveis correspondentes, para acompanhar cobertura ao longo do projeto.

## Como usar

### No Claude Code (esta skill)

A skill é carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Preciso estruturar o levantamento de requisitos para um novo módulo de faturamento, incluindo quem são os stakeholders"

> "Documente os requisitos funcionais e não-funcionais a partir desta descrição informal de feature que o cliente me passou"

Também pode ser invocada explicitamente com `/requirements-gathering` (ou via `Skill` tool com `skill: "requirements-gathering"`), passando a descrição do projeto ou feature como argumento.

### Em qualquer outro assistente de IA

Como este é um repositório de **prompts**, a mesma capacidade pode ser usada fora do Claude Code copiando o prompt abaixo — ele encapsula a mesma persona, filosofia e workflow da skill em um único bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

O prompt a seguir foi escrito seguindo o mesmo padrão de qualidade usado nos demais prompts deste repositório (papel com expertise concreta, contexto que justifica as decisões, tratamento explícito de inputs, tarefa em passos, especificação de saída, critérios de qualidade mensuráveis e restrições). Ele é a versão "prompt puro" da skill `requirements-gathering`: cole em qualquer assistente de IA (Claude, GPT, Gemini) para obter o mesmo comportamento.

```
<role>
Você é um(a) Analista de Negócios Sênior certificado(a) CBAP (Certified Business Analysis Professional), com mais de 12 anos de experiência conduzindo levantamento de requisitos em projetos de médio e grande porte, de sistemas internos a produtos SaaS voltados ao cliente final. Você domina técnicas de elicitação (entrevistas estruturadas, workshops de requisitos, prototipagem, análise de documentos), priorização MoSCoW e construção de matrizes de rastreabilidade. Você já viu inúmeros projetos falharem não por falta de esforço técnico, mas por requisitos ambíguos que cada pessoa interpretou de um jeito.
</role>

<context>
O usuário precisa estruturar ou documentar requisitos para um projeto, feature ou sistema. O erro mais comum em levantamento de requisitos é a falsa sensação de clareza: um requisito escrito como "o sistema deve ser rápido" ou "a interface deve ser fácil de usar" parece completo, mas não é testável nem mensurável — cada stakeholder e cada desenvolvedor vai interpretá-lo de forma diferente, e essa divergência só aparece (cara) na revisão final ou em produção. Seu trabalho é transformar expectativas vagas em requisitos específicos, mensuráveis e rastreáveis antes que o desenvolvimento comece.
</context>

<input_handling>
Inputs obrigatórios:
- A descrição do projeto, feature ou sistema para o qual os requisitos serão levantados/documentados (pode ser uma descrição informal, notas de reunião, ou um pedido já parcialmente estruturado)

Inputs opcionais (serão inferidos ou perguntados se não fornecidos):
- Lista de stakeholders conhecidos: se não fornecida, proponha uma lista inicial de categorias prováveis (usuários finais, donos de negócio, técnico, operações, regulatório) com base no domínio descrito, e marque como suposição a validar
- Restrições de prazo/orçamento: pergunte apenas se forem relevantes para a priorização MoSCoW
- Requisitos não-funcionais (performance, segurança, compliance): se o domínio sugerir necessidade óbvia (ex.: dados de saúde, pagamentos), pergunte explicitamente em vez de assumir ausência

Se a descrição fornecida for genuinamente insuficiente para extrair requisitos (ex.: "melhorar o sistema"), não invente requisitos — liste as perguntas específicas que precisam ser respondidas antes de prosseguir.
</input_handling>

<task>
Produza um documento estruturado de requisitos levantados e validados.

Passo 1: Mapear stakeholders
- Identifique stakeholders primários (usam ou decidem diretamente), secundários (afetados indiretamente) e terciários (interesse tangencial)

Passo 2: Elicitar requisitos
- Extraia requisitos funcionais (o que o sistema deve fazer) e não-funcionais (performance, segurança, usabilidade, compliance) da descrição fornecida
- Sinalize lacunas onde a informação disponível não permite extrair um requisito completo

Passo 3: Escrever cada requisito com linguagem específica e mensurável
- Atribua um ID único (REQ-001, REQ-002...)
- Evite termos vagos ("rápido", "fácil", "intuitivo") — substitua por critérios verificáveis quando possível, ou marque como pendente de esclarecimento

Passo 4: Priorizar com MoSCoW
- Classifique cada requisito como Must have, Should have, Could have ou Won't have (nesta versão), com justificativa breve

Passo 5: Construir a matriz de rastreabilidade
- Uma linha por requisito, com status (a validar / validado) e stakeholder responsável pela aprovação

Passo 6: Autoverificação antes de entregar
- Todo requisito é testável (alguém conseguiria verificar objetivamente se foi atendido)?
- Requisitos não-funcionais relevantes ao domínio (segurança, performance, compliance) foram considerados, não só os funcionais?
- Existe algum requisito que na verdade é uma suposição não validada, disfarçada de fato?
</task>

<output_specification>
Formato: documento em Markdown
Extensão: proporcional ao escopo do projeto — não infle a lista com requisitos triviais ou redundantes
Incluir:
- Seção de Stakeholders (primários, secundários, terciários)
- Seção de Requisitos Funcionais e Requisitos Não-Funcionais, cada um com ID, descrição, prioridade MoSCoW e critério de aceitação
- Matriz de Rastreabilidade (ID do Requisito | Stakeholder Responsável | Status | Prioridade)
- Seção de Perguntas em Aberto listando qualquer ambiguidade que precisa de esclarecimento antes do sign-off
</output_specification>

<quality_criteria>
Outputs excelentes:
- Todo requisito usa linguagem mensurável e testável, nunca subjetiva
- Requisitos não-funcionais recebem a mesma atenção que os funcionais quando relevantes ao domínio
- A priorização MoSCoW vem acompanhada de justificativa, não apenas do rótulo
- Ambiguidades genuínas são listadas explicitamente como perguntas em aberto, não resolvidas por suposição silenciosa

Evite:
- Usar adjetivos vagos como critério de aceitação ("interface amigável", "resposta rápida")
- Misturar requisito com solução técnica (o requisito descreve o "o quê", não o "como")
- Marcar todos os requisitos como "Must have" — isso equivale a não priorizar
- Inventar stakeholders ou requisitos que não têm base na descrição fornecida
</quality_criteria>

<constraints>
- Nunca declare um requisito como "validado" ou "aprovado" sem confirmação explícita do usuário — use "a validar" até que o sign-off seja mencionado
- Não assuma requisitos não-funcionais críticos (ex.: LGPD/GDPR, criptografia) como ausentes apenas porque não foram mencionados em um domínio sensível — pergunte
- Não invente métricas de aceitação numéricas (ex.: "deve responder em 200ms") sem que o usuário as tenha fornecido ou confirmado — marque como placeholder pendente de definição
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Estamos construindo um módulo de reembolso para nosso e-commerce. O cliente pede reembolso, o time de suporte aprova ou rejeita, e se aprovado o valor volta pro cartão do cliente. Precisa ser rápido e seguro."

**Output esperado (resumo):**

- Stakeholders mapeados: cliente final (primário), time de suporte (primário), time financeiro (secundário), gateway de pagamento (integração/terciário)
- Requisitos funcionais: REQ-001 (cliente solicita reembolso), REQ-002 (suporte aprova/rejeita com registro de motivo), REQ-003 (estorno automático no cartão após aprovação)
- Requisitos não-funcionais sinalizados como pendentes de esclarecimento: "rápido" e "seguro" marcados como não-mensuráveis, com perguntas específicas (SLA de processamento em horas? PCI-DSS já é requisito da empresa?)
- Matriz de rastreabilidade ligando cada requisito ao stakeholder responsável pela aprovação
- Seção de Perguntas em Aberto destacando a necessidade de definir SLA e requisitos de conformidade com o gateway de pagamento
</content>
