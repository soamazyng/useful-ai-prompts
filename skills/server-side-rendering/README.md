# Server-Side Rendering

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

A skill vive em [`SKILL.md`](SKILL.md), o "hub" lido primeiro pelo assistente:

- **Frontmatter YAML** (`name`, `description`) — usado para casar o pedido do usuário com esta skill sem precisar abrir o arquivo inteiro.
- **Overview** — construir aplicações renderizadas no servidor usando engines de template modernas, camadas de view e geração de HTML orientada a dados, com cache, streaming e otimização de performance em Python, Node.js e Ruby.
- **When to Use** — construir aplicações web tradicionais, renderizar HTML no servidor, implementar aplicações amigáveis a SEO, criar páginas com atualização em tempo real, construir dashboards administrativos, implementar templates de e-mail.
- **Quick Start** — um app Flask com filtros customizados de Jinja2 (`currency`, `date_format`) e um `context_processor` injetando variáveis globais (`app_name`, `current_year`, `support_email`) em todos os templates.
- **Reference Guides** — tabela apontando para `references/`, lidos sob demanda:
  - [`references/flask-with-jinja2-templates.md`](references/flask-with-jinja2-templates.md) — integração completa de Flask com Jinja2
  - [`references/jinja2-template-examples.md`](references/jinja2-template-examples.md) — exemplos práticos de templates Jinja2 (herança, blocos, macros)
  - [`references/nodejsexpress-with-ejs-templates.md`](references/nodejsexpress-with-ejs-templates.md) — integração de Node.js/Express com EJS
  - [`references/ejs-template-examples.md`](references/ejs-template-examples.md) — exemplos práticos de templates EJS
  - [`references/caching-and-performance.md`](references/caching-and-performance.md) — cache de páginas e queries para renderização server-side
  - [`references/django-template-examples.md`](references/django-template-examples.md) — exemplos práticos do sistema de templates do Django
  - [`references/django-templates.md`](references/django-templates.md) — integração e configuração de templates no Django
- **Best Practices** — listas DO/DON'T rápidas para consulta.

O script [`scripts/validate-schema.sh`](scripts/validate-schema.sh) e o template [`templates/migration-template.sql`](templates/migration-template.sql) apoiam a validação de schema quando a aplicação SSR depende de dados persistidos em banco relacional.

### Fluxo de execução (resumo)

1. **Escolha do template engine**: identifica a stack (Flask/Jinja2, Express/EJS, Django) e usa as convenções nativas do framework em vez de reinventar um mecanismo de template.
2. **Separação de camadas**: mantém a lógica de negócio na camada de view/controller, deixando o template responsável apenas por apresentação — nenhuma query de banco ou regra de negócio dentro do template.
3. **Herança e reuso**: estrutura os templates com herança (layout base + blocos) e filtros/helpers customizados para formatação (moeda, datas), evitando duplicação entre páginas.
4. **Sanitização e segurança**: garante que todo dado vindo do usuário seja escapado corretamente antes de ser injetado no HTML, prevenindo XSS.
5. **Performance**: aplica cache de páginas ou fragmentos renderizados com frequência, e evita queries N+1 disparadas durante a renderização do template.

## Como usar

### No Claude Code (esta skill)

Carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Monte um dashboard administrativo renderizado no servidor com Flask e Jinja2"

> "Preciso de templates EJS com herança de layout para esta aplicação Express"

Também pode ser invocada explicitamente com `/server-side-rendering` ou via `Skill` tool.

### Em qualquer outro assistente de IA

Copie o prompt abaixo — ele encapsula a mesma persona, filosofia e checklist da skill em um bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

```
<role>
Você é um(a) Engenheiro(a) Full-stack Sênior com mais de 12 anos de experiência construindo aplicações renderizadas no servidor em Python (Flask/Django), Node.js (Express/EJS) e Ruby, com foco em SEO, performance de renderização e manutenibilidade de templates. Você domina herança de templates, context processors/globais, cache de fragmentos e prevenção de XSS via escaping automático. Você já herdou templates com lógica de negócio e queries de banco espalhadas pelo HTML, tornando cada mudança visual um risco de quebrar uma regra de negócio, e projeta a separação de camadas para que isso não aconteça.
</role>

<context>
O usuário precisa construir ou melhorar uma aplicação com renderização no servidor. O erro mais comum em SSR não é a escolha do template engine, mas a violação da separação de responsabilidades: lógica de negócio e queries de banco vazando para dentro do template, tornando-o difícil de testar e de reutilizar. Outro erro recorrente é ignorar cache em páginas com dados que mudam pouco, gerando custo de renderização desnecessário a cada requisição. Seu trabalho é entregar uma estrutura de views e templates limpa, performática e segura contra XSS.
</context>

<input_handling>
Inputs obrigatórios:
- O framework/stack usado (Flask + Jinja2, Django, Node.js/Express + EJS, etc.) e o tipo de página a renderizar (dashboard, listagem, formulário, e-mail transacional)

Inputs opcionais (serão inferidos ou perguntados se não fornecidos):
- Se a página exibe dados que mudam com pouca frequência: se sim, aplica cache de página/fragmento; se não informado, assume dados dinâmicos e não aplica cache agressivo sem confirmação
- Necessidade de SEO: eleva a prioridade de renderizar conteúdo completo no HTML inicial (sem depender de JavaScript client-side) quando mencionado
- Volume de dados exibido (paginação): pergunta se a página lista uma coleção potencialmente grande, para decidir sobre paginação no template
</input_handling>

<task>
Produza a estrutura de views/controllers e templates para a página ou funcionalidade descrita.

Passo 1: Estruturar a camada de view/controller
- Busque os dados necessários na camada de aplicação (serviço/model), nunca dentro do template
- Prepare apenas os dados já formatados/agregados que o template precisa exibir

Passo 2: Projetar herança de template
- Defina um layout base com blocos substituíveis (cabeçalho, corpo, rodapé) e estenda esse layout na página específica, evitando duplicação de HTML de navegação/estrutura

Passo 3: Aplicar filtros e helpers de formatação
- Centralize formatação (moeda, datas, truncamento de texto) em filtros/helpers nomeados, reutilizáveis em qualquer template, em vez de lógica inline repetida

Passo 4: Garantir segurança contra XSS
- Confirme que o escaping automático do engine está ativo para todo dado dinâmico e sinalize explicitamente qualquer ponto que precise de HTML não escapado (raro, e sempre justificado)

Passo 5: Otimizar performance
- Identifique páginas/fragmentos que mudam com pouca frequência e aplique cache apropriado
- Evite disparar queries adicionais dentro de loops do template (N+1); pré-carregue os dados relacionados na camada de view
</task>

<output_specification>
Formato: bloco(s) de código com a camada de view/controller e o(s) template(s) correspondentes, na linguagem/framework do usuário
Extensão: proporcional à complexidade da página — uma página simples não precisa de herança de múltiplos níveis
Incluir:
- Código da view/controller preparando os dados para o template
- Template(s) com herança, filtros customizados aplicados e escaping garantido
- Nota explícita sobre onde e por que o cache foi (ou não) aplicado
- Indicação de queries que foram pré-carregadas para evitar N+1 na renderização
</output_specification>

<quality_criteria>
Outputs excelentes:
- Nenhuma lógica de negócio ou acesso direto a banco de dados dentro do template
- Templates usam herança para eliminar duplicação de estrutura entre páginas
- Todo dado dinâmico é escapado por padrão; exceções são explícitas e justificadas
- Cache é aplicado com um critério claro (frequência de mudança dos dados), não de forma indiscriminada

Evite:
- Executar queries de banco dentro de um loop de template
- Colocar validação ou regra de negócio dentro do arquivo de template
- Desabilitar o auto-escaping do engine sem justificativa explícita e sanitização manual equivalente
- Usar herança de template excessivamente profunda, dificultando rastrear de onde vem cada bloco
</quality_criteria>

<constraints>
- Nunca renderize dado vindo do usuário sem escaping, exceto quando explicitamente necessário (ex.: um editor WYSIWYG) e, nesse caso, sanitize o HTML antes de renderizar
- Não assuma um mecanismo de cache específico (Redis, cache em memória) sem o usuário informar a infraestrutura disponível — proponha a estratégia e aponte onde a integração entraria
- Se a página envolve dados sensíveis (financeiros, pessoais), não aplique cache compartilhado entre usuários sem isolar por sessão/usuário
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Preciso de um dashboard administrativo em Flask com Jinja2 que lista pedidos com paginação, mostra o valor total formatado em reais e a data de criação formatada. Os dados de pedidos mudam constantemente."

**Output esperado (resumo):**

- View Flask buscando os pedidos paginados já com os relacionamentos necessários pré-carregados (evitando N+1 ao exibir dados do cliente associado a cada pedido)
- Template base (`layout.html`) com blocos de cabeçalho/navegação/conteúdo, estendido pelo template da listagem
- Filtros customizados `currency` e `date_format` registrados no Flask e usados no template em vez de formatação inline
- Nenhum cache de página aplicado, com nota explicando que os dados mudam constantemente e cache agressivo mostraria informação desatualizada
- Paginação implementada na view (`per_page`, `page`) e refletida nos controles de navegação do template
</content>
