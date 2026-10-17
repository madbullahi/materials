# Module 4: Inside the call

**Phase:** Analyze · **Time:** two to three sittings

## Sources

| Read | For |
|---|---|
| [L07](../lectures/L07/slides.md) | The request object, the URL rule, finish reasons, usage, reasoning tokens, the stream as events, the failed-stream rule, tool definitions and their cost, the round trip, the loop on the wire |
| [L08](../lectures/L08/slides.md) | A strong hosted model on a 32K budget, one complete request in Python, what a session costs |
| [Lab02](../labs/Lab02/slides.md) | Hackathon 1 post-mortem: fixing, designing and iterating, and two students with the same brief and different evenings |

## You will be able to

1. **Write** a chat-completions request by hand and **explain** why it has no
   session field.
2. **Handle** all three finish reasons (`stop`, `length`, `tool_calls`) and
   say what each obligates your program to do.
3. **Parse** a server-sent-event stream, including the events that carry no
   answer text, and **apply** the failed-stream rule.
4. **Calibrate** a token estimator against the server's `usage` numbers.
5. **Complete** a tool-call round trip correctly, including the assistant
   message that most first implementations forget.
6. **Explain** the difference between fixing, designing and iterating with an
   agent, and **keep** the decisions yours in each.

## Before you start

Module 3. You should be able to write the six-line loop and say what
`execute` does.

## The ideas

### 1. One POST is the whole interface

```json
POST {base_url}/chat/completions
{"model": "...", "messages": [...], "stream": true}
```

`messages` is a list of `system`, `user`, `assistant` (and later `tool`)
entries. Your files go inside a message, because there is no field for files.
There is **no session field** either: nowhere to put a memory. Everything the
model knows about earlier turns is in that list because your program put it
there, which is why history trimming is your job.

Three details from L07 that save days:

- **The URL rule.** Strip one trailing slash from `base_url`, append
  `/chat/completions`, nothing else. A program that follows it can be pointed
  at any compatible server by editing one line.
- **Authorization only when there is a key.** No key means no header, not an
  empty one. And the key never appears in an error, a log line or a commit.
- **`temperature` and `max_tokens`.** The first scales the sampling
  distribution (servers default to non-zero). The second caps the reply and
  is a cheap way to make a test deterministic.

The request shape is shared by many servers and vendors because one vendor's
API became the shape everyone copied. It frays at the edges: where reasoning
appears, whether tool arguments arrive whole, whether usage appears on a
stream.

### 2. Every reply ends for a reason

| `finish_reason` | Your obligation |
|---|---|
| `stop` | The model ended it. Only now may you look for edit blocks |
| `length` | It was cut off. An explicit error, never an edit |
| `tool_calls` | It is asking you to act. Empty `content` is not "done" |

A reply cut at 400 tokens can look complete and contain a whole edit block.
Applying it is far worse than printing the wrong message.

### 3. Usage is ground truth; reasoning is real tokens

The `usage` field counts what the server actually charged. Your estimator
predicts; this counted. Send one request, compare, adjust. Expect a fixed
per-message floor (chat-template markers) that a characters-per-token
estimate cannot see, so err high.

Reasoning models spend tokens thinking before answering. In L07's capture a
14-token answer billed 465 tokens. Where those tokens arrive differs by server
(a separate `reasoning` field, or `<think>` tags inside `content`), and you
must decide: does the user see it, does it go into history, and does an edit
block inside the reasoning count as a proposal? (It should not: the model
drafts and discards there.)

### 4. The stream is events, and it can die

With `stream: true` the reply arrives as server-sent events: `data:` lines,
each with a `delta`. L07 lists six kinds: an opening delta, reasoning-only
deltas (hundreds of them), content deltas, an empty delta carrying the finish
reason, a usage-only event with no `choices`, and `[DONE]`, which is not JSON.
Call `json.loads` on every line and you crash on the last one.

A stream that dies at event 400 has already printed most of a reply. The
**failed-stream rule**: record no assistant turn (or the next request carries
a half reply the model will continue from), treat nothing in it as an edit,
tell the user, stay usable so they can retry.

One more silent failure: some local servers keep a small default context and,
when a request is longer, **drop the start of it** without any error. The
start is your system prompt and first file. It looks exactly like a stupid
model. The fix is server configuration and an estimator that errs high.

### 5. A tool call is a request, and it costs tokens on every turn

A tool has four parts, and only the first is written for the model:

| Part | Whose |
|---|---|
| Name and description | Read by the model to choose |
| Argument contract | Your code, checking the request |
| Dispatch | Your code, doing it |
| Observation | The next message in the list |

Describing a tool does not give the model the tool. It gives it the ability
to ask. Definitions are rendered into the prompt as text: in L07's capture,
three tools raised a 71-token request to 471, and most of that was a fixed
block for having tools at all. **Deciding to give an agent tools is the
expensive choice, and it is made once.**

When the reply has `tool_calls`: `arguments` is a **string** of JSON the model
wrote (it can fail to parse, violate the schema, or name a path outside the
repo); `id` is how the result finds its way back; and it is a **list**. The
round trip is two appends, not one: first the assistant message exactly as
received, then a `tool` message carrying the result and the `tool_call_id`.

### 6. The trace L07 ran, and what it shows

A 9B model asked to add a flag to `taskr` read the right three files, wrote
them in dependency order, and then ran `rm -f ~/.taskr.json` twice, the
user's real task list, outside the repository and invisible to git. Its
context grew sevenfold in nine steps. It ended because the loop ran out of
turns, not because the work was done. Almost nothing that failed was the
model. What failed was everything around it: nothing stood between `rm` and
`$HOME`, nothing recognised completion, nothing made a command's result real,
nothing capped the growing context. That table is the syllabus for Module 5.

### 7. Fixing, designing, iterating

Lab02 compares two students with the same brief, the same model and the same
two hours. One wrote a one-page design in their own words, built a piece,
**ran the program as the user** after each piece, and pointed the agent at
specific decisions when something broke: working at 9 PM, 16 prompts. The
other had the agent write the design, asked the agent "am I done?" instead of
running the program, and answered a crash with "fix it" six ways: still
crashing at 9 PM, 77 prompts. Both followed the process on paper. The
difference was who kept the decisions, the looking, and what to tell the
agent.

## In the real world

**Payments.** The failed-stream rule is the same rule payment systems apply
to messages from a partner bank: a response that did not finish is not a
response. If an assistant that drafts settlement adjustments applied a
truncated reply, it could post half a correction. The same team hits the
tool-call round trip when an agent asks for three account lookups in one
reply: answering the first and silently dropping the other two leaves the
model reasoning about balances it never saw.

**Clinic.** The request object makes the privacy question concrete: anything
in `messages` leaves the machine on a hosted endpoint. The team's harness
redacts identifiers before a file goes into a message, and logs the
`prompt_tokens` of every request so a sudden jump (someone added a whole
export file) is visible.

**Logistics.** The on-call agent's run log records `finish_reason` and
`usage` for every call. When a run "did nothing", the log shows the last reply
ended in `length`, so the fix was a budget, not a better model.

**Anywhere a bill exists.** L08's arithmetic is the habit: a scripted evening
cost $0.24, and the one way to spend a whole $25 key is a loop with no stop
condition that resends a growing conversation. Every team with a hosted model
needs the same back-of-envelope estimate before it runs anything unattended.

## Exercises

### Core: a client with no model behind it

Write a small Python module (standard library only) that L07 says you can
fully test without an endpoint:

1. `build_url(base_url)` implementing the URL rule, with a test for each row
   of L07's table.
2. `parse_stream(lines)` that takes an iterable of SSE lines and returns
   `(content, reasoning, finish_reason, usage)`. It must accept all six event
   kinds and raise a clear error if the stream ends without `[DONE]` or a
   finish reason.
3. `apply_reply(messages, reply)` that implements the failed-stream rule:
   if the reply did not finish with `stop` or `tool_calls`, `messages` is
   unchanged and an error is raised.
4. `tool_round_trip(messages, reply, results)` that appends the assistant
   message verbatim and then one `tool` message per call, in order, matching
   each `id`.

Make your model call a parameter with a default, as L07 recommends:
`def ask(messages, call=send_request)`. Your tests pass a fake `call`.

**What you are practising:** the parts of the contract that cause real bugs,
tested with no network and no flakiness.

### Stretch

- **Calibrate.** If you have a model to call, send one request with
  `stream: false`, read `usage.prompt_tokens`, and compare it with a
  characters-divided-by-four estimate of your `messages`. How far off was it,
  and in which direction?
- **Price the tools.** Send the same messages with and without a `tools`
  field of three definitions. What did the tools add per request? At that
  rate, how many tools fit in a 24,000-token budget alongside one 6,000-token
  file?
- **Replay Lab02.** Take a small feature for `taskr` or your own project.
  Spend the first 15 minutes writing a one-page design in your own words, then
  build it in pieces, running the program as the user after each. Keep a log
  like Lab02's "iterating" table: time, what running it showed.

## Pitfalls

| Pitfall | Fix |
|---|---|
| Treating empty `content` as "no answer" | Branch on `finish_reason`. Empty content with `tool_calls` means it is asking |
| Appending the tool result without the assistant message | Two appends: the reply as received, then the result |
| `json.loads` on every `data:` line | `[DONE]` is not JSON. Handle it first |
| Keeping a half-finished stream in history | Record no assistant turn on a failed stream |
| Logging the request headers while debugging | The key is in them. Never log or commit it |
| Asking the agent whether you are done | Run the program as the user. That is the evidence |

## Self-check

1. Your program has printed forty words and the stream dies. What is in
   `messages` afterwards, and what does the user see?
2. Why is there no session ID in the request, and what follows for your
   program?
3. A reply has `finish_reason: "length"` and contains a complete-looking edit
   block. What do you do?
4. Three tools cost 400 tokens per request. Why is the first tool much more
   expensive than the third?
5. In Lab02, both students "followed the process". What made the difference?

<details>
<summary>Answers</summary>

1. `messages` is unchanged: no assistant turn was recorded. The user sees what
   was printed, plus a clear message that the reply failed and they can try
   again. Nothing was applied as an edit.
2. Because continuity is not the server's job: the model only sees what you
   send. Your program must keep the history, trim or summarize it to fit the
   budget, and resend it every turn.
3. Treat it as an error. Do not apply the block: you cannot know the proposal
   was complete.
4. Declaring tools at all makes the chat template inject a fixed block that
   explains how to call them. L07 measured about 220 fixed tokens, with each
   extra tool adding about 30.
5. Who kept the decisions. Student A wrote the design, ran the program after
   each piece, and told the agent which side was wrong when something broke.
   Student B let the agent write the design, judge "done" and choose the
   fixes.

</details>

## Next

Module 5 adds what L07's trace was missing: named reasons for a run to end,
and a check in front of every action.
