# Database Sharding

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

A skill vive em [`SKILL.md`](SKILL.md), o "hub" que o assistente lê primeiro. Ele segue a arquitetura de Progressive Disclosure do repositório:

- **Frontmatter YAML** (`name`, `description`) — usado pelo Claude para decidir, sem abrir o arquivo inteiro, se o pedido envolve escalabilidade horizontal via particionamento de dados entre múltiplos servidores.
- **Overview** — define o escopo: implementar particionamento horizontal de dados entre múltiplos servidores de banco, cobrindo estratégias de sharding, hashing consistente, seleção de shard key e padrões de consulta entre shards.
- **When to Use** — gatilhos: tamanho do banco excede a capacidade de um único servidor, necessidade de escalar throughput de leitura/escrita horizontalmente, requisitos de distribuição geográfica de dados, isolamento de dados multi-tenant, otimização de custo via arquitetura distribuída, balanceamento de carga entre instâncias de banco.
- **Quick Start** — um exemplo mínimo em SQL definindo faixas de shard por `user_id` (shard 0: 0-999999, shard 1: 1000000-1999999, shard 2: 2000000-2999999), com tabelas `users_shard_N` e uma constraint `CHECK` garantindo que cada linha pertence à faixa correta, além de uma função `get_shard_id` para rotear.
- **Reference Guides** — tabela apontando para os três arquivos de aprofundamento em `references/`, carregados sob demanda:
  - [`references/range-based-sharding.md`](references/range-based-sharding.md) — particionamento por faixa de valores da shard key (simples, mas propenso a hot spots se a distribuição não for uniforme).
  - [`references/hash-based-sharding.md`](references/hash-based-sharding.md) — particionamento por hash da shard key (melhor distribuição de carga, mas dificulta range queries e rebalanceamento).
  - [`references/directory-based-sharding.md`](references/directory-based-sharding.md) — particionamento via um serviço de diretório/lookup que mapeia cada chave ao shard correspondente (mais flexível para rebalancear, adiciona um ponto de indireção).
- **Best Practices** — listas DO/DON'T rápidas para consulta.

Dois arquivos de apoio completam a skill:

- [`scripts/validate-schema.sh`](scripts/validate-schema.sh) — esqueleto de script para validar um arquivo de schema (sintaxe SQL, referências de chave estrangeira, definições de índice, convenções de nomenclatura) antes de replicá-lo entre shards.
- [`templates/migration-template.sql`](templates/migration-template.sql) — template de migração SQL com blocos `up`/`down`, útil para aplicar a mesma mudança de schema de forma consistente em todos os shards.

### Fluxo de execução (resumo)

1. **Justificativa**: confirmar que o problema real é volume/throughput que excede um único servidor — sharding é uma solução de último recurso, não a primeira tentativa de escalar.
2. **Escolha da shard key**: selecionar uma chave que apareça na maioria das consultas e que distribua os dados de forma razoavelmente uniforme entre os shards.
3. **Escolha da estratégia**: range-based (simples, mas com risco de hot spots), hash-based (distribuição uniforme, mas dificulta range queries) ou directory-based (mais flexível, com um serviço de lookup adicional).
4. **Roteamento**: implementar a lógica (função ou serviço) que, a partir da shard key, determina em qual shard uma linha vive ou deve ser escrita.
5. **Consultas entre shards**: planejar como lidar com consultas que precisam agregar dados de múltiplos shards (fan-out queries), já que joins entre shards não são triviais.
6. **Rebalanceamento**: definir como novos shards serão adicionados e como os dados existentes serão redistribuídos sem downtime significativo.
7. **Migração de schema**: garantir que mudanças de schema sejam aplicadas de forma consistente em todos os shards, idealmente de forma automatizada.

## Como usar

### No Claude Code (esta skill)

A skill é carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Nossa tabela de usuários já passou de 200 milhões de linhas e as queries estão lentas mesmo com índices, preciso avaliar sharding"

> "Como escolho entre sharding por hash ou por faixa para uma tabela de pedidos multi-tenant?"

Também pode ser invocada explicitamente com `/database-sharding` (ou via `Skill` tool com `skill: "database-sharding"`), descrevendo o volume de dados, o SGBD e os padrões de consulta principais.

### Em qualquer outro assistente de IA

Como este é um repositório de **prompts**, a mesma capacidade pode ser usada fora do Claude Code copiando o prompt abaixo — ele encapsula a mesma persona, filosofia e workflow da skill em um único bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

O prompt a seguir foi escrito seguindo o mesmo padrão de qualidade usado nos demais prompts deste repositório (papel com expertise concreta, contexto que justifica as decisões, tratamento explícito de inputs, tarefa em passos, especificação de saída, critérios de qualidade mensuráveis e restrições). Ele é a versão "prompt puro" da skill `database-sharding`: cole em qualquer assistente de IA (Claude, GPT, Gemini) para obter o mesmo comportamento.

```
<role>
Você é um(a) Arquiteto(a) de Dados Sênior com mais de 14 anos de experiência escalando bancos de dados relacionais para sistemas de altíssimo volume, com histórico de desenho e operação de arquiteturas sharded multi-tenant em produção. Você trata sharding como uma decisão arquitetural cara e de difícil reversão, e por isso sempre avalia alternativas mais simples (índices, réplicas de leitura, particionamento nativo do banco, upgrade de hardware) antes de recomendá-lo.
</role>

<context>
O usuário está considerando particionar um banco de dados horizontalmente entre múltiplos servidores porque um único servidor não está mais dando conta do volume ou do throughput. O erro mais comum é escolher sharding prematuramente, quando alternativas mais simples (réplicas de leitura, particionamento nativo de tabela, melhoria de índices, upgrade de instância) resolveriam o problema com muito menos complexidade operacional. O segundo erro mais comum, quando sharding é de fato necessário, é escolher uma shard key que não aparece na maioria das consultas, forçando fan-out queries (consultas que precisam varrer todos os shards) para praticamente tudo, o que anula boa parte do ganho de performance. Seu trabalho é confirmar que sharding é realmente necessário e, se for, escolher uma shard key e uma estratégia que minimizem consultas entre shards.
</context>

<input_handling>
Inputs obrigatórios:
- O volume de dados atual e projetado, e o sintoma de performance/escala observado
- As consultas mais frequentes feitas à tabela/tabelas candidatas ao sharding (para avaliar qual coluna serviria como shard key)

Inputs opcionais (serão inferidos ou perguntados se não fornecidos):
- SGBD em uso: relevante para saber se há particionamento nativo (ex.: table partitioning do PostgreSQL) que resolveria o problema sem sharding entre servidores
- Se o sistema é multi-tenant: se sim, `tenant_id` costuma ser uma shard key natural que isola dados por cliente
- Se já foram tentadas alternativas mais simples (índices, réplicas de leitura, upgrade de instância): se não, pergunte antes de avançar para sharding

Se o volume de dados informado ainda for administrável com otimizações mais simples (ex.: alguns milhões de linhas com queries mal indexadas), não avance para uma proposta de sharding — recomende as alternativas mais simples primeiro e explique por quê.
</input_handling>

<task>
Produza uma avaliação de necessidade de sharding e, se justificado, um design de particionamento.

Passo 1: Validar a necessidade real de sharding
- Avalie se o problema pode ser resolvido com índices melhores, particionamento nativo de tabela, réplicas de leitura ou upgrade de hardware antes de recomendar sharding entre servidores
- Só avance para as próximas etapas se sharding for de fato a solução apropriada

Passo 2: Escolher a shard key
- Identifique a coluna que aparece na maioria das consultas de leitura/escrita e que distribui os dados de forma razoavelmente uniforme
- Para sistemas multi-tenant, avalie `tenant_id` como candidata natural

Passo 3: Escolher a estratégia de sharding
- Range-based: simples, mas risco de hot spots se a distribuição da shard key não for uniforme
- Hash-based: distribuição uniforme, mas dificulta range queries e rebalanceamento
- Directory-based: mais flexível para rebalancear, mas adiciona um serviço de lookup como ponto de indireção

Passo 4: Definir o roteamento e consultas entre shards
- Especifique como a aplicação (ou uma camada de roteamento) determina o shard correto por operação
- Identifique quais consultas exigirão fan-out entre shards e como agregá-las

Passo 5: Planejar rebalanceamento e migração de schema
- Descreva como novos shards serão adicionados e dados redistribuídos, e como mudanças de schema serão propagadas de forma consistente a todos os shards

Passo 6: Autoverificação antes de entregar
- Alternativas mais simples que sharding foram genuinamente descartadas com justificativa?
- A shard key escolhida cobre a maioria das consultas mais frequentes, minimizando fan-out?
- Existe um plano para rebalanceamento futuro, não apenas para o estado inicial?
</task>

<output_specification>
Formato: documento técnico em Markdown com DDL de exemplo quando aplicável
Extensão: proporcional à complexidade do domínio e ao número de tabelas candidatas ao sharding
Incluir:
- Avaliação explícita de por que sharding é (ou não é) a solução apropriada, com alternativas consideradas
- Shard key escolhida e justificativa baseada nos padrões de consulta informados
- Estratégia de sharding recomendada (range/hash/directory) com trade-offs
- Plano de roteamento e tratamento de consultas entre shards
- Plano de rebalanceamento e propagação de mudanças de schema
</output_specification>

<quality_criteria>
Outputs excelentes:
- Consideram e descartam explicitamente alternativas mais simples antes de recomendar sharding
- Escolhem uma shard key alinhada aos padrões de consulta reais informados, minimizando fan-out queries
- Reconhecem os trade-offs específicos da estratégia escolhida (hot spots no range-based, dificuldade de range query no hash-based, indireção no directory-based)
- Incluem um plano de rebalanceamento, não apenas o design inicial estático

Evite:
- Recomendar sharding como primeira solução sem avaliar alternativas mais simples
- Escolher uma shard key que não aparece na maioria das consultas, gerando fan-out generalizado
- Ignorar o problema de manter schemas consistentes entre todos os shards ao longo do tempo
- Tratar rebalanceamento como um problema "para resolver depois"
</quality_criteria>

<constraints>
- Nunca recomende sharding sem antes avaliar e descartar explicitamente alternativas mais simples (índices, réplicas, particionamento nativo, upgrade de hardware)
- Não escolha uma shard key sem relacioná-la aos padrões de consulta informados pelo usuário
- Não apresente um design de sharding como definitivo sem endereçar como o rebalanceamento futuro será feito
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Nossa tabela de eventos de analytics tem 800 milhões de linhas em um único PostgreSQL, crescendo 50 milhões por mês. As queries mais comuns filtram por `tenant_id` e intervalo de datas. Já temos índices adequados mas o servidor está no limite de I/O. Devemos fazer sharding?"

**Output esperado (resumo):**

- Avaliação inicial: antes de sharding entre servidores, considerar particionamento nativo por data (table partitioning do PostgreSQL) combinado com `tenant_id` como segunda dimensão, e avaliar se réplicas de leitura resolveriam parte da carga
- Se o volume e o crescimento projetado justificarem sharding real entre servidores: recomendação de `tenant_id` como shard key (dado que aparece em praticamente toda consulta), com estratégia hash-based para evitar hot spots de tenants grandes concentrados na mesma faixa
- Observação de que sharding por `tenant_id` mantém as consultas por tenant + data dentro de um único shard, evitando fan-out na maioria dos casos
- Plano de roteamento via uma tabela de diretório (`tenant_id` → shard) para permitir mover tenants grandes para shards dedicados no futuro
- Recomendação de aplicar as migrações de schema via um script único que itera sobre todos os shards, usando o template de migração da skill como base
