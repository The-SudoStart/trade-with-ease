# Security and Risk

Trading creates a higher-consequence boundary than ordinary AI chat.

Therefore safety-critical rules must exist in deterministic application code.

---

# Core Rule

> **The LLM may propose. It may not bypass risk policy.**

---

# Trust Boundaries

## Merryl

Merryl is responsible for calculated market intelligence.

It should have no brokerage credentials.

---

## LLM

Treat model output as untrusted input.

Every actionable output must be:

- structured
- parsed
- validated
- checked against policy

---

## Risk Engine

The Risk Engine is authoritative for platform risk policy.

It must not depend on the LLM deciding whether a rule should apply.

---

## Broker

The broker is authoritative for:

- account state
- available funds
- broker restrictions
- order acceptance
- execution
- fills

---

## User

The user remains the final authority for live execution in the initial system.

---

# Initial Risk Controls

Candidate controls include:

```text
maximum trade value

maximum position percentage

maximum sector exposure

maximum daily loss

maximum portfolio drawdown

allowed asset classes

margin policy

short-selling policy

options policy

event/earnings restrictions

market-hours policy

global kill switch

mandatory live-trade confirmation
```

Exact thresholds are policy/configuration decisions.

The LLM must never invent or modify them.

---

# Execution Integrity

Authorization must eventually bind to the exact proposal.

For example:

```text
User
Account
Symbol
Side
Quantity
Order type
Limit price
Risk decision
Expiration
```

If any meaningful property changes, a new risk check and authorization are required.

---

# Operational Requirements

Live execution should eventually include:

- idempotency
- no broker secrets in prompts
- no secrets in logs
- encrypted secrets where storage is unavoidable
- tenant isolation
- audit logs
- final risk validation
- broker reconciliation
- kill switch

---

# AI-Specific Risks

We must explicitly test:

## Hallucinated instruments

The model proposes an invalid or incorrect symbol.

## Hallucinated prices

The model uses a price that does not correspond to current market information.

## Invalid quantities

Negative, zero or unreasonable quantities.

## Stale context

The model reasons using outdated information.

## Prompt injection

News articles, filings or external text attempt to manipulate agent instructions.

## Risk-rule override

A prompt attempts to convince the model or system to ignore policy.

## Replay

An old authorization is reused.

## Duplicate execution

The same trade is submitted multiple times.

## Partial tool failure

One source fails while the model assumes it succeeded.

## Portfolio inconsistency

The portfolio changes between analysis and execution.

---

# Development Policy

Development progresses in this order:

```text
Analysis
 ↓
Paper Trading
 ↓
Evaluation
 ↓
Live Trading
```

Live execution is the final stage, not the first.
