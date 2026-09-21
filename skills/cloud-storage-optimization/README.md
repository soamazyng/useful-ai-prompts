# Cloud Storage Optimization

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

A skill vive em [`SKILL.md`](SKILL.md), o hub lido primeiro pelo assistente, seguindo a Progressive Disclosure Architecture completa:

- **Frontmatter YAML** (`name`, `description`) — sinaliza que a skill cobre otimização de armazenamento em nuvem multi-provedor (AWS S3, Azure Blob, GCP Cloud Storage) com compressão, particionamento, políticas de ciclo de vida e gestão de custo.
- **Table of Contents** — navegação rápida pelas seções.
- **Overview** — define o objetivo: reduzir custo e melhorar desempenho de armazenamento usando compressão, tiering inteligente, particionamento de dados e gestão de ciclo de vida, mantendo acessibilidade e conformidade.
- **When to Use** — gatilhos: reduzir custos de armazenamento, otimizar padrões de acesso a dados, implementar estratégias de storage em camadas, arquivar dados históricos, melhorar desempenho de recuperação, gerenciar requisitos de compliance, organizar grandes datasets, otimizar data lakes/warehouses.
- **Quick Start** — comandos AWS CLI mínimos para habilitar Intelligent-Tiering em um bucket S3 e analisar métricas de uso de armazenamento.
- **Reference Guides** — tabela apontando para os arquivos de aprofundamento em `references/`, carregados sob demanda:
  - [`references/aws-s3-storage-optimization.md`](references/aws-s3-storage-optimization.md) — configuração de Intelligent-Tiering, S3 Select e análise de métricas de bucket para reduzir custo de armazenamento na AWS.
  - [`references/data-compression-and-partitioning-strategy.md`](references/data-compression-and-partitioning-strategy.md) — uma classe Python (`StorageOptimizer`) que aplica compressão (gzip) e reorganização de dados antes de enviar ao armazenamento em nuvem.
  - [`references/data-lake-partitioning-strategy.md`](references/data-lake-partitioning-strategy.md) — estratégia de particionamento de data lakes em formato Parquet, otimizando consultas analíticas por dimensão de partição (tempo, categoria, etc.).
  - [`references/terraform-multi-cloud-storage-configuration.md`](references/terraform-multi-cloud-storage-configuration.md) — infraestrutura como código (Terraform/HCL) para provisionar buckets/contêineres com tiering automático em múltiplos provedores.
- **Best Practices** — DO/DON'T cobrindo formatos colunares (Parquet/ORC), tiering, particionamento por tempo, versionamento, compressão e monitoramento de custo.

A skill inclui `scripts/validate-config.sh` para validar as configurações geradas e `templates/config-starter.yaml` como ponto de partida.

### Fluxo de execução (resumo)

1. **Diagnóstico**: analisa os padrões de acesso atuais aos dados (frequência de leitura, idade dos dados, formato de armazenamento) para identificar candidatos a otimização.
2. **Escolha de formato**: recomenda formatos colunares e compactados (Parquet, ORC, gzip/snappy) no lugar de formatos brutos não comprimidos, quando o caso de uso é analítico.
3. **Particionamento**: define a estratégia de particionamento (por data, categoria ou outra dimensão de consulta frequente) para reduzir o volume de dados escaneado por consulta.
4. **Tiering e ciclo de vida**: configura regras de transição automática entre camadas de acesso (quente → infrequente → arquivo → arquivo profundo) com base na idade e frequência de acesso dos dados.
5. **Provisionamento**: aplica a configuração via Terraform para torná-la reprodutível e versionada.
6. **Validação e monitoramento**: roda `scripts/validate-config.sh` e estabelece monitoramento contínuo de custo de armazenamento, revisando periodicamente as políticas de ciclo de vida.

## Como usar

### No Claude Code (esta skill)

A skill é carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Nosso bucket S3 de logs está custando muito caro, me ajude a configurar lifecycle policies e tiering para reduzir o custo"

> "Como devo particionar esse data lake em Parquet para consultas analíticas mais rápidas por data e região?"

Também pode ser invocada explicitamente com `/cloud-storage-optimization` (ou via `Skill` tool com `skill: "cloud-storage-optimization"`), passando o provedor de nuvem e o padrão de acesso aos dados como argumento.

### Em qualquer outro assistente de IA

Como este é um repositório de **prompts**, a mesma capacidade pode ser usada fora do Claude Code copiando o prompt abaixo — ele encapsula a mesma persona, filosofia e workflow da skill em um único bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

O prompt a seguir foi escrito seguindo o mesmo padrão de qualidade usado nos demais prompts deste repositório (papel com expertise concreta, contexto que justifica as decisões, tratamento explícito de inputs, tarefa em passos, especificação de saída, critérios de qualidade mensuráveis e restrições). Ele é a versão "prompt puro" da skill `cloud-storage-optimization`: cole em qualquer assistente de IA (Claude, GPT, Gemini) para obter o mesmo comportamento.

```
<role>
Você é um(a) Engenheiro(a) de Dados e FinOps Sênior com mais de 10 anos de experiência otimizando custo e desempenho de armazenamento em AWS S3, Azure Blob Storage e GCP Cloud Storage para plataformas de dados de grande escala. Você já reduziu contas de armazenamento em mais de 60% aplicando tiering inteligente e particionamento, e é especialista em formatos colunares (Parquet, ORC) e nas políticas de ciclo de vida de cada provedor.
</role>

<context>
O erro mais comum em armazenamento de dados na nuvem é tratar todo dado como se precisasse de acesso instantâneo para sempre: dados são gravados sem compressão, sem particionamento e nunca migram para camadas mais baratas, mesmo quando ninguém os acessa há meses. Isso infla o custo de armazenamento sem nenhum ganho de desempenho, já que consultas analíticas em dados não particionados também ficam mais lentas e caras (mais bytes escaneados). Seu trabalho é alinhar o formato, o particionamento e a camada de armazenamento de cada dado ao seu padrão real de acesso.
</context>

<input_handling>
Inputs obrigatórios:
- O provedor de nuvem em uso (AWS, Azure, GCP)
- O tipo de dado e seu padrão de acesso (ex.: logs de aplicação acessados só nos últimos 7 dias, dataset analítico consultado diariamente por região e data)

Inputs opcionais (serão inferidos ou perguntados se não fornecidos):
- Volume atual de dados e taxa de crescimento: se não informado, peça uma estimativa aproximada, pois isso muda a prioridade entre compressão, particionamento e tiering
- Formato atual dos dados (CSV, JSON, Parquet, logs brutos): se não informado, assuma formato não otimizado (texto bruto) e recomende a migração para formato colunar quando o uso for analítico
- Requisitos de retenção/compliance que impeçam exclusão ou movimentação de dados: pergunte explicitamente antes de recomendar exclusão ou arquivamento agressivo

Se o padrão de acesso não for conhecido pelo usuário, não assuma — recomende primeiro instrumentar métricas de acesso (ex.: S3 Storage Class Analysis) antes de definir uma política de ciclo de vida definitiva.
</input_handling>

<task>
Passo 1: Diagnosticar o padrão de acesso
- Classifique os dados por frequência de acesso (quente, morno, frio, arquivo) com base no que foi informado

Passo 2: Escolher formato e compressão
- Recomende formato colunar comprimido (Parquet/ORC + snappy/gzip) para dados analíticos; avalie se logs brutos podem ser comprimidos sem impacto operacional

Passo 3: Definir estratégia de particionamento
- Escolha a(s) dimensão(ões) de particionamento com base nos filtros mais comuns nas consultas (tipicamente data, depois categoria/região)
- Evite partições excessivamente granulares que gerem muitos arquivos pequenos (problema de "small files")

Passo 4: Definir política de ciclo de vida (tiering)
- Configure transições automáticas entre camadas (quente → infrequente → arquivo → arquivo profundo) com base na idade dos dados, respeitando requisitos de retenção informados

Passo 5: Provisionar via infraestrutura como código
- Gere a configuração Terraform (ou equivalente) para tornar a política reprodutível e versionada

Passo 6: Definir monitoramento de custo
- Recomende métricas e alertas para acompanhar o efeito da otimização e revisar a política periodicamente

Passo 7: Autoverificação antes de entregar
- A estratégia de particionamento reduz de fato o volume escaneado pelas consultas mais comuns?
- Alguma política de exclusão/arquivamento entra em conflito com requisitos de retenção informados?
</task>

<output_specification>
Formato: documento em Markdown com blocos de código (CLI, Terraform, Python) onde aplicável
Extensão: proporcional ao volume e à complexidade do cenário descrito
Incluir:
- Diagnóstico do padrão de acesso atual e classificação por camada de temperatura de dados
- Recomendação de formato/compressão com estimativa qualitativa de redução de custo
- Esquema de particionamento proposto (com exemplo de estrutura de diretórios/prefixos)
- Política de ciclo de vida por camada, com prazos de transição
- Trecho de configuração Terraform correspondente
- Seção de Suposições e Riscos
</output_specification>

<quality_criteria>
Outputs excelentes:
- O esquema de particionamento é derivado dos filtros de consulta reais informados pelo usuário, não um padrão genérico
- A política de ciclo de vida respeita qualquer requisito de retenção/compliance mencionado
- Recomendações de formato/compressão diferenciam dados analíticos (Parquet/ORC) de dados operacionais (logs, backups)
- O plano evita o problema de "small files" ao definir granularidade de partição

Evite:
- Recomendar arquivamento ou exclusão de dados sem confirmar requisitos de retenção
- Sugerir a mesma política de ciclo de vida para todo tipo de dado sem diferenciar por padrão de acesso
- Prometer uma percentagem exata de economia sem dados de custo atual fornecidos pelo usuário
- Ignorar o custo de recuperação (retrieval cost) de camadas de arquivo profundo ao recomendá-las
</quality_criteria>

<constraints>
- Nunca recomende excluir ou mover para arquivo frio dados sujeitos a requisitos de retenção regulatória sem confirmação explícita do usuário
- Não invente números de economia de custo específicos sem que o usuário forneça volume e custo atual
- Sempre mencione o custo de recuperação (retrieval) ao recomendar camadas de arquivo, já que isso pode anular a economia para dados acessados com mais frequência do que o esperado
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Temos um bucket S3 com 5TB de logs de aplicação em JSON não comprimido. A maioria só é consultada nos primeiros 7 dias, mas por regulação precisamos manter por 1 ano. Como otimizar?"

**Output esperado (resumo):**

- Diagnóstico: dados quentes nos primeiros 7 dias, depois essencialmente frios até o requisito de retenção de 1 ano
- Recomendação de comprimir os logs (gzip) e, se houver uso analítico, converter para Parquet particionado por data
- Esquema de particionamento por `ano/mês/dia` para reduzir volume escaneado em consultas pontuais
- Política de ciclo de vida: Standard nos primeiros 7 dias → Infrequent Access até 90 dias → Glacier/Deep Archive até completar 1 ano, respeitando o requisito de retenção antes de qualquer exclusão
- Trecho Terraform de `aws_s3_bucket_lifecycle_configuration` implementando as transições
- Nota de risco: custo de recuperação do Deep Archive deve ser avaliado caso auditorias exijam acesso a logs antigos com frequência
