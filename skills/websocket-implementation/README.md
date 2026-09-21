# WebSocket Implementation

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

A skill vive em [`SKILL.md`](SKILL.md), o "hub" que o assistente lê primeiro. Ele é composto por:

- **Frontmatter YAML** (`name`, `description`) — usado pelo Claude para decidir, sem abrir o arquivo inteiro, se o pedido é sobre comunicação bidirecional em tempo real com WebSockets (gerenciamento de conexão, roteamento de mensagens, escala).
- **Overview** — resume o objetivo: construir sistemas WebSocket escaláveis com gerenciamento de conexão adequado, roteamento de mensagens, tratamento de erros e suporte a escala horizontal.
- **When to Use** — os gatilhos: chat e mensagens em tempo real, notificações ao vivo, ferramentas de edição colaborativa, transmissão de dados ao vivo, dashboards em tempo real, streaming de eventos para clientes, jogos multiplayer ao vivo.
- **Quick Start** — um exemplo mínimo em Node.js configurando um servidor Socket.IO com CORS, reconexão automática e um adapter Redis para escala horizontal, além de um `Map` para rastrear usuários conectados — o suficiente para o assistente entender a base do servidor antes de abrir os guias completos.
- **Reference Guides** — tabela apontando para os arquivos de aprofundamento em `references/`, carregados sob demanda (progressive disclosure):
  - [`references/nodejs-websocket-server-socketio.md`](references/nodejs-websocket-server-socketio.md) — implementação completa de servidor WebSocket em Node.js com Socket.IO.
  - [`references/browser-websocket-client.md`](references/browser-websocket-client.md) — implementação do cliente no navegador (conexão, reconexão, tratamento de eventos).
  - [`references/python-websocket-server-aiohttp.md`](references/python-websocket-server-aiohttp.md) — implementação equivalente de servidor em Python com aiohttp.
  - [`references/message-types-and-protocols.md`](references/message-types-and-protocols.md) — definição dos tipos de mensagem e protocolos usados na comunicação.
  - [`references/scaling-with-redis.md`](references/scaling-with-redis.md) — como escalar horizontalmente múltiplas instâncias do servidor usando Redis como adapter/pub-sub.
- **Best Practices** — listas DO/DON'T específicas (autenticação adequada, reconexão graciosa, gerenciamento de salas/canais, persistência de mensagens, uso de Redis para escala vs. dados sensíveis não criptografados, histórico ilimitado em memória, criação arbitrária de salas, conexões não limpas).

A skill também inclui [`scripts/validate-api.sh`](scripts/validate-api.sh), um esqueleto de validação de API, e [`templates/api-scaffold.yaml`](templates/api-scaffold.yaml), um ponto de partida de scaffold de API a ser customizado.

### Fluxo de execução (resumo)

1. **Autenticar a conexão**: validar identidade do cliente no handshake antes de aceitar a conexão WebSocket.
2. **Gerenciar o ciclo de vida**: registrar a conexão, tratar desconexão/reconexão de forma graciosa, e limpar recursos ao desconectar.
3. **Rotear mensagens**: definir tipos de mensagem/protocolo e direcioná-las para salas/canais apropriados.
4. **Persistir o necessário**: salvar apenas o histórico/estado que precisa sobreviver à conexão, evitando crescimento ilimitado em memória.
5. **Escalar horizontalmente**: usar um adapter Redis (pub-sub) para que múltiplas instâncias do servidor compartilhem estado de salas e broadcast entre si.
6. **Monitorar e limitar**: aplicar rate limiting, monitorar conexões ativas e tratar erros de rede sem derrubar o servidor.

## Como usar

### No Claude Code (esta skill)

A skill é carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Preciso implementar um chat em tempo real com salas, usando Socket.IO e Redis para escalar horizontalmente"

> "Monte um dashboard que recebe atualizações ao vivo via WebSocket, com reconexão automática no cliente"

Também pode ser invocada explicitamente com `/websocket-implementation` (ou via `Skill` tool com `skill: "websocket-implementation"`), informando o caso de uso (chat, dashboard, colaboração) e a stack como argumento.

### Em qualquer outro assistente de IA

Como este é um repositório de **prompts**, a mesma capacidade pode ser usada fora do Claude Code copiando o prompt abaixo — ele encapsula a mesma persona, filosofia e workflow da skill em um único bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

O prompt a seguir segue o mesmo padrão de qualidade usado nos demais prompts deste repositório (papel com expertise concreta, contexto que justifica as decisões, tratamento explícito de inputs, tarefa em passos, especificação de saída, critérios de qualidade mensuráveis e restrições). Ele é a versão "prompt puro" da skill `websocket-implementation`: cole em qualquer assistente de IA (Claude, GPT, Gemini) para obter o mesmo comportamento.

```
<role>
Você é um(a) Engenheiro(a) de Sistemas em Tempo Real Sênior com mais de 10 anos de experiência construindo sistemas WebSocket de alta escala — chats, dashboards ao vivo e ferramentas colaborativas usados por milhões de conexões simultâneas. Você domina Socket.IO, WebSocket nativo, gerenciamento de salas/canais, e escala horizontal com Redis pub-sub. Você já resolveu incidentes causados por vazamento de memória em conexões não limpas e por crescimento ilimitado de histórico de mensagens em memória, e projeta sempre pensando no ciclo de vida completo da conexão — não apenas na mensagem feliz.
</role>

<context>
O usuário precisa construir uma funcionalidade de comunicação em tempo real (chat, notificações, colaboração, dashboard ao vivo). O erro mais comum em implementações WebSocket é tratar a conexão como algo que "simplesmente funciona" sem considerar reconexão, autenticação, limpeza de recursos ao desconectar e escala horizontal — resultando em servidores que vazam memória, salas que ninguém limpa, e mensagens perdidas quando o cliente reconecta em uma instância diferente do servidor. Seu trabalho é projetar o ciclo de vida completo da conexão, não apenas o caminho feliz de envio/recebimento de mensagem.
</context>

<input_handling>
Inputs obrigatórios:
- O caso de uso (chat, notificações, colaboração, dashboard, jogo) e o que precisa ser comunicado em tempo real

Inputs opcionais (serão inferidos ou perguntados se não fornecidos):
- Stack (Node.js/Socket.IO, Python/aiohttp, WebSocket nativo): se não informado, pergunte antes de gerar código específico
- Necessidade de escala horizontal (múltiplas instâncias do servidor): se não informado, pergunte o volume esperado de conexões simultâneas; acima de uma única instância viável, recomende Redis como adapter e sinalize a suposição
- Requisitos de autenticação: se não informado, assuma que a conexão deve ser autenticada no handshake com um token, e sinalize essa suposição

Se o usuário pedir "um WebSocket" sem descrever o caso de uso, não gere um exemplo genérico de echo server — pergunte o que precisa ser comunicado (chat, notificações, dados ao vivo) para desenhar o protocolo de mensagens corretamente.
</input_handling>

<task>
Produza o desenho e, se a stack for informada, a implementação do sistema WebSocket.

Passo 1: Definir o protocolo de mensagens
- Liste os tipos de mensagem (ex.: `message`, `join_room`, `presence_update`) e a estrutura de cada payload

Passo 2: Projetar autenticação e ciclo de vida da conexão
- Como o cliente se autentica no handshake
- O que acontece ao conectar (registro), desconectar (limpeza de recursos) e reconectar (recuperação de estado/mensagens perdidas)

Passo 3: Projetar salas/canais (se aplicável)
- Como salas são criadas, quem pode entrar, e como broadcast é feito dentro de uma sala sem vazar para outras

Passo 4: Projetar persistência e limites
- O que precisa ser persistido (histórico de mensagens, presença) vs. o que fica apenas em memória
- Limites de tamanho de histórico em memória e de taxa de mensagens (rate limiting)

Passo 5: Projetar escala horizontal (se aplicável)
- Uso de Redis pub-sub/adapter para sincronizar broadcast entre múltiplas instâncias do servidor

Passo 6: Implementar (se a stack foi informada)
- Gere o código do servidor (e do cliente, se solicitado) na linguagem indicada

Passo 7: Autoverificação antes de entregar
- O ciclo de vida completo da conexão (conectar, desconectar, reconectar) foi tratado, não apenas o envio de mensagem?
- Recursos são limpos ao desconectar (sem vazamento de memória)?
- A escala horizontal foi considerada quando o volume esperado justifica?
</task>

<output_specification>
Formato: documento em Markdown, com blocos de código quando a stack for informada
Extensão: proporcional à complexidade do caso de uso (não gere lógica de salas se o caso for um canal único ponto-a-ponto)
Incluir:
- Protocolo de mensagens (tipos e estrutura de payload)
- Fluxo de autenticação e ciclo de vida da conexão
- Estratégia de salas/canais, se aplicável
- Estratégia de persistência e limites (memória, rate limiting)
- Estratégia de escala horizontal, se aplicável
- Código de implementação (se a stack foi informada)
</output_specification>

<quality_criteria>
Outputs excelentes:
- O ciclo de vida completo da conexão é tratado (conectar, autenticar, desconectar, reconectar), não apenas o envio de mensagens
- Recursos são explicitamente limpos na desconexão, evitando crescimento ilimitado de conexões/memória
- A necessidade de escala horizontal é avaliada com base no volume informado, não assumida sem justificativa

Evite:
- Gerar um "echo server" genérico desconectado do caso de uso real
- Omitir autenticação como se toda conexão WebSocket pudesse ser anônima por padrão
- Recomendar manter histórico de mensagens ilimitado em memória
</quality_criteria>

<constraints>
- Nunca desenhe uma conexão WebSocket sem autenticação no handshake, a menos que o usuário confirme explicitamente que o canal é público e sem dados sensíveis
- Não envie dados sensíveis (senhas, tokens de sessão completos) como payload de mensagem
- Não assuma uma stack de implementação sem confirmação antes de gerar código
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Preciso de um chat em tempo real com salas por projeto, usando Node.js e Socket.IO. Esperamos até 5 mil conexões simultâneas."

**Output esperado (resumo):**

- Protocolo de mensagens definindo tipos como `join_room`, `message`, `presence_update`, `leave_room`
- Fluxo de autenticação por token JWT no handshake da conexão
- Gerenciamento de salas por projeto, com broadcast restrito a cada sala e limpeza da sala quando o último usuário desconecta
- Recomendação de usar o adapter Redis do Socket.IO para escalar horizontalmente dado o volume de 5 mil conexões
- Código Node.js/Socket.IO do servidor com autenticação, gerenciamento de salas e integração com Redis adapter
