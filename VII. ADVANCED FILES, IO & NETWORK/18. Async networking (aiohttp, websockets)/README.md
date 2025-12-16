# 18. Async Networking (aiohttp, websockets)

## 1. Async networking degani nima o‘zi?

**Oddiy qilib aytganda:**
Async networking — bu network IO paytida CPU’ni bekor o‘tirg‘izmaslik san’ati.

**Network IO:**
- HTTP request
- WebSocket message
- DB connection
- Redis call

**Bularning hammasi:**
- Kutish
- CPU ishlamaydi

> [!NOTE]
> ❌ **Sync modelda** → CPU kutadi
>
> ✅ **Async modelda** → CPU boshqa ishga o‘tadi

## 2. Blocking vs Async networking (real farq)

### Blocking (klassik model)
```python
response = requests.get(url)  # shu yerda thread kutib turadi
```
- 1 request = 1 thread
- 1000 request = 1000 thread ❌

### Async networking
```python
response = await session.get(url)
```
- 1 thread
- 1000 request
- event loop boshqaradi ✔️

> [!TIP]
> **Farq kodda emas — arxitekturada.**

## 3. aiohttp nima uchun yaratilgan?
**aiohttp — bu:**
- async HTTP client
- async HTTP server
- async socket + HTTP parser

**aiohttp qiladigan ishlar:**
- connection pooling
- keep-alive
- timeout
- streaming
- backpressure

> [!WARNING]
> Bularni o‘zing yozsang → **2000+ qator kod**

## 4. aiohttp Client — ichidan qaraymiz

**Oddiy foydalanish:**
```python
async with aiohttp.ClientSession() as session:
    async with session.get(url) as response:
        data = await response.text()
```

**Aslida nima bo‘lyapti?**
1. socket ochiladi
2. TCP connection reuse qilinadi
3. event loop orqali IO boshqariladi
4. response chunk-chunk o‘qiladi

> [!NOTE]
> **ClientSession:**
> - har request uchun socket ochmaydi
> - connection pool saqlaydi

## 5. aiohttp va performance (nega tez?)

| Feature | Sabab |
| :--- | :--- |
| **Async IO** | Thread yo‘q |
| **Connection pooling** | TCP handshake kam |
| **Streaming** | RAM kam ishlaydi |
| **Backpressure** | Server yiqilmaydi |

**Shu sabab:**
- crawler
- payment gateway
- microservice → microservice call
**uchun juda mos.**

## 6. aiohttp Server (FastAPI’ga o‘xshash, lekin pastroq)
```python
from aiohttp import web

async def hello(request):
    return web.Response(text="Hello async")

app = web.Application()
app.add_routes([web.get('/', hello)])

web.run_app(app)
```

> [!TIP]
> **FastAPI:**
> - bundan ham yuqori abstraction
> - ammo ostida xuddi shu model

## 7. WebSocket — async networkingning real kuchi

**HTTP:**
- request → response → yopiladi

**WebSocket:**
- doimiy ochiq connection
- client ↔ server real-time

**Bu degani:**
- chat
- notification
- live dashboard

## 8. WebSocket async modelda nega ideal?

**WebSocket:**
- ko‘p client
- kam data
- doimiy aloqa

> [!NOTE]
> **Sync model:** har client thread ❌
>
> **Async model:**
> - event loop
> - socket tayyor bo‘lsa → o‘qiysan
> - bo‘lmasa → boshqasiga o‘tasan ✔️

## 9. websockets library (minimal tushuncha)
```python
import websockets
import asyncio

async def handler(ws):
    async for message in ws:
        await ws.send(f"Echo: {message}")

async def main():
    async with websockets.serve(handler, "0.0.0.0", 9000):
        await asyncio.Future()

asyncio.run(main())
```

**Bu kod nimani yashiryapti?**
- HTTP Upgrade
- Frame parsing
- Ping/Pong
- Close handshake

> [!IMPORTANT]
> Barchasi **async socket ustida.**

## 10. WebSocket vs HTTP (Advanced jadval)

| Feature | HTTP | WebSocket |
| :--- | :--- | :--- |
| **Connection** | qisqa | doimiy |
| **Direction** | client → server | ikki tomonlama |
| **Latency** | yuqori | past |
| **Async mosligi** | o‘rtacha | ideal |

## 11. Async networkingdagi eng katta xatolar

❌ **Async ichida blocking kod:**
```python
requests.get(url)
time.sleep(3)
```

❌ **CPU-heavy ishni async’da qilish**

✔️ **To‘g‘ri yo‘l:**
- IO → async
- CPU → thread/process

## 12. Qachon async networking kerak EMAS?

❌ **Agar:**
- request kam
- CPU-heavy
- oddiy CRUD

> [!WARNING]
> Async — hamma joyga qo‘yiladigan narsa emas.

**Advanced qaror:**
> “Bu yerda async kerakmi yoki yo‘qmi?”

## 13. Mental model (yodda saqla)

- **aiohttp** → async HTTP
- **websockets** → async real-time
- **asyncio** → IO scheduler
- **event loop** → socket dispatcher

## ✅ Xulosa

- **Async networking** — yuqori yuklama uchun
- **aiohttp** — HTTP’ni bosh og‘riqsiz qiladi
- **WebSocket** — real-time tizimlarning asosi
- **Frameworklar** — faqat qulay qobiq

> [!TIP]
> **Asosini bilgan odam frameworkga qaram bo‘lmaydi.**
