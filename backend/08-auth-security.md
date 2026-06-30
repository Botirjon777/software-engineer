# Auth va Xavfsizlik

Autentifikatsiya, avtorizatsiya va backend dasturchi bilishi shart bo'lgan himoya muhandisligi. Bu mavzu senior intervyularda chuqur so'raladi — har bir tushunchani "nega" bilan tushunish muhim.

## Mundarija

- [Authentication vs Authorization](#authentication-vs-authorization)
- [Session vs Token](#session-vs-token)
- [Cookie (httpOnly, secure, sameSite)](#cookie-httponly-secure-samesite)
- [JWT](#jwt)
- [Refresh token va revocation](#refresh-token-va-revocation)
- [OAuth2 va OpenID Connect](#oauth2-va-openid-connect)
- [Password hashing](#password-hashing)
- [OWASP Top 10](#owasp-top-10)
- [HTTPS / TLS](#https--tls)
- [CORS xavfsizlik nuqtai nazaridan](#cors-xavfsizlik-nuqtai-nazaridan)
- [Secrets management](#secrets-management)
- [Rate limiting va brute-force himoyasi](#rate-limiting-va-brute-force-himoyasi)
- [Input validation va sanitizatsiya](#input-validation-va-sanitizatsiya)
- [Least privilege, RBAC vs ABAC](#least-privilege-rbac-vs-abac)
- [Intervyu savollari (Q&A)](#intervyu-savollari-qa)
- [Masalalar](#masalalar)

---

## Authentication vs Authorization

**💡 Tushuncha:**

- **Authentication (authn)** — "sen kimsan?" savoliga javob. Shaxsni tasdiqlash (parol, token, biometriya).
- **Authorization (authz)** — "senga nimaga ruxsat bor?" savoliga javob. Tasdiqlangan shaxsning huquqlarini tekshirish.

Authn har doim authz'dan oldin keladi.

**⚠️ Ehtiyot bo'l:** Yaroqli token *kim ekaningni* isbotlaydi, lekin *bu amalga ruxsating borligini* emas. Ko'p buzilishlar (breach) aynan buzilgan **avtorizatsiya** sababli — masalan **IDOR** (Insecure Direct Object Reference): `/orders/123` boshqa foydalanuvchiniki bo'lsa ham, faqat ID'ni o'zgartirib kirish. Har bir so'rovda **server tomonida** egalik/huquqni tekshir.

---

## Session vs Token

**💡 Tushuncha:** Foydalanuvchini "eslab qolish"ning ikki asosiy usuli:

**Session-based (stateful):**
- Server session yaratadi, uni DB/Redis'da saqlaydi, client'ga `session_id` cookie beradi.
- Har so'rovda server `session_id` bo'yicha holatni qidiradi.
- Logout/bekor qilish oson (session'ni o'chirib tashlaysan).
- Lekin server holat saqlaydi — scaling uchun umumiy session store kerak.

**Token-based (stateless, JWT):**
- Server imzolangan token beradi; barcha ma'lumot token ichida.
- Server hech narsa saqlamaydi — faqat imzoni tekshiradi.
- Scaling oson, lekin bekor qilish (revocation) qiyin (token muddati tugaguncha yaroqli).

| Mezon | Session | Token (JWT) |
|-------|---------|-------------|
| Holat | Serverda | Tokenda |
| Scaling | Umumiy store kerak | Oson |
| Revocation | Oson | Qiyin |
| O'lcham | Kichik cookie | Kattaroq token |

**⚠️ Ehtiyot bo'l:** "JWT har doim yaxshiroq" — noto'g'ri. JWT'ning revocation muammosi jiddiy: o'g'irlangan token muddati tugaguncha ishlaydi. Ko'p tizimlar uchun Redis-asosli session aslida soddaroq va xavfsizroq.

---

## Cookie (httpOnly, secure, sameSite)

**💡 Tushuncha:** Cookie — brauzer saqlaydigan va har so'rovda avtomatik yuboradigan kichik ma'lumot. Auth uchun muhim atributlar:

```http
Set-Cookie: session=abc123; HttpOnly; Secure; SameSite=Strict; Max-Age=3600; Path=/
```

- **HttpOnly** — JavaScript cookie'ga kira olmaydi (`document.cookie`). XSS orqali token o'g'irlashni qiyinlashtiradi.
- **Secure** — faqat HTTPS orqali yuboriladi (tarmoqda ochiq ketmaydi).
- **SameSite** — cross-site so'rovlarda cookie yuborilishini boshqaradi:
  - `Strict` — faqat bir xil saytdan (CSRF'ga kuchli himoya, lekin tashqi linkdan kelganda login tushadi).
  - `Lax` — yuqori darajadagi navigatsiyada yuboriladi (default, muvozanat).
  - `None` — har doim (cross-site uchun, lekin `Secure` shart).

**⚠️ Ehtiyot bo'l:** Auth token'ni `localStorage`'da saqlash XSS'ga ochiq (har qanday JS uni o'qiy oladi). `HttpOnly` cookie xavfsizroq, lekin CSRF xavfini keltiradi — buni `SameSite` + CSRF token bilan yopasan. "Cookie XSS'dan, token CSRF'dan himoyalangan" — noto'g'ri, ikkalasiga ham e'tibor kerak.

---

## JWT

**💡 Tushuncha:** JWT (JSON Web Token) — 3 qismdan iborat, nuqta bilan ajratilgan imzolangan token:

```
eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiIxMjMiLCJyb2xlIjoiYWRtaW4ifQ.SflKxw...
└── Header ──┘ └──────── Payload ────────┘ └── Signature ──┘
```

```jsonc
// Header — algoritm va tip
{ "alg": "HS256", "typ": "JWT" }

// Payload — "claims" (da'volar)
{ "sub": "123", "role": "admin", "iat": 1719700000, "exp": 1719703600 }

// Signature = HMACSHA256(base64(header) + "." + base64(payload), secret)
```

- **Header** va **Payload** — shunchaki base64url (shifrlanmagan! hamma o'qiy oladi).
- **Signature** — maxfiy kalit (HS256) yoki private key (RS256) bilan imzolangan. Server imzoni tekshiradi.

**Stateless afzalliklari:** server holat saqlamaydi, scaling oson, mikroservislar token'ni mustaqil tekshira oladi.

**Kamchiliklari:** revocation qiyin, token kattaroq, payload o'zgartirib bo'lmaydi (yangi token kerak).

**Qayerda saqlash:** `HttpOnly` + `Secure` cookie (XSS'dan himoya) eng yaxshi; `localStorage` qulay lekin XSS'ga ochiq.

**⚠️ Ehtiyot bo'l:**
- Payload **shifrlanmagan** — hech qachon parol, maxfiy ma'lumot solma. Faqat imzo uni *o'zgartirishdan* himoya qiladi, *o'qishdan* emas.
- **`alg: none` hujumi** — eski kutubxonalar imzosiz token'ni qabul qilgan. Har doim kutilgan algoritmni majburla, server tomonida tekshir.
- **RS256 vs HS256 chalkashligi** — agar server algoritmni token'dan o'qisa, hujumchi public key'ni HMAC secret sifatida ishlatib soxta token yasashi mumkin.

---

## Refresh token va revocation

**💡 Tushuncha:** Access token qisqa muddatli (5-15 daqiqa) bo'lishi kerak (o'g'irlansa zarar kam). Lekin foydalanuvchini har 15 daqiqada login qildirib bo'lmaydi — shu uchun **refresh token**:

- **Access token** — qisqa muddat, har so'rovda ishlatiladi, JWT.
- **Refresh token** — uzoq muddat (kun/hafta), faqat yangi access token olish uchun, DB'da saqlanadi (revocation mumkin).

```
1. Login → access (15min) + refresh (7kun)
2. Access muddati o'tdi → refresh bilan yangi access ol
3. Logout → refresh'ni DB'dan o'chir (bekor qilingan)
```

**Revocation muammosi:** Stateless JWT'ni darhol bekor qilib bo'lmaydi. Yechimlar:
- Qisqa muddat (`exp`) — zarar oynasini kichraytiradi.
- **Token denylist** (Redis) — bekor qilingan token jti'larini saqlash (lekin bu stateless'likni qisman buzadi).
- **Refresh token rotation** — har refresh'da yangi refresh chiqarish, eskisini bekor qilish (token o'g'irligini aniqlash).

**⚠️ Ehtiyot bo'l:** Refresh token o'g'irlansa, hujumchi uzoq vaqt access olishi mumkin. **Rotation** + reuse detection ishlat: agar eski (allaqachon ishlatilgan) refresh qayta kelsa, butun zanjirni bekor qil (o'g'irlik signali).

---

## OAuth2 va OpenID Connect

**💡 Tushuncha:**

- **OAuth2** — **avtorizatsiya** framework'i. "Bu ilovaga mening Google diskimga kirishga ruxsat" — parolni bermasdan, cheklangan ruxsat (scope) berish.
- **OpenID Connect (OIDC)** — OAuth2 ustiga qurilgan **autentifikatsiya** qatlami. "Google bilan kirish" — kim ekaningni isbotlash (ID token beradi).

**Authorization Code flow (eng xavfsiz, server-side):**

```
1. User → App: "Google bilan kir"
2. App → Google: redirect (client_id, redirect_uri, scope)
3. User Google'da login + ruxsat beradi
4. Google → App: authorization code (redirect orqali)
5. App → Google: code + client_secret → access_token + id_token
6. App access_token bilan API'ga kiradi
```

**PKCE (Proof Key for Code Exchange):** SPA/mobil ilovalar `client_secret` saqlay olmaydi (kod ochiq). PKCE buni hal qiladi:
- App tasodifiy `code_verifier` yaratadi, uning hash'ini (`code_challenge`) yuboradi.
- Code'ni almashtirayotganda asl `code_verifier`ni ko'rsatadi.
- Hujumchi code'ni o'g'irlasa ham, `code_verifier`siz ishlata olmaydi.

**⚠️ Ehtiyot bo'l:** **Implicit flow** (token'ni to'g'ridan-to'g'ri URL'da qaytarish) endi eskirgan va xavfli — har doim **Authorization Code + PKCE** ishlat, hatto SPA uchun ham. `state` parametrini CSRF himoyasi uchun tekshir.

---

## Password hashing

**💡 Tushuncha:** Parolni hech qachon ochiq (plaintext) yoki oddiy hash bilan saqlama. **Sekin**, **salt** bilan ishlovchi algoritm kerak:

- **bcrypt** — sinovdan o'tgan, salt o'rnatilgan, "cost factor" (sekinlikni sozlash).
- **argon2** (argon2id) — zamonaviy tavsiya, GPU/ASIC hujumlariga chidamli, memory-hard.
- **scrypt** — argon2 muqobili, memory-hard.

```ts
import bcrypt from "bcrypt";

// Ro'yxatdan o'tish
const hash = await bcrypt.hash(plainPassword, 12); // 12 = cost factor

// Login
const ok = await bcrypt.compare(plainPassword, hash);
```

**Salt** — har parolga unikal tasodifiy qiymat. Bir xil parollar har xil hash beradi → **rainbow table** hujumi va bir xil parollarni aniqlashning oldini oladi. bcrypt/argon2 salt'ni o'zi yaratib, hash ichiga qo'shadi.

**⚠️ Ehtiyot bo'l:** **MD5/SHA-256 parol uchun YAROMAYDI** — ular *tez* (sekundiga milliardlab hash), bu brute-force'ni osonlashtiradi. Parol hashlash *qasddan sekin* bo'lishi kerak. Hech qachon o'zing "salt + SHA256" yozma — bcrypt/argon2 ishlat. Cost factor'ni vaqt o'tishi bilan oshirib bor (hardware tezlashgani uchun).

---

## OWASP Top 10

**💡 Tushuncha:** Eng keng tarqalgan web zaifliklar va ularning oldini olish:

**1. SQL Injection** — foydalanuvchi kiritmasi SQL'ga to'g'ridan qo'shilsa:
```ts
// ❌ XAVFLI
db.query(`SELECT * FROM users WHERE email = '${email}'`);
// email = "' OR '1'='1" → hammasini qaytaradi
// ✅ XAVFSIZ — parametrlangan so'rov
db.query("SELECT * FROM users WHERE email = $1", [email]);
```

**2. XSS (Cross-Site Scripting)** — foydalanuvchi kiritmasi HTML'ga injekt qilinsa:
```js
// ❌ user.bio = "<script>steal(document.cookie)</script>"
el.innerHTML = user.bio;
// ✅ escape qil / textContent ishlat / CSP qo'y
el.textContent = user.bio;
```

**3. CSRF (Cross-Site Request Forgery)** — boshqa sayt foydalanuvchi nomidan so'rov yuboradi (cookie avtomatik ketadi). Himoya: `SameSite` cookie + CSRF token + state-changing amallarda `GET` ishlatmaslik.

**4. Broken Authentication** — zaif parol siyosati, sessiya boshqaruvi xatolari, brute-force. Himoya: MFA, rate limiting, kuchli hashing, xavfsiz session.

**5. Broken Access Control / IDOR** — avtorizatsiya tekshirmaslik. Himoya: har resursda egalik tekshiruvi.

Qolganlar: **Security Misconfiguration**, **Sensitive Data Exposure**, **Insecure Deserialization**, **Vulnerable Components** (eski paketlar), **Insufficient Logging**.

**⚠️ Ehtiyot bo'l:** ORM (Prisma, TypeORM) SQL injection'dan ko'p himoya qiladi, lekin raw query yozsang yana xavf qaytadi. XSS'da `innerHTML` o'rniga `textContent`, React/Vue avtomatik escape qiladi — lekin `dangerouslySetInnerHTML`/`v-html` bilan ehtiyot bo'l.

---

## HTTPS / TLS

**💡 Tushuncha:** TLS (Transport Layer Security, eski nomi SSL) — client va server orasidagi trafikni shifrlaydi. HTTPS = HTTP over TLS. U uchta narsani beradi:

1. **Confidentiality** — trafik shifrlangan (eshitib bo'lmaydi).
2. **Integrity** — yo'lda o'zgartirib bo'lmaydi.
3. **Authentication** — sertifikat serverning haqiqiyligini isbotlaydi (CA imzosi).

TLS handshake'da asimmetrik shifrlash (kalit almashish) bilan boshlanib, keyin tezroq simmetrik shifrlashga o'tiladi. TLS 1.3 — eng zamonaviy, tezroq (1-RTT/0-RTT), zaif shifrlarni olib tashlagan.

**⚠️ Ehtiyot bo'l:** Faqat HTTPS yetarli emas — **HSTS** (`Strict-Transport-Security`) header qo'y, shunda brauzer kelajakda faqat HTTPS ishlatadi (downgrade hujumidan himoya). HTTP'dan HTTPS'ga redirect qil. Self-signed sertifikat production'da yaramaydi.

---

## CORS xavfsizlik nuqtai nazaridan

**💡 Tushuncha:** CORS (07-mavzuda batafsil) — brauzer himoyasi, foydalanuvchi brauzerini boshqa origin'lar nomidan so'rov yuborishdan to'sadi. Xavfsizlik nuqtai nazaridan:

- `Access-Control-Allow-Origin` ni **dinamik aks ettirma** (reflektsiya): `origin` header'ni ko'r-ko'rona qaytarish har saytga ruxsat berishga teng.
- `credentials: true` bilan `*` ishlatib bo'lmaydi — aniq allowlist ishlat.
- Ichki/admin API'larni CORS bilan butunlay yopiq tut.

**⚠️ Ehtiyot bo'l:** CORS — server himoyasi EMAS. U faqat brauzer JS'iga tegishli. Server xavfsizligi auth, validation, rate limiting bilan ta'minlanadi. CORS'ni keng ochib qo'yish (`*`) o'z-o'zidan teshik emas (agar credentials bo'lmasa), lekin credentials bilan birga jiddiy zaiflik.

---

## Secrets management

**💡 Tushuncha:** Maxfiy ma'lumotlar (DB parol, API kalit, JWT secret) hech qachon kodda yoki git'da bo'lmasligi kerak.

- **Environment variables** — minimal yondashuv (`process.env.JWT_SECRET`), `.env` faylni `.gitignore`'ga qo'sh.
- **Secret manager** — AWS Secrets Manager, HashiCorp Vault, GCP Secret Manager: shifrlangan saqlash, audit, rotation.
- **Rotation** — kalitlarni davriy almashtirish.

```ts
// ✅ env'dan o'qish
const secret = process.env.JWT_SECRET;
if (!secret) throw new Error("JWT_SECRET not set");
```

**⚠️ Ehtiyot bo'l:** `.env` faylni git'ga tasodifan commit qilish — eng keng tarqalgan xato. Agar bo'lib qolsa, kalitni darhol **almashtir** (git tarixidan o'chirish yetarli emas — u allaqachon ko'rilgan deb hisobla). `git-secrets`/pre-commit hook bilan oldini ol. Secret'larni logga yozma.

---

## Rate limiting va brute-force himoyasi

**💡 Tushuncha:** Login/parol tiklash kabi sezgir endpoint'lar brute-force (parol topish) hujumiga nishon. Himoya qatlamlari:

- **Rate limiting** — IP va account bo'yicha urinishlarni cheklash (`429`).
- **Account lockout / exponential backoff** — ketma-ket xatodan keyin kechikish yoki vaqtinchalik bloklash.
- **CAPTCHA** — bir necha xatodan keyin.
- **MFA** — ikkinchi faktor brute-force'ni amalda imkonsiz qiladi.
- **Generic error** — "email yoki parol noto'g'ri" (qaysi biri xato ekanini oshkor qilma).

```ts
import rateLimit from "express-rate-limit";

const loginLimiter = rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 5,                       // 15 daqiqada 5 urinish
  skipSuccessfulRequests: true, // muvaffaqiyatlilarni hisoblama
});
app.post("/login", loginLimiter, loginHandler);
```

**⚠️ Ehtiyot bo'l:** Faqat IP bo'yicha limit yetarli emas — botnet har xil IP ishlatadi. Account (email) bo'yicha ham cheklov qo'y. Lekin email bo'yicha lockout'ni e'tiborli ishlat — hujumchi qasddan boshqalarning hisobini bloklashi mumkin (DoS). Distributed muhitda Redis-asosli limiter kerak.

---

## Input validation va sanitizatsiya

**💡 Tushuncha:** Tashqi ma'lumotga ishonma. Ikki bosqich:

- **Validation** — kutilgan formatga mosligini tekshirish (email, son chegarasi, enum). Mos kelmasa rad et.
- **Sanitization** — xavfli belgilarni tozalash/escape qilish (HTML, SQL maxsus belgilar).

**Allowlist > denylist** — ruxsat etilganni belgilash, taqiqlanganni emas (denylist har doim to'liq emas).

```ts
import { z } from "zod";
const schema = z.object({
  email: z.string().email(),
  age: z.number().int().min(0).max(120),
}).strict();
```

**⚠️ Ehtiyot bo'l:** "Context-aware" tozalash kerak — SQL uchun parametrlangan so'rov, HTML uchun escape, shell uchun argument escape, har biri boshqacha. Bitta universal "sanitize" funksiyasi yetarli emas. Validatsiyani server tomonida majburla; frontend faqat UX uchun.

---

## Least privilege, RBAC vs ABAC

**💡 Tushuncha:** **Least privilege (eng kam imtiyoz)** — har komponent/foydalanuvchi/servis faqat o'z ishi uchun kerakli minimal ruxsatga ega bo'lishi kerak. DB user faqat kerakli jadvallarga, servis faqat kerakli resurslarga.

**Avtorizatsiya modellari:**

- **RBAC (Role-Based)** — ruxsatlar **rol**larga biriktiriladi (`admin`, `editor`, `viewer`), foydalanuvchiga rol beriladi. Oddiy, tushunarli, ko'p holatda yetarli.
- **ABAC (Attribute-Based)** — qaror **atributlar** asosida (foydalanuvchi, resurs, kontekst): "faqat o'z bo'limidagi hujjatni, ish vaqtida, o'z mintaqasidan tahrirlay oladi". Moslashuvchan, lekin murakkab.

```ts
// RBAC misol
function requireRole(role: string) {
  return (req, res, next) =>
    req.user?.roles.includes(role) ? next() : res.status(403).json({ error: "Forbidden" });
}
app.delete("/users/:id", requireRole("admin"), deleteHandler);
```

**⚠️ Ehtiyot bo'l:** RBAC katta tizimda "rol portlashi"ga olib keladi (har kichik farq uchun yangi rol). Murakkab, kontekstga bog'liq qoidalar kerak bo'lsa ABAC'ga o't. Lekin avtorizatsiyani har doim **server tomonida** tekshir — frontend'da tugmani yashirish himoya emas.

---

## Intervyu savollari (Q&A)

### ❓ Authentication va authorization farqi?

**✅ Javob:** Authentication — "kimsan?" (shaxsni tasdiqlash: parol, token). Authorization — "nimaga ruxsating bor?" (huquqlarni tekshirish). Authn oldin, authz keyin. Yaroqli token kimligingni isbotlaydi, lekin har bir amalga ruxsat borligini server alohida tekshirishi kerak.

### ❓ Session va JWT — qaysi birini tanlaysan?

**✅ Javob:** Session (stateful) — revocation oson, lekin server store kerak. JWT (stateless) — scaling oson, mikroservislarda qulay, lekin darhol bekor qilib bo'lmaydi. Logout/ban tez-tez kerak bo'lsa session yaxshiroq; tarqoq, ko'p-servisli tizimda qisqa-muddatli JWT + refresh ishlat.

### ❓ JWT payload'ga parol solsa bo'ladimi?

**✅ Javob:** Yo'q! Payload faqat base64 — shifrlanmagan, hamma o'qiy oladi. Imzo uni *o'zgartirishdan* himoya qiladi, *o'qishdan* emas. Maxfiy hech narsa solma, faqat identifikator va ruxsat ma'lumotlari.

### ❓ JWT'ni qayerda saqlash kerak?

**✅ Javob:** `HttpOnly` + `Secure` + `SameSite` cookie eng xavfsiz (JS o'qiy olmaydi → XSS'dan himoya). `localStorage` qulay lekin XSS'ga ochiq. Cookie ishlatsang CSRF himoyasini (`SameSite`/CSRF token) qo'sh.

### ❓ Nega parol uchun MD5/SHA-256 yaramaydi?

**✅ Javob:** Ular *juda tez* — hujumchi sekundiga milliardlab hash sinab brute-force qiladi. Parol hashlash *qasddan sekin* va memory-hard bo'lishi kerak: bcrypt/argon2. Ular salt'ni o'zi qo'shib, cost factor bilan sekinlikni sozlaydi.

### ❓ Salt nima va nega kerak?

**✅ Javob:** Salt — har parolga qo'shiladigan unikal tasodifiy qiymat. Bir xil parollar har xil hash beradi, bu rainbow table hujumini va bir xil parollarni aniqlashni imkonsiz qiladi. bcrypt/argon2 salt'ni avtomatik yaratib hash ichiga joylaydi.

### ❓ Refresh token nima uchun kerak?

**✅ Javob:** Access token qisqa muddatli bo'lishi kerak (o'g'irlansa zarar kam), lekin har 15 daqiqada login qildirib bo'lmaydi. Refresh token (uzoq muddat, DB'da) yangi access token olish uchun ishlatiladi. Logout'da refresh'ni bekor qilasan. Rotation + reuse detection o'g'irlikni aniqlaydi.

### ❓ JWT revocation muammosi nima va qanday hal qilasan?

**✅ Javob:** Stateless JWT muddati tugaguncha yaroqli — o'g'irlangan token'ni darhol bekor qilib bo'lmaydi. Yechimlar: qisqa `exp` (zarar oynasini kichraytirish), Redis denylist (jti bo'yicha), refresh token rotation. To'liq darhol-revocation kerak bo'lsa, session-based yondashuv soddaroq.

### ❓ OAuth2 va OpenID Connect farqi?

**✅ Javob:** OAuth2 — **avtorizatsiya** (resursga cheklangan ruxsat berish, parolni bermasdan). OIDC — OAuth2 ustidagi **autentifikatsiya** qatlami (kim ekaningni isbotlash, ID token beradi). "Google bilan kirish" — OIDC. Google diskingга ruxsat berish — OAuth2.

### ❓ PKCE nima uchun kerak?

**✅ Javob:** SPA/mobil ilovalar `client_secret`ni xavfsiz saqlay olmaydi (kod ochiq). PKCE: app `code_verifier` yaratib uning hash'ini yuboradi, code almashtirishda aslini ko'rsatadi. Hujumchi authorization code'ni o'g'irlasa ham, `code_verifier`siz token ololmaydi.

### ❓ SQL injection'dan qanday himoyalanasan?

**✅ Javob:** Parametrlangan so'rov (prepared statement) ishlat — kiritmani hech qachon SQL stringiga to'g'ridan qo'shma. ORM ko'p himoya qiladi (lekin raw query'da ehtiyot bo'l). Least privilege DB user, input validation qo'shimcha qatlam.

### ❓ XSS va CSRF farqi?

**✅ Javob:** **XSS** — hujumchi sahifaga zararli JS injekt qiladi (kiritma escape qilinmaganda); cookie/data o'g'irlaydi. Himoya: escape/`textContent`, CSP, `HttpOnly` cookie. **CSRF** — boshqa sayt foydalanuvchi nomidan so'rov yuboradi (cookie avtomatik ketadi). Himoya: `SameSite` cookie, CSRF token.

### ❓ HttpOnly cookie nimadan himoya qiladi, nimadan yo'q?

**✅ Javob:** XSS orqali token o'qishdan himoya qiladi (JS `document.cookie`'ga kira olmaydi). Lekin CSRF'dan himoya qilmaydi — cookie baribir avtomatik yuboriladi. CSRF uchun `SameSite` + CSRF token kerak.

### ❓ RBAC va ABAC farqi, qachon qaysi biri?

**✅ Javob:** RBAC — ruxsat rollarga biriktiriladi (`admin`, `editor`), oddiy va ko'p holatda yetarli. ABAC — qaror atributlar asosida (foydalanuvchi/resurs/kontekst), moslashuvchan lekin murakkab. Kontekstga bog'liq nozik qoidalar (bo'lim, vaqt, mintaqa) kerak bo'lsa ABAC; aks holda RBAC.

### ❓ Brute-force'dan login'ni qanday himoya qilasan?

**✅ Javob:** IP + account bo'yicha rate limiting, exponential backoff/lockout, CAPTCHA (bir necha xatodan keyin), MFA, generic xato xabari ("email yoki parol noto'g'ri"). Distributed'da Redis limiter. Account lockout'ni ehtiyot ishlat (DoS xavfi).

### ❓ Secret'ni kodga commit qilib qo'ysang nima qilasan?

**✅ Javob:** Kalitni darhol **almashtir** (rotate) — git tarixidan o'chirish yetarli emas, u allaqachon oshkor bo'ldi deb hisobla. Kelajakda `.gitignore` + pre-commit hook (`git-secrets`) bilan oldini ol, secret manager ishlat.

---

## Masalalar

> Yechimlar: [solutions/backend/08-auth-security.md](../solutions/backend/08-auth-security.md)

1. **Auth oqimini loyihalash.** Web ilova uchun JWT-asosli auth tizimini loyihala: access + refresh token muddatlari, ularni qayerda saqlash, logout qanday ishlashi, refresh rotation. Har tanlovingni xavfsizlik nuqtai nazaridan asosla.

2. **Zaiflikni top.** Quyidagi kodda kamida 3 ta xavfsizlik muammosini top va tuzat:
   ```ts
   app.post("/login", async (req, res) => {
     const { email, password } = req.body;
     const user = await db.query(`SELECT * FROM users WHERE email = '${email}'`);
     if (user.password === password) {
       const token = jwt.sign({ email, password, isAdmin: user.isAdmin }, "secret123");
       res.json({ token });
     } else {
       res.status(401).json({ error: `No user with password ${password}` });
     }
   });
   ```

3. **Parol hashlash.** bcrypt bilan ro'yxatdan o'tish va login funksiyalarini yoz. Cost factor'ni qanday tanlaysan? Nega plain SHA-256 yaramaydi — tushuntir.

4. **CSRF himoyasi.** Cookie-asosli session ishlatuvchi ilovada CSRF'ni qanday oldini olasan? `SameSite` atributining `Strict`/`Lax`/`None` qiymatlari va CSRF token yondashuvini tasvirla. Qaysi holatda qaysi birini tanlaysan?

5. **OAuth2 flow chizmasi.** "Google bilan kirish" uchun Authorization Code + PKCE oqimini bosqichma-bosqich yoz (client, brauzer, Google, app o'rtasidagi so'rovlar). `state` parametri nima uchun kerak?

6. **JWT tekshirish middleware.** Express middleware yoz: `Authorization: Bearer <token>` header'ni o'qiydi, JWT'ni tekshiradi (algoritmni majburlab), `req.user`'ni o'rnatadi, xatolarni to'g'ri status bilan qaytaradi (`401`). `alg: none` hujumini qanday to'sasan?

7. **RBAC tizimi.** Uch rolli (`admin`, `editor`, `viewer`) RBAC tizimini loyihala. `requireRole` va `requirePermission` middleware'larini yoz. IDOR'dan himoya uchun resurs egaligini ham tekshiradigan misol qo'sh.

8. **Cookie sozlamalari.** Login muvaffaqiyatli bo'lganda session cookie o'rnatuvchi kodni yoz. Production uchun barcha xavfsizlik atributlarini (`HttpOnly`, `Secure`, `SameSite`, `Max-Age`, `Path`) to'g'ri qo'y va har birini izohla.

9. **Rate limiter strategiyasi.** Login endpoint'i uchun ikki qatlamli rate limiting yoz: IP bo'yicha umumiy + email bo'yicha qattiq. Account lockout'ning DoS xavfini qanday yumshatasan? Distributed muhitda nima o'zgaradi?

10. **Input validatsiya va sanitizatsiya.** `POST /comments` endpoint uchun validatsiya + XSS sanitizatsiyasini yoz. Foydalanuvchi izohlari boshqa foydalanuvchilarga ko'rsatiladi. Server va render bosqichida nimalarni qilasan?

---

← [Backend bo'limiga qaytish](./README.md)
