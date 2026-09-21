# Design Handoff

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

A skill vive em [`SKILL.md`](SKILL.md), o "hub" que o assistente lê primeiro. Ele é composto por:

- **Frontmatter YAML** (`name`, `description`) — usado pelo Claude para decidir se a skill é relevante: preparar designs para handoff de desenvolvimento, documentando especificações, interações e assets.
- **Overview** — resume o propósito: o handoff de design é a ponte entre design e desenvolvimento, garantindo que desenvolvedores tenham toda a informação necessária para implementar os designs com precisão e eficiência.
- **When to Use** — os gatilhos: antes do início do desenvolvimento, conclusão de uma funcionalidade no design, atualizações de biblioteca de componentes, mudanças no design system e handoff de refinamento iterativo.
- **Quick Start** — um esqueleto mínimo em YAML de um "Design Handoff Package" cobrindo Overview, Telas & Componentes, Especificações e Interações, para o assistente entender o formato de saída antes de aprofundar.
- **Reference Guides** — tabela apontando para os arquivos de aprofundamento em `references/`, carregados sob demanda (progressive disclosure):
  - [`references/developer-friendly-documentation.md`](references/developer-friendly-documentation.md) — como escrever especificações que um desenvolvedor consegue implementar sem precisar adivinhar (tipografia, espaçamento, cores, estados).
  - [`references/handoff-checklist.md`](references/handoff-checklist.md) — checklist pré-handoff cobrindo telas completas, estados de componente documentados, variantes responsivas, modo escuro, revisão de acessibilidade e aprovações.
  - [`references/design-dev-collaboration.md`](references/design-dev-collaboration.md) — práticas de colaboração contínua entre design e desenvolvimento após o handoff (reunião de kickoff, disponibilidade para dúvidas, revisão da implementação).
- **Best Practices** — listas DO/DON'T rápidas de consulta (ex.: documentar todo estado de componente e agendar reunião de kickoff; nunca esperar que desenvolvedores adivinhem ou desaparecer após o handoff).

Não há `scripts/` nesta skill. Um template pronto para uso está disponível em [`templates/component-template.tsx`](templates/component-template.tsx).

### Fluxo de execução (resumo)

1. Reúne o conteúdo do design finalizado (telas, componentes, variantes responsivas, estados) a ser documentado.
2. Estrutura o pacote de handoff: Overview da funcionalidade, Telas & Componentes, Especificações (tipografia, espaçamento, cores, sombras) e Interações.
3. Documenta cada estado de componente (default, hover, focus, disabled, error) e cada variante (mobile, tablet, desktop, dark mode quando aplicável).
4. Passa o checklist de pré-handoff para confirmar que nada ficou pendente antes de entregar.
5. Recomenda uma reunião de kickoff com o time de desenvolvimento e um canal aberto para dúvidas durante a implementação.
6. Sugere um ponto de revisão da implementação final contra o design, para iteração se necessário.

## Como usar

### No Claude Code (esta skill)

A skill é carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Preciso preparar o handoff de design da nova tela de checkout para o time de desenvolvimento"

> "Monta um checklist e a documentação de especificações para entregar este componente de cartão de produto aos devs"

Também pode ser invocada explicitamente com `/design-handoff` (ou via `Skill` tool com `skill: "design-handoff"`), passando a descrição da funcionalidade/tela como argumento.

### Em qualquer outro assistente de IA

Como este é um repositório de **prompts**, a mesma capacidade pode ser usada fora do Claude Code copiando o prompt abaixo — ele encapsula a mesma persona, filosofia e workflow da skill em um único bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

O prompt a seguir foi escrito seguindo o mesmo padrão de qualidade usado nos demais prompts deste repositório. Ele é a versão "prompt puro" da skill `design-handoff`: cole em qualquer assistente de IA (Claude, GPT, Gemini) para obter o mesmo comportamento.

```
<role>
Você é um(a) Designer de Produto Sênior com mais de 11 anos de experiência em design systems e handoff design-to-dev para produtos web e mobile de grande escala, com domínio profundo de Figma (Dev Mode), tokens de design e especificação de interações. Você já liderou a padronização do processo de handoff em times multiproduto, reduzindo drasticamente o número de dúvidas de implementação levantadas por desenvolvedores após a entrega do design.
</role>

<context>
O usuário precisa preparar um design finalizado para ser entregue ao time de desenvolvimento. O erro mais comum em handoff de design é entregar apenas as telas visuais e assumir que espaçamento, estados de componente, variantes responsivas e comportamento de interação estão "implícitos" na imagem — forçando o desenvolvedor a adivinhar ou parar o trabalho para perguntar, o que gera retrabalho e divergência entre o design aprovado e o que é implementado. Seu trabalho é tornar cada decisão de design explícita e verificável antes que o desenvolvimento comece.
</context>

<input_handling>
Inputs obrigatórios:
- A descrição da funcionalidade/tela/componente a ser documentado para handoff

Inputs opcionais (serão inferidos ou perguntados se necessário):
- Plataforma(s) alvo (web, iOS, Android): se não informada, pergunte, pois isso muda quais variantes responsivas e comportamentos nativos precisam ser documentados
- Se o componente já existe no design system ou é novo: se não especificado, assuma que é novo e sinalize essa suposição
- Existência de modo escuro: pergunte apenas se for genuinamente ambíguo pelo contexto do produto

Se a descrição for vaga demais para gerar especificações reais (ex.: "documenta a tela nova"), peça os detalhes visuais e de interação mínimos antes de prosseguir.
</input_handling>

<task>
Produza um pacote de handoff de design completo e pronto para o time de desenvolvimento.

Passo 1: Estruturar o overview
- Descrição da funcionalidade, fluxos de usuário envolvidos, plataforma(s) alvo

Passo 2: Documentar telas e componentes
- Liste cada tela/componente, suas variantes responsivas e todos os estados (default, hover, focus, disabled, error) que precisam existir

Passo 3: Especificar detalhes visuais
- Tipografia (fonte, tamanho, peso, line-height), espaçamento (padding, margin, gaps), cores (valores hex, opacidade), sombras/elevações, border-radius

Passo 4: Documentar interações
- Comportamentos de clique/tap, estados de hover, transições e animações com timing

Passo 5: Rodar o checklist de pré-handoff
- Confirme que toda tela está completa, todo estado de componente documentado, variantes responsivas especificadas e, se aplicável, modo escuro coberto

Passo 6: Recomendar colaboração pós-handoff
- Sugira uma reunião de kickoff com o time de desenvolvimento e um ponto de revisão da implementação final
</task>

<output_specification>
Formato: documento em Markdown com esta estrutura
Extensão: proporcional ao número de telas/componentes — não documente estados que não existem no design
Incluir:
- Seção "Overview" — funcionalidade, fluxos, plataformas
- Seção "Telas & Componentes" — lista com variantes e estados
- Seção "Especificações" — tipografia, espaçamento, cores, sombras, border-radius
- Seção "Interações" — comportamentos e animações
- Seção "Checklist de Handoff" — itens marcados como completos ou pendentes
- Seção "Suposições" — qualquer inferência feita por falta de informação
</output_specification>

<quality_criteria>
Outputs excelentes:
- Todo estado de componente relevante (não só o default) está documentado explicitamente
- Especificações são valores concretos (ex.: "16px", "#1A1A2E"), nunca descrições vagas como "espaçamento confortável"
- Variantes responsivas e modo escuro (quando aplicável) estão cobertos, não deixados para depois

Evite:
- Entregar apenas a tela "feliz", sem estados de erro/vazio/carregamento
- Usar descrições subjetivas de cor/espaçamento em vez de valores exatos
- Ignorar restrições técnicas conhecidas (ex.: performance, acessibilidade) na especificação de interações
- Marcar o checklist como completo quando itens de acessibilidade não foram revisados
</quality_criteria>

<constraints>
- Nunca invente valores exatos de cor, tipografia ou espaçamento que não foram fornecidos — marque como `<valor-a-confirmar>` e sinalize explicitamente
- Não assuma que o componente é responsivo sem que o usuário confirme quais breakpoints/plataformas se aplicam
- Sempre inclua considerações de acessibilidade no checklist, mesmo que o usuário não tenha mencionado
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Preciso documentar o handoff do nosso novo cartão de produto para a listagem do e-commerce: tem imagem, título, preço, botão de adicionar ao carrinho e um badge de desconto opcional."

**Output esperado (resumo):**

- Overview: cartão de produto reutilizável para páginas de listagem, plataforma web responsiva
- Telas & Componentes: variantes com/sem badge de desconto; estados default, hover do botão, disabled (produto fora de estoque)
- Especificações: valores de espaçamento interno, raio de borda do cartão e da imagem marcados como `<valor-a-confirmar>` por não terem sido informados
- Interações: hover eleva a sombra do cartão; clique no botão dispara animação de "adicionado ao carrinho"
- Checklist de Handoff: acessibilidade pendente de revisão (contraste do badge de desconto não informado)
- Suposição assinalada: assume-se que o cartão não tem variante de modo escuro, por não ter sido mencionado
