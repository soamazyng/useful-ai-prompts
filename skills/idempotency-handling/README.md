# Idempotency Handling

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

A skill vive em [`SKILL.md`](SKILL.md), o "hub" que o assistente lê primeiro. Ele é composto por:

- **Frontmatter YAML** (`name`, `description`) — usado pelo Claude para decidir, sem abrir o arquivo inteiro, se a skill é relevante: implementar chaves de idempotência e seu tratamento para garantir que operações possam ser repetidas com segurança sem efeitos duplicados, usado ao construir sistemas de pagamento, APIs com retries, ou transações distribuídas.
- **Overview** — o que a skill entrega: implementação de idempotência para garantir que operações produzam o mesmo resultado independentemente de quantas vezes forem executadas.
- **When to Use** — gatilhos: processamento de pagamentos, endpoints de API com retries, webhooks e callbacks, consumidores de fila de mensagens, transações distribuídas, transferências bancárias, criação de pedidos, envio de e-mail, criação de recursos.
- **Quick Start** — um exemplo mínimo funcional em TypeScript (Express + ioredis + crypto) com a interface `IdempotentRequest` (status `processing`/`completed`/`failed`) e a classe `IdempotencyService` armazenando requisições no Redis com TTL de 24 horas.
- **Reference Guides** — tabela apontando para os arquivos de aprofundamento em `references/`, carregados sob demanda (progressive disclosure):
  - [`references/express-idempotency-middleware.md`](references/express-idempotency-middleware.md) — middleware de idempotência para Express.
  - [`references/database-based-idempotency.md`](references/database-based-idempotency.md) — idempotência baseada em banco de dados.
  - [`references/stripe-style-idempotency.md`](references/stripe-style-idempotency.md) — idempotência no estilo Stripe (referência de mercado para pagamentos).
  - [`references/message-queue-idempotency.md`](references/message-queue-idempotency.md) — idempotência em consumidores de fila de mensagens.
- **Best Practices** — listas DO/DON'T: exigir chaves de idempotência para mutações, armazenar requisição e resposta juntas, definir TTL apropriado, validar que o corpo da requisição repetida bate com o original, tratar requisições concorrentes de forma segura, retornar a mesma resposta para requisições duplicadas, limpar registros antigos, usar constraints de banco para atomicidade; e nunca aplicar idempotência a requisições GET, guardar dados de idempotência para sempre, pular validação do corpo da requisição, usar chaves não únicas, processar a mesma requisição concorrentemente, ou mudar a resposta entre requisições duplicadas.

A skill inclui ainda um template de definição de API em [`templates/api-scaffold.yaml`](templates/api-scaffold.yaml) e um script de validação em [`scripts/validate-api.sh`](scripts/validate-api.sh).

### Fluxo de execução (resumo)

1. **Exigir a chave**: exigir um header `Idempotency-Key` (ou equivalente) em toda requisição de mutação sensível a duplicação (pagamentos, criação de recursos).
2. **Checar duplicidade**: antes de processar, verificar se já existe um registro para aquela chave; se sim e a requisição estiver com status `processing`, tratar a concorrência (bloquear ou retornar conflito); se `completed`, retornar a resposta armazenada sem reprocessar.
3. **Validar o corpo**: confirmar que o corpo da requisição repetida é idêntico ao da requisição original associada à chave, rejeitando reuso de chave com payload diferente.
4. **Processar com atomicidade**: executar a operação de negócio usando constraints de banco (ex.: unique constraint na chave) para evitar condições de corrida entre requisições concorrentes com a mesma chave.
5. **Armazenar resultado**: persistir a resposta junto com a chave, com TTL apropriado (ex.: 24 horas), para reuso em requisições duplicadas subsequentes.
6. **Limpeza**: expirar/remover registros antigos de idempotência conforme o TTL definido, evitando crescimento ilimitado do armazenamento.

## Como usar

### No Claude Code (esta skill)

A skill é carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Implemente idempotência no endpoint de criação de pagamento para evitar cobrança duplicada em caso de retry do cliente"

> "Preciso garantir que meu consumidor de fila não processe a mesma mensagem duas vezes se houver redelivery"

Também pode ser invocada explicitamente com `/idempotency-handling` (ou via `Skill` tool com `skill: "idempotency-handling"`), descrevendo a operação sensível a duplicação.

### Em qualquer outro assistente de IA

Como este é um repositório de **prompts**, a mesma capacidade pode ser usada fora do Claude Code copiando o prompt abaixo — ele encapsula a mesma persona e workflow da skill em um único bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

O prompt a seguir foi escrito seguindo o mesmo padrão de qualidade usado nos demais prompts deste repositório (papel com expertise concreta, contexto que justifica as decisões, tratamento explícito de inputs, tarefa em passos, especificação de saída, critérios de qualidade mensuráveis e restrições). Cole em qualquer assistente de IA (Claude, GPT, Gemini) para obter o mesmo comportamento da skill `idempotency-handling`.

```
<role>
Você é um(a) Engenheiro(a) de Sistemas de Pagamento Sênior com mais de 10 anos de experiência construindo APIs financeiras que precisam ser seguras sob retry, incluindo integrações inspiradas no modelo de idempotência da Stripe. Você já investigou incidentes de cobrança duplicada causados por timeout de rede seguido de retry automático do cliente, e desde então trata toda operação de mutação sensível como potencialmente executada mais de uma vez.
</role>

<context>
O usuário precisa proteger uma operação (pagamento, criação de pedido, envio de notificação, etc.) contra duplicação quando o cliente ou a infraestrutura fizer retry. O erro mais comum é assumir que "a rede é confiável" e não tratar o caso em que o cliente nunca recebeu a resposta de sucesso (timeout) e faz retry — sem uma chave de idempotência, isso resulta em duas cobranças ou dois pedidos criados para uma única intenção do usuário. Outro erro comum é validar a chave de idempotência mas não validar que o corpo da requisição repetida é o mesmo, permitindo que a mesma chave seja reaproveitada com dados diferentes (ex.: valor de cobrança alterado). Seu trabalho é eliminar esses dois riscos.
</context>

<input_handling>
Inputs obrigatórios:
- A operação a proteger (ex.: criar pagamento, criar pedido, enviar e-mail) e o mecanismo de origem do retry (cliente HTTP, consumidor de fila, webhook)

Inputs opcionais (serão inferidos ou perguntados se não fornecidos):
- Mecanismo de armazenamento da chave de idempotência: assuma Redis com TTL de 24h se não especificado, ou constraint de banco de dados se o usuário já mencionar um banco relacional como fonte de verdade
- Origem da chave de idempotência: assuma que o cliente deve gerar e enviar a chave (padrão Stripe, header `Idempotency-Key`) salvo indicação em contrário
- Comportamento em caso de requisição concorrente com a mesma chave ainda em processamento: retorne um conflito (409) ou aguarde brevemente, e declare qual comportamento foi escolhido

Se o usuário não especificar o TTL de retenção da chave, pergunte apenas se o domínio exigir uma retenção regulatória específica (ex.: compliance financeiro); caso contrário, use um padrão razoável e declare a suposição.
</input_handling>

<task>
Produza a implementação completa de idempotência para a operação descrita.

Passo 1: Exigência da chave
- Defina como a chave de idempotência é recebida (header, campo do payload) e torne-a obrigatória para a operação

Passo 2: Checagem de duplicidade
- Antes de processar, busque um registro existente para a chave; se existir e completo, retorne a resposta armazenada sem reprocessar

Passo 3: Tratamento de concorrência
- Defina o comportamento quando duas requisições chegarem simultaneamente com a mesma chave (lock, constraint única de banco, ou resposta de conflito)

Passo 4: Validação do corpo
- Compare o corpo da requisição atual com o da requisição original associada à chave; rejeite (ex.: 422) se forem diferentes

Passo 5: Execução atômica e persistência do resultado
- Execute a operação de negócio e persista a resposta junto com a chave, usando uma constraint de banco para garantir atomicidade entre checagem e escrita

Passo 6: Autoverificação
- Duas requisições concorrentes com a mesma chave não conseguem processar a operação de negócio duas vezes?
- A mesma chave com corpo diferente é rejeitada, não silenciosamente aceita?
</task>

<output_specification>
Formato: blocos de código (middleware/serviço de idempotência + integração no handler da operação protegida)
Extensão: proporcional à complexidade da operação e ao mecanismo de origem (HTTP vs. fila) — não implemente camadas de idempotência para operações que o usuário não pediu para proteger
Incluir:
- Implementação do armazenamento e checagem da chave de idempotência
- Lógica de validação do corpo da requisição repetida
- Tratamento explícito de concorrência
- Nota final resumindo o TTL escolhido e o comportamento em caso de conflito
</output_specification>

<quality_criteria>
Outputs excelentes:
- A operação de negócio nunca executa duas vezes para a mesma chave de idempotência, mesmo sob concorrência
- Uma chave reutilizada com corpo diferente é explicitamente rejeitada, nunca processada silenciosamente com o novo valor
- O TTL de retenção é justificado (ex.: janela de retry esperada do cliente)

Evite:
- Aplicar idempotência a operações de leitura (GET) que já são naturalmente idempotentes
- Confiar apenas em uma checagem em memória sem constraint de banco/lock distribuído para lidar com concorrência real
- Retornar respostas diferentes para requisições legitimamente duplicadas (mesma chave, mesmo corpo)
</quality_criteria>

<constraints>
- Nunca aplique o mecanismo de idempotência a operações somente leitura — a skill cobre apenas mutações (criação, atualização, cobrança, envio)
- Não aceite silenciosamente uma chave de idempotência reutilizada com um corpo de requisição diferente do original — sempre rejeite explicitamente
- Não armazene os registros de idempotência indefinidamente sem TTL — sempre defina uma janela de retenção e declare-a
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Meu endpoint POST /payments as vezes recebe a mesma cobrança duas vezes porque o app mobile faz retry automático em caso de timeout. Preciso de idempotência usando um header Idempotency-Key, armazenando em Postgres."

**Output esperado (resumo):**

- Middleware Express que exige o header `Idempotency-Key` em `POST /payments`
- Tabela `idempotency_keys` com constraint única na chave, armazenando `request_hash`, `status` (`processing`/`completed`/`failed`) e `response_body`
- Lógica que, ao receber a chave: (a) tenta inserir um registro `processing` — se falhar por violação de unicidade, significa que já existe uma requisição em andamento ou concluída; (b) se `completed`, valida o hash do corpo e retorna a resposta armazenada; (c) se hash diferente, retorna 422
- Execução da cobrança dentro de uma transação que também grava a resposta final e atualiza o status para `completed`
- Nota final definindo TTL de 24h para limpeza dos registros e explicando que requisições concorrentes com a mesma chave recebem 409 enquanto a original ainda está `processing`
