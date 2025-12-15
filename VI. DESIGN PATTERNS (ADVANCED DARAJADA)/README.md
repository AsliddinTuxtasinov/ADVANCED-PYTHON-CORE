# Python Design Patterns

> **Meta-qoida:** Pattern — bu maqsad emas, balki muammoga javob. Pattern kerak bo'lgandan oldin qo'llanmaydi.

---

## 0. Pattern Qachon Paydo Bo'ladi?

### 🎯 Asosiy Konsepsiya

Pattern oldindan qo'yiladigan narsa emas. U muammo paydo bo'lgandan keyin tabiiy ravishda kelib chiqadi.

### 📊 Real Evolyutsiya

**Loyiha boshida:**
```python
def process_payment(amount: int, payment_type: str):
    if payment_type == "payme":
        # PayMe integratsiyasi
        return payme_api.charge(amount)
```

**3 oy o'tib:**
```python
def process_payment(amount: int, payment_type: str):
    if payment_type == "payme":
        return payme_api.charge(amount)
    elif payment_type == "click":
        return click_api.pay(amount)
    elif payment_type == "stripe":
        return stripe_api.create_charge(amount)
    elif payment_type == "paypal":
        return paypal_api.process(amount)
```

**❗ Muammo:** Har safar yangi provider qo'shilganda barcha kod o'zgaradi.

### ✅ Advanced Qaror

- **1-2 variant:** Oddiy `if/else` yetarli
- **3+ variant + kengayish rejasi:** Strategy Pattern kerak
- **Runtime'da tanlash:** Factory Pattern qo'shiladi

---

## 1. Strategy Pattern — "Xatti-harakat Almashinadi"

### 💡 Qachon Kerak?

- Bir xil natija, turli usullar
- Algoritm runtime'da tanlanadi
- Yangi variantlar tez-tez qo'shiladi

### 🔍 Real Ssenarilar

- To'lov tizimlari (Payme, Click, Stripe)
- Autentifikatsiya (JWT, OAuth, Session)
- Chegirma hisoblash (VIP, Aksiya, Promo)
- File eksport (PDF, Excel, CSV)

### 🐍 Pythonic Yondashuv

Python'da Strategy uchun class majburiy emas. Function yetarli:

```python
from typing import Protocol

# Protocol — type hint uchun
class PaymentStrategy(Protocol):
    async def charge(self, amount: int) -> dict: ...

# Har bir strategy — oddiy async function
async def payme_strategy(amount: int) -> dict:
    # PayMe API integration
    return {"status": "success", "provider": "payme"}

async def click_strategy(amount: int) -> dict:
    # Click API integration
    return {"status": "success", "provider": "click"}

# Service strategy bilan ishlaydi
class PaymentService:
    def __init__(self, strategy: PaymentStrategy):
        self.strategy = strategy
    
    async def process(self, amount: int) -> dict:
        return await self.strategy(amount)

# Ishlatish
service = PaymentService(payme_strategy)
result = await service.process(10000)
```

### ⚡ Dynamic Strategy Selection

```python
def get_payment_strategy(provider: str) -> PaymentStrategy:
    strategies = {
        "payme": payme_strategy,
        "click": click_strategy,
        "stripe": stripe_strategy,
    }
    return strategies[provider]

# Runtime'da tanlash
strategy = get_payment_strategy(user_choice)
service = PaymentService(strategy)
```

### ❌ Keng Tarqalgan Xato

```python
# Anti-pattern: Class hierarchy kerak deb o'ylash
class PaymeStrategy:
    def charge(self): ...

class ClickStrategy:
    def charge(self): ...
```

**Muammo:** Agar farq faqat 2-3 qator bo'lsa, Strategy kerak emas.

### 📐 Architecture Ta'siri

- ✅ Service kodi tozalanadi
- ✅ Yangi strategy qo'shish — mavjud kodga tegmaslik
- ✅ Test qilish osonlashadi (mock strategy)

---

## 2. Factory Pattern — "Kimni Yaratishni Bilmayman"

### 💡 Qachon Kerak?

- Object yaratish runtime'da hal qilinadi
- ENV, config yoki user type ga bog'liq
- Yaratish logic'i murakkab

### 🔍 Real Ssenarilar

- Database backend (PostgreSQL, MySQL, SQLite)
- Storage (S3, Google Cloud, Local)
- Notification (Email, SMS, Push)
- Auth backend (JWT, OAuth, LDAP)

### 🐍 Pythonic Implementation

```python
from typing import Protocol
from enum import Enum

class StorageBackend(Protocol):
    def save(self, key: str, data: bytes) -> None: ...
    def load(self, key: str) -> bytes: ...

class S3Storage:
    def save(self, key: str, data: bytes) -> None:
        # AWS S3 logic
        pass
    
    def load(self, key: str) -> bytes:
        # AWS S3 logic
        return b"data"

class LocalStorage:
    def save(self, key: str, data: bytes) -> None:
        # Local filesystem logic
        pass
    
    def load(self, key: str) -> bytes:
        return b"data"

# Simple Factory
def create_storage(storage_type: str) -> StorageBackend:
    if storage_type == "s3":
        return S3Storage()
    elif storage_type == "local":
        return LocalStorage()
    else:
        raise ValueError(f"Unknown storage: {storage_type}")

# Ishlatish
storage = create_storage(settings.STORAGE_TYPE)
storage.save("file.txt", b"content")
```

### 🚀 Registry-Based Factory (Plugin System)

```python
# Advanced: Extensible Factory
STORAGE_REGISTRY: dict[str, type] = {}

def register_storage(name: str):
    def decorator(cls):
        STORAGE_REGISTRY[name] = cls
        return cls
    return decorator

@register_storage("s3")
class S3Storage:
    pass

@register_storage("gcs")
class GCSStorage:
    pass

def create_storage(name: str) -> StorageBackend:
    if name not in STORAGE_REGISTRY:
        raise ValueError(f"Storage '{name}' not registered")
    return STORAGE_REGISTRY[name]()
```

### ❌ Factory Anti-Pattern

```python
# Noto'g'ri: Factory ichida business logic
def create_payment(amount: int, user):
    if user.is_vip:
        # Business logic bu yerda bo'lmasligi kerak
        amount = amount * 0.9
    return Payment(amount)
```

**To'g'risi:** Factory faqat object yaratadi, business logic service'da.

### 📌 Advanced Note

Factory:
- Object **yaratish** uchun
- Business logic **emas**
- Configuration-based **tanlash**

---

## 3. Repository Pattern — "Business DB'ni Bilmasin"

### 💡 Qachon Kerak?

- ORM almashishi mumkin (SQLAlchemy → Tortoise → Prisma)
- Business logic murakkab
- Test-friendly kod kerak
- Domain-driven design

### 🔍 Real Ssenarilar

- Banking tizimlari
- Contract management
- ERP/CRM
- Har qanday business-heavy app

### 🐍 Repository vs ORM

**ORM — texnik qatlam:**
```python
# Texnik tilda
session.query(User).filter_by(status="active").all()
```

**Repository — business tili:**
```python
# Business tilda
user_repo.get_active_users()
```

### 💻 Implementation

```python
from typing import Protocol
from sqlalchemy.ext.asyncio import AsyncSession

class UserRepository(Protocol):
    async def get_active_users(self) -> list[User]: ...
    async def get_by_email(self, email: str) -> User | None: ...
    async def save(self, user: User) -> None: ...

class SQLAlchemyUserRepository:
    def __init__(self, session: AsyncSession):
        self.session = session
    
    async def get_active_users(self) -> list[User]:
        result = await self.session.execute(
            select(User).where(User.status == "active")
        )
        return result.scalars().all()
    
    async def get_by_email(self, email: str) -> User | None:
        result = await self.session.execute(
            select(User).where(User.email == email)
        )
        return result.scalar_one_or_none()
    
    async def save(self, user: User) -> None:
        self.session.add(user)
        await self.session.commit()
```

### 🎯 CQRS Pattern bilan

```python
# Query — faqat o'qish
class UserQueryRepository:
    async def list_active(self) -> list[User]: ...
    async def search(self, query: str) -> list[User]: ...

# Command — faqat yozish
class UserCommandRepository:
    async def create(self, user: User) -> None: ...
    async def update(self, user: User) -> None: ...
    async def delete(self, user_id: int) -> None: ...
```

**Afzalligi:** Read va write optimizatsiya qilish oson.

### ✅ Advanced Qaror

- **CRUD-only app:** Repository ortiqcha
- **Business rules ko'p:** Repository zarur
- **Multiple data sources:** Repository majburiy

---

## 4. Dependency Injection — "Bog'liqlikni Bo'shatish"

### 💡 Qachon Kerak?

- Test yozmoqchisiz
- Component'larni almashtirish kerak
- Global variable'dan qochmoqchisiz

### 🔍 Real Ssenarilar

- FastAPI service layer
- Test uchun mock repository
- Feature toggle
- Multi-tenant applications

### 🐍 FastAPI DI

```python
from fastapi import Depends, FastAPI
from sqlalchemy.ext.asyncio import AsyncSession

app = FastAPI()

# Dependency function
async def get_db_session() -> AsyncSession:
    async with async_session() as session:
        yield session

# Repository dependency
def get_user_repo(
    session: AsyncSession = Depends(get_db_session)
) -> UserRepository:
    return SQLAlchemyUserRepository(session)

# Service dependency
def get_user_service(
    repo: UserRepository = Depends(get_user_repo)
) -> UserService:
    return UserService(repo)

# Endpoint
@app.get("/users")
async def list_users(
    service: UserService = Depends(get_user_service)
):
    return await service.list_active_users()
```

### 🧪 Test-Friendly

```python
# Test'da mock repository
class MockUserRepository:
    async def get_active_users(self):
        return [User(id=1, email="test@example.com")]

def test_list_users():
    # DI override
    app.dependency_overrides[get_user_repo] = lambda: MockUserRepository()
    
    # Test
    response = client.get("/users")
    assert response.status_code == 200
```

### ❌ DI Anti-Pattern

```python
# Global import — yashirin dependency
from app.database import session

class UserService:
    def get_users(self):
        return session.query(User).all()
```

**Muammo:**
- Test qilish qiyin
- Coupling yuqori
- Mock qilish mumkin emas

### 📐 Composition Root

```python
# Barcha dependency wiring — bitta joyda
def get_user_service(
    user_repo: UserRepository = Depends(get_user_repo),
    email_service: EmailService = Depends(get_email_service),
    cache: CacheBackend = Depends(get_cache),
) -> UserService:
    return UserService(
        repo=user_repo,
        email=email_service,
        cache=cache
    )
```

---

## 5. Builder Pattern — "Object Juda Murakkab"

### 💡 Qachon Kerak?

- 10+ optional field
- Step-by-step yaratish
- Validation har qadamda
- Conditional creation

### 🔍 Real Ssenarilar

- Report generation
- PDF contract
- Complex configuration
- User onboarding flow

### 🐍 Builder vs Pydantic

| Use Case | Tool |
|----------|------|
| API validation | Pydantic |
| Step-by-step business object | Builder |
| Conditional creation | Builder |
| Simple data structure | Pydantic |

### 💻 Implementation

```python
from dataclasses import dataclass
from typing import Optional

@dataclass
class Report:
    title: str
    data: dict
    format: str
    filters: Optional[dict] = None
    sorting: Optional[str] = None
    pagination: Optional[dict] = None

class ReportBuilder:
    def __init__(self):
        self._title: Optional[str] = None
        self._data: Optional[dict] = None
        self._format: str = "pdf"
        self._filters: Optional[dict] = None
        self._sorting: Optional[str] = None
        self._pagination: Optional[dict] = None
    
    def with_title(self, title: str) -> "ReportBuilder":
        self._title = title
        return self
    
    def with_data(self, data: dict) -> "ReportBuilder":
        self._data = data
        return self
    
    def with_format(self, format: str) -> "ReportBuilder":
        self._format = format
        return self
    
    def with_filters(self, filters: dict) -> "ReportBuilder":
        self._filters = filters
        return self
    
    def build(self) -> Report:
        if not self._title or not self._data:
            raise ValueError("Title and data required")
        
        return Report(
            title=self._title,
            data=self._data,
            format=self._format,
            filters=self._filters,
            sorting=self._sorting,
            pagination=self._pagination
        )

# Fluent interface
report = (
    ReportBuilder()
    .with_title("Sales Report")
    .with_data({"sales": 1000})
    .with_format("pdf")
    .with_filters({"date": "2024"})
    .build()
)
```

### 🚀 Immutable Builder

```python
from dataclasses import dataclass, replace

@dataclass(frozen=True)
class ImmutableReportBuilder:
    title: Optional[str] = None
    data: Optional[dict] = None
    format: str = "pdf"
    
    def with_title(self, title: str) -> "ImmutableReportBuilder":
        return replace(self, title=title)
    
    def with_data(self, data: dict) -> "ImmutableReportBuilder":
        return replace(self, data=data)
    
    def build(self) -> Report:
        return Report(
            title=self.title,
            data=self.data,
            format=self.format
        )
```

---

## 6. Observer Pattern — "Side-Effect Ko'payadi"

### 💡 Qachon Kerak?

- 1 action → ko'p reaction
- Loose coupling kerak
- Event-driven architecture

### 🔍 Real Ssenarilar

- User created → (Email, SMS, Analytics, Slack)
- Order paid → (Invoice, Notification, Stock update)
- Contract signed → (PDF, Email, Archive, Audit log)

### 🐍 Simple Observer

```python
from typing import Callable, Awaitable

EventHandler = Callable[[dict], Awaitable[None]]

class EventBus:
    def __init__(self):
        self._handlers: dict[str, list[EventHandler]] = {}
    
    def subscribe(self, event: str, handler: EventHandler):
        if event not in self._handlers:
            self._handlers[event] = []
        self._handlers[event].append(handler)
    
    async def publish(self, event: str, data: dict):
        if event not in self._handlers:
            return
        
        for handler in self._handlers[event]:
            try:
                await handler(data)
            except Exception as e:
                # Log error but don't stop other handlers
                print(f"Handler failed: {e}")

# Event handlers
async def send_welcome_email(data: dict):
    user_id = data["user_id"]
    # Send email logic
    print(f"Email sent to user {user_id}")

async def send_slack_notification(data: dict):
    # Slack logic
    print(f"Slack notified about user {data['user_id']}")

# Setup
event_bus = EventBus()
event_bus.subscribe("user.created", send_welcome_email)
event_bus.subscribe("user.created", send_slack_notification)

# Trigger
await event_bus.publish("user.created", {"user_id": 123})
```

### ⚡ Async Observer (Production)

```python
import asyncio

class AsyncEventBus:
    async def publish(self, event: str, data: dict):
        if event not in self._handlers:
            return
        
        # Parallel execution
        await asyncio.gather(
            *[handler(data) for handler in self._handlers[event]],
            return_exceptions=True
        )
```

### 🚀 Distributed Observer

```python
# Celery example
from celery import Celery

celery = Celery("app")

@celery.task
def send_email_task(user_id: int):
    # Email logic
    pass

# Publish event
send_email_task.delay(user_id=123)
```

### ❌ Django Signals Anti-Pattern

```python
# Yashirin dependency
from django.db.models.signals import post_save

@receiver(post_save, sender=User)
def user_created(sender, instance, created, **kwargs):
    if created:
        send_email(instance.email)
```

**Muammo:**
- Implicit behavior
- Debug qiyin
- Flow ko'rinmaydi

**To'g'risi:** Explicit event bus ishlatish.

---

## 7. Pattern Composition

### 🧠 Real Architecture Flow

```
Request
  ↓
Controller (FastAPI endpoint)
  ↓
DI (Depends)
  ↓
Service
  ├→ Strategy (payment method)
  ├→ Repository (data access)
  └→ Observer (side effects)
```

### 💻 Full Example

```python
# 1. Strategy
async def payme_strategy(amount: int) -> dict:
    return {"status": "paid", "provider": "payme"}

# 2. Repository
class OrderRepository:
    async def save(self, order: Order): ...

# 3. Observer
event_bus = EventBus()

# 4. Service (composition)
class OrderService:
    def __init__(
        self,
        payment_strategy: PaymentStrategy,
        order_repo: OrderRepository,
        event_bus: EventBus
    ):
        self.payment = payment_strategy
        self.repo = order_repo
        self.events = event_bus
    
    async def create_order(self, order: Order):
        # Strategy
        payment_result = await self.payment(order.amount)
        
        # Repository
        await self.repo.save(order)
        
        # Observer
        await self.events.publish("order.created", {
            "order_id": order.id
        })

# 5. DI (FastAPI)
def get_order_service(
    payment: PaymentStrategy = Depends(get_payment_strategy),
    repo: OrderRepository = Depends(get_order_repo),
    events: EventBus = Depends(get_event_bus)
) -> OrderService:
    return OrderService(payment, repo, events)
```

---

## 8. Performance & Scale

### ⚡ Performance Impact

| Pattern | Overhead | Benefit |
|---------|----------|---------|
| Strategy | Minimal (function call) | Maintainability |
| Repository | Small (abstraction layer) | Flexibility |
| Observer | Medium (async calls) | Decoupling |
| Factory | Minimal (object creation) | Clean code |

### 🚀 Optimization Tips

**Strategy:**
- Function inline (fast)
- Lazy loading

**Repository:**
- Batch queries
- Eager loading
- Connection pooling

**Observer:**
- Async execution
- Message queue (Celery, RabbitMQ)
- Error handling

### 📊 When to Optimize

```
1. Correctness (to'g'ri ishlash)
2. Clarity (tushunarli kod)
3. Optimization (tezlik)
```

**Advanced qoida:** Premature optimization — evil.

---

## 9. Interview & Decision Framework

### 💬 Typical Interview Questions

**Q: Qachon Singleton ishlatmaslik kerak?**

A: Singleton muammolari:
- Test qiyin (global state)
- Thread safety issue
- Hidden dependency
- Alternative: DI ishlatish

**Q: Qachon pattern ishlatmaslik kerak?**

A: Pattern keraksiz qachon:
- Muammo hali yo'q
- Kod oddiy (3-4 qator)
- Overengineering bo'ladi
- YAGNI (You Aren't Gonna Need It)

**Q: Strategy vs Factory farqi?**

A:
- Strategy: behavior tanlash
- Factory: object yaratish
- Ko'pincha birga ishlatiladi

**Q: Repository nega ORM'dan yaxshi?**

A:
- Business tili (domain language)
- Test-friendly
- ORM almashtirish oson
- Ammo CRUD-only app'da ortiqcha

### 🎯 Decision Tree

```
Muammo bor?
├─ Ha
│  ├─ Behavior almashadi? → Strategy
│  ├─ Object yaratish murakkab? → Factory/Builder
│  ├─ DB bilan coupling kuchli? → Repository
│  ├─ Side-effect ko'p? → Observer
│  └─ Test qiyin? → DI qo'shish
└─ Yo'q → Pattern kerak emas
```

---

## 🎓 Final Advanced Summary

### ✅ Pythonic Pattern Principles

1. **Function > Class**
   - Strategy = function
   - Kam inheritance

2. **Explicit > Implicit**
   - DI > global import
   - Event bus > signals

3. **Composition > Inheritance**
   - Mix patterns
   - Flexible architecture

4. **Practical > Theoretical**
   - Real muammo → real pattern
   - Academic purity emas

### ❌ Anti-Patterns (Keraksiz)

- **Singleton:** Global state, test qiyin
- **Abstract Factory:** Python'da ortiqcha
- **Overengineering:** 3 qator uchun class hierarchy

### 🚀 Real Project Checklist

- [ ] Payment: Strategy ✅
- [ ] DB access: Repository (agar business logic murakkab)
- [ ] Object creation: Factory (agar runtime choice)
- [ ] Dependencies: DI (FastAPI Depends)
- [ ] Side-effects: Event bus (agar ko'p listener)

### 📚 Further Reading

- Martin Fowler — Patterns of Enterprise Application Architecture
- FastAPI documentation — Dependency Injection
- Python Cookbook — Design Patterns chapter

---

**Muhim eslatma:** Pattern — bu maqsad emas, tool. Muammoni hal qilish uchun eng oddiy yo'l — eng yaxshi yo'l. Pattern shundan keyingisi.
