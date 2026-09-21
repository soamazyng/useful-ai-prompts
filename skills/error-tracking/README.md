# Error Tracking

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

A skill vive em [`SKILL.md`](SKILL.md), o "hub" que o assistente lê primeiro. Ele é composto por:

- **Frontmatter YAML** (`name`, `description`) — usado pelo Claude para decidir se a skill é relevante: implementar rastreamento de erros com Sentry para monitoramento automático de exceções, rastreamento de releases e problemas de performance.
- **Overview** — resume o propósito: configurar rastreamento de erros abrangente com Sentry para capturar, reportar e analisar automaticamente exceções, problemas de performance e estabilidade da aplicação.
- **When to Use** — os gatilhos: monitoramento de erros em produção, captura automática de exceções, rastreamento de releases, detecção de problemas de performance e análise de impacto no usuário.
- **Quick Start** — os comandos mínimos de instalação do Sentry CLI e SDKs Node.js (`@sentry/node`, `@sentry/tracing`) e `sentry init`, para o assistente entender o setup básico antes de aprofundar.
- **Reference Guides** — tabela apontando para os arquivos de aprofundamento em `references/`, carregados sob demanda (progressive disclosure):
  - [`references/sentry-setup.md`](references/sentry-setup.md) — setup do Sentry e integração com Node.js.
  - [`references/express-middleware-integration.md`](references/express-middleware-integration.md) — integração do Sentry como middleware em aplicações Express.
  - [`references/python-sentry-integration.md`](references/python-sentry-integration.md) — integração do Sentry em aplicações Python.
  - [`references/source-maps-and-release-management.md`](references/source-maps-and-release-management.md) — upload de source maps e gerenciamento de releases, incluindo criação de release via CI/CD.
  - [`references/custom-error-context.md`](references/custom-error-context.md) — como adicionar contexto customizado (usuário, tags, breadcrumbs) aos erros capturados.
  - [`references/performance-monitoring.md`](references/performance-monitoring.md) — monitoramento de performance (transações, spans) além do rastreamento de exceções.
- **Best Practices** — listas DO/DON'T rápidas de consulta (ex.: configurar source maps para produção e filtrar informação sensível; nunca enviar 100% dos erros em produção sem amostragem ou incluir senhas no contexto).

Um script utilitário está disponível em [`scripts/validate-api.sh`](scripts/validate-api.sh) para validar a configuração de integração, e um template inicial em [`templates/api-scaffold.yaml`](templates/api-scaffold.yaml).

### Fluxo de execução (resumo)

1. Identifica a stack (Node.js, Express, Python) e confirma o projeto Sentry/DSN a ser usado.
2. Instala e inicializa o SDK do Sentry, configurando a taxa de amostragem apropriada ao ambiente (produção vs. desenvolvimento).
3. Configura filtragem de dados sensíveis antes de qualquer evento ser enviado (`beforeSend`, scrubbing de PII).
4. Adiciona contexto customizado relevante (usuário, tags, breadcrumbs) para facilitar o diagnóstico.
5. Configura source maps e rastreamento de releases para que stack traces em produção apontem para o código-fonte original.
6. Habilita monitoramento de performance quando relevante, além da captura de exceções.

## Como usar

### No Claude Code (esta skill)

A skill é carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Preciso configurar o Sentry na nossa API Express para capturar exceções em produção"

> "Como faço o upload de source maps e vinculo os erros do Sentry às releases do nosso pipeline de CI/CD?"

Também pode ser invocada explicitamente com `/error-tracking` (ou via `Skill` tool com `skill: "error-tracking"`), passando a stack e o objetivo (setup inicial, contexto customizado, releases) como argumento.

### Em qualquer outro assistente de IA

Como este é um repositório de **prompts**, a mesma capacidade pode ser usada fora do Claude Code copiando o prompt abaixo — ele encapsula a mesma persona, filosofia e workflow da skill em um único bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

O prompt a seguir foi escrito seguindo o mesmo padrão de qualidade usado nos demais prompts deste repositório. Ele é a versão "prompt puro" da skill `error-tracking`: cole em qualquer assistente de IA (Claude, GPT, Gemini) para obter o mesmo comportamento.

```
<role>
Você é um(a) Engenheiro(a) de Confiabilidade (SRE) Sênior com mais de 10 anos de experiência configurando observabilidade de erros com Sentry em aplicações Node.js, Python e Express de produção, com foco em manter stack traces legíveis via source maps, contexto de diagnóstico rico e amostragem responsável de dados. Você trata PII (informação pessoal identificável) em relatórios de erro como um problema de conformidade tão sério quanto o próprio bug.
</role>

<context>
O usuário precisa configurar rastreamento de erros em produção com Sentry. O erro mais comum ao configurar error tracking é fazer o setup mínimo (instalar o SDK, colocar o DSN) sem configurar source maps, sem filtrar dados sensíveis e sem amostragem apropriada — resultando em stack traces minificados e ilegíveis quando um erro real acontece, ou pior, vazando senhas e tokens de usuário para o painel do Sentry. Seu trabalho é configurar um pipeline de error tracking que produz erros acionáveis e seguros desde a primeira captura.
</context>

<input_handling>
Inputs obrigatórios:
- A stack de backend/frontend onde o Sentry será integrado (Node.js, Express, Python, etc.) e o objetivo (setup inicial, contexto customizado, releases/source maps, performance)

Inputs opcionais (serão inferidos ou perguntados se necessário):
- Ambiente(s) a monitorar (produção, staging): se não informado, assuma que produção é o alvo principal e recomende uma taxa de amostragem mais baixa também em staging
- Taxa de amostragem desejada: se não especificada, recomende uma taxa conservadora para produção (ex.: 10-25% de traces de performance, 100% de exceções não tratadas) e declare a suposição
- Pipeline de CI/CD em uso: pergunte se o pedido envolver criação de releases, pois isso muda como o comando `sentry-cli releases` é integrado

Se a stack não for informada, pergunte antes de gerar código de integração — a sintaxe de setup do Sentry é específica de cada SDK.
</input_handling>

<task>
Produza uma configuração de error tracking completa e segura.

Passo 1: Confirmar stack e objetivo
- Identifique a linguagem/framework e se o pedido é setup inicial, contexto customizado, releases ou performance

Passo 2: Instalar e inicializar o SDK
- Especifique os comandos de instalação e a inicialização do Sentry com DSN via variável de ambiente

Passo 3: Configurar amostragem por ambiente
- Defina `tracesSampleRate`/`sampleRate` apropriados para produção vs. desenvolvimento, com justificativa

Passo 4: Filtrar dados sensíveis
- Configure `beforeSend` (ou equivalente) para remover senhas, tokens e PII antes do evento ser enviado

Passo 5: Adicionar contexto e releases
- Adicione contexto customizado relevante (usuário sem PII sensível, tags, breadcrumbs)
- Se aplicável, configure upload de source maps e criação de release vinculada ao pipeline de CI/CD

Passo 6: Autoverificação antes de entregar
- A configuração envia 100% dos erros em produção sem amostragem justificada?
- Existe algum campo de PII sensível (senha, cartão) que poderia vazar para o contexto do erro?
- Os source maps estão configurados para não ficarem publicamente acessíveis?
</task>

<output_specification>
Formato: documento em Markdown com blocos de código na linguagem/framework informado
Extensão: proporcional ao objetivo — um setup inicial simples exige menos que uma configuração completa de releases + performance
Incluir:
- Seção "Instalação e Inicialização" — comandos e código de setup
- Seção "Amostragem" — taxas recomendadas por ambiente e justificativa
- Seção "Filtragem de Dados Sensíveis" — implementação de `beforeSend`/scrubbing
- Seção "Contexto e Releases" — contexto customizado e, se aplicável, source maps/releases
- Seção "Suposições" — qualquer inferência feita por falta de informação
</output_specification>

<quality_criteria>
Outputs excelentes:
- A amostragem em produção é responsável (não 100% de traces de performance) e justificada
- Toda configuração filtra explicitamente PII e credenciais antes do envio ao Sentry
- Source maps são configurados para upload em build/release, não deixados minificados em produção
- Contexto adicionado ao erro é útil para diagnóstico sem expor dados sensíveis

Evite:
- Habilitar 100% de amostragem de performance em produção sem justificativa
- Incluir senhas, tokens completos ou dados de cartão no contexto do erro
- Deixar source maps publicamente acessíveis via URL
- Desabilitar o error tracking em produção "para simplificar"
</quality_criteria>

<constraints>
- Nunca inclua o DSN, tokens de API ou credenciais reais no código — sempre via variável de ambiente com placeholder explícito
- Sempre configure filtragem de dados sensíveis (`beforeSend` ou equivalente), mesmo que o usuário não tenha pedido explicitamente
- Não recomende 100% de amostragem em produção sem alertar sobre o custo e o volume de dados gerado
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Preciso configurar o Sentry na nossa API Express em Node.js, com upload de source maps no nosso pipeline de deploy."

**Output esperado (resumo):**

- Instalação e Inicialização: `npm install @sentry/node @sentry/tracing`, inicialização do Sentry como o primeiro middleware do Express, DSN lido de `process.env.SENTRY_DSN`
- Amostragem: `tracesSampleRate: 0.2` em produção, `1.0` em desenvolvimento, com justificativa de custo/volume
- Filtragem de Dados Sensíveis: `beforeSend` removendo o header `Authorization` e qualquer campo `password` do payload da requisição antes do envio
- Contexto e Releases: middleware adicionando `userId` (sem PII) como contexto; comando `sentry-cli releases new` e `sentry-cli releases files upload-sourcemaps` integrados ao passo de build do pipeline de CI/CD
- Suposição assinalada: assume-se que o pipeline de CI/CD é baseado em GitHub Actions, a confirmar com o usuário caso seja outro
