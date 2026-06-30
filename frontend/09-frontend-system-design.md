# Frontend System Design

Frontend system design intervyusi — bu "tugmani qanday yasaysiz" emas, balki "millionlab foydalanuvchi uchun ishlaydigan, tez, barqaror va kengaytiriladigan UI tizimini qanday loyihalashtirasiz" degan savol. Senior darajada sizdan trade-off'larni ovoz chiqarib o'ylash, talablarni aniqlashtirish va arxitektura qarorlarini asoslab berish kutiladi. Bu hujjat o'sha intervyuga to'liq tayyorgarlikdir.

## Mundarija

- [Intervyuga umumiy yondashuv (RADIO)](#intervyuga-umumiy-yondashuv-radio)
- [1. Requirements: talablarni aniqlashtirish](#1-requirements-talablarni-aniqlashtirish)
- [2. API dizayni](#2-api-dizayni)
- [3. Data model](#3-data-model)
- [4. Komponent arxitekturasi](#4-komponent-arxitekturasi)
- [5. State boshqaruvi](#5-state-boshqaruvi)
- [Rendering strategiyalari: CSR vs SSR vs SSG vs ISR vs Streaming](#rendering-strategiyalari-csr-vs-ssr-vs-ssg-vs-isr-vs-streaming)
- [Hydration va uning muammolari](#hydration-va-uning-muammolari)
- [Next.js vs Remix: qachon nima](#nextjs-vs-remix-qachon-nima)
- [Performance budget va Core Web Vitals](#performance-budget-va-core-web-vitals)
- [Caching: HTTP cache, CDN, Service Worker](#caching-http-cache-cdn-service-worker)
- [Networking: REST vs GraphQL vs WebSocket vs SSE](#networking-rest-vs-graphql-vs-websocket-vs-sse)
- [Optimistic update, pagination, infinite scroll](#optimistic-update-pagination-infinite-scroll)
- [Offline va PWA](#offline-va-pwa)
- [Accessibility katta hajmda](#accessibility-katta-hajmda)
- [i18n: xalqarolashtirish](#i18n-xalqarolashtirish)
- [Xavfsizlik: XSS, CSRF, CSP](#xavfsizlik-xss-csrf-csp)
- [Rasm optimizatsiyasi](#rasm-optimizatsiyasi)
- [Micro-frontend](#micro-frontend)
- [Build tooling: Vite, code splitting](#build-tooling-vite-code-splitting)
- [Monitoring: error tracking va RUM](#monitoring-error-tracking-va-rum)
- [Ishlangan dizayn 1: News Feed / Infinite Scroll](#ishlangan-dizayn-1-news-feed--infinite-scroll)
- [Ishlangan dizayn 2: Autocomplete / Typeahead](#ishlangan-dizayn-2-autocomplete--typeahead)
- [Ishlangan dizayn 3: Real-time Chat UI](#ishlangan-dizayn-3-real-time-chat-ui)
- [Savol-javoblar](#savol-javoblar)
- [Masalalar](#masalalar)

---

## Intervyuga umumiy yondashuv (RADIO)

**💡 Tushuncha:** Frontend system design intervyusi 45–60 daqiqa davom etadi va u kod yozish emas, balki *muloqot* va *qaror qabul qilish*ni tekshiradi. Tartibsiz boshlash eng ko'p uchraydigan xato. Shuning uchun **RADIO** freymvorkidan foydalaning:

- **R — Requirements** (talablar): funksional va non-funksional talablarni aniqlashtiring.
- **A — Architecture / API** (arxitektura): yuqori darajadagi komponentlar va ular orasidagi ma'lumot oqimi.
- **D — Data model**: client'da qanday ma'lumotlar saqlanadi, ular qayerdan keladi.
- **I — Interface / API**: client va server orasidagi shartnoma (REST/GraphQL endpoint'lar).
- **O — Optimizations**: performance, accessibility, edge case'lar, xavfsizlik.

**⚠️ Ehtiyot bo'l:** Vaqtning 50% ini birinchi 10 daqiqaga sarflab qo'ymang. Taxminan: Requirements 5–7 daqiqa, Architecture 10 daqiqa, Data/API 10 daqiqa, qolgan barchasi (deep dive + optimizations) 20–25 daqiqa. Intervyuer odatda bitta sohaga (masalan, "performance haqida gapiring") chuqurroq kirishni so'raydi — shunga tayyor bo'ling.

Eng muhim qoida: **ovoz chiqarib o'ylang va trade-off'larni ayting**. "Men SSR ishlataman" emas, balki "SEO muhim bo'lgani uchun SSR ishlataman, lekin bu server xarajatini oshiradi va TTFB'ni biroz sekinlashtiradi — buni CDN cache bilan yumshataman" deng.

---

## 1. Requirements: talablarni aniqlashtirish

**💡 Tushuncha:** Talablarni ikki turga ajrating. **Funksional** — tizim nima qiladi (feed'da post ko'rsatadi, like bosadi). **Non-funksional** — tizim qanchalik yaxshi qiladi (tez, qulay, ishonchli). Senior'lar non-funksional talablarni unutmaydi.

So'raladigan savollar:

- **Platforma:** faqat web'mi, yoki mobile web ham? Responsive kerakmi?
- **Foydalanuvchilar soni:** kuniga necha million? Bu pagination va caching qarorlariga ta'sir qiladi.
- **Real-time kerakmi?** Chat — ha; blog — yo'q.
- **Offline ishlashi kerakmi?**
- **SEO muhimmi?** Bu rendering strategiyasini hal qiladi.
- **i18n / RTL** (o'ngdan-chapga) kerakmi?
- **Brauzer qo'llab-quvvatlash:** faqat zamonaviymi yoki IE11 ham?
- **Accessibility** darajasi (WCAG AA majburiymi)?

```ts
// Talablarni doimo yozib boring — intervyuer ko'rishi uchun
interface Requirements {
  functional: string[];   // "Foydalanuvchi feed'ni cheksiz scroll qila oladi"
  nonFunctional: {
    performance: string;   // "LCP < 2.5s, INP < 200ms"
    accessibility: string; // "WCAG 2.1 AA"
    devices: string[];     // ["mobile", "desktop"]
    offline: boolean;
    i18n: string[];        // ["uz", "en", "ru"]
  };
}
```

**⚠️ Ehtiyot bo'l:** Hech qachon talablarni faraz qilib o'tib ketmang. "Faraz qilaman SEO kerak" deyish o'rniga so'rang. Noto'g'ri faraz butun dizaynni buzadi.

---

## 2. API dizayni

**💡 Tushuncha:** Frontend dizaynda API — bu client va server orasidagi *shartnoma*. Siz buni to'liq belgilashingiz shart emas, lekin asosiy endpoint'lar, request/response shaklini ko'rsating. Bu sizning ma'lumot oqimini tushunishingizni isbotlaydi.

```ts
// REST misol: news feed
// GET /api/v1/feed?cursor=<id>&limit=20
interface FeedResponse {
  items: Post[];
  nextCursor: string | null; // null bo'lsa — oxiriga yetdik
}

interface Post {
  id: string;
  authorId: string;
  text: string;
  media: Media[];
  likeCount: number;
  likedByMe: boolean;
  createdAt: string; // ISO 8601
}
```

Muhokama qiladigan nuqtalar:

- **Cursor-based vs offset pagination** (pastda batafsil).
- **Field selection** — GraphQL bilan client kerakli maydonlarni so'raydi, REST'da over-fetching bo'ladi.
- **Versiyalash** — `/api/v1/` URL'da yoki `Accept` header'da.
- **Xato formati** — barqaror error shakli (`{ code, message }`).
- **Rate limiting** — 429 javobini client qanday boshqaradi.

---

## 3. Data model

**💡 Tushuncha:** Client-side data model — bu sizning state'ingizning shakli. Eng katta xato: server javobini to'g'ridan-to'g'ri ichma-ich (nested) saqlash. To'g'ri yondashuv — **normalizatsiya**: ma'lumotlarni ID bo'yicha kalitlangan jadvallarga ajratish (xuddi relyatsion DB kabi).

```ts
// ❌ Yomon: nested, dublikat, yangilash qiyin
interface BadState {
  feed: Array<{ post: Post; author: User }>; // author har postda takrorlanadi
}

// ✅ Yaxshi: normalizatsiya qilingan
interface GoodState {
  posts: Record<string, Post>;       // postId -> Post
  users: Record<string, User>;       // userId -> User
  feedOrder: string[];               // faqat ID'lar tartibi
}
```

Normalizatsiyaning foydasi: bitta user ma'lumoti o'zgarsa (masalan, avatar), uni faqat bitta joyda yangilaysiz va hamma post avtomatik yangilanadi. RTK Query, Apollo Client, Relay buni avtomatik qiladi.

**⚠️ Ehtiyot bo'l:** Normalizatsiya hamma joyda kerak emas. Kichik, bir martalik ma'lumot uchun u ortiqcha murakkablik. Faqat bir xil entity ko'p joyda takrorlansa qo'llang.

---

## 4. Komponent arxitekturasi

**💡 Tushuncha:** Yuqori darajadagi komponent daraxtini chizing. Intervyuer ma'lumot qaysi yo'nalishda oqishini ko'rishni xohlaydi. Komponentlarni mas'uliyat bo'yicha qatlamlarga ajrating.

```
<App>
 ├─ <FeedPage>                  // route, data fetching koordinatsiyasi
 │   ├─ <FeedList>              // virtualizatsiya, scroll boshqaruvi
 │   │   └─ <PostCard>          // bitta postni ko'rsatish (presentational)
 │   │       ├─ <PostHeader>    // avatar, ism, vaqt
 │   │       ├─ <PostBody>      // matn, media
 │   │       └─ <PostActions>   // like, comment, share
 │   └─ <NewPostsBanner>        // "5 ta yangi post" tugmasi
 └─ <Toast>                     // global bildirishnomalar
```

Qatlamlar:

- **Container / route komponentlar** — data fetching, navigatsiya.
- **Smart / feature komponentlar** — biznes logikasi, lokal state.
- **Presentational / UI komponentlar** — faqat props qabul qiladi, qayta ishlatiladi (design system).

```tsx
// Presentational komponent — testlash oson, qayta ishlatiladi
interface PostCardProps {
  post: Post;
  onLike: (id: string) => void;
}

function PostCard({ post, onLike }: PostCardProps) {
  return (
    <article aria-labelledby={`post-${post.id}-author`}>
      <PostHeader authorId={post.authorId} />
      <PostBody text={post.text} media={post.media} />
      <PostActions
        likeCount={post.likeCount}
        liked={post.likedByMe}
        onLike={() => onLike(post.id)}
      />
    </article>
  );
}
```

---

## 5. State boshqaruvi

**💡 Tushuncha:** State'ni to'rt kategoriyaga ajrating va har biriga to'g'ri vositani tanlang. Eng ko'p uchraydigan senior xatosi — hamma narsani global store'ga (Redux) tiqish.

| Kategoriya | Misol | Vosita |
|-----------|-------|--------|
| **Server state** | feed, user profili | TanStack Query / RTK Query / SWR |
| **Global UI state** | tema, til, auth | Zustand / Context / Redux |
| **Lokal state** | input qiymati, modal ochiqmi | `useState` |
| **URL state** | filtrlar, sahifa raqami | URL search params |

```ts
// Server state'ni alohida boshqaring — caching, retry, dedup tekin keladi
const { data, fetchNextPage, isLoading } = useInfiniteQuery({
  queryKey: ["feed"],
  queryFn: ({ pageParam }) => fetchFeed(pageParam),
  getNextPageParam: (last) => last.nextCursor,
  staleTime: 60_000, // 1 daqiqa ichida qayta so'ramaydi
});
```

**⚠️ Ehtiyot bo'l:** Server state'ni Redux'da `useEffect` bilan qo'lda boshqarish — kuni o'tgan pattern. Loading/error/caching/retry/dedup'ni qo'lda yozish minglab qator buggy kod degani. TanStack Query bularning hammasini beradi.

---

## Rendering strategiyalari: CSR vs SSR vs SSG vs ISR vs Streaming

**💡 Tushuncha:** Render qayerda va qachon bajariladi — bu frontend system design'ning markaziy savoli. Har birining TTFB (time to first byte), FCP (first contentful paint), SEO va server xarajatiga ta'siri har xil.

| Strategiya | Qayerda HTML | Qachon | Plus | Minus |
|-----------|--------------|--------|------|-------|
| **CSR** | Browser | Runtime | Arzon server, boy interaktivlik | Sekin FCP, yomon SEO, katta JS bundle |
| **SSR** | Server | Har request'da | SEO, tez FCP | Server yuki, yuqori TTFB |
| **SSG** | Build time | Deploy paytida | Eng tez, CDN'dan, arzon | Stale ma'lumot, ko'p sahifa = sekin build |
| **ISR** | Build + fon | Vaqt o'tib regenerate | SSG tezligi + yangilik | Birinchi foydalanuvchi stale ko'radi |
| **Streaming SSR** | Server (bo'lak-bo'lak) | Request'da, oqim bilan | Tez TTFB, progressiv | Murakkab, infra talabi |

```tsx
// SSG — content kamdan-kam o'zgaradigan sahifa (blog, marketing)
export async function getStaticProps() {
  const posts = await fetchPosts();
  return { props: { posts }, revalidate: 60 }; // ISR: 60s da yangilanadi
}

// SSR — har request shaxsiy (dashboard, narx)
export async function getServerSideProps(ctx) {
  const user = await fetchUser(ctx.req.cookies.token);
  return { props: { user } };
}
```

**Streaming SSR** — React 18 Server Components bilan server HTML'ni bo'laklarga bo'lib yuboradi. Sekin qism (`<Suspense>`) yuklanguncha qolgan UI ko'rsatiladi. Bu TTFB'ni dramatik kamaytiradi.

```tsx
// App Router: sekin komponentni stream qilish
export default function Page() {
  return (
    <>
      <Header />                  {/* darhol yuboriladi */}
      <Suspense fallback={<Spinner />}>
        <SlowFeed />              {/* tayyor bo'lganda stream qilinadi */}
      </Suspense>
    </>
  );
}
```

**Qaror jadvali:**

- Marketing sahifa, blog → **SSG/ISR**.
- E-commerce mahsulot ro'yxati (SEO + tez-tez o'zgaruvchi) → **ISR** yoki **Streaming SSR**.
- Shaxsiy dashboard (SEO kerak emas) → **CSR** yoki SSR.
- Tezlik + SEO + dinamik → **Streaming SSR + RSC**.

---

## Hydration va uning muammolari

**💡 Tushuncha:** Hydration — server yuborgan o'lik (statik) HTML'ga browser'da JavaScript "jon kiritishi": event listener'lar ulanadi, React virtual DOM'ni mavjud HTML'ga "biriktiradi". SSR'ning maxfiy narxi shu yerda.

Muammolar:

- **Hydration mismatch** — server va client har xil HTML chiqarsa (masalan, `Date.now()`, `Math.random()`, `window` ishlatilsa). React xato beradi va qayta render qiladi.
- **Hydration sekin** — katta sahifa uchun JS yuklanishi va parse bo'lishi vaqt oladi. Sahifa *ko'rinadi* lekin *bosib bo'lmaydi* (TTI gap).

Yechimlar:

- **Selective / Progressive hydration** — eng muhim qismlarni avval hydrate qilish.
- **Islands architecture** (Astro) — faqat interaktiv "orollar"ni hydrate qilish, qolgani statik.
- **React Server Components** — interaktiv bo'lmagan komponentlar umuman client JS yubormaydi.

```tsx
// Hydration mismatch'ni oldini olish: client-only qiymatni effekt'da o'rnating
function Clock() {
  const [time, setTime] = useState<string | null>(null);
  useEffect(() => setTime(new Date().toLocaleTimeString()), []);
  return <span>{time ?? "—"}</span>; // server "—" chiqaradi, mismatch yo'q
}
```

**⚠️ Ehtiyot bo'l:** `typeof window !== "undefined"` ni render paytida ishlatib, server va client'da turli JSX qaytarmang — bu klassik hydration mismatch sababi.

---

## Next.js vs Remix: qachon nima

**💡 Tushuncha:** Ikkalasi ham full-stack React freymvork, lekin falsafasi farq qiladi. Senior intervyuda "nega bu?" degan savolga tayyor bo'ling.

**Next.js (App Router)** — React Server Components, ISR, keng ekotizim, Vercel bilan chambarchas. Eng yaxshi: SSG/ISR ko'p bo'lgan content saytlar, marketing, e-commerce, RSC'dan foydalanmoqchi bo'lsangiz.

**Remix** — web standartlariga (Fetch, FormData, `<form>`) tayanadi. Nested routing va per-route data loading kuchli. Progressive enhancement (JS'siz ham ishlaydi) markazda. Eng yaxshi: ko'p mutatsiyali, form-og'ir ilovalar, edge'da ishlash.

| Mezon | Next.js | Remix |
|-------|---------|-------|
| Rendering | SSG, ISR, SSR, RSC, streaming | SSR, streaming (defer) |
| Data loading | Server Components / `fetch` | `loader` / `action` |
| Mutatsiya | Server Actions | `<Form>` + `action` |
| Falsafa | Hybrid render, RSC | Web standartlari, PE |

**⚠️ Ehtiyot bo'l:** "Qaysi biri yaxshi?" — noto'g'ri savol. To'g'ri javob: talabga bog'liq. Content-og'ir + SEO → Next. Form/mutatsiya-og'ir + standartlar → Remix. Ko'pincha ikkalasi ham ishlaydi; jamoa tajribasi muhim omil.

---

## Performance budget va Core Web Vitals

**💡 Tushuncha:** Performance budget — bu o'lchanadigan, kelishilgan chegaralar (masalan, "JS bundle < 170KB gzip", "LCP < 2.5s"). Budgetsiz performance "keyinroq qilamiz" bo'lib qoladi va hech qachon qilinmaydi.

**Core Web Vitals (2024+):**

- **LCP (Largest Contentful Paint)** — eng katta element ko'rinishi. Maqsad: **< 2.5s**. Optimizatsiya: rasm/font preload, CDN, server tez javob.
- **INP (Interaction to Next Paint)** — bosishdan keyingi javobgarchilik (FID o'rnini bosdi, 2024-mart). Maqsad: **< 200ms**. Optimizatsiya: og'ir ishni bo'lish, `useTransition`, web worker.
- **CLS (Cumulative Layout Shift)** — kutilmagan layout siljishi. Maqsad: **< 0.1**. Optimizatsiya: rasm/video o'lchamlarini oldindan belgilash, `font-display: optional`.

```ts
// Real qurilmalarda CWV'ni o'lchash (RUM)
import { onLCP, onINP, onCLS } from "web-vitals";

function sendToAnalytics(metric: { name: string; value: number }) {
  navigator.sendBeacon("/analytics", JSON.stringify(metric));
}
onLCP(sendToAnalytics);
onINP(sendToAnalytics);
onCLS(sendToAnalytics);
```

**Budgetni amalga oshirish:**

```js
// Vite + rollup-plugin-visualizer yoki bundlesize CI'da
// package.json
{
  "bundlesize": [
    { "path": "dist/assets/index-*.js", "maxSize": "170 kB" }
  ]
}
```

**⚠️ Ehtiyot bo'l:** Lab data (Lighthouse, sizning tez kompyuteringiz) bilan field data (haqiqiy foydalanuvchilar, sekin 3G, arzon telefon) bir xil emas. Har doim RUM (Real User Monitoring) bilan field data'ni o'lchang.

---

## Caching: HTTP cache, CDN, Service Worker

**💡 Tushuncha:** Caching — performance'ning eng kuchli quroli. To'rtta qatlam bor: browser memory/disk, HTTP cache (header'lar), CDN (edge), Service Worker (programmatik). Har biri turli muammoni hal qiladi.

**HTTP cache header'lar:**

```
# Versiyalangan asset (fayl nomida hash) — abadiy cache
Cache-Control: public, max-age=31536000, immutable

# HTML — har doim tekshir, lekin stale'ni ko'rsatishi mumkin
Cache-Control: no-cache
# yoki: stale-while-revalidate
Cache-Control: max-age=0, s-maxage=86400, stale-while-revalidate=604800
```

- **`immutable`** — fayl hech qachon o'zgarmaydi (hashlangan), browser tekshirmaydi ham.
- **`stale-while-revalidate`** — eski versiyani darhol ber, fonda yangisini ol.
- **ETag / `If-None-Match`** — server 304 qaytarib, body yubormaydi (band kengligini tejaydi).

**CDN** — asset'larni foydalanuvchiga geografik yaqin edge serverdan beradi. Statik fayllar (JS, CSS, rasm) va hatto SSR HTML (edge cache) ham. Cache invalidation — deploy'da hash o'zgaradi yoki API orqali purge.

**Service Worker** — browser va tarmoq orasidagi proxy. Offline, push, background sync. Strategiyalar:

```ts
// Service Worker caching strategiyalari (Workbox)
// App shell — cache-first (tez, kamdan-kam o'zgaradi)
registerRoute(({ request }) => request.destination === "script",
  new CacheFirst());

// API — network-first (yangi ma'lumot muhim, offline fallback bor)
registerRoute(({ url }) => url.pathname.startsWith("/api/"),
  new NetworkFirst({ networkTimeoutSeconds: 3 }));
```

**⚠️ Ehtiyot bo'l:** HTML'ni uzoq cache qilmang — yangi deploy'da foydalanuvchi eski HTML + yangi asset'larni olib "white screen" yoki chunk load error'ga uchraydi. HTML har doim `no-cache` yoki qisqa `s-maxage`.

---

## Networking: REST vs GraphQL vs WebSocket vs SSE

**💡 Tushuncha:** Tarmoq protokolini *muammoga* qarab tanlang. Yagona to'g'ri javob yo'q — har biri ma'lum stsenariy uchun.

| Protokol | Yo'nalish | Eng yaxshi holat |
|----------|-----------|------------------|
| **REST** | Client → Server (req/res) | Oddiy CRUD, cache oson, keng qo'llab |
| **GraphQL** | Client → Server (req/res) | Murakkab nested data, over/under-fetching muammosi, ko'p client |
| **WebSocket** | Ikki tomonlama, doimiy | Chat, o'yin, kollaboratsiya (real-time, ikki yo'nalish) |
| **SSE** | Server → Client (bir yo'nalish) | Live feed, bildirishnoma, narx tickeri |
| **Polling** | Client so'rab turadi | Real-time ehtiyoji past, infra oddiy |

**REST vs GraphQL:**

- REST — har resurs uchun endpoint, oddiy, HTTP cache tabiiy ishlaydi. Kamchilik: over-fetching (keraksiz maydonlar), under-fetching (N+1 so'rov).
- GraphQL — bitta endpoint, client aniq kerakli maydonni so'raydi. Kamchilik: HTTP cache qiyin, server murakkab, query xavfsizligi (depth/complexity limit kerak).

**WebSocket vs SSE:**

```ts
// SSE — server'dan bir yo'nalishli oqim (avtomatik reconnect bor!)
const es = new EventSource("/api/notifications");
es.onmessage = (e) => console.log("Yangi:", JSON.parse(e.data));
es.onerror = () => {/* brauzer o'zi qayta ulanadi */};

// WebSocket — ikki yo'nalishli (chat uchun)
const ws = new WebSocket("wss://chat.example.com");
ws.onmessage = (e) => addMessage(JSON.parse(e.data));
ws.send(JSON.stringify({ type: "msg", text: "Salom" }));
```

**⚠️ Ehtiyot bo'l:** WebSocket avtomatik qayta ulanmaydi va heartbeat (ping/pong) o'zingiz qilishingiz kerak. SSE esa avtomatik reconnect qiladi. Faqat server→client kerak bo'lsa, WebSocket emas, SSE tanlang — u oddiyroq va HTTP/2 ustida ishlaydi.

---

## Optimistic update, pagination, infinite scroll

**💡 Tushuncha:** Optimistic update — server javobini kutmasdan UI'ni *darhol* yangilash (like bosilganda raqam shu zahoti oshadi), keyin server xato bersa orqaga qaytarish. Bu ilovani "tez" his qildiradi.

```ts
// TanStack Query bilan optimistic like
const likeMutation = useMutation({
  mutationFn: (postId: string) => api.like(postId),
  onMutate: async (postId) => {
    await queryClient.cancelQueries({ queryKey: ["feed"] });
    const prev = queryClient.getQueryData(["feed"]);
    // darhol UI'ni yangilash
    queryClient.setQueryData(["feed"], (old) => toggleLike(old, postId));
    return { prev }; // rollback uchun
  },
  onError: (_err, _id, ctx) => {
    queryClient.setQueryData(["feed"], ctx.prev); // xato — orqaga qaytar
  },
  onSettled: () => queryClient.invalidateQueries({ queryKey: ["feed"] }),
});
```

**Pagination turlari:**

- **Offset (`?page=2&limit=20`)** — oddiy, "3-sahifaga o't" mumkin. Kamchilik: ma'lumot o'zgarsa dublikat/o'tkazib yuborish; katta offset sekin.
- **Cursor (`?cursor=<id>`)** — barqaror (yangi post qo'shilsa ham siljimaydi), katta hajmga mos. Kamchilik: "N-sahifaga sakra" mumkin emas. Feed uchun **cursor** standart.

**Infinite scroll** — `IntersectionObserver` bilan oxirgi elementga yetganda keyingisini yuklash:

```tsx
function useInfiniteScroll(onReachEnd: () => void) {
  const sentinel = useRef<HTMLDivElement>(null);
  useEffect(() => {
    const obs = new IntersectionObserver(
      ([entry]) => entry.isIntersecting && onReachEnd(),
      { rootMargin: "200px" } // ko'rinishdan 200px oldin yukla
    );
    if (sentinel.current) obs.observe(sentinel.current);
    return () => obs.disconnect();
  }, [onReachEnd]);
  return sentinel;
}
```

**⚠️ Ehtiyot bo'l:** Cheksiz scroll footer'ni yetib bo'lmas qiladi va xotira o'sib ketadi (DOM node'lar to'planadi). Yechim: **windowing/virtualizatsiya** (`react-window`, TanStack Virtual) — faqat ko'rinadigan elementlarni DOM'da saqlash.

---

## Offline va PWA

**💡 Tushuncha:** PWA (Progressive Web App) — web ilovani native ilovaga o'xshatadi: o'rnatish mumkin, offline ishlaydi, push oladi. Asosi: Service Worker + Web App Manifest + HTTPS.

```json
// manifest.json
{
  "name": "Mening Ilovam",
  "short_name": "Ilova",
  "start_url": "/",
  "display": "standalone",
  "theme_color": "#000000",
  "icons": [{ "src": "/icon-512.png", "sizes": "512x512", "type": "image/png" }]
}
```

**Offline strategiyalar:**

- **App shell caching** — UI karkasini (HTML/CSS/JS) cache'lab, offline'da ham ilova ochiladi.
- **Background Sync** — offline'da yuborilgan amallar (xabar, like) navbatga qo'yilib, ulanish tiklanganda yuboriladi.
- **IndexedDB** — katta strukturali ma'lumotni client'da saqlash (offline feed, draft'lar).

**⚠️ Ehtiyot bo'l:** Service Worker'ni noto'g'ri yangilash "abadiy eski versiya" tuzog'iga olib keladi. `skipWaiting()` va versiyalash strategiyasini puxta o'ylang. Hamma narsani offline qilishga urinmang — bank balansini offline ko'rsatish xavfli.

---

## Accessibility katta hajmda

**💡 Tushuncha:** Senior darajada a11y "alt qo'shish" emas. U arxitektura qarori: semantik HTML, klaviatura navigatsiyasi, focus management, screen reader uchun ARIA va dinamik kontent e'lonlari (`aria-live`).

Asosiy prinsiplar:

- **Semantik HTML** — `<button>`, `<nav>`, `<main>`, `<h1>` to'g'ri ishlatish. `<div onClick>` emas.
- **Klaviatura** — barcha interaktiv element Tab bilan yetib bo'ladigan va Enter/Space bilan ishlaydigan bo'lishi shart.
- **Focus management** — modal ochilganda focus ichiga "qamab" (focus trap), yopilganda qaytarish. Route o'zgarganda focus'ni boshqarish.
- **`aria-live`** — dinamik o'zgarishlarni (yangi post, toast) screen reader'ga e'lon qilish.
- **Color contrast** — WCAG AA: matn uchun 4.5:1.

```tsx
// Live region — yangi xabarlar screen reader'ga e'lon qilinadi
function FeedAnnouncer({ newCount }: { newCount: number }) {
  return (
    <div aria-live="polite" className="sr-only">
      {newCount > 0 && `${newCount} ta yangi post mavjud`}
    </div>
  );
}

// Focus trap'li modal (yoki radix/react-aria ishlatish tavsiya etiladi)
function Modal({ onClose, children }: ModalProps) {
  const ref = useRef<HTMLDivElement>(null);
  useEffect(() => {
    const prev = document.activeElement as HTMLElement;
    ref.current?.focus();
    return () => prev?.focus(); // yopilganda focus qaytadi
  }, []);
  return (
    <div role="dialog" aria-modal="true" tabIndex={-1} ref={ref}
      onKeyDown={(e) => e.key === "Escape" && onClose()}>
      {children}
    </div>
  );
}
```

**⚠️ Ehtiyot bo'l:** ARIA'ni ortiqcha ishlatmang. "No ARIA is better than bad ARIA". Semantik HTML allaqachon bergan narsani ARIA bilan takrorlamang (`<button role="button">` keraksiz).

---

## i18n: xalqarolashtirish

**💡 Tushuncha:** i18n — bu nafaqat matn tarjimasi. U raqam/sana/valyuta formatlash, ko'plik shakllari (plural rules), RTL (o'ngdan-chapga til, masalan arab), va bundle'ni tilga qarab bo'lishni o'z ichiga oladi.

```ts
// Intl API — built-in, kutubxonasiz
new Intl.NumberFormat("uz-UZ").format(1234567);        // "1 234 567"
new Intl.DateTimeFormat("uz-UZ").format(new Date());    // "30.06.2026"
new Intl.PluralRules("ru").select(2);                   // "few" — rus tilida murakkab

// react-i18next bilan
const { t } = useTranslation();
t("posts.count", { count: 5 }); // "5 ta post" / "5 posts" / "5 постов"
```

E'tibor qiladigan nuqtalar:

- **Lazy load translations** — barcha tilni emas, faqat tanlanganini yuklang.
- **RTL** — `dir="rtl"`, CSS logical properties (`margin-inline-start`, `padding-inline-end`).
- **String interpolation va plurallar** — qo'lda emas, ICU MessageFormat bilan.
- **SSR** — server foydalanuvchi tilini (`Accept-Language`) aniqlab to'g'ri tilda render qilishi kerak.

**⚠️ Ehtiyot bo'l:** Matnni kodda "Salom" + name kabi yopishtirib (concatenation) tarjima qilib bo'lmaydi — har tilda so'z tartibi har xil. Har doim parametrli shablon ishlating.

---

## Xavfsizlik: XSS, CSRF, CSP

**💡 Tushuncha:** Frontend xavfsizligi senior intervyuda doimiy. Uchta asosiy hujum: XSS (kod inyeksiyasi), CSRF (soxta so'rov), va himoya qatlami sifatida CSP.

**XSS (Cross-Site Scripting)** — hujumchi sahifaga zararli JS kiritadi. React JSX'ni avtomatik escape qiladi, lekin `dangerouslySetInnerHTML` teshik ochadi.

```tsx
// ❌ XSS teshigi
<div dangerouslySetInnerHTML={{ __html: userComment }} />

// ✅ Sanitizatsiya
import DOMPurify from "dompurify";
<div dangerouslySetInnerHTML={{ __html: DOMPurify.sanitize(userComment) }} />
```

**CSRF (Cross-Site Request Forgery)** — foydalanuvchining cookie'sidan foydalanib soxta so'rov yuborish. Himoya: `SameSite=Strict/Lax` cookie, CSRF token, yoki cookie o'rniga `Authorization` header (token).

**CSP (Content Security Policy)** — qaysi manbalardan skript/stil yuklanishi mumkinligini cheklaydi. XSS'ga qarshi kuchli ikkinchi qatlam.

```
Content-Security-Policy: default-src 'self'; script-src 'self' 'nonce-abc123'; img-src 'self' https:;
```

**⚠️ Ehtiyot bo'l:** JWT'ni `localStorage`'da saqlash XSS orqali o'g'irlanish xavfi tug'diradi. Sezgir token uchun `HttpOnly` cookie xavfsizroq (JS o'qiy olmaydi). Hech qachon mijoz tomonida maxfiy kalit (API secret) saqlamang.

---

## Rasm optimizatsiyasi

**💡 Tushuncha:** Rasmlar odatda sahifa og'irligining 50%+ ini tashkil qiladi va LCP'ga to'g'ridan-to'g'ri ta'sir qiladi. Senior dizaynda rasm optimizatsiyasi alohida mavzu.

Texnikalar:

- **Zamonaviy formatlar** — AVIF (eng kichik) > WebP > JPEG. `<picture>` bilan fallback.
- **Responsive images** — `srcset` + `sizes` bilan qurilmaga mos o'lcham.
- **Lazy loading** — `loading="lazy"` ekrandan tashqaridagi rasmlarni keyin yuklaydi.
- **O'lcham belgilash** — `width`/`height` yoki `aspect-ratio` CLS'ni oldini oladi.
- **Blur placeholder / LQIP** — past sifatli placeholder, keyin asl rasm.
- **CDN image transform** — on-the-fly o'lcham/format (Cloudinary, imgix, Next Image).

```tsx
// Responsive, lazy, zamonaviy format
<picture>
  <source srcSet="hero.avif" type="image/avif" />
  <source srcSet="hero.webp" type="image/webp" />
  <img src="hero.jpg" alt="..." width={1200} height={630}
    loading="lazy" decoding="async" />
</picture>

// Next.js Image — avtomatik optimizatsiya
<Image src="/hero.jpg" alt="..." width={1200} height={630} priority />
```

**⚠️ Ehtiyot bo'l:** LCP rasmga (hero) `loading="lazy"` QO'YMANG — bu LCP'ni sekinlashtiradi. Birinchi ekrandagi muhim rasmga `priority` / preload bering, qolganlarini lazy qiling.

---

## Micro-frontend

**💡 Tushuncha:** Micro-frontend — katta ilovani mustaqil deploy qilinadigan, alohida jamoalarga tegishli bo'laklarga bo'lish (xuddi microservice'ning frontend versiyasi). Bu tashkiliy masshtablanish uchun, texnik emas.

Yondashuvlar:

- **Module Federation** (Webpack/Vite) — runtime'da boshqa app'dan kod yuklash.
- **Build-time integration** — npm package sifatida.
- **Iframe** — to'liq izolyatsiya, lekin og'ir va UX cheklangan.
- **Web Components** — freymvork-agnostik kapsulalash.

**Qiyinchiliklar:** umumiy dependency (React ikki marta yuklanmasligi), umumiy design system, routing koordinatsiyasi, bundle hajmi, versiya mosligi, jamoalararo kontrakt.

**⚠️ Ehtiyot bo'l:** Micro-frontend katta narx bilan keladi (murakkablik, bundle, koordinatsiya). U faqat ko'p mustaqil jamoa bitta katta ilova ustida ishlaganda mantiqiy. Kichik jamoa uchun bu over-engineering — monolit yoki monorepo yetarli.

---

## Build tooling: Vite, code splitting

**💡 Tushuncha:** Build tooling foydalanuvchiga yetadigan bundle hajmini va dev tajribasini belgilaydi. Vite zamonaviy standart: dev'da native ESM (bundle'siz, tez HMR), prod'da Rollup bilan optimallashtirilgan build.

**Code splitting** — bitta katta bundle o'rniga kichik bo'laklar, faqat kerak bo'lganda yuklanadi:

```tsx
// Route bo'yicha splitting — har sahifa alohida chunk
const Dashboard = lazy(() => import("./Dashboard"));

<Suspense fallback={<Spinner />}>
  <Route path="/dashboard" element={<Dashboard />} />
</Suspense>

// Og'ir kutubxonani kechiktirib yuklash (masalan, chart)
async function showChart() {
  const { Chart } = await import("heavy-chart-lib");
  // ...
}
```

Boshqa optimizatsiyalar:

- **Tree shaking** — ishlatilmagan eksportlarni olib tashlash (ESM kerak).
- **Vendor chunk** — kamdan-kam o'zgaruvchi kutubxonalarni alohida cache uchun.
- **Preload / prefetch** — `<link rel="prefetch">` bilan ehtimoliy keyingi sahifani fonda yuklash.
- **Compression** — Brotli > gzip.

**⚠️ Ehtiyot bo'l:** Haddan tashqari mayda splitting — ko'p kichik so'rov ham yomon (HTTP/1.1'da, waterfall). Balansni saqlang va bundle analyzer bilan haqiqiy hajmni tekshiring.

---

## Monitoring: error tracking va RUM

**💡 Tushuncha:** Deploy qilgandan keyin "qanday ishlayapti?" degan savolga javob beradigan ikki tizim: **error tracking** (nima buzilyapti) va **RUM** (qanchalik tez/sekin haqiqiy foydalanuvchilar uchun).

**Error tracking** (Sentry kabi):

```ts
// Global error va promise rejection'ni ushlash
window.addEventListener("error", reportError);
window.addEventListener("unhandledrejection", reportRejection);

// React'da Error Boundary + Sentry
Sentry.init({ dsn: "...", tracesSampleRate: 0.1 });
```

Muhim: **source map** yuklash (minified stack trace'ni o'qib bo'lmaydi), **release/version** belgilash, **user context** (lekin PII'ga ehtiyot), **sampling** (hamma session'ni emas).

**RUM (Real User Monitoring)** — haqiqiy foydalanuvchilardan CWV, sahifa yuklanish vaqti, API latency yig'ish. `web-vitals` kutubxonasi + `sendBeacon`. Bu lab data'dan farqli — haqiqiy qurilma va tarmoqlarni ko'rsatadi.

**⚠️ Ehtiyot bo'l:** Monitoring'ning o'zi performance'ni buzmasin. Analytics'ni `sendBeacon` yoki `requestIdleCallback` bilan yuboring, asosiy thread'ni bloklamang. PII (shaxsiy ma'lumot) va token'larni xato hisobotlariga yuborib qo'ymang — bu maxfiylik buzilishi.

---

## Ishlangan dizayn 1: News Feed / Infinite Scroll

**💡 Tushuncha:** Bu klassik intervyu savoli (Twitter/Facebook feed). U pagination, virtualizatsiya, optimistic update, caching va real-time yangilanishni birlashtiradi.

**1. Requirements:**
- Cheksiz scroll qilinadigan post feed; like/comment; yangi post'lar bildirishnomasi.
- Tez (LCP < 2.5s), mobil-birinchi, ming-millionlab post.
- Media (rasm/video) qo'llab-quvvatlash.

**2. Arxitektura:**

```
<FeedPage> (useInfiniteQuery)
 ├─ <VirtualList>           // TanStack Virtual — faqat ko'rinadigan postlar
 │   └─ <PostCard />        // memoizatsiya qilingan
 ├─ <NewPostsBanner />      // SSE orqali "12 ta yangi post"
 └─ sentinel <div>          // IntersectionObserver
```

**3. API:** `GET /feed?cursor=&limit=20` → cursor-based pagination (barqaror).

**4. Asosiy yechimlar:**
- **Virtualizatsiya** — DOM'da faqat ~10–15 post, scroll bilan qayta ishlatiladi. Aks holda 10000 post = 10000 DOM node = xotira halokati.
- **Cursor pagination** — yangi post qo'shilsa ham siljish yo'q.
- **Optimistic like** — darhol UI, xatoda rollback.
- **Real-time** — SSE bilan yangi post hisobi, foydalanuvchi tugmani bossa qo'shiladi (avtomatik emas — scroll buzmaslik uchun).
- **Caching** — TanStack Query'da feed cache, `staleTime` bilan ortiqcha so'rov yo'q.

**5. Edge case'lar:** bo'sh feed, xato/retry, sekin tarmoq (skeleton), takroriy post (dedup by ID), scroll pozitsiyasini saqlash (orqaga qaytishda).

> To'liq yechim kodi va diagrammasi yechimlar faylida.

---

## Ishlangan dizayn 2: Autocomplete / Typeahead

**💡 Tushuncha:** Qidiruv autocomplete — debounce, race condition, caching, accessibility va klaviatura navigatsiyasini sinaydigan zich savol.

**1. Requirements:** matn yozilganda takliflar; klaviatura bilan tanlash; tez; a11y.

**2. Asosiy yechimlar:**
- **Debounce** — har harf uchun emas, yozish to'xtagach (250–300ms) so'rov.
- **Race condition** — sekin javob tez javobni bosib o'tmasligi uchun eski so'rovni `AbortController` bilan bekor qilish yoki faqat oxirgi natijani qabul qilish.
- **Caching** — bir xil so'rovni qayta yubormaslik (`Map` cache).
- **Klaviatura** — ↑/↓ harakat, Enter tanlash, Esc yopish.
- **a11y** — `role="combobox"`, `aria-activedescendant`, `aria-expanded`.

```tsx
// Race condition'ga qarshi AbortController
useEffect(() => {
  const ctrl = new AbortController();
  fetch(`/search?q=${query}`, { signal: ctrl.signal })
    .then((r) => r.json()).then(setResults)
    .catch((e) => e.name !== "AbortError" && setError(e));
  return () => ctrl.abort(); // har yangi query eskisini bekor qiladi
}, [query]);
```

**3. Edge case'lar:** bo'sh natija, tarmoq xatosi, juda tez yozish, tashqariga bosish, mobil klaviatura.

> To'liq yechim yechimlar faylida.

---

## Ishlangan dizayn 3: Real-time Chat UI

**💡 Tushuncha:** Chat — WebSocket, xabar holati (yuborilmoqda/yuborildi/o'qildi), reconnection, optimistic UI va xabar tartibini sinaydigan eng murakkab dizayn.

**1. Requirements:** real-time xabar; yuborilish holati; reconnect; offline navbat; tarix (pagination yuqoriga).

**2. Asosiy yechimlar:**
- **WebSocket** — ikki yo'nalishli; heartbeat (ping/pong) bilan o'lik ulanishni aniqlash; exponential backoff bilan reconnect.
- **Optimistic UI** — xabar darhol "yuborilmoqda" holatida ko'rinadi, server ACK kelganda "yuborildi".
- **Xabar tartibi** — server timestamp/sequence bilan tartiblash, ID bo'yicha dedup.
- **Offline navbat** — ulanish uzilsa, xabarlar IndexedDB'ga, tiklanganda yuboriladi.
- **Tarix** — yuqoriga scroll'da eski xabarlarni cursor bilan yuklash; scroll pozitsiyasini saqlash.

```ts
// Reconnect: exponential backoff
function connect(attempt = 0) {
  const ws = new WebSocket(URL);
  ws.onclose = () => {
    const delay = Math.min(1000 * 2 ** attempt, 30_000);
    setTimeout(() => connect(attempt + 1), delay);
  };
  ws.onopen = () => { attempt = 0; flushOfflineQueue(); };
}
```

**3. Edge case'lar:** ulanish uzilishi, dublikat xabar, tartibsiz kelish, katta tarix (virtualizatsiya), typing indikatori, o'qildi statusi.

> To'liq yechim yechimlar faylida.

---

## Savol-javoblar

### ❓ Frontend system design intervyusini qanday boshlash kerak?

**✅ Javob:** Hech qachon kod yoki komponent bilan boshlamang. Avval **talablarni aniqlashtiring** (funksional + non-funksional): platforma, foydalanuvchilar soni, real-time/offline/SEO/i18n kerakmi. RADIO freymvorkidan foydalaning: Requirements → Architecture → Data → Interface(API) → Optimizations. Eng muhimi — ovoz chiqarib o'ylang va trade-off'larni aytib turing. Intervyuer sizning *qaror jarayoningizni* baholaydi, mukammal javobni emas.

### ❓ CSR, SSR, SSG, ISR farqini va qachon qaysi birini tanlashni tushuntiring.

**✅ Javob:** **CSR** — HTML browser'da render bo'ladi: arzon server, lekin sekin FCP va yomon SEO (shaxsiy dashboard uchun yaxshi). **SSR** — server har request'da render qiladi: yaxshi SEO va tez FCP, lekin server yuki va yuqori TTFB (shaxsiy + SEO kerak bo'lganda). **SSG** — build paytida render, CDN'dan beriladi: eng tez va arzon, lekin stale (blog, marketing). **ISR** — SSG + fonda vaqti-vaqti bilan regenerate: SSG tezligi + nisbiy yangilik (e-commerce katalog). Qaror: SEO + dinamiklik + server byudjeti uchburchagiga qarab tanlanadi.

### ❓ Hydration nima va u qanday muammolar keltiradi?

**✅ Javob:** Hydration — server yuborgan statik HTML'ga browser'da JS event listener'larni ulashi ("jon kiritish"). Ikki asosiy muammo: (1) **mismatch** — server va client har xil HTML chiqarsa (`Date.now()`, `window`, `Math.random()` sababli), React ogohlantiradi va qayta render qiladi; (2) **TTI gap** — sahifa ko'rinadi lekin JS yuklanguncha bosib bo'lmaydi. Yechimlar: progressive/selective hydration, islands architecture, React Server Components (interaktiv bo'lmagan qism umuman JS yubormaydi). Client-only qiymatlarni `useEffect`'da o'rnating.

### ❓ Core Web Vitals nima va ularni qanday yaxshilash mumkin?

**✅ Javob:** Uchta asosiy metrika: **LCP** (eng katta element ko'rinishi, < 2.5s — rasm/font preload, CDN bilan), **INP** (bosishga javob, < 200ms, 2024-da FID o'rnini bosdi — og'ir ishni bo'lish, `useTransition`, web worker bilan), **CLS** (layout siljishi, < 0.1 — rasm/font o'lchamlarini oldindan belgilash bilan). Eng muhim nuance: lab data (Lighthouse) ≠ field data (haqiqiy foydalanuvchilar). Doimo **RUM** bilan haqiqiy qurilmalardan o'lchang.

### ❓ REST va GraphQL orasida qanday tanlaysiz?

**✅ Javob:** **REST** — oddiy CRUD, HTTP cache tabiiy ishlaydi, keng qo'llab-quvvatlanadi; kamchilik: over-fetching va under-fetching (N+1). **GraphQL** — client aniq kerakli maydonlarni so'raydi, murakkab nested data va ko'p xil client (web/mobile) uchun ideal; kamchilik: HTTP cache qiyin, server murakkab, query depth/complexity limit kerak (DoS xavfi). Qaror: sodda, cache-og'ir API → REST; murakkab, har-xil-shaklli data ehtiyoji → GraphQL.

### ❓ WebSocket va SSE qachon ishlatiladi?

**✅ Javob:** **WebSocket** — ikki tomonlama, doimiy ulanish kerak bo'lganda (chat, kollaboratsiya, o'yin). Kamchilik: reconnect va heartbeat'ni o'zingiz yozasiz. **SSE (Server-Sent Events)** — faqat server→client bir yo'nalishli oqim (live feed, bildirishnoma, narx ticker). Afzalligi: avtomatik reconnect, oddiy, HTTP ustida. Qoida: agar client server'ga real-time yuborishi shart bo'lmasa, WebSocket emas, SSE tanlang — soddaroq. **Polling** — real-time ehtiyoji past va infra oddiy bo'lishi kerak bo'lganda.

### ❓ Optimistic update nima va xato bo'lsa nima qilasiz?

**✅ Javob:** Optimistic update — server javobini kutmasdan UI'ni darhol yangilash (like bosilganda raqam shu zahoti oshadi), bu ilovani tez his qildiradi. Naqsh: `onMutate`'da avvalgi state'ni saqlab UI'ni yangilash, `onError`'da saqlangan state'ga **rollback**, `onSettled`'da serverdan haqiqiy holatni qayta olish (invalidate). Muhim: rollback uchun avvalgi qiymatni kontekstda saqlash va in-flight query'larni `cancelQueries` bilan to'xtatib, ular optimistic state'ni bosib o'tmasligini ta'minlash.

### ❓ Cursor va offset pagination farqini tushuntiring.

**✅ Javob:** **Offset** (`?page=2&limit=20`) — oddiy, "N-sahifaga sakra" mumkin; kamchiligi: ma'lumot o'rtada o'zgarsa dublikat yoki o'tkazib yuborish, va katta offset DB'da sekin. **Cursor** (`?cursor=<id>`) — oxirgi ko'rilgan elementdan keyingisini oladi, yangi element qo'shilsa ham barqaror (siljish yo'q), katta hajmga mos; kamchiligi: ixtiyoriy sahifaga sakrab bo'lmaydi. Cheksiz scroll/feed uchun har doim **cursor** tanlanadi.

### ❓ Infinite scroll'da xotira muammosini qanday hal qilasiz?

**✅ Javob:** Cheksiz scroll'da DOM node'lar to'planib ketadi (10000 post = halokat) va scroll sekinlashadi. Yechim — **virtualizatsiya/windowing** (`react-window`, TanStack Virtual): faqat ko'rinadigan ~10–15 elementni DOM'da saqlash, qolganlari uchun bo'sh joy (spacer) qoldirish. Yuklash uchun `IntersectionObserver` bilan oxirgi elementga yetganda keyingi sahifani olish. Qo'shimcha: footer'ga yetish muammosi uchun footer'ni alohida joylashtirish yoki "yana yuklash" tugmasi.

### ❓ Frontend'da XSS'ni qanday oldini olasiz?

**✅ Javob:** **XSS** — hujumchi sahifaga zararli JS kiritadi. Himoya qatlamlari: (1) React JSX'ni avtomatik escape qiladi — uni buzmaslik; (2) `dangerouslySetInnerHTML` ishlatish kerak bo'lsa, **DOMPurify** bilan sanitizatsiya qilish; (3) **CSP** (Content Security Policy) header bilan ruxsat etilgan skript manbalarini cheklash — bu ikkinchi mudofaa qatlami; (4) URL'larni validatsiya qilish (`javascript:` protokoliga ehtiyot); (5) token'ni `localStorage` o'rniga `HttpOnly` cookie'da saqlash (JS o'qiy olmaydi).

### ❓ CSRF nima va undan qanday himoyalanasiz?

**✅ Javob:** **CSRF** — foydalanuvchining mavjud cookie session'idan foydalanib, boshqa saytdan soxta so'rov yuborish. Himoya: (1) **`SameSite=Lax/Strict`** cookie — cross-site so'rovlarda cookie yuborilmaydi (zamonaviy asosiy himoya); (2) **CSRF token** — serverning bergan maxsus tokenini har mutatsiya so'roviga qo'shish; (3) cookie o'rniga **`Authorization` header** (Bearer token) ishlatish — bu avtomatik yuborilmaydi, shuning uchun CSRF tabiiy yo'qoladi. Odatda SameSite + token kombinatsiyasi ishlatiladi.

### ❓ JS bundle hajmini qanday kamaytirasiz?

**✅ Javob:** (1) **Code splitting** — route va og'ir komponentlarni `lazy`/`import()` bilan alohida chunk'larga bo'lish; (2) **Tree shaking** — ishlatilmagan eksportlarni olib tashlash (ESM import ishlatish, named import); (3) og'ir kutubxonalarni yengilroq alternativaga almashtirish (moment → date-fns/Intl); (4) **vendor chunk** alohida cache uchun; (5) **compression** (Brotli); (6) **bundle analyzer** bilan haqiqiy aybdorlarni topish. Lekin haddan tashqari mayda splitting waterfall'ga olib keladi — balans saqlang.

### ❓ Service Worker bilan qanday caching strategiyalari bor?

**✅ Javob:** **Cache-first** — avval cache'dan, faqat yo'q bo'lsa tarmoqdan (kamdan-kam o'zgaruvchi asset: JS, CSS, fontlar). **Network-first** — avval tarmoqdan, ishlamasa cache'dan (yangi ma'lumot muhim, lekin offline fallback kerak: API). **Stale-while-revalidate** — cache'dan darhol berib, fonda yangilash (tezlik + nisbiy yangilik). **Cache-only / Network-only** — maxsus holatlar. Tanlov ma'lumotning yangilik talabiga bog'liq. Ehtiyot: SW yangilanishini noto'g'ri boshqarsa "abadiy eski versiya" tuzog'i bo'ladi.

### ❓ Micro-frontend qachon mantiqiy va qachon emas?

**✅ Javob:** Micro-frontend **tashkiliy** masshtablanish uchun — ko'plab mustaqil jamoa bitta katta ilovada ishlab, alohida deploy qilishni xohlaganda (Module Federation eng zamonaviy yondashuv). U **texnik** muammoni hal qilmaydi, balki jamoa avtonomiyasini beradi. Narxi yuqori: umumiy dependency boshqaruvi (React ikki marta yuklanmasligi), umumiy design system, routing koordinatsiyasi, bundle hajmi, versiya mosligi. Kichik jamoa uchun bu **over-engineering** — monorepo (Nx/Turborepo) yetarli. Faqat haqiqiy tashkiliy og'riq bo'lsa qo'llang.

### ❓ Deploy'dan keyin ilova sog'ligini qanday kuzatasiz?

**✅ Javob:** Ikki tizim: (1) **Error tracking** (Sentry) — JS xatolari, unhandled rejection'lar, Error Boundary integratsiyasi; source map bilan o'qib bo'ladigan stack trace, release versiyasi, sampling. (2) **RUM (Real User Monitoring)** — `web-vitals` bilan haqiqiy foydalanuvchilardan LCP/INP/CLS, API latency, `sendBeacon` orqali yuborish. Muhim: monitoring'ning o'zi performance'ni buzmasligi (`requestIdleCallback`/`sendBeacon`) va xato hisobotlariga PII/token tushib qolmasligi (maxfiylik).

---

## Masalalar

> Yechimlar: [`solutions/frontend/09-frontend-system-design.md`](../solutions/frontend/09-frontend-system-design.md)

Quyidagi mashqlar — to'liq frontend system design intervyu savollari. Har birini RADIO freymvorki bilan ishlang: requirements → architecture → data/API → optimizations. Diagramma chizing, trade-off'larni yozing.

**1. Twitter-style News Feed dizayn qiling.** Cheksiz scroll, like/retweet, media, real-time yangi post bildirishnomasi. Virtualizatsiya, cursor pagination, optimistic update va caching strategiyasini batafsil ko'rsating. 10 million foydalanuvchi uchun masshtablang.

**2. Autocomplete / Typeahead komponentini dizayn qiling.** Debounce, race condition (AbortController), natija caching, klaviatura navigatsiyasi va to'liq accessibility (`combobox` ARIA pattern). API shaklini va edge case'larni belgilang.

**3. Real-time guruh chat ilovasini dizayn qiling.** WebSocket vs SSE tanlovini asoslang, xabar holatlari (yuborilmoqda/yuborildi/o'qildi), reconnection (exponential backoff), offline navbat (IndexedDB), xabar tartibi va dedup, tarix pagination'ni yoriting.

**4. Tasvirlar uchun og'ir Image Gallery / Pinterest-style masonry dizayn qiling.** Responsive masonry layout, lazy loading, zamonaviy rasm formatlari, LQIP placeholder, CLS'ni nolga yaqinlashtirish va virtualizatsiyani birlashtiring. Performance budget belgilang.

**5. Hujjatlar uchun real-time kollaborativ editor (Google Docs kabi) dizayn qiling.** Konflikt hal qilish (OT yoki CRDT konseptual darajada), kursorlar prezentatsiyasi, offline tahrir, ulanish uzilishida sinxronlash va katta hujjat performance'ini muhokama qiling.

**6. E-commerce mahsulot ro'yxati sahifasini dizayn qiling.** SEO (rendering strategiyasi tanlovi va asoslash), filtrlash/saralash (URL state), pagination, rasm optimizatsiyasi, Core Web Vitals budgeti va A/B test infratuzilmasini yoriting.

**7. Analytics Dashboard dizayn qiling.** Ko'p widget, og'ir grafiklar (chart kutubxonasini kechiktirib yuklash), real-time yangilanish (SSE/polling), katta dataset (server-side aggregation), va INP'ni saqlash (web worker, virtualizatsiya) ustida ishlang.

**8. Offline-first eslatma (note-taking) PWA dizayn qiling.** Service Worker, IndexedDB lokal saqlash, background sync, konflikt hal qilish (local vs remote), va serverga sinxronlash strategiyasini batafsil yoriting. PWA o'rnatish oqimini ham qo'shing.

---

← [Frontend bo'limiga qaytish](./README.md)
