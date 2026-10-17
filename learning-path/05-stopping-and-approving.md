# Module 5: Stopping and approving

**Phase:** Analyze · **Time:** three sittings

## Sources

| Read | For |
|---|---|
| [L09](../lectures/L09/slides.md) | Done signals, the six stop conditions, where each check sits in the loop, the exit line, what `/undo` cannot walk back, the approval step, approval fatigue |
| [L11](../lectures/L11/slides.md) | Sorting failures, nested retries, a retry loop you own, why a timed-out tool is never retried, what the approval prompt cannot show, Codex and Aider approval, rules, reviewer models and typed classifiers as approval gates, measuring a gate |
| L12 (Oct 8) | **Not yet published.** L11 says the Analyze specification was handed out at the end of it. Check back in `lectures/L12/` |

## You will be able to

1. **Name** the six stop conditions, **explain** what each catches that the
   ones before it miss, and **place** each check at the right point in the
   loop.
2. **Design** a done signal and **say** what a reply that asks a question
   means when nobody is watching.
3. **Write** one exit path that records a distinct, named reason for every
   way a run ends.
4. **Sort** a failure into model call, your request, or tool, and **write** a
   retry loop that retries only the first, with backoff, jitter and a
   deadline.
5. **Explain** why a tool call is never retried by the harness, using the
   word idempotent.
6. **Build** an approval step (parse, boundary checks, person, dispatch or
   denial) and **design** a gate that answers the easy cases in code.

## Before you start

Module 4. You should know what a tool-call round trip looks like and why
every call gets exactly one `tool` message.

## The ideas

### 1. "No tool call" is the default done test, and it is weak

The six-line loop stops when a reply has no tool call. Two replies with no
tool call are not done: one that ended with `finish_reason: "length"`, and
one that asks a question back. With nobody at the keyboard, your design must
say what a question means: done, a separate "needs input" exit, or failure.

Better done signals: a `finish` tool the model calls on purpose (one more
tool definition, and small models sometimes call it early), or
**verification**, where the harness runs a check it trusts before accepting
the claim. L09's four rehearsal runs all ended "done"; one of four features
worked. One run's "test" imported a module and ran nothing, exited 0, and the
model believed it. A check the model chose is evidence of what it believes,
not of what works.

### 2. Six stop conditions: done, plus five limits

| Condition | Catches what the others miss |
|---|---|
| Task complete | The model signals done, in a way you defined |
| Step limit | The obvious runaway. The backstop |
| Token budget | A loop whose context is growing (one big read can add 10,000 tokens in one step) |
| Wall clock | A hung command: no request, so no step and no tokens are spent |
| Repeated call | The same call forever, the most common small-model failure |
| Consecutive errors | A run that has lost the plot and is not recovering |

A run that ends on a limit **has not finished**. It needs a different name in
the log from done. Each limit has a number you choose and at least one
decision inside it: does a step mean a request or a tool call? Does the token
budget protect the next request or the run total? Does a repeat after a write
count, or is that progress? Does `pytest` exiting 1 count as an error, or is
it information? Does a denial count?

Where each check sits follows from what it needs to see:

```python
while True:
    # before the request: step limit, token budget, wall clock
    reply = model.call(messages, tools=TOOLS)
    # after the reply: done, truncated
    for call in reply.tool_calls:
        # before executing: repeated call
        result = execute(call)
        # after the result: consecutive errors
```

Every exit goes through **one** handler that writes **one** exit line, e.g.
`{"event": "exit", "reason": "token_budget", "steps": 14, ...}`. A crash is
not an exit reason; it is a run with no exit line. Six is a minimum: add
`truncated` and `endpoint_error` too.

Wall clock is two mechanisms: a timeout per command and per model request,
plus a budget for the whole run. Close standard input on every command, and
set `GIT_EDITOR=true` so git never opens an editor.

### 3. Three kinds of failure, three different answers

| What failed | Examples | Answer |
|---|---|---|
| The model call | 429, 502, timeout | Nothing happened, so retry, with backoff |
| Your request | 400, 401, too long | Same bytes, same no. Stop and say what to fix |
| A tool | exit 1, no such file | Information for the model. Return it as the result |

Retries multiply across layers. L11 pointed Codex at an endpoint that always
returned 502 and counted 30 requests in 24 seconds: five outer reconnects plus
the first try, each with up to five inner retries. The OpenAI Python SDK
retries twice by default, so wrapping it in your own loop triples every try.
Build the client with `max_retries=0` and own one loop: retry only the
"nothing happened" statuses, back off by doubling with **jitter** (so thirty
clients do not all come back at the same instant), honour `Retry-After`, give
up on a **deadline** rather than a count, and exit non-zero with a reason.

### 4. A timeout is not a no

The harness killed `taskr add "buy milk"` after 30 seconds. Did it save? You
cannot know. A failed model call did nothing; a timed-out tool call may have
done everything. An operation is **idempotent** when doing it twice is the
same as once: reads are, `taskr add`, `git push`, sending an email and
charging a card are not. So the harness never retries a tool by itself. It
returns "timed out after 30 s; it may have run", and the model can check
before trying again.

### 5. The approval step, and what it cannot show you

The check goes between the argument contract and the dispatch:

1. Parse the arguments (invalid JSON is an error result; nobody is asked).
2. Boundary checks in code (a path outside the repository is refused without
   asking).
3. The person: allow or deny.
4. Dispatch, or return the denial, with the person's reason, as the call's
   result. A no is an answer, not an error. Every call still gets exactly one
   tool message.

Decisions inside it: show the **whole** call, never truncated or paraphrased,
and for a write, a diff against the file on disk. Never show the model's own
summary as if it were evidence. Decide whether reads ask (on a hosted model,
a read sends the file off your machine). Count waiting time separately from
working time.

`/undo` walks back a write to a tracked file. It walks back nothing for
`rm` outside the repo, `git reset --hard`, `pip install`, or `curl ... | sh`.
And the unit you approve is a **command in a state**: approving `pytest` at
step 12 approves whatever `conftest.py` the model wrote at step 11. A test
that calls the real `add` writes to the real task list though nothing in its
text mentions a home directory.

### 6. Approval gates: code first, then a model, then a person

People approve almost everything. L11 cites Anthropic's figure that users
accept 93% of permission prompts. By step 30 a prompt is a keypress, not a
decision. So something else answers most of them:

| Gate | Judges | Returns |
|---|---|---|
| Rules in code | The text of the call | Match or not. Deny wins over allow; unmatched asks |
| A reviewer model | Intent and effect | A decision and a reason |
| A typed classifier | Intent and effect | Probabilities your thresholds turn into allow, ask or deny |

Shipped systems layer them: code decides what it can decide for certain, a
model judges the rest, and a **middle band** goes to a person. Claude Code's
auto mode and Codex's auto-review (L11) both strip or fence the agent's own
explanation so it cannot argue its case, return denials as tool results, and
stop the run after three denials in a row. Every gate is a classifier, and
classifiers get **evaluated**: a labelled set of actions, the rate of safe
ones blocked, the rate of dangerous ones let through.

## In the real world

**Payments.** This module is the payment team's daily life. Card networks
and payment APIs solve "a timeout is not a no" with **idempotency keys**: the
client sends a unique key with each request, and a retried request with the
same key returns the original result instead of charging twice (Stripe's API
documents exactly this, see [further reading](further-reading.md)). An agent
that can call the transfer API needs the same rule: the harness never retries
a transfer, and the tool itself takes an idempotency key the model must
reuse. Their approval gate is code-first: any call that moves money is denied
unless it carries a ticket ID from a user message, and refunds over a
threshold always go to a person.

**Logistics.** The on-call agent runs at 2 a.m. with nobody to approve
anything. Its stop conditions are its safety net: a wall-clock budget so a
`kubectl logs -f` cannot hold it all night, a repeated-call detector so it
does not restart the same pod in a loop, and an exit line the morning shift
reads first. Its rules file allows read-only commands, denies anything that
changes production, and routes the middle band to a message on the on-call
phone (Module 7 shows how).

**Clinic.** "Do reads ask?" is not academic here. On a hosted model, reading
a fixture that contains real records sends them to the host. The clinic's
rules deny reads under `fixtures/real/` outright and allow reads of source
files, so the person is asked only about the cases in between.

**Anywhere with a shared API.** Exponential backoff with jitter is the
standard answer to rate limits in distributed systems generally, not only for
models; AWS's architecture blog has the classic analysis of why jitter
matters (see [further reading](further-reading.md)).

## Exercises

### Core: stop conditions and a manual approval prompt

Extend the loop you wrote in Module 3 (or start fresh with a fake model):

1. Add all six stop conditions, each with a number you choose and write down,
   plus `truncated` and `endpoint_error`. Route every exit through one
   exception type and one handler that prints one JSON exit line.
2. Write a scripted fake model for each condition that makes it fire, and a
   test per condition asserting the exit reason.
3. Add an approval step to `execute`: parse, refuse any path outside a given
   root without asking, show the full command or a diff, read `y`/`n` and a
   reason from the terminal, and return a denial as the tool result.
4. Write the `call_model` retry loop from L11 against a fake `send` that
   raises 502 twice and then succeeds, and another that raises 401. Assert
   the first succeeds after two waits and the second raises immediately.

**What you are practising:** turning "a run should stop" and "a person should
decide" into code with names you can find in a log.

### Stretch

- **Keep an approval diary.** For a week, every time an agent asks you to
  approve something, write it down with your answer and whether you read it.
  At the end, write the rules file that would have answered the most cases
  for you. Which cases can code not decide?
- **Price your retries.** For a tool you use (an SDK or a CLI), find out how
  many layers retry and with what defaults. How many requests can one failed
  user action become?
- **Evaluate a gate.** Write 20 labelled tool calls (10 safe, 10 not) for
  your agent. Run your rules file over them. Report safe-blocked and
  dangerous-allowed rates, as L11 does for the shipped gates.

## Pitfalls

| Pitfall | Fix |
|---|---|
| Logging "stopped" | Every exit gets a distinct name. "Stopped" tells you nothing next week |
| Retrying a 401 | It is your request. Retrying burns time and gets the same no |
| Retrying a tool that timed out | Return the timeout; let the model check before repeating |
| Two hidden retry layers | Set the SDK's retries to zero and own one loop |
| Approving the explanation | Approve the command, in its state. The reason was written by the same model |
| A wall clock checked only between steps | It cannot interrupt a hung command. Add per-command timeouts |

## Self-check

1. `pytest` exits 1 three times in a row while the model fixes a test. Which
   exit fires, and should it?
2. Your wall clock is 300 seconds and you took a ten-minute phone call at an
   approval prompt. What should your log say?
3. You approved `pytest` at step 12. What did you approve?
4. Why is a 502 safe to retry but a timed-out `taskr add` not?
5. Why do shipped gates hide the agent's own explanation from the reviewer?

<details>
<summary>Answers</summary>

1. Consecutive errors fires if you count non-zero exits as errors. Whether it
   should is your decision: a failing test is often information, not lost
   plot. Many designs count only "could not run" as an error, or reset on
   any write.
2. That the run was not killed by the wall clock: waiting time is recorded
   separately and does not count against the limit. The log shows working
   seconds and waiting seconds apart.
3. `pytest` plus everything written since you last read the repository,
   including any `conftest.py` and test files the model wrote. pytest runs
   them all.
4. A 502 means no reply came back, so nothing happened. A timed-out tool may
   have completed all or part of its work, and `add` is not idempotent, so a
   retry may duplicate it.
5. The explanation comes from the same model that chose the action, so it is
   not evidence. Anthropic reported that giving its classifier the agent's
   text and tool output made it worse on real overeager actions.

</details>

## Next

Module 6 changes what goes into the request: methods packaged as skills, and
a design document tested by a reader with no memory.
