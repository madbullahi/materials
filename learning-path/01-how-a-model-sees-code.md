# Module 1: How a model sees code

**Phase:** Apply · **Time:** two sittings, plus setup

## Sources

| Read | For |
|---|---|
| [Lab00](../labs/Lab00/slides.md) | Serving a model locally with Ollama and pointing Aider at it. Do this first; every exercise in the course needs it |
| [L01](../lectures/L01/slides.md) | The four waves of AI coding, the typist-to-curator shift, and why the course starts with Aider |
| [L02](../lectures/L02/slides.md) | Tokens, the context window, attention cost, hallucination, capability against knowledge |
| [demos/L01-notes-cli](../demos/L01-notes-cli/README.md) | The first hands-on loop: three prompts, three diffs to judge |

## You will be able to

1. **Explain** why a model's input is tokens rather than characters or words,
   and **estimate** how the way code is written changes its token cost.
2. **List** what is inside a model's context window in a coding session, and
   state the one rule about what is outside it.
3. **Explain** why the cost of attention grows with the square of the context
   length, and what that means for "just paste the whole repo in".
4. **Diagnose** a wrong answer as missing context, a hallucinated API, or a
   capability or knowledge limit.
5. **Run** one Aider session against a local model, and **judge** a diff
   before accepting it.

## Before you start

Nothing from the course. You need Python, git, and a machine that can serve a
small model or access to one that can (Lab00 covers both).

## The ideas

### 1. The model sees tokens

A tokenizer splits text into pieces it learned during training by merging the
most common adjacent pairs, over and over. `getOrderById` becomes four
pieces, `get`, `Order`, `By`, `Id`. A leading space is part of a token, so
`" order"` and `"order"` are different tokens, and punctuation can be glued to
a word.

Two consequences matter every day:

- **Tokens are the unit of cost.** Hosted models charge per token, local ones
  spend time and memory per token. L02's example: the same doubling function
  costs about 9 tokens written tersely and about 35 written verbosely.
- **Tokens are the unit of attention.** Every token in the window competes for
  the model's attention when it chooses the next one.

### 2. The context window is the only thing the model knows about your task

Inside the window: the system prompt, the files you added, the repo map, the
conversation so far, your message. Outside the window: nothing. There is no
database lookup and no web fetch inside the core call. Tools can fetch things,
but tools are code wrapped around the model (Module 3), not part of it.

### 3. Attention costs grow with the square

When the model writes one token, it scores it against every token already in
the window. Double the window and the work quadruples; 32 times the text is
about 1,000 times the work. And more text is not only slower, it is often
**worse**: the right fact gets diluted by everything around it. That is why a
long chat degrades. Fifty turns in, most of the window is dead weight the
model cannot tell apart from what is true now.

### 4. "Hallucination" is the design working as intended

The model picks a likely next token from a distribution. There is no
fact-check step. When your prompt is silent about something, training-data
priors fill the gap: your project uses `httpx`, the internet uses `requests`
far more, so the model writes `requests`. That is not a mistake in the model,
it is the right answer for the average project and the wrong one for yours.

The fix is almost always context: **when the model needs a fact, show it the
fact.** What context cannot fix is a fact that is in nobody's files, such as
the latest version of an internal library.

### 5. Capability and knowledge are different axes

**Capability** is how well a model reasons. **Knowledge** is what it saw in
training, frozen at a cutoff. A bigger model will not know a library released
after its cutoff; a model that knows the library may still be too weak for a
hard refactor. Mixing the two up leads to bad model choices: paying for a
larger model to solve what was a context problem.

### 6. From typist to curator

L01's frame for the whole course: the model writes most of the lines, and your
time moves to deciding what to ask, what to show it, and whether what came
back is right. If you can explain it, it is yours. If you cannot, it is not,
however well it runs.

## In the real world

**Clinic.** A developer asks a coding agent to "add the new allergy field to
the referral letter". The agent has the letter template in context but not the
data-access layer, so it guesses a column name that sounds right and is wrong.
Nothing crashes until a letter goes out without the allergy. The fix is not a
better model, it is putting the schema file in the window. And because this is
patient data, every file put in the window is a file sent to the model's host:
the context decision is also a privacy decision (Module 4 comes back to this).

**Open source.** A contributor pastes 40 files into a chat to "give the model
the whole picture" and gets a vaguer answer than when they pasted three. Every
added file costs attention the three relevant ones needed. The maintainers who
get good results add the files the change touches and the files those call,
and stop.

**Payments, as a counter-example.** The model "knows" a payment-provider SDK,
but from before the provider renamed a parameter. Capability is not the issue,
knowledge is. The team's fix is to keep the provider's current reference page
in the repo and add it to context for any change that touches the
integration.

**Anywhere.** Token cost is a budget line for any team using a hosted model.
Verbose code, long logs and long chat histories all cost money on every
request, not once.

## Exercises

### Core: run the L01 demo, and catch the quiet failure

1. Do Lab00 until `bin/check` passes in your `aider-ollama` clone.
2. In `demos/L01-notes-cli`, run `./reset`, then follow the README: `cd` to
   the scratch copy it prints and start `aider notes.py test_notes.py`.
3. Send the three prompts from the README, one at a time. After each one,
   read the diff **before** accepting it, then run `/run pytest -q` **and**
   `python3 notes.py` with no arguments.
4. Write two or three sentences: after prompt 1, did the green test suite tell
   you whether prompt 1 worked? (The README says why it might not.)

**What you are practising:** the curator loop. The model generates, you
judge, and "the tests pass" is evidence you still have to interpret.

### Stretch

- **Count tokens.** In Aider, run `/tokens` after each prompt and note which
  line is the largest. Then `/clear` and run it again. What moved?
- **Make it hallucinate on purpose.** In a scratch file, write a two-line
  Python file that imports `httpx` and calls one function, then ask Aider to
  "add a retry around the HTTP call" with only a *different* file in context.
  Does the answer use `httpx` or `requests`? Now add the file and ask again.
- **The 4B.** Run `./reset --model 4b` and send the same three prompts. Count
  the diffs you reject. Which lever (context, model, prompt) do you think each
  failure came from? Keep the list; Module 2 asks for it.

## Pitfalls

| Pitfall | Fix |
|---|---|
| Blaming the model first | Ask the three diagnostic questions in order: context, hallucination, capability or knowledge. Context is the most common cause |
| Pasting everything "to be safe" | More context is often worse, not just more expensive. Add what the change touches |
| Trusting a green suite | A suite that never calls the changed path cannot tell you it works. Run the program |
| A model that suddenly seems stupid | Check the context size the server allows. Lab00's `bin/check` exists because a model with a tiny default window silently truncates your files |

## Self-check

1. Why might `" order"` and `"order"` cost you different amounts of context?
2. A session was sharp for twenty turns and is now repeating ideas you
   rejected. What is happening inside the window, and what is the cheapest
   fix?
3. Your model writes `import requests` in a project that only uses `httpx`.
   Which of the three diagnostic questions does that answer, and what do you
   change?
4. Why does doubling the context more than double the work?
5. Give one task where a smaller model is the right choice.

<details>
<summary>Answers</summary>

1. They are different tokens with different IDs, because the leading space is
   part of the token. The model sees them as different pieces, and code
   formatting changes how many tokens a line costs.
2. History has filled the window with stale turns, including rejected ideas,
   and the model cannot tell which statements are current. Clear the history
   (`/clear` in Aider keeps your files) or start a fresh session with a short
   summary of the decisions.
3. It is a hallucination driven by a training prior, made possible by missing
   context: nothing in the window showed the project uses `httpx`. Add a file
   that shows it, or say so in the prompt.
4. Each new token is scored against every token already in the window, so the
   work grows with the square of the length: twice the tokens, four times the
   pairs.
5. A cheap, repeated batch job where a small model clears the bar (L02's
   example), or, while learning, any task where you want failures to show up
   where you made the mistake.

</details>

## Next

Module 2 turns these mechanics into three levers you control in every
session: context, model, and prompt.
