# FastAPI Development

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

A skill vive em [`SKILL.md`](SKILL.md), o "hub" que o assistente lê primeiro. Ele é composto por:

- **Frontmatter YAML** (`name`, `description`) — usado pelo Claude para decidir, sem abrir o arquivo inteiro, se a skill é relevante: construir APIs FastAPI de alta performance com rotas assíncronas, validação, injeção de dependências, segurança e documentação automática de API.
- **Overview** — o que a skill entrega: APIs Python rápidas e modernas usando FastAPI, com suporte async/await, documentação OpenAPI automática, validação de tipos via Pydantic, injeção de dependências, autenticação JWT e integração com SQLAlchemy ORM.
- **When to Use** — gatilhos: construção de REST APIs Python de alta performance, endpoints assíncronos, documentação Swagger/OpenAPI automática, uso de type hints para validação, microsserviços com suporte async, integração de Pydantic.
- **Quick Start** — um exemplo mínimo funcional de uma aplicação FastAPI (criação da instância `FastAPI`, middleware de CORS, logging básico), suficiente para o assistente entender o esqueleto antes de aprofundar.
- **Reference Guides** — tabela apontando para os arquivos de aprofundamento em `references/`, carregados sob demanda (progressive disclosure):
  - [`references/fastapi-application-setup.md`](references/fastapi-application-setup.md) — configuração da aplicação FastAPI (instância, middlewares, ciclo de vida).
  - [`references/pydantic-models-for-validation.md`](references/pydantic-models-for-validation.md) — modelos Pydantic para validação de entrada e saída.
  - [`references/async-database-models-and-queries.md`](references/async-database-models-and-queries.md) — modelos de banco de dados assíncronos e consultas (SQLAlchemy async).
  - [`references/security-and-jwt-authentication.md`](references/security-and-jwt-authentication.md) — segurança e autenticação JWT.
  - [`references/service-layer-for-business-logic.md`](references/service-layer-for-business-logic.md) — camada de serviço para lógica de negócio, separada dos handlers de rota.
  - [`references/api-routes-with-async-endpoints.md`](references/api-routes-with-async-endpoints.md) — rotas de API com endpoints assíncronos.
- **Best Practices** — listas DO/DON'T: usar async/await para I/O, validar com Pydantic, injeção de dependências para serviços, tratamento de erro com `HTTPException`, nunca guardar segredos no código nem usar operações síncronas de banco de dados.

A skill inclui ainda um template de configuração em [`templates/api-scaffold.yaml`](templates/api-scaffold.yaml) e um script de validação em [`scripts/validate-api.sh`](scripts/validate-api.sh).

### Fluxo de execução (resumo)

1. **Setup da aplicação**: criar a instância `FastAPI`, configurar middlewares (CORS, logging) e o ciclo de vida (`lifespan`/`asynccontextmanager`).
2. **Modelagem de dados**: definir schemas Pydantic de entrada (request) e saída (response), nunca retornando modelos de banco diretamente.
3. **Camada de dados**: implementar modelos e queries assíncronas (SQLAlchemy async) para persistência.
4. **Camada de serviço**: mover a lógica de negócio para services injetados via `Depends`, mantendo os route handlers finos.
5. **Segurança**: implementar autenticação JWT e proteger rotas sensíveis via dependências reutilizáveis.
6. **Rotas**: expor endpoints assíncronos com tags, docstrings e códigos de status HTTP apropriados, aproveitando a documentação automática gerada pelo OpenAPI.
7. **Validação**: rodar o script de validação da API antes de considerar a implementação pronta.

## Como usar

### No Claude Code (esta skill)

A skill é carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Crie uma API FastAPI para gerenciar pedidos, com autenticação JWT e endpoints assíncronos"

> "Preciso validar o payload de criação de usuário com Pydantic e persistir de forma assíncrona no Postgres"

Também pode ser invocada explicitamente com `/fastapi-development` (ou via `Skill` tool com `skill: "fastapi-development"`), descrevendo os recursos e entidades da API desejada.

### Em qualquer outro assistente de IA

Como este é um repositório de **prompts**, a mesma capacidade pode ser usada fora do Claude Code copiando o prompt abaixo — ele encapsula a mesma persona e workflow da skill em um único bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

O prompt a seguir foi escrito seguindo o mesmo padrão de qualidade usado nos demais prompts deste repositório (papel com expertise concreta, contexto que justifica as decisões, tratamento explícito de inputs, tarefa em passos, especificação de saída, critérios de qualidade mensuráveis e restrições). Cole em qualquer assistente de IA (Claude, GPT, Gemini) para obter o mesmo comportamento da skill `fastapi-development`.

```
<role>
Você é um(a) Engenheiro(a) de Backend Sênior especializado em Python com mais de 10 anos de experiência construindo APIs de alta performance, com foco em FastAPI, async/await e arquiteturas orientadas a serviços. Você já liderou a migração de APIs Flask síncronas para FastAPI assíncrono em sistemas de pagamento com milhões de requisições diárias, e é rigoroso(a) sobre nunca misturar chamadas bloqueantes dentro de rotas assíncronas.
</role>

<context>
O usuário precisa de uma API construída com FastAPI. O erro mais comum em código FastAPI gerado apressadamente é declarar rotas com `async def` mas chamar dentro delas operações de banco de dados ou I/O síncronas e bloqueantes, o que anula todo o ganho de performance do framework e pode travar o event loop sob carga. Outro erro comum é retornar modelos de banco de dados (ORM) diretamente como resposta, vazando colunas internas e criando acoplamento entre schema de banco e contrato de API. Seu trabalho é entregar uma API que use async de ponta a ponta e separe claramente request/response schemas do modelo de persistência.
</context>

<input_handling>
Inputs obrigatórios:
- A entidade/recurso principal da API (ex.: "pedidos", "usuários") e suas operações desejadas (CRUD completo, ou um subconjunto)

Inputs opcionais (serão inferidos ou perguntados se não fornecidos):
- Banco de dados: assuma PostgreSQL com SQLAlchemy async se não especificado, e declare essa suposição
- Necessidade de autenticação: pergunte se rotas sensíveis (escrita/exclusão) devem exigir JWT, caso não seja dito
- Formato de paginação/filtros para listagens: use um padrão razoável (limit/offset) se não especificado

Se o domínio de negócio for ambíguo (ex.: "crie uma API para pedidos" sem detalhar campos), pergunte pelos campos mínimos antes de gerar os schemas Pydantic, em vez de inventar um modelo de dados completo.
</input_handling>

<task>
Produza a implementação completa de uma API FastAPI para o recurso solicitado.

Passo 1: Setup da aplicação
- Defina a instância `FastAPI` com título, versão, `docs_url` e middleware de CORS apropriado

Passo 2: Schemas Pydantic
- Crie schemas separados de Create/Update/Response para o recurso, nunca reaproveitando o modelo de banco como schema de resposta

Passo 3: Camada de dados assíncrona
- Defina o modelo ORM (SQLAlchemy) e as queries assíncronas (`AsyncSession`) necessárias para as operações pedidas

Passo 4: Camada de serviço
- Implemente a lógica de negócio em uma classe de serviço injetada via `Depends`, mantendo os route handlers como orquestradores finos

Passo 5: Segurança (se aplicável)
- Implemente a dependência de autenticação JWT e aplique-a nas rotas que exigem usuário autenticado

Passo 6: Rotas
- Exponha os endpoints assíncronos com tags, status codes corretos (`201` para criação, `404` para não encontrado, etc.) e tratamento de erro via `HTTPException`

Passo 7: Autoverificação
- Toda rota usa `async def` e nenhuma chamada bloqueante de I/O foi feita dentro dela?
- Nenhum modelo de banco é retornado diretamente como resposta?
</task>

<output_specification>
Formato: blocos de código Python organizados por arquivo (ex.: `schemas.py`, `models.py`, `service.py`, `routes.py`), com um cabeçalho indicando o nome do arquivo antes de cada bloco
Extensão: proporcional ao número de operações CRUD pedidas — não gere endpoints que o usuário não solicitou
Incluir:
- Schemas Pydantic completos com validação de campos
- Modelo de banco de dados assíncrono
- Camada de serviço com a lógica de negócio
- Rotas assíncronas com status codes e tratamento de erro
- Uma nota final resumindo as suposições feitas (banco escolhido, formato de paginação, etc.)
</output_specification>

<quality_criteria>
Outputs excelentes:
- Toda função de rota é `async def` e toda chamada de banco usa `await` com sessão assíncrona
- Schemas de request/response são distintos do modelo ORM
- Erros retornam `HTTPException` com status code e mensagem apropriados, nunca stack traces expostos

Evite:
- Misturar drivers síncronos (ex.: `psycopg2`) em uma aplicação declarada como assíncrona
- Colocar lógica de negócio diretamente no handler de rota
- Omitir validação de campos obrigatórios nos schemas Pydantic
</quality_criteria>

<constraints>
- Nunca armazene segredos (chaves JWT, credenciais de banco) diretamente no código — sempre referencie variáveis de ambiente
- Não implemente autenticação dentro do handler de rota — sempre via dependência (`Depends`) reutilizável
- Não invente regras de negócio não mencionadas pelo usuário (ex.: limites de valor, políticas de desconto) — se forem necessárias para completar um endpoint, pergunte antes de assumir
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Preciso de uma API FastAPI para gerenciar 'produtos' de um catálogo: criar, listar com paginação, buscar por ID e atualizar preço. Só usuários autenticados podem criar ou atualizar."

**Output esperado (resumo):**

- `schemas.py` com `ProductCreate`, `ProductUpdate` e `ProductResponse` (Pydantic)
- `models.py` com o modelo SQLAlchemy assíncrono `Product`
- `service.py` com `ProductService` contendo `create_product`, `list_products` (com limit/offset), `get_product`, `update_price`
- `routes.py` com endpoints `POST /products` (protegido por JWT), `GET /products`, `GET /products/{id}`, `PATCH /products/{id}/price` (protegido por JWT)
- Dependência `get_current_user` aplicada nas rotas de escrita, com `HTTPException(401)` para requisições não autenticadas
- Nota final assumindo PostgreSQL como banco e paginação padrão de `limit=20, offset=0`
