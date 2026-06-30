# HTTP va REST API

Web backend'ning poydevori — HTTP protokoli qanday ishlaydi, REST prinsiplari nima, va Node.js'da qanday qilib professional, xavfsiz va hujjatlashtirilgan API quriladi. Bu mavzu deyarli har bir backend intervyuda chiqadi.

## Mundarija

- [HTTP protokol asoslari](#http-protokol-asoslari)
- [HTTP metodlari, safety va idempotency](#http-metodlari-safety-va-idempotency)
- [HTTP headerlar](#http-headerlar)
- [Status kodlar](#status-kodlar)
- [HTTP/1.1 vs HTTP/2 vs HTTP/3 va keep-alive](#http11-vs-http2-vs-http3-va-keep-alive)
- [REST prinsiplari](#rest-prinsiplari)
- [Richardson Maturity Model va HATEOAS](#richardson-maturity-model-va-hateoas)
- [RESTful endpoint dizayni](#restful-endpoint-dizayni)
- [Versioning, pagination, filtering, sorting](#versioning-pagination-filtering-sorting)
- [Request/Response body va content negotiation](#requestresponse-body-va-content-negotiation)
- [Express'da API qurish](#expressda-api-qurish)
- [Validation](#validation)
- [CORS](#cors)
- [Rate limiting](#rate-limiting)
- [Idempotency key](#idempotency-key)
- [REST vs GraphQL vs gRPC vs tRPC](#rest-vs-graphql-vs-grpc-vs-trpc)
- [WebSocket vs SSE vs polling](#websocket-vs-sse-vs-polling)
- [OpenAPI](#openapi)
- [Intervyu savollari (Q&A)](#intervyu-savollari-qa)
- [Masalalar](#masalalar)

---

## HTTP protokol asoslari

**💡 Tushuncha:** HTTP (HyperText Transfer Protocol) — client va server o'rtasidagi so'rov/javob (request/response) protokoli. U **stateless** (holatsiz): server avvalgi so'rovlarni eslab qolmaydi, har bir request o'zicha to'liq bo'lishi kerak. Odatda TCP ustida ishlaydi (HTTP/3'da esa QUIC/UDP ustida).

Bir HTTP request quyidagilardan iborat:

```http
POST /api/users HTTP/1.1
Host: api.example.com
Content-Type: application/json
Authorization: Bearer eyJhbGci...

{ "name": "Ali", "email": "ali@example.com" }
```

- **Request line**: metod (`POST`) + path (`/api/users`) + versiya (`HTTP/1.1`).
- **Headers**: meta-ma'lumot (`Content-Type`, `Authorization`...).
- **Body**: ixtiyoriy yuk (payload).

Response esa:

```http
HTTP/1.1 201 Created
Content-Type: application/json
Location: /api/users/42

{ "id": 42, "name": "Ali" }
```

- **Status line**: versiya + status kod (`201`) + reason (`Created`).
- **Headers** va ixtiyoriy **body**.

**⚠️ Ehtiyot bo'l:** "Stateless" degani server hech narsa saqlamaydi degani emas — DB'da, albatta, saqlaydi. Bu shuni anglatadiki, *har bir request* o'zini kim ekanligini (auth token, session id) o'zi bilan olib kelishi kerak; server bir request'ni boshqasiga bog'lab "eslab" qolmaydi.

---

## HTTP metodlari, safety va idempotency

**💡 Tushuncha:** Metod (verb) — resurs ustida qanday amal qilishni bildiradi.

| Metod | Maqsad | Safe? | Idempotent? | Body? |
|-------|--------|-------|-------------|-------|
| `GET` | O'qish | Ha | Ha | Yo'q |
| `POST` | Yangi resurs yaratish | Yo'q | Yo'q | Ha |
| `PUT` | To'liq almashtirish (replace) | Yo'q | Ha | Ha |
| `PATCH` | Qisman yangilash | Yo'q | Yo'q* | Ha |
| `DELETE` | O'chirish | Yo'q | Ha | Yo'q* |
| `HEAD` | GET kabi, lekin faqat header | Ha | Ha | Yo'q |
| `OPTIONS` | Qo'llab-quvvatlanadigan metodlar | Ha | Ha | Yo'q |

- **Safe (xavfsiz)**: server holatini o'zgartirmaydi (read-only). `GET`, `HEAD`, `OPTIONS`.
- **Idempotent**: bir xil so'rovni bir necha marta yuborsang ham natija (server holati) bir xil bo'ladi. `PUT /users/1` ni 10 marta yuborsang ham, foydalanuvchi 1 ta xil holatda qoladi.

**⚠️ Ehtiyot bo'l:** `POST` idempotent EMAS — uni 3 marta yuborsang, 3 ta resurs yaratiladi (3 ta to'lov!). `PATCH` umumiy holatda idempotent emas (masalan `{ "balance": "+10" }` har safar qo'shadi), lekin uni idempotent qilib loyihalash mumkin (`{ "balance": 100 }`). `DELETE` idempotent: 1-marta o'chiradi (`204`), keyin `404` qaytaradi, lekin server holati bir xil — resurs yo'q.

---

## HTTP headerlar

**💡 Tushuncha:** Headerlar — request/response haqida metama'lumot tashuvchi `Key: Value` juftliklari.

Eng muhimlari:

| Header | Tomon | Vazifa |
|--------|-------|--------|
| `Content-Type` | Ikkalasi | Body formati (`application/json`, `text/html`) |
| `Accept` | Request | Client qaysi formatni qabul qiladi |
| `Authorization` | Request | Auth ma'lumoti (`Bearer <token>`) |
| `Cache-Control` | Ikkalasi | Cache siyosati (`no-store`, `max-age=3600`) |
| `ETag` / `If-None-Match` | Ikkalasi | Conditional request, cache validatsiya |
| `Location` | Response | Yaratilgan/ko'chirilgan resurs URL'i |
| `Set-Cookie` / `Cookie` | Response/Request | Cookie almashish |
| `Content-Length` | Ikkalasi | Body o'lchami (bayt) |
| `Retry-After` | Response | `429`/`503` da qachon qayta urinish |

**⚠️ Ehtiyot bo'l:** Header nomlari **case-insensitive** (`content-type` == `Content-Type`). HTTP/2+ da headerlar lowercase ga aylantiriladi. Custom header'larni `X-` bilan boshlash eskirgan amaliyot (RFC 6648) — endi shart emas.

---

## Status kodlar

**💡 Tushuncha:** 3 xonali kod javob natijasini bildiradi. Birinchi raqam sinfni belgilaydi:

| Sinf | Ma'no | Misol |
|------|-------|-------|
| **1xx** | Informational | `100 Continue`, `101 Switching Protocols` |
| **2xx** | Success | `200`, `201`, `204` |
| **3xx** | Redirection | `301`, `302`, `304` |
| **4xx** | Client xatosi | `400`, `401`, `403`, `404`, `409`, `422`, `429` |
| **5xx** | Server xatosi | `500`, `502`, `503`, `504` |

Muhimlari:

- **200 OK** — muvaffaqiyatli (GET, PUT, PATCH).
- **201 Created** — resurs yaratildi (`POST`), `Location` header bilan.
- **204 No Content** — muvaffaqiyatli, lekin body yo'q (`DELETE`).
- **301 / 302** — doimiy / vaqtinchalik redirect.
- **304 Not Modified** — cache hali yaroqli (`ETag` bilan).
- **400 Bad Request** — noto'g'ri so'rov (sintaksis/format).
- **401 Unauthorized** — autentifikatsiya yo'q/noto'g'ri (aslida "unauthenticated").
- **403 Forbidden** — kim ekaningni bilamiz, lekin ruxsat yo'q.
- **404 Not Found** — resurs topilmadi.
- **409 Conflict** — holat ziddiyati (masalan, takroriy email).
- **422 Unprocessable Entity** — sintaksis to'g'ri, lekin validatsiya xatosi.
- **429 Too Many Requests** — rate limit oshib ketdi.
- **500 Internal Server Error** — kutilmagan server xatosi.
- **502 Bad Gateway** / **503 Service Unavailable** / **504 Gateway Timeout** — proxy/upstream muammolari.

**⚠️ Ehtiyot bo'l:** `401` aslida "autentifikatsiya muvaffaqiyatsiz" (kim ekaning noma'lum), `403` esa "kimligingni bilamiz, lekin bu amalga ruxsating yo'q". Ko'p dasturchilar bularni adashtiradi. Validatsiya xatosida `400` yoki `422` ishlatish mumkin — jamoa ichida bitta konvensiyaga kelishing kerak.

---

## HTTP/1.1 vs HTTP/2 vs HTTP/3 va keep-alive

**💡 Tushuncha:**

- **HTTP/1.1** — matnli protokol, har bir TCP ulanishda bir vaqtda bitta so'rov-javob. **keep-alive** (`Connection: keep-alive`) bir TCP ulanishni qayta ishlatishga imkon beradi (har request uchun yangi TCP handshake qilmaslik). Lekin **head-of-line (HOL) blocking** muammosi: bitta sekin javob keyingilarni bloklaydi. Brauzerlar buni 6 ta parallel ulanish bilan aylanib o'tadi.
- **HTTP/2** — **binary** protokol, **multiplexing**: bitta TCP ulanishda ko'plab so'rovlar bir vaqtda (stream'lar orqali). **Header compression** (HPACK), **server push**. Lekin TCP darajasida HOL blocking saqlanib qoladi (paket yo'qolsa, hammasi kutadi).
- **HTTP/3** — **QUIC** (UDP ustida) ishlatadi. TCP-darajadagi HOL blocking yo'q (har stream mustaqil). Tezroq connection o'rnatish (0-RTT/1-RTT), TLS 1.3 ichiga qurilgan, tarmoq almashganda (Wi-Fi → mobil) ulanish uzilmaydi (connection migration).

**⚠️ Ehtiyot bo'l:** HTTP/2 multiplexing TCP HOL blocking'ni **yechmaydi** — faqat application darajasidagi blocking'ni yechadi. Aynan shu sabab HTTP/3 UDP'ga o'tdi. Intervyuda "HTTP/2 nega yetarli emas edi?" degan savolga shu javob.

---

## REST prinsiplari

**💡 Tushuncha:** REST (Representational State Transfer) — Roy Fielding tomonidan ta'riflangan arxitektura uslubi. Asosiy cheklovlari:

1. **Client-Server** — UI va data alohida.
2. **Stateless** — har request o'zini to'liq tavsiflaydi; server session holatini saqlamaydi.
3. **Cacheable** — javoblar cache qilinishi mumkinligini bildirishi kerak.
4. **Uniform Interface** — yagona, izchil interfeys (resurslar, metodlar, representatsiyalar).
5. **Layered System** — client orada proxy/cache borligini bilmasligi mumkin.
6. **Resource-oriented** — hamma narsa **resurs** (`/users`, `/orders/5`), ular URL bilan identifikatsiya qilinadi, metodlar amalni bildiradi.

**Resurs** — ot (noun), URL'da ko'plikda: `/users`, `/users/5/orders`. Amal — HTTP metod (verb): `GET /users`, `POST /users`. URL'da fe'l ishlatma: ❌ `/getUsers`, ❌ `/createUser`.

**⚠️ Ehtiyot bo'l:** Statelessness — REST'ning eng ko'p buziladigan prinsipi. Agar server xotirasida session saqlasang (in-memory session), bu horizontal scaling'ni buzadi: ikkinchi server instance'i o'sha session'ni bilmaydi. Yechim — session'ni Redis/DB'da yoki tokenda (stateless JWT) saqlash.

---

## Richardson Maturity Model va HATEOAS

**💡 Tushuncha:** Leonard Richardson REST "yetuklik" darajalarini taklif qildi:

- **Level 0** — bitta URL, bitta metod (POST). Aslida RPC (masalan, SOAP).
- **Level 1 — Resources** — ko'p URL (resurslar), lekin metodlar noto'g'ri.
- **Level 2 — HTTP Verbs** — to'g'ri metodlar + status kodlar. Aksariyat "REST" API'lar shu yerda.
- **Level 3 — HATEOAS** — javoblar mavjud amallarning linklarini ham qaytaradi.

**HATEOAS** (Hypermedia As The Engine Of Application State):

```json
{
  "id": 5,
  "status": "pending",
  "_links": {
    "self":   { "href": "/orders/5" },
    "cancel": { "href": "/orders/5/cancel", "method": "POST" },
    "pay":    { "href": "/orders/5/pay", "method": "POST" }
  }
}
```

Client keyingi amallarni qattiq kodlamaydi — server linklarni beradi.

**⚠️ Ehtiyot bo'l:** HATEOAS go'zal nazariya, lekin amalda kam qo'llaniladi (qo'shimcha murakkablik, kam client kutubxonasi qo'llab-quvvatlaydi). Intervyuda "bizning API REST'mi?" deyilsa, ko'pchilik aslida **Level 2** ni nazarda tutadi. Buni bilish — yetuklik belgisi.

---

## RESTful endpoint dizayni

**💡 Tushuncha:** Yaxshi endpoint dizayni — izchillik (consistency) va bashoratlilik (predictability).

| Amal | Metod + URL |
|------|-------------|
| Ro'yxat | `GET /users` |
| Bitta | `GET /users/5` |
| Yaratish | `POST /users` |
| To'liq yangilash | `PUT /users/5` |
| Qisman yangilash | `PATCH /users/5` |
| O'chirish | `DELETE /users/5` |
| Ichki resurs | `GET /users/5/orders` |

Nomlash qoidalari:

- Ko'plik ot ishlatish: `/users`, `/products` (izchil).
- Kichik harf + `kebab-case`: `/order-items`, `/users` (URL'da camelCase emas).
- Fe'l yo'q: amal HTTP metod bilan beriladi.
- Ierarxiyani ichki yo'l bilan: `/users/5/orders/9`.
- Murakkab amallar (RPC-uslub) uchun "controller" pattern: `POST /orders/5/cancel` — ba'zan REST sof emasligini tan olib, amaliy yechim.

**⚠️ Ehtiyot bo'l:** Ichki resurslarni 2 darajadan chuqurlashtirma: `/users/5/orders/9/items/3/...` — bu murakkab va mo'rt. Yaxshisi top-level: `GET /order-items?orderId=9`.

---

## Versioning, pagination, filtering, sorting

**💡 Tushuncha:**

**Versioning** — API'ni buzuvchi (breaking) o'zgarishlardan himoya qilish:

```http
GET /v1/users          # URL-based (eng keng tarqalgan)
Accept: application/vnd.api.v2+json   # Header-based
```

**Pagination** — katta ro'yxatlarni bo'lib berish:

```http
# Offset-based (oddiy, lekin katta offsetda sekin)
GET /users?page=2&limit=20
GET /users?offset=40&limit=20

# Cursor-based (katta datasetlar uchun barqaror, tez)
GET /users?limit=20&cursor=eyJpZCI6MTAwfQ
```

**Filtering & sorting**:

```http
GET /users?role=admin&status=active   # filter
GET /users?sort=-createdAt,name       # sort (- = teskari)
GET /products?price[gte]=100&price[lte]=500   # range filter
```

**⚠️ Ehtiyot bo'l:** Offset pagination katta sahifalarda (`OFFSET 1000000`) DB'da juda sekin, va yangi yozuv qo'shilsa elementlar siljiydi (dublikat/o'tkazib yuborish). Cheksiz scroll va real-time ro'yxatlar uchun **cursor-based** pagination ishlat. Har doim `limit` ga maksimum qo'y (masalan, max 100), aks holda DoS xavfi.

---

## Request/Response body va content negotiation

**💡 Tushuncha:** **Content negotiation** — client va server qaysi formatda gaplashishini kelishadi. Client `Accept` header bilan nimani xohlashini, server `Content-Type` bilan nima yuborayotganini aytadi.

```http
GET /users/5 HTTP/1.1
Accept: application/json

HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

{ "id": 5, "name": "Ali" }
```

Agar server `Accept` dagi formatni qo'llab-quvvatlamasa → `406 Not Acceptable`. Agar request body `Content-Type` qo'llab-quvvatlanmasa → `415 Unsupported Media Type`.

**⚠️ Ehtiyot bo'l:** JSON eng keng tarqalgan, lekin u sana (`Date`), `BigInt`, `undefined` ni to'g'ri ko'chira olmaydi. Sanalarni ISO 8601 string sifatida (`"2026-06-30T10:00:00Z"`) yubor. Katta sonlarni (ID'lar) string sifatida yubor, aks holda JS `Number` aniqlikni yo'qotadi (`> 2^53`).

---

## Express'da API qurish

**💡 Tushuncha:** Express — Node.js'da eng mashhur minimal web framework. Asoslari: **routing**, **middleware**, **error handling**.

```ts
import express, { Request, Response, NextFunction } from "express";

const app = express();

// 1. Body parsing middleware
app.use(express.json({ limit: "1mb" }));

// 2. Custom middleware (logging)
app.use((req: Request, _res: Response, next: NextFunction) => {
  console.log(`${req.method} ${req.path}`);
  next(); // keyingisiga uzatish — UNUTMA!
});

// 3. Routes
app.get("/users/:id", async (req, res, next) => {
  try {
    const user = await db.findUser(req.params.id);
    if (!user) return res.status(404).json({ error: "Not found" });
    res.json(user);
  } catch (err) {
    next(err); // xatoni error middleware'ga uzatish
  }
});

app.post("/users", async (req, res, next) => {
  try {
    const user = await db.createUser(req.body);
    res.status(201).location(`/users/${user.id}`).json(user);
  } catch (err) {
    next(err);
  }
});

// 4. Error-handling middleware — OXIRIDA, 4 ta argument bilan!
app.use((err: any, _req: Request, res: Response, _next: NextFunction) => {
  console.error(err);
  const status = err.status ?? 500;
  res.status(status).json({ error: err.message ?? "Internal Server Error" });
});

app.listen(3000);
```

**⚠️ Ehtiyot bo'l:**

- Error middleware **4 ta argument** (`err, req, res, next`) bilan aniqlanishi SHART — Express buni signaturadan ajratadi. 3 ta yozsang, oddiy middleware bo'lib qoladi.
- Express 4'da `async` route'dagi `throw` avtomatik ushlanmaydi — `try/catch` + `next(err)` qil yoki wrapper ishlat. (Express 5 buni avtomatik qiladi.)
- `next()` chaqirmasang request "osilib" qoladi (timeout).
- Middleware tartibi muhim: `express.json()` route'lardan oldin, error handler eng oxirida.

---

## Validation

**💡 Tushuncha:** Tashqaridan kelgan har qanday ma'lumotga **ishonma**. Validation — request body/query/params'ni biznes-mantiqqa o'tkazishdan oldin tekshirish. Zod, Joi, yoki class-validator ishlatiladi.

```ts
import { z } from "zod";

const CreateUserSchema = z.object({
  name: z.string().min(2).max(50),
  email: z.string().email(),
  age: z.number().int().positive().optional(),
});

function validate(schema: z.ZodSchema) {
  return (req: Request, res: Response, next: NextFunction) => {
    const result = schema.safeParse(req.body);
    if (!result.success) {
      return res.status(422).json({ errors: result.error.flatten() });
    }
    req.body = result.data; // tozalangan, tiplangan data
    next();
  };
}

app.post("/users", validate(CreateUserSchema), createUserHandler);
```

**⚠️ Ehtiyot bo'l:** Faqat frontend validatsiyasiga tayanma — uni har kim aylanib o'tishi mumkin (curl, Postman). Validatsiya **har doim server tomonida** bo'lishi shart. Schema'ni "strict" qil (`.strict()`), aks holda kutilmagan maydonlar o'tib ketadi (mass-assignment xavfi — masalan `isAdmin: true`).

---

## CORS

**💡 Tushuncha:** CORS (Cross-Origin Resource Sharing) — brauzerning **Same-Origin Policy** himoyasini boshqaruvchi mexanizm. Origin = `protocol + host + port`. Brauzer boshqa origin'ga so'rov yuborganda, server `Access-Control-Allow-Origin` header bilan ruxsat berishi kerak, aks holda brauzer javobni bloklaydi.

**Preflight** — "murakkab" so'rovlardan (`PUT`, `DELETE`, custom header, yoki `application/json` POST) oldin brauzer `OPTIONS` so'rovi yuboradi:

```http
OPTIONS /api/users HTTP/1.1
Origin: https://app.example.com
Access-Control-Request-Method: POST
Access-Control-Request-Headers: Content-Type, Authorization

HTTP/1.1 204 No Content
Access-Control-Allow-Origin: https://app.example.com
Access-Control-Allow-Methods: POST, GET, OPTIONS
Access-Control-Allow-Headers: Content-Type, Authorization
Access-Control-Allow-Credentials: true
```

Express'da:

```ts
import cors from "cors";
app.use(cors({
  origin: ["https://app.example.com"],
  credentials: true,
}));
```

**⚠️ Ehtiyot bo'l:**

- CORS — bu **brauzer** himoyasi, server himoyasi EMAS. curl/Postman CORS'ni e'tiborsiz qoldiradi. U serverni himoya qilmaydi, balki foydalanuvchi brauzerini boshqa saytlar nomidan so'rov yuborishdan to'sadi.
- `Access-Control-Allow-Origin: *` va `credentials: true` ni birga ishlatib bo'lmaydi (brauzer rad etadi). Cookie/auth bilan ishlasang, aniq origin ko'rsat.
- CORS xatosi `4xx`/`5xx` emas — javob keladi, lekin brauzer JS'ga ko'rsatmaydi.

---

## Rate limiting

**💡 Tushuncha:** Rate limiting — bir client (IP/user/API key) ma'lum vaqt oynasida nechta so'rov yuborishini cheklaydi. Abuse, brute-force, va DoS'dan himoya qiladi. Limit oshsa → `429 Too Many Requests` + `Retry-After`.

Algoritmlar: **Fixed window**, **Sliding window**, **Token bucket** (eng moslashuvchan — burst'ga ruxsat berib, o'rtacha tezlikni cheklaydi).

```ts
import rateLimit from "express-rate-limit";

const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 daqiqa
  max: 100,                  // har IP uchun 100 so'rov
  standardHeaders: true,     // RateLimit-* headerlar
  message: { error: "Too many requests" },
});

app.use("/api/", limiter);
```

**⚠️ Ehtiyot bo'l:** In-memory rate limiter ko'p server instance'da ishlamaydi (har biri o'z hisobini yuritadi). Distributed muhitda **Redis** asosida limiter ishlat. Login/parol tiklash kabi sezgir endpoint'larga alohida, qattiqroq limit qo'y.

---

## Idempotency key

**💡 Tushuncha:** `POST` idempotent emas — tarmoq xatosida client qayta yuborsa, ikki marta to'lov bo'lishi mumkin. **Idempotency key** — client unikal kalit (`Idempotency-Key: <uuid>`) yuboradi; server bu kalitni saqlaydi va takror so'rovda yangi amal bajarmasdan birinchi natijani qaytaradi.

```http
POST /payments HTTP/1.1
Idempotency-Key: 7a1f9c2e-...
Content-Type: application/json

{ "amount": 5000, "currency": "UZS" }
```

```ts
app.post("/payments", async (req, res) => {
  const key = req.header("Idempotency-Key");
  if (!key) return res.status(400).json({ error: "Idempotency-Key required" });

  const cached = await store.get(key);
  if (cached) return res.status(cached.status).json(cached.body); // takror

  const result = await processPayment(req.body);
  await store.set(key, { status: 201, body: result }, { ttl: 86400 });
  res.status(201).json(result);
});
```

**⚠️ Ehtiyot bo'l:** Stripe, PayPal kabi to'lov tizimlari aynan shu pattern'ni ishlatadi. Kalitni atomar saqlash kerak (race condition: ikki bir vaqtdagi so'rov) — DB unique constraint yoki Redis `SET NX` ishlat.

---

## REST vs GraphQL vs gRPC vs tRPC

**💡 Tushuncha:**

| Mezon | REST | GraphQL | gRPC | tRPC |
|-------|------|---------|------|------|
| Format | JSON/HTTP | JSON, bitta endpoint | Protobuf/HTTP2 | TS, RPC |
| Schema | OpenAPI (ixtiyoriy) | Strong (SDL) | Strong (`.proto`) | TS tiplar |
| Over/under-fetch | Bor | Yo'q (client tanlaydi) | Kam | Kam |
| Tezlik | O'rtacha | O'rtacha | Eng tez (binary) | Tez |
| Brauzer | Tabiiy | Tabiiy | Cheklangan (grpc-web) | Faqat TS |
| Eng yaxshi | Public API, CRUD | Murakkab, har xil client | Mikroservis-ichi | TS monorepo (full-stack) |

- **REST** — universal, oddiy, cache'lash oson, public API uchun standart.
- **GraphQL** — client kerakli maydonlarni so'raydi (over/under-fetch yo'q), lekin caching va rate limiting murakkab.
- **gRPC** — binary, juda tez, strongly-typed; mikroservislar orasida ideal, lekin brauzerda noqulay.
- **tRPC** — TypeScript end-to-end type-safety, kod generatsiyasiz; faqat TS monorepo'da.

**⚠️ Ehtiyot bo'l:** "GraphQL har doim yaxshiroq" — noto'g'ri. GraphQL caching (HTTP-darajada), N+1 query, va so'rov murakkabligi cheklash bo'yicha qo'shimcha mehnat talab qiladi. Oddiy CRUD uchun REST yetarli va arzonroq.

---

## WebSocket vs SSE vs polling

**💡 Tushuncha:** Real-time ma'lumot yetkazishning 3 usuli:

| Usul | Yo'nalish | Protokol | Eng yaxshi |
|------|-----------|----------|------------|
| **Polling** | Client → Server (takror) | HTTP | Oddiy, kam o'zgaruvchan data |
| **Long-polling** | Client kutadi, server javob beradi | HTTP | Eski brauzer fallback |
| **SSE** | Server → Client (bir tomon) | HTTP (`text/event-stream`) | Feed, notifikatsiya, narx |
| **WebSocket** | Ikki tomonlama (full-duplex) | `ws://`/`wss://` | Chat, o'yin, hamkorlik |

- **Polling** — client har N soniyada so'raydi. Oddiy, lekin samarasiz (ko'p bo'sh so'rov).
- **SSE** — server bir tomonlama oqim yuboradi, avtomatik qayta ulanish, HTTP ustida. Faqat server→client.
- **WebSocket** — bitta doimiy ulanish, ikki tomonlama. Chat, multiplayer o'yin, jonli kursorlar uchun.

**⚠️ Ehtiyot bo'l:** Real-time har doim WebSocket kerak emas. Agar faqat server→client kerak bo'lsa (jonli narx, notifikatsiya), **SSE** soddaroq: oddiy HTTP, proxy/load-balancer bilan yaxshi ishlaydi, avtomatik reconnect bor. WebSocket sticky-session va alohida infratuzilma talab qiladi.

---

## OpenAPI

**💡 Tushuncha:** OpenAPI (avvalgi Swagger) — REST API'ni mashina o'qiy oladigan formatda (YAML/JSON) tavsiflovchi standart. Undan avtomatik: hujjat (Swagger UI), client SDK, server stub, va validatsiya generatsiya qilinadi.

```yaml
openapi: 3.0.0
info:
  title: Users API
  version: 1.0.0
paths:
  /users/{id}:
    get:
      summary: Get a user
      parameters:
        - name: id
          in: path
          required: true
          schema: { type: integer }
      responses:
        "200":
          description: OK
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/User"
        "404":
          description: Not found
components:
  schemas:
    User:
      type: object
      properties:
        id:   { type: integer }
        name: { type: string }
```

**⚠️ Ehtiyot bo'l:** OpenAPI'ni qo'lda yozish va kodni alohida saqlash → ular sinxrondan chiqib ketadi. Yaxshisi — kod yoki Zod schemalardan OpenAPI generatsiya qil (`zod-to-openapi`), shunda yagona haqiqat manbai (single source of truth) bo'ladi.

---

## Intervyu savollari (Q&A)

### ❓ HTTP "stateless" degani nima va bu nima uchun muhim?

**✅ Javob:** Server avvalgi so'rovlarni eslab qolmaydi — har request o'zini (auth, kontekst) to'liq olib kelishi kerak. Bu horizontal scaling'ni osonlashtiradi: har qanday server instance'i har qanday request'ni qabul qila oladi, chunki holat request'da yoki tashqi store'da (Redis/DB) saqlanadi.

### ❓ Idempotency nima? Qaysi metodlar idempotent?

**✅ Javob:** Idempotent — so'rovni bir necha marta yuborganda server holati bir xil qoladi. `GET`, `PUT`, `DELETE`, `HEAD`, `OPTIONS` idempotent. `POST` va (umumiy holatda) `PATCH` idempotent emas. Bu retry mexanizmida muhim: idempotent so'rovni xavfsiz qayta yuborish mumkin.

### ❓ `PUT` va `PATCH` farqi nima?

**✅ Javob:** `PUT` — resursni **to'liq almashtiradi** (yuborilmagan maydonlar o'chadi/null bo'ladi), idempotent. `PATCH` — **qisman yangilash** (faqat yuborilgan maydonlar), umumiy holatda idempotent emas. `PUT /users/5` da butun user obyektini yuborasan; `PATCH /users/5` da `{ "name": "Yangi" }` yetarli.

### ❓ `401` va `403` farqi?

**✅ Javob:** `401 Unauthorized` — autentifikatsiya yo'q yoki noto'g'ri (kim ekaning noma'lum). `403 Forbidden` — kim ekaning aniq, lekin bu amalga ruxsating yo'q. `401` da login qil; `403` da loginning foydasi yo'q.

### ❓ HTTP/2 multiplexing TCP HOL blocking'ni yechadimi?

**✅ Javob:** Yo'q. HTTP/2 application-darajadagi HOL blocking'ni yechadi (bitta TCP'da parallel stream'lar), lekin TCP paketi yo'qolsa, TCP barcha stream'larni bloklaydi. Aynan shu sabab HTTP/3 QUIC (UDP) ga o'tdi — u har stream'ni mustaqil qiladi.

### ❓ REST'da statelessness qanday horizontal scaling'ga yordam beradi?

**✅ Javob:** Server in-memory session saqlamasa, load balancer request'ni istalgan instance'ga yo'naltira oladi (sticky session shart emas). Yangi instance qo'shish/o'chirish oson. Holat tashqarida (JWT, Redis) saqlanadi.

### ❓ Offset vs cursor pagination — qachon qaysi biri?

**✅ Javob:** Offset (`page/limit`) — oddiy, ixtiyoriy sahifaga sakrash mumkin, lekin katta offsetda sekin va data o'zgarsa siljish bo'ladi. Cursor — barqaror va tez (indeksli ustun bo'yicha `WHERE id > cursor`), cheksiz scroll va real-time feed uchun, lekin ixtiyoriy sahifaga sakrab bo'lmaydi.

### ❓ CORS serverni himoya qiladimi?

**✅ Javob:** Yo'q. CORS — brauzer mexanizmi, u foydalanuvchi brauzerini zararli saytlar nomidan so'rov yuborishdan to'sadi. curl/Postman/server-to-server so'rovlar CORS'ni e'tiborsiz qoldiradi. Server xavfsizligi uchun auth, validation, rate limiting kerak.

### ❓ Preflight so'rov nima va qachon yuboriladi?

**✅ Javob:** Brauzer "murakkab" cross-origin so'rovdan (`PUT`/`DELETE`, custom header, yoki non-simple `Content-Type`) oldin `OPTIONS` so'rovi yuboradi — serverdan ruxsat so'raydi. Server `Access-Control-Allow-*` headerlar bilan javob bersa, asosiy so'rov yuboriladi.

### ❓ POST'ni xavfsiz retry qilib bo'ladimi?

**✅ Javob:** O'zicha yo'q (idempotent emas). Lekin **idempotency key** (`Idempotency-Key` header) bilan: server kalitni saqlaydi, takror so'rovda yangi amal qilmasdan birinchi natijani qaytaradi. To'lovlarda muhim.

### ❓ Express error-handling middleware'ni qanday aniqlaysan?

**✅ Javob:** **4 ta argument** bilan: `(err, req, res, next)`. Express buni signaturadan tanib oladi. U boshqa route/middleware'lardan KEYIN, eng oxirida ro'yxatdan o'tishi kerak. Route'larda xato yuz berganda `next(err)` chaqirilsa, bu middleware ishga tushadi.

### ❓ REST vs GraphQL — qachon GraphQL?

**✅ Javob:** GraphQL — turli xil client'lar turli ma'lumot kombinatsiyalarini so'raganda (over/under-fetch muammosi), murakkab bog'langan grafiklarda foydali. REST — oddiy CRUD, public API, kuchli HTTP caching kerak bo'lganda. GraphQL caching, rate limiting, N+1 bo'yicha qo'shimcha murakkablik keltiradi.

### ❓ WebSocket o'rniga qachon SSE?

**✅ Javob:** Agar faqat **server→client** bir tomonlama oqim kerak bo'lsa (notifikatsiya, jonli narx, feed) — SSE soddaroq: oddiy HTTP, avtomatik reconnect, proxy bilan oson. Ikki tomonlama interaktivlik kerak bo'lsa (chat, o'yin) — WebSocket.

### ❓ `200`, `201`, `204` qachon qaytariladi?

**✅ Javob:** `200 OK` — muvaffaqiyatli javob body bilan (GET, PUT, PATCH). `201 Created` — `POST` resurs yaratdi, `Location` header bilan. `204 No Content` — muvaffaqiyatli, lekin body yo'q (`DELETE`, yoki body qaytarishga hojat yo'q PUT).

### ❓ Validatsiyani nega faqat frontendda qilib bo'lmaydi?

**✅ Javob:** Frontend validatsiyasini har kim aylanib o'tishi mumkin (curl, Postman, DevTools). Validatsiya har doim **server tomonida** majburiy. Frontend — faqat UX uchun (tez fikr-mulohaza), xavfsizlik chegarasi emas.

---

## Masalalar

> Yechimlar: [solutions/backend/07-http-rest-apis.md](../solutions/backend/07-http-rest-apis.md)

1. **Status kod tanlash.** Quyidagi holatlar uchun to'g'ri HTTP metod va status kodni yoz: (a) yangi user yaratish muvaffaqiyatli; (b) mavjud bo'lmagan user'ni o'chirish; (c) takroriy email bilan ro'yxatdan o'tish; (d) noto'g'ri JSON body; (e) auth token muddati o'tgan; (f) admin emas user admin endpoint'ga kirgan.

2. **RESTful redizayn.** Quyidagi noto'g'ri endpoint'larni RESTful ko'rinishga keltir: `GET /getUserOrders?userId=5`, `POST /createOrder`, `POST /deleteOrder?id=9`, `GET /user/listAll`, `POST /order/9/setStatusCancelled`.

3. **Idempotent to'lov endpoint.** Express'da `POST /payments` endpoint yoz: `Idempotency-Key` header'ni tekshiradi, takror so'rovda avvalgi natijani qaytaradi, race condition'ni inobatga oladi (Redis `SET NX` yoki DB unique constraint mantiqini tasvirla).

4. **Cursor pagination.** `GET /messages?limit=20&cursor=<id>` endpoint'ini loyihala. Response qanday ko'rinishda bo'lishi kerak (keyingi sahifa kursori bilan)? Offset pagination'dan qaysi afzalliklari bor — yoz.

5. **CORS sozlash.** Frontend `https://app.example.com` dan, API `https://api.example.com` da. Cookie-asosli auth ishlatiladi. To'g'ri CORS konfiguratsiyasini (Express `cors`) yoz va nega `origin: "*"` ishlamasligini tushuntir.

6. **Validation middleware.** Zod bilan `POST /products` uchun validatsiya middleware yoz: `name` (3-100 belgi), `price` (musbat son), `tags` (string array, max 10), `category` (enum). Xatoda `422` qaytarsin, ortiqcha maydonlarni rad etsin.

7. **Rate limiter dizayni.** Login endpoint'i uchun rate limiting strategiyasini loyihala: IP bo'yicha umumiy limit + email bo'yicha qattiqroq limit. Distributed muhitda nega Redis kerakligini tushuntir. Token bucket algoritmini qisqa tasvirla.

8. **Protokol tanlash.** Quyidagi ssenariylar uchun REST/GraphQL/gRPC/tRPC/WebSocket/SSE dan qaysi birini tanlaysan va nega: (a) public to'lov API; (b) ikki mikroservis orasidagi yuqori tezlikli aloqa; (c) jonli chat ilovasi; (d) mobil app uchun moslashuvchan ma'lumot olish; (e) TypeScript monorepo full-stack ilova; (f) birja narxlarini jonli ko'rsatish.

9. **Express middleware tartibi.** Quyidagi middleware'larni to'g'ri tartibda joylashtir va nega: error handler, body parser, auth, CORS, rate limiter, route'lar, request logger. Tartib noto'g'ri bo'lsa nima buziladi?

10. **OpenAPI spec.** `GET /orders/{id}` va `POST /orders` uchun minimal OpenAPI 3.0 spetsifikatsiyasi yoz: parametrlar, request body schema, `200`/`201`/`404` javoblar va `Order` schema bilan.

---

← [Backend bo'limiga qaytish](./README.md)
