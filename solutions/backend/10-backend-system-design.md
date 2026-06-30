# Backend System Design — Masalalar Yechimi

Bu fayl [`backend/10-backend-system-design.md`](../../backend/10-backend-system-design.md) bo'limidagi "Masalalar" qismining yechimlari.

---

## 1. Capacity baholash

**Berilgan:** 500M user, 100M post/kun, 5 rasm/post, 2MB/rasm.

**Yozish QPS (post):**
- 100M post/kun ÷ 86 400 sek ≈ **1 160 post/sek** (o'rtacha).
- Peak (3x) ≈ **3 480 post/sek**.

**Rasm yozish:** 100M × 5 = 500M rasm/kun ≈ 5 800 rasm/sek o'rtacha, peak ≈ 17 400/sek.

**Storage/kun:** 500M rasm × 2MB = **1 000 000 000 MB = ~1 PB/kun** (rasmlar). Metadata (post, user) bunga nisbatan kichik (~bir necha GB/kun).

**5 yillik storage:** ~1 PB/kun × 365 × 5 ≈ **~1.8 EB** (xom). Amalda CDN, compression, thumbnail va replikatsiya (3x) hisobga olinadi — replikatsiya bilan ~5.5 EB. Bu masshtab → object storage (S3) + CDN majburiy, hech qachon bitta DB emas.

**Xulosa:** O'qish-og'ir (feed ko'rish post yaratishdan ancha ko'p), storage rasmlarda hukmron — rasmlarni object storage + CDN'ga, metadata'ni sharded DB'ga ajrating.

---

## 2. Caching strategiyasi tanlash

| Holat | Strategiya | Sabab |
|-------|-----------|-------|
| (a) Mahsulot katalogi | **Cache-aside** (uzun TTL) | Kam yoziladi, ko'p o'qiladi; lazy load + uzun TTL ideal. Yangilanganda `del` bilan invalidate. |
| (b) Foydalanuvchi sessiyalari | **Write-through** (yoki cache asosiy store) | Sessiya cache'da yashaydi, yozish darhol ko'rinishi kerak; Redis o'zi asosiy store bo'lishi mumkin (TTL bilan). |
| (c) Real-time o'yin skorlari | **Write-back** | Juda tez-tez yoziladi; har yozishni DB'ga urish DB'ni o'ldiradi. Cache'da to'plab, periodik DB'ga flush. Crash'da oz yo'qotish toqat qilinadi. |

**Izoh:** Tanlov yozish chastotasi va izchillik talabiga bog'liq. O'qish-og'ir + stale toqat → cache-aside. Yozish darhol ko'rinishi → write-through. Yozish portlashi + DB himoyasi → write-back.

---

## 3. Rate limiter dizayni

**Algoritm:** Token bucket (burst ruxsati) yoki fixed/sliding window. Bu yerda sliding window + Redis ko'rsataman, holat markazlashgan.

```ts
// sliding window log — Redis sorted set bilan, atomar Lua
const LUA = `
local key = KEYS[1]
local now = tonumber(ARGV[1])
local window = tonumber(ARGV[2])
local limit = tonumber(ARGV[3])
redis.call('ZREMRANGEBYSCORE', key, 0, now - window)  -- eski yozuvlarni o'chir
local count = redis.call('ZCARD', key)
if count < limit then
  redis.call('ZADD', key, now, now)
  redis.call('EXPIRE', key, math.ceil(window/1000))
  return 1
end
return 0
`;

async function allow(apiKey: string) {
  const now = Date.now();
  const ok = await redis.eval(LUA, 1, `rl:${apiKey}`, now, 3_600_000, 1000);
  return ok === 1; // 1000 so'rov / soat (3 600 000 ms)
}
```

**Atomarlik:** Butun "eski o'chir → sana → qo'sh" mantiqi bitta Lua script'da bajariladi — 5 instance bir vaqtda kelsa ham Redis single-threaded bo'lib ketma-ket atomar ishlaydi, race yo'q.

**Markazlashganlik:** Holat faqat Redis'da, app instance'lar stateless — shuning uchun haqiqiy limit 1000/soat bo'lib qoladi, 5×1000 emas.

**Javob:** Limit oshganda:
```
HTTP 429 Too Many Requests
Retry-After: 1800
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 0
```

---

## 4. CAP qarori

| Holat | Tanlov | Sabab |
|-------|--------|-------|
| (a) Bank pul o'tkazmasi | **CP** | Noto'g'ri balans qabul qilinmaydi; partition'da yozishni rad etish "ikki marta sarflash"dan yaxshi. |
| (b) Twitter like soni | **AP** | Like soni vaqtincha noto'g'ri bo'lsa zarari yo'q; availability muhimroq, eventual consistency yetarli. |
| (c) E-commerce inventar | **CP** (yoki kuchli koordinatsiya) | Oversell (yo'q mahsulotni sotish) jiddiy muammo; stok bo'yicha izchillik kerak. (Ba'zi tizimlar AP + keyin reconcile qiladi, lekin xavfli.) |
| (d) DNS | **AP** | DNS yuqori available bo'lishi shart; yangilanish eventual tarqaladi (TTL), vaqtincha eski yozuv toqat qilinadi. |

**Umumiy qoida:** Pul/inventar kabi "noto'g'ri = zarar" → CP. Hisoblagich/feed/DNS kabi "vaqtincha eski = mayli" → AP.

---

## 5. URL shortener deep dive (counter scaling)

Muammo: global counter bitta nuqta bo'lsa SPOF va bottleneck.

**Variant 1 — Range allocation (blok ajratish):**
Markaziy "ID service" har app serverga ID **bloki** ajratadi (masalan 1–10 000, keyin 10 001–20 000). Server o'z bloki ichida lokal hisoblaydi, blok tugaganda yangi blok so'raydi. Markazga murojaat kam (har 10 000 ID'da bir marta), to'qnashuv yo'q, bir server o'lsa faqat o'sha blok yo'qoladi (zarari yo'q).

**Variant 2 — Snowflake-ga o'xshash ID:**
Har ID koordinatsiyasiz generatsiya qilinadi: `timestamp (41 bit) + machine_id (10 bit) + sequence (12 bit)`. Markaziy counter umuman kerak emas, har mashina mustaqil unique ID yaratadi. Keyin base62'ga aylantiriladi. To'liq distributed, SPOF yo'q.

**Tavsiya:** Variant 1 qisqaroq kod beradi (ketma-ket counter), Variant 2 to'liq markazsiz. Yuqori scale va koordinatsiyani minimallashtirish kerak bo'lsa — Snowflake.

---

## 6. Cache stampede himoyasi

```ts
const TTL = 600; // 10 min
const JITTER = 60; // ± tasodifiy

async function getProductPage(id: string) {
  const key = `page:${id}`;
  const hit = await redis.get(key);
  if (hit) return JSON.parse(hit);

  // single-flight: faqat lock olgan DB'ga boradi
  const gotLock = await redis.set(`lock:${key}`, "1", "NX", "EX", 10);
  if (!gotLock) {
    await sleep(100);
    return getProductPage(id); // qolganlar qisqa kutib qayta urinadi
  }

  try {
    const data = await db.fetchProductPage(id);
    const ttl = TTL + Math.floor(Math.random() * JITTER); // jittered TTL
    await redis.set(key, JSON.stringify(data), "EX", ttl);
    return data;
  } finally {
    await redis.del(`lock:${key}`);
  }
}
```

**Ikki himoya birga:**
- **Single-flight lock** — kalit expire bo'lganda 1M so'rovdan faqat bittasi DB'ga boradi, qolganlari qisqa kutib cache'dan oladi. DB bitta query oladi, million emas.
- **Jittered TTL** — kalitlar bir vaqtda expire bo'lmaydi, har birining TTL'i biroz farq qiladi → sinxron stampede oldini olinadi.

---

## 7. DB scaling rejasi

**Bosqich 0 — bitta Postgres.**
Boshlang'ich; oddiy, yetarli. Index'lar, query optimizatsiya, connection pooling birinchi navbatda.

**Bosqich 1 — vertical + caching.**
Yuk oshganda: kuchliroq mashina + Redis cache (cache-aside). 95% o'qish bo'lgani uchun cache hit rate yuqori — DB o'qish yukining katta qismi cache'ga ko'chadi. Trade-off: cache invalidation murakkabligi, stale risk.

**Bosqich 2 — read replica.**
Cache yetmay qolganda: master (yozish) + bir nechta read replica (o'qish). 95% o'qish replica'larga taqsimlanadi, master faqat 5% yozishni ko'taradi. Trade-off: **replication lag** — read-after-write muhim joyda master'dan o'qish kerak.

**Bosqich 3 — sharding.**
Yozish (5%) ham bitta master'ni bosganda yoki dataset bitta mashinaga sig'maganda: shard key bo'yicha bo'lish (masalan `user_id`). Trade-off: cross-shard query/JOIN qiyin, rebalance murakkab, ilova logikasi shard-aware bo'lishi kerak. Eng oxirgi chora — murakkablik yuqori.

**Tartib muhim:** Index → cache → vertical → read replica → sharding. Sharding'ga oxirida boring, chunki u eng katta operatsion narx keltiradi.

---

## 8. Notification system

```
Trigger (event) → API → Producer → [ Kafka/SQS queue ]
                                          │
                              ┌───────────┴───────────┐
                          Worker 1   Worker 2 ...  Worker N  (auto-scale)
                              │
                     idempotency check (sent_id)
                              │
                   ┌──────────┼──────────┐
                 Push       Email       SMS  (kanal adapterlar)
                              │
                       fail → retry (backoff) → N marta keyin → DLQ
```

**Komponentlar:**
- **Queue** — Kafka yoki SQS; spike'da (1M bir vaqtda) **load leveling**: barcha 1M xabar queue'ga buferlanadi, worker'lar o'z barqaror tezligida iste'mol qiladi. Tashqi servis (APNs/FCM/email provider) rate limit'iga mos. DB/provider ezilmaydi.
- **Worker'lar** — gorizontal scale; backlog o'sganda avtomatik ko'payadi (queue depth metric'iga qarab).
- **Idempotency** — har notification'ga unique `notification_id`; worker yuborishdan oldin "bu allaqachon yuborilganmi?" tekshiradi (Redis/DB), at-least-once delivery'da dublikatni oldini oladi.
- **Retry** — vaqtinchalik xatoda (provider 503) eksponensial backoff + jitter bilan qayta urinish.
- **DLQ** — N marta urinishdan keyin ham yiqilsa, "dead-letter queue"ga ko'chiriladi; poison xabar boshqa xabarlarni bloklamaydi, keyin qo'lda tekshiriladi.

**Spike boshqaruvi:** Queue = bufer. 1M xabar darhol provider'ga emas, queue'ga tushadi; worker'lar (masalan 100 ta, har biri 100/sek) ≈ 10 000/sek tezlikda yuboradi, 1M xabar ~100 sekundda tekis ketadi — provider va tizim barqaror qoladi.

---

← [Backend bo'limiga qaytish](../../backend/README.md)
