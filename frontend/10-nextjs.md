# Next.js

Next.js — React ekotizimidagi eng ko'p ishlatiladigan **full-stack framework**. Agar React "kutubxona" (library) bo'lsa, Next.js — o'sha React atrofiga rendering, routing, data fetching, optimization va backend'ni bir joyga yig'ib bergan "batteries included" tizim. 2020-yildan buyon intervyularda Next.js savollari deyarli standart bo'lib qoldi, chunki ko'p kompaniyalar production'da aynan uni ishlatadi. Bu hujjat 2026-yil holatiga ko'ra yozilgan — asosiy urg'u **App Router** (`app/`) va **React Server Components** (RSC) ga qaratilgan, chunki bugun yangi loyihalar aynan shu asosda quriladi.

Intervyuda Next.js savollari ko'pincha bitta o'zakka boradi: **"Bu sahifani qaysi rendering strategiyasi bilan render qilasan va nega?"** Shu savolga trade-off'lar bilan javob bera olsangiz — senior darajaga yaqinlashasiz. Shu sababli bu hujjatning yuragida rendering strategiyalari (CSR, SSR, SSG, ISR) turadi.

## Mundarija

- [Next.js nima va nega](#nextjs-nima-va-nega)
- [Rendering strategiyalari: umumiy manzara](#rendering-strategiyalari-umumiy-manzara)
- [CSR — Client-Side Rendering](#csr--client-side-rendering)
- [SSR — Server-Side Rendering](#ssr--server-side-rendering)
- [SSG — Static Site Generation](#ssg--static-site-generation)
- [ISR — Incremental Static Regeneration](#isr--incremental-static-regeneration)
- [Rendering strategiyalarini taqqoslash jadvali](#rendering-strategiyalarini-taqqoslash-jadvali)
- [Hydration nima va nega](#hydration-nima-va-nega)
- [App Router vs Pages Router](#app-router-vs-pages-router)
- [React Server Components va Client Components](#react-server-components-va-client-components)
- [Data fetching](#data-fetching)
- [File-based routing](#file-based-routing)
- [Next.js'ning React'dan afzalliklari](#nextjsning-reactdan-afzalliklari)
- [next/image, next/link, metadata](#nextimage-nextlink-metadata)
- [API routes va Route Handlers](#api-routes-va-route-handlers)
- [Middleware va environment variables](#middleware-va-environment-variables)
- [Savol-javoblar](#savol-javoblar)
- [Masalalar](#masalalar)

---

## Next.js nima va nega

**💡 Tushuncha:** React o'zi faqat **UI kutubxonasi** — u komponentlarni brauzerda ekranga chizadi, xolos. React'da routing yo'q, server yo'q, SEO yo'q, rasm optimizatsiyasi yo'q. Bularning hammasini o'zingiz yig'ib olishingiz kerak (react-router, Vite, custom server...). Next.js esa — React atrofidagi **framework**, ya'ni bularning hammasi allaqachon ichida, bitta standart yechim bilan.

Oddiy React (Vite yoki eskiroq CRA) bilan farqni oddiy qilib solishtiraylik:

| Vazifa | Oddiy React (Vite) | Next.js |
|---|---|---|
| Routing | react-router-dom o'rnatasan | Fayl tizimi orqali (`app/` papkasi) |
| Server-side rendering | Yo'q (yoki juda qiyin) | Ichida, avtomatik |
| SEO (meta teglar) | Qo'lda, `react-helmet` | `metadata` API bilan built-in |
| Rasm optimizatsiyasi | Qo'lda | `next/image` avtomatik |
| Backend / API | Alohida server (Express) | `route.ts` — o'sha loyihada |
| Code splitting | Qo'lda `React.lazy` | Har sahifa avtomatik split |
| Build/bundler | O'zing sozlaysan | Ichida (Turbopack) |

**"Batteries included"** iborasi shuni anglatadi: batareyalar (kerakli qismlar) qutining ichida keladi — qo'shimcha sotib olishing shart emas. Next.js xuddi shunday: routing, SSR, optimization, backend — hammasi bir paketda.

**Nega Next.js kerak bo'ladi?** Uch asosiy sabab:

1. **SEO** — Google botlari sahifaning tayyor HTML'ini ko'rishi kerak. Oddiy React SPA'da bot bo'sh `<div id="root"></div>` ko'radi. Next.js serverda HTML'ni to'ldirib beradi.
2. **Birinchi yuklanish tezligi** — foydalanuvchi tayyor kontentni tezroq ko'radi (blank page yo'q).
3. **Full-stack** — bitta loyihada frontend ham, backend API ham — alohida server tikish shart emas.

**⚠️ Ehtiyot bo'l:** "Next.js har doim yaxshiroq" degan xato tushuncha keng tarqalgan. Agar loyihang faqat login orqasidagi ichki **dashboard** bo'lsa (SEO kerak emas, hamma sahifa dinamik), oddiy React + Vite ko'pincha yetarli va soddaroq. Next.js kuchini SEO va content-heavy sahifalarda ko'rsatadi.

---

## Rendering strategiyalari: umumiy manzara

**💡 Tushuncha:** "Rendering strategiyasi" — bu **HTML qayerda va qachon yaratiladi** degan savolga javob. Uch o'rin bor: brauzerda (CSR), serverda har so'rovda (SSR), yoki build vaqtida bir marta (SSG). ISR esa SSG'ning yangilanadigan varianti. Next.js'ning eng kuchli tomoni — bu strategiyalarni **sahifama-sahifa aralashtira olishing**. Bitta loyihada blog SSG, profil SSR, hisobot CSR bo'lishi mumkin.

Farqni tushunish uchun bitta savolni yodda tut: **"HTML foydalanuvchiga yetib borgunicha to'liq tayyor bo'ladimi, yoki brauzerda JavaScript ishga tushgach yasaladimi?"**

Endi har birini alohida, misol bilan ko'ramiz.

---

## CSR — Client-Side Rendering

**💡 Tushuncha:** CSR — bu klassik React SPA. Server deyarli bo'sh HTML jo'natadi, so'ng brauzerda JavaScript yuklanadi, ishga tushadi va DOM'ni **brauzerda** yasaydi. Ma'lumot ham brauzerdan `fetch` bilan olinadi. Vite yoki CRA bilan qilingan har qanday oddiy React app — bu CSR.

Oqim:

```
1. Brauzer bo'sh HTML oladi:  <div id="root"></div>
2. JS bundle yuklanadi (kutish...)
3. React ishga tushadi
4. useEffect ichida fetch qiladi (yana kutish...)
5. Nihoyat kontent ekranda
```

Next.js'da CSR odatda `"use client"` komponent ichida, `useEffect` bilan ma'lumot olganingizda yuzaga keladi:

```tsx
"use client";

import { useEffect, useState } from "react";

type Stat = { visits: number };

export function DashboardStats() {
  const [data, setData] = useState<Stat | null>(null);

  useEffect(() => {
    // Bu fetch faqat brauzerda ishlaydi
    fetch("/api/stats")
      .then((r) => r.json())
      .then(setData);
  }, []);

  if (!data) return <p>Yuklanmoqda...</p>; // foydalanuvchi shuni ko'radi
  return <p>Tashriflar: {data.visits}</p>;
}
```

**Kamchiliklari:**

- **SEO yomon** — Google boti dastlab bo'sh `<div>` ko'radi. Kontent JS ishga tushgach paydo bo'ladi; ba'zi botlar buni kutmaydi.
- **Blank page / spinner** — birinchi kadr bo'sh yoki "Yuklanmoqda...". Foydalanuvchi kutadi.
- **Sekin birinchi kontent (FCP/LCP yomon)** — JS yuklanishi + ishga tushishi + fetch = ketma-ket kutishlar.

**Qachon mos:** SEO kerak bo'lmagan, login orqasidagi interaktiv sahifalar (dashboard, admin panel, ichki asboblar).

**⚠️ Ehtiyot bo'l:** CSR'da ma'lumot faqat brauzerda olingani uchun API kaliti kabi maxfiy narsalarni bu yerda ishlatib bo'lmaydi — ular brauzerga oshkor bo'lib qoladi.

---

## SSR — Server-Side Rendering

**💡 Tushuncha:** SSR — **har bir so'rovda** server HTML'ni to'liq yasab, tayyor holda brauzerga jo'natadi. Foydalanuvchi darhol kontentli sahifani ko'radi. So'ng brauzerda JavaScript yuklanib, o'sha HTML'ni "jonlantiradi" (hydration — pastda batafsil).

App Router'da SSR — bu ma'lumotni **dinamik** (har so'rovda yangi) olayotgan server component. Masalan `cookies()` ishlatsang yoki `cache: "no-store"` desang, sahifa avtomatik dinamik (SSR) bo'ladi:

```tsx
// app/profile/page.tsx  — server component (default)
import { cookies } from "next/headers";

export default async function ProfilePage() {
  const session = (await cookies()).get("session")?.value;

  // Har so'rovda serverda yangi ma'lumot olinadi
  const res = await fetch("https://api.example.com/me", {
    headers: { Authorization: `Bearer ${session}` },
    cache: "no-store", // dinamik = SSR
  });
  const user = await res.json();

  return <h1>Salom, {user.name}!</h1>;
}
```

**Qachon mos:**

- **Personalizatsiya** — har foydalanuvchiga har xil kontent (profil, savat, dashboard'ning ommaviy qismi).
- **Tez-tez o'zgaradigan dinamik ma'lumot** — narxlar, mavjudlik, real-time'ga yaqin ma'lumot.
- **SEO + dinamiklik birga kerak** — masalan har so'rovda yangilanadigan e-commerce mahsulot sahifasi.

**Afzalligi:** SEO yaxshi (bot tayyor HTML ko'radi), foydalanuvchi darhol kontent ko'radi, ma'lumot har doim yangi.

**Narxi (trade-off):** Har so'rov serverda ish talab qiladi → **server yuki va TTFB** (Time To First Byte) oshadi. SSG'ga qaraganda sekinroq va qimmatroq. Buni CDN cache yoki ISR bilan yumshatish mumkin.

---

## SSG — Static Site Generation

**💡 Tushuncha:** SSG — HTML **build vaqtida** (siz `next build` qilganingizda) bir marta yasaladi va tayyor fayl sifatida saqlanadi. Foydalanuvchi so'rov yuborsa, server hech narsa hisoblamaydi — shunchaki tayyor HTML faylni beradi (odatda CDN'dan). Bu **eng tez** variant.

App Router'da agar komponent oddiy `fetch` qilsa (default cache) yoki umuman dinamik funksiya ishlatmasa, u avtomatik statik bo'ladi:

```tsx
// app/blog/page.tsx — build vaqtida bir marta render bo'ladi
export default async function BlogPage() {
  // Default cache — natija build vaqtida "muzlatiladi"
  const res = await fetch("https://api.example.com/posts");
  const posts: { id: string; title: string }[] = await res.json();

  return (
    <ul>
      {posts.map((p) => (
        <li key={p.id}>{p.title}</li>
      ))}
    </ul>
  );
}
```

**Qachon mos:** kamdan-kam o'zgaradigan, hamma uchun bir xil kontent:

- **Blog, hujjatlar (docs)**
- **Marketing / landing sahifalar**
- **Mahsulot katalogi** (agar ma'lumot tez-tez o'zgarmasa)

**Afzalligi:** eng tez (TTFB deyarli nol), server yuki minimal, CDN'dan tarqatiladi, arzon, SEO mukammal.

**⚠️ Ehtiyot bo'l:** SSG'ning kamchiligi — ma'lumot **build vaqtidagi** holatda "muzlaydi". Agar post o'zgarса, sahifani ko'rish uchun qayta build qilish kerak (yoki ISR ishlatish). Shu sababli tez-tez o'zgaradigan ma'lumot uchun toza SSG mos emas.

---

## ISR — Incremental Static Regeneration

**💡 Tushuncha:** ISR — SSG'ning aqlli varianti. Sahifa baribir statik (tez, CDN'dan), lekin Next.js uni **vaqti-vaqti bilan fon rejimida qayta yasaydi**. Ya'ni "static, lekin bayat bo'lib qolmaydigan". Bu SSG tezligi bilan SSR yangiligini birlashtiradi — hech kimni kutdirmasdan.

App Router'da ISR — bu `next.revalidate` bilan beriladi:

```tsx
// app/products/page.tsx
export default async function ProductsPage() {
  const res = await fetch("https://api.example.com/products", {
    next: { revalidate: 60 }, // har 60 soniyada qayta yangilash
  });
  const products = await res.json();

  return <ProductGrid items={products} />;
}
```

**Qanday ishlaydi (revalidate: 60):**

```
1. Foydalanuvchi keladi → tayyor statik HTML oladi (tez)
2. Agar oxirgi yasashdan 60s o'tgan bo'lsa:
   - Joriy foydalanuvchi baribir eski (stale) sahifani oladi — kutmaydi
   - Fon rejimida Next.js yangi HTML yasaydi
3. Keyingi foydalanuvchi yangi HTML oladi
```

Bu **"stale-while-revalidate"** naqshi — eskisini ber, orqada yangila.

**Qachon mos:** tez-tez emas, lekin muntazam o'zgaradigan ma'lumot bo'lganda: e-commerce narxlari, yangiliklar ro'yxati, ko'p o'qiladigan lekin har soatda bir yangilanadigan sahifalar.

**On-demand revalidation** — vaqt o'rniga hodisaga qarab yangilash ham mumkin (masalan CMS'da post tahrirlanganda `revalidatePath("/blog")` chaqirish). Bu eng nozik nazorat.

---

## Rendering strategiyalarini taqqoslash jadvali

| Xususiyat | CSR | SSR | SSG | ISR |
|---|---|---|---|---|
| **HTML qayerda yasaladi** | Brauzerda | Serverda (har so'rov) | Build vaqtida | Build + fon yangilash |
| **SEO** | Yomon | Yaxshi | Mukammal | Mukammal |
| **Birinchi kontent tezligi** | Sekin (blank) | O'rta | Eng tez | Eng tez |
| **Server yuki** | Yo'q | Yuqori | Yo'q | Juda kam |
| **Ma'lumot yangiligi** | Har doim yangi | Har doim yangi | Build vaqtidagi | Muntazam yangilanadi |
| **Qachon** | Login orqasi dashboard | Personalizatsiya, dinamik | Blog, marketing | Narx, yangilik ro'yxati |
| **Narx** | Arzon | Qimmat | Eng arzon | Arzon |

**Eslab qol:** App Router'da bu strategiyalarni **alohida "rejim"** deb tanlamaysan — ular sening `fetch` sozlamalaringdan **avtomatik kelib chiqadi**:

- `cache: "no-store"` yoki dinamik funksiya (`cookies`, `headers`) → **SSR**
- default `fetch` (statik) → **SSG**
- `next: { revalidate: N }` → **ISR**
- `"use client"` + `useEffect` fetch → **CSR**

---

## Hydration nima va nega

**💡 Tushuncha:** Hydration ("suvlantirish") — server yuborgan **quruq HTML**'ga brauzerdagi React **interaktivlik qo'shishi** jarayoni. Server tayyor HTML beradi (foydalanuvchi darhol ko'radi), keyin JS yuklanib, o'sha mavjud HTML'ga event listener'lar, state, click'lar — hayot ulaydi. HTML'ni "suv bilan jonlantirish" degani.

Nega kerak? SSR/SSG'da server faqat **HTML matni** yuboradi — undagi `onClick` ishlamaydi, `useState` bo'sh. React brauzerda ishga tushib, "men bu HTML'ni yasagan bo'lardim, endi uni o'zimniki qilib olaman" deydi va tugmalarni jonlantiradi.

```
SSR/SSG oqimi:
1. Server → tayyor HTML (ko'rinadi, lekin "o'lik" — tugmalar ishlamaydi)
2. Brauzer → HTML'ni chizadi (foydalanuvchi kontentni ko'radi, FCP tez)
3. JS bundle yuklanadi
4. HYDRATION — React HTML'ni "o'ziniki" qilib, interaktiv qiladi
5. Endi tugmalar, formalar ishlaydi
```

**⚠️ Ehtiyot bo'l — Hydration mismatch xatosi:** Server va client **bir xil HTML** yasashi shart. Agar farq qilsa (masalan `Date.now()`, `Math.random()`, yoki faqat brauzerda mavjud `window`'dan foydalansang), React "hydration failed" xatosini beradi. Klassik xato:

```tsx
// XATO — server va client turli natija beradi
function Clock() {
  return <p>{new Date().toLocaleTimeString()}</p>; // hydration mismatch!
}

// TO'G'RI — vaqtga bog'liq narsani client'da, mount'dan keyin ko'rsat
"use client";
function Clock() {
  const [time, setTime] = useState<string | null>(null);
  useEffect(() => setTime(new Date().toLocaleTimeString()), []);
  return <p>{time ?? "—"}</p>;
}
```

---

## App Router vs Pages Router

**💡 Tushuncha:** Next.js'da ikki xil routing tizimi bor. **Pages Router** (`pages/` papkasi) — eski, 2016-2022 asosiy tizim. **App Router** (`app/` papkasi) — Next.js 13+ (2022) dan boshlab yangi standart, React Server Components asosida qurilgan. 2026-yilda yangi loyihalar App Router bilan boshlanadi. Ikkalasi bitta loyihada birga ham yashashi mumkin (migratsiya davri uchun).

| Xususiyat | Pages Router (`pages/`) | App Router (`app/`) |
|---|---|---|
| Fayl nomi | `pages/about.tsx` → `/about` | `app/about/page.tsx` → `/about` |
| Server Components | Yo'q | Ha (default) |
| Layout | `_app.tsx` (bitta global) | `layout.tsx` (har papkada, nested) |
| Data fetching | `getServerSideProps` / `getStaticProps` | `async` server component + `fetch` |
| Loading holati | Qo'lda | `loading.tsx` (avtomatik) |
| Error holati | Qo'lda | `error.tsx` (avtomatik) |
| Streaming | Cheklangan | Built-in (Suspense) |

App Router'ning asosiy g'oyasi: **default'da hamma narsa serverda ishlaydi** (Server Component), faqat kerak bo'lganda `"use client"` bilan brauzerga o'tkazasan. Bu bundle hajmini kamaytiradi va ma'lumot olishni serverga yaqinlashtiradi.

**⚠️ Ehtiyot bo'l:** Intervyuda kod misol so'ralsa, qaysi router haqida gaplashayotganingni aniq ayt. `getServerSideProps` — Pages Router; `async` server component — App Router. Ularni aralashtirsang, "eskirgan bilim" degan taassurot qoldirasan.

---

## React Server Components va Client Components

**💡 Tushuncha:** App Router'ning yuragida **React Server Components (RSC)** turadi. `app/` ichidagi har bir komponent **default'da Server Component** — ya'ni u faqat **serverda** ishlaydi, uning kodi brauzerga umuman yuborilmaydi. Agar komponentga interaktivlik (state, event, brauzer API) kerak bo'lsa, fayl boshiga `"use client"` yozib, uni **Client Component** qilasan.

**Server Component** nima qila oladi:
- To'g'ridan-to'g'ri `await fetch(...)` yoki DB so'rovi (async bo'lishi mumkin)
- Maxfiy narsalar ishlatishi (API kalitlari — brauzerga chiqmaydi)
- Katta kutubxonalarni ishlatib, ularni brauzerga yubormaslik (bundle kichik qoladi)

**Server Component** nima qila OLMAYDI:
- `useState`, `useEffect`, `useContext` — yo'q
- `onClick`, `onChange` — event listener'lar yo'q
- `window`, `localStorage` — brauzer API'lari yo'q

```tsx
// app/dashboard/page.tsx — SERVER Component (default, "use client" yo'q)
import { db } from "@/lib/db";
import { LikeButton } from "./LikeButton";

export default async function Dashboard() {
  const posts = await db.post.findMany(); // to'g'ridan-to'g'ri DB!

  return (
    <div>
      {posts.map((p) => (
        <article key={p.id}>
          <h2>{p.title}</h2>
          <LikeButton postId={p.id} /> {/* interaktiv qism — client */}
        </article>
      ))}
    </div>
  );
}
```

```tsx
// app/dashboard/LikeButton.tsx — CLIENT Component
"use client";

import { useState } from "react";

export function LikeButton({ postId }: { postId: string }) {
  const [liked, setLiked] = useState(false);
  return (
    <button onClick={() => setLiked((v) => !v)}>
      {liked ? "❤️" : "🤍"} {postId}
    </button>
  );
}
```

**Muhim naqsh:** Server Component ichida Client Component'ni **chaqirish mumkin** (yuqoridagi kabi). Lekin Client Component ichiga Server Component'ni to'g'ridan-to'g'ri import qilib bo'lmaydi — uni `children` prop orqali uzatish kerak.

**⚠️ Ehtiyot bo'l:** `"use client"` "bu komponent faqat brauzerda ishlaydi" degani EMAS. Client Component ham SSR'da serverda bir marta HTML uchun render bo'ladi, keyin brauzerda hydrate bo'ladi. `"use client"` chegara belgilaydi: "shu yerdan pastda hamma narsa brauzer bundle'iga kiradi".

---

## Data fetching

**💡 Tushuncha:** App Router'da ma'lumot olish oddiy: **async server component ichida to'g'ridan-to'g'ri `await fetch`**. Alohida `getServerSideProps` funksiyalari kerak emas. Rendering strategiyasi (SSR/SSG/ISR) esa `fetch`'ning **cache sozlamasidan** kelib chiqadi.

**App Router — hamma variant bitta joyda:**

```tsx
export default async function Page() {
  // 1. SSG (default) — build vaqtida cache
  const a = await fetch("https://api.x.com/static");

  // 2. ISR — har 60s yangilash
  const b = await fetch("https://api.x.com/products", {
    next: { revalidate: 60 },
  });

  // 3. SSR — hech qachon cache qilma, har so'rov yangi
  const c = await fetch("https://api.x.com/live", {
    cache: "no-store",
  });

  // ...
}
```

**Parallel fetching** — bir-biriga bog'liq bo'lmagan so'rovlarni `Promise.all` bilan parallel qil (waterfall'dan qoching):

```tsx
const [user, posts] = await Promise.all([
  fetch("/api/user").then((r) => r.json()),
  fetch("/api/posts").then((r) => r.json()),
]);
```

**Pages Router — qisqacha (eski loyihalar uchun bilish kerak):**

```tsx
// getServerSideProps — har so'rovda serverda (SSR)
export const getServerSideProps: GetServerSideProps = async (ctx) => {
  const data = await fetchData(ctx.query.id);
  return { props: { data } };
};

// getStaticProps — build vaqtida (SSG); revalidate bilan ISR
export const getStaticProps: GetStaticProps = async () => {
  const data = await fetchData();
  return { props: { data }, revalidate: 60 }; // ISR
};
```

**⚠️ Ehtiyot bo'l:** App Router'da Next.js `fetch`'ni avtomatik kesh qiladi va **deduplikatsiya** qiladi (bir render ichida bir xil URL ikki marta so'ralsa — bir marta yuboradi). Client component'da esa `fetch` bu keshdan foydalanmaydi — u yerda TanStack Query kabi kutubxona ishlat.

---

## File-based routing

**💡 Tushuncha:** Next.js'da URL'lar **fayl tuzilishidan** kelib chiqadi — alohida router konfiguratsiyasi yozmaysan. App Router'da papka = route segmenti, va papkada maxsus nomli fayllar maxsus rol o'ynaydi.

**Maxsus fayllar (`app/` ichida):**

| Fayl | Roli |
|---|---|
| `page.tsx` | Route'ning ko'rinadigan UI'i (URL'ni hosil qiladi) |
| `layout.tsx` | O'rab turuvchi qobiq (nested, navigatsiyada saqlanadi) |
| `loading.tsx` | Suspense fallback — sahifa yuklanayotganda |
| `error.tsx` | Error boundary — xato yuz berganda |
| `not-found.tsx` | 404 sahifasi |
| `route.ts` | API endpoint (Route Handler) |

**Dinamik route** — kvadrat qavs bilan:

```
app/
  blog/
    page.tsx              → /blog
    [slug]/
      page.tsx            → /blog/har-qanday-slug
  shop/
    [...categories]/
      page.tsx            → /shop/a/b/c (catch-all)
```

```tsx
// app/blog/[slug]/page.tsx
export default async function Post({
  params,
}: {
  params: Promise<{ slug: string }>;
}) {
  const { slug } = await params; // Next 15+ da params — Promise
  const post = await getPost(slug);
  return <article>{post.title}</article>;
}
```

**Route groups** — `(nom)` qavs bilan: URL'ga ta'sir qilmasdan fayllarni guruhlash yoki har guruhga alohida layout berish uchun:

```
app/
  (marketing)/
    layout.tsx     ← marketing layout
    about/page.tsx → /about  ((marketing) URL'da ko'rinmaydi)
  (shop)/
    layout.tsx     ← shop layout
    cart/page.tsx  → /cart
```

**Nested layout misoli:**

```tsx
// app/layout.tsx — root layout (majburiy)
export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="uz">
      <body>
        <nav>Menyu</nav>
        {children}
      </body>
    </html>
  );
}
```

**⚠️ Ehtiyot bo'l:** `loading.tsx` avtomatik ravishda `page.tsx`'ni `<Suspense>` bilan o'raydi — server component ma'lumot kutayotganda foydalanuvchi darhol loading holatini ko'radi (streaming). Bu App Router'ning kuchli, lekin ko'pincha e'tibordan chetda qoladigan xususiyati.

---

## Next.js'ning React'dan afzalliklari

**💡 Tushuncha:** Intervyuda tez-tez "Nega oddiy React o'rniga Next.js?" deb so'raydi. Aniq ro'yxat bilan javob ber:

1. **SSR/SSG built-in → SEO** — server tayyor HTML beradi, Google ko'radi. Oddiy React'da bu deyarli imkonsiz.
2. **File-based routing** — react-router o'rnatmaysan, papka tuzilishi = URL.
3. **Image optimization (`next/image`)** — avtomatik resize, WebP/AVIF, lazy load, layout shift'ni oldini olish.
4. **Font optimization (`next/font`)** — shriftlarni self-host qiladi, layout shift'siz.
5. **API routes / Route Handlers** — full-stack: backend'ni o'sha loyihada yozasan.
6. **Avtomatik code splitting** — har route alohida bundle, kerakligini yuklaydi.
7. **Middleware** — so'rov serverga yetmasdan oldin (auth, redirect, geo).
8. **Caching** — `fetch` cache, ISR, CDN integratsiyasi built-in.
9. **Server Components** — bundle kichik, ma'lumot serverga yaqin.

**Qachon oddiy React (Vite) yetarli:**
- SEO kerak emas (login orqasidagi **dashboard**, admin panel, ichki asbob).
- To'liq client-side interaktiv **SPA** (masalan Figma-ga o'xshash editor).
- Statik hosting va oddiy sozlash yetarli.

**Qachon Next.js kerak:**
- **SEO muhim** (blog, marketing, e-commerce, ommaviy kontent).
- **Server-side rendering** yoki personalizatsiya kerak.
- **Full-stack** — frontend + backend bitta joyda.
- Rasm/shrift/performance optimizatsiyasi jiddiy talab.

**⚠️ Ehtiyot bo'l:** "Next.js har doim to'g'ri tanlov" deb ayting**ma**. Senior javob — trade-off bilan: "Bu loyiha SEO talab qilmaydigan ichki dashboard bo'lgani uchun Vite yetarli; Next.js qo'shimcha murakkablik keltiradi." Kontekstga qarab tanlash — senior belgisidir.

---

## next/image, next/link, metadata

**💡 Tushuncha:** Next.js SEO va performance uchun maxsus komponentlar beradi. Bularni ishlatish — "Next.js'ni to'g'ri ishlatyapman" belgisidir.

**`next/image`** — rasmlarni avtomatik optimallashtiradi:

```tsx
import Image from "next/image";

<Image
  src="/hero.jpg"
  alt="Bosh rasm"
  width={800}
  height={600}
  priority // LCP rasm uchun — darhol yukla
/>;
```

Nima qiladi: kerakli o'lchamga resize, zamonaviy format (WebP/AVIF), lazy load (ekranga kelganda), va `width`/`height` orqali **layout shift** (CLS)'ni oldini oladi.

**`next/link`** — client-side navigatsiya (sahifa to'liq qayta yuklanmaydi) + **prefetch** (link ekranga kirsa, keyingi sahifani oldindan yuklaydi):

```tsx
import Link from "next/link";

<Link href="/blog/salom">Blogga o'tish</Link>;
```

**`metadata`** — SEO meta teglari (App Router):

```tsx
// app/blog/[slug]/page.tsx
import type { Metadata } from "next";

// Statik metadata
export const metadata: Metadata = {
  title: "Blogim",
  description: "Dasturlash haqida",
};

// Yoki dinamik — ma'lumotga qarab
export async function generateMetadata({
  params,
}: {
  params: Promise<{ slug: string }>;
}): Promise<Metadata> {
  const { slug } = await params;
  const post = await getPost(slug);
  return {
    title: post.title,
    openGraph: { images: [post.cover] },
  };
}
```

**⚠️ Ehtiyot bo'l:** `metadata` faqat Server Component'da ishlaydi. Agar sahifada `"use client"` bo'lsa, `metadata` eksport qilib bo'lmaydi — meta teglarni yuqoridagi server layout/page'ga ko'chir.

---

## API routes va Route Handlers

**💡 Tushuncha:** Next.js full-stack — backend endpoint'larni o'sha loyihada yozasan. App Router'da bu **Route Handlers** (`route.ts`), Pages Router'da **API Routes** (`pages/api/`) deyiladi.

```ts
// app/api/users/route.ts
import { NextRequest, NextResponse } from "next/server";

export async function GET(req: NextRequest) {
  const users = await db.user.findMany();
  return NextResponse.json(users);
}

export async function POST(req: NextRequest) {
  const body = await req.json();
  const user = await db.user.create({ data: body });
  return NextResponse.json(user, { status: 201 });
}
```

Fayl `app/api/users/route.ts` → `GET /api/users`, `POST /api/users`. Har HTTP metod uchun alohida eksport funksiya.

**⚠️ Ehtiyot bo'l:** App Router'da ko'p hollarda ma'lumot **olish** uchun API route kerak EMAS — server component to'g'ridan-to'g'ri DB'ga boradi. Route Handler'lar ko'proq **tashqi client'lar** (mobil app, webhook), yoki client component'dan chaqiriladigan **mutatsiyalar** uchun (yoki Server Actions ishlat).

---

## Middleware va environment variables

**💡 Tushuncha:** **Middleware** — har so'rov serverga (yoki sahifaga) yetib bormasdan **oldin** ishlaydigan kod. Auth tekshirish, redirect, geo-lokatsiya, A/B test uchun ideal. Edge'da ishlaydi (tez).

```ts
// middleware.ts (loyiha ildizida)
import { NextRequest, NextResponse } from "next/server";

export function middleware(req: NextRequest) {
  const token = req.cookies.get("session")?.value;
  if (!token && req.nextUrl.pathname.startsWith("/dashboard")) {
    return NextResponse.redirect(new URL("/login", req.url));
  }
  return NextResponse.next();
}

export const config = {
  matcher: ["/dashboard/:path*"], // faqat shu yo'llarda ishlaydi
};
```

**Environment variables** — maxfiy sozlamalar:

```
# .env.local
DATABASE_URL=postgres://...          # faqat serverda (maxfiy)
NEXT_PUBLIC_API_URL=https://api.x.com # brauzerga ham chiqadi
```

Qoida: **`NEXT_PUBLIC_` prefiksi bo'lgan o'zgaruvchilargina brauzerga yuboriladi.** Prefikssiz o'zgaruvchilar faqat serverda (server component, route handler, middleware) mavjud.

**⚠️ Ehtiyot bo'l:** API kaliti yoki DB parolini HECH QACHON `NEXT_PUBLIC_` bilan boshlama — u brauzer bundle'iga tushib, hammaga oshkor bo'ladi. Maxfiy narsalarni faqat server tomonda ishlat.

---

## Savol-javoblar

### ❓ Next.js nima va u oddiy React'dan nimasi bilan farq qiladi?

**✅ Javob:** React — UI kutubxonasi (faqat komponentlarni render qiladi). Next.js — React atrofidagi to'liq framework ("batteries included"): routing, SSR/SSG/ISR, image/font optimization, API routes, middleware, caching — hammasi ichida. Oddiy React (Vite)'da bularni o'zing yig'asan; Next.js'da standart yechim tayyor. Asosiy qo'shimcha — server-side rendering va SEO, hamda full-stack imkoniyati.

### ❓ CSR, SSR, SSG, ISR — farqini bir gapda tushuntir.

**✅ Javob:** Farq — **HTML qayerda/qachon yasaladi**. CSR — brauzerda (JS ishga tushgach). SSR — serverda, har so'rovda. SSG — build vaqtida bir marta. ISR — SSG, lekin fon rejimida vaqti-vaqti bilan yangilanadi. Tezlik bo'yicha: SSG ≈ ISR > SSR > CSR; yangilik bo'yicha: SSR = CSR > ISR > SSG.

### ❓ Bir sahifa uchun rendering strategiyasini qanday tanlaysan?

**✅ Javob:** Ikki savol: (1) SEO kerakmi? (2) Ma'lumot qanchalik tez o'zgaradi va u hamma uchun bir xilmi? Login orqasi, SEO kerak emas → CSR. Hamma uchun bir xil, kam o'zgaradi → SSG. Muntazam o'zgaradi → ISR. Har so'rovda yangi/personal → SSR. App Router'da buni `fetch` cache sozlamasi orqali beraman.

### ❓ Hydration nima va nega mismatch xatosi chiqadi?

**✅ Javob:** Hydration — server yuborgan quruq HTML'ga brauzerdagi React interaktivlik (event, state) ulash jarayoni. Server tayyor HTML beradi, JS yuklanib uni "jonlantiradi". Mismatch — server va client turli HTML yasaganda chiqadi: `Date.now()`, `Math.random()`, `window`, yoki `localStorage`'ga bog'liq render. Yechim: bunday narsalarni `useEffect` ichida, mount'dan keyin (faqat client'da) ko'rsatish.

### ❓ App Router va Pages Router farqi nimada? Qaysi birini tanlaysan?

**✅ Javob:** Pages Router (`pages/`) — eski, `getServerSideProps`/`getStaticProps` bilan. App Router (`app/`) — yangi (Next 13+), React Server Components asosida, `layout.tsx`/`loading.tsx`/`error.tsx` nested fayllari bilan. 2026-da yangi loyiha uchun App Router tanlayman: server components bundle'ni kichraytiradi, data fetching soddaroq, streaming built-in.

### ❓ React Server Component nima? Client Component'dan farqi?

**✅ Javob:** Server Component (App Router'da default) faqat serverda ishlaydi, kodi brauzerga yuborilmaydi — u `await fetch`/DB qila oladi, maxfiy kalit ishlata oladi, lekin `useState`/`onClick`/`window` ishlata olmaydi. Client Component (`"use client"`) interaktivlik uchun — state, event, brauzer API bor, lekin kodi bundle'ga kiradi. Server Component'lar bilan bundle kichik qoladi va ma'lumot serverga yaqin bo'ladi.

### ❓ `"use client"` aniq nimani anglatadi?

**✅ Javob:** U "shu fayl va undan pastda import qilingan hamma narsa client bundle'iga kiradi" degan **chegara** belgilaydi. Bu "faqat brauzerda ishlaydi" degani EMAS — client component ham SSR paytida serverda bir marta HTML uchun render bo'ladi, keyin brauzerda hydrate bo'ladi. Ya'ni `"use client"` — server/client chegarasi, "server-siz" emas.

### ❓ App Router'da SSR, SSG, ISR'ni qanday farqlaysan (kod darajasida)?

**✅ Javob:** `fetch` cache sozlamasi orqali. `cache: "no-store"` yoki `cookies()`/`headers()` ishlatish → dinamik = **SSR**. Oddiy default `fetch` → **SSG**. `next: { revalidate: N }` → **ISR**. Ya'ni alohida rejim tanlamayman — Next.js sozlamalardan avtomatik aniqlaydi.

### ❓ `next/image` nima foyda beradi?

**✅ Javob:** Avtomatik: kerakli o'lchamga resize (turli qurilma uchun), zamonaviy format (WebP/AVIF), lazy loading (ekranga kelganda), va `width`/`height` orqali layout shift (CLS)ni oldini olish. LCP rasmiga `priority` beraman. Bu Core Web Vitals'ni sezilarli yaxshilaydi.

### ❓ App Router'da `loading.tsx` va `error.tsx` nima qiladi?

**✅ Javob:** `loading.tsx` — sahifa (server component) ma'lumot kutayotganda avtomatik `<Suspense>` fallback sifatida ko'rsatiladi (streaming — foydalanuvchi darhol UI ko'radi). `error.tsx` — o'sha segmentda xato yuz berganda avtomatik error boundary sifatida ishlaydi (client component bo'lishi shart, chunki `reset` funksiyasi bor). Ikkalasi ham qo'lda Suspense/ErrorBoundary yozishni almashtiradi.

### ❓ Middleware qachon va nima uchun ishlatiladi?

**✅ Javob:** Middleware — so'rov sahifaga yetib bormasdan **oldin** ishlaydi (Edge'da). Asosiy holatlar: auth tekshirish (token yo'q bo'lsa `/login`ga redirect), geo/locale aniqlash, A/B test, bot bloklash. `matcher` bilan qaysi yo'llarda ishlashini cheklayman. Og'ir ish (DB) qilmaslik kerak — u tez va yengil bo'lishi lozim.

### ❓ Environment variable'da `NEXT_PUBLIC_` prefiksi nima uchun?

**✅ Javob:** Faqat `NEXT_PUBLIC_` bilan boshlangan o'zgaruvchilar brauzer bundle'iga qo'shiladi va client'da mavjud bo'ladi. Prefikssizlari faqat serverda (server component, route handler, middleware). Shu sababli API kaliti, DB paroli kabi maxfiy narsalarni HECH QACHON `NEXT_PUBLIC_` bilan yozmaslik kerak — aks holda ular hammaga oshkor bo'ladi.

### ❓ App Router'da ma'lumotni qanday olasan? `getServerSideProps` kerakmi?

**✅ Javob:** App Router'da `getServerSideProps` yo'q va kerak emas. Ma'lumotni to'g'ridan-to'g'ri async server component ichida `await fetch(...)` bilan olaman. Bir-biriga bog'liq bo'lmagan so'rovlarni `Promise.all` bilan parallel qilaman (waterfall'dan qochish uchun). `getServerSideProps`/`getStaticProps` — Pages Router'ga tegishli, endi eski.

### ❓ Client component'da ma'lumotni qanday olasan (server component o'rniga)?

**✅ Javob:** Client component'da (`"use client"`) server-side `fetch` cache ishlamaydi, shuning uchun `useEffect` + `fetch` yoki, yaxshirog'i, **TanStack Query / SWR** ishlataman — ular cache, retry, revalidation beradi. Lekin imkon bo'lsa ma'lumotni yuqoridagi server component'da olib, client'ga prop sifatida uzataman — bu tezroq va SEO'ga foydali.

### ❓ Qachon oddiy React (Vite) yetarli, qachon Next.js kerak?

**✅ Javob:** SEO kerak bo'lmagan, login orqasidagi to'liq interaktiv dashboard/admin panel/SPA uchun — Vite yetarli va soddaroq. SEO muhim (blog, marketing, e-commerce), server-side rendering yoki personalizatsiya kerak, yoki full-stack (frontend+backend bir joyda) bo'lsa — Next.js. Tanlovni kontekst va trade-off bilan asoslash — senior belgisi.

### ❓ Server Component ichida Client Component'ni, va aksincha, ishlatish mumkinmi?

**✅ Javob:** Server Component ichida Client Component'ni to'g'ridan-to'g'ri import qilib ishlatish mumkin (odatiy naqsh). Lekin Client Component ichiga Server Component'ni import qilib bo'lmaydi — server component'ni `children` (yoki boshqa prop) sifatida uzatish kerak. Sabab: client bundle server kodini o'z ichiga ololmaydi.

### ❓ ISR'da `revalidate` qanday ishlaydi? Foydalanuvchi kutadimi?

**✅ Javob:** Yo'q, kutmaydi. Bu "stale-while-revalidate": `revalidate: 60` bo'lsa, 60s o'tgach kelgan birinchi foydalanuvchi ham **eski** (stale) sahifani darhol oladi, shu bilan birga Next.js fon rejimida yangi HTML yasaydi. Keyingi foydalanuvchi yangisini oladi. Hodisaga qarab yangilash kerak bo'lsa — `revalidatePath`/`revalidateTag` (on-demand) ishlataman.

---

## Masalalar

> Yechimlar: [solutions/frontend/10-nextjs.md](../solutions/frontend/10-nextjs.md)

1. **Rendering tanlovi.** Quyidagi sahifalar uchun eng mos rendering strategiyasini (CSR/SSR/SSG/ISR) tanlang va **nega** ekanini bir jumlada asoslang: (a) shaxsiy dashboard, (b) kompaniya blog posti, (c) har 10 daqiqada yangilanadigan valyuta kurslari sahifasi, (d) e-commerce mahsulot sahifasi (narx tez-tez o'zgaradi, SEO kerak).

2. **Hydration mismatch tuzatish.** Quyidagi komponent hydration mismatch xatosini beradi. Sababini ayting va App Router'ga mos tuzating:
   ```tsx
   export default function Greeting() {
     const hour = new Date().getHours();
     return <p>{hour < 12 ? "Xayrli tong" : "Xayrli kun"}</p>;
   }
   ```

3. **Blog dinamik route.** App Router'da `/blog/[slug]` sahifasini yozing: ma'lumotni ISR bilan (har 300s) oling, hamda `generateMetadata` orqali post sarlavhasini `<title>`ga chiqaring.

4. **Server vs Client bo'lish.** Bitta "mahsulot kartasi" komponentini to'g'ri Server/Client qismlarga ajrating: mahsulot ma'lumoti serverda olinsin, "Savatga qo'shish" tugmasi interaktiv (client) bo'lsin. Ikkala faylni yozing.

5. **Auth middleware.** `/admin/*` yo'llarini himoya qiluvchi `middleware.ts` yozing: `session` cookie bo'lmasa `/login`ga redirect qilsin, faqat `/admin` ostidagi yo'llarda ishlasin.

6. **Route Handler CRUD.** `app/api/todos/route.ts` da `GET` (ro'yxat) va `POST` (yangi qo'shish) endpoint'larini yozing. `POST` `201` status qaytarsin.

7. **Parallel data fetching.** Bir sahifada foydalanuvchi profili va uning postlari ikki alohida endpoint'dan keladi. Waterfall bo'lmasligi uchun ularni to'g'ri (parallel) oling.

8. **Env variable xatosi.** Bir dasturchi `.env.local` da `NEXT_PUBLIC_STRIPE_SECRET=sk_live_...` deb yozgan va uni server route'da ishlatgan. Bu yerda qanday xavfsizlik muammosi bor va qanday tuzatish kerak?

---

← [Frontend bo'limiga qaytish](./README.md)
