# Five running cases

Every module asks the same question: where would this matter outside the
course? To keep the answers comparable, they always come from the same five
teams. They are composites, not real organizations, but each one is built from
problems that teams like it actually have.

Read this page once. Each module's **In the real world** section refers back
to the cases by name.

---

## 1. Payments: a mobile-money transfer service

A team runs the backend for a service that moves money between phone wallets,
with agents (shopkeepers) who cash in and out for customers. Thousands of
transfers a minute, a partner bank's API behind every one of them, and
regulators who want to know who approved what.

**Why it is a good test of the course:** almost nothing here is safe to do
twice. A transfer that timed out may have gone through. A retry that looks
harmless can pay someone double. An agent that "cleans up test data" in the
wrong place deletes real ledgers.

**Where it bites hardest:** retries and idempotency (Module 5), approval gates
(Module 5), what a command really touches (Modules 5 and 7).

---

## 2. Clinic: a patient-records system for a group of clinics

A small team maintains the records and appointments system for several
outpatient clinics. Patient data is regulated. Clinicians want help
summarizing histories and drafting referral letters, and the developers want a
coding agent on a codebase that is fifteen years old.

**Why it is a good test:** every file a model reads is sent somewhere. On a
hosted endpoint, a `read_file` of a fixture with real patient data has left
the building. And a confident wrong answer about a dosage is worse than no
answer.

**Where it bites hardest:** what goes into the context and where it goes
(Modules 1, 2 and 4), hallucination and knowledge versus capability
(Module 1), whether reads should ask (Module 5), prompt injection through
documents (Module 7).

---

## 3. Campus: a university student-services portal

A university IT unit runs the portal students use for course registration,
fee payment, timetables and results. Most of it is an old web application
with no public API. Staff want an assistant students can message, and the
developers want to test every change against the real pages before the
registration rush.

**Why it is a good test:** the work happens in browsers and on phones, not in
a terminal. Load arrives in spikes (registration week). Students will try to
talk the assistant into things.

**Where it bites hardest:** browser automation and "check what you built"
(Module 7), assistants you can text and who is allowed to text them
(Module 7), a spec a fresh reader can build from (Modules 2 and 6).

---

## 4. Logistics: an on-call platform team for a delivery company

A platform team keeps a delivery company's services running: route planning,
driver apps, warehouse scanners. Its continuous-integration pipeline runs
thousands of builds a day, and someone is on call every night.

**Why it is a good test:** this is where agents run with nobody watching. CI
fails at 2 a.m., an agent investigates, and the question is whether it stops
when it should and asks before it pushes.

**Where it bites hardest:** stop conditions (Module 5), triggers and the
heartbeat (Module 7), streams that die part way (Module 4), splitting an
investigation across subagents (Module 8).

---

## 5. Open source: a maintainer of a large Python tool

One maintainer and a handful of contributors look after a widely used
open-source tool, about 20,000 lines of code plus tests, written over years by
dozens of people. New contributors arrive every week, and most of them have
never read the code.

**Why it is a good test:** this is the course's own situation from Lab03 on,
when the projects moved into the real Aider codebase. Understanding code you
did not write comes before changing it.

**Where it bites hardest:** mapping a codebase (Module 6, `peanuts`), design
before code and the Goldfish test (Module 6), reference points in a long
build (Module 3), worktrees and parallel features (Module 8).

---

## How the cases line up with the modules

| | Payments | Clinic | Campus | Logistics | Open source |
|---|:-:|:-:|:-:|:-:|:-:|
| 1. How a model sees code | | ● | | | ● |
| 2. Context, model, prompt | ● | ● | ● | | ● |
| 3. Harness and autonomy | ● | | | ● | ● |
| 4. Inside the call | ● | ● | | ● | |
| 5. Stopping and approving | ● | ● | | ● | |
| 6. Skills and design | | | ● | | ● |
| 7. Reaching the computer | ● | ● | ● | ● | |
| 8. Subagents | | | | ● | ● |

A filled dot means the module has a worked example for that case. Every module
mentions more of them than it works through.
