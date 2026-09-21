# Image Optimization

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

A skill vive em [`SKILL.md`](SKILL.md), o "hub" que o assistente lê primeiro. Ele é composto por:

- **Frontmatter YAML** (`name`, `description`) — usado pelo Claude para decidir, sem abrir o arquivo inteiro, se a skill é relevante: otimizar imagens para web para reduzir o tamanho de arquivo sem sacrificar qualidade, usando compressão, formatos modernos e técnicas responsivas para carregamento mais rápido.
- **Overview** — o contexto que justifica a skill: imagens tipicamente compõem 50% do peso de uma página, então a otimização melhora dramaticamente a performance, especialmente em redes móveis.
- **When to Use** — gatilhos: otimização de site, implementação de imagens responsivas, melhoria de performance, aprimoramento de experiência mobile, antes de um deploy.
- **Quick Start** — uma tabela de referência de seleção de formato (JPEG, PNG, WebP, SVG) com o melhor uso de cada um, tipo de compressão, redução de tamanho esperada, ferramentas recomendadas (ImageMagick, PNGQuant, OptiPNG, cwebp) e comandos prontos de exemplo.
- **Reference Guides** — tabela apontando para os arquivos de aprofundamento em `references/`, carregados sob demanda (progressive disclosure):
  - [`references/image-compression-formats.md`](references/image-compression-formats.md) — compressão de imagem e formatos.
  - [`references/responsive-images.md`](references/responsive-images.md) — imagens responsivas (`srcset`, `<picture>`, tamanhos por viewport).
  - [`references/optimization-process.md`](references/optimization-process.md) — processo de otimização passo a passo.
  - [`references/monitoring-best-practices.md`](references/monitoring-best-practices.md) — monitoramento e boas práticas contínuas.
- **Best Practices** — listas DO/DON'T genéricas: seguir padrões e convenções estabelecidos, escrever código limpo e manutenível, documentar adequadamente, testar antes de implantar; e nunca pular testes/validação, ignorar tratamento de erro, ou fixar valores de configuração no código.

A skill inclui ainda um template de componente em [`templates/component-template.tsx`](templates/component-template.tsx) (componente de imagem responsiva pronto para adaptar).

### Fluxo de execução (resumo)

1. **Auditoria**: identificar as imagens mais pesadas da página/aplicação e seu impacto no peso total (imagens costumam ser ~50% do peso da página).
2. **Seleção de formato**: escolher o formato certo por tipo de conteúdo — JPEG para fotografias, PNG para ícones/transparência, WebP como formato moderno preferencial (com fallback), SVG para ícones e logos vetoriais.
3. **Compressão**: aplicar compressão lossy (JPEG/WebP, qualidade 70-85) ou lossless (PNG via OptiPNG/PNGQuant) conforme o formato escolhido.
4. **Responsividade**: implementar `srcset`/`<picture>` com tamanhos apropriados por breakpoint, evitando servir uma imagem grande demais para uma tela pequena.
5. **Automação**: integrar a otimização ao pipeline de build/deploy, evitando depender de otimização manual recorrente.
6. **Monitoramento**: acompanhar métricas de performance (peso de página, LCP) após o deploy para confirmar o ganho e detectar regressões futuras.

## Como usar

### No Claude Code (esta skill)

A skill é carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Preciso otimizar as imagens de produto do meu e-commerce, que estão pesando muito e afetando o carregamento mobile"

> "Implemente imagens responsivas com srcset e fallback WebP para o hero da landing page"

Também pode ser invocada explicitamente com `/image-optimization` (ou via `Skill` tool com `skill: "image-optimization"`), descrevendo o tipo de imagem e o contexto de uso (hero, thumbnail, galeria, etc.).

### Em qualquer outro assistente de IA

Como este é um repositório de **prompts**, a mesma capacidade pode ser usada fora do Claude Code copiando o prompt abaixo — ele encapsula a mesma persona e workflow da skill em um único bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

O prompt a seguir foi escrito seguindo o mesmo padrão de qualidade usado nos demais prompts deste repositório (papel com expertise concreta, contexto que justifica as decisões, tratamento explícito de inputs, tarefa em passos, especificação de saída, critérios de qualidade mensuráveis e restrições). Cole em qualquer assistente de IA (Claude, GPT, Gemini) para obter o mesmo comportamento da skill `image-optimization`.

```
<role>
Você é um(a) Engenheiro(a) de Performance Web Sênior com mais de 10 anos de experiência otimizando Core Web Vitals (LCP, CLS) para sites de alto tráfego em e-commerce e mídia. Você domina compressão de imagem (JPEG, PNG, WebP, AVIF), técnicas de imagem responsiva (`srcset`, `<picture>`, `sizes`) e ferramentas de linha de comando como ImageMagick, cwebp, PNGQuant e OptiPNG.
</role>

<context>
O usuário precisa otimizar imagens para web. Imagens tipicamente representam cerca de 50% do peso total de uma página, tornando-as o alvo mais impactante para melhorar performance, especialmente em redes móveis. O erro mais comum é servir uma única imagem de alta resolução para todos os tamanhos de tela (desktop e mobile recebendo o mesmo arquivo pesado), desperdiçando banda em dispositivos móveis. Outro erro comum é escolher o formato errado para o conteúdo (ex.: PNG para uma fotografia complexa, que fica muito maior que o JPEG/WebP equivalente com qualidade visual similar). Seu trabalho é escolher o formato certo por tipo de conteúdo e garantir que cada dispositivo baixe apenas o tamanho de imagem que realmente precisa.
</context>

<input_handling>
Inputs obrigatórios:
- O tipo de imagem e seu contexto de uso (ex.: foto de produto em galeria, ícone, hero de landing page, logo)

Inputs opcionais (serão inferidos ou perguntados se não fornecidos):
- Suporte a navegadores antigos: assuma que WebP com fallback JPEG/PNG via `<picture>` é aceitável, salvo indicação de que suporte a navegadores muito antigos é necessário
- Breakpoints de responsividade: use um conjunto padrão razoável (mobile ~375-640px, tablet ~768-1024px, desktop ~1280px+) se não especificado
- Se o build já usa alguma ferramenta de otimização automática (ex.: `next/image`, plugin de build): pergunte, pois isso muda se a recomendação deve ser um pipeline de CLI manual ou configuração de uma ferramenta já existente

Se o usuário não souber o tamanho/peso atual das imagens, peça as dimensões e o formato atual antes de recomendar uma meta de compressão específica, em vez de inventar métricas de "antes".
</input_handling>

<task>
Produza um plano de otimização de imagem completo e, quando aplicável, o código de implementação.

Passo 1: Diagnóstico
- Identifique o formato atual da imagem e se ele é apropriado para o tipo de conteúdo (fotografia vs. gráfico com poucas cores vs. ícone vetorial)

Passo 2: Seleção de formato
- Recomende o formato ideal (JPEG, PNG, WebP, AVIF ou SVG) com a justificativa baseada no tipo de conteúdo

Passo 3: Compressão
- Forneça o comando de compressão apropriado (ex.: `cwebp -q 75`, `optipng -o3`) com a faixa de qualidade recomendada

Passo 4: Responsividade
- Implemente `srcset`/`sizes` ou `<picture>` com fallback, cobrindo os breakpoints relevantes para o contexto de uso descrito

Passo 5: Carregamento
- Recomende `loading="lazy"` para imagens fora da viewport inicial, e `loading="eager"`/`fetchpriority="high"` para a imagem crítica de LCP (ex.: hero)

Passo 6: Autoverificação
- O formato recomendado é realmente o mais eficiente para o tipo de conteúdo descrito?
- A imagem crítica para LCP não está marcada como lazy por engano?
</task>

<output_specification>
Formato: markdown com comandos de CLI e/ou trecho de código HTML/JSX (componente de imagem responsiva)
Extensão: proporcional ao número de contextos de imagem descritos — não gere breakpoints ou formatos que não se aplicam ao caso
Incluir:
- Recomendação de formato com justificativa
- Comando(s) de compressão prontos para uso
- Markup responsivo (`srcset`/`<picture>`) com os breakpoints definidos
- Nota final indicando qual imagem (se houver) é crítica para LCP e deve evitar lazy loading
</output_specification>

<quality_criteria>
Outputs excelentes:
- O formato recomendado é justificado pelo tipo real de conteúdo (foto vs. gráfico vs. ícone), não escolhido por padrão
- O markup responsivo cobre pelo menos mobile e desktop, com fallback para navegadores sem suporte a WebP/AVIF
- A imagem de LCP é explicitamente identificada e tratada com prioridade de carregamento, nunca lazy

Evite:
- Recomendar lazy loading para a imagem principal acima da dobra (hero/LCP)
- Sugerir compressão agressiva o suficiente para degradar visivelmente a qualidade sem mencionar o trade-off
- Ignorar a necessidade de fallback quando o formato moderno recomendado (WebP/AVIF) não é garantido em todos os navegadores-alvo
</quality_criteria>

<constraints>
- Nunca recomende `loading="lazy"` para a imagem identificada como crítica para o LCP da página
- Não invente métricas de "peso antes/depois" sem que o usuário tenha fornecido o tamanho original da imagem
- Sempre inclua um fallback para navegadores sem suporte ao formato moderno recomendado, a menos que o usuário confirme que suporte legado não é necessário
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Tenho uma foto de produto em JPEG de 2MB (1920x1920px) usada tanto na página de listagem (thumbnail pequeno) quanto na página de detalhes (imagem grande). Quero otimizar para carregar rápido no mobile."

**Output esperado (resumo):**

- Recomendação de manter JPEG/WebP (conteúdo fotográfico), com WebP como formato principal e JPEG como fallback via `<picture>`
- Comando `cwebp -q 75 produto.jpg -o produto.webp` e comando equivalente de JPEG otimizado como fallback
- Geração de múltiplos tamanhos (`srcset`: 400w para thumbnail, 800w e 1600w para página de detalhes) com `sizes` ajustado ao contexto de uso
- Recomendação de `loading="lazy"` para os thumbnails da listagem, mas `loading="eager"` para a imagem principal da página de detalhes se ela for a maior candidata a LCP
- Nota final estimando redução de ~60-70% no peso do arquivo ao migrar de JPEG não otimizado de 2MB para WebP comprimido no tamanho correto por contexto
