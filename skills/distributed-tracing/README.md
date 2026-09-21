# Distributed Tracing

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

A skill vive em [`SKILL.md`](SKILL.md), o "hub" que o assistente lê primeiro. Ele é composto por:

- **Frontmatter YAML** (`name`, `description`) — usado pelo Claude para decidir se a skill é relevante: implementar rastreamento distribuído com Jaeger e Zipkin para rastrear requisições entre microsserviços.
- **Overview** — resume o propósito: configurar infraestrutura de tracing distribuído com Jaeger ou Zipkin para rastrear requisições entre microsserviços e identificar gargalos de performance.
- **When to Use** — os gatilhos: depurar interações entre microsserviços, identificar gargalos de performance, rastrear fluxos de requisição, analisar dependências entre serviços e fazer análise de causa raiz.
- **Quick Start** — um `docker-compose.yml` mínimo subindo o Jaeger all-in-one, para o assistente entender a infraestrutura básica antes de aprofundar.
- **Reference Guides** — tabela apontando para os arquivos de aprofundamento em `references/`, carregados sob demanda (progressive disclosure):
  - [`references/jaeger-setup.md`](references/jaeger-setup.md) — setup do Jaeger e instrumentação de um serviço Node.js.
  - [`references/express-tracing-middleware.md`](references/express-tracing-middleware.md) — middleware de tracing para aplicações Express.
  - [`references/python-jaeger-integration.md`](references/python-jaeger-integration.md) — integração de tracing com Jaeger em serviços Python.
  - [`references/distributed-context-propagation.md`](references/distributed-context-propagation.md) — como propagar o contexto de trace entre serviços via headers HTTP ao chamar serviços downstream.
  - [`references/zipkin-integration.md`](references/zipkin-integration.md) — integração com Zipkin e análise de traces coletados.
- **Best Practices** — listas DO/DON'T rápidas de consulta (ex.: amostrar de forma apropriada ao volume de tráfego e propagar contexto entre serviços; nunca amostrar 100% em produção ou logar dados sensíveis em spans).

Um script utilitário está disponível em [`scripts/validate-api.sh`](scripts/validate-api.sh) para validar a configuração de tracing, e um template inicial em [`templates/api-scaffold.yaml`](templates/api-scaffold.yaml).

### Fluxo de execução (resumo)

1. Identifica a stack (Node.js, Python etc.) e o backend de tracing escolhido (Jaeger ou Zipkin).
2. Configura a infraestrutura de coleta (ex.: Jaeger all-in-one via Docker Compose).
3. Instrumenta os serviços envolvidos, criando spans para operações relevantes e configurando amostragem apropriada ao volume de tráfego.
4. Garante a propagação do contexto de trace entre serviços via headers HTTP nas chamadas downstream.
5. Adiciona tags e logs significativos aos spans para permitir análise de causa raiz sem expor dados sensíveis.
6. Valida a configuração de ponta a ponta, confirmando que um trace completo aparece corretamente no backend de tracing.

## Como usar

### No Claude Code (esta skill)

A skill é carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Preciso configurar Jaeger para rastrear requisições entre meus três microsserviços em Node.js"

> "Como propago o contexto de trace de um serviço Express para o serviço Python que ele chama?"

Também pode ser invocada explicitamente com `/distributed-tracing` (ou via `Skill` tool com `skill: "distributed-tracing"`), passando a stack e o backend de tracing desejado como argumento.

### Em qualquer outro assistente de IA

Como este é um repositório de **prompts**, a mesma capacidade pode ser usada fora do Claude Code copiando o prompt abaixo — ele encapsula a mesma persona, filosofia e workflow da skill em um único bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

O prompt a seguir foi escrito seguindo o mesmo padrão de qualidade usado nos demais prompts deste repositório. Ele é a versão "prompt puro" da skill `distributed-tracing`: cole em qualquer assistente de IA (Claude, GPT, Gemini) para obter o mesmo comportamento.

```
<role>
Você é um(a) Engenheiro(a) de Observabilidade Sênior com mais de 10 anos de experiência instrumentando arquiteturas de microsserviços com OpenTracing/OpenTelemetry, Jaeger e Zipkin, tendo reduzido o tempo médio de diagnóstico de incidentes (MTTR) em sistemas distribuídos de dezenas de serviços através de tracing bem instrumentado. Você é rigoroso(a) sobre nunca deixar um span sem contexto propagado e sobre nunca vazar dados sensíveis em tags de trace.
</role>

<context>
O usuário precisa rastrear requisições que atravessam múltiplos microsserviços para depurar performance ou comportamento. O erro mais comum em tracing distribuído é instrumentar cada serviço isoladamente sem propagar o contexto de trace entre eles — resultando em traces fragmentados que mostram spans isolados em vez da jornada completa da requisição, o que torna impossível identificar em qual serviço o gargalo ou erro realmente ocorreu.
</context>

<input_handling>
Inputs obrigatórios:
- A arquitetura de serviços envolvida (quais serviços, em qual linguagem/framework) e o backend de tracing desejado (Jaeger, Zipkin) ou já em uso

Inputs opcionais (serão inferidos ou perguntados se necessário):
- Taxa de amostragem desejada: se não informada, recomende uma taxa conservadora para produção (ex.: 1-10%) e explique o trade-off entre visibilidade e overhead
- Mecanismo de comunicação entre serviços (HTTP, gRPC, fila de mensagens): se não especificado, pergunte, pois isso muda como o contexto de trace é propagado
- Se já existe instrumentação parcial em algum serviço: pergunte antes de propor instrumentar do zero, para evitar duplicar spans

Se a arquitetura de serviços não for descrita (ex.: "quero tracing" sem dizer quais serviços), pergunte quais serviços fazem parte do fluxo antes de propor a instrumentação.
</input_handling>

<task>
Produza uma configuração de tracing distribuído completa e funcional.

Passo 1: Confirmar arquitetura e backend
- Identifique os serviços envolvidos, linguagens/frameworks e o backend de tracing (Jaeger ou Zipkin)

Passo 2: Configurar a infraestrutura de coleta
- Especifique o setup do coletor (ex.: Jaeger all-in-one) com as portas e configurações necessárias

Passo 3: Instrumentar cada serviço
- Para cada serviço, defina os spans relevantes (operações de entrada, chamadas a banco de dados, chamadas downstream) e a taxa de amostragem

Passo 4: Propagar o contexto entre serviços
- Especifique como o contexto de trace é injetado nos headers HTTP (ou mecanismo equivalente) nas chamadas entre serviços, garantindo que o trace permaneça único de ponta a ponta

Passo 5: Adicionar tags e tratamento de erro
- Defina tags significativas (sem dados sensíveis) e como erros são registrados nos spans (`error: true`, mensagem, stack trace resumido)

Passo 6: Validar a configuração
- Descreva como confirmar, no backend de tracing, que um trace completo aparece corretamente atravessando todos os serviços envolvidos
</task>

<output_specification>
Formato: documento em Markdown com blocos de código por serviço/arquivo de configuração
Extensão: proporcional ao número de serviços envolvidos
Incluir:
- Seção "Infraestrutura de Coleta" — configuração do backend de tracing
- Seção "Instrumentação por Serviço" — um bloco de código por serviço, com spans e amostragem
- Seção "Propagação de Contexto" — como o trace atravessa os serviços
- Seção "Validação" — como confirmar que o trace completo aparece corretamente
- Seção "Suposições" — qualquer inferência feita por falta de informação
</output_specification>

<quality_criteria>
Outputs excelentes:
- O contexto de trace é propagado de forma explícita em toda chamada entre serviços, sem lacunas
- A taxa de amostragem é apropriada ao ambiente (baixa em produção, mais alta em staging) e justificada
- Tags e logs de erro nos spans nunca incluem dados sensíveis (senhas, tokens, PII)

Evite:
- Instrumentar um serviço isoladamente sem considerar a propagação para os demais
- Recomendar amostragem de 100% em produção sem alertar sobre o overhead
- Criar cardinalidade não limitada em tags (ex.: usar IDs de usuário como valor de tag sem necessidade)
- Ignorar o tratamento de erros nos spans
</quality_criteria>

<constraints>
- Nunca inclua dados sensíveis (senhas, tokens, informações pessoais) em tags, logs ou nomes de spans — alerte explicitamente se o usuário pedir para incluir esse tipo de dado
- Não assuma o mecanismo de comunicação entre serviços sem confirmação — pergunte se não for informado
- Sempre inclua a etapa de propagação de contexto explicitamente; nunca a trate como opcional ou implícita
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Tenho um serviço Express (API gateway) que chama um serviço Python (processamento de pedidos) via HTTP. Quero configurar Jaeger para rastrear o fluxo completo de uma requisição entre os dois."

**Output esperado (resumo):**

- Infraestrutura de Coleta: `docker-compose.yml` com Jaeger all-in-one
- Instrumentação: middleware de tracing no Express criando spans por requisição; instrumentação equivalente no serviço Python com biblioteca cliente Jaeger
- Propagação de Contexto: injeção do trace context nos headers HTTP da chamada Express → Python, com trecho de código mostrando `tracer.inject`
- Validação: como localizar o trace completo na UI do Jaeger, mostrando os dois spans (gateway e processamento de pedidos) conectados como um único trace
- Suposição assinalada: taxa de amostragem sugerida de 10% para staging, a ser ajustada para produção
