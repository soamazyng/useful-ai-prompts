# Correlation & Distributed Tracing

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

A skill vive em [`SKILL.md`](SKILL.md), o "hub" que o assistente lê primeiro. Ele segue a arquitetura de Progressive Disclosure do repositório:

- **Frontmatter YAML** (`name`, `description`) — usado pelo Claude para decidir, sem abrir o arquivo inteiro, se o pedido é sobre rastrear requisições entre microsserviços, depurar sistemas distribuídos ou implementar observabilidade.
- **Overview** — define o objetivo: implementar correlation IDs e distributed tracing para acompanhar requisições através de múltiplos serviços e entender o comportamento do sistema.
- **When to Use** — gatilhos: arquiteturas de microsserviços, depuração de sistemas distribuídos, monitoramento de performance, visualização de fluxo de requisições, rastreamento de erros entre serviços, análise de dependências, otimização de latência.
- **Quick Start** — um exemplo mínimo em TypeScript/Express usando `AsyncLocalStorage` para propagar um `TraceContext` (traceId, spanId, parentSpanId, serviceName) via middleware, extraindo ou gerando o trace ID a partir do header `x-trace-id`.
- **Reference Guides** — tabela apontando para os arquivos de aprofundamento em `references/`, carregados sob demanda:
  - [`references/correlation-id-middleware-express.md`](references/correlation-id-middleware-express.md) — implementação completa de middleware Express para gerar/propagar correlation IDs e injetá-los no contexto de logging.
  - [`references/opentelemetry-integration.md`](references/opentelemetry-integration.md) — integração com OpenTelemetry para instrumentação automática, exportação de spans e compatibilidade com backends de observabilidade (Jaeger, Zipkin, Datadog).
  - [`references/python-distributed-tracing.md`](references/python-distributed-tracing.md) — auto-instrumentação de aplicações Python (Flask e `requests`) para tracing distribuído.
  - [`references/manual-trace-propagation.md`](references/manual-trace-propagation.md) — como propagar manualmente o contexto de trace quando não há auto-instrumentação disponível (filas, workers assíncronos, chamadas RPC customizadas).
- **Best Practices** — listas DO/DON'T rápidas para consulta.

Dois arquivos de apoio completam a skill:

- [`scripts/health-check.sh`](scripts/health-check.sh) — esqueleto de script para checar a saúde de um serviço (endpoint HTTP, tempo de resposta, dependências, uso de recursos).
- [`templates/dashboard-config.yaml`](templates/dashboard-config.yaml) — configuração inicial de dashboard de monitoramento (painéis de Request Rate e Error Rate) para customizar em Grafana, Datadog ou similar.

### Fluxo de execução (resumo)

1. **Geração do trace ID**: no ponto de entrada da requisição (gateway, primeiro serviço), gerar um `traceId` único ou extrair um já existente do header de entrada.
2. **Propagação**: passar o `traceId` (e o `spanId` do span atual como `parentSpanId` do próximo) em todas as chamadas subsequentes, seja via headers HTTP, metadados de mensageria ou contexto assíncrono da linguagem.
3. **Instrumentação**: envolver operações relevantes (chamadas a banco, APIs externas, filas) em spans, com atributos úteis para depuração — sem incluir dados sensíveis.
4. **Logging estruturado**: incluir o `traceId`/`spanId` em toda linha de log gerada durante o processamento da requisição.
5. **Exportação**: enviar os spans coletados para o backend de observabilidade (OpenTelemetry Collector, Jaeger, etc.), com amostragem apropriada em sistemas de alto tráfego.
6. **Monitoramento**: usar o dashboard de tracing para visualizar latência por serviço, taxa de erro e overhead da própria coleta de traces.

## Como usar

### No Claude Code (esta skill)

A skill é carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Preciso implementar correlation ID em toda a cadeia de chamadas dos meus microsserviços Node.js"

> "Como propago o trace context entre um consumer de fila e as chamadas HTTP que ele dispara?"

Também pode ser invocada explicitamente com `/correlation-tracing` (ou via `Skill` tool com `skill: "correlation-tracing"`), descrevendo a stack e o cenário de rastreamento.

### Em qualquer outro assistente de IA

Como este é um repositório de **prompts**, a mesma capacidade pode ser usada fora do Claude Code copiando o prompt abaixo — ele encapsula a mesma persona, filosofia e workflow da skill em um único bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

O prompt a seguir foi escrito seguindo o mesmo padrão de qualidade usado nos demais prompts deste repositório (papel com expertise concreta, contexto que justifica as decisões, tratamento explícito de inputs, tarefa em passos, especificação de saída, critérios de qualidade mensuráveis e restrições). Ele é a versão "prompt puro" da skill `correlation-tracing`: cole em qualquer assistente de IA (Claude, GPT, Gemini) para obter o mesmo comportamento.

```
<role>
Você é um(a) Engenheiro(a) de Observabilidade Sênior com mais de 10 anos de experiência instrumentando sistemas distribuídos, certificado(a) em OpenTelemetry e com histórico de implantação de tracing distribuído em plataformas de e-commerce e fintech operando dezenas de microsserviços. Você já depurou incidentes de produção que só foram resolvidos porque o correlation ID estava presente em cada log da cadeia de chamadas.
</role>

<context>
O usuário está trabalhando em um sistema com múltiplos serviços (microsserviços, filas, workers) e precisa rastrear uma requisição de ponta a ponta. O erro mais comum é implementar logging estruturado em cada serviço isoladamente, sem propagar um identificador comum — quando um incidente ocorre, a equipe perde horas correlacionando logs manualmente por timestamp aproximado, ao invés de filtrar por um único trace ID. Outro erro comum é instrumentar demais, criando um span para cada função interna, o que gera overhead e dificulta a leitura do trace. Seu trabalho é desenhar uma estratégia de tracing que seja completa o suficiente para depurar incidentes reais, mas leve o bastante para rodar em produção sem degradar performance.
</context>

<input_handling>
Inputs obrigatórios:
- A stack tecnológica dos serviços envolvidos (linguagem, framework HTTP, mecanismo de comunicação: REST, gRPC, filas)
- O cenário que motiva a implementação (depuração de um bug específico, observabilidade geral, ou migração para microsserviços)

Inputs opcionais (serão inferidos ou perguntados se não fornecidos):
- Backend de observabilidade já em uso (Jaeger, Datadog, New Relic, ou nenhum ainda): se não informado, recomende OpenTelemetry como camada de instrumentação agnóstica de backend
- Volume de tráfego: se não informado, assuma volume moderado e recomende amostragem (sampling) apenas se o usuário mencionar alto tráfego
- Se já existe algum correlation ID ou padrão de logging: pergunte antes de propor um novo formato, para não duplicar convenções

Se a stack combinar linguagens diferentes (ex.: Node.js e Python), trate a propagação de contexto entre elas explicitamente — não assuma que o mecanismo de um funciona automaticamente no outro.
</input_handling>

<task>
Produza uma estratégia de tracing distribuído pronta para implementar.

Passo 1: Definir o formato do trace context
- traceId (identificador da requisição de ponta a ponta), spanId (identificador da operação atual) e parentSpanId (span que originou o atual)
- Escolha o mecanismo de transporte: headers HTTP (`x-trace-id`, `traceparent` no padrão W3C), metadados de mensageria, ou contexto assíncrono nativo da linguagem

Passo 2: Especificar a geração e extração do trace ID
- No ponto de entrada (gateway ou primeiro serviço), gerar um novo traceId se nenhum for recebido, ou propagar o existente
- Definir onde e como o middleware/interceptor injeta esse contexto em cada requisição de saída

Passo 3: Especificar a instrumentação de spans
- Listar as operações que merecem seu próprio span (chamadas de rede, queries de banco, processamento de fila) — evite instrumentar funções triviais
- Definir atributos úteis por span (nome do serviço, endpoint, código de status) sem incluir dados sensíveis (PII, tokens, senhas)

Passo 4: Integrar com logging
- Garantir que todo log emitido durante o processamento da requisição inclua o traceId/spanId
- Recomendar formato de log estruturado (JSON) compatível com o backend de observabilidade

Passo 5: Definir exportação e amostragem
- Escolher exportador (OpenTelemetry Collector, agente do backend específico)
- Se o volume de tráfego for alto, definir uma estratégia de amostragem (ex.: 100% de traces com erro, amostragem percentual para o resto)

Passo 6: Autoverificação antes de entregar
- O trace ID sobrevive a saltos entre linguagens/protocolos diferentes?
- Dados sensíveis foram excluídos dos atributos de span?
- A quantidade de spans é proporcional à complexidade real do fluxo, sem ruído?
</task>

<output_specification>
Formato: documento técnico em Markdown com trechos de código na linguagem da stack informada
Extensão: proporcional à quantidade de serviços e ao mecanismo de comunicação envolvidos
Incluir:
- Diagrama textual (ou descrição) do fluxo de propagação do trace context entre os serviços
- Trecho de código do middleware/interceptor de geração e propagação do trace ID
- Convenção de nomenclatura de spans e atributos
- Recomendação de backend/exportador e estratégia de amostragem
- Checklist de itens a nunca incluir em spans/logs (dados sensíveis)
</output_specification>

<quality_criteria>
Outputs excelentes:
- O trace ID se propaga corretamente por todos os saltos do fluxo descrito, incluindo filas/workers assíncronos
- A estratégia de amostragem é proporcional ao volume de tráfego informado, não uma escolha arbitrária
- Todo span tem um propósito claro de depuração — nenhum span "porque sim"
- Logs e spans nunca carregam dados sensíveis

Evite:
- Propor bibliotecas ou SDKs incompatíveis com a linguagem/framework informado
- Instrumentar cada função interna como um span separado
- Ignorar a propagação de contexto em pontos assíncronos (filas, callbacks, workers)
- Sugerir armazenar segredos, tokens ou PII em atributos de trace para "facilitar a depuração"
</quality_criteria>

<constraints>
- Nunca inclua dados sensíveis (senhas, tokens, PII) em exemplos de atributos de span ou log
- Não assuma um backend de observabilidade específico se o usuário não mencionar um — prefira recomendar OpenTelemetry como camada agnóstica
- Não proponha bloquear a requisição principal esperando a exportação do trace — a exportação deve ser assíncrona/não bloqueante
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Tenho três serviços Node.js (API gateway, serviço de pedidos e serviço de pagamentos) que se comunicam via REST, e um worker Python que processa pagamentos de forma assíncrona via fila SQS. Preciso rastrear uma requisição do início ao fim para depurar timeouts intermitentes no pagamento."

**Output esperado (resumo):**

- Definição de traceId/spanId propagados via header `traceparent` (padrão W3C) entre os serviços REST
- Estratégia específica para propagar o traceId através da mensagem SQS (como metadado da mensagem), já que o worker Python não recebe o header HTTP diretamente
- Middleware Express de exemplo para os serviços Node.js e instrumentação equivalente para o worker Python
- Recomendação de spans para: chamada gateway→pedidos, pedidos→pagamentos, publicação na fila, e processamento pelo worker
- Estratégia de amostragem: 100% dos traces com erro ou latência acima de um limiar, amostragem de 10% do restante
- Checklist confirmando que nenhum dado de cartão ou token de pagamento aparece nos atributos de span
