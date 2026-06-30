# Backend nima

Bu fayl "backend" so'zining aniq ma'nosini, client-server modelni, HTTP so'rovining brauzerdan serverga va orqaga to'liq sayohatini, hamda har bir harakatlanuvchi qism (OS, process, port, DB, cache, queue) qayerga joylashishini boshidan tushuntiradi. Maqsad: intervyuda "backend nima?" degan oddiy savolga ham, "request hayot sikli" degan murakkab savolga ham ishonch bilan javob bera olish.

## Mundarija

- [Tushuncha: "backend" nimani anglatadi](#tushuncha-backend-nimani-anglatadi)
- [Client-server model](#client-server-model)
- [HTTP so'rovining to'liq hayot sikli](#http-sorovining-toliq-hayot-sikli)
- [Server nima?](#server-nima)
- [Frontend vs Backend mas'uliyatlari](#frontend-vs-backend-masuliyatlari)
- [API nima?](#api-nima)
- [Statelessness (holatsizlik)](#statelessness-holatsizlik)
- [OS, process va thread roli](#os-process-va-thread-roli)
- [Port va socket](#port-va-socket)
- [Environment variable va config](#environment-variable-va-config)
- [Runtime roli](#runtime-roli)
- [Monolith vs services](#monolith-vs-services)
- [DB, cache va queue qayerda](#db-cache-va-queue-qayerda)
- [Deployment asoslari](#deployment-asoslari)
- [Intervyu savollari (Q&A)](#intervyu-savollari-qa)
- [Masalalar](#masalalar)

---

## Tushuncha: "backend" nimani anglatadi

**💡 Tushuncha:** **Backend** — bu foydalanuvchi ko'zidan yashirin, serverda ishlaydigan hamma narsa: so'rovlarni qayta ishlovchi application kodi, ma'lumotni saqlovchi database'lar va ularni bog'lab turuvchi infrastruktura. **Frontend** esa foydalanuvchi qurilmasida (brauzer, telefon) ishlaydigan qism — u nimani ko'rsatishni biladi, lekin ma'lumotning manbai va haqiqiy biznes-logika serverda turadi.

Sodda qilib aytganda: frontend "qanday ko'rinadi"ni, backend esa "qanday ishlaydi va qayerda saqlanadi"ni hal qiladi. Brauzeringizda Instagram lentasini ko'rganingizda — rasmlar, like soni, kommentlar backend serverdan keladi; frontend ularni faqat chiroyli joylashtiradi.

Backend muhandisi odatda quyidagilar bilan shug'ullanadi:
- **API** — frontend va boshqa servislar so'rov yuboradigan "eshik".
- **Business logic** — qoidalar (masalan, "balansda yetarli pul bo'lsagina pul o'tkazish").
- **Data layer** — database, cache, fayl saqlash.
- **Infrastructure** — server, deployment, monitoring, xavfsizlik.

---

## Client-server model

**💡 Tushuncha:** Internetning asosini **client-server** modeli tashkil qiladi. **Client** (mijoz) — so'rov *boshlovchi* tomon (brauzer, mobil ilova, boshqa server). **Server** — so'rovni *kutib turuvchi* va unga javob qaytaruvchi tomon. Aloqa har doim client tomonidan boshlanadi.

```text
   CLIENT                              SERVER
 (brauzer)                          (Node.js ilova)
    |                                     |
    |  ----  HTTP request  ----------->   |   "menga /users/42 ber"
    |                                     |   (DB'dan o'qiydi, logika)
    |  <----  HTTP response  ----------   |   "mana JSON: { id: 42 }"
    |                                     |
```

Bitta server bir vaqtning o'zida minglab client'larga xizmat qiladi. Bitta client esa ko'plab serverlarga (API, rasm CDN, analytics) so'rov yuborishi mumkin. "Server" so'zi ikki ma'noda ishlatiladi: (1) jismoniy/virtual *kompyuter* (machine), (2) shu kompyuterda ishlaydigan *dastur* (process). Intervyuda qaysi ma'noda gaplashayotganingizni aniq bilib turing.

**⚠️ Ehtiyot bo'l:** "Client" har doim brauzer emas. Microservice arxitekturasida bir backend servisi boshqa backend servisiga client bo'ladi. Rol — kim so'rov boshlaganiga qarab aniqlanadi, qurilma turiga emas.

---

## HTTP so'rovining to'liq hayot sikli

**💡 Tushuncha:** Brauzerda `https://example.com/users/42` ni ochganingizda, "Enter" bilan javob orasida o'nlab bosqich kechadi. Intervyuda eng sevimli savol — shu zanjirni tartibi bilan ayta olishingiz.

**1. URL parse.** Brauzer URL'ni qismlarga ajratadi: scheme (`https`), host (`example.com`), port (default `443`), path (`/users/42`).

**2. DNS resolution.** `example.com` — bu nom, kompyuterlar esa IP manzil bilan ishlaydi. Brauzer DNS so'roviga IP topadi (`93.184.216.34`). Avval cache'lar tekshiriladi: brauzer cache → OS cache → router → ISP'ning DNS resolver'i → root/TLD/authoritative serverlar.

```bash
# DNS qanday yechilishini ko'rish
nslookup example.com
dig example.com +short    # 93.184.216.34
```

**3. TCP connection (3-way handshake).** IP topilgach, brauzer server bilan ishonchli kanal o'rnatadi: `SYN → SYN-ACK → ACK`. Bu HTTP'ning ostida turuvchi transport qatlami.

**4. TLS handshake (HTTPS uchun).** `https` bo'lsa, shifrlangan kanal o'rnatiladi: server sertifikatini yuboradi, kalitlar kelishiladi. Endi trafik o'qib bo'lmaydigan holatda shifrlanadi.

**5. HTTP request yuborish.** Brauzer matnli so'rov jo'natadi:

```http
GET /users/42 HTTP/1.1
Host: example.com
Accept: application/json
Authorization: Bearer eyJhbGciOi...
```

**6. Server processing.** Server (masalan, Node.js) so'rovni qabul qiladi, **routing** qiladi (qaysi handler `/users/42` ga javob beradi?), **middleware** o'tkazadi (auth, logging), so'ng **business logic** ishga tushadi.

**7. Database / cache so'rovi.** Handler kerakli ma'lumotni oladi: avval cache (Redis) tekshiriladi, bo'lmasa database (PostgreSQL) so'raladi.

```sql
SELECT id, name, email FROM users WHERE id = 42;
```

**8. Response qaytarish.** Server status code, header va body bilan javob quradi:

```http
HTTP/1.1 200 OK
Content-Type: application/json
Content-Length: 54

{ "id": 42, "name": "Ali", "email": "ali@example.com" }
```

**9. Client render.** Brauzer JSON'ni qabul qiladi, frontend uni ekranda chiroyli ko'rsatadi. TCP ulanish keyingi so'rovlar uchun ochiq qolishi mumkin (`keep-alive`).

**⚠️ Ehtiyot bo'l:** DNS va TCP/TLS faqat *birinchi* so'rovda to'liq kechadi. Keyingi so'rovlar cache va ochiq ulanishdan foydalanadi — shu sababli birinchi yuklash sekinroq bo'ladi.

---

## Server nima?

**💡 Tushuncha:** Backend kontekstida **server** — bu tarmoqda ma'lum bir **port**ni "tinglab" (listen), kelgan so'rovlarga javob beruvchi uzluksiz ishlaydigan **process** (dastur). U cheksiz tsiklda so'rov kutadi:

```js
const http = require('http');

const server = http.createServer((req, res) => {
  res.writeHead(200, { 'Content-Type': 'application/json' });
  res.end(JSON.stringify({ message: 'Salom, dunyo!' }));
});

server.listen(3000, () => {
  console.log('Server 3000-portni tinglayapti');
});
// Process ishlab turadi va so'rov kelmaguncha "uxlaydi" (block bo'lmaydi).
```

Web server (Nginx, Apache) odatda statik fayl uzatadi va so'rovni application server'ga uzatadi (reverse proxy). **Application server** (Node.js ilovangiz) esa biznes logikani bajaradi. Bu farqni bilish foydali: prod'da odatda `client → Nginx → Node.js` zanjiri bo'ladi.

---

## Frontend vs Backend mas'uliyatlari

**💡 Tushuncha:** Mas'uliyatlarni to'g'ri taqsimlash — yaxshi arxitekturaning asosi. Quyidagi jadval intervyu uchun aniq mezon beradi:

| Mas'uliyat | Frontend | Backend |
|---|---|---|
| UI / render | ✅ | ❌ |
| Input validation (UX uchun) | ✅ | ✅ (majburiy) |
| Business logic | ❌ | ✅ |
| Authentication / authorization | ❌ (faqat token saqlash) | ✅ |
| Data persistence (DB) | ❌ | ✅ |
| Maxfiy kalitlar (secrets) | ❌ | ✅ |
| Performance (render) | ✅ | — |
| Performance (so'rov, DB) | — | ✅ |

**⚠️ Ehtiyot bo'l:** Validatsiya **ikkala** tomonda ham bo'lishi kerak. Frontend validatsiyasi faqat UX uchun (tez fikr-mulohaza), uni xohlagan odam chetlab o'tishi mumkin (curl, Postman). **Backend validatsiyasi — yagona ishonchli himoya.** "Frontend tekshirgan, backend ishonsa bo'ladi" — bu klassik xavfsizlik xatosi.

---

## API nima?

**💡 Tushuncha:** **API (Application Programming Interface)** — bu ikki dastur bir-biri bilan gaplashish uchun kelishilgan "shartnoma". Backend kontekstida u ko'pincha **HTTP API** (REST yoki GraphQL): aniq URL'lar, metodlar (`GET`, `POST`, ...) va ma'lumot formati (odatda JSON).

```text
GET    /users         → barcha foydalanuvchilar ro'yxati
GET    /users/42      → 42-foydalanuvchi
POST   /users         → yangi foydalanuvchi yaratish
PUT    /users/42      → 42-foydalanuvchini yangilash
DELETE /users/42      → 42-foydalanuvchini o'chirish
```

API — bu "implementatsiya yashiringan eshik". Client serverning ichida DB qanaqaligini, kod qanaqaligini bilmaydi — u faqat shartnomani (endpoint, format) biladi. Shu sababli backend ichini butunlay qayta yozsangiz ham, API bir xil qolsa, client'lar sezmaydi.

---

## Statelessness (holatsizlik)

**💡 Tushuncha:** HTTP **stateless** protokol — har bir so'rov mustaqil, server oldingi so'rovlarni "eslab qolmaydi". Agar server kim ekanligingizni bilishi kerak bo'lsa, har so'rovda buni o'zingiz aytib turishingiz kerak (masalan, `Authorization` header'ida token bilan).

```http
GET /profile HTTP/1.1
Authorization: Bearer eyJhbGciOi...   ← har so'rovda "men kimligimni" eslataman
```

**Nega muhim?** Stateless server — bu **scalable** server. Agar so'rov hech qanday lokal holatga bog'liq bo'lmasa, uni 1 ta yoki 100 ta server nusxasi (instance) bajara oladi — load balancer so'rovni istalgan instance'ga yuboradi. Holat (session) esa tashqi joyda (Redis, DB) saqlanadi.

**⚠️ Ehtiyot bo'l:** Holatni server xotirasida (`let sessions = {}`) saqlash — bitta server uchun ishlaydi, ammo siz ikkinchi instance qo'shganingizda buziladi: ikkinchi server birinchining xotirasini ko'rmaydi. Shuning uchun session'ni har doim tashqi store'da saqlang.

---

## OS, process va thread roli

**💡 Tushuncha:** Backend kodi yalang'och temirda emas, **Operating System (OS)** ustida ishlaydi. OS resurslarni (CPU, RAM, tarmoq, fayl) boshqaradi va ularni dasturlarga taqsimlaydi.

- **Process** — OS ishga tushirgan dasturning ishlayotgan nusxasi. O'z mustaqil xotirasiga ega. Sizning Node.js ilovangiz — bitta process.
- **Thread** — process ichidagi bajarilish oqimi. Bir process'da bir nechta thread bo'lishi mumkin, ular xotirani *bo'lishadi*.

```bash
ps aux | grep node      # ishlayotgan node process'larini ko'rish
top                     # CPU/RAM ishlatilishini real vaqtda kuzatish
```

Node.js sizning JavaScript kodingizni **bitta asosiy thread**da ishlatadi (event loop), lekin og'ir I/O ishlarini libuv orqali fondagi thread pool'ga beradi. Bu nozik detal keyingi mavzuda (Node.js asoslari) chuqurroq ochiladi.

---

## Port va socket

**💡 Tushuncha:** Bitta serverda (IP manzilda) ko'plab dastur ishlashi mumkin. **Port** — bu IP ichida qaysi dasturga so'rov borishini ajratuvchi raqam (0–65535). Masalan, HTTP odatda 80, HTTPS 443, sizning dev serveringiz 3000.

**Socket** — bu ulanishning ikki uchini ifodalovchi `(IP + port)` juftligi. To'liq ulanish to'rt qismdan iborat: `(client_IP:client_port, server_IP:server_port)`. Shu unikal kombinatsiya tufayli server minglab client'ni ajrata oladi.

```bash
# Qaysi port band ekanini ko'rish (Linux/Mac)
lsof -i :3000
# Windows
netstat -ano | findstr :3000
```

**⚠️ Ehtiyot bo'l:** `EADDRINUSE: address already in use :::3000` — eng ko'p uchraydigan xato. Bu shu portda boshqa process allaqachon tinglayotganini bildiradi. Eski process'ni to'xtating yoki boshqa port tanlang.

---

## Environment variable va config

**💡 Tushuncha:** **Environment variable** — kodning *tashqarisida*, OS/process muhitida saqlanadigan kalit-qiymat sozlamalar. Ular orqali bir xil kodni turli muhitda (development, staging, production) turlicha sozlaysiz. **Eng muhimi:** maxfiy ma'lumotlar (DB parol, API key) kodga *yozilmaydi* — ular env'dan o'qiladi.

```js
const port = process.env.PORT || 3000;
const dbUrl = process.env.DATABASE_URL;
const isProd = process.env.NODE_ENV === 'production';
```

```bash
# Terminalda vaqtincha o'rnatish
PORT=8080 DATABASE_URL="postgres://..." node server.js
```

Amaliyotda `.env` faylidan foydalaniladi (`dotenv` paketi bilan o'qiladi), ammo `.env` hech qachon Git'ga qo'yilmaydi (`.gitignore`ga qo'shiladi).

**⚠️ Ehtiyot bo'l:** Parol yoki API key'ni kodga yozib, GitHub'ga push qilish — eng keng tarqalgan xavfsizlik incident'i. Bir marta push qilingan secret "tarixda" qoladi va darhol almashtirilishi kerak.

---

## Runtime roli

**💡 Tushuncha:** **Runtime** — yozgan kodingizni haqiqatda ishga tushiruvchi muhit. JavaScript brauzerda brauzer runtime'ida (V8 + Web API), serverda esa **Node.js** runtime'ida ishlaydi. Runtime tilning o'zida bo'lmagan imkoniyatlarni beradi: fayl o'qish, tarmoq, process boshqaruvi.

Til (JavaScript) — bu faqat sintaksis va qoidalar. Runtime esa shu tilga "qo'l-oyoq" beradi. Bir xil JS kodi brauzerda `fs.readFile` ni topa olmaydi (chunki brauzer fayl tizimini bermaydi), Node.js'da esa ishlaydi. Bu farq keyingi mavzuda chuqur ochiladi.

---

## Monolith vs services

**💡 Tushuncha:** **Monolith** — butun ilova bitta deploylanadigan kod bazasi va process. **Microservices** — ilova kichik, mustaqil servislarga bo'linadi (auth-service, payment-service, ...), har biri alohida deploylanadi va tarmoq orqali gaplashadi.

| | Monolith | Microservices |
|---|---|---|
| Boshlash oson | ✅ | ❌ |
| Deploy oddiyligi | ✅ | ❌ (murakkab) |
| Mustaqil scaling | ❌ | ✅ |
| Jamoa mustaqilligi | ❌ | ✅ |
| Tarmoq murakkabligi | ❌ yo'q | ✅ ko'p |

**⚠️ Ehtiyot bo'l:** "Microservices zamonaviy, demak yaxshiroq" — noto'g'ri xulosa. Kichik jamoa va loyiha uchun monolith deyarli har doim to'g'ri tanlov. Microservices tarmoq, monitoring, distributed transaction kabi katta murakkabliklarni keltiradi. Ehtiyoj paydo bo'lganda bo'lib chiqasiz.

---

## DB, cache va queue qayerda

**💡 Tushuncha:** Backend faqat application kodi emas — uning yonida uchta tipik infratuzilma turadi:

- **Database (DB)** — ma'lumotning doimiy, ishonchli manbai (source of truth). Masalan, PostgreSQL, MongoDB. Server o'chib-yonsa ham ma'lumot saqlanadi.
- **Cache** — tez, ammo vaqtinchalik xotira (Redis, Memcached). Tez-tez so'raladigan ma'lumotni RAM'da saqlab, DB yukini kamaytiradi va javobni tezlashtiradi.
- **Queue** — vazifalarni *keyinroq* bajarish uchun navbat (RabbitMQ, Kafka, Redis). Masalan, "email yuborish" so'rovni bloklamasdan navbatga tashlanadi va fonda ishlovchi worker uni bajaradi.

```text
                     ┌──────────┐
 Client → API Server →│  Cache   │  ← avval shu tekshiriladi (tez)
                 │   └──────────┘
                 │   ┌──────────┐
                 └──→│ Database │  ← cache'da yo'q bo'lsa (sekinroq, ishonchli)
                 │   └──────────┘
                 └──→ Queue → Worker → (email, hisobot, og'ir ish)
```

**⚠️ Ehtiyot bo'l:** Cache "source of truth" emas — undagi ma'lumot eskirgan (stale) bo'lishi mumkin. Cache invalidation (cache'ni qachon yangilash/tozalash) — informatikadagi eng qiyin masalalardan biri sifatida mashhur.

---

## Deployment asoslari

**💡 Tushuncha:** **Deployment** — kodingizni o'z kompyuteringizdan olib, foydalanuvchilar kira oladigan serverga joylashtirish jarayoni. Zamonaviy oqim odatda quyidagicha:

1. Kod **Git**'ga push qilinadi.
2. **CI/CD** (GitHub Actions, GitLab CI) avtomatik test ishlatadi va build qiladi.
3. Ilova ko'pincha **container** (Docker) ichiga joylanadi — "mening kompyuterimda ishlardi" muammosini hal qiladi.
4. Container bulutda (AWS, GCP, yoki Kubernetes klasterida) ishga tushiriladi.
5. **Reverse proxy** (Nginx) va **load balancer** trafikni instance'lar orasida taqsimlaydi.

```bash
# Eng oddiy lokal "deploy" — process menejer bilan doimiy ishlatish
npm install -g pm2
pm2 start server.js --name my-api
pm2 logs my-api
```

**⚠️ Ehtiyot bo'l:** `node server.js` ni shunchaki ishlatib qo'yish prod uchun yetarli emas: process qulab tushsa, hech kim uni qayta yoqmaydi. Shuning uchun `pm2`, `systemd` yoki container orchestrator (Kubernetes) "crash bo'lsa qayta yoq" vazifasini bajaradi.

---

## Intervyu savollari (Q&A)

### ❓ "Backend" so'zi aniq nimani anglatadi?

**✅ Javob:** Backend — foydalanuvchi qurilmasidan tashqarida, serverda ishlaydigan hamma narsa: application kodi (so'rovlarni qayta ishlash, business logic), data layer (DB, cache) va infrastruktura. Frontend "qanday ko'rinadi"ni, backend "qanday ishlaydi va qayerda saqlanadi"ni hal qiladi.

### ❓ Client-server modelni tushuntiring.

**✅ Javob:** Client so'rovni *boshlaydi*, server so'rovni *kutadi* va javob qaytaradi. Aloqa har doim client tomonidan ochiladi. Bitta server ko'p client'ga xizmat qiladi. "Client" rol nomi — microservicesda bir server boshqasiga client bo'lishi mumkin.

### ❓ Brauzerda URL kiritib Enter bosgandan keyin nima sodir bo'ladi?

**✅ Javob:** URL parse → DNS resolution (nomdan IP) → TCP handshake → TLS handshake (https bo'lsa) → HTTP request → server processing (routing, middleware, business logic) → DB/cache so'rovi → HTTP response (status, header, body) → client render. DNS/TCP/TLS faqat birinchi so'rovda to'liq kechadi.

### ❓ Nima uchun validatsiya backendda ham bo'lishi shart?

**✅ Javob:** Frontend validatsiyasini har kim chetlab o'tishi mumkin (curl, Postman, modified client). Backend — yagona ishonchli himoya chizig'i. Frontend validatsiya faqat UX (tez fikr-mulohaza) uchun, xavfsizlik uchun emas.

### ❓ HTTP "stateless" degani nima va nega muhim?

**✅ Javob:** Har bir so'rov mustaqil, server oldingilarini eslamaydi. Demak, kim ekanligingizni har so'rovda (token bilan) aytib turasiz. Bu serverni scalable qiladi: holat lokal emas, tashqi store'da (Redis/DB) bo'lgani uchun so'rovni istalgan instance bajara oladi.

### ❓ Process va thread orasidagi farq nima?

**✅ Javob:** Process — mustaqil xotiraga ega ishlayotgan dastur nusxasi. Thread — process ichidagi bajarilish oqimi; bir process'dagi thread'lar xotirani bo'lishadi. Node.js JS kodini bitta asosiy thread'da ishlatadi, og'ir I/O'ni esa libuv thread pool'iga beradi.

### ❓ Port va socket nima?

**✅ Javob:** Port — bitta IP ichida qaysi dasturga so'rov borishini ajratuvchi raqam (0–65535). Socket — ulanish uchini ifodalovchi (IP + port) juftligi. To'liq ulanish 4 qismli: client (IP:port) va server (IP:port) — shu unikal kombinatsiya client'larni ajratadi.

### ❓ Environment variable nima uchun kerak?

**✅ Javob:** Bir xil kodni turli muhitda (dev/staging/prod) turlicha sozlash uchun va maxfiy ma'lumotlarni (parol, key) kodga yozmasdan tashqarida saqlash uchun. `process.env`dan o'qiladi, `.env` fayli Git'ga qo'yilmaydi.

### ❓ API nima va u nimani yashiradi?

**✅ Javob:** API — ikki dastur o'rtasidagi shartnoma (endpoint'lar, metodlar, format). U implementatsiyani yashiradi: client serverning ichki kodi yoki DB'sini bilmaydi, faqat shartnomani biladi. Shu sababli ichki qismni qayta yozsangiz ham, API bir xil bo'lsa, client'lar sezmaydi.

### ❓ Monolith va microservices: qachon qaysi biri?

**✅ Javob:** Monolith — kichik jamoa va loyiha uchun, oddiy deploy, tez boshlash. Microservices — mustaqil scaling va jamoa mustaqilligi kerak bo'lganda, ammo tarmoq, monitoring, distributed transaction murakkabliklarini keltiradi. "Zamonaviy" degani har doim "to'g'ri" emas — ehtiyojga qarang.

### ❓ Cache, database va queue rollari nimada farq qiladi?

**✅ Javob:** Database — doimiy, ishonchli source of truth. Cache — tez, vaqtinchalik RAM-store, DB yukini kamaytiradi (lekin stale bo'lishi mumkin). Queue — og'ir/keyinroq bajariladigan ishlarni (email, hisobot) navbatga tashlab, so'rovni bloklamaydi.

### ❓ Deployment'da `node server.js` nima uchun yetarli emas?

**✅ Javob:** Process qulab tushsa, uni hech kim qayta yoqmaydi. Prod'da process menejer (pm2, systemd) yoki orchestrator (Kubernetes) "crash bo'lsa qayta yoq", logging, bir nechta instance va load balancing'ni ta'minlaydi.

### ❓ Web server va application server farqi nima?

**✅ Javob:** Web server (Nginx, Apache) — statik fayl uzatadi va so'rovni application server'ga yo'naltiradi (reverse proxy). Application server (Node.js ilovangiz) — business logicni bajaradi. Prod zanjiri odatda `client → Nginx → Node.js`.

### ❓ Session'ni server xotirasida saqlash nima uchun yomon g'oya?

**✅ Javob:** Bitta instance uchun ishlaydi, ammo siz ikkinchi server qo'shganda buziladi — ikkinchi instance birinchining xotirasini ko'rmaydi. Stateless qolish uchun session'ni tashqi store'da (Redis/DB) saqlang.

---

## Masalalar

> Yechimlar: [solutions/backend/01-what-is-backend.md](../solutions/backend/01-what-is-backend.md)

1. **Request hayot sikli diagrammasi.** `https://api.shop.uz/products/7` URL'i uchun DNS'dan client render'gacha bo'lgan barcha bosqichlarni tartibi bilan yozing va har bir bosqichda aynan nima sodir bo'lishini bir jumlada izohlang.

2. **Minimal server.** `http` moduli bilan `/health` endpoint'i `{ "status": "ok" }` qaytaradigan, boshqa har qanday path'ga `404` beradigan server yozing. Portni `process.env.PORT` yoki default `3000`'dan oling.

3. **Stateless vs stateful.** Bir backend ikki instance'da ishlayapti, sessionlar bitta instance xotirasida saqlanadi. Foydalanuvchi nima uchun "ba'zan tizimda, ba'zan tizimdan chiqib ketgan" bo'lib ko'rinadi? Muammoni va to'g'ri yechimni tushuntiring.

4. **Mas'uliyat taqsimoti.** Quyidagi vazifalarni Frontend yoki Backend (yoki ikkalasi) bajarishi kerak, deb belgilang va sababini yozing: (a) parolni hash qilish, (b) tugma rangini o'zgartirish, (c) "email noto'g'ri formatda" xabarini ko'rsatish, (d) to'lov summasini hisoblash.

5. **Port band xatosi.** Serverni ishga tushirganda `EADDRINUSE` xatosi chiqdi. Bu nima degani? Qaysi process portni band qilganini qanday topib, uni qanday bo'shatasiz? (Linux va Windows buyruqlarini yozing.)

6. **Cache strategiyasi.** `/products/7` so'rovi tez-tez keladi va ma'lumot kam o'zgaradi. Cache'ni qayerga qo'yasiz, qanday ketma-ketlikda tekshiruvni amalga oshirasiz (cache → DB), va eski (stale) ma'lumot muammosini qanday yumshatasiz?

7. **Env config.** Loyiha `DATABASE_URL`, `PORT` va `NODE_ENV` ga muhtoj. Bu qiymatlarni kodga yozmasdan qanday berasiz? `.env` fayli nima uchun `.gitignore`da bo'lishi kerak? Secret tasodifan push qilinsa nima qilasiz?

8. **Monolith → service ajratish.** Katta monolitda "email yuborish" funksiyasi butun so'rovni sekinlashtirmoqda. Uni qanday ajratib, queue orqali async qilib bo'ladi? Qisqa diagramma chizing.

← [Backend bo'limiga qaytish](./README.md)
