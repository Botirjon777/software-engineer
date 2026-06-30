# HTML

Bu fayl HTML asoslari, semantika, formalar, accessibility va rendering bo'yicha intervyu tayyorgarligini qamrab oladi.

## Mundarija

- [HTML nima va brauzer uni qanday parse qiladi](#html-nima-va-brauzer-uni-qanday-parse-qiladi)
- [DOM vs HTML](#dom-vs-html)
- [Semantik elementlar](#semantik-elementlar)
- [Hujjat strukturasi](#hujjat-strukturasi)
- [Block vs inline](#block-vs-inline)
- [Formalar](#formalar)
- [Jadvallar](#jadvallar)
- [`<script>`: async vs defer](#script-async-vs-defer)
- [Rasm va srcset](#rasm-va-srcset)
- [Link rel: noopener/noreferrer](#link-rel-noopenernoreferrer)
- [Accessibility](#accessibility)
- [SEO asoslari](#seo-asoslari)
- [data-* atributlar](#data--atributlar)
- [iframe sandbox](#iframe-sandbox)
- [HTML vs XHTML](#html-vs-xhtml)
- [Web storage qisqacha](#web-storage-qisqacha)
- [Critical rendering path](#critical-rendering-path)
- [Intervyu savollari](#intervyu-savollari)
- [Masalalar](#masalalar)

---

## HTML nima va brauzer uni qanday parse qiladi

**💡 Tushuncha:** HTML (HyperText Markup Language) — bu sahifa kontentini *struktura* va *ma'no* bilan ta'minlovchi belgilash (markup) tili. U dasturlash tili emas: mantiq (logic) yo'q, faqat elementlar daraxti.

Brauzer HTML ni quyidagi bosqichlarda parse qiladi:

1. **Bytes → Characters:** baytlar kodlash (masalan UTF-8) bo'yicha belgilarga aylantiriladi.
2. **Tokenization:** belgilar oqimi tokenlarga (`<div>`, `</div>`, matn) bo'linadi.
3. **Lexing → Nodes:** tokenlar node obyektlariga aylanadi.
4. **DOM tree:** node lar ota-bola munosabati bilan daraxtga yig'iladi.

```html
<!DOCTYPE html>
<html>
  <head>
    <title>Salom</title>
  </head>
  <body>
    <h1>Dunyo</h1>
  </body>
</html>
```

**⚠️ Ehtiyot bo'l:** Parser HTML ni yuqoridan pastga *oqimli* (streaming) o'qiydi. Shu sababli `<script>` (defer/async siz) parsing ni to'xtatadi (blocking).

---

## DOM vs HTML

**💡 Tushuncha:** HTML — bu *matn* (manba kod), DOM (Document Object Model) — bu brauzer xotirasidagi *jonli obyektlar daraxti*. JavaScript HTML ni emas, DOM ni o'zgartiradi.

Muhim farqlar:

- HTML — statik fayl; DOM — dinamik, JS bilan o'zgaradigan struktura.
- Brauzer noto'g'ri HTML ni "tuzatadi": yetishmayotgan `<tbody>` ni qo'shadi, yopilmagan teglarni yopadi. Shu sababli DevTools dagi DOM ko'pincha manba HTML dan farq qiladi.
- `document.write` HTML ga emas, DOM ga ta'sir qiladi.

```js
document.querySelector('h1').textContent = 'Yangi sarlavha';
// Manba HTML o'zgarmaydi, faqat DOM yangilanadi.
```

---

## Semantik elementlar

**💡 Tushuncha:** Semantik element — bu o'z *ma'nosini* ifodalovchi teg (`<header>`, `<nav>`, `<main>`, `<article>`, `<section>`, `<aside>`, `<footer>`). `<div>` va `<span>` — ma'nosiz (generic) elementlar.

Nega muhim:

- **Accessibility:** screen reader landmark larni o'qiy oladi (`<nav>`, `<main>`).
- **SEO:** qidiruv tizimlari struktura va muhimlikni tushunadi.
- **O'qilishi:** kod ravshanroq, `<div class="header">` o'rniga `<header>`.

```html
<body>
  <header>Logo va menyu</header>
  <nav>Asosiy navigatsiya</nav>
  <main>
    <article>
      <h1>Maqola sarlavhasi</h1>
      <section>Bo'lim</section>
    </article>
    <aside>Yon panel</aside>
  </main>
  <footer>Mualliflik huquqi</footer>
</body>
```

---

## Hujjat strukturasi

**💡 Tushuncha:** To'g'ri HTML hujjati `doctype`, `<html lang>`, `<head>` (meta-ma'lumot) va `<body>` (ko'rinadigan kontent) dan iborat.

```html
<!DOCTYPE html>
<html lang="uz">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <meta name="description" content="Sahifa tavsifi" />
    <title>Sahifa nomi</title>
  </head>
  <body>...</body>
</html>
```

- **`<!DOCTYPE html>`:** brauzerni *standards mode* da ishlashga majburlaydi (aks holda *quirks mode*).
- **`charset=UTF-8`:** kodlash; `<head>` boshida bo'lishi kerak.
- **`viewport`:** responsive dizayn uchun majburiy — qurilma kengligiga moslaydi.
- **`lang`:** til; accessibility va tarjima uchun.

---

## Block vs inline

**💡 Tushuncha:** Block elementlar (`<div>`, `<p>`, `<h1>`, `<section>`) yangi qatordan boshlanadi va to'liq kenglikni egallaydi. Inline elementlar (`<span>`, `<a>`, `<strong>`, `<img>`) faqat o'z kontenti kengligini oladi va bitta qatorda yonma-yon turadi.

| Xususiyat | Block | Inline |
|---|---|---|
| Yangi qator | Ha | Yo'q |
| width/height | Qabul qiladi | Qabul qilmaydi* |
| margin/padding | To'liq | Faqat gorizontal samarali |

**⚠️ Ehtiyot bo'l:** Inline elementga `width`/`height` ta'sir qilmaydi. Kerak bo'lsa `display: inline-block` ishlat. `<img>` — inline bo'lsa-da replaced element bo'lgani uchun o'lcham qabul qiladi.

---

## Formalar

**💡 Tushuncha:** `<form>` foydalanuvchi kiritgan ma'lumotni yig'adi. Har input `<label>` bilan bog'lanishi kerak (accessibility uchun).

```html
<form action="/submit" method="post">
  <label for="email">Email</label>
  <input type="email" id="email" name="email" required />

  <label for="age">Yosh</label>
  <input type="number" id="age" name="age" min="18" max="99" />

  <button type="submit">Yuborish</button>
</form>
```

Input turlari: `text`, `email`, `password`, `number`, `tel`, `url`, `date`, `checkbox`, `radio`, `file`, `range`, `color`, `search`, `hidden`.

**Validation** (HTML5): `required`, `min`/`max`, `minlength`/`maxlength`, `pattern`, `type="email"`. Brauzer avtomatik tekshiradi.

**FormData** — formani JS bilan yig'ish:

```js
const form = document.querySelector('form');
form.addEventListener('submit', (e) => {
  e.preventDefault();
  const data = new FormData(form);
  console.log(data.get('email'));
  // fetch('/submit', { method: 'POST', body: data });
});
```

**⚠️ Ehtiyot bo'l:** `name` atributi bo'lmagan input form bilan yuborilmaydi va FormData ga tushmaydi.

---

## Jadvallar

**💡 Tushuncha:** Jadvallar *tabular* (qatorli-ustunli) ma'lumot uchun, layout uchun emas. To'g'ri struktura `<thead>`, `<tbody>`, `<tfoot>` dan iborat.

```html
<table>
  <caption>Talabalar ro'yxati</caption>
  <thead>
    <tr>
      <th scope="col">Ism</th>
      <th scope="col">Ball</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>Ali</td>
      <td>90</td>
    </tr>
  </tbody>
</table>
```

- **`<th scope>`:** sarlavha katakchasi; `scope="col"`/`"row"` accessibility uchun.
- **`<caption>`:** jadval nomi.
- **`colspan`/`rowspan`:** kataklarni birlashtirish.

---

## `<script>`: async vs defer

**💡 Tushuncha:** Oddiy `<script>` HTML parsing ni *bloklaydi*. `async` va `defer` buni hal qiladi.

| Atribut | Yuklab olish | Bajarish |
|---|---|---|
| (yo'q) | Parsing to'xtaydi | Darhol, parsing to'xtagan holda |
| `async` | Parallel | Yuklangach darhol (tartib kafolatsiz) |
| `defer` | Parallel | Parsing tugagach, tartibda |

```html
<script src="analytics.js" async></script>
<script src="app.js" defer></script>
```

- **`defer`:** asosiy ilova skriptlari uchun (tartib muhim, DOM tayyor bo'ladi).
- **`async`:** mustaqil skriptlar uchun (analytics, reklama).

**⚠️ Ehtiyot bo'l:** `async`/`defer` faqat tashqi (`src` bilan) skriptlarda ishlaydi, inline skriptda emas.

---

## Rasm va srcset

**💡 Tushuncha:** `srcset` brauzerga turli ekran zichligi/kengligi uchun mos rasmni tanlash imkonini beradi — performance va sifat uchun.

```html
<img
  src="rasm-800.jpg"
  srcset="rasm-400.jpg 400w, rasm-800.jpg 800w, rasm-1200.jpg 1200w"
  sizes="(max-width: 600px) 400px, 800px"
  alt="Tog' manzarasi"
/>
```

- **`srcset` + `w`:** rasm haqiqiy kengligini bildiradi; brauzer `sizes` bilan tanlaydi.
- **`<picture>`:** turli formatlar (WebP/AVIF) yoki art direction uchun:

```html
<picture>
  <source srcset="rasm.avif" type="image/avif" />
  <source srcset="rasm.webp" type="image/webp" />
  <img src="rasm.jpg" alt="Tavsif" />
</picture>
```

**⚠️ Ehtiyot bo'l:** `alt` har doim bo'lishi shart. Dekorativ rasm uchun `alt=""` (bo'sh).

---

## Link rel: noopener/noreferrer

**💡 Tushuncha:** `target="_blank"` bilan yangi tab ochilganda yangi sahifa `window.opener` orqali ota sahifaga kirishi mumkin — bu xavfsizlik teshigi (tabnabbing).

```html
<a href="https://example.com" target="_blank" rel="noopener noreferrer">
  Tashqi havola
</a>
```

- **`noopener`:** `window.opener` ni `null` qiladi — tabnabbing dan himoya.
- **`noreferrer`:** `Referer` sarlavhasini yubormaydi (maxfiylik) + noopener ham bajaradi.

**💡 Tushuncha:** Zamonaviy brauzerlar `target="_blank"` uchun avtomatik `noopener` qo'yadi, lekin aniq yozish xavfsizroq.

---

## Accessibility

**💡 Tushuncha:** Accessibility (a11y) — sahifani nogironligi bo'lgan (screen reader, klaviatura) foydalanuvchilar uchun ishlatishga yaroqli qilish.

Asoslar:

- **Semantik HTML birinchi navbatda:** `<button>` o'rniga `<div onclick>` ishlatma.
- **`alt`:** har rasmda; ma'noni tasvirlaydi.
- **Landmark lar:** `<nav>`, `<main>`, `<header>` screen reader navigatsiyasi uchun.
- **Klaviatura:** barcha interaktiv element `Tab` bilan yetib bo'lishi va `Enter`/`Space` bilan ishlashi kerak.
- **Focus:** ko'rinadigan focus indikatori (`:focus-visible`) bo'lsin; `outline: none` ni e'tiborsiz olib tashlama.

**ARIA** (faqat zarur bo'lganda):

```html
<button aria-label="Yopish" aria-expanded="false">×</button>
<div role="alert">Xatolik yuz berdi</div>
```

**⚠️ Ehtiyot bo'l:** ARIA ni semantik HTML o'rniga ishlatma. "No ARIA is better than bad ARIA." Avval to'g'ri teg, keyin kerak bo'lsa ARIA.

---

## SEO asoslari

**💡 Tushuncha:** SEO (Search Engine Optimization) qidiruv tizimlariga sahifani tushunish va saralashda yordam beradi.

- **`<title>` va `<meta name="description">`:** qidiruv natijalarida ko'rinadi.
- **Heading ierarxiyasi:** sahifada bitta `<h1>`, keyin `<h2>`, `<h3>` tartibda — bosqichni o'tkazib yuborma.
- **Semantik struktura:** `<article>`, `<nav>` muhimlikni ifodalaydi.
- **Open Graph** (ijtimoiy tarmoqlar uchun):

```html
<meta property="og:title" content="Maqola nomi" />
<meta property="og:description" content="Qisqa tavsif" />
<meta property="og:image" content="https://example.com/rasm.jpg" />
<meta property="og:url" content="https://example.com/maqola" />
```

**⚠️ Ehtiyot bo'l:** `<h1>` ni kattalik uchun ishlatma — bu CSS vazifasi. Heading lar ma'no ierarxiyasi uchun.

---

## data-* atributlar

**💡 Tushuncha:** `data-*` atributlari HTML elementga maxsus (custom) ma'lumot biriktirish uchun. JS dan `dataset` orqali o'qiladi.

```html
<div id="card" data-user-id="42" data-role="admin">Ali</div>
```

```js
const el = document.getElementById('card');
console.log(el.dataset.userId); // "42"
console.log(el.dataset.role);   // "admin"
el.dataset.userId = '43';        // o'zgartirish
```

**⚠️ Ehtiyot bo'l:** `data-user-id` → `dataset.userId` (kebab-case camelCase ga aylanadi).

---

## iframe sandbox

**💡 Tushuncha:** `<iframe>` boshqa sahifani ichiga joylaydi. `sandbox` uning ruxsatlarini cheklaydi — xavfsizlik uchun.

```html
<iframe
  src="https://example.com"
  sandbox="allow-scripts allow-same-origin"
  title="Tashqi kontent"
></iframe>
```

- **`sandbox=""`** (bo'sh): hammasi taqiqlangan (script, form, popup yo'q).
- **`allow-scripts`:** JS ga ruxsat.
- **`allow-same-origin`:** o'z origin sifatida ishlash.
- **`allow-forms`:** form yuborish.

**⚠️ Ehtiyot bo'l:** `allow-scripts` va `allow-same-origin` ni birga berish sandbox himoyasini ishonchsiz qiladi (iframe sandbox ni o'chira oladi).

---

## HTML vs XHTML

**💡 Tushuncha:** XHTML — HTML ning XML qoidalariga bo'ysunadigan qattiqroq versiyasi.

| Xususiyat | HTML5 | XHTML |
|---|---|---|
| Teg yopish | Ixtiyoriy (`<br>`) | Majburiy (`<br />`) |
| Katta-kichik harf | Befarq | Faqat kichik |
| Xato | Brauzer tuzatadi | Parsing to'xtaydi |
| Atribut tirnoq | Ixtiyoriy | Majburiy |

Bugun deyarli har doim HTML5 ishlatiladi; XHTML asosan eski tizimlarda qoldi.

---

## Web storage qisqacha

**💡 Tushuncha:** Brauzerda ma'lumot saqlash usullari.

| Usul | Hajm | Yashash muddati | Server ga yuboriladimi |
|---|---|---|---|
| `localStorage` | ~5-10MB | Doimiy | Yo'q |
| `sessionStorage` | ~5MB | Tab yopilguncha | Yo'q |
| `cookie` | ~4KB | Belgilangan | Ha (har so'rovda) |

```js
localStorage.setItem('til', 'uz');
localStorage.getItem('til'); // "uz"
localStorage.removeItem('til');
```

**⚠️ Ehtiyot bo'l:** `localStorage` faqat string saqlaydi — obyekt uchun `JSON.stringify`/`JSON.parse`. Maxfiy ma'lumotni (token) saqlash xavfli (XSS).

---

## Critical rendering path

**💡 Tushuncha:** Critical Rendering Path (CRP) — brauzer HTML/CSS/JS ni ekrandagi piksellarga aylantirish bosqichlari.

1. **DOM** — HTML dan.
2. **CSSOM** — CSS dan.
3. **Render tree** — DOM + CSSOM (faqat ko'rinadigan elementlar).
4. **Layout (reflow)** — har element joyi va o'lchami hisoblanadi.
5. **Paint** — piksellar chiziladi.

**Optimallashtirish:**

- CSS ni `<head>` ga, render-blocking bo'lgani uchun tez yukla.
- JS ga `defer`/`async`.
- Critical CSS ni inline qil, qolganini kechiktir.

**⚠️ Ehtiyot bo'l:** CSS render-blocking: CSSOM tayyor bo'lmaguncha brauzer paint qilmaydi. Shu sababli CSS ni kichik va tez yetkazish muhim.

---

## Intervyu savollari

### ❓ HTML dasturlash tilimi?

**✅ Javob:** Yo'q. HTML — *markup* (belgilash) tili. Unda mantiq, shartlar, sikllar yo'q; u faqat kontent strukturasi va ma'nosini ifodalaydi. Dasturlash tili (JS) bilan birlashtirilganda interaktiv bo'ladi.

### ❓ DOM va HTML o'rtasidagi farq nima?

**✅ Javob:** HTML — bu statik matn (manba kod). DOM — brauzer shu HTML ni o'qib, xotirada yaratgan jonli obyektlar daraxti. JS DOM ni o'zgartiradi, HTML manba o'zgarmaydi. Brauzer noto'g'ri HTML ni tuzatib DOM yaratadi, shuning uchun ikkalasi farq qilishi mumkin.

### ❓ Semantik HTML nima va nega muhim?

**✅ Javob:** Semantik elementlar o'z ma'nosini ifodalaydi (`<header>`, `<nav>`, `<article>`). Ular accessibility (screen reader landmark), SEO (qidiruv tizimi tushunishi) va kod o'qilishi uchun muhim. `<div>` ma'nosiz, semantik teg ma'noli.

### ❓ Block va inline element farqi?

**✅ Javob:** Block element yangi qatordan boshlanadi, to'liq kenglik oladi va `width`/`height` qabul qiladi (`<div>`, `<p>`). Inline element bitta qatorda yonma-yon turadi, faqat kontent kengligini oladi va `width`/`height` qabul qilmaydi (`<span>`, `<a>`). Ikkovi orasi — `inline-block`.

### ❓ `async` va `defer` farqi nima?

**✅ Javob:** Ikkalasi ham skriptni parallel yuklaydi (parsing ni bloklamaydi). `async` yuklangan zahoti, tartibsiz bajaradi — mustaqil skriptlar uchun. `defer` HTML parsing tugagach, yozilgan tartibda bajaradi — asosiy ilova skriptlari uchun. Ikkalasi faqat tashqi (`src`) skriptda ishlaydi.

### ❓ `<label>` ni input bilan bog'lash nega muhim?

**✅ Javob:** `<label for="id">` accessibility uchun: screen reader input maqsadini o'qiydi, label bosilganda input fokuslanadi (klik maydoni kattalashadi). Bog'lash `for`/`id` orqali yoki input ni label ichiga o'rab amalga oshiriladi.

### ❓ `target="_blank"` da `rel="noopener"` nega kerak?

**✅ Javob:** Usiz yangi ochilgan sahifa `window.opener` orqali ota sahifani boshqarishi (boshqa URL ga yo'naltirishi) mumkin — tabnabbing hujumi. `noopener` `opener` ni `null` qilib bu xavfni yo'qotadi. `noreferrer` qo'shimcha Referer sarlavhasini ham yashiradi.

### ❓ `<img>` da `alt` atributi nima uchun?

**✅ Javob:** `alt` rasm yuklanmasa yoki screen reader uchun matnli muqobil beradi. Accessibility va SEO uchun zarur. Dekorativ rasmlar uchun `alt=""` (bo'sh) yoziladi, shunda screen reader uni o'tkazib yuboradi.

### ❓ `data-*` atributlari nima uchun?

**✅ Javob:** Elementga maxsus ma'lumot biriktirish uchun (masalan `data-user-id`). JS dan `element.dataset.userId` orqali o'qiladi. Bu standart bo'lmagan atributlardan farqli ravishda valid va kelajakda kafolatlangan.

### ❓ localStorage va sessionStorage farqi?

**✅ Javob:** `localStorage` ma'lumotni doimiy (brauzer yopilsa ham) saqlaydi. `sessionStorage` faqat tab ochiq bo'lguncha. Ikkalasi ~5MB, server ga yuborilmaydi (cookie dan farqli). Ikkalasi ham faqat string saqlaydi.

### ❓ `<!DOCTYPE html>` nima qiladi?

**✅ Javob:** Brauzerni *standards mode* da ishlashga majburlaydi. Usiz brauzer *quirks mode* ga o'tib, eski/notiniq render qoidalarini qo'llaydi (masalan box model boshqacha hisoblanadi). HTML5 da doctype oddiy: `<!DOCTYPE html>`.

### ❓ Critical rendering path nima?

**✅ Javob:** Brauzer HTML/CSS/JS ni ekran piksellariga aylantirish bosqichlari: DOM → CSSOM → Render tree → Layout → Paint. CSS render-blocking bo'lgani uchun uni tez yetkazish, JS ga `defer` qo'yish sahifa tez ko'rinishini ta'minlaydi.

---

## Masalalar

> Yechimlar: [Yechimlarni ko'rish](../solutions/frontend/01-html.md)

1. Berilgan matnli sahifani to'liq semantik HTML5 struktura (`header`, `nav`, `main`, `article`, `footer`) bilan belgilang.
2. Email, parol va yosh (18+) maydonli formani HTML5 validation bilan yarating; har inputga to'g'ri `<label>` bog'lang.
3. `<thead>`, `<tbody>`, `<caption>` va `scope` atributli to'liq accessible jadval tuzing.
4. Bitta rasmni `srcset` va `sizes` bilan responsive qiling: 400px, 800px, 1200px variantlari.
5. FormData yordamida formani JS bilan yig'ib, `console.log` ga chiqaruvchi kod yozing (default submit ni to'xtatib).
6. Berilgan inline va block elementlar ro'yxatini ikki guruhga ajrating va `display: inline-block` qachon kerakligini misol bilan ko'rsating.
7. Xavfsiz tashqi havola yarating: yangi tabda ochilsin va tabnabbing dan himoyalangan bo'lsin.
8. `data-*` atributlaridan foydalanib, kartochka elementidan foydalanuvchi ID va rolini JS bilan o'qib chiqaruvchi kod yozing.
9. Tashqi kontentni ko'rsatuvchi, faqat skript va form ga ruxsat beruvchi sandboxli `<iframe>` yozing.
10. Sahifaga to'liq SEO va Open Graph meta-teglarini (`title`, `description`, `og:*`) qo'shing va heading ierarxiyasini to'g'rilang.

---

← [Frontend bo'limiga qaytish](./README.md)
