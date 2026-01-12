## Operational Notes on Using Cursor Effectively

These are practical notes from using Cursor daily on real production codebases.
They assume you already know how to design systems and review code.

Cursor is treated here as an execution engine, not a decision-maker.

---

## 1. Core Rule: Never Let Cursor Guess

If Cursor has to infer intent, the output will be confident and wrong.

### Don’t 
```
Add caching to this service.
```
### Do
```
Implement read-through caching using Redis.

* Cache key format: user:{userId}
* TTL: read from @cache-config.ts
* On cache miss: fetch from DB and populate cache
* On Redis failure: bypass cache and continue
* Do not change the public interface

```

---

## 2. Prompt Structure (Non-Negotiable)

Every non-trivial prompt should follow a fixed structure.

```
GOAL:
CONSTRAINTS:
REFERENCES:
NON-GOALS:
OUTPUT FORMAT:
```

### Don't
```
Make this API faster.
```

### Do
```

GOAL:
Reduce average latency of `getUserData`.

CONSTRAINTS:

* Backward compatible
* No new dependencies
* No interface changes

REFERENCES:

* @getUserData.ts
* @cache-config.ts

NON-GOALS:
* No auth changes
* No schema changes

OUTPUT FORMAT:
* Unified diff only
* No explanations
```

---

## 3. Be Deterministic, Not Suggestive
Remove all decision-making from the assistant.

### Don't
```
Improve error handling.
```
### Do
```
Handle the following error cases explicitly:

* Redis timeout → log warning and continue
* User not found → return 404
* DB failure → return 500

Do not introduce retries.

```

---

## 4. Use `.cursorrules` as Architectural Law

Rules prevent slow decay and inconsistency.

### Don't
- Rely on “remembering” preferences
- Repeat the same constraints in every prompt

### Do

**`.cursorrules`**
```

Language: TypeScript (strict)
Frontend: Next.js App Router
Backend: AWS Lambda (AWS SAM)
IaC: AWS CDK (Python only)

Testing:

* Jest only
* No snapshot tests

Architecture:

* One CDK stack per service
* No cross-stack imports
* Config must come from config layer

Anti-patterns:

* No global state
* No hardcoded env vars
* No inline IAM policies

```

---

## 5. Control Context Explicitly
Cursor does not manage context for you.
### Don't
- Keep a session open for hours
- Let unrelated files stay in context
- Argue with hallucinated output

### Do
- Restart the session
- Remove unrelated files
- Re-run with a clean prompt

**Rule of thumb:**  
If you feel friction, reset immediately.

---

## 6. Index External Documentation
Model memory is not a source of truth.

### Don't
```
Use the latest Firebase auth patterns.

```

### Do
- Index official Firebase Auth docs
- Index Next.js App Router docs
- Then prompt against indexed sources
**Result:** No deprecated APIs, fewer rewrites.

---

## 7. Enforce Output Discipline
Verbosity is a liability.

### Don't
```
Explain the changes and update the file.

```

### Do
```

OUTPUT RULES:

* Unified diff only
* No comments unless required
* Do not reformat unrelated code
* No explanations

```

---

## 8. Always Ask for Diffs

Full rewrites hide bugs.

### Don't
```

Rewrite this file to add caching.

```

### Do
```

Modify only the required sections.
Output unified diff.
Do not touch unrelated logic.

```

---

## 9. Tests Before Implementation

Tests clarify intent better than prose.

### Don't
```

Add tests after implementing caching.

```

### Do
```

GOAL:
Generate Jest tests for `getUserData`.

COVER:

* Cache hit
* Cache miss
* Redis failure fallback

REFERENCES:

* @getUserData.ts

OUTPUT:

* Test file only
* No implementation changes

```

If tests can’t be generated, the requirements are incomplete.

---

## 10. Model Selection Rules

Large models are not default.

### Don't
- Use the biggest model for CSS
- Use reasoning models for boilerplate

### Do
- Large models → architecture, refactors
- Small models → boilerplate, types, CSS

Execution ≠ thinking.

---

## 11. Credit Management

Most credit burn is self-inflicted.

### Don't
- Prompt reactively
- Fix output in multiple passes
- Debug hallucinations with more prompts

### Do
- Write a Markdown plan first
- Use it as shared context
- Send one complete prompt

Good prompts cost less than retries.

---

## 12. Common Failure Modes

### Over-Trust

Don't Accept correct-looking logic  
Always Review like a junior engineers PR

---

### Style Drift

Let patterns change slowly  
Enforce via prompt or restart session if style drifts

---

### Autopilot Usage

Don't “Accept all”  
Read every diff

If you wouldn’t merge it from a junior, don’t merge it from an LLM.

---

## Final Notes
Cursor does not make engineers productive.
Clear intent does.
The role is shifting from:

**writing code → defining constraints and editing logic**

Everything else is noise.

Follow me on [Linkedin](https://www.linkedin.com/in/m-ali-imran/) & [X](https://x.com/malii2k)
