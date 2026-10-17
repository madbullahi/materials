# Module 2: Context, model, prompt

**Phase:** Apply · **Time:** two to three sittings

## Sources

| Read | For |
|---|---|
| [L03](../lectures/L03/slides.md) | The Big Three, the bullseye, the diagnostic order, surgical context, the specification spectrum |
| [L04](../lectures/L04/slides.md) | Information-dense keywords (IDKs), prompt levels, "the spec is the prompt", the five-section spec, Architect Mode |
| [demos/L03-transcript-analytics](../demos/L03-transcript-analytics/README.md) | A two-file change that falls apart when one file is missing |
| [demos/L04-transcript-analytics](../demos/L04-transcript-analytics/README.md) and its [spec.md](../demos/L04-transcript-analytics/spec.md) | The same codebase, a bigger change, landed in one pass from a written spec |

## You will be able to

1. **Name** the three levers in every AI coding session and **apply** the
   diagnostic order (context, then model, then prompt) to a failed result.
2. **Choose** the files to put in context for a change, separating what will
   be edited from what must only be read.
3. **Write** a prompt at the interface level using the
   `ACTION LOCATION: DETAIL` pattern.
4. **Write** a five-section spec, including beginning and ending context, with
   tasks in dependency order.
5. **Decide** when a spec is worth writing and when it is overkill.
6. **Run** Architect Mode and **review** a plan before any file changes.

## Before you start

Module 1. You should be able to run Aider against your model and read a diff.

## The ideas

### 1. Three levers, and two out of three is not enough

Every session has three choices in it. **Context**: what the model can see.
**Model**: which one, and is it strong and recent enough. **Prompt**: how
clearly you said what you want. A good result sits where all three overlap.
Effort on two does not buy back the third: a perfect prompt to a model that
never saw the file still produces a wrong file.

When output is wrong, ask in this order:

1. **Context.** Was the file in the window?
2. **Model.** Was it strong enough, and recent enough?
3. **Prompt.** Was the intent clear?

Most people blame the model. In practice the cause is most often context.

### 2. Surgical context

Before every task, not once per session, ask three questions about each file:
will it be **edited**, must it be **read** to get the edit right, or is it
only **reference** (where the repo map is usually enough)? Only edited files
need to be editable. Adding everything makes results **worse**, not merely
more expensive. `/tokens` tells you where the budget went; when history or
the repo map is the biggest line, that is the finding.

Two failures dominate: **missing context** (a file the change needed was
never added, so you get half a change that looks finished) and **history
bleed** (task A's assumptions are still in the window during task B).

### 3. Specify down to the interface, and say why

Prompts sit on a spectrum. **Outcome** ("another way to output the data")
leaves every decision to the model's priors. **Implementation** ("iterate the
speakers dict") does the work for it. **Interface** is the target: name the
file, the function, the signature and the call site, and let the model write
the body. Adding a short clause of **intent** (why you want it) steers the
decisions you left open. On a small model keep intent to a clause; a paragraph
of why can bury the one sentence of what.

### 4. IDKs narrow the sampling distribution

Information-dense keywords give a prompt a fixed shape:
`[ACTION] [LOCATION]: [DETAIL]`, with verbs like `CREATE`, `UPDATE`, `ADD`,
`DELETE`, `MOVE`. They work because of the mechanics in Module 1, not because
the model likes capital letters: a vague prompt leaves the distribution wide
and the model picks the most common completion; a precise one narrows it to
what you said.

### 5. The spec is the prompt

In 2024 the question was whether the AI could understand you. Now it is
whether you can say what you want. A human contractor asks when a spec is
vague; a model guesses. So a written spec is not paperwork before the work, it
is the instruction itself. L04's five sections:

| Section | Holds |
|---|---|
| High-level objective | What and why, in a sentence |
| Mid-level objectives | Three to five outcomes you can check |
| Implementation notes | Dependencies, standards, constraints: everything you would otherwise say in review |
| Context | Beginning files (editable or read-only) and ending files (new or updated) |
| Ordered low-level tasks | IDK-rich steps, each one building on the last |

Specs earn their cost on changes that touch three or more coordinated files or
start from scratch. A one-line bug fix does not need one.

### 6. Architect Mode: plan, then edit

Aider's Architect Mode splits one job into two calls: the architect writes a
plan in prose, the editor turns it into edits. With
`auto-accept-architect: false` the plan stops and waits for you. Nothing is on
disk yet, so this is the cheapest place to catch a misunderstanding.

## In the real world

**Payments.** A team adds a "transfer limit per agent per day" rule. The
change touches the rule engine, the database migration, the API validator and
the agent dashboard. Written as one sentence to a coding agent, it lands in
three of the four places and the dashboard still shows the old limit. Written
as a five-section spec, the Context section lists all four files and the
ordered tasks put the migration first, so the validator can reference the new
column. The Implementation notes carry the rule nobody would otherwise think
to tell a model: amounts are integers in the smallest currency unit, never
floats.

**Clinic.** The beginning-context list doubles as a data-handling decision.
Marking the fixtures folder read-only, or leaving it out entirely, is how the
team keeps sample patient records out of a hosted model's request.

**Campus.** Before registration week, three developers each write a spec for
a timetable feature. The test of a good spec is L04's: could another person
execute it? The team swaps specs and has each one built by someone who did not
write it. That habit is the Goldfish test you meet in Module 6.

**Open source.** A maintainer reviewing an AI-assisted pull request asks
"which files were in context?". A PR that changed `main.py` but not the
argument parser it depends on is L03's demo, in the wild.

## Exercises

### Core: the same change, twice

1. Run the [L03 demo](../demos/L03-transcript-analytics/README.md) exactly as
   its README describes: one file in context, the `--output-format` prompt,
   decline the invented file, run the program and see it break. Then `/undo`,
   add `src/arg_parse.py`, and send the same prompt again.
2. Run the [L04 demo](../demos/L04-transcript-analytics/README.md): paste
   `spec.md`, **read the plan before you accept it**, then run the two check
   commands at the end of its README to prove the flag was not silently
   ignored.
3. Write a short comparison (half a page): what did the spec carry that the
   L03 prompt did not? Point at specific lines in `spec.md`.

### Stretch

- **Write your own spec.** Pick a new feature for transcript-analytics (for
  example, `--min-length N` to ignore short words, or a `--speaker NAME`
  filter). Write all five sections, then run it in Architect Mode on the 4B
  with `./reset --model 4b`. If the 4B lands it, the spec is carrying its
  weight.
- **Calibrate your prompt level.** Write the same change at three levels:
  outcome, interface, implementation. Run each on a fresh copy. Which was the
  highest level that still landed first time, on which model?
- **Find the four grader mistakes.** L04 lists them: vague mid-level
  objectives, a missing context section, tasks out of order, code inside
  tasks. Deliberately put one into a copy of `spec.md` and describe what
  breaks.

## Pitfalls

| Pitfall | Fix |
|---|---|
| Saying "yes" when Aider offers to add a file mid-plan | In Architect Mode that re-runs the architect and loses the plan. Put every file the spec needs in its Context section up front |
| A spec whose objectives say "make it better" | Each mid-level objective should be checkable by running the program |
| Code inside the tasks | Describe the change; do not write it. A spec is instructions, not an implementation |
| Writing a spec for a two-line fix | Specs pay off at three or more coordinated files. Below that, an IDK prompt is enough |
| Re-wording a prompt five times | If it failed twice, the lever is probably context or model, not wording |

## Self-check

1. A change to authentication touches the API layer across 50 files. How do
   you decide what to `/add`?
2. Why does adding **all** the files make results worse, not just slower?
3. What is the difference between specification and intent, and which one
   extends which end of L03's spectrum?
4. How is a spec different from one very long prompt?
5. Why put "comment every function" in the spec instead of hoping for it?

<details>
<summary>Answers</summary>

1. Add the files that will be edited, then the files those must read to be
   edited correctly (the definitions they call, the types they use); leave the
   rest to the repo map. Do it per task, and split the change into tasks if
   the edited set is large.
2. Every token competes for attention. Irrelevant files dilute the relevant
   ones, so the needle is harder to find in the haystack (L02).
3. Specification closes decisions; intent steers the ones you left open.
   Intent widens the target toward the outcome end, so you can aim higher on
   the spectrum without the model guessing. It does nothing at the
   implementation end.
4. A spec is structured: separate objectives you can check, notes that carry
   your standards, an explicit list of files the model may touch, and tasks in
   dependency order so later tasks can rely on earlier ones.
5. Standards you do not write down are standards the model never had. It will
   follow the most common practice in its training data instead.

</details>

## Next

Module 3 asks who, in all of this, picks the next step. So far it has always
been you.
