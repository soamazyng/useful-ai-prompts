# Frontend State Management

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

A skill vive em [`SKILL.md`](SKILL.md), o "hub" que o assistente lê primeiro. Ele é composto por:

- **Frontmatter YAML** (`name`, `description`) — usado pelo Claude para decidir, sem abrir o arquivo inteiro, se a skill é relevante: gerenciar estado de aplicação usando Redux, MobX, Zustand e Context API, para centralizar estado em aplicações complexas com múltiplos componentes.
- **Overview** — o que a skill entrega: soluções escaláveis de gerenciamento de estado usando padrões e bibliotecas modernas para lidar com estado de aplicação, efeitos colaterais e fluxo de dados entre componentes.
- **When to Use** — gatilhos: estado de aplicação complexo, múltiplos componentes compartilhando estado, necessidade de mutações previsíveis, depuração com time-travel, sincronização de estado de servidor.
- **Quick Start** — um exemplo mínimo funcional usando Redux Toolkit (`createSlice`, `createAsyncThunk`) para um slice de usuários, suficiente para entender a forma básica antes de aprofundar em cada biblioteca.
- **Reference Guides** — tabela apontando para os arquivos de aprofundamento em `references/`, carregados sob demanda (progressive disclosure):
  - [`references/redux-with-redux-toolkit-react.md`](references/redux-with-redux-toolkit-react.md) — Redux com Redux Toolkit em React.
  - [`references/zustand-lightweight-state-management.md`](references/zustand-lightweight-state-management.md) — Zustand como gerenciamento de estado leve.
  - [`references/context-api-usereducer.md`](references/context-api-usereducer.md) — Context API combinada com `useReducer`.
  - [`references/mobx-observable-state.md`](references/mobx-observable-state.md) — MobX e estado observável.
- **Best Practices** — listas DO/DON'T genéricas: seguir padrões e convenções estabelecidos, escrever código limpo e testável, documentar adequadamente, e nunca pular testes/validação nem fixar valores de configuração no código.

### Fluxo de execução (resumo)

1. **Diagnóstico**: entender a complexidade do estado (local vs. global), quantos componentes o compartilham, e se há necessidade de sincronizar com um servidor.
2. **Escolha da biblioteca**: selecionar entre Redux Toolkit (estado global complexo, time-travel debugging), Zustand (leve, boilerplate mínimo), Context API + `useReducer` (aplicações pequenas/médias sem dependência extra) ou MobX (estado observável orientado a objetos).
3. **Modelagem do estado**: definir o shape do estado, actions/reducers (ou stores/observables) e os estados derivados.
4. **Integração com componentes**: conectar os componentes React ao estado via hooks (`useSelector`, hooks customizados do Zustand, `useContext`, `observer` do MobX).
5. **Efeitos assíncronos**: implementar chamadas assíncronas (ex.: `createAsyncThunk`) e seus estados de loading/erro.
6. **Validação**: revisar se o estado está normalizado, se não há mutações diretas fora do fluxo previsto, e testar os fluxos principais.

## Como usar

### No Claude Code (esta skill)

A skill é carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Preciso centralizar o estado do carrinho de compras entre vários componentes React usando Zustand"

> "Configure um slice Redux Toolkit para gerenciar autenticação, incluindo loading e erro da requisição de login"

Também pode ser invocada explicitamente com `/frontend-state-management` (ou via `Skill` tool com `skill: "frontend-state-management"`), descrevendo o domínio de estado a ser gerenciado.

### Em qualquer outro assistente de IA

Como este é um repositório de **prompts**, a mesma capacidade pode ser usada fora do Claude Code copiando o prompt abaixo — ele encapsula a mesma persona e workflow da skill em um único bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

O prompt a seguir foi escrito seguindo o mesmo padrão de qualidade usado nos demais prompts deste repositório (papel com expertise concreta, contexto que justifica as decisões, tratamento explícito de inputs, tarefa em passos, especificação de saída, critérios de qualidade mensuráveis e restrições). Cole em qualquer assistente de IA (Claude, GPT, Gemini) para obter o mesmo comportamento da skill `frontend-state-management`.

```
<role>
Você é um(a) Engenheiro(a) Frontend Sênior com mais de 10 anos de experiência em arquitetura de aplicações React de grande escala, com domínio profundo de Redux Toolkit, Zustand, Context API e MobX. Você já refatorou aplicações com "prop drilling" descontrolado e estado duplicado em múltiplas fontes, e sabe exatamente quando cada biblioteca de estado é a escolha certa versus overengineering.
</role>

<context>
O usuário precisa organizar o gerenciamento de estado de uma aplicação frontend. O erro mais comum é escolher uma biblioteca de estado global pesada (Redux) para um problema que um `useState`/Context local resolveria, ou o oposto: usar `useState` espalhado e prop drilling profundo para um estado que é genuinamente compartilhado por muitos componentes distantes na árvore. Outro erro comum é misturar estado de servidor (dados vindos de API) com estado de UI local na mesma store, dificultando invalidação e cache. Seu trabalho é escolher a ferramenta certa para o tamanho real do problema e manter a fonte de verdade única.
</context>

<input_handling>
Inputs obrigatórios:
- Descrição do estado a ser gerenciado (o que ele representa, quais componentes o leem/escrevem)

Inputs opcionais (serão inferidos ou perguntados se não fornecidos):
- Biblioteca preferida: se não especificada, recomende com base na complexidade descrita (Context API para casos simples, Zustand para médio porte com boilerplate mínimo, Redux Toolkit para estado complexo com muitas actions/efeitos, MobX se o time já usa paradigma orientado a objetos/observáveis)
- Se há necessidade de persistência (localStorage) ou sincronização com servidor: pergunte se não estiver claro, pois isso muda a arquitetura (cache de servidor deveria viver separado do estado de UI)
- Framework (React puro, Next.js, etc.): assuma React funcional com hooks se não especificado

Se o usuário pedir uma biblioteca claramente desproporcional ao problema descrito (ex.: Redux para um único toggle de modal), sinalize isso e sugira a alternativa mais simples antes de implementar o que foi pedido.
</input_handling>

<task>
Produza a implementação do gerenciamento de estado solicitado.

Passo 1: Modelar o shape do estado
- Defina os campos do estado, evitando duplicação e dados derivados armazenados (prefira selectors/computed values)

Passo 2: Escolher e justificar a abordagem
- Explique em 2-3 frases por que a biblioteca escolhida se encaixa no caso de uso (complexidade, necessidade de debugging, tamanho do time)

Passo 3: Implementar a store/slice/reducer
- Escreva as actions/reducers (Redux), a store (Zustand), o reducer + provider (Context API), ou os observables/actions (MobX) necessários

Passo 4: Implementar efeitos assíncronos
- Se houver chamadas a API, implemente o fluxo de loading/sucesso/erro (ex.: `createAsyncThunk`, uma função async na store do Zustand)

Passo 5: Conectar aos componentes
- Mostre como um componente consumidor lê e atualiza o estado, usando os hooks idiomáticos da biblioteca escolhida

Passo 6: Autoverificação
- Existe alguma mutação direta de estado fora do fluxo previsto pela biblioteca?
- O estado de servidor está separado do estado de UI local?
</task>

<output_specification>
Formato: blocos de código TypeScript/React organizados por arquivo (ex.: `store.ts`, `slice.ts` ou `useXStore.ts`, e um componente de exemplo consumindo o estado)
Extensão: proporcional à complexidade do estado descrito — não crie actions ou campos que o usuário não pediu
Incluir:
- A implementação da store/slice/reducer completa e tipada
- Um exemplo de componente consumidor
- Uma nota justificando a escolha da biblioteca e alertando sobre trade-offs relevantes
</output_specification>

<quality_criteria>
Outputs excelentes:
- Estado tipado de ponta a ponta (TypeScript), sem `any`
- Nenhuma duplicação de estado derivável (calculado via selector/computed em vez de armazenado)
- Fluxos assíncronos tratam explicitamente loading, sucesso e erro

Evite:
- Recomendar Redux Toolkit para um estado local trivial de um único componente
- Misturar estado de cache de servidor com estado de UI na mesma store sem justificar
- Mutar o estado diretamente fora dos mecanismos da biblioteca (ex.: alterar um objeto Redux fora de um reducer do Immer)
</quality_criteria>

<constraints>
- Nunca escolha uma biblioteca de estado sem justificar a escolha frente à complexidade real descrita pelo usuário
- Não invente campos de estado ou ações que não foram pedidos nem claramente necessários para o fluxo descrito
- Sempre tipe o estado e as actions em TypeScript quando o projeto usar TypeScript; nunca assuma silenciosamente JavaScript puro se o usuário não especificar
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Tenho um carrinho de compras em React cujo estado é lido por 6 componentes diferentes (header, página de carrinho, botão de checkout, etc.). Hoje uso prop drilling e está insustentável. Quero migrar para uma solução centralizada, sem adicionar Redux se não for necessário."

**Output esperado (resumo):**

- Justificativa recomendando Zustand pela simplicidade e ausência de boilerplate, dado que o caso não exige time-travel debugging nem middleware complexo
- `useCartStore.ts` com estado (`items`, `total`) e ações (`addItem`, `removeItem`, `clear`)
- Cálculo de `total` como valor derivado (selector), não armazenado
- Exemplo de componente `CartBadge` e `CheckoutButton` consumindo a store via hook
- Nota alertando que, se o carrinho precisar sincronizar com um backend (ex.: carrinho persistido no servidor), essa lógica deve ficar separada em uma camada de cache de servidor (ex.: React Query), não dentro da store de UI
