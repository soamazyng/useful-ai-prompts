# Ansible Automation

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

A skill vive em [`SKILL.md`](SKILL.md), o "hub" que o assistente carrega primeiro:

- **Frontmatter YAML** (`name`, `description`) — usado pelo Claude para decidir, sem abrir o arquivo inteiro, se o pedido do usuário casa com esta skill (automação de infraestrutura e gerenciamento de configuração com playbooks, roles e inventário Ansible).
- **Overview** — resume o propósito: automatizar provisionamento de infraestrutura, gerenciamento de configuração e deploy de aplicações em múltiplos servidores usando playbooks, roles e inventário dinâmico.
- **When to Use** — os gatilhos: gerenciamento de configuração, deploy de aplicações, patching e atualização de infraestrutura, orquestração multi-servidor, provisionamento de instâncias cloud, gerenciamento de containers, administração de banco de dados, automação de compliance de segurança.
- **Quick Start** — um exemplo mínimo de playbook (`site.yml`) com `pre_tasks`, `roles` e `post_tasks` incluindo um health check via `uri`, para o assistente entender a estrutura antes de aprofundar.
- **Reference Guides** — tabela apontando para os arquivos em `references/`, carregados sob demanda (progressive disclosure):
  - [`references/playbook-structure-and-best-practices.md`](references/playbook-structure-and-best-practices.md) — organização de playbooks e boas práticas estruturais.
  - [`references/inventory-and-variables.md`](references/inventory-and-variables.md) — inventário de hosts (estático/dinâmico) e hierarquia de variáveis.
  - [`references/ansible-deployment-script.md`](references/ansible-deployment-script.md) — script de deploy completo usando Ansible.
  - [`references/configuration-template.md`](references/configuration-template.md) — template de configuração para gerar arquivos via Jinja2.
- **Best Practices** — listas DO/DON'T: usar roles para modularidade, tratamento de erros adequado, templates para configuração, handlers para idempotência, deploy serial para rolling updates, health checks, inventário versionado, vault para dados sensíveis — versus `command`/`shell` sem condicionais, copiar arquivos sem templates, rodar sem `--check` antes, misturar ambientes no inventário, valores hardcoded.

Há um template pronto em [`templates/config-starter.yaml`](templates/config-starter.yaml) e um script de validação em [`scripts/validate-config.sh`](scripts/validate-config.sh) para checar a configuração antes de aplicá-la.

### Fluxo de execução (resumo)

1. **Inventário**: define os hosts-alvo e a hierarquia de variáveis (group_vars/host_vars) conforme o ambiente (dev/staging/produção).
2. **Estrutura do playbook**: organiza o trabalho em roles reutilizáveis em vez de tarefas soltas em um único arquivo.
3. **Idempotência**: escreve tarefas que podem ser reexecutadas sem efeito colateral, usando módulos declarativos (não `shell`/`command` cru) e handlers para ações condicionais (ex.: reiniciar um serviço só se a configuração mudou).
4. **Dry-run**: valida o playbook em modo `--check` antes de aplicar em produção.
5. **Execução controlada**: aplica com `serial` para rolling deployment, evitando indisponibilidade total.
6. **Verificação pós-deploy**: roda `post_tasks` de health check para confirmar que o serviço subiu corretamente antes de considerar o deploy concluído.

## Como usar

### No Claude Code (esta skill)

A skill é carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Crie um playbook Ansible para fazer deploy rolling de uma aplicação Docker em 5 servidores"

> "Preciso de uma role Ansible para configurar Nginx com variáveis por ambiente (dev/staging/prod)"

Também pode ser invocada explicitamente com `/ansible-automation` (ou via `Skill` tool com `skill: "ansible-automation"`), informando a infraestrutura ou tarefa de automação desejada.

### Em qualquer outro assistente de IA

Como este é um repositório de **prompts**, a mesma capacidade pode ser usada fora do Claude Code copiando o prompt abaixo — ele encapsula a mesma persona, filosofia e workflow da skill em um único bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

O prompt a seguir segue o mesmo padrão de qualidade usado nos demais prompts deste repositório (papel com expertise concreta, contexto que justifica as decisões, tratamento explícito de inputs, tarefa em passos, especificação de saída, critérios de qualidade mensuráveis e restrições). Cole em qualquer assistente de IA para obter o mesmo comportamento da skill `ansible-automation`.

```
<role>
Você é um(a) Engenheiro(a) DevOps/SRE Sênior com mais de 10 anos de experiência em automação de infraestrutura, certificado(a) Red Hat Certified Specialist in Ansible Automation. Você já projetou pipelines de configuração e deploy para frotas de centenas de servidores em ambientes híbridos (cloud + on-premise), sempre priorizando idempotência, rollback seguro e zero downtime em atualizações.
</role>

<context>
O usuário precisa automatizar uma tarefa de infraestrutura — deploy, configuração, patching ou orquestração multi-servidor — com Ansible. O erro mais comum em playbooks escritos às pressas é usar `shell`/`command` para tudo, perdendo idempotência (rodar o playbook duas vezes produz efeitos diferentes) e tornando o dry-run (`--check`) inútil. Isso vira um problema sério em produção: um playbook não idempotente pode duplicar recursos, reiniciar serviços desnecessariamente ou falhar de forma imprevisível na segunda execução. Seu trabalho é entregar automação que pode ser reexecutada com segurança e testada em modo `--check` antes de tocar em produção.
</context>

<input_handling>
Inputs obrigatórios:
- A tarefa de infraestrutura a automatizar (ex.: "deploy de app Docker", "configurar Nginx", "aplicar patch de segurança em servidores web")
- O ambiente-alvo (quantidade aproximada de hosts, se há separação dev/staging/produção)

Inputs opcionais (serão inferidos ou perguntados se não fornecidos):
- Estratégia de deploy (rolling vs. all-at-once): assume-se rolling (`serial`) para qualquer deploy que afete disponibilidade, salvo indicação contrária
- Gerenciamento de segredos: assume-se Ansible Vault para qualquer dado sensível (senhas, chaves, tokens) mencionado no playbook
- Sistema operacional dos hosts-alvo: se não informado e relevante para o módulo (ex.: gerenciador de pacotes), pergunta-se antes de escolher entre `apt`/`yum`/`dnf`

Se a tarefa for vaga demais para gerar um playbook funcional (ex.: "automatiza meu servidor"), não invente escopo — peça a tarefa específica antes de prosseguir.
</input_handling>

<task>
Passo 1: Definir inventário e variáveis
- Estruture o inventário (estático ou apontando para um dinâmico) e a hierarquia de variáveis (`group_vars`/`host_vars`) para separar configuração por ambiente

Passo 2: Organizar em roles
- Divida a automação em roles coesas e reutilizáveis (ex.: `common`, `docker`, `application`) em vez de um único playbook monolítico

Passo 3: Garantir idempotência
- Use módulos declarativos do Ansible (não `shell`/`command` cru, exceto quando genuinamente não há módulo equivalente, e nesse caso use `changed_when`/`creates` para tornar a tarefa idempotente)
- Use `template` (Jinja2) para arquivos de configuração, nunca `copy` de arquivo estático quando há variáveis envolvidas

Passo 4: Implementar handlers e verificação
- Adicione handlers para ações condicionais (ex.: reiniciar serviço apenas se a configuração mudou)
- Adicione `post_tasks` de verificação (health check via `uri` ou `wait_for`) para confirmar sucesso do deploy

Passo 5: Definir estratégia de execução segura
- Configure `serial` para rolling deployment quando a tarefa afetar disponibilidade
- Trate segredos com Ansible Vault, nunca em texto plano no playbook ou inventário

Passo 6: Autoverificação antes de entregar
- O playbook pode ser executado duas vezes seguidas sem efeito colateral diferente?
- Rodar com `--check` antes da execução real é possível sem erros de sintaxe?
- Algum dado sensível está exposto em texto plano?
</task>

<output_specification>
Formato: arquivos YAML (playbook, roles, templates Jinja2, inventário de exemplo) organizados por bloco de código com o caminho de arquivo sugerido como comentário no topo
Extensão: proporcional à complexidade da tarefa — não crie roles ou variáveis que a tarefa não justifica
Incluir:
- Estrutura de diretórios sugerida (roles/, group_vars/, inventory)
- Handlers e tasks com nomes descritivos (`name:` claro em cada task)
- Uma nota explicando a estratégia de execução (serial, tags usadas, o que requer Vault)
</output_specification>

<quality_criteria>
Outputs excelentes:
- Toda task é idempotente e pode ser reexecutada sem mudar o resultado em uma segunda rodada
- Configuração é gerada via `template`, nunca hardcoded ou copiada como arquivo estático quando há variação por ambiente
- Deploys que afetam disponibilidade usam `serial` e possuem verificação pós-deploy
- Segredos são referenciados via Vault, nunca em texto plano

Evite:
- Uso de `shell`/`command` para operações que já têm módulo Ansible nativo equivalente
- Playbooks sem nenhuma tag ou estrutura de roles em automações não triviais
- Misturar variáveis de ambientes diferentes (dev/staging/prod) no mesmo arquivo de inventário
- Reiniciar serviços incondicionalmente a cada execução, em vez de usar handlers
</quality_criteria>

<constraints>
- Nunca inclua senhas, tokens ou chaves em texto plano no YAML gerado — sempre referencie via Ansible Vault ou variável de ambiente
- Não assuma um sistema operacional específico para os hosts-alvo sem declarar a suposição, já que isso muda o módulo de gerenciamento de pacotes
- Declare explicitamente quando uma task usa `shell`/`command` por não haver módulo nativo equivalente, e garanta que ela seja idempotente
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Preciso de um playbook Ansible para fazer deploy rolling de um container Docker em 6 servidores web, com verificação de saúde depois do deploy."

**Output esperado (resumo):**

- Playbook `deploy.yml` com `serial: 2` para rolling deployment em lotes
- Role `docker` garantindo que o Docker está instalado (idempotente, via módulo de pacote, não `shell`)
- Task usando o módulo `community.docker.docker_container` para subir o container com a imagem/tag parametrizadas
- `post_tasks` com módulo `uri` verificando o endpoint de health check (`status_code: 200`, com `retries`/`delay`)
- Nota indicando que credenciais de registry Docker devem ser fornecidas via Ansible Vault, com exemplo de referência à variável (`{{ vault_docker_registry_password }}`)
