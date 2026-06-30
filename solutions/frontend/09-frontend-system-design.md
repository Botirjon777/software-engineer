# Frontend System Design — Yechimlar

Bu fayl [`frontend/09-frontend-system-design.md`](../../frontend/09-frontend-system-design.md) dagi masalalar yechimlari. Har bir yechim **yondashuv (RADIO)** + **arxitektura diagrammasi** (matn) + **kod**dan iborat. Intervyuda bularni ovoz chiqarib bayon qiling — trade-off'larni unutmang.

## Mundarija

- [1. Twitter-style News Feed](#1-twitter-style-news-feed)
- [2. Autocomplete / Typeahead](#2-autocomplete--typeahead)
- [3. Real-time Guruh Chat](#3-real-time-guruh-chat)
- [4. Image Gallery / Masonry](#4-image-gallery--masonry)
- [5. Kollaborativ Editor](#5-kollaborativ-editor)
- [6. E-commerce Mahsulot Ro'yxati](#6-e-commerce-mahsulot-royxati)
- [7. Analytics Dashboard](#7-analytics-dashboard)
- [8. Offline-first Note PWA](#8-offline-first-note-pwa)

---

## 1. Twitter-style News Feed

**Yondashuv.** Talablar: cheksiz scroll, like/retweet, media, real-time yangi post banneri. Non-funksional: LCP < 2.5s, mobil-birinchi, 10M foydalanuvchi. Eng katta texnik xavf — DOM o'sib ketishi va scroll performance'i, shuning uchun **virtualizatsiya** markaziy qaror.

**Arxitektura diagrammasi (matn):**

```
                 ┌────────────────────────────┐
                 │   FeedPage (route)          │
                 │   useInfiniteQuery("feed")  │
                 └──────────┬─────────────────┘
                            │ pages[] (cursor)
              ┌─────────────┼───────────────────┐
              ▼             ▼                   ▼
     ┌────────────┐  ┌─────────────┐    ┌──────────────┐
     │NewPosts    │  │ VirtualList │    │ Sentinel div │
     │Banner (SSE)│  │ (windowing) │    │ (IO observer)│
     └────────────┘  └──────┬──────┘    └──────┬───────┘
                            │                  │ onReachEnd
                     ┌──────▼──────┐           ▼ fetchNextPage
                     │ PostCard    │      [keyingi cursor]
                     │ (memo)      │
                     └─────────────┘

Data oqimi:
  API: GET /feed?cursor=&limit=20 -> { items, nextCursor }
  Normalizatsiya: posts{id->Post}, users{id->User}, feedOrder[]
  Real-time: SSE /feed/updates -> newCount++ (auto-insert YO'Q)
```

**Kod — infinite query + virtualizatsiya + optimistic like:**

```tsx
import { useInfiniteQuery, useMutation, useQueryClient } from "@tanstack/react-query";
import { useVirtualizer } from "@tanstack/react-virtual";
import { useRef, useEffect } from "react";

interface Post { id: string; text: string; likeCount: number; likedByMe: boolean; }
interface FeedPage { items: Post[]; nextCursor: string | null; }

function FeedPage() {
  const { data, fetchNextPage, hasNextPage, isFetchingNextPage } =
    useInfiniteQuery<FeedPage>({
      queryKey: ["feed"],
      queryFn: ({ pageParam }) =>
        fetch(`/api/feed?cursor=${pageParam ?? ""}&limit=20`).then((r) => r.json()),
      initialPageParam: "",
      getNextPageParam: (last) => last.nextCursor, // null -> oxiri
      staleTime: 60_000,
    });

  const posts = data?.pages.flatMap((p) => p.items) ?? [];
  const parentRef = useRef<HTMLDivElement>(null);

  // Virtualizatsiya: faqat ko'rinadigan postlar DOM'da
  const rowVirtualizer = useVirtualizer({
    count: posts.length,
    getScrollElement: () => parentRef.current,
    estimateSize: () => 180,
    overscan: 5,
  });

  // Oxiriga yetganda keyingi sahifa
  useEffect(() => {
    const last = rowVirtualizer.getVirtualItems().at(-1);
    if (last && last.index >= posts.length - 3 && hasNextPage && !isFetchingNextPage) {
      fetchNextPage();
    }
  }, [rowVirtualizer.getVirtualItems(), hasNextPage, isFetchingNextPage]);

  return (
    <div ref={parentRef} style={{ height: "100vh", overflow: "auto" }}>
      <div style={{ height: rowVirtualizer.getTotalSize(), position: "relative" }}>
        {rowVirtualizer.getVirtualItems().map((vItem) => (
          <div key={posts[vItem.index].id}
            style={{ position: "absolute", top: 0, transform: `translateY(${vItem.start}px)`, width: "100%" }}>
            <PostCard post={posts[vItem.index]} />
          </div>
        ))}
      </div>
    </div>
  );
}

// Optimistic like
function useLike() {
  const qc = useQueryClient();
  return useMutation({
    mutationFn: (id: string) => fetch(`/api/posts/${id}/like`, { method: "POST" }),
    onMutate: async (id) => {
      await qc.cancelQueries({ queryKey: ["feed"] });
      const prev = qc.getQueryData(["feed"]);
      qc.setQueryData<{ pages: FeedPage[] }>(["feed"], (old) => old && ({
        ...old,
        pages: old.pages.map((pg) => ({
          ...pg,
          items: pg.items.map((p) => p.id === id
            ? { ...p, likedByMe: !p.likedByMe, likeCount: p.likeCount + (p.likedByMe ? -1 : 1) }
            : p),
        })),
      }));
      return { prev };
    },
    onError: (_e, _id, ctx) => qc.setQueryData(["feed"], ctx?.prev),
    onSettled: () => qc.invalidateQueries({ queryKey: ["feed"] }),
  });
}

const PostCard = /* memo */ ({ post }: { post: Post }) => (
  <article aria-label="post">{post.text} — ❤ {post.likeCount}</article>
);
```

**Real-time banner (SSE, avtomatik kiritmasdan):**

```ts
useEffect(() => {
  const es = new EventSource("/api/feed/updates");
  es.onmessage = () => setNewCount((c) => c + 1); // foydalanuvchi tugmani bossa qo'shamiz
  return () => es.close();
}, []);
```

**Trade-off'lar va edge case'lar:** cursor pagination (offset emas) — yangi post qo'shilsa siljish yo'q. Yangi postni avtomatik kiritmaymiz — scroll pozitsiyasini buzmaslik uchun banner orqali. Edge: bo'sh feed (empty state), xato (retry + skeleton), dedup by ID, orqaga qaytishda scroll restore (`scrollRestoration`).

---

## 2. Autocomplete / Typeahead

**Yondashuv.** Talablar: yozilganda takliflar, klaviatura tanlash, tez, a11y. Texnik xavflar: har harfga so'rov (debounce kerak), race condition (eski javob yangisini bosib o'tishi), takroriy so'rov (cache).

**Arxitektura diagrammasi (matn):**

```
  [input] --onChange--> query state
       │ debounce 280ms
       ▼
  useSearch(query):
    cache.has(query)? --> natija (so'rovsiz)
       │ yo'q
       ▼
    AbortController(eski.abort()) -> fetch(/search?q) -> setResults
       │
       ▼
  combobox (ARIA): aria-expanded, aria-activedescendant
       │ ↑/↓ Enter Esc
       ▼
  listbox > option[id=opt-i]
```

**Kod — debounce + abort + cache + klaviatura + ARIA:**

```tsx
import { useState, useEffect, useRef, useCallback } from "react";

function useDebounced<T>(value: T, ms: number) {
  const [v, setV] = useState(value);
  useEffect(() => {
    const t = setTimeout(() => setV(value), ms);
    return () => clearTimeout(t);
  }, [value, ms]);
  return v;
}

interface Item { id: string; label: string; }

function Autocomplete({ onSelect }: { onSelect: (i: Item) => void }) {
  const [query, setQuery] = useState("");
  const [items, setItems] = useState<Item[]>([]);
  const [active, setActive] = useState(-1);
  const [open, setOpen] = useState(false);
  const cache = useRef(new Map<string, Item[]>());
  const debounced = useDebounced(query, 280);

  useEffect(() => {
    if (!debounced) { setItems([]); return; }
    if (cache.current.has(debounced)) { setItems(cache.current.get(debounced)!); return; }
    const ctrl = new AbortController();
    fetch(`/api/search?q=${encodeURIComponent(debounced)}`, { signal: ctrl.signal })
      .then((r) => r.json())
      .then((data: Item[]) => { cache.current.set(debounced, data); setItems(data); setActive(-1); })
      .catch((e) => { if (e.name !== "AbortError") setItems([]); });
    return () => ctrl.abort(); // race condition'ga qarshi: yangi query eskisini bekor qiladi
  }, [debounced]);

  const onKeyDown = useCallback((e: React.KeyboardEvent) => {
    if (e.key === "ArrowDown") { e.preventDefault(); setActive((a) => Math.min(a + 1, items.length - 1)); }
    else if (e.key === "ArrowUp") { e.preventDefault(); setActive((a) => Math.max(a - 1, 0)); }
    else if (e.key === "Enter" && active >= 0) { onSelect(items[active]); setOpen(false); }
    else if (e.key === "Escape") setOpen(false);
  }, [items, active, onSelect]);

  return (
    <div role="combobox" aria-expanded={open} aria-haspopup="listbox" aria-owns="ac-list">
      <input
        value={query}
        onChange={(e) => { setQuery(e.target.value); setOpen(true); }}
        onKeyDown={onKeyDown}
        aria-autocomplete="list"
        aria-controls="ac-list"
        aria-activedescendant={active >= 0 ? `opt-${active}` : undefined}
      />
      {open && items.length > 0 && (
        <ul id="ac-list" role="listbox">
          {items.map((it, i) => (
            <li key={it.id} id={`opt-${i}`} role="option" aria-selected={i === active}
              onMouseDown={() => { onSelect(it); setOpen(false); }}
              style={{ background: i === active ? "#eee" : undefined }}>
              {it.label}
            </li>
          ))}
        </ul>
      )}
    </div>
  );
}
```

**Trade-off'lar va edge case'lar:** debounce 280ms — tezlik vs so'rov soni balansi. `onMouseDown` (`onClick` emas) — input blur'dan oldin ishlashi uchun. Edge: bo'sh natija ("topilmadi"), tarmoq xatosi, juda tez yozish (abort hammasini hal qiladi), tashqariga bosish (blur), mobil klaviatura (`enterKeyHint="search"`). a11y: to'liq `combobox` ARIA pattern, `aria-activedescendant` bilan virtual focus.

---

## 3. Real-time Guruh Chat

**Yondashuv.** Talablar: real-time xabar, holatlar (yuborilmoqda/yuborildi/o'qildi), reconnect, offline navbat, tarix. **WebSocket** tanlanadi — chat ikki yo'nalishli (SSE faqat server→client). Asosiy xavflar: ulanish uzilishi, xabar tartibi, dublikat.

**Arxitektura diagrammasi (matn):**

```
  ┌──────────────┐   send    ┌──────────────────┐
  │ ChatInput    ├──────────>│ ChatSocket       │
  └──────────────┘           │ - reconnect(bckf)│
                             │ - heartbeat ping │
  ┌──────────────┐   recv    │ - offline queue  │<──IndexedDB
  │ MessageList  │<──────────┤ - dedup by id    │
  │ (virtual)    │           └────────┬─────────┘
  └──────────────┘                    │ ws://
                                      ▼
                                 [Chat Server]

  Xabar holati: pending --ACK--> sent --read receipt--> read
  Tartib: server seq/timestamp bo'yicha sort; clientId bilan dedup
```

**Kod — socket menejeri (reconnect + offline queue):**

```ts
type Status = "pending" | "sent" | "read";
interface Message { id: string; clientId: string; text: string; ts: number; status: Status; }

class ChatSocket {
  private ws: WebSocket | null = null;
  private attempt = 0;
  private queue: Message[] = []; // offline navbat (IndexedDB'ga ham yoziladi)
  private heartbeat?: ReturnType<typeof setInterval>;

  constructor(private url: string, private onMessage: (m: Message) => void) {}

  connect() {
    this.ws = new WebSocket(this.url);
    this.ws.onopen = () => {
      this.attempt = 0;
      this.startHeartbeat();
      this.flushQueue(); // tiklanganda navbatdagilarni yubor
    };
    this.ws.onmessage = (e) => this.onMessage(JSON.parse(e.data));
    this.ws.onclose = () => { this.stopHeartbeat(); this.reconnect(); };
    this.ws.onerror = () => this.ws?.close();
  }

  private reconnect() {
    const delay = Math.min(1000 * 2 ** this.attempt++, 30_000); // exponential backoff
    setTimeout(() => this.connect(), delay);
  }

  private startHeartbeat() {
    this.heartbeat = setInterval(() => this.ws?.send(JSON.stringify({ type: "ping" })), 15_000);
  }
  private stopHeartbeat() { if (this.heartbeat) clearInterval(this.heartbeat); }

  send(msg: Message) {
    if (this.ws?.readyState === WebSocket.OPEN) {
      this.ws.send(JSON.stringify({ type: "msg", ...msg }));
    } else {
      this.queue.push(msg);          // offline -> navbatga
      persistQueue(this.queue);      // IndexedDB
    }
  }
  private flushQueue() {
    this.queue.forEach((m) => this.ws?.send(JSON.stringify({ type: "msg", ...m })));
    this.queue = [];
    persistQueue(this.queue);
  }
}

declare function persistQueue(q: Message[]): void;
```

**Kod — optimistic UI + dedup + tartib (React tomon):**

```tsx
function useChat(socket: ChatSocket) {
  const [messages, setMessages] = useState<Message[]>([]);

  const upsert = useCallback((incoming: Message) => {
    setMessages((prev) => {
      // dedup: clientId yoki id bo'yicha mavjudini yangilash
      const idx = prev.findIndex((m) => m.clientId === incoming.clientId || m.id === incoming.id);
      const next = idx >= 0 ? prev.map((m, i) => (i === idx ? incoming : m)) : [...prev, incoming];
      return next.sort((a, b) => a.ts - b.ts); // server timestamp bo'yicha tartib
    });
  }, []);

  const sendMessage = useCallback((text: string) => {
    const optimistic: Message = {
      id: "", clientId: crypto.randomUUID(), text, ts: Date.now(), status: "pending",
    };
    upsert(optimistic);          // darhol "pending" ko'rsat
    socket.send(optimistic);     // ACK kelganda status="sent" bo'lib upsert qilinadi
  }, [socket, upsert]);

  return { messages, sendMessage };
}
```

**Trade-off'lar va edge case'lar:** WebSocket vs SSE — chat ikki yo'nalishli bo'lgani uchun WebSocket. Heartbeat — o'lik ulanishni aniqlash (NAT/proxy timeout). Exponential backoff — serverni reconnect bo'roniga uchratmaslik. Edge: dublikat (clientId dedup), tartibsiz kelish (timestamp sort), katta tarix (virtualizatsiya + cursor pagination yuqoriga), typing indikatori (throttle), o'qildi statusi (read receipt event).

---

## 4. Image Gallery / Masonry

**Yondashuv.** Talablar: Pinterest-style masonry, lazy load, zamonaviy formatlar, LQIP, CLS ≈ 0, virtualizatsiya. Asosiy xavf — CLS (rasm yuklanganda layout siljishi) va xotira (minglab rasm).

**Arxitektura diagrammasi (matn):**

```
  GalleryPage (useInfiniteQuery)
   └─ MasonryGrid (CSS columns yoki absolute layout)
       └─ ImageCard
           ├─ aspect-ratio box (CLS=0, o'lcham oldindan)
           ├─ LQIP blur placeholder (base64)
           ├─ <picture> AVIF/WebP/JPEG + srcset
           └─ loading="lazy" (off-screen)
  Virtualizatsiya: faqat ko'rinadigan ustun-bo'laklari render
  CDN: image transform (?w=400&fmt=avif)
```

**Kod — CLS-siz, zamonaviy formatli, LQIP rasm karta:**

```tsx
interface Img { id: string; w: number; h: number; lqip: string; baseUrl: string; alt: string; }

function ImageCard({ img }: { img: Img }) {
  const [loaded, setLoaded] = useState(false);
  return (
    // aspect-ratio CLS'ni nolga tushiradi — joy oldindan band qilinadi
    <div style={{ aspectRatio: `${img.w} / ${img.h}`, position: "relative", background: "#eee" }}>
      {/* LQIP — blur placeholder, asl rasm yuklangunча */}
      <img src={img.lqip} alt="" aria-hidden
        style={{ position: "absolute", inset: 0, width: "100%", height: "100%",
                 filter: "blur(12px)", opacity: loaded ? 0 : 1, transition: "opacity .3s" }} />
      <picture>
        <source srcSet={`${img.baseUrl}?w=400&fmt=avif 400w, ${img.baseUrl}?w=800&fmt=avif 800w`}
          sizes="(max-width: 600px) 50vw, 25vw" type="image/avif" />
        <source srcSet={`${img.baseUrl}?w=400&fmt=webp 400w`} type="image/webp" />
        <img src={`${img.baseUrl}?w=400`} alt={img.alt}
          width={img.w} height={img.h}
          loading="lazy" decoding="async"
          onLoad={() => setLoaded(true)}
          style={{ width: "100%", height: "100%", objectFit: "cover" }} />
      </picture>
    </div>
  );
}
```

**Trade-off'lar va edge case'lar:** `aspect-ratio` + `width/height` — CLS=0 ning kaliti. AVIF birinchi (eng kichik), WebP/JPEG fallback. `srcset`/`sizes` — qurilmaga mos o'lcham (band kengligini tejaydi). Birinchi ekrandagi rasmlarga `loading="lazy"` QO'YILMAYDI (LCP). Masonry virtualizatsiyasi murakkab (notekis balandlik) — `react-virtualized Masonry` yoki absolute pozitsiyalash. Performance budget: hero rasm < 200KB, jami initial < 1MB.

---

## 5. Kollaborativ Editor

**Yondashuv.** Talablar: real-time birgalikda tahrir, konflikt hal, kursorlar, offline tahrir, sinxronlash. Markaziy qaror — konkurent tahrirni qanday birlashtirish: **OT (Operational Transformation)** yoki **CRDT (Conflict-free Replicated Data Type)**. Zamonaviy tanlov ko'pincha CRDT (Yjs) — markazsiz, offline-do'st.

**Arxitektura diagrammasi (matn):**

```
  ClientA ──ops──┐                  ┌── ops── ClientB
                 ▼                  ▼
            ┌─────────────────────────────┐
            │  Sync Server (relay)        │
            │  - broadcast ops            │
            │  - awareness (kursorlar)    │
            └─────────────────────────────┘
                 │ persist
                 ▼
            [Document store]

  CRDT (Yjs):  local op -> apply -> encode update -> broadcast
               remote update -> merge (konfliktsiz, commutative)
  Awareness:   kursor/selection -> ephemeral state (saqlanmaydi)
  Offline:     local ops navbatda -> reconnect -> merge
```

**Kod — CRDT (Yjs) konseptual integratsiya:**

```ts
import * as Y from "yjs";
import { WebsocketProvider } from "y-websocket";

const ydoc = new Y.Doc();
// WebSocket provider: ops'ni broadcast qiladi, offline'da local saqlaydi
const provider = new WebsocketProvider("wss://sync.example.com", "doc-123", ydoc);
const ytext = ydoc.getText("content");

// Lokal tahrir -> avtomatik broadcast (CRDT merge konfliktsiz)
ytext.insert(0, "Salom dunyo");

// Remote o'zgarishlarni kuzatish
ytext.observe(() => render(ytext.toString()));

// Awareness — boshqalarning kursori (ephemeral, saqlanmaydi)
provider.awareness.setLocalStateField("cursor", { index: 5, user: "Ali", color: "#f00" });
provider.awareness.on("change", () => renderRemoteCursors(provider.awareness.getStates()));

// Offline-first: provider ulanish uzilsa ops'ni saqlaydi, tiklanganda merge qiladi
provider.on("status", ({ status }: { status: string }) => {
  showConnectionState(status); // "connected" | "disconnected"
});

declare function render(s: string): void;
declare function renderRemoteCursors(states: Map<number, any>): void;
declare function showConnectionState(s: string): void;
```

**Trade-off'lar:** **OT** — server avtoritar, transform logikasi murakkab, lekin matn uchun yetuk (Google Docs). **CRDT** — markazsiz, offline tabiiy, lekin metadata o'sishi (garbage collection kerak) va xotira og'irligi. Edge: katta hujjat (incremental render, faqat ko'rinadigan qism), kursor jiggle (throttle awareness), ulanish uzilishi (local-first shuni hal qiladi), undo/redo (CRDT-aware undo manager). a11y: `aria-live` bilan remote o'zgarishlarni mo''tadil e'lon qilish.

---

## 6. E-commerce Mahsulot Ro'yxati

**Yondashuv.** Talablar: SEO muhim (rendering tanlovi kalit), filtr/sort (URL state), pagination, rasm optimizatsiya, CWV budget, A/B test. **Rendering: ISR yoki Streaming SSR** — SEO kerak (SSR/SSG) + katalog tez-tez o'zgaradi (ISR/streaming).

**Arxitektura diagrammasi (matn):**

```
  Request -> Edge (CDN cache) -> SSR/ISR sahifa
       │
       ▼
  ProductListPage (server component, RSC)
   ├─ Filters (URL search params: ?cat=&price=&sort=)
   │      └─ navigate -> yangi URL -> server re-render (cache key)
   ├─ ProductGrid (streaming <Suspense>)
   │      └─ ProductCard (optimized <Image>, priority hero)
   └─ Pagination (cursor/offset, URL)

  A/B test: edge middleware -> variant cookie -> bucket
  CWV: LCP rasm priority, font preload, CLS=0 (aspect-ratio)
```

**Kod — URL state filtrlash (Next.js App Router):**

```tsx
"use client";
import { useRouter, useSearchParams, usePathname } from "next/navigation";

function Filters() {
  const router = useRouter();
  const pathname = usePathname();
  const params = useSearchParams();

  function setFilter(key: string, value: string) {
    const next = new URLSearchParams(params.toString());
    value ? next.set(key, value) : next.delete(key);
    next.delete("cursor"); // filtr o'zgarsa paginatsiyani reset
    router.push(`${pathname}?${next.toString()}`); // URL state -> shareable, SSR cache key
  }

  return (
    <>
      <select value={params.get("sort") ?? ""} onChange={(e) => setFilter("sort", e.target.value)}>
        <option value="">Saralash</option>
        <option value="price_asc">Narx ↑</option>
        <option value="price_desc">Narx ↓</option>
      </select>
    </>
  );
}
```

```tsx
// Server component — ISR bilan, SEO uchun tayyor HTML
export const revalidate = 60; // ISR: 60s da regenerate

export default async function ProductListPage({
  searchParams,
}: { searchParams: { cat?: string; sort?: string; cursor?: string } }) {
  const products = await fetchProducts(searchParams); // server'da
  return (
    <main>
      <Filters />
      <Suspense fallback={<GridSkeleton />}>
        <ProductGrid products={products} />
      </Suspense>
    </main>
  );
}
```

**Trade-off'lar:** SSG — eng tez lekin million SKU build'ni o'ldiradi; **ISR** — balans (popular sahifalar cache, qolgani on-demand). URL state — filtrlar shareable, brauzer back ishlaydi, SSR cache key bo'ladi (Redux state buni bermaydi). A/B test — edge middleware'da variant cookie. CWV budget: LCP < 2.5s (hero `priority`), CLS < 0.1 (aspect-ratio), JS < 170KB. Edge: bo'sh natija, noto'g'ri filtr kombinatsiyasi, deep-link (URL'dan to'g'ridan ochish).

---

## 7. Analytics Dashboard

**Yondashuv.** Talablar: ko'p widget, og'ir grafik, real-time, katta dataset, INP saqlash. Markaziy xavf — og'ir chart kutubxonasi bundle'ni va main thread'ni o'ldiradi. Yechim: **lazy load chart + web worker + virtualizatsiya + server aggregation**.

**Arxitektura diagrammasi (matn):**

```
  DashboardPage (grid layout, lazy widgets)
   ├─ KpiCards (yengil, darhol)
   ├─ <Suspense> ChartWidget (lazy import heavy-chart)
   │      └─ data: useQuery(server-aggregated, NOT raw)
   ├─ DataTable (virtualizatsiya, ming qator)
   └─ live updates: SSE yoki polling (interval)

  Og'ir hisob -> Web Worker (main thread bloklanmaydi -> INP saqlanadi)
  Server: pre-aggregation (raw 1M qator emas, summary)
```

**Kod — lazy chart + web worker aggregation:**

```tsx
import { lazy, Suspense } from "react";
import { useQuery } from "@tanstack/react-query";

// Og'ir chart kutubxonasi faqat kerak bo'lganda yuklanadi (code splitting)
const HeavyChart = lazy(() => import("./HeavyChart"));

function DashboardPage() {
  return (
    <div className="grid">
      <KpiCards />
      <Suspense fallback={<ChartSkeleton />}>
        <RevenueChart />
      </Suspense>
    </div>
  );
}

function RevenueChart() {
  // Server allaqachon aggregate qilgan (raw 1M qator emas)
  const { data } = useQuery({
    queryKey: ["revenue", "daily"],
    queryFn: () => fetch("/api/analytics/revenue?granularity=day").then((r) => r.json()),
    refetchInterval: 30_000, // polling: 30s
  });
  return <HeavyChart data={data ?? []} />;
}
```

```ts
// Og'ir client-side hisob -> Web Worker (INP saqlash uchun main thread bo'sh qoladi)
// worker.ts
self.onmessage = (e: MessageEvent<number[]>) => {
  const result = heavyAggregate(e.data); // bloklamaydi, alohida thread
  self.postMessage(result);
};

// main.ts
const worker = new Worker(new URL("./worker.ts", import.meta.url), { type: "module" });
worker.postMessage(rawData);
worker.onmessage = (e) => setComputed(e.data);
```

**Trade-off'lar:** Lazy chart — initial bundle kichik, lekin widget ochilganda kichik kechikish (skeleton bilan yashiriladi). Web worker — INP'ni saqlaydi (og'ir hisob main thread'ni bloklamaydi) lekin serialization narxi bor. Server aggregation > client aggregation (1M qatorni client'ga yubormaslik). Polling vs SSE — dashboard uchun ko'pincha polling yetarli (oddiyroq). Edge: ko'p widget bir vaqtda yuklanishi (request batching), stale data ko'rsatish (stale-while-revalidate), katta jadval (virtualizatsiya).

---

## 8. Offline-first Note PWA

**Yondashuv.** Talablar: SW, IndexedDB lokal saqlash, background sync, konflikt hal, serverga sync, o'rnatish oqimi. Markaziy prinsip — **local-first**: hamma yozish avval lokal (IndexedDB), keyin fonda serverga sync. UI hech qachon tarmoqni kutmaydi.

**Arxitektura diagrammasi (matn):**

```
  NoteEditor --write--> IndexedDB (local, darhol) --UI yangilanadi
                            │
                            ▼ (online bo'lganda)
                   Background Sync queue
                            │
                            ▼
                  Service Worker --POST--> Server
                            │
                   Konflikt? (updatedAt taqqoslash / version)
                     ├─ local newer -> push
                     └─ remote newer -> merge/last-write-wins

  Install: beforeinstallprompt -> "Ilovani o'rnatish" tugmasi
  Offline: SW app shell cache -> ilova ochiladi
```

**Kod — local-first yozish + sync (IndexedDB orqali):**

```ts
import { openDB } from "idb";

const dbPromise = openDB("notes-db", 1, {
  upgrade(db) {
    db.createObjectStore("notes", { keyPath: "id" });
    db.createObjectStore("syncQueue", { keyPath: "id" });
  },
});

interface Note { id: string; text: string; updatedAt: number; synced: boolean; }

// Yozish: avval lokal (darhol), keyin sync navbatiga
async function saveNote(note: Omit<Note, "synced">) {
  const db = await dbPromise;
  await db.put("notes", { ...note, synced: false }); // UI shu zahoti yangilanadi
  await db.put("syncQueue", { id: note.id });
  if (navigator.onLine) syncNow();
  else await registerBackgroundSync(); // ulanish tiklanganda SW yuboradi
}

async function syncNow() {
  const db = await dbPromise;
  const queued = await db.getAll("syncQueue");
  for (const { id } of queued) {
    const note = await db.get("notes", id);
    const res = await fetch(`/api/notes/${id}`, {
      method: "PUT",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify(note),
    });
    if (res.ok) {
      const server: Note = await res.json();
      // Konflikt: oxirgi yangilangan g'olib (yoki merge strategiyasi)
      const winner = server.updatedAt > note.updatedAt ? server : note;
      await db.put("notes", { ...winner, synced: true });
      await db.delete("syncQueue", id);
    }
  }
}

async function registerBackgroundSync() {
  const reg = await navigator.serviceWorker.ready;
  // @ts-expect-error: sync API
  await reg.sync.register("sync-notes");
}
```

```ts
// service-worker.ts — background sync va app shell
self.addEventListener("sync", (event: any) => {
  if (event.tag === "sync-notes") event.waitUntil(syncNow());
});

// App shell cache-first (offline'da ilova ochiladi)
self.addEventListener("fetch", (event: any) => {
  if (event.request.mode === "navigate") {
    event.respondWith(caches.match("/app-shell.html").then((r) => r ?? fetch(event.request)));
  }
});
```

```tsx
// Install oqimi
function InstallButton() {
  const [prompt, setPrompt] = useState<any>(null);
  useEffect(() => {
    const handler = (e: any) => { e.preventDefault(); setPrompt(e); };
    window.addEventListener("beforeinstallprompt", handler);
    return () => window.removeEventListener("beforeinstallprompt", handler);
  }, []);
  if (!prompt) return null;
  return <button onClick={() => prompt.prompt()}>Ilovani o'rnatish</button>;
}
```

**Trade-off'lar:** Local-first — UI darhol javob beradi (tarmoqni kutmaydi) lekin konflikt hal qilish kerak. Last-write-wins — oddiy lekin ma'lumot yo'qotishi mumkin; muhim data uchun merge yoki CRDT. Background Sync — ulanish tiklanganda avtomatik, lekin brauzer qo'llab-quvvatlashi cheklangan (fallback: online event). Edge: bir vaqtda ikki qurilmadan tahrir (version vector), katta note (debounce save), IndexedDB kvota tugashi, SW yangilanish strategiyasi (skipWaiting ehtiyotkorlik bilan).

---

← [Frontend bo'limiga qaytish](../../frontend/README.md)
