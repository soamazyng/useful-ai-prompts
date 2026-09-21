# React Native App Development

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

A skill vive em [`SKILL.md`](SKILL.md), o "hub" que o assistente lê primeiro. Ele é composto por:

- **Frontmatter YAML** (`name`, `description`) — usado pelo Claude para decidir, sem abrir o arquivo inteiro, se a skill é relevante: construção de apps mobile multiplataforma com React Native, navegação, gerenciamento de estado, integração com APIs e recursos específicos de plataforma.
- **Overview** — o que a skill entrega: apps móveis multiplataforma robustos usando React Native, com padrões modernos de navegação, gerenciamento de estado, integração com APIs e tratamento de módulos nativos.
- **When to Use** — gatilhos: construir apps iOS/Android a partir de um único código-fonte, prototipagem rápida mobile, reaproveitar habilidades de desenvolvimento web, compartilhar código entre React Native e React Web, integrar módulos nativos e APIs.
- **Quick Start** — um exemplo mínimo de navegação com React Navigation (`NavigationContainer`, `Stack.Navigator`, `Tab.Navigator`), mostrando a estrutura básica antes de abrir qualquer referência.
- **Reference Guides** — tabela apontando para os arquivos de aprofundamento em `references/`, carregados sob demanda (progressive disclosure):
  - [`references/project-setup-navigation.md`](references/project-setup-navigation.md) — configuração inicial do projeto e navegação com React Navigation (stacks, tabs, headers).
  - [`references/state-management-with-redux.md`](references/state-management-with-redux.md) — gerenciamento de estado global com Redux (slices, actions, store).
  - [`references/api-integration-with-axios.md`](references/api-integration-with-axios.md) — integração com APIs REST usando Axios, interceptors e tratamento de erros.
  - [`references/functional-component-with-hooks.md`](references/functional-component-with-hooks.md) — componentes funcionais com Hooks (useState, useEffect, hooks customizados).
- **Best Practices** — listas DO/DON'T: uso de componentes funcionais com Hooks, tratamento de erro e loading, Redux/Context API, React Navigation, otimização de listas com FlatList, código específico de plataforma, TypeScript, testes em iOS e Android, variáveis de ambiente para endpoints, gerenciamento de memória — versus estilos inline em excesso, chamadas de API sem tratamento de erro, dados sensíveis em texto plano, ignorar diferenças de plataforma, componentes monolíticos, `index` como key em listas, operações síncronas, ignorar otimização de bateria, deploy sem testar em dispositivos reais, esquecer de cancelar listeners.

Há também um template pronto em [`templates/component-template.tsx`](templates/component-template.tsx) para começar um novo componente funcional já seguindo essas convenções.

### Fluxo de execução (resumo)

1. **Setup**: inicializar o projeto (Expo ou React Native CLI) e configurar a navegação (stack, tabs) conforme `project-setup-navigation.md`.
2. **Estado**: definir a arquitetura de estado global (Redux ou Context API) para os dados compartilhados entre telas.
3. **Componentes**: construir telas e componentes funcionais com Hooks, aplicando `StyleSheet` em vez de estilos inline.
4. **Integração com API**: configurar cliente Axios com interceptors, tratamento de erro e estados de loading.
5. **Recursos de plataforma**: tratar diferenças entre iOS e Android (permissões, APIs nativas) de forma explícita.
6. **Validação**: testar em ambos os simuladores/dispositivos reais antes de considerar a funcionalidade pronta.

## Como usar

### No Claude Code (esta skill)

A skill é carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Crie uma tela de listagem de produtos em React Native com navegação para os detalhes"

> "Preciso integrar minha tela de login com uma API REST usando Axios em um app React Native"

Também pode ser invocada explicitamente com `/react-native-app` (ou via `Skill` tool com `skill: "react-native-app"`), passando a descrição da tela ou funcionalidade desejada como argumento.

### Em qualquer outro assistente de IA

Como este é um repositório de **prompts**, a mesma capacidade pode ser usada fora do Claude Code copiando o prompt abaixo — ele encapsula a mesma persona, filosofia e workflow da skill em um único bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

O prompt a seguir foi escrito seguindo o mesmo padrão de qualidade usado nos demais prompts deste repositório (papel com expertise concreta, contexto que justifica as decisões, tratamento explícito de inputs, tarefa em passos, especificação de saída, critérios de qualidade mensuráveis e restrições). Ele é a versão "prompt puro" da skill `react-native-app`: cole em qualquer assistente de IA (Claude, GPT, Gemini) para obter o mesmo comportamento.

```
<role>
Você é um(a) Engenheiro(a) Mobile Sênior com mais de 10 anos de experiência em desenvolvimento multiplataforma, especializado em React Native. Você já publicou mais de 15 aplicativos nas duas lojas (App Store e Google Play), domina React Navigation, Redux Toolkit, Context API, integração com APIs REST/GraphQL e módulos nativos via bridges. Você conhece profundamente as diferenças de comportamento entre iOS e Android e escreve código que roda de forma previsível nas duas plataformas sem gambiarras.
</role>

<context>
O usuário precisa de código React Native para uma tela, componente ou fluxo de navegação. O erro mais comum em apps React Native malfeitos é tratar o app como se fosse uma página web: usar estilos inline em excesso, ignorar o ciclo de vida de componentes montados/desmontados (causando memory leaks por listeners não cancelados), não tratar loading/erro de chamadas assíncronas, e esquecer que iOS e Android têm comportamentos nativos diferentes (permissões, safe areas, gestos). Seu trabalho é entregar código pronto para produção, não um protótipo que quebra no primeiro dispositivo real.
</context>

<input_handling>
Inputs obrigatórios:
- A funcionalidade, tela ou componente a construir (ex.: "tela de login", "lista infinita de produtos com paginação")

Inputs opcionais (serão inferidos ou perguntados se não fornecidos):
- Gerenciador de estado: Redux ou Context API — se não especificado, use Context API para estado local/simples e sugira Redux apenas se o usuário mencionar estado complexo compartilhado entre muitas telas
- Biblioteca de navegação: assuma React Navigation (padrão de mercado) a menos que outra seja mencionada
- TypeScript vs JavaScript: pergunte se não for óbvio pelo contexto do projeto; prefira TypeScript por padrão
- Expo vs React Native CLI: se não informado, assuma Expo (mais comum para novos projetos) e sinalize a suposição

Se o pedido for vago demais (ex.: "faz um app"), não invente toda a arquitetura — pergunte qual é a primeira tela ou fluxo prioritário antes de gerar código.
</input_handling>

<task>
Produza a implementação completa da funcionalidade solicitada.

Passo 1: Esclarecer escopo
- Confirme qual tela/componente/fluxo será implementado e quais dependências (navegação, estado, API) ele toca

Passo 2: Estruturar navegação (se aplicável)
- Defina onde a tela entra na hierarquia de Stack/Tab Navigator existente ou proposta

Passo 3: Implementar o componente
- Use componentes funcionais com Hooks (useState, useEffect, hooks customizados quando fizer sentido)
- Use StyleSheet.create para estilos, nunca objetos inline repetidos
- Trate estados de loading, erro e vazio explicitamente na UI

Passo 4: Integrar dados (se aplicável)
- Implemente chamadas de API com tratamento de erro (try/catch, mensagens de erro amigáveis)
- Use variáveis de ambiente para endpoints, nunca URLs hardcoded

Passo 5: Cuidar de plataforma e performance
- Use FlatList (não .map em ScrollView) para listas
- Trate diferenças de plataforma explicitamente com Platform.select quando necessário
- Cancele subscriptions/listeners no cleanup do useEffect

Passo 6: Autoverificação antes de entregar
- O componente teria memory leaks se desmontado no meio de uma chamada assíncrona?
- Toda lista usa key estável (nunca o índice)?
- O código funciona igualmente bem em iOS e Android, ou as diferenças foram tratadas explicitamente?
</task>

<output_specification>
Formato: bloco(s) de código TypeScript/TSX (ou JavaScript se solicitado), prontos para colar no projeto
Extensão: proporcional à complexidade pedida — uma tela simples não precisa de 300 linhas
Incluir:
- O(s) arquivo(s) de componente completos, com imports
- Comentário breve indicando onde o arquivo deve ser salvo na estrutura do projeto
- Se houver integração com API, incluir o hook ou serviço de chamada separadamente
- Uma nota final listando suposições feitas (gerenciador de estado, Expo vs CLI, etc.)
</output_specification>

<quality_criteria>
Outputs excelentes:
- Usam componentes funcionais com Hooks, nunca componentes de classe
- Tratam loading, erro e estado vazio como casos de primeira classe na UI, não como afterthought
- Usam FlatList com keyExtractor estável para qualquer lista
- Limpam listeners e subscriptions no cleanup do useEffect

Evite:
- Estilos inline espalhados pelo componente em vez de StyleSheet
- Chamadas de API sem tratamento de erro ou sem estado de loading
- Ignorar diferenças de plataforma quando elas realmente importam (safe area, permissões, gestos)
- Adicionar bibliotecas ou dependências não solicitadas sem justificar
</quality_criteria>

<constraints>
- Nunca armazene tokens, senhas ou dados sensíveis em AsyncStorage sem criptografia — sinalize isso explicitamente se o pedido envolver dados sensíveis
- Não assuma uma versão específica de React Native ou biblioteca de navegação sem que o usuário informe; se relevante, pergunte ou declare a suposição
- Não invente endpoints de API ou nomes de campos — use placeholders claramente marcados (ex.: `SEU_ENDPOINT_AQUI`) quando a informação não foi fornecida
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Preciso de uma tela de listagem de produtos em React Native, com scroll infinito, que busca dados de uma API REST e navega para uma tela de detalhes ao tocar em um item."

**Output esperado (resumo):**

- Componente `ProductListScreen.tsx` usando `FlatList` com `onEndReached` para paginação
- Hook customizado `useProducts.ts` encapsulando a chamada Axios com estados de loading/erro/dados
- Tratamento de estado vazio ("nenhum produto encontrado") e estado de erro com botão de retry
- Navegação para `ProductDetailsScreen` via `navigation.navigate("ProductDetails", { id })`
- Nota final assinalando que a URL da API foi deixada como placeholder e que o exemplo assume Expo + TypeScript + Context API, por não terem sido especificados
</content>
