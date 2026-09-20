# Política de Segurança

## Escopo

Este repositório contém uma biblioteca de prompts de IA projetados para fluxos de trabalho profissionais. Diferentemente de repositórios de software tradicionais, não há código executável que possa conter vulnerabilidades de segurança convencionais, como buffer overflows, injeção de SQL ou falhas de autenticação.

No entanto, bibliotecas de prompts têm suas próprias considerações de segurança únicas relacionadas à segurança de conteúdo e integridade dos prompts.

## Versões Suportadas

| Versão | Suportada          |
| ------ | ------------------ |
| main   | :white_check_mark: |

Apenas o branch `main` atual é mantido ativamente. Não fornecemos atualizações de segurança para commits ou branches históricos.

## O que Constitui um Problema de Segurança

Para uma biblioteca de prompts, problemas de segurança podem incluir:

### Alta Prioridade

- **Ataques de injeção de prompt**: Prompts projetados para manipular sistemas de IA a contornar diretrizes de segurança ou executar ações não intencionadas
- **Conteúdo malicioso**: Prompts que incentivam ou facilitam atividades prejudiciais, ilegais ou antiéticas
- **Prompts de exfiltração de dados**: Conteúdo projetado para enganar sistemas de IA a revelar informações sensíveis

### Média Prioridade

- **Tentativas de jailbreak**: Prompts criados para contornar medidas de segurança de IA
- **Prompts enganosos**: Conteúdo que desvirtua seu propósito ou contém instruções ocultas
- **Templates de engenharia social**: Prompts projetados para manipular usuários ou terceiros

### Prioridade Menor

- **Conteúdo tendencioso ou discriminatório**: Prompts que promovem tratamento injusto de indivíduos ou grupos
- **Templates de desinformação**: Prompts projetados para gerar conteúdo falso ou enganoso

## Reportando uma Vulnerabilidade

Se você descobrir um problema de segurança nesta biblioteca de prompts, por favor reporte através de um dos seguintes canais:

### Preferido: GitHub Security Advisories

1. Navegue até a [aba Security](../../security) deste repositório
2. Clique em "Report a vulnerability"
3. Forneça uma descrição detalhada do problema

### Alternativa: Issue Privada

Se você não puder usar o GitHub Security Advisories:

1. Abra uma nova issue com o prefixo `[SECURITY]` no título
2. Forneça detalhes mínimos publicamente
3. Solicite um canal de comunicação privado para divulgação completa

**Por favor, não:**

- Divulgue publicamente os detalhes completos de problemas de segurança antes que sejam resolvidos
- Envie relatórios de segurança para problemas que não se enquadram no escopo definido acima

## O que Incluir no seu Relatório

- Localização do prompt problemático (caminho do arquivo e números de linha, se aplicável)
- Descrição da preocupação de segurança
- Impacto potencial ou dano que poderia resultar
- Quaisquer passos de remediação sugeridos

## Cronograma de Resposta

| Ação                                          | Prazo Esperado         |
| --------------------------------------------- | ---------------------- |
| Confirmação inicial                           | Dentro de 48 horas     |
| Avaliação preliminar                          | Dentro de 5 dias úteis |
| Resolução para problemas de alta prioridade   | Dentro de 14 dias      |
| Resolução para prioridade média/menor         | Dentro de 30 dias      |

Esses prazos são metas e podem variar com base na complexidade do problema e disponibilidade dos mantenedores.

## Nosso Compromisso

Levamos a segurança e integridade desta biblioteca de prompts a sério. Quando um problema de segurança válido for reportado, nós iremos:

1. Confirmar o recebimento do seu relatório prontamente
2. Investigar o problema minuciosamente
3. Remover ou remediar o conteúdo problemático
4. Creditar os reportadores (a menos que anonimato seja solicitado) em nosso changelog
5. Comunicar de forma transparente sobre a resolução

## Dúvidas

Para perguntas gerais sobre esta política de segurança, por favor abra uma issue padrão no repositório.
