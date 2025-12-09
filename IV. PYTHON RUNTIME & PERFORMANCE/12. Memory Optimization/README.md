# 🧠 Python Memory Optimization — CPython Nimani Qanday Saqlaydi? (Advanced)

Pythonning xotira modeli oddiy emas — u object-oriented heap, reference counting, arena-based allocator (pymalloc) va garbage collector atrofida qurilgan.

Ushbu bo'limda quyidagilarni chuqur o'rganamiz:

- `sys.getsizeof()` — haqiqiy xotira emas, faqat obyekt strukturasi
- Python'da object allocation qanday ishlaydi
- `__slots__` yordamida RAMni 2–10 barobar tejash
- C-extensionlar orqali xotira va tezlikni optimallashtirish

---

## 1. sys.getsizeof() — Aslida nimani o'lchaydi?

`sys.getsizeof()` faqat obyektning ustki metadata + bazaviy xotirasini qaytaradi.

```python
import sys

print(sys.getsizeof(1))       # 28 bytes
print(sys.getsizeof("a"))     # 50 bytes
print(sys.getsizeof([]))      # 56 bytes
```

⚠️ **Ammo muhim cheklov:**

`sys.getsizeof()` obyekt ichidagi referenslarni hisoblamaydi.

**Misol:**

```python
lst = [1, 2, 3]
print(sys.getsizeof(lst))    # 64 bytes
```

Bu 64 byte — faqat ro'yxat "tubining" tuzilmasi.

**Lekin haqiqiy xotira:**

- 3 ta integer obyekt (har biri ~28 bytes)
- list head (~64 bytes)
- pointers (3 × 8 bytes)

👉 **Real xotira:** ~64 + 3×28 + 3×8 = 164 bytes  
`sys.getsizeof()` buni ko'rsatmaydi.

**To'liq xotirani o'lchash uchun?**

`pympler`, `tracemalloc`, `memory_profiler`, `objgraph`

---

## 2. CPython Memory Allocation — Arena → Pool → Block

Python "small object allocator" (pymalloc) orqali 512 bytegacha bo'lgan obyektlarni boshqaradi.

```
+-------------------+
| 32 KB Arena       |  --> OS dan olinadi
+-------------------+
| Pools (512 byte)  |  --> turli obyekt turlari uchun maydon
+-------------------+
| Blocks (8–512 b)  |  --> real obyektlar saqlanadi
+-------------------+
```

### 🧩 Obyektlar xotiraga qanday joylanadi?

- **0–512 byte obyektlar:** pymalloc (tezyurar)
- **512+ byte obyektlar:** `malloc()` orqali OS'dan olinadi
- **Katta obyektlar** fragmentatsiya keltirib chiqaradi

### 2.1 Reference Counting va Free List

Ko'pgina obyekt turlarida Python re-use strategiyasidan foydalanadi:

**Misollar:**

- 🟩 Integers 0–256 oldindan allocate qilinadi
- 🟩 Bo'sh tuple'lar, dict'lar, frame'lar free-list orqali qayta ishlatiladi

Bu shuni anglatadi:

```python
a = 10
b = 10
id(a) == id(b)  # True
```

---

## 3. __slots__ bilan RAM optimizatsiyasi

Odatda Python obyektining tarkibida `__dict__` bo'ladi — bu hash-map bo'lib, har bir attribute uchun entry saqlaydi.

**Bu juda katta xarajat:**

- Har bir obyekt → `__dict__` (kamida ~250–300 bytes)
- Har bir attribute → `PyObject*` pointeri (8 bytes)

**`__slots__` esa:**

- `__dict__`ni o'chiradi
- attribute'larni C-level fixed array sifatida saqlaydi
- Obyekt RAMini 2–10 baravar kamaytiradi

### 🧪 Misol:

```python
class User:
    def __init__(self, name, age):
        self.name = name
        self.age = age

class UserSlots:
    __slots__ = ("name", "age")
    def __init__(self, name, age):
        self.name = name
        self.age = age
```

**Xotira solishtirish:**

```python
import sys

u1 = User("a", 1)
u2 = UserSlots("a", 1)

print(sys.getsizeof(u1))   # ~360 bytes
print(sys.getsizeof(u2))   # ~56 bytes
```

🟩 **Farq:** 300 bytes dan ortiq tejaladi  
100,000 obyekt → ~30 MB tejash

### 3.1 __slots__ning cheklovlari

| Cheklov | Sabab |
|---------|-------|
| Yangi attribute qo'sha olmaysiz | `__dict__` yo'q |
| Inheritance ishlashda murakkablik | Slotlarni tarqatish kerak |
| Default qiymatlar bo'lmaydi | Har bir slot explicit yozilishi kerak |

---

## 4. Python Object Layout (C struct level)

Har bir Python obyektida minimal struktura bor:

```
PyObject
+------------------+
| Py_ssize_t refcnt|
+------------------+
| PyTypeObject* tp |
+------------------+
| payload          |
+------------------+
```

Integer uchun payload ~30 bytes.  
Shu sababli Python integerlari C-dagi integerdan 20× katta turadi.

---

## 5. C-Extensionlar → Performance + Memory Boost

Python sekin va xotirani ko'p ishlatadi, chunki:

- obyektlar juda og'ir
- dynamic typing
- interpreter overhead

**C-extensionlar quyidagilarni beradi:**

- 🚀 Aniq-tiplangan struct'lar orqali xotira 10–20x kamayadi
- 🚀 CPU instruction darajasidagi tezlik
- 🚀 Zero-overhead loops

### 👇 Misol: Pure Python vs Cython Struct

**Python versiyasi:**

```python
class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y
```

**Cython:**

```python
cdef class Point:
    cdef double x
    cdef double y
```

**Farqlar:**

| Mezonga | Python | Cython |
|---------|--------|--------|
| Xotira | ~300 bytes | 16 bytes |
| Tezlik | sekin | 100x tez |
| CPU integration | yo'q | to'liq |

---

## 6. Memory Profiling Tools (Real ADVANCED Level)

| Tool | Nima qiladi |
|------|-------------|
| `tracemalloc` | Allocations + snapshot profiling |
| `pympler` | Obyektlar hajmini aniq o'lchash |
| `memory_profiler` | Line-by-line memory profiler |
| `objgraph` | Obyekt graflari va leaklarni ko'rsatadi |

**Misol:**

```python
import tracemalloc

tracemalloc.start()

# your code here

print(tracemalloc.get_traced_memory())
```

---

## 7. Real Memory Optimization Cheatsheet

- 🟩 1. `__slots__` dan foydalaning
- 🟩 2. `list` emas, `tuple` ishlating
- 🟩 3. `dict` emas, `dataclass(slots=True)`
- 🟩 4. String duplication oldini olish → `sys.intern()`
- 🟩 5. Katta massivlar uchun → `array`, `numpy`, `memoryview`
- 🟩 6. Serialization uchun → `msgpack`, `protobuf`
- 🟩 7. Tight loops → Cython yoki Rust-extension

---

## 📌 Yakuniy ADVANCED-Level Xulosa

| Mavzu | Asosiy tushuncha |
|-------|------------------|
| `sys.getsizeof()` | Faqat obyekt "container" hajmini o'lchaydi, ichidagi referenslarni emas |
| Object allocation | Python heap → arenas, pools, blocks |
| `__slots__` | Xotirani 2–10x kamaytiradi, lekin dynamic attribute'larni bloklaydi |
| C-extensionlar | Pythonni haqiqiy compiled tildek tezlashtiradi |

Bu bo'limni o'zlashtirish Pythonning performance va memory usage bo'yicha ADVANCED-level optimizator bo'lishga imkon beradi.