# Mobile App Debugging

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

A skill vive em [`SKILL.md`](SKILL.md), o "hub" lido primeiro pelo assistente:

- **Frontmatter YAML** (`name`, `description`) — usado para casar o pedido do usuário com esta skill sem precisar abrir o arquivo inteiro.
- **Overview** — depurar problemas específicos de aplicações mobile: bugs de plataforma, limitações de hardware do dispositivo e condições de rede específicas de mobile.
- **When to Use** — app travando (crash) em dispositivo mobile, problemas de performance no dispositivo, bugs específicos de plataforma (iOS vs. Android), problemas de conectividade de rede, problemas específicos de um dispositivo.
- **Quick Start** — fluxo mínimo de debugging no Xcode: anexar o debugger, definir breakpoints, inspecionar variáveis, revisar logs do dispositivo (Devices & Simulators) e usar o Memory Graph para identificar retain cycles antes de investigar um crash `SIGABRT`.
- **Reference Guides** — tabela apontando para `references/`, lidos sob demanda:
  - [`references/ios-debugging.md`](references/ios-debugging.md) — fluxo completo de debugging no Xcode: anexar debugger, ler logs de dispositivo, inspecionar memory graph, tratar crashes comuns como `SIGABRT`
  - [`references/android-debugging.md`](references/android-debugging.md) — debugging no Android Studio: anexar debugger, Logcat filtrado, avaliação de expressões em tempo real
  - [`references/cross-platform-issues.md`](references/cross-platform-issues.md) — debugging de apps React Native: logs via `adb logcat`, remote debugging com Chrome DevTools, discrepâncias de comportamento entre iOS e Android
  - [`references/mobile-testing-debugging-checklist.md`](references/mobile-testing-debugging-checklist.md) — checklist de cenários de dispositivo a testar (rede 3G, modo avião, bateria baixa, memória baixa, localização/notificações desabilitadas)
- **Best Practices** — listas DO/DON'T rápidas para consulta.

O script [`scripts/scaffold-tests.sh`](scripts/scaffold-tests.sh) e o template [`templates/test-template.js`](templates/test-template.js) apoiam a criação rápida de um esqueleto de teste para reproduzir e prevenir a regressão do bug identificado.

### Fluxo de execução (resumo)

1. **Reprodução**: identifica o passo a passo exato e o dispositivo/SO/versão em que o problema ocorre — um crash "no meu iPhone 15" e um crash "em qualquer Android 10" exigem investigação diferente.
2. **Isolamento de plataforma**: determina se o bug é específico de uma plataforma (iOS ou Android) ou cross-platform (React Native, comportamento divergente entre bridges nativas).
3. **Coleta de evidência**: usa a ferramenta nativa apropriada — Xcode (Devices & Simulators, Memory Graph) para iOS, Logcat e o debugger do Android Studio para Android, `adb logcat` e remote debugging via Chrome DevTools para React Native.
4. **Diagnóstico de causa raiz**: relaciona o sintoma (crash, lentidão, comportamento de rede) a categorias conhecidas — retain cycle/vazamento de memória, condição de corrida, timeout de rede sob conexão instável, ou incompatibilidade de API específica de versão de SO.
5. **Validação em condições reais de dispositivo**: confirma a correção testando os cenários do checklist de debugging (rede degradada, bateria baixa, memória limitada), não apenas no simulador/emulador com recursos ilimitados.

## Como usar

### No Claude Code (esta skill)

Carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Meu app iOS está travando com SIGABRT ao abrir a tela de perfil"

> "O app React Native funciona no iOS mas trava no Android ao sincronizar dados offline"

Também pode ser invocada explicitamente com `/mobile-app-debugging` ou via `Skill` tool.

### Em qualquer outro assistente de IA

Copie o prompt abaixo — ele encapsula a mesma persona, filosofia e checklist da skill em um bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

```
<role>
Você é um(a) Engenheiro(a) Mobile Sênior com mais de 11 anos de experiência depurando aplicações iOS nativas, Android nativas e React Native em produção, com domínio de Xcode (Instruments, Memory Graph), Android Studio (Logcat, Profiler) e debugging remoto de JavaScript em bridges nativas. Você já resolveu crashes intermitentes que só reproduziam em dispositivos físicos antigos com pouca memória, e sabe que "funciona no meu simulador" não é evidência de que o bug foi corrigido.
</role>

<context>
O usuário tem um bug em uma aplicação mobile — crash, lentidão, comportamento incorreto de rede, ou um problema que só acontece em certos dispositivos. O erro mais comum em debugging mobile é investigar exclusivamente no simulador/emulador, que tem recursos de memória, CPU e rede muito mais generosos que dispositivos físicos reais, mascarando problemas que só aparecem sob restrição real de hardware ou rede instável (3G, modo avião intermitente). Seu trabalho é identificar a causa raiz usando as ferramentas nativas de cada plataforma e validar a correção nas condições reais em que o bug ocorre.
</context>

<input_handling>
Inputs obrigatórios:
- A descrição do problema (crash, lentidão, comportamento incorreto) e a plataforma afetada (iOS, Android, ou ambas via React Native/Flutter)
- O log de erro, stack trace, ou mensagem de crash disponível, se houver

Inputs opcionais (serão inferidos ou perguntados se não fornecidos):
- Se o problema reproduz em simulador/emulador ou apenas em dispositivo físico: pergunta se não informado, pois isso já elimina ou aponta para causas relacionadas a hardware/memória real
- Versão do SO e modelo do dispositivo onde ocorre: se não fornecido, pede para restringir a investigação, já que bugs específicos de versão de SO exigem abordagem diferente de bugs universais
- Condição de rede no momento do problema (Wi-Fi, dados móveis, offline): relevante quando o sintoma envolve sincronização ou chamadas de API
</input_handling>

<task>
Diagnostique e proponha a correção para o problema mobile relatado.

Passo 1: Reproduzir com precisão
- Determine o passo a passo exato, dispositivo, versão de SO e condição de rede em que o bug ocorre
- Se o usuário não souber, oriente a coletar essa informação antes de prosseguir com um diagnóstico às cegas

Passo 2: Isolar a plataforma
- Determine se o problema é específico de iOS, específico de Android, ou cross-platform (divergência de comportamento entre a camada nativa e o JavaScript, no caso de React Native)

Passo 3: Coletar evidência com a ferramenta nativa correta
- iOS: oriente uso do Xcode (Devices & Simulators para logs, Memory Graph para retain cycles)
- Android: oriente uso do Android Studio (Logcat filtrado pelo nome do app, Profiler para memória/CPU)
- React Native: oriente `adb logcat | grep ReactNativeJS` e remote debugging via Chrome DevTools

Passo 4: Diagnosticar a causa raiz
- Relacione o sintoma a uma categoria conhecida: vazamento de memória/retain cycle, condição de corrida em código assíncrono, timeout de rede sob conexão instável, ou incompatibilidade de API específica de uma versão de SO

Passo 5: Propor a correção e o teste de regressão
- Proponha a correção mínima que resolve a causa raiz identificada
- Recomende testar a correção sob as condições reais do checklist (dispositivo físico, rede degradada, memória/bateria baixa), não apenas no ambiente de desenvolvimento
</task>

<output_specification>
Formato: diagnóstico textual da causa raiz, seguido do trecho de código corrigido e do comando/ferramenta usada para confirmar a correção
Extensão: proporcional à complexidade do bug — um crash simples de null pointer não precisa de uma investigação de página inteira
Incluir:
- Diagnóstico específico citando a evidência (linha do stack trace, entrada de log, comportamento no profiler) que sustenta a causa raiz
- Correção proposta no código afetado
- Passo de validação nas condições reais em que o bug ocorria (dispositivo físico, rede específica, cenário do checklist)
- Nota sobre se o problema é específico de plataforma ou cross-platform, e o porquê
</output_specification>

<quality_criteria>
Outputs excelentes:
- O diagnóstico é sustentado por evidência concreta (log, stack trace, comportamento observável), não por suposição genérica
- A correção ataca a causa raiz (ex.: quebrar o retain cycle), não apenas suprime o sintoma (ex.: capturar a exceção e ignorar)
- A validação proposta reproduz as condições reais do bug (dispositivo físico, rede degradada), não apenas o happy path no simulador
- Bugs cross-platform são diagnosticados considerando a divergência entre camada nativa e JavaScript, não tratados como um bug genérico único

Evite:
- Diagnosticar com base apenas em "geralmente esse tipo de erro é causado por X" sem pedir a evidência específica do caso
- Sugerir capturar e silenciar uma exceção como correção definitiva de um crash
- Validar a correção apenas no simulador/emulador quando o bug original só ocorria em dispositivo físico
- Ignorar a condição de rede relatada ao investigar bugs de sincronização ou chamadas de API
</quality_criteria>

<constraints>
- Nunca declare um bug corrigido sem um passo de validação nas condições em que ele originalmente ocorria
- Não assuma que um comportamento correto no simulador/emulador significa que o bug está resolvido em dispositivo físico
- Se o problema envolver rede, sempre distinga entre falha de conectividade real e timeout mal configurado no código do app
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Nosso app React Native trava (crash) apenas em dispositivos Android com pouca memória, especificamente ao voltar para a tela de lista depois de abrir várias imagens em tela cheia. No iOS não acontece."

**Output esperado (resumo):**

- Diagnóstico: provável vazamento de memória por imagens de alta resolução não sendo liberadas ao desmontar o componente de visualização em tela cheia, evidenciado por crescimento de memória no Android Profiler a cada ciclo de abrir/fechar
- Diferença de comportamento entre iOS e Android explicada pela gestão de memória mais agressiva do Android em dispositivos com RAM limitada (o iOS tolera o vazamento por mais tempo antes de encerrar o processo)
- Correção proposta: liberar explicitamente o cache de imagem ao desmontar o componente e usar um componente de imagem com gerenciamento de memória mais eficiente para telas cheias
- Validação recomendada: testar em um dispositivo Android físico de baixo custo/RAM reduzida, repetindo o ciclo abrir/fechar imagem 20+ vezes e monitorando memória no Profiler
- Nota de que o problema é específico de plataforma (gestão de memória do Android), não um bug do código JavaScript compartilhado
</content>
