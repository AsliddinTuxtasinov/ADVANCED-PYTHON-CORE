# 21. Unit & Integration Testing, Advanced Mocking, Patch Techniques, Dependency Overriding (FastAPI)

## 0️⃣ Asosiy mindset (eng muhim qoidalar)

> **Test — bu kodni tekshirish emas, dizaynni tekshirish.**

Agar test yozish qiyin bo‘lsa:
- ❌ bu test muammosi emas
- ✅ bu architecture muammosi

**Clean Architecture + to‘g‘ri dependency boundary → testlar osonlashadi.**

---

## 1️⃣ Unit Testing — “Isolation is king”

### 🎯 Maqsad
- **1 ta funksiya / class**
- **Tashqi dunyosiz** (DB, HTTP, Redis, FS yo‘q)
- **Tez** (millisekundlar)

### Unit test nimani tekshiradi?
- Business rule
- Edge case
- Error handling
- State change

### ❌ Yomon unit test belgisi
```python
def test_create_user():
    db = Session()
    user = create_user(db, "a@b.com")
```
👉 Bu unit emas, bu integration.

### ✅ To‘g‘ri unit test
```python
def test_price_with_discount():
    calc = PriceCalculator(discount=0.1)
    assert calc.final_price(100) == 90
```

> [!NOTE]
> **Advanced Note:**
> - Unit test frameworkni emas, biznesni tekshiradi
> - Django ORM / FastAPI — bu infrastructure, unit testga kirmaydi

---

## 2️⃣ Integration Testing — “Boundaries are real”

### 🎯 Maqsad
- 2–3 komponent birga ishlashini tekshirish
- DB, HTTP, Redis bo‘lishi mumkin
- Ammo external service emas

### Real integration misollar:
- Repository ↔ Database
- API endpoint ↔ Service ↔ Repository
- Migration ↔ Model

```python
def test_create_user_api(client):
    resp = client.post("/users", json={"email": "a@b.com"})
    assert resp.status_code == 201
```

> [!TIP]
> **Advanced Note:**
> - Integration testlar environmentga bog‘liq
> - Docker + test DB ishlatish ideal
> - Integration test = contract tekshiruvi

---

## 3️⃣ Mocking — Advanced darajada

### Mock qachon kerak?
- External API
- Payment gateway
- Email / SMS
- Time / UUID / random

> [!WARNING]
> ❗ **Mock — ehtiyotkorlik bilan**
> “Mock ko‘paygan sari test yolg‘onlashadi”

### 3.1 Mock vs Fake vs Stub

| Type | Tavsif |
| :--- | :--- |
| **Mock** | Xatti-harakatni tekshiradi |
| **Stub** | Oldindan javob beradi |
| **Fake** | Yengil real implementatsiya |

> **Advanced rule:**
> - Business logic → **fake**
> - External IO → **mock**

### 3.2 unittest.mock.Mock (advanced usage)

```python
from unittest.mock import Mock

repo = Mock()
repo.get_user.return_value = User(id=1)

service = UserService(repo)
service.process(1)

repo.get_user.assert_called_once_with(1)
```

> **📌 Bu yerda:**
> - DB yo‘q
> - Service isolationda test qilinyapti

---

## 4️⃣ Patch techniques — eng ko‘p xato qilinadigan joy

### ⚠️ Asosiy qoida
> **Patch — foydalanilgan joyda qilinadi, e’lon qilingan joyda emas**

#### ❌ Noto‘g‘ri
```python
@patch("services.email.send_email")
```

#### ✅ To‘g‘ri
```python
@patch("user_service.send_email")
```

> **📌 Sabab:**
> - Python import — bu name binding
> - Patch name qayerda ishlatilgan bo‘lsa, o‘sha yerda qilinadi

### 4.1 Context manager patch
```python
with patch("service.uuid4", return_value="fixed"):
    ...
```

> **📌 Bu approach:**
> - Scope nazoratda
> - Parallel testlarda xavfsiz

---

## 5️⃣ Dependency Overriding — FastAPI’da SUPER MUHIM

FastAPI’ning eng kuchli joyi — **dependency injection**.

### Real dependency
```python
def get_db():
    return SessionLocal()
```

### Endpoint
```python
@app.get("/users")
def users(db=Depends(get_db)):
    ...
```

### 5.1 Testda override qilish
```python
def override_get_db():
    return FakeSession()

app.dependency_overrides[get_db] = override_get_db
```

> **📌 Advanced afzalliklar:**
> - DB kerak emas
> - Test tez
> - Full API flow tekshiriladi

### 5.2 Dependency override = Clean Architecture indicator

Agar:
- Dependency override qilish oson bo‘lsa → **architecture yaxshi**
- Qiyin bo‘lsa → **tightly coupled design**

---

## 6️⃣ Clean Architecture + Testing

### Layerlar:
1. **Domain (Entities, Rules)** → Unit tests
2. **Application (Use cases)** → Unit / Fake tests
3. **Infrastructure (DB, HTTP)** → Integration tests
4. **Delivery (API, CLI)** → Integration / Contract tests

> **📌 Rule:**
> Yuqori layerlar pastkini bilmaydi

---

## 7️⃣ Test Pyramid (real hayotda)

```
        E2E (kam)
     Integration (o‘rtacha)
 Unit Tests (ko‘p)
```

> **📌 Ideal nisbat:**
> - 70% unit
> - 20% integration
> - 10% e2e

---

## 8️⃣ Advanced Notes (Senior checklist)

- ❗ Testlar parallel ishlay oladimi?
- ❗ Global state bormi?
- ❗ Time / random controllablemi?
- ❗ Test environment deterministicmi?
- ❗ Dependency override ishlayaptimi?

---

## 9️⃣ Yakuniy xulosa

- **Test** — bu design feedback
- **Mock** — dori, lekin ko‘p ichilsa zahar
- **Patch** — faqat to‘g‘ri joyda
- **FastAPI dependency override** — professional daraja belgisi
- **Clean Architecture** → testlar o‘z-o‘zidan toza bo‘ladi