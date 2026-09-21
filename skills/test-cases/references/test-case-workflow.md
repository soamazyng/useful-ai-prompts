# Test Case Generation Workflow

Step-by-step process for turning a PRD or a set of requirements into a complete, traceable test case document.

## Step 1: Gather Requirements

Identify the source of requirements:

1. If the user provides a file path to a PRD, read it.
2. If the user describes requirements verbally, capture them.
3. If requirements are unclear or incomplete, ask for clarification:
   - What are the core user flows?
   - What are the acceptance criteria?
   - What are the edge cases or error scenarios to consider?
   - Are there any state transitions or workflows?
   - What platforms or environments need testing?

## Step 2: Extract Test Scenarios

Analyze requirements and extract test scenarios:

1. **Functional scenarios** — normal use cases from requirements
2. **Edge case scenarios** — boundary conditions, empty states, maximum limits
3. **Error scenarios** — invalid inputs, permission failures, network errors
4. **State transition scenarios** — if the feature involves state, map all transitions

For each requirement, identify:

- Preconditions (what must be true before testing)
- Test steps (actions to perform)
- Expected results (what should happen)
- Postconditions (state after the test completes)

## Step 3: Structure Test Cases

Organize test cases using the structure in [templates/test-cases-template.md](../templates/test-cases-template.md):

- A document header (feature, requirements source, coverage summary, last updated)
- One section per category: Functional Tests, Edge Case Tests, Error Handling Tests, State Transition Tests
- A Test Coverage Matrix mapping requirement IDs to test case IDs and coverage status
- A Notes section for assumptions and known limitations

## Step 4: Generate Test Cases

For each identified scenario, create a detailed test case. Ensure:

1. **Unique IDs** — use prefixes: `TC-F` (functional), `TC-E` (edge), `TC-ERR` (error), `TC-ST` (state)
2. **Clear titles** — descriptive titles that explain what's being tested
3. **Requirement traceability** — link each test case to specific requirements
4. **Priority assignment** — mark critical paths as High priority
5. **Executable steps** — steps must be clear enough for any QA engineer to execute
6. **Measurable results** — expected results must be verifiable

## Step 5: Validate Coverage

Before finalizing, verify:

1. Every requirement has at least one test case
2. Happy path is covered for all user flows
3. Edge cases are identified for boundary conditions
4. Error scenarios are covered for failure modes
5. State transitions are tested if the feature is stateful

If coverage gaps exist, generate additional test cases.

## Step 6: Output Test Cases

Write the test cases to `tests/<name>-test-cases.md`, where `<name>` is derived from:

- The feature name from the PRD
- The user's specified name
- A sanitized version of the requirement title

## Step 7: Summary

After generating test cases, provide a brief summary in the same language the user is using:

- Total number of test cases generated
- Coverage breakdown (functional, edge, error, state)
- Any assumptions made or areas needing clarification
- File path where test cases were saved

## Quality Checklist

Before finalizing test cases, verify:

- [ ] Every requirement has corresponding test cases
- [ ] Happy path scenarios are covered
- [ ] Edge cases include boundary values, empty inputs, max limits
- [ ] Error handling covers invalid inputs and failure scenarios
- [ ] State transitions are tested if applicable
- [ ] Test case IDs are unique and follow the naming convention
- [ ] Test steps are clear and executable
- [ ] Expected results are measurable and verifiable
- [ ] Coverage matrix shows complete coverage
- [ ] File is written to `tests/<name>-test-cases.md`

## Example Usage

**User**: "Generate test cases for the user authentication feature in docs/auth-prd.md"

**Process**:

1. Read `docs/auth-prd.md`
2. Extract requirements: login, logout, password reset, session management
3. Identify scenarios: successful login, invalid credentials, expired session, etc.
4. Generate test cases covering all scenarios
5. Write to `tests/auth-test-cases.md`
6. Summarize coverage for the user
