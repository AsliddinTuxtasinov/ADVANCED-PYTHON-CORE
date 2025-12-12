# 🌀 Advanced Asyncio Internals
> gather · wait · shield · semaphore · lock · cancellation · TaskGroup (Python 3.11+)

## 🔥 KIRISH

Asyncio — faqat “async/await” emas.
U murakkab cooperative concurrency modeliga ega:

* Task state machine
* Cancellation propagation
* Gather/wait scheduling
* Semaphore/Lock primitivlari
* Structured concurrency (TaskGroup)

Quyida — har biri internal mechanism bilan.

---

## 1️⃣ asyncio.gather INTERNALS
> Parallel coroutine execution + result aggregation

`gather` — barcha tasklarni parallel ishga tushiradi va natijalarni tartib bilan qaytaradi.

**API:**
```python
results = await asyncio.gather(coro1(), coro2(), coro3())
```

### 🔬 Gather ichida nima bo‘ladi?

1. Har coroutine → **Task** yaratiladi
2. Har Task event loopga **schedule** qilinadi
3. Gather — **master future** yaratadi
4. Har Task tugaganda → gather master future callback ishga tushadi
5. Barcha Task tugaganda → gather master future “done” bo‘ladi
6. Natijalar `coro1` → `coro2` → `coro3` tartibida qaytariladi

### ❗ Default: exceptions propagate immediately

Agar bitta task xato bersa — gather butun operatsiyani to‘xtatadi.

```python
await asyncio.gather(task1(), task2())  
# task2 xato qilsa task1 cancel bo‘ladi
```

**Yechim — xatolarni o‘chirmaslik:**
```python
results = await asyncio.gather(a(), b(), return_exceptions=True)
```

## 2️⃣ asyncio.wait
> Low-level task coordination — “done/pending” modeli

* **Gather** → hasil to‘plovchi
* **Wait** → task holatini nazorat qiluvchi primitiv

**Basic usage:**
```python
done, pending = await asyncio.wait(tasks)
```

`wait` hech qachon exceptionni raise qilmaydi — `task.result()` da aniqlaysiz.

### wait modes:

**✔ FULL completion**
```python
await asyncio.wait(tasks, return_when=asyncio.ALL_COMPLETED)
```

**✔ FIRST completed**
```python
await asyncio.wait(tasks, return_when=asyncio.FIRST_COMPLETED)
```

**✔ FIRST exception**
```python
await asyncio.wait(tasks, return_when=asyncio.FIRST_EXCEPTION)
```

### Internals:

1. Event loop har taskga “done callback” qo‘shadi
2. Callbacklar ishga tushganda wait future “wake up” qilinadi
3. Pending tasklar qaytariladi

Bu gatheringdan ko‘ra past darajadagi boshqaruvni beradi.

## 3️⃣ asyncio.shield
> Cancellation propagationni to‘liq tushunish uchun muhim

`shield(task)` → task cancel bo‘lishdan himoyalanadi.

**Shunchaki misol:**
```python
await asyncio.shield(my_task)
```

Agar parent task cancel qilinsa ham:
* `my_task` cancel bo‘lmaydi
* faqat `shield()` o‘zi cancel qilinadi

**Diagram:**
```text
Parent Task cancel →
   cancel shield() →
      my_task continues running
```

**Real production use-case:**
Fayl S3 ga upload qilinayotgan bo‘lsa:

```python
async def upload():
    await asyncio.sleep(10)

await asyncio.shield(upload())
```

Agar request timeout bo‘lsa:
* background upload tugaguncha ishlayveradi
* connection poolda leak bo‘lmaydi

## 4️⃣ asyncio.Semaphore
> Concurrency limit — N ta task bir vaqtning o‘zida ishlashi

**Pattern:**

```python
sem = asyncio.Semaphore(5)

async def worker():
    async with sem:
        await do_something()
```

→ Bir vaqtda 5 ta worker ishlaydi.

### Internals:

**Semaphore:**
* `value > 0` → immediate acquire
* `value == 0` → queue’da kutadi
* Release bo‘lganda next waiter wake qilinadi

**Diagram:**
```text
acquire → value -= 1  
release → value += 1  
queue’dagi task → wake
```

**Real example:** 1000 HTTP request, lekin faqat 10 parallel

```python
sem = asyncio.Semaphore(10)

async def fetch(url):
    async with sem:
        return await client.get(url)
```

## 5️⃣ asyncio.Lock
> Mutual exclusion (race condition oldini olish)

**Pattern:**

```python
lock = asyncio.Lock()

async def critical():
    async with lock:
        # only one task can enter here
```

### Internals:

1. `lock.locked()` tekshiriladi
2. agar `locked` → waiter queue ga qo‘shiladi
3. `unlock` → birinchi waiter wake qilinadi

**Diagram:**
```text
locked → queue
unlock → wake next waiter
```

**Real use-case:**
Shared counter

```python
counter = 0
lock = asyncio.Lock()

async def inc():
    global counter
    async with lock:
        counter += 1
```

## 6️⃣ Asyncio Cancellation Internals
> Ko‘p backendchilar noto‘g‘ri tushunadigan murakkab process

Cancellation — bu exception, aniqrog‘i:
➡ `asyncio.CancelledError`

**Task cancel qilinsa:**
1. Task.next iteration → raise `CancelledError`
2. Agar task ichida `await` yo‘q bo‘lsa, cancel darhol bo‘lmaydi
3. Cancel signal coroutine ichiga `await` chaqirilganda kiradi
4. `try/except CancelledError` bilan tutsa bo‘ladi
5. `finally` blok **HAR DOIM** ishga tushadi

**Misol:**
```python
task = asyncio.create_task(long_job())
task.cancel()

try:
    await task
except asyncio.CancelledError:
    print("Cancelled")
```

### ❗Important: cancellation → cooperative.

Agar coroutine ichida:

```python
while True:
    calc_cpu()  # no await
```

Bo‘lsa cancel hech qachon ishlamaydi.

**Yechim:**
```python
await asyncio.sleep(0)
```
Bu → control event loopga qaytadi.

### Cancellation cleanup (resource-safe pattern)

```python
async def worker():
    try:
        ...
    except asyncio.CancelledError:
        await cleanup()
        raise
```

If cleanup unutilsa → pool leak, lock leak, half-written file — production crash.

## 7️⃣ TaskGroup — STRUCTURED CONCURRENCY (Python 3.11+)
> TaskGroup = safer gather() + automatic cancellation trees

Modern concurrency modeli.

**Basic usage:**
```python
async with asyncio.TaskGroup() as tg:
    t1 = tg.create_task(job1())
    t2 = tg.create_task(job2())
```

### 🔬 TaskGroup Internals

1. TaskGroup kontekstga kiradi → boshqaruv strukturasi yaratiladi
2. Har `create_task` → monitor qilinadigan task yaratadi
3. Agar bitta task xato qilsa:
   * qolgan tasklar cancel qilinadi
4. TaskGroup exceptionni propagate qiladi
5. Exit paytida barcha tasklar tugaguncha kutadi

### TaskGroup = gather + cancellation tree

| Feature | gather | TaskGroup |
| :--- | :---: | :---: |
| cancel siblings | ❌ | ✔ |
| structured concurrency | ❌ | ✔ |
| exceptions grouping | ❌ | ✔ (ExceptionGroup) |
| recommended | ❌ | ✔ (modern) |

**Real example: parallel jobs, safe cancellation**

```python
async def handler():
    async with asyncio.TaskGroup() as tg:
        tg.create_task(fetch_user())
        tg.create_task(fetch_orders())
        tg.create_task(fetch_notifications())
```

Agar notification fetch umrboqiy bo‘lib ketsa → TaskGroup buni cancel qiladi.

**ExceptionGroup example**

```python
try:
    async with asyncio.TaskGroup() as tg:
        tg.create_task(f1())
        tg.create_task(f2())
except ExceptionGroup as eg:
    for exc in eg.exceptions:
        print("->", exc)
```

Gather buni qila olmaydi.

## 🧠 XULOSA

Bu bo‘limda siz async concurrency’ning eng murakkab va eng muhim mexanizmlarini o‘rgandingiz:

* **gather** — result aggregation + cancellation propagation
* **wait** — low-level done/pending orchestration
* **shield** — cancellation prevention
* **semaphore** — concurrency limiting (HTTP, DB concurrency)
* **lock** — critical sections & race condition prevention
* **cancellation internals** — cooperative abort, cleanup, exception flow
* **TaskGroup** — Python 3.11 structured concurrency (modern standard)

Bu bilimlar high-load backend, microservices, FastAPI, distributed systems, event-driven architectures uchun muhim.
