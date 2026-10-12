# Webhook Development

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

A skill vive em [`SKILL.md`](SKILL.md), o "hub" que o assistente lê primeiro. Ele é composto por:

- **Frontmatter YAML** (`name`, `description`) — usado pelo Claude para decidir, sem abrir o arquivo inteiro, se o pedido é sobre construir sistemas de webhook (notificações de evento, integrações, arquiteturas orientadas a eventos).
- **Overview** — resume o objetivo: construir sistemas de webhook confiáveis com entrega de eventos, verificação de assinatura, lógica de retry e tratamento de dead-letter para integrações assíncronas.
- **When to Use** — os gatilhos: notificações em tempo real para sistemas externos, arquiteturas orientadas a eventos, integração com plataformas terceiras, trilhas de auditoria/logging, disparo de workflows automatizados, notificações de pagamento/pedido.
- **Quick Start** — um exemplo mínimo em JSON de um payload de evento de webhook (`order.created`) com `id`, `timestamp`, `event`, `version`, `data`, `attempt` e `retryable` — o suficiente para o assistente entender o formato esperado de evento antes de abrir os guias completos.
- **Reference Guides** — tabela apontando para os arquivos de aprofundamento em `references/`, carregados sob demanda (progressive disclosure):
  - [`references/webhook-event-schema.md`](references/webhook-event-schema.md) — o schema de evento de webhook em detalhe (campos obrigatórios, versionamento).
  - [`references/nodejs-webhook-service.md`](references/nodejs-webhook-service.md) — implementação completa de um serviço de entrega de webhooks em Node.js.
  - [`references/python-webhook-handler.md`](references/python-webhook-handler.md) — implementação equivalente em Python.
  - [`references/best-practices.md`](references/best-practices.md) — boas práticas adicionais de eventos e entrega de webhooks.
- **Best Practices** — listas DO/DON'T genéricas de engenharia (seguir padrões estabelecidos, testar antes de fazer deploy vs. pular testes/validação, ignorar tratamento de erros, hard-code de configuração).

A skill também inclui [`scripts/validate-schema.sh`](scripts/validate-schema.sh), um esqueleto de validação de schema, e [`templates/migration-template.sql`](templates/migration-template.sql), um ponto de partida para a migração de banco de dados que registra eventos/tentativas de entrega.

### Fluxo de execução (resumo)

1. **Definir o schema de evento**: estabelecer os campos obrigatórios (id único, timestamp, tipo de evento, versão, payload de dados) seguindo o padrão descrito em `webhook-event-schema.md`.
2. **Implementar o disparo do evento**: publicar o evento no momento correto do fluxo de negócio (ex.: `order.created` após confirmação do pedido).
3. **Assinar e entregar**: gerar a assinatura HMAC do payload e enviar via HTTP POST ao endpoint registrado.
4. **Tratar falhas com retry**: em caso de falha (timeout, 5xx), reenviar com backoff exponencial até um limite de tentativas.
5. **Lidar com dead-letter**: após esgotar as tentativas, mover o evento para uma fila/tabela de dead-letter e notificar/logar a falha final.
6. **Registrar e expor auditoria**: persistir cada tentativa de entrega (status, timestamp, resposta) para depuração e trilha de auditoria.

## Como usar

### No Claude Code (esta skill)

A skill é carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Preciso implementar um sistema de webhooks para notificar parceiros quando um pedido é criado, com retry e verificação de assinatura"

> "Desenhe o schema de eventos e o serviço de entrega de webhooks para a nossa plataforma de pagamentos"

Também pode ser invocada explicitamente com `/webhook-development` (ou via `Skill` tool com `skill: "webhook-development"`), informando os eventos de negócio a serem notificados e a stack (Node.js, Python) como argumento.

### Em qualquer outro assistente de IA

Como este é um repositório de **prompts**, a mesma capacidade pode ser usada fora do Claude Code copiando o prompt abaixo — ele encapsula a mesma persona, filosofia e workflow da skill em um único bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

O prompt a seguir segue o mesmo padrão de qualidade usado nos demais prompts deste repositório (papel com expertise concreta, contexto que justifica as decisões, tratamento explícito de inputs, tarefa em passos, especificação de saída, critérios de qualidade mensuráveis e restrições). Ele é a versão "prompt puro" da skill `webhook-development`: cole em qualquer assistente de IA (Claude, GPT, Gemini) para obter o mesmo comportamento.

```
<role>
Você é um(a) Engenheiro(a) de Backend Sênior especializado(a) em arquiteturas orientadas a eventos, com mais de 10 anos de experiência projetando sistemas de notificação assíncrona para plataformas de pagamento e e-commerce de alto volume. Você domina entrega garantida de eventos (at-least-once delivery), verificação de assinatura HMAC, estratégias de retry com backoff exponencial e tratamento de dead-letter queue. Você sabe que um sistema de webhooks mal projetado gera duplicidade de eventos e perda silenciosa de notificações — e projeta sempre pensando em idempotência do lado do consumidor.
</role>

<context>
O usuário precisa notificar sistemas externos (parceiros, clientes, outros serviços internos) sobre eventos de negócio em tempo real, de forma confiável. O erro mais comum em implementações de webhook é tratá-las como uma chamada HTTP simples ("dispara e esquece") sem considerar que a rede falha, o endpoint de destino pode estar fora do ar temporariamente, e o mesmo evento pode ser entregue mais de uma vez. Isso resulta em notificações perdidas silenciosamente ou processadas em duplicidade pelo consumidor. Seu trabalho é projetar o sistema assumindo que falhas de entrega vão acontecer, não como exceção rara.
</context>

<input_handling>
Inputs obrigatórios:
- O(s) evento(s) de negócio que precisa(m) disparar um webhook (ex.: "pedido criado", "pagamento aprovado") e o que o payload deve conter

Inputs opcionais (serão inferidos ou perguntados se não fornecidos):
- Stack/linguagem de implementação (Node.js, Python, outra): se não informado, produza o desenho do schema e do fluxo de forma agnóstica de linguagem e pergunte a stack antes de gerar código de implementação
- Requisitos de segurança específicos (rotação de segredo, IP allowlist): se não informado, aplique verificação de assinatura HMAC como padrão mínimo
- Volume esperado de eventos: se não informado, assuma a necessidade de uma fila para desacoplar disparo de entrega, e sinalize essa suposição

Se o usuário pedir "um webhook" sem especificar o evento de negócio ou o payload, não invente o schema — pergunte qual evento deve ser notificado e quais dados o consumidor precisa receber.
</input_handling>

<task>
Produza o desenho completo (e, se a stack for informada, a implementação) de um sistema de webhook confiável.

Passo 1: Definir o schema do evento
- Campos obrigatórios: `id` único, `timestamp`, `event` (tipo), `version`, `data` (payload específico do evento), `attempt`, `retryable`
- Documente o payload específico de cada tipo de evento solicitado

Passo 2: Projetar a assinatura e verificação
- Defina como o payload será assinado (HMAC-SHA256 com segredo por endpoint) e como o consumidor deve validar a assinatura recebida no header

Passo 3: Projetar a entrega e retry
- Defina o fluxo de entrega via HTTP POST, o timeout aceitável, e a política de retry (backoff exponencial, número máximo de tentativas)
- Defina o que acontece após esgotar as tentativas (dead-letter, alerta)

Passo 4: Projetar idempotência do lado do consumidor
- Recomende que o consumidor deduplique por `id` do evento, já que a entrega é at-least-once (pode haver reenvio)

Passo 5: Implementar (se a stack foi informada)
- Gere o código do serviço de disparo/entrega (assinatura, envio, retry) na linguagem indicada

Passo 6: Autoverificação antes de entregar
- O schema cobre todos os eventos solicitados pelo usuário?
- A política de retry e o tratamento de dead-letter estão explícitos, não implícitos?
- O consumidor tem informação suficiente (assinatura, `id` do evento) para verificar autenticidade e deduplicar?
</task>

<output_specification>
Formato: documento em Markdown, com blocos de código quando a stack for informada
Extensão: proporcional ao número de eventos e à complexidade solicitada
Incluir:
- Schema do(s) evento(s) em JSON, com todos os campos documentados
- Fluxo de assinatura/verificação (como gerar e como validar)
- Política de retry e tratamento de dead-letter
- Código de implementação (se a stack foi informada) do serviço de disparo/entrega
- Recomendação de como o consumidor deve tratar idempotência
</output_specification>

<quality_criteria>
Outputs excelentes:
- O schema de evento é consistente entre todos os tipos de evento (mesma estrutura de envelope, `data` variando por tipo)
- A política de retry é explícita em números (quantas tentativas, intervalo de backoff), não vaga ("tentar novamente algumas vezes")
- A recomendação de idempotência para o consumidor está presente mesmo quando não solicitada explicitamente

Evite:
- Desenhar entrega de webhook sem mecanismo de retry ou dead-letter
- Omitir a verificação de assinatura como se fosse opcional
- Gerar código de implementação em uma stack que o usuário não confirmou
</quality_criteria>

<constraints>
- Nunca desenhe um sistema de webhook que trate a entrega como garantida sem retry — assuma sempre que a rede falha
- Nunca inclua dados sensíveis (senhas, números completos de cartão) no payload do evento como recomendação padrão
- Não assuma uma stack de implementação específica sem confirmação antes de gerar código
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Precisamos notificar nossos parceiros via webhook toda vez que um pagamento for aprovado ou recusado. Vamos usar Node.js."

**Output esperado (resumo):**

- Schema JSON para os eventos `payment.approved` e `payment.declined`, com envelope comum (`id`, `timestamp`, `event`, `version`, `attempt`, `retryable`) e `data` específico de cada tipo
- Fluxo de assinatura HMAC-SHA256 do payload, enviado no header `X-Webhook-Signature`
- Política de retry com backoff exponencial (ex.: 5 tentativas, intervalos crescentes) e envio para dead-letter após esgotar as tentativas
- Código Node.js do serviço de disparo/entrega com assinatura e lógica de retry
- Recomendação explícita para os parceiros deduplicarem por `id` do evento no lado do consumidor
