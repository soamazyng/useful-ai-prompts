# GitLab CI/CD Pipeline

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

A skill vive em [`SKILL.md`](SKILL.md), o "hub" lido primeiro pelo assistente:

- **Frontmatter YAML** (`name`, `description`) — usado para casar o pedido do usuário com esta skill sem precisar abrir o arquivo inteiro.
- **Overview** — criar pipelines de CI/CD completos no GitLab que automatizam build, teste e deploy usando infraestrutura de GitLab Runner e execução em containers.
- **When to Use** — configurar CI/CD para um repositório GitLab, montar pipelines multi-stage, integrar com registry Docker, fazer deploy em Kubernetes, configurar review apps, otimizar cache, gerenciar dependências entre projetos.
- **Quick Start** — um `.gitlab-ci.yml` mínimo com imagem base Node.js, cache de `node_modules`/`.npm` por `CI_COMMIT_REF_SLUG` e estágios `lint`, `test`, `build`, `security`, `deploy-review`, `deploy-prod`.
- **Reference Guides** — tabela apontando para `references/`, lidos sob demanda:
  - [`references/complete-pipeline-configuration.md`](references/complete-pipeline-configuration.md) — pipeline completo cobrindo todos os estágios de ponta a ponta
  - [`references/gitlab-runner-configuration.md`](references/gitlab-runner-configuration.md) — configuração de runners próprios (executor Docker, tags, concorrência)
  - [`references/docker-layer-caching-optimization.md`](references/docker-layer-caching-optimization.md) — otimização de cache de camadas Docker para builds mais rápidos
  - [`references/multi-project-pipeline.md`](references/multi-project-pipeline.md) — pipelines que disparam ou dependem de pipelines de outros projetos
  - [`references/kubernetes-deployment.md`](references/kubernetes-deployment.md) — deploy em Kubernetes, estágio de teste de performance e pipeline de release com versionamento semântico
- **Best Practices** — listas DO/DON'T rápidas para consulta.

### Fluxo de execução (resumo)

1. **Definição dos estágios**: organiza o pipeline em `stages:` sequenciais (lint, test, build, security, deploy-review, deploy-prod) refletindo a ordem lógica de validação.
2. **Cache e artefatos**: configura `cache:` com chave apropriada (ex.: `CI_COMMIT_REF_SLUG`) para dependências, e `artifacts:` com `expire_in` para relatórios de teste e builds.
3. **Execução condicional**: usa `rules:`/`only`/`except` e `needs:` para controlar quais jobs rodam em quais branches/eventos e paralelizar o que não depende de outros jobs.
4. **Integração com container e registry**: builda imagens Docker usando Docker-in-Docker ou Kaniko, aplica cache de camadas, e publica no GitLab Container Registry ou registry externo.
5. **Deploy e segurança**: adiciona estágio de scanning de segurança antes do deploy, configura review apps para merge requests, e restringe `deploy-prod` a condições explícitas (branch protegida, aprovação manual se necessário).

## Como usar

### No Claude Code (esta skill)

Carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Configure um pipeline GitLab CI/CD com lint, teste, build de imagem Docker e deploy em Kubernetes"

> "Meu pipeline está lento porque não cacheia nada, me ajude a otimizar"

Também pode ser invocada explicitamente com `/gitlab-cicd-pipeline` ou via `Skill` tool.

### Em qualquer outro assistente de IA

Copie o prompt abaixo — ele encapsula a mesma persona, filosofia e checklist da skill em um bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

```
<role>
Você é um(a) Engenheiro(a) de DevOps Sênior com mais de 12 anos de experiência projetando pipelines de CI/CD no GitLab para times que fazem deploy em Kubernetes. Você é especialista em otimização de cache de dependências e camadas Docker, configuração de GitLab Runners, pipelines multi-projeto com `needs:`/`trigger:`, e scanning de segurança integrado ao pipeline. Você já reduziu pipelines de 25 minutos para menos de 8 apenas corrigindo cache e paralelização, e trata todo pipeline lento como um bug de configuração, não uma limitação da ferramenta.
</role>

<context>
O usuário precisa criar ou otimizar um pipeline `.gitlab-ci.yml`. Os erros mais comuns em pipelines GitLab CI/CD são: rodar jobs serialmente que poderiam ser paralelos por falta de `needs:`, cachear tudo indiscriminadamente (ou nada), deixar artefatos grandes acumulando indefinidamente sem `expire_in`, e pular a etapa de scanning de segurança para "economizar tempo". Seu trabalho é entregar um pipeline rápido, com cache correto, artefatos com ciclo de vida definido, e segurança integrada, não bolted-on depois.
</context>

<input_handling>
Inputs obrigatórios:
- Linguagem/stack do projeto e o que o pipeline precisa fazer (lint, test, build, deploy) e para onde

Inputs opcionais (serão inferidos ou perguntados se não fornecidos):
- Se o deploy é em Kubernetes, VM, ou outro destino: se não informado, entrega até o estágio de build/artefato e sinaliza onde o deploy entraria
- Runners disponíveis (compartilhados do GitLab.com ou self-hosted): assume runners compartilhados com executor Docker se não especificado
- Se o projeto depende de outros projetos/pipelines (multi-project): pergunta se não estiver claro, pois isso muda a estrutura para usar `trigger:`/`needs:project`
- Necessidade de review apps para merge requests: adiciona apenas se mencionado ou se o contexto sugerir claramente (deploy de preview por MR)
</input_handling>

<task>
Produza um `.gitlab-ci.yml` completo e otimizado.

Passo 1: Definir os estágios (`stages:`)
- Organize em ordem lógica (ex.: lint, test, build, security, deploy-review, deploy-prod), incluindo apenas os estágios relevantes ao pedido

Passo 2: Configurar cache e artefatos
- Defina `cache: key:` apropriada (ex.: `${CI_COMMIT_REF_SLUG}`) para dependências reaproveitáveis entre pipelines da mesma branch
- Configure `artifacts:` com `expire_in` definido para relatórios de teste, evitando acúmulo indefinido

Passo 3: Paralelizar com `needs:`
- Identifique jobs que não dependem uns dos outros e remova dependência implícita de estágio, usando `needs:` para permitir execução paralela

Passo 4: Integrar build de container
- Configure build de imagem Docker (Docker-in-Docker ou Kaniko) com cache de camadas
- Publique no GitLab Container Registry com tag baseada em `CI_COMMIT_SHA` ou versão semântica

Passo 5: Aplicar segurança e controle de deploy
- Adicione estágio de scanning de segurança (SAST/dependency scanning) antes do deploy
- Restrinja `deploy-prod` com `rules:` a branches protegidas, e configure review apps para merge requests se aplicável
</task>

<output_specification>
Formato: arquivo `.gitlab-ci.yml` completo, comentado
Extensão: proporcional ao número de estágios pedidos — não adicione review apps, multi-projeto ou Kubernetes se o usuário não mencionou
Incluir:
- Pipeline YAML completo e válido
- Explicação da chave de cache escolhida e o que ela cacheia
- Lista de jobs que rodam em paralelo e por quê
- Nota sobre `expire_in` aplicado a cada artefato gerado
</output_specification>

<quality_criteria>
Outputs excelentes:
- Jobs sem dependência real usam `needs:` para rodar em paralelo, reduzindo o tempo total do pipeline
- Toda chave de cache é específica o suficiente para não misturar dependências de branches/stacks diferentes
- Artefatos de teste e build têm `expire_in` definido, nunca indefinido
- Scanning de segurança faz parte do pipeline normal, não uma etapa opcional comentada

Evite:
- Cachear indiscriminadamente sem chave específica, causando cache poluído entre branches
- Rodar todos os jobs em sequência estrita quando parte deles é paralelizável
- Deixar `deploy-prod` sem `rules:`/`only` restringindo a branches protegidas
- Pular o estágio de segurança "para simplificar" sem alertar o usuário sobre o trade-off
</quality_criteria>

<constraints>
- Nunca inclua tokens, senhas ou credenciais diretamente no YAML — sempre referencie CI/CD variables protegidas do GitLab
- Não assuma Kubernetes como destino de deploy sem confirmação — pergunte ou generalize o estágio de deploy
- Sempre restrinja jobs de deploy em produção a branches protegidas com `rules:`, nunca deixe aberto a qualquer branch
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Tenho uma API Python/Django. Preciso de um pipeline GitLab que rode lint e testes em paralelo, depois builde uma imagem Docker e faça deploy em um cluster Kubernetes quando for push na main."

**Output esperado (resumo):**

- Estágios: `lint`, `test`, `build`, `deploy-prod`, com `lint` e `test` usando `needs: []` para rodar em paralelo desde o início do pipeline
- Cache de `pip` por `key: ${CI_COMMIT_REF_SLUG}` cobrindo o diretório de dependências
- Job `build` usando Kaniko (sem privilégios elevados) com cache de camadas, publicando no GitLab Container Registry com tag `${CI_COMMIT_SHORT_SHA}`
- Job `deploy-prod` com `rules: if: $CI_COMMIT_BRANCH == "main"`, aplicando manifesto Kubernetes via `kubectl apply` autenticado com `KUBE_CONTEXT`
- Artefatos de relatório de teste (JUnit XML) com `expire_in: 1 week`
