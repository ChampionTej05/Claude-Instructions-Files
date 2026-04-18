

## PROMPT 1 — Generate `copilot-instructions.md`

> Open your main pipeline module files before running this. The more files open, the better the output.

Paste this into Copilot Chat:

---

```
@project

You are about to generate a `.github/copilot-instructions.md` file for this Java codebase.
This file will be injected into every future Copilot interaction automatically.
Its purpose is to make you a disciplined, production-safe collaborator on this project — not a fast code generator.

Before writing anything, do the following:
1. Scan the open files and identify the dominant patterns: how errors are handled, how state is managed, how classes are structured, how dependencies are injected, how concurrency is handled if present.
2. Identify what is consistent across the codebase — these are the canonical patterns to preserve.
3. Identify any areas that look inconsistent — do NOT document those as standards. Note them as areas requiring caution.

Then generate the `copilot-instructions.md` file with the following sections:

**Section 1 — How to Behave Before Writing Code**
Write a set of mandatory pre-edit steps the agent must follow before changing any file.
These should enforce: read the full class first, read the callers, read the tests, understand the pattern in use, state the plan before executing.
Do not make these generic — make them specific to what you observed in this codebase.

**Section 2 — Patterns Observed in This Codebase**
Document the actual patterns you found: error handling approach, logging style, class structure, naming conventions, state management, and any concurrency patterns present.
For each pattern, write it as a rule: "In this codebase, X is done by Y. Continue this pattern."
Do not invent patterns that are not present. Do not suggest improvements here.

**Section 3 — What You Must Not Do Without Explicit Instruction**
Based on what you observed, list the changes that would be most dangerous or disruptive to this codebase.
Think about: signature changes, exception type changes, access modifier changes, dependency additions, refactoring mixed with feature work, reformatting mixed with logic changes.
Write each as a hard rule.

**Section 4 — When to Stop and Ask**
List the conditions under which the agent must pause and ask the developer before proceeding.
Base these on the complexity and coupling you observed — not on generic best practices.

**Section 5 — Code Review Mindset**
Write a short paragraph the agent must apply when reviewing its own diff before presenting it.
It should reflect the standards you observed, not generic clean code advice.

Output only the markdown content for the file. Do not explain your reasoning outside the file.
```

---

## PROMPT 2 — Generate `git-commit-instructions.md`

> Run this separately. It is short and focused.

```
@project

Generate a `.github/git-commit-instructions.md` file for this repository.

Scan the existing git commit history if accessible, or infer from the codebase structure what kind of changes are common: bug fixes, pipeline changes, config changes, schema changes, dependency updates, refactoring.

Generate commit message instructions that:
- Enforce a consistent format (use Conventional Commits: feat, fix, refactor, chore, perf, test, docs)
- Require a scope that reflects the module or layer being changed (e.g., feat(pipeline), fix(db-writer), perf(cache))
- Prohibit vague messages ("fix bug", "update code", "changes", "WIP" as a final commit)
- Require that the body of the commit (when present) explains WHY, not what — the diff already shows what
- For any change touching concurrency, database writes, or cache logic: require a note on what was tested or verified

Output only the markdown content for the file.
```

---

## PROMPT 3 — Generate `refactor-review.prompt.md`

> This becomes a reusable slash-command you invoke before any refactor session.
> Save the output to `.github/prompts/refactor-review.prompt.md`

```
@project

Generate a `.github/prompts/refactor-review.prompt.md` file.

This prompt file will be invoked by the developer before any refactoring session.
Its job is to make the agent do a full read-and-understand pass on the target before suggesting any changes.

The prompt file should instruct the agent to:
1. Read the entire target class — not just the method in focus
2. Identify all callers and dependents of the class or method being refactored
3. List the patterns the class currently uses (error handling, state, threading, logging)
4. Identify tests that cover this class — and flag if there are none
5. Produce a written summary of: what the class does, what its contracts are, what is safe to change, and what is risky to change
6. Only after completing the above: propose the minimal refactor that achieves the goal
7. Explicitly state what it will NOT change and why

The prompt file should use the correct frontmatter format for GitHub Copilot prompt files:
---
mode: 'agent'
description: 'Read and understand a class fully before proposing any refactor'
---

Then the prompt body. Make it specific enough to enforce discipline, not so long it becomes noise.
Output only the file content.
```

---

## PROMPT 4 — Generate `code-review.prompt.md`

> Save the output to `.github/prompts/code-review.prompt.md`
> Invoke this when you want Copilot to review a class or diff before you commit.

```
@project

Generate a `.github/prompts/code-review.prompt.md` file.

This prompt will be invoked when the developer wants a structured code review of a class or a set of changes.

The review prompt should instruct the agent to evaluate the code against:
1. Consistency with the patterns observed in the rest of this codebase — not against external best practices
2. Error handling completeness: are all failure paths handled? Is anything silently swallowed?
3. Thread safety: if this code touches shared state, is the synchronization correct and consistent with the codebase's approach?
4. Correctness of resource management: connections, streams, executors — are they closed?
5. Logging adequacy: are failures logged with enough context to diagnose in production?
6. Unnecessary changes: does the diff contain any changes unrelated to the stated goal?
7. Risk surface: what is the blast radius of this change? What could break that is not obvious?

The review should produce:
- A verdict per category: PASS / WARN / FAIL
- For each WARN or FAIL: a specific observation with the line or method in question
- A final recommendation: SAFE TO COMMIT / NEEDS DISCUSSION / DO NOT COMMIT

Frontmatter:
---
mode: 'agent'
description: 'Structured code review against codebase patterns and production safety standards'
---

Output only the file content.
```

---

## After Running These Prompts

1. Review each output carefully — you will recognize immediately what looks right vs generic
2. Edit anything that does not match how the project actually works
3. Commit all four files to the repo so the whole team (and future Copilot sessions) benefits
4. For the `.prompt.md` files: in IntelliJ Copilot Chat, you can reference them manually if slash-commands are not yet fully available — just open the file and include it in the chat context

## File Structure After Generation

```
your-project/
├── .github/
│   ├── copilot-instructions.md
│   ├── git-commit-instructions.md
│   └── prompts/
│       ├── refactor-review.prompt.md
│       └── code-review.prompt.md
```