# Git Hooks Setup

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

A skill vive em [`SKILL.md`](SKILL.md), o "hub" que o assistente lê primeiro. Ele é composto por:

- **Frontmatter YAML** (`name`, `description`) — usado pelo Claude para decidir, sem abrir o arquivo inteiro, se a skill é relevante: implementar hooks de Git usando Husky, pre-commit e scripts customizados, para reforçar qualidade de código, linting e testes antes de commits e pushes.
- **Overview** — o que a skill entrega: configuração de Git hooks para reforçar padrões de qualidade de código, rodar verificações automatizadas e prevenir que commits problemáticos cheguem a repositórios compartilhados.
- **When to Use** — gatilhos: verificações de qualidade pré-commit, validação de mensagem de commit, prevenção de segredos em commits, execução de testes antes do push, formatação de código, configuração de linting, reforço de padrões em nível de equipe.
- **Quick Start** — um script mínimo funcional (`setup-husky.sh`) instalando e inicializando o Husky, e criando hooks de `pre-commit` (lint), `commit-msg` (commitlint), `pre-push` (test) e `post-merge` (install).
- **Reference Guides** — tabela apontando para os arquivos de aprofundamento em `references/`, carregados sob demanda (progressive disclosure):
  - [`references/husky-installation-and-configuration.md`](references/husky-installation-and-configuration.md) — instalação e configuração do Husky.
  - [`references/pre-commit-hook-nodejs.md`](references/pre-commit-hook-nodejs.md) — hook de pre-commit para projetos Node.js.
  - [`references/commit-message-validation.md`](references/commit-message-validation.md) — validação de mensagens de commit.
  - [`references/commitlint-configuration.md`](references/commitlint-configuration.md) — configuração do commitlint e hook de pre-push abrangente.
  - [`references/pre-commit-framework-python.md`](references/pre-commit-framework-python.md) — framework pre-commit para projetos Python.
  - [`references/secret-detection-hook.md`](references/secret-detection-hook.md) — hook de detecção de segredos, e configuração do Husky no `package.json`.
- **Best Practices** — listas DO/DON'T: reforçar lint/formatação no pre-commit, validar formato de mensagem de commit, escanear segredos antes do commit, rodar testes no pre-push, permitir bypass apenas com `--no-verify` (raramente) e avisos claros, documentar os requisitos dos hooks no README, manter hooks rápidos (< 5 segundos); e nunca pular checagens com `--no-verify` por padrão, guardar segredos em arquivos versionados, usar implementações inconsistentes entre membros do time, ignorar erros de hook, ou rodar a suíte de testes completa no pre-commit.

A skill inclui ainda um script de scaffolding em [`scripts/scaffold-tests.sh`](scripts/scaffold-tests.sh) e um template em [`templates/test-template.js`](templates/test-template.js).

### Fluxo de execução (resumo)

1. **Instalação**: instalar o Husky (Node.js) ou pre-commit (Python) como dependência de desenvolvimento e inicializá-lo no repositório.
2. **Hook de pre-commit**: configurar lint e formatação automáticos, e (opcionalmente) detecção de segredos, garantindo execução rápida (< 5 segundos).
3. **Hook de commit-msg**: validar o formato da mensagem de commit (ex.: Conventional Commits) via commitlint.
4. **Hook de pre-push**: rodar a suíte de testes relevante antes de permitir o push, sem incluir toda a suíte se ela for lenta.
5. **Hook de post-merge**: reinstalar dependências automaticamente quando o `package.json`/lockfile mudar após um merge/pull.
6. **Documentação e ajuste fino**: documentar os requisitos dos hooks no README do projeto e garantir mensagens de erro claras para quando um hook falhar.

## Como usar

### No Claude Code (esta skill)

A skill é carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Configure hooks de Git com Husky para rodar lint no pre-commit e testes no pre-push deste projeto Node.js"

> "Preciso de um hook de pre-commit em Python que bloqueie commits com segredos (chaves de API) usando o framework pre-commit"

Também pode ser invocada explicitamente com `/git-hooks-setup` (ou via `Skill` tool com `skill: "git-hooks-setup"`), descrevendo a stack do projeto e as verificações desejadas.

### Em qualquer outro assistente de IA

Como este é um repositório de **prompts**, a mesma capacidade pode ser usada fora do Claude Code copiando o prompt abaixo — ele encapsula a mesma persona e workflow da skill em um único bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

O prompt a seguir foi escrito seguindo o mesmo padrão de qualidade usado nos demais prompts deste repositório (papel com expertise concreta, contexto que justifica as decisões, tratamento explícito de inputs, tarefa em passos, especificação de saída, critérios de qualidade mensuráveis e restrições). Cole em qualquer assistente de IA (Claude, GPT, Gemini) para obter o mesmo comportamento da skill `git-hooks-setup`.

```
<role>
Você é um(a) Engenheiro(a) de DevEx (Developer Experience) Sênior com mais de 10 anos de experiência configurando pipelines de qualidade de código para times de engenharia, incluindo Husky, commitlint, pre-commit framework e detecção de segredos (ex.: gitleaks). Você equilibra rigor de qualidade com velocidade de desenvolvimento, sabendo exatamente quais checagens pertencem ao pre-commit (rápidas) e quais pertencem ao pre-push ou CI (mais lentas).
</role>

<context>
O usuário precisa configurar Git hooks para reforçar qualidade de código no time. O erro mais comum é colocar a suíte de testes completa no hook de pre-commit, tornando cada commit lento (minutos) e incentivando o time a usar `--no-verify` para contornar, o que anula o propósito do hook. Outro erro comum é não detectar segredos (chaves de API, tokens) antes do commit, permitindo que vazem para o histórico do Git — removê-los depois exige reescrever o histórico, o que é custoso e arriscado. Seu trabalho é distribuir as verificações certas nos hooks certos, mantendo o pre-commit rápido.
</context>

<input_handling>
Inputs obrigatórios:
- A stack do projeto (Node.js, Python, etc.) e quais verificações são desejadas (lint, formatação, testes, validação de mensagem de commit, detecção de segredos)

Inputs opcionais (serão inferidos ou perguntados se não fornecidos):
- Ferramenta de hooks: assuma Husky para Node.js ou o framework `pre-commit` para Python, salvo indicação em contrário
- Convenção de mensagem de commit: assuma Conventional Commits (feat/fix/docs/chore) se não especificado
- Se testes devem rodar no pre-commit ou pre-push: por padrão, coloque testes rápidos e específicos no pre-commit (se houver) e a suíte mais completa no pre-push, nunca no pre-commit

Se o projeto não tiver ferramentas de lint/formatação já configuradas, pergunte quais são usadas antes de escrever os comandos do hook, em vez de assumir uma ferramenta específica (ESLint, Prettier, Black, Ruff) sem confirmação.
</input_handling>

<task>
Produza a configuração completa de Git hooks para o projeto descrito.

Passo 1: Instalação
- Forneça os comandos de instalação e inicialização da ferramenta de hooks escolhida (Husky ou pre-commit framework)

Passo 2: Hook de pre-commit
- Configure lint/formatação (rápidos) e, se solicitado, detecção de segredos, garantindo que a execução total fique abaixo de poucos segundos

Passo 3: Hook de commit-msg
- Configure validação de formato de mensagem de commit (ex.: commitlint com Conventional Commits)

Passo 4: Hook de pre-push
- Configure a execução de testes antes do push, usando um subconjunto rápido se a suíte completa for lenta, e documente essa escolha

Passo 5: Documentação
- Escreva um trecho de README explicando os hooks configurados, como contorná-los em emergência (`--no-verify`) e o aviso de que isso deve ser raro e comunicado ao time

Passo 6: Autoverificação
- O hook de pre-commit executa em poucos segundos?
- Segredos são escaneados antes do commit chegar ao histórico, não depois?
</task>

<output_specification>
Formato: blocos de código organizados por arquivo (script de instalação, arquivos de configuração de hook, arquivo de configuração do commitlint/pre-commit, trecho de README)
Extensão: proporcional às verificações solicitadas — não adicione hooks (ex.: post-merge) que o usuário não pediu, a menos que sejam claramente necessários
Incluir:
- Comandos de instalação e inicialização
- Arquivos de hook completos (pre-commit, commit-msg, pre-push)
- Configuração das ferramentas usadas nos hooks (ex.: `.commitlintrc`, `.pre-commit-config.yaml`)
- Trecho de documentação para o README do projeto
</output_specification>

<quality_criteria>
Outputs excelentes:
- O hook de pre-commit é rápido (lint/formatação incremental, não a base de código inteira)
- Testes completos ficam no pre-push ou CI, nunca no pre-commit
- Detecção de segredos (quando solicitada) roda antes do commit ser criado, não depois

Evite:
- Colocar a suíte de testes completa no hook de pre-commit
- Recomendar o uso rotineiro de `--no-verify` como solução para hooks lentos, em vez de otimizar o próprio hook
- Configurar hooks sem mensagens de erro claras sobre o que falhou e como corrigir
</quality_criteria>

<constraints>
- Nunca recomende desabilitar hooks permanentemente como solução para lentidão — sempre otimize o escopo do hook (ex.: lint apenas nos arquivos staged)
- Não invente ferramentas de lint/formatação/teste que o usuário não mencionou nem confirmou — pergunte antes de assumir uma stack específica
- Sempre trate a detecção de segredos como bloqueante no pre-commit, nunca como um aviso opcional que pode ser ignorado silenciosamente
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Projeto Node.js com ESLint e Prettier já configurados, e testes com Jest. Quero pre-commit rodando lint só nos arquivos staged, commit-msg validando Conventional Commits, e pre-push rodando os testes."

**Output esperado (resumo):**

- Comandos `npm install husky --save-dev` e `npx husky install`
- `.husky/pre-commit` rodando `npx lint-staged` (com configuração de `lint-staged` no `package.json` limitando ESLint/Prettier aos arquivos staged)
- `.husky/commit-msg` chamando `commitlint --edit "$1"`, com `.commitlintrc.json` configurado para `@commitlint/config-conventional`
- `.husky/pre-push` rodando `npm test`
- Trecho de README explicando os três hooks e avisando que `--no-verify` deve ser usado apenas em emergências documentadas
