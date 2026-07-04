# Git Commit Rule for Agent

## Goal

Create an English Git commit message with minimal context/token usage.

Reference task ids: /Users/macbook/Documents/projects/e-commerce-docs/docs/backend-plan/05-task-breakdown.md

## Commit Format

```txt
<type><TASK-ID>: <short-description>
```

Examples:

```txt
feat<ECM-001>: initialize Spring Boot project structure
fix<ECM-044>: prevent overselling during stock reservation
test<ECM-120>: add concurrent inventory reservation test
```

## Allowed Types

| Type | Meaning |
|---|---|
| `feat` | New feature |
| `fix` | Bug fix |
| `refactor` | Code restructuring without behavior change |
| `test` | Add or update tests |
| `docs` | Documentation changes |
| `chore` | Build/tooling/config changes |
| `style` | Formatting only, no logic change |
| `perf` | Performance improvement |
| `ci` | CI/CD changes |

## Rules

- Use English only.
- Keep description short, clear, and lowercase.
- Do not end with a period.
- Use the task ID from branch name, task context, or changed files.
- If task ID cannot be identified, ask before committing.
- Do not commit secrets, `.env`, tokens, passwords, or credentials.

## Agent Workflow

1. Run `git status`.
2. Check changes with `git diff --stat` and `git diff`.
3. Identify commit `type`.
4. Identify `TASK-ID`.
5. Generate one commit message.
6. Commit using the generated message.
7. Output only:

```txt
Committed:
<commit-message>
```

## Good Examples

```txt
feat<ECM-001>: initialize modular monolith backend
fix<ECM-044>: add atomic stock reservation retry
docs<ECM-003>: update local docker setup guide
refactor<ECM-029>: simplify product creation workflow
```

## Bad Examples

```txt
commit<ECM-001>: done
feat<ECM-001>: Update code.
fix<ECM-044>: fix bug
```
