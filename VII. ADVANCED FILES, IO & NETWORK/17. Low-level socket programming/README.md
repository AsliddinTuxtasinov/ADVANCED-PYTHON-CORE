# 17. Low-level Socket Programming (Advanced)

Socketlar qanday ishlashini ichidan tushunish, HTTP, FastAPI, Django, gRPC, WebSocket kabi texnologiyalar nimaning ustiga qurilganini anglash.

Agar sen:
> "HTTP nima o‘zi aslida?"
>
> "Nima uchun asyncio tez?"
>
> "WebSocket nimasi bilan boshqacha?"

degan savollarga chinakam javob xohlasang — bu maqola sen uchun.

## 1. Socket degani nima? (Formal emas, real tushuntirish)

**Socket** — bu ikki process orasidagi past darajadagi aloqa nuqtasi.

Bu processlar:
- bir xil kompyuterda (IPC)
- yoki turli serverlarda (network orqali)
bo‘lishi mumkin.

> [!IMPORTANT]
> HTTP, WebSocket, gRPC — socket ustidagi protokollar, socketning o‘zi emas.

## 2. Network stack: real hayotdagi joyi

```text
Application Layer   → HTTP / WebSocket / FTP
Transport Layer     → TCP / UDP
Internet Layer      → IP
Link Layer          → Ethernet / WiFi
```

> [!NOTE]
> Socket asosan **Transport layer (TCP/UDP)** bilan ishlaydi.

Python’da:
- `socket.SOCK_STREAM` → **TCP**
- `socket.SOCK_DGRAM` → **UDP**

## 3. TCP Socket hayot sikli (MUHIM!)

**Server tomoni:**
1. `socket()` — socket obyekt yaratish
2. `bind()` — IP + portga bog‘lash
3. `listen()` — ulanishlarni kutish
4. `accept()` — clientni qabul qilish
5. `recv() / send()`
6. `close()`

**Client tomoni:**
1. `socket()`
2. `connect()`
3. `send() / recv()`
4. `close()`

> [!WARNING]
> Agar shu ketma-ketlikni bilmasang — framework seni aldab yuribdi.

## 4. Minimal TCP server (raw, frameworksiz)

```python
import socket

server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
server.bind(("0.0.0.0", 9000))
server.listen()

print("Server listening...")

conn, addr = server.accept()
print("Client connected:", addr)

data = conn.recv(1024)
print("Received:", data.decode())

conn.send(b"Hello from server")
conn.close()
```

> [!NOTE]
> **Bu kod:**
> - Blocking
> - 1 ta client
> - Production uchun **YARAMAS**

Lekin Frameworklar aynan shuni avtomatlashtiradi.

## 5. Nima uchun bu kod yomon?

**Muammolar:**
- ❌ Har bir client blocking
- ❌ 1000 client bo‘lsa — 1000 thread kerak
- ❌ Slow client → butun server sekinlashadi

Shu yerda **IO muammosi** paydo bo‘ladi.

## 6. Blocking vs Non-blocking socket

| Turi | Kod | Izoh |
| :--- | :--- | :--- |
| **Blocking** (default) | `data = conn.recv(1024)` | Data kelmaguncha kutadi |
| **Non-blocking** | `conn.setblocking(False)` | Kutmaydi, xato beradi yoki o'tib ketadi |

Ammo:
- Non-blocking → **busy loop**
- CPU **100%**

Shu sabab: **Multiplexing** paydo bo‘ladi.

## 7. Multiplexing: select / poll / epoll

### select()
- Eski
- 1024 FD limit
- Cross-platform
- Python: `select.select(reads, writes, errors)`

### poll()
- Yaxshiroq
- Linux-friendly

### epoll (Linux)
- Juda tez
- Event-driven
- **asyncio asosida**

> [!TIP]
> **asyncio:** event loop → epoll
>
> **FastAPI tezligining siri shu yerda.**

## 8. UDP socket (qachon kerak?)

**UDP:**
- Connection yo‘q
- Paket yo‘qolishi mumkin
- Juda tez

**Qo‘llanilishi:**
- Video streaming
- Online o‘yinlar
- Metrics (StatsD)

```python
sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
sock.sendto(b"ping", ("localhost", 9001))
```

> [!NOTE]
> - HTTP → TCP
> - DNS → UDP
> - QUIC → UDP + crypto

## 9. HTTP aslida nima?

HTTP request — bu oddiy text:

```http
GET / HTTP/1.1
Host: example.com
```

**FastAPI, Django:**
1. socketni o‘qiydi
2. HTTP ni parse qiladi
3. response yasaydi

> [!IMPORTANT]
> Framework emas, socket ishlayapti.

## 10. WebSocket nima uchun boshqacha?

**HTTP:**
- Request → Response → yopiladi

**WebSocket:**
- Upgrade
- Socket ochiq qoladi
- Full-duplex

**Shuning uchun:**
- Chat
- Real-time dashboard
- Notification

## 11. Async socketlar: asyncio asoslari

```python
await loop.sock_recv(sock, 1024)
```

Bu yerda:
- Thread yo‘q
- epoll ishlayapti
- 10k+ connection mumkin

> [!TIP]
> FastAPI / Starlette → async socketlar

## 12. Real backendda qayerda kerak bo‘ladi?

**Low-level socket bilimi:**
- Reverse proxy (NGINX tushunish)
- Custom protocol
- High-load service
- WebSocket debugging
- Timeout / keep-alive tushunish

**Advanced savol:**
> “Nima uchun request osilib qoladi?”

**Junior javob:**
> “Bilmayman”

**Advanced javob:**
> “Socket read blok bo‘lib qoldi”

## 13. Xulosa (Advanced mindset)

- **Socket** — hamma network texnologiyaning ildizi
- **Frameworklar** — qulaylik, bilim emas

**Low-level bilgan odam:**
- Muammoni tez topadi
- Frameworkga kamroq qaram bo‘ladi
- Arxitekturani to‘g‘ri quradi

---

# 🔥 ADVANCED SOCKETS — AMALIY SERIYA (Advanced level)

**Tavsiya etilgan tartib (NEGA aynan shunday):**

1. **asyncio + socket**
   - event loop, epoll, concurrency tushuniladi
   - FastAPI/Django async nega ishlashini anglaysan
2. **HTTP serverni noldan yozish**
   - framework ichida nima bo‘layotgani ochiladi
   - middleware, request parsing, response lifecycle
3. **WebSocket protocol ichki ishlashi**
   - Upgrade, frame, ping/pong
   - chat, realtime systemlarni to‘g‘ri dizayn qilish
4. **TCP handshake & TIME_WAIT muammosi**
   - production muammolar
   - “port band”, “connection leak” sabablarini tushunish

---

## 🔥 1. asyncio + socket (Deep Dive)

### ❓ Nega oddiy socket yetmaydi?

**Oddiy (blocking) socket:**
- 1 socket = 1 thread
- 10 000 client → 10 000 thread ❌
- RAM + context switch = **o‘lim**

**👉 asyncio:**
- 1 thread
- 1 event loop
- 10 000+ socket ✔️

### 🧠 Asosiy g‘oya

> “Kutish paytida CPU bekor turmasin”

**IO:** network, disk, db

**asyncio:** kutayotgan socketni tashlab, tayyor socketga o‘tadi.

### 1.1 Event Loop nima o‘zi?

**Event loop:**
- tayyor bo‘lgan socketlarni kuzatadi
- epoll (Linux) ishlatadi
- callback / coroutine’larni ishga tushiradi

```python
while True:
    ready_sockets = epoll.wait()
    for sock in ready_sockets:
        resume_coroutine(sock)
```

> [!NOTE]
> Bu thread emas, bu **state machine**.

### 1.2 asyncio socket qanday ishlaydi?

**Oddiy socket:**
```python
data = conn.recv(1024)  # BLOCK
```

**asyncio:**
```python
data = await loop.sock_recv(conn, 1024)  # NON-BLOCK
```

**Farq:**
- `await` → “hozircha chiqib tur”
- event loop boshqa socketlarga o‘tadi

### 1.3 Minimal asyncio TCP server (RAW)

```python
import asyncio
import socket

async def handle_client(conn, addr, loop):
    print("Connected:", addr)
    data = await loop.sock_recv(conn, 1024)
    print("Received:", data.decode())

    await loop.sock_sendall(conn, b"Hello async client")
    conn.close()

async def main():
    loop = asyncio.get_running_loop()

    server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    server.bind(("0.0.0.0", 9000))
    server.listen()
    server.setblocking(False)

    print("Async server listening...")

    while True:
        conn, addr = await loop.sock_accept(server)
        asyncio.create_task(handle_client(conn, addr, loop))

asyncio.run(main())
```

**🔍 Bu yerda nima bo‘lyapti?**
- `setblocking(False)` → **muhim!**
- `sock_accept` → event-based
- `create_task` → concurrency
- 👉 **1 thread, cheksiz client**

### 1.4 Nima uchun bu model kuchli?

| Model | Natija |
| :--- | :--- |
| **Thread-per-connection** | sekin, qimmat |
| **Async + epoll** | tez, arzon |
| **FastAPI** | aynan shu |

> [!TIP]
> **FastAPI = async socket + HTTP parser**

### 1.5 asyncio’da eng ko‘p qilinadigan xato

❌ **Blocking kodni async ichida yozish:**
```python
time.sleep(5)  # ❌ event loop o‘ladi
```

✔️ **To‘g‘risi:**
```python
await asyncio.sleep(5)
```

> [!WARNING]
> Bitta blocking funksiya → butun server osiladi

### 1.6 asyncio + socket qayerda real ishlatiladi?

- FastAPI / Starlette
- WebSocket server
- Custom protocol
- High-load proxy
- TCP gateway

**Advanced savol:**
> “Nega async serverim sekin?”

**Advanced javob:**
> “Ichida blocking IO bor”

### 1.7 Mental model (eslab qol!)

- **Thread** = parallel CPU
- **Async** = parallel IO
- **asyncio** = socket scheduler
- **epoll** = kernel yordamchisi

---

## 🔜 Keyingi qadam

Agar tayyor bo‘lsang, keyingi darsda:

### 🔥 2. HTTP serverni noldan yozish
- raw socket → HTTP parse
- request line
- headers
- response
- keep-alive

> [!TIP]
> Shundan keyin **FastAPI** juda oddiy ko‘rinadi.
