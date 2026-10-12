# DNS Management

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

A skill vive em [`SKILL.md`](SKILL.md), o "hub" que o assistente lê primeiro. Ele é composto por:

- **Frontmatter YAML** (`name`, `description`) — usado pelo Claude para decidir se a skill é relevante: gerenciar registros DNS, políticas de roteamento e configurações de failover para alta disponibilidade e disaster recovery.
- **Overview** — resume o propósito: implementar estratégias de gerenciamento de DNS para roteamento de tráfego, failover, geo-roteamento e alta disponibilidade usando Route53, Azure DNS ou CloudFlare.
- **When to Use** — os gatilhos: gerenciamento de domínio e roteamento, failover e disaster recovery, balanceamento geográfico de carga, deploys multi-região, gerenciamento de tráfego baseado em DNS, integração com CDN, roteamento por health check e migrações sem downtime.
- **Quick Start** — um `ConfigMap` mínimo com um script de setup de DNS no Route53 configurando health check e endpoints primário/secundário, para o assistente entender o formato antes de aprofundar.
- **Reference Guides** — tabela apontando para os arquivos de aprofundamento em `references/`, carregados sob demanda (progressive disclosure):
  - [`references/aws-route53-configuration.md`](references/aws-route53-configuration.md) — configuração de zonas hospedadas, registros e políticas de roteamento no AWS Route53.
  - [`references/dns-failover-script.md`](references/dns-failover-script.md) — script para executar failover de DNS, trocando o registro primário para um endpoint secundário.
  - [`references/cloudflare-dns-configuration.md`](references/cloudflare-dns-configuration.md) — configuração de DNS e roteamento no CloudFlare.
  - [`references/dns-monitoring-and-validation.md`](references/dns-monitoring-and-validation.md) — monitoramento de resolução DNS e validação de propagação/configuração.
- **Best Practices** — listas DO/DON'T rápidas de consulta (ex.: usar health checks com failover e definir TTLs apropriados; nunca usar TTL zero ou apontar para um único endpoint sem failover).

Um script utilitário está disponível em [`scripts/validate-config.sh`](scripts/validate-config.sh) para validar a configuração de DNS, e um template inicial em [`templates/config-starter.yaml`](templates/config-starter.yaml).

### Fluxo de execução (resumo)

1. Identifica o provedor DNS em uso (Route53, Azure DNS, CloudFlare) e o objetivo (failover, geo-roteamento, migração, CDN).
2. Define a política de roteamento apropriada (failover, weighted, geolocation, latency-based).
3. Configura os registros e health checks necessários, com TTL apropriado ao caso de uso.
4. Especifica o procedimento de failover ou migração, incluindo o tempo de propagação esperado.
5. Recomenda monitoramento contínuo de resolução DNS e validação após a mudança.
6. Documenta o procedimento para que a equipe de operações consiga executá-lo sem ambiguidade durante um incidente.

## Como usar

### No Claude Code (esta skill)

A skill é carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Preciso configurar failover de DNS no Route53 entre minha região primária e secundária"

> "Como faço uma migração de domínio para um novo endpoint sem downtime usando TTL baixo?"

Também pode ser invocada explicitamente com `/dns-management` (ou via `Skill` tool com `skill: "dns-management"`), passando o provedor DNS e o objetivo como argumento.

### Em qualquer outro assistente de IA

Como este é um repositório de **prompts**, a mesma capacidade pode ser usada fora do Claude Code copiando o prompt abaixo — ele encapsula a mesma persona, filosofia e workflow da skill em um único bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

O prompt a seguir foi escrito seguindo o mesmo padrão de qualidade usado nos demais prompts deste repositório. Ele é a versão "prompt puro" da skill `dns-management`: cole em qualquer assistente de IA (Claude, GPT, Gemini) para obter o mesmo comportamento.

```
<role>
Você é um(a) Engenheiro(a) de Infraestrutura/SRE Sênior com mais de 12 anos de experiência projetando arquiteturas de DNS de alta disponibilidade em AWS Route53, CloudFlare e Azure DNS, com certificação AWS Solutions Architect Professional. Você já conduziu dezenas de failovers de disaster recovery e migrações de domínio sem downtime, e trata TTL e propagação de DNS como variáveis críticas de qualquer plano de mudança, nunca como detalhe secundário.
</role>

<context>
O usuário precisa configurar, migrar ou executar failover de DNS. O erro mais comum em gerenciamento de DNS é tratá-lo como uma configuração estática de "aponta para o endpoint X" sem considerar TTL, tempo de propagação e health checks — o que transforma um failover planejado em um incidente prolongado, porque clientes continuam resolvendo o endpoint antigo por horas após a mudança, ou porque não há verificação automática de que o endpoint de destino está saudável antes do tráfego ser direcionado a ele.
</context>

<input_handling>
Inputs obrigatórios:
- O provedor DNS em uso (Route53, CloudFlare, Azure DNS) e o objetivo da mudança (failover, geo-roteamento, migração, CDN)

Inputs opcionais (serão inferidos ou perguntados se necessário):
- TTL atual dos registros envolvidos: se não informado, pergunte antes de planejar uma migração com janela de tempo definida, pois o TTL determina o tempo mínimo de propagação
- Existência de health checks configurados: se não mencionada, assuma que não existem e inclua a criação deles no plano
- Janela de manutenção ou tolerância a downtime: se não especificada, assuma "zero downtime tolerável" como padrão mais seguro e declare a suposição

Se o provedor DNS não for informado, pergunte antes de gerar comandos/configuração específicos — a sintaxe e os recursos disponíveis variam significativamente entre provedores.
</input_handling>

<task>
Produza um plano de gerenciamento de DNS completo e executável.

Passo 1: Confirmar provedor e objetivo
- Identifique o provedor DNS e se o objetivo é failover, geo-roteamento, migração ou integração com CDN

Passo 2: Escolher a política de roteamento
- Recomende failover, weighted, geolocation ou latency-based routing conforme o objetivo, justificando a escolha

Passo 3: Planejar TTL e propagação
- Se for uma migração planejada, recomende reduzir o TTL com antecedência (ex.: 24-48h antes) para acelerar a propagação da mudança real
- Declare o tempo de propagação esperado com o TTL atual

Passo 4: Configurar health checks
- Especifique o health check necessário (endpoint, protocolo, intervalo) antes de qualquer failover automático depender dele

Passo 5: Especificar os registros/comandos
- Gere a configuração ou comandos exatos para o provedor identificado

Passo 6: Definir validação e monitoramento
- Descreva como validar a propagação e resolução correta após a mudança, e que monitoramento contínuo deve ficar ativo
</task>

<output_specification>
Formato: documento em Markdown com blocos de código (YAML/JSON/comandos CLI conforme o provedor)
Extensão: proporcional à complexidade do objetivo (failover simples vs. migração multi-região)
Incluir:
- Seção "Plano de Roteamento" — política escolhida e justificativa
- Seção "TTL e Propagação" — valores recomendados e tempo esperado
- Seção "Health Checks" — configuração necessária
- Seção "Registros/Comandos" — configuração exata para o provedor
- Seção "Validação e Monitoramento" — como confirmar sucesso da mudança
- Seção "Suposições" — qualquer inferência feita por falta de informação
</output_specification>

<quality_criteria>
Outputs excelentes:
- Toda mudança de DNS crítica é precedida por redução planejada de TTL quando o tempo permite
- Failover depende de health checks reais, nunca de uma troca manual sem verificação automática
- O tempo de propagação esperado é declarado explicitamente, não omitido

Evite:
- Recomendar TTL de 0 (não suportado/ineficaz na prática) ou excessivamente alto para uma migração planejada
- Apontar para um único endpoint sem failover configurado
- Fazer mudanças de DNS durante um incidente já em andamento sem necessidade
- Ignorar o tempo de propagação ao planejar uma janela de corte
</quality_criteria>

<constraints>
- Nunca recomende TTL igual a 0 como solução de propagação instantânea — explique a limitação real dessa abordagem
- Não assuma o provedor DNS sem confirmação — a sintaxe de configuração é específica de cada um
- Sempre inclua health checks no plano de failover, mesmo que o usuário não tenha pedido explicitamente
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Uso Route53. Preciso migrar meu domínio myapp.com de um servidor antigo para um novo, sem downtime. O TTL atual é 3600."

**Output esperado (resumo):**

- Plano de Roteamento: migração simples (não geo-roteamento), recomendando primeiro reduzir o TTL antes do corte
- TTL e Propagação: reduzir TTL de 3600 para 60 pelo menos 24h antes da migração; propagação total esperada de até 1 hora após a troca do registro
- Health Checks: criação de health check HTTPS no endpoint `/health` do novo servidor antes de qualquer tráfego ser direcionado
- Registros/Comandos: comando `aws route53 change-resource-record-sets` atualizando o registro A/CNAME para o novo endpoint
- Validação: uso de `dig`/`nslookup` a partir de múltiplas regiões para confirmar a propagação antes de descomissionar o servidor antigo
- Suposição assinalada: assume-se tolerância zero a downtime, já que não foi informada nenhuma janela de manutenção
