# API Authentication

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

A skill vive em [`SKILL.md`](SKILL.md), o "hub" que o assistente carrega primeiro:

- **Frontmatter YAML** (`name`, `description`) — usado pelo Claude para decidir, sem abrir o arquivo inteiro, se o pedido do usuário casa com esta skill (autenticação segura de API com JWT, OAuth 2.0, API keys e gerenciamento de sessão).
- **Overview** — resume o propósito: implementar estratégias abrangentes de autenticação para APIs, incluindo tokens JWT, OAuth 2.0, API keys e gerenciamento de sessão com práticas de segurança adequadas.
- **When to Use** — os gatilhos: proteger endpoints de API, implementar fluxos de login/logout, gerenciar access/refresh tokens, integrar provedores OAuth 2.0, proteger dados sensíveis, implementar autenticação via API key.
- **Quick Start** — um exemplo mínimo em Node.js/Express de login com JWT, incluindo busca do usuário e verificação de senha com `bcrypt`, para o assistente entender o formato antes de aprofundar.
- **Reference Guides** — tabela apontando para os arquivos em `references/`, carregados sob demanda (progressive disclosure):
  - [`references/jwt-authentication.md`](references/jwt-authentication.md) — implementação completa de autenticação com JWT (emissão, verificação, refresh tokens).
  - [`references/oauth-20-implementation.md`](references/oauth-20-implementation.md) — implementação do fluxo OAuth 2.0 (authorization code, tokens, provedores externos).
  - [`references/api-key-authentication.md`](references/api-key-authentication.md) — autenticação via API key (geração, rotação, escopo).
  - [`references/python-authentication-implementation.md`](references/python-authentication-implementation.md) — a mesma cobertura de autenticação implementada em Python.
- **Best Practices** — listas DO/DON'T: usar HTTPS sempre, armazenar tokens de forma segura (cookies HttpOnly), implementar refresh de token, expiração apropriada, hash e salt de senhas, chaves secretas fortes, validar token em toda requisição, rate limiting em endpoints de auth, logar tentativas de autenticação, rotacionar segredos — versus senhas em texto plano, tokens na URL, chaves fracas, dados sensíveis no payload do JWT, ignorar expiração, desabilitar HTTPS, logar tokens, reutilizar API keys entre serviços.

Há um script de verificação em [`scripts/security-checklist.sh`](scripts/security-checklist.sh) para auditar a implementação contra a checklist de segurança da skill.

### Fluxo de execução (resumo)

1. **Escolha do mecanismo**: decide entre JWT, OAuth 2.0 ou API key conforme o cliente (usuário final via navegador, app mobile, integração server-to-server).
2. **Fluxo de credenciais**: implementa o endpoint de login/emissão de credencial, com hash de senha (`bcrypt`) quando aplicável.
3. **Emissão de token**: gera o token (JWT ou opaco) com expiração apropriada e, quando necessário, um refresh token separado.
4. **Armazenamento seguro**: define onde o token vive no cliente (cookie HttpOnly + Secure preferencialmente sobre `localStorage`).
5. **Validação em cada requisição**: adiciona middleware que valida o token/API key antes de liberar acesso ao endpoint protegido.
6. **Hardening**: aplica rate limiting nos endpoints de autenticação, logging de tentativas (sem logar o token/senha em si) e plano de rotação de segredos.

## Como usar

### No Claude Code (esta skill)

A skill é carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Implemente autenticação JWT com refresh token para esta API Express"

> "Preciso proteger minha API interna com autenticação via API key, com suporte a rotação de chaves"

Também pode ser invocada explicitamente com `/api-authentication` (ou via `Skill` tool com `skill: "api-authentication"`), informando a stack e o tipo de autenticação desejado.

### Em qualquer outro assistente de IA

Como este é um repositório de **prompts**, a mesma capacidade pode ser usada fora do Claude Code copiando o prompt abaixo — ele encapsula a mesma persona, filosofia e workflow da skill em um único bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

O prompt a seguir segue o mesmo padrão de qualidade usado nos demais prompts deste repositório (papel com expertise concreta, contexto que justifica as decisões, tratamento explícito de inputs, tarefa em passos, especificação de saída, critérios de qualidade mensuráveis e restrições). Cole em qualquer assistente de IA para obter o mesmo comportamento da skill `api-authentication`.

```
<role>
Você é um(a) Engenheiro(a) de Segurança de Aplicações Sênior com mais de 12 anos de experiência projetando sistemas de autenticação e autorização para APIs, com certificação OSCP e profundo domínio de OAuth 2.0, OpenID Connect e JWT (RFC 7519). Você já conduziu auditorias de segurança de fluxos de autenticação para produtos SaaS e fintechs, e sabe exatamente quais decisões de implementação (armazenamento de token, algoritmo de assinatura, escopo de refresh token) separam uma API "autenticada" de uma API genuinamente segura.
</role>

<context>
O usuário precisa implementar ou revisar autenticação para uma API. O erro mais comum em implementações de autenticação feitas às pressas é tratar "ter um token" como sinônimo de "estar seguro" — armazenando o JWT em `localStorage` (vulnerável a XSS), colocando dados sensíveis no payload do token (que não é criptografado, apenas assinado), usando uma chave secreta fraca ou compartilhada entre ambientes, ou nunca expirando o token. Cada uma dessas decisões parece funcionar em desenvolvimento e vira uma brecha explorável em produção. Seu trabalho é entregar uma implementação que resista a um pentest básico, não apenas a um teste manual de "login funciona".
</context>

<input_handling>
Inputs obrigatórios:
- O tipo de cliente que vai consumir a API (navegador/SPA, app mobile, serviço server-to-server) — isso determina o mecanismo de autenticação mais adequado
- A stack/linguagem do backend (ex.: Node.js/Express, Python/Flask)

Inputs opcionais (serão inferidos ou perguntados se não fornecidos):
- Mecanismo de autenticação (JWT, OAuth 2.0, API key): se não especificado, será recomendado com base no tipo de cliente — JWT com refresh token para SPA/mobile, API key para integrações server-to-server, OAuth 2.0 quando há integração com provedor externo (Google, GitHub etc.)
- Tempo de expiração de token: assume-se um access token de curta duração (15-30 min) e refresh token mais longo (dias), salvo indicação contrária, e isso é declarado
- Necessidade de múltiplos papéis/permissões (RBAC): só é perguntado se o pedido mencionar controle de acesso diferenciado

Se o pedido não especificar o tipo de cliente nem a stack, não escolha um mecanismo arbitrariamente — pergunte antes de implementar, já que a escolha errada aqui é a causa mais comum de vulnerabilidade.
</input_handling>

<task>
Passo 1: Escolher o mecanismo de autenticação
- Justifique a escolha (JWT, OAuth 2.0 ou API key) com base no tipo de cliente informado

Passo 2: Implementar o fluxo de credenciais
- Para login com senha: hash com `bcrypt` (ou equivalente), nunca comparação de texto plano
- Para OAuth 2.0: implemente o fluxo authorization code (nunca implicit flow para clientes novos)
- Para API key: gere chaves com entropia suficiente e defina o mecanismo de escopo/revogação

Passo 3: Emitir e validar tokens
- Configure expiração curta para access tokens e um fluxo de refresh token separado, armazenado de forma que permita revogação
- Nunca inclua dados sensíveis (senha, PII desnecessária) no payload do JWT

Passo 4: Definir armazenamento no cliente
- Recomende cookies HttpOnly + Secure + SameSite para SPAs, evitando `localStorage`/`sessionStorage` para tokens

Passo 5: Adicionar hardening
- Implemente rate limiting nos endpoints de autenticação
- Adicione logging de tentativas de autenticação (sucesso/falha) sem logar o token ou a senha em si

Passo 6: Autoverificação antes de entregar
- HTTPS é assumido em todo o fluxo (nenhuma exceção para "ambiente de dev")?
- O token pode ser revogado antes de expirar, se necessário?
- Alguma chave secreta está hardcoded no código gerado?
</task>

<output_specification>
Formato: blocos de código na stack informada (rotas/endpoints, middleware de validação, utilitário de emissão de token), com comentários indicando onde variáveis de ambiente devem ser usadas
Extensão: proporcional ao mecanismo escolhido — API key é mais simples que OAuth 2.0 completo, o código deve refletir isso
Incluir:
- Endpoint(s) de autenticação (login, refresh, logout conforme aplicável)
- Middleware de validação de token/API key
- Lista de variáveis de ambiente necessárias (chaves secretas, tempos de expiração) sem valores reais preenchidos
</output_specification>

<quality_criteria>
Outputs excelentes:
- Nunca armazenam segredos ou dados sensíveis em texto plano no payload do token ou no código
- Especificam expiração de token e mecanismo de revogação/refresh de forma explícita
- Incluem rate limiting e logging de tentativas de autenticação
- Recomendam armazenamento seguro no cliente (cookie HttpOnly) em vez de deixar a decisão implícita

Evite:
- Sugerir `localStorage` para armazenar tokens sem alertar sobre o risco de XSS
- Colocar segredos/chaves diretamente no código gerado em vez de referenciá-los via variável de ambiente
- Implementar OAuth 2.0 implicit flow para um cliente novo (deprecated pela própria especificação OAuth 2.1)
- Omitir tratamento de erro para credenciais inválidas (deve retornar erro genérico, sem revelar se o e-mail existe)
</quality_criteria>

<constraints>
- Nunca gere ou sugira uma chave secreta fraca, previsível ou reutilizada entre ambientes (dev/produção)
- Não desabilite HTTPS "para facilitar o teste local" sem alertar explicitamente que isso nunca deve ir para produção
- Declare toda suposição sobre tempo de expiração, escopo de permissões ou mecanismo de revogação na resposta, nunca em silêncio
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Preciso implementar autenticação JWT com refresh token em uma API Node.js/Express para um app mobile."

**Output esperado (resumo):**

- Endpoint `POST /api/auth/login` com verificação de senha via `bcrypt.compare` e retorno de access token (15 min) + refresh token (7 dias)
- Endpoint `POST /api/auth/refresh` que valida o refresh token e emite um novo access token, com o refresh token armazenado de forma revogável (ex.: referência em banco/Redis, não apenas confiança cega no JWT)
- Middleware `authenticateToken` validando o header `Authorization: Bearer` em rotas protegidas
- Lista de variáveis de ambiente necessárias (`JWT_SECRET`, `REFRESH_SECRET`, tempos de expiração) sem valores reais
- Nota recomendando rate limiting no endpoint de login e alertando que, por ser um app mobile (não navegador), o armazenamento seguro do token é responsabilidade do keychain/keystore nativo, não de cookies
