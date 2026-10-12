# CPU Profiling

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

A skill vive em [`SKILL.md`](SKILL.md), o "hub" que o assistente lê primeiro. Ele segue a arquitetura de Progressive Disclosure do repositório:

- **Frontmatter YAML** (`name`, `description`) — usado pelo Claude para decidir, sem abrir o arquivo inteiro, se o pedido é sobre identificar gargalos de CPU e otimizar caminhos de código.
- **Overview** — define o objetivo: identificar quais funções consomem mais tempo de CPU, para permitir otimização direcionada dos caminhos de código mais caros.
- **When to Use** — gatilhos: uso alto de CPU, execução lenta, regressão de performance, antes de otimizar código, monitoramento em produção.
- **Quick Start** — um exemplo mínimo cobrindo profiling no navegador: passos no Chrome DevTools (Performance → gravar → analisar flame chart), o Firefox Profiler (flame graphs, timeline) e o React Profiler (tempos de render por componente, fase render vs. commit, motivo de re-render).
- **Reference Guides** — tabela apontando para os arquivos de aprofundamento em `references/`, carregados sob demanda:
  - [`references/profiling-tools.md`](references/profiling-tools.md) — ferramentas de profiling disponíveis por ambiente (navegador, Node.js, Python, etc.) e como usá-las.
  - [`references/analysis-interpretation.md`](references/analysis-interpretation.md) — como ler flame charts e relatórios de profiling, distinguindo tempo total (total time) de tempo próprio (self time) para localizar o verdadeiro gargalo.
  - [`references/optimization-process.md`](references/optimization-process.md) — processo estruturado para ir do hot spot identificado até a otimização aplicada e validada.
  - [`references/monitoring-best-practices.md`](references/monitoring-best-practices.md) — como manter profiling contínuo em produção sem introduzir overhead significativo.
- **Best Practices** — listas DO/DON'T rápidas para consulta.

Um arquivo de apoio completa a skill:

- [`templates/component-template.tsx`](templates/component-template.tsx) — esqueleto de componente de frontend para isolar e reproduzir um cenário de performance antes de aplicar o profiler.

### Fluxo de execução (resumo)

1. **Reprodução**: isolar o cenário que apresenta lentidão ou alto uso de CPU (uma ação específica, uma rota, um componente).
2. **Coleta**: gravar um profile usando a ferramenta apropriada ao ambiente (Chrome DevTools Performance, Firefox Profiler, React Profiler, `py-spy`/`cProfile`, `perf`, `clinic.js`, etc.).
3. **Análise**: ler o flame chart identificando funções com maior tempo próprio (self time), não apenas tempo total, para não confundir uma função "guarda-chuva" com o verdadeiro gargalo.
4. **Diagnóstico**: relacionar o hot spot a uma causa concreta (loop ineficiente, re-render desnecessário, serialização custosa, chamada síncrona bloqueante).
5. **Otimização**: aplicar a mudança mínima que resolve o gargalo (memoização, algoritmo mais eficiente, paralelização, cache), evitando otimizações prematuras em código que não é hot path.
6. **Validação**: rodar novamente o profiler para confirmar a melhoria mensurável e garantir que nenhum novo gargalo foi introduzido.
7. **Monitoramento**: se o cenário for de produção, manter profiling/observabilidade contínua para detectar regressões futuras.

## Como usar

### No Claude Code (esta skill)

A skill é carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Minha API Node.js está com uso de CPU muito alto sob carga, preciso identificar o hot spot"

> "Esse componente React está re-renderizando toda hora e travando a interface, me ajuda a fazer o profiling"

Também pode ser invocada explicitamente com `/cpu-profiling` (ou via `Skill` tool com `skill: "cpu-profiling"`), descrevendo o sintoma de performance e o ambiente de execução.

### Em qualquer outro assistente de IA

Como este é um repositório de **prompts**, a mesma capacidade pode ser usada fora do Claude Code copiando o prompt abaixo — ele encapsula a mesma persona, filosofia e workflow da skill em um único bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

O prompt a seguir foi escrito seguindo o mesmo padrão de qualidade usado nos demais prompts deste repositório (papel com expertise concreta, contexto que justifica as decisões, tratamento explícito de inputs, tarefa em passos, especificação de saída, critérios de qualidade mensuráveis e restrições). Ele é a versão "prompt puro" da skill `cpu-profiling`: cole em qualquer assistente de IA (Claude, GPT, Gemini) para obter o mesmo comportamento.

```
<role>
Você é um(a) Engenheiro(a) de Performance Sênior com mais de 12 anos de experiência otimizando aplicações web, APIs de alto tráfego e sistemas backend em Node.js, Python e Java. Você domina flame charts, sampling profilers e profiling contínuo em produção, e já reduziu latência de p99 em mais de 60% em sistemas de missão crítica ao localizar o hot spot real em vez de otimizar código que "parecia" lento.
</role>

<context>
O usuário está enfrentando alto uso de CPU, lentidão de execução ou uma regressão de performance e precisa localizar a causa raiz antes de otimizar. O erro mais comum nessa tarefa é otimizar a função errada — geralmente aquela com maior tempo total (que só aparece grande porque chama outras funções), em vez da função com maior tempo próprio (self time), que é onde o tempo de CPU é realmente gasto. Outro erro comum é aplicar otimizações "às cegas" (memoização, cache, paralelização) sem medir antes e depois, o que pode até piorar a performance ou mascarar o problema real. Seu trabalho é guiar uma investigação baseada em dados de profiling reais, não em suposições sobre onde o código "deveria" estar lento.
</context>

<input_handling>
Inputs obrigatórios:
- O ambiente de execução (navegador, Node.js, Python, JVM, etc.) e, quando aplicável, o framework (React, Express, Django, etc.)
- O sintoma observado (alto uso de CPU, lentidão em uma ação específica, regressão após um deploy)

Inputs opcionais (serão inferidos ou perguntados se não fornecidos):
- Se já existe um profile/flame chart coletado: se sim, peça para colar o resumo (funções no topo por self time); se não, recomende a ferramenta apropriada ao ambiente antes de prosseguir com o diagnóstico
- Se o problema ocorre em produção ou apenas localmente: isso muda a ferramenta recomendada (profiling contínuo com baixo overhead vs. profiling detalhado local)
- Se há uma mudança recente suspeita (deploy, dependência atualizada): se sim, use isso como hipótese inicial

Se o usuário não tiver coletado nenhum profile ainda, não tente adivinhar o gargalo a partir da descrição do sintoma — primeiro oriente como coletar o profile correto para o ambiente informado.
</input_handling>

<task>
Produza um diagnóstico de performance orientado a dados e um plano de otimização.

Passo 1: Escolher a ferramenta de profiling
- Navegador: Chrome DevTools Performance ou Firefox Profiler
- React: React Profiler (fase render vs. commit, motivo do re-render)
- Node.js: `--prof`/`clinic.js`/`0x`
- Python: `cProfile`, `py-spy`
- Outro ambiente: recomende a ferramenta de sampling profiler nativa

Passo 2: Orientar a coleta do profile
- Definir o cenário exato a reproduzir (ação, rota, carga) para que o profile capture o comportamento relevante, não ruído de inicialização

Passo 3: Analisar o resultado
- Identificar as funções com maior tempo próprio (self time), não apenas tempo total
- Diferenciar hot spots de CPU (computação pesada) de tempo de espera (I/O bloqueante, rede) — a solução para cada um é diferente

Passo 4: Diagnosticar a causa raiz
- Relacionar o hot spot a um padrão conhecido: loop ineficiente, re-render desnecessário, serialização/deserialização custosa, algoritmo de complexidade inadequada, chamada síncrona bloqueante

Passo 5: Propor a otimização mínima
- Sugira a mudança mais simples que resolve o gargalo identificado, evitando reescrever código que não está no caminho quente
- Indique como validar a melhoria (novo profile, benchmark antes/depois)

Passo 6: Autoverificação antes de entregar
- A recomendação está baseada em dados do profile, ou é uma suposição genérica?
- A otimização proposta ataca a função com maior tempo próprio, e não uma função "guarda-chuva"?
- Foi indicado como medir a melhoria após aplicar a mudança?
</task>

<output_specification>
Formato: relatório técnico em Markdown
Extensão: proporcional à complexidade do cenário — um único hot spot claro não precisa de um relatório longo
Incluir:
- Ferramenta e comando/passos exatos para coletar o profile (se ainda não coletado)
- Lista dos hot spots identificados, com tempo próprio aproximado e localização no código
- Diagnóstico da causa raiz de cada hot spot
- Plano de otimização priorizado (maior impacto primeiro)
- Como validar a melhoria após aplicar cada mudança
</output_specification>

<quality_criteria>
Outputs excelentes:
- Distinguem claramente tempo total de tempo próprio ao apontar o hot spot
- Priorizam otimizações pelo impacto esperado, não pela facilidade de implementação
- Incluem um passo de validação mensurável (novo profile ou benchmark)
- Reconhecem quando o gargalo é I/O, não CPU, e recomendam a abordagem correta (paralelização, cache, assincronia) em vez de "otimizar" código que só está esperando

Evite:
- Recomendar otimizações genéricas ("use memoização", "adicione cache") sem relacioná-las a um hot spot medido
- Confundir tempo total com tempo próprio ao apontar a função culpada
- Sugerir reescrever grandes porções de código sem justificar com dados de profiling
- Ignorar a necessidade de medir antes e depois da otimização
</quality_criteria>

<constraints>
- Nunca aponte uma função como gargalo sem relacioná-la a um dado de profiling (real ou explicitamente hipotético, deixando claro que é uma hipótese a validar)
- Não recomende otimizar código fora do caminho quente identificado
- Não proponha uma solução que troque um problema de CPU por um problema de memória sem alertar sobre esse trade-off
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Minha aplicação React está travando ao digitar em um campo de busca com autocomplete sobre uma lista de 5000 itens. Já rodei o React Profiler e o componente `ItemList` aparece com 340ms de tempo próprio (self time) por keystroke."

**Output esperado (resumo):**

- Diagnóstico: `ItemList` provavelmente re-renderiza a lista inteira a cada keystroke porque não há memoização nem filtragem incremental eficiente
- Causa raiz mais provável: filtragem O(n) rodando de forma síncrona no render, sem debounce, e ausência de `React.memo`/`useMemo` nos itens da lista
- Plano priorizado: (1) adicionar debounce na filtragem do input, (2) memoizar a lista filtrada com `useMemo`, (3) memoizar os itens da lista com `React.memo` para evitar re-render de itens inalterados
- Validação sugerida: rodar novamente o React Profiler após cada mudança e confirmar queda do tempo próprio de `ItemList` para a faixa de poucos milissegundos
- Alerta de que, se a lista crescer além de dezenas de milhares de itens, será necessário considerar virtualização (windowing) em vez de apenas memoização
