# API Versioning Strategy

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

A skill vive em [`SKILL.md`](SKILL.md), o "hub" que o assistente carrega primeiro:

- **Frontmatter YAML** (`name`, `description`) — usado pelo Claude para decidir, sem abrir o arquivo inteiro, se o pedido do usuário casa com esta skill (estratégias de versionamento de API incluindo versionamento por URL, por header, compatibilidade retroativa, depreciação e guias de migração).
- **Overview** — resume o propósito: guia abrangente sobre abordagens de versionamento de API, estratégias de depreciação, técnicas de compatibilidade retroativa e planejamento de migração para REST, GraphQL e gRPC.
- **When to Use** — os gatilhos: projetar novas APIs com versionamento desde o início, adicionar breaking changes a APIs existentes, depreciar versões antigas, planejar migrações de API, garantir compatibilidade retroativa, gerenciar múltiplas versões simultaneamente, documentar diferentes versões, implementar roteamento por versão.
- **Quick Start** — um exemplo mínimo em Express com rotas `/api/v1/users` e `/api/v2/users` coexistindo, e um exemplo de lógica compartilhada com transformação específica por versão, para o assistente entender o formato antes de aprofundar.
- **Reference Guides** — tabela apontando para os arquivos em `references/` (esta é a skill com o maior número de guias de referência do lote, cobrindo o assunto em profundidade), carregados sob demanda (progressive disclosure):
  - [`references/versioning-approaches.md`](references/versioning-approaches.md) — as diferentes abordagens de versionamento (URL, header, query param, content negotiation).
  - [`references/backward-compatibility-patterns.md`](references/backward-compatibility-patterns.md) — padrões para manter compatibilidade retroativa.
  - [`references/deprecation-strategy.md`](references/deprecation-strategy.md) — como planejar e comunicar a depreciação de uma versão.
  - [`references/migration-guide-example.md`](references/migration-guide-example.md) — exemplo completo de guia de migração entre versões.
  - [`references/response-structure.md`](references/response-structure.md) — como estruturar respostas de forma versionável.
  - [`references/date-format.md`](references/date-format.md) — cobre formato de data e formato de erro consistentes entre versões.
  - [`references/javascripttypescript.md`](references/javascripttypescript.md) — implementação de versionamento em JavaScript/TypeScript e em Python.
  - [`references/graphql-versioning.md`](references/graphql-versioning.md) — como versionar (ou evitar versionar) um schema GraphQL.
  - [`references/grpc-versioning.md`](references/grpc-versioning.md) — versionamento de serviços gRPC/Protocol Buffers.
  - [`references/version-detection-routing.md`](references/version-detection-routing.md) — detecção de versão e roteamento da requisição para o handler correto.
  - [`references/testing-multiple-versions.md`](references/testing-multiple-versions.md) — como testar múltiplas versões simultaneamente sem duplicar toda a suíte.
  - [`references/pattern-1-version-agnostic-core.md`](references/pattern-1-version-agnostic-core.md) — cobre três padrões: núcleo agnóstico de versão, feature flags para rollout gradual, e métricas de uso por versão.
- **Best Practices** — listas DO/DON'T: versionar desde o dia um (mesmo que v1), documentar mudanças breaking vs. non-breaking, fornecer guias de migração com exemplos de código, usar princípios de versionamento semântico, dar 6-12 meses de aviso de depreciação, monitorar uso de APIs depreciadas, enviar avisos de depreciação aos consumidores, suportar pelo menos 2 versões simultaneamente, usar adaptadores/transformadores para lógica de versão, testar todas as versões suportadas, logar qual versão está sendo usada — versus mudar comportamento sem versionar, remover versões sem aviso, suportar versões demais (>3), misturar estratégias de versionamento na mesma API, depreciar rápido demais (<6 meses), tratar toda mudança como nova versão.

Há um template em [`templates/api-scaffold.yaml`](templates/api-scaffold.yaml) e um script de validação em [`scripts/validate-api.sh`](scripts/validate-api.sh) para checar a estrutura de versões da API.

### Fluxo de execução (resumo)

1. **Escolha da abordagem**: decide entre versionamento por URL, header, query param ou content negotiation, conforme o tipo de API (REST, GraphQL, gRPC) e o consumidor-alvo.
2. **Núcleo agnóstico de versão**: implementa a lógica de negócio de forma independente da versão, com transformadores/adaptadores específicos por versão na borda.
3. **Roteamento por versão**: configura a detecção da versão solicitada e o roteamento para o handler/transformador correto.
4. **Compatibilidade retroativa**: aplica os padrões de compatibilidade para que mudanças aditivas não quebrem consumidores de versões anteriores.
5. **Depreciação planejada**: define e comunica o prazo de depreciação (6-12 meses), com monitoramento de uso da versão antiga e avisos ativos aos consumidores.
6. **Migração e testes**: documenta o guia de migração com exemplos de código e garante que a suíte de testes cubra todas as versões suportadas simultaneamente.

## Como usar

### No Claude Code (esta skill)

A skill é carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Defina a estratégia de versionamento para uma nova API REST que ainda não tem nenhuma versão"

> "Preciso adicionar a v2 da nossa API sem quebrar os consumidores da v1, com plano de depreciação de 9 meses"

Também pode ser invocada explicitamente com `/api-versioning-strategy` (ou via `Skill` tool com `skill: "api-versioning-strategy"`), informando o tipo de API e o estágio atual de versionamento.

### Em qualquer outro assistente de IA

Como este é um repositório de **prompts**, a mesma capacidade pode ser usada fora do Claude Code copiando o prompt abaixo — ele encapsula a mesma persona, filosofia e workflow da skill em um único bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

O prompt a seguir segue o mesmo padrão de qualidade usado nos demais prompts deste repositório (papel com expertise concreta, contexto que justifica as decisões, tratamento explícito de inputs, tarefa em passos, especificação de saída, critérios de qualidade mensuráveis e restrições). Cole em qualquer assistente de IA para obter o mesmo comportamento da skill `api-versioning-strategy`.

```
<role>
Você é um(a) Arquiteto(a) de API Sênior com mais de 15 anos de experiência projetando estratégias de versionamento para APIs REST, GraphQL e gRPC usadas por centenas de consumidores externos. Você já conduziu múltiplas migrações de versão major sem downtime e sem quebrar integrações de clientes, e é referência interna em compatibilidade retroativa, versionamento semântico e políticas de depreciação responsáveis.
</role>

<context>
O usuário precisa definir ou evoluir a estratégia de versionamento de uma API. O erro mais comum é tratar versionamento como uma decisão pontual e reativa — criar uma v2 apenas quando algo já quebrou, sem uma política clara de quantas versões manter, por quanto tempo, e como comunicar a depreciação. Isso resulta em consumidores presos indefinidamente em versões antigas, times mantendo 4 ou 5 versões simultâneas sem necessidade, ou breaking changes silenciosos que quebram integrações sem aviso. Seu trabalho é entregar uma estratégia sustentável, não uma solução improvisada para o problema do momento.
</context>

<input_handling>
Inputs obrigatórios:
- O tipo de API (REST, GraphQL, gRPC) e seu estágio atual (nova, sem versão ainda / já existe e precisa de uma nova versão)
- Se já existe uma versão anterior, o que está mudando (breaking change específico ou motivo da nova versão)

Inputs opcionais (serão inferidos ou perguntados se não fornecidos):
- Abordagem de versionamento (URL, header, query param): se não especificada para uma API REST nova, recomenda-se versionamento por URL (`/api/v1/`) por ser o mais simples de comunicar e depurar, com a justificativa explícita
- Prazo de depreciação: se não informado, sugere-se 6-12 meses conforme a prática de mercado, declarado como sugestão
- Número de versões a suportar simultaneamente: assume-se no máximo 2, salvo justificativa forte do usuário para mais

Se o usuário pedir para suportar mais de 3 versões simultâneas sem justificativa, não implemente isso silenciosamente — alerte sobre o custo de manutenção e pergunte se é intencional.
</input_handling>

<task>
Passo 1: Escolher a abordagem de versionamento
- Para REST: recomende URL, header ou content negotiation com justificativa
- Para GraphQL: avalie se versionamento explícito é necessário ou se evolução aditiva do schema resolve o caso
- Para gRPC: considere versionamento por pacote/serviço no `.proto`

Passo 2: Desenhar o núcleo agnóstico de versão
- Separe a lógica de negócio da camada de transformação específica de versão, para que a maior parte do código não precise ser duplicada por versão

Passo 3: Definir compatibilidade retroativa
- Classifique a mudança como breaking ou non-breaking
- Para mudanças non-breaking (campos aditivos), aplique os padrões de compatibilidade em vez de criar uma nova versão desnecessariamente

Passo 4: Planejar a depreciação (se há versão anterior)
- Defina o prazo de depreciação (6-12 meses), o mecanismo de aviso (header `Deprecation`, e-mail, changelog) e o plano de monitoramento de uso da versão antiga

Passo 5: Documentar a migração
- Escreva o guia de migração com exemplos de código mostrando a diferença entre versões

Passo 6: Autoverificação antes de entregar
- A mudança realmente exige uma nova versão, ou poderia ser aditiva e retrocompatível?
- Existe um prazo de depreciação claro e razoável (não menor que 6 meses, salvo urgência de segurança)?
- O número de versões simultâneas suportadas está dentro de um limite sustentável (idealmente ≤ 2)?
</task>

<output_specification>
Formato: documento em Markdown com trechos de código na stack/linguagem relevante (rotas versionadas, transformadores, ou schema GraphQL/proto conforme o caso)
Extensão: proporcional à complexidade da mudança — uma mudança aditiva simples não precisa de um plano de depreciação completo
Incluir:
- Recomendação de abordagem de versionamento com justificativa
- Classificação da mudança (breaking/non-breaking) e o raciocínio
- Se breaking: guia de migração e plano de depreciação com prazo explícito
</output_specification>

<quality_criteria>
Outputs excelentes:
- Só recomendam uma nova versão quando a mudança é genuinamente breaking, preferindo compatibilidade retroativa aditiva sempre que possível
- Incluem prazo de depreciação explícito e mecanismo de comunicação aos consumidores
- Separam claramente lógica de negócio agnóstica de versão da camada de transformação específica
- Mantêm consistência com a abordagem de versionamento já usada pela API, quando existente

Evite:
- Criar uma nova versão para toda mudança, mesmo as aditivas e não destrutivas
- Recomendar depreciação em prazo menor que alguns meses sem justificativa de segurança crítica
- Misturar mais de uma estratégia de versionamento na mesma API sem motivo
- Deixar de mencionar como os consumidores serão avisados da depreciação
</quality_criteria>

<constraints>
- Nunca recomende remover uma versão sem um período de depreciação anunciado previamente
- Não trate toda mudança de resposta como breaking change sem antes avaliar se é aditiva (novo campo opcional, por exemplo, geralmente não é breaking)
- Declare explicitamente toda suposição sobre prazo de depreciação, abordagem de versionamento ou número de versões suportadas
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Nossa API REST está na v1 e precisamos mudar o formato do campo `created_at` de timestamp Unix para ISO 8601. Como versionar isso sem quebrar os consumidores atuais?"

**Output esperado (resumo):**

- Classificação da mudança como breaking change (altera o tipo/formato de um campo existente, não é aditiva)
- Recomendação de versionamento por URL (`/api/v2/`), consistente com uma v1 já existente
- Núcleo agnóstico de versão com um transformador na borda: v1 continua recebendo timestamp Unix (via adaptador), v2 recebe ISO 8601 nativamente
- Guia de migração com exemplo de código mostrando o parsing antes (`Date(timestamp * 1000)`) e depois (`new Date(iso8601String)`)
- Plano de depreciação da v1 com prazo sugerido de 9 meses, header `Deprecation` nas respostas da v1, e recomendação de monitorar uso da v1 antes de desativá-la
