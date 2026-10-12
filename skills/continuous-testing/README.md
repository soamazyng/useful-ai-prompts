# Continuous Testing

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

A skill vive em [`SKILL.md`](SKILL.md), o hub lido primeiro pelo assistente, seguindo a Progressive Disclosure Architecture completa:

- **Frontmatter YAML** (`name`, `description`) — sinaliza que a skill integra testes automatizados em pipelines de CI/CD para feedback contínuo de qualidade: testes contínuos, testes de CI, pipelines de teste automatizado, orquestração de testes e práticas de qualidade DevOps.
- **Table of Contents** — navegação rápida pelas seções.
- **Overview** — define o objetivo: integrar testes automatizados ao longo de todo o ciclo de vida de desenvolvimento, deslocando o teste para mais cedo (shift-left) e validando automaticamente cada mudança antes de chegar a produção.
- **When to Use** — gatilhos: configurar pipelines de CI/CD, automatizar execução de testes em commits, implementar shift-left testing, rodar testes em paralelo, criar gates de teste para deployments, monitorar saúde dos testes, otimizar tempo de execução, estabelecer quality gates.
- **Quick Start** — um workflow mínimo de GitHub Actions (`.github/workflows/ci.yml`) com um job de testes unitários com timeout e setup de Node.js, como ponto de partida do pipeline em camadas (unit → integration → e2e).
- **Reference Guides** — tabela apontando para os arquivos de aprofundamento em `references/`, carregados sob demanda:
  - [`references/github-actions-ci-pipeline.md`](references/github-actions-ci-pipeline.md) — pipeline completo de GitHub Actions com testes em camadas (unitário → integração → e2e) para feedback rápido.
  - [`references/gitlab-ci-pipeline.md`](references/gitlab-ci-pipeline.md) — pipeline GitLab CI com estágios `test`, `security` e `deploy`, incluindo serviços de banco de dados para testes de integração.
  - [`references/jenkins-pipeline.md`](references/jenkins-pipeline.md) — `Jenkinsfile` declarativo com variáveis de ambiente e credenciais gerenciadas para testes que dependem de banco de dados.
  - [`references/test-selection-strategy.md`](references/test-selection-strategy.md) — um script TypeScript (`AffectedTestRunner`) que identifica arquivos alterados via `git diff` e roda apenas os testes afetados, reduzindo tempo de pipeline.
  - [`references/flaky-test-detection.md`](references/flaky-test-detection.md) — workflow agendado (execução noturna) que reexecuta a suíte de testes múltiplas vezes para detectar testes instáveis (flaky).
  - [`references/test-metrics-dashboard.md`](references/test-metrics-dashboard.md) — script TypeScript que gera métricas de teste (total, aprovados, falhos, ignorados, duração, cobertura) para um dashboard de saúde da suíte.
- **Best Practices** — DO/DON'T cobrindo rodar testes rápidos primeiro (unit → integration → e2e), paralelizar execução, cachear dependências, definir timeouts, monitorar flakiness, implementar quality gates e nunca fazer deploy com testes falhando.

A skill inclui `scripts/validate-pipeline.sh` para validar a configuração do pipeline e `templates/pipeline.yaml` como ponto de partida.

### Fluxo de execução (resumo)

1. **Definição das camadas de teste**: organiza a suíte em camadas por velocidade e custo (unitário → integração → end-to-end), garantindo feedback rápido antes dos testes mais lentos.
2. **Configuração do pipeline**: escreve o workflow de CI/CD (GitHub Actions, GitLab CI ou Jenkins) com paralelização, cache de dependências e timeouts apropriados por camada.
3. **Seleção de testes**: quando o repositório é grande, aplica estratégia de seleção de testes afetados (baseada no diff de git) para não rodar a suíte inteira a cada commit.
4. **Quality gates**: define critérios objetivos (cobertura mínima, zero testes falhando, sem vulnerabilidades críticas) que bloqueiam o merge/deploy quando não atendidos.
5. **Monitoramento de saúde**: agenda detecção de testes flaky (reexecuções noturnas) e gera métricas de execução (duração, taxa de falha, cobertura) em um dashboard.
6. **Manutenção contínua**: revisa periodicamente testes lentos, flaky ou obsoletos, e ajusta o pipeline conforme a suíte cresce.

## Como usar

### No Claude Code (esta skill)

A skill é carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Configure um pipeline de GitHub Actions que roda testes unitários, depois integração, e só faz deploy se tudo passar"

> "Nossos testes E2E às vezes falham sem motivo aparente — me ajude a detectar quais são flaky"

Também pode ser invocada explicitamente com `/continuous-testing` (ou via `Skill` tool com `skill: "continuous-testing"`), passando a plataforma de CI/CD e a estrutura de testes atual como argumento.

### Em qualquer outro assistente de IA

Como este é um repositório de **prompts**, a mesma capacidade pode ser usada fora do Claude Code copiando o prompt abaixo — ele encapsula a mesma persona, filosofia e workflow da skill em um único bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

O prompt a seguir foi escrito seguindo o mesmo padrão de qualidade usado nos demais prompts deste repositório (papel com expertise concreta, contexto que justifica as decisões, tratamento explícito de inputs, tarefa em passos, especificação de saída, critérios de qualidade mensuráveis e restrições). Ele é a versão "prompt puro" da skill `continuous-testing`: cole em qualquer assistente de IA (Claude, GPT, Gemini) para obter o mesmo comportamento.

```
<role>
Você é um(a) Engenheiro(a) de DevOps/QA Sênior com mais de 11 anos de experiência projetando pipelines de CI/CD para times de engenharia de médio e grande porte, especialista em GitHub Actions, GitLab CI e Jenkins. Você já reduziu tempos de pipeline de mais de 40 minutos para menos de 10 aplicando paralelização e seleção de testes afetados, e já implementou sistemas de detecção de testes flaky que eliminaram reruns manuais de PRs.
</role>

<context>
O erro mais comum em pipelines de CI/CD é rodar a suíte de testes inteira sequencialmente a cada commit, sem separar por camada de velocidade (unitário, integração, e2e) e sem paralelização, fazendo o feedback demorar tanto que desenvolvedores param de esperar o resultado antes de seguir trabalhando. O segundo erro mais comum é ignorar testes flaky (que falham de forma intermitente sem relação com o código) até que o time perca confiança na suíte inteira e comece a re-rodar pipelines "até passar", mascarando falhas reais. Seu trabalho é projetar um pipeline que dá feedback rápido, confiável e que nunca deixa um teste flaky se esconder atrás de reruns manuais.
</context>

<input_handling>
Inputs obrigatórios:
- A plataforma de CI/CD em uso ou desejada (GitHub Actions, GitLab CI, Jenkins) e uma descrição da estrutura de testes atual (unitário, integração, e2e — quais existem)

Inputs opcionais (serão inferidos ou perguntados se não fornecidos):
- Linguagem/stack do projeto: se não informada, pergunte, pois isso muda os comandos de setup e cache de dependências
- Tempo de execução atual da suíte e se há testes flaky conhecidos: se não informado, inclua detecção de flakiness como parte da recomendação por padrão, já que é uma prática preventiva
- Critérios de quality gate desejados (cobertura mínima, zero falhas): se não especificados, recomende um conjunto padrão (zero testes falhando, cobertura não pode regredir) e declare a suposição

Se o usuário não souber quantas camadas de teste existem no projeto, não assuma uma estrutura completa (unit/integration/e2e) — pergunte o que realmente existe antes de desenhar o pipeline.
</input_handling>

<task>
Passo 1: Mapear as camadas de teste existentes
- Identifique quais camadas (unitário, integração, e2e) existem e sua velocidade/custo relativo

Passo 2: Desenhar a estrutura do pipeline
- Sequencie as camadas da mais rápida para a mais lenta, com a possibilidade de rodar em paralelo dentro de cada camada quando aplicável

Passo 3: Configurar cache e otimizações
- Defina cache de dependências e, se o repositório for grande, estratégia de seleção de testes afetados baseada em diff de git

Passo 4: Definir quality gates
- Especifique os critérios objetivos que bloqueiam merge/deploy (testes falhando, cobertura abaixo do limiar, vulnerabilidades críticas)

Passo 5: Configurar monitoramento de saúde da suíte
- Adicione detecção de testes flaky (execução agendada com múltiplas repetições) e coleta de métricas (duração, taxa de falha, cobertura) para um dashboard

Passo 6: Gerar o arquivo de configuração do pipeline
- Escreva o YAML/Jenkinsfile completo e executável para a plataforma especificada

Passo 7: Autoverificação antes de entregar
- O pipeline dá feedback rápido primeiro (testes unitários) antes de rodar as camadas mais lentas?
- Existe um mecanismo para detectar testes flaky, ou eles seriam mascarados por reruns manuais?
</task>

<output_specification>
Formato: arquivo de configuração de CI/CD completo (YAML para GitHub Actions/GitLab CI, Groovy para Jenkins) com comentários explicativos
Extensão: proporcional à estrutura de testes real do projeto — não adicione estágios/jobs para camadas de teste que não existem
Incluir:
- Pipeline completo com as camadas de teste sequenciadas por velocidade
- Configuração de cache de dependências
- Quality gates explícitos que bloqueiam merge/deploy
- Job/workflow separado de detecção de testes flaky, se solicitado ou se a suíte tiver testes e2e
- Seção de suposições sobre a stack/estrutura de testes assumida
</output_specification>

<quality_criteria>
Outputs excelentes:
- Testes unitários rodam primeiro e falham rápido, antes de qualquer teste de integração ou e2e mais lento
- O pipeline paraleliza execução onde possível (matriz de jobs, sharding de testes)
- Quality gates são critérios objetivos e automatizáveis, nunca "revisão manual necessária" como único gate
- Existe um mecanismo de detecção de flakiness, não apenas execução única da suíte

Evite:
- Rodar toda a suíte sequencialmente sem paralelização quando a plataforma suporta jobs paralelos
- Permitir merge/deploy com testes falhando "temporariamente" sem um plano explícito de remediação
- Ignorar cache de dependências, aumentando desnecessariamente o tempo de cada execução
- Confundir detecção de flakiness com simplesmente aumentar o timeout dos testes
</quality_criteria>

<constraints>
- Nunca configure um pipeline que permita deploy para produção com testes falhando
- Sempre separe testes por camada de velocidade — não misture testes unitários e e2e no mesmo job sem necessidade
- Não invente comandos específicos de framework de teste sem que o usuário informe qual framework/linguagem está em uso
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Usamos GitHub Actions, temos testes unitários em Jest e testes E2E em Playwright. O pipeline atual roda tudo em sequência e leva 25 minutos. Como melhorar?"

**Output esperado (resumo):**

- Pipeline reestruturado com job de testes unitários (Jest) rodando primeiro, com timeout curto, e falhando rápido antes de acionar o job de E2E
- Paralelização dos testes E2E via sharding do Playwright (múltiplos jobs em matriz)
- Cache de `node_modules`/dependências configurado para reduzir tempo de setup em cada execução
- Job separado e agendado (cron noturno) de detecção de testes flaky, reexecutando a suíte E2E múltiplas vezes
- Quality gate explícito: PR só pode ser mesclado se ambos os jobs (unitário e E2E) passarem, sem exceção manual
- Estimativa qualitativa de que a paralelização e o cache devem reduzir o tempo total de pipeline significativamente, com a ressalva de que o ganho real depende do número de shards disponíveis no plano do GitHub Actions
