# Module 6: Skills and design before code

**Phase:** Analyze · **Time:** three sittings, one of them long

## Sources

| Read | For |
|---|---|
| [L10](../lectures/L10/slides.md) | What a skill is on disk, the description as trigger, skills against tools, progressive disclosure, Rensin's Elephant-Goldfish Model, the four-section design doc, three Goldfish reviewers, a real design prompt critiqued |
| [Lab03](../labs/Lab03/slides.md) | The pivot into the real Aider codebase, "learn, then design, then build", peanuts as a map, Codex as a teacher, designing skill loading for Aider |
| [egm/README.md](../egm/README.md) | The five EGM skills, how they hand off, how to install them in Claude Code, Codex and Aider |
| [egm/anatomy-of-a-skill.md](../egm/anatomy-of-a-skill.md) | How a skill is built, using the five as examples. Has its own exercises |
| [egm/walkthroughs/](../egm/walkthroughs/) | One page per skill. Read in pipeline order: peanuts, elephant, goldfish, egm-implement, mean-review |

## You will be able to

1. **Describe** a skill's three levels (catalog, body, resources) and **say**
   which turn's request each one enters.
2. **Write** a skill description that loads the skill at the right moment and
   not at near-miss moments, and **test** it with trigger evals.
3. **Distinguish** a skill from a tool and from an always-on instruction file,
   and **choose** where an instruction should live.
4. **Run** an Elephant design session and **produce** a four-section design
   doc with no code in it.
5. **Run** a Goldfish review with fresh sessions and **revise** the doc until
   only nits remain.
6. **Learn** an unfamiliar codebase with an agent as teacher, checking its
   claims against file names and line numbers.

## Before you start

Modules 2 and 4. You should know what goes into an assembled request and what
a tool round trip costs.

## The ideas

### 1. A skill is portable context plus a trigger

A skill is a folder an agent loads when a task needs it: a required
`SKILL.md` (YAML frontmatter, then Markdown instructions), plus optional
`references/` (read when the body says so), `assets/` (templates copied into
output), `scripts/` (code run when a step must come out the same every time)
and `evals/` (for the author, never read by the model). Everything is written
for a model to read. There is no runtime and no SDK, which is much of why the
format spread: Claude Code, Codex and many other harnesses read the same open
format.

### 2. The description decides when; the body decides how

Only `name` and `description` are in every request. The description is the
only part the model sees before deciding to load the rest, so most of the
craft is there: what the skill does, in the third person; when to use it,
with the phrases users actually type; and when **not** to. EGM's skills ship
twelve trigger evals each, six that should load the skill and six near misses
that should not.

### 3. Three levels of loading

| Level | Enters the request | Five EGM skills |
|---|---|---|
| Name and description | Every request, usually in the system prompt | about 900 tokens in all |
| `SKILL.md` body | After the model asks for it (a tool call), as that call's result in the next request | about 3,500 tokens each |
| References, assets, scripts | Only when the body points to them | Whatever is read, or only a script's output |

Loading all five bodies every turn would cost about 18,000 tokens before the
user typed anything; a working setup has twenty or forty skills. Anthropic
calls the alternative progressive disclosure. It is a table of contents.

### 4. Four homes for an instruction

| Home | Enters the request | Use it for |
|---|---|---|
| System prompt, `AGENTS.md`, `CLAUDE.md` | Every request | Rules that always apply |
| Tool definition | Every request, as a schema | Actions the harness performs |
| Skill | When the task calls for it | A way of working for one kind of task |
| Your own head | When you retype it | Nothing, ideally |

A tool is a function the harness runs and returns a result. A skill is
instructions the model follows and changes how it works. Skills often tell the
model which tools to use and when.

### 5. Design is the new code

Dave Rensin's Elephant-Goldfish Model, which `egm/` implements:

- **The Elephant** is a long, context-rich session where the design is grown:
  no code until the doc passes, interview before proposing, the model drafts
  first so its opinion is not just yours echoed back (the "sycophant
  defense"), and one four-section document comes out.
- **The design doc**: Problem (three to five plain sentences), Technical plan
  (block-diagram level), Alternatives (rejected ideas and why), Detailed
  implementation (every file that changes, and how).
- **The Goldfish** is a brand-new session that sees only the doc. If it cannot
  explain the system back, find no serious gaps, and could build it without
  asking anything, the doc is leaning on context that dies with the chat. EGM
  runs three at once: comprehension, critic, and readiness.

A Goldfish is an assemble decision: a request built with almost nothing in
it. The course's own design went through six Goldfish rounds and eighteen
reviewer reports; one caught a contradiction everybody in the Elephant
session had missed, because they all knew what was meant.

The full pipeline is `peanuts` (map a big codebase bottom-up with a README
per directory) → `elephant` → `goldfish` → `egm-implement` (build only the
files the doc lists, stop on drift) → `mean-review` (a hostile review of the
diff). Each handoff is started by a person on purpose.

### 6. Learn, then design, then build

Lab03 moved the course into the real Aider codebase: about 20,000 lines plus
tests, written by other people. The order matters. **Learn** first, with the
agent as teacher: ask how the code works, make it cite files and line numbers,
check what it says. Then **design**: you cannot make a design decision about
code you do not understand, and if you try, the model makes it for you
without either of you noticing. Then **build** in small steps whose prompts
carry your design. A one-shot prompt for a whole feature gives you the
model's design, not yours.

## In the real world

**Open source.** This is the maintainer's onboarding problem solved with the
course's tools. A new contributor runs `peanuts` on a branch, reviews each
generated README with the agent as teacher (peanuts' own notes warn that
about half of the leaf descriptions are wrong at first, and finding them is
how you learn the code), then designs their first change with `elephant`
and gets three fresh reviewers' reports before writing a line. The
maintainer reviews a design doc instead of a 900-line PR.

**Campus.** The portal team writes a skill for its house method of adding a
page: which template, which permission check, how to test it in a browser.
Before, every developer retyped half of it into the chat. Now the
description loads it when someone says "add a page for…", and the body
costs nothing on the other 95% of requests. Its trigger evals include near
misses like "fix the typo on the fees page", which should not load it.

**Payments.** For a regulated change, the Alternatives section is the
artifact auditors and future engineers ask for: why the team did not use the
partner bank's batch API, and why amounts are not floats. Six months later,
when someone proposes the rejected idea again, the answer is already written.

**Anywhere.** "If a fresh reader cannot build it from the doc, the doc is not
done" is a test any team can adopt, with or without AI: hand the design to a
colleague who was not in the meeting.

## Exercises

### Core: one Elephant session and one Goldfish round

1. Install the EGM skills for your harness as
   [egm/README.md](../egm/README.md) describes (Claude Code, Codex, or Aider
   with `/read-only`).
2. Read [anatomy-of-a-skill.md](../egm/anatomy-of-a-skill.md), then the
   elephant and goldfish walkthroughs beside their `SKILL.md` files.
3. In a throwaway copy of `demos/L04-transcript-analytics` (run its `./reset`),
   run `elephant` on a small feature, for example "speaker-level statistics:
   words per speaker and their top three words". Argue with it. Do not let it
   write code.
4. Run `goldfish` on the doc it writes. Read all three reports. Make one round
   of edits, run it again, and record in two or three sentences what the
   second round found that the first did not.

**What you are practising:** keeping the design decisions yours, and testing
a document against a reader who has none of your context.

### Stretch

- **Write a skill.** Pick a method you retype to coding agents (how you want
  commits written, how you run tests, how you review a diff). Write the
  description first. Then write twelve trigger evals in the format of
  `egm/skills/*/evals/trigger-evals.json`: six should load it, six near
  misses should not. Then write the body.
- **Measure progressive disclosure.** In Claude Code, run `/context` before
  and after a skill loads (anatomy-of-a-skill exercise 2). How much did the
  request grow?
- **Critique a real prompt.** Read L10's pivot prompt slides. List three
  things it does well and three a Goldfish would flag, then rewrite it with
  acceptance criteria.
- **Design Aider's loader.** Lab03's open question: Aider has no skill loader,
  so a skill costs its full body on every request. On paper, decide where the
  catalog goes, how the model asks for a body, and which turn it lands in.

## Pitfalls

| Pitfall | Fix |
|---|---|
| A vague description | The skill never fires, or fires on the wrong request. Say what, when, and when not |
| Running a skill you have not read | Read `SKILL.md` first: its hard rules, its stops, and the ledger it writes |
| Letting the Elephant write code | No code until the doc passes the Goldfish |
| Forking a Goldfish from the design chat | It fills gaps from memory and says the doc is fine. It must be fresh |
| Asking the agent to design the whole feature in one prompt | Learn, then design, then build in small steps |
| Trusting a generated README because it reads well | Check its claims against files and line numbers |

## Self-check

1. With twenty skills of about 4,000 tokens each, what is in the request on
   turn 1, and in which turn's request does a skill's body first appear?
2. Nothing executes when a skill is used. How does its body get into the
   conversation?
3. Why does the Elephant draft before hearing your proposal?
4. Which section of the design doc is most often skipped, and why does it
   matter most later?
5. Why must a Goldfish start with no memory of the design session?

<details>
<summary>Answers</summary>

1. Turn 1 carries only names and descriptions (a couple of thousand tokens).
   The body appears in the request after the model asks for it, as the result
   of a load call, in the messages.
2. Something does execute to load it: the model makes a tool call (in Claude
   Code, `Skill`; in a harness without one, a plain file read), and the
   harness returns the body as that call's result.
3. If you propose first, the model tends to agree with you. Committing to its
   own draft first gets you a real second opinion to push against.
4. Alternatives. It records rejected ideas and why, so nobody re-proposes them
   in six weeks without knowing they were already considered.
5. A Goldfish that remembers the session fills gaps from memory and reports
   that the doc is fine. The test only works if the reader knows nothing but
   the page.

</details>

## Next

Module 7 widens how far the same loop reaches: a shell, other people's
services, a browser, and your phone.
