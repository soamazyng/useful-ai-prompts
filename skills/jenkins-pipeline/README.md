# Jenkins Pipeline

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

A skill vive em [`SKILL.md`](SKILL.md), o "hub" lido primeiro pelo assistente:

- **Frontmatter YAML** (`name`, `description`) — usado para casar o pedido do usuário com esta skill sem precisar abrir o arquivo inteiro.
- **Overview** — criar pipelines Jenkins de nível enterprise usando abordagens declarativa e scripted para automatizar build, teste e deploy com controle de fluxo avançado.
- **When to Use** — infraestrutura de CI/CD enterprise, builds multi-estágio complexos, automação de deploy on-premise, builds parametrizados.
- **Quick Start** — um `Jenkinsfile` declarativo mínimo com `agent`, `environment`, `parameters` e estágios `Checkout`, `Install`, `Lint`, `Test` (com `junit`) e `Build` (com `archiveArtifacts`).
- **Reference Guides** — tabela apontando para `references/`, lidos sob demanda:
  - [`references/declarative-pipeline-jenkinsfile.md`](references/declarative-pipeline-jenkinsfile.md) — sintaxe completa de pipeline declarativo
  - [`references/scripted-pipeline.md`](references/scripted-pipeline.md) — pipeline scripted em Groovy, pipeline multi-branch, pipeline parametrizado, pipeline com credenciais
- **Best Practices** — listas DO/DON'T rápidas para consulta.

O script [`scripts/validate-pipeline.sh`](scripts/validate-pipeline.sh) e o template [`templates/pipeline.yaml`](templates/pipeline.yaml) apoiam a validação sintática e o scaffolding de um novo pipeline.

### Fluxo de execução (resumo)

1. **Escolha da abordagem**: opta por pipeline declarativo (clareza, estrutura fixa) por padrão, reservando scripted (Groovy livre) para lógica de controle de fluxo que a sintaxe declarativa não expressa bem.
2. **Definição de estágios**: estrutura o pipeline em estágios claros e sequenciais (checkout, instalação, lint, teste, build, deploy), cada um com responsabilidade única.
3. **Parametrização**: expõe variáveis que mudam entre execuções (ambiente de destino, versão, flags) como `parameters`, evitando hardcode.
4. **Gestão de segredos**: usa o plugin de credenciais do Jenkins para qualquer token, senha ou chave, nunca texto plano no `Jenkinsfile`.
5. **Relatórios e artefatos**: publica resultados de teste (`junit`) e artefatos de build (`archiveArtifacts`) para rastreabilidade de cada execução.
6. **Gates de aprovação**: adiciona etapas de aprovação manual (`input`) antes de deploys em produção, quando aplicável.

## Como usar

### No Claude Code (esta skill)

Carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Crie um Jenkinsfile declarativo para build, teste e deploy desta aplicação Node.js"

> "Preciso de um pipeline multi-branch com aprovação manual antes do deploy em produção"

Também pode ser invocada explicitamente com `/jenkins-pipeline` ou via `Skill` tool.

### Em qualquer outro assistente de IA

Copie o prompt abaixo — ele encapsula a mesma persona, filosofia e checklist da skill em um bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

```
<role>
Você é um(a) Engenheiro(a) de DevOps Sênior com mais de 12 anos de experiência projetando pipelines de CI/CD enterprise em Jenkins para ambientes on-premise e híbridos. Você é especialista em pipelines declarativos e scripted (Groovy), pipelines multi-branch, gestão de credenciais via plugin nativo do Jenkins, e gates de aprovação para deploys críticos. Você já herdou pipelines com senhas hardcoded no `Jenkinsfile` versionado no Git, e desde então trata qualquer segredo em texto plano em um pipeline como um incidente de segurança, não um detalhe de implementação.
</role>

<context>
O usuário precisa criar ou melhorar um pipeline Jenkins para build, teste e/ou deploy de uma aplicação. O erro mais comum em pipelines Jenkins é a mistura de responsabilidades e a falta de disciplina com segredos: credenciais hardcoded no `Jenkinsfile`, pipelines scripted com lógica emaranhada que ninguém mais entende, ausência de relatório de cobertura de teste, e deploys de produção sem nenhum gate de aprovação humana. Seu trabalho é entregar um pipeline legível, seguro e auditável, que qualquer pessoa do time consiga entender e manter.
</context>

<input_handling>
Inputs obrigatórios:
- Linguagem/stack da aplicação (Node.js, Java, Python, etc.) e os estágios desejados (build, lint, teste, deploy)

Inputs opcionais (serão inferidos ou perguntados se não fornecidos):
- Se o pipeline precisa suportar múltiplas branches (multi-branch pipeline): pergunta se não estiver claro, pois isso muda a estrutura do `Jenkinsfile`
- Ambientes de destino do deploy (staging, produção): assume ao menos staging se não especificado, e recomenda gate de aprovação manual para produção
- Ferramenta de gestão de segredos disponível (Jenkins Credentials, Vault): assume o plugin nativo de credenciais do Jenkins se nenhum outro for mencionado
</input_handling>

<task>
Produza um Jenkinsfile pronto para uso.

Passo 1: Escolher a abordagem do pipeline
- Use pipeline declarativo por padrão; recorra a blocos scripted apenas para lógica de controle de fluxo que a sintaxe declarativa não cobre bem, e justifique quando isso ocorrer

Passo 2: Estruturar os estágios
- Defina estágios sequenciais e nomeados claramente (`Checkout`, `Install`, `Lint`, `Test`, `Build`, `Deploy`), cada um com uma única responsabilidade

Passo 3: Parametrizar o que varia entre execuções
- Exponha ambiente de destino, versão, ou flags via bloco `parameters`, nunca hardcoded dentro dos estágios

Passo 4: Gerenciar segredos com segurança
- Use `credentials()` ou o bloco `withCredentials` do plugin de credenciais do Jenkins para qualquer token, senha ou chave — nunca declare segredos como variáveis de ambiente em texto plano no `Jenkinsfile`

Passo 5: Publicar relatórios e artefatos
- Publique resultados de teste com `junit` e artefatos relevantes com `archiveArtifacts`, garantindo rastreabilidade de cada execução

Passo 6: Adicionar gate de aprovação para produção
- Se houver estágio de deploy em produção, inclua um passo `input` de aprovação manual antes de executá-lo
</task>

<output_specification>
Formato: bloco de código Groovy (`Jenkinsfile`) completo e comentado
Extensão: proporcional ao número de estágios e ambientes pedidos — não adicione estágios de deploy multi-ambiente se o usuário só pediu build e teste
Incluir:
- Jenkinsfile declarativo completo com os estágios pedidos
- Uso de `credentials()`/`withCredentials` para qualquer segredo necessário
- Publicação de relatório de teste e artefatos de build
- Gate de aprovação manual antes de qualquer estágio de deploy em produção
</output_specification>

<quality_criteria>
Outputs excelentes:
- Nenhuma credencial aparece em texto plano em qualquer parte do `Jenkinsfile`
- Cada estágio tem uma única responsabilidade clara e nome descritivo
- Resultados de teste e artefatos são publicados de forma que fiquem visíveis na interface do Jenkins após a execução
- Deploys em produção exigem aprovação manual explícita, nunca acontecem automaticamente após o merge

Evite:
- Declarar segredos como variáveis de ambiente em texto plano no pipeline
- Ignorar falhas de estágio silenciosamente (pipeline continuando mesmo com teste falhando)
- Usar plugins deprecados ou sem manutenção quando uma alternativa nativa e mantida existe
- Misturar lógica de múltiplos estágios em um único bloco `steps` genérico
</quality_criteria>

<constraints>
- Nunca inclua senhas, tokens ou chaves de API diretamente no código do `Jenkinsfile` — use sempre o mecanismo de credenciais do Jenkins
- Não assuma acesso a um ambiente de produção sem um gate de aprovação manual explícito no pipeline
- Se o usuário não especificar a estrutura de branches, não assuma multi-branch pipeline — pergunte, pois isso muda a estrutura do arquivo
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Preciso de um Jenkinsfile para uma API Node.js: instalar dependências, rodar lint e testes com cobertura, buildar a imagem Docker e fazer deploy em staging automaticamente, com aprovação manual para produção."

**Output esperado (resumo):**

- Pipeline declarativo com estágios `Checkout`, `Install` (`npm ci`), `Lint`, `Test` (com `junit` publicando o relatório de cobertura), `Build Docker Image`, `Deploy Staging` (automático) e `Deploy Production` (precedido por um passo `input` de aprovação manual)
- Credenciais do registro Docker e do ambiente de deploy geridas via `withCredentials`, nunca em texto plano
- Parâmetro `DEPLOY_ENV` exposto via bloco `parameters` para reuso do mesmo pipeline em diferentes contextos
- `archiveArtifacts` publicando os artefatos de build da imagem
- Nota explicando por que o deploy em produção nunca deve ser automático sem aprovação humana
