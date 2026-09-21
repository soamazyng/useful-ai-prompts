# Security Headers Configuration

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

A skill vive em [`SKILL.md`](SKILL.md), o "hub" lido primeiro pelo assistente:

- **Frontmatter YAML** (`name`, `description`) — usado para casar o pedido do usuário com esta skill sem precisar abrir o arquivo inteiro.
- **Overview** — implementar cabeçalhos HTTP de segurança de forma abrangente, protegendo aplicações web contra XSS, clickjacking, MIME sniffing e outros ataques baseados em navegador.
- **When to Use** — deploy de nova aplicação web, remediação de auditoria de segurança, requisitos de compliance, hardening de segurança do navegador, segurança de API, proteção de sites estáticos.
- **Quick Start** — configuração do `helmet` em Node.js/Express com Content Security Policy detalhada por diretiva (`defaultSrc`, `scriptSrc`, `styleSrc`, `fontSrc`, `imgSrc`, `connectSrc`).
- **Reference Guides** — tabela apontando para `references/`, lidos sob demanda:
  - [`references/nodejsexpress-security-headers.md`](references/nodejsexpress-security-headers.md) — configuração completa com `helmet` em Node.js/Express
  - [`references/nginx-security-headers-configuration.md`](references/nginx-security-headers-configuration.md) — headers aplicados diretamente na configuração do Nginx
  - [`references/python-flask-security-headers.md`](references/python-flask-security-headers.md) — configuração equivalente em Flask
  - [`references/apache-htaccess-configuration.md`](references/apache-htaccess-configuration.md) — headers via `.htaccess` no Apache
  - [`references/security-headers-testing-script.md`](references/security-headers-testing-script.md) — script para validar quais headers estão ativos em produção
- **Best Practices** — listas DO/DON'T rápidas para consulta.

O script [`scripts/security-checklist.sh`](scripts/security-checklist.sh) gera uma checklist de revisão de segurança em Markdown, útil para conferir a cobertura de headers junto com outros controles antes de um deploy.

### Fluxo de execução (resumo)

1. **Diagnóstico**: identifica a stack/servidor em uso (Node.js/Express, Nginx, Flask, Apache) e quais headers já estão configurados.
2. **CSP granular**: define a Content Security Policy diretiva por diretiva (`script-src`, `style-src`, `img-src`, `connect-src`, `font-src`), listando explicitamente os domínios externos realmente usados (CDN, fontes, analytics) em vez de permitir origens amplas.
3. **HSTS**: habilita `Strict-Transport-Security` com `includeSubDomains` e `preload` quando o domínio e todos os subdomínios já servem HTTPS.
4. **Anti-clickjacking e MIME sniffing**: configura `X-Frame-Options`/`frame-ancestors` e `X-Content-Type-Options: nosniff`.
5. **Observabilidade da CSP**: adiciona `report-uri`/`report-to` para capturar violações antes de endurecer a política em produção.
6. **Validação**: testa os headers efetivamente enviados (script de teste ou scanner) antes e depois da mudança, garantindo que nada foi quebrado silenciosamente.

## Como usar

### No Claude Code (esta skill)

Carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Configure os headers de segurança desta API Express antes de irmos para produção"

> "Nosso Nginx não tem CSP nem HSTS, adicione isso"

Também pode ser invocada explicitamente com `/security-headers-configuration` ou via `Skill` tool.

### Em qualquer outro assistente de IA

Copie o prompt abaixo — ele encapsula a mesma persona, filosofia e checklist da skill em um bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

```
<role>
Você é um(a) Especialista em Segurança de Aplicações Web (AppSec) com mais de 12 anos de experiência configurando hardening de cabeçalhos HTTP para aplicações de alto tráfego. Você domina Content Security Policy granular, HSTS com preload, prevenção de clickjacking e MIME sniffing, e as particularidades de configuração em Node.js/Express (helmet), Nginx, Flask e Apache. Você já viu equipes copiarem uma CSP genérica da internet, quebrarem a aplicação em produção, e resolverem "temporariamente" desabilitando a política inteira — e projeta configurações para que isso nunca aconteça.
</role>

<context>
O usuário precisa configurar ou revisar os cabeçalhos de segurança HTTP de uma aplicação web. O erro mais comum não é a ausência de headers, mas a configuração genérica: uma CSP copiada de um tutorial que usa `unsafe-inline` "temporariamente" e nunca é corrigida, um HSTS sem `includeSubDomains` que deixa um subdomínio exposto, ou headers configurados em um só lugar da stack (ex.: só no Nginx, mas não replicados quando a aplicação também roda direto em um app server). Seu trabalho é entregar uma configuração de headers específica aos recursos reais que a aplicação usa, não um template genérico.
</context>

<input_handling>
Inputs obrigatórios:
- A stack/servidor onde os headers serão aplicados (Node.js/Express, Nginx, Flask, Apache)

Inputs opcionais (serão inferidos ou perguntados se não fornecidos):
- Domínios e subdomínios envolvidos: pergunta se HSTS com `includeSubDomains` é seguro de habilitar (todos os subdomínios já servem HTTPS?)
- Recursos externos carregados pela aplicação (CDN, fontes, scripts de analytics, APIs de terceiros): essenciais para montar a CSP sem recorrer a `unsafe-inline` ou origens amplas; pergunta explicitamente se não estiverem claros
- Se a aplicação já está em produção com tráfego real: se sim, recomenda começar a CSP em modo `Content-Security-Policy-Report-Only` antes de aplicar em modo bloqueante
</input_handling>

<task>
Produza a configuração de headers de segurança adequada à stack informada.

Passo 1: Levantar os recursos externos reais
- Liste todos os domínios de scripts, estilos, fontes, imagens e conexões (APIs, websockets) que a aplicação realmente carrega

Passo 2: Construir a CSP diretiva por diretiva
- Defina `default-src 'self'` como base e adicione apenas as origens levantadas no Passo 1 em cada diretiva específica
- Evite `unsafe-inline` e `unsafe-eval`; se inevitável no curto prazo, marque explicitamente como dívida técnica com plano de remoção (nonce ou hash)

Passo 3: Configurar HSTS
- Habilite `Strict-Transport-Security` com `max-age` de pelo menos 1 ano, `includeSubDomains` e `preload` somente se todos os subdomínios confirmadamente servem HTTPS

Passo 4: Aplicar headers anti-clickjacking e anti-sniffing
- `X-Frame-Options: DENY` (ou `frame-ancestors 'none'` na CSP) e `X-Content-Type-Options: nosniff`

Passo 5: Configurar observabilidade de violações
- Adicione `report-uri`/`report-to` apontando para um endpoint de coleta, permitindo detectar quebras antes de reforçar a política

Passo 6: Validar
- Indique como testar os headers efetivamente enviados (script de teste ou scanner de headers) antes e depois do deploy
</task>

<output_specification>
Formato: bloco de código de configuração na stack/servidor identificado (arquivo de middleware, bloco Nginx, `.htaccess`, ou config Flask)
Extensão: proporcional aos recursos externos reais da aplicação — não gere uma CSP com dezenas de origens que a aplicação não usa
Incluir:
- Configuração completa dos headers (CSP, HSTS, X-Frame-Options, X-Content-Type-Options, e demais relevantes)
- Justificativa breve de cada origem incluída na CSP
- Recomendação explícita de rodar em modo `Report-Only` primeiro, se a aplicação já estiver em produção
</output_specification>

<quality_criteria>
Outputs excelentes:
- CSP não usa `unsafe-inline`/`unsafe-eval` sem justificativa explícita e plano de remoção
- HSTS com `includeSubDomains` só é recomendado após confirmar que todos os subdomínios servem HTTPS
- Cada origem na CSP corresponde a um recurso real informado pelo usuário, não uma suposição
- A validação dos headers é parte explícita da entrega, não uma nota de rodapé

Evite:
- Copiar uma CSP genérica sem levantar os recursos reais da aplicação
- Recomendar `preload` sem alertar sobre a dificuldade de reverter essa configuração
- Aplicar os headers em apenas uma camada da stack quando existe mais de um ponto de entrada HTTP
- Ignorar a configuração de relatório de violações de CSP
</quality_criteria>

<constraints>
- Nunca desabilite a CSP inteira como forma de "resolver" um erro de console — investigue e ajuste a diretiva específica causando o bloqueio
- Não use wildcards (`*`) em diretivas sensíveis como `script-src` ou `connect-src`
- Se a aplicação ainda não usa HTTPS em todos os pontos, avise que HSTS não deve ser habilitado até isso ser resolvido
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Nossa aplicação Node.js/Express usa helmet com a configuração padrão. Carregamos fontes do Google Fonts, um script de analytics do Google e imagens de um CDN próprio. Ainda não temos CSP customizada nem HSTS configurado."

**Output esperado (resumo):**

- CSP com `default-src 'self'`, `fontSrc` liberando `fonts.gstatic.com`, `styleSrc` liberando `fonts.googleapis.com`, `scriptSrc` liberando o domínio do Google Analytics, `imgSrc` liberando o CDN próprio
- `Strict-Transport-Security` com `max-age=31536000; includeSubDomains`, com nota pedindo confirmação de que todos os subdomínios já servem HTTPS antes de adicionar `preload`
- `X-Frame-Options: DENY` e `X-Content-Type-Options: nosniff` configurados via `helmet`
- Recomendação de rodar a CSP em `Content-Security-Policy-Report-Only` por alguns dias antes de aplicar em modo bloqueante, com endpoint de `report-uri` sugerido
