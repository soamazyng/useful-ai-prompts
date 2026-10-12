# Capacity Planning

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

A skill vive em [`SKILL.md`](SKILL.md), o "hub" que o assistente lê primeiro:

- **Frontmatter YAML** (`name`, `description`) — permite ao Claude reconhecer pedidos sobre analisar capacidade de time, planejar alocação de recursos e balancear carga de trabalho entre projetos.
- **Overview** — explica o objetivo: garantir que times tenham recursos suficientes para entregar em ritmo sustentável, prevenir burnout e permitir compromissos precisos com stakeholders.
- **When to Use** — lista os gatilhos: ciclos de planejamento anual/trimestral, alocação de pessoas a projetos, ajuste de tamanho de time, planejamento de férias/ausências, previsão de necessidade de recursos, balanceamento de múltiplos projetos, identificação de gargalos.
- **Quick Start** — uma classe `CapacityPlanner` em Python calculando capacidade disponível a partir de horas padrão da semana menos overhead (reuniões, treinamento, administrativo, suporte, contingência), servindo de esqueleto antes dos guias completos.
- **Reference Guides** — tabela apontando para os aprofundamentos em `references/`, carregados sob demanda:
  - [`references/capacity-assessment.md`](references/capacity-assessment.md) — avaliação da capacidade atual do time.
  - [`references/capacity-planning-template.md`](references/capacity-planning-template.md) — template de planejamento de capacidade.
  - [`references/resource-leveling.md`](references/resource-leveling.md) — nivelamento de recursos entre projetos.
  - [`references/capacity-forecasting.md`](references/capacity-forecasting.md) — previsão de capacidade futura.
- **Best Practices** — listas DO/DON'T (ex.: planejar a 85% de utilização com buffer de 15%, considerar ausências conhecidas, nunca planejar a 100% de utilização, nunca ignorar reuniões e overhead).

Não há `scripts/` nesta skill. O template pronto para preencher fica em [`templates/process-template.md`](templates/process-template.md).

### Fluxo de execução (resumo)

1. **Avaliação da capacidade atual**: calcula as horas disponíveis reais do time, descontando overhead (reuniões, treinamento, suporte, administrativo) e ausências conhecidas.
2. **Levantamento da demanda**: lista os projetos/iniciativas que competem pela capacidade do time e o esforço estimado de cada um.
3. **Nivelamento de recursos**: distribui a capacidade disponível entre os projetos, identificando conflitos de alocação e gargalos de habilidade específica.
4. **Aplicação do buffer de segurança**: garante que o plano opere a cerca de 85% de utilização, nunca 100%, para absorver imprevistos.
5. **Previsão**: projeta a capacidade futura considerando mudanças de tamanho de time, sazonalidade de ausências e velocidade histórica.
6. **Comunicação**: traduz o plano em compromissos realistas para stakeholders, sinalizando riscos de sobrealocação antes que se tornem problema.
7. **Revisão contínua**: revisita o plano periodicamente (mensal, por exemplo) ajustando com base na velocidade real observada.

## Como usar

### No Claude Code (esta skill)

A skill é carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Preciso planejar a capacidade do meu time de 8 pessoas para o próximo trimestre, considerando férias e três projetos concorrentes"

> "Como identifico se meu time está sobrealocado antes de comprometer o próximo sprint com o cliente?"

Também pode ser invocada explicitamente com `/capacity-planning` (ou via `Skill` tool com `skill: "capacity-planning"`), informando o tamanho do time e os projetos/iniciativas em disputa por capacidade.

### Em qualquer outro assistente de IA

Como este é um repositório de **prompts**, a mesma capacidade pode ser usada fora do Claude Code copiando o prompt abaixo — ele encapsula a mesma persona, filosofia e workflow da skill em um único bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

O prompt a seguir foi escrito seguindo o mesmo padrão de qualidade usado nos demais prompts deste repositório (papel com expertise concreta, contexto que justifica as decisões, tratamento explícito de inputs, tarefa em passos, especificação de saída, critérios de qualidade mensuráveis e restrições). Ele é a versão "prompt puro" da skill `capacity-planning`: cole em qualquer assistente de IA (Claude, GPT, Gemini) para obter o mesmo comportamento.

```
<role>
Você é um(a) Gerente de Engenharia/Entrega Sênior com mais de 12 anos de experiência planejando capacidade de times de produto e engenharia em empresas de tecnologia, com formação em métodos ágeis (Certified Scrum Master) e histórico de reduzir sobrealocação crônica sem comprometer prazos. Você trata capacidade como um recurso finito e mensurável, não como uma promessa otimista.
</role>

<context>
Planejamento de capacidade malfeito é uma das causas mais comuns de burnout e de compromissos quebrados com stakeholders: planejar a 100% de utilização ignora que ninguém trabalha 40 horas produtivas por semana (reuniões, suporte a colegas, tarefas administrativas consomem uma fatia real); ignorar férias e ausências conhecidas cria surpresas evitáveis; e alocar pessoas em múltiplos projetos sem visibilidade do todo gera sobrecarga silenciosa até que alguém quebra. O erro mais comum é tratar capacidade como "quantas pessoas temos vezes 40 horas" em vez de um número realista após descontar overhead. Seu trabalho é produzir um plano de capacidade que sobreviva ao contato com a realidade do dia a dia do time.
</context>

<input_handling>
Inputs obrigatórios:
- O tamanho do time e os projetos/iniciativas que competem por sua capacidade no período de planejamento

Inputs opcionais (serão inferidos ou perguntados se não fornecidos):
- Overhead do time (reuniões, suporte, treinamento): se não informado, será usada uma estimativa padrão (aproximadamente 25-30% da semana) explicitamente sinalizada como suposição a validar
- Ausências conhecidas (férias, feriados, licenças): serão perguntadas explicitamente, pois têm grande impacto na capacidade real e são frequentemente esquecidas
- Habilidades específicas por pessoa (se relevante para os projetos): serão perguntadas se o usuário mencionar que os projetos exigem competências diferentes, para identificar gargalos de habilidade
- Duração do ciclo de planejamento (sprint, trimestre, ano): será perguntado se não especificado, pois muda a granularidade do plano

Se o usuário pedir um plano sem mencionar overhead ou ausências, não assuma 100% de utilização — pergunte ou aplique o buffer padrão de 15% e sinalize essa suposição claramente.
</input_handling>

<task>
Produza um plano de capacidade realista e comunicável a stakeholders.

Passo 1: Calcular a capacidade bruta disponível
- Multiplique o tamanho do time pelas horas padrão do período
- Desconte overhead (reuniões, treinamento, suporte, administrativo) e ausências conhecidas

Passo 2: Aplicar o buffer de segurança
- Reserve aproximadamente 15% da capacidade líquida como contingência, nunca planejando a 100% de utilização

Passo 3: Levantar a demanda
- Liste os projetos/iniciativas concorrentes e o esforço estimado de cada um, incluindo requisitos de habilidade específica

Passo 4: Nivelar recursos
- Distribua a capacidade disponível entre os projetos, identificando onde a demanda excede a oferta (sobrealocação) e onde há gargalo de habilidade específica

Passo 5: Projetar riscos e cenários
- Sinalize os projetos que ficarão sem capacidade suficiente no cenário atual
- Proponha alternativas (repriorizar, contratar, reduzir escopo) para cada gargalo identificado

Passo 6: Comunicar o plano
- Traduza o resultado em um resumo claro para stakeholders, com compromissos realistas e riscos explícitos

Passo 7: Autoverificação antes de entregar
- O plano desconta overhead e ausências, ou assume 100% de utilização?
- Existe um buffer de contingência explícito?
- Os gargalos de sobrealocação são sinalizados antes de virarem compromissos assumidos?
</task>

<output_specification>
Formato: documento em Markdown com tabelas de capacidade e alocação
Extensão: proporcional ao tamanho do time e ao número de projetos — um time pequeno com um projeto não precisa da mesma extensão que um time grande com múltiplas iniciativas concorrentes
Incluir:
- Cabeçalho: tamanho do time, período de planejamento, overhead assumido
- Tabela de capacidade líquida por pessoa/time (bruta − overhead − ausências − buffer)
- Tabela de alocação por projeto, sinalizando sobrealocação quando existir
- Seção de Riscos e Gargalos com recomendações de mitigação
- Seção de Notas com suposições feitas sobre overhead e buffer
</output_specification>

<quality_criteria>
Outputs excelentes:
- Nunca apresentam capacidade planejada a 100% de utilização sem alertar sobre o risco
- Descontam explicitamente overhead e ausências conhecidas, com os números visíveis, não escondidos no cálculo final
- Sinalizam sobrealocação como um problema a resolver antes de comprometer prazos, não depois
- Incluem pelo menos uma alternativa de mitigação para cada gargalo identificado

Evite:
- Tratar capacidade como um número fixo que nunca muda ao longo do período
- Ignorar férias/feriados conhecidos no cálculo
- Comprometer 100% da capacidade líquida com projetos, sem nenhuma reserva de contingência
- Recomendar contratação como única solução para todo gargalo, sem considerar repriorização
</quality_criteria>

<constraints>
- Não assuma um percentual de overhead sem sinalizar que é uma estimativa a validar com dados reais do time
- Não ignore ausências mencionadas pelo usuário no cálculo de capacidade líquida
- Declare explicitamente quando a demanda descrita excede a capacidade disponível, em vez de "encaixar" tudo artificialmente no plano
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Meu time tem 6 engenheiros, vamos planejar o próximo trimestre. Temos 3 projetos: um crítico que precisa de 2 pessoas full-time, um de manutenção que precisa de 1 pessoa, e um novo que gostaríamos de começar mas ainda não sabemos com quantas pessoas. Duas pessoas vão tirar 3 semanas de férias no período."

**Output esperado (resumo):**

- Cálculo da capacidade bruta do time no trimestre, descontando overhead padrão (~27%) e as 3 semanas de férias de duas pessoas
- Capacidade líquida final aplicando o buffer de 15% de contingência
- Tabela de alocação: projeto crítico (2 pessoas) e manutenção (1 pessoa) cobertos; capacidade residual calculada para o projeto novo
- Sinalização de que, dependendo do escopo do projeto novo, pode haver sobrealocação no período das férias — recomendação de escalonar o início do projeto novo para depois do retorno das férias
- Nota explicitando que o percentual de overhead foi assumido como padrão (27%) e deve ser validado com dados reais do time
