# 22. Clean Architecture in Python (Advanced Deep Dive)

## 0️⃣ Clean Architecture — bu nima EMAS

### ❌ Bu:
- faqat papkalar nomi emas
- framework pattern emas
- “ortiqcha abstraksiya” emas

### ✅ Bu:
- dependency control
- business logic mustaqilligi
- testability
- frameworkga qaram bo‘lmaslik

> **🔑 Frameworklar keladi-ketadi, biznes qoladi.**

---

## 1️⃣ Dependency Rule — eng muhim qonun

- **Outer layer → Inner layer**
- **Inner layer ← Outer layer** (taqiqlangan)

> **📌 Ichki layerlar tashqi layerlarni bilmaydi.**

**Masalan:**
- Entity → FastAPI’ni bilmaydi
- Use Case → SQLAlchemy’ni bilmaydi
- Business → HTTP’ni bilmaydi

---

## 2️⃣ Entities — Core Business Rules

### 🎯 Vazifasi
- Biznes qoidalar
- Eng barqaror qism
- Eng kam o‘zgaradi

### Entity xususiyatlari:
- Frameworksiz
- DB’siz
- JSON’siz
- HTTP’siz

```python
@dataclass
class User:
    id: int
    email: str
    is_active: bool = True

    def deactivate(self):
        if not self.is_active:
            raise ValueError("User already inactive")
        self.is_active = False
```

> **📌 Advanced Notes:**
> - Entity — bu anemic model emas
> - Behavior entity ichida bo‘lishi kerak
> - Validation — entity darajasida bo‘lsa eng to‘g‘ri

---

## 3️⃣ Use Cases (Application Layer)

### 🎯 Vazifasi
- Biznes flow’ni boshqarish
- Entity’larni orchestratsiya qilish
- External dependency yo‘q

### Use Case:
- “User yaratish”
- “Buyurtmani tasdiqlash”
- “To‘lovni tekshirish”

### Use Case misoli
```python
class CreateUserUseCase:
    def __init__(self, repo):
        self.repo = repo

    def execute(self, email: str) -> User:
        if self.repo.exists(email):
            raise ValueError("Email exists")

        user = User(id=0, email=email)
        return self.repo.save(user)
```

> **📌 Advanced Notes:**
> - Use Case — bu transaction boundary
> - FastAPI endpoint = use case chaqiruvchi
> - Use Case return qiladi, HTTP response emas

---

## 4️⃣ Interfaces (Ports) — Boundary’lar

### 🎯 Maqsad
- Infrastructure bilan bog‘lanish
- Dependency inversion

```python
from abc import ABC, abstractmethod

class UserRepository(ABC):
    @abstractmethod
    def save(self, user: User) -> User:
        ...

    @abstractmethod
    def exists(self, email: str) -> bool:
        ...
```

> **📌 Muhim:**
> - Interface → inner layer
> - Implementation → outer layer
> - Interface’ni biznes yozadi, infra implement qiladi

---

## 5️⃣ Infrastructure Layer — Detallar

### 🎯 Vazifasi
- DB
- HTTP
- Redis
- Email
- Framework integration

```python
class SqlAlchemyUserRepository(UserRepository):
    def __init__(self, session):
        self.session = session

    def save(self, user: User) -> User:
        self.session.add(user)
        self.session.commit()
        return user
```

> **📌 Advanced Notes:**
> - Infrastructure — almashtiriladigan qism
> - Bugun PostgreSQL, ertaga Mongo → biznes o‘zgarmaydi
> - Bu layer testda fake bilan almashtiriladi

---

## 6️⃣ Controller / Delivery Layer (FastAPI misolida)

```python
@app.post("/users")
def create_user(
    data: UserCreateSchema,
    repo: UserRepository = Depends(get_user_repo),
):
    uc = CreateUserUseCase(repo)
    user = uc.execute(data.email)
    return {"id": user.id}
```

> **📌 Controller:**
> - HTTP ↔ Use Case adapter
> - Business logic YO‘Q
> - Faqat translate qiladi

---

## 7️⃣ Clean Architecture + Testing (Advanced)

### Test mapping

| Layer | Test turi |
| :--- | :--- |
| **Entity** | Unit |
| **Use Case** | Unit + Fake |
| **Infrastructure** | Integration |
| **API** | Integration / Contract |

### Use Case test (fake repo bilan)

```python
class FakeRepo(UserRepository):
    def __init__(self):
        self.users = []

    def exists(self, email):
        return any(u.email == email for u in self.users)

    def save(self, user):
        self.users.append(user)
        return user
```

> **📌 Advanced insight:**
> - Mock emas, Fake
> - Real behavior
> - Testlar yolg‘on bo‘lmaydi

---

## 8️⃣ Clean Architecture — real project structure

```
app/
├── domain/
│   ├── entities/
│   └── interfaces/
├── application/
│   └── use_cases/
├── infrastructure/
│   ├── db/
│   └── repositories/
├── delivery/
│   └── api/
```

> **📌 Bu qoida emas, yo‘l-yo‘riq.**

---

## 9️⃣ Common mistakes (Advanced darajada)

- ❌ Entity ichida ORM
- ❌ Use Case ichida Depends
- ❌ FastAPI response entity qaytarish
- ❌ Repository’da business logic
- ❌ Interface’ni infra layerda yozish

---

## 🔟 Yakuniy xulosa

- **Clean Architecture** = dependency nazorati
- **Entity** = biznes qoidalari
- **Use Case** = flow
- **Interface** = boundary
- **Infrastructure** = detal
- **Framework** = tashqi qobiq

---
---

# FastAPI + Clean Architecture — REAL TEMPLATE (Advanced)

## 0️⃣ Target mindset (1 daqiqa)

- **FastAPI** = Delivery layer
- **Business** = frameworkdan mustaqil
- **DI** = boundary
- **Repo/DB** = detail
- **Test** = design validator

---

## 1️⃣ Project Structure (battle-tested)

```
app/
├── domain/
│   ├── entities/
│   │   └── user.py
│   └── interfaces/
│       └── user_repository.py
│
├── application/
│   └── use_cases/
│       └── create_user.py
│
├── infrastructure/
│   ├── db/
│   │   └── session.py
│   └── repositories/
│       └── user_sqlalchemy.py
│
├── delivery/
│   └── api/
│       ├── deps.py
│       └── users.py
│
├── main.py
└── tests/
```

> **📌 Note:**
> Bu “papka fetish” emas. Maqsad — dependency yo‘nalishini majburlash.

---

## 2️⃣ Domain — Entity (pure Python)

`# domain/entities/user.py`
```python
from dataclasses import dataclass

@dataclass
class User:
    id: int | None
    email: str
    is_active: bool = True

    def deactivate(self):
        if not self.is_active:
            raise ValueError("Already inactive")
        self.is_active = False
```

> **🔎 Advanced notes**
> - Entity = behavior + state
> - Validation va invariantlar shu yerda
> - ORM, Pydantic, FastAPI — yo‘q

---

## 3️⃣ Domain — Interface (Port)

`# domain/interfaces/user_repository.py`
```python
from abc import ABC, abstractmethod
from domain.entities.user import User

class UserRepository(ABC):
    @abstractmethod
    def exists(self, email: str) -> bool: ...

    @abstractmethod
    def save(self, user: User) -> User: ...
```

> **🔎 Advanced notes**
> - Interface biznes tomonidan yoziladi
> - Infra unga bo‘ysunadi
> - Bu — Dependency Inversionning markazi

---

## 4️⃣ Application — Use Case (transaction boundary)

`# application/use_cases/create_user.py`
```python
from domain.entities.user import User
from domain.interfaces.user_repository import UserRepository

class CreateUserUseCase:
    def __init__(self, repo: UserRepository):
        self.repo = repo

    def execute(self, email: str) -> User:
        if self.repo.exists(email):
            raise ValueError("Email already exists")

        user = User(id=None, email=email)
        return self.repo.save(user)
```

> **🔎 Advanced notes**
> - Use Case = 1 business action
> - HTTP, DB, JSON bilmaydi
> - Return = entity / result, response emas

---

## 5️⃣ Infrastructure — SQLAlchemy implementation (detail)

`# infrastructure/repositories/user_sqlalchemy.py`
```python
from domain.interfaces.user_repository import UserRepository
from domain.entities.user import User

class SqlAlchemyUserRepository(UserRepository):
    def __init__(self, session):
        self.session = session

    def exists(self, email: str) -> bool:
        return self.session.query(User).filter_by(email=email).first() is not None

    def save(self, user: User) -> User:
        self.session.add(user)
        self.session.commit()
        return user
```

> **⚠️ Advanced warning**
> - ORM model ≠ Domain Entity (ideal holatda)
> - Katta loyihada mapping layer bo‘ladi

---

## 6️⃣ Delivery — FastAPI (adapter)

**Dependency provider**
`# delivery/api/deps.py`
```python
from infrastructure.repositories.user_sqlalchemy import SqlAlchemyUserRepository
from infrastructure.db.session import get_session

def get_user_repo():
    session = get_session()
    return SqlAlchemyUserRepository(session)
```

**Endpoint**
`# delivery/api/users.py`
```python
from fastapi import APIRouter, Depends
from application.use_cases.create_user import CreateUserUseCase
from delivery.api.deps import get_user_repo

router = APIRouter()

@router.post("/users")
def create_user(email: str, repo=Depends(get_user_repo)):
    uc = CreateUserUseCase(repo)
    user = uc.execute(email)
    return {"id": user.id, "email": user.email}
```

> **🔎 Advanced notes**
> - Controller = translator
> - Logic yo‘q
> - DI = boundary switch

---

## 7️⃣ main.py (composition root)

```python
from fastapi import FastAPI
from delivery.api.users import router as user_router

app = FastAPI()
app.include_router(user_router)
```

> **📌 Composition Root — barcha bog‘lashlar shu yerda.**

---

## 8️⃣ Testing — Clean Architecture style

**Use Case unit test (Fake repo)**
```python
class FakeUserRepo:
    def __init__(self):
        self.users = []

    def exists(self, email):
        return any(u.email == email for u in self.users)

    def save(self, user):
        user.id = len(self.users) + 1
        self.users.append(user)
        return user

def test_create_user():
    repo = FakeUserRepo()
    uc = CreateUserUseCase(repo)

    user = uc.execute("a@b.com")

    assert user.id == 1
```

> **🔎 Advanced insight**
> - Fake > Mock (business testlarda)
> - Testlar frameworksiz

---

## 9️⃣ FastAPI Dependency Override (integration test)

```python
def override_repo():
    return FakeUserRepo()

app.dependency_overrides[get_user_repo] = override_repo
```

> **📌 Agar override qiyin bo‘lsa → architecture yomon.**

---

## 🔟 Django bilan farqi (qisqa)

- **Django’da** ORM juda markazda
- **Clean Architecture’da** ORM chetda
- Django service layer bilan ishlaydi
- FastAPI — ideal fit

---

## 1️⃣1️⃣ Common Advanced mistakes

- ❌ Use case ichida Depends
- ❌ Entity ichida Pydantic
- ❌ Repo ichida business logic
- ❌ Controller’da validation logic
- ❌ Frameworkni “core”ga olib kirish

---

## 1️⃣2️⃣ Yakuniy xulosa

- **Clean Architecture** = nazorat
- **FastAPI** = faqat delivery
- **Use Case** = haqiqiy business
- **Interface** = mustahkam chegara
- **Infra** = almashtiriladigan detal
- **Test** = sifat kafolati

---
---

# Django + ORM izolyatsiyasi · CQRS · DDD · Real Production Template

## 1️⃣ Django + Clean Architecture

### ORM’ni izolyatsiya qilish (eng qiyin, eng muhim qism)

**Muammo (real hayotdan)**
Django’da:
- Model = ORM + business logic + validation
- View = HTTP + business logic
- **Hammasi bir joyda**

👉 **Natija:**
- Test yozish qiyin
- ORM o‘zgarsa — hammasi buziladi
- Business logic frameworkga qattiq bog‘lanadi

**🎯 Maqsad**
- Django ORM → **Infrastructure**
- Business logic → **Frameworkdan mustaqil**
- Model → faqat **persistence detail**

### Clean Architecture mapping (Django uchun)

| Clean Arch | Django |
| :--- | :--- |
| **Entity** | Pure Python class |
| **Use Case** | Service / Interactor |
| **Interface** | Repository (ABC) |
| **Infrastructure** | Django ORM |
| **Delivery** | Views / DRF |

**✅ To‘g‘ri struktura**
```
app/
├── domain/
│   ├── entities/
│   └── interfaces/
├── application/
│   └── use_cases/
├── infrastructure/
│   └── orm/
│       └── models.py
│       └── repositories.py
├── presentation/
│   └── views.py
```

### ORM izolyatsiyasi (eng muhim fikr)

#### ❌ Noto‘g‘ri
```python
class Order(models.Model):
    def pay(self):
        if self.status != "new":
            raise Exception()
```

#### ✅ To‘g‘ri
```python
# domain/entity
class Order:
    def pay(self):
        ...

# infrastructure/orm
class OrderModel(models.Model):
    status = models.CharField(...)
```

> **📌 Advanced note**
> ORM — bu DB adapter, biznes emas.

---

## 2️⃣ CQRS + Clean Architecture
**(Command Query Responsibility Segregation)**

### CQRS nima?
- Write ≠ Read
- **Command** → state o‘zgartiradi
- **Query** → faqat o‘qiydi

### Nega CQRS kerak?
- Murakkab biznes
- Ko‘p o‘qish / kam yozish
- Performance muhim
- Audit / history kerak

### CQRS + Clean Architecture mapping

**COMMAND SIDE**
`Controller → Command → Use Case → Domain → Repo`

**QUERY SIDE**
`Controller → Query → Read Model → DB`

> **📌 Command va Query aralashmaydi.**

**Command misoli (business)**
```python
class PayOrderCommand:
    def __init__(self, repo):
        self.repo = repo

    def execute(self, order_id):
        order = self.repo.get(order_id)
        order.pay()
        self.repo.save(order)
```

**Query misoli (fast, optimized)**
```python
def get_orders_list():
    return OrderModel.objects.values("id", "status")
```

> **📌 Advanced note**
> - Query → ORM + raw SQL bo‘lishi mumkin
> - Query → domain’ga kirmaydi
> - Command → doim domain orqali

---

## 3️⃣ DDD + Clean Architecture
**(Domain Driven Design)**

### DDD — bu nima EMAS
- ❌ Faqat Entity yozish
- ❌ Juda murakkab diagrammalar

### DDD — bu:
- ✅ Biznes tilini kodga ko‘chirish

### Asosiy tushunchalar (short & sharp)

**Entity**
- Identity bor
- Hayoti davomida o‘zgaradi

**Value Object**
- Immutable
- Identity yo‘q
```python
class Money:
    def __init__(self, amount, currency):
        self.amount = amount
        self.currency = currency
```

**Aggregate Root**
- Bitta kirish nuqtasi
- Ichki invariantlarni himoya qiladi
```python
class Order:
    def add_item(self, item):
        ...
```

**Repository**
- Aggregate’ni saqlash uchun
- Collection sifatida

### DDD + Clean Architecture uyg‘unligi

| DDD | Clean Arch |
| :--- | :--- |
| **Entity** | Entity |
| **Aggregate** | Entity |
| **Repository** | Interface |
| **Domain Service** | Use Case |

> **📌 Clean Architecture — texnik boundary**
> **📌 DDD — biznes boundary**

---

## 4️⃣ Real Production Template
**Auth · Users · Payments (battle-tested)**

### Production-level structure
```
src/
├── domain/
│   ├── user/
│   ├── auth/
│   └── payment/
│
├── application/
│   ├── commands/
│   ├── queries/
│   └── services/
│
├── infrastructure/
│   ├── db/
│   ├── payment_gateways/
│   └── messaging/
│
├── delivery/
│   ├── api/
│   └── admin/
```

### Auth module (example)

**Domain**
- User
- PasswordHash (Value Object)

**Application**
- LoginUseCase
- RegisterUseCase

**Infrastructure**
- Django ORM UserModel
- JWT provider

**Delivery**
- API endpoints

### Payment module (example)

**Domain**
- Payment
- Money
- PaymentStatus

**Application**
- CreatePaymentCommand
- ConfirmPaymentCommand

**Infrastructure**
- Payme / Click / Stripe adapters

> **📌 Gateway pattern ishlatiladi.**

### Eng muhim production qoidalar
- ❗ Framework yo‘q = Core
- ❗ DB almashtirilsa → biznes qoladi
- ❗ Testlar fake bilan yoziladi
- ❗ Use Case = transaction boundary
- ❗ Query ≠ Command

---

## 5️⃣ Advanced-level anti-patternlar

- ❌ “Service” ichida ORM
- ❌ “Fat View / Fat Serializer”
- ❌ Business logic signal’da
- ❌ Model.save() override qilib biznes yozish
- ❌ CQRS’ni majburlab ishlatish

---

## 6️⃣ Yakuniy xulosa

- **Django** → mumkin, lekin ehtiyotkorlik bilan
- **Clean Architecture** → nazorat
- **CQRS** → murakkablik bo‘lsa
- **DDD** → biznes murakkab bo‘lsa
- **Production** → oddiy + qat’iy boundary

> **Agar bugun frameworkni olib tashlasang va biznes ishlashda davom etsa — sen architect darajadasan.**

---
---

# Legacy Django → Clean Architecture (Advanced, Step-by-Step)

Bu maqola real legacy Django loyihani bosqichma-bosqich Clean Architecture ga o‘tkazish uchun yozilgan.
Maqsad — ishlab turgan sistemani buzmasdan, katta rewrite qilmasdan, riskni minimallashtirib migratsiya qilish.

> **⚠️ Muhim:** Clean Architecture — rewrite emas, refactor strategiyasi.

## 0️⃣ Boshlashdan oldin: to‘g‘ri mindset

**❌ Noto‘g‘ri fikr:**
“Keling, hammasini qaytadan yozamiz”

**✅ To‘g‘ri fikr:**
“Biznesni ajratamiz, frameworkni chetga suramiz”

**Asosiy qoida**
- Support layer ishlayveradi.
- Legacy kod ishlashda davom etadi
- Yangi kod — Clean Architecture’da
- Eski kod asta-sekin o‘lik zonaga aylanadi

> **Bu yondashuv Strangler Pattern deyiladi.**

---

## 1️⃣ Bosqich: Diagnosis (eng muhim qadam)

**Savollar:**
- Business logic qayerda?
- Qaysi view/serializer eng murakkab?
- Qaysi model eng ko‘p o‘zgaradi?
- Qaysi joy test bilan yopilmagan?

**Amaliy qoidalar**
- ❌ Eng katta moduldan boshlama
- ❌ Eng kritik joydan boshlama
- ✅ O‘rtacha murakkab, tez-tez o‘zgaradigan joydan boshlagin

> **📌 Advanced note**
> Migratsiya — texnik emas, strategik qaror.

---

## 2️⃣ Bosqich: “Service Layer” ni ajratish (Bridge bosqichi)

Legacy Django’da odatda shunday holat bor:

```python
class OrderView(APIView):
    def post(self, request):
        order = Order.objects.create(...)
        if order.total > 1_000_000:
            send_email(...)
        order.status = "paid"
        order.save()
```
Bu yerda: HTTP, ORM, Business logic hammasi bitta joyda.

### ✅ 2.1 Birinchi refactor (hali Clean Arch emas)

`# services/order_service.py`
```python
def pay_order(order_id):
    order = Order.objects.get(id=order_id)
    if order.total > 1_000_000:
        send_email(...)
    order.status = "paid"
    order.save()

class OrderView(APIView):
    def post(self, request):
        pay_order(request.data["order_id"])
```

> **📌 Muhim**
> - Bu hali Clean Architecture emas
> - Lekin business logic ajraldi
> - Bu — oraliq bosqich

---

## 3️⃣ Bosqich: Domain Entity’ni ajratish

Endi eng muhim burilish nuqtasi.

**❌ Oldin**
```python
class Order(models.Model):
    def pay(self):
        if self.status != "new":
            raise Exception()
```

**✅ Keyin**
`# domain/entities/order.py`
```python
class Order:
    def __init__(self, id, status, total):
        self.id = id
        self.status = status
        self.total = total

    def pay(self):
        if self.status != "new":
            raise ValueError("Invalid state")
        self.status = "paid"
```

> **📌 Advanced insight**
> - ORM = persistence detail
> - Entity = business truth

---

## 4️⃣ Bosqich: Repository Interface (Dependency inversion)

### 4.1 Interface (domain)
`# domain/interfaces/order_repository.py`
```python
from abc import ABC, abstractmethod

class OrderRepository(ABC):
    @abstractmethod
    def get(self, order_id): ...
    @abstractmethod
    def save(self, order): ...
```

### 4.2 Django ORM implementation (infrastructure)
`# infrastructure/repositories/order_django.py`
```python
class DjangoOrderRepository(OrderRepository):
    def get(self, order_id):
        model = OrderModel.objects.get(id=order_id)
        return Order(
            id=model.id,
            status=model.status,
            total=model.total,
        )

    def save(self, order):
        OrderModel.objects.filter(id=order.id).update(
            status=order.status
        )
```

> **📌 Advanced note**
> - Mapping bor (Entity ↔ ORM)
> - Bu og‘irdek tuyuladi → lekin ozodlik beradi

---

## 5️⃣ Bosqich: Use Case (Application layer)

`# application/use_cases/pay_order.py`
```python
class PayOrderUseCase:
    def __init__(self, repo):
        self.repo = repo

    def execute(self, order_id):
        order = self.repo.get(order_id)
        order.pay()
        self.repo.save(order)
```

> **📌 Endi:**
> - HTTP yo‘q
> - ORM yo‘q
> - Framework yo‘q
> - **Bu — Clean Core.**

---

## 6️⃣ Bosqich: Django View → Adapter

```python
class OrderView(APIView):
    def post(self, request):
        repo = DjangoOrderRepository()
        uc = PayOrderUseCase(repo)
        uc.execute(request.data["order_id"])
        return Response({"status": "ok"})
```

> **📌 Advanced insight**
> - View = adapter
> - Hech qanday biznes yo‘q

---

## 7️⃣ Bosqich: Testlar bilan mustahkamlash

**Use Case unit test (frameworksiz)**
```python
class FakeRepo:
    def __init__(self):
        self.saved = False

    def get(self, _):
        return Order(id=1, status="new", total=100)

    def save(self, order):
        self.saved = True

def test_pay_order():
    uc = PayOrderUseCase(FakeRepo())
    uc.execute(1)
```

> **📌 Muhim**
> Migratsiya davomida test — xavfsizlik kamaridir

---

## 8️⃣ Bosqich: Parallel yashash (Hybrid mode)

Bir muddat:
- Eski view’lar — legacy
- Yangi view’lar — clean
- Bir DB
- Bir deploy

**Bu NORMAL.**

> **📌 Hech qachon “hammasi birdan” qilma.**

---

## 9️⃣ Bosqich: Legacy code’ni o‘chirish

**Qachon o‘chiriladi?**
- Use case bor
- Test bor
- Production’da ishlayapti
- Hech kim o‘zgartirmayapti

**👉 Shunda:** Eski service/view/model logic o‘chadi.

---

## 🔟 Real migratsiya checklist (Advanced)

- [ ] Business logic view’dan chiqdi
- [ ] Entity ORM’dan ajratildi
- [ ] Repository interface mavjud
- [ ] Use case framework bilmaydi
- [ ] Fake bilan unit test bor
- [ ] View = adapter
- [ ] Legacy kod kamayib boryapti

---

## 1️⃣1️⃣ Eng ko‘p uchraydigan xatolar

- ❌ Birdan rewrite qilish
- ❌ Hammasini Clean Arch qilishga urinish
- ❌ Juda ko‘p abstraksiya
- ❌ Test yozmasdan refactor
- ❌ ORM’ni “yashirib qo‘ydim” deb o‘ylash

---

## 1️⃣2️⃣ Yakuniy xulosa

- **Migratsiya** — jarayon
- **Clean Architecture** — maqsad emas, vosita
- Eng muhim narsa — **biznesni himoyalash**
- Django bilan ham toza arxitektura mumkin
- **Advanced developer belgisi** — eski sistemani buzmasdan yaxshilay olish.

---
---

# Real Legacy Django Modulini Refactor Qilish
**(Step-by-step · Clean Architecture’ga o‘tish · Advanced)**

Quyida haqiqiy hayotda uchraydigan Django modulni bosqichma-bosqich refactor qilamiz. Bu rewrite emas, balki ishlayotgan kodni buzmasdan ajratish jarayoni.

> **📌 Scenario: Order to‘lov qilish (klassik, murakkab va legacy’ga boy joy)**

## 0️⃣ Boshlang‘ich holat (LEGACY REALITY)

**models.py**
```python
class Order(models.Model):
    STATUS_CHOICES = (
        ("new", "New"),
        ("paid", "Paid"),
    )

    status = models.CharField(max_length=10, choices=STATUS_CHOICES)
    total = models.DecimalField(max_digits=12, decimal_places=2)
```

**views.py (❌ hamma narsa shu yerda)**
```python
class PayOrderView(APIView):
    def post(self, request):
        order = Order.objects.get(id=request.data["order_id"])

        if order.status != "new":
            return Response({"error": "Invalid state"}, status=400)

        if order.total > 1_000_000:
            send_email("admin@site.com")

        order.status = "paid"
        order.save()

        return Response({"status": "ok"})
```

**❌ Muammolar:**
- Business logic → view ichida
- ORM → bevosita ishlatilgan
- Test yozish deyarli imkonsiz
- Qayta ishlatib bo‘lmaydi

---

## 1️⃣ STEP 1 — Service Layer ajratamiz (Bridge bosqich)

**🎯 Maqsad: business logic’ni view’dan chiqarish**

`services/order_service.py`
```python
def pay_order(order_id: int):
    order = Order.objects.get(id=order_id)

    if order.status != "new":
        raise ValueError("Invalid state")

    if order.total > 1_000_000:
        send_email("admin@site.com")

    order.status = "paid"
    order.save()
```

`views.py`
```python
class PayOrderView(APIView):
    def post(self, request):
        try:
            pay_order(request.data["order_id"])
        except ValueError as e:
            return Response({"error": str(e)}, status=400)

        return Response({"status": "ok"})
```

> **📌 Advanced note**
> Bu hali Clean Architecture emas.
> Bu — **qutqaruv bosqichi** (legacy’dan chiqish).

---

## 2️⃣ STEP 2 — Domain Entity ajratamiz (CORE BOSHLANADI)

**🎯 Maqsad: biznes qoidani ORM’dan ajratish**

`domain/entities/order.py`
```python
class Order:
    def __init__(self, id: int, status: str, total):
        self.id = id
        self.status = status
        self.total = total

    def pay(self):
        if self.status != "new":
            raise ValueError("Invalid state")
        self.status = "paid"
```

> **📌 Endi:**
> - `pay()` — biznes qoida
> - ORM bu qoidani bilmaydi

---

## 3️⃣ STEP 3 — Repository Interface (Dependency Inversion)

`domain/interfaces/order_repository.py`
```python
from abc import ABC, abstractmethod

class OrderRepository(ABC):
    @abstractmethod
    def get(self, order_id: int): ...

    @abstractmethod
    def save(self, order): ...
```

> **📌 Muhim**
> - Interface → domain’da
> - Implementation → infrastructure’da

---

## 4️⃣ STEP 4 — Django ORM implementation (Adapter)

`infrastructure/repositories/order_django.py`
```python
from domain.entities.order import Order
from domain.interfaces.order_repository import OrderRepository
from app.models import Order as OrderModel

class DjangoOrderRepository(OrderRepository):
    def get(self, order_id: int) -> Order:
        m = OrderModel.objects.get(id=order_id)
        return Order(id=m.id, status=m.status, total=m.total)

    def save(self, order: Order):
        OrderModel.objects.filter(id=order.id).update(
            status=order.status
        )
```

> **📌 Advanced note**
> - Mapping bor (Entity ↔ ORM)
> - Bu “ortiqcha” emas — mustaqillik narxi

---

## 5️⃣ STEP 5 — Use Case (Application Layer)

**🎯 Endi haqiqiy Clean Architecture boshlanadi**

`application/use_cases/pay_order.py`
```python
class PayOrderUseCase:
    def __init__(self, repo):
        self.repo = repo

    def execute(self, order_id: int):
        order = self.repo.get(order_id)
        order.pay()
        self.repo.save(order)
```

> **📌 Bu yerda:**
> - Django yo‘q
> - ORM yo‘q
> - HTTP yo‘q
> 👉 **100% testable core**

---

## 6️⃣ STEP 6 — View = Adapter (final holat)

`views.py`
```python
class PayOrderView(APIView):
    def post(self, request):
        repo = DjangoOrderRepository()
        uc = PayOrderUseCase(repo)

        try:
            uc.execute(request.data["order_id"])
        except ValueError as e:
            return Response({"error": str(e)}, status=400)

        return Response({"status": "ok"})
```

> **📌 Advanced insight**
> - View = translator
> - Business logic = 0

---

## 7️⃣ STEP 7 — Unit Test (frameworksiz!)

`tests/test_pay_order_uc.py`
```python
class FakeRepo:
    def __init__(self):
        self.saved = False

    def get(self, _):
        return Order(id=1, status="new", total=500)

    def save(self, order):
        self.saved = True


def test_pay_order():
    uc = PayOrderUseCase(FakeRepo())
    uc.execute(1)
```

> **📌 Bu test:**
> - Django’siz
> - DB’siz
> - Fast

---

## 8️⃣ Migratsiya davomida REAL qoidalar

- ✅ Bitta moduldan boshlagin
- ✅ Test bilan yop
- ✅ Parallel ishlashga ruxsat ber
- ❌ Hammasini birdan refactor qilma
- ❌ ORM’ni core’ga kiritma

---

## 9️⃣ Oldin vs Keyin (farq)

| | Oldin | Keyin |
| :--- | :--- | :--- |
| **View’da business** | ✅ | ❌ Use Case’da |
| **ORM** | Markazda | Chetda |
| **Test** | Qiyin | Oson |
| **O‘zgartirish** | Xavfli | Oson |

---

## 🔚 Yakuniy xulosa

Bu — real, ishlaydigan refactor:
- **Rewrite yo‘q**
- **Risk minimal**
- **Clean Architecture asta-sekin kiradi**
- **Django “dushman” emas, adapter**

> **Advanced developer belgisi — legacy kodni asta-sekin sog‘lomlashtira olish.**
