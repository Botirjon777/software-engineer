# CSS — Yechimlar

Quyida `02-css.md` faylidagi har bir masalaning to'liq yechimi va o'zbekcha izohi berilgan.

---

## 1-masala: Specificity hisoblash

```css
#header .nav a          { }  /* (0, 1, 1, 1) */
.nav .item a:hover      { }  /* (0, 0, 3, 1) */
ul li a                 { }  /* (0, 0, 0, 3) */
```

**Hisoblash:**

- Birinchi: 1 ID, 1 class, 1 tag → `(0,1,1,1)`.
- Ikkinchi: 0 ID, 2 class + 1 pseudo-class = 3, 1 tag → `(0,0,3,1)`.
- Uchinchi: 3 tag → `(0,0,0,3)`.

**G'olib:** birinchi selektor, chunki uning ID si bor. ID darajasi class va tag darajalaridan har doim kuchliroq — ya'ni bitta ID istalgancha class/tag dan ustun.

**Izoh:** Specificity ustun-ustun (leksikografik) taqqoslanadi: avval ID lar, teng bo'lsa class lar, keyin tag lar. `(0,1,0,0)` har qanday `(0,0,99,99)` dan kuchli.

---

## 2-masala: box-sizing hisoblash

```css
.box {
  width: 200px;
  padding: 20px;
  border: 5px solid;
}
```

**content-box (default):**
Real kenglik = `width + 2*padding + 2*border` = `200 + 40 + 10` = **250px**.

**border-box:**
Real kenglik = `width` = **200px** (padding va border ichkariga kiritiladi, content esa `200 - 40 - 10 = 150px` bo'ladi).

```css
.box {
  box-sizing: border-box; /* umumiy o'lcham 200px qoladi */
}
```

**Izoh:** `content-box` da padding/border qo'shiladi va element kattalashadi. `border-box` da belgilangan `width` o'zgarmaydi — bu layout ni bashorat qilishni osonlashtiradi.

---

## 3-masala: Flexbox navbar

```html
<nav class="navbar">
  <div class="logo">Logo</div>
  <ul class="menu">
    <li>Bosh sahifa</li>
    <li>Xizmatlar</li>
    <li>Aloqa</li>
  </ul>
</nav>
```

```css
.navbar {
  display: flex;
  justify-content: space-between; /* logo chapda, menyu o'ngda */
  align-items: center;            /* vertikal markaz */
  padding: 0 24px;
  height: 60px;
}
.menu {
  display: flex;
  gap: 20px;
  list-style: none;
}
```

**Izoh:** `justify-content: space-between` logo va menyuni qarama-qarshi chetlarga suradi. `align-items: center` ularni vertikal markazga keltiradi. Menyu ichidagi elementlar ham flex bilan yonma-yon va `gap` orqali oraliqli joylashadi.

---

## 4-masala: Grid layout (areas)

```html
<div class="layout">
  <header class="header">Header</header>
  <aside class="sidebar">Sidebar</aside>
  <main class="main">Main</main>
  <footer class="footer">Footer</footer>
</div>
```

```css
.layout {
  display: grid;
  grid-template-areas:
    "header  header"
    "sidebar main"
    "footer  footer";
  grid-template-columns: 200px 1fr;
  grid-template-rows: auto 1fr auto;
  gap: 16px;
  min-height: 100vh;
}
.header  { grid-area: header; }
.sidebar { grid-area: sidebar; }
.main    { grid-area: main; }
.footer  { grid-area: footer; }
```

**Izoh:** `grid-template-areas` layout ni vizual qator-qator chizadi. `header` va `footer` ikki ustunni qamrab oladi (nom takrorlangani uchun), `sidebar` 200px, `main` qolgan joyni (`1fr`) egallaydi. Bu usul juda o'qiladigan.

---

## 5-masala: Mobile-first kartochka

```css
/* Bazaviy: telefon (mobile-first) */
.card {
  width: 100%;
}

/* Planshet */
@media (min-width: 768px) {
  .card {
    width: 50%;
  }
}

/* Desktop */
@media (min-width: 1024px) {
  .card {
    width: 33.33%;
  }
}
```

**Izoh:** Mobile-first da bazaviy uslub eng kichik ekran uchun yoziladi (media query siz), keyin `min-width` bilan kattaroq ekranlar uchun qo'shiladi. Bu yondashuv kichik qurilmalarga ortiqcha kod yuklamaydi va performant.

---

## 6-masala: Modalni markazlash (3 usul)

```css
/* 1-usul: Flexbox */
.overlay-flex {
  display: flex;
  justify-content: center;
  align-items: center;
}

/* 2-usul: Grid */
.overlay-grid {
  display: grid;
  place-items: center;
}

/* 3-usul: Absolute + transform */
.overlay-abs {
  position: relative;
}
.overlay-abs .modal {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
}
```

**Izoh:** Flexbox va Grid usullari konteynerga qo'yiladi va bola o'lchamini bilish shart emas. `absolute + transform` esa bolani o'z markazidan teskari siljitadi (`-50%`), shu sababli element o'lchami noma'lum bo'lsa ham aniq markazlashadi.

---

## 7-masala: CSS variables bilan theme switch

```css
:root {
  --bg: #ffffff;
  --text: #111111;
  --primary: #3b82f6;
}

[data-theme="dark"] {
  --bg: #111111;
  --text: #ffffff;
  --primary: #60a5fa;
}

body {
  background: var(--bg);
  color: var(--text);
}
```

```js
const btn = document.getElementById('themeToggle');
btn.addEventListener('click', () => {
  const root = document.documentElement;
  const current = root.getAttribute('data-theme');
  root.setAttribute('data-theme', current === 'dark' ? 'light' : 'dark');
});
```

**Izoh:** O'zgaruvchilar `:root` da default, `[data-theme="dark"]` da qayta belgilanadi. JS faqat `data-theme` atributini almashtiradi — barcha `var()` ishlatgan elementlar avtomatik yangilanadi. Bu CSS variables ning kaskadlanish kuchini ko'rsatadi.

---

## 8-masala: Keyframes animatsiya + transition

```css
@keyframes pulse {
  0%   { transform: scale(1); }
  50%  { transform: scale(1.08); }
  100% { transform: scale(1); }
}

.btn {
  background: #3b82f6;
  color: white;
  padding: 12px 24px;
  border: none;
  border-radius: 8px;
  transition: background 0.3s ease;       /* hover uchun */
  animation: pulse 2s ease-in-out infinite; /* doimiy */
}

.btn:hover {
  background: #2563eb;
}
```

**Izoh:** `@keyframes pulse` mustaqil, takrorlanuvchi (`infinite`) harakat beradi — trigger kerak emas. `transition` esa faqat holat o'zgarganda (`:hover`) ishlaydi va rangni silliq o'zgartiradi. Animatsiya `transform` bilan qilingani uchun reflow chaqirmaydi.

---

## 9-masala: z-index muammosi va yechimi

```html
<div class="parent">
  <div class="child">Men tepada bo'lishim kerak (z-index: 9999)</div>
</div>
<div class="sibling">Lekin men ustidaman</div>
```

```css
/* MUAMMO: parent stacking context yaratadi */
.parent {
  position: relative;
  z-index: 1;
  opacity: 0.99; /* yangi stacking context! */
}
.child {
  position: absolute;
  z-index: 9999; /* faqat parent ICHIDA kuchli */
}
.sibling {
  position: relative;
  z-index: 2; /* parent (1) dan yuqori -> sibling g'olib */
}
```

```css
/* YECHIM: parent ning z-index ini oshirish yoki context ni olib tashlash */
.parent {
  position: relative;
  z-index: 3; /* endi sibling (2) dan yuqori */
  /* opacity: 0.99; ni olib tashlash ham yordam beradi */
}
```

**Izoh:** `.child` ning `z-index: 9999` i faqat o'z ota stacking context i ichida amal qiladi. `.parent` (z-index 1) `.sibling` (z-index 2) dan past bo'lgani uchun butun parent daraxti sibling ostida qoladi. Yechim — parent ning z-index ini oshirish, chunki taqqoslash kontekstlar darajasida boradi.

---

## 10-masala: transform bilan animatsiya (performance)

```css
/* SEKIN: top/left reflow chaqiradi */
.box-slow {
  position: absolute;
  left: 0;
  transition: left 0.3s ease;
}
.box-slow:hover {
  left: 200px;
}

/* TEZ: transform faqat compositing */
.box-fast {
  transition: transform 0.3s ease;
}
.box-fast:hover {
  transform: translateX(200px);
}
```

**Izoh:** `left` ni o'zgartirish har kadrda **reflow (layout)** ni chaqiradi — brauzer barcha elementlar joyini qayta hisoblaydi, bu qimmat va kadr tushishiga olib keladi. `transform: translateX()` esa faqat **compositing** bosqichida ishlaydi (ko'pincha GPU da), reflow va repaint ni o'tkazib yuboradi — natijada animatsiya silliq 60fps da ishlaydi.

---

← [CSS mavzusiga qaytish](../../frontend/02-css.md)
