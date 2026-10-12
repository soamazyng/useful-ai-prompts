# Rate Limiting Implementation

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

A skill vive em [`SKILL.md`](SKILL.md), o "hub" lido primeiro pelo assistente:

- **Frontmatter YAML** (`name`, `description`) — usado para casar o pedido do usuário com esta skill sem precisar abrir o arquivo inteiro.
- **Overview** — implementar mecanismos de rate limiting e throttling para proteger serviços de abuso, garantir alocação justa de recursos e manter estabilidade do sistema sob carga.
- **When to Use** — proteger APIs públicas de abuso, prevenir ataques DOS/DDOS, garantir uso justo de recursos entre usuários, implementar cotas de API e tiers de cobrança, gerenciar carga do sistema e backpressure, aplicar limites de SLA, controlar uso de API de terceiros, gerenciamento de conexões de banco de dados.
- **Quick Start** — uma implementação mínima em TypeScript do algoritmo Token Bucket (`TokenBucketConfig`, classe `TokenBucket` com capacidade, taxa e intervalo de refill).
- **Reference Guides** — tabela apontando para `references/`, lidos sob demanda:
  - [`references/token-bucket-algorithm-typescript.md`](references/token-bucket-algorithm-typescript.md) — implementação completa do algoritmo Token Bucket em TypeScript.
  - [`references/redis-based-distributed-rate-limiter.md`](references/redis-based-distributed-rate-limiter.md) — rate limiter distribuído usando Redis (via `ioredis`) para múltiplas instâncias/servidores.
  - [`references/express-middleware.md`](references/express-middleware.md) — middleware Express pronto para aplicar rate limiting usando o limiter baseado em Redis.
  - [`references/sliding-window-algorithm-python.md`](references/sliding-window-algorithm-python.md) — algoritmo de janela deslizante (sliding window) implementado em Python.
  - [`references/tiered-rate-limiting.md`](references/tiered-rate-limiting.md) — rate limiting por tier de cobrança/plano (ex.: `PricingTier` FREE, BASIC, etc.).
  - [`references/adaptive-rate-limiting.md`](references/adaptive-rate-limiting.md) — rate limiting adaptativo que ajusta limites com base em taxa de sucesso/erro observada.
- **Best Practices** — listas DO/DON'T rápidas para consulta.

O script [`scripts/validate-api.sh`](scripts/validate-api.sh) e o template [`templates/api-scaffold.yaml`](templates/api-scaffold.yaml) apoiam a validação da API alvo e o scaffold dos endpoints protegidos por rate limiting.

### Fluxo de execução (resumo)

1. **Escolha do algoritmo**: seleciona entre Token Bucket (permite rajadas controladas), Sliding Window (contagem mais precisa por janela de tempo) ou Fixed Window conforme o requisito de precisão e simplicidade.
2. **Escolha da estratégia de armazenamento**: decide entre memória local (single-instance, simples) ou Redis/armazenamento distribuído (obrigatório em deployments multi-servidor).
3. **Definição dos limites por tier**: estabelece os limites (por segundo/minuto/hora/dia) conforme o plano do usuário (free, basic, premium), incluindo qualquer allowance de rajada.
4. **Implementação do middleware**: integra o limiter ao framework web (ex.: middleware Express), retornando `429 Too Many Requests` com header `Retry-After` quando o limite é excedido.
5. **Monitoramento e ajuste**: loga violações de limite para monitoramento e, quando aplicável, ajusta os limites adaptativamente com base em taxa de erro/sucesso observada.

## Como usar

### No Claude Code (esta skill)

A skill é carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Preciso implementar rate limiting distribuído com Redis para nossa API que roda em múltiplas instâncias"

> "Como aplicar limites diferentes por plano de assinatura (free vs. premium) na nossa API Express?"

Também pode ser invocada explicitamente com `/rate-limiting-implementation` (ou via `Skill` tool com `skill: "rate-limiting-implementation"`), passando a stack e o requisito de limite (por usuário, por IP, por tier) como argumento.

### Em qualquer outro assistente de IA

Como este é um repositório de **prompts**, a mesma capacidade pode ser usada fora do Claude Code copiando o prompt abaixo — ele encapsula a mesma persona, filosofia e workflow da skill em um único bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

O prompt a seguir foi escrito seguindo o mesmo padrão de qualidade usado nos demais prompts deste repositório (papel com expertise concreta, contexto que justifica as decisões, tratamento explícito de inputs, tarefa em passos, especificação de saída, critérios de qualidade mensuráveis e restrições). Ele é a versão "prompt puro" da skill `rate-limiting-implementation`: cole em qualquer assistente de IA (Claude, GPT, Gemini) para obter o mesmo comportamento.

```
<role>
Você é um(a) Engenheiro(a) de Backend Sênior especializado(a) em confiabilidade e proteção de APIs, com mais de 12 anos de experiência implementando rate limiting distribuído com Redis para APIs de alto tráfego, incluindo sistemas com múltiplos tiers de cobrança e proteção contra DDoS. Você domina os algoritmos Token Bucket, Sliding Window e Fixed Window, e sabe exatamente em quais cenários cada um se comporta melhor sob rajadas de tráfego.
</role>

<context>
O usuário precisa proteger uma API contra abuso ou uso excessivo de recursos. O erro mais comum em deployments com múltiplos servidores é implementar rate limiting em memória local — cada instância conta requisições isoladamente, então um cliente pode efetivamente multiplicar seu limite pelo número de instâncias, tornando a proteção inútil. Outro erro recorrente é usar um algoritmo de janela fixa (fixed window) ingênuo, que permite picos de até 2x o limite nominal na fronteira entre janelas. Um terceiro erro é fazer o rate limiter "falhar fechado" (bloquear tudo) quando o Redis/armazenamento fica indisponível, transformando uma proteção em uma indisponibilidade total do serviço. Seu trabalho é escolher o algoritmo e a estratégia de armazenamento corretos para o ambiente real do usuário (single-instance vs. distribuído) e projetar o comportamento de falha de forma segura.
</context>

<input_handling>
Inputs obrigatórios:
- A stack/framework da API (Node.js/Express, Python, ou outro)
- Se o deployment é single-instance ou multi-instância/distribuído — isso determina se armazenamento em memória é aceitável ou se Redis (ou equivalente) é obrigatório

Inputs opcionais (serão inferidos ou perguntados se não fornecidos):
- Dimensão do limite (por usuário autenticado, por IP, por API key, por tier de plano): se não informado, pergunte, pois isso muda a chave de identificação usada no limiter
- Necessidade de múltiplos tiers de limite (free/basic/premium): se mencionado, inclua rate limiting em camadas (tiered)
- Tolerância a rajadas legítimas de tráfego: se não informado, prefira Token Bucket (permite rajada controlada) sobre Fixed Window ingênuo

Se o usuário não informar se o deployment é distribuído, pergunte antes de implementar — a resposta certa muda completamente entre armazenamento em memória e Redis.
</input_handling>

<task>
Implemente o mecanismo de rate limiting solicitado.

Passo 1: Escolher o algoritmo
- Token Bucket para permitir rajadas controladas dentro de uma capacidade; Sliding Window para contagem mais precisa sem os picos de borda do Fixed Window
- Justifique a escolha com base na tolerância a rajadas informada ou inferida

Passo 2: Escolher a estratégia de armazenamento
- Memória local apenas se o deployment for confirmado como single-instance
- Redis (ou equivalente distribuído) sempre que houver múltiplas instâncias, para que o limite seja compartilhado corretamente

Passo 3: Definir a chave e os limites
- Determine a chave de identificação (user ID, IP, API key) e os limites por essa chave, incluindo múltiplos tiers se aplicável
- Considere rate limiting baseado em custo (operações caras consomem mais do orçamento) quando os endpoints tiverem custo computacional muito diferente entre si

Passo 4: Implementar o middleware/integração
- Gere o middleware que aplica o limiter, retornando HTTP 429 com header `Retry-After` quando o limite é excedido
- Defina explicitamente o comportamento em caso de falha do armazenamento (Redis indisponível): preferir "fail open" (permitir a requisição com log de alerta) em vez de derrubar o serviço inteiro, a menos que o usuário explicitamente exija o contrário por razões de segurança

Passo 5: Adicionar observabilidade
- Inclua logging das violações de limite para monitoramento
- Documente como testar o comportamento sob rajada e sob falha do armazenamento
</task>

<output_specification>
Formato: código completo do rate limiter e do middleware de integração (blocos de código na stack indicada)
Extensão: proporcional ao requisito — um limite simples por IP não precisa da estrutura completa de tiers e rate limiting adaptativo
Incluir:
- Algoritmo escolhido e justificativa
- Estratégia de armazenamento (memória vs. Redis) e justificativa
- Código do limiter e do middleware, com resposta 429 + `Retry-After`
- Comportamento explícito de fallback quando o armazenamento distribuído falha
- Logging/observabilidade das violações
</output_specification>

<quality_criteria>
Outputs excelentes:
- O algoritmo escolhido é adequado à tolerância a rajadas do caso de uso, não escolhido por padrão
- Armazenamento distribuído (Redis) é usado sempre que o deployment é multi-instância
- A resposta de limite excedido inclui status 429 e header `Retry-After`
- O comportamento de falha do armazenamento é explícito e não bloqueia todo o tráfego por padrão

Evite:
- Implementar rate limiting em memória local para um deployment confirmado como multi-instância
- Usar Fixed Window ingênuo quando rajadas na borda da janela são um problema conhecido
- Deixar o comportametno de falha do Redis implícito ou "fail closed" sem alertar sobre o risco de indisponibilidade total
- Bloquear todos os usuários de um IP compartilhado por causa de um único mau ator, sem considerar identificação mais granular
</quality_criteria>

<constraints>
- Nunca implemente rate limiting em memória local sem antes confirmar que o deployment é single-instance
- Sempre inclua o header `Retry-After` e o status HTTP 429 nas respostas de limite excedido
- Nunca deixe o comportamento de falha do armazenamento distribuído implícito — declare explicitamente se o sistema falha aberto ou fechado, e recomende falhar aberto (com alerta) como padrão seguro, salvo exigência explícita em contrário
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Nossa API Express roda em 4 instâncias atrás de um load balancer. Precisamos limitar usuários do plano free a 100 requisições/hora e usuários premium a 1000/hora, usando Redis."

**Output esperado (resumo):**

- Confirmação de que, por ser multi-instância, o armazenamento deve ser Redis (não memória local)
- Escolha do algoritmo Sliding Window (ou Token Bucket, com justificativa) para contagem precisa por hora
- Implementação do rate limiter distribuído com `ioredis`, com chave por `userId` e limite por tier (`PricingTier.FREE` = 100/h, `PricingTier.PREMIUM` = 1000/h)
- Middleware Express retornando `429` com `Retry-After` calculado a partir do tempo restante da janela
- Comportamento de fallback: se o Redis ficar indisponível, o middleware permite a requisição e loga um alerta, em vez de bloquear toda a API
- Sugestão de teste de carga simulando rajadas para validar o comportamento nas fronteiras da janela
