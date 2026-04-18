# CLAUDE.md — Java Project Agent Guidelines

These guidelines govern how you (the AI agent) should think and behave when working on this codebase.
The goal is: **do less, do it right, leave the codebase better than you found it — not just different.**

---

## Prime Directive

> **Understand before you change. Change only what is necessary. Justify every deviation from the existing style.**

You are working on a production Java codebase. Every edit has a blast radius. A change that looks like an improvement in isolation may break a thread contract, invalidate a performance assumption, or violate a pattern established for a reason that is not visible in the immediate context.

When in doubt, **do not change**. Ask instead.

---

## Before You Write a Single Line of Code

Do all of the following before touching any file:

1. **Read the affected class fully** — not just the method you are targeting. The whole class.
2. **Read the callers** — understand how the method or class is used upstream.
3. **Read the tests** — if tests exist, they document expected behavior, sometimes better than comments.
4. **Identify the pattern in use** — how is error handling done in this class? Logging? State management? Inheritance vs composition? Match it.
5. **Ask: is this change actually necessary?** — many requests can be solved with a smaller edit than first assumed.
6. **State your plan before executing it** — for non-trivial changes, describe what you are going to do and why, then wait for confirmation.

---

## The Four Questions Before Every Edit

Before writing any code, answer these:

1. What is the existing pattern here, and am I continuing it?
2. What is the minimum change that solves the problem?
3. What could break from this change that is not immediately obvious?
4. Is there a reason this code is written the way it is that I do not yet understand?

If you cannot answer question 4 confidently, investigate further before proceeding.

---

## Code Quality Standards

### Clarity over Cleverness
- Write code the next engineer can read without needing your explanation
- Prefer explicit over implicit
- Avoid compressing logic into one-liners at the cost of readability
- Method names should describe *what*, not *how*

### Method Design
- A method should do one thing
- If you find yourself writing `// Step 1` and `// Step 2` inside a single method body, that is a signal to split
- Prefer early returns to deeply nested conditionals
- Keep methods short enough to read without scrolling — but do not split artificially just to hit a line count

### State and Side Effects
- Minimize mutable state; make it explicit and visible when it must exist
- Side effects must be obvious from the method signature or name — a `get*` method should not write to anything
- Do not introduce shared mutable state without first understanding the threading model of the class you are in

### Defensive Programming
- Validate inputs at the entry boundary of the module, not deep inside it
- Never trust external data (streams, queues, network, files) without validation
- Fail fast and loudly on unexpected state — a clear exception beats silent corruption
- Understand the null contract of the codebase before adding null checks or wrapping things in `Optional`

---

## Error Handling Principles

- Every exception must be handled or explicitly propagated — never silently swallowed
- Catch specific exceptions, not `Exception` or `Throwable`, unless there is a documented reason
- Log at the right level: action-required failures = `ERROR`, notable but non-critical = `WARN`, developer detail = `DEBUG`
- Every `catch` block needs a comment explaining the recovery strategy — "log and continue" is not a strategy, it is a decision that needs justification
- Distinguish between recoverable and unrecoverable failures and treat them differently in code

---

## Logging Standards

- Use whichever logging framework is already present — do not introduce a new one
- Log messages should include enough context to diagnose the issue without a debugger: IDs, record counts, states, keys
- Do not log sensitive data
- Guard `DEBUG` statements inside loops or high-frequency paths: `if (log.isDebugEnabled())`
- Logging is not a substitute for readable code

---

## Concurrency Rules

- Before touching any code in a multithreaded context, read and understand the threading model of that class
- Do not introduce new shared state without understanding the synchronization strategy already in use
- Do not change thread pool sizes, queue bounds, or executor settings without understanding the throughput and backpressure implications
- If a class carries comments about thread safety, treat them as binding contracts
- If the codebase has an established concurrency pattern, extend it — do not introduce a new one without explicit discussion

---

## Performance Rules

- Do not optimize prematurely, but do not introduce obvious inefficiency either
- Before changing a data structure or algorithm, understand why the current one was chosen — it may reflect memory constraints, external system limits, or measured access patterns
- N+1 access patterns (database, cache, or otherwise) are always wrong — batch or preload
- Any change to sizes, counts, thresholds, or timeouts must come from configuration — never hardcode these values
- Loading large volumes of data into memory must be intentional and bounded — always ask why if you see it, and never add it without justification

---

## Dependency and Import Discipline

- Do not add a new library to solve a problem the standard library or an already-present dependency can solve
- If you add a dependency, state what it is, why it is needed, and what it replaces or avoids
- Do not upgrade dependency versions unless that is the explicit task
- Remove unused imports — do not leave them in "just in case"
- Do not introduce frameworks, paradigms, or architectural patterns (reactive, event-driven, functional) that are not already present in the module

---

## Refactoring Rules

Refactoring is a separate activity from feature work. Do not mix them in the same change.

- If you notice something worth refactoring while doing another task, note it as a suggestion — do not refactor inline
- A refactor must be behavior-preserving by definition — if behavior changes, it is not a refactor, it is a feature or a fix
- Before refactoring a class, check: is it tested? If not, say so — refactoring untested code removes the safety net
- Do not rename widely-used identifiers without IDE-level verification of all usages
- Do not reformat code in the same change as logic edits — it destroys diff readability and hides what actually changed

---

## What You Must Not Do Without Explicit Instruction

- Do not delete code — raise it for discussion, or comment it out with a clear reason attached
- Do not change method signatures that appear in interfaces or have many callers
- Do not change the types of exceptions being thrown — callers may catch specific types
- Do not change log message wording on lines that are likely used in alerting or monitoring
- Do not replace working, tested code with an alternative you prefer — even if it is objectively better by some metric
- Do not add annotations (`@Deprecated`, `@SuppressWarnings`, etc.) without a documented reason
- Do not widen access modifiers (`private` → `protected` → `public`) without understanding the class design intent
- Do not make cosmetic changes (reordering imports, reformatting blocks, renaming variables to your preference) — they add noise to the diff with zero functional value

---

## When You Are Unsure

Say so. Explicitly. Examples:

- "I am not certain what the threading contract is here — can you confirm before I proceed?"
- "This change would affect N callers — do you want me to update all of them, or just this one?"
- "I see two valid approaches here. Here are the tradeoffs: [A vs B]. Which do you prefer?"
- "I noticed something unrelated while reading this class — [describe it]. Should I address it as a separate task?"

Guessing is worse than asking. A wrong assumption in a production codebase is a bug waiting to be found.

---

## Code Review Mindset

Before submitting any change, re-read your own diff as if you are a senior engineer who:

- Does not know the AI suggested it
- Will reject anything that looks like an unnecessary change
- Will reject style inconsistency with the surrounding code
- Will ask "why was this changed?" for every modified line

If you cannot justify a changed line in one sentence, reconsider it.

---

## Summary: The Hierarchy of Actions

When given a task, work through this sequence — in order:

1. **Read** — understand the full context before forming any opinion
2. **Plan** — state what you intend to change and why
3. **Minimize** — reduce scope to the smallest correct solution
4. **Match** — adopt the existing style, patterns, and conventions of the file you are in
5. **Execute** — make the change
6. **Review** — read your own diff as the reviewer, not the author