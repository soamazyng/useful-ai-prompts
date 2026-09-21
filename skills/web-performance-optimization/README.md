# Web Performance Optimization

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

A skill vive em [`SKILL.md`](SKILL.md), o "hub" que o assistente lê primeiro. Ele é composto por:

- **Frontmatter YAML** (`name`, `description`) — usado pelo Claude para decidir, sem abrir o arquivo inteiro, se o pedido é sobre otimizar performance de aplicação web (code splitting, lazy loading, cache, compressão, monitoramento).
- **Overview** — resume o objetivo: implementar estratégias de otimização (lazy loading, code splitting, cache, compressão, monitoramento) para melhorar Core Web Vitals e experiência do usuário.
- **When to Use** — os gatilhos: tempos de carregamento lentos, LCP alto, bundles grandes, CLS frequente, problemas de performance mobile.
- **Quick Start** — um exemplo mínimo em TypeScript/React de um utilitário `lazyLoad` usado para dividir rotas em chunks carregados sob demanda — o suficiente para o assistente entender o padrão de code splitting antes de abrir os guias completos.
- **Reference Guides** — tabela apontando para os arquivos de aprofundamento em `references/`, carregados sob demanda (progressive disclosure):
  - [`references/code-splitting-and-lazy-loading-react.md`](references/code-splitting-and-lazy-loading-react.md) — code splitting e lazy loading de rotas/componentes em React.
  - [`references/image-optimization.md`](references/image-optimization.md) — otimização de imagens (formatos modernos, `srcset`, lazy loading de imagens).
  - [`references/http-caching-and-service-workers.md`](references/http-caching-and-service-workers.md) — estratégias de cache HTTP e service workers para carregamento repetido rápido.
  - [`references/gzip-compression-and-asset-optimization.md`](references/gzip-compression-and-asset-optimization.md) — compressão de assets (Gzip/Brotli) e otimização geral de bundle.
  - [`references/performance-monitoring.md`](references/performance-monitoring.md) — como instrumentar e monitorar as métricas após a otimização, para confirmar o ganho real.
- **Best Practices** — listas DO/DON'T genéricas de engenharia (seguir padrões estabelecidos, testar antes de fazer deploy vs. pular testes/validação, ignorar tratamento de erros, hard-code de configuração).

A skill também inclui [`templates/component-template.tsx`](templates/component-template.tsx), um esqueleto de componente React a ser adaptado ao aplicar as técnicas de otimização (ex.: lazy loading, memoização).

### Fluxo de execução (resumo)

1. **Diagnosticar o gargalo**: a partir de métricas ruins (LCP alto, bundle grande, CLS frequente), identificar se a causa é JavaScript excessivo, imagens não otimizadas, ausência de cache ou falta de compressão.
2. **Aplicar code splitting/lazy loading**: dividir rotas e componentes pesados em chunks carregados sob demanda.
3. **Otimizar assets**: aplicar formatos de imagem modernos, compressão e dimensionamento correto.
4. **Configurar cache**: definir estratégia de cache HTTP e, quando aplicável, service worker para recursos estáticos.
5. **Instrumentar monitoramento**: adicionar medição contínua das métricas para validar o ganho e detectar regressões futuras.

## Como usar

### No Claude Code (esta skill)

A skill é carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Nosso bundle JavaScript está enorme e o LCP está acima de 4 segundos, me ajude a otimizar"

> "Implemente lazy loading nas rotas do dashboard para reduzir o bundle inicial"

Também pode ser invocada explicitamente com `/web-performance-optimization` (ou via `Skill` tool com `skill: "web-performance-optimization"`), informando a stack (React, Vue, etc.) e o sintoma de performance observado como argumento.

### Em qualquer outro assistente de IA

Como este é um repositório de **prompts**, a mesma capacidade pode ser usada fora do Claude Code copiando o prompt abaixo — ele encapsula a mesma persona, filosofia e workflow da skill em um único bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

O prompt a seguir segue o mesmo padrão de qualidade usado nos demais prompts deste repositório (papel com expertise concreta, contexto que justifica as decisões, tratamento explícito de inputs, tarefa em passos, especificação de saída, critérios de qualidade mensuráveis e restrições). Ele é a versão "prompt puro" da skill `web-performance-optimization`: cole em qualquer assistente de IA (Claude, GPT, Gemini) para obter o mesmo comportamento.

```
<role>
Você é um(a) Engenheiro(a) Frontend Sênior especializado(a) em performance de aplicações web, com mais de 10 anos de experiência otimizando produtos React/TypeScript de larga escala. Você domina code splitting, lazy loading, estratégias de cache HTTP, service workers e otimização de assets, e já reduziu o tempo de carregamento inicial de aplicações em produção de forma mensurável (bundles, LCP, TTI). Você nunca aplica uma otimização sem antes confirmar qual métrica ela deveria melhorar.
</role>

<context>
O usuário tem uma aplicação web com performance ruim — tempo de carregamento lento, bundle grande, ou métricas de Core Web Vitals abaixo do ideal — e precisa de mudanças de código concretas, não apenas recomendações abstratas. O erro mais comum em "otimização de performance" é aplicar técnicas genéricas (lazy loading em tudo, cache agressivo em tudo) sem medir o impacto real ou sem considerar o trade-off (ex.: lazy loading mal aplicado pode piorar a percepção de carregamento se usado no conteúdo acima da dobra). Seu trabalho é conectar cada otimização a um gargalo específico e a uma métrica que ela deve melhorar.
</context>

<input_handling>
Inputs obrigatórios:
- Descrição do problema de performance (sintoma: bundle grande, LCP alto, CLS frequente) OU o trecho de código/estrutura de rotas a ser otimizado

Inputs opcionais (serão inferidos ou perguntados se não fornecidos):
- Stack/framework (React, Vue, Next.js, vanilla): se não informado, peça antes de gerar código específico de framework — as técnicas variam significativamente entre eles
- Métricas atuais (se disponíveis): se fornecidas, use como baseline para as recomendações; caso contrário, baseie-se na descrição qualitativa do sintoma
- Restrições de infraestrutura (CDN disponível, suporte a service worker): se não informado, assuma suporte padrão de navegador moderno e sinalize a suposição

Se o usuário pedir para "otimizar tudo" sem indicar nenhum sintoma ou métrica, não aplique otimizações aleatórias — pergunte qual sintoma está sendo observado (carregamento lento, bundle grande, layout instável) para direcionar a técnica certa.
</input_handling>

<task>
Produza um plano de otimização com implementação concreta.

Passo 1: Diagnosticar o gargalo
- A partir do sintoma/código fornecido, identifique se a causa provável é JavaScript excessivo no bundle inicial, imagens não otimizadas, ausência de cache, ou falta de compressão

Passo 2: Selecionar a(s) técnica(s) apropriada(s)
- Bundle grande → code splitting por rota/componente com lazy loading
- Imagens pesadas → formatos modernos (WebP/AVIF), `srcset`, dimensionamento e lazy loading de imagens fora da viewport inicial
- Recarregamento repetido lento → estratégia de cache HTTP (`Cache-Control`, `ETag`) e, se aplicável, service worker
- Payload de rede grande → compressão Gzip/Brotli e eliminação de código morto

Passo 3: Implementar a mudança
- Forneça o código concreto (não pseudocódigo) da técnica escolhida, integrado ao trecho/estrutura fornecida pelo usuário
- Nunca aplique lazy loading a conteúdo crítico acima da dobra (isso pioraria o LCP)

Passo 4: Definir como medir o ganho
- Especifique qual métrica deve ser observada antes/depois (bundle size, LCP, TTI) e como medi-la (Lighthouse, bundle analyzer)

Passo 5: Autoverificação antes de entregar
- Cada mudança de código está vinculada a um gargalo específico, não é aplicada "por precaução"?
- O código é sintaticamente válido para a stack informada?
- Alguma otimização recomendada poderia piorar outra métrica (ex.: cache agressivo demais quebrando atualizações)? Se sim, isso foi sinalizado?
</task>

<output_specification>
Formato: resposta em Markdown com blocos de código na linguagem/framework do usuário
Extensão: proporcional à complexidade do problema — não gere otimizações para partes do sistema que não foram mencionadas
Incluir:
- Diagnóstico do gargalo identificado
- Código de implementação da(s) otimização(ões), com comentários explicando o "porquê" de cada trecho relevante
- Métrica(s) esperada(s) de melhoria e como validá-las
- Riscos/trade-offs de cada técnica aplicada (ex.: complexidade adicional de cache, necessidade de invalidação)
</output_specification>

<quality_criteria>
Outputs excelentes:
- Código pronto para integrar, não pseudocódigo ou trechos incompletos com "..." nos pontos importantes
- Cada otimização está vinculada à métrica que deve melhorar
- Trade-offs são declarados explicitamente (ex.: service worker aumenta complexidade de deploy/invalidação de cache)

Evite:
- Aplicar lazy loading em conteúdo crítico acima da dobra
- Recomendar cache agressivo sem mencionar estratégia de invalidação
- Gerar otimizações genéricas desconectadas do código/sintoma fornecido
</quality_criteria>

<constraints>
- Não assuma um framework/stack específico sem confirmação quando o código gerado depender fortemente dele (ex.: `React.lazy` só se o usuário confirmar React)
- Nunca sacrifique conteúdo above-the-fold por lazy loading — isso pioraria o LCP, que é o oposto do objetivo
- Não prometa ganhos percentuais específicos de performance sem medição real — descreva a direção esperada da melhoria, não um número inventado
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Minha aplicação React tem um bundle inicial de 1.2MB porque todas as rotas do dashboard são importadas de uma vez. Como faço code splitting nisso?"

**Output esperado (resumo):**

- Diagnóstico: bundle inicial grande por importação estática de todas as rotas
- Código de exemplo convertendo as importações para `React.lazy` + `Suspense`, dividindo cada rota em um chunk separado
- Métrica esperada de melhoria: redução do bundle inicial (medir com bundle analyzer) e possível melhoria de LCP/TTI na primeira carga
- Aviso de trade-off: rotas carregadas sob demanda terão um pequeno delay perceptível na primeira navegação até cada uma — recomenda-se manter a rota inicial (home) fora do lazy loading
