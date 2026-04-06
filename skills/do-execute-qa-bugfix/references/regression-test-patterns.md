# Regression Test Patterns

Reference for writing effective regression tests for each bug type.

## Core Rule
A regression test MUST fail when the fix is reverted. If it passes without the fix, it's not testing the bug.

## Pattern by Bug Type

### Unit Test (isolated logic bug)
```
describe("BugFix: <bug description>", () => {
  it("should <correct behavior> when <bug scenario>", () => {
    // Arrange: reproduce the exact input that triggered the bug
    const input = <exact_bad_input>;

    // Act
    const result = functionUnderTest(input);

    // Assert: the previously broken behavior now works
    expect(result).toBe(<expected_correct_value>);
  });
});
```
Use for: off-by-one errors, null handling, incorrect calculations, wrong conditions.

### Integration Test (module communication bug)
```
describe("BugFix: <bug description>", () => {
  it("should <correct behavior> across <module-a> and <module-b>", async () => {
    // Arrange: set up the real dependency chain that was broken
    const dep = new RealDependency();
    const sut = new SystemUnderTest(dep);

    // Act: trigger the exact flow that caused the bug
    const result = await sut.methodThatWasBroken(<inputs>);

    // Assert
    expect(result).toMatchObject(<expected_shape>);
    expect(dep.someState).toBe(<expected_side_effect>);
  });
});
```
Use for: wrong data passed between layers, missing await, incorrect mapping, bad error propagation.

### E2E Test (UI/flow bug)
```
// Use MCP browser tools to reproduce the user flow that triggered the bug
// 1. Navigate to the affected page
// 2. Perform the exact sequence of actions that caused the bug
// 3. Assert the corrected behavior (screenshot evidence)
```
Use for: UI state not updating, form validation not triggering, navigation broken, visual regression.

## Edge Cases to Cover
After the main regression test, add at least one edge case test:
- Null / undefined input
- Empty string or empty array
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
