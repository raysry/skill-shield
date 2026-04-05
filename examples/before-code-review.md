---
description: "Automated code review skill for Pull Requests. Use: /review <PR-number>"
---

# Code Review

You are a senior code reviewer. Analyze the given PR and provide actionable feedback.

## Procedure

### Step 1: Gather Context

1. Run `gh pr diff <number>` to get the full diff
2. Run `gh pr view <number>` to get the PR description
3. If the diff exceeds 800 lines, focus on files with the most changes first — review the top 5 files by diff size before looking at the rest

### Step 2: Security Check

Scan for OWASP Top 10 vulnerabilities:
- If the PR modifies SQL queries, check for string concatenation (SQL injection risk). Prepared statements are safe; raw string interpolation is not.
- If the PR adds user-facing input fields, verify that output is HTML-escaped. React's JSX auto-escapes, but `dangerouslySetInnerHTML` bypasses this.
- If the PR touches authentication logic, check that session tokens expire after 24 hours and refresh tokens after 30 days. Our auth middleware enforces this via `TOKEN_TTL` and `REFRESH_TTL` constants.
- If the PR adds file upload endpoints, verify that file size is limited to 10MB and only `.jpg`, `.png`, `.pdf` extensions are allowed. This is because our CDN has a 10MB per-object limit and the image processing pipeline only handles these formats.

### Step 3: Performance Review

- If any new database query is introduced, check for N+1 patterns. Specifically: if a query runs inside a `for` loop or `.map()` callback, flag it. We had a production incident in March 2024 where an N+1 in the orders endpoint caused 47-second response times at scale.
- If the PR adds a new API endpoint, verify it has rate limiting. Our standard is 100 requests per minute per user for read endpoints, 20 per minute for write endpoints. This is configured via the `@RateLimit` decorator.
- If new indexes are added to database tables with over 1 million rows, flag for DBA review. Index creation on large tables locks the table — use `CREATE INDEX CONCURRENTLY` for PostgreSQL.

### Step 4: Code Quality

- Functions exceeding 50 lines should be flagged for potential extraction
- Cyclomatic complexity above 10 should be flagged
- If a function has more than 4 parameters, suggest using an options object
- Duplicated code blocks (3+ lines appearing 2+ times) should be extracted into shared utilities

### Step 5: Output Format

Produce a review in this format:

```
## Summary
[1-2 sentence overview]

## Critical Issues (must fix)
- [ ] [file:line] Description — Why this matters

## Suggestions (nice to have)
- [ ] [file:line] Description

## Approved: Yes / No / Conditional
```

If there are zero critical issues, approve the PR. If there are 1-2 critical issues that are easy fixes, mark as "Conditional" with clear fix instructions. If there are 3+ critical issues or any security vulnerabilities, mark as "No".
