# Proxy, Gateway va Load Balancing — Yechimlar

Bu yerda [`05-proxy-load-balancing.md`](../../networking/05-proxy-load-balancing.md) bo'limidagi masalalarning to'liq yechimlari keltirilgan. Avval o'zingiz urinib ko'ring, keyin solishtiring.

## Mundarija

- [1-masala: Forward yoki reverse](#1-masala)
- [2-masala: LB algoritmi tanlash](#2-masala)
- [3-masala: L4 yoki L7](#3-masala)
- [4-masala: Health check loyihasi](#4-masala)
- [5-masala: Sticky session muammosi](#5-masala)
- [6-masala: Consistent hashing](#6-masala)
- [7-masala: API Gateway dizayni](#7-masala)
- [8-masala: CDN strategiyasi](#8-masala)
- [9-masala: To'liq arxitektura](#9-masala)
- [10-masala: Nosozlik tahlili](#10-masala)

---

## 1-masala

**Forward yoki reverse?**

| Holat | Tur | Sabab |
|-------|-----|-------|
| (a) maktabda o'yin saytlarini bloklash | **Forward proxy** | Client tomonida turadi, chiqayotgan trafikni filtrlaydi. Client (o'quvchi) proxy orqali chiqishga majbur, proxy bloklash ro'yxatini qo'llaydi. |
| (b) 5 backend orasida yuk taqsimlash | **Reverse proxy** (LB) | Server tomonida, kelayotgan so'rovlarni backendlar orasida taqsimlaydi. |
| (c) xodimlar trafigini bitta IP orqali chiqarish | **Forward proxy** | Clientlarni yashiradi, hammasi bitta tashqi IP (proxy IP)dan ko'rinadi (anonimlik/NAT). |
| (d) HTTPS sertifikatni bitta joyda boshqarish | **Reverse proxy** | TLS termination — sertifikat reverse proxy'da markazlashtiriladi, backendlar oddiy HTTP ishlaydi. |

**Asosiy printsip:** chiquvchi (client → internet) trafik → forward; kiruvchi (internet → server) trafik → reverse.

---

## 2-masala

**LB algoritmi tanlash.**

| Senariy | Algoritm | Asos |
|---------|----------|------|
| (a) bir xil quvvatdagi stateless serverlar | **Round Robin** | Eng sodda, serverlar teng bo'lgani uchun teng taqsimlash adolatli. |
| (b) ba'zilari 2x kuchli | **Weighted Round Robin** | Kuchli serverga weight=2 berib, ko'proq so'rov yuboriladi. |
| (c) so'rovlar davomiyligi juda turlicha | **Least Connections** | Uzun so'rovlar bir serverni band qilsa, yangi so'rov eng bo'sh (kam ulanishli) serverga boradi — round robin bu yerda nomutanosib yuk hosil qiladi. |
| (d) distributed cache key taqsimoti | **Consistent Hashing** | Server qo'shilganda/o'chirilganda minimal qayta taqsimlash — cache locality saqlanadi, cache storm bo'lmaydi. |

**Eslatma:** Stateless bo'lsa round robin / least connections odatda yetarli. Cache locality yoki sticky kerak bo'lsagina hashing.

---

## 3-masala

**L4 yoki L7?**

Bu **L7 (application layer)** load balancing talab qiladi, chunki marshrutlash URL path'ga (`/api/`, `/static/`) qarab bo'ladi va TLS termination kerak — bularning hammasi HTTP qatlamida ishlaydi. L4 faqat IP/portni ko'radi, path'ni bilmaydi.

```nginx
upstream api_pool   { server 10.0.0.21:8080; server 10.0.0.22:8080; }
upstream web_pool   { server 10.0.0.31:8080; server 10.0.0.32:8080; }

server {
    listen 443 ssl;                       # TLS termination shu yerda
    ssl_certificate     /etc/ssl/site.crt;
    ssl_certificate_key /etc/ssl/site.key;

    location /api/ {
        proxy_pass http://api_pool;       # API so'rovlari -> API pool
    }

    location /static/ {
        proxy_pass http://cdn_origin;     # statik -> CDN origin
        proxy_cache static_cache;
    }

    location / {
        proxy_pass http://web_pool;       # qolgani -> web pool
    }
}
```

---

## 4-masala

**Health check loyihasi.**

Backend Postgres va Redis'ga bog'liq bo'lgani uchun ikki xil endpoint kerak:

- **`/health` (liveness):** "Process tirikmi?" — faqat ilovaning o'zi javob bera olishini tekshiradi. Tashqi bog'liqliklarni tekshirmaydi. Agar muvaffaqiyatsiz bo'lsa — process restart qilinadi (orkestrator, masalan Kubernetes).
- **`/ready` (readiness):** "Trafik qabul qilishga tayyormi?" — Postgres va Redis ulanishlarini ham tekshiradi. Agar DB tushib qolsa, bu `503` qaytaradi.

**LB qaysi birini ishlatadi:** LB **`/ready`** (readiness) ishlatishi kerak. Sababi: agar DB tushib qolgan bo'lsa, process tirik bo'lsa-da, server foydali ishlay olmaydi — LB unga trafik yubormasligi kerak. `/health`'ni faqat liveness uchun ishlatish noto'g'ri, chunki u DB nosozligini ko'rmaydi.

```js
app.get('/health', (req, res) => res.status(200).json({ status: 'alive' }));

app.get('/ready', async (req, res) => {
  const pgOk    = await checkPostgres();
  const redisOk = await checkRedis();
  if (pgOk && redisOk) return res.status(200).json({ status: 'ready' });
  res.status(503).json({ status: 'not_ready', pg: pgOk, redis: redisOk });
});
```

**⚠️ Nozik nuqta:** Readiness'da bog'liqliklarni tekshirish foydali, lekin uni juda "qattiq" qilmaslik kerak — agar Redis vaqtincha sekinlashsa va hamma serverlar bir vaqtda `not_ready` bo'lsa, butun pool trafikdan chiqib, "cascading failure" yuz beradi. Shuning uchun ko'pincha kritik bog'liqliklarnigina (Postgres) readinessga qo'shadilar.

---

## 5-masala

**Sticky session muammosi.**

**Tahlil:** Session in-memory saqlanganidan kelib chiqqan muammo. Sticky session bir clientni doim bir serverga bog'laydi. O'sha server o'lganda, uning xotirasidagi barcha sessionlar yo'qoladi → o'sha foydalanuvchilar boshqa serverga tushadi, lekin u yerda session yo'q → tizimdan chiqib ketishadi. Bu shuningdek yukni notekis taqsimlaydi (eski sessionlar eski serverda to'planadi).

**Stateless yechim (qadamlab):**

1. **Tashqi session store qo'shish:** Redis (yoki DB) o'rnatib, sessionlarni shu yerda saqlash. `session_id` cookie clientda, ma'lumot Redis'da.
2. **Kodni o'zgartirish:** Har bir backend session o'qish/yozishni in-memory'dan Redis'ga ko'chirish (`req.session` → Redis store).
3. **Sticky'ni o'chirish:** LB'da session affinity'ni o'chirib, oddiy round robin'ga o'tish — endi har server har clientga xizmat qiladi.
4. **JWT alternativasi:** Yoki butunlay stateless JWT'ga o'tish — session ma'lumoti imzolangan token ichida clientda saqlanadi, server hech narsa saqlamaydi (lekin token bekor qilish/revocation murakkablashadi).

```text
OLDIN (sticky):                  KEYIN (stateless):
Client -> doim Server 2          Client -> har qanday Server
session Server 2 xotirasida      session -> Redis (umumiy)
Server o'lsa -> session yo'q     Server o'lsa -> boshqasi davom etadi
```

**Natija:** Har qanday server har qanday clientga xizmat qiladi, yuk teng taqsimlanadi, server o'limi sessionlarni buzmaydi.

---

## 6-masala

**Consistent hashing.**

**Oddiy hashing'da (`% 3`):** 4-server qo'shilganda formula `hash(key) % 4` ga o'zgaradi. Endi deyarli **barcha** kalitlarning indeksi o'zgaradi (chunki `% 3` va `% 4` butunlay boshqa natija beradi). Distributed cache'da bu — deyarli barcha kalitlar yangi serverga "ko'chadi" → eski keshlar foydasiz → barcha so'rovlar cache miss → originga zarba (**cache storm / thundering herd**).

**Consistent hashing buni qanday yaxshilaydi:** Serverlar va kalitlar bir xil hash makonida halqa (ring) bo'ylab joylashtiriladi. Har bir kalit halqada o'zidan keyingi serverga tegishli. Yangi server qo'shilganda, faqat **uning halqadagi pozitsiyasi yonidagi** kalitlar (taxminan 1/N qism, ya'ni 4 serverda ~1/4) ko'chadi — qolganlari joyida qoladi.

```text
Oddiy:  4-server qo'shildi -> ~100% kalit ko'chadi  (yomon)
Consistent: 4-server qo'shildi -> ~25% kalit ko'chadi (yaxshi)
```

**Virtual node nima uchun kerak:** Faqat N ta server N nuqta sifatida halqaga qo'yilsa, taqsimot **notekis** bo'ladi — ba'zi serverga ko'p, ba'zisiga kam kalit tushadi (halqada bo'sh joylar teng emas). Yechim: har bir fizik serverni halqaga **ko'p nuqta** (virtual node, masalan 100-200 ta) sifatida joylashtirish. Shunda kalitlar serverlar orasiga statistik teng taqsimlanadi va server o'chganda uning yuki qolganlariga teng bo'linadi.

```text
Virtual node'siz:  Server A ████████  Server B ██  Server C ████  (notekis)
Virtual node bilan: har biri halqada ~150 nuqta -> teng taqsimot
```

---

## 7-masala

**API Gateway dizayni.**

**Routing jadvali:**

```text
/users/*         -> User Service
/orders/*        -> Order Service
/payments/*      -> Payment Service
/notifications/* -> Notification Service
```

**Cross-cutting concern'lar taqsimoti:**

| Concern | Qayerda | Sabab |
|---------|---------|-------|
| Authentication (JWT tekshirish) | **Gateway** | Bir joyda, har xizmat takrorlamaydi. Gateway tokenni tekshirib, foydalanuvchi ID'sini ishonchli header (`X-User-Id`) bilan downstream'ga uzatadi. |
| Rate limiting | **Gateway** | Tashqi abuse'ni eng chetda to'xtatish kerak. |
| Routing | **Gateway** | Gatewayning asosiy vazifasi. |
| TLS termination | **Gateway** (yoki oldidagi LB) | Markazlashtirilgan. |
| Logging / tracing (correlation ID) | **Gateway** boshlaydi | Gateway trace ID hosil qiladi, xizmatlar davom ettiradi. |
| **Authorization** (biznes ruxsatlar) | **Servislar** | "Bu foydalanuvchi shu buyurtmani ko'ra oladimi?" — bu biznes mantiq, gateway buni bilmaydi. |
| Biznes validatsiya, ma'lumotlar | **Servislar** | Domen mantig'i xizmatlarda qoladi. |

**JWT auth qayerda tekshiriladi:** Tokenning **imzosi va amal qilishi (signature, expiry)** — **Gateway**'da (markazlashtirilgan, tez rad etish). Lekin **fine-grained authorization** (kim nimaga ruxsatli) — **servislarda**, chunki bu biznes mantiqqa bog'liq. "Authentication gateway'da, authorization servisda" — yaxshi qoida. Servislar gateway'ga ishonadi, lekin ichki tarmoq himoyalangan bo'lishi yoki servislar tokenni qayta tekshirishi kerak (zero-trust).

---

## 8-masala

**CDN strategiyasi.**

| Kontent turi | CDN'da keshlash? | TTL | Izoh |
|--------------|------------------|-----|------|
| Statik rasmlar (logo, ikonlar) | **Ha** | Uzoq (30 kun+) | Kamdan-kam o'zgaradi. Fingerprint nom (`logo.a3f9.png`) bilan. |
| Foydalanuvchi profil rasmlari | **Ha, lekin qisqa TTL** | Qisqa (5-60 min) yoki `Cache-Control: private` | Tez o'zgaradi. Yoki versiyalangan URL (`avatar?v=12`). |
| Dinamik HTML | **Odatda yo'q** | `no-cache` yoki juda qisqa | Foydalanuvchiga xos, doim yangi. (Lekin edge-side rendering yoki micro-caching mumkin.) |

**TTL belgilash printsipi:** Kontent qancha kam o'zgarsa, TTL shuncha uzoq. O'zgaruvchan kontent uchun **versiyalangan URL** (content hash yoki version query) ishlatib, TTL'ni cheksiz qilish va o'zgarganda URL'ni almashtirish — eng yaxshi usul (immutable caching).

**Cache invalidation muammosi va yechimlari:**

1. **Versioned/fingerprinted URL (eng yaxshi):** Fayl o'zgarsa nomi o'zgaradi (`app.v2.js`). Eski URL hech qachon o'zgarmaydi, shuning uchun TTL cheksiz bo'lishi mumkin. Invalidation kerak emas.
2. **Purge/invalidation API:** CDN'ga "bu URL'ni keshdan o'chir" buyrug'i (Cloudflare purge). Sekin tarqaladi, hamma edge'da bir vaqtda emas.
3. **Qisqa TTL:** Oddiy, lekin originга ko'proq yuk.

**Tavsiya:** Statik aktivlar uchun fingerprinted URL + uzoq TTL; dinamik kontent uchun `Cache-Control` bilan boshqarish.

---

## 9-masala

**To'liq arxitektura (O'zbekiston e-commerce, Black Friday).**

```text
          [Foydalanuvchilar: Toshkent / Samarqand / Andijon]
                              │
                              ▼
                  ┌───────────────────────┐
                  │          CDN          │  statik (rasm, JS, CSS),
                  │  (edge: yaqin POP'lar)│  geo-routing, DDoS himoya,
                  └───────────┬───────────┘  WAF, cache hit -> originga kam yuk
                              │ (dinamik so'rovlar)
                              ▼
                  ┌───────────────────────┐
                  │    Load Balancer (L7) │  TLS termination, health check,
                  │   (nginx/HAProxy/ALB) │  round robin/least conn, auto-scaling
                  └───────────┬───────────┘
                              ▼
                  ┌───────────────────────┐
                  │      API Gateway      │  auth (JWT), rate limit (Black Friday!),
                  │     (Kong/Envoy)      │  routing, aggregation
                  └───────────┬───────────┘
              ┌───────────────┼───────────────┐
              ▼               ▼               ▼
       [Catalog Svc]   [Cart/Order Svc]  [Payment Svc]   <- stateless, auto-scale
              │               │               │
              ▼               ▼               ▼
       [Read replica]   [Primary DB]    [Payment GW]
              └──────── Redis cache ─────────┘
                    (session, hot data, rate-limit counter)
```

**Qatlam tanlovlari asoslari:**

- **CDN:** Foydalanuvchilar O'zbekiston ichida, lekin origin uzoqroq (masalan, Yevropa DC) bo'lishi mumkin. CDN statik kontentni yaqin edge'dan berib latency'ni kamaytiradi va Black Friday DDoS/yuk cho'qqisini yutadi.
- **L7 LB:** Path-based routing va TLS termination kerak; auto-scaling bilan integratsiya — yuk oshganda yangi backendlar avtomatik qo'shiladi.
- **API Gateway:** Black Friday'da **rate limiting** kritik (bot va abuse'dan himoya). Auth markazlashtirilgan.
- **Stateless servislar + Redis:** Sticky session yo'q, session Redis'da — har qanday instansiya har clientga xizmat qiladi, horizontal scaling oson.
- **DB read replica:** O'qish-og'ir e-commerce'da (mahsulot ko'rish) read replica'lar yukni bo'ladi; primary faqat yozish uchun.
- **Caching:** Mahsulot kataloglari hot data sifatida Redis'da — DB'ga zarbani kamaytiradi.

---

## 10-masala

**Nosozlik tahlili: "ba'zida login, ba'zida chiqib ketadi".**

**Eng ehtimoliy sabab:** Session in-memory saqlanmoqda va LB round-robin bilan har so'rovni boshqa backendga yuboryapti. Foydalanuvchi Server 1'da login qiladi (session faqat Server 1 xotirasida), keyingi so'rov Server 2'ga tushadi — u yerda session yo'q — chiqib ketgan deb ko'rinadi. "Ba'zida" — chunki round-robin ba'zan o'sha serverga, ba'zan boshqasiga qaytaradi.

**Diagnostika qadamlari:**

1. **Session storage'ni tekshirish:** Session in-memory'mi yoki tashqi store (Redis)'da? In-memory bo'lsa — sabab tasdiqlandi.
2. **LB rejimini tekshirish:** Sticky session yoqilganmi? Yo'q (round robin) bo'lsa, har so'rov boshqa serverga ketishi mumkin.
3. **Loglarni solishtirish:** Bir foydalanuvchi so'rovlari turli backendlarga tarqalayotganini access log'dan ko'rish (correlation ID bilan).
4. **Cookie domeni/secure flag:** Cookie barcha backendlarda bir xil ko'rinadimi (domain, path to'g'rimi)?

**Yechim variantlari:**

- **Eng yaxshi:** Session'ni Redis/DB'ga ko'chirib, backendni stateless qilish (5-masaladagi yondashuv). Round robin saqlanadi, muammo yo'qoladi.
- **Tezkor (vaqtinchalik):** LB'da sticky session yoqish — bir clientni bir serverga bog'lash. Lekin bu anti-pattern (yuk notekis, server o'lsa session yo'qoladi).
- **Alternativa:** Stateless JWT'ga o'tish — session ma'lumoti tokenda, server saqlamaydi.

**Tavsiya:** Vaqtinchalik sticky bilan to'xtatib turish mumkin, lekin asl yechim — stateless arxitektura (tashqi session store).

---

← [Networking bo'limiga qaytish](../../networking/README.md)
