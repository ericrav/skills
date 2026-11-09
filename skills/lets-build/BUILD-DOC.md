# Build document template

Copy to `.scratch/builds/<slug>.md`. Keep entries short. Link files, scripts,
and tickets instead of pasting them.

```md
# <Feature name>

Ticket: <issue link>
Branch / worktree: <name and path>
Status: <loop N — what is happening now>

## Goal

<One or two sentences. The destination, in the human's words.>

## Decisions

Only what the human said or confirmed. Newest last. Strike replaced lines.

- D1 (Q3 b) — <decision>. Why: <reason the human gave, if any>.
- D2 (Q4) — <decision>. Keeps V1 parity.
- ~~D3 — <old decision>~~ → replaced by D5.
- D5 — <new decision>.

## Slices

### S1 — <name> ✅

- Built: <what it does>. Files: <links>.
- Proof: `.scratch/builds/<slug>/s1-<name>.ts` → <key output>.
- Learned: <unknown unknowns, surprises, facts that changed the plan>.
- Repo tests added: <none | test and why it guards a real regression>.

### S2 — <name> 🚧

- <current state>

## Remaining

- [ ] <task> (blocked by: <open question or slice>)

## Open questions

- Q9 — <question>. Recommended: <answer>. Waiting on: <human | sub-agent fact>.

## Deferred

- <task> — why: <reason>. Ticket: <link or "not filed">.
```
