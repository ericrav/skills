---
name: semantic-compression
description: Run after a large coding task to find the concepts latent in the new code and reshape the code around them (Muratori's semantic compression), autonomously or in dialogue. Not a cosmetic refactor.
---

# Semantic compression

You have just finished a large piece of work. It runs and the checks pass. The code was written case by case, each case typed out until it worked. That was the right way to write it, and it is now the raw material for the second half of the job: finding out what the code is about and making it say so.

The method is Casey Muratori's semantic compression. Treat the code the way a dictionary compressor treats a file: wherever the same thing is expressed more than once, give it one definition and a name, and refer to it. The measure is semantic size, meaning how many distinct ideas a reader must hold and how many places state the same idea. Text length usually falls with it; when the two diverge, follow the ideas.

This differs from an ordinary refactor in what it produces. The names a compressor ends up with are the vocabulary of the problem: things said often get a word, things said once are spelled out where they occur. Architecture is derived from details that exist instead of being designed ahead of details that don't. The deliverable is a small set of named concepts, each defined once, with call sites that state only what is particular to them.

The cost being minimized is total human effort over the code's life: reading it, debugging it, changing it, adding the next case. Every decision below is judged against that.

## Preconditions

- **The task works.** Tests, typecheck and lint are green. Compressing code that is still changing gives names to behavior that is about to move.
- **A checkpoint exists.** The working, uncompressed state is committed (or stashed). Compression lands as its own commits so a reviewer can read behavior and shape separately, and any one step can be reverted alone.
- **Scope is known.** The scope is the task's diff plus any existing code the diff repeats. Get the diff from version control and read it; do not work from your recollection of what you wrote.
- **Mode is known.** See below.

## Mode

The pass runs the same way in both modes. They differ only in who settles the open questions in step 3.

**Dialogue** is the default when an operator is present. Some questions can't be answered from the code: whether two similar things are the same thing in the domain, what the team calls it, which direction the code will grow. The operator knows; you would be guessing.

**Autonomous** applies when the operator asked for it or nobody is there to answer. Compress only where the concept is unambiguous from the code, and write every judgment call into the report as an open question instead of acting on it. A missed compression costs little and can be done later. A wrong one welds together things that will need to be pulled apart.

## The pass

### 1. Read as a compressor

Read the whole diff top to bottom before changing anything, then look outward at the existing code it touches. Collect candidates of these kinds:

- **Repeated shapes.** The same sequence of operations with different values plugged in. Literal copies are the easy case; look harder for copies that drifted apart in variable names and ordering.
- **Variables that travel together.** A group of locals or parameters that are set up together, passed together and updated in step. This is a concept waiting for a data structure. Muratori calls the result a shared stack frame: the state several operations need in common, pulled out so those operations can become functions.
- **Protocols.** Steps that always occur in the same order around varying middles: set up, do a unit, do a unit, finish.
- **Facts stated twice.** A count declared up front that must match the number of things that follow; a list of keys that must match a switch; a type that restates a schema. Two statements of one fact can disagree. Prefer deriving one from the other.
- **Existing vocabulary you didn't use.** The codebase already had a name for something and the new code spelled it out again. Also the reverse: old code had one instance, your diff added the second, and now it qualifies.
- **Premature abstractions.** The opposite problem, and agents produce it often: a wrapper with one caller, an options object where every caller passes the same options, an interface with one implementation, a generic parameter instantiated once. These are compression applied before there was anything to compress. List them for inlining.

A candidate needs at least two real instances. One instance plus a belief that more are coming is not a candidate. Two instances are what tell you which parts are shared and which vary; with one, you are guessing at both and the guess is usually wrong in a way that makes the second use awkward.

### 2. Name what you found

For each candidate, write one row of a concept map:

| Field | Content |
|---|---|
| Name | What it is, in the problem's words |
| Definition | One sentence, no reference to how it's implemented |
| Instances | Every site, as file:line |
| What varies | The part each site supplies |
| What stays inline | Site-specific work that should not move |
| Confidence | Certain, or the specific question that would make it certain |

Naming is the test of whether you found a concept or a coincidence. If the best available name describes mechanics (`processItems`, `handleStep`, `utils`, `BaseManager`) or needs "and" in it, the repetition is probably incidental: two pieces of code that look alike today and will change for unrelated reasons. Leave those alone. A real concept can be explained to someone who knows the domain and has never seen the code.

A second test: would a change to one instance necessarily be a change to the others? If yes, they are one thing. If each could plausibly change alone, they are separate things that currently resemble each other.

While listing instances you will find sites that differ slightly from the rest: an extra check here, a different default there. Record each difference. It is either deliberate (the concept needs a parameter or the site is not an instance) or drift (a bug). You usually cannot tell which from the code.

### 3. Settle the open questions

**In dialogue mode**, show the operator the concept map and ask. Keep it to the questions that change what you'll build, give the evidence with each, and state your default so they can answer "defaults are fine". Useful forms:

- *Same thing or lookalike?* "A (file:line) and B (file:line) both do X. In your domain is that one operation, or do they just match today?"
- *The name.* "I'd call this `ImportRun`. What does your team call it?" The operator's word is better than yours even when yours is more precise; it is the word they'll search for.
- *Direction of growth.* "What's the next thing like this you expect to add?" The answer says which axis has to be cheap, and therefore what the concept's parameter is.
- *Deliberate or drift?* "C skips the validation that A and B do. Intended?"
- *Where it lives.* "This overlaps the existing `Session`. Is it part of that, a layer over it, or separate?"

Expect the answers to redraw the map. A concept may split in two, merge with one the operator names, or disappear because the similarity was coincidence. That redrawing is the point of the dialogue; update the map and confirm it before editing code.

**In autonomous mode**, proceed with rows marked certain. Move every other row to the report's open questions, with the evidence and your recommended answer.

### 4. Compress, one concept at a time

Take the concepts in order of how much they remove. Within one concept, the order that keeps each step small and obvious is:

1. **Data first.** Gather the variables that travel together into one structure. Rename fields from what they were locally (`x0`, `tmp`, `n`) to what they mean in the concept. The code gets no shorter at this step. It is groundwork.
2. **Then operations.** Pull out the setup, the repeated unit, and the finish as functions over that structure.
3. **Then derive the twice-stated facts.** With the state in one place, the count declared up front can be computed at the end from what happened.
4. **Then collapse the call sites** onto the new operations and strip the scaffolding that is left: temporaries used once, blocks that existed only to scope them.

Run the checks after each concept and commit it alone. If a step requires thinking hard about whether it preserves behavior, it is too big; split it.

Three rules while editing:

- **Behavior stays fixed.** Compression changes where things are said. If unifying the instances forces a choice between two behaviors, that is the drift question from step 2, and it belongs to the operator.
- **Unique code stays where it is used.** Only the shared part moves. A site's particular logic remains inline at that site, visible, in order.
- **Follow the codebase's conventions.** If it uses classes, the concept is a class; if it uses modules of functions, it is that. The concept matters; its packaging should look like its neighbors.

When an existing compressed piece almost fits a new site, there are three honest options: use it as is, change it so it fits every site, or expose the pieces it is built from so the odd site can assemble its own variant. Offering both the convenient call and the lower-level pieces is normal and good. Adding a boolean that switches the function between two behaviors is none of these options; it is two functions sharing a name.

Inline the premature abstractions from step 1 in the same pass. Each one is a commit of its own.

### 5. Judge the result

Read the call sites, since that is where the result shows.

- **Minimum information.** Does each site state only what is particular to it? If you deleted everything a reader could infer from the concept, would anything be left to delete?
- **Next instance.** Write the next case in your head. Count the lines and count the decisions. Both should be small, and none of the decisions should be bookkeeping.
- **Reads as the problem.** Could someone who knows the domain follow a call site without opening the definitions?
- **Nothing hidden.** Is anything unique to one site now buried inside a shared function behind a condition?

If a concept fails these, the usual cause is a wrong boundary: too much swept in, or the varying part chosen badly. Move the boundary or revert that commit. Then look again for second-order repetition, because compressed code often reveals a pattern the uncompressed code hid.

### 6. Report

```
## Semantic compression: <task>

Concepts introduced
- <Name>: <one-sentence definition>. Replaces <n> sites. <commit>

Abstractions inlined
- <what>, <why it had not earned its place>. <commit>

Facts now derived
- <fact> was stated in <places>; now computed from <source>.

Differences found between instances
- <site> differs by <what>. Preserved as is. Deliberate or drift?

Left alone, and why
- <lookalike>: <reason it is not one concept>

Open questions
- <question>, evidence <file:line>, recommended answer <...>

Checks: <what ran, result>
```

## Worked example

After a task adding CRM imports, three functions share this skeleton. `importDeals` and `importNotes` differ only in their middles, and `importNotes` gained a fourth stage while its `totalStages` still says 3.

```ts
export async function importContacts(jobId: string, src: Source) {
  const totalStages = 3
  let stage = 0
  let written = 0
  let skipped = 0

  stage += 1
  await setProgress(jobId, stage, "fetching contacts")
  const rows = await src.list("contacts")

  stage += 1
  await setProgress(jobId, stage, "validating contacts")
  const valid = rows.filter(isContact)
  skipped += rows.length - valid.length

  stage += 1
  await setProgress(jobId, stage, "writing contacts")
  await db.contacts.upsert(valid)
  written += valid.length

  await saveSummary(jobId, { stages: totalStages, written, skipped })
}
```

Reading as a compressor: `jobId`, `stage`, `written` and `skipped` travel together (shared state). Advance-then-announce is a repeated unit. Filter-and-count is a repeated unit. `totalStages` is a fact stated twice, once as a constant and once as the number of blocks, and in `importNotes` the two already disagree. The concept is a single run of an import: it has stages, it keeps or skips rows, it ends with a summary.

```ts
function beginImport(jobId: string) {
  let stage = 0
  let kept = 0
  let skipped = 0
  return {
    async stage<T>(label: string, work: () => Promise<T> | T): Promise<T> {
      stage += 1
      await setProgress(jobId, stage, label)
      return work()
    },
    keep<T, U extends T>(rows: T[], accept: (row: T) => row is U): U[] {
      const ok = rows.filter(accept)
      kept += ok.length
      skipped += rows.length - ok.length
      return ok
    },
    complete: () => saveSummary(jobId, { stages: stage, written: kept, skipped }),
  }
}

export async function importContacts(jobId: string, src: Source) {
  const run = beginImport(jobId)

  const rows = await run.stage("fetching contacts", () => src.list("contacts"))
  const valid = await run.stage("validating contacts", () => run.keep(rows, isContact))
  await run.stage("writing contacts", () => db.contacts.upsert(valid))

  await run.complete()
}
```

What to notice. The call site now lists the stages and what each does, which is the only thing particular to importing contacts. The stage count is derived, so the `importNotes` mismatch cannot recur. The fetch, the predicate and the upsert stayed inline because they are unique. Nobody designed an `ImportRun` up front; it was found in three functions that already worked. And two things went to the report rather than being decided silently: the `importNotes` summary has been saying 3 stages for a 4-stage import (drift, almost certainly, but the summary's consumers are the operator's call), and `written` is now counted at validation instead of after the upsert, which is equivalent only because a failed upsert throws before the summary is saved.

## Anti-patterns

- **Nouns-first design.** Inventing types or a hierarchy from the domain's nouns and fitting the code to them. Concepts come from repeated code, after it exists.
- **Speculative reuse.** Abstracting a single instance because more seem likely. Wait for the second.
- **Coincidental merge.** Unifying code that matches today and changes for different reasons. The naming test and the change-together test in step 2 exist to catch this.
- **Flag-parameter unification.** Forcing two behaviors through one function with a boolean or a mode string. Expose the shared pieces and let each site compose them.
- **Mechanism names.** `helper`, `util`, `manager`, `process`, `handle`, `common`. If that is the best name, there is no concept yet.
- **Golfing.** Fewer characters with the same number of ideas: dense one-liners, clever generics, chained ternaries. Semantic size is unchanged and reading cost went up.
- **Scattering.** Moving code that is used once into its own function or file for tidiness. It separates unique logic from its only use and adds a name the reader must look up.
- **Pass-through layers.** A wrapper that forwards its arguments. It adds a name and removes nothing.
- **Cosmetic pass.** Renaming, reordering and reformatting, reported as compression. If the number of places each idea is stated did not go down, nothing was compressed.
- **Silent drift-fix.** Quietly making divergent instances agree. It is a behavior change inside a commit that claims not to have one.
- **Compressed assertions.** In tests, compress setup and fixtures freely, but leave the expected values literal at each test. A test whose expectation is computed by a shared helper stops being evidence.

## When to stop

Stop when the call sites pass the checks in step 5 and the remaining repetition is either single-instance or failed the naming test. Some duplication is correct and should survive the pass. Stop early, and say so in the report, if the concepts you are finding belong to code well outside the task's scope: list them as follow-ups instead of widening the diff.

## If you delegate

Steps 1 to 3 and step 5 are judgment and belong with the strongest model available, or with the operator. Step 4 is mechanical once the map is settled. To hand it to a subagent or a faster model, give it one concept at a time: the map row, the intended signature, the instance list, what stays inline, and the instruction to run the checks and commit. Review each commit against step 5 before issuing the next.
