# Backend nima — Yechimlar

Bu fayl [`backend/01-what-is-backend.md`](../../backend/01-what-is-backend.md) dagi "Masalalar" bo'limining yechimlarini izohlari bilan beradi. Avval mustaqil urinib ko'ring, keyin solishtiring.

---

## 1. Request hayot sikli diagrammasi

`https://api.shop.uz/products/7` uchun bosqichlar:

```text
1. URL parse        → scheme=https, host=api.shop.uz, port=443, path=/products/7
2. DNS resolution   → "api.shop.uz" nomidan IP topiladi (cache: brauzer→OS→ISP resolver)
3. TCP handshake    → SYN → SYN-ACK → ACK (ishonchli kanal o'rnatiladi)
4. TLS handshake    → sertifikat tekshiriladi, shifrlash kalitlari kelishiladi
5. HTTP request     → GET /products/7 HTTP/1.1, Host, Accept headerlari yuboriladi
6. Server processing→ routing (/products/:id), middleware (auth, log), business logic
7. DB/cache so'rovi → avval Redis cache, bo'lmasa PostgreSQL: SELECT ... WHERE id=7
8. HTTP response    → 200 OK, Content-Type: application/json, body: {"id":7,...}
9. Client render    → frontend JSON'ni qabul qilib, mahsulot kartasini chizadi
```

**Izoh:** 2–4 bosqichlar faqat *birinchi* so'rovda to'liq kechadi; keyingilari cache va `keep-alive` ulanishdan foydalanadi. Shu sababli birinchi yuklash sekinroq seziladi.

---

## 2. Minimal server

```js
const http = require('http');

const server = http.createServer((req, res) => {
  if (req.method === 'GET' && req.url === '/health') {
    res.writeHead(200, { 'Content-Type': 'application/json' });
    res.end(JSON.stringify({ status: 'ok' }));
    return;
  }

  res.writeHead(404, { 'Content-Type': 'application/json' });
  res.end(JSON.stringify({ error: 'Not Found' }));
});

const port = process.env.PORT || 3000;
server.listen(port, () => {
  console.log(`Server ${port}-portni tinglayapti`);
});
// GET /health  → 200 { "status": "ok" }
// boshqa path  → 404 { "error": "Not Found" }
```

**Izoh:** Metodni ham tekshiramiz (`req.method === 'GET'`), aks holda `POST /health` ham 200 qaytarardi. `process.env.PORT || 3000` — env bo'lsa o'sha, bo'lmasa default.

---

## 3. Stateless vs stateful

**Muammo:** Ikki instance (A va B) load balancer ortida turibdi. Foydalanuvchi login qiladi — so'rov A'ga tushadi, session A xotirasida yaratiladi. Keyingi so'rov B'ga tushadi — B foydalanuvchining sessionini *ko'rmaydi* (u faqat A xotirasida), shuning uchun "tizimdan chiqib ketgan" deb biladi. Load balancer so'rovlarni A va B orasida aylantirgani uchun foydalanuvchi "goh tizimda, goh tizimdan tashqarida" bo'lib ko'rinadi.

**Yechim:** Sessionni *tashqi, umumiy* store'da saqlash:
- **Redis** kabi markazlashgan session store — ikkala instance ham bir joyga qaraydi.
- Yoki **stateless token** (JWT) — holat tokenning o'zida, serverda hech narsa saqlanmaydi.

```js
// Yomon: lokal xotira (faqat bitta instance uchun)
const sessions = {};                  // ❌ A va B bo'lishmaydi

// Yaxshi: umumiy Redis store
await redis.set(`session:${id}`, JSON.stringify(data), 'EX', 3600);  // ✅
```

**Izoh:** "Sticky session" (load balancer bir foydalanuvchini doim bir instance'ga yuborishi) ham mavjud, lekin u instance qulasa session yo'qoladi va scaling moslashuvchanligini kamaytiradi — shuning uchun tashqi store afzal.

---

## 4. Mas'uliyat taqsimoti

| Vazifa | Kim | Sabab |
|---|---|---|
| (a) parolni hash qilish | **Backend** | Maxfiy operatsiya; hash algoritmi va salt serverda. Parol hech qachon frontendda hash qilinmaydi (raw parol baribir serverga TLS orqali keladi). |
| (b) tugma rangini o'zgartirish | **Frontend** | Sof UI/render masalasi, serverning aloqasi yo'q. |
| (c) "email noto'g'ri formatda" xabari | **Ikkalasi** | Frontend — UX uchun darhol ko'rsatadi; Backend — ishonchli, majburiy validatsiya (curl orqali chetlab o'tishga qarshi). |
| (d) to'lov summasini hisoblash | **Backend** | Business logic va pul — hech qachon clientga ishonib bo'lmaydi. Client narxni o'zgartirib yuborishi mumkin. |

**Izoh:** Umumiy qoida — *pul, xavfsizlik, ishonch* talab qiladigan hamma narsa backendda. Frontend faqat ko'rsatish va tezkor UX uchun nusxa qiladi.

---

## 5. Port band xatosi

`EADDRINUSE` = "address already in use": shu portda boshqa process allaqachon `listen` qilyapti.

```bash
# Linux / Mac — portni egallagan PID'ni topish
lsof -i :3000
# yoki
fuser 3000/tcp
# Process'ni to'xtatish
kill -9 <PID>
```

```bash
# Windows — PID'ni topish
netstat -ano | findstr :3000
# Process'ni to'xtatish
taskkill /PID <PID> /F
```

**Izoh:** Ko'pincha bu eski, "muzlab qolgan" dev server bo'ladi (nodemon to'g'ri yopilmagan). Agar portni bo'shatib bo'lmasa, vaqtinchalik boshqa portda ishga tushiring: `PORT=3001 node server.js`.

---

## 6. Cache strategiyasi

**Joylashuv:** Cache (Redis) API server bilan DB orasida. Oqim — **cache-aside (lazy loading)**:

```js
async function getProduct(id) {
  const key = `product:${id}`;

  // 1. Avval cache
  const cached = await redis.get(key);
  if (cached) return JSON.parse(cached);          // cache HIT — tez

  // 2. Cache'da yo'q bo'lsa — DB
  const product = await db.query('SELECT * FROM products WHERE id=$1', [id]);

  // 3. Keyingi safar uchun cache'ga yozamiz, TTL bilan
  await redis.set(key, JSON.stringify(product), 'EX', 300);  // 5 daqiqa
  return product;
}
```

**Stale (eski) ma'lumotni yumshatish:**
- **TTL** (`EX 300`) — har 5 daqiqada cache eskirib, DB'dan yangilanadi. Eskirish oynasi cheklanadi.
- **Write invalidation** — mahsulot yangilanganda (`UPDATE products`) shu kalitni o'chiramiz: `redis.del('product:7')`. Keyingi so'rov yangi ma'lumotni DB'dan oladi.

**Izoh:** TTL — soddalik, invalidation — aniqlik beradi. Ko'pincha ikkalasi birga ishlatiladi: TTL "xavfsizlik to'ri", invalidation esa darhol yangilanish uchun.

---

## 7. Env config

```bash
# .env fayli (Git'ga qo'yilmaydi)
DATABASE_URL=postgres://user:pass@localhost:5432/shop
PORT=3000
NODE_ENV=development
```

```js
require('dotenv').config();           // .env'ni process.env'ga yuklaydi
const dbUrl = process.env.DATABASE_URL;
const port = process.env.PORT || 3000;
const isProd = process.env.NODE_ENV === 'production';
```

**`.env` nima uchun `.gitignore`da:** U DB parol kabi maxfiy ma'lumotlarni saqlaydi. Git'ga tushsa, repo'ga kirgan har bir kishi (va ochiq repo bo'lsa — butun internet) parolni ko'radi. Buning o'rniga `.env.example` (qiymatlarsiz, faqat kalit nomlari) commit qilinadi.

**Secret tasodifan push qilinsa:**
1. Darhol o'sha secretni **almashtirib** yuboring (rotate) — push qilingan kalit "compromised" hisoblanadi.
2. `.gitignore`ga qo'shing va repo'dan o'chiring.
3. Faqat oxirgi commit'dan o'chirish yetarli emas — secret Git *tarixida* qoladi. Tarixni tozalash uchun `git filter-repo` yoki BFG ishlatiladi, lekin baribir kalit allaqachon ochilgani uchun rotate majburiy.

**Izoh:** "Faylни o'chirdim, demak xavfsiz" — eng keng tarqalgan xato. Tarix qoladi; yagona ishonchli yechim — kalitni almashtirishdir.

---

## 8. Monolith → service ajratish

**Muammo:** `POST /orders` ichida `sendEmail()` sinxron chaqirilgan — SMTP javobini kutib, so'rov 2–3 soniya bloklanadi.

**Yechim:** Email yuborishni queue orqali async qilamiz. So'rov faqat "vazifani navbatga tashlash"ni bajaradi va darhol javob qaytaradi; haqiqiy yuborishni fondagi worker bajaradi.

```text
  ┌──────────────┐   1. order yaratiladi      ┌──────────┐
  │  API Server  │ ─────────────────────────→ │    DB    │
  │ POST /orders │                            └──────────┘
  └──────┬───────┘   2. "email-job" qo'yiladi
         │           ┌──────────┐
         └──────────→│  Queue   │  (RabbitMQ / Redis / Kafka)
                     └────┬─────┘
                          │ 3. job olinadi
                     ┌────▼─────────┐
                     │ Email Worker │ → 4. SMTP orqali email yuboradi
                     └──────────────┘
   API darhol 201 Created qaytaradi (emailni kutmaydi).
```

```js
// API ichida — bloklamaydi
await db.insertOrder(order);
await queue.add('send-email', { to: user.email, orderId: order.id });
res.status(201).json({ id: order.id });           // tez javob

// Alohida worker process'ida
queue.process('send-email', async (job) => {
  await mailer.send(job.data.to, /* ... */);        // og'irlik fonda
});
```

**Izoh:** Bu "async/queue" pattern. Foyda: so'rov tez, email xatosi so'rovni qulatmaydi, worker'larni alohida scale qilish mumkin. Bunda email "darhol" emas, "tez orada" yetkaziladi — bu odatda maqbul kelishuv (trade-off).

---

← [Backend bo'limiga qaytish](../../backend/README.md)
