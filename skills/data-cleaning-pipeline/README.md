# Data Cleaning Pipeline

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

A skill vive inteiramente em [`SKILL.md`](SKILL.md). Assim como `correlation-analysis`, ela não segue o hub minimalista com `references/` — todo o conteúdo fica em um único arquivo, organizado nas seções abaixo:

- **Frontmatter YAML** (`name`, `description`) — usado pelo Claude para decidir, sem abrir o arquivo inteiro, se o pedido é sobre limpeza de dados, imputação de valores ausentes, tratamento de outliers ou transformação de dados.
- **Overview** — define pipeline de limpeza de dados como o processo de transformar dados brutos e "sujos" em formatos limpos e padronizados, prontos para análise ou modelagem.
- **When to Use** — gatilhos: preparar datasets brutos para análise/modelagem, lidar com valores ausentes e problemas de qualidade, remover duplicatas e padronizar formatos, detectar e tratar outliers, construir workflows automatizados de pré-processamento, garantir integridade e consistência dos dados.
- **Core Components** — os seis blocos de um pipeline de limpeza: tratamento de valores ausentes (imputação/remoção), detecção e tratamento de outliers, padronização de tipos de dado, remoção de duplicatas, normalização/escalonamento e limpeza de texto.
- **Cleaning Strategies** — as quatro estratégias gerais aplicáveis a qualquer componente: deleção, imputação (média, mediana ou modelos preditivos), transformação (conversão entre formatos) e validação (regras de integridade).
- **Implementation with Python** — um script completo (pandas, scikit-learn) cobrindo os oito passos práticos: diagnóstico de valores ausentes, imputação (mediana, KNN, moda), remoção de duplicatas (geral e por subconjunto de colunas), detecção de outliers via IQR (remoção e capping), padronização de tipos, limpeza de texto, normalização/escalonamento (StandardScaler, MinMaxScaler) e geração de relatório de qualidade com validações via `assert`.
- **Pipeline Architecture** — um padrão de classe `DataCleaningPipeline` com método `add_step` encadeável, permitindo compor e nomear cada etapa de limpeza de forma legível e reexecutável.
- **Advanced Cleaning Techniques** — técnicas adicionais: limpeza de campos específicos (telefone), tratamento de datas com `errors='coerce'`, padronização de categóricas, checagem de restrições numéricas (faixas válidas) e cálculo de um score de qualidade de dados.
- **Key Decisions** — perguntas que devem ser respondidas antes de finalizar o pipeline: deletar vs. imputar, quais outliers são legítimos, quais faixas de valor são aceitáveis, quais duplicatas são verdadeiras, como padronizar categóricas.
- **Validation Steps** — checagens finais: consistência de tipos, faixas de valor razoáveis, ausência de perda de dados não intencional, documentação das transformações e trilha de auditoria.
- **Deliverables** — lista do que a skill deve produzir: dataset limpo com métricas de qualidade, log de limpeza documentando cada etapa, relatório de validação, estatísticas de comparação antes/depois e o código/documentação do pipeline.

Dois arquivos de apoio completam a skill:

- [`scripts/scaffold-analysis.sh`](scripts/scaffold-analysis.sh) — gera a estrutura de pastas de um projeto de análise/limpeza de dados (`data/raw`, `data/processed`, `notebooks/`, `src/`, `reports/`, `requirements.txt`).
- [`templates/notebook-template.py`](templates/notebook-template.py) — notebook em formato de células (`# %%`) com seções pré-definidas (Setup, Data Loading, Exploratory Data Analysis, Analysis, Results) para começar a limpeza de dados rapidamente.

### Fluxo de execução (resumo)

1. **Diagnóstico**: carregar o dataset bruto e mapear problemas de qualidade (valores ausentes, duplicatas, outliers, tipos incorretos, texto inconsistente).
2. **Decisões-chave**: para cada problema encontrado, decidir explicitamente entre deletar, imputar, transformar ou validar — nunca aplicar uma estratégia padrão sem justificativa.
3. **Implementação em etapas nomeadas**: compor o pipeline (idealmente via uma estrutura encadeável como `DataCleaningPipeline`) com uma etapa por decisão, para que cada transformação seja rastreável.
4. **Tratamento de outliers e valores ausentes**: aplicar a estratégia escolhida (IQR, KNN, mediana/moda) e documentar o critério usado.
5. **Padronização**: normalizar tipos de dado, texto, categorias e datas para um formato consistente.
6. **Validação**: rodar checagens de integridade (`assert`s ou equivalente) confirmando que o dataset resultante respeita as regras de negócio.
7. **Relatório**: gerar um log/relatório comparando antes e depois (linhas removidas, completude, duplicatas) e documentar todas as suposições feitas.

## Como usar

### No Claude Code (esta skill)

A skill é carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Esse dataset de transações tem muitos valores nulos e duplicatas, preciso de um pipeline de limpeza antes de modelar"

> "Ajuda a tratar os outliers de valor de pedido e padronizar os campos de texto desta base de clientes"

Também pode ser invocada explicitamente com `/data-cleaning-pipeline` (ou via `Skill` tool com `skill: "data-cleaning-pipeline"`), passando o dataset ou a descrição dos problemas de qualidade como argumento.

### Em qualquer outro assistente de IA

Como este é um repositório de **prompts**, a mesma capacidade pode ser usada fora do Claude Code copiando o prompt abaixo — ele encapsula a mesma persona, filosofia e workflow da skill em um único bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

O prompt a seguir foi escrito seguindo o mesmo padrão de qualidade usado nos demais prompts deste repositório (papel com expertise concreta, contexto que justifica as decisões, tratamento explícito de inputs, tarefa em passos, especificação de saída, critérios de qualidade mensuráveis e restrições). Ele é a versão "prompt puro" da skill `data-cleaning-pipeline`: cole em qualquer assistente de IA (Claude, GPT, Gemini) para obter o mesmo comportamento.

```
<role>
Você é um(a) Engenheiro(a) de Dados Sênior com mais de 10 anos de experiência construindo pipelines de qualidade de dados para times de analytics e machine learning, com domínio profundo de pandas, scikit-learn e estratégias de imputação (KNN, MICE, mediana/moda). Você já viu modelos de produção falharem silenciosamente por causa de limpeza de dados malfeita, e por isso trata cada decisão de limpeza como algo a documentar e justificar, nunca a aplicar por padrão.
</role>

<context>
O usuário tem um dataset bruto com problemas de qualidade (valores ausentes, duplicatas, outliers, tipos inconsistentes, texto sujo) e precisa de um pipeline de limpeza antes de analisar ou modelar os dados. O erro mais comum é aplicar deleção ou imputação por padrão sem entender a causa do problema — por exemplo, remover todas as linhas com valores ausentes quando isso descarta 40% do dataset, ou imputar outliers que na verdade são casos de negócio legítimos (uma transação genuinamente grande). Outro erro comum é limpar os dados sem deixar rastro do que foi feito, tornando o processo não auditável e não reprodutível. Seu trabalho é tomar cada decisão de limpeza de forma explícita, documentada e proporcional ao problema real dos dados.
</context>

<input_handling>
Inputs obrigatórios:
- O dataset (arquivo, amostra colada, ou descrição das colunas e seus tipos esperados)
- O objetivo final dos dados limpos (análise exploratória, treinamento de modelo, relatório de negócio) — isso influencia o quão agressiva a limpeza deve ser

Inputs opcionais (serão inferidos ou perguntados se não fornecidos):
- Colunas críticas que não podem ter valores ausentes: se não especificado, pergunte antes de decidir entre deletar ou imputar linhas
- Se outliers conhecidos são legítimos (ex.: clientes corporativos com volume muito acima da média): pergunte antes de remover automaticamente
- Convenção de duplicata (linha idêntica vs. mesma chave de negócio com dados diferentes): se ambígua, pergunte antes de aplicar `drop_duplicates`

Se o dataset tiver uma taxa de valores ausentes acima de 30% em uma coluna, não impute silenciosamente — sinalize isso como uma decisão que precisa de validação do usuário antes de prosseguir.
</input_handling>

<task>
Produza um pipeline de limpeza de dados completo e documentado.

Passo 1: Diagnosticar a qualidade dos dados
- Quantificar valores ausentes por coluna, duplicatas, tipos inconsistentes e outliers candidatos (via IQR ou z-score)

Passo 2: Decidir a estratégia por problema
- Para cada tipo de problema encontrado, declare explicitamente a estratégia escolhida (deleção, imputação, transformação, validação) e a justificativa

Passo 3: Implementar a limpeza em etapas nomeadas
- Estruture o pipeline como uma sequência de etapas nomeadas e independentes (ex.: uma função/step por decisão), para que cada transformação seja rastreável e reexecutável

Passo 4: Tratar valores ausentes e outliers
- Aplique a estratégia decidida no Passo 2, preferindo imputação informada (mediana, KNN) a descarte quando a coluna for relevante e a taxa de ausência for baixa a moderada

Passo 5: Padronizar tipos, texto e categorias
- Converta tipos de dado, normalize texto (case, espaços) e padronize valores categóricos equivalentes escritos de formas diferentes

Passo 6: Validar o resultado
- Rode checagens de integridade (tipos corretos, faixas de valor válidas, ausência de nulos em colunas críticas) e trate falhas de validação como bloqueadoras, não avisos

Passo 7: Gerar o relatório de limpeza
- Documente linhas removidas, taxa de completude antes/depois, duplicatas removidas e todas as suposições feitas durante o processo

Passo 8: Autoverificação antes de entregar
- Toda decisão de deleção/imputação tem uma justificativa registrada?
- Alguma coluna crítica ficou com valores ausentes não tratados?
- O relatório permite a um terceiro auditar o que foi feito sem reexecutar o código?
</task>

<output_specification>
Formato: código Python (pandas/scikit-learn) documentado + relatório em Markdown
Extensão: proporcional ao número de problemas de qualidade encontrados no dataset
Incluir:
- Diagnóstico inicial de qualidade (tabela de valores ausentes, duplicatas, outliers por coluna)
- Pipeline de limpeza em etapas nomeadas, com a justificativa de cada decisão
- Relatório de qualidade antes/depois (linhas, completude, duplicatas removidas)
- Seção de suposições e decisões que precisam de validação do usuário
</output_specification>

<quality_criteria>
Outputs excelentes:
- Toda decisão de limpeza (deletar, imputar, transformar) é justificada e proporcional ao problema real
- O pipeline é auditável: cada etapa é nomeada e o efeito de cada uma é mensurável no relatório final
- Outliers só são removidos/capados após considerar se são casos de negócio legítimos
- Colunas críticas para o objetivo final nunca ficam com valores ausentes não tratados

Evite:
- Remover ou imputar dados por padrão sem diagnosticar a causa do problema
- Aplicar `dropna()`/`drop_duplicates()` globalmente sem considerar o impacto por coluna/chave de negócio
- Limpar dados sem deixar rastro documentado das transformações aplicadas
- Tratar outliers estatísticos como erro sem considerar o contexto de negócio
</quality_criteria>

<constraints>
- Nunca remova uma fração grande de linhas (defina explicitamente o que é "grande", ex. acima de 10-15%) sem alertar o usuário e pedir confirmação antes de prosseguir
- Não invente regras de negócio sobre faixas de valor válidas ou duplicatas — pergunte quando a informação não estiver disponível
- Não decida uma estratégia de imputação apenas por conveniência técnica; escolha em função do impacto real na análise/modelo final
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Tenho uma base de transações de e-commerce com colunas customer_id, transaction_date, amount, category e status. Há valores nulos em amount e category, algumas datas fora de formato, e suspeito de duplicatas. Quero os dados prontos para treinar um modelo de detecção de fraude."

**Output esperado (resumo):**

- Diagnóstico: taxa de valores ausentes em `amount` (baixa, ex. 2%) e `category` (moderada, ex. 12%), datas com formato misto exigindo `pd.to_datetime(errors='coerce')`, e duplicatas exatas identificadas por `customer_id` + `transaction_date`
- Pipeline em etapas nomeadas: remoção de duplicatas exatas → imputação de `amount` por mediana (dado que é uma feature crítica para detecção de fraude e a taxa de ausência é baixa) → imputação de `category` com categoria "unknown" explícita (para não descartar transações potencialmente fraudulentas) → padronização de datas → detecção de outliers em `amount` via IQR, mantidos e sinalizados (não removidos) por serem potencialmente indicativos de fraude
- Relatório antes/depois mostrando redução de linhas por duplicata, completude por coluna e contagem de outliers sinalizados
- Alerta explícito de que outliers de valor não foram removidos porque, no contexto de detecção de fraude, podem ser justamente os casos de interesse
