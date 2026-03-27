---
name: git-commit-pro
description: Write clear, conventional commit messages and manage git workflows. Use when committing code, writing PR descriptions, managing branches, or structuring commit history. Follows conventional commits spec.
license: Apache-2.0
metadata:
  original-author: terminal-skills
  original-repo: TerminalSkills/skills
  vendored-by: good-stories-llc
  version: "1.0"
---

# Git Commit Pro

Write clear, consistent commit messages following the Conventional Commits specification. Manage branches, structure PR descriptions, and maintain a clean git history.

## When to Use

- User asks to commit code with a good message
- User wants to write a PR description
- User needs help with branch naming or git workflow
- User wants to clean up commit history

## Conventional Commits Format

Every commit message follows this structure:

```
<type>(<scope>): <description>

[optional body]

[optional footer(s)]
```

### Type (required)

| Type | When to Use |
|---|---|
| `feat` | A new feature or capability |
| `fix` | A bug fix |
| `refactor` | Code change that neither fixes a bug nor adds a feature |
| `docs` | Documentation only changes |
| `test` | Adding or updating tests |
| `chore` | Build, tooling, CI, dependency updates |
| `style` | Formatting, whitespace, semicolons (no logic change) |
| `perf` | Performance improvement |
| `ci` | CI/CD configuration changes |
| `revert` | Reverting a previous commit |

### Scope (optional)

A noun describing the section of the codebase:

```
feat(auth): add OAuth2 login flow
fix(api): handle null response from payment gateway
refactor(db): extract query builder into separate module
```

### Description (required)

- Use imperative mood: "add", "fix", "change" (not "added", "fixes", "changed")
- Lowercase first letter
- No period at the end
- Under 72 characters
- Describe what the commit does, not what you did

Good:
```
feat(cart): add quantity validation before checkout
fix(auth): prevent token refresh race condition
refactor(api): extract rate limiter into middleware
```

Bad:
```
updated stuff
fix bug
WIP
asdf
Fixed the thing where users couldn't log in sometimes
```

### Body (optional)

Use the body to explain **why** the change was made, not **what** changed (the diff shows what):

```
fix(payments): retry failed webhook deliveries

Webhook deliveries were silently failing when the target server
returned a 503. This caused missed payment notifications for
roughly 2% of transactions.

Added exponential backoff retry (3 attempts, 1s/5s/30s delays)
and a dead letter queue for permanently failed deliveries.
```

### Footer (optional)

```
BREAKING CHANGE: rename `userId` field to `user_id` in API response

Refs: #1234
Closes: #5678
```

## Commit Message Examples by Type

### feat

```
feat(dashboard): add real-time notification counter

Displays unread notification count in the header badge.
Counter updates via WebSocket without page refresh.
```

### fix

```
fix(search): return empty results instead of 500 for special characters

The search endpoint threw an unhandled regex error when queries
contained brackets or backslashes. Now escapes special characters
before passing to the search engine.
```

### refactor

```
refactor(auth): consolidate token validation into single middleware

Three separate endpoints had duplicate token validation logic.
Extracted into a shared middleware that handles validation,
refresh, and error responses consistently.
```

### chore

```
chore(deps): upgrade fastapi from 0.104 to 0.110

No breaking changes. Picks up the security fix for multipart
form parsing (CVE-2024-XXXXX).
```

## Branch Naming Conventions

```
<type>/<ticket-id>-<short-description>
```

Examples:
```
feat/PROJ-123-oauth-login
fix/PROJ-456-null-payment-response
refactor/PROJ-789-extract-rate-limiter
chore/update-dependencies
docs/api-authentication-guide
```

Rules:
- All lowercase
- Hyphens between words (no underscores, no spaces)
- Include ticket ID when one exists
- Keep under 50 characters when possible

## PR Description Template

```markdown
## Summary

[1-3 sentences: what this PR does and why]

## Changes

- [Bullet list of specific changes]
- [Group by file or feature area if many changes]

## Testing

- [ ] Unit tests added/updated
- [ ] Integration tests pass
- [ ] Manual testing performed: [describe what you tested]

## Screenshots

[If UI changes, include before/after screenshots]

## Notes for Reviewers

[Any context that helps review: tradeoffs made, alternatives considered,
areas where you want specific feedback]
```

## Workflow: Creating a Good Commit

### 1. Review Changes

Before committing, always review what will be committed:

```bash
git status                    # See which files changed
git diff                      # See unstaged changes
git diff --staged             # See staged changes
```

### 2. Stage Intentionally

Stage files that belong together in one logical change:

```bash
git add src/auth/middleware.py src/auth/tokens.py tests/test_auth.py
```

Avoid `git add .` unless every changed file belongs in the same commit.

### 3. Write the Message

Use the type/scope/description format. Add a body if the change needs explanation:

```bash
git commit -m "feat(auth): add token refresh middleware

Automatically refreshes expired access tokens using the refresh
token cookie. Falls back to login redirect if refresh fails."
```

### 4. One Commit Per Logical Change

Each commit should represent one complete, working change:

- Adding a feature: one commit for the feature + its tests
- Fixing a bug: one commit for the fix + its test
- Refactoring: one commit per refactoring step (not one giant commit)

Do not mix unrelated changes in one commit.

## Workflow: Structuring History Before PR

If you have messy work-in-progress commits, clean them up before opening a PR:

```bash
# Interactive rebase to squash/reorder last N commits
git rebase -i HEAD~5
```

Common operations during interactive rebase:
- `squash` (s): Merge this commit into the previous one
- `reword` (r): Change the commit message
- `drop` (d): Remove the commit entirely
- `edit` (e): Pause to amend the commit

Target: a PR with 1-5 clean, logical commits that tell the story of the change.

## Breaking Changes

When a commit introduces a breaking API change:

```
feat(api)!: rename user endpoints to follow REST conventions

BREAKING CHANGE: All `/user/*` endpoints are now `/users/*`.
The old endpoints return 301 redirects for 90 days, then will
be removed.

Migration guide:
- /user/profile -> /users/me
- /user/:id -> /users/:id
- /user/list -> /users
```

The `!` after the scope and the `BREAKING CHANGE:` footer both signal a breaking change.

## Commit Message Anti-Patterns

| Anti-Pattern | Why It Hurts | Better Alternative |
|---|---|---|
| "fix bug" | Which bug? Where? | "fix(cart): prevent negative quantity on update" |
| "WIP" | Not a finished thought | Squash WIP commits before PR |
| "address review feedback" | Meaningless in history | Squash into the original commit |
| "misc changes" | Impossible to search | Split into separate, typed commits |
| 200-char subject line | Truncated in git log | Keep under 72 chars; use body for detail |
