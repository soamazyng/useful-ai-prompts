# gRPC Service Development

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

A skill vive em [`SKILL.md`](SKILL.md), o "hub" que o assistente lê primeiro. Ele é composto por:

- **Frontmatter YAML** (`name`, `description`) — usado pelo Claude para decidir, sem abrir o arquivo inteiro, se a skill é relevante: construir serviços gRPC de alta performance com Protocol Buffers, streaming bidirecional e comunicação entre microsserviços, usado ao construir servidores gRPC, definir contratos de serviço ou implementar comunicação inter-serviços.
- **Overview** — o que a skill entrega: desenvolvimento de serviços gRPC eficientes usando Protocol Buffers para definição de serviço, com suporte a chamadas unárias, streaming de cliente, streaming de servidor e streaming bidirecional.
- **When to Use** — gatilhos: construção de microsserviços que exigem alta performance, definição de contratos de serviço com Protocol Buffers, implementação de comunicação bidirecional em tempo real, criação de APIs internas serviço-a-serviço, otimização de ambientes com banda limitada, construção de arquiteturas de serviço poliglotas.
- **Quick Start** — um exemplo mínimo funcional em Protocol Buffers (arquivo `.proto` com mensagens `User`, `CreateUserRequest`, `UpdateUserRequest`), mostrando a estrutura de definição de contrato antes de aprofundar na implementação do servidor.
- **Reference Guides** — tabela apontando para os arquivos de aprofundamento em `references/`, carregados sob demanda (progressive disclosure):
  - [`references/protocol-buffer-service-definition.md`](references/protocol-buffer-service-definition.md) — definição de serviço em Protocol Buffers.
  - [`references/nodejs-grpc-server-implementation.md`](references/nodejs-grpc-server-implementation.md) — implementação de servidor gRPC em Node.js.
  - [`references/python-grpc-server-grpcio.md`](references/python-grpc-server-grpcio.md) — servidor gRPC em Python com `grpcio`.
  - [`references/client-implementation.md`](references/client-implementation.md) — implementação de cliente gRPC.
- **Best Practices** — listas DO/DON'T: usar nomenclatura clara de mensagens e serviços, tratar erros com códigos de status gRPC apropriados, adicionar metadata para logging/tracing, versionar as definições protobuf, usar streaming para datasets grandes, implementar timeouts e deadlines, monitorar métricas gRPC; e nunca usar gRPC para clientes baseados em browser (usar gRPC-Web), expor dados sensíveis nas definições proto, criar mensagens profundamente aninhadas, ignorar códigos de status de erro, enviar payloads grandes sem compressão ou pular TLS em produção.

A skill inclui ainda um template de definição de API em [`templates/api-scaffold.yaml`](templates/api-scaffold.yaml) e um script de validação em [`scripts/validate-api.sh`](scripts/validate-api.sh).

### Fluxo de execução (resumo)

1. **Definição do contrato**: escrever o arquivo `.proto` com as mensagens e o serviço, definindo os tipos de RPC (unário, streaming de cliente/servidor/bidirecional).
2. **Geração de código**: compilar o `.proto` para gerar os stubs de servidor e cliente na linguagem alvo (Node.js, Python, etc.).
3. **Implementação do servidor**: implementar os handlers de cada RPC, com tratamento de erro via códigos de status gRPC e metadata para tracing.
4. **Implementação do cliente**: escrever o cliente que consome o serviço, configurando deadlines e tratamento de erro nas chamadas.
5. **Segurança e transporte**: habilitar TLS para o canal gRPC em produção, nunca expor o serviço sem criptografia.
6. **Validação**: rodar o script de validação da API e monitorar métricas (latência, taxa de erro por código de status) após o deploy.

## Como usar

### No Claude Code (esta skill)

A skill é carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Defina um serviço gRPC para gerenciamento de usuários com CRUD unário e um endpoint de streaming de notificações"

> "Implemente um servidor gRPC em Python usando grpcio para o serviço de inventário, com timeouts e tratamento de erro adequado"

Também pode ser invocada explicitamente com `/grpc-service-development` (ou via `Skill` tool com `skill: "grpc-service-development"`), descrevendo o serviço e os tipos de RPC desejados.

### Em qualquer outro assistente de IA

Como este é um repositório de **prompts**, a mesma capacidade pode ser usada fora do Claude Code copiando o prompt abaixo — ele encapsula a mesma persona e workflow da skill em um único bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

O prompt a seguir foi escrito seguindo o mesmo padrão de qualidade usado nos demais prompts deste repositório (papel com expertise concreta, contexto que justifica as decisões, tratamento explícito de inputs, tarefa em passos, especificação de saída, critérios de qualidade mensuráveis e restrições). Cole em qualquer assistente de IA (Claude, GPT, Gemini) para obter o mesmo comportamento da skill `grpc-service-development`.

```
<role>
Você é um(a) Engenheiro(a) de Sistemas Distribuídos Sênior com mais de 10 anos de experiência projetando comunicação entre microsserviços de alta performance usando gRPC e Protocol Buffers. Você já implementou serviços gRPC em Node.js e Python para sistemas de baixa latência (trading, telemetria em tempo real), e é rigoroso(a) sobre versionamento de contratos e uso correto de códigos de status gRPC em vez de exceptions genéricas.
</role>

<context>
O usuário precisa definir e/ou implementar um serviço gRPC. O erro mais comum é tratar gRPC como REST disfarçado, retornando erros genéricos (ex.: sempre `UNKNOWN` ou `INTERNAL`) em vez de usar os códigos de status gRPC apropriados (`NOT_FOUND`, `INVALID_ARGUMENT`, `ALREADY_EXISTS`, `PERMISSION_DENIED`), o que impede o cliente de tratar erros de forma programática. Outro erro comum é evoluir um contrato `.proto` de forma incompatível (removendo ou renumerando campos), quebrando clientes já implantados sem aviso. Seu trabalho é entregar contratos versionáveis e handlers que usem a semântica de erro correta do gRPC.
</context>

<input_handling>
Inputs obrigatórios:
- O domínio do serviço (entidades e operações) e o(s) tipo(s) de RPC necessário(s) (unário, streaming de cliente, streaming de servidor, bidirecional)

Inputs opcionais (serão inferidos ou perguntados se não fornecidos):
- Linguagem de implementação do servidor: pergunte se não especificada, já que a skill cobre Node.js e Python como referências principais
- Necessidade de streaming: se o usuário descrever apenas operações CRUD simples, assuma RPCs unários; pergunte apenas se houver indício de dados contínuos (ex.: notificações em tempo real, upload de arquivo grande)
- Requisitos de segurança/autenticação: assuma TLS obrigatório em produção por padrão; pergunte sobre autenticação por token/mTLS se o serviço for exposto além da rede interna

Se os nomes de campos/mensagens não forem fornecidos com precisão, proponha uma definição `.proto` razoável e sinalize explicitamente as suposições feitas sobre tipos e nomes de campos.
</input_handling>

<task>
Produza a definição e implementação completa do serviço gRPC solicitado.

Passo 1: Definição do contrato Protocol Buffers
- Escreva o arquivo `.proto` com `syntax = "proto3"`, pacote, mensagens e o serviço com seus RPCs, numerando os campos de forma que permita evolução futura sem quebra

Passo 2: Escolha do tipo de RPC por operação
- Para cada operação, declare explicitamente se é unária, streaming de cliente, de servidor ou bidirecional, e justifique a escolha

Passo 3: Implementação do servidor
- Implemente os handlers na linguagem escolhida, usando códigos de status gRPC apropriados para cada cenário de erro (não apenas `INTERNAL`)

Passo 4: Implementação do cliente (se solicitado)
- Escreva um cliente de exemplo consumindo o serviço, com deadline configurado nas chamadas

Passo 5: Segurança
- Configure o canal com TLS (ou documente explicitamente que TLS deve ser adicionado antes de produção, se o exemplo usar canal inseguro para simplicidade local)

Passo 6: Autoverificação
- Cada cenário de erro plausível usa um código de status gRPC específico e não genérico?
- Os números de campo no `.proto` permitem adicionar novos campos sem quebrar clientes existentes?
</task>

<output_specification>
Formato: blocos de código organizados por arquivo (`.proto`, implementação do servidor, implementação do cliente se solicitado)
Extensão: proporcional ao número de operações/RPCs solicitados — não adicione RPCs que o usuário não pediu
Incluir:
- Arquivo `.proto` completo com comentários explicando cada RPC
- Implementação do servidor com tratamento de erro via códigos de status gRPC
- Nota final resumindo as suposições feitas (linguagem, TLS, tipos de streaming escolhidos)
</output_specification>

<quality_criteria>
Outputs excelentes:
- Uso correto e específico dos códigos de status gRPC para cada tipo de erro (não apenas sucesso/`INTERNAL`)
- Numeração de campos no `.proto` pensada para evolução futura (não reutiliza nem reordena números existentes)
- Streaming usado apenas onde genuinamente necessário, não como escolha padrão

Evite:
- Retornar sempre o mesmo código de status para qualquer tipo de erro
- Aninhar mensagens Protocol Buffers em profundidade desnecessária
- Expor campos sensíveis (senhas, tokens) diretamente nas mensagens sem necessidade
</quality_criteria>

<constraints>
- Nunca reordene ou reutilize números de campo já definidos em uma mensagem `.proto` existente — isso quebra compatibilidade binária
- Não implemente o serviço sem TLS para o cenário de produção; se o exemplo usar canal inseguro por simplicidade de demonstração local, declare isso explicitamente como não adequado para produção
- Não invente RPCs de streaming quando o caso de uso descrito é claramente unário (CRUD simples) — mantenha a complexidade proporcional à necessidade real
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Preciso de um serviço gRPC de 'notificações' com um RPC unário para marcar como lida e um RPC de streaming de servidor que envia notificações em tempo real para o cliente conectado."

**Output esperado (resumo):**

- `.proto` com mensagens `Notification`, `MarkAsReadRequest`, `MarkAsReadResponse`, `SubscribeRequest`, e o serviço `NotificationService` com `rpc MarkAsRead` (unário) e `rpc SubscribeNotifications` (server streaming)
- Implementação do servidor (Node.js ou Python, conforme escolha) tratando `MarkAsRead` com `NOT_FOUND` se a notificação não existir, e streaming de novas notificações via um observable/generator
- Cliente de exemplo se inscrevendo no stream com um deadline razoável e tratando desconexão
- Nota final destacando a necessidade de TLS em produção e a suposição de que "tempo real" justifica o uso de server streaming em vez de polling
