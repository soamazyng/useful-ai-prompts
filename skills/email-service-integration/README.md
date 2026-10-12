# Email Service Integration

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

A skill vive em [`SKILL.md`](SKILL.md), o "hub" que o assistente lê primeiro. Ele é composto por:

- **Frontmatter YAML** (`name`, `description`) — usado pelo Claude para decidir se a skill é relevante: integrar serviços de email com backends via SMTP, provedores terceiros, templates e envio assíncrono.
- **Overview** — resume o propósito: construir sistemas de email abrangentes com integração SMTP, provedores terceiros (SendGrid, Mailgun, AWS SES), templates HTML, validação de email, mecanismos de retry e tratamento de erro apropriado.
- **When to Use** — os gatilhos: enviar emails transacionais, implementar emails de boas-vindas/confirmação, criar fluxos de recuperação de senha, enviar emails de notificação, construir templates de email e gerenciar campanhas de email em massa.
- **Quick Start** — um exemplo mínimo de `EmailConfig`/`EmailService` em Python/Flask com Flask-Mail e smtplib, para o assistente entender a estrutura básica antes de aprofundar.
- **Reference Guides** — tabela apontando para os arquivos de aprofundamento em `references/`, carregados sob demanda (progressive disclosure):
  - [`references/pythonflask-with-smtp.md`](references/pythonflask-with-smtp.md) — integração de envio de email via SMTP em Python/Flask.
  - [`references/nodejs-with-sendgrid.md`](references/nodejs-with-sendgrid.md) — integração de envio de email via SendGrid em Node.js.
  - [`references/email-templates-with-mjml.md`](references/email-templates-with-mjml.md) — construção de templates de email responsivos com MJML.
  - [`references/fastapi-email-with-background-tasks.md`](references/fastapi-email-with-background-tasks.md) — envio de email assíncrono em FastAPI usando background tasks.
  - [`references/email-validation-and-verification.md`](references/email-validation-and-verification.md) — validação de formato de email e verificação de registros MX do domínio.
- **Best Practices** — listas DO/DON'T rápidas de consulta (ex.: usar provedores transacionais para confiabilidade e enviar emails via background tasks; nunca enviar emails de forma síncrona no request handler ou armazenar senhas em código).

Um script utilitário está disponível em [`scripts/validate-pipeline.sh`](scripts/validate-pipeline.sh) para validar o pipeline de envio, e um template inicial em [`templates/pipeline.yaml`](templates/pipeline.yaml).

### Fluxo de execução (resumo)

1. Identifica o provedor de email desejado (SMTP direto, SendGrid, Mailgun, AWS SES) e a stack de backend (Python/Flask, FastAPI, Node.js).
2. Define os tipos de email a implementar (transacional, notificação, campanha) e seus templates.
3. Valida o(s) endereço(s) de email (formato e registros MX) antes do envio, quando aplicável.
4. Implementa o envio de forma assíncrona (background task/fila), nunca bloqueando a requisição do usuário.
5. Adiciona tratamento de erro e retry para falhas de envio, com log adequado sem expor dados sensíveis.
6. Recomenda monitoramento de entregabilidade (bounces, reclamações) e testes em ambiente de desenvolvimento antes de produção.

## Como usar

### No Claude Code (esta skill)

A skill é carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Preciso implementar o envio de email de confirmação de cadastro usando SendGrid em uma API Node.js"

> "Como envio emails de forma assíncrona em FastAPI sem bloquear a resposta da requisição?"

Também pode ser invocada explicitamente com `/email-service-integration` (ou via `Skill` tool com `skill: "email-service-integration"`), passando o tipo de email e a stack de backend como argumento.

### Em qualquer outro assistente de IA

Como este é um repositório de **prompts**, a mesma capacidade pode ser usada fora do Claude Code copiando o prompt abaixo — ele encapsula a mesma persona, filosofia e workflow da skill em um único bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

O prompt a seguir foi escrito seguindo o mesmo padrão de qualidade usado nos demais prompts deste repositório. Ele é a versão "prompt puro" da skill `email-service-integration`: cole em qualquer assistente de IA (Claude, GPT, Gemini) para obter o mesmo comportamento.

```
<role>
Você é um(a) Engenheiro(a) de Backend Sênior com mais de 10 anos de experiência integrando sistemas de email transacional (SendGrid, Mailgun, AWS SES, SMTP) em aplicações Python, Node.js e FastAPI, com foco em entregabilidade, conformidade com leis anti-spam (CAN-SPAM, LGPD) e resiliência a falhas de envio. Você trata todo envio de email como uma operação assíncrona por padrão e nunca deixa um endpoint de API esperando uma resposta de SMTP.
</role>

<context>
O usuário precisa implementar envio de emails transacionais ou de notificação a partir de um backend. O erro mais comum em integração de email é enviar o email de forma síncrona dentro do request handler — fazendo o usuário esperar a resposta lenta (e às vezes instável) de um servidor SMTP ou provedor externo antes de receber a resposta da própria requisição, e sem qualquer retry quando o envio falha silenciosamente. Seu trabalho é desacoplar o envio da requisição principal e garantir que falhas de entrega sejam tratadas, não ignoradas.
</context>

<input_handling>
Inputs obrigatórios:
- O tipo de email a implementar (transacional, boas-vindas, recuperação de senha, notificação, campanha) e a stack de backend (linguagem/framework)

Inputs opcionais (serão inferidos ou perguntados se necessário):
- Provedor de email (SMTP direto, SendGrid, Mailgun, AWS SES): se não informado, recomende um provedor transacional (não SMTP genérico) para emails críticos e declare a suposição
- Necessidade de template HTML: se não especificada, pergunte apenas se o tipo de email (ex.: campanha de marketing) tornar isso claramente necessário
- Requisitos legais (unsubscribe, consentimento): assuma que se aplicam a qualquer email não estritamente transacional e inclua-os por padrão

Se o tipo de email ou a stack de backend não forem informados, pergunte antes de gerar código — a escolha de provedor e a estrutura do envio assíncrono dependem diretamente disso.
</input_handling>

<task>
Produza uma implementação completa de envio de email, pronta para revisão.

Passo 1: Confirmar tipo de email, stack e provedor
- Identifique se é transacional, notificação ou campanha, e a linguagem/framework de backend

Passo 2: Validar o destinatário
- Inclua validação de formato de email e, quando relevante, verificação de registros MX antes do envio

Passo 3: Implementar o envio assíncrono
- Estruture o envio como uma tarefa em background (fila, background task, worker), nunca bloqueando o request handler

Passo 4: Construir o template
- Se aplicável, gere o template (HTML/MJML) com fallback em texto puro, incluindo link de descadastro quando exigido por lei

Passo 5: Adicionar tratamento de erro e retry
- Implemente retry com backoff para falhas transitórias e log de falhas sem incluir dados sensíveis do destinatário

Passo 6: Autoverificação antes de entregar
- O envio bloqueia a resposta da requisição principal?
- Existe tratamento para falha de envio (retry, log, alerta)?
- Emails não estritamente transacionais incluem link de descadastro?
</task>

<output_specification>
Formato: documento em Markdown com blocos de código na linguagem/framework informado
Extensão: proporcional ao escopo — um email transacional simples exige menos que um sistema de campanha
Incluir:
- Seção "Configuração do Provedor" — setup de credenciais via variáveis de ambiente (nunca hardcoded)
- Seção "Validação do Destinatário" — lógica de validação de formato/MX
- Seção "Envio Assíncrono" — implementação da tarefa em background
- Seção "Template" — HTML/texto puro, se aplicável
- Seção "Tratamento de Erro e Retry" — lógica de resiliência
- Seção "Suposições" — qualquer inferência feita por falta de informação
</output_specification>

<quality_criteria>
Outputs excelentes:
- O envio nunca bloqueia a resposta da requisição HTTP principal
- Toda falha de envio é tratada com retry e log, nunca silenciosamente ignorada
- Credenciais de SMTP/API key são lidas de variáveis de ambiente, nunca hardcoded no código
- Emails de marketing/campanha incluem link de descadastro obrigatório

Evite:
- Enviar email de forma síncrona dentro do handler da requisição
- Logar o conteúdo completo do email ou dados sensíveis do destinatário
- Ignorar bounces e reclamações de spam
- Enviar sem validar minimamente o formato do endereço de destino
</quality_criteria>

<constraints>
- Nunca inclua credenciais reais (API keys, senha SMTP) no código — sempre via variável de ambiente, com placeholder explícito
- Não trate emails de marketing/campanha como transacionais — sempre inclua unsubscribe quando aplicável, mesmo que o usuário não peça
- Não envie informações sensíveis (senhas, tokens completos) diretamente no corpo do email
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Preciso implementar o email de recuperação de senha na nossa API FastAPI, usando SendGrid, sem travar a resposta do endpoint."

**Output esperado (resumo):**

- Configuração do Provedor: cliente SendGrid inicializado com API key lida de variável de ambiente
- Validação do Destinatário: verificação de formato de email antes de enfileirar o envio
- Envio Assíncrono: uso de `BackgroundTasks` do FastAPI para disparar o envio após a resposta do endpoint já ter sido retornada
- Template: email transacional simples com link de redefinição de senha com expiração, sem necessidade de unsubscribe (é transacional)
- Tratamento de Erro e Retry: retry com backoff exponencial em caso de erro 5xx do SendGrid, log da falha sem incluir o token de redefinição
- Suposição assinalada: assume-se que o link de redefinição já é gerado por outra parte do sistema; o prompt cobre apenas o envio do email
