# Backend/API Deploy va CORS

Frontend'ni Vercel'ga qo'ydingiz, u ochilyapti — lekin ma'lumot kelmayapti, tugmalar ishlamayapti. Sabab: **backend (API)** hali internetda yo'q, u faqat sizning `localhost:3000` da ishlayapti. Bu fayl backend'ni qayerga va qanday deploy qilishni (Railway/Render/Fly.io yoki VPS), environment variable'larni to'g'ri sozlashni va deploy'da eng ko'p og'ritadigan muammo — **CORS xatosini** boshdan-oyoq hal qilishni ko'rsatadi. Misollar Node.js/Express uchun, lekin mantiq har qanday backend'ga tegishli.

## Mundarija

- [Backend'ni qayerga deploy qilish](#backendni-qayerga-deploy-qilish)
- [Node.js API'ni Railway'ga deploy (qadama-qadam)](#nodejs-apini-railwayga-deploy)
- [Node.js API'ni Render'ga deploy](#nodejs-apini-renderga-deploy)
- [Environment variable'lar va secret'lar](#environment-variablelar-va-secretlar)
- [Health check endpoint](#health-check-endpoint)
- [CORS batafsil](#cors-batafsil)
- [Express'da cors middleware bilan to'g'rilash](#expressda-cors-middleware-bilan-tog'rilash)
- [Production'da to'g'ri origin (wildcard xavfi)](#productionda-tog'ri-origin)
- [API'ni frontend bilan ulash](#apini-frontend-bilan-ulash)
- [API'ga domain qo'shish (api.example.com)](#apiga-domain-qoshish)
- [Savol-Javob (Troubleshooting)](#savol-javob-troubleshooting)
- [Amaliy Checklist](#amaliy-checklist)

---

## Backend'ni qayerga deploy qilish

Backend'ni joylashtirishning uchta asosiy yo'li bor. Har birining o'ziga yarasha o'rni bor:

### 1. PaaS (Platform as a Service) — Railway, Render, Fly.io

Sizga faqat kod kerak, qolgan hammasini platforma qiladi (server, OS, deploy, SSL, monitoring).

| Platforma | Afzallik | Kamchilik | Narx (2026) |
|-----------|----------|-----------|-------------|
| **Railway** | Eng oson, git push → deploy, DB birga | Bepul limit kam | $5/oy dan (usage-based) |
| **Render** | Bepul reja bor, sodda | Bepul reja "uxlaydi" (15 daq harakatsiz) | Bepul / $7/oy dan |
| **Fly.io** | Tez, global (edge), Docker | Biroz murakkabroq | Usage-based, arzon boshlanish |

**✅ Boshlovchiga tavsiya:** **Railway** yoki **Render**. VPS bilan boshingizni og'ritmasdan, 10 daqiqada API internetda bo'ladi.

### 2. VPS (Virtual Private Server) — DigitalOcean, Hetzner, Contabo

To'liq server sizniki. Hamma narsani o'zingiz sozlaysiz.

- **Afzallik:** To'liq nazorat, arzon (katta yuklamada), cheklovsiz.
- **Kamchilik:** Server administratsiyasi (SSH, Nginx, PM2, security, backup) — o'rganish kerak.
- **Narx:** $4-6/oy dan (Hetzner/Contabo arzon).

To'liq VPS deploy → **05, 06-mavzular** (VPS asoslari, Nginx + PM2 + SSL).

### 3. Serverless — Vercel/Netlify Functions, AWS Lambda

Server yo'q — faqat funksiyalar. So'rov kelganda funksiya ishga tushadi, tugagach o'chadi.

- **Afzallik:** Faqat ishlatilganda to'laysiz, avtomatik masshtablanadi, kam trafikda ~bepul.
- **Kamchilik:** "Cold start" (birinchi so'rov sekin), uzoq/og'ir vazifalar uchun mos emas, state saqlamaydi.
- **Qachon:** Kichik API, webhook, forma yuborish, oddiy CRUD.

**💡 Tushuncha:** Agar frontend Vercel'da bo'lsa, kichik API'ni **Vercel Functions** (`/api` papka) sifatida shu yerda qo'yishingiz mumkin — CORS ham muammo bo'lmaydi (bir xil domen). Lekin to'laqonli, doim ishlab turadigan backend uchun Railway/Render qulayroq.

---

## Node.js API'ni Railway'ga deploy

Oddiy Express API'ni Railway'ga qo'yamiz. Loyihangiz GitHub'da bo'lsin.

### 1-qadam: Kod tayyor bo'lsin

`package.json` da **start** skripti bo'lishi shart va **PORT** ni env'dan olsin:

```json
{
  "scripts": {
    "start": "node index.js"
  }
}
```

```js
// index.js
const express = require("express");
const app = express();

// MUHIM: PORT ni env'dan oling, hardcode qilmang
const PORT = process.env.PORT || 3000;

app.get("/", (req, res) => res.send("API ishlayapti"));

app.listen(PORT, () => console.log(`Server ${PORT}-portda`));
```

**⚠️ Ehtiyot bo'l:** `const PORT = 3000` deb qattiq yozib qo'ymang. Railway/Render o'zi PORT beradi (`process.env.PORT` orqali). Hardcode qilsangiz, deploy "ishlamaydi" yoki port to'qnashuvi bo'ladi.

### 2-qadam: Railway'da loyiha yaratish

1. `railway.app` da GitHub bilan kiring.
2. **New Project → Deploy from GitHub repo**.
3. Repozitoriyangizni tanlang.
4. Railway avtomatik Node.js ekanligini aniqlaydi, `npm install` va `npm start` ni o'zi ishlatadi.

### 3-qadam: Build/Start command (kerak bo'lsa)

Railway odatda o'zi topadi. Agar kerak bo'lsa **Settings → Deploy** da qo'lda kiriting:

```text
Build Command:  npm install
Start Command:  npm start
```

TypeScript bo'lsa:

```text
Build Command:  npm install && npm run build
Start Command:  node dist/index.js
```

### 4-qadam: Environment variable qo'shish

**Variables** bo'limida secret'larni kiriting (pastda batafsil).

### 5-qadam: Domain olish

**Settings → Networking → Generate Domain** → `your-api.up.railway.app` manzilini beradi. Brauzerda ochib tekshiring — "API ishlayapti" chiqsa, tayyor.

**💡 Tushuncha:** Railway har `git push` da avtomatik qayta deploy qiladi (CI/CD). Kod'ni GitHub'ga push qilsangiz, bir necha daqiqada yangi versiya jonli bo'ladi.

---

## Node.js API'ni Render'ga deploy

Render'da jarayon deyarli bir xil:

1. `render.com` → **New → Web Service**.
2. GitHub repo'ni ulang.
3. Sozlamalar:

```text
Environment:    Node
Build Command:  npm install
Start Command:  npm start
```

4. **Environment** bo'limida env variable'larni qo'shing.
5. **Create Web Service** → deploy boshlanadi.
6. Render sizga `your-api.onrender.com` domenini beradi (SSL avtomatik).

**⚠️ Ehtiyot bo'l:** Render'ning **bepul rejasi** 15 daqiqa harakatsizlikdan keyin serverni "uxlatadi". Birinchi so'rov 30-50 soniya kutadi (cold start). Jiddiy loyihada pullik ($7/oy) rejaga o'ting yoki uptime-monitor bilan "uyg'oq" tuting.

---

## Environment variable'lar va secret'lar

**Environment variable** (env) — bu koddan tashqarida saqlanadigan sozlama: ma'lumotlar bazasi manzili, API kalitlar, parollar. Kodda `process.env.NOM` orqali o'qiladi.

```js
const dbUrl = process.env.DATABASE_URL;
const apiKey = process.env.STRIPE_SECRET_KEY;
```

Lokal ishda bularni `.env` faylda saqlaysiz:

```text
# .env
DATABASE_URL=postgres://user:pass@localhost:5432/mydb
STRIPE_SECRET_KEY=sk_test_abc123
JWT_SECRET=super-maxfiy-kalit
```

### NEGA .env'ni git'ga qo'ymaslik kerak

**⚠️ Ehtiyot bo'l:** `.env` faylida **parollar va maxfiy kalitlar** bor. Uni git'ga (ayniqsa public repo'ga) qo'ysangiz, butun dunyo ko'radi. Botlar GitHub'ni doim skanerlaydi va ochiq kalitlarni bir necha daqiqada topib, DB'ingizni buzadi yoki hisobingizdan pul sarflaydi. Bu **eng ko'p uchraydigan xavfsizlik xatosi**.

Har doim `.gitignore` ga qo'shing:

```text
# .gitignore
.env
.env.local
node_modules/
```

O'rniga `.env.example` yarating (maxfiy qiymatsiz, faqat kalit nomlari) — jamoadagilar nima kerakligini bilsin:

```text
# .env.example (buni git'ga qo'ysa bo'ladi)
DATABASE_URL=
STRIPE_SECRET_KEY=
JWT_SECRET=
```

### Deploy'da env'ni qanday sozlash

Platformada `.env` fayl **yuklanmaydi** — o'rniga panel orqali kiritasiz:
- **Railway:** loyiha → **Variables** → har birini `NOM` = `qiymat` shaklida.
- **Render:** service → **Environment** → **Add Environment Variable**.

Platforma bularni ishga tushirishda `process.env` ga joylashtiradi. Kod o'zgarmaydi.

**💡 Tushuncha:** Agar `.env` ni allaqachon git'ga push qilib qo'ygan bo'lsangiz, `.gitignore` ga qo'shishning o'zi kifoya emas — u tarixda qoladi. Barcha ochilib qolgan kalitlarni **darhol yangilang** (rotate), keyin git tarixidan tozalang.

---

## Health check endpoint

Health check — bu "server tirikmi?" degan savolga javob beradigan oddiy endpoint. Platformalar va monitoring vositalari uni doimiy tekshiradi.

```js
app.get("/health", (req, res) => {
  res.status(200).json({ status: "ok", uptime: process.uptime() });
});
```

**💡 Tushuncha:** Railway/Render bu endpoint'ni tekshirib, server "sog'lom" deb bilsagina trafikni yo'naltiradi. Deploy sozlamalarida "Health Check Path" ni `/health` qilib bering. Bu, ayniqsa, nol-downtime deploy uchun muhim.

Managed database (PostgreSQL/MySQL) ulash — Railway/Render'da bir tugma bilan qo'shiladi va `DATABASE_URL` env avtomatik beriladi. VPS'da o'zingiz o'rnatasiz → **08-mavzu**.

---

## CORS batafsil

Backend deploy'da eng ko'p och qoldiradigan xato — **CORS**. Console'da qip-qizil xat chiqadi, hamma narsa to'g'ri ko'rinadi, lekin ishlamaydi. Keling, tubidan tushunamiz.

### CORS nima va nega browser bloklaydi

**CORS** (Cross-Origin Resource Sharing) — brauzerning xavfsizlik qoidasi. U **same-origin policy** (bir xil manba siyosati) ustiga qurilgan.

**💡 Tushuncha:** **Origin** = protokol + domen + port. Masalan `https://example.com:443`. Brauzer sukut bo'yicha bitta origin'dagi sahifaga **boshqa origin'dagi** API'ga so'rov yuborishga ruxsat bermaydi — bu sizning ma'lumotlaringizni himoya qiladi (zararli sayt sizning bankingiz API'siga so'rov yuborolmasin).

Deploy'da nima bo'ladi:
- Frontend: `https://example.com`
- Backend: `https://api.example.com` (yoki `your-api.up.railway.app`)

Bular **boshqa origin** (domen har xil). Frontend backend'ga `fetch` qilsa, brauzer bloklaydi — **agar backend ruxsat berayotganini aytmasa**.

**⚠️ Muhim:** CORS'ni **brauzer** amalga oshiradi, backend emas. Shuning uchun Postman yoki `curl` da API mukammal ishlaydi (ular CORS'ni tekshirmaydi), lekin brauzerda xato beradi. Bu ko'pchilikni chalg'itadi.

### Preflight (OPTIONS so'rovi)

Ba'zi so'rovlar (POST + JSON, PUT, DELETE, maxsus header'lar) uchun brauzer avval **preflight** so'rovini yuboradi:

```text
Browser → OPTIONS /api/users → Server
          "Men POST yubormoqchiman, ruxsatmi?"
Server  → "Ha, bu origin'ga ruxsat" (Access-Control-Allow-* header'lar)
Browser → endi asl POST so'rovini yuboradi
```

**💡 Tushuncha:** Preflight — bu brauzerning "ruxsat so'rashi". Agar server preflight'ga to'g'ri javob bermasa, asl so'rov umuman yuborilmaydi. Network tab'da `OPTIONS` so'rovi qizil bo'lsa — CORS muammosi shu.

### CORS xatosini qanday o'qish (console)

Brauzer console'da (F12) tipik xato:

```text
Access to fetch at 'https://api.example.com/users'
from origin 'https://example.com' has been blocked by CORS policy:
No 'Access-Control-Allow-Origin' header is present on the requested resource.
```

Buni shunday o'qing:
- `from origin 'https://example.com'` — **kim** so'rayapti (frontend).
- `at 'https://api.example.com/users'` — **qayerdan** (backend).
- `No 'Access-Control-Allow-Origin' header` — backend "ruxsat" header'ini yubormayapti.

Yechim: backend'ga o'sha ruxsat header'ini qo'shish (pastda).

---

## Express'da cors middleware bilan to'g'rilash

Express'da `cors` paketi bu ishni oson qiladi:

```bash
npm install cors
```

### Eng oddiy (development uchun — hamma origin'ga ruxsat)

```js
const cors = require("cors");
app.use(cors()); // BARCHA origin'ga ruxsat — faqat test uchun
```

Bu `Access-Control-Allow-Origin: *` qo'yadi. Ishlaydi, lekin production uchun **xavfli** (pastda).

### To'g'ri sozlash (production)

```js
const cors = require("cors");

const corsOptions = {
  origin: "https://example.com",          // faqat sizning frontend
  methods: ["GET", "POST", "PUT", "DELETE"],
  allowedHeaders: ["Content-Type", "Authorization"],
  credentials: true,                       // cookie/auth yuborilsa
};

app.use(cors(corsOptions));
```

Bir nechta origin (masalan localhost + production) uchun:

```js
const allowedOrigins = [
  "http://localhost:5173",       // dev (Vite)
  "https://example.com",         // production
  "https://www.example.com",
];

const corsOptions = {
  origin: function (origin, callback) {
    // Postman/server-to-server (origin yo'q) — ruxsat
    if (!origin || allowedOrigins.includes(origin)) {
      callback(null, true);
    } else {
      callback(new Error("CORS: bu origin'ga ruxsat yo'q"));
    }
  },
  credentials: true,
};

app.use(cors(corsOptions));
```

**💡 Tushuncha:** `app.use(cors(...))` ni **barcha route'lardan oldin** qo'ying. Aks holda ba'zi route'lar CORS header'siz qoladi. `cors` middleware preflight (`OPTIONS`) so'rovlarni ham avtomatik hal qiladi.

### credentials (cookie/auth) bilan muhim nuqta

```js
// Frontend tomonda:
fetch("https://api.example.com/me", {
  credentials: "include",   // cookie yuborish uchun
});
```

**⚠️ Ehtiyot bo'l:** Agar `credentials: true` ishlatsangiz, backend origin'ni `*` (wildcard) qila **olmaysiz** — aniq origin yozishingiz shart. Brauzer `credentials` + `*` birga kelsa, so'rovni bloklaydi. Bu keng uchraydigan tuzoq.

---

## Production'da to'g'ri origin

**⚠️ Ehtiyot bo'l:** `app.use(cors())` yoki `origin: "*"` — production'da **xavfli**. Bu **istalgan sayt** sizning API'ingizga so'rov yuborishiga ruxsat beradi. Zararli sayt foydalanuvchi nomidan so'rov yuborishi mumkin.

To'g'ri yondashuv:
- **Development:** `localhost` origin'lariga ruxsat (yuqoridagi ro'yxat).
- **Production:** **faqat** o'z domeningiz (`https://example.com`).

Origin'ni ham env orqali sozlash — eng toza yo'l:

```js
const allowedOrigins = process.env.ALLOWED_ORIGINS.split(",");
// .env: ALLOWED_ORIGINS=http://localhost:5173,https://example.com
```

Shunda dev va prod'da kod o'zgarmaydi, faqat env o'zgaradi.

---

## API'ni frontend bilan ulash

Frontend API manzilini bilishi kerak. Uni ham **env** orqali bering — kodga hardcode qilmang:

```js
// Frontend (Vite/React)
const API_URL = import.meta.env.VITE_API_URL;

fetch(`${API_URL}/users`)
  .then((res) => res.json())
  .then((data) => console.log(data));
```

```text
# Frontend .env (dev)
VITE_API_URL=http://localhost:3000

# Frontend .env (production — Vercel panelida)
VITE_API_URL=https://api.example.com
```

**💡 Tushuncha:** Frontend env nomi platformaga bog'liq: Vite — `VITE_` prefiks, Next.js — brauzerda ishlatilsa `NEXT_PUBLIC_` prefiks. Bu prefikssiz o'zgaruvchilar brauzer koddan ko'rinmaydi. Vercel/Netlify panelida bu env'ni qo'shib, qayta deploy qiling.

---

## API'ga domain qo'shish (api.example.com)

`your-api.up.railway.app` o'rniga chiroyli `api.example.com` ishlatish:

1. Railway/Render panelida **Custom Domain** qo'shing → `api.example.com`.
2. Platforma sizga CNAME beradi.
3. DNS panelida (02-mavzu) CNAME record qo'shing:

```text
Type: CNAME
Name: api
Value: your-api.up.railway.app
```

4. Platforma SSL'ni avtomatik beradi.
5. CORS origin va frontend `VITE_API_URL` ni yangilang → `https://api.example.com`.

**💡 Tushuncha:** Frontend `example.com`, backend `api.example.com` — bu keng tarqalgan, toza arxitektura. Domain va DNS bo'yicha to'liq ma'lumot → **02-mavzu**.

---

## Savol-Javob (Troubleshooting)

### ❓ Console'da "No 'Access-Control-Allow-Origin' header" — nima qilay?

**✅ Javob:** Backend CORS ruxsatini yubormayapti. Express'da `app.use(cors({ origin: "https://frontend-manzilingiz" }))` qo'shing va **barcha route'dan oldin** joylashtiring. Deploy'dan keyin brauzerni hard-refresh (Ctrl+Shift+R) qiling.

### ❓ Postman'da ishlayapti, lekin brauzerda CORS xatosi?

**✅ Javob:** Bu normal — CORS'ni faqat **brauzer** tekshiradi, Postman/curl emas. API'ning o'zi to'g'ri ishlayapti; muammo — backend brauzerga CORS header'larini yubormayapti. `cors` middleware qo'shing.

### ❓ `credentials: 'include'` ishlatyapman, lekin baribir bloklanyapti?

**✅ Javob:** `credentials` bilan origin `*` bo'la olmaydi. Backend'da aniq origin yozing (`origin: "https://example.com"`, `*` emas) va `credentials: true` qo'ying. Frontend'da ham `credentials: "include"` bo'lsin. Ikkala tomon mos kelishi shart.

### ❓ Deploy bo'ldi, lekin "Application failed to respond" / sayt ochilmaydi?

**✅ Javob:** Deyarli har doim **PORT** muammosi. `app.listen(3000)` o'rniga `app.listen(process.env.PORT || 3000)` yozing. Platforma o'z portini env orqali beradi; hardcode qilingan port ishlamaydi.

### ❓ Lokal ishlaydi, deploy'da "Cannot find module" xatosi?

**✅ Javob:** Kutubxona `devDependencies` da qolib ketgan yoki `node_modules` git'ga qo'yilgan. Kerakli paketlar `dependencies` da bo'lsin (`npm install paket --save`), `node_modules` ni `.gitignore` ga qo'ying (platforma o'zi `npm install` qiladi).

### ❓ Environment variable o'qilmayapti (`undefined`)?

**✅ Javob:** Tekshiring: (1) env platforma panelida (Railway Variables / Render Environment) qo'shilganmi; (2) qo'shgandan keyin **qayta deploy** qildingizmi (env o'zgarishi yangi deploy talab qiladi); (3) nomi aynan to'g'rimi (katta-kichik harf muhim). Lokal'da `dotenv` paketini `require("dotenv").config()` bilan yuklaganmisiz.

### ❓ Network tab'da OPTIONS so'rovi qizil (failed) — nega?

**✅ Javob:** Bu preflight muvaffaqiyatsiz. Backend `OPTIONS` so'roviga javob bermayapti. `cors` middleware buni avtomatik hal qiladi — uni to'g'ri o'rnatganingizni tekshiring. Qo'lda route yozgan bo'lsangiz, `app.options("*", cors())` qo'shing.

### ❓ Render'da birinchi so'rov juda sekin (30 soniya)?

**✅ Javob:** Bepul rejada server 15 daqiqa harakatsizlikdan keyin uxlaydi (cold start). Yechim: pullik rejaga o'tish, yoki UptimeRobot kabi vosita bilan har 10 daqiqada `/health` ga so'rov yuborib "uyg'oq" tutish.

### ❓ `.env` faylni git'ga push qilib yuboribman — nima qilay?

**✅ Javob:** Darhol harakat qiling: (1) barcha ochilib qolgan kalit/parollarni **yangilang** (rotate) — eskisini o'chirib, yangi yarating; (2) `.env` ni `.gitignore` ga qo'shing; (3) git tarixidan tozalang (`git filter-repo` yoki BFG). Faqat `.gitignore` ga qo'shishning o'zi kifoya emas — kalit tarixda qolgan.

### ❓ Frontend deploy'da API'ga ulanmayapti, lekin lokal'da ishlaydi?

**✅ Javob:** Frontend hali `localhost:3000` ga so'rov yuboryapti. `VITE_API_URL` (yoki `NEXT_PUBLIC_API_URL`) env'ni production API manziliga sozlab, frontend'ni **qayta deploy** qiling. Frontend env build vaqtida "muhrlanadi", shuning uchun qayta build shart.

### ❓ CORS origin'da `https://example.com/` (oxirida slash) yozsam bo'ladimi?

**✅ Javob:** Yo'q — origin'da oxirgi slash **bo'lmasligi** kerak. To'g'ri: `https://example.com`. Noto'g'ri: `https://example.com/`. Oxiriga slash qo'ysangiz, brauzer origin'ni mos kelmadi deb bloklaydi. Kichik xato, katta bosh og'riq.

### ❓ Serverless (Vercel Functions) ishlatsam CORS bo'ladimi?

**✅ Javob:** Agar frontend va API bir xil domenda bo'lsa (`example.com` sayt, `example.com/api` funksiya) — CORS umuman muammo bo'lmaydi (same-origin). Ular ajralgan domenda bo'lsagina CORS kerak. Kichik loyihalar uchun bu qulay yechim.

---

## Amaliy Checklist

- [ ] Deploy joyi tanlandi (PaaS / VPS / serverless — loyihaga mos)
- [ ] `package.json` da `start` skripti bor
- [ ] PORT `process.env.PORT` dan olinadi (hardcode emas)
- [ ] GitHub repo Railway/Render'ga ulandi
- [ ] Environment variable'lar panel orqali qo'shildi
- [ ] `.env` `.gitignore` da (git'ga hech qachon push qilinmagan)
- [ ] `.env.example` yaratildi (jamoa uchun)
- [ ] `/health` endpoint qo'shildi
- [ ] `cors` middleware o'rnatildi va barcha route'dan oldin qo'yildi
- [ ] CORS origin production'da aniq domen (`*` emas)
- [ ] credentials ishlatilsa — origin aniq, `*` emas
- [ ] Frontend `VITE_API_URL` env orqali production API'ga ulandi
- [ ] Brauzerda CORS xatosi yo'q (F12 → Console toza)
- [ ] Kerak bo'lsa `api.example.com` custom domain qo'shildi + SSL

---

← [Deployment bo'limiga qaytish](./README.md)
