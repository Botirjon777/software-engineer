# Arxitektura Blueprint va Freymvorklar — Yechimlar

Bu fayl [system-design/05-architecture-blueprints.md](../../system-design/05-architecture-blueprints.md) masalalarining namunaviy yechimlarini beradi. Yechimlar — yagona to'g'ri javob emas, balki senior darajadagi fikrlash va asoslashning namunasi. Muhimi — **javob** emas, **trade-off** zanjiri.

## Mundarija

- [1-masala: URL qisqartirgich freymvork](#1-masala-url-qisqartirgich-freymvork)
- [2-masala: Blueprint tanlash](#2-masala-blueprint-tanlash)
- [3-masala: CQRS + Event Sourcing](#3-masala-cqrs--event-sourcing)
- [4-masala: Migratsiya rejasi](#4-masala-migratsiya-rejasi)
- [5-masala: Building block xaritasi](#5-masala-building-block-xaritasi)
- [6-masala: Trade-off tahlili](#6-masala-trade-off-tahlili)
- [7-masala: Hexagonal dizayn](#7-masala-hexagonal-dizayn)
- [8-masala: Kombinatsiya dizayni](#8-masala-kombinatsiya-dizayni)

---

## 1-masala: URL qisqartirgich freymvork

**1. Requirements.** Functional: uzun URL → qisqa kod, qisqa kod → redirect. Non-functional: past latency (redirect < 50ms), yuqori availability (99.9%), read-heavy.

**2. Scale estimation.** 100M URL/oy ≈ 40 write/s. Read:write ≈ 100:1 → 4000 read/s (peak ~10k/s). 5 yilda 6B URL. Har yozuv ~500 bayt → ~3 TB. Bu **bitta DB** uchun katta emas, lekin read uchun cache kritik.

**3. High-level.** Client → LB → API service → (write) DB, (read) Cache → DB. Redirect yo'li 302 qaytaradi.

**4. Data model.** `urls(short_code PK, long_url, created_at, expiry)`. Short_code — base62 (7 belgi → 62^7 ≈ 3.5 trillion). NoSQL key-value (DynamoDB) mos, chunki access — sof key-based.

**5. API.** `POST /shorten {long_url} → {short_code}`; `GET /{short_code} → 302 Location: long_url`.

**6. Deep dive (redirect yo'li).** Cache-first: Redis'da short_code→long_url. Cache hit → darhol 302. Miss → DB'dan o'qib, cache'ga yoz. Kod generatsiyasi: counter + base62 (kollaziyasiz) yoki hash+retry.

**7. Trade-off.** SPOF — DB; yechim: replication + cache. 302 (temporary) vs 301 (permanent): 301 tez, lekin analytics yo'qoladi (brauzer cache qiladi) → 302 tanlaymiz.

**8. Evolve.** 10× o'sganda: read replica'lar, DB sharding (short_code bo'yicha), CDN edge redirect, global multi-region.

---

## 2-masala: Blueprint tanlash

**(a) 3 kishilik startup MVP → Monolith.** Tez ishga tushirish, past operatsion narx, hali domen noaniq. Distributed murakkablik 3 kishini o'ldiradi. Modular tuzilishga rioya qilib, kelajakda ajratishga tayyorlab qo'ying.

**(b) 50 muhandisli e-commerce → Microservices (+ EDA).** Ko'p jamoa mustaqil deploy qilishi kerak; catalog, orders, payments, shipping turli scale profilga ega. Service'lar orasida event-driven aloqa (OrderPlaced → shipping, email, analytics). Har service o'z DB'si.

**(c) IoT real-time → Event-driven (pub/sub) + Pipe-and-filter.** Millionlab sensor event'i → message bus (Kafka) → filtr zanjiri (validate → enrich → aggregate → store). EDA yuk portlashlarini yutadi, decoupling beradi.

**(d) Bank tranzaksiya (audit majburiy) → Event Sourcing + CQRS.** Har tranzaksiya immutable event; balans event'lardan hisoblanadi (source of truth). To'liq audit trail tabiiy. CQRS bilan hisobot/o'qish alohida optimallashadi. Kuchli consistency uchun write tomonida SQL.

---

## 3-masala: CQRS + Event Sourcing

```text
COMMAND tomoni (write):
  PlaceOrder ──► Command Handler ──► validate biznes qoidalar
                                          │
                                          ▼
                              ┌───────────────────────┐
                              │   EVENT STORE          │
                              │   (append-only)        │
                              │  1. OrderCreated       │
                              │  2. ItemAdded (x2)     │
                              │  3. OrderConfirmed     │
                              │  4. OrderShipped       │
                              └───────────┬────────────┘
                                          │ event publish
                    ┌─────────────────────┼─────────────────────┐
                    ▼                     ▼                     ▼
           ┌────────────────┐   ┌────────────────┐   ┌────────────────┐
           │ Read Model:    │   │ Read Model:    │   │ Read Model:    │
           │ OrderSummary   │   │ Analytics      │   │ CustomerView   │
           │ (Postgres)     │   │ (ClickHouse)   │   │ (Redis)        │
           └───────┬────────┘   └────────────────┘   └────────────────┘
                   ▲
QUERY tomoni (read): GET /orders/{id} ──► OrderSummary read model'dan
```

Joriy holat (`Order.status = Shipped`) event'larni replay qilib olinadi. Read model'lar event subscriber orqali async yangilanadi (eventual consistency). Snapshot: har 100 event'da joriy holat saqlanib, replay qisqartiriladi.

---

## 4-masala: Migratsiya rejasi

**Bosqich 0 — Tayyorgarlik.** Facade/API gateway qo'ying (barcha trafik shu orqali). CI/CD, monitoring, distributed tracing sozlang. Modul chegaralarini kodda aniqlang (modular monolith'ga aylantiring).

**Bosqich 1 — Birinchi service tanlash.** Eng mos nomzod: **Notifications** (email/SMS) yoki **Reporting** — chunki ular: (a) low-risk (buzilsa asosiy oqim to'xtamaydi), (b) kam data bog'lanishi (o'z ma'lumoti), (c) turli scale profili. Payments emas — u eng kritik, birinchi bo'lmasin.

**Bosqich 2 — Ajratish.** Notifications kodini yangi service'ga ko'chiring, o'z DB'sini bering. Gateway trafikni yangi service'ga marshrutlaydi. Monolith event chiqaradi (`OrderPlaced`), service obuna bo'ladi.

**Bosqich 3 — Ma'lumot ajratimi.** Anti-corruption layer bilan shared DB bog'lanishini uzing. Data ownership'ni service'ga o'tkazing.

**Bosqich 4 — Takrorlash.** Keyingi og'riqli modulni ajrating (Catalog → Orders → ... → oxirida Payments). Har bosqichda ishlab turgan tizim buzilmaydi (Strangler Fig).

---

## 5-masala: Building block xaritasi (video streaming)

| Block | Muammo |
|-------|--------|
| **CDN** | Video segmentlarini foydalanuvchiga yaqin edge'dan yetkazish (latency, bandwidth) |
| **Object storage (S3)** | Xom va transkod qilingan video fayllarni arzon, chidamli saqlash |
| **Load balancer** | API va streaming trafikni serverlarga taqsimlash |
| **API Gateway** | Auth, rate-limit, routing (upload vs playback) |
| **Message queue (Kafka)** | Upload'dan keyin transkodlash ishini asinxron navbatga qo'yish |
| **Worker / job queue** | Video transkodlash (turli sifat/format) fon rejimida |
| **Cache (Redis)** | Metadata, mashhur video manifest, session ma'lumoti |
| **Search index (Elastic)** | Video qidiruv, ranking, tavsiya uchun facet |
| **DB (SQL + NoSQL)** | Metadata/foydalanuvchi (SQL), ko'rishlar/analitika (NoSQL) |

Oqim: Upload → S3 → Kafka → transcode workers → S3 (segmentlar) → CDN → playback.

---

## 6-masala: Trade-off tahlili

**5 argument (10 dev, 15 microservice — yomon):**

1. **Operatsion overhead.** 15 service = 15 pipeline, 15 monitoring dashboard, 15 deploy — 10 kishi buni boqolmaydi.
2. **Distributed debugging.** Bitta bug'ni 15 service orasidan izlash (tracing'siz) do'zax; kichik jamoada bu tooling yo'q.
3. **Distributed transaction.** Bir necha service'ni qamragan operatsiya saga/compensation talab qiladi — katta murakkablik.
4. **Conway qonuni buzilishi.** 15 service > 10 dev; har kishi bir necha service'ni tashiydi — mustaqillik foydasi yo'qoladi.
5. **Tarmoq latency va failure.** In-process chaqiruvlar tarmoq chaqiruviga aylanadi — sekinroq va ishonchsizroq.

**Tavsiya: Modular monolith.** Toza bounded context modullar, bitta deploy, bitta DB (yoki schema-per-module). Bu jamoaga distributed narxsiz chegara intizomini beradi. Og'riq signali (deploy bloki, scale farqi) paydo bo'lganda tanlab extract qiling.

---

## 7-masala: Hexagonal dizayn (to'lov ishlovi)

```text
                ┌──────────────────────────────────────┐
   HTTP  ──────►│ [Web Adapter]                        │
                │      │                                │
                │      ▼  PORT: PaymentUseCase          │
                │  ┌────────────────────────────────┐  │
                │  │      DOMAIN CORE                │  │
                │  │  - Payment entity              │  │
                │  │  - biznes qoidalar (limit,     │  │
                │  │    valyuta, idempotency)       │  │
                │  └────────────────────────────────┘  │
                │   PORT: PaymentGateway  PORT: Repo    │
                │        │                    │         │
                │   [Stripe Adapter]   [Postgres Adapter]│
                │        │                    │         │
                │   [Kafka Adapter → PaymentCompleted]  │
                └────────┼────────────────────┼─────────┘
                         ▼                    ▼
                      Stripe API          PostgreSQL
```

**Portlar (interfeyslar):** `PaymentUseCase` (kirish), `PaymentGateway` (chiqish — to'lov provayderi), `PaymentRepository` (saqlash), `EventPublisher`.

**Adapterlar:** `StripePaymentAdapter implements PaymentGateway`, `PostgresPaymentRepository implements PaymentRepository`, `KafkaEventPublisher implements EventPublisher`.

**Nega provayder almashtirish oson:** Domain core faqat `PaymentGateway` **interfeysiga** tayanadi, Stripe'ga emas. PayPal'ga o'tsangiz — `PayPalPaymentAdapter` yozasiz, konfiguratsiyada ulaysiz, **yadro va biznes mantiq tegilmaydi**. Testda esa `FakePaymentGateway` bilan tashqi API'siz test qilasiz.

---

## 8-masala: Kombinatsiya dizayni (ovqat yetkazish)

```text
Mobile/Web ─► [BFF] ─► [API Gateway] ─► Microservices
                                          ├─ Orders (Hexagonal)
                                          ├─ Restaurants
                                          ├─ Delivery/Matching
                                          └─ Payments (Hexagonal)
                                              │
                        Event Bus (Kafka) ◄───┤ OrderPlaced
                          ├─► Notifications
                          ├─► Analytics
                          └─► Delivery matching
```

**Ishlatilgan blueprint'lar va sabab:**

1. **Microservices** — Orders, Restaurants, Delivery, Payments turli jamoa/scale; mustaqil deploy kerak.
2. **Event-driven (pub/sub)** — `OrderPlaced` bir voqeaga ko'p consumer (notification, analytics, kuryer matching) reaksiya qiladi; decoupling.
3. **BFF** — mobil (kuryer, mijoz) va web turli data talab qiladi; per-client aggregation.
4. **Hexagonal** — Payments va Orders murakkab biznes mantiqli, uzoq umr ko'radi; to'lov provayderi/DB almashtirilishi mumkin.
5. **CQRS (ixtiyoriy)** — restaurant/menu o'qish juda read-heavy; read model'ni Elastic/Redis'da ajratish. Trade-off har joyda: microservices operatsion narx, EDA eventual consistency, BFF ko'proq kod — lekin bu scale'da oqlanadi.

---

← [System Design bo'limiga qaytish](../../system-design/README.md)
