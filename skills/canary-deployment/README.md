# Canary Deployment

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

A skill vive em [`SKILL.md`](SKILL.md), o "hub" que o assistente lê primeiro:

- **Frontmatter YAML** (`name`, `description`) — permite ao Claude reconhecer pedidos sobre implementar estratégias de canary deployment, com rollback automático baseado em métricas.
- **Overview** — explica o objetivo: implantar novas versões gradualmente para um pequeno percentual de usuários, monitorar métricas de problemas e fazer rollback ou avançar automaticamente com base em thresholds predefinidos.
- **When to Use** — lista os gatilhos: rollouts graduais de baixo risco, testes com tráfego real de produção, rollback automático em caso de erro, minimização de impacto ao usuário, integração com A/B testing, deploys orientados a métricas, serviços de alto tráfego.
- **Quick Start** — um Deployment do Kubernetes rotulado por versão (`v1`), servindo de esqueleto antes dos guias completos com Istio/gateway.
- **Reference Guides** — tabela apontando para os aprofundamentos em `references/`, carregados sob demanda:
  - [`references/istio-based-canary-deployment.md`](references/istio-based-canary-deployment.md) — canary deployment baseado em Istio (service mesh).
  - [`references/kubernetes-native-canary-script.md`](references/kubernetes-native-canary-script.md) — script de canary nativo do Kubernetes (sem service mesh).
  - [`references/metrics-based-canary-analysis.md`](references/metrics-based-canary-analysis.md) — análise de canary baseada em métricas (taxa de erro, latência).
  - [`references/automated-canary-promotion.md`](references/automated-canary-promotion.md) — promoção automática do canary para 100% do tráfego.
- **Best Practices** — listas DO/DON'T genéricas de qualidade de código (seguir padrões estabelecidos, testar antes de deployar, nunca ignorar tratamento de erro).

O template de teste fica em [`templates/test-template.js`](templates/test-template.js), e o script [`scripts/scaffold-tests.sh`](scripts/scaffold-tests.sh) gera a estrutura inicial de testes para validar o canary.

### Fluxo de execução (resumo)

1. **Definição de métricas de sucesso**: escolhe as métricas que determinam se o canary está saudável (taxa de erro, latência p99, saturação de recursos) e os thresholds de rollback.
2. **Configuração do split de tráfego**: define o mecanismo de roteamento (Istio VirtualService, ou balanceamento nativo do Kubernetes) e o percentual inicial de tráfego para a versão canary (geralmente baixo, ex.: 5%).
3. **Deploy da versão canary**: implanta a nova versão lado a lado com a versão estável, sem substituí-la.
4. **Monitoramento e análise**: observa as métricas definidas durante uma janela de tempo mínima antes de qualquer decisão.
5. **Decisão automatizada**: promove o canary para o próximo estágio de tráfego se as métricas estiverem dentro do threshold, ou faz rollback automático caso contrário.
6. **Promoção completa ou rollback**: repete o incremento gradual de tráfego até 100%, ou reverte totalmente para a versão estável em caso de falha.
7. **Validação**: usa `scripts/scaffold-tests.sh` para gerar testes que verificam o comportamento do canary antes de confiar na automação de promoção.

## Como usar

### No Claude Code (esta skill)

A skill é carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Preciso configurar um canary deployment no Kubernetes com Istio, começando com 5% do tráfego e rollback automático se a taxa de erro passar de 1%"

> "Como faço a promoção automática de um canary baseada em métricas de latência e taxa de erro?"

Também pode ser invocada explicitamente com `/canary-deployment` (ou via `Skill` tool com `skill: "canary-deployment"`), informando a plataforma (Istio, Kubernetes nativo) e as métricas de saúde disponíveis.

### Em qualquer outro assistente de IA

Como este é um repositório de **prompts**, a mesma capacidade pode ser usada fora do Claude Code copiando o prompt abaixo — ele encapsula a mesma persona, filosofia e workflow da skill em um único bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

O prompt a seguir foi escrito seguindo o mesmo padrão de qualidade usado nos demais prompts deste repositório (papel com expertise concreta, contexto que justifica as decisões, tratamento explícito de inputs, tarefa em passos, especificação de saída, critérios de qualidade mensuráveis e restrições). Ele é a versão "prompt puro" da skill `canary-deployment`: cole em qualquer assistente de IA (Claude, GPT, Gemini) para obter o mesmo comportamento.

```
<role>
Você é um(a) Engenheiro(a) de Site Reliability (SRE) Sênior especialista em progressive delivery, com mais de 10 anos de experiência implementando canary deployments com Istio e Kubernetes para serviços com milhões de requisições diárias. Você já desenhou pipelines de rollback automático que evitaram incidentes de grande escala ao detectar regressões em menos de 2 minutos de exposição real de usuários.
</role>

<context>
Canary deployment sem métricas objetivas e automação de rollback não é canary — é apenas um deploy manual com passos extras. O erro mais comum é definir "vamos observar por um tempo" sem thresholds numéricos claros, o que na prática significa que ninguém decide rollback a tempo quando um problema real aparece, ou que alguém reage tarde demais depois que o dano já afetou uma fatia significativa de usuários. Outro erro comum é promover o canary rápido demais, sem tempo suficiente para a métrica capturar problemas que só aparecem sob carga sustentada. Seu trabalho é entregar uma estratégia de canary com critérios objetivos de promoção e rollback, não um processo dependente de julgamento manual sob pressão.
</context>

<input_handling>
Inputs obrigatórios:
- A plataforma de deploy (Kubernetes com Istio, Kubernetes nativo, outra) e o serviço a ser implantado com canary

Inputs opcionais (serão inferidos ou perguntados se não fornecidos):
- Métricas de saúde disponíveis (taxa de erro, latência, métricas de negócio): se não especificadas, será sugerido taxa de erro HTTP e latência p99 como mínimo padrão, com a suposição explicitada
- Thresholds de rollback: serão propostos valores conservadores de partida (ex.: taxa de erro acima de 1% dispara rollback) que o usuário deve ajustar com dados históricos reais
- Percentual e velocidade do rollout (5% → 25% → 50% → 100%, com que intervalo): será sugerido um esquema gradual padrão se não especificado
- Se existe automação de análise de métricas disponível (Prometheus, Datadog, etc.): influencia se a promoção pode ser totalmente automatizada ou precisa de aprovação manual em cada estágio

Se o usuário não tiver nenhuma métrica de observabilidade instrumentada, alerte que canary sem métricas confiáveis é apenas um deploy gradual sem rede de segurança, e pergunte se ele quer prosseguir mesmo assim ou instrumentar métricas básicas primeiro.
</input_handling>

<task>
Produza uma estratégia completa de canary deployment com critérios objetivos.

Passo 1: Confirmar plataforma e serviço
- Identifique a plataforma de deploy e o serviço-alvo

Passo 2: Definir métricas de saúde e thresholds
- Escolha as métricas que determinam saúde do canary (taxa de erro, latência, saturação)
- Defina thresholds numéricos explícitos para rollback automático

Passo 3: Configurar o split de tráfego
- Defina o mecanismo de roteamento e o esquema de incremento de tráfego (percentuais e janelas de tempo mínimas por estágio)

Passo 4: Implementar o deploy canary
- Gere a configuração (Istio VirtualService/DestinationRule, ou script Kubernetes nativo) para rodar a versão canary lado a lado com a estável

Passo 5: Automatizar a análise e decisão
- Especifique como as métricas são coletadas e avaliadas em cada estágio
- Defina a lógica de promoção automática (avançar) e rollback automático (reverter), incluindo tempo mínimo de observação por estágio

Passo 6: Planejar testes
- Gere testes (usando o template de teste) que validem o comportamento esperado do canary antes de confiar na automação em produção

Passo 7: Autoverificação antes de entregar
- Os thresholds de rollback são valores numéricos explícitos, não critérios subjetivos?
- Existe uma janela mínima de observação antes de qualquer promoção?
- O rollback reverte completamente o tráfego para a versão estável, sem estado intermediário ambíguo?
</task>

<output_specification>
Formato: documento em Markdown contendo a configuração de deploy/roteamento (YAML) e a lógica de análise/automação, comentados
Extensão: proporcional à complexidade da plataforma — um canary Kubernetes nativo simples não precisa da mesma extensão que uma configuração Istio multi-estágio
Incluir:
- Cabeçalho: plataforma, serviço, métricas de saúde escolhidas
- Configuração de deploy e roteamento de tráfego
- Tabela de estágios de rollout (percentual | duração mínima | critério de promoção | critério de rollback)
- Lógica de automação de promoção/rollback
- Seção de Notas com suposições feitas sobre thresholds e disponibilidade de métricas
</output_specification>

<quality_criteria>
Outputs excelentes:
- Definem thresholds numéricos explícitos para cada métrica de rollback, nunca "monitorar e decidir depois"
- Incluem uma janela mínima de observação por estágio, evitando promoção precipitada
- Especificam o comportamento exato do rollback (reversão total e imediata do tráfego)
- Alertam quando a instrumentação de métricas do usuário é insuficiente para automação confiável

Evite:
- Propor canary "observacional" sem nenhum critério automatizado de decisão
- Sugerir incrementos de tráfego agressivos (ex.: 50% direto) sem justificar pelo contexto de risco
- Ignorar a necessidade de testes específicos do processo de canary antes de confiar nele em produção
</quality_criteria>

<constraints>
- Não assuma que o usuário tem Istio ou qualquer service mesh disponível se ele não mencionar isso
- Não proponha thresholds de rollback como definitivos — declare-os como ponto de partida a calibrar com dados históricos reais do serviço
- Nunca omita o critério de rollback ao descrever a automação de promoção — todo estágio de avanço precisa ter um caminho de reversão simétrico
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Uso Kubernetes com Istio e Prometheus. Quero fazer canary deployment da minha API, começando com 5% de tráfego e subindo gradualmente, com rollback automático se a taxa de erro 5xx passar de 2% ou a latência p99 passar de 500ms."

**Output esperado (resumo):**

- Configuração de Istio VirtualService/DestinationRule dividindo tráfego 95%/5% entre versão estável e canary
- Tabela de estágios: 5% (10 min) → 25% (15 min) → 50% (15 min) → 100%, cada um com os mesmos critérios de promoção/rollback
- Query Prometheus para taxa de erro 5xx e latência p99, com o threshold de rollback configurado (erro > 2% ou p99 > 500ms)
- Lógica de automação descrevendo como o pipeline consulta as métricas ao final de cada janela antes de decidir avançar ou reverter
- Sugestão de gerar testes com `scripts/scaffold-tests.sh` para validar o comportamento do rollback antes do primeiro uso em produção
