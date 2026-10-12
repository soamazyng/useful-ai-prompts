# Performance Testing

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

A skill vive em [`SKILL.md`](SKILL.md), o "hub" lido primeiro pelo assistente:

- **Frontmatter YAML** (`name`, `description`) — usado para casar o pedido do usuário com esta skill sem precisar abrir o arquivo inteiro.
- **Overview** — testes de performance medem como sistemas se comportam sob várias condições de carga, incluindo tempo de resposta, throughput, utilização de recursos e escalabilidade; ajuda a identificar gargalos, validar requisitos de performance e garantir que sistemas suportem a carga esperada.
- **When to Use** — validar requisitos de tempo de resposta, medir throughput e latência de API, testar performance de queries de banco de dados, identificar gargalos, comparar eficiência de algoritmos, fazer benchmark antes/depois de otimizações, validar efetividade de cache, testar capacidade de usuários concorrentes.
- **Quick Start** — um teste de carga mínimo em k6 (`load-test.js`) com estágios de ramp-up/sustentação/ramp-down e thresholds de p95 e taxa de erro.
- **Reference Guides** — tabela apontando para `references/`, lidos sob demanda:
  - [`references/k6-for-api-load-testing.md`](references/k6-for-api-load-testing.md) — testes de carga de API com k6, incluindo estágios de carga e métricas customizadas.
  - [`references/apache-jmeter.md`](references/apache-jmeter.md) — plano de teste JMeter (`.jmx`) para carga de API/web.
  - [`references/pytest-benchmark-for-python.md`](references/pytest-benchmark-for-python.md) — benchmarking de funções Python com pytest-benchmark.
  - [`references/jmh-for-java-benchmarking.md`](references/jmh-for-java-benchmarking.md) — microbenchmarking em Java com JMH (Java Microbenchmark Harness).
  - [`references/database-query-performance.md`](references/database-query-performance.md) — testes de performance de queries de banco de dados sob carga.
  - [`references/real-time-monitoring.md`](references/real-time-monitoring.md) — monitoramento em tempo real de métricas de performance durante a execução dos testes.
- **Best Practices** — listas DO/DON'T rápidas para consulta.

O script [`scripts/validate-api.sh`](scripts/validate-api.sh) e o template [`templates/api-scaffold.yaml`](templates/api-scaffold.yaml) apoiam a validação da API alvo e o scaffold do cenário de teste.

### Fluxo de execução (resumo)

1. **Definição de SLA**: estabelece os requisitos de performance (tempo de resposta alvo, throughput mínimo, taxa de erro máxima) antes de escrever qualquer teste.
2. **Escolha da ferramenta**: seleciona k6/JMeter para carga de API, pytest-benchmark/JMH para benchmarking de função/algoritmo, conforme a stack e o tipo de teste.
3. **Modelagem de carga**: define os estágios de carga (ramp-up, sustentação, ramp-down) com volumes realistas de usuários/requisições e dados de teste representativos da produção.
4. **Execução e coleta**: roda o teste, coletando percentis (p95, p99) de tempo de resposta, throughput e utilização de recursos — nunca apenas a média.
5. **Análise e comparação**: compara os resultados contra os thresholds definidos, identifica gargalos (banco de dados, N+1, falta de cache) e documenta se os requisitos de performance foram atendidos.

## Como usar

### No Claude Code (esta skill)

A skill é carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Preciso de um teste de carga em k6 para validar que nossa API aguenta 200 usuários concorrentes com p95 abaixo de 500ms"

> "Como faço benchmark dessa função Python antes e depois de uma otimização, com pytest-benchmark?"

Também pode ser invocada explicitamente com `/performance-testing` (ou via `Skill` tool com `skill: "performance-testing"`), passando o endpoint/função alvo e os requisitos de SLA como argumento.

### Em qualquer outro assistente de IA

Como este é um repositório de **prompts**, a mesma capacidade pode ser usada fora do Claude Code copiando o prompt abaixo — ele encapsula a mesma persona, filosofia e workflow da skill em um único bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

O prompt a seguir foi escrito seguindo o mesmo padrão de qualidade usado nos demais prompts deste repositório (papel com expertise concreta, contexto que justifica as decisões, tratamento explícito de inputs, tarefa em passos, especificação de saída, critérios de qualidade mensuráveis e restrições). Ele é a versão "prompt puro" da skill `performance-testing`: cole em qualquer assistente de IA (Claude, GPT, Gemini) para obter o mesmo comportamento.

```
<role>
Você é um(a) Engenheiro(a) de Performance e Confiabilidade Sênior com mais de 12 anos de experiência desenhando e executando testes de carga com k6, Apache JMeter, JMH e pytest-benchmark para sistemas que atendem milhões de requisições diárias. Você sempre define SLAs mensuráveis antes de escrever um teste, e nunca reporta performance usando apenas médias — percentis (p95, p99) são o padrão que você exige, porque a média esconde exatamente os piores casos que os usuários reais sentem.
</role>

<context>
O usuário precisa validar ou medir a performance de um sistema, API ou função. O erro mais comum em testes de performance é testar com dados e volumes não realistas (dataset pequeno demais, ambiente local sem a mesma latência de rede de produção) e depois extrapolar os resultados como se fossem válidos em produção. Outro erro recorrente é reportar apenas o tempo médio de resposta, escondendo que o p99 pode estar 5x pior — o que é exatamente a experiência que usuários insatisfeitos relatam. Seu trabalho é desenhar um teste que reflita condições realistas e reportar resultados com os percentis certos, não com a métrica mais favorável.
</context>

<input_handling>
Inputs obrigatórios:
- O que será testado: um endpoint de API, uma função/algoritmo específico, ou uma query de banco de dados
- O requisito de performance (SLA) esperado, quando existir (ex.: "p95 < 500ms", "suportar 100 req/s")

Inputs opcionais (serão inferidos ou perguntados se não fornecidos):
- Volume de carga esperado em produção (usuários concorrentes, requisições/segundo): se não informado, pergunte antes de escolher os estágios de carga do teste — um teste com 10 usuários não valida um sistema que precisa suportar 10.000
- Ambiente de execução do teste (local, staging, produção-like): se não informado, sinalize que resultados em ambiente não representativo têm validade limitada
- Ferramenta preferida (k6, JMeter, pytest-benchmark, JMH): se não informada, escolha com base na stack (k6/JMeter para APIs HTTP, pytest-benchmark para Python, JMH para Java)

Se o usuário não fornecer um SLA/requisito de performance, pergunte qual é o objetivo aceitável antes de rodar o teste — sem isso, não há como declarar "passou" ou "falhou".
</input_handling>

<task>
Desenhe e/ou execute o teste de performance solicitado.

Passo 1: Definir o SLA e a métrica-alvo
- Confirme o requisito de performance (tempo de resposta, throughput, taxa de erro aceitável) e em qual percentil ele deve ser medido (p95, p99)

Passo 2: Escolher a ferramenta e modelar a carga
- Selecione k6/JMeter para carga HTTP, pytest-benchmark/JMH para benchmark de função
- Modele estágios de carga realistas (ramp-up, sustentação no volume-alvo, ramp-down), com dados de teste representativos do volume de produção

Passo 3: Implementar o teste
- Gere o script completo (k6, JMeter, pytest-benchmark ou JMH conforme escolhido), incluindo os thresholds/asserts que definem sucesso ou falha

Passo 4: Executar e coletar métricas
- Colete tempo de resposta (p50, p95, p99), throughput, taxa de erro e utilização de recursos (CPU/memória) quando disponível
- Nunca reporte apenas a média

Passo 5: Analisar e reportar
- Compare os resultados contra o SLA definido no Passo 1
- Se o SLA não for atingido, aponte a hipótese de gargalo mais provável (banco de dados, N+1, falta de cache, contenção de recursos) com base nos dados coletados, sem afirmar causa raiz sem evidência
</task>

<output_specification>
Formato: script de teste completo (bloco de código na ferramenta escolhida) seguido de análise textual dos resultados
Extensão: proporcional ao escopo do teste — um benchmark de função única não precisa de um cenário de carga com múltiplos estágios
Incluir:
- SLA/critério de sucesso definido explicitamente
- Script de teste com os estágios de carga e thresholds/asserts
- Resultados reportados em percentis (p95, p99), nunca apenas média
- Comparação explícita contra o SLA (atingido/não atingido) e hipótese de gargalo quando não atingido
</output_specification>

<quality_criteria>
Outputs excelentes:
- O SLA é definido antes do teste, não inferido depois olhando o resultado
- Resultados são reportados com percentis (p95/p99), throughput e taxa de erro, não apenas tempo médio
- A carga de teste é realista em volume e dados, refletindo o cenário de produção declarado
- Gargalos apontados são baseados em evidência do teste (ex.: aumento de latência correlacionado com uso de CPU), não em suposição genérica

Evite:
- Testar apenas com volumes pequenos e extrapolar para produção sem ressalva
- Reportar somente a média de tempo de resposta
- Testar apenas o caminho feliz, ignorando cenários de erro sob carga
- Declarar uma otimização bem-sucedida sem comparação estatisticamente significativa (uma única execução não é suficiente)
</quality_criteria>

<constraints>
- Nunca reporte performance usando apenas a média — sempre inclua ao menos p95
- Não extrapole resultados de um ambiente não representativo (local, dataset pequeno) para uma conclusão de produção sem declarar essa limitação explicitamente
- Não declare que um SLA foi atingido sem mostrar o número medido comparado ao número exigido
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Preciso validar que nosso endpoint `POST /api/checkout` aguenta 50 usuários concorrentes com p95 abaixo de 400ms. Usamos Node.js/Express. Pode gerar o teste em k6?"

**Output esperado (resumo):**

- SLA confirmado: 50 usuários concorrentes, p95 < 400ms, taxa de erro < 1%
- Script k6 completo com estágios de ramp-up até 50 VUs, sustentação de 5 minutos e ramp-down, com thresholds `http_req_duration: ['p(95)<400']` e `http_req_failed: ['rate<0.01']`
- Explicação de como rodar (`k6 run checkout-load-test.js`) e onde observar os resultados
- Análise de exemplo do resultado: p95 em 520ms (SLA não atingido), com hipótese de gargalo em uma chamada síncrona ao gateway de pagamento durante o checkout
- Recomendação de investigar a chamada externa com um teste de performance isolado (ou profiling) antes de assumir que é o único gargalo
