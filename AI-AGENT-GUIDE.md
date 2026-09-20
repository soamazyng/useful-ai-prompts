# Guia de Integração para Agentes de IA

Especificações técnicas para agentes de IA, assistentes e sistemas de automação acessarem e utilizarem programaticamente a biblioteca Useful AI Prompts.

## Visão Geral da Biblioteca

| Recurso     | Qtd   | Localização         | Descrição                                            |
| ----------- | ----- | ------------------- | ---------------------------------------------------- |
| **Prompts** | 557+  | `/prompts/`         | Prompts criados por especialistas em 47 categorias   |
| **Skills**  | 260+  | `/skills/`          | Habilidades de ativação automática do Claude Code    |
| **Hooks**   | 7     | `/hooks/`           | Scripts de automação orientados a eventos            |
| **Índice**  | 1     | `PROMPT-INDEX.json` | Catálogo de prompts legível por máquina              |

---

## Arquitetura de Integração

```mermaid
graph LR
    A[Solicitação do Usuário] --> B[Agente de IA]
    B --> C[Analisador de Tarefa]
    C --> D{Tipo de Recurso?}
    D -->|Prompt| E[Seletor de Prompt]
    D -->|Skill| F[Matcher de Skill]
    D -->|Hook| G[Gatilho de Hook]
    E --> H[Biblioteca de Prompts]
    F --> I[Biblioteca de Skills]
    G --> J[Biblioteca de Hooks]
    H --> K[Injetor de Variáveis]
    I --> K
    K --> L[Motor de Execução]
    L --> M[Saída Estruturada]
```

---

## Seleção de Prompts

### Taxonomia de Categorias

```yaml
# 47 categorias organizadas por domínio
categories:
  business:
    - business
    - finance
    - financial-planning
    - marketing
    - operations
    - project-management
    - human-resources
    - customer-service
    - administrative

  technical:
    - technical
    - technical-workflows
    - technical-templates
    - security
    - development

  creative:
    - creative
    - content-creation
    - communication

  research:
    - research
    - research-workflows
    - academic

  specialized:
    - healthcare
    - healthcare-digital
    - engineering
    - education

  emerging_tech:
    - quantum-computing
    - blockchain
    - biotechnology
    - space-economy
    - renewable-energy
    - government
    - supply-chain

  personal:
    - personal-growth
    - personal-productivity
    - career-development
    - health-wellness
    - relationships-communication
    - learning-skills
    - learning-development

  workflow:
    - analysis
    - planning
    - problem-solving
    - decision-making
    - evaluation-assessment
    - creativity-innovation
    - management-leadership
    - optimization
    - customer-focused
    - creation
```

### Algoritmo de Seleção

```python
def select_prompt(user_request: str) -> str:
    """
    Seleciona o prompt mais apropriado baseado na análise da solicitação.
    """
    # Passo 1: Extrair indicadores-chave
    indicators = extract_indicators(user_request)

    # Passo 2: Comparar com a taxonomia de prompts
    category = match_category(indicators)
    subcategory = match_subcategory(indicators, category)

    # Passo 3: Pontuar prompts específicos
    candidates = load_prompts(category, subcategory)
    scores = score_prompts(candidates, indicators)

    # Passo 4: Retornar melhor correspondência
    return select_best_match(scores)


def extract_indicators(request: str) -> dict:
    """Extrai indicadores de classificação da solicitação."""
    return {
        'keywords': extract_keywords(request),
        'domain': detect_domain(request),
        'task_type': detect_task_type(request),
        'complexity': assess_complexity(request),
        'output_format': detect_output_format(request)
    }
```

### Palavras-Chave de Classificação de Tarefas

```yaml
technical:
  keywords:
    [
      code,
      develop,
      debug,
      deploy,
      architecture,
      security,
      data,
      api,
      database,
      test,
    ]
  subcategories:
    software-engineering: [build, implement, refactor, optimize, review]
    devops: [pipeline, deployment, CI/CD, infrastructure, container, kubernetes]
    security: [threat, vulnerability, compliance, audit, encryption]
    data-science:
      [model, analysis, prediction, validation, ml, machine learning]

business:
  keywords: [strategy, manage, analyze, plan, organize, lead, budget, forecast]
  subcategories:
    finance: [budget, forecast, valuation, investment, roi, financial]
    marketing: [campaign, brand, audience, conversion, content, seo]
    operations: [process, efficiency, workflow, optimization, supply chain]
    management: [team, project, resource, stakeholder, leadership]

emerging_tech:
  keywords: [quantum, blockchain, biotech, space, renewable, web3, defi]
  subcategories:
    quantum-computing: [qubit, circuit, algorithm, quantum ml, optimization]
    blockchain: [smart contract, defi, nft, tokenization, web3]
    biotechnology: [drug discovery, genomics, crispr, clinical trial]
    space-economy: [satellite, spacecraft, mission, orbital, launch]
    renewable-energy: [solar, wind, battery, grid, sustainability]

creative:
  keywords: [design, create, brand, user, experience, content, visual]
  subcategories:
    design: [visual, graphic, brand, identity, ui]
    ux-design: [user, research, interface, experience, usability]
    content: [write, editorial, strategy, messaging, copywriting]

specialized:
  keywords: [research, healthcare, education, engineering, pharmaceutical]
  subcategories:
    healthcare: [clinical, pharmaceutical, patient, treatment, medical]
    research: [study, analysis, hypothesis, methodology, academic]
    engineering: [build, construct, manufacture, technical, mechanical]
```

### Mapeamento Direto de Tarefas

```python
TASK_TO_PROMPT_MAP = {
    # Tarefas financeiras
    "analyze financial performance": "financial-analysis-expert",
    "create financial projections": "financial-model-builder",
    "evaluate investment": "financial-analysis-expert",

    # Tarefas de desenvolvimento
    "build web application": "fullstack-developer-architect",
    "optimize code performance": "algorithm-optimization-expert",
    "design system architecture": "system-architecture-design-expert",

    # Tarefas de segurança
    "security assessment": "cybersecurity-defense-architect",
    "incident response": "incident-response-commander",
    "design secure architecture": "security-implementation-expert",

    # Análise de negócios
    "gather requirements": "requirements-engineering-expert",
    "process improvement": "process-optimization-expert",
    "create specifications": "specification-creation-expert",

    # Tecnologias emergentes
    "quantum algorithm": "quantum-circuit-optimization-design",
    "smart contract": "smart-contract-security-audit-platform",
    "drug discovery": "ai-powered-drug-screening-optimization",
    "satellite operations": "commercial-space-mission-architecture",
    "solar project": "utility-scale-solar-farm-development"
}
```

---

## Integração de Skills

### O que São Skills?

Skills são capacidades especializadas do Claude Code que são ativadas automaticamente baseadas em palavras-chave da solicitação. Diferente dos prompts, skills:

- São ativadas automaticamente sem seleção explícita
- Fornecem orientação passo a passo com exemplos de código
- Incluem implementações em múltiplas linguagens
- Variam de 200 a 500+ linhas de instruções detalhadas

### Correspondência de Skills

```python
def match_skill(user_request: str) -> Optional[str]:
    """
    Corresponde a solicitação do usuário à skill apropriada baseada em palavras-chave.
    """
    skills_index = load_skills_index()

    for skill in skills_index:
        if any(trigger in user_request.lower() for trigger in skill['triggers']):
            return skill['name']

    return None


# Exemplo de gatilhos de skills
SKILL_TRIGGERS = {
    "refactor-legacy-code": ["refactor", "modernize", "legacy code", "technical debt"],
    "docker-containerization": ["docker", "containerize", "container", "dockerfile"],
    "unit-testing-framework": ["unit test", "test coverage", "testing framework"],
    "rest-api-design": ["api design", "rest api", "endpoint design"],
    "sql-optimization": ["sql optimization", "query performance", "slow query"],
    "kubernetes-deployment": ["kubernetes", "k8s", "deploy to cluster"],
    "security-audit": ["security audit", "vulnerability scan", "penetration test"]
}
```

### Skills por Domínio

| Domínio                    | Qtd | Skills Principais                                             |
| -------------------------- | --- | ------------------------------------------------------------- |
| Desenvolvimento de Software | 35  | refactor-legacy-code, code-review-analysis, design-patterns   |
| DevOps                     | 20  | docker-containerization, kubernetes-deployment, terraform-iac |
| Testes                     | 15  | unit-testing-framework, e2e-testing, test-automation          |
| Segurança                  | 15  | vulnerability-scanning, oauth-implementation, data-encryption |
| API                        | 12  | rest-api-design, graphql-implementation, webhook-development  |
| Banco de Dados             | 12  | sql-optimization, schema-design, database-indexing            |
| Cloud                      | 15  | aws-lambda, serverless-architecture, cloud-cost-optimization  |
| Frontend                   | 12  | react-components, responsive-design, css-architecture         |
| Backend                    | 12  | nodejs-express, django-application, background-jobs           |
| ML/IA                      | 10  | ml-model-training, model-deployment, hyperparameter-tuning    |

### Carregando Skills

```python
import os
from pathlib import Path

def load_skill(skill_name: str) -> str:
    """Carrega conteúdo da skill do diretório de skills."""
    skill_path = Path("skills") / f"{skill_name}.md"
    if skill_path.exists():
        return skill_path.read_text()
    raise FileNotFoundError(f"Skill não encontrada: {skill_name}")


def list_skills() -> list:
    """Lista todas as skills disponíveis."""
    skills_dir = Path("skills")
    return [f.stem for f in skills_dir.glob("*.md")]
```

---

## Integração de Hooks

### O que São Hooks?

Hooks são scripts de automação que executam em resposta a eventos do Claude Code:

| Hook                      | Evento Gatilho  | Finalidade                                   |
| ------------------------- | --------------- | -------------------------------------------- |
| security-scan             | Pré-commit      | Escanear vulnerabilidades e segredos         |
| pre-commit-linting        | Pré-commit      | Formatação e estilo de código                |
| test-runner               | Pré-commit      | Executar testes automatizados                |
| dependency-check          | Pré-commit      | Auditar dependências                         |
| breaking-change-detection | Pré-commit      | Detectar breaking changes em APIs            |
| auto-format               | Pós-salvamento  | Formatação automática de código              |
| session-setup             | Início de sessão | Inicialização de ambiente                   |

### Execução de Hooks

```python
def execute_hook(hook_name: str, context: dict) -> dict:
    """
    Executa um hook com o contexto fornecido.
    """
    hook_path = Path("hooks") / hook_name
    if not hook_path.exists():
        raise FileNotFoundError(f"Hook não encontrado: {hook_name}")

    # Carregar configuração do hook
    config = load_hook_config(hook_path)

    # Executar script do hook
    result = run_hook_script(config, context)

    return {
        'status': result.returncode == 0,
        'output': result.stdout,
        'errors': result.stderr
    }
```

---

## Schema de Metadados de Prompts

```json
{
  "prompt_id": "financial-analysis-expert",
  "file_path": "prompts/finance/financial-analysis-expert.md",
  "category": "finance",
  "subcategory": "analysis",
  "title": "Financial Analysis Expert",
  "description": "Análise financeira especializada com avaliação de investimentos e gestão de portfólio",
  "tags": ["finance", "investment", "analysis", "valuation", "portfolio"],
  "use_cases": [
    "avaliação de empresa",
    "decisões de investimento",
    "revisão de portfólio",
    "análise de mercado"
  ],
  "complexity_level": "advanced",
  "estimated_output_lines": 600
}
```

---

## Injeção de Variáveis

### Variáveis Padrão

```yaml
common_variables:
  - company_name: "Nome da organização"
  - industry: "Setor da indústria"
  - team_size: "Número de membros da equipe"
  - timeline: "Cronograma do projeto"
  - budget: "Orçamento disponível"
  - constraints: "Limitações específicas"
  - goals: "Resultados desejados"

domain_specific:
  technical:
    - technology_stack: "Stack tecnológica atual"
    - architecture_type: "Arquitetura do sistema"
    - performance_requirements: "Metas de desempenho"
    - security_requirements: "Padrões de segurança"
    - codebase_size: "Linhas de código ou tamanho do repositório"

  business:
    - market_conditions: "Estado atual do mercado"
    - competition: "Cenário competitivo"
    - regulatory_environment: "Requisitos de conformidade"
    - stakeholders: "Stakeholders principais"
    - revenue_model: "Modelo de receita do negócio"

  emerging_tech:
    - technology_readiness: "Nível TRL"
    - regulatory_status: "Status de aprovação regulatória"
    - infrastructure_requirements: "Necessidades de infraestrutura"
```

### Extração de Variáveis

```python
def extract_variables(user_request: str, prompt_template: str) -> dict:
    """
    Extrai valores para variáveis do prompt a partir da solicitação do usuário.
    """
    variables = {}
    required_vars = extract_template_variables(prompt_template)

    for var in required_vars:
        value = (
            extract_explicit_value(user_request, var) or
            extract_implicit_value(user_request, var) or
            infer_from_context(user_request, var) or
            get_default_value(var)
        )
        variables[var] = value

    return variables


def extract_template_variables(template: str) -> list:
    """Extrai nomes de variáveis do template."""
    import re
    pattern = r'\{\{(\w+)\}\}'
    return re.findall(pattern, template)
```

---

## Especificação de API

### Endpoints RESTful

```yaml
endpoints:
  # Prompts
  GET /api/prompts:
    description: "Listar todos os prompts disponíveis"
    parameters:
      - category: "Filtrar por categoria"
      - tags: "Filtrar por tags (separadas por vírgula)"
      - search: "Busca por texto completo"
    response:
      - prompts: "Array de metadados de prompts"
      - total: "Contagem total"

  GET /api/prompts/{prompt_id}:
    description: "Obter prompt específico"
    response:
      - metadata: "Metadados do prompt"
      - content: "Conteúdo do prompt"
      - variables: "Variáveis obrigatórias"

  POST /api/match:
    description: "Encontrar prompt com melhor correspondência"
    body:
      - request: "Texto da solicitação do usuário"
      - context: "Contexto adicional"
      - preferences: "Preferências do usuário"
    response:
      - prompt_id: "Prompt com melhor correspondência"
      - confidence: "Score de confiança da correspondência (0-1)"
      - alternatives: "Outras correspondências potenciais"

  # Skills
  GET /api/skills:
    description: "Listar todas as skills disponíveis"
    parameters:
      - domain: "Filtrar por domínio"
    response:
      - skills: "Array de metadados de skills"

  GET /api/skills/{skill_name}:
    description: "Obter skill específica"
    response:
      - metadata: "Metadados da skill"
      - content: "Conteúdo da skill"
      - triggers: "Palavras-chave de ativação"

  # Hooks
  GET /api/hooks:
    description: "Listar todos os hooks disponíveis"
    response:
      - hooks: "Array de metadados de hooks"

  POST /api/hooks/{hook_name}/execute:
    description: "Executar um hook"
    body:
      - context: "Contexto de execução"
    response:
      - status: "Sucesso/falha"
      - output: "Saída do hook"
```

---

## Framework de Execução

### Processamento em Quatro Fases

```python
class PromptExecutor:
    def execute(self, prompt: str, variables: dict) -> str:
        """
        Executa prompt através do framework padrão de quatro fases.
        """
        # Preparar prompt com variáveis
        prepared_prompt = self.inject_variables(prompt, variables)

        # Fase 1: Avaliação/Análise
        assessment = self.execute_phase("assessment", prepared_prompt)

        # Fase 2: Design Estratégico
        strategy = self.execute_phase("strategy", assessment)

        # Fase 3: Implementação/Execução
        implementation = self.execute_phase("implementation", strategy)

        # Fase 4: Otimização/Controle
        optimization = self.execute_phase("optimization", implementation)

        # Compilar saída estruturada
        return self.compile_output([
            assessment,
            strategy,
            implementation,
            optimization
        ])
```

### Estrutura de Saída

```yaml
output_structure:
  executive_summary:
    - key_findings: "Top 3-5 insights"
    - recommendations: "Ações primárias"
    - impact_assessment: "Resultados esperados"

  detailed_analysis:
    - current_state: "Avaliação abrangente"
    - gap_analysis: "Lacunas identificadas"
    - root_causes: "Problemas subjacentes"

  strategic_plan:
    - objectives: "Metas SMART"
    - strategies: "Abordagem para cada objetivo"
    - tactics: "Ações específicas"

  implementation_roadmap:
    - phases: "Divisão por cronograma"
    - milestones: "Entregáveis-chave"
    - resources: "Recursos necessários"

  risk_management:
    - risk_assessment: "Riscos identificados"
    - mitigation_strategies: "Respostas aos riscos"
    - contingency_plans: "Abordagens de contingência"

  metrics_and_monitoring:
    - kpis: "Indicadores-chave de desempenho"
    - dashboards: "Abordagem de monitoramento"
    - review_cycles: "Cronograma de avaliação"
```

---

## Otimização de Desempenho

### Estratégia de Cache

```python
from functools import lru_cache
from pathlib import Path

@lru_cache(maxsize=100)
def get_prompt_content(prompt_id: str) -> str:
    """Cacheia conteúdo do prompt para acesso repetido."""
    prompt_path = find_prompt_path(prompt_id)
    return prompt_path.read_text()


@lru_cache(maxsize=50)
def get_skill_content(skill_name: str) -> str:
    """Cacheia conteúdo da skill para acesso repetido."""
    skill_path = Path("skills") / f"{skill_name}.md"
    return skill_path.read_text()


def clear_caches():
    """Limpa todos os caches quando o conteúdo é atualizado."""
    get_prompt_content.cache_clear()
    get_skill_content.cache_clear()
```

### Processamento em Lote

```python
from concurrent.futures import ThreadPoolExecutor

def batch_process_requests(requests: list[dict]) -> list[dict]:
    """Processa múltiplas solicitações de forma eficiente."""
    # Agrupar por tipo de recurso
    prompts_requests = [r for r in requests if r['type'] == 'prompt']
    skills_requests = [r for r in requests if r['type'] == 'skill']

    results = []

    # Processar em paralelo
    with ThreadPoolExecutor(max_workers=4) as executor:
        prompt_futures = [
            executor.submit(process_prompt_request, r)
            for r in prompts_requests
        ]
        skill_futures = [
            executor.submit(process_skill_request, r)
            for r in skills_requests
        ]

        results.extend([f.result() for f in prompt_futures])
        results.extend([f.result() for f in skill_futures])

    return results
```

---

## Tratamento de Erros

```python
class PromptLibraryError(Exception):
    """Exceção base para erros da biblioteca de prompts."""
    pass

class PromptNotFoundError(PromptLibraryError):
    """Prompt solicitado não existe."""
    pass

class SkillNotFoundError(PromptLibraryError):
    """Skill solicitada não existe."""
    pass

class VariableExtractionError(PromptLibraryError):
    """Falha ao extrair variáveis obrigatórias."""
    pass

def handle_error(error: Exception) -> dict:
    """Tratamento de erros elegante com fallbacks."""
    if isinstance(error, PromptNotFoundError):
        return {
            'status': 'error',
            'message': f'Prompt não encontrado: {error}',
            'suggestion': 'Tente navegar pelos prompts disponíveis em /api/prompts',
            'fallback': get_general_purpose_prompt()
        }
    elif isinstance(error, SkillNotFoundError):
        return {
            'status': 'error',
            'message': f'Skill não encontrada: {error}',
            'suggestion': 'Tente navegar pelas skills disponíveis em /api/skills'
        }
    elif isinstance(error, VariableExtractionError):
        return {
            'status': 'partial',
            'message': 'Algumas variáveis não puderam ser extraídas',
            'missing_variables': error.missing_vars,
            'partial_result': execute_with_defaults(error.prompt)
        }
    else:
        return {
            'status': 'error',
            'message': str(error)
        }
```

---

## Testes

### Validação de Prompts

```python
def validate_prompt(prompt_path: str) -> bool:
    """Valida se o prompt atende aos padrões de qualidade."""
    prompt = load_prompt(prompt_path)

    # Verificar estrutura
    assert has_metadata_section(prompt), "Metadados ausentes"
    assert has_use_cases(prompt), "Casos de uso ausentes"
    assert has_deliverables(prompt), "Entregáveis ausentes"

    # Verificar qualidade do conteúdo
    assert len(prompt) > 500, "Prompt muito curto"
    assert has_context_questions(prompt), "Perguntas de contexto ausentes"

    return True


def validate_skill(skill_path: str) -> bool:
    """Valida se a skill atende aos padrões de qualidade."""
    skill = load_skill(skill_path)

    # Verificar estrutura
    assert has_triggers(skill), "Palavras-chave de ativação ausentes"
    assert has_code_examples(skill), "Exemplos de código ausentes"
    assert has_best_practices(skill), "Boas práticas ausentes"

    # Verificar conteúdo
    assert len(skill) > 200, "Skill muito curta"

    return True
```

---

## Exemplos de Integração

### Integração Python

```python
import requests

class PromptLibraryClient:
    def __init__(self, base_url: str = "https://api.example.com"):
        self.base_url = base_url

    def find_prompt(self, user_request: str) -> dict:
        """Encontra o prompt com melhor correspondência para a solicitação."""
        response = requests.post(
            f"{self.base_url}/api/match",
            json={"request": user_request}
        )
        return response.json()

    def get_prompt(self, prompt_id: str) -> dict:
        """Obtém conteúdo e metadados do prompt."""
        response = requests.get(
            f"{self.base_url}/api/prompts/{prompt_id}"
        )
        return response.json()

    def list_skills(self, domain: str = None) -> list:
        """Lista skills disponíveis."""
        params = {"domain": domain} if domain else {}
        response = requests.get(
            f"{self.base_url}/api/skills",
            params=params
        )
        return response.json()["skills"]


# Uso
client = PromptLibraryClient()
match = client.find_prompt("Help me analyze financial performance")
prompt = client.get_prompt(match["prompt_id"])
```

### Integração JavaScript/TypeScript

```typescript
interface PromptMatch {
  prompt_id: string;
  confidence: number;
  alternatives: string[];
}

interface Prompt {
  metadata: Record<string, unknown>;
  content: string;
  variables: string[];
}

class PromptLibrary {
  constructor(private baseUrl: string = "https://api.example.com") {}

  async findPrompt(request: string): Promise<PromptMatch> {
    const response = await fetch(`${this.baseUrl}/api/match`, {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ request }),
    });
    return response.json();
  }

  async getPrompt(promptId: string): Promise<Prompt> {
    const response = await fetch(`${this.baseUrl}/api/prompts/${promptId}`);
    return response.json();
  }

  async listSkills(domain?: string): Promise<string[]> {
    const params = domain ? `?domain=${domain}` : "";
    const response = await fetch(`${this.baseUrl}/api/skills${params}`);
    const data = await response.json();
    return data.skills;
  }
}
```

---

## Padrões de Acesso a Arquivos

### Acesso Direto a Arquivos

```python
from pathlib import Path
import json

# Carregar índice de prompts
def load_prompt_index() -> list:
    with open("PROMPT-INDEX.json") as f:
        return json.load(f)

# Encontrar arquivo de prompt
def find_prompt_file(prompt_id: str) -> Path:
    prompts_dir = Path("prompts")
    for md_file in prompts_dir.rglob("*.md"):
        if md_file.stem == prompt_id:
            return md_file
    raise FileNotFoundError(f"Prompt não encontrado: {prompt_id}")

# Listar todas as skills
def list_all_skills() -> list:
    skills_dir = Path("skills")
    return sorted([f.stem for f in skills_dir.glob("*.md")])

# Listar todos os hooks
def list_all_hooks() -> list:
    hooks_dir = Path("hooks")
    return [d.name for d in hooks_dir.iterdir()
            if d.is_dir() and not d.name.startswith('.')]
```

---

## Suporte

- **Documentação**: [README.md](README.md)
- **Guia do Usuário**: [README-HUMANS.md](README-HUMANS.md)
- **Referência de Skills**: [SKILLS-MATRIX.md](SKILLS-MATRIX.md)
- **Referência de Hooks**: [HOOKS-LIBRARY.md](HOOKS-LIBRARY.md)
- **Issues**: [GitHub Issues](https://github.com/aj-geddes/useful-ai-prompts/issues)
