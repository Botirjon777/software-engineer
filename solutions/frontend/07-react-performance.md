# React Performance — Yechimlar

Bu yerda [07-react-performance.md](../../frontend/07-react-performance.md) "Masalalar" bo'limining yechimlari. Har yechim — bitta to'g'ri yondashuv; muhimi *sabab* va *trade-off*ni tushunish.

---

## 1. Ortiqcha render'ni topish

**Yechim A — state colocation:** input state'ni alohida komponentga ko'chiramiz, shunda `Dashboard` uning render doirasidan chiqadi.

```tsx
function SearchBox() {
  const [query, setQuery] = useState("");
  return <input value={query} onChange={(e) => setQuery(e.target.value)} />;
}
function App() {
  return (
    <>
      <SearchBox />
      <Dashboard /> {/* SearchBox render bo'lsa ham, App render bo'lmaydi */}
    </>
  );
}
```

**Yechim B — children sifatida uzatish (lifting content down):** state'ni ko'chira olmasak (masalan, `query` parent'da kerak), `Dashboard`ni `children` qilamiz.

```tsx
function SearchLayout({ children }: { children: React.ReactNode }) {
  const [query, setQuery] = useState("");
  return (
    <>
      <input value={query} onChange={(e) => setQuery(e.target.value)} />
      {children} {/* App'da yaratilgan, query'ga bog'liq emas → render bo'lmaydi */}
    </>
  );
}
function App() {
  return (
    <SearchLayout>
      <Dashboard />
    </SearchLayout>
  );
}
```

**Izoh:** A — eng toza, agar `query` boshqa joyda kerak bo'lmasa. B — `query`ni layout'da ushlab turish kerak bo'lganda, lekin qimmat farzandni render qilmaslik kerak bo'lganda. Ikkalasi ham `React.memo`'dan yaxshiroq: muammoni manbada (render doirasini qisqartirib) hal qiladi, qo'shimcha solishtirish narxi yo'q.

---

## 2. memo'ni buzgan props

**Muammo:** `style={{padding: 8}}` va `onClick={() => select(item.id)}` har render'da yangi reference yaratadi. `React.memo` shallow solishtirishda bu propslarni "o'zgargan" deb topadi → har scroll'da (parent render'da) `Row` render bo'ladi.

**Yechim:**

```tsx
const ROW_STYLE = { padding: 8 }; // modul darajasi — bir marta yaratiladi

const Row = React.memo(function Row({
  data,
  onClick,
}: {
  data: Item;
  onClick: (id: string) => void;
  style: React.CSSProperties;
}) {
  return <div style={ROW_STYLE} onClick={() => onClick(data.id)}>{data.name}</div>;
});

function List({ items }: { items: Item[] }) {
  // bitta barqaror funksiya — id'ni argument sifatida qabul qiladi
  const handleClick = useCallback((id: string) => select(id), []);
  return (
    <>
      {items.map((item) => (
        <Row key={item.id} data={item} onClick={handleClick} style={ROW_STYLE} />
      ))}
    </>
  );
}
```

**Izoh:** Asosiy texnika — funksiyani `useCallback` bilan barqaror qilib, `item.id`ni inline closure'da emas, callback argumenti orqali uzatish. Statik object'ni modul darajasiga chiqarish — eng arzon yechim (`useMemo` ham shart emas). Endi `data` (item) o'zgarmaган qatorlar render bo'lmaydi.

---

## 3. Debounce qidiruv hook

```tsx
function useDebouncedValue<T>(value: T, delay: number): T {
  const [debounced, setDebounced] = useState(value);

  useEffect(() => {
    const id = setTimeout(() => setDebounced(value), delay);
    return () => clearTimeout(id); // value o'zgarsa, eski timer bekor bo'ladi
  }, [value, delay]);

  return debounced;
}

function Search() {
  const [query, setQuery] = useState("");
  const debouncedQuery = useDebouncedValue(query, 300);

  useEffect(() => {
    if (!debouncedQuery) return;
    const controller = new AbortController();
    fetchResults(debouncedQuery, controller.signal).catch(() => {});
    return () => controller.abort(); // eskirgan so'rovni bekor qilish
  }, [debouncedQuery]);

  return <input value={query} onChange={(e) => setQuery(e.target.value)} />;
}
```

**Izoh:** Cleanup (`clearTimeout`) — debounce'ning yuragi: har klaviatura bosishida eski timer bekor bo'lib, faqat 300ms jimlikdan keyin oxirgisi ishlaydi. `AbortController` qo'shilishi — race condition oldini oladi (tez yozilganda eskirgan javob yangisini bosib ketmasligi uchun).

---

## 4. Context'ni ajratish

**Muammo:** `{ theme, user, notifications }` bitta context'da; `notifications` tez o'zgaradi → har o'zgarishda butun ilova render bo'ladi.

**Yechim — har mas'uliyatni alohida context, har `value`ni memoize:**

```tsx
const ThemeContext = createContext<Theme | null>(null);
const UserContext = createContext<User | null>(null);
const NotificationsContext = createContext<Notification[] | null>(null);

function Providers({ children }: { children: React.ReactNode }) {
  const [theme, setTheme] = useState<Theme>("light");
  const [user, setUser] = useState<User | null>(null);
  const [notifications, setNotifications] = useState<Notification[]>([]);

  // har value alohida memoize — faqat o'z dependency'si o'zgarganda yangilanadi
  const themeValue = useMemo(() => ({ theme, setTheme }), [theme]);
  const userValue = useMemo(() => ({ user, setUser }), [user]);

  return (
    <ThemeContext.Provider value={themeValue}>
      <UserContext.Provider value={userValue}>
        <NotificationsContext.Provider value={notifications}>
          {children}
        </NotificationsContext.Provider>
      </UserContext.Provider>
    </ThemeContext.Provider>
  );
}
```

**Izoh:** Endi `notifications` o'zgarganda faqat `useContext(NotificationsContext)` ishlatadigan komponentlar render bo'ladi. `useContext(ThemeContext)` ishlatadiganlar tegilmaydi. `value`larni `useMemo` qilish referential barqarorlikni ta'minlaydi (aks holda Providers render'ida yangi object → ortiqcha render). Agar `notifications` juda tez (har soniyada ko'p marta) o'zgarsa, uni Zustand/Jotai'ga ko'chirish va selector bilan obuna bo'lish yanada yaxshi.

---

## 5. useTransition bilan qidiruv

```tsx
function Search({ allItems }: { allItems: Item[] }) {
  const [query, setQuery] = useState("");
  const [results, setResults] = useState<Item[]>(allItems);
  const [isPending, startTransition] = useTransition();

  function onChange(e: React.ChangeEvent<HTMLInputElement>) {
    const next = e.target.value;
    setQuery(next); // urgent: input darhol yangilanadi, hech qachon "lag" bermaydi
    startTransition(() => {
      // non-urgent: 10k element filtri input'ni bloklamaydi
      setResults(allItems.filter((i) => i.name.includes(next)));
    });
  }

  return (
    <>
      <input value={query} onChange={onChange} />
      {isPending && <span>Filtrlanmoqda...</span>}
      <ul style={{ opacity: isPending ? 0.6 : 1 }}>
        {results.map((i) => <li key={i.id}>{i.name}</li>)}
      </ul>
    </>
  );
}
```

**Izoh:** `setQuery` urgent qoladi (input javobgar), `setResults` transition ichida — React uni interrupt qila oladi, shuning uchun yozish bloklanmaydi. **Nima uchun bu hisobni tezlashtirmaydi:** filtr baribir 10k elementni aylanib chiqadi, o'sha vaqtni oladi. `useTransition` faqat *qachon* va *qaysi ustuvorlikda* bajarilishini boshqaradi — UI'ni javobgar qoldiradi, lekin ish hajmini kamaytirmaydi. Haqiqiy tezlashtirish uchun memoizatsiya, virtualizatsiya yoki indeksli qidiruv kerak. (`useDeferredValue` ham muqobil: `const deferredQuery = useDeferredValue(query)` va filtrni `deferredQuery` bo'yicha `useMemo` qilish.)

---

## 6. Route-based code splitting

```tsx
import { lazy, Suspense } from "react";
import { Routes, Route, Link } from "react-router-dom";

const Home = lazy(() => import("./Home"));
const Dashboard = lazy(() => import("./Dashboard"));
const Settings = lazy(() => import("./Settings"));

function App() {
  return (
    <>
      <nav>
        <Link to="/">Home</Link>
        {/* hover'da prefetch: chunk oldindan yuklanadi, bosilganda darhol ochiladi */}
        <Link to="/dashboard" onMouseEnter={() => import("./Dashboard")}>
          Dashboard
        </Link>
        <Link to="/settings">Settings</Link>
      </nav>
      <Suspense fallback={<PageSkeleton />}>
        <Routes>
          <Route path="/" element={<Home />} />
          <Route path="/dashboard" element={<Dashboard />} />
          <Route path="/settings" element={<Settings />} />
        </Routes>
      </Suspense>
    </>
  );
}
```

**Juda mayda bo'lishning trade-off'i:** Har chunk uchun alohida HTTP so'rov, header overhead va round-trip kechikish bor. Agar kodni juda ko'p mayda chunkga bo'lsangiz, boshlang'ich yuklanish *sekinlashishi* mumkin — brauzer parallel so'rovlarni cheklaydi va waterfall paydo bo'ladi. Mantiqiy chegaralarda (route, og'ir mustaqil feature) bo'ling, har kichik komponentni emas. `PageSkeleton` fallback'ini layout siljimaydigan o'lchamda bering (CLS oldini olish).

---

## 7. List virtualization

```tsx
import { FixedSizeList } from "react-window";

function VirtualTable({ rows }: { rows: Row[] }) {
  return (
    <FixedSizeList height={500} width="100%" itemCount={rows.length} itemSize={40}>
      {({ index, style }) => (
        <div style={style} role="row">
          <span role="cell">{rows[index].name}</span>
          <span role="cell">{rows[index].email}</span>
        </div>
      )}
    </FixedSizeList>
  );
}
```

**UX trade-off'lar va yumshatish:**

| Trade-off | Sabab | Yumshatish |
|-----------|-------|-----------|
| `Ctrl+F` ishlamaydi | ko'rinmagan qatorlar DOM'da yo'q | ilova ichida o'z qidiruv/filtr berish |
| Dinamik balandlik qiyin | `FixedSizeList` qat'iy balandlik kutadi | `VariableSizeList` yoki `react-virtuoso` (auto-measure) |
| Accessibility | virtual DOM screen reader'ni chalkashtiradi | `role="row/cell"`, `aria-rowcount`/`aria-rowindex` to'g'ri berish |
| Scroll pozitsiyasi yo'qolishi | qayta render'da reset | kalit (key) barqarorligi, `react-virtuoso` saqlash imkoni |

**Izoh:** Virtualizatsiya 100+ (ayniqsa murakkab) element uchun katta foyda, lekin DOM to'liqligini "qurbon qiladi". Agar ro'yxat kichik bo'lsa, virtualizatsiya keraksiz murakkablik.

---

## 8. Memoizatsiya auditi

Berilgan har `useMemo`/`useCallback`ni shu mezon bilan tasniflang:

```tsx
function Component({ items, userId }: Props) {
  // 1. ❌ ZARARLI/BEFOYDA: oddiy hisob, hech qayerga uzatilmaydi
  const count = useMemo(() => items.length, [items]);
  // → const count = items.length;

  // 2. ✅ KERAK: qimmat filtr, natija memo child'ga uzatiladi
  const filtered = useMemo(() => items.filter((i) => i.active), [items]);

  // 3. ❌ BEFOYDA: callback oddiy DOM elementga uzatiladi (memo child emas)
  const onPlain = useCallback(() => console.log("x"), []);
  // → inline qoldirish mumkin: <button onClick={() => ...}>

  // 4. ✅ KERAK: callback React.memo child'ga uzatiladi
  const onSelect = useCallback((id: string) => select(id), []);

  // 5. ⚠️ ZARARLI: dependency har render'da o'zgaradi → memo hech qachon ishlamaydi
  const config = useMemo(() => ({ userId }), [{ userId }]);
  // → noto'g'ri deps; to'g'risi [userId]

  // 6. ✅ KERAK: natija useEffect dependency'si, referential barqarorlik kerak
  const query = useMemo(() => ({ userId, page: 1 }), [userId]);

  return (
    <>
      <span>{count}</span>
      <MemoList items={filtered} onSelect={onSelect} />
      <button onClick={onPlain}>plain</button>
      <Fetcher query={query} />
    </>
  );
}
```

**Tasniflash qoidasi:**
- **Kerak** — (a) hisob *haqiqatan* qimmat, YOKI (b) natija `React.memo` child'ga / hook dependency'siga uzatiladi (referential equality kerak).
- **Befoyda** — oddiy hisob va memo child / dependency yo'q. Faqat narx qo'shadi, olib tashlang.
- **Zararli** — yuqoridagi + noto'g'ri deps (har render'da yangi reference) memo'ni har doim "miss" qildiradi yoki bug keltiradi.

---

← [Frontend bo'limiga qaytish](../../frontend/README.md)
