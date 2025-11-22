## 🧠 1. DESCRIPTOR PROTOCOL

Python’da descriptor — bu quyidagi methodlardan bittasini yoki barchasini implement qilgan class:

```python
class Descriptor:
    def __get__(self, instance, owner):
        ...
    def __set__(self, instance, value):
        ...
    def __delete__(self, instance):
        ...
```

Va descriptor boshqa klassning atributi sifatida ishlatiladi:

```python
class User:
    name = Descriptor()
```

Descriptorning vazifasi:
atributga kirish, yozish, o‘chirish jarayonini nazorat qilish.

## 🚀 2. __get__ — atributni o‘qish

`__get__(self, instance, owner)`:

- `instance` → obyekt (u = User() bo‘lsa instance = u)
- `owner` → classning o‘zi (User)

Misol: o‘qishda qiymatni modifikatsiya qiladigan descriptor

```python
class Upper:
    def __get__(self, instance, owner):
        return instance.__dict__['name'].upper()
```

Foydalanish:

```python
class User:
    name = Upper()

u = User()
u.name = "asliddin"
print(u.name)   # ASLIDDIN
```

## 🔥 3. __set__ — qiymatni yozishni boshqarish

```python
class Positive:
    def __set__(self, instance, value):
        if value < 0:
            raise ValueError("Manfiy bo'lishi mumkin emas")
        instance.__dict__['age'] = value
```

Ishlatish:

```python
class User:
    age = Positive()

u = User()
u.age = 25   # OK
u.age = -5   # ERROR
```

Bu — value validation descriptor.

## 🧨 4. __delete__ — atributni o‘chirishni boshqarish

```python
class ProtectDelete:
    def __delete__(self, instance):
        raise AttributeError("Bu fieldni o‘chira olmaysan!")
```

Ishlatish:

```python
class User:
    token = ProtectDelete()

u = User()
del u.token   # ERROR
```

## 🧩 5. property — descriptorning maxsus ko‘rinishi

`@property` — aslida descriptor class:

```python
class property:
    def __get__(self, instance, owner): ...
    def __set__(self, instance, value): ...
    def __delete__(self, instance): ...
```

Shuning uchun:

```python
class User:
    @property
    def name(self):
        return "Asliddin"
```

Ishlash mexanizmi descriptorga asoslangan.

## 🧠 6. cached_property — ADVANCED darajadagi descriptor

`cached_property` birinchi chaqirilganda:

1. methodni chaqiradi
2. natijani `instance.__dict__`ga saqlaydi
3. keyingi chaqiriqlarda — descriptor ishlamaydi
4. bevosita dict’dan o‘qiydi → juda tez

KOD:

```python
class cached_property:
    def __init__(self, func):
        self.func = func

    def __get__(self, instance, owner):
        if instance is None:
            return self
        value = self.func(instance)
        instance.__dict__[self.func.__name__] = value
        return value
```

Demak:

- Birinchi marta descriptor ishlaydi
- Keyin umuman ishlamaydi → to‘g‘ridan-to‘g‘ri value qaytadi

Bu lazy-loaded cache mexanizmi

## 🔥 7. Django ORM DESCRIPTORLARI (Real, ADVANCED daraja)

Django’da:

```python
class User(models.Model):
    name = models.CharField(...)
```

Bu `name` — descriptor.

Django Field qachon ishlaydi?

Atributga yozilganda:

```python
u.name = "Ali"
```

Django quyidagiga aylantiradi:

```python
User._meta.get_field("name").__set__(u, "Ali")
```

`__set__` ichida:

- valoidaydi
- modelni "dirty" qiladi
- change tracking
- form cleaning
- type conversion
- DB serialization
- relational field logic

Atributni o‘qiganda esa:

```python
User._meta.get_field("name").__get__(u, User)
```

Bu — ORM magic.

### 🧨 Django’da ForeignKey descriptorlari ikkita bo‘ladi:

1. **Forward Descriptor**
   `user.profile` → object qaytaradi

2. **Reverse Descriptor**
   `profile.user_set` → QuerySet qaytaradi

Reverse relationship — ham descriptor!

## 🧩 8. Real Django misoli: CharField descriptor

Simplified:

```python
class CharField:
    def __get__(self, instance, owner):
        return instance.__dict__[self.name]

    def __set__(self, instance, value):
        if not isinstance(value, str):
            raise ValueError("String bo'lishi kerak")
        instance.__dict__[self.name] = value
```

Django bundan 10x murakkabroq qilsa ham, asos — descriptor.

## 🚀 9. Real FastAPI misol — Lazy dependency descriptor

FastAPI dependencylar ham descriptor-like behavior qiladi:

```python
def get_db():
    db = Session()
    try:
        yield db
    finally:
        db.close()
```

Bu “lazy-loaded attribute” sifatida ishlaydi.

## 🚨 10. Descriptor — ADVANCED Pythonning yuragi

- ✔ ORM
- ✔ property
- ✔ cached_property
- ✔ validatorlar
- ✔ lazy fieldlar
- ✔ dependencylar
- ✔ computed fieldlar
- ✔ secure attribute access
- ✔ attribute-level middleware

Hammasi descriptor orqali qurilgan.

## 🎯 Yakun

Biz descriptor protokolning:

- ✔ `__get__` mexanizmini
- ✔ `__set__` mexanizmini
- ✔ `__delete__` mexanizmini
- ✔ `property` descriptor tuzilishini
- ✔ `cached_property` cache mexanizmini
- ✔ Django ORM descriptorlarini