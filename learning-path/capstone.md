# Capstone: an agent for a real team

This is not a graded course project. It is a way to prove to yourself that
the eight modules fit together, on a problem shaped like real work. It is
sized for two to three weeks alongside other study.

## The brief

Choose one of the [running cases](running-cases.md) and build a small agent
that does **one** useful job for that team, end to end, with the safety and
evidence a real team would ask for. Suggested jobs:

| Case | A job of the right size |
|---|---|
| Payments | Investigate a failed transfer from its ID: read logs and the ledger, explain what happened, and propose (never perform) a correction |
| Clinic | Draft a referral letter from one de-identified record and a template, citing which record fields each sentence came from |
| Campus | Check one portal page after a change: open it in a fresh browser, complete one form as a test student, report errors, and write a regression test |
| Logistics | Triage a red CI run: find the first failing test, the commit that broke it, and post a short summary |
| Open source | Answer "where is X implemented?" questions about a codebase you did not write, with file and line citations, using a `peanuts` map |

Use fake data throughout. Nothing real belongs in a practice agent.

## What to build, module by module

| Module | Deliverable |
|---|---|
| 1. How a model sees code | A short note on your model choice: capability, knowledge cutoff, context size, and cost per run (estimated, then measured) |
| 2. Context, model, prompt | A five-section spec for each increment, committed **before** the code it describes |
| 3. Harness and autonomy | The rung you chose and why; acceptance criteria with stable IDs, each traced to a test |
| 4. Inside the call | A client that obeys the URL rule, handles all three finish reasons and the failed-stream rule, and round-trips tool calls correctly; tested with a fake `call` |
| 5. Stopping and approving | All six stop conditions plus `truncated` and `endpoint_error`, one exit path, a JSON run log; a retry loop that owns its retries; an approval step with a rules file, and an eval of that rules file on at least 20 labelled calls |
| 6. Skills and design | A four-section design doc that passed at least one Goldfish round (keep the reports); optionally, your method packaged as a skill with trigger evals |
| 7. Reaching the computer | A table of your agent's tools and which trifecta leg each adds, and the leg you removed |
| 8. Subagents | Either one justified use of a subagent (with its brief), or a paragraph on why your job does not need one |

## Order of work

1. **Learn** the domain and any code you are building on (Module 6). Write
   down what you checked and how.
2. **Design** with an Elephant session; **test** with a Goldfish round.
3. **Spec** the first increment. **Build** it. **Run it as the user.**
   Record what running it showed, as in Lab02's iterating table.
4. Repeat 3 for each increment. Commit the spec, then the code, every time.
5. **Run the agent unattended** on at least five realistic tasks. Keep every
   run log.
6. **Run `mean-review`** on the final diff and record what you fixed and what
   you rejected.
7. **Write a one-page reflection**: what the agent does, what it refuses,
   what surprised you, and the most alarming thing it ever asked to do.

## Evidence to keep

- Git history showing specs before code.
- Run logs with an exit line for every run, and a count of exit reasons.
- The rules-file evaluation: safe actions blocked, dangerous actions allowed.
- Goldfish and mean-review reports.
- The reflection.

## Self-assessment

Score yourself honestly. The course's own principle applies: no points for
suffering. More prompts and longer logs earn nothing; clear decisions and
evidence do.

| | Not yet | Getting there | Solid |
|---|---|---|---|
| **Design** | The doc was written by the model and approved | Your decisions, but a Goldfish found gaps you did not fix | A fresh reader could build it; the alternatives are recorded |
| **Specs and traceability** | Code first, specs after | Specs exist; IDs are inconsistent | Every criterion has an ID, a test and a commit that cites it |
| **The call** | Works on the happy path | Handles errors, but a dead stream can corrupt history | Every finish reason and the failed-stream rule are tested without a network |
| **Stopping** | A step limit | Six conditions, "stopped" in the log | Six-plus named exits, one handler, numbers you can justify |
| **Approval** | Asks about everything, or nothing | A rules file, never evaluated | Code first, a measured rules file, a defined middle band |
| **Reach and risk** | Tools added because they were available | Tools listed | Each tool's trifecta leg named, one leg removed on purpose |
| **Evidence** | "It works" | Some logs | Logs, evals and reports someone else could check |

## Make it real

If you are studying in a team or teaching this course, the capstone works as
a group exercise: one person per running case, a shared Goldfish round in
which each person reviews another's design doc cold, and a final session
where each team demonstrates one unattended run and one denied action.
