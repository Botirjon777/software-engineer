# System Design Asoslari — Masalalar Yechimi (CHUQUR instruksiya)

Bu fayl [`system-design/01-fundamentals.md`](../../system-design/01-fundamentals.md) dagi "Masalalar" bo'limining namunaviy yechimlarini o'z ichiga oladi. System design'da yagona to'g'ri javob yo'q — quyidagilar **namunaviy yondashuv**; muhimi, qaroringizni trade-off bilan asoslay olishingiz.

## Mundarija

- [1. Capacity estimation](#1-capacity-estimation)
- [2. Cache strategiyasi tanlash](#2-cache-strategiyasi-tanlash)
- [3. SQL vs NoSQL](#3-sql-vs-nosql)
- [4. CAP tahlili](#4-cap-tahlili)
- [5. Rate limiter dizayni](#5-rate-limiter-dizayni)
- [6. URL shortener](#6-url-shortener)
- [7. Consistent hashing](#7-consistent-hashing)
- [8. Idempotency](#8-idempotency)
- [9. Scaling stsenariysi](#9-scaling-stsenariysi)
- [10. SPOF auditi](#10-spof-auditi)

---

## 1. Capacity estimation

```text
Berilgan: 500M o'qish/kun, o'qish:yozish = 1000:1, tweet ≈ 300 B

YOZISH:
  o'qish 500M/kun, nisbat 1000:1 → yozish = 500M / 1000 = 500K/kun
  500K / 86400 s ≈ 5.8 yozish/s  (≈ 6 QPS)

O'QISH:
  500M / 86400 s ≈ 5787 o'qish/s  (≈ 5.8K QPS)

STORAGE (3 yil):
  500K yozish/kun * 365 * 3 ≈ 5.5 * 10^8 tweet
  5.5*10^8 * 300 B ≈ 1.65 * 10^11 B ≈ 165 GB
```

**💡 Tushuncha:** Bu read-heavy tizim (o'qish yozishdan 1000x ko'p), shuning uchun caching va read replica'lar kritik. Storage esa kichik (165 GB) — bitta DB bemalol ko'taradi.

---

## 2. Cache strategiyasi tanlash

| Stsenariy | Strategiya | Sabab |
|-----------|-----------|-------|
| (a) Mahsulot katalogi | **Cache-aside** | Read-heavy, ma'lumot kam o'zgaradi; miss bo'lganda DB'dan o'qib cache'lash ideal. Stale TTL bilan boshqariladi. |
| (b) Bank balansi | **Write-through** | Consistency kritik; har yozish cache va DB'da bir vaqtda yangilanadi, stale balans ko'rsatilmaydi. |
| (c) Analytics hisoblagich | **Write-back** | Yuqori yozish hajmi, biroz ma'lumot yo'qolishi qabul qilinadi; cache'da to'planib, asinxron DB'ga flush qilinadi (performance). |

**⚠️ Ehtiyot bo'l:** Bank uchun write-back ishlatish — xato, chunki crash'da balans yo'qolishi mumkin. Har stsenariyda consistency vs performance trade-off'ini ko'rsatish kerak.

---

## 3. SQL vs NoSQL

| Stsenariy | Tanlov | Sabab |
|-----------|--------|-------|
| (a) Bank tizimi | **SQL** | Kuchli ACID transactions, munosabatli ma'lumot, to'g'rilik kritik. |
| (b) IoT sensor loglari | **NoSQL** (time-series / Cassandra) | Juda katta yozish hajmi, oddiy access pattern, horizontal scaling, schema moslashuvchanligi. |
| (c) Do'stlik grafi | **Graph DB** (Neo4j) yoki NoSQL | Ko'p bog'lanish/traversal (do'stning do'sti); relational JOIN bu yerda sekin va noqulay. |

**💡 Tushuncha:** "To'g'ri" tanlov access pattern'ga bog'liq. Loglar uchun SQL ham ishlaydi-yu, lekin yozish hajmi va scaling NoSQL'ni afzal qiladi.

---

## 4. CAP tahlili

| Stsenariy | Tanlov | Sabab |
|-----------|--------|-------|
| (a) Bank to'lov | **CP** | To'g'rilik (consistency) muhim; partition paytida noto'g'ri balans ko'rsatgandan ko'ra so'rovni rad etgan yaxshi. |
| (b) Instagram layk | **AP** | Mavjudlik muhim; layk soni biroz eski (stale) bo'lsa zarari yo'q, eventual consistency yetarli. |
| (c) Aviabilet bron | **CP** | Ikki kishi bir o'rindiqni band qilmasligi uchun consistency shart; partition paytida bron'ni rad etish xavfsizroq. |

**💡 Tushuncha:** Umumiy qoida — pul/inventar bilan bog'liq → CP; ijtimoiy/ko'rsatkich ma'lumot → AP. Lekin aniq talab biznesga bog'liq.

---

## 5. Rate limiter dizayni

**Talab:** har foydalanuvchi daqiqada 100 so'rov.

**Algoritm:** Sliding window (silliqroq, fixed window'ning chegara effektisiz) yoki token bucket (burst'ga ruxsat beradi).

**Holat:** Redis'da, atomik operatsiya (`INCR` + `EXPIRE`, yoki sorted set timestamp bilan), chunki distributed app server'lar bir holatni bo'lishishi kerak.

```text
  Client → API Gateway / Middleware
              │
              ▼
        ┌───────────┐   key = user:123
        │   Redis   │   INCR + EXPIRE 60s
        └─────┬─────┘
              │ count > 100 ?
        ┌─────┴─────────┐
       ha                yo'q
        │                 │
   429 Too Many       so'rovni o'tkaz
   Requests
```

**💡 Tushuncha:** Holatni Redis'da saqlash shart — agar har app server o'z hisobini yuritsa, 3 server bo'lsa foydalanuvchi 300 so'rov yubora oladi. Redis atomik bo'lishi (race condition'siz) kerak.

---

## 6. URL shortener

```text
1. REQUIREMENTS
   Functional: URL qisqartirish, short → original redirect.
   Non-functional: past latency (redirect), yuqori availability, read-heavy.

2. ESTIMATION
   10M yozish/kun → 10M/86400 ≈ 116 yozish/s
   o'qish 100:1 → ~11.6K o'qish/s
   Storage 5 yil: 10M*365*5 ≈ 1.8*10^10 yozuv * 500B ≈ 9 TB

3. API
   POST /shorten { url }  → { short }
   GET  /{short}          → 301 redirect

4. HIGH-LEVEL
   Client → CDN/LB → App (stateless) → Cache(Redis) → DB
```

```text
5. DATA MODEL
   urls( short_code PK, original_url, created_at, expires_at )
   - short_code'ga index (redirect query uchun)

6. DEEP DIVE — short kod generatsiya
   Variant A: auto-increment ID → base62 encode (0-9,a-z,A-Z)
     7 belgi base62 ≈ 62^7 ≈ 3.5*10^12 kombinatsiya (yetarli)
   Variant B: hash(url) → birinchi 7 belgi (collision tekshirish kerak)
   Tanlov: base62(counter) — kollisiyasiz, qisqa, oldindan aytib bo'lmaydigan
   bo'lishi kerak bo'lsa counter'ni shuffle qilish.
```

**💡 Tushuncha:** Redirect read-heavy — short_code'larni Redis'da cache'lash latency'ni keskin kamaytiradi. 301 (permanent) vs 302 (temporary): analytics kerak bo'lsa 302 (har redirect serverga keladi).

---

## 7. Consistent hashing

```text
Berilgan: 4 server, biri ishlamay qoldi (4 → 3).

ODDIY MODULO hash(key) % N:
  N 4'dan 3'ga o'zgaradi → deyarli BARCHA kalitlar boshqa serverga ko'chadi
  (chunki % 4 va % 3 butunlay boshqa natija beradi).
  ≈ kalitlarning ~75%+ ko'chadi → katta cache miss "bo'roni".

CONSISTENT HASHING:
  Faqat ishlamay qolgan serverning kalitlari qo'shni serverga ko'chadi.
  ≈ K/N = 1/4 ≈ 25% kalit ko'chadi (faqat o'sha shard).
```

**💡 Tushuncha:** Consistent hashing minimal ko'chish (~K/N) kafolatlaydi, modulo esa deyarli to'liq re-shuffle qiladi. Virtual node'lar bilan taqsimot yanada bir tekis bo'ladi va issiq nuqta kamayadi.

---

## 8. Idempotency

```text
POST /payments
Header: Idempotency-Key: <UUID>  (mijoz generatsiya qiladi)

OQIM:
  1. Server Idempotency-Key'ni Redis/DB'da qidiradi.
  2. Topilsa  → avvalgi natijani qaytaradi (qayta to'lov YO'Q).
  3. Topilmasa → to'lovni bajaradi, natijani key bilan saqlaydi (TTL),
                 keyin qaytaradi.
```

```text
  Client ──Idempotency-Key: abc──► Server
                                     │
                              key exists?
                          ┌──────────┴──────────┐
                         ha                      yo'q
                          │                       │
                  saqlangan javob           to'lovni bajar +
                  (takror to'lov yo'q)      key→natija saqla
```

**💡 Tushuncha:** Race condition'dan saqlanish uchun key'ni atomik (`SET NX`) qo'yish kerak — bir vaqtda ikkita bir xil key kelsa, faqat biri o'tadi. Bu to'lovni ikki marta yechishdan saqlaydi.

---

## 9. Scaling stsenariysi

| Bosqich | Harakat | Qaysi bottleneck'ni hal qiladi |
|---------|---------|--------------------------------|
| 1 | **Cache (Redis)** qo'shish | Takroriy o'qishlar DB'ni urmaydi, latency tushadi |
| 2 | **Read replica**'lar | O'qish yukini replikalar bo'lishadi (read scaling) |
| 3 | **Load balancer** + bir nechta app | App qatlami bottleneck'ini hal qiladi |
| 4 | **Horizontal app** (stateless) | Trafik o'sishini ko'p server bilan ko'taradi; session Redis'ga |
| 5 | **Sharding / partitioning** | Yozish va storage bitta DB'ga sig'maganda |
| 6 | **CDN + async (queue)** | Static content edge'ga, og'ir ishlar asinxron |

**💡 Tushuncha:** Tartib muhim — eng arzon/oson yutuqdan (cache) boshlab, eng murakkabga (sharding) qarab boriladi. Har bosqichdan keyin o'lchang: keyingi bottleneck qayerda?

---

## 10. SPOF auditi

| Qatlam | SPOF xavfi | Yechim |
|--------|-----------|--------|
| CDN | Provider uzilishi | Multi-CDN yoki origin fallback |
| Load balancer | Bitta LB | Ikkita LB (active-passive) + floating IP + health check |
| App server | Bitta instance | Bir nechta stateless instance ortida LB |
| Cache (Redis) | Bitta node | Redis cluster / replica + sentinel (failover) |
| Database | Bitta primary | Primary + replica, avtomatik failover; multi-AZ |
| Region | Bitta data-markaz | Multi-region replikatsiya (disaster recovery) |

**💡 Tushuncha:** Har kritik komponentni dublikat qilish va avtomatik failover qo'shish kerak. Stateless app'lar oson takrorlanadi; stateful (DB, cache) qatlamlar uchun replikatsiya + failover mexanizmi muhim. Geografik redundancy bitta region falokatiga qarshi himoya.

---

← [System Design bo'limiga qaytish](../../system-design/README.md)
