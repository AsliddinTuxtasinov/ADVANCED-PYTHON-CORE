# 🌀 Python Asyncio — Event Loop Deep Dive
> Tasks, Future, Awaitable Protocol (Advanced)

## 🔥 KIRISH

Python’dagi **asyncio** — concurrency framework, lekin real parallelizm emas. Uning yuragi — **Event Loop**, u korutinalarni (coroutines) scheduling, IO operatsiyalarini boshqarish, callbacklar va timerlar bilan ishlaydi.

Agar siz backend (FastAPI/Django ASGI) yoki mikroservislar bilan ishlayotgan bo‘lsangiz, Event Loop’ni advanced darajada tushunish — majburiy.

## 🧠 EVENT LOOP NIMA?

**Event Loop** — bu infinite loop, quyidagi ishlarni bajaradi:

* 🔹 Rejada turgan callbacklarni bajaradi
* 🔹 I/O readiness ni kuzatadi (epoll, kqueue, IOCP)
* 🔹 Timeoutlar va Delaylarni boshqaradi
* 🔹 Tasklar progressini kuzatadi
* 🔹 Future va Task natijalarini qayta ishlaydi

> **Oddiy qilib:** "Event tayyor bo‘lganda" bajariladigan kodlarni boshqaradigan mexanizm.

### 🔄 Loop Lifecycle

```python
while True:
    run_ready_callbacks()
    run_scheduled_tasks()
    poll_io_events()
    handle_timeouts()
```

Event Loop **multi-thread emas**, concurrent ishlaydi — ya'ni bir vaqtning o‘zida bir nechta ishlarni parallel emas, balki almashlab bajaradi.

## 🧩 AWAITABLE PROTOKOLI

Asyncio’dagi istalgan obyekt "awaitable" bo‘lishi uchun quyidagi shartlardan birini bajarishi kerak:

1. ✔ **Coroutine** (`async def`)
2. ✔ **Future**
3. ✔ **Custom Awaitable** (ya’ni `__await__()` metodi mavjud)

### 🛠 Custom Awaitable yaratish

```python
import asyncio

class Delay:
    def __init__(self, seconds):
        self.seconds = seconds

    def __await__(self):
        print("Starting delay...")
        yield from asyncio.sleep(self.seconds).__await__()
        print("Delay finished!")
        return self.seconds
```

## 🎯 FUTURE (Low-level primitive)

* **Future** — natija keyin bo‘ladigan obyekt.
* **Future** — Task emas.
* **Task** — Future’ning executing versiyasi.
* **Future** — aslida Promise’ga o‘xshaydi.

### Future Lifecycle
`PENDING` → (`set_result` / `set_exception`) → `FINISHED`

### Future misoli:

```python
import asyncio

async def main():
    loop = asyncio.get_running_loop()
    fut = loop.create_future()

    loop.call_later(1, fut.set_result, "Done!")
    result = await fut
    print(result)

asyncio.run(main())
```

Callback orqali natija beriladi → `await fut` uni qaytaradi.

### Future + Callback

```python
fut.add_done_callback(lambda f: print("Completed:", f.result()))
```

Callback event loop’ning ready queueiga qo‘shiladi.

## 🚀 TASK (Coroutine Execution Manager)

**Task** — coroutine’ni event loop tomonidan ishlatishni boshlaydigan entity.

### Task yaratilishi:
```python
task = asyncio.create_task(my_coroutine())
```

Task avtomatik ravishda Futurega o‘xshab turadi:
`PENDING` → `RUNNING` → `FINISHED`

### Task vs Future — asosiy farqlar

| Future | Task |
| :--- | :--- |
| Natija holder | Coroutine runner |
| Qo‘lda natija beriladi | Coroutine natijasidan avtomatik natija olinadi |
| O‘zi bajarilmaydi | Event Loop Task’ni bajaradi |
| Low-level | High-level |

### Task Cancellation

```python
task.cancel()
try:
    await task
except asyncio.CancelledError:
    print("Task cancelled!")
```

Cancellation — bu cooperative, ya’ni coroutine ichida `await` bo‘lishi kerak.

## 🧬 EVENT LOOP ICHIDA NIMA BO‘LADI?

Event Loop quyidagilarni boshqaradi:

1. **Ready Queue (Microtasks)**
   * Callbacklar, Task continuations shu yerda saqlanadi.

2. **IO Polling**
   * `select`/`epoll`/`kqueue`/`IOCP` orqali I/O tayyor holatini kuzatadi.

3. **Timer Queue**
   * `call_later`, `call_at`, `sleep` kabi operatsiyalar shu yerda.

### Task Scheduling Flow

1. `asyncio.create_task(coro)` → Task yaratiladi
2. Task `PENDING` bo‘ladi
3. Event Loop uni **Ready Queue** ga qo‘yadi
4. Task birinchi `await`ga yetguncha ishlaydi
5. Control qaytadi
6. Await bo‘lgan Future tayyor bo‘lganda Task davom ettiriladi

## 🏗 HIGH-LEVEL PATTERNS

### ◾ Task Group (Python 3.11+)

```python
async with asyncio.TaskGroup() as tg:
    tg.create_task(worker(1))
    tg.create_task(worker(2))
```

**TaskGroup:**
* barcha tasklar tugaguncha kutadi
* xatolik bo‘lsa, qolganlarni cancel qiladi

### ◾ Shielding (cancellationni bloklash)

```python
await asyncio.shield(my_task)
```

Task cancel qilinmaydi → cancel signal faqat `shield()` ga ta'sir qiladi.

### ◾ Waiters Pattern

```python
done, pending = await asyncio.wait(tasks, timeout=2)
```

## 🧪 DEBUG & MONITORING

Event Loop debug holati:
```python
asyncio.run(main(), debug=True)
```

Slow callbacklar:
```python
asyncio.get_running_loop().slow_callback_duration = 0.005
```

Tracing:
```bash
PYTHONASYNCIODEBUG=1 python app.py
```

## 📘 XULOSA

Bu bo‘lim sizga quyidagilarni advanced darajada tushuntirdi:

* Event Loop ichki mexanizmlari
* Awaitable protocol (custom `__await__`)
* Future va Task farqlari
* Task scheduling, cancellation, callbacks
* IO polling va microtask queue
* High-level concurrency patterns

Bu bilimlar sizni FastAPI, microservices, high-load backend, distributed async systemsda professional darajaga olib chiqadi.

---

### 🔷 1. EVENT LOOP — DIAGRAMLAR BILAN TUSHUNTIRISH

#### 🔁 Event Loop Architecture Diagram

```text
               ┌───────────────────────┐
               │      Your Code        │
               │    (coroutines)       │
               └───────────▲───────────┘
                           │ await
                           │
                 ┌─────────┴──────────┐
                 │    Event Loop      │
                 └─────────┬──────────┘
                           │
        ┌──────────────────┼───────────────────┐
        │                  │                   │
┌───────▼───────┐   ┌──────▼───────┐   ┌──────▼────────┐
│  Ready Queue  │   │   IO Poller  │   │ Timer Queue   │
│ (microtasks)  │   │ (epoll/kqueue│   │ (call_later)  │
└───────▲───────┘   │   /IOCP)     │   └──────▲────────┘
        │           └──────▲───────┘          │
        │   wake            │ readiness       │ timeout
        │                   │                 │
┌───────┴────────┐   ┌──────┴────────┐     ┌──┴────────────┐
│   Task/Future  │   │ Clocked Tasks │     │ Scheduled     │
│  awaitability  │   │ awaiting IO   │     │ callbacks     │
└────────────────┘   └───────────────┘     └───────────────┘
```

Event Loop hech qachon “thread” ochmaydi — faqat I/O readiness bo‘lganda davom ettiradi.

### 🔷 2. REAL-LIFE PRODUCTION MISOLLARI

#### 🏭 Misol 1: High-load FastAPI service-da DB va HTTP parallel bajarilishi

**Muammo:**
Sizda PostgreSQL so‘rovi va tashqi API request bor. Ikkalasini ketma-ket bajarish 200–400ms vaqt oladi.

**Yechim — TaskGroup bilan parallel bajarish:**

```python
import asyncio
import httpx
import asyncpg

async def get_user_from_db(conn, user_id):
    return await conn.fetchrow("SELECT * FROM users WHERE id=$1", user_id)

async def get_user_profile(user_id):
    async with httpx.AsyncClient() as client:
        return (await client.get(f"https://external.api/user/{user_id}")).json()

async def orchestrate(user_id):
    conn = await asyncpg.connect("postgres://...")

    async with asyncio.TaskGroup() as tg:
        db_task = tg.create_task(get_user_from_db(conn, user_id))
        api_task = tg.create_task(get_user_profile(user_id))

    # natijalar TaskGroup'dan chiqayotganda tayyor bo‘ladi
    return {
        "db": db_task.result(),
        "profile": api_task.result()
    }
```

**Production foydasi:**
* Latency 40–70% ga kamayadi
* Server throughput oshadi
* Bitta request boshqa requestni bloklamaydi

#### 🏭 Misol 2: Queue asosidagi background worker

Background tasklar uchun `asyncio.Queue` real-life pattern:

```python
queue = asyncio.Queue()

async def worker():
    while True:
        job = await queue.get()
        try:
            print("Processing:", job)
            await asyncio.sleep(2)
        finally:
            queue.task_done()

async def producer():
    for i in range(10):
        await queue.put(f"job-{i}")

async def main():
    workers = [asyncio.create_task(worker()) for _ in range(3)]
    await producer()
    await queue.join()

    for w in workers:
        w.cancel()

asyncio.run(main())
```

**Foyda:**
* — 0 thread — 1000 thread ochilmaydi
* — Ishlar parallel bo‘ladi
* — Worker pool oddiy async bilan yaratiladi

### 🔷 3. FASTAPI EVENT LOOP INTERNALS

`FastAPI` → `Starlette` → `ASGI` → `uvloop` (yoki `asyncio`)

#### 🧬 Request kelganda nima bo‘ladi?

**Diagram:**
`Client` → `ASGI Server (uvicorn/uvloop)` → `Event Loop` → `FastAPI router` → `Your async view`

**Ishlash tartibi:**
1. Uvicorn bir dona Event Loop yaratadi
2. Har bir HTTP request ASGI scope sifatida qabul qilinadi
3. Event Loop requestni coroutine sifatida FastAPI’ga uzatadi
4. Siz `await` qilgan joyda control loopga qaytadi
5. DB, redis, httpx I/O tayyor bo‘lganda Task qayta davom etadi
6. Response ASGI orqali qaytariladi

#### 🔍 Nima uchun FastAPI juda tez?

* **uvloop** (libuv asosida) — Node.js event loop’idan ham tez
* **Multi-thread emas** — context switch yo‘q
* **Concurrency** → 10k requestlarni bir vaqtda boshqarish
* **ASGI stack** — minimal overhead

#### 🚨 Yana bir muhim narsa:

FastAPI endpoint ichida **blocking code** ishlatsangiz — event loopni o‘ldirasiz!

**Yomon:**

```python
def slow():
    time.sleep(3)

@app.get("/")
async def index():
    slow()  # ♻️ event loop bloklandi
```

**To‘g‘ri:**

```python
@app.get("/")
async def index():
    await asyncio.to_thread(slow)
```

### 🔷 4. CUSTOM EVENT LOOP SCHEDULER (Mini Implementation)

Quyidagi kod event loopning ishlashini minimal darajada o‘zingizga ko‘rsatadi.

#### 🛠 Custom Scheduler — Tasklarni manual boshqarish

```python
import heapq
import time

class Scheduler:
    def __init__(self):
        self.ready = []      # immediate queue
        self.sleeping = []   # (wake_time, task)
    
    def call_soon(self, coro):
        self.ready.append(coro)
    
    def call_later(self, delay, coro):
        wake = time.time() + delay
        heapq.heappush(self.sleeping, (wake, coro))
    
    def run(self):
        while self.ready or self.sleeping:
            if not self.ready:
                wake, coro = heapq.heappop(self.sleeping)
                sleep = max(0, wake - time.time())
                if sleep:
                    time.sleep(sleep)
                self.ready.append(coro)

            coro = self.ready.pop(0)
            try:
                delay = coro.send(None)   # await → yields delay
                self.call_later(delay, coro)
            except StopIteration:
                pass

# -------- example coroutine ----------
def countdown(n):
    while n > 0:
        print("Tick", n)
        yield 1     # sleep 1 second
        n -= 1

sched = Scheduler()
sched.call_soon(countdown(3))
sched.run()
```

**Natija:**
```text
Tick 3
Tick 2
Tick 1
```

Bu — asyncio yo‘qligida event loop qanday ishlashini ko‘rsatadigan mini scheduler.

#### 🔍 Bu sizga nima beradi?

* Event loop ichki mexanizmlarini yanada chuqur tushunasiz
* `yield` va `await` o‘rtasidagi bog‘liqlikni ko‘rasiz
* Future/Task scheduling aslida qanday ishlashini anglaysiz

### 🔷 BONUS: REAL SYSTEM DESIGN MISOL (ASYNC)

#### 🔥 10k long-polling clientlar ulanishi

**Event Loop asosida:**
* Har bir client uchun Task yaratasiz
* Server 10k connectionni bitta thread bilan boshqaradi
* Hech qanday thread-pool zarur emas

**Misol:**

```python
async def handle_client(socket):
    while True:
        data = await socket.recv()
        await socket.send(process(data))

async def main():
    tasks = []
    server = await create_server()
    async for client in server:
        tasks.append(asyncio.create_task(handle_client(client)))

asyncio.run(main())
```

Thread'larda bu imkonsiz bo‘lgan joyda **asyncio** — super-cheap concurrency beradi.
