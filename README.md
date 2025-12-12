# 🐍 ADVANCED PYTHON CORE

Ushbu repository Advanced Python bilimlarini chuqur o'rganish uchun mo'ljallangan.

## ✅ I. PYTHON FOUNDATIONS (Advanced darajada qayta ko'rib chiqish)

### [1. Advanced Data Types](I.%20PYTHON%20FOUNDATIONS%20(Advanced)/1.%20Advanced%20Data%20Types/README.md)
- Mutable vs Immutable (interning, memory optimization)
- tuple optimizatsiyasi, frozenset, memoryview
- bytes vs bytearray

### [2. Object Model](I.%20PYTHON%20FOUNDATIONS%20(Advanced)/2.%20Object%20Model/README.md)
- `id()`, `type()`, `__class__`, `__dict__`, `__slots__`
- Python memory model (stack → heap → reference counting)
- Garbage Collector: generations, cyclic GC

### [3. Deep Copy vs Shallow Copy](I.%20PYTHON%20FOUNDATIONS%20(Advanced)/3.%20Deep%20Copy%20vs%20Shallow%20Copy/README.md)
- Copy protokoli (`__copy__`, `__deepcopy__`)

---

## II. ADVANCED OOP & DESIGN

### [4. Mixin Architecture](II.%20ADVANCED%20OOP%20&%20DESIGN/4.%20Mixin%20Architecture/README.md)
- Django va FastAPI loyihalaringda ishlatish
- Multiple inheritance safe patternlar

### [5. Descriptor Protocol](II.%20ADVANCED%20OOP%20&%20DESIGN/5.%20Descriptor%20Protocol/README.md)
- `__get__`, `__set__`, `__delete__`
- Django ORM descriptorlarini tushunish
- CachedProperty, property descriptorlar

### [6. Metaclasses — Real Use](II.%20ADVANCED%20OOP%20&%20DESIGN/6.%20Metaclasses%20—%20Real%20Use/README.md)
- Class yaratilishini boshqarish
- Django Model metaclass qanday ishlaydi?
- FastAPI Pydantic model generation

### [7. Abstract Base Classes (ABC)](II.%20ADVANCED%20OOP%20&%20DESIGN/7.%20Abstract%20Base%20Classes%20(ABC)/README.md)
- `abc.ABC`, `abstractmethod`
- Interface segregationni Python usulida qilish

---

## III. FUNCTIONAL PYTHON

### [8. Closures & advanced scopes](III.%20FUNCTIONAL%20PYTHON/8.%20Closures%20&%20advanced%20scopes/README.md)
- LEGB rule — compiled bytecode orqali tushuntirish
- Closure'larning real backend ishlanmasi (cache, factory, config)

### [9. Decoratorlar — advanced](III.%20FUNCTIONAL%20PYTHON/9.%20Decoratorlar%20—%20advanced/README.md)
- Decorator inside decorator (parameter decorator)
- Class decorator
- Method decorator
- Funksional middleware patternlari

### [10. Generatorlar va Iteratorlar](III.%20FUNCTIONAL%20PYTHON/10.%20Generatorlar%20va%20Iteratorlar/README.md)
- Custom iteratorlar
- Generator pipelines
- `yield from`
- Infinite streamlar

---

## IV. PYTHON RUNTIME & PERFORMANCE

### [11. CPython Interpreter ichki ishlashi](IV.%20PYTHON%20RUNTIME%20&%20PERFORMANCE/11.%20CPython%20Interpreter%20ichki%20ishlashi/README.md)
- Bytecode
- `dis` moduli bilan tahlil
- GIL — real tushuntirish
- Concurrency model: threading vs multiprocessing

### [12. Memory Optimization](IV.%20PYTHON%20RUNTIME%20&%20PERFORMANCE/12.%20Memory%20Optimization/README.md)
- `sys.getsizeof()`
- Object allocation
- `__slots__` bilan RAM optimizatsiya
- C-extensionlar

### [13. Profiling](IV.%20PYTHON%20RUNTIME%20&%20PERFORMANCE/13.%20Profiling/README.md)
- `cProfile`, `line_profiler`, `py-spy`
- Bottleneck aniqlash
- Request-level profiling (FastAPI & Django)

---

## V. ADVANCED ASYNC PYTHON

### [14. Event Loop deep dive](V.%20ADVANCED%20ASYNC%20PYTHON/14.%20Event%20Loop%20deep%20dive/README.md)
- Tasks
- Future
- Awaitable protokoli

### [15. Async context managers](V.%20ADVANCED%20ASYNC%20PYTHON/15.%20Async%20context%20managers/README.md)
- `async with` protokoli
- Connection pooling, Redis, DB sesiyalarida

### [16. Asyncio internals](V.%20ADVANCED%20ASYNC%20PYTHON/16.%20Asyncio%20internals/README.md)
- `gather`, `wait`, `shield`, `semaphore`, `lock`
- Asyncio cancellation
- Task groups (`asyncio.TaskGroup`)

---

## VI. DESIGN PATTERNS (ADVANCED DARAJADA)

> 🔥 **Muhimi:** Pythonic tarzda

- **Strategy Pattern** (payment, auth)
- **Factory Pattern**
- **Repository Pattern**
- **Dependency Injection** (FastAPI ning ichki DI modeli)
- **Builder Pattern**
- **Observer Pattern** (event-driven architecture)

---

## VII. ADVANCED FILES, IO & NETWORK

### 17. Low-level socket programming
### 18. Async networking (aiohttp, websockets)
### 19. Binary data handling (struct, pickle, protobuf)
### 20. Memory-mapped files (mmap)

---

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

---

## IX. EXTRA Advanced Topics (Super Advanced)

- Custom interpreter ("meta-programming")
- AST manipulation (python code generator)
- Plugin system architecture
- Python C API

---

## 📚 Qo'shimcha Resurslar

- [Python Documentation](https://docs.python.org/3/)
- [Real Python](https://realpython.com/)
- [Python Enhancement Proposals (PEPs)](https://peps.python.org/)

---

## 🎯 Maqsad

Bu repository orqali siz:

✅ Python ichki mexanizmlarini chuqur tushunasiz  
✅ Production-ready kod yozishni o'rganasiz  
✅ ADVANCED/Advanced-level Python developer bo'lasiz  
✅ FastAPI va Django loyihalarida expert bo'lasiz

---

**Happy Coding! 🚀**