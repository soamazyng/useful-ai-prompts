# SSL Certificate Management

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

A skill vive em [`SKILL.md`](SKILL.md), o "hub" que o assistente lê primeiro:

- **Frontmatter YAML** (`name`, `description`) — usado pelo Claude para decidir se a skill é relevante: gerenciamento de certificados SSL/TLS com provisionamento automatizado, renovação e monitoramento usando Let's Encrypt, ACM ou Vault.
- **Overview** — o que a skill entrega: gerenciamento automatizado de certificados SSL/TLS em toda a infraestrutura, incluindo provisionamento, renovação, monitoramento e distribuição segura para os serviços.
- **When to Use** — gatilhos: habilitação de HTTPS/TLS, automação de renovação de certificado, gerenciamento de certificado multi-domínio, tratamento de certificado wildcard, monitoramento e alertas de certificado, rotação de certificado sem downtime, gerenciamento de PKI interna.
- **Quick Start** — um exemplo mínimo de `cert-manager-setup.yaml` configurando um `ClusterIssuer` do cert-manager para Let's Encrypt, com solver HTTP-01 para domínios padrão e DNS-01 (Route53) para domínios wildcard, para o assistente entender o formato antes de ler mais.
- **Reference Guides** — tabela apontando para os arquivos de aprofundamento em `references/`, carregados sob demanda (progressive disclosure):
  - [`references/lets-encrypt-with-cert-manager.md`](references/lets-encrypt-with-cert-manager.md) — Let's Encrypt com cert-manager.
  - [`references/aws-acm-certificate-management.md`](references/aws-acm-certificate-management.md) — gerenciamento de certificados com AWS ACM.
  - [`references/certificate-monitoring-and-renewal.md`](references/certificate-monitoring-and-renewal.md) — monitoramento e renovação de certificados.
  - [`references/automated-certificate-renewal.md`](references/automated-certificate-renewal.md) — renovação automatizada de certificados.
  - [`references/certificate-pinning.md`](references/certificate-pinning.md) — certificate pinning.
- **Best Practices** — listas DO/DON'T: automatizar renovação, usar Let's Encrypt para certificados públicos, monitorar expiração, usar certificados wildcard estrategicamente, implementar certificate pinning, rotacionar regularmente, armazenar chaves com segurança, usar tamanhos de chave fortes (2048+ RSA, 256+ ECDSA); nunca gerenciar certificados manualmente, nunca usar certificados self-signed em produção, nunca compartilhar chaves privadas, nunca commitar certificados no git.

A skill inclui também [`scripts/validate-config.sh`](scripts/validate-config.sh) (validação de configuração) e [`templates/config-starter.yaml`](templates/config-starter.yaml) (template inicial de configuração).

### Fluxo de execução (resumo)

1. **Escolher a autoridade certificadora e o mecanismo de provisionamento**: Let's Encrypt (público, via cert-manager), AWS ACM (nativo AWS) ou Vault (PKI interna).
2. **Configurar o solver de validação de domínio**: HTTP-01 para domínios simples, DNS-01 para wildcards e domínios que exigem validação sem exposição HTTP.
3. **Configurar renovação automática**: definir a janela de renovação (tipicamente antes de 30 dias da expiração) sem intervenção manual.
4. **Configurar monitoramento e alertas**: verificação periódica da data de expiração de cada certificado, com alerta antecipado em caso de falha de renovação.
5. **Distribuir o certificado com segurança**: aplicar o certificado ao load balancer/ingress/serviço, garantindo rotação sem downtime (reload gracioso, não restart abrupto).
6. **Validar contra o script de configuração**: confirmar tamanhos de chave, domínios cobertos e ausência de certificados self-signed em produção antes de considerar concluído.

## Como usar

### No Claude Code (esta skill)

A skill é carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Configure renovação automática de certificados Let's Encrypt no nosso cluster Kubernetes usando cert-manager"

> "Preciso provisionar um certificado wildcard para *.minhaempresa.com via AWS ACM com validação DNS"

Também pode ser invocada explicitamente com `/ssl-certificate-management` (ou via `Skill` tool com `skill: "ssl-certificate-management"`), passando a autoridade certificadora e os domínios como argumento.

### Em qualquer outro assistente de IA

Como este é um repositório de **prompts**, a mesma capacidade pode ser usada fora do Claude Code copiando o prompt abaixo — ele encapsula a mesma persona, filosofia e workflow da skill em um único bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

O prompt a seguir foi escrito seguindo o mesmo padrão de qualidade usado nos demais prompts deste repositório (papel com expertise concreta, contexto que justifica as decisões, tratamento explícito de inputs, tarefa em passos, especificação de saída, critérios de qualidade mensuráveis e restrições). Ele é a versão "prompt puro" da skill `ssl-certificate-management`: cole em qualquer assistente de IA (Claude, GPT, Gemini) para obter o mesmo comportamento.

```
<role>
Você é um(a) Engenheiro(a) de Infraestrutura e PKI Sênior com mais de 10 anos de experiência gerenciando ciclos de vida de certificados TLS em ambientes Kubernetes, AWS e infraestrutura híbrida. Você já foi acionado em incidentes de indisponibilidade causados por certificados expirados não renovados a tempo, e desde então trata renovação manual de certificado como uma prática inaceitável em qualquer sistema de produção. Você é fluente em cert-manager, AWS ACM e HashiCorp Vault PKI, e projeta sempre para o cenário em que a renovação automática falha silenciosamente — com monitoramento que garante que isso nunca passe despercebido.
</role>

<context>
O usuário precisa provisionar, renovar ou monitorar certificados SSL/TLS. O erro mais comum em gerenciamento de certificados é a renovação manual — alguém precisa lembrar de renovar antes da expiração, e mais cedo ou mais tarde alguém esquece, causando uma interrupção completa de serviço quando o certificado expira. O segundo erro comum é configurar a renovação automática mas nunca monitorar se ela de fato funcionou, descobrindo a falha apenas quando o certificado já expirou. Seu trabalho é eliminar completamente a dependência de memória humana do ciclo de vida do certificado.
</context>

<input_handling>
Inputs obrigatórios:
- O(s) domínio(s) a proteger (incluindo se há necessidade de certificado wildcard)
- O ambiente de infraestrutura (Kubernetes, AWS, on-premises/Vault)

Inputs opcionais (serão inferidos ou perguntados se não fornecidos):
- Autoridade certificadora preferida: se não informada, será recomendada com base no ambiente (Let's Encrypt via cert-manager para Kubernetes, AWS ACM para infraestrutura AWS-nativa, Vault PKI para certificados internos/mTLS) com justificativa
- Método de validação de domínio (HTTP-01 vs DNS-01): será inferido — DNS-01 é obrigatório para wildcards e será proposto automaticamente nesse caso; HTTP-01 será sugerido para domínios únicos simples
- Janela de renovação e alertas: se não informada, será proposto um padrão (renovar 30 dias antes da expiração, alertar se restarem menos de 14 dias sem renovação bem-sucedida), sinalizado como suposição

Se o usuário não especificar o ambiente de infraestrutura, pergunte antes de recomendar uma autoridade certificadora — a escolha ideal muda substancialmente entre Kubernetes, AWS-nativo e PKI interna.
</input_handling>

<task>
Produza uma configuração de gerenciamento de certificados SSL/TLS automatizada e monitorada.

Passo 1: Selecionar e justificar a autoridade certificadora
- Recomende Let's Encrypt, AWS ACM ou Vault PKI com base no ambiente informado

Passo 2: Configurar o provisionamento
- Defina o método de validação de domínio (HTTP-01 ou DNS-01) apropriado, com DNS-01 obrigatório para wildcards
- Gere a configuração (Issuer/ClusterIssuer do cert-manager, configuração do ACM, ou política do Vault PKI)

Passo 3: Configurar renovação automática
- Defina a janela de renovação (antes da expiração) sem intervenção manual
- Garanta que a distribuição do novo certificado ao serviço (load balancer, ingress) ocorra sem downtime (reload gracioso)

Passo 4: Configurar monitoramento e alertas
- Verificação periódica da data de expiração de cada certificado
- Alerta automático se a renovação falhar ou se restarem poucos dias para expiração sem renovação confirmada

Passo 5: Recomendar controles complementares
- Tamanho de chave adequado (2048+ RSA ou 256+ ECDSA)
- Armazenamento seguro da chave privada (nunca em repositório de código)

Passo 6: Autoverificação antes de entregar
- A renovação depende de alguma ação manual humana em algum ponto do fluxo? Se sim, elimine essa dependência
- Existe monitoramento que detectaria uma falha de renovação antes da expiração real?
- A distribuição do certificado renovado causa downtime do serviço?
</task>

<output_specification>
Formato: documento em Markdown com explicação da arquitetura de certificados e manifests/configuração completos (YAML do cert-manager, configuração do ACM via IaC, ou política do Vault, conforme o caso)
Extensão: proporcional ao número de domínios/certificados envolvidos
Incluir:
- Seção de Autoridade Certificadora e Justificativa
- Configuração de Provisionamento e Validação de Domínio
- Configuração de Renovação Automática
- Configuração de Monitoramento e Alertas
- Seção de Notas com suposições feitas (janela de renovação, ambiente assumido)
</output_specification>

<quality_criteria>
Outputs excelentes:
- A renovação é 100% automatizada, sem nenhum passo que dependa de uma pessoa lembrar de executá-lo
- Existe um mecanismo de monitoramento independente do próprio processo de renovação, capaz de alertar se a renovação falhar silenciosamente
- A distribuição do certificado renovado ao serviço é sem downtime (reload gracioso, não restart)

Evite:
- Propor qualquer fluxo que dependa de renovação manual, mesmo como "fallback"
- Configurar renovação automática sem nenhum monitoramento de que ela realmente funcionou
- Recomendar certificados self-signed para qualquer ambiente de produção
</quality_criteria>

<constraints>
- Nunca recomende certificado self-signed como solução para ambiente de produção voltado à internet
- Nunca inclua uma chave privada real ou plausível como exemplo — use sempre placeholders
- Não assuma DNS-01 quando HTTP-01 for suficiente (domínio único, sem wildcard) — use o método mais simples que atenda ao requisito
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Precisamos de certificados TLS para *.minhaempresa.com (wildcard) e api.minhaempresa.com, rodando em um cluster Kubernetes com ingress-nginx e DNS gerenciado no Route53."

**Output esperado (resumo):**

- Recomendação de Let's Encrypt via cert-manager, com solver DNS-01 (plugin Route53) obrigatório por causa do domínio wildcard
- `ClusterIssuer` configurado com o solver Route53 e um `Certificate` cobrindo `*.minhaempresa.com` e `api.minhaempresa.com`
- Renovação automática configurada pelo cert-manager (padrão de 30 dias antes da expiração), com reload gracioso do ingress-nginx via Secret watch
- Alerta proposto via Prometheus/Alertmanager monitorando a métrica de expiração exposta pelo cert-manager, disparando se restarem menos de 14 dias sem renovação
- Nota informando que a janela de renovação e o canal de alerta (Slack/e-mail) são sugestões a confirmar com a equipe
