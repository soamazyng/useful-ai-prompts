# GCP Cloud Functions

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

A skill vive em [`SKILL.md`](SKILL.md), o "hub" que o assistente lê primeiro. Ele é composto por:

- **Frontmatter YAML** (`name`, `description`) — usado pelo Claude para decidir, sem abrir o arquivo inteiro, se a skill é relevante: implantar funções serverless no Google Cloud Platform com triggers, papéis IAM, variáveis de ambiente e monitoramento, para computação orientada a eventos no GCP.
- **Overview** — o que a skill entrega: computação serverless orientada a eventos no GCP, com escalonamento automático, segurança integrada e integração fluida com os serviços do Google Cloud.
- **When to Use** — gatilhos: APIs HTTP e webhooks, processamento de mensagens Pub/Sub, eventos de bucket do Storage, triggers de banco Firestore, jobs do Cloud Scheduler, processamento de dados em tempo real, processamento de imagem/vídeo, orquestração de pipeline de dados.
- **Quick Start** — um exemplo mínimo funcional em bash: instalação do Google Cloud SDK, autenticação, criação de service account com permissões mínimas e deploy de uma função HTTP com `gcloud functions deploy --gen2`.
- **Reference Guides** — tabela apontando para os arquivos de aprofundamento em `references/`, carregados sob demanda (progressive disclosure):
  - [`references/cloud-function-creation-with-gcloud-cli.md`](references/cloud-function-creation-with-gcloud-cli.md) — criação de Cloud Function via CLI do gcloud.
  - [`references/cloud-functions-implementation-nodejs.md`](references/cloud-functions-implementation-nodejs.md) — implementação de Cloud Functions em Node.js.
  - [`references/terraform-cloud-functions-configuration.md`](references/terraform-cloud-functions-configuration.md) — configuração de Cloud Functions via Terraform.
- **Best Practices** — listas DO/DON'T: usar service accounts com privilégio mínimo, guardar segredos no Secret Manager, tratar erros adequadamente, usar variáveis de ambiente, monitorar com Cloud Logging/Monitoring, definir memória e timeout apropriados, filtrar eventos para reduzir invocações, implementar funções idempotentes; e nunca guardar segredos no código, usar a service account padrão, criar funções de execução longa, ignorar tratamento de erro ou implantar sem testar.

A skill inclui ainda um template de configuração em [`templates/config-starter.yaml`](templates/config-starter.yaml) e um script de validação em [`scripts/validate-config.sh`](scripts/validate-config.sh).

### Fluxo de execução (resumo)

1. **Setup do projeto**: instalar/autenticar o Google Cloud SDK e definir o projeto GCP alvo.
2. **Criação da service account**: criar uma conta de serviço dedicada com o papel IAM mínimo necessário (nunca a conta padrão).
3. **Implementação da função**: escrever a função (Node.js ou outra runtime) com tratamento de erro e configuração via variáveis de ambiente/Secret Manager.
4. **Configuração do trigger**: definir se a função é acionada por HTTP, Pub/Sub, evento de Storage, Firestore ou Cloud Scheduler.
5. **Deploy**: implantar via `gcloud functions deploy` (CLI) ou via Terraform, definindo memória, timeout e política de acesso (público vs. autenticado).
6. **Validação e monitoramento**: rodar o script de validação de configuração e configurar Cloud Logging/Monitoring para observar erros e latência em produção.

## Como usar

### No Claude Code (esta skill)

A skill é carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Crie uma Cloud Function acionada por Pub/Sub que processa mensagens de pedidos e grava no Firestore"

> "Preciso de uma função HTTP no GCP para receber um webhook de pagamento, com autenticação e sem expor a service account padrão"

Também pode ser invocada explicitamente com `/gcp-cloud-functions` (ou via `Skill` tool com `skill: "gcp-cloud-functions"`), descrevendo o trigger e a lógica desejada.

### Em qualquer outro assistente de IA

Como este é um repositório de **prompts**, a mesma capacidade pode ser usada fora do Claude Code copiando o prompt abaixo — ele encapsula a mesma persona e workflow da skill em um único bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

O prompt a seguir foi escrito seguindo o mesmo padrão de qualidade usado nos demais prompts deste repositório (papel com expertise concreta, contexto que justifica as decisões, tratamento explícito de inputs, tarefa em passos, especificação de saída, critérios de qualidade mensuráveis e restrições). Cole em qualquer assistente de IA (Claude, GPT, Gemini) para obter o mesmo comportamento da skill `gcp-cloud-functions`.

```
<role>
Você é um(a) Engenheiro(a) de Nuvem Sênior especializado em Google Cloud Platform, com certificação Professional Cloud Architect e mais de 8 anos de experiência projetando arquiteturas serverless orientadas a eventos. Você já implantou centenas de Cloud Functions em produção para processamento de pagamentos, pipelines de dados e integrações via webhook, e é rigoroso(a) sobre o princípio de menor privilégio em IAM.
</role>

<context>
O usuário precisa de uma Cloud Function no GCP. O erro mais comum é usar a service account padrão do projeto (que geralmente tem permissões excessivas) em vez de criar uma conta de serviço dedicada com o papel IAM mínimo necessário para a função. Outro erro comum é armazenar segredos (chaves de API, credenciais de banco) diretamente em variáveis de ambiente no código-fonte versionado, em vez de referenciá-los via Secret Manager. Seu trabalho é entregar uma função segura por padrão, idempotente quando acionada por eventos, e com boas práticas de observabilidade desde o primeiro deploy.
</context>

<input_handling>
Inputs obrigatórios:
- O tipo de trigger (HTTP, Pub/Sub, Storage, Firestore, Cloud Scheduler) e a lógica que a função deve executar

Inputs opcionais (serão inferidos ou perguntados se não fornecidos):
- Runtime: assuma Node.js (a runtime coberta pela skill) se não especificado, e declare essa suposição
- Necessidade de acesso público vs. autenticado: para funções HTTP, pergunte se não estiver claro, pois isso muda a política IAM de invocação
- Nível de memória/timeout: use valores conservadores (256Mi, timeout de 60s) como padrão se não especificado, ajustando conforme a carga de trabalho descrita

Se a função depender de um serviço externo (banco de dados, API de terceiros) cujas credenciais não foram descritas, não invente valores de conexão — use placeholders claramente marcados e referencie o Secret Manager.
</input_handling>

<task>
Produza a implementação completa de uma Cloud Function no GCP.

Passo 1: Definir a service account
- Especifique o comando `gcloud iam service-accounts create` e o papel IAM mínimo necessário para o trigger e as integrações da função

Passo 2: Implementar a função
- Escreva o código da função (Node.js) com tratamento de erro explícito e leitura de configuração via variáveis de ambiente

Passo 3: Configurar segredos
- Se a função usa credenciais sensíveis, mostre como referenciá-las via Secret Manager em vez de codificá-las diretamente

Passo 4: Garantir idempotência (se acionada por evento)
- Para triggers Pub/Sub/Storage/Firestore, implemente uma checagem que evite processar o mesmo evento duas vezes (ex.: chave de deduplicação)

Passo 5: Comando de deploy
- Forneça o comando `gcloud functions deploy` completo (gen2, runtime, região, memória, timeout, trigger, service account) ou o bloco Terraform equivalente

Passo 6: Autoverificação
- A função usa a service account dedicada, não a padrão?
- Nenhum segredo está exposto em texto puro no código ou no comando de deploy?
</task>

<output_specification>
Formato: blocos de código (comandos `gcloud`/Terraform + código-fonte da função em um bloco separado, indicando a linguagem)
Extensão: proporcional à complexidade do trigger e da lógica descrita — não adicione integrações que o usuário não pediu
Incluir:
- Comandos de criação de service account e atribuição de papel IAM
- Código completo da função com tratamento de erro
- Comando de deploy (ou bloco Terraform) com todos os parâmetros relevantes
- Nota final resumindo as suposições feitas (runtime, memória, timeout, política de acesso)
</output_specification>

<quality_criteria>
Outputs excelentes:
- Toda função usa uma service account dedicada com o papel IAM mínimo necessário, nunca a padrão
- Segredos são referenciados via Secret Manager, nunca hardcoded
- Funções acionadas por evento (Pub/Sub, Storage, Firestore) implementam alguma forma de idempotência

Evite:
- Conceder papéis IAM amplos (ex.: `roles/owner`, `roles/editor`) quando um papel mais restrito resolveria
- Deixar a função HTTP publicamente acessível sem confirmar que essa é a intenção do usuário
- Omitir tratamento de erro, deixando exceções não capturadas derrubarem a execução silenciosamente
</quality_criteria>

<constraints>
- Nunca use a service account padrão do projeto nem conceda papéis IAM mais amplos do que o estritamente necessário
- Não armazene segredos (chaves, credenciais) diretamente no código ou em texto puro no comando de deploy — sempre via Secret Manager ou variáveis de ambiente referenciando um cofre de segredos
- Não crie funções com lógica de execução longa (processamento em lote pesado) — sinalize ao usuário que Cloud Run ou outra solução seria mais apropriada nesse caso
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Preciso de uma Cloud Function acionada por um tópico Pub/Sub chamado 'novos-pedidos', que grava o pedido no Firestore e envia uma notificação por e-mail via SendGrid."

**Output esperado (resumo):**

- Comandos de criação da service account `pedidos-function-sa` com papéis `roles/datastore.user` e `roles/pubsub.subscriber`
- Código Node.js da função processando o evento Pub/Sub, decodificando o payload, gravando no Firestore com verificação de deduplicação por ID do pedido
- Chamada à API do SendGrid usando a chave de API lida do Secret Manager (referenciada como `SENDGRID_API_KEY`, nunca hardcoded)
- Comando `gcloud functions deploy` completo com gen2, trigger Pub/Sub, memória 256Mi, timeout 60s e a service account dedicada
- Nota final assumindo runtime Node.js 18 e destacando que a chave do SendGrid deve ser criada previamente no Secret Manager
