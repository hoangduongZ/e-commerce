# Git Commit Rule for Agent

## Goal

Create a clean English Git commit with minimal context/token usage.

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

## Core Rules

- Use English only.
- Keep description short, clear, and lowercase.
- Do not end with a period.
- Use the task ID from branch name, task context, or changed files.
- If task ID cannot be identified, ask before committing.
- Do not commit secrets, `.env`, tokens, passwords, or credentials.

## File Grouping Rule

Before committing, check whether the workspace contains changes from multiple tasks.

Do **not** commit all changes blindly.

Avoid:

```bash
git add .
```

unless all changed files clearly belong to the same task.

Group files by:

- task ID
- module/domain
- feature/bug scope
- related test files
- related migration/config/docs files

If changes belong to multiple unrelated tasks, create separate commits.

Example:

```txt
catalog files + catalog tests + product migration
=> one catalog-related commit

order files + payment files + unrelated README update
=> split into separate commits or ask before committing
```

If unsure whether files belong together, ask before staging.

## Agent Workflow

1. Run `git status`.
2. Check changed files with `git diff --stat` and `git diff --name-only`.
3. Group files by related task/scope.
4. Stage only files related to the current task.
5. Never use `git add .` if unrelated changes exist.
6. Check staged files with `git diff --cached --stat`.
7. Identify commit `type`.
8. Identify `TASK-ID`.
9. Generate one commit message.
10. Commit using the generated message.
11. Output only:

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
feat<ECM-029>: update many files
```
