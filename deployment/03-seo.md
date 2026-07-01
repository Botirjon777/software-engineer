# SEO'ni To'g'rilash

Saytingiz internetda chiqqanidan keyin uni odamlar topishi kerak. **SEO (Search Engine Optimization)** — saytingizni Google kabi qidiruv tizimlarida yuqoriroq chiqadigan qilib optimallashtirish. Bu qo'llanmada SEO'ning asoslari, on-page optimizatsiya, sitemap/robots, structured data, Core Web Vitals va Next.js'da SEO'ni qadama-qadam o'rganamiz (2026-yil holati).

## Mundarija

- [SEO nima va nega muhim](#seo-nima-va-nega-muhim)
- [Qidiruv tizimlari qanday ishlaydi](#qidiruv-tizimlari-qanday-ishlaydi)
- [On-page SEO asoslari](#on-page-seo-asoslari)
- [Open Graph va Twitter Card](#open-graph-va-twitter-card)
- [sitemap.xml va robots.txt](#sitemapxml-va-robotstxt)
- [Structured data / JSON-LD](#structured-data--json-ld)
- [Texnik SEO: tezlik va Core Web Vitals](#texnik-seo-tezlik-va-core-web-vitals)
- [SPA/CSR muammosi va yechimi](#spacsr-muammosi-va-yechimi)
- [Next.js'da SEO](#nextjsda-seo)
- [Google Search Console va tekshirish vositalari](#google-search-console-va-tekshirish-vositalari)
- [Amaliy: bir sahifani SEO uchun to'g'rilash](#amaliy-bir-sahifani-seo-uchun-tografrilash)
- [Amaliy Q&A (Troubleshooting)](#amaliy-qa-troubleshooting)
- [Amaliy Checklist](#amaliy-checklist)

---

## SEO nima va nega muhim

**SEO (Search Engine Optimization)** — saytingizni qidiruv tizimlari (Google, Bing, Yandex) uchun tushunarli va jozibador qilib, qidiruv natijalarida yuqoriroq o'ringa chiqarish jarayoni.

**💡 Tushuncha:** Ko'pchilik foydalanuvchilar Google natijalarining faqat **birinchi sahifasini** ko'radi. Agar saytingiz 2-3-sahifada bo'lsa, deyarli hech kim topmaydi. Yaxshi SEO = ko'proq bepul (organik) tashrif = ko'proq mijoz/o'quvchi.

SEO'ning asosiy afzalligi — u **bepul va uzoq muddatli**. Reklama pulingiz tugasa to'xtaydi, lekin yaxshi SEO oylar davomida trafik keltiraveradi.

---

## Qidiruv tizimlari qanday ishlaydi

Google saytingizni natijaga chiqarishdan oldin uch bosqichdan o'tkazadi.

**💡 Tushuncha:** Uchta bosqich: **Crawl → Index → Rank**.

1. **Crawl (skanerlash):** Googlebot degan robot internetni kezib, sahifalarni topadi va o'qiydi. U linklar orqali sahifadan sahifaga o'tadi.
2. **Index (indekslash):** O'qigan sahifalarni Google o'zining ulkan bazasiga (indeks) saqlaydi. Indekslanmagan sahifa qidiruvda **umuman chiqmaydi**.
3. **Rank (saralash):** Kimdir qidiruv so'rovi kiritganda, Google indeksdagi eng mos sahifalarni **tartiblab** ko'rsatadi. Bu yerda kontent sifati, tezlik, mobil moslik, linklar va boshqa yuzlab omillar hisobga olinadi.

**⚠️ Ehtiyot bo'l:** Agar sahifangiz JavaScript orqali render bo'lsa (SPA), Googlebot ba'zan kontentni to'liq ko'rmasligi mumkin — bu indekslashda muammo yaratadi. Bu haqda pastda "SPA muammosi" bo'limida.

---

## On-page SEO asoslari

On-page SEO — bu sahifangiz ichidagi HTML'ni to'g'ri yozish. Bu SEO'ning eng asosiy va nazorat qilinadigan qismi.

### Title tag

Har sahifaning noyob `<title>` bo'lishi kerak. Bu qidiruv natijasida ko'k rangda ko'rinadigan sarlavha.

```html
<title>React Deploy Qo'llanmasi | MySite</title>
```

**⚠️ Ehtiyot bo'l:** Title 50-60 belgidan oshmasin — Google uzunini kesib tashlaydi. Har sahifada boshqa (noyob) title yozing.

### Meta description

Qidiruv natijasida title ostidagi tavsif matni. Bevosita rank'ga ta'sir qilmaydi, lekin bosilish (CTR) ni oshiradi.

```html
<meta name="description" content="React (Vite) loyihasini Vercel va Netlify'ga qadama-qadam deploy qilish. SPA routing muammosi va yechimi.">
```

Ideal uzunlik: 120-160 belgi.

### Heading ierarxiyasi (h1 bitta)

**💡 Tushuncha:** Har sahifada **faqat bitta `<h1>`** bo'lishi kerak — bu sahifaning asosiy mavzusi. Qolgan sarlavhalar `<h2>`, `<h3>` bo'lib, ierarxiya buzilmasligi kerak (h1'dan keyin h3'ga sakramang).

```html
<h1>Frontend Deploy Qo'llanmasi</h1>
  <h2>React loyihasini deploy qilish</h2>
    <h3>Vite bilan build</h3>
  <h2>Next.js deploy</h2>
```

### Semantic HTML

`<div>` o'rniga ma'noli teglardan foydalaning: `<header>`, `<nav>`, `<main>`, `<article>`, `<section>`, `<footer>`. Bu qidiruv tizimlari va ekran o'quvchilarga (screen reader) sahifa tuzilishini tushunishga yordam beradi.

### Alt text

Har rasmga `alt` atributi bering — rasm nimani ko'rsatishini tavsiflang.

```html
<img src="/vercel-deploy.png" alt="Vercel dashboard'da Next.js loyihani deploy qilish oynasi">
```

**⚠️ Ehtiyot bo'l:** Bo'sh yoki "image1.png" kabi ma'nosiz alt yozmang. Bu ham SEO, ham qulaylik (accessibility) uchun zarur.

### Canonical URL

Agar bir kontent bir nechta URL'da ochilsa (masalan `?utm=...` bilan), Google'ga "asosiy" URL qaysi ekanini ayting:

```html
<link rel="canonical" href="https://mysite.com/react-deploy">
```

Bu duplikat kontent muammosini oldini oladi.

### Meta robots

Sahifani indekslashni boshqarish:

```html
<!-- Indekslama va linklarga ergashma -->
<meta name="robots" content="noindex, nofollow">
```

**⚠️ Ehtiyot bo'l:** Ishlab chiqishda qo'ygan `noindex` ni production'da olib tashlashni **unutmang** — aks holda saytingiz Google'da umuman chiqmaydi. Bu eng ko'p uchraydigan halokatli xato.

---

## Open Graph va Twitter Card

**💡 Tushuncha:** Open Graph (OG) teglari — havolangiz Telegram, Facebook, LinkedIn kabi ijtimoiy tarmoqlarda ulashilganda ko'rinadigan **kartochka** (rasm, sarlavha, tavsif) ni belgilaydi. Bularsiz link quruq va jozibasiz ko'rinadi.

```html
<!-- Open Graph -->
<meta property="og:title" content="React Deploy Qo'llanmasi">
<meta property="og:description" content="React loyihasini bepul deploy qilish bo'yicha to'liq qo'llanma.">
<meta property="og:image" content="https://mysite.com/og-image.png">
<meta property="og:url" content="https://mysite.com/react-deploy">
<meta property="og:type" content="article">

<!-- Twitter Card -->
<meta name="twitter:card" content="summary_large_image">
<meta name="twitter:title" content="React Deploy Qo'llanmasi">
<meta name="twitter:description" content="React loyihasini bepul deploy qilish bo'yicha to'liq qo'llanma.">
<meta name="twitter:image" content="https://mysite.com/og-image.png">
```

**⚠️ Ehtiyot bo'l:** `og:image` **to'liq (absolute) URL** bo'lishi shart (`https://...`), nisbiy yo'l ishlamaydi. Tavsiya etilgan o'lcham: 1200×630 px.

---

## sitemap.xml va robots.txt

Bu ikki fayl saytning ildizida (`https://mysite.com/robots.txt`) turadi va qidiruv robotlariga yo'l-yo'riq beradi.

### robots.txt

**💡 Tushuncha:** `robots.txt` — robotlarga qaysi sahifalarni skanerlash mumkin/mumkin emasligini va sitemap qayerda ekanini aytadi.

```
User-agent: *
Allow: /
Disallow: /admin/

Sitemap: https://mysite.com/sitemap.xml
```

### sitemap.xml

**💡 Tushuncha:** `sitemap.xml` — saytingizdagi barcha muhim sahifalar ro'yxati. Bu Google'ga sahifalarni tezroq va to'liq topishga yordam beradi.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
  <url>
    <loc>https://mysite.com/</loc>
    <lastmod>2026-06-30</lastmod>
  </url>
  <url>
    <loc>https://mysite.com/react-deploy</loc>
    <lastmod>2026-06-30</lastmod>
  </url>
</urlset>
```

Static saytda bu fayllarni `public/` papkaga qo'lda qo'ying. Next.js'da avtomatlashtirsa bo'ladi (pastda).

---

## Structured data / JSON-LD

**💡 Tushuncha:** Structured data — sahifa mazmunini qidiruv tizimlariga **mashina tushunadigan** formatda tavsiflash. Buning eng tavsiya etilgan formati — **JSON-LD** (schema.org lug'atidan foydalanadi). Natijada Google "rich result" (yulduzchali reyting, FAQ, retsept, narx) ko'rsatishi mumkin — bu bosilishni oshiradi.

```html
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "Article",
  "headline": "React Deploy Qo'llanmasi",
  "author": {
    "@type": "Person",
    "name": "Ramziddin"
  },
  "datePublished": "2026-06-30",
  "image": "https://mysite.com/og-image.png"
}
</script>
```

Boshqa foydali `@type`'lar: `Product`, `FAQPage`, `BreadcrumbList`, `Organization`, `WebSite`.

**⚠️ Ehtiyot bo'l:** Structured data'dagi ma'lumot sahifadagi haqiqiy kontentga mos kelishi shart. Yolg'on/spam markup uchun Google jazo (penalty) beradi.

---

## Texnik SEO: tezlik va Core Web Vitals

Google saytning **tezligi va foydalanuvchi tajribasini** rank omili sifatida hisobga oladi. Buni **Core Web Vitals** o'lchaydi.

**💡 Tushuncha:** Uchta asosiy ko'rsatkich (2026 holati):

- **LCP (Largest Contentful Paint):** eng katta element (odatda bosh rasm/matn) qancha vaqtda yuklanadi. Yaxshi: **< 2.5 s**.
- **INP (Interaction to Next Paint):** foydalanuvchi bosганда sayt qancha tez javob beradi. (Bu FID'ni almashtirgan.) Yaxshi: **< 200 ms**.
- **CLS (Cumulative Layout Shift):** sahifa yuklanayotganda elementlar "sakrab" o'rin almashadimi. Yaxshi: **< 0.1**.

### Tezlikni yaxshilash usullari

1. Rasmlarni optimallashtirish: zamonaviy formatlar (WebP/AVIF), to'g'ri o'lcham, lazy loading.
2. `<img>` ga `width` va `height` berish (CLS'ni kamaytiradi).
3. Foydalanilmagan JS/CSS'ni olib tashlash, kodni bo'lish (code splitting).
4. CDN'dan foydalanish (Vercel/Netlify/Cloudflare avtomatik beradi).
5. Font'larni oldindan yuklash (`preload`), `font-display: swap`.

### Boshqa texnik omillar

- **Mobile-friendly:** sayt telefonda yaxshi ko'rinishi shart (responsive design). Google mobil versiyani asosiy deb hisoblaydi (mobile-first indexing).
- **HTTPS:** SSL sertifikat majburiy. Vercel/Netlify/Cloudflare buni bepul avtomatik beradi.

---

## SPA/CSR muammosi va yechimi

**💡 Tushuncha:** SPA (React Vite kabi) — CSR (Client-Side Rendering) qiladi: server bo'sh `index.html` beradi, kontent brauzerda JavaScript orqali yasaladi. Muammo: qidiruv robotlari (ayniqsa Google'dan boshqalari — Bing, ijtimoiy tarmoq botlari) JS'ni to'liq ijro etmasligi mumkin va **bo'sh sahifa** ko'radi. Natijada SEO zaif bo'ladi.

### Yechimlar

1. **SSR (Server-Side Rendering):** server tayyor HTML beradi. Bot darhol to'liq kontentni ko'radi. Next.js buni oson qiladi.
2. **SSG (Static Site Generation):** sahifalar build vaqtida HTML sifatida yasaladi. Eng tez va SEO uchun eng yaxshi.
3. **Prerendering:** mavjud SPA'ni o'zgartirmasdan, botlar uchun oldindan render qilingan HTML berish (masalan prerender xizmatlari yoki static prerender).

**⚠️ Ehtiyot bo'l:** Agar loyihangiz kontent/marketing sayti bo'lsa va SEO muhim bo'lsa — sof CSR SPA o'rniga **Next.js (SSR/SSG)** ni tanlang. Admin panel yoki login orqasidagi ilovalar uchun SEO muhim emas, u yerda SPA yetarli.

---

## Next.js'da SEO

Next.js SEO uchun eng qulay tanlov, chunki u SSR/SSG'ni tabiiy qo'llaydi va maxsus **Metadata API** beradi (App Router).

### Statik metadata

```js
// app/react-deploy/page.js
export const metadata = {
  title: 'React Deploy Qo\'llanmasi | MySite',
  description: 'React loyihasini bepul deploy qilish bo\'yicha to\'liq qo\'llanma.',
  openGraph: {
    title: 'React Deploy Qo\'llanmasi',
    images: ['https://mysite.com/og-image.png'],
  },
}

export default function Page() {
  return <h1>React Deploy Qo'llanmasi</h1>
}
```

### Dinamik metadata (generateMetadata)

Ma'lumotga qarab (masalan blog post) metadata yasash:

```js
export async function generateMetadata({ params }) {
  const post = await getPost(params.slug)
  return {
    title: post.title,
    description: post.excerpt,
    alternates: { canonical: `https://mysite.com/blog/${params.slug}` },
  }
}
```

### Sitemap va robots (Next.js avtomatik)

`app/` papkada `sitemap.js` va `robots.js` yaratsangiz, Next.js avtomatik `/sitemap.xml` va `/robots.txt` yasaydi:

```js
// app/sitemap.js
export default function sitemap() {
  return [
    { url: 'https://mysite.com', lastModified: new Date() },
    { url: 'https://mysite.com/react-deploy', lastModified: new Date() },
  ]
}
```

```js
// app/robots.js
export default function robots() {
  return {
    rules: { userAgent: '*', allow: '/', disallow: '/admin/' },
    sitemap: 'https://mysite.com/sitemap.xml',
  }
}
```

**💡 Tushuncha:** Ko'p sahifali loyihada `next-sitemap` paketi build'dan keyin sitemap'ni avtomatik generatsiya qiladi — qo'lda yozishdan qulayroq.

---

## Google Search Console va tekshirish vositalari

**💡 Tushuncha:** Google Search Console (GSC) — Google'ning bepul vositasi. U orqali saytingiz indekslanganini, qaysi so'rovlar bilan chiqayotganini va xatolarni ko'rasiz.

### Search Console'ni sozlash

1. [search.google.com/search-console](https://search.google.com/search-console) ga kiring.
2. Saytni qo'shing (domain yoki URL prefix).
3. Egaligini tasdiqlang (DNS TXT record yoki HTML meta tag).
4. **Sitemaps** bo'limiga `sitemap.xml` manzilini yuboring.
5. **URL Inspection** bilan alohida sahifalar indekslanganini tekshiring va "Request Indexing" bosing.

### Foydali tekshirish vositalari

- **Google Rich Results Test** — structured data (JSON-LD) to'g'ri ekanini tekshiradi.
- **PageSpeed Insights / Lighthouse** — tezlik va Core Web Vitals'ni o'lchaydi (Chrome DevTools ichida ham bor).
- **Mobile-Friendly ko'rinish** — DevTools'da device toolbar bilan tekshiring.
- **Ijtimoiy tarmoq debuggerlari** — OG kartochka qanday ko'rinishini tekshirish uchun.

---

## Amaliy: bir sahifani SEO uchun to'g'rilash

Faraz qilaylik, `react-deploy.html` sahifangiz bor va SEO'siz. Uni to'g'rilaymiz.

1. **`<head>` ga asosiy meta teglar qo'shing:**

```html
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>React Deploy Qo'llanmasi | MySite</title>
  <meta name="description" content="React (Vite) loyihasini Vercel va Netlify'ga deploy qilish bo'yicha qadama-qadam qo'llanma.">
  <link rel="canonical" href="https://mysite.com/react-deploy">
</head>
```

2. **Open Graph teglarini qo'shing** (yuqoridagi OG blokini `<head>` ga).

3. **Heading ierarxiyasini to'g'rilang:** bitta `<h1>`, qolgani `<h2>/<h3>`.

4. **Barcha rasmga `alt` va `width/height` bering.**

5. **JSON-LD structured data qo'shing** (`Article` tipida).

6. **`robots.txt` va `sitemap.xml` ni `public/` ga joylang.**

7. **Lighthouse ishga tushiring** (F12 → Lighthouse) va SEO/Performance ballarini ko'ring, tavsiyalarni bajaring.

8. **Search Console'ga sahifani yuboring** va "Request Indexing" bosing.

---

## Amaliy Q&A (Troubleshooting)

### ❓ Saytim tayyor, lekin Google'da umuman chiqmayapti

**✅ Javob:** (1) Sabr qiling — yangi sayt indekslanishi kunlar/haftalar oladi. (2) Search Console'da saytni ro'yxatdan o'tkazib, sitemap yuboring va "Request Indexing" bosing. (3) `<meta name="robots" content="noindex">` yoki `robots.txt` da `Disallow: /` yozib qo'ymaganingizni tekshiring — bu eng ko'p uchraydigan sabab.

### ❓ Sahifam indekslangan, lekin juda past o'rinda

**✅ Javob:** Ranking ko'p omilga bog'liq: kontent sifati va hajmi, kalit so'zlarga moslik, sayt tezligi, boshqa saytlardan kelgan linklar (backlink), sayt yoshi. Bir kechada o'zgarmaydi. Sifatli, foydali kontent yozing va texnik SEO'ni to'g'rilang.

### ❓ React (SPA) saytimni Google indekslamayapti / bo'sh ko'ryapti

**✅ Javob:** CSR SPA'da bot JS render'ini to'liq ko'rmasligi mumkin. Search Console'ning "URL Inspection → View Crawled Page" bilan Google nima ko'rayotganini tekshiring. Yechim: SEO muhim bo'lsa Next.js (SSR/SSG) ga o'ting yoki prerendering ishlating.

### ❓ Havolamni Telegram/Facebook'ga tashlasam rasm/tavsif chiqmayapti

**✅ Javob:** Open Graph teglari yo'q yoki noto'g'ri. Tekshiring: (1) `og:title`, `og:description`, `og:image` bormi. (2) `og:image` to'liq `https://` URL'mi (nisbiy emas). (3) Ijtimoiy tarmoqlar keshni saqlaydi — platformaning URL debugger'i orqali keshni yangilang.

### ❓ Har sahifada bir xil title chiqyapti

**✅ Javob:** SPA yoki shablon (template) da title statik qolib ketgan. Har sahifada `<title>` ni dinamik o'zgartiring. React'da `react-helmet` yoki Next.js'da `metadata`/`generateMetadata` ishlating.

### ❓ Lighthouse SEO balli past chiqyapti

**✅ Javob:** Lighthouse aniq muammolarni ro'yxatlaydi: yo'q meta description, `alt`siz rasmlar, kichik shrift, `noindex`, kanonik URL yo'qligi. Ro'yxatni yuqoridan pastga bajaring — har biri aniq nimani tuzatishni ko'rsatadi.

### ❓ CLS balim yomon (sahifa sakrab yuklanyapti)

**✅ Javob:** Asosiy sabab: rasm/video/reklama uchun joy oldindan belgilanmagan. Har `<img>` ga `width` va `height` bering, font'lar uchun `font-display: swap` va joy ajrating, dinamik kontentni mavjud kontent ustiga emas, ostiga joylang.

### ❓ sitemap.xml yozdim, lekin Search Console "couldn't fetch" deyapti

**✅ Javob:** (1) Fayl haqiqatan `https://mysite.com/sitemap.xml` da ochiladimi — brauzerda ochib ko'ring. (2) XML sintaksisi to'g'rimi (tashqi validator bilan tekshiring). (3) `robots.txt` da sitemap manzili to'g'ri ko'rsatilganmi. (4) Sarlavha (Content-Type) `application/xml` bo'lsa yaxshi.

### ❓ HTTPS yo'q, "not secure" chiqyapti — bu SEO'ga zararmi?

**✅ Javob:** Ha, HTTPS rank omili va foydalanuvchi ishonchini pasaytiradi. Vercel/Netlify/Cloudflare SSL'ni bepul avtomatik beradi. Custom domain ulaganingizdan keyin sertifikat faollashishini kuting (bir necha daqiqa/soat).

### ❓ Structured data yozdim, lekin "rich result" chiqmayapti

**✅ Javob:** (1) Google Rich Results Test bilan xatolarni tekshiring. (2) Barcha majburiy maydonlar (`@type` uchun kerakli property'lar) to'ldirilganmi. (3) Rich result kafolatlanmagan — Google o'zi qaror qiladi, indekslanishdan keyin vaqt kerak.

### ❓ Bir xil kontent bir nechta URL'da ochilyapti (duplikat)

**✅ Javob:** Har sahifada `<link rel="canonical" href="asosiy-url">` qo'ying. Bu Google'ga qaysi versiya "asl" ekanini aytadi va reyting bo'linib ketishini oldini oladi. Trailing slash va `www` masalasida ham izchil bo'ling.

### ❓ Mobil versiyada SEO past — nima qilay?

**✅ Javob:** Google mobile-first indekslaydi. Sayt responsive bo'lishi, matn o'qilishi (juda kichik bo'lmasligi), tugmalar bosilishi qulay bo'lishi va gorizontal scroll bo'lmasligi kerak. DevTools device rejimida va PageSpeed Insights'ning mobil balida tekshiring.

---

## Amaliy Checklist

- [ ] Har sahifada noyob `<title>` (50-60 belgi) bor.
- [ ] Har sahifada `<meta name="description">` (120-160 belgi) bor.
- [ ] Har sahifada faqat bitta `<h1>`, ierarxiya to'g'ri (h2/h3).
- [ ] Semantic HTML ishlatilgan (`header`, `nav`, `main`, `article`, `footer`).
- [ ] Barcha rasmda ma'noli `alt` va `width/height` bor.
- [ ] `<link rel="canonical">` qo'yilgan.
- [ ] Production'da `noindex` YO'Q (xatoan qolib ketmagan).
- [ ] Open Graph va Twitter Card teglari bor (`og:image` — to'liq URL).
- [ ] `robots.txt` va `sitemap.xml` saytning ildizida ochiladi.
- [ ] JSON-LD structured data qo'shilgan va Rich Results Test'dan o'tgan.
- [ ] Lighthouse'da Performance/SEO balli tekshirilgan, LCP < 2.5s, CLS < 0.1.
- [ ] Sayt HTTPS'da va mobil-friendly.
- [ ] SEO muhim SPA bo'lsa — SSR/SSG (Next.js) ga o'tilgan.
- [ ] Google Search Console'ga sayt qo'shilgan, sitemap yuborilgan.

---

← [Deployment bo'limiga qaytish](./README.md)
