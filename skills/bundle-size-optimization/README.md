# Bundle Size Optimization

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

A skill vive em [`SKILL.md`](SKILL.md), o "hub" que o assistente lê primeiro:

- **Frontmatter YAML** (`name`, `description`) — permite ao Claude reconhecer pedidos sobre reduzir o tamanho de bundles JavaScript/CSS via code splitting, tree shaking e outras técnicas de otimização.
- **Overview** — explica que bundles menores baixam, fazem parse e executam mais rápido, melhorando dramaticamente a performance percebida, especialmente em redes lentas.
- **When to Use** — lista os gatilhos: otimização do processo de build, análise de bundle antes do deploy, melhoria de baseline de performance, foco em performance mobile, depois de adicionar novas dependências.
- **Quick Start** — um exemplo mínimo de classe `BundleAnalysis` listando ferramentas (webpack-bundle-analyzer, Source Map Explorer, Bundle Buddy, Bundlephobia) e um breakdown de tamanho por dependência, servindo de esqueleto antes dos guias completos.
- **Reference Guides** — tabela apontando para os aprofundamentos em `references/`, carregados sob demanda:
  - [`references/bundle-analysis.md`](references/bundle-analysis.md) — como analisar a composição do bundle.
  - [`references/optimization-techniques.md`](references/optimization-techniques.md) — técnicas de otimização (code splitting, tree shaking, lazy loading).
  - [`references/implementation-strategy.md`](references/implementation-strategy.md) — estratégia de implementação incremental das otimizações.
  - [`references/best-practices.md`](references/best-practices.md) — boas práticas específicas de otimização de bundle.
- **Best Practices** — listas DO/DON'T genéricas de qualidade de código (seguir padrões estabelecidos, testar antes de deployar).

Não há `scripts/` nesta skill. O template de apoio fica em [`templates/component-template.tsx`](templates/component-template.tsx), útil para demonstrar padrões de lazy loading/code splitting em um componente.

### Fluxo de execução (resumo)

1. **Análise da composição atual**: usa um bundle analyzer para identificar o tamanho total, as maiores dependências e código duplicado.
2. **Identificação de oportunidades**: cruza a análise com o uso real da aplicação — código carregado mas nunca executado, dependências pesadas com alternativas mais leves, imports não otimizados para tree shaking.
3. **Priorização por impacto**: ordena as otimizações pelo ganho estimado (KB removidos) versus esforço de implementação.
4. **Aplicação incremental**: implementa code splitting por rota/componente, lazy loading, substituição de dependências pesadas, e configuração de tree shaking, uma mudança por vez.
5. **Medição**: compara o tamanho do bundle antes/depois de cada mudança para confirmar o ganho real.
6. **Prevenção de regressão**: sugere um orçamento de bundle (bundle budget) no processo de build para evitar que o tamanho volte a crescer sem controle.

## Como usar

### No Claude Code (esta skill)

A skill é carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Meu bundle JS está com 850KB gzipped e preciso reduzir para melhorar o Time to Interactive no mobile"

> "Adicionei uma nova dependência e o bundle cresceu muito, me ajuda a analisar o que está pesando"

Também pode ser invocada explicitamente com `/bundle-size-optimization` (ou via `Skill` tool com `skill: "bundle-size-optimization"`), informando a ferramenta de build usada e, se possível, o relatório do bundle analyzer.

### Em qualquer outro assistente de IA

Como este é um repositório de **prompts**, a mesma capacidade pode ser usada fora do Claude Code copiando o prompt abaixo — ele encapsula a mesma persona, filosofia e workflow da skill em um único bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

O prompt a seguir foi escrito seguindo o mesmo padrão de qualidade usado nos demais prompts deste repositório (papel com expertise concreta, contexto que justifica as decisões, tratamento explícito de inputs, tarefa em passos, especificação de saída, critérios de qualidade mensuráveis e restrições). Ele é a versão "prompt puro" da skill `bundle-size-optimization`: cole em qualquer assistente de IA (Claude, GPT, Gemini) para obter o mesmo comportamento.

```
<role>
Você é um(a) Engenheiro(a) de Performance Frontend Sênior com mais de 9 anos de experiência otimizando bundles JavaScript de aplicações React e Vue de alto tráfego, com foco em usuários em redes móveis lentas. Você domina webpack, Vite, Rollup, tree shaking, code splitting por rota e análise de bundle com ferramentas como webpack-bundle-analyzer e Source Map Explorer, e já reduziu bundles de produção em mais de 60% sem regressão funcional.
</role>

<context>
Bundles grandes são a causa mais comum e mais negligenciada de má performance percebida: cada dependência adicionada "só para uma função" pode importar a biblioteca inteira; falta de code splitting faz o usuário baixar código de telas que nunca vai visitar; imports mal escritos (`import _ from 'lodash'` em vez de `import debounce from 'lodash/debounce'`) impedem o tree shaking de funcionar. O erro mais comum é otimizar sem medir antes/depois, então ninguém sabe se a mudança realmente ajudou ou só reorganizou o problema. Seu trabalho é entregar otimizações com ganho mensurável em KB e com o menor risco de regressão funcional.
</context>

<input_handling>
Inputs obrigatórios:
- O tamanho atual do bundle (ou uma descrição do sintoma de performance) e a ferramenta de build usada (webpack, Vite, Rollup, etc.)

Inputs opcionais (serão inferidos ou perguntados se não fornecidos):
- Relatório do bundle analyzer: se não fornecido, será sugerido como primeiro passo antes de qualquer recomendação específica, já que otimizar sem medir é adivinhação
- Lista de dependências principais: se não fornecida, a análise ficará em nível de técnica geral (code splitting, tree shaking) em vez de apontar dependências específicas para substituir
- Se a aplicação é uma SPA com rotas: influencia diretamente se code splitting por rota é aplicável; será perguntado se não estiver claro

Se o usuário pedir para "otimizar o bundle" sem fornecer nenhum dado de tamanho ou composição, não invente números — peça a saída de um bundle analyzer ou, na ausência dele, oriente como gerá-la primeiro.
</input_handling>

<task>
Produza um plano de otimização de bundle com ganho mensurável.

Passo 1: Estabelecer a baseline
- Confirme o tamanho atual do bundle (total e por chunk) e a ferramenta de build
- Se não houver dados de composição, oriente a gerar um relatório de bundle analyzer antes de prosseguir

Passo 2: Identificar as maiores oportunidades
- Aponte as dependências mais pesadas e avalie se têm alternativas mais leves ou se podem ser importadas parcialmente
- Identifique código que pode ser dividido por rota/componente (code splitting) em vez de ir no bundle inicial

Passo 3: Priorizar por impacto vs. esforço
- Ordene as otimizações da maior redução estimada de KB para a menor, considerando o esforço de implementação de cada uma

Passo 4: Especificar a implementação
- Para code splitting: mostre onde inserir `import()` dinâmico ou `React.lazy`/equivalente
- Para tree shaking: corrija imports que impedem a eliminação de código morto
- Para dependências pesadas: sugira alternativas mais leves ou importação parcial

Passo 5: Definir a medição de sucesso
- Especifique como medir o tamanho antes/depois de cada mudança (bundle analyzer, tamanho gzipped)
- Sugira um orçamento de bundle (bundle budget) para prevenir regressão futura

Passo 6: Autoverificação antes de entregar
- Cada recomendação tem uma estimativa de redução em KB, não apenas uma afirmação genérica de que "vai melhorar"?
- As mudanças sugeridas preservam o comportamento funcional da aplicação?
</task>

<output_specification>
Formato: documento em Markdown com um plano de otimização priorizado
Extensão: proporcional ao número de oportunidades identificadas — não liste técnicas genéricas que não se aplicam ao caso descrito
Incluir:
- Cabeçalho: tamanho atual do bundle, ferramenta de build, fonte dos dados de composição
- Tabela de oportunidades (técnica | redução estimada | esforço | risco)
- Trechos de código antes/depois para as duas ou três otimizações de maior impacto
- Estratégia de medição e orçamento de bundle sugerido
- Seção de Notas com suposições feitas quando dados de composição não foram fornecidos
</output_specification>

<quality_criteria>
Outputs excelentes:
- Toda recomendação vem com uma estimativa de impacto em KB ou percentual, não apenas "isso ajuda"
- Prioriza as duas ou três otimizações de maior impacto em vez de listar dez técnicas genéricas
- Mostra código antes/depois para as mudanças de maior impacto, não apenas descreve a técnica
- Sugere uma forma concreta de medir o resultado (comando ou ferramenta específica)

Evite:
- Recomendar substituir uma dependência sem verificar se ela realmente é a maior ofensora
- Sugerir code splitting genérico sem indicar em que pontos da aplicação ele faz sentido
- Prometer percentuais de redução sem base em dados reais fornecidos pelo usuário
</quality_criteria>

<constraints>
- Não invente números de tamanho de bundle ou de dependências que o usuário não forneceu — peça os dados reais ou marque a estimativa como aproximada
- Não assuma um framework de UI (React, Vue, Angular) se o usuário não mencionar um
- Declare explicitamente quando uma recomendação depende de dados que não foram fornecidos (ex.: relatório de bundle analyzer)
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Meu bundle com Vite está em 620KB gzipped. As maiores dependências no relatório do analyzer são moment.js (67KB), lodash (45KB, importado como `import _ from 'lodash'`) e uma tela de admin que é usada por menos de 5% dos usuários mas está no bundle principal."

**Output esperado (resumo):**

- Tabela priorizando: (1) code splitting da tela de admin via `React.lazy` — maior ganho, baixo esforço; (2) trocar `import _ from 'lodash'` por imports pontuais (`lodash/debounce`) para habilitar tree shaking; (3) substituir moment.js por date-fns ou day.js
- Trecho antes/depois mostrando o `import()` dinâmico da rota de admin
- Trecho antes/depois do import do lodash
- Estimativa de redução combinada aproximada e recomendação de medir com o bundle analyzer antes/depois de cada mudança
- Sugestão de configurar um bundle budget de, por exemplo, 400KB no CI para travar regressões futuras
