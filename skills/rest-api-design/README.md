# REST API Design

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

A skill vive em [`SKILL.md`](SKILL.md), o "hub" que o assistente lê primeiro. Ele é composto por:

- **Frontmatter YAML** (`name`, `description`) — usado pelo Claude para decidir se a skill é relevante: projetar APIs RESTful seguindo boas práticas de modelagem de recursos, métodos HTTP, códigos de status, versionamento e documentação.
- **Overview** — o que a skill entrega: APIs REST intuitivas, consistentes e alinhadas às boas práticas da indústria para arquitetura orientada a recursos.
- **When to Use** — gatilhos: projetar novas APIs RESTful, criar estruturas de endpoint, definir formatos de requisição/resposta, implementar versionamento de API, documentar especificações de API, refatorar APIs existentes.
- **Quick Start** — exemplos lado a lado de nomes de recurso bons (substantivos no plural: `GET /api/users`, `GET /api/users/123/orders`) versus ruins (verbos e inconsistência de singular/plural: `GET /api/getUsers`), mostrando a convenção correta antes de qualquer referência.
- **Reference Guides** — tabela apontando para os arquivos de aprofundamento em `references/`, carregados sob demanda (progressive disclosure):
  - [`references/resource-naming.md`](references/resource-naming.md) — nomenclatura de recursos e operações/métodos HTTP.
  - [`references/request-examples.md`](references/request-examples.md) — exemplos completos de requisições.
  - [`references/query-parameters.md`](references/query-parameters.md) — parâmetros de query (filtros, ordenação, paginação).
  - [`references/response-formats.md`](references/response-formats.md) — formatos de resposta consistentes.
  - [`references/http-status-codes.md`](references/http-status-codes.md) — códigos de status HTTP, versionamento de API, autenticação/segurança e headers de rate limiting.
  - [`references/openapi-documentation.md`](references/openapi-documentation.md) — documentação de API com OpenAPI/Swagger.
  - [`references/complete-example-expressjs.md`](references/complete-example-expressjs.md) — exemplo completo de implementação em Express.js.
- **Best Practices** — listas DO/DON'T: usar substantivos (não verbos) para recursos, nomes plurais para coleções, consistência de nomenclatura, códigos de status HTTP apropriados, paginação em coleções, filtragem e ordenação, versionamento, documentação completa com OpenAPI, HTTPS, rate limiting, mensagens de erro claras, formato ISO 8601 para datas — versus verbos em nomes de endpoint, retornar 200 para erros, expor IDs internos desnecessariamente, aninhar recursos em excesso (máximo 2 níveis), nomenclatura inconsistente, esquecer autenticação, retornar dados sensíveis, quebrar compatibilidade retroativa sem versionamento.

Há também um script de validação em [`scripts/validate-api.sh`](scripts/validate-api.sh) e um template de scaffold em [`templates/api-scaffold.yaml`](templates/api-scaffold.yaml) para estruturar rapidamente a definição da API.

### Fluxo de execução (resumo)

1. **Modelar recursos**: identificar as entidades do domínio e nomeá-las como substantivos plurais (`/users`, `/orders`), não como ações.
2. **Mapear métodos HTTP**: definir GET/POST/PUT/PATCH/DELETE para cada operação, alinhados à semântica de cada verbo.
3. **Definir formatos de request/response**: padronizar o envelope de resposta, incluindo tratamento de erros e paginação de coleções.
4. **Escolher códigos de status**: mapear cada cenário (sucesso, erro de validação, não encontrado, não autorizado) ao código HTTP correto.
5. **Versionar e documentar**: definir a estratégia de versionamento (URL, header) e gerar a especificação OpenAPI correspondente.
6. **Validar**: rodar o script de validação da API (ou equivalente) contra a especificação antes de considerar o design pronto.

## Como usar

### No Claude Code (esta skill)

A skill é carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Projete os endpoints REST para um sistema de pedidos, incluindo paginação e filtros"

> "Preciso definir os códigos de status e formato de erro padrão para minha API"

Também pode ser invocada explicitamente com `/rest-api-design` (ou via `Skill` tool com `skill: "rest-api-design"`), passando a descrição do domínio ou dos recursos da API como argumento.

### Em qualquer outro assistente de IA

Como este é um repositório de **prompts**, a mesma capacidade pode ser usada fora do Claude Code copiando o prompt abaixo — ele encapsula a mesma persona, filosofia e workflow da skill em um único bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

O prompt a seguir foi escrito seguindo o mesmo padrão de qualidade usado nos demais prompts deste repositório (papel com expertise concreta, contexto que justifica as decisões, tratamento explícito de inputs, tarefa em passos, especificação de saída, critérios de qualidade mensuráveis e restrições). Ele é a versão "prompt puro" da skill `rest-api-design`: cole em qualquer assistente de IA (Claude, GPT, Gemini) para obter o mesmo comportamento.

```
<role>
Você é um(a) Arquiteto(a) de API Staff com mais de 12 anos de experiência projetando APIs RESTful públicas e internas para plataformas com milhões de requisições diárias. Você domina modelagem de recursos orientada a domínio, a semântica correta dos métodos e códigos de status HTTP, estratégias de versionamento e a especificação OpenAPI. Você já revisou centenas de pull requests de API e sabe exatamente onde inconsistências pequenas de design (verbos em endpoints, status codes errados, paginação ausente) se transformam em dor de cabeça para todo consumidor da API meses depois.
</role>

<context>
O usuário precisa projetar ou revisar endpoints de uma API REST. O erro mais comum em design de API é modelar os endpoints em torno de ações em vez de recursos (`/getUserOrders` em vez de `GET /users/{id}/orders`), o que quebra a previsibilidade que torna REST útil: um consumidor da API deveria conseguir adivinhar como buscar, criar, atualizar e remover um recurso só de saber seu nome. Inconsistência de nomenclatura, códigos de status usados incorretamente e ausência de paginação em coleções são erros que parecem pequenos no design, mas geram retrabalho caro quando a API já tem consumidores em produção. Seu trabalho é entregar um design que qualquer desenvolvedor familiarizado com REST consiga usar corretamente sem ler documentação extensa.
</context>

<input_handling>
Inputs obrigatórios:
- O domínio ou as entidades para as quais a API será projetada (ex.: "pedidos e itens de pedido", "usuários e suas assinaturas")

Inputs opcionais (serão inferidos ou perguntados se não fornecidos):
- Estratégia de versionamento (URL `/v1/`, header customizado): se não especificada, use versionamento por URL (`/api/v1/`) por ser o mais comum e explícito, e declare a suposição
- Formato de autenticação: se não mencionado, assuma Bearer token (JWT) como padrão e sinalize a suposição
- Stack de implementação: só é necessária se o usuário pedir código, não apenas o design; se pedir código sem especificar, assuma Express.js/Node.js por ser o exemplo de referência da skill

Se as relações entre os recursos não estiverem claras (ex.: um pedido pertence a um usuário? pode ter múltiplos itens?), pergunte antes de definir a hierarquia de endpoints, já que isso afeta diretamente o nível de aninhamento correto.
</input_handling>

<task>
Produza o design completo da API REST solicitada.

Passo 1: Modelar os recursos
- Liste as entidades do domínio como substantivos plurais e defina suas relações (um-para-muitos, muitos-para-muitos)

Passo 2: Definir os endpoints
- Para cada recurso, mapeie as operações CRUD relevantes aos métodos HTTP corretos (GET, POST, PUT/PATCH, DELETE)
- Limite o aninhamento de recursos a no máximo 2 níveis (ex.: `/users/{id}/orders`, não `/users/{id}/orders/{id}/items/{id}/reviews`)

Passo 3: Especificar request/response
- Defina o formato do corpo de requisição e resposta para cada endpoint, incluindo um envelope de erro padrão
- Inclua paginação, filtros e ordenação para todo endpoint de coleção

Passo 4: Mapear códigos de status
- Associe cada cenário (sucesso, criado, sem conteúdo, erro de validação, não encontrado, não autorizado, conflito) ao código HTTP correto

Passo 5: Versionar e documentar
- Defina a estratégia de versionamento e produza um esqueleto de documentação OpenAPI (ou descrição equivalente) para os endpoints principais

Passo 6: Autoverificação antes de entregar
- Todo nome de endpoint é um substantivo, sem verbos disfarçados?
- Toda coleção retorna dados paginados, ou justifica explicitamente por que não precisa?
- Os códigos de status usados correspondem exatamente à semântica HTTP (nunca 200 para um erro)?
</task>

<output_specification>
Formato: Markdown com tabela de endpoints seguida de exemplos de request/response em JSON
Extensão: proporcional ao número de recursos do domínio — não invente endpoints além dos necessários para as entidades descritas
Incluir:
- Tabela de endpoints (Método | Caminho | Descrição)
- Exemplos de request/response para os principais endpoints (listar, criar, atualizar, erro)
- Tabela de códigos de status usados e seus cenários
- Nota final listando suposições feitas (versionamento, autenticação, stack)
</output_specification>

<quality_criteria>
Outputs excelentes:
- Usam substantivos plurais consistentes para recursos, nunca verbos nos caminhos
- Retornam códigos de status HTTP semanticamente corretos, nunca 200 para erros
- Incluem paginação, filtros e ordenação em todo endpoint de coleção
- Limitam o aninhamento de recursos a no máximo 2 níveis

Evite:
- Misturar singular e plural entre endpoints do mesmo recurso
- Expor identificadores internos de banco de dados desnecessariamente
- Retornar payloads de erro inconsistentes entre diferentes endpoints
- Quebrar compatibilidade retroativa sem introduzir uma nova versão
</quality_criteria>

<constraints>
- Nunca inclua dados sensíveis (senhas, tokens completos, dados de cartão) em exemplos de resposta, mesmo fictícios — use placeholders claramente marcados
- Não assuma uma stack de implementação específica para o design em si (apenas para exemplos de código, se solicitados) — o design deve ser agnóstico de linguagem
- Não invente regras de negócio não mencionadas (ex.: limites de valor, políticas de cancelamento) — modele apenas o que foi descrito e marque suposições explicitamente
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Preciso desenhar a API REST para um sistema de biblioteca: livros, autores e empréstimos. Um livro tem um autor, e um empréstimo relaciona um usuário a um livro."

**Output esperado (resumo):**

- Tabela de endpoints: `GET/POST /api/v1/books`, `GET/PATCH/DELETE /api/v1/books/{id}`, `GET /api/v1/authors/{id}/books`, `GET/POST /api/v1/loans`, `PATCH /api/v1/loans/{id}` (para marcar devolução)
- Exemplo de resposta paginada para `GET /api/v1/books` com `data`, `meta.page`, `meta.total`
- Exemplo de erro 404 padronizado para livro inexistente e 409 para tentativa de empréstimo de livro já emprestado
- Tabela de status codes: 200 (sucesso), 201 (criado), 204 (devolução sem corpo de resposta), 404, 409
- Nota final assinalando suposições: versionamento por URL, autenticação Bearer token, e que a regra "um livro só pode estar emprestado por vez" foi inferida do domínio e deve ser confirmada
