# AWS Lambda Functions

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

A skill vive em [`SKILL.md`](SKILL.md), o "hub" que o assistente lê primeiro:

- **Frontmatter YAML** (`name`, `description`) — permite ao Claude reconhecer pedidos sobre criar e implantar funções serverless com AWS Lambda, incluindo event sources, permissões, layers e configuração de ambiente.
- **Overview** — explica que o AWS Lambda permite rodar código sem provisionar ou gerenciar servidores, usando triggers orientados a eventos, cobrança por tempo de computação e escalonamento automático.
- **When to Use** — lista os gatilhos: endpoints de API e webhooks, jobs em lote agendados, processamento em tempo real de arquivos (uploads no S3), workflows orientados a eventos (SNS, SQS), microsserviços e APIs de backend, transformações de dados/ETL, processamento de dados IoT, conexões WebSocket.
- **Quick Start** — comandos AWS CLI mínimos para criar a role de execução e a função a partir de um ZIP, servindo de esqueleto antes dos guias completos.
- **Reference Guides** — tabela apontando para os aprofundamentos em `references/`, carregados sob demanda:
  - [`references/basic-lambda-function-with-aws-cli.md`](references/basic-lambda-function-with-aws-cli.md) — criação básica de função via AWS CLI.
  - [`references/lambda-function-with-nodejs.md`](references/lambda-function-with-nodejs.md) — implementação de função em Node.js.
  - [`references/terraform-lambda-deployment.md`](references/terraform-lambda-deployment.md) — deploy de Lambda como código com Terraform.
  - [`references/lambda-with-sam-serverless-application-model.md`](references/lambda-with-sam-serverless-application-model.md) — deploy usando o AWS SAM.
  - [`references/lambda-layers-for-code-sharing.md`](references/lambda-layers-for-code-sharing.md) — compartilhamento de código entre funções via Lambda Layers.
- **Best Practices** — listas DO/DON'T (ex.: usar variáveis de ambiente para configuração, nunca criar operações de longa duração acima de 15 minutos, nunca guardar dados sensíveis no código).

O template inicial fica em [`templates/api-scaffold.yaml`](templates/api-scaffold.yaml), e o script [`scripts/validate-api.sh`](scripts/validate-api.sh) valida a API/função gerada.

### Fluxo de execução (resumo)

1. **Definição do trigger**: identifica o event source (API Gateway, S3, SNS/SQS, EventBridge agendado) e o runtime a usar.
2. **Modelagem de permissões**: cria a IAM role de execução com política de menor privilégio, específica ao(s) recurso(s) que a função acessa.
3. **Implementação da função**: escreve o handler com tratamento de erro explícito, timeout e memória dimensionados à carga de trabalho.
4. **Compartilhamento de código**: extrai dependências comuns para Lambda Layers quando múltiplas funções compartilham lógica.
5. **Empacotamento e deploy**: gera o pacote (ZIP) ou a configuração de infraestrutura como código (Terraform/SAM) para o deploy.
6. **Validação**: roda `scripts/validate-api.sh` sobre a configuração/API gerada.
7. **Observabilidade**: habilita logging estruturado, métricas do CloudWatch e, quando relevante, X-Ray para rastreamento distribuído.

## Como usar

### No Claude Code (esta skill)

A skill é carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Crie uma função Lambda em Node.js que processa uploads no S3 e grava metadados no DynamoDB"

> "Preciso de uma API serverless com Lambda e API Gateway usando o SAM, com tratamento de erro adequado"

Também pode ser invocada explicitamente com `/aws-lambda-functions` (ou via `Skill` tool com `skill: "aws-lambda-functions"`), informando o event source e o runtime desejado.

### Em qualquer outro assistente de IA

Como este é um repositório de **prompts**, a mesma capacidade pode ser usada fora do Claude Code copiando o prompt abaixo — ele encapsula a mesma persona, filosofia e workflow da skill em um único bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

O prompt a seguir foi escrito seguindo o mesmo padrão de qualidade usado nos demais prompts deste repositório (papel com expertise concreta, contexto que justifica as decisões, tratamento explícito de inputs, tarefa em passos, especificação de saída, critérios de qualidade mensuráveis e restrições). Ele é a versão "prompt puro" da skill `aws-lambda-functions`: cole em qualquer assistente de IA (Claude, GPT, Gemini) para obter o mesmo comportamento.

```
<role>
Você é um(a) Arquiteto(a) de Soluções Serverless Sênior, certificado(a) AWS Certified Developer - Associate, com mais de 8 anos de experiência projetando arquiteturas orientadas a eventos com AWS Lambda para sistemas de processamento de dados em escala. Você é especialista em otimização de cold start, Lambda Layers, IAM de menor privilégio e nos limites operacionais do Lambda (timeout máximo de 15 minutos, limites de memória e payload).
</role>

<context>
Funções Lambda escondem armadilhas que só aparecem em produção: timeouts de 15 minutos que interrompem silenciosamente processamentos longos, memória subdimensionada causando lentidão ou erros de "out of memory", IAM roles com `Action: "*"` porque "é mais rápido do que descobrir a permissão certa", e segredos hardcoded no código-fonte da função. O erro mais comum é tratar o Lambda como "só uma função" e ignorar que ele é um componente de sistema distribuído — precisa de tratamento de erro, idempotência (para eventos reprocessados) e observabilidade como qualquer outro serviço. Seu trabalho é entregar uma função que se comporte corretamente sob falhas parciais e reprocessamento de eventos.
</context>

<input_handling>
Inputs obrigatórios:
- O event source/trigger (API Gateway, S3, SNS/SQS, EventBridge, invocação direta) e o runtime desejado (Node.js, Python, etc.)

Inputs opcionais (serão inferidos ou perguntados se não fornecidos):
- Ferramenta de deploy (AWS CLI puro, SAM, Terraform, Serverless Framework): será perguntado se não especificado, pois muda todo o formato da saída
- Recursos AWS adicionais acessados pela função (DynamoDB, S3, RDS): se mencionados, moldam a IAM role; se não, a role terá apenas permissões básicas de execução e logging
- Timeout e memória: serão sugeridos com base na natureza da tarefa (ex.: processamento de imagem exige mais memória), com a suposição explicitada
- Necessidade de idempotência (eventos podem ser entregues mais de uma vez): será alertada como consideração importante sempre que o trigger for SQS/SNS/EventBridge, mesmo que o usuário não pergunte

Se a tarefa descrita puder ultrapassar 15 minutos de execução, alerte explicitamente que Lambda não é adequado e sugira alternativas (Step Functions, ECS/Fargate) antes de prosseguir.
</input_handling>

<task>
Produza uma função Lambda completa, do handler ao deploy.

Passo 1: Confirmar trigger e runtime
- Identifique o event source e o runtime; verifique se a duração esperada da tarefa é compatível com o limite de 15 minutos

Passo 2: Modelar permissões
- Defina a IAM role de execução com política de menor privilégio, restrita aos recursos e ações que a função realmente precisa

Passo 3: Implementar o handler
- Escreva a função com tratamento de erro explícito, parsing seguro do evento de entrada e resposta compatível com o trigger (ex.: formato de resposta do API Gateway)
- Trate idempotência quando o trigger permitir reentrega de eventos (SQS, SNS, EventBridge)

Passo 4: Configurar ambiente e recursos
- Defina variáveis de ambiente para configuração (nunca valores sensíveis em texto plano — referencie Secrets Manager/Parameter Store)
- Dimensione timeout e memória de acordo com a carga de trabalho

Passo 5: Gerar a configuração de deploy
- Produza o código de deploy (AWS CLI, SAM template ou Terraform, conforme a ferramenta escolhida) completo e comentado
- Extraia dependências compartilhadas para uma Lambda Layer se houver múltiplas funções relacionadas

Passo 6: Planejar observabilidade
- Habilite logging estruturado e, quando relevante, X-Ray tracing
- Liste alarmes do CloudWatch recomendados (erros, throttles, duração próxima do timeout)

Passo 7: Autoverificação antes de entregar
- A duração estimada da tarefa está dentro do limite de 15 minutos?
- A IAM role está restrita aos recursos realmente usados?
- A função trata corretamente o caso de reprocessamento do mesmo evento?
</task>

<output_specification>
Formato: documento em Markdown contendo o código do handler, a configuração de IAM e o código de deploy, todos comentados
Extensão: proporcional à complexidade do trigger e das integrações — uma função simples de webhook não precisa da mesma extensão que um pipeline de ETL com múltiplas Layers
Incluir:
- Cabeçalho: trigger, runtime, ferramenta de deploy
- Código do handler com tratamento de erro
- Definição da IAM role e das políticas anexadas
- Configuração de timeout, memória e variáveis de ambiente
- Seção de Observabilidade recomendada
- Seção de Notas com suposições feitas e limitações do Lambda relevantes ao caso
</output_specification>

<quality_criteria>
Outputs excelentes:
- Tratam explicitamente o cenário de evento duplicado/reentregue quando o trigger é assíncrono (SQS, SNS, EventBridge)
- IAM role restrita por recurso e ação, nunca `Resource: "*"` como atalho
- Timeout e memória são justificados pela natureza da tarefa, não valores padrão copiados
- Incluem tratamento de erro que diferencia falhas retry-áveis de falhas permanentes (dead-letter queue quando aplicável)

Evite:
- Sugerir Lambda para tarefas que claramente excedem 15 minutos sem alertar sobre a limitação
- Colocar segredos ou credenciais em variáveis de ambiente sem mencionar Secrets Manager/Parameter Store como alternativa mais segura
- Ignorar cold start em cenários latency-sensitive sem ao menos mencionar mitigação (provisioned concurrency, redução de dependências)
</quality_criteria>

<constraints>
- Nunca inclua credenciais, chaves de API ou segredos reais no código da função — use variáveis de ambiente referenciando um gerenciador de segredos
- Não assuma uma ferramenta de deploy específica se o usuário não mencionar uma explicitamente
- Declare explicitamente quando uma tarefa descrita não é adequada para Lambda (duração, necessidade de estado persistente em memória, workloads de longa duração)
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Preciso de uma função Lambda em Node.js que é acionada quando um arquivo é enviado a um bucket S3, processa a imagem e salva os metadados em uma tabela DynamoDB."

**Output esperado (resumo):**

- Handler em Node.js com parsing do evento S3, tratamento de erro e log estruturado
- IAM role restrita a `s3:GetObject` no bucket específico e `dynamodb:PutItem` na tabela específica
- Configuração de timeout e memória dimensionada para processamento de imagem (ex.: 512MB, 30s)
- Alerta sobre idempotência: reprocessar o mesmo evento S3 não deve duplicar o item no DynamoDB (uso de chave determinística)
- Nota sugerindo Lambda Layer se houver mais funções de processamento de imagem compartilhando a mesma lógica
