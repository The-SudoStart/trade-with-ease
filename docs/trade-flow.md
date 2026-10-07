# Trade Flow

## Goal

A conversational request can become a safe, structured and auditable trade proposal without allowing model output to bypass deterministic controls.

---

# Main Actors

## 1. User

Initiates intent and authorizes live execution.

## 2. Chat/UI

Provides the conversational interface.

## 3. AI Orchestrator

Interprets the user's request and coordinates tools.

## 4. Merryl

Provides market intelligence.

## 5. Risk Engine

Applies deterministic rules.

## 6. Broker Integration

Provides account and brokerage capabilities.

## 7. Interactive Brokers

Provides actual brokerage and market execution.

---

# Example

User says:

> Buy 5 shares of NVDA at a $190 limit.

The system should NOT immediately send this to the broker.

Instead:

```text
User
 │
 ▼
Chat/UI
 │
 ▼
AI Orchestrator
 │
 ├──────────────► Merryl
 │                  │
 │                  ├── Market regime
 │                  ├── Sector strength
 │                  ├── Stock analysis
 │                  ├── Catalysts
 │                  └── Historical evidence
 │
 └──────────────► Portfolio/Broker Context
                    │
                    ├── Cash
                    ├── Positions
                    ├── Exposure
                    └── Open orders

             AI combines evidence
                    │
                    ▼
              TradeProposal
                    │
                    ▼
                Risk Engine
                    │
            ┌───────┴───────┐
            │               │
          REJECT          APPROVE
            │               │
            ▼               ▼
      Explain reason     User Review
                            │
                      ┌─────┴─────┐
                      │           │
                    Cancel     Authorize
                                  │
                                  ▼
                           Broker Workflow
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
                                  ▼
                              Journal
```

---

# Intent Is Not Execution

The user's sentence:

> Buy 5 NVDA.

represents trading intent.

It does not mean the AI should bypass validation.

The process remains:

```text
Intent
 ↓
Analysis
 ↓
Proposal
 ↓
Risk
 ↓
Authorization
 ↓
Execution
```

---

# Final Validation

Immediately before execution, the system should validate again that:

- the proposal has not changed
- the portfolio still permits the trade
- risk limits have not changed
- authorization is still valid

---

# Future Automation

Controlled automation may eventually be investigated.

It should only happen after:

- paper trading
- risk-engine validation
- evaluation
- sufficient historical evidence
- strong operational safeguards

The architecture must not assume autonomous live execution.
