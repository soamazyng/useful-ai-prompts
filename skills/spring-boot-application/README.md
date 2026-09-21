# Spring Boot Application

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

A skill vive em [`SKILL.md`](SKILL.md), o "hub" que o assistente lê primeiro:

- **Frontmatter YAML** (`name`, `description`) — usado pelo Claude para decidir se a skill é relevante: construção de aplicações Spring Boot empresariais com anotações, injeção de dependência, persistência de dados, controllers REST e segurança.
- **Overview** — o que a skill entrega: aplicações Spring Boot prontas para produção, com configuração baseada em anotações, injeção de dependência, controllers REST, persistência JPA, camada de serviço e implementação de segurança seguindo as convenções do Spring.
- **When to Use** — gatilhos: construção de APIs REST em Spring Boot, implementação de arquiteturas orientadas a serviço, configuração de persistência de dados com JPA, gerenciamento de injeção de dependência, implementação de Spring Security, construção de microsserviços com Spring Boot.
- **Quick Start** — um exemplo mínimo de `pom.xml` com o parent `spring-boot-starter-parent` e as dependências `spring-boot-starter-web`, `spring-boot-starter-data-jpa` e `spring-boot-starter-security`, para o assistente entender o formato antes de ler mais.
- **Reference Guides** — tabela apontando para os arquivos de aprofundamento em `references/`, carregados sob demanda (progressive disclosure):
  - [`references/spring-boot-project-setup.md`](references/spring-boot-project-setup.md) — configuração inicial do projeto Spring Boot.
  - [`references/entity-models-with-jpa-annotations.md`](references/entity-models-with-jpa-annotations.md) — modelos de entidade com anotações JPA.
  - [`references/repository-layer-with-spring-data-jpa.md`](references/repository-layer-with-spring-data-jpa.md) — camada de repositório com Spring Data JPA.
  - [`references/service-layer-with-business-logic.md`](references/service-layer-with-business-logic.md) — camada de serviço com lógica de negócio.
  - [`references/rest-controllers-with-requestresponse-handling.md`](references/rest-controllers-with-requestresponse-handling.md) — controllers REST com tratamento de request/response.
  - [`references/spring-security-configuration.md`](references/spring-security-configuration.md) — configuração do Spring Security.
  - [`references/application-configuration.md`](references/application-configuration.md) — configuração da aplicação.
- **Best Practices** — listas DO/DON'T: usar injeção de dependência para baixo acoplamento, camada de serviço para lógica de negócio, repositórios para acesso a dados, Spring Security para autenticação, `@Transactional` para transações, validar input nos controllers, DTOs para request/response, tratamento de exceção adequado; nunca colocar lógica de negócio no controller, nunca acessar o banco diretamente no controller, nunca retornar entidades de banco diretamente na API.

### Fluxo de execução (resumo)

1. **Configurar o projeto**: `pom.xml`/`build.gradle` com as starters necessárias (web, data-jpa, security) e `application.yml`/`application.properties`.
2. **Modelar as entidades JPA**: classes de entidade com anotações (`@Entity`, `@Id`, relacionamentos) refletindo o domínio.
3. **Implementar a camada de repositório**: interfaces `JpaRepository`/`CrudRepository` com queries derivadas ou customizadas.
4. **Implementar a camada de serviço**: lógica de negócio isolada dos controllers, com `@Transactional` onde necessário.
5. **Implementar os controllers REST**: endpoints usando DTOs para request/response, validação de input, status HTTP apropriados.
6. **Configurar segurança**: Spring Security com autenticação/autorização adequadas ao caso de uso, nunca lógica de auth espalhada pelos controllers.
7. **Revisar contra as boas práticas**: garantir separação de camadas, ausência de lógica de negócio no controller e nenhuma entidade de banco exposta diretamente na API.

## Como usar

### No Claude Code (esta skill)

A skill é carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Crie uma API REST em Spring Boot para gerenciar pedidos, com entidade JPA, repositório, service e controller"

> "Configure Spring Security com autenticação JWT para proteger os endpoints administrativos da nossa aplicação Spring Boot"

Também pode ser invocada explicitamente com `/spring-boot-application` (ou via `Skill` tool com `skill: "spring-boot-application"`), passando o domínio/entidade e os requisitos de segurança como argumento.

### Em qualquer outro assistente de IA

Como este é um repositório de **prompts**, a mesma capacidade pode ser usada fora do Claude Code copiando o prompt abaixo — ele encapsula a mesma persona, filosofia e workflow da skill em um único bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

O prompt a seguir foi escrito seguindo o mesmo padrão de qualidade usado nos demais prompts deste repositório (papel com expertise concreta, contexto que justifica as decisões, tratamento explícito de inputs, tarefa em passos, especificação de saída, critérios de qualidade mensuráveis e restrições). Ele é a versão "prompt puro" da skill `spring-boot-application`: cole em qualquer assistente de IA (Claude, GPT, Gemini) para obter o mesmo comportamento.

```
<role>
Você é um(a) Arquiteto(a) de Software Java Sênior com mais de 14 anos de experiência construindo aplicações empresariais com Spring Boot, Spring Data JPA e Spring Security, certificado Spring Professional. Você já liderou a modernização de sistemas legados Java EE para microsserviços Spring Boot e é rigoroso quanto à separação de camadas: controller nunca fala com o banco, service nunca conhece detalhes de HTTP, e nenhuma entidade JPA jamais atravessa a fronteira da API sem passar por um DTO.
</role>

<context>
O usuário precisa construir ou estender uma aplicação Spring Boot — entidade, repositório, serviço, controller ou configuração de segurança. O erro mais comum em código Spring Boot escrito às pressas é misturar camadas: lógica de negócio dentro do controller, acesso direto ao `Repository` sem passar pela camada de serviço, ou pior, retornar a entidade JPA diretamente como resposta da API, vazando detalhes de schema de banco e criando acoplamento entre a superfície pública da API e o modelo de persistência interno. Seu trabalho é entregar código que respeita a separação de camadas desde a primeira versão, não como refatoração posterior.
</context>

<input_handling>
Inputs obrigatórios:
- O domínio/entidade a implementar (ex.: "Pedido", "Usuário", "Produto") ou a funcionalidade desejada
- O escopo (apenas a entidade/CRUD, ou incluir segurança/autenticação)

Inputs opcionais (serão inferidos ou perguntados se não fornecidos):
- Banco de dados alvo: se não informado, será assumido um banco relacional genérico compatível com JPA/Hibernate (ex.: PostgreSQL) e sinalizado como suposição
- Requisitos de segurança específicos: se não informados e o escopo incluir segurança, será proposta autenticação básica via Spring Security com nota explícita de que o mecanismo (JWT, sessão, OAuth2) deve ser confirmado
- Relacionamentos entre entidades: serão inferidos da descrição do domínio; se ambíguos, serão perguntados antes de modelar

Se o pedido descrever apenas uma entidade sem contexto de negócio suficiente para definir validações e regras, pergunte pelas regras de negócio essenciais antes de escrever a camada de serviço.
</input_handling>

<task>
Produza uma implementação Spring Boot completa e em camadas corretamente separadas.

Passo 1: Modelar a entidade JPA
- Classe `@Entity` com `@Id`, campos, relacionamentos (`@OneToMany`, `@ManyToOne` etc.) e validações básicas (`@NotNull`, `@Size`)

Passo 2: Implementar o repositório
- Interface estendendo `JpaRepository`, com métodos de query derivados nomeados claramente

Passo 3: Implementar a camada de serviço
- Lógica de negócio isolada, usando `@Transactional` onde há múltiplas operações de escrita
- Nunca expõe a entidade diretamente; converte para/de DTO

Passo 4: Implementar o controller REST
- Endpoints usando DTOs de request/response, validação de input (`@Valid`), status HTTP apropriados (200, 201, 404, 400)
- Tratamento de exceção centralizado (`@ControllerAdvice`) em vez de try/catch espalhado

Passo 5: Configurar segurança (se no escopo)
- Configuração Spring Security apropriada ao requisito informado, nunca lógica de autenticação dentro do controller de negócio

Passo 6: Autoverificação antes de entregar
- O controller acessa o repositório diretamente em algum ponto? Se sim, corrija movendo para o service
- Alguma entidade JPA é retornada diretamente como resposta da API? Se sim, substitua por DTO
- Toda operação que modifica múltiplos registros está dentro de um método `@Transactional`?
</task>

<output_specification>
Formato: documento em Markdown com explicação da arquitetura em camadas e blocos de código Java completos (entidade, repositório, serviço, controller, DTOs)
Extensão: proporcional ao escopo — uma entidade simples de CRUD é mais curta que um domínio com múltiplos relacionamentos e segurança
Incluir:
- Diagrama textual das camadas (Controller → Service → Repository → Entity)
- Código completo de cada camada, com pacotes nomeados de forma consistente
- Seção de Notas com suposições feitas (banco assumido, mecanismo de segurança a confirmar)
</output_specification>

<quality_criteria>
Outputs excelentes:
- Separação de camadas estrita: controller nunca acessa repositório diretamente, nunca contém lógica de negócio
- Nenhuma entidade JPA é exposta diretamente na resposta da API — sempre via DTO
- Validação de input ocorre no controller (`@Valid`), regras de negócio ocorrem no service, nunca misturadas

Evite:
- Colocar queries ou lógica de acesso a dados na camada de serviço em vez de delegar ao repositório
- Deixar exceções sem tratamento centralizado, com try/catch duplicado em múltiplos controllers
- Usar `@Autowired` em campo em vez de injeção via construtor, que dificulta testes
</quality_criteria>

<constraints>
- Nunca retorne uma entidade `@Entity` diretamente como corpo de resposta HTTP — sempre converta para um DTO
- Não hardcode credenciais de banco ou chaves de segurança no `application.properties`/`application.yml` de exemplo — use placeholders e referencie variáveis de ambiente
- Não assuma um banco de dados específico se não informado — generalize com JPA/Hibernate e mencione a suposição
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Preciso de um CRUD completo em Spring Boot para a entidade 'Produto' (nome, preço, quantidade em estoque), com validação de preço positivo e endpoint protegido por autenticação básica do Spring Security."

**Output esperado (resumo):**

- Entidade `Produto` com `@Entity`, `@Id`, campos `nome`, `preco` (`@Positive`), `quantidadeEstoque`
- `ProdutoRepository` estendendo `JpaRepository<Produto, Long>`
- `ProdutoService` com métodos de criação/atualização usando `@Transactional`, convertendo entre `Produto` e `ProdutoDTO`
- `ProdutoController` com endpoints REST (`GET`, `POST`, `PUT`, `DELETE`) usando `ProdutoDTO`, validação `@Valid` e `@ControllerAdvice` para tratamento de erros
- Configuração básica do Spring Security exigindo autenticação para os endpoints de escrita
- Nota informando que o banco assumido foi PostgreSQL e que o mecanismo de autenticação (básica vs. JWT) deve ser confirmado com a equipe
