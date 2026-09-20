# CLAUDE.md

Este arquivo fornece orientações ao Claude Code (claude.ai/code) ao trabalhar com código neste repositório.

## Visão Geral do Repositório

Este é o repositório **Useful AI Prompts** — uma biblioteca abrangente de prompts especializados projetados para assistentes de IA adotarem personas de especialistas ao completar tarefas. O repositório contém 259+ prompts organizados em 14 categorias, combinando múltiplas perspectivas de especialistas com frameworks profissionais.

## Comandos Principais

### Comandos de Desenvolvimento
```bash
# Instalar dependências
npm install

# Formatação de código
npx prettier --write .

# Scripts de validação
python validate_jekyll_conversion.py
python update_prompt_index.py
```

### Website Jekyll (diretório docs/)
```bash
cd docs/
bundle install           # Instalar dependências Ruby
bundle exec jekyll build # Compilar o site
bundle exec jekyll serve # Servir localmente (http://localhost:4000)
```

### Gerenciamento de Prompts
```bash
# Converter prompts do formato antigo para o novo formato conversacional
python batch_convert_prompts.py

# Corrigir erros de conversão
python fix_conversion_errors.py

# Atualizar o índice de prompts
python update_prompt_index.py
```

## Visão Geral da Arquitetura

### Estrutura de Diretórios
- **`/prompts/`** - Biblioteca principal de prompts organizada por domínio (técnico, negócios, criativo, especializado)
- **`/docs/`** - Website Jekyll para navegação de prompts (deploy via GitHub Pages)
- **`/metadata/`** - Definições de frameworks e diretrizes
- **`/_prompts/`** (em docs) - Coleção Jekyll de prompts convertidos para o website
- **Scripts Python** - Utilitários de conversão, validação e manutenção

### Arquitetura de Prompts
Cada prompt segue um formato conversacional e amigável:
- **Metadados claros** (categoria, tags, casos de uso)
- **Descrição útil** explicando o propósito do prompt
- **Perguntas interativas** para entender o contexto do usuário
- **Entregáveis estruturados** baseados nas necessidades do usuário
- **Exemplos práticos** mostrando uso

### Fluxo de Dados
1. Prompts fonte nos diretórios `/prompts/`
2. Scripts de conversão transformam para formato Jekyll em `/docs/_prompts/`
3. Jekyll compila o website a partir do diretório `/docs/`
4. PROMPT-INDEX.json fornece catálogo legível por máquina

## Padrões de Organização de Arquivos

### Arquivos de Prompts
- Localizados em subdiretórios categorizados em `/prompts/`
- Usar nomenclatura kebab-case: `strategic-roadmap-generator.md`
- Incluir papel/função no nome do arquivo
- Seguir formato conversacional com estrutura clara

### Coleções Jekyll
- `_prompts/` - Páginas individuais de prompts
- `_categories/` - Páginas de índice de categorias
- Usa frontmatter para metadados (título, categoria, tags, etc.)

### Utilitários Python
- `batch_convert_prompts.py` - Converte formato antigo para novo formato conversacional
- `convert_prompts_to_jekyll.py` - Conversão para formato Jekyll
- `validate_jekyll_conversion.py` - Validação e verificação de erros
- `update_prompt_index.py` - Gera/atualiza PROMPT-INDEX.json

## Trabalhando com Prompts

### Skill Obrigatória: prompt-refactor

**IMPORTANTE**: Ao criar, editar, refatorar ou melhorar qualquer prompt neste repositório, você DEVE usar a skill `prompt-refactor` localizada em `.claude/skills/prompt-refactor/`.

A skill fornece:
- Estrutura de template padronizada para todos os prompts
- 11 verificações de qualidade para validação
- Suporte a processamento em lote para múltiplos prompts
- Metadados e formatação consistentes

**Como usar:**
```bash
# Refatoração de prompt único
claude "Using prompt-refactor skill, refactor this prompt: [cole ou caminho]"

# Processamento em lote
./.claude/skills/prompt-refactor/scripts/orchestrate-refactor.sh ./prompts ./output 4

# Validar um prompt refatorado
./.claude/skills/prompt-refactor/scripts/validate-prompt.sh ./prompts/my-prompt.md
```

**Gatilhos da skill** (ativada automaticamente nestas frases):
- "refactor prompt"
- "improve prompt"
- "standardize prompts"
- "prompt template"
- "apply prompt template"

Veja `.claude/skills/prompt-refactor/SKILL.md` para especificação completa do template e verificações de qualidade.

### Adicionando Novos Prompts
1. Criar prompt no diretório `/prompts/[categoria]/` apropriado
2. **Usar a skill prompt-refactor** para garantir a estrutura adequada
3. Seguir o template padronizado com metadados, papel, tarefa, especificação de saída
4. Validar com o script de validação
5. Executar scripts de conversão para gerar versão Jekyll
6. Atualizar PROMPT-INDEX.json

### Padrões de Qualidade de Prompts (aplicados pela skill prompt-refactor)
- Metadados completos (ID, versão, categoria, tags, complexidade, interação, modelos)
- Visão geral concisa (≤3 frases)
- Definição de papel específica com expertise concreta
- Inputs categorizados (obrigatórios vs opcionais)
- Tarefa estruturada (3-7 passos claros)
- Saídas especificadas (formato + extensão + requisitos)
- Critérios de qualidade mensuráveis
- Exemplos realistas (entrada: 20-200 palavras, saída: 100-600 palavras)
- Seção de prompt pronta para copiar e colar

### Processo de Conversão
Prompts no formato antigo são convertidos para o novo formato padronizado usando:
- Extrair metadados chave (título, categoria, tags)
- Transformar em tags XML estruturadas (role, input_handling, task, output_specification, etc.)
- Aplicar validação de verificações de qualidade
- Adicionar exemplos práticos e entregáveis claros

## Deploy do Website

O website Jekyll é automaticamente implantado no GitHub Pages a partir do diretório `/docs/`:
- URL base: `/useful-ai-prompts`
- Coleções: prompts e categorias
- Funcionalidade de busca habilitada
- Design responsivo com navegação por categorias

## Diretrizes de Integração

Para agentes de IA trabalhando com este repositório:
1. Analisar requisitos da tarefa para identificar a categoria de prompt apropriada
2. Corresponder à taxonomia de prompts usando `/prompts/[categoria]/[subcategoria]/`
3. Carregar e personalizar o prompt selecionado com variáveis específicas da tarefa
4. Executar usando o framework padrão de 4 fases
5. Referenciar PROMPT-INDEX.json para seleção programática

## Notas Importantes

- Este repositório foca em **prompts úteis** para fluxos de trabalho profissionais
- Cada prompt fornece orientação especializada em formato conversacional
- Prompts coletam contexto através de perguntas e fornecem entregáveis estruturados
- O sistema de conversão mantém compatibilidade entre formatos fonte e Jekyll
- Padrões de qualidade focam em clareza, usabilidade e valor prático
