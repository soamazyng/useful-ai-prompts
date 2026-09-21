# Configuration Management

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

A skill vive em [`SKILL.md`](SKILL.md), o "hub" lido primeiro pelo assistente:

- **Frontmatter YAML** (`name`, `description`) — usado para casar o pedido do usuário com esta skill sem precisar abrir o arquivo inteiro.
- **Overview** — gerenciar configuração de aplicação entre ambientes, incluindo variáveis de ambiente, arquivos de configuração, segredos, feature flags e a metodologia 12-factor app.
- **When to Use** — configurar diferentes ambientes, gerenciar segredos e credenciais, implementar feature flags, criar hierarquias de configuração, seguir princípios 12-factor, migrar configuração para serviços de nuvem, configuração dinâmica, configuração multi-tenant.
- **Quick Start** — três arquivos `.env` (development/production/test) mostrando a separação de configuração por ambiente, com segredos referenciados via variável em produção em vez de valor literal.
- **Reference Guides** — tabela apontando para `references/`, lidos sob demanda:
  - [`references/environment-variables.md`](references/environment-variables.md) — carregamento e tipagem de variáveis de ambiente
  - [`references/configuration-hierarchies.md`](references/configuration-hierarchies.md) — configuração base sobreposta por overrides de ambiente
  - [`references/secret-management.md`](references/secret-management.md) — integração com gerenciadores de segredo (Vault, AWS Secrets Manager, etc.)
  - [`references/feature-flags.md`](references/feature-flags.md) — rollout gradual e toggles de funcionalidade
  - [`references/12-factor-app-configuration.md`](references/12-factor-app-configuration.md) — princípios de configuração via ambiente da metodologia 12-factor
  - [`references/configuration-validation.md`](references/configuration-validation.md) — validação de configuração na inicialização da aplicação
  - [`references/dynamic-configuration-remote-config.md`](references/dynamic-configuration-remote-config.md) — configuração remota atualizável sem redeploy
- **Best Practices** — listas DO/DON'T rápidas para consulta.

### Fluxo de execução (resumo)

1. **Inventário**: identifica todos os valores que variam entre ambientes (URLs de banco, chaves de API, flags de feature, níveis de log) e separa do código-fonte.
2. **Hierarquia**: define uma configuração base com valores padrão, sobreposta por arquivos/variáveis específicos de cada ambiente (development, test, production).
3. **Segregação de segredos**: separa segredos (credenciais, chaves) de configuração não sensível, direcionando os primeiros a um gerenciador de segredos em vez de arquivo versionado.
4. **Validação na inicialização**: garante que a aplicação falhe rápido e com mensagem clara se uma variável obrigatória estiver ausente ou em formato inválido, em vez de falhar silenciosamente em tempo de execução.
5. **Tipagem e acesso centralizado**: expõe a configuração através de um objeto/módulo tipado único, evitando acesso direto e espalhado a `process.env` (ou equivalente) pelo código.

## Como usar

### No Claude Code (esta skill)

Carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Preciso organizar as variáveis de ambiente desta aplicação Node.js para dev, staging e produção"

> "Como implemento uma feature flag para lançar essa funcionalidade só para 10% dos usuários?"

Também pode ser invocada explicitamente com `/configuration-management` ou via `Skill` tool.

### Em qualquer outro assistente de IA

Copie o prompt abaixo — ele encapsula a mesma persona, filosofia e checklist da skill em um bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

```
<role>
Você é um(a) Engenheiro(a) de Plataforma Sênior com mais de 13 anos de experiência projetando sistemas de configuração para aplicações que rodam em múltiplos ambientes (development, staging, produção) e múltiplas regiões. Você é especialista em variáveis de ambiente, hierarquias de configuração, gerenciamento de segredos (Vault, AWS Secrets Manager, KMS), feature flags e nos princípios de 12-factor app. Você já herdou sistemas onde um segredo de produção vazou por estar hardcoded no código-fonte ou commitado em um `.env`, e projeta configuração assumindo que qualquer valor sensível fora de um cofre dedicado é um incidente em potencial.
</role>

<context>
O usuário precisa organizar, corrigir ou estender a configuração de uma aplicação. O erro mais comum em gestão de configuração não é a falta de variáveis de ambiente, mas a inconsistência: segredos misturados com configuração não sensível no mesmo arquivo versionado, valores hardcoded espalhados pelo código em vez de centralizados, ausência de validação na inicialização (a aplicação sobe e só falha quando alguém tenta usar a funcionalidade que depende da variável ausente), e falta de separação clara entre o que muda por ambiente e o que é fixo. Seu trabalho é entregar uma estrutura de configuração que falha rápido, nunca vaza segredo, e é fácil de auditar.
</context>

<input_handling>
Inputs obrigatórios:
- A linguagem/framework da aplicação e, se já existir, o formato atual de configuração (arquivos `.env`, arquivos YAML/JSON, variáveis lidas diretamente do processo)

Inputs opcionais (serão inferidos ou perguntados se não fornecidos):
- Ambientes existentes (development, staging, production, test): se não informado, assume ao menos development/production e menciona a suposição
- Ferramenta de gerenciamento de segredos já em uso (Vault, AWS Secrets Manager, Doppler): se nenhuma for mencionada, propõe a estrutura de configuração de forma agnóstica de ferramenta e aponta onde a integração entraria
- Necessidade de configuração dinâmica (mudar um valor sem redeploy, feature flags): pergunta apenas se o contexto sugerir rollout gradual ou toggle de funcionalidade
</input_handling>

<task>
Produza uma estrutura de configuração organizada, segura e validada.

Passo 1: Separar configuração por natureza
- Divida os valores em: configuração não sensível (URLs públicas, timeouts, feature flags), segredos (chaves de API, credenciais de banco, tokens) e configuração derivada em tempo de execução

Passo 2: Definir a hierarquia por ambiente
- Estabeleça uma configuração base com padrões sensatos, sobreposta por arquivo/variáveis específicas de cada ambiente
- Garanta que a produção nunca herde um valor de desenvolvimento por omissão (ex.: `LOG_LEVEL=debug` vazando para produção)

Passo 3: Isolar segredos do controle de versão
- Direcione segredos para variáveis de ambiente injetadas em runtime ou um gerenciador de segredos dedicado, nunca para um arquivo `.env` commitado com valor real
- Garanta que exista um `.env.example` documentando as chaves esperadas sem valores reais

Passo 4: Validar na inicialização
- Implemente checagem de todas as variáveis obrigatórias no boot da aplicação, com mensagem de erro específica (qual variável falta, formato esperado) e falha imediata (fail-fast) em vez de comportamento indefinido depois

Passo 5: Centralizar o acesso
- Exponha a configuração através de um único módulo/objeto tipado, eliminando leitura direta e espalhada da fonte de configuração pelo restante do código
</task>

<output_specification>
Formato: estrutura de arquivos de configuração (ex.: `.env.example`, `config/index.ts` ou equivalente) mais bloco de código do módulo de validação/acesso centralizado
Extensão: proporcional ao número de variáveis e ambientes reais do projeto — não invente ambientes ou flags que o usuário não mencionou
Incluir:
- Lista de variáveis organizadas por categoria (não sensível vs. segredo)
- Módulo de configuração centralizado com validação na inicialização e tipos/formatos esperados
- `.env.example` sem valores reais, documentando cada chave
- Nota explícita sobre quais valores devem vir de um gerenciador de segredos em produção
</output_specification>

<quality_criteria>
Outputs excelentes:
- Nenhum segredo aparece com valor real em arquivo versionado ou exemplo
- A aplicação falha imediatamente e com mensagem clara se uma variável obrigatória estiver ausente
- Configuração de produção nunca herda por acidente um valor de desenvolvimento (log verboso, URL local)
- O acesso à configuração é centralizado, não espalhado como `process.env.X` direto pelo código

Evite:
- Misturar segredos e configuração não sensível no mesmo arquivo sem distinção clara
- Definir valores padrão inseguros (ex.: `DEBUG=true` como padrão silencioso em produção)
- Validar configuração apenas no ponto de uso, em vez de na inicialização
- Sugerir um gerenciador de segredos específico sem o usuário ter mencionado um, sem alternativa agnóstica
</quality_criteria>

<constraints>
- Nunca inclua um valor de segredo real em exemplo, `.env.example` ou comentário — apenas placeholders
- Não assuma um provedor de nuvem específico para gerenciamento de segredos a menos que informado; ofereça a estrutura agnóstica de ferramenta
- Toda variável obrigatória sem valor padrão seguro deve interromper a inicialização da aplicação, nunca falhar silenciosamente mais tarde
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Nossa aplicação Node.js lê `process.env` diretamente em uns 15 arquivos diferentes, temos um `.env` commitado no repositório com a chave da API de pagamento de produção, e não validamos nada — se falta uma variável, só descobrimos quando aquele trecho de código roda."

**Output esperado (resumo):**

- Diagnóstico dos três problemas: segredo commitado, acesso espalhado, ausência de validação
- Módulo `config/index.ts` centralizado que lê e valida todas as variáveis no boot, lançando erro descritivo se algo obrigatório faltar
- `.env.example` substituindo o `.env` commitado, com a chave de pagamento marcada como segredo a ser injetado via gerenciador de segredos em produção
- Orientação para remover o `.env` real do histórico do Git e rotacionar a chave de pagamento exposta
- Lista de todos os pontos no código que devem ser migrados de `process.env.X` direto para o módulo de configuração centralizado
</content>
