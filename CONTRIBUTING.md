# Contributing to Trade With Ease

Thanks for helping build Trade With Ease.

This project combines AI, financial data, market intelligence and eventually brokerage capabilities.

Because of that, correctness, security and clear architectural boundaries matter.

---

# Before Coding

Please read:

1. `README.md`
2. `docs/architecture.md`
3. `docs/reuse-vs-build.md`
4. `docs/trade-flow.md`
5. `docs/security-and-risk.md`
6. `docs/roadmap.md`

---

# Current Priority

The first engineering milestone is:

> **Merryl MCP/API**

Avoid adding live brokerage execution until the project reaches that roadmap phase.

---

# Development Workflow

1. Pick or create an issue.
2. Discuss architecture-impacting changes first.
3. Create a focused branch.
4. Implement the smallest useful change.
5. Add/update tests.
6. Run formatting and linting.
7. Open a pull request.
8. Explain what changed and why.
9. Get another contributor to review security-sensitive changes.

---

# Branch Naming

Examples:

```text
feat/merryl-market-regime-tool

feat/merryl-watchlist-tool

feat/stock-analysis

feat/risk-engine

fix/stock-analysis-validation

docs/risk-boundaries
```

---

# Engineering Principles

## Prefer explicit domain types

Avoid passing loosely structured JSON throughout the application.

## Treat AI output as untrusted

LLM output must be validated.

## Keep deterministic rules outside prompts

Financial risk policy belongs in application code.

## Keep Merryl broker-independent

Merryl is market intelligence, not brokerage infrastructure.

## Protect secrets

Never expose secrets to:

- prompts
- logs
- source code
- commits

## Fail safely

If required data or an external dependency fails, the system should not assume success.

## Make decisions auditable

Important decisions should be explainable and traceable.

## Avoid premature microservices

Prefer a modular monolith until operational requirements justify splitting services.

---

# Rust Guidelines

For Rust components:

```bash
cargo fmt --check
cargo clippy
cargo test
```

Public domain contracts should be documented.

Avoid panics in runtime/request paths where errors can be represented explicitly.

---

# Pull Requests

A pull request should explain:

## Problem

What problem are we solving?

## Proposed Change

What changed?

## Why

Why was this approach selected?

## Security / Risk Impact

Does this affect:

- market data
- AI behavior
- risk validation
- account information
- trading
- authentication
- secrets

## Testing

How was the change tested?

## Follow-Up

What work remains?

---

# Security

Never commit:

- API keys
- broker credentials
- access tokens
- refresh tokens
- private keys
- real brokerage account identifiers
- personal financial information

Security-sensitive vulnerabilities should be reported privately to the maintainers rather than disclosed publicly.

---

# Core Principle

> **AI reasons. Code controls risk. The user controls live execution.**
