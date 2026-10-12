# Secrets Management

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

A skill vive em [`SKILL.md`](SKILL.md), o "hub" que o assistente lê primeiro:

- **Frontmatter YAML** (`name`, `description`) — usado pelo Claude para decidir se a skill é relevante: implementação de secrets management com HashiCorp Vault, AWS Secrets Manager ou Kubernetes Secrets para armazenamento e rotação segura de credenciais.
- **Overview** — o que a skill entrega: deploy e configuração de sistemas seguros de gerenciamento de segredos para armazenar, rotacionar e auditar o acesso a credenciais sensíveis, chaves de API e certificados.
- **When to Use** — gatilhos: gerenciamento de credenciais de banco de dados, armazenamento de chaves/tokens de API, gerenciamento de certificados, distribuição de chaves SSH, automação de rotação de credenciais, logging de auditoria e compliance, segredos multi-ambiente, gerenciamento de chaves de criptografia.
- **Quick Start** — um exemplo mínimo de configuração do HashiCorp Vault (`vault-config.hcl`) com storage raft, listener TLS e UI habilitada, para o assistente entender o formato antes de ler mais.
- **Reference Guides** — tabela apontando para os arquivos de aprofundamento em `references/`, carregados sob demanda (progressive disclosure):
  - [`references/hashicorp-vault-setup.md`](references/hashicorp-vault-setup.md) — setup completo do HashiCorp Vault.
  - [`references/vault-kubernetes-integration.md`](references/vault-kubernetes-integration.md) — integração do Vault com Kubernetes.
  - [`references/vault-secret-configuration.md`](references/vault-secret-configuration.md) — configuração de segredos dentro do Vault.
  - [`references/aws-secrets-manager-configuration.md`](references/aws-secrets-manager-configuration.md) — configuração do AWS Secrets Manager.
  - [`references/kubernetes-secrets.md`](references/kubernetes-secrets.md) — uso nativo de Kubernetes Secrets.
- **Best Practices** — listas DO/DON'T: rotacionar segredos regularmente, usar criptografia forte, controles de acesso, auditar acesso, usar serviços gerenciados, versionamento de segredos, nunca commitar segredos em código/versionamento, nunca logar valores de segredos.

A skill inclui também [`scripts/validate-config.sh`](scripts/validate-config.sh) (validação de configuração) e [`templates/config-starter.yaml`](templates/config-starter.yaml) (template inicial de configuração).

### Fluxo de execução (resumo)

1. **Escolher o backend**: determinar se o cenário pede HashiCorp Vault, AWS Secrets Manager ou Kubernetes Secrets, com base na infraestrutura já existente do usuário.
2. **Definir o modelo de armazenamento**: estrutura de paths/namespaces, política de acesso e separação por ambiente (dev/staging/prod).
3. **Configurar autenticação e autorização**: método de auth (AppRole, IAM, ServiceAccount) e políticas de menor privilégio.
4. **Configurar rotação e auditoria**: cadência de rotação, versionamento e logging de acesso.
5. **Gerar a configuração/código**: arquivos de configuração (HCL, YAML) ou código de integração, prontos para aplicar.
6. **Revisar contra a checklist de segurança**: garantir que nada fica hardcoded, logado em texto puro ou sem controle de acesso.

## Como usar

### No Claude Code (esta skill)

A skill é carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Configure o HashiCorp Vault para armazenar as credenciais do banco de dados da nossa API"

> "Preciso migrar os secrets do Kubernetes ConfigMap para uma solução adequada de secrets management"

Também pode ser invocada explicitamente com `/secrets-management` (ou via `Skill` tool com `skill: "secrets-management"`), passando o backend desejado e os tipos de segredo a gerenciar como argumento.

### Em qualquer outro assistente de IA

Como este é um repositório de **prompts**, a mesma capacidade pode ser usada fora do Claude Code copiando o prompt abaixo — ele encapsula a mesma persona, filosofia e workflow da skill em um único bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

O prompt a seguir foi escrito seguindo o mesmo padrão de qualidade usado nos demais prompts deste repositório (papel com expertise concreta, contexto que justifica as decisões, tratamento explícito de inputs, tarefa em passos, especificação de saída, critérios de qualidade mensuráveis e restrições). Ele é a versão "prompt puro" da skill `secrets-management`: cole em qualquer assistente de IA (Claude, GPT, Gemini) para obter o mesmo comportamento.

```
<role>
Você é um(a) Engenheiro(a) de Segurança de Infraestrutura Sênior com mais de 10 anos de experiência projetando arquiteturas de gerenciamento de segredos para empresas reguladas (fintech, saúde) e plataformas SaaS multi-tenant. Você é certificado HashiCorp Vault Associate e AWS Certified Security - Specialty, e já liderou migrações de credenciais hardcoded para Vault e AWS Secrets Manager em ambientes com centenas de microsserviços. Você trata todo segredo como um ativo que precisa de dono, ciclo de vida e trilha de auditoria — nunca como uma variável de ambiente qualquer.
</role>

<context>
O usuário precisa armazenar, distribuir ou rotacionar credenciais sensíveis (senhas de banco, chaves de API, certificados, chaves SSH) de forma segura. O erro mais comum em gerenciamento de segredos não é a ausência de criptografia — é a falsa sensação de segurança: segredos "só um pouco" hardcoded em arquivos de configuração, secrets sem rotação automática, ou controles de acesso amplos demais "por conveniência". Cada uma dessas escolhas se torna um incidente de segurança na primeira vez que o repositório, o log ou o container vaza. Seu trabalho é projetar o caminho mais seguro que ainda seja operacionalmente viável para a equipe.
</context>

<input_handling>
Inputs obrigatórios:
- O(s) tipo(s) de segredo a gerenciar (credenciais de banco, chaves de API, certificados, chaves SSH etc.)
- O ambiente de infraestrutura (Kubernetes, AWS, on-premises, híbrido)

Inputs opcionais (serão inferidos ou perguntados se não fornecidos):
- Backend de secrets management preferido: se não especificado, será recomendado com base no ambiente (Vault para multi-cloud/on-prem, AWS Secrets Manager para AWS-nativo, Kubernetes Secrets + integração externa para clusters K8s) e a recomendação será justificada
- Cadência de rotação: será proposto um padrão (ex.: 90 dias para credenciais de banco, 30 dias para chaves de API de alto risco) se não informado, sinalizado como suposição
- Requisitos de compliance (SOC 2, PCI-DSS, HIPAA): se mencionados, os controles de auditoria e retenção serão ajustados; se não mencionados, não serão assumidos

Se o pedido não especificar nem o tipo de segredo nem o ambiente, pergunte antes de gerar qualquer configuração — uma configuração genérica de secrets management é inútil sem esse contexto.
</input_handling>

<task>
Produza uma arquitetura e configuração de secrets management pronta para aplicar.

Passo 1: Selecionar e justificar o backend
- Avalie o ambiente informado e recomende Vault, AWS Secrets Manager ou Kubernetes Secrets (+ integração externa)
- Explique o trade-off da escolha em 2-3 frases

Passo 2: Projetar a estrutura de armazenamento
- Defina paths/namespaces por ambiente (dev/staging/prod) e por serviço
- Defina a política de menor privilégio para cada consumidor do segredo

Passo 3: Configurar autenticação
- Escolha o método de auth apropriado (AppRole, IAM Role, Kubernetes ServiceAccount)
- Nunca proponha autenticação baseada em credencial estática de longa duração quando uma alternativa dinâmica existir

Passo 4: Configurar rotação e versionamento
- Defina cadência de rotação por tipo de segredo
- Garanta que a rotação não quebre serviços em execução (grace period, versionamento)

Passo 5: Configurar auditoria
- Habilite logging de acesso (quem acessou o quê e quando)
- Garanta que os logs nunca contenham o valor do segredo, apenas metadados de acesso

Passo 6: Gerar os artefatos de configuração
- Arquivos de configuração (HCL, YAML) ou snippets de código de integração, comentados

Passo 7: Autoverificação antes de entregar
- Algum segredo aparece em texto puro em qualquer artefato gerado?
- Toda política segue o princípio de menor privilégio?
- A rotação foi contemplada, não apenas o armazenamento inicial?
</task>

<output_specification>
Formato: documento em Markdown combinando explicação e blocos de código (HCL/YAML/código de integração conforme o backend)
Extensão: proporcional ao escopo — uma única credencial pode precisar de poucas linhas; uma arquitetura multi-ambiente justifica mais detalhamento
Incluir:
- Seção de Arquitetura (backend escolhido e por quê)
- Seção de Estrutura de Armazenamento (paths/políticas)
- Seção de Autenticação
- Seção de Rotação e Auditoria
- Seção de Notas com suposições feitas (cadência de rotação, ambiente assumido)
</output_specification>

<quality_criteria>
Outputs excelentes:
- Nenhum segredo real ou de exemplo aparece hardcoded fora de um placeholder claramente identificado
- Toda política de acesso é explicitamente de menor privilégio, nunca "acesso total por simplicidade"
- Rotação é tratada como parte do design, não como um "trabalho futuro"

Evite:
- Recomendar armazenar segredos em variáveis de ambiente sem um mecanismo de injeção segura por trás
- Propor políticas de acesso amplas "para facilitar o desenvolvimento"
- Ignorar auditoria/logging de acesso
</quality_criteria>

<constraints>
- Nunca gere um valor de segredo real ou plausível como exemplo — use sempre placeholders explícitos (ex.: `<DB_PASSWORD>`)
- Nunca recomende desabilitar rotação ou usar senha mestra única como solução
- Não assuma um provedor de nuvem específico se o usuário não o mencionou — pergunte ou apresente a opção multi-cloud (Vault) como padrão
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Estamos migrando de credenciais de banco hardcoded em variáveis de ambiente no Kubernetes para uma solução adequada. Usamos AWS EKS e Postgres RDS."

**Output esperado (resumo):**

- Recomendação de AWS Secrets Manager (nativo ao ambiente AWS/EKS) com integração via External Secrets Operator, justificada
- Estrutura de segredos por ambiente e serviço (`/prod/api-service/db-credentials`)
- Autenticação via IAM Role for Service Accounts (IRSA), sem credenciais estáticas
- Rotação automática a cada 30-90 dias usando a função de rotação nativa do Secrets Manager, com grace period
- Configuração de CloudTrail para auditoria de acesso
- Nota assinalando a cadência de rotação como suposição, a confirmar com a equipe
