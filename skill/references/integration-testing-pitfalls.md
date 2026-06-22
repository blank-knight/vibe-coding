# Integration Testing Pitfalls

Real bugs caught during integration testing. These only appear with real data —
import-only checks pass even when DB queries, LLM calls, and network requests are broken.

## Don't skip real integration tests between phases

After Phase 3-4, spin up Docker + real DB + real API keys and run an end-to-end smoke test
before writing more phases. Bugs found early are 10x cheaper to fix.

## Common bugs that only appear with real data

- LLM 401: URL was `/messages` instead of `/v1/messages` for relay services
- DB naming mismatch: `async_session_maker` vs `AsyncSessionLocal` across files
- API returning empty data (302 redirect / anti-scrape)
- .env API key written incorrectly due to tool masking

## East Money API → use AKShare instead

`push2.eastmoney.com` and `push2hist.eastmoney.com` frequently return 302 or empty
`klines: []` for index data. Use AKShare instead:

```python
import akshare as ak
df = ak.stock_zh_index_daily(symbol='sh000300')  # 沪深300
df = ak.stock_zh_index_daily(symbol='sz399006')  # 创业板
df = ak.stock_zh_index_daily(symbol='sh000016')  # 上证50
```

AKShare is stable, free, and purpose-built for A-share data.

## Docker postgres port conflict

If port 5432 is already in use (system PostgreSQL running), Docker compose will fail.
Check first: `ss -tlnp | grep 5432`
Fix: map to different host port in docker-compose.yml:
```yaml
ports:
  - "5433:5432"
```
Then update DATABASE_URL and alembic.ini to use port 5433.

## LLM API URL format

Anthropic format via relay services:
- Base URL: `https://your-relay.com`
- Messages endpoint: `https://your-relay.com/v1/messages` (note: `/v1/` prefix required)
- Header: `x-api-key: <key>` + `anthropic-version: 2023-06-01`

OpenAI-compatible format (DeepSeek, GLM, Gemini):
- Endpoint: `<base_url>/v1/chat/completions`
- Header: `Authorization: Bearer <key>`

## Alembic setup for async SQLAlchemy projects

Alembic uses sync drivers for migration (even if app uses asyncpg).
Install: `pip install alembic psycopg2-binary`

alembic.ini: use sync URL
```
sqlalchemy.url = postgresql+psycopg2://user:pass@localhost:5433/dbname
```

alembic/env.py: import all models so autogenerate detects them:
```python
import sys, os
sys.path.insert(0, os.path.dirname(os.path.dirname(__file__)))
os.environ.setdefault('JWT_SECRET', 'migration-placeholder')
os.environ.setdefault('LLM_API_KEY', 'migration-placeholder')
from app.models.models import Base
target_metadata = Base.metadata
```
