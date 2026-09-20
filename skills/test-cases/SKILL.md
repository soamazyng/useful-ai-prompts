---
name: test-cases
description: >
  Generate comprehensive, requirement-driven test cases from PRD documents or
  user requirements, covering functional, edge case, error handling, and
  state transition scenarios. Use for test case generation, QA planning, test
  scenario creation, and structured test documentation.
---

# Test Cases Generator

## Table of Contents

- [Overview](#overview)
- [When to Use](#when-to-use)
- [Quick Start](#quick-start)
- [Reference Guides](#reference-guides)
- [Best Practices](#best-practices)

## Overview

Transforms product requirements into structured test cases that ensure complete coverage of functionality, edge cases, error scenarios, and state transitions. Follows a pragmatic testing philosophy: test what matters, ensure every requirement has corresponding test coverage, and favor test quality over quantity.

## When to Use

- User provides a PRD or requirements document and requests test cases
- User asks to "generate test cases", "create test scenarios", or "plan QA"
- User mentions testing coverage for a feature or requirement
- User needs structured test documentation in markdown format

## Quick Start

Minimal working example of a single test case:

```markdown
#### TC-F-001: Successful login with valid credentials

- **Requirement**: REQ-001 — User authentication
- **Priority**: High
- **Preconditions**:
  - User account exists with username "testuser"
  - User is logged out
- **Test Steps**:
  1. Navigate to the login page
  2. Enter username "testuser" and password "password123"
  3. Click "Login"
- **Expected Results**:
  - User is redirected to the dashboard
  - Welcome message displays "Welcome, testuser"
- **Postconditions**: User session is active
```

Full test suites follow the same shape at scale — see the template and workflow in the reference guides below.

## Reference Guides

Detailed implementations in the `references/` directory:

| Guide | Contents |
|---|---|
| [Testing Principles](references/testing-principles.md) | Core testing philosophy, coverage requirements, test design patterns (AAA, equivalence partitioning, state transition tables), prioritization, and common pitfalls |
| [Test Case Workflow](references/test-case-workflow.md) | Step-by-step process for gathering requirements, extracting scenarios, generating test cases, validating coverage, and the pre-delivery quality checklist |

A ready-to-fill document structure is available in [templates/test-cases-template.md](templates/test-cases-template.md).

## Best Practices

### ✅ DO

- Trace every test case back to a specific requirement
- Cover happy path, edge cases, error handling, and state transitions for each requirement
- Use unique ID prefixes: `TC-F` (functional), `TC-E` (edge), `TC-ERR` (error), `TC-ST` (state)
- Write test steps clear and specific enough for any QA engineer to execute without ambiguity
- Make expected results measurable and verifiable
- Build a requirement-to-test-case coverage matrix before finalizing
- Save output to `tests/<name>-test-cases.md`

### ❌ DON'T

- Write test cases that describe implementation details instead of observable behavior
- Skip edge cases (empty inputs, boundary values, maximum limits)
- Omit error and failure scenarios in favor of only happy-path coverage
- Leave a requirement without at least one corresponding test case
- Use vague expected results ("works correctly", "looks fine")
- Create test cases that depend on execution order or shared mutable state

## Attribution

This skill is adapted from [stellarlinkco/myclaude](https://github.com/stellarlinkco/myclaude/tree/master/skills/test-cases). See [README.md](README.md) for details.
