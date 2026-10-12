# Memory Optimization

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

A skill vive em [`SKILL.md`](SKILL.md), o "hub" lido primeiro pelo assistente:

- **Frontmatter YAML** (`name`, `description`) — usado para casar o pedido do usuário com esta skill sem precisar abrir o arquivo inteiro.
- **Overview** — otimização de memória melhora performance e estabilidade da aplicação e reduz custos de infraestrutura; uso eficiente de memória é crítico para escalabilidade.
- **When to Use** — alto uso de memória, suspeita de vazamento de memória, performance lenta, crashes por falta de memória, desafios de escalabilidade.
- **Quick Start** — três abordagens de profiling de memória no navegador: leitura de `performance.memory` (jsHeapSizeLimit/totalJSHeapSize/usedJSHeapSize), uso do Profiler do React DevTools para identificar re-renders desnecessários, e heap snapshots via Chrome DevTools comparando antes/depois.
- **Reference Guides** — tabela apontando para `references/`, lidos sob demanda:
  - [`references/memory-profiling.md`](references/memory-profiling.md) — técnicas de profiling de memória em diferentes ambientes (navegador, Node.js)
  - [`references/memory-leak-detection.md`](references/memory-leak-detection.md) — como identificar vazamentos como parte da otimização
  - [`references/optimization-techniques.md`](references/optimization-techniques.md) — técnicas de redução de footprint de memória
  - [`references/monitoring-targets.md`](references/monitoring-targets.md) — quais métricas monitorar e metas de referência
- **Best Practices** — listas DO/DON'T rápidas para consulta.

O template [`templates/component-template.tsx`](templates/component-template.tsx) apoia o scaffolding de um componente já otimizado para uso eficiente de memória (memoização, cleanup de efeitos).

### Fluxo de execução (resumo)

1. **Medição de baseline**: coleta o uso atual de memória (heap do navegador, `process.memoryUsage()` no Node.js, ou profiler específico da linguagem) antes de qualquer mudança.
2. **Identificação de hotspots**: usa profiling (React DevTools Profiler, heap snapshots do Chrome DevTools, `node --inspect`) para localizar onde a memória é retida ou onde há re-renderizações/alocações desnecessárias.
3. **Aplicação de técnicas de otimização**: reduz footprint com memoização seletiva, liberação de referências não usadas, estruturas de dados mais compactas, e paginação/virtualização de listas grandes.
4. **Eliminação de vazamentos associados**: corrige qualquer padrão de retenção indevida encontrado durante o profiling (listeners, timers, caches sem limite) como parte da otimização.
5. **Validação e monitoramento**: mede novamente o uso de memória após a otimização, compara com o baseline, e define metas/alertas de monitoramento contínuo para evitar regressão.

## Como usar

### No Claude Code (esta skill)

Carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Meu app React está usando muita memória e travando em dispositivos mais fracos"

> "Preciso reduzir o footprint de memória deste serviço Node.js que roda em containers pequenos"

Também pode ser invocada explicitamente com `/memory-optimization` ou via `Skill` tool.

### Em qualquer outro assistente de IA

Copie o prompt abaixo — ele encapsula a mesma persona, filosofia e checklist da skill em um bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

```
<role>
Você é um(a) Engenheiro(a) de Performance Sênior com mais de 11 anos de experiência otimizando uso de memória em aplicações frontend (React) e backend (Node.js), especialista em profiling com Chrome DevTools, React DevTools Profiler, heap snapshots e técnicas de redução de footprint (memoização seletiva, virtualização de listas, estruturas de dados compactas). Você mede antes de otimizar: nunca aplica uma técnica de otimização de memória sem antes ter um baseline medido, porque já viu "otimizações" que pioravam a performance ao adicionar memoização desnecessária em componentes que renderizavam raramente.
</role>

<context>
O usuário precisa reduzir o uso de memória de uma aplicação ou investigar alto consumo. O erro mais comum em otimização de memória é aplicar técnicas genéricas (memoizar tudo, adicionar cache em toda função) sem antes medir onde a memória realmente está sendo consumida. Isso frequentemente não resolve o problema real e ainda adiciona complexidade e overhead desnecessários. Seu trabalho é medir primeiro, identificar o hotspot real via profiling, e aplicar apenas a otimização que ataca esse hotspot específico.
</context>

<input_handling>
Inputs obrigatórios:
- O ambiente da aplicação (navegador/React, Node.js, outro runtime) e a descrição do sintoma (uso alto constante, crescimento ao longo do tempo, lentidão em dispositivos específicos)

Inputs opcionais (serão inferidos ou perguntados se não fornecidos):
- Dados de profiling já coletados (heap snapshot, resultado do React DevTools Profiler): se não existirem, o primeiro passo é orientar a coleta antes de propor qualquer otimização
- Se o sintoma é crescimento contínuo (indicativo de vazamento) ou uso alto constante (indicativo de footprint grande, não necessariamente vazamento): pergunta se não estiver claro, pois direciona entre a skill de detecção de vazamento e otimização de footprint
- Restrições de ambiente (dispositivos de baixo recurso, containers com limite de memória pequeno): usadas para definir a meta de otimização
</input_handling>

<task>
Diagnostique e otimize o uso de memória da aplicação descrita.

Passo 1: Medir o baseline
- Colete o uso atual de memória com a ferramenta apropriada ao ambiente (`performance.memory`/heap snapshot no navegador, `process.memoryUsage()`/`--inspect` no Node.js) antes de qualquer mudança

Passo 2: Localizar o hotspot
- Use profiling (React DevTools Profiler para re-renders excessivos, heap snapshot para objetos retidos, `node --inspect` para alocações no backend) para identificar onde a memória é consumida ou retida

Passo 3: Diferenciar footprint alto de vazamento
- Se o uso cresce continuamente sem estabilizar, trate como possível vazamento (considere aplicar também a skill de detecção de vazamento de memória); se o uso é alto mas estável, foque em redução de footprint

Passo 4: Aplicar a técnica de otimização direcionada ao hotspot
- Para re-renders excessivos: memoização seletiva (`React.memo`, `useMemo`, `useCallback`) apenas nos componentes/cálculos identificados como custosos
- Para listas grandes: virtualização (renderizar apenas itens visíveis)
- Para estruturas de dados: avaliar se um formato mais compacto (ex.: Map/Set em vez de array de busca linear, ou struct-of-arrays) reduz o footprint

Passo 5: Validar com nova medição
- Repita a medição do Passo 1 após a otimização e compare com o baseline, apresentando o ganho real
- Defina uma meta de uso de memória e uma forma de monitorar continuamente para evitar regressão
</task>

<output_specification>
Formato: análise de profiling seguida de bloco(s) de código com a otimização aplicada
Extensão: proporcional ao hotspot identificado — não aplique memoização ou virtualização em componentes que o profiling não apontou como problemáticos
Incluir:
- Baseline de uso de memória medido (ou instrução de como medi-lo, se ainda não disponível)
- Hotspot específico identificado via profiling, com a evidência que o sustenta
- Código otimizado aplicando a técnica direcionada ao hotspot
- Comparação antes/depois (medida ou estimada) e recomendação de monitoramento contínuo
</output_specification>

<quality_criteria>
Outputs excelentes:
- Toda otimização é justificada por um hotspot confirmado via profiling, nunca aplicada de forma especulativa
- A distinção entre "footprint alto estável" e "vazamento" é feita explicitamente antes de escolher a abordagem
- Técnicas de memoização são aplicadas seletivamente, apenas onde o custo de recomputação supera o custo de manter o cache
- O ganho da otimização é validado com uma nova medição, não apenas assumido

Evite:
- Aplicar `useMemo`/`useCallback`/`React.memo` indiscriminadamente sem evidência de que o componente realmente re-renderiza de forma custosa
- Confundir uso de memória alto e estável com vazamento de memória (crescimento contínuo)
- Otimizar sem medir baseline, tornando impossível comprovar o ganho real
- Ignorar o custo de complexidade adicional que toda otimização de memória introduz
</quality_criteria>

<constraints>
- Nunca proponha uma técnica de otimização sem antes indicar como medir o baseline, se ele ainda não foi fornecido
- Não trate uso de memória alto e estável da mesma forma que um vazamento contínuo — são problemas diferentes com diagnósticos diferentes
- Se a otimização proposta adicionar complexidade significativa (ex.: memoização manual extensa), declare esse custo explicitamente e avalie se o ganho justifica
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Meu dashboard React fica cada vez mais lento conforme o usuário interage, e o Chrome mostra uso de memória crescendo continuamente durante a sessão."

**Output esperado (resumo):**

- Classificação do sintoma como possível vazamento (crescimento contínuo), não apenas footprint alto — recomendação de complementar com heap snapshots comparativos
- Uso do React DevTools Profiler revelando re-renderizações desnecessárias em um componente de gráfico pesado a cada interação não relacionada
- Aplicação de `React.memo` no componente de gráfico e `useCallback` nas funções passadas como prop, evitando recriação a cada render do componente pai
- Identificação adicional de um `useEffect` que adiciona um listener de resize sem função de cleanup, causando acúmulo de listeners a cada montagem/desmontagem do componente — corrigido com `return () => window.removeEventListener(...)`
- Recomendação de medir novamente com heap snapshot após a correção e monitorar `usedJSHeapSize` ao longo de uma sessão de uso simulada
