# Software Engineering — Intervyuga Tayyorgarlik (O'zbek tilida)

Dasturchilar uchun intervyuga tayyorgarlik bo'yicha to'liq bilimlar bazasi — **JavaScript va TypeScript** asosida. Har bir mavzu **kerakli tushunchalar + intervyu savollari + javoblar** ko'rinishida yozilgan, ishlaydigan kod misollari bilan.

> Maqsad: **Frontend**, **Backend** va **Database** bo'yicha asoslardan tortib system design darajasigacha tayyorlash.

> Tushuntirishlar o'zbek tilida; kod, kalit so'zlar va standart texnik atamalar (closure, event loop, JOIN va h.k.) ingliz tilida — chunki real intervyularda aynan inglizcha atamalar ishlatiladi.

---

## Bu repodan qanday foydalanish kerak

- Har bir bo'limning o'z papkasi va `README.md` mundarijasi bor.
- Fayllar o'rganish tartibida raqamlangan.
- Hujjat ichidagi belgilar:
  - **💡 Tushuncha** — bilishingiz shart bo'lgan asosiy g'oya.
  - **❓ Intervyu savoli** — intervyuda so'raladigan ko'rinishda.
  - **✅ Javob** — moslashtirib aytsa bo'ladigan namunaviy javob.
  - **⚠️ Ehtiyot bo'l** — keng tarqalgan xatolar va qo'shimcha savollar.
  - **🧪 Kod** — JS/TS misollari.
- Har bir mavzuning oxirida **## Masalalar** bo'limi bor — yechimlarsiz. Yechimlar alohida [`solutions/`](./solutions/) papkasida joylashgan, shu strukturani takrorlaydi.

---

## Dastur (Curriculum)

### 1. [Frontend](./frontend/README.md)
Brauzerda ishlaydigan hamma narsa.

| # | Mavzu | Nimani o'rganasiz |
|---|-------|--------------------|
| 01 | [HTML](./frontend/01-html.md) | Semantika, formalar, accessibility, DOM |
| 02 | [CSS](./frontend/02-css.md) | Box model, flexbox, grid, specificity, responsive |
| 03 | [JavaScript](./frontend/03-javascript.md) | Tiplar, scope, closure, prototype, `this`, async |
| 04 | [TypeScript](./frontend/04-typescript.md) | Tiplar, generics, narrowing, utility types |
| 05 | [React Asoslari](./frontend/05-react-fundamentals.md) | JSX, komponentlar, props, state, rendering |
| 06 | [React Hooks](./frontend/06-react-hooks.md) | Barcha hooklar, qoidalar, custom hooks |
| 07 | [React Performance](./frontend/07-react-performance.md) | Memoization, re-render, profiling, code splitting |
| 08 | [React Arxitektura](./frontend/08-react-architecture.md) | State management, struktura, patternlar |
| 09 | [Frontend System Design](./frontend/09-frontend-system-design.md) | Katta frontend ilovalarni loyihalash |

### 2. [Backend](./backend/README.md)
Serverda ishlaydigan qism — Node.js orqali.

| # | Mavzu | Nimani o'rganasiz |
|---|-------|--------------------|
| 01 | [Backend nima](./backend/01-what-is-backend.md) | Client/server, so'rov hayot sikli |
| 02 | [Node.js Asoslari](./backend/02-nodejs-fundamentals.md) | Runtime, modullar, V8, libuv |
| 03 | [Event Loop](./backend/03-event-loop.md) | Fazalar, microtask, timerlar, starvation |
| 04 | [Events & EventEmitter](./backend/04-events-eventemitter.md) | Event-driven model |
| 05 | [Streams & Buffers](./backend/05-streams.md) | Readable/writable, backpressure, piping |
| 06 | [Async Patternlar](./backend/06-async-patterns.md) | Callbacks → Promises → async/await |
| 07 | [HTTP & REST API](./backend/07-http-rest-apis.md) | HTTP, REST, status kodlar, dizayn |
| 08 | [Auth & Xavfsizlik](./backend/08-auth-security.md) | Sessions, JWT, OWASP, hashing |
| 09 | [Arxitektura Patternlar](./backend/09-architecture-patterns.md) | Layers, monolith vs microservices, queue |
| 10 | [Backend System Design](./backend/10-backend-system-design.md) | Scaling, caching, load balancing |

### 3. [Database](./databases/README.md)
Ma'lumotlarni saqlash — relational va NoSQL.

| # | Mavzu | Nimani o'rganasiz |
|---|-------|--------------------|
| 01 | [Database Asoslari](./databases/01-fundamentals.md) | Nega DB, data modellar, OLTP vs OLAP |
| 02 | [SQL Asoslari](./databases/02-sql-basics.md) | CRUD, join, aggregatsiya, grouping |
| 03 | [SQL Advanced](./databases/03-sql-advanced.md) | Subquery, window functions, CTE |
| 04 | [Transactions & ACID](./databases/04-transactions-acid.md) | Isolation levels, locking, MVCC |
| 05 | [Indexing & Performance](./databases/05-indexing-performance.md) | B-tree, query plan, optimizatsiya |
| 06 | [NoSQL](./databases/06-nosql.md) | Document, key-value, column, graph |
| 07 | [Database Design](./databases/07-database-design.md) | Normalizatsiya, modeling, CAP, sharding |

---

## Tavsiya etilgan o'rganish tartibi

1. **Avval asoslar**: JS → TS → HTML → CSS.
2. **Yo'nalish tanlang**: Frontend (React) yoki Backend (Node) — yoki ikkalasi.
3. **Database parallel**: SQL asoslarini erta, advanced + design keyinroq.
4. **System design oxirida**: hammasini bog'laydi.

---

## Masalalar va yechimlar

Har bir mavzu oxirida **Masalalar** bo'limi bor (yechimlarsiz). To'liq yechimlar [`solutions/`](./solutions/) papkasida — avval o'zingiz yechib ko'ring, keyin yechimlardan tekshiring.

_Bu — yashayotgan hujjat. Tuzatish va qo'shimchalar uchun ochiq._
