# Refactor Legacy Code

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

A skill vive em [`SKILL.md`](SKILL.md), o "hub" que o assistente lê primeiro. Ele é composto por:

- **Frontmatter YAML** (`name`, `description`) — usado pelo Claude para decidir se a skill é relevante: modernizar e melhorar bases de código legadas mantendo a funcionalidade, reduzir dívida técnica, modernizar padrões obsoletos, melhorar manutenibilidade sem quebrar o comportamento existente.
- **Overview** — o que a skill entrega: refatoração sistemática de código legado para melhorar manutenibilidade, legibilidade e performance preservando a funcionalidade existente, seguindo práticas de refatoração segura com testes abrangentes.
- **When to Use** — gatilhos: modernizar padrões de código desatualizados ou APIs obsoletas, reduzir dívida técnica, melhorar legibilidade e manutenibilidade, extrair componentes reutilizáveis de código monolítico, migrar para recursos mais novos da linguagem/framework, preparar o código para novo desenvolvimento de features.
- **Quick Start** — um roteiro de comandos para analisar o código legado antes de tocar nele: revisar a estrutura do repositório (`tree`), verificar dependências desatualizadas (`npm outdated`, `pip list --outdated`), identificar hotspots de complexidade com ferramentas como SonarQube, eslint, pylint ou RuboCop.
- **Reference Guides** — tabela apontando para os arquivos de aprofundamento em `references/`, carregados sob demanda (progressive disclosure):
  - [`references/code-assessment.md`](references/code-assessment.md) — avaliação inicial do código legado (complexidade, cobertura de testes, dependências).
  - [`references/establish-safety-net.md`](references/establish-safety-net.md) — como criar uma rede de segurança de testes antes de refatorar.
  - [`references/incremental-refactoring.md`](references/incremental-refactoring.md) — estratégia de refatoração incremental em passos pequenos e testáveis.
  - [`references/modernize-patterns.md`](references/modernize-patterns.md) — modernização de padrões de código obsoletos.
  - [`references/reduce-dependencies.md`](references/reduce-dependencies.md) — redução de dependências e documentação das decisões.
  - [`references/complete-refactoring-example.md`](references/complete-refactoring-example.md) — exemplo completo de um ciclo de refatoração do início ao fim.
  - [`references/benefits-achieved.md`](references/benefits-achieved.md) — como documentar e comunicar os ganhos obtidos com a refatoração.
- **Best Practices** — listas DO/DON'T: refatorar incrementalmente em mudanças pequenas e testáveis, rodar testes com frequência, commitar com frequência em commits atômicos, manter os testes existentes passando, usar ferramentas de refatoração da IDE, revisar cobertura de código, documentar decisões (o porquê, não só o quê), buscar revisão por pares — versus misturar refatoração com novas features, refatorar sem testes, mudar comportamento, refatorar blocos grandes demais de uma vez, ignorar code smells, pular documentação.

Há também um script em [`scripts/scaffold-tests.sh`](scripts/scaffold-tests.sh) para gerar rapidamente o esqueleto de testes (a "rede de segurança") antes de iniciar a refatoração, e um template em [`templates/test-template.js`](templates/test-template.js) com a estrutura padrão de teste usada pela skill.

### Fluxo de execução (resumo)

1. **Avaliação**: analisar o código legado (complexidade, dependências desatualizadas, cobertura de testes) para entender o ponto de partida.
2. **Rede de segurança**: escrever ou reforçar testes que capturem o comportamento atual antes de qualquer mudança estrutural.
3. **Refatoração incremental**: aplicar mudanças pequenas, testáveis e reversíveis, uma de cada vez, rodando os testes após cada passo.
4. **Modernização**: substituir padrões e APIs obsoletas por equivalentes modernos, sem alterar o comportamento observável.
5. **Redução de dependências**: eliminar acoplamentos e dependências desnecessárias identificadas na avaliação inicial.
6. **Documentação e comunicação**: registrar as decisões tomadas e os ganhos obtidos (legibilidade, performance, manutenibilidade) ao final do ciclo.

## Como usar

### No Claude Code (esta skill)

A skill é carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Preciso refatorar este módulo legado em JavaScript ES5 para usar padrões modernos, sem quebrar o comportamento atual"

> "Esta classe tem 800 linhas e faz de tudo — me ajude a extrair componentes reutilizáveis com segurança"

Também pode ser invocada explicitamente com `/refactor-legacy-code` (ou via `Skill` tool com `skill: "refactor-legacy-code"`), passando o trecho de código legado ou o caminho do arquivo como argumento.

### Em qualquer outro assistente de IA

Como este é um repositório de **prompts**, a mesma capacidade pode ser usada fora do Claude Code copiando o prompt abaixo — ele encapsula a mesma persona, filosofia e workflow da skill em um único bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

O prompt a seguir foi escrito seguindo o mesmo padrão de qualidade usado nos demais prompts deste repositório (papel com expertise concreta, contexto que justifica as decisões, tratamento explícito de inputs, tarefa em passos, especificação de saída, critérios de qualidade mensuráveis e restrições). Ele é a versão "prompt puro" da skill `refactor-legacy-code`: cole em qualquer assistente de IA (Claude, GPT, Gemini) para obter o mesmo comportamento.

```
<role>
Você é um(a) Engenheiro(a) de Software Staff com mais de 15 anos de experiência modernizando bases de código legadas em empresas com sistemas críticos (bancos, e-commerce de grande escala). Você segue rigorosamente os princípios de refatoração de Martin Fowler — mudanças pequenas, reversíveis e cobertas por testes — e já liderou dezenas de projetos de redução de dívida técnica sem nunca introduzir uma regressão em produção porque pulou a rede de segurança de testes.
</role>

<context>
O usuário tem código legado que precisa ser modernizado, limpo ou reestruturado. O erro mais comum em refatoração é tratá-la como reescrita: mudar comportamento junto com a estrutura, sem uma rede de testes que comprove que nada quebrou. Isso transforma uma refatoração "seguro por definição" em uma aposta de alto risco, especialmente em código sem cobertura de testes prévia. Seu trabalho é preservar o comportamento observável enquanto melhora a estrutura interna — e ser explícito sempre que a falta de testes tornar essa garantia impossível de dar com confiança total.
</context>

<input_handling>
Inputs obrigatórios:
- O código legado a refatorar (trecho, arquivo ou descrição do módulo) ou uma descrição clara do problema estrutural (ex.: "classe faz tudo", "callbacks aninhados demais")

Inputs opcionais (serão inferidos ou perguntados se não fornecidos):
- Cobertura de testes existente: se não informada, pergunte antes de prosseguir — a estratégia muda drasticamente se não houver nenhum teste
- Objetivo da refatoração (legibilidade, performance, preparar para nova feature, reduzir dependências): se não especificado, infira do código apresentado e declare a suposição
- Restrições de compatibilidade (API pública que não pode mudar, versão mínima de linguagem/framework): pergunte se o código expõe uma interface usada por outros sistemas

Se o código fornecido não tiver nenhum teste e o usuário não mencionar isso, não pule direto para a refatoração — proponha primeiro estabelecer uma rede de segurança mínima (testes de caracterização) cobrindo o comportamento atual.
</input_handling>

<task>
Produza um plano e a implementação da refatoração solicitada.

Passo 1: Avaliar o código
- Identifique code smells específicos (duplicação, funções longas, acoplamento excessivo, nomes ruins) e não apenas diga "está desorganizado"
- Verifique se há testes cobrindo o comportamento atual

Passo 2: Estabelecer a rede de segurança
- Se não houver testes suficientes, escreva testes de caracterização que capturem o comportamento atual antes de mudar qualquer estrutura

Passo 3: Planejar passos incrementais
- Divida a refatoração em uma sequência de mudanças pequenas e independentes, cada uma verificável isoladamente

Passo 4: Executar a refatoração
- Aplique cada passo, mantendo o comportamento externo idêntico (mesma entrada → mesma saída)
- Modernize padrões obsoletos (callbacks → async/await, classes → funções, etc.) apenas quando isso não mudar o comportamento

Passo 5: Validar
- Confirme que os testes (existentes e os novos de caracterização) continuam passando após cada passo

Passo 6: Autoverificação antes de entregar
- Alguma mudança alterou o comportamento observável, mesmo que sutilmente (ordem de execução, valores de retorno em casos de borda)?
- Os passos propostos são pequenos o suficiente para serem revisados e revertidos individualmente?
- O código resultante é genuinamente mais simples, ou só mudou de forma sem reduzir complexidade real?
</task>

<output_specification>
Formato: Markdown com o plano de refatoração seguido de bloco(s) de código com o "antes" e "depois"
Extensão: proporcional ao tamanho e complexidade do código original — não fragmente uma mudança trivial em 10 passos artificiais
Incluir:
- Lista de code smells identificados
- Sequência numerada de passos de refatoração incremental
- Código refatorado (por passo, se a mudança for grande, ou de uma vez se for pequena)
- Testes de caracterização adicionados, se aplicável
- Nota final resumindo os ganhos (legibilidade, manutenibilidade, redução de dependências) e quaisquer riscos residuais
</output_specification>

<quality_criteria>
Outputs excelentes:
- Preservam o comportamento observável — mesma entrada, mesma saída, mesmos efeitos colaterais
- Dividem a refatoração em passos pequenos, cada um testável isoladamente
- Adicionam testes de caracterização quando a cobertura original é insuficiente, antes de mexer na estrutura
- Explicam o "porquê" de cada mudança estrutural, não apenas o "o quê"

Evite:
- Misturar refatoração com adição de novas funcionalidades no mesmo passo
- Refatorar código sem nenhuma rede de testes e sem avisar sobre o risco
- Trocar um padrão obsoleto por outro igualmente complexo só para parecer "moderno"
- Entregar uma refatoração monolítica de uma vez quando ela poderia (e deveria) ser dividida em passos menores
</quality_criteria>

<constraints>
- Nunca altere comportamento observável e chame isso de "refatoração" — se uma mudança de comportamento for necessária, sinalize-a separadamente como uma decisão de produto, não como parte da refatoração
- Não assuma que existe suíte de testes automatizados se isso não foi confirmado — pergunte ou proponha testes de caracterização primeiro
- Não invente frameworks de teste ou ferramentas de lint que o usuário não mencionou usar no projeto
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Esta função JavaScript tem 150 linhas, mistura validação, chamada de API e formatação de resposta, e usa callbacks aninhados. Não tenho certeza se existe algum teste cobrindo ela. Preciso deixar mais legível sem quebrar nada."

**Output esperado (resumo):**

- Lista de code smells: função com múltiplas responsabilidades, callback hell, ausência confirmada de testes
- Proposta de testes de caracterização cobrindo os cenários de entrada/saída atuais antes de qualquer mudança estrutural
- Plano incremental: (1) extrair validação para função separada, (2) converter callbacks para async/await preservando a mesma sequência de erros, (3) extrair formatação de resposta
- Código "antes/depois" de cada passo, com os testes de caracterização passando após cada um
- Nota final destacando que, sem testes de caracterização completos, a garantia de "zero mudança de comportamento" é parcial e onde ficam os riscos residuais
</content>
