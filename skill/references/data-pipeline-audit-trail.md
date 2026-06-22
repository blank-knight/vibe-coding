# Data Pipeline Audit Trail Pattern

When building any data pipeline (crawlers, LLM analysis, trading signals),
you want **full-chain auditability** — every step saved so you can verify
what happened after the fact.

## DB Layer Requirements

For every table that involves external data or LLM processing, add:

```python
# On any table that stores LLM output:
llm_raw_response: Mapped[Optional[dict]] = mapped_column(JSON, nullable=True)
# Stores the complete LLM API response, not just the parsed result

# On prediction/signal tables:
is_prediction: Mapped[bool] = mapped_column(Boolean, default=False)
# Marks whether LLM judged this record as a valid signal (vs noise)
```

## CrawlLog Table (mandatory for any crawler project)

Add a `crawl_logs` table to every project with crawlers:

```python
class CrawlLog(Base):
    __tablename__ = "crawl_logs"
    id, crawler_name, run_at, status  # identity
    items_fetched, items_saved, items_skipped  # counts
    duration_seconds, error_message  # diagnostics
    raw_snapshot: JSON  # full raw data snapshot for that run
```

Every crawler run writes one row. Status: `success / partial / failed`.

## Daily JSON Export (file layer)

Complement the DB with a daily export service that dumps everything to:

```
项目根目录/data/YYYY-MM-DD/
  predictions.json      # posts + LLM parse + T+1 verification
  verifications.json    # T+1 comparison detail
  news.json             # articles + sentiment
  market_data.json      # OHLC for tracked indices
  crawl_logs.json       # crawler run history
  daily_signal.json     # final composite signal
```

Key implementation notes:
- Use `json.dump(..., default=_json_serial)` with a datetime serializer
- Use `selectinload()` for related models to avoid N+1 queries
- Export runs after each full crawl+analysis cycle (scheduler triggers it)
- Files are human-readable — open them directly to verify

## Orchestrator Pattern

Wrap each crawler in a try/except that always writes a CrawlLog:

```python
async def run_X_crawl(db):
    t0 = time.time()
    try:
        # ... do work, track fetched/saved/skipped counts
        status = "success"
    except Exception as e:
        error_msg = str(e)
        status = "failed"
    finally:
        await _write_crawl_log(db, "X", status, fetched, saved, skipped,
                               time.time()-t0, error_msg, snapshot)
```

## Database Schema Documentation (self-service verification)

Create `docs/database-schema.md` with:
1. **Quick query template** — copy-pasteable snippet to run ad-hoc SQL
2. **Every table**: column names, types, what they store, concrete examples
3. **Every table**: 2-3 useful query examples (today's data, recent failures, aggregations)
4. **Data flow diagram** — showing table→table relationships
5. **Current data counts** — live snapshot so you can spot "zero rows = broken"

Update this doc whenever the schema changes.

## Third-Party Pre-Analyzed Data: Mark Clearly

Some data sources return already-analyzed scores (not raw data for our LLM to process).
These MUST be documented differently.

In docs and code comments, explicitly state:
- What the source analyzed
- What we did with it (just averaged / mapped / passed through)
- What our LLM actually processed vs what we took as-is

## Data Source Replacement vs Repair Decision

When a data source becomes permanently unavailable:
- **Replace entirely** if: the source requires expensive bypass (WAF + headless browser) AND alternative sources exist with similar coverage
- **Repair** if: the issue is a simple API change, rate limit, or temporary block

Decision heuristic: if fixing costs > 2x the effort of finding an alternative, replace.

## Frontend Data Traceability

When displaying pipeline output on a frontend, include a data traceability panel:
- News titles link to original article URLs
- Each item shows: source, timestamp, sentiment label, raw score
- Panel is collapsible (default expanded for first-time users)

## Why This Matters

When building a system that makes recommendations based on data:
1. Verify that scoring is fair (see raw LLM reasoning)
2. Audit why a signal was generated on a given day
3. Catch crawler bugs (e.g. 0 articles fetched = API changed)
4. Build trust in the system before acting on its signals
5. Self-verify data without depending on the developer (schema docs + traceability)

Never skip the audit trail for "speed" — it's a first-class requirement.
