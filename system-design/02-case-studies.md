# System Design Case Study'lar — Klassik Loyihalar

System design intervyusida eng ko'p so'raladigan klassik tizimlarni boshidan oxirigacha loyihalash bu yerda. Har bir case study **bir xil 7-bosqichli tuzilma** bilan yoritiladi: (1) Requirements, (2) Capacity estimation, (3) API design, (4) High-level arxitektura, (5) Data model, (6) Deep dive, (7) Bottleneck/trade-off/scaling. Maqsad — yodlash emas, balki **fikrlash jarayonini** ko'rsatish: talabni torting, raqam bilan baholang, trade-off'ni asoslang.

Eng muhim uchtasi (URL Shortener, Twitter feed, Chat) chuqurroq; qolganlari qisqaroq, lekin baribir to'liq tuzilmada. Oxirida system design'ning umumiy savollari (Q&A) va o'zingiz loyihalaydigan masalalar bor.

## Mundarija

- [1. URL Shortener (TinyURL)](#1-url-shortener)
- [2. Twitter / X — News Feed](#2-twitter-news-feed)
- [3. Chat tizimi (WhatsApp)](#3-chat-whatsapp)
- [4. Rate Limiter (distributed)](#4-rate-limiter)
- [5. Uber / Yetkazib berish](#5-uber)
- [6. Instagram / Video streaming](#6-instagram-video)
- [Umumiy savollar (Q&A)](#qa)
- [Masalalar](#masalalar)

---

<a id="1-url-shortener"></a>
## 1. URL Shortener (TinyURL)

Uzun URL'ni qisqa kodga aylantiruvchi servis: `https://example.com/very/long/path?x=1` → `https://sho.rt/aB3xK9`. Klassik "warm-up" case — kichik ko'rinadi, lekin ID generatsiya, redirect va scale'da chuqur savollar bor.

### 1.1 Requirements

**Functional:**
- Uzun URL → qisqa URL (`POST /shorten`).
- Qisqa URL → asl URL'ga redirect (`GET /{code}`).
- Ixtiyoriy: custom alias (`sho.rt/mybrand`), expiration (TTL), analytics (klik soni).

**Non-functional:**
- **Yuqori availability** — redirect ishlamasligi mijoz uchun "link buzilgan" degani.
- **Past latency** — redirect < 50ms (foydalanuvchi kutmaydi).
- **O'qish-og'ir** — read:write ≈ 100:1 (link bir marta yaratiladi, ming marta bosiladi).
- **Kodlar takrorlanmasin** (no collision), iloji boricha qisqa.

**💡 Tushuncha:** Boshida aniqlovchi savol bering: "Custom alias kerakmi? Linklar muddatsizmi yoki expire bo'ladimi? Analytics kerakmi?" Scope'ni torting — bu intervyuda baholanadigan birinchi ko'nikma.

### 1.2 Capacity estimation

Faraz: **100M yangi URL/oy**.

```text
Yozish (write):
  100M / oy ÷ 30 kun ÷ 86 400 sek ≈ 40 yozish/sek (o'rtacha)
  Peak (3x)                        ≈ 120 yozish/sek

O'qish (read), 100:1 nisbat:
  40 × 100 = 4 000 o'qish/sek (o'rtacha)
  Peak                  ≈ 12 000 o'qish/sek

Storage (5 yil):
  100M/oy × 12 × 5 = 6 mlrd URL
  Har yozuv ≈ 500 bayt (kod + uzun URL + metadata)
  6e9 × 500B ≈ 3 TB   → bitta DB sig'ar, lekin replikatsiya + index bilan ~10 TB

Bandwidth:
  Yozish: 120/sek × 500B ≈ 60 KB/sek — arzimas
  O'qish: 12 000/sek × 500B ≈ 6 MB/sek — yengil
```

**Xulosa:** Storage va bandwidth muammo emas. Asosiy yuk — **redirect o'qish QPS** va **past latency**. Demak cache (Redis) markaziy rol o'ynaydi.

**Kod uzunligi:** base62 (`a-z A-Z 0-9` = 62 belgi) ishlatsak:
```text
62^6 ≈ 56 mlrd   — 6 belgi bizning 6 mlrd uchun yetarli
62^7 ≈ 3.5 trln  — 7 belgi yillar uchun zaxira bilan
```
7 belgili kod tanlaymiz.

### 1.3 API design

```text
POST /api/v1/shorten
  Body: { "longUrl": "https://...", "customAlias?": "mybrand", "ttlDays?": 30 }
  200:  { "shortUrl": "https://sho.rt/aB3xK9", "code": "aB3xK9" }
  409:  custom alias band bo'lsa

GET /{code}
  301/302 → Location: <longUrl>   (redirect)
  404: kod topilmasa

DELETE /api/v1/{code}    (egasi o'chiradi)
GET    /api/v1/{code}/stats  (klik analytics)
```

**⚠️ Ehtiyot bo'l:** **301 vs 302** — 301 (Permanent) brauzerda kesh'lanadi, demak keyingi kliklar serveringizga **kelmaydi** (analytics yo'qoladi, lekin yuk kamayadi). 302 (Found) har klikda serverga keladi (analytics ishlaydi, yuk ko'p). Analytics kerak bo'lsa **302** tanlang.

### 1.4 High-level arxitektura

```text
                    ┌──────────────┐
   Client  ───────▶ │ Load Balancer│
                    └──────┬───────┘
                           │
                  ┌────────┴────────┐
                  │   App servers   │  (stateless, horizontal scale)
                  │  /shorten /{code}│
                  └───┬────────┬─────┘
                      │        │
            cache miss│        │ write path
                      ▼        ▼
              ┌──────────┐  ┌──────────────┐
              │  Redis   │  │  ID Generator│ (counter / Snowflake)
              │ (LRU TTL)│  └──────┬───────┘
              └────┬─────┘         │
                   │ miss → DB     ▼
                   ▼        ┌──────────────┐
              ┌──────────────┐ │  Database   │ (sharded by code)
              │   Database    │◀┘ code → longUrl
              └──────────────┘
```

Redirect oqimi: `GET /{code}` → Redis'da bormi? → bor: darhol redirect. Yo'q: DB'dan o'qi, Redis'ga yoz (cache-aside), redirect.

### 1.5 Data model

```text
Table: urls
  code        VARCHAR(7)  PRIMARY KEY   -- aB3xK9
  long_url    TEXT        NOT NULL
  created_at  TIMESTAMP
  expires_at  TIMESTAMP   NULL
  user_id     BIGINT      NULL
  click_count BIGINT      DEFAULT 0     -- yoki alohida analytics store

Index: code (PK, lookup), expires_at (cleanup job uchun)
```

NoSQL (DynamoDB/Cassandra) ham mos — bizga faqat key-value lookup (`code → longUrl`) kerak, JOIN yo'q. Sharding `code` bo'yicha (consistent hashing).

### 1.6 Deep dive — kod generatsiya va collision

Eng muhim savol: **qisqa kodni qanday yaratamiz?**

**Variant A — Hash (MD5/SHA + base62):**
```js
const crypto = require("crypto");
function hashCode(longUrl) {
  const hash = crypto.createHash("md5").update(longUrl).digest("hex");
  return base62(BigInt("0x" + hash)).slice(0, 7); // dastlabki 7 belgi
}
```
- ➕ Stateless, hech qanday counter kerak emas, bir xil URL → bir xil kod.
- ➖ **Collision** — turli URL bir xil 7 belgi berishi mumkin (kesib olganimiz uchun). Yozishdan oldin DB'da kod bormi tekshirish kerak; bor bo'lsa salt qo'shib qayta hash. Bu qo'shimcha DB lookup.

**Variant B — Counter + base62 (afzal):**
```js
// global auto-increment ID → base62
function encode(id) {
  const chars = "0123456789abcdefghijklmnopqrstuvwxyzABCDEF...XYZ"; // 62 belgi
  let s = "";
  while (id > 0) { s = chars[id % 62n] + s; id /= 62n; }
  return s.padStart(7, "0");
}
// id=125 → "00000cb", id=1e9 → "015ftgg"
```
- ➕ **Collision YO'Q** (har ID noyob), qisqa, tekshiruv kerak emas.
- ➖ Global counter — single point of failure va bottleneck bo'lishi mumkin.

**Counter'ni distributed qilish:**

| Yondashuv | Qanday | Trade-off |
|-----------|--------|-----------|
| **ID range allocation** | Har app server markaziy servisdan blok oladi (masalan 1–1000, 1001–2000), lokal sarflaydi | Kam koordinatsiya; server o'lsa o'sha blok yo'qoladi (zarar yo'q) |
| **Snowflake ID** | 64-bit: timestamp + machine_id + sequence | Vaqt tartibida, markazsiz; clock skew'ga ehtiyot bo'l |
| **Redis INCR** | Atomar `INCR` global counter | Oddiy, lekin Redis SPOF — replikatsiya kerak |

**💡 Tushuncha:** Intervyuda **ID range allocation** odatda eng yaxshi javob — markaziy bottleneck'ni yo'qotadi va to'qnashuvsiz. "Counter + base62 + range allocation" deganingizda senior fikrlash ko'rsatasiz.

### 1.7 Bottleneck / trade-off / scaling

- **Redirect QPS** — Redis cache 95%+ hit beradi (mashhur linklar). Cache'siz DB 12k QPS'ni o'qiy oladi, lekin latency oshadi.
- **Hot key** — viral link bitta Redis node'ni ezishi mumkin → CDN edge cache yoki client-side 301 cache.
- **DB scaling** — `code` bo'yicha sharding; lookup bitta shard'ga tushadi (no cross-shard query).
- **Expiration** — TTL'li linklar uchun background cleanup job (`DELETE WHERE expires_at < now`), Redis'da TTL avtomatik.
- **Analytics** — klik eventlarini sinxron yozmang; Kafka'ga push qilib, alohida pipeline'da agregatsiya qiling (redirect latency'ga ta'sir qilmasin).

---

<a id="2-twitter-news-feed"></a>
## 2. Twitter / X — News Feed

Foydalanuvchi follow qilganlar postlarini xronologik (yoki ranked) ko'radigan feed. Asosiy qiyinlik — **timeline generatsiya** va **celebrity (fan-out) muammosi**.

### 2.1 Requirements

**Functional:**
- Post (tweet) yozish.
- O'zi follow qilganlar feed'ini ko'rish (home timeline).
- Foydalanuvchini follow / unfollow qilish.
- Ixtiyoriy: like, retweet, reply.

**Non-functional:**
- **O'qish-og'ir** — feed ko'rish post yozishdan ~100x ko'p.
- **Past latency** feed yuklashda (< 200ms).
- **Eventual consistency** maqbul — postingiz follower'ga 1-2 sekund kechikib chiqsa, dunyo yiqilmaydi.
- **Yuqori availability**.

### 2.2 Capacity estimation

Faraz: **300M DAU**, har biri kuniga 2 post, kuniga 50 marta feed ochadi.

```text
Yozish (post):
  300M × 2 = 600M post/kun ÷ 86 400 ≈ 7 000 post/sek
  Peak (3x)                          ≈ 21 000 post/sek

O'qish (feed):
  300M × 50 = 15 mlrd feed-ochish/kun ÷ 86 400 ≈ 173 000 feed/sek
  Peak                                          ≈ 520 000 feed/sek

→ Read:Write ≈ 25:1 (feed-ochish) yoki post o'qish bo'yicha 100:1+

Storage (post):
  600M post/kun × 300 bayt (matn + metadata) ≈ 180 GB/kun
  5 yil ≈ 330 TB (faqat matn; media alohida object storage'da)
```

**Xulosa:** O'qish hukmron (520k feed/sek peak). Feed'ni har so'rovda generatsiya qilish (pull) qimmat — **precompute** kerak.

### 2.3 API design

```text
POST /api/v1/tweet
  Body: { "text": "...", "mediaIds?": [...] }
  201:  { "tweetId": "...", "createdAt": "..." }

GET /api/v1/feed?cursor=<id>&limit=20
  200:  { "tweets": [...], "nextCursor": "..." }   -- cursor pagination

POST   /api/v1/follow/{userId}
DELETE /api/v1/follow/{userId}
```

**⚠️ Ehtiyot bo'l:** Feed'da **offset pagination** (`?page=5`) ishlatmang — yangi postlar qo'shilganda elementlar siljiydi (duplicate/skip). **Cursor pagination** (oxirgi ko'rilgan tweet ID) ishlating.

### 2.4 High-level arxitektura

```text
              ┌──────────────┐
   Client ───▶│ Load Balancer│
              └──────┬───────┘
          ┌──────────┴───────────┐
          │                      │
   ┌──────▼──────┐        ┌──────▼───────┐
   │ Write API   │        │  Read API    │
   │ (post)      │        │  (feed)      │
   └──────┬──────┘        └──────┬───────┘
          │ yangi post           │ feed o'qi
          ▼                      ▼
   ┌─────────────┐        ┌──────────────┐
   │ Fan-out svc │───────▶│ Feed cache   │ (Redis: user→[tweetIds])
   │ (worker)    │        │ "timeline"   │
   └──────┬──────┘        └──────┬───────┘
          │                      │ tweet detallari
          ▼                      ▼
   ┌─────────────┐        ┌──────────────┐
   │ Tweet DB    │        │  Tweet store │ (id→tweet)
   │ (sharded)   │        │  + cache     │
   └─────────────┘        └──────────────┘
          │
          ▼
   ┌─────────────┐
   │ Graph DB    │  (follow munosabati: who follows whom)
   └─────────────┘
```

### 2.5 Data model

```text
tweets:        tweet_id (PK, snowflake) | user_id | text | media | created_at
follows:       follower_id | followee_id | created_at   (index: followee_id)
user_timeline: user_id → [tweet_id, ...]   (o'zi yozgan postlar, sharded)
home_timeline: user_id → [tweet_id, ...]   (Redis list, precomputed feed)
```

`tweet_id` Snowflake bo'lsa — vaqt tartibida sort qilingan, cursor pagination uchun ideal (ID = vaqt tartibi).

### 2.6 Deep dive — Fan-out: write vs read

Bu case study'ning **yuragi**. Feed'ni qanday yig'amiz?

**Yondashuv 1 — Fan-out on READ (pull):**
Foydalanuvchi feed so'raganda: follow qilganlarini top → har birining oxirgi postlarini o'qi → merge + sort.
```text
feed(user) = merge_sort( top_tweets(f) for f in follows(user) )
```
- ➕ Yozish arzon (faqat tweet DB'ga 1 yozish).
- ➖ **O'qish qimmat** — 1000 follow qilgan bo'lsa, har feed-ochishda 1000 query + merge. 520k feed/sek'da o'ladi.

**Yondashuv 2 — Fan-out on WRITE (push):**
Post yozilganda darhol **barcha follower'ning home_timeline'iga** tweet_id yoziladi (precompute).
```text
on_tweet(user, tweet):
    for f in followers(user):
        redis.lpush("timeline:" + f, tweet.id)
```
- ➕ **O'qish juda tez** — feed allaqachon tayyor, faqat Redis list o'qiladi (O(1)).
- ➖ Yozish qimmat — million follower bo'lsa, 1 post = million yozish.

**Solishtirish:**

| Mezon | Fan-out on Read (pull) | Fan-out on Write (push) |
|-------|------------------------|--------------------------|
| Feed latency | Sekin (runtime merge) | Tez (precomputed) |
| Yozish narxi | Arzon | Qimmat (follower soniga) |
| Mos kim uchun | Kam follower, kam aktiv | Ko'p o'qiydigan, oddiy user |
| Asosiy muammo | O'qishda yuk | **Celebrity** (million follower) |

### 2.6.1 Celebrity problem (hybrid yechim)

**Muammo:** 100M follower'li hisob (Ronaldo, Elon) bitta post yozsa — push yondashuvda 100M Redis yozish kerak. Bu sekin, qimmat va "yozish portlashi" (write amplification).

**✅ Yechim — Hybrid (push + pull):**
- **Oddiy foydalanuvchilar** (kam follower) uchun: **push** (fan-out on write) — feed'lariga oldindan yoziladi.
- **Celebrity'lar** (threshold'dan ko'p follower, masalan > 100k) uchun: **push qilinmaydi**. Feed o'qilganda celebrity postlari **runtime'da pull** qilinib, precomputed feed bilan merge qilinadi.

```text
get_feed(user):
    base = redis.lrange("timeline:" + user, 0, 20)   # push'langan oddiy postlar
    celebs = followed_celebrities(user)
    celeb_tweets = [latest_tweets(c) for c in celebs]  # runtime pull
    return merge_sort(base + celeb_tweets)[:20]
```

**💡 Tushuncha:** Hybrid eng yaxshi javob. "Push for the masses, pull for the celebrities" — yozish portlashini oldini oladi, o'qish baribir tez. Threshold (kim "celebrity") konfiguratsiyalanadi.

### 2.7 Bottleneck / trade-off / scaling

- **Fan-out worker** — post yozilganda fan-out queue'ga (Kafka) tushadi, worker'lar follower feed'iga asinxron yozadi (yozish latency'ni post API'dan ajratadi).
- **Feed cache (Redis)** — har user uchun home_timeline list (oxirgi ~800 tweet ID). Eski feed evict, so'ralganda regenerate.
- **Inactive users** — 6 oy kirmagan user feed'ini precompute qilmang (resurs isrof). Login'da regenerate qiling.
- **Hot tweet store** — mashhur tweet detallari Redis'da cache (1 tweet ID → 1M user o'qishi mumkin).
- **Sharding** — tweet DB `user_id` bo'yicha; graph DB (follows) alohida; feed cache `user_id` bo'yicha.

---

<a id="3-chat-whatsapp"></a>
## 3. Chat tizimi (WhatsApp)

Real-time 1:1 va guruh xabar almashinuvi. Asosiy qiyinliklar: **real-time delivery (WebSocket)**, **message ordering**, **delivery status** (sent/delivered/read), **online presence**.

### 3.1 Requirements

**Functional:**
- 1:1 xabar yuborish/qabul qilish (real-time).
- Guruh chat.
- Delivery status: ✓ (sent), ✓✓ (delivered), ✓✓ ko'k (read).
- Online/last-seen status.
- Offline'da xabar saqlanib, online bo'lganda yetkaziladi.

**Non-functional:**
- **Past latency** — xabar ~100ms ichida yetsin.
- **Message ordering** kafolati (suhbat ichida).
- **Durability** — yetkazilmagan xabar yo'qolmasin.
- **Yuqori availability**, 2 mlrd user scale.

### 3.2 Capacity estimation

Faraz: **2 mlrd user, 500M DAU**, har biri kuniga 40 xabar.

```text
Xabarlar:
  500M × 40 = 20 mlrd xabar/kun ÷ 86 400 ≈ 230 000 xabar/sek
  Peak (3x)                              ≈ 700 000 xabar/sek

Connections (WebSocket):
  500M concurrent connection (peak'da ~katta ulush)
  Bir server ~65k conn (port limiti emas, resurs limiti) → ~10k+ chat server

Storage:
  20 mlrd × 200 bayt ≈ 4 TB/kun (matn)
  Media alohida object storage'da
```

**Xulosa:** Asosiy challenge — **millionlab doimiy WebSocket connection** boshqaruvi va **xabar marshrutlash** (qaysi server qaysi user'ni ushlab turibdi).

### 3.3 API / protocol

REST emas, asosan **WebSocket** (yoki MQTT — WhatsApp aynan MQTT ishlatadi, yengil, mobil-do'st).

```text
WebSocket (persistent connection):
  → SEND   { type:"msg", to:"userB", text:"salom", clientMsgId:"uuid" }
  ← ACK    { type:"ack", clientMsgId:"uuid", serverMsgId:"...", status:"sent" }
  ← MSG    { type:"msg", from:"userA", text:"...", serverMsgId:"..." }
  → READ   { type:"read", serverMsgId:"..." }     -- read receipt
  ← STATUS { type:"status", serverMsgId:"...", status:"delivered|read" }

REST (yon funksiyalar):
  POST /media/upload   → mediaUrl
  GET  /messages/history?chatId=&cursor=
```

### 3.4 High-level arxitektura

```text
   ┌────────┐   WebSocket    ┌──────────────┐
   │ Client │◀══════════════▶│ Chat server  │ (millionlab conn ushlaydi)
   │  (A)   │                │  (stateful)  │
   └────────┘                └──────┬───────┘
                                    │ "userB qayerda?"
                                    ▼
                            ┌──────────────┐
                            │ Presence /   │  user_id → chat_server_id
                            │ Session store│  (Redis)
                            └──────┬───────┘
                                   │ route to B's server
                                   ▼
   ┌────────┐   WebSocket    ┌──────────────┐
   │ Client │◀══════════════▶│ Chat server  │
   │  (B)   │                │  (B ushlagan)│
   └────────┘                └──────┬───────┘
                                    │ persist + offline
                                    ▼
                            ┌──────────────┐    ┌──────────────┐
                            │  Message DB  │    │ Message queue│
                            │ (Cassandra)  │    │ (offline B)  │
                            └──────────────┘    └──────────────┘
```

A → A'ning chat server'i → session store'dan B'ning server'ini top → o'sha server'ga marshrutla (yoki Kafka/pub-sub orqali) → B online bo'lsa push, bo'lmasa DB + queue'ga saqla.

### 3.5 Data model

```text
messages (Cassandra — yozish-og'ir, time-series):
  PARTITION KEY: chat_id
  CLUSTERING KEY: message_id (timeuuid, vaqt tartibida sort)
  fields: sender_id, content, created_at, status

  → bir chat'ning xabarlari bitta partition'da, vaqt bo'yicha tartiblangan

user_session (Redis):  user_id → { server_id, last_seen, status }
group_members:         group_id → [user_id, ...]
```

**💡 Tushuncha:** Cassandra (yoki HBase) bu yerda mukammal — yozish-og'ir, time-series, `chat_id` bo'yicha partition, `message_id` (timeuuid) bo'yicha tartib. Ordering muammosi data model darajasida hal bo'ladi.

### 3.6 Deep dive — delivery, ordering, presence

**(a) Message delivery (online va offline):**
```text
1. A xabar yuboradi → A'ning chat server'i ACK qaytaradi (status: sent ✓)
2. Server xabarni Message DB'ga yozadi (durability)
3. Session store'dan B'ning holatini tekshiradi:
   - B online: B'ning server'iga push → B oladi (status: delivered ✓✓)
   - B offline: queue/DB'da "undelivered" qoladi
4. B online bo'lganda: undelivered xabarlarni pull qiladi, delivered qaytaradi
5. B o'qiganda: READ event → A'ga status: read (ko'k ✓✓)
```

**(b) Message ordering:**
- Suhbat ichida tartib **`message_id`** (server tomonda berilgan, monotonik — timeuuid/Snowflake) bilan kafolatlanadi.
- Mijoz `clientMsgId` (idempotency uchun) yuboradi; server `serverMsgId` (tartib uchun) qaytaradi.
- Cassandra clustering key `message_id` bo'yicha avtomatik sort — o'qishda tartib tayyor.

**⚠️ Ehtiyot bo'l:** Faqat client timestamp'ga ishonmang — soatlar farq qiladi (clock skew), tartib buziladi. **Tartibni server beradi** (xabar server'ga kelgan vaqt/sequence bo'yicha).

**(c) Idempotency / "exactly-once" his-tuyg'u:**
- Tarmoq uzilishida mijoz qayta yuboradi. `clientMsgId` (UUID) bilan server duplicate'ni aniqlaydi (bir xil ID = bir xil xabar) → bir marta saqlaydi.
- "At-least-once delivery + idempotency = exactly-once effect".

**(d) Online presence:**
```text
- WebSocket ulanganda: Redis'ga "userA online, server_id=7, TTL=30s"
- Har 10-15 sek heartbeat → TTL yangilanadi
- Heartbeat to'xtasa → TTL tugaydi → offline (last_seen yoziladi)
```
- ⚠️ Presence "ko'p yozish" muammosi: million user har 10 sekundda yozsa — Redis yuki katta. **Yechim:** holatni faqat o'zgarganda yoki suhbatdoshlarga lozim bo'lganda yangilang (lazy/subscription-based presence).

**(e) Guruh chat:**
- Kichik guruh (< 256): xabar guruh a'zolari uchun fan-out (har biriga marshrutla).
- A'zo offline bo'lsa — undelivered queue'da.

### 3.7 Bottleneck / trade-off / scaling

- **Connection management** — chat server'lar stateful (conn ushlaydi). Yangi conn'ni LB consistent hashing yoki "least connections" bilan taqsimlaydi.
- **Inter-server routing** — A va B turli server'da bo'lsa, server'lararo Kafka/Redis pub-sub yoki to'g'ridan-to'g'ri marshrutlash.
- **Server crash** — chat server o'lsa, mijozlar qayta ulanadi (boshqa server'ga), session store yangilanadi. Yetkazilmagan xabar DB/queue'da turibdi.
- **Trade-off** — WebSocket (durdona, ammo har conn resurs) vs HTTP long-polling (sodda, ammo samarasiz). Mobil uchun MQTT (yengil, batareya-do'st).

---

<a id="4-rate-limiter"></a>
## 4. Rate Limiter (distributed)

API'ni suiiste'mol va overload'dan himoya qiluvchi tizim: "1000 so'rov/soat per API key".

### 4.1 Requirements

**Functional:**
- Belgilangan chegaradan oshган so'rovni rad et (`429 Too Many Requests`).
- Kalit bo'yicha (user, API key, IP) cheklash.
- Konfiguratsiyalanadigan limit va oyna.

**Non-functional:**
- **Past latency** (har so'rovda tekshiriladi — < 5ms qo'shimcha).
- **Distributed** — bir nechta app instance bitta global limitni hisoblashi kerak.
- **Aniq** (accurate) — limitni jiddiy oshirmasin.

### 4.2 Capacity estimation

```text
Agar 1M API key, har biri ~10 so'rov/sek:
  Tekshiruv QPS ≈ 10M check/sek (har so'rovda counter o'qish/yozish)
  → markaziy store (Redis) atomar va tez bo'lishi shart
Storage: 1M key × ~100 bayt counter ≈ 100 MB — Redis'da yengil
```

### 4.3 API / integratsiya

Alohida endpoint emas — **middleware** sifatida har request'da ishlaydi:
```text
har request → allow(key) ?
   true  → davom et
   false → 429 Too Many Requests
            Headers: Retry-After: 3600
                     X-RateLimit-Remaining: 0
```

### 4.4 High-level arxitektura

```text
   Client ──▶ ┌──────────────┐  allow(key)?  ┌─────────┐
              │ App + RL      │──────────────▶│  Redis  │ (atomar counter)
              │ middleware    │◀──────────────│         │
              └──────────────┘   yes/no       └─────────┘
                     │
              ko'p instance, hammasi bitta Redis'ni so'raydi
              → global, izchil limit
```

### 4.5 Data model (Redis)

```text
Token bucket:  key → { tokens, lastRefillTs }   (HASH)
Sliding log:   key → sorted set (score = timestamp)
Fixed window:  key:windowId → counter (INCR, TTL)
```

### 4.6 Deep dive — algoritmlar va distributed atomarlik

**Token Bucket (afzal — burst ruxsati):**
Har key uchun "chelak" tokenlar bilan to'ladi (masalan 1000 token/soat = ~0.28 token/sek). Har so'rov 1 token oladi; token yo'q bo'lsa rad.
```text
allow(key):
   refill = (now - lastRefill) × rate
   tokens = min(capacity, tokens + refill)
   if tokens >= 1:
       tokens -= 1; lastRefill = now; return true
   return false
```
- ➕ **Burst** ruxsat etadi (chelak to'la bo'lsa darhol 1000 so'rov), silliq.

**Boshqa algoritmlar:**

| Algoritm | Qanday | Kamchilik |
|----------|--------|-----------|
| **Fixed window** | Oyna boshida counter 0, `INCR`, limit'da rad | **Chegara muammosi** — oyna chetida 2x burst (oxiri + boshi) |
| **Sliding window log** | Har timestamp'ni saqlash, oynadagi sonni hisoblash | Aniq, lekin xotira ko'p (har so'rov uchun yozuv) |
| **Sliding window counter** | Joriy + oldingi oyna interpolatsiyasi | Aniqlik/xotira balansi — amalda eng ko'p ishlatiladi |
| **Token bucket** | Tokenlar to'lib turadi | Burst ruxsati (afzallik ham, ehtiyot ham) |

**Distributed atomarlik (asosiy qiyinlik):**

Ko'p instance bitta Redis counter'ni o'qib-yozsa — **race condition**: ikki instance bir vaqtda "tokens=1" o'qib, ikkalasi ham ruxsat berishi mumkin (limit oshadi).

**✅ Yechim — Redis Lua skript (atomar):**
```text
-- read-modify-write bitta atomar operatsiyada
local tokens = redis.call('HGET', key, 'tokens')
-- refill hisobla, tekshir, dekrement — hammasi atomar
if tokens >= 1 then
   redis.call('HSET', key, 'tokens', tokens - 1)
   return 1
end
return 0
```
Lua skript Redis'da **atomar** ishlaydi (single-threaded) — race condition yo'q.

**💡 Tushuncha:** Distributed rate limiter'da **eng muhim so'z — atomarlik**. "Redis + Lua skript bilan read-modify-write'ni atomar qilaman" deganingizda to'g'ri javob bergan bo'lasiz. INCR + EXPIRE'ni alohida qilsangiz — race bor.

### 4.7 Bottleneck / trade-off

- **Redis SPOF** — Redis o'lsa rate limiter ishlamaydi. Replikatsiya + Sentinel/Cluster.
- **Latency** — har so'rovda Redis round-trip. **Optimizatsiya:** local in-memory cache + periodik Redis sync (biroz aniqlik yo'qoladi, latency yutadi).
- **Fail-open vs fail-closed** — Redis o'lsa: ruxsat berasizmi (fail-open, availability) yoki rad qilasizmi (fail-closed, himoya)? Odatda **fail-open** (rate limiter ishlamasligi tufayli mijozni bloklamang).
- **Trade-off** — markaziy (Redis, aniq, latency) vs lokal (har instance o'zi, tez, global aniq emas).

---

<a id="5-uber"></a>
## 5. Uber / Yetkazib berish

Yo'lovchini eng yaqin haydovchi bilan bog'lash. Asosiy: **geo-spatial indexing**, **location update**, **matching**.

### 5.1 Requirements

**Functional:**
- Haydovchilar lokatsiyasini real-time yangilash.
- Yo'lovchi so'raganda yaqin haydovchilarni topish.
- Match qilish (haydovchi ↔ yo'lovchi), narx hisoblash.

**Non-functional:**
- **Past latency** matching va lokatsiya'da.
- **Yuqori yozish QPS** (har haydovchi har 4 sekundda lokatsiya yuboradi).
- **Geo-aniqlik**.

### 5.2 Capacity estimation

```text
Faraz: 5M aktiv haydovchi, har 4 sekundda lokatsiya:
  5M / 4 = 1.25M location update/sek   ← juda katta yozish QPS!
  Peak                ≈ 3.5M/sek

Storage (joriy holat): 5M × ~100 bayt ≈ 500 MB — Redis'da
  (tarix kerak bo'lsa alohida time-series store)
```

**Xulosa:** Asosiy challenge — **massiv location-update yozish** va **tez geo-qidiruv**.

### 5.3 API design

```text
POST /api/v1/location   (haydovchi, har 4s)
  Body: { driverId, lat, lng, heading }

POST /api/v1/ride/request   (yo'lovchi)
  Body: { riderId, pickupLat, pickupLng }
  200:  { rideId, matchedDriver, eta }

GET /api/v1/drivers/nearby?lat=&lng=&radius=
```

### 5.4 High-level arxitektura

```text
   Haydovchi ──▶ ┌─────────────┐  yangilash  ┌──────────────┐
   (location)    │ Location svc│────────────▶│ Geo index    │ (Redis GEO /
                 └─────────────┘             │ QuadTree)    │  QuadTree)
                                             └──────┬───────┘
   Yo'lovchi ──▶ ┌─────────────┐  nearby?           │
   (request)     │ Matching svc│◀───────────────────┘
                 └──────┬──────┘
                        │ match → notify
                        ▼
                 ┌─────────────┐
                 │  Ride store │ (Postgres: ride, holat)
                 └─────────────┘
```

### 5.5 Data model

```text
driver_location (Redis GEO):  GEOADD drivers <lng> <lat> <driverId>
rides (Postgres):  ride_id | rider_id | driver_id | status | pickup | dropoff | fare
```

### 5.6 Deep dive — geo-spatial indexing

**Muammo:** "Bu nuqtadan 3 km radiusdagi haydovchilarni top" — `WHERE lat BETWEEN... AND lng BETWEEN...` sekin (full scan, index yordam bermaydi 2D'da).

**✅ Yechim — geo-spatial index:**

| Texnika | Qanday | Foydalanish |
|---------|--------|-------------|
| **Geohash** | Lat/lng'ni string'ga kodlash; umumiy prefix = yaqin | Redis `GEOADD/GEOSEARCH` shu asosda |
| **QuadTree** | Xaritani rekursiv 4 ga bo'lish, zich joyda chuqurroq | Dinamik zichlikka moslashadi |
| **S2 / H3** | Sferani hujayralarga bo'lish (Google S2, Uber H3) | Uber aynan H3 ishlatadi |

```text
Geohash: yaqin nuqtalar umumiy prefix beradi
  (40.71, -74.00) → "dr5ru"
  (40.72, -74.01) → "dr5rv"   ← prefix "dr5r" umumiy → yaqin
qidiruv: shu prefix + qo'shni hujayralarni skanla (kichik to'plam)
```

**Location update'ni boshqarish (1.25M/sek):**
- Har update'ni Postgres'ga yozmang — **Redis GEO** (in-memory, tez yozish).
- Faqat joriy holat kerak (tarix emas) → eski qiymat ustiga yoziladi.
- Tarix kerak bo'lsa — Kafka'ga stream, alohida pipeline.

### 5.7 Bottleneck / trade-off

- **Yozish QPS** — Redis GEO yoki sharded geo-service (region bo'yicha sharding: shahar/grid).
- **Matching** — yaqin N haydovchini top, eng yaxshisini tanlash (ETA, reyting). Ikki yo'lovchi bitta haydovchini olmasligi uchun lock/atomar assign.
- **Sharding** — geografik (region/grid bo'yicha) — har shahar mustaqil scale.

---

<a id="6-instagram-video"></a>
## 6. Instagram / Video streaming

Rasm/video yuklash, saqlash va yetkazish. Asosiy: **object storage**, **CDN**, **encoding/transcoding**.

### 6.1 Requirements

**Functional:**
- Rasm/video yuklash.
- Feed'da ko'rish (turli sifat/o'lcham).
- Video streaming (adaptive bitrate).

**Non-functional:**
- **Past latency** yetkazishda (CDN orqali).
- **Massiv storage** (petabayt'lar).
- **O'qish-og'ir** (yuklash kam, ko'rish ko'p).

### 6.2 Capacity estimation

```text
Faraz: 500M user, kuniga 100M media yuklash, har biri 2 MB:
  Yozish: 100M / 86 400 ≈ 1 160 upload/sek (peak 3x ≈ 3 500)
  Storage: 100M × 2MB = 200 TB/kun
           5 yil ≈ 360 PB (xom; replikatsiya 3x bilan ~1 EB)
  → object storage (S3) + CDN MAJBURIY, hech qachon bitta DB emas
Bandwidth (o'qish): ko'rish yuklashdan 100x → CDN hal qiladi
```

### 6.3 API design

```text
POST /api/v1/media/upload
  → 1) presigned URL ol  2) to'g'ridan-to'g'ri object storage'ga yukla
  Body: { type, size }  → { uploadUrl, mediaId }

GET /api/v1/media/{id}   → CDN URL (turli sifat varianti)
```

**💡 Tushuncha:** Katta fayllarni app server orqali emas — **presigned URL** bilan mijoz to'g'ridan-to'g'ri S3'ga yuklaydi (app server bandwidth bottleneck bo'lmaydi).

### 6.4 High-level arxitektura

```text
   Client ──upload──▶ ┌────────────┐  presigned  ┌──────────────┐
                      │ Upload svc │────────────▶│Object storage│ (S3)
                      └────────────┘             └──────┬───────┘
                                                        │ trigger
                                                        ▼
                                                 ┌──────────────┐
                                                 │ Transcoding  │ (turli
                                                 │ pipeline     │  sifat)
                                                 └──────┬───────┘
                                                        ▼
   Client ──view──▶ ┌──────┐  cache miss   ┌──────────────┐
                    │ CDN  │──────────────▶│Object storage│
                    │ edge │◀──────────────│              │
                    └──────┘   yetkazish   └──────────────┘
```

### 6.5 Data model

```text
media (metadata DB):  media_id | user_id | type | s3_key | variants[] | created_at
  variants: { "240p": url, "720p": url, "1080p": url }   (video uchun)
```

### 6.6 Deep dive — encoding va CDN

**Transcoding (video):**
- Yuklangan video bir nechta sifatga **transcode** qilinadi (240p/480p/720p/1080p) va segmentlarga bo'linadi (HLS/DASH).
- **Adaptive bitrate streaming** — mijoz tarmoq tezligiga qarab sifatni dinamik tanlaydi.
- Pipeline asinxron (queue + worker) — yuklash darhol tugaydi, transcoding fonida.

**CDN (asosiy yetkazish):**
- Media foydalanuvchiga eng yaqin **edge server**'dan beriladi (latency past).
- **Cache-aside at edge** — birinchi so'rovda origin'dan (S3), keyin edge'da cache.
- **Pull vs Push CDN** — pull (so'ralganda origin'dan torta) odatda; viral kontent uchun push (oldindan tarqat).

### 6.7 Bottleneck / trade-off

- **Storage** — petabayt scale → object storage (S3/GCS) + lifecycle (eski/ko'rilmagan media'ni arzon tier'ga, Glacier).
- **Bandwidth** — ko'rish yuki CDN'ga offlload qilinadi (origin himoyalanadi).
- **Transcoding xarajati** — CPU-intensiv; spot instance / serverless bilan arzonlashtirish.
- **Trade-off** — storage (barcha variantni saqlash, ko'p joy) vs compute (so'ralganda transcode, ko'p CPU). Odatda mashhur sifatlarni oldindan, kamyoblarini on-demand.

---

<a id="qa"></a>
## Umumiy savollar (Q&A)

### ❓ System design intervyusiga qanday yondashasiz?

**✅ Javob:** Tartibli ketma-ketlik: (1) **Requirements** — functional + non-functional, scope'ni savol bilan torting; (2) **Capacity estimation** — QPS, storage, bandwidth raqam bilan; (3) **API design**; (4) **High-level arxitektura** (diagramma); (5) **Data model**; (6) **Deep dive** — eng qiyin qismni chuqurlashtiring; (7) **Bottleneck / scaling**. Chalkash boshlamang — struktura ko'rsating.

### ❓ Fan-out on write va fan-out on read farqi nima?

**✅ Javob:** **Write (push)** — post yozilganda darhol barcha follower feed'iga yoziladi; o'qish tez, yozish qimmat. **Read (pull)** — feed so'ralganda runtime'da yig'iladi; yozish arzon, o'qish qimmat. Twitter'da **hybrid**: oddiy user'ga push, celebrity'ga pull — bu "celebrity problem"ni (million yozish) hal qiladi.

### ❓ Push (server→client) real-time uchun WebSocket, SSE, long-polling — qaysi biri?

**✅ Javob:** **WebSocket** — to'liq ikki tomonlama (chat, o'yin); doimiy conn, resurs ko'p. **SSE** (Server-Sent Events) — bir tomonlama server→client (feed, notification), HTTP ustida, sodda. **Long-polling** — fallback, samarasiz. Chat uchun WebSocket (yoki mobil'da MQTT), notification uchun SSE odatda yetarli.

### ❓ Cache strategiyalari (cache-aside, write-through, write-back) qachon?

**✅ Javob:** **Cache-aside** — o'qishda lazy load, eng keng tarqalgan (read-heavy, stale toqat). **Write-through** — yozish cache+DB ga birga (izchillik, yozish sekinroq). **Write-back** — cache'ga yoz, keyin DB'ga flush (yozish portlashi, crash'da yo'qotish xavfi). O'qish-og'ir → cache-aside; izchillik muhim → write-through.

### ❓ Hot key / hot partition muammosini qanday hal qilasiz?

**✅ Javob:** Bitta kalit (viral tweet, mashhur link) bir node'ni eziydi. Yechimlar: (1) **client/edge cache** (CDN), (2) **kalitni replikatsiya** (bir nechta node'da nusxa), (3) **kalitni sharding** (`key#1`, `key#2` — yukni tarqat), (4) **local cache** app server'da. Sharding kalitida ham hot key oldini olish uchun yaxshi hash funksiya tanlang.

### ❓ CAP teoremasi — qachon CP, qachon AP?

**✅ Javob:** Tarmoq bo'linganda (partition) **izchillik (C)** yoki **mavjudlik (A)** dan birini tanlash kerak. **CP** (consistency) — bank, inventar, to'lov (noto'g'ri ma'lumotdan ko'ra rad qilish yaxshi). **AP** (availability) — like soni, feed, DNS (eskiroq ma'lumot toqat, lekin ishlab tursin). "Hamma narsada strong consistency" — xato yondashuv.

### ❓ SQL vs NoSQL'ni qanday tanlaysiz?

**✅ Javob:** **SQL** — strukturali, JOIN, tranzaksiya, kuchli izchillik kerak (to'lov, foydalanuvchi, buyurtma). **NoSQL** — massiv scale, oddiy access pattern (key-value lookup), yuqori yozish (chat, feed, metrics, geo). Twitter timeline → Redis; chat → Cassandra; URL lookup → key-value. "Access pattern'ga qarab tanla", buzz'ga emas.

### ❓ Message queue nima uchun kerak?

**✅ Javob:** (1) **Decoupling** — producer/consumer mustaqil; (2) **Load leveling** — spike'ni buffer qiladi (1M notification queue'da turadi, worker o'z tezligida ishlaydi); (3) **Asinxron** — sekin ishni (transcoding, fan-out, email) so'rov yo'lidan chiqarish; (4) **Reliability** — retry, DLQ. Kafka (throughput, stream), RabbitMQ/SQS (task queue).

### ❓ Idempotency nima va nima uchun muhim?

**✅ Javob:** Bir xil operatsiyani ko'p marta bajarish bir martalik effekt berishi. Tarmoq retry'da to'lov ikki marta bo'lmasligi uchun **idempotency key** (UUID) — server bir xil key'ni bir marta qayta ishlaydi. "At-least-once delivery + idempotency = exactly-once effect" (chat, to'lov, webhook'da hal qiluvchi).

### ❓ Latency'ni o'rtacha emas, percentile bilan nega o'lchaysiz?

**✅ Javob:** O'rtacha (mean) tail'ni yashiradi — 99 so'rov 10ms, 1 tasi 5s bo'lsa o'rtacha "yaxshi" ko'rinadi, lekin har 100-foydalanuvchidan biri dahshatli tajriba oladi. **p95/p99/p999** "eng yomon 5%/1%/0.1%" qancha kutishini ko'rsatadi. SLO va foydalanuvchi tajribasi tail latency bilan belgilanadi.

### ❓ Single point of failure (SPOF)'ni qanday yo'qotasiz?

**✅ Javob:** Har komponentni redundant qiling: app server'lar stateless + bir nechta, DB replikatsiya (primary + replica + failover), cache cluster (Sentinel), LB ikkita (active-passive), multi-AZ deploy. "Bu komponent o'lsa nima bo'ladi?" savolini har element uchun bering — javob "tizim to'xtaydi" bo'lmasligi kerak.

### ❓ Consistent hashing nima uchun kerak?

**✅ Javob:** Oddiy `hash(key) % N`'da node qo'shilsa/o'chsa **barcha** kalit qayta taqsimlanadi (massiv cache miss / rebalance). **Consistent hashing**'da faqat `~1/N` kalit ko'chadi. Virtual node'lar bilan yuk teng taqsimlanadi. Sharded cache, DB, distributed store'da node qo'shish/olishni arzon qiladi.

---

<a id="masalalar"></a>
## Masalalar

> Yechimlar: [solutions/system-design/02-case-studies.md](../solutions/system-design/02-case-studies.md)

Quyidagi tizimlarni **7-bosqichli tuzilma** bilan loyihalashtiring: (1) Requirements, (2) Capacity estimation (raqam bilan), (3) API design, (4) High-level arxitektura (ASCII diagramma), (5) Data model, (6) Deep dive, (7) Bottleneck/trade-off/scaling.

1. **Pastebin / kod-ulashish servisi.** Foydalanuvchi matn (kod) yuklaydi, qisqa link oladi, link orqali ko'riladi. Expiration, syntax highlighting, public/private. URL shortener'dan farqini (matn saqlash, o'lcham limiti) deep dive'da ko'rsating.

2. **Notification system.** 50M foydalanuvchiga push/email/SMS yuboruvchi tizim. Channel routing, queue, worker, retry, DLQ, idempotency, throttling (har user'ga kuniga max N). Spike'da (bir vaqtda 5M notification) load leveling'ni yoriting.

3. **Typeahead / autocomplete (qidiruv taklifi).** Foydalanuvchi yozayotganda top-K mashhur taklifni real-time ko'rsatish. Trie data struktura, prefix qidiruv, top-K ranking, har keypress'da past latency (< 100ms) deep dive bo'lsin.

4. **Distributed ID generator.** Sekundiga millionlab noyob, vaqt-tartibli (sortable) 64-bit ID generatsiya qiluvchi servis (Snowflake-ga o'xshash). Clock skew, machine ID, sequence overflow muammolarini deep dive'da hal qiling.

5. **News feed (Instagram/Facebook).** Twitter case study'dagi fan-out tahlilini kengaytiring: ranked feed (xronologik emas), ML ranking signallari, "celebrity problem" hybrid yechimi, feed cache invalidation. Capacity'ni 1 mlrd user uchun hisoblang.

6. **Web crawler.** Internetni skanlaydigan crawler: URL frontier (queue), dedup (ko'rilgan URL), politeness (robots.txt, rate-per-domain), distributed worker'lar. Trillion sahifa scale'da deep dive — dedup va frontier'ni qanday sharding qilasiz.

7. **Google Drive / Dropbox (fayl sinxronizatsiya).** Fayllarni qurilmalar aro sinxronlash, versioning, conflict resolution, chunking (katta fayl), deduplication. Chunk-level sync va delta sinxronizatsiyani deep dive'da yoriting.

8. **Online ko'p o'yinchili leaderboard.** Real-time global reyting jadvali: million o'yinchi skori, top-100 + "mening o'rnim", real-time yangilanish. Redis sorted set (ZADD/ZRANK), sharding, percentile rank deep dive bo'lsin.

9. **Distributed cache (Redis-ga o'xshash).** O'z key-value distributed cache'ingizni loyihalang: consistent hashing, replikatsiya, eviction (LRU/LFU), TTL, node qo'shish/olish'da rebalance. Cache stampede himoyasini deep dive'da ko'rsating.

10. **Video konferensiya (Zoom).** Ko'p ishtirokchili real-time video/audio: WebRTC vs SFU/MCU, media server, bandwidth adaptatsiya, ekran ulashish. 100 ishtirokchili qo'ng'iroqda media routing'ni deep dive'da yoriting.

---

← [System Design bo'limiga qaytish](./README.md)
