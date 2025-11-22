## 🧠 1. Abstract Base Class (ABC) nima?

ABC — bu to'liq class emas, balki:

- umumiy interface belgilaydi
- o'zidan bevosita obyekt yaratilmaydi
- child classlar metodlarni majburiy implement qiladi

Python'da ABC — Java/C# dagi interface'ning analogidir.

## 🔥 2. ABC yaratish

```python
from abc import ABC, abstractmethod

class Storage(ABC):

    @abstractmethod
    def save(self, data):
        pass

    @abstractmethod
    def load(self, key):
        pass
```

Muqarrar qoidalar:

- ✔ ABC'dan obyekt yaratilmaydi
- ❌ `s = Storage()` → ERROR
- ✔ Abstract metodlar override qilinmasa → ERROR

## 🚀 3. Real implementatsiya

```python
class FileStorage(Storage):
    def save(self, data):
        print("Saving to file:", data)

    def load(self, key):
        print("Loading from file")
```

Endi ishlaydi:

```python
s = FileStorage()
s.save("hello")
```

## 🔥 4. ABC — backend arxitekturasida interfeysni majburiy qiladi

Masalan FastAPI loyihangda:

```python
class UserRepository(ABC):

    @abstractmethod
    def get(self, user_id: int):
        ...

    @abstractmethod
    def create(self, data):
        ...
```

Har qanday repo yozuvchisi shart:

```python
class SQLUserRepository(UserRepository):
    ...
```

Aks holda:

```
TypeError: Can't instantiate abstract class...
```

Bu architecture safetyni ta'minlaydi.

## 🧠 5. ABC da property, classmethod, staticmethod ham abstract bo'ladi

Abstract property:

```python
class Service(ABC):
    @property
    @abstractmethod
    def name(self):
        ...
```

Abstract classmethod:

```python
class BaseWorker(ABC):
    @classmethod
    @abstractmethod
    def build(cls):
        ...
```

## 🔥 6. Python'da INTERFACE yaratishning to'g'ri yo'li

Python'da interface — faqat abstract methodlardan iborat ABC:

```python
class PaymentInterface(ABC):

    @abstractmethod
    def pay(self, amount):
        ...
```

interface ga xos:

- ❌ State (atributlar) bo'lmasligi kerak
- ❌ Real implementatsiya bo'lmasligi kerak
- ✔ Faqat methodlar

Java dagi interface → Python'dagi ABC + @abstractmethod.

## 🚀 7. Interface Segregation Principle (ISP) — Python uslubida

ISP — class faqat o'zi ehtiyoj sezgan interface'ni implement qilishi kerak.

Yomon misol (ISP buzilgan):

```python
class Worker(ABC):
    @abstractmethod
    def clean(self): ...
    @abstractmethod
    def cook(self): ...
    @abstractmethod
    def code(self): ...
```

Barcha workerlar bu methodlarni implement qilishga majbur — bu noto'g'ri.

To'g'ri Pythonic yechim — INTERFACE'larni bo'lish

```python
class Cleaner(ABC):
    @abstractmethod
    def clean(self): ...

class Cook(ABC):
    @abstractmethod
    def cook(self): ...

class Programmer(ABC):
    @abstractmethod
    def code(self): ...
```

Endi har kimga kerak bo'lgani bilan ishlaydi:

```python
class Chef(Cook):
    def cook(self): ...

class Janitor(Cleaner):
    def clean(self): ...

class Developer(Programmer):
    def code(self): ...
```

Mana bu Pythonic ISP.

## 🔥 8. Real ADVANCED BACKEND MISOLLAR

### 8.1. Storage Interface

```python
class Storage(ABC):
    @abstractmethod
    def save(self, key, value): ...
```

S3:

```python
class S3Storage(Storage):
    ...
```

Local:

```python
class LocalStorage(Storage):
    ...
```

FastAPI service:

```python
class FileService:
    def __init__(self, storage: Storage):
        self.storage = storage
```

Service hech qachon storage turiga bog'lanmaydi.

### 8.2. Payment Interface (real fintech)

```python
class PaymentGateway(ABC):

    @abstractmethod
    def charge(self, amount): ...
```

Click:

```python
class ClickGateway(PaymentGateway):
    ...
```

Payme:

```python
class PaymeGateway(PaymentGateway):
    ...
```

PaymentService:

```python
class PaymentService:
    def __init__(self, gateway: PaymentGateway):
        self.gateway = gateway
```

Polymorphism → paymentlarni kengaytirish oson.

### 8.3. Repository pattern

```python
class UserRepo(ABC):
    @abstractmethod
    def get_by_id(self, user_id): ...
```

SQLAlchemy repo:

```python
class SQLUserRepo(UserRepo):
    ...
```

Mongo repo:

```python
class MongoUserRepo(UserRepo):
    ...
```

Ularning ikkalasi ham servicega mos keladi.

## 🎯 9. ABC orqali runtime contract enforcement

Misol:

```python
def run_service(svc: Service):
    assert isinstance(svc, Service)
```

Yoki Pythonic:

```python
isinstance(obj, ABCClass)
```

Bu ADVANCED arxitektura nazorati.

## 🧠 10. ABC vs Mixin (farqi)

| ABC | Mixin |
| :--- | :--- |
| Interface belgilaydi | Ҳarakat qo'shadi |
| methodlar abstract | methodlar real |
| class yaratishda majburiy | optional |
| contract | behavior |

Django'da mixin + ABC birga ishlatiladi — lekin rollari farq qiladi.

## 🔥 Yakun

Biz ABC bo'limini ADVANCED darajada tugatdik:

- ✔ ABC
- ✔ abstractmethod
- ✔ abstract property/classmethod
- ✔ Pythonic interface
- ✔ Interface Segregation Principle
- ✔ Backend architecture misollari