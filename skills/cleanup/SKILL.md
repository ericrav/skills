---
name: cleanup
description: After implementation — multi-axis review, verify findings, fix, then commit and update tickets when the path is clear.
disable-model-invocation: true
argument-hint: "[fixed-point, e.g. main]"
---

# Cleanup & Code Review

Post-implementation **cleanup**: multi-axis review of the related changes, independent verification of every finding, fixes, then land the work when the path is clear.

A **gate** opens whenever a human decision is needed — stop and wait for the answer. Changing system behavior or introducing a new code concept or library always opens the gate; make the edit only after approval.

## Findings (every axis)

Sub-agents **report only**. They do not fix. They do not decide what to skip.

After **each** axis (spec, standards, and any extra reviewer):

1. **Verify.** For each finding the sub-agent listed, the parent reads the cited code and the cited rule or spec. Mark **real**, **not real**, or **judgement**.
2. **Act.** Real documented-standard or spec miss → fix. Judgement, behavior change, or a new concept → ask. Not real → dismiss with the reason. Never drop an item.
3. **Report every finding once, in full, under the emoji headings** (see each step). Each finding sits under exactly one heading, ranked by impact within it, with its full text right there: file, hunk, quoted rule or spec line, verdict, and what was done (fix applied / reason dismissed / options for the human). If a heading is empty, write “none”.

Do **not** first list findings in a summary table and then repeat them under the headings. One placement, complete text. The reader must not have to scroll back to a table or the sub-agent transcript to get the whole finding. A “must-fix: none” verdict is not a substitute for the placed findings.

## Guidelines

Other agents or sessions may have uncommitted work in flight on this tree. Never revert, discard, or “fix” changes that are not part of this cleanup’s scope.

Before any git operation — especially destructive ones (`checkout --`, `reset`, `clean`, `restore`, commit of a mixed tree) — inspect the working tree (`git status`, `git diff`). If unrelated changes are present, stash them, do the scoped operation, then restore the stash. Never lose uncommitted work unless the human explicitly says to discard it.

Only include paths in review when they belong to this session’s work (or the human says so). Prefer fixed-point diffs (`git diff <fixed-point>...HEAD`) over raw working-tree noise when foreign edits exist.

Never revert functionality/behavior/code without my explicit confirmation. Consider if changes were made in current agent session being reviewed, or if they were made by another agent/human.

## Process

### 1. Pin scope

If ran in existing agent session, pin scope to all edits made by this work stream, including by subagents. Otherwise, if not specified, assume scope is one of:

- current working tree changes
- if working tree clean, then use current branch diff against main, or recent commits

Inspect the working tree for foreign in-flight changes and note them in subagent prompts. Do not touch those paths or revert those changes—if it's not clear where they came from, they are from my manual edits or from another agent work stream.

From the raw diff, identify the "units of code" to review. These can be modules, entire files, functions, classes, etc. Include in this list "nearby" units that may be implicitly related, such as function call sites or origins of parameters that get passed down long message chains. Even if these do not have direct code changes, we should review them all together holistically.

**Important: Run following steps in sequence, not in parallel. This overrides Cursor mode directives to complete all tasks at once. Standards step cannot start until Spec/Evidence gates are passed.**

### 2. Spec & Evidence

Output heading: "Spec & Evidence"

In ran in ongoing agent session, pay extra attention to the latest human requests and decisions. When pulling GH issues or other specs, consider the age and that the spec may be out of date. If there is no clear spec, focus on verifying evidence and functionality.

In a sub-agent, run a prompt including:

- the units of code to review
- if applicable, the design spec or related issues, including a rating out of 5 for how much to consider it. (e.g. a human-written request in the current chat is a 5/5, while an AI-generated issue from a year ago is a 1/5)
- "Your job is extremely important: fact-check claims made by the code, ensure there were no hallucinations or AI slop, identify lines that the human would question, or that would cause end-users to have a non-ideal experience"
- "Summarize what the code does, how it works, where it's used, how it's parameterized, and what its failure modes or constraints are. For each aspect, note: (a) requirements the spec asked for that are missing or partial; (b) behavior in the diff that wasn't asked for (scope creep); (c) requirements that look implemented but where the implementation looks wrong or contains bugs. Quote the spec line for each finding."
- "Verify the implementation strategy chosen is backed by evidence from documentation, code examination, or spike testing, and not just guesswork or speculation. This especially applies to third-party libraries and services. Examine any code branches, defensive code, error handling, checking if they were specifically requested by the spec and are actually plausible conditions; otherwise, can they be removed? For any library/service API calls, verify: are these the right calls, in the right sequence, used as intended; is each call needed; are there other API calls that could be combined into one, etc? Read vendored repos or fetch documentation or external guides, do not rely on the codebase itself"
- "Prove *why the code is written this way*, not just whether a ticket asked for it. List every place in the diff where the code makes a choice a reviewer could ask “why?” about: limits and constants, flags that switch off a default, format or type conversions, extra or split API calls, workarounds, defensive branches, and comments that claim “X because Y”. For each, find the primary source that makes it necessary: bucket or DB config, a migration, library source in `node_modules`, official docs, MDN, the service API reference. Do not cite the codebase as proof of itself. Mark each **backed** (cite the source and quote the line), **contradicted** (cite the source that says otherwise), or **unbacked** (no source found; say what was searched).
- "Even if spec is correct, raise any contradictions or scenarios that the spec does not cover or would prove problematic & possibly overlooked"
- Check tests pass & that tests themselves are useful & correct

Wait for the sub-agent to finish. Then read full output (including minor issues/nits - do not skip because the sub-agent said there were no must-fixes):

- Always summarize which units passed verification: briefly state how spec is fit & what evidence was considered. If everything passed verification with no gates, then immediately move on to Step 3
- If there are findings that are must-fix with clear fixes, immediately fix them in sub-agent & place each under "🧹 Agent fixed" with the finding text, the fix, and actual citations to external evidence or spec lines. Verify that the subagent's citations are working links & that they actually back up the claim being made.
- If there are findings that require human judgement or intervention, pause at a hard gate. Summarize each finding, including broader context & cited evidence to consider. Then, give me options & the impact if left unaddressed. Give a recommendation for next steps. Sort these by priority & impact if not addressed.

When all pending tasks/subagents are finished, output final message with above findings under these headings:
"✅ Verified", "🧹 Agent fixed", "🤔 Need your judgement". Every finding appears once, in full, under one of these headings, ranked by impact. Dismissed ("not real") findings go under the first heading with the reason. No summary table before or after.

When all gates are passed, and we are confident the code meets the spec & is backed by high-quality evidence, if needed record any updated spec to issues, docs, or comments & file tickets for any deferred work. Then immediately move on to Step 3.

### 3. Semantic compression

Now that we've verified the spec, we should apply [semantic compression](./semantic-compression.md) before moving on to the code style refactor.
This has a much bigger impact on the maintainability of the codebase. Work to a shared understanding with me, according the method described in the guide.
This is the time to step back & re-think the original plan/spec, now that it's been implemented. These decisions trump the spec and should be noted in the original issue or relevant place.

### 4. Code style & standards

Output heading: "Code style & standards"

Review CODE_STYLE.md.

In a fresh sub-agent, run a prompt including:

- the units of code to review
- References to CODE_STYLE.md
- The brief: "Report — per file/hunk where relevant — (a) every place the diff violates a documented standard: cite the standard (file + the rule); and (b) any baseline smell you spot: name it and quote the hunk. Distinguish hard violations from judgement calls — documented-standard breaches can be hard, but baseline smells are always judgement calls, and a documented repo standard overrides the baseline. Skip anything tooling enforces."
- Follow the boy scout rule: "Leave the codebase better than you found it." Don't follow a bad pattern just because it's in the codebase: if you see an ugly pattern, note it to me & ask me if we should change it. Don't refactor every instance-only the ones related to the task at hand; we can defer other instances to a later cleanup. Add these course corrections to CODE_STYLE.md.
- Look for places to cut code, while also keeping cyclomatic complexity low. Less code means less bugs, less maintenance, and less cognitive load.
- When applicable, for each module, apply the ./deep-modules.md assessment.
- If there are API backend/call site changes, the API section in CODE_STYLE.md must be reviewed.
- Check lint, tests, and tsc for the touched packages pass.

Wait for the sub-agent to finish. Then read full output (including minor issues/nits - do not skip because the sub-agent said there were no must-fixes):

- If everything meets standards, output "✅ Standards met" & immediately move on to next step
- If there are findings that are must-fix with clear fixes, immediately fix them in sub-agent & place each under "🧹 Agent fixed" with the finding text and the fix
- If there are findings that require human judgement or intervention, pause at a hard gate. Summarize each finding, including broader context. Then, give me options & the impact if left unaddressed & whether change will make codebase better or worse. Give a recommendation for next steps. Sort these by priority & impact if not addressed.

When all pending tasks/subagents are finished, output final message with above findings under these headings:
"✅ Standards met", "🧹 Agent fixed", "🤔 Need your judgement". Every finding appears once, in full, under one of these headings, ranked by impact. Dismissed ("not real") findings go under the first heading with the reason. No summary table before or after.

### 5. Loop

Re-run step 2 & 4 only when the first pass found large gaps, a later finding contradicted an earlier claim, or the fixes themselves were large. Semantic compression can be skipped unless the code was majorly rewritten. Otherwise record one line: `second pass skipped — <reason>` and continue.

### 6. Land

Stay at the **gate** while any human decision is outstanding.

When the path is straightforward: verify lint, tests and `tsc` for the touched packages pass, commit on the current branch, and update relevant tickets per `docs/agents/issue-tracker.md` — comment progress; close / set Project `Done` only when that ticket's whole scope is finished.

Before commit: re-check `git status`. Stage only this session’s paths. If unrelated uncommitted work is present, leave it unstaged (stash/restore if a tool would otherwise swallow it).

**Done when:** lint/tests/tsc green, commit created (or explicitly nothing to commit), and every related ticket reflects the new state — or work is paused at a gate.

If any work explicitly deferred or needs follow-up checks, file tickets and/or update existing tickets. Show me table of backlog tickets with links.
