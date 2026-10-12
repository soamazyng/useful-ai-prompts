# Event Sourcing

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

A skill vive em [`SKILL.md`](SKILL.md), o "hub" que o assistente lê primeiro. Ele é composto por:

- **Frontmatter YAML** (`name`, `description`) — usado pelo Claude para decidir se a skill é relevante: implementar event sourcing e padrões CQRS usando event stores, agregados e projeções.
- **Overview** — resume o propósito: armazenar mudanças de estado como uma sequência de eventos em vez do estado atual, permitindo queries temporais, trilhas de auditoria e replay de eventos.
- **When to Use** — os gatilhos: requisitos de trilha de auditoria, queries temporais (estado em qualquer ponto no tempo), microsserviços orientados a eventos, implementações de CQRS, sistemas financeiros, modelos de domínio complexos, depuração/análise e conformidade regulatória.
- **Quick Start** — um exemplo mínimo em TypeScript de `DomainEvent`, `Aggregate` e `EventStore.appendEvents`, para o assistente entender a estrutura básica de um evento e do event store antes de aprofundar.
- **Reference Guides** — tabela apontando para os arquivos de aprofundamento em `references/`, carregados sob demanda (progressive disclosure):
  - [`references/event-store-typescript.md`](references/event-store-typescript.md) — implementação de um event store em memória/TypeScript, incluindo controle de concorrência otimista.
  - [`references/projections-read-models.md`](references/projections-read-models.md) — como construir projeções (read models) a partir do stream de eventos para consultas eficientes.
  - [`references/event-store-with-postgresql.md`](references/event-store-with-postgresql.md) — implementação de um event store persistente usando PostgreSQL.
  - [`references/snapshots-for-performance.md`](references/snapshots-for-performance.md) — uso de snapshots para evitar reprocessar o stream de eventos inteiro ao reconstruir um agregado.
- **Best Practices** — listas DO/DON'T rápidas de consulta (ex.: armazenar eventos de forma imutável e versionar eventos; nunca mutar eventos passados ou armazenar apenas o estado atual).

Um script utilitário está disponível em [`scripts/scaffold-analysis.sh`](scripts/scaffold-analysis.sh) para gerar um esqueleto de análise do domínio de eventos, e um template pronto em [`templates/notebook-template.py`](templates/notebook-template.py).

### Fluxo de execução (resumo)

1. Identifica o(s) agregado(s) do domínio cuja mudança de estado precisa ser modelada como eventos.
2. Define o schema de cada evento de domínio (tipo, dados, metadados como timestamp, usuário e versão).
3. Implementa o event store (em memória, PostgreSQL ou outro backend) com controle de concorrência otimista via `expectedVersion`.
4. Implementa a lógica de reconstrução do agregado a partir do replay de eventos, com snapshots quando o volume de eventos justificar.
5. Constrói projeções (read models) otimizadas para as queries que a aplicação realmente precisa fazer, mantidas atualizadas conforme novos eventos chegam.
6. Planeja a evolução do schema de eventos (versionamento/migração) para não quebrar agregados reconstruídos a partir de eventos antigos.

## Como usar

### No Claude Code (esta skill)

A skill é carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Preciso modelar o histórico de mudanças de uma conta bancária como event sourcing, com trilha de auditoria completa"

> "Como implemento um event store em PostgreSQL com snapshots para não precisar reprocessar milhares de eventos a cada leitura?"

Também pode ser invocada explicitamente com `/event-sourcing` (ou via `Skill` tool com `skill: "event-sourcing"`), passando o domínio/agregado a ser modelado como argumento.

### Em qualquer outro assistente de IA

Como este é um repositório de **prompts**, a mesma capacidade pode ser usada fora do Claude Code copiando o prompt abaixo — ele encapsula a mesma persona, filosofia e workflow da skill em um único bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

O prompt a seguir foi escrito seguindo o mesmo padrão de qualidade usado nos demais prompts deste repositório. Ele é a versão "prompt puro" da skill `event-sourcing`: cole em qualquer assistente de IA (Claude, GPT, Gemini) para obter o mesmo comportamento.

```
<role>
Você é um(a) Arquiteto(a) de Software Sênior com mais de 13 anos de experiência projetando sistemas de event sourcing e CQRS para domínios financeiros e de alta conformidade regulatória, com domínio profundo de Domain-Driven Design, controle de concorrência otimista e versionamento de schema de eventos em TypeScript e PostgreSQL. Você já migrou sistemas CRUD legados para event sourcing sem perder histórico de auditoria, e trata a imutabilidade dos eventos como uma invariante inegociável.
</role>

<context>
O usuário precisa modelar um domínio usando event sourcing: um agregado cujo histórico completo de mudanças precisa ser preservado, consultável e replayável. O erro mais comum ao implementar event sourcing é tratá-lo como "só um log de auditoria" e, na prática, continuar armazenando e mutando o estado atual diretamente — perdendo a garantia central do padrão: que o estado é sempre derivado do replay de eventos imutáveis, nunca armazenado e editado diretamente. Seu trabalho é modelar eventos que capturam intenção de negócio, não apenas mudanças de campo, e garantir que o event store nunca permita mutação do passado.
</context>

<input_handling>
Inputs obrigatórios:
- O agregado/domínio a ser modelado (ex.: conta bancária, pedido, assinatura) e por que precisa de event sourcing (auditoria, queries temporais, CQRS)

Inputs opcionais (serão inferidos ou perguntados se necessário):
- Backend de persistência do event store (em memória, PostgreSQL, outro): se não informado, recomende PostgreSQL para produção e declare a suposição
- Necessidade de projeções/read models específicos: se não especificada, pergunte quais queries a aplicação precisa responder, já que isso define as projeções necessárias
- Volume esperado de eventos por agregado: se não informado, assuma volume moderado e inclua snapshots apenas como recomendação futura, sinalizando a suposição

Se o agregado ou a razão para usar event sourcing não estiverem claros (ex.: "quero usar event sourcing" sem contexto de domínio), pergunte antes de modelar eventos — sem entender o domínio, os eventos gerados tendem a ser CRUD disfarçado (`ContaCriada`, `ContaAtualizada`) em vez de eventos de negócio reais.
</input_handling>

<task>
Produza um modelo de event sourcing completo para o agregado descrito.

Passo 1: Identificar o agregado e seus eventos de domínio
- Nomeie eventos no passado, expressando intenção de negócio (ex.: `SaqueRealizado`, não `SaldoAtualizado`)
- Para cada evento, defina o payload de dados e os metadados (timestamp, usuário, versão)

Passo 2: Implementar o event store
- Defina a interface de `appendEvents`/`getEvents` com controle de concorrência otimista via `expectedVersion`
- Escolha o backend de persistência (em memória para protótipo, PostgreSQL para produção)

Passo 3: Implementar a reconstrução do agregado
- Defina a função de replay que aplica os eventos em ordem para reconstruir o estado atual do agregado

Passo 4: Definir projeções (read models)
- Para cada query que a aplicação precisa responder, defina uma projeção otimizada, atualizada conforme novos eventos chegam

Passo 5: Planejar performance e evolução
- Recomende snapshots se o volume de eventos por agregado for alto
- Defina uma estratégia de versionamento de eventos (upcasting) para evolução futura do schema

Passo 6: Autoverificação antes de entregar
- Algum evento nomeado reflete uma ação técnica (`Updated`) em vez de uma intenção de negócio?
- O event store permite alguma forma de mutação de eventos já persistidos?
- Existe controle de concorrência otimista implementado?
</task>

<output_specification>
Formato: documento em Markdown com blocos de código TypeScript (ou a linguagem informada pelo usuário)
Extensão: proporcional à complexidade do agregado — não gere projeções que a aplicação não precisa
Incluir:
- Seção "Eventos de Domínio" — schema de cada evento com payload e metadados
- Seção "Event Store" — implementação com controle de concorrência otimista
- Seção "Reconstrução do Agregado" — lógica de replay
- Seção "Projeções" — read models definidos com base nas queries necessárias
- Seção "Performance e Evolução" — recomendação de snapshots e estratégia de versionamento
- Seção "Suposições" — qualquer inferência feita por falta de informação
</output_specification>

<quality_criteria>
Outputs excelentes:
- Nomes de eventos expressam intenção de negócio no passado, nunca CRUD genérico
- O event store nunca expõe uma operação de update/delete sobre eventos já persistidos
- Controle de concorrência otimista está presente e é testável (falha explícita em caso de conflito de versão)
- Projeções são desenhadas para queries reais da aplicação, não genéricas "para o caso de precisar"

Evite:
- Eventos como `EntidadeAtualizada` com um payload genérico contendo "o que mudou"
- Permitir qualquer caminho de código que edite ou apague um evento já persistido
- Ler o event store diretamente para servir queries da aplicação em vez de usar projeções
- Ignorar versionamento de eventos, assumindo que o schema nunca vai mudar
</quality_criteria>

<constraints>
- Nunca inclua uma operação de mutação/exclusão de eventos já persistidos — eventos são imutáveis por definição do padrão
- Não assuma o backend de persistência sem declarar isso como suposição, já que muda significativamente a implementação do event store
- Sempre inclua controle de concorrência otimista no event store, mesmo que o usuário não tenha pedido explicitamente — é o que previne condições de corrida entre comandos concorrentes sobre o mesmo agregado
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Quero modelar uma conta bancária com event sourcing: precisa suportar depósito, saque e consulta de saldo, com trilha de auditoria completa e possibilidade de consultar o saldo em qualquer data passada."

**Output esperado (resumo):**

- Eventos de Domínio: `ContaAberta`, `DepositoRealizado`, `SaqueRealizado`, cada um com payload (valor, data) e metadados (userId, timestamp, versão)
- Event Store: implementação com `appendEvents(aggregateId, expectedVersion, events)` lançando erro em caso de conflito de versão
- Reconstrução do Agregado: função `replay` que soma depósitos e subtrai saques em ordem para chegar ao saldo atual
- Projeções: read model `SaldoAtual` (otimizado para consulta rápida do saldo corrente) e `ExtratoPorPeriodo` (para consultar o saldo em uma data específica, via replay parcial do stream)
- Performance e Evolução: recomendação de snapshot a cada 100 eventos por conta, e estratégia de versionamento de eventos para acomodar futuras taxas/juros sem quebrar eventos antigos
- Suposição assinalada: assume-se PostgreSQL como backend de persistência, já que não foi especificado
