# 🎨 Decoratorlar — Advanced

## 📋 Mundarija

- [1. Parameterized Decorators (Decorator inside Decorator)](#1-parameterized-decorators-decorator-inside-decorator)
- [2. Class Decorators — class'ni o'zgartirish / patch qilish](#2-class-decorators--classni-ozgartirish--patch-qilish)
- [3. Method Decorators — advanced method resolution](#3-method-decorators--advanced-method-resolution)
- [4. Functional Middleware Pattern (Django/FastAPI style)](#4-functional-middleware-pattern-djangofastapi-style)
- [Yakuniy Advanced Points](#-yakuniy-advanced-points)

---

## 1. Parameterized Decorators (Decorator inside Decorator)

Bu — decorator yaratishning **advanced usuli**, ya'ni decoratorning o'zi parametr qabul qiladi.

### Farqi

- **Oddiy decorator** → funksiyani qabul qiladi
- **Parameter decorator** → parametrlarni qabul qiladi → keyin asl decoratorni yaratadi

### ⚡ Strukturasi 3 qavat

```
decorator(param) → real_decorator(func) → wrapper(*args, **kwargs)
```

### Misol — logging level bilan decorator

```python
def logger(level):
    def decorator(func):
        def wrapper(*args, **kwargs):
            print(f"[{level}] Calling {func.__name__}")
            return func(*args, **kwargs)
        return wrapper
    return decorator


@logger("INFO")
def process():
    return "Done"
```

---

### ⚙ CPython jihati

`logger("INFO")` chaqirilganda, Python:

1. `logger("INFO")` → dekoratorni yasaydi
2. Natijada decorator obyektini qaytaradi
3. Keyin `func = decorator(func)` bajariladi

> **Muhim:** Bu mexanizm — `LOAD_CONST` + `CALL_FUNCTION` + `MAKE_FUNCTION` bytecode kombinatsiyasi bilan amalga oshadi.

---

### 🎯 Advanced: Parameter decorator bilan caching

```python
def cache(size=128):
    def decorator(func):
        store = {}
        def wrapper(x):
            if x in store:
                return store[x]
            if len(store) >= size:
                store.pop(next(iter(store)))
            store[x] = func(x)
            return store[x]
        return wrapper
    return decorator


@cache(size=2)
def slow(x):
    return x * x
```

---

## 2. Class Decorators — class'ni o'zgartirish / patch qilish

**Class decorator** — bu klass yaratilgandan keyin ishga tushadi, va class'ni:

- O'zgartirishi
- Yangi method qo'shishi
- Validatsiya qo'shishi

mumkin.

### Misol — klassga avtomatik `__repr__` qo'shish

```python
def auto_repr(cls):
    def __repr__(self):
        fields = ", ".join(f"{k}={v}" for k, v in self.__dict__.items())
        return f"{cls.__name__}({fields})"
    cls.__repr__ = __repr__
    return cls


@auto_repr
class User:
    def __init__(self, name, age):
        self.name = name
        self.age = age
```

**Natija:**

```python
print(User("Ali", 20))
# → User(name=Ali, age=20)
```

---

### 🎯 Advanced: Class decorator bilan validation qo'shish

```python
def validate_attrs(not_null_fields):
    def decorator(cls):
        orig_init = cls.__init__
        def new_init(self, *args, **kwargs):
            orig_init(self, *args, **kwargs)
            for f in not_null_fields:
                if getattr(self, f) is None:
                    raise ValueError(f"{f} may not be null")
        cls.__init__ = new_init
        return cls
    return decorator


@validate_attrs(["name"])
class User:
    def __init__(self, name=None):
        self.name = name
```

---

## 3. Method Decorators — advanced method resolution

**Method decoratorlar** — funksiya emas, descriptor protokoli bilan ishlaydi.

### Muhim farq

**Decorating a function:**
- Oddiy funksiya ichiga wrapper qo'shadi

**Decorating a method:**
- `function` → `descriptor` (function object)
- Class ichiga qo'yilganda `__get__` ishga tushadi
- `func.__get__(instance, owner)` → `MethodType(func, instance)` bo'ladi

---

### Misol — method-level throttling

```python
import time

def throttle(limit):
    def decorator(func):
        last_called = 0

        def wrapper(self, *args, **kwargs):
            nonlocal last_called
            now = time.time()
            if now - last_called < limit:
                raise Exception("Too many calls")
            last_called = now
            return func(self, *args, **kwargs)
        return wrapper
    return decorator


class API:
    @throttle(1)
    def request(self):
        return "OK"
```

> **Eslatma:** Bu yerda `self` avtomatik qo'shiladi (descriptor sabab).

---

## 4. Functional Middleware Pattern (Django/FastAPI style)

**Middleware** — bu request → response oqimiga funksiya "orasiga qo'shilish" patterni.

### Decorator orqali backend middleware yasash

**Universal middleware decorator:**

```python
def middleware(handler):
    def wrapper(request):
        print(f"Request: {request}")
        response = handler(request)
        print(f"Response: {response}")
        return response
    return wrapper
```

**Real example — logging middleware:**

```python
@middleware
def get_user(request):
    return {"id": 1}
```

**Chaqirish:**

```python
get_user({"path": "/user"})
```

---

### ⚡ Advanced Functional Middleware Chain (Express.js style)

Express.js ga o'xshash **"pipeline"** yaratish:

```python
def middleware(func):
    middlewares = []

    def use(m):
        middlewares.append(m)

    def wrapper(request):
        def call(i, req):
            if i == len(middlewares):
                return func(req)
            return middlewares[i](req, lambda new_req: call(i + 1, new_req))
        return call(0, request)

    wrapper.use = use
    return wrapper


@middleware
def handler(req):
    return req
```

**Middleware qo'shamiz:**

```python
@handler.use
def m1(req, next):
    req["a"] = 1
    return next(req)

@handler.use
def m2(req, next):
    req["b"] = 2
    return next(req)
```

**Chaqirish:**

```python
handler({})
```

**Natija:**

```python
{"a": 1, "b": 2}
```

**Bu professional-level backend pattern:**

- Chain-of-responsibility
- Pipeline architecture

---

## 🎓 Yakuniy Advanced Points

| Decorator turi | Asosiy g'oya | Qachon qo'llanadi |
|----------------|--------------|-------------------|
| **Parameterized** | Decorator konfiguratsiya qabul qiladi | caching, auth, rate limit |
| **Class decorator** | Klassni o'zgartiradi | validation, auto-repr, registering |
| **Method decorator** | Descriptor protokoli bilan ishlaydi | throttling, access control |
| **Middleware** | Pipeline yaratuvchi decorator | request/response transform |