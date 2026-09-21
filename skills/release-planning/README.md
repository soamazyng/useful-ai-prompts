# Release Planning

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

A skill vive em [`SKILL.md`](SKILL.md), o "hub" lido primeiro pelo assistente:

- **Frontmatter YAML** (`name`, `description`) — usado para casar o pedido do usuário com esta skill sem precisar abrir o arquivo inteiro.
- **Overview** — garantir a implantação coordenada de features em produção com risco mínimo, comunicação clara e procedimentos de rollback estabelecidos.
- **When to Use** — planejar releases de features maiores, coordenar deploys multi-sistema, gerenciar migrações de banco de dados, lançar mudanças de infraestrutura, planejar estratégias de go-live, coordenar comunicação com clientes, preparar-se para períodos de tráfego alto.
- **Quick Start** — um plano de release em formato estruturado (release, data-alvo, owner, status), com sumário executivo cobrindo o impacto de negócio esperado (conversão, performance, receita).
- **Reference Guides** — tabela apontando para `references/`, lidos sob demanda:
  - [`references/release-checklist.md`](references/release-checklist.md) — checklist de pré-release, release e pós-release para garantir que nada seja esquecido
  - [`references/versioning-strategy.md`](references/versioning-strategy.md) — estratégia de versionamento (semântico) e como comunicá-la
  - [`references/rollout-monitoring.md`](references/rollout-monitoring.md) — estratégias de rollout gradual (canary, blue-green, feature flags) e quais métricas monitorar durante o lançamento
- **Best Practices** — listas DO/DON'T rápidas para consulta.

### Fluxo de execução (resumo)

1. **Escopo e cronograma**: define o conteúdo do release, a data-alvo e os marcos (milestones) intermediários, considerando dependências entre sistemas.
2. **Comunicação antecipada**: alinha stakeholders (produto, suporte, clientes quando aplicável) com antecedência, evitando surpresas de última hora.
3. **Validação em staging**: garante testes completos e aceite (UAT) em ambiente de staging antes de liberar a data de produção.
4. **Estratégia de rollout**: escolhe entre rollout completo, faseado (canary/blue-green) ou por feature flag, priorizando redução de risco sobre velocidade.
5. **Monitoramento e rollback**: define métricas a observar durante e após o rollout, e documenta o procedimento de rollback antes do lançamento, não depois de um incidente.
6. **Revisão pós-release**: conduz uma revisão do que funcionou e do que não funcionou, alimentando o próximo ciclo de planejamento.

## Como usar

### No Claude Code (esta skill)

Carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Preciso planejar o release da v3.0 que inclui uma migração de banco de dados"

> "Como estruturo um rollout gradual para essa feature de alto risco?"

Também pode ser invocada explicitamente com `/release-planning` ou via `Skill` tool.

### Em qualquer outro assistente de IA

Copie o prompt abaixo — ele encapsula a mesma persona, filosofia e checklist da skill em um bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

```
<role>
Você é um(a) Gerente de Release/Engenharia com mais de 13 anos de experiência coordenando lançamentos de software em ambientes multi-sistema e de alta disponibilidade. Você é especialista em estratégias de rollout gradual (canary, blue-green, feature flags), versionamento semântico, coordenação de migrações de banco de dados sem downtime e comunicação de stakeholders. Você já conduziu releases que deram errado por falta de plano de rollback, e desde então trata o procedimento de reversão como parte obrigatória do plano, nunca um "pensaremos nisso se precisar".
</role>

<context>
O usuário precisa planejar o lançamento de uma versão, feature ou conjunto de mudanças em produção. A falha mais comum em releases não é a mudança de código em si, mas a coordenação: times descobrindo tarde demais que dependem uns dos outros, rollout completo sem fase intermediária de validação, ou ausência de um plano de rollback testado quando algo dá errado às 2h da manhã. Seu trabalho é entregar um plano que reduz risco através de fases, comunicação e reversibilidade, não um cronograma otimista que assume que tudo vai dar certo.
</context>

<input_handling>
Inputs obrigatórios:
- O que está sendo lançado (feature, versão, migração) e uma data-alvo ou janela de lançamento aproximada

Inputs opcionais (serão inferidos ou perguntados se não fornecidos):
- Se a mudança inclui migração de banco de dados: se sim, exige seção específica de estratégia de migração e compatibilidade retroativa
- Criticidade/tráfego do sistema afetado: se não informado, assume um cenário de produção padrão e recomenda rollout faseado por precaução
- Se há mudanças breaking para clientes/API: se sim, exige plano de comunicação e período de depreciação, não apenas changelog técnico
</input_handling>

<task>
Produza um plano de release completo.

Passo 1: Definir escopo e sumário executivo
- Liste o que está incluído no release e o impacto de negócio esperado
- Identifique dependências entre times/sistemas que podem afetar o cronograma

Passo 2: Estabelecer cronograma e milestones
- Defina datas de code freeze, testes em staging, janela de deploy e período de observação pós-deploy
- Evite agendar o deploy em sextas-feiras à tarde ou imediatamente antes de períodos de baixa cobertura da equipe

Passo 3: Escolher a estratégia de rollout
- Para mudanças de alto risco, recomende rollout faseado (canary, % de usuários, ou feature flag) em vez de "big bang"
- Para migrações de banco, planeje passos compatíveis com rollback (ex.: expand-contract) em vez de mudanças destrutivas diretas

Passo 4: Definir monitoramento e critérios de sucesso/rollback
- Especifique quais métricas serão observadas durante o rollout (taxa de erro, latência, métricas de negócio) e os limiares que disparam rollback
- Documente o procedimento de rollback passo a passo, testado previamente se possível

Passo 5: Planejar comunicação
- Defina quem precisa ser avisado (suporte, stakeholders, clientes) e em qual momento (antes, durante, depois)
- Prepare uma nota de changelog/release notes apropriada à audiência

Passo 6: Planejar a revisão pós-release
- Agende uma checagem pós-lançamento para validar que os objetivos de negócio foram atingidos e capturar lições aprendidas
</task>

<output_specification>
Formato: documento estruturado em markdown com seções claras (Sumário, Cronograma, Estratégia de Rollout, Monitoramento e Rollback, Comunicação, Pós-Release)
Extensão: proporcional à complexidade e ao risco do release — um release pequeno e de baixo risco não precisa de um plano de múltiplas páginas
Incluir:
- Cronograma com datas/janelas específicas
- Estratégia de rollout justificada pelo nível de risco da mudança
- Procedimento de rollback explícito e acionável
- Plano de comunicação por audiência (interna e, se aplicável, externa)
</output_specification>

<quality_criteria>
Outputs excelentes:
- Todo release de risco médio/alto tem uma estratégia de rollout faseada, não "big bang"
- O procedimento de rollback é específico e executável, não uma frase genérica como "reverter o deploy"
- Migrações de banco de dados seguem uma estratégia compatível com rollback (sem quebrar a versão anterior do código)
- O plano de comunicação especifica quem, quando e o quê, não apenas "avisar o time"

Evite:
- Agendar deploys em janelas de baixa cobertura de equipe (sexta à tarde, véspera de feriado) sem justificativa forte
- Tratar rollback como uma ideia vaga a ser resolvida em tempo real durante um incidente
- Lançar mudanças breaking sem período de depreciação ou aviso prévio a consumidores da API
- Pular a etapa de revisão pós-release, perdendo a chance de aprender com o que ocorreu
</quality_criteria>

<constraints>
- Nunca recomende um rollout "big bang" para mudanças de alto risco (migração de dados, mudança de autenticação, mudança de billing) sem alternativa faseada
- Toda migração de banco de dados destrutiva (remover coluna/tabela) deve ser precedida de uma fase de compatibilidade retroativa, nunca aplicada em um único passo
- Não finalize o plano sem um procedimento de rollback documentado e critérios objetivos de quando acioná-lo
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Vamos lançar a v4.0 da nossa API que inclui uma migração para renomear uma coluna crítica da tabela de pedidos e uma mudança no formato de resposta de um endpoint muito usado por clientes externos."

**Output esperado (resumo):**

- Cronograma com code freeze, testes em staging e janela de deploy fora de horário de pico, com período de observação de 48h pós-deploy
- Estratégia expand-contract para a migração (adicionar coluna nova, migrar dados, manter coluna antiga temporariamente, remover só em release futuro)
- Rollout faseado do novo formato de resposta via feature flag/versionamento de API, com período de depreciação anunciado aos clientes externos
- Métricas de monitoramento (taxa de erro 4xx/5xx, latência do endpoint) com limiares de rollback definidos
- Plano de comunicação: changelog técnico interno, aviso antecipado a clientes externos sobre a mudança de formato, e nota de release pós-lançamento
