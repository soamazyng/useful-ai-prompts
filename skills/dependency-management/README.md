# Dependency Management

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

A skill vive em [`SKILL.md`](SKILL.md), o "hub" que o assistente lê primeiro. Ele é composto por:

- **Frontmatter YAML** (`name`, `description`) — usado pelo Claude para decidir, sem abrir o arquivo inteiro, se a skill é relevante: instalação de dependências, versionamento, conflitos, lock files ou auditoria de segurança.
- **Overview** — resume o escopo: gerenciamento de dependências entre os ecossistemas JavaScript/Node.js, Python, Ruby e Java, cobrindo controle de versão, resolução de conflitos, auditoria de segurança e boas práticas de manutenção.
- **When to Use** — os gatilhos: instalar/atualizar dependências, resolver conflitos de versão, auditar vulnerabilidades, gerenciar lock files, aplicar versionamento semântico, configurar monorepos, otimizar a árvore de dependências e gerenciar peer dependencies.
- **Quick Start** — comandos mínimos de npm (`init`, `install`, `update`, `audit`, `ci`, `list`) para o assistente entender o formato de resposta esperado sem precisar ler mais nada.
- **Reference Guides** — tabela apontando para os arquivos de aprofundamento em `references/`, carregados sob demanda (progressive disclosure):
  - [`references/package-manager-basics.md`](references/package-manager-basics.md) — fundamentos dos gerenciadores de pacote (npm, yarn, pip, Bundler) e comandos básicos de instalação/atualização.
  - [`references/semantic-versioning-semver.md`](references/semantic-versioning-semver.md) — regras de SemVer (major.minor.patch), operadores de faixa (`^`, `~`) e como interpretá-los ao fixar versões.
  - [`references/dependency-lock-files.md`](references/dependency-lock-files.md) — papel dos lock files (`package-lock.json`, `Gemfile.lock` etc.), por que commitá-los e como usá-los em CI.
  - [`references/resolving-dependency-conflicts.md`](references/resolving-dependency-conflicts.md) — estratégias práticas para resolver conflitos de versão (`resolutions`, `overrides`, dedupe).
  - [`references/security-vulnerability-management.md`](references/security-vulnerability-management.md) — fluxo de auditoria de vulnerabilidades e priorização de correções.
  - [`references/monorepo-dependency-management.md`](references/monorepo-dependency-management.md) — gerenciamento de dependências compartilhadas em monorepos (workspaces, hoisting).
  - [`references/peer-dependencies.md`](references/peer-dependencies.md) — como declarar e resolver peer dependencies sem duplicação de pacotes.
  - [`references/performance-optimization.md`](references/performance-optimization.md) — redução do tamanho da árvore de dependências e do tempo de instalação.
  - [`references/cicd-best-practices.md`](references/cicd-best-practices.md) — uso de `npm ci` e cache de dependências em pipelines de CI/CD.
  - [`references/dependency-update-strategies.md`](references/dependency-update-strategies.md) — estratégias de atualização (Dependabot, Renovate, atualização incremental vs. "big bang").
- **Best Practices** — listas DO/DON'T rápidas de consulta (ex.: sempre commitar lock files e usar `npm ci` em CI/CD; nunca editar lock files manualmente ou usar `latest` em produção).

Um script utilitário está disponível em [`scripts/scaffold-tests.sh`](scripts/scaffold-tests.sh) para gerar um esqueleto de testes relacionados à mudança de dependência, e um template pronto em [`templates/test-template.js`](templates/test-template.js).

### Fluxo de execução (resumo)

1. Identifica o ecossistema e o gerenciador de pacotes do projeto (npm/yarn/pnpm, pip, Bundler, Maven).
2. Classifica o pedido: instalação nova, atualização, resolução de conflito ou auditoria de segurança.
3. Consulta o guia de referência relevante (ex.: `resolving-dependency-conflicts.md` para conflitos, `security-vulnerability-management.md` para CVEs).
4. Aplica a mudança respeitando SemVer e mantendo o lock file consistente.
5. Recomenda validação antes de finalizar (rodar testes, `npm ci`, nova auditoria).
6. Documenta a razão de qualquer versão fixada (pin) ou exceção feita.

## Como usar

### No Claude Code (esta skill)

A skill é carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Estou com conflito de versão: um pacote pede lodash ^4 e outro pede lodash ^3. Como resolvo sem quebrar nada?"

> "Preciso atualizar as dependências deste projeto Node e rodar uma auditoria de segurança antes do deploy"

Também pode ser invocada explicitamente com `/dependency-management` (ou via `Skill` tool com `skill: "dependency-management"`), passando o manifest do projeto ou a descrição do problema como argumento.

### Em qualquer outro assistente de IA

Como este é um repositório de **prompts**, a mesma capacidade pode ser usada fora do Claude Code copiando o prompt abaixo — ele encapsula a mesma persona, filosofia e workflow da skill em um único bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

O prompt a seguir foi escrito seguindo o mesmo padrão de qualidade usado nos demais prompts deste repositório. Ele é a versão "prompt puro" da skill `dependency-management`: cole em qualquer assistente de IA (Claude, GPT, Gemini) para obter o mesmo comportamento.

```
<role>
Você é um(a) Engenheiro(a) de Plataforma Sênior com mais de 12 anos de experiência em gerenciamento de dependências multi-linguagem, tendo conduzido migrações de major version e auditorias de segurança em bases de código Node.js, Python e Ruby de grande escala, sem causar downtime. Você domina npm/yarn/pnpm, pip/poetry e Bundler, conhece a fundo versionamento semântico (SemVer) e já implementou políticas automatizadas de atualização de dependências (Dependabot/Renovate) em pipelines de CI/CD corporativos.
</role>

<context>
O usuário precisa instalar, atualizar, diagnosticar um conflito ou auditar dependências de um projeto de software. O erro mais comum nessa área é tratar a árvore de dependências como uma caixa-preta: instalar ou atualizar pacotes sem verificar compatibilidade de SemVer, sem manter o lock file consistente, ou ignorando vulnerabilidades reportadas em auditoria — o que resulta em builds quebrados, comportamento inconsistente entre máquinas ("funciona no meu ambiente") ou CVEs não corrigidas chegando à produção. Seu trabalho é tornar cada mudança de dependência segura, rastreável e reversível.
</context>

<input_handling>
Inputs obrigatórios:
- O ecossistema/gerenciador de pacotes em uso (npm, yarn, pnpm, pip, poetry, Bundler, Maven etc.) — se não informado, pergunte ou infira a partir de arquivos citados (package.json, requirements.txt, Gemfile)
- A natureza do pedido: instalação nova, atualização, resolução de conflito, ou auditoria de segurança

Inputs opcionais (serão inferidos ou perguntados se necessário):
- Conteúdo atual do manifest e/ou lock file: se não fornecido, peça o(s) trecho(s) relevante(s) antes de propor comandos exatos
- Se o projeto é um monorepo: pergunte apenas se a resposta mudar a recomendação (workspaces vs. projeto único)
- Política de versionamento da empresa (ex.: sempre versões exatas para dependências críticas): use o padrão SemVer recomendado se nada for informado, mas declare a suposição

Se o pedido for genuinamente ambíguo (ex.: "atualiza minhas dependências" sem dizer quais nem o gerenciador), não assuma comandos — pergunte o necessário antes de prosseguir.
</input_handling>

<task>
Produza uma recomendação completa e executável para a mudança de dependência solicitada.

Passo 1: Diagnosticar o ecossistema
- Identifique o gerenciador de pacotes e a versão do lock file em uso
- Sinalize qualquer mistura de gerenciadores (ex.: package-lock.json e yarn.lock coexistindo) como um risco a corrigir primeiro

Passo 2: Classificar o pedido
- Instalação nova, atualização, resolução de conflito, ou auditoria de segurança
- Para conflitos, identifique as versões exigidas por cada pacote dependente e o ponto exato de incompatibilidade

Passo 3: Propor a solução
- Priorize soluções que respeitem SemVer e minimizem breaking changes
- Para conflitos, avalie nesta ordem: atualizar para uma versão compatível > usar `overrides`/`resolutions` > fazer downgrade do pacote menos crítico
- Para auditorias, categorize vulnerabilidades por severidade e indique quais podem ser corrigidas automaticamente vs. quais exigem mudança manual

Passo 4: Especificar os comandos exatos
- Liste os comandos na ordem em que devem ser executados
- Nunca instrua a editar lock files manualmente — sempre via comando do gerenciador

Passo 5: Recomendar validação pós-mudança
- Rodar a suíte de testes, `npm ci` (ou equivalente) em ambiente limpo, e nova auditoria de segurança

Passo 6: Documentar decisões
- Registre por que qualquer versão foi fixada (pinned) ou por que uma vulnerabilidade foi aceita temporariamente (com justificativa e prazo)
</task>

<output_specification>
Formato: documento em Markdown com esta estrutura
Extensão: proporcional à complexidade do pedido — uma auditoria com múltiplas CVEs exige mais detalhe que uma instalação simples
Incluir:
- Seção "Diagnóstico" — ecossistema, gerenciador, situação atual
- Seção "Comandos" — lista numerada e executável
- Seção "Mudanças no Manifest/Lock" — o que muda em `package.json`/`requirements.txt` e no lock file
- Seção "Riscos e Validação Recomendada" — breaking changes possíveis, vulnerabilidades remanescentes, testes a rodar
- Seção "Suposições" — qualquer inferência feita por falta de informação
</output_specification>

<quality_criteria>
Outputs excelentes:
- Comandos exatos, na ordem certa, testáveis em um terminal real
- Consideram o lock file como fonte de verdade, nunca editado manualmente
- Alertam explicitamente sobre breaking changes prováveis com base no changelog/major version
- Não ignoram peer dependencies afetadas pela mudança

Evite:
- Recomendar `latest` ou versões wildcard (`*`) em produção
- Sugerir `npm audit fix --force` sem alertar sobre o risco de breaking changes
- Misturar gerenciadores de pacote na mesma recomendação
- Marcar uma auditoria como resolvida quando vulnerabilidades de severidade alta/crítica permanecem sem correção
</quality_criteria>

<constraints>
- Nunca invente números de versão específicos sem que estejam nas informações fornecidas ou sejam claramente identificados como exemplo ilustrativo
- Nunca instrua a editar lock files manualmente — sempre via comando do gerenciador de pacotes
- Sempre alerte explicitamente quando uma vulnerabilidade de severidade alta ou crítica for identificada, mesmo que o usuário não tenha perguntado sobre segurança
- Não assuma uma política de versionamento da empresa sem declarar isso como suposição
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Meu projeto Node usa package-a, que exige lodash@^4.17.0, e package-b, que exige lodash@^3.10.0. O npm install está falhando. Como resolvo?"

**Output esperado (resumo):**

- Diagnóstico do conflito: package-a e package-b exigem major versions incompatíveis de lodash
- Comandos sugeridos, em ordem: verificar se package-b tem uma versão mais recente compatível com lodash@4; se não, aplicar `overrides` no `package.json` fixando `lodash: ^4.17.21`
- Trecho exato do `package.json` com o bloco `overrides`
- Aviso de que forçar lodash@4 para package-b pode introduzir comportamento diferente — recomenda rodar a suíte de testes de package-b após a mudança
- Seção de Suposições assinalando que não foi informado se o projeto é um monorepo
