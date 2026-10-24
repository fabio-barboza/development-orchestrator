# Regression Test Patterns

Reference for writing effective regression tests for each bug type.

## Core Rule
A regression test MUST fail when the fix is reverted. If it passes without the fix, it's not testing the bug.

## Pattern by Bug Type

### Unit Test (isolated logic bug)
Pseudo-code — write it in the project's own test framework and style:
```
test "BugFix: <bug description>":
    # Arrange: reproduce the exact input that triggered the bug
    input = <exact_bad_input>

    # Act
    result = unit_under_test(input)

    # Assert: the previously broken behavior now works
    assert result equals <expected_correct_value>
```
Use for: off-by-one errors, missing-value handling, incorrect calculations, wrong conditions.

### Integration Test (module communication bug)
```
test "BugFix: <bug description>":
    # Arrange: set up the real dependency chain that was broken
    dependency = real <dependency>
    system = <system_under_test>(dependency)

    # Act: trigger the exact flow that caused the bug
    result = system.<operation_that_was_broken>(<inputs>)

    # Assert
    assert result has <expected_shape>
    assert dependency state equals <expected_side_effect>
```
Use for: wrong data passed between layers, missing synchronization/await, incorrect mapping, bad error propagation.

### E2E Test (UI/flow bug)
```
# Use the UI-testing MCP tools to reproduce the user flow that triggered the bug
# 1. Navigate to the affected screen
# 2. Perform the exact sequence of actions that caused the bug
# 3. Assert the corrected behavior (screenshot evidence)
```
Use for: UI state not updating, form validation not triggering, navigation broken, visual regression.

## Edge Cases to Cover
After the main regression test, add at least one edge case test:
- Missing / null-like input
- Empty text or empty collection
- Boundary values (0, -1, max)
- Concurrent operations (if applicable)

## Naming Convention
```
"BugFix #<id>: <what should happen> when <trigger condition>"
```
Example: `"BugFix #12: should return empty array when input list is null"`

## Verification Checklist
- [ ] Test fails on the unfixed code (verified by reverting and running)
- [ ] Test passes on the fixed code
- [ ] Test name clearly describes the bug scenario
- [ ] At least one edge case covered
- [ ] No mocks hiding the root cause (integration/E2E tests use real dependencies)
