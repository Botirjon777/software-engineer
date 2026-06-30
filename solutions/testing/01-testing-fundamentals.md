# Testing Asoslari — Yechimlar

Bu fayl [01-testing-fundamentals.md](../../testing/01-testing-fundamentals.md) dagi "Masalalar" bo'limining yechimlari. Har bir yechim to'liq test kodi va o'zbekcha izoh bilan.

---

## 1. AAA bilan test yozish

```js
// applyDiscount.js
export function applyDiscount(price, percent) {
  if (percent < 0 || percent > 100) {
    throw new Error("Foiz 0–100 oralig'ida bo'lishi kerak");
  }
  return price - (price * percent) / 100;
}
```

```js
// applyDiscount.test.js
import { applyDiscount } from "./applyDiscount.js";

describe("applyDiscount", () => {
  test("normal holat — 20% chegirma qo'llaydi", () => {
    // Arrange
    const price = 100;
    const percent = 20;
    // Act
    const result = applyDiscount(price, percent);
    // Assert
    expect(result).toBe(80);
  });

  test("chegara — 0% narxni o'zgartirmaydi", () => {
    // Arrange
    const price = 100;
    // Act
    const result = applyDiscount(price, 0);
    // Assert
    expect(result).toBe(100);
  });

  test("chegara — 100% narxni nolga tushiradi", () => {
    expect(applyDiscount(100, 100)).toBe(0);
  });

  test("noto'g'ri foiz — 100 dan katta bo'lsa xato", () => {
    expect(() => applyDiscount(100, 150)).toThrow("0–100 oralig'ida");
  });

  test("noto'g'ri foiz — manfiy bo'lsa xato", () => {
    expect(() => applyDiscount(100, -5)).toThrow();
  });
});
```

**Izoh:** Har bir test AAA bosqichlarini kommentlar bilan aniq ajratadi: Arrange (ma'lumot tayyorlash), Act (funksiyani chaqirish), Assert (natijani solishtirish). Normal holat (`toBe(80)`), ikkala chegara (0% va 100%) va noto'g'ri input (xato — `toThrow`) qoplangan. Xato testlarida chaqiruv `() => ...` callback ichiga o'raladi, aks holda xato `expect` ga yetmasdan testni yiqitadi.

---

## 2. Test double turini tanlash

```js
// (a) STUB — getExchangeRate har doim 12500 qaytarsin.
// Sabab: bizga faqat OLDINDAN TAYYOR JAVOB kerak, chaqiruvni
// tekshirmaymiz. Bu — stub (state verification uchun).
const rateStub = { getExchangeRate: () => 12500 };

test("stub — kurs bo'yicha hisoblaydi", () => {
  const result = convertToUsd(25000, rateStub);
  expect(result).toBe(2); // 25000 / 12500
});

// (b) SPY — logger.log chaqirilganini tekshirish.
// Sabab: haqiqiy xulqni o'zgartirmasdan CHAQIRUVNI KUZATAMIZ.
// jest.fn() spy rolini bajaradi.
test("spy — logger chaqirilganini kuzatadi", () => {
  const logger = { log: jest.fn() };
  doWork(logger);
  expect(logger.log).toHaveBeenCalled();
});

// (c) MOCK — email yuborilgani va aynan qaysi manzilga ekanini tasdiqlash.
// Sabab: KUTILGAN O'ZARO TA'SIR (qaysi argument bilan chaqirilishi)
// muhim — bu behavior verification, ya'ni mock.
test("mock — email to'g'ri manzilga yuboriladi", () => {
  const mailer = { send: jest.fn() };
  notifyUser(mailer, "ali@example.uz");
  expect(mailer.send).toHaveBeenCalledWith(
    "ali@example.uz",
    expect.any(String),
  );
});

// (d) FAKE — in-memory user repository.
// Sabab: ISHLAYDIGAN, lekin soddalashtirilgan implementatsiya.
// Real DB o'rniga Map ishlatadi.
class FakeUserRepo {
  #data = new Map();
  save(user) {
    this.#data.set(user.id, user);
  }
  findById(id) {
    return this.#data.get(id);
  }
}

test("fake — repo saqlaydi va o'qiydi", () => {
  const repo = new FakeUserRepo();
  repo.save({ id: 1, name: "Ali" });
  expect(repo.findById(1)).toEqual({ id: 1, name: "Ali" });
});
```

**Izoh:** Tanlovning kaliti — *state verification* (natija to'g'rimi?) yoki *behavior verification* (to'g'ri chaqiruv bo'ldimi?). (a) Stub — faqat tayyor javob beradi, natijani tekshiramiz. (b) Spy — xulqni o'zgartirmay chaqiruvni yozadi. (c) Mock — kutilgan o'zaro ta'sirni (`toHaveBeenCalledWith`) majburlaydi. (d) Fake — Map asosida ishlaydigan soddalashtirilgan DB. Amalda Jest'da `jest.fn()` spy va mock ikkala rolni ham bajara oladi — farq biz uni *qanday tekshirayotganimizda*.

---

## 3. TDD bilan FizzBuzz

```js
// QADAM 1 — RED: funksiya yo'q, test fail
test("3 ga bo'linsa Fizz", () => {
  expect(fizzbuzz(3)).toBe("Fizz");
});

// QADAM 2 — GREEN: eng oddiy o'tkazuvchi kod
function fizzbuzz(n) {
  return "Fizz";
}

// QADAM 3 — RED: yangi holat qo'shamiz, test fail
test("5 ga bo'linsa Buzz", () => {
  expect(fizzbuzz(5)).toBe("Buzz");
});

// QADAM 4 — GREEN
function fizzbuzz2(n) {
  if (n % 3 === 0) return "Fizz";
  if (n % 5 === 0) return "Buzz";
  return String(n);
}

// QADAM 5 — RED: ikkalasiga bo'linish holati
test("15 ga bo'linsa FizzBuzz", () => {
  expect(fizzbuzz(15)).toBe("FizzBuzz");
});

// QADAM 6 — GREEN: FizzBuzz shartini ENG OLDIN tekshiramiz
test("oddiy son o'zini qaytaradi", () => {
  expect(fizzbuzz(7)).toBe("7");
});
```

```js
// fizzbuzz.js — yakuniy (REFACTOR'dan keyin) kod
export function fizzbuzz(n) {
  if (n % 15 === 0) return "FizzBuzz"; // 3 VA 5 — birinchi tekshiriladi
  if (n % 3 === 0) return "Fizz";
  if (n % 5 === 0) return "Buzz";
  return String(n);
}
```

```js
// fizzbuzz.test.js — to'liq test to'plami
import { fizzbuzz } from "./fizzbuzz.js";

describe("fizzbuzz", () => {
  test.each([
    [3, "Fizz"],
    [6, "Fizz"],
    [5, "Buzz"],
    [10, "Buzz"],
    [15, "FizzBuzz"],
    [30, "FizzBuzz"],
    [7, "7"],
    [1, "1"],
  ])("fizzbuzz(%i) === %s", (input, expected) => {
    expect(fizzbuzz(input)).toBe(expected);
  });
});
```

**Izoh:** Red-Green-Refactor sikli har holat uchun takrorlanadi: avval *fail bo'ladigan* test (Red), keyin uni o'tkazadigan *minimal* kod (Green). Muhim dizayn nuqtasi — `% 15` (FizzBuzz) shartini *eng oldin* tekshirish kerak, aks holda 15 uchun "Fizz" qaytib qoladi (`% 3` birinchi mos kelib ketadi). Bu xatoni 15 uchun yozilgan test ushlaydi — TDD'ning foydasi shu.

---

## 4. Branch coverage tutqun

```js
// classify.js
export function classify(n) {
  if (n > 0) return "positive";
  else return "non-positive";
}
```

```js
// 100% LINE coverage, lekin bug o'tib ketadi
test("faqat positive holat", () => {
  expect(classify(5)).toBe("positive");
});
// Bu test classify funksiyasidagi HAR IKKALA qatorni "bajardi"
// deb hisoblanadimi? Yo'q — "else" shoxi (n <= 0) HECH QACHON
// sinalmadi. Agar else'da bug bo'lsa (masalan, "positive"
// qaytarsa ham), bu test pass bo'lib qolaveradi.
```

```js
// classify.test.js — branch coverage 100%
import { classify } from "./classify.js";

describe("classify", () => {
  test("musbat son — positive shoxi", () => {
    expect(classify(5)).toBe("positive");
  });

  test("nol — non-positive shoxi (avval qoplanmagan!)", () => {
    expect(classify(0)).toBe("non-positive");
  });

  test("manfiy son — non-positive shoxi", () => {
    expect(classify(-3)).toBe("non-positive");
  });
});
```

**Izoh:** Birinchi test faqat `n > 0` shoxini bajaradi. Line coverage "har bir qator ishga tushdimi?" deydi — `if` qatori ishga tushdi, lekin `else` *natijasi* (n <= 0) hech qachon olinmadi. **Branch coverage** har bir `if`/`else` shoxining *ikkala* tomonini talab qiladi. Asosiy dars: coverage to'g'rilikni isbotlamaydi — `else` shoxida bug bo'lsa, faqat positive testi uni ko'rmaydi. Qo'shilgan testlar `n = 0` (chegara) va `n < 0` ni qoplab, ikkinchi shoxni sinaydi.

---

## 5. Flaky testni tuzatish

```js
// jobRunner.js
export function startJob(onDone) {
  setTimeout(() => onDone("natija"), 100);
}
```

```js
// ❌ FLAKY — qattiq sleep/timing'ga umid
test("flaky — kutilmagan timing", (done) => {
  let result;
  startJob((r) => {
    result = r;
  });
  setTimeout(() => {
    expect(result).toBe("natija"); // ba'zan hali tayyor emas!
    done();
  }, 100); // job ham 100ms, bu ham 100ms — yarish (race)
});
// Nega flaky: ikkala setTimeout ham ~100ms. Qaysi biri avval
// ishga tushishi kafolatlanmagan — ba'zan assert callback'dan
// oldin ishlaydi va result hali undefined bo'ladi.
```

```js
// ✅ YONDASHUV 1 — fake timers (deterministik)
test("barqaror — fake timers bilan", () => {
  jest.useFakeTimers();
  const onDone = jest.fn();

  startJob(onDone);
  expect(onDone).not.toHaveBeenCalled(); // hali emas

  jest.advanceTimersByTime(100); // vaqtni qo'lda surdik
  expect(onDone).toHaveBeenCalledWith("natija");

  jest.useRealTimers();
});

// ✅ YONDASHUV 2 — waitFor (natija paydo bo'lishini kutish)
import { waitFor } from "@testing-library/dom"; // yoki o'z waitFor'ingiz

test("barqaror — waitFor bilan", async () => {
  let result;
  startJob((r) => {
    result = r;
  });

  // qattiq sleep emas, SHART bajarilguncha qayta tekshiradi
  await waitFor(() => expect(result).toBe("natija"));
});
```

**Izoh:** Flaky'lik sababi — ikki `setTimeout` orasidagi *race* (qaysi biri avval ishga tushishi noma'lum). Yechim 1 (**fake timers**) vaqtni umuman real qilmaydi: `advanceTimersByTime` bilan vaqtni aniq nazorat qilamiz — deterministik va tez. Yechim 2 (**waitFor**) qattiq `sleep(100)` o'rniga shart bajarilguncha qisqa intervallarda qayta tekshiradi — natija tayyor bo'lishi bilan davom etadi, kerakmas kutish yo'q. Ikkalasi ham timing'ga "umid"ni yo'q qiladi.

---

## 6. FIRST buzilishini topish

```js
// ❌ BUZILGAN — global cache testlar orasida bo'lishilgan
let cache = [];

test("a — bitta element qo'shadi", () => {
  cache.push("x");
  expect(cache.length).toBe(1);
});

test("b — yana qo'shadi", () => {
  cache.push("y");
  expect(cache.length).toBe(2); // OLDINGI test'ga bog'liq!
});

test("c — uchinchi", () => {
  cache.push("z");
  expect(cache.length).toBe(3); // tartibga bog'liq, alohida fail bo'ladi
});
```

Buzilgan printsip — **Independent** (mustaqillik): har bir test boshqasidan ajralgan bo'lishi kerak. Bu yerda `cache` global o'zgaruvchan holat — `b` testi `a` qoldirgan holatga tayanadi. Testlar boshqa tartibda yoki yakka (`.only`) ishga tushsa, yiqiladi.

```js
// ✅ TO'G'RI — har test o'z toza holatini oladi
describe("cache", () => {
  let cache;

  beforeEach(() => {
    cache = []; // har testdan oldin yangi, toza massiv
  });

  test("a — bitta element qo'shadi", () => {
    cache.push("x");
    expect(cache.length).toBe(1);
  });

  test("b — bitta element qo'shadi", () => {
    cache.push("y");
    expect(cache.length).toBe(1); // endi boshqa testga bog'liq emas
  });

  test("c — bitta element qo'shadi", () => {
    cache.push("z");
    expect(cache.length).toBe(1);
  });
});
```

**Izoh:** Buzilgan printsip — **I (Independent)**. Yechim: o'zgaruvchan holatni `beforeEach` da qayta yaratish, shunda har test toza `cache = []` dan boshlaydi. Endi har biri faqat *o'zi qo'shgan* elementni tekshiradi (`length === 1`) va tartibga, boshqa testlarga bog'liq emas. Global mutable holat — Independence'ni buzadigan eng keng tarqalgan sabab.

---

## 7. Mocking chegarasi

```js
// createUser.js
import { hashPassword } from "./crypto.js"; // sof funksiya
// db va mailer — DI orqali uzatiladi

export async function createUser(input, db, mailer) {
  const passwordHash = hashPassword(input.password); // sof — mock QILMAYMIZ
  const user = await db.save({ email: input.email, passwordHash }); // mock
  await mailer.send(input.email, "Xush kelibsiz!"); // mock
  return user;
}
```

```js
// createUser.test.js
import { createUser } from "./createUser.js";

test("user yaratadi: DB va mailer mock, hash haqiqiy", async () => {
  // DB — tashqi bog'liqlik, mock qilamiz (real DB'ga yozmaymiz)
  const db = { save: jest.fn().mockResolvedValue({ id: 1 }) };
  // mailer — yon ta'sir (email), mock qilamiz (real email yubormaymiz)
  const mailer = { send: jest.fn().mockResolvedValue(true) };

  const result = await createUser(
    { email: "ali@example.uz", password: "secret123" },
    db,
    mailer,
  );

  // DB'ga to'g'ri ma'lumot yozildimi? passwordHash HAQIQIY hash bo'lishi kerak
  expect(db.save).toHaveBeenCalledWith({
    email: "ali@example.uz",
    passwordHash: expect.any(String),
  });
  // Hash original parolga teng EMAS (haqiqatan hash qilingan)
  expect(db.save.mock.calls[0][0].passwordHash).not.toBe("secret123");

  // Email to'g'ri manzilga yuborildimi?
  expect(mailer.send).toHaveBeenCalledWith("ali@example.uz", "Xush kelibsiz!");

  expect(result).toEqual({ id: 1 });
});
```

**Izoh:** Mock qilish chegarasi (boundary) — *tashqi va yon ta'sirli* bog'liqliklar. `db` (tashqi resurs — real DB'ga yozmaslik kerak) va `mailer` (yon ta'sir — real email yubormaslik kerak) mock qilinadi. `hashPassword` esa — o'z *sof funksiyamiz*: uni mock qilmaymiz, haqiqiy ishlatamiz, chunki uni mock qilsak, kodning haqiqiy mantig'i sinalmaydi. `expect.any(String)` va `not.toBe("secret123")` — hash haqiqatan bajarilganini, lekin aniq qiymatga bog'lanmasdan tekshiradi.

---

## 8. Nimani test qilmaslik

```js
// User.js
export class User {
  #id;
  #income;
  constructor(id, income) {
    this.#id = id;
    this.#income = income;
  }
  getId() {
    return this.#id; // trivial getter
  }
  setId(id) {
    this.#id = id; // trivial setter
  }
  calculateTax() {
    // biznes logika — progressiv soliq
    if (this.#income <= 1_000_000) return this.#income * 0.07;
    return 1_000_000 * 0.07 + (this.#income - 1_000_000) * 0.12;
  }
  _buildQuery() {
    return `SELECT * FROM users WHERE id = ${this.#id}`; // private detal
  }
}
```

```js
// User.test.js — FAQAT arzigan qism: calculateTax
import { User } from "./User.js";

describe("User.calculateTax", () => {
  test("chegaradan past daromad — 7% soliq", () => {
    const user = new User(1, 500_000);
    expect(user.calculateTax()).toBe(35_000); // 500000 * 0.07
  });

  test("aniq chegara — 1 000 000", () => {
    const user = new User(1, 1_000_000);
    expect(user.calculateTax()).toBe(70_000);
  });

  test("chegaradan yuqori — progressiv stavka", () => {
    const user = new User(1, 2_000_000);
    // 1000000*0.07 + 1000000*0.12 = 70000 + 120000
    expect(user.calculateTax()).toBe(190_000);
  });
});
```

**Izoh:** Test qilamiz — **`calculateTax()`**: bu *biznes logika*, shartlar va chegaralarga ega, bug yashirinishi mumkin. Chegara qiymatlar (past, aniq `1_000_000`, yuqori) sinaladi. Test *qilmaymiz*: **`getId`/`setId`** — trivial getter/setter, hech qanday mantiq yo'q (ularni test qilish til ishlashini test qilish bilan teng — ortiqcha). **`_buildQuery`** — *private* implementatsiya detali; uni to'g'ridan-to'g'ri test qilish kodni ichki tuzilishiga bog'lab qo'yadi va refactoringda sinadi. Uning natijasi `calculateTax` kabi ko'rinadigan xulq orqali bilvosita qoplanadi. Qoida: *xulq-atvorni* test qiling, implementatsiyani emas.

---

← [Testing bo'limiga qaytish](../../testing/README.md)
