# Learning path

The decks in `lectures/` and `labs/` are written to be delivered in a room.
This folder is written to be studied alone. It puts the course in order as
eight modules, says what you should be able to do after each one, and ties
every idea to a place you will meet it outside a classroom.

Nothing here replaces a deck. Each module links to the decks, demos and EGM
pages it is built on, and tells you which slides to read. When a module and a
deck disagree, the deck wins: it was delivered, and this was written after.

## How to use it

1. Read [running-cases.md](running-cases.md) once. It introduces five
   real-world teams that come back in every module, so that "where would I use
   this" always has the same five answers to compare.
2. Work the modules in order. Each one takes one to three sittings.
3. In each module, read the decks it lists **first**, then the module. The
   module is the second pass: it names the objectives, joins the decks
   together, and gives you something to do with them.
4. Do at least the core exercise in each module. They run on the demos in
   `demos/` and the skills in `egm/`, so you need nothing that is not in this
   repository and a model to point at.
5. Answer the self-check questions before you open the answers.
6. Finish with the [capstone](capstone.md), which uses all eight modules on
   one of the running cases.

[glossary.md](glossary.md) defines every term the course uses in a specific
sense. [further-reading.md](further-reading.md) lists the outside sources the
modules cite.

## Every module has the same shape

| Section | What it gives you |
|---|---|
| **Sources** | The decks, demos and EGM pages it is built on, with what to read in each |
| **You will be able to** | Four to six objectives, each something you can do and someone can check |
| **Before you start** | What you need from earlier modules |
| **The ideas** | The core of the module, in order, short |
| **In the real world** | Where each idea shows up in the running cases, and what goes wrong without it |
| **Exercises** | A core exercise on this repo, then stretch exercises |
| **Pitfalls** | The mistakes the course saw students make, and the fix for each |
| **Self-check** | Questions, with answers folded away |
| **Next** | What the next module builds on |

## The course map

Three phases, one repository. **Apply** teaches you to drive a coding agent.
**Analyze** opens up the program around the model, the harness, and has you
build its parts. **Create** is where the work goes next (posted as it
happens).

| Module | Phase | Lectures | Labs | Demos and EGM | The question it answers |
|---|---|---|---|---|---|
| [1. How a model sees code](01-how-a-model-sees-code.md) | Apply | [L01](../lectures/L01/slides.md), [L02](../lectures/L02/slides.md) | [Lab00](../labs/Lab00/slides.md) | [L01-notes-cli](../demos/L01-notes-cli/) | What is actually in front of the model? |
| [2. Context, model, prompt](02-context-model-prompt.md) | Apply | [L03](../lectures/L03/slides.md), [L04](../lectures/L04/slides.md) | | [L03](../demos/L03-transcript-analytics/), [L04](../demos/L04-transcript-analytics/) transcript-analytics | When output is wrong, which lever do I pull? |
| [3. The harness and the autonomy gradient](03-harness-and-autonomy.md) | Apply | [L05](../lectures/L05/slides.md), [L06](../lectures/L06/slides.md) | [Lab01](../labs/Lab01/slides.md) | | Who picks the next action? |
| [4. Inside the call](04-inside-the-call.md) | Analyze | [L07](../lectures/L07/slides.md), [L08](../lectures/L08/slides.md) | [Lab02](../labs/Lab02/slides.md) | | What goes over the wire, and what can break? |
| [5. Stopping and approving](05-stopping-and-approving.md) | Analyze | [L09](../lectures/L09/slides.md), [L11](../lectures/L11/slides.md) | | | When does a run end, and who says yes? |
| [6. Skills and design before code](06-skills-and-design.md) | Analyze | [L10](../lectures/L10/slides.md) | [Lab03](../labs/Lab03/slides.md) | [egm/](../egm/README.md) | How do I package a method, and prove a design stands alone? |
| [7. Reaching the computer](07-reaching-the-computer.md) | Analyze | [L13](../lectures/L13/slides.md) | [Lab04](../labs/Lab04/slides.md) | | How does an agent reach a shell, other services, a browser, a phone? |
| [8. Subagents and orchestration](08-subagents-and-orchestration.md) | Analyze | [L14](../lectures/L14/slides.md) | | | When is one conversation not enough? |
| [Capstone](capstone.md) | All | | | | Can I design, build and measure an agent for a real team? |

### Gaps in the published materials

- **L12 (Oct 8)** has not been published. L11 says the Analyze specification
  was handed out at the end of it, and L10 says serving a local model was
  moved out of its original slot to a date still to be set. Module 5 notes
  where L12 would sit. Nothing in this folder guesses at its content.
- **Lab02** was published as a PDF only. [`labs/Lab02/slides.md`](../labs/Lab02/slides.md)
  is its slide text, recovered from that PDF so it can be read and searched.
  It has no speaker notes.
- **L15 onward** and later labs are not yet delivered. L14 says L15 is about
  "the wiki". Module 8 ends where the published decks end.

## The arc in one picture

```text
 Module 1        Module 2           Module 3            Module 4
 tokens,     ->  context/model/  -> who picks the   ->  the request,
 the window      prompt, specs      next action?        the stream,
                                    (the harness)       tool calls
                                                            |
       +----------------------------------------------------+
       v
 Module 5           Module 6             Module 7             Module 8
 six stop       ->  skills, and a    ->  shell, MCP,      ->  a subagent is
 conditions,        design doc that      browser, phone       the loop, started
 approval gates     survives a fresh     (same loop,          by a tool call
                    session              longer reach)
```

One sentence runs through all eight: **the model only writes text; everything
that happens in the world is code somebody wrote around it, and that code is
where the engineering is.**
