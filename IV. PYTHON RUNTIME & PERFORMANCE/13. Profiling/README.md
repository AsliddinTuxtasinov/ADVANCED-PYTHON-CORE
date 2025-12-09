# 🧠 Python Performance Profiling (Advanced Guide)

Profiling — bu Python dasturining qayerda sekinlashayotganini aniqlash jarayoni.

Advanced-level backend dasturchi uchun profiling majburiy skill, chunki:

- Optimallashtirish → o'lchovdan boshlanadi
- Performance debugging → profiling orqali
- Bottleneck'lar intuitiv emas — ko'p hollarda kutilmagan joyda yashirinadi

Ushbu bo'lim quyidagilarni qamrab oladi:

- `cProfile` — built-in deterministic profiler
- `line_profiler` — qaysi satr sekinligini aniq ko'rsatadi
- `py-spy` — production-level sampler profiler (GIL-safe, zero-overhead)
- Bottleneck aniqlash strategiyalari
- FastAPI & Django uchun request-level profiling

---

## 1. cProfile — Pythonning rasmiy profiler'i

`cProfile` — CPythonning built-in deterministic profiler.

### 🔹 Qanday ishlaydi?

U har bir funksiyaning:

- umumiy ishlash vaqti
- chaqirilgan soni
- recursive chaqiriqlar
- cumulative time (bolalari bilan)
- self time (faqat o'zi)

ma'lumotlarini yig'adi.

### 🧪 Misol:

```python
import cProfile

def slow_func():
    for _ in range(10_000_000):
        pass

cProfile.run("slow_func()")
```

**Natija (qisqartirilgan):**

```
ncalls  tottime  percall  cumtime  function
1       0.320    0.320    0.320    slow_func
```

### 🔍 Profiling natijalarini chiroyli ko'rish — pstats + snakeviz

**Install:**

```bash
pip install snakeviz
```

**Profil yaratish:**

```python
import cProfile

cProfile.run("slow_func()", "out.prof")
```

**Vizual ishlatish:**

```bash
snakeviz out.prof
```

🍀 **Bundan yaxshiroq vizualizator yo'q.**

---

## 2. line_profiler — Satr darajasida profiling (Advanced)

`cProfile` funksiyalar bo'yicha profiling qiladi.  
Lekin ba'zida funksiya ichidagi qaysi satr eng sekinligi muhim.

Shu uchun `line_profiler`.

**Install:**

```bash
pip install line_profiler
```

### 🧪 Misol:

```python
from line_profiler import LineProfiler

def compute():
    x = [i for i in range(10_000_000)]
    s = sum(x)
    return s

lp = LineProfiler()
lp.add_function(compute)
lp.run('compute()')
lp.print_stats()
```

**Natija:**

```
Line #    Hits     Time  Per Hit   % Time  Line Contents
--------------------------------------------------------
     2                                         x = [...]
     3                                         s = sum(x)
```

`line_profiler` satr bo'yicha exact timing beradi — bu ultra-muhim.

---

## 3. py-spy — production profiling uchun eng kuchli vosita

`py-spy` — sampling profiler, Python kodiga tegmaydi, GILdan qorqmaydi va:

- production serverda ishlaydi
- source code ga tegmaydi
- overhead juda kichik (~1–2%)
- lock qilmaydi
- har qanday Python versiyasida ishlaydi

**Install:**

```bash
pip install py-spy
```

### 🔥 Running on live Python process

**Jarayonlarni ko'rish:**

```bash
py-spy top
```

**Pid bo'yicha profiling:**

```bash
py-spy top --pid 12345
```

**Flamegraph yaratish:**

```bash
py-spy record -o profile.svg --pid 12345
```

**Ochish:**

```bash
profile.svg
```

### 🔥 Nima uchun bu eng kuchlisi?

- CPython interpreter memory read orqali profiling qiladi (zero instrumentation)
- Katta serverlarda ham ishlaydi
- Worker pool (Gunicorn/Uvicorn/Django) bilan mos keladi

Advanced backendchilar production profiling uchun buni ishlatadi.

---

## 4. Bottleneck aniqlash strategiyasi (Advanced-level)

### 🔹 1. Always measure first

"Bu kod sekin bo'lsa kerak" → xato mindset.  
Profiling har doim kutilmagan joyni ko'rsatadi.

### 🔹 2. CPU-bound mi yoki I/O-bound?

- `cProfile` CPU-bound uchun
- `py-spy` har xil vaziyatlar uchun idealdir

### 🔹 3. Satr darajasida profiling → line-profiler

Ayniqsa:

- katta looplar
- list/dict comprehensions
- massiv operatsiyalar
- web endpoint ichidagi og'ir transformlar

### 🔹 4. Always inspect cumulative time

Ko'pincha bottleneck → funksiya ichida emas, uning chaqirilayotgan yordamchilarida.

### 🔹 5. Flamegraph → Eng aniq analiz usuli

Call-stack ichidagi vaqt taqsimotini vizual ko'rsatadi.

---

## 5. Request-Level Profiling (FastAPI & Django)

Backend developer uchun eng muhim qism — endpoint qayerda sekinlashyapti?

### 🔥 FASTAPI PROFILING (Advanced Middleware)

```python
import time
from fastapi import FastAPI, Request

app = FastAPI()

@app.middleware("http")
async def profiler(request: Request, call_next):
    start = time.perf_counter()
    response = await call_next(request)
    duration = (time.perf_counter() - start) * 1000
    print(f"{request.url.path} took {duration:.2f} ms")
    return response
```

**Yoki:**

### 🔥 pyinstrument bilan professional profiling

```bash
pip install pyinstrument
```

```python
from pyinstrument import Profiler
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
def index():
    profiler = Profiler()
    profiler.start()

    # your business logic here

    profiler.stop()
    print(profiler.output_text(unicode=True, color=True))
```

---

## 🔥 DJANGO VIEW PROFILING (Production-style)

### 1) Django Debug Toolbar

Development uchun:

```bash
pip install django-debug-toolbar
```

### 2) Django Silk (production profiling uchun)

```bash
pip install django-silk
```

**Settings:**

```python
INSTALLED_APPS += ['silk']
MIDDLEWARE += ['silk.middleware.SilkyMiddleware']
```

**Endpointlar:**

```
/silk/profile
```

**Silk:**

- view vaqtini
- SQL querylarni
- call graphlarni
- profiling loglarini

real vaqt rejimida ko'rsatadi.

### 3) py-spy with Django

Gunicorn worker PID topib:

```bash
py-spy record -o django.svg --pid 12345
```

---

## Yakuniy xulosa (Advanced-Level)

| Profiling turi | Qachon ishlatiladi |
|----------------|-------------------|
| `cProfile` | Funksiya-level CPU profiling |
| `line_profiler` | Satr darajasida bottleneck aniqlash |
| `py-spy` | Production profiling (no overhead) |
| `snakeviz` | Vizual analiz (call graph) |
| Middleware profiling | FastAPI/Django endpoint bottlenecklari |
| Flamegraph | Murakkab system performance tahlili |

Real tajribada — ushbu vositalarning uchtasi birga ishlatiladi.

---

# ✅ 1 — REAL FASTAPI PROFILING (ADVANCED)

Quyida prod-level FastAPI profilingning to'liq amaliy qo'llanmasi beriladi.

## 🚀 REAL FASTAPI PERFORMANCE PROFILING (Advanced)

FastAPI juda tez, lekin sizning biznes logikingiz, SQL querylar yoki JSON serialization sekin bo'lishi mumkin. Profiling orqali endpointlarning real bottlenecklarini topamiz.

### 1. Middleware orqali REQUEST-LEVEL PROFILING (Low overhead)

```python
import time
from fastapi import FastAPI, Request

app = FastAPI()

@app.middleware("http")
async def profiler(request: Request, call_next):
    start = time.perf_counter()
    response = await call_next(request)
    duration = (time.perf_counter() - start) * 1000
    print(f"[PROFILE] {request.url.path}: {duration:.2f} ms")
    return response
```

### 2. pyinstrument bilan ADVANCED PROFILING (Eng foydalisi)

**Install:**

```bash
pip install pyinstrument
```

**Endpointni profiling qilish:**

```python
from pyinstrument import Profiler
from fastapi import FastAPI

app = FastAPI()

@app.get("/reports")
def heavy():
    profiler = Profiler()
    profiler.start()

    # YOUR HEAVY BUSINESS LOGIC
    result = sum([i*i for i in range(10_000_000)])

    profiler.stop()
    print(profiler.output_text(unicode=True, color=True))
    return {"result": result}
```

**Natija:**

- Function call tree
- Time breakdown
- Bottleneck joylar

### 3. py-spy bilan PRODUCTION PROFILING

FastAPI Uvicorn + Gunicorn bilan ishlaganda pid topib:

```bash
ps aux | grep gunicorn
```

Keyin:

```bash
py-spy record -o fastapi.svg --pid 12345
```

Hosil bo'lgan SVG flamegraph → eng aniq profiling shakli.

### 4. SQL profiling (SQLAlchemy)

```python
from sqlalchemy import event
from sqlalchemy.engine import Engine

@event.listens_for(Engine, "before_cursor_execute")
def before_cursor_execute(conn, cursor, statement, params, context, executemany):
    context._query_start_time = time.time()

@event.listens_for(Engine, "after_cursor_execute")
def after_cursor_execute(conn, cursor, statement, params, context, executemany):
    total = (time.time() - context._query_start_time) * 1000
    print(f"SQL Time: {total:.3f}ms → {statement[:80]}")
```

### 5. FastAPI profiling bo'yicha Production Checklist

| Nima qilish kerak | Sababi |
|-------------------|--------|
| `uvicorn --workers=4` | GIL cheklovini kamaytiradi |
| `orjson` serializer | JSON perf 6–10x oshadi |
| Connection pool (asyncpg) | DB latency kamayadi |
| `py-spy` profiling | zero-overhead, production-safe |

---

# ✅ 2 — REAL DJANGO PROJECT PROFILING (ADVANCED)

Endi to'liq Django profilingini beraman.

## 🧵 REAL DJANGO PROFILING (Advanced)

Django profiling uch qismda beriladi:

- Development profiling
- Production profiling
- SQL-level profiling

### 1. DEVELOPMENT PROFILING — Django Debug Toolbar

**Install:**

```bash
pip install django-debug-toolbar
```

**settings.py:**

```python
INSTALLED_APPS += ['debug_toolbar']
MIDDLEWARE = ['debug_toolbar.middleware.DebugToolbarMiddleware'] + MIDDLEWARE
```

**`/__debug__/` orqali:**

- SQL vaqti
- Template rendering
- Cache hit/miss
- View execution time

hammasini ko'rasiz.

### 2. Silk — ADVANCED DJANGO PROFILER

**Install:**

```bash
pip install django-silk
```

**settings.py:**

```python
INSTALLED_APPS += ['silk']
MIDDLEWARE += ['silk.middleware.SilkyMiddleware']
```

**Run:**

```
/silk/
```

**Silk:**

- View profiling
- Function profiling
- SQL call graph
- Flamegraphs
- Request timeline

hammasini beradi.

### 3. Production profiling — py-spy

Django Gunicorn worker pidini topamiz:

```bash
ps aux | grep gunicorn
```

**Profiling:**

```bash
py-spy record -o django.svg --pid 12345
```

**Flamegraph → production bottlenecklarni ko'rsatadi:**

- ORM slow queries
- JSON serialization
- Heavy Python loops
- Blocking I/O

### 4. Django ORM profiling

```python
from django.db import connection
from time import perf_counter

def my_view(request):
    start = perf_counter()

    result = list(MyModel.objects.all())

    print(connection.queries)
    print("Execution time:", perf_counter() - start)
```

### 5. Django caching bottlenecklarini topish

```python
from django.core.cache import cache
import time

start = time.perf_counter()
data = cache.get("users")
print("Cache read:", (time.perf_counter() - start) * 1000, "ms")
```

---

# ✅ 3 — CPU / GIL / I/O ADVANCED DIAGNOSTICS

Bu eng chuqur bo'lim.

## ⚙️ ADVANCED CPU, GIL, and I/O DIAGNOSTICS

Python performance debugging uch asosiy turga bo'linadi:

- CPU-bound performance
- GIL contention
- I/O latency bottlenecklari

Har birini qaysi vosita bilan tahlil qilishni ko'rsataman.

### 1. CPU-BOUND profiling

`py-spy top` bilan CPU hotspotlarni ko'ramiz:

```bash
py-spy top --pid 12345
```

**Natija:**

- qaysi funksiya CPUning ko'p qismini olgan
- qaysi thread GILni ushlab o'tirgan
- real-time CPU load

### 2. GIL contention diagnostikasi

**py-spy GIL indikatorlari**

Flamegraphda `<gil>` degan blok ko'rsangiz → threading samarasiz.

```
| <gil> → busy
| <gil> → blocking
```

**Diagnostika:**

Agar CPU-bound ish → threading → xato.

**To'g'ri yechim:**

- `multiprocessing`
- `joblib`
- `ray`
- `cython`
- yoki Rust extension

### 3. I/O bottleneck diagnostikasi

1. Slow DB queries → Django yoki FastAPI SQL logging
2. Slow external API calls
3. Large serialization (Pydantic / DRF)
4. Network round-trip latency

**Diagnostika:**

```python
import time
from httpx import AsyncClient

async def fetch():
    start = time.time()
    await AsyncClient().get("https://example.com")
    print("I/O time:", time.time() - start)
```

### 4. Event-loop diagnostika (FastAPI)

```bash
uvloop install
```

Keyin:

```bash
uvicorn app:app --loop uvloop
```

**Profiling:**

```bash
py-spy top --pid 12345
```

Agar ko'p vaqt `event_loop` ichida ketayotgan bo'lsa → locking yoki await chain muammosi.
