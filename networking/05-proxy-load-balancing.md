# Proxy, Gateway va Load Balancing

Zamonaviy backend arxitekturada client va server o'rtasida hech qachon to'g'ridan-to'g'ri "yalang'och" ulanish bo'lmaydi. Trafik **proxy**, **load balancer**, **API gateway** va **CDN** kabi qatlamlardan o'tadi. Bu komponentlar xavfsizlik (security), masshtablanish (scalability), ishonchlilik (reliability) va tezlik (performance) uchun javobgar. Bu bo'limda har bir qatlam nima qilishini, ular qanday farq qilishini va bitta arxitekturada qanday joylashishini ko'rib chiqamiz.

**💡 Tushuncha:** Bu komponentlarning hammasi mohiyatan **"o'rtadagi vositachi" (intermediary)**. Farqi — ular *kim tomonda* turishi (client yoki server) va *qaysi qatlamda* (L4 transport yoki L7 application) ishlashida.

## Mundarija

- [Proxy nima](#proxy-nima)
- [Forward Proxy](#forward-proxy)
- [Reverse Proxy](#reverse-proxy)
- [Forward vs Reverse Proxy](#forward-vs-reverse-proxy)
- [Load Balancer nima va nega kerak](#load-balancer-nima-va-nega-kerak)
- [L4 vs L7 Load Balancing](#l4-vs-l7-load-balancing)
- [Load Balancing algoritmlari](#load-balancing-algoritmlari)
- [Health Check](#health-check)
- [Sticky Session](#sticky-session)
- [API Gateway](#api-gateway)
- [CDN nima va qanday ishlaydi](#cdn-nima-va-qanday-ishlaydi)
- [Hammasi bitta arxitekturada](#hammasi-bitta-arxitekturada)
- [Savol-Javoblar](#savol-javoblar)
- [Masalalar](#masalalar)

---

## Proxy nima

**Proxy** — bu client va server o'rtasida turuvchi va so'rovlarni (request) ularning nomidan uzatuvchi vositachi server. So'rov to'g'ridan-to'g'ri manzilga bormaydi, balki avval proxy'ga keladi, proxy esa uni keyingi nuqtaga yo'naltiradi.

**💡 Tushuncha:** "Proxy" so'zining o'zi "vakil", "ishonchli shaxs" degani — ya'ni sizning nomingizdan harakat qiluvchi.

Ikki asosiy tur bor:
- **Forward proxy** — **client** tomonida turadi, clientlar nomidan tashqariga so'rov yuboradi.
- **Reverse proxy** — **server** tomonida turadi, serverlar nomidan tashqaridan so'rovlarni qabul qiladi.

Farqni eslab qolishning oson yo'li: proxy *kimni yashiradi*? Forward proxy clientni yashiradi, reverse proxy serverni yashiradi.

---

## Forward Proxy

**Forward proxy** client guruhi va internet o'rtasida turadi. Clientlar internetga to'g'ridan-to'g'ri emas, balki proxy orqali chiqadi.

```text
[Client A] ─┐
[Client B] ─┼──> [Forward Proxy] ──> Internet ──> [Server]
[Client C] ─┘         (client tomonda)
```

**Vazifalari:**

1. **Anonimlik (anonymity):** Server faqat proxy'ning IP'sini ko'radi, asl clientning IP'sini emas. VPN va Tor shu printsipda ishlaydi.
2. **Filtering / access control:** Korxona tarmog'ida ba'zi saytlarni bloklash (masalan, ishlash vaqtida ijtimoiy tarmoqlar). Maktab/ofis tarmoqlari shunday qiladi.
3. **Caching:** Ko'p so'raladigan resurslarni proxy'da saqlab, bir xil kontentni qayta-qayta internetdan tortmaslik — bandwidth tejaladi.
4. **Logging / monitoring:** Tashkilot xodimlari qanday saytlarga kirayotganini kuzatish.

**💡 Tushuncha:** Forward proxy'da *client uni bilib* sozlanadi (brauzer yoki OS proxy sozlamasi). Server esa proxy borligini bilmaydi.

---

## Reverse Proxy

**Reverse proxy** internet va backend serverlar o'rtasida, **server tomonida** turadi. Tashqi clientlar reverse proxy bilan gaplashadi va orqada nechta server borligini bilmaydi.

```text
                          ┌──> [App Server 1]
Internet ──> [Reverse ────┼──> [App Server 2]
            Proxy/nginx]  └──> [App Server 3]
            (server tomonda)
```

**nginx misoli:**

```nginx
server {
    listen 443 ssl;
    server_name example.uz;

    ssl_certificate     /etc/ssl/example.crt;
    ssl_certificate_key /etc/ssl/example.key;

    # gzip compression
    gzip on;
    gzip_types text/plain application/json application/javascript;

    location /api/ {
        proxy_pass http://backend_pool;          # orqadagi serverlarga uzatish
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;  # asl client IP'ni uzatish
    }

    location /static/ {
        root /var/www;        # statik fayllarni o'zi beradi
        expires 30d;          # caching
    }
}

upstream backend_pool {
    server 10.0.0.11:8080;
    server 10.0.0.12:8080;
}
```

**Reverse proxy vazifalari:**

| Vazifa | Tushuntirish |
|--------|--------------|
| **TLS termination** | HTTPS'ni proxy'da ochib (decrypt), backendga oddiy HTTP uzatadi. Backend TLS bilan shug'ullanmaydi → CPU tejaladi, sertifikat bir joyda. |
| **Caching** | Tez-tez so'raladigan javoblarni keshda saqlab, backendga yukni kamaytiradi. |
| **Compression** | Javoblarni gzip/brotli bilan siqib, trafikni kamaytiradi. |
| **Load balancing** | So'rovlarni bir nechta backend orasida taqsimlaydi (ko'p reverse proxy LB ham bo'ladi). |
| **Security** | Backendni to'g'ridan-to'g'ri internetga ochmaydi; WAF, rate limiting, DDoS himoyasi. |
| **Static serving** | Statik fayllarni (rasm, JS, CSS) backendga bormay o'zi beradi. |

**💡 Tushuncha:** nginx, HAProxy, Envoy, Traefik — eng mashhur reverse proxy'lar. Caddy ham (avtomatik TLS bilan).

---

## Forward vs Reverse Proxy

```text
FORWARD PROXY (client tomonda)        REVERSE PROXY (server tomonda)

[Clients] ─> [Proxy] ─> [Internet]    [Internet] ─> [Proxy] ─> [Servers]
              │                                       │
   clientni yashiradi                      serverni yashiradi
   client uni sozlaydi                     client uni ko'rmaydi
   "men kimga chiqyapman?"                 "men kimga so'rov yuborilyapti?"
```

| Xususiyat | Forward Proxy | Reverse Proxy |
|-----------|---------------|---------------|
| Tomoni | Client | Server |
| Nimani yashiradi | Clientni (server clientni ko'rmaydi) | Serverni (client backendni ko'rmaydi) |
| Kim sozlaydi | Client (brauzer/OS) | Server admini |
| Asosiy maqsad | Anonimlik, filtering, caching | Load balancing, TLS termination, security |
| Misol | VPN, korporativ proxy, Tor | nginx, HAProxy, Cloudflare |

**⚠️ Ehtiyot bo'l:** Intervyuda eng ko'p adashtiradigan joy shu. Esda tut: **forward = client uchun**, **reverse = server uchun**. "Reverse" so'zi "teskari tomonga" ishlatilishini bildiradi.

---

## Load Balancer nima va nega kerak

**Load balancer (LB)** — kelayotgan so'rovlarni bir nechta backend server orasida taqsimlovchi komponent. Maqsadi — yukni teng bo'lib, hech bir serverni ortiqcha yuklab yubormaslik.

**Nega kerak:**

1. **Scalability (masshtablanish):** Bitta server cheklangan. Trafik o'ssa, ko'p server qo'shib (horizontal scaling), LB ular orasida taqsimlaydi.
2. **High availability (yuqori ishonchlilik):** Bir server o'lsa, LB uni health check orqali aniqlab, trafikni faqat tirik serverlarga yuboradi. Foydalanuvchi buzilishni sezmaydi.
3. **Maintenance:** Bitta serverni yangilash uchun uni LB'dan chiqarib, qolganlari ishlab turaveradi (rolling deployment).

```text
                 ┌──> [Server 1]  (sog'lom)
[Clients] ──> [LB]──> [Server 2]  (sog'lom)
                 └──> [Server 3]  (o'lik — LB yubormaydi)
```

**💡 Tushuncha:** Reverse proxy ko'pincha LB vazifasini ham bajaradi, lekin tushuncha sifatida ular farqlanadi: reverse proxy "vositachilik", load balancer "taqsimlash" haqida.

---

## L4 vs L7 Load Balancing

Load balancing OSI modelining qaysi qatlamida ishlashiga qarab ikkiga bo'linadi.

**L4 (Transport layer) load balancing:**
- TCP/UDP darajasida ishlaydi. Faqat **IP va port**ni ko'radi.
- Paket ichidagi mazmunni (HTTP header, URL) **ko'rmaydi**.
- Ulanishni shunchaki backendga yo'naltiradi (forwarding).
- Juda tez, kam CPU. Lekin "aqlsiz" — kontentga qarab qaror qabul qila olmaydi.

**L7 (Application layer) load balancing:**
- HTTP/HTTPS darajasida ishlaydi. URL path, header, cookie, method — hammasini ko'radi.
- Kontentga qarab marshrutlash (routing): `/api/*` → API serverlar, `/img/*` → rasm serverlar.
- TLS termination, caching, compression — hammasi shu yerda.
- "Aqlli", lekin sekinroq (paketni to'liq ochib o'qishi kerak).

```text
L4 LB:                              L7 LB:
ko'radi: IP + port                  ko'radi: URL, header, cookie, body
"10.0.0.5:443 ga ulanish bor,       "/api/users ga GET so'rov bor,
 backendga uzat"                     API pool'ga yubor"
TEZ, AQLSIZ                          SEKINROQ, AQLLI
```

| | L4 | L7 |
|---|----|----|
| Qatlam | Transport (TCP/UDP) | Application (HTTP) |
| Ko'radi | IP, port | URL, header, cookie, body |
| Content-based routing | Yo'q | Ha |
| TLS termination | Yo'q (passthrough) | Ha |
| Tezlik | Juda tez | Sekinroq |
| Misol | AWS NLB, HAProxy (tcp mode) | AWS ALB, nginx, Envoy |

**⚠️ Ehtiyot bo'l:** "Path'ga qarab marshrutlash kerak" desa — bu **L7**. "Faqat tez TCP forwarding kerak, kontent ahamiyatsiz" desa — **L4**.

---

## Load Balancing algoritmlari

Load balancer so'rovni *qaysi* serverga yuborishni qanday hal qiladi?

**1. Round Robin** — navbat bilan: 1, 2, 3, 1, 2, 3... Eng sodda. Serverlar bir xil quvvatda bo'lsa yaxshi.

```text
So'rov 1 -> Server A
So'rov 2 -> Server B
So'rov 3 -> Server C
So'rov 4 -> Server A   (aylanib qaytadi)
```

**2. Weighted Round Robin** — har serverga "og'irlik" (weight) beriladi. Kuchli server ko'proq so'rov oladi.

```text
Server A (weight=3), Server B (weight=1)
-> A, A, A, B, A, A, A, B ...
```

**3. Least Connections** — ayni paytda eng kam aktiv ulanishi bor serverga yuboradi. So'rovlar turli davom etadigan bo'lsa (ba'zilari uzoq) yaxshi.

**4. IP Hash** — client IP'sini hash qilib, doim bir xil serverga yuboradi. Bir client doim bir serverga tushadi (sticky'ning bir ko'rinishi).

```text
hash(client_IP) % server_count = server_index
1.2.3.4 -> doim Server B
```

**5. Consistent Hashing** — server qo'shilganda/chiqarilganda *minimal* qayta taqsimlash bo'ladigan maxsus hashing. Oddiy `% N` da bitta server qo'shilsa, deyarli hamma kalit boshqa serverga ko'chadi; consistent hashingda faqat kichik qismi ko'chadi. Distributed cache (Redis cluster, Memcached) va sharding'da muhim.

```text
Oddiy hash (% N):  server qo'shilsa -> deyarli HAMMA kalit ko'chadi (yomon)
Consistent hash:   server qo'shilsa -> faqat 1/N qism kalit ko'chadi (yaxshi)

   Halqa (ring) bo'ylab serverlar va kalitlar joylashtiriladi:
        [S1]
      /      \
   key3      [S2]
     |        |
   [S3]──────key1
   yangi server faqat o'z yonidagi kalitlarni oladi
```

**💡 Tushuncha:** Stateless backendlar uchun round robin / least connections yetarli. State yoki cache locality muhim bo'lsa — IP hash yoki consistent hashing.

---

## Health Check

**Health check** — LB har bir backend serverning "tirik"ligini muntazam tekshirishi. O'lik serverga so'rov yubormaslik uchun zarur.

- **Active health check:** LB o'zi vaqti-vaqti bilan serverga so'rov yuboradi (masalan, har 5 soniyada `GET /health`). Javob bermasa yoki 500 qaytarsa — serverni "unhealthy" deb belgilab, trafikni to'xtatadi.
- **Passive health check:** LB oddiy so'rovlardagi xatolarni kuzatadi. Server ketma-ket xato bersa — uni chetlatadi.

```text
LB ──GET /health──> Server 1  ──> 200 OK   (sog'lom, trafik beriladi)
LB ──GET /health──> Server 2  ──> timeout  (o'lik, chetlatiladi)
```

```js
// Backenddagi health endpoint misoli (Node.js)
app.get('/health', (req, res) => {
  const dbOk = checkDatabaseConnection();
  if (dbOk) res.status(200).json({ status: 'ok' });
  else res.status(503).json({ status: 'db_down' });
});
```

**⚠️ Ehtiyot bo'l:** Health check faqat "process tirikmi" emas, "haqiqatan xizmat ko'rsata oladimi" ni tekshirishi kerak (DB, downstream xizmatlar). Aks holda LB tirik, lekin foydasiz serverga trafik yuboraveradi.

---

## Sticky Session

**Sticky session (session affinity)** — bir client doim bir xil backend serverga yo'naltirilishi. Odatda cookie yoki IP hash orqali.

**Nega kerak bo'lishi mumkin:** Agar session ma'lumoti server xotirasida (in-memory) saqlansa, client boshqa serverga tushsa, o'z sessionini topa olmaydi (tizimdan chiqib ketadi).

```text
Client X ─cookie: srv=2─> [LB] ─> doim Server 2
```

**⚠️ Ehtiyot bo'l:** Sticky session — "anti-pattern" hisoblanadi. U scalability'ni buzadi (yuk teng taqsimlanmaydi) va server o'lsa, o'sha clientlarning sessioni yo'qoladi. **To'g'ri yechim:** session'ni tashqi store'ga (Redis, DB) chiqarib, backendni **stateless** qilish. Shunda har qanday server har qanday clientga xizmat qiladi.

---

## API Gateway

**API Gateway** — barcha client so'rovlari uchun yagona kirish nuqtasi (single entry point), ayniqsa microservice arxitekturada. U so'rovni kerakli mikroservisga yo'naltiradi va umumiy vazifalarni (cross-cutting concerns) bir joyda bajaradi.

**Vazifalari:**

1. **Routing:** `/users/*` → User Service, `/orders/*` → Order Service.
2. **Authentication / Authorization:** Token (JWT) tekshirish bir joyda — har bir mikroservis o'zi qilmaydi.
3. **Rate limiting:** Bir clientdan kelayotgan so'rovlar sonini cheklash (abuse'dan himoya).
4. **Request aggregation:** Bir client so'rovi uchun bir nechta mikroservisni chaqirib, javoblarni birlashtirib qaytarish (mobil ilovalar uchun foydali — kam round trip).
5. **Protocol translation, logging, monitoring, caching.**

```text
                    ┌──/users──> [User Service]
[Client] ──> [API ──┼──/orders─> [Order Service]
            Gateway] └──/pay────> [Payment Service]
            (auth, rate limit, routing bir joyda)
```

**API Gateway vs Reverse Proxy farqi:**

Har ikkalasi ham "server tomonda turuvchi vositachi", lekin abstraksiya darajasi farq qiladi:

| | Reverse Proxy | API Gateway |
|---|---------------|-------------|
| Fokus | Tarmoq darajasidagi vositachilik (TLS, LB, caching) | API/biznes darajasidagi boshqaruv |
| Vazifa | Trafikni uzatish, taqsimlash | Auth, rate limit, aggregation, API versioning |
| Kim biladi | HTTP haqida biladi | API'larning ma'nosini biladi |
| Misol | nginx, HAProxy | Kong, AWS API Gateway, Apigee |

**💡 Tushuncha:** API Gateway — bu "aqlli reverse proxy"ning yuqori darajadagi ko'rinishi. Aslida ko'p gateway'lar (Kong) ichida nginx/Envoy ishlaydi. Reverse proxy "qanday uzatish", gateway "nima qilib uzatish" haqida.

---

## CDN nima va qanday ishlaydi

**CDN (Content Delivery Network)** — dunyo bo'ylab geografik tarqalgan serverlar (edge servers) tarmog'i. Maqsad — kontentni foydalanuvchiga jismonan yaqin joydan yetkazib, latency'ni kamaytirish.

**Muammo:** Serveringiz Frankfurt'da, foydalanuvchi Toshkentda. Har bir rasm/JS uchun ming kilometrlik yo'l — sekin.

**Yechim:** Kontentni Toshkentdagi (yoki yaqin) edge serverda kesh qilib, undan beriladi.

```text
                 ┌─ [Edge: Toshkent] <── O'zbekistondagi userlar
[Origin Server] ─┼─ [Edge: Frankfurt] <── Yevropa userlari
   (asl manba)   └─ [Edge: Singapur]  <── Osiyo userlari

Edge'da kontent bo'lsa -> tezda beradi (cache HIT)
Edge'da yo'q bo'lsa    -> origindan oladi, kesh qiladi (cache MISS), keyin beradi
```

**Qanday ishlaydi:**

1. **Geo-routing (DNS/anycast):** Foydalanuvchi domenga so'rov yuborganda, CDN uni eng yaqin edge serverga yo'naltiradi (geografik yaqinlikka qarab).
2. **Edge caching:** Edge server statik kontentni (rasm, video, CSS, JS) keshda saqlaydi. Birinchi so'rov origindan keladi (cache miss), keyingilar edge'dan (cache hit).
3. **Origin:** Asl server. Edge'da kontent bo'lmasa yoki eskirsa (TTL tugasa), origindan yangilab oladi.

**Foydasi:** Past latency, originga kam yuk, DDoS himoyasi, yuqori availability.

**💡 Tushuncha:** CDN asosan **statik** kontent uchun ideal. Lekin zamonaviy CDN'lar (Cloudflare, Fastly) edge'da kod ham ishlatadi (edge computing) va dinamik kontentni ham keshlaydi. Cloudflare, Akamai, Fastly, AWS CloudFront — mashhur misollar.

---

## Hammasi bitta arxitekturada

Endi barcha qatlamlarni bitta katta arxitekturada ko'raylik. Trafik tashqaridan ichkariga qanday yo'l bosadi:

```text
   [Foydalanuvchi brauzeri]
            │  (forward proxy/VPN bo'lishi mumkin — client tomonda)
            ▼
   ┌─────────────────┐
   │       CDN       │  <- statik kontent edge'dan, geo-routing, DDoS himoya
   │  (edge servers) │
   └────────┬────────┘
            │  (dinamik so'rovlar origin'ga)
            ▼
   ┌─────────────────┐
   │   Load Balancer │  <- L4/L7, so'rovni taqsimlash, health check, TLS termination
   └────────┬────────┘
            ▼
   ┌─────────────────┐
   │   API Gateway   │  <- auth, rate limit, routing, aggregation
   └────────┬────────┘
       ┌────┼────┐
       ▼    ▼    ▼
   [Svc A][Svc B][Svc C]   <- mikroservislar (stateless)
       │
       ▼
   [DB / Cache / Queue]
```

**Qatlamlarning roli:**

| Qatlam | Tomoni | Asosiy roli |
|--------|--------|-------------|
| Forward proxy / VPN | Client | Anonimlik, filtering (ixtiyoriy) |
| CDN | Edge (global) | Statik kontent, geo-yaqinlik, DDoS |
| Load balancer | Server | Yuk taqsimlash, HA, health check |
| API Gateway | Server | Auth, rate limit, routing |
| Reverse proxy | Server | TLS, caching (ko'pincha LB/gateway ichida) |

**⚠️ Ehtiyot bo'l:** Real arxitekturada bu qatlamlar tez-tez **birlashtiriladi**. Masalan, nginx bir vaqtning o'zida reverse proxy + L7 LB + TLS termination + caching bo'lishi mumkin. Cloudflare esa CDN + WAF + LB + DDoS himoyani birlashtiradi. Intervyuda muhimi — har qatlamning *roli*ni tushunish, ularni alohida qutilar deb yodlash emas.

---

## Savol-Javoblar

### ❓ Proxy nima va u qanday muammoni hal qiladi?

**✅ Javob:** Proxy — client va server o'rtasida turib, so'rovlarni ularning nomidan uzatuvchi vositachi. To'g'ridan-to'g'ri ulanish o'rniga trafik proxy orqali o'tadi. Bu anonimlik, caching, filtering, security, load balancing kabi imkoniyatlarni bir joyga jamlab beradi. Forward proxy clientni, reverse proxy serverni "vakil" qiladi.

### ❓ Forward proxy va reverse proxy o'rtasidagi farq nima?

**✅ Javob:** Forward proxy **client tomonida** turadi, clientlarni yashiradi va ularning nomidan internetga chiqadi (anonimlik, filtering, korporativ caching). Clientning o'zi uni sozlaydi. Reverse proxy **server tomonida** turadi, backend serverlarni yashiradi va tashqaridan kelayotgan so'rovlarni qabul qiladi (TLS termination, load balancing, security). Client uning borligini bilmaydi. Qisqasi: forward = client uchun, reverse = server uchun.

### ❓ Reverse proxy qanday vazifalarni bajaradi?

**✅ Javob:** TLS termination (HTTPS'ni ochib backendga oddiy HTTP uzatish), caching (tez-tez so'raladigan javoblarni saqlash), compression (gzip/brotli), load balancing (so'rovlarni backendlar orasida taqsimlash), security (WAF, rate limiting, backendni internetdan yashirish) va statik fayllarni o'zi berish. nginx, HAProxy, Envoy — eng mashhur misollar.

### ❓ Load balancer nima va nega kerak?

**✅ Javob:** Load balancer so'rovlarni bir nechta backend server orasida taqsimlaydi. Kerakligi: (1) scalability — trafik o'sganda ko'p server qo'shib yukni bo'lish; (2) high availability — bir server o'lsa, trafikni faqat tirik serverlarga yuborib, foydalanuvchini buzilishdan himoya qilish; (3) maintenance — serverlarni ketma-ket yangilash (rolling deployment).

### ❓ L4 va L7 load balancing farqi nimada?

**✅ Javob:** L4 (transport) load balancing TCP/UDP darajasida, faqat IP va portni ko'rib ishlaydi — juda tez, lekin paket mazmunini ko'rmaydi, kontentga qarab qaror qabul qila olmaydi. L7 (application) load balancing HTTP darajasida URL, header, cookie'ni ko'rib, kontentga qarab marshrutlaydi (path-based routing), TLS termination va caching qiladi — aqlli, lekin sekinroq. Path'ga qarab marshrutlash kerak bo'lsa L7, faqat tez TCP forwarding kerak bo'lsa L4.

### ❓ Qanday load balancing algoritmlarini bilasiz?

**✅ Javob:** Round robin (navbat bilan), weighted round robin (serverlar quvvatiga qarab og'irlik), least connections (eng kam aktiv ulanishi bor serverga), IP hash (client IP'ga qarab doim bir serverga), va consistent hashing (server qo'shilganda minimal qayta taqsimlash — distributed cache va sharding uchun). Stateless backendlar uchun round robin yoki least connections odatda yetarli.

### ❓ Consistent hashing oddiy hashingdan nimasi bilan yaxshi?

**✅ Javob:** Oddiy `hash(key) % N` da serverlar soni (N) o'zgarsa (server qo'shilsa/chiqsa), deyarli barcha kalitlar boshqa serverga ko'chadi — distributed cache'da bu "cache storm" keltirib chiqaradi. Consistent hashing serverlar va kalitlarni halqa (ring) bo'ylab joylashtiradi, server qo'shilganda faqat ~1/N qism kalit ko'chadi. Shuning uchun Redis cluster, Memcached, sharding tizimlarida ishlatiladi.

### ❓ Health check nima va nega muhim?

**✅ Javob:** Health check — load balancer har bir backendning tirikligini muntazam tekshirishi. Active (LB o'zi `GET /health` yuboradi) yoki passive (oddiy so'rovlardagi xatolarni kuzatadi) bo'ladi. O'lik yoki nosog'lom serverga trafik yubormaslik uchun zarur. Muhim nuqta: health check nafaqat process tirikligini, balki haqiqatan xizmat ko'rsata olishini (DB ulanishi, downstream xizmatlar) tekshirishi kerak.

### ❓ Sticky session nima va nega undan qochish kerak?

**✅ Javob:** Sticky session (session affinity) — bir clientni doim bir xil backendga yo'naltirish (cookie yoki IP hash orqali). Session in-memory saqlansa kerak bo'ladi. Lekin u anti-pattern: yukni teng taqsimlanmasligiga olib keladi va server o'lsa o'sha clientlarning sessioni yo'qoladi. To'g'ri yechim — session'ni Redis/DB kabi tashqi store'ga chiqarib, backendni stateless qilish.

### ❓ API Gateway nima va reverse proxy'dan farqi nimada?

**✅ Javob:** API Gateway — microservicelar uchun yagona kirish nuqtasi. U routing, authentication, rate limiting, request aggregation, logging kabi cross-cutting concern'larni bir joyda bajaradi. Reverse proxy tarmoq darajasida ishlaydi (TLS, LB, caching — "qanday uzatish"), API Gateway esa API/biznes darajasida ishlaydi (auth, rate limit, aggregation — "nima qilib uzatish"). Gateway — aqlli reverse proxy'ning yuqori darajadagi ko'rinishi; ko'p gateway'lar ichida nginx/Envoy ishlaydi.

### ❓ Request aggregation nima va qachon foydali?

**✅ Javob:** API Gateway bitta client so'rovi uchun bir nechta mikroservisni chaqirib, javoblarni birlashtirib qaytarishi. Masalan, mobil ilova bitta sahifa uchun user, order va recommendation ma'lumotini so'rasa, gateway uch xizmatni chaqirib, bitta javob beradi. Bu mobil/sekin tarmoqlarda foydali, chunki round trip soni kamayadi (3 ta o'rniga 1 ta).

### ❓ CDN nima va qanday ishlaydi?

**✅ Javob:** CDN (Content Delivery Network) — dunyo bo'ylab tarqalgan edge serverlar tarmog'i. Foydalanuvchiga jismonan yaqin joydan kontent beradi, shu bilan latency'ni kamaytiradi. Ishlash: (1) geo-routing — foydalanuvchi eng yaqin edge'ga yo'naltiriladi; (2) edge caching — statik kontent edge'da keshlanadi (birinchi so'rov origindan — cache miss, keyingilar edge'dan — cache hit); (3) origin — asl server, edge'da kontent bo'lmaganda undan olinadi. Past latency, originga kam yuk, DDoS himoya beradi.

### ❓ Cache HIT va cache MISS nima?

**✅ Javob:** Cache hit — so'ralgan kontent keshda (masalan, CDN edge'da) mavjud, darhol qaytariladi (tez). Cache miss — kontent keshda yo'q, asl manbadan (origin) olinadi, keyin keshga yoziladi, so'ng beriladi (sekinroq). CDN samaradorligi cache hit ratio bilan o'lchanadi — qancha yuqori bo'lsa, originga shuncha kam yuk va past latency.

### ❓ TLS termination nima va u qayerda bo'ladi?

**✅ Javob:** TLS termination — shifrlangan HTTPS trafikni ochib (decrypt), undan keyin backendga oddiy HTTP uzatish. Odatda reverse proxy yoki load balancer (L7)da bo'ladi. Foydasi: backend serverlar TLS shifrlash/ochish bilan shug'ullanmaydi (CPU tejaladi), sertifikatlar bir markaziy joyda boshqariladi. Ehtiyot: proxy va backend o'rtasidagi ulanish ichki tarmoqda bo'lishi yoki re-encrypt qilinishi kerak (xavfsizlik uchun).

### ❓ Bir arxitekturada proxy, LB, gateway va CDN qanday joylashadi?

**✅ Javob:** Trafik tashqaridan ichkariga: foydalanuvchi → (ixtiyoriy forward proxy/VPN) → CDN (statik kontent, geo-routing, DDoS) → Load Balancer (yuk taqsimlash, TLS termination, health check) → API Gateway (auth, rate limit, routing) → mikroservislar → DB/cache. Real hayotda bu qatlamlar tez-tez birlashtiriladi: nginx bir o'zi reverse proxy + L7 LB + TLS + caching bo'lishi mumkin; Cloudflare esa CDN + WAF + LB + DDoS himoyani birlashtiradi.

### ❓ Reverse proxy backendni qanday himoya qiladi?

**✅ Javob:** Reverse proxy backend serverlarni internetdan to'g'ridan-to'g'ri yashiradi — tashqi clientlar faqat proxy IP'sini ko'radi, backend manzillarini emas. Bunga qo'shimcha: WAF (Web Application Firewall) bilan zararli so'rovlarni filtrlaydi, rate limiting bilan abuse'ni cheklaydi, DDoS hujumlarini yutadi, va TLS terminationni markazlashtiradi. Backend xususiy tarmoqda qoladi, hujum yuzasi (attack surface) kichrayadi.

---

## Masalalar

> Yechimlar: [solutions/networking/05-proxy-load-balancing.md](../solutions/networking/05-proxy-load-balancing.md)

1. **Forward yoki reverse?** Quyidagi holatlar uchun qaysi proxy turi kerakligini aniqlang va sababini yozing: (a) maktab tarmog'ida o'yin saytlarini bloklash; (b) 5 ta backend orasida yukni taqsimlash; (c) korxona xodimlari trafigini bitta IP orqali chiqarish; (d) HTTPS sertifikatni bitta joyda boshqarish.

2. **LB algoritmi tanlash.** Quyidagi senariylar uchun mos load balancing algoritmini tanlang va asoslang: (a) bir xil quvvatdagi stateless serverlar; (b) ba'zi serverlar 2 barobar kuchli; (c) so'rovlar davomiyligi juda turlicha (ba'zilari uzun stream); (d) distributed cache uchun key'larni serverlarga taqsimlash.

3. **L4 yoki L7?** Bir tizimda quyidagilar kerak: `/api/*` → API pool, `/static/*` → CDN origin, qolgani → web pool, hamda TLS termination. Bu L4 yoki L7 load balancing talab qiladi? nginx misolida `location` bloklarini eskiz qilib chizing.

4. **Health check loyihasi.** Backend service Postgres va Redis'ga bog'liq. Yaxshi health check endpoint qanday bo'lishi kerak? `/health` (liveness) va `/ready` (readiness) o'rtasidagi farqni tushuntiring va qaysi birini LB ishlatishi kerakligini ayting.

5. **Sticky session muammosi.** Bir jamoa session'ni server xotirasida saqlagani uchun sticky session yoqib qo'ygan. Endi bir server o'lsa, foydalanuvchilar tizimdan chiqib ketmoqda. Muammoni tahlil qiling va stateless yechimni qadamlab tasvirlang.

6. **Consistent hashing.** 3 ta cache serveringiz bor (`% 3` bilan taqsimlangan). 4-serverni qo'shsangiz, oddiy hashing'da nima yuz beradi? Consistent hashing buni qanday yaxshilaydi? "Virtual node" tushunchasi nima uchun kerakligini izohlang.

7. **API Gateway dizayni.** 4 ta mikroservisli (user, order, payment, notification) tizim uchun API Gateway dizaynlang: routing jadvali, qaysi cross-cutting concern'lar gateway'da, qaysilari servislarda bo'lishi kerak. JWT auth'ni qayerda tekshirasiz?

8. **CDN strategiyasi.** Saytda statik rasmlar, foydalanuvchi profil rasmlari (tez-tez o'zgaradi) va dinamik HTML bor. Qaysilarini CDN'da keshlaysiz, qaysilarini yo'q? TTL'larni qanday belgilaysiz? Cache invalidation muammosini qanday hal qilasiz?

9. **To'liq arxitektura.** O'zbekistondagi e-commerce sayt uchun: 3 ta region'dan foydalanuvchilar, statik + dinamik kontent, microservicelar, yuqori yuk (Black Friday). Proxy/LB/gateway/CDN qatlamlarini joylashtirib, to'liq arxitektura diagrammasini chizing va har bir qatlam tanlovini asoslang.

10. **Nosozlik tahlili.** Foydalanuvchilar "ba'zida login bo'ladi, ba'zida chiqib ketadi" deb shikoyat qilmoqda. Tizimda 3 ta backend va round-robin LB bor. Sabab nima bo'lishi mumkin? Diagnostika qadamlarini va yechim variantlarini sanang.

---

← [Networking bo'limiga qaytish](./README.md)
