# Frontend Deploy (Static, React, Next.js)

Frontend loyihangizni yozib bo'lgandan keyin uni internetga chiqarish kerak — buni **deployment** deb ataymiz. Bu qo'llanmada oddiy HTML/CSS/JS static saytdan tortib React (Vite) va Next.js loyihalarini bepul platformalarga qanday joylashtirishni qadama-qadam o'rganamiz. Barcha misollar 2026-yil holatiga mos.

## Mundarija

- [Deploy nima va "build" jarayoni](#deploy-nima-va-build-jarayoni)
- [Static site vs SPA vs SSR/SSG](#static-site-vs-spa-vs-ssrssg)
- [(a) Oddiy HTML/CSS/JS deploy](#a-oddiy-htmlcssjs-deploy)
- [(b) React (Vite) deploy](#b-react-vite-deploy)
- [(c) Next.js deploy](#c-nextjs-deploy)
- [Environment variable'lar](#environment-variablelar)
- [Preview deployment va bepul tier cheklovlari](#preview-deployment-va-bepul-tier-cheklovlari)
- [Amaliy Q&A (Troubleshooting)](#amaliy-qa-troubleshooting)
- [Amaliy Checklist](#amaliy-checklist)

---

## Deploy nima va "build" jarayoni

**Deploy** — bu sizning kodingizni o'zingizning kompyuteringizdan (localhost) internetdagi serverga ko'chirib, hamma ko'ra oladigan qilib joylashtirish jarayoni. `localhost:3000` faqat sizda ishlaydi; deploy'dan keyin esa `https://mysite.com` orqali dunyoning istalgan nuqtasidan kirish mumkin.

Zamonaviy frontend loyihalarda kodni to'g'ridan-to'g'ri serverga tashlab bo'lmaydi — avval **build** qilish kerak.

**💡 Tushuncha:** "Build" — sizning yozgan kodingizni (React JSX, TypeScript, SCSS, modern JS) brauzer tushunadigan sof HTML, CSS va JavaScript'ga aylantirish jarayoni. Build vositalari (Vite, Webpack, Next.js compiler) kodni bundle qiladi, minify qiladi (bo'sh joy va izohlarni olib tashlaydi), optimallashtiradi.

Build jarayonining natijasi odatda bitta papkaga tushadi:

- Vite → `dist/`
- Create React App → `build/`
- Next.js (static export) → `out/`

Deploy platformasi aynan shu papkani oladi va serverga joylaydi.

```bash
# Tipik build buyruq
npm run build
```

Bu buyruq `package.json` dagi `"scripts"` bo'limidagi `"build"` skriptini ishga tushiradi.

---

## Static site vs SPA vs SSR/SSG

Deploy qanday bo'lishi loyihangiz **qaysi turga** kirishiga bog'liq. Bu eng muhim tushuncha.

**💡 Tushuncha:** Uch xil asosiy render turi bor. Ular sahifa qayerda va qachon "yasaladi" degan savolga javob beradi.

### 1. Static site (statik sayt)

Sahifalar **oldindan** tayyor HTML fayllar sifatida turadi. Foydalanuvchi kirsa, server tayyor faylni beradi. Hech qanday hisob-kitob real vaqtda bo'lmaydi.

- Misol: oddiy HTML/CSS/JS sayt, blog, landing page.
- Deploy: eng oson va eng arzon (ko'pincha bepul). Faqat fayllarni serverga tashlash.

### 2. SPA (Single Page Application)

Server bitta `index.html` beradi, qolgan hamma narsa JavaScript orqali brauzerda quriladi. Sahifalar o'rtasida o'tish (routing) JS orqali bo'ladi, sahifa qayta yuklanmaydi.

- Misol: React (Vite), Vue SPA.
- Deploy: static kabi, lekin **routing muammosi** bor (pastda ko'ramiz).

### 3. SSR / SSG / ISR

- **SSR (Server-Side Rendering):** har so'rovda server HTML'ni real vaqtda yasaydi. Dinamik ma'lumot uchun yaxshi.
- **SSG (Static Site Generation):** HTML build vaqtida oldindan yasaladi, static kabi tez.
- **ISR (Incremental Static Regeneration):** SSG'ning takomillashgani — sahifalar oldindan yasaladi, lekin ma'lum vaqt oralig'ida yangilanadi.
- Misol: Next.js.
- Deploy: server kerak (yoki Vercel kabi maxsus platforma).

**⚠️ Ehtiyot bo'l:** SSR loyihani oddiy static hosting (GitHub Pages) ga tashlab bo'lmaydi — u yerda server yo'q. SSR uchun Node.js server yoki Vercel/Netlify Functions kerak. Bu farqni oldindan bilish deploy vaqtidagi ko'p muammolarni oldini oladi.

| Tur | Qayerda render | Deploy joyi | Misol |
|-----|----------------|-------------|-------|
| Static | Oldindan (build) | Har qanday static host | HTML sayt |
| SPA | Brauzerda (JS) | Static host + redirect | React Vite |
| SSG | Oldindan (build) | Static host | Next.js SSG |
| SSR | Serverda (har so'rov) | Node server / Vercel | Next.js SSR |

---

## (a) Oddiy HTML/CSS/JS deploy

Sizda `index.html`, `style.css`, `script.js` bor deylik. Build kerak emas — to'g'ridan-to'g'ri deploy qilamiz. Bir nechta bepul variant bor.

### Qaysi platforma qachon?

- **GitHub Pages** — kod allaqachon GitHub'da bo'lsa, portfolio/blog uchun eng oson va bepul.
- **Netlify** — drag & drop kerak bo'lsa (build'siz eng tez usul), yoki qulay CI/CD kerak bo'lsa.
- **Cloudflare Pages** — eng tez CDN va cheksiz bandwidth kerak bo'lsa.
- **Vercel** — keyinchalik Next.js'ga o'tishni rejalashtirsangiz.

### GitHub Pages (qadama-qadam)

1. GitHub'da yangi repository yarating (masalan `my-site`).
2. Loyihani push qiling:

```bash
git init
git add .
git commit -m "Initial commit"
git branch -M main
git remote add origin https://github.com/USERNAME/my-site.git
git push -u origin main
```

3. Repository sahifasida **Settings → Pages** bo'limiga o'ting.
4. **Source** bo'limida `Deploy from a branch` ni tanlang.
5. **Branch** sifatida `main` va papka `/ (root)` ni tanlab **Save** bosing.
6. 1-2 daqiqadan so'ng saytingiz `https://USERNAME.github.io/my-site/` da ochiladi.

**⚠️ Ehtiyot bo'l:** GitHub Pages'da sayt asosiy papkada emas, `/my-site/` subpath'da ochiladi. Shuning uchun CSS/JS/rasm yo'llarini **nisbiy (relative)** qilib yozing (`./style.css`, `href="./style.css"`), aks holda `/style.css` topilmaydi.

### Netlify (drag & drop)

1. [netlify.com](https://netlify.com) ga kiring va ro'yxatdan o'ting.
2. Dashboard'da **"Add new site" → "Deploy manually"** ni tanlang.
3. Loyiha papkangizni to'g'ridan-to'g'ri brauzerga **sudrab tashlang** (drag & drop).
4. Bir necha soniyada `random-name.netlify.app` manzili beriladi.

### Netlify (git orqali — tavsiya etiladi)

1. Kodni GitHub'ga push qiling.
2. Netlify'da **"Add new site" → "Import an existing project"** → GitHub'ni ulang.
3. Repository'ni tanlang. Build'siz static uchun **Build command** ni bo'sh qoldiring, **Publish directory** ni `.` (yoki loyiha papkasi) qiling.
4. **Deploy** bosing. Endi har `git push` avtomatik yangi deploy qiladi.

### Cloudflare Pages

1. [dash.cloudflare.com](https://dash.cloudflare.com) → **Workers & Pages → Create → Pages**.
2. GitHub repository'ni ulang.
3. Build settings: static uchun build command bo'sh, output directory loyiha papkasi.
4. **Save and Deploy**. Manzil: `project.pages.dev`.

**💡 Tushuncha:** Cloudflare Pages bepul tarifda **cheksiz bandwidth** beradi — trafik ko'p bo'ladigan saytlar uchun ayni muddao.

---

## (b) React (Vite) deploy

Vite bilan yasalgan React loyiha — bu **SPA**. Uni deploy qilishdan oldin build qilish shart.

### 1-qadam: Build

```bash
npm install
npm run build
```

Natijada `dist/` papka paydo bo'ladi. Ichida `index.html` va `assets/` (bundle qilingan JS/CSS) bo'ladi.

Build'ni mahalliy tekshirish:

```bash
npm run preview
```

### 2-qadam: Vercel yoki Netlify'ga ulash

**Vercel (git orqali):**

1. Kodni GitHub'ga push qiling.
2. [vercel.com](https://vercel.com) → **Add New → Project** → GitHub repo'ni import qiling.
3. Vercel Vite'ni avtomatik aniqlaydi:
   - Framework Preset: `Vite`
   - Build Command: `npm run build`
   - Output Directory: `dist`
4. **Deploy** bosing. Tayyor.

**Netlify (git orqali):**

1. **Import an existing project** → repo'ni tanlang.
2. Build settings:
   - Build command: `npm run build`
   - Publish directory: `dist`
3. **Deploy**.

### SPA routing muammosi (404 on refresh)

Bu React deploy'da **eng ko'p uchraydigan muammo**. Sabab: React Router `/about` kabi yo'llarni brauzerda JS orqali boshqaradi. Lekin foydalanuvchi to'g'ridan-to'g'ri `mysite.com/about` ga kirsa yoki sahifani refresh qilsa, server `/about` degan **fayl** qidiradi — u yo'q — **404** qaytaradi.

**💡 Tushuncha:** Yechim — serverga aytish: "Qaysi yo'l so'ralsa ham, `index.html` ni ber. Qolganini React o'zi hal qiladi." Buni **redirect / rewrite** deb ataladi.

**Netlify yechimi** — loyiha ildizida yoki `public/` papkada `_redirects` fayli yarating:

```
/*    /index.html   200
```

**Vercel yechimi** — loyiha ildizida `vercel.json` yarating:

```json
{
  "rewrites": [
    { "source": "/(.*)", "destination": "/index.html" }
  ]
}
```

**Cloudflare Pages yechimi** — `public/_redirects` fayl (Netlify bilan bir xil sintaksis).

### Vite base path (GitHub Pages uchun)

Agar Vite loyihani GitHub Pages subpath'ida deploy qilsangiz, `vite.config.js` da `base` ni sozlang:

```js
export default defineConfig({
  base: '/my-repo-name/',
  // ...
})
```

---

## (c) Next.js deploy

Next.js — React freymvorki bo'lib, SSR, SSG, ISR va API route'larni qo'llab-quvvatlaydi. Deploy'ning eng oson va tavsiya etilgan yo'li — **Vercel** (Next.js'ni yaratgan kompaniya).

### Vercel'ga deploy (birinchi tanlov)

1. Kodni GitHub'ga push qiling.
2. [vercel.com](https://vercel.com) → **Add New → Project** → repo'ni import qiling.
3. Vercel Next.js'ni **avtomatik aniqlaydi** — hech narsa sozlash shart emas:
   - Build Command: `next build`
   - Output: avtomatik
4. **Deploy** bosing. 1-2 daqiqada tayyor.

**💡 Tushuncha:** Vercel'da SSR, SSG, ISR, API route'lar hammasi **avtomatik** ishlaydi. Siz hech qanday server sozlamaysiz — Vercel har sahifa turini tushunib, kerakli infratuzilmani o'zi yaratadi (SSR uchun serverless functions, SSG uchun CDN).

### SSR/SSG/ISR deploy'da qanday ishlaydi

- **SSG:** sahifa `next build` vaqtida bir marta yasaladi va CDN'da static fayl sifatida turadi. Eng tez.
- **SSR:** har so'rovda server (serverless function) HTML yasaydi. Foydalanuvchiga qarab o'zgaradigan sahifalar uchun.
- **ISR:** `revalidate` bilan sozlanadi — sahifa static, lekin belgilangan vaqtda fonda yangilanadi:

```js
// app/page.js (App Router)
export const revalidate = 60 // har 60 soniyada yangilanadi
```

### Boshqa platformalar

**Netlify** — Next.js'ni qo'llab-quvvatlaydi (Netlify'ning Next.js runtime'i orqali). Import qilganda avtomatik aniqlaydi. Ba'zi eng yangi Next.js xususiyatlari kechikib qo'llanilishi mumkin.

**Self-host (`next start`)** — o'z serveringizda (VPS, Docker):

```bash
npm run build      # next build
npm run start      # next start (Node.js server ishga tushadi)
```

**⚠️ Ehtiyot bo'l:** `next start` uchun serveringizda Node.js doim ishlab turishi kerak (masalan `pm2` yoki `systemd` bilan). Bu GitHub Pages kabi bepul static hosting'da ishlamaydi. Faqat static export (`output: 'export'`) qilsangizgina static hosting mumkin, lekin unda SSR/ISR/API route'lar ishlamaydi.

### Static export (agar SSR kerak bo'lmasa)

```js
// next.config.js
module.exports = {
  output: 'export',
}
```

```bash
npm run build   # 'out/' papka yasaladi -> static hostingga tashlash mumkin
```

---

## Environment variable'lar

Environment variable (env) — bu kodga tashqaridan beriladigan sozlama qiymatlari (API kalitlari, backend URL). Ularni kod ichiga yozib qo'ymaslik kerak.

### Vite'da

Faqat `VITE_` prefiksli o'zgaruvchilar brauzerga chiqadi:

```bash
# .env
VITE_API_URL=https://api.example.com
```

```js
console.log(import.meta.env.VITE_API_URL)
```

### Next.js'da (NEXT_PUBLIC_ prefiks)

**💡 Tushuncha:** Next.js'da `NEXT_PUBLIC_` prefiksli o'zgaruvchilar **brauzerga** yuboriladi (public). Prefiksz o'zgaruvchilar faqat **serverda** qoladi (maxfiy, xavfsiz).

```bash
# .env.local
NEXT_PUBLIC_SITE_URL=https://mysite.com   # brauzerda ko'rinadi
DATABASE_PASSWORD=secret123               # faqat serverda, maxfiy
```

```js
// Brauzerda ham, serverda ham ishlaydi
console.log(process.env.NEXT_PUBLIC_SITE_URL)

// Faqat server komponent / API route ichida ishlaydi
console.log(process.env.DATABASE_PASSWORD)
```

**⚠️ Ehtiyot bo'l:** API kalit yoki parolni HECH QACHON `NEXT_PUBLIC_` yoki `VITE_` prefiksi bilan qo'ymang — u brauzer kodiga chiqib, hamma ko'ra oladi.

### Platformada env sozlash

Vercel/Netlify dashboard'da: **Settings → Environment Variables** bo'limiga o'zgaruvchilarni qo'shing. `.env` faylni git'ga push qilmang (`.gitignore` ga qo'shing).

**⚠️ Ehtiyot bo'l:** Env o'zgaruvchini qo'shgandan keyin **qayta deploy** qilish kerak — build vaqtida o'qiladigan o'zgaruvchilar faqat yangi build'da ta'sir qiladi.

---

## Preview deployment va bepul tier cheklovlari

**💡 Tushuncha:** Vercel/Netlify har `git push` (yoki Pull Request) uchun avtomatik **Preview deployment** — alohida URL bilan test versiyasini yaratadi. Asosiy (production) saytga tegmasdan o'zgarishlarni ko'rish mumkin. PR merge bo'lsa production yangilanadi.

**Custom domain** — barcha platformalar o'z domeningizni (`mysite.uz`) ulashga imkon beradi. Bu keyingi mavzuda batafsil.

### Bepul tier cheklovlari (taxminiy, 2026)

- **Vercel Hobby:** shaxsiy/nokommersiya loyihalar uchun, oylik bandwidth va build vaqti cheklangan, serverless function ishlash vaqti cheklangan.
- **Netlify Free:** oylik bandwidth va build daqiqalari cheklangan.
- **Cloudflare Pages Free:** **cheksiz bandwidth**, oyiga cheklangan build soni.
- **GitHub Pages:** faqat static, repo hajmi va bandwidth uchun yumshoq limitlar, kommersiya uchun mos emas.

**⚠️ Ehtiyot bo'l:** Bepul tarif limitlari platformalarda tez-tez o'zgaradi. Deploy qilishdan oldin har doim rasmiy narx (pricing) sahifasini tekshiring.

---

## Amaliy Q&A (Troubleshooting)

### ❓ `npm run build` xato beryapti, deploy'ni davom ettira olmayapman

**✅ Javob:** Avval **mahalliy** (local) build qiling: `npm run build`. Deploy platformasi ham xuddi shu buyruqni ishlatadi, shuning uchun local'da o'tmasa, platformada ham o'tmaydi. Xato matnini o'qing — ko'pincha import qilinmagan modul, TypeScript type xatosi yoki katta-kichik harf farqi (Linux serverda `Header.js` va `header.js` farqli!) sabab bo'ladi.

### ❓ Sayt deploy bo'ldi, lekin ekran oq (blank page), CSS/JS yuklanmayapti

**✅ Javob:** Bu deyarli har doim **noto'g'ri yo'l (path)** muammosi. Brauzerda F12 → Console/Network'da `404` xatolarni ko'ring. Yechim: GitHub Pages subpath'ida bo'lsangiz, Vite'da `base: '/repo-name/'` sozlang; oddiy HTML'da nisbiy yo'l (`./style.css`) ishlating.

### ❓ React saytda bosh sahifa ochiladi, lekin `/about` ni refresh qilsam 404

**✅ Javob:** Bu klassik SPA routing muammosi. Netlify'da `public/_redirects` fayliga `/*  /index.html  200` qo'shing; Vercel'da `vercel.json` da `rewrites` sozlang (yuqorida ko'rsatilgan). Server barcha yo'llarni `index.html` ga yo'naltirishi kerak.

### ❓ Environment variable `undefined` chiqyapti

**✅ Javob:** Uchta narsani tekshiring: (1) Prefiks to'g'rimi — Vite'da `VITE_`, Next.js brauzer o'zgaruvchisida `NEXT_PUBLIC_`. (2) Platform dashboard'ida o'zgaruvchi qo'shilganmi. (3) O'zgaruvchini qo'shgandan keyin **qayta deploy** qilganmisiz. Env build vaqtida "yopishtiriladi", shuning uchun eski build'da yangi qiymat bo'lmaydi.

### ❓ Next.js loyiham Netlify'da SSR/API route ishlamayapti

**✅ Javob:** Agar `next.config.js` da `output: 'export'` yozilgan bo'lsa — bu static export, SSR/API o'chadi. Uni olib tashlang. Yoki eng ishonchli variant — Next.js uchun **Vercel** ishlating; u SSR/ISR/API'ni to'liq va avtomatik qo'llaydi.

### ❓ Local'da ishlaydi, lekin deploy'da modul topilmadi (Module not found)

**✅ Javob:** Sabablar: (1) Fayl nomida katta-kichik harf farqi (Windows farqni sezmaydi, deploy serveri Linux — sezadi). Import va fayl nomini aynan bir xil qiling. (2) Paket `devDependencies` da bo'lib, production build'da o'rnatilmayapti — kerak bo'lsa `dependencies` ga o'tkazing. (3) `node_modules` git'ga tushib qolgan (`.gitignore` ga qo'shing).

### ❓ Deploy juda uzoq davom etyapti yoki timeout bo'lyapti

**✅ Javob:** Katta bog'liqliklar (dependencies) yoki katta rasm/video fayllar sabab bo'lishi mumkin. `node_modules` va build papkasi (`dist/`, `.next/`) git'ga tushmaganini tekshiring (`.gitignore`). Katta media fayllarni optimallashtiring yoki tashqi CDN'ga ko'chiring.

### ❓ Rasmlar (images) deploy'da ko'rinmayapti

**✅ Javob:** (1) Yo'l noto'g'ri — `public/` papkadagi rasmlarga `/image.png` (ildizdan) deb murojaat qiling. (2) Fayl nomida katta-kichik harf yoki bo'sh joy bor. (3) Rasm git'ga push qilinmagan — `git status` bilan tekshiring.

### ❓ Custom domain ulasam "not secure" (SSL) chiqyapti

**✅ Javob:** Vercel/Netlify/Cloudflare domain ulanganidan keyin SSL sertifikatni **avtomatik** (bepul, Let's Encrypt) beradi, lekin bu bir necha daqiqa (ba'zan soatlar) davom etadi. DNS to'g'ri ulanganini kuting. Domain sozlash keyingi mavzuda batafsil ko'riladi.

### ❓ `git push` qildim, lekin sayt yangilanmadi

**✅ Javob:** (1) Deploy qaysi branch'dan bo'lishini tekshiring — platformada `main` (yoki `master`) tanlanganmi. (2) Platform dashboard'da oxirgi deploy'ning holatini (build log) ko'ring — u xato bilan to'xtagan bo'lishi mumkin. (3) Brauzer keshi — `Ctrl+Shift+R` bilan hard refresh qiling.

### ❓ SPA/Vite loyihamda 404 sahifasini o'zim yaratmoqchiman

**✅ Javob:** React Router'da barcha mos kelmagan yo'llar uchun `path="*"` route qo'shing va o'zingizning `<NotFound />` komponentingizni ko'rsating. Server tomonidagi rewrite (`index.html` ga) hali ham kerak — u so'rovni React'ga yetkazadi, keyin React o'z 404'ini ko'rsatadi.

---

## Amaliy Checklist

- [ ] Loyiha turini aniqladim: static / SPA / SSR-SSG (deploy strategiyasini shunga qarab tanladim).
- [ ] Local'da `npm run build` muvaffaqiyatli o'tdi (xatosiz).
- [ ] Build papkasini (`dist/`, `.next/`, `out/`) va `node_modules` ni `.gitignore` ga qo'shdim.
- [ ] Kodni GitHub'ga push qildim.
- [ ] Platformani tanladim: static → GitHub Pages/Netlify/Cloudflare; React → Vercel/Netlify; Next.js → Vercel.
- [ ] Build command va output directory'ni to'g'ri sozladim (`npm run build`, `dist`).
- [ ] SPA bo'lsa: redirect/rewrite (`_redirects` yoki `vercel.json`) qo'shdim — refresh'da 404 yo'q.
- [ ] Environment variable'larni platform dashboard'ida sozladim (to'g'ri prefiks bilan).
- [ ] Maxfiy kalitlarni `NEXT_PUBLIC_`/`VITE_` prefiksisiz qoldirdim (brauzerga chiqmaydi).
- [ ] Deploy'dan keyin saytni real telefon/brauzerda ochib, barcha sahifa va rasm ishlashini tekshirdim.
- [ ] Preview deployment ishlashini (PR ochib) sinab ko'rdim.
- [ ] Bepul tarif limitlarini rasmiy pricing sahifasidan tekshirdim.

---

← [Deployment bo'limiga qaytish](./README.md)
