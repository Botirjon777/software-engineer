# Auth va Xavfsizlik — Masalalar Yechimi

Bu fayl [`backend/08-auth-security.md`](../../backend/08-auth-security.md) dagi "Masalalar" bo'limining yechimlarini o'z ichiga oladi.

## Mundarija

- [1. Auth oqimini loyihalash](#1-auth-oqimini-loyihalash)
- [2. Zaiflikni top](#2-zaiflikni-top)
- [3. Parol hashlash](#3-parol-hashlash)
- [4. CSRF himoyasi](#4-csrf-himoyasi)
- [5. OAuth2 flow chizmasi](#5-oauth2-flow-chizmasi)
- [6. JWT tekshirish middleware](#6-jwt-tekshirish-middleware)
- [7. RBAC tizimi](#7-rbac-tizimi)
- [8. Cookie sozlamalari](#8-cookie-sozlamalari)
- [9. Rate limiter strategiyasi](#9-rate-limiter-strategiyasi)
- [10. Input validatsiya va sanitizatsiya](#10-input-validatsiya-va-sanitizatsiya)

---

## 1. Auth oqimini loyihalash

**Tokenlar:**
- **Access token** — JWT, 15 daqiqa muddat. `Authorization: Bearer` header'da yuboriladi (yoki `HttpOnly` cookie).
- **Refresh token** — 7 kun, DB'da `userId`, `expiresAt`, `revoked` bilan saqlanadi. Faqat `/auth/refresh` endpoint'da ishlatiladi.

**Saqlash:** Ikkalasini ham `HttpOnly; Secure; SameSite=Strict` cookie'da (XSS'dan himoya). Agar SPA cross-origin bo'lsa, `SameSite=None; Secure` + CSRF token.

**Logout:** Refresh token'ni DB'da `revoked=true` qil, cookie'larni tozala. Access token muddati tugaguncha (15 daqiqa) yaroqli qoladi — bu maqbul zarar oynasi.

**Refresh rotation:** Har `/refresh`da eski refresh'ni bekor qilib, yangi access+refresh ber. Agar bekor qilingan (qayta ishlatilgan) refresh kelsa → o'g'irlik signali, foydalanuvchining barcha tokenlarini bekor qil.

**Asoslash:** Qisqa access = o'g'irlik zarari kam. DB'dagi refresh = darhol bekor qilish imkoni. Rotation = o'g'irlikni aniqlash. HttpOnly = XSS himoyasi.

---

## 2. Zaiflikni top

Muammolar:

1. **SQL injection** — `email` to'g'ridan stringga qo'shilgan.
2. **Plaintext parol taqqoslash** — parol hash'lanmagan saqlangan va `===` bilan solishtirilgan.
3. **JWT payload'da parol** — `password` token ichida (base64, hamma o'qiydi).
4. **Hardcoded secret** — `"secret123"` kodda, zaif.
5. **Information leak** — xato xabarida parol oshkor (`No user with password ...`).
6. **Timing/enumeration** — email mavjudligini ajratib bo'ladi.

Tuzatilgan:

```ts
app.post("/login", loginLimiter, async (req, res, next) => {
  try {
    const { email, password } = loginSchema.parse(req.body); // validation
    const user = await db.query("SELECT * FROM users WHERE email = $1", [email]); // param
    const ok = user && (await bcrypt.compare(password, user.passwordHash)); // hash compare
    if (!ok) {
      return res.status(401).json({ error: "Invalid email or password" }); // generic
    }
    const token = jwt.sign(
      { sub: user.id, role: user.role },          // parol YO'Q
      process.env.JWT_SECRET!,                      // env'dan
      { algorithm: "HS256", expiresIn: "15m" }
    );
    res.json({ token });
  } catch (err) { next(err); }
});
```

---

## 3. Parol hashlash

```ts
import bcrypt from "bcrypt";

const COST = 12; // 2^12 round

export async function register(email: string, password: string) {
  const passwordHash = await bcrypt.hash(password, COST);
  return db.createUser({ email, passwordHash });
}

export async function login(email: string, password: string) {
  const user = await db.findUserByEmail(email);
  if (!user) {
    await bcrypt.hash(password, COST); // dummy — timing attack'ni kamaytirish
    return null;
  }
  const ok = await bcrypt.compare(password, user.passwordHash);
  return ok ? user : null;
}
```

**Cost factor tanlash:** Hashlash ~100-250ms olishi uchun sozla (production hardware'da o'lchab). Hozir 12 yaxshi boshlang'ich. Hardware tezlashgani sayin oshirib bor.

**Nega SHA-256 yaramaydi:** SHA-256 tezligi uchun yaratilgan — sekundiga milliardlab hash. Parol hashlash *qasddan sekin* va memory-hard bo'lishi kerak, shunda brute-force qimmatga tushadi. bcrypt sekinlikni cost factor bilan sozlaydi, salt'ni o'zi qo'shadi.

---

## 4. CSRF himoyasi

**SameSite atributlari:**
- `Strict` — cookie faqat bir xil saytdan yuboriladi. Eng kuchli himoya, lekin tashqi linkdan kelganda foydalanuvchi loginsiz ko'rinadi.
- `Lax` (default) — yuqori darajadagi navigatsiya (`GET` link) da yuboriladi, lekin cross-site `POST`da yo'q. Aksariyat holat uchun yaxshi muvozanat.
- `None` — har doim yuboriladi (cross-site SPA uchun), `Secure` shart.

**CSRF token (synchronizer token pattern):**
1. Server forma/sessiya bilan tasodifiy token beradi.
2. Client har state-changing so'rovda uni header/body'da qaytaradi.
3. Server cookie va token mosligini tekshiradi.

Hujumchi sayti cookie'ni avtomatik yuborsa ham, CSRF token'ni bilmaydi (boshqa origin uni o'qiy olmaydi).

**Tanlov:** Bir xil saytli ilova → `SameSite=Lax`/`Strict` ko'p hollarda yetarli. Cross-site SPA yoki qo'shimcha qatlam kerak bo'lsa → `SameSite=None; Secure` + CSRF token (double-submit yoki synchronizer).

---

## 5. OAuth2 flow chizmasi (Authorization Code + PKCE)

```
1. App:    code_verifier = random()
           code_challenge = SHA256(code_verifier)

2. Browser → Google /authorize:
   ?client_id=...&redirect_uri=...&response_type=code
   &scope=openid email&state=<csrf>&code_challenge=<hash>&code_challenge_method=S256

3. User Google'da login + ruxsat (consent)

4. Google → Browser → App /callback:
   ?code=<auth_code>&state=<csrf>

5. App: state'ni tekshiradi (CSRF himoyasi)

6. App → Google /token:
   code=<auth_code>&client_id=...&code_verifier=<asl verifier>&grant_type=authorization_code

7. Google: SHA256(code_verifier) == saqlangan code_challenge ? OK
   → access_token + id_token (JWT, OIDC)

8. App id_token'ni tekshirib foydalanuvchini aniqlaydi
```

**`state` nima uchun:** CSRF himoyasi. App tasodifiy `state` yuboradi, callback'da uni qaytishini tekshiradi. Bu hujumchining soxta authorization code'ni foydalanuvchi sessiyasiga "ulashtirishi" (login CSRF)ning oldini oladi.

**PKCE:** `client_secret`siz public client'larni himoya qiladi — o'g'irlangan code `code_verifier`siz yaroqsiz.

---

## 6. JWT tekshirish middleware

```ts
import jwt from "jsonwebtoken";
import { Request, Response, NextFunction } from "express";

export function authenticate(req: Request, res: Response, next: NextFunction) {
  const header = req.header("Authorization");
  if (!header?.startsWith("Bearer ")) {
    return res.status(401).json({ error: "Missing token" });
  }
  const token = header.slice(7);
  try {
    const payload = jwt.verify(token, process.env.JWT_SECRET!, {
      algorithms: ["HS256"], // FAQAT kutilgan algoritm — alg:none/RS256 chalkashligini to'sadi
    });
    (req as any).user = payload;
    next();
  } catch (err) {
    return res.status(401).json({ error: "Invalid or expired token" });
  }
}
```

**`alg: none` hujumini to'sish:** `algorithms: ["HS256"]` ni aniq belgilash shart. Bu bo'lmasa, kutubxona token'dagi `alg` ni ishonadi — hujumchi `alg: none` (imzosiz) yoki RS256→HS256 chalkashlik bilan soxta token yasashi mumkin. Algoritmni hech qachon token'dan o'qima.

---

## 7. RBAC tizimi

```ts
const PERMISSIONS: Record<string, string[]> = {
  admin:  ["read", "write", "delete", "manage_users"],
  editor: ["read", "write"],
  viewer: ["read"],
};

function requireRole(...roles: string[]) {
  return (req, res, next) =>
    roles.includes(req.user?.role)
      ? next()
      : res.status(403).json({ error: "Forbidden" });
}

function requirePermission(perm: string) {
  return (req, res, next) =>
    PERMISSIONS[req.user?.role]?.includes(perm)
      ? next()
      : res.status(403).json({ error: "Forbidden" });
}

// IDOR'dan himoya — resurs egaligini tekshirish
async function requireOwnership(req, res, next) {
  const doc = await db.findDoc(req.params.id);
  if (!doc) return res.status(404).json({ error: "Not found" });
  if (doc.ownerId !== req.user.sub && req.user.role !== "admin") {
    return res.status(403).json({ error: "Forbidden" });
  }
  req.doc = doc;
  next();
}

app.delete("/docs/:id", authenticate, requirePermission("delete"), requireOwnership, deleteHandler);
```

**Izoh:** Rol tekshiruvi yetarli emas — `editor` boshqa editor'ning hujjatini o'chira olmasligi uchun egalik (`ownerId`) ham tekshiriladi. Bu IDOR'ni to'sadi.

---

## 8. Cookie sozlamalari

```ts
res.cookie("session", sessionId, {
  httpOnly: true,                       // JS o'qiy olmaydi (XSS himoyasi)
  secure: true,                         // faqat HTTPS orqali
  sameSite: "strict",                   // cross-site so'rovda yuborilmaydi (CSRF himoyasi)
  maxAge: 1000 * 60 * 60,               // 1 soat (millisekund)
  path: "/",                            // butun sayt uchun
  // domain: ".example.com",            // subdomenlar kerak bo'lsa
});
```

- **httpOnly: true** — `document.cookie` orqali o'qib bo'lmaydi, XSS token o'g'irligini to'sadi.
- **secure: true** — ochiq HTTP'da yuborilmaydi, tarmoqda eshitishdan himoya.
- **sameSite: "strict"** — boshqa saytdan kelgan so'rovda cookie ketmaydi, CSRF'ni to'sadi. Cross-site SPA kerak bo'lsa `"none"` + `secure`.
- **maxAge** — muddat; bo'lmasa "session cookie" (brauzer yopilganda o'chadi).
- **path** — cookie qaysi yo'llarga yuborilishi.

---

## 9. Rate limiter strategiyasi

```ts
import rateLimit from "express-rate-limit";
import RedisStore from "rate-limit-redis";

// 1-qatlam: IP bo'yicha umumiy
const ipLimiter = rateLimit({
  store: new RedisStore({ /* redis client */ }),
  windowMs: 15 * 60 * 1000,
  max: 50,
  keyGenerator: (req) => req.ip,
});

// 2-qatlam: email bo'yicha qattiq
const emailLimiter = rateLimit({
  store: new RedisStore({ /* redis client */ }),
  windowMs: 15 * 60 * 1000,
  max: 5,
  skipSuccessfulRequests: true,
  keyGenerator: (req) => `login:${req.body.email}`,
});

app.post("/login", ipLimiter, emailLimiter, loginHandler);
```

**DoS xavfini yumshatish:** To'liq "account lockout" (bloklash) o'rniga **exponential backoff** yoki **CAPTCHA** ishlat — hujumchi qasddan birovning hisobini bloklay olmaydi. Email bo'yicha hisoblagich vaqt oynasidan keyin tabiiy tozalanadi.

**Distributed muhitda:** In-memory limiter har instance'da alohida ishlaydi → 3 server = 3× limit. **Redis store** markazlashgan, atomar (`INCR`/`EXPIRE`) hisoblagich beradi, barcha instance bir manbadan o'qiydi.

---

## 10. Input validatsiya va sanitizatsiya

```ts
import { z } from "zod";

const CommentSchema = z.object({
  text: z.string().min(1).max(2000),
  postId: z.number().int().positive(),
}).strict();

app.post("/comments", authenticate, async (req, res, next) => {
  try {
    const { text, postId } = CommentSchema.parse(req.body); // 1. validation
    const comment = await db.createComment({
      text,                          // RAW saqla (escape qilmasdan)
      postId,
      authorId: req.user.sub,
    });
    res.status(201).json(comment);
  } catch (err) { next(err); }
});
```

**Server bosqichi:** Validatsiya (uzunlik, tip, strict). Matnni *raw* saqla — bu yerda HTML escape qilma.

**Render bosqichi (XSS himoyasi):** Eng muhim qoida — **kontekstda escape qil**:
- React/Vue ishlatsang — `{comment.text}` avtomatik escape qiladi (`dangerouslySetInnerHTML`/`v-html` ISHLATMA).
- Server-side template (EJS/Handlebars) — `<%= %>`/`{{ }}` (escape) ishlat, raw `<%- %>`/`{{{ }}}` emas.
- Agar HTML formatlash kerak bo'lsa (rich text), **DOMPurify** bilan allowlist asosida tozalа.
- Qo'shimcha qatlam: `Content-Security-Policy` header (inline script'ni bloklaydi).

**Nega render'da escape, saqlashda emas:** Bir xil ma'lumot turli kontekstda (HTML, JSON, attribut, URL) turlicha escape qilinadi. Output'da escape qilish — to'g'ri kontekstni tanlash imkonini beradi va ma'lumotni buzmaydi.

---

← [Backend bo'limiga qaytish](../../backend/README.md)
