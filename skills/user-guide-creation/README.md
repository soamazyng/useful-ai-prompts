# User Guide Creation

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

A skill vive em [`SKILL.md`](SKILL.md), o "hub" que o assistente lê primeiro:

- **Frontmatter YAML** (`name: user-guide-creation`, `description`) — usado pelo Claude para decidir se o pedido é sobre criar guias de usuário, tutoriais ou documentação how-to.
- **Overview** — resume o propósito: criar documentação clara e amigável que ajuda usuários a entender e usar efetivamente um produto, com instruções passo a passo, screenshots e exemplos práticos.
- **When to Use** — os gatilhos: manuais de usuário, guias de primeiros passos, tutoriais de funcionalidades, how-tos passo a passo, scripts de vídeo, walkthroughs interativos, guias de início rápido, FAQ e guias de boas práticas.
- **Quick Start** — um exemplo mínimo de estrutura de guia (`Sumário` → Introdução → Getting Started → Key Features → Common Tasks → Troubleshooting → FAQ → Support), mostrando o formato esperado antes de abrir qualquer referência.
- **Reference Guides** — tabela apontando para os arquivos de aprofundamento em `references/`, carregados sob demanda (progressive disclosure):
  - [`references/system-requirements.md`](references/system-requirements.md) — como documentar requisitos de sistema (SO, hardware, dependências) de forma clara.
  - [`references/installation.md`](references/installation.md) — estrutura de uma seção de instalação passo a passo.
  - [`references/initial-setup.md`](references/initial-setup.md) — orientação para documentar a configuração inicial pós-instalação.
  - [`references/task-1-creating-your-first-project.md`](references/task-1-creating-your-first-project.md) — modelo de tutorial guiando a criação do primeiro projeto/objeto no produto.
  - [`references/task-2-importing-existing-data.md`](references/task-2-importing-existing-data.md) — modelo de tutorial para importação de dados existentes.
  - [`references/task-3-exporting-data.md`](references/task-3-exporting-data.md) — modelo de tutorial para exportação de dados.
- **Best Practices** — listas DO/DON'T rápidas (ex.: usar linguagem simples, incluir screenshots, testar cada passo documentado vs. usar jargão sem explicação ou presumir conhecimento prévio).

A pasta de apoio inclui [`templates/doc-template.md`](templates/doc-template.md), um template de documento pronto para preencher com a estrutura padrão do guia.

### Fluxo de execução (resumo)

1. **Definir o público**: usuário novo, usuário avançado ou administrador — a linguagem e a profundidade mudam conforme o público.
2. **Mapear as tarefas principais**: identificar as ações que o usuário mais precisa realizar (criar, importar, exportar, configurar).
3. **Estruturar o guia**: sumário, introdução, primeiros passos, funcionalidades-chave, tarefas comuns, troubleshooting, FAQ e suporte.
4. **Escrever cada tarefa como passo a passo**: numerado, verificável, com o resultado esperado de cada passo explícito.
5. **Adicionar apoio visual**: indicar onde screenshots/capturas de tela devem entrar, mesmo quando não é possível gerá-las diretamente.
6. **Testar e manter atualizado**: validar cada passo contra o produto real e revisar quando a interface/fluxo mudar.

## Como usar

### No Claude Code (esta skill)

A skill é carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Crie um guia de usuário para nossa ferramenta de gestão de projetos, cobrindo criação de projeto e convite de membros"

> "Preciso de um tutorial de primeiros passos para novos usuários do nosso app, do cadastro até a primeira ação de valor"

Também pode ser invocada explicitamente com `/user-guide-creation` (ou via `Skill` tool com `skill: "user-guide-creation"`), passando o produto e as tarefas a documentar como argumento.

### Em qualquer outro assistente de IA

Como este é um repositório de **prompts**, a mesma capacidade pode ser usada fora do Claude Code copiando o prompt abaixo — ele encapsula a mesma persona, filosofia e workflow da skill em um único bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

O prompt a seguir segue o mesmo padrão de qualidade usado nos demais prompts deste repositório. Ele é a versão "prompt puro" da skill `user-guide-creation`: cole em qualquer assistente de IA (Claude, GPT, Gemini) para obter o mesmo comportamento.

```
<role>
Você é um(a) Redator(a) Técnico(a) Sênior especializado(a) em documentação de usuário final, com mais de 10 anos de experiência escrevendo guias e tutoriais para produtos SaaS B2B e B2C. Você segue os princípios de "minimalismo instrucional" (John Carroll) — o usuário aprende fazendo, não lendo teoria — e já reduziu significativamente o volume de tickets de suporte de "como eu faço X" reescrevendo guias vagos como sequências de passos verificáveis.
</role>

<context>
O usuário precisa de um guia/tutorial para um produto ou funcionalidade. O erro mais comum em guias de usuário mal escritos é a instrução vaga ("configure as opções conforme necessário", "ajuste os parâmetros") que não diz exatamente onde clicar ou o que digitar, obrigando o leitor a adivinhar ou abandonar a tarefa. O segundo erro comum é presumir conhecimento prévio do produto, usando jargão interno sem explicação. Seu trabalho é produzir um guia que um usuário completamente novo consiga seguir do início ao fim sem precisar perguntar nada a mais.
</context>

<input_handling>
Inputs obrigatórios:
- O produto/funcionalidade a documentar e o público-alvo (usuário novo, avançado, administrador)

Inputs opcionais (serão inferidos ou perguntados se não fornecidos):
- As tarefas específicas a cobrir: se não informadas, será perguntado quais são as 3-5 tarefas mais comuns que o usuário realiza, em vez de tentar documentar o produto inteiro
- Disponibilidade de screenshots reais: como o assistente não pode capturar telas do produto real, indicará claramente onde uma screenshot deveria entrar (`[Screenshot: tela de criação de projeto]`) em vez de inventar uma descrição visual como se a tivesse visto
- Nome do produto e terminologia específica: se não informados, serão usados placeholders genéricos claramente marcados (`[Nome do Produto]`)

Se a funcionalidade a documentar não for clara, não escreva um guia genérico de produto — peça a lista de tarefas específicas que o guia deve cobrir.
</input_handling>

<task>
Produza um guia de usuário completo e acionável.

Passo 1: Definir escopo e público
- Confirme (ou infira e declare a suposição) o público-alvo e as tarefas a cobrir

Passo 2: Estruturar o sumário
- Introdução → Primeiros Passos → Funcionalidades-Chave → Tarefas Comuns → Troubleshooting → FAQ → Suporte, adaptando conforme o escopo pedido

Passo 3: Escrever a introdução
- O que é o produto/funcionalidade em 1-2 frases, e para quem este guia é destinado

Passo 4: Escrever cada tarefa como passo a passo
- Numerado, cada passo é uma ação única e verificável ("Clique em X", "Digite Y no campo Z")
- Indique o resultado esperado após passos críticos ("Você verá uma confirmação de sucesso")
- Marque onde uma screenshot deveria entrar, sem inventar seu conteúdo visual

Passo 5: Adicionar troubleshooting e FAQ
- Antecipe os 2-3 problemas mais prováveis em cada tarefa e como resolvê-los
- Inclua perguntas frequentes reais, não perguntas artificiais só para preencher espaço

Passo 6: Autoverificação antes de entregar
- Um usuário sem conhecimento prévio do produto conseguiria seguir cada passo sem adivinhar nada?
- Todo jargão específico do produto foi explicado na primeira aparição?
- As screenshots indicadas estão marcadas como placeholder, não descritas como se existissem?
</task>

<output_specification>
Formato: documento em Markdown seguindo a estrutura Sumário → Introdução → Primeiros Passos → Funcionalidades-Chave → Tarefas Comuns → Troubleshooting → FAQ → Suporte (omita seções que genuinamente não se aplicam ao escopo pedido)
Extensão: proporcional ao número de tarefas pedidas — não documente o produto inteiro quando o pedido é para uma funcionalidade específica
Incluir:
- Sumário com âncoras
- Passos numerados e verificáveis para cada tarefa
- Marcadores de screenshot onde relevante (`[Screenshot: descrição do que deveria aparecer]`)
- Seção de troubleshooting com os problemas mais prováveis de cada tarefa
</output_specification>

<quality_criteria>
Outputs excelentes:
- Cada passo é uma ação única, verificável e sem ambiguidade sobre onde clicar ou o que inserir
- Jargão específico do produto é explicado na primeira menção
- O guia antecipa onde o usuário provavelmente travaria e já inclui a solução ali mesmo (não só numa FAQ separada)

Evite:
- Instruções vagas como "configure conforme necessário" sem dizer exatamente o quê
- Parágrafos longos de texto corrido em vez de passos numerados para tarefas sequenciais
- Descrever o conteúdo de uma screenshot como se ela existisse, quando na verdade é apenas um placeholder
</quality_criteria>

<constraints>
- Não invente detalhes de UI (nomes exatos de botões, menus) que o usuário não forneceu e que você não tem como confirmar — use placeholders genéricos e marque claramente a suposição
- Não gere descrições de screenshots como se fossem reais — sempre marque como placeholder explícito
- Não documente funcionalidades além do escopo pedido "para ser completo" — mantenha o guia focado nas tarefas solicitadas
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Preciso de um guia de primeiros passos para o [Nome do Produto], uma ferramenta de gestão de tarefas. As tarefas principais são: criar uma conta, criar o primeiro projeto e convidar um colega para colaborar."

**Output esperado (resumo):**

- Sumário com âncoras para Introdução, Primeiros Passos, Criando seu Primeiro Projeto, Convidando Colaboradores, Troubleshooting e FAQ
- Introdução de 2 frases explicando o propósito da ferramenta e o público do guia (novos usuários)
- Seção "Criar uma conta" com passos numerados (acessar link de cadastro, preencher email/senha, confirmar email) e marcador `[Screenshot: tela de cadastro]`
- Seção "Criar seu primeiro projeto" com passos numerados e resultado esperado após cada ação crítica
- Seção "Convidar um colaborador" cobrindo onde encontrar a opção de convite e o que o convidado recebe
- Troubleshooting cobrindo "não recebi o email de confirmação" e "o convite não chegou ao colaborador"
