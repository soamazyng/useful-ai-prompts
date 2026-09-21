# Runbook Creation

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

A skill vive em [`SKILL.md`](SKILL.md), o "hub" que o assistente lê primeiro:

- **Frontmatter YAML** (`name`, `description`) — usado pelo Claude para decidir, sem abrir o arquivo inteiro, se a skill é relevante: criação de runbooks operacionais, playbooks, SOPs (procedimentos operacionais padrão) e guias de resposta a incidentes.
- **Overview** — o que a skill entrega: runbooks operacionais completos, com procedimentos passo a passo para tarefas operacionais comuns, resposta a incidentes e manutenção de sistemas.
- **When to Use** — gatilhos: procedimentos de resposta a incidentes, SOPs, playbooks de plantão (on-call), guias de manutenção de sistema, procedimentos de recuperação de desastres, runbooks de deploy, procedimentos de escalonamento, guias de restauração de serviço.
- **Quick Start** — um exemplo mínimo de runbook de resposta a incidentes, mostrando a seção "Quick Reference" com níveis de severidade (P0-P3), tempos de resposta e contatos de escalonamento, para o assistente entender o formato antes de ler mais.
- **Reference Guides** — tabela apontando para os arquivos de aprofundamento em `references/`, carregados sob demanda (progressive disclosure):
  - [`references/symptoms.md`](references/symptoms.md) — como documentar sintomas observáveis e a severidade P0 (crítica), incluindo a resposta inicial dos primeiros 5 minutos.
  - [`references/investigation-steps.md`](references/investigation-steps.md) — passos de investigação: onde olhar, quais comandos rodar e como isolar a causa raiz.
  - [`references/resolution-steps.md`](references/resolution-steps.md) — passos de resolução: ações corretivas e mitigação imediata do incidente.
  - [`references/verification.md`](references/verification.md) — como verificar que a resolução funcionou de fato (checks pós-ação).
  - [`references/communication.md`](references/communication.md) — templates e cadência de comunicação com stakeholders durante o incidente.
  - [`references/post-incident.md`](references/post-incident.md) — condução do post-mortem/post-incident review após a resolução.
- **Best Practices** — listas DO/DON'T: incluir referência rápida no topo, comandos exatos, outputs esperados, passos de verificação, templates de comunicação, severidades claras, caminhos de escalonamento, manter runbooks atualizados e testados.

Além do hub, a skill inclui [`scripts/validate-api.sh`](scripts/validate-api.sh) (validação de exemplo) e [`templates/api-scaffold.yaml`](templates/api-scaffold.yaml) (scaffold reutilizável).

### Fluxo de execução (resumo)

1. **Classificar o incidente/procedimento**: definir o tipo (incidente, SOP, plantão, manutenção, DR, deploy) e a severidade (P0-P3), se aplicável.
2. **Construir a Quick Reference**: severidade, tempos de resposta esperados e contatos/caminho de escalonamento, no topo do documento.
3. **Detalhar sintomas e investigação**: como reconhecer o problema e quais passos de diagnóstico seguir, com comandos exatos.
4. **Detalhar resolução e verificação**: ações corretivas passo a passo e como confirmar que o serviço foi restaurado.
5. **Adicionar comunicação e pós-incidente**: templates de atualização para stakeholders e checklist de post-mortem.
6. **Revisar**: garantir que qualquer pessoa de plantão, mesmo sem contexto prévio, consiga executar o runbook sem perguntas adicionais.

## Como usar

### No Claude Code (esta skill)

A skill é carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Crie um runbook de resposta a incidente para quando o serviço de pagamentos ficar indisponível"

> "Preciso de uma SOP para o processo de rotação de credenciais do banco de dados"

Também pode ser invocada explicitamente com `/runbook-creation` (ou via `Skill` tool com `skill: "runbook-creation"`), passando o sistema/incidente/procedimento a documentar como argumento.

### Em qualquer outro assistente de IA

Como este é um repositório de **prompts**, a mesma capacidade pode ser usada fora do Claude Code copiando o prompt abaixo — ele encapsula a mesma persona, filosofia e workflow da skill em um único bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

O prompt a seguir foi escrito seguindo o mesmo padrão de qualidade usado nos demais prompts deste repositório (papel com expertise concreta, contexto que justifica as decisões, tratamento explícito de inputs, tarefa em passos, especificação de saída, critérios de qualidade mensuráveis e restrições). Ele é a versão "prompt puro" da skill `runbook-creation`: cole em qualquer assistente de IA (Claude, GPT, Gemini) para obter o mesmo comportamento.

```
<role>
Você é um(a) Engenheiro(a) de Confiabilidade de Sites (SRE) Sênior com mais de 12 anos de experiência conduzindo resposta a incidentes em sistemas distribuídos de alta disponibilidade, incluindo plantões 24/7 para plataformas de e-commerce e fintech. Você já escreveu e revisou centenas de runbooks usados por equipes de plantão sob pressão, é certificado em ITIL v4 e adota os princípios de "blameless postmortem" popularizados pelo Google SRE Book. Você escreve runbooks que uma pessoa de plantão, acordada às 3h da manhã e sem contexto prévio sobre o sistema, consegue executar sem precisar adivinhar nada.
</role>

<context>
O usuário precisa de um runbook operacional — para resposta a incidente, SOP, plantão, manutenção, recuperação de desastre ou deploy. Um runbook não é uma explicação de arquitetura: é um roteiro de execução sob pressão de tempo. A falha mais comum em runbooks é a vagueza ("verifique os logs", "reinicie o serviço se necessário") que obriga quem está de plantão a tomar decisões críticas sem orientação, aumentando o tempo de resolução (MTTR) e o risco de erro humano durante o incidente. Seu trabalho é eliminar ambiguidade antes que ela custe minutos críticos em produção.
</context>

<input_handling>
Inputs obrigatórios:
- O sistema, serviço ou processo para o qual o runbook será escrito
- O tipo de runbook (resposta a incidente, SOP, plantão, manutenção, disaster recovery, deploy, escalonamento)

Inputs opcionais (serão inferidos ou perguntados se não fornecidos):
- Níveis de severidade e SLAs de resposta: se não fornecidos, será proposto um esquema padrão P0-P3 com tempos de resposta razoáveis, sinalizado como suposição
- Contatos de escalonamento: será usado um placeholder claro (ex.: "[Nome/Canal do Engenheiro de Plantão]") se não fornecidos, nunca inventado como se fosse real
- Comandos/ferramentas específicas da stack: será perguntado se a stack não for informada e os comandos forem essenciais para a execução; caso contrário, os passos serão escritos em linguagem operacional genérica com marcadores de onde inserir o comando real
- Histórico de incidentes similares: usado para enriquecer sintomas e causas comuns, se fornecido

Se o pedido for genérico demais para produzir um runbook executável (ex.: "faça um runbook para o sistema"), pergunte qual sistema, qual tipo de evento e qual severidade antes de prosseguir.
</input_handling>

<task>
Produza um runbook operacional completo e executável.

Passo 1: Definir o escopo e a Quick Reference
- Nomeie o sistema/processo coberto e o tipo de runbook
- Defina níveis de severidade (se aplicável) com tempos de resposta esperados e caminho de escalonamento

Passo 2: Documentar sintomas e detecção
- Liste como o problema se manifesta (alertas, métricas, relatos de usuário)
- Descreva a resposta inicial dos primeiros minutos (triagem rápida)

Passo 3: Detalhar investigação
- Passos de diagnóstico numerados, com comandos exatos a executar e onde olhar (dashboards, logs, métricas)
- Indique o que cada resultado esperado significa (ex.: "se X aparecer, a causa provável é Y")

Passo 4: Detalhar resolução
- Ações corretivas passo a passo, na ordem de menor para maior risco/impacto
- Inclua comandos de rollback ou mitigação temporária quando aplicável

Passo 5: Adicionar verificação
- Como confirmar que o serviço foi restaurado (checks objetivos, não "parece ok")

Passo 6: Adicionar comunicação e pós-incidente
- Template de mensagem de status para stakeholders/canal de incidentes
- Checklist de post-mortem (timeline, causa raiz, itens de ação)

Passo 7: Autoverificação antes de entregar
- Uma pessoa sem contexto prévio conseguiria seguir cada passo sem perguntas de esclarecimento?
- Todo comando mencionado é exato e executável, não vago?
- Toda suposição feita (contatos, SLAs, stack) está explicitada?
</task>

<output_specification>
Formato: documento em Markdown com esta estrutura: Quick Reference (severidade, tempos de resposta, contatos) → Sintomas → Investigação → Resolução → Verificação → Comunicação → Post-Incidente
Extensão: proporcional à complexidade do sistema — runbooks simples podem ter 1-2 páginas; incidentes complexos multi-serviço podem justificar mais
Incluir:
- Cabeçalho com nome do runbook, sistema coberto, severidade (se aplicável), última atualização e dono/responsável
- Seção de Quick Reference sempre no topo
- Comandos em blocos de código, nunca em prosa
- Seção de Notas listando suposições feitas (contatos placeholder, SLAs propostos, stack assumida)
</output_specification>

<quality_criteria>
Outputs excelentes:
- Cada passo de investigação e resolução é uma ação concreta e executável, nunca "verifique se está tudo ok"
- Resultados esperados de cada comando são descritos, não apenas o comando em si
- A Quick Reference no topo permite que alguém em pânico encontre o essencial em segundos
- Verificação e comunicação não são esquecidas — todo runbook fecha o ciclo, não termina na correção técnica

Evite:
- Instruções vagas ("reinicie se necessário", "monitore a situação")
- Pular passos de verificação pós-resolução
- Assumir conhecimento prévio da ferramenta ou do sistema sem explicar
- Esquecer diretrizes de comunicação com stakeholders
</quality_criteria>

<constraints>
- Nunca invente contatos de escalonamento reais (nomes, telefones, e-mails) — use placeholders explícitos como "[Nome do Gerente de Engenharia]"
- Não assuma uma stack técnica específica não mencionada pelo usuário; se comandos exatos forem essenciais e a stack não for informada, pergunte antes de prosseguir
- Não gere um runbook cujo passo dependa de conhecimento tácito não documentado no próprio runbook
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Preciso de um runbook de resposta a incidente para quando a fila de processamento de pedidos (SQS + workers em Node.js) parar de consumir mensagens e o backlog começar a crescer."

**Output esperado (resumo):**

- Quick Reference com severidade sugerida (P1 — funcionalidade principal degradada) e tempo de resposta proposto
- Sintomas: alerta de backlog da fila acima de um limiar, latência de processamento de pedidos crescente
- Investigação: comandos para checar status dos workers, métricas de consumo da fila, logs de erro recentes
- Resolução: reiniciar workers travados, escalar consumidores, drenar mensagens em dead-letter queue
- Verificação: backlog voltando a zero, latência normalizada
- Template de comunicação para o canal de incidentes
- Checklist de post-mortem com campos para causa raiz e itens de ação
- Nota assinalando que contatos de escalonamento e nomes exatos dos workers foram deixados como placeholder por não terem sido informados
