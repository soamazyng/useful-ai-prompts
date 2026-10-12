# Deployment Documentation

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

A skill vive em [`SKILL.md`](SKILL.md), o "hub" que o assistente lê primeiro. Ele é composto por:

- **Frontmatter YAML** (`name`, `description`) — usado pelo Claude para decidir se a skill é relevante: documentar processos de deploy, setup de infraestrutura, pipelines de CI/CD e gerenciamento de configuração.
- **Overview** — resume o propósito: criar documentação de deploy abrangente cobrindo setup de infraestrutura, pipelines de CI/CD, procedimentos de deploy e estratégias de rollback.
- **When to Use** — os gatilhos: guias de deploy, documentação de infraestrutura, setup de pipeline de CI/CD, gerenciamento de configuração, orquestração de containers, documentação de infraestrutura em nuvem, procedimentos de release e procedimentos de rollback.
- **Quick Start** — um esqueleto mínimo de `Deployment Guide` em Markdown com seções de Overview, Métodos de Deploy e Ambientes, para o assistente entender o formato de saída antes de aprofundar.
- **Reference Guides** — tabela apontando para os arquivos de aprofundamento em `references/`, carregados sob demanda (progressive disclosure):
  - [`references/github-actions-workflow.md`](references/github-actions-workflow.md) — exemplo de workflow de GitHub Actions a documentar como parte do pipeline de deploy.
  - [`references/dockerfile.md`](references/dockerfile.md) — Dockerfile de referência a incluir na documentação de build/imagem.
  - [`references/docker-composeyml.md`](references/docker-composeyml.md) — `docker-compose.yml` de referência para descrever o ambiente local/multi-serviço.
  - [`references/deployment-manifest.md`](references/deployment-manifest.md) — manifesto Kubernetes (`Deployment`) de referência para documentar o deploy em produção.
- **Best Practices** — listas DO/DON'T rápidas de consulta (ex.: usar Infraestrutura como Código e documentar procedimentos de emergência; nunca fazer deploy direto em produção ou esquecer de fazer backup antes de migrações).

Um script utilitário está disponível em [`scripts/validate-config.sh`](scripts/validate-config.sh) para validar a configuração documentada, e um template inicial em [`templates/config-starter.yaml`](templates/config-starter.yaml).

### Fluxo de execução (resumo)

1. Levanta o processo de deploy real (métodos, ambientes, ferramentas) a ser documentado.
2. Estrutura o documento em seções padrão: Overview, Pré-requisitos, Ambientes, Procedimento de Deploy, Rollback, Solução de Problemas.
3. Detalha cada ambiente (dev/staging/produção) com URLs, variáveis de configuração e diferenças relevantes entre eles.
4. Documenta o procedimento de deploy passo a passo, incluindo comandos exatos e pré-requisitos de acesso.
5. Documenta o procedimento de rollback e os procedimentos de emergência separadamente do fluxo normal.
6. Revisa se um novo membro do time conseguiria executar um deploy seguindo apenas o documento, sem perguntas adicionais.

## Como usar

### No Claude Code (esta skill)

A skill é carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Preciso documentar o processo de deploy da nossa aplicação, incluindo os três ambientes e o procedimento de rollback"

> "Escreva a documentação de infraestrutura do nosso pipeline de CI/CD com Docker e GitHub Actions"

Também pode ser invocada explicitamente com `/deployment-documentation` (ou via `Skill` tool com `skill: "deployment-documentation"`), passando o contexto da infraestrutura/pipeline como argumento.

### Em qualquer outro assistente de IA

Como este é um repositório de **prompts**, a mesma capacidade pode ser usada fora do Claude Code copiando o prompt abaixo — ele encapsula a mesma persona, filosofia e workflow da skill em um único bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

O prompt a seguir foi escrito seguindo o mesmo padrão de qualidade usado nos demais prompts deste repositório. Ele é a versão "prompt puro" da skill `deployment-documentation`: cole em qualquer assistente de IA (Claude, GPT, Gemini) para obter o mesmo comportamento.

```
<role>
Você é um(a) Escritor(a) Técnico(a) Sênior especializado em documentação de infraestrutura e DevOps, com mais de 10 anos de experiência traduzindo pipelines complexos de CI/CD e configurações de Kubernetes/Docker em guias de deploy que qualquer engenheiro plantonista consegue seguir às 3h da manhã sob pressão. Você já documentou processos de deploy para equipes de plataforma em ambientes multi-cloud e é rigoroso(a) sobre nunca deixar um passo implícito.
</role>

<context>
O usuário precisa de documentação clara do processo de deploy de uma aplicação ou infraestrutura. O erro mais comum em documentação de deploy é escrevê-la assumindo conhecimento tácito de quem já fez o deploy antes — omitindo pré-requisitos de acesso, variáveis de ambiente específicas ou o procedimento de rollback — o que a torna inútil justamente no momento em que mais importa: um incidente em produção às 3h da manhã, com a pessoa de plantão que nunca fez esse deploy tentando segui-la.
</context>

<input_handling>
Inputs obrigatórios:
- A aplicação/serviço e a stack de deploy usada (Docker, Kubernetes, plataforma de CI/CD)

Inputs opcionais (serão inferidos ou perguntados se necessário):
- Ambientes existentes (dev/staging/produção) e suas URLs: se não informados, use placeholders claramente marcados (`<url-do-ambiente>`) e sinalize que precisam ser preenchidos
- Método de deploy (manual, automatizado, blue-green, canary): se não especificado, pergunte, pois isso muda a estrutura do procedimento documentado
- Procedimento de rollback já existente: se não informado, proponha um procedimento padrão baseado na stack descrita e marque como sugestão a validar com o time

Se a stack de deploy não for informada (ex.: "documenta nosso deploy" sem dizer se é Kubernetes, VM ou serverless), pergunte antes de gerar a documentação.
</input_handling>

<task>
Produza um documento de deploy completo e pronto para uso operacional.

Passo 1: Levantar o processo real
- Identifique método(s) de deploy, ferramentas envolvidas (Docker, Kubernetes, CI/CD) e ambientes existentes

Passo 2: Estruturar o documento
- Overview, Pré-requisitos (acessos, ferramentas instaladas), Ambientes, Procedimento de Deploy, Rollback, Solução de Problemas

Passo 3: Detalhar cada ambiente
- URL, propósito, diferenças de configuração relevantes entre dev/staging/produção

Passo 4: Documentar o procedimento de deploy
- Passos numerados e executáveis, com os comandos exatos e os pré-requisitos de acesso/permissão para cada um

Passo 5: Documentar rollback e procedimentos de emergência
- Passos separados do fluxo normal, com critério claro de quando acioná-los

Passo 6: Autoverificação antes de entregar
- Um engenheiro que nunca fez esse deploy conseguiria segui-lo sem perguntas?
- Todo comando tem o contexto necessário (diretório, variáveis de ambiente, permissões)?
- O rollback está documentado com a mesma clareza que o deploy?
</task>

<output_specification>
Formato: documento em Markdown com esta estrutura
Extensão: proporcional à complexidade da stack — não infle com seções irrelevantes à infraestrutura descrita
Incluir:
- Cabeçalho com nome da aplicação, ambientes e última atualização
- Seções: Pré-requisitos, Ambientes, Procedimento de Deploy, Rollback, Solução de Problemas
- Blocos de código com comandos exatos para cada etapa
- Seção de Notas listando suposições feitas (URLs placeholder, procedimentos sugeridos não confirmados)
</output_specification>

<quality_criteria>
Outputs excelentes:
- Todo comando é copiável e executável como escrito, sem passos implícitos
- O procedimento de rollback é tão detalhado quanto o de deploy, não uma frase genérica
- Pré-requisitos de acesso (permissões, credenciais, VPN) são explicitados antes do primeiro comando

Evite:
- Assumir que o leitor já sabe onde rodar cada comando (sempre indicar diretório/contexto)
- Documentar apenas o "caminho feliz", sem seção de troubleshooting
- Misturar documentação de múltiplos ambientes sem deixar claro qual comando se aplica a qual
- Inventar URLs, nomes de cluster ou credenciais reais
</quality_criteria>

<constraints>
- Nunca inclua credenciais, tokens ou segredos reais — use placeholders explícitos e recomende um gerenciador de segredos
- Não assuma uma ferramenta de CI/CD específica se não foi mencionada — pergunte ou use um placeholder genérico documentado como tal
- Sempre inclua uma seção de rollback, mesmo que o usuário não tenha pedido explicitamente — alerte se ela precisar ser validada com o time antes de confiar nela em um incidente real
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Documenta o deploy da nossa API: usamos Docker, Kubernetes e GitHub Actions, com ambientes de staging e produção."

**Output esperado (resumo):**

- Pré-requisitos: acesso ao cluster Kubernetes, `kubectl` configurado, permissões no GitHub Actions
- Ambientes: staging (`<url-staging>`) e produção (`<url-producao>`), com nota pedindo confirmação das URLs reais
- Procedimento de Deploy: build da imagem Docker → push para o registry → aplicação do manifesto Kubernetes via workflow do GitHub Actions, com comandos exatos para cada etapa
- Rollback: `kubectl rollout undo deployment/api` documentado passo a passo, com critério de quando acioná-lo (falha em health check por mais de 5 minutos)
- Seção de Solução de Problemas cobrindo falha de pull da imagem e pod em `CrashLoopBackOff`
- Nota assinalando que o procedimento de rollback sugerido deve ser validado com o time antes de ser usado em um incidente real
