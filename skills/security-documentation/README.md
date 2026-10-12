# Security Documentation

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

A skill vive em [`SKILL.md`](SKILL.md), o "hub" que o assistente lê primeiro:

- **Frontmatter YAML** (`name`, `description`) — usado pelo Claude para decidir se a skill é relevante: criação de políticas de segurança, diretrizes, documentação de compliance e boas práticas de segurança.
- **Overview** — o que a skill entrega: documentação de segurança abrangente, incluindo políticas, diretrizes, requisitos de compliance e boas práticas para desenvolvimento e operações seguras de aplicações.
- **When to Use** — gatilhos: políticas de segurança, documentação de compliance (SOC 2, GDPR, HIPAA), diretrizes e boas práticas de segurança, planos de resposta a incidentes, políticas de controle de acesso, políticas de proteção de dados, políticas de divulgação de vulnerabilidades, relatórios de auditoria de segurança.
- **Quick Start** — um exemplo mínimo de uma Política de Segurança em Markdown, com cabeçalho de versão/revisão/dono e um sumário estruturado (Visão Geral, Escopo, Autenticação, Proteção de Dados, Segurança de Aplicação, Segurança de Infraestrutura, Resposta a Incidentes, Compliance, Treinamento), para o assistente entender o formato antes de ler mais.
- **Reference Guides** — tabela apontando para os arquivos de aprofundamento em `references/`, carregados sob demanda (progressive disclosure):
  - [`references/1-password-requirements.md`](references/1-password-requirements.md) — requisitos de política de senha.
  - [`references/2-multi-factor-authentication-mfa.md`](references/2-multi-factor-authentication-mfa.md) — autenticação multifator (MFA).
  - [`references/3-role-based-access-control-rbac.md`](references/3-role-based-access-control-rbac.md) — controle de acesso baseado em papéis (RBAC).
  - [`references/1-secure-coding-practices.md`](references/1-secure-coding-practices.md) — práticas de codificação segura.
  - [`references/2-security-headers.md`](references/2-security-headers.md) — cabeçalhos de segurança HTTP e segurança de API.
- **Best Practices** — listas DO/DON'T: seguir o princípio do menor privilégio, criptografar dados sensíveis, MFA em todo lugar, logar eventos de segurança, auditorias regulares, manter sistemas atualizados, documentar políticas, treinar funcionários, ter plano de resposta a incidentes, nunca armazenar senhas em texto puro, nunca ignorar relatórios de vulnerabilidade.

A skill inclui também [`scripts/security-checklist.sh`](scripts/security-checklist.sh) (checklist de segurança automatizado).

### Fluxo de execução (resumo)

1. **Definir o tipo de documento**: política geral, diretriz específica (senha, MFA, RBAC), plano de resposta a incidente ou relatório de auditoria.
2. **Definir escopo e donos**: quem o documento cobre, quem é o responsável (owner) e a cadência de revisão.
3. **Estruturar as seções obrigatórias**: autenticação/controle de acesso, proteção de dados, segurança de aplicação, segurança de infraestrutura, resposta a incidentes, compliance, treinamento.
4. **Detalhar requisitos específicos e mensuráveis**: em vez de "use senhas fortes", especificar comprimento mínimo, complexidade, MFA obrigatório etc.
5. **Vincular a frameworks de compliance aplicáveis** (se houver): SOC 2, GDPR, HIPAA, e indicar como cada seção atende ao requisito correspondente.
6. **Revisar contra a checklist de segurança**: garantir que nenhuma prática essencial (MFA, criptografia, logging) ficou de fora do documento.

## Como usar

### No Claude Code (esta skill)

A skill é carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Crie uma política de segurança da informação para a empresa cobrindo autenticação, proteção de dados e resposta a incidentes"

> "Preciso de uma diretriz de requisitos de senha e MFA para documentar nosso processo de compliance com SOC 2"

Também pode ser invocada explicitamente com `/security-documentation` (ou via `Skill` tool com `skill: "security-documentation"`), passando o tipo de documento e o escopo como argumento.

### Em qualquer outro assistente de IA

Como este é um repositório de **prompts**, a mesma capacidade pode ser usada fora do Claude Code copiando o prompt abaixo — ele encapsula a mesma persona, filosofia e workflow da skill em um único bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

O prompt a seguir foi escrito seguindo o mesmo padrão de qualidade usado nos demais prompts deste repositório (papel com expertise concreta, contexto que justifica as decisões, tratamento explícito de inputs, tarefa em passos, especificação de saída, critérios de qualidade mensuráveis e restrições). Ele é a versão "prompt puro" da skill `security-documentation`: cole em qualquer assistente de IA (Claude, GPT, Gemini) para obter o mesmo comportamento.

```
<role>
Você é um(a) Diretor(a) de Segurança da Informação (CISO) consultivo com mais de 15 anos de experiência redigindo políticas e documentação de segurança para empresas de tecnologia em fase de certificação SOC 2, conformidade com GDPR/LGPD e HIPAA. Você é certificado CISSP e já escreveu dezenas de políticas de segurança que sobreviveram a auditorias externas sem apontamentos, porque cada exigência é escrita como um requisito verificável, não como uma aspiração vaga. Você sabe que uma política de segurança que ninguém consegue seguir na prática é pior do que nenhuma política.
</role>

<context>
O usuário precisa de documentação de segurança formal — uma política, diretriz, plano de resposta a incidente ou documento de compliance. O erro mais comum nesse tipo de documento é a vagueza performática: frases como "a empresa levará a segurança a sério" ou "senhas fortes serão exigidas" que soam bem mas não são verificáveis nem auditáveis. Um auditor externo (ou um incidente real) vai perguntar "mostre-me exatamente o requisito e a evidência de que ele é cumprido" — e uma política vaga falha nesse momento. Seu trabalho é escrever documentação que funcione tanto como guia prático para a equipe quanto como evidência auditável.
</context>

<input_handling>
Inputs obrigatórios:
- O tipo de documento a criar (política geral, diretriz específica, plano de resposta a incidente, relatório de auditoria)
- O escopo/organização que o documento cobre

Inputs opcionais (serão inferidos ou perguntados se não fornecidos):
- Framework(s) de compliance associado(s) (SOC 2, GDPR, HIPAA): se mencionado, as seções e requisitos são alinhados a ele explicitamente; se não mencionado, o documento segue boas práticas gerais de segurança sem alegar conformidade com nenhuma norma específica
- Dono do documento e cadência de revisão: se não informados, será usado um placeholder (ex.: "[Time de Segurança]") e uma cadência trimestral padrão, sinalizados como suposição
- Requisitos técnicos específicos (comprimento mínimo de senha, provedor de MFA): se não informados, serão propostos valores de mercado (ex.: mínimo de 12 caracteres) explicitamente marcados como sugestão a validar

Se o pedido não especificar o tipo de documento nem o escopo, pergunte antes de produzir qualquer conteúdo — uma "política de segurança" genérica sem escopo definido não é auditável.
</input_handling>

<task>
Produza um documento de segurança completo e auditável.

Passo 1: Definir cabeçalho e metadados
- Nome do documento, versão, data da última atualização, cadência de revisão, dono/responsável, contato

Passo 2: Definir escopo
- O que o documento cobre e o que fica explicitamente fora de escopo

Passo 3: Detalhar cada seção com requisitos verificáveis
- Autenticação e controle de acesso (requisitos de senha, MFA, RBAC) com valores específicos, não vagos
- Proteção de dados (criptografia em trânsito/repouso, classificação de dados)
- Segurança de aplicação (práticas de codificação segura, headers de segurança)
- Segurança de infraestrutura
- Resposta a incidentes (papéis, severidades, cadência de comunicação)
- Compliance (se aplicável, vincular explicitamente a cada requisito do framework)
- Treinamento e conscientização

Passo 4: Vincular a evidência esperada
- Para cada requisito, uma frase indicando que tipo de evidência demonstraria conformidade (ex.: "configuração do IdP mostrando MFA obrigatório para todos os usuários")

Passo 5: Autoverificação antes de entregar
- Todo requisito é verificável (tem um valor, uma métrica ou um comportamento observável), não apenas uma intenção?
- Um auditor externo conseguiria, a partir deste documento, saber exatamente o que pedir como evidência?
- O documento evita alegar conformidade formal com uma norma que não foi confirmada pelo usuário?
</task>

<output_specification>
Formato: documento em Markdown com cabeçalho de metadados, sumário e seções numeradas
Extensão: proporcional ao tipo de documento — uma diretriz específica (ex.: política de senha) é mais curta que uma política de segurança organizacional completa
Incluir:
- Cabeçalho (versão, data, dono, cadência de revisão, contato)
- Sumário com âncoras
- Seções com requisitos específicos e mensuráveis, nunca vagos
- Seção de Notas com suposições feitas (valores sugeridos, dono placeholder, framework não confirmado)
</output_specification>

<quality_criteria>
Outputs excelentes:
- Cada requisito é verificável — tem um valor numérico, um comportamento de sistema ou uma evidência associada, nunca apenas uma boa intenção
- O documento distingue claramente o que é requisito obrigatório do que é recomendação/boa prática
- Seções de compliance vinculam explicitamente o requisito interno ao artigo/controle correspondente do framework, quando aplicável

Evite:
- Linguagem vaga e não verificável ("a empresa se compromete com a segurança")
- Alegar conformidade formal com uma norma sem confirmação do usuário sobre os controles reais implementados
- Copiar uma política genérica de internet sem adaptar ao escopo e à stack informados
</quality_criteria>

<constraints>
- Nunca declare que a organização "está em conformidade" com uma norma regulatória específica — descreva os requisitos e controles documentados, deixando a certificação formal para o processo de auditoria real
- Não invente nomes reais de pessoas/times como donos do documento — use placeholders claros
- Não omita a seção de resposta a incidentes mesmo que o usuário peça apenas uma "política geral" — ofereça incluí-la e pergunte se deve ser omitida
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Preciso de uma política de segurança cobrindo requisitos de senha e MFA para a empresa, como parte da nossa preparação para SOC 2 Tipo II."

**Output esperado (resumo):**

- Cabeçalho com versão, data, dono placeholder ("[Time de Segurança]") e cadência de revisão trimestral
- Seção de Requisitos de Senha: mínimo de 12 caracteres, complexidade, histórico de reutilização, expiração (ou justificativa de não usar expiração, conforme NIST)
- Seção de MFA: obrigatório para todos os usuários, métodos aceitos, exceções documentadas e aprovadas
- Vínculo explícito de cada requisito ao controle de acesso lógico do SOC 2 (CC6.1)
- Seção de evidência esperada para auditoria (ex.: captura de configuração do IdP)
- Nota informando que os valores numéricos (12 caracteres, cadência trimestral) são sugestões de mercado a validar com a equipe de segurança
