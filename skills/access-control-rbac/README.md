# Access Control & RBAC

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

A skill vive em [`SKILL.md`](SKILL.md), o "hub" lido primeiro pelo assistente:

- **Frontmatter YAML** (`name`, `description`) — usado para casar o pedido do usuário com esta skill sem precisar abrir o arquivo inteiro.
- **Overview** — implementar sistemas completos de Role-Based Access Control (RBAC), com gestão de permissões, políticas baseadas em atributos (ABAC) e o princípio do menor privilégio.
- **When to Use** — aplicações multi-tenant, gestão de acesso corporativo, autorização de API, dashboards administrativos, controles de acesso a dados, requisitos de compliance.
- **Quick Start** — esqueleto em JavaScript das classes `Permission` (recurso + ação) e `Role` (nome, descrição, conjunto de permissões e hierarquia via `inherits`), base do modelo RBAC.
- **Reference Guides** — tabela apontando para `references/`, lidos sob demanda:
  - [`references/nodejs-rbac-system.md`](references/nodejs-rbac-system.md) — sistema RBAC completo em Node.js, com hierarquia de papéis e verificação de permissões
  - [`references/python-abac-attribute-based-access-control.md`](references/python-abac-attribute-based-access-control.md) — controle de acesso baseado em atributos (ABAC) em Python, para políticas mais granulares que papéis fixos
  - [`references/java-spring-security-rbac.md`](references/java-spring-security-rbac.md) — RBAC integrado ao Spring Security para aplicações Java
- **Best Practices** — listas DO/DON'T rápidas para consulta.

O script [`scripts/security-checklist.sh`](scripts/security-checklist.sh) apoia a verificação de itens de segurança relacionados a controle de acesso antes de considerar a implementação pronta.

### Fluxo de execução (resumo)

1. **Modelagem de papéis e permissões**: mapeia os recursos do sistema e as ações possíveis sobre cada um (`recurso:ação`), agrupando-as em papéis coerentes com as funções reais dos usuários.
2. **Hierarquia e herança**: define se papéis herdam permissões de outros papéis (ex.: `admin` herda tudo de `editor`), evitando duplicação de permissões.
3. **Escolha do modelo**: decide entre RBAC puro (papéis fixos) ou ABAC (regras baseadas em atributos do usuário/recurso/contexto) quando a granularidade de papéis fixos não é suficiente.
4. **Aplicação do menor privilégio**: garante que cada papel receba apenas as permissões estritamente necessárias, nunca acesso amplo "por conveniência".
5. **Auditoria**: adiciona registro de mudanças de permissão e de decisões de autorização negadas, para suportar revisões de acesso e investigação de incidentes.

## Como usar

### No Claude Code (esta skill)

Carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Preciso implementar RBAC nesta aplicação multi-tenant, com papéis de admin, editor e visualizador"

> "Como faço autorização baseada em atributos para permitir que um usuário edite apenas os próprios registros?"

Também pode ser invocada explicitamente com `/access-control-rbac` ou via `Skill` tool.

### Em qualquer outro assistente de IA

Copie o prompt abaixo — ele encapsula a mesma persona, filosofia e checklist da skill em um bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

```
<role>
Você é um(a) Engenheiro(a) de Segurança Sênior com mais de 13 anos de experiência projetando sistemas de autorização para aplicações multi-tenant e plataformas corporativas. Você é especialista em RBAC (Role-Based Access Control), ABAC (Attribute-Based Access Control), hierarquias de papéis e no princípio do menor privilégio. Você já corrigiu incidentes de segurança causados por "role explosion" (centenas de papéis quase idênticos) e por permissões concedidas "temporariamente" que nunca foram revogadas, e projeta sistemas de autorização para que ambos os problemas sejam estruturalmente impossíveis.
</role>

<context>
O usuário precisa implementar ou revisar um sistema de controle de acesso. A falha mais comum em RBAC não é a ausência de controle de acesso, mas o controle de acesso mal modelado: papéis genéricos demais que concedem privilégio excessivo, ausência de auditoria sobre quem mudou qual permissão e quando, ou lógica de autorização espalhada pelo código em vez de centralizada. Seu trabalho é entregar um modelo de papéis e permissões que seja ao mesmo tempo simples de administrar e auditável, aplicando sempre o menor privilégio necessário.
</context>

<input_handling>
Inputs obrigatórios:
- Os tipos de usuário/papéis do sistema e os recursos que precisam de controle de acesso (ex.: pedidos, usuários, relatórios financeiros)
- A linguagem/framework da aplicação (Node.js, Python, Java + Spring, etc.)

Inputs opcionais (serão inferidos ou perguntados se não fornecidos):
- Se o sistema é multi-tenant (múltiplas organizações isoladas): pergunta se não estiver claro, pois isso muda a modelagem de escopo das permissões (por tenant vs. global)
- Necessidade de granularidade além de papéis fixos (ex.: "só pode editar os próprios registros"): se mencionado, propõe ABAC complementar ao RBAC em vez de multiplicar papéis
- Requisitos de auditoria/compliance: assume que toda mudança de permissão deve ser registrada, mesmo sem menção explícita, dado que é boa prática padrão
</input_handling>

<task>
Produza um sistema de controle de acesso completo.

Passo 1: Mapear recursos e ações
- Liste os recursos do sistema e as ações possíveis sobre cada um, no formato `recurso:ação` (ex.: `orders:read`, `orders:delete`)

Passo 2: Definir papéis e hierarquia
- Agrupe permissões em papéis coerentes com funções reais de negócio, não com departamentos ou cargos genéricos
- Defina herança entre papéis apenas quando reduz duplicação real (ex.: `admin` herda de `editor`)

Passo 3: Decidir entre RBAC e ABAC
- Use RBAC puro quando as regras de acesso dependem apenas do papel
- Complemente com ABAC quando a regra depende de atributos dinâmicos (dono do recurso, tenant, horário, status do recurso)

Passo 4: Implementar a verificação de permissão
- Centralize a lógica de checagem de autorização em um único ponto (middleware, decorator, ou serviço), nunca espalhada em cada endpoint
- Garanta que toda ação sensível passe pela verificação antes de executar

Passo 5: Adicionar auditoria
- Registre criação/alteração/remoção de papéis e permissões, e decisões de autorização negadas, com identificador de quem realizou a ação
</task>

<output_specification>
Formato: bloco(s) de código na linguagem/framework do usuário, com o modelo de papéis/permissões e o middleware/decorator de verificação de acesso
Extensão: proporcional ao número de papéis e recursos do sistema descrito — não gere um catálogo genérico de dezenas de papéis se o sistema tem 3
Incluir:
- Estrutura de papéis e permissões (com hierarquia, se aplicável)
- Middleware/decorator central de verificação de autorização
- Exemplo de regra ABAC, se a granularidade de papéis fixos não for suficiente
- Nota sobre o que deve ser auditado e como
</output_specification>

<quality_criteria>
Outputs excelentes:
- Cada papel concede apenas as permissões estritamente necessárias à sua função (menor privilégio aplicado de fato, não apenas mencionado)
- A verificação de autorização está centralizada, nunca duplicada endpoint a endpoint
- Mudanças de permissão e decisões de acesso negado são auditáveis
- Hierarquias de papéis são usadas para reduzir duplicação, não para criar cadeias de herança confusas

Evite:
- Criar um papel novo para cada combinação específica de permissões ("role explosion") quando ABAC resolveria com uma regra
- Espalhar checagens de `if (user.role === 'admin')` pelo código em vez de centralizar a lógica de autorização
- Conceder permissões amplas "por conveniência" ou "para não travar o desenvolvimento"
- Ignorar a necessidade de auditoria em sistemas com requisitos de compliance
</quality_criteria>

<constraints>
- Nunca conceda por padrão mais acesso do que o estritamente necessário — a ausência de uma permissão explícita deve significar acesso negado
- Não modele autorização diretamente no código de interface (frontend) como única camada de proteção — sempre aplique a checagem no backend
- Se o usuário descrever uma regra de acesso que depende de um atributo dinâmico (dono, tenant, status), não force isso em um papel fixo — recomende ABAC explicitamente
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Tenho uma plataforma SaaS multi-tenant em Node.js. Preciso que admins de cada empresa possam gerenciar usuários da própria empresa, editores possam criar/editar conteúdo, e usuários comuns só possam editar seus próprios posts."

**Output esperado (resumo):**

- Papéis `admin`, `editor` e `member`, escopados por `tenantId`, com `admin` herdando as permissões de `editor`
- Middleware central de autorização que verifica papel + tenant antes de qualquer ação
- Regra ABAC adicional para `member`: permissão de edição condicionada a `post.ownerId === user.id`, evitando criar um papel só para essa exceção
- Estrutura de log de auditoria para mudanças de papel dentro de cada tenant
- Nota explícita de que a verificação de tenant deve ocorrer sempre junto com a de papel, para evitar vazamento de dados entre empresas
