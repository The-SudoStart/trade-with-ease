# Trade With Ease

Trade With Ease is an AI-assisted investing and trading platform built around **Merryl's market intelligence**.

The goal is not to let an LLM trade freely.

The goal is to combine:

- structured market intelligence
- AI reasoning
- portfolio context
- deterministic risk controls
- human authorization
- existing brokerage infrastructure
- continuous evaluation

into one transparent investment workflow.

---

## Core Idea

```text
Market Data
    ↓
Merryl
    ↓
Structured Market Intelligence
    ↓
AI Reasoning
    ↓
Trade Proposal
    ↓
Deterministic Risk Engine
    ↓
Human Authorization
    ↓
Broker
    ↓
Market
    ↓
Trade Journal
    ↓
Evaluation
```

Each component has a clear responsibility.

### Merryl

> What is happening in the market and what is worth investigating?

### AI Agent

> Given Merryl's evidence, portfolio context and the user's request, what action may make sense and why?

### Risk Engine

> Is this proposed action allowed under our deterministic rules?

### User

> Do I want to authorize this trade?

### Broker

> Execute the authorized trade and report what happened.

---

# What Already Exists

Merryl already provides much of the intelligence foundation.

It currently includes:

- market regime analysis
- sector rotation
- industry/theme analysis
- stock leadership
- relative strength
- volume analysis
- catalyst context
- ranked watchlists
- historical storage
- backtesting
- daily reports
- dashboard

Merryl remains the **market-intelligence engine**.

Trade With Ease builds the decision and trading workflow around it.

---

# What We Reuse

We intentionally reuse mature infrastructure instead of rebuilding it.

### Market and Context Data

- Alpaca
- FRED
- SEC EDGAR
- Alpha Vantage

### AI

Existing LLM providers such as OpenAI.

### AI Integration

Model Context Protocol (MCP).

### Brokerage

Interactive Brokers and other supported broker infrastructure.

### Development Trading

Existing paper-trading infrastructure where appropriate.

See:

`docs/reuse-vs-build.md`

---

# What We Build

Our differentiated work consists of:

1. Merryl Agent Interface (MCP/API)
2. AI Orchestrator
3. Typed Trade Proposal model
4. Deterministic Risk Engine
5. Trade Journal
6. Evaluation Engine
7. Chat/Product UI
8. Broker-specific integrations only when existing integrations cannot satisfy a verified requirement

---

# First Milestone

The first engineering milestone is:

> **Build the Merryl MCP/API.**

Before connecting brokerage accounts, an AI agent must be able to query Merryl through stable structured tools.

Initial tools should include:

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

---

# Documentation

Read these before contributing:

- `docs/architecture.md`
- `docs/reuse-vs-build.md`
- `docs/trade-flow.md`
- `docs/security-and-risk.md`
- `docs/roadmap.md`
- `CONTRIBUTING.md`

---

# Guiding Principle

> **AI reasons. Code controls risk. The user controls live execution.**

---

# Project Status

Trade With Ease is currently in its architecture and early implementation stage.

The immediate engineering objective is:

**Merryl → MCP/API → AI Agent**

Live brokerage execution comes later.
