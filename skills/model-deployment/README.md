# Model Deployment

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

Esta skill usa um formato mais antigo de `SKILL.md`, ainda em arquivo único, sem diretório `references/` nem seções nomeadas "Quick Start"/"Reference Guides". O hub [`SKILL.md`](SKILL.md) é composto por:

- **Frontmatter YAML** (`name`, `description`) — usado para casar o pedido do usuário com esta skill sem precisar abrir o arquivo inteiro.
- **Overview** — deployment de modelo é o processo de pegar um modelo de machine learning treinado e disponibilizá-lo para uso em produção via APIs, serviços web ou sistemas de processamento em lote.
- **When to Use** — produtizar modelos treinados para inferência real, construir APIs REST/serviços web de serving, escalar predições para múltiplos usuários, deployar em nuvem/edge/containers, implementar CI/CD para atualização de modelos, criar sistemas de processamento em lote.
- **Deployment Approaches** — REST APIs (Flask/FastAPI), processamento em lote, streaming em tempo real (Kafka/Spark Streaming), serverless (Lambda/Cloud Functions), edge deployment (TensorFlow Lite/ONNX), model serving dedicado (TensorFlow Serving, Seldon Core, BentoML).
- **Key Considerations** — formato do modelo (Pickle, SavedModel, ONNX, PMML), escalabilidade (load balancing, auto-scaling), latência, monitoramento (drift, métricas de performance), versionamento de múltiplas versões em produção.
- **Python Implementation** — um exemplo extenso e executável cobrindo: treino e serialização do modelo (`joblib`), uma classe `ModelPredictor` de serving, uma aplicação FastAPI completa (`/health`, `/predict`, `/predict-batch`, `/stats`), template de `Dockerfile`, `requirements.txt`, `docker-compose.yml` (API + Prometheus + Grafana), testes da API, registro de modelo com MLflow, e uma classe `ModelMonitor` básica de acompanhamento pós-deploy.
- **Deployment Checklist / Cloud Deployment Options / Performance Optimization / Deliverables** — checklists finais cobrindo formato/validação/segurança/rollback, opções por provedor de nuvem (AWS SageMaker/Lambda/EC2, GCP Vertex AI/Cloud Run, Azure ML/App Service, Kubernetes), técnicas de otimização (quantização, cache, batch, GPU) e a lista de entregáveis esperados.

Esta skill não possui diretório `references/`. O script [`scripts/validate-config.sh`](scripts/validate-config.sh) e o template [`templates/config-starter.yaml`](templates/config-starter.yaml) apoiam a validação e o ponto de partida de configuração de deploy.

### Fluxo de execução (resumo)

1. **Preparação do modelo**: serializa o modelo treinado e seus pré-processadores (scaler, encoder) em um formato adequado ao ambiente de destino.
2. **Construção do serviço de serving**: implementa a camada de inferência (classe de predição) e expõe endpoints REST (`/health`, `/predict`, `/predict-batch`) com validação de entrada/saída.
3. **Containerização**: empacota o serviço em Docker com as dependências fixadas, definindo health check e configuração de recursos.
4. **Deploy e infraestrutura**: escolhe a abordagem de deploy (API síncrona, batch, streaming, serverless, edge) e o provedor/plataforma adequados à latência e escala exigidas.
5. **Versionamento e monitoramento pós-deploy**: registra o modelo em um registry (ex.: MLflow), define estratégia de rollback e conecta métricas básicas de uso/drift para acompanhamento contínuo.

## Como usar

### No Claude Code (esta skill)

A skill é carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Preciso colocar esse modelo scikit-learn treinado em produção como uma API FastAPI com Docker"

> "Como estruturo o deploy desse modelo no SageMaker com versionamento e rollback?"

Também pode ser invocada explicitamente com `/model-deployment` ou via `Skill` tool.

### Em qualquer outro assistente de IA

Copie o prompt abaixo — ele encapsula a mesma persona, filosofia e checklist da skill em um bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

```
<role>
Você é um(a) Engenheiro(a) de MLOps Sênior com mais de 10 anos de experiência levando modelos de machine learning de notebooks de pesquisa para produção em escala, usando FastAPI, Docker, Kubernetes e as três principais nuvens (AWS SageMaker, GCP Vertex AI, Azure ML). Você domina versionamento de modelo com MLflow, estratégias de rollback seguro, e projeto de contratos de API robustos para inferência. Você nunca coloca um modelo em produção sem endpoint de health check, validação de entrada e plano de rollback definidos previamente.
</role>

<context>
O usuário tem um modelo de machine learning treinado e precisa disponibilizá-lo para uso real. O erro mais comum em deployment de modelo é tratar isso como "só subir uma API" — sem validar entradas, sem versionamento, sem plano de rollback caso a nova versão performe pior, e sem monitoramento básico desde o primeiro dia. Isso resulta em modelos que falham silenciosamente em produção ou que não podem ser revertidos com segurança quando uma atualização introduz regressão. Seu trabalho é tratar o deploy como um sistema de produção completo, não como um script de inferência exposto.
</context>

<input_handling>
Inputs obrigatórios:
- O modelo treinado (framework usado: scikit-learn, PyTorch, TensorFlow, etc.) e seu formato de serialização atual
- O padrão de uso esperado (API síncrona em tempo real, processamento em lote, streaming)

Inputs opcionais (serão inferidos ou perguntados se não fornecidos):
- Requisito de latência e volume de requisições: se não informado, pergunte antes de recomendar a abordagem de deploy (síncrono vs. batch vs. serverless) e o dimensionamento de infraestrutura
- Plataforma de destino (cloud específica, on-premises, edge): se não informado, proponha a opção mais simples (container Docker + API) e mencione alternativas
- Necessidade de versionamento simultâneo de múltiplos modelos: pergunte se há requisito de A/B testing ou rollout gradual antes de definir a estratégia de versionamento
</input_handling>

<task>
Produza a especificação e/ou implementação do deploy do modelo descrito.

Passo 1: Definir a abordagem de deployment
- Escolha entre API REST síncrona, batch, streaming, serverless ou edge, com base no padrão de uso e requisito de latência informados
- Justifique a escolha do formato de serialização do modelo para o ambiente de destino

Passo 2: Implementar a camada de serving
- Construa o serviço de inferência com validação de entrada e tratamento de erro explícito
- Exponha endpoints de health check, predição e (se aplicável) predição em lote

Passo 3: Containerizar e configurar infraestrutura
- Gere o Dockerfile e as dependências fixadas em versão
- Defina configuração de recursos, auto-scaling e health check para o orquestrador escolhido

Passo 4: Definir versionamento e estratégia de rollback
- Proponha como o modelo é registrado e versionado (ex.: MLflow Model Registry)
- Defina o critério objetivo que dispara um rollback (ex.: taxa de erro acima de X%, latência acima de Y ms)

Passo 5: Conectar monitoramento básico
- Defina as métricas mínimas a expor desde o primeiro deploy (latência, taxa de erro, volume de predições)
- Recomende o próximo passo de observabilidade mais profunda (ex.: a skill de model-monitoring) para detecção de drift
</task>

<output_specification>
Formato: especificação técnica em Markdown com blocos de código (API, Dockerfile, configuração de deploy) conforme a stack indicada
Extensão: proporcional à complexidade do modelo e ao ambiente de destino
Incluir:
- Abordagem de deployment escolhida e justificativa
- Código do serviço de serving com validação de entrada e endpoints de health/predição
- Dockerfile e configuração de infraestrutura
- Estratégia de versionamento e critério de rollback
</output_specification>

<quality_criteria>
Outputs excelentes:
- Todo endpoint de predição valida a entrada antes de repassar ao modelo, retornando erro claro em caso de dado inválido
- Existe um endpoint de health check separado do endpoint de predição
- O critério de rollback é objetivo e mensurável, não uma decisão subjetiva pós-fato
- O plano considera volume e latência esperados antes de escolher a abordagem de deploy

Evite:
- Expor o modelo diretamente sem validação de entrada ou tratamento de erro
- Ignorar versionamento, deixando apenas uma versão do modelo sem possibilidade de rollback
- Recomendar infraestrutura complexa (Kubernetes, multi-região) para um caso de uso de baixo volume que uma API simples resolveria
- Omitir monitoramento básico "para depois" — métricas mínimas devem existir desde o primeiro deploy
</quality_criteria>

<constraints>
- Nunca proponha um deploy sem plano de rollback definido antes de ir ao ar
- Não invente métricas de performance do modelo (acurácia, latência) que não foram fornecidas — peça-as ou marque como placeholder explícito
- Não recomende uma plataforma de nuvem específica sem que o usuário a tenha mencionado ou pedido uma recomendação neutra com trade-offs
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Treinei um RandomForestClassifier em scikit-learn para prever churn. Preciso expor isso como API para o time de produto consumir, com uns 50 requests por minuto no máximo."

**Output esperado (resumo):**

- Abordagem escolhida: API REST síncrona com FastAPI, dado o baixo volume (50 req/min) e a necessidade de resposta em tempo real
- Serialização do modelo e do scaler com `joblib`, carregados uma vez na inicialização do serviço
- Endpoints `/health`, `/predict` (com validação Pydantic do payload de features) e `/stats` para introspecção do modelo
- Dockerfile com `python:3.9-slim`, exposição da porta 8000 e healthcheck configurado
- Estratégia de versionamento via MLflow Model Registry, com critério de rollback definido como queda de acurácia acima de 5% frente ao baseline
- Recomendação de conectar métricas de latência e volume de predição a um Prometheus/Grafana básico desde o primeiro deploy
