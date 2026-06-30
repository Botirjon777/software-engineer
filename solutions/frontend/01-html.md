# HTML — Yechimlar

Quyida `01-html.md` faylidagi har bir masalaning to'liq yechimi va o'zbekcha izohi berilgan.

---

## 1-masala: Semantik HTML5 struktura

```html
<!DOCTYPE html>
<html lang="uz">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Blog sahifasi</title>
  </head>
  <body>
    <header>
      <h1>Mening blogim</h1>
    </header>

    <nav>
      <a href="/">Bosh sahifa</a>
      <a href="/about">Biz haqimizda</a>
    </nav>

    <main>
      <article>
        <h2>Birinchi maqola</h2>
        <p>Maqola matni shu yerda.</p>
      </article>
    </main>

    <footer>
      <p>© 2026 Mening blogim</p>
    </footer>
  </body>
</html>
```

**Izoh:** `<div>` o'rniga semantik teglar ishlatildi. `<header>` sahifa boshi, `<nav>` navigatsiya, `<main>` asosiy kontent (sahifada bitta bo'lishi kerak), `<article>` mustaqil mazmun, `<footer>` pastki qism. Bu struktura screen reader va SEO uchun foydali.

---

## 2-masala: Validation bilan forma

```html
<form action="/register" method="post">
  <label for="email">Email</label>
  <input type="email" id="email" name="email" required />

  <label for="password">Parol</label>
  <input
    type="password"
    id="password"
    name="password"
    minlength="8"
    required
  />

  <label for="age">Yosh</label>
  <input type="number" id="age" name="age" min="18" max="120" required />

  <button type="submit">Ro'yxatdan o'tish</button>
</form>
```

**Izoh:** `type="email"` brauzer formatini tekshiradi. `required` bo'sh qoldirishga yo'l qo'ymaydi. `minlength="8"` parol uzunligini, `min="18"` esa yoshni cheklaydi. Har `<input>` `for`/`id` orqali `<label>` bilan bog'langan — label bosilganda input fokuslanadi.

---

## 3-masala: Accessible jadval

```html
<table>
  <caption>Sinf ballari</caption>
  <thead>
    <tr>
      <th scope="col">Talaba</th>
      <th scope="col">Fan</th>
      <th scope="col">Ball</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th scope="row">Ali</th>
      <td>Matematika</td>
      <td>95</td>
    </tr>
    <tr>
      <th scope="row">Vali</th>
      <td>Fizika</td>
      <td>88</td>
    </tr>
  </tbody>
</table>
```

**Izoh:** `<caption>` jadval nomini beradi. `<thead>` va `<tbody>` strukturani ajratadi. `scope="col"` ustun sarlavhasi, `scope="row"` qator sarlavhasini bildiradi — screen reader qaysi katak qaysi sarlavhaga tegishli ekanligini to'g'ri o'qiydi.

---

## 4-masala: Responsive rasm (srcset)

```html
<img
  src="banner-800.jpg"
  srcset="
    banner-400.jpg   400w,
    banner-800.jpg   800w,
    banner-1200.jpg 1200w
  "
  sizes="(max-width: 600px) 400px,
         (max-width: 1000px) 800px,
         1200px"
  alt="Sayt bannери"
/>
```

**Izoh:** `srcset` har rasmning haqiqiy kengligini (`w`) e'lon qiladi. `sizes` esa rasm ekrandagi qaysi kenglikni egallashini media shartlar orqali bildiradi. Brauzer ekran o'lchami va piksel zichligiga qarab eng mos faylni o'zi tanlaydi — bu trafikni tejaydi.

---

## 5-masala: FormData bilan formani yig'ish

```html
<form id="loginForm">
  <input type="email" name="email" />
  <input type="password" name="password" />
  <button type="submit">Kirish</button>
</form>

<script>
  const form = document.getElementById('loginForm');
  form.addEventListener('submit', (e) => {
    e.preventDefault(); // default reload ni to'xtatadi
    const data = new FormData(form);
    const obj = Object.fromEntries(data.entries());
    console.log(obj); // { email: "...", password: "..." }
  });
</script>
```

**Izoh:** `e.preventDefault()` brauzer formani server ga yuborib sahifani qayta yuklashini to'xtatadi. `new FormData(form)` `name` atributli barcha inputlarni yig'adi. `Object.fromEntries` ni qulay obyektga aylantiradi. `name` bo'lmagan inputlar bu yerga tushmaydi.

---

## 6-masala: Inline/block ajratish

**Block elementlar:** `<div>`, `<p>`, `<h1>`, `<section>`, `<ul>`, `<form>`
**Inline elementlar:** `<span>`, `<a>`, `<strong>`, `<em>`, `<img>`, `<label>`

```html
<!-- Muammo: inline <a> ga balandlik kerak, lekin inline qabul qilmaydi -->
<a href="#" class="btn">Tugma</a>

<style>
  .btn {
    display: inline-block; /* endi width/height/padding ishlaydi */
    width: 120px;
    height: 40px;
    text-align: center;
    line-height: 40px;
  }
</style>
```

**Izoh:** `inline-block` element bitta qatorda qoladi (inline kabi), lekin `width`/`height`/vertical `padding` qabul qiladi (block kabi). Tugma yoki menyu elementlarini yonma-yon joylab, ularga o'lcham berish kerak bo'lganda ishlatiladi.

---

## 7-masala: Xavfsiz tashqi havola

```html
<a
  href="https://tashqi-sayt.com"
  target="_blank"
  rel="noopener noreferrer"
>
  Tashqi saytga o'tish
</a>
```

**Izoh:** `target="_blank"` yangi tabda ochadi. `noopener` yangi sahifaning `window.opener` orqali ota sahifani boshqarishiga (tabnabbing) yo'l qo'ymaydi. `noreferrer` qaysi saytdan kelganini (Referer) yashiradi va o'zi ham noopener vazifasini bajaradi.

---

## 8-masala: data-* atributlarini o'qish

```html
<div id="userCard" data-user-id="42" data-user-role="admin">
  Ali Valiyev
</div>

<script>
  const card = document.getElementById('userCard');
  const id = card.dataset.userId;     // "42"
  const role = card.dataset.userRole; // "admin"
  console.log(`ID: ${id}, Rol: ${role}`);
</script>
```

**Izoh:** `data-user-id` HTML da kebab-case yoziladi, JS da esa `dataset.userId` (camelCase) bo'lib o'qiladi. `dataset` qiymatlar har doim string bo'ladi — son kerak bo'lsa `Number(card.dataset.userId)` qil.

---

## 9-masala: Sandboxli iframe

```html
<iframe
  src="https://tashqi-kontent.com"
  sandbox="allow-scripts allow-forms"
  title="Tashqi kontent"
  width="600"
  height="400"
></iframe>
```

**Izoh:** `sandbox` belgilanganda hammasi taqiqlanadi, faqat ro'yxatdagi ruxsatlar yoqiladi. Bu yerda `allow-scripts` JS ga, `allow-forms` form yuborishga ruxsat beradi. `allow-same-origin` ataylab berilmadi — `allow-scripts` bilan birga berilsa sandbox himoyasi ishonchsiz bo'ladi.

---

## 10-masala: SEO va Open Graph

```html
<head>
  <meta charset="UTF-8" />
  <title>Frontend bo'yicha qo'llanma — 2026</title>
  <meta
    name="description"
    content="Frontend intervyu uchun HTML, CSS va JS qo'llanmasi."
  />

  <!-- Open Graph -->
  <meta property="og:title" content="Frontend bo'yicha qo'llanma" />
  <meta
    property="og:description"
    content="HTML, CSS va JS bo'yicha to'liq tayyorgarlik."
  />
  <meta property="og:image" content="https://example.com/preview.jpg" />
  <meta property="og:url" content="https://example.com/qollanma" />
  <meta property="og:type" content="article" />
</head>

<body>
  <h1>Frontend qo'llanma</h1>
  <h2>HTML</h2>
  <h3>Semantika</h3>
  <h2>CSS</h2>
</body>
```

**Izoh:** `<title>` va `description` qidiruv natijalarida ko'rinadi. `og:*` teglar ijtimoiy tarmoqlarda havola ulashilganda ko'rinadigan kartochkani belgilaydi. Heading ierarxiyasi to'g'ri: bitta `<h1>`, keyin `<h2>` va `<h3>` bosqichma-bosqich, hech bir daraja o'tkazib yuborilmagan.

---

← [HTML mavzusiga qaytish](../../frontend/01-html.md)
