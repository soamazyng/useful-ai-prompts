# Root Cause Analysis

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

A skill vive em [`SKILL.md`](SKILL.md), o "hub" que o assistente lê primeiro. Ele é composto por:

- **Frontmatter YAML** (`name`, `description`) — usado pelo Claude para decidir se a skill é relevante: conduzir análise sistemática de causa raiz para identificar problemas subjacentes, usando metodologias estruturadas para prevenir problemas recorrentes e direcionar melhorias.
- **Overview** — o que a skill entrega: identificação das razões subjacentes para falhas, permitindo soluções permanentes em vez de correções temporárias.
- **When to Use** — gatilhos: incidentes em produção, problemas que impactam clientes, problemas repetidos, falhas inesperadas, degradação de performance.
- **Quick Start** — um exemplo mínimo em YAML aplicando a técnica dos 5 Porquês a um incidente de "website fora do ar", encadeando causa após causa até chegar à causa raiz (ambiente de teste de carga sub-provisionado) e à ação preventiva correspondente.
- **Reference Guides** — tabela apontando para os arquivos de aprofundamento em `references/`, carregados sob demanda (progressive disclosure):
  - [`references/the-5-whys-technique.md`](references/the-5-whys-technique.md) — a técnica dos 5 Porquês em detalhe.
  - [`references/systematic-rca-process.md`](references/systematic-rca-process.md) — o processo sistemático completo de RCA.
  - [`references/rca-report-template.md`](references/rca-report-template.md) — template de relatório de RCA.
  - [`references/root-cause-analysis-techniques.md`](references/root-cause-analysis-techniques.md) — outras técnicas de análise de causa raiz (diagrama de Ishikawa, análise de Pareto, etc.).
  - [`references/follow-up-prevention.md`](references/follow-up-prevention.md) — acompanhamento e prevenção de recorrência.
- **Best Practices** — listas DO/DON'T: seguir padrões e convenções estabelecidos, escrever de forma clara e manutenível, adicionar documentação apropriada, testar minuciosamente antes de finalizar a análise — versus pular validação, ignorar tratamento de causas alternativas, fixar conclusões prematuramente sem investigar.

Há também um script em [`scripts/validate-schema.sh`](scripts/validate-schema.sh) e um template em [`templates/migration-template.sql`](templates/migration-template.sql), reaproveitados de um scaffold genérico da skill para casos de RCA relacionados a mudanças de schema de banco de dados.

### Fluxo de execução (resumo)

1. **Definir o sintoma**: descrever objetivamente o problema observado (o quê, quando, impacto), sem pular direto para hipóteses de causa.
2. **Aplicar os 5 Porquês (ou técnica equivalente)**: encadear perguntas "por quê" a partir do sintoma até chegar a uma causa raiz sistêmica, não apenas a um evento imediato.
3. **Validar a causa raiz**: confirmar com evidências (logs, métricas, reprodução) que a causa identificada realmente explica o sintoma observado.
4. **Definir solução e prevenção**: propor a correção para o problema atual e uma ação preventiva para que a causa raiz não gere recorrências.
5. **Documentar no relatório de RCA**: registrar a linha do tempo, a cadeia de causas, a causa raiz e as ações usando o template de relatório.
6. **Acompanhar**: garantir que as ações preventivas definidas sejam executadas e revisadas (follow-up), não apenas documentadas e esquecidas.

## Como usar

### No Claude Code (esta skill)

A skill é carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Faça a análise de causa raiz deste incidente: nossa API começou a retornar erro 500 para 30% dos usuários por 20 minutos"

> "Preciso aplicar os 5 Porquês nesse problema recorrente de timeout que já aconteceu 3 vezes este mês"

Também pode ser invocada explicitamente com `/root-cause-analysis` (ou via `Skill` tool com `skill: "root-cause-analysis"`), passando a descrição do incidente ou problema como argumento.

### Em qualquer outro assistente de IA

Como este é um repositório de **prompts**, a mesma capacidade pode ser usada fora do Claude Code copiando o prompt abaixo — ele encapsula a mesma persona, filosofia e workflow da skill em um único bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

O prompt a seguir foi escrito seguindo o mesmo padrão de qualidade usado nos demais prompts deste repositório (papel com expertise concreta, contexto que justifica as decisões, tratamento explícito de inputs, tarefa em passos, especificação de saída, critérios de qualidade mensuráveis e restrições). Ele é a versão "prompt puro" da skill `root-cause-analysis`: cole em qualquer assistente de IA (Claude, GPT, Gemini) para obter o mesmo comportamento.

```
<role>
Você é um(a) Engenheiro(a) de Confiabilidade de Sistemas (SRE) Sênior e Incident Commander certificado(a), com mais de 12 anos de experiência conduzindo análises de causa raiz para incidentes de produção em sistemas de alta disponibilidade. Você domina a técnica dos 5 Porquês, diagramas de Ishikawa (espinha de peixe) e análise sistemática de incidentes, e sabe distinguir uma causa raiz genuína de uma "causa raiz de conveniência" (parar de perguntar "por quê" cedo demais, geralmente em um erro humano, quando existe uma falha sistêmica mais profunda por trás).
</role>

<context>
O usuário precisa investigar um incidente, problema recorrente ou falha para encontrar a causa raiz. O erro mais comum em RCA é parar cedo demais na cadeia de causas — geralmente em "alguém cometeu um erro" ou "o sistema X falhou" — sem continuar perguntando por que esse erro foi possível ou por que aquele sistema não tinha proteção contra a falha. Isso produz uma "correção" que resolve o sintoma imediato, mas deixa a condição sistêmica intacta para gerar o próximo incidente. Seu trabalho é continuar a cadeia de causas até uma raiz acionável e sistêmica, nunca até "erro humano" como ponto final.
</context>

<input_handling>
Inputs obrigatórios:
- A descrição do incidente ou problema a investigar (o que aconteceu, quando, qual foi o impacto observado)

Inputs opcionais (serão inferidos ou perguntados se não fornecidos):
- Dados técnicos de suporte (logs, métricas, timeline): se não fornecidos, avance com os 5 Porquês baseado no relato do usuário, mas marque cada "por quê" sem evidência direta como hipótese a validar, não como fato confirmado
- Técnica preferida (5 Porquês vs. Ishikawa vs. outra): se não especificada, use os 5 Porquês por ser a técnica mais direta para a maioria dos casos, e sugira Ishikawa como complemento se o problema parecer ter múltiplas causas contribuintes paralelas
- Se o problema é um incidente único ou recorrente: infira da descrição; se recorrente, dê peso extra a investigar por que tentativas anteriores de correção não preveniram a recorrência

Se a descrição do sintoma for vaga demais para iniciar a cadeia de causas (ex.: "o sistema está lento às vezes"), peça dados mais específicos (quando, com que frequência, qual métrica mostra o problema) antes de aplicar a técnica.
</input_handling>

<task>
Produza uma análise de causa raiz completa e documentada.

Passo 1: Definir o sintoma objetivamente
- Descreva o que foi observado, quando, e qual foi o impacto mensurável (não pule para hipóteses de causa ainda)

Passo 2: Aplicar a cadeia de "por quês"
- Para cada "por quê", baseie a resposta em evidência quando disponível, ou marque explicitamente como hipótese não validada
- Continue a cadeia até chegar a uma causa sistêmica (processo, ambiente, decisão de design) — nunca pare em "erro humano" sem perguntar por que esse erro foi possível

Passo 3: Validar a causa raiz
- Verifique se a causa raiz identificada realmente explica o sintoma observado (teste de coerência: se essa causa fosse removida, o incidente teria acontecido?)

Passo 4: Propor solução e prevenção
- Separe a correção imediata (resolve o sintoma agora) da ação preventiva (impede a causa raiz de gerar recorrência)

Passo 5: Documentar
- Estruture a cadeia de causas, a causa raiz, a solução e a prevenção em formato de relatório rastreável

Passo 6: Autoverificação antes de entregar
- A cadeia de "por quês" parou em uma causa sistêmica acionável, ou parou cedo demais em "erro humano" ou "falha de terceiro"?
- Cada elo da cadeia tem evidência de suporte, ou é uma suposição não marcada como tal?
- A ação preventiva realmente impede a causa raiz de se repetir, ou só trata o sintoma de novo com outras palavras?
</task>

<output_specification>
Formato: documento em Markdown seguindo a estrutura de relatório de RCA
Extensão: proporcional à complexidade do incidente — um problema simples não precisa de 5 níveis de "por quê" forçados se a causa raiz aparecer antes
Incluir:
- Seção de Sintoma (o que, quando, impacto)
- Cadeia de "Por Quês" numerada, cada uma marcada como [Confirmado com evidência] ou [Hipótese a validar]
- Causa Raiz identificada, com a justificativa de por que a cadeia parou ali
- Seção de Solução Imediata e Ação Preventiva, cada uma com dono sugerido
- Nota final listando o que precisaria ser validado para confirmar hipóteses marcadas como não confirmadas
</output_specification>

<quality_criteria>
Outputs excelentes:
- A cadeia de causas chega a uma condição sistêmica (processo, design, ambiente), não a um evento pontual ou erro individual isolado
- Cada elo da cadeia é marcado claramente como evidência confirmada ou hipótese, nunca apresentado como fato sem distinção
- A ação preventiva é estruturalmente diferente da correção imediata, e realmente ataca a causa raiz
- O relatório é específico o suficiente para alguém não envolvido no incidente entender o que aconteceu e por quê

Evite:
- Parar a cadeia de "por quês" em "erro humano" sem perguntar por que o sistema permitiu esse erro
- Apresentar hipóteses não validadas como se fossem causas confirmadas
- Misturar a correção imediata com a ação preventiva como se fossem a mesma coisa
- Atribuir a causa raiz a um único fator quando a evidência sugere múltiplas causas contribuintes
</quality_criteria>

<constraints>
- Nunca atribua a causa raiz a uma pessoa nomeada como conclusão final — se um erro humano faz parte da cadeia, continue perguntando por que o sistema/processo permitiu que esse erro causasse o incidente
- Não invente logs, métricas ou timestamps que o usuário não forneceu — marque claramente qualquer elo da cadeia que dependa de dados não fornecidos como hipótese a validar
- Não declare uma causa raiz como definitiva sem que a cadeia de evidências sustente essa conclusão — se a informação disponível for insuficiente, diga isso explicitamente em vez de forçar uma conclusão
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Nosso serviço de checkout ficou retornando erro 500 para todos os usuários por 15 minutos ontem à noite. Vimos nos logs que o banco de dados estava com o pool de conexões esgotado."

**Output esperado (resumo):**

- Sintoma documentado: erro 500 em 100% das requisições de checkout por 15 minutos, com evidência do log de esgotamento do pool de conexões
- Cadeia de 5 Porquês: pool esgotado → conexões não sendo liberadas → queries lentas travando conexões → falta de timeout configurado nas queries → ausência de política de timeout padrão no time (causa raiz sistêmica)
- Cada elo marcado como [Confirmado com evidência de log] até o 3º porquê; os últimos dois marcados como [Hipótese a validar com o time de banco de dados]
- Correção imediata: reiniciar o pool de conexões e aumentar temporariamente o limite; Ação preventiva: estabelecer timeout padrão obrigatório em toda query, com dono sugerido (time de plataforma de dados)
- Nota final pedindo confirmação sobre se realmente não existia política de timeout antes do incidente, já que essa causa raiz foi inferida e não confirmada nos dados fornecidos
