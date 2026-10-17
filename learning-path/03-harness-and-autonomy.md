# Module 3: The harness and the autonomy gradient

**Phase:** Apply, into Analyze · **Time:** two to three sittings

## Sources

| Read | For |
|---|---|
| [Lab01](../labs/Lab01/slides.md) | What a harness is, who chooses each part of the prompt, the repo map, how the prompt is rebuilt every turn, the seven features of the build |
| [L05](../lectures/L05/slides.md) | The six steps of a turn, the five-rung autonomy gradient, the four primitives, permissions as the safety layer |
| [L06](../lectures/L06/slides.md) | Owning decisions without reading every line, reference points, and how one edit command grows into tool calling and an agent loop |

## You will be able to

1. **Define** a harness and **draw** the assemble, answer, apply cycle, naming
   which steps belong to the model.
2. **Place** any coding tool on the five-rung autonomy gradient, and **say**
   what changes when the model starts choosing the next action.
3. **Identify** the four primitives of an agentic tool (loop, tools,
   planning, memory) in a tool you use.
4. **Use** reference points (stable IDs for features, criteria, specs and
   tests) to prompt by name and to get checkable reports back.
5. **Derive** tool calling from an edit-block protocol, and **write** the
   six-line agent loop.
6. **List** what that loop still needs before it is safe to run.

## Before you start

Modules 1 and 2. You should have used Aider enough to have an opinion about
it.

## The ideas

### 1. A harness is everything around the model

The model takes text in and gives text out. The harness is the program that
decides what text goes in and what happens to the text that comes out.
Aider, Claude Code, Codex and a chat window are all harnesses. Lab01's table
is the key to the rest of the course:

| Part of the prompt | Chosen by |
|---|---|
| System prompt | The harness, by mode |
| Added and read-only files | You, by command |
| Repo map | The harness, by a ranking |
| History | You start it, the harness trims it |
| Your message | You |

The repo map is why Aider seemed to know files you never added: it sends a
ranked summary of signatures, cut off where its token budget runs out.
Anything below the line is invisible, however relevant. And nothing carries
over between turns. Every part of the prompt is rebuilt each time.

### 2. Six steps, and one belongs to the model

A turn in Aider is: assemble the request, the model answers, parse the reply,
apply the edit, repair if the edit did not match (send the failure back and
try again, up to three times), commit. Exactly one of those six steps is the
model's. At every other step, the chooser is you or a rule somebody wrote in
advance.

### 3. The autonomy gradient

| Rung | Tool shape | Who picks the next action |
|---|---|---|
| 1 | Completion | You |
| 2 | Chat beside the code | You |
| 3 | Pair-programmer (Aider) | You |
| 4 | Agentic CLI (Claude Code, Codex) | **The model** |
| 5 | Many agents at once | The model, many times over |

Rungs 1 to 3 differ in how much they automate. Rung 4 differs in **who
decides**, which is a change of kind. Higher rungs spend more to ask less of
you. The skill is picking the rung for the task: a two-line fix in a file you
can name is cheaper and safer at rung 3.

Aider already has all four primitives: a loop (failed edits feed back, tests
can feed back), tools (read, write, commit, run configured commands), planning
(Architect Mode) and memory (history and the repo map). What it does not do is
pick the next task. **The difference is who chooses.**

### 4. Owning what you did not type

When a model writes most of the code, you do not read every line, and you do
not accept what you never checked. You check contracts and outcomes. That
needs shared names. A **reference point** is a short, stable name for a piece
of project state: `F6` for a feature, `F6-3` for one acceptance criterion,
`specs/03-preview.md` for the increment that delivers it,
`test_stale_preview_refused` for the test that checks it.

Names let you prompt precisely ("specs/03, task 2 only. Deliver F6-3. Make
`test_stale_preview_refused` pass. Edit `preview.py` only.") and ask for
reports you can check ("which criteria does this deliver, what did you run,
what did you not verify?"). They fail when a paraphrase replaces the ID, when
the model invents an ID, when the named file is not in context, or when an ID
changes meaning mid-build.

### 5. From one command to tool calling

Aider's edit block is a command: the model writes a marker, the harness
recognizes it, pulls out the arguments, and acts after asking. Add a `READ`
marker and a `RUN` marker and you have a dispatcher with three commands.
Strip the markers and every command is "which one, and with what arguments".
Describe each as a name, a description and a parameter schema, let the model
reply in a structured format, and that is **function calling**. The model
never runs anything. It asks.

### 6. The loop that picks its own step

Once a person is only relaying results back, replace them with a loop:

```python
while True:
    reply = model.call(messages, tools=TOOLS)
    if not reply.tool_calls:
        return reply.text          # the model is done
    for call in reply.tool_calls:
        result = execute(call)     # your code, your rules
        messages.append(result)
```

The model still only writes text, and your code still performs every action
and can refuse. The one thing that moved is who decides what happens next. And
the loop still needs: a cap on iterations and tokens, a detector for repeated
calls, a clean interrupt, and approval before running a command. That list is
Module 5.

## In the real world

**Logistics.** The on-call team's first agent is rung 3: a person picks the
failing service and the files, and the tool proposes a patch. Their second is
rung 4: given "CI is red on main", the agent reads the log, chooses files and
runs tests on its own. The second one is faster on the easy nights and
dangerous on the hard ones, because nothing in the six-line loop says "stop
before restarting a production service". Choosing the rung per task, not per
team, is the skill L05 asks for.

**Payments.** Reference points are already how regulated teams work, under
other names: requirement IDs traced to test cases and to change tickets. A
team that adds "which requirement IDs does this change deliver, and what did
you not verify?" to every agent session gets a report an auditor can read,
instead of a paragraph claiming the change is "safe".

**Open source.** A maintainer reviews an agent-written PR the "about right"
way: does it keep each decision the issue asked for, do the tests cover it,
is the diff the size the change should be? Not line-by-line, not rubber stamp.

**Anywhere.** "Prose asks, code and permissions enforce." A line in a system
prompt saying "do not touch production" is a request. A permission check in
the harness is a rule. Every serious deployment of an agent rests on the
second kind.

## Exercises

### Core: draw your harness, then grow it

1. Take one Aider session from your own history (or run the L03 demo again).
   Fill in Lab01's "who chooses each part" table for it, with the actual
   numbers from `/tokens`.
2. On paper, write the six steps of one turn of that session, and mark the
   one the model did.
3. Write the dispatcher from L06 for three commands (`edit_file`,
   `read_file`, `run_command`) as a Python function that takes a parsed list
   of `{"name": ..., "arguments": {...}}` and returns a list of results. Do
   not call a model; feed it a hand-written list. Make `run_command` refuse
   everything for now and return the refusal as its result.
4. Wrap it in the six-line loop with a fake `model.call` that returns two
   scripted tool calls and then a plain answer. Your loop should end on the
   plain answer.

**What you are practising:** seeing the model as the part that only writes
text, and your code as the part that acts.

### Stretch

- **Place your tools.** For every AI tool you have used this term, give its
  rung and one task you would and would not give it.
- **Reference-point a project.** Take any small project of yours and write
  five acceptance criteria with stable IDs, a test name for each, and one
  prompt that delivers exactly one of them by name.
- **Ask for the report.** In your next agent session, end a task with "which
  criteria does this deliver, what did you run, what did you not verify?".
  Check each claim. How many were only claimed?

## Pitfalls

| Pitfall | Fix |
|---|---|
| Reaching for the most autonomous tool because it is there | Pick the rung for the task. A change you can name fits rung 3 |
| Thinking Aider "is not agentic" | It has all four primitives. The difference is who chooses the next action |
| Paraphrasing a requirement in a prompt | Use its ID. Paraphrases drift; IDs do not |
| Taking "all tests pass" from the model | That is a claim. The test runner's output is the evidence |
| Treating a prompt instruction as a safety rule | Put rules in code. The model can ignore a sentence |

## Self-check

1. In Aider's six steps, which one is the model's, and which one is the only
   backward arrow?
2. What single change moves a tool from rung 3 to rung 4?
3. Why is reading a file the safest action to hand to a model first?
4. What does `"Ran: nothing"` in a model's report tell you, and what do you do
   next?
5. The six-line loop deletes a file outside the repository. Which line has to
   change to stop that?

<details>
<summary>Answers</summary>

1. Step 2, the answer. Repair is the only backward arrow: a block that does
   not match sends the failure back as a prompt and re-enters at step 2.
2. Moving the choice of the next action from the harness (or you) to the
   model. No new model capability is needed.
3. It changes nothing on disk and has nothing to undo. If the model can
   already name the file it needs, making a person type `/add` adds a step
   and no judgment.
4. That the change was never executed: any claim that it works is
   unverified. Run the tests or the program yourself before accepting.
5. `execute(call)`: that is where your code decides whether to run an action,
   so a check there (or an approval prompt) can refuse it.

</details>

## Next

Module 4 opens the call itself: the exact request your harness sends, the
stream that comes back, and what a tool call looks like on the wire.
