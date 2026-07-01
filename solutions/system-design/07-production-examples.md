# Production Tizim Misollari — Masalalar Yechimi

Bu fayl [`system-design/07-production-examples.md`](../../system-design/07-production-examples.md) bo'limidagi "Masalalar" qismining to'liq yechimlari. Har yechim **real production nuqtai nazaridan**: masshtab taxmini, high-level arxitektura, asosiy texnik qarorlar (nega aynan shu), pattern/texnologiya, va o'rganiladigan dars.

**⚠️ Ehtiyot bo'l:** Raqamlar taxminiy va o'quv maqsadida. Intervyuda muhimi — **raqamning aniqligi emas**, balki fikrlash jarayoni: taxminni aytib, undan xulosaga borish.

---

## 1. O'z "Netflix"ingizni loyihalash (video streaming)

**Masshtab taxmini.**
```text
Faraz: 100M obunachi, peak 20M bir vaqtda ko'radi, o'rtacha 5 Mbps oqim:
  Video bandwidth: 20M × 5 Mbps = 100 Tbps (!) → BU CDN'siz IMKONSIZ
  → shuning uchun video HECH QACHON markaziy backend'dan o'tmasligi shart
  Control-plane (login, katalog, "play"): peak ~ 500K RPS — oddiy microservice
  Encoding: har video → ~20 variant (bitrate × codec × qurilma)
```

**High-level arxitektura.**
```text
   Foydalanuvchi
      │ (1) "play" bosADI → control-plane
      ▼
  ┌─────────────┐   ┌──────────────────────────────────┐
  │ API Gateway │──▶│ Playback svc: DRM, manifest, qaysi │
  │  (Zuul-vari)│   │ CDN appliance'dan olishni belgilash │
  └─────────────┘   └──────────────┬───────────────────┘
      ↕ circuit breaker             │ (2) manifest (URL'lar) qaytadi
  microservices (katalog,          ▼
   auth, recommendation)   ┌──────────────────────────┐
                           │ (3) VIDEO baytlari to'g'ri │
                           │  CDN appliance'dan oqadi   │  ◀── data-plane
                           │  (ISP ichida, eng yaqin)   │
                           └──────────────────────────┘

  ENCODING (asinxron, offline):
   yuklangan master → queue → transcoding worker'lar →
   har variant → object storage → CDN'ga tarqatilADI (pre-position)
```

**Asosiy texnik qarorlar.**
- **Control/data-plane ajratish:** control-plane faqat "qaysi videoni, qaysi CDN'dan, qaysi sifatda" ni hal qiladi (kichik JSON). Haqiqiy video to'g'ridan-to'g'ri CDN'dan. Sabab: 100 Tbps'ni backend orqali o'tkazib bo'lmaydi.
- **Encoding asinxron:** yuklash yo'lidan chiqarilADI — transcoding daqiqalar/soatlar oladi, foydalanuvchi kutmasin. Queue + worker pool. Per-title encoding (har video optimal bitrate egri).
- **Resilience:** har microservice'da circuit breaker + fallback (recommendation o'lsa → default katalog ko'rsatiladi, "play" ishlashda davom etadi).
- **CDN pre-position:** mashhur kontent peak'dan **oldin** CDN'larga tarqatiladi (yangi mavsum chiqishidan oldin).

**Chaos testing.** Production'da, ish soatida, kichik blast radius bilan: instance o'ldirish (Chaos Monkey), butun AZ "o'chirish" mashqi (Chaos Kong) — failover ishlashini isbotlash.

**O'rganiladigan dars.** Katta data'ni (video) control mantiqidan ajrating — bu CDN'ning butun mavjudlik sababi. Resilience "qo'shimcha" emas, **dizayn asosi**: har downstream o'lishini kuting va fallback bering.

---

## 2. Real-time ride-matching (Uber-vari)

**Masshtab taxmini.**
```text
Faraz: 1M faol haydovchi, har biri 4s'da location update:
  Location ingest: 1M / 4s = 250K update/sek — yozish-og'ir stream (Kafka)
  Trip so'rov: peak ~ 50K/sek
  Matching latency talabi: < 1-2s ("eng yaqin haydovchi")
```

**High-level arxitektura.**
```text
  Haydovchi app ──location(4s)──▶ ┌────────────────┐
                                  │ Location ingest │──▶ Kafka
                                  └────────┬───────┘
                                           ▼
                               ┌─────────────────────────┐
   Yo'lovchi ──"trip"──▶ ┌──────┤ Geo-index (H3 hexagon →  │
                         │ Match│  shu hujayradagi bo'sh   │
                         │ svc  │  haydovchilar)           │
                         └───┬──┘ └────────────────────────┘
                             │ trip.requested → Kafka
        ┌────────────────────┼───────────────────┐
        ▼                    ▼                   ▼
    ┌─────────┐        ┌──────────┐        ┌───────────┐
    │ Pricing │        │ Trip     │        │ ETA/route │
    │ (surge) │        │ lifecycle│        │           │
    └─────────┘        └──────────┘        └───────────┘
```

**Asosiy texnik qarorlar.**
- **Geo-index (H3):** haydovchi joylashuvi → hexagon hujayra ID. Matching = yo'lovchi hujayrasi + k-ring qo'shni hujayralardagi bo'sh haydovchilar. Nega hexagon: teng masofali qo'shnilar. Hierarchik rezolyutsiya: kam haydovchi bo'lsa k'ni (radiusni) kengaytirish.
- **Event-driven:** trip lifecycle (requested → matched → started → completed) event'lar Kafka'da. Pricing, payment, analytics, fraud — hammasi mustaqil eshitADI. Decoupling.
- **Idempotency:** trip event'lar idempotency key bilan (retry ikki marta trip yaratmasin).

**Surge pricing (per-hudud).** Har hexagon (yoki hexagon guruhi) uchun real-time `talab / taklif` nisbati hisoblanadi. Nisbat oshsa → multiplier oshadi. Bu yo'lovchi talabini kamaytiradi va qo'shni hududdan haydovchini jalb qiladi.

**Hot hudud (aeroport, konsert).** Muammo: bitta hexagon'da ulkan talab. Yechim: (a) surge bilan talab/taklifni tabiiy muvozanatlash; (b) navbat (virtual queue) — haydovchilar FIFO tartibda; (c) shu hexagon'ni alohida (dedicated) resurs bilan ishlash (hot-partition mantiqi).

**O'rganiladigan dars.** Muammoning **shakli** (geo-yaqinlik) ma'lumot strukturasini belgilaydi — umumiy DB so'rovi yetmaydi, maxsus geo-index kerak. Ba'zi yuk muammolarini narx (surge) — iqtisodiy signal — bilan hal qilinadi, faqat server bilan emas.

---

## 3. Messaging tizimi minimal server bilan (WhatsApp-vari)

**Masshtab taxmini.**
```text
Faraz: 500M foydalanuvchi, ~100M bir vaqtda ulangan:
  Concurrent ulanish: 100M → agar 1 server ~1-2M ulanish ko'tarsa → ~50-100 server
  Xabar: 50B/kun → 50e9 / 86400 ≈ 600K xabar/sek (peak 3x ≈ 2M/sek)
  → server-side storage MINIMAL (store-and-forward) → kam infra
```

**High-level arxitektura.**
```text
  Tel A ──doimiy TCP──▶ ┌──────────────────────────────┐ ◀──doimiy TCP── Tel B
                        │  Chat server (Erlang/BEAM)    │
                        │  har ulanish = yengil process │
                        │  A → routing: B onlaynmi?      │
                        │    ha  → darrov push (B proc)  │
                        │    yo'q → offline store'ga qo'y │
                        └──────────────┬───────────────┘
                                       ▼
                          ┌──────────────────────────┐
                          │ Offline message store     │
                          │ (yetkazilmagan; yetkazil-  │
                          │  gach O'CHADI)             │
                          └──────────────────────────┘
```

**Asosiy texnik qarorlar.**
- **Store-and-forward:** xabar server'da faqat yetkazilgunicha. Yetkazildi → o'chadi. Tarix telefonda. Sabab: server storage'ni radikal minimallashtirish (kam server).
- **Erlang/BEAM:** millionlab yengil process (ulanish = process), izolyatsiya, past latency. C10M muammosiga tabiiy mos.
- **Doimiy TCP:** push uchun (poll emas) — server istalgan payt xabar yubora oladi.
- **Delivery ack:** ✓ (server oldi), ✓✓ (qurilma oldi). At-least-once + client dedup (message ID) = exactly-once effect. Per-conversation ordering (bitta suhbat tartibi).
- **E2E encryption (Signal):** server mazmunni ko'rmaydi.

**Presence fan-out'ni cheklash.** Naive: har holat o'zgarishini **hammaga** broadcast → million ulanishда portlash. Yechim: presence'ni faqat **aktiv suhbatdagi** tomonga yuborish (hozir chat oynasini ochgan), hammaga emas; throttling (holatni tez-tez emas, kamdan-kam yangilash).

**O'rganiladigan dars.** Eng arzon storage — saqlamaganingiz. Scope'ni radikal cheklab (tarixni serverda saqlamaslik), murakkab distributed DB'dan qutulasiz. To'g'ri runtime (Erlang messaging'ga aynan mos) minglab emas, yuzlab server bilan ishlash beradi.

---

## 4. Home timeline (Twitter-vari) hybrid fan-out

**Masshtab taxmini.**
```text
Faraz: 300M faol foydalanuvchi, read:write ~ 100:1:
  Tvit: ~6K/sek. Timeline o'qish: ~600K/sek → O'QISH-OG'IR
  → o'qishni tez qilish uchun pre-compute (fan-out on write) mantiqiy
  Celebrity: 100M follower → bitta tvit = 100M insert → PORTLASH
```

**High-level arxitektura.**
```text
  ODDIY user tvit yozADI:
     tvit → fan-out worker → har follower timeline cache'iga tweet_id INSERT

  Timeline o'qish:
     ┌──────────────────────────────────────────────┐
     │ 1. cache'dan pre-computed tweet_id ro'yxati    │
     │    (oddiy odamlar tvitlari)                    │
     │ 2. + men obuna CELEBRITY'larni PULL qil        │
     │ 3. MERGE + ML ranking → tweet_id ro'yxati      │
     │ 4. HYDRATE (id → to'liq tvit, alohida store)   │
     └──────────────────────────────────────────────┘
                    ▲
   Redis (timeline cache: user_id → [tweet_id...])
```

**Asosiy texnik qarorlar.**
- **Hybrid fan-out:** oddiy user → write (push follower cache'iga); celebrity → read (push qilmaydi). O'qishda ikkalasi merge qilinadi. Bu 100M insert portlashini ham, sekin o'qishni ham chetlab o'tadi.
- **Timeline = ID ro'yxati:** cache'da faqat tweet_id (kichik), to'liq tvit alohida store'da. O'qishda **hydrate** (id → mazmun). Sabab: tvit mazmuni takrorlanmasin (bitta tvit million timeline'da faqat id sifatida).
- **ML ranking:** xronologik emas — engagement bo'yicha saralash.

**"Celebrity" chegarasi.** Statik son emas, dinamik: follower soni chegaradan (masalan, 100K-1M) oshsa yoki fan-out xarajati juda katta bo'lsa → read rejimiga o'tkaziladi. Ba'zan hybrid per-tweet ham qilinadi.

**Merge + rank past latency.** Pre-computed qism tayyor (cache); faqat celebrity qismi + ranking real-time. Celebrity soni kam (per-user o'nlab), shuning uchun pull arzon. Ranking cache'lanadi/asinxron yangilanadi.

**O'rganiladigan dars.** Bitta strategiya hammaga ishlamaydi — foydalanuvchini **segmentlab** (oddiy vs celebrity), har segmentga mos yechim bering. "90% holat"ni optimallashtiring, "og'ir 10%"ni alohida ishlang.

---

## 5. Foto/kontent platforma kichik jamoa bilan (Instagram-vari)

**Masshtab taxmini.**
```text
Faraz: 100M foydalanuvchi, 50M rasm/kun:
  Rasm yozish: 50e6 / 86400 ≈ 600/sek (peak 3x ≈ 1800)
  Storage: 50M/kun × 500KB ≈ 25 TB/kun → object storage'ga (DB'ga EMAS)
  Metadata (kim, qachon, caption): kichik → sharded Postgres
```

**High-level arxitektura.**
```text
  Foydalanuvchi ──rasm──▶ ┌───────────┐──▶ ┌──────────────────┐
                          │ App server │    │ Object storage +  │
                          │ (Django)   │    │ CDN (rasm baytlari)│
                          └─────┬─────┘    └──────────────────┘
                                │ metadata
                                ▼
                ┌──────────────────────────────────┐
                │ PostgreSQL (LOGICAL sharding)      │
                │  logical shard → fizik server      │
                │  [Shard1][Shard2]...[ShardN]       │
                └──────────────────────────────────┘
                                │
                          ┌─────┴─────┐
                          │ Memcached │ + Redis (feed, counter)
                          └───────────┘
```

**Asosiy texnik qarorlar.**
- **Rasm object storage + CDN, metadata Postgres:** katta blob'ni DB'da saqlamaslik. DB kichik, tez; baytlar arzon store'da, CDN'dan yetkaziladi.
- **Logical sharding:** ko'p logical shard bitta fizik Postgres'da. Scale kerak bo'lsa, logical shard'ni boshqa serverga **ko'chirish** (butun rebalance emas).
- **Shard-encoded ID (Snowflake-vari):** ID ichida vaqt + logical shard ID. ID'dan qaysi shardga borishni **hisoblab** olish — markaziy lookup jadval kerak emas.
- **Boring technology:** Postgres, Memcached — sinalgan, jamoa biladi.

**Logical shard'ni og'riqsiz ko'chirish.** (a) yangi serverga logical shard'ni **replikatsiya** qil; (b) sinxron bo'lgach, yozishni yangi serverga o'tkaz (qisqa read-only oyna); (c) eskisini o'chir. Foydalanuvchi ID'dagi logical shard ID o'zgarmaydi — faqat "logical → fizik" xaritasi yangilanadi.

**ID → shard.** `logical_shard = ID'dan bitlarni ajratib olish` → `logical → fizik xarita` (kichik, cache'lanadigan config). Har so'rov to'g'ri serverni ID'ning o'zidan biladi.

**O'rganiladigan dars.** Zerikarli, sinalgan texnologiya kichik jamoa uchun ustunlik — har vosita operatsion narx qo'shadi. Logical sharding scale'ni **bosqichma-bosqich** (birdan emas) qiladi.

---

## 6. Real-time chat trillion-xabar scale (Discord-vari)

**Masshtab taxmini.**
```text
Faraz: 10M concurrent ulanish, trillionlab xabar tarixi:
  Ulanish: 10M WebSocket → gateway node'lar (har biri ~yuz mingdan)
  Xabar: yozish-og'ir, trillionlab → gorizontal DB (Cassandra/ScyllaDB)
  O'qish: kanalning oxirgi N xabari (range o'qish) — tez bo'lishi shart
```

**High-level arxitektura.**
```text
  Client ──WebSocket──▶ ┌────────────────────────────────┐
                        │ Gateway (Elixir/BEAM)           │
                        │ har ulanish = process; guild     │
                        │ hodisalarini shu guild'dagi      │
                        │ hamma ulanishga FAN-OUT           │
                        └───────────────┬────────────────┘
                                        ▼ yozish/o'qish
                        ┌────────────────────────────────┐
                        │ ScyllaDB (Cassandra-mos, C++)    │
                        │ partition: (channel_id, bucket)  │
                        │ → range o'qish (oxirgi xabarlar)  │
                        └────────────────────────────────┘
```

**Asosiy texnik qarorlar.**
- **Elixir/BEAM gateway:** millionlab yengil process; guild (server) hodisalarini shu guild a'zolariga fan-out. WebSocket real-time uchun.
- **ScyllaDB (Cassandra'dan ko'chib):** yozish-og'ir, gorizontal. ScyllaDB C++ (JVM GC yo'q) → past va **barqaror** latency, kamroq node.
- **Partition kaliti `(channel_id, time_bucket)`:** faqat `channel_id` bo'lsa, katta kanal = ulkan partition (hot). Vaqt bucketi (masalan, ~10 kunlik oyna) katta kanalni bir necha partition'ga bo'ladi.

**Hot partition'ni oldini olish.** Partition kalitini **eng katta holat** (million-xabarli kanal) bo'yicha tanlash. Vaqt bucketi bilan har partition o'lchami cheklanADI. O'qish: oxirgi bucket(lar)'ni so'rash (range query, tez).

**Katta kanal (million xabar).** Bucketing bilan avtomatik bo'linADI. O'qishda oxirgi bucket yetadi (odam eski xabarni kamdan-kam qidiradi); eski bucket kerak bo'lsa — alohida so'rov.

**Nega JVM GC latency manbai.** JVM garbage collector vaqti-vaqti bilan **pauza** qiladi (stop-the-world) → o'sha paytda so'rovlar kutADI → p99 latency spike. ScyllaDB (C++, GC yo'q, shard-per-core) buni yo'qotadi.

**O'rganiladigan dars.** Partition kaliti — sekin qotil: noto'g'ri kalit darrov emas, scale oshgach buzADI. Eng issiq/katta holat bo'yicha tanlang. Runtime (GC) ham latency manbai — barqaror past latency uchun poydevorni almashtirish kerak bo'lishi mumkin.

---

## 7. Cell-based arxitektura bilan global servis

**Masshtab taxmini.**
```text
Faraz: 500M foydalanuvchi, global:
  Bitta ulkan tizim → bitta bug butun 500M'ni o'chiradi (blast radius = 100%)
  Cell'larga bo'lish: har cell ~5M foydalanuvchi → 100 cell
  Bitta cell buzilsa → faqat 1% ta'sirlanadi
```

**High-level arxitektura.**
```text
        ┌──────────── Cell router (thin, ishonchli) ─────────────┐
        │  user_id → qaysi cell (mapping)                         │
        └──────────────────────────┬─────────────────────────────┘
                                    ▼
  ┌───Cell 1──────┐  ┌───Cell 2──────┐  ┌───Cell N──────┐
  │ to'liq stack: │  │ to'liq stack: │  │ to'liq stack: │
  │  API + svc +  │  │  API + svc +  │  │  API + svc +  │
  │  DB + cache   │  │  DB + cache   │  │  DB + cache   │
  │  (5M user)    │  │  (5M user)    │  │  (5M user)    │
  └───────────────┘  └───────────────┘  └───────────────┘
   Cell'lar bir-biridan MUSTAQIL — biri o'lsa boshqasi ishlaydi
```

**Asosiy texnik qarorlar.**
- **Cell = to'liq stack nusxasi:** har cell o'z API, servis, DB, cache'ига ega — mustaqil ishlaydi. Foydalanuvchi butunlay bitta cell ichida.
- **Thin cell router:** faqat "user → cell" xaritasini biladi (kichik, o'ta ishonchli, deyarli o'zgarmas mantiq). Router'ning o'zi SPOF bo'lmasligi uchun oddiy va zaxiralangan.
- **Blast radius izolyatsiyasi:** cell buzilsa faqat undagi foydalanuvchilar ta'sirlanadi.
- **Cell-by-cell deployment:** yangi versiya avval 1 cell'da → kuzat → keyingi cell'lar.

**Foydalanuvchini cell'ga yo'naltirish.** `user_id → cell` xaritasi (consistent hashing yoki explicit mapping). Xarita o'ta ishonchli store'da (yoki client'ga login'da beriladi). Yangi foydalanuvchi eng bo'sh cell'ga.

**Cell to'lganda.** Yangi cell qo'shish; yangi foydalanuvchilarni unga yo'naltirish. Mavjud foydalanuvchini ko'chirish kamdan-kam (qimmat) — asosan yangilar orqali balans.

**Cell bo'yicha deployment bosqichi.** Canary'ning cell versiyasi: v2 → cell 1 (1%) → metrikalar yaxshi? → cell 2..5 → ... → hamma. Yomonlashsa: faqat shu cell rollback, qolgani v1'da xavfsiz.

**O'rganiladigan dars.** Scale oshgach savol "tezroq" dan "**izolyatsiya**"ga o'tadi. Cell — failure'ni foydalanuvchilarning kichik qismiga cheklaydi; deployment ham xavfsizroq (cell-by-cell).

---

## 8. Production observability va deployment pipeline

**Arxitektura (observability).**
```text
  So'rov (trace_id butun yo'l bo'ylab ko'chadi):
   Client → API GW → Svc A → Svc B → Svc C → DB
      │        │        │       │       │
      ▼        ▼        ▼       ▼       ▼
   ┌─────────────────────────────────────────┐
   │ METRICS (RPS, p50/p99, error%) → alert   │
   │ LOGS (structured, trace_id bilan)         │
   │ TRACES (trace_id → butun yo'l, har qadam  │
   │  qancha vaqt) → "sekinlik QAYERDA?"       │
   └─────────────────────────────────────────┘
```

**Asosiy texnik qarorlar.**
- **Uch ustun:** metrics (alert — "nimadir buzildi"), logs (nima/qachon), traces (qayerda/nega sekin). Microservice'da traces hal qiluvchi.
- **Distributed tracing:** har so'rovga `trace_id`, u butun zanjir bo'ylab uzatiladi (header'da). Har servis o'z "span"ini shu trace_id bilan yozadi.

**"Sekinlik qayerda?"** Trace'ni ochib, har span vaqtini ko'rish: agar Svc B → DB span 2s bo'lsa, sekinlik DB so'rovida. Metrics muammoni **sezadi**, trace **joyni** topadi.

**Deployment (canary + avtomatik rollback).**
```text
  Kod → CI (test/build) → CD:
    v2 → 1% trafik (canary) → metrikalarni kuzat (error%, p99)
       ┌── yaxshi? → 5% → 25% → 100%
       └── yomon (error/latency chegaradan oshdi)? → AVTOMATIK ROLLBACK
```

**Avtomatik rollback qurish.** Canary metrikalari (error rate, p99) baseline (eski versiya) bilan **taqqoslanADI**. Statistik sezilarli yomonlashuv → deploy pipeline avtomatik eski versiyaga qaytaradi (health-gate). Inson tasdig'i kutilmaydi (tez).

**O'rganiladigan dars.** Diagramma emas, **kuzatuvchanlik** tizimni tirik ushlaydi. Deployni bosqichli (canary) va avtomatik rollback bilan qilib, xatoni kichik % da ushlaysiz.

---

## 9. On-call va incident response tizimi

**Arxitektura (jarayon).**
```text
  Alert (actionable, kam soxta) ──▶ On-call muhandis (rotatsiya)
                                        │
                    ┌───────────────────┴────────────────────┐
                    ▼                                         ▼
              INCIDENT jarayoni:                        agar kichik → o'zi hal
   ┌──────────────────────────────────────────────────────────┐
   │ 1. DETECT (alert/monitoring)                              │
   │ 2. MITIGATE (AVVAL to'xtat: rollback/failover/rate-limit) │
   │ 3. TIKLASH (to'liq servis)                                │
   │ 4. BLAMELESS POSTMORTEM (aybsiz sabab tahlili, action)    │
   └──────────────────────────────────────────────────────────┘
```

**Asosiy texnik qarorlar.**
- **Alerting:** har alert **actionable** (aniq harakat) va **kam soxta**. Alert threshold'lari to'g'ri sozlanADI (SLO'ga asoslanган).
- **On-call rotatsiya:** navbatchi almashib turadi (burnout'ni oldini olish), aniq escalation zanjiri (javob bermasa → keyingisi).
- **Runbook:** har alert uchun "nima qilish" hujjati (mitigate qadamlari).

**Alert fatigue'ni oldini olish.** Soxta/noise alert'larni **kamaytir**: threshold'ni to'g'irla, takrorlanuvchilarni birlashtir (dedupe), faqat foydalanuvchiga ta'sir qiladigan (symptom-based, SLO) alert. Ko'p soxta alert → e'tibor yo'qoladi → haqiqiy alert o'tkazib yuboriladi.

**"Avval to'xtat, keyin sabab qidir".** Jarayonga aylantirish: incident boshlanganда birinchi qadam — **mitigate** (oxirgi deploy'ni rollback, traffic'ni sog' region'ga failover, muammoli funksiyani flag bilan o'chirish). Root cause **postmortem'da**, incident vaqtida emas. Runbook'da "mitigate" bo'limi "diagnose"dan **oldin**.

**Blameless postmortem.** "Kim aybdor?" emas, "**qaysi himoya** ushlamadi?". Action item'lar (avtomatik test, alert, guard) — tizim keyingi safar **o'zi** ushlashi uchun.

**O'rganiladigan dars.** Incident'da foydalanuvchi azobini **to'xtatish** birinchi, sabab keyin. Aybsiz madaniyat haqiqiy sabablarni ochadi va tizim o'rganadi.

---

## 10. Capacity planning "Qora juma" (Black Friday) uchun

**Masshtab taxmini.**
```text
Oddiy: 10K RPS. Black Friday peak: 10x = 100K RPS kutilADI:
  → 100K'ni ko'tara oladimi? LOAD TEST bilan chegarani top
  → auto-scaling + headroom (peak'dan zaxira)
  → chegaraga yaqinlashsa graceful degradation rejasi
```

**Arxitektura (tayyorgarlik).**
```text
  1. LOAD TEST (peak'dan oldin, sun'iy yuk):
       yukni 10K → 50K → 100K → ... oshirib, TIZIM QAYERDA sinishini top
       (DB? cache? bitta servis?) → bottleneck'ni oldindan tuzat

  2. AUTO-SCALING: metrika (CPU/RPS/queue) oshsa avtomatik instance qo'sh
       (lekin scale-up VAQT oladi → oldindan warm-up / pre-scale)

  3. HEADROOM: 100% to'la ishlatma — peak'dan ~30-40% zaxira

  4. GRACEFUL DEGRADATION rejasi (yuk chegaradan oshsa):
       muhim BO'LMAGAN funksiyalarni o'chir (tavsiya, analytics, rich UI)
       yadro (checkout, to'lov) ishlashda davom etsin
```

**Asosiy texnik qarorlar.**
- **Load testing:** production'ga o'xshash muhitда, real trafik shakli bilan yuk berib **chegarani** (breaking point) topish. Bottleneck (odatda DB yoki bitta servis) oldindan ochiladi.
- **Auto-scaling + pre-scale:** avtomatik, lekin scale vaqt olADI — kutilgan voqeadan oldin **pre-scale** (qo'lda oldindan kengaytirish).
- **Headroom:** 100% ni ishlatmaslik — kutilmagan spike va scale kechikishiga zaxira.

**Chegarani topish (load test).** Yukni bosqichma-bosqich oshirib, latency/error qaysi nuqtada portlashini kuzatish. O'sha RPS — hozirgi chegara. Bottleneck komponentni aniqlab (trace/metrics), oldindan optimallash yoki scale.

**Nimani birinchi "o'chirish" (degradation).** Prioritet: **yadro** (checkout, to'lov, savat) hech qachon; **ikkilamchi** (tavsiya, "boshqalar ko'rgan", rich animatsiya, analytics) yuk oshsa avval o'chiriladi (feature flag bilan). Maqsad: yadro ishlashda davom etsin, foydalanuvchi xarid qila olsin.

**O'rganiladigan dars.** Peak'ni **taxmin qilmang — sinang** (load test). Auto-scaling yetmaydi (vaqt olADI) — pre-scale + headroom kerak. Degradation rejasi oldindan (incident vaqtida emas) tayyor bo'lsin: nima muhim, nima o'chsa bo'ladi.

---

← [System Design bo'limiga qaytish](../../system-design/README.md)
