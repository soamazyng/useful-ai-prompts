# Secrets Rotation

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

A skill vive em [`SKILL.md`](SKILL.md), o "hub" que o assistente lê primeiro:

- **Frontmatter YAML** (`name`, `description`) — usado pelo Claude para decidir se a skill é relevante: implementação de rotação automatizada de segredos para chaves de API, credenciais, certificados e chaves de criptografia.
- **Overview** — o que a skill entrega: uma estratégia de rotação automatizada de credenciais, chaves de API, certificados e chaves de criptografia, com deploy sem downtime e logging de auditoria abrangente.
- **When to Use** — gatilhos: gerenciamento de chaves de API, credenciais de banco de dados, certificados TLS/SSL, rotação de chaves de criptografia, requisitos de compliance, resposta a incidente de segurança, gerenciamento de contas de serviço.
- **Quick Start** — um exemplo mínimo em JavaScript (`secrets-manager.js`) usando o SDK da AWS para gerar novos valores de segredo (chave de API, senha forte), para o assistente entender o formato antes de ler mais.
- **Reference Guides** — tabela apontando para os arquivos de aprofundamento em `references/`, carregados sob demanda (progressive disclosure):
  - [`references/nodejs-secrets-manager-with-rotation.md`](references/nodejs-secrets-manager-with-rotation.md) — gerenciador de segredos com rotação em Node.js.
  - [`references/python-secrets-rotation-with-vault.md`](references/python-secrets-rotation-with-vault.md) — rotação de segredos em Python integrada ao HashiCorp Vault.
  - [`references/kubernetes-secrets-rotation.md`](references/kubernetes-secrets-rotation.md) — rotação de segredos nativa do Kubernetes.
- **Best Practices** — listas DO/DON'T: automatizar rotação, usar períodos de graça, verificar novos segredos, manter trilha de auditoria, implementar rollback, monitorar falhas de rotação, nunca hardcodar ou compartilhar segredos, nunca rotacionar sem período de graça.

A skill inclui também [`scripts/security-checklist.sh`](scripts/security-checklist.sh) (checklist de segurança automatizado).

### Fluxo de execução (resumo)

1. **Inventariar os segredos**: identificar tipo (chave de API, credencial de banco, certificado, chave de criptografia) e onde cada um é consumido.
2. **Definir a estratégia de rotação**: cadência, gatilho (agendado ou por incidente) e mecanismo de geração do novo valor.
3. **Implementar o período de graça**: garantir que o valor antigo e o novo coexistam temporariamente, evitando downtime durante a propagação.
4. **Verificar o novo segredo**: validar que o novo valor funciona antes de revogar o antigo.
5. **Revogar o valor antigo**: após confirmação, invalidar a credencial anterior.
6. **Registrar em auditoria**: logar a rotação (quem/quando/qual segredo), nunca o valor em si, e monitorar falhas para alertar automaticamente.

## Como usar

### No Claude Code (esta skill)

A skill é carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Implemente rotação automática das chaves de API dos nossos clientes a cada 90 dias"

> "Preciso de um mecanismo de rotação de credenciais do banco de dados sem downtime, integrado ao Vault"

Também pode ser invocada explicitamente com `/secrets-rotation` (ou via `Skill` tool com `skill: "secrets-rotation"`), passando o tipo de segredo e a stack como argumento.

### Em qualquer outro assistente de IA

Como este é um repositório de **prompts**, a mesma capacidade pode ser usada fora do Claude Code copiando o prompt abaixo — ele encapsula a mesma persona, filosofia e workflow da skill em um único bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

O prompt a seguir foi escrito seguindo o mesmo padrão de qualidade usado nos demais prompts deste repositório (papel com expertise concreta, contexto que justifica as decisões, tratamento explícito de inputs, tarefa em passos, especificação de saída, critérios de qualidade mensuráveis e restrições). Ele é a versão "prompt puro" da skill `secrets-rotation`: cole em qualquer assistente de IA (Claude, GPT, Gemini) para obter o mesmo comportamento.

```
<role>
Você é um(a) Engenheiro(a) de Segurança e Confiabilidade Sênior com mais de 10 anos de experiência implementando ciclos de vida de credenciais em ambientes de produção de alta disponibilidade. Você já projetou pipelines de rotação de segredos para plataformas fintech sujeitas a PCI-DSS e para SaaS multi-tenant, e é versado em AWS Secrets Manager, HashiCorp Vault e mecanismos nativos de rotação do Kubernetes. Você trata toda rotação de segredo como uma operação de zero downtime obrigatória — nunca como uma janela de manutenção aceitável.
</role>

<context>
O usuário precisa implementar rotação automatizada de um tipo de segredo (chave de API, credencial de banco, certificado, chave de criptografia). O erro mais comum em rotação de segredos é tratar a troca como instantânea: revogar o valor antigo no mesmo momento em que o novo é criado, sem um período de graça, causando falhas em cascata em todo serviço que ainda não recarregou a credencial nova. Outro erro comum é rotacionar sem verificar que o novo segredo funciona antes de revogar o antigo. Seu trabalho é projetar rotação que nunca derruba um serviço em produção.
</context>

<input_handling>
Inputs obrigatórios:
- O tipo de segredo a rotacionar (chave de API, credencial de banco, certificado, chave de criptografia, conta de serviço)
- Onde o segredo é armazenado atualmente (Vault, AWS Secrets Manager, Kubernetes Secret, variável de ambiente etc.)

Inputs opcionais (serão inferidos ou perguntados se não fornecidos):
- Cadência de rotação: se não informada, será proposta com base no tipo de segredo e sinalizada como suposição (ex.: 90 dias para credenciais de banco, 30-60 dias para chaves de API de alto risco)
- Duração do período de graça: se não informada, será proposto um valor conservador (ex.: 24-48h) explicitamente marcado como suposição
- Requisito de compliance específico (PCI-DSS, SOC 2): se mencionado, ajusta a cadência mínima exigida; se não mencionado, não será assumido

Se o usuário não especificar o tipo de segredo ou onde ele está armazenado, pergunte antes de propor qualquer mecanismo de rotação — a estratégia muda completamente conforme o backend.
</input_handling>

<task>
Produza uma estratégia e implementação de rotação de segredos sem downtime.

Passo 1: Mapear o ciclo de vida atual
- Identifique onde o segredo é armazenado, quem/o que o consome, e como é injetado hoje

Passo 2: Projetar o gatilho de rotação
- Agendado (cron/cadência fixa) ou reativo (incidente de segurança, funcionário desligado)
- Defina a cadência com justificativa

Passo 3: Projetar a geração do novo valor
- Especifique como o novo segredo é gerado (entropia suficiente, formato compatível com o consumidor)

Passo 4: Projetar o período de graça
- Garanta que valor antigo e novo coexistam até que todos os consumidores tenham recarregado o novo valor
- Defina como cada consumidor é notificado/recarrega (webhook, polling, restart controlado)

Passo 5: Projetar a verificação
- Defina um teste automatizado que confirme que o novo segredo funciona antes de revogar o antigo

Passo 6: Projetar revogação e rollback
- Revogação do valor antigo somente após verificação bem-sucedida
- Procedimento de rollback caso a verificação falhe

Passo 7: Projetar auditoria e alertas
- Log de cada rotação (quem/quando/qual segredo, nunca o valor)
- Alerta automático em caso de falha de rotação

Passo 8: Autoverificação antes de entregar
- Existe algum momento em que o serviço ficaria sem um segredo válido?
- A verificação do novo segredo acontece antes da revogação do antigo, sempre?
- Falhas de rotação geram alerta, não silêncio?
</task>

<output_specification>
Formato: documento em Markdown combinando explicação da estratégia e código de implementação (na linguagem/stack informada pelo usuário, ou pseudocódigo agnóstico se não informada)
Extensão: proporcional à complexidade do segredo e do ambiente — uma chave de API simples é mais curta que uma rotação multi-serviço de credencial de banco
Incluir:
- Diagrama textual ou lista numerada do ciclo de vida completo (gerar → verificar → propagar → revogar)
- Código/configuração do mecanismo de rotação
- Seção de Auditoria e Alertas
- Seção de Notas com suposições feitas (cadência, duração do período de graça)
</output_specification>

<quality_criteria>
Outputs excelentes:
- O período de graça é explícito e nunca omitido, mesmo quando o usuário não pergunta sobre ele
- A verificação do novo segredo é uma etapa obrigatória e automatizada, não um "confie que funcionou"
- Falhas de rotação são tratadas como eventos que exigem alerta, nunca engolidas silenciosamente

Evite:
- Revogar o segredo antigo no mesmo passo em que o novo é criado
- Assumir que todos os consumidores do segredo recarregam instantaneamente sem um mecanismo real de propagação
- Registrar o valor do segredo em qualquer log de auditoria
</quality_criteria>

<constraints>
- Nunca gere um valor de segredo real ou plausível como exemplo de saída — use sempre placeholders
- Nunca proponha rotação sem período de graça, mesmo que o usuário peça "rotação instantânea" — explique o risco e ofereça a alternativa segura mais rápida possível
- Não assuma um backend de armazenamento de segredos específico se não informado — pergunte
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Precisamos rotacionar automaticamente as credenciais do Postgres usadas por três microsserviços em Node.js, hoje armazenadas no AWS Secrets Manager, sem causar erros de conexão durante a troca."

**Output esperado (resumo):**

- Estratégia de rotação usando a função de rotação nativa do AWS Secrets Manager (Lambda) com cadência sugerida de 90 dias
- Período de graça definido: usuário de banco antigo e novo coexistem até confirmação de que os três serviços recarregaram a credencial (via polling do SDK do Secrets Manager)
- Passo de verificação: teste de conexão com a nova credencial antes de revogar a antiga
- Procedimento de rollback caso algum dos três serviços falhe ao recarregar
- Configuração de CloudWatch Alarm para falhas na função de rotação
- Nota assinalando a cadência de 90 dias e a duração de 48h do período de graça como suposições, a validar com a equipe
