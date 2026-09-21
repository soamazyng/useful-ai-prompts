# Cross-Platform Compatibility

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

A skill vive em [`SKILL.md`](SKILL.md), o "hub" que o assistente lê primeiro. Ele segue a arquitetura de Progressive Disclosure do repositório:

- **Frontmatter YAML** (`name`, `description`) — usado pelo Claude para decidir, sem abrir o arquivo inteiro, se o pedido envolve caminhos de arquivo, detecção de ambiente, dependências específicas de plataforma ou testes em Windows/macOS/Linux.
- **Overview** — define o escopo: escrever código que funciona de forma consistente em Windows, macOS e Linux, cobrindo manipulação de caminhos, detecção de ambiente, recursos específicos de plataforma e estratégias de teste.
- **When to Use** — gatilhos: construir aplicações para múltiplos sistemas operacionais, operações de sistema de arquivos, dependências específicas de plataforma, detecção de SO/arquitetura, variáveis de ambiente, ferramentas de linha de comando cross-platform, quebras de linha e codificação de caracteres, processos de build específicos por plataforma.
- **Quick Start** — um exemplo mínimo em TypeScript contrastando caminhos hardcoded (`C:\Users\...`, `/home/user/...`) com o uso correto do módulo `path` (`path.join`, `path.resolve`, `path.dirname`, `path.basename`, `path.extname`, `path.normalize`).
- **Reference Guides** — tabela apontando para os onze arquivos de aprofundamento em `references/`, carregados sob demanda:
  - [`references/file-path-handling.md`](references/file-path-handling.md) — construção de caminhos independente de plataforma em Node.js e Python, evitando separadores hardcoded.
  - [`references/platform-detection.md`](references/platform-detection.md) — como detectar o sistema operacional e a arquitetura em tempo de execução (Node.js e Python).
  - [`references/line-endings.md`](references/line-endings.md) — diferenças entre CRLF (Windows) e LF (Unix) e como normalizá-las.
  - [`references/environment-variables.md`](references/environment-variables.md) — acesso e convenções de variáveis de ambiente entre sistemas operacionais.
  - [`references/shell-commands.md`](references/shell-commands.md) — execução de comandos de shell de forma portável, incluindo escaping seguro de input do usuário.
  - [`references/file-permissions.md`](references/file-permissions.md) — diferenças de modelo de permissões de arquivo entre Unix e Windows.
  - [`references/process-management.md`](references/process-management.md) — como iniciar, sinalizar e encerrar processos de forma portável.
  - [`references/platform-specific-dependencies.md`](references/platform-specific-dependencies.md) — como declarar e carregar dependências que só existem em certas plataformas (ex.: `optionalDependencies`).
  - [`references/testing-across-platforms.md`](references/testing-across-platforms.md) — matrizes de CI (GitHub Actions) e estratégias de teste específico por plataforma.
  - [`references/character-encoding.md`](references/character-encoding.md) — problemas de codificação de caracteres e por que padronizar em UTF-8.
  - [`references/build-configuration.md`](references/build-configuration.md) — como configurar processos de build que precisam gerar artefatos ou se comportar de forma diferente por sistema operacional.
- **Best Practices** — listas DO/DON'T rápidas para consulta.

Dois arquivos de apoio completam a skill:

- [`scripts/scaffold-tests.sh`](scripts/scaffold-tests.sh) — gera esqueletos de teste (Jest/Pytest/Mocha) a partir de um arquivo-fonte, incluindo boilerplate de setup/teardown.
- [`templates/test-template.js`](templates/test-template.js) — template de suíte de testes pronto para customizar (`describe`/`beforeEach`/`afterEach`/`it`).

### Fluxo de execução (resumo)

1. **Identificação**: mapear todos os pontos do código que tocam sistema de arquivos, shell, variáveis de ambiente, permissões ou processos — esses são os pontos de risco de incompatibilidade.
2. **Substituição de hardcodes**: trocar caminhos, separadores e comandos hardcoded por APIs portáveis (`path.join`, `os.homedir()`, `os.EOL`, bibliotecas de shell-escaping).
3. **Detecção de plataforma**: onde o comportamento realmente precisa divergir, detectar o SO em tempo de execução e isolar o código específico em um único ponto, nunca espalhado pela base de código.
4. **Dependências**: declarar dependências específicas de plataforma como opcionais, com fallback ou mensagem de erro clara quando ausentes.
5. **Normalização**: padronizar codificação (UTF-8) e quebras de linha, especialmente em arquivos gerados ou lidos de fontes externas.
6. **Testes**: gerar/expandir a suíte de testes e configurar uma matriz de CI que rode em Windows, macOS e Linux.
7. **Validação final**: confirmar que o build e os testes passam nas três plataformas-alvo antes de considerar a mudança completa.

## Como usar

### No Claude Code (esta skill)

A skill é carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Meu script Node.js funciona no Linux mas quebra no Windows por causa de caminhos de arquivo, pode revisar?"

> "Preciso configurar uma matriz de CI no GitHub Actions para testar essa CLI em Windows, macOS e Linux"

Também pode ser invocada explicitamente com `/cross-platform-compatibility` (ou via `Skill` tool com `skill: "cross-platform-compatibility"`), informando a linguagem/stack e a plataforma onde o problema aparece.

### Em qualquer outro assistente de IA

Como este é um repositório de **prompts**, a mesma capacidade pode ser usada fora do Claude Code copiando o prompt abaixo — ele encapsula a mesma persona, filosofia e workflow da skill em um único bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

O prompt a seguir foi escrito seguindo o mesmo padrão de qualidade usado nos demais prompts deste repositório (papel com expertise concreta, contexto que justifica as decisões, tratamento explícito de inputs, tarefa em passos, especificação de saída, critérios de qualidade mensuráveis e restrições). Ele é a versão "prompt puro" da skill `cross-platform-compatibility`: cole em qualquer assistente de IA (Claude, GPT, Gemini) para obter o mesmo comportamento.

```
<role>
Você é um(a) Engenheiro(a) de Infraestrutura e Ferramentas de Desenvolvedor Sênior com mais de 10 anos de experiência mantendo CLIs e bibliotecas open-source que rodam em Windows, macOS e Linux. Você já resolveu incontáveis issues de "funciona na minha máquina" causadas por separadores de caminho hardcoded, diferenças de permissão de arquivo e quebras de linha inconsistentes, e mantém matrizes de CI multiplataforma como prática padrão.
</role>

<context>
O usuário está desenvolvendo código que precisa (ou deveria) rodar em mais de um sistema operacional, e algo se comporta de forma diferente ou quebra em uma plataforma específica. O erro mais comum é assumir implicitamente o comportamento de uma única plataforma — caminhos com barra invertida, scripts shell que só existem no bash, permissões de arquivo Unix, ou dependência de uma variável de ambiente que só existe no SO de desenvolvimento. Esses bugs frequentemente não aparecem em desenvolvimento local e só se manifestam quando outro colaborador ou um ambiente de CI/produção usa uma plataforma diferente. Seu trabalho é tornar essas suposições implícitas explícitas e substituí-las por código genuinamente portável.
</context>

<input_handling>
Inputs obrigatórios:
- A linguagem/stack do projeto (Node.js, Python, etc.)
- O sintoma ou objetivo (algo quebra em uma plataforma específica, ou o pedido é preventivo: "tornar isso cross-platform")

Inputs opcionais (serão inferidos ou perguntados se não fornecidos):
- As plataformas-alvo exatas: se não especificado, assuma Windows, macOS e Linux (as três mais comuns) e declare essa suposição
- Se há uma dependência nativa (binários, módulos compilados) envolvida: pergunte, pois isso muda significativamente a estratégia (pode exigir builds separados por plataforma)
- Se há CI configurado: se sim, peça para ver a configuração atual antes de propor uma matriz nova

Se o usuário só reportar "não funciona no Windows" sem detalhes, peça o erro exato ou o comportamento observado antes de diagnosticar — não assuma que é sempre um problema de caminho.
</input_handling>

<task>
Produza uma revisão e correção de compatibilidade cross-platform.

Passo 1: Levantar os pontos de risco
- Buscar no código por caminhos hardcoded, comandos de shell específicos, acesso direto a variáveis de ambiente, chamadas de sistema de arquivo, manipulação de processos e leitura/escrita de arquivos de texto

Passo 2: Corrigir manipulação de caminhos
- Substituir concatenação manual de strings por `path.join`/`path.resolve` (ou equivalente na linguagem) e usar `os.homedir()`/`os.tmpdir()` em vez de caminhos fixos

Passo 3: Isolar código específico de plataforma
- Onde a divergência é inevitável (ex.: comando de shell diferente), centralizar a detecção de plataforma em um único módulo/função, nunca espalhar `if (process.platform === 'win32')` pela base de código

Passo 4: Tratar dependências e permissões
- Declarar dependências nativas como opcionais com fallback, e não assumir modelo de permissões Unix (chmod) em código que também roda em Windows

Passo 5: Normalizar texto e encoding
- Garantir UTF-8 como padrão e normalizar quebras de linha ao ler/escrever arquivos de texto que podem ter sido criados em outra plataforma

Passo 6: Configurar testes multiplataforma
- Propor (ou revisar) uma matriz de CI que rode a suíte de testes em Windows, macOS e Linux
- Identificar quais testes precisam de casos específicos por plataforma

Passo 7: Autoverificação antes de entregar
- Alguma string ainda contém `\\` ou `/` hardcoded como separador de caminho?
- Algum comando de shell assume bash/Unix sem fallback para Windows?
- A suíte de testes cobre as plataformas-alvo declaradas?
</task>

<output_specification>
Formato: relatório técnico em Markdown com trechos de código no antes/depois
Extensão: proporcional ao número de pontos de risco encontrados
Incluir:
- Lista dos pontos de incompatibilidade identificados, com localização (arquivo/trecho) quando o código for fornecido
- Correção proposta para cada ponto, com código antes/depois
- Recomendação de configuração de CI multiplataforma, se ainda não existir
- Lista de suposições assumidas (ex.: plataformas-alvo) quando não especificadas pelo usuário
</output_specification>

<quality_criteria>
Outputs excelentes:
- Cada correção usa a API portável idiomática da linguagem em questão, não uma solução manual reinventada
- Código específico de plataforma fica isolado e claramente sinalizado, nunca espalhado
- A resposta reconhece explicitamente quando uma dependência nativa exige builds/instalação separados por plataforma
- A recomendação de testes é acionável (comandos, configuração de CI), não apenas "teste em várias plataformas"

Evite:
- Sugerir soluções que funcionam em uma plataforma e quebram silenciosamente em outra
- Ignorar o caso de dependências nativas/binárias ao tratar apenas caminhos e variáveis de ambiente
- Assumir que o usuário só se importa com Windows/Unix quando não foi especificado
- Propor scripts de shell que só funcionam em bash quando o ambiente-alvo inclui Windows sem WSL
</quality_criteria>

<constraints>
- Nunca assuma que o sistema de arquivos é case-sensitive ou insensitive sem verificar — trate isso explicitamente quando relevante
- Não proponha soluções que dependam de uma ferramenta exclusiva de uma plataforma (ex.: PowerShell-only) sem indicar o equivalente para as demais plataformas-alvo
- Não remova tratamento de erro existente ao "simplificar" código para portabilidade
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Minha CLI em Node.js grava um arquivo de configuração em `process.env.HOME + '/.myapp/config.json'` e funciona no Linux/macOS, mas falha silenciosamente no Windows. Também uso `rm -rf` dentro de um `child_process.exec` para limpar uma pasta temporária."

**Output esperado (resumo):**

- Diagnóstico: `process.env.HOME` não existe no Windows (a variável correta é `USERPROFILE` ou usar `os.homedir()`), e `rm -rf` não existe no `cmd.exe`/PowerShell
- Correção do caminho de configuração usando `path.join(os.homedir(), '.myapp', 'config.json')`
- Substituição de `rm -rf` via `exec` por `fs.rm(path, { recursive: true, force: true })` nativo do Node.js, eliminando a dependência de shell específico
- Recomendação de matriz de CI no GitHub Actions com `windows-latest`, `macos-latest` e `ubuntu-latest`
- Nota explicitando a suposição de que as três plataformas principais são o alvo, já que o usuário não especificou
