# Resale Support Agent

A customer support resolution agent for a second-hand fashion shop (clothes, shoes, bags), built with the Claude Agent SDK. The agent handles returns, refunds, and disputes against a synthetic backend, with a hard target: **80%+ first-contact resolution while knowing when to escalate to a human.**

> **Status: in progress.** Dataset and test suite complete. Next: MCP-style tools, agent loop, eval harness.


## Design principle: the dataset is the test suite

Every record in the database exists for a reason. Fifteen hand-designed edge cases each target one branch of the agent's decision logic, and each maps 1:1 to a test case with an expected outcome:

| Expected action | Cases | Examples |
|---|---|---|
| `resolve_refund` | 4 | clean return; boundary test (28 days, £95); exchange request on one-of-one stock; wrong item sent overriding final-sale |
| `resolve_decline` | 4 | outside 30-day window; final-sale; hygiene-excluded swimwear; duplicate refund attempt |
| `resolve_partial` | 1 | condition dispute on a £30 item (goodwill partial refund) |
| `escalate` | 5 | refund above £100 authority; condition dispute above £50; authenticity claim (always); delivery dispute; suspended account |
| `clarify` | 1 | nonexistent order number — agent must ask, not hallucinate |

Edge cases and test cases are generated from a single source in the notebook, so they can never drift out of sync. Thresholds live in `policy.json`; cases are defined relative to them, so the suite survives policy changes.

The escalation cases are not failures — a correct escalation beats an incorrect resolution. The eval harness (coming) will score FCR, escalation accuracy, and incorrect resolutions separately.

## Repo structure

```
build_dataset.py   # generates shop.db, test_cases.json, policy.json — also the reset button
test_cases.json       # 15 cases: customer message + expected action + rationale
policy.json           # shop policy the prompt and tools both read (incl. precedence rules)
shop.db               # SQLite backend — generated, gitignored; rebuild via the notebook
```

## Roadmap

- [x] Synthetic dataset + policy + test suite
- [ ] Backend tools: `get_customer`, `lookup_order`, `process_refund`, `escalate_to_human` — with policy guardrails enforced in the tool layer, not just the prompt
- [ ] Agent loop (stop_reason-controlled, tool results fed back into context)
- [ ] Eval harness: run all 15 cases, score by tool calls made, not by parsing replies
- [ ] Results + failure analysis in this README

## Stack

Python · SQLite · Claude Agent SDK · Anthropic API
