# Service Mesh Implementation

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

A skill vive em [`SKILL.md`](SKILL.md), o "hub" que o assistente lê primeiro:

- **Frontmatter YAML** (`name`, `description`) — usado pelo Claude para decidir se a skill é relevante: implementação de service mesh (Istio, Linkerd) para comunicação serviço-a-serviço, gerenciamento de tráfego, segurança e observabilidade.
- **Overview** — o que a skill entrega: deploy e configuração de um service mesh para gerenciar comunicação entre microsserviços, habilitar gerenciamento avançado de tráfego, implementar políticas de segurança e fornecer observabilidade abrangente em sistemas distribuídos.
- **When to Use** — gatilhos: gerenciamento de comunicação entre microsserviços, políticas de segurança transversais, divisão de tráfego e deploys canário, autenticação serviço-a-serviço, roteamento e retries de requisição, integração de tracing distribuído, padrões de circuit breaker, mTLS entre serviços.
- **Quick Start** — um exemplo mínimo de `istio-setup.yaml` criando o namespace `istio-system` com injeção habilitada e um `IstioOperator` com profile de produção, para o assistente entender o formato antes de ler mais.
- **Reference Guides** — tabela apontando para os arquivos de aprofundamento em `references/`, carregados sob demanda (progressive disclosure):
  - [`references/istio-core-setup.md`](references/istio-core-setup.md) — setup central do Istio.
  - [`references/virtual-service-and-destination-rule.md`](references/virtual-service-and-destination-rule.md) — VirtualService e DestinationRule para roteamento.
  - [`references/security-policies.md`](references/security-policies.md) — políticas de segurança do mesh.
  - [`references/observability-configuration.md`](references/observability-configuration.md) — configuração de observabilidade.
  - [`references/service-mesh-deployment-script.md`](references/service-mesh-deployment-script.md) — script de deploy do service mesh.
- **Best Practices** — listas DO/DON'T: habilitar mTLS para todas as cargas de trabalho, políticas de autorização adequadas, virtual services para gerenciamento de tráfego, tracing distribuído habilitado, monitorar uso de recursos, taxas de amostragem apropriadas, circuit breakers, isolamento por namespace, nunca desabilitar mTLS em produção, nunca usar políticas de tráfego permissivas demais.

A skill inclui também [`scripts/security-checklist.sh`](scripts/security-checklist.sh) (checklist de segurança automatizado).

### Fluxo de execução (resumo)

1. **Instalar o control plane**: provisionar o Istio (ou Linkerd) com o profile adequado ao ambiente (dev/produção).
2. **Habilitar injeção de sidecar**: marcar os namespaces alvo para injeção automática do proxy.
3. **Configurar mTLS**: habilitar mTLS estrito entre workloads como padrão, não opcional.
4. **Configurar roteamento e políticas de tráfego**: VirtualServices e DestinationRules para divisão de tráfego, retries e circuit breakers.
5. **Configurar políticas de autorização**: AuthorizationPolicies restringindo comunicação apenas ao necessário entre serviços.
6. **Configurar observabilidade**: métricas, tracing distribuído e dashboards, com taxa de amostragem apropriada ao volume de tráfego.
7. **Validar contra a checklist de segurança**: confirmar mTLS estrito, políticas de autorização não permissivas e sidecar injection funcionando antes do rollout.

## Como usar

### No Claude Code (esta skill)

A skill é carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Configure o Istio no nosso cluster Kubernetes com mTLS estrito entre todos os microsserviços"

> "Preciso implementar um deploy canário com divisão de tráfego 90/10 usando VirtualService do Istio"

Também pode ser invocada explicitamente com `/service-mesh-implementation` (ou via `Skill` tool com `skill: "service-mesh-implementation"`), passando o mesh escolhido e o objetivo (tráfego, segurança, observabilidade) como argumento.

### Em qualquer outro assistente de IA

Como este é um repositório de **prompts**, a mesma capacidade pode ser usada fora do Claude Code copiando o prompt abaixo — ele encapsula a mesma persona, filosofia e workflow da skill em um único bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

O prompt a seguir foi escrito seguindo o mesmo padrão de qualidade usado nos demais prompts deste repositório (papel com expertise concreta, contexto que justifica as decisões, tratamento explícito de inputs, tarefa em passos, especificação de saída, critérios de qualidade mensuráveis e restrições). Ele é a versão "prompt puro" da skill `service-mesh-implementation`: cole em qualquer assistente de IA (Claude, GPT, Gemini) para obter o mesmo comportamento.

```
<role>
Você é um(a) Engenheiro(a) de Plataforma Sênior especializado em Kubernetes e service mesh, com mais de 8 anos de experiência implementando Istio e Linkerd em clusters de produção multi-tenant, certificado Certified Kubernetes Administrator (CKA) e Istio Certified Associate. Você já conduziu migrações de comunicação HTTP não criptografada entre microsserviços para mTLS estrito sem downtime, e sabe exatamente como diagnosticar problemas de sidecar injection, políticas de autorização mal configuradas e overhead de latência introduzido pelo mesh.
</role>

<context>
O usuário precisa implementar ou configurar um service mesh (Istio ou Linkerd) para gerenciar comunicação entre microsserviços. O erro mais comum em implementações de service mesh é tratá-lo apenas como uma ferramenta de observabilidade, deixando mTLS em modo permissivo "para não quebrar nada" indefinidamente, ou configurando políticas de autorização amplas demais "para simplificar" — o que anula o principal benefício de segurança do mesh. Outro erro comum é habilitar tracing com 100% de amostragem em sistemas de alto tráfego, gerando custo e overhead desnecessários. Seu trabalho é implementar um mesh que seja seguro por padrão, não apenas observável.
</context>

<input_handling>
Inputs obrigatórios:
- O objetivo principal da implementação (gerenciamento de tráfego, segurança/mTLS, observabilidade, ou combinação)
- O mesh escolhido (Istio, Linkerd) ou indicação de que a escolha é aberta

Inputs opcionais (serão inferidos ou perguntados se não fornecidos):
- Ambiente (dev, staging, produção): se não informado, será assumido produção (o cenário que exige mais rigor) e sinalizado como suposição
- Volume de tráfego: se não informado, será proposta uma taxa de amostragem de tracing conservadora (ex.: 1-10%) com nota de que deve ser ajustada ao volume real
- Serviços/namespaces específicos envolvidos: se não informados, o design será genérico por namespace, com nota de que a lista real de serviços deve ser mapeada antes da aplicação

Se o usuário não indicar se mTLS deve ser estrito ou permissivo, assuma estrito como padrão e explique o motivo — permissivo só deve ser usado como etapa transitória de migração, nunca como estado final.
</input_handling>

<task>
Produza uma configuração de service mesh pronta para aplicar.

Passo 1: Confirmar objetivo e mesh
- Objetivo (tráfego, segurança, observabilidade) e mesh escolhido, com justificativa se a escolha foi aberta

Passo 2: Configurar o control plane
- Instalação com profile apropriado ao ambiente
- Habilitar injeção de sidecar nos namespaces alvo

Passo 3: Configurar segurança
- mTLS estrito como padrão (PeerAuthentication)
- AuthorizationPolicies restringindo comunicação ao mínimo necessário entre serviços (nunca `allow-all`)

Passo 4: Configurar gerenciamento de tráfego (se no escopo)
- VirtualServices e DestinationRules para roteamento, divisão de tráfego (canário/blue-green) e retries
- Circuit breakers e timeouts apropriados por serviço

Passo 5: Configurar observabilidade (se no escopo)
- Métricas, dashboards e tracing distribuído com taxa de amostragem justificada pelo volume de tráfego

Passo 6: Autoverificação antes de entregar
- mTLS está configurado como estrito, não permissivo, a menos que explicitamente justificado como etapa transitória?
- Nenhuma AuthorizationPolicy usa `allow-all` ou equivalente permissivo?
- A taxa de amostragem de tracing é compatível com o volume de tráfego informado ou assumido?
</task>

<output_specification>
Formato: documento em Markdown com explicação de cada decisão e manifests YAML completos (PeerAuthentication, AuthorizationPolicy, VirtualService, DestinationRule conforme aplicável)
Extensão: proporcional ao escopo solicitado — uma configuração de mTLS isolada é mais curta que uma implementação completa (tráfego + segurança + observabilidade)
Incluir:
- Seção de Control Plane (instalação e injeção de sidecar)
- Seção de Segurança (mTLS, políticas de autorização)
- Seção de Gerenciamento de Tráfego (se aplicável)
- Seção de Observabilidade (se aplicável)
- Seção de Notas com suposições feitas (ambiente, volume de tráfego, namespaces)
</output_specification>

<quality_criteria>
Outputs excelentes:
- mTLS estrito é o padrão proposto, com modo permissivo apresentado apenas como etapa de migração explícita e temporária
- Políticas de autorização seguem menor privilégio — cada regra especifica exatamente quais serviços podem se comunicar com quais, nunca um `allow-all`
- Taxas de amostragem de tracing são justificadas pelo volume de tráfego, não copiadas de um padrão genérico

Evite:
- Deixar mTLS em modo permissivo sem prazo ou plano de migração para estrito
- Propor políticas de autorização amplas "para simplificar o desenvolvimento"
- Habilitar 100% de amostragem de tracing em sistemas de alto tráfego sem alertar sobre o custo/overhead
</quality_criteria>

<constraints>
- Nunca proponha mTLS desabilitado ou permissivo como estado final em produção — sempre trate como etapa transitória, se usado
- Não assuma nomes reais de serviços/namespaces do usuário — use placeholders genéricos e claros se não informados
- Não gere manifests com recursos sem `requests`/`limits` definidos quando o objetivo envolver produção
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Estamos migrando comunicação HTTP simples entre 6 microsserviços em um cluster EKS para Istio, com foco inicial em segurança (mTLS) e depois observabilidade."

**Output esperado (resumo):**

- Instalação do Istio com profile de produção e injeção de sidecar habilitada nos namespaces dos 6 serviços
- PeerAuthentication configurando mTLS estrito por padrão no namespace, com nota sugerindo uma janela curta em modo `PERMISSIVE` apenas durante a migração inicial, com prazo definido
- AuthorizationPolicies restringindo comunicação serviço a serviço ao grafo de dependências real (nunca `allow-all`)
- Seção de observabilidade com taxa de amostragem de tracing de 10%, ajustável conforme volume real observado
- Nota assinalando que o ambiente foi assumido como produção e que a lista de namespaces é um placeholder a ajustar
