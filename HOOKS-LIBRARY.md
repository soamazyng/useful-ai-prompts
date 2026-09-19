# Biblioteca de Hooks do Claude Code

## Visão Geral

Uma coleção abrangente de hooks prontos para produção para o Claude Code. Hooks permitem validação automatizada, testes, formatação e configuração de ambiente em pontos-chave do seu fluxo de trabalho de desenvolvimento.

**Total de Hooks**: 6 exemplos abrangentes cobrindo todos os principais casos de uso

## O que São Hooks?

Hooks são scripts automatizados que executam em eventos específicos no fluxo de trabalho do Claude Code. Eles permitem:

- **Controle de Qualidade Automatizado**: Executar linters, testes e varreduras de segurança automaticamente
- **Configuração de Ambiente**: Inicializar ambiente de desenvolvimento no início da sessão
- **Formatação de Código**: Autoformatar código após edições
- **Segurança**: Prevenir commits com segredos expostos ou dependências vulneráveis
- **Detecção de Breaking Changes**: Alertar antes de commitar mudanças que quebram APIs

## Início Rápido

### 1. Copie os Hooks para Seu Projeto

```bash
# Copiar todos os hooks
cp -r hooks/ /caminho/para/seu/projeto/.claude/hooks/

# Ou copiar hooks individuais
cp -r hooks/security-scan /caminho/para/seu/projeto/.claude/hooks/
```

### 2. Configure nas Definições

Adicione ao `.claude/settings.json`:

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "$CLAUDE_PROJECT_DIR/.claude/hooks/security-scan/hook.sh",
            "timeout": 30
          }
        ]
      }
    ]
  }
}
```

### 3. Torne os Scripts Executáveis

```bash
chmod +x .claude/hooks/*/hook.sh
```

## Hooks Disponíveis

### 🔒 security-scan

**Evento:** PreToolUse (Bash) | **Finalidade:** Prevenir vazamento de segredos

Escaneia segredos expostos antes de commits:

- Chaves AWS, tokens de API, chaves privadas
- Credenciais de banco de dados e senhas
- 15+ detecções de padrões de segredos
- Suporte a whitelist via `.secretsignore`

```json
{
  "PreToolUse": [
    {
      "matcher": "Bash",
      "hooks": [
        {
          "type": "command",
          "command": "$CLAUDE_PROJECT_DIR/.claude/hooks/security-scan/hook.sh"
        }
      ]
    }
  ]
}
```

**Funcionalidades Principais:**

- ✅ Bloqueia commits com segredos expostos
- ✅ Orientação clara de remediação
- ✅ Níveis de sensibilidade configuráveis
- ✅ Gerenciamento de falsos positivos

---

### ✅ test-runner

**Evento:** PreToolUse (Bash) | **Finalidade:** Executar testes antes de commits

Executa testes automaticamente com detecção inteligente de framework:

- Jest, pytest, RSpec, Go test, Cargo test
- Executa testes apenas para arquivos alterados
- Suporte a execução paralela
- Cache de resultados de testes

```json
{
  "PreToolUse": [
    {
      "matcher": "Bash",
      "hooks": [
        {
          "type": "command",
          "command": "$CLAUDE_PROJECT_DIR/.claude/hooks/test-runner/hook.sh",
          "timeout": 300
        }
      ]
    }
  ]
}
```

**Funcionalidades Principais:**

- ✅ Suporte a múltiplos frameworks
- ✅ Testes incrementais rápidos
- ✅ Relatórios detalhados de falhas
- ✅ Rastreamento de cobertura

---

### 🚀 session-setup

**Evento:** SessionStart | **Finalidade:** Inicializar ambiente de desenvolvimento

Prepara seu ambiente no início da sessão:

- Carrega variáveis `.env`
- Verifica versões de dependências
- Verifica conexões de banco de dados
- Exibe status do git e lembretes

```json
{
  "SessionStart": [
    {
      "matcher": "*",
      "hooks": [
        {
          "type": "command",
          "command": "$CLAUDE_PROJECT_DIR/.claude/hooks/session-setup/hook.sh"
        }
      ]
    }
  ]
}
```

**Funcionalidades Principais:**

- ✅ Validação de ambiente
- ✅ Verificação de dependências
- ✅ Painel de status
- ✅ Lembretes do projeto

---

### 🎨 auto-format

**Evento:** PostToolUse (Edit|Write) | **Finalidade:** Autoformatar código

Formata código automaticamente após edições:

- Prettier (JS/TS), Black (Python), RuboCop (Ruby)
- gofmt (Go), rustfmt (Rust), Spotless (Java)
- Mostra diff de formatação
- Auto-commit opcional

```json
{
  "PostToolUse": [
    {
      "matcher": "Edit|Write",
      "hooks": [
        {
          "type": "command",
          "command": "$CLAUDE_PROJECT_DIR/.claude/hooks/auto-format/hook.sh"
        }
      ]
    }
  ]
}
```

**Funcionalidades Principais:**

- ✅ 8+ formatadores de linguagem
- ✅ Visualização de diff
- ✅ Respeita configurações do projeto
- ✅ Comportamento configurável

---

### ⚠️ breaking-change-detection

**Evento:** PreToolUse (Bash) | **Finalidade:** Detectar breaking changes em APIs

Alerta antes de commitar mudanças que quebram APIs:

- Compara assinaturas de API
- Detecta exports removidos
- Identifica mudanças de parâmetros
- Integração com versionamento semântico

```json
{
  "PreToolUse": [
    {
      "matcher": "Bash",
      "hooks": [
        {
          "type": "command",
          "command": "$CLAUDE_PROJECT_DIR/.claude/hooks/breaking-change-detection/hook.sh"
        }
      ]
    }
  ]
}
```

**Funcionalidades Principais:**

- ✅ Suporte a múltiplas linguagens
- ✅ Comparação de assinaturas
- ✅ Relatórios claros de mudanças
- ✅ Recomendações de SemVer

---

### 🛡️ dependency-check

**Evento:** PreToolUse (Bash) | **Finalidade:** Verificar dependências vulneráveis

Escaneia dependências em busca de vulnerabilidades de segurança:

- npm audit, pip-audit, bundle audit
- Bloqueio baseado em severidade
- Sugestões de autocorreção
- Geração de relatório de segurança

```json
{
  "PreToolUse": [
    {
      "matcher": "Bash",
      "hooks": [
        {
          "type": "command",
          "command": "$CLAUDE_PROJECT_DIR/.claude/hooks/dependency-check/hook.sh"
        }
      ]
    }
  ]
}
```

**Funcionalidades Principais:**

- ✅ 5+ gerenciadores de pacotes
- ✅ Severidade configurável
- ✅ Sugestões de atualização
- ✅ Whitelist de vulnerabilidades

---

## Referência de Eventos de Hooks

| Evento               | Gatilho                  | Usos Comuns                                       |
| -------------------- | ------------------------ | ------------------------------------------------- |
| **PreToolUse**       | Antes da execução da ferramenta | Validação, linting, varreduras de segurança |
| **PostToolUse**      | Após conclusão da ferramenta   | Autoformatação, notificações               |
| **UserPromptSubmit** | Usuário envia prompt     | Filtragem de conteúdo, aplicação de políticas     |
| **SessionStart**     | Sessão inicia            | Configuração de ambiente, exibição de status      |
| **SessionEnd**       | Sessão encerra           | Limpeza, geração de relatórios                    |
| **Stop**             | Agente termina           | Verificações de qualidade, resumos                |
| **Notification**     | Solicitação de permissão | Regras de aprovação automática                    |

## Configurações Recomendadas

### Configuração Mínima (Segurança + Testes)

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "$CLAUDE_PROJECT_DIR/.claude/hooks/security-scan/hook.sh"
          },
          {
            "type": "command",
            "command": "$CLAUDE_PROJECT_DIR/.claude/hooks/test-runner/hook.sh",
            "timeout": 300
          }
        ]
      }
    ],
    "SessionStart": [
      {
        "matcher": "*",
        "hooks": [
          {
            "type": "command",
            "command": "$CLAUDE_PROJECT_DIR/.claude/hooks/session-setup/hook.sh"
          }
        ]
      }
    ]
  }
}
```

### Desenvolvimento Full Stack

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "$CLAUDE_PROJECT_DIR/.claude/hooks/security-scan/hook.sh"
          },
          {
            "type": "command",
            "command": "$CLAUDE_PROJECT_DIR/.claude/hooks/test-runner/hook.sh",
            "timeout": 300
          },
          {
            "type": "command",
            "command": "$CLAUDE_PROJECT_DIR/.claude/hooks/dependency-check/hook.sh"
          },
          {
            "type": "command",
            "command": "$CLAUDE_PROJECT_DIR/.claude/hooks/breaking-change-detection/hook.sh"
          }
        ]
      }
    ],
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "$CLAUDE_PROJECT_DIR/.claude/hooks/auto-format/hook.sh"
          }
        ]
      }
    ],
    "SessionStart": [
      {
        "matcher": "*",
        "hooks": [
          {
            "type": "command",
            "command": "$CLAUDE_PROJECT_DIR/.claude/hooks/session-setup/hook.sh"
          }
        ]
      }
    ]
  }
}
```

### Desenvolvimento de API/Biblioteca

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "$CLAUDE_PROJECT_DIR/.claude/hooks/breaking-change-detection/hook.sh"
          },
          {
            "type": "command",
            "command": "$CLAUDE_PROJECT_DIR/.claude/hooks/test-runner/hook.sh",
            "timeout": 300
          }
        ]
      }
    ],
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "$CLAUDE_PROJECT_DIR/.claude/hooks/auto-format/hook.sh"
          }
        ]
      }
    ]
  }
}
```

## Boas Práticas de Desenvolvimento de Hooks

### Validação de Entrada

```bash
#!/bin/bash
set -euo pipefail

# Sempre analisar e validar a entrada
INPUT=$(cat)
TOOL_INPUT=$(echo "$INPUT" | jq -r '.tool_input.command // empty')

# Validar antes de usar
if [ -z "$TOOL_INPUT" ]; then
    echo '{"continue": true, "suppressOutput": true}'
    exit 0
fi
```

### Uso Correto de Aspas

```bash
# ✅ Bom - variáveis entre aspas
if [[ "$TOOL_INPUT" =~ git[[:space:]]+commit ]]; then
    FILES=$(git diff --cached --name-only --diff-filter=ACM)
    for FILE in "$FILES"; do
        check_file "$FILE"
    done
fi

# ❌ Ruim - variáveis sem aspas
if [[ $TOOL_INPUT =~ git commit ]]; then
    for FILE in $FILES; do
        check_file $FILE
    done
fi
```

### Tratamento de Erros

```bash
# Código de saída 0: Sucesso
echo '{"permissionDecision": "allow"}'
exit 0

# Código de saída 2: Erro bloqueante (alimentado ao Claude)
echo '{"permissionDecision": "deny", "reason": "Tests failed"}'
exit 2

# Outros códigos: Erro não bloqueante (mostrado ao usuário)
echo "Aviso: Linter não encontrado" >&2
exit 1
```

### Timeouts

```json
{
  "hooks": [
    {
      "type": "command",
      "command": "./hook.sh",
      "timeout": 300 // 5 minutos para operações lentas
    }
  ]
}
```

## Considerações de Segurança

⚠️ **Importante**: Hooks executam comandos shell arbitrários com as permissões do seu usuário.

**Boas Práticas:**

1. ✅ Validar toda entrada do JSON do hook
2. ✅ Usar caminhos absolutos ou `$CLAUDE_PROJECT_DIR`
3. ✅ Colocar todas as variáveis entre aspas
4. ✅ Verificar path traversal (`..` em caminhos)
5. ✅ Revisar scripts de hook antes de usar
6. ✅ Usar permissões mínimas
7. ✅ Testar em ambiente seguro primeiro

## Depuração

### Verificar Registro de Hooks

```bash
claude
> /hooks
```

### Modo de Depuração

```bash
claude --debug
```

### Testar Hook Manualmente

```bash
# Simular entrada do hook
echo '{"tool_name":"Bash","tool_input":{"command":"git commit -m test"}}' | \
  .claude/hooks/security-scan/hook.sh
```

### Problemas Comuns

**Hook não ativando:**

- Verifique se o matcher é case-sensitive e está correto
- Verifique se o script tem permissões de execução (`chmod +x`)
- Certifique-se de que a sintaxe JSON é válida

**Erros de permissão:**

- Torne scripts executáveis: `chmod +x .claude/hooks/*/hook.sh`
- Verifique se os caminhos de arquivo estão corretos

**Erros de timeout:**

- Aumente o valor do timeout para operações lentas
- Otimize o desempenho do script do hook

## Contribuindo

Para criar hooks customizados:

1. Crie o diretório do hook: `.claude/hooks/nome-do-seu-hook/`
2. Adicione `README.md` com documentação
3. Crie `hook.sh` com o script
4. Torne executável: `chmod +x hook.sh`
5. Adicione exemplo de configuração
6. Teste minuciosamente

## Recursos

- [Documentação Oficial de Hooks do Claude Code](https://code.claude.com/docs/en/hooks)
- [Repositório de Exemplos de Hooks](https://github.com/aj-geddes/useful-ai-prompts)
- [Guia de Configuração de Settings](https://code.claude.com/docs/en/settings)

---

**Total de Hooks**: 6 exemplos prontos para produção
**Total de Linhas**: 6.135+ linhas de código e documentação
**Linguagens Suportadas**: JavaScript, TypeScript, Python, Ruby, Go, Rust, PHP, Java, C/C++
**Gerenciadores de Pacotes**: npm, pip, bundler, cargo, go modules
