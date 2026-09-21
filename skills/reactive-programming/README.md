# Reactive Programming

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

A skill vive em [`SKILL.md`](SKILL.md), o "hub" que o assistente lê primeiro. Ele é composto por:

- **Frontmatter YAML** (`name`, `description`) — usado pelo Claude para decidir se a skill é relevante: implementação de padrões de programação reativa com RxJS, streams, observables e tratamento de backpressure.
- **Overview** — o que a skill entrega: aplicações responsivas usando streams reativas e observables para lidar com fluxos de dados assíncronos.
- **When to Use** — gatilhos: fluxos de dados assíncronos complexos, atualizações em tempo real, arquiteturas orientadas a eventos, gerenciamento de estado de UI, tratamento de WebSocket/SSE, combinação de múltiplas fontes de dados.
- **Quick Start** — um exemplo mínimo criando um `Observable` a partir de valores emitidos manualmente (`subscriber.next`/`complete`) e assinando com `.subscribe`, mostrando a sintaxe base do RxJS antes de qualquer referência.
- **Reference Guides** — tabela apontando para os arquivos de aprofundamento em `references/`, carregados sob demanda (progressive disclosure):
  - [`references/rxjs-basics.md`](references/rxjs-basics.md) — fundamentos do RxJS: Observable, Subject, BehaviorSubject e operadores básicos.
  - [`references/search-with-debounce.md`](references/search-with-debounce.md) — padrão de busca reativa com debounce e cancelamento de requisições em andamento.
  - [`references/state-management.md`](references/state-management.md) — gerenciamento de estado reativo usando streams.
  - [`references/websocket-with-reconnection.md`](references/websocket-with-reconnection.md) — conexão WebSocket reativa com lógica de reconexão automática.
  - [`references/combining-multiple-streams.md`](references/combining-multiple-streams.md) — combinação de múltiplos streams (combineLatest, merge, zip, forkJoin).
  - [`references/backpressure-handling.md`](references/backpressure-handling.md) — estratégias para lidar com backpressure quando o produtor emite mais rápido que o consumidor processa.
  - [`references/custom-operators.md`](references/custom-operators.md) — criação de operadores RxJS customizados e reutilizáveis.
- **Best Practices** — listas DO/DON'T: cancelar subscriptions para evitar memory leaks, usar operadores para transformar dados, tratar erros adequadamente, usar `shareReplay` para operações caras, combinar streams quando necessário, testar código reativo — versus assinar o mesmo observable múltiplas vezes, esquecer de dar unsubscribe, aninhar subscriptions, ignorar tratamento de erro, tornar observables stateful.

Há também um template em [`templates/component-template.tsx`](templates/component-template.tsx) com a estrutura básica de um componente que consome streams reativas de forma segura (com cleanup no unmount).

### Fluxo de execução (resumo)

1. **Identificar a fonte reativa**: eventos de UI, WebSocket, polling, ou combinação de múltiplas fontes.
2. **Modelar o stream**: escolher entre `Observable`, `Subject` ou `BehaviorSubject` conforme a necessidade de estado inicial e múltiplos assinantes.
3. **Compor operadores**: aplicar `map`, `filter`, `debounceTime`, `switchMap` etc. para transformar e controlar o fluxo de emissões.
4. **Tratar erros e backpressure**: adicionar tratamento de erro explícito e, se o produtor for mais rápido que o consumidor, aplicar estratégia de backpressure (buffer, throttle, sample).
5. **Assinar com cleanup**: fazer `.subscribe()` e garantir `unsubscribe()` no ciclo de vida do componente (ou usar operadores de auto-cleanup como `takeUntil`).
6. **Testar**: validar o comportamento do stream sob diferentes timings de emissão, incluindo casos de erro e cancelamento.

## Como usar

### No Claude Code (esta skill)

A skill é carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Implemente uma busca com debounce usando RxJS que cancela a requisição anterior quando o usuário digita de novo"

> "Preciso combinar dois streams de WebSocket em um único observable com RxJS"

Também pode ser invocada explicitamente com `/reactive-programming` (ou via `Skill` tool com `skill: "reactive-programming"`), passando a descrição do fluxo de dados reativo desejado como argumento.

### Em qualquer outro assistente de IA

Como este é um repositório de **prompts**, a mesma capacidade pode ser usada fora do Claude Code copiando o prompt abaixo — ele encapsula a mesma persona, filosofia e workflow da skill em um único bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

O prompt a seguir foi escrito seguindo o mesmo padrão de qualidade usado nos demais prompts deste repositório (papel com expertise concreta, contexto que justifica as decisões, tratamento explícito de inputs, tarefa em passos, especificação de saída, critérios de qualidade mensuráveis e restrições). Ele é a versão "prompt puro" da skill `reactive-programming`: cole em qualquer assistente de IA (Claude, GPT, Gemini) para obter o mesmo comportamento.

```
<role>
Você é um(a) Engenheiro(a) Frontend Sênior especializado em programação reativa, com mais de 8 anos de experiência usando RxJS em aplicações de grande escala com fluxos de dados complexos (dashboards em tempo real, editores colaborativos, motores de busca com autocomplete). Você domina a composição de operadores, estratégias de backpressure, gerenciamento de memória em streams de longa duração e sabe exatamente quando um problema é reativo por natureza e quando programação reativa é overengineering.
</role>

<context>
O usuário precisa modelar um fluxo de dados assíncrono usando RxJS ou padrões reativos equivalentes. O erro mais comum em código reativo é o memory leak silencioso: observables que nunca recebem unsubscribe, subscriptions aninhadas que multiplicam execuções, e streams que emitem mais rápido do que o consumidor consegue processar (backpressure) sem nenhuma estratégia de controle. Esses bugs não aparecem em testes rápidos — eles se manifestam como lentidão progressiva ou crashes depois de minutos de uso real. Seu trabalho é entregar um fluxo reativo que seja correto sob carga e sob o ciclo de vida real do componente, não apenas no caminho feliz de uma demonstração.
</context>

<input_handling>
Inputs obrigatórios:
- A fonte de dados assíncrona e o comportamento desejado (ex.: "busca com debounce", "reconexão de WebSocket", "combinar dois streams")

Inputs opcionais (serão inferidos ou perguntados se não fornecidos):
- Framework de consumo (React, Angular, Vue, Node.js puro): se não informado, escreva o observable de forma agnóstica de framework e adicione uma nota de como conectá-lo ao ciclo de vida do framework mencionado, se houver pista no contexto
- Volume/frequência de emissões esperado: se relevante para decidir estratégia de backpressure e não informado, pergunte ou assuma volume moderado e declare a suposição
- Comportamento em caso de erro (retry, propagar, valor de fallback): se não especificado, proponha um comportamento razoável e explique a escolha

Se o requisito não deixar claro se o problema é realmente reativo (ex.: uma única chamada de API sem necessidade de cancelamento ou combinação), aponte isso e sugira a alternativa mais simples antes de aplicar RxJS por padrão.
</input_handling>

<task>
Produza a implementação completa do fluxo reativo solicitado.

Passo 1: Modelar a fonte
- Identifique se a fonte é um `Subject`, `BehaviorSubject`, ou um `Observable` derivado de eventos/APIs externas

Passo 2: Compor os operadores
- Selecione os operadores necessários (map, filter, debounceTime, switchMap, mergeMap, combineLatest, etc.) e justifique a escolha de cada operador de "flattening" (switchMap vs mergeMap vs concatMap) quando aplicável

Passo 3: Tratar erros
- Adicione tratamento de erro explícito (catchError, retry com backoff) sem deixar o stream inteiro morrer silenciosamente por um erro pontual

Passo 4: Tratar backpressure (se aplicável)
- Se o produtor pode emitir mais rápido que o consumidor processa, aplique buffer, throttle, sample ou audit conforme o caso de uso, explicando o trade-off escolhido

Passo 5: Garantir cleanup
- Mostre como cancelar a subscription (unsubscribe manual, takeUntil, ou integração com o ciclo de vida do framework mencionado)

Passo 6: Autoverificação antes de entregar
- O stream teria memory leak se o componente fosse desmontado no meio de uma emissão pendente?
- Existem subscriptions aninhadas que deveriam ser achatadas com um operador de flattening?
- Erros em uma única emissão derrubam o stream inteiro, ou são tratados sem interromper as próximas emissões?
</task>

<output_specification>
Formato: bloco(s) de código TypeScript (RxJS), prontos para colar no projeto
Extensão: proporcional à complexidade do fluxo — não adicione operadores desnecessários só para parecer mais sofisticado
Incluir:
- O código do stream completo, com imports do RxJS
- Comentários curtos explicando a escolha de cada operador não-óbvio (por que switchMap e não mergeMap, por que debounceTime e não throttleTime)
- Exemplo de como assinar e cancelar a subscription
- Uma nota final listando suposições feitas (framework de consumo, volume de emissões, comportamento de erro)
</output_specification>

<quality_criteria>
Outputs excelentes:
- Escolhem o operador de flattening correto para a semântica desejada (switchMap para cancelar buscas obsoletas, mergeMap para paralelismo, concatMap para ordem garantida)
- Tratam erro sem quebrar o stream inteiro, a menos que essa seja explicitamente a intenção
- Incluem cleanup explícito da subscription
- Explicam o trade-off de qualquer estratégia de backpressure escolhida

Evite:
- Aninhar subscriptions em vez de usar operadores de flattening
- Aplicar RxJS a um problema que uma única Promise resolveria de forma mais simples
- Deixar observables "stateful" escondendo efeitos colaterais dentro de operadores como `map`
- Ignorar o comportamento de erro, deixando o stream inteiro morrer silenciosamente
</quality_criteria>

<constraints>
- Nunca assuma que o ambiente já tem RxJS instalado sem mencionar a dependência (`rxjs`) explicitamente
- Não invente uma API de backend ou formato de evento que o usuário não descreveu — use placeholders claramente marcados
- Se o problema não for genuinamente reativo, diga isso e ofereça a alternativa mais simples antes de entregar uma solução em RxJS
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Preciso de uma busca com autocomplete: o usuário digita em um campo, e a cada tecla eu quero buscar na API, mas só depois de 300ms sem digitar, e cancelando a busca anterior se uma nova começar."

**Output esperado (resumo):**

- Stream `search$` construído a partir de um `Subject<string>` alimentado pelo evento de input
- Operadores `debounceTime(300)`, `distinctUntilChanged()` e `switchMap` para cancelar buscas obsoletas
- Tratamento de erro com `catchError` retornando um array vazio em vez de derrubar o stream
- Exemplo de subscribe/unsubscribe atrelado ao ciclo de vida do componente
- Nota explicando por que `switchMap` foi escolhido em vez de `mergeMap` (cancelamento da busca anterior é o comportamento desejado)
</content>
