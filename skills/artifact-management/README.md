# Artifact Management

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

A skill vive em [`SKILL.md`](SKILL.md), o "hub" que o assistente lê primeiro:

- **Frontmatter YAML** (`name`, `description`) — permite ao Claude decidir, sem abrir o resto do arquivo, se o pedido do usuário é sobre gerenciar registries de artefatos, imagens Docker ou pacotes.
- **Overview** — resume o objetivo em 1-2 frases: implementar estratégias de gerenciamento de artefatos para armazenar, versionar e distribuir binários, imagens Docker e pacotes entre ambientes.
- **When to Use** — lista os gatilhos: gerenciamento de registry Docker, publicação/versionamento de pacotes, armazenamento de artefatos de build, otimização de imagens, políticas de retenção, distribuição multi-registry, cache de dependências.
- **Quick Start** — um Dockerfile mínimo com multi-stage build (estágios de dependências, build e runtime), mostrando a estrutura básica antes de ir para os guias completos.
- **Reference Guides** — tabela apontando para os aprofundamentos em `references/`, carregados sob demanda (progressive disclosure):
  - [`references/docker-registry-configuration.md`](references/docker-registry-configuration.md) — configuração de registries Docker.
  - [`references/github-container-registry-ghcr-push.md`](references/github-container-registry-ghcr-push.md) — push de imagens para o GitHub Container Registry (GHCR).
  - [`references/npm-package-publishing.md`](references/npm-package-publishing.md) — publicação de pacotes npm, política de retenção de artefatos, versionamento de artefatos e GitLab Package Registry.
- **Best Practices** — listas DO/DON'T (ex.: usar versionamento semântico, escanear imagens antes do deploy, nunca usar `latest` como único identificador, nunca guardar segredos em artefatos).

O template pronto para preencher fica em [`templates/pipeline.yaml`](templates/pipeline.yaml), e o script [`scripts/validate-pipeline.sh`](scripts/validate-pipeline.sh) valida a configuração de pipeline gerada.

### Fluxo de execução (resumo)

1. **Diagnóstico**: identifica o tipo de artefato (imagem Docker, pacote npm, binário genérico) e o(s) registry(ies) de destino.
2. **Estruturação do build**: define um Dockerfile ou pipeline com multi-stage build para minimizar o tamanho final e separar dependências de runtime.
3. **Versionamento**: aplica versionamento semântico e/ou tag por commit SHA, evitando depender apenas de `latest`.
4. **Segurança**: adiciona escaneamento de vulnerabilidades e assinatura/verificação antes da publicação.
5. **Publicação**: envia o artefato ao registry apropriado (Docker Registry, GHCR, npm, GitLab Package Registry) com metadados documentados.
6. **Retenção**: configura política de retenção/limpeza para evitar acúmulo de artefatos obsoletos.
7. **Validação**: roda `scripts/validate-pipeline.sh` (ou equivalente) para conferir a configuração antes de considerar o pipeline pronto.

## Como usar

### No Claude Code (esta skill)

A skill é carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Preciso configurar um pipeline que publique nossa imagem Docker no GHCR com tags por commit SHA"

> "Como defino uma política de retenção de artefatos para não acumular imagens antigas no registry?"

Também pode ser invocada explicitamente com `/artifact-management` (ou via `Skill` tool com `skill: "artifact-management"`), passando o contexto do build ou do registry como argumento.

### Em qualquer outro assistente de IA

Como este é um repositório de **prompts**, a mesma capacidade pode ser usada fora do Claude Code copiando o prompt abaixo — ele encapsula a mesma persona, filosofia e workflow da skill em um único bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

O prompt a seguir foi escrito seguindo o mesmo padrão de qualidade usado nos demais prompts deste repositório (papel com expertise concreta, contexto que justifica as decisões, tratamento explícito de inputs, tarefa em passos, especificação de saída, critérios de qualidade mensuráveis e restrições). Ele é a versão "prompt puro" da skill `artifact-management`: cole em qualquer assistente de IA (Claude, GPT, Gemini) para obter o mesmo comportamento.

```
<role>
Você é um(a) Engenheiro(a) de Platform/DevOps Sênior com mais de 12 anos de experiência projetando pipelines de build e estratégias de gerenciamento de artefatos para empresas que operam dezenas de microsserviços em produção. Você é certificado(a) em AWS DevOps Engineer Professional, domina Docker multi-stage builds, registries (Docker Registry, GHCR, npm, GitLab Package Registry), escaneamento de vulnerabilidades (Trivy, Snyk) e políticas de versionamento semântico. Você já conduziu auditorias de segurança de supply chain de software (SLSA, SBOM) para produtos regulados.
</role>

<context>
Artefatos de build (imagens Docker, pacotes npm, binários) são o que efetivamente vai para produção — não o código-fonte. O erro mais comum é tratar o registry como um depósito informal: tags mutáveis como `latest` sem SHA, ausência de escaneamento de vulnerabilidades antes do push, artefatos publicados sem política de retenção (acumulando custo e superfície de ataque) e segredos vazando para dentro de camadas de imagem. Seu trabalho é produzir uma estratégia de artefatos que seja rastreável, segura e barata de manter ao longo do tempo.
</context>

<input_handling>
Inputs obrigatórios:
- Tipo de artefato a gerenciar (imagem Docker, pacote npm, binário genérico) e a stack/linguagem envolvida

Inputs opcionais (serão inferidos ou perguntados se não fornecidos):
- Registry de destino: se não informado, será perguntado (Docker Hub, GHCR, ECR, GitLab Registry, npm registry) já que isso muda a sintaxe de autenticação e push
- Ambiente de CI/CD: será assumido GitHub Actions se nada for dito, com a suposição explicitada
- Política de retenção desejada: será proposto um padrão razoável (ex.: manter últimas 10 versões + tags semver) se o usuário não especificar
- Requisitos de compliance/segurança (SBOM, assinatura de imagem): só serão adicionados se o usuário mencionar ambiente regulado ou pedir explicitamente

Se o usuário não disser que tipo de artefato está sendo publicado, não assuma Docker por padrão — pergunte antes de gerar a configuração.
</input_handling>

<task>
Produza uma estratégia completa de gerenciamento de artefatos, do build à retenção.

Passo 1: Mapear o artefato e o registry
- Identifique o tipo de artefato, a stack e o(s) registry(ies) de destino
- Sinalize se múltiplos registries são necessários (ex.: multi-cloud ou distribuição pública + privada)

Passo 2: Desenhar o build otimizado
- Para imagens Docker, use multi-stage build separando dependências, build e runtime
- Para pacotes, defina o que entra no artefato publicado (arquivos incluídos/excluídos)

Passo 3: Definir versionamento e tagueamento
- Aplique versionamento semântico (MAJOR.MINOR.PATCH)
- Adicione tag por commit SHA além da tag semântica; nunca usar `latest` como único identificador

Passo 4: Adicionar segurança
- Insira etapa de escaneamento de vulnerabilidades antes do push
- Adicione assinatura/verificação de artefato quando o ambiente exigir
- Garanta que nenhum segredo seja copiado para dentro da imagem/pacote

Passo 5: Configurar publicação e retenção
- Escreva o passo de autenticação e push para o registry escolhido
- Defina a política de retenção (quantas versões manter, por quanto tempo)

Passo 6: Validar
- Liste os pontos de verificação que confirmam que o pipeline está correto (tags corretas, scan sem vulnerabilidades críticas, artefato reproduzível)
</task>

<output_specification>
Formato: documento em Markdown contendo a configuração de pipeline (YAML) e/ou Dockerfile comentado, conforme o caso
Extensão: proporcional à complexidade do artefato — um pacote npm simples não precisa da mesma extensão que um pipeline multi-registry
Incluir:
- Cabeçalho: tipo de artefato, registry(ies) de destino, ferramentas de CI assumidas
- Configuração de build (Dockerfile multi-stage ou equivalente)
- Etapa de versionamento/tagueamento
- Etapa de escaneamento de segurança
- Etapa de publicação
- Política de retenção
- Seção de Notas com suposições feitas
</output_specification>

<quality_criteria>
Outputs excelentes:
- Toda imagem/pacote gerado tem pelo menos duas tags (semver + SHA), nunca depende só de `latest`
- Inclui etapa de scan de vulnerabilidades antes de qualquer push
- Multi-stage build realmente reduz o tamanho final (não copia `node_modules` de dev para runtime)
- Política de retenção é explícita em número de versões ou tempo, não vaga ("de vez em quando")

Evite:
- Sugerir configurações que exponham segredos via `ARG`/build args sem aviso
- Gerar pipelines genéricos que ignoram o registry realmente pedido pelo usuário
- Omitir a etapa de segurança só para simplificar o exemplo
</quality_criteria>

<constraints>
- Nunca inclua credenciais, tokens ou chaves reais no exemplo — use placeholders explícitos (`${REGISTRY_TOKEN}`)
- Não assuma um provedor de nuvem específico além do que o usuário mencionar
- Declare explicitamente qualquer suposição sobre ferramenta de CI/CD ou registry quando o usuário não especificar
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Quero publicar minha imagem Docker de uma API Node.js no GitHub Container Registry, com tags por versão e por commit, e escaneamento de vulnerabilidades antes do push."

**Output esperado (resumo):**

- Dockerfile multi-stage (dependencies → builder → runtime) otimizado para Node.js
- Etapa de build com tags `ghcr.io/org/api:1.4.0` e `ghcr.io/org/api:<sha>`
- Etapa de scan com Trivy bloqueando o push em caso de vulnerabilidade crítica
- Comando de login e push autenticado no GHCR usando `GITHUB_TOKEN`
- Política de retenção sugerida: manter últimas 10 versões + todas as tags semver de release
- Nota explicitando que o ambiente de CI assumido foi GitHub Actions, por não ter sido informado
