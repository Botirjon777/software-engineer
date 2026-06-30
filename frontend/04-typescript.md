# TypeScript — Intervyu uchun

TypeScript intervyularda frontend pozitsiyalar uchun deyarli majburiy. Bu fayl tip tizimini amaliy savol-javob ko'rinishida yoritadi: structural typing, generics, narrowing, utility va conditional tiplar hamda muhim compiler option'lar. Maqsad — tiplarni nafaqat ishlatish, balki ular qanday ishlashini tushunish.

## Mundarija

- [Nega TypeScript](#nega-typescript)
- [Structural typing](#structural-typing)
- [Asosiy tiplar](#asosiy-tiplar)
- [Interface vs type](#interface-vs-type)
- [Union va intersection](#union-va-intersection)
- [Literal tiplar](#literal-tiplar)
- [Enum (va nega ba'zilar undan qochadi)](#enum-va-nega-bazilar-undan-qochadi)
- [Generics](#generics)
- [Narrowing va type guard](#narrowing-va-type-guard)
- [`unknown` vs `any` vs `never`](#unknown-vs-any-vs-never)
- [Utility tiplar](#utility-tiplar)
- [Mapped tiplar](#mapped-tiplar)
- [Conditional tiplar](#conditional-tiplar)
- [`keyof` va indexed access](#keyof-va-indexed-access)
- [`as const`](#as-const)
- [Function overload](#function-overload)
- [Declaration file va `declare`](#declaration-file-va-declare)
- [Strict mode va strictNullChecks](#strict-mode-va-strictnullchecks)
- [Type assertion](#type-assertion)
- [React'da generics](#reactda-generics)
- [Muhim compiler option'lar](#muhim-compiler-optionlar)
- [Masalalar](#masalalar)

---

## Nega TypeScript

**💡 Tushuncha:** TypeScript — JavaScript'ning **statik tipli** superset'i; u oddiy JS'ga kompilyatsiya (transpile) bo'ladi. Tiplar faqat compile vaqtida mavjud va chiqishdan to'liq **o'chiriladi** (erased) — runtime'da tip tekshiruvi yo'q.

### ❓ Nega TypeScript ishlatamiz?

**✅ Javob:** TypeScript tip xatolarini runtime emas, compile vaqtida ushlaydi, bu butun xato turkumlarini oldini oladi (noto'g'ri shakl uzatish, typo, `undefined` ga kirish). U yuqori darajadagi tooling beradi — autocomplete, inline hujjat, xavfsiz refactoring, go-to-definition — chunki muharrir ma'lumot shakllaringizni tushunadi. U yashovchi hujjat vazifasini ham bajaradi va katta kodbazani ancha boshqarib bo'ladigan qiladi. Narxi — build qadami va biroz oldindan tip yozish mehnati.

**⚠️ Ehtiyot bo'l:** Tiplar compile vaqtida o'chiriladi. `if (x instanceof MyInterface)` qila **olmaysiz** — interface'lar runtime'da mavjud emas. Runtime validatsiya uchun haqiqiy tekshiruvlar (yoki Zod kabi kutubxona) kerak.

---

## Structural typing

**💡 Tushuncha:** TypeScript **structural typing** ("duck typing") ishlatadi: moslik tipning **shakli** bilan aniqlanadi, nomi yoki deklaratsiya identiteti bilan emas. Agar kerakli member'lar bo'lsa — u mos keladi.

### ❓ Structural typing nima? Nominal typing'dan farqi?

**✅ Javob:** Structural tizimda ikki tip strukturasi mos kelsa — mos keladi: qiymat tipga assignable bo'ladi, agar unda kamida barcha talab qilingan xususiyatlar mos tiplar bilan bo'lsa. **Nominal** tizimlarda (Java, C#) tiplar nom/deklaratsiya bo'yicha aniq bog'langan bo'lishi shart. TypeScript structural, shuning uchun to'g'ri shaklli object literal interface'ni hech "implement" qilmasdan qondiradi.

```ts
interface Point { x: number; y: number; }
function dist(p: Point) { return Math.hypot(p.x, p.y); }

const thing = { x: 3, y: 4, label: "qo'shimcha" };
dist(thing); // ✅ OK — x va y bor; o'zgaruvchilar uchun qo'shimcha prop ruxsat
```

**⚠️ Ehtiyot bo'l:** **Excess property check** faqat to'g'ridan-to'g'ri uzatilgan **fresh object literal** ga qo'llanadi. `dist({ x: 3, y: 4, label: "..." })` `label` da xato beradi, lekin avval o'zgaruvchiga tayinlash (yuqoridagidek) o'tadi. Nominal typing'ni taqlid qilish uchun **branded type** ishlating: `type UserId = string & { __brand: "UserId" }`.

---

## Asosiy tiplar

**💡 Tushuncha:** Asosiy tiplar: `string`, `number`, `boolean`, `null`, `undefined`, `symbol`, `bigint`, `object`, massiv (`T[]` yoki `Array<T>`), tuple (`[string, number]`), `any`, `unknown`, `never`, `void`.

### ❓ `void` va `undefined` farqi nimada?

**✅ Javob:** `void` — qaytarish qiymatining yo'qligi — "qaytarish qiymatiga tayanmang" degan ma'noni bildiruvchi funksiya return tipi sifatida ishlatiladi. `undefined` — haqiqiy qiymat. `() => void` funksiya qiymat qaytarishi mumkin, lekin chaqiruvchi uni e'tiborsiz qoldirishi kerak (bu `forEach` kabi array callback'lar qiymat qaytaradigan funksiyalarni qabul qilishi uchun foydali).

```ts
const nums: number[] = [1, 2, 3];
const tuple: [string, number] = ["yosh", 30];
let u: unknown = fetchData();
function log(): void { console.log("salom"); }
```

---

## Interface vs type

**💡 Tushuncha:** Ikkalasi ham obyekt shaklini tasvirlaydi. **`interface`** kengaytiriladi va **declaration merging** ni qo'llab-quvvatlaydi. **`type`** moslashuvchanroq — union, primitive, tuple va computed/mapped tiplarni alias qila oladi.

### ❓ Qachon `interface`, qachon `type`?

**✅ Javob:** **`interface`** ni — kengaytiriladigan yoki implement qilinadigan obyekt/class shakllari uchun va declaration merging kerakda (masalan, uchinchi tomon tiplarini augment qilish) ishlating. **`type`** ni — union, intersection, tuple, primitive, function tip va mapped/conditional logika kerakda ishlating. Oddiy obyektlar uchun ko'p o'rinda ustma-ust tushadi; jamoalar ko'pincha public API uchun `interface`, qolgani uchun `type` ni standartlashtiradi.

```ts
interface User { id: number; name: string; }
interface User { email: string; }        // ✅ declaration merging — User'da endi 3 ta

type ID = string | number;                // ✅ union — interface buni qila olmaydi
type Pair = [number, number];             // tuple
type Handler = (e: Event) => void;        // function tip

interface Admin extends User { role: string; }   // interface kengaytirish
type Admin2 = User & { role: string };            // intersection ekvivalenti
```

**⚠️ Ehtiyot bo'l:** `interface` **declaration merging** ni qo'llab-quvvatlaydi (bir xil nomli ikki interface birlashadi); `type` esa **yo'q** (takroriy nom xato beradi). Shuning uchun kutubxona augmentatsiyasi interface ishlatadi.

---

## Union va intersection

**💡 Tushuncha:** **Union** (`A | B`) — "yo A, yo B". **Intersection** (`A & B`) — "ham A, ham B birlashgan".

### ❓ Union vs intersection'ni tushuntiring.

**✅ Javob:** Union — qiymat bir nechta tipdan biri; **narrow** qilmaguningizcha faqat barcha a'zolarda umumiy bo'lgan member'larga kira olasiz. Intersection bir nechta tipni hammasi member'lariga ega bittasiga birlashtiradi. Union — tanlov haqida; intersection — birlashtirish haqida.

```ts
type Status = "loading" | "success" | "error";   // literal'lar union'i

function format(x: string | number) {
  // x.toUpperCase()  ❌ number'da yo'q — avval narrow qiling
  return typeof x === "string" ? x.toUpperCase() : x.toFixed(2);
}

type Draggable = { drag(): void };
type Resizable = { resize(): void };
type Widget = Draggable & Resizable;   // IKKALA method ham bo'lishi shart
```

**⚠️ Ehtiyot bo'l:** Mos kelmaydigan primitive tiplarni intersect qilish `never` beradi (masalan `string & number`), chunki hech qaysi qiymat ikkalasi ham bo'la olmaydi.

---

## Literal tiplar

**💡 Tushuncha:** **Literal tip** qiymatni bitta aniq konstantaga cheklaydi (`"GET"`, `42`, `true`). Union bilan birga ular to'g'ri qiymatlarning yopiq to'plamini modellashtiradi.

### ❓ Literal tiplar nimaga kerak?

**✅ Javob:** Ular qiymatlarni aniq ruxsat etilgan konstantalar bilan cheklaydi, compile vaqtida noto'g'ri argumentlarni ushlaydi va exhaustive `switch` tekshiruvlarini quvvatlantiradi. HTTP method, action type, o'lcham va state machine'lar uchun keng tarqalgan.

```ts
type Method = "GET" | "POST" | "PUT" | "DELETE";
function request(url: string, method: Method) { /* ... */ }
request("/x", "PATCH"); // ❌ Argument Method'ga assignable emas
```

**⚠️ Ehtiyot bo'l:** `const` literal tiplarni infer qiladi (`"GET"`), lekin `let` `string` ga kengayadi (widen). Obyekt/massivlarda literal'ni saqlash uchun `as const` ishlating.

---

## Enum (va nega ba'zilar undan qochadi)

**💡 Tushuncha:** **Enum** nomlangan konstantalar to'plamini belgilaydi. Numeric enum avto-oshadi; string enum aniq string'lar saqlaydi. `const enum` compile vaqtida inline qilinadi (runtime obyekt yo'q).

### ❓ Nega ba'zilar enum'dan qochadi?

**✅ Javob:** Oddiy enum'lar **runtime kod emit qiladi** (ikki tomonlama lookup obyekt), bu TS'ning "tiplar o'chiriladi" modeliga zid keladi va tree-shaking'da kutilmagan natija berishi mumkin. Numeric enum'lar tip-xavfsiz emas (ba'zi konfiglarda istalgan son assignable) va reverse mapping'ga ega. `const enum` inline orqali runtime obyektdan qochadi, lekin `isolatedModules` (Babel/SWC/esbuild ishlatadi) ostida buziladi. Ko'p jamoalar **string literal union** yoki `as const` obyektni afzal ko'radi — ular soddaroq, to'liq o'chiriladigan va bir xil darajada tip-xavfsiz.

```ts
enum Color { Red, Green, Blue }     // runtime obyekt emit qiladi; Color.Red === 0

const enum Dir { Up, Down }          // inline; runtime obyekt yo'q

// ✅ Ko'pincha afzal alternativa:
const Color2 = { Red: "red", Green: "green" } as const;
type Color2 = (typeof Color2)[keyof typeof Color2]; // "red" | "green"
```

**⚠️ Ehtiyot bo'l:** `const enum` ko'p zamonaviy toolchain'larda (esbuild, Babel) `isolatedModules`/`preserveConstEnums` bilan taqiqlangan, chunki ular fayllarni izolyatsiyada kompilyatsiya qiladi va inline qila olmaydi. Portativlik uchun string-literal union afzal.

---

## Generics

**💡 Tushuncha:** **Generics** tiplarni parametr qiladi, shuning uchun funksiya/class/tip ko'p tiplar ustida ishlaydi va tip munosabatlarini saqlaydi. Imkoniyat talab qilish uchun **constraint** (`extends`) va ixtiyoriy parametr uchun **default** qo'shing.

### ❓ Nega generics? Constraint'li generic ko'rsating.

**✅ Javob:** Generics qayta ishlatiladigan, tip-xavfsiz abstraksiyalar beradi — ular `any` ga murojaat qilish o'rniga input va output tiplari orasidagi munosabatni saqlaydi. Constraint (`T extends ...`) `T` da xususiyatlarni taxmin qilishga imkon beradi; default (`T = ...`) fallback beradi.

```ts
function identity<T>(x: T): T { return x; }
const n = identity(5);          // T number sifatida infer qilinadi

// Constraint: T da length bo'lishi shart
function longest<T extends { length: number }>(a: T, b: T): T {
  return a.length >= b.length ? a : b;
}
longest([1, 2], [1]);           // ✅ massivlarda length bor
// longest(1, 2);               // ❌ number'da length yo'q

// Default'li generic interface
interface ApiResponse<T = unknown> { data: T; status: number; }

// Ikki tip parametr bilan keyed kirish
function getProp<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}
```

**⚠️ Ehtiyot bo'l:** Haddan tashqari constraint qilmang yoki `any` ga default bermang. Xavfsizroq default sifatida `unknown` ishlating. Yana, generics faqat haqiqiy tip munosabati bo'lganda yordam beradi — faqat bitta o'rinda ishlatilgan generic odatda keraksiz.

---

## Narrowing va type guard

**💡 Tushuncha:** **Narrowing** — TS'ning keng tipni kod tarmog'i ichida aniqroq tipga torini control-flow tahlili orqali aniqlash usuli. **Type guard** — narrowing'ni ishga tushiruvchi konstruksiyalar.

### ❓ Tipni narrow qilish usullarini sanang.

**✅ Javob:**
- **`typeof`** — primitive'lar: `typeof x === "string"`.
- **`instanceof`** — class instance'lari: `x instanceof Date`.
- **`in`** — xususiyat mavjudligi: `"swim" in animal`.
- **Truthiness / equality** — `if (x)`, `x === null`.
- **Discriminated union** — umumiy literal tag bo'yicha switch.
- **Custom type guard** — `arg is Type` qaytaruvchi funksiya.

```ts
// Discriminated union — `kind` tag narrowing'ni boshqaradi
type Shape =
  | { kind: "circle"; r: number }
  | { kind: "square"; side: number };

function area(s: Shape): number {
  switch (s.kind) {
    case "circle": return Math.PI * s.r ** 2;   // s circle'ga narrow bo'ldi
    case "square": return s.side ** 2;
    default: {
      const _exhaustive: never = s;             // exhaustiveness tekshiruvi
      return _exhaustive;
    }
  }
}

// Custom type guard
function isString(x: unknown): x is string {
  return typeof x === "string";
}
```

**⚠️ Ehtiyot bo'l:** `never` exhaustiveness hiylasi — yangi union a'zosi qo'shsangiz-u `case` ni unutsangiz, compiler xato beradi. Custom guard'lar **tekshirilmagan assertion** — agar predikat logikasi noto'g'ri bo'lsa, siz compiler'ga yolg'on aytdingiz.

---

## `unknown` vs `any` vs `never`

**💡 Tushuncha:** `any` tip tekshiruvini o'chiradi (escape hatch). `unknown` — tip-xavfsiz top type — istalgan narsani qabul qiladi, lekin ishlatishdan oldin narrow majbur qiladi. `never` — bottom type — unga hech qaysi qiymat assignable emas; u erishib bo'lmaydigan kod yoki imkonsiz tiplarni ifodalaydi.

### ❓ `unknown`, `any`, `never` ni solishtiring.

**✅ Javob:**
- **`any`** — tekshiruvdan to'liq voz kechadi. Unga/undan istalgan narsa assignable; istalgan narsani chaqirib/kirib bo'ladi. Qochish kerak: u tarqaladi va haqiqiy xatolarni o'chiradi.
- **`unknown`** — xavfsiz muqobil. Unga istalgan narsani assign qila olasiz, lekin u bilan biror narsa qilishdan oldin **narrow** qilishingiz shart. API javoblari va `catch` o'zgaruvchilari uchun ideal.
- **`never`** — hech qachon yuz bermaydigan qiymatlarni ifodalaydi: doim throw/loop qiluvchi funksiya, tugagan union tipi yoki imkonsiz intersection.

```ts
let a: any = 5;     a.foo.bar();          // ✅ kompilyatsiya, ❌ crash bo'lishi mumkin
let u: unknown = 5; // u.toFixed();        ❌ avval narrow qiling
if (typeof u === "number") u.toFixed();   // ✅

function fail(msg: string): never { throw new Error(msg); }
type Impossible = string & number;        // never
```

**⚠️ Ehtiyot bo'l:** `unknown` faqat `unknown`/`any` ga assignable, `any` esa hamma narsaga assignable. Zamonaviy TS'da `useUnknownInCatchVariables` ni yoqing — shunda `catch (e)` `unknown` bo'ladi va xavfsiz boshqarishni majbur qiladi.

---

## Utility tiplar

**💡 Tushuncha:** Mavjud tiplarni transformatsiya qiluvchi o'rnatilgan generic tiplar. Bularni yoddan biling.

### ❓ Keng tarqalgan utility tiplarni ko'rib chiqing.

**✅ Javob:**

```ts
interface User { id: number; name: string; email: string; }

Partial<User>;          // barcha prop ixtiyoriy     { id?, name?, email? }
Required<User>;         // barcha prop majburiy (? ni olib tashlaydi)
Readonly<User>;         // barcha prop readonly
Pick<User, "id" | "name">;   // qism                { id, name }
Omit<User, "email">;         // key'larni olib tashlash  { id, name }
Record<string, number>;      // key→tip map         { [k: string]: number }

// Union'lar ustida:
Exclude<"a" | "b" | "c", "a">;   // => "b" | "c"   (union'dan olib tashlash)
Extract<"a" | "b", "a" | "z">;   // => "a"         (mos kelganini saqlash)
NonNullable<string | null>;       // => string

// Funksiyalar ustida:
type Fn = (id: number) => Promise<User>;
ReturnType<Fn>;          // => Promise<User>
Parameters<Fn>;          // => [id: number]
Awaited<ReturnType<Fn>>; // => User   (Promise'ni ochadi)
```

**⚠️ Ehtiyot bo'l:** `Omit` olib tashlanayotgan key mavjudligini tekshirmaydi (`Pick`'dan farqli), shuning uchun typo sokin no-op bo'ladi. `Awaited<T>` ichma-ich joylashgan promise'larni rekursiv ochadi va `await` aynan shu qiymatni qaytaradi.

---

## Mapped tiplar

**💡 Tushuncha:** **Mapped type** boshqa tipning key'lari bo'ylab `[K in keyof T]` orqali iteratsiya qilib yangi tip hosil qiladi. `Partial`, `Readonly` va boshqalar aynan shunday qurilgan.

### ❓ `Partial` va `Mutable` ni noldan yozing.

**✅ Javob:** Key'lar bo'ylab iteratsiya qiling va **mapping modifier** larni qo'llang (`?`, `readonly`, qo'shish/olib tashlash uchun `+`/`-`).

```ts
type MyPartial<T> = { [K in keyof T]?: T[K] };
type MyReadonly<T> = { readonly [K in keyof T]: T[K] };
type Mutable<T> = { -readonly [K in keyof T]: T[K] };   // readonly'ni olib tashlash
type Concrete<T> = { [K in keyof T]-?: T[K] };          // ixtiyoriylikni olib tashlash

// `as` bilan key remapping (TS 4.1+)
type Getters<T> = {
  [K in keyof T as `get${Capitalize<string & K>}`]: () => T[K];
};
// Getters<{ name: string }> => { getName: () => string }
```

**⚠️ Ehtiyot bo'l:** Key remapping (`as`) va template literal tiplar key'larni qayta nomlash uchun, `as never` esa shartli ravishda key'larni filtrlash uchun ishlatiladi.

---

## Conditional tiplar

**💡 Tushuncha:** **Conditional type** munosabatga qarab tiplar orasida tanlaydi: `T extends U ? X : Y`. `infer` bilan ichidan tip ajratib olishingiz mumkin.

### ❓ Conditional tip va `infer` qanday ishlaydi?

**✅ Javob:** `T extends U ? X : Y` `extends` tekshiruvini baholaydi va `X` yoki `Y` ga hal qiladi. **Union** ustida conditional **distribute** bo'ladi (har a'zoga qo'llanadi). `infer` mos kelgan pozitsiyadan tip o'zgaruvchisini ushlaydi — `ReturnType` funksiyaning return tipini aynan shunday ajratadi.

```ts
type IsString<T> = T extends string ? true : false;
IsString<"hi">;   // => true
IsString<42>;     // => false

// Union ustida distributiv:
type ToArray<T> = T extends any ? T[] : never;
ToArray<string | number>;   // => string[] | number[]

// `infer` tip ajratadi:
type MyReturnType<T> = T extends (...args: any[]) => infer R ? R : never;
type Unwrap<T> = T extends Promise<infer V> ? V : T;
Unwrap<Promise<number>>;    // => number
```

**⚠️ Ehtiyot bo'l:** Distribution faqat tekshirilayotgan tip **naked** type parametr bo'lganda yuz beradi. Union'ni yaxlit holda tekshirmoqchi bo'lsangiz, distribution'ni **o'chirish** uchun uni tuple'ga o'rang (`[T] extends [U]`).

---

## `keyof` va indexed access

**💡 Tushuncha:** `keyof T` — `T` ning xususiyat key'lari union'i. **Indexed access** `T[K]` — `K` xususiyatining tipini oladi.

### ❓ `keyof` va `T[K]` ni tushuntiring.

**✅ Javob:** `keyof T` barcha key nomlarini literal tip sifatida union qaytaradi. `T[K]` (indexed access / lookup tip) `K` key'da saqlangan tipni beradi. Generics bilan birga ular tip-xavfsiz xususiyatga kirishni ta'minlaydi, bunda return tip haqiqiy key'ni kuzatadi.

```ts
interface User { id: number; name: string; }
type UserKeys = keyof User;        // => "id" | "name"
type NameType = User["name"];      // => string
type Values = User[keyof User];    // => number | string  (barcha qiymat tiplari)

function pluck<T, K extends keyof T>(obj: T, keys: K[]): T[K][] {
  return keys.map((k) => obj[k]);
}
```

**⚠️ Ehtiyot bo'l:** String index signature'li obyektda (`{[k: string]: number}`) `keyof` `string | number` bo'ladi (JS'da number key'lar string'ga coerce bo'ladi). Ma'lum key'siz tipda `keyof` `never` bo'lishi mumkin.

---

## `as const`

**💡 Tushuncha:** **Const assertion** (`as const`) literalni **chuqur readonly** qiladi va uni eng aniq literal tiplarga narrow qiladi, kengaytirmasdan.

### ❓ `as const` nima qiladi?

**✅ Javob:** U TS'ga eng tor literal tiplarni infer qilish va hamma narsani `readonly` belgilashni aytadi: string/number literal'lar literal qoladi (`string`/`number` ga kengaymaydi), massivlar `readonly` tuple bo'ladi, obyekt xususiyatlari `readonly` bo'ladi. Config obyektlar, action creator'lar va qiymatlardan union tip olish uchun ajoyib.

```ts
const config = { mode: "dark", retries: 3 } as const;
// tipi: { readonly mode: "dark"; readonly retries: 3 }

const ROUTES = ["home", "about", "contact"] as const;
type Route = (typeof ROUTES)[number];   // => "home" | "about" | "contact"

// as const'siz, mode string'ga kengayadi:
const c2 = { mode: "dark" };            // { mode: string }
```

**⚠️ Ehtiyot bo'l:** `as const` massivlar `readonly` tuple bo'ladi — ularga endi `push` qila olmaysiz, va mutable massiv kutilgan joyga uzatish xato beradi. `(typeof arr)[number]` orqali union olish — kanonik pattern.

---

## Function overload

**💡 Tushuncha:** **Overload** bitta funksiya uchun bir nechta call signature e'lon qiladi, shunda return tip argument tiplariga bog'liq bo'la oladi. Bitta implementation signature (kengroq, chaqiruvchilardan yashirin) barcha holatlarni boshqaradi.

### ❓ Function overload qachon va qanday ishlatiladi?

**✅ Javob:** Overload'ni funksiya input'lariga qarab turli tiplar qaytarganda va bitta signature buni aniq ifodalay olmaganda ishlating. Bir nechta deklaratsiya signature, keyin bitta implementation signature (u hammasi bilan mos bo'lishi shart, lekin to'g'ridan-to'g'ri chaqirilmaydi) yoziladi.

```ts
function parse(x: string): string[];
function parse(x: number): number;
function parse(x: string | number): string[] | number {   // implementation
  return typeof x === "string" ? x.split("") : x * 2;
}
const a = parse("hi");  // string[] tipida
const b = parse(5);     // number tipida
```

**⚠️ Ehtiyot bo'l:** Implementation signature public API'ning **qismi emas** — chaqiruvchilar faqat e'lon qilingan overload'lardan foydalana oladi. Tartib muhim: aniqroq overload'larni oldinga qo'ying. Ko'pincha bitta generic yoki union signature overload'dan toza.

---

## Declaration file va `declare`

**💡 Tushuncha:** **Declaration file** (`.d.ts`) mavjud JS'ning tiplarini implementatsiyasiz tasvirlaydi. `declare` ambient tiplarni kiritadi — kod emit qilmasdan TS'ga "bu runtime'da mavjud" deydi.

### ❓ `.d.ts` fayllar va `declare` nimaga kerak?

**✅ Javob:** `.d.ts` fayllar JavaScript kutubxonalari uchun tip ma'lumotini beradi (masalan DefinitelyTyped'dan `@types/*` paketlar), shunda iste'molchilar kutubxona TS'da yozilmagan bo'lsa ham tip tekshiruvi va autocomplete oladi. `declare` runtime'da mavjud, lekin TS ko'rmaydigan ambient o'zgaruvchi, modul yoki global'larni e'lon qiladi — masalan inject qilingan global'lar, CSS-module import'lari yoki `window` augmentatsiyasi.

```ts
// global.d.ts
declare const __APP_VERSION__: string;     // bundler tomonidan build vaqtida inject

declare global {
  interface Window { dataLayer: unknown[]; }
}

declare module "*.svg" {                    // .svg import'lari tip tekshiruvidan o'tsin
  const content: string;
  export default content;
}
export {};   // `declare global` ishlashi uchun buni modulga aylantiramiz
```

**⚠️ Ehtiyot bo'l:** Top-level `import`/`export` li `.d.ts` **modulga** aylanadi; uning ichidan global scope'ga qo'shish uchun deklaratsiyalarni `declare global { ... }` ga o'rashingiz va `export {}` qo'shishingiz kerak.

---

## Strict mode va strictNullChecks

**💡 Tushuncha:** `"strict": true` strict tekshiruvlar oilasini yoqadi. Eng ta'sirlisi — **`strictNullChecks`**, u `null`/`undefined` ni boshqalarga assignable bo'lmaydigan alohida tiplar qiladi (agar aniq kiritilmasa).

### ❓ `strictNullChecks` nimani o'zgartiradi?

**✅ Javob:** Usiz `null` va `undefined` har bir tipga assignable bo'lib, "cannot read property of undefined" kabi katta xato turkumini yashiradi. U yoqilganda `T | null | undefined` ni aniq yozishingiz va ishlatishdan oldin narrow qilishingiz kerak (optional chaining `?.`, nullish coalescing `??` yoki guard'lar). Bu eng yuqori qiymatli strict flag.

```ts
function greet(name: string | null) {
  // return name.toUpperCase();          ❌ name null bo'lishi mumkin
  return name?.toUpperCase() ?? "notanish"; // ✅
}
```

**⚠️ Ehtiyot bo'l:** `strict` — umbrella flag, u `strictNullChecks`, `noImplicitAny`, `strictFunctionTypes`, `strictBindCallApply`, `strictPropertyInitialization`, `alwaysStrict` va boshqalarni yoqadi. Non-null assertion `name!` tekshiruvni o'chiradi, lekin xavfli — faqat compiler'dan ko'ra ko'proq bilganingizda ishlating.

---

## Type assertion

**💡 Tushuncha:** **Type assertion** (`x as T` yoki `<T>x`) compiler'ga qiymatni berilgan tip sifatida ko'rishni aytadi. Bu **runtime effekti yo'q** compile-vaqt override — konvertatsiya ham, tekshiruv ham yo'q.

### ❓ `as` qachon o'rinli, qachon xavfli?

**✅ Javob:** Assertion'ni compiler'dan ko'ra ko'proq bilganingizda ishlating — masalan DOM query natijasini narrow qilish (`as HTMLInputElement`) yoki tekshirilgan `unknown`. U xavfli, chunki tip xavfsizligini chetlab o'tadi: noto'g'ri assertion runtime'da crash bo'lguncha xato bermaydi. Haqiqiy narrowing/guard afzal; `as` ni faqat haqiqiy bo'shliqlar uchun saqlang.

```ts
const input = document.getElementById("email") as HTMLInputElement;
input.value;                       // ✅ endi tiplangan

const data = JSON.parse(raw) as User;   // ⚠️ tekshirilmagan — JSON istalgan narsa bo'lishi mumkin

// Double assertion (escape hatch) — qattiq tavsiya etilmaydi:
const x = someValue as unknown as TargetType;
```

**⚠️ Ehtiyot bo'l:** TS "imkonsiz" to'g'ridan-to'g'ri assertion'larni taqiqlaydi (masalan `string as number`); odamlar buni `as unknown as T` bilan aylanib o'tadi — bu tiplar noto'g'ri ekanining belgisi. Assertion **cast emas**: runtime'da hech narsa konvertatsiya qilinmaydi. Qiymatni tipga tekshirib, lekin kengaytirmasdan saqlamoqchi bo'lsangiz `satisfies` afzal.

---

## React'da generics

**💡 Tushuncha:** React component prop'lari va hook'lari generics bilan tiplanadi. `useState<T>`, `useRef<T>` va generic component'lar prop hamda state'ni tip-xavfsiz saqlaydi.

### ❓ Tiplangan hook va generic component ko'rsating.

**✅ Javob:**

```tsx
const [count, setCount] = useState<number>(0);      // infer bo'lmaganda aniq beriladi
const [user, setUser] = useState<User | null>(null);// "hali yuklanmagan" uchun union
const inputRef = useRef<HTMLInputElement>(null);

interface ListProps<T> {
  items: T[];
  render: (item: T) => React.ReactNode;
}
function List<T>({ items, render }: ListProps<T>) {
  return <ul>{items.map((it, i) => <li key={i}>{render(it)}</li>)}</ul>;
}
// Foydalanish T'ni items'dan infer qiladi:
<List items={users} render={(u) => u.name} />;       // u User tipida
```

**⚠️ Ehtiyot bo'l:** Eskirgan `React.FC` (u implicit `children` ni majbur qilar va generics'ni murakkablashtirar edi) o'rniga prop'larni to'g'ridan-to'g'ri tiplash afzal (`{ children }: { children: React.ReactNode }`). `.tsx` da arrow-generic `<T>` JSX bilan noaniq — disambiguate qilish uchun `<T,>` yozing.

---

## Muhim compiler option'lar

**💡 Tushuncha:** `tsconfig.json` kompilyatsiyani boshqaradi. Yuqori qiymatli flag'larni biling.

### ❓ Qaysi compiler option'lar eng muhim?

**✅ Javob:**
- **`strict`** — barcha strict tekshiruvlarni yoqadi (har doim tavsiya etiladi).
- **`target`** — emit qilinadigan JS versiyasi (`ES2020`, `ESNext`).
- **`module`** — modul formati (`ESNext`, `CommonJS`, `NodeNext`).
- **`moduleResolution`** — import'lar qanday hal qilinadi (`bundler`, `node16`/`nodenext`).
- **`noImplicitAny`** — infer qilingan `any` da xato.
- **`noUncheckedIndexedAccess`** — index access `T | undefined` qaytaradi (xavfsizroq massiv/record).
- **`esModuleInterop`** — silliq CJS/ESM default-import interop.
- **`skipLibCheck`** — `.d.ts` tip tekshiruvini o'tkazib yuboradi (tezroq build).
- **`jsx`** — zamonaviy avtomatik runtime uchun `react-jsx`.
- **`isolatedModules`** — har bir fayl yolg'iz kompilyatsiya bo'lishini ta'minlaydi (Babel/SWC/esbuild talab qiladi).

```jsonc
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "ESNext",
    "moduleResolution": "bundler",
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "jsx": "react-jsx",
    "isolatedModules": true
  }
}
```

**⚠️ Ehtiyot bo'l:** `noUncheckedIndexedAccess` — kam baholanadigan gavhar — u `arr[i]` yoki `record[key]` `undefined` bo'lishi mumkinligini boshqarishni majbur qiladi va keng tarqalgan runtime crash'ni yo'qotadi. `isolatedModules` `const enum` va `export type` siz type-only re-export'ni taqiqlaydi.

---

## Masalalar

> Yechimlar: [`solutions/frontend/04-typescript.md`](../solutions/frontend/04-typescript.md)

1. **`DeepReadonly<T>`** — obyektni rekursiv (ichma-ich) `readonly` qiluvchi mapped tip yozing.
2. **`MyPick<T, K>`** — `Pick` ni noldan, mapped tip va `K extends keyof T` constraint bilan yozing.
3. **`MyExclude<T, U>` va `MyExtract<T, U>`** — ikkalasini distributive conditional tip bilan yozing.
4. **`MyReturnType<T>`** — funksiya return tipini `infer` orqali ajratuvchi tipni yozing.
5. **`isShape` type guard** — `kind` discriminated union ustida ishlaydigan custom type guard funksiyani yozing va `area` da exhaustiveness (`never`) tekshiruvi qo'shing.
6. **`PartialBy<T, K>`** — `T` ning faqat `K` key'larini ixtiyoriy qiluvchi (qolganlari majburiy) tipni yozing.
7. **`UnwrapPromise<T>`** — ichma-ich joylashgan promise'larni ham (`Promise<Promise<number>>` → `number`) ochadigan rekursiv conditional tip yozing.
8. **`Mutable<T>`** — `readonly` modifier'ni olib tashlaydigan mapped tip yozing va `as const` obyektga qo'llab sinab ko'ring.
9. **`fromEntries` overload** — `parse(x: string): string[]` va `parse(x: number): number` kabi input-bog'liq return tipli funksiya yozing.
10. **`Route` tipini olish** — `as const` massivdan `(typeof arr)[number]` orqali string-literal union tip oling va undan tip-xavfsiz `navigate(route)` funksiya yozing.

---

← [Frontend bo'limiga qaytish](./README.md)
