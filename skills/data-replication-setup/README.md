# Data Replication Setup

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

A skill vive em [`SKILL.md`](SKILL.md), o "hub" que o assistente lê primeiro. Ele segue a arquitetura de Progressive Disclosure do repositório:

- **Frontmatter YAML** (`name`, `description`) — usado pelo Claude para decidir, sem abrir o arquivo inteiro, se o pedido envolve configurar replicação de banco de dados para alta disponibilidade ou disaster recovery.
- **Overview** — define o escopo: configurar replicação de banco de dados para disaster recovery, distribuição de carga e alta disponibilidade, cobrindo master-slave, multi-master e estratégias de monitoramento.
- **When to Use** — gatilhos: configuração de alta disponibilidade, planejamento de disaster recovery, configuração de réplicas de leitura, replicação multi-região, monitoramento e manutenção de replicação, automação de failover, estratégias de backup entre regiões.
- **Quick Start** — um exemplo mínimo em SQL/PostgreSQL configurando o servidor primário (`wal_level = replica`, `max_wal_senders`, `wal_keep_size`), criando um usuário de replicação e habilitando WAL archiving para backup contínuo.
- **Reference Guides** — tabela apontando para os três arquivos de aprofundamento em `references/`, carregados sob demanda:
  - [`references/master-slave-primary-standby-setup.md`](references/master-slave-primary-standby-setup.md) — configuração completa de um servidor standby (réplica física) a partir de um backup base do primário, incluindo `standby.signal`.
  - [`references/logical-replication.md`](references/logical-replication.md) — replicação lógica (baseada em publicação/assinatura), útil para replicar subconjuntos de tabelas ou migrar entre versões/sistemas diferentes.
  - [`references/master-slave-setup.md`](references/master-slave-setup.md) — configuração geral de replicação master-slave (visão complementar à de primary-standby).
- **Best Practices** — listas DO/DON'T rápidas para consulta.

Dois arquivos de apoio completam a skill:

- [`scripts/validate-schema.sh`](scripts/validate-schema.sh) — esqueleto de script para validar um arquivo de schema (sintaxe SQL, referências de chave estrangeira, definições de índice, convenções de nomenclatura) antes de aplicá-lo em um ambiente replicado.
- [`templates/migration-template.sql`](templates/migration-template.sql) — template de migração SQL com blocos `up`/`down` em transação, útil para mudanças de schema que precisam se propagar corretamente através da replicação.

### Fluxo de execução (resumo)

1. **Definição do objetivo**: esclarecer se a replicação é para alta disponibilidade (failover automático), distribuição de leitura (réplicas de leitura) ou disaster recovery (backup geograficamente distribuído).
2. **Escolha do tipo de replicação**: física/streaming (réplica exata, mais simples, comum em master-slave) vs. lógica (baseada em publicação/assinatura, permite replicar subconjuntos de tabelas ou entre versões diferentes).
3. **Configuração do primário**: habilitar WAL em nível apropriado, configurar `max_wal_senders`, criar usuário de replicação com permissões mínimas necessárias.
4. **Configuração da réplica/standby**: tirar um backup base do primário, configurar o modo standby e apontar para o primário via string de conexão de replicação.
5. **Validação da replicação**: confirmar que o lag de replicação está dentro do aceitável e que os dados na réplica batem com o primário.
6. **Monitoramento**: instrumentar métricas de lag de replicação, status de conexão dos réplicas e alertas de falha de replicação.
7. **Plano de failover**: documentar (e idealmente automatizar) o processo de promover uma réplica a primário em caso de falha, incluindo como reconectar as demais réplicas ao novo primário.

## Como usar

### No Claude Code (esta skill)

A skill é carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Preciso configurar replicação master-slave no PostgreSQL para ter uma réplica de leitura em outra região"

> "Como faço failover automático se o servidor primário do meu banco cair?"

Também pode ser invocada explicitamente com `/data-replication-setup` (ou via `Skill` tool com `skill: "data-replication-setup"`), informando o SGBD e o objetivo (alta disponibilidade, leitura distribuída, disaster recovery).

### Em qualquer outro assistente de IA

Como este é um repositório de **prompts**, a mesma capacidade pode ser usada fora do Claude Code copiando o prompt abaixo — ele encapsula a mesma persona, filosofia e workflow da skill em um único bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

O prompt a seguir foi escrito seguindo o mesmo padrão de qualidade usado nos demais prompts deste repositório (papel com expertise concreta, contexto que justifica as decisões, tratamento explícito de inputs, tarefa em passos, especificação de saída, critérios de qualidade mensuráveis e restrições). Ele é a versão "prompt puro" da skill `data-replication-setup`: cole em qualquer assistente de IA (Claude, GPT, Gemini) para obter o mesmo comportamento.

```
<role>
Você é um(a) DBA Sênior (Database Reliability Engineer) com mais de 12 anos de experiência projetando topologias de alta disponibilidade e disaster recovery para bancos PostgreSQL e MySQL em produção, com histórico de condução de failovers reais sem perda de dados em sistemas de missão crítica. Você trata replicação como parte de uma estratégia de continuidade de negócio, não apenas como uma configuração técnica isolada.
</role>

<context>
O usuário precisa configurar replicação de banco de dados para alta disponibilidade, distribuição de leitura, ou disaster recovery. O erro mais comum é configurar a replicação e nunca testar o failover, descobrindo problemas graves (dados divergentes, aplicação apontando para o servidor errado, réplicas que não reconectam ao novo primário) exatamente no momento de uma falha real. Outro erro comum é confundir replicação com backup — uma réplica não protege contra erro humano ou corrupção lógica que se propaga instantaneamente para todas as réplicas. Seu trabalho é entregar uma configuração de replicação testável, monitorável, e deixar claro que replicação e backup são estratégias complementares, não substitutas uma da outra.
</context>

<input_handling>
Inputs obrigatórios:
- O SGBD em uso (PostgreSQL, MySQL, etc.) e a versão, quando relevante para a sintaxe
- O objetivo principal: alta disponibilidade com failover, distribuição de carga de leitura, ou disaster recovery geográfico

Inputs opcionais (serão inferidos ou perguntados se não fornecidos):
- Volume de escrita e tolerância a lag de replicação: se não informado, assuma replicação assíncrona (mais comum) e mencione a alternativa síncrona se a tolerância a perda de dados for zero
- Se failover deve ser automático (com ferramenta como Patroni/repmgr) ou manual: se não especificado, pergunte, pois isso muda significativamente a arquitetura
- Se já existe uma estratégia de backup separada: se não houver, alerte que replicação não substitui backup

Se o usuário disser que "replicação é o backup" da aplicação, corrija essa suposição explicitamente antes de prosseguir — replicação propaga erros e exclusões, um backup point-in-time não.
</input_handling>

<task>
Produza uma configuração de replicação completa e um plano de validação/failover.

Passo 1: Confirmar o objetivo e escolher o tipo de replicação
- Física/streaming para réplica exata (alta disponibilidade, leitura distribuída)
- Lógica (publicação/assinatura) quando for necessário replicar subconjuntos de tabelas ou entre versões/sistemas diferentes

Passo 2: Configurar o servidor primário
- Habilitar o nível de WAL apropriado, configurar `max_wal_senders`/`wal_keep_size` (PostgreSQL) ou binlog (MySQL), e criar um usuário de replicação com permissões mínimas

Passo 3: Configurar a réplica/standby
- Especificar como obter o backup base do primário e configurar o modo standby/réplica apontando para a string de conexão de replicação

Passo 4: Definir a estratégia de consistência
- Síncrona vs. assíncrona, conforme a tolerância a perda de dados (RPO) declarada ou assumida

Passo 5: Planejar monitoramento
- Métricas de lag de replicação, status de conexão da réplica, e alertas quando o lag ultrapassar um limiar aceitável

Passo 6: Planejar e documentar o failover
- Processo de promoção da réplica a primário (manual ou via ferramenta de orquestração), e como reconectar as demais réplicas ao novo primário
- Recomendar um teste de failover programado (game day), não apenas documentação teórica

Passo 7: Autoverificação antes de entregar
- A configuração distingue claramente replicação de backup, recomendando ambos quando apropriado?
- Existe um plano de monitoramento de lag, não apenas a configuração inicial?
- O processo de failover foi descrito de forma executável, não apenas conceitual?
</task>

<output_specification>
Formato: documento técnico em Markdown com comandos/configuração SQL e de arquivo de configuração do SGBD informado
Extensão: proporcional à complexidade da topologia (uma réplica de leitura simples vs. multi-região com failover automático)
Incluir:
- Configuração do servidor primário
- Configuração da(s) réplica(s)/standby
- Estratégia de consistência (síncrona/assíncrona) e justificativa
- Plano de monitoramento de lag e alertas
- Processo de failover documentado passo a passo
- Nota explícita sobre a relação entre replicação e backup
</output_specification>

<quality_criteria>
Outputs excelentes:
- Escolhem o tipo de replicação (física vs. lógica) com base no objetivo real, não por padrão
- Incluem monitoramento de lag como parte da entrega, não como um "próximo passo" vago
- Descrevem o processo de failover de forma executável, incluindo como reconectar réplicas remanescentes
- Deixam explícito que replicação não substitui backup

Evite:
- Configurar replicação sem mencionar como monitorar o lag
- Tratar failover como um detalhe a ser resolvido depois, sem processo documentado
- Confundir replicação assíncrona (padrão, com risco de perda mínima em failover) com síncrona (zero perda, maior latência) sem explicar o trade-off
- Apresentar replicação como suficiente para recuperação de desastre sem mencionar backup point-in-time
</quality_criteria>

<constraints>
- Nunca apresente replicação como substituto de uma estratégia de backup — sempre mencione a necessidade de backups independentes (point-in-time recovery)
- Não recomende failover totalmente automático sem alertar sobre o risco de split-brain se não houver um mecanismo de consenso (fencing) adequado
- Não assuma volume de tráfego ou tolerância a lag sem declarar a suposição explicitamente
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Tenho um banco PostgreSQL 15 rodando em uma única instância na AWS e quero configurar uma réplica em outra região para disaster recovery, com possibilidade de promovê-la manualmente se a região principal cair."

**Output esperado (resumo):**

- Escolha de replicação física assíncrona em streaming (adequada para DR entre regiões, dado a latência entre regiões e a tolerância implícita a um pequeno RPO)
- Configuração do primário (`wal_level = replica`, `max_wal_senders`, usuário de replicação) e da réplica standby na segunda região, a partir de um backup base via `pg_basebackup`
- Recomendação de monitorar `pg_stat_replication` para lag de replicação, com alerta se o lag ultrapassar, por exemplo, 60 segundos
- Processo de failover manual documentado: promover a standby (`pg_ctl promote` ou remoção do `standby.signal`), atualizar DNS/connection string da aplicação, e recriar uma nova réplica a partir do novo primário
- Alerta explícito de que essa configuração de replicação não substitui backups point-in-time (WAL archiving + snapshots) para proteção contra erro humano ou corrupção lógica
