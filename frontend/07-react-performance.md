# React Performance

React'ni tez qilish — bu aslida *kamroq ish bajarish* san'ati: kamroq re-render, kichikroq bundle va main thread'ni hech qachon bloklamaslik. Senior darajada performance — bu "hamma joyni memoize qilish" emas, balki qayerda muammo borligini o'lchash, sababini tushunish va to'g'ri trade-off bilan hal qilish.

**💡 Tushuncha:** Premature optimization React'da real muammo. Avval o'lcha (Profiler), keyin optimallashtir. Memoization ham bepul emas — uning o'z narxi bor.

## Mundarija

- [Render, commit, paint fazalari](#render-commit-paint-fazalari)
- [Nima re-render keltiradi](#nima-re-render-keltiradi)
- [Re-render'larni topish: DevTools Profiler](#re-renderlarni-topish-devtools-profiler)
- [Referential equality](#referential-equality)
- [React.memo](#reactmemo)
- [useMemo va useCallback](#usememo-va-usecallback)
- [Inline object va function'dan qochish](#inline-object-va-functiondan-qochish)
- [Key barqarorligi](#key-barqarorligi)
- [State colocation](#state-colocation)
- [Context performance muammolari](#context-performance-muammolari)
- [useTransition](#usetransition)
- [Debounce va throttle](#debounce-va-throttle)
- [List virtualization](#list-virtualization)
- [Code splitting, React.lazy va Suspense](#code-splitting-reactlazy-va-suspense)
- [Bundle size va tree shaking](#bundle-size-va-tree-shaking)
- [Lazy loading](#lazy-loading)
- [Web Vitals](#web-vitals)
- [Server Components (qisqacha)](#server-components-qisqacha)
- [Savol-javoblar](#savol-javoblar)
- [Masalalar](#masalalar)

---

## Render, commit, paint fazalari

React UI'ni yangilashda uch fazadan o'tadi. Bularni ajrata bilish performance haqida to'g'ri o'ylash uchun zarur.

1. **Render (render phase)** — React komponent funksiyalarini chaqiradi va yangi virtual DOM (React element daraxti) hosil qiladi. Bu **toza** (pure) bo'lishi kerak: side effect yo'q. Eski va yangi daraxt **reconciliation** orqali solishtiriladi. Bu faza concurrent rejimda to'xtatilishi (interruptible) mumkin.
2. **Commit (commit phase)** — React aniqlangan farqlarni real DOM'ga qo'llaydi. `useLayoutEffect` shu yerda sinxron ishlaydi. Bu faza to'xtatilmaydi.
3. **Paint (browser paint)** — brauzer DOM'ni ekranga chizadi. `useEffect` odatda paint'dan keyin ishlaydi.

```tsx
function Counter() {
  const [count, setCount] = useState(0);
  // render phase: bu funksiya har setCount'da qayta chaqiriladi
  console.log("render");
  useLayoutEffect(() => {
    // commit phase: DOM tayyor, lekin paint hali yo'q
  });
  useEffect(() => {
    // paint'dan keyin
  });
  return <button onClick={() => setCount((c) => c + 1)}>{count}</button>;
}
```

**💡 Tushuncha:** "Re-render" — bu komponent funksiyasi qayta chaqirilishi, *real DOM yangilanishi emas*. Render bo'lishi mumkin, lekin DOM o'zgarmasligi mumkin (reconciliation farq topmasa). Shuning uchun "re-render = sekin" degani noto'g'ri — re-render odatda arzon. Muammo — *ortiqcha* yoki *qimmat* render.

---

## Nima re-render keltiradi

Komponent quyidagi hollarda re-render bo'ladi:

1. **State o'zgarishi** — `setState` (`useState`/`useReducer`) chaqirilganda (yangi qiymat eskisidan `Object.is` bo'yicha farq qilsa).
2. **Props o'zgarishi** — ota-ona (parent) re-render bo'lganda, bola (child) ham re-render bo'ladi — **propslar o'zgarmagan bo'lsa ham** (memo bo'lmasa).
3. **Context o'zgarishi** — komponent `useContext` orqali iste'mol qilgan context qiymati o'zgarsa.
4. **Parent re-render** — eng ko'p uchraydigan sabab. Parent render bo'lsa, butun child daraxti render bo'ladi (memo bilan to'xtatilmasa).

**⚠️ Ehtiyot bo'l:** Ko'pchilik "props o'zgarmasa, child render bo'lmaydi" deb o'ylaydi — bu xato. React default holda parent render'da barcha child'larni qayta render qiladi. Buni faqat `React.memo` to'xtatadi.

```tsx
function Parent() {
  const [count, setCount] = useState(0);
  return (
    <div>
      <button onClick={() => setCount((c) => c + 1)}>{count}</button>
      <Child /> {/* props yo'q, lekin har count o'zgarishida render bo'ladi */}
    </div>
  );
}
```

---

## Re-render'larni topish: DevTools Profiler

Optimizatsiyadan oldin **o'lchang**. React DevTools'da:

1. **Profiler** tab → record → interfeys bilan ishlang → stop.
2. **Flame graph** — qaysi komponent qancha vaqt render bo'lganini ko'rsatadi (kenglik = vaqt).
3. **Ranked chart** — komponentlarni render vaqti bo'yicha tartiblaydi.
4. **"Highlight updates when components render"** (Components tab sozlamasi) — render bo'layotgan komponentlar ekranda yashil ramka bilan miltillaydi. Tez topish uchun zo'r.
5. **"Record why each component rendered"** (Profiler sozlamasi) — props/state/hooks/parent'dan qaysi biri sababchi ekanini ko'rsatadi.

**💡 Tushuncha:** Production build'da yoki kamida CPU throttling yoqib o'lchang (DevTools → Performance → 4x/6x slowdown). Development build sekin va realistik emas.

---

## Referential equality

React qiymatlarni `Object.is` (taxminan `===`) bilan solishtiradi. Primitivlar (`number`, `string`, `boolean`) qiymat bo'yicha teng. Lekin **object, array, function** har render'da yangi reference oladi.

```tsx
const a = { x: 1 };
const b = { x: 1 };
a === b; // false — boshqa reference

const fn1 = () => {};
const fn2 = () => {};
fn1 === fn2; // false
```

Shuning uchun:

```tsx
function Parent() {
  // har render'da YANGI object/function — child'ga uzatilsa, memo ishlamaydi
  const style = { color: "red" };
  const handleClick = () => doSomething();
  return <Child style={style} onClick={handleClick} />;
}
```

**⚠️ Ehtiyot bo'l:** Referential equality — React performance'ning markaziy tushunchasi. `React.memo`, `useMemo`, `useCallback`, dependency array'lar — barchasi shu solishtirishga tayanadi. Bu mexanikani tushunmasangiz, memoization tasodifiy ishlaydi.

---

## React.memo

`React.memo` — komponentni o'rab, props **shallow** (yuzaki) o'zgarmagan bo'lsa, re-render'ni to'xtatadigan HOC.

```tsx
interface ItemProps {
  label: string;
  onSelect: (id: string) => void;
}

const Item = React.memo(function Item({ label, onSelect }: ItemProps) {
  return <li onClick={() => onSelect(label)}>{label}</li>;
});
```

Endi parent render bo'lganda, agar `label` va `onSelect` reference'lari o'zgarmasa, `Item` render bo'lmaydi.

**Qachon foyda:**
- Komponent tez-tez render bo'ladigan parent ostida.
- Render qimmat yoki child'lar ko'p (katta ro'yxat elementi).
- Props barqaror (primitiv yoki memoize qilingan).

**Qachon zarar / foydasiz:**
- Props har render'da o'zgaradi (memo har safar solishtirib, baribir render qiladi → faqat ortiqcha solishtirish narxi).
- Komponent baribir kam render bo'ladi.
- Props ichida har safar yangi object/function bor (memo'ni "buzadi").

**⚠️ Ehtiyot bo'l:** `React.memo` bepul emas — u har render'da props'ni solishtiradi. Agar props har doim o'zgarsa, siz solishtirish narxini qo'shdingiz, foyda olmadingiz. Hamma joyni memo qilish — anti-pattern.

Maxsus solishtirish funksiyasi:

```tsx
const Item = React.memo(ItemComponent, (prev, next) => {
  return prev.id === next.id; // true qaytsa, render bo'lmaydi
});
```

---

## useMemo va useCallback

- **`useMemo`** — qiymatni (hisoblash natijasini) cache qiladi.
- **`useCallback`** — funksiyani cache qiladi. `useCallback(fn, deps) === useMemo(() => fn, deps)`.

```tsx
function Search({ items, query }: { items: Item[]; query: string }) {
  // qimmat filtrlash — faqat items yoki query o'zgarganda qayta hisoblanadi
  const filtered = useMemo(
    () => items.filter((i) => i.name.includes(query)),
    [items, query]
  );

  // referential equality saqlash — memo qilingan child'ga uzatish uchun
  const onSelect = useCallback((id: string) => {
    console.log(id);
  }, []);

  return <List items={filtered} onSelect={onSelect} />;
}
```

**`useMemo` qachon kerak:**
1. Hisoblash **haqiqatan qimmat** (katta massiv, og'ir hisob).
2. Natija **memo qilingan child**'ga yoki boshqa hook'ning dependency'siga uzatiladi (referential equality kerak).

**`useCallback` qachon kerak:**
- Funksiya `React.memo` child'ga yoki `useEffect`/`useMemo` dependency'siga uzatilsa.

**Ortiqcha memoization narxi:**
- Har `useMemo`/`useCallback` o'z dependency'sini saqlaydi va solishtiradi — bu xotira va vaqt.
- Kod o'qilishi qiyinlashadi.
- Agar child memo emas yoki natija hech qayerga uzatilmasa — memo **mutlaqo befoyda**, faqat narx.

```tsx
// ❌ Befoyda: oddiy hisob, hech qayerga uzatilmaydi
const total = useMemo(() => a + b, [a, b]);
// ✅ To'g'ri: oddiy hisobni shunchaki yozing
const total = a + b;
```

**💡 Tushuncha:** Qoida: `useMemo`/`useCallback`ni "yiltiroq bo'lsin" deb emas, *aniq sabab* bilan qo'shing — yo qimmat hisob, yo referential equality kerak. React Compiler (React 19) kelajakda buni avtomatlashtiradi, lekin hozircha qo'lda boshqarasiz.

---

## Inline object va function'dan qochish

Memo qilingan child'ga inline object/function uzatish memo'ni "buzadi":

```tsx
// ❌ har render'da yangi reference → MemoChild har safar render bo'ladi
<MemoChild style={{ margin: 8 }} onClick={() => save()} />

// ✅ barqaror reference
const style = useMemo(() => ({ margin: 8 }), []);
const onClick = useCallback(() => save(), []);
<MemoChild style={style} onClick={onClick} />
```

**⚠️ Ehtiyot bo'l:** Agar child memo EMAS bo'lsa, inline object/function performance'ga ta'sir qilmaydi — child baribir render bo'ladi. Shuning uchun inline'dan qochish faqat memo zanjiri kontekstida ma'noli. Oddiy DOM elementga inline `onClick` — muammo emas.

Statik object'larni komponentdan tashqariga chiqarish — eng arzon yechim:

```tsx
const STATIC_STYLE = { margin: 8 }; // modul darajasi — bir marta yaratiladi
function Row() {
  return <div style={STATIC_STYLE} />;
}
```

---

## Key barqarorligi

`key` React'ga ro'yxat elementlarini render'lar orasida moslashtirishga yordam beradi. Noto'g'ri key — performance va xatolik manbai.

```tsx
// ❌ index key — element o'rni o'zgarsa, React noto'g'ri moslaydi
{items.map((item, i) => <Row key={i} item={item} />)}

// ✅ barqaror, noyob id
{items.map((item) => <Row key={item.id} item={item} />)}
```

**Index key muammosi:** ro'yxat boshiga element qo'shilsa, barcha indekslar siljiydi. React `key={0}`ni eski `key={0}` bilan moslaydi → noto'g'ri DOM qayta ishlatiladi, input state aralashadi, ortiqcha DOM mutatsiya.

**⚠️ Ehtiyot bo'l:** `key`ni `Math.random()` bilan bermang — har render'da yangi key → React har elementni o'chirib qayta yaratadi (eng yomon performance + state yo'qoladi).

**💡 Tushuncha:** Key faqat ro'yxat ichida noyob va render'lar orasida barqaror bo'lishi kerak (global noyob shart emas). Barqaror key = arzon reconciliation + saqlangan child state.

---

## State colocation

Eng oddiy va kuchli optimizatsiya: **state'ni iloji boricha pastga, undan foydalanadigan komponentga yaqin joylashtiring** (colocation). Yuqorida turgan state butun pastdagi daraxtni render qiladi.

```tsx
// ❌ input state App'da → har bosishda butun App render
function App() {
  const [query, setQuery] = useState("");
  return (
    <>
      <input value={query} onChange={(e) => setQuery(e.target.value)} />
      <ExpensiveDashboard /> {/* query bilan aloqasi yo'q, lekin render bo'ladi */}
    </>
  );
}

// ✅ state'ni pastga ko'chirish
function SearchBox() {
  const [query, setQuery] = useState("");
  return <input value={query} onChange={(e) => setQuery(e.target.value)} />;
}
function App() {
  return (
    <>
      <SearchBox />
      <ExpensiveDashboard /> {/* endi render bo'lmaydi */}
    </>
  );
}
```

**Lifting content down (children sifatida uzatish):** state'ni pastga ko'chira olmasangiz, qimmat qismni `children` qilib uzating — u parent state o'zgarsa ham render bo'lmaydi.

```tsx
function Counter({ children }: { children: React.ReactNode }) {
  const [c, setC] = useState(0);
  return (
    <div onClick={() => setC(c + 1)}>
      {c}
      {children} {/* parent'da yaratilgan, Counter state'iga bog'liq emas */}
    </div>
  );
}
// <Counter><ExpensiveTree /></Counter> — ExpensiveTree render bo'lmaydi
```

**💡 Tushuncha:** Colocation ko'pincha memo'dan yaxshiroq — chunki muammoni *manbada* hal qiladi, qo'shimcha narx qo'shmaydi.

---

## Context performance muammolari

Context qiymati o'zgarganda, uni `useContext` orqali iste'mol qiladigan **barcha** komponentlar re-render bo'ladi — qiymatning faqat ishlatilmaydigan qismi o'zgargan bo'lsa ham.

**Muammo 1: value object har render'da yangi reference**

```tsx
// ❌ har render'da yangi { user, setUser } → barcha iste'molchi render bo'ladi
<UserContext.Provider value={{ user, setUser }}>

// ✅ memoize
const value = useMemo(() => ({ user, setUser }), [user]);
<UserContext.Provider value={value}>
```

**Muammo 2: bitta katta context** — turli o'zgaruvchanlikdagi qiymatlar bir contextda. Yechim — **context'larni ajratish**:

```tsx
// state va dispatch'ni alohida context'ga ajrating
const StateContext = createContext<State | null>(null);
const DispatchContext = createContext<Dispatch | null>(null);
// dispatch hech qachon o'zgarmaydi → dispatch iste'molchilari render bo'lmaydi
```

**Yechimlar umumiy:**
- `value`'ni `useMemo` bilan barqarorlashtiring.
- Context'ni mantiqiy bo'lib tashlang (state vs dispatch, tez vs sekin o'zgaruvchi).
- Tez-tez o'zgaradigan global state uchun **Zustand/Jotai/Redux** — ular selector orqali faqat kerakli qismga obuna bo'lishni beradi (context bunday selektiv obunani bermaydi).

**⚠️ Ehtiyot bo'l:** Context — dependency injection vositasi, yuqori chastotali state manager emas. Har millisekundda o'zgaradigan qiymatni (masalan, mouse koordinatasi) context'ga qo'ymang.

---

## useTransition

`useTransition` (React 18) yangilanishni **non-urgent** (shoshilinch emas) deb belgilaydi — React uni interrupt qila oladi, shunda urgent yangilanishlar (input yozish) bloklanmaydi.

```tsx
function SearchPage() {
  const [query, setQuery] = useState("");
  const [results, setResults] = useState<Item[]>([]);
  const [isPending, startTransition] = useTransition();

  function onChange(e: React.ChangeEvent<HTMLInputElement>) {
    setQuery(e.target.value); // urgent — input darhol yangilanadi
    startTransition(() => {
      // non-urgent — qimmat ro'yxat keyin yangilanadi, input'ni bloklamaydi
      setResults(expensiveFilter(e.target.value));
    });
  }

  return (
    <>
      <input value={query} onChange={onChange} />
      {isPending && <Spinner />}
      <Results items={results} />
    </>
  );
}
```

**💡 Tushuncha:** `useTransition` qimmat hisobni *tezlashtirmaydi* — u faqat uni shoshilinch emas deb belgilab, UI'ni javobgar (responsive) qoldiradi. Bu INP (interaktivlik) uchun foydali. `useDeferredValue` — shunga o'xshash, lekin qiymatni "kechiktiradi" (setState'ni o'rab bo'lmaganda qulay).

---

## Debounce va throttle

Tez-tez sodir bo'ladigan event'lar (input, scroll, resize) qimmat ishni chaqirsa, ularni cheklang.

- **Debounce** — oxirgi event'dan keyin N ms jim turilsa, bir marta ishga tushadi (qidiruv input uchun).
- **Throttle** — har N ms'da ko'pi bilan bir marta (scroll handler uchun).

```tsx
function useDebouncedValue<T>(value: T, delay: number): T {
  const [debounced, setDebounced] = useState(value);
  useEffect(() => {
    const id = setTimeout(() => setDebounced(value), delay);
    return () => clearTimeout(id); // har o'zgarishda eski timer bekor
  }, [value, delay]);
  return debounced;
}

function Search() {
  const [query, setQuery] = useState("");
  const debouncedQuery = useDebouncedValue(query, 300);
  useEffect(() => {
    fetchResults(debouncedQuery); // faqat 300ms jimlikdan keyin
  }, [debouncedQuery]);
  return <input value={query} onChange={(e) => setQuery(e.target.value)} />;
}
```

**⚠️ Ehtiyot bo'l:** Debounce'ni `useTransition` bilan adashtirmang. Debounce — *qancha marta* bajarishni kamaytiradi. Transition — *qachon va qaysi tartibda* bajarishni boshqaradi. Ko'pincha ikkalasi birga ishlatiladi.

---

## List virtualization

Minglab elementli ro'yxatda hammasini DOM'ga chizish — sekin (ko'p DOM tugun, ko'p render). **Virtualization** faqat ko'rinadigan (viewport'dagi) elementlarni render qiladi.

```tsx
import { FixedSizeList } from "react-window";

function BigList({ items }: { items: string[] }) {
  return (
    <FixedSizeList
      height={400}
      itemCount={items.length}
      itemSize={35}
      width="100%"
    >
      {({ index, style }) => <div style={style}>{items[index]}</div>}
    </FixedSizeList>
  );
}
```

**G'oya:** konteyner balandligini bilamiz, har elementning balandligini bilamiz → qaysi indekslar ko'rinishini hisoblaymiz va faqat o'shalarni render qilamiz. `style` (absolute position) bilan to'g'ri joyga qo'yamiz. Scroll bo'lganda ko'rinadigan oyna siljiydi.

**Qachon kerak:** 100+ element, ayniqsa har element murakkab bo'lsa. **Trade-off:** `Ctrl+F` brauzer qidiruvi ishlamaydi (DOM'da yo'q), accessibility murakkablashadi, dinamik balandlik qiyinroq (`VariableSizeList` kerak).

**💡 Tushuncha:** `react-window` yengilroq, `react-virtuoso` ko'proq feature (dinamik balandlik, grouping). TanStack Virtual — headless, ko'p moslashuvchan.

---

## Code splitting, React.lazy va Suspense

Butun ilovani bitta JS faylga to'plash — boshlang'ich yuklanish sekin. **Code splitting** kodni bo'laklarga (chunk) ajratadi, faqat kerakligini yuklaydi.

```tsx
import { lazy, Suspense } from "react";

const Dashboard = lazy(() => import("./Dashboard")); // alohida chunk

function App() {
  return (
    <Suspense fallback={<Spinner />}>
      <Dashboard /> {/* faqat ko'rsatilganda yuklanadi */}
    </Suspense>
  );
}
```

Eng samarali bo'linish nuqtalari:
- **Route darajasi** — har sahifa alohida chunk (eng katta foyda).
- **Og'ir modal/widget** — kerak bo'lguncha yuklanmaydigan og'ir komponentlar (chart kutubxonasi, rich text editor).

```tsx
const routes = [
  { path: "/", element: lazy(() => import("./Home")) },
  { path: "/settings", element: lazy(() => import("./Settings")) },
];
```

**⚠️ Ehtiyot bo'l:** Juda mayda bo'lish ham yomon — har chunk uchun alohida network so'rov bor. Mantiqiy chegaralarda (route, og'ir feature) bo'ling. Suspense fallback'ni layout siljimaydigan qilib bering (CLS uchun).

---

## Bundle size va tree shaking

**Tree shaking** — ishlatilmaydigan eksportlarni bundle'dan olib tashlash. ES modules (`import`/`export`) statik tahlil qilinadi, shuning uchun ishlaydi; CommonJS (`require`) — yo'q.

```tsx
// ✅ named import — faqat debounce bundle'ga kiradi (tree-shakeable kutubxonada)
import { debounce } from "lodash-es";

// ❌ butun lodash kiradi (CommonJS, tree-shake bo'lmaydi)
import _ from "lodash";
_.debounce(/* ... */);
```

Strategiyalar:
- **Bundle'ni tahlil qiling** — `vite-plugin-visualizer` yoki `webpack-bundle-analyzer` — nima og'irligini ko'ring.
- **Yengil alternativalar** — `date-fns` (tree-shakeable) `moment` o'rniga; `lodash-es` named import.
- **`sideEffects: false`** — `package.json`'da kutubxona toza ekanini bildiradi, agressivroq tree shaking.
- **Polyfill'larni cheklang** — kerak bo'lmagan brauzerlarni `browserslist`'dan chiqaring.

**💡 Tushuncha:** Bundle size to'g'ridan-to'g'ri LCP/FCP'ga ta'sir qiladi — kam JS = tez parse/execute. "Eng tez kod — yozilmagan kod."

---

## Lazy loading

- **Image lazy loading:** `<img loading="lazy" />` — viewport'ga yaqinlashganda yuklanadi. `width`/`height` bering (CLS oldini olish).
- **Komponent lazy loading:** yuqoridagi `React.lazy`.
- **Prefetch/preload:** muhim resurs uchun `<link rel="preload">`; ehtimoliy keyingi sahifa uchun `rel="prefetch"`. Route komponentni hover'da prefetch qilish — UX yaxshilaydi.

```tsx
const Settings = lazy(() => import("./Settings"));
// hover'da prefetch
<Link onMouseEnter={() => import("./Settings")} to="/settings">
  Settings
</Link>;
```

---

## Web Vitals

Core Web Vitals — Google'ning real foydalanuvchi tajribasi metrikalari:

- **LCP (Largest Contentful Paint)** — eng katta ko'rinadigan element qachon chiziladi. Yaxshi: < 2.5s. React'da: bundle kichraytirish, SSR/RSC, image optimizatsiya, kritik CSS.
- **INP (Interaction to Next Paint)** — foydalanuvchi harakatidan keyingi paint'gacha kechikish (FID o'rnini bosgan). Yaxshi: < 200ms. React'da: main thread'ni bloklamaslik (`useTransition`, debounce, ish bo'laklash, virtualizatsiya).
- **CLS (Cumulative Layout Shift)** — kutilmagan layout siljishi. Yaxshi: < 0.1. React'da: image/video o'lchamlarini berish, fallback'lar joy egallashi, font swap'ni boshqarish.

**💡 Tushuncha:** LCP — yuklanish tezligi, INP — javobgarlik, CLS — vizual barqarorlik. React'da INP ko'pincha eng ko'p e'tibor talab qiladi, chunki ortiqcha render va og'ir handlerlar main thread'ni bloklaydi. `web-vitals` kutubxonasi bilan real foydalanuvchidan o'lchang (RUM).

---

## Server Components (qisqacha)

**React Server Components (RSC)** — server'da render bo'ladigan, brauzerga JS yubormaydigan komponentlar (Next.js App Router'da default).

- **Foyda:** bundle kichik (komponent kodi client'ga bormaydi), ma'lumotlarga to'g'ridan-to'g'ri (DB, fayl) kirish, kam client JS → tez LCP/INP.
- **Cheklov:** state yo'q, `useState`/`useEffect`/event handler yo'q, browser API yo'q. Interaktivlik kerak bo'lsa `"use client"` bilan Client Component'ga o'tasiz.
- **Pattern:** Server Component ma'lumot oladi, kichik interaktiv qismni Client Component'ga uzatadi ("islands"). Bu client bundle'ni minimal qiladi.

```tsx
// Server Component (default) — JS client'ga bormaydi
async function ProductPage({ id }: { id: string }) {
  const product = await db.product.find(id); // server'da
  return <ProductView product={product} />;
}
```

**💡 Tushuncha:** RSC performance'ning kelajagi — "kamroq client JS" g'oyasini arxitektura darajasiga ko'taradi. Lekin u SSR'ni almashtirmaydi, balki to'ldiradi; va u faqat mos framework (hozircha Next.js) ichida ishlaydi.

---

## Savol-javoblar

### ❓ Re-render aslida nimani anglatadi va u har doim sekinmi?

**✅ Javob:** Re-render — komponent funksiyasining qayta chaqirilishi va yangi virtual DOM hosil qilishi. Bu *real DOM yangilanishi emas* — reconciliation farq topmasa, DOM o'zgarmaydi. Re-render odatda arzon; muammo *ortiqcha* (kerak bo'lmagan) yoki *qimmat* (og'ir hisob ichidagi) render. Shuning uchun "re-render'ni har doim oldini olish kerak" — noto'g'ri intuitsiya; avval o'lchang.

### ❓ React.memo qachon foyda keltiradi va qachon faqat zarar?

**✅ Javob:** Foyda — komponent tez-tez render bo'ladigan parent ostida, props barqaror (memoize qilingan yoki primitiv), va render qimmat bo'lsa. Zarar/befoyda — props har render'da o'zgaradigan bo'lsa (memo solishtiradi, baribir render qiladi → faqat solishtirish narxi qo'shiladi), yoki komponent baribir kam render bo'lsa. Hamma joyni memo qilish anti-pattern: solishtirish va xotira narxi bor, kod murakkablashadi.

### ❓ useMemo va useCallback orasidagi farq nima?

**✅ Javob:** `useMemo(fn, deps)` — `fn()` *natijasini* (qiymatni) cache qiladi. `useCallback(fn, deps)` — `fn` *funksiyasining o'zini* cache qiladi. `useCallback(fn, deps)` aynan `useMemo(() => fn, deps)`ga teng. useMemo qimmat hisob yoki referential equality kerak bo'lgan qiymat uchun; useCallback memo child'ga yoki dependency'ga uzatiladigan funksiya uchun.

### ❓ Hamma narsani useMemo/useCallback bilan o'rab qo'yish nima uchun yomon?

**✅ Javob:** Har biri o'z dependency array'ini saqlaydi va har render'da solishtiradi — bu xotira va CPU narxi. Agar natija memo child'ga yoki dependency'ga uzatilmasa va hisob qimmat bo'lmasa — memo mutlaqo befoyda, faqat narx va o'qilishi qiyin kod qo'shadi. Qoida: aniq sabab (qimmat hisob YOKI referential equality) bo'lmasa, qo'shmang.

### ❓ Inline object/function qachon muammo, qachon emas?

**✅ Javob:** Muammo — agar u `React.memo` qilingan child'ga uzatilsa: har render'da yangi reference memo'ni "buzadi", child ortiqcha render bo'ladi. Muammo emas — agar child memo emas (u baribir render bo'lardi) yoki oddiy DOM elementga (`<button onClick={...}>`) uzatilsa. Ya'ni inline'dan qochish faqat memoization zanjiri kontekstida ma'noli.

### ❓ Ro'yxatda index'ni key qilib ishlatish nega xavfli?

**✅ Javob:** Ro'yxat tartibi o'zgarsa (qo'shish, o'chirish, sortlash) indekslar siljiydi. React `key={2}`ni eski `key={2}` bilan moslaydi, lekin bu endi boshqa element → noto'g'ri DOM/state qayta ishlatiladi (input qiymati aralashadi), ortiqcha mutatsiya. Statik, hech qachon tartibi o'zgarmaydigan ro'yxatda index xavfsiz, aks holda barqaror noyob id ishlating.

### ❓ Context qiymati o'zgarganda kim re-render bo'ladi va buni qanday cheklash mumkin?

**✅ Javob:** `useContext` orqali shu context'ni iste'mol qilgan *barcha* komponentlar — qiymatning faqat ishlatilmaydigan qismi o'zgargan bo'lsa ham. Cheklash: (1) `value` object'ni `useMemo` bilan barqarorlashtirish; (2) context'ni mantiqan ajratish (state vs dispatch); (3) tez o'zgaradigan global state uchun selektiv obunali kutubxona (Zustand/Jotai/Redux) ishlatish — ular faqat kerakli qismga obuna bo'lishni beradi.

### ❓ State colocation nima va u nega ko'pincha memo'dan yaxshiroq?

**✅ Javob:** State'ni undan foydalanadigan komponentga iloji boricha yaqin (past) joylashtirish. Yuqoridagi state butun pastdagi daraxtni render qiladi; pastga ko'chirsangiz, render doirasi qisqaradi. Memo'dan yaxshiroq, chunki muammoni *manbada* hal qiladi — qo'shimcha solishtirish/xotira narxi yo'q, kod sodda qoladi. Ko'chira olmasangiz, qimmat qismni `children` orqali uzatish ("lifting content down") shunga o'xshash foyda beradi.

### ❓ useTransition qimmat hisobni tezlashtiradimi?

**✅ Javob:** Yo'q. U yangilanishni non-urgent deb belgilaydi, shunda React urgent yangilanishlarni (input yozish) bloklamasdan, qimmat ishni interrupt qila oladi. Hisobning o'zi sekinligicha qoladi, lekin UI javobgar bo'lib qoladi (INP yaxshilanadi). Hisobni tezlashtirish uchun memoizatsiya, virtualizatsiya yoki algoritmni o'zgartirish kerak.

### ❓ Code splitting'da juda mayda bo'lish nega yomon?

**✅ Javob:** Har chunk alohida network so'rov + HTTP overhead + waterfall xavfi. Juda ko'p mayda chunk boshlang'ich yuklanishni sekinlashtirishi mumkin (parallel so'rovlar cheklangan, har biriga round-trip). Mantiqiy chegaralarda bo'ling: route darajasi (eng samarali), og'ir mustaqil feature (chart, editor). O'lchang — bundle analyzer bilan real foydani ko'ring.

### ❓ Tree shaking nima va nima uchun ba'zan ishlamaydi?

**✅ Javob:** Ishlatilmaydigan eksportlarni bundle'dan chiqarib tashlash. U ES modules'ning statik tuzilishiga tayanadi (`import`/`export` kompilyatsiya vaqtida tahlil qilinadi). Ishlamaydigan holatlar: CommonJS (`require` dinamik), default import butun kutubxonani olib kirsa (`import _ from "lodash"`), yoki kutubxona `sideEffects` bilan noto'g'ri belgilangan bo'lsa. Yechim: named import + tree-shakeable kutubxonalar (`lodash-es`, `date-fns`).

### ❓ INP nima va React'da uni nimalar yomonlashtiradi?

**✅ Javob:** Interaction to Next Paint — foydalanuvchi harakatidan (bosish, yozish) keyingi paint'gacha bo'lgan kechikish; javobgarlik metrikasi (yaxshi < 200ms). React'da yomonlashtiradi: og'ir event handler, ortiqcha/qimmat re-render, katta virtuallashtirilmagan ro'yxat, main thread'ni bloklaydigan sinxron hisob. Yaxshilash: `useTransition`/`useDeferredValue`, debounce, virtualizatsiya, ishni bo'laklash, ortiqcha render'ni kamaytirish.

### ❓ Server Components performance'ga qanday yordam beradi?

**✅ Javob:** Ular server'da render bo'ladi va brauzerga JS yubormaydi — client bundle kichrayadi (komponent kodi va ko'pincha og'ir kutubxonalar client'ga bormaydi), bu LCP/INP'ni yaxshilaydi. Ma'lumotga to'g'ridan-to'g'ri (DB) kirish data waterfall'ni kamaytiradi. Interaktiv qismlar `"use client"` "orollari" sifatida qoladi. Lekin RSC SSR'ni almashtirmaydi va faqat mos framework ichida ishlaydi.

### ❓ Profiler bilan ortiqcha render manbasini qanday topasiz?

**✅ Javob:** React DevTools → Profiler → record → interfeys bilan ishlash → stop. Flame/ranked grafikda qaysi komponent qancha render bo'lganini ko'rasiz. Sozlamalarda "Record why each component rendered" yoqilsa, har render sababini (props/state/hooks/parent) ko'rsatadi. "Highlight updates" rejimi render'larni ekranda miltillatadi. Production yoki CPU throttling bilan o'lchang — development build realistik emas.

---

## Masalalar

> Yechimlar: [solutions/frontend/07-react-performance.md](../solutions/frontend/07-react-performance.md)

1. **Ortiqcha render'ni topish.** Bir input va og'ir `Dashboard` bitta `App`da. Har harf yozilganda `Dashboard` render bo'lyapti. State colocation va `children` pattern bilan ikki xil yechim yozing, qaysi biri qachon afzalligini izohlang.

2. **memo'ni buzgan props.** `const Row = React.memo(...)` ro'yxat elementi har scroll'da render bo'lyapti. Parent quyidagini uzatadi: `<Row data={item} style={{padding: 8}} onClick={() => select(item.id)} />`. Memo nima uchun ishlamayotganini aniqlang va to'g'rilang.

3. **Debounce qidiruv hook.** `useDebouncedValue<T>(value, delay)` hook'ini noldan yozing (cleanup bilan), va undan foydalanib faqat 300ms jimlikdan keyin API chaqiradigan qidiruvni quring.

4. **Context'ni ajratish.** Bir `AppContext` ichida `{ theme, user, notifications }` bor; `notifications` tez-tez o'zgaradi va butun ilovani render qiladi. Context'larni qayta loyihalashtiring, `value`'larni barqarorlashtiring, qaysi iste'molchi qachon render bo'lishini tushuntiring.

5. **useTransition bilan qidiruv.** 10 000 elementli ro'yxatdan filtrlash inputni bloklayapti. `useTransition` (yoki `useDeferredValue`) bilan inputni javobgar qiling, `isPending` indikatorini qo'shing. Nima uchun bu hisobni tezlashtirmasligini izohlang.

6. **Route-based code splitting.** Uch route (`/`, `/dashboard`, `/settings`)ni `React.lazy` + `Suspense` bilan bo'ling, og'ir `/dashboard`ni hover'da prefetch qiling. Juda mayda bo'lishning trade-off'ini bir paragrafda yozing.

7. **List virtualization.** 5 000 qatorli jadvalni `react-window` (yoki shunchaki g'oyani qo'lda) bilan virtuallang. Qaysi UX trade-off'lar paydo bo'lishini (Ctrl+F, dinamik balandlik, a11y) sanab, har biriga yumshatish (mitigation) taklif qiling.

8. **Memoizatsiya auditi.** Berilgan komponentda 6 ta `useMemo`/`useCallback` bor. Har birini ko'rib chiqing va "kerak / befoyda / zararli" deb tasniflang, sababini referential equality va child memo holatiga asoslab yozing.

---

← [Frontend bo'limiga qaytish](./README.md)
