---
name: unit-testing-standards
description: Describes codestyle and rules on writing or reviewing unit tests. Use when reviewing, editing, planning or debugging unit vitest/jest/jasmine tests
disable-model-invocation: true
---

# Unit Testing Standards

## Philosophy

The goal of testing is to protect against regressions. Larger apps have more failure points. Tests ensure stable growth by protecting business logic. Good tests never break after refactoring, but always break when business logic changes. Bad tests don't protect against regressions, so bugs slip into production. Or break when business logic didn't change, which numbs the developer's attention and wastes time.

Rules below *usually* achieve the goal above. If following a rule goes against the goal or makes no difference, ignore the rule.

## Rules

Replace {runner} with vitest, jest or jasmine. Default: vitest. Read all files that apply. Each is separate to allow parallell subagents. Use:

- ./naming.md when editing and reviewing test case names
- ./what-to-test.md when deciding what code to cover or reviewing cases
- ./new-angular-suite.{runner}.md when planning a new angular test suite
- ./components.{runner}.md when editing or reviewing component tests
- ./mocking-pattern.{runner}.md when editing or reviewing suite mocks

./codestyle.{runner}.md describes all best practices and how to fix anti-patterns. Use it only when editing or reviewing individual cases
