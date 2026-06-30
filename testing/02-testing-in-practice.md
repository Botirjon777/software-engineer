# Amaliyotda Testing (JS/TS)

Bu fayl test yozishni nazariyadan amaliyotga ko'chiradi: test runner tanlash (Jest vs Vitest), test struktura (`describe`/`it`/`test`, `expect` va matcher'lar, `beforeEach`/`afterEach`), assertion'larni to'g'ri tanlash (`toBe` vs `toEqual`, `toThrow`, async matcher'lar), async kodni test qilish (`resolves`/`rejects`), mocking'ning amaliy turlari (`jest.fn`, `jest.mock`, `spyOn`, module/timer/fetch mock), snapshot testing va uning tanqidi, React Testing Library falsafasi ("test like a user", `getBy`/`findBy`/`queryBy`, `userEvent`), e2e (Playwright/Cypress qisqacha), CI'da test, parametrized test va DI bilan test-friendly kod. Maqsad: intervyuda "qanday test yozasiz?" degan amaliy savolga ishonchli, real kod bilan javob bera olish. Misollar JS/TS (Jest/Vitest).

## Mundarija

- [Tushuncha: test runner nima](#tushuncha-test-runner-nima)
- [Jest vs Vitest](#jest-vs-vitest)
- [Test struktura: describe / it / test](#test-struktura-describe--it--test)
- [Hooks: beforeEach / afterEach / beforeAll / afterAll](#hooks-beforeeach--aftereach--beforeall--afterall)
- [Assertion va matcher'lar: toBe vs toEqual](#assertion-va-matcherlar-tobe-vs-toequal)
- [Xatoni test qilish: toThrow](#xatoni-test-qilish-tothrow)
- [Async kodni test qilish: resolves / rejects](#async-kodni-test-qilish-resolves--rejects)
- [Mocking: jest.fn, jest.mock, spyOn](#mocking-jestfn-jestmock-spyon)
- [Timer va fetch mock](#timer-va-fetch-mock)
- [Snapshot testing va uning tanqidi](#snapshot-testing-va-uning-tanqidi)
- [React Testing Library: test like a user](#react-testing-library-test-like-a-user)
- [getBy / findBy / queryBy va userEvent](#getby--findby--queryby-va-userevent)
- [E2e: Playwright va Cypress qisqacha](#e2e-playwright-va-cypress-qisqacha)
- [CI'da test](#cida-test)
- [Parametrized test (test.each)](#parametrized-test-testeach)
- [DI bilan test-friendly kod](#di-bilan-test-friendly-kod)
- [Intervyu savollari (Q&A)](#intervyu-savollari-qa)
- [Masalalar](#masalalar)

---

## Tushuncha: test runner nima

**💡 Tushuncha:** **Test runner** — bu test fayllarini topadigan, ularni ishga tushiradigan, har bir `expect` natijasini tekshiradigan va oxirida "nechta pass, nechta fail" hisobotini chiqaradigan dastur. U sizga uchta narsani beradi: test e'lon qilish API'si (`describe`, `test`), assertion API'si (`expect` va matcher'lar), va orkestratsiya (parallel ishga tushirish, watch mode, coverage hisoboti).

JS dunyosida ikkita asosiy runner bor: **Jest** (Meta tomonidan, ko'p yillik standart) va **Vitest** (Vite ekotizimi uchun, tez va zamonaviy). Ikkalasining API'si deyarli bir xil — `describe`/`it`/`expect` bir xil ishlaydi, shuning uchun birini bilsangiz, ikkinchisi oson o'tadi.

```js
// Eng oddiy test fayli (math.test.js)
import { add } from "./math.js";

test("add ikki sonni qo'shadi", () => {
  expect(add(2, 3)).toBe(5);
});
```

Runner odatda fayl nomi naqshi (`*.test.js`, `*.spec.ts`) bo'yicha test fayllarini avtomatik topadi. Siz faqat `npm test` deysiz — qolganini runner qiladi.

**⚠️ Ehtiyot bo'l:** Test fayllari production bundle'ga kirmasligi kerak. Build konfiguratsiyangiz `*.test.*` fayllarni chiqarib tashlashiga ishonch hosil qiling, aks holda test kodi va mock'lar prodga sizib chiqadi.

---

## Jest vs Vitest

**💡 Tushuncha:** **Jest** va **Vitest** — eng mashhur ikki JS test runner. Jest yetuk, juda keng ekotizimga ega va CRA/Babel asosidagi loyihalarda standart. Vitest esa Vite asosida qurilgan: u native ESM va TypeScript bilan to'g'ridan-to'g'ri ishlaydi, juda tez (HMR'ga o'xshash watch mode), va konfiguratsiyasi Vite bilan birga keladi.

| Xususiyat | Jest | Vitest |
|-----------|------|--------|
| Tezlik | Tez | Juda tez (Vite transform) |
| ESM / TS | Konfiguratsiya/transform kerak | Native qo'llab-quvvatlash |
| API | `jest.fn`, `jest.mock` | `vi.fn`, `vi.mock` (deyarli bir xil) |
| Vite loyihalar | Qo'shimcha sozlash | Tabiiy mos |
| Ekotizim | Eng katta | O'sib boryapti |

```ts
// Jest
import { jest } from "@jest/globals";
const fn = jest.fn();

// Vitest — bir xil g'oya, faqat nom boshqacha
import { vi } from "vitest";
const fn2 = vi.fn();
```

Amaliy tavsiya: Vite (yoki yangi loyiha) ishlatayotgan bo'lsangiz — Vitest. Eski/CRA/Next eski versiya yoki katta Jest ekotizimiga bog'liq loyiha bo'lsa — Jest. Migratsiya odatda oson, chunki API mos.

**⚠️ Ehtiyot bo'l:** API o'xshash bo'lsa ham, mock'larni avtomatik reset qilish standartlari farq qiladi. Vitest `config.test.clearMocks`/`restoreMocks` ni aniq yoqishni talab qilishi mumkin; Jest'da ham buni `clearMocks: true` bilan yoqing — aks holda mock holati testlar orasida sizadi.

---

## Test struktura: describe / it / test

**💡 Tushuncha:** Testlar uch darajali tuzilmaga ega: **`describe`** bloki bog'liq testlarni guruhlaydi (odatda bitta funksiya/komponent atrofida), **`it`** yoki **`test`** bitta aniq holatni tekshiradi, ichida **`expect`** assertion'lar bo'ladi. `it` va `test` — bir xil narsa; `it('should ...')` BDD uslubida jumla hosil qiladi.

```ts
describe("applyDiscount", () => {
  it("normal foizni qo'llaydi", () => {
    expect(applyDiscount(100, 20)).toBe(80);
  });

  it("0% chegarasi narxni o'zgartirmaydi", () => {
    expect(applyDiscount(100, 0)).toBe(100);
  });

  it("noto'g'ri foizda xato tashlaydi", () => {
    expect(() => applyDiscount(100, 150)).toThrow();
  });
});
```

`describe` ni ichma-ich (nested) yozish mumkin — masalan, "auth qilingan foydalanuvchi" va "auth qilinmagan" guruhlarini ajratish uchun. Test nomi to'liq jumla bo'lishi kerak: fail bo'lganda nom o'zi muammoni tushuntirsin (`"throws if amount is negative"`, `"empty string return qiladi"` emas).

**⚠️ Ehtiyot bo'l:** `it.only` / `test.only` va `it.skip` ni kommitlab yuborib qo'ymang — `.only` qolgan barcha testlarni "o'chirib" qo'yadi va CI yashil ko'rinadi-yu, aslida deyarli hech narsa sinalmaydi. ESLint qoidasi (`no-focused-tests`) bilan buni bloklang.

---

## Hooks: beforeEach / afterEach / beforeAll / afterAll

**💡 Tushuncha:** **Lifecycle hooks** testlarning takrorlanuvchi tayyorgarlik/tozalash kodini ajratib oladi. **`beforeEach`** har testdan *oldin*, **`afterEach`** har testdan *keyin* ishlaydi — odatda holatni reset qilish uchun. **`beforeAll`**/**`afterAll`** esa butun `describe` uchun *bir marta* (qimmat sozlash — DB ulash — uchun).

```ts
describe("Cart", () => {
  let cart: Cart;

  beforeEach(() => {
    cart = new Cart(); // har test toza obyekt oladi
  });

  afterEach(() => {
    jest.clearAllMocks(); // mock holatini tozalash
  });

  it("mahsulot qo'shadi", () => {
    cart.add({ id: 1, price: 100 });
    expect(cart.total()).toBe(100);
  });

  it("bo'sh savat 0 qaytaradi", () => {
    expect(cart.total()).toBe(0);
  });
});
```

`beforeEach` da yangi obyekt yaratish — FIRST'ning "Independent" printsipini ta'minlaydi: har test toza holatdan boshlaydi va tartibga bog'liq bo'lmaydi.

**⚠️ Ehtiyot bo'l:** Holatni `beforeAll` da bir marta yaratib, testlar orasida bo'lishish — eng keng tarqalgan flaky'lik sababi. Bir test obyektni o'zgartirsa (mutate), keyingisi buzilgan holatni oladi. O'zgaruvchan holat uchun har doim `beforeEach` ishlating.

---

## Assertion va matcher'lar: toBe vs toEqual

**💡 Tushuncha:** **Matcher** — `expect(...)` dan keyin keladigan tekshiruv usuli (`.toBe`, `.toEqual`, `.toContain` ...). Eng muhim farq: **`toBe`** identiklikni (`Object.is`, ya'ni `===` ga yaqin) tekshiradi — primitivlar va bir xil reference uchun. **`toEqual`** esa obyekt/massivni *chuqur* (deep) — qiymat-qiymat — solishtiradi.

```ts
// Primitivlar — toBe to'g'ri
expect(2 + 2).toBe(4);
expect("ali").toBe("ali");

// Obyektlar — toBe YIQILADI (har xil reference), toEqual ISHLAYDI
expect({ a: 1 }).toBe({ a: 1 });    // ❌ fail: boshqa obyekt
expect({ a: 1 }).toEqual({ a: 1 }); // ✅ pass: qiymatlar mos

// Foydali matcher'lar
expect([1, 2, 3]).toContain(2);
expect("hello world").toMatch(/world/);
expect(value).toBeNull();
expect(value).toBeUndefined();
expect(value).toBeTruthy();
expect(arr).toHaveLength(3);
expect(obj).toHaveProperty("name", "Ali");
expect(0.1 + 0.2).toBeCloseTo(0.3); // float uchun!
```

`toStrictEqual` — `toEqual`'dan qattiqroq: `undefined` qiymatli kalitlarni va tur (class) farqini ham hisobga oladi.

**⚠️ Ehtiyot bo'l:** Float'larni `toBe` bilan solishtirmang: `expect(0.1 + 0.2).toBe(0.3)` yiqiladi, chunki `0.1 + 0.2 === 0.30000000000000004`. Suzuvchi nuqta uchun har doim `toBeCloseTo` ishlating.

---

## Xatoni test qilish: toThrow

**💡 Tushuncha:** Funksiya xato tashlashini tekshirish uchun chaqiruvni **funksiya ichiga o'rab** `toThrow` ishlatiladi. Muhim nuqta: `expect(fn())` emas, `expect(() => fn())` — aks holda xato `expect` ga yetmasdan testni o'zini yiqitadi.

```ts
function withdraw(balance: number, amount: number) {
  if (amount > balance) throw new Error("Yetarli mablag' yo'q");
  return balance - amount;
}

// To'g'ri: chaqiruvni callback ichiga o'rab
expect(() => withdraw(100, 200)).toThrow();
expect(() => withdraw(100, 200)).toThrow("Yetarli mablag' yo'q"); // xabar mos
expect(() => withdraw(100, 200)).toThrow(/mablag'/);              // regex
expect(() => withdraw(100, 200)).toThrow(Error);                  // tur

// NOTO'G'RI: xato darrov otiladi, expect chaqirilmaydi
expect(withdraw(100, 200)).toThrow(); // ❌ test crash bo'ladi
```

Async funksiyaning xatosini tekshirish uchun `rejects.toThrow` ishlatiladi (pastdagi async bo'limga qarang).

**⚠️ Ehtiyot bo'l:** `toThrow()` ni argumentsiz ishlatish "biror xato tashladi" deydi, lekin *qaysi* xato ekanini tekshirmaydi. Agar funksiya boshqa (kutilmagan) sababdan xato tashlasa, test baribir pass bo'ladi. Xato xabari yoki turini ham bering.

---

## Async kodni test qilish: resolves / rejects

**💡 Tushuncha:** Async kodni test qilishda asosiy qoida — **promise tugashini kutish**. Aks holda test promise hal bo'lmasdan tugaydi va assertion umuman ishlamaydi (false pass). Ikki uslub bor: `async/await` bilan to'g'ridan-to'g'ri, yoki `expect(promise).resolves`/`.rejects` matcher'lari.

```ts
// 1-uslub: async/await (eng aniq)
it("foydalanuvchini yuklaydi", async () => {
  const user = await fetchUser(1);
  expect(user.name).toBe("Ali");
});

// 2-uslub: resolves / rejects matcher'lari
it("promise hal bo'ladi", async () => {
  await expect(fetchUser(1)).resolves.toEqual({ id: 1, name: "Ali" });
});

it("xato bilan rejected bo'ladi", async () => {
  await expect(fetchUser(-1)).rejects.toThrow("Noto'g'ri id");
});
```

E'tibor bering: `resolves`/`rejects` oldida ham `await` (yoki `return`) bo'lishi shart — bu runner'ga "men kutyapman" deb signal beradi.

**⚠️ Ehtiyot bo'l:** Eng xavfli xato — `await`ni unutish. `expect(fetchUser(-1)).rejects.toThrow()` (await'siz) hech qachon yiqilmaydi, chunki assertion promise hal bo'lishidan oldin test tugaydi. `eslint-plugin-jest` ning `valid-expect` / `no-floating-promises` qoidalari buni ushlaydi.

---

## Mocking: jest.fn, jest.mock, spyOn

**💡 Tushuncha:** Jest'da uchta asosiy mocking quroli bor. **`jest.fn()`** — soxta funksiya yaratadi (chaqiruvlarni yozadi, qaytar qiymatni boshqarish mumkin). **`jest.mock('module')`** — butun modulni avtomatik soxta versiya bilan almashtiradi. **`jest.spyOn(obj, 'method')`** — mavjud obyektning metodini kuzatadi (va xohlasa o'rnini bosadi), keyin `mockRestore` bilan tiklaydi.

```ts
// jest.fn() — soxta funksiya, javobni boshqaramiz
const getRate = jest.fn().mockReturnValue(12500);
const getRateAsync = jest.fn().mockResolvedValue(12500);

expect(getRate()).toBe(12500);
expect(getRate).toHaveBeenCalledTimes(1);
expect(getRate).toHaveBeenCalledWith();

// jest.mock() — butun modulni mock qilish
jest.mock("./mailer");
import { sendEmail } from "./mailer";
// endi sendEmail — avtomatik mock funksiya
(sendEmail as jest.Mock).mockResolvedValue(true);

// jest.spyOn() — mavjud metodni kuzatish
const spy = jest.spyOn(console, "warn").mockImplementation(() => {});
doRiskyThing();
expect(spy).toHaveBeenCalled();
spy.mockRestore(); // asl metodni tiklash
```

Foydali assertion'lar: `toHaveBeenCalled`, `toHaveBeenCalledTimes(n)`, `toHaveBeenCalledWith(args)`, `toHaveBeenLastCalledWith(args)`. `mockReturnValue` (sinxron), `mockResolvedValue` (promise), `mockImplementation` (to'liq xulq).

**⚠️ Ehtiyot bo'l:** Mock holati testlar orasida saqlanadi. Har testdan keyin `jest.clearAllMocks()` (chaqiruv tarixini tozalaydi) yoki `jest.restoreAllMocks()` (asl implementatsiyani tiklaydi) ni `afterEach` da chaqiring — yoki config'da `clearMocks: true`. Aks holda "necha marta chaqirildi" hisoblari testlar orasida qo'shilib ketadi.

---

## Timer va fetch mock

**💡 Tushuncha:** Vaqt va network — flaky'likning ikki asosiy manbasi. Ularni mock qilib *deterministik* qilamiz. **Fake timers** (`jest.useFakeTimers`) — `setTimeout`/`setInterval`/`Date` ni boshqarib, vaqtni qo'lda "oldinga suramiz" (`jest.advanceTimersByTime`). **Fetch mock** — `global.fetch` ni soxta qilib, real network so'rovsiz javob qaytaramiz.

```ts
// Fake timers — vaqtni boshqarish
jest.useFakeTimers();

it("debounce 300ms dan keyin chaqiradi", () => {
  const cb = jest.fn();
  const debounced = debounce(cb, 300);

  debounced();
  expect(cb).not.toHaveBeenCalled(); // hali emas

  jest.advanceTimersByTime(300);     // vaqtni "oldinga surdik"
  expect(cb).toHaveBeenCalledTimes(1);
});

afterEach(() => jest.useRealTimers());

// Fetch mock — real network'siz
beforeEach(() => {
  global.fetch = jest.fn().mockResolvedValue({
    ok: true,
    json: async () => ({ id: 1, name: "Ali" }),
  } as Response);
});

it("foydalanuvchini fetch qiladi", async () => {
  const user = await getUser(1);
  expect(fetch).toHaveBeenCalledWith("/api/users/1");
  expect(user.name).toBe("Ali");
});
```

Sistema vaqtini muzlatish uchun `jest.setSystemTime(new Date("2026-01-01"))`. Real loyihalarda `fetch` o'rniga ko'pincha **MSW (Mock Service Worker)** ishlatiladi — u network qatlamida intercept qilib, mock'ni real'ga yaqinlashtiradi.

**⚠️ Ehtiyot bo'l:** Fake timer'larni yoqib, real timer'ga qaytarmasangiz (`useRealTimers`), keyingi testlarda `setTimeout` "muzlab" qoladi va kutilmagan tarzda osilib qoladi. Har doim `afterEach` da tiklang.

---

## Snapshot testing va uning tanqidi

**💡 Tushuncha:** **Snapshot test** — natijani (odatda render qilingan UI yoki katta obyekt) birinchi marta faylga "suratga oladi" (`.snap`), keyingi run'larda esa yangi natijani saqlangan surat bilan solishtiradi. Farq bo'lsa — test yiqiladi. Maqsad: katta strukturalardagi *kutilmagan* o'zgarishlarni ushlash.

```tsx
it("Button to'g'ri render qiladi", () => {
  const { container } = render(<Button label="Saqlash" />);
  expect(container).toMatchSnapshot();
});

// Inline snapshot — surat to'g'ridan-to'g'ri test ichida
expect(formatUser(user)).toMatchInlineSnapshot(`"Ali (ali@example.uz)"`);
```

Snapshot kuchli, lekin xavfli quroli. **Tanqid:** (1) ular *nimani* tasdiqlashini ko'rsatmaydi — faqat "o'zgarmadi" deydi, "to'g'ri" demaydi. (2) Dasturchilar fail bo'lganda o'ylamasdan `--updateSnapshot` bosadi — bug snapshot'ga "muhrlanadi". (3) Katta snapshot'lar review'da o'qilmaydi, diff shovqinli bo'ladi.

**⚠️ Ehtiyot bo'l:** Snapshot'ni "lazy assertion" sifatida ishlatmang. Aniq xulq uchun aniq matcher (`toBe`, `getByText`) yozing; snapshot'ni faqat kichik, barqaror, ataylab kuzatilayotgan natijalar uchun va inline shaklda ishlating. `--updateSnapshot` ni o'ylab bosing.

---

## React Testing Library: test like a user

**💡 Tushuncha:** **React Testing Library (RTL)** falsafasi bitta jumlada: **"test like a user"** — foydalanuvchi kabi test qil. Ya'ni komponentni *ichki holati* (state, props, instance) orqali emas, *ekrandagi ko'rinishi va o'zaro ta'siri* orqali sinaysiz. Foydalanuvchi `state.count` ni ko'rmaydi — u "3" yozuvini ko'radi va tugmani bosadi. Test ham xuddi shunday qilishi kerak.

```tsx
import { render, screen } from "@testing-library/react";
import userEvent from "@testing-library/user-event";

it("hisoblagich tugma bosilganda oshadi", async () => {
  render(<Counter />);

  // Foydalanuvchi ko'rgan narsani qidiramiz — role/text orqali
  expect(screen.getByText("Hisob: 0")).toBeInTheDocument();

  await userEvent.click(screen.getByRole("button", { name: /oshirish/i }));

  expect(screen.getByText("Hisob: 1")).toBeInTheDocument();
});
```

Bu yondashuv testni implementatsiyadan ajratadi: `useState` ni `useReducer` ga o'zgartirsangiz ham, ekrandagi xulq o'zgarmasa, test yashil qoladi. Bu aynan "refactoring ishonchi" — testning asosiy foydasi.

**⚠️ Ehtiyot bo'l:** Element'ni `container.querySelector('.btn-primary')` yoki test-id bilan qidirish "implementatsiyaga bog'lanish"dir. Imkon qadar `getByRole`/`getByLabelText`/`getByText` (foydalanuvchi va accessibility ko'radigan narsa) ishlating; `data-testid` faqat oxirgi chora.

---

## getBy / findBy / queryBy va userEvent

**💡 Tushuncha:** RTL'da element topishning uchta oilasi bor va ular qachon ishlatilishi muhim. **`getBy`** — element *bo'lishi shart*; topilmasa darrov xato. **`queryBy`** — element *bo'lmasligi* mumkin; topilmasa `null` qaytaradi (yo'qligini tekshirish uchun). **`findBy`** — element *keyinroq* (async) paydo bo'ladi; promise qaytaradi, `await` bilan kutiladi.

```tsx
// getBy — hozir mavjud bo'lishi kerak
expect(screen.getByText("Salom")).toBeInTheDocument();

// queryBy — yo'qligini tasdiqlash (getBy bu yerda xato tashlardi)
expect(screen.queryByText("Xato")).not.toBeInTheDocument();

// findBy — async paydo bo'lishini kutish (fetch'dan keyin)
const user = await screen.findByText("Ali yuklandi");
expect(user).toBeInTheDocument();
```

**`userEvent`** — `fireEvent`'dan afzal: u haqiqiy foydalanuvchi xatti-harakatini taqlid qiladi (klik oldidan hover, fokus, klaviatura ketma-ketligi). Zamonaviy versiyada `userEvent` async — `await userEvent.click(...)`, `await userEvent.type(input, 'matn')`.

| Holat | Metod |
|-------|-------|
| Element bor (sinxron) | `getBy*` |
| Element yo'qligini tekshirish | `queryBy*` |
| Element async paydo bo'ladi | `findBy*` (await) |
| Bir nechta element | `getAllBy*` / `queryAllBy*` / `findAllBy*` |

**⚠️ Ehtiyot bo'l:** Yo'qlikni tekshirishda `getBy` ishlatmang — u topilmasa *test yiqiladi*, sizning `not.toBeInTheDocument` ga yetmaydi ham. Yo'qlik uchun har doim `queryBy`. Async natija uchun esa `getBy` emas, `findBy` (yoki `waitFor`).

---

## E2e: Playwright va Cypress qisqacha

**💡 Tushuncha:** **Playwright** va **Cypress** — brauzer asosidagi e2e (end-to-end) test asboblari. Ular real brauzer ochib, foydalanuvchi kabi sahifa bilan ishlaydi: sahifaga o'tadi, tugma bosadi, forma to'ldiradi, natijani ekrandan o'qiydi. Farqi: Cypress brauzer ichida ishlaydi (developer-friendly UI), Playwright tashqaridan boshqaradi (ko'p brauzer, parallel, tez).

```ts
// Playwright
import { test, expect } from "@playwright/test";

test("foydalanuvchi tizimga kiradi", async ({ page }) => {
  await page.goto("/login");
  await page.getByLabel("Email").fill("ali@example.uz");
  await page.getByLabel("Parol").fill("secret123");
  await page.getByRole("button", { name: "Kirish" }).click();

  await expect(page.getByText("Xush kelibsiz, Ali")).toBeVisible();
});
```

```js
// Cypress — o'xshash, lekin zanjir (chaining) uslubi
it("login ishlaydi", () => {
  cy.visit("/login");
  cy.get('[name="email"]').type("ali@example.uz");
  cy.get('[name="password"]').type("secret123");
  cy.contains("Kirish").click();
  cy.contains("Xush kelibsiz, Ali").should("be.visible");
});
```

Ikkalasi ham **auto-waiting** qiladi — element paydo bo'lishini avtomatik kutadi, qattiq `sleep` shart emas. Bu flaky'likni kamaytiradi. Tanlov: ko'p brauzer/CI tezligi kerak bo'lsa Playwright, debugging tajribasi muhim bo'lsa Cypress.

**⚠️ Ehtiyot bo'l:** E2e testni ko'p yozmang — pyramid'ning eng tepasi. Har bir validatsiya qoidasini e2e bilan emas, unit bilan tekshiring. E2e faqat eng muhim "happy path" oqimlari (login, checkout, ro'yxatdan o'tish) uchun.

---

## CI'da test

**💡 Tushuncha:** **CI (Continuous Integration)** — har push/PR'da testlarni avtomatik ishga tushiradigan tizim (GitHub Actions, GitLab CI ...). Maqsad: buzilgan kod merge bo'lmasligi. Testlar yashil bo'lmasa, PR bloklanadi. CI'da testlar **CI mode** da ishlaydi: watch yo'q, parallel, coverage hisoboti, va flaky'likka nol tolerantlik.

```yaml
# .github/workflows/test.yml
name: Test
on: [push, pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: npm
      - run: npm ci
      - run: npm test -- --coverage --ci
      - run: npm run test:e2e   # alohida, sekinroq qadam
```

Amaliy nuqtalar: `npm ci` (deterministik o'rnatish, `npm install` emas), `--ci` flag (snapshot avtomatik yangilanmaydi — yangi snapshot fail bo'ladi), unit va e2e ni alohida job'larga ajratish (e2e sekin), va coverage chegarasini (`coverageThreshold`) belgilash.

**⚠️ Ehtiyot bo'l:** CI'da `--updateSnapshot` ni hech qachon yoqmang — bu har qanday o'zgarishni "to'g'ri" deb qabul qiladi va snapshot testlarini foydasiz qiladi. CI faqat *tekshiradi*, snapshot'ni *yangilamaydi*. Yangilash faqat lokal, ataylab qilinadi.

---

## Parametrized test (test.each)

**💡 Tushuncha:** **Parametrized (data-driven) test** — bitta test mantig'ini ko'p kirish/natija juftligi ustida takrorlash. `test.each` (yoki `it.each`) jadval beradi, har qator alohida test bo'lib ishlaydi. Bu copy-paste o'rniga yagona, o'qiladigan test beradi — va fail bo'lganda *qaysi qator* yiqilganini aniq ko'rsatadi.

```ts
describe("isEven", () => {
  test.each([
    [2, true],
    [3, false],
    [0, true],
    [-4, true],
    [-5, false],
  ])("isEven(%i) === %s", (input, expected) => {
    expect(isEven(input)).toBe(expected);
  });
});

// Obyekt shakli — o'qishga qulayroq
test.each([
  { a: 1, b: 2, sum: 3 },
  { a: -1, b: 1, sum: 0 },
  { a: 0, b: 0, sum: 0 },
])("add($a, $b) === $sum", ({ a, b, sum }) => {
  expect(add(a, b)).toBe(sum);
});
```

`%i`, `%s`, `%o` — formatlash belgilari (integer, string, object); obyekt shaklida `$a`, `$sum` — maydon nomlari. Bu edge case'larni bir joyda jamlash uchun ideal.

**⚠️ Ehtiyot bo'l:** `test.each` ni murakkab mantiq (har qator uchun boshqacha setup) uchun cho'zib yubormang — bu testni o'qib bo'lmaydigan qiladi. U faqat *bir xil mantiq, har xil ma'lumot* uchun. Setup farq qilsa, alohida testlar yozing.

---

## DI bilan test-friendly kod

**💡 Tushuncha:** **Dependency Injection (DI)** — bog'liqlikni kod ichida *yaratish* o'rniga, tashqaridan (parametr orqali) *uzatish*. Bu test uchun hal qiluvchi: bog'liqlik tashqaridan kelsa, testda uni mock/fake bilan almashtirish oson. DI'siz kod (ichida `new Database()` yoki `import` qilingan global) ni test qilish uchun murakkab modul mock kerak bo'ladi.

```ts
// YOMON — bog'liqlik ichida "qattiq sim" qilingan (test qiyin)
class OrderService {
  async place(order: Order) {
    const db = new PostgresDb();        // qattiq bog'liqlik
    await db.save(order);
    await sendEmail(order.email);       // import qilingan global
  }
}

// YAXSHI — bog'liqlik inject qilingan (test oson)
class OrderService {
  constructor(
    private db: Database,
    private mailer: Mailer,
  ) {}

  async place(order: Order) {
    await this.db.save(order);
    await this.mailer.send(order.email);
  }
}

// Test — fake/mock ni bemalol uzatamiz
it("buyurtmani saqlaydi va email yuboradi", async () => {
  const db = { save: jest.fn() };
  const mailer = { send: jest.fn() };
  const service = new OrderService(db as any, mailer as any);

  await service.place({ id: 1, email: "ali@example.uz" } as Order);

  expect(db.save).toHaveBeenCalledTimes(1);
  expect(mailer.send).toHaveBeenCalledWith("ali@example.uz");
});
```

DI faqat klass uchun emas — funksiyaga bog'liqlikni parametr sifatida uzatish ham (`function place(order, db, mailer)`) DI'ning oddiy shakli. Asosiy g'oya: "kod o'z bog'liqligini o'zi tanlamasin, unga berilsin".

**⚠️ Ehtiyot bo'l:** DI'ni haddan oshirib, hamma narsani interfeys va konteyner orqali abstraktlash kichik loyihada ortiqcha murakkablik (over-engineering). Faqat *tashqi, almashtiriladigan* bog'liqliklarni (DB, network, vaqt, mailer) inject qiling; sof yordamchi funksiyalarni emas.

---

## Intervyu savollari (Q&A)

### ❓ Jest va Vitest farqi nimada, qaysi birini tanlaysiz?

**✅ Javob:** API'lari deyarli bir xil (`describe`/`it`/`expect`). Vitest Vite asosida, native ESM/TS bilan ishlaydi va juda tez; yangi yoki Vite loyihalar uchun tabiiy tanlov. Jest yetuk, eng katta ekotizimga ega; eski/CRA/katta Jest infrastrukturasiga bog'liq loyihalarda qulay. Migratsiya odatda oson, chunki API mos keladi.

### ❓ `it` va `test` orasida farq bormi?

**✅ Javob:** Yo'q, ular alias — bir xil ishlaydi. `it('should ...')` BDD uslubida o'qiladigan jumla hosil qiladi ("it should ..."), `test('...')` esa neytralroq yangraydi. Tanlov uslubiy; loyiha ichida bittasiga rioya qiling.

### ❓ `toBe` va `toEqual` qachon ishlatiladi?

**✅ Javob:** `toBe` — identiklik (`Object.is`, `===` ga yaqin): primitivlar (son, string, boolean) va bir xil reference uchun. `toEqual` — chuqur (deep) qiymat solishtirish: obyekt va massivlar uchun. Obyektni `toBe` bilan solishtirsangiz, har xil reference bo'lgani uchun yiqiladi — `toEqual` ishlating. Qattiqroq tekshiruv kerak bo'lsa `toStrictEqual`.

### ❓ Float qiymatlarni qanday solishtirasiz?

**✅ Javob:** `toBe` bilan emas — `toBeCloseTo` bilan. `0.1 + 0.2 === 0.30000000000000004` bo'lgani uchun `expect(0.1 + 0.2).toBe(0.3)` yiqiladi. Suzuvchi nuqta arifmetikasi aniq emas, shuning uchun yaqinlik matcher'i kerak.

### ❓ Funksiya xato tashlashini qanday test qilasiz?

**✅ Javob:** Chaqiruvni callback ichiga o'rab `toThrow` ishlataman: `expect(() => fn()).toThrow('xabar')`. To'g'ridan-to'g'ri `expect(fn())` yozsam, xato `expect` ga yetmasdan testni yiqitadi. Xabar yoki turini ham beraman, aks holda "biror xato bo'ldi" deydi-yu, *qaysi* ekanini tekshirmaydi. Async uchun `await expect(p).rejects.toThrow()`.

### ❓ Async kodni test qilishda eng ko'p uchraydigan xato nima?

**✅ Javob:** Promise tugashini kutmaslik — `await` (yoki `return`) unutish. Bunda test promise hal bo'lishidan oldin tugaydi, assertion umuman ishlamaydi va test noto'g'ri "pass" bo'ladi. Har doim `await fetchUser()` yoki `await expect(p).resolves/rejects...` yozaman.

### ❓ `jest.fn`, `jest.mock`, `jest.spyOn` farqi?

**✅ Javob:** `jest.fn()` — yangi soxta funksiya yaratadi (qaytar qiymatni boshqarasiz, chaqiruvni kuzatasiz). `jest.mock('module')` — butun modulni avtomatik mock bilan almashtiradi (tashqi bog'liqliklar uchun). `jest.spyOn(obj, 'method')` — mavjud metodni kuzatadi/o'rnini bosadi, keyin `mockRestore` bilan asl holiga qaytaradi. spyOn — asl kodga "minimal aralashuv" kerak bo'lganda yaxshi.

### ❓ Mock'lar testlar orasida holatni sizdirsa nima qilasiz?

**✅ Javob:** `afterEach` da `jest.clearAllMocks()` (chaqiruv tarixini tozalaydi) yoki `jest.restoreAllMocks()` (asl implementatsiyani tiklaydi) chaqiraman, yoki config'da `clearMocks: true` yoqaman. Aks holda "necha marta chaqirildi" hisobi testlar orasida qo'shilib, noto'g'ri natija beradi.

### ❓ Fake timer'lar nima uchun kerak?

**✅ Javob:** `setTimeout`/`setInterval`/`Date` ga bog'liq kodni deterministik test qilish uchun. Real vaqt kutish o'rniga `jest.useFakeTimers()` yoqib, `jest.advanceTimersByTime(300)` bilan vaqtni qo'lda "oldinga suraman". Bu debounce/throttle/timeout mantig'ini soniyalar kutmasdan, bir zumda va barqaror sinaydi. Oxirida `useRealTimers` bilan tiklayman.

### ❓ Snapshot testing'ning kamchiliklari nimada?

**✅ Javob:** (1) *Nimani* tasdiqlashini ko'rsatmaydi — faqat "o'zgarmadi" deydi, "to'g'ri" demaydi. (2) Fail bo'lganda dasturchilar o'ylamasdan snapshot'ni yangilab, bug'ni muhrlab qo'yadi. (3) Katta snapshot review'da o'qilmaydi. Shuning uchun snapshot'ni faqat kichik, barqaror natijalar uchun (afzal inline) ishlataman; asosiy xulqni aniq matcher bilan tekshiraman.

### ❓ React Testing Library falsafasini tushuntiring.

**✅ Javob:** "Test like a user" — komponentni ichki holati (state, props, instance) orqali emas, foydalanuvchi ko'rgan va qilgan narsa orqali sinaysiz. `getByRole`/`getByText` bilan ekrandagi elementni topib, `userEvent` bilan bosib/yozasiz, natijani ekrandan tekshirasiz. Bu testni implementatsiyadan ajratadi — refactoring qilsangiz ham, ko'rinadigan xulq o'zgarmasa, test yashil qoladi.

### ❓ `getBy`, `queryBy`, `findBy` qachon ishlatiladi?

**✅ Javob:** `getBy` — element hozir bo'lishi shart (yo'q bo'lsa xato). `queryBy` — element *yo'qligini* tekshirish uchun (yo'q bo'lsa `null`, xato emas). `findBy` — element async paydo bo'lishini kutish uchun (promise, `await`). Yo'qlikni `getBy` bilan tekshirish xato (u darrov yiqiladi); async natijani `getBy` bilan olish ham xato (hali render bo'lmagan).

### ❓ `fireEvent` va `userEvent` farqi?

**✅ Javob:** `fireEvent` bitta DOM hodisasini to'g'ridan-to'g'ri otadi. `userEvent` haqiqiy foydalanuvchi xatti-harakatini taqlid qiladi: klik oldidan hover/fokus, yozishda har bir tugma bosilishi. Shu sababli `userEvent` real'roq va afzal. Zamonaviy versiyasi async — `await userEvent.click(...)`.

### ❓ Unit va e2e testni CI'da qanday tashkil qilasiz?

**✅ Javob:** Ularni alohida job/qadamga ajrataman, chunki e2e ancha sekin. Unit har push/PR'da, e2e ham (lekin parallel/keyinroq) ishlaydi. `npm ci` (deterministik), `--ci` flag (snapshot avtomatik yangilanmasin), coverage chegarasi. E2e fail-fast bo'lmasligi va flaky bo'lmasligiga alohida e'tibor — flaky e2e CI'ga ishonchni yo'qotadi.

### ❓ `test.each` qachon ishlatiladi?

**✅ Javob:** Bir xil test mantig'ini ko'p kirish/natija juftligi ustida takrorlash kerak bo'lganda (edge case'lar jadvali). Copy-paste o'rniga yagona, o'qiladigan test beradi va fail bo'lganda qaysi qator yiqilganini ko'rsatadi. Faqat *bir xil mantiq, har xil ma'lumot* uchun; har qatorda boshqacha setup kerak bo'lsa, alohida testlar yozaman.

### ❓ DI kodni qanday test-friendly qiladi?

**✅ Javob:** Bog'liqlik (DB, mailer, vaqt) kod ichida `new` qilinish o'rniga tashqaridan (konstruktor/parametr orqali) uzatilsa, testda uni mock/fake bilan almashtirish oson bo'ladi — modul mock'ning murakkabligi kerak emas. Faqat tashqi, almashtiriladigan bog'liqliklarni inject qilaman; sof funksiyalarni emas (over-engineering bo'lib ketmasligi uchun).

### ❓ `beforeEach` va `beforeAll` ni qachon ishlatasiz?

**✅ Javob:** `beforeEach` — har testdan oldin toza holat yaratish uchun (yangi obyekt, mock reset) — bu Independence'ni ta'minlaydi. `beforeAll` — butun blok uchun bir marta qilinadigan qimmat sozlash uchun (DB ulanish). O'zgaruvchan holatni `beforeAll` da yaratib bo'lishish flaky'lik beradi, shuning uchun mutable holat uchun har doim `beforeEach`.

---

## Masalalar

> Yechimlar: [solutions/testing/02-testing-in-practice.md](../solutions/testing/02-testing-in-practice.md)

1. **AAA + matcher tanlash.** `parsePrice(input)` funksiyasi uchun (`"12 500"` kabi string'ni `12500` raqamga aylantiradi, noto'g'ri input'da xato tashlaydi) `describe`/`it` strukturasida testlar yozing: normal holat, bo'sh string, va xato holat. Har bir assertion uchun aniq matcher (`toBe`/`toThrow`) tanlang va nega aynan shuni tanlaganingizni izohlang.

2. **toEqual tuzog'i.** Quyidagi test yiqiladi: `expect(getUser()).toBe({ id: 1, name: "Ali" })`. Nega yiqilishini tushuntiring va to'g'rilang. So'ng obyekt ichida `undefined` maydon bo'lgan holatda `toEqual` va `toStrictEqual` farqini ko'rsatuvchi test yozing.

3. **Async funksiyani test qilish.** `fetchUser(id)` `id < 1` bo'lsa rejected promise qaytaradi, aks holda `{ id, name }`. Buni ikki xil uslubda test qiling: (a) `async/await` bilan, (b) `resolves`/`rejects` matcher'lari bilan. Nima uchun `await`ni unutish xavfli ekanini ham bitta misol bilan ko'rsating.

4. **jest.fn bilan callback test qilish.** `retry(fn, times)` funksiyasi `fn` muvaffaqiyatli bo'lguncha (yoki `times` tugaguncha) qayta urinadi. `jest.fn()` ishlatib: (a) birinchi urinishda muvaffaqiyat, (b) ikki marta fail, uchinchida muvaffaqiyat holatlarini test qiling. `toHaveBeenCalledTimes` bilan urinishlar sonini tasdiqlang.

5. **Fake timer bilan debounce.** `debounce(fn, delay)` ni test qiling: tez ketma-ket chaqiruvlardan keyin `fn` faqat bir marta va faqat `delay` o'tgach chaqirilishini `jest.useFakeTimers` va `advanceTimersByTime` bilan tasdiqlang.

6. **RTL bilan komponent testi.** `<LoginForm onSubmit={...} />` komponenti email va parol input, hamda "Kirish" tugmasiga ega. RTL + `userEvent` bilan: foydalanuvchi maydonlarni to'ldirib tugmani bosganda `onSubmit` to'g'ri qiymatlar bilan chaqirilishini test qiling. `getByLabelText`/`getByRole` ishlating.

7. **findBy bilan async UI.** `<UserCard userId={1} />` mount bo'lganda foydalanuvchini fetch qiladi va ismini ko'rsatadi. `fetch` ni mock qilib, `findByText` bilan ism async paydo bo'lishini test qiling. Yuklanish vaqtida "Yuklanmoqda..." ko'rinishini ham tekshiring.

8. **test.each bilan parametrizatsiya.** `classifyAge(n)` funksiyasi: `<13` "bola", `13–17` "o'smir", `18–64` "kattalar", `≥65` "keksa" qaytaradi. `test.each` bilan har bir chegara (12, 13, 17, 18, 64, 65) uchun bitta jadval testi yozing.

9. **DI bilan test-friendly refactor.** Ichida `new SmsClient()` yaratib SMS yuboradigan `NotificationService` berilgan. Uni DI ishlatadigan qilib qayta yozing va mock client bilan "SMS to'g'ri raqamga yuborildi" testini yozing. Refaktordan oldin nega test qiyin bo'lganini izohlang.

← [Testing bo'limiga qaytish](./README.md)
