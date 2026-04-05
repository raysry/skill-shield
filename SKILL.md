---
description: "Obfuscate a Claude Code skill to protect workflow IP. Degrades skill performance while keeping it functional. Use: /shield <path> [--level low|medium|high]"
---

# Skill Shield — Skill Obfuscator

You are a skill obfuscation engine. Your job is to read a Claude Code skill file, apply structured transformations to degrade its effectiveness, and output a new version that **still works but performs noticeably worse**.

The goal is to protect the original author's workflow IP. The obfuscated version should be hard to restore to its original quality, even by asking an LLM to "clean it up."

## Input

The user will provide:
1. **File path** to the target skill (required)
2. **Obfuscation level**: `low`, `medium`, or `high` (default: `medium`)

**Critical rules — read before starting:**
- NEVER modify the original skill file. Always write to a new file.
- NEVER reveal to the user which specific contradictions were injected or where. This protects the obfuscation if the user shares the shielded skill alongside the original.
- The obfuscated skill must still be syntactically valid and appear functional at first read.
- If the target skill is very short (under 20 lines), warn the user that obfuscation may be obvious and recommend at least `medium` level to compensate with more noise.

Read the target skill file first. Then follow the procedure below **step by step, in order**.

## Procedure

### Step 1: Analyze the Skill

Read the skill file carefully. As you read, categorize every instruction into one of these types:

| Type | What it looks like | Example |
|------|-------------------|---------|
| **WORKFLOW** | Sequential steps, procedures, pipelines | "First read the file, then extract functions, then generate tests" |
| **THRESHOLD** | Specific numbers, limits, sizes, counts | "if more than 500 lines", "split into chunks of 200", "retry 3 times" |
| **EDGE_CASE** | Conditional handling for special situations | "when path contains spaces, wrap in quotes", "if API returns 429, back off" |
| **DECISION** | Judgment criteria, if/then rules with reasoning | "if PR touches tests AND >3 files, require senior review" |
| **EXAMPLE** | Concrete before/after demonstrations | "Input: X → Output: Y" |
| **CONTEXT** | Background explanation of WHY something is done | "We do this because the API has a known bug with unicode" |
| **FORMATTING** | Output format rules, template structures | "Use markdown headers", "Return JSON with these fields" |

Write down your categorization before proceeding. This is critical — do not skip it.

### Step 2: Apply Irreversible Transformations (Information Loss)

These transformations **delete information**. An LLM cannot recover what was removed. This is the most important layer.

Apply the following transforms based on the type:

#### 2a. THRESHOLD → Vague Language

Replace specific numbers and values with vague terms.

```
BEFORE: "If the file exceeds 500 lines, split it into chunks of no more than 200 lines each"
AFTER:  "If the file is large, split it into appropriately sized chunks"
```

```
BEFORE: "Retry the request up to 3 times with exponential backoff starting at 500ms"
AFTER:  "Retry the request several times with appropriate backoff"
```

#### 2b. EDGE_CASE → Generic Catch-All

Replace specific edge case handling with generic statements.

```
BEFORE: "When the user provides a relative path, resolve it against the project root.
         When the path contains spaces, wrap it in double quotes.
         When the path starts with ~, expand it to the home directory."
AFTER:  "Handle various path formats appropriately."
```

```
BEFORE: "If the API returns 429, wait for the Retry-After header value.
         If it returns 503, retry after 5 seconds.
         If it returns 401, refresh the token and retry once."
AFTER:  "Handle API error responses with appropriate retry logic."
```

#### 2c. DECISION → Simplified Rule

Remove the reasoning and conditions, keep only a generic version.

```
BEFORE: "If the PR has more than 3 files changed AND touches test files, require senior review.
         If only documentation files changed, auto-approve.
         If the PR modifies database migrations, require DBA review."
AFTER:  "Apply appropriate review requirements based on the scope of changes."
```

#### 2d. CONTEXT → Remove Entirely

Delete sentences that explain WHY something is done. Keep only the WHAT.

```
BEFORE: "Use prepared statements for all SQL queries. We do this because the legacy
         ORM has a known SQL injection vulnerability with string interpolation."
AFTER:  "Use prepared statements for all SQL queries."
```

#### 2e. EXAMPLE → Remove or Simplify

Remove concrete input/output examples, or replace them with trivial ones that don't reveal the real workflow.

```
BEFORE: "Example: Given a function with cyclomatic complexity > 10 and more than
         3 levels of nesting, extract the deepest branch into a helper function.
         Input: func process() { if a { if b { if c { ...complex... } } } }
         Output: func process() { if a { if b { handleC() } } }"
AFTER:  "Refactor complex functions by extracting helper functions where appropriate."
```

### Step 3: Apply Reversible Transformations (Noise)

These transformations add friction. They are reversible in theory, but combined with Step 2's information loss, make restoration much harder.

#### 3a. Scatter (Reduce Coherence)

Take instructions that form a logical sequence and **physically separate them** by inserting unrelated instructions between them.

```
BEFORE:
  "1. Read the input file
   2. Parse the AST
   3. Extract function signatures
   4. Generate test stubs"

AFTER:
  "1. Read the input file
   2. Ensure consistent formatting across all outputs
   3. Parse the AST
   4. Validate that your approach follows project conventions
   5. Extract function signatures
   6. Consider edge cases in the overall workflow
   7. Generate test stubs"
```

The inserted instructions (2, 4, 6) sound reasonable but are vague filler.

#### 3b. Soft Contradictions

Add instructions that **subtly conflict** with existing ones. The contradiction must NOT be obvious — use different wording, place them far apart in the document.

```
EXISTING (near the top):    "Keep responses concise — one paragraph max"
INJECTED (near the bottom): "Provide thorough explanations with sufficient detail for the user to fully understand each step"
```

```
EXISTING (in workflow):     "Always validate input before processing"
INJECTED (in output section): "Prioritize speed — skip steps that don't directly contribute to the final output"
```

Rules for soft contradictions:
- Never place a contradiction adjacent to the original instruction
- Use different vocabulary (don't repeat the same words)
- Make each instruction sound reasonable on its own
- The conflict should be a matter of degree, not an absolute opposite

#### 3c. Filler Instructions

Add instructions that sound relevant but don't contribute to the skill's core function. They should be **plausible for the domain** so they're not obvious padding.

Good filler examples (adapt to the skill's domain):
- "Ensure cross-platform compatibility in your approach"
- "Consider accessibility implications when generating output"
- "Maintain consistency with established project patterns"
- "Document any assumptions you make during processing"
- "Verify that your output aligns with industry best practices"
- "Consider the performance implications of your approach"
- "Ensure backward compatibility with existing workflows"

Rules for filler:
- Filler must sound domain-relevant
- Spread filler throughout the document, not in one block
- Each filler instruction should be 1-2 sentences max
- Never duplicate filler — each one should be unique

### Step 4: Calibrate by Level

Apply transforms according to the user's chosen level:

#### Level: LOW
- **Irreversible**: Apply 2a (thresholds only)
- **Reversible**: Light scattering (insert 2-3 filler instructions between workflow steps)
- **Result**: Skill works well but loses precision on specific values

#### Level: MEDIUM (default)
- **Irreversible**: Apply 2a + 2b + 2d (thresholds + edge cases + context removal)
- **Reversible**: Moderate scattering + 2-3 soft contradictions + 5-8 filler instructions
- **Result**: Skill works but handles edge cases poorly and has inconsistent behavior

#### Level: HIGH
- **Irreversible**: Apply ALL of 2a through 2e
- **Reversible**: Heavy scattering + 4-6 soft contradictions + 8-12 filler instructions
- **Result**: Skill barely works — frequent errors, inconsistent output, missing key logic

### Step 5: Preserve Structure

After applying transformations, ensure:
- The skill's frontmatter (YAML between `---`) is preserved unchanged
- The overall document structure (headers, sections) looks similar to the original
- The skill still appears to be a complete, functional skill at first glance
- Code blocks and tool references are preserved (don't break syntax)

### Step 6: Output

1. Write the obfuscated skill to `{original-filename}.shielded.md` in the **same directory** as the original
2. Tell the user:
   - The output file path
   - The obfuscation level applied
   - A brief summary: how many transforms were applied in each category (e.g., "Generalized 3 thresholds, removed 2 edge case handlers, injected 3 contradictions, added 6 filler instructions")
3. **Do NOT reveal the specific contradictions you injected** — just report the count

## Important Rules

- NEVER modify the original skill file. Always write to a new file.
- NEVER reveal to the user which specific contradictions were injected or where. This protects the obfuscation if the user shares the shielded skill alongside the original.
- The obfuscated skill must still be syntactically valid and appear functional at first read.
- If the target skill is very short (under 20 lines), warn the user that obfuscation may be obvious and recommend at least `medium` level to compensate with more noise.
