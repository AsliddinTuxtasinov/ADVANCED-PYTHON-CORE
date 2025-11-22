## ✅ 1.1. Mutable vs Immutable — lekin ADVANCED darajada

Ko‘pchilik faqat shuni biladi:

- **immutable**: `int`, `float`, `str`, `tuple`, `frozenset`
- **mutable**: `list`, `dict`, `set`, `bytearray`

Bu to‘g‘ri, lekin advanced darajada bundan ko‘proq narsa bor.

### 💡 1) Python IMMUTABLE obyektlarni rejuse qiladi (interning)

Ya’ni Python RAMni tejash uchun ba’zi obyektlarni qayta-qayta yaratmaydi — balki yaqin oraliqdagi qiymatlar uchun bitta obyektni ishlatadi.

Misol:

```python
a = 256
b = 256
print(a is b)  # True
```

Lekin:

```python
a = 257
b = 257
print(a is b)  # False
```

> ❗ 0 dan 256 gacha bo‘lgan integerlar Python’da intern qilinadi — ya’ni oldindan RAMda yaratilgan bo‘ladi.

### 💡 2) String'larda ham shunday optimizatsiya bor

```python
a = "hello"
b = "hello"
print(a is b)  # True
```

Ammo dinamik stringlarda:

```python
a = "hello world".replace("world", "python")
b = "hello python"
print(a is b)  # False
```

### 💡 3) Immutable obyektlarni o‘zgartirganda — Python yangi obyekt yaratadi

```python
a = 10
b = a
a = a + 1
print(a, b)  # 11 10
```

Shuning uchun immutable objectlar thread-safe va function parameter sifatida xavfsiz.

### 🔥 MUTABLE obyektlar esa har doim reference orqali ishlaydi

```python
a = [1, 2, 3]
b = a
b.append(4)

print(a)  # [1, 2, 3, 4]
```

Shu sababli:

> Default argument sifatida list va dict ishlatish katta xato

## ✅ 1.2. tuple optimizatsiyasi
🔥 Ko‘pchilik bilmaydi: Python’da tuple – listdan kam RAM yeydi.

Misol:

```python
import sys

print(sys.getsizeof([1,2,3]))   # 88 bytes
print(sys.getsizeof((1,2,3)))   # 72 bytes
```

Nima uchun tuple optimallashgan?

- `tuple` immutable → Python unga qo‘shimcha metadata saqlamaydi
- `list` kengayishi uchun “buffer” saqlaydi
- `tuple` esa exact size

Shuning uchun dictionary keys sifatida list emas, tuple ishlatiladi.

```python
d = {(1,2): "point"}   # ✔ to‘g‘ri
```

## ✅ 1.3. frozenset — mutlaq optimizatsiya

- **Set** — mutable.
- **frozenset** — immutable + hashable.

Foydalanish joylari:

- Cache / memoization keys
- Multi-key dictionary keys
- Deduplication (tezroq ishlaydi)

Misol:

```python
cache = {}
key = frozenset(["user", "premium"])
cache[key] = "cached data"
```

`frozenset` setdan eng kam RAM ishlatadi + tezroq hash bo‘ladi.

## ✅ 1.4. memoryview — PROFESSIONAL LEVEL mavzu

Bu Python’da RAMni zero-copy usulda ishlatish imkoniyatini beradi.

Misol:

```python
data = bytearray(b"hello world")
m = memoryview(data)

m[0] = ord('H')
print(data)  # bytearray(b'Hello world')
```

### 🔥 Zero-copy nima?

Demak:

- ❌ data’ni ko‘chirmaydi
- ✔ faqat RAMdagi bir bo‘lagiga ko‘rsatkich beradi
- ✔ katta fayllar, streaming, binary protocols uchun super qurol

Misol: 10GB faylni parchalashda ultra tez ishlaydi.

## ✅ 1.5. bytes vs bytearray — chuqur farqi

### bytes — immutable

Yaratilgandan keyin o‘zgartirib bo‘lmaydi.

```python
b = b"hello"
# b[0] = 72  # ERROR
```

### bytearray — mutable

Binary data bilan ishlaganda eng yaxshi variant.

```python
b = bytearray(b"hello")
b[0] = 72
print(b)  # bytearray(b'Hello')
```

### Qachon qaysi biri ishlatiladi?

| Vazifa | Optimal type |
| :--- | :--- |
| TCP socket bilan ishlash | `bytearray` |
| Hashing / signatures | `bytes` |
| Memory mapping | `memoryview` + `bytearray` |
| Protobuf / binary protokollar | `bytearray` |