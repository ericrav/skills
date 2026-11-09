---
name: lets-review
description: Review someone else's PR to the /lets-build and /cleanup standard. Says whether it is safe to merge, what to look at closely, and what it undoes.
disable-model-invocation: true
argument-hint: "[PR number or URL]"
---

# Let's Review

Review someone else's PR. Use the PR body as a hint for what the intent was, but review the actual code changes to explain what it does. Report only: do not edit, merge, comment, or file tickets until asked. Do not check out over a dirty tree. If PR is part of a stack, review in context of changes that may have been made later (point out if something is undone or changed later). Reverse-prompt to explain what the author's high-level intent was and what they likely prompted for.

Print the report once, at the end. Keep it short: lead with what needs attention and leave out what is fine.

Safely check out the branch in the workspace & update the PR to the latest main, resolving conflicts as needed, so it can be tested locally. If there are local changes, stash them, then re-apply.

## Report

### Summary / Prompt

### Verdict

**Merge** / **Merge after …** / **Hold**, then one sentence why. Then **Look closer at:** the few spots that need human eyes, with a reason for each.

### Undoes your work

Find lines this PR deletes or rewrites that the user wrote: blame the base for each removed or rewritten hunk, and match the user by `git config user.email` or the `gh` login. Split it into:

- **Reverts:** behavior, UI, or decisions the user built that this PR removes or changes. Name the user's commit or PR, then what changes. This section matters most.
- **Additive:** new code, not worth listing line by line.

If the PR body does not mention a revert, say so.

### Wide reach

Changes that affect more than their own feature: shared components and hooks, design tokens and motion constants, global styles, navigation structure, native modules, dependencies, migrations, API contracts, and copy keys. Say what else each one affects.

### DB diff

If applicable, show the DB diff for any changes to the database. Both schema migrations and application-level persistence changes.


### How it works

What shipped, with the smallest visual that makes it clear (`/show-me`), and `file:line` citations.

### How to test

Concrete checks: role, app, action, and expected result. Mark what CI already covers. Add one or two things to watch for.

### Decisions

One line saying why the PR exists, then one-line decisions from the PR body, commits, and code for each concrete behavior or UI change made. Add a `Mismatch:` line when the PR text says something the code does not do.

### Evidence

Explain *why the code has to be this way* for any choice a reviewer would ask about: limits, flags that turn off a default, format changes, extra API calls, workarounds, and "X because Y" comments. Cite a primary source: config, library source in `node_modules`, official docs, MDN, or an API reference. Report what is **contradicted**, and what is **unbacked** where it matters. List backed items in one line each. Skip taste values (durations, sizes, easing curves) unless they break a platform rule.

### Bugs and standards

Real bugs, with the line and the fix. Real `CODE_STYLE.md` violations, with the rule. Judgement calls get options and a recommendation. Drop anything you dismiss.

### Tickets

Which tickets the PR addresses (fully or partly), and the holes it leaves that need a ticket. Leave out unrelated tickets. Draft only.
