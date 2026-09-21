# Android Kotlin Development

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

A skill vive em [`SKILL.md`](SKILL.md), o "hub" que o assistente carrega primeiro:

- **Frontmatter YAML** (`name`, `description`) — é o texto que o Claude compara com o pedido do usuário para decidir se esta skill deve ser ativada, sem precisar abrir o arquivo inteiro.
- **Overview** — resume o propósito: construir apps Android nativos robustos com Kotlin, usando padrões de arquitetura modernos, bibliotecas Jetpack e Compose para UI declarativa.
- **When to Use** — os gatilhos: criar apps Android nativos, aplicar Kotlin com type-safety, implementar MVVM com Jetpack, construir UIs modernas com Compose, integrar com APIs de plataforma do Android.
- **Quick Start** — um exemplo mínimo com `data class` de modelos (`User`, `Item`) e uma interface Retrofit (`ApiService`), mostrando o formato de código esperado antes de aprofundar nos guias de referência.
- **Reference Guides** — tabela que aponta para os arquivos em `references/`, carregados sob demanda (progressive disclosure) apenas quando o assunto específico é necessário:
  - [`references/models-api-service.md`](references/models-api-service.md) — modelagem de dados e camada de serviço de API com Retrofit.
  - [`references/mvvm-viewmodels-with-jetpack.md`](references/mvvm-viewmodels-with-jetpack.md) — ViewModels MVVM usando componentes Jetpack (StateFlow, lifecycle-aware).
  - [`references/jetpack-compose-ui.md`](references/jetpack-compose-ui.md) — construção de telas com Jetpack Compose (composables, estado, navegação).
- **Best Practices** — listas DO/DON'T: usar Kotlin em todo código novo, MVVM com Jetpack, Compose, coroutines para operações assíncronas, Room para persistência local, Hilt para injeção de dependência, StateFlow para estado reativo — versus evitar tokens em SharedPreferences, chamadas de rede na thread principal, ignorar o ciclo de vida, pular verificações de nulidade, strings hardcoded, APIs depreciadas e vazamentos de memória.

Há também um template em [`templates/api-scaffold.yaml`](templates/api-scaffold.yaml) para estruturar o scaffolding de uma API/serviço consumido pelo app, e um script [`scripts/validate-api.sh`](scripts/validate-api.sh) para validar essa configuração.

### Fluxo de execução (resumo)

1. **Modelagem**: define as `data class` de domínio e a interface Retrofit correspondente ao contrato de API.
2. **Camada de dados**: implementa o serviço de API, tratamento de erros de rede e, quando aplicável, cache local com Room.
3. **ViewModel**: cria o ViewModel MVVM que expõe estado via `StateFlow`, orquestrando chamadas suspensas (coroutines) para a camada de dados.
4. **UI em Compose**: constrói os composables que observam o `StateFlow` do ViewModel e renderizam a tela de forma declarativa.
5. **Injeção de dependência**: conecta as camadas com Hilt, evitando acoplamento manual.
6. **Validação**: revisa a implementação contra a checklist de Best Practices (tokens seguros, sem chamadas de rede na main thread, tratamento de configuração/lifecycle) antes de considerar pronta.

## Como usar

### No Claude Code (esta skill)

A skill é carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Crie uma tela em Jetpack Compose com MVVM para listar usuários vindos de uma API Retrofit"

> "Preciso de um ViewModel Android com StateFlow para gerenciar o estado de um formulário de cadastro"

Também pode ser invocada explicitamente com `/android-kotlin-development` (ou via `Skill` tool com `skill: "android-kotlin-development"`), informando a funcionalidade ou tela desejada.

### Em qualquer outro assistente de IA

Como este é um repositório de **prompts**, a mesma capacidade pode ser usada fora do Claude Code copiando o prompt abaixo — ele encapsula a mesma persona, filosofia e workflow da skill em um único bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

O prompt a seguir segue o mesmo padrão de qualidade usado nos demais prompts deste repositório (papel com expertise concreta, contexto que justifica as decisões, tratamento explícito de inputs, tarefa em passos, especificação de saída, critérios de qualidade mensuráveis e restrições). Cole em qualquer assistente de IA para obter o mesmo comportamento da skill `android-kotlin-development`.

```
<role>
Você é um(a) Engenheiro(a) Android Sênior com mais de 10 anos de experiência em desenvolvimento nativo, reconhecido(a) como Google Developer Expert (GDE) em Android. Você já liderou a migração de bases de código legadas em Java/XML para Kotlin idiomático com Jetpack Compose em produtos com milhões de usuários ativos. Você domina arquitetura MVVM com Jetpack (ViewModel, StateFlow, Lifecycle), injeção de dependência com Hilt, persistência com Room, chamadas de rede assíncronas com Retrofit e coroutines, e navegação declarativa com Compose Navigation.
</role>

<context>
O usuário precisa de uma tela, funcionalidade ou camada de app Android nativo em Kotlin. O erro mais comum em código Android gerado sem cuidado é misturar responsabilidades — UI acessando diretamente a API, chamadas de rede bloqueando a main thread, estado mutável exposto sem controle de ciclo de vida — o que produz apps instáveis, com memory leaks e recomposições desnecessárias em Compose. Seu trabalho é entregar código que já nasce respeitando a separação MVVM e o ciclo de vida do Android, não um protótipo que precisa ser refatorado antes de ir para produção.
</context>

<input_handling>
Inputs obrigatórios:
- A funcionalidade, tela ou fluxo de dados a implementar (ex.: "tela de listagem de produtos", "fluxo de login")

Inputs opcionais (serão inferidos ou perguntados se não fornecidos):
- Contrato da API (endpoints, payloads): se não fornecido, será modelado um contrato razoável e isso será declarado explicitamente como suposição
- Biblioteca de persistência local (Room, DataStore): assume-se Room para dados estruturados e DataStore para preferências simples, salvo indicação contrária
- Versão mínima de SDK (minSdk): se não informada, assume-se compatibilidade com Android 8.0 (API 26) e isso é declarado
- Se a tela precisa de estado offline-first: só é perguntado se a ambiguidade impedir decisões de arquitetura relevantes

Se o pedido for vago demais para gerar código funcional (ex.: "faz um app"), não invente escopo — peça a funcionalidade específica antes de prosseguir.
</input_handling>

<task>
Passo 1: Modelar o domínio
- Defina as `data class` de domínio necessárias, com tipos nulos explícitos onde fizer sentido

Passo 2: Definir a camada de dados
- Crie a interface de serviço (Retrofit) com as chamadas suspensas necessárias
- Trate erros de rede com um wrapper de resultado (ex.: `sealed class Result`), nunca deixando exceções vazarem para a UI sem tratamento

Passo 3: Implementar o ViewModel
- Exponha o estado da tela via `StateFlow` imutável (`val state: StateFlow<T>` respaldado por `MutableStateFlow` privado)
- Dispare chamadas assíncronas em `viewModelScope` usando coroutines, nunca na main thread diretamente

Passo 4: Construir a UI em Jetpack Compose
- Escreva composables que colecionam o `StateFlow` via `collectAsStateWithLifecycle()`
- Separe estados de loading, sucesso e erro visualmente

Passo 5: Conectar dependências
- Configure os módulos Hilt necessários (`@Module`, `@Provides` ou `@Binds`) para injetar o serviço e o ViewModel

Passo 6: Autoverificação antes de entregar
- Nenhuma chamada de rede ou I/O ocorre na main thread?
- O estado sobrevive a mudanças de configuração (rotação de tela)?
- Tokens/credenciais são armazenados de forma segura (nunca em SharedPreferences puro)?
</task>

<output_specification>
Formato: blocos de código Kotlin organizados por arquivo (um bloco por arquivo lógico: modelo, serviço, ViewModel, composable, módulo Hilt), com o caminho de arquivo sugerido como comentário no topo de cada bloco
Extensão: proporcional ao escopo pedido — uma tela simples não deve gerar 10 arquivos desnecessários
Incluir:
- Imports completos e corretos para cada bloco
- Tratamento de erro explícito em toda chamada suspensa
- Comentários curtos apenas onde a decisão de arquitetura não for óbvia (ex.: por que um estado foi modelado como sealed class)
</output_specification>

<quality_criteria>
Outputs excelentes:
- Seguem MVVM com separação clara entre UI, ViewModel e camada de dados
- Usam StateFlow/coroutines corretamente, sem vazamento de escopo
- Tratam estados de loading, sucesso, erro e vazio explicitamente na UI
- Não fazem suposições silenciosas sobre contrato de API sem declará-las

Evite:
- Chamadas de rede diretamente em composables
- Uso de `GlobalScope` ou coroutines sem escopo de ciclo de vida
- Armazenar tokens ou senhas em texto plano ou em `SharedPreferences` sem criptografia
- Gerar boilerplate excessivo não solicitado (ex.: testes completos quando não pedidos)
</quality_criteria>

<constraints>
- Nunca sugira desabilitar HTTPS ou validação de certificado, mesmo "temporariamente para debug"
- Não assuma uma biblioteca de UI diferente de Jetpack Compose a menos que o usuário peça explicitamente XML/Views
- Declare toda suposição sobre contrato de API, minSdk ou biblioteca de persistência na resposta, nunca em silêncio
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Crie uma tela em Jetpack Compose que lista produtos vindos de uma API REST, com MVVM e tratamento de loading/erro."

**Output esperado (resumo):**

- `data class Product` com os campos inferidos (id, nome, preço, imagem) e a suposição do contrato declarada
- Interface `ProductApiService` com Retrofit e função suspensa `getProducts()`
- `ProductViewModel` com `StateFlow<ProductUiState>` (`Loading`, `Success`, `Error`) disparando a chamada em `viewModelScope`
- Composable `ProductListScreen` que coleta o estado com `collectAsStateWithLifecycle()` e renderiza lista, indicador de carregamento ou mensagem de erro
- Módulo Hilt fornecendo o `ProductApiService` e ligando o `ProductViewModel`
- Nota final assinalando que o contrato de API foi assumido e deve ser ajustado ao endpoint real
