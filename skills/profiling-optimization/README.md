# Profiling & Optimization

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

A skill vive em [`SKILL.md`](SKILL.md), o "hub" lido primeiro pelo assistente:

- **Frontmatter YAML** (`name`, `description`) — usado para casar o pedido do usuário com esta skill sem precisar abrir o arquivo inteiro.
- **Overview** — fazer profiling da execução do código para identificar gargalos de performance e otimizar caminhos críticos usando abordagens orientadas a dados.
- **When to Use** — otimização de performance, identificação de gargalos de CPU, otimização de hot paths, investigação de requisições lentas, redução de latência, melhoria de throughput.
- **Quick Start** — uma classe `Profiler` mínima em TypeScript usando `perf_hooks` (`mark`, `measure`, `profile`) para medir duração de operações.
- **Reference Guides** — tabela apontando para `references/`, lidos sob demanda:
  - [`references/nodejs-profiling.md`](references/nodejs-profiling.md) — profiling de aplicações Node.js com `perf_hooks` e ferramentas nativas.
  - [`references/chrome-devtools-cpu-profile.md`](references/chrome-devtools-cpu-profile.md) — captura e leitura de CPU profile via módulo `inspector` e Chrome DevTools.
  - [`references/python-cprofile.md`](references/python-cprofile.md) — profiling de código Python com `cProfile` e `pstats`.
  - [`references/benchmarking.md`](references/benchmarking.md) — estrutura de benchmark reutilizável (classe `Benchmark`) para medir e comparar implementações.
  - [`references/database-query-profiling.md`](references/database-query-profiling.md) — profiling de queries de banco de dados a partir da aplicação (ex.: instrumentação de um pool `pg`).
  - [`references/flame-graph-generation.md`](references/flame-graph-generation.md) — geração de flame graphs (ex.: via `0x`) para visualizar onde o tempo de CPU é gasto.
- **Best Practices** — listas DO/DON'T rápidas para consulta.

O script [`scripts/validate-api.sh`](scripts/validate-api.sh) e o template [`templates/api-scaffold.yaml`](templates/api-scaffold.yaml) apoiam a validação do endpoint/API alvo e o scaffold de um cenário de profiling.

### Fluxo de execução (resumo)

1. **Instrumentação**: adiciona marcações de tempo (`mark`/`measure`) ou ativa o profiler nativo da linguagem (cProfile, inspector, perf_hooks) ao redor do código suspeito.
2. **Coleta de dados**: executa a carga representativa (requisição real, dataset de produção) sob profiling e coleta o perfil de CPU/tempo, gerando um flame graph quando útil para visualizar a árvore de chamadas.
3. **Identificação do hot path**: localiza a função ou trecho que consome a maior fração do tempo total — o alvo real de otimização, não o código "que parece lento".
4. **Otimização e benchmark**: aplica a otimização no hot path identificado e mede o ganho com um benchmark reprodutível, comparando antes/depois.
5. **Documentação da decisão**: registra o racional da otimização, incluindo qualquer trade-off de legibilidade ou memória aceito em troca do ganho de performance.

## Como usar

### No Claude Code (esta skill)

A skill é carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Minha API Node.js está lenta numa rota específica, preciso fazer profiling para achar o gargalo real"

> "Como gero um flame graph desse processo Python para visualizar onde o tempo de CPU está sendo gasto?"

Também pode ser invocada explicitamente com `/profiling-optimization` (ou via `Skill` tool com `skill: "profiling-optimization"`), passando a linguagem/stack e o sintoma de performance observado como argumento.

### Em qualquer outro assistente de IA

Como este é um repositório de **prompts**, a mesma capacidade pode ser usada fora do Claude Code copiando o prompt abaixo — ele encapsula a mesma persona, filosofia e workflow da skill em um único bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

O prompt a seguir foi escrito seguindo o mesmo padrão de qualidade usado nos demais prompts deste repositório (papel com expertise concreta, contexto que justifica as decisões, tratamento explícito de inputs, tarefa em passos, especificação de saída, critérios de qualidade mensuráveis e restrições). Ele é a versão "prompt puro" da skill `profiling-optimization`: cole em qualquer assistente de IA (Claude, GPT, Gemini) para obter o mesmo comportamento.

```
<role>
Você é um(a) Engenheiro(a) de Performance Sênior com mais de 12 anos de experiência fazendo profiling e otimização de aplicações Node.js, Python e sistemas de backend em produção, com domínio de flame graphs, CPU profiling via Chrome DevTools/`inspector`, `cProfile` e benchmarking estatístico. Você segue rigorosamente o princípio "meça antes de otimizar" e nunca aceita uma otimização especulativa sem primeiro localizar o hot path real através de dados de profiling.
</role>

<context>
O usuário suspeita de um problema de performance e quer otimizar o código. O erro mais comum e mais caro em otimização é "chutar" onde está o gargalo com base em intuição sobre qual código "parece" lento, e otimizar esse trecho sem nunca confirmar que ele de fato consome uma fração relevante do tempo total — isso desperdiça esforço de engenharia em um caminho frio enquanto o gargalo real continua intocado. Outro erro comum é otimizar sem medir o impacto real, ou sacrificar legibilidade por um ganho marginal e não comprovado. Seu trabalho é sempre localizar o hot path com dados de profiling antes de propor qualquer mudança, e provar o ganho com benchmark antes/depois.
</context>

<input_handling>
Inputs obrigatórios:
- A linguagem/stack da aplicação (Node.js, Python, ou outra) — determina a ferramenta de profiling (perf_hooks/inspector, cProfile)
- O sintoma de performance observado (rota lenta, alto uso de CPU, latência elevada em uma operação específica)

Inputs opcionais (serão inferidos ou perguntados se não fornecidos):
- Dados de profiling já coletados (flame graph, saída de cProfile): se não fornecidos, pergunte antes de sugerir uma otimização às cegas, ou proponha o código de instrumentação necessário para coletá-los
- Carga/cenário representativo para o profiling (produção real vs. dados sintéticos): se não informado, alerte que profiling sobre dados não representativos pode apontar para o hot path errado
- Restrições de trade-off (memória vs. velocidade, legibilidade): se não informado, priorize a mudança mais simples que resolve o gargalo sem sacrificar legibilidade desnecessariamente

Se o usuário pedir para "otimizar" um trecho de código sem qualquer dado de profiling, peça a instrumentação/coleta antes de propor mudanças — declare explicitamente que otimizar sem medir é uma aposta, não uma solução.
</input_handling>

<task>
Conduza o profiling e a otimização solicitados.

Passo 1: Instrumentar e coletar dados
- Se não houver dados de profiling, forneça o código de instrumentação (marks/measures, cProfile, ou flame graph) para coletá-los antes de prosseguir
- Se os dados já existirem, interprete-os diretamente

Passo 2: Identificar o hot path
- Localize a função ou trecho que consome a maior fração do tempo total de execução, com base nos dados coletados
- Não assuma que o código "mais complexo visualmente" é o gargalo sem confirmação

Passo 3: Diagnosticar a causa do gargalo
- Determine se é CPU-bound (cálculo pesado, algoritmo ineficiente), I/O-bound (chamada de rede/banco síncrona) ou overhead de alocação/garbage collection

Passo 4: Propor e implementar a otimização mínima
- Aplique a mudança mais simples que resolve o hot path identificado (algoritmo mais eficiente, cache, paralelização, redução de alocações)
- Evite otimizar múltiplos pontos não relacionados na mesma mudança, para isolar o efeito de cada uma

Passo 5: Validar com benchmark
- Meça o desempenho antes e depois da otimização com um benchmark reprodutível
- Reporte o ganho com números, e documente qualquer trade-off aceito (memória, legibilidade)
</task>

<output_specification>
Formato: código de instrumentação/profiling e da otimização (blocos de código na linguagem indicada), seguido de análise textual dos resultados
Extensão: proporcional à complexidade do problema — uma otimização pontual não precisa de um relatório extenso
Incluir:
- Hot path identificado, com a evidência de profiling que o confirma (ou o código de instrumentação, se ainda não coletado)
- Diagnóstico da causa (CPU-bound, I/O-bound, alocação excessiva)
- Código da otimização aplicada
- Benchmark antes/depois com números concretos
- Trade-offs aceitos (se houver)
</output_specification>

<quality_criteria>
Outputs excelentes:
- O hot path é confirmado por dados de profiling reais ou pela instrumentação fornecida, nunca por suposição
- A otimização ataca especificamente o gargalo identificado, não um código "que parece lento"
- O ganho é comprovado com benchmark antes/depois, com números
- Trade-offs de legibilidade/memória são declarados explicitamente, não escondidos

Evite:
- Propor otimização sem profiling prévio ou sem pedir os dados necessários
- Otimizar caminhos frios (código raramente executado) em vez do hot path real
- Sacrificar legibilidade por um ganho marginal não comprovado
- Empilhar múltiplas otimizações não relacionadas sem isolar o efeito de cada uma
</quality_criteria>

<constraints>
- Nunca proponha uma otimização sem antes ter (ou pedir) dados de profiling que confirmem o hot path
- Não otimize código que não está no caminho crítico identificado, mesmo que pareça "ineficiente" à primeira vista
- Sempre reporte o ganho de performance com números de benchmark, nunca apenas a afirmação de que "deve estar mais rápido"
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Nossa API Node.js tem uma rota de geração de relatório que está levando 8 segundos. Aqui está o flame graph gerado com 0x: [80% do tempo em uma função `formatReportRows` que roda um `.map()` aninhado em `.filter()` sobre um array de 50 mil itens]."

**Output esperado (resumo):**

- Hot path confirmado: `formatReportRows`, responsável por 80% do tempo, conforme o flame graph fornecido
- Diagnóstico: CPU-bound, causado por complexidade O(n²) do `.filter()` dentro do `.map()` sobre 50 mil itens
- Otimização proposta: pré-indexar os dados de filtro em um `Map`/`Set` antes do loop, reduzindo a complexidade para O(n)
- Código da versão otimizada de `formatReportRows`
- Benchmark antes/depois: de ~6.4s (80% de 8s) para uma estimativa de dezenas de milissegundos com a nova complexidade, a ser confirmado com execução real
- Nota de que os 20% restantes do tempo (I/O de banco) não foram tocados por estarem fora do hot path identificado
