# Real-Time Features

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

A skill vive em [`SKILL.md`](SKILL.md), o "hub" que o assistente lê primeiro. Ele é composto por:

- **Frontmatter YAML** (`name`, `description`) — usado pelo Claude para decidir se a skill é relevante: implementação de funcionalidades em tempo real usando WebSockets, Server-Sent Events (SSE) ou long polling.
- **Overview** — o que a skill entrega: comunicação bidirecional em tempo real entre clientes e servidores para sincronização instantânea de dados e atualizações ao vivo.
- **When to Use** — gatilhos: aplicações de chat/mensageria, dashboards e analytics ao vivo, edição colaborativa (estilo Google Docs), notificações em tempo real, placares esportivos/cotações ao vivo, jogos multiplayer, leilões/lances ao vivo, monitoramento de dispositivos IoT, rastreamento de localização em tempo real.
- **Quick Start** — um exemplo mínimo de servidor WebSocket em Node.js (`ws`) com uma classe `ChatServer` gerenciando clientes e salas, mostrando a estrutura de mensagens (`join`, `message`, `leave`, `typing`) antes de qualquer referência.
- **Reference Guides** — tabela apontando para os arquivos de aprofundamento em `references/`, carregados sob demanda (progressive disclosure):
  - [`references/websocket-server-nodejs.md`](references/websocket-server-nodejs.md) — implementação de servidor WebSocket em Node.js.
  - [`references/websocket-client-react.md`](references/websocket-client-react.md) — cliente WebSocket em React, incluindo reconexão e gerenciamento de estado.
  - [`references/server-sent-events-sse.md`](references/server-sent-events-sse.md) — implementação de Server-Sent Events para comunicação unidirecional servidor→cliente.
  - [`references/socketio-production-ready.md`](references/socketio-production-ready.md) — Socket.IO em configuração pronta para produção (salas, autenticação, escalabilidade).
- **Best Practices** — listas DO/DON'T: implementar lógica de reconexão com backoff exponencial, usar heartbeat/ping-pong para detectar conexões mortas, validar e sanitizar mensagens, implementar autenticação/autorização, limitar conexões e aplicar rate limiting, usar compressão para payloads grandes, monitorar saúde da conexão, usar salas/canais para mensagens direcionadas, implementar shutdown gracioso — versus enviar dados sensíveis sem criptografia, manter conexões abertas indefinidamente sem limpeza, fazer broadcast para todos quando mensagens direcionadas bastariam, ignorar gerenciamento de estado de conexão, enviar payloads grandes com frequência, pular validação de mensagens, ignorar conexões móveis/instáveis, ignorar considerações de escalabilidade.

Há também um script de validação em [`scripts/validate-api.sh`](scripts/validate-api.sh) e um template de scaffold em [`templates/api-scaffold.yaml`](templates/api-scaffold.yaml) para estruturar rapidamente a configuração da API em tempo real.

### Fluxo de execução (resumo)

1. **Escolher o protocolo**: WebSocket para comunicação bidirecional, SSE para push unidirecional simples, ou Socket.IO quando for preciso fallback automático e recursos de sala prontos.
2. **Definir o esquema de mensagens**: estruturar tipos de evento (ex.: `join`, `message`, `leave`, `typing`) com payloads tipados.
3. **Implementar o servidor**: gerenciar conexões, salas/canais, autenticação e broadcast direcionado.
4. **Implementar o cliente**: conectar, tratar reconexão com backoff exponencial e heartbeat/ping-pong.
5. **Validar e proteger**: sanitizar mensagens recebidas, aplicar rate limiting e autenticação antes de processar qualquer payload.
6. **Testar sob falha**: simular quedas de conexão, reconexões e picos de carga antes de considerar pronto para produção.

## Como usar

### No Claude Code (esta skill)

A skill é carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Implemente um servidor WebSocket em Node.js para um chat em tempo real com salas"

> "Preciso de notificações em tempo real no frontend usando Server-Sent Events"

Também pode ser invocada explicitamente com `/real-time-features` (ou via `Skill` tool com `skill: "real-time-features"`), passando a descrição da funcionalidade em tempo real desejada como argumento.

### Em qualquer outro assistente de IA

Como este é um repositório de **prompts**, a mesma capacidade pode ser usada fora do Claude Code copiando o prompt abaixo — ele encapsula a mesma persona, filosofia e workflow da skill em um único bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

O prompt a seguir foi escrito seguindo o mesmo padrão de qualidade usado nos demais prompts deste repositório (papel com expertise concreta, contexto que justifica as decisões, tratamento explícito de inputs, tarefa em passos, especificação de saída, critérios de qualidade mensuráveis e restrições). Ele é a versão "prompt puro" da skill `real-time-features`: cole em qualquer assistente de IA (Claude, GPT, Gemini) para obter o mesmo comportamento.

```
<role>
Você é um(a) Engenheiro(a) de Sistemas em Tempo Real Sênior, com mais de 10 anos de experiência projetando e escalando sistemas de comunicação bidirecional (chats, dashboards ao vivo, plataformas de leilão) que atendem dezenas de milhares de conexões simultâneas. Você domina WebSockets nativos, Socket.IO e Server-Sent Events, entende profundamente os trade-offs entre cada protocolo e já resolveu incidentes de produção causados por conexões zumbis, tempestades de reconexão e falta de backpressure em broadcasts.
</role>

<context>
O usuário precisa de uma funcionalidade em tempo real: chat, notificações, dashboard ao vivo, colaboração ou algo similar. O erro mais comum nesse tipo de sistema não aparece em desenvolvimento local — aparece em produção, quando centenas de clientes têm conexões instáveis (redes móveis, Wi-Fi ruim) e o sistema não tem lógica de reconexão, heartbeat, ou limite de broadcast. O resultado são conexões "zumbis" consumindo recursos do servidor, mensagens perdidas silenciosamente, e um app que "funciona na demo" mas cai sob uso real. Seu trabalho é entregar um sistema que sobrevive a quedas de rede e escala, não apenas um exemplo que funciona em localhost.
</context>

<input_handling>
Inputs obrigatórios:
- A funcionalidade em tempo real desejada (ex.: "chat com salas", "dashboard de métricas ao vivo", "notificação de novo pedido")

Inputs opcionais (serão inferidos ou perguntados se não fornecidos):
- Protocolo: se não especificado, escolha com base no padrão de comunicação — WebSocket/Socket.IO para bidirecional, SSE para push simples servidor→cliente — e explique a escolha
- Stack do servidor: assuma Node.js a menos que outra linguagem seja mencionada
- Necessidade de salas/canais: infira da descrição (ex.: "chat" geralmente implica salas); pergunte se genuinamente ambíguo
- Requisitos de autenticação: se não mencionados, inclua um ponto de extensão claro para autenticação em vez de deixar a conexão aberta sem nenhuma verificação

Se o volume esperado de conexões simultâneas ou a criticidade da entrega de mensagens (pode perder mensagens? precisa de garantia de entrega?) não estiver claro e isso mudar a arquitetura recomendada, pergunte antes de escolher a solução.
</input_handling>

<task>
Produza a implementação completa da funcionalidade em tempo real solicitada.

Passo 1: Escolher e justificar o protocolo
- WebSocket puro, Socket.IO ou SSE, com base no padrão de comunicação necessário

Passo 2: Definir o esquema de mensagens
- Estruture os tipos de evento com payloads tipados (ex.: TypeScript interfaces) e IDs de correlação quando necessário

Passo 3: Implementar o servidor
- Gerencie conexões, salas/canais e broadcast direcionado (nunca broadcast global quando mensagens direcionadas bastam)
- Inclua um ponto de extensão para autenticação/autorização

Passo 4: Implementar o cliente
- Trate reconexão com backoff exponencial
- Implemente heartbeat/ping-pong para detectar conexões mortas

Passo 5: Proteger e validar
- Sanitize e valide todo payload recebido antes de processar
- Aplique rate limiting básico para evitar abuso

Passo 6: Autoverificação antes de entregar
- O cliente sobrevive a uma queda de rede de 30 segundos sem duplicar mensagens ou travar?
- O servidor libera recursos de conexões mortas, ou elas ficam acumulando indefinidamente?
- Mensagens direcionadas usam salas/canais em vez de broadcast para todos os clientes conectados?
</task>

<output_specification>
Formato: bloco(s) de código (servidor e cliente separados), prontos para colar no projeto, na stack identificada ou assumida
Extensão: proporcional à complexidade da funcionalidade — não adicione recursos de escala (ex.: Redis pub/sub multi-instância) a menos que o volume descrito justifique
Incluir:
- Código do servidor com gerenciamento de conexões/salas
- Código do cliente com reconexão e heartbeat
- Esquema de mensagens (tipos/interfaces)
- Nota final listando suposições feitas (protocolo, stack, autenticação, volume esperado)
</output_specification>

<quality_criteria>
Outputs excelentes:
- Implementam reconexão com backoff exponencial no cliente, não apenas uma tentativa única
- Usam heartbeat/ping-pong para detectar e limpar conexões mortas no servidor
- Direcionam mensagens a salas/canais específicos em vez de broadcast global desnecessário
- Validam e sanitizam todo payload recebido antes de processá-lo

Evite:
- Deixar conexões abertas indefinidamente sem timeout ou heartbeat
- Enviar dados sensíveis sem qualquer camada de autenticação ou criptografia
- Fazer broadcast para todos os clientes quando apenas um subconjunto deveria receber a mensagem
- Ignorar o comportamento em redes móveis/instáveis, tratando apenas o caminho feliz de conexão estável
</quality_criteria>

<constraints>
- Nunca envie dados sensíveis (tokens, informações pessoais) sem mencionar explicitamente a necessidade de TLS/WSS e autenticação
- Não assuma escala horizontal (múltiplas instâncias, Redis pub/sub) a menos que o usuário mencione a necessidade — para um único servidor, mantenha a solução simples
- Não invente endpoints, nomes de eventos ou formatos de payload que o usuário não descreveu — use placeholders claramente marcados
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Quero um dashboard que mostra o número de pedidos em tempo real conforme eles chegam, sem precisar dar refresh na página."

**Output esperado (resumo):**

- Escolha justificada de Server-Sent Events (SSE) em vez de WebSocket, por ser um fluxo unidirecional servidor→cliente
- Endpoint SSE no servidor (`/api/orders/stream`) emitindo eventos `order.created` a cada novo pedido
- Cliente React com `EventSource`, tratando reconexão automática e parsing dos eventos
- Nota sobre autenticação (token no header ou query string, já que `EventSource` nativo não permite headers customizados) e sobre o limite de conexões simultâneas do navegador por domínio
- Nota final assinalando que o volume de pedidos por minuto não foi informado e que a solução assume volume baixo/moderado (sem necessidade de Redis pub/sub)
