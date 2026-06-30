# Backend System Design

System design intervyusi — senior pozitsiyalarda hal qiluvchi bosqich. Bu yerda kod emas, **fikrlash jarayoni** baholanadi: talablarni aniqlash, miqyosni baholash, trade-off'larni ko'rsatish va bottleneck'larni topish. Bu bo'lim system design'ga yondashuv, scalability, caching, DB scaling, CAP, rate limiting va failure'ga dizayn'ni qamrab oladi, so'ngra 3 ta ishlangan dizayn beradi.

## Mundarija

- [System Design Intervyusiga Yondashuv](#yondashuv)
- [Scalability (Vertical vs Horizontal)](#scalability)
- [Load Balancing](#load-balancing)
- [Caching Strategiyalari](#caching-strategiyalari)
- [Cache Stampede / Thundering Herd](#cache-stampede)
- [Database Scaling](#database-scaling)
- [CAP Theorem va Consistency Modellar](#cap-theorem)
- [CDN](#cdn)
- [Message Queue (Decoupling / Load Leveling)](#message-queue)
- [Rate Limiting Algoritmlari](#rate-limiting)
- [Idempotency](#idempotency)
- [Consistent Hashing](#consistent-hashing)
- [Failure uchun Dizayn](#failure-dizayn)
- [Monitoring va SLA/SLO](#monitoring-sla-slo)
- [Ishlangan Dizayn: URL Shortener](#dizayn-url-shortener)
- [Ishlangan Dizayn: Rate Limiter](#dizayn-rate-limiter)
- [Ishlangan Dizayn: Notification / Feed](#dizayn-notification-feed)
- [Intervyu Savollari (Q&A)](#intervyu-savollari)
- [Masalalar](#masalalar)

---

## Yondashuv

**💡 Tushuncha:** System design intervyusida tartibli qadam ketma-ketligi muhim — chalkash boshlama, struktura ko'rsat:

1. **Requirements aniqlash** — functional (nima qiladi: URL qisqartirish, redirect) va non-functional (qancha tez, qancha foydalanuvchi, availability). Savol berib scope'ni torting.
2. **Scale / capacity baholash** — QPS (queries per second), storage, bandwidth. Taxminiy hisoblang: "100M URL/oy ≈ 40 yozish/sek, o'qish 100x ko'p ≈ 4000 o'qish/sek".
3. **API dizayn** — asosiy endpointlar (`POST /shorten`, `GET /{code}`).
4. **High-level dizayn** — quti-strelka diagramma: client → LB → app → cache → DB.
5. **Deep dive** — bitta qismni chuqurlashtir (DB schema, hashing, cache strategiyasi).
6. **Bottleneck va trade-off** — qaerda yiqiladi? Qanday scale qilamiz? Nima qurbon qilamiz?

**⚠️ Ehtiyot bo'l:** Eng ko'p uchraydigan xato — talablarni so'ramay darhol dizayn chizishni boshlash. Avval **scope'ni torting** va **assumption'larni ovoz chiqarib** ayting. Intervyuer fikrlash jarayonini ko'rmoqchi, tayyor javobni emas.

---

## Scalability

**💡 Tushuncha:** Tizimni kattalashtirishning ikki yo'li:

- **Vertical scaling (scale up):** bitta mashinaga ko'proq resurs (CPU, RAM). Oddiy, lekin chegarasi bor va single point of failure.
- **Horizontal scaling (scale out):** ko'proq mashina qo'shish. Deyarli cheksiz, fault-tolerant, lekin distributed murakkablik keltiradi.

Horizontal scaling ishlashi uchun ilova **stateless** bo'lishi shart — holat lokal emas, Redis/DB'da. Shunda istalgan instance istalgan so'rovni qabul qila oladi.

| | Vertical | Horizontal |
|---|----------|------------|
| Qanday | kuchliroq mashina | ko'proq mashina |
| Chegara | apparat chegarasi | deyarli cheksiz |
| Fault tolerance | yo'q (SPOF) | bor |
| Murakkablik | past | yuqori |
| Talab | — | stateless app |

**⚠️ Ehtiyot bo'l:** "Shunchaki ko'proq server qo'shamiz" deyish yetarli emas — agar ilova stateful bo'lsa (session lokal xotirada), horizontal scaling buziladi. Avval stateless qiling.

---

## Load Balancing

**💡 Tushuncha:** Load balancer kelayotgan trafikni ko'p server orasida taqsimlaydi. Algoritmlar:

- **Round robin** — navbat bilan har serverga.
- **Least connections** — eng kam aktiv ulanishi bor serverga.
- **Weighted** — serverlarga vazn berib (kuchliroq ko'proq oladi).
- **IP hash** — mijoz IP'siga ko'ra (session affinity uchun).

**L4 vs L7:**
- **L4 (transport)** — TCP/UDP darajasida, IP/port'ga qarab marshrutlaydi. Tez, lekin so'rov mazmunini ko'rmaydi.
- **L7 (application)** — HTTP darajasida, URL/header/cookie'ga qarab marshrutlaydi. Aqlli (path-based routing, SSL termination), lekin biroz sekinroq.

**⚠️ Ehtiyot bo'l:** Load balancer'ning o'zi single point of failure bo'lishi mumkin — productionda u ham redundant (active-passive yoki active-active) bo'lishi kerak. Sticky session (IP hash) ishlatsangiz, stateless prinsip buziladi va scaling notekis bo'ladi.

---

## Caching Strategiyalari

**💡 Tushuncha:** Cache tez-tez so'raladigan ma'lumotni tez xotirada saqlab latency va DB yukini kamaytiradi. Asosiy strategiyalar:

- **Cache-aside (lazy loading):** ilova cache'ni o'zi boshqaradi — avval cache, miss bo'lsa DB'dan olib cache'ga yozadi. Eng keng tarqalgan.
- **Write-through:** har yozishda cache va DB bir vaqtda yangilanadi. Cache doim yangi, lekin yozish sekinroq.
- **Write-back (write-behind):** avval cache'ga yoziladi, DB'ga keyinroq asinxron. Tez, lekin crash'da ma'lumot yo'qotish riski.

**Eviction va TTL:**
- **TTL (time-to-live)** — yozuv qancha vaqt yashaydi, keyin avtomatik o'chadi.
- **LRU (Least Recently Used)** — joy tugaganda eng kam ishlatilgan yozuvni chiqaradi. Redis'da `maxmemory-policy allkeys-lru`.

```ts
// cache-aside
async function getProduct(id: string) {
  const key = `product:${id}`;
  const hit = await redis.get(key);
  if (hit) return JSON.parse(hit);

  const product = await db.findProduct(id);
  await redis.set(key, JSON.stringify(product), "EX", 300); // TTL
  return product;
}
```

**Cache invalidation** — eng qiyin masala. Strategiyalar: TTL bilan tabiiy eskirish, yozishda `del` (cache-aside), yoki write-through bilan har doim yangi tutish.

**⚠️ Ehtiyot bo'l:** "There are only two hard things: cache invalidation and naming things." Stale data riski va hit rate orasida muvozanat toping. Qisqa TTL = yangiroq, lekin ko'p miss; uzun TTL = kam miss, lekin stale risk.

---

## Cache Stampede

**💡 Tushuncha:** **Cache stampede** (thundering herd) — mashhur kalit cache'dan o'chgan/expire bo'lgan zahoti, yuzlab so'rov bir vaqtda DB'ga uriladi (chunki barchasi miss oldi), DB'ni ezadi.

Yechimlar:
- **Lock / single-flight:** faqat bitta so'rov DB'ga boradi, qolganlari natijani kutadi.
- **Stale-while-revalidate:** eski qiymatni qaytarib, fonda yangilash.
- **Jittered TTL:** TTL'ga tasodifiy qo'shimcha (masalan 300 ± 30s) — barcha kalitlar bir vaqtda expire bo'lmaydi.
- **Probabilistic early expiration:** expire'ga yaqinlashganda ehtimollik bilan oldindan yangilash.

```ts
// single-flight lock bilan
async function getWithLock(key: string) {
  const hit = await redis.get(key);
  if (hit) return JSON.parse(hit);

  const lock = await redis.set(`lock:${key}`, "1", "NX", "EX", 5);
  if (lock) {
    const data = await db.fetch(key); // faqat lock olgan boradi
    await redis.set(key, JSON.stringify(data), "EX", 300);
    await redis.del(`lock:${key}`);
    return data;
  }
  await sleep(50);          // qolganlar kutadi
  return getWithLock(key);  // qayta urin
}
```

**⚠️ Ehtiyot bo'l:** Stampede ko'pincha katta trafikda kutilmaganda yuz beradi (mashhur post viral bo'ldi). Jittered TTL — eng arzon va samarali oldini olish usuli.

---

## Database Scaling

**💡 Tushuncha:** DB scaling usullari:

- **Replication:** master (yozish) + read replica'lar (o'qish). O'qishni replica'larga taqsimlab master yukini kamaytiradi. Replica'lar **asinxron** yangilanadi → replication lag (stale read).
- **Read replica:** o'qish nisbati yuqori tizimlarda (ko'p ilovalar 90%+ o'qish) juda samarali.
- **Sharding / partitioning:** ma'lumotni bir nechta DB'ga bo'lish. **Partitioning** bir DB ichida (table'larni bo'lish), **sharding** alohida serverlarga. Shard key tanlash (masalan `user_id`) muhim — notekis bo'lsa "hot shard" paydo bo'ladi.

```
yozish → [ Master ]
              │ replikatsiya (asinxron)
         ┌────┴────┐
    [Replica 1] [Replica 2]  ← o'qish
```

**⚠️ Ehtiyot bo'l:** Read replica'da **replication lag** bor — yangi yozilgan ma'lumot replica'da darhol ko'rinmasligi mumkin. "Read-after-write" muhim joylarda (foydalanuvchi o'z postini yaratib darhol ko'rishi) master'dan o'qing. Sharding'da **cross-shard query va JOIN** qiyinlashadi — shard key'ni access pattern'ga qarab tanlang.

---

## CAP Theorem

**💡 Tushuncha:** CAP teoremasi: distributed sistemada **network partition** (P) bo'lganda **Consistency** (C) va **Availability** (A) orasidan faqat bittasini tanlash mumkin.

- **CP** — partition'da consistency saqlanadi, availability qurbon (xato qaytaradi). Masalan: bank balansi, kuchli izchillik kerak joylar.
- **AP** — partition'da availability saqlanadi, consistency qurbon (eski/nomuvofiq ma'lumot qaytishi mumkin). Masalan: social feed, "like" soni.

Partition bo'lmaganda C va A ikkalasi ham bo'ladi — tanlash faqat partition vaqtida.

**Consistency modellar:**
- **Strong consistency** — har o'qish eng oxirgi yozishni ko'radi.
- **Eventual consistency** — vaqt o'tib barcha node'lar bir xil bo'ladi, lekin vaqtincha farq bo'lishi mumkin.
- **Read-your-writes** — foydalanuvchi hech bo'lmasa o'z yozganini ko'radi.

**⚠️ Ehtiyot bo'l:** "CAP'da C va A'dan birini tanlaymiz" deganda har doim P bo'lishini eslang — tarmoq partition'i muqarrar, shuning uchun amalda tanlov CP yoki AP orasida. Ko'p real tizimlar joyiga qarab ikkalasini ham aralashtiradi (PACELC kengaytmasini ham bilib qo'ying).

---

## CDN

**💡 Tushuncha:** CDN (Content Delivery Network) — statik kontentni (rasm, video, JS/CSS, hatto API javoblari) foydalanuvchiga geografik yaqin **edge** serverlardan tarqatadi. Latency tushadi, origin server yuki kamayadi.

CDN cache'ni TTL va cache header'lar (`Cache-Control`, `ETag`) boshqaradi. Dinamik kontentni cache qilmang yoki qisqa TTL bering.

**⚠️ Ehtiyot bo'l:** CDN'da cache invalidation sekin — yangi versiya chiqarganda eski kontent edge'larda qoladi. Yechim: **fayl nomida versiya/hash** (`app.a3f9.js`) — yangi nom = yangi cache, eskisi tabiiy expire bo'ladi.

---

## Message Queue

**💡 Tushuncha:** Message queue tizimni ikki muhim yo'l bilan yaxshilaydi:

- **Decoupling** — producer va consumer mustaqil; biri yiqilsa boshqasi to'xtamaydi.
- **Load leveling** — trafik tepasida xabarlar buferlanadi, consumer o'z barqaror tezligida ishlaydi. DB'ni spike'dan himoya qiladi.

Misol: rasm yuklash → queue'ga "process image" → worker'lar o'z tezligida thumbnail yaratadi. Foydalanuvchi kutmaydi, DB ezilmaydi.

**⚠️ Ehtiyot bo'l:** Queue ortib ketsa (consumer producer'dan sekin) **backlog** o'sadi — monitoring va consumer auto-scaling kerak. Dead-letter queue (DLQ) ni unutmang — qayta-qayta yiqilayotgan "poison" xabar boshqalarni bloklamasligi uchun.

---

## Rate Limiting

**💡 Tushuncha:** Rate limiting — mijoz/IP'ga vaqt birligida ruxsat etilgan so'rovlar sonini cheklash (abuse, DDoS, fair use). Asosiy algoritmlar:

- **Token bucket** — bucket tokenlar bilan to'ladi (sekundiga N ta), har so'rov bitta token "yeydi". Burst'ga ruxsat beradi (to'plangan tokenlar). Eng mashhur.
- **Leaky bucket** — so'rovlar navbatga tushadi va doimiy tezlikda "oqib chiqadi". Trafikni tekislaydi, burst'ni silliqlaydi.
- **Fixed window** — vaqt oynasi (masalan 1 daqiqa) ichidagi so'rovni sanaydi. Oddiy, lekin oyna chegarasida 2x burst muammosi.
- **Sliding window** — surilib boruvchi oyna, chegara muammosini hal qiladi, aniqroq.

```ts
// token bucket (Redis bilan soddalashtirilgan)
async function allow(userId: string, rate: number, capacity: number) {
  const now = Date.now();
  const key = `rl:${userId}`;
  const data = await redis.hgetall(key);
  let tokens = parseFloat(data.tokens ?? String(capacity));
  const last = parseFloat(data.ts ?? String(now));

  tokens = Math.min(capacity, tokens + ((now - last) / 1000) * rate); // to'ldirish
  if (tokens < 1) return false; // limit oshdi

  tokens -= 1;
  await redis.hmset(key, { tokens, ts: now });
  return true;
}
```

**⚠️ Ehtiyot bo'l:** Distributed sistemada rate limit holati **markazlashtirilgan** bo'lishi kerak (Redis), aks holda har instance alohida hisoblab haqiqiy limit N×instances bo'lib ketadi. Atomar bo'lishi uchun Redis Lua script yoki `INCR`+`EXPIRE` ishlating.

---

## Idempotency

**💡 Tushuncha:** Distributed sistemada retry va at-least-once delivery sababli bir so'rov ikki marta yetishi mumkin. Idempotent dizayn — bir yoki ko'p marta bajarish bir xil natija. `Idempotency-Key` orqali takroriy so'rovni aniqlab, saqlangan natijani qaytarish standart usul (Stripe modeli). To'lov, buyurtma kabi muhim yozishlarda majburiy.

**⚠️ Ehtiyot bo'l:** Idempotency keyni saqlash va asosiy operatsiyani **atomar** qiling (unique constraint/tranzaksiya), aks holda race condition'da ikkala so'rov ham "yangi" bo'lib ketadi. (Batafsil — Arxitektura Patternlar bo'limida.)

---

## Consistent Hashing

**💡 Tushuncha:** Oddiy `hash(key) % N` da node soni (N) o'zgarsa (server qo'shildi/o'chdi), deyarli barcha kalitlar boshqa node'ga ko'chadi — katta cache miss/rebalance. **Consistent hashing** kalitlar va node'larni xalqa (ring) ustiga joylashtiradi; node qo'shilganda/o'chganda faqat qo'shni segment kalitlari ko'chadi (≈ K/N).

**Virtual nodes** — har fizik node'ni ring'da ko'p nuqtaga joylab, taqsimotni tekislaydi (hot spot oldini oladi).

Qo'llanish: distributed cache (Memcached/Redis cluster), sharding, load balancing.

**⚠️ Ehtiyot bo'l:** Virtual node'larsiz oddiy consistent hashing notekis taqsimot beradi (ba'zi node'lar ko'p kalit oladi). Har fizik node uchun yetarli virtual node (masalan 100-200) qo'ying.

---

## Failure Dizayn

**💡 Tushuncha:** Distributed sistemada **failure normal holat** — tarmoq uziladi, servis yiqiladi. Mustahkamlik patternlari:

- **Timeout** — cheksiz kutmaslik; har tashqi chaqiruvga chegara. Bo'lmasa thread'lar to'lib qoladi.
- **Retry + backoff + jitter** — vaqtinchalik xatoda qayta urinish, lekin eksponensial backoff bilan (1s, 2s, 4s) va jitter bilan (bir vaqtda hammasi urinmasin). Faqat idempotent operatsiyalarda.
- **Circuit breaker** — servis qayta-qayta yiqilsa, "ochiq" holatga o'tib chaqiruvni darhol rad etadi (fast fail), vaqt o'tib "half-open"da sinab ko'radi. Cascading failure'ni to'xtatadi.
- **Bulkhead** — resurslarni izolyatsiya qilish (alohida thread pool/connection pool), bitta qism yiqilsa boshqasiga ta'sir qilmaydi (kema bo'lmalari kabi).

```ts
// circuit breaker holatlari
// CLOSED  → normal, chaqiruvlar o'tadi, xatolarni sanaydi
// OPEN    → chegara oshdi, chaqiruvlar darhol rad (fast fail)
// HALF_OPEN → vaqt o'tdi, bir nechta sinov chaqiruvi; muvaffaqiyat → CLOSED
```

**⚠️ Ehtiyot bo'l:** Retry'ni faqat **idempotent** operatsiyalarda ishlating — aks holda to'lov ikki marta bo'lishi mumkin. Backoff'siz retry "retry storm" keltiradi (yiqilgan servisga yana ko'proq yuk). Circuit breaker'siz timeout ham cascading failure'ni butunlay to'xtatmaydi.

---

## Monitoring va SLA/SLO

**💡 Tushuncha:** Atamalar:
- **SLI (Indicator)** — o'lchanadigan ko'rsatkich (latency p99, error rate, availability).
- **SLO (Objective)** — ichki maqsad (masalan "99.9% so'rov < 200ms").
- **SLA (Agreement)** — mijoz bilan shartnoma, buzilsa jarima/kompensatsiya.

**Latency**'ni o'rtacha (mean) emas, **percentile** bilan o'lchang: p50, p95, p99. O'rtacha tail latency'ni yashiradi — p99 = "100 so'rovdan eng yomon 1 tasi qancha kutadi".

**Availability "nine"lar:**

| Availability | Yiliga downtime |
|--------------|------------------|
| 99% | ~3.65 kun |
| 99.9% | ~8.8 soat |
| 99.99% | ~52 daqiqa |
| 99.999% | ~5 daqiqa |

**⚠️ Ehtiyot bo'l:** O'rtacha latency'ga ishonmang — 99 ta so'rov 10ms, 1 tasi 5s bo'lsa, o'rtacha "yaxshi" ko'rinadi, lekin p99 dahshatli. Foydalanuvchi tajribasi tail latency bilan belgilanadi. Alert'ni SLO buzilishiga (error budget) bog'lang, har bir spike'ga emas.

---

## Dizayn: URL Shortener

**Talablar:**
- Functional: uzun URL → qisqa kod; qisqa kod → redirect.
- Non-functional: o'qish >> yozish (100:1), past latency, yuqori availability.

**Capacity (taxminiy):** 100M yangi URL/oy ≈ 40 yozish/sek; o'qish 100x ≈ 4000 o'qish/sek. 5 yil ≈ 6B URL. 7 belgili base62 (62^7 ≈ 3.5 trillion) yetarli.

**API:**
```
POST /shorten   { url } → { shortCode }
GET  /{code}    → 301/302 redirect
```

**High-level:**
```
Client → LB → App servers → Cache (Redis) → DB
                              ↑ hit ko'pchilik o'qish
```

**Kod generatsiya — ikki yondashuv:**
- **Hash (MD5/SHA) + base62, kesib olish** — to'qnashuv (collision) bo'lishi mumkin, tekshirish kerak.
- **Counter + base62 (yaxshiroq)** — global o'suvchi counter'ni base62'ga aylantirish. To'qnashuvsiz, qisqa. Counter'ni shard qilish uchun har serverga blok ajratish (range allocation) yoki Snowflake-ga o'xshash ID.

```ts
function toBase62(n: bigint): string {
  const chars = "0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ";
  let s = "";
  while (n > 0n) { s = chars[Number(n % 62n)] + s; n /= 62n; }
  return s || "0";
}
```

**O'qish yo'li:** `GET /{code}` → Redis'da qidir (hit bo'lsa darhol redirect) → miss bo'lsa DB → cache'ga yoz → redirect. O'qish ko'p bo'lgani uchun cache hit rate yuqori bo'ladi.

**Trade-off'lar:** 301 (permanent, brauzer cache qiladi — analytics yo'qoladi) vs 302 (temporary, har safar serverga keladi — analytics bor, lekin yuk ko'p). DB tanlovi: yozish-o'qish nisbati va scale uchun key-value (DynamoDB) yoki Postgres + read replica.

---

## Dizayn: Rate Limiter

**Talablar:** Har foydalanuvchi/IP/API key uchun "N so'rov / vaqt birligi" cheklash; distributed (ko'p app instance); past latency (har so'rovda chaqiriladi).

**Joylashuv:** API Gateway'da yoki middleware sifatida. Holat **markazlashgan Redis**'da (instance'lar bo'lsa bham yagona hisob).

**Algoritm tanlovi:** Token bucket — burst'ga ruxsat (real foydalanuvchi tabiati) va sodda. Sliding window log — aniqroq, lekin xotira ko'proq.

```ts
// Redis INCR + EXPIRE bilan fixed window (sodda variant)
async function allow(key: string, limit: number, windowSec: number) {
  const count = await redis.incr(key);
  if (count === 1) await redis.expire(key, windowSec); // birinchi so'rovda TTL
  return count <= limit;
}
```

**Atomarlik:** `INCR`+`EXPIRE`'ni atomar qilish uchun Lua script (aks holda EXPIRE qo'yilmasdan crash bo'lsa kalit abadiy qoladi). Token bucket uchun ham Lua bilan to'ldirish va kamaytirishni bir atomar qadamga jamlang.

**Javob:** Limit oshganda `429 Too Many Requests` + `Retry-After` header. `X-RateLimit-Remaining` bilan mijozga qolgan limitni ayting.

**Trade-off'lar:** Fixed window oddiy, lekin oyna chegarasida 2x burst. Sliding window aniqroq, lekin og'irroq. Redis SPOF bo'lmasligi uchun replica/cluster.

---

## Dizayn: Notification / Feed

**Talablar:** Foydalanuvchilarga push/email/SMS yuborish; yuqori throughput; ishonchli yetkazib berish; foydalanuvchi feed'ini ko'rsatish.

**Notification arxitekturasi:**
```
Event manbai → Queue (Kafka/SQS) → Worker'lar → kanal adapterlar (push/email/SMS)
                                         ↓
                                    retry + DLQ
```
Queue **decoupling va load leveling** beradi — spike'da xabarlar buferlanadi, worker'lar o'z tezligida ishlaydi. Har kanal alohida adapter; idempotency key bilan dublikat yuborishdan saqlanish.

**Feed — fan-out modellari:**
- **Fan-out on write (push):** post yaratilganda barcha follower'larning feed'iga oldindan yoziladi. O'qish tez (feed tayyor), lekin "celebrity problem" — millionlab follower'ga yozish qimmat.
- **Fan-out on read (pull):** feed so'ralganda follower'lar postlarini real vaqtda yig'adi. Yozish arzon, o'qish qimmat.
- **Hybrid:** oddiy foydalanuvchi push, mashhur (celebrity) hisoblar pull. Eng amaliy yondashuv.

**⚠️ Ehtiyot bo'l (celebrity problem):** Million follower'li hisob post yozsa, fan-out-on-write million yozish keltiradi. Shunday hisoblarni pull modeliga o'tkazing — follower feed so'raganda ularning postini qo'shib qo'ying.

**Yetkazib berish:** at-least-once + idempotent consumer, retry + backoff, qayta-qayta yiqilgan xabar uchun DLQ. Yuborilgan/ochilgan holatni kuzatish uchun tracking.

---

## Intervyu Savollari

### ❓ System design intervyusini qaysi qadamdan boshlaysiz?

**✅ Javob:** Talablarni aniqlashdan — functional va non-functional. Darhol chizmaslik kerak; savol berib scope'ni toraytirib, assumption'larni ovoz chiqarib aytaman. Keyin capacity baholash (QPS, storage), API dizayn, high-level diagramma, deep dive, va oxirida bottleneck/trade-off. Intervyuer tayyor javobni emas, fikrlash jarayonini ko'rmoqchi.

### ❓ Vertical va horizontal scaling farqi va qaysi biri afzal?

**✅ Javob:** Vertical — bitta mashinani kuchaytirish (oddiy, lekin chegarali va SPOF). Horizontal — ko'proq mashina (deyarli cheksiz, fault-tolerant, lekin distributed murakkablik). Katta tizimlar uchun horizontal afzal, lekin u ilova **stateless** bo'lishini talab qiladi — holat lokal emas, Redis/DB'da. Amalda avval vertical (oddiy), keyin horizontal.

### ❓ L4 va L7 load balancer farqi nimada?

**✅ Javob:** L4 — transport (TCP/UDP) darajasida, IP/port'ga qarab marshrutlaydi; tez, lekin so'rov mazmunini ko'rmaydi. L7 — application (HTTP) darajasida, URL/header/cookie'ga qarab marshrutlaydi; path-based routing, SSL termination kabi aqlli imkoniyatlar, lekin biroz sekinroq. Mikroservis routing va A/B uchun L7, oddiy yuqori-throughput uchun L4.

### ❓ Cache-aside, write-through va write-back farqi?

**✅ Javob:** **Cache-aside** — ilova cache'ni boshqaradi, miss'da DB'dan olib yozadi (eng keng tarqalgan). **Write-through** — yozishda cache va DB birga yangilanadi, cache doim yangi lekin yozish sekin. **Write-back** — avval cache, DB'ga keyinroq asinxron; tez, lekin crash'da yo'qotish riski. Tanlov: o'qish-og'ir → cache-aside; izchillik muhim → write-through.

### ❓ Cache stampede nima va qanday oldini olasiz?

**✅ Javob:** Mashhur kalit expire bo'lgan zahoti yuzlab so'rov bir vaqtda DB'ga uriladi (hammasi miss oldi), DB ezadi. Oldini olish: single-flight lock (faqat bittasi DB'ga boradi), stale-while-revalidate (eski qiymat + fonda yangilash), jittered TTL (tasodifiy qo'shimcha bilan bir vaqtda expire bo'lmaslik), probabilistic early expiration. Jittered TTL eng arzon va samarali.

### ❓ Read replica nima beradi va qanday muammosi bor?

**✅ Javob:** Read replica o'qishni master'dan replica'larga taqsimlab master yukini kamaytiradi — o'qish-og'ir tizimlarda (ko'p ilova 90%+ o'qish) juda samarali. Muammo: replica'lar asinxron yangilanadi → **replication lag** (yangi yozilgan ma'lumot replica'da darhol ko'rinmaydi). "Read-after-write" muhim joyda master'dan o'qish kerak.

### ❓ Sharding nima va shard key qanday tanlanadi?

**✅ Javob:** Sharding — ma'lumotni bir nechta alohida DB serverga bo'lish (horizontal partitioning). Shard key (masalan `user_id`) ma'lumot qaysi shard'ga tushishini belgilaydi. To'g'ri tanlash muhim: notekis taqsimot "hot shard" keltiradi; cross-shard query/JOIN qiyinlashadi. Access pattern'ga mos key tanlang (eng ko'p qaysi maydon bo'yicha so'raladi).

### ❓ CAP teoremasini tushuntiring. CP va AP qachon tanlanadi?

**✅ Javob:** Partition (P) bo'lganda Consistency va Availability'dan faqat bittasini tanlash mumkin. **CP** — consistency muhim joyda (bank balansi), partition'da xato qaytaradi lekin nomuvofiq ma'lumot bermaydi. **AP** — availability muhim joyda (social feed, like soni), partition'da ishlashda davom etadi lekin eski ma'lumot ko'rsatishi mumkin. Partition muqarrar bo'lgani uchun amalda tanlov CP yoki AP.

### ❓ Eventual consistency nima va qachon maqbul?

**✅ Javob:** Eventual consistency — yozishdan keyin barcha node'lar vaqt o'tib bir xil bo'ladi, lekin vaqtincha farq bo'lishi mumkin. Strong consistency'dan tez va available, lekin o'qish stale bo'lishi mumkin. Maqbul: like soni, view soni, feed kabi vaqtincha nomuvofiqlik muhim bo'lmagan joylar. Maqbul emas: pul balansi, inventar (oversell xavfi).

### ❓ Token bucket va leaky bucket farqi?

**✅ Javob:** **Token bucket** — bucket tokenlar bilan to'ladi (N/sek), har so'rov token "yeydi"; to'plangan tokenlar tufayli **burst'ga ruxsat** beradi. **Leaky bucket** — so'rovlar navbatga tushib doimiy tezlikda "oqib chiqadi"; burst'ni **silliqlaydi**, tekis chiqish beradi. Burst ruxsat kerak bo'lsa token bucket; barqaror chiqish kerak bo'lsa leaky bucket.

### ❓ Distributed rate limiter'da qanday muammo bor?

**✅ Javob:** Agar har app instance limitni o'zida (lokal) hisoblasa, haqiqiy limit N×instances bo'lib ketadi. Yechim: holatni **markazlashgan Redis**'da saqlash. Yana atomarlik muhim — `INCR`+`EXPIRE` yoki token to'ldirish/kamaytirishni Redis Lua script bilan atomar qiling, aks holda race condition'da hisob noto'g'ri bo'ladi.

### ❓ Consistent hashing nima muammoni hal qiladi?

**✅ Javob:** Oddiy `hash % N`da node soni o'zgarsa deyarli barcha kalitlar ko'chadi (katta rebalance/cache miss). Consistent hashing kalit va node'larni ring'ga joylab, node qo'shilganda/o'chganda faqat ≈ K/N kalit ko'chadi. Virtual node'lar taqsimotni tekislaydi (hot spot oldini oladi). Distributed cache, sharding, LB'da ishlatiladi.

### ❓ Circuit breaker qanday ishlaydi?

**✅ Javob:** Uch holat: **Closed** (normal, xatolarni sanaydi), **Open** (xato chegarasi oshdi, chaqiruvlar darhol rad etiladi — fast fail), **Half-open** (vaqt o'tgach bir nechta sinov chaqiruvi; muvaffaqiyat bo'lsa Closed'ga qaytadi). Maqsad — yiqilgan servisga yuk yog'dirmaslik va cascading failure'ni to'xtatish. Foydalanuvchi cheksiz kutish o'rniga tez xato/fallback oladi.

### ❓ Retry'ni qachon ishlatish XAVFLI?

**✅ Javob:** Non-idempotent operatsiyalarda — masalan to'lov yoki "balansga qo'shish". Retry ikki marta bajarib qo'yishi mumkin. Faqat idempotent operatsiyalarda retry qiling, yoki idempotency key bilan himoyalang. Bundan tashqari backoff'siz retry "retry storm" — yiqilgan servisga yana ko'proq yuk; har doim eksponensial backoff + jitter ishlating.

### ❓ Latency'ni nega o'rtacha emas, percentile bilan o'lchaysiz?

**✅ Javob:** O'rtacha (mean) tail latency'ni yashiradi — 99 so'rov 10ms, 1 tasi 5s bo'lsa o'rtacha "yaxshi" ko'rinadi, lekin har 100-foydalanuvchidan biri dahshatli tajriba oladi. p95/p99 "eng yomon 5%/1%" qancha kutishini ko'rsatadi. Foydalanuvchi tajribasi va SLO tail latency bilan belgilanadi.

### ❓ URL shortener'da kodni qanday generatsiya qilasiz?

**✅ Javob:** Ikki yo'l: (1) hash (MD5/SHA) + base62 kesib olish — to'qnashuv bo'lishi mumkin, tekshirish kerak; (2) global counter + base62 — to'qnashuvsiz, qisqa, afzalroq. Counter'ni scale qilish uchun har serverga ID bloki ajratish (range) yoki Snowflake-ga o'xshash distributed ID. 7 belgili base62 (~3.5 trillion) yillar uchun yetarli.

### ❓ Feed dizaynida "celebrity problem" nima va qanday hal qilasiz?

**✅ Javob:** Fan-out-on-write'da post yaratilganda barcha follower feed'iga yoziladi. Million follower'li hisob bitta post yozsa, million yozish kerak — qimmat va sekin. Yechim: **hybrid** — oddiy foydalanuvchilar uchun push (fan-out on write), mashhur hisoblar uchun pull (follower feed so'raganda celebrity postini qo'shib qo'yish). Shunday qilib yozish portlashi oldini olinadi.

---

## Masalalar

> Yechimlar: [solutions/backend/10-backend-system-design.md](../solutions/backend/10-backend-system-design.md)

1. **Capacity baholash.** "Instagram-ga o'xshash" servis: 500M foydalanuvchi, kuniga 100M post, har post o'rtacha 5 ta rasm (har biri 2MB). Hisoblang: kunlik yozish QPS, peak QPS (3x deb oling), yangi storage/kun, 5 yillik storage.

2. **Caching strategiyasi tanlash.** Quyidagi 3 holat uchun cache strategiyasini (cache-aside / write-through / write-back) tanlang va asoslang: (a) mahsulot katalogi (kam yoziladi, ko'p o'qiladi), (b) foydalanuvchi sessiyalari, (c) real-time o'yin skorlari (juda tez-tez yoziladi).

3. **Rate limiter dizayni.** "1000 so'rov/soat per API key" rate limiter'ni distributed muhitda (5 ta app instance, Redis bilan) loyihalashtiring. Algoritm tanlang, atomarlikni qanday ta'minlashni va `429` javobini tushuntiring.

4. **CAP qarori.** Quyidagilar uchun CP yoki AP tanlang va sababini yozing: (a) bank pul o'tkazmasi, (b) Twitter like soni, (c) e-commerce inventar (stok), (d) DNS.

5. **URL shortener deep dive.** URL shortener'da kod generatsiya uchun "counter + base62" yondashuvini distributed muhitda qanday scale qilasiz (to'qnashuvsiz, single point of failure'siz)? Kamida ikki variant taklif qiling.

6. **Cache stampede.** Mashhur mahsulot sahifasi (kuniga 1M ko'rinish) cache'i expire bo'lganda DB ezilmasligi uchun stampede'dan himoya dizayn qiling. Single-flight va jittered TTL'ni birga ishlating.

7. **DB scaling rejasi.** Tez o'sayotgan ilova: avval bitta Postgres, o'qish 95%, yozish 5%. Bosqichma-bosqich scaling rejasini yozing (qachon read replica, qachon sharding) va har bosqichning trade-off'ini ko'rsating.

8. **Notification system.** 10M foydalanuvchiga push notification yuboruvchi tizim loyihalashtiring: queue, worker, retry, DLQ, idempotency. Throughput spike'da (bir vaqtda 1M notification) qanday load leveling qilishni tushuntiring.

---

← [Backend bo'limiga qaytish](./README.md)
