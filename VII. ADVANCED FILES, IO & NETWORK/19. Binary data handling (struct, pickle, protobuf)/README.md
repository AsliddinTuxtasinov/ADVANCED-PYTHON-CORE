# 19. Binary Data Handling (struct, pickle, protobuf)

## 1. Binary data degani nima?

**Oddiy qilib aytsak:**
Binary data — bu odam o‘qiy olmaydigan, lekin kompyuter juda tez tushunadigan ma’lumot.

**Misol:**
- JSON → text (odam o‘qiydi)
- Binary → `01010101` (faqat mashina uchun)

> [!NOTE]
> Network, file, protocol’larning **80%** binary ishlatadi.

## 2. Nega JSON har doim yetarli emas?

**JSON:**
- o‘qish oson
- debug qilish qulay

**Lekin:**
- hajmi katta
- parsing sekin
- tiplar noaniq (number hamma narsa)

**Misol:**
```json
{"id":123,"active":true}
```

**Binary’da:**
- aniq format
- kam joy
- tez parse

> [!WARNING]
> High-load tizimlarda **JSON — muammo**.

## 3. Binary data bilan ishlashning 3 yondashuvi

**Python’da 3 asosiy yo‘l bor:**
1. **struct** → past daraja, maksimal nazorat
2. **pickle** → Python obyektlarini saqlash
3. **protobuf** → network & microservice standarti

## 4. struct — eng past daraja (C-ga yaqin)

**Nima uchun struct kerak?**
 - Protocol yozish
 - Binary file format
 - Embedded / low-level system
 - TCP/UDP payload

**Asosiy g‘oya:**
> “Ma’lumot xotirada qanday yotsa, shunday yoz”

```python
import struct

data = struct.pack("i f ?", 10, 3.14, True)
print(data)
```

**Bu yerda:**
- `i` → int (4 byte)
- `f` → float (4 byte)
- `?` → bool (1 byte)

> [!NOTE]
> **Natija:** 9 byte, JSON’da esa ~30 byte bo‘lardi.

**Unpack qilish:**
```python
values = struct.unpack("i f ?", data)
print(values)
```

**❗ struct’ning kamchiligi:**
- Readability yo‘q
- Versioning qiyin
- Manual format

> [!TIP]
> Lekin tezlik va nazorat **maksimal**.

## 5. pickle — Python obyektlarini “muzlatish”

**pickle nima qiladi?**
Python obyektini to‘liq binary holatga keltiradi.

```python
import pickle

data = {"id": 1, "roles": ["admin", "user"]}
blob = pickle.dumps(data)
```

**Va qayta tiklash:**
```python
obj = pickle.loads(blob)
```
> [!NOTE]
> Juda qulay, lekin…

## 6. pickle — eng xavfli joyi ⚠️

> [!IMPORTANT]
> ❌ pickle — ishonchsiz data uchun **YARAMAYDI**

**Sababi:**
- `loads()` vaqtida kod ishlashi mumkin
- RCE (Remote Code Execution) xavfi

**👉 Faqat:**
- internal cache
- trusted storage
- local file
**uchun.**

> [!WARNING]
> ❌ Network orqali hech qachon.

## 7. pickle qachon ishlatiladi?

✅ **Mos holatlar:**
- Celery cache
- ML model saqlash
- Python-only microservice

❌ **Mos emas:**
- Public API
- Client-server protocol
- Long-term storage

## 8. protobuf — production-grade yechim

**protobuf nima?**
Google tomonidan yaratilgan, strict, binary serialization formati.

**Xususiyatlari:**
- juda ixcham
- juda tez
- tilga bog‘liq emas
- versioning bor

> [!TIP]
> gRPC, microservice, distributed system’larning asosi.

## 9. protobuf qanday ishlaydi?

**1. Schema yoziladi (.proto)**
```protobuf
syntax = "proto3";

message User {
  int32 id = 1;
  string name = 2;
  bool active = 3;
}
```

**2. Kod generatsiya qilinadi**
```bash
protoc --python_out=. user.proto
```

**3. Python’da ishlatish**
```python
user = User(id=1, name="Ali", active=True)
data = user.SerializeToString()
```

## 10. protobuf’ning kuchli tomonlari

| Feature | Sabab |
| :--- | :--- |
| **Schema** | aniq tip |
| **Backward compatibility** | eski client sinmaydi |
| **Binary size** | juda kichik |
| **Speed** | juda tez |

> [!NOTE]
> **JSON** → qulay
>
> **Protobuf** → professional

## 11. struct vs pickle vs protobuf (Advanced taqqoslash)

| Feature | struct | pickle | protobuf |
| :--- | :--- | :--- | :--- |
| **Level** | very low | medium | high |
| **Speed** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| **Safety** | ⭐⭐⭐⭐ | ❌ | ⭐⭐⭐⭐⭐ |
| **Versioning** | ❌ | ❌ | ✔️ |
| **Cross-language** | ❌ | ❌ | ✔️ |

## 12. Qaysi biri qachon?

**Advanced qaror jadvali:**

| Maqsad | Tavsiya |
| :--- | :--- |
| Protocol yozyapsan | **struct** |
| Python obyekt saqlayapsan | **pickle** |
| Microservice / network | **protobuf** |

## 13. Real backend misollar

- **Redis protocol** → binary
- **Kafka** → binary
- **gRPC** → protobuf
- **Image / video** → binary stream
- **IoT** → struct + binary

## 14. Mental model (eslab qol)

- **Binary** → tezlik + kam joy
- **struct** → nazorat
- **pickle** → qulaylik (lekin xavf)
- **protobuf** → production standart

## ✅ Xulosa (Advanced mindset)

- **Binary data** — performance kaliti
- **JSON** — default, lekin doim to‘g‘ri emas
- **pickle** — faqat ishonchli joyda
- **protobuf** — katta tizimlar tanlovi

> [!TIP]
> **Advanced developer** format tanlaydi, **junior** esa default’ni.