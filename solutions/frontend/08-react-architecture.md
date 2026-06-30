# React Arxitektura va State Management — Yechimlar

Bu yerda [08-react-architecture.md](../../frontend/08-react-architecture.md) "Masalalar" bo'limining yechimlari. Arxitektura masalalarida "bitta to'g'ri javob" kam — muhimi tanlovni asoslash.

---

## 1. State turini tasniflash

| State | Tur | Vosita | Sabab |
|-------|-----|--------|-------|
| Savatdagi mahsulotlar | client (yoki server+client) | Zustand / Context (yoki backend cart bo'lsa Query) | foydalanuvchiga xos, sessiya davomida saqlanadi; agar serverda saqlansa — Query |
| Mahsulot katalogi | **server** | TanStack Query | API'dan keladi, cache/refetch/invalidation kerak |
| Qidiruv so'rovi va filtrlar | **URL** | router search params | ulashiladigan, bookmark qilinadigan, back/forward ishlasin |
| "Checkout modal ochiqmi" | client/UI | useState (lokal) | sof UI holati, hech qayerda saqlash shart emas |
| Foydalanuvchi profili | **server** | TanStack Query | server'dan keladi, eskirishi mumkin |
| Tanlangan til | client | Context (kam o'zgaradi) | global, lekin deyarli o'zgarmaydi → Context ideal |

**Izoh:** Eng ko'p xato — katalog va profilni "global client state" deb Redux/Context'ga qo'yish. Ular server state — Query bilan cache va invalidation tekin keladi. Qidiruv/filtrni URL'ga qo'yish UX'ni sezilarli yaxshilaydi.

---

## 2. Compound component (Accordion)

```tsx
import { createContext, useContext, useState, useMemo } from "react";

interface AccordionCtx {
  openId: string | null;
  toggle: (id: string) => void;
}
const AccordionContext = createContext<AccordionCtx | null>(null);

function useAccordion() {
  const ctx = useContext(AccordionContext);
  if (!ctx) throw new Error("Accordion.* faqat <Accordion> ichida ishlatiladi");
  return ctx;
}

function Accordion({ children }: { children: React.ReactNode }) {
  const [openId, setOpenId] = useState<string | null>(null);
  const value = useMemo<AccordionCtx>(
    () => ({ openId, toggle: (id) => setOpenId((cur) => (cur === id ? null : id)) }),
    [openId]
  );
  return <AccordionContext.Provider value={value}>{children}</AccordionContext.Provider>;
}

const ItemContext = createContext<string | null>(null);
function Item({ id, children }: { id: string; children: React.ReactNode }) {
  return <ItemContext.Provider value={id}>{children}</ItemContext.Provider>;
}

function Header({ children }: { children: React.ReactNode }) {
  const { toggle } = useAccordion();
  const id = useContext(ItemContext)!;
  return <button onClick={() => toggle(id)}>{children}</button>;
}

function Panel({ children }: { children: React.ReactNode }) {
  const { openId } = useAccordion();
  const id = useContext(ItemContext)!;
  return openId === id ? <div>{children}</div> : null;
}

Accordion.Item = Item;
Accordion.Header = Header;
Accordion.Panel = Panel;

// Foydalanish:
// <Accordion>
//   <Accordion.Item id="a">
//     <Accordion.Header>Birinchi</Accordion.Header>
//     <Accordion.Panel>Tarkib</Accordion.Panel>
//   </Accordion.Item>
// </Accordion>
```

**Izoh:** Ikki context: tashqi (`openId`/`toggle`) va har item uchun `id`. `useAccordion` guard — `<Accordion>` tashqarisida aniq xato beradi, bu API'ni xato ishlatishdan himoya qiladi. `value`ni `useMemo` qilish ortiqcha render oldini oladi.

---

## 3. Custom hook ekstraktsiyasi

```tsx
function useResource<T>(key: string, fetcher: () => Promise<T>) {
  const [data, setData] = useState<T | null>(() => {
    const cached = localStorage.getItem(key);
    return cached ? (JSON.parse(cached) as T) : null;
  });
  const [loading, setLoading] = useState(!data);
  const [error, setError] = useState<Error | null>(null);

  useEffect(() => {
    let cancelled = false;
    setLoading(true);
    fetcher()
      .then((res) => {
        if (cancelled) return;
        setData(res);
        localStorage.setItem(key, JSON.stringify(res));
      })
      .catch((e) => !cancelled && setError(e as Error))
      .finally(() => !cancelled && setLoading(false));
    return () => {
      cancelled = true; // unmount/key o'zgarsa eskirgan javobni e'tiborsiz qoldirish
    };
  }, [key]);

  return { data, loading, error };
}

// Komponent endi toza presentational:
function UserView({ id }: { id: string }) {
  const { data, loading, error } = useResource(`user:${id}`, () => fetchUser(id));
  if (loading) return <Skeleton />;
  if (error) return <ErrorState message={error.message} />;
  return <UserCard user={data!} />;
}
```

**Izoh:** Barcha mantiq (fetch, loading, error, cache, race guard) hook'da; komponent faqat holatni UI'ga moslaydi. `cancelled` bayrog'i unmount va `key` o'zgarishidagi race condition'dan himoya qiladi. Real loyihada bu hook o'rniga TanStack Query ishlatish yaxshiroq (masala 4'ga qarang).

---

## 4. Redux'ni TanStack Query bilan almashtirish

**Oldin (Redux thunk + slice — soddalashtirilgan):** action type'lar, `usersSlice`, `fetchUsers` thunk, `loading`/`error`/`data` holatlari, `dispatch(fetchUsers())` `useEffect`'da, selektorlar — taxminan 60+ qator boilerplate.

**Keyin:**

```tsx
function useUsers() {
  return useQuery({ queryKey: ["users"], queryFn: fetchUsers, staleTime: 30_000 });
}

function useAddUser() {
  const qc = useQueryClient();
  return useMutation({
    mutationFn: addUser,
    onSuccess: () => qc.invalidateQueries({ queryKey: ["users"] }),
  });
}

function UsersList() {
  const { data: users, isLoading, isError } = useUsers();
  const addUser = useAddUser();
  if (isLoading) return <Skeleton />;
  if (isError) return <ErrorState />;
  return (
    <>
      {users!.map((u) => <li key={u.id}>{u.name}</li>)}
      <button onClick={() => addUser.mutate({ name: "Yangi" })}>Qo'shish</button>
    </>
  );
}
```

**Yo'qolgan boilerplate:** action type'lar, slice, thunk, `loading`/`error`/`data` reducer mantiqi, manual dispatch, selektorlar, store ulanishi.

**Tekin kelgan xususiyat:** caching, dedupe (bir vaqtdagi bir xil so'rov birlashadi), background refetch (window focus'da), stale-while-revalidate, retry, `invalidateQueries` orqali avtomatik yangilanish, optimistic update imkoni.

---

## 5. Context'ni refaktor qilish

**Muammo:** `AppContext = { theme, auth, notifications }`. `notifications` real-time tez o'zgaradi → har xabarda butun ilova (theme/auth iste'molchilari ham) render bo'ladi.

**Yechim:**

```tsx
// Theme va auth — kam o'zgaradi → alohida, memoize qilingan Context
const ThemeContext = createContext<ThemeCtx | null>(null);
const AuthContext = createContext<AuthCtx | null>(null);

// Notifications — tez o'zgaradi → selektiv obunali store (Zustand)
const useNotifications = create<NotifState>((set) => ({
  items: [],
  add: (n) => set((s) => ({ items: [...s.items, n] })),
}));

function Providers({ children }: { children: React.ReactNode }) {
  const [theme, setTheme] = useState<Theme>("light");
  const [auth, setAuth] = useState<Auth | null>(null);
  const themeValue = useMemo(() => ({ theme, setTheme }), [theme]);
  const authValue = useMemo(() => ({ auth, setAuth }), [auth]);
  return (
    <ThemeContext.Provider value={themeValue}>
      <AuthContext.Provider value={authValue}>{children}</AuthContext.Provider>
    </ThemeContext.Provider>
  );
}

// Komponent faqat kerakli "slice"ga obuna:
function Bell() {
  const count = useNotifications((s) => s.items.length); // faqat count o'zgarsa render
  return <span>{count}</span>;
}
```

**Izoh:** Theme/auth kam o'zgaradigani uchun Context (memoize qilingan value bilan) mos. Notifications tez o'zgargani uchun Zustand — selector (`(s) => s.items.length`) orqali faqat kerakli qismga obuna bo'lib, ortiqcha render'ni kesadi. To'g'ri vosita = o'zgarish chastotasiga mos vosita.

---

## 6. react-hook-form + Zod

```tsx
import { useForm } from "react-hook-form";
import { zodResolver } from "@hookform/resolvers/zod";
import { z } from "zod";

const schema = z
  .object({
    email: z.string().email("Email noto'g'ri"),
    password: z.string().min(8, "Kamida 8 ta belgi"),
    confirm: z.string(),
    age: z.number({ invalid_type_error: "Yosh kiriting" }).min(18, "18+ bo'lishi kerak"),
  })
  .refine((d) => d.password === d.confirm, {
    message: "Parollar mos emas",
    path: ["confirm"], // xatoni confirm maydoniga biriktirish
  });

type FormValues = z.infer<typeof schema>;

function SignupForm() {
  const {
    register,
    handleSubmit,
    formState: { errors, isSubmitting },
  } = useForm<FormValues>({ resolver: zodResolver(schema) });

  return (
    <form onSubmit={handleSubmit((data) => api.signup(data))}>
      <input {...register("email")} />
      {errors.email && <span>{errors.email.message}</span>}

      <input type="password" {...register("password")} />
      {errors.password && <span>{errors.password.message}</span>}

      <input type="password" {...register("confirm")} />
      {errors.confirm && <span>{errors.confirm.message}</span>}

      <input type="number" {...register("age", { valueAsNumber: true })} />
      {errors.age && <span>{errors.age.message}</span>}

      <button disabled={isSubmitting}>Ro'yxatdan o'tish</button>
    </form>
  );
}
```

**Izoh:** Parol tasdig'i `.refine()` bilan sxema darajasida tekshiriladi va xato `path: ["confirm"]` orqali to'g'ri maydonga biriktiriladi. `FormValues` tipi `z.infer`'dan keladi — validatsiya va tip bitta manba'dan, qo'lda takror yo'q. `valueAsNumber` input qiymatini number'ga aylantiradi (aks holda string keladi).

---

## 7. Type-safe polymorphic Button

```tsx
type ButtonOwnProps = { variant?: "primary" | "secondary"; loading?: boolean };

type ButtonProps =
  | ({ as?: "button" } & ButtonOwnProps & React.ComponentPropsWithoutRef<"button">)
  | ({ as: "a"; href: string } & ButtonOwnProps & React.ComponentPropsWithoutRef<"a">);

function Button(props: ButtonProps) {
  const { variant = "primary", loading, ...rest } = props;
  const className = `btn btn-${variant}`;

  if (rest.as === "a") {
    const { as: _as, ...anchorProps } = rest;
    return <a className={className} {...anchorProps} />;
  }
  const { as: _as, ...buttonProps } = rest as Extract<ButtonProps, { as?: "button" }>;
  return <button className={className} disabled={loading} {...buttonProps} />;
}

// <Button onClick={...}>Yuborish</Button>            // button — onClick ruxsat
// <Button as="a" href="/x">Havola</Button>           // anchor — href majburiy
// <Button as="a" onClick={...}>...</Button>           // a uchun ham onClick OK
// <Button as="a">...</Button>                         // ❌ href yo'q → tip xatosi
```

**Izoh:** Discriminated union (`as` discriminant bo'yicha) har holatga mos native propslarni talab qiladi: `as="a"` bo'lganda `href` majburiy va anchor atributlari ruxsat etiladi; default holatda `<button>` atributlari. Bu noto'g'ri kombinatsiyani (anchor'siz `href`, yoki button'da `href`) kompilyatsiya vaqtida ushlaydi.

---

## 8. Feature-based migratsiya rejasi

**Maqsad tuzilish:**

```
src/
  app/                 # providers, router, global setup
  features/
    users/
      components/       # UserCard, UserList...
      hooks/            # useUsers, useAddUser
      api/              # fetchUsers, addUser
      types.ts
      index.ts          # public API (faqat shu yerdan import qilinadi)
    auth/
      ...
  shared/
    ui/                 # Button, Input, Dialog (haqiqatan umumiy)
    lib/                # formatDate, http client
    hooks/              # useDebouncedValue...
```

**Migratsiya qadamlari:**

1. **Feature'larni aniqlash** — domen bo'yicha (users, auth, checkout). Mavjud `components/`, `hooks/`, `services/`dagi fayllarni domeniga ko'ra guruhlash.
2. **Har feature'ga ko'chirish** — feature'ga xos komponent/hook/api'ni `features/<name>/` ostiga olib o'tish.
3. **Shared'ni ajratish** — *ikki yoki undan ko'p* feature ishlatadigan narsani (`Button`, `formatDate`, `http`) `shared/`ga qo'yish. Bitta feature ishlatsa — feature ichida qoldiring.
4. **Public API (`index.ts`)** — har feature faqat `index.ts`dan eksport qilgan narsani tashqariga beradi. Boshqa feature feature ichki fayliga to'g'ridan import qilmasligi kerak (ESLint `no-restricted-imports` bilan majburlash mumkin).
5. **`app/` qatlami** — routing, global providers (Query client, theme), entry point.

**Chegara qoidasi:** feature → shared'ga bog'lanishi mumkin; feature → boshqa feature'ga to'g'ridan bog'lanmasligi kerak (faqat public API orqali yoki app qatlamida ulanadi). Bu bog'liqlik yo'nalishini bir tomonlama qilib, modullikni saqlaydi.

---

← [Frontend bo'limiga qaytish](../../frontend/README.md)
