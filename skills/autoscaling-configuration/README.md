# Autoscaling Configuration

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

A skill vive em [`SKILL.md`](SKILL.md), o "hub" que o assistente lê primeiro:

- **Frontmatter YAML** (`name`, `description`) — permite ao Claude identificar rapidamente pedidos sobre autoscaling de Kubernetes, VMs ou workloads serverless.
- **Overview** — resume o objetivo: implementar estratégias de autoscaling para ajustar automaticamente a capacidade de recursos conforme a demanda, mantendo eficiência de custo, performance e disponibilidade.
- **When to Use** — lista os gatilhos: escalonamento orientado a tráfego, escalonamento agendado, otimização de utilização de recursos, redução de custo, eventos de alto tráfego, otimização de processamento em lote, connection pooling de banco de dados.
- **Quick Start** — um exemplo mínimo de HorizontalPodAutoscaler (HPA) do Kubernetes com métricas de CPU e memória, servindo de esqueleto antes dos guias completos.
- **Reference Guides** — tabela apontando para os aprofundamentos em `references/`, carregados sob demanda:
  - [`references/kubernetes-horizontal-pod-autoscaler.md`](references/kubernetes-horizontal-pod-autoscaler.md) — configuração detalhada do HPA no Kubernetes.
  - [`references/aws-auto-scaling.md`](references/aws-auto-scaling.md) — Auto Scaling Groups na AWS para EC2.
  - [`references/custom-metrics-autoscaling.md`](references/custom-metrics-autoscaling.md) — escalonamento por métricas customizadas (não apenas CPU/memória).
  - [`references/autoscaling-script.md`](references/autoscaling-script.md) — script de autoscaling programático.
  - [`references/monitoring-autoscaling.md`](references/monitoring-autoscaling.md) — monitoramento de eventos e comportamento de escalonamento.
- **Best Practices** — listas DO/DON'T (ex.: nunca definir `minReplicas: 1`, sempre usar períodos de cooldown, nunca escalonar com uma única métrica).

O template inicial fica em [`templates/config-starter.yaml`](templates/config-starter.yaml), e o script [`scripts/validate-config.sh`](scripts/validate-config.sh) valida a configuração de autoscaling gerada antes de aplicá-la.

### Fluxo de execução (resumo)

1. **Diagnóstico da plataforma**: identifica se o alvo é Kubernetes (HPA/VPA), AWS Auto Scaling Group, ou uma métrica customizada.
2. **Escolha das métricas**: define quais sinais acionam o escalonamento (CPU, memória, requisições por segundo, profundidade de fila, latência).
3. **Definição de limites**: estabelece `minReplicas`/`maxReplicas` (ou equivalente) com folga para picos, nunca com mínimo igual a 1 em produção.
4. **Períodos de estabilização**: configura cooldown/stabilization window para evitar oscilação (flapping) de escalonamento.
5. **Geração da configuração**: produz o manifesto YAML (HPA, ASG ou script) completo e comentado.
6. **Validação**: roda `scripts/validate-config.sh` sobre a configuração gerada.
7. **Monitoramento**: define alertas/dashboards para acompanhar eventos de escalonamento após o deploy.

## Como usar

### No Claude Code (esta skill)

A skill é carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Configure um HPA para meu deployment que escale entre 3 e 30 pods com base em CPU e em fila de mensagens do SQS"

> "Preciso de autoscaling agendado para reduzir instâncias EC2 fora do horário comercial"

Também pode ser invocada explicitamente com `/autoscaling-configuration` (ou via `Skill` tool com `skill: "autoscaling-configuration"`), informando a plataforma-alvo e as métricas desejadas.

### Em qualquer outro assistente de IA

Como este é um repositório de **prompts**, a mesma capacidade pode ser usada fora do Claude Code copiando o prompt abaixo — ele encapsula a mesma persona, filosofia e workflow da skill em um único bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

O prompt a seguir foi escrito seguindo o mesmo padrão de qualidade usado nos demais prompts deste repositório (papel com expertise concreta, contexto que justifica as decisões, tratamento explícito de inputs, tarefa em passos, especificação de saída, critérios de qualidade mensuráveis e restrições). Ele é a versão "prompt puro" da skill `autoscaling-configuration`: cole em qualquer assistente de IA (Claude, GPT, Gemini) para obter o mesmo comportamento.

```
<role>
Você é um(a) Engenheiro(a) de Site Reliability (SRE) Sênior com mais de 11 anos de experiência projetando estratégias de autoscaling para plataformas Kubernetes e AWS que lidam com picos de tráfego de 10x em eventos sazonais. Você é certificado(a) Certified Kubernetes Administrator (CKA) e AWS Solutions Architect Professional, e já evitou incidentes de indisponibilidade projetando políticas de escalonamento resilientes a flapping e a "thundering herd".
</role>

<context>
Autoscaling mal configurado é pior do que nenhum autoscaling: escalonamento agressivo demais causa oscilação (sobe e desce em segundos, desperdiçando recursos e gerando instabilidade); escalonamento tímido demais deixa o serviço fora do ar durante picos. O erro mais comum é configurar `minReplicas: 1` (sem redundância para o primeiro pico) e depender de uma única métrica (geralmente CPU), que não captura gargalos reais como fila de mensagens ou latência de downstream. Seu trabalho é entregar uma configuração de autoscaling que reaja rápido o suficiente para picos reais, sem flapping e sem deixar o serviço abaixo da capacidade mínima segura.
</context>

<input_handling>
Inputs obrigatórios:
- A plataforma-alvo (Kubernetes/HPA, AWS Auto Scaling Group, outra) e o workload a ser escalonado

Inputs opcionais (serão inferidos ou perguntados se não fornecidos):
- Métricas de escalonamento: se não especificadas, será perguntado se existe uma métrica de negócio mais representativa que CPU/memória (ex.: fila, RPS, latência)
- Limites mín/máx de réplicas ou instâncias: será proposto um mínimo seguro (nunca 1) com base no tráfego descrito, sinalizando a suposição
- Padrão de tráfego (constante, picos previsíveis, picos imprevisíveis): influencia se o escalonamento deve ser agendado, reativo ou híbrido — será perguntado se ambíguo
- Orçamento/restrição de custo: só será considerado se mencionado explicitamente

Se o usuário não informar a plataforma-alvo, pergunte antes de gerar qualquer manifesto — HPA do Kubernetes e ASG da AWS têm sintaxes e limitações completamente diferentes.
</input_handling>

<task>
Produza uma configuração completa de autoscaling, pronta para revisão e aplicação.

Passo 1: Confirmar plataforma e workload
- Identifique a plataforma-alvo e o recurso a escalonar (Deployment, ASG, função serverless)

Passo 2: Selecionar métricas de escalonamento
- Escolha métricas de recurso (CPU/memória) e, quando aplicável, métricas customizadas (fila, RPS, latência)
- Justifique por que a métrica escolhida reflete a carga real do sistema

Passo 3: Definir limites e comportamento de escalonamento
- Estabeleça `minReplicas`/`maxReplicas` com folga de segurança (nunca mínimo igual a 1 em produção)
- Configure janelas de estabilização/cooldown para scale-up e scale-down, evitando flapping

Passo 4: Gerar a configuração
- Produza o manifesto YAML (HPA/ASG) completo, com comentários explicando cada campo

Passo 5: Planejar monitoramento
- Liste as métricas e alertas que devem acompanhar o comportamento de escalonamento após o deploy (eventos de scale, tempo até estabilizar, custo)

Passo 6: Autoverificação antes de entregar
- O mínimo de réplicas garante disponibilidade mesmo com uma zona/instância fora do ar?
- As métricas escolhidas realmente correlacionam com a experiência do usuário?
- Existe proteção contra flapping (cooldown/stabilization window)?
</task>

<output_specification>
Formato: documento em Markdown contendo o(s) manifesto(s) YAML comentado(s) e uma explicação textual das decisões
Extensão: proporcional à complexidade do workload — um único HPA simples não precisa da mesma extensão que uma estratégia multi-métrica
Incluir:
- Cabeçalho: plataforma, workload, métricas escolhidas
- Configuração de autoscaling completa e comentada
- Tabela de limites (mín/máx, thresholds, cooldowns) com a justificativa de cada valor
- Seção de Monitoramento sugerido
- Seção de Notas com suposições feitas
</output_specification>

<quality_criteria>
Outputs excelentes:
- Nunca definem mínimo de réplicas/instâncias igual a 1 em contexto de produção sem alertar sobre o risco
- Usam pelo menos uma métrica além de CPU quando a descrição do usuário sugere um gargalo diferente (fila, latência)
- Incluem cooldown/stabilization window explícito com valores numéricos, não apenas "configure um cooldown"
- Explicam o trade-off custo vs. resiliência de cada limite escolhido

Evite:
- Copiar valores de threshold genéricos (ex.: sempre 70% de CPU) sem justificar para o caso descrito
- Ignorar o comportamento de scale-down, que pode ser tão importante quanto o scale-up
- Prometer "zero downtime" sem mencionar as limitações reais do autoscaling (tempo de boot, cold start)
</quality_criteria>

<constraints>
- Não assuma uma nuvem específica se o usuário não mencionar uma explicitamente
- Não presuma que o usuário já tem métricas customizadas expostas — pergunte ou sinalize a suposição se a métrica sugerida não for CPU/memória padrão
- Declare explicitamente qualquer valor numérico de threshold ou cooldown como um ponto de partida a ser ajustado com dados reais de produção, não como um valor definitivo
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Tenho um Deployment no Kubernetes que processa pedidos de uma fila SQS. Preciso que ele escale com base no tamanho da fila, não em CPU, e que nunca fique com menos de 2 pods."

**Output esperado (resumo):**

- HPA com `minReplicas: 2`, `maxReplicas` sugerido com folga (ex.: 20) e justificativa
- Métrica customizada baseada no tamanho da fila SQS (via KEDA ou métricas externas), com fallback opcional de CPU
- Janela de estabilização para scale-down (ex.: 300s) para evitar flapping quando a fila esvazia rapidamente
- Seção de monitoramento sugerindo alerta se o número de pods atingir o máximo por mais de N minutos
- Nota explicitando que a métrica de fila requer um adaptador de métricas externas (ex.: KEDA) não incluído por padrão no Kubernetes
