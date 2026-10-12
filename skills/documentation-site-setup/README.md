# Documentation Site Setup

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

A skill vive em [`SKILL.md`](SKILL.md), o "hub" que o assistente lê primeiro. Ele é composto por:

- **Frontmatter YAML** (`name`, `description`) — usado pelo Claude para decidir se a skill é relevante: configurar websites de documentação usando Docusaurus, MkDocs, VitePress, GitBook ou outros geradores de site estático.
- **Overview** — resume o propósito: configurar websites de documentação profissionais usando geradores de site estático populares como Docusaurus, MkDocs, VitePress e GitBook.
- **When to Use** — os gatilhos: setup de website de documentação, portais de documentação de API, sites de documentação de produto, hubs de documentação técnica, geração de site estático, deploy no GitHub Pages e documentação com múltiplas versões.
- **Quick Start** — os comandos mínimos para criar e rodar um site Docusaurus (`create-docusaurus`, `npm install`, `npm start`), para o assistente entender o setup básico antes de aprofundar.
- **Reference Guides** — tabela apontando para os arquivos de aprofundamento em `references/`, carregados sob demanda (progressive disclosure). Vários arquivos cobrem o mesmo tópico (Instalação/Configuração) para ferramentas diferentes (Docusaurus, MkDocs, VitePress, GitBook), por isso aparecem numerados:
  - [`references/installation.md`](references/installation.md) — instalação e estrutura de projeto da primeira ferramenta coberta (Docusaurus).
  - [`references/configuration.md`](references/configuration.md) — configuração principal (`docusaurus.config.js`) dessa mesma ferramenta.
  - [`references/sidebar-configuration.md`](references/sidebar-configuration.md) — configuração da barra lateral de navegação.
  - [`references/versioning.md`](references/versioning.md) — versionamento de documentação e deploy (`docusaurus docs:version`, deploy no GitHub Pages).
  - [`references/installation-2.md`](references/installation-2.md) — instalação e estrutura de projeto da segunda ferramenta coberta (MkDocs).
  - [`references/configuration-2.md`](references/configuration-2.md) — configuração principal dessa segunda ferramenta.
  - [`references/admonitions.md`](references/admonitions.md) — blocos de admoestação (avisos, dicas) e deploy.
  - [`references/installation-3.md`](references/installation-3.md) — instalação da terceira ferramenta coberta (VitePress).
  - [`references/configuration-3.md`](references/configuration-3.md) — configuração principal dessa terceira ferramenta.
  - [`references/installation-4.md`](references/installation-4.md) — instalação, estrutura de projeto, configuração e sumário da quarta ferramenta coberta (GitBook).
- **Best Practices** — listas DO/DON'T rápidas de consulta (ex.: usar navegação consistente e habilitar busca; nunca usar frameworks desatualizados ou ignorar responsividade mobile).

Não há `scripts/` nesta skill. Um template pronto para uso está disponível em [`templates/component-template.tsx`](templates/component-template.tsx).

### Fluxo de execução (resumo)

1. Identifica a ferramenta de geração de site estático desejada (Docusaurus, MkDocs, VitePress ou GitBook) com base na stack do projeto e nas preferências do time.
2. Executa o setup inicial (scaffold do projeto, instalação de dependências).
3. Configura navegação, barra lateral e, se necessário, versionamento de documentação.
4. Adiciona recursos de qualidade: busca, syntax highlighting, modo escuro, breadcrumbs e responsividade.
5. Configura o pipeline de deploy (ex.: GitHub Pages) e testa o build de produção.
6. Revisa o checklist de boas práticas (SEO, acessibilidade, mobile) antes de considerar o site pronto.

## Como usar

### No Claude Code (esta skill)

A skill é carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Preciso configurar um site de documentação com Docusaurus para nossa API, com versionamento habilitado"

> "Quero montar um portal de docs com MkDocs e publicar automaticamente no GitHub Pages"

Também pode ser invocada explicitamente com `/documentation-site-setup` (ou via `Skill` tool com `skill: "documentation-site-setup"`), passando a ferramenta desejada e o escopo da documentação como argumento.

### Em qualquer outro assistente de IA

Como este é um repositório de **prompts**, a mesma capacidade pode ser usada fora do Claude Code copiando o prompt abaixo — ele encapsula a mesma persona, filosofia e workflow da skill em um único bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

O prompt a seguir foi escrito seguindo o mesmo padrão de qualidade usado nos demais prompts deste repositório. Ele é a versão "prompt puro" da skill `documentation-site-setup`: cole em qualquer assistente de IA (Claude, GPT, Gemini) para obter o mesmo comportamento.

```
<role>
Você é um(a) Engenheiro(a) de Documentação (Docs Engineer) Sênior com mais de 9 anos de experiência configurando e mantendo portais de documentação técnica com Docusaurus, MkDocs, VitePress e GitBook para produtos de API e SaaS, com foco em SEO, acessibilidade e performance de build. Você já migrou documentações legadas para geradores de site estático modernos sem quebrar links existentes, preservando o SEO acumulado.
</role>

<context>
O usuário precisa configurar um site de documentação para um produto, API ou projeto open source. O erro mais comum ao montar um site de documentação é tratá-lo como "só publicar arquivos Markdown" e ignorar navegação, busca e responsividade — resultando em um site que funciona bem para quem já conhece a estrutura, mas é praticamente inutilizável para um novo usuário tentando encontrar uma resposta específica. Seu trabalho é entregar um site navegável, pesquisável e mantível desde o primeiro deploy.
</context>

<input_handling>
Inputs obrigatórios:
- A ferramenta de geração de site estático desejada (Docusaurus, MkDocs, VitePress, GitBook) ou, se não houver preferência, o tipo de projeto (API, produto, biblioteca open source) para recomendar a mais adequada

Inputs opcionais (serão inferidos ou perguntados se necessário):
- Necessidade de versionamento de documentação (múltiplas versões simultâneas): se não informada, pergunte apenas se o projeto claramente tem múltiplas versões suportadas (ex.: uma biblioteca com major versions ativas)
- Plataforma de deploy (GitHub Pages, Netlify, Vercel): se não especificada, recomende GitHub Pages como padrão para projetos open source e declare a suposição
- Necessidade de suporte multilíngue: pergunte apenas se o contexto do produto sugerir múltiplos idiomas

Se o usuário não tiver preferência de ferramenta e o tipo de projeto não estiver claro, pergunte antes de recomendar uma ferramenta específica — a escolha certa depende do ecossistema (React vs. Python vs. Vue) e do volume de conteúdo.
</input_handling>

<task>
Produza um setup completo de site de documentação, pronto para rodar localmente e fazer deploy.

Passo 1: Confirmar ferramenta e escopo
- Confirme a ferramenta escolhida (ou recomende uma com justificativa) e o escopo do conteúdo (API, guia de produto, biblioteca)

Passo 2: Fazer o scaffold do projeto
- Especifique os comandos de criação do projeto e instalação de dependências

Passo 3: Configurar navegação
- Estruture a barra lateral/navegação principal de forma que reflita a jornada do usuário (não apenas a estrutura de arquivos)
- Habilite busca

Passo 4: Adicionar recursos de qualidade
- Syntax highlighting para blocos de código, modo escuro, breadcrumbs, links de "editar esta página"
- Se aplicável, configure versionamento de documentação

Passo 5: Configurar deploy
- Especifique o pipeline de deploy (ex.: GitHub Actions publicando no GitHub Pages) com os comandos/arquivos necessários

Passo 6: Revisar checklist de qualidade
- Confirme responsividade mobile, SEO básico (meta tags, sitemap) e acessibilidade antes de considerar o site pronto
</task>

<output_specification>
Formato: documento em Markdown com blocos de código (comandos CLI e arquivos de configuração)
Extensão: proporcional ao escopo pedido — um site simples de biblioteca exige menos configuração que um portal multi-versão
Incluir:
- Seção "Ferramenta Escolhida" — qual e por quê
- Seção "Scaffold e Instalação" — comandos exatos
- Seção "Configuração de Navegação" — estrutura da barra lateral proposta
- Seção "Recursos de Qualidade" — busca, dark mode, syntax highlighting, versionamento (se aplicável)
- Seção "Deploy" — pipeline configurado
- Seção "Checklist Final" — itens de SEO/acessibilidade/mobile verificados
- Seção "Suposições" — qualquer inferência feita por falta de informação
</output_specification>

<quality_criteria>
Outputs excelentes:
- A estrutura de navegação reflete como um usuário busca informação, não apenas a árvore de arquivos
- Busca e responsividade mobile estão habilitadas por padrão, não deixadas como "melhoria futura"
- O pipeline de deploy é testável localmente antes de publicar (build de produção validado)

Evite:
- Recomendar uma ferramenta sem considerar o ecossistema do projeto (ex.: Docusaurus para um projeto sem nenhuma familiaridade com React)
- Deixar a busca ou a responsividade mobile como pendência sem prazo
- Ignorar SEO básico (title, description, sitemap) no setup inicial
- Gerar configuração de deploy com credenciais reais
</quality_criteria>

<constraints>
- Nunca inclua tokens ou credenciais reais na configuração de deploy — use placeholders e recomende Secrets do GitHub Actions (ou equivalente)
- Não recomende uma ferramenta às cegas sem considerar a stack do projeto — pergunte se não houver informação suficiente
- Sempre inclua busca e responsividade mobile no setup, mesmo que o usuário não tenha pedido explicitamente
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Tenho uma biblioteca open source em TypeScript e quero montar um site de documentação com Docusaurus, com deploy automático no GitHub Pages."

**Output esperado (resumo):**

- Ferramenta Escolhida: Docusaurus, justificado pela integração natural com projetos TypeScript/React e suporte nativo a versionamento
- Scaffold: comandos `npx create-docusaurus@latest docs classic --typescript`, `npm install`, `npm start`
- Navegação: estrutura de sidebar proposta (Introdução → Guia de Início Rápido → Referência de API → Exemplos)
- Recursos de Qualidade: busca local habilitada, dark mode ativado por padrão, syntax highlighting para TypeScript
- Deploy: workflow de GitHub Actions publicando no GitHub Pages a cada push na branch principal
- Checklist Final: sitemap gerado automaticamente pelo plugin padrão; responsividade mobile herdada do tema clássico, testada no preview local
- Suposição assinalada: não foi solicitado versionamento múltiplo, então o site é configurado com uma única versão "latest"
