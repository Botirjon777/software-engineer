# Next.js — Yechimlar

Bu fayl [`frontend/10-nextjs.md`](../../frontend/10-nextjs.md) dagi masalalar yechimlari. Har bir yechimda **javob + kod + nega shunday** bor. Intervyuda yechimni ovoz chiqarib, trade-off bilan bayon qiling.

## Mundarija

- [1. Rendering tanlovi](#1-rendering-tanlovi)
- [2. Hydration mismatch tuzatish](#2-hydration-mismatch-tuzatish)
- [3. Blog dinamik route (ISR + metadata)](#3-blog-dinamik-route-isr--metadata)
- [4. Server vs Client bo'lish](#4-server-vs-client-bolish)
- [5. Auth middleware](#5-auth-middleware)
- [6. Route Handler CRUD](#6-route-handler-crud)
- [7. Parallel data fetching](#7-parallel-data-fetching)
- [8. Env variable xatosi](#8-env-variable-xatosi)

---

## 1. Rendering tanlovi

**(a) Shaxsiy dashboard → CSR (yoki SSR agar SEO kerak bo'lmasa ham personalizatsiya bilan).**
Odatda **CSR**: login orqasida, SEO kerak emas, hamma ma'lumot foydalanuvchiga xos va interaktiv. Toza React SPA yetarli.

**(b) Kompaniya blog posti → SSG.**
Hamma uchun bir xil, kamdan-kam o'zgaradi, SEO muhim. Build vaqtida bir marta yasab, CDN'dan tarqatish eng tez va arzon.

**(c) Har 10 daqiqada yangilanadigan valyuta kurslari → ISR (`revalidate: 600`).**
Hamma uchun bir xil, muntazam lekin tez-tez emas o'zgaradi. SSG tezligi + fon yangilanishi ideal. SSR bu yerda ortiqcha server yuki bo'lardi.

**(d) E-commerce mahsulot sahifasi (narx tez-tez, SEO kerak) → SSR yoki qisqa `revalidate` bilan ISR.**
SEO kerak (SSG/SSR/ISR mumkin), lekin narx tez o'zgaradi. Agar narx sekundlik aniqlikda bo'lishi shart bo'lsa → **SSR** (`cache: "no-store"`). Agar bir necha soniya eskilik qabul qilinsa → qisqa `revalidate` (masalan 30s) bilan **ISR** — bu server yukini kamaytiradi.

**Nega:** Ikki savol yechadi — SEO kerakmi (kerak bo'lsa CSR chiqib ketadi) va ma'lumot qanchalik tez o'zgaradi (tez → SSR, muntazam → ISR, kam → SSG).

---

## 2. Hydration mismatch tuzatish

**Sabab:** `new Date().getHours()` server va client'da **turli** natija berishi mumkin (server timezone yoki so'rov vaqti ≠ brauzer vaqti). Server bir HTML ("Xayrli tong"), client boshqa ("Xayrli kun") yasaydi → **hydration mismatch**.

**Yechim:** Vaqtga bog'liq qismni faqat client'da, `mount`'dan keyin ko'rsatish.

```tsx
"use client";

import { useEffect, useState } from "react";

export default function Greeting() {
  const [greeting, setGreeting] = useState<string | null>(null);

  useEffect(() => {
    const hour = new Date().getHours();
    setGreeting(hour < 12 ? "Xayrli tong" : "Xayrli kun");
  }, []);

  // Birinchi server render va birinchi client render bir xil (null) — mismatch yo'q
  return <p>{greeting ?? "Salom"}</p>;
}
```

**Muqobil:** Agar salomlashish serverda aniqlanishi kerak bo'lsa (SEO uchun), timezone'ni aniq belgilab, server component'da hisoblang va prop sifatida uzating — shunda mismatch bo'lmaydi.

---

## 3. Blog dinamik route (ISR + metadata)

```tsx
// app/blog/[slug]/page.tsx
import type { Metadata } from "next";
import { notFound } from "next/navigation";

type Post = { slug: string; title: string; body: string };

async function getPost(slug: string): Promise<Post | null> {
  const res = await fetch(`https://api.example.com/posts/${slug}`, {
    next: { revalidate: 300 }, // ISR — har 300 soniyada yangilash
  });
  if (!res.ok) return null;
  return res.json();
}

// Dinamik metadata — post sarlavhasini <title>ga chiqaradi
export async function generateMetadata({
  params,
}: {
  params: Promise<{ slug: string }>;
}): Promise<Metadata> {
  const { slug } = await params;
  const post = await getPost(slug);
  if (!post) return { title: "Topilmadi" };
  return {
    title: post.title,
    description: post.body.slice(0, 150),
  };
}

export default async function BlogPost({
  params,
}: {
  params: Promise<{ slug: string }>;
}) {
  const { slug } = await params;
  const post = await getPost(slug);
  if (!post) notFound();

  return (
    <article>
      <h1>{post.title}</h1>
      <p>{post.body}</p>
    </article>
  );
}
```

**Nega:** `next: { revalidate: 300 }` sahifani statik qiladi, lekin 5 daqiqada bir fon rejimida yangilaydi (ISR). `generateMetadata` ham o'sha `getPost`'ni chaqiradi — Next.js `fetch`'ni deduplikatsiya qilgani uchun ikkinchi so'rov yuborilmaydi.

---

## 4. Server vs Client bo'lish

```tsx
// app/products/[id]/ProductCard.tsx — SERVER Component (default)
import { AddToCartButton } from "./AddToCartButton";

type Product = { id: string; name: string; price: number };

export async function ProductCard({ id }: { id: string }) {
  // Ma'lumot serverda olinadi — bundle'ga fetch kodi tushmaydi, SEO'ga foydali
  const res = await fetch(`https://api.example.com/products/${id}`);
  const product: Product = await res.json();

  return (
    <div className="card">
      <h2>{product.name}</h2>
      <p>{product.price} so'm</p>
      {/* Faqat interaktiv qism client */}
      <AddToCartButton productId={product.id} />
    </div>
  );
}
```

```tsx
// app/products/[id]/AddToCartButton.tsx — CLIENT Component
"use client";

import { useState } from "react";

export function AddToCartButton({ productId }: { productId: string }) {
  const [added, setAdded] = useState(false);

  async function addToCart() {
    await fetch("/api/cart", {
      method: "POST",
      body: JSON.stringify({ productId }),
    });
    setAdded(true);
  }

  return (
    <button onClick={addToCart} disabled={added}>
      {added ? "Qo'shildi ✓" : "Savatga qo'shish"}
    </button>
  );
}
```

**Nega:** Ma'lumot (statik, SEO kerak) — server component'da. Faqat `useState`/`onClick` talab qiladigan tugma — alohida client component. Bu bundle'ni minimal saqlaydi: karta HTML'i serverda tayyorlanadi, brauzerga faqat tugma logikasi yuboriladi.

---

## 5. Auth middleware

```ts
// middleware.ts  (loyiha ildizida, app/ yonida)
import { NextRequest, NextResponse } from "next/server";

export function middleware(req: NextRequest) {
  const session = req.cookies.get("session")?.value;

  if (!session) {
    const loginUrl = new URL("/login", req.url);
    loginUrl.searchParams.set("from", req.nextUrl.pathname); // qaytish uchun
    return NextResponse.redirect(loginUrl);
  }

  return NextResponse.next();
}

export const config = {
  matcher: ["/admin/:path*"], // faqat /admin ostidagi yo'llar
};
```

**Nega:** `matcher` middleware'ni faqat `/admin/*` da ishga tushiradi — qolgan yo'llar tegilmaydi (performance). `session` cookie bo'lmasa `/login`ga redirect, `from` bilan qaytish manzilini saqlaymiz. Middleware Edge'da ishlaydi — tez va yengil, DB so'rovi qilmaymiz (faqat cookie mavjudligini tekshiramiz).

**⚠️ Eslatma:** Middleware faqat cookie **mavjudligini** tekshiradi. Token'ning haqiqatan yaroqli ekanini (imzo, muddat) sahifa/route handler ichida yoki tokenning o'zini tekshirib qo'shimcha tasdiqlash kerak — middleware'da og'ir kripto ish qilinmaydi.

---

## 6. Route Handler CRUD

```ts
// app/api/todos/route.ts
import { NextRequest, NextResponse } from "next/server";

type Todo = { id: string; title: string; done: boolean };

// Soddalik uchun xotirada; real loyihada DB
const todos: Todo[] = [];

export async function GET() {
  return NextResponse.json(todos);
}

export async function POST(req: NextRequest) {
  const body = (await req.json()) as { title?: string };

  if (!body.title) {
    return NextResponse.json({ error: "title majburiy" }, { status: 400 });
  }

  const todo: Todo = {
    id: crypto.randomUUID(),
    title: body.title,
    done: false,
  };
  todos.push(todo);

  return NextResponse.json(todo, { status: 201 }); // 201 Created
}
```

**Nega:** `GET /api/todos` ro'yxatni, `POST /api/todos` yangi todo qo'shadi va `201` qaytaradi. Validatsiya (bo'sh `title`) `400` beradi. Har HTTP metod alohida eksport funksiya — Next.js fayl nomidan (`route.ts`) endpoint yasaydi.

---

## 7. Parallel data fetching

```tsx
// app/user/[id]/page.tsx
type User = { id: string; name: string };
type Post = { id: string; title: string };

async function getUser(id: string): Promise<User> {
  return fetch(`https://api.example.com/users/${id}`).then((r) => r.json());
}
async function getPosts(id: string): Promise<Post[]> {
  return fetch(`https://api.example.com/users/${id}/posts`).then((r) =>
    r.json()
  );
}

export default async function UserPage({
  params,
}: {
  params: Promise<{ id: string }>;
}) {
  const { id } = await params;

  // TO'G'RI — parallel: ikkala so'rov bir vaqtda ketadi
  const [user, posts] = await Promise.all([getUser(id), getPosts(id)]);

  return (
    <div>
      <h1>{user.name}</h1>
      <ul>
        {posts.map((p) => (
          <li key={p.id}>{p.title}</li>
        ))}
      </ul>
    </div>
  );
}
```

**Nega — waterfall'dan qochish:**

```tsx
// XATO — sekvensial (waterfall): posts user tugagach boshlanadi
const user = await getUser(id);   // kutamiz...
const posts = await getPosts(id); // endi boshlanadi — 2x sekin
```

`getUser` va `getPosts` bir-biriga bog'liq emas, shuning uchun `Promise.all` bilan **parallel** yuboramiz — umumiy vaqt ikkitasining eng uzunига teng, yig'indisiga emas.

---

## 8. Env variable xatosi

**Muammo:** `NEXT_PUBLIC_STRIPE_SECRET` — `NEXT_PUBLIC_` prefiksi bor. Bu prefiks o'zgaruvchini **brauzer bundle'iga** qo'shadi, ya'ni Stripe **maxfiy kaliti (`sk_live_...`) har bir tashrifchiga oshkor bo'ladi.** Buni ko'rgan istalgan odam hisob orqali to'lov qila oladi — jiddiy xavfsizlik teshigi.

**Tuzatish:** Maxfiy kalitdan `NEXT_PUBLIC_` prefiksini olib tashlash — u faqat serverda qolsin:

```
# .env.local
STRIPE_SECRET_KEY=sk_live_...        # prefikssiz → faqat serverda
NEXT_PUBLIC_STRIPE_PUBLISHABLE=pk_live_...  # public kalit — bu brauzerga OK
```

```ts
// app/api/checkout/route.ts — server, maxfiy kalit shu yerda
import Stripe from "stripe";

const stripe = new Stripe(process.env.STRIPE_SECRET_KEY!); // faqat serverda mavjud
```

**Qoida:** `NEXT_PUBLIC_` faqat oshkor bo'lishi mumkin bo'lgan qiymatlar uchun (public API kaliti, tahlil ID'si). Maxfiy kalit, DB parol, secret token — HECH QACHON `NEXT_PUBLIC_` bilan boshlanmaydi va faqat server tomonda (route handler, server component, middleware) ishlatiladi.

---

← [Frontend bo'limiga qaytish](../../frontend/README.md)
