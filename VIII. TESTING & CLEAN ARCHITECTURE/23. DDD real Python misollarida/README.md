# 23 DDD (Domain-Driven Design) — Real Python misollarida (Advanced Article)

**🎯 Maqsad:** DDD’ni “terminlar to‘plami” sifatida emas, balki real backend loyihada qaror qabul qilish modeli sifatida tushunish.

---

## 🧠 DDD — aslida nima?

### DDD bu:
- **framework emas** ❌
- **folder struktura emas** ❌
- **ORM pattern emas** ❌

### DDD bu:
- **Murakkab biznes qoidalarini kod orqali to‘g‘ri ifodalash usuli** ✅

> **Muhim haqiqat:**
> Agar loyihangda:
> - `if`, `status`, `type`, `flag`lar ko‘payib ketgan bo‘lsa
> - logika `views/services` ichida yoyilib ketgan bo‘lsa
> - test yozish qiyin bo‘lsa
>
> 👉 **DDD kechikib bo‘lsa ham kerak bo‘ladi**

---

## 1️⃣ DDD ning 4 ta asosiy ustuni

### 1. Domain
- Biznes qoidalar joylashgan qatlam
- ➡️ **eng muhim joy**

### 2. Application (Use Cases)
- Domain’ni qanday ishlatishni belgilaydi
- `transaction`, `orchestration`

### 3. Infrastructure
- DB
- ORM
- API
- Cache
- External service

### 4. Interface / Delivery
- FastAPI / Django views
- CLI
- gRPC
- Celery task

> **📌 Dependency qoidasi**
> - Tashqi qatlamlar → ichki qatlamlarga bog‘lanadi
> - **Hech qachon aksincha emas**

---

## 2️⃣ DDD qachon kerak (qachon EMAS)

### ✅ Kerak bo‘ladi agar:
- Biznes qoidalar murakkab
- Holatlar (state) ko‘p
- Qoidalar tez-tez o‘zgaradi
- Katta jamoa

### ❌ Kerak emas agar:
- CRUD + admin panel
- MVP / landing
- 1–2 endpoint

---

## 3️⃣ Entity — oddiy ORM model emas

### ❌ Yomon (anemic model)
```python
class Order(models.Model):
    status = models.CharField(...)
    total = models.DecimalField(...)
```
> *Barcha logika tashqarida.*

### ✅ To‘g‘ri DDD Entity (rich model)
```python
class Order:
    def __init__(self, items: list["OrderItem"]):
        self.items = items
        self.status = "NEW"

    def total_price(self) -> int:
        return sum(item.price for item in self.items)

    def pay(self):
        if self.status != "NEW":
            raise DomainError("Order already processed")
        self.status = "PAID"
```

> **📌 Muhim:**
> - Entity o‘z holatini o‘zi boshqaradi
> - Hech qanday `if status == ...` tashqarida bo‘lmasligi kerak

---

## 4️⃣ Value Object — immutable biznes qiymat

### Belgilari:
- o‘zgarmaydi (immutable)
- identity yo‘q
- faqat qiymat

```python
@dataclass(frozen=True)
class Money:
    amount: int
    currency: str

    def add(self, other: "Money") -> "Money":
        if self.currency != other.currency:
            raise ValueError("Currency mismatch")
        return Money(self.amount + other.amount, self.currency)
```

> **📌 Note:**
> Money, Email, PhoneNumber, Percentage — Value Object bo‘lishi shart

---

## 5️⃣ Aggregate — consistency chegarasi

**Aggregate** — birgalikda transaction bo‘ladigan obyektlar guruhi.

### Misol: Order Aggregate
```python
class Order:
    def __init__(self):
        self.items: list[OrderItem] = []

    def add_item(self, item: OrderItem):
        self.items.append(item)
```

> **📌 Qoidalar:**
> - Tashqaridan faqat **Aggregate Root** bilan ishlaysan
> - Ichki entity’ga to‘g‘ridan-to‘g‘ri murojaat ❌

---

## 6️⃣ Repository — persistence abstraction

### ❌ Yomon
```python
Order.objects.filter(...)
```

### ✅ To‘g‘ri
```python
class OrderRepository(Protocol):
    def get(self, order_id: UUID) -> Order: ...
    def save(self, order: Order) -> None: ...
```

**Infrastructure’da:**
```python
class DjangoOrderRepository:
    def get(self, order_id):
        ...
```

> **📌 Natija:**
> - Domain ORM’dan mustaqil
> - Test oson

---

## 7️⃣ Use Case — Application layer yuragi

**Use Case:**
- Domain obyektlarini chaqiradi
- Transaction boshqaradi
- Business flow

```python
class PayOrderUseCase:
    def __init__(self, repo: OrderRepository):
        self.repo = repo

    def execute(self, order_id: UUID):
        order = self.repo.get(order_id)
        order.pay()
        self.repo.save(order)
```

> **📌 Note:**
> - `execute()` → bitta aniq ish
> - Hech qanday HTTP, ORM, serializer yo‘q

---

## 8️⃣ Domain Event — side-effectlarni ajratish

```python
class OrderPaid(DomainEvent):
    order_id: UUID
```

**Use case ichida:**
```python
order.pay()
events.publish(OrderPaid(order.id))
```

> **📌 Email, notification, analytics — Domain ichida emas**

---

## 9️⃣ DDD + Testing = ❤️

### Domain test (ENG MUHIM)
```python
def test_order_can_be_paid():
    order = Order(...)
    order.pay()
    assert order.status == "PAID"
```

- ➡️ DB ❌
- ➡️ API ❌
- ➡️ Mock ❌

> **📌 Agar domain test yozish qiyin bo‘lsa — dizayn noto‘g‘ri**

---

## 🔥 Advanced NOTE’lar (Senior level)

### ⚠️ 1. Folder struktura ≠ DDD
```
domain/
application/
infrastructure/
```
> *Bu yetarli emas — fikrlash modeli muhim*

### ⚠️ 2. Django ORM = Infrastructure
> *ORM modelni domain deb o‘ylama.*

### ⚠️ 3. DDD + CQRS
- **Read** → oddiy SQL / ORM
- **Write** → Domain orqali

### ⚠️ 4. DDD sekin boshlanadi, lekin keyin tezlashtiradi

# 🧩 Yakuniy xulosa

**DDD:**
- katta loyihalarda majburiy
- testability’ni oshiradi
- biznes va kodni bir tilga keltiradi

---
---

# DDD amaliyotda: FastAPI, Django Monolit va Event-Driven Architecture (ADVANCED)

Quyida real production loyihalarda DDD qanday ishlatilishini qadam-qadam, arxitektura qarorlari va senior-level note’lar bilan ko‘rib chiqamiz.

## 🔹 1. DDD + FastAPI — Real Project Architecture

### 🎯 Qachon bu kombinatsiya ideal?
- Microservice yoki modulga bo‘lingan tizim
- Tez o‘sadigan biznes logika
- Test-first / clean architecture talabi

### 📁 Tavsiya etiladigan struktura
```
app/
├── domain/
│   ├── entities/
│   ├── value_objects/
│   ├── events/
│   ├── exceptions.py
│
├── application/
│   ├── use_cases/
│   ├── ports/          # repository, event publisher
│
├── infrastructure/
│   ├── db/
│   ├── repositories/
│   ├── event_bus/
│
├── interfaces/
│   ├── api/
│   │   ├── routers/
│   │   └── schemas/
│
└── main.py
```

> **📌 Asosiy qoida:** FastAPI → faqat Interface layer

### 🧠 Real misol: Buyurtmani to‘lash

**Domain (toza Python)**
```python
class Order:
    def __init__(self, status: str):
        self.status = status

    def pay(self):
        if self.status != "NEW":
            raise DomainError("Order already paid")
        self.status = "PAID"
```

**Application (Use Case)**
```python
class PayOrderUseCase:
    def __init__(self, repo):
        self.repo = repo

    def execute(self, order_id):
        order = self.repo.get(order_id)
        order.pay()
        self.repo.save(order)
```

**Interface (FastAPI)**
```python
@router.post("/orders/{order_id}/pay")
def pay(order_id: UUID, uc: PayOrderUseCase = Depends()):
    uc.execute(order_id)
    return {"status": "ok"}
```

> **📌 Senior NOTE:**
> - FastAPI dependency injection → UseCase inject qilish uchun ideal
> - Domain layer’da pydantic, Depends, Request ❌

### 🧪 Test strategiya (FastAPI + DDD)
| Layer | Test turi |
| :--- | :--- |
| **Domain** | Pure unit |
| **Use Case** | Unit + fake repo |
| **API** | Minimal integration |

> ➡️ Test piramida to‘g‘ri bo‘ladi

---

## 🔹 2. DDD + Django Monolit (To‘g‘ri yo‘li)

### ❗ Eng katta xato
> **Django Model = Domain ❌**

### 📁 Django monolit + DDD struktura
```
orders/
├── domain/
│   ├── entities.py
│   ├── value_objects.py
│
├── application/
│   ├── services.py
│
├── infrastructure/
│   ├── models.py     # Django ORM
│   ├── repositories.py
│
├── interfaces/
│   ├── views.py
│   ├── serializers.py
```

### 🔁 ORM ↔ Domain mapping
`# infrastructure/models.py`
```python
class OrderModel(models.Model):
    status = models.CharField(...)
```

`# repositories.py`
```python
class OrderRepository:
    def get(self, id):
        orm = OrderModel.objects.get(id=id)
        return Order(status=orm.status)

    def save(self, order: Order):
        ...
```

> **📌 Advanced NOTE:**
> - ORM — persistence detail xolos
> - Domain → ORM’ni bilmasligi shart

### 🧠 Django’da qachon DDD kerak?
- Murakkab state machine
- Contract, billing, workflow
- Audit, history, rules
- **Agar oddiy CRUD bo‘lsa → overkill**

---

## 🔹 3. DDD + Event-Driven Architecture (ADVANCED)

### 🎯 Muammo
Domain ichida:
- email yuborish
- notification
- analytics
- billing trigger
- **❌ Side-effectlar Domain’ni buzadi**

### ✅ Yechim: Domain Events

**Domain Event**
```python
class OrderPaid:
    def __init__(self, order_id):
        self.order_id = order_id
```

**Entity ichida event chiqarish**
```python
class Order:
    def pay(self):
        self.status = "PAID"
        self.events.append(OrderPaid(self.id))
```

**Use Case**
```python
order.pay()
repo.save(order)
event_bus.publish(order.events)
```

**Event Handler (Infrastructure)**
```python
def send_email(event: OrderPaid):
    ...
```

> **📌 Muhim:**
> - Domain → eventni biladi
> - Email, Kafka, Celery → bilmaydi

### ⚙️ Event Bus variantlari
| Variant | Qachon |
| :--- | :--- |
| **In-memory** | Monolit |
| **Celery** | Background task |
| **Kafka / RabbitMQ** | Microservices |
| **Redis Pub/Sub** | Light async |

---

## 🔥 Senior-Level NOTE’lar

### ⚠️ 1. DDD sekin boshlanadi
Lekin:
- test yozish tezlashadi
- refactor qo‘rqinchli bo‘lmaydi
- biznes talabi oson qo‘shiladi

### ⚠️ 2. DDD hamma joyda emas
- Admin panel
- Report
- Read-only API
> → **DDD shart emas**

### ⚠️ 3. DDD + CQRS = POWER
- **Write** → Domain
- **Read** → SQL/View/Materialized table

---

## 🧩 Yakuniy xulosa

| Stack | Tavsiya |
| :--- | :--- |
| **FastAPI** | ✅ Juda mos |
| **Django monolit** | ✅ Agar to‘g‘ri ajratsang |
| **Event-driven** | ✅ Murakkab tizimlar uchun |

---
---

# Django Billing System — DDD bilan (REAL PRODUCTION ARTICLE, ADVANCED)

**🎯 Maqsad:** Django’da real billing / payment tizimini DDD + Clean Architecture asosida to‘g‘ri dizayn qilishni o‘rganish.

## 🧠 Billing domenini to‘g‘ri tushunish (ENG MUHIM QADAM)

- **❌ Katta xato:** “Billing — bu Payment model qo‘shish”
- **✅ To‘g‘ri fikrlash:** Billing — bu qoidalar, holatlar, cheklovlar va pul oqimi

### 💼 Real biznes savollari
Billing tizimi quyidagilarga javob berishi kerak:
- Qachon to‘lov qilish mumkin / mumkin emas?
- Qisman to‘lov bo‘ladimi?
- Qayta to‘lov (retry)?
- Bekor qilish (cancel)?
- Refund?
- Valyuta?
- Audit?

> **📌 DDD shu savollarga javob berish uchun yaratilgan**

---

## 🔹 1. Bounded Context: Billing

```
Billing Context
├── Invoice
├── Payment
├── Balance
├── Money
├── BillingStatus
```

> **📌 NOTE:**
> Billing — alohida bounded context. User, Order, Contract — boshqa context bo‘lishi mumkin.

---

## 🔹 2. Folder Structure (Django + DDD)

```
billing/
├── domain/
│   ├── entities/
│   │   ├── invoice.py
│   │   └── payment.py
│   ├── value_objects/
│   │   └── money.py
│   ├── events/
│   │   └── payment_events.py
│   ├── exceptions.py
│
├── application/
│   ├── use_cases/
│   │   ├── pay_invoice.py
│   │   └── refund_payment.py
│
├── infrastructure/
│   ├── models.py          # Django ORM
│   ├── repositories.py
│
├── interfaces/
│   ├── views.py
│   └── serializers.py
```

> **🔥 Muhim qoida:** `models.py` — domain emas, bu infrastructure

---

## 🔹 3. Value Object: Money (Pul bilan xato qilinmaydi)

`# domain/value_objects/money.py`
```python
from dataclasses import dataclass

@dataclass(frozen=True)
class Money:
    amount: int  # tiyin / cent
    currency: str = "UZS"

    def add(self, other: "Money") -> "Money":
        if self.currency != other.currency:
            raise ValueError("Currency mismatch")
        return Money(self.amount + other.amount, self.currency)

    def is_positive(self) -> bool:
        return self.amount > 0
```

> **📌 Advanced NOTE:**
> - Decimal emas → integer (cent) ishlatiladi
> - Float — har doim ❌

---

## 🔹 4. Entity: Invoice (Aggregate Root)

`# domain/entities/invoice.py`
```python
from billing.domain.exceptions import DomainError

class Invoice:
    def __init__(self, total: Money):
        self.total = total
        self.paid = Money(0, total.currency)
        self.status = "UNPAID"

    def pay(self, amount: Money):
        if self.status == "PAID":
            raise DomainError("Invoice already paid")

        self.paid = self.paid.add(amount)

        if self.paid.amount > self.total.amount:
            raise DomainError("Overpayment")

        if self.paid.amount == self.total.amount:
            self.status = "PAID"
```

> **🔥 MUHIM:**
> - Hech qanday `if status == ...` tashqarida YO‘Q
> - Barcha qoidalar Entity ichida

---

## 🔹 5. Domain Event: PaymentCompleted

`# domain/events/payment_events.py`
```python
class PaymentCompleted:
    def __init__(self, invoice_id):
        self.invoice_id = invoice_id
```

> **📌 Event:** side-effect trigger, lekin side-effect emas

---

## 🔹 6. Repository (ORM’dan uzilish)

`# application/ports.py`
```python
class InvoiceRepository:
    def get(self, invoice_id): ...
    def save(self, invoice): ...
```

`# infrastructure/repositories.py`
```python
class DjangoInvoiceRepository(InvoiceRepository):
    def get(self, invoice_id):
        orm = InvoiceModel.objects.get(id=invoice_id)
        return Invoice(total=Money(orm.total))

    def save(self, invoice):
        ...
```

> **📌 Advanced NOTE:**
> - Domain → ORM’ni bilmaydi
> - Django ORM — detail xolos

---

## 🔹 7. Use Case: Pay Invoice

`# application/use_cases/pay_invoice.py`
```python
class PayInvoiceUseCase:
    def __init__(self, repo, event_bus):
        self.repo = repo
        self.event_bus = event_bus

    def execute(self, invoice_id, amount: Money):
        invoice = self.repo.get(invoice_id)
        invoice.pay(amount)
        self.repo.save(invoice)
        self.event_bus.publish(PaymentCompleted(invoice_id))
```

> **🔥 Use Case vazifasi:**
> - Transaction
> - Domain chaqirish
> - Event chiqarish

---

## 🔹 8. Django View (Interface layer)

```python
class PayInvoiceView(APIView):
    def post(self, request, invoice_id):
        amount = Money(request.data["amount"])
        uc = PayInvoiceUseCase(repo, event_bus)
        uc.execute(invoice_id, amount)
        return Response({"status": "PAID"})
```

> **📌 View:** yupqa qatlam. Hech qanday biznes logika YO‘Q.

---

## 🔹 9. Testing Strategy (ENG KUCHLI JOY)

**✅ Domain test (DB yo‘Q)**
```python
def test_invoice_full_payment():
    invoice = Invoice(total=Money(1000))
    invoice.pay(Money(1000))
    assert invoice.status == "PAID"
```

**✅ Use Case test (fake repo)**
```python
def test_use_case_triggers_event():
    ...
```

**❌ Domain testda Django ❌**

---

## 🔥 Senior-Level XATOLAR

- ❌ **Django Model ichida pay()**
> *Bu Active Record — DDD emas*
- ❌ **Serializer’da biznes qoidalar**
> *Bu arxitektura o‘limi*
- ❌ **Signal bilan domain logic**
> *Signal — infra daraja*

### 🧠 Qachon bu arxitektura O‘ZINI OQLAYDI?

| Holat | Javob |
| :--- | :--- |
| **Billing / Contract / Payment** | ✅ HA |
| **Murakkab holatlar** | ✅ HA |
| **Audit / history** | ✅ HA |
| **Oddiy CRUD** | ❌ YO‘Q |

---

## 🧩 Yakuniy xulosa

**DDD bilan Django Billing:**
- testable
- kengaytiriladigan
- biznesga mos
- production-ready

---
---

# DDD + Celery — Asinxron biznes logikani TO‘G‘RI qurish (ADVANCED, PRODUCTION)

**🎯 Maqsad:** Django + Celery ishlatayotganda Domain’ni buzmasdan, background task’larni DDD asosida qurishni o‘rganish.

> *Bu joyda ko‘pchilik eng katta arxitektura xatolarni qiladi.*

## ❌ ENG KENG TARQALGAN XATO (REAL HAYOTDA)

```python
@shared_task
def pay_invoice(invoice_id):
    invoice = Invoice.objects.get(id=invoice_id)
    if invoice.status != "PAID":
        invoice.status = "PAID"
        invoice.save()
        send_email(...)
```

**❌ Muammolar:**
- Celery task = biznes logika
- ORM = domain
- Test qilish qiyin
- Side-effectlar aralashib ketgan

👉 **Bu DDD emas**

---

## ✅ TO‘G‘RI FIKRLASH MODELI

**Asosiy qoida:**
- **Celery — bu Infrastructure layer**
- Celery:
  - Domain’ni bilmaydi
  - Qoidalarni bajarmaydi
  - Faqat event/reactor

### 🧠 To‘g‘ri oqim (MENTAL MODEL)
```
HTTP / API
   ↓
Use Case
   ↓
Domain (Entity / Aggregate)
   ↓
Domain Event
   ↓
Event Handler
   ↓
Celery Task
   ↓
Side-effect (email, sms, pdf, webhook)
```

> **📌 Celery hech qachon Domain’ga kirmaydi**

---

## 🔹 1. Domain Event — markaziy nuqta

`# billing/domain/events/payment_events.py`
```python
class InvoicePaid:
    def __init__(self, invoice_id: int):
        self.invoice_id = invoice_id
```

> **📌 Event:** faqat fakt. “NIMA bo‘ldi?” — “Invoice PAID bo‘ldi”

---

## 🔹 2. Entity — faqat event chiqaradi

```python
class Invoice:
    def pay(self, amount: Money):
        ...
        if self.is_fully_paid():
            self.status = "PAID"
            self.events.append(InvoicePaid(self.id))
```

> **❗ Entity:**
> - email yubormaydi
> - celery chaqirmaydi
> - faqat event ishlab chiqaradi

---

## 🔹 3. Use Case — eventlarni tashqariga chiqaradi

```python
class PayInvoiceUseCase:
    def __init__(self, repo, event_bus):
        self.repo = repo
        self.event_bus = event_bus

    def execute(self, invoice_id, amount):
        invoice = self.repo.get(invoice_id)
        invoice.pay(amount)
        self.repo.save(invoice)

        for event in invoice.events:
            self.event_bus.publish(event)
```

> **📌 MUHIM:**
> Event’lar DB transaction tugagandan keyin chiqishi kerak (production’da `on_commit` ishlatiladi)

---

## 🔹 4. Event Bus — Domain va Celery orasidagi ko‘prik

```python
class EventBus:
    def publish(self, event):
        if isinstance(event, InvoicePaid):
            send_invoice_paid.delay(event.invoice_id)
```

> **📌 Bu joy:** Infrastructure. Mapping qilinadi: Event → Celery task.

---

## 🔹 5. Celery Task — faqat side-effect

```python
@shared_task(bind=True, autoretry_for=(Exception,), retry_backoff=5)
def send_invoice_paid(self, invoice_id):
    invoice = InvoiceModel.objects.get(id=invoice_id)
    send_email(
        to=invoice.client_email,
        subject="Invoice paid",
    )
```

- **❌ Bu yerda:** status o‘zgartirish yo‘q, biznes qoidalar yo‘q
- **✅ Faqat:** IO, external service, retry

---

## 🔹 6. Django transaction bilan TO‘G‘RI ishlash (CRITICAL)

**❌ XATO:** Celery task DB commit bo‘lmasdan ishga tushadi.

**✅ TO‘G‘RI:**
```python
from django.db import transaction

def execute(...):
    with transaction.atomic():
        ...
        transaction.on_commit(
            lambda: event_bus.publish(event)
        )
```

> **🔥 BU SENIOR-LEVEL NUQTA.** Ko‘p production bug’lar shu yerda bo‘ladi.

---

## 🔹 7. Multiple Side-effectlar (Fan-out)

```python
class EventBus:
    def publish(self, event):
        if isinstance(event, InvoicePaid):
            send_email_task.delay(event.invoice_id)
            notify_admin_task.delay(event.invoice_id)
            generate_pdf_task.delay(event.invoice_id)
```

> **📌 Domain bundan behabar**

---

## 🔹 8. Test strategiya (DDD + Celery)

**✅ Domain test:**
```python
def test_invoice_emits_event():
    invoice.pay(Money(1000))
    assert isinstance(invoice.events[0], InvoicePaid)
```

**✅ Use Case test:** fake repo, fake event bus

**❌ Celery task test = oddiy unit:** mock external API

---

## 🔥 ADVANCED NOTE’LAR (REAL PRODUCTION)

- **⚠️ 1. Celery task** — hech qachon Use Case bo‘lmasin
- **⚠️ 2. Retry** — Domain emas (Infra masala)
- **⚠️ 3. Idempotency** — Celery task bir necha marta kelishi mumkin → side-effect idempotent bo‘lsin
- **⚠️ 4. Event ≠ Signal** — Django signal (yashirin coupling) vs Domain Event (ochiq va testable)

---

## 🧩 QACHON BU ARXITEKTURA SHART?

| Holat | Kerakmi |
| :--- | :--- |
| **Billing / Payment** | ✅ |
| **Email / SMS** | ✅ |
| **PDF generation** | ✅ |
| **Webhook** | ✅ |
| **Oddiy CRUD** | ❌ |

## 🧠 YAKUNIY XULOSA

**DDD + Celery:**
- Domain toza qoladi
- Celery faqat worker
- Testlar oson
- Production bug’lar kamayadi

> **Bu arxitektura: “Katta loyiha uchun majburiy”**
