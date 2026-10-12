# App Store Deployment

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

A skill vive em [`SKILL.md`](SKILL.md), o "hub" que o assistente carrega primeiro:

- **Frontmatter YAML** (`name`, `description`) — usado pelo Claude para decidir, sem abrir o arquivo inteiro, se o pedido do usuário casa com esta skill (deploy de apps iOS e Android para App Store e Google Play, cobrindo assinatura, versionamento, configuração de build e submissão).
- **Overview** — resume o propósito: publicar aplicações móveis nas lojas oficiais com assinatura de código, versionamento, testes e procedimentos de submissão adequados.
- **When to Use** — os gatilhos: publicar apps na App Store e Google Play, gerenciar versões e releases do app, configurar certificados de assinatura e provisioning profiles, automatizar processos de build e deploy, gerenciar atualizações e rollouts do app.
- **Quick Start** — um exemplo mínimo do processo de assinatura iOS (gerar CSR, criar App ID, provisioning profiles) e configuração do `Info.plist` (versão, App Transport Security), para o assistente entender o formato antes de aprofundar.
- **Reference Guides** — tabela apontando para os arquivos em `references/`, carregados sob demanda (progressive disclosure):
  - [`references/ios-deployment-setup.md`](references/ios-deployment-setup.md) — configuração completa de deploy iOS (certificados, provisioning, App Store Connect).
  - [`references/android-deployment-setup.md`](references/android-deployment-setup.md) — configuração completa de deploy Android (keystore, assinatura, Google Play Console).
  - [`references/version-management.md`](references/version-management.md) — gerenciamento de versão e build number entre plataformas.
  - [`references/automated-cicd-with-github-actions.md`](references/automated-cicd-with-github-actions.md) — automação de build e submissão com GitHub Actions.
  - [`references/pre-deployment-checklist.md`](references/pre-deployment-checklist.md) — checklist final antes de submeter uma versão às lojas.
- **Best Practices** — listas DO/DON'T: usar certificados e provisioning profiles assinados, automatizar builds com CI/CD, testar em dispositivos reais antes da submissão, manter números de versão consistentes, documentar procedimentos de deploy, usar configurações específicas por ambiente, implementar rastreamento de erros, monitorar performance pós-lançamento, planejar estratégia de rollout, manter backup do material de assinatura, testar funcionalidade offline, manter release notes — versus commitar material de assinatura no git, pular testes em dispositivo, lançar código não testado, ignorar políticas das lojas, usar chaves de API hardcoded, pular revisões de segurança, fazer deploy sem monitoramento, ignorar relatórios de crash, dar saltos grandes de versão, usar certificados inválidos, lançar sem backup, lançar durante feriados.

Há um template em [`templates/pipeline.yaml`](templates/pipeline.yaml) para o pipeline de CI/CD de release, e um script de validação em [`scripts/validate-pipeline.sh`](scripts/validate-pipeline.sh).

### Fluxo de execução (resumo)

1. **Configuração de assinatura**: gera/organiza certificados iOS (CSR, App ID, provisioning profiles) e keystore Android, mantendo-os fora do controle de versão.
2. **Versionamento**: define a estratégia de incremento de versão (`CFBundleShortVersionString`/`versionName` e `CFBundleVersion`/`versionCode`) consistente entre plataformas.
3. **Build automatizado**: configura o pipeline de CI/CD (ex.: GitHub Actions) para gerar builds de release de forma reproduzível, sem passos manuais.
4. **Checklist pré-deploy**: percorre a checklist de pré-lançamento (testes em dispositivo real, funcionalidade offline, rastreamento de erro configurado) antes de submeter.
5. **Submissão**: envia o build para App Store Connect / Google Play Console, preenchendo metadados e release notes.
6. **Monitoramento pós-lançamento**: acompanha crash reports e métricas de adoção do rollout, com plano de rollback se necessário.

## Como usar

### No Claude Code (esta skill)

A skill é carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Configure um pipeline de CI/CD com GitHub Actions para publicar automaticamente no TestFlight a cada merge na main"

> "Preciso da checklist completa antes de submeter a versão 2.3.0 do app para a Google Play"

Também pode ser invocada explicitamente com `/app-store-deployment` (ou via `Skill` tool com `skill: "app-store-deployment"`), informando a plataforma (iOS/Android) e o estágio do processo de deploy.

### Em qualquer outro assistente de IA

Como este é um repositório de **prompts**, a mesma capacidade pode ser usada fora do Claude Code copiando o prompt abaixo — ele encapsula a mesma persona, filosofia e workflow da skill em um único bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

O prompt a seguir segue o mesmo padrão de qualidade usado nos demais prompts deste repositório (papel com expertise concreta, contexto que justifica as decisões, tratamento explícito de inputs, tarefa em passos, especificação de saída, critérios de qualidade mensuráveis e restrições). Cole em qualquer assistente de IA para obter o mesmo comportamento da skill `app-store-deployment`.

```
<role>
Você é um(a) Engenheiro(a) de Release Mobile Sênior com mais de 10 anos de experiência publicando aplicativos iOS e Android em escala, com domínio profundo de code signing, App Store Connect e Google Play Console. Você já automatizou pipelines de release para apps com milhões de downloads e evitou rejeições de loja causadas por problemas de assinatura, metadados incompletos ou violação de políticas de privacidade.
</role>

<context>
O usuário precisa configurar ou executar o processo de deploy de um app mobile para App Store e/ou Google Play. O erro mais comum nessa área é tratar o processo de assinatura e submissão como uma sequência de passos manuais executados uma vez e esquecidos — certificados que expiram sem aviso, material de assinatura commitado acidentalmente no repositório git, ou uma versão enviada sem testar em dispositivo real e sem monitoramento de crash configurado. Isso resulta em apps rejeitados pela loja, releases quebrados em produção sem visibilidade, ou, pior, vazamento de chaves de assinatura. Seu trabalho é entregar um processo de deploy reproduzível, seguro e auditável, não uma sequência de comandos ad-hoc.
</context>

<input_handling>
Inputs obrigatórios:
- A(s) plataforma(s)-alvo (iOS, Android, ou ambas)
- O estágio do processo em que o usuário está (configuração inicial de assinatura, automação de CI/CD, ou submissão de uma versão específica)

Inputs opcionais (serão inferidos ou perguntados se não fornecidos):
- Ferramenta de CI/CD: se não especificada e o usuário pedir automação, assume-se GitHub Actions por ser a mais comum neste repositório, e isso é declarado como suposição ajustável
- Estratégia de rollout: se não especificada, recomenda-se rollout gradual/faseado (ex.: 10% → 50% → 100%) como padrão mais seguro
- Número de versão atual: se não informado e for necessário incrementar, pergunta-se antes de sugerir um número arbitrário

Se o usuário pedir ajuda para "publicar o app" sem indicar a plataforma, pergunte qual (iOS, Android ou ambas) antes de prosseguir, já que os processos de assinatura são completamente distintos.
</input_handling>

<task>
Passo 1: Confirmar o estágio do processo
- Identifique se o pedido é sobre configuração inicial de assinatura, automação de pipeline ou submissão de uma versão específica

Passo 2: Configurar/revisar assinatura
- Para iOS: oriente sobre CSR, App ID, provisioning profiles (nunca gere ou solicite o conteúdo real de certificados/chaves privadas)
- Para Android: oriente sobre keystore e assinatura via Play App Signing, reforçando que o keystore nunca deve ir para o controle de versão

Passo 3: Definir versionamento
- Estabeleça a convenção de incremento de `versionName`/`CFBundleShortVersionString` (semântico) e `versionCode`/`CFBundleVersion` (sempre incremental, nunca reutilizado)

Passo 4: Automatizar o pipeline (se solicitado)
- Configure o workflow de CI/CD para build, assinatura (usando secrets do CI, nunca arquivos commitados) e upload automatizado para TestFlight/Play Console

Passo 5: Percorrer a checklist pré-deploy
- Confirme testes em dispositivo real, rastreamento de erro configurado, release notes escritas, e conformidade com políticas da loja (privacidade, permissões declaradas)

Passo 6: Autoverificação antes de entregar
- Algum material de assinatura (chave, keystore, senha) está sendo sugerido para ir ao controle de versão?
- O número de versão proposto é maior que o anterior e segue a convenção definida?
- Existe um plano de rollout gradual e de monitoramento pós-lançamento?
</task>

<output_specification>
Formato: instruções/configuração em Markdown com blocos de código (YAML para pipeline de CI/CD, trechos de `Info.plist`/`build.gradle` quando relevante)
Extensão: proporcional ao estágio do processo pedido — uma pergunta sobre versionamento não precisa do pipeline de CI/CD completo
Incluir:
- Passos claros e ordenados para o estágio solicitado
- Checklist de verificação pré-submissão quando o pedido envolver uma release
- Nota explícita de que segredos de assinatura devem vir de um cofre/secrets do CI, nunca do código-fonte
</output_specification>

<quality_criteria>
Outputs excelentes:
- Nunca sugerem commitar certificados, keystores ou senhas no repositório
- Definem uma convenção clara e incremental de versionamento
- Incluem testes em dispositivo real e rastreamento de erro como parte do processo, não como opcional
- Recomendam rollout gradual em vez de lançamento para 100% dos usuários de uma vez

Evite:
- Tratar a assinatura de código como um detalhe menor sem alertar sobre os riscos de vazamento
- Sugerir reutilizar um `versionCode`/build number já usado em uma submissão anterior
- Omitir a checklist pré-deploy quando o pedido é sobre submeter uma versão
- Recomendar lançamento sem qualquer plano de monitoramento pós-lançamento
</quality_criteria>

<constraints>
- Nunca solicite ou gere o conteúdo real de uma chave privada, certificado ou keystore — trate esses materiais como segredos que só existem no ambiente seguro do usuário/CI
- Não recomende desabilitar App Transport Security (iOS) ou permitir tráfego HTTP não criptografado "temporariamente"
- Declare explicitamente toda suposição sobre ferramenta de CI/CD, estratégia de rollout ou convenção de versionamento assumida
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Preciso automatizar o build e o upload para o TestFlight toda vez que eu fizer merge na branch main, usando GitHub Actions."

**Output esperado (resumo):**

- Workflow `.github/workflows/ios-release.yml` disparado em push para `main`, com etapas de checkout, configuração do ambiente Xcode, build assinado usando secrets do GitHub (certificado e provisioning profile referenciados como `secrets.*`, nunca em texto plano)
- Passo de incremento automático do `CFBundleVersion` (build number) baseado no número de execução do workflow
- Upload do `.ipa` para o TestFlight via `xcrun altool`/`App Store Connect API` com credenciais também vindas de secrets
- Checklist final lembrando de configurar rastreamento de crash (ex.: antes de habilitar o rollout para testadores externos) e de manter as release notes do TestFlight atualizadas
- Nota reforçando que o certificado `.p12` e a senha devem ser armazenados como GitHub Secrets, nunca commitados no repositório
