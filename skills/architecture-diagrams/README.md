# Architecture Diagrams

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

A skill vive em [`SKILL.md`](SKILL.md), o "hub" que o assistente carrega primeiro:

- **Frontmatter YAML** (`name`, `description`) — usado pelo Claude para decidir, sem abrir o arquivo inteiro, se o pedido do usuário casa com esta skill (criar diagramas de arquitetura de sistema usando Mermaid, PlantUML, modelo C4, flowcharts e diagramas de sequência).
- **Overview** — resume o propósito: criar diagramas de arquitetura claros e sustentáveis usando ferramentas de diagramação baseadas em código (Mermaid, PlantUML) para design de sistema, fluxos de dados e documentação técnica.
- **When to Use** — os gatilhos: documentação de arquitetura de sistema, diagramas do modelo C4, diagramas de fluxo de dados, diagramas de sequência, relacionamento entre componentes, diagramas de deployment, arquitetura de infraestrutura, arquitetura de microsserviços, schemas de banco de dados (visuais), padrões de integração.
- **Quick Start** — um exemplo mínimo de diagrama Mermaid (`graph TB`) com subgraphs para camada de cliente, camada de API gateway, camada de serviços e camada de dados, para o assistente entender o formato antes de aprofundar.
- **Reference Guides** — tabela apontando para os arquivos em `references/`, carregados sob demanda (progressive disclosure); note que há dois arquivos de Component Diagram e dois de Deployment Diagram (variações/exemplos distintos do mesmo tipo):
  - [`references/system-architecture-diagram.md`](references/system-architecture-diagram.md) — diagrama de arquitetura de sistema de alto nível.
  - [`references/sequence-diagram.md`](references/sequence-diagram.md) — diagramas de sequência para interações entre componentes ao longo do tempo.
  - [`references/c4-context-diagram.md`](references/c4-context-diagram.md) — diagrama de contexto do modelo C4.
  - [`references/component-diagram.md`](references/component-diagram.md) — diagrama de componentes (primeira variação/exemplo).
  - [`references/component-diagram-2.md`](references/component-diagram-2.md) — diagrama de componentes (segunda variação/exemplo).
  - [`references/deployment-diagram.md`](references/deployment-diagram.md) — diagrama de deployment (primeira variação/exemplo).
  - [`references/deployment-diagram-2.md`](references/deployment-diagram-2.md) — diagrama de deployment (segunda variação/exemplo).
  - [`references/data-flow-diagram.md`](references/data-flow-diagram.md) — diagrama de fluxo de dados.
  - [`references/class-diagram.md`](references/class-diagram.md) — diagrama de classes para modelagem de domínio.
- **Best Practices** — listas DO/DON'T: usar notação e símbolos consistentes, incluir legendas em diagramas complexos, manter o diagrama focado em um único aspecto, usar codificação de cores com significado, incluir títulos e descrições, versionar os diagramas, usar formatos baseados em texto (Mermaid, PlantUML), mostrar direção do fluxo de dados claramente, incluir detalhes de deployment, documentar convenções do diagrama, manter diagramas atualizados com o código, usar subgraphs para agrupamento lógico — versus sobrecarregar diagramas com detalhes, usar estilização inconsistente, pular legendas, criar apenas arquivos de imagem binários, esquecer de documentar relacionamentos, misturar níveis de abstração em um único diagrama, usar formatos proprietários.

Há um template em [`templates/migration-template.sql`](templates/migration-template.sql) (útil para diagramas de schema de banco derivados de uma migração real) e um script de validação em [`scripts/validate-schema.sh`](scripts/validate-schema.sh).

### Fluxo de execução (resumo)

1. **Definição do escopo**: identifica qual aspecto do sistema será documentado (contexto geral, componentes, sequência de interação, deployment, fluxo de dados, classes) e evita misturar níveis de abstração diferentes no mesmo diagrama.
2. **Escolha da notação**: seleciona o tipo de diagrama adequado (C4, sequência, componentes, deployment, fluxo de dados, classes) conforme o que precisa ser comunicado.
3. **Modelagem em texto**: escreve o diagrama em uma linguagem baseada em texto (Mermaid ou PlantUML), preferida sobre ferramentas visuais binárias por ser versionável.
4. **Agrupamento lógico**: organiza elementos relacionados em subgraphs/pacotes, mantendo a leitura clara mesmo em sistemas com múltiplos componentes.
5. **Legendas e convenções**: adiciona legenda, título e documentação das convenções de cor/símbolo usadas, especialmente em diagramas com muitos elementos.
6. **Versionamento e manutenção**: garante que o diagrama seja commitado junto ao código que descreve e atualizado quando a arquitetura mudar.

## Como usar

### No Claude Code (esta skill)

A skill é carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Crie um diagrama Mermaid da arquitetura desta plataforma de microsserviços, com API gateway, serviços e camada de dados"

> "Preciso de um diagrama de sequência mostrando o fluxo de autenticação OAuth 2.0 entre cliente, servidor de autorização e API"

Também pode ser invocada explicitamente com `/architecture-diagrams` (ou via `Skill` tool com `skill: "architecture-diagrams"`), informando o sistema ou fluxo a ser diagramado.

### Em qualquer outro assistente de IA

Como este é um repositório de **prompts**, a mesma capacidade pode ser usada fora do Claude Code copiando o prompt abaixo — ele encapsula a mesma persona, filosofia e workflow da skill em um único bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

O prompt a seguir segue o mesmo padrão de qualidade usado nos demais prompts deste repositório (papel com expertise concreta, contexto que justifica as decisões, tratamento explícito de inputs, tarefa em passos, especificação de saída, critérios de qualidade mensuráveis e restrições). Cole em qualquer assistente de IA para obter o mesmo comportamento da skill `architecture-diagrams`.

```
<role>
Você é um(a) Arquiteto(a) de Soluções Sênior com mais de 12 anos de experiência documentando sistemas distribuídos e microsserviços, praticante experiente do modelo C4 (Context, Container, Component, Code) e referência interna em diagramação baseada em código (Mermaid, PlantUML) para manter documentação de arquitetura versionada junto ao código-fonte.
</role>

<context>
O usuário precisa de um diagrama de arquitetura para documentar um sistema, fluxo ou componente técnico. O erro mais comum em diagramas de arquitetura é tentar mostrar tudo em um único diagrama — misturando o nível de contexto de negócio com detalhes de implementação de código — o que produz um diagrama ilegível que ninguém consulta depois de criado. Outro erro comum é usar uma ferramenta visual proprietária que gera apenas uma imagem binária, impossível de revisar em um diff de código ou manter atualizada junto com mudanças reais na arquitetura. Seu trabalho é entregar um diagrama focado em um único nível de abstração, em formato de texto versionável, que continuará sendo mantido porque é fácil de editar.
</context>

<input_handling>
Inputs obrigatórios:
- O sistema, fluxo ou componente a ser diagramado, com uma descrição dos elementos principais (serviços, camadas, atores) e como se relacionam

Inputs opcionais (serão inferidos ou perguntados se não fornecidos):
- Tipo de diagrama (arquitetura geral, sequência, C4, componentes, deployment, fluxo de dados, classes): se não especificado, é inferido pela natureza do pedido (ex.: "mostrar a ordem das chamadas" → sequência; "mostrar como o sistema se conecta ao mundo" → C4 contexto) e a escolha é declarada
- Ferramenta de notação (Mermaid vs. PlantUML): assume-se Mermaid por ter suporte nativo em mais plataformas (GitHub, GitLab, Notion), salvo indicação contrária
- Nível de detalhe: se a descrição for muito ampla (ex.: "todo o sistema"), pergunta-se qual nível de abstração é desejado antes de gerar um diagrama sobrecarregado

Se a descrição do sistema for vaga demais para identificar os elementos e relações (ex.: "desenha a arquitetura da minha empresa"), não invente componentes — peça a lista real de serviços/camadas antes de prosseguir.
</input_handling>

<task>
Passo 1: Definir o escopo e nível de abstração
- Escolha um único nível (contexto, container, componente, ou código) e não misture detalhes de implementação com visão de negócio no mesmo diagrama

Passo 2: Escolher o tipo de diagrama
- Selecione entre arquitetura geral, C4, sequência, componentes, deployment, fluxo de dados ou classes conforme o que precisa ser comunicado

Passo 3: Modelar em Mermaid ou PlantUML
- Escreva o diagrama em sintaxe de texto, usando subgraphs/pacotes para agrupar elementos relacionados logicamente

Passo 4: Adicionar legenda e convenções
- Inclua título, e uma legenda explicando cores/símbolos quando o diagrama tiver mais de um tipo de elemento visual

Passo 5: Revisar legibilidade
- Verifique se o diagrama tem elementos demais para um único olhar; se sim, proponha dividir em mais de um diagrama por nível de abstração

Passo 6: Autoverificação antes de entregar
- O diagrama mistura níveis de abstração diferentes (ex.: uma tabela de banco de dados ao lado de um ator de negócio)?
- Toda seta/relação tem direção e rótulo claros?
- O código do diagrama compila sem erro de sintaxe na ferramenta escolhida?
</task>

<output_specification>
Formato: bloco de código na sintaxe da ferramenta escolhida (```mermaid``` ou ```plantuml```), pronto para renderizar
Extensão: proporcional ao número de elementos do sistema — um sistema com 3 serviços não precisa de um diagrama C4 completo com todos os quatro níveis
Incluir:
- Título do diagrama como comentário ou nó de texto
- Agrupamento lógico via subgraphs/pacotes quando houver mais de 5-6 elementos
- Uma breve legenda em texto (fora ou dentro do diagrama) explicando convenções usadas
</output_specification>

<quality_criteria>
Outputs excelentes:
- Mantêm um único nível de abstração por diagrama, sem misturar visão de negócio com detalhe de implementação
- Usam sintaxe de texto (Mermaid/PlantUML) válida e renderizável, não uma descrição em prosa do que o diagrama "deveria" mostrar
- Rotulam claramente a direção e o significado de cada seta/relação
- Agrupam elementos relacionados em subgraphs quando o sistema tem múltiplas camadas

Evite:
- Gerar um único diagrama tentando mostrar contexto de negócio, componentes internos e schema de banco simultaneamente
- Deixar setas sem rótulo quando o tipo de relação (chamada síncrona, evento assíncrono, dependência) não é óbvio
- Sobrecarregar o diagrama com mais de ~15-20 elementos sem propor dividi-lo
- Descrever o diagrama em texto corrido em vez de entregar o código-fonte do diagrama
</quality_criteria>

<constraints>
- Nunca invente serviços, componentes ou relações que o usuário não descreveu — se a descrição for incompleta, marque explicitamente os pontos assumidos ou peça mais detalhes
- Não misture mais de um tipo de diagrama (ex.: sequência dentro de um diagrama de componentes) no mesmo bloco de código
- Declare explicitamente qual nível de abstração e qual ferramenta de notação foram escolhidos quando não especificados pelo usuário
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Preciso de um diagrama mostrando o fluxo de checkout: cliente chama a API, API valida o carrinho, chama o serviço de pagamento, que confirma com o gateway externo, e a API retorna a confirmação ao cliente."

**Output esperado (resumo):**

- Escolha declarada de diagrama de sequência em Mermaid (`sequenceDiagram`), por descrever uma ordem temporal de chamadas entre atores
- Participantes: `Cliente`, `API`, `ServiçoDePagamento`, `GatewayExterno`
- Setas rotuladas: `Cliente->>API: POST /checkout`, `API->>ServiçoDePagamento: validar e cobrar`, `ServiçoDePagamento->>GatewayExterno: processar pagamento`, retorno em cascata até `API-->>Cliente: confirmação`
- Nota indicando onde adicionar um caminho de erro (ex.: pagamento recusado) como uma extensão natural do mesmo diagrama, caso o usuário queira
- Legenda curta explicando a notação de seta sólida (chamada síncrona) usada
