# Production Tizim Misollari

Klassik case study'lar "nazariy jihatdan qanday loyihalanadi" ni ko'rsatadi. Bu bo'lim esa **real kompaniyalar production'da aslida qanday qurgan** ini yoritadi — Netflix, Uber, WhatsApp, Twitter/X, Instagram, Discord va Amazon/Google darajasidagi umumiy tamoyillar. Maqsad — arxitektura qarorlarining **sababini** (nega aynan shu texnologiya, shu pattern) tushunish va ulardan o'rganiladigan **dars** (lesson) ni olib qolish.

**⚠️ Ehtiyot bo'l:** Bu misollar **ochiq manbalar** (engineering bloglar, konferensiya tech talk'lari, muhandislik intervyulari) asosida umumlashtirilgan. Aniq **raqamlar taxminiy** va yillar davomida o'zgargan — ularni "tartib kattaligi" (order of magnitude) sifatida qabul qiling, aniq fakt sifatida emas. Har bir kompaniyaning arxitekturasi doimo evolyutsiyada; bu yerda "abadiy haqiqat" emas, balki **fikrlash namunasi** beriladi.

**💡 Tushuncha:** Real tizimlarda "eng to'g'ri" arxitektura yo'q — bor-yo'g'i muayyan cheklovlar (jamoa hajmi, budjet, latency talabi, mavjud texnologiya) ostidagi **trade-off'lar**. Netflix'ning yechimi WhatsApp'ga to'g'ri kelmaydi, va aksincha. Har misolda "**qaysi cheklov ularni shu qarorga majbur qildi?**" degan savolni bering.

## Mundarija

- [1. Netflix — microservices va resilience](#1-netflix)
- [2. Uber — real-time matching va geo](#2-uber)
- [3. WhatsApp — kam server, katta scale](#3-whatsapp)
- [4. Twitter / X — timeline va fan-out](#4-twitter)
- [5. Instagram — kichik jamoa, katta scale](#5-instagram)
- [6. Discord — real-time chat va voice](#6-discord)
- [7. Amazon / Google — umumiy tamoyillar](#7-amazon-google)
- [Production'da nima muhim](#production-muhim)
- [Umumiy savollar (Q&A)](#qa)
- [Masalalar](#masalalar)

---

<a id="1-netflix"></a>
## 1. Netflix — microservices va resilience

### Muammo va masshtab

Netflix — dunyo internet trafigining sezilarli qismini (peak vaqtda ba'zi mintaqalarda ~15%) tashkil qiladigan video streaming platformasi: yuzlab million obunachi, minglab qurilma turi, o'nlab mintaqada past latency bilan yuqori sifatli video. Ikki tomonlama muammo bor: **(1)** ulkan hajmdagi videoni butun dunyoga yetkazish (bandwidth), **(2)** yuzlab microservice'dan iborat backend'ni ishonchli ushlab turish.

Netflix'ning muhim qarori — 2008-2016 yillarda o'z data-markazlaridan **butunlay AWS bulutiga** ko'chib o'tish. Bu ularni "everything fails" falsafasiga majbur qildi: bulutda instance'lar istalgan payt o'ladi.

### High-level arxitektura

```text
                         ┌──────────────────────────┐
   Foydalanuvchi ───────▶│   Open Connect (CDN)     │  ◀── VIDEO baytlari
   (TV, telefon)         │  ISP ichidagi appliance  │      (butun trafik hajmi)
        │                └──────────────────────────┘
        │ control-plane (login, katalog, "play" bosish)
        ▼
   ┌─────────────┐
   │ API Gateway │  ← Zuul: routing, auth, rate limit, canary
   │   (Zuul)    │
   └──────┬──────┘
          │
   ┌──────┴────────────────────────────────────────┐
   │            Microservices (yuzlab)              │
   │  ┌─────────┐ ┌──────────┐ ┌───────────────┐    │
   │  │ Auth    │ │ Katalog  │ │ Recommendation│    │
   │  └─────────┘ └──────────┘ └───────────────┘    │
   │  ┌─────────┐ ┌──────────┐ ┌───────────────┐    │
   │  │ Playback│ │ Billing  │ │ Encoding/     │    │
   │  │ (DRM,   │ │          │ │ Transcoding   │    │
   │  │  manifest)│└──────────┘ └───────────────┘   │
   │  └─────────┘                                    │
   │   ↕ har servis: Hystrix (circuit breaker)       │
   └────────────────┬───────────────────────────────┘
                    │
          ┌─────────┴─────────┐
          │ Cassandra (yozish- │  EVCache (Memcached asosida)
          │  og'ir, multi-DC)  │  Kafka (event stream)
          └───────────────────┘
```

**💡 Tushuncha:** Diagrammadagi eng muhim ajratish — **control-plane** (Zuul orqali o'tadigan "nima ko'rsatish, kim kirdi") va **data-plane** (Open Connect orqali oqadigan haqiqiy video baytlari) alohida yo'llarda. Video hech qachon Zuul yoki microservice'lardan o'tmaydi — u to'g'ridan-to'g'ri ISP ichidagi CDN appliance'dan foydalanuvchiga oqadi. Bu markaziy backend'ni video trafigining ulkan hajmidan **butunlay himoya qiladi**.

### Asosiy texnik qarorlar

| Qaror | Texnologiya / pattern | Nega |
|-------|----------------------|------|
| API kirish nuqtasi | **Zuul** (edge gateway) | routing, auth, dynamic filter, canary — barcha edge mantiq bir joyda |
| Resilience | **Hystrix** (circuit breaker, bulkhead, timeout) | bitta sekin servis butun so'rovni bloklamasin |
| Failure testing | **Chaos Monkey** (Simian Army) | ataylab instance o'ldirib, "fault tolerance" ni doimo tekshirish |
| Video yetkazish | **Open Connect** (o'z CDN) | ISP ichiga appliance qo'yib, backbone trafik va latency kamaytirish |
| Video tayyorlash | **encoding/transcoding pipeline** | har film o'nlab bitrate/codec/qurilma varianti (per-title encoding) |
| Ma'lumot bazasi | **Cassandra** + **EVCache** | yozish-og'ir, multi-datacenter, past latency o'qish |
| Tavsiya (recommendation) | ML pipeline + A/B test | ko'rish vaqtini oshirish — bosh sahifadagi har element personalizatsiya |

### Muhim patternlar

- **Circuit breaker (Hystrix):** downstream servis sekinlashsa yoki o'lsa, unga qo'ng'iroqni "uzib" (open) fallback qaytariladi (masalan, personalizatsiyalanmagan default tavsiya). Bu **cascading failure** (bitta servis o'lishi zanjirli qulashga olib kelishi) ni oldini oladi.
- **Chaos Engineering:** Chaos Monkey production'da tasodifiy instance'larni o'ldiradi. Falsafa: "agar failure muqarrar bo'lsa, uni **doimo mashq qil**, tuni bilan ogohsiz kutma". Bu Netflix'ni haqiqatan ham chidamli qildi.
- **Per-title encoding:** har video uchun optimal bitrate egri chizig'i alohida hisoblanadi — oddiy multfilm va tez harakatli film bir xil bitrate talab qilmaydi. Bu bandwidth'ni sezilarli tejaydi.

### O'rganiladigan dars (lesson)

> **Failure — istisno emas, norma.** Netflix'ning butun arxitekturasi "har narsa o'ladi" (everything fails, all the time) taxminiga qurilgan. Circuit breaker, timeout, fallback, chaos testing — bularning hammasi failure'ni oldini olish emas, balki **failure bo'lganda ham tizim ishlashda davom etishi** uchun. Katta scale'da 100% uptime yo'q — bor-yo'g'i **graceful degradation** (video sifati pasayadi, lekin ishlaydi).

---

<a id="2-uber"></a>
## 2. Uber — real-time matching va geo

### Muammo va masshtab

Uber — haydovchi va yo'lovchini **real-time** bog'laydigan marketplace: dunyo bo'ylab minglab shahar, sekundiga o'n minglab so'rov, million faol haydovchi joylashuvi (location) doimo yangilanadi. Asosiy texnik qiyinchilik — **geo-spatial matching**: "shu nuqtaga eng yaqin bo'sh haydovchini past latency bilan top" — buni butun shahar/dunyo miqyosida qilish.

Uber dastlab monolit (Python) bilan boshlagan, keyin yuzlab microservice'ga ajralgan. Ular geo-indexlash uchun o'z ochiq kutubxonasi **H3** (hexagonal hierarchical grid) ni yaratdi.

### High-level arxitektura

```text
  Yo'lovchi app                          Haydovchi app
      │  "trip so'rov"                        │  location update (har ~4s)
      ▼                                       ▼
  ┌─────────────┐                    ┌───────────────────┐
  │ API Gateway │                    │ Location ingest    │
  └──────┬──────┘                    │ (kafka'ga stream)  │
         │                           └─────────┬─────────┘
         ▼                                     ▼
  ┌───────────────────┐          ┌────────────────────────────┐
  │ Dispatch / Match  │◀────────▶│ Geo-index (H3 hexagon → shu │
  │ service (DISCO)   │          │  hujayradagi haydovchilar)  │
  └────────┬──────────┘          └────────────────────────────┘
           │
   ┌───────┴─────────┬──────────────┬────────────────┐
   ▼                 ▼              ▼                ▼
 ┌────────┐    ┌──────────┐   ┌───────────┐   ┌───────────┐
 │ Pricing│    │ Trip     │   │ Payment   │   │ ETA /     │
 │ (surge)│    │ state    │   │           │   │ routing   │
 └────────┘    └──────────┘   └───────────┘   └───────────┘
       ↕ event-driven (Kafka): trip.requested, trip.matched, trip.completed ...
```

**💡 Tushuncha:** Uber'ning arxitekturasi **event-driven** — har trip holati o'zgarishi (so'ralди, moslashtirildi, boshlandi, tugadi) event sifatida Kafka'ga chiqadi. Pricing, analytics, fraud detection, payment — hammasi shu event oqimini "eshitadi". Bu servislarni bir-biridan **ajratadi** (decouple): payment servisi dispatch servisini bilmasligi mumkin, u faqat `trip.completed` event'ini kutadi.

### H3 — geo-indexing yuragi

Yer sharini o'lchamli **olti burchak (hexagon)** hujayralarga bo'lish. Nega hexagon (kvadrat emas)? Chunki hexagon markazidan barcha qo'shni hujayra markazigacha masofa **bir xil** (kvadratda diagonal qo'shni uzoqroq) — bu "yaqinlik" hisoblarini aniqroq qiladi.

```text
Har haydovchi joylashuvi → H3 hujayra ID (masalan, "8a2a1072b59ffff")
Matching:  yo'lovchi hujayrasini top → shu + qo'shni hujayralardagi
           haydovchilarni ol (k-ring) → eng yaqin/tez yetadiganini tanla

    ⬡ ⬡ ⬡          Yo'lovchi markaziy hujayrada.
   ⬡ ⬡[P]⬡         k=1 ring: 6 qo'shni hujayra ham qidiriladi.
    ⬡ ⬡ ⬡          Kerak bo'lsa k kengaytiriladi (kam haydovchi bo'lsa).
```

Hierarchik: hujayralar turli **rezolyutsiya** (resolution) darajasida — shahar darajasidan ko'cha darajasigacha. Bu "keng qidiruv → torroq qidiruv" ni arzon qiladi.

### Asosiy texnik qarorlar

| Qaror | Yechim | Nega |
|-------|--------|------|
| Geo-index | **H3** (hexagonal grid) | teng masofali qo'shnilar, hierarchik rezolyutsiya, "eng yaqin" tez |
| Servislararo | **Event-driven** (Kafka) | trip lifecycle event'lari orqali decoupling |
| Matching | **DISCO** dispatch (supply/demand) | real-time supply-demand moslashtirish |
| Surge pricing | talab/taklif nisbati per-hexagon | kam taklif → narx oshadi → ko'proq haydovchi jalb / talab kamayadi |
| Trip state | statexmachine / durable store | trip holati (requested→ongoing→done) ishonchli saqlanishi shart |
| Storage | Schemaless (MySQL ustida) + keyin turli DB | tez o'sish davrida moslashuvchan sxema |

### O'rganiladigan dars (lesson)

> **Domenga mos ma'lumot strukturasini tanlang.** Uber oddiy "lat/lng + radius so'rov" bilan qutulmadi — o'ta yuqori tezlikda "eng yaqin" ni topish uchun maxsus **geo-index (H3)** kerak bo'ldi. Dars: real scale'da umumiy yechim yetmaydi — muammoning **shakli** (bu yerda: geo-yaqinlik) ma'lumot strukturasini belgilaydi. Ikkinchi dars: **surge pricing** — bu texnik emas, iqtisodiy mexanizm bilan tizim yukini boshqarish (talab oshsa narx bilan uni tartibga solish).

---

<a id="3-whatsapp"></a>
## 3. WhatsApp — kam server, katta scale

### Muammo va masshtab

WhatsApp mashhur bo'lgan sabab — **hayratlanarli darajada kam muhandis va server** bilan ulkan scale. Ma'lum bo'lgan holat: ~50 muhandis bilan yuzlab million foydalanuvchiga xizmat, bitta serverda ~million dan ortiq bir vaqtdagi (concurrent) ulanish. Bu — extremal **operatsion soddalik** namunasi.

Sirning kaliti — **Erlang/OTP** (va uning BEAM virtual mashinasi). Erlang telecom uchun yaratilgan: millionlab yengil "process", "let it crash" falsafasi, issiq kodni yangilash (hot code swap), yuqori parallellik.

### High-level arxitektura

```text
   Telefon A                                        Telefon B
      │  doimiy TCP ulanish (custom XMPP-vari protokol)  │
      ▼                                                  ▲
  ┌──────────────────────────────────────────────────────┐
  │                  Chat server (Erlang)                 │
  │  ┌────────────────────────────────────────────────┐  │
  │  │ Har ulanish = yengil Erlang process             │  │
  │  │ (millionlab process bitta node'da)              │  │
  │  └────────────────────────────────────────────────┘  │
  │   Xabar routing: A → server → (B onlaynmi?)           │
  │      onlayn  → darrov yetkaz                          │
  │      oflayn  → NAVBATGA qo'y, B ulanganda yetkaz      │
  └──────────────────────────┬───────────────────────────┘
                             │
                    ┌────────┴─────────┐
                    │ Mnesia / offline  │  (yetkazilmagan xabar
                    │ message store     │   store — yetkazilgach o'chadi)
                    └──────────────────┘

  MUHIM: WhatsApp xabarni DOIMIY saqlamaydi — yetkazilgach serverdan o'chadi.
         Butun tarix telefonda (E2E encryption bilan) turadi.
```

**💡 Tushuncha:** WhatsApp'ning katta "hiylasi" — u **ma'lumot omborini minimallashtirdi**. Xabar server'da faqat **yetkazilgunicha** turadi (store-and-forward), keyin o'chadi. Butun chat tarixi **telefonda**. Bu server tomonda ulkan storage, sharding, replikatsiya muammolarini **butunlay yo'q qiladi** — shuning uchun kam server yetadi. "Eng arzon storage — saqlamaydiganingiz".

### Asosiy texnik qarorlar

| Qaror | Yechim | Nega |
|-------|--------|------|
| Til / runtime | **Erlang/OTP (BEAM)** | millionlab yengil process, past latency, "let it crash", hot upgrade |
| Ulanish | **doimiy TCP** (long-lived) | push uchun — server istalgan payt xabar yubora oladi (poll emas) |
| Xabar saqlash | **store-and-forward**, yetkazilgach o'chirish | server-side storage'ni minimallashtirish |
| Delivery kafolat | at-least-once + client dedup | tarmoq uzilsa qayta yetkazish; ✓ ✓✓ (sent/delivered) belgilar |
| Tartib (ordering) | per-conversation ketma-ketlik | bitta suhbatda xabarlar tartibi buzilmasin |
| Encryption | **E2E** (Signal protokoli) | server ham xabar mazmunini ko'ra olmaydi |
| Presence | onlayn/last-seen holati (mas'uliyatli) | ulanish holatidan chiqariladi |

### Delivery, ordering va presence

- **Delivery belgilar:** ✓ (server qabul qildi), ✓✓ (qurilmaga yetkazildi), ko'k ✓✓ (o'qildi). Bular **acknowledgement** (ack) mexanizmi — har bosqichda client server'ga tasdiq yuboradi.
- **Ordering:** global tartib emas, **per-conversation** tartib yetarli — bitta suhbatdagi xabarlar to'g'ri ketma-ketlikda kelishi kifoya (Lamport soatiga o'xshash mantiq).
- **Presence** (kim onlayn): bu **hard problem** — million ulanish holatini real-time tarqatish qimmat. WhatsApp buni faqat "sizga muhim odamlar" (suhbatdagi tomon) uchun cheklaydi, hammaga emas.

**⚠️ Ehtiyot bo'l:** "Presence" (onlayn holati) ko'pincha undan baholanmaydi. Har foydalanuvchi holati o'zgarganда, **uni kuzatayotgan hamma**ga bildirish kerak — bu "fan-out" muammosi. Millionlab ulanishда naive yechim (har holat o'zgarishini hammaga broadcast) tizimni ag'daradi. Real yechimlar buni **cheklaydi** (faqat aktiv suhbat, throttling).

### O'rganiladigan dars (lesson)

> **Soddalik — super-power.** WhatsApp murakkab distributed database qurmadi — u muammoni **qayta ta'rifladi**: "xabarni saqlamaymiz, faqat uzatamiz". Bu ularga microservice armiyasi kerak bo'lmadi. Dars: eng yaxshi arxitektura — **qurmaganingiz**. To'g'ri runtime tanlash (Erlang'ning concurrency modeli aynan messaging'ga mos) va scope'ni radikal cheklash minglab server o'rniga yuzlab server bilan ishlash imkonini beradi.

---

<a id="4-twitter"></a>
## 4. Twitter / X — timeline va fan-out

### Muammo va masshtab

Twitter/X'ning markaziy muammosi — **home timeline** (uy tasmasi): "men obuna bo'lgan odamlarning tvitlari, so'nggisi yuqorida". Yuzlab million foydalanuvchi, har biri o'z tasmasini **past latency** bilan ko'radi. Qiyinchilik — masshtab assimetriyasi: oddiy foydalanuvchining 100 obunachisi bor, mashhur odam (celebrity) 100 million.

Asosiy trade-off — **fan-out on write vs fan-out on read**.

### Ikki strategiya

```text
FAN-OUT ON WRITE (push):
  Tvit yozilganda → DARROV barcha obunachi timeline'iga nusxa qo'yiladi
  ┌─────────┐  tvit   ┌─────────────────────────────────┐
  │ User A  │────────▶│ A'ning har follower'i uchun cache │
  └─────────┘          │ timeline'ga INSERT (pre-compute) │
                       └─────────────────────────────────┘
  ✓ O'qish TEZ (timeline tayyor, cache'dan o'qi)
  ✗ Yozish QIMMAT (100M follower = 100M insert!) ← celebrity muammo

FAN-OUT ON READ (pull):
  Timeline SO'RALGANDA → obuna bo'lganlarning tvitlarini yig'ib, saralab beriladi
  ✓ Yozish ARZON (bitta insert)
  ✗ O'qish QIMMAT (har so'rovda merge/sort) ← faol o'qish uchun yomon
```

**💡 Tushuncha:** Twitter **read:write** nisbati juda katta (odamlar yozganidan ko'ra ko'proq o'qiydi). Shu sabab, umumiy holatda **fan-out on write** (pre-compute) yaxshi — o'qishni tez qiladi. Lekin bu **celebrity muammosi**ga uriladi: mashhur odam bitta tvit yozsa, uni 100M timeline'ga yozish kerak bo'ladi — bu portlash.

### Hybrid yechim (celebrity muammosi)

Twitter'ning amaliy yechimi — **aralash (hybrid)**:

```text
Oddiy foydalanuvchi tvitи   → FAN-OUT ON WRITE (follower cache'iga push)
Celebrity (juda ko'p follower) → FAN-OUT ON READ (push QILMAYDI)

Home timeline yig'ish:
  1. Cache'dan pre-computed qismni o'qi (oddiy odamlar tvitlari)
  2. + men obuna bo'lgan CELEBRITY'larni alohida so'ra (pull)
  3. Ikkalasini MERGE qil + ML ranking → ko'rsat
```

Bu ikki yomon nuqtani ham chetlab o'tadi: celebrity uchun 100M insert yo'q, oddiy foydalanuvchi uchun o'qish tez.

### Asosiy texnik qarorlar

| Qaror | Yechim | Nega |
|-------|--------|------|
| Timeline (ko'pchilik) | fan-out on write + **cache** (Redis) | o'qish-og'ir yuk uchun tez timeline |
| Celebrity | fan-out on read (hybrid) | 100M insert portlashini oldini olish |
| Timeline saqlash | cache'da pre-computed ro'yxat (tweet ID) | timeline = ID ro'yxati, mazmun alohida hydrate qilinadi |
| Ranking | xronologik → **ML ranked** feed | eng relevantni yuqoriga (engagement) |
| Real-time | streaming / push (yangi tvit) | "N ta yangi tvit" bildirishi |
| Search | max/qidiruv indeks (Earlybird) | real-time full-text qidiruv |

### O'rganiladigan dars (lesson)

> **Bir yechim hamma uchun ishlamaydi — segmentga bo'ling.** Twitter'ning eng katta darsi: fan-out on write "yaxshi", lekin celebrity'da buzADI; fan-out on read "arzon", lekin faol o'qishda buzADI. Yagona to'g'ri javob **yo'q** — foydalanuvchini **segmentlab** (oddiy vs celebrity), har segmentga mos strategiya berish kerak. Bu system design'ning umumiy darsi: **"90% holat"** uchun optimallashtiring, lekin **"og'ir 10%"** (heavy hitter, hot key, celebrity) ni alohida ishlov bering.

---

<a id="5-instagram"></a>
## 5. Instagram — kichik jamoa, katta scale

### Muammo va masshtab

Instagram mashhur bo'lgan yana bir hikoya — **juda kichik jamoa** (dastlab ~o'nlab muhandis) bilan o'nlab million foydalanuvchiga xizmat. Ular murakkab, ekzotik texnologiya emas, **oddiy va sinalgan** (boring but proven) texnologiyani tanladi: **PostgreSQL**, Redis, Memcached, va keyinchalik sharding.

Muammoning yuragi — **foto storage** (petabaytlab rasm) va **feed** (obuna bo'lgan odamlarning postlari).

### High-level arxitektura

```text
   Foydalanuvchi
      │  rasm yuklash / feed so'rov
      ▼
  ┌──────────────┐     ┌──────────────────────────┐
  │ App server   │────▶│  Object storage (rasm     │
  │ (Django /    │     │  fayllari) + CDN           │
  │  Python)     │     └──────────────────────────┘
  └──────┬───────┘
         │ metadata (kim, qachon, caption, like)
         ▼
  ┌──────────────────────────────────────────────┐
  │  PostgreSQL (sharded)                          │
  │  ┌─────────┐ ┌─────────┐ ┌─────────┐          │
  │  │ Shard 1 │ │ Shard 2 │ │ Shard N │  ...      │
  │  └─────────┘ └─────────┘ └─────────┘          │
  │   sharding kaliti: user ID → logical shard     │
  └──────────────────────────────────────────────┘
         │
    ┌────┴─────┐
    │ Memcached│  (o'qish kesh) + Redis (feed, counter)
    └──────────┘
```

**💡 Tushuncha:** Instagram **rasm fayllarini** (katta, o'zgarmas) object storage + CDN'ga, **metadata'ni** (kichik, so'raladigan) PostgreSQL'ga ajratdi. Bu klassik pattern — "**katta blob'ni DB'da saqlama**". DB faqat "kim, qachon, qaysi rasm, nechta like" ni biladi; haqiqiy baytlar arzon object store'da va foydalanuvchiga CDN'dan yetadi.

### Sharding: kichik jamoa qanday scale qildi

Instagram'ning nafis sharding yechimi — **logical shard'lar** (mantiqiy shard). Bir nechta logical shard bitta fizik Postgres'da yashaydi; scale kerak bo'lsa, logical shard'ni boshqa fizik serverga **ko'chirish** oson.

ID generatsiya ham aqlli: har ID ichига **shard ID** (qaysi shardda) va vaqt joylanadi (Snowflake'ga o'xshash). Shunday qilib ID'dan qaysi shardga borishni **hisoblab** olish mumkin — markaziy lookup jadval kerak emas.

```text
64-bit ID tarkibi (soddalashtirilgan):
  [ 41 bit: vaqt (ms) ] [ 13 bit: logical shard ID ] [ 10 bit: sequence ]
   └ vaqt-tartibli          └ qaysi shard              └ shu ms ichida noyob
```

### Asosiy texnik qarorlar

| Qaror | Yechim | Nega |
|-------|--------|------|
| Asosiy DB | **PostgreSQL** (sinalgan) | ekzotik emas, jamoa biladi, ishonchli |
| Rasm | object storage + **CDN** | katta blob DB'dan tashqarida, CDN'dan yetkazish |
| Sharding | **logical shard** (Postgres ichida) | fizik serverga ko'chirish oson, scale bosqichma-bosqich |
| ID | shard-ичida encoded (Snowflake-vari) | ID'dan shardni hisoblab ol, markaziy lookup yo'q |
| Kesh | **Memcached** + Redis | o'qish-og'ir yukni DB'dan uzoqlashtirish |
| Feed | pre-computed / fan-out (Twitter'ga o'xshash) | timeline tez o'qilsin |

### O'rganiladigan dars (lesson)

> **Zerikarli texnologiya — ustunlik (boring technology).** Instagram eng yangi, chiroyli database'ni quvmadi — PostgreSQL, Memcached kabi **sinalgan** vositalarni oldi va ularni yaxshi bilan ishlatdi. Kichik jamoa uchun bu hal qiluvchi: har texnologiya "operatsion yuk" (o'rganish, debug, on-call) qo'shadi. Dars: **jamoangiz biladigan** va **sinalgan** vositani tanlang; innovatsiyani biznes muammosi talab qilgan **bitta** joyga sarflang, hamma joyga emas.

---

<a id="6-discord"></a>
## 6. Discord — real-time chat va voice

### Muammo va masshtab

Discord — millionlab **bir vaqtda** onlayn foydalanuvchi, real-time chat va ovozli (voice) muloqot. Ikki og'ir muammo: **(1)** millionlab doimiy WebSocket ulanishini boshqarish, **(2)** ulkan hajmdagi xabarni (trillionlab) saqlash va tez o'qish.

Discord'ning tanlovi — WhatsApp singari **Elixir** (Erlang/BEAM ustida) real-time qatlam uchun; xabar saqlash uchun esa dastlab **Cassandra**, keyin **ScyllaDB** ga ko'chdi.

### High-level arxitektura

```text
   Client (desktop/mobile)
      │  WebSocket (Gateway) — real-time hodisalar
      ▼
  ┌────────────────────────────────────┐        ┌──────────────────┐
  │ Gateway (Elixir) — WebSocket fan-out│        │ Voice server     │
  │ har ulanish = BEAM process          │        │ (media, WebRTC / │
  │ guild (server) hodisalarini push    │        │  UDP, SFU)       │
  └──────────────┬─────────────────────┘        └──────────────────┘
                 │ xabar yozish/o'qish
                 ▼
  ┌────────────────────────────────────────────────┐
  │  ScyllaDB (Cassandra-mos, C++ da qayta yozilgan)│
  │  Partition kaliti: (channel_id, vaqt bucketi)   │
  │  → trillionlab xabar, tez range o'qish           │
  └────────────────────────────────────────────────┘
```

**💡 Tushuncha:** Discord xabarlarini **partition** qilishda muhim dars oldi. Dastlab partition kaliti faqat `channel_id` edi — lekin **juda katta kanal** (million xabarli) bitta ulkan partition'ga aylanib, "hot partition" muammosini keltirdi. Yechim: kalitga **vaqt bucketi** qo'shish — `(channel_id, bucket)`, shunda katta kanal ham vaqt bo'yicha bir necha partition'ga bo'linadi.

### Nega Cassandra → ScyllaDB

Discord dastlab **Cassandra** ishlatdi (yozish-og'ir, gorizontal scale). Lekin muammolar: JVM **garbage collection** paузalari (latency spike), operatsion murakkablik, "hot partition". Ular **ScyllaDB** ga ko'chdi — bu Cassandra bilan mos (bir xil model/protokol), lekin **C++** da yozilgan (JVM/GC yo'q), shard-per-core arxitektura. Natija: kamroq node, past va barqaror latency.

### Asosiy texnik qarorlar

| Qaror | Yechim | Nega |
|-------|--------|------|
| Real-time qatlam | **Elixir/BEAM** | millionlab yengil process, WebSocket fan-out uchun ideal |
| Xabar saqlash | **Cassandra → ScyllaDB** | yozish-og'ir, gorizontal scale; ScyllaDB — GC yo'q, past latency |
| Partition | `(channel_id, time bucket)` | hot partition (katta kanal) muammosini bo'lish |
| Voice | media server + **WebRTC/UDP** | past latency ovoz uchun UDP (TCP emas) |
| Presence / fan-out | guild bo'yicha hodisa push | server (guild)dagi hamma real-time yangilanadi |

### O'rganiladigan dars (lesson)

> **Partition kaliti — sekin qotil (silent killer).** Discord'ning "hot partition" darsi klassik: noto'g'ri partition kaliti tizimni **darrov** buzmaydi — u miqyos oshgach, ba'zi partition'lar ulkanlashib, latency asta-sekin yomonlashadi. Dars: partition/shard kalitini tanlashда **eng katta holat** (biggest tenant, hottest channel) ni o'ylang, o'rtachani emas. Ikkinchi dars: runtime (JVM GC) ham latency manbai bo'lishi mumkin — barqaror past latency uchun ba'zan poydevorni (ScyllaDB) almashtirish kerak.

---

<a id="7-amazon-google"></a>
## 7. Amazon / Google — umumiy tamoyillar

Har bir servisni alohida yoritish o'rniga, Amazon va Google darajasidagi tashkilotlardan chiqadigan **umumiy tamoyillar**ni jamlaymiz — bular "hyperscale"da qayta-qayta uchraydi.

### Cell-based architecture (hujayrali arxitektura)

Tizimni mustaqil, izolyatsiyalangan **hujayralar (cell)**ga bo'lish — har hujayra to'liq stack'ning nusxasi va foydalanuvchilarning bir qismiga xizmat qiladi.

```text
  ┌────────── Region ──────────────────────────────┐
  │  ┌─Cell 1─┐  ┌─Cell 2─┐  ┌─Cell 3─┐  ┌─Cell N─┐ │
  │  │ to'liq │  │ to'liq │  │ to'liq │  │ to'liq │ │
  │  │ stack  │  │ stack  │  │ stack  │  │ stack  │ │
  │  └────────┘  └────────┘  └────────┘  └────────┘ │
  │   users 0-9%  10-19%      20-29%      ...        │
  └────────────────────────────────────────────────┘
   Cell 2 buzilsa → faqat 10-19% foydalanuvchi ta'sirlanadi
   (blast radius CHEKLANGAN)
```

**💡 Tushuncha:** Cell-based arxitekturaning maqsadi — **blast radius** (portlash radiusi) ni cheklash. Agar hamma foydalanuvchi bitta ulkan tizimda bo'lsa, bitta bug butun dunyoni o'chiradi. Cell'larga bo'linsa, failure bitta cell (foydalanuvchilarning kichik %) bilan cheklanadi. Deployment ham cell-by-cell — yangi versiya avval bitta cell'da sinaladi.

### Boshqa asosiy tamoyillar

| Tamoyil | Ma'nosi | Nega muhim |
|---------|---------|------------|
| **Everything fails, all the time** | har disk, server, tarmoq, AZ o'ladi | dizayn failure'ni **kutishi** kerak, kutulmasligi emas |
| **Service ownership** ("you build it, you run it") | jamoa o'z servisini yozADI **va** on-call qiladi | mas'uliyat va sifat bir jamoada |
| **Blast radius izolyatsiyasi** | cell, shuffle sharding, bulkhead | bitta failure hammani o'chirmasin |
| **Design for failure** | timeout, retry (backoff), idempotency, fallback | qisman failure'da ham ishlash |
| **Automate everything** | deploy, scale, recovery — qo'lда emas | inson xatosi va sekinlik manbai |
| **Data durability > availability (ba'zan)** | S3-vari: ma'lumot yo'qolmasligi ustuvor | yo'qolgan ma'lumotni tiklab bo'lmaydi |

**⚠️ Ehtiyot bo'l:** "You build it, you run it" — bu tashkiliy tamoyil, texnik emas, lekin arxitekturaga **bevosita** ta'sir qiladi. Agar jamoa o'z kodini tunda on-call qilsa, ular yaxshi observability, alerting, graceful degradation qurishга **rag'batlantiriladi** — chunki buzilsa **o'zlari** uyg'onadi. Bu Conway qonunining bir ko'rinishi: tashkilot tuzilishi arxitekturani shakllantiradi.

### O'rganiladigan dars (lesson)

> **Izolyatsiya — hyperscale'ning asosi.** Amazon/Google darajasida bitta katta tizim mumkin emas — hamma narsa **bo'linadi**: cell, region, AZ, shard. Har bo'linma mustaqil ishlaydi va boshqasining failure'idan himoyalangan. Dars: scale oshgan sari savol "qanday tezroq qilaman?" dan "qanday **izolyatsiya** qilaman?" ga o'tadi — failure'ni cheklab, uni butun tizimga tarqalishiga yo'l qo'ymaslik.

---

<a id="production-muhim"></a>
## Production'da nima muhim

Yuqoridagi arxitekturalar chiroyli diagrammalar, lekin real production ularni **ishlab turishidan** iborat. Bu bo'lim — arxitektura diagrammasida ko'rinmaydigan, lekin tizimni tirik ushlaydigan narsalar.

### Observability (kuzatuvchanlik)

"Tizim ishlayaptimi?" degan savolga javob bera olish. Uch ustun:

```text
  ┌─────────────┬──────────────────┬─────────────────────┐
  │  METRICS    │      LOGS         │       TRACES        │
  ├─────────────┼──────────────────┼─────────────────────┤
  │ raqamlar    │ hodisa yozuvlari  │ so'rovning butun     │
  │ (RPS, p99,  │ (nima bo'ldi,     │ yo'li (servis A→B→C, │
  │  error %,   │  qachon, qayerda) │  har qadamda qancha  │
  │  CPU)       │                   │  vaqt) — distributed │
  │ trend, alert│ qidiruv, debug    │ tracing              │
  └─────────────┴──────────────────┴─────────────────────┘
```

**💡 Tushuncha:** Microservice'da bitta so'rov o'nlab servisdan o'tadi. Bitta servisning log'i yetmaydi — **distributed tracing** (trace ID butun yo'l bo'ylab ko'chadi) kerak, aks holda "sekinlik qayerda?" savoliga javob topolmaysiz. Metrics **alert** beradi ("nimadir buzildi"), traces **sababini** topadi ("mana bu servis sekin").

### Deployment (CI/CD, canary)

```text
  Kod → CI (test, build) → CD (avtomatik deploy):

  CANARY DEPLOY (xavfsiz):
    Yangi versiyani AVVAL 1% trafikka ber → metrikalarni kuzat
       error/latency yaxshi? → 5% → 25% → 100% (bosqichma-bosqich)
       yomonlashdi?          → AVTOMATIK rollback (eski versiyaga qaytar)
```

- **CI/CD:** har commit avtomatik test qilinadi, build bo'ladi, deploy'ga tayyorlanadi — qo'lda emas.
- **Canary:** yangi versiya avval **kichik %** foydalanuvchiga chiqadi. Muammo bo'lsa, faqat shu % ta'sirlanadi va avtomatik rollback bo'ladi. (Blue-green — ikkita to'liq muhit, darrov almashtirish; feature flag — kodni deploy qilib, funksiyani alohida yoqish.)

### On-call va incident response

- **On-call:** navbatchi muhandis alert'ga javob beradi. Yaxshi alert — **actionable** (harakat talab qiladi) va **kam** (alert fatigue — ko'p soxta alert e'tiborni yo'qotadi).
- **Incident response:** buzilganda aniq jarayon — aniqlash (detect) → chekgach (mitigate, avval **to'xtat**, keyin sabab qidır) → tiklash → **blameless postmortem** (aybsiz tahlil: "kim aybdor emas, **jarayon** qayerda buzildi").

**⚠️ Ehtiyot bo'l:** Incident'da birinchi maqsad — **sababni topish emas, ta'sirni to'xtatish (mitigate)**. Ko'p yangi muhandis "nega buzildi?" ni qidirib vaqt yo'qotadi, foydalanuvchi esa azoblanaveradi. To'g'ri tartib: avval **rollback/failover** bilan qonni to'xtat, keyin sabab tahlili. Root cause postmortem'da, incident vaqtida emas.

### Capacity planning (sig'im rejalashtirish)

Kelajakdagi yukni oldindan hisoblab, resurs yetishini ta'minlash. **Load testing** (sun'iy yuk berib chegara topish), **auto-scaling** (yuk oshsa avtomatik server qo'shish), **headroom** (peak'dan zaxira qoldirish — 100% to'la ishlatmaslik). "Qora juma" (Black Friday) kabi voqealar oldindan rejalashtiriladi.

---

<a id="qa"></a>
## Umumiy savollar (Q&A)

### ❓ Nega ko'p kompaniya monolitdan boshlab, keyin microservice'ga o'tadi?

**✅ Javob:** Monolit **boshda tezroq** — bitta kodbaza, oddiy deploy, jamoa kichik. Microservice **erta** kiritilsa, uning narxi (tarmoq latency, distributed debugging, deploy murakkabligi, ma'lumot mosligi) foydasidan **oshadi**. Jamoa va tizim o'sgach (bir kodbazaga ko'p jamoa siqilib, deploy sekinlashADI), microservice mantiqiy bo'ladi — u **jamoa mustaqilligi** uchun kerak, "modanг uchun" emas. Uber, Twitter — hammasi monolitdan boshladi. Dars: **muammo paydo bo'lgach** ajrating, oldindan emas.

### ❓ Circuit breaker aniq nima qiladi va nega kerak?

**✅ Javob:** Downstream servis sekinlashsa/o'lsa, unga qo'ng'iroq **timeout** kutib, thread'lar band bo'lADI — bu yuqoriga tarqalib **cascading failure** keltiradi. Circuit breaker xatolarni sanaydi: chegaradan oshsa "ochiladi" (open) — endi qo'ng'iroq **darrov** fail bo'ladi (kutmaydi), fallback qaytariladi. Bir muddatdan keyin "yarim ochiq" (half-open) — bitta test qo'ng'iroq yuboradi, tiklansa yopadi. Netflix Hystrix buni mashhur qildi. Maqsad — **sekin servis butun tizimni bloklamasin**.

### ❓ Fan-out on write va fan-out on read qachon qaysi biri?

**✅ Javob:** **Fan-out on write** (pre-compute) — o'qish-og'ir, cheklangan follower (oddiy foydalanuvchi): o'qishni tez qiladi, yozish qimmatliroq. **Fan-out on read** — yozish-og'ir yoki juda ko'p follower (celebrity): yozish arzon, o'qish qimmat. Real tizimlar (Twitter) **hybrid** — ko'pchilikка write, celebrity'ga read. Qaror **read:write nisbati** va **fan-out kattaligi** bilan belgilanadi.

### ❓ Nega WhatsApp/Discord Erlang/Elixir tanladi?

**✅ Javob:** Messaging muammosi — **millionlab bir vaqtdagi, uzoq yashovchi ulanish**, har biri kam ish qiladi lekin doimo tirik. Erlang/BEAM aynan shunga qurilgan: yengil process'lar (OS thread emas, million'lab bo'la oladi), izolyatsiya ("bitta process crash bo'lsa boshqasi ta'sirlanmaydi"), past latency scheduler. Bu "C10M" (10 million ulanish) muammosiga tabiiy mos. Boshqa tilda bu ancha ko'p kod va server talab qiladi.

### ❓ Nega Netflix video'ni microservice'lardan o'tkazmaydi (CDN)?

**✅ Javob:** Video baytlari — trafikning ~99%+ hajmi. Uni backend microservice'lardan o'tkazish **ulkan** bandwidth, server, latency talab qilardi. Yechim: **control-plane** (kim, nima ko'rsatish — kichik so'rovlar, Zuul orqali) va **data-plane** (haqiqiy video — Open Connect CDN orqali, ISP ichida) ni ajratish. Video foydalanuvchiga **eng yaqin** appliance'dan oqadi. Dars: **katta data'ni control mantiqidan ajrat** — CDN aynan buning uchun.

### ❓ "Hot partition" nima va qanday oldini olinadi?

**✅ Javob:** Sharded/partition'langan store'da bitta partition **nomutanosib** ko'p yuk oladi (Discord'da million-xabarli kanal, umuman "hot key"). Natija — o'sha node zo'riqadi, latency oshadi, boshqalari bo'sh turadi. Oldini olish: partition kalitiga **qo'shimcha o'lchov** qo'shish (Discord: `channel_id` + vaqt bucketi), yoki tabiiy taqsimlangan kalit tanlash, yoki hot key'ni alohida ishlash. Dars: partition kalitini **eng katta/eng issiq holat** bo'yicha tanlang.

### ❓ Chaos Engineering nega foydali, u xavfli emasmi?

**✅ Javob:** Falsafa: "failure baribir bo'ladi — uni **nazorat ostida**, ish vaqtida, kuzatuv bilan mashq qilgan afzal, tuni bilan ogohsiz kutgandan". Chaos Monkey production'da instance o'ldiradi — bu tizim **haqiqatan** chidamli ekanini isbotlaydi (diagramma emas, amaliyot). Xavfni cheklash: blast radius kichik (bitta instance), ish soatida (jamoa hozir), avtomatik to'xtatish. Dars: chidamlilikni **taxmin qilmang — sinang**.

### ❓ Idempotency real tizimlarda qayerda hal qiluvchi?

**✅ Javob:** Tarmoq **retry** muqarrar (timeout, uzilish). Retry'da operatsiya ikki marta bajarilishi mumkin — to'lov ikki marta, xabar ikki marta yetkazilishi. **Idempotency key** (client bergan UUID) bilan server bir xil key'ni **bir marta** ishlaydi. Bu "**at-least-once delivery + idempotency = exactly-once effect**" ni beradi. Uber (trip event), to'lov, webhook, WhatsApp (dedup) — hammasida markaziy. Exactly-once delivery deyarli imkonsiz; exactly-once **effect** — erishsa bo'ladigan maqsad.

### ❓ Nega "boring technology" (Instagram, Postgres) ko'pincha to'g'ri tanlov?

**✅ Javob:** Har texnologiya **operatsion narx** qo'shadi: o'rganish, debug, monitoring, on-call, "kim biladi?". Yangi/ekzotik vosita ko'pincha noma'lum failure rejimlariga ega. Sinalgan vosita (Postgres, Memcached) — hujjatlangan, jamoa biladi, muammolari ma'lum. Kichik jamoa uchun bu hal qiluvchi. Innovatsiya "budjeti"ni **biznes ustunligi beradigan** bitta joyga sarflang, hamma joyga emas. Instagram, WhatsApp — ikkalasi ham radikal soddalik bilan yutdi.

### ❓ Blameless postmortem nima va nega "aybsiz"?

**✅ Javob:** Incident'dan keyin nima bo'lganini tahlil qilish — lekin **shaxsni ayblamasdan**. Sabab: agar odamlar ayblanishdan qo'rqsa, ular xatoni **yashiradi**, va tizim **o'rganmaydi**. Aybsiz yondashuvda savol "kim aybdor?" emas, "**qaysi jarayon/himoya** bu xatoni oldini olmadi?" — chunki inson doim xato qiladi, tizim uni **ushlab** qolishi kerak edi. Bu psixologik xavfsizlik yaratadi va haqiqiy sabablar ochiladi. Google/Amazon madaniyatining o'zagi.

### ❓ Cell-based architecture microservice'dan qanday farq qiladi?

**✅ Javob:** Microservice — tizimni **funksiya** bo'yicha bo'lish (auth servisi, payment servisi...). Cell — tizimni **foydalanuvchi/yuk** bo'yicha bo'lish: har cell **to'liq stack**ning nusxasi, foydalanuvchilarning bir qismiga xizmat qiladi. Ular birga ishlaydi: har cell ichida microservice'lar bor. Cell'ning maqsadi — **blast radius**: bitta cell buzilsa, faqat undagi foydalanuvchilar ta'sirlanadi. Microservice — kod tashkiloti; cell — failure izolyatsiyasi.

### ❓ Surge pricing texnik yechimmi yoki biznes?

**✅ Javob:** **Ikkalasi** — va bu darsning o'zi. Uber'da talab taklifdan oshsa (kam haydovchi, ko'p yo'lovchi), tizim **iqtisodiy signal** (narx oshadi) bilan muvozanatni tiklaydi: yuqori narx ba'zi yo'lovchini kutishга/voz kechishga undaydi (talab kamayadi) va haydovchini shu hududga jalb qiladi (taklif oshadi). Texnik tomondan bu per-hexagon real-time supply/demand hisobi. Dars: ba'zi "scaling" muammolarini **faqat** ko'proq server bilan emas, **tizim xatti-harakatini boshqarish** (narx, rate limit, prioritet) bilan ham hal qilinadi.

---

<a id="masalalar"></a>
## Masalalar

> Yechimlar: [solutions/system-design/07-production-examples.md](../solutions/system-design/07-production-examples.md)

Quyidagi masalalarni **real production nuqtai nazaridan** loyihalashtiring. Har biriga: masshtab taxmini, high-level arxitektura (ASCII diagramma), asosiy texnik qarorlar (nega aynan shu), ishlatiladigan pattern/texnologiya, va **o'rganiladigan dars**. Yuqoridagi misollarga (Netflix, Uber, WhatsApp, Twitter, Instagram, Discord) tayaning.

1. **O'z "Netflix"ingizni loyihalang.** Video streaming platformasi: yuklash/encoding pipeline (bir video → o'nlab bitrate/codec), CDN bilan yetkazish, resilience (circuit breaker, fallback). Control-plane va data-plane'ni qanday ajratasiz? Encoding'ni nega asinxron qilasiz? Chaos testing'ni qayerga qo'yasiz?

2. **Real-time ride-matching (Uber-vari).** Shaharда haydovchi-yo'lovchi moslashtirish: geo-index (H3-vari hexagon grid), location update oqimi (har haydovchi ~4s'da), "eng yaqin haydovchi" so'rovi, event-driven trip lifecycle. Surge pricing'ni per-hudud qanday hisoblaysiz? Hot hudud (aeroport, konsert) muammosini qanday hal qilasiz?

3. **Messaging tizimi minimal server bilan (WhatsApp-vari).** Store-and-forward xabar yetkazish: doimiy ulanish, onlayn/oflayn routing, delivery ack (✓ ✓✓), per-conversation ordering, E2E encryption. Nega xabarni serverda saqlamaysiz? Presence (onlayn holati) fan-out muammosini qanday cheklaysiz?

4. **Home timeline (Twitter-vari) hybrid fan-out bilan.** Fan-out on write vs read tahlili, celebrity muammosi uchun **hybrid** yechim, timeline cache (ID ro'yxati + hydrate), ML ranking. Qaysi foydalanuvchini "celebrity" deb belgilaysiz (chegara)? Merge + rank'ni qanday past latency qilasiz?

5. **Foto/kontent platforma kichik jamoa bilan (Instagram-vari).** Object storage + CDN'da rasm, metadata Postgres'da, **logical sharding**, shard-encoded ID (Snowflake-vari). Nega "boring technology"? Logical shard'ni fizik serverga ko'chirishni qanday og'riqsiz qilasiz? ID'dan shardni qanday hisoblaysiz?

6. **Real-time chat trillion-xabar scale'da (Discord-vari).** WebSocket gateway (millionlab ulanish), xabar store (Cassandra/ScyllaDB), partition strategiyasi. Partition kalitini qanday tanlaysiz **hot partition**'ni oldini olib? Katta kanal (million xabarli) muammosini qanday hal qilasiz? Nega JVM GC latency manbai bo'lishi mumkin?

7. **Cell-based arxitektura bilan global servis.** Foydalanuvchilarni cell'larga bo'lish (har cell to'liq stack), blast radius izolyatsiyasi, cell-by-cell deployment. Foydalanuvchini qaysi cell'ga qanday yo'naltirasiz (routing)? Bitta cell to'lganda nima qilasiz? Deployment'ni cell bo'yicha qanday bosqichlaysiz?

8. **Production observability va deployment pipeline.** Bitta so'rov o'nlab microservice'dan o'tadigan tizim uchun observability (metrics/logs/traces, distributed tracing) va deployment (CI/CD, canary, avtomatik rollback) loyihalang. "Sekinlik qayerda?" savoliga qanday javob berasiz? Canary metrikalari yomonlashsa avtomatik rollback'ni qanday qurasiz?

9. **On-call va incident response tizimi.** Alerting (actionable, kam soxta alert), on-call rotatsiya, incident jarayoni (detect → mitigate → tiklash → blameless postmortem). Alert fatigue'ni qanday oldini olasiz? Incident'da "avval to'xtat, keyin sabab qidir" ni qanday jarayonga aylantirasiz?

10. **Capacity planning "Qora juma" (Black Friday) uchun.** Peak yuk (10x oddiy) kutilayotgan e-commerce tizimi: load testing, auto-scaling, headroom, degradation rejasi. Chegarani qanday topasiz (load test)? Yuk chegaradan oshsa, qaysi funksiyalarni birinchi "o'chirasiz" (graceful degradation)?

---

← [System Design bo'limiga qaytish](./README.md)
