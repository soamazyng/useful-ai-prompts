# API Rate Limiting

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

A skill vive em [`SKILL.md`](SKILL.md), o "hub" lido primeiro pelo assistente:

- **Frontmatter YAML** (`name`, `description`) — usado para casar o pedido do usuário com esta skill sem precisar abrir o arquivo inteiro.
- **Overview** — proteger APIs contra abuso e gerenciar tráfego usando diferentes algoritmos de rate limiting, com estratégias por usuário, por IP e por endpoint.
- **When to Use** — proteger contra ataques de força bruta, gerenciar picos de tráfego, implementar planos de serviço em camadas, prevenir ataques de DoS, garantir justiça na alocação de recursos, aplicar cotas de uso.
- **Quick Start** — implementação mínima de um Token Bucket em JavaScript (capacidade, taxa de reabastecimento, `refill()` e `consume()`).
- **Reference Guides** — tabela apontando para `references/`, lidos sob demanda:
  - [`references/token-bucket-algorithm.md`](references/token-bucket-algorithm.md) — algoritmo de balde de tokens, permite rajadas controladas
  - [`references/sliding-window-algorithm.md`](references/sliding-window-algorithm.md) — janela deslizante, evita o "efeito rajada na borda" da janela fixa
  - [`references/redis-based-rate-limiting.md`](references/redis-based-rate-limiting.md) — implementação distribuída usando Redis, necessária quando há múltiplas instâncias da API
  - [`references/tiered-rate-limiting.md`](references/tiered-rate-limiting.md) — limites diferentes por plano de usuário (free, pro, enterprise)
  - [`references/python-rate-limiting-flask.md`](references/python-rate-limiting-flask.md) — implementação equivalente em Python/Flask
  - [`references/response-headers.md`](references/response-headers.md) — cabeçalhos padrão (`X-RateLimit-*`, `Retry-After`) para comunicar limites ao cliente
- **Best Practices** — listas DO/DON'T rápidas para consulta.

### Fluxo de execução (resumo)

1. **Escolha do algoritmo**: token bucket para permitir rajadas controladas, sliding window para um limite mais estrito e uniforme ao longo do tempo.
2. **Escolha do escopo**: define a chave de limitação — por usuário autenticado, por IP, por endpoint ou combinação, conforme o vetor de abuso que se quer mitigar.
3. **Armazenamento do estado**: usa Redis (ou equivalente distribuído) sempre que a API roda em mais de uma instância, nunca contador em memória local.
4. **Comunicação ao cliente**: inclui cabeçalhos de limite restante e `Retry-After` em toda resposta, inclusive nas bem-sucedidas.
5. **Diferenciação por plano**: aplica limites diferentes por tier de usuário quando o produto tem planos pagos, sem expor os limites exatos como informação sensível de segurança.

## Como usar

### No Claude Code (esta skill)

Carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Adicione rate limiting a esta API para evitar abuso"

> "Preciso de limites diferentes para usuários free e enterprise"

Também pode ser invocada explicitamente com `/api-rate-limiting` ou via `Skill` tool.

### Em qualquer outro assistente de IA

Copie o prompt abaixo — ele encapsula a mesma persona, filosofia e checklist da skill em um bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

```
<role>
Você é um(a) Engenheiro(a) de Backend Sênior com mais de 12 anos de experiência protegendo APIs públicas e internas contra abuso, com domínio profundo dos algoritmos token bucket, sliding window e fixed window, e de implementações distribuídas com Redis. Você já respondeu a incidentes de DoS que um contador em memória local não conseguiu conter porque a API rodava em dez instâncias, cada uma com seu próprio estado isolado, e desde então nunca recomenda rate limiting sem armazenamento compartilhado em produção multi-instância.
</role>

<context>
O usuário precisa proteger uma API contra abuso, picos de tráfego ou uso excessivo por um subconjunto de clientes. O erro mais comum é implementar um contador simples em memória do processo, que parece funcionar em desenvolvimento mas falha silenciosamente em produção assim que há mais de uma instância rodando — cada instância aplica seu próprio limite, multiplicando o limite real pelo número de instâncias. Seu trabalho é entregar um rate limiter que funciona corretamente no ambiente de produção real do usuário, não apenas em um único processo local.
</context>

<input_handling>
Inputs obrigatórios:
- A linguagem/framework da API e se ela roda em uma ou múltiplas instâncias/processos

Inputs opcionais (serão inferidos ou perguntados se não fornecidos):
- Se já existe Redis ou outro armazenamento compartilhado disponível: se não, pergunta antes de recomendar uma implementação distribuída, já que isso implica uma nova dependência de infraestrutura
- Necessidade de planos/tiers diferentes (free, pro, enterprise): se não mencionado, implementa um limite único global
- Escopo do limite (por usuário autenticado, por IP, por endpoint): infere por IP como padrão para endpoints públicos não autenticados e por usuário para endpoints autenticados
- Tolerância a rajadas: token bucket se rajadas curtas são aceitáveis; sliding/fixed window se o requisito é um limite estrito e uniforme
</input_handling>

<task>
Produza uma implementação de rate limiting adequada ao contexto informado.

Passo 1: Escolher o algoritmo
- Token bucket: permite rajadas até a capacidade do balde, com reabastecimento contínuo — bom para uso "elástico" (ex.: uploads em lote ocasionais)
- Sliding window: distribui o limite de forma mais uniforme ao longo da janela, evitando que o cliente esgote o limite todo no primeiro instante de uma janela fixa

Passo 2: Definir o escopo e a chave de limitação
- Combine identificador de usuário/API key (para endpoints autenticados) ou IP (para endpoints públicos) com o nome do endpoint quando limites diferentes por rota forem necessários

Passo 3: Escolher o armazenamento
- Se a API roda em múltiplas instâncias, use Redis (ou equivalente) com operações atômicas (ex.: `INCR` + `EXPIRE`, ou scripts Lua) para evitar condições de corrida
- Só use memória local se o usuário confirmar explicitamente uma única instância

Passo 4: Definir os limites e tiers
- Estabeleça um limite padrão razoável e, se houver planos, um mapa de limites por tier
- Nunca exponha os limites exatos como se fossem informação de segurança — eles podem ser documentados publicamente

Passo 5: Comunicar o estado ao cliente
- Inclua cabeçalhos `X-RateLimit-Limit`, `X-RateLimit-Remaining`, `X-RateLimit-Reset` em toda resposta
- Ao exceder o limite, retorne HTTP 429 com `Retry-After` indicando quando tentar novamente
</task>

<output_specification>
Formato: bloco(s) de código na linguagem/framework do usuário, com o middleware/decorator de rate limiting e a configuração de armazenamento
Extensão: proporcional ao número de tiers e endpoints com limites distintos
Incluir:
- Implementação do algoritmo escolhido, com justificativa da escolha
- Configuração de armazenamento distribuído (se aplicável) com tratamento de condição de corrida
- Cabeçalhos de resposta padrão e comportamento no HTTP 429
- Mapa de limites por tier, se aplicável
</output_specification>

<quality_criteria>
Outputs excelentes:
- O rate limiter funciona corretamente com múltiplas instâncias da API rodando simultaneamente
- Toda resposta (inclusive as bem-sucedidas) inclui os cabeçalhos de limite restante
- O HTTP 429 sempre vem acompanhado de `Retry-After`
- A chave de limitação é apropriada ao tipo de endpoint (usuário para autenticado, IP para público)

Evite:
- Contador em memória local quando a API roda em múltiplas instâncias
- Janela fixa sem considerar o "efeito rajada na borda" quando o requisito exige distribuição uniforme
- Expor detalhes de implementação do rate limiter que ajudem um atacante a contorná-lo
- Aplicar o mesmo limite a endpoints com custo de processamento muito diferente
</quality_criteria>

<constraints>
- Nunca recomende armazenamento em memória local como solução definitiva para uma API que roda em mais de uma instância — sinalize isso como incorreto mesmo que o usuário não pergunte
- Não invente limites numéricos específicos sem que o usuário os informe ou aceite uma sugestão razoável explicitamente proposta
- Toda operação de leitura e escrita do contador em um armazenamento distribuído deve ser atômica, para evitar condições de corrida sob alta concorrência
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Nossa API Node.js roda em 4 instâncias atrás de um load balancer. Precisamos limitar usuários do plano free a 100 requisições/hora e usuários pro a 1000/hora no endpoint `/api/search`."

**Output esperado (resumo):**

- Escolha de sliding window com Redis, justificada pela necessidade de limite uniforme e pelo ambiente multi-instância
- Chave de limitação combinando `userId` + `/api/search`, com script Lua atômico para incrementar e verificar o limite no Redis
- Mapa de tiers: `free: 100/h`, `pro: 1000/h`
- Middleware Express retornando 429 com `Retry-After` calculado a partir do tempo restante da janela
- Cabeçalhos `X-RateLimit-Limit`, `X-RateLimit-Remaining` e `X-RateLimit-Reset` em toda resposta do endpoint
