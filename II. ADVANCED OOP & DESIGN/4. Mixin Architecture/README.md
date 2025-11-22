## 🧠 1. MIXIN nima? (ADVANCED darajada)

Mixin — bu to‘liq klass emas, balki boshqa klasslarga qo‘shimcha funksionallik qo‘shadigan klass.

- ✔ O‘z holida ishlamaydi
- ✔ Instantiatsiya qilinmaydi
- ✔ Faqat methodlar beradi

Yuqori darajadagi ta’rif:

> Mixin — multiple inheritance’ni xavfsiz va izchil qorish usuli.

## 🔥 2. Oddiy misol

```python
class LoggerMixin:
    def log(self, msg):
        print(f"[LOG]: {msg}")

class Service(LoggerMixin):
    def run(self):
        self.log("running service")
```

Mixinning vazifasi — Service ga log funksiyasi qo‘shish.

## 🚀 3. Qoidalar (ADVANCED best practices)

✔ **Mixin instantiatsiya qilinmaydigan class bo‘lishi kerak**

Hech qachon:

```python
l = LoggerMixin()  # yomon
```

✔ **Mixin konstruktorga ega bo‘lmasligi kerak (`__init__` yozilmaydi)**

Agar kerak bo‘lsa → super-safe variant ishlatiladi.

✔ **Mixin faqat functionality qo‘shishi kerak**

Model emas, service emas.

✔ **Mixin BOSHQA mixinlarga bog‘liq bo‘lmasligi kerak**

Yuqori coupling → arxitektura buziladi.

## 🧩 4. Django’da mixinlar juda ko‘p ishlatiladi

Masalan:

- `LoginRequiredMixin`
- `PermissionRequiredMixin`
- `FormMixin`
- `ContextMixin`

Django CBV’lar (Class-Based Views) aynan mixin architecture bilan qurilgan.

## 🔥 5. Real backend misol — “Auditable” Mixin

```python
class AuditMixin:
    created_by = None
    updated_by = None

    def set_created(self, user):
        self.created_by = user

    def set_updated(self, user):
        self.updated_by = user
```

Bu mixinni har qanday modelga qo‘shish mumkin:

```python
class Order(AuditMixin):
    pass

order = Order()
order.set_created("asliddin")
```

## 🚀 6. FastAPI misol — Service mixin

```python
class DBTransactionMixin:
    def commit(self):
        self.db.commit()

    def rollback(self):
        self.db.rollback()
```

Yangi service:

```python
class UserService(DBTransactionMixin):
    def __init__(self, db):
        self.db = db

    def create(self, data):
        self.db.add(data)
        self.commit()
```

## 🧨 7. Multiple Inheritance xavfsiz pattern (ADVANCED daraja)

Python MRO (Method Resolution Order) ishlaydi:

> Child → Mixins → Parent → object

XAVFSIZ tartib:

```python
class Child(Mixin1, Mixin2, Base):
    ...
```

XAVFLI tartib:

```python
class Child(Base, Mixin1):   # noto‘g‘ri
    ...
```

Base class oldin bo‘lmasligi kerak, chunki mixinlar o‘z methodlarini override qila olmay qoladi.

## 🧠 8. MRO (Method Resolution Order) ADVANCED tushuntirish

```python
class A: pass
class B: pass
class C(A, B): pass

print(C.mro())
```

Result:

```
[C, A, B, object]
```

Python shu tartibda method qidiradi.

Mixin architecture → shu MRO’ga 100% bog‘liq.

## 🔥 9. Mixin ichida __init__ yozish BO‘LSA

Asosan tavsiya etilmaydi.

Ammo agar yozish kerak bo‘lsa → `super()` ishlatish majburiy:

```python
class MyMixin:
    def __init__(self, *args, **kwargs):
        self.flag = True
        super().__init__(*args, **kwargs)
```

Bu mixin hamma parent classlarni buzmasdan ishlaydi.

Bu patternni Django CBV'lari ishlatadi.

## 🧨 10. Antipattern: “Fat Mixin”

Mixingda quyidagilar bo‘lmasligi kerak:

- Database connection
- Big logic (services)
- Request/response handling
- Global state

Bu mixin emas — bu “God object”.

## 🚀 11. Real ADVANCED Example — Validation Mixin

Validator mixin:

```python
class ValidateMixin:
    def validate_not_empty(self, fieldname, value):
        if not value:
            raise ValueError(f"{fieldname} bo'sh bo'lmasligi kerak")
```

User service:

```python
class UserService(ValidateMixin):
    def create_user(self, data):
        self.validate_not_empty("username", data.get("username"))
        ...
```

Kodni DRY qiladi — million marta foydali.

## 🌟 12. Mixin Pattern Super-Safe Implementation (ADVANCED)

Python’da behavior injection qilishning eng toza yo‘li:

```python
class Base:
    def save(self):
        print("saving from Base")

class TimestampMixin:
    def save(self):
        print("timestamp set")
        super().save()   # ketma-ket mixin behaviour
```

Child class:

```python
class User(TimestampMixin, Base):
    pass

User().save()
```

Result:

```
timestamp set
saving from Base
```

Bu ADVANCED professional pattern.

## 🧠 13. Django Model + Mixin pattern

Django ORM modeliga mixin qo‘shish:

```python
class SoftDeleteMixin:
    is_deleted = models.BooleanField(default=False)

    def delete(self, using=None, keep_parents=False):
        self.is_deleted = True
        self.save()
```

Model:

```python
class Product(SoftDeleteMixin, models.Model):
    name = models.CharField(max_length=255)
```

Soft delete → mixin bilan tugadi.

## 🎯 14. FINAL ADVANCED MIXIN PRINCIPLES (MUHIM)

| Qoidalar | Sabab |
| :--- | :--- |
| Mixin hech qachon mustaqil class bo‘lmasligi kerak | couplingdan qochish |
| `__init__` ishlatilsa → `super()` majburiy | MRO |
| Mixinga faqat behavior yoziladi | Pure functionality |
| Atributlar kam bo‘lishi kerak | Memory footprint |
| Tartib: `MyClass(Mixin1, Mixin2, Base)` | MRO safety |

## 🔥 6. MULTIPLE INHERITANCE — SAFE PATTERNS & MRO (ADVANCED LEVEL)

Bu mavzu Python’dagi eng muhim advanced konseptlardan biri.
Django CBV (Class-Based Views), DRF GenericAPIView, Mixins, hatta Python standarti — HAMMASI shu mexanizmga tayanadi.

Multiple inheritance to‘g‘ri ishlatilmasa → katta bug lar, infinite looplar, noto‘g‘ri `super()` ishlashi, method override muammolari paydo bo‘ladi.

KETDI! 🔥🔥🔥

## 🧠 1. Python MRO nima?

MRO — Method Resolution Order
Ya’ni Python methodni qaysi tartibda qidiradi.

`MyClass.mro()` orqali ko‘ramiz:

```python
print(MyClass.mro())
```

MRO algoritmi — **C3 Linearization** deb ataladi.
Bu ADVANCED-lar bilishi shart bo‘lgan mavzu.

## 🧩 2. Oddiy inheritance MRO

```python
class A: pass
class B: pass
class C(A, B): pass

print(C.mro())
```

Natija:

```
[C, A, B, object]
```

Python methodni shu tartibda axtaradi.

## 🚀 3. Multiple inheritance xavfi

Mashhur Diamond Inheritance Problem:

```python
class A:
    def hello(self): print("A")

class B(A):
    def hello(self): print("B")

class C(A):
    def hello(self): print("C")

class D(B, C):
    pass

D().hello()
```

Natija:

```
B
```

Chunki MRO:

> D → B → C → A

A AYLANIB KETMASDAN ishlaydi — C3 algorithm buni hal qiladi.

## 🏆 4. ADVANCED QOIDALAR: MRO safe bo‘lishi uchun tartib

Har doim mixinlar chap tomonda, asosiy base class o‘ng tomonda bo‘lishi kerak

```python
class MyClass(Mixin1, Mixin2, BaseClass):
    ...
```

Noto‘g‘ri:

```python
class MyClass(BaseClass, Mixin1):   # ❌
```

Bu MROni buzadi → mixin methodlari override bo‘lmaydi.

## 🔥 5. super() ADVANCED darajada tushuntirish

Ko‘pchilik `super()`ni noto‘g‘ri tushunadi.

❗ `super()` — bu “parent class” emas.

> `super()` → MRO bo‘yicha keyingi class.

Misol:

```python
class A:
    def run(self):
        print("A")

class B(A):
    def run(self):
        print("B")
        super().run()

class C(A):
    def run(self):
        print("C")
        super().run()

class D(B, C):
    pass

D().run()
```

Natija:

```
B
C
A
```

NIMA UCHUN?

MRO:

> D → B → C → A → object

Demak `super()`:

1. B.da → C.ga o‘tadi
2. C.da → A.ga o‘tadi
3. A.da → to‘xtaydi

Bu ADVANCED multiple inheritance mexanizmining yuragi!

## 🎯 6. Cooperative Multiple Inheritance Pattern (Eng Muhimi!)

Cooperative inheritance — hamma parentlar ketma-ket ishlashi uchun kerak.

Har bir klass:

- ✔ `super()` chaqirishi majburiy
- ✔ imzo (signature) mos bo‘lishi kerak
- ✔ argumentlar `**kwargs` bilan uzatilishi kerak

To‘g‘ri pattern:

```python
class A:
    def process(self, *args, **kwargs):
        print("A start")
        super().process(*args, **kwargs)
        print("A end")

class B:
    def process(self, *args, **kwargs):
        print("B start")
        super().process(*args, **kwargs)
        print("B end")

class C(A, B):
    def process(self, *args, **kwargs):
        print("C start")
        super().process(*args, **kwargs)
        print("C end")
```

Run:

```python
C().process()
```

Natija:

```
C start
A start
B start
object.process  # default
B end
A end
C end
```

Bu pattern:

- ✔ Django CBV’larda
- ✔ DRF GenericAPIView’da
- ✔ Mixins’da
- ✔ ORM Fieldlarda

hech qachon buzilmasligi uchun zarur.

## 🔥 7. Multiple inheritance yozishda ADVANCED QOIDALAR

1. **Mixinlarni har doim chapga qo‘y**
   ```python
   class View(LoginRequiredMixin, PermissionMixin, BaseView)
   ```

2. **Har bir class `super()` chaqirsin**
   ```python
   def dispatch(self, *args, **kw):
       super().dispatch(*args, **kw)
   ```

3. **Constructor ham `super()` bilan bo‘lsin (agar bor bo‘lsa)**
   ```python
   def __init__(self, *args, **kwargs):
       self.flag = True
       super().__init__(*args, **kwargs)
   ```

4. **Yagona massive parent emas — ko‘p mixin ishlatish tavsiya**

   Modular architecture.

## 🚨 8. Qanday muammolarni oldini oladi?

- ✔ MRO conflict
- ✔ Method override muammolari
- ✔ `super()` o‘tmasligi
- ✔ Mixinning ishlamasligi
- ✔ “base class ikki marta ishlashi” bug’i
- ✔ Django CBV'da “dispatch working twice” bug’i

## 🧠 9. ADVANCED real masala — MRO debugging

```python
print(MyClass.mro())
```

Yoki:

```python
import inspect
inspect.getmro(MyClass)
```

Productionda classlar chigal bo‘lsa → shu orqali tartibni tekshirasan.

## 🏆 10. ADVANCED Real Life Example — Django CBV

Django CBV MRO shunday:

```
YourView
  ↓
Mixin1
  ↓
Mixin2
  ↓
GenericAPIView
  ↓
APIView
  ↓
View
  ↓
object
```

Har bir metodda:

```python
def get(self, request, *args, **kwargs):
    return super().get(request, *args, **kwargs)
```

Shunday qilib:

1. Auth check
2. Permission check
3. Throttling
4. Caching
5. Rendering

hammasi birma-bir ishlaydi.

Bu cooperative inheritance.

## 🚀 Yakun: Endi sen multiple inheritance va MRO ni ADVANCED darajada tushunding.

Bu mavzular Python Core’ning eng advanced qismi.

