# React Arxitektura va State Management

Senior darajada React — bu komponent yozish emas, balki *tizimni loyihalash*: qaysi state qayerda yashaydi, papkalar qanday tashkillanadi, qaysi pattern qaysi muammoni hal qiladi va qachon biror kutubxonani umuman olib kelmaslik kerak. Eng yaxshi arxitektura — kerakli darajada sodda bo'lib, o'sishga tayyor turgan arxitektura.

**💡 Tushuncha:** Arxitektura bo'yicha qarorlarning aksariyati — trade-off. "To'g'ri javob" kontekstga (jamoa hajmi, ilova murakkabligi, o'sish bashorati) bog'liq. Intervyuda muhimi — *nima uchun* shu tanlovni qilganingizni asoslay olish.

## Mundarija

- [State turlari](#state-turlari)
- [Komponent dizayni patternlari](#komponent-dizayni-patternlari)
- [Custom hook — zamonaviy pattern](#custom-hook--zamonaviy-pattern)
- [Papka strukturasi](#papka-strukturasi)
- [State management variantlari taqqosi](#state-management-variantlari-taqqosi)
- [Qachon Redux kerak EMAS](#qachon-redux-kerak-emas)
- [Server state: data fetching va caching](#server-state-data-fetching-va-caching)
- [Error boundary va Suspense](#error-boundary-va-suspense)
- [Form kutubxonalari](#form-kutubxonalari)
- [Styling strategiyalari](#styling-strategiyalari)
- [Testing strategiyasi](#testing-strategiyasi)
- [TypeScript patternlar](#typescript-patternlar)
- [Design system](#design-system)
- [Savol-javoblar](#savol-javoblar)
- [Masalalar](#masalalar)

---

## State turlari

Arxitekturaning eng muhim qarori — state'ni **turi bo'yicha** ajratish. Har xil state har xil vositani talab qiladi. Ko'p loyihalar hamma state'ni bitta yechimga (masalan, Redux'ga) tiqib, muammoga duch keladi.

| Tur | Misol | Eng yaxshi vosita |
|-----|-------|-------------------|
| **Server state** | API'dan kelgan ma'lumot, cache, loading/error | TanStack Query / SWR |
| **Client/UI state** | modal ochiqmi, tab, theme, sidebar | useState / Zustand / Context |
| **URL state** | qidiruv, filtr, sahifa raqami, tanlangan id | URL (router, search params) |
| **Form state** | input qiymatlari, validatsiya, dirty/touched | react-hook-form |

**💡 Tushuncha:** **Server state — client state emas.** U serverda "haqiqiy" yashaydi, sizda faqat nusxasi (cache). U eskiradi (stale), qayta tiklanishi (refetch), invalidate bo'lishi kerak. Buni oddiy `useState`/Redux bilan boshqarish — ko'p qo'lda kod va xato manbai. Shuning uchun alohida sinf vositalari (TanStack Query) mavjud.

**URL state** ko'pincha unutiladi: filtr/qidiruv/sahifani URL'ga qo'yish — bookmark qilinadi, ulashiladi, back/forward ishlaydi. Bu state'ni `useState`'da saqlash — UX'ni qashshoqlashtiradi.

---

## Komponent dizayni patternlari

**Presentational vs Container (smart/dumb):**

- **Presentational** — faqat props oladi, UI chizadi, mantiq yo'q. Qayta ishlatiluvchan, oson test qilinadi.
- **Container** — ma'lumot oladi, state boshqaradi, presentational'larga uzatadi.

```tsx
// Presentational — "qanday ko'rinadi"
function UserCard({ name, avatar }: { name: string; avatar: string }) {
  return (
    <div>
      <img src={avatar} alt={name} />
      <span>{name}</span>
    </div>
  );
}
// Container — "qanday ishlaydi" (zamonaviy: ko'pincha custom hook)
function UserCardContainer({ id }: { id: string }) {
  const { data } = useUser(id);
  if (!data) return <Skeleton />;
  return <UserCard name={data.name} avatar={data.avatar} />;
}
```

**⚠️ Ehtiyot bo'l:** Hooks paydo bo'lgach, "container" qatlam ko'pincha **custom hook**ga aylandi. Endi qattiq container/presentational bo'linishi shart emas — mantiqni hook'ga, UI'ni komponentga ajratish ko'proq uchraydi. Lekin bo'linish g'oyasi (UI'ni mantiqdan ajratish) hali ham qimmatli.

**Compound components** — bir necha komponent birga ishlab, holatni implicit (context orqali) bo'lishadi. Foydalanuvchi tuzilishni o'zi quradi, moslashuvchanlik yuqori.

```tsx
<Tabs defaultValue="profile">
  <Tabs.List>
    <Tabs.Trigger value="profile">Profil</Tabs.Trigger>
    <Tabs.Trigger value="settings">Sozlamalar</Tabs.Trigger>
  </Tabs.List>
  <Tabs.Panel value="profile">...</Tabs.Panel>
  <Tabs.Panel value="settings">...</Tabs.Panel>
</Tabs>
```

```tsx
const TabsContext = createContext<{ value: string; setValue: (v: string) => void } | null>(null);

function Tabs({ defaultValue, children }: { defaultValue: string; children: React.ReactNode }) {
  const [value, setValue] = useState(defaultValue);
  const ctx = useMemo(() => ({ value, setValue }), [value]);
  return <TabsContext.Provider value={ctx}>{children}</TabsContext.Provider>;
}
function useTabs() {
  const ctx = useContext(TabsContext);
  if (!ctx) throw new Error("Tabs.* faqat <Tabs> ichida ishlaydi");
  return ctx;
}
```

**Render props** — komponent o'z renderlash mantiqini funksiya-prop sifatida oladi. Kuchli, lekin "wrapper hell" ga olib kelishi mumkin; ko'p hollarda custom hook bilan almashtirildi.

```tsx
<MouseTracker>{({ x, y }) => <p>{x}, {y}</p>}</MouseTracker>
```

**HOC (Higher-Order Component)** — komponentni olib, yangilangan komponent qaytaradigan funksiya (`withAuth(Component)`). Zamonaviy kodda kamroq — hook'lar ko'p holatda toza yechim. Lekin third-party (`connect`, `memo`) hali ishlatadi.

**💡 Tushuncha:** Tarixiy evolyutsiya: HOC → render props → **custom hooks**. Hooks ko'p eski patternni almashtirdi, chunki ular wrapper qo'shmaydi, kompozitsiya qiladi va tip-xavfsiz. Lekin compound components va render props o'z o'rni bor (ayniqsa UI kutubxonalarda).

---

## Custom hook — zamonaviy pattern

Custom hook — qayta ishlatiluvchan mantiqni (state + effect) ajratishning asosiy zamonaviy usuli. UI'dan mustaqil, kompozitsiyalanadi, oson test qilinadi.

```tsx
function useToggle(initial = false) {
  const [on, setOn] = useState(initial);
  const toggle = useCallback(() => setOn((v) => !v), []);
  return [on, toggle] as const; // tuple tipi saqlanadi
}

function useLocalStorage<T>(key: string, initial: T) {
  const [value, setValue] = useState<T>(() => {
    const raw = localStorage.getItem(key);
    return raw ? (JSON.parse(raw) as T) : initial;
  });
  useEffect(() => {
    localStorage.setItem(key, JSON.stringify(value));
  }, [key, value]);
  return [value, setValue] as const;
}
```

**💡 Tushuncha:** Yaxshi custom hook — bitta mas'uliyat, aniq nom (`use...`), va UI qaytarmaydi (qiymat/funksiya qaytaradi). "Container" mantiqini hook'ga ko'chirish komponentni toza presentational qoldiradi va testni soddalashtiradi.

---

## Papka strukturasi

Ikki asosiy yondashuv:

**Type-based (turi bo'yicha):**

```
src/
  components/
  hooks/
  utils/
  services/
  pages/
```

Kichik loyihada tushunarli. Lekin o'sganda, bitta feature'ni o'zgartirish uchun ko'p papkani aylanib chiqasiz; bog'liq fayllar tarqoq.

**Feature-based (xususiyat bo'yicha) — kattaroq loyihalar uchun afzal:**

```
src/
  features/
    auth/
      components/
      hooks/
      api/
      types.ts
      index.ts        # public API
    checkout/
      components/
      hooks/
      ...
  shared/             # haqiqatan umumiy narsalar
    ui/
    lib/
  app/                # routing, providers, global setup
```

**💡 Tushuncha:** Feature-based — kod *o'zgarish sababiga* yaqin joylashadi (colocation). Bir feature ustida ishlovchi dasturchi bitta papkada qoladi. `index.ts` orqali "public API" berib, feature'lar orasidagi bog'liqlikni nazorat qilasiz (ichki fayllarga to'g'ridan import qilmaslik).

**⚠️ Ehtiyot bo'l:** "Shared" papka axlat qutisiga aylanmasligi kerak — faqat *haqiqatan* bir necha feature ishlatadigan narsa shared bo'ladi. Erta abstraksiya qilmang; ikki marta takrorlangach umumlashtiring.

---

## State management variantlari taqqosi

| Vosita | Eng mos | Kuchli tomoni | Cheklov |
|--------|---------|---------------|---------|
| **useState** | lokal UI state | sodda, nol kutubxona | global emas, prop drilling |
| **useReducer** | murakkab lokal state, ko'p o'tish (transition) | bashoratli, test oson | global emas |
| **Context** | kam o'zgaradigan global (theme, auth, locale) | built-in, DI | selektiv obuna yo'q, tez state'da render muammosi |
| **Redux Toolkit** | katta, murakkab global state, DevTools/middleware kerak | bashoratli, ekotizim, time-travel | boilerplate, o'rganish egri chizig'i |
| **Zustand** | sodda global client state | minimal API, selektiv obuna, hook'ga asoslangan | Redux'cha qattiq struktura yo'q |
| **Jotai** | atom-asosli, granular state | bottom-up, juda granular render | yangiroq mental model |
| **TanStack Query / SWR** | **server state** | cache, refetch, invalidation, dedupe | client state uchun emas |

**Tanlash mantiqi (qaror daraxti):**

1. Bu **server state**mi? → TanStack Query / SWR. (Aksariyat ilovada "global state" deb o'ylangan narsa aslida server state.)
2. URL'ga tegishlimi (filtr, sahifa)? → URL state (router).
3. Lokal UI? → `useState`/`useReducer`.
4. Bir necha komponent kerak, kam o'zgaradi? → Context.
5. Tez-tez o'zgaradigan, keng tarqalgan client state? → Zustand / Jotai.
6. Juda katta, murakkab, audit/middleware/time-travel kerakmi? → Redux Toolkit.

**💡 Tushuncha:** Zamonaviy stack ko'pincha: **TanStack Query (server)** + **Zustand yoki Context (oz client state)** + **URL (filtr/navigatsiya)** + **react-hook-form (formalar)**. Bu kombinatsiya Redux'ni ko'p loyihada keraksiz qiladi.

---

## Qachon Redux kerak EMAS

Redux Toolkit — kuchli, lekin ko'p loyiha uni keraksiz olib keladi.

- **"Global state kerak" deganingiz aslida server state bo'lsa** — TanStack Query yaxshiroq (cache, refetch, invalidation tekin keladi; Redux'da bularni qo'lda yozasiz).
- **State asosan lokal/UI bo'lsa** — `useState` yetarli.
- **Kichik/o'rta ilovada** — Redux boilerplate'i va abstraksiyasi foydadan ko'p yuk.
- **Faqat prop drilling muammosi bo'lsa** — Context yoki kompozitsiya yetadi.

**Redux qachon HAQIQATAN foydali:**
- Juda katta client state, ko'p o'zaro bog'liq yangilanish.
- Time-travel debugging, qat'iy audit log, murakkab middleware (saga, optimistic flow).
- Katta jamoa — qat'iy, bashoratli struktura intizom beradi.

**⚠️ Ehtiyot bo'l:** "Hamma Redux ishlatadi" — yomon asos. Intervyuda Redux'ni *qachon ishlatmaslik* kerakligini ayta olish senior signal. Default'ni Redux'dan boshlamang.

---

## Server state: data fetching va caching

TanStack Query (yoki SWR) server state'ni declarative boshqaradi:

```tsx
function useUser(id: string) {
  return useQuery({
    queryKey: ["user", id],
    queryFn: () => fetchUser(id),
    staleTime: 60_000, // 1 daqiqa "fresh" hisoblanadi
  });
}

function Profile({ id }: { id: string }) {
  const { data, isLoading, isError } = useUser(id);
  if (isLoading) return <Skeleton />;
  if (isError) return <ErrorState />;
  return <UserCard user={data} />;
}
```

Mutation va invalidation:

```tsx
const queryClient = useQueryClient();
const mutation = useMutation({
  mutationFn: updateUser,
  onSuccess: () => {
    queryClient.invalidateQueries({ queryKey: ["user"] }); // cache yangilanadi
  },
});
```

**Tekin keladigan narsalar:** caching, dedupe (bir vaqtda bir xil so'rov bir marta), background refetch, stale-while-revalidate, retry, pagination/infinite, optimistic update.

**💡 Tushuncha:** `queryKey` — cache'ning kaliti. To'g'ri kalit dizayni (parametrlarni kalitga kiritish) — invalidation va cache mosligi uchun markaziy. Bu "Redux store'ni qo'lda yangilash"dan ancha kam xato beradi.

---

## Error boundary va Suspense

**Error boundary** — render paytidagi xatolarni ushlab, fallback UI ko'rsatadigan komponent. Faqat **class component** bo'la oladi (yoki `react-error-boundary` kutubxonasi).

```tsx
class ErrorBoundary extends React.Component<
  { fallback: React.ReactNode; children: React.ReactNode },
  { hasError: boolean }
> {
  state = { hasError: false };
  static getDerivedStateFromError() {
    return { hasError: true };
  }
  componentDidCatch(error: Error) {
    logToService(error); // monitoring (Sentry)
  }
  render() {
    return this.state.hasError ? this.props.fallback : this.props.children;
  }
}
```

**⚠️ Ehtiyot bo'l:** Error boundary event handler ichidagi, async, yoki SSR xatolarini ushlamaydi — faqat render/lifecycle/constructor xatolarini. Async xato uchun `try/catch` yoki query kutubxonasining `isError` holatidan foydalaning. Boundary'larni strategik joylashtiring (per-route yoki per-widget), butun ilovani bitta boundary bilan o'rab, kichik xatoda ham bo'sh ekran bermang.

**Suspense** — async qism tayyor bo'lguncha fallback ko'rsatadi. `React.lazy`, RSC va mos data kutubxonalari (TanStack Query `useSuspenseQuery`) bilan ishlaydi.

```tsx
<ErrorBoundary fallback={<ErrorState />}>
  <Suspense fallback={<Skeleton />}>
    <ProfileWithSuspense id={id} />
  </Suspense>
</ErrorBoundary>
```

---

## Form kutubxonalari

Murakkab formalarda `useState`'ni har inputga qo'yish — ko'p re-render va boilerplate. **react-hook-form** uncontrolled (ref-asosli) yondashuv bilan kam render va yaxshi performance beradi.

```tsx
import { useForm } from "react-hook-form";
import { zodResolver } from "@hookform/resolvers/zod";
import { z } from "zod";

const schema = z.object({
  email: z.string().email("Email noto'g'ri"),
  age: z.number().min(18, "18 dan katta bo'lishi kerak"),
});
type FormValues = z.infer<typeof schema>;

function SignupForm() {
  const { register, handleSubmit, formState: { errors } } =
    useForm<FormValues>({ resolver: zodResolver(schema) });

  return (
    <form onSubmit={handleSubmit((data) => api.signup(data))}>
      <input {...register("email")} />
      {errors.email && <span>{errors.email.message}</span>}
      <input type="number" {...register("age", { valueAsNumber: true })} />
      <button type="submit">Yuborish</button>
    </form>
  );
}
```

**💡 Tushuncha:** Zod (yoki Yup) bilan sxema validatsiyasi — bitta manba'dan ham runtime validatsiya, ham TypeScript tipi (`z.infer`) keladi. Bu form va API chegarasida tipni qo'lda takrorlashni yo'qotadi.

---

## Styling strategiyalari

| Strategiya | Misol | Trade-off |
|-----------|-------|-----------|
| **CSS Modules** | `styles.button` | scoped, oddiy, build-vaqt; dinamik stil cheklangan |
| **Utility-first** | Tailwind | tez, izchil, kichik CSS; markup "shovqinli" |
| **CSS-in-JS (runtime)** | styled-components, Emotion | dinamik, komponentga yaqin; runtime narx, RSC bilan qiyin |
| **Zero-runtime CSS-in-JS** | vanilla-extract, Linaria | tip-xavfsiz, build-vaqt; o'rnatish murakkabroq |

**💡 Tushuncha:** RSC va performance e'tibori bilan trend **zero-runtime** (Tailwind, vanilla-extract) tomon. Runtime CSS-in-JS (styled-components) hali keng tarqalgan, lekin server components bilan ishqalanadi va bundle/runtime narxi bor. Tanlovni jamoa tanishligi va SSR/RSC talabiga qarab qiling.

---

## Testing strategiyasi

**React Testing Library (RTL) falsafasi:** "foydalanuvchidek test qil" — implementatsiya detalini emas, *xulq-atvor*ni test qiling. Komponent ichki state'ini emas, ekranda nima ko'rinishini va interaksiya natijasini tekshiring.

```tsx
import { render, screen } from "@testing-library/react";
import userEvent from "@testing-library/user-event";

test("submit qilinganda xush kelibsiz ko'rsatadi", async () => {
  render(<Greeter />);
  await userEvent.type(screen.getByLabelText(/ism/i), "Ali");
  await userEvent.click(screen.getByRole("button", { name: /yuborish/i }));
  expect(screen.getByText(/xush kelibsiz, ali/i)).toBeInTheDocument();
});
```

- **Query'larni rol/label bo'yicha tanlang** (`getByRole`, `getByLabelText`) — `getByTestId` oxirgi chora. Bu accessibility'ni ham tekshiradi.
- **Tarmoqni mock qiling** — MSW (Mock Service Worker) bilan tarmoq darajasida, fetch'ni emas.
- **Test piramidasi:** ko'p unit/komponent test (RTL), kamroq integratsiya, eng kam E2E (Playwright/Cypress).

**⚠️ Ehtiyot bo'l:** Implementatsiyaga bog'liq test (ichki state, metod chaqiruvini tekshirish) — refaktorda sinadi, lekin xulq buzilmagan. Bunday testlar yolg'on signal beradi va refaktorni qiyinlashtiradi. Foydalanuvchi ko'rgan/qilgan narsani test qiling.

---

## TypeScript patternlar

**Props uchun discriminated union** — mumkin bo'lmagan holatlarni tip darajasida taqiqlash:

```tsx
type ButtonProps =
  | { variant: "link"; href: string }
  | { variant: "button"; onClick: () => void };

function Button(props: ButtonProps) {
  if (props.variant === "link") return <a href={props.href}>...</a>;
  return <button onClick={props.onClick}>...</button>;
}
// <Button variant="link" onClick={...} /> — tip xatosi, mantiqan to'g'ri
```

**Generic komponent:**

```tsx
interface ListProps<T> {
  items: T[];
  renderItem: (item: T) => React.ReactNode;
  keyOf: (item: T) => string;
}
function List<T>({ items, renderItem, keyOf }: ListProps<T>) {
  return <ul>{items.map((i) => <li key={keyOf(i)}>{renderItem(i)}</li>)}</ul>;
}
```

**Native element propslarini kengaytirish:**

```tsx
interface Props extends React.ComponentPropsWithoutRef<"button"> {
  loading?: boolean;
}
function MyButton({ loading, children, ...rest }: Props) {
  return <button {...rest}>{loading ? "..." : children}</button>;
}
```

**💡 Tushuncha:** Yaxshi tiplar — "noto'g'ri ishlatib bo'lmaydigan" API yaratadi. Discriminated union, `as const`, generic, va `ComponentPropsWithoutRef` — komponent API'sini xavfsiz va qulay qiladi. `React.FC`'dan ko'pchilik voz kechgan (implicit children, generic qiyin) — props'ni to'g'ridan-to'g'ri tiplang.

---

## Design system

Design system — qayta ishlatiluvchan, izchil UI komponentlar va token'lar to'plami (rang, masofa, tipografiya). Yaxshi tuzilishi:

- **Tokens** (manba: rang, spacing, radius) → tema sifatida (CSS variables yoki TS object).
- **Primitives / UI komponentlar** (`Button`, `Input`, `Dialog`) — accessible, variant'li, neytral.
- **Patterns** — primitivlardan tuzilgan murakkabroq bloklar.
- **Headless asos** — Radix UI / React Aria xulq + a11y beradi, siz stil berasiz. Komplekt logikani (focus trap, keyboard nav) qaytadan yozishdan saqlaydi.

```tsx
// variant'lar bilan tip-xavfsiz Button (cva yoki shunga o'xshash)
type Variant = "primary" | "secondary" | "ghost";
interface ButtonProps extends React.ComponentPropsWithoutRef<"button"> {
  variant?: Variant;
}
```

**💡 Tushuncha:** Design system'ning maqsadi — izchillik + tezlik. Trade-off: dastlab ko'p investitsiya, lekin masshtabda chiqim qaytadi. Kichik loyihada to'liq design system qurish — over-engineering; tayyor (shadcn/ui, MUI) dan boshlash ko'pincha to'g'riroq.

---

## Savol-javoblar

### ❓ Server state va client state orasidagi farq nima va nega muhim?

**✅ Javob:** Server state — server'da "haqiqiy" yashaydigan, sizda faqat nusxasi (cache) bo'lgan ma'lumot; u eskiradi, refetch/invalidate bo'lishi kerak (foydalanuvchi ro'yxati, profil). Client state — faqat brauzerda yashaydigan UI holati (modal ochiqmi, theme). Muhim, chunki ularni bir vositaga (Redux'ga) tiqish server state'ning cache/refetch/dedupe muammosini qo'lda hal qilishga majbur qiladi. To'g'ri yondashuv — server state uchun TanStack Query, client state uchun useState/Zustand.

### ❓ Qachon Redux KERAK EMAS?

**✅ Javob:** Aksariyat hollarda. "Global state" deb o'ylangan narsa ko'pincha server state — u uchun TanStack Query yaxshiroq. State asosan lokal bo'lsa — useState; faqat prop drilling bo'lsa — Context yoki kompozitsiya; oz client state — Zustand. Redux haqiqatan time-travel, qat'iy audit, murakkab middleware yoki juda katta o'zaro bog'liq client state kerak bo'lgandagina oqlanadi. Default'ni Redux'dan boshlamaslik kerak.

### ❓ Context performance jihatdan qachon muammo va alternativasi nima?

**✅ Javob:** Context selektiv obunani bermaydi — qiymat o'zgarsa, barcha iste'molchi render bo'ladi. Tez-tez o'zgaradigan state'da bu ko'p ortiqcha render beradi. Yana `value` object har render'da yangi reference bo'lsa, hatto o'zgarmagan qiymatda ham render bo'ladi (yechim: `useMemo`). Tez o'zgaradigan global state uchun selektiv obunali Zustand/Jotai/Redux yaxshiroq; Context kam o'zgaradigan narsalar (theme, auth, locale) uchun ideal.

### ❓ Compound component pattern nima va qachon ishlatasiz?

**✅ Javob:** Bir necha komponent (masalan, `Tabs`, `Tabs.List`, `Tabs.Trigger`, `Tabs.Panel`) birga ishlab, holatni implicit context orqali bo'lishadi. Foydalanuvchi tuzilishni o'zi quradi → katta moslashuvchanlik va "drill" qilinadigan ko'p prop kerak emas. Qachon: qayta ishlatiluvchan, moslashuvchan UI komponentlar uchun (tabs, accordion, select, menu). Trade-off: implementatsiya murakkabroq va API kamroq "yo'naltirilgan".

### ❓ HOC, render props va custom hook orasida qanday tanlaysiz?

**✅ Javob:** Zamonaviy default — **custom hook**: wrapper qo'shmaydi, kompozitsiyalanadi, tip-xavfsiz, "wrapper hell" yo'q. HOC va render props ko'p holatda hook'ga o'tdi. Lekin render props/compound hali UI kutubxonalarda foydali (render boshqaruvini berish kerak bo'lganda), HOC esa third-party API'lar (`connect`, `memo`) yoki cross-cutting wrapper uchun qoladi. Mantiq qayta ishlatish kerak bo'lsa — hook; renderlash boshqaruvi kerak bo'lsa — render props/compound.

### ❓ Feature-based va type-based papka strukturasini qanday solishtirasiz?

**✅ Javob:** Type-based (components/, hooks/, utils/) kichik loyihada sodda, lekin o'sganda bog'liq fayllar tarqab ketadi — bir feature'ni o'zgartirish ko'p papkani aylanishni talab qiladi. Feature-based (features/auth/, features/checkout/) kodni o'zgarish sababiga yaqin joylaydi (colocation), jamoa ishini izolyatsiya qiladi va `index.ts` public API orqali bog'liqlikni nazorat qiladi. Katta loyiha uchun feature-based afzal; "shared" faqat haqiqatan umumiy narsa uchun.

### ❓ TanStack Query nimani "tekin" beradi va nega uni qo'lda yozmaslik kerak?

**✅ Javob:** Caching, dedupe (bir xil so'rovni birlashtirish), background refetch, stale-while-revalidate, retry, pagination/infinite, optimistic update va `queryKey` orqali invalidation. Buni Redux/useState bilan qo'lda yozish ko'p kod va xato manbai (race condition, eskirgan cache, takroriy so'rov). Declarative query kutubxonasi bu muammolarni sinovdan o'tgan tarzda hal qiladi va kodni qisqartiradi.

### ❓ Error boundary nimani ushlaydi va nimani ushlamaydi?

**✅ Javob:** Ushlaydi: render, lifecycle metod va constructor ichidagi xatolarni (fallback UI ko'rsatadi). Ushlamaydi: event handler ichidagi, async (setTimeout, promise), SSR va boundary'ning o'zidagi xatolarni. Async/handler xatolari uchun `try/catch` yoki query kutubxonasining `isError` holatidan foydalaning. Boundary'larni strategik (per-route/per-widget) joylashtiring, butun ilovani bitta boundary bilan o'ramang.

### ❓ react-hook-form nega ko'p re-render muammosini hal qiladi?

**✅ Javob:** U asosan uncontrolled (ref-asosli) yondashuvni ishlatadi — har input'ning qiymati React state'da emas, DOM'da ref orqali kuzatiladi. Shuning uchun har klaviatura bosishida butun forma re-render bo'lmaydi (controlled `useState` yondashuvidan farqli). Bu katta formalarda performance va kam boilerplate beradi; Zod/Yup resolver bilan sxema validatsiyasi va `z.infer` orqali tip ham keladi.

### ❓ RTL falsafasi nima va "implementatsiyaga bog'liq" test nima uchun yomon?

**✅ Javob:** RTL falsafasi — komponentni foydalanuvchidek test qilish: ekranda nima ko'rinishi va interaksiya natijasini tekshirish, ichki state/metodni emas. Query'larni rol/label bo'yicha tanlash a11y'ni ham tekshiradi. Implementatsiyaga bog'liq test (ichki state, private metod) refaktorda sinadi, lekin xulq buzilmagan — bu yolg'on signal beradi va refaktorni qiyinlashtiradi. Foydalanuvchi ko'rgan/qilgan narsani test qilish testni mustahkam qiladi.

### ❓ React.FC nega ko'pchilik tomonidan tark etildi?

**✅ Javob:** `React.FC` implicit `children` qo'shadi (komponent children qabul qilmasa ham), generic komponent yozishni qiyinlashtiradi va default props bilan yaxshi ishlamaydi. Zamonaviy uslub — props'ni to'g'ridan-to'g'ri tiplash (`function C(props: Props)`) va kerak bo'lsa `children`ni aniq e'lon qilish. Bu aniqroq va moslashuvchanroq.

### ❓ Discriminated union props qanday muammoni hal qiladi?

**✅ Javob:** "Mumkin bo'lmagan holat"ni tip darajasida taqiqlaydi. Masalan, `variant: "link"` bo'lsa `href` majburiy va `onClick` taqiqlangan; `variant: "button"` bo'lsa aksincha. Bu noto'g'ri prop kombinatsiyasini kompilyatsiya vaqtida ushlaydi, runtime tekshiruv va kommentariya o'rniga tip kafolat beradi. Natija — "noto'g'ri ishlatib bo'lmaydigan" API.

### ❓ CSS-in-JS (runtime) va zero-runtime orasida qanday tanlaysiz?

**✅ Javob:** Runtime CSS-in-JS (styled-components, Emotion) — dinamik va komponentga yaqin, lekin runtime narx bor va RSC bilan ishqalanadi. Zero-runtime (Tailwind, vanilla-extract, Linaria) — stil build vaqtida hosil bo'ladi, runtime narx yo'q, RSC bilan mos, performance yaxshi. Trend zero-runtime tomon (ayniqsa SSR/RSC bilan). Tanlovni jamoa tanishligi, dinamik stil ehtiyoji va SSR/RSC talabiga qarab qiling.

### ❓ "Shared" papka qanday axlat qutisiga aylanib qoladi va buni qanday oldini olasiz?

**✅ Javob:** Dasturchilar "balki boshqa joyda kerak bo'lar" deb hamma narsani shared'ga tashlaydi, natijada bog'liqliklar chalkashadi va feature izolyatsiyasi yo'qoladi. Oldini olish: faqat *haqiqatan* bir necha feature ishlatadigan narsani shared qiling; erta abstraksiya qilmang (ikki marta takrorlangach umumlashtiring); shared ichini ham tuzilmali (ui/, lib/, hooks/) qiling va public API'ni nazorat qiling.

---

## Masalalar

> Yechimlar: [solutions/frontend/08-react-architecture.md](../solutions/frontend/08-react-architecture.md)

1. **State turini tasniflash.** E-commerce ilovasida: savatdagi mahsulotlar, mahsulot katalogi, qidiruv so'rovi va filtrlar, "checkout modal ochiqmi", foydalanuvchi profili, tanlangan til. Har birini server/client/URL/form state'ga ajrating va qaysi vositani tanlashingizni asoslang.

2. **Compound component.** `Accordion`, `Accordion.Item`, `Accordion.Header`, `Accordion.Panel` ni context bilan TypeScript'da yozing. `useAccordion` hook'i `<Accordion>` tashqarisida ishlatilsa, aniq xato bersin.

3. **Custom hook ekstraktsiyasi.** Berilgan "container" komponentda fetch, loading, error va localStorage cache mantig'i bor. Uni `useResource<T>` custom hook'iga ajrating, komponentni toza presentational qoldiring.

4. **Redux'ni TanStack Query bilan almashtirish.** Foydalanuvchilar ro'yxatini Redux thunk + slice bilan boshqaradigan kod berilgan. Uni TanStack Query'ga ko'chiring, qaysi boilerplate yo'qolganini va qaysi xususiyat (cache, refetch, invalidation) tekin kelganini sanab bering.

5. **Context'ni refaktor qilish.** Bitta `AppContext` ichida theme, auth va real-time bildirishnomalar bor; bildirishnomalar tez o'zgaradi. Performance muammosini aniqlang va context'larni qayta loyihalashtiring, har biri uchun to'g'ri vosita tanlang.

6. **react-hook-form + Zod.** Ro'yxatdan o'tish formasini (email, parol, parolni tasdiqlash, yosh) react-hook-form va Zod sxemasi bilan quring. Parol tasdig'i mosligini sxema darajasida tekshiring va `z.infer` bilan tipni chiqaring.

7. **Type-safe polymorphic Button.** `as` prop orqali `<button>` yoki `<a>` bo'la oladigan, har holatga mos native propslarni tip darajasida talab qiladigan `Button` komponentini yozing (discriminated union yoki polymorphic pattern).

8. **Feature-based migratsiya.** Type-based (`components/`, `hooks/`, `services/`) tuzilishdagi loyihani feature-based'ga ko'chirish rejasini tuzing: qaysi narsa feature'ga, qaysi narsa shared'ga ketadi, public API (`index.ts`) chegaralarini qanday qo'yasiz.

---

← [Frontend bo'limiga qaytish](./README.md)
