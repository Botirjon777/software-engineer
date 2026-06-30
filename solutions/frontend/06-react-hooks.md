# React Hooks — Yechimlar

Bu fayl [06-react-hooks.md](../../frontend/06-react-hooks.md) dagi "Masalalar" bo'limining yechimlari. Har bir yechim kod va o'zbekcha izoh bilan.

---

## 1. useToggle

```tsx
import { useCallback, useState } from "react";

function useToggle(initial = false) {
  const [value, setValue] = useState(initial);

  const toggle = useCallback(() => setValue((v) => !v), []);
  const setTrue = useCallback(() => setValue(true), []);
  const setFalse = useCallback(() => setValue(false), []);

  return [value, toggle, setTrue, setFalse] as const;
}

// Foydalanish:
function Box() {
  const [open, toggle, , close] = useToggle();
  return (
    <>
      <button onClick={toggle}>{open ? "Yopish" : "Ochish"}</button>
      {open && <div onClick={close}>Kontent</div>}
    </>
  );
}
```

**Izoh:** `toggle` functional update (`(v) => !v`) ishlatadi — bu eski qiymatga emas, eng so'nggi state'ga tayanadi, shuning uchun stale closure muammosi yo'q. `useCallback` funksiyalar reference'ini barqaror qiladi (memoized child'larga uzatilsa foydali). `as const` qaytarish tuple turini saqlaydi.

---

## 2. useDebounce qidiruv

```tsx
import { useEffect, useState } from "react";

function useDebounce<T>(value: T, delay = 400): T {
  const [debounced, setDebounced] = useState(value);
  useEffect(() => {
    const id = setTimeout(() => setDebounced(value), delay);
    return () => clearTimeout(id);
  }, [value, delay]);
  return debounced;
}

function Search() {
  const [query, setQuery] = useState("");
  const debouncedQuery = useDebounce(query, 400);

  useEffect(() => {
    if (debouncedQuery) {
      console.log("Qidirilmoqda:", debouncedQuery);
      // fetch(`/api/search?q=${debouncedQuery}`)
    }
  }, [debouncedQuery]);

  return <input value={query} onChange={(e) => setQuery(e.target.value)} />;
}
```

**Izoh:** Har bosishda `query` o'zgaradi, bu `useDebounce` effect'ini qayta ishga tushiradi va eski `setTimeout` cleanup orqali bekor qilinadi. Foydalanuvchi 400ms to'xtaganda timer ishlab, `debouncedQuery` yangilanadi. Faqat shu yangilangan qiymatga `useEffect` reaksiya bildiradi, shuning uchun fetch har bosishda emas, faqat to'xtaganda chaqiriladi.

---

## 3. Interval bug'ni tuzating (stale closure)

Muammoli kod:

```tsx
// ❌ count doim 0 closure'da qotib qolgan → har safar 0 + 1 = 1
useEffect(() => {
  const id = setInterval(() => setCount(count + 1), 1000);
  return () => clearInterval(id);
}, []);
```

Tuzatilgan:

```tsx
// ✅ functional update eng so'nggi qiymatga tayanadi
useEffect(() => {
  const id = setInterval(() => setCount((c) => c + 1), 1000);
  return () => clearInterval(id);
}, []);
```

**Izoh:** Birinchi variantda effect bo'sh dependency (`[]`) bilan faqat bir marta ishlaydi va `setInterval` ichidagi callback **birinchi render'dagi** `count = 0`'ni closure orqali ushlab qoladi. Shuning uchun har sekundda `0 + 1`. Functional update (`(c) => c + 1`) closure'dagi `count`'ga emas, React beradigan eng so'nggi qiymatga tayanadi, shuning uchun hisob to'g'ri ortadi va `count`'ni dependency'ga qo'shish (intervalni qayta yaratish) shart emas.

---

## 4. useLocalStorage (dark mode)

```tsx
import { useEffect, useState } from "react";

function useLocalStorage<T>(key: string, initial: T) {
  const [value, setValue] = useState<T>(() => {
    const raw = localStorage.getItem(key);
    return raw ? (JSON.parse(raw) as T) : initial; // lazy init
  });

  useEffect(() => {
    localStorage.setItem(key, JSON.stringify(value));
  }, [key, value]);

  return [value, setValue] as const;
}

function DarkModeToggle() {
  const [dark, setDark] = useLocalStorage("dark-mode", false);
  return (
    <button onClick={() => setDark((d) => !d)}>
      {dark ? "Yorug' rejim" : "Tungi rejim"}
    </button>
  );
}
```

**Izoh:** Lazy init (`useState(() => ...)`) `localStorage`'dan boshlang'ich qiymatni faqat birinchi render'da o'qiydi (har render'da emas). `value` o'zgarganda effect uni `localStorage`'ga yozadi. Reload bo'lganda lazy init saqlangan qiymatni qaytaradi, shuning uchun sozlama saqlanib qoladi.

---

## 5. useReducer todo

```tsx
import { useReducer } from "react";

type Todo = { id: number; text: string; done: boolean };
type Action =
  | { type: "add"; text: string }
  | { type: "toggle"; id: number }
  | { type: "remove"; id: number };

function todoReducer(state: Todo[], action: Action): Todo[] {
  switch (action.type) {
    case "add":
      return [...state, { id: Date.now(), text: action.text, done: false }];
    case "toggle":
      return state.map((t) =>
        t.id === action.id ? { ...t, done: !t.done } : t
      );
    case "remove":
      return state.filter((t) => t.id !== action.id);
    default:
      return state;
  }
}

function TodoApp() {
  const [todos, dispatch] = useReducer(todoReducer, []);
  return (
    <ul>
      {todos.map((t) => (
        <li key={t.id}>
          <span onClick={() => dispatch({ type: "toggle", id: t.id })}>
            {t.done ? "✓ " : ""}{t.text}
          </span>
          <button onClick={() => dispatch({ type: "remove", id: t.id })}>×</button>
        </li>
      ))}
      <button onClick={() => dispatch({ type: "add", text: "Yangi" })}>
        Qo'shish
      </button>
    </ul>
  );
}
```

**Izoh:** Barcha state o'zgarishlari `todoReducer` sof funksiyasida jamlangan — har action yangi massiv qaytaradi (immutable). Bu murakkab, bog'liq o'zgarishlar uchun `useState`'dan toza: mantiq bir joyda, test qilish oson, component'da faqat `dispatch` qoladi.

---

## 6. memo + useCallback

Muammo:

```tsx
const Child = React.memo(({ onClick }: { onClick: () => void }) => {
  console.log("Child render");
  return <button onClick={onClick}>Bosing</button>;
});

function Parent() {
  const [count, setCount] = useState(0);
  // ❌ har render'da yangi funksiya reference → memo ishlamaydi
  const handleClick = () => console.log("clicked");
  return (
    <>
      <button onClick={() => setCount((c) => c + 1)}>{count}</button>
      <Child onClick={handleClick} />
    </>
  );
}
```

Tuzatilgan:

```tsx
function Parent() {
  const [count, setCount] = useState(0);
  // ✅ reference barqaror → memo ishlaydi, Child re-render bo'lmaydi
  const handleClick = useCallback(() => console.log("clicked"), []);
  return (
    <>
      <button onClick={() => setCount((c) => c + 1)}>{count}</button>
      <Child onClick={handleClick} />
    </>
  );
}
```

**Izoh:** `Parent` re-render bo'lganda inline `handleClick` har safar **yangi reference** oladi. `React.memo` props'ni reference bo'yicha solishtiradi, shuning uchun yangi funksiya = "props o'zgardi" deb hisoblanadi va `Child` baribir re-render bo'ladi. `useCallback(fn, [])` reference'ni barqaror saqlaydi, shunda `memo` to'g'ri ishlaydi. `useCallback` faqat `memo`'langan child bilan birga ma'noli.

---

## 7. useContext theme

```tsx
import { createContext, useContext, useMemo, useState } from "react";

type ThemeCtx = { theme: "light" | "dark"; toggle: () => void };
const ThemeContext = createContext<ThemeCtx | null>(null);

function ThemeProvider({ children }: { children: React.ReactNode }) {
  const [theme, setTheme] = useState<"light" | "dark">("light");
  // value'ni memoize qilish → keraksiz consumer re-render'larini kamaytiradi
  const value = useMemo<ThemeCtx>(
    () => ({ theme, toggle: () => setTheme((t) => (t === "light" ? "dark" : "light")) }),
    [theme]
  );
  return <ThemeContext.Provider value={value}>{children}</ThemeContext.Provider>;
}

function useTheme() {
  const ctx = useContext(ThemeContext);
  if (!ctx) throw new Error("useTheme ThemeProvider ichida bo'lishi kerak");
  return ctx;
}

function DeepButton() {
  const { theme, toggle } = useTheme();
  return <button className={theme} onClick={toggle}>{theme}</button>;
}
```

**Izoh:** `value` `useMemo` bilan memoize qilingan, shuning uchun u faqat `theme` o'zgarganda yangi reference oladi — aks holda Provider har render'da yangi object berib, barcha consumer'larni keraksiz re-render qilardi. `useTheme` custom hook context'ni o'qiydi va Provider'siz ishlatilsa aniq xato beradi. `useContext` prop drilling'siz chuqurdagi `DeepButton`'ga ma'lumotni yetkazadi.

---

## 8. useFetch race condition

```tsx
import { useEffect, useState } from "react";

function useFetch<T>(url: string) {
  const [data, setData] = useState<T | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<Error | null>(null);

  useEffect(() => {
    let cancelled = false;
    const controller = new AbortController();

    setLoading(true);
    setError(null);

    fetch(url, { signal: controller.signal })
      .then((r) => {
        if (!r.ok) throw new Error(`HTTP ${r.status}`);
        return r.json();
      })
      .then((d: T) => { if (!cancelled) setData(d); })
      .catch((e: Error) => { if (!cancelled && e.name !== "AbortError") setError(e); })
      .finally(() => { if (!cancelled) setLoading(false); });

    return () => {
      cancelled = true;
      controller.abort();
    };
  }, [url]);

  return { data, loading, error };
}
```

**Izoh:** `url` o'zgarganda eski effect'ning cleanup'i ishlaydi: `cancelled = true` qilinadi va `controller.abort()` so'rovni bekor qiladi. Shuning uchun eski (sekin) so'rov kechroq tugasa ham, uning natijasi `cancelled` tekshiruvi tufayli state'ga yozilmaydi — yangi so'rovning natijasini bosib o'tmaydi. Bu race condition'ni hal qiladi. `AbortError`'ni alohida e'tiborsiz qoldiramiz, chunki u atayin bekor qilish (xato emas).

---

← [Frontend bo'limiga qaytish](../../frontend/README.md)
