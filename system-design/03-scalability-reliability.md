# Scalability va Reliability

Katta tizimlar ikki narsa bilan yashaydi: **o'sishga bardosh berish** (scalability) va **buzilmasdan xizmat qilish** (reliability). Bir server minglab foydalanuvchini ko'taradi, lekin millionlab foydalanuvchi, tarmoq uzilishlari, disk buzilishi va kutilmagan trafik spike'lari sharoitida tizim baribir ishlashi kerak. Bu hujjat masshtablanuvchanlik va ishonchlilikni O'zbek tilida, ingliz atamalari bilan senior darajada ko'rib chiqadi.

Biz quyidagilarni yoritamiz: scalability turlari (vertical vs horizontal, stateless design, scale cube), reliability vs availability vs durability farqi va "how many nines" jadvali, fault tolerance, chuqur **resilience patternlar** (redundancy, replication, failover, health check, circuit breaker, retry, timeout, bulkhead, rate limiting, load shedding, graceful degradation, fallback), single point of failure (SPOF), chaos engineering, auto-scaling, idempotency, backpressure, disaster recovery (RTO/RPO) va observability (metrics/logs/traces, SLA/SLO/SLI). Maqsad — har patternni **qachon** ishlatishni tushunish.

## Mundarija

- [Scalability nima](#scalability-nima)
- [Reliability vs Availability vs Durability](#reliability-vs-availability-vs-durability)
- [How many nines](#how-many-nines)
- [Fault tolerance](#fault-tolerance)
- [Resilience patternlar](#resilience-patternlar)
- [Circuit breaker](#circuit-breaker)
- [Retry, timeout, backoff](#retry-timeout-backoff)
- [Bulkhead, rate limiting, load shedding](#bulkhead-rate-limiting-load-shedding)
- [Graceful degradation va fallback](#graceful-degradation-va-fallback)
- [Single point of failure (SPOF)](#single-point-of-failure-spof)
- [Chaos engineering](#chaos-engineering)
- [Auto-scaling](#auto-scaling)
- [Idempotency va backpressure](#idempotency-va-backpressure)
- [Disaster recovery (RTO/RPO)](#disaster-recovery-rtorpo)
- [Observability va SLA/SLO/SLI](#observability-va-slaslosli)
- [Patternni qachon ishlatish](#patternni-qachon-ishlatish)
- [❓ Savol-javoblar](#-savol-javoblar)
- [Masalalar](#masalalar)

---

## Scalability nima

**💡 Tushuncha:** Scalability — tizimning **yuk oshganda** (ko'proq foydalanuvchi, so'rov, ma'lumot) unumdorlikni saqlab qolish yoki resurs qo'shish orqali o'sa olish qobiliyati. Yaxshi masshtablanadigan tizimda 2x resurs ≈ 2x throughput beradi (linear scaling).

Ikki asosiy yo'l bor:

```text
VERTICAL SCALING (scale up)          HORIZONTAL SCALING (scale out)
┌──────────────┐                     ┌────┐ ┌────┐ ┌────┐ ┌────┐
│  BITTA        │                    │ N1 │ │ N2 │ │ N3 │ │ N4 │
│  server:      │  →                 └────┘ └────┘ └────┘ └────┘
│  32 → 128 CPU │                      ↑ ko'proq mashina qo'shasan
│  64 → 512 GB  │
└──────────────┘
```

| Xususiyat | Vertical (scale up) | Horizontal (scale out) |
|-----------|---------------------|------------------------|
| Usul | Kuchliroq mashina | Ko'proq mashina |
| Chegara | Hardware chegarasi (bor) | Deyarli cheksiz |
| Murakkablik | Oddiy (kod o'zgarmaydi) | Murakkab (distributed) |
| Downtime | Odatda restart kerak | Nol downtime mumkin |
| SPOF | Ha (bitta mashina) | Yo'q (redundancy) |
| Narx | Eksponensial qimmatlashadi | Chiziqli (commodity HW) |

**💡 Tushuncha: Stateless design** — horizontal scaling'ning kaliti. Agar app server so'rovlar orasida **holat saqlamasa** (session, cache local'da bo'lmasa), har bir so'rovni istalgan instance ko'tara oladi. Holatni tashqariga chiqar: session → Redis, fayl → S3, ma'lumot → DB.

```text
STATEFUL (yomon)                    STATELESS (yaxshi)
so'rov → App1 (session xotirada)    so'rov → LB → istalgan App
keyingi so'rov → App2 → SESSION       ↓
YO'Q! → login qayta                 session → Redis (umumiy)
```

**💡 Tushuncha: Scale cube (AKF cube)** — masshtablashning uch o'qi:

```text
       Y (functional decomposition — microservices)
       │  turli xizmatlarga bo'lish
       │
       │
       └──────── X (horizontal duplication — clone/replica)
      /            bir xil app'ni ko'paytirish
     /
    Z (data partitioning — sharding)
      ma'lumotni bo'lish (user 1-1M → shard A)
```

- **X o'qi**: app'ni klonlash, LB ortiga qo'yish (eng oddiy).
- **Y o'qi**: funksiya bo'yicha bo'lish (auth service, payment service...).
- **Z o'qi**: ma'lumotni sharding qilish (user ID bo'yicha bo'lish).

**⚠️ Ehtiyot bo'l:** Avval **X o'qi** (klonlash) bilan boshlang — u eng arzon va oson. Sharding (Z) va microservices (Y) katta operatsion narx keltiradi; ularni faqat kerak bo'lganda qo'shing.

---

## Reliability vs Availability vs Durability

**💡 Tushuncha:** Bu uch atama tez-tez chalkashtiriladi, lekin ular boshqa-boshqa narsalar:

| Atama | Ta'rif | Savol |
|-------|--------|-------|
| **Reliability** | Tizim **to'g'ri** ishlaydi, xatosiz natija beradi | "Ishlaydimi va to'g'ri ishlaydimi?" |
| **Availability** | Tizim **so'rovga javob bera oladi** (up) | "Hozir ishlayaptimi (foizda)?" |
| **Durability** | Yozilgan ma'lumot **yo'qolmaydi** | "Ma'lumotim saqlanib qoladimi?" |

**💡 Tushuncha:**
- **Reliability** = MTBF (Mean Time Between Failures) — buzilishlar orasidagi o'rtacha vaqt. Tizim yuqori available bo'lishi, lekin noto'g'ri javob berishi mumkin (available lekin unreliable).
- **Availability** = uptime foizi = `MTBF / (MTBF + MTTR)`. MTTR = Mean Time To Recovery (tiklanish vaqti). Availability'ni oshirish uchun MTBF'ni oshir yoki MTTR'ni kamaytir.
- **Durability** = ma'lumot doimiyligi. Masalan S3 "11 nines durability" (99.999999999%) — 10 million obyektdan biri 10,000 yilda yo'qolishi mumkin. Bu replikatsiya orqali erishiladi.

```text
Available emas, durable  → DB o'chgan, lekin disk saqlangan
Available, durable emas   → javob beryapti, lekin yozuvni yo'qotdi (data loss)
Reliable emas             → javob beryapti, lekin NOTO'G'RI natija
```

---

## How many nines

**💡 Tushuncha:** Availability "to'qqizlar" (nines) bilan o'lchanadi. Har qo'shimcha "9" — downtime'ni ~10x kamaytiradi, lekin narx eksponensial oshadi.

| Availability | Nomi | Yiliga downtime | Oyiga | Kuniga |
|--------------|------|-----------------|-------|--------|
| 90% | one nine | 36.5 kun | 72 soat | 2.4 soat |
| 99% | two nines | 3.65 kun | 7.2 soat | 14.4 daqiqa |
| 99.9% | three nines | 8.76 soat | 43.8 daqiqa | 1.44 daqiqa |
| 99.95% | | 4.38 soat | 21.9 daqiqa | 43 soniya |
| 99.99% | four nines | 52.6 daqiqa | 4.38 daqiqa | 8.6 soniya |
| 99.999% | five nines | 5.26 daqiqa | 26 soniya | 0.86 soniya |

**💡 Tushuncha:** 99.9% dan 99.99% ga o'tish — downtime'ni 8.76 soatdan 52 daqiqaga tushirish. Bu multi-region, avtomatik failover, redundant hamma qatlam degani — juda qimmat.

**⚠️ Ehtiyot bo'l:** Ketma-ket komponentlar availability'ni **ko'paytiradi** (multiply). Agar zanjirda 3 ta komponent har biri 99.9% bo'lsa: `0.999 * 0.999 * 0.999 ≈ 99.7%`. Ko'proq bog'liqlik = kam availability. Redundancy esa aksincha oshiradi: ikkita parallel 99% komponent = `1 - (0.01 * 0.01) = 99.99%`.

---

## Fault tolerance

**💡 Tushuncha:** Fault tolerance — tizimning **bir qism buzilganda ham ishlashda davom etishi**. Maqsad: bitta komponent nosozligini foydalanuvchiga sezdirmaslik. Bu redundancy (dublikat) va failover (avtomatik o'tish) orqali erishiladi.

Farqni ajrat:
- **Fault** — nosozlik (disk o'ldi, tarmoq uzildi).
- **Error** — nosozlik keltirgan noto'g'ri holat.
- **Failure** — tizim xizmat ko'rsata olmasligi (foydalanuvchi ko'radi).

**Maqsad:** fault → error bo'lsin, lekin failure'gacha yetmasin. Yaxshi tizim fault'larni **izolyatsiya** qiladi (bulkhead) va ulardan tiklanadi (retry, failover).

---

## Resilience patternlar

**💡 Tushuncha:** Resilience — tizimning nosozlikdan **tiklanish** va uni **ushlab turish** qobiliyati. Quyida senior darajada bilishing kerak bo'lgan asosiy patternlar.

### Redundancy

**💡 Tushuncha:** Kritik komponentni **dublikat** qilish. Agar biri o'lsa, ikkinchisi davom etadi. SPOF'ni yo'qotishning asosiy usuli.

```text
Redundancy turlari:
  Active-Active   → ikkalasi ham ishlaydi (yuk taqsimlanadi)
  Active-Passive  → biri ishlaydi, ikkinchisi standby (kutadi)
  N+1             → N ta kerak, +1 zaxira
```

### Replication

**💡 Tushuncha:** Ma'lumotni bir nechta nusxada saqlash (DB replica, S3 3x kopiya). Redundancy ma'lumot uchun. Bu durability va read scaling beradi.

```text
                ┌──────────┐  yozish
   Client ────► │ PRIMARY  │ ────┐
                └──────────┘     │ replikatsiya
                                 ▼
                ┌──────────┐  ┌──────────┐
   o'qish ◄──── │ REPLICA1 │  │ REPLICA2 │
                └──────────┘  └──────────┘
```

### Failover

**💡 Tushuncha:** Primary o'lsa, avtomatik ravishda replica'ga (yoki standby'ga) o'tish.

```text
ACTIVE-PASSIVE                    ACTIVE-ACTIVE
┌─────────┐   ┌─────────┐        ┌─────────┐   ┌─────────┐
│ ACTIVE  │   │ PASSIVE │        │ ACTIVE  │   │ ACTIVE  │
│ (ishlar)│   │(standby)│        │ (50%)   │   │ (50%)   │
└────┬────┘   └────┬────┘        └────┬────┘   └────┬────┘
     │ o'ldi       │                  └───── LB ────┘
     └──failover──►│ endi active      biri o'lsa → boshqasi 100%
```

| | Active-Passive | Active-Active |
|-|----------------|---------------|
| Resurs | Yarmi bo'sh turadi | Hammasi ishlaydi |
| Failover vaqti | Bor (standby ko'tariladi) | ~nol (allaqachon ishlar) |
| Murakkablik | Oddiyroq | Murakkab (data sync, conflict) |

### Health check

**💡 Tushuncha:** LB yoki orkestrator har komponentga muntazam "tirikmisan?" so'rovi yuboradi. Javob bermasa — trafikdan chiqaradi.

```text
LB ──/health──► App1  → 200 OK  ✓ (trafik oladi)
LB ──/health──► App2  → timeout ✗ (trafikdan chiqadi)
```

- **Liveness**: jarayon tirikmi? (o'lsa restart).
- **Readiness**: so'rov qabul qilishga tayyormi? (tayyor emas → trafik yubormaslik).

---

## Circuit breaker

**💡 Tushuncha:** Circuit breaker — pastki (downstream) xizmat buzilganda, unga so'rov yuborishni **to'xtatib**, tizimni himoya qiladi. Elektrdagi avtomat kabi: qisqa tutashuvda uzadi. Maqsad — o'lgan xizmatni so'rovlar bilan ko'mib, **kaskad nosozlik** (cascading failure) keltirmaslik.

Uch holati bor:

```text
        xatolar chegaradan oshdi
  ┌──────────────────────────────────┐
  │                                   ▼
┌────────┐  timeout tugadi   ┌──────────┐
│ CLOSED │ ◄──── success ────│ HALF-OPEN│
│ (normal)│                  │ (sinov)  │
└────────┘                   └──────────┘
  │  ▲                          ▲   │ fail
  │  │ N soniya kutish          │   ▼
  ▼  │                       ┌──────────┐
 ...└───────────────────────│   OPEN   │
   xatolar oshsa            │ (bloklangan)│
   OPEN'ga                  └──────────┘
```

| Holat | Ma'nosi | Xatti-harakat |
|-------|---------|---------------|
| **CLOSED** | Normal | So'rovlar o'tadi. Xatolar sanaladi. |
| **OPEN** | Buzilgan | So'rovlar darhol rad etiladi (fail fast), fallback ishlaydi. |
| **HALF-OPEN** | Sinov | Bir nechta test so'rov o'tkaziladi. Muvaffaqiyat → CLOSED, xato → OPEN. |

```js
// Soddalashtirilgan circuit breaker
class CircuitBreaker {
  constructor(fn, { threshold = 5, cooldown = 30000 } = {}) {
    this.fn = fn;
    this.threshold = threshold;   // ketma-ket xato limiti
    this.cooldown = cooldown;     // OPEN'da kutish (ms)
    this.state = "CLOSED";
    this.failures = 0;
    this.nextTry = 0;
  }
  async call(...args) {
    if (this.state === "OPEN") {
      if (Date.now() < this.nextTry) throw new Error("circuit OPEN"); // fail fast
      this.state = "HALF-OPEN";   // sinovga o'tamiz
    }
    try {
      const res = await this.fn(...args);
      this.failures = 0;
      this.state = "CLOSED";      // muvaffaqiyat → tiklandi
      return res;
    } catch (e) {
      this.failures++;
      if (this.failures >= this.threshold) {
        this.state = "OPEN";
        this.nextTry = Date.now() + this.cooldown;
      }
      throw e;
    }
  }
}
```

**⚠️ Ehtiyot bo'l:** Circuit breaker'siz retry — falokat. O'lgan xizmatga hamma retry qilaversa, u tiklanganda ham **retry storm** bilan yana o'ladi. Circuit breaker + retry birga ishlashi kerak.

---

## Retry, timeout, backoff

**💡 Tushuncha:** Vaqtinchalik (transient) nosozliklar — tarmoq gliti, qisqa timeout — retry bilan hal bo'ladi. Lekin **soddadan retry qilma** — bu yukni oshiradi.

### Timeout

**💡 Tushuncha:** Har tashqi chaqiruvga **timeout** qo'y. Timeout'siz so'rov abadiy osilib qolishi mumkin — thread pool to'ladi, hamma bloklanadi.

```text
Client → Service (timeout 2s)
  2s ichida javob yo'q → uzadi, resursni bo'shatadi
```

### Retry + exponential backoff + jitter

**💡 Tushuncha:** Xatodan keyin darhol qayta urinma — **kutib**, keyin ur. Har urinishda kutishni **ikki barobar** oshir (exponential backoff). Va **jitter** (tasodifiy shovqin) qo'sh, aks holda hamma client bir vaqtda retry qiladi (thundering herd).

```text
Backoff jitter'siz:  1s, 2s, 4s, 8s   ← hamma bir vaqtda uradi
Backoff + jitter:    0.7s, 2.3s, 3.1s, 7.8s  ← tarqaladi
```

```js
async function retry(fn, { retries = 3, base = 200, cap = 5000 } = {}) {
  for (let i = 0; i <= retries; i++) {
    try {
      return await fn();
    } catch (e) {
      if (i === retries) throw e;
      const backoff = Math.min(cap, base * 2 ** i);      // exponential
      const jitter = Math.random() * backoff;            // full jitter
      await new Promise((r) => setTimeout(r, jitter));
    }
  }
}
```

**⚠️ Ehtiyot bo'l:** Faqat **idempotent** operatsiyalarni bemalol retry qil. `POST /payments`ni idempotency key'siz retry qilsang — ikki marta pul yechiladi. Retry'ni faqat retryable xatolar (5xx, timeout) uchun qil, 4xx (bad request) uchun emas.

---

## Bulkhead, rate limiting, load shedding

### Bulkhead

**💡 Tushuncha:** Kema bo'linmalari (bulkhead) kabi — bir bo'lim suvga to'lsa, boshqalari saqlanadi. Resurslarni **izolyatsiya** qilib, bir qism nosozligi butun tizimni cho'ktirmasin.

```text
Bulkhead'siz: bitta umumiy thread pool (100)
  Service-A sekinlashsa → 100 thread band → hamma bloklandi

Bulkhead bilan: alohida poollar
  ┌──────────┐ ┌──────────┐ ┌──────────┐
  │ A: 40 th │ │ B: 40 th │ │ C: 20 th │
  └──────────┘ └──────────┘ └──────────┘
  A o'lsa → faqat A pool band, B va C ishlaydi
```

### Rate limiting

**💡 Tushuncha:** Foydalanuvchi/client boshiga so'rov tezligini cheklash. Overload va abuse'dan himoya. Algoritmlar: token bucket, leaky bucket, sliding window.

```text
Token bucket: har soniyada N token to'ldiriladi
  so'rov keldi → token bormi? bor → o'tkaz, token oling
                            yo'q → 429 Too Many Requests
```

### Load shedding

**💡 Tushuncha:** Tizim overload bo'lganda, **past prioritetli** so'rovlarni ataylab rad etib, tizim to'liq qulashining oldini olish. "Hammani sekin qilgandan ko'ra, ba'zilarni tez rad etgan yaxshi."

```text
CPU 95% → load shedding yoqiladi
  kritik so'rov (checkout) → o'tadi
  past prioritet (recommendations) → 503 rad etiladi
```

**💡 Tushuncha:** Rate limiting **oldindan** (client bo'yicha kvota), load shedding **reaktiv** (tizim holati bo'yicha, real vaqtda). Ikkalasi ham overload'dan himoya, lekin turli qatlamda.

---

## Graceful degradation va fallback

**💡 Tushuncha:** Graceful degradation — bir qism buzilganda, tizim **to'liq qulamasdan**, cheklangan funksiya bilan ishlashda davom etadi. "Yomonroq, lekin ishlaydi" — "umuman ishlamaydi"dan yaxshi.

**💡 Tushuncha:** Fallback — asosiy manba ishlamaganda **zaxira javob**. Masalan: recommendation service o'lsa → mashhur mahsulotlar ro'yxatini ko'rsat; jonli narx yo'q → cache'dagi eski narxni ko'rsat (stale-while-revalidate).

```text
Product page:
  ✓ nom, rasm, narx     (kritik — DB)
  ✗ recommendations     (o'ldi → bo'sh yoki "top mahsulotlar" fallback)
  ✗ real-time stock     (o'ldi → "mavjud" deb ko'rsat, checkout'da tekshir)
Natija: sahifa ochiladi, faqat qo'shimchalar yo'q → degradation
```

**⚠️ Ehtiyot bo'l:** Fallback ma'lumoti **stale** (eski) bo'lishi mumkin — buni foydalanuvchiga bildirish yoki kritik joyda (to'lov) ishlatmaslik kerak.

---

## Single point of failure (SPOF)

**💡 Tushuncha:** SPOF — bitta komponent, u o'lsa **butun tizim** o'ladi. Reliable tizimda SPOF bo'lmasligi kerak. Har qatlamni tekshir: LB, DB, cache, tarmoq, DNS, hatto deploy pipeline.

```text
SPOF                          Bartaraf etish
┌──────────┐                  ┌──────┐  ┌──────┐
│ bitta LB │  o'ldi → hammasi │ LB1  │  │ LB2  │  (active-passive
└──────────┘  o'ldi           └──────┘  └──────┘   + floating IP)

bitta DB primary          → primary + replica + auto-failover
bitta region              → multi-region
bitta cache node          → cluster + replica
```

**💡 Tushuncha:** SPOF'ni topish uchun har komponentga qara: "Buni o'chirsam nima bo'ladi?" Agar javob "hamma to'xtaydi" bo'lsa — bu SPOF. Yechim doim: **redundancy + failover**.

---

## Chaos engineering

**💡 Tushuncha:** Chaos engineering — production'da **ataylab nosozlik keltirib**, tizim qanday chidashini sinash. Maqsad: nosozlik real bo'lishidan **oldin** zaifliklarni topish. "Umid strategiya emas" — buzilishni kutib o'tirma, o'zing keltir va o'rgan.

**Nega kerak:** Distributed tizimda nosozliklar muqarrar. Buni test muhitida taqlid qilish qiyin. Faqat real yukda haqiqiy zaifliklar ko'rinadi.

**💡 Tushuncha: Netflix Chaos Monkey** — bu g'oyaning boshlovchisi. U production'da tasodifiy instance'larni o'chiradi, jamoa esa tizimni shunga chidamli qilishga majbur bo'ladi. Kengroq "Simian Army": Latency Monkey (kechikish qo'shadi), Chaos Gorilla (butun AZ'ni o'chiradi).

```text
Chaos experiment tartibi:
1. "Steady state" (normal) metrikani aniqla
2. Gipoteza: "instance o'lsa, availability o'zgarmaydi"
3. Nosozlik kirit (instance o'chir)
4. Metrikani kuzat — gipoteza tasdiqlandimi?
5. Zaiflik topilsa — tuzat
```

**⚠️ Ehtiyot bo'l:** Chaos'ni kichik "blast radius" bilan boshla (bitta instance), production'ni butunlay buzma. Avval staging'da sina, keyin ehtiyotkorlik bilan production'da.

---

## Auto-scaling

**💡 Tushuncha:** Auto-scaling — yuk o'zgarishiga qarab instance sonini **avtomatik** oshirish/kamaytirish. Peak'da resurs qo'shadi, tinch vaqtda kamaytirib pul tejaydi.

```text
Metric (CPU > 70%)  → scale out (+2 instance)
Metric (CPU < 30%)  → scale in  (-1 instance)
```

Turlari:
- **Reactive (target tracking)**: joriy metrika (CPU, RPS) bo'yicha.
- **Scheduled**: ma'lum vaqtda (masalan har kuni soat 9:00 da peak kutiladi).
- **Predictive**: ML bilan kelajak yukni bashorat qilib, oldindan tayyorlash.

**⚠️ Ehtiyot bo'l:** Yangi instance darrov tayyor bo'lmaydi (warm-up). Spike tez kelsa, scale-up kech qolishi mumkin — shuning uchun buffer capacity qoldiring. Va "flapping" dan (tez oshirib-kamaytirish) qochish uchun cooldown qo'ying.

---

## Idempotency va backpressure

### Idempotency

**💡 Tushuncha:** Idempotent operatsiya — bir necha marta bajarilsa ham **natija bir xil** bo'ladi. Reliability uchun kritik: retry, at-least-once delivery, network noaniqligida ikki marta bajarilish xavfsiz bo'lishi kerak.

```text
GET, PUT, DELETE — tabiatan idempotent
POST — emas! (ikki marta → ikki buyurtma)

Yechim: idempotency key
  POST /orders  Idempotency-Key: abc-123
  server: bu key'ni ko'rdimmi?
    ha → oldingi natijani qaytar (yangi buyurtma yo'q)
    yo'q → bajaradi, key'ni saqlaydi
```

### Backpressure

**💡 Tushuncha:** Backpressure — iste'molchi (consumer) ishlab chiqaruvchidan (producer) sekinroq bo'lganda, producer'ga "sekinroq!" signali. Aks holda buffer/queue to'lib toshadi (OOM).

```text
Producer (tez) ──► Queue ──► Consumer (sekin)
                    │
              to'lib boradi → backpressure signali
                    ▼
   Producer sekinlaydi / rad etadi / diskka yozadi
```

**💡 Tushuncha:** Backpressure strategiyalari: bloklash (producer kutadi), buffer (chegara bilan), drop (eski/yangini tashla), yoki upstream'ga xato qaytarish. Load shedding ham backpressure'ning bir shakli.

---

## Disaster recovery (RTO/RPO)

**💡 Tushuncha:** Disaster recovery (DR) — katta falokat (region o'chishi, ma'lumot buzilishi) dan keyin tizimni tiklash rejasi. Ikki asosiy o'lchov:

```text
        FALOKAT
          │
  ...─────┼─────► vaqt
      ▲   │   ▲
      │   │   │
   oxirgi │  tiklandi
   backup │
      │←RPO→│←──RTO──►│
```

| Metrik | To'liq nomi | Savol |
|--------|-------------|-------|
| **RTO** | Recovery Time Objective | "Qancha vaqtda tiklanishimiz kerak?" (downtime chegarasi) |
| **RPO** | Recovery Point Objective | "Qancha ma'lumot yo'qolishi mumkin?" (backup oralig'i) |

**💡 Tushuncha:** RTO = 1 soat → 1 soatda tiklanish. RPO = 5 daqiqa → ko'pi bilan 5 daqiqalik ma'lumot yo'qoladi (ya'ni backup har 5 daqiqada). Kichik RTO/RPO = qimmat.

DR strategiyalari (arzondan qimmatga):
- **Backup & restore**: eng arzon, RTO soatlar.
- **Pilot light**: minimal zaxira ishlab turadi, kerak bo'lsa kengaytiriladi.
- **Warm standby**: kichraytirilgan nusxa doim ishlaydi.
- **Multi-site active-active**: to'liq ikki region, RTO ~nol, eng qimmat.

---

## Observability va SLA/SLO/SLI

**💡 Tushuncha:** Observability — tizim ichida **nima bo'layotganini** tashqaridan tushuna olish. "Uch ustun" (three pillars):

| Ustun | Nima | Misol |
|-------|------|-------|
| **Metrics** | Vaqt bo'yicha sonli qiymatlar | CPU %, RPS, p99 latency |
| **Logs** | Diskret hodisalar yozuvi | "user 42 login failed" |
| **Traces** | So'rovning xizmatlar orasidagi yo'li | request → API → DB → cache |

**💡 Tushuncha:** Monitoring ("ma'lum savollarga javob": CPU qancha?) va observability ("noma'lum savollarni surishtirish": nega bu 3 ta so'rov sekin?) farqi bor. Traces distributed tizimda kritik — bir so'rov 10 ta xizmatdan o'tadi.

Reliability shartnomalari:

| Atama | To'liq nomi | Ma'nosi |
|-------|-------------|---------|
| **SLI** | Service Level Indicator | O'lchanadigan metrika (masalan availability %, p99 latency) |
| **SLO** | Service Level Objective | Ichki maqsad (masalan "99.9% availability") |
| **SLA** | Service Level Agreement | Mijoz bilan **shartnoma** + jarima (SLO buzilsa) |

```text
SLI  = 99.95% (o'lchandi)
SLO  = 99.9%  (maqsad)   → SLI > SLO, yaxshi
SLA  = 99.5%  (shartnoma, buzilsa pul qaytariladi)

Error budget = 100% - SLO = 0.1% → bu qadar "buzilishga ruxsat"
```

**💡 Tushuncha: Error budget** — SLO 99.9% bo'lsa, 0.1% "buzilish byudjeti" bor. Byudjet ichida bo'lsa — risk qilib deploy qilsang bo'ladi. Byudjet tugasa — yangi feature to'xtatilib, barqarorlikka e'tibor qaratiladi. Bu dev tezligi va reliability o'rtasidagi muvozanat.

---

## Patternni qachon ishlatish

| Pattern | Qachon ishlatish |
|---------|------------------|
| Horizontal scaling | Trafik oshsa, vertical chegaraga yetsa |
| Stateless design | Har doim (horizontal scaling sharti) |
| Redundancy / failover | Har kritik komponent uchun (SPOF yo'qotish) |
| Replication | Read scaling + durability kerak bo'lsa |
| Circuit breaker | Ishonchsiz tashqi/downstream xizmatga chaqiruvda |
| Retry + backoff | Transient (vaqtinchalik) xatolar bo'lsa, idempotent bo'lsa |
| Timeout | Har tashqi chaqiruvda — istisnosiz |
| Bulkhead | Bir necha downstream'ni izolyatsiya qilish kerak bo'lsa |
| Rate limiting | Abuse/overload'dan himoya, adolatli taqsimot |
| Load shedding | Overload'da tizimni saqlab qolish uchun |
| Graceful degradation | Foydalanuvchiga qisman xizmat qabul bo'lsa |
| Auto-scaling | Yuk o'zgaruvchan (peak/off-peak) bo'lsa |
| Idempotency | Retry/queue bo'lgan har joyda (ayniqsa yozishda) |
| Chaos engineering | Yetuk, kritik tizimda ishonchni oshirish uchun |
| Multi-region DR | Kichik RTO/RPO talab qilinsa (biznes-kritik) |

---

## ❓ Savol-javoblar

### ❓ Vertical va horizontal scaling farqi nima, qaysi biri afzal?

**✅ Javob:** Vertical (scale up) — bitta mashinani kuchaytirish (ko'proq CPU/RAM). Oddiy, kod o'zgarmaydi, lekin hardware chegarasi bor va SPOF qoladi. Horizontal (scale out) — ko'proq mashina qo'shish. Deyarli cheksiz, redundancy beradi, lekin distributed murakkabligi (state, consistency, LB) keltiradi. Amalda: avval vertical (arzon, oson), keyin horizontal (chegaraga yaqinlashganda). Stateless dizayn horizontal scaling sharti.

### ❓ Reliability va availability farqi nimada?

**✅ Javob:** Availability — tizim **javob bera oladimi** (uptime %). Reliability — tizim **to'g'ri** ishlaydimi (xatosiz, kutilgan natija). Tizim available bo'lib, lekin noto'g'ri natija berishi mumkin (available lekin unreliable). Availability = MTBF/(MTBF+MTTR). Durability — bu boshqa narsa: yozilgan ma'lumot yo'qolmasligi.

### ❓ 99.9% va 99.99% availability orasidagi farq amalda qancha?

**✅ Javob:** 99.9% (three nines) = yiliga ~8.76 soat downtime. 99.99% (four nines) = yiliga ~52 daqiqa. Farqi ~10x kam downtime, lekin narx eksponensial: multi-region, avtomatik failover, hamma qatlamda redundancy kerak. Har qo'shimcha "9" jiddiy muhandislik investitsiyasi talab qiladi.

### ❓ Circuit breaker qanday ishlaydi, uch holati nima?

**✅ Javob:** Downstream xizmat buzilganda so'rovlarni to'xtatib kaskad nosozlikni oldini oladi. **CLOSED** — normal, so'rovlar o'tadi, xatolar sanaladi. Chegaradan oshsa → **OPEN** — so'rovlar darhol rad etiladi (fail fast) + fallback. Cooldown o'tgach → **HALF-OPEN** — bir necha test so'rov: muvaffaqiyat bo'lsa CLOSED'ga, xato bo'lsa yana OPEN'ga. Retry bilan birga ishlatilishi shart, aks holda retry storm bo'ladi.

### ❓ Nega retry'da jitter kerak?

**✅ Javob:** Jitter'siz exponential backoff'da hamma client bir xil intervalda (1s, 2s, 4s) retry qiladi — natijada barcha so'rovlar **bir vaqtda** kelib, o'lgan xizmatni yana ko'mib tashlaydi (thundering herd / retry storm). Jitter (tasodifiy shovqin) retry'larni vaqt bo'yicha tarqatadi, yukni tekislaydi. Odatda "full jitter": `random(0, backoff)`.

### ❓ Bulkhead pattern nima uchun kerak?

**✅ Javob:** Resurslarni (thread pool, connection pool) izolyatsiya qiladi, shunda bir downstream xizmat sekinlashsa yoki o'lsa, u faqat o'z poolini band qiladi va butun tizimni bloklashga olib kelmaydi. Kema bo'linmalari kabi. Bulkhead'siz umumiy pool'da bitta sekin xizmat hamma thread'ni band qilib, boshqa sog'lom xizmatlarni ham cho'ktiradi.

### ❓ Rate limiting va load shedding farqi?

**✅ Javob:** Rate limiting — **proaktiv**, client boshiga oldindan belgilangan kvota (masalan daqiqada 100 so'rov), abuse va adolatli taqsimot uchun. Load shedding — **reaktiv**, tizim real vaqtdagi holati (CPU 95%) bo'yicha past prioritetli so'rovlarni rad etib to'liq qulashning oldini oladi. Ikkalasi overload'dan himoya, lekin biri client bo'yicha, ikkinchisi tizim holati bo'yicha.

### ❓ SPOF nima va uni qanday topasiz?

**✅ Javob:** SPOF (Single Point Of Failure) — bitta komponent, u o'lsa butun tizim ishdan chiqadi. Topish uchun har komponentga "buni o'chirsam nima bo'ladi?" deb so'ra — javob "hamma to'xtaydi" bo'lsa, bu SPOF. Odatiy joylar: bitta LB, bitta DB primary, DNS, bitta region, deploy pipeline. Yechim doim bir xil: redundancy + avtomatik failover.

### ❓ Chaos engineering nima va nega Netflix uni ixtiro qildi?

**✅ Javob:** Production'da ataylab nosozlik keltirib (instance o'chirish, latency qo'shish), tizim chidamliligini sinash. Netflix cloud'ga ko'chganda, instance'lar tasodifan o'lishini bilardi — shuning uchun Chaos Monkey yaratdi: u tasodifiy instance'larni o'chiradi, jamoa esa tizimni har doim shunga chidamli qurishga majbur bo'ladi. "Nosozlikni kutma, o'zing keltir va undan oldin o'rgan" falsafasi. Kichik blast radius bilan boshlash muhim.

### ❓ Idempotency reliability uchun nega muhim?

**✅ Javob:** Distributed tizimda tarmoq noaniq: so'rov yetdimi yoki javob yo'qoldimi — bilib bo'lmaydi, shuning uchun retry kerak. Agar operatsiya idempotent bo'lsa, uni xavfsiz qayta yuborish mumkin (natija o'zgarmaydi). At-least-once delivery bilan queue'da ham xabar ikki marta yetishi mumkin. Idempotency key (POST uchun) takroriy bajarilishni oldini oladi — masalan to'lov ikki marta yechilmaydi.

### ❓ RTO va RPO farqi nima?

**✅ Javob:** RTO (Recovery Time Objective) — falokatdan keyin **qancha vaqtda** tiklanish kerak (downtime chegarasi). RPO (Recovery Point Objective) — **qancha ma'lumot** yo'qolishi mumkin (backup oralig'i). Masalan RTO=1soat, RPO=5daqiqa → 1 soatda tiklan, ko'pi bilan 5 daqiqalik ma'lumot yo'qot (demak backup har 5 daqiqada). Kichik qiymatlar qimmat: warm standby yoki multi-region kerak bo'ladi.

### ❓ SLA, SLO, SLI orasidagi farq?

**✅ Javob:** SLI (Indicator) — o'lchanadigan metrika (real availability %, p99 latency). SLO (Objective) — ichki maqsad ("99.9% availability"). SLA (Agreement) — mijoz bilan yuridik shartnoma + jarima (buzilsa pul qaytariladi). Odatda SLO > SLA (ichki bar balandroq). Error budget = 100% - SLO: bu qadar buzilishga "ruxsat"; byudjet ichida bo'lsa risk qilib deploy qilinadi, tugasa barqarorlikka o'tiladi.

### ❓ Observability'ning uch ustuni nima?

**✅ Javob:** Metrics (vaqt bo'yicha sonli qiymatlar — CPU, RPS, latency, agregatsiya uchun), logs (diskret hodisalar — "user X failed", batafsil kontekst uchun), traces (bir so'rovning xizmatlar orasidagi yo'li, distributed'da bottleneck topish uchun). Monitoring ma'lum savollarga javob beradi, observability esa oldindan bilinmagan muammolarni surishtirishga imkon beradi.

### ❓ Availability zanjirda va parallelda qanday hisoblanadi?

**✅ Javob:** Ketma-ket (seriya) bog'liqlik availability'ni **ko'paytiradi**: uchta 99.9% komponent = 0.999³ ≈ 99.7% (kamayadi!). Ko'proq bog'liqlik = kam availability. Parallel (redundant) esa **oshiradi**: ikkita 99% komponent (biri ishlasa yetadi) = 1 - 0.01² = 99.99%. Shuning uchun kritik yo'lda bog'liqlikni kamaytir va redundancy qo'sh.

### ❓ Graceful degradation'ga misol ayting.

**✅ Javob:** E-commerce mahsulot sahifasida: DB'dan nom/narx/rasm (kritik) keladi, lekin recommendation service o'lsa — sahifa baribir ochiladi, faqat tavsiyalar bo'lmaydi (yoki "top mahsulotlar" fallback ko'rsatiladi). Real-time stock service o'lsa — "mavjud" deb ko'rsatiladi va checkout'da qayta tekshiriladi. Natija: to'liq qulash o'rniga qisman, ishlaydigan xizmat.

### ❓ Backpressure nima va qanday hal qilinadi?

**✅ Javob:** Consumer producer'dan sekinroq bo'lganda, buffer/queue to'lib toshadi (OOM xavfi). Backpressure — producer'ga "sekinroq" signali. Hal qilish: bloklash (producer kutadi), chegaralangan buffer, drop (eski/yangi xabarni tashlash), yoki upstream'ga xato qaytarish (429/503). Reactive Streams kabi framework'larda backpressure o'rnatilgan. Load shedding ham backpressure shakli.

---

## Masalalar

> Yechimlar: [`solutions/system-design/03-scalability-reliability.md`](../solutions/system-design/03-scalability-reliability.md)

1. **Nines hisobi.** Tizimingiz zanjirida 4 ta bog'liq komponent bor, har biri 99.95% available. Umumiy availability qancha? Uni 99.99% ga ko'tarish uchun nima qilasiz? Yiliga downtime farqini jadval bilan ko'rsating.

2. **Circuit breaker dizayni.** Tashqi to'lov provayderiga (payment gateway) chaqiruv uchun circuit breaker dizayn qiling: threshold, cooldown, half-open logikasi va OPEN holatda fallback nima bo'lishini aniqlang. Holatlar diagrammasini chizing.

3. **Retry strategiyasi.** Bir microservice boshqasiga chaqiruv qiladi va vaqti-vaqti bilan 503 oladi. To'liq retry strategiyasini yozing: qaysi xatolar retryable, backoff formulasi, jitter, max retries, va timeout. Idempotency masalasini qanday hal qilasiz?

4. **SPOF auditi.** Quyidagi arxitekturadagi har qatlamdagi SPOF'ni toping va yeching: `DNS → CDN → LB → App → Redis → PostgreSQL (bitta primary) → S3`. Har biri uchun redundancy/failover taklif qiling.

5. **DR rejasi.** Biznes-kritik to'lov tizimi uchun RTO=15 daqiqa, RPO=1 daqiqa talab qilinadi. Qaysi DR strategiyasini (backup&restore / pilot light / warm standby / active-active) tanlaysiz va nega? Arxitekturani tasvirlang.

6. **Load shedding.** API gateway'ingiz peak'da overload bo'lyapti. Load shedding mexanizmini dizayn qiling: qaysi signalga qaraysiz (CPU/queue depth/latency), so'rovlarni qanday prioritetlaysiz, va nimani rad etasiz. Rate limiting'dan farqini tushuntiring.

7. **Auto-scaling tuning.** CPU-ga asoslangan auto-scaling'ingiz trafik spike'da kech qolyapti (foydalanuvchilar xato ko'ryapti) va tinch vaqtda flapping qilyapti. Ikki muammoni ham qanday hal qilasiz? Qaysi metrik, threshold, cooldown va warm-up sozlamalarini o'zgartirasiz?

8. **Observability rejasi.** Yangi microservice deploy qildingiz. Uni to'liq observable qilish uchun qaysi metrics (SLI), logs va traces kerak? SLO va error budget'ni qanday belgilaysiz? "p99 latency oshdi" alarm ishlaganda qanday tekshirasiz?

9. **Graceful degradation.** Video streaming platformasi dizayn qiling, unda recommendation, comments va analytics servislari o'lsa ham asosiy oqim (video ko'rish) ishlashda davom etsin. Har bog'liqlik uchun fallback strategiyasini yozing.

10. **Chaos experiment.** Ishlab turgan tizimingiz uchun birinchi chaos engineering eksperimentini rejalashtiring: steady-state metrikasi, gipoteza, kiritiladigan nosozlik, blast radius, va muvaffaqiyat mezoni. Nima uchun staging'dan boshlash kerakligini asoslang.

---

← [System Design bo'limiga qaytish](./README.md)
