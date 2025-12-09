## ✅ 2.1. Python’da hamma narsa obyekt!

Yozgan har bir narsa — class instance.

Misol:

```python
x = 5
```

Bu yerda `5` → `int` klassining obyektidir.

## ✅ 2.2. id() nima? (C-level pointer)

Ko‘pchilik `id()` faqat “unikal identifikator” deb o‘ylaydi — lekin SENIOR darajada bilish kerak:

> 📌 `id(obj)` → RAMdagi obyektning C-pointeri

```python
x = 10
print(id(x))  # C-level address
```

Ammo bu faqat CPython uchun to‘g‘ri. PyPy, Jython boshqacha.

## ✅ 2.3. type() va __class__ farqi

Python’da:

```python
type(obj) is obj.__class__
```

Har bir obyekt RAMda pointer orqali o‘z classiga bog‘langan.

Misol:

```python
x = 10
print(type(x))        # <class 'int'>
print(x.__class__)    # <class 'int'>
```

Lekin advanced holatlar bor:
`type()` override qilinganda, proxy objectlarda, metaclasslarda farq qiladi.

## ✅ 2.4. __dict__ — obyektning attribute storage’i

Oddiy class obyektining atributlari dictionaryda saqlanadi.

```python
class A:
    def __init__(self):
        self.x = 10

a = A()
print(a.__dict__)
```

Output:

```python
{'x': 10}
```

Bu nega muhim?

🚀 Chunki `__dict__` juda ko‘p RAM yeydi.
Har bir obyekt uchun alohida dict → katta systemlarda RAM portlashiga sabab bo‘ladi.

## 🚀 2.5. __slots__ — RAMni 40–60% kamaytiradi

Agar sen `__slots__` ishlatsang:

- ✔ `__dict__` O‘CHADI
- ✔ Obyekt attributlari dict’da emas → C-level array’da saqlanadi
- ✔ RAM 2–4 barobar tejaladi
- ✔ Performance oshadi

Misol:

```python
class User:
    __slots__ = ("id", "name")
    def __init__(self, id, name):
        self.id = id
        self.name = name
```

Endi:

```python
u = User(1, "Asliddin")
print(u.__dict__)  # ERROR: 'User' object has no attribute '__dict__'
```

User obyektlari uchun endi hech qachon dict yaratilmaydi.

### 📌 QACHON __slots__ ishlatish kerak?

Juda foydali:

- **Ko‘p obyekt yaratiladigan systemlar**: chat messages, sensors, logs, tokens, ORM metadata, celery tasks
- **Microservices performance**

Keraksiz:

- Dynamic attribute qo‘shmoqchi bo‘lsang
- Multiple inheritance

### ⭐ REAL PERFORMANCE TEST

```python
import sys

class A:
    def __init__(self):
        self.x = 1
        self.y = 2

class B:
    __slots__ = ("x", "y")
    def __init__(self):
        self.x = 1
        self.y = 2

a = A()
b = B()

print(sys.getsizeof(a))  # 48 bytes + dict ≈ 112 bytes total
print(sys.getsizeof(b))  # ~48 bytes
```

RAM 2x kamroq.

## 🔥 2.6. Python Memory Model (stack → heap)

Python’da:

- **Stack** → faqat pointerlar saqlanadi
- **Heap** → real obyektlar saqlanadi

`x = 10` qilganda:

> Stack: pointer → Heap: <int object 10>

Shuning uchun:

```python
a = [1, 2, 3]
b = a
```

`b` faqat pointerni ko‘chiradi — listni emas.

## 🔥 2.7. Reference Counting (CPythonning asosiy mexanizmi)

Har bir obyekt nechta pointer unga qarab turganini hisoblaydi.

Misol:

```python
import sys

x = []
print(sys.getrefcount(x))  # 2
```

> ❗ 1 + Py tonning ichki reference

Har safar pointer qo‘shilsa:

```python
y = x
```

refcount +1 bo‘ladi.

Agar 0 bo‘lsa → obyekt O‘CHIRILADI.

## 🔥 2.8. Garbage Collector (Cyclic GC)

Reference counting bitta muammo beradi:

➡ Agar ikkita obyekt bir-biriga ishora qilsa — hech qachon refcount 0 bo‘lmaydi.

Shu uchun Python’da yana qo‘shimcha GC bor:

📌 3 ta generatsiya:

1. **gen0**
2. **gen1**
3. **gen2**

Gen0 tez-tez tozalanadi, gen2 juda kam.

GC cyclic obyektlarni topib, o‘chiradi:

```python
import gc
print(gc.get_stats())
```

### 🧨 MUHIM XULOSA:

Python’dagi OOP, classlar, async, ORM — HAMMASI shu object modelga tayanadi.