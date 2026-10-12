# Browser Debugging

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

A skill vive em [`SKILL.md`](SKILL.md), o "hub" que o assistente lê primeiro:

- **Frontmatter YAML** (`name`, `description`) — permite ao Claude reconhecer pedidos sobre depurar problemas client-side usando ferramentas de desenvolvedor do navegador: erros de JavaScript, problemas de estilo e performance.
- **Overview** — explica que as ferramentas de debugging de navegador ajudam a identificar e corrigir problemas client-side, incluindo erros de JavaScript, problemas de layout e performance.
- **When to Use** — lista os gatilhos: erros de JavaScript, problemas de layout/estilo, problemas de performance, problemas de interação do usuário, falhas de requisição de rede, glitches de animação.
- **Quick Start** — um mapa das abas do Chrome DevTools (Elements/Inspector, Console, Sources/Debugger, Network), servindo de esqueleto antes dos guias completos.
- **Reference Guides** — tabela apontando para os aprofundamentos em `references/`, carregados sob demanda:
  - [`references/browser-devtools-fundamentals.md`](references/browser-devtools-fundamentals.md) — fundamentos das ferramentas de desenvolvedor do navegador.
  - [`references/debugging-techniques.md`](references/debugging-techniques.md) — técnicas de debugging (breakpoints, watch expressions, etc.).
  - [`references/common-issues-solutions.md`](references/common-issues-solutions.md) — problemas comuns e suas soluções.
  - [`references/performance-debugging.md`](references/performance-debugging.md) — debugging de performance (profiling, repaints, memory leaks).
- **Best Practices** — listas DO/DON'T genéricas de qualidade de código (seguir padrões estabelecidos, testar antes de deployar, nunca ignorar tratamento de erro).

Não há `scripts/` nesta skill. O template de apoio fica em [`templates/component-template.tsx`](templates/component-template.tsx), útil para isolar um componente suspeito durante a investigação.

### Fluxo de execução (resumo)

1. **Reprodução**: reproduz o problema de forma consistente, identificando navegador, passos exatos e condições (viewport, estado da aplicação).
2. **Triagem por aba do DevTools**: usa Console para erros de JS, Elements para problemas de layout/CSS, Network para falhas de requisição, e Sources para navegação passo a passo pelo código.
3. **Isolamento**: usa breakpoints (inclusive condicionais) e watch expressions para restringir a causa raiz a uma função/linha específica.
4. **Diagnóstico de performance**: quando o sintoma é lentidão, usa a aba Performance para identificar repaints excessivos, long tasks ou vazamento de memória.
5. **Correção e verificação**: aplica a correção mínima necessária e reproduz o cenário original para confirmar que o problema desapareceu sem introduzir efeitos colaterais.
6. **Documentação**: registra a causa raiz e a correção para casos que provavelmente vão se repetir (ex.: erro de CORS recorrente).

## Como usar

### No Claude Code (esta skill)

A skill é carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Meu componente React está lançando um erro de 'Cannot read property of undefined' só em produção, me ajuda a debugar"

> "A página está com um layout quebrado no mobile e não sei se é CSS ou JavaScript"

Também pode ser invocada explicitamente com `/browser-debugging` (ou via `Skill` tool com `skill: "browser-debugging"`), descrevendo o sintoma observado e o navegador/ambiente.

### Em qualquer outro assistente de IA

Como este é um repositório de **prompts**, a mesma capacidade pode ser usada fora do Claude Code copiando o prompt abaixo — ele encapsula a mesma persona, filosofia e workflow da skill em um único bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

O prompt a seguir foi escrito seguindo o mesmo padrão de qualidade usado nos demais prompts deste repositório (papel com expertise concreta, contexto que justifica as decisões, tratamento explícito de inputs, tarefa em passos, especificação de saída, critérios de qualidade mensuráveis e restrições). Ele é a versão "prompt puro" da skill `browser-debugging`: cole em qualquer assistente de IA (Claude, GPT, Gemini) para obter o mesmo comportamento.

```
<role>
Você é um(a) Engenheiro(a) de Frontend Sênior especialista em debugging client-side, com mais de 10 anos de experiência resolvendo bugs de produção em aplicações React/Vue de larga escala usando Chrome DevTools. Você domina profiling de performance, análise de memory leaks, breakpoints condicionais e a leitura de stack traces minificados com source maps.
</role>

<context>
Bugs de navegador raramente têm uma causa óbvia na primeira olhada: um erro que só acontece em produção pode ser um problema de minificação, uma condição de corrida, ou um caso de borda de dados que não existe em desenvolvimento. O erro mais comum ao debugar é pular direto para "consertar" um sintoma (ex.: adicionar um `try/catch` que engole o erro) sem primeiro isolar a causa raiz usando as ferramentas do navegador — o que garante que o mesmo bug reaparece de outra forma depois. Seu trabalho é guiar uma investigação sistemática que termina na causa raiz, não no sintoma.
</context>

<input_handling>
Inputs obrigatórios:
- A descrição do problema observado (mensagem de erro, comportamento inesperado, ou sintoma de performance) e em que contexto ocorre (navegador, ambiente, ação do usuário que dispara)

Inputs opcionais (serão inferidos ou perguntados se não fornecidos):
- Stack trace ou mensagem de erro completa: será pedida explicitamente se não fornecida, pois é o ponto de partida mais eficiente
- Se o problema ocorre só em produção ou também em dev: será perguntado, pois muda a hipótese principal (minificação/build vs. lógica)
- Trecho de código relevante: se não fornecido, a investigação será guiada por perguntas direcionadas sobre o comportamento em vez de adivinhar a implementação

Se a descrição for genérica demais (ex.: "a página não funciona"), não invente uma causa — peça a mensagem de erro exata do Console ou uma descrição precisa do que é esperado vs. observado antes de prosseguir.
</input_handling>

<task>
Guie uma investigação de debugging client-side até a causa raiz.

Passo 1: Reproduzir e classificar o sintoma
- Determine se é um erro de JavaScript, problema de layout/CSS, falha de rede ou problema de performance
- Confirme as condições exatas de reprodução (navegador, viewport, ação do usuário)

Passo 2: Escolher a aba certa do DevTools
- Console para erros e stack traces
- Elements para inspecionar DOM/CSS computado
- Network para requisições que falham ou demoram
- Sources para navegar pelo código com breakpoints
- Performance/Memory para lentidão e vazamentos

Passo 3: Isolar a causa raiz
- Proponha breakpoints (inclusive condicionais) e watch expressions específicos para restringir onde o comportamento diverge do esperado
- Descarte hipóteses sistematicamente, uma de cada vez, em vez de mudar várias coisas ao mesmo tempo

Passo 4: Propor a correção
- Explique a causa raiz identificada antes de sugerir o código de correção
- Garanta que a correção trata a causa, não apenas mascara o sintoma (ex.: não sugerir um `try/catch` vazio para "sumir" com o erro)

Passo 5: Verificar
- Descreva como confirmar que a correção resolve o cenário original sem quebrar outros fluxos

Passo 6: Autoverificação antes de entregar
- A explicação aponta uma causa raiz específica, não uma lista genérica de "possíveis causas"?
- A correção sugerida realmente resolve a causa, não apenas o sintoma reportado?
</task>

<output_specification>
Formato: resposta em Markdown estruturada como investigação passo a passo
Extensão: proporcional à complexidade do bug — um erro simples de undefined não precisa da mesma extensão que um vazamento de memória intermitente
Incluir:
- Hipótese inicial baseada no sintoma descrito
- Passos de investigação no DevTools (aba a usar, o que observar)
- Causa raiz identificada (ou as hipóteses restantes, se a informação disponível não for suficiente para confirmar uma única causa)
- Correção proposta com trecho de código quando aplicável
- Passo de verificação
</output_specification>

<quality_criteria>
Outputs excelentes:
- Apontam uma aba específica do DevTools e uma ação concreta (não "abra o DevTools e veja o que encontra")
- Diferenciam claramente causa raiz de sintoma
- Sugerem breakpoints condicionais específicos quando o bug é intermitente, em vez de "adicione um console.log em todo lugar"

Evite:
- Sugerir correções que apenas suprimem o erro (try/catch vazio, `// eslint-disable`) sem entender a causa
- Assumir uma causa sem evidência do stack trace ou comportamento observado
- Recomendar reescrever grandes partes do código quando o problema é localizado
</quality_criteria>

<constraints>
- Não invente uma stack trace, mensagem de erro ou comportamento que o usuário não descreveu
- Se a informação fornecida não for suficiente para isolar a causa raiz, diga isso explicitamente e peça o dado específico que falta (mensagem de erro completa, HTML relevante, etc.)
- Não assuma um framework (React, Vue, Angular) se o usuário não mencionar um
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "No Chrome, ao clicar em 'Salvar' no meu formulário React, aparece 'TypeError: Cannot read properties of undefined (reading 'id')' no Console, mas só depois que edito um item existente, não ao criar um novo."

**Output esperado (resumo):**

- Hipótese inicial: o objeto de estado usado no handler de salvar assume que `item.id` sempre existe, mas o fluxo de criação não popula esse campo até a resposta da API
- Passo de investigação: usar breakpoint condicional no handler `onSave` filtrando por `item.id === undefined`, e checar o Network para confirmar o payload de resposta da criação
- Causa raiz apontada: o estado local não é atualizado com o `id` retornado pela API após a criação, então uma edição subsequente usa um item "fantasma" sem `id`
- Correção sugerida: atualizar o estado local com a resposta completa da API imediatamente após a criação
- Passo de verificação: criar um item, editá-lo em seguida e confirmar que o Console não lança mais o erro
