# ⚡ Generatorlar va Iteratorlar

## 📋 Mundarija

### Asosiy Generatorlar va Iteratorlar
- [1. Iterator Protokoli (Advanced)](#1-iterator-protokoli-advanced)
- [2. Custom Iterator — Advanced (Stateful Iterator)](#2-custom-iterator--advanced-stateful-iterator)
- [3. Generator Pipelines (Functional Data Pipeline)](#3-generator-pipelines-functional-data-pipeline)
- [4. yield from — Advanced Delegation (PEP 380)](#4-yield-from--advanced-delegation-pep-380)
- [5. Infinite Streams — real backend patternlari](#5-infinite-streams--real-backend-patternlari)
- [Senior-Level Patterns](#-senior-level-patterns)

### Async Generatorlar
- [6. Async Generatorlar nima?](#6-async-generatorlar-nima)
- [7. Awaitable + AsyncIterator protokoli (PEP 525)](#7-awaitable--asynciterator-protokoli-pep-525)
- [8. async for qanday ishlaydi?](#8-async-for-qanday-ishlaydi)
- [9. async yield — Async streamingning yuragi](#9-async-yield--async-streamingning-yuragi)
- [10. Real World: Async Pipeline](#10-real-world-async-pipeline)
- [11. Async generatorlarning advanced metodlari](#11-async-generatorlarning-advanced-metodlari)
- [12. Real Backend Misollar](#12-real-backend-misollar)
- [13. Backpressure va Flow Control](#13-backpressure-va-flow-control)
- [Yakuniy Advanced Xulosa](#-yakuniy-advanced-xulosa)

---

## 1. Iterator Protokoli (Advanced)

### Iterator bo'lishi uchun:

1. `__iter__(self)` → `self`'ni qaytaradi (iteratorning o'zi)
2. `__next__(self)` → navbatdagi elementni qaytaradi
3. Element tugaganida → `StopIteration` ko'taradi

### CPython ichida bu qanday ishlaydi?

```python
for x in iterable:
    ...
```

CPython bajaradi:

```python
ITER = iter(iterable)
while True:
    try:
        value = next(ITER)
    except StopIteration:
        break
```

### `iter()` chaqirganda:

- Agar obyekt `__iter__()` ga ega bo'lsa → iterator qaytaradi
- Agar `__getitem__` bo'lsa (0 dan boshlab) → eski iterator modeli (sequence protocol) ishlaydi

---

## 2. Custom Iterator — Advanced (Stateful Iterator)

Oddiy emas, balki **stateful, memory-friendly** iterator yaratamiz.

### Misol: file'ni bo'laklab o'qiydigan iterator

```python
class ChunkReader:
    def __init__(self, file, size=1024):
        self.file = open(file, "rb")
        self.size = size

    def __iter__(self):
        return self

    def __next__(self):
        chunk = self.file.read(self.size)
        if not chunk:
            self.file.close()
            raise StopIteration
        return chunk
```

**Bu:**

- Katta fayllarni to'liq RAM-ga olmadi
- Backpressure mexanizmi sifatida ishlaydi
- Streaming pipeline uchun asos

---

## 3. Generator Pipelines (Functional Data Pipeline)

Generatorlar — **lazy evaluation** ishlatadi.  
Data pipeline (ETL, log processing, streaming) uchun juda mos.

### UNIX pipeline falsafasi:

```bash
cat file | grep error | awk '{print $1}'
```

### Python versiyasi:

```python
def read_lines(path):
    with open(path) as f:
        for line in f:
            yield line

def filter_errors(lines):
    for line in lines:
        if "ERROR" in line:
            yield line

def extract_timestamp(lines):
    for line in lines:
        yield line.split()[0]
```

### Pipeline:

```python
lines = read_lines("app.log")
errors = filter_errors(lines)
timestamps = extract_timestamp(errors)

for ts in timestamps:
    print(ts)
```

### Foyda:

- ✅ Har bir bosqich lazy
- ✅ RAM deyarli ishlatilmaydi
- ✅ Parallel ishlay oladi
- ✅ Backpressure yordamida haddan oshgan yuklanishdan saqlaydi

---

## 4. yield from — Advanced Delegation (PEP 380)

`yield from` — generator ichida boshqa generatorni ishlatish uchun.

### Avval oddiy variant:

```python
def gen():
    for x in subgen():
        yield x
```

### Advanced variant:

```python
def gen():
    yield from subgen()
```

### Nima uchun professional daraja?

- ✅ Delegatsiya → exception propagation to'g'ri ishlaydi
- ✅ Return value → subgenerator natijasini qaytaradi
- ✅ Stack depth kamroq

### Misol:

```python
def sub():
    yield 1
    yield 2
    return "done"

def main():
    result = yield from sub()
    print("Subgen returned:", result)
```

**Chaqirish:**

```python
g = main()
print(next(g))   # 1
print(next(g))   # 2
print(next(g))   # Subgen returned: done
```

> 👉 **Bu coroutine-architecture'da juda muhim!**

---

### `yield from` qanday ishlaydi? (CPython darajasida)

`yield from` generator delegatsiyasi uchun quyidagilarni qo'llaydi:

- `__next__`
- `send`
- `throw`
- `close`

**Yani:**

- Bolaning exceptionlari ota generatorga yuboriladi
- Bolaning return value → ota generatorning `yield from` natijasiga aylanadi

---

## 5. Infinite Streams — real backend patternlari

**Infinite stream** — tugamaydigan iterator.  
Bunday generatorlar real-time eventlar, loglar, sensorlar uchun ishlatiladi.

### Oddiy infinite stream:

```python
def counter(start=0):
    while True:
        yield start
        start += 1
```

### Real-time variant (log tailing):

```python
import time

def follow(file):
    file.seek(0, 2)  # end of file
    while True:
        line = file.readline()
        if line:
            yield line
        else:
            time.sleep(0.1)
```

> Bu Linux `tail -f` funksiyasiga teng.

---

### ⚡ Advanced: Infinite Stream + Pipeline

Real-time eventlarni filtrlaymiz:

```python
log = follow(open("app.log"))

errors = (line for line in log if "ERROR" in line)

for e in errors:
    print("ERR:", e)
```

**Lazy pipeline → real-time monitoring.**

---

### ⚙ Backpressure — professional muammo

Infinite streamda muammo:

- Producer juda tez
- Consumer sekin

> Bu queue + coroutine orqali hal qilinadi (lekin Async bo'limida o'tamiz).

---

## 📌 Senior-Level Patterns

| Pattern | Foydasi |
|---------|---------|
| **Custom iterator** | Specialized memory-efficient traversal |
| **Generator pipeline** | ETL, streaming, lazy transform |
| **yield from** | Coroutine delegation, deep generator composition |
| **Infinite stream** | Real-time systems, continuous data feeds |

---

## 🎓 Yakuniy advanced tushunchalar

- ✅ **Generatorlar** → stackless coroutines sifatida ishlaydi
- ✅ **yield from** → to'liq delegatsiya (return values + exceptions)
- ✅ **Iterator protokoli** → CPythonning eng asosiy ko'prigi
- ✅ **Pipeline** → parallel, lazy, minimal memory
- ✅ **Infinite stream** → real backend uchun poydevor (monitoring, logging)

---

# 🔄 Async Generatorlar

ADVANCED darajadagi **Async Generatorlar** — bu Python'da high-performance streaming, async networking, websocketlar, async pipelines, event-driven arxitekturalar uchun fundamental mavzu.

---

## 6. Async Generatorlar nima?

### Oddiy generator:

```python
def gen():
    yield 1
```

### Async generator:

```python
async def agen():
    yield 1
```

> **ASYNC generator** ham coroutine, ham iterator.  
> Buning uchun unda ikkita protokol birlashgan.

---

## 7. Awaitable + AsyncIterator protokoli (PEP 525)

Async generator quyidagi protokollarni implement qiladi:

### 📌 AsyncIterator protokoli:

- `__aiter__(self)`
- `__anext__(self)` → awaitable qaytarishi shart

> `async for` ishlashi uchun `__anext__` natijasi **awaitable** bo'lishi kerak.

---

## 8. async for qanday ishlaydi?

### Kod:

```python
async for x in agen():
    ...
```

### CPython bu kodni quyidagiga aylantiradi:

```python
iterator = agen().__aiter__()

while True:
    try:
        value = await iterator.__anext__()
    except StopAsyncIteration:
        break
    else:
        ... body ...
```

### Muhim farq:

- **Oddiy for** → `__next__()` chaqiradi
- **async for** → `await __anext__()` chaqiradi
- Tugaganida → `StopAsyncIteration` ko'tariladi

---

## 9. async yield — Async streamingning yuragi

### Oddiy generator yield:

```python
yield value
```

### Async generator yield:

```python
yield value
await something
yield value
```

### Async generatorning kuchi shunda:

- U **non-blocking** ishlab turadi
- Event loop uni pause/resume qiladi

### Oddiy misol:

```python
async def ticker():
    n = 0
    while True:
        yield n
        n += 1
        await asyncio.sleep(1)
```

> Bu — real **infinite async stream**.

---

## 10. Real World: Async Pipeline

Bu async barcha bosqichlar **parallel** ishlaydi.

```python
async def reader():
    for i in range(5):
        await asyncio.sleep(0.1)
        yield i

async def mapper(aiter):
    async for x in aiter:
        yield x * 2

async def printer(aiter):
    async for x in aiter:
        print(x)

async def main():
    await printer(mapper(reader()))
```

**Natija: (lazy, streaming, async pipeline)**

```
0
2
4
6
8
```

> Bu **FASTAPI, aiohttp, websockets, Redis streams** bilan juda ko'p ishlatiladi.

---

## 11. Async generatorlarning advanced metodlari

Async generatorlarda **3 ta advanced API** bor:

| Metod | Tavsif |
|-------|--------|
| `agen.asend(value)` | Generator ichiga qiymat yuboradi (yield expression tarafidan qabul qilinadi) |
| `agen.athrow(exc)` | Generator ichiga exception tashlaydi |
| `agen.aclose()` | Generatorni toza yopadi (async finalizer chaqiradi) |

### Misol:

```python
async def agen():
    try:
        yield 1
        yield 2
    finally:
        print("cleaning...")
```

**Chaqiramiz:**

```python
g = agen()
await g.__anext__()
await g.aclose()
```

**Natija:**

```
cleaning...
```

> Bu real backendlarda **connection pooling, cleanup, socket close** uchun juda muhim.

---

## 12. Real Backend Misollar

### 📌 12.1. WebSocket stream

```python
async def ws_stream(ws):
    async for msg in ws:
        yield msg.data
```

**Bu generator:**

- Real-time kelayotgan xabarlarni stream qiladi
- Consumer esa `async for` bilan iste'mol qiladi

---

### 📌 12.2. Async DB cursor (PostgreSQL, asyncpg)

```python
async def cursor_stream(conn, query):
    async with conn.transaction():
        async for row in conn.cursor(query):
            yield row
```

> RAMga to'plamaydi → **streaming!**

---

### 📌 12.3. Log streaming (tail -f)

```python
async def follow(path):
    f = open(path)
    f.seek(0, 2)

    while True:
        line = f.readline()
        if line:
            yield line
        await asyncio.sleep(0.1)
```

**Backendlarda:**

- Monitoring
- Audit logging
- Real-time analytics

uchun ishlatiladi.

---

## 13. Backpressure va Flow Control

### Eng muhim professional masala:

- **Producer** (async generator) → juda tez
- **Consumer** (async for loop) → sekin

### Agar boshqarilmasa:

- ❌ RAM to'ladi
- ❌ Timeoutlar
- ❌ Event-loop qotadi

### Yechimlar:

1. **Buffer** (asyncio.Queue bilan)
2. **Flow control** (producer await bilan pauza oladi)
3. **Windowing** (limited items per second)

### Misol:

```python
async def producer(q):
    for i in range(1000000):
        await q.put(i)  # bu backpressure yaratadi

async def consumer(q):
    while True:
        item = await q.get()
        ...
```

> **Queue** — backpressure mexanizmi.

Async generatorlar bilan natural backpressure avtomatik mavjud, chunki:

- Consumer `await` orqali producerni to'xtatib turadi

---

## 🎓 Yakuniy Advanced Xulosa

| Funksiya | Oddiy Generator | Async Generator |
|----------|-----------------|-----------------|
| **yield** | bloklaydi | non-blocking yield |
| **__next__** | sync | `__anext__` → awaitable |
| **StopIteration** | sync | `StopAsyncIteration` |
| **Pipeline** | lazy | lazy + concurrent |
| **Streaming** | limited | real-time infinite streams |
| **Backpressure** | manual | event-loop orqali avtomatik |

### Async generator:

- ✅ Coroutine + iterator gibrid
- ✅ Real-time streamlar uchun asos
- ✅ Advanced pipelines yaratadi
- ✅ Websockets, async DB cursorlar, log streaming uchun ideal