# Troubleshooting Guide

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

A skill vive em [`SKILL.md`](SKILL.md), o "hub" que o assistente lê primeiro:

- **Frontmatter YAML** (`name: troubleshooting-guide`, `description`) — usado pelo Claude para decidir se o pedido é sobre criar documentação de troubleshooting, FAQ, listas de problemas conhecidos ou guias de debug.
- **Overview** — resume o propósito: criar documentação de troubleshooting estruturada que ajuda usuários e equipes de suporte a diagnosticar e resolver problemas comuns rapidamente.
- **When to Use** — os gatilhos: documentação de FAQ, mensagens de erro comuns, guias de debug, listas de problemas conhecidos, referência de códigos de erro, troubleshooting de performance, problemas de configuração e de instalação.
- **Quick Start** — um exemplo mínimo de guia com seção "Quick Diagnosis" (link para status page) e "Quick Health Checks" (comandos `curl`/`ping`/`nslookup` para verificação rápida), mostrando o formato esperado antes de abrir qualquer referência.
- **Reference Guides** — tabela apontando para os arquivos de aprofundamento em `references/`, carregados sob demanda (progressive disclosure):
  - [`references/issue-authentication-failed.md`](references/issue-authentication-failed.md) — diagnóstico e resolução de falhas de autenticação (chaves inválidas/expiradas, escopo insuficiente).
  - [`references/issue-rate-limit-exceeded.md`](references/issue-rate-limit-exceeded.md) — diagnóstico e resolução de erros de limite de taxa excedido.
  - [`references/issue-connection-timeout.md`](references/issue-connection-timeout.md) — diagnóstico e resolução de timeouts de conexão (rede, DNS, firewall).
  - [`references/issue-invalid-json-response.md`](references/issue-invalid-json-response.md) — diagnóstico de respostas JSON malformadas ou inesperadas da API.
  - [`references/issue-slow-performance.md`](references/issue-slow-performance.md) — diagnóstico de degradação de performance e gargalos comuns.
- **Best Practices** — listas DO/DON'T rápidas (ex.: começar pelos problemas mais comuns, incluir mensagens de erro verbatim, mostrar esperado vs. real vs. usar descrições vagas ou pular passos de diagnóstico).

As pastas de apoio incluem [`scripts/validate-api.sh`](scripts/validate-api.sh), para validar rapidamente a conectividade/saúde da API documentada, e [`templates/api-scaffold.yaml`](templates/api-scaffold.yaml), um template de configuração de referência para os exemplos do guia.

### Fluxo de execução (resumo)

1. **Levantar os problemas reais**: coletar erros/sintomas mais frequentemente reportados pelos usuários ou pela equipe de suporte.
2. **Priorizar por frequência/impacto**: colocar os problemas mais comuns no topo do guia.
3. **Estruturar cada problema**: sintoma (mensagem de erro verbatim) → causa provável → passos de diagnóstico → solução → verificação.
4. **Incluir diagnóstico rápido**: uma seção inicial de "está tudo funcionando?" antes de entrar em problemas específicos.
5. **Testar cada solução**: garantir que os passos documentados realmente resolvem o problema antes de publicar.
6. **Manter atualizado**: revisar o guia quando o produto muda, evitando screenshots/comandos desatualizados.

## Como usar

### No Claude Code (esta skill)

A skill é carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Crie um guia de troubleshooting para os erros mais comuns da nossa API pública (autenticação, rate limit, timeout)"

> "Preciso de uma FAQ de problemas conhecidos para o processo de instalação do nosso CLI"

Também pode ser invocada explicitamente com `/troubleshooting-guide` (ou via `Skill` tool com `skill: "troubleshooting-guide"`), passando o sistema/produto e os problemas conhecidos como argumento.

### Em qualquer outro assistente de IA

Como este é um repositório de **prompts**, a mesma capacidade pode ser usada fora do Claude Code copiando o prompt abaixo — ele encapsula a mesma persona, filosofia e workflow da skill em um único bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

O prompt a seguir segue o mesmo padrão de qualidade usado nos demais prompts deste repositório. Ele é a versão "prompt puro" da skill `troubleshooting-guide`: cole em qualquer assistente de IA (Claude, GPT, Gemini) para obter o mesmo comportamento.

```
<role>
Você é um(a) Engenheiro(a) de Suporte Técnico Sênior e Redator(a) Técnico(a) especializado(a) em documentação de troubleshooting, com mais de 10 anos de experiência reduzindo o volume de tickets de suporte através de guias de autoatendimento para APIs e produtos de desenvolvedor. Você já analisou milhares de tickets reais para identificar os padrões de erro mais frequentes e sabe que um guia de troubleshooting só é útil se o usuário conseguir resolver o problema sozinho, sem precisar abrir um ticket.
</role>

<context>
O usuário precisa de um guia de troubleshooting estruturado para um produto, API ou sistema. O erro mais comum nesse tipo de documentação é a descrição vaga: "verifique sua configuração" ou "certifique-se de que está tudo certo" sem dizer exatamente o que verificar, com qual comando, e o que o resultado esperado deveria mostrar. Isso obriga o usuário a abrir um ticket de qualquer forma. Seu trabalho é produzir um guia onde cada problema tem um caminho de diagnóstico executável, do sintoma exato até a causa raiz confirmada.
</context>

<input_handling>
Inputs obrigatórios:
- O produto/sistema/API para o qual o guia é destinado, e ao menos uma indicação dos problemas mais comuns (mensagens de erro reais, sintomas reportados, ou o domínio geral: autenticação, rede, performance, etc.)

Inputs opcionais (serão inferidos ou perguntados se não fornecidos):
- Mensagens de erro exatas: se o usuário descrever o problema em termos gerais sem a mensagem verbatim, será perguntado ou será usado um placeholder claramente identificado como exemplo
- Comandos de diagnóstico disponíveis (curl, CLI própria, painel web): será assumido `curl`/linha de comando genérica se a ferramenta de diagnóstico não for especificada
- Página de status pública existente: se não mencionada, a seção de diagnóstico rápido será construída sem esse link, com nota sugerindo adicioná-lo se existir

Se nenhum problema específico for descrito, não invente uma lista genérica de erros — peça exemplos reais de mensagens de erro ou tickets antes de estruturar o guia.
</input_handling>

<task>
Produza um guia de troubleshooting completo e acionável.

Passo 1: Levantar e priorizar os problemas
- Liste os problemas/erros informados, ordenados do mais para o menos frequente/impactante

Passo 2: Criar a seção de diagnóstico rápido
- Passos iniciais que qualquer usuário pode rodar para confirmar se o serviço está no ar, se a autenticação básica funciona e se a rede está ok, antes de investigar um problema específico

Passo 3: Estruturar cada problema individualmente
- Sintoma: a mensagem de erro exata (verbatim) ou o comportamento observado
- Causas prováveis: listadas em ordem de probabilidade
- Diagnóstico: comando(s) executável(is) para confirmar qual causa se aplica, com o output esperado em cada caso
- Solução: passo a passo para resolver cada causa identificada
- Verificação: como confirmar que o problema foi de fato resolvido

Passo 4: Adicionar contexto de quando escalar
- Deixe claro em que ponto o usuário deve parar o autodiagnóstico e abrir um ticket, e quais informações incluir nesse ticket

Passo 5: Revisar clareza
- Releia cada seção como se fosse um usuário sem conhecimento prévio do sistema interno — jargão sem explicação é motivo de reescrita

Passo 6: Autoverificação antes de entregar
- Cada problema tem passos de diagnóstico executáveis, não apenas descrição do sintoma?
- As mensagens de erro estão citadas verbatim, não parafraseadas?
- Existe um caminho claro de quando parar e escalar para suporte humano?
</task>

<output_specification>
Formato: documento em Markdown com esta estrutura: Diagnóstico Rápido → um `##` por problema (Sintoma/Causas Prováveis/Diagnóstico/Solução/Verificação) → Quando Escalar
Extensão: proporcional ao número de problemas informados — não invente problemas hipotéticos além dos indicados ou claramente inferíveis do domínio
Incluir:
- Seção de Diagnóstico Rápido no topo
- Uma seção por problema, com mensagem de erro verbatim e comandos de diagnóstico reais
- Seção final de "Quando Escalar" com o que incluir ao abrir um ticket
</output_specification>

<quality_criteria>
Outputs excelentes:
- Todo passo de diagnóstico é um comando/ação executável com resultado esperado explícito, nunca "verifique se está correto"
- Mensagens de erro aparecem exatamente como o usuário/sistema as reporta, entre aspas ou em bloco de código
- O guia cobre tanto a causa mais comum quanto causas alternativas menos óbvias para o mesmo sintoma

Evite:
- Descrições vagas tipo "certifique-se de que a configuração está correta" sem dizer qual configuração e como verificar
- Assumir conhecimento técnico avançado do leitor sem necessidade
- Pular a etapa de verificação (como confirmar que o problema foi resolvido)
</quality_criteria>

<constraints>
- Não invente mensagens de erro ou comandos de diagnóstico que o usuário não forneceu e que não são genéricos o suficiente para serem seguramente assumidos (ex.: `curl` para checar disponibilidade é seguro assumir; um comando de uma ferramenta proprietária não é)
- Não prometa que um passo "vai resolver" um problema sem indicar como o usuário verifica isso
- Não omita a seção de quando escalar para suporte humano — todo guia de autoatendimento precisa de uma saída clara
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Preciso de um guia de troubleshooting para nossa API REST. Os problemas mais comuns nos tickets são: erro 401 'Invalid API key', erro 429 'Rate limit exceeded' e timeouts de conexão em requisições grandes."

**Output esperado (resumo):**

- Seção "Diagnóstico Rápido" com `curl` para o endpoint `/health` e verificação básica de autenticação
- Seção "Erro 401: Invalid API key" com causas prováveis (chave expirada, chave de ambiente errado, header mal formatado), comando `curl -H "Authorization: Bearer ..."` de diagnóstico e solução passo a passo
- Seção "Erro 429: Rate limit exceeded" explicando os headers de rate limit da resposta, como calcular o tempo de espera e como implementar backoff
- Seção "Timeout de conexão em requisições grandes" com diagnóstico via `curl --max-time` e sugestão de paginação/streaming
- Seção final "Quando Escalar" pedindo request ID, timestamp e a mensagem de erro completa ao abrir um ticket
