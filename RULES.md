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

## 🔒 CRITICAL: Pull Requests Must Target the Fork Only — Never the Upstream Repository

**This repository is a fork.** `git remote -v` shows two remotes:

```
origin    https://github.com/soamazyng/useful-ai-prompts.git   (this fork — the ONLY valid PR target)
upstream  https://github.com/aj-geddes/useful-ai-prompts       (the original project — NEVER open PRs here)
```

`gh pr create` auto-detects the parent repository when a fork has an `upstream` remote configured, and **defaults to opening the PR against `upstream`** unless told otherwise. This is the opposite of what we want here — every PR from this repo must land on `origin` (`soamazyng/useful-ai-prompts`), not on `aj-geddes/useful-ai-prompts`.

### Rule

**Always pass `--repo` explicitly** when creating a PR from this repository:

```bash
gh pr create --repo soamazyng/useful-ai-prompts --base main --head feature/[description]
```

Never run a bare `gh pr create` in this repo — it will silently target the wrong repository.

### If a PR is accidentally opened against `upstream`

1. Close it on the upstream repo (`gh pr close <number> --repo aj-geddes/useful-ai-prompts`)
2. Re-open it correctly with `--repo soamazyng/useful-ai-prompts`

### Checklist Before Any `gh pr create`

- [ ] Command includes `--repo soamazyng/useful-ai-prompts`
- [ ] `--base` is `main` (this fork's main, not upstream's)
- [ ] `--head` is the local feature branch that was pushed to `origin`
- [ ] Confirm the printed PR URL starts with `github.com/soamazyng/useful-ai-prompts/pull/`, not `github.com/aj-geddes/useful-ai-prompts/pull/`

---

## 📘 Required README.md Format for Every Skill (`skills/<name>/README.md`)

**Every new or adapted skill added under `skills/<name>/` must ship a `README.md`** in addition to the `SKILL.md` hub. The canonical reference implementation is [`skills/test-cases/README.md`](skills/test-cases/README.md) — use it as the template for structure, tone, and depth.

### Required Sections (in this order)

1. **Attribution + License** (if the skill is adapted from an external source)
   - Name/handle of the original author and a link to the original repository
   - Note that it was adapted to this project's Progressive Disclosure architecture (link to `skills/README.md`)
   - License of the original work

2. **"Como a skill funciona" (How the skill works)**
   - Walk through every section of `SKILL.md` (frontmatter, Overview, When to Use, Quick Start, Reference Guides, Best Practices) and explain, in plain language, what each one is for and why it exists — not just that it exists
   - List each file under `references/` with a one-line description of what it covers, and explain that these are loaded on demand (progressive disclosure), not upfront
   - Include a short numbered **"Fluxo de execução"** summarizing the skill's workflow end to end (the steps a reader would find inside `references/*-workflow.md`, condensed)

3. **"Como usar" (How to use)**
   - A subsection for using it inside Claude Code (trigger phrases matching the `description`, explicit invocation via `/skill-name` or the `Skill` tool)
   - A subsection for using the skill's capability in **any other AI assistant** by copying the standalone prompt (see next section) — this repository is a prompt library first, so every skill must remain usable without Claude Code

4. **"Prompt de Exemplo — Copiar e Colar" (Example Prompt — Copy and Paste)**
   - A single, self-contained, production-quality prompt that reproduces the skill's behavior in any LLM chat interface
   - **Must follow the same structure used throughout `prompts/`**: `<role>` (concrete persona with real expertise, years of experience, named methodologies/certifications — never a generic "you are a helpful assistant"), `<context>` (why this matters, what failure mode the skill prevents), `<input_handling>` (required vs. optional inputs, what to do when input is ambiguous), `<task>` (numbered steps), `<output_specification>` (format, length, required contents), `<quality_criteria>` (what excellent output looks like vs. what to avoid), `<constraints>` (hard rules, what never to fabricate)
   - **Write the prompt body in Portuguese** (pt-BR) — translate everything inside the tags, but **keep the XML tag names themselves in English** (`<role>`, `<context>`, etc.) since they are structural delimiters, not prose, and match the convention used across this repository's schema (see the Critical Rule section above)
   - Follow the prompt block with a short **"Exemplo de uso do prompt"** subsection showing a realistic Input and a summarized Output, so a reader can verify the prompt works before pasting it elsewhere

### Why This Matters

A skill without this README is only usable by an agent that already knows to read `SKILL.md` — it is opaque to a human browsing the repository and worthless outside Claude Code. The standalone prompt in section 4 is what makes every skill in `skills/` double as a prompt in the spirit of this repository's name: **Useful AI Prompts**.

### Checklist Before Adding a New Skill

- [ ] `SKILL.md` hub exists with valid frontmatter (`name`, `description`)
- [ ] `README.md` exists with all four required sections above, in the order given
- [ ] The example prompt inside `README.md` is a complete `<role>/<context>/<input_handling>/<task>/<output_specification>/<quality_criteria>/<constraints>` block, written in Portuguese, with English tag names
- [ ] The example prompt was sanity-checked against the "Exemplo de uso" — does the described input plausibly produce the described output?
- [ ] `npx prettier --check` passes on all new files

---

## 📝 Last Updated

- **2026-09-20** — Created to prevent schema translation errors in README.md
- **2026-09-20** — Added Git Workflow & Best Practices section
- **2026-09-20** — Added critical rule: PRs must target the fork (`origin`) only, never `upstream`
- **2026-09-20** — Added required README.md format for skills, using `skills/test-cases/README.md` as the canonical reference
- **Status**: Active for all future translation and contribution work in this repository

---

**Responsibility**:

- Any translator should review this document before making changes
- Every contributor must follow feature branch workflow
- No direct commits to `main` — always use feature branches and PRs
