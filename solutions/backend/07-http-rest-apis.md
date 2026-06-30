# HTTP va REST API — Masalalar Yechimi

Bu fayl [`backend/07-http-rest-apis.md`](../../backend/07-http-rest-apis.md) dagi "Masalalar" bo'limining yechimlarini o'z ichiga oladi.

## Mundarija

- [1. Status kod tanlash](#1-status-kod-tanlash)
- [2. RESTful redizayn](#2-restful-redizayn)
- [3. Idempotent to'lov endpoint](#3-idempotent-tolov-endpoint)
- [4. Cursor pagination](#4-cursor-pagination)
- [5. CORS sozlash](#5-cors-sozlash)
- [6. Validation middleware](#6-validation-middleware)
- [7. Rate limiter dizayni](#7-rate-limiter-dizayni)
- [8. Protokol tanlash](#8-protokol-tanlash)
- [9. Express middleware tartibi](#9-express-middleware-tartibi)
- [10. OpenAPI spec](#10-openapi-spec)

---

## 1. Status kod tanlash

| Holat | Metod | Status |
|-------|-------|--------|
| (a) Yangi user yaratish muvaffaqiyatli | `POST /users` | `201 Created` (+ `Location`) |
| (b) Mavjud bo'lmagan user'ni o'chirish | `DELETE /users/5` | `404 Not Found` (yoki idempotent `204`) |
| (c) Takroriy email | `POST /users` | `409 Conflict` |
| (d) Noto'g'ri JSON body | — | `400 Bad Request` |
| (e) Token muddati o'tgan | — | `401 Unauthorized` |
| (f) Admin emas user admin endpoint'ga | — | `403 Forbidden` |

**Izoh:** (b) da ikki yondashuv bor — RESTful idempotency bo'yicha "yo'q resursni o'chirish" ni `204` deb qaytarish mumkin (natija bir xil: resurs yo'q), lekin ko'p jamoalar aniqlik uchun `404` ni afzal ko'radi. Validatsiya xatosi (d) uchun `422` ham to'g'ri, agar sintaksis to'g'ri-yu semantik xato bo'lsa; noto'g'ri JSON (parse xatosi) uchun `400` aniqroq.

---

## 2. RESTful redizayn

| Noto'g'ri | To'g'ri |
|-----------|---------|
| `GET /getUserOrders?userId=5` | `GET /users/5/orders` |
| `POST /createOrder` | `POST /orders` |
| `POST /deleteOrder?id=9` | `DELETE /orders/9` |
| `GET /user/listAll` | `GET /users` |
| `POST /order/9/setStatusCancelled` | `POST /orders/9/cancel` (controller pattern) yoki `PATCH /orders/9` `{ "status": "cancelled" }` |

**Prinsiplar:** resurs nomi ot va ko'plikda (`/users`, `/orders`); amal HTTP metod bilan; URL'da fe'l yo'q; ierarxiya ichki yo'l bilan (`/users/5/orders`). Status o'zgartirish kabi "harakat" amallarini ikki xil qilish mumkin: sof REST'da `PATCH` orqali holatni yangilash, yoki amaliy "controller" pattern'da `POST /orders/9/cancel`.

---

## 3. Idempotent to'lov endpoint

```ts
import { Request, Response } from "express";
import { redis } from "./redis";

app.post("/payments", async (req: Request, res: Response, next) => {
  try {
    const key = req.header("Idempotency-Key");
    if (!key) return res.status(400).json({ error: "Idempotency-Key required" });

    // 1. Avval natija saqlanganmi?
    const cached = await redis.get(`idem:${key}`);
    if (cached) {
      const { status, body } = JSON.parse(cached);
      return res.status(status).json(body);
    }

    // 2. Atomar "lock" — race condition'dan himoya (SET NX)
    const locked = await redis.set(`lock:${key}`, "1", "NX", "EX", 30);
    if (!locked) {
      return res.status(409).json({ error: "Request already in progress" });
    }

    // 3. To'lovni amalga oshirish
    const result = await processPayment(req.body);

    // 4. Natijani 24 soatga saqlash
    await redis.set(`idem:${key}`, JSON.stringify({ status: 201, body: result }), "EX", 86400);
    await redis.del(`lock:${key}`);

    res.status(201).json(result);
  } catch (err) {
    next(err);
  }
});
```

**Race condition mantiqi:** Ikki bir vaqtdagi so'rov bir xil kalit bilan kelsa, `SET ... NX` faqat birinchisiga muvaffaqiyat beradi (atomar). Ikkinchisi `409` oladi yoki birinchisi tugashini kutadi. Muqobil — DB'da `idempotency_key` ustuniga `UNIQUE` constraint qo'yib, ikkinchi `INSERT` xato berishiga tayanish.

---

## 4. Cursor pagination

```http
GET /messages?limit=20&cursor=eyJpZCI6MTQyfQ
```

Response:

```json
{
  "data": [ { "id": 143, "text": "..." }, { "id": 144, "text": "..." } ],
  "pageInfo": {
    "nextCursor": "eyJpZCI6MTYyfQ",
    "hasNextPage": true
  }
}
```

Cursor odatda oxirgi elementning indeksli ustuni (`id` yoki `createdAt`) base64'da kodlangani. DB so'rovi:

```sql
SELECT * FROM messages WHERE id > :cursorId ORDER BY id ASC LIMIT :limit + 1;
-- limit+1 ta olib, oxirgisi bor-yo'qligidan hasNextPage aniqlanadi
```

**Offset'dan afzalliklari:** (1) **tezlik** — `WHERE id > X` indeksdan foydalanadi, `OFFSET 100000` esa 100000 qatorni o'qib tashlaydi; (2) **barqarorlik** — yangi yozuv qo'shilsa, offset'da elementlar siljiydi (dublikat/o'tkazib yuborish), cursor'da bunday bo'lmaydi. Kamchiligi — ixtiyoriy sahifaga ("7-sahifa") sakrab bo'lmaydi.

---

## 5. CORS sozlash

```ts
import cors from "cors";

app.use(cors({
  origin: "https://app.example.com",  // aniq origin, "*" emas
  credentials: true,                   // cookie yuborishga ruxsat
  methods: ["GET", "POST", "PUT", "PATCH", "DELETE"],
  allowedHeaders: ["Content-Type", "Authorization"],
}));
```

**Nega `origin: "*"` ishlamaydi:** Brauzer spetsifikatsiyasi bo'yicha `Access-Control-Allow-Credentials: true` (cookie yuborish) bilan `Access-Control-Allow-Origin: *` ni birga ishlatish **taqiqlangan** — bu xavfsizlik teshigi bo'lardi (har qanday sayt cookie bilan so'rov yubora olishi). Shuning uchun credentials bilan aniq origin ko'rsatish shart. Cookie esa `Secure; SameSite=None` bo'lishi kerak (cross-site yuborilishi uchun).

---

## 6. Validation middleware

```ts
import { z } from "zod";
import { Request, Response, NextFunction } from "express";

const ProductSchema = z.object({
  name: z.string().min(3).max(100),
  price: z.number().positive(),
  tags: z.array(z.string()).max(10),
  category: z.enum(["food", "electronics", "clothing"]),
}).strict(); // ortiqcha maydonlarni rad etadi

function validate(schema: z.ZodSchema) {
  return (req: Request, res: Response, next: NextFunction) => {
    const result = schema.safeParse(req.body);
    if (!result.success) {
      return res.status(422).json({ errors: result.error.flatten().fieldErrors });
    }
    req.body = result.data;
    next();
  };
}

app.post("/products", validate(ProductSchema), createProductHandler);
```

`.strict()` — sxemada e'lon qilinmagan maydon kelsa (`isAdmin`, `id`...) xato beradi, mass-assignment xavfini oldini oladi.

---

## 7. Rate limiter dizayni

**Strategiya — ikki qatlam:**

1. **IP bo'yicha umumiy limit**: masalan 15 daqiqada 100 so'rov — keng botnet/skan'dan.
2. **Email bo'yicha qattiqroq limit**: masalan 15 daqiqada 5 muvaffaqiyatsiz urinish — bitta hisobga credential-stuffing'dan. Muvaffaqiyatli loginда hisoblagich tozalanadi.

```
key1 = rate:ip:<ip>          -> 100 / 15min
key2 = rate:login:<email>    -> 5 failed / 15min
```

**Nega Redis:** Bir nechta server instance bo'lsa, har biri o'z xotirasida hisoblagich yuritsa, umumiy limit buziladi (3 server → amalda 3× limit). Redis — markazlashgan, atomar (`INCR`, `EXPIRE`) hisoblagich beradi, barcha instance'lar bir manbadan o'qiydi.

**Token bucket:** Har client uchun "chelak" bor, unda token'lar to'planadi (masalan, sekundiga 10 ta, max 100). Har so'rov 1 token sarflaydi. Token tugasa → `429`. Chelak burst'ga ruxsat beradi (max gacha to'plangan), lekin uzoq muddatda o'rtacha tezlikni cheklaydi.

---

## 8. Protokol tanlash

| Ssenariy | Tanlov | Sabab |
|----------|--------|-------|
| (a) Public to'lov API | **REST** | Universal, standart, hujjatlash oson, har qanday client ishlatadi |
| (b) Mikroservislar orasi yuqori tezlik | **gRPC** | Binary protobuf, HTTP/2, eng past latency, strong typing |
| (c) Jonli chat | **WebSocket** | Ikki tomonlama, doimiy ulanish, past latency |
| (d) Mobil app moslashuvchan data | **GraphQL** | Client kerakli maydonlarni so'raydi, over/under-fetch yo'q, mobil trafik tejaladi |
| (e) TS monorepo full-stack | **tRPC** | End-to-end type-safety, kod generatsiyasiz |
| (f) Birja narxlarini jonli | **SSE** | Faqat server→client oqim, oddiy HTTP, auto-reconnect |

---

## 9. Express middleware tartibi

To'g'ri tartib:

```ts
app.use(cors());                 // 1. CORS — eng birinchi (preflight uchun)
app.use(express.json());         // 2. Body parser — route'lardan oldin
app.use(requestLogger);          // 3. Logger — barcha so'rovni ko'rish uchun
app.use(rateLimiter);            // 4. Rate limiter — qimmat ishdan oldin
app.use(authMiddleware);         // 5. Auth — himoyalangan route'lardan oldin
app.use("/api", routes);         // 6. Route'lar
app.use(errorHandler);           // 7. Error handler — ENG OXIRIDA, 4 arg
```

**Nima buziladi:**

- `errorHandler` route'lardan oldin bo'lsa — xatolarni ushlamaydi (Express oxirgi 4-argumentli middleware'ni qidiradi).
- `express.json()` route'lardan keyin bo'lsa — `req.body` `undefined` bo'ladi.
- `cors` kech bo'lsa — preflight `OPTIONS` so'rovlar bloklanadi.
- `authMiddleware` rate limiter'dan oldin bo'lsa — autentifikatsiyasiz so'rovlar ham qimmat auth tekshiruvini ishga soladi (DoS yuzasi). Odatda rate limit auth'dan oldin.

---

## 10. OpenAPI spec

```yaml
openapi: 3.0.0
info:
  title: Orders API
  version: 1.0.0
paths:
  /orders/{id}:
    get:
      summary: Get an order by ID
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
              schema: { $ref: "#/components/schemas/Order" }
        "404":
          description: Order not found
  /orders:
    post:
      summary: Create an order
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              required: [productId, quantity]
              properties:
                productId: { type: integer }
                quantity:  { type: integer, minimum: 1 }
      responses:
        "201":
          description: Created
          content:
            application/json:
              schema: { $ref: "#/components/schemas/Order" }
components:
  schemas:
    Order:
      type: object
      properties:
        id:        { type: integer }
        productId: { type: integer }
        quantity:  { type: integer }
        status:    { type: string, enum: [pending, paid, cancelled] }
```

---

← [Backend bo'limiga qaytish](../../backend/README.md)
