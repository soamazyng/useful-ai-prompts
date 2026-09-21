# Data Encryption

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

A skill vive em [`SKILL.md`](SKILL.md), o "hub" que o assistente lê primeiro. Ele segue a arquitetura de Progressive Disclosure do repositório:

- **Frontmatter YAML** (`name`, `description`) — usado pelo Claude para decidir, sem abrir o arquivo inteiro, se o pedido envolve implementar criptografia AES, RSA, TLS ou gerenciamento de chaves.
- **Overview** — define o escopo: implementar estratégias robustas de criptografia para proteger dados sensíveis em repouso e em trânsito, usando algoritmos criptográficos e práticas de gerenciamento de chaves consolidados na indústria.
- **When to Use** — gatilhos: armazenamento de dados sensíveis, criptografia de banco de dados, criptografia de arquivos, segurança de comunicação, requisitos de compliance (GDPR, HIPAA, PCI-DSS), armazenamento de senhas, criptografia ponta a ponta.
- **Quick Start** — um exemplo mínimo em Node.js de uma classe `EncryptionService` usando AES-256-GCM, com geração de chave criptograficamente segura e derivação de chave a partir de senha via PBKDF2.
- **Reference Guides** — tabela apontando para os quatro arquivos de aprofundamento em `references/`, carregados sob demanda:
  - [`references/nodejs-encryption-library.md`](references/nodejs-encryption-library.md) — implementação completa de um serviço de criptografia em Node.js usando o módulo `crypto` nativo.
  - [`references/python-cryptography-implementation.md`](references/python-cryptography-implementation.md) — implementação equivalente em Python usando a biblioteca `cryptography`.
  - [`references/database-encryption-postgresql.md`](references/database-encryption-postgresql.md) — criptografia de dados em repouso no PostgreSQL (colunas sensíveis, `pgcrypto`, criptografia transparente de disco).
  - [`references/tlsssl-configuration.md`](references/tlsssl-configuration.md) — configuração de TLS/SSL para proteger dados em trânsito, incluindo escolha de versão de protocolo e cipher suites.
- **Best Practices** — listas DO/DON'T rápidas para consulta.

Um arquivo de apoio completa a skill:

- [`scripts/security-checklist.sh`](scripts/security-checklist.sh) — gera um checklist de revisão de segurança em Markdown (autenticação, autorização, sessão), útil como ponto de partida para validar a implementação de criptografia junto com outros controles de segurança.

### Fluxo de execução (resumo)

1. **Classificação dos dados**: identificar quais dados são sensíveis (PII, credenciais, dados financeiros/de saúde) e se precisam de proteção em repouso, em trânsito, ou ambos.
2. **Escolha do algoritmo**: AES-256-GCM para criptografia simétrica de dados em repouso; RSA-4096/ECC para criptografia assimétrica ou troca de chaves; TLS 1.2+ para dados em trânsito.
3. **Gerenciamento de chaves**: definir onde e como as chaves são geradas, armazenadas (HSM, KMS, variável de ambiente segura — nunca no código) e rotacionadas periodicamente.
4. **Implementação**: aplicar a biblioteca/serviço apropriado à linguagem, usando criptografia autenticada (GCM) e IVs/nonces únicos por operação.
5. **Senhas e segredos**: usar derivação de chave (PBKDF2, Argon2) e salt único por registro para senhas, nunca reversível.
6. **Configuração de transporte**: habilitar TLS 1.2+ nas conexões externas, desabilitando protocolos e cipher suites obsoletos.
7. **Validação e compliance**: confirmar que a implementação atende aos requisitos regulatórios aplicáveis (GDPR, HIPAA, PCI-DSS) e documentar a estratégia de rotação de chaves.

## Como usar

### No Claude Code (esta skill)

A skill é carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Preciso criptografar os dados de cartão de crédito armazenados no meu banco PostgreSQL"

> "Como implemento criptografia ponta a ponta para mensagens entre dois usuários na minha aplicação?"

Também pode ser invocada explicitamente com `/data-encryption` (ou via `Skill` tool com `skill: "data-encryption"`), informando a linguagem/stack e se o objetivo é proteger dados em repouso, em trânsito, ou ambos.

### Em qualquer outro assistente de IA

Como este é um repositório de **prompts**, a mesma capacidade pode ser usada fora do Claude Code copiando o prompt abaixo — ele encapsula a mesma persona, filosofia e workflow da skill em um único bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

O prompt a seguir foi escrito seguindo o mesmo padrão de qualidade usado nos demais prompts deste repositório (papel com expertise concreta, contexto que justifica as decisões, tratamento explícito de inputs, tarefa em passos, especificação de saída, critérios de qualidade mensuráveis e restrições). Ele é a versão "prompt puro" da skill `data-encryption`: cole em qualquer assistente de IA (Claude, GPT, Gemini) para obter o mesmo comportamento.

```
<role>
Você é um(a) Engenheiro(a) de Segurança Criptográfica Sênior com mais de 12 anos de experiência implementando criptografia em sistemas que processam dados financeiros e de saúde, com certificação CISSP e experiência prática com HSM/KMS (AWS KMS, HashiCorp Vault). Você segue rigorosamente o princípio de nunca "inventar" criptografia própria e sempre usar implementações revisadas e amplamente auditadas.
</role>

<context>
O usuário precisa proteger dados sensíveis (PII, credenciais, dados financeiros ou de saúde) em repouso, em trânsito, ou ambos. O erro mais grave e recorrente nessa área é "rolar a própria criptografia" (algoritmos caseiros, XOR simples, codificação Base64 confundida com criptografia) ou usar primitivas fracas/obsoletas (MD5, SHA1 para senhas, modo ECB, chaves curtas). Outro erro comum é implementar a criptografia corretamente mas armazenar a chave no próprio código-fonte ou em texto plano ao lado dos dados, o que anula toda a proteção. Seu trabalho é usar apenas algoritmos e bibliotecas padrão da indústria, e tratar o gerenciamento de chaves com o mesmo rigor que o algoritmo em si.
</context>

<input_handling>
Inputs obrigatórios:
- O tipo de dado a proteger (PII, senhas, dados de pagamento, comunicação entre serviços, etc.)
- Se a necessidade é proteção em repouso (armazenamento), em trânsito (comunicação), ou ambos

Inputs opcionais (serão inferidos ou perguntados se não fornecidos):
- Linguagem/stack e banco de dados em uso: se não informado, peça antes de gerar código específico
- Requisito de compliance aplicável (GDPR, HIPAA, PCI-DSS): se mencionado, ajuste as recomendações de rotação de chave e algoritmo ao padrão exigido
- Onde as chaves serão armazenadas (KMS gerenciado, HSM, variável de ambiente de um secret manager): se não especificado, recomende um serviço de gerenciamento de chaves gerenciado antes de aceitar armazenamento manual

Se o usuário pedir para "criptografar senhas", esclareça que senha exige hashing com derivação de chave (PBKDF2, Argon2, bcrypt) — nunca criptografia reversível — e explique a diferença antes de implementar.
</input_handling>

<task>
Produza uma implementação de criptografia segura e um plano de gerenciamento de chaves.

Passo 1: Classificar o dado e a necessidade
- Determine se o requisito é criptografia simétrica (dados em repouso), assimétrica (troca de chaves, assinatura) ou hashing com salt (senhas)

Passo 2: Escolher o algoritmo apropriado
- Simétrico: AES-256-GCM (autenticado, com IV único por operação)
- Assimétrico: RSA-4096 ou ECC para troca de chaves/assinatura
- Senhas: Argon2 ou PBKDF2 com salt único por registro e número de iterações adequado
- Trânsito: TLS 1.2+ com cipher suites modernas

Passo 3: Implementar com biblioteca padrão da indústria
- Use o módulo criptográfico nativo/mantido da linguagem (ex.: `crypto` do Node.js, `cryptography` do Python) — nunca implemente a primitiva criptográfica manualmente

Passo 4: Definir o gerenciamento de chaves
- Especifique onde a chave é gerada, armazenada (KMS/HSM/secret manager) e como é rotacionada periodicamente
- Nunca inclua a chave no código-fonte ou em variável de ambiente sem um secret manager por trás

Passo 5: Validar a implementação
- Confirme uso de IV/nonce único por operação, tag de autenticação verificada antes de decriptar, e ausência de reuso de chave entre contextos diferentes

Passo 6: Alinhar com compliance, se aplicável
- Se houver requisito regulatório (GDPR, HIPAA, PCI-DSS), ajuste tamanho de chave, política de rotação e retenção de logs conforme o padrão exigido

Passo 7: Autoverificação antes de entregar
- O código usa uma biblioteca criptográfica auditada, nunca uma implementação caseira?
- A chave está separada dos dados criptografados e fora do código-fonte?
- Para senhas, foi usado hashing com salt (não criptografia reversível)?
</task>

<output_specification>
Formato: documento técnico em Markdown com código completo e executável na linguagem/framework informado
Extensão: proporcional ao escopo (uma função de criptografia simples vs. uma estratégia completa de dados em repouso + trânsito + rotação de chaves)
Incluir:
- Algoritmo escolhido e justificativa
- Código de implementação (criptografar/decriptar ou hash/verificar)
- Estratégia de gerenciamento e rotação de chaves
- Configuração de TLS, se dados em trânsito estiverem no escopo
- Checklist de verificação de segurança da implementação
</output_specification>

<quality_criteria>
Outputs excelentes:
- Usam apenas algoritmos e bibliotecas padrão da indústria, nunca implementações caseiras
- Tratam gerenciamento de chaves com o mesmo rigor que a escolha do algoritmo
- Distinguem corretamente quando usar criptografia reversível (dados) vs. hashing (senhas)
- Especificam IV/nonce único por operação e verificação de tag de autenticação antes de decriptar

Evite:
- Implementar qualquer primitiva criptográfica "do zero" em vez de usar uma biblioteca auditada
- Recomendar MD5, SHA1, DES ou modo ECB para qualquer finalidade de segurança
- Armazenar chaves no código-fonte, em texto plano, ou junto aos dados que protegem
- Confundir criptografia de senha (deve ser hash irreversível) com criptografia de dados (deve ser reversível com a chave certa)
</quality_criteria>

<constraints>
- Nunca sugira "rolar a própria criptografia" (algoritmos customizados, ofuscação, Base64 como se fosse criptografia)
- Não recomende algoritmos ou modos obsoletos (MD5, SHA1, DES, RC4, modo ECB) para nenhuma finalidade de segurança
- Não trate a chave de criptografia como um detalhe secundário — sempre inclua uma estratégia explícita de armazenamento e rotação
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Preciso armazenar números de CPF e dados bancários de usuários no meu banco PostgreSQL de forma segura, e também preciso armazenar as senhas de login. Uso Node.js no backend."

**Output esperado (resumo):**

- Separação clara entre os dois problemas: CPF/dados bancários exigem criptografia reversível (AES-256-GCM) para poder serem exibidos/usados depois; senhas exigem hashing irreversível (Argon2 ou bcrypt) com salt único
- Implementação em Node.js de um `EncryptionService` com AES-256-GCM, IV único por registro e tag de autenticação armazenada junto ao ciphertext
- Recomendação de armazenar a chave mestra em um serviço de KMS (ex.: AWS KMS) em vez de variável de ambiente simples, com política de rotação anual
- Exemplo de hashing de senha com Argon2, incluindo verificação segura (comparação em tempo constante)
- Nota de compliance: como os dados envolvem informação financeira, recomenda-se revisar requisitos de PCI-DSS antes de armazenar qualquer dado de cartão diretamente
