# Code Metrics Analysis

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

A skill vive em [`SKILL.md`](SKILL.md), o hub lido primeiro pelo assistente, seguindo a Progressive Disclosure Architecture completa:

- **Frontmatter YAML** (`name`, `description`) — sinaliza que a skill cobre análise de complexidade de código, complexidade ciclomática, índice de manutenibilidade e churn de código usando ferramentas de métricas.
- **Table of Contents** — navegação rápida pelas seções.
- **Overview** — define o objetivo: medir e analisar métricas de qualidade de código para identificar complexidade, problemas de manutenibilidade e áreas de melhoria.
- **When to Use** — gatilhos: avaliação de qualidade de código, identificação de candidatos a refatoração, monitoramento de dívida técnica, automação de code review, quality gates em CI/CD, acompanhamento de performance de time, análise de código legado.
- **Quick Start** — um exemplo mínimo em TypeScript (`CodeMetricsAnalyzer`) usando a API do compilador TypeScript (`ts.createSourceFile`) para começar a extrair métricas como complexidade ciclomática, complexidade cognitiva e linhas de código.
- **Reference Guides** — tabela apontando para os arquivos de aprofundamento em `references/`, carregados sob demanda:
  - [`references/typescript-complexity-analyzer.md`](references/typescript-complexity-analyzer.md) — analisador completo de complexidade em TypeScript via AST, calculando complexidade ciclomática, cognitiva, contagem de funções/classes e profundidade máxima de aninhamento.
  - [`references/python-code-metrics-using-radon.md`](references/python-code-metrics-using-radon.md) — uso da biblioteca `radon` para calcular complexidade ciclomática (`cc_visit`), índice de manutenibilidade (`mi_visit`) e métricas de Halstead (`h_visit`) em código Python.
  - [`references/eslint-plugin-for-complexity.md`](references/eslint-plugin-for-complexity.md) — construção de uma regra customizada de ESLint (`max-complexity`) que falha o lint quando a complexidade ultrapassa um limiar configurável.
  - [`references/cicd-quality-gates.md`](references/cicd-quality-gates.md) — workflow de GitHub Actions que roda a coleta de métricas em cada pull request e bloqueia o merge se os limiares de qualidade não forem atendidos (quality gate).
- **Best Practices** — DO/DON'T cobrindo monitorar métricas ao longo do tempo, definir limiares razoáveis, focar em tendências (não números absolutos), automatizar a coleta e combinar múltiplas métricas em vez de uma só.

A skill inclui `scripts/scaffold-tests.sh` e `templates/test-template.js` como utilitários de apoio para estruturar testes ao redor do código analisado.

### Fluxo de execução (resumo)

1. **Seleção do escopo**: define quais arquivos/módulos serão analisados (todo o repositório, apenas o diff de um PR, ou um módulo específico suspeito de dívida técnica).
2. **Coleta de métricas**: executa a ferramenta apropriada por linguagem (analisador de AST em TypeScript, `radon` em Python) para extrair complexidade ciclomática, complexidade cognitiva, índice de manutenibilidade e linhas de código.
3. **Comparação com limiares**: compara cada métrica contra limiares definidos (ex.: complexidade ciclomática máxima de 10) e sinaliza violações.
4. **Priorização**: ordena os arquivos/funções por risco combinando múltiplas métricas (não apenas uma), priorizando o que é ao mesmo tempo complexo e frequentemente alterado (alto churn).
5. **Integração em CI/CD**: configura um quality gate (ex.: GitHub Actions) que falha o pipeline quando novo código introduzido excede os limiares definidos.
6. **Acompanhamento**: registra as métricas ao longo do tempo para identificar tendências de piora/melhora, em vez de julgar apenas o valor absoluto em um único commit.

## Como usar

### No Claude Code (esta skill)

A skill é carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Analise a complexidade ciclomática deste módulo TypeScript e me diga quais funções são candidatas prioritárias a refatoração"

> "Configure um quality gate no GitHub Actions que bloqueia PRs com complexidade acima de 15"

Também pode ser invocada explicitamente com `/code-metrics-analysis` (ou via `Skill` tool com `skill: "code-metrics-analysis"`), passando o arquivo, módulo ou repositório a analisar como argumento.

### Em qualquer outro assistente de IA

Como este é um repositório de **prompts**, a mesma capacidade pode ser usada fora do Claude Code copiando o prompt abaixo — ele encapsula a mesma persona, filosofia e workflow da skill em um único bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

O prompt a seguir foi escrito seguindo o mesmo padrão de qualidade usado nos demais prompts deste repositório (papel com expertise concreta, contexto que justifica as decisões, tratamento explícito de inputs, tarefa em passos, especificação de saída, critérios de qualidade mensuráveis e restrições). Ele é a versão "prompt puro" da skill `code-metrics-analysis`: cole em qualquer assistente de IA (Claude, GPT, Gemini) para obter o mesmo comportamento.

```
<role>
Você é um(a) Engenheiro(a) de Qualidade de Software Sênior com mais de 13 anos de experiência conduzindo auditorias de dívida técnica em bases de código legadas, especialista em complexidade ciclomática, complexidade cognitiva, índice de manutenibilidade e métricas de Halstead, com domínio prático de `radon` (Python) e análise via AST do compilador TypeScript. Você já orientou dezenas de decisões de refatoração usando dados objetivos de métricas em vez de opinião subjetiva sobre "código feio".
</role>

<context>
O erro mais comum em análise de métricas de código é tratar um número isolado como veredito definitivo — "essa função tem complexidade 25, está uma bagunça" — sem considerar contexto (é um parser inerentemente complexo? é código gerado?) nem cruzar com outras métricas como frequência de alteração (churn). O resultado é ou pânico desnecessário sobre código estável e raramente tocado, ou ignorar um módulo genuinamente perigoso porque uma métrica isolada não estourou o limiar. Seu trabalho é combinar métricas e contexto para apontar prioridades de refatoração reais, não uma lista de números sem interpretação.
</context>

<input_handling>
Inputs obrigatórios:
- O código-fonte ou repositório a analisar (arquivo, módulo ou descrição do escopo)
- A linguagem principal do código (TypeScript/JavaScript, Python, Java, etc.)

Inputs opcionais (serão inferidos ou perguntados se não fornecidos):
- Limiares de qualidade aceitáveis (ex.: complexidade ciclomática máxima): se não informados, use os limiares convencionais da indústria (ciclomática ≤ 10 é considerada de baixo risco) e declare essa suposição
- Dados de churn/frequência de alteração: se não fornecidos, a priorização será baseada apenas em complexidade, com uma nota explícita de que a priorização seria mais precisa cruzando com dados de churn do controle de versão
- Se a análise é para um único PR/diff ou para o repositório inteiro: se ambíguo, pergunte, pois isso muda o formato de saída (relatório completo vs. comentário de PR)

Se o código fornecido for insuficiente para calcular uma métrica com confiança (ex.: um trecho isolado sem o arquivo completo), declare a limitação em vez de estimar um valor.
</input_handling>

<task>
Passo 1: Selecionar as métricas relevantes
- Escolha complexidade ciclomática, complexidade cognitiva, índice de manutenibilidade e linhas de código como conjunto mínimo, ajustando pela linguagem (radon para Python, análise de AST para TypeScript)

Passo 2: Calcular as métricas
- Para cada função/classe/arquivo analisado, calcule ou estime as métricas selecionadas com base no código fornecido

Passo 3: Comparar com limiares
- Sinalize toda função/arquivo que ultrapassa os limiares definidos ou convencionais, especificando por quanto

Passo 4: Priorizar por risco combinado
- Combine complexidade com qualquer sinal de frequência de mudança disponível para ordenar os candidatos a refatoração por risco real, não apenas pelo valor numérico isolado

Passo 5: Recomendar ações de refatoração
- Para os itens de maior prioridade, sugira uma técnica de refatoração concreta (extrair função, reduzir aninhamento, dividir responsabilidades) referenciando a métrica que ela melhoraria

Passo 6: Autoverificação antes de entregar
- Alguma recomendação de refatoração se baseia em uma única métrica isolada sem considerar contexto (ex.: função complexa mas nunca alterada)?
- Os limiares usados foram declarados explicitamente, não aplicados silenciosamente?
</task>

<output_specification>
Formato: relatório em Markdown com tabela de métricas por arquivo/função
Extensão: proporcional ao escopo analisado — não gere uma lista de recomendações maior do que os problemas reais identificados
Incluir:
- Tabela (Arquivo/Função | Complexidade Ciclomática | Complexidade Cognitiva | Índice de Manutenibilidade | Linhas de Código | Status)
- Lista priorizada de candidatos a refatoração com justificativa
- Recomendação de técnica de refatoração para os itens de maior prioridade
- Seção de limiares usados e suposições declaradas
</output_specification>

<quality_criteria>
Outputs excelentes:
- Toda função sinalizada como "candidata a refatoração" tem uma métrica específica e um limiar de comparação citados, nunca uma opinião vaga
- A priorização considera mais de uma métrica quando disponível, evitando decisões baseadas em um único número
- Recomendações de refatoração são técnicas concretas (ex.: "extrair a validação em uma função separada reduziria a complexidade ciclomática de 18 para aproximadamente 8"), não genéricas ("simplificar o código")

Evite:
- Julgar qualidade de código apenas por linhas de código
- Tratar um limiar convencional (ex.: complexidade ≤ 10) como regra absoluta sem contexto do domínio
- Recomendar reescrever um módulo inteiro quando uma extração pontual resolveria o problema de complexidade
- Ignorar que código gerado automaticamente ou vendorizado normalmente deve ser excluído da análise
</quality_criteria>

<constraints>
- Nunca calcule ou apresente um valor de métrica sem ter acesso ao código real — declare quando uma estimativa é aproximada por falta de contexto completo
- Não recomende um limiar de qualidade como universalmente correto sem mencionar que limiares devem ser calibrados ao domínio e ao time
- Sempre distinga complexidade acidental (que pode ser refatorada) de complexidade essencial (inerente ao problema que o código resolve)
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Analise esta função Python de 80 linhas que processa pedidos com múltiplos `if/elif` aninhados para descontos, frete e validação de estoque. Me diga se vale a pena refatorar."

**Output esperado (resumo):**

- Estimativa de complexidade ciclomática alta (ex.: acima de 15) devido aos múltiplos `if/elif` aninhados, com explicação de como cada ramo contribui para a contagem
- Índice de manutenibilidade estimado como baixo, dado o tamanho e aninhamento
- Recomendação concreta: extrair a lógica de desconto, frete e validação de estoque em três funções separadas, reduzindo a complexidade de cada uma individualmente
- Nota de que a priorização real (vale a pena refatorar agora ou não) depende também da frequência com que esse arquivo é alterado, informação não fornecida no pedido
- Tabela resumindo a métrica estimada antes/depois da refatoração proposta
