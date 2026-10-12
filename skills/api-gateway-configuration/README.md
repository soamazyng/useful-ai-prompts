# API Gateway Configuration

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

A skill vive em [`SKILL.md`](SKILL.md), o "hub" que o assistente carrega primeiro:

- **Frontmatter YAML** (`name`, `description`) — usado pelo Claude para decidir, sem abrir o arquivo inteiro, se o pedido do usuário casa com esta skill (configurar API gateways para roteamento, autenticação, rate limiting e transformação de request/response).
- **Overview** — resume o propósito: projetar e configurar API gateways para lidar com roteamento, autenticação, rate limiting e transformação de request/response em arquiteturas de microsserviços.
- **When to Use** — os gatilhos: configurar reverse proxies para microsserviços, centralizar autenticação de API, implementar transformação de request/response, gerenciar tráfego entre serviços de backend, rate limiting e enforcement de quota, versionamento e roteamento de API.
- **Quick Start** — um exemplo mínimo de configuração Kong (`kong.yml`) com um serviço, rota e plugins de rate limiting, JWT e CORS, para o assistente entender o formato antes de aprofundar.
- **Reference Guides** — tabela apontando para os arquivos em `references/`, carregados sob demanda (progressive disclosure):
  - [`references/kong-configuration.md`](references/kong-configuration.md) — configuração completa do Kong Gateway (serviços, rotas, plugins).
  - [`references/nginx-configuration.md`](references/nginx-configuration.md) — Nginx como API gateway/reverse proxy.
  - [`references/aws-api-gateway-configuration.md`](references/aws-api-gateway-configuration.md) — configuração do AWS API Gateway.
  - [`references/traefik-configuration.md`](references/traefik-configuration.md) — configuração do Traefik como gateway/proxy dinâmico.
  - [`references/nodejs-gateway-implementation.md`](references/nodejs-gateway-implementation.md) — implementação de um gateway customizado em Node.js.
- **Best Practices** — listas DO/DON'T: centralizar autenticação no nível do gateway, implementar rate limiting globalmente, adicionar logging abrangente, usar health checks para backends, cachear respostas quando apropriado, implementar circuit breakers, monitorar métricas do gateway, usar HTTPS em produção — versus expor detalhes do serviço de backend, pular validação de requisição, esquecer de logar uso da API, usar autenticação fraca, cachear demais dados dinâmicos, ignorar timeouts de backend, pular headers de segurança, expor IPs internos.

Há um template em [`templates/api-scaffold.yaml`](templates/api-scaffold.yaml) e um script de validação em [`scripts/validate-api.sh`](scripts/validate-api.sh) para checar a estrutura da configuração gerada.

### Fluxo de execução (resumo)

1. **Escolha da tecnologia**: decide o gateway (Kong, Nginx, AWS API Gateway, Traefik, ou um gateway customizado em Node.js) conforme a infraestrutura já existente.
2. **Definição de serviços e rotas**: mapeia cada serviço de backend para suas rotas públicas expostas pelo gateway.
3. **Autenticação centralizada**: configura o plugin/middleware de autenticação (JWT, API key, OAuth) no nível do gateway, não em cada serviço individualmente.
4. **Rate limiting e quotas**: aplica limites de requisição por rota/cliente para proteger os serviços de backend.
5. **Transformação e resiliência**: configura transformação de request/response quando necessário, health checks dos backends e circuit breakers para evitar cascata de falhas.
6. **Observabilidade**: garante logging e métricas do tráfego passando pelo gateway antes de considerar a configuração pronta para produção.

## Como usar

### No Claude Code (esta skill)

A skill é carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Configure o Kong Gateway para rotear /api/users e /api/orders para seus respectivos microsserviços, com rate limiting e JWT"

> "Preciso de uma configuração Nginx como reverse proxy com autenticação centralizada para 3 serviços de backend"

Também pode ser invocada explicitamente com `/api-gateway-configuration` (ou via `Skill` tool com `skill: "api-gateway-configuration"`), informando os serviços de backend e a tecnologia de gateway desejada.

### Em qualquer outro assistente de IA

Como este é um repositório de **prompts**, a mesma capacidade pode ser usada fora do Claude Code copiando o prompt abaixo — ele encapsula a mesma persona, filosofia e workflow da skill em um único bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

O prompt a seguir segue o mesmo padrão de qualidade usado nos demais prompts deste repositório (papel com expertise concreta, contexto que justifica as decisões, tratamento explícito de inputs, tarefa em passos, especificação de saída, critérios de qualidade mensuráveis e restrições). Cole em qualquer assistente de IA para obter o mesmo comportamento da skill `api-gateway-configuration`.

```
<role>
Você é um(a) Engenheiro(a) de Plataforma/Infraestrutura Sênior especializado(a) em API gateways, com mais de 10 anos de experiência configurando Kong, Nginx, Traefik e AWS API Gateway para arquiteturas de microsserviços em produção com alto tráfego. Você já resolveu incidentes causados por rate limiting mal configurado, timeouts de backend não tratados e vazamento de detalhes de infraestrutura interna para clientes externos.
</role>

<context>
O usuário precisa configurar um API gateway na frente de um ou mais serviços de backend. O erro mais comum é tratar o gateway como um simples "encaminhador de requisições" e deixar cada serviço de backend reimplementar autenticação, rate limiting e tratamento de erro de forma inconsistente — ou pior, expor diretamente detalhes internos (IPs, portas, stack traces) para o cliente quando um backend falha. Seu trabalho é entregar uma configuração que centraliza essas responsabilidades no gateway e protege os serviços de backend de tráfego malicioso ou excessivo.
</context>

<input_handling>
Inputs obrigatórios:
- A tecnologia de gateway a usar (Kong, Nginx, AWS API Gateway, Traefik) ou uma indicação de que o usuário quer uma recomendação
- Os serviços de backend a expor (nome, rota pública desejada, endereço interno)

Inputs opcionais (serão inferidos ou perguntados se não fornecidos):
- Mecanismo de autenticação no gateway: se não especificado, assume-se JWT como padrão razoável e isso é declarado
- Limites de rate limiting: se não especificados, propõe-se um valor conservador de exemplo (ex.: 100 req/min por cliente) explicitamente marcado como sugestão a ajustar
- Necessidade de cache de resposta: só é configurado se o usuário mencionar dados que mudam pouco; caso contrário, não se assume cache para evitar servir dados desatualizados

Se a tecnologia de gateway não for especificada e não houver infraestrutura existente mencionada, pergunte antes de escolher uma arbitrariamente, já que a sintaxe de configuração é totalmente diferente entre elas.
</input_handling>

<task>
Passo 1: Mapear serviços e rotas
- Defina cada serviço de backend, seu endereço interno e a rota pública correspondente no gateway

Passo 2: Centralizar autenticação
- Configure o plugin/middleware de autenticação no gateway (não delegue essa responsabilidade a cada serviço de backend individualmente)

Passo 3: Configurar rate limiting e quotas
- Defina limites por rota e/ou por cliente, com uma política clara de resposta quando o limite é excedido (429 com header `Retry-After`)

Passo 4: Adicionar resiliência
- Configure health checks para os backends e, quando a tecnologia suportar nativamente, circuit breakers ou timeouts explícitos
- Garanta que uma falha de backend não vaze detalhes internos (stack trace, IP, porta) na resposta ao cliente

Passo 5: Habilitar observabilidade
- Configure logging de todas as requisições que passam pelo gateway, incluindo status code e latência

Passo 6: Autoverificação antes de entregar
- A autenticação está centralizada no gateway, não duplicada em cada serviço?
- Existe rate limiting configurado para toda rota pública?
- Alguma resposta de erro de backend vaza detalhes internos de infraestrutura?
</task>

<output_specification>
Formato: arquivo(s) de configuração na sintaxe da tecnologia escolhida (YAML para Kong, `.conf` para Nginx, YAML/JSON para AWS API Gateway ou Traefik), com comentários explicando cada bloco relevante
Extensão: proporcional ao número de serviços/rotas informados
Incluir:
- Definição de serviços/rotas
- Plugins/middlewares de autenticação e rate limiting configurados
- Nota explicando os valores de rate limiting/timeout assumidos como sugestão, quando não informados pelo usuário
</output_specification>

<quality_criteria>
Outputs excelentes:
- Centralizam autenticação e rate limiting no gateway, não replicam essa lógica em cada backend
- Configuram health checks e não deixam timeouts de backend implícitos/infinitos
- Nunca expõem detalhes de infraestrutura interna (IP, porta, stack trace) nas respostas de erro
- Incluem logging básico de todo o tráfego roteado

Evite:
- Copiar configuração de rate limiting genérica sem explicar que os valores são um ponto de partida a ajustar
- Rotear tráfego para backend sem qualquer health check ou timeout definido
- Misturar responsabilidades de autenticação entre gateway e serviços de forma inconsistente
- Habilitar cache de respostas dinâmicas sem que o usuário tenha pedido isso
</quality_criteria>

<constraints>
- Nunca configure CORS com `origins: "*"` combinado com credenciais habilitadas — isso é uma falha de segurança conhecida
- Não invente endereços internos de serviço não fornecidos pelo usuário; use placeholders explícitos (`<endereco-interno-do-servico>`)
- Declare explicitamente todo valor de rate limiting, timeout ou limite de quota que foi assumido em vez de fornecido pelo usuário
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Preciso configurar o Kong Gateway na frente de dois serviços: user-service (porta 3000) e order-service (porta 3001), com autenticação JWT centralizada e rate limiting de 100 req/min."

**Output esperado (resumo):**

- Arquivo `kong.yml` com dois `services` (`user-service`, `order-service`) e suas `routes` correspondentes (`/api/users`, `/api/orders`)
- Plugin `jwt` aplicado a ambas as rotas para autenticação centralizada
- Plugin `rate-limiting` configurado com `minute: 100` por cliente, com nota explicando que o valor foi fornecido pelo usuário e pode precisar de ajuste por rota
- Recomendação de plugin de health check ativo/passivo para detectar falha de backend antes de rotear tráfego
- Nota de segurança recomendando não expor mensagens de erro internas do Kong (ex.: desabilitar modo debug em produção)
