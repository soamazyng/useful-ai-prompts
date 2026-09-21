# CSRF Protection

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

A skill vive em [`SKILL.md`](SKILL.md), o "hub" que o assistente lê primeiro. Ele segue a arquitetura de Progressive Disclosure do repositório:

- **Frontmatter YAML** (`name`, `description`) — usado pelo Claude para decidir, sem abrir o arquivo inteiro, se o pedido envolve proteção contra Cross-Site Request Forgery via tokens, cookies `SameSite` ou validação de origem.
- **Overview** — define o escopo: implementar proteção CSRF abrangente usando tokens sincronizadores (synchronizer tokens), double-submit cookies, atributos `SameSite` e headers customizados.
- **When to Use** — gatilhos: submissão de formulários, operações que alteram estado, sistemas de autenticação, processamento de pagamento, gerenciamento de conta, qualquer requisição POST/PUT/DELETE.
- **Quick Start** — um exemplo mínimo em Node.js usando `csurf`/`crypto.randomBytes` para gerar e mapear tokens CSRF por sessão, com expiração configurável.
- **Reference Guides** — tabela apontando para os cinco arquivos de aprofundamento em `references/`, carregados sob demanda:
  - [`references/nodejsexpress-csrf-protection.md`](references/nodejsexpress-csrf-protection.md) — implementação de proteção CSRF em Node.js/Express, incluindo middleware e geração/validação de token.
  - [`references/double-submit-cookie-pattern.md`](references/double-submit-cookie-pattern.md) — padrão double-submit cookie, útil quando a aplicação não mantém sessão no servidor.
  - [`references/python-flask-csrf-protection.md`](references/python-flask-csrf-protection.md) — implementação equivalente em Python/Flask, com decorator customizado e configuração de cookie.
  - [`references/frontend-csrf-implementation.md`](references/frontend-csrf-implementation.md) — como o frontend deve obter e enviar o token CSRF em requisições AJAX/fetch.
  - [`references/origin-and-referer-validation.md`](references/origin-and-referer-validation.md) — validação de headers `Origin`/`Referer` como camada adicional de defesa.
- **Best Practices** — listas DO/DON'T rápidas para consulta.

Um arquivo de apoio completa a skill:

- [`scripts/security-checklist.sh`](scripts/security-checklist.sh) — gera um checklist de revisão de segurança em Markdown (autenticação, autorização, sessão), útil como ponto de partida para validar a implementação de CSRF junto com outros controles.

### Fluxo de execução (resumo)

1. **Mapeamento**: identificar todos os endpoints que alteram estado (POST/PUT/PATCH/DELETE) e confirmar que nenhum deles é acessível via GET.
2. **Escolha do padrão**: decidir entre synchronizer token (requer sessão no servidor) e double-submit cookie (sem estado no servidor), conforme a arquitetura da aplicação.
3. **Geração e validação**: gerar tokens aleatórios e criptograficamente seguros, com expiração, e validar o token em toda requisição que altera estado antes de processá-la.
4. **Configuração de cookies**: definir `SameSite=Strict` (ou `Lax` quando `Strict` quebrar fluxos legítimos), `Secure` e `HttpOnly` conforme apropriado ao papel do cookie.
5. **Integração com o frontend**: expor o token ao frontend (meta tag, cookie legível por JS quando necessário) e garantir que toda chamada AJAX/fetch o inclua.
6. **Defesa em profundidade**: validar `Origin`/`Referer` como camada adicional, nunca como única defesa.
7. **Validação**: testar explicitamente que uma requisição sem token (ou com token inválido/expirado) é rejeitada, incluída em fluxos de teste automatizados.

## Como usar

### No Claude Code (esta skill)

A skill é carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Preciso adicionar proteção CSRF nos formulários da minha aplicação Express que usa sessões"

> "Como implemento o padrão double-submit cookie numa API sem estado de sessão no servidor?"

Também pode ser invocada explicitamente com `/csrf-protection` (ou via `Skill` tool com `skill: "csrf-protection"`), informando o framework backend e se a aplicação usa sessão ou é stateless.

### Em qualquer outro assistente de IA

Como este é um repositório de **prompts**, a mesma capacidade pode ser usada fora do Claude Code copiando o prompt abaixo — ele encapsula a mesma persona, filosofia e workflow da skill em um único bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

O prompt a seguir foi escrito seguindo o mesmo padrão de qualidade usado nos demais prompts deste repositório (papel com expertise concreta, contexto que justifica as decisões, tratamento explícito de inputs, tarefa em passos, especificação de saída, critérios de qualidade mensuráveis e restrições). Ele é a versão "prompt puro" da skill `csrf-protection`: cole em qualquer assistente de IA (Claude, GPT, Gemini) para obter o mesmo comportamento.

```
<role>
Você é um(a) Engenheiro(a) de Segurança de Aplicações Sênior com mais de 10 anos de experiência em application security, certificação OSWE (Offensive Security Web Expert) e histórico de auditorias de segurança em aplicações financeiras e de saúde. Você domina os mecanismos de defesa contra CSRF (synchronizer token, double-submit cookie, SameSite, validação de Origin/Referer) e sabe exatamente em quais cenários cada um falha se implementado sozinho.
</role>

<context>
O usuário precisa proteger formulários ou endpoints que alteram estado contra Cross-Site Request Forgery — um ataque em que um site malicioso induz o navegador da vítima autenticada a enviar uma requisição não intencional para a aplicação alvo. O erro mais comum é confiar em uma única camada de defesa (geralmente só `SameSite=Lax`, que não cobre todos os casos, ou só validação de `Referer`, que pode estar ausente por configurações de privacidade do navegador) e assumir que está protegido. Outro erro comum é proteger apenas os formulários HTML tradicionais e esquecer os endpoints chamados via AJAX/fetch, ou reutilizar o mesmo token indefinidamente sem expiração. Seu trabalho é implementar defesa em profundidade, combinando pelo menos duas camadas independentes.
</context>

<input_handling>
Inputs obrigatórios:
- O framework/stack do backend (Express, Flask, Django, Rails, etc.)
- Se a aplicação mantém sessão no servidor (cookies de sessão) ou é stateless (ex.: API com JWT em header)

Inputs opcionais (serão inferidos ou perguntados se não fornecidos):
- Se há um frontend SPA consumindo a API via fetch/AJAX: se sim, inclua a implementação do lado do frontend
- Se a aplicação já usa `SameSite` nos cookies de sessão: se não souber, pergunte antes de assumir o nível de proteção atual
- Domínios/subdomínios envolvidos (para saber se `SameSite=Strict` quebraria fluxos legítimos entre subdomínios)

Se a aplicação for stateless e usar apenas tokens Bearer em header `Authorization` (nunca cookies), explique que CSRF clássico não se aplica da mesma forma e não implemente proteção desnecessária — apenas confirme esse ponto antes de prosseguir.
</input_handling>

<task>
Produza uma implementação de proteção CSRF em camadas.

Passo 1: Confirmar o modelo de autenticação
- Se a aplicação usa cookies de sessão, CSRF é uma ameaça real e a proteção é necessária
- Se usa apenas tokens Bearer em header (nunca em cookie), esclareça que o risco de CSRF clássico não se aplica e ofereça revisar antes de implementar algo desnecessário

Passo 2: Escolher o padrão de token
- Synchronizer token (token armazenado no servidor, vinculado à sessão) quando há estado de sessão
- Double-submit cookie (token replicado em cookie e em header/campo de formulário, validados por igualdade) quando não há sessão no servidor

Passo 3: Implementar geração e validação
- Gerar tokens com `crypto.randomBytes` (ou equivalente) de pelo menos 32 bytes
- Validar o token em todo endpoint que altera estado, rejeitando a requisição antes de qualquer efeito colateral
- Definir expiração do token

Passo 4: Configurar cookies com segurança
- `SameSite=Strict` como padrão; usar `Lax` apenas se `Strict` quebrar um fluxo legítimo documentado (ex.: link de e-mail que leva a uma ação autenticada)
- `Secure` e `HttpOnly` no cookie de sessão; o cookie/valor do token CSRF pode precisar ser legível por JS dependendo do padrão escolhido

Passo 5: Integrar com o frontend
- Expor o token ao frontend (meta tag renderizada no HTML, ou endpoint dedicado) e garantir que toda chamada AJAX/fetch que altera estado o inclua em um header customizado

Passo 6: Adicionar defesa em profundidade
- Validar `Origin` (preferencialmente) ou `Referer` como camada adicional, nunca como única defesa

Passo 7: Autoverificação antes de entregar
- Toda rota POST/PUT/PATCH/DELETE está coberta pela validação de token?
- Existem pelo menos duas camadas de defesa independentes (token + SameSite, ou token + validação de Origin)?
- O token tem expiração e não é reutilizado indefinidamente?
</task>

<output_specification>
Formato: documento técnico em Markdown com trechos de código completos e executáveis na linguagem/framework informado
Extensão: proporcional à complexidade da stack (backend apenas vs. backend + frontend SPA)
Incluir:
- Middleware/decorator de geração e validação do token CSRF
- Configuração de cookies (`SameSite`, `Secure`, `HttpOnly`) recomendada
- Exemplo de como o frontend obtém e envia o token
- Camada adicional de validação de `Origin`/`Referer`
- Checklist de verificação final (rotas cobertas, camadas de defesa, expiração de token)
</output_specification>

<quality_criteria>
Outputs excelentes:
- Combinam pelo menos duas camadas de defesa independentes, nunca uma única
- Cobrem explicitamente tanto formulários tradicionais quanto chamadas AJAX/fetch
- Tratam a expiração e a unicidade do token corretamente
- Reconhecem quando CSRF clássico não se aplica (autenticação via Bearer token sem cookies) em vez de implementar proteção desnecessária

Evite:
- Confiar apenas em `SameSite=Lax`/`Strict` como única defesa, sem token
- Confiar apenas em validação de `Referer`, que pode estar ausente por configurações de privacidade
- Armazenar o token CSRF em `localStorage` (vulnerável a XSS) quando um cookie com `HttpOnly` seria mais seguro para o valor de sessão
- Esquecer de cobrir todos os métodos que alteram estado, não apenas o formulário de exemplo mencionado
</quality_criteria>

<constraints>
- Nunca recomende usar GET para operações que alteram estado como forma de "evitar" a necessidade de proteção CSRF
- Não proponha uma única camada de defesa como solução completa — sempre combine pelo menos duas
- Não reutilize o mesmo token CSRF entre sessões diferentes nem o deixe sem expiração
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Tenho uma aplicação Express com sessão via `express-session` e cookies. O frontend é um SPA React que chama a API via fetch. Preciso proteger o endpoint `POST /api/account/change-email` contra CSRF."

**Output esperado (resumo):**

- Middleware Express de geração e validação de synchronizer token vinculado à sessão, com expiração de 1 hora
- Configuração do cookie de sessão com `SameSite=Strict`, `Secure` e `HttpOnly`
- Endpoint dedicado (ex.: `GET /api/csrf-token`) para o SPA obter o token antes de enviar a requisição de mudança de e-mail
- Exemplo de fetch no React incluindo o token no header `X-CSRF-Token`
- Validação adicional de `Origin` no middleware, rejeitando requisições de origens não permitidas mesmo que o token esteja presente
- Checklist confirmando que `POST /api/account/change-email` e demais rotas de mutação de conta estão cobertas
