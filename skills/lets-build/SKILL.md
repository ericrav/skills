---
name: lets-build
description: >-
  Build a feature in small loops: a short round of questions, then build one
  concrete slice, prove it with a throwaway script, report back, and repeat.
disable-model-invocation: true
---

# Let's Build

I am adding a new feature or making a change to this project. We will incrementally plan and build until we reach the precise final state. Walk down each branch of the design tree, resolving dependencies between decisions one-by-one, and start writing code for the smallest slice that has been made concrete enough to build—we can revise as we go. Some decisions will come from my judgement, some will come from running spikes/experiments to determine the best approach, or by exploring the codebase or external research.

```
setup → [ grill → build slice → prove → report ] → … → finish
```

## Build document

Sessions run long and context fills up. The build document is the memory.
Write it in place of a plan.

- Path: `.scratch/builds/<slug>.md` in the current checkout (gitignored).
  `<slug>` is the issue number plus a short name, e.g. `312-deposits`.
- Throwaway scripts go in `.scratch/builds/<slug>/`.
- Give the human the absolute path when you create it.
- Update it at the end of every loop, before the report.
- Re-read it at the start of every loop and after any context summary.
  If it disagrees with memory, the document wins.
- A new chat can resume the build from the document alone.

Use the template in [BUILD-DOC.md](BUILD-DOC.md).

**Decisions** hold only what the human said or confirmed. Do not record
agent guesses as decisions. When a decision replaces an earlier one, strike
the old line and link the new one.

## Setup (once)

1. Check GitHub for the ticket and related issues.
2. Read `CONTEXT.md` and relevant `docs/adr/`.
3. Scope check. If the work spans independent subsystems, suggest one build
   per subsystem. Each build must give working, testable software on its
   own. If it is a vertical slice from backend to frontend, this
   build covers backend and domain. Ask UI questions only when they change
   the data shape. Defer UI specifics to `/lets-build-ui`. Use a throwaway
   debug UI if a screen helps prove the backend (loading, errors, state).
4. Create the build document.

## The loop

### 1. Grill (one round)

Follow [GRILLING.md](GRILLING.md) for the design tree, the round format, and
facts versus decisions. Use `/domain-modeling` as terms crystallise.

Any question that needs the human's input, anywhere in the session, starts
on a new line with ❓ and has a ➡️ recommendation.

- If the codebase answers a question, look it up. Do not ask.
- Do not guess library APIs. Check skills, docs, `node_modules`, or the web.
- If the technical shape is not obvious, read `/spec-boundaries`. Ask
  whether the human wants to guide it, and collect code or pseudocode.
- If a branch needs an untested library or API, offer a spike. The spike is
  just a small slice: run it, report, continue.
- State the next slice, its estimated difficulty, and how you will prove it.
  If the code alone cannot show the current state, find it first: debug
  logs, the terminal, a script, or ask.

Wait for answers. Record them in **Decisions**.

### 2. Build one slice

Pick the smallest slice that is fully decided and concrete. It must run on
its own. Check that its prerequisites are in place. Follow `CODE_STYLE.md`,
`/simple-first`, and for Effect backend code `/effect` and `/repo-service`. Do not build ahead of open questions.

### 3. Prove it

Prove the slice works 100%. Do not argue from reading the code.

- Write a throwaway script in `.scratch/builds/<slug>/`: call the endpoint,
  run the query, drive the workflow, read the DB. Run it. Keep the output.
- Show the before and after states. When possible, run the same script
  before and after the change.
- Cover the failure paths that matter, not only the happy path.
- Use debug logs and the terminal output when the script cannot reach it.
- Add an automated test to the repo **only** when a regression here is
  realistic and future changes need the guard. No trivial tests. No tests
  for coverage. Say why each new test earns its place.

### 4. Report

Update the build document first. Then report in chat:

- **Done**: what the slice does, in one or two sentences.
- **How it works**: call tree, pseudocode, or diff, with file links.
- **Proof**: the script path, the command, and the key output.
- **Learned**: facts found while building, especially unknown unknowns.
  Say which earlier decisions or remaining tasks they change.
- **Next**: the next slice and the next round of questions, per
  [GRILLING.md](GRILLING.md).

Then go back to step 1.

## Finish

When no tasks remain (or the human calls it):

1. **Review the whole build.** Read every decision and fact together. Note
   contradictions and ripple effects that simplify the work. Push back
   gently on anything overlooked. Check against `CONTEXT.md`; note
   amendments needed.
2. **Clean up.** Review the full diff against `CODE_STYLE.md`. Refactor and
   simplify. Delete the throwaway debug UI. Leave `.scratch/` alone.
3. **Code review.** Restate what was built. Show it visually: pseudocode for
   logic, call tree for control flow, component tree for UI, shallow file
   tree for file roles, diff when the shape already exists. Link each file
   and line in the order presented.
4. **Manual verification.** Short steps to run and check it, with what to
   look for.
5. **Product management.** Record the human decisions on the relevant
   tickets. Record deferred tasks as tickets or ticket notes. Report project
   health and progress.
6. Open one PR against `main` per `CLAUDE.md`.

## Anti-patterns

- Asking every question before writing any code.
- Building past an open question.
- "It should work" without a script run as proof.
- Repo tests that only prove the slice once.
- Recording agent assumptions under **Decisions**.
- Letting the build document fall behind the chat.
