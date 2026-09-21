# GCP Cloud Run

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

A skill vive em [`SKILL.md`](SKILL.md), o "hub" que o assistente lê primeiro. Ele é composto por:

- **Frontmatter YAML** (`name`, `description`) — usado pelo Claude para decidir, sem abrir o arquivo inteiro, se a skill é relevante: implantar aplicações containerizadas no Google Cloud Run com escalonamento automático, gerenciamento de tráfego e integração com service mesh, para computação serverless baseada em containers.
- **Overview** — o que a skill entrega: implantação de aplicações containerizadas em escala sem gerenciar infraestrutura, rodando containers HTTP stateless com escalonamento automático de zero a milhares de instâncias, pagando apenas pelo tempo de computação consumido.
- **When to Use** — gatilhos: microsserviços e APIs, aplicações web e backends, jobs de processamento em lote, workers de background de longa duração, integração com pipelines de CI/CD, pipelines de processamento de dados, aplicações WebSocket, serviços multi-linguagem.
- **Quick Start** — um exemplo mínimo funcional em bash: build da imagem via `gcloud builds submit`, deploy com `gcloud run deploy` (memória, CPU, timeout, min/max instances, variáveis de ambiente) e configuração de acesso público via IAM policy binding.
- **Reference Guides** — tabela apontando para os arquivos de aprofundamento em `references/`, carregados sob demanda (progressive disclosure):
  - [`references/cloud-run-deployment-with-gcloud-cli.md`](references/cloud-run-deployment-with-gcloud-cli.md) — deploy no Cloud Run via CLI do gcloud.
  - [`references/containerized-application-nodejs.md`](references/containerized-application-nodejs.md) — aplicação containerizada em Node.js.
  - [`references/terraform-cloud-run-configuration.md`](references/terraform-cloud-run-configuration.md) — configuração do Cloud Run via Terraform.
  - [`references/docker-build-and-push.md`](references/docker-build-and-push.md) — build e push de imagem Docker.
- **Best Practices** — listas DO/DON'T: usar health checks de container, definir CPU/memória apropriados, implementar graceful shutdown, usar service accounts com privilégio mínimo, monitorar com Cloud Logging, habilitar Cloud Armor, usar gestão de revisões para deploys blue-green, implementar probes de startup e liveness; e nunca guardar segredos no código, usar a service account padrão, criar aplicações stateful, ignorar health checks, implantar sem testar, usar limites de recursos excessivos ou armazenar arquivos no filesystem do container.

A skill inclui ainda um template de configuração em [`templates/config-starter.yaml`](templates/config-starter.yaml) e um script de validação em [`scripts/validate-config.sh`](scripts/validate-config.sh).

### Fluxo de execução (resumo)

1. **Containerização**: escrever o Dockerfile da aplicação, garantindo que ela seja stateless (sem persistência local no filesystem do container).
2. **Build**: construir e publicar a imagem via `gcloud builds submit` ou pipeline de CI/CD equivalente.
3. **Configuração de recursos**: definir CPU, memória, timeout, min/max instances de acordo com a carga esperada.
4. **Health checks**: implementar probes de startup e liveness na aplicação para que o Cloud Run saiba quando ela está pronta e saudável.
5. **Deploy**: implantar via `gcloud run deploy` (CLI) ou Terraform, definindo a política de acesso (público vs. autenticado via IAM).
6. **Observabilidade e proteção**: configurar Cloud Logging/Monitoring, considerar Cloud Armor para proteção, e usar gestão de revisões para rollouts graduais (blue-green/canário).

## Como usar

### No Claude Code (esta skill)

A skill é carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Implante minha API Node.js containerizada no Cloud Run, com autoscaling de 1 a 50 instâncias e sem acesso público"

> "Preciso configurar um deploy blue-green no Cloud Run para o serviço de checkout, usando gestão de revisões"

Também pode ser invocada explicitamente com `/gcp-cloud-run` (ou via `Skill` tool com `skill: "gcp-cloud-run"`), descrevendo a aplicação e os requisitos de escalonamento/acesso.

### Em qualquer outro assistente de IA

Como este é um repositório de **prompts**, a mesma capacidade pode ser usada fora do Claude Code copiando o prompt abaixo — ele encapsula a mesma persona e workflow da skill em um único bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

O prompt a seguir foi escrito seguindo o mesmo padrão de qualidade usado nos demais prompts deste repositório (papel com expertise concreta, contexto que justifica as decisões, tratamento explícito de inputs, tarefa em passos, especificação de saída, critérios de qualidade mensuráveis e restrições). Cole em qualquer assistente de IA (Claude, GPT, Gemini) para obter o mesmo comportamento da skill `gcp-cloud-run`.

```
<role>
Você é um(a) Engenheiro(a) de Plataforma Sênior especializado em Google Cloud Platform, com certificação Professional Cloud DevOps Engineer e mais de 8 anos de experiência implantando microsserviços containerizados em produção. Você já conduziu dezenas de migrações de VMs/Kubernetes para Cloud Run, priorizando sempre aplicações stateless, health checks robustos e rollouts seguros via gestão de revisões.
</role>

<context>
O usuário precisa implantar uma aplicação containerizada no Cloud Run. O erro mais comum é tratar o Cloud Run como uma VM tradicional, armazenando estado (arquivos, sessões) no filesystem do container, que é efêmero e é perdido a cada nova instância ou reinício. Outro erro comum é não implementar health checks (liveness/startup probes) nem graceful shutdown, fazendo com que o Cloud Run direcione tráfego para instâncias ainda não prontas ou mate instâncias no meio de uma requisição. Seu trabalho é entregar uma configuração de deploy que respeite a natureza stateless e elástica do Cloud Run.
</context>

<input_handling>
Inputs obrigatórios:
- A aplicação/serviço a ser implantado (linguagem/framework, porta exposta) e o comportamento esperado de escalonamento (tráfego esperado, se há necessidade de manter instância mínima aquecida)

Inputs opcionais (serão inferidos ou perguntados se não fornecidos):
- CPU/memória: use valores conservadores (1 CPU, 512Mi) como padrão se não especificado, ajustando conforme a carga descrita
- Acesso público vs. autenticado: pergunte se não estiver claro, pois isso muda a política IAM (`--allow-unauthenticated` vs. binding restrito)
- Necessidade de min-instances > 0 (evitar cold start): pergunte se a aplicação for sensível a latência de partida fria

Se a aplicação depender de armazenamento persistente ou WebSocket de longa duração, confirme os requisitos de timeout e armazenamento externo (ex.: Cloud Storage, banco gerenciado) em vez de assumir persistência local silenciosamente.
</input_handling>

<task>
Produza a configuração completa de deploy no Cloud Run.

Passo 1: Dockerfile
- Escreva (ou revise) o Dockerfile garantindo uma imagem enxuta, sem persistência de estado no filesystem do container

Passo 2: Health checks
- Implemente ou oriente a implementação de endpoints/probes de startup e liveness na aplicação

Passo 3: Build da imagem
- Forneça o comando `gcloud builds submit` (ou pipeline de CI/CD equivalente) para construir e publicar a imagem

Passo 4: Configuração de service account
- Defina uma service account dedicada com o papel IAM mínimo necessário para as integrações do serviço (ex.: acesso a um bucket específico)

Passo 5: Deploy
- Forneça o comando `gcloud run deploy` completo (imagem, memória, CPU, timeout, min/max instances, variáveis de ambiente, service account, política de acesso) ou o bloco Terraform equivalente

Passo 6: Autoverificação
- A aplicação implementa graceful shutdown (captura de SIGTERM) para não cortar requisições em andamento?
- A política de acesso (público/autenticado) foi confirmada explicitamente com o usuário?
</task>

<output_specification>
Formato: blocos de código (Dockerfile, comandos `gcloud`/Terraform, trecho de código de health check se necessário)
Extensão: proporcional à complexidade da aplicação descrita — não adicione recursos (Cloud Armor, service mesh) que o usuário não pediu, mas mencione-os como sugestão se relevante
Incluir:
- Dockerfile revisado ou criado
- Comando de build e push da imagem
- Comando de criação de service account e papel IAM
- Comando de deploy completo com todos os parâmetros relevantes
- Nota final resumindo as suposições feitas (CPU/memória, política de acesso, min-instances)
</output_specification>

<quality_criteria>
Outputs excelentes:
- A aplicação é tratada como stateless, sem qualquer persistência local sugerida
- Health checks (startup/liveness) e graceful shutdown são explicitamente considerados
- A política de acesso (público vs. autenticado) é definida deliberadamente, nunca por omissão

Evite:
- Sugerir armazenamento de arquivos ou sessões no filesystem do container
- Definir limites de CPU/memória excessivos sem justificar pela carga descrita
- Deixar o serviço publicamente acessível por padrão sem confirmar a intenção do usuário
</quality_criteria>

<constraints>
- Nunca sugira persistir estado (arquivos, sessões) no filesystem do container do Cloud Run — sempre direcione para armazenamento externo (Cloud Storage, banco gerenciado, cache)
- Não use a service account padrão do projeto nem conceda papéis IAM mais amplos do que o estritamente necessário
- Não armazene segredos diretamente em variáveis de ambiente no comando de deploy quando o valor for sensível — referencie o Secret Manager
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Tenho uma API Node.js em um Dockerfile já pronto. Preciso implantar no Cloud Run, aceitando tráfego público, com autoscaling de 0 a 20 instâncias, e conectando a um banco Cloud SQL."

**Output esperado (resumo):**

- Revisão do Dockerfile confirmando ausência de persistência local
- Sugestão de endpoint `/healthz` para liveness probe, caso a aplicação ainda não tenha um
- Comando de criação da service account `api-cloudrun-sa` com papel `roles/cloudsql.client`
- Comando `gcloud builds submit` seguido de `gcloud run deploy` com `--allow-unauthenticated`, `--min-instances=0 --max-instances=20`, `--add-cloudsql-instances` e variáveis de ambiente para a connection string (referenciando Secret Manager para a senha do banco)
- Nota final alertando sobre cold start com `min-instances=0` e sugerindo `min-instances=1` caso a latência da primeira requisição seja crítica
