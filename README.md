# Open-source agentic venture builder

A team of AI agents that takes a founder's brief through six phases (Discover, Validate, Build, Commercial
setup, Launch, Operate and learn) toward a validated, sellable digital product. Gates between phases check
evidence, tests or founder approval, and the system recommends **stop** when the evidence is weak. Nothing is
published, sent or spent without the founder's approval, enforced in code.

## Quickstart

```bash
docker compose up
```

Then open http://localhost:8000, pick an example brief (or write your own) and press **Start**. The page shows
the one thing that needs you next: a gate decision, an approval, or numbers to enter.

- **Live models:** copy `.env.example` to `.env` and add a free Groq key (console.groq.com).
- **No key:** the app runs in replay mode (STUBBED) on recorded responses of the two example briefs
  (`planner_pdf`, `weak_idea`). Other briefs need a key.
- **CLI:** `python -m app.cli --brief evals/briefs/planner_pdf.json` (add `--auto` for an unattended demo; its
  approvals are logged as `auto-demo`, never as `founder`).
- **Tests:** `pytest` (191 tests, no network, no key).

## What works and what is mocked

| Phase / step | Status |
|---|---|
| 1 Discover: sharpen idea, score alternatives, market, interview script | Working (live models + web search) |
| Web search (SearXNG in compose; Hacker News / GitHub / StackExchange fallback) | Working |
| Discover gate: rubric score (min 60) + constraint fit, recommends STOP on weak ideas, says why, suggests changes; founder can revise the idea and re-run | Working |
| 2 Validate: lean canvas, success target, landing page copy | Working |
| 2 Validate gate: assumptions register (every open ASSUMPTION from Discover and Validate, with how to verify) | Working |
| 2 Validate: landing page publishing | MOCKED (written to `outbox/` after approval, never posted) |
| 2 Validate: real sign-up numbers | Entered by the founder; logged as `founder_reported` |
| 2 Validate: customer interview notes (Mom Test tagging, weighed by the gate) | Working; notes entered by the founder, skipped when none |
| 3 Build: spec, product file pack, user docs; static tests gate | Working |
| 4 Commercial setup: sourced competitor price benchmark, pricing, policies (research, not legal advice); approval gate | Working |
| 5 Launch: store page, launch posts | MOCKED (outbox only, after founder approval) |
| 5 Payment, delivery, refund | MOCKED (`app/payments.py`, `delivery.py`, `refunds.py`; no real money) |
| 5 Reconciliation of orders, payments, fees, refunds, payouts | Working on the mocked ledger; fees ASSUMPTION (`outputs/payment_test_log.md`) |
| 6 Operate: support FAQ, metrics vs. target, funnel drop-off diagnosis, persevere or pivot | Working (metrics entered by the founder) |
| Founder approve / reject before anything goes out | Working, enforced in code (token tied to content hash) |
| Guard: never-rules, excluded categories, prompt-injection wrapper | Working (`evals/trick_prompts.yaml`) |
| Replay mode without a key | STUBBED (recorded responses in `replay/`) |

## Architecture

The orchestrator (`app/orchestrator.py`) is a deterministic Python state machine; no model decides control flow.
Six agent teams (`app/agents/<team>.py`, role prompts in `prompts/<team>.md`) own the 19 steps; each step is a
skill in `config/skills.yaml` with a task prompt in `prompts/skills/`, run by one generic skill runner
on free open-weight models through a router with an allowlist, token budgets and rate-limit fallback. A sourced claim
must cite a searched URL whose snippet actually states it, or it is downgraded to `ASSUMPTION`. Demand scores are
capped by the sources found, and the unit economics (break-even included) are computed in code, not by a model. Gates (`app/gates.py`) recommend go on, repeat,
pivot or stop; the founder decides, and the code refuses to go on past a failed or pending approval. Outgoing
items pass the Guard and a second-opinion critic before the founder sees them. See [ARCHITECTURE.md](ARCHITECTURE.md).

## Models and free resources

- **Models:** open-weight only, on Groq's free tier: `openai/gpt-oss-120b`, `openai/gpt-oss-20b`
  and `qwen/qwen3.8-27b` (all Apache 2.0). The router refuses any model not on the allowlist in `config/models.yaml`.
- **Search:** self-hosted SearXNG (in `docker-compose.yml`), with free public APIs as fallback.
- **No payment needed:** a free Groq key, or no key at all (replay mode).

## Cost per run

About **$0.00 per run**. The free tier costs nothing. Measured on the recorded six-phase run: about 32k tokens
over 20 model calls, taking about 5 minutes (mostly rate-limit waits). A weak idea stops at Discover after about
8k tokens. The per-run budget cap is 150k tokens.

## Demo video

Link to be added before the final push.

## What our system will NOT do

- Promise income or success.
- Create fake reviews, testimonials, sign-ups or countdown timers.
- Pose as the founder, a customer or another brand.
- Hold, move or refund money by itself, create accounts, or store passwords or card numbers.
- Publish, send or spend anything without founder approval.
- Build physical goods, financial, investment or crypto products, or health or legal advice products.
- Hide core features behind a paywall: everything here is MIT licensed.

## How the project stays alive after the hackathon

- **License:** MIT.
- **Easy to extend:** skills are plain YAML plus Markdown prompts, so adding a product type or step needs no
  orchestrator change.
- **Roadmap:** real search backends, more product types, local models through Ollama, and an export of the
  approved store page and file pack that the founder uploads to their own store themselves. Payments stay mocked
  in this project: no payment code of any kind, not even sandbox.
- **Community:** contributions through issues and pull requests.
- **Funding:** none needed; it runs on free tiers.
