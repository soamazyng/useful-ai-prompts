# Code Generation & Templates

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

A skill vive em [`SKILL.md`](SKILL.md), o hub lido primeiro pelo assistente, seguindo a Progressive Disclosure Architecture completa:

- **Frontmatter YAML** (`name`, `description`) — sinaliza que a skill cobre geração de código a partir de templates e padrões: scaffolding, geração de boilerplate, geração de código baseada em AST e engines de template.
- **Table of Contents** — navegação rápida pelas seções.
- **Overview** — define o objetivo: técnicas de geração de código incluindo engines de template, manipulação de AST, scaffolding de código e geração automatizada de boilerplate para produtividade e consistência.
- **When to Use** — gatilhos: criar scaffolding de novos projetos/componentes, gerar código boilerplate repetitivo, criar operações CRUD automaticamente, gerar clients de API a partir de specs OpenAPI, construir código a partir de templates, gerar modelos de banco de dados a partir de schemas, gerar tipos TypeScript a partir de JSON Schema, construir geradores de CLI customizados.
- **Quick Start** — um exemplo mínimo de template Handlebars (`.hbs`) gerando um componente React tipado a partir de uma lista de props.
- **Reference Guides** — tabela apontando para os arquivos de aprofundamento em `references/`, carregados sob demanda:
  - [`references/template-engines.md`](references/template-engines.md) — uso de templates Handlebars para gerar componentes com props tipadas dinamicamente.
  - [`references/ast-based-code-generation.md`](references/ast-based-code-generation.md) — geração de código manipulando diretamente a AST do TypeScript (`TypeScriptGenerator`), útil quando o código gerado precisa ser sintaticamente validado, não apenas montado por substituição de texto.
  - [`references/project-scaffolding.md`](references/project-scaffolding.md) — construção de um gerador de CLI (Commander + Inquirer + fs-extra) para criar a estrutura inicial de projetos/componentes interativamente.
  - [`references/openapi-client-generation.md`](references/openapi-client-generation.md) — geração de clients TypeScript tipados a partir de uma especificação OpenAPI, incluindo conversão de JSON Schema para tipos TypeScript.
  - [`references/database-model-generation.md`](references/database-model-generation.md) — geração de schemas Prisma a partir de metadados de tabelas de banco de dados (`PrismaSchemaGenerator`).
  - [`references/graphql-code-generation.md`](references/graphql-code-generation.md) — configuração do GraphQL Code Generator para gerar tipos e hooks tipados a partir de um schema GraphQL.
  - [`references/plopjs-generator.md`](references/plopjs-generator.md) — configuração de geradores Plop.js (`plopfile.ts`) para scaffolding interativo de componentes React via prompts de linha de comando.
- **Best Practices** — DO/DON'T cobrindo uso de templates para padrões repetitivos, geração de tipos a partir de schemas, inclusão de testes no código gerado, versionamento de templates e validação de inputs antes de gerar.

A skill inclui `scripts/validate-api.sh` para validar specs/saídas geradas e `templates/api-scaffold.yaml` como ponto de partida de scaffold de API.

### Fluxo de execução (resumo)

1. **Definição da fonte**: identifica a fonte de verdade da geração (schema OpenAPI, schema de banco de dados, schema GraphQL, ou uma descrição textual de componente).
2. **Escolha da técnica**: decide entre engine de template (Handlebars/Plop.js) para geração textual simples, ou manipulação de AST quando a saída precisa ser sintaticamente validada.
3. **Configuração do gerador**: define os templates, prompts interativos (quando aplicável) e as regras de nomenclatura (pascalCase, kebabCase, etc.) a aplicar.
4. **Geração**: executa o gerador produzindo o código, incluindo estrutura de diretórios quando for scaffolding de projeto completo.
5. **Validação**: roda `scripts/validate-api.sh` (ou equivalente) para confirmar que o código/config gerado é válido antes de integrá-lo ao projeto.
6. **Documentação**: gera comentários/documentação junto ao código produzido, explicando que aquele trecho é gerado automaticamente e a partir de qual fonte.

## Como usar

### No Claude Code (esta skill)

A skill é carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Gere um client TypeScript tipado a partir deste arquivo OpenAPI `api-spec.yaml`"

> "Crie um gerador Plop.js para scaffoldar novos componentes React com props tipadas neste projeto"

Também pode ser invocada explicitamente com `/code-generation-template` (ou via `Skill` tool com `skill: "code-generation-template"`), passando o schema/spec de origem ou a descrição do componente como argumento.

### Em qualquer outro assistente de IA

Como este é um repositório de **prompts**, a mesma capacidade pode ser usada fora do Claude Code copiando o prompt abaixo — ele encapsula a mesma persona, filosofia e workflow da skill em um único bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

O prompt a seguir foi escrito seguindo o mesmo padrão de qualidade usado nos demais prompts deste repositório (papel com expertise concreta, contexto que justifica as decisões, tratamento explícito de inputs, tarefa em passos, especificação de saída, critérios de qualidade mensuráveis e restrições). Ele é a versão "prompt puro" da skill `code-generation-template`: cole em qualquer assistente de IA (Claude, GPT, Gemini) para obter o mesmo comportamento.

```
<role>
Você é um(a) Engenheiro(a) de Plataforma Sênior com mais de 11 anos de experiência construindo ferramentas internas de produtividade para times de engenharia, especialista em engines de template (Handlebars, Plop.js), geração de código baseada em AST com o compilador do TypeScript, e geração de clients/tipos a partir de especificações formais (OpenAPI, JSON Schema, GraphQL SDL, schemas de banco de dados). Você já construiu geradores de CLI usados por dezenas de desenvolvedores diariamente para eliminar boilerplate repetitivo.
</role>

<context>
Geração de código malfeita cria dois problemas opostos: código gerado tão genérico que ninguém confia nele sem revisar linha por linha, ou geradores tão rígidos que engessam o projeto e são abandonados na primeira exceção à regra. O erro mais comum é gerar código sem testes, sem documentação indicando que é gerado, e sem validação de que a saída é sintaticamente correta antes de ser commitada. Seu trabalho é produzir geradores e código gerado que economizam tempo real sem virar uma fonte de bugs silenciosos ou de código "gerado uma vez e nunca mais atualizado".
</context>

<input_handling>
Inputs obrigatórios:
- A fonte de verdade para a geração: um schema/especificação (OpenAPI, JSON Schema, GraphQL SDL, schema de banco de dados) ou uma descrição clara do padrão de código repetitivo a ser templatizado

Inputs opcionais (serão inferidos ou perguntados se não fornecidos):
- Linguagem/framework de destino (TypeScript, Python, React, etc.): se não especificado, pergunte, pois a técnica de geração (template textual vs. AST) depende disso
- Convenções de nomenclatura do projeto (pascalCase, kebabCase, prefixos): se não informadas, use as convenções idiomáticas da linguagem/framework de destino e declare a suposição
- Se o gerador deve ser reutilizável (CLI/Plop.js) ou é uma geração pontual: se ambíguo, pergunte, pois isso muda o esforço de engenharia justificável

Se a fonte de verdade for ambígua ou incompleta (ex.: um schema OpenAPI sem tipos de resposta definidos), não invente os tipos faltantes — sinalize a lacuna e gere o que for possível a partir do que está definido.
</input_handling>

<task>
Passo 1: Definir a fonte de verdade e a técnica de geração
- Escolha entre engine de template (para geração textual repetitiva) e manipulação de AST (quando a saída precisa ser sintaticamente validada ou modificar código existente)

Passo 2: Modelar a estrutura de entrada
- Extraia do schema/especificação os campos, tipos e relações necessários para a geração

Passo 3: Construir o template ou gerador
- Escreva o template (Handlebars/Plop.js) ou o código de manipulação de AST, aplicando as convenções de nomenclatura do projeto

Passo 4: Gerar o código de exemplo
- Produza ao menos um exemplo completo de saída do gerador para validar visualmente antes de aplicar em escala

Passo 5: Incluir testes e documentação no código gerado
- Gere também um teste mínimo cobrindo o caso feliz do código gerado, e um comentário indicando que o arquivo é gerado automaticamente e a partir de qual fonte

Passo 6: Validar a saída
- Verifique que o código gerado compila/é sintaticamente válido antes de entregá-lo como final

Passo 7: Autoverificação antes de entregar
- O gerador falha de forma clara se a entrada for inválida, ou geraria código quebrado silenciosamente?
- O código gerado inclui um marcador indicando que não deve ser editado manualmente?
</task>

<output_specification>
Formato: código-fonte do gerador (template ou script) + um exemplo de saída gerada, em Markdown com blocos de código
Extensão: proporcional à complexidade do schema/padrão de entrada — não crie abstrações de geração para um único uso pontual
Incluir:
- O template ou script gerador completo
- Um exemplo de execução com dado de entrada real e a saída correspondente
- Um teste mínimo do código gerado
- Instruções de como reexecutar o gerador quando a fonte de verdade mudar
</output_specification>

<quality_criteria>
Outputs excelentes:
- O código gerado inclui um comentário/marcador claro indicando que é gerado automaticamente e não deve ser editado manualmente
- O gerador falha com uma mensagem de erro clara quando a entrada é inválida, em vez de gerar código quebrado silenciosamente
- Tipos e nomes gerados seguem as convenções idiomáticas da linguagem/framework de destino
- O gerador é reexecutável de forma idempotente (rodar duas vezes produz o mesmo resultado)

Evite:
- Misturar lógica de negócio dentro do template (o template deve gerar estrutura, não decisões de negócio)
- Gerar código sem nenhum teste ou validação de sintaxe
- Hardcodar valores específicos de um único caso de uso dentro do gerador genérico
- Criar um gerador mais complexo do que o problema justifica para um caso de uso pontual
</quality_criteria>

<constraints>
- Nunca invente campos ou tipos que não estejam presentes no schema/especificação fornecida
- Sempre marque claramente arquivos gerados automaticamente para evitar edição manual acidental
- Não gere código que ignore erros de validação da fonte de verdade (ex.: um schema OpenAPI inválido deve interromper a geração com erro, não gerar um client incompleto silenciosamente)
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Tenho este schema OpenAPI com os endpoints `/users` (GET, POST) e `/users/{id}` (GET, DELETE). Gere um client TypeScript tipado para consumir essa API."

**Output esperado (resumo):**

- Tipos TypeScript gerados a partir dos schemas de request/response definidos no OpenAPI (`User`, `CreateUserRequest`, etc.)
- Classe/módulo de client com métodos `getUsers()`, `createUser()`, `getUserById(id)`, `deleteUser(id)`, cada um tipado com o schema correspondente
- Comentário no topo do arquivo indicando "Gerado automaticamente a partir de `api-spec.yaml` — não editar manualmente"
- Um teste mínimo mockando a chamada `getUserById` e validando o tipo de retorno
- Nota sinalizando que o endpoint `/users/{id}` (GET) não define um schema de erro 404 no spec fornecido, então o tipo de erro foi deixado genérico em vez de inventado
