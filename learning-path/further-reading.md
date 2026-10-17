# Further reading

Outside sources the modules refer to, grouped by module. The decks cite many
more in their own speaker notes and source slides; start there for anything
specific to a lecture.

## Throughout

- Dave Rensin, [Elephants, Goldfish and the New Golden Age of Software
  Engineering](https://drensin.medium.com/elephants-goldfish-and-the-new-golden-age-of-software-engineering-c33641a48874)
  (April 2026). The method `egm/` implements. Read it before Module 6.
- Anthropic, [Building effective agents](https://www.anthropic.com/engineering/building-effective-agents).
  Workflows against agents, and why the simplest design that works is
  usually right.
- Anthropic, [Effective context engineering for AI
  agents](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents).
  The context window as a budget, from the harness builder's side.

## Module 1: How a model sees code

- The two YouTube deep dives on L02's last slide (tokenizers and attention).
  Not assessed; the first half hour of the tokenizer one is the part that pays.
- Liu et al., [Lost in the Middle: How Language Models Use Long
  Contexts](https://arxiv.org/abs/2307.03172) (2023). Evidence that more
  context can make retrieval of the right fact worse.
- OpenAI, [tiktoken](https://github.com/openai/tiktoken). The tokenizer
  library behind L02's token IDs; useful for counting tokens yourself.

## Module 2: Context, model, prompt

- [Aider: repository map](https://aider.chat/docs/repomap.html). How the map
  in Lab01 and L03 is built and budgeted.

## Module 4: Inside the call

- MDN, [Using server-sent
  events](https://developer.mozilla.org/en-US/docs/Web/API/Server-sent_events/Using_server-sent_events).
  The event-stream format L07 reads line by line.
- OpenAI, [Function calling](https://platform.openai.com/docs/guides/function-calling).
  The tool-call shape most compatible servers copy.

## Module 5: Stopping and approving

- Marc Brooker, [Exponential Backoff And
  Jitter](https://aws.amazon.com/blogs/architecture/exponential-backoff-and-jitter/)
  (AWS Architecture Blog). Why many clients retrying at once need jitter.
- Stripe, [Idempotent requests](https://docs.stripe.com/api/idempotent_requests).
  Idempotency keys: how a payment API makes "retry after a timeout" safe.
- The sources on L11's own slides: Anthropic's engineering post on Claude
  Code auto mode (Mar 25, 2026) and OpenAI's post on Codex auto-review
  (Apr 30, 2026), for the measured numbers in Module 5.

## Module 6: Skills and design before code

- [Agent Skills specification](https://agentskills.io/specification) and
  [clients](https://agentskills.io/clients). The open `SKILL.md` format and
  the harnesses that read it.
- [egm/anatomy-of-a-skill.md](../egm/anatomy-of-a-skill.md) and the five
  [walkthroughs](../egm/walkthroughs/) in this repository.

## Module 7: Reaching the computer

- [Model Context Protocol](https://modelcontextprotocol.io/), including the
  specification.
- Simon Willison, [The lethal trifecta for AI
  agents](https://simonwillison.net/2025/Jun/16/the-lethal-trifecta/) (June
  2025).
- OWASP, [Top 10 for Large Language Model
  Applications](https://owasp.org/www-project-top-10-for-large-language-model-applications/).
  Prompt injection and excessive agency, framed as security risks.
- [Playwright](https://playwright.dev/). The browser automation library
  behind Lab04's captures, and the tests Module 7 asks you to write.
- Chrome for Developers, [Native
  messaging](https://developer.chrome.com/docs/extensions/develop/concepts/native-messaging).
  The "helper program Chrome starts" path L13 describes.

## Module 8: Subagents and orchestration

- Git, [git-worktree](https://git-scm.com/docs/git-worktree). The mechanism
  behind a worktree per subagent.
- The documentation pages on L14's sources slide (read by the instructor on
  Oct 13, 2026), for Claude Code subagents, agent teams and Codex's
  `spawn_agent`.
