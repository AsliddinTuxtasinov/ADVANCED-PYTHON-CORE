# 🧩 Closures & Advanced Scopes

## 📋 Mundarija

- [1. LEGB Rule — Advanced (bytecode orqali tushuntirish)](#1-legb-rule--advanced-bytecode-orqali-tushuntirish)
- [2. Closure — CPython ichki tuzilishi (cell, freevars)](#2-closure--cpython-ichki-tuzilishi-cell-freevars)
- [3. Backend'da closure'larning real ishlanmalari](#3-backendda-closurelarning-real-ishlanmalari)
- [Yakuniy advanced tushuncha](#-yakuniy-advanced-tushuncha)

---

## 1. LEGB Rule — Advanced (bytecode orqali tushuntirish)

### LEGB nima?

**LEGB** — Python'da o'zgaruvchilarni qidirish tartibi:

- **L** — Local (lokal scope)
- **E** — Enclosing (funksiya ichidagi tashqi scope)
- **G** — Global (global scope)
- **B** — Builtins (o'rnatilgan funksiyalar)

> **Muhim:** Python o'zgaruvchini qayerdan izlashni **kompilyatsiya vaqtida** hal qiladi, runtimeda emas.

### Misol

```python
x = 10

def outer():
    y = 20
    def inner():
        return x + y
    return inner
```

### Bytecode tahlili

```python
import dis
dis.dis(outer)
```

**Natija:**

```
LOAD_CONST               20 (y)
LOAD_CLOSURE             <cell x>
LOAD_CLOSURE             <cell y>
BUILD_TUPLE              2
LOAD_CONST               <function inner>
MAKE_FUNCTION            8 (closure)
RETURN_VALUE
```

**Muhim joy:**
- `LOAD_CLOSURE` → bu compiled function ichidagi free variable uchun cell obyekt yaratadi.

### Inner funksiyani tahlil qilish

```python
dis.dis(outer())
```

**Natija:**

```
LOAD_DEREF              x
LOAD_DEREF              y
BINARY_ADD
RETURN_VALUE
```

> 👉 **LOAD_DEREF** — closure-dan (cell) qiymatni olish.  
> Agar oddiy global bo'lsa, `LOAD_GLOBAL` ishlagan bo'lardi.

---

## 2. Closure — CPython ichki tuzilishi (cell, freevars)

### Closure nima?

**Python'da closure** — bu funksiya + uning tashqi scope'dagi o'zgaruvchilarining cell-lari.

### Misol

```python
def outer():
    a = 5
    def inner():
        return a
    return inner

fn = outer()
```

### Closure tekshirish

```python
fn.__closure__
```

**Natija:**

```python
(<cell at 0x...: int object at ...>,)
```

### Cell obyekt ichidagi qiymat

```python
fn.__closure__[0].cell_contents
```

**Natija:** `5`

---

### CPython closure qanday saqlaydi?

Har bir funksiya objectining quyidagi atributlari bor:

| Atribut | Tavsif |
|---------|--------|
| `co_freevars` | Tashqi scope-da bor o'zgaruvchilar nomlari |
| `co_cellvars` | Closure ichida foydalaniladigan local o'zgaruvchilar |
| `__closure__` | Real qiymatlarni saqlaydigan cell obyektlar |

**Tekshirish:**

```python
fn.__code__.co_freevars
```

**Natija:** `('a',)`

---

### ❗ Muhim CPython mexanizmi

Closure ishlashi uchun:

1. Tashqi funksiyadagi o'zgaruvchi **local** bo'lishi kerak
2. Uni ichki funksiya **ishlatishi** kerak

**Agar ishlatilmasa → closure emas.**

```python
def outer():
    a = 5
    def inner():
        return 10
    return inner

outer().__closure__
```

**Natija:** `None`

---

## 3. Backend'da closure'larning real ishlanmalari

Yuqori darajadagi backend kodda closure juda ko'p ishlatiladi.

Quyida **uchta eng real world misol**:

---

### 3.1. CACHE closure (Decorator ichida cache qo'llash)

Professional Python'da shu pattern juda ko'p uchraydi.

```python
def cached():
    store = {}  # closure ichida saqlanadi
    def decorator(func):
        def wrapper(arg):
            if arg in store:
                return store[arg]
            result = func(arg)
            store[arg] = result
            return result
        return wrapper
    return decorator


@cached()
def heavy_calc(x):
    print("Calculating...")
    return x * x
```

**Bu yerda:**

- `store` → closure
- Har bir `wrapper` → shu `store`'ni ko'radi
- Hech qanday global o'zgaruvchi yo'q

**Nima uchun bu professional level?**

- ✅ Thread-safety katta masala (cache lokal bo'ladi)
- ✅ Per-function cache izolyatsiya bo'ladi
- ✅ Dependency injection kerak emas

---

### 3.2. Factory pattern — closure orqali configurable function yaratish

Backend'da ko'p ishlatiladi: auth providers, client wrappers, etc.

```python
def serializer_factory(prefix):
    def serializer(data):
        return {prefix + k: v for k, v in data.items()}
    return serializer

user_serializer = serializer_factory("user_")
product_serializer = serializer_factory("product_")
```

**Natija:**

```python
user_serializer({"id": 1})
# → {'user_id': 1}
```

Bu **FastAPI/Django**-da common transform yaratayotganda juda foydali.

---

### 3.3. Config loader (Django/FastAPI microservices)

Microservice config'lar environment'dan o'qiladi.

Closure bilan config-ni memory'da saqlash mumkin:

```python
def config_loader():
    config = None

    def get_config():
        nonlocal config
        if config is None:  # caching once
            print("Loading config...")
            config = {
                "DB_URL": "postgres://...",
                "REDIS": "redis://..."
            }
        return config

    return get_config

get_config = config_loader()

cfg1 = get_config()  
cfg2 = get_config()  
```

**Natija (faqat birinchi chaqirilganda yuklanadi):**

```
Loading config...
```

**Bu pattern quyidagilar uchun juda keng ishlatiladi:**

- Microservices
- FastAPI dependency injection
- Expensive config parsing

---

## 🎯 Yakuniy advanced tushuncha

- ✅ **Closure** — runtime emas, **compile-time mexanizm**
- ✅ CPython closure ni **cell objects** orqali saqlaydi
- ✅ **LOAD_DEREF** — closure'dan o'qiydi
- ✅ Real backend'da closure **3 joyda** qo'llanadi:
  - **Caching**
  - **Factory**
  - **Config loader**

> Ular klassiz DI (dependency injection) uchun eng yengil variant.