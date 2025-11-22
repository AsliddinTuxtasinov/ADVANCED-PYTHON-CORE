## 🧠 3. Shallow Copy vs Deep Copy (Advanced tushuntirish)

🔥 Eng muhimi: Python obyektlari reference orqali ishlaydi.

Shuning uchun copy qilish — oddiy “ko‘chirish” emas. Bu RAM pointer bilan bog‘liq masala.

## ✅ 3.1. Shallow Copy (Yuzaki ko‘chirish)

Shallow copy → faqat tashqi obyektni ko‘chiradi, ichidagi ichki obyektlarga pointer qo‘yadi.

Misol:

```python
import copy

a = [1, [2, 3]]
b = copy.copy(a)
```

Diagram:

```
 a ---> [1,  [2,3]]
             ↑
 b ----------┘   # UShBU [2,3] IKKALASIGA BIRXIL POINTER
```

Natija:

```python
b[1].append(4)
print(a)   # [1, [2,3,4]]
print(b)   # [1, [2,3,4]]
```

Ichki obyektlar deep copy bo‘lmaydi.

- Shallow copy → juda tez
- Deep copy → sekinroq

## ✅ 3.2. Deep Copy (Chuqur ko‘chirish)

Deep copy → ichkaridagi barcha obyektlarni ham ko‘chiradi.

```python
b = copy.deepcopy(a)
```

Diagram:

```
 a ---> [1,  [2,3]]
 
 b ---> [1,  [2,3]]   # ICHKI LIST HAM YANGI OBYEKT
```

Endi:

```python
b[1].append(4)
print(a)  # [1, [2,3]]
print(b)  # [1, [2,3,4]]
```

### ⚠️ Deep copy — xavfli bo'lishi mumkin

Agar obyekt juda katta bo‘lsa:

- deep copy → juda sekin
- recursion chuqur bo‘lsa → stack overflow
- DB connectionlar, locks, file handles → copy qilib bo‘lmaydi

Shuning uchun Python copy protokoli mavjud.

## 🏆 3.3. Copy Protokoli: __copy__ va __deepcopy__

Real SENIOR lar shuni bilishi kerak:

Copy qilish jarayonini o‘zing nazorat qilishing mumkin.

### 🔥 1) __copy__(self) — shallow copy’ni boshqarish

```python
class User:
    def __init__(self, name, roles):
        self.name = name
        self.roles = roles

    def __copy__(self):
        print("Shallow copy ishladi!")
        return User(self.name, self.roles)   # roles pointer
```

### 🔥 2) __deepcopy__(self, memo) — deep copy’ni boshqarish

```python
class User:
    def __init__(self, name, roles):
        self.name = name
        self.roles = roles

    def __deepcopy__(self, memo):
        print("Deep copy ishladi!")
        return User(
            copy.deepcopy(self.name, memo),
            copy.deepcopy(self.roles, memo),
        )
```

`memo` — infinite recursionni oldini oladi (cycle obyektlarda).

## 🧨 3.4. Real life SENIOR misol — Config object

Oddiy copy ishlamaydi:

```python
class Config:
    def __init__(self):
        self.db = {"host": "localhost"}
        self.cache = {"redis": True}

config = Config()

new_config = copy.deepcopy(config)
```

Lekin yangi config’da DB connection deep copy bo‘lmasligi kerak.

Shuning uchun SENIOR yechim:

```python
class Config:
    def __deepcopy__(self, memo):
        new_cfg = Config()
        new_cfg.db = self.db  # shallow
        new_cfg.cache = copy.deepcopy(self.cache, memo)
        return new_cfg
```

Mana SENIOR-level deep copy 🔥

## 🧨 3.5. Django ORM obyektlari copy qilinmaydi

Bu juda muhim!

Django model instance deep copy qilinsa:

- primary key copy bo‘ladi → uniqueness buziladi
- relationlar buziladi

Shuning uchun Django modelda:

- `__copy__` — shallow, safe
- `__deepcopy__` — forbidden yoki custom

## 🧨 3.6. FastAPI Pydantic model copy

Pydantic (V2) da:

```python
u2 = user.model_copy(deep=True)
```

Bu deep copy sifatida ishlaydi — ammo validation va metadata kamaytiriladi.

## 🧠 Yakuniy xulosa

| Konsept | Ma’nosi |
| :--- | :--- |
| Shallow copy | Faqat tashqi obyekt ko‘chadi, ichkarisi shared pointer |
| Deep copy | Hammasi to‘liq ko‘chadi |
| `__copy__` | Shallow copy’ni custom boshqarish |
| `__deepcopy__` | Deep copy’ni custom boshqarish |
| `memo` | Cyclic objectlarda recursionni oldini oladi |