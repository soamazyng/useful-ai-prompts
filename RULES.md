# Translation and Localization Rules

**Language**: This document defines strict rules for translating content in the `useful-ai-prompts` repository to maintain compatibility with automated tooling and the prompt schema.

---

## 🔴 Critical Rule: Schema Compatibility

**The following MUST remain in English to preserve compatibility with Python processing scripts** (`convert_prompts_to_jekyll.py`, `update_prompt_index.py`, Jekyll generation, etc.):

### 1. Schema Metadata Headings (Exact Match Required)

These headings are hardcoded into scripts and MUST NOT be translated:

```markdown
## Metadata
## Overview
## When to Use
## Prompt
## Example Usage
## Related Prompts
```

### 2. Metadata Keys (Exact Match Required)

These keys are parsed by regex/string matching and MUST remain in English:

```
- **ID**: (unchanged)
- **Version**: (not Versão)
- **Category**: (not Categoria)
- **Tags**: (unchanged)
- **Complexity**: (not Complexidade)
- **Interaction**: (not Interação)
- **Models**: (not Modelos)
- **Author**: (if used)
- **Last Updated**: (if used)
- **Use Cases**: (if used)
```

### 3. Metadata Enum Values (Exact Match Required)

These values are validated by enum matching:

```
Complexity:
  - simple (not simples)
  - intermediate (not intermediário)
  - advanced (not avançado)

Interaction:
  - single-shot (not única)
  - conversational (not conversacional)
  - iterative (not iterativo)

Models:
  - Claude 3+
  - GPT-4+
  (keep as-is)
```

### 4. XML Tag Names (Exact Match Required)

The following tags are part of the schema and MUST remain in English:

```xml
<role>
<context>
<input_handling>
<task>
<output_specification>
<output_format>
<quality_criteria>
<constraints>
```

### 5. Main Section Headings (Schema-Level)

The following section headings are referenced by tooling and MUST be in English:

```markdown
## Metadata
## Overview
## When to Use
## Prompt
## Example Usage
## Related Prompts
## XML Tag Structure (or similar schema doc sections)
## Quality Checks (or similar validation sections)
```

---

## ✅ What CAN be translated to Portuguese (pt-BR)

These elements should be translated for user readability:

### Prose and Descriptions
- **Descriptive text** in section introductions
- **Examples of prompt outputs** (the user-facing content within `<role>`, `<context>`, `<task>` tags when showing translations of what prompts do)
- **Navigation text** ("Consulte X para...", "Navigate to...", "See...")
- **Commentary in tables** (column descriptions, explanations)
- **Instructions and guidance** in sections like "Quick Start", "Contributing", etc.

### Section Headings (NOT Schema-Level)
These can be translated if they are organizational/navigational (not part of prompt schema):

```markdown
✅ CAN translate:
- Quick Start → "Início Rápido" (organizational, not schema)
- Contributing → "Contribuindo" (organizational)
- Web Interface → "Interface Web" (organizational)
- License → "Licença" (organizational)
- Links → "Links" (organizational)

❌ CANNOT translate:
- Metadata → must stay "Metadata"
- Overview → must stay "Overview"
- When to Use → must stay "When to Use"
- Prompt → must stay "Prompt"
- Example Usage → must stay "Example Usage"
```

---

## 📋 Translation Checklist

Before translating any content in this repository, verify:

- [ ] **Metadata headings** remain: `## Metadata`, `## Overview`, `## When to Use`, `## Prompt`, `## Example Usage`, `## Related Prompts`
- [ ] **Metadata keys** remain: `ID`, `Version`, `Category`, `Tags`, `Complexity`, `Interaction`, `Models`
- [ ] **Enum values** are not translated: `simple`, `intermediate`, `advanced`, `single-shot`, `conversational`, `iterative`
- [ ] **XML tags** remain: `<role>`, `<context>`, `<input_handling>`, `<task>`, `<output_specification>`, `<quality_criteria>`, `<constraints>`
- [ ] **Scripts and code examples** are unchanged (paths, filenames, command names)
- [ ] **Links and URLs** are preserved
- [ ] **JSON keys** in schema remain in English
- [ ] Only **descriptive prose** is translated

---

## 🛠️ Processing Scripts That Depend on Schema Compatibility

These Python scripts parse exact heading/key names and will fail if schema is translated:

1. **`convert_prompts_to_jekyll.py`** — Converts prompts to Jekyll format; searches for exact heading names
2. **`update_prompt_index.py`** — Generates `PROMPT-INDEX.json`; parses metadata keys
3. **`validate_jekyll_conversion.py`** — Validates prompt structure; checks for schema headings
4. **Jekyll configuration** (`docs/_config.yml`) — References collection names and front matter keys
5. **GitHub automation** — May depend on consistent metadata format

### Example of Why This Matters

```python
# From convert_prompts_to_jekyll.py (pseudocode)
if re.search(r'^## Metadata$', line):
    # This will NOT match "## Metadados"
    # The script will fail to extract metadata
```

---

## 🚀 Translation Workflow

When translating repository content:

1. **Identify the content** — Is it schema-level or descriptive?
2. **Check this RULES.md** — If it's in the critical list, leave it in English
3. **Translate only descriptive text** — Table descriptions, introduction paragraphs, guidance
4. **Verify in template** — Look at `CLAUDE.md` example structure to see which fields must stay English
5. **Test processing scripts** — Run `validate_jekyll_conversion.py` after translation to ensure no breakage
6. **Update this document** — If new critical fields are discovered, add them here

---

## ❓ Questions?

If you're unsure whether something should be translated:

1. **Does a Python script reference it by name?** → Leave it in English
2. **Does it affect `PROMPT-INDEX.json` generation?** → Leave it in English
3. **Is it user-facing prose?** → Translate it to pt-BR
4. **Is it a code example or technical path?** → Leave it as-is
5. **Uncertain?** → Check if it's listed in "Critical Rule" above. If in doubt, keep it English

---

## 🌿 Git Workflow & Best Practices

**CRITICAL**: Never push directly to `main` branch. Always follow feature branch workflow:

### Branch Naming Convention

Create feature branches with descriptive names following this pattern:

```
feature/[what-you-are-doing]
```

**Examples**:
```
feature/translate-readme-pt-br
feature/add-missing-hooks
feature/update-prompt-count
feature/fix-schema-compatibility
feature/add-rules-documentation
```

### Workflow Steps

1. **Create a feature branch** from `main`:
   ```bash
   git checkout -b feature/[description]
   ```

2. **Make your changes** in the feature branch:
   - Commit with clear, descriptive messages
   - Follow conventional commits (feat:, fix:, docs:, etc.)
   - Include proper attribution in commits

3. **Push to remote**:
   ```bash
   git push origin feature/[description]
   ```

4. **Create a Pull Request** on GitHub:
   - Link the PR to relevant issues
   - Add clear description of changes
   - Wait for code review and automated checks
   - Address feedback if any

5. **Merge to main** only after:
   - ✅ Pull request is approved
   - ✅ All CI/CD checks pass
   - ✅ Code review is complete
   - ✅ No conflicts with main

### What NOT to Do ❌

- ❌ **Never push directly to main**: `git push origin main`
- ❌ **Never force-push**: `git push --force`
- ❌ **Never commit to main locally and push**: Create a branch first
- ❌ **Never skip PR review**: Always create a PR for peer review
- ❌ **Never merge without passing checks**: Ensure CI/CD is green

### Commit Message Format

```
[type]: [description]

[optional detailed explanation]

Co-Authored-By: Claude Haiku 4.5 <noreply@anthropic.com>
```

**Types**: `feat:`, `fix:`, `docs:`, `refactor:`, `test:`, `chore:`

---

## 📝 Last Updated

- **2026-09-20** — Created to prevent schema translation errors in README.md
- **2026-09-20** — Added Git Workflow & Best Practices section
- **Status**: Active for all future translation work in this repository

---

**Responsibility**: 
- Any translator should review this document before making changes
- Every contributor must follow feature branch workflow
- No direct commits to `main` — always use feature branches and PRs
