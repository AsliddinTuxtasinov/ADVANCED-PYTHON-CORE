## 🧠 1. Metaclass nima?

Oddiy qilib aytganda:

- Class — obyekt.
- Metaclass — classni yaratuvchi obyekt.

Yani:

> object → instance yaratadi
>
> metaclass → class yaratadi

Python’dagi har bir classni `type` metaclass yaratadi.

## 🔥 2. type — Python’ning default metaclassi

```python
class A:
    pass
```

Bu aslida shunga teng:

```python
A = type("A", (), {})
```

`type()` 3 ta argument oladi:

`type(name, bases, attrs)`

Masalan:

```python
User = type(
    "User",
    (object,),
    {"x": 10, "hello": lambda self: print("Hello")}
)
```

Bu bilan class runtime paytida yaratiladi.

## 🚀 3. Custom metaclass yozish

Metaclass — bu `type`dan meros olgan class:

```python
class MyMeta(type):
    def __new__(mcls, name, bases, attrs):
        print("Class yaratilyapti:", name)
        return super().__new__(mcls, name, bases, attrs)
```

Uni ishlatish:

```python
class User(metaclass=MyMeta):
    pass
```

Chiqarish:

```
Class yaratilyapti: User
```

**MUHIM**

Metaclass ishlashi class e’lon qilinganda, runtime emas.
Shuning uchun ORMlar uchun ideal.

## 🧩 4. Metaclass yordamida classga avtmatik xususiyat qo‘shish

ADVANCED pattern:

```python
class AutoAttr(type):
    def __new__(mcls, name, bases, attrs):
        attrs['id'] = 0
        return super().__new__(mcls, name, bases, attrs)

class User(metaclass=AutoAttr):
    pass

print(User.id)  # 0
```

Class yaratish jarayonida avtomatik field qo‘shildi.

## 🔥 5. Django Model Metaclass qanday ishlaydi?

Django’da barcha model klasslar `ModelBase` metaclass bilan yaratiladi.

Simplified:

```python
class ModelBase(type):
    def __new__(mcls, name, bases, attrs):
        # 1) Fieldlarni yig‘ish
        fields = {}
        for key, value in attrs.items():
            if isinstance(value, Field):
                fields[key] = value
        
        # 2) _meta object yaratish
        attrs["_meta"] = Options(fields)
        
        # 3) Model klass yaratish
        cls = super().__new__(mcls, name, bases, attrs)

        return cls
```

Nima bo‘ladi?

1. Django klassni ko‘rgan zahoti
   `CharField`, `IntegerField`, `ForeignKey` larni descriptor sifatida yig‘adi

2. Ulardan MetaData obyekt beradi
   (`_meta`)

3. Model klassni yaratadi
   - ✔ relationlar
   - ✔ field nomlari
   - ✔ primary key
   - ✔ constraints
   - ✔ signals
   - ✔ managers

HAMMASI metaclass orqali tashkil etiladi.

## 🔥 6. Django’da model yaratish jarayoni

```python
class User(models.Model):
    name = models.CharField(...)
    age = models.IntegerField(...)
```

Django metaclassi:

- `name` → `CharField`
- `age` → `IntegerField`

Fieldlarni yig‘adi

`User._meta.fields` ga yozadi

Descriptorlarni classga o‘rnatadi

Manager (`objects`) ni classga qo‘shadi

Shuning uchun:

```python
User.objects.all()
```

metaclass yordamida ishlaydi.

## 🚀 7. Pydantic V2 qanday metaclass ishlatadi?

Pydantic V2 — `ModelMetaclass` ishlatadi (typer + dataclass + schema generation uchun).

Simplified:

```python
class ModelMetaclass(type):
    def __new__(mcls, name, bases, attrs):
        # 1) Annotationsni yig‘adi
        annotations = attrs.get('__annotations__', {})

        # 2) validation rules yaratadi
        schema = create_schema(name, annotations)

        # 3) new classni yaratadi
        cls = super().__new__(mcls, name, bases, attrs)

        # 4) schema classga biriktiriladi
        cls.__schema__ = schema
        return cls
```

Pydantic:

- typelarni analiz qiladi
- validatorlarni yozadi
- field order’ini belgilaydi
- json schema yaratadi
- performance uchun transformatsiya qiladi

HAMMASI metaclass orqali.

## 🧠 8. FastAPI qanday foydalanadi?

FastAPI Pydantic Modellarni ishlatadi →
Pydantic esa metaclass orqali modelni yaratadi.

Shuning uchun:

```python
class User(BaseModel):
    id: int
    name: str
```

shaklan oddiy ko‘rinsa ham:

- runtime validator
- schema
- type casting
- OpenAPI generation
- field mapping

HAMMASI metaclass orqali ishlaydi.

## 🧩 9. Real ADVANCED misol — Interface Checker metaclass

```python
class InterfaceMeta(type):
    def __new__(mcls, name, bases, attrs):
        if name != "Service":
            if "run" not in attrs:
                raise TypeError("Service klassida run() bo‘lishi shart")
        return super().__new__(mcls, name, bases, attrs)

class Service(metaclass=InterfaceMeta):
    pass

class MyService(Service):
    def run(self):
        print("running")
```

Agar `run()` bo‘lmasa:

```
TypeError: Service klassida run() bo‘lishi shart
```

Bu compile-time validation — ADVANCED pattern.

## 🚀 10. CLI, ORM, Microservices uchun metaclass ishlatish (Real Life)

Metaclass bilan:

- Automatic Register (plugin architecture)
- ORM field yig‘ish
- API endpoint avtoregistratsiya
- Dependency injection
- Auto-validator
- Permissions mapping
- Singleton yaratish

Hammasi QIYIN emas.
Metaclass = “class-level middleware”.

## 🎯 Yakun — ADVANCED darajadagi Metaclassni bilish

Endi sen metaclass orqali:

- ✔ Class yaratishni boshqarishni
- ✔ Fieldlarni avtomatik yig‘ishni
- ✔ Django ORM qanday ishlashini
- ✔ Pydantic Model generation mexanizmini
- ✔ Custom compile-time validation
- ✔ Advanced inheritance control

bilar ekansan.

Bu Pythonning eng yuqori OOP layeri hisoblanadi.