# Glossary

Terms the course uses in a specific sense. The module where each one is
taught is in brackets.

**Accessibility tree** — The structure a browser builds for screen readers:
each element's role and name. Browser tools give elements handles in it (such
as `e13`) so the model can click by reference. [7]

**Acceptance criterion** — A checkable statement of what a feature must do,
with a stable ID (for example `F6-3`) that appears in the design, the spec,
the test name and the commit. [3]

**Agent loop** — The few lines that call the model, run any tool calls it
asks for, append the results, and repeat until a stop condition. [3]

**Agent team** — Several conversations started by a lead, where any member
can message any other and they share a task list. [8]

**Approval gate** — Code, a reviewer model or a classifier that answers an
approval prompt so a person does not have to. Usually layered: code first, a
model next, a person for the middle band. [5]

**Approval step** — The check between a tool call's argument contract and its
dispatch: parse, boundary checks, the person, then dispatch or a denial
returned as the result. [5]

**Architect Mode** — Aider's split of one job into a planning call and an
editing call, with the plan shown before anything is written. [2]

**Assemble, answer, apply** — The three stages of every turn: the harness
assembles a request, the model answers with text, the harness applies it. [3]

**Autonomy gradient** — Five rungs from completion to many agents at once.
Rung 4 is where the model starts choosing the next action. [3]

**Big Three** — Context, model and prompt: the three levers in every session.
[2]

**Brief** — The prompt a model writes for a subagent; the only thing the
subagent is told about its task. [8]

**Capability** — How well a model reasons. Distinct from knowledge. [1]

**Channel** — An MCP server that starts a turn when a message arrives (for
example from a phone) and offers a reply tool. [7]

**Code mode** — A tool call that is a short program calling other tools as
functions, so intermediate output stays in the program. [7]

**Consecutive errors** — A stop condition: too many tool errors in a row, reset
by any success. [5]

**Context window** — Everything the model sees for one request. Outside it,
nothing. [1]

**Design doc (four-section)** — Problem, technical plan, alternatives,
detailed implementation. The Elephant's output. [6]

**Done signal** — How a run knows the task is complete: no tool call, a
`finish` tool, or verification by a check the harness trusts. [5]

**Elephant** — A long, context-rich design session, with no code, that grows
a design doc. [6]

**Exit line** — The single log line written when a run ends, carrying its
named reason. [5]

**Failed-stream rule** — If a streamed reply does not finish: record no
assistant turn, treat nothing as an edit, tell the user, stay usable. [4]

**Finish reason** — Why a reply ended: `stop`, `length` or `tool_calls`. Each
obligates your program differently. [4]

**Fork (subagent)** — A subagent that starts with a copy of the conversation,
then the brief. [8]

**Goldfish** — A fresh session that sees only the design doc. If it cannot
understand, critique and build from the doc, the doc is not done. [6]

**Handoff** — One conversation passed from one agent to another, swapping
instructions and tools. It does not come back. [8]

**Harness** — Everything around the model: the program that assembles
requests, runs tools and applies replies. Aider, Claude Code and Codex are
harnesses. [3]

**Heartbeat** — A scheduled turn that checks a short list and stays silent
(`NO_REPLY`) when there is nothing to report. [7]

**History bleed** — Earlier turns, including rejected ideas, shaping later
answers because they are still in the window. [2]

**Idempotent** — Running it twice has the same effect as running it once.
Reads are; adding a task, pushing, sending and charging are not. [5]

**IDK (information-dense keyword)** — Prompt verbs such as `CREATE`, `UPDATE`,
`ADD`, `DELETE` in the pattern `ACTION LOCATION: DETAIL`. [2]

**Knowledge** — What a model saw in training, frozen at a cutoff. Distinct
from capability. [1]

**Lethal trifecta** — Private data, untrusted content, and a way to send data
out. With all three, injected text can exfiltrate data. Remove one leg. [7]

**MCP (Model Context Protocol)** — A standard way for a harness to get tools
from a separate server: list the tools, call one. [7]

**Middle band** — Approval cases neither code nor a classifier can decide
with confidence, sent to a person. [5]

**Peanuts** — The EGM skill that maps a codebase bottom-up with a README per
directory, reviewed by a person at each level. [6]

**Progressive disclosure** — Loading instructions in levels: names and
descriptions always, a skill's body when asked for, its resources when the
body points to them. [6]

**Prompt injection** — Text the model reads that it follows as if you had
written it. [7]

**Reference point** — A short, stable name for project state (a feature, a
criterion, a spec file, a test) that you, a grader and a model can all
resolve. [3]

**Repeated call** — A stop condition: the same tool with the same parsed
arguments again, usually with nothing written in between. [5]

**Repo map** — Aider's ranked summary of signatures across the repository,
cut off at a token budget. [3]

**Skill** — A folder with a `SKILL.md` (frontmatter and instructions) plus
optional references, assets, scripts and evals, loaded when a task needs it.
[6]

**Spec (five-section)** — High-level objective, mid-level objectives,
implementation notes, context (beginning and ending files), ordered low-level
tasks. [2]

**Stop conditions (six)** — Task complete, step limit, token budget, wall
clock, repeated call, consecutive errors. A run that ends on a limit has not
finished. [5]

**Subagent** — A second conversation, started by a tool call, that does one
task and returns one report. [8]

**Token** — The unit a model reads and writes, and the unit of cost and
attention. [1]

**Token budget** — A stop condition on context size (of the next request, or
the run total; you decide). [5]

**Tool call** — A structured request from the model for the harness to run a
named action with arguments. The model never runs anything; it asks. [3, 4]

**Trigger evals** — A set of requests that should and should not load a
skill, used to test its description. [6]

**Usage** — The server's count of prompt and completion tokens for a request;
the ground truth your estimator is calibrated against. [4]

**Wall clock** — A stop condition on elapsed time: a timeout per command and
request plus a budget for the run. [5]

**Worktree** — A second checkout of the same git repository on its own branch
and directory, used so parallel subagents do not overwrite each other. [8]
