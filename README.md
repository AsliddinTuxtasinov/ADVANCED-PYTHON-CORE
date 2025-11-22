## ✅ I. PYTHON FOUNDATIONS (Advanced darajada qayta ko‘rib chiqish)

### [1. Advanced Data Types](I.%20PYTHON%20FOUNDATIONS%20%28Advanced%20darajada%20qayta%20ko%E2%80%98rib%20chiqish%29/1.%20Advanced%20Data%20Types/README.MD)
- Mutable vs Immutable (interning, memory optimization)
- tuple optimizatsiyasi, frozenset, memoryview
- bytes vs bytearray

### [2. Object Model:](I.%20PYTHON%20FOUNDATIONS%20%28Advanced%20darajada%20qayta%20ko%E2%80%98rib%20chiqish%29/2.%20Object%20Model/README.MD)
- id(), type(), __class__, __dict__, __slots__
- Python memory model (stack → heap → reference counting)
- Garbage Collector: generations, cyclic GC

### [3. Deep Copy vs Shallow Copy](I.%20PYTHON%20FOUNDATIONS%20%28Advanced%20darajada%20qayta%20ko%E2%80%98rib%20chiqish%29/3.%20Deep%20Copy%20vs%20Shallow%20Copy/README.MD)
- Copy protokoli (__copy__, __deepcopy__)

## II. ADVANCED OOP & DESIGN

### 4. Mixin Architecture
- Django va FastAPI loyihalaringda ishlatish
- Multiple inheritance safe patternlar

### 5. Descriptor Protocol
- __get__, __set__, __delete__
- Django ORM descriptorlarini tushunish
- CachedProperty, property descriptorlar

### 6. Metaclasses — Real Use
- Class yaratilishini boshqarish
- Django Model metaclass qanday ishlaydi?
- FastAPI Pydantic model generation

### 7. Abstract Base Classes (ABC)
- abc.ABC, abstractmethod
- Interface segregationni Python usulida qilish

## III. FUNCTIONAL PYTHON

### 8. Closures & advanced scopes
- LEGB rule — compiled bytecode orqali tushuntirish
- Closure’larning real backend ishlanmasi (cache, factory, config)

### 9. Decoratorlar — advanced
- Decorator inside decorator (parameter decorator)
- Class decorator
- Method decorator
- Funksional middleware patternlari

### 10. Generatorlar va Iteratorlar
- Custom iteratorlar
- Generator pipelines
- Yield from
- Infinite streamlar

## IV. PYTHON RUNTIME & PERFORMANCE

### 11. CPython Interpreter ichki ishlashi
- Bytecode
- dis moduli bilan tahlil
- GIL — real tushuntirish
- Concurrency model: threading vs multiprocessing

### 12. Memory Optimization
- sys.getsizeof()
- Object allocation
- __slots__ bilan RAM optimizatsiya
- C-extensionlar

### 13. Profiling
- cProfile, line_profiler, py-spy
- Bottleneck aniqlash
- Request-level profiling (FastAPI & Django)

## V. ADVANCED ASYNC PYTHON

### 14. Event Loop deep dive
- Tasks
- Future
- Awaitable protokoli

### 15. Async context managers
- async with protokoli
- Connection pooling, Redis, DB sesiyalarida

### 16. Asyncio internals
- gather, wait, shield, semaphore, lock
- Asyncio cancellation
- Task groups (asyncio.TaskGroup)

## VI. DESIGN PATTERNS (SENIOR DARADA)
> 🔥 Muhimi: Pythonic tarzda

- Strategy Pattern (payment, auth)
- Factory Pattern
- Repository Pattern
- Dependency Injection (FASTAPI ning ichki DI modeli)
- Builder Pattern
- Observer Pattern (event-driven architecture)

## VII. ADVANCED FILES, IO & NETWORK
### 17. Low-level socket programming
### 18. Async networking (aiohttp, websockets)
### 19. Binary data handling (struct, pickle, protobuf)
### 20. Memory-mapped files (mmap)

## VIII. TESTING & CLEAN ARCHITECTURE

### 21. Unit & Integration testing
- Mocking (advanced)
- Patch techniques
- Dependency overriding (FastAPI-da super muhim)

### 22. Clean Architecture in Python
- Entities
- Use Cases
- Interfaces
- Infrastructure layer

### 23. DDD real Python misollarida

## IX. EXTRA Advanced Topics (Super Senior)
- Custom interpreter (“meta-programming”)
- AST manipulation (python code generator)
- Plugin system architecture
- Python C API