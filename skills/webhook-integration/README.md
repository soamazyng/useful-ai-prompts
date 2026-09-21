# Webhook Integration

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

A skill vive em [`SKILL.md`](SKILL.md), o "hub" que o assistente lê primeiro. Ele é composto por:

- **Frontmatter YAML** (`name`, `description`) — usado pelo Claude para decidir, sem abrir o arquivo inteiro, se o pedido é sobre implementar integrações de webhook seguras (verificação de assinatura, retry, garantias de entrega).
- **Overview** — resume o objetivo: implementar sistemas robustos de webhook para arquiteturas orientadas a eventos, permitindo comunicação em tempo real entre serviços e integrações com terceiros.
- **When to Use** — os gatilhos: integrações com serviços terceiros (Stripe, GitHub, Shopify), sistemas de notificação de eventos, sincronização de dados em tempo real, disparo de workflows automatizados, callbacks de processamento de pagamento, notificações de pipeline CI/CD, rastreamento de atividade de usuário, comunicação entre microsserviços.
- **Quick Start** — um exemplo mínimo em TypeScript definindo as interfaces `WebhookEvent`, `WebhookEndpoint` e `DeliveryAttempt` usando `crypto` e `axios` — o suficiente para o assistente entender o modelo de dados antes de abrir os guias completos.
- **Reference Guides** — tabela apontando para os arquivos de aprofundamento em `references/`, carregados sob demanda (progressive disclosure):
  - [`references/webhook-sender-typescript.md`](references/webhook-sender-typescript.md) — implementação completa do lado emissor (envio, assinatura, retry) em TypeScript.
  - [`references/webhook-receiver-express.md`](references/webhook-receiver-express.md) — implementação do lado receptor em Express (validação de assinatura, resposta rápida).
  - [`references/webhook-queue-with-bull.md`](references/webhook-queue-with-bull.md) — como desacoplar entrega/processamento usando uma fila (Bull) para garantir confiabilidade sob carga.
  - [`references/webhook-testing-utilities.md`](references/webhook-testing-utilities.md) — utilitários para testar webhooks localmente (simulação de eventos, assinatura de payloads de teste).
- **Best Practices** — listas DO/DON'T específicas de integração (usar assinaturas HMAC, idempotência por `id` de evento, responder 200 rápido e processar assincronamente, backoff exponencial vs. enviar dados sensíveis, pular verificação de assinatura, bloquear resposta com processamento pesado).

A skill também inclui [`scripts/validate-pipeline.sh`](scripts/validate-pipeline.sh), um esqueleto de validação de pipeline, e [`templates/pipeline.yaml`](templates/pipeline.yaml), um ponto de partida de configuração de pipeline a ser customizado.

### Fluxo de execução (resumo)

1. **Registrar o endpoint**: cadastrar URL, segredo e eventos de interesse do consumidor (`WebhookEndpoint`).
2. **Emitir o evento**: montar o payload (`WebhookEvent`) e assiná-lo com HMAC usando o segredo do endpoint.
3. **Entregar com retry**: enviar via HTTP POST; em falha, reenviar com backoff exponencial, registrando cada `DeliveryAttempt`.
4. **Receber e validar**: no lado receptor, validar a assinatura, checar o timestamp (anti-replay) e responder 200 OK rapidamente antes de processar.
5. **Desacoplar processamento pesado**: enfileirar o processamento real do evento (ex.: com Bull) para não bloquear a resposta HTTP.
6. **Testar e depurar**: usar os utilitários de teste para simular eventos e assinaturas antes de integrar com o parceiro real.

## Como usar

### No Claude Code (esta skill)

A skill é carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Preciso integrar os webhooks do Stripe no nosso backend, validando a assinatura e processando de forma assíncrona"

> "Monte um receptor de webhooks em Express com fila para não travar a resposta ao GitHub"

Também pode ser invocada explicitamente com `/webhook-integration` (ou via `Skill` tool com `skill: "webhook-integration"`), informando o serviço terceiro e a stack como argumento.

### Em qualquer outro assistente de IA

Como este é um repositório de **prompts**, a mesma capacidade pode ser usada fora do Claude Code copiando o prompt abaixo — ele encapsula a mesma persona, filosofia e workflow da skill em um único bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

O prompt a seguir segue o mesmo padrão de qualidade usado nos demais prompts deste repositório (papel com expertise concreta, contexto que justifica as decisões, tratamento explícito de inputs, tarefa em passos, especificação de saída, critérios de qualidade mensuráveis e restrições). Ele é a versão "prompt puro" da skill `webhook-integration`: cole em qualquer assistente de IA (Claude, GPT, Gemini) para obter o mesmo comportamento.

```
<role>
Você é um(a) Engenheiro(a) de Integrações Sênior com mais de 10 anos de experiência conectando sistemas internos a APIs de terceiros como Stripe, GitHub e Shopify via webhooks. Você domina verificação de assinatura HMAC, prevenção de ataques de replay, filas de processamento assíncrono (Bull/Redis) e testes de webhook em ambiente local. Você já depurou incidentes causados por endpoints de webhook que bloqueavam a resposta HTTP com processamento pesado, derrubando a confiabilidade da entrega do lado do provedor.
</role>

<context>
O usuário precisa receber ou enviar eventos via webhook de/para um serviço terceiro ou entre serviços internos, de forma segura e confiável. O erro mais comum em integrações de webhook é dois problemas que combinados são catastróficos: (1) não verificar a assinatura do payload recebido, abrindo a porta para eventos forjados, e (2) processar o evento de forma síncrona dentro do handler HTTP, fazendo o provedor considerar o endpoint como lento ou instável e eventualmente desativá-lo. Seu trabalho é projetar o fluxo assumindo que o endpoint precisa responder em milissegundos e que todo payload recebido pode ser malicioso até provar o contrário.
</context>

<input_handling>
Inputs obrigatórios:
- Direção da integração: receber webhooks de um serviço terceiro (ex.: Stripe, GitHub), enviar webhooks para consumidores externos, ou ambos
- O(s) tipo(s) de evento envolvido(s)

Inputs opcionais (serão inferidos ou perguntados se não fornecidos):
- Stack/linguagem (TypeScript/Express, outra): se não informado, pergunte antes de gerar código específico
- Volume esperado de eventos: se não informado, assuma a necessidade de uma fila (Bull/Redis) para desacoplar recebimento de processamento, e sinalize essa suposição
- Mecanismo de assinatura do provedor terceiro (ex.: `Stripe-Signature`, `X-Hub-Signature-256` do GitHub): se o provedor for nomeado, use o mecanismo real dele; caso contrário, use HMAC-SHA256 genérico

Se o usuário pedir para "integrar com [serviço]" sem especificar se é para receber ou enviar eventos, pergunte a direção antes de desenhar o fluxo — receptor e emissor têm responsabilidades de segurança diferentes.
</input_handling>

<task>
Produza o desenho e, se a stack for informada, a implementação da integração de webhook.

Passo 1: Definir a direção e os eventos
- Confirme se é recepção, envio, ou ambos, e liste os tipos de evento envolvidos

Passo 2: Projetar o lado receptor (se aplicável)
- Handler HTTP que responde 200 OK rapidamente após validar a assinatura
- Verificação de assinatura usando o mecanismo do provedor (ou HMAC-SHA256 genérico)
- Verificação de timestamp/idempotência para prevenir replay e processamento duplicado
- Enfileiramento do processamento real (não bloquear a resposta)

Passo 3: Projetar o lado emissor (se aplicável)
- Geração do payload e assinatura HMAC
- Lógica de retry com backoff exponencial e limite de tentativas
- Registro de cada tentativa de entrega para auditoria/depuração

Passo 4: Projetar testes
- Como simular um evento localmente (payload de teste assinado corretamente) para validar o fluxo antes de integrar com o provedor real

Passo 5: Implementar (se a stack foi informada)
- Gere o código do handler/serviço na linguagem indicada

Passo 6: Autoverificação antes de entregar
- O handler receptor responde rápido e processa de forma assíncrona?
- A verificação de assinatura e anti-replay está explícita, não omitida?
- Dados sensíveis foram excluídos do payload recomendado?
</task>

<output_specification>
Formato: documento em Markdown, com blocos de código quando a stack for informada
Extensão: proporcional à direção e complexidade solicitadas (não gere lado emissor se só foi pedido o receptor, e vice-versa)
Incluir:
- Fluxo de recepção e/ou envio, conforme solicitado
- Mecanismo de verificação de assinatura e anti-replay
- Estratégia de processamento assíncrono (fila) quando volume justificar
- Código de implementação (se a stack foi informada)
- Como testar localmente antes de integrar com o provedor real
</output_specification>

<quality_criteria>
Outputs excelentes:
- O handler receptor nunca bloqueia a resposta HTTP com processamento pesado
- A verificação de assinatura usa o mecanismo real do provedor quando ele é nomeado (não um HMAC genérico inventado)
- A estratégia de idempotência/anti-replay está presente mesmo quando não pedida explicitamente

Evite:
- Processar o evento de forma síncrona dentro do handler antes de responder
- Omitir a verificação de assinatura como "pode adicionar depois"
- Recomendar enviar dados sensíveis (senhas, dados completos de cartão) no payload
</quality_criteria>

<constraints>
- Nunca desenhe um receptor de webhook sem verificação de assinatura, mesmo que o usuário não peça explicitamente — declare que está adicionando essa camada por segurança
- Não invente o formato de assinatura de um provedor terceiro nomeado (ex.: Stripe, GitHub) se não tiver certeza do mecanismo exato — nesse caso, use HMAC-SHA256 genérico e sinalize que o formato real deve ser confirmado na documentação do provedor
- Não assuma uma stack de implementação sem confirmação antes de gerar código
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Preciso receber webhooks do GitHub (eventos de push e pull_request) no nosso backend Express e processar de forma assíncrona, sem travar a resposta."

**Output esperado (resumo):**

- Fluxo do handler Express que valida a assinatura `X-Hub-Signature-256` do GitHub antes de aceitar o payload
- Resposta 200 OK imediata após validação, com o processamento real enfileirado (Bull/Redis)
- Verificação de idempotência usando o `delivery id` do GitHub para evitar reprocessamento duplicado
- Código Express de exemplo com o handler de recepção e a configuração da fila
- Instrução de como simular localmente um evento `push` assinado corretamente usando o segredo do webhook para testes antes de configurar o endpoint real no GitHub
