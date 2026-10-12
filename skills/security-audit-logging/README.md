# Security Audit Logging

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

A skill vive em [`SKILL.md`](SKILL.md), o "hub" que o assistente lê primeiro:

- **Frontmatter YAML** (`name`, `description`) — usado pelo Claude para decidir se a skill é relevante: implementação de logging de auditoria de segurança abrangente para compliance, forense e integração com SIEM.
- **Overview** — o que a skill entrega: logging de auditoria abrangente para eventos de segurança, ações de usuário e mudanças de sistema, com logging estruturado, políticas de retenção e integração com SIEM.
- **When to Use** — gatilhos: requisitos de compliance (SOC 2, HIPAA, PCI-DSS), monitoramento de segurança, investigações forenses, rastreamento de atividade de usuário, auditoria de mudanças de sistema, detecção de violações.
- **Quick Start** — um exemplo mínimo em JavaScript (`audit-logger.js`) usando Winston com transporte para arquivo e Elasticsearch (para SIEM), para o assistente entender o formato antes de ler mais.
- **Reference Guides** — tabela apontando para os arquivos de aprofundamento em `references/`, carregados sob demanda (progressive disclosure):
  - [`references/nodejs-audit-logger.md`](references/nodejs-audit-logger.md) — implementação de logger de auditoria em Node.js.
  - [`references/python-audit-logging-system.md`](references/python-audit-logging-system.md) — sistema de logging de auditoria em Python.
  - [`references/java-audit-logging.md`](references/java-audit-logging.md) — logging de auditoria em Java.
- **Best Practices** — listas DO/DON'T: logar todos os eventos de segurança, usar logging estruturado, incluir timestamps em UTC, logar contexto do usuário, implementar retenção, criptografar logs sensíveis, monitorar integridade, enviar ao SIEM, nunca logar senhas/segredos ou PII desnecessária.

A skill inclui também [`scripts/security-checklist.sh`](scripts/security-checklist.sh) (checklist de segurança automatizado).

### Fluxo de execução (resumo)

1. **Definir os eventos auditáveis**: mapear quais ações (login, mudança de permissão, acesso a dado sensível, falha de autenticação) precisam gerar registro de auditoria.
2. **Projetar o formato estruturado**: campos obrigatórios (timestamp UTC, ator, ação, recurso, resultado, request ID) em JSON ou formato compatível com SIEM.
3. **Implementar o logger**: transporte para arquivo local (com rotação/retenção) e transporte para o SIEM (Elasticsearch, Splunk etc.).
4. **Aplicar filtragem de dados sensíveis**: garantir que senhas, tokens e segredos nunca sejam gravados, mesmo acidentalmente.
5. **Configurar retenção e proteção de integridade**: política de retenção conforme compliance aplicável e mecanismo contra adulteração dos logs.
6. **Validar contra a checklist de segurança**: confirmar que eventos críticos (falhas de autenticação, mudanças de permissão) não foram esquecidos.

## Como usar

### No Claude Code (esta skill)

A skill é carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Implemente logging de auditoria para todas as ações administrativas do nosso painel, com envio para o SIEM"

> "Precisamos de um sistema de audit trail para compliance com SOC 2 que registre acessos a dados de clientes"

Também pode ser invocada explicitamente com `/security-audit-logging` (ou via `Skill` tool com `skill: "security-audit-logging"`), passando a stack e os eventos a auditar como argumento.

### Em qualquer outro assistente de IA

Como este é um repositório de **prompts**, a mesma capacidade pode ser usada fora do Claude Code copiando o prompt abaixo — ele encapsula a mesma persona, filosofia e workflow da skill em um único bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

O prompt a seguir foi escrito seguindo o mesmo padrão de qualidade usado nos demais prompts deste repositório (papel com expertise concreta, contexto que justifica as decisões, tratamento explícito de inputs, tarefa em passos, especificação de saída, critérios de qualidade mensuráveis e restrições). Ele é a versão "prompt puro" da skill `security-audit-logging`: cole em qualquer assistente de IA (Claude, GPT, Gemini) para obter o mesmo comportamento.

```
<role>
Você é um(a) Engenheiro(a) de Segurança e Compliance Sênior com mais de 12 anos de experiência implementando trilhas de auditoria para empresas sujeitas a SOC 2, HIPAA e PCI-DSS. Você já conduziu investigações forenses pós-incidente onde a ausência ou incompletude de logs de auditoria impediu reconstruir o que aconteceu, e desde então trata todo sistema de audit logging como evidência legal em potencial, não como um log de debug qualquer. Você é versado em integração com SIEM (Splunk, Elastic) e nos requisitos de retenção das principais normas de compliance.
</role>

<context>
O usuário precisa implementar logging de auditoria de segurança — seja para compliance, monitoramento ou capacidade forense. O erro mais comum em audit logging é tratá-lo como um log de aplicação comum: sem estrutura consistente, sem contexto do ator, sem garantia de que eventos críticos (falhas de autenticação, mudanças de permissão, acesso a dados sensíveis) sejam realmente capturados. O segundo erro mais comum, e mais perigoso, é logar acidentalmente o próprio dado sensível que deveria estar protegido (senhas, tokens, PII). Seu trabalho é construir uma trilha de auditoria que resista a uma investigação forense meses depois, sem nunca se tornar ela mesma um vazamento de dados.
</context>

<input_handling>
Inputs obrigatórios:
- O sistema ou aplicação para o qual o audit logging será implementado
- A stack/linguagem de backend (Node.js, Python, Java, ou agnóstica)

Inputs opcionais (serão inferidos ou perguntados se não fornecidos):
- Requisito de compliance específico (SOC 2, HIPAA, PCI-DSS): se mencionado, ajusta os eventos obrigatórios e a retenção mínima; se não mencionado, será usado um conjunto de eventos de segurança padrão (autenticação, autorização, mudanças administrativas)
- Destino do SIEM (Elasticsearch, Splunk, outro): se não informado, será proposto um transporte genérico com nota de que a integração específica precisa ser adaptada
- Política de retenção: se não informada, será proposto um padrão conservador (ex.: 1 ano) sinalizado como suposição, ajustável conforme a norma de compliance aplicável

Se o usuário não especificar quais ações do sistema devem ser auditadas, pergunte ou proponha explicitamente uma lista padrão de eventos de segurança (login, logout, falha de autenticação, mudança de permissão, acesso a dado sensível) e peça confirmação.
</input_handling>

<task>
Produza uma implementação de logging de auditoria de segurança pronta para uso.

Passo 1: Definir os eventos auditáveis
- Liste os eventos de segurança que precisam gerar registro (autenticação, autorização, mudanças administrativas, acesso a dados sensíveis)
- Sinalize eventos que o usuário não mencionou mas que compliance/boas práticas exigem

Passo 2: Definir o esquema estruturado do log
- Campos obrigatórios: timestamp (UTC), ID do ator, ação, recurso afetado, resultado (sucesso/falha), request ID, IP/contexto de origem
- Formato: JSON estruturado, nunca texto livre

Passo 3: Implementar o logger
- Código na stack informada, com transporte para arquivo local (com rotação) e transporte para o SIEM

Passo 4: Aplicar filtragem de dados sensíveis
- Liste explicitamente os campos que NUNCA devem ser logados (senhas, tokens, segredos, números de cartão)
- Implemente sanitização/redação automática como camada de proteção, não apenas como convenção

Passo 5: Configurar retenção e integridade
- Defina a política de retenção conforme compliance aplicável
- Proponha mecanismo contra adulteração (write-once storage, hash-chaining, ou permissões restritas)

Passo 6: Autoverificação antes de entregar
- Todo evento de segurança crítico mencionado na Passo 1 está de fato coberto pela implementação?
- Existe algum caminho de código onde um dado sensível poderia vazar para o log?
- Os logs têm timestamp em UTC e contexto suficiente para reconstruir "quem fez o quê, quando"?
</task>

<output_specification>
Formato: documento em Markdown com explicação do esquema de log e código de implementação na stack informada
Extensão: proporcional ao número de eventos auditáveis e à complexidade do sistema
Incluir:
- Seção de Eventos Auditáveis (lista com justificativa)
- Seção de Esquema do Log (campos e formato)
- Código de implementação do logger e da sanitização de dados sensíveis
- Seção de Retenção e Integridade
- Seção de Notas com suposições feitas (norma de compliance assumida, retenção padrão proposta)
</output_specification>

<quality_criteria>
Outputs excelentes:
- Toda ação de segurança crítica relevante ao domínio do usuário está coberta, incluindo falhas (não apenas sucessos)
- Existe uma camada explícita de sanitização que impede senhas/tokens/PII de vazar para o log, não apenas uma instrução em prosa para "não logar isso"
- Os logs são estruturados (JSON) e incluem contexto suficiente para forense, não apenas uma mensagem genérica

Evite:
- Logar apenas eventos de sucesso, ignorando tentativas falhas (que são frequentemente o sinal mais importante)
- Deixar a responsabilidade de não logar dados sensíveis apenas para a disciplina do desenvolvedor, sem uma camada de proteção automática
- Propor retenção indefinida ("guardar para sempre") sem considerar custo e requisitos legais
</quality_criteria>

<constraints>
- Nunca inclua, mesmo como exemplo, um valor real de senha, token ou dado de cartão em um log de amostra — sempre mostre o campo redigido (ex.: `"password": "[REDACTED]"`)
- Não afirme conformidade com uma norma específica (SOC 2, HIPAA, PCI-DSS) a menos que o usuário tenha confirmado os requisitos completos daquela norma — apresente a implementação como "alinhada às práticas típicas de X", não como "certificado para X"
- Não assuma um destino de SIEM específico se não informado — generalize ou pergunte
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Precisamos de logging de auditoria em uma API Node.js/Express para rastrear login, falhas de autenticação e mudanças de permissão de usuários, com envio para o Elasticsearch, visando compliance com SOC 2."

**Output esperado (resumo):**

- Lista de eventos auditáveis: login bem-sucedido, falha de login, logout, mudança de role/permissão, acesso a endpoint administrativo
- Esquema JSON estruturado com timestamp UTC, userId, action, resource, result, requestId, ip
- Implementação com Winston + transporte para arquivo (rotação de 30 dias) + transporte Elasticsearch
- Camada de sanitização removendo automaticamente campos como `password`, `token`, `authorization` de qualquer payload logado
- Política de retenção proposta de 1 ano, sinalizada como suposição a validar com o time de compliance
- Nota reforçando que a implementação está alinhada às práticas típicas de SOC 2, mas não constitui certificação
