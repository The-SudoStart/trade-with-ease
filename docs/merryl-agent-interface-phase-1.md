# Phase 1: Merryl Agent Interface proposal

Status: design proposal, 7 October 2026. No implementation is included here.

## Decision

Add a read-only MCP server **inside the Merryl Rust crate**, launched as `merryl mcp` over stdio. It should read Merryl's already-scored SQLite state through a small query service. The daily workflow remains the only producer of scores. Neither a tool call nor MCP startup should fetch market data, run scoring, migrate the database, or refresh a screener.

This keeps one source of truth, avoids a second service and database, and lets an agent use the existing Rust domain and dashboard mapping code. The first consumer can be local. A remote HTTP transport can be considered after authentication, hosting, and data access requirements exist. The [official Rust MCP SDK](https://github.com/modelcontextprotocol/rust-sdk) (`rmcp`) supplies protocol handling and stdio transport; pin a tested release when implementation starts.

The MCP service exposes market intelligence only. It has no tools for orders, accounts, positions, trade proposals, or brokerage access.

## Repository evidence

Inspected Merryl at [`f3822b6`](https://github.com/AssahBismarkabah/Merryl/commit/f3822b667adad82831f71d51172b47be708d0e3a). These are observations about that revision, rather than assumptions from Trade With Ease's roadmap:

- [`src/main.rs`](https://github.com/AssahBismarkabah/Merryl/blob/f3822b667adad82831f71d51172b47be708d0e3a/src/main.rs) has a Clap CLI and Tokio runtime for `dashboard`; there is no MCP command.
- [`src/dashboard/server.rs`](https://github.com/AssahBismarkabah/Merryl/blob/f3822b667adad82831f71d51172b47be708d0e3a/src/dashboard/server.rs) serves `/api/dates`, `/api/dashboard/latest`, and `/api/dashboard/{date}`. Its screener route can perform a live refresh, so the MCP server must not delegate to that route.
- [`src/dashboard/models.rs`](https://github.com/AssahBismarkabah/Merryl/blob/f3822b667adad82831f71d51172b47be708d0e3a/src/dashboard/models.rs) already defines serialized regime, sector, industry, stock, watchlist, and backtest DTOs.
- [`src/dashboard/repository.rs`](https://github.com/AssahBismarkabah/Merryl/blob/f3822b667adad82831f71d51172b47be708d0e3a/src/dashboard/repository.rs) builds a snapshot from SQLite and adds macro context and actionability labels. It limits industry and stock rows to dashboard report limits; its watchlist enrichment looks up names and labels in that limited stock slice. The MCP list and symbol tools need targeted queries to avoid silently losing rows or details.
- [`src/storage/read_repository.rs`](https://github.com/AssahBismarkabah/Merryl/blob/f3822b667adad82831f71d51172b47be708d0e3a/src/storage/read_repository.rs) has date-scoped regime, sector, industry, stock, and watchlist reads; date-range score reads; and `latest_backtest_result`. It has no public event-by-symbol or backtest-by-id read.
- [`src/storage/schema.rs`](https://github.com/AssahBismarkabah/Merryl/blob/f3822b667adad82831f71d51172b47be708d0e3a/src/storage/schema.rs) stores `events`, `stock_scores`, `watchlists`, and `backtest_results`, so the missing tools need queries, not new scoring or provider integrations.
- [`src/backtest/mod.rs`](https://github.com/AssahBismarkabah/Merryl/blob/f3822b667adad82831f71d51172b47be708d0e3a/src/backtest/mod.rs) describes backtests as score and forward-return validation. These are not simulated trade P&L or evidence of profitable execution.

## Rust shape

```text
src/main.rs                 add `mcp` subcommand; start Tokio runtime
src/agent/mod.rs            register tools and stdio transport
src/agent/contracts.rs      serde request/response types and JSON Schemas
src/agent/queries.rs        date resolution, validation, bounds, DTO mapping
src/storage/read_repository.rs
                            add focused SELECTs for symbol, events, history,
                            full date-scoped rankings, and backtest lookup
src/dashboard/repository.rs reuse pure DTO conversion where useful;
                            do not call HTTP handlers or load full snapshots
tests/agent_interface.rs    fixture-backed query and MCP contract tests
```

`AgentQueries` can own a database path and expose typed read methods. Each call opens a short-lived **read-only** SQLite connection and uses parameterized SQL. Existing `Database::open` creates a directory and opens a writable connection, while dashboard loaders call `migrate()`; those paths should not be reused for MCP reads. Check that the DB exists before opening, open with read-only flags, set a bounded busy timeout, and return `DATA_UNAVAILABLE` if the daily workflow has not populated it. Never expose raw SQL or a filesystem path as a tool argument.

The MCP handlers validate input, call `AgentQueries`, then return a small typed object. SQLite work is blocking, so execute it with `tokio::task::spawn_blocking` and cap result sizes. Keep the tool names and output shapes stable even if Merryl changes its internal tables. Reuse existing DTO mapping functions only after extracting them into pure shared helpers; avoid making the agent interface depend on the dashboard's list limits or its full snapshot construction.

On stdio, stdout is reserved for MCP protocol messages; logs and diagnostics go to stderr. Tool discovery order should be deterministic. Mark every tool read-only. Use object-shaped `structuredContent` matching an `outputSchema`, and include JSON text content for clients that only read text. These choices follow the [MCP tool contract](https://modelcontextprotocol.io/specification/2025-11-25/server/tools) and [stdio transport rules](https://modelcontextprotocol.io/specification/2025-11-25/basic/transports).

## Shared contract rules

All dates are ISO `YYYY-MM-DD` score dates, not wall-clock timestamps. An omitted `date` means the latest **stored scored date**; each response reports the resolved `score_date`. An explicit date must exist in Merryl's scored dates. `symbol` is canonicalized to uppercase and checked against the stored symbol universe; `sector` must match a stored sector. `limit` defaults to 20 and is capped at 100. History windows are capped at 252 scored dates. Reject unknown input fields and invalid ranges.

Generate each MCP `inputSchema` from its Rust request type, with `additionalProperties: false`. For example, the required sector filter has this shape (the other tool fields and required inputs are specified in the table below):

```json
{
  "type": "object",
  "additionalProperties": false,
  "properties": {
    "sector": {"type": "string", "minLength": 1},
    "date": {"type": "string", "format": "date"},
    "limit": {"type": "integer", "minimum": 1, "maximum": 100},
    "require_fresh": {"type": "boolean"}
  },
  "required": ["sector"]
}
```

Every successful tool returns an object with `schema_version: "1"`, `score_date` when applicable, `data_status` (`complete` or `partial`), and `data` (the tool-specific payload). Include `warnings: string[]` for missing optional context, and `source: "merryl_sqlite"`. Null means a documented optional value is unavailable; zero remains a real measured zero. Responses must not substitute zero for malformed or missing persisted JSON. A missing requested symbol/date is `NOT_FOUND`, not an empty success; empty legitimate lists remain empty arrays. This envelope is a proposed public contract, not an existing Merryl DTO.

Use bounded, stable tool errors: `INVALID_ARGUMENT`, `NOT_FOUND`, `DATA_UNAVAILABLE`, `DATA_STALE`, and `INTERNAL`. Return `isError: true` with a concise code/message and optional retry guidance. Do not leak SQL, absolute paths, credentials, or raw upstream text in errors. Data freshness is visible, not implied: include `latest_score_date` in successful responses and warn when the chosen snapshot is older than the configured freshness threshold. For a current-market request, let the caller specify `require_fresh: true`; then stale data returns `DATA_STALE`. Historical queries do not apply a current-freshness check.

For the first release, set the freshness threshold to four calendar days from the stored score date, configurable as `MERRYL_MCP_MAX_SCORE_AGE_DAYS`. This is an explicit operational approximation until Merryl has a market-session calendar. Return the score date so clients can make a stricter choice. `get_catalysts` uses either `date` or an explicit `from_date`/`to_date` pair, never both; `get_signal_history` accepts a date pair or neither, and its `limit` cap is 252 rather than 100.

### Tool contracts

The type names below are proposed Rust/JSON DTOs. Required fields are listed first; `?` means optional input. Lists retain Merryl's stored rank order. Percent-like return fields are decimal fractions (`0.05` means 5%); scores and ranks retain Merryl's stored units. Each response is wrapped in the shared envelope.

| Tool | Input | `data` payload | Backing path / work |
| --- | --- | --- | --- |
| `get_market_regime` | `{date?, require_fresh?}` | `regime: {label, score, explanation, spy_return_20d, spy_return_60d, qqq_relative_return_vs_spy, iwm_relative_return_vs_spy, dia_relative_return_vs_spy, components, macro_context?}` | `market_regime_for_date`; reuse dashboard macro overlay, without full dashboard load. |
| `get_sector_rankings` | `{date?, limit?, require_fresh?}` | `sectors: [{sector, sector_etf, rank, score, return_1d, return_5d, return_20d, return_60d, relative_return_vs_spy, relative_volume, breadth_20d, breadth_50d, rank_change, explanation}]` | Existing `sector_scores_for_date`; apply bounded limit after rank ordering. |
| `get_industry_rankings` | `{sector, date?, limit?, require_fresh?}` | `industries: [{industry, sector, rank, score, return_5d, return_20d, return_60d, relative_return_vs_sector, relative_return_vs_spy, relative_volume, breadth_20d, breadth_50d, high_20d_rate, member_count, components}]` | Add sector-filtered date query; current method only has a global limit. Preserve the stored global rank and document it. |
| `get_watchlist` | `{date?, limit?, require_fresh?}` | `watchlist: [{rank, symbol, name, sector, industry, score, reason, catalyst_status, classifications, primary_actionability, actionability_labels}]` | `watchlist_for_date` plus a targeted join/lookup for **all** listed symbols; reuse classification and actionability functions. Never enrich from a truncated top-stock slice. |
| `get_stock_analysis` | `{symbol, date?, require_fresh?}` | `stock: {symbol, name, rank, sector, industry, score, sector_score, return_1d, return_5d, return_20d, return_60d, relative_return_vs_sector, relative_return_vs_spy, relative_volume, avg_dollar_volume, trend_state, catalyst_status, primary_actionability, actionability_labels, components, explanation}` | Add stock-score-by-symbol/date read; reuse pure stock DTO mapping. A ranked stock need not be on the watchlist. |
| `get_catalysts` | `{symbol, date?, from_date?, to_date?, limit?}` | `events: [{event_date, event_type, headline, source, url?, quality_status, fetched_at?}], catalyst_status?` | Add bounded `events` SELECT by symbol/date range; `catalyst_status` comes from the requested stock score, if present. Exclude `raw_json` and source event IDs. Default window: 30 calendar days before through 30 days after the resolved score date; reject ranges over 366 days. Events are context, not score inputs. |
| `get_signal_history` | `{symbol, from_date?, to_date?, limit?}` | `observations: [{score_date, rank, score, sector, industry, relative_return_vs_spy, relative_volume, trend_state, catalyst_status, on_watchlist}]` | Add a bounded chronological `stock_scores` query with `watchlists` left join. Default to latest 20 scored observations. No forward-return or trade-performance claim. |
| `get_backtest_results` | `{result_id?}` | `backtest: {id, run_name, from_date, to_date, created_at, validation_scope, summaries, observation_counts}` | Existing latest result for omitted id; add id lookup. Parse stored `metrics_json` into the existing `BacktestMetrics` shape. Never run a backtest from a tool call. The request does **not** accept arbitrary `from`/`to` until persisted-run selection semantics are designed. |
| `explain_score` | `{symbol, date?}` | `explanation: {symbol, score_date, score, rank, narrative, components, sector_context?, industry_context?, limitations}` | Reuse the persisted stock explanation and components, plus date-matched sector/industry rows. Do not ask an LLM to fabricate an explanation. Mark unavailable component fields explicitly. |

The current dashboard's `latest_backtest` field is global, not tied to a selected dashboard score date. The MCP backtest result therefore reports its own `from_date`, `to_date`, and `created_at` and never implies it belongs to the requested market snapshot.

### Example wire payload

`get_stock_analysis({"symbol":"NVDA","date":"2026-08-25"})` could return the following **illustrative shape**. The numbers are examples, not Merryl observations:

```json
{
  "schema_version": "1",
  "score_date": "2026-08-25",
  "latest_score_date": "2026-08-25",
  "source": "merryl_sqlite",
  "data_status": "complete",
  "warnings": [],
  "data": {
    "stock": {
      "symbol": "NVDA",
      "name": "Example name",
      "rank": 3,
      "sector": "Technology",
      "industry": "Semiconductors",
      "score": 81.2,
      "sector_score": 74.1,
      "return_1d": 0.01,
      "return_5d": 0.04,
      "return_20d": 0.12,
      "return_60d": 0.20,
      "relative_return_vs_sector": 0.03,
      "relative_return_vs_spy": 0.06,
      "relative_volume": 1.4,
      "avg_dollar_volume": 1000000000,
      "trend_state": "example",
      "catalyst_status": "example",
      "primary_actionability": "example",
      "actionability_labels": [],
      "components": {},
      "explanation": "Example stored explanation"
    }
  }
}
```

## Delivery order and acceptance checks

1. Extract stable response types and pure dashboard mappings where they are already correct. Add the read-only SQLite opener and date/symbol validation. Verify a missing DB does not create a file or directory.
2. Ship `get_market_regime`, `get_sector_rankings`, `get_industry_rankings`, and `get_watchlist` against a seeded database. Verify selected dates, rank order, list bounds, and complete watchlist enrichment.
3. Add targeted reads for stock detail, events, history, and persisted backtests; then ship the remaining five tools. Verify symbol/date/range errors, source metadata, and malformed stored JSON behavior.
4. Wire `merryl mcp` using `rmcp` stdio. Run an MCP client through `initialize`, `tools/list`, and every `tools/call`; validate responses against declared output schemas. Assert stdout contains only protocol traffic.
5. Run `cargo fmt --check`, `cargo clippy`, and `cargo test` in Merryl. The existing Trade With Ease repository remains documentation-only for this proposal.

The Phase 1 exit test is an external MCP client answering the four roadmap questions about regime, leading sectors, strongest opportunities, and why a symbol ranked highly, using only stored Merryl data and without direct database access.
