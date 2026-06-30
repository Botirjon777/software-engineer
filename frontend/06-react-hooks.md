# React Hooks

> Yechimlar: [solutions/frontend/06-react-hooks.md](../solutions/frontend/06-react-hooks.md)

Bu fayl React hook'larini chuqur ko'rib chiqadi: har bir hook nima qiladi, qachon ishlatiladi, qanday bug'lar keltiradi va ularni qanday tuzatish kerak. Tushuntirishlar o'zbekcha, kod va atamalar inglizcha (TypeScript misollarda).

## Mundarija

- [useState (functional update, lazy init)](#usestate-functional-update-lazy-init)
- [useEffect (dependency, cleanup, qachon ishlaydi)](#useeffect-dependency-cleanup-qachon-ishlaydi)
- [useEffect bilan data fetching](#useeffect-bilan-data-fetching)
- [useLayoutEffect (vs useEffect)](#uselayouteffect-vs-useeffect)
- [useContext (Context API)](#usecontext-context-api)
- [useReducer (qachon)](#usereducer-qachon)
- [useRef (DOM va mutable qiymat)](#useref-dom-va-mutable-qiymat)
- [useMemo](#usememo)
- [useCallback](#usecallback)
- [useImperativeHandle](#useimperativehandle)
- [useId](#useid)
- [useTransition va useDeferredValue](#usetransition-va-usedeferredvalue)
- [Rules of Hooks](#rules-of-hooks)
- [Stale closure muammosi](#stale-closure-muammosi)
- [Custom hooks](#custom-hooks)
- [Dependency array tuzoqlari](#dependency-array-tuzoqlari)
- [Masalalar](#masalalar)

---

## useState (functional update, lazy init)

**💡 Tushuncha:** `useState` component'ga lokal, o'zgaruvchan state qo'shadi. U `[value, setValue]` juftligini qaytaradi. State o'zgarsa, component re-render bo'ladi.

```tsx
const [count, setCount] = useState(0);
```

### ❓ Functional update nima va qachon kerak?

**✅ Javob:** `setCount((prev) => prev + 1)` — functional update. Bu hozirgi closure'dagi `count` o'zgaruvchisiga emas, React beradigan eng so'nggi state'ga tayanadi. Bir hodisada (yoki tez ketma-ket) bir nechta yangilanish bo'lganda yoki yangilanish eski qiymatga bog'liq bo'lganda har doim functional update ishlating.

```tsx
// ❌ ikkalasi ham bir xil "count"'ga tayanadi → faqat +1 bo'ladi
setCount(count + 1);
setCount(count + 1);

// ✅ har biri oldingisining natijasiga tayanadi → +2 bo'ladi
setCount((c) => c + 1);
setCount((c) => c + 1);
```

### ❓ Lazy initialization nima?

**✅ Javob:** Agar boshlang'ich qiymatni hisoblash qimmat bo'lsa, `useState(expensiveCompute())` har render'da `expensiveCompute`'ni chaqiradi (natija faqat birinchi marta ishlatilsa ham). Buning o'rniga funksiya bering: `useState(() => expensiveCompute())` — u faqat birinchi render'da bir marta chaqiriladi.

```tsx
const [data, setData] = useState(() => JSON.parse(localStorage.getItem("x") ?? "[]"));
```

**⚠️ Ehtiyot bo'l:** State yangilanishi **asinxron** (batched). `setCount(...)`'dan keyin darhol `count`'ni o'qisangiz, eski qiymatni ko'rasiz. Yangi qiymat keyingi render'da paydo bo'ladi.

---

## useEffect (dependency, cleanup, qachon ishlaydi)

**💡 Tushuncha:** `useEffect` — render'dan **keyin** bajariladigan side effect'lar uchun: subscription, timer, DOM bilan tashqi sinxronizatsiya, data fetching. U render natijasini ekranga chizilgandan keyin ishlaydi.

```tsx
useEffect(() => {
  // side effect
  return () => {
    // cleanup
  };
}, [dep1, dep2]);
```

### ❓ Dependency array nima qiladi? Uch holatni ayting.

**✅ Javob:**
- **`[]` (bo'sh):** effect faqat mount'da bir marta ishlaydi, cleanup unmount'da.
- **`[a, b]`:** effect mount'da va `a` yoki `b` o'zgarganda ishlaydi.
- **array umuman berilmasa:** effect **har render'da** ishlaydi (kamdan-kam kerak, ehtiyot bo'ling).

Dependency array — "bu effect qaysi qiymatlarga sinxronlanadi" degan ro'yxat.

### ❓ Cleanup function qachon ishlaydi va nega kerak?

**✅ Javob:** Cleanup ikki holatda ishlaydi: (1) component unmount bo'lganda, (2) effect qayta ishlashidan **oldin** (dependency o'zgargani uchun). U subscription'larni bekor qilish, timer'larni tozalash, listener'larni olib tashlash uchun kerak — aks holda memory leak yoki ikki marta ishlovchi listener'lar paydo bo'ladi.

```tsx
useEffect(() => {
  const id = setInterval(() => console.log("tik"), 1000);
  return () => clearInterval(id); // muhim
}, []);
```

### ❓ useEffect render'dan oldin ishlaydimi yoki keyin?

**✅ Javob:** **Keyin** — DOM yangilanib, brauzer ekranga chizgandan so'ng, asinxron ishlaydi. Shuning uchun u UI'ni bloklamaydi. Agar DOM o'lchovini olib, ekranga chizishdan **oldin** o'zgartirish kerak bo'lsa, `useLayoutEffect` ishlatiladi.

### ❓ Nega StrictMode'da effect ikki marta ishlaydi?

**✅ Javob:** Development'da StrictMode component'ni mount → unmount → mount qiladi, shuning uchun effect ikki marta ishlaydi (cleanup oralig'ida). Bu sizning effect'ingiz **cleanup'siz yoki idempotent emasligini** ochib beradigan tekshiruv. To'g'ri cleanup yozsangiz, ikki marta ishlash hech qanday muammo tug'dirmaydi. Production'da bu takror yo'q.

**⚠️ Ehtiyot bo'l (keng xatolar):**
- Effect ichida ishlatilgan qiymatni dependency'ga qo'shmaslik → stale qiymat.
- Har render'da yangi object/array/function dependency sifatida berish → effect cheksiz ishlaydi.
- Effect ichida `setState` qilib, o'sha state'ni dependency'ga qo'yish → cheksiz loop.
- Cleanup yozmaslik → memory leak, dublikat listener.

---

## useEffect bilan data fetching

**💡 Tushuncha:** `useEffect` ichida ma'lumot olishda race condition va leak'dan ehtiyot bo'lish kerak. Tez almashadigan so'rovlar bir-birini "bosib o'tishi" mumkin.

```tsx
useEffect(() => {
  let cancelled = false;
  const controller = new AbortController();

  async function load() {
    try {
      const res = await fetch(`/api/users/${id}`, { signal: controller.signal });
      const data = await res.json();
      if (!cancelled) setUser(data);
    } catch (e) {
      if (!cancelled) setError(e);
    }
  }
  load();

  return () => {
    cancelled = true;
    controller.abort();
  };
}, [id]);
```

### ❓ Data fetching'da race condition nima va qanday hal qilinadi?

**✅ Javob:** `id` tez o'zgarsa, eski so'rov yangidan kechroq tugab, eski natijani state'ga yozib qo'yishi mumkin — UI noto'g'ri ma'lumot ko'rsatadi. Hal qilish: cleanup'da `cancelled = true` qilib, javob kelganda uni tekshiramiz (yoki `AbortController` bilan so'rovni bekor qilamiz). Shu sababli eski so'rovning natijasi e'tiborsiz qoldiriladi.

**⚠️ Ehtiyot bo'l:** Real loyihada data fetching uchun `useEffect`'ni qo'lda yozish o'rniga TanStack Query / SWR kabi library tavsiya etiladi — ular caching, retry, race condition'ni o'zi hal qiladi. Intervyuda esa cleanup mantiqini bilish muhim.

---

## useLayoutEffect (vs useEffect)

**💡 Tushuncha:** `useLayoutEffect` `useEffect`'ga o'xshaydi, lekin DOM o'zgargandan keyin, **ekranga chizilishdan oldin** sinxron ishlaydi. Brauzer "paint" qilishdan oldin bloklaydi.

### ❓ useLayoutEffect qachon useEffect'dan afzal?

**✅ Javob:** DOM'dan o'lchov (`getBoundingClientRect`) olib, darhol layout'ni o'zgartirish kerak bo'lganda — tooltip pozitsiyasi, element balandligini o'lchash, scroll pozitsiyasini tiklash. Agar buni `useEffect`'da qilsangiz, foydalanuvchi qisqa "miltillash"ni (flicker) ko'radi, chunki avval eski holat chiziladi, keyin tuzatiladi. `useLayoutEffect` esa chizishdan oldin tuzatadi.

**⚠️ Ehtiyot bo'l:** `useLayoutEffect` sinxron va paint'ni bloklaydi — ortiqcha ishlatish UI'ni sekinlashtiradi. Default'da `useEffect` ishlating; faqat vizual miltillash muammosi bo'lsa `useLayoutEffect`'ga o'ting.

---

## useContext (Context API)

**💡 Tushuncha:** Context — prop drilling'siz ma'lumotni daraxt bo'ylab pastga uzatish mexanizmi. `createContext` bilan yaratiladi, `Provider` bilan beriladi, `useContext` bilan o'qiladi.

```tsx
type Theme = "light" | "dark";
const ThemeContext = React.createContext<Theme>("light");

function App() {
  return (
    <ThemeContext.Provider value="dark">
      <Toolbar />
    </ThemeContext.Provider>
  );
}

function Toolbar() {
  const theme = React.useContext(ThemeContext); // "dark"
  return <div className={theme} />;
}
```

### ❓ Context prop drilling'ni qanday hal qiladi?

**✅ Javob:** Ma'lumotni har bir oraliq component orqali qo'lda uzatish o'rniga, Provider'ga bir marta beramiz, kerakli chuqurlikdagi har qanday component `useContext` bilan to'g'ridan-to'g'ri o'qiydi. Oraliq component'lar bu prop haqida bilmasligi mumkin.

### ❓ Context'ning perf ogohlantirishi nima?

**✅ Javob:** Context value o'zgarganda, o'sha context'ni **iste'mol qiladigan har bir** component re-render bo'ladi — value reference o'zgarsa, qiymat mazmunan bir xil bo'lsa ham. Agar Provider'ga har render'da yangi object bersangiz (`value={{ user }}`), barcha consumer'lar keraksiz re-render bo'ladi. Yechim: value'ni `useMemo` bilan memoize qilish, yoki tez-tez o'zgaradigan va kam o'zgaradigan ma'lumotni alohida context'larga bo'lish.

**⚠️ Ehtiyot bo'l:** Context — global state management library emas. Tez-tez o'zgaradigan, katta state uchun u perf jihatdan zaif; bunday holda Zustand/Redux yoki context'ni bo'lish to'g'riroq.

---

## useReducer (qachon)

**💡 Tushuncha:** `useReducer` — murakkab state mantig'i uchun `useState`'ga muqobil. State o'zgarishlari `reducer(state, action)` funksiyasi orqali markazlashtiriladi.

```tsx
type State = { count: number };
type Action = { type: "inc" } | { type: "dec" } | { type: "reset" };

function reducer(state: State, action: Action): State {
  switch (action.type) {
    case "inc": return { count: state.count + 1 };
    case "dec": return { count: state.count - 1 };
    case "reset": return { count: 0 };
  }
}

function Counter() {
  const [state, dispatch] = React.useReducer(reducer, { count: 0 });
  return (
    <>
      <span>{state.count}</span>
      <button onClick={() => dispatch({ type: "inc" })}>+</button>
    </>
  );
}
```

### ❓ useState o'rniga useReducer'ni qachon tanlaysiz?

**✅ Javob:** State murakkab bo'lganda yoki keyingi state oldingisiga bog'liq bo'lganda; bir nechta bog'liq qiymat birga o'zgarganda; o'zgarish mantig'i tarqoq va takrorlanganda. `useReducer` barcha o'zgarishlarni bitta sof funksiyaga jamlaydi — bu test qilish va o'qishni osonlashtiradi. Oddiy mustaqil qiymatlar uchun `useState` yetarli.

---

## useRef (DOM va mutable qiymat)

**💡 Tushuncha:** `useRef` `{ current }` object qaytaradi. U ikki maqsadda ishlatiladi: (1) DOM element'ga murojaat, (2) re-render keltirmaydigan mutable qiymat saqlash. `ref.current` o'zgarishi **re-render keltirmaydi**.

```tsx
// DOM ref
const inputRef = useRef<HTMLInputElement>(null);
// mutable qiymat (timer id, oldingi qiymat)
const timerRef = useRef<number | null>(null);
```

### ❓ useRef state'dan qanday farq qiladi?

**✅ Javob:** State o'zgarishi re-render keltiradi va UI'da ko'rinadi; ref o'zgarishi **re-render keltirmaydi**. Shuning uchun UI'ga ta'sir qiladigan ma'lumot uchun state, faqat eslab qolish kerak bo'lgan, render'ga ta'sir qilmaydigan ma'lumot (timer ID, instance, oldingi qiymat) uchun ref ishlating.

### ❓ Nega ref'ni render paytida o'qib/yozib bo'lmaydi?

**✅ Javob:** Render pure bo'lishi kerak — bir xil input bir xil natija berishi shart. Ref'ni render davomida o'qish/yozish bu qoidani buzadi (natija ref tarixiga bog'liq bo'lib qoladi) va concurrent rendering'da kutilmagan xatti-harakat keltiradi. Ref'ni faqat event handler yoki effect ichida ishlating.

**⚠️ Ehtiyot bo'l:** `ref.current = ...` qilib UI'ni yangilashni kutmang — u re-render qilmaydi. Ekranda ko'rinishi kerak bo'lsa, state ishlating.

---

## useMemo

**💡 Tushuncha:** `useMemo` qimmat hisoblash natijasini eslab qoladi (memoize). Dependency'lar o'zgarmaguncha, qayta hisoblamaydi.

```tsx
const sorted = useMemo(() => {
  return [...items].sort((a, b) => a.price - b.price);
}, [items]);
```

### ❓ useMemo'ni qachon ishlatish kerak?

**✅ Javob:** Ikki holatda: (1) haqiqatan qimmat hisoblashni har render'da takrorlamaslik uchun; (2) `React.memo` qilingan child'ga yoki boshqa hook dependency'siga **barqaror reference** (object/array) berish uchun. Aks holda har render'da yangi reference hosil bo'ladi.

**⚠️ Ehtiyot bo'l:** Har joyga `useMemo` qo'yish — premature optimization. Memoize'ning o'zi xotira va solishtirish xarajati bor. Oddiy, arzon hisoblashlarni memoize qilmang. Avval o'lchang.

---

## useCallback

**💡 Tushuncha:** `useCallback` — funksiya reference'ini memoize qiladi. `useCallback(fn, deps)` = `useMemo(() => fn, deps)`. Dependency'lar o'zgarmaguncha bir xil funksiya reference'i qaytariladi.

```tsx
const handleClick = useCallback(() => {
  doSomething(id);
}, [id]);
```

### ❓ Nega child prop uchun useCallback kerak?

**✅ Javob:** Har render'da inline funksiya (`onClick={() => ...}`) yangi reference hosil qiladi. Agar bu funksiyani `React.memo` qilingan child'ga prop sifatida bersangiz, reference har safar o'zgargani uchun `memo` ishlamay, child baribir re-render bo'ladi. `useCallback` funksiya reference'ini barqaror qiladi, shunda `memo` to'g'ri ishlaydi.

**⚠️ Ehtiyot bo'l:** `useCallback`'ning o'zi child memoize qilinmagan bo'lsa foyda bermaydi — odatda u `React.memo` bilan birga ma'no kasb etadi. `useCallback` faqat reference'ni barqarorlashtiradi, funksiyani "tezlashtirmaydi".

---

## useImperativeHandle

**💡 Tushuncha:** `useImperativeHandle` parent'ga child'ning ref orqali ochiladigan API'sini moslashtirish imkonini beradi — child o'zining ichki DOM'ini emas, faqat tanlangan metodlarni "ochadi". `forwardRef` (yoki React 19'da `ref` prop) bilan ishlatiladi.

```tsx
type InputHandle = { focus: () => void; clear: () => void };

const FancyInput = React.forwardRef<InputHandle>((_, ref) => {
  const innerRef = useRef<HTMLInputElement>(null);
  useImperativeHandle(ref, () => ({
    focus: () => innerRef.current?.focus(),
    clear: () => { if (innerRef.current) innerRef.current.value = ""; },
  }));
  return <input ref={innerRef} />;
});
```

### ❓ useImperativeHandle qachon kerak?

**✅ Javob:** Kamdan-kam. Faqat parent child'ning imperative metodlarini (`focus`, `play`, `scrollTo`, `clear`) chaqirishi kerak bo'lganda, lekin child'ning butun DOM'ini ochib qo'ymasdan. U declarative React'dan istisno, shuning uchun avval props bilan hal qilib bo'lmasligiga ishonch hosil qiling.

---

## useId

**💡 Tushuncha:** `useId` — barqaror, noyob ID generatsiya qiladi (asosan accessibility uchun: `label`/`input` bog'lash, `aria-*`). U server va client'da bir xil ID beradi (SSR mos).

```tsx
function Field() {
  const id = useId();
  return (
    <>
      <label htmlFor={id}>Email</label>
      <input id={id} />
    </>
  );
}
```

### ❓ Nega ID uchun `Math.random()` emas, useId ishlatiladi?

**✅ Javob:** `Math.random()` server va client'da turli qiymat beradi → SSR'da hydration mismatch. Shuningdek u har render'da o'zgaradi. `useId` esa barqaror va SSR-mos. Muhim: u **list key uchun emas** — faqat element ID'lari uchun.

---

## useTransition va useDeferredValue

**💡 Tushuncha:** Bu hook'lar yangilanishni "shoshilmaydigan" (non-urgent) deb belgilab, UI'ni javobgar (responsive) saqlaydi. Og'ir render'lar input kabi shoshilinch yangilanishlarni bloklamaydi.

```tsx
// useTransition
const [isPending, startTransition] = useTransition();
startTransition(() => setQuery(value)); // bu yangilanish past prioritetda

// useDeferredValue
const deferredQuery = useDeferredValue(query); // og'ir ro'yxat shu qiymatga tayanadi
```

### ❓ useTransition va useDeferredValue farqi nima?

**✅ Javob:** Ikkalasi ham bir maqsadga xizmat qiladi (og'ir yangilanishni kechiktirib UI'ni javobgar saqlash), lekin: `useTransition` siz **boshqaradigan state yangilanishini** transition'ga o'rab beradi (`startTransition`'ni o'zingiz chaqirasiz). `useDeferredValue` esa siz **boshqarmaydigan** qiymat (masalan prop'dan kelgan) uchun kechiktirilgan nusxa beradi. Qiymatni o'zingiz set qilsangiz — `useTransition`; tashqaridan kelgan qiymatni kechiktirish kerak bo'lsa — `useDeferredValue`.

**⚠️ Ehtiyot bo'l:** Bu hook'lar render'ni tezlashtirmaydi — ular faqat prioritet beradi. Real render og'ir bo'lsa, baribir `memo`/virtualizatsiya kabi optimizatsiya kerak.

---

## Rules of Hooks

**💡 Tushuncha:** Hook'lar ikki qoidaga bo'ysunadi: (1) faqat **top-level**'da chaqiring (loop, condition, nested funksiya ichida emas); (2) faqat React function component yoki custom hook ichida chaqiring.

### ❓ Nega hook'larni condition yoki loop ichida chaqirib bo'lmaydi?

**✅ Javob:** React hook'larni **chaqirilish tartibi** bo'yicha eslab qoladi (indeks bilan). Agar hook'ni shart ichiga qo'ysangiz, ba'zi render'da u chaqiriladi, ba'zida yo'q — tartib siljiydi va React state'larni noto'g'ri hook'ga bog'laydi. Shuning uchun har render'da **bir xil sondagi va tartibdagi** hook chaqirilishi shart. Shartli mantiq hook **ichida** bo'lishi mumkin, lekin hook chaqiruvi shartsiz bo'lishi kerak.

```tsx
// ❌ NOTO'G'RI
if (loggedIn) {
  const [x, setX] = useState(0);
}

// ✅ TO'G'RI
const [x, setX] = useState(0);
if (loggedIn) { /* x bilan ishlash */ }
```

---

## Stale closure muammosi

**💡 Tushuncha:** Stale closure — effect yoki callback eski render'dagi qiymatni "yodda saqlab" qolishi. JavaScript closure'i yaratilgan paytdagi o'zgaruvchini ushlaydi; agar dependency'ni yangilamasangiz, eski qiymat ishlatiladi.

```tsx
// ❌ count doim 0 bo'lib qoladi — closure eski count'ni ushlagan
useEffect(() => {
  const id = setInterval(() => {
    setCount(count + 1); // har doim 0 + 1
  }, 1000);
  return () => clearInterval(id);
}, []); // count dependency'da yo'q

// ✅ functional update stale closure'ni chetlab o'tadi
useEffect(() => {
  const id = setInterval(() => {
    setCount((c) => c + 1);
  }, 1000);
  return () => clearInterval(id);
}, []);
```

### ❓ Stale closure'ni qanday tuzatasiz?

**✅ Javob:** Uch yo'l: (1) kerakli qiymatni dependency array'ga qo'shish (effect qayta yaratiladi); (2) state yangilanishi uchun functional update ishlatish (`setX(prev => ...)`) — eski qiymatga tayanmaydi; (3) eng so'nggi qiymat doim kerak bo'lsa, uni `useRef`'da saqlab, effect ichida `ref.current` orqali o'qish.

**⚠️ Ehtiyot bo'l:** Linter (`eslint-plugin-react-hooks`) etishmayotgan dependency'ni ogohlantiradi. Uni "o'chirib qo'yish" o'rniga sababini tushunib hal qiling.

---

## Custom hooks

**💡 Tushuncha:** Custom hook — `use`'dan boshlanadigan, boshqa hook'lardan foydalanadigan funksiya. U **logikani** qayta ishlatish uchun (UI emas). Har bir component custom hook'dan o'z mustaqil state'ini oladi.

### ❓ useFetch custom hook

**✅ Javob:**

```tsx
function useFetch<T>(url: string) {
  const [data, setData] = useState<T | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<Error | null>(null);

  useEffect(() => {
    let cancelled = false;
    setLoading(true);
    fetch(url)
      .then((r) => r.json())
      .then((d) => { if (!cancelled) setData(d); })
      .catch((e) => { if (!cancelled) setError(e); })
      .finally(() => { if (!cancelled) setLoading(false); });
    return () => { cancelled = true; };
  }, [url]);

  return { data, loading, error };
}
```

### ❓ useLocalStorage custom hook

**✅ Javob:**

```tsx
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
```

### ❓ useDebounce custom hook

**✅ Javob:**

```tsx
function useDebounce<T>(value: T, delay = 300): T {
  const [debounced, setDebounced] = useState(value);
  useEffect(() => {
    const id = setTimeout(() => setDebounced(value), delay);
    return () => clearTimeout(id); // har o'zgarishda eski timer bekor
  }, [value, delay]);
  return debounced;
}
```

**⚠️ Ehtiyot bo'l:** Custom hook'da `use` prefix shart — linter shu prefixga qarab Rules of Hooks'ni tekshiradi. Custom hook **state'ni baham ko'rmaydi**; faqat **logikani** qayta ishlatadi (har chaqiruv alohida state oladi).

---

## Dependency array tuzoqlari

**💡 Tushuncha:** Dependency array tuzoqlari — React hook'larida eng ko'p uchraydigan bug manbai. Asosan reference barqarorligi va etishmayotgan dependency bilan bog'liq.

### ❓ Object/array dependency nega cheksiz loop keltiradi?

**✅ Javob:** Har render'da yangi object/array hosil bo'ladi (reference boshqa). Uni dependency'ga qo'ysangiz, React "o'zgardi" deb hisoblaydi va effect har render'da ishlaydi; agar effect state yangilasa — re-render → yana yangi object → cheksiz loop. Yechim: primitive dependency ishlating, yoki object/funksiyani `useMemo`/`useCallback` bilan barqarorlashtiring.

```tsx
// ❌ har render'da yangi options object → effect cheksiz ishlaydi
const options = { id };
useEffect(() => { load(options); }, [options]);

// ✅ primitive dependency
useEffect(() => { load({ id }); }, [id]);
```

### ❓ Etishmayotgan dependency nega xavfli?

**✅ Javob:** Effect ichida ishlatilgan, lekin array'da yo'q qiymat stale bo'ladi — effect eski qiymatni ko'radi va UI haqiqatga mos kelmaydi. Linterga ishoning: u etishmayotgan dependency'ni topadi. Effect tez-tez ishlamasligini xohlasangiz, dependency'ni o'chirish emas, balki kodni qayta tuzish (functional update, ref, yoki logikani effect tashqarisiga chiqarish) to'g'ri yo'l.

**⚠️ Ehtiyot bo'l:** `// eslint-disable-next-line react-hooks/exhaustive-deps` bilan ogohlantirishni o'chirish — bug'ni yashirish. Faqat sababini to'liq tushunganingizda va boshqa yo'l bo'lmaganda ishlating.

---

## Masalalar

> Yechimlar: [solutions/frontend/06-react-hooks.md](../solutions/frontend/06-react-hooks.md)

1. **useToggle:** `useToggle(initial = false)` custom hook yozing. U `[value, toggle, setTrue, setFalse]` qaytarsin. `toggle` functional update ishlatsin.

2. **useDebounce qidiruv:** Input qiymatini `useDebounce` bilan 400ms kechiktiring va kechiktirilgan qiymat o'zgarganda `console.log` (yoki fetch) chaqiring. Har bosishda emas, faqat foydalanuvchi to'xtaganda ishlasin.

3. **Interval bug'ni tuzating:** Berilgan `useEffect` sekundomeri har sekundda `setCount(count + 1)` qiladi, lekin hisob `1`'da qotib qoladi. Sababini (stale closure) tushuntiring va functional update bilan tuzating.

4. **useLocalStorage:** Yuqoridagi `useLocalStorage` hook'ini yozing va undan foydalanib, "dark mode" sozlamasini saqlaydigan kichik component qiling (sahifa reload bo'lganda saqlanib qolsin).

5. **useReducer todo:** Todo ro'yxatini `useReducer` bilan boshqaring: `add`, `toggle`, `remove` action'lari bo'lsin. Reducer'ni alohida sof funksiya sifatida yozing.

6. **memo + useCallback:** `React.memo` qilingan `<Child onClick={...} />` har safar re-render bo'lyapti. Sababini ko'rsating va `useCallback` bilan tuzating.

7. **useContext theme:** `ThemeContext` yarating (`light`/`dark`), `ThemeProvider` (value'ni `useMemo` bilan) yozing va chuqurdagi component'da `useContext` orqali o'qing hamda almashtiring.

8. **useFetch race condition:** `useFetch(url)` hook'ini yozing va `url` tez o'zgarganda eski javob yangisini bosib o'tmasligini (race condition) cleanup orqali ta'minlang.

---

← [Frontend bo'limiga qaytish](./README.md)
