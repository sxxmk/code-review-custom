---
name: code-review-custom
description: Self-contained two-axis code review — no other skill or setup required. Reviews the changes since a fixed point (commit, branch, tag, or merge-base) on two independent axes, Standards (does the code follow this repo's conventions?) and Spec (does it implement the originating issue/PRD — or, absent one, the intent stated in the commit messages?), running both as parallel sub-agents. Use when the user asks to review a branch, a PR, or work-in-progress changes, or says "review since X".
---

Review the diff between `HEAD` and a fixed point the user names, on two axes:

- **Standards** — does the code conform to this repo's documented conventions?
- **Spec** — does the code faithfully implement the originating issue / PRD?

Each axis runs as its own **parallel sub-agent** so neither review pollutes the other's context; this skill then reports their findings side by side. Self-contained: no other skill, setup command, or pre-configured tracker file is required.

## 1. Pin the fixed point

Use the fixed point the user gave — a commit SHA, branch, tag, `main`, `HEAD~5`, etc. If they didn't give one, ask.

Capture once:

- `git diff <fixed-point>...HEAD` — three-dot, so the comparison is against the merge-base.
- `git log <fixed-point>..HEAD --oneline`

Before going further, confirm the ref resolves (`git rev-parse <fixed-point> --`) and the diff is non-empty. A bad ref or empty diff fails here — not inside two parallel sub-agents. If the fixed point is a PR, use `gh pr diff <n>` instead. Run these from the repo root; if this isn't a git checkout, say so and stop.

## 2. Find the spec

Try these in order, stopping at the first hit:

1. **Issue references in commit messages** — `#123`, `Closes #45`, GitLab `!67`. Fetch with the tracker's CLI: `gh issue view <n> --comments` or `glab issue view <n> --comments`.
2. **The branch's PR** — `gh pr view --json title,body,comments` (or `glab mr view`).
3. **A path the user passed**, else a spec or PRD file under `docs/`, `specs/`, `prd/`, or `.scratch/` that matches the branch or feature name.
4. **A spec the user pasted into the conversation.**
5. **The commit messages themselves** — read them with `git log <fixed-point>..HEAD --format='%h %s%n%b'` and treat each message as a claim about what the change is meant to do. This is a *weak* spec: it is the author's own account, so it catches work the messages promise that the diff doesn't deliver, and diff behaviour the messages never asked for, but it cannot catch a wrong plan. State in the report that the Spec axis was judged against commit messages.
6. **Ask the user** where the spec is.

If the user explicitly names the source (a path, an issue, or "the commit messages"), use that instead of searching. If items 1–5 all come up empty — no refs, no PR, no file, no pasted spec, and messages that say nothing verifiable ("wip", "fix stuff") — run only the Standards axis and say so.

## 3. Find the standards

Collect whatever the repo documents about how code should be written — `CONTRIBUTING.md`, `CODING_STANDARDS.md`, `AGENTS.md`, `CLAUDE.md`, linter and formatter configs, and the like.

On top of that, the Standards axis always carries the **smell baseline** below — a fixed set of Fowler code smells (_Refactoring_, ch. 3) that applies even when the repo documents nothing. Two rules bind it:

- **The repo overrides.** A documented repo standard always wins; where it endorses something the baseline would flag, suppress the smell.
- **Always a judgement call.** Each smell is a labelled heuristic, never a hard violation — and skip anything tooling already enforces.

Each smell reads *what it is* → *how to fix*; match it against the diff:

- **Mysterious Name** — a function, variable, or type whose name doesn't reveal what it does or holds. → rename it; if no honest name comes, the design's murky.
- **Duplicated Code** — the same logic shape appears in more than one hunk or file in the change. → extract the shared shape, call it from both.
- **Feature Envy** — a method that reaches into another object's data more than its own. → move the method onto the data it envies.
- **Data Clumps** — the same few fields or params keep travelling together. → bundle them into one type, pass that.
- **Primitive Obsession** — a primitive or string standing in for a domain concept that deserves its own type. → give the concept its own small type.
- **Repeated Switches** — the same `switch`/`if`-cascade on the same type recurs across the change. → replace with polymorphism, or one map both sites share.
- **Shotgun Surgery** — one logical change forces scattered edits across many files in the diff. → gather what changes together into one module.
- **Divergent Change** — one file or module is edited for several unrelated reasons. → split so each module changes for one reason.
- **Speculative Generality** — abstraction, parameters, or hooks added for needs the spec doesn't have. → delete it; inline back until a real need shows.
- **Message Chains** — long `a.b().c().d()` navigation the caller shouldn't depend on. → hide the walk behind one method on the first object.
- **Middle Man** — a class or function that mostly just delegates onward. → cut it, call the real target direct.
- **Refused Bequest** — a subclass or implementer that ignores or overrides most of what it inherits. → drop the inheritance, use composition.

## 4. Run both reviews in parallel

Launch two sub-agents in a single step so they run at the same time — in DSH, two `subagent` calls; in Claude Code, two `Agent` calls.

**Standards sub-agent prompt** — include:

- The diff command and commit list.
- The standards-source files found in step 3, **plus the smell baseline from step 3 pasted in full** — the sub-agent has no other access to it.
- The brief: "Report — per file/hunk where relevant — (a) every place the diff violates a documented standard: cite the standard (file + the rule); and (b) any baseline smell you spot: name it and quote the hunk. Distinguish hard violations from judgement calls — documented-standard breaches can be hard, but baseline smells are always judgement calls, and a documented repo standard overrides the baseline. Skip anything tooling enforces. Under 400 words."

**Spec sub-agent prompt** — include:

- The diff command and commit list.
- The path, fetched contents, or reconstructed text of the spec.
- The brief: "Report: (a) requirements the spec asked for that are missing or partial; (b) behaviour in the diff that wasn't asked for (scope creep); (c) requirements that look implemented but where the implementation looks wrong. Quote the spec line for each finding. If the spec was reconstructed from commit messages, treat each message as a claimed requirement, cite its short SHA, and flag any message too vague to judge. Under 400 words."

When there is no spec at all, skip the Spec sub-agent and note it in the final report.

## 5. Aggregate

Present the two reports under `## Standards` and `## Spec` headings, verbatim or lightly cleaned. Do **not** merge or rerank findings — the two axes are deliberately separate.

End with a one-line summary: total findings per axis, and the worst issue within each axis (if any). Don't pick a single winner across axes — that's the reranking the separation exists to prevent.
