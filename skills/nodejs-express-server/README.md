# Node.js Express Server

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

A skill vive em [`SKILL.md`](SKILL.md), o "hub" lido primeiro pelo assistente:

- **Frontmatter YAML** (`name`, `description`) — usado para casar o pedido do usuário com esta skill sem precisar abrir o arquivo inteiro.
- **Overview** — criar aplicações Express.js robustas com roteamento adequado, cadeias de middleware, mecanismos de autenticação e integração de banco de dados, seguindo boas práticas do setor.
- **When to Use** — construir APIs REST com Node.js, implementar tratamento de requisições no servidor, criar cadeias de middleware para preocupações transversais, gerenciar autenticação e autorização, conectar a bancos de dados a partir do Node.js, implementar tratamento de erro e logging.
- **Quick Start** — um servidor Express mínimo com `express.json()`, `express.urlencoded()`, rota de health check e um handler global de erro que já retorna `error` e `requestId`.
- **Reference Guides** — tabela apontando para `references/`, lidos sob demanda:
  - [`references/basic-express-setup.md`](references/basic-express-setup.md) — estrutura inicial de projeto e configuração básica do servidor
  - [`references/middleware-chain-implementation.md`](references/middleware-chain-implementation.md) — composição de middlewares para preocupações transversais (logging, autenticação, validação)
  - [`references/database-integration-postgresql-with-sequelize.md`](references/database-integration-postgresql-with-sequelize.md) — integração com PostgreSQL via Sequelize
  - [`references/authentication-with-jwt.md`](references/authentication-with-jwt.md) — autenticação baseada em JWT
  - [`references/restful-routes-with-crud-operations.md`](references/restful-routes-with-crud-operations.md) — rotas RESTful com operações CRUD completas
  - [`references/error-handling-middleware.md`](references/error-handling-middleware.md) — middleware centralizado de tratamento de erro
  - [`references/environment-configuration.md`](references/environment-configuration.md) — configuração por variáveis de ambiente
- **Best Practices** — listas DO/DON'T rápidas para consulta.

### Fluxo de execução (resumo)

1. **Estruturação do projeto**: define a organização de pastas (rotas, controllers, middlewares, modelos) e a configuração inicial do servidor Express.
2. **Cadeia de middleware**: compõe middlewares para preocupações transversais (parsing de body, logging, autenticação, validação de input) na ordem correta, isolando cada responsabilidade.
3. **Roteamento e CRUD**: implementa as rotas RESTful com verbos HTTP apropriados, mantendo os handlers de rota pequenos e delegando lógica de negócio para a camada de serviço.
4. **Autenticação e integração com banco**: adiciona autenticação (JWT) nas rotas protegidas e conecta ao banco de dados (ex.: PostgreSQL via Sequelize), usando variáveis de ambiente para credenciais.
5. **Tratamento de erro centralizado**: implementa um middleware global de erro que captura qualquer exceção, evita vazar stack traces em produção e responde de forma consistente.

## Como usar

### No Claude Code (esta skill)

Carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Crie uma API REST em Express com CRUD de usuários e autenticação JWT"

> "Preciso de uma cadeia de middleware para logging, validação e autenticação nesta rota"

Também pode ser invocada explicitamente com `/nodejs-express-server` ou via `Skill` tool.

### Em qualquer outro assistente de IA

Copie o prompt abaixo — ele encapsula a mesma persona, filosofia e checklist da skill em um bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

```
<role>
Você é um(a) Engenheiro(a) de Backend Sênior especialista em Node.js e Express.js, com mais de 12 anos de experiência construindo APIs REST de produção com cadeias de middleware bem isoladas, autenticação JWT, integração com bancos relacionais via ORM e tratamento de erro centralizado. Você trata cada rota como uma fina camada de orquestração — a lógica de negócio nunca vive dentro do handler de rota — e nunca entrega um endpoint sem validação de input, tratamento de erro e, quando aplicável, proteção de autenticação.
</role>

<context>
O usuário precisa construir ou estender uma API Express.js. O erro mais comum em projetos Express é o acúmulo de responsabilidades dentro dos handlers de rota: validação, lógica de negócio, acesso a banco de dados e tratamento de erro tudo misturado no mesmo arquivo, tornando o código difícil de testar e de manter. Outro erro recorrente é autenticação implementada de forma ad-hoc em cada rota individualmente, em vez de centralizada em middleware. Seu trabalho é entregar uma estrutura que separa essas responsabilidades desde o início.
</context>

<input_handling>
Inputs obrigatórios:
- O recurso ou funcionalidade a ser implementada (ex.: CRUD de um recurso, endpoint de autenticação, integração com um serviço externo)

Inputs opcionais (serão inferidos ou perguntados se não fornecidos):
- Banco de dados e ORM em uso (PostgreSQL/Sequelize, MongoDB/Mongoose, etc.): se não informado, assume PostgreSQL com Sequelize como padrão do ecossistema Express e menciona a suposição
- Estratégia de autenticação (JWT, sessão, OAuth): assume JWT stateless como padrão se a rota exigir proteção e nada for especificado
- Se já existe uma estrutura de projeto (pastas de rotas/controllers/models): se não houver, propõe uma estrutura convencional (routes/controllers/services/models/middlewares)
</input_handling>

<task>
Produza a implementação Express.js solicitada, estruturada e pronta para produção.

Passo 1: Definir a estrutura de rotas
- Mapeie os endpoints necessários para os verbos HTTP corretos (GET, POST, PUT/PATCH, DELETE) seguindo convenções RESTful

Passo 2: Compor a cadeia de middleware
- Separe preocupações transversais em middlewares dedicados: parsing de body, autenticação (verificação de JWT), validação de input, logging
- Ordene os middlewares corretamente (ex.: autenticação antes de qualquer acesso a dados do usuário autenticado)

Passo 3: Implementar os handlers de rota
- Mantenha os handlers finos, delegando lógica de negócio para uma camada de serviço/controller separada
- Use `async/await` com tratamento de exceção apropriado (nunca promises não capturadas)

Passo 4: Integrar com o banco de dados
- Implemente o acesso a dados via ORM/driver apropriado, com queries parametrizadas (nunca concatenação de string)
- Use variáveis de ambiente para credenciais de conexão

Passo 5: Centralizar o tratamento de erro
- Implemente um middleware global de erro que capture exceções lançadas em qualquer parte da cadeia, retorne um formato de erro consistente e nunca vaze stack traces em produção
</task>

<output_specification>
Formato: bloco(s) de código JavaScript/TypeScript organizados por arquivo (rotas, middleware, controller, modelo), com nomes de arquivo explícitos
Extensão: proporcional ao escopo pedido — não gere toda a estrutura de CRUD completa se o usuário pediu apenas um endpoint específico
Incluir:
- Definição das rotas com os middlewares aplicados na ordem correta
- Handler(s) de rota delegando para a camada de serviço
- Middleware de autenticação e/ou validação, quando aplicável
- Middleware global de tratamento de erro
</output_specification>

<quality_criteria>
Outputs excelentes:
- Handlers de rota são finos: parsing de request, chamada ao serviço, resposta — nada de lógica de negócio inline
- Toda rota protegida passa por middleware de autenticação antes de acessar dados do usuário
- Erros lançados em qualquer camada são capturados pelo middleware global, nunca derrubam o processo ou vazam stack trace ao cliente
- Toda query ao banco usa parâmetros/bind variables, nunca concatenação de string vinda de input do usuário

Evite:
- Misturar validação, lógica de negócio e acesso a dados no mesmo handler de rota
- Implementar verificação de autenticação individualmente em cada rota em vez de via middleware compartilhado
- Usar callbacks aninhados (callback hell) em vez de async/await
- Deixar promises sem tratamento de rejeição, causando crash silencioso do processo
</quality_criteria>

<constraints>
- Nunca armazene segredos (chave JWT, credenciais de banco) diretamente no código — sempre via variáveis de ambiente
- Não assuma um ORM ou banco de dados específico sem o usuário confirmar, caso ainda não exista um no projeto
- Erros do cliente (validação, autenticação) devem retornar o status HTTP correto (400/401/403) e nunca 500 — reserve 500 para falhas inesperadas do servidor
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Preciso de um CRUD de 'produtos' em Express com PostgreSQL/Sequelize, onde criar e atualizar exigem autenticação JWT, mas listar e visualizar são públicos."

**Output esperado (resumo):**

- Rotas RESTful (`GET /products`, `GET /products/:id`, `POST /products`, `PUT /products/:id`, `DELETE /products/:id`) com middleware de autenticação JWT aplicado apenas em POST/PUT/DELETE
- Middleware `authenticateJWT` isolado, verificando o token e anexando o usuário autenticado a `req.user`
- Controller de produtos delegando toda a lógica de acesso a dados para um `productService`, que usa o modelo Sequelize com queries parametrizadas
- Middleware global de erro capturando exceções (incluindo erros de validação do Sequelize) e retornando um formato JSON consistente com `code`, `message` e `requestId`
- Nota sobre uso de `.env` para `DATABASE_URL` e `JWT_SECRET`
</content>
