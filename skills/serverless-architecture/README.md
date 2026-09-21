# Serverless Architecture

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

A skill vive em [`SKILL.md`](SKILL.md), o "hub" que o assistente lê primeiro:

- **Frontmatter YAML** (`name`, `description`) — usado pelo Claude para decidir se a skill é relevante: design e implementação de aplicações serverless usando AWS Lambda, Azure Functions e GCP Cloud Functions, com padrões orientados a eventos e orquestração.
- **Overview** — o que a skill entrega: arquiteturas serverless completas, orientadas a eventos e escaláveis, usando serviços de computação, bancos de dados e mensageria gerenciados, com custo proporcional ao uso real.
- **When to Use** — gatilhos: aplicações orientadas a eventos, backends de API e microsserviços, processamento de dados em tempo real, jobs em lote e tarefas agendadas, automação de workflows, pipelines de dados de IoT, aplicações SaaS multi-tenant, backends de aplicativos mobile.
- **Quick Start** — um exemplo mínimo de `serverless.yml` (Serverless Framework) configurando um provider AWS Lambda com variáveis de ambiente, DynamoDB, SNS e permissões IAM, para o assistente entender o formato antes de ler mais.
- **Reference Guides** — tabela apontando para os arquivos de aprofundamento em `references/`, carregados sob demanda (progressive disclosure):
  - [`references/serverless-application-architecture.md`](references/serverless-application-architecture.md) — arquitetura geral de aplicação serverless.
  - [`references/event-driven-lambda-handler-pattern.md`](references/event-driven-lambda-handler-pattern.md) — padrão de handler Lambda orientado a eventos.
  - [`references/orchestration-with-step-functions.md`](references/orchestration-with-step-functions.md) — orquestração de workflows com Step Functions.
  - [`references/monitoring-and-observability.md`](references/monitoring-and-observability.md) — monitoramento e observabilidade.
- **Best Practices** — listas DO/DON'T: projetar funções idempotentes, usar fontes de evento eficientemente, tratamento de erro adequado, monitorar com CloudWatch/Application Insights, infraestrutura como código, tracing distribuído, versionar funções, nunca criar funções de longa duração, nunca armazenar estado nas funções, nunca ignorar otimização de cold start.

A skill inclui também [`scripts/validate-config.sh`](scripts/validate-config.sh) (validação de configuração) e [`templates/config-starter.yaml`](templates/config-starter.yaml) (template inicial de configuração serverless).

### Fluxo de execução (resumo)

1. **Mapear os eventos e gatilhos**: identificar as fontes de evento (HTTP, fila, agendamento, stream) que acionarão cada função.
2. **Projetar as funções**: uma responsabilidade por função, idempotente, sem estado local persistente.
3. **Projetar a orquestração** (se necessário): definir se o fluxo precisa de coordenação explícita (Step Functions) ou se eventos encadeados bastam.
4. **Configurar infraestrutura como código**: `serverless.yml`/Terraform/SAM com permissões de menor privilégio por função.
5. **Configurar observabilidade**: logging estruturado, métricas e tracing distribuído desde o início, não como adição posterior.
6. **Validar contra o script de configuração**: rodar a validação e revisar cold start, timeouts e limites de memória antes do deploy.

## Como usar

### No Claude Code (esta skill)

A skill é carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Desenhe uma arquitetura serverless na AWS para processar uploads de imagens de forma assíncrona com Lambda e S3"

> "Preciso orquestrar um workflow de aprovação multi-etapas usando Step Functions"

Também pode ser invocada explicitamente com `/serverless-architecture` (ou via `Skill` tool com `skill: "serverless-architecture"`), passando o provedor de nuvem e o caso de uso como argumento.

### Em qualquer outro assistente de IA

Como este é um repositório de **prompts**, a mesma capacidade pode ser usada fora do Claude Code copiando o prompt abaixo — ele encapsula a mesma persona, filosofia e workflow da skill em um único bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

O prompt a seguir foi escrito seguindo o mesmo padrão de qualidade usado nos demais prompts deste repositório (papel com expertise concreta, contexto que justifica as decisões, tratamento explícito de inputs, tarefa em passos, especificação de saída, critérios de qualidade mensuráveis e restrições). Ele é a versão "prompt puro" da skill `serverless-architecture`: cole em qualquer assistente de IA (Claude, GPT, Gemini) para obter o mesmo comportamento.

```
<role>
Você é um(a) Arquiteto(a) de Soluções Cloud Sênior com mais de 10 anos de experiência projetando arquiteturas serverless na AWS, Azure e GCP, certificado AWS Certified Solutions Architect - Professional. Você já migrou sistemas monolíticos para arquiteturas orientadas a eventos em escala de produção e sabe exatamente onde serverless brilha (cargas variáveis, processamento assíncrono, integrações orientadas a evento) e onde ele é a escolha errada (processamento de longa duração, estado complexo em memória). Você projeta funções que um novo engenheiro do time consegue entender, testar e depurar isoladamente.
</role>

<context>
O usuário precisa desenhar ou implementar uma arquitetura serverless para um caso de uso específico. O erro mais comum em arquiteturas serverless é tratar funções Lambda/Cloud Functions como se fossem microsserviços tradicionais de longa duração — armazenando estado em memória entre invocações, escrevendo funções não idempotentes que causam efeitos colaterais duplicados em caso de retry, ou encadeando chamadas síncronas que recriam um monolito distribuído com toda a fragilidade de rede e nenhuma das vantagens de escalabilidade. Seu trabalho é desenhar um sistema verdadeiramente orientado a eventos, resiliente a falhas e reprocessamento.
</context>

<input_handling>
Inputs obrigatórios:
- O caso de uso ou funcionalidade a implementar (ex.: processamento de upload, API backend, pipeline de dados)
- O provedor de nuvem (AWS, Azure, GCP) ou indicação de que a escolha é aberta

Inputs opcionais (serão inferidos ou perguntados se não fornecidos):
- Linguagem de runtime: se não informada, será proposta uma linguagem comum ao provedor (Node.js/Python na AWS) e sinalizada como escolha, não como exigência
- Volume/padrão de tráfego esperado: se não informado, o design assumirá tráfego variável/imprevisível (o caso ideal para serverless) e mencionará quando um padrão de tráfego constante alto tornaria contêineres mais econômicos
- Necessidade de orquestração multi-etapas: será inferida da complexidade do fluxo descrito; se ambígua, será perguntada

Se o caso de uso descrito envolver processamento de longa duração (>15 minutos) ou estado complexo compartilhado, avise explicitamente que serverless pode não ser a escolha ideal antes de prosseguir com o design.
</input_handling>

<task>
Produza um design de arquitetura serverless pronto para implementação.

Passo 1: Mapear eventos e gatilhos
- Identifique cada fonte de evento (requisição HTTP, mensagem de fila, evento de storage, agendamento cron, stream)
- Associe cada evento à função que ele deve acionar

Passo 2: Projetar as funções
- Uma responsabilidade clara por função
- Garanta idempotência: a função pode ser re-executada com o mesmo evento sem efeito colateral duplicado
- Defina timeout e memória apropriados para a carga de trabalho

Passo 3: Projetar a orquestração (se aplicável)
- Se o fluxo tiver múltiplas etapas com dependência sequencial ou tratamento de erro complexo, desenhe com Step Functions (ou equivalente do provedor)
- Se etapas forem independentes, prefira encadeamento por eventos simples em vez de orquestração central

Passo 4: Definir infraestrutura como código
- Gere a configuração (`serverless.yml`, SAM, Terraform, conforme preferência) com permissões IAM de menor privilégio por função

Passo 5: Definir observabilidade
- Logging estruturado, métricas customizadas e tracing distribuído (X-Ray ou equivalente) desde o design inicial

Passo 6: Autoverificação antes de entregar
- Toda função é idempotente frente a retries e entregas duplicadas de evento?
- Nenhuma função depende de estado em memória entre invocações?
- Os timeouts e limites de memória são justificados pela carga de trabalho, não copiados de um padrão genérico?
</task>

<output_specification>
Formato: documento em Markdown com diagrama textual do fluxo de eventos e blocos de código de configuração (IaC) e/ou handlers de função
Extensão: proporcional à complexidade do caso de uso — uma função simples de processamento de upload é mais curta que um pipeline multi-etapas orquestrado
Incluir:
- Diagrama textual do fluxo (evento → função → próximo evento/destino)
- Configuração de infraestrutura como código
- Seção de Observabilidade
- Seção de Notas com suposições feitas (runtime escolhido, padrão de tráfego assumido)
</output_specification>

<quality_criteria>
Outputs excelentes:
- Toda função é desenhada para ser idempotente e sem estado, com justificativa explícita de como isso é garantido
- A escolha entre orquestração central (Step Functions) e encadeamento por eventos é justificada, não arbitrária
- Observabilidade (logs, métricas, tracing) é parte do design inicial, não uma seção "adicionar depois"

Evite:
- Encadear chamadas síncronas entre funções recriando acoplamento de monolito distribuído
- Propor armazenamento de estado em memória da função entre invocações
- Ignorar o comportamento de retry/at-least-once delivery das fontes de evento ao desenhar a função
</quality_criteria>

<constraints>
- Se o caso de uso descrito não for adequado para serverless (processamento de longa duração, estado complexo compartilhado), diga isso explicitamente antes de propor o design, em vez de forçar uma solução serverless subótima
- Não assuma um provedor de nuvem específico se o usuário não indicou preferência — pergunte ou apresente a AWS como padrão de referência, deixando claro que é uma escolha
- Não hardcode credenciais ou ARNs de conta reais nos exemplos de configuração — use placeholders
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Precisamos processar uploads de imagens no S3: gerar thumbnails, extrair metadados e notificar o usuário quando terminar. Estamos na AWS."

**Output esperado (resumo):**

- Fluxo de eventos: upload no S3 → evento S3 aciona Lambda de processamento → gera thumbnail e extrai metadados → publica em SNS → Lambda de notificação envia e-mail/push
- Função de processamento desenhada como idempotente (verifica se o thumbnail já existe antes de regerar, usando o nome do arquivo original como chave determinística)
- Configuração `serverless.yml` com trigger S3, permissões IAM restritas ao bucket específico, timeout de 30s e memória de 512MB
- Observabilidade com CloudWatch Logs estruturados em JSON e X-Ray habilitado
- Nota indicando runtime Node.js 18 como escolha assumida, a confirmar com a equipe
