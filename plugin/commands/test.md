---
name: test
description: >
  Run a QA test against a web application using your Chrome browser.
  Delegates to the test-runner agent which plans scenarios, executes them
  in your real Chrome (via Claude in Chrome), captures network + console
  telemetry at every step, and produces a unified diagnostic report.
  
  Usage: /chrome-test-runner:test [URL and description of what to test]
  
  Examples:
    /chrome-test-runner:test stress test https://myapp.vercel.app — send lots of messages, create child chats
    /chrome-test-runner:test https://localhost:3000 — verify the login flow and dashboard load correctly  
    /chrome-test-runner:test click around the app and find bugs

  Requires: Claude in Chrome extension (run with claude --chrome)
---

Use the test-runner agent to execute a QA test run.

The user wants to test: $ARGUMENTS

Instructions for the test-runner agent:
1. Parse the request to identify the target URL and test objectives
2. Create a test plan with numbered scenarios
3. Create a new Chrome tab for testing
4. Execute each scenario using the flush-act-capture cycle described in your system prompt
5. Write rich reasoning BEFORE every action explaining what you expect
6. After every action, capture network + console telemetry and assess the result
7. Flag any bugs with severity, expected vs actual behavior, and repro steps
8. At the end, compile the unified log and write it to a markdown file
9. Return a concise summary: steps executed, bugs found, key findings
