## B02 — get_db_connection
ID: M-002
Layer: serving
Source file: api/main.py:69–79

---

**Module** — get_db_connection
**ID** — M-002
**Layer** — serving
**Primary Responsibility** — Open a psycopg2 connection to the Postgres db container using environment variable credentials. Return the connection object on success. Catch all exceptions and raise HTTPException(500, "Internal server error") — never propagating exception detail to the caller.

**Inputs**
Five environment variables read via `os.environ["KEY"]` (direct dict access — KeyError if missing):
- `POSTGRES_HOST` — hostname of the Postgres container (expected: `db`, the Compose service name)
- `POSTGRES_PORT` — port number as string (expected: `5432`; psycopg2 accepts string)
- `POSTGRES_DB` — database name
- `POSTGRES_USER` — Postgres username
- `POSTGRES_PASSWORD` — Postgres password

No arguments from callers. Environment is read at call time, not at module import time.

**Outputs**
- `psycopg2.connection` object on success — open TCP connection to Postgres, returned to caller. Caller (M-005) is responsible for closing the connection in a `finally` block.
- Raises `HTTPException(status_code=500, detail="Internal server error")` on any exception. Static literal detail — no exception message, no `str(e)`, no traceback fragment.

**Public Interface**
```python
def get_db_connection() -> psycopg2.connection
```
Called directly (not via FastAPI Depends) by M-005 at api/main.py:94.

**Error Behaviour**

| Failure condition | Exception raised by psycopg2/Python | Caught by | Result to caller |
|---|---|---|---|
| DB container not running | `psycopg2.OperationalError` | `except Exception` | HTTPException(500, "Internal server error") |
| Wrong credentials | `psycopg2.OperationalError` | `except Exception` | HTTPException(500, "Internal server error") |
| Wrong host/port | `psycopg2.OperationalError` | `except Exception` | HTTPException(500, "Internal server error") |
| Any env var missing | `KeyError` | `except Exception` | HTTPException(500, "Internal server error") |

`except Exception` intentionally broad — covers psycopg2 errors AND KeyError from missing env vars. Narrowing to `except psycopg2.Error` would expose KeyError as an unhandled exception, producing a FastAPI 500 with framework-generated detail rather than the controlled static literal.

There is no `finally` block in `get_db_connection` itself. This is correct: `psycopg2.connect()` is atomic — either returns a valid connection object or raises without leaving a half-open connection to clean up. Connection lifecycle management (open, use, close) is the caller's responsibility.

**Known Fragility**
- `port=os.environ["POSTGRES_PORT"]` passes port as a string. psycopg2 accepts str or int for the `port` keyword argument. Not a bug, but differs from the idiom `int(os.environ["POSTGRES_PORT"])`. A non-numeric value in `POSTGRES_PORT` raises `psycopg2.ProgrammingError`, which is caught by `except Exception`.
- `except Exception:` without binding (no `as e`) — the exception object is explicitly discarded. This is the correct pattern for INV-09: no path exists to accidentally propagate exception detail.
- No connection validation — returns the connection object as-is. If psycopg2 returns a connection that subsequently fails on `.cursor()`, the failure propagates from M-005's try block. M-005's `finally: conn.close()` still runs correctly in this case.
- No connection pooling. Each call opens a new TCP handshake to Postgres. Under concurrent load this accumulates (R-004). CLAUDE.md fixed stack prohibits adding a pool.

**Change Impact**
- Changing `except Exception` to a narrower type risks unhandled exceptions reaching the HTTP response (INV-09 violation — highest-severity regression in this system).
- Adding an f-string or `str(e)` to the HTTPException detail violates INV-09.
- Any change to the `detail` string violates `execution_plan.md` permitted error literals.
- Adding connection pooling violates CLAUDE.md fixed stack.

**Callers** — M-005 (get_customer route handler) at api/main.py:94
**Calls** — IP-001 (Postgres 15) via `psycopg2.connect()`
**Integration Points Used** — IP-001
