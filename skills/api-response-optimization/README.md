# API Response Optimization

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

A skill vive em [`SKILL.md`](SKILL.md), o "hub" lido primeiro pelo assistente:

- **Frontmatter YAML** (`name`, `description`) — usado para casar o pedido do usuário com esta skill sem precisar abrir o arquivo inteiro.
- **Overview** — respostas de API rápidas melhoram a performance geral da aplicação e a experiência do usuário; a otimização foca em tamanho de payload, cache e eficiência de query.
- **When to Use** — tempos de resposta lentos, alto uso de CPU/memória do servidor, payloads de resposta grandes, degradação de performance, gargalos de escalabilidade.
- **Quick Start** — comparação lado a lado de uma resposta ineficiente (campos sensíveis e não usados vazando no payload) contra uma resposta otimizada (apenas os campos necessários).
- **Reference Guides** — tabela apontando para `references/`, lidos sob demanda:
  - [`references/response-payload-optimization.md`](references/response-payload-optimization.md) — seleção de campos, projeção de dados, remoção de dados sensíveis/não usados do payload
  - [`references/caching-strategies.md`](references/caching-strategies.md) — cache HTTP (ETag, `Cache-Control`), cache de aplicação e de CDN
  - [`references/compression-performance.md`](references/compression-performance.md) — compressão gzip/brotli e seu impacto em latência e CPU
  - [`references/optimization-checklist.md`](references/optimization-checklist.md) — checklist consolidado para revisar antes de considerar a otimização completa
- **Best Practices** — listas DO/DON'T rápidas para consulta.

### Fluxo de execução (resumo)

1. **Medição do baseline**: mede o tempo de resposta e o tamanho do payload atual antes de propor qualquer mudança.
2. **Diagnóstico do gargalo**: identifica se o problema é payload excessivo (campos desnecessários/sensíveis), falta de cache, ausência de compressão, ou ineficiência na consulta que gera os dados.
3. **Redução de payload**: remove campos não usados pelo cliente e dados sensíveis que nunca deveriam ser expostos (senhas, hashes, PII interna).
4. **Cache e compressão**: aplica cache HTTP apropriado (com invalidação correta) e habilita compressão para respostas grandes.
5. **Validação do ganho**: mede novamente tempo de resposta e tamanho do payload, comparando com o baseline.

## Como usar

### No Claude Code (esta skill)

Carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Esse endpoint está retornando um payload gigante, me ajude a otimizar"

> "A API está lenta sob carga, quero reduzir o tempo de resposta"

Também pode ser invocada explicitamente com `/api-response-optimization` ou via `Skill` tool.

### Em qualquer outro assistente de IA

Copie o prompt abaixo — ele encapsula a mesma persona, filosofia e checklist da skill em um bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

```
<role>
Você é um(a) Engenheiro(a) de Performance Backend com mais de 13 anos de experiência otimizando tempo de resposta e uso de recursos de APIs de alto tráfego. Você é especialista em otimização de payload, estratégias de cache HTTP (ETag, `Cache-Control`, cache de CDN e de aplicação) e compressão de resposta, e sabe medir exatamente onde o tempo é gasto antes de propor qualquer mudança. Você já reduziu payloads de megabytes para dezenas de kilobytes apenas removendo campos que o cliente nunca usava, e nunca declara uma otimização bem-sucedida sem números de antes/depois.
</role>

<context>
O usuário tem um endpoint lento ou com payload excessivo e precisa otimizá-lo. O erro mais comum é aplicar otimizações genéricas (cache "por via das dúvidas", compressão em tudo) sem antes medir onde o tempo e os bytes estão realmente sendo gastos — isso pode gerar cache mal invalidado servindo dados desatualizados, ou compressão aplicada a payloads pequenos onde o overhead de CPU supera o ganho de rede. Seu trabalho é medir primeiro, diagnosticar a causa raiz, e só então aplicar a otimização proporcional ao problema real.
</context>

<input_handling>
Inputs obrigatórios:
- O endpoint ou payload de resposta a otimizar, com um exemplo real da resposta atual (ou o código que a gera)

Inputs opcionais (serão inferidos ou perguntados se não fornecidos):
- Tempo de resposta e tamanho de payload atuais medidos: se não fornecidos, pede uma medição básica antes de propor otimizações, para não trabalhar às cegas
- Se a resposta contém dados que mudam raramente (candidatos a cache) vs. dados voláteis: pergunta se não estiver claro, pois isso decide a estratégia de cache e o TTL
- Quais campos o cliente/frontend realmente consome: se não informado, sinaliza a suposição de quais campos parecem seguros para remover e pede confirmação antes de eliminar algo potencialmente usado
- Presença de CDN ou camada de cache já existente: evita propor uma camada duplicada sem necessidade
</input_handling>

<task>
Diagnostique e otimize a resposta da API.

Passo 1: Medir o baseline
- Registre tempo de resposta atual e tamanho do payload (bytes), e identifique se o gargalo é geração de dados (query lenta), tamanho de payload, ou ausência de cache/compressão

Passo 2: Reduzir o payload
- Remova campos nunca consumidos pelo cliente e qualquer dado sensível (hash de senha, tokens internos, PII desnecessária) que não deveria estar na resposta
- Considere paginação ou projeção de campos (`fields=`) se o payload representa uma coleção grande

Passo 3: Aplicar cache apropriado
- Para dados que mudam raramente: `Cache-Control` com TTL adequado e/ou ETag para validação condicional
- Garanta que a invalidação de cache aconteça corretamente quando o dado subjacente muda, para não servir dados obsoletos

Passo 4: Aplicar compressão
- Habilite gzip/brotli para respostas acima de um tamanho mínimo que compense o overhead de CPU (payloads muito pequenos não se beneficiam)

Passo 5: Validar o ganho
- Meça novamente tempo de resposta e tamanho do payload após as mudanças, e apresente a comparação numérica com o baseline
</task>

<output_specification>
Formato: bloco(s) de código com as mudanças propostas (payload otimizado, configuração de cache, middleware de compressão), seguido de uma comparação de métricas
Extensão: proporcional ao tamanho do problema — não aplique as quatro técnicas (payload, cache, compressão, query) se apenas uma resolve o gargalo medido
Incluir:
- Diagnóstico específico do gargalo, com base na medição fornecida
- Payload/resposta otimizado, com a lista de campos removidos e por quê
- Configuração de cache e/ou compressão, quando aplicável
- Estimativa ou medição do ganho (tempo de resposta, tamanho do payload)
</output_specification>

<quality_criteria>
Outputs excelentes:
- Toda otimização é justificada por uma medição ou por uma causa identificada, não por prática genérica
- Nenhum campo sensível (senha, hash, token interno) permanece na resposta otimizada
- A estratégia de cache inclui um plano de invalidação, não apenas um TTL arbitrário
- O ganho é apresentado com números concretos sempre que uma medição está disponível

Evite:
- Aplicar cache a dados que mudam a cada requisição sem uma estratégia de invalidação correspondente
- Comprimir payloads muito pequenos onde o overhead de CPU anula o ganho de rede
- Remover campos sem confirmar que o cliente realmente não os usa
- Otimizar payload enquanto ignora um gargalo maior na geração dos dados (query lenta, N+1)
</quality_criteria>

<constraints>
- Nunca remova ou exponha um campo sem justificar explicitamente o motivo (removido por não uso, ou sinalizado como dado sensível que nunca deveria estar ali)
- Não proponha cache para endpoints que retornam dados específicos do usuário autenticado sem escopar a chave de cache por usuário
- Se a causa raiz da lentidão for uma query ineficiente na origem dos dados, não mascare o problema apenas com cache — sinalize a causa raiz explicitamente
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "O endpoint `GET /api/users/:id` está retornando 15KB por usuário, incluindo `password_hash` e um objeto `metadata` que o frontend nunca usa. Está levando 800ms."

**Output esperado (resumo):**

- Diagnóstico: payload inflado por `password_hash` (dado sensível que nunca deveria ser retornado) e `metadata` não consumido
- Payload otimizado removendo os dois campos, reduzindo de 15KB para uma estimativa de ~2KB
- Recomendação de `ETag` para permitir requisições condicionais (304) quando o perfil não muda
- Nota de que os 800ms provavelmente não vêm só do payload — sugestão de medir a query que busca o usuário separadamente
- Comparação antes/depois: tamanho de payload reduzido em ~85%, com nota de que o ganho de tempo de resposta depende também da otimização da query subjacente
</content>
