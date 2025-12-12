# 🌀 Advanced Async Context Managers
> async with protokoli · Connection Pooling · Redis · DB Sessions

## 🔥 KIRISH

Async context managers — bu async resurslar (DB connection, Redis connection, lock, file, network stream) bilan ishlashning eng to‘g‘ri va xavfsiz usuli.

Ular quyidagilarni avtomatik boshqaradi:

* Connection ochish / yopish
* Transaction boshlash / commit / rollback
* Lock acquisition / release
* Resource leasing / freeing
* Pool ichidan connection olish / qaytarish

Backend arxitekturasida bu perfomance + correctness uchun juda muhim.

## 🧩 async with PROTOKOLI

Async context manager quyidagilarga ega bo‘lishi shart:

```python
class A:
    async def __aenter__(self):
        ...
    async def __aexit__(self, exc_type, exc, tb):
        ...
```

**Diagram:**

```python
async with manager:
    body
```

1. `await manager.__aenter__()`
2. `body` ichidagi kod bajariladi
3. `await manager.__aexit__(...)`

Agar exception bo‘lsa — `__aexit__` exceptionni boshqaradi.

## 🔧 ASYNC CONTEXT MANAGER INTERNALS

```python
async with X() as conn:
    ...
```

Event Loop quyidagi tartibda ishlaydi:

`call __aenter__()` → `allocate resource` → `return object`
`execute body`
`call __aexit__()` → `release resource` → `commit/rollback`

### __aexit__ argumentlari:

* `exc_type` → Exception class
* `exc` → Exception instance
* `tb` → traceback

Agar `__aexit__` → `True` qaytarsa exception suppression qilinadi.

## 🛠 CUSTOM ASYNC CONTEXT MANAGER YOZISH

### Minimal misol:

```python
class AsyncTimer:
    async def __aenter__(self):
        self.start = time.monotonic()
        return self

    async def __aexit__(self, exc_type, exc, tb):
        self.duration = time.monotonic() - self.start
        print(f"Finished in {self.duration:.3f}s")
```

**Foydalanish:**

```python
async with AsyncTimer():
    await asyncio.sleep(2)
```

### 🔥 More Advanced — Connection leasing simulation

```python
class FakeConnection:
    async def open(self):
        print("Opening connection...")
    async def close(self):
        print("Closing connection...")

class ConnectionCtx:
    def __init__(self, pool):
        self.pool = pool

    async def __aenter__(self):
        self.conn = await self.pool.acquire()
        return self.conn

    async def __aexit__(self, exc_type, exc, tb):
        await self.pool.release(self.conn)
```

Async context manager asosiy vazifasi — pooldan connection olish va qaytarish.

## 🏗 CONNECTION POOLING — ARXITEKTURA

Pooling — resurslarni qayta ishlatish tamoyili.

**Diagram:**

```text
 ┌───────────────┐
 │   Connection   │  ← 10–100 db connections
 └─────────┬──────┘
           │ acquired / released
 ┌─────────▼────────┐
 │    Connection    │
 │       Pool       │
 │  (async safe)    │
 └─────────▲────────┘
           │
   async with get_session()
```

**Nima uchun zarur?**

* DB connection ochish → juda qimmat (TCP handshake, auth, TLS)
* Har request uchun connection ochish → high latency
* Pool performance’ni 10–20 barobar oshiradi

## 🏛 ASYNC DB SESSION (POSTGRESQL) — ADVANCED FLOW

SQLAlchemy / asyncpg / databases library’dagi jarayonlar bir xil tamoyilga ega.

### Session lifecycle (advanced):
`pool.acquire()` → `connection` → `begin transaction`
↓
`session (unit of work)`
↓
`commit / rollback`
↓
`pool.release()`

### Misol (SQLAlchemy + AsyncSession):

```python
from sqlalchemy.ext.asyncio import async_sessionmaker

async_session = async_sessionmaker(engine, expire_on_commit=False)

async def get_db():
    async with async_session() as session:
        try:
            yield session
        except:
            await session.rollback()
            raise
        finally:
            await session.close()
```

Bu yerda `async with` → `aenter()` ichida db connection olinadi.

## ⚡ REDIS ASYNC CONNECTION POOLING

`aioredis` / `redis.async` library’da pool default.

### Redis pool yaratish:

```python
from redis.asyncio import ConnectionPool, Redis

pool = ConnectionPool.from_url("redis://localhost:6379", max_connections=20)
redis = Redis(connection_pool=pool)
```

**Async context manager sifatida:**

```python
async with redis.client() as conn:
    await conn.set("key", "value")
```

**Ichkarida:**

* `pool` → connection qaytaradi
* `aenter` → leasing
* `aexit` → releasing

## 🌐 FASTAPI’DA ASYNC DB SESSION DEPENDENCY

```python
from fastapi import Depends

async def get_session() -> AsyncSession:
    async with async_session() as session:
        yield session

@app.get("/users")
async def get_users(db: AsyncSession = Depends(get_session)):
    return await db.execute(...)
```

**FastAPI dependency injection:**

* bir request = bir session
* session `async with` orqali to‘g‘ri boshqariladi
* exception bo‘lsa → rollback
* responsedan keyin → release

## 🐛 POOLING ADVANCED MUAMMOLARI

### 1️⃣ Connection leak

Session yopilmasa pool to‘ladi → API down bo‘ladi.

**Shuning uchun:**
* har doim `async with` ishlating

### 2️⃣ Pool starvation

Barcha connectionlar band → so‘rovlar kutib qoladi.

**Yechimlar:**
* pool size oshirish
* read replica qo‘shish
* long-running querylarni optimallashtirish

### 3️⃣ Zombie connection

Network breakdown → connection alive tuyuladi lekin ishlamaydi.

**Yechim:**
* pool.recycle
* health-check pingi

### 4️⃣ Transaction never closed

`await session.begin()` → `commit()` unutilgan bo‘lsa:

* deadlock
* locks oshadi
* performance pasayadi

## 🧪 ADVANCED CUSTOM CONTEXT MANAGER — TRANSACTION MANAGER

Bu profi-level pattern:

```python
class Transaction:
    def __init__(self, session):
        self.session = session

    async def __aenter__(self):
        self.tx = await self.session.begin()
        return self.session
    
    async def __aexit__(self, exc_type, exc, tb):
        if exc_type:
            await self.tx.rollback()
        else:
            await self.tx.commit()
```

**Foydalanish:**

```python
async with Transaction(session) as db:
    await db.execute(...)
```

## 📘 XULOSA

Siz endi quyidagi advanced mavzularni to‘liq o‘rgandingiz:

* ✓ `async with` protokoli (internal state machine)
* ✓ Custom async context managerlar
* ✓ Connection pooling arxitekturasi (DB/Redis)
* ✓ AsyncSession lifecycle (acquire → begin → commit → release)
* ✓ FastAPI dependency injection bilan integratsiya
* ✓ Poolingdagi advanced muammolar

Bu bilimlar high-performance backend, distributed systems, microservices, FastAPI, SQLAlchemy, asyncpg, Redis, va network IO kod yozishda majburiy.

---

# 🧩 ADVANCED ASYNC DATABASE & REDIS MECHANICS
> Isolation Levels · Pool Starvation Monitoring · Redis Pipelining · Async File Context Managers · High-Performance DB Middleware

## 1️⃣ ASYNC TRANSACTION ISOLATION LEVELS
> PostgreSQL / SQLAlchemy / asyncpg — Arxitektura & real misollar

Transaction isolation — transactionlar bir-biriga qanchalik ta’sir qilishini belgilaydi.

**PostgreSQL’dagi 4 daraja:**

| Isolation level | Dirty Read | Non-Repeatable Read | Phantom Read |
| :--- | :--- | :--- | :--- |
| **READ UNCOMMITTED** | possible | possible | possible |
| **READ COMMITTED** (default) | ❌ | possible | possible |
| **REPEATABLE READ** | ❌ | ❌ | possible |
| **SERIALIZABLE** | ❌ | ❌ | ❌ (strict) |

### ▶ READ COMMITTED (default)

Har query transaction ichida ishlatilmaydi — har query fresh snapshot oladi.
FastAPI uchun eng xavfsiz default setting.

```python
async with async_session.begin() as s:
    await s.execute(text("SET TRANSACTION ISOLATION LEVEL READ COMMITTED"))
```

### ▶ REPEATABLE READ

Bir transaction → bitta snapshot.
Long-running querylar phantom readlarga olib kelishi mumkin.

```python
await s.execute(text("SET TRANSACTION ISOLATION LEVEL REPEATABLE READ"))
```

### ▶ SERIALIZABLE (most strict)

PostgreSQL konflikt bo‘lsa transactionni abort qiladi.
High-concurrency API’larda abort → retry loop talab qiladi.

```python
async with Transaction(session, isolation="SERIALIZABLE"):
    ...
```

**Custom manager:**

```python
class Transaction:
    def __init__(self, session, isolation="READ COMMITTED"):
        self.session = session
        self.isolation = isolation

    async def __aenter__(self):
        await self.session.execute(
            text(f"SET TRANSACTION ISOLATION LEVEL {self.isolation}")
        )
        return self.session

    async def __aexit__(self, exc_type, exc, tb):
        if exc_type:
            await self.session.rollback()
        else:
            await self.session.commit()
```

**🧪 SERIALIZABLE uchun retry pattern**

```python
for _ in range(5):
    try:
        async with Transaction(session, isolation="SERIALIZABLE") as tx:
            await tx.execute(...)    
        break
    except SerializationFailure:
        await asyncio.sleep(0.01)
```

Production bank tizimlari shuni ishlatadi.

## 2️⃣ POOL STARVATION MONITORING
> Pool band bo‘lib qolishi (dead pool) — advanced diagnostics

Pool starvation → application hanging.

**Sabablari:**
* Session yopilmay qolgan (connection leak)
* Pool sizaga yetmaydi
* Long-running querylar
* Transactionlar hech tugamaydi

**Monitoring pattern:**

### ▶ SQLAlchemy: connection borrow time log

```python
from time import monotonic

class LogPool:
    async def __aenter__(self):
        self.start = monotonic()
        self.conn = await engine.connect()
        return self.conn

    async def __aexit__(self, exc_type, exc, tb):
        duration = monotonic() - self.start
        if duration > 0.5:
            print(f"⚠️ Connection borrowed too long: {duration:.2f}s")
        await self.conn.close()
```

### ▶ Pool overflow detection

```python
if pool.overflow() > 20:
    print("⚠️ Pool overflow detected!")
```

### ▶ Periodik monitoring task

```python
async def pool_metrics():
    while True:
        print("Checked out:", engine.pool.checkedout())
        print("Available:", engine.pool.size() - engine.pool.checkedout())
        await asyncio.sleep(2)
```

FastAPI startup eventiga qo‘shiladi.

## 3️⃣ REDIS PIPELINING + POOLING
> High-performance Redis operations (10x faster)

Pipelining → bir nechta Redis komandalarini bitta network paketda yuborish.

### ▶ aioredis pipelining

```python
async with redis.pipeline(transaction=False) as pipe:
    pipe.set("a", 1)
    pipe.set("b", 2)
    pipe.incr("counter")
    results = await pipe.execute()
```

4 network roundtrip → 1 roundtrip

### ▶ Pipelining + connection pool

**ConnectionPool:**

```python
from redis.asyncio import Redis, ConnectionPool

pool = ConnectionPool.from_url("redis://localhost:6379", max_connections=50)
redis = Redis(connection_pool=pool)
```

**Pipelining that reuses pooled connection:**

```python
async with redis.pipeline() as p:
    for i in range(1000):
        p.incr(f"u:{i}:clicks")
    await p.execute()
```

### ▶ Redis pub/sub with pooled connections

```python
async with redis.pubsub() as ps:
    await ps.subscribe("events")
    async for msg in ps.listen():
        print(msg)
```

## 4️⃣ ASYNC FILE CONTEXT MANAGERS
> aiofiles, streaming, chunked reading — large files uchun

### ▶ Basic async file reading

```python
import aiofiles

async with aiofiles.open("data.txt", "r") as f:
    async for line in f:
        print(line)
```

### ▶ Chunk-based async copy

Production S3 → disk → S3 pipeline uslubi:

```python
async def async_copy(src, dst):
    async with aiofiles.open(src, "rb") as fsrc:
        async with aiofiles.open(dst, "wb") as fdst:
            while True:
                chunk = await fsrc.read(1024 * 1024)
                if not chunk:
                    break
                await fdst.write(chunk)
```

### ▶ Async temporary file manager

```python
class TempFile:
    def __init__(self, path):
        self.path = path

    async def __aenter__(self):
        self.f = await aiofiles.open(self.path, "w+")
        return self.f

    async def __aexit__(self, exc_type, exc, tb):
        await self.f.close()
        os.remove(self.path)
```

## 5️⃣ HIGH-PERFORMANCE DB MIDDLEWARE
> FastAPI + async SQLAlchemy — request-level session management

**Pattern:**
`request` → `open session` → `begin transaction` → `handler` → `commit or rollback` → `release connection` → `response`

### ▶ Full middleware code (production-safe)

```python
from starlette.middleware.base import BaseHTTPMiddleware

class DBSessionMiddleware(BaseHTTPMiddleware):
    async def dispatch(self, request, call_next):
        async with async_session() as session:
            request.state.db = session

            try:
                response = await call_next(request)
                await session.commit()
            except Exception:
                await session.rollback()
                raise
            finally:
                await session.close()

        return response
```

**Use in FastAPI:**

```python
app.add_middleware(DBSessionMiddleware)
```

### ▶ Katta load ostida performance optimallashtirish

* `sessionmaker expire_on_commit=False`
* Pool sizing:
  * min: CPU * 2
  * max: CPU * 5
* Read-only queries uchun read replica pool
* Heavy queries uchun `to_thread` (blocking operations)
* Query cache (Redis, in-memory LRU)

### ▶ Middleware monitoring

**Add timing:**

```python
start = time.monotonic()
response = await call_next(request)
duration = (time.monotonic() - start)

if duration > 0.3:
    print("⚠️ Slow DB request:", request.url)
```

## 📘 XULOSA (super-qisqa)

Siz endi fully advanced darajada o‘rgandingiz:

* ✓ Transaction isolation levels + retry pattern
* ✓ Pool starvation monitoring + metrics
* ✓ Redis pipelining + connection pooling
* ✓ Async file context managers (large file pipeline)
* ✓ High-performance async DB middleware (production pattern)

Bu mavzular senior Python backend, high-load FastAPI, distributed systems, async microservicesda ishlatiladigan professional texnikalar.
