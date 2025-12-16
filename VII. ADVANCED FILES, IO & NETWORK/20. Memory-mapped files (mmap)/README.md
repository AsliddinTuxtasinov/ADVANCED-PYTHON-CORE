# 20. Memory-Mapped Files (mmap)

## 1. Muammo: oddiy file I/O qayerda yetmaydi?

**Oddiy file o‘qish:**
```python
with open("data.bin", "rb") as f:
    data = f.read()
```

**Bu yerda:**
- fayl to‘liq RAM’ga yuklanadi
- katta fayl → katta RAM
- random access sekin

> [!WARNING]
> ❌ 10 GB fayl
>
> ❌ faqat 10 MB kerak
>
> 👉 **resurs isrofi**

## 2. mmap g‘oyasi (oddiy qilib)

**Faylni RAM’ga ko‘chirma, RAM’ga o‘xshat.**

**mmap:**
- faylni **virtual memory** ga bog‘laydi
- OS o‘zi qaysi sahifa kerakligini yuklaydi
- sen uni oddiy **byte array** deb ishlatasan

> [!NOTE]
> Disk ↔ RAM o‘rtasidagi koprik — **OS**

## 3. mmap qanday ishlaydi? (ichki mexanizm)

1. Fayl ochiladi
2. OS **virtual address space** ga map qiladi
3. Sahifalar (page, odatda 4KB):
    - kerak bo‘lsa RAM’ga kiradi
    - kerak bo‘lmasa chiqariladi

> [!TIP]
> 👉 **Lazy loading**

**Shuning uchun:**
- katta fayl
- kam RAM
- tez random access
**mumkin.**

## 4. Python’da minimal mmap misoli

```python
import mmap

with open("data.bin", "rb") as f:
    mm = mmap.mmap(f.fileno(), 0, access=mmap.ACCESS_READ)

    print(mm[:100])   # birinchi 100 byte
    print(mm.find(b"magic"))
    
    mm.close()
```

> [!NOTE]
> - `0` → butun fayl
> - `slicing` → disk emas, memory view

## 5. mmap vs read() — real farq

| Feature | read() | mmap |
| :--- | :--- | :--- |
| **RAM ishlatish** | yuqori | minimal |
| **Random access** | sekin | tez |
| **Startup time** | sekin | tez |
| **OS optimizatsiya** | yo‘q | bor |
| **Katta fayl** | muammo | ideal |

> [!TIP]
> 👉 **Katta fayl = mmap**

## 6. Writeable mmap (o‘ta ehtiyot!)

```python
with open("data.bin", "r+b") as f:
    mm = mmap.mmap(f.fileno(), 0)
    mm[0:4] = b"DATA"
    mm.flush()
    mm.close()
```

**📌 Bu:**
- faylni joyida o‘zgartiradi
- rollback yo‘q
- crash → data corrupt

> [!WARNING]
> ⚠️ **Transaction yo‘q → ehtiyot bo‘l**

## 7. Real hayotda qayerda ishlatiladi?

- Database engine (SQLite)
- Search engine (Lucene)
- Log analyzer
- Video / media processing
- Binary index file

> [!TIP]
> 👉 **High-performance system’lar**

## 8. mmap + struct = kuchli kombinatsiya

**Binary record o‘qish:**

```python
import struct

record_size = struct.calcsize("i d")
offset = record_id * record_size

record = mm[offset : offset + record_size]
user_id, balance = struct.unpack("i d", record)
```

> [!NOTE]
> - Disk yo‘q, RAM copy yo‘q
> - O‘ta tez random access

## 9. mmap va multiprocessing

**Juda muhim joy 👇**
**mmap:**
- processlar orasida shared memory
- IPC uchun ideal

**Misol:**
- 5 worker
- 1 katta fayl
- barchasi bir xil mmap

> [!TIP]
> 👉 **RAM tejaladi**

## 10. mmap + asyncio

**📌 mmap — sync**

**Lekin:**
- disk IO OS’da
- asyncio event loop bloklanmaydi

**Shuning uchun:**
- async server + mmap → **OK**
- read() → **IO blok**

## 11. mmap qachon YARAMAYDI?

❌ **Kichik fayl**
❌ **Sequential read**
❌ **Ko‘p write**
❌ **Transaction kerak**

**Bu holatda:**
- oddiy read()
- buffered IO
**yaxshiroq.**

## 12. Xavflar va ehtiyot choralar

- ⚠️ Platformaga bog‘liq xatti-harakatlar
- ⚠️ Windows vs Linux farqi
- ⚠️ File truncate → segmentation fault
- ⚠️ Concurrency write → race condition

> [!IMPORTANT]
> 👉 mmap — qurol, o‘yinchoq emas.

## 13. Mental model (yodda saqla)

- **mmap** = file → memory
- copy yo‘q
- OS boshqaradi
- tezlik → OS page cache

## ✅ Xulosa (Advanced mindset)

- **mmap** — katta fayl + random access
- OS kuchidan foydalanish
- RAM’ni tejash
- noto‘g‘ri ishlatsang — data yo‘qoladi

> [!TIP]
> 👉 **Senior** mmap qachon kerak emasligini ham biladi.