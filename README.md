# Useful AI Prompts - Biblioteca de Prompts de IA Prontos para Uso

> *Este repositório é um fork traduzido para português do projeto original [useful-ai-prompts](https://github.com/aj-geddes/useful-ai-prompts), criado por [aj-geddes](https://github.com/aj-geddes).*

[![Run in Smithery](https://smithery.ai/badge/skills/aj-geddes)](https://smithery.ai/skills?ns=aj-geddes&utm_source=github&utm_medium=badge)
[![GitHub Stars](https://img.shields.io/github/stars/aj-geddes/useful-ai-prompts?style=social)](https://github.com/aj-geddes/useful-ai-prompts)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Prompts](https://img.shields.io/badge/Prompts-679+-green)](prompts/)
[![Skills](https://img.shields.io/badge/Skills-260+-orange)](skills/)
[![Hooks](https://img.shields.io/badge/Hooks-7-purple)](hooks/)

> **679 prompts de IA prontos para uso, todos seguindo um template padronizado com verificações de qualidade validadas.**

Transforme o ChatGPT, Claude e outros assistentes de IA em consultores especialistas. Cada prompt desta biblioteca passa por 11 verificações de qualidade, garantindo estrutura consistente, entregáveis claros e critérios de qualidade mensuráveis.

---

## O que tem aqui

| Tipo de Recurso                   | Qtd   | Descrição                                                |
| --------------------------------- | ----- | -------------------------------------------------------- |
| **[Prompts de IA](prompts/)**     | 679+  | Prompts especializados padronizados em 47 categorias     |
| **[Skills para Claude Code](skills/)** | 260+ | Habilidades com ativação automática e exemplos de código |
| **[Hooks de Automação](hooks/)** | 7     | Segurança, testes, formatação e automação de CI/CD       |

---

## Formato Padronizado de Prompts

Cada prompt segue a mesma estrutura validada:

```markdown
# [Nome do Prompt]

## Metadata

- **ID**: `categoria-slug-do-prompt`
- **Version**: 1.0.0
- **Category**: Categoria principal
- **Tags**: palavras, chave, aqui
- **Complexity**: simple | intermediate | advanced
- **Interaction**: single-shot | conversational | iterative
- **Models**: Claude 3+, GPT-4+

## Overview

2-3 frases explicando o que o prompt faz e para quem é.

## When to Use

- Cenário específico 1
- Cenário específico 2

**Não use para**: Anti-padrões onde este prompt não é apropriado

---

## Prompt

<role>Identidade do especialista com credenciais e experiência específicas</role>

<context>Situação, critérios de sucesso e premissas principais</context>

<input_handling>
Obrigatório: Inputs indispensáveis
Opcional: Valores padrão inferidos quando não fornecidos
</input_handling>

<task>
Passos claros (3-7) do que deve ser realizado
</task>

<output_specification>
Requisitos de formato, extensão e estrutura
</output_specification>

<quality_criteria>
Padrões mensuráveis para uma saída de excelência
</quality_criteria>

<constraints>
Limites rígidos e restrições
</constraints>

---

## Example Usage

### Input

Solicitação realista do usuário (20-200 palavras)

### Output

Resposta representativa demonstrando qualidade (100-600 palavras)

## Related Prompts

Links para prompts complementares
```

---

## XML Tag Structure

A seção de prompt usa tags XML semânticas para análise consistente:

| Tag                      | Purpose                    | Content                                                     |
| ------------------------ | -------------------------- | ----------------------------------------------------------- |
| `<role>`                 | Expert identity            | Credenciais, experiência, abordagem de raciocínio           |
| `<context>`              | Situation framing          | Quando usar, critérios de sucesso, premissas principais     |
| `<input_handling>`       | Input specification        | Inputs obrigatórios vs opcionais com valores padrão         |
| `<task>`                 | Process steps              | 3-7 passos de ação numerados                                |
| `<output_specification>` | Deliverable format         | Formato, extensão, estrutura, elementos obrigatórios        |
| `<quality_criteria>`     | Success metrics            | Padrões objetivos e anti-padrões                            |
| `<constraints>`          | Hard limits                | Restrições não negociáveis de escopo/formato                |

---

## Quality Checks

Cada prompt passa por estas 11 verificações de validação:

| Check                 | Requirement                                                                                          |
| --------------------- | ---------------------------------------------------------------------------------------------------- |
| `metadata_complete`   | Todos os campos obrigatórios presentes (ID, Version, Category, Tags, Complexity, Interaction, Models) |
| `overview_concise`    | No máximo 3 frases, sem linguagem promocional                                                        |
| `role_specific`       | Expertise concreta definida, não apenas "vou te ajudar"                                              |
| `inputs_categorized`  | Obrigatórios vs opcionais distinguidos com valores padrão                                            |
| `task_structured`     | 3-7 passos claros e numerados                                                                        |
| `outputs_specified`   | Formato + extensão + requisitos para cada entregável                                                 |
| `criteria_measurable` | Padrões de qualidade objetivos, não aspirações vagas                                                 |
| `example_realistic`   | Entrada mostra uso real (20-200 palavras)                                                            |
| `example_concise`     | Saída demonstra o padrão (100-600 palavras)                                                          |
| `no_duplication`      | Sem informações repetidas entre seções                                                               |
| `copy_paste_ready`    | Seção do prompt é independente e claramente delimitada                                               |

---

## Quick Start

### Using the Prompts

1. **Navegue** pelo [diretório de prompts](prompts/) ou pela [interface web](https://aj-geddes.github.io/useful-ai-prompts/)
2. **Copie** o conteúdo da seção `## Prompt` (incluindo as tags XML)
3. **Cole** no ChatGPT, Claude ou no seu assistente de IA preferido
4. **Forneça** sua entrada específica — o prompt cuida do resto

### Example: Competitive Analysis

```text
<role>
Você é um estrategista de inteligência competitiva com mais de 12 anos de
experiência em análise de mercado, tendo liderado estratégia competitiva
tanto em startups quanto em empresas Fortune 500.
</role>

<context>
Empresas precisam de inteligência competitiva para tomar decisões estratégicas
informadas. Sucesso significa identificar oportunidades específicas que orientem
ações imediatas.
</context>

<input_handling>
Obrigatório: Setor, principais concorrentes, posição atual no mercado
Inferir se não fornecido: Escopo geográfico, linha do tempo da análise
</input_handling>

<task>
1. Mapear o cenário competitivo com matriz de posicionamento
2. Criar perfis dos principais concorrentes (forças, fraquezas, estratégias)
3. Identificar lacunas competitivas e oportunidades de espaço em branco
4. Elaborar recomendações de ação com cronogramas
</task>

<output_specification>
- Formato: Análise estratégica com frameworks visuais
- Extensão: 500-800 palavras
- Deve incluir: Mapa de posicionamento, perfis de concorrentes, plano de ação
</output_specification>
```

---

## Prompt Categories

### Negócios e Estratégia

| Categoria                                                | Prompts | Principais Casos de Uso                         |
| -------------------------------------------------------- | ------- | ------------------------------------------------ |
| [Análise de Negócios](prompts/business/)                 | 45+     | Engenharia de requisitos, melhoria de processos  |
| [Finanças](prompts/finance/)                             | 30+     | Modelagem financeira, análise de investimentos   |
| [Marketing](prompts/business/marketing/)                 | 25+     | Estratégia de campanhas, desenvolvimento de marca |
| [Operações](prompts/operations/)                         | 35+     | Otimização de processos, cadeia de suprimentos   |
| [Gestão de Projetos](prompts/project-management/)        | 40+     | Avaliação de riscos, metodologias ágeis          |

### Tecnologia e Engenharia

| Categoria                                                  | Prompts | Principais Casos de Uso                          |
| ---------------------------------------------------------- | ------- | ------------------------------------------------ |
| [Engenharia de Software](prompts/technical/)               | 50+     | Design de arquitetura, revisão de código         |
| [DevOps](prompts/technical/devops/)                        | 25+     | Pipelines de CI/CD, infraestrutura como código   |
| [Segurança](prompts/security/)                             | 20+     | Modelagem de ameaças, resposta a incidentes      |
| [Ciência de Dados](prompts/technical/data-science/)        | 30+     | Desenvolvimento de ML, análise de dados          |

### Tecnologias Emergentes

| Categoria                                                    | Prompts | Principais Casos de Uso                              |
| ------------------------------------------------------------ | ------- | ---------------------------------------------------- |
| [Computação Quântica](prompts/quantum-computing/)            | 14      | Desenvolvimento de algoritmos, otimização de circuitos |
| [Blockchain e Web3](prompts/blockchain/)                     | 15      | Contratos inteligentes, protocolos DeFi              |
| [Biotecnologia](prompts/biotechnology/)                      | 15      | Descoberta de medicamentos, bioinformática           |
| [Economia Espacial](prompts/space-economy/)                  | 24      | Operações de satélite, planejamento de missões       |
| [Energia Renovável](prompts/renewable-energy/)               | 19      | Desenvolvimento solar, integração à rede             |
| [Saúde Digital](prompts/healthcare-digital/)                 | 20      | Telemedicina, diagnósticos por IA                    |

### Criatividade e Comunicação

| Categoria                                                        | Prompts | Principais Casos de Uso                            |
| ---------------------------------------------------------------- | ------- | -------------------------------------------------- |
| [Criatividade](prompts/creative/)                                | 25+     | Design gráfico, pesquisa de UX                     |
| [Comunicação](prompts/communication/)                            | 30+     | Apresentações, redação técnica                     |
| [Aprendizagem e Desenvolvimento](prompts/learning-development/) | 20+     | Design de currículo, programas de treinamento      |

---

## Skills for Claude Code (260+)

As skills são ativadas automaticamente quando o Claude Code detecta palavras-chave relevantes:

| Domínio                          | Skills | Exemplos                                           |
| -------------------------------- | ------ | -------------------------------------------------- |
| **Desenvolvimento de Software**  | 35     | refactor-legacy-code, code-review-analysis         |
| **DevOps e Infraestrutura**      | 20     | docker-containerization, kubernetes-deployment     |
| **Testes e QA**                  | 15     | unit-testing-framework, e2e-testing                |
| **Segurança**                    | 15     | vulnerability-scanning, oauth-implementation       |
| **API e Integração**             | 12     | rest-api-design, graphql-implementation            |
| **Banco de Dados**               | 12     | sql-optimization, schema-design                    |

Consulte [SKILLS-MATRIX.md](SKILLS-MATRIX.md) para a referência completa.

---

## Automation Hooks (7)

| Hook                                            | Gatilho        | Finalidade                                |
| ----------------------------------------------- | -------------- | ----------------------------------------- |
| [security-scan](hooks/security-scan/)           | Pré-commit     | Escanear vulnerabilidades e segredos      |
| [pre-commit-linting](hooks/pre-commit-linting/) | Pré-commit     | Aplicação de formatação de código         |
| [test-runner](hooks/test-runner/)               | Pré-commit     | Execução automatizada de testes           |
| [dependency-check](hooks/dependency-check/)     | Pré-commit     | Auditoria de vulnerabilidades em dependências |
| [auto-format](hooks/auto-format/)               | Pós-salvamento | Formatação automática de código           |
| [session-setup](hooks/session-setup/)           | Início de sessão | Inicialização do ambiente               |

Consulte [HOOKS-LIBRARY.md](HOOKS-LIBRARY.md) para instalação e configuração.

---

## Repository Structure

```
useful-ai-prompts/
├── prompts/               # 679+ prompts padronizados por categoria
│   ├── analysis/          # Análise competitiva, de dados, financeira
│   ├── technical/         # Software, DevOps, segurança, ciência de dados
│   ├── business/          # Finanças, marketing, operações
│   ├── blockchain/        # Web3, DeFi, contratos inteligentes
│   ├── quantum-computing/ # Algoritmos quânticos, circuitos
│   └── ...                # 47 categorias no total
├── skills/                # 260+ skills para Claude Code
├── hooks/                 # 7 hooks de automação
├── docs/                  # Site Jekyll (GitHub Pages)
├── .claude/skills/        # Skill de refatoração de prompts
│   └── prompt-refactor/   # Especificação de template e validação
├── PROMPT-INDEX.json      # Catálogo de prompts legível por máquina
├── SKILLS-MATRIX.md       # Referência completa de skills
└── HOOKS-LIBRARY.md       # Documentação de hooks
```

---

## Integration

### For AI Agents

```python
import json

# Carregar índice de prompts
with open('PROMPT-INDEX.json') as f:
    prompts = json.load(f)

# Selecionar por categoria
prompts_analise = [p for p in prompts if p['category'] == 'analysis']
```

Consulte [AI-AGENT-GUIDE.md](AI-AGENT-GUIDE.md) para classificação de tarefas e padrões de integração com APIs.

### For Users

Consulte [README-HUMANS.md](README-HUMANS.md) para primeiros passos, dicas de personalização e exemplos práticos.

---

## Web Interface

Navegue pelos prompts com busca e filtros em:
**[https://aj-geddes.github.io/useful-ai-prompts/](https://aj-geddes.github.io/useful-ai-prompts/)**

Funcionalidades:

- Navegação por categorias
- Busca por texto completo
- Design responsivo
- Copiar para a área de transferência

---

## Contributing

Contribuições são bem-vindas! Todos os novos prompts devem:

1. Seguir a estrutura do template padronizado
2. Passar por todas as 11 verificações de qualidade
3. Incluir exemplos de uso realistas

Use a skill de refatoração de prompts para validação:

```bash
./.claude/skills/prompt-refactor/scripts/validate-prompt.sh ./prompts/my-prompt.md
```

Consulte [CONTRIBUTING.md](CONTRIBUTING.md) para as diretrizes completas.

---

## License

Licença MIT — veja [LICENSE](LICENSE) para detalhes.

---

## Links

- **Website**: [https://aj-geddes.github.io/useful-ai-prompts/](https://aj-geddes.github.io/useful-ai-prompts/)
- **Original Repository**: [https://github.com/aj-geddes/useful-ai-prompts](https://github.com/aj-geddes/useful-ai-prompts)
- **Issues**: [Reportar bugs ou solicitar funcionalidades](https://github.com/aj-geddes/useful-ai-prompts/issues)
- **Discussions**: [Perguntas e respostas da comunidade](https://github.com/aj-geddes/useful-ai-prompts/discussions)

---

**Palavras-chave**: prompts de IA, prompts ChatGPT, prompts Claude, engenharia de prompts, prompts padronizados, produtividade com IA, skills Claude Code, biblioteca de prompts, prompts LLM, ferramentas profissionais de IA
