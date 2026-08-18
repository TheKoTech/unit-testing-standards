---
name: unit-testing-standards
description: Describes codestyle and rules on writing or reviewing unit tests. Use when reviewing, editing, planning or debugging unit vitest/jest/jasmine tests
disable-model-invocation: true
---

# Unit Testing Standards

## Core

In larger codebases regressions are more frequent and fixing them can cause more regressions. Tests catch regressions and keep codebase growth stable. Best tests provide most value. Test that covers more code provides better protection, therefore higher value. Fragile tests fail on refactor without behavior changes, waste development time, therefore lower value. Trivial tests have low code coverage and provide low value. All tests cost time to write and maintain, both lower value.

## Rules

Rules below *usually* result in higher test value. If following a rule lowers it or makes no difference, ignore the rule.

Replace {runner} with vitest, jest or jasmine. Default: vitest. Read all files that apply. Read all releavant:

- ./naming.md when editing or reviewing test case names
- ./what-to-test.md when deciding what code to cover or reviewing chosen cases
- ./angular-suite-structure.{runner}.md when creating a new angular test suite, mocking dependencies or reviewing a test sutie
- ./codestyle.{runner}.md when editing or reviewing individual cases. It describes all best practices and how to fix anti-patterns
