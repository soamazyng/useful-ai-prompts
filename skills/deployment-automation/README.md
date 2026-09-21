# Deployment Automation

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

A skill vive em [`SKILL.md`](SKILL.md), o "hub" que o assistente lê primeiro. Ele é composto por:

- **Frontmatter YAML** (`name`, `description`) — usado pelo Claude para decidir se a skill é relevante: automação de deploys com Helm, Terraform e ArgoCD, blue-green, canary e estratégias de rollback.
- **Overview** — resume o propósito: estabelecer pipelines de deploy automatizados que movem aplicações entre ambientes (dev, staging, produção) com o mínimo de intervenção manual e risco.
- **When to Use** — os gatilhos: deploy contínuo para Kubernetes, deploy de Infraestrutura como Código, promoção multi-ambiente, estratégias blue-green, gerenciamento de canary release, provisionamento de infraestrutura e procedimentos automatizados de rollback.
- **Quick Start** — um exemplo mínimo de `Chart.yaml`/`values.yaml` do Helm com réplicas, imagem, service e limites de recursos, para o assistente entender o formato antes de aprofundar.
- **Reference Guides** — tabela apontando para os arquivos de aprofundamento em `references/`, carregados sob demanda (progressive disclosure):
  - [`references/helm-deployment-chart.md`](references/helm-deployment-chart.md) — estrutura de um Helm chart de deploy (Chart.yaml, values.yaml, templates).
  - [`references/github-actions-deployment-workflow.md`](references/github-actions-deployment-workflow.md) — workflow de GitHub Actions para automatizar o pipeline de deploy.
  - [`references/argocd-deployment.md`](references/argocd-deployment.md) — configuração de deploy GitOps com ArgoCD.
  - [`references/blue-green-deployment.md`](references/blue-green-deployment.md) — script de deploy blue-green: subir a versão "green", rodar smoke tests e só então trocar o tráfego.
- **Best Practices** — listas DO/DON'T rápidas de consulta (ex.: usar Infraestrutura como Código e testar em staging antes; nunca fazer deploy direto em produção ou pular a validação pré-deploy).

Um script utilitário está disponível em [`scripts/validate-config.sh`](scripts/validate-config.sh) para validar a configuração de deploy, e um template inicial em [`templates/config-starter.yaml`](templates/config-starter.yaml).

### Fluxo de execução (resumo)

1. Identifica a plataforma alvo (Kubernetes, cloud provider) e a ferramenta de automação em uso ou desejada (Helm, Terraform, ArgoCD, GitHub Actions).
2. Define a estratégia de deploy adequada ao risco da mudança (rolling, blue-green ou canary).
3. Especifica os artefatos de automação (chart, workflow, manifesto) necessários para o pipeline.
4. Inclui validações pré-deploy (testes em staging, health checks) e critérios de promoção entre ambientes.
5. Define o procedimento de rollback automatizado caso os health checks falhem após o deploy.
6. Documenta o procedimento completo para que outra pessoa da equipe consiga executá-lo sem ambiguidade.

## Como usar

### No Claude Code (esta skill)

A skill é carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Preciso configurar um deploy blue-green no Kubernetes usando Helm para minha aplicação"

> "Quero automatizar a promoção de staging para produção via ArgoCD com rollback automático se o health check falhar"

Também pode ser invocada explicitamente com `/deployment-automation` (ou via `Skill` tool com `skill: "deployment-automation"`), passando a stack de infraestrutura e a estratégia desejada como argumento.

### Em qualquer outro assistente de IA

Como este é um repositório de **prompts**, a mesma capacidade pode ser usada fora do Claude Code copiando o prompt abaixo — ele encapsula a mesma persona, filosofia e workflow da skill em um único bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

O prompt a seguir foi escrito seguindo o mesmo padrão de qualidade usado nos demais prompts deste repositório. Ele é a versão "prompt puro" da skill `deployment-automation`: cole em qualquer assistente de IA (Claude, GPT, Gemini) para obter o mesmo comportamento.

```
<role>
Você é um(a) Engenheiro(a) de Plataforma/SRE Sênior com mais de 13 anos de experiência projetando pipelines de deploy automatizados para aplicações em Kubernetes, com certificação CKA (Certified Kubernetes Administrator) e profundo domínio de Helm, Terraform, ArgoCD e GitOps. Você já implementou estratégias blue-green e canary release para sistemas de missão crítica com SLA de 99,95%, sempre com rollback automatizado como rede de segurança.
</role>

<context>
O usuário precisa automatizar o deploy de uma aplicação entre ambientes (dev, staging, produção). O erro mais comum em automação de deploy é tratar o pipeline como "só rodar o script e torcer": sem testes em staging antes de produção, sem health checks pós-deploy e sem um caminho de rollback automatizado, um deploy ruim vira um incidente em produção antes que alguém perceba. Seu trabalho é projetar um pipeline que detecta falhas automaticamente e reverte sozinho, sem depender de alguém estar de plantão observando o dashboard.
</context>

<input_handling>
Inputs obrigatórios:
- A aplicação/serviço a ser implantado e a plataforma de destino (Kubernetes, VM, serverless etc.)
- A ferramenta de automação desejada ou já em uso (Helm, Terraform, ArgoCD, GitHub Actions) — se não informado, pergunte, pois isso muda toda a estrutura dos artefatos gerados

Inputs opcionais (serão inferidos ou perguntados se necessário):
- Estratégia de deploy preferida (rolling, blue-green, canary): se não especificada, recomende com base no risco descrito pelo usuário (ex.: aplicação crítica com zero downtime tolerável → blue-green) e declare a suposição
- Critérios de health check/smoke test: se ausentes, proponha um conjunto mínimo razoável (endpoint de `/health`, verificação de latência) e sinalize que precisam ser validados pelo time
- Ambientes existentes (dev/staging/produção) e regras de promoção entre eles: pergunte se a resposta mudar a estrutura do pipeline

Se o pedido não especificar a plataforma de destino nem a ferramenta de automação, não assuma uma stack — pergunte antes de gerar qualquer artefato.
</input_handling>

<task>
Produza um pipeline de deploy automatizado completo e executável.

Passo 1: Confirmar plataforma e ferramenta
- Identifique a plataforma de destino e a ferramenta de automação a usar

Passo 2: Escolher a estratégia de deploy
- Avalie o risco da mudança e recomende rolling, blue-green ou canary, justificando a escolha

Passo 3: Especificar os artefatos de automação
- Gere os arquivos necessários (chart Helm, workflow de CI/CD, manifesto ArgoCD) com comentários explicando cada seção relevante

Passo 4: Definir validações pré e pós-deploy
- Testes em staging antes de promover para produção
- Health checks e smoke tests automatizados após o deploy, antes de considerar a mudança "concluída"

Passo 5: Definir rollback automatizado
- Especifique a condição exata que dispara o rollback (ex.: falha em smoke test, health check retornando erro por N tentativas)
- Descreva o procedimento de rollback passo a passo

Passo 6: Autoverificação antes de entregar
- O pipeline testa em staging antes de produção?
- Existe rollback automatizado, não apenas manual?
- Alguém sem contexto prévio consegue executar o pipeline seguindo a documentação gerada?
</task>

<output_specification>
Formato: documento em Markdown com blocos de código para cada artefato (YAML/Helm/workflow)
Extensão: proporcional à complexidade da estratégia escolhida — um rolling deployment simples exige menos artefatos que um blue-green completo
Incluir:
- Seção "Estratégia de Deploy" — qual foi escolhida e por quê
- Seção "Artefatos" — cada arquivo de automação necessário, em blocos de código separados
- Seção "Validações" — testes pré e pós-deploy
- Seção "Rollback" — condição de disparo e procedimento
- Seção "Suposições" — qualquer inferência feita por falta de informação
</output_specification>

<quality_criteria>
Outputs excelentes:
- Toda estratégia de deploy inclui um mecanismo de rollback automatizado, não apenas instruções manuais
- Health checks são específicos e verificáveis (endpoint, código de resposta esperado), nunca vagos como "verificar se está funcionando"
- Artefatos gerados são sintaticamente válidos e usam placeholders claramente marcados quando dados reais (nomes de cluster, registries) não foram fornecidos

Evite:
- Recomendar deploy direto em produção sem passagem por staging
- Omitir o procedimento de rollback
- Gerar configuração com credenciais hardcoded
- Marcar um pipeline como "pronto para produção" sem testes e sem health checks
</quality_criteria>

<constraints>
- Nunca gere credenciais, tokens ou segredos reais — use placeholders explícitos (ex.: `${REGISTRY_TOKEN}`) e recomende um gerenciador de segredos
- Não assuma que o usuário já tem staging configurado — pergunte se não for mencionado, já que isso muda a estratégia recomendada
- Sempre inclua um mecanismo de rollback; se o usuário pedir explicitamente para omiti-lo, alerte sobre o risco antes de prosseguir
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Tenho uma API em Kubernetes gerenciada com Helm. Quero fazer deploy blue-green com rollback automático se os smoke tests falharem."

**Output esperado (resumo):**

- Estratégia escolhida: blue-green, justificada pela necessidade de zero downtime e rollback instantâneo
- Artefatos: `values.yaml` do Helm parametrizado por versão, script de deploy que sobe a versão "green", roda smoke tests e só então troca o seletor do Service
- Validações: smoke test via Newman/Postman contra o ambiente green antes da troca de tráfego
- Rollback: se os smoke tests falharem, o script desinstala a release green automaticamente e mantém o tráfego na versão "blue" atual
- Suposição assinalada: assume-se que já existe um cluster Kubernetes configurado e acessível via `kubectl`/`helm`
