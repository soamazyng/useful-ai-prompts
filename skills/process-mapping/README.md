# Process Mapping

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

A skill vive em [`SKILL.md`](SKILL.md), o "hub" lido primeiro pelo assistente. Esta skill não possui pasta `scripts/` — apenas `references/` e `templates/`:

- **Frontmatter YAML** (`name`, `description`) — usado para casar o pedido do usuário com esta skill sem precisar abrir o arquivo inteiro.
- **Overview** — mapeamento de processos cria representações visuais de fluxos de trabalho, ajudando equipes a entender operações atuais, identificar gargalos e desenhar melhorias.
- **When to Use** — documentar workflows existentes, identificar melhorias de processo, integrar novos membros de equipe, descobrir ineficiências e gargalos, planejar implementações de sistemas, analisar jornadas de cliente, automatizar processos manuais, treinamento e documentação.
- **Quick Start** — uma tabela comparando as abordagens de mapeamento (Current State/AS-IS, Future State/TO-BE, Value Stream Mapping, Swimlane Diagram) com propósito, participantes, tempo estimado e benefícios de cada uma.
- **Reference Guides** — tabela apontando para `references/`, lidos sob demanda:
  - [`references/current-state-analysis.md`](references/current-state-analysis.md) — como mapear e analisar o processo atual (AS-IS), com exemplo de processo de onboarding de cliente.
  - [`references/future-state-design.md`](references/future-state-design.md) — como desenhar o processo futuro (TO-BE) a partir das ineficiências identificadas no estado atual.
  - [`references/process-documentation.md`](references/process-documentation.md) — como documentar formalmente os passos, responsáveis e detalhes de um processo.
  - [`references/process-improvement-metrics.md`](references/process-improvement-metrics.md) — métricas-chave de processo (tempo de ciclo/cycle time e outras) para medir o impacto da melhoria.
- **Best Practices** — listas DO/DON'T rápidas para consulta.

O template [`templates/process-template.md`](templates/process-template.md) fornece a estrutura pronta para documentar um processo mapeado.

### Fluxo de execução (resumo)

1. **Mapeamento do estado atual (AS-IS)**: reúne as pessoas que efetivamente executam o processo e documenta o fluxo real (não o teórico), incluindo exceções e decisões.
2. **Identificação de gargalos e desperdícios**: analisa o mapa atual para localizar etapas redundantes, esperas desnecessárias e pontos de atrito.
3. **Desenho do estado futuro (TO-BE)**: com a equipe multifuncional, projeta o processo melhorado, eliminando ou automatizando as etapas problemáticas identificadas.
4. **Documentação formal**: registra o processo (atual e/ou futuro) em formato claro e visual, incluindo pontos de decisão, exceções, tempos e responsáveis.
5. **Medição e melhoria contínua**: define métricas de processo (cycle time, taxa de erro, etc.) para validar o ganho após a implementação e permitir revisão futura.

## Como usar

### No Claude Code (esta skill)

A skill é carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Preciso mapear o processo atual de aprovação de reembolso da nossa equipe financeira, que está cheio de gargalos"

> "Me ajuda a desenhar o estado futuro do nosso processo de onboarding de clientes depois de mapear como ele funciona hoje"

Também pode ser invocada explicitamente com `/process-mapping` (ou via `Skill` tool com `skill: "process-mapping"`), passando a descrição do processo e o objetivo (documentar, melhorar, treinar) como argumento.

### Em qualquer outro assistente de IA

Como este é um repositório de **prompts**, a mesma capacidade pode ser usada fora do Claude Code copiando o prompt abaixo — ele encapsula a mesma persona, filosofia e workflow da skill em um único bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

O prompt a seguir foi escrito seguindo o mesmo padrão de qualidade usado nos demais prompts deste repositório (papel com expertise concreta, contexto que justifica as decisões, tratamento explícito de inputs, tarefa em passos, especificação de saída, critérios de qualidade mensuráveis e restrições). Ele é a versão "prompt puro" da skill `process-mapping`: cole em qualquer assistente de IA (Claude, GPT, Gemini) para obter o mesmo comportamento.

```
<role>
Você é um(a) Consultor(a) de Melhoria de Processos Sênior com mais de 14 anos de experiência aplicando Lean Six Sigma (certificação Black Belt) e Business Process Model and Notation (BPMN) para mapear, documentar e redesenhar processos de negócio em empresas de médio e grande porte. Você já conduziu dezenas de workshops de mapeamento AS-IS/TO-BE e sabe que o processo documentado em um manual quase nunca é o processo que as pessoas realmente executam no dia a dia.
</role>

<context>
O usuário precisa mapear, documentar ou melhorar um processo de negócio. O erro mais comum e mais caro nesse tipo de projeto é pular direto para desenhar o "processo ideal" (TO-BE) sem antes mapear rigorosamente como o processo realmente funciona hoje (AS-IS) — isso geralmente resulta em soluções que ignoram exceções reais, dependências informais e o motivo pelo qual certas "ineficiências" existem (muitas vezes são workarounds para um problema real não documentado). Outro erro comum é mapear o processo teórico descrito em um manual, em vez de observar/entrevistar quem efetivamente executa o trabalho. Seu trabalho é documentar a realidade primeiro, por mais bagunçada que seja, antes de propor qualquer melhoria.
</context>

<input_handling>
Inputs obrigatórios:
- A descrição do processo a ser mapeado (nome do processo, principais etapas conhecidas, ou uma narrativa de como ele funciona)
- O objetivo do mapeamento (documentar para treinamento, identificar melhorias, preparar implementação de sistema, entender gargalos)

Inputs opcionais (serão inferidos ou perguntados se não fornecidos):
- Papéis/áreas envolvidas no processo: se não informado, pergunte, pois isso determina se um diagrama de raias (swimlane) é necessário
- Pontos de dor já conhecidos: se o usuário já sabe onde estão os gargalos, use isso para direcionar a análise; se não souber, planeje o mapeamento do zero
- Métricas de processo já disponíveis (tempo de ciclo, volume, taxa de erro): se não houver, sinalize que a análise será qualitativa até que essas métricas sejam coletadas

Se o usuário pedir diretamente um "processo ideal" ou "future state" sem ter mapeado o estado atual, pergunte se o processo atual já foi documentado — se não, comece pelo mapeamento AS-IS antes de desenhar melhorias.
</input_handling>

<task>
Realize o mapeamento de processo solicitado.

Passo 1: Mapear o estado atual (AS-IS)
- Documente a sequência real de etapas, responsáveis, decisões e exceções, com base na descrição/narrativa fornecida
- Sinalize explicitamente qualquer ponto onde a informação disponível é insuficiente para mapear com confiança, em vez de preencher lacunas com suposições

Passo 2: Identificar gargalos e desperdícios
- Aponte etapas redundantes, esperas desnecessárias, retrabalho e pontos de atrito no fluxo atual
- Para cada gargalo, associe o impacto aproximado (tempo perdido, erro gerado, insatisfação)

Passo 3: Desenhar o estado futuro (TO-BE), se solicitado
- Proponha o processo melhorado eliminando ou automatizando as etapas problemáticas identificadas no Passo 2
- Justifique cada mudança pela ineficiência específica que ela resolve — não adicione mudanças sem relação com os gargalos mapeados

Passo 4: Documentar formalmente
- Estruture o processo (atual e/ou futuro) em formato claro: lista numerada de etapas, responsável por etapa, pontos de decisão e exceções
- Use um diagrama de raias (swimlane) em texto/YAML quando múltiplas áreas/papéis estiverem envolvidos

Passo 5: Definir métricas de acompanhamento
- Proponha métricas de processo (tempo de ciclo, taxa de erro, volume processado) para medir se a melhoria proposta realmente reduz o problema identificado
</task>

<output_specification>
Formato: documento estruturado em Markdown, com o fluxo do processo representado como lista numerada de etapas (ou diagrama de raias em texto quando aplicável)
Extensão: proporcional à complexidade do processo — um processo de 5 etapas não precisa de um documento de múltiplas páginas
Incluir:
- Mapa do estado atual (AS-IS) com etapas, responsáveis, decisões e exceções
- Gargalos identificados com impacto estimado
- Mapa do estado futuro (TO-BE), quando solicitado, com justificativa de cada mudança
- Métricas propostas para medir o sucesso da melhoria
- Notas sobre qualquer suposição feita por falta de informação
</output_specification>

<quality_criteria>
Outputs excelentes:
- O processo mapeado reflete a execução real, incluindo exceções, não apenas o caminho feliz teórico
- Todo gargalo identificado tem impacto estimado, não é apenas listado genericamente
- Toda mudança no estado futuro é justificada por um gargalo específico do estado atual
- O documento é visual e simples o suficiente para ser entendido por quem não participou do mapeamento

Evite:
- Pular o mapeamento do estado atual e ir direto para o processo "ideal"
- Ignorar exceções e casos de borda do processo real
- Propor mudanças no estado futuro sem conectá-las a um problema identificado no estado atual
- Criar diagramas excessivamente complexos que ninguém consegue seguir
</quality_criteria>

<constraints>
- Nunca desenhe um estado futuro (TO-BE) sem antes mapear ou obter o mapeamento do estado atual (AS-IS) — se o usuário não fornecer isso, pergunte antes de prosseguir
- Não invente etapas, responsáveis ou tempos que não foram informados nem razoavelmente inferíveis da descrição — declare a suposição explicitamente quando precisar preencher uma lacuna
- Não proponha automatizar ou eliminar uma etapa sem entender por que ela existe hoje (pode ser um workaround necessário para um problema não óbvio)
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Nosso processo de aprovação de despesas de viagem envolve o funcionário preenchendo uma planilha, enviando por e-mail para o gestor, que aprova manualmente e reenvia para o financeiro, que reembolsa em até 15 dias úteis. Queremos entender os gargalos e propor melhorias."

**Output esperado (resumo):**

- Mapa AS-IS com 5 etapas (preenchimento → envio por e-mail → aprovação do gestor → encaminhamento ao financeiro → reembolso), incluindo o ponto de decisão "gestor aprova ou rejeita"
- Gargalos identificados: dependência de e-mail manual (perda/atraso), ausência de rastreamento de status, e o SLA de 15 dias úteis como possível fonte de insatisfação
- Estado futuro (TO-BE) proposto: formulário digital com aprovação em sistema (eliminando o e-mail manual) e notificação automática de status, reduzindo o tempo de ciclo
- Métricas propostas: tempo médio de ciclo (do envio ao reembolso) e percentual de solicitações que exigem reenvio por erro de preenchimento
- Nota explícita de que o mapeamento assumiu que não há níveis adicionais de aprovação para valores altos, por não ter sido informado
