# Reuse vs Build

## Decision

Trade With Ease should reuse commodity, regulated and mature infrastructure.

We should spend engineering effort on the components that differentiate the product:

- Merryl intelligence
- AI orchestration
- risk management
- decision tracking
- evaluation

---

# What We Reuse

## Merryl Core

### How

Keep Merryl's existing:

- ingestion
- scoring
- ranking
- historical storage
- backtesting
- reporting
- dashboard

### Why

Merryl is already our primary market-intelligence asset.

Rebuilding it provides little value.

---

## Alpaca

### How

Continue using Alpaca market-data capabilities already consumed by Merryl.

Use existing paper/development trading capabilities where appropriate.

### Why

We do not need to build exchange-data infrastructure.

---

## FRED

### How

Continue ingesting macroeconomic series.

### Why

FRED already provides authoritative macroeconomic data.

---

## SEC EDGAR

### How

Continue retrieving official company filings.

### Why

It is the primary source for U.S. public-company filings.

---

## Alpha Vantage

### How

Continue consuming structured earnings/event information where Merryl already depends on it.

### Why

Maintaining an earnings-data service is not our differentiation.

---

## LLM Providers

### How

Use existing models for:

- language understanding
- reasoning
- tool selection
- explanations
- structured output

### Why

Training a general-purpose foundation model is outside the scope of Trade With Ease.

---

## MCP

### How

Expose Merryl functionality through MCP-compatible tools.

Consume external MCP-compatible capabilities where useful.

### Why

MCP gives us a standard AI-to-tool interface.

It also reduces dependence on a single model provider.

---

## Interactive Brokers

### How

Use IBKR for:

- brokerage accounts
- authentication
- portfolio/account capabilities
- order handling
- market execution

Prefer official integrations where they satisfy our requirements.

### Why

Trade With Ease is not a broker.

We should not rebuild regulated brokerage infrastructure.

---

# What We Build

## Merryl MCP/API

Expose Merryl through stable structured contracts.

---

## AI Orchestrator

Coordinate:

```text
User intent
+
Merryl intelligence
+
Portfolio context
+
Risk policy
```

and produce understandable conclusions and structured proposals.

---

## Trade Proposal Model

Represent possible trades as strict typed domain objects.

Example:

```text
TradeProposal

symbol
side
quantity
order_type
limit_price
reason
```

---

## Deterministic Risk Engine

Own hard financial safety rules.

The LLM must not determine whether its own proposal is allowed.

---

## Trade Journal

Capture:

- evidence
- proposal
- risk decision
- user decision
- broker result
- outcome

---

## Evaluation Engine

Determine whether:

```text
Merryl + AI
```

actually performs better than:

```text
Merryl alone
```

---

## Product UI

Provide:

- Chat
- Market overview
- Opportunities
- Portfolio
- Trade proposals
- Journal
- Performance

---

# What We Deliberately Do NOT Build Initially

We do not initially build:

- our own LLM
- an exchange-data provider
- a macroeconomic database
- a filings database
- a brokerage
- exchange connectivity
- an IBKR MCP server when an official integration meets our needs
- autonomous live trading
- a complex multi-agent system

---

# Principle

> Reuse infrastructure. Own the intelligence workflow.
