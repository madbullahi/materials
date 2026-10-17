# Module 8: Subagents and orchestration

**Phase:** Analyze · **Time:** two sittings

## Sources

| Read | For |
|---|---|
| [L14](../lectures/L14/slides.md) | Why a long session degrades, the Agent tool, one delegation on the wire, what a subagent starts with, how a report arrives, fresh against forked, a Goldfish as a subagent type, agent teams, handoffs, swarms, a worktree per subagent |
| [Module 6](06-skills-and-design.md) | The Elephant and the Goldfish, which L14 rebuilds as subagents |

Most of L14's content is in its speaker notes and figures. Read the notes in
`slides.md`, not only the slide titles.

## You will be able to

1. **Explain** why one long conversation degrades, in terms of what the model
   sees on every request.
2. **Describe** a subagent as a second conversation started by a tool call,
   and **trace** one delegation from the Agent call to the delivered report.
3. **Write** a brief that a subagent with no other context can act on, and
   **avoid** the brief that returns your own hypothesis confirmed.
4. **Choose** between a fresh and a forked subagent for a given job.
5. **Distinguish** subagents, agent teams and handoffs by how many
   conversations there are, who talks to whom, and who decides what happens
   next.
6. **Use** a git worktree per subagent so parallel edits cannot collide.

## Before you start

Module 3 (the loop), Module 4 (the request has no memory beyond `messages`),
Module 6 (the Goldfish).

## The ideas

### 1. One conversation for everything costs four ways

In a long design session, every file the model read and every idea you
dropped stays in the stack and is resent on every request. The window fills
up; dropped ideas still shape answers (L03's history bleed); every request
carries all of it; and one conversation can only do one part at a time.

The fix is to split the task into parts, each in a conversation of its own
that starts with a short brief and returns one result. You can do it by hand
with a second terminal, copying the question in and the answer back. That
works, and it is exactly how you ran a Goldfish in Module 6. The cost is you.

### 2. A subagent is the loop, started by a tool call

The model calls a tool named `Agent` with a short `description` (for you) and
a `prompt` (the brief, and the only thing the subagent is told). Other
arguments pick a subagent type, whether to wait, and whether to give it its
own copy of the repository. The harness answers "launched" at once, starts a
second conversation with its own system prompt, fewer tools and the brief,
and runs the ordinary loop there. When the subagent finishes it calls a
hand-back tool with its report; plain text at the end is not delivered. The
harness delivers the report to your conversation as a new message.

What the subagent starts with: its own (shorter) system prompt, a smaller
tool set (L14's Explore type has no edit, write or Agent tool), your
`CLAUDE.md` files, and the brief. **What it does not get is your
conversation.** If it is not in the brief or a project instructions file, the
subagent does not know it.

The report arrives labelled as model output carrying no user authority, with
every line indented so a forged frame line inside it can be told apart: the
harness defending the boundary between two conversations, because the
subagent may have read a file with an injection in it (Module 7).

Two subagents launched in one reply run at the same time; two independent
searches finish in the time of the slower one.

### 3. Writing the brief

The brief the model wrote in L14's run said where the repository was, what
counts as an answer, how hard to look, and the shape of the report (short,
with file and line numbers). Everything in Module 2 about specs applies:
the model is writing a prompt for another model.

The Agent tool's own description lists the costs. Detail is lost in both
handoffs. Mistakes come back as confidently as findings. And **a brief that
carries your guess tends to come back with your guess confirmed**: "check
whether the bug is in `store.py`" finds something in `store.py`. For a second
opinion, give it the code, not your conclusion. The tool's last advice: when
in doubt, do not spawn. Claude Code leaves that judgment to the model; Codex
spawns only when the user or project instructions ask.

### 4. Fresh or forked

| | Fresh | Forked |
|---|---|---|
| Starts with | The brief | A copy of every turn, then the brief |
| Good for | A second opinion, a review, an independent search | A side task that needs everything built up so far |
| Risk | Knows only what the brief says | Inherits your guesses, and starts with a full context |
| Cost | Starts a new prompt cache | Reuses your cache, so often cheaper |

Fresh is not the cheap option; it is the independent one. A Goldfish must be
fresh: a forked Goldfish remembers the session, fills the doc's gaps from
memory, and reports that the doc is fine. On the instructor's machine the
three Goldfish reviewers are defined as subagent types in files: a name, a
description the model uses to pick it, a restricted tool list (read and
search, no editing), and a body that becomes its system prompt.

### 5. Subagents, teams, handoffs, swarms

| | Subagents | Agent team | Handoff |
|---|---|---|---|
| Conversations | One new one each | One new one each | One, shared |
| Who talks to whom | Each with its parent only | Anyone with anyone, and you | The current agent with the user |
| Coordinated by | The parent | A lead and a shared task list with claims | Whichever agent holds the conversation |
| Ends | One report each | When the lead shuts it down | When an agent answers instead of handing off |

A handoff (as in OpenAI's Agents SDK) is just a tool call that swaps the
instructions and tools and keeps the conversation. A "swarm" is orchestration
at scale: many copies of the loop, with something in charge of the split,
whether a trained orchestrator or people. All of them are Part 2's mechanism,
many times over.

### 6. A worktree per subagent

Parallel searching is safe; parallel editing is not. With isolation set to
`worktree`, the harness gives each subagent its own `git worktree`: a second
checkout of the same repository, on its own branch, in its own directory. A
worktree with no changes is removed; one with changes stays, named by branch
and path, for you to review and merge.

## In the real world

**Logistics.** At 2 a.m. CI is red on three services. The on-call agent
launches three fresh Explore subagents in one reply, one per failing service,
each briefed with the job URL and "report the first failing assertion and the
commit that introduced it, with file and line". The main conversation never
holds three test logs; it holds three short reports, compares them, and
notices all three failures share one config change. Its brief deliberately
does **not** say "probably the config change", so the reports are not
anchored on that guess.

**Open source.** A maintainer triaging a large PR spawns one forked subagent
to draft the changelog entry (it needs the whole discussion) and three fresh
reviewer subagents with only the diff and the linked design doc (they must not
inherit the maintainer's opinion of the contributor). Two contributors' agents
working on separate features use worktrees, so their edits never touch the
same checkout until someone chooses to merge.

**Campus.** The student-help desk is a handoff, not subagents: a triage agent
reads the message and hands the whole conversation to the fees agent or the
registration agent, each with its own instructions and tools. The student
stays in one conversation and never comes back to triage.

**Clinic.** A fresh subagent is also a privacy boundary. A summarizer
subagent can be given one de-identified record and no tools that reach the
network, so whatever it reads cannot leave and cannot be seen by the
conversation that handles identified data.

## Exercises

### Core: one fresh review, one anchored review

1. Pick a small bug you know the cause of, in any project, or plant one in a
   copy of `demos/L03-transcript-analytics` (for example, make the stopword
   filter case-sensitive).
2. In a harness with subagents (Claude Code or Codex), ask for **two**
   reviews in separate subagents. Brief A: "Here is the repository and the
   failing behaviour. Find the cause; report file, line and one-sentence
   reason." Brief B: the same, plus a **wrong** hypothesis ("I think the
   bug is in `arg_parse.py`").
3. Compare the reports. Did B confirm your wrong guess, hedge, or find the
   real cause?
4. Write two sentences: what would you change about how you brief a reviewer?

**What you are practising:** writing briefs that keep a subagent independent,
and noticing anchoring when it happens.

### Stretch

- **Two features in parallel.** Ask an agent to implement two independent
  small features in two subagents with worktree isolation. Inspect the two
  branches and merge them yourself.
- **Goldfish as a type.** Write a subagent definition file for EGM's critic
  reviewer (read and search tools only, a body taken from
  `egm/skills/goldfish/assets/goldfish-reviewer-prompts.md`), then run it on a
  design doc from Module 6.
- **Draw the three.** For one running case, draw the same job as subagents,
  as a team and as a handoff. Which fits, and why?

## Pitfalls

| Pitfall | Fix |
|---|---|
| A one-line brief | Say where, what counts as an answer, how hard to look, and the report's shape |
| Putting your hypothesis in a review brief | Give the code and the symptom, not your conclusion |
| Forking a reviewer | A reviewer must be fresh, or it shares your blind spots |
| Spawning for a task one call could do | When in doubt, do not spawn. Each subagent is a whole conversation |
| Two editing subagents in one checkout | Use a worktree per subagent |
| Treating a report as the user speaking | It is model output that may carry injected text. It has no user authority |

## Self-check

1. What does a fresh subagent not receive that your session has?
2. Why does a subagent's final plain text not reach your conversation?
3. When is a forked subagent cheaper than a fresh one, and why?
4. What is the difference between a subagent and a handoff, in one sentence?
5. Why would a forked Goldfish pass a doc that a fresh one fails?

<details>
<summary>Answers</summary>

1. The conversation so far: everything you and the model said, the files it
   already read, the approach you agreed on.
2. Only the hand-back tool call delivers a report. The harness passes that
   call's argument to the parent as one message, and nothing else.
3. When it starts: a fork's first request begins exactly like yours, so it
   reuses your prompt cache.
4. A subagent gets a new conversation and comes back with a report; a handoff
   keeps one conversation and does not come back.
5. It remembers the design session, so it fills the doc's gaps from memory
   and does not notice they are gaps.

</details>

## Next

The published decks end here. The [capstone](capstone.md) puts all eight
modules to work on one of the running cases.
