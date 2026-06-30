# System Design Asoslari

System design intervyusi — bu sizning **katta miqyosda fikrlash** qobiliyatingizni tekshiradi. Coding intervyudan farqli, bu yerda yagona to'g'ri javob yo'q; muhimi — **trade-off**larni (murosalar) tushunish, requirement'larni aniqlash, va tizimni bosqichma-bosqich qurish. Bu hujjat distributed systems (taqsimlangan tizimlar) asoslarini va system design intervyusiga yondashuv freymvorkini O'zbek tilida, ingliz atamalari bilan ko'rib chiqadi.

Biz quyidagilarni yoritamiz: 7 qadamli dizayn freymvorki, capacity estimation (QPS/storage/bandwidth hisoblash), scalability, load balancing, caching, ma'lumotlar bazasi tanlash, CAP/PACELC teoremalar, consistency modellari, message queue, microservices, rate limiting, idempotency, consistent hashing va back-of-envelope hisob-kitoblar. Maqsad — sizni har qanday "X ni dizayn qiling" savoliga tizimli yondasha oladigan qilish.

## Mundarija

- [Tushunchalar: nega distributed?](#tushunchalar-nega-distributed)
- [Dizayn freymvorki (7 qadam)](#dizayn-freymvorki-7-qadam)
- [Capacity estimation](#capacity-estimation)
- [Scalability](#scalability)
- [Load balancing](#load-balancing)
- [Caching](#caching)
- [Ma'lumotlar bazasi](#malumotlar-bazasi)
- [CAP theorem va PACELC](#cap-theorem-va-pacelc)
- [Consistency modellari](#consistency-modellari)
- [Message queue](#message-queue)
- [Microservices vs Monolith](#microservices-vs-monolith)
- [Rate limiting va idempotency](#rate-limiting-va-idempotency)
- [Consistent hashing](#consistent-hashing)
- [Redundancy va failover](#redundancy-va-failover)
- [Building block'lar va latency raqamlari](#building-blocklar-va-latency-raqamlari)
- [❓ Savol-javoblar](#-savol-javoblar)
- [Masalalar](#masalalar)

---

## Tushunchalar: nega distributed?

**💡 Tushuncha:** Bitta server (vertical) qancha kuchli bo'lsa ham, chegarasi bor: CPU, RAM, disk, tarmoq. Foydalanuvchilar millionlab bo'lganda, ish bir nechta mashinaga **taqsimlanadi** (distributed). Bu yangi muammolar tug'diradi: tarmoq uzilishi, ma'lumot mosligi (consistency), bir qism ishlamay qolishi (partial failure).

**💡 Tushuncha:** System design — bu **trade-off san'ati**. Hech narsa bepul emas. Tezlik uchun consistency'ni qurbon qilasiz; ishonchlilik uchun pul to'laysiz; soddalik uchun scalability'ni cheklaysiz. Yaxshi dizayner "to'g'ri" javobni emas, **kontekstga mos** javobni topadi.

**⚠️ Ehtiyot bo'l:** Intervyuda darhol murakkab distributed arxitekturaga sakramang. "Sizning kompaniyangizning aksariyat muammolari bitta PostgreSQL bilan hal bo'ladi" — bu real. Murakkablikni **kerak bo'lganda** qo'shing, requirement'lar talab qilganda.

---

## Dizayn freymvorki (7 qadam)

**💡 Tushuncha:** Har qanday "X ni dizayn qiling" savoliga quyidagi freymvork bilan yondashing. Bu sizni tartibli va to'liq qiladi.

```text
1. REQUIREMENTS  → functional + non-functional
2. ESTIMATION    → QPS, storage, bandwidth (back-of-envelope)
3. API DESIGN    → asosiy endpoint'lar
4. HIGH-LEVEL    → komponentlar diagrammasi
5. DATA MODEL    → schema, SQL/NoSQL
6. DEEP DIVE     → bitta-ikkita komponentni chuqur
7. BOTTLENECK    → SPOF, scaling, trade-off
```

### 1. Requirements

- **Functional**: tizim NIMA qiladi. ("Foydalanuvchi URL qisqartiradi va qayta yo'naltiriladi.")
- **Non-functional**: tizim QANDAY ishlaydi. (scalability, latency, availability, consistency, durability.)

**⚠️ Ehtiyot bo'l:** Doim avval scope'ni toraytiring. "Faqat URL qisqartirish va redirect'ga e'tibor qaratamiz, analytics'ni keyinroq" deb intervyuer bilan kelishing.

### 2. Estimation

QPS, storage, bandwidth hisoblang (pastdagi bo'limga qarang).

### 3. API design

```text
POST /api/shorten   { "url": "https://..." }  → { "short": "abc123" }
GET  /{short}       → 301 redirect to original
```

### 4. High-level design

ASCII diagramma chizing (pastda namuna).

### 5. Data model

Jadval/kolleksiyalar, indekslar, SQL yoki NoSQL.

### 6. Deep dive

Intervyuer ko'pincha bitta qismni chuqurroq so'raydi: "Short kod qanday generatsiya qilinadi?", "Cache strategiyasi qanaqa?".

### 7. Bottleneck va trade-off

SPOF (single point of failure), scaling nuqtalari, qaysi trade-off'lar qabul qilingani.

```text
        ┌──────────┐
        │  Client  │
        └────┬─────┘
             │
        ┌────▼─────┐
        │   CDN    │  (static, redirect cache)
        └────┬─────┘
             │
      ┌──────▼───────┐
      │ Load Balancer│  (L7)
      └──────┬───────┘
        ┌────┴─────┐
        │          │
   ┌────▼───┐ ┌────▼───┐
   │ App #1 │ │ App #2 │  (stateless)
   └────┬───┘ └────┬───┘
        └────┬─────┘
       ┌─────▼──────┐
       │   Cache    │  (Redis)
       └─────┬──────┘
       ┌─────▼──────┐
       │  Database  │  (primary + replicas)
       └────────────┘
```

---

## Capacity estimation

**💡 Tushuncha:** "Back-of-envelope" hisob-kitob — taxminiy, lekin tartiblar (order of magnitude) to'g'ri bo'lishi kerak. Intervyuer aniq raqam emas, **fikrlash usulini** ko'radi.

### Namuna: URL shortener

Faraz: kuniga 100M (million) yangi URL yoziladi, o'qish:yozish = 100:1.

```text
YOZISH QPS:
  100M / kun
  100M / 86400 s ≈ 1160 yozish/s  (≈ 1.2K QPS)

O'QISH QPS:
  100:1 nisbat → 116K o'qish/s  (≈ 100K+ QPS)

STORAGE (5 yil):
  100M/kun * 365 * 5 ≈ 1.8 * 10^11 ta yozuv
  Har yozuv ≈ 500 bayt (url + metadata)
  1.8*10^11 * 500 B ≈ 9 * 10^13 B ≈ 90 TB

BANDWIDTH (o'qish):
  116K req/s * 500 B ≈ 58 MB/s  (outgoing)
```

**💡 Tushuncha:** Foydali doimiyliklar:

| Birlik | Qiymat |
|--------|--------|
| 1 kun | ≈ 86,400 s (≈ 10^5) |
| 1 oy | ≈ 2.6M s |
| 1 KB | 10^3 B |
| 1 MB | 10^6 B |
| 1 GB | 10^9 B |
| 1 TB | 10^12 B |

**⚠️ Ehtiyot bo'l:** Hisoblashda yumaloq raqamlar ishlating (86400 ≈ 10^5). Maqsad — kalkulyator emas, miqyos hissi. "90 TB" va "100 TB" — bir xil xulosa beradi.

---

## Scalability

**💡 Tushuncha:** Ikki yo'l bilan kengaytirish mumkin:

| | Vertical (scale up) | Horizontal (scale out) |
|---|---|---|
| Nima | Bitta mashinaga ko'proq resurs | Ko'p mashina qo'shish |
| Misol | 16GB → 128GB RAM | 1 → 50 server |
| Chegarasi | Apparatura chegarasi bor | Deyarli cheksiz |
| Murakkablik | Oddiy | Murakkab (koordinatsiya) |
| SPOF | Ha (bitta mashina) | Yo'q (taqsimlangan) |

**💡 Tushuncha:** Horizontal scaling'ning kaliti — **stateless** servislar. Agar server hech qanday holatni (session, vaqtinchalik ma'lumot) lokal saqlamasa, istalgan so'rovni istalgan serverga yuborsa bo'ladi. Holat tashqi store'ga (Redis, DB) ko'chiriladi.

**⚠️ Ehtiyot bo'l:** Server xotirasida session saqlash — horizontal scaling'ni buzadi, chunki foydalanuvchi har safar boshqa serverga tushishi mumkin. Yechim: session'ni Redis kabi shared store'da saqlash, yoki stateless JWT token.

---

## Load balancing

**💡 Tushuncha:** Load balancer — so'rovlarni bir nechta server o'rtasida taqsimlaydi. Ikki turi:

- **L4 (transport)**: TCP/UDP darajasida, IP/port bo'yicha. Tez, lekin so'rov mazmunini ko'rmaydi.
- **L7 (application)**: HTTP darajasida, URL/header/cookie bo'yicha marshrutlash. Aqlliroq, lekin sekinroq.

Algoritmlar:

| Algoritm | Tavsif |
|----------|--------|
| Round robin | Navbat bilan har serverga |
| Weighted RR | Kuchli serverga ko'proq |
| Least connections | Eng kam yuk bilan serverga |
| IP hash | Bir IP doim bir serverga (sticky) |

**⚠️ Ehtiyot bo'l:** Load balancer'ning o'zi SPOF bo'lishi mumkin. Yechim: ikkita LB (active-passive) + health check + failover.

---

## Caching

**💡 Tushuncha:** Cache — tez-tez so'raladigan ma'lumotni tezkor xotirada (RAM) saqlash, sekin manbaga (DB, disk) bormaslik uchun. Latency'ni keskin kamaytiradi.

### Cache strategiyalari

| Strategiya | Qanday ishlaydi | Qachon |
|------------|-----------------|--------|
| **Cache-aside** | App avval cache'ni tekshiradi, yo'q bo'lsa DB'dan o'qib cache'ga yozadi | Eng keng tarqalgan, read-heavy |
| **Write-through** | Yozish cache va DB'ga bir vaqtda | Consistency muhim |
| **Write-back** | Avval cache'ga, keyin asinxron DB'ga | Yozish tez, lekin ma'lumot yo'qolishi xavfi |

```text
CACHE-ASIDE oqimi (read):
  App → Cache?  ──hit──→ qaytar
         │ miss
         ▼
        DB → o'qi → Cache'ga yoz → qaytar
```

### Eviction policy (siqib chiqarish)

Cache to'lganda nimani o'chirish:

- **LRU** (Least Recently Used) — eng eski ishlatilgan.
- **LFU** (Least Frequently Used) — eng kam ishlatilgan.
- **TTL** — vaqt tugagach o'chadi.

### Cache joylari

- **CDN** — static content (rasm, JS, CSS) foydalanuvchiga yaqin edge'da.
- **Redis/Memcached** — app va DB orasida.
- **Browser cache** — mijoz tomonda.

**⚠️ Ehtiyot bo'l: Cache invalidation** — Computer science'ning eng qiyin muammolaridan biri. DB o'zgarsa, cache eski (stale) qoladi. Yechimlar: TTL (vaqtli muddat), write-through (yozishda yangilash), yoki event-based invalidation (o'zgarishda cache'ni o'chirish). Mukammal yechim yo'q — trade-off bor.

---

## Ma'lumotlar bazasi

### SQL vs NoSQL

| | SQL (PostgreSQL, MySQL) | NoSQL (MongoDB, Cassandra) |
|---|---|---|
| Schema | Qat'iy, oldindan | Moslashuvchan |
| Munosabatlar | JOIN, foreign key | Cheklangan |
| Transactions | ACID kuchli | Ko'pincha cheklangan |
| Scaling | Vertical (asosan) | Horizontal (oson) |
| Qachon | Munosabatli ma'lumot, transactions | Katta hajm, oddiy query, moslashuvchanlik |

**💡 Tushuncha:** "NoSQL har doim tezroq/yaxshiroq" — noto'g'ri. SQL ma'lumotlar bazalari ham juda yaxshi scale qiladi. Tanlov ma'lumot strukturasi va access pattern'ga bog'liq, "moda"ga emas.

### Replication

```text
        ┌─────────┐  write
write → │ Primary │ ───────► replikatsiya
        └────┬────┘
       ┌─────┴─────┐
  ┌────▼───┐  ┌────▼───┐
  │Replica │  │Replica │  ← read (read scaling)
  └────────┘  └────────┘
```

- **Read replicas**: o'qishni replikalar bo'lishadi (read scaling).
- **Failover**: primary ishlamasa, replica primary bo'ladi.
- Trade-off: replikatsiya kechikishi (replication lag) → eventual consistency.

### Sharding / partitioning

**💡 Tushuncha:** Ma'lumot bitta mashinaga sig'maganda, uni qismlarga (shard) bo'linadi. Har shard alohida mashinada.

- **Range-based**: ID 1-1M shard A, 1M-2M shard B. (Issiq nuqta xavfi.)
- **Hash-based**: `hash(key) % N`. Bir tekis taqsimot, lekin shard qo'shish qiyin.
- **Geo-based**: hududga qarab.

**⚠️ Ehtiyot bo'l:** Sharding murakkablik qo'shadi: cross-shard JOIN qiyin, transactions cheklanadi, re-sharding og'riqli. Faqat haqiqatan kerak bo'lganda qiling.

---

## CAP theorem va PACELC

**💡 Tushuncha: CAP teoremasi** — taqsimlangan tizimda quyidagi 3 dan faqat **2 tasini** bir vaqtda kafolatlash mumkin:

- **C** (Consistency) — har o'qish eng so'nggi yozishni ko'radi.
- **A** (Availability) — har so'rov javob oladi (xato bo'lsa ham emas).
- **P** (Partition tolerance) — tarmoq uzilsa ham ishlaydi.

```text
Tarmoq uzilishi (P) bo'lganda tanlov:
  CP → consistency'ni saqla, ba'zi so'rovlar rad etilsin
  AP → mavjudlikni saqla, ba'zi ma'lumot eski (stale) bo'lsin
```

**💡 Tushuncha:** Real distributed tizimda **P shart** (tarmoq uziladi), shuning uchun amalda tanlov **C vs A** o'rtasida. Misol: bank balansi → CP (to'g'rilik muhim); ijtimoiy tarmoq layk soni → AP (mavjudlik muhim, biroz kechiksa zarari yo'q).

**💡 Tushuncha: PACELC** — CAP'ning kengaytmasi: "Agar Partition (P) bo'lsa, A va C orasida tanla; **Else** (E, normal holat) — Latency (L) va Consistency (C) orasida tanla." Ya'ni tarmoq sog'lom bo'lganda ham latency uchun consistency'ni qurbon qilish mumkin.

**⚠️ Ehtiyot bo'l:** CAP'ni "2 tasini tanla, bittasini tashla" deb soddalashtirish chalg'ituvchi. Aniqrog'i: P berilgan bo'lib, faqat partition paytida C va A orasida tanlaysiz. Normal vaqtda ikkalasiga ham erishish mumkin.

---

## Consistency modellari

| Model | Tavsif | Misol |
|-------|--------|-------|
| **Strong** | Har o'qish eng so'nggi yozishni ko'radi | Bank tranzaksiyasi |
| **Eventual** | Ma'lumot oxir-oqibat birxillashadi, lekin vaqtinchalik eski bo'lishi mumkin | DNS, ijtimoiy feed |
| **Read-your-writes** | O'z yozuvingni darhol ko'rasan | Profil yangilash |
| **Causal** | Sababiy bog'liq operatsiyalar tartibi saqlanadi | Izoh-javob threadi |

**💡 Tushuncha:** Strong consistency qulay, lekin qimmat (koordinatsiya, latency). Eventual consistency tezroq va available, lekin ilova vaqtinchalik nomuvofiqlikni ko'tara olishi kerak. Tanlov biznes talabiga bog'liq.

---

## Message queue

**💡 Tushuncha:** Message queue (Kafka, RabbitMQ, SQS) — servislarni **decouple** (ajratish) qiladi va asinxron ishlashga imkon beradi. Producer xabarni queue'ga tashlaydi, consumer keyinroq qayta ishlaydi.

```text
  Producer → [ Queue ] → Consumer
   (tez)      (buffer)    (asinxron)
```

Foydasi:

- **Decoupling**: producer consumer'ni bilmaydi, mustaqil scale qiladi.
- **Buffering**: trafik tepasini (spike) yumshatadi.
- **Reliability**: consumer ishlamasa, xabar queue'da turadi.
- **Async**: foydalanuvchi javobni kutmaydi (masalan email yuborish).

| | Kafka | RabbitMQ |
|---|---|---|
| Model | Log (stream, replay) | Queue (broker) |
| Throughput | Juda yuqori | O'rta |
| Qachon | Event streaming, analytics | Task queue, routing |

**⚠️ Ehtiyot bo'l:** Queue murakkablik qo'shadi: xabar tartibi (ordering), takror yetkazish (at-least-once → idempotency kerak), dead-letter queue. Faqat decoupling/async kerak bo'lganda ishlating.

---

## Microservices vs Monolith

| | Monolith | Microservices |
|---|---|---|
| Struktura | Bitta katta ilova | Ko'p mustaqil servis |
| Deploy | Bitta | Har servis alohida |
| Scaling | Butun ilova | Servis-darajada |
| Murakkablik | Past | Yuqori (tarmoq, koordinatsiya) |
| Jamoa | Kichik | Katta, mustaqil jamoalar |

**💡 Tushuncha:** **Monolith'dan boshlang.** Microservices "moda" emas — u tashkiliy va texnik narx (distributed transactions, network latency, service discovery, monitoring) bilan keladi. Ko'p startaplar uchun yaxshi tuzilgan monolith yetarli. Microservices'ni real ehtiyoj (jamoalar mustaqilligi, har xil scaling) paydo bo'lganda joriy eting.

**⚠️ Ehtiyot bo'l:** "Distributed monolith" — eng yomon holat: microservices murakkabligi, lekin ular bir-biriga qattiq bog'langan (tight coupling). Bu monolithning ham, microservices'ning ham foydasini yo'qotadi.

---

## Rate limiting va idempotency

### Rate limiting

**💡 Tushuncha:** Rate limiting — bir mijoz (IP/token) ma'lum vaqtda nechta so'rov yuborishini cheklaydi. Abuse, DDoS va ortiqcha yukdan himoya qiladi.

Algoritmlar:

- **Token bucket**: vedroda token to'planadi, har so'rov bitta token sarflaydi.
- **Leaky bucket**: so'rovlar barqaror tezlikda chiqadi.
- **Fixed window**: vaqt oynasida N so'rov.
- **Sliding window**: silliqroq, chegara effektisiz.

### Idempotency

**💡 Tushuncha: Idempotent** operatsiya — bir necha marta bajarilsa ham, natija bir xil bo'ladi. Tarmoq xatosida so'rov takrorlansa, bu juda muhim.

```text
GET, PUT, DELETE  → tabiatan idempotent
POST              → idempotent EMAS (har safar yangi yaratadi)
```

**💡 Tushuncha:** POST'ni idempotent qilish uchun **idempotency key** ishlatiladi: mijoz noyob kalit yuboradi, server uni saqlaydi va takror kalitli so'rovni qayta bajarmaydi. To'lov tizimlarida muhim (ikki marta yechmaslik uchun).

---

## Consistent hashing

**💡 Tushuncha:** Oddiy `hash(key) % N` muammosi: server soni (N) o'zgarsa, **deyarli barcha** kalitlar boshqa serverga ko'chadi → katta cache miss / re-shuffle.

**Consistent hashing** buni hal qiladi: server va kalitlar bir **halqaga** (ring) joylashtiriladi. Server qo'shilsa/o'chsa, faqat **qo'shni** kalitlar ko'chadi (o'rtacha K/N).

```text
        0 ────── Node A
       /            \
   key3              key1
      |                |
   Node C            Node B
       \            /
        key2 ──────
  (har kalit soat yo'nalishida keyingi node'ga)
```

**💡 Tushuncha:** Bir tekis taqsimot uchun **virtual nodes** (har fizik node uchun bir nechta nuqta) ishlatiladi. Bu issiq nuqtalarni (hot spot) kamaytiradi. Distributed cache (Redis cluster) va DB sharding'da keng ishlatiladi.

---

## Redundancy va failover

**💡 Tushuncha: Single Point of Failure (SPOF)** — ishlamay qolsa butun tizimni to'xtatadigan komponent. Yaxshi dizaynda SPOF bo'lmasligi kerak.

- **Redundancy**: har kritik komponentni dublikat qilish (LB, DB, app server).
- **Failover**: asosiy komponent ishlamasa, zaxiraga avtomatik o'tish.
  - **Active-passive**: zaxira kutib turadi, kerak bo'lsa faollashadi.
  - **Active-active**: ikkalasi ham ishlaydi, yuk bo'linadi.
- **Health checks**: komponent sog'lig'ini doimiy tekshirish.

```text
            ┌──────────┐
       ┌───►│ Primary  │ (active)
       │    └──────────┘
  Health        │ replikatsiya
  check         ▼
       │    ┌──────────┐
       └───►│ Standby  │ (passive → failover)
            └──────────┘
```

**⚠️ Ehtiyot bo'l:** Redundancy faqat apparatura emas — geografik ham bo'lishi kerak. Bitta data-markaz yonsa, boshqa region'dagi replika qutqaradi (multi-region, multi-AZ).

---

## Building block'lar va latency raqamlari

**💡 Tushuncha:** System design intervyusida ishlatiladigan asosiy "building block"lar:

| Block | Vazifa |
|-------|--------|
| Load balancer | So'rovlarni taqsimlash |
| Reverse proxy | Routing, SSL termination |
| CDN | Static content edge'da |
| Cache (Redis) | Tezkor o'qish |
| Database (SQL/NoSQL) | Doimiy saqlash |
| Message queue | Async, decoupling |
| Object storage (S3) | Fayl/media |
| Search index (Elasticsearch) | To'liq matn qidiruv |
| API gateway | Auth, rate limit, routing |

### Latency raqamlari (taxminiy)

**💡 Tushuncha:** Estimation uchun foydali tartiblar (Jeff Dean's numbers):

| Operatsiya | Latency |
|------------|---------|
| L1 cache | ~1 ns |
| Main memory (RAM) | ~100 ns |
| SSD random read | ~150 µs |
| Datacenter ichida round-trip | ~0.5 ms |
| HDD seek | ~10 ms |
| Internet (qit'alararo) | ~150 ms |

**💡 Tushuncha:** Asosiy xulosalar: RAM diskdan ~1000x tez; bir region ichidagi tarmoq qit'alararodan ~300x tez. Shuning uchun cache (RAM) va ma'lumotni foydalanuvchiga yaqin saqlash (CDN/edge) shunchalik muhim.

---

## ❓ Savol-javoblar

### ❓ System design intervyusiga qanday yondashish kerak?

**✅ Javob:** Tartibli freymvork bilan: (1) requirements (functional + non-functional), (2) capacity estimation, (3) API design, (4) high-level diagramma, (5) data model, (6) deep dive, (7) bottleneck/trade-off. Darhol arxitektura chizmang — avval scope'ni toraytiring va requirement'larni aniqlang. Intervyuer yagona to'g'ri javobni emas, sizning fikrlash jarayoningiz va trade-off tushunchangizni baholaydi.

### ❓ Vertical va horizontal scaling farqi nima?

**✅ Javob:** Vertical (scale up) — bitta mashinaga ko'proq resurs qo'shish (RAM, CPU); oddiy, lekin apparatura chegarasi va SPOF bor. Horizontal (scale out) — ko'p mashina qo'shish; deyarli cheksiz, lekin koordinatsiya murakkabligi keltiradi. Horizontal scaling uchun servislar **stateless** bo'lishi shart — holat tashqi store'ga (Redis, DB) ko'chiriladi.

### ❓ CAP teoremasi nimani anglatadi?

**✅ Javob:** Taqsimlangan tizimda Consistency, Availability va Partition tolerance dan faqat 2 tasini bir vaqtda kafolatlash mumkin. Real tizimda tarmoq uzilishi (P) muqarrar, shuning uchun amalda tanlov C va A o'rtasida: partition paytida consistency'ni saqlash (CP, masalan bank) yoki mavjudlikni saqlash (AP, masalan ijtimoiy feed). PACELC buni kengaytiradi: partition bo'lmasa ham, latency va consistency orasida tanlov bor.

### ❓ Cache-aside, write-through va write-back farqi?

**✅ Javob:** Cache-aside — app cache'ni o'zi boshqaradi: miss bo'lsa DB'dan o'qib cache'ga yozadi (eng keng tarqalgan, read-heavy). Write-through — yozish cache va DB'ga bir vaqtda (consistency yaxshi, yozish sekinroq). Write-back — avval cache'ga, keyin asinxron DB'ga (yozish tez, lekin crash'da ma'lumot yo'qolishi xavfi). Tanlov read/write nisbati va consistency ehtiyojiga bog'liq.

### ❓ Cache invalidation nega qiyin?

**✅ Javob:** DB o'zgarganda cache eski (stale) qoladi va uni qachon/qanday yangilash murakkab. Yechimlar: TTL (vaqt tugagach o'chish — sodda lekin vaqtincha stale), write-through (yozishda cache'ni yangilash), event-based invalidation (o'zgarishda cache'ni o'chirish). Mukammal yechim yo'q — har biri consistency, performance va murakkablik o'rtasida trade-off.

### ❓ SQL va NoSQL ni qachon tanlash kerak?

**✅ Javob:** SQL — munosabatli ma'lumot, kuchli transactions (ACID), murakkab query/JOIN kerak bo'lganda (masalan moliyaviy tizim). NoSQL — katta hajm, oddiy access pattern, moslashuvchan schema va oson horizontal scaling kerak bo'lganda (masalan loglar, katalog). "NoSQL har doim tezroq" degan tushuncha noto'g'ri — tanlov ma'lumot strukturasi va access pattern'ga bog'liq, modaga emas.

### ❓ Replication va sharding farqi nima?

**✅ Javob:** Replication — bir xil ma'lumotning nusxalarini bir nechta serverda saqlash (read scaling, failover, durability uchun). Sharding — ma'lumotni qismlarga bo'lib har shardni alohida serverda saqlash (write scaling va hajm uchun, bitta mashinaga sig'maganda). Replication o'qishni kengaytiradi, sharding yozishni va saqlash hajmini. Ko'pincha ikkalasi birga ishlatiladi.

### ❓ Message queue nega kerak?

**✅ Javob:** Decoupling (producer va consumer'ni ajratish, mustaqil scaling), buffering (trafik spike'ni yumshatish), reliability (consumer ishlamasa xabar saqlanadi), va async ishlash (foydalanuvchi javobni kutmaydi, masalan email/notification). Kafka — event streaming va yuqori throughput uchun; RabbitMQ — task queue va murakkab routing uchun. Lekin queue murakkablik qo'shadi (ordering, takror yetkazish), shuning uchun kerak bo'lganda ishlating.

### ❓ Idempotency nima va nega muhim?

**✅ Javob:** Idempotent operatsiya bir necha marta bajarilsa ham natija o'zgarmaydi. GET/PUT/DELETE tabiatan idempotent, POST esa emas (har safar yangi resurs yaratadi). Tarmoq xatosida so'rov takrorlanishi mumkin, shuning uchun POST'ni **idempotency key** bilan idempotent qilish kerak — server kalitni saqlaydi va takror kalitli so'rovni qayta bajarmaydi. To'lov tizimlarida juda muhim (ikki marta pul yechmaslik).

### ❓ Consistent hashing oddiy modulo hashing'dan nimasi yaxshi?

**✅ Javob:** Oddiy `hash(key) % N` da server soni o'zgarsa deyarli barcha kalitlar boshqa serverga ko'chadi (katta cache miss / re-shuffle). Consistent hashing'da server va kalitlar halqaga joylashtiriladi va server qo'shilsa/o'chsa faqat qo'shni kalitlar (~K/N) ko'chadi. Bir tekis taqsimot uchun virtual node'lar ishlatiladi. Distributed cache (Redis cluster) va DB sharding'da keng qo'llanadi.

### ❓ Strong va eventual consistency farqi?

**✅ Javob:** Strong consistency — har o'qish eng so'nggi yozishni ko'radi; qulay, lekin koordinatsiya va latency narxi bor (masalan bank balansi). Eventual consistency — ma'lumot oxir-oqibat birxillashadi, lekin vaqtinchalik eski bo'lishi mumkin; tezroq va available (masalan DNS, ijtimoiy feed). Tanlov biznes talabiga bog'liq: ilova vaqtinchalik nomuvofiqlikni ko'tara olsa, eventual yaxshiroq scale qiladi.

### ❓ Microservices'ni qachon tanlash kerak?

**✅ Javob:** Ko'p hollarda — yo'q, monolith'dan boshlang. Microservices tashkiliy va texnik narx keltiradi: distributed transactions, network latency, service discovery, murakkab monitoring. Uni real ehtiyoj paydo bo'lganda joriy eting: mustaqil jamoalar, har xil scaling profillari, har xil texnologiya stack'i. "Distributed monolith" (qattiq bog'langan microservices) — eng yomon holat, ikki dunyoning ham kamchiligini oladi.

### ❓ Back-of-envelope estimation qanday qilinadi?

**✅ Javob:** Taxminiy, lekin to'g'ri tartibdagi (order of magnitude) hisob. Kunlik so'rovlar sonidan QPS chiqaring (kunlik / 86400 ≈ 10^5). Storage uchun: yozuvlar soni × yozuv hajmi × saqlash muddati. Bandwidth uchun: QPS × o'rtacha javob hajmi. Yumaloq raqamlar ishlating — intervyuer kalkulyator emas, miqsos hissini va metodni ko'radi.

### ❓ SPOF nima va undan qanday qochish kerak?

**✅ Javob:** Single Point of Failure — ishlamay qolsa butun tizimni to'xtatadigan komponent (masalan bitta DB yoki LB). Undan qochish: redundancy (kritik komponentlarni dublikat qilish), failover (asosiy ishlamasa zaxiraga avtomatik o'tish — active-passive yoki active-active), health checks, va geografik tarqatish (multi-AZ/multi-region). Hatto load balancer ham SPOF bo'lmasligi uchun ikkita bo'lishi kerak.

### ❓ Load balancer'da L4 va L7 farqi nima?

**✅ Javob:** L4 (transport) — TCP/UDP darajasida IP/port bo'yicha taqsimlaydi; tez, lekin so'rov mazmunini ko'rmaydi. L7 (application) — HTTP darajasida URL, header, cookie bo'yicha aqlli marshrutlash qiladi (masalan `/api` bir serverga, `/static` boshqasiga); moslashuvchan, lekin biroz sekinroq. Algoritmlar: round robin, weighted, least connections, IP hash (sticky session).

### ❓ Latency raqamlarini bilish nega muhim?

**✅ Javob:** Ular dizayn qarorlarini boshqaradi. RAM (~100ns) diskdan (~10ms) ~1000x, bir region ichidagi tarmoq (~0.5ms) qit'alararodan (~150ms) ~300x tez. Shuning uchun cache (RAM) DB so'rovini almashtirsa katta yutuq, va ma'lumotni foydalanuvchiga yaqin (CDN/edge) saqlash kerak. Estimation paytida bu raqamlar bottleneck qayerda ekanini ko'rsatadi.

---

## Masalalar

> Yechimlar: [solutions/system-design/01-fundamentals.md](../solutions/system-design/01-fundamentals.md)

1. **Capacity estimation.** Twitter kabi tizim uchun: kuniga 500M tweet o'qiladi, o'qish:yozish = 1000:1. Yozish QPS, o'qish QPS, va 3 yillik storage'ni hisoblang (har tweet ≈ 300 bayt).

2. **Cache strategiyasi tanlash.** Quyidagi 3 stsenariy uchun mos cache strategiyasini (cache-aside / write-through / write-back) tanlang va asoslang: (a) mahsulot katalogi (read-heavy), (b) bank balansi, (c) analytics hisoblagich (counter).

3. **SQL vs NoSQL.** Quyidagilar uchun SQL yoki NoSQL tanlang va sababini yozing: (a) bank tizimi, (b) IoT sensor loglari (kuniga milliardlab yozuv), (c) ijtimoiy tarmoq do'stlik grafi.

4. **CAP tahlili.** Quyidagi tizimlar CP yoki AP bo'lishi kerakligini aniqlang va asoslang: (a) bank to'lov tizimi, (b) Instagram layk hisoblagichi, (c) aviabilet bron qilish.

5. **Rate limiter dizayni.** API uchun "har foydalanuvchi daqiqada 100 so'rov" rate limiter'ni dizayn qiling. Qaysi algoritm (token bucket / sliding window) va qayerda holat saqlanishini (Redis) tushuntiring, ASCII diagramma chizing.

6. **URL shortener.** TinyURL'ni 7 qadamli freymvork bilan dizayn qiling: requirements, estimation (kuniga 10M yangi URL), API, high-level diagramma, data model, va short kod generatsiya strategiyasi (deep dive).

7. **Consistent hashing.** 4 ta cache serveridan biri ishlamay qoldi. Oddiy modulo hashing va consistent hashing'da nechta kalit ko'chishini taqqoslang (taxminiy) va nega consistent hashing afzalligini tushuntiring.

8. **Idempotency.** To'lov API'si (`POST /payments`) uchun idempotency mexanizmini dizayn qiling: idempotency key qanday ishlaydi, server qayerda saqlaydi, va takror so'rovni qanday aniqlaydi.

9. **Scaling stsenariysi.** Bitta serverdagi monolith ilova 100x trafik o'sishini boshdan kechirdi. Bosqichma-bosqich qanday scale qilasiz (cache → read replica → load balancer → horizontal app → sharding)? Har bosqich qaysi bottleneck'ni hal qilishini yozing.

10. **SPOF auditi.** Yuqoridagi high-level diagrammadagi (Client → CDN → LB → App → Cache → DB) har bir qatlamdagi SPOF'larni aniqlang va har biri uchun redundancy/failover yechimini taklif qiling.

---

← [System Design bo'limiga qaytish](./README.md)
