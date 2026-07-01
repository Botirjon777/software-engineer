# Scalability va Reliability — Masalalar Yechimi (CHUQUR instruksiya)

Bu fayl [`system-design/03-scalability-reliability.md`](../../system-design/03-scalability-reliability.md) dagi "Masalalar" bo'limining namunaviy yechimlarini o'z ichiga oladi. System design'da yagona to'g'ri javob yo'q — quyidagilar **namunaviy yondashuv**; muhimi, qaroringizni trade-off bilan asoslay olishingiz.

## Mundarija

- [1. Nines hisobi](#1-nines-hisobi)
- [2. Circuit breaker dizayni](#2-circuit-breaker-dizayni)
- [3. Retry strategiyasi](#3-retry-strategiyasi)
- [4. SPOF auditi](#4-spof-auditi)
- [5. DR rejasi](#5-dr-rejasi)
- [6. Load shedding](#6-load-shedding)
- [7. Auto-scaling tuning](#7-auto-scaling-tuning)
- [8. Observability rejasi](#8-observability-rejasi)
- [9. Graceful degradation](#9-graceful-degradation)
- [10. Chaos experiment](#10-chaos-experiment)

---

## 1. Nines hisobi

```text
4 ta bog'liq (ketma-ket) komponent, har biri 99.95% = 0.9995

Umumiy = 0.9995^4 = 0.99800... ≈ 99.80%

Yiliga downtime:
  99.95% → 4.38 soat/yil (bitta komponent)
  99.80% → (1 - 0.998) * 8760 soat ≈ 17.5 soat/yil (butun zanjir)
```

**💡 Tushuncha:** Ketma-ket bog'liqlik availability'ni **ko'paytiradi** — har komponent yaxshi bo'lsa ham, zanjir yomonroq. 4 ta 99.95% → 99.80%.

99.99% ga ko'tarish uchun:
- **Bog'liqlikni kamaytir**: kritik yo'ldagi komponentlar sonini qisqartir (kamroq hop).
- **Redundancy qo'sh**: har komponentni parallel dublikat qil. Ikkita parallel 99.95% = `1 - (0.0005)² = 99.999975%`.
- **Failover avtomatlashtir**: MTTR'ni kamaytir (tez tiklanish).

```text
Downtime taqqoslash (yiliga):
  99.80% → 17.5 soat
  99.99% → 52.6 daqiqa
  Farq   → ~20x kam downtime
```

**💡 Tushuncha:** Kritik yo'ldagi har komponentni active-active redundant qilish + health check + avtomatik failover — 99.99% ga yo'l.

---

## 2. Circuit breaker dizayni

```text
Payment gateway circuit breaker:
  threshold  = 5 ketma-ket xato (yoki 50% xato oxirgi 20 so'rovda)
  cooldown   = 30 soniya (OPEN'da kutish)
  half-open  = 3 test so'rov; 2+ muvaffaqiyat → CLOSED, aks holda OPEN
  timeout    = har chaqiruvga 3s (timeout ham xato sanaladi)
```

```text
        5 xato
  CLOSED ────► OPEN ──30s──► HALF-OPEN
    ▲                          │  │
    │ 2/3 success              │  │ fail
    └──────────────────────────┘  └──► OPEN
```

OPEN holatda fallback:
- Sinxron to'lov o'rniga: "to'lov jarayonda, tez orada tasdiqlanadi" + so'rovni **queue**'ga qo'yish (keyin retry).
- Yoki: alternativ payment provider'ga o'tish (agar bor bo'lsa).
- Foydalanuvchiga: "hozir to'lov mumkin emas, keyinroq urinib ko'ring" (fail fast, muzlab qolmasdan).

**⚠️ Ehtiyot bo'l:** To'lovda fallback **ma'lumot yo'qotmasligi** kerak — queue'ga qo'yilgan to'lovlar idempotency key bilan, dublikat bo'lmasin.

---

## 3. Retry strategiyasi

```text
Retryable xatolar:  503, 502, 504, 429, timeout, connection reset
Retry QILMA:        400, 401, 403, 404, 422 (client xatosi — retry befoyda)

Formula:  backoff = min(cap, base * 2^attempt)
          delay   = random(0, backoff)   // full jitter
  base = 200ms, cap = 5s, max_retries = 3, timeout = 3s/urinish

Urinishlar: ~0-0.2s, ~0-0.4s, ~0-0.8s (jitter bilan tarqalgan)
```

Idempotency:
- **GET, PUT, DELETE** — bemalol retry (tabiatan idempotent).
- **POST** — idempotency key qo'sh: `Idempotency-Key: <uuid>`. Server key'ni saqlaydi, takror kelsa oldingi natijani qaytaradi (yangi harakat yo'q).
- Retry'ni **circuit breaker** bilan birlashtir — o'lgan xizmatga retry storm bo'lmasin.

**💡 Tushuncha:** 429 (rate limited) da `Retry-After` header'iga rioya qil — o'zboshimcha backoff emas, server aytgan vaqtni kut.

---

## 4. SPOF auditi

| Qatlam | SPOF | Yechim |
|--------|------|--------|
| DNS | Bitta DNS provayder | Ikki DNS provider (Route53 + Cloudflare), qisqa TTL |
| CDN | Provider uzilishi | Multi-CDN yoki origin fallback |
| LB | Bitta LB instance | Ikkita LB (active-passive) + floating/virtual IP + health check |
| App | Bitta instance | Bir nechta stateless instance ortida LB, auto-scaling |
| Redis | Bitta node | Redis Cluster yoki Sentinel (replica + avtomatik failover) |
| PostgreSQL | Bitta primary | Primary + streaming replica + avtomatik failover (Patroni), multi-AZ |
| S3 | (odatda managed, redundant) | Cross-region replication (backup uchun) |

**💡 Tushuncha:** Stateless qatlamlar (App) oson dublikat bo'ladi. Stateful qatlamlar (DB, Redis) uchun replikatsiya + failover mexanizmi shart. Geografik (multi-AZ/region) redundancy bitta zona falokatiga qarshi.

**⚠️ Ehtiyot bo'l:** DB failover'da split-brain xavfi — quorum yoki fencing kerak (ikki primary bo'lib qolmasin).

---

## 5. DR rejasi

```text
Talab: RTO = 15 daqiqa, RPO = 1 daqiqa (biznes-kritik to'lov)

RPO = 1 daqiqa → deyarli uzluksiz replikatsiya kerak (async lag < 1 min)
RTO = 15 daqiqa → tez failover, katta cluster ko'tarishga vaqt yo'q
```

Tanlov: **Warm standby** (yoki active-active agar byudjet ko'tarsa).

**Nega backup&restore emas:** RTO soatlar — 15 daqiqaga sig'maydi. **Nega pilot light emas:** to'liq kengaytirish 15 daqiqadan oshishi mumkin. **Warm standby** — kichraytirilgan lekin doim ishlaydigan nusxa, failover'da tez kengayadi.

```text
Region A (primary)          Region B (warm standby)
┌──────────────┐            ┌──────────────┐
│ App (full)   │            │ App (minimal)│
│ DB primary   │ ──sync/────│ DB replica   │  (lag < 1 min → RPO ✓)
│              │  async     │              │
└──────────────┘            └──────────────┘
        │                          ▲
        └── falokat → DNS/traffic  │
            B'ga o'tadi (< 15 min → RTO ✓)
```

**💡 Tushuncha:** RPO=1 min uchun DB'da synchronous yoki tez async replikatsiya. RTO=15 min uchun standby oldindan tayyor, faqat DNS/traffic switch va scale-up kerak. Muntazam DR drill (mashq) o'tkazish shart — reja qog'ozda emas, amalda ishlashi kerak.

---

## 6. Load shedding

```text
Signal: queue depth (kutish navbati) + p99 latency + CPU
  (queue depth eng yaxshi — overload'ni erta ko'rsatadi)

Prioritet darajalari:
  P0 (kritik)  → checkout, to'lov          → hech qachon shed qilinmaydi
  P1 (muhim)   → login, mahsulot sahifasi   → oxirgi chozada shed
  P2 (past)    → tavsiya, analytics, banner → birinchi shed qilinadi

Qaror:
  queue > 80% → P2 rad et (503 + Retry-After)
  queue > 90% → P1 ham rad et
  P0 doim o'tadi
```

**💡 Tushuncha:** Load shedding — **reaktiv** (tizim holatiga qarab, real vaqtda). Rate limiting — **proaktiv** (client boshiga oldindan belgilangan kvota, tizim holatidan mustaqil). Ikkalasi birga ishlatiladi: rate limiting adolatli taqsimot, load shedding oxirgi himoya.

**⚠️ Ehtiyot bo'l:** Shed qilingan so'rovga aniq `503 + Retry-After` qaytar (client to'g'ri backoff qilsin), timeout'da muzlatib qo'yma.

---

## 7. Auto-scaling tuning

**Muammo 1 — spike'da kech qolish:**
- CPU metrik reaktiv va kech — o'rniga **RPS** yoki **queue depth** ishlatilsin (yukni erta ko'rsatadi).
- **Buffer capacity** qoldir (masalan 30% bo'sh) — spike'ni yangi instance kelguncha ko'taradi.
- **Scale-out'ni agressiv, scale-in'ni sekin** qil (tez kengay, asta kichray).
- **Predictive/scheduled scaling** — ma'lum peak (masalan aksiya) oldidan oldindan tayyorla.
- Instance **warm-up** vaqtini kamaytir (oldindan tayyor AMI/image, tez start).

**Muammo 2 — flapping (tez oshirib-kamaytirish):**
- **Cooldown** qo'sh (scale amalidan keyin N daqiqa kutish).
- Scale-out va scale-in threshold'lari orasida **hysteresis** (bo'shliq): out 70%, in 30% (bir-biriga yaqin bo'lmasin).
- Metrikani **agregatsiya oynasi** bilan silliqla (bir soniyalik spike'ga reaksiya qilma, 3-5 daqiqa o'rtacha).

```text
scale-out: RPS > 800/instance, 3 min ketma-ket → +2 instance
scale-in:  RPS < 300/instance, 10 min ketma-ket → -1 instance
cooldown:  300s har amaldan keyin
buffer:    doim +30% capacity
```

---

## 8. Observability rejasi

```text
SLI (metrics):
  - Availability: 2xx/3xx so'rovlar foizi
  - Latency: p50, p95, p99 (o'rtacha emas — p99 muhim!)
  - Error rate: 5xx foizi
  - Throughput: RPS
  - Saturation: CPU, memory, connection pool foizi

Logs:  structured (JSON), correlation ID bilan, xato konteksti
Traces: request → service → DB, har hop latency (bottleneck topish)
```

```text
SLO:  99.9% availability, p99 < 300ms
Error budget = 0.1% → oyiga ~43 daqiqa "buzilishga ruxsat"
  budget ichida → risk qilib deploy qilinadi
  budget tugadi → feature to'xtatilib, barqarorlikka o'tiladi
```

"p99 latency oshdi" alarm tekshiruvi:
1. **Metrics**: qaysi endpoint? qachondan? saturation (CPU/pool) oshdimi?
2. **Traces**: sekin so'rovlarni ol — qaysi hop (DB? tashqi API?) sekin?
3. **Logs**: o'sha vaqtdagi xatolar, deploy, migration bormi?
4. Root cause → tuzat (masalan DB indeks, connection pool, downstream circuit breaker).

**💡 Tushuncha:** p99 (99-persentil) o'rtachadan muhimroq — o'rtacha yaxshi bo'lib, foydalanuvchilarning 1% dahshatli tajriba olishi mumkin.

---

## 9. Graceful degradation

```text
Video streaming — bog'liqliklar va fallback:

Xizmat            Kritikmi?  Fallback (o'lsa)
─────────────────────────────────────────────────
Video oqim (CDN)  KRITIK     yo'q — bu asosiy funksiya (redundant CDN)
Auth              KRITIK     cache'langan token, qisqa muddat toqat
Recommendations   yo'q       "top/trending" statik ro'yxat yoki bo'sh
Comments          yo'q       "izohlar vaqtincha mavjud emas"
Analytics/metrics yo'q       lokal buffer'ga yoz, keyin yubor (fire-and-forget)
Thumbnails        yo'q       placeholder rasm
Subtitles         yo'q       "subtitr mavjud emas", video baribir o'ynaydi
```

```text
Video sahifa:
  ✓ video o'ynaydi (kritik yo'l himoyalangan)
  ✗ recommendations → trending fallback
  ✗ comments → xabar ko'rsatiladi
Natija: asosiy tajriba buzilmaydi, faqat qo'shimchalar cheklanadi
```

**💡 Tushuncha:** Kritik yo'lni (video oqim) qo'shimcha xizmatlardan **izolyatsiya** qil (bulkhead, circuit breaker). Analytics o'lishi hech qachon videoni to'xtatmasligi kerak — fire-and-forget bilan.

---

## 10. Chaos experiment

```text
1. STEADY STATE (normal metrika):
   availability 99.95%, p99 = 250ms, error rate < 0.1%

2. GIPOTEZA:
   "Bitta app instance o'chsa, availability va latency o'zgarmaydi
    (LB health check trafikni qolganlarga yo'naltiradi)."

3. NOSOZLIK:
   Tasodifiy bitta app instance'ni to'xtat (Chaos Monkey uslubi).

4. BLAST RADIUS:
   - Faqat 1 instance (butun pool emas)
   - Ish vaqti (peak emas), jamoa kuzatib turadi
   - "Abort" tugmasi tayyor (metrika buzilsa darrov to'xtat)

5. MUVAFFAQIYAT MEZONI:
   availability > 99.9%, p99 < 350ms, error rate < 0.5% saqlangan.
   Agar buzilsa → zaiflik topildi (masalan health check sekin,
   yoki qolgan instance'lar yukni ko'tarolmadi → auto-scaling tuning kerak).
```

**Nega staging'dan boshlash:** Birinchi marta xatti-harakat noma'lum — real foydalanuvchilarga ta'sir qilmasligi uchun avval staging'da sina. Ishonch hosil bo'lgach, kichik blast radius bilan production'ga o't. Chaos'ning maqsadi — nazorat ostida zaiflikni topish, tasodifiy falokat emas.

**💡 Tushuncha:** Har eksperimentdan keyin topilgan zaiflikni tuzat va uni **regressiya testi** sifatida avtomatlashtir (har deploy'da qayta o'tkaz). Chaos — bir martalik emas, doimiy amaliyot.

---

← [System Design bo'limiga qaytish](../../system-design/README.md)
