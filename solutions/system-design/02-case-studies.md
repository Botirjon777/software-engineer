# System Design Case Study'lar — Masalalar Yechimi

Bu fayl [`system-design/02-case-studies.md`](../../system-design/02-case-studies.md) bo'limidagi "Masalalar" qismining to'liq yechimlari. Har yechim xuddi shu **7-bosqichli tuzilma** bilan: (1) Requirements, (2) Capacity estimation, (3) API design, (4) High-level arxitektura, (5) Data model, (6) Deep dive, (7) Bottleneck/trade-off/scaling.

---

## 1. Pastebin / kod-ulashish servisi

**(1) Requirements.**
- Functional: matn (kod) yuklash → qisqa link; link orqali ko'rish; expiration (TTL); public/private; ixtiyoriy syntax highlighting.
- Non-functional: o'qish-og'ir (read:write ~10:1, URL shortener'dan kamroq nisbat — paste kamroq viral); past latency o'qishda; matn 1-10 MB gacha bo'lishi mumkin.

**(2) Capacity estimation.**
```text
Faraz: 10M paste/oy, o'rtacha 100 KB:
  Yozish: 10M / 30 / 86 400 ≈ 4 paste/sek (peak 3x ≈ 12)
  O'qish (10:1): ≈ 40 read/sek
  Storage: 10M × 100KB = 1 TB/oy → 5 yil ≈ 60 TB
  → matn KATTA (URL shortener'da ~500B edi, bu yerda 100KB) → matnni
    object storage'ga, metadata'ni DB'ga ajratish kerak
```

**(3) API design.**
```text
POST /api/v1/paste   { content, expireDays?, visibility, language? } → { pasteId, url }
GET  /{pasteId}      → paste matni (private bo'lsa auth tekshir)
DELETE /api/v1/{pasteId}
```

**(4) High-level arxitektura.**
```text
   Client ──▶ ┌──────────┐  metadata  ┌──────────┐
              │ App svc  │───────────▶│ Metadata │ (pasteId→s3_key, meta)
              └────┬─────┘            │   DB     │
                   │ matn             └──────────┘
                   ▼
              ┌──────────┐   read    ┌──────────┐
              │ Object   │◀──────────│  CDN /   │
              │ storage  │           │  cache   │
              └──────────┘           └──────────┘
```

**(5) Data model.**
```text
pastes:  paste_id (PK, base62) | s3_key | visibility | language
         | expires_at | created_at | owner_id
matn: object storage (S3), key = paste_id   — DB'da matn EMAS, faqat metadata
```

**(6) Deep dive — URL shortener'dan farqi.**
- URL shortener: qiymat kichik (~200 bayt URL) → DB'da to'g'ridan-to'g'ri saqlash mumkin.
- Pastebin: qiymat katta (100KB-10MB matn) → DB'ga solsa shishadi, sekinlashadi. **Matn object storage'ga, DB'da faqat pointer (s3_key)**. O'qishda: metadata DB'dan → matn S3/CDN'dan.
- ID generatsiya: counter+base62 (URL shortener bilan bir xil).
- O'lcham limiti: yuklashda max size tekshiruvi (masalan 10MB), katta paste'ni presigned URL bilan to'g'ridan-to'g'ri S3'ga.

**(7) Bottleneck / scaling.**
- O'qish-og'ir mashhur paste → CDN edge cache.
- Expiration → TTL cleanup job (S3 lifecycle + DB delete) yoki S3 object expiration.
- Private paste → CDN'da cache qilmaslik yoki signed URL.

---

## 2. Notification system

**(1) Requirements.**
- Functional: push/email/SMS yuborish; channel routing (user preference); template; idempotency; throttling (kuniga max N).
- Non-functional: yuqori throughput; reliability (yo'qolmasin → retry+DLQ); spike'ga chidamli (5M bir vaqtda).

**(2) Capacity estimation.**
```text
50M user, har biri kuniga 5 notification = 250M/kun ÷ 86 400 ≈ 2 900/sek
Spike: bir vaqtda 5M (masalan global e'lon) → queue buffer SHART
  agar worker'lar 50k/sek qayta ishlasa: 5M / 50k = 100 sek'da tugaydi
```

**(3) API / integratsiya.**
```text
POST /api/v1/notify  { userId, type, template, data, channels? } → 202 Accepted
  (sinxron emas — queue'ga tushadi, darhol 202)
```

**(4) High-level arxitektura.**
```text
   Producer ──▶ ┌──────┐    ┌─────────┐   ┌──────────┐
   (event)      │ API  │──▶ │  Queue  │──▶│ Worker'lar│──┬─▶ Push (FCM/APNS)
                └──────┘    │ (Kafka) │   │ (consumer)│  ├─▶ Email (SES)
                           └─────────┘   └────┬──────┘  └─▶ SMS (Twilio)
                                              │ fail
                                              ▼
                                          ┌──────┐
                                          │ DLQ  │  (retry tugagach)
                                          └──────┘
```

**(5) Data model.**
```text
notifications:  id | user_id | channel | status | attempts | created_at
user_prefs:     user_id → { push:bool, email:bool, sms:bool, quietHours }
sent_dedup:     idempotency_key → ttl   (duplicate oldini olish, Redis)
```

**(6) Deep dive — load leveling + reliability.**
- **Load leveling (spike):** API darhol queue'ga push qilib `202` qaytaradi (sinxron yubormaydi). Queue (Kafka) 5M event'ni buffer qiladi; worker'lar o'z tezligida (rate-limited, masalan FCM limiti) iste'mol qiladi. Spike "tekislanadi".
- **Retry + DLQ:** yuborish muvaffaqiyatsiz (provider 5xx) → eksponensial backoff bilan retry (3 marta); baribir fail → **DLQ**'ga (keyin tekshirish/manual).
- **Idempotency:** `idempotency_key` (event_id) Redis'da; bir xil key qayta kelsa skip (duplicate notification yubormaslik).
- **Throttling:** user_prefs + kunlik counter (Redis) — limit oshsa skip yoki batch.

**(7) Bottleneck / scaling.**
- Provider rate limit (FCM/SES) → worker'da rate limiter; queue buffer qiladi.
- Worker'larni horizontal scale (Kafka partition = parallellik).
- Quiet hours / preference → yuborishdan oldin tekshir.

---

## 3. Typeahead / autocomplete

**(1) Requirements.**
- Functional: prefix bo'yicha top-K (5-10) mashhur taklif; real-time har keypress.
- Non-functional: **juda past latency** (< 100ms, ideal < 50ms); read-og'ir (yozish — taklif bazasini yangilash — kam).

**(2) Capacity estimation.**
```text
100M qidiruv/kun, har biri ~5 keypress = 500M typeahead so'rov/kun
  ÷ 86 400 ≈ 5 800/sek (peak 3x ≈ 17 400)
  → har keypress so'rov → cache/in-memory MAJBURIY
Storage: top mashhur so'rovlar (trie) ~10M prefix → bir necha GB, RAM'ga sig'adi
```

**(3) API design.**
```text
GET /api/v1/suggest?q=<prefix>&limit=10 → { suggestions: ["...", ...] }
  (debounce client'da — har harfda emas, ~50ms kechikish bilan)
```

**(4) High-level arxitektura.**
```text
   Client ──prefix──▶ ┌──────────┐  miss  ┌──────────────┐
   (debounced)        │ Suggest  │───────▶│ Trie service │ (in-memory,
                      │ svc+cache│◀───────│ + top-K       │  precomputed)
                      └──────────┘        └──────────────┘
                                                ▲
                                          ┌─────┴──────┐
                                          │ Offline    │  (qidiruv loglaridan
                                          │ aggregation│   trie qayta quriladi)
                                          └────────────┘
```

**(5) Data model.**
```text
Trie node: char → children, isWord, topK[]   (har node'da oldindan
   hisoblangan top-K — qidiruv runtime'da arzon)
frequency: query → count   (offline agregatsiya, trie'ni yangilash uchun)
```

**(6) Deep dive — Trie + top-K + latency.**
- **Trie (prefix tree):** har prefix uchun bola tugunlar. `q="ka"` → "ka" tuguniga tushib, ostidagi top-K so'zlarni qaytarish.
- **Precomputed top-K:** har trie tugunida **oldindan hisoblangan** top-K saqlanadi → runtime'da pastki daraxtni skanlamaysiz (O(1) o'qish). Bu past latency uchun hal qiluvchi.
- **Yangilash:** trie real-time emas — qidiruv loglari **offline agregatsiya** (har soat/kun) bilan frequency hisoblanib, trie qayta quriladi. Stale toqat (1 soat eski taklif normal).
- **Latency:** trie RAM'da, top-K tayyor → < 10ms. Client **debounce** (har keypress emas, 50ms pauzada).
- **Sharding:** trie katta bo'lsa birinchi harf bo'yicha sharding (a-* bir server, b-* boshqa).

**(7) Bottleneck / scaling.**
- Read QPS → in-memory replica'lar (read-only, ko'p nusxa).
- Trie xotira → sharding (prefix bo'yicha) yoki compressed trie.
- Personalizatsiya kerak bo'lsa — global trie + per-user re-rank.

---

## 4. Distributed ID generator

**(1) Requirements.**
- Functional: noyob 64-bit ID; vaqt-tartibli (sortable); sekundiga millionlab.
- Non-functional: yuqori availability; past latency (< 1ms); markazsiz (SPOF yo'q); ID'lar taxmin qilib bo'lmaydigan bo'lishi shart emas (Snowflake taxmin qilinadi).

**(2) Capacity estimation.**
```text
Talab: 1M+ ID/sek global. Snowflake: 1 machine 4096 ID/ms = 4M ID/sek
  → bir necha machine yetarli; har machine mustaqil (koordinatsiyasiz)
```

**(3) API.**
```text
nextId() → 64-bit int   (kutubxona sifatida har servisda lokal, yoki
                          ID-service sifatida gRPC)
```

**(4) High-level arxitektura.**
```text
   Service A ─┐
   Service B ─┼─▶ har birida lokal Snowflake generator (machine_id bilan)
   Service C ─┘     koordinatsiya YO'Q (timestamp + machine_id noyoblikni beradi)
   machine_id → ZooKeeper/etcd dan bir marta olinadi (deploy'da)
```

**(5) Data model (64-bit layout).**
```text
 1 bit  | 41 bit timestamp | 10 bit machine_id | 12 bit sequence
 (sign) | (ms, ~69 yil)    | (1024 machine)    | (4096/ms)
```

**(6) Deep dive — clock skew, machine_id, overflow.**
- **Vaqt-tartiblilik:** timestamp eng yuqori bitlarda → ID'lar vaqt bo'yicha sort qilinadi (DB index, cursor pagination uchun ideal).
- **Machine ID:** har instance noyob 10-bit ID oladi (ZooKeeper/etcd'dan deploy vaqtida) → ikki machine bir xil ID bermaydi.
- **Sequence:** bir machine bir ms ichida 4096 ID (12 bit). Tugasa — keyingi ms'ni kut.
- **Clock skew (eng qiyin):** soat orqaga ketsa (NTP sync) — takror ID xavfi. Yechim: oxirgi timestamp'ni eslab qol; agar `now < lastTs` (soat orqaga ketdi) → kichik vaqt kuting yoki xato qaytaring (ID bermang). Hech qachon orqaga ketgan timestamp bilan ID bermang.
- **Overflow:** 41-bit timestamp ~69 yil (epoch'dan); sequence overflow → keyingi ms.

**(7) Bottleneck / scaling.**
- Markazsiz → cheksiz scale (machine qo'shilsa, yangi machine_id).
- Machine_id cheklovi (1024) → kerak bo'lsa bit taqsimotini o'zgartirish.
- Alternativ: DB ticket server (range allocation) yoki UUID (sortable emas).

---

## 5. News feed (Instagram/Facebook)

**(1) Requirements.**
- Functional: post; **ranked** feed (xronologik emas, ML-ranked); follow; like/comment.
- Non-functional: o'qish-og'ir (~100:1); past latency feed'da; eventual consistency maqbul.

**(2) Capacity estimation.**
```text
1 mlrd user, 600M DAU, har biri kuniga 30 feed-ochish:
  O'qish: 600M × 30 = 18 mlrd/kun ÷ 86 400 ≈ 208 000 feed/sek (peak ≈ 625k)
  Yozish (post): 600M × 1 ≈ 7 000 post/sek
  → o'qish hukmron → precompute (push) + cache MAJBURIY
```

**(3) API.**
```text
POST /api/v1/post  { text, mediaIds }
GET  /api/v1/feed?cursor=&limit=20  → ranked feed (cursor pagination)
```

**(4) High-level arxitektura.**
```text
   post ──▶ ┌──────────┐   ┌──────────┐   ┌──────────────┐
            │ Write API│──▶│ Fan-out  │──▶│ Feed cache   │ (user→[postId])
            └──────────┘   │ (Kafka)  │   └──────┬───────┘
                          └──────────┘          │
   feed ──▶ ┌──────────┐                         ▼
            │ Read API │◀──── ranking ──── ┌──────────────┐
            │          │     (ML service)  │ Ranking svc  │
            └──────────┘                   └──────────────┘
```

**(5) Data model.**
```text
posts:        post_id (snowflake) | user_id | content | created_at | engagement
follows:      follower_id | followee_id   (index: followee_id)
feed_cache:   user_id → [post_id, ...]   (Redis, precomputed candidate'lar)
```

**(6) Deep dive — ranked feed + celebrity + invalidation.**
- **Fan-out (Twitter case'dan kengaytma):** oddiy user'ga **push** (feed_cache'ga post_id yoziladi); celebrity (> threshold follower) **pull** (runtime merge) → **hybrid**, "celebrity problem"ni hal qiladi.
- **Ranked (xronologik emas):** feed_cache'da **candidate'lar** (oxirgi N post). Read'da **ranking service** ML signallari bilan tartiblaydi: engagement (like/comment ehtimoli), recency, munosabat yaqinligi, kontent turi. Top-20 qaytadi.
- **Ranking signallari:** user-post affinity, post yoshi, global popularity, "siz ko'rmagan" boost.
- **Cache invalidation:** post o'chirilsa/yangilanса feed_cache'dan olib tashlash; engagement o'zgarsa re-rank (lekin candidate cache shu, faqat tartib o'zgaradi).

**(7) Bottleneck / scaling.**
- Ranking service CPU-og'ir → candidate'larni cache, ranking'ni feature store bilan tezlashtirish.
- Inactive user (oylar kirmagan) → push qilmaslik, login'da generate.
- Feed cache memory → faqat oxirgi ~500 candidate/user.

---

## 6. Web crawler

**(1) Requirements.**
- Functional: URL frontier'dan sahifa yuklab, link'larni ajratib, qayta queue'ga; matn saqlash/indekslash.
- Non-functional: massiv scale (trillion sahifa); politeness (server'larni ezmaslik); dedup (bir sahifani qayta crawl qilmaslik); freshness.

**(2) Capacity estimation.**
```text
1 mlrd sahifa/kun ÷ 86 400 ≈ 11 600 sahifa/sek
  Har sahifa ~100 KB → 1e9 × 100KB = 100 TB/kun (xom)
  → distributed worker'lar + object storage MAJBURIY
URL frontier: trillion URL → distributed queue, sharded
```

**(3) API / komponent (servis emas, pipeline).**
```text
Frontier.next() → URL
Fetcher.fetch(URL) → html
Parser.extract(html) → [links], content
Dedup.seen(URL) → bool
```

**(4) High-level arxitektura.**
```text
   ┌──────────┐   URL    ┌─────────┐  html  ┌─────────┐  links
   │ URL      │─────────▶│ Fetcher │───────▶│ Parser  │───┐
   │ Frontier │          │ workers │        │         │   │
   └────▲─────┘          └─────────┘        └────┬────┘   │
        │                                        │ content│
        │  yangi URL (dedup'dan o'tgan)          ▼        │
        │                                  ┌─────────┐    │
        └──────────────────────────────────│ Dedup   │◀───┘
                                           │ (seen?) │
                                           └─────────┘
                                  content → Object storage + Index
```

**(5) Data model.**
```text
frontier:  priority queue (sharded by domain) — politeness uchun domain-aware
seen_urls: Bloom filter + DB (trillion URL → Bloom xotirani tejaydi)
pages:     url_hash | content_s3_key | last_crawled | checksum
```

**(6) Deep dive — dedup va frontier sharding.**
- **Dedup (ko'rilgan URL):** trillion URL'ni to'liq saqlash qimmat → **Bloom filter** (xotira-samarali, false-positive bor lekin false-negative yo'q — "ko'rilmagan" deyilsa aniq ko'rilmagan). Bloom "ehtimol ko'rilgan" desa → DB tekshiruv. URL normalizatsiya (trailing slash, query tartibi) muhim.
- **Frontier sharding:** URL'ni **domain bo'yicha** sharding → bir domain bitta worker'da → **politeness** (per-domain rate limit oson, robots.txt, crawl-delay). Domain ichida priority (PageRank-ga o'xshash, freshness).
- **Politeness:** robots.txt cache; per-domain rate (masalan 1 req/sek); distributed bo'lsa ham domain bitta shard'da.
- **Trap'lar:** cheksiz kalendar/sessiya URL → depth limit, URL pattern filtr.

**(7) Bottleneck / scaling.**
- Fetcher I/O-bound → asinxron, ko'p worker, horizontal scale.
- Dedup Bloom filter sharded (URL hash bo'yicha).
- Freshness — muhim sahifalar tez-tez re-crawl (priority), kamuhim kam.

---

## 7. Google Drive / Dropbox

**(1) Requirements.**
- Functional: fayl upload/download; qurilmalar aro sync; versioning; conflict resolution; sharing.
- Non-functional: durability (fayl yo'qolmasin); sync past latency; bandwidth tejash (delta sync); katta fayl (GB).

**(2) Capacity estimation.**
```text
100M user, har biri 10 GB → 1 EB jami storage
  Yuklash: 100M × 5 fayl/kun × 1MB ≈ 500M/kun ÷ 86 400 ≈ 5 800/sek
  → object storage + chunk dedup (takroriy chunk'ni bir marta saqlash)
```

**(3) API.**
```text
POST /api/v1/file/upload  (chunk'lar bilan: initiate → upload chunks → complete)
GET  /api/v1/file/{id}
GET  /api/v1/changes?since=<cursor>   (sync: o'zgargan fayllar)
```

**(4) High-level arxitektura.**
```text
   Client ──chunks──▶ ┌──────────┐   ┌──────────────┐
   (watcher)          │ Block svc│──▶│ Object storage│ (chunk'lar)
                      └────┬─────┘   └──────────────┘
                           │ metadata
                           ▼
                      ┌──────────┐   notify   ┌──────────┐
                      │ Metadata │───────────▶│ Sync svc │──▶ boshqa qurilmalar
                      │   DB     │            │ (push)   │
                      └──────────┘            └──────────┘
```

**(5) Data model.**
```text
files:     file_id | user_id | name | version | [chunk_hash, ...]
chunks:    chunk_hash (PK) | s3_key | ref_count   (dedup: bir xil hash = bir nusxa)
versions:  file_id | version | chunk_list | created_at
```

**(6) Deep dive — chunking + delta sync + conflict.**
- **Chunking:** katta fayl ~4MB block'larga bo'linadi. Har block **content hash** (SHA) → chunk_hash. Faqat **o'zgargan chunk'lar** yuklanadi/yuklab olinadi (delta sync) → bandwidth tejaladi.
- **Deduplication:** bir xil chunk_hash → bir marta saqlash (ref_count). Ikki user bir xil faylni yuklasa — bir nusxa.
- **Delta sync:** fayl o'zgarganda butun faylni emas, faqat farq (o'zgargan chunk) sync. Client local chunk hash'larini server bilan solishtirib, faqat yangini yuboradi.
- **Conflict resolution:** ikki qurilma offline o'zgartirса — versioning (ikkalasi saqlanadi, "conflicted copy") yoki last-write-wins (timestamp). Drive odatda "conflicted copy" yaratadi (ma'lumot yo'qotmaslik).
- **Sync notification:** o'zgarish → sync service boshqa qurilmalarga push (long-poll/WebSocket); ular `changes?since` bilan pull.

**(7) Bottleneck / scaling.**
- Metadata DB sharding (user_id bo'yicha).
- Chunk storage → object storage (cheksiz scale).
- Sync push → connection management (chat'ga o'xshash).

---

## 8. Online ko'p o'yinchili leaderboard

**(1) Requirements.**
- Functional: skor yangilash; top-K (top-100); "mening o'rnim" (rank); real-time.
- Non-functional: past latency o'qish/yozish; yuqori yozish (o'yin davomida tez-tez skor); million o'yinchi.

**(2) Capacity estimation.**
```text
10M aktiv o'yinchi, har biri o'yin oxirida skor + jonli yangilanish:
  Yozish: ~50k skor-yangilash/sek (peak)
  O'qish (leaderboard ko'rish): ~200k/sek
  Storage: 10M × ~50 bayt ≈ 500 MB → Redis sorted set'ga sig'adi
```

**(3) API.**
```text
POST /api/v1/score   { userId, score }        → yangilash
GET  /api/v1/leaderboard?top=100              → top-K
GET  /api/v1/rank/{userId}                    → mening o'rnim
```

**(4) High-level arxitektura.**
```text
   Client ──score──▶ ┌──────────┐  ZADD  ┌──────────────┐
                     │ Score svc│───────▶│ Redis Sorted │ (leaderboard)
                     └──────────┘        │   Set        │
   Client ──view──▶  ┌──────────┐ ZREVRANGE └──────┬─────┘
                     │ Read svc │◀──────────────────┘
                     └──────────┘        durable copy → Postgres (async)
```

**(5) Data model.**
```text
Redis Sorted Set:  key="leaderboard:global"
  ZADD leaderboard <score> <userId>     -- skor = sort kaliti
  ZREVRANGE 0 99 WITHSCORES             -- top-100
  ZREVRANK <userId>                     -- mening o'rnim (0-indexed)
Postgres: durable backup (Redis volatile)
```

**(6) Deep dive — sorted set + sharding + percentile.**
- **Redis Sorted Set** ideal: `ZADD` O(log N) yozish, `ZREVRANGE` top-K, `ZREVRANK` user rank — barchasi tez. Bu data struktura leaderboard'ni "yaratilgan" kabi.
- **"Mening o'rnim":** `ZREVRANK` — to'g'ridan-to'g'ri rank qaytaradi (boshqa store'da bu butun jadvalni sortlash kerak bo'lardi).
- **Sharding (million+):** bitta Redis sig'masa — geografik/region bo'yicha shard (har region o'z leaderboard'i) yoki score-range bo'yicha. Global top uchun har shard top-K'sini merge.
- **Percentile rank:** `ZREVRANK / ZCARD` → "siz top 5%'dasiz". `ZCOUNT` bilan oraliq.
- **Durability:** Redis volatile → skor Postgres'ga ham async yoziladi; Redis o'lsa undan qayta tiklash.

**(7) Bottleneck / scaling.**
- Hot leaderboard read → Redis replica'lar (read scale).
- Yozish portlashi → batch/throttle yangilanish (har skor emas, periodik).
- Time-window leaderboard (kunlik/haftalik) → alohida sorted set + TTL.

---

## 9. Distributed cache (Redis-ga o'xshash)

**(1) Requirements.**
- Functional: get/set/delete; TTL; eviction (xotira to'lganda); distributed (ko'p node).
- Non-functional: past latency (< 1ms); yuqori availability; node qo'shish/olish'da minimal buzilish.

**(2) Capacity estimation.**
```text
1 TB cache, 100 GB/node → 10+ node
  1M ops/sek → har node ~100k ops/sek (in-memory uchun normal)
```

**(3) API.**
```text
GET key, SET key value [TTL], DEL key
```

**(4) High-level arxitektura.**
```text
   Client ──▶ ┌─────────────┐  consistent hash(key) → node
              │ Cache client│──┬──▶ Node 1 (+ replica)
              │ (routing)   │  ├──▶ Node 2 (+ replica)
              └─────────────┘  └──▶ Node 3 (+ replica)
              ring: virtual node'lar bilan teng taqsimot
```

**(5) Data model.**
```text
Har node: in-memory hash map (key→value) + LRU/LFU metadata + TTL heap
Ring: consistent hash ring (virtual node'lar)
```

**(6) Deep dive — consistent hashing + eviction + stampede.**
- **Consistent hashing:** `hash(key) % N` o'rniga ring. Node qo'shilsa/o'chsa faqat `~1/N` kalit ko'chadi (massiv rebalance yo'q). **Virtual node'lar** (har fizik node ringda ko'p nuqta) → yuk teng taqsimlanadi, "hot node" oldini oladi.
- **Replikatsiya:** har kalit keyingi M node'da nusxa (node o'lsa replica'dan o'qish). Async replikatsiya (latency) vs sync (izchillik) trade-off.
- **Eviction:** xotira to'lganda **LRU** (eng kam yaqinda ishlatilgan) yoki **LFU** (eng kam chastotali) chiqarish. TTL'li kalitlar — passive (o'qishda) + active (periodik) expiration.
- **Cache stampede:** mashhur kalit expire bo'lганда minglab so'rov bir vaqtda DB'ga uradi. **Single-flight** (faqat bitta so'rov DB'ga, qolganlari kutadi) + **jittered TTL** (expire vaqtlarini tarqatish) + **probabilistic early refresh**.

**(7) Bottleneck / scaling.**
- Node qo'shish → ring'ga qo'shilib, `1/N` kalit migratsiya.
- Hot key → kalitni replikatsiya yoki client-side cache.
- Split-brain (node ajralsa) → gossip protokol bilan membership.

---

## 10. Video konferensiya (Zoom)

**(1) Requirements.**
- Functional: ko'p ishtirokchili real-time audio/video; ekran ulashish; mute/kamera.
- Non-functional: juda past latency (< 150ms, aks holda suhbat buziladi); bandwidth adaptatsiya; 100+ ishtirokchi.

**(2) Capacity estimation.**
```text
100 ishtirokchili qo'ng'iroq, har biri 2 Mbps yuklaydi:
  Mesh (har kim har kimga): har user 99 oqim yuborar/olar → portlash! (mumkin emas)
  SFU bilan: har user 1 yuboradi (2 Mbps), 99 oladi → server relay qiladi
Server bandwidth: 100 user × 99 oqim × 2 Mbps = katta → media server scale
```

**(3) API / signaling.**
```text
WebSocket signaling: offer/answer (SDP), ICE candidate almashish
Media: WebRTC (UDP/SRTP), server orqali (SFU)
```

**(4) High-level arxitektura.**
```text
   ┌────────┐  signaling  ┌──────────────┐
   │ Client │◀═══════════▶│ Signaling svc│ (SDP, ICE almashish)
   └───┬────┘             └──────────────┘
       │ media (WebRTC)
       ▼
   ┌──────────────┐   forward   ┌────────┐
   │ SFU (media   │────────────▶│ boshqa │  (har user 1 yuboradi,
   │ server)      │             │ client │   server N ga relay qiladi)
   └──────────────┘             └────────┘
```

**(5) Data model.**
```text
rooms:    room_id | participants[] | settings
sessions: user_id → { sfu_node, streams, quality }   (transient, Redis)
```

**(6) Deep dive — Mesh vs SFU vs MCU, media routing.**
- **Mesh (P2P):** har ishtirokchi har biri bilan to'g'ridan-to'g'ri. 2-3 kishi uchun yaxshi; N kishi → har user N-1 oqim (yuklash portlashi). 100 user uchun mumkin emas.
- **SFU (Selective Forwarding Unit) — afzal:** har user **bitta** oqim server'ga yuboradi; server uni boshqalarga **forward** qiladi (transcode qilmaydi, faqat relay). Client bir necha oqim oladi. Server CPU yengil (relay), bandwidth og'ir. Zoom/Meet shu modelni ishlatadi.
- **MCU (Multipoint Control Unit):** server barcha oqimni **bitta**ga mix qiladi (transcode). Client 1 oqim oladi (yengil client), lekin server CPU juda og'ir (transcoding). Eski model.
- **Bandwidth adaptatsiya:** **simulcast** — har user bir necha sifatda yuboradi (low/med/high); SFU har qabul qiluvchiga uning tarmog'iga mos sifatni forward qiladi. Zaif tarmoq → past sifat.
- **100 ishtirokchi routing:** SFU faqat **faol so'zlovchi** + bir nechta videoni forward (hammasini emas) → bandwidth tejaladi (active speaker detection).

**(7) Bottleneck / scaling.**
- SFU server bandwidth → bir room bir SFU; katta room'lar uchun SFU cascade (server'lar aro).
- Geografik — eng yaqin SFU (latency).
- Trade-off: SFU (server bandwidth og'ir, client CPU yengil) vs MCU (server CPU og'ir, client yengil). SFU odatda g'olib (server'da bandwidth arzon, CPU qimmat).

---

← [System Design bo'limiga qaytish](../../system-design/README.md)
