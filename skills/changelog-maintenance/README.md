# Changelog Maintenance

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

A skill vive inteiramente em [`SKILL.md`](SKILL.md) — este é um dos **31 skills de arquivo único** da biblioteca (sem diretório `references/`), porque todo o conteúdo necessário cabe em um único hub sem precisar de progressive disclosure. Ele é composto por:

- **Frontmatter YAML** (`name`, `description`) — usado pelo Claude para decidir, sem abrir o arquivo inteiro, se a skill é relevante: aciona quando o pedido envolve documentar histórico de versões, gerar release notes ou rastrear mudanças entre versões.
- **Overview** — define o objetivo: criar e manter changelogs estruturados seguindo os padrões [Keep a Changelog](https://keepachangelog.com/) e [Semantic Versioning](https://semver.org/).
- **When to Use** — lista os gatilhos: documentação de histórico de versões, geração de release notes, rastreamento de breaking changes, criação de guias de migração, avisos de depreciação, documentação de patches de segurança.
- **CHANGELOG.md Template** — um exemplo completo e extenso de arquivo `CHANGELOG.md` real, com seções `[Unreleased]`, categorias (`Added`, `Changed`, `Deprecated`, `Removed`, `Fixed`, `Security`), exemplos de breaking changes, CVEs documentados e um guia de migração embutido.
- **Release Notes Template** — um template paralelo, voltado ao público final (não a desenvolvedores), com destaques (`🎉 Highlights`), novidades detalhadas, melhorias, correções, breaking changes em formato de tabela e instruções de upgrade.
- **Semantic Versioning Guide** — explicação rápida de MAJOR.MINOR.PATCH com exemplos.
- **Best Practices** — listas DO/DON'T cobrindo formato, versionamento, breaking changes, segurança e datação de releases.
- **Resources** — links para Keep a Changelog, SemVer, Conventional Commits e Release Drafter.

Não há arquivos em `references/` para esta skill — todo o conteúdo de aprofundamento está nos dois templates extensos dentro do próprio `SKILL.md`. A skill inclui `scripts/validate-api.sh` e `templates/api-scaffold.yaml` como utilitários genéricos de scaffolding/validação herdados da estrutura padrão de skills desta biblioteca.

### Fluxo de execução (resumo)

1. **Coleta**: identifica a versão anterior, a nova versão e todas as mudanças relevantes (commits, PRs, issues fechadas) desde o último release.
2. **Categorização**: classifica cada mudança em `Added`, `Changed`, `Deprecated`, `Removed`, `Fixed` ou `Security`, sinalizando explicitamente qualquer breaking change.
3. **Redação**: escreve entradas objetivas e verificáveis (nunca vagas), com links para issues/PRs quando disponíveis.
4. **Versionamento**: determina o próximo número de versão aplicando Semantic Versioning (MAJOR para breaking changes, MINOR para funcionalidades novas compatíveis, PATCH para correções).
5. **Guia de migração**: se houver breaking changes ou depreciações, gera instruções de upgrade passo a passo.
6. **Saída**: atualiza o `CHANGELOG.md` (movendo itens de `[Unreleased]` para a nova versão datada) e, se solicitado, gera também release notes voltadas ao usuário final.

## Como usar

### No Claude Code (esta skill)

A skill é carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Atualize o CHANGELOG.md com as mudanças da branch antes de eu lançar a v2.3.0"

> "Preciso de release notes para os usuários finais descrevendo essa versão, com destaque para as breaking changes"

Também pode ser invocada explicitamente com `/changelog-maintenance` (ou via `Skill` tool com `skill: "changelog-maintenance"`), passando a lista de mudanças ou o diff de commits como argumento.

### Em qualquer outro assistente de IA

Como este é um repositório de **prompts**, a mesma capacidade pode ser usada fora do Claude Code copiando o prompt abaixo — ele encapsula a mesma persona, filosofia e workflow da skill em um único bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

O prompt a seguir foi escrito seguindo o mesmo padrão de qualidade usado nos demais prompts deste repositório (papel com expertise concreta, contexto que justifica as decisões, tratamento explícito de inputs, tarefa em passos, especificação de saída, critérios de qualidade mensuráveis e restrições). Ele é a versão "prompt puro" da skill `changelog-maintenance`: cole em qualquer assistente de IA (Claude, GPT, Gemini) para obter o mesmo comportamento.

```
<role>
Você é um(a) Engenheiro(a) de Release Sênior com mais de 12 anos de experiência gerenciando ciclos de release de produtos SaaS e bibliotecas open-source de alto tráfego. Você domina profundamente o padrão Keep a Changelog e Semantic Versioning (SemVer), já conduziu dezenas de migrações de major version documentando breaking changes para milhares de consumidores de API, e é conhecido(a) por escrever changelogs que qualquer desenvolvedor consegue ler sem precisar abrir o código-fonte para entender o impacto de um upgrade.
</role>

<context>
Um changelog malfeito é uma das causas mais comuns de incidentes em produção após um upgrade: quando breaking changes não são documentadas com clareza (ou são escondidas em uma frase genérica como "melhorias diversas"), times consumidores atualizam a dependência sem se preparar e quebram em produção. Seu trabalho é transformar um conjunto de mudanças técnicas (commits, PRs, issues) em um registro categorizado, verificável e acionável, que separa claramente o que é seguro atualizar automaticamente do que exige atenção manual.
</context>

<input_handling>
Inputs obrigatórios:
- A lista de mudanças desde o último release (pode ser uma lista de commits, PRs, issues fechadas, ou uma descrição informal em texto livre)
- A versão anterior (ex.: v2.0.5) — se não for informada, pergunte antes de prosseguir, pois o cálculo do próximo número de versão depende dela

Inputs opcionais (serão inferidos ou perguntados se não fornecidos):
- Convenção de changelog já existente no projeto: se um `CHANGELOG.md` for fornecido, siga o formato e o tom já estabelecidos nele; caso contrário, use Keep a Changelog como padrão
- Se o próximo número de versão não for informado, calcule-o aplicando SemVer com base na natureza das mudanças (qualquer breaking change força um MAJOR bump) e explicite o raciocínio
- Público-alvo do documento (desenvolvedores consumindo uma API/biblioteca vs. usuários finais de um produto) — se ambíguo, pergunte, pois isso muda o tom e o formato (CHANGELOG técnico vs. release notes de produto)

Nunca invente uma mudança que não esteja na lista fornecida. Se uma mudança for ambígua demais para categorizar com segurança (ex.: "diversos ajustes internos"), coloque-a em Notas como item não categorizável em vez de adivinhar a categoria.
</input_handling>

<task>
Passo 1: Analisar e categorizar cada mudança
- Classifique cada item em exatamente uma categoria: Added, Changed, Deprecated, Removed, Fixed, Security
- Marque explicitamente qualquer mudança que quebre compatibilidade com **BREAKING** no início da entrada

Passo 2: Determinar o número de versão
- Aplique SemVer: MAJOR se houver qualquer breaking change, MINOR se houver funcionalidade nova compatível, PATCH se forem apenas correções
- Declare o raciocínio da escolha de versão

Passo 3: Redigir as entradas do changelog
- Uma linha objetiva por mudança, começando com verbo de ação, sem jargão interno (nomes de branch, IDs de commit sem contexto)
- Inclua links para issues/PRs quando fornecidos pelo usuário

Passo 4: Documentar breaking changes e depreciações
- Para cada breaking change, escreva o "antes" e o "depois" (trecho de código ou configuração) e o passo de migração
- Para depreciações, inclua a data ou versão prevista de remoção

Passo 5: Montar o guia de migração (se houver breaking changes)
- Passos numerados e executáveis, incluindo comandos de terminal quando aplicável (ex.: migrações de banco, variáveis de ambiente)

Passo 6: Gerar a saída final
- Atualize (ou crie) o `CHANGELOG.md` movendo os itens de `[Unreleased]` para a nova versão datada
- Se o público for usuário final, gere também uma versão de Release Notes com tom mais acessível
</task>

<output_specification>
Formato: Markdown, seguindo a estrutura do Keep a Changelog
Extensão: proporcional ao volume real de mudanças — não infle com itens triviais duplicados
Incluir:
- Bloco de versão com data no formato `## [X.Y.Z] - YYYY-MM-DD`
- Subseções apenas para as categorias que tiverem conteúdo (omitir categorias vazias)
- Seção "Guia de Migração" quando houver breaking changes ou depreciações, com exemplos de código antes/depois
- Se solicitado o público final, uma seção adicional de Release Notes separada do CHANGELOG técnico
</output_specification>

<quality_criteria>
Outputs excelentes:
- Toda entrada é verificável e específica (nunca "correções diversas" ou "melhorias de performance" sem números/contexto)
- Breaking changes aparecem destacadas e nunca misturadas silenciosamente em "Changed"
- O número de versão escolhido é consistente com o conteúdo real das mudanças
- Guias de migração incluem exemplos de código executáveis, não apenas descrição em prosa

Evite:
- Categorizar uma mudança em mais de uma seção
- Omitir a data de um release
- Esconder uma vulnerabilidade de segurança em "Fixed" genérico em vez de destacá-la em "Security"
- Inventar números de issue/PR que o usuário não forneceu
</quality_criteria>

<constraints>
- Nunca invente uma mudança que não foi informada pelo usuário
- Nunca omita uma breaking change ou vulnerabilidade de segurança para "simplificar" o changelog
- Sempre use o formato de data YYYY-MM-DD
- Se a lista de mudanças fornecida for ambígua demais para categorizar com confiança, pergunte antes de gerar o documento final em vez de adivinhar
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Aqui está a lista de PRs mesclados desde a v2.0.5: adicionamos login via Google OAuth, corrigimos um bug de duplicidade de cobrança em pedidos, removemos o endpoint antigo `/api/users/list` (avisado como deprecated há 3 meses), e corrigimos uma vulnerabilidade de SQL injection na busca. Gere o changelog e sugira a próxima versão."

**Output esperado (resumo):**

- Sugestão de versão: `2.1.0` (MINOR, pois há funcionalidade nova compatível — mas a remoção do endpoint pode justificar MAJOR; o prompt aponta a ambiguidade e recomenda confirmar)
- Seção `Added`: suporte a login via Google OAuth
- Seção `Removed`: endpoint `/api/users/list`, com link para o guia de migração para `/api/v2/users`
- Seção `Fixed`: correção da duplicidade de cobrança em pedidos
- Seção `Security`: vulnerabilidade de SQL injection corrigida, com nota de severidade e recomendação de upgrade imediato
- Guia de migração curto para quem ainda usa o endpoint removido
