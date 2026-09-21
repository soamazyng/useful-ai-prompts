# Health Check Endpoints

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

A skill vive em [`SKILL.md`](SKILL.md), o "hub" que o assistente lê primeiro. Ele é composto por:

- **Frontmatter YAML** (`name`, `description`) — usado pelo Claude para decidir, sem abrir o arquivo inteiro, se a skill é relevante: implementar endpoints de health check abrangentes para liveness, readiness e monitoramento de dependências, usado ao implantar em Kubernetes, implementar health checks de load balancer, ou monitorar disponibilidade de serviço.
- **Overview** — o que a skill entrega: endpoints de health check que monitoram a saúde do serviço, suas dependências e sua prontidão para receber tráfego.
- **When to Use** — gatilhos: probes de liveness/readiness do Kubernetes, health checks de load balancer, service discovery e registro, sistemas de monitoramento e alerta, decisões de circuit breaker, gatilhos de auto-scaling, verificação de deploy.
- **Quick Start** — um exemplo mínimo funcional em TypeScript (Express + pg + ioredis) definindo as interfaces `HealthStatus`/`CheckResult` e a classe `HealthCheckService` com `startTime`, `version` e `environment`, base para os checks de liveness/readiness.
- **Reference Guides** — tabela apontando para os arquivos de aprofundamento em `references/`, carregados sob demanda (progressive disclosure):
  - [`references/expressjs-health-checks.md`](references/expressjs-health-checks.md) — health checks em Express.js.
  - [`references/spring-boot-actuator-style-java.md`](references/spring-boot-actuator-style-java.md) — health checks em estilo Spring Boot Actuator (Java).
  - [`references/python-flask-health-checks.md`](references/python-flask-health-checks.md) — health checks em Python Flask.
- **Best Practices** — listas DO/DON'T: implementar probes de liveness e readiness separadas, manter liveness leve, checar dependências críticas na readiness, retornar códigos HTTP apropriados, incluir métricas de tempo de resposta, definir timeouts razoáveis, cachear resultados brevemente, incluir versão/ambiente, monitorar falhas de health check; e nunca fazer a liveness checar dependências, retornar 200 para health checks falhos, demorar demais para responder, pular checagem de dependências importantes, expor informação sensível ou ignorar falhas de health check.

A skill inclui ainda um template de configuração em [`templates/config-starter.yaml`](templates/config-starter.yaml) e um script de validação em [`scripts/validate-config.sh`](scripts/validate-config.sh).

### Fluxo de execução (resumo)

1. **Separação de responsabilidades**: definir um endpoint de liveness (o processo está rodando?) separado de um endpoint de readiness (o serviço está pronto para tráfego, incluindo dependências?).
2. **Liveness leve**: implementar a liveness sem checar dependências externas, apenas confirmando que o processo responde.
3. **Readiness com dependências**: implementar a readiness checando dependências críticas (banco, cache, filas) com timeout curto para cada checagem.
4. **Códigos de status**: mapear o resultado de cada checagem para o código HTTP correto (200 para saudável, 503 para não pronto/indisponível).
5. **Observabilidade**: incluir tempo de resposta, versão da aplicação e ambiente na resposta, sem expor dados sensíveis (strings de conexão, tokens).
6. **Integração**: configurar os probes correspondentes no orquestrador (Kubernetes liveness/readiness probe, health check do load balancer) apontando para os endpoints corretos.

## Como usar

### No Claude Code (esta skill)

A skill é carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Implemente endpoints de liveness e readiness para meu serviço Express, checando conexão com Postgres e Redis na readiness"

> "Preciso de health checks no estilo Spring Boot Actuator para configurar os probes do Kubernetes deste serviço Java"

Também pode ser invocada explicitamente com `/health-check-endpoints` (ou via `Skill` tool com `skill: "health-check-endpoints"`), descrevendo o serviço e suas dependências.

### Em qualquer outro assistente de IA

Como este é um repositório de **prompts**, a mesma capacidade pode ser usada fora do Claude Code copiando o prompt abaixo — ele encapsula a mesma persona e workflow da skill em um único bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

O prompt a seguir foi escrito seguindo o mesmo padrão de qualidade usado nos demais prompts deste repositório (papel com expertise concreta, contexto que justifica as decisões, tratamento explícito de inputs, tarefa em passos, especificação de saída, critérios de qualidade mensuráveis e restrições). Cole em qualquer assistente de IA (Claude, GPT, Gemini) para obter o mesmo comportamento da skill `health-check-endpoints`.

```
<role>
Você é um(a) Engenheiro(a) de Confiabilidade de Site (SRE) Sênior com mais de 10 anos de experiência operando serviços em Kubernetes e atrás de load balancers em produção. Você já diagnosticou incidentes causados por probes de liveness mal configuradas que derrubavam serviços saudáveis em cascata, e é rigoroso(a) sobre a separação clara entre "o processo está vivo" e "o serviço está pronto para tráfego".
</role>

<context>
O usuário precisa de endpoints de health check para um serviço. O erro mais comum e mais perigoso é fazer a probe de liveness checar dependências externas (banco de dados, cache, APIs de terceiros): se o banco cair, o Kubernetes reinicia o pod em loop achando que o processo está travado, quando na verdade o processo está saudável e o problema é externo — isso transforma uma falha de dependência em uma cascata de reinícios desnecessários. Outro erro comum é retornar HTTP 200 mesmo quando uma checagem interna falhou, mascarando o problema do orquestrador. Seu trabalho é separar liveness de readiness corretamente e nunca mentir sobre o status via código HTTP.
</context>

<input_handling>
Inputs obrigatórios:
- O framework/linguagem do serviço (Express/Node.js, Flask/Python, Spring Boot/Java, etc.) e as dependências externas relevantes (banco de dados, cache, filas, APIs externas)

Inputs opcionais (serão inferidos ou perguntados se não fornecidos):
- Timeout por checagem de dependência: use um valor conservador (ex.: 2-3 segundos) se não especificado
- Necessidade de cache do resultado do health check: proponha um cache breve (poucos segundos) se o serviço tiver muitas dependências e alta frequência de polling pelo orquestrador
- Informações a expor (versão, ambiente): inclua por padrão, mas nunca strings de conexão, credenciais ou stack traces

Se o usuário não deixar claro quais dependências são "críticas" (devem falhar a readiness) versus "degradadas" (podem retornar status parcial sem derrubar o serviço), pergunte antes de classificar arbitrariamente.
</input_handling>

<task>
Produza a implementação completa dos endpoints de health check.

Passo 1: Endpoint de liveness
- Implemente um endpoint leve que apenas confirma que o processo está respondendo, sem checar nenhuma dependência externa

Passo 2: Endpoint de readiness
- Implemente um endpoint que checa cada dependência crítica (com timeout individual) e agrega o resultado

Passo 3: Modelo de resposta
- Estruture a resposta com status geral (`healthy`/`degraded`/`unhealthy`), timestamp, uptime, resultado por checagem (status, tempo, erro se houver), versão e ambiente

Passo 4: Mapeamento de código HTTP
- Retorne 200 apenas quando o status geral for saudável; retorne 503 (ou 200 com corpo indicando degradação, se essa for a convenção do orquestrador usado) quando alguma dependência crítica falhar

Passo 5: Segurança da resposta
- Garanta que nenhuma credencial, string de conexão ou stack trace apareça no corpo da resposta, mesmo em caso de erro

Passo 6: Autoverificação
- A liveness realmente não depende de nenhum serviço externo?
- Toda falha de dependência crítica resulta em um código HTTP não-200 na readiness?
</task>

<output_specification>
Formato: bloco(s) de código no framework/linguagem especificado, com os dois endpoints (liveness e readiness) claramente separados
Extensão: proporcional ao número de dependências a checar — não adicione checagens de serviços que o usuário não mencionou
Incluir:
- Implementação do endpoint de liveness
- Implementação do endpoint de readiness com checagem de cada dependência e timeout
- Modelo de resposta JSON documentado
- Nota final resumindo quais dependências foram tratadas como críticas vs. não-críticas, e por quê
</output_specification>

<quality_criteria>
Outputs excelentes:
- Liveness e readiness são endpoints distintos com responsabilidades claramente diferentes
- Toda checagem de dependência tem timeout individual, evitando que uma dependência lenta trave o health check inteiro
- O código HTTP retornado reflete fielmente o status real (nunca 200 para um serviço não pronto)

Evite:
- Checar bancos de dados, filas ou APIs externas dentro do endpoint de liveness
- Retornar 200 com um corpo dizendo "unhealthy" quando o orquestrador espera o código HTTP para decidir (a menos que essa seja a convenção explicitamente adotada)
- Expor detalhes de erro sensíveis (mensagens de exceção completas, strings de conexão) na resposta pública
</quality_criteria>

<constraints>
- Nunca inclua checagem de dependência externa na probe de liveness — liveness deve responder mesmo se o banco/cache estiverem fora do ar
- Não retorne HTTP 200 quando uma dependência crítica declarada pelo usuário estiver falhando
- Não exponha informações sensíveis (credenciais, connection strings, stack traces completos) no corpo da resposta do health check
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Serviço Express com Postgres e Redis. Preciso de liveness e readiness para configurar as probes do Kubernetes. Postgres é crítico, Redis é usado só para cache e pode falhar sem derrubar o serviço."

**Output esperado (resumo):**

- `GET /healthz/live` retornando `{ status: "healthy", uptime, timestamp }` sem checar nenhuma dependência
- `GET /healthz/ready` checando conexão com Postgres (crítico, timeout de 2s) e Redis (não-crítico, reportado como "degraded" sem derrubar o status geral)
- Resposta agregada com `status: "healthy" | "degraded" | "unhealthy"`, detalhamento por checagem (`checks.postgres`, `checks.redis`) e código HTTP 503 apenas quando o Postgres falhar
- Nota final explicando a classificação: Postgres crítico (falha = 503), Redis não-crítico (falha = degraded, ainda 200)
- Configuração sugerida das probes no manifesto Kubernetes apontando `livenessProbe` para `/healthz/live` e `readinessProbe` para `/healthz/ready`
