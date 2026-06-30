# Latency, Throughput va Performance

Tizim "tez" deganda odamlar ko'pincha bir narsani nazarda tutadi, lekin aslida ikki butunlay boshqa o'lchov bor: **latency** (bir so'rov qancha vaqt oladi) va **throughput** (vaqt birligida nechta so'rov bajariladi). Senior darajadagi muhandis bu ikkisini aralashtirmaydi, raqamlarning kattalik tartibini (orders of magnitude) yodda tutadi va bottleneck'larni o'lchov asosida topa oladi. Bu bo'limda latency manbalari, "har bir dasturchi bilishi kerak bo'lgan raqamlar", tail latency va tarmoq optimizatsiyasini ko'rib chiqamiz.

**💡 Tushuncha:** Performance haqida gapirganda har doim **qaysi metrika** haqida gapirayotganingni aniqlashtir. "Sekin" — bu latency yuqorimi yoki throughput pastmi? Bu ikki muammoning yechimi butunlay boshqacha.

## Mundarija

- [Latency va Throughput](#latency-va-throughput)
- [Bandwidth va RTT](#bandwidth-va-rtt)
- [Har bir dasturchi bilishi kerak bo'lgan raqamlar](#har-bir-dasturchi-bilishi-kerak-bolgan-raqamlar)
- [Latency manbalari](#latency-manbalari)
- [Tail Latency (p50/p95/p99)](#tail-latency-p50p95p99)
- [Bandwidth-Delay Product](#bandwidth-delay-product)
- [Tarmoq optimizatsiyasi](#tarmoq-optimizatsiyasi)
- [Little's Law](#littles-law)
- [Performance o'lchash va budget](#performance-olchash-va-budget)
- [Bottleneck topish](#bottleneck-topish)
- [Savol-Javoblar](#savol-javoblar)
- [Masalalar](#masalalar)

---

## Latency va Throughput

**Latency** — bitta operatsiya boshidan oxirigacha o'tgan vaqt. O'lchov birligi: vaqt (ms, µs). "So'rov qancha tez javob qaytaradi?"

**Throughput** — vaqt birligida bajarilgan operatsiyalar soni. O'lchov birligi: soni/vaqt (requests/sec, MB/s). "Sekundiga nechta so'rov bajarila oladi?"

**Analogiya (suv trubasi):**

```text
        ┌──────────────────────────────────┐
suv ───>│                                  │───> suv
        └──────────────────────────────────┘
   LATENCY  = bir tomchi trubadan o'tish vaqti (uzunligi)
   THROUGHPUT = sekundiga necha litr oqib o'tishi (kengligi)
```

Truba **uzun** bo'lsa latency yuqori (tomchi uzoq yuradi), lekin **keng** bo'lsa throughput baland (ko'p suv sig'adi). Ular bir-biriga bog'liq emas — uzun va keng truba ham bo'lishi mumkin.

**Yana bir analogiya (magistral yo'l):** Latency — bitta mashina A'dan B'ga yetib borish vaqti. Throughput — soatiga nechta mashina o'tishi. Yo'lni kengaytirsangiz (ko'p qatorli) throughput oshadi, lekin har bir mashinaning yetib borish vaqti (latency) o'zgarmaydi.

```text
PAST latency, YUQORI throughput  -> ideal
YUQORI latency, YUQORI throughput -> batch tizimlar (uzoq, lekin ko'p)
PAST latency, PAST throughput    -> tez, lekin kam sig'imli
```

**⚠️ Ehtiyot bo'l:** Throughput'ni oshirish (ko'proq server, parallellik) latency'ni kamaytirmaydi. Latency'ni kamaytirish uchun yo'l qisqartiriladi (CDN, kesh), throughput uchun yo'l kengaytiriladi (parallellik, batching). Intervyuda bu farqni aralashtirish — keng tarqalgan xato.

---

## Bandwidth va RTT

**Bandwidth** — tarmoq kanalining maksimal nazariy o'tkazuvchanlik qobiliyati (masalan, 100 Mbps). Bu "trubaning eng katta kengligi". Throughput esa amalda erishilgan tezlik — u har doim bandwidth'dan kichik yoki teng.

**RTT (Round-Trip Time)** — paketning manzilga borib-qaytish vaqti. Ping shu narsani o'lchaydi. Ko'p protokollar (TCP handshake, TLS) bir nechta round trip talab qiladi, shuning uchun RTT to'g'ridan-to'g'ri foydalanuvchi seziladigan latency'ga ta'sir qiladi.

```text
Client ──── so'rov ────> Server
       <─── javob ─────
       └──── RTT ──────┘  (borib-qaytish vaqti)

RTT misollari:
  bir shahar ichida:     ~1-5 ms
  bir mamlakat:          ~10-40 ms
  qit'alararo:           ~100-200 ms
```

**💡 Tushuncha:** Bandwidth — "qancha keng", latency/RTT — "qancha uzoq". Bandwidth'ni pul bilan oshirish oson, lekin latency'ni fizika (yorug'lik tezligi) cheklaydi — Toshkentdan Nyu-Yorkga signal yorug'lik tezligida ham ~40 ms ketadi, undan tezroq mumkin emas.

---

## Har bir dasturchi bilishi kerak bo'lgan raqamlar

Bu mashhur "Latency Numbers Every Programmer Should Know" jadvali (Jeff Dean). Aniq raqamlardan ko'ra **kattalik tartibi** (nisbatlar) muhim:

```text
Operatsiya                              Taxminiy vaqt      Nisbat
─────────────────────────────────────────────────────────────────
L1 cache reference                      ~1 ns              1x
L2 cache reference                      ~4 ns              4x
RAM (main memory) reference             ~100 ns            100x
SSD random read                         ~16 µs             16,000x
Datacenter ichida round trip            ~500 µs            500,000x
HDD (disk) seek                         ~10 ms             10,000,000x
Internet: bir qit'a ichida              ~50 ms             50,000,000x
Internet: qit'alararo (round trip)      ~150 ms            150,000,000x
─────────────────────────────────────────────────────────────────
1 ns  = 1/1,000,000,000 sekund
1 µs  = 1,000 ns
1 ms  = 1,000,000 ns = 1,000 µs
```

**Intuitiv miqyos (agar L1 cache = 1 sekund bo'lsa):**

```text
L1 cache    = 1 sekund
RAM         = ~2 daqiqa
SSD read    = ~4.5 soat
Disk seek   = ~4 oy
Datacenter RT = ~6 kun
Internet RT = ~5 yil  (qit'alararo)
```

**💡 Tushuncha:** Asosiy xulosa — **memory (ns) → SSD (µs) → disk/network (ms)** har qadamda ~1000 barobar sekinlashadi. Shuning uchun: keshni RAM'da ushlash, disk/tarmoqqa borishni kamaytirish, round trip'larni qisqartirish — performance'ning eng kuchli vositalari.

**⚠️ Ehtiyot bo'l:** Aniq nanosekundlarni yodlash shart emas — intervyuda muhimi nisbatlarni bilish: "RAM diskdan ~100,000 barobar tez", "datacenter ichidagi tarmoq qit'alararodan ~100 barobar tez". Bu "buni keshlash arziydimi?" degan qarorni asoslaydi.

---

## Latency manbalari

Bir so'rovning umumiy latency'si to'rt komponentdan iborat:

```text
Total latency = Propagation + Transmission + Processing + Queuing
```

1. **Propagation delay** — signalning fizik masofani bosib o'tish vaqti. Yorug'lik/elektr tezligi bilan cheklangan. Masofaga bog'liq, o'zgartirib bo'lmaydi (faqat yaqinroq server — CDN bilan kamaytiriladi).

2. **Transmission delay** — ma'lumotni kanalga "yuklash" vaqti. `data_size / bandwidth`. Katta fayl + tor kanal = yuqori transmission delay. Bandwidth oshirish yoki compression bilan kamayadi.

3. **Processing delay** — server (yoki router) so'rovni qayta ishlash vaqti. Kod optimizatsiyasi, tezroq DB so'rovi, kesh bilan kamayadi.

4. **Queuing delay** — so'rov navbatda kutish vaqti. Tizim yuklanganda o'sadi. Bu eng "xavfli" komponent — yuk ortgani sayin keskin (eksponensial) o'sadi.

```text
[Client] ──propagation──> [kanal] ──transmission──> [Server queue]
                                                      │
                                                   queuing
                                                      ▼
                                                  [processing] ──> javob
```

**💡 Tushuncha:** Latency'ni optimizatsiya qilishda avval *qaysi komponent* katta ulushga ega ekanini aniqla. Qit'alararo so'rovda propagation ustun (CDN kerak), katta fayl yuborishda transmission (compression), yuklangan serverda queuing (ko'proq quvvat/parallellik).

---

## Tail Latency (p50/p95/p99)

O'rtacha (average) latency yolg'on gapiradi. Foydalanuvchilar **eng yomon** holatlarni sezadi. Shuning uchun **percentile** (foizlik) ishlatiladi:

- **p50 (median)** — so'rovlarning 50% shu vaqtdan tez. "Tipik" tajriba.
- **p95** — 95% shu vaqtdan tez (ya'ni 5% sekinroq).
- **p99** — 99% shu vaqtdan tez (1% sekinroq). Bu **tail latency**.

```text
So'rovlar taqsimoti:

soni
 │   ▆▆▆
 │  ▆▆▆▆▆
 │ ▆▆▆▆▆▆▆▆
 │▆▆▆▆▆▆▆▆▆▆▆▆▆___________▆__
 └──────────────────────────────> latency
   p50      p95   p99      (tail — uzun "dum")
   20ms    80ms  300ms
```

**Nega tail latency muhim:** Bitta sahifa ko'p so'rovdan iborat. Agar har bir so'rovning p99 = 1% sekin bo'lsa va sahifa 100 ta so'rov qilsa, foydalanuvchining sahifasida deyarli **doim** kamida bitta sekin so'rov bo'ladi. Microservice'larda bu kuchayadi: 10 ta xizmatni chaqirsangiz, umumiy latency eng sekin xizmat bilan belgilanadi.

**⚠️ Ehtiyot bo'l:** "O'rtacha latency 20ms" deyish — yolg'on tasalli. p99 = 2 sekund bo'lsa, foydalanuvchilarning 1% (millionlab so'rovda — minglab odam) yomon tajriba oladi. SLO/SLA'lar deyarli har doim percentile bilan beriladi (masalan, "p99 < 200ms"), o'rtacha bilan emas.

```text
Sabablari (tail latency):
  - GC (garbage collection) pauzalari
  - cache miss (kesh topilmadi -> diskka/tarmoqqa)
  - queuing (yuk cho'qqilarida navbat)
  - "noisy neighbor" (umumiy resursni boshqa yuk band qildi)
  - ret, slow disk, network jitter
```

---

## Bandwidth-Delay Product

**Bandwidth-Delay Product (BDP)** — bir vaqtning o'zida "yo'lda" (in-flight) bo'lishi mumkin bo'lgan ma'lumot miqdori:

```text
BDP = bandwidth × RTT
```

Bu — "trubadagi suv hajmi": uzun (yuqori RTT) va keng (yuqori bandwidth) trubada bir vaqtda ko'p ma'lumot bo'ladi.

**Nega muhim:** TCP "window" (bir vaqtda tasdiqsiz yuborilishi mumkin bo'lgan ma'lumot) BDP'dan kichik bo'lsa, kanal **to'liq ishlatilmaydi** — yuboruvchi har safar ACK kutib turishga majbur, bandwidth bo'sh qoladi.

```text
Misol: bandwidth = 100 Mbps, RTT = 100 ms
BDP = 100 Mbps × 0.1 s = 10 Mbit = 1.25 MB

Agar TCP window < 1.25 MB bo'lsa,
kanal to'liq band bo'lmaydi -> throughput pasayadi.
```

**💡 Tushuncha:** "Long fat network" (yuqori bandwidth + yuqori RTT, masalan sun'iy yo'ldosh yoki qit'alararo) muammosi shu — katta BDP uchun katta TCP window kerak. Shuning uchun OS'larda window scaling sozlamasi bor.

---

## Tarmoq optimizatsiyasi

Tarmoq orqali keladigan latency'ni kamaytirish usullari:

**1. Keep-alive / connection reuse** — TCP+TLS ulanishini har so'rovda qayta ochmaslik. Bitta ulanishni qayta ishlatish handshake round trip'larini tejaydi.

```text
Yangi ulanish:  TCP(1 RTT) + TLS(1-2 RTT) + so'rov(1 RTT) = 3-4 RTT
Keep-alive:     so'rov(1 RTT)                              = 1 RTT
```

**2. HTTP/2 multiplexing** — bitta ulanish ustidan ko'p so'rovni parallel yuborish (head-of-line blocking kamayadi). HTTP/1.1 da bir ulanishda navbat bilan; HTTP/2 da aralash.

**3. Compression (gzip/brotli)** — javob tanasini siqib, transmission delay'ni kamaytirish. Matn (JSON, HTML, JS) yaxshi siqiladi. Brotli gzip'dan zichroq, lekin sekinroq kodlaydi.

**4. Caching** — javobni qayta hisoblamay/qayta tortmay, keshdan berish (brauzer kesh, CDN, server-side cache). Eng arzon optimizatsiya — umuman so'rov yubormaslik.

**5. CDN** — kontentni foydalanuvchiga yaqin edge'dan berib, propagation delay'ni kamaytirish.

**6. Batching** — ko'p kichik so'rovni bittaga birlashtirish (10 ta alohida API call o'rniga 1 ta bulk call). Round trip soni kamayadi.

**7. Prefetching** — kerak bo'lishidan oldin ma'lumotni oldindan yuklab qo'yish (foydalanuvchi keyingi sahifaga o'tishini bashorat qilib).

**8. Reducing round trips** — eng kuchli printsip. Har bir round trip = kamida 1 RTT. So'rovlarni birlashtirish, sahifa resurslarini kamaytirish, redirect'lardan qochish.

```text
Optimizatsiya ta'siri (taxminan):
  connection reuse   -> handshake RTT'larini yo'qotadi
  HTTP/2             -> parallellik, kam ulanish
  compression        -> transmission delay ↓
  caching/CDN        -> propagation + processing ↓ (yoki butunlay yo'q)
  batching           -> round trip soni ↓
```

**💡 Tushuncha:** Eng samarali tartib: (1) so'rovni butunlay yo'qotish (cache), (2) round trip sonini kamaytirish (batch, HTTP/2), (3) har so'rovni yengillashtirish (compression). Mikrooptimizatsiyaga kirishishdan oldin shularni qil.

---

## Little's Law

**Little's Law** — navbat (queue) nazariyasidagi fundamental qonun:

```text
L = λ × W

L = tizimda ayni paytdagi o'rtacha so'rovlar soni (concurrency)
λ = kelish tezligi (arrival rate, requests/sec — ya'ni throughput)
W = bir so'rovning tizimda o'rtacha vaqti (latency)
```

**Amaliy ma'no:** Throughput, latency va concurrency bir-biriga bog'liq. Masalan:

```text
Agar throughput (λ) = 1000 req/s va o'rtacha latency (W) = 0.2 s bo'lsa,
tizimda doim L = 1000 × 0.2 = 200 ta so'rov "uchib" turadi.

=> Kamida 200 ta parallel ishlovchi (thread/connection) kerak.
```

**Nega muhim:** Bu thread pool, connection pool o'lchamini, kerakli server sonini hisoblashga yordam beradi. Agar latency oshsa (W↑) va throughputni saqlamoqchi bo'lsangiz (λ sobit), concurrency (L) oshishi kerak — ko'proq resurs.

**💡 Tushuncha:** Little's Law'ning kuchi — uning ichki tafsilotlarga bog'liq emasligi. Tizim qanday ishlashidan qat'i nazar, bu uchta o'lchov har doim shu munosabatda bog'liq.

---

## Performance o'lchash va budget

**Latency metrikalari:** p50, p95, p99, p99.9, max. Har doim percentile bilan o'lchanadi.

**Throughput metrikalari:** requests/sec (RPS), transactions/sec (TPS), MB/s. Yuk ostidagi maksimal barqaror tezlik.

**Boshqa muhim metrikalar:**
- **Saturation** — resurs qanchalik to'lganligi (CPU %, memory, connection pool).
- **Error rate** — yuk ortganda xatolar ko'payadimi.
- **Utilization** — resurs qanchalik band.

**Performance budget** — har bir komponentga ajratilgan "vaqt byudjeti". Masalan, sahifa 1 sekundda ochilishi kerak bo'lsa:

```text
Performance budget (jami 1000 ms):
  DNS + TLS handshake     150 ms
  Backend (API) javob     300 ms
  ├─ DB so'rovi           120 ms
  ├─ biznes logika         80 ms
  └─ tashqi xizmat        100 ms
  Tarmoq (transfer)       200 ms
  Frontend render         350 ms
  ──────────────────────────────
  JAMI                   1000 ms
```

**💡 Tushuncha:** Performance budget — komandaga "qaysi qism qancha vaqt sarflashi mumkin"ni oldindan kelishish imkonini beradi. Yangi feature budget'ni buzsa, optimizatsiya yoki kompromiss kerak. O'lchamasangiz — boshqarolmaysiz.

**⚠️ Ehtiyot bo'l:** Productionda o'lchamasdan optimizatsiya qilish — "premature optimization". Avval profiling/monitoring bilan haqiqiy bottleneck'ni toping, keyin optimizatsiya qiling. Taxmin ko'pincha noto'g'ri bo'ladi.

---

## Bottleneck topish

Backend sekin bo'lganda, qaysi qism aybdor ekanini topish jarayoni:

**1. Yuqoridan pastga o'lchash (RED metrics):** Rate (RPS), Errors, Duration (latency) — har bir xizmat/endpoint uchun. Qaysi endpoint sekin?

**2. Distributed tracing:** Bir so'rovning barcha xizmatlar bo'ylab yo'lini kuzatish (OpenTelemetry, Jaeger). Vaqt qayerda yo'qolayotganini ko'rsatadi.

```text
So'rov tracing (waterfall):
  API Gateway      ▆ 5ms
  Auth Service     ▆▆ 15ms
  Order Service    ▆▆▆▆▆▆▆▆▆▆▆▆▆▆ 180ms  <- BOTTLENECK!
    └─ DB query    ▆▆▆▆▆▆▆▆▆▆▆▆ 160ms     <- aslida bu yerda
  Notification     ▆ 8ms
```

**3. Resurs tekshirish (USE metrics):** Utilization, Saturation, Errors — har bir resurs (CPU, memory, disk, network) uchun. Qaysi resurs to'lgan?

**Keng tarqalgan backend bottleneck'lari:**

```text
Belgi                          Ehtimoliy sabab
──────────────────────────────────────────────────────────
DB so'rovlari sekin            indeks yo'q, N+1 query, lock
Yuk ortganda latency keskin↑   connection pool/thread pool tugadi
CPU 100% lekin RPS past        og'ir hisoblash, serialization
Memory o'sib boradi            memory leak, kesh cheksiz
Ba'zida juda sekin (p99)       GC pause, cache miss, noisy neighbor
Tarmoq kutish                  ketma-ket tashqi API chaqiruvlari
```

**💡 Tushuncha:** Eng ko'p uchraydigan backend bottleneck — **ma'lumotlar bazasi**. N+1 query (bir so'rov o'rniga yuzlab), yo'q indeks, yoki connection pool tugashi. Tracing va slow query log shularni ochib beradi.

**⚠️ Ehtiyot bo'l:** "Server sekin" deganda darhol ko'proq server qo'shmang. Avval o'lchang: agar bitta DB so'rovi 160ms bo'lsa, 10 ta server qo'shsangiz ham har bir so'rov baribir 160ms — throughput oshadi, lekin latency o'zgarmaydi. Bottleneck'ni *to'g'ri* aniqlash — yarim yechim.

---

## Savol-Javoblar

### ❓ Latency va throughput o'rtasidagi farq nima?

**✅ Javob:** Latency — bitta operatsiyaning boshidan oxirigacha vaqti (ms bilan o'lchanadi, "qancha tez"). Throughput — vaqt birligida bajarilgan operatsiyalar soni (req/s yoki MB/s, "qancha ko'p"). Truba analogiyasi: latency — bir tomchining trubadan o'tish vaqti (uzunligi), throughput — sekundiga necha litr oqishi (kengligi). Ular bog'liq emas — throughputni oshirish (ko'proq parallellik) latencyni kamaytirmaydi va aksincha.

### ❓ Bandwidth va throughput bir xilmi?

**✅ Javob:** Yo'q. Bandwidth — kanalning maksimal nazariy o'tkazuvchanligi (masalan, 100 Mbps), bu "trubaning eng katta kengligi". Throughput — amalda erishilgan tezlik. Throughput har doim bandwidth'dan kichik yoki teng, chunki protokol overhead'i, congestion, rettransmission va boshqa omillar nazariy maksimumga yetishga to'sqinlik qiladi.

### ❓ RTT nima va u nima uchun muhim?

**✅ Javob:** RTT (Round-Trip Time) — paketning manzilga borib-qaytish vaqti (ping shuni o'lchaydi). Muhim, chunki ko'p protokollar bir nechta round trip talab qiladi (TCP handshake 1 RTT, TLS 1-2 RTT), shuning uchun RTT to'g'ridan-to'g'ri foydalanuvchi seziladigan latency'ga ta'sir qiladi. Yorug'lik tezligi RTT'ni cheklaydi — qit'alararo ~100-200ms dan tezroq fizik jihatdan mumkin emas.

### ❓ "Har bir dasturchi bilishi kerak bo'lgan" latency raqamlaridan nimani eslab qolish kerak?

**✅ Javob:** Aniq raqamlardan ko'ra nisbatlar muhim: L1 cache ~1ns, RAM ~100ns (cache'dan ~100x sekin), SSD ~16µs, datacenter round trip ~500µs, disk seek ~10ms, qit'alararo internet ~150ms. Asosiy xulosa: memory (ns) → SSD (µs) → disk/network (ms) har qadamda ~1000x sekinlashadi. Shuning uchun keshlash, disk/tarmoqqa borishni kamaytirish, round triplarni qisqartirish eng kuchli optimizatsiyalar.

### ❓ Latency qanday komponentlardan tashkil topadi?

**✅ Javob:** To'rt manba: (1) propagation delay — signalning fizik masofani bosib o'tishi (yorug'lik tezligi bilan cheklangan, CDN bilan kamayadi); (2) transmission delay — ma'lumotni kanalga yuklash (data_size/bandwidth, compression bilan kamayadi); (3) processing delay — server qayta ishlashi (kod/DB optimizatsiyasi); (4) queuing delay — navbatda kutish (yuk ortganda keskin o'sadi). Optimizatsiyadan oldin qaysi komponent ustun ekanini aniqlash kerak.

### ❓ Tail latency nima va nega average'dan muhimroq?

**✅ Javob:** Tail latency — taqsimotning "dumi", ya'ni eng sekin so'rovlar (p99, p99.9). Average yolg'on gapiradi: o'rtacha 20ms bo'lsa ham, p99 = 2s bo'lishi mumkin. Foydalanuvchilar eng yomon holatlarni sezadi. Bitta sahifa 100 ta so'rov qilsa va har birining p99 1% sekin bo'lsa, deyarli har bir sahifada kamida bitta sekin so'rov bo'ladi. Shuning uchun SLO/SLA'lar percentile bilan beriladi, average bilan emas.

### ❓ p50, p95, p99 nimani anglatadi?

**✅ Javob:** Percentile (foizlik): p50 (median) — so'rovlarning 50% shu vaqtdan tez (tipik tajriba); p95 — 95% shu vaqtdan tez (5% sekinroq); p99 — 99% shu vaqtdan tez (1% sekinroq, bu tail latency). Misol: p50=20ms, p95=80ms, p99=300ms degani — tipik so'rov 20ms, lekin eng sekin 1% 300ms+. Tail latency sabablari: GC pause, cache miss, queuing, noisy neighbor.

### ❓ Tail latency'ni nima keltirib chiqaradi?

**✅ Javob:** Asosiy sabablar: GC (garbage collection) pauzalari (JVM/Go), cache miss (kesh topilmay diskka/tarmoqqa borish), queuing delay (yuk cho'qqilarida navbat), "noisy neighbor" (umumiy resursni boshqa workload band qilishi), slow disk, network jitter, va retry'lar. Microservice'larda kuchayadi: 10 ta xizmatni chaqirsangiz, umumiy latency eng sekin xizmatga bog'lanadi (fan-out amplifikatsiya).

### ❓ Bandwidth-delay product nima va nima uchun ahamiyatli?

**✅ Javob:** BDP = bandwidth × RTT — bir vaqtda "yo'lda" (in-flight) bo'lishi mumkin bo'lgan ma'lumot miqdori, ya'ni "trubadagi suv hajmi". Muhim, chunki TCP window BDP'dan kichik bo'lsa, kanal to'liq ishlatilmaydi — yuboruvchi ACK kutishga majbur, bandwidth bo'sh qoladi. "Long fat network" (yuqori bandwidth + yuqori RTT, masalan qit'alararo) uchun katta TCP window kerak; shuning uchun window scaling mavjud.

### ❓ Tarmoq latency'sini qanday kamaytirasiz?

**✅ Javob:** Eng samarali tartibda: (1) caching/CDN — so'rovni butunlay yo'qotish yoki yaqin edge'dan berish; (2) round trip sonini kamaytirish — keep-alive (connection reuse), HTTP/2 multiplexing, batching, redirect'lardan qochish; (3) har so'rovni yengillashtirish — gzip/brotli compression; (4) prefetching — kerak bo'lishidan oldin yuklash. Eng kuchli printsip — round trip sonini kamaytirish, chunki har biri kamida 1 RTT.

### ❓ HTTP keep-alive nima uchun latency'ni kamaytiradi?

**✅ Javob:** Keep-alive TCP+TLS ulanishini qayta ishlatadi, har so'rovda yangi ulanish ochmaydi. Yangi ulanish TCP handshake (1 RTT) + TLS handshake (1-2 RTT) talab qiladi — bu har so'rovga qo'shimcha 2-3 RTT. Keep-alive bilan bu handshake'lar faqat bir marta bo'ladi, keyingi so'rovlar tayyor ulanishdan foydalanadi (1 RTT). Yuqori RTT'li ulanishlarda bu juda katta tejamkorlik.

### ❓ Batching qanday yordam beradi va qachon ishlatiladi?

**✅ Javob:** Batching ko'p kichik so'rovni bittaga birlashtiradi (10 ta alohida API call o'rniga 1 ta bulk call). Round trip soni kamayadi — agar har biri 1 RTT bo'lsa, 10 RTT o'rniga 1 RTT. Throughputni oshiradi. Qachon: ko'p kichik operatsiyalar bir-biriga bog'liq bo'lmaganda (bulk insert, batch read). Kompromis: batching latency'ni biroz oshiradi (so'rovlarni to'plash kutiladi), shuning uchun real-time talablar bilan balanslash kerak.

### ❓ Little's Law nima va qayerda ishlatiladi?

**✅ Javob:** Little's Law: L = λ × W, bu yerda L — tizimdagi o'rtacha so'rovlar soni (concurrency), λ — kelish tezligi (throughput), W — o'rtacha latency. U throughput, latency va concurrencyni bog'laydi. Amalda: thread pool/connection pool o'lchamini hisoblashda ishlatiladi. Masalan, 1000 req/s va 0.2s latency bo'lsa, tizimda doim 200 ta so'rov uchib turadi — kamida 200 ta parallel ishlovchi kerak. Kuchi — tizim ichki ishlashidan qat'i nazar har doim amal qiladi.

### ❓ Performance budget nima?

**✅ Javob:** Performance budget — umumiy maqsadli vaqtni (masalan, sahifa 1 sekundda ochilishi) komponentlarga oldindan taqsimlash: DNS+TLS 150ms, backend 300ms, tarmoq 200ms, frontend render 350ms. Bu komandaga "qaysi qism qancha vaqt sarflashi mumkin"ni kelishish imkonini beradi. Yangi feature budget'ni buzsa, optimizatsiya yoki kompromiss talab qilinadi. O'lchamasangiz boshqarolmaysiz.

### ❓ Backend bottleneck'ini qanday topasiz?

**✅ Javob:** (1) RED metrics (Rate, Errors, Duration) bilan qaysi endpoint sekinligini aniqlash; (2) distributed tracing (Jaeger/OpenTelemetry) bilan so'rovning xizmatlar bo'ylab yo'lini kuzatib, vaqt qayerda yo'qolayotganini ko'rish; (3) USE metrics (Utilization, Saturation, Errors) bilan qaysi resurs to'lganini tekshirish. Eng ko'p uchraydigan bottleneck — ma'lumotlar bazasi (N+1 query, yo'q indeks, connection pool tugashi). Avval o'lchang, keyin optimizatsiya qiling — taxmin ko'pincha noto'g'ri.

### ❓ Throughput past bo'lsa server qo'shish latency'ni tuzatadimi?

**✅ Javob:** Yo'q. Server qo'shish (horizontal scaling) throughputni oshiradi (ko'proq so'rov bajariladi), lekin bitta so'rovning latency'sini o'zgartirmaydi. Agar bottleneck bitta DB so'rovi 160ms bo'lsa, 10 server qo'shsangiz ham har bir so'rov 160ms qoladi. Latency'ni kamaytirish uchun yo'lni qisqartirish kerak (kesh, indeks, CDN, kod optimizatsiyasi), throughput uchun esa kengaytirish (parallellik, ko'proq server). Bu farqni aralashtirmaslik muhim.

---

## Masalalar

> Yechimlar: [solutions/networking/06-latency-performance.md](../solutions/networking/06-latency-performance.md)

1. **Latency yoki throughput?** Quyidagi muammolarni latency yoki throughput muammosi sifatida tasniflang va yechim yo'nalishini ko'rsating: (a) sahifa 5 sekundda ochiladi; (b) tizim sekundiga atigi 50 so'rovni eplaydi, talab 500; (c) video oqimi uzilib-uzilib o'ynaydi; (d) batch hisobot 6 soat ishlaydi.

2. **Raqamlar intuitsiyasi.** Quyidagilarni tezdan sekinga tartiblang va taxminiy nisbatni ayting: RAM read, qit'alararo round trip, SSD read, L1 cache, datacenter ichidagi round trip, disk seek. Keyin tushuntiring: nima uchun "bir marta diskdan o'qib RAM'da keshlash" deyarli har doim foydali?

3. **Latency dekompozitsiyasi.** Toshkentdagi foydalanuvchi Frankfurtdagi serverga 2 MB JSON yuboradi (bandwidth 20 Mbps, RTT 120ms). Propagation, transmission, processing, queuing — qaysi komponent ustun? Har biri uchun bitta optimizatsiya taklif qiling.

4. **Percentile hisoblash.** Sizda 10 ta so'rov latency'si bor: 10, 12, 11, 13, 10, 14, 12, 11, 350, 13 (ms). p50, p90, p99 (taxminan) va average'ni baholang. Qaysi metrikani SLA'ga qo'yasiz va nega? 350ms'ning ehtimoliy sababi nima?

5. **Tail latency amplifikatsiyasi.** Bir sahifa 20 ta mustaqil mikroservis chaqiradi, har birining p99 = 1% so'rov sekin. Sahifaning kamida bitta sekin so'rovga duch kelish ehtimolini taxminan hisoblang. Buni kamaytirish uchun 3 ta strategiya taklif qiling.

6. **BDP va TCP window.** Ulanish: bandwidth 1 Gbps, RTT 80ms. BDP'ni hisoblang. Agar TCP window 256 KB bo'lsa, kanal to'liq ishlatiladimi? Bo'lmasa, qancha window kerak? Bu "long fat network" muammosini qanday ifodalaydi?

7. **Little's Law amaliyoti.** Servis 2000 req/s ni o'rtacha 50ms latency bilan eplaydi. Tizimda nechta so'rov bir vaqtda "uchib" turadi? Connection pool o'lchamini qancha qilasiz? Agar latency 200ms'gacha oshsa va throughputni saqlamoqchi bo'lsangiz, nima o'zgaradi?

8. **Optimizatsiya tartibi.** Mobil ilova bosh sahifasi sekin: 30 ta alohida API call, har biri yangi HTTPS ulanish, javoblar siqilmagan JSON, hech narsa keshlanmagan. Optimizatsiyalarni ta'sir/xarajat bo'yicha tartiblang va kutilayotgan natijani tushuntiring.

9. **Bottleneck diagnostikasi.** Backend yuk ortganda latency keskin (eksponensial) o'sadi, lekin CPU 40% da turadi. Distributed trace'da vaqtning ko'pi "DB query kutish"da. Ehtimoliy sabablarni sanang va tasdiqlash/yechish qadamlarini yozing.

10. **Performance budget loyihasi.** E-commerce mahsulot sahifasi uchun 800ms maqsadli budget tuzing: DNS/TLS, CDN/static, backend API (DB + logika + tashqi to'lov xizmati), tarmoq transfer, frontend render. Har bir qismга vaqt ajrating va qaysi qism eng xavfli (budget'ni buzishi mumkin) ekanini belgilang.

---

← [Networking bo'limiga qaytish](./README.md)
