# Trade With Ease Architecture

## Purpose

Trade With Ease connects Merryl's market intelligence to an AI-assisted investment workflow.

The architecture deliberately prevents an LLM from having unrestricted control over brokerage execution.

---

# High-Level Architecture

```text
Alpaca Market Data ─┐
                    │
FRED ───────────────┤
                    │
SEC EDGAR ──────────┼──────► MERRYL
                    │           │
Alpha Vantage ──────┘           │
                                │
                         Market Intelligence
                                │
                                ▼
                         Merryl MCP/API
                                │
                                ▼
                         AI Orchestrator
                           ▲          ▲
                           │          │
                         User      Portfolio
                           │        Context
                           │          │
                           └────┬─────┘
                                │
                                ▼
                         Trade Proposal
                                │
                                ▼
                     Deterministic Risk Engine
                                │
                         Approve / Reject
                                │
                                ▼
                        Human Authorization
                                │
                                ▼
                         Broker Integration
                                │
                                ▼
                              IBKR
                                │
                                ▼
                              Market
                                │
                                ▼
                         Trade Outcome
                                │
                     ┌──────────┴─────────┐
                     ▼                    ▼
               Trade Journal       Evaluation Engine
```

---

# 1. Merryl

Merryl remains a market-intelligence system.

It owns:

- market regime
- sector rotation
- industry/theme strength
- stock leadership
- stock rankings
- relative strength
- volume evidence
- catalyst context
- watchlists
- historical signal storage
- backtesting
- reports
- dashboard

Merryl answers questions such as:

> Where is money moving?

> Which sectors are gaining strength?

> Which stocks are showing leadership?

> Why is a stock appearing on the watchlist?

> How have similar signals historically performed?

Merryl should NOT own:

- IBKR credentials
- brokerage authentication
- live orders
- portfolio execution
- unrestricted trade placement

---

# 2. Merryl MCP/API

The Merryl Agent Interface exposes Merryl intelligence through stable structured tools.

Examples:

```text
get_market_regime()

get_sector_rankings()

get_industry_rankings(sector)

get_watchlist()

get_stock_analysis(symbol)

get_catalysts(symbol)

get_signal_history(symbol)

get_backtest_results(...)

explain_score(symbol)
```

This separates Merryl's internal implementation from AI consumers.

An AI application should not need direct database access.

---

# 3. AI Orchestrator

The AI Orchestrator handles conversational reasoning.

Responsibilities include:

- understand user intent
- determine which tools are required
- query Merryl
- obtain portfolio/account context
- combine evidence
- explain conclusions
- construct structured trade proposals

The AI does NOT have final authority over risk or live execution.

---

# 4. Trade Proposal

AI conclusions must be converted into typed domain objects.

Example:

```json
{
  "symbol": "NVDA",
  "side": "BUY",
  "quantity": 5,
  "order_type": "LIMIT",
  "limit_price": 190
}
```

Free-form AI text must never directly become a brokerage order.

---

# 5. Risk Engine

The Risk Engine is deterministic application code.

It evaluates proposed trades against rules such as:

- maximum position size
- maximum sector exposure
- maximum trade value
- daily loss limits
- portfolio drawdown
- event risk
- permitted assets
- margin rules
- short-selling rules
- options rules
- kill switch

The LLM cannot override the Risk Engine.

---

# 6. Human Authorization

For the initial live-trading system, a human remains the final authorization point.

The user must see the exact proposed action before live execution.

Changing the proposal after authorization requires a new validation and authorization cycle.

---

# 7. Broker Integration

The broker layer provides access to:

- account information
- cash
- positions
- buying power
- open orders
- trade instructions/orders
- execution status
- fills

We should reuse official broker integrations whenever possible.

A custom broker abstraction should only be introduced when there is a verified requirement.

A future internal abstraction may look like:

```text
BrokerPort
    │
    ├── PaperBroker
    ├── InteractiveBrokers
    └── FutureBroker
```

---

# 8. Trade Journal

Every meaningful decision should be recorded.

Examples:

```text
Merryl signal

Market regime

Sector state

AI proposal

AI explanation

Portfolio state

Risk decision

User decision

Broker instruction

Execution result

Trade outcome
```

This creates an audit trail and an evaluation dataset.

---

# 9. Evaluation Engine

The system must measure whether AI actually improves Merryl.

The primary experiment is:

```text
Merryl alone

       VS

Merryl + AI
```

Possible metrics include:

- total return
- relative return
- win rate
- expectancy
- Sharpe ratio
- maximum drawdown
- profit factor
- performance versus SPY

AI complexity should only be retained when evidence justifies it.

---

# Architectural Principles

1. Keep Merryl broker-independent.

2. Keep broker credentials away from Merryl and the LLM.

3. Treat model output as untrusted input.

4. Use strict domain types.

5. Validate every trade proposal.

6. Keep risk rules deterministic.

7. Require human authorization for initial live trading.

8. Journal important state transitions.

9. Prefer a modular monolith initially.

10. Split services only when operational requirements justify it.

---

# Core Principle

> **AI reasons. Code controls risk. The user controls live execution.**
