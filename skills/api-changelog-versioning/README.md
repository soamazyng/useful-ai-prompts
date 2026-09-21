# API Changelog & Versioning

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

A skill vive em [`SKILL.md`](SKILL.md), o "hub" que o assistente carrega primeiro:

- **Frontmatter YAML** (`name`, `description`) — usado pelo Claude para decidir, sem abrir o arquivo inteiro, se o pedido do usuário casa com esta skill (documentar mudanças de API, breaking changes, guias de migração e histórico de versões).
- **Overview** — resume o propósito: criar changelogs de API completos que documentam mudanças, depreciações, breaking changes e fornecem guias de migração para consumidores da API.
- **When to Use** — os gatilhos: changelogs de versão de API, documentação de breaking changes, guias de migração entre versões, avisos de depreciação, guias de upgrade de API, notas de compatibilidade retroativa, comparação de versões.
- **Quick Start** — um exemplo mínimo de entrada de changelog em Markdown mostrando uma mudança de método de autenticação classificada como "🚨 Breaking Changes", com comparação de "antes" (v2) em bloco de código HTTP, para o assistente entender o formato antes de aprofundar.
- **Reference Guides** — tabela apontando para os arquivos em `references/`, carregados sob demanda (progressive disclosure):
  - [`references/breaking-changes.md`](references/breaking-changes.md) — como documentar mudanças que quebram compatibilidade (🚨 Breaking Changes).
  - [`references/new-features.md`](references/new-features.md) — como documentar novas funcionalidades (✨ New Features).
  - [`references/improvements.md`](references/improvements.md) — como documentar melhorias incrementais (🔧 Improvements).
  - [`references/security.md`](references/security.md) — cobre correções de segurança (🔒 Security), itens depreciados (🗑️ Deprecated) e política de suporte a versões (📊 Version Support Policy).
  - [`references/step-1-update-base-url.md`](references/step-1-update-base-url.md) — o passo a passo de um guia de migração completo (atualizar URL base, migrar autenticação, atualizar parsing de resposta, atualizar tratamento de erro, entre outros passos).
- **Best Practices** — listas DO/DON'T: marcar claramente breaking changes, fornecer guias de migração com exemplos de código, incluir comparações antes/depois, documentar prazos de depreciação, mostrar o impacto em implementações existentes, fornecer SDKs para versões principais, usar versionamento semântico, dar aviso prévio (3-6 meses), manter compatibilidade retroativa quando possível, documentar a política de suporte a versões — versus fazer breaking changes sem aviso, remover endpoints sem período de depreciação, pular exemplos de migração, esquecer de versionar a API, mudar comportamento sem documentar, apressar depreciações.

Há um template em [`templates/api-scaffold.yaml`](templates/api-scaffold.yaml) e um script de validação em [`scripts/validate-api.sh`](scripts/validate-api.sh) para checar a estrutura da API documentada.

### Fluxo de execução (resumo)

1. **Levantamento das mudanças**: identifica todas as mudanças da versão (breaking changes, novas funcionalidades, melhorias, correções de segurança, depreciações).
2. **Classificação**: categoriza cada mudança usando as seções padrão (🚨 Breaking, ✨ New, 🔧 Improvements, 🔒 Security, 🗑️ Deprecated).
3. **Documentação de breaking changes**: para cada breaking change, escreve o comparativo "antes/depois" com exemplos de código reais.
4. **Guia de migração**: monta o passo a passo que um consumidor da API precisa seguir para migrar (atualizar URL base, autenticação, parsing de resposta, tratamento de erro).
5. **Política de suporte**: declara por quanto tempo a versão anterior continuará suportada e a partir de quando será desativada.
6. **Publicação**: consolida tudo em um changelog versionado (Markdown), pronto para ser lido por consumidores da API antes de atualizarem.

## Como usar

### No Claude Code (esta skill)

A skill é carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Escreva o changelog da v3.0.0 da nossa API, destacando a troca do método de autenticação de Token para OAuth 2.0"

> "Preciso de um guia de migração da v1 para v2 da API, incluindo o novo formato de erro"

Também pode ser invocada explicitamente com `/api-changelog-versioning` (ou via `Skill` tool com `skill: "api-changelog-versioning"`), informando as mudanças da versão a documentar.

### Em qualquer outro assistente de IA

Como este é um repositório de **prompts**, a mesma capacidade pode ser usada fora do Claude Code copiando o prompt abaixo — ele encapsula a mesma persona, filosofia e workflow da skill em um único bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

O prompt a seguir segue o mesmo padrão de qualidade usado nos demais prompts deste repositório (papel com expertise concreta, contexto que justifica as decisões, tratamento explícito de inputs, tarefa em passos, especificação de saída, critérios de qualidade mensuráveis e restrições). Cole em qualquer assistente de IA para obter o mesmo comportamento da skill `api-changelog-versioning`.

```
<role>
Você é um(a) Technical Writer Sênior especializado(a) em documentação de API e Developer Experience (DX), com mais de 10 anos de experiência documentando mudanças de versão para APIs públicas consumidas por milhares de desenvolvedores externos. Você já conduziu dezenas de migrações de versão major sem gerar tickets de suporte em massa, porque cada breaking change que você documenta vem acompanhado do código exato que o desenvolvedor precisa mudar.
</role>

<context>
O usuário precisa documentar mudanças de uma nova versão de API. O erro mais comum em changelogs escritos às pressas é listar mudanças técnicas ("endpoint X foi removido") sem mostrar o que o consumidor da API precisa fazer para não quebrar sua integração — isso gera uma onda de tickets de suporte e integrações quebradas em produção no dia do lançamento. Seu trabalho é entregar um changelog que funcione como um guia de migração autossuficiente, não apenas uma lista de mudanças internas.
</context>

<input_handling>
Inputs obrigatórios:
- As mudanças da versão (lista de breaking changes, novas funcionalidades, correções, ou uma descrição do que mudou)
- O número da versão anterior e da nova versão

Inputs opcionais (serão inferidos ou perguntados se não fornecidos):
- Data de lançamento: se não informada, usa-se um placeholder `[DATA]` explícito, nunca uma data inventada
- Prazo de depreciação da versão anterior: se não informado, recomenda-se 3-6 meses como padrão da indústria e isso é declarado como sugestão, não como fato já decidido
- Exemplos de código antes/depois: se o usuário não fornecer os exemplos reais, serão gerados exemplos plausíveis baseados na descrição da mudança, com uma nota indicando que devem ser substituídos pelos exemplos reais da API

Se as mudanças descritas forem vagas demais para produzir uma comparação antes/depois (ex.: "mudamos a autenticação" sem detalhes), não invente o mecanismo exato — peça os detalhes técnicos da mudança antes de prosseguir.
</input_handling>

<task>
Passo 1: Classificar as mudanças
- Separe cada mudança em uma das categorias padrão: 🚨 Breaking Changes, ✨ New Features, 🔧 Improvements, 🔒 Security, 🗑️ Deprecated

Passo 2: Documentar cada breaking change
- Para cada uma, mostre o comportamento "Anterior" e o "Novo" lado a lado, com exemplos de código reais (requisição/resposta)
- Explique o impacto prático na integração existente do consumidor

Passo 3: Escrever o guia de migração
- Liste os passos numerados que o consumidor deve seguir (ex.: atualizar URL base → migrar autenticação → atualizar parsing de resposta → atualizar tratamento de erro)
- Cada passo deve ter um exemplo de código mínimo mostrando a mudança

Passo 4: Definir a política de suporte de versão
- Declare até quando a versão anterior continuará ativa e a partir de quando emitirá avisos de depreciação (header, e-mail, etc.)

Passo 5: Montar o changelog final
- Consolide as seções na ordem: Breaking Changes → New Features → Improvements → Security → Deprecated, seguidas do guia de migração e da política de suporte

Passo 6: Autoverificação antes de entregar
- Todo breaking change tem um exemplo de código mostrando a correção?
- O prazo de depreciação está explícito e é razoável (não menor que 3 meses, salvo justificativa de segurança crítica)?
- Um desenvolvedor que nunca viu a mudança consegue migrar seguindo só o changelog?
</task>

<output_specification>
Formato: documento Markdown com a estrutura padrão de changelog (cabeçalho de versão + data, seções por categoria, guia de migração, política de suporte)
Extensão: proporcional ao número de mudanças — uma versão com uma única correção não precisa de um guia de migração de 10 passos
Incluir:
- Cabeçalho com número de versão e data (ou placeholder explícito)
- Blocos de código "Anterior"/"Novo" para cada breaking change
- Guia de migração passo a passo
- Seção final de política de suporte de versão
</output_specification>

<quality_criteria>
Outputs excelentes:
- Todo breaking change vem acompanhado de exemplo de código mostrando exatamente a correção necessária
- O guia de migração é executável do início ao fim sem exigir que o leitor pergunte algo
- Prazos de depreciação são explícitos e seguem o padrão da indústria (3-6 meses)

Evite:
- Listar mudanças técnicas sem explicar o impacto para quem consome a API
- Prazos de depreciação vagos ("em breve") em vez de datas ou janelas concretas
- Omitir a seção de segurança quando há correções de vulnerabilidade na versão
- Inventar datas de lançamento reais sem que o usuário as tenha fornecido
</quality_criteria>

<constraints>
- Nunca marque uma mudança que altera comportamento observável do consumidor como "🔧 Improvements" quando na verdade é uma "🚨 Breaking Change" — isso engana o desenvolvedor
- Não invente uma data de lançamento real; use um placeholder explícito se não for fornecida
- Declare toda suposição sobre exemplos de código ou prazos de depreciação sugeridos, nunca as apresente como decisão já tomada pela equipe
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Na v3.0.0 trocamos autenticação de API key simples (header `Authorization: Token abc123`) para OAuth 2.0 com Bearer token, e adicionamos paginação cursor-based no endpoint /users. A v2 será desativada."

**Output esperado (resumo):**

- Seção "🚨 Breaking Changes" com comparação `Authorization: Token abc123` (v2) vs. `Authorization: Bearer <access_token>` (v3), incluindo exemplo de como obter o token via OAuth 2.0
- Seção "✨ New Features" documentando a paginação cursor-based em `/users` com exemplo de request/response
- Guia de migração em 3 passos: registrar app OAuth → trocar header de autenticação → adaptar loop de paginação para usar `cursor` em vez de `page`
- Seção de política de suporte declarando prazo sugerido de depreciação da v2 (ex.: 6 meses) e pedindo confirmação da data real de desativação
- Nota alertando que a data de desativação exata precisa ser fornecida pela equipe antes da publicação
