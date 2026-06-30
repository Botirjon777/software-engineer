# Latency, Throughput va Performance — Yechimlar

Bu yerda [`06-latency-performance.md`](../../networking/06-latency-performance.md) bo'limidagi masalalarning to'liq yechimlari keltirilgan. Avval o'zingiz urinib ko'ring, keyin solishtiring.

## Mundarija

- [1-masala: Latency yoki throughput](#1-masala)
- [2-masala: Raqamlar intuitsiyasi](#2-masala)
- [3-masala: Latency dekompozitsiyasi](#3-masala)
- [4-masala: Percentile hisoblash](#4-masala)
- [5-masala: Tail latency amplifikatsiyasi](#5-masala)
- [6-masala: BDP va TCP window](#6-masala)
- [7-masala: Little's Law amaliyoti](#7-masala)
- [8-masala: Optimizatsiya tartibi](#8-masala)
- [9-masala: Bottleneck diagnostikasi](#9-masala)
- [10-masala: Performance budget loyihasi](#10-masala)

---

## 1-masala

**Latency yoki throughput?**

| Muammo | Tur | Yechim yo'nalishi |
|--------|-----|-------------------|
| (a) sahifa 5s ochiladi | **Latency** | Yo'lni qisqartirish: kesh, CDN, round triplarni kamaytirish, DB so'rovini tezlashtirish. |
| (b) 50 req/s, talab 500 | **Throughput** | Kengaytirish: ko'proq server (horizontal scaling), parallellik, batching, connection pool oshirish. |
| (c) video uzilib o'ynaydi | **Throughput (bandwidth) + jitter** | Yetarli throughput/bandwidth, adaptive bitrate, buffering, CDN. (Latency ham muhim, lekin uzilish odatda bandwidth/jitter.) |
| (d) batch 6 soat ishlaydi | **Throughput** | Parallellashtirish, batch hajmini optimizatsiya, ortiqcha ish kamaytirish. (Bir batch'ning umumiy davomiyligi — throughput muammosi.) |

**Printsip:** "bitta narsa sekin" → latency; "ko'p narsa sig'maydi" → throughput.

---

## 2-masala

**Raqamlar intuitsiyasi.**

Tezdan sekinga tartib (taxminiy):

```text
1. L1 cache                ~1 ns          (asos)
2. RAM read                ~100 ns        L1'dan ~100x sekin
3. SSD read                ~16 µs         RAM'dan ~160x sekin
4. Datacenter round trip   ~500 µs        SSD'dan ~30x sekin
5. Disk (HDD) seek         ~10 ms         datacenter RT'dan ~20x sekin
6. Qit'alararo round trip  ~150 ms        disk seek'dan ~15x sekin
```

**Nima uchun "diskdan o'qib RAM'da keshlash" deyarli har doim foydali:** RAM diskdan ~100,000 barobar (va tarmoqdan ham shunga yaqin) tezroq. Bir marta diskdan/originddan o'qib RAM'ga (yoki Redis'ga) keshlasangiz, keyingi barcha o'qishlar ~100,000x tez bo'ladi. Ko'p so'raladigan (hot) ma'lumot uchun bu zarba — shuning uchun deyarli har bir performance-critical tizimda keshlash qatlami bor. Faqat ma'lumot bir marta o'qilsa yoki tez eskirsa keshlash foydasiz.

---

## 3-masala

**Latency dekompozitsiyasi (2 MB, 20 Mbps, RTT 120ms, qit'alararo).**

```text
1. Propagation:   RTT/2 ≈ 60 ms har tomon (yorug'lik tezligi, masofa)
2. Transmission:  2 MB = 16 Mbit;  16 / 20 Mbps = 0.8 s = 800 ms   <- USTUN!
3. Processing:    server-ga bog'liq, taxminan ~50-100 ms
4. Queuing:       yuk past bo'lsa ~0; cho'qqida o'sadi
```

**Ustun komponent:** **Transmission delay (~800ms)** — chunki kanal tor (20 Mbps) va fayl katta (2 MB). Bu propagation (~60ms)dan ancha katta.

**Optimizatsiyalar (har biri uchun):**

- **Transmission:** Compression (gzip/brotli) — JSON ~70-90% siqiladi, 2 MB → ~300 KB, transmission ~120ms'gacha tushadi. Eng katta yutuq shu.
- **Propagation:** Foydalanuvchiga yaqin region/CDN/edge — masofa qisqaradi.
- **Processing:** Server kodi/DB optimizatsiyasi, kesh.
- **Queuing:** Yetarli quvvat/auto-scaling, navbatni qisqa tutish.

**Xulosa:** Bu holatda eng kuchli optimizatsiya — **compression**, chunki transmission ustun.

---

## 4-masala

**Percentile hisoblash.**

Saralangan: `10, 10, 11, 11, 12, 12, 13, 13, 14, 350` (ms).

```text
Average = (10+12+11+13+10+14+12+11+350+13)/10 = 456/10 = 45.6 ms
p50 (median) ≈ 12 ms       (5-6-o'rin orasi)
p90 ≈ 14 ms                (9-o'rin atrofida — 350'dan oldingi)
p99 ≈ ~350 ms              (eng yuqori 1% -> 350'ga yaqin)
```

**Tahlil:** Average (45.6ms) yolg'on rasm beradi — u 350ms outlier tomonidan "tortilgan". Aslida so'rovlarning 90% 14ms va undan tez. Median (12ms) tipik tajribani to'g'ri ko'rsatadi.

**SLA'ga nima qo'yiladi:** **p99** (yoki p95) qo'yiladi, average emas. Sababi: foydalanuvchilar eng yomon holatlarni sezadi, va average outlier'larni yashiradi. Masalan "p99 < 100ms" — bu 350ms holatni ushlaydi va muammo borligini ko'rsatadi; average esa "45ms, yaxshi" deb aldaydi.

**350ms sababi:** Outlier — ehtimol GC pause, cache miss (bu so'rov keshda topilmay diskka/DB'ga bordi), queuing (qisqa cho'qqi), yoki cold start. Bitta sekin so'rov — tail latency'ning klassik belgisi.

---

## 5-masala

**Tail latency amplifikatsiyasi (20 mikroservis, har biri p99=1% sekin).**

Bir so'rov **tez** bo'lish ehtimoli = 0.99 (99%). Sahifa 20 ta mustaqil so'rov qiladi, hammasi tez bo'lish ehtimoli:

```text
P(hammasi tez) = 0.99^20 ≈ 0.818  (81.8%)
P(kamida bittasi sekin) = 1 - 0.818 ≈ 0.182  ≈ 18%
```

**Natija:** Har bir mikroservis "atigi 1%" sekin bo'lsa-da, sahifaning **~18%** kamida bitta sekin so'rovga duch keladi. Fan-out qancha katta bo'lsa, shuncha yomon (100 ta xizmatda: `1 - 0.99^100 ≈ 63%`). Bu tail latency amplifikatsiyasi.

**Kamaytirish strategiyalari:**

1. **Hedged requests:** Sekin javob kelmasa, bir oz kutib ikkinchi nusxasini boshqa replikaga yuborish; qaysi tez kelsa o'shani olish. Tail keskin kamayadi.
2. **Har bir xizmatning p99'ini yaxshilash:** GC tuning, cache hit ratio oshirish, queuingni kamaytirish — manbada tail'ni qisqartirish.
3. **Parallellashtirish + timeout/fallback:** Mustaqil so'rovlarni parallel yuborish (ketma-ket emas) va sekin xizmat uchun timeout + default/degraded javob (graceful degradation).
4. **Fan-out kamaytirish:** Aggregation/BFF qatlami bilan kerakli ma'lumotni oldindan birlashtirish, so'rovlar sonini kamaytirish.

---

## 6-masala

**BDP va TCP window (1 Gbps, RTT 80ms).**

```text
BDP = bandwidth × RTT
    = 1 Gbps × 0.08 s
    = 1,000,000,000 bit/s × 0.08 s
    = 80,000,000 bit = 80 Mbit = 10 MB
```

**256 KB window yetarlimi?** Yo'q. BDP = 10 MB, lekin window atigi 256 KB. Yuboruvchi 256 KB yuborgach, ACK kutishga majbur — kanalning katta qismi bo'sh qoladi.

```text
Foydalanish ≈ window / BDP = 256 KB / 10 MB ≈ 2.5%
=> kanalning atigi ~2.5% ishlatiladi!
```

**Qancha window kerak:** Kanalni to'liq band qilish uchun window ≥ BDP = **10 MB** bo'lishi kerak.

**"Long fat network" muammosi:** Bu aynan shu — yuqori bandwidth (fat) + yuqori RTT (long) bo'lgan kanal katta BDP'ga ega. Standart TCP window (64 KB, scaling'siz) bunday kanalni to'ldira olmaydi. Yechim — **TCP window scaling** (RFC 1323) bilan window'ni MB darajasiga oshirish. Aks holda bandwidth qimmat, lekin throughput past qoladi.

---

## 7-masala

**Little's Law amaliyoti (2000 req/s, 50ms latency).**

```text
L = λ × W = 2000 req/s × 0.05 s = 100
```

**Tizimda bir vaqtda 100 ta so'rov "uchib" turadi.**

**Connection pool o'lchami:** Kamida **100** (concurrency'ni qoplash uchun). Amalda biroz zaxira bilan ~120-150 qo'yiladi (cho'qqilar, jitter uchun). Juda kichik pool — so'rovlar pool kutishida navbatda qoladi (queuing delay oshadi); juda katta pool — resurs isrofi va downstream'ni (DB) ortiqcha yuklash xavfi.

**Agar latency 200ms'gacha oshsa (throughput sobit):**

```text
L = 2000 × 0.2 = 400
```

Concurrency 100 → 400 ga oshadi. Ya'ni o'sha throughputni saqlash uchun endi **400 ta** parallel ishlovchi/ulanish kerak. Agar pool 100'da qolsa, throughput 500 req/s'ga tushadi (`100/0.2`) — tizim talabni eplay olmaydi, queuing portlaydi. Xulosa: latency oshganda, throughputni saqlash ko'proq resurs (concurrency) talab qiladi — shuning uchun latency'ni past ushlash throughput uchun ham muhim.

---

## 8-masala

**Optimizatsiya tartibi (30 API call, yangi HTTPS har safar, siqilmagan JSON, kesh yo'q).**

Ta'sir/xarajat bo'yicha tartiblangan:

| # | Optimizatsiya | Ta'sir | Xarajat | Kutilayotgan natija |
|---|---------------|--------|---------|---------------------|
| 1 | **Caching** (client/CDN) | Juda yuqori | Past | Ko'p so'rovni butunlay yo'qotadi — eng arzon yutuq. |
| 2 | **Batching / aggregation** (30 → 1-3 call) | Juda yuqori | O'rta | Round trip soni 30 → bir nechta. Eng katta latency manbai. |
| 3 | **Connection reuse / keep-alive + HTTP/2** | Yuqori | Past | Har so'rovdagi TCP+TLS handshake (2-3 RTT) yo'qoladi; HTTP/2 multiplexing bilan parallel. |
| 4 | **Compression (gzip/brotli)** | O'rta | Past | JSON ~70-90% siqiladi → transmission delay ↓. |
| 5 | **Prefetching** | O'rta | O'rta | Keyingi sahifa ma'lumotini oldindan yuklash (UX yaxshilanadi). |

**Tartib mantig'i:** Avval **so'rovni butunlay yo'qot** (cache), keyin **round trip sonini kamaytir** (batch + keep-alive/HTTP/2), keyin **har so'rovni yengillashtir** (compression). 30 ta alohida HTTPS ulanish = 30 × (TCP+TLS handshake + transfer) — bu eng katta jarima, shuning uchun batching va connection reuse eng katta ta'sir beradi.

**Taxminiy natija:** 30 ta yangi HTTPS call (har biri ~4 RTT) → bir nechta keep-alive call (1 RTT) + compression — latency bir necha barobar kamayadi.

---

## 9-masala

**Bottleneck diagnostikasi (yuk ortganda latency eksponensial↑, CPU 40%, vaqt "DB query kutish"da).**

**Belgilarning ma'nosi:** CPU past (40%) lekin latency keskin o'sadi va vaqt DB kutishda — bu **CPU bottleneck emas**, balki **resurs to'yinishi (saturation) DB tomonida yoki connection pool tugashi**. Eksponensial o'sish — queuing delay'ning klassik belgisi (Little's Law / navbat nazariyasi: utilization 100%'ga yaqinlashganda kutish vaqti portlaydi).

**Ehtimoliy sabablar:**

1. **DB connection pool tugagan:** So'rovlar bo'sh ulanish kutib navbatda turadi. Yuk ortganda pool to'lib, queuing eksponensial o'sadi.
2. **DB'da sekin so'rovlar:** Yo'q indeks, N+1 query, yoki lock contention — har so'rov uzoq, pool tez to'ladi.
3. **DB o'zi saturated:** DB CPU/IO 100%, lock waiting, ya'ni bottleneck app'da emas, DB'da.

**Tasdiqlash/yechish qadamlari:**

1. **Pool metrikalarini ko'rish:** Active vs idle connections, "wait time for connection". Pool doim to'la bo'lsa — sabab tasdiqlandi.
2. **DB monitoring:** DB CPU/IO, active queries, lock waits, `pg_stat_activity`. DB saturated'mi yoki app kutyaptimi?
3. **Slow query log + EXPLAIN:** Sekin so'rovlarni topib, indeks/N+1 muammosini aniqlash.
4. **Yechimlar:**
   - Indeks qo'shish, N+1 ni `JOIN`/batch bilan tuzatish, sekin query optimizatsiyasi.
   - Connection pool to'g'ri o'lchamlash (Little's Law bilan), lekin DB sig'imidan oshirmaslik.
   - Read replica'lar bilan o'qish yukini bo'lish.
   - Hot data uchun caching (DB'ga borishni kamaytirish).

**⚠️ Eslatma:** Bu yerda ko'proq app server qo'shish **yordam bermaydi** — bottleneck DB. Ko'proq app server faqat DB'ga ko'proq zarba beradi.

---

## 10-masala

**Performance budget loyihasi (mahsulot sahifasi, 800ms maqsad).**

```text
Performance budget (jami 800 ms):
─────────────────────────────────────────────
  DNS + TLS handshake             80 ms
  CDN / static aktivlar (parallel) 100 ms
  Backend API javob              350 ms
    ├─ DB so'rovi (katalog)       120 ms
    ├─ biznes logika               60 ms
    ├─ tashqi to'lov xizmati      150 ms  <- ENG XAVFLI
    └─ serialization/overhead      20 ms
  Tarmoq transfer (gzip)          70 ms
  Frontend render                200 ms
─────────────────────────────────────────────
  JAMI                           800 ms
```

**Eng xavfli qism — tashqi to'lov xizmati (150ms):** Sababi: bu **sizning nazoratingizdan tashqarida**. Tashqi to'lov gateway'i sekinlashsa yoki tail latency bersa (p99 yuqori), butun budget buziladi. Shuningdek u tarmoq orqali — propagation + ularning processing'i sizga bog'liq emas.

**Xavfni kamaytirish:**

- **Tashqi chaqiruvni kritik yo'ldan chiqarish:** Mahsulot sahifasini ko'rsatish uchun to'lov gateway'i kerakmi? Agar yo'q bo'lsa, uni asinxron qilish yoki faqat checkout'da chaqirish.
- **Timeout + fallback:** Tashqi xizmat sekin bo'lsa, timeout qo'yib, degraded/cached javob ko'rsatish (graceful degradation).
- **Caching:** Tashqi xizmat javobini (agar o'zgarmas bo'lsa) keshlash.
- **Hedging/circuit breaker:** Doimiy sekinlikda circuit breaker bilan xizmatni vaqtincha chetlab o'tish.

**Umumiy printsip:** Nazoratdan tashqaridagi va tail latency'li (tashqi xizmat) komponentlar — budget'ning eng zaif joyi. Ularni kritik yo'ldan olib tashlash yoki himoyalash (timeout, fallback, cache) kerak.

---

← [Networking bo'limiga qaytish](../../networking/README.md)
