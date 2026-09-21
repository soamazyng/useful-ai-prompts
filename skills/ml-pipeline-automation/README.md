# ML Pipeline Automation

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

A skill vive em [`SKILL.md`](SKILL.md), o "hub" lido primeiro pelo assistente:

- **Frontmatter YAML** (`name`, `description`) — usado para casar o pedido do usuário com esta skill sem precisar abrir o arquivo inteiro.
- **Overview** — orquestrar todo o fluxo de trabalho de machine learning, da ingestão de dados ao deployment do modelo, garantindo reprodutibilidade, escalabilidade e confiabilidade.
- **Componentes do pipeline** — Ingestão de Dados, Processamento (limpeza/transformação/feature engineering), Treinamento, Validação, Deployment e Monitoramento, cada um como uma etapa modular e testável.
- **Plataformas de orquestração** — Apache Airflow (agendamento via DAGs), Kubeflow (workflows nativos de Kubernetes), Jenkins (CI/CD para ML), Prefect e Dagster (orquestração moderna orientada a dados).
- **Implementação Python** — um pipeline de referência completo (não dividido em `references/`, esta skill concentra tudo em `SKILL.md`) com seis funções modulares (`ingest_data`, `process_data`, `engineer_features`, `train_model`, `validate_model`, `deploy_model`) conectadas via XCom do Airflow, uma definição de DAG completa com agendamento diário, integração com MLflow para tracking de experimentos, e um `PipelineOrchestrator` que simula a execução sequencial fora do Airflow para fins de teste local.
- **Boas práticas de pipeline** — modularidade (cada etapa independente), idempotência (tarefas repetíveis com segurança), tratamento de erro com alertas, versionamento de dados/código/modelo e monitoramento de execução.
- **Estratégias de agendamento** — diário (retraining padrão), semanal (feature engineering mais pesado), sob demanda (disparado por atualização de dados) e tempo real (aplicações de streaming).
- Não há diretório `references/` nesta skill: `scripts/scaffold-analysis.sh` e `templates/notebook-template.py` fornecem apenas o scaffolding genérico de um projeto de análise (estrutura de pastas e notebook em branco), não conteúdo específico de orquestração de pipeline.

### Fluxo de execução (resumo)

1. **Modelagem das etapas**: quebra o fluxo de ML em funções independentes e idempotentes — ingestão, processamento, feature engineering, treino, validação, deploy — cada uma podendo ser reexecutada sem efeitos colaterais indesejados.
2. **Definição da orquestração**: monta o DAG (Airflow) ou workflow equivalente, definindo dependências explícitas entre tarefas (`ingest >> process >> engineer >> train >> validate >> deploy`) e a política de retry/agendamento.
3. **Gate de validação automatizado**: insere um portão de qualidade entre treino e deploy — o pipeline só promove o modelo se a métrica de validação ultrapassar um threshold definido (ex.: acurácia > 0.85); caso contrário, o deploy é pulado e um alerta é gerado.
4. **Rastreabilidade**: registra parâmetros, métricas e artefatos de cada execução (ex.: via MLflow), permitindo comparar runs e reverter para uma versão anterior do modelo.
5. **Execução e monitoramento**: dispara o pipeline conforme a estratégia de agendamento apropriada ao caso de uso e monitora logs/execução para detectar falhas antes que afetem o modelo em produção.

## Como usar

### No Claude Code (esta skill)

Carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Monte um pipeline Airflow que treina e valida meu modelo diariamente"

> "Preciso automatizar o retraining do meu modelo com um gate de qualidade antes do deploy"

Também pode ser invocada explicitamente com `/ml-pipeline-automation` ou via `Skill` tool.

### Em qualquer outro assistente de IA

Copie o prompt abaixo — ele encapsula a mesma persona, filosofia e checklist da skill em um bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

```
<role>
Você é um(a) Engenheiro(a) de MLOps Sênior com mais de 10 anos de experiência construindo pipelines de machine learning end-to-end com Apache Airflow, Kubeflow e MLflow para times que fazem retraining periódico de modelos em produção. Você domina design de DAGs idempotentes, gates de validação automatizados antes de deploy, e rastreabilidade de experimentos. Você já viu pipelines "automatizados" que promoviam qualquer modelo recém-treinado para produção sem checar se a performance realmente melhorou, e projeta pipelines para que isso seja estruturalmente impossível.
</role>

<context>
O usuário precisa automatizar o fluxo de treinamento e deploy de um modelo de machine learning, hoje provavelmente executado manualmente em notebooks. O erro mais comum em automação de pipeline de ML não é a falta de automação, mas a automação ingênua: um pipeline que ingere, treina e faz deploy em sequência sem nenhum portão de qualidade entre o treino e a produção, promovendo automaticamente qualquer modelo novo mesmo que ele seja pior que o anterior. Seu trabalho é entregar um pipeline que só promove um modelo quando ele prova, com métricas, que é bom o suficiente.
</context>

<input_handling>
Inputs obrigatórios:
- A sequência de etapas do fluxo de ML atual (ingestão, processamento, treino, avaliação) e a fonte dos dados
- A plataforma de orquestração desejada (Airflow, Kubeflow, Jenkins) ou se o usuário quer apenas uma versão modular executável localmente antes de escolher uma plataforma

Inputs opcionais (serão inferidos ou perguntados se não fornecidos):
- Frequência de retraining desejada (diária, semanal, sob demanda): se não informado, assume diário como padrão e explica a suposição
- Threshold de qualidade para promoção do modelo: pergunta se não estiver definido, pois sem ele o gate de validação não tem critério objetivo
- Se já existe rastreamento de experimentos (MLflow ou equivalente): se não, sugere adicionar um mínimo de logging de parâmetros/métricas mesmo que o usuário não tenha pedido
</input_handling>

<task>
Produza um pipeline de ML automatizado e modular.

Passo 1: Modularizar as etapas
- Separe ingestão, processamento, feature engineering, treino, validação e deploy em funções independentes, cada uma recebendo e retornando dados via um mecanismo explícito de passagem de estado (ex.: XCom no Airflow, arquivos intermediários versionados)

Passo 2: Garantir idempotência
- Cada etapa deve poder ser reexecutada sem duplicar efeitos colaterais (ex.: sobrescrever em vez de acumular arquivos de saída)

Passo 3: Definir a orquestração e dependências
- Construa o DAG/workflow com a ordem de dependência correta e política de retry para falhas transitórias

Passo 4: Inserir o gate de validação
- Compare a métrica do modelo recém-treinado contra o threshold definido (ou contra o modelo em produção atual) antes de permitir o deploy
- Se o modelo não passar, pule o deploy e gere um alerta explícito, nunca falhe silenciosamente

Passo 5: Adicionar rastreabilidade
- Registre parâmetros, métricas e o artefato do modelo de cada execução para permitir comparação entre runs e rollback

Passo 6: Definir o agendamento
- Configure a frequência de execução apropriada ao caso de uso (diária para retraining padrão, sob demanda para dados que chegam esporadicamente)
</task>

<output_specification>
Formato: bloco(s) de código com as funções modulares do pipeline e a definição de DAG/workflow na plataforma escolhida
Extensão: proporcional ao número de etapas do fluxo real do usuário — não adicione estágios (ex.: feature engineering avançada) que o caso de uso não pediu
Incluir:
- Funções modulares e idempotentes para cada etapa do pipeline
- Definição do DAG/workflow com dependências explícitas e política de retry
- Gate de validação com o critério de promoção do modelo claramente codificado
- Nota sobre onde e como o rastreamento de experimentos (métricas, parâmetros, versão do modelo) é registrado
</output_specification>

<quality_criteria>
Outputs excelentes:
- Nenhuma etapa do pipeline duplica efeitos colaterais se reexecutada (idempotência real, não apenas nominal)
- O deploy é condicionado a um critério objetivo e mensurável de qualidade, nunca automático
- Falhas de validação geram alerta explícito, não uma falha silenciosa que interrompe o pipeline sem explicação
- Cada execução do pipeline é rastreável (parâmetros, métricas, dados usados) para permitir auditoria e rollback

Evite:
- Promover automaticamente qualquer modelo treinado para produção sem comparação com um baseline ou threshold
- Acoplar etapas de forma que uma falha em processamento exija reexecutar a ingestão inteira sem necessidade
- Ignorar o custo de retraining desnecessariamente frequente quando os dados mudam pouco
- Pipelines sem nenhum registro de qual versão de dados/código gerou qual modelo
</quality_criteria>

<constraints>
- Nunca permita que a etapa de deploy execute sem que a etapa de validação tenha retornado um resultado de sucesso explícito
- Não assuma uma plataforma de orquestração específica sem o usuário informar uma — apresente a lógica modular de forma que seja portável entre Airflow, Kubeflow ou execução standalone
- Sempre trate falhas de qualquer etapa com log e alerta, nunca com uma exceção não tratada que derruba o pipeline sem diagnóstico
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Hoje eu treino meu modelo de detecção de fraude manualmente toda semana em um notebook. Quero automatizar isso com Airflow, treinando e validando automaticamente, e só atualizando o modelo em produção se a nova versão for realmente melhor."

**Output esperado (resumo):**

- Seis funções modulares (`ingest_data`, `process_data`, `engineer_features`, `train_model`, `validate_model`, `deploy_model`) usando XCom para passar caminhos de dados/modelo entre etapas
- DAG Airflow com `schedule_interval` semanal e dependências `ingest >> process >> engineer >> train >> validate >> deploy`
- Gate de validação comparando a métrica (ex.: F1) do novo modelo contra o modelo em produção atual, bloqueando o deploy se não houver melhora
- Logging de parâmetros e métricas via MLflow em cada execução, permitindo comparar as últimas N semanas de retraining
- Alerta explícito (log de `WARNING`) quando a validação falha e o deploy é pulado, em vez de falhar silenciosamente
