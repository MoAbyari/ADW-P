# Application Validation Test Suite

Execute the project's validation commands, returning results in a standardized JSON format for automated processing.

## Purpose

Proactively identify and fix issues in the application before they impact users or developers. By running this comprehensive test suite, you can:
- Detect syntax errors, type mismatches, and import failures
- Identify broken tests or security vulnerabilities  
- Verify build processes and dependencies
- Ensure the application is in a healthy state

## Variables

TEST_COMMAND_TIMEOUT: 5 minutes

## Instructions

- Read the `Validation Commands` section of `.claude/adw_project.md` and execute each test in the sequence listed there
- Capture the result (passed/failed) and any error messages
- IMPORTANT: Return ONLY the JSON array with test results
  - IMPORTANT: Do not include any additional text, explanations, or markdown formatting
  - We'll immediately run JSON.parse() on the output, so make sure it's valid JSON
- If a test passes, omit the error field
- If a test fails, include the error message in the error field
- Execute all tests even if some fail
- Error Handling:
  - If a command returns non-zero exit code, mark as failed and immediately stop processing tests
  - Capture stderr output for error field
  - Timeout commands after `TEST_COMMAND_TIMEOUT`
  - IMPORTANT: If a test fails, stop processing tests and return the results thus far
- Test execution order is important - dependencies should be validated first
- All file paths are relative to the project root
- Always run `pwd` and `cd` before each test to ensure you're operating in the correct directory for the given test

## Test Execution Sequence

The tests are defined in the `Validation Commands` section of `.claude/adw_project.md`.
For each entry listed there:

- test_name: the entry's name
- Command: the entry's command, run from the project root
- test_purpose: the entry's purpose

If that section lists no commands, return an empty JSON array: `[]`

## Report

- IMPORTANT: Return results exclusively as a JSON array based on the `Output Structure` section below.
- Sort the JSON array with failed tests (passed: false) at the top
- Include all tests in the output, both passed and failed
- The execution_command field should contain the exact command that can be run to reproduce the test
- This allows subsequent agents to quickly identify and resolve errors

### Output Structure

```json
[
  {
    "test_name": "string",
    "passed": boolean,
    "execution_command": "string",
    "test_purpose": "string",
    "error": "optional string"
  },
  ...
]
```

### Example Output

```json
[
  {
    "test_name": "unit_tests",
    "passed": false,
    "execution_command": "npm test",
    "test_purpose": "Validates application behaviour by running the unit test suite",
    "error": "AssertionError: expected 3 to equal 4"
  },
  {
    "test_name": "lint",
    "passed": true,
    "execution_command": "npm run lint",
    "test_purpose": "Validates code quality and identifies style violations and potential bugs"
  }
]
```