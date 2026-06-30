# CSS

Bu fayl CSS asoslari: cascade, box model, layout (Flexbox/Grid), positioning, birliklar va performance bo'yicha intervyu tayyorgarligini qamrab oladi.

## Mundarija

- [Cascade va specificity](#cascade-va-specificity)
- [Box model](#box-model)
- [Display turlari](#display-turlari)
- [Positioning](#positioning)
- [Flexbox](#flexbox)
- [CSS Grid](#css-grid)
- [Birliklar](#birliklar)
- [Responsive va mobile-first](#responsive-va-mobile-first)
- [em vs rem](#em-vs-rem)
- [z-index va stacking context](#z-index-va-stacking-context)
- [Pseudo-class vs pseudo-element](#pseudo-class-vs-pseudo-element)
- [CSS variables](#css-variables)
- [Transition, animation, keyframes](#transition-animation-keyframes)
- [Transform](#transform)
- [Selektorlar](#selektorlar)
- [Margin collapsing](#margin-collapsing)
- [Overflow](#overflow)
- [Centerlash usullari](#centerlash-usullari)
- [BEM](#bem)
- [CSS Modules vs CSS-in-JS vs Tailwind](#css-modules-vs-css-in-js-vs-tailwind)
- [Reflow vs repaint](#reflow-vs-repaint)
- [Intervyu savollari](#intervyu-savollari)
- [Masalalar](#masalalar)

---

## Cascade va specificity

**💡 Tushuncha:** Cascade — bir elementga bir nechta qoida tegsa, qaysi biri g'olib bo'lishini hal qilish tartibi. Tartib: (1) muhimlik (`!important`), (2) specificity, (3) manba tartibi (oxirgisi g'olib).

**Specificity hisoblash** — `(a, b, c)` ko'rinishida:

- `a` — inline style (`style="..."`).
- `b` — ID lar soni (`#id`).
- `c` — class, attribute, pseudo-class lar soni (`.class`, `[type]`, `:hover`).
- tag va pseudo-element lar eng past darajada.

Misol:

```css
#nav .item a:hover { color: red; }   /* (0,1,2,1) -> 1 ID, 2 class/pseudo, 1 tag */
.item a              { color: blue; } /* (0,0,1,1) */
```

Bu yerda birinchi qoida g'olib, chunki uning ID si bor.

**⚠️ Ehtiyot bo'l:** `!important` specificity ni chetlab o'tadi, lekin uni ortiqcha ishlatish kodni boshqarib bo'lmas qiladi. Iloji boricha specificity bilan hal qil.

---

## Box model

**💡 Tushuncha:** Har element to'rt qatlamdan iborat to'rtburchak: `content` → `padding` → `border` → `margin`.

```css
.box {
  width: 200px;
  padding: 20px;
  border: 5px solid;
  /* content-box: umumiy kenglik = 200 + 40 + 10 = 250px */
}
```

**`box-sizing`:**

- `content-box` (default): `width` faqat content ga tegishli, padding/border qo'shiladi.
- `border-box`: `width` padding va border ni *o'z ichiga oladi* — hisoblash osonroq.

```css
* {
  box-sizing: border-box;
}
.box {
  width: 200px; /* padding/border bilan birga 200px qoladi */
}
```

**⚠️ Ehtiyot bo'l:** Loyihada deyarli har doim `box-sizing: border-box` ni global qo'yish tavsiya etiladi — bu o'lcham hisobini ancha soddalashtiradi.

---

## Display turlari

**💡 Tushuncha:** `display` element qanday joylashishini belgilaydi.

- `block` — to'liq kenglik, yangi qator.
- `inline` — kontent kengligi, width/height yo'q.
- `inline-block` — inline joylashuv + block o'lchamlari.
- `none` — DOM da bor, lekin ko'rinmaydi va joy egallamaydi.
- `flex` — flex konteyner.
- `grid` — grid konteyner.

**⚠️ Ehtiyot bo'l:** `display: none` joy egallamaydi; `visibility: hidden` ko'rinmaydi lekin joy egallaydi; `opacity: 0` ko'rinmaydi lekin bosiladi.

---

## Positioning

**💡 Tushuncha:** `position` elementni normal oqimdan chiqarib joylashtiradi.

| Qiymat | Tavsif |
|---|---|
| `static` | Default; top/left ishlamaydi |
| `relative` | O'z joyiga nisbatan siljiydi, joyi saqlanadi |
| `absolute` | Eng yaqin `position`li ajdodga nisbatan, oqimdan chiqadi |
| `fixed` | Viewport ga nisbatan, skroll bilan harakatlanmaydi |
| `sticky` | Threshold gacha relative, keyin fixed kabi yopishadi |

```css
.parent { position: relative; }
.child {
  position: absolute;
  top: 0;
  right: 0; /* parent ning yuqori-o'ng burchagi */
}
.header {
  position: sticky;
  top: 0; /* skrollda yuqorida yopishib qoladi */
}
```

**⚠️ Ehtiyot bo'l:** `absolute` element eng yaqin `static` bo'lmagan ajdodga bog'lanadi. Ko'pincha ota elementga `position: relative` qo'yish unutiladi.

---

## Flexbox

**💡 Tushuncha:** Flexbox — bir o'lchovli (qator yoki ustun) layout tizimi. Konteyner `display: flex`, ichidagilar flex item.

Ikki o'q: **main axis** (`flex-direction` belgilaydi) va **cross axis** (perpendikulyar).

```css
.container {
  display: flex;
  flex-direction: row;        /* main o'q: gorizontal */
  justify-content: space-between; /* main o'q bo'ylab */
  align-items: center;        /* cross o'q bo'ylab */
  gap: 16px;
}
```

- **`justify-content`:** main o'q (`flex-start`, `center`, `space-between`, `space-around`).
- **`align-items`:** cross o'q (`stretch`, `center`, `flex-start`).

**Item xususiyatlari:**

```css
.item {
  flex-grow: 1;   /* ortiqcha joyni qanday taqsimlash */
  flex-shrink: 1; /* joy yetmasa qanchaga qisqarish */
  flex-basis: 200px; /* boshlang'ich o'lcham */
  /* qisqacha: flex: 1 1 200px; */
}
```

**⚠️ Ehtiyot bo'l:** `flex: 1` = `flex: 1 1 0%` — bu barcha itemlarni teng kenglikka cho'zadi, content o'lchamiga emas.

---

## CSS Grid

**💡 Tushuncha:** Grid — ikki o'lchovli (qator + ustun) layout tizimi. Murakkab tarmoqli joylashuv uchun ideal.

```css
.grid {
  display: grid;
  grid-template-columns: 1fr 2fr 1fr; /* 3 ustun, o'rta 2x kengroq */
  grid-template-rows: auto;
  gap: 20px;
}
```

- **`fr`:** ortiqcha joyni ulush bo'yicha taqsimlaydi.
- **`gap`:** kataklar orasidagi masofa.
- **`repeat()`:** `grid-template-columns: repeat(3, 1fr)`.
- **`minmax()`:** `repeat(auto-fill, minmax(200px, 1fr))` — responsive.

**Grid areas** — nomli sohalar bilan:

```css
.layout {
  display: grid;
  grid-template-areas:
    "header header"
    "sidebar main"
    "footer footer";
  grid-template-columns: 200px 1fr;
}
.header { grid-area: header; }
.sidebar { grid-area: sidebar; }
.main { grid-area: main; }
.footer { grid-area: footer; }
```

**💡 Tushuncha:** Bir o'lchov (qatorlar yoki ustunlar) uchun Flexbox, ikki o'lchovli to'liq layout uchun Grid yaxshiroq.

---

## Birliklar

**💡 Tushuncha:** CSS birliklari absolyut yoki nisbiy bo'ladi.

| Birlik | Asos |
|---|---|
| `px` | Absolyut piksel |
| `%` | Ota element o'lchamiga nisbatan |
| `em` | Element `font-size` iga nisbatan |
| `rem` | Root (`<html>`) `font-size` iga nisbatan |
| `vw` / `vh` | Viewport kengligi/balandligining 1% i |
| `ch` | "0" belgisi kengligi |

```css
.title {
  font-size: 2rem;   /* 32px, agar root 16px bo'lsa */
  padding: 1em;      /* element font-size iga bog'liq */
  width: 50vw;       /* viewport kengligining yarmi */
}
```

**⚠️ Ehtiyot bo'l:** `em` zanjirlanib ko'payadi: ichma-ich elementlarda `font-size: 1.5em` har darajada kattalashib ketadi. `rem` bunday muammoga uchramaydi.

---

## Responsive va mobile-first

**💡 Tushuncha:** Media query ekran o'lchamiga qarab uslubni o'zgartiradi. **Mobile-first** — avval kichik ekran uslubini yozib, keyin `min-width` bilan kattaroqlarni qo'shish.

```css
/* Mobile-first: bazaviy uslub kichik ekran uchun */
.card {
  width: 100%;
}

/* Keyin kattaroq ekranlar */
@media (min-width: 768px) {
  .card {
    width: 50%;
  }
}

@media (min-width: 1024px) {
  .card {
    width: 33.33%;
  }
}
```

**⚠️ Ehtiyot bo'l:** Mobile-first da `min-width`, desktop-first da `max-width` ishlatiladi. Aralashtirish breakpoint chalkashligiga olib keladi.

---

## em vs rem

**💡 Tushuncha:** Ikkalasi nisbiy, lekin asosi farq qiladi.

- **`em`:** elementning *o'z* `font-size` iga (yoki padding/margin uchun shu elementning font-size iga) nisbatan.
- **`rem`:** doim *root* (`<html>`) `font-size` iga nisbatan.

```css
html { font-size: 16px; }

.parent { font-size: 20px; }
.parent .child {
  font-size: 2em;  /* 2 * 20 = 40px (ota dan) */
  margin: 2rem;    /* 2 * 16 = 32px (root dan) */
}
```

**💡 Tushuncha:** Komponent ichida masshtablanadigan o'lchamlar uchun `em`, global izchil o'lchamlar uchun `rem` qulay.

---

## z-index va stacking context

**💡 Tushuncha:** `z-index` elementlarning qatlamlanish tartibini belgilaydi, lekin faqat **stacking context** ichida ishlaydi.

Stacking context hosil qiluvchi holatlar: `position` + `z-index` (auto bo'lmagan), `opacity < 1`, `transform`, `filter`, `will-change`, `flex`/`grid` item + `z-index`.

```css
.modal {
  position: fixed;
  z-index: 1000;
}
```

**⚠️ Ehtiyot bo'l:** Agar ota element yangi stacking context yaratsa, bolaning `z-index: 9999` i ham boshqa kontekstdagi elementdan o'ta olmaydi. Bu eng ko'p uchraydigan z-index muammosi.

---

## Pseudo-class vs pseudo-element

**💡 Tushuncha:** Pseudo-class element *holatini* tanlaydi (`:hover`, `:focus`, `:nth-child`). Pseudo-element elementning *qismini* yoki yangi virtual qism yaratadi (`::before`, `::after`, `::first-line`).

```css
a:hover { color: red; }              /* holat */
.quote::before { content: "« "; }    /* virtual element */
p::first-line { font-weight: bold; } /* qism */
```

**⚠️ Ehtiyot bo'l:** Pseudo-element ikki ikki nuqta (`::`) bilan, pseudo-class bitta (`:`) bilan yoziladi. `::before`/`::after` uchun `content` majburiy, aks holda ko'rinmaydi.

---

## CSS variables

**💡 Tushuncha:** CSS custom properties (variables) qiymatni qayta ishlatish uchun. `--` bilan e'lon qilinadi, `var()` bilan o'qiladi. Ular *kaskadlanadi* va JS dan o'zgartiriladi (preprocessor o'zgaruvchilaridan farqli).

```css
:root {
  --primary: #3b82f6;
  --space: 16px;
}

.button {
  background: var(--primary);
  padding: var(--space);
  color: var(--text, white); /* fallback: white */
}
```

```js
document.documentElement.style.setProperty('--primary', '#ef4444');
```

**💡 Tushuncha:** `:root` da e'lon qilinsa global; biror selektor ichida e'lon qilinsa faqat shu daraxtda amal qiladi (theme switching uchun qulay).

---

## Transition, animation, keyframes

**💡 Tushuncha:** `transition` ikki holat orasini silliq o'zgartiradi (trigger kerak, masalan `:hover`). `animation` `@keyframes` bilan mustaqil, takrorlanuvchi harakat beradi.

```css
.btn {
  background: blue;
  transition: background 0.3s ease;
}
.btn:hover {
  background: darkblue;
}
```

```css
@keyframes pulse {
  0%   { transform: scale(1); }
  50%  { transform: scale(1.1); }
  100% { transform: scale(1); }
}
.icon {
  animation: pulse 2s ease-in-out infinite;
}
```

**⚠️ Ehtiyot bo'l:** `transition` faqat animatsiyalanuvchi xususiyatlarda ishlaydi (`opacity`, `transform`, ranglar). `display` ni transition qilib bo'lmaydi.

---

## Transform

**💡 Tushuncha:** `transform` elementni layout ni buzmasdan ko'chiradi, aylantiradi, masshtablaydi. GPU da ishlagani uchun performant.

```css
.box {
  transform: translateX(50px) rotate(45deg) scale(1.2);
}
```

- `translate(x, y)` — siljitish.
- `rotate(deg)` — aylantirish.
- `scale(n)` — kattalashtirish.
- `skew()` — qiyshaytirish.

**💡 Tushuncha:** `transform` va `opacity` faqat **compositing** bosqichida ishlaydi — reflow/repaint chaqirmaydi, shuning uchun animatsiya uchun eng tejamli.

---

## Selektorlar

**💡 Tushuncha:** Selektorlar elementlarni tanlaydi. Combinator lar element munosabatini bildiradi.

```css
.a .b   { }  /* descendant: .a ichidagi har qanday .b */
.a > .b { }  /* child: faqat bevosita bola */
.a + .b { }  /* adjacent sibling: keyingi qo'shni */
.a ~ .b { }  /* general sibling: keyingi barcha qo'shnilar */
```

**Attribute selektorlar:**

```css
input[type="email"] { }      /* aniq qiymat */
a[href^="https"]   { }       /* https bilan boshlanuvchi */
a[href$=".pdf"]    { }       /* .pdf bilan tugaydigan */
a[href*="blog"]    { }       /* blog ni o'z ichiga oluvchi */
```

---

## Margin collapsing

**💡 Tushuncha:** Vertikal (yuqori/past) margin lar ba'zan birlashadi — ikkitasidan kattasi qoladi, yig'indisi emas.

```css
.a { margin-bottom: 30px; }
.b { margin-top: 20px; }
/* .a va .b orasidagi masofa 50px emas, 30px (kattasi) */
```

Birlashish holatlari: qo'shni block elementlar, ota-bola (orada border/padding bo'lmasa), bo'sh blok.

**⚠️ Ehtiyot bo'l:** Margin collapsing faqat **vertikal** marginlarda va **block** oqimda bo'ladi. Flex/grid item larda bo'lmaydi. `padding`, `border` yoki `overflow` qo'shish uni to'xtatadi.

---

## Overflow

**💡 Tushuncha:** `overflow` kontent konteynerdan oshib ketsa nima bo'lishini boshqaradi.

- `visible` (default) — oshib chiqib ko'rinadi.
- `hidden` — kesib tashlanadi.
- `scroll` — doim skrollbar.
- `auto` — kerak bo'lganda skrollbar.

```css
.box {
  height: 100px;
  overflow-y: auto;
}
```

**💡 Tushuncha:** `overflow: hidden` yoki `auto` yangi **block formatting context** yaratadi — float ni o'rab oladi va margin collapsing ni to'xtatadi.

---

## Centerlash usullari

**💡 Tushuncha:** Markazlashtirishning bir necha usuli bor; vaziyatga qarab tanlanadi.

**Flexbox (eng oson):**

```css
.parent {
  display: flex;
  justify-content: center; /* gorizontal */
  align-items: center;     /* vertikal */
}
```

**Grid:**

```css
.parent {
  display: grid;
  place-items: center;
}
```

**Absolute + transform:**

```css
.child {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
}
```

**Margin auto (faqat gorizontal block):**

```css
.child { width: 300px; margin: 0 auto; }
```

---

## BEM

**💡 Tushuncha:** BEM (Block, Element, Modifier) — class nomlash konvensiyasi. Specificity past va izchil bo'ladi.

```css
.card { }              /* Block */
.card__title { }       /* Element (block ning qismi) */
.card--featured { }    /* Modifier (variant) */
.card__title--large { }/* Element + Modifier */
```

```html
<div class="card card--featured">
  <h2 class="card__title card__title--large">Sarlavha</h2>
</div>
```

**💡 Tushuncha:** BEM ning maqsadi — tekis (flat), past specificity li selektorlar bilan stil to'qnashuvlarini oldini olish va kodni o'qilishini oshirish.

---

## CSS Modules vs CSS-in-JS vs Tailwind

**💡 Tushuncha:** Zamonaviy ilovalarda stilni boshqarishning uch yondashuvi.

| Yondashuv | Ishlash | Plus | Minus |
|---|---|---|---|
| **CSS Modules** | Lokal scope li class lar (`.css` fayl) | Sof CSS, scope kafolati | Dinamik stil cheklangan |
| **CSS-in-JS** | JS ichida stil (`styled-components`) | Dinamik, komponentga bog'liq | Runtime narxi, bundle |
| **Tailwind** | Utility class lar (`p-4 flex`) | Tez, izchil dizayn tizimi | HTML to'lib ketadi, o'rganish |

**💡 Tushuncha:** CSS Modules build vaqtida class nomlarini unikal qiladi (`card_x7f3`), shu sababli global to'qnashuv bo'lmaydi. Tailwind oldindan yozilgan utility class larni beradi.

---

## Reflow vs repaint

**💡 Tushuncha:** **Reflow (layout)** — geometriya (o'lcham, joylashuv) qayta hisoblanishi; qimmat. **Repaint** — faqat ko'rinish (rang, soya) yangilanishi; arzonroq.

- **Reflow chaqiradi:** `width`, `height`, `margin`, `top`, font o'zgarishi, DOM qo'shish.
- **Faqat repaint:** `color`, `background`, `visibility`, `box-shadow`.
- **Ikkalasi ham YO'Q (eng tez):** `transform`, `opacity` (compositing).

**⚠️ Ehtiyot bo'l:** Animatsiyada `top`/`left` o'rniga `transform: translate()` ishlat — bu reflow ni oldini oladi va silliqroq ishlaydi (60fps).

---

## Intervyu savollari

### ❓ Specificity qanday hisoblanadi?

**✅ Javob:** `(inline, ID, class/attr/pseudo-class, tag/pseudo-element)` ko'rinishida. Inline eng kuchli, keyin ID lar soni, keyin class/attribute/pseudo-class soni, eng oxiri tag soni. Teng bo'lsa manbada oxirgi yozilgan g'olib. `!important` bularning hammasini chetlab o'tadi.

### ❓ `box-sizing: border-box` nima qiladi?

**✅ Javob:** `width`/`height` ni padding va border ni *o'z ichiga oladigan* qiladi. Default `content-box` da padding va border qo'shimcha qo'shiladi va umumiy o'lcham oshadi. `border-box` o'lcham hisobini soddalashtiradi, shuning uchun ko'pincha global qo'yiladi.

### ❓ Flexbox va Grid qachon ishlatiladi?

**✅ Javob:** Flexbox bir o'lchovli layout (bitta qator yoki ustun, masalan navbar) uchun. Grid ikki o'lchovli layout (qator + ustun bir vaqtda, masalan sahifa tarkibi) uchun. Ko'pincha sahifa Grid bilan, komponentlar ichi Flexbox bilan tuziladi.

### ❓ `position: absolute` nimaga nisbatan joylashadi?

**✅ Javob:** Eng yaqin `position` qiymati `static` bo'lmagan (relative/absolute/fixed/sticky) ajdodga nisbatan. Agar bunday ajdod bo'lmasa — `<html>` (initial containing block) ga nisbatan. Shu sababli ota elementga `position: relative` qo'yiladi.

### ❓ `em` va `rem` farqi nima?

**✅ Javob:** `em` elementning o'z `font-size` iga nisbatan (ichma-ich zanjirlanib ko'payadi). `rem` doim root (`<html>`) `font-size` iga nisbatan (zanjir muammosi yo'q). Izchil global o'lchamlar uchun `rem`, komponentga moslashuvchan o'lcham uchun `em`.

### ❓ z-index nega ba'zan ishlamaydi?

**✅ Javob:** Chunki u faqat o'z **stacking context** i ichida ishlaydi. Agar ota element yangi stacking context yaratsa (`opacity < 1`, `transform`, `position`+`z-index`), bolaning katta `z-index` i ham boshqa kontekstdagi elementdan o'ta olmaydi. `z-index` faqat `position` static bo'lmaganda ishlaydi.

### ❓ Pseudo-class va pseudo-element farqi?

**✅ Javob:** Pseudo-class element holatini tanlaydi (`:hover`, `:focus`, `:nth-child`) va bitta nuqta bilan yoziladi. Pseudo-element elementning qismini yoki virtual element yaratadi (`::before`, `::after`) va ikki nuqta bilan yoziladi. `::before`/`::after` uchun `content` shart.

### ❓ Margin collapsing nima?

**✅ Javob:** Qo'shni block elementlarning vertikal marginlari birlashib, yig'indi emas, kattasi qoladi. Masalan 30px va 20px margin orasida 30px masofa qoladi. Faqat vertikal va block oqimda bo'ladi; flex/grid da yoki `padding`/`border`/`overflow` qo'shilganda to'xtaydi.

### ❓ Reflow va repaint farqi?

**✅ Javob:** Reflow — element geometriyasi (o'lcham, joylashuv) qayta hisoblanishi, qimmat. Repaint — faqat ko'rinish (rang, soya) yangilanishi, arzonroq. `transform` va `opacity` ikkalasini ham chaqirmaydi (compositing), shuning uchun animatsiya uchun eng tejamli.

### ❓ CSS variables va Sass o'zgaruvchilari farqi?

**✅ Javob:** CSS variables (`--x`) runtime da yashaydi, kaskadlanadi, JS dan o'zgartiriladi, media query bilan o'zgaradi. Sass o'zgaruvchilari (`$x`) faqat compile vaqtida mavjud — natija CSS da o'zgaruvchi qolmaydi. Theme switching uchun CSS variables kerak.

### ❓ Elementni vertikal va gorizontal markazlash usullari?

**✅ Javob:** Eng oson — `display: flex; justify-content: center; align-items: center`. Grid da `place-items: center`. Yoki `position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%)`. Faqat gorizontal block uchun `margin: 0 auto`.

### ❓ Tailwind, CSS Modules va CSS-in-JS ni taqqoslang?

**✅ Javob:** CSS Modules — lokal scope li sof CSS (build vaqtida unikal class). CSS-in-JS — JS ichida dinamik stil, lekin runtime narxi bor. Tailwind — utility class lar, tez va izchil, lekin HTML to'lib ketadi. Tanlash jamoa va loyiha ehtiyojiga bog'liq.

---

## Masalalar

> Yechimlar: [Yechimlarni ko'rish](../solutions/frontend/02-css.md)

1. Berilgan uchta selektorning specificity sini `(a,b,c)` ko'rinishida hisoblang va qaysi biri g'olibligini aniqlang.
2. `box-sizing: content-box` va `border-box` da bir xil `width: 200px; padding: 20px; border: 5px` elementning real kengligini hisoblang.
3. Flexbox bilan navbar tuzing: chapda logo, o'ngda menyu, vertikal markazlangan.
4. CSS Grid bilan `header / sidebar+main / footer` layout ni `grid-template-areas` orqali yarating.
5. Mobile-first yondashuvda kartochka: telefonda 100%, planshetda 50%, desktopda 33% kenglik bo'lsin.
6. Modal oynani ekran markaziga uchta turli usul (flex, grid, absolute+transform) bilan joylashtiring.
7. `:root` da rang o'zgaruvchilarini e'lon qilib, ularni JS bilan o'zgartiruvchi (theme switch) kod yozing.
8. `@keyframes` bilan tugmaga "pulse" animatsiyasi qo'shing va `:hover` da transition ishlatib rangini o'zgartiring.
9. z-index ishlamayotgan holatni keltiring (ota stacking context yaratgan) va uni to'g'rilang.
10. Animatsiyani `top`/`left` o'rniga `transform` bilan qayta yozib, nega tezroq ishlashini izohlang.

---

← [Frontend bo'limiga qaytish](./README.md)
