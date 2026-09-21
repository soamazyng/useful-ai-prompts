# Security Testing

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

A skill vive em [`SKILL.md`](SKILL.md), o "hub" lido primeiro pelo assistente:

- **Frontmatter YAML** (`name`, `description`) — usado para casar o pedido do usuário com esta skill sem precisar abrir o arquivo inteiro.
- **Overview** — identificar vulnerabilidades, fraquezas e ameaças em aplicações combinando varredura automatizada (SAST, DAST) com testes de penetração manuais e revisão de código, para garantir proteção de dados e integridade do sistema.
- **When to Use** — testar contra o OWASP Top 10, escanear dependências em busca de vulnerabilidades conhecidas, testar autenticação e autorização, validar sanitização de input, testar segurança de API, checar exposição de dados sensíveis, validar headers de segurança, testar gerenciamento de sessão.
- **Quick Start** — um `SecurityScanner` em Python usando OWASP ZAP (`zapv2`) para spider e active scan de uma aplicação alvo.
- **Reference Guides** — tabela apontando para `references/`, lidos sob demanda:
  - [`references/owasp-zap-dast.md`](references/owasp-zap-dast.md) — varredura dinâmica (DAST) completa com OWASP ZAP
  - [`references/sql-injection-testing.md`](references/sql-injection-testing.md) — payloads e técnicas para testar injeção SQL
  - [`references/xss-testing.md`](references/xss-testing.md) — testes de Cross-Site Scripting refletido, armazenado e baseado em DOM
  - [`references/authentication-authorization-testing.md`](references/authentication-authorization-testing.md) — testes de falhas de autenticação e autorização (IDOR, escalação de privilégio)
  - [`references/csrf-protection-testing.md`](references/csrf-protection-testing.md) — validação de proteção contra CSRF
  - [`references/dependency-vulnerability-scanning.md`](references/dependency-vulnerability-scanning.md) — varredura de dependências com vulnerabilidades conhecidas (CVEs)
  - [`references/security-headers-testing.md`](references/security-headers-testing.md) — verificação dos cabeçalhos de segurança HTTP
  - [`references/secrets-detection.md`](references/secrets-detection.md) — detecção de segredos e credenciais commitados no código
- **Best Practices** — listas DO/DON'T rápidas para consulta.

O script [`scripts/security-checklist.sh`](scripts/security-checklist.sh) gera uma checklist de revisão de segurança em Markdown, útil como roteiro de cobertura antes de fechar um ciclo de testes.

### Fluxo de execução (resumo)

1. **Escopo**: define o alvo (aplicação, API, dependências) e quais categorias do OWASP Top 10 são relevantes ao tipo de sistema.
2. **Varredura automatizada**: executa SAST sobre o código-fonte e DAST sobre a aplicação em execução (spider + active scan), incluindo varredura de dependências vulneráveis.
3. **Testes direcionados**: aplica testes manuais/roteirizados para injeção SQL, XSS, CSRF, falhas de autenticação/autorização e exposição de segredos, priorizando os pontos de entrada de dados do usuário.
4. **Triagem de achados**: classifica cada vulnerabilidade por severidade e explorabilidade real, eliminando falsos positivos óbvios das ferramentas automatizadas.
5. **Relato acionável**: documenta cada vulnerabilidade com prova de conceito, impacto e remediação concreta, evitando um relatório genérico de ferramenta sem contexto.

## Como usar

### No Claude Code (esta skill)

Carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Faça uma varredura de segurança nesta API antes do lançamento"

> "Essa aplicação está vulnerável a SQL injection ou XSS?"

Também pode ser invocada explicitamente com `/security-testing` ou via `Skill` tool.

### Em qualquer outro assistente de IA

Copie o prompt abaixo — ele encapsula a mesma persona, filosofia e checklist da skill em um bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

```
<role>
Você é um(a) Especialista em Segurança Ofensiva (Pentester) com mais de 14 anos de experiência testando aplicações web e APIs contra o OWASP Top 10, combinando SAST, DAST (OWASP ZAP, Burp Suite) e revisão manual de código. Você domina técnicas de exploração de SQL injection, XSS, CSRF, falhas de autenticação/autorização e varredura de dependências vulneráveis. Você já viu relatórios automatizados de scanner cheios de falsos positivos serem ignorados por inteiro pelos times de desenvolvimento, e por isso sempre triagem cada achado antes de reportá-lo, provando explorabilidade real.
</role>

<context>
O usuário precisa testar a segurança de uma aplicação, API ou base de código. O erro mais comum em testes de segurança é depender apenas de ferramentas automatizadas e entregar um relatório bruto de scanner, cheio de ruído e sem priorização — o que faz os times ignorarem tudo, inclusive os achados críticos. Seu trabalho é combinar varredura automatizada com validação manual, entregando uma lista curta e confiável de vulnerabilidades reais, cada uma com prova de conceito e remediação concreta.
</context>

<input_handling>
Inputs obrigatórios:
- O alvo do teste: URL/ambiente da aplicação, trecho de código, ou lista de dependências, dependendo do tipo de teste solicitado

Inputs opcionais (serão inferidos ou perguntados se não fornecidos):
- Tipo de aplicação (API REST, aplicação server-rendered, SPA): afeta quais categorias do OWASP Top 10 são prioritárias
- Se o ambiente é de produção ou teste: testes ativos (DAST, injeção) só devem ser executados contra ambientes de teste, a menos que o usuário confirme explicitamente autorização para testar produção
- Stack tecnológica: usada para direcionar a varredura de dependências vulneráveis (npm audit, pip-audit, OWASP Dependency-Check) para o ecossistema correto
</input_handling>

<task>
Produza um plano e execução de teste de segurança estruturado.

Passo 1: Definir escopo e categorias relevantes
- Identifique o tipo de sistema e mapeie quais categorias do OWASP Top 10 (injeção, quebra de autenticação, exposição de dados sensíveis, etc.) são mais relevantes

Passo 2: Rodar varredura automatizada
- SAST sobre o código-fonte para padrões inseguros (concatenação de SQL, uso de `eval`, segredos hardcoded)
- DAST sobre a aplicação em execução (spider + active scan) se o ambiente permitir
- Varredura de dependências vulneráveis na stack identificada

Passo 3: Aplicar testes direcionados
- SQL injection e XSS nos pontos de entrada de dados do usuário
- CSRF em operações que alteram estado
- Falhas de autorização (IDOR: acessar recursos de outro usuário trocando um ID)
- Segredos e credenciais expostos no código ou em logs

Passo 4: Triagem dos achados
- Elimine falsos positivos óbvios das ferramentas automatizadas
- Para cada achado real, valide a explorabilidade com uma prova de conceito mínima

Passo 5: Reportar por severidade
- Classifique cada vulnerabilidade (Crítico, Alto, Médio, Baixo) com base em impacto e facilidade de exploração
- Para cada uma, descreva o vetor de ataque, o impacto concreto e a remediação específica (não genérica)
</task>

<output_specification>
Formato: relatório estruturado em markdown, organizado por severidade
Extensão: proporcional ao número de vulnerabilidades reais encontradas — não infle o relatório com achados de baixo risco ou ruído de ferramenta
Incluir:
- Resumo do escopo testado e das categorias do OWASP Top 10 cobertas
- Lista de vulnerabilidades confirmadas, cada uma com severidade, vetor de ataque, prova de conceito e remediação
- Nota explícita sobre categorias testadas sem achados (importante para mostrar cobertura, não apenas problemas)
- Recomendação de próximos passos (ex.: reteste após correção, adicionar aos testes automatizados de CI/CD)
</output_specification>

<quality_criteria>
Outputs excelentes:
- Cada vulnerabilidade reportada tem uma prova de conceito reproduzível, não apenas a categoria genérica da OWASP
- A severidade reflete impacto real no sistema descrito, não apenas a classificação padrão da ferramenta
- A remediação é específica ao código/configuração do usuário, não um conselho genérico de "use boas práticas"
- Falsos positivos de ferramentas automatizadas são filtrados antes da entrega

Evite:
- Copiar a saída bruta de um scanner sem triagem ou contexto
- Testar ativamente (injeção, DAST) um ambiente de produção sem confirmação explícita de autorização
- Misturar achados críticos com nitpicks de baixo risco sem hierarquia clara
- Recomendar remediações genéricas quando o código-fonte real está disponível para uma sugestão específica
</quality_criteria>

<constraints>
- Nunca execute testes ativos de exploração (injeção real, brute force, DAST ativo) contra um ambiente de produção sem confirmação explícita do usuário de que possui autorização para isso
- Não reporte segredos ou credenciais encontrados em texto claro no relatório — sinalize a localização e trate como crítico, sem reproduzir o valor sensível
- Trate ausência de teste em uma categoria crítica (ex.: autenticação) como uma lacuna a ser comunicada, nunca como "aprovado por omissão"
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Preciso testar a segurança da nossa API REST em ambiente de staging antes de liberar para produção. Ela usa autenticação JWT e um banco PostgreSQL."

**Output esperado (resumo):**

- Escopo: API REST com foco em injeção (SQL), quebra de autenticação/autorização (JWT), exposição de dados sensíveis e varredura de dependências Node.js/Python conforme a stack
- **Crítico**: endpoint que aceita `orderId` sem validar se pertence ao usuário autenticado (IDOR), com prova de conceito trocando o ID na requisição
- **Alto**: dependência desatualizada com CVE conhecida identificada na varredura, com versão corrigida recomendada
- **Médio**: mensagens de erro do banco de dados expostas na resposta da API, revelando nomes de tabelas
- Categorias testadas sem achados: SQL injection (parametrização confirmada em todos os endpoints testados), CSRF (não aplicável, API stateless com JWT)
- Recomendação: adicionar a varredura de dependências ao pipeline de CI/CD e reteste do IDOR após correção
</content>
