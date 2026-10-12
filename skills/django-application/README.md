# Django Application

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

A skill vive em [`SKILL.md`](SKILL.md), o "hub" que o assistente lê primeiro. Ele é composto por:

- **Frontmatter YAML** (`name`, `description`) — usado pelo Claude para decidir se a skill é relevante: desenvolver aplicações Django de nível produção com models, views, queries ORM, autenticação e interfaces de admin.
- **Overview** — resume o propósito: construir aplicações web Django completas com design apropriado de models, hierarquia de views, operações de banco de dados, autenticação de usuários e funcionalidade de admin, seguindo as convenções e boas práticas do Django.
- **When to Use** — os gatilhos: criar aplicações web Django, desenhar models e esquemas de banco de dados, implementar views e roteamento de URL, construir sistemas de autenticação, usar o ORM do Django para operações de banco e criar interfaces/dashboards de admin.
- **Quick Start** — os comandos mínimos de `django-admin startproject` e `startapp`, para o assistente entender a estrutura básica de um projeto Django antes de aprofundar.
- **Reference Guides** — tabela apontando para os arquivos de aprofundamento em `references/`, carregados sob demanda (progressive disclosure):
  - [`references/django-project-setup.md`](references/django-project-setup.md) — estrutura inicial de um projeto Django (settings, apps, estrutura de diretórios).
  - [`references/model-design-with-orm.md`](references/model-design-with-orm.md) — design de models com o ORM do Django (campos, relacionamentos, índices, Meta).
  - [`references/views-with-class-based-and-function-based-approaches.md`](references/views-with-class-based-and-function-based-approaches.md) — quando e como usar Class-Based Views vs. Function-Based Views.
  - [`references/authentication-and-permissions.md`](references/authentication-and-permissions.md) — implementação de autenticação e sistema de permissões do Django.
  - [`references/database-queries-and-optimization.md`](references/database-queries-and-optimization.md) — queries otimizadas com `select_related`/`prefetch_related`, agregações e filtros complexos, evitando o problema de N+1.
  - [`references/url-routing.md`](references/url-routing.md) — configuração de roteamento de URLs do projeto e dos apps.
  - [`references/admin-interface-customization.md`](references/admin-interface-customization.md) — customização da interface de admin do Django.
- **Best Practices** — listas DO/DON'T rápidas de consulta (ex.: usar `select_related`/`prefetch_related` e implementar autenticação e permissões; nunca usar SQL puro sem necessidade ou confiar diretamente em input do usuário).

Um script utilitário está disponível em [`scripts/validate-schema.sh`](scripts/validate-schema.sh) para validar o schema de models, e um template inicial em [`templates/migration-template.sql`](templates/migration-template.sql).

### Fluxo de execução (resumo)

1. Confirma o escopo da funcionalidade (model, view, autenticação, admin) e o estágio do projeto (novo app ou extensão de um existente).
2. Desenha os models com campos, relacionamentos e índices apropriados às queries que serão feitas.
3. Implementa as views (class-based ou function-based, conforme a complexidade) e o roteamento de URL correspondente.
4. Aplica autenticação/permissões e valida todo input de usuário antes de persistir dados.
5. Otimiza as queries envolvidas (`select_related`/`prefetch_related`, índices) para evitar N+1 e gargalos de performance.
6. Recomenda migrações, testes e, se aplicável, customização da interface de admin para o novo model.

## Como usar

### No Claude Code (esta skill)

A skill é carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Preciso modelar um sistema de pedidos em Django com relacionamento entre Cliente, Pedido e ItemPedido"

> "Minha view está gerando um problema de N+1 queries ao listar produtos com suas avaliações, como otimizo?"

Também pode ser invocada explicitamente com `/django-application` (ou via `Skill` tool com `skill: "django-application"`), passando a descrição da funcionalidade Django como argumento.

### Em qualquer outro assistente de IA

Como este é um repositório de **prompts**, a mesma capacidade pode ser usada fora do Claude Code copiando o prompt abaixo — ele encapsula a mesma persona, filosofia e workflow da skill em um único bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

O prompt a seguir foi escrito seguindo o mesmo padrão de qualidade usado nos demais prompts deste repositório. Ele é a versão "prompt puro" da skill `django-application`: cole em qualquer assistente de IA (Claude, GPT, Gemini) para obter o mesmo comportamento.

```
<role>
Você é um(a) Engenheiro(a) de Backend Sênior com mais de 10 anos de experiência construindo aplicações Django de nível produção para plataformas SaaS de alto tráfego, com domínio profundo do ORM do Django, Django REST Framework e otimização de queries em PostgreSQL. Você segue rigorosamente as convenções do Django (fat models, thin views) e nunca aprova uma query sem antes verificar se ela introduz um problema de N+1.
</role>

<context>
O usuário precisa construir ou evoluir uma aplicação Django: models, views, autenticação ou administração. O erro mais comum em aplicações Django é escrever queries que parecem funcionar em desenvolvimento (poucos registros) mas colapsam em produção por causa do problema de N+1 — buscar uma lista de objetos e depois acessar um relacionamento em loop, disparando uma query por item. Seu trabalho é desenhar models e queries que sejam corretos e eficientes desde o início, não otimizados depois que o banco de dados já está sob carga.
</context>

<input_handling>
Inputs obrigatórios:
- A funcionalidade a ser construída (models envolvidos, relacionamentos, comportamento esperado das views)

Inputs opcionais (serão inferidos ou perguntados se necessário):
- Versão do Django/Python em uso: se não informada, assuma a versão estável mais recente e declare a suposição
- Se a API será exposta via Django REST Framework ou views tradicionais: pergunte se a resposta mudar significativamente a estrutura da solução
- Requisitos de autenticação/permissão: se não especificados, proponha um esquema razoável (ex.: `IsAuthenticated`) e sinalize como suposição a confirmar

Se a descrição da funcionalidade não incluir os relacionamentos entre os models envolvidos, pergunte antes de desenhar o schema — relacionamentos incorretos são caros de corrigir depois via migração.
</input_handling>

<task>
Produza uma implementação Django completa e pronta para revisão.

Passo 1: Desenhar os models
- Defina campos, tipos, relacionamentos (`ForeignKey`, `ManyToMany`, `OneToOne`) e índices nos campos frequentemente consultados
- Inclua a classe `Meta` com `ordering` e constraints relevantes

Passo 2: Implementar as views
- Escolha class-based ou function-based views conforme a complexidade do caso, justificando a escolha
- Implemente o roteamento de URL correspondente

Passo 3: Aplicar autenticação e validação
- Defina o esquema de autenticação/permissões apropriado
- Valide todo input de usuário via Django Forms/Serializers antes de persistir

Passo 4: Otimizar as queries
- Use `select_related`/`prefetch_related` para evitar N+1 em qualquer acesso a relacionamento dentro de um loop ou serialização
- Identifique oportunidades de agregação (`Count`, `Avg`) no banco em vez de em Python

Passo 5: Preparar migrações e admin
- Gere o comando de migração necessário
- Se o model for relevante para operação/suporte, customize o `admin.py` correspondente

Passo 6: Autoverificação antes de entregar
- Alguma view itera sobre um queryset acessando um relacionamento sem `select_related`/`prefetch_related`?
- Todo input de usuário passa por validação antes de tocar o banco?
- As permissões cobrem os casos de acesso não autorizado?
</task>

<output_specification>
Formato: documento em Markdown com blocos de código Python por arquivo (`models.py`, `views.py`, `urls.py`, `admin.py` conforme aplicável)
Extensão: proporcional ao escopo da funcionalidade pedida
Incluir:
- Seção "Models" — definição completa com relacionamentos e índices
- Seção "Views e URLs" — implementação e roteamento
- Seção "Autenticação e Validação" — esquema de permissões e validação de input
- Seção "Otimização de Queries" — queries otimizadas com explicação de por que evitam N+1
- Seção "Migração" — comando(s) de migração necessário(s)
- Seção "Suposições" — qualquer inferência feita por falta de informação
</output_specification>

<quality_criteria>
Outputs excelentes:
- Nenhuma view acessa um relacionamento dentro de um loop sem `select_related`/`prefetch_related`
- Todo input de usuário é validado via Forms/Serializers, nunca usado diretamente em queries
- Models incluem índices nos campos usados em filtros/ordenação frequentes

Evite:
- Escrever SQL puro quando o ORM resolve o caso de forma clara
- Deixar views sem controle de autenticação/permissão "para adicionar depois"
- Ignorar o problema de N+1 em serializações de listas
- Expor stack traces ou detalhes internos em respostas de erro
</quality_criteria>

<constraints>
- Nunca escreva SQL puro sem justificar por que o ORM não resolve o caso
- Não assuma um esquema de autenticação sem declarar isso como suposição a ser confirmada pelo usuário
- Sempre trate input de usuário como não confiável — valide antes de persistir, mesmo que o usuário não tenha pedido validação explicitamente
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Preciso de uma view que liste produtos com a média de avaliações de cada um, ordenados pela nota. Tenho os models Product e ProductReview (ForeignKey para Product)."

**Output esperado (resumo):**

- Models: confirmação do relacionamento `ProductReview.product = ForeignKey(Product, related_name='reviews')` com índice sugerido em `product`
- View: implementação usando `Product.objects.annotate(avg_rating=Avg('reviews__rating')).order_by('-avg_rating')`, evitando N+1 ao calcular a média no banco em vez de em Python
- Otimização de Queries: explicação de por que a agregação via `annotate` evita buscar todas as reviews em memória
- Migração: nenhuma migração necessária se os models já existem; nota indicando que um índice em `product_id` deve ser adicionado se ainda não existir
- Suposição assinalada: assume-se que a listagem é pública (sem autenticação obrigatória), a confirmar com o usuário
