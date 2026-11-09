# Grilling

Interview the human until the next slice has no open decisions. Ask in chat,
not the Question Tool.

## Design tree

Map the work as a **design tree**: every decision branches into the decisions
that hang off it.

The **frontier** is every decision whose prerequisites are already settled:
the questions you can ask _now_ without guessing at answers you have not heard.
In `/lets-build`, ask the part of the frontier that blocks the next slice,
plus any decision that changes the shape of everything (data model,
ownership, choices that are hard to undo). Leave the rest for later rounds.

A question whose answer depends on another question still open in this round
belongs to a _later_ round.

Each answer reshapes the tree. Settled decisions push the frontier outward
and unblock the questions that hang off them. Recompute the frontier at the
start of every round. Record the open branches under **Open questions** in
the build document so they survive a context summary.

## Facts versus decisions

Finding _facts_ is your job, never the user's. When a frontier question needs a fact from the environment (filesystem, tools, etc.), dispatch a sub-agent to find it; don't ask the user for anything you could look up yourself. Don't block on it: a running exploration is an unsettled prerequisite, so only the questions downstream of it wait for the sub-agent to report; ask the rest of the frontier now. The _decisions_ are the user's: put each to them and wait.


## Format

Number the rounds. Number questions across the whole build (Q1, Q2, … Q14),
so an answer like "Q7 b" stays unique and maps to the build document.

Every question that needs the human's input starts on a new line with ❓.
Give options as lowercase letters when options exist. Not every question
needs options.

Every question gets a recommendation on a line that starts with ➡️. Say why,
and the impact if the human picks it or skips it.

```
### Round 3

❓ **Q7** - **<question title>**: <question body, might be multiple paragraphs> options:
a. <option>
b. <option>

➡️ <recommended answer>. Why: <reason>. If chosen: <impact>. If skipped: <impact>.

---

❓ **Q8** - **<question title>**: <question body>

➡️ <recommended answer>. Why: <reason>. If chosen: <impact>. If skipped: <impact>.
```

## Done

A round is done when the human has answered it. The build is done grilling
only when the frontier is empty: every branch visited, nothing silently
assumed. Do not act on an answer you inferred. Confirm it first.
