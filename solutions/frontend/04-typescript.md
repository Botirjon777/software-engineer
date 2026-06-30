# TypeScript — Yechimlar

Bu fayl [`frontend/04-typescript.md`](../../frontend/04-typescript.md) dagi **Masalalar** bo'limining to'liq yechimlarini, kod va izohlar bilan, oson→qiyin tartibida saqlaydi.

---

## 1. `DeepReadonly<T>`

Obyektni rekursiv `readonly` qiladi. Mapped tip har bir key uchun, agar qiymat obyekt bo'lsa — o'ziga rekursiv qo'llaydi.

```ts
type DeepReadonly<T> = {
  readonly [K in keyof T]: T[K] extends object
    ? DeepReadonly<T[K]> // ichki obyektga rekursiya
    : T[K];
};

// Sinov
type Config = DeepReadonly<{
  name: string;
  nested: { value: number };
}>;
// { readonly name: string; readonly nested: { readonly value: number } }

const c: Config = { name: "x", nested: { value: 1 } };
// c.nested.value = 2; // ❌ Cannot assign to 'value' (readonly)
```

**Izoh:** `T[K] extends object` — funksiyalar ham `object`, shuning uchun ehtiyot bo'ling: funksiya tiplari ham "readonly"lanadi (zararsiz, lekin biling). Massivlar ham `object`, shuning uchun ular ham chuqur readonly bo'ladi.

---

## 2. `MyPick<T, K>`

`Pick` ni noldan. `K extends keyof T` constraint faqat `T` da mavjud key'larni ruxsat etadi.

```ts
type MyPick<T, K extends keyof T> = {
  [P in K]: T[P]; // faqat K key'lari bo'ylab iteratsiya
};

// Sinov
interface User { id: number; name: string; email: string; }
type UserPreview = MyPick<User, "id" | "name">;
// { id: number; name: string }

// MyPick<User, "phone">; // ❌ "phone" keyof User'ga assignable emas
```

**Izoh:** `Pick` (`MyExclude`'dan farqli) mavjud bo'lmagan key'da xato beradi — bu `K extends keyof T` constraint tufayli. Bu `Omit`'dan farqi (`Omit` typo'larni sokin o'tkazadi).

---

## 3. `MyExclude<T, U>` va `MyExtract<T, U>`

Distributive conditional tip — union'ning har a'zosiga alohida qo'llanadi.

```ts
// T ning U ga assignable a'zolarini OLIB TASHLAYDI
type MyExclude<T, U> = T extends U ? never : T;

// T ning U ga assignable a'zolarini SAQLAYDI
type MyExtract<T, U> = T extends U ? T : never;

// Sinov
type A = MyExclude<"a" | "b" | "c", "a" | "c">; // => "b"
type B = MyExtract<"a" | "b" | "c", "a" | "z">; // => "a"
```

**Izoh:** Naked type parameter `T` conditional ichida bo'lgani uchun **distribution** yuz beradi: `"a" | "b" | "c"` har biri alohida `extends` tekshiriladi. `never` union'da yo'qoladi, shuning uchun mos kelganlar (Exclude) yoki kelmaganlar (Extract) tushib qoladi.

---

## 4. `MyReturnType<T>`

Funksiya return tipini `infer` orqali ajratadi.

```ts
type MyReturnType<T> = T extends (...args: any[]) => infer R ? R : never;

// Sinov
type Fn = (id: number) => { name: string };
type R = MyReturnType<Fn>;      // => { name: string }
type R2 = MyReturnType<() => Promise<number>>; // => Promise<number>
type R3 = MyReturnType<string>; // => never (funksiya emas)
```

**Izoh:** `infer R` `extends` ifodasi ichida return pozitsiyasidan tipni ushlaydi. Agar `T` funksiya bo'lmasa, conditional `never` ga hal qiladi. Bu native `ReturnType<T>` ning aynan implementatsiyasi.

---

## 5. `isShape` type guard + exhaustiveness

Discriminated union ustida custom type guard va `never` exhaustiveness.

```ts
type Shape =
  | { kind: "circle"; r: number }
  | { kind: "square"; side: number }
  | { kind: "rect"; w: number; h: number };

// Custom type guard — `s is ...` predikat qaytaradi
function isCircle(s: Shape): s is Extract<Shape, { kind: "circle" }> {
  return s.kind === "circle";
}

function area(s: Shape): number {
  switch (s.kind) {
    case "circle": return Math.PI * s.r ** 2;
    case "square": return s.side ** 2;
    case "rect": return s.w * s.h;
    default: {
      // yangi kind qo'shilsa-yu, case yozilmasa — bu yerda xato chiqadi
      const _exhaustive: never = s;
      return _exhaustive;
    }
  }
}

// Sinov
const shapes: Shape[] = [
  { kind: "circle", r: 2 },
  { kind: "rect", w: 3, h: 4 },
];
if (isCircle(shapes[0])) shapes[0].r; // ✅ narrow bo'ldi
```

**Izoh:** `Extract<Shape, { kind: "circle" }>` — union'dan circle variantini ajratadi. `_exhaustive: never` — agar `Shape` ga yangi a'zo qo'shilsa va `case` yozilmasa, `s` `never` bo'lmaydi va compile xatosi chiqadi. Bu refactoring xavfsizligini ta'minlaydi.

---

## 6. `PartialBy<T, K>`

Faqat `K` key'larini ixtiyoriy qiladi, qolganlari majburiy qoladi. `Omit` + `Partial<Pick>` birlashmasi.

```ts
type PartialBy<T, K extends keyof T> = Omit<T, K> & Partial<Pick<T, K>>;

// Sinov
interface User { id: number; name: string; email: string; }
type DraftUser = PartialBy<User, "id" | "email">;
// { name: string } & { id?: number; email?: string }
// = { name: string; id?: number; email?: string }

const draft: DraftUser = { name: "Ali" }; // ✅ id va email ixtiyoriy
```

**Izoh:** `Omit<T, K>` `K` siz qolgan majburiy qismni beradi; `Partial<Pick<T, K>>` `K` ni ixtiyoriy qiladi; `&` ularni birlashtiradi. Bu juda keng tarqalgan amaliy pattern (masalan, yangi entity yaratishda `id` server beradi).

---

## 7. `UnwrapPromise<T>`

Ichma-ich joylashgan promise'larni rekursiv ochadi.

```ts
type UnwrapPromise<T> = T extends Promise<infer V>
  ? UnwrapPromise<V> // ichkaridan yana ochamiz (rekursiya)
  : T;

// Sinov
type A = UnwrapPromise<Promise<number>>;          // => number
type B = UnwrapPromise<Promise<Promise<string>>>; // => string
type C = UnwrapPromise<boolean>;                  // => boolean (promise emas)
```

**Izoh:** `infer V` promise ichidagi tipni ushlaydi, keyin natijani yana `UnwrapPromise` ga uzatamiz — shu tariqa har qanday chuqurlikdagi ichma-ich promise yechiladi. Native `Awaited<T>` aynan shunday rekursiv ishlaydi.

---

## 8. `Mutable<T>`

`readonly` modifier'ni `-readonly` orqali olib tashlaydi.

```ts
type Mutable<T> = {
  -readonly [K in keyof T]: T[K]; // - prefiks readonly'ni olib tashlaydi
};

// Sinov: as const obyekt readonly bo'ladi, Mutable uni qaytaradi
const config = { mode: "dark", retries: 3 } as const;
// tipi: { readonly mode: "dark"; readonly retries: 3 }

type EditableConfig = Mutable<typeof config>;
// { mode: "dark"; retries: 3 }  (readonly yo'q)

const c: EditableConfig = { mode: "dark", retries: 3 };
c.retries = 5; // ✅ endi o'zgartirsa bo'ladi
```

**Izoh:** `-readonly` va `-?` — mapping modifier'larni olib tashlash sintaksisi. `+readonly`/`+?` (default) ularni qo'shadi. `Mutable` — `Readonly` ning teskarisi.

---

## 9. `parse` overload

Input tipiga qarab har xil return tipli funksiya.

```ts
// Deklaratsiya signature'lari (public API)
function parse(x: string): string[];
function parse(x: number): number;
// Implementation signature (yashirin, chaqirib bo'lmaydi)
function parse(x: string | number): string[] | number {
  return typeof x === "string" ? x.split("") : x * 2;
}

// Sinov
const a = parse("hi"); // a: string[]  => ["h", "i"]
const b = parse(5);    // b: number    => 10
// const c = parse(true); // ❌ true hech qaysi overload'ga mos kelmaydi
```

**Izoh:** Chaqiruvchilar faqat ikkita deklaratsiya signature'ini ko'radi — `string` berilsa `string[]`, `number` berilsa `number`. Implementation signature (`string | number`) faqat ichki, chaqirilmaydi. Aniqroq overload'lar oldinga qo'yiladi.

---

## 10. `Route` tip + tip-xavfsiz `navigate`

`as const` massivdan string-literal union olish va undan funksiya tiplash.

```ts
const ROUTES = ["home", "about", "contact"] as const;

// (typeof ROUTES)[number] — massiv element tiplari union'i
type Route = (typeof ROUTES)[number]; // => "home" | "about" | "contact"

function navigate(route: Route): void {
  console.log("navigatsiya:", route);
}

// Sinov
navigate("home");    // ✅
navigate("about");   // ✅
// navigate("profile"); // ❌ "profile" Route'ga assignable emas

// Bonus: barcha route'lar bo'ylab xavfsiz iteratsiya
ROUTES.forEach((r) => navigate(r)); // r: Route — to'liq tip-xavfsiz
```

**Izoh:** `as const` massivni `readonly ["home", "about", "contact"]` tuple qiladi. `(typeof ROUTES)[number]` indexed access barcha element tiplarini union qiladi. Bu — bitta manba (`ROUTES`) dan ham runtime qiymat, ham compile-vaqt tipini olish kanonik pattern'i: yangi route qo'shsangiz, tip avtomatik yangilanadi.

---

← [TypeScript mavzusiga qaytish](../../frontend/04-typescript.md)
