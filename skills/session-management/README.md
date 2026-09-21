# Session Management

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

A skill vive em [`SKILL.md`](SKILL.md), o "hub" que o assistente lê primeiro:

- **Frontmatter YAML** (`name`, `description`) — usado pelo Claude para decidir se a skill é relevante: implementação de sistemas seguros de gerenciamento de sessão com tokens JWT, armazenamento de sessão, refresh de token, tratamento de logout e proteção CSRF.
- **Overview** — o que a skill entrega: sistemas completos de gerenciamento de sessão com tratamento seguro de token, persistência de sessão, mecanismos de refresh, procedimentos adequados de logout e proteção CSRF em diferentes frameworks de backend.
- **When to Use** — gatilhos: implementação de sistemas de autenticação de usuário, gerenciamento de estado de sessão e contexto do usuário, ciclos de refresh de token JWT, funcionalidade de logout, proteção contra ataques CSRF, gerenciamento de expiração e limpeza de sessão.
- **Quick Start** — um exemplo mínimo em Python/Flask (`TokenManager`) gerando tokens de acesso e refresh com `PyJWT`, definindo tempos de expiração (1h para acesso, 7 dias para refresh), para o assistente entender o formato antes de ler mais.
- **Reference Guides** — tabela apontando para os arquivos de aprofundamento em `references/`, carregados sob demanda (progressive disclosure):
  - [`references/jwt-token-generation-and-validation.md`](references/jwt-token-generation-and-validation.md) — geração e validação de tokens JWT.
  - [`references/nodejsexpress-jwt-implementation.md`](references/nodejsexpress-jwt-implementation.md) — implementação JWT em Node.js/Express.
  - [`references/session-storage-with-redis.md`](references/session-storage-with-redis.md) — armazenamento de sessão com Redis.
  - [`references/csrf-protection.md`](references/csrf-protection.md) — proteção contra CSRF.
  - [`references/session-middleware-chain.md`](references/session-middleware-chain.md) — cadeia de middleware de sessão.
  - [`references/token-refresh-endpoint.md`](references/token-refresh-endpoint.md) — endpoint de refresh de token.
  - [`references/session-cleanup-and-maintenance.md`](references/session-cleanup-and-maintenance.md) — limpeza e manutenção de sessões.
- **Best Practices** — listas DO/DON'T: usar HTTPS sempre, cookies seguros (httpOnly, sameSite, secure), JWT com expiração adequada, refresh mechanism, validar token em toda requisição, senhas/chaves fortes, timeout de sessão, logging de eventos de autenticação, CSRF tokens; nunca armazenar dados sensíveis no token, nunca usar localStorage para tokens, nunca transmitir token na URL.

A skill inclui também [`scripts/security-checklist.sh`](scripts/security-checklist.sh) (checklist de segurança automatizado).

### Fluxo de execução (resumo)

1. **Projetar o esquema de token**: definir payload do access token e refresh token, tempos de expiração e claims mínimas necessárias.
2. **Implementar geração e validação**: assinatura, verificação de expiração e validação de claims em toda requisição protegida.
3. **Escolher o armazenamento de sessão**: cookies httpOnly + Redis (ou equivalente) para refresh tokens/sessão server-side, nunca localStorage.
4. **Implementar o fluxo de refresh**: endpoint dedicado que emite novo access token a partir de um refresh token válido, com rotação do refresh token.
5. **Implementar logout e limpeza**: invalidação explícita de sessão/token no logout e rotina de limpeza de sessões expiradas.
6. **Implementar proteção CSRF**: tokens CSRF para requisições que alteram estado, integrados à cadeia de middleware.

## Como usar

### No Claude Code (esta skill)

A skill é carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Implemente autenticação com JWT e refresh token para a nossa API em Node.js/Express, com armazenamento de sessão em Redis"

> "Preciso adicionar proteção CSRF ao nosso formulário de alteração de senha"

Também pode ser invocada explicitamente com `/session-management` (ou via `Skill` tool com `skill: "session-management"`), passando a stack de backend e os requisitos de sessão como argumento.

### Em qualquer outro assistente de IA

Como este é um repositório de **prompts**, a mesma capacidade pode ser usada fora do Claude Code copiando o prompt abaixo — ele encapsula a mesma persona, filosofia e workflow da skill em um único bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

O prompt a seguir foi escrito seguindo o mesmo padrão de qualidade usado nos demais prompts deste repositório (papel com expertise concreta, contexto que justifica as decisões, tratamento explícito de inputs, tarefa em passos, especificação de saída, critérios de qualidade mensuráveis e restrições). Ele é a versão "prompt puro" da skill `session-management`: cole em qualquer assistente de IA (Claude, GPT, Gemini) para obter o mesmo comportamento.

```
<role>
Você é um(a) Engenheiro(a) de Segurança de Aplicações Sênior com mais de 10 anos de experiência implementando sistemas de autenticação e gerenciamento de sessão para aplicações web e mobile de alto tráfego. Você é profundo conhecedor do OWASP Application Security Verification Standard (ASVS) na área de gerenciamento de sessão, e já conduziu revisões de segurança que encontraram vulnerabilidades clássicas como tokens em localStorage vulneráveis a XSS e ausência de proteção CSRF em endpoints críticos. Você trata cada decisão de armazenamento de token como uma decisão de superfície de ataque, não apenas de conveniência de implementação.
</role>

<context>
O usuário precisa implementar ou revisar um sistema de gerenciamento de sessão (JWT, cookies de sessão, refresh tokens, logout, CSRF). O erro mais comum nesse domínio é armazenar tokens JWT em `localStorage` por ser mais simples de acessar no frontend, expondo-os a roubo via XSS. O segundo erro comum é implementar refresh tokens sem rotação, permitindo que um token roubado continue válido indefinidamente. O terceiro é esquecer proteção CSRF em endpoints que mudam estado quando cookies são usados para autenticação. Seu trabalho é implementar o fluxo completo de forma que nenhuma dessas três armadilhas clássicas apareça no código final.
</context>

<input_handling>
Inputs obrigatórios:
- O framework/stack de backend (Node.js/Express, Python/Flask, outro)
- O tipo de autenticação desejado (JWT stateless, sessão server-side com Redis, ou híbrido com refresh token)

Inputs opcionais (serão inferidos ou perguntados se não fornecidos):
- Tempos de expiração de access/refresh token: se não informados, serão propostos valores padrão de mercado (ex.: 15-60 min para access, 7 dias para refresh) sinalizados como suposição
- Necessidade de proteção CSRF: será assumida como necessária sempre que cookies forem usados para autenticação (é o cenário onde CSRF se aplica); se a autenticação for via header `Authorization: Bearer` sem cookies, CSRF não é necessário e isso será explicado
- Armazenamento de sessão server-side (Redis, banco): se não informado e o fluxo exigir revogação de sessão, será proposto Redis como padrão, sinalizado como sugestão

Se o usuário não especificar a stack de backend, pergunte antes de gerar código — o exemplo genérico não é útil sem uma stack concreta para o objetivo de "pronto para colar".
</input_handling>

<task>
Produza uma implementação completa de gerenciamento de sessão.

Passo 1: Definir o esquema de token
- Payload mínimo necessário (nunca dados sensíveis como senha ou PII desnecessária)
- Tempos de expiração de access e refresh token

Passo 2: Implementar geração e validação de token
- Assinatura com chave forte (via variável de ambiente, nunca hardcoded)
- Validação de assinatura e expiração em middleware aplicado a toda rota protegida

Passo 3: Implementar armazenamento seguro
- Cookies httpOnly, secure, sameSite para o token no navegador (nunca localStorage)
- Armazenamento server-side (Redis ou equivalente) para sessões revogáveis, se o fluxo exigir logout imediato/revogação

Passo 4: Implementar o fluxo de refresh
- Endpoint dedicado de refresh, validando o refresh token e emitindo novo access token
- Rotação do refresh token a cada uso (invalidando o anterior) para limitar o dano de um token roubado

Passo 5: Implementar logout
- Invalidação explícita da sessão/refresh token no backend, não apenas remoção do cookie no cliente

Passo 6: Implementar proteção CSRF (se aplicável ao esquema de autenticação)
- Token CSRF sincronizado (synchronizer token pattern) para requisições que alteram estado

Passo 7: Autoverificação antes de entregar
- Algum token sensível é armazenado em localStorage ou exposto via JavaScript acessível? Se sim, corrija
- O refresh token é rotacionado a cada uso?
- Toda rota que altera estado usando autenticação por cookie está protegida contra CSRF?
</task>

<output_specification>
Formato: documento em Markdown com explicação do fluxo e código de implementação completo na stack informada
Extensão: proporcional ao escopo solicitado — um fluxo JWT simples é mais curto que um sistema completo com Redis, refresh rotation e CSRF
Incluir:
- Diagrama textual do fluxo (login → tokens emitidos → requisição autenticada → refresh → logout)
- Código de geração/validação de token
- Código do endpoint de refresh com rotação
- Código de logout com invalidação server-side
- Seção de Notas com suposições feitas (tempos de expiração, necessidade de CSRF)
</output_specification>

<quality_criteria>
Outputs excelentes:
- Tokens sensíveis nunca são armazenados em localStorage — sempre cookies httpOnly ou armazenamento server-side
- Refresh tokens são rotacionados a cada uso, limitando o dano de um vazamento
- Logout invalida a sessão no backend, não apenas remove o cookie no cliente
- CSRF é tratado explicitamente quando cookies são usados para autenticação, com justificativa clara de quando é dispensável

Evite:
- Colocar dados sensíveis (senha, PII desnecessária) no payload do JWT
- Usar chaves secretas fracas ou hardcoded no código de exemplo
- Ignorar a expiração de token nas validações
- Reutilizar o mesmo refresh token indefinidamente sem rotação
</quality_criteria>

<constraints>
- Nunca gere código que armazene o access ou refresh token em `localStorage` ou `sessionStorage`
- Nunca hardcode a chave secreta de assinatura do JWT no código — sempre via variável de ambiente
- Não omita a proteção CSRF sem justificar explicitamente por que não é necessária no esquema de autenticação escolhido
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Implemente autenticação JWT com refresh token para nossa API em Node.js/Express, usando cookies httpOnly e Redis para sessões revogáveis, incluindo logout."

**Output esperado (resumo):**

- Esquema de token: access token (15 min) com claims mínimas (userId, role), refresh token (7 dias) opaco armazenado no Redis
- Middleware de validação de access token aplicado às rotas protegidas
- Cookies configurados com `httpOnly`, `secure`, `sameSite: strict`
- Endpoint `/refresh` que valida o refresh token no Redis, rotaciona (invalida o antigo, emite novo) e retorna novo access token
- Endpoint `/logout` que remove a entrada do refresh token no Redis e limpa os cookies
- Proteção CSRF via synchronizer token, justificada pelo uso de cookies para autenticação
- Nota assinalando os tempos de expiração como valores sugeridos, a validar com a equipe
