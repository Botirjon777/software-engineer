# Amaliyotda Testing (JS/TS) — Yechimlar

Bu fayl [02-testing-in-practice.md](../../testing/02-testing-in-practice.md) dagi "Masalalar" bo'limining yechimlari. Har bir yechim to'liq test kodi va o'zbekcha izoh bilan.

---

## 1. AAA + matcher tanlash

```ts
// parsePrice.ts
export function parsePrice(input: string): number {
  if (typeof input !== "string" || input.trim() === "") {
    throw new Error("Bo'sh yoki noto'g'ri input");
  }
  const cleaned = input.replace(/\s/g, ""); // probellarni olib tashlash
  const n = Number(cleaned);
  if (Number.isNaN(n)) {
    throw new Error(`Raqamga aylantirib bo'lmadi: ${input}`);
  }
  return n;
}
```

```ts
// parsePrice.test.ts
import { parsePrice } from "./parsePrice";

describe("parsePrice", () => {
  it("probelli string'ni raqamga aylantiradi", () => {
    // Arrange
    const input = "12 500";
    // Act
    const result = parsePrice(input);
    // Assert — son qaytadi, primitiv, shuning uchun toBe
    expect(result).toBe(12500);
  });

  it("bo'sh string'da xato tashlaydi", () => {
    expect(() => parsePrice("")).toThrow("Bo'sh yoki noto'g'ri input");
  });

  it("harf bo'lsa xato tashlaydi", () => {
    expect(() => parsePrice("abc")).toThrow(/aylantirib bo'lmadi/);
  });
});
```

**Izoh:** Normal holatda natija — *primitiv son*, shuning uchun `toBe` (identiklik, `===` ga yaqin) to'g'ri matcher. Xato holatlarida `toThrow` ishlatiladi va chaqiruv `() => ...` callback ichiga o'raladi — aks holda xato `expect` ga yetmasdan testni yiqitadi. Xato xabarini (string yoki regex) ham beramiz, shunda "biror xato bo'ldi" emas, *aynan kutgan* xato sinaladi. AAA bosqichlari kommentlar bilan ajratilgan.

---

## 2. toEqual tuzog'i

```ts
function getUser() {
  return { id: 1, name: "Ali" };
}

// ❌ YIQILADI
// expect(getUser()).toBe({ id: 1, name: "Ali" });
//
// Sabab: toBe Object.is (===) ishlatadi — bu REFERENCE'ni solishtiradi.
// getUser() yangi obyekt qaytaradi, literal { id: 1, name: "Ali" } esa
// boshqa obyekt. Ikkalasining qiymati bir xil bo'lsa-da, reference'i
// har xil, shuning uchun toBe yiqiladi.

// ✅ TO'G'RI — chuqur qiymat solishtirish
it("getUser to'g'ri obyekt qaytaradi", () => {
  expect(getUser()).toEqual({ id: 1, name: "Ali" });
});
```

```ts
// toEqual vs toStrictEqual — undefined maydon farqi
describe("toEqual vs toStrictEqual", () => {
  const obj = { id: 1, name: undefined };

  it("toEqual undefined maydonni e'tiborsiz qoldiradi", () => {
    // toEqual { id: 1 } va { id: 1, name: undefined } ni TENG deb biladi
    expect(obj).toEqual({ id: 1 });
  });

  it("toStrictEqual undefined maydonni hisobga oladi", () => {
    // toStrictEqual ulardan farq topadi
    expect(obj).not.toStrictEqual({ id: 1 });
    expect(obj).toStrictEqual({ id: 1, name: undefined });
  });
});
```

**Izoh:** `toBe` primitivlar va bir xil reference uchun; obyekt/massiv uchun `toEqual` (deep, qiymat-qiymat). Asosiy tuzoq — obyektni `toBe` bilan solishtirish: qiymatlar mos bo'lsa ham reference har xil bo'lgani uchun yiqiladi. `toStrictEqual` esa `toEqual`'dan qattiqroq — `undefined` qiymatli kalitlarni va tur (class) farqini ham hisobga oladi.

---

## 3. Async funksiyani test qilish

```ts
// fetchUser.ts
export async function fetchUser(id: number): Promise<{ id: number; name: string }> {
  if (id < 1) {
    throw new Error("Noto'g'ri id");
  }
  return { id, name: "Ali" };
}
```

```ts
// fetchUser.test.ts
import { fetchUser } from "./fetchUser";

describe("fetchUser", () => {
  // (a) async/await uslubi
  it("await bilan — foydalanuvchini qaytaradi", async () => {
    const user = await fetchUser(1);
    expect(user).toEqual({ id: 1, name: "Ali" });
  });

  it("await bilan — xatoni try/catch orqali", async () => {
    await expect(fetchUser(-1)).rejects.toThrow("Noto'g'ri id");
  });

  // (b) resolves / rejects matcher'lari
  it("resolves matcher", async () => {
    await expect(fetchUser(2)).resolves.toEqual({ id: 2, name: "Ali" });
  });

  it("rejects matcher", async () => {
    await expect(fetchUser(0)).rejects.toThrow("Noto'g'ri id");
  });
});

// ⚠️ AWAIT'NI UNUTISH XAVFI:
it("XATO — await yo'q, hech qachon yiqilmaydi", () => {
  // Bu assertion promise hal bo'lishidan OLDIN test tugaydi.
  // rejects ishlamaydi, test noto'g'ri "pass" bo'ladi.
  expect(fetchUser(-1)).rejects.toThrow("BU XABAR NOTO'G'RI BO'LSA HAM PASS"); // ❌
});
```

**Izoh:** Async testda asosiy qoida — promise tugashini kutish. `async/await` uslubi eng aniq. `resolves`/`rejects` matcher'lari oldida ham `await` (yoki `return`) bo'lishi *shart* — bu runner'ga "men kutyapman" deb signal beradi. Eng xavfli xato — oxirgi misoldagidek `await`ni unutish: assertion umuman tekshirilmaydi va test soxta "yashil" bo'ladi. `eslint-plugin-jest` ning `valid-expect` qoidasi buni ushlaydi.

---

## 4. jest.fn bilan callback test qilish

```ts
// retry.ts
export async function retry<T>(fn: () => Promise<T>, times: number): Promise<T> {
  let lastError: unknown;
  for (let i = 0; i < times; i++) {
    try {
      return await fn();
    } catch (e) {
      lastError = e;
    }
  }
  throw lastError;
}
```

```ts
// retry.test.ts
import { retry } from "./retry";

describe("retry", () => {
  it("(a) birinchi urinishda muvaffaqiyat — bir marta chaqiradi", async () => {
    const fn = jest.fn().mockResolvedValue("ok");

    const result = await retry(fn, 3);

    expect(result).toBe("ok");
    expect(fn).toHaveBeenCalledTimes(1);
  });

  it("(b) ikki marta fail, uchinchida muvaffaqiyat", async () => {
    const fn = jest
      .fn()
      .mockRejectedValueOnce(new Error("1-fail"))
      .mockRejectedValueOnce(new Error("2-fail"))
      .mockResolvedValue("ok");

    const result = await retry(fn, 5);

    expect(result).toBe("ok");
    expect(fn).toHaveBeenCalledTimes(3); // uch urinish
  });

  it("hamma urinish fail bo'lsa oxirgi xatoni tashlaydi", async () => {
    const fn = jest.fn().mockRejectedValue(new Error("doim fail"));

    await expect(retry(fn, 3)).rejects.toThrow("doim fail");
    expect(fn).toHaveBeenCalledTimes(3);
  });
});
```

**Izoh:** `jest.fn()` soxta funksiya yaratadi va chaqiruvlarni yozib boradi. `mockResolvedValue` — promise qaytaradigan funksiya uchun. `mockRejectedValueOnce` — *faqat shu safargi* chaqiruv uchun rejected qaytaradi (zanjir qilib har urinishga boshqa xulq beriladi), keyin `mockResolvedValue` qolgan hamma chaqiruvga amal qiladi. `toHaveBeenCalledTimes` urinishlar sonini aniq tasdiqlaydi — bu retry mantig'ining to'g'riligini ko'rsatadi.

---

## 5. Fake timer bilan debounce

```ts
// debounce.ts
export function debounce<A extends unknown[]>(
  fn: (...args: A) => void,
  delay: number,
) {
  let timer: ReturnType<typeof setTimeout> | undefined;
  return (...args: A) => {
    if (timer) clearTimeout(timer);
    timer = setTimeout(() => fn(...args), delay);
  };
}
```

```ts
// debounce.test.ts
import { debounce } from "./debounce";

describe("debounce", () => {
  beforeEach(() => jest.useFakeTimers());
  afterEach(() => jest.useRealTimers());

  it("delay o'tmaguncha chaqirmaydi", () => {
    const fn = jest.fn();
    const debounced = debounce(fn, 300);

    debounced();
    expect(fn).not.toHaveBeenCalled(); // hali emas

    jest.advanceTimersByTime(299);
    expect(fn).not.toHaveBeenCalled(); // hali ham emas

    jest.advanceTimersByTime(1); // jami 300ms
    expect(fn).toHaveBeenCalledTimes(1);
  });

  it("tez ketma-ket chaqiruvlardan faqat bittasi ishlaydi", () => {
    const fn = jest.fn();
    const debounced = debounce(fn, 300);

    debounced("a");
    jest.advanceTimersByTime(100);
    debounced("b"); // oldingisini bekor qiladi
    jest.advanceTimersByTime(100);
    debounced("c"); // yana bekor qiladi
    jest.advanceTimersByTime(300);

    expect(fn).toHaveBeenCalledTimes(1);
    expect(fn).toHaveBeenCalledWith("c"); // faqat oxirgi argument
  });
});
```

**Izoh:** `jest.useFakeTimers()` `setTimeout`/`clearTimeout` ni mock qiladi — real vaqt kutmaymiz. `jest.advanceTimersByTime(ms)` bilan vaqtni qo'lda "oldinga suramiz". Ikkinchi test debounce'ning asl mohiyatini sinaydi: tez ketma-ket chaqiruvlar oldingisini bekor qiladi, faqat oxirgisi `delay`dan keyin ishlaydi (`toHaveBeenCalledWith("c")`). `afterEach` da `useRealTimers` — aks holda keyingi testlarda timer "muzlab" qoladi.

---

## 6. RTL bilan komponent testi

```tsx
// LoginForm.tsx
import { useState } from "react";

export function LoginForm({
  onSubmit,
}: {
  onSubmit: (data: { email: string; password: string }) => void;
}) {
  const [email, setEmail] = useState("");
  const [password, setPassword] = useState("");

  return (
    <form
      onSubmit={(e) => {
        e.preventDefault();
        onSubmit({ email, password });
      }}
    >
      <label>
        Email
        <input value={email} onChange={(e) => setEmail(e.target.value)} />
      </label>
      <label>
        Parol
        <input
          type="password"
          value={password}
          onChange={(e) => setPassword(e.target.value)}
        />
      </label>
      <button type="submit">Kirish</button>
    </form>
  );
}
```

```tsx
// LoginForm.test.tsx
import { render, screen } from "@testing-library/react";
import userEvent from "@testing-library/user-event";
import { LoginForm } from "./LoginForm";

it("forma to'ldirilib yuborilganda onSubmit to'g'ri chaqiriladi", async () => {
  const onSubmit = jest.fn();
  render(<LoginForm onSubmit={onSubmit} />);

  // Foydalanuvchi kabi: label orqali maydonni topib, yozamiz
  await userEvent.type(screen.getByLabelText("Email"), "ali@example.uz");
  await userEvent.type(screen.getByLabelText("Parol"), "secret123");
  await userEvent.click(screen.getByRole("button", { name: "Kirish" }));

  expect(onSubmit).toHaveBeenCalledTimes(1);
  expect(onSubmit).toHaveBeenCalledWith({
    email: "ali@example.uz",
    password: "secret123",
  });
});
```

**Izoh:** "Test like a user" — komponentning ichki `useState` holatini emas, foydalanuvchi ko'rgan va qilgan narsani sinaymiz. `getByLabelText` maydonni *label* orqali topadi (accessibility'ga mos, foydalanuvchi ham labelni o'qiydi). `userEvent.type` real klaviatura kiritishini taqlid qiladi, `userEvent.click` tugmani bosadi. Natijada `onSubmit` ning to'g'ri argument bilan, bir marta chaqirilishini tekshiramiz. `onSubmit` ni `jest.fn()` (spy) sifatida uzatdik.

---

## 7. findBy bilan async UI

```tsx
// UserCard.tsx
import { useEffect, useState } from "react";

export function UserCard({ userId }: { userId: number }) {
  const [name, setName] = useState<string | null>(null);

  useEffect(() => {
    fetch(`/api/users/${userId}`)
      .then((r) => r.json())
      .then((u) => setName(u.name));
  }, [userId]);

  if (name === null) return <p>Yuklanmoqda...</p>;
  return <h2>{name}</h2>;
}
```

```tsx
// UserCard.test.tsx
import { render, screen } from "@testing-library/react";
import { UserCard } from "./UserCard";

beforeEach(() => {
  global.fetch = jest.fn().mockResolvedValue({
    ok: true,
    json: async () => ({ id: 1, name: "Ali" }),
  } as Response);
});

afterEach(() => jest.restoreAllMocks());

it("yuklanish holatini, keyin ismni ko'rsatadi", async () => {
  render(<UserCard userId={1} />);

  // Boshida — yuklanish matni (sinxron, getBy)
  expect(screen.getByText("Yuklanmoqda...")).toBeInTheDocument();

  // fetch hal bo'lgach ism async paydo bo'ladi — findBy bilan kutamiz
  expect(await screen.findByText("Ali")).toBeInTheDocument();

  // Yuklanish matni endi yo'q — yo'qlikni queryBy bilan tekshiramiz
  expect(screen.queryByText("Yuklanmoqda...")).not.toBeInTheDocument();

  expect(fetch).toHaveBeenCalledWith("/api/users/1");
});
```

**Izoh:** Uchta query oilasi bir testda ko'rsatilgan: `getByText` — *hozir mavjud* "Yuklanmoqda..." uchun (sinxron). `findByText` — fetch hal bo'lgach *async paydo bo'ladigan* ism uchun (`await` bilan kutadi, ichida `waitFor` bor). `queryByText` — yuklanish matnining *yo'qligini* tekshirish uchun (`getBy` bu yerda xato tashlardi). `global.fetch` ni mock qilib real network'siz, deterministik javob beramiz.

---

## 8. test.each bilan parametrizatsiya

```ts
// classifyAge.ts
export function classifyAge(n: number): string {
  if (n < 13) return "bola";
  if (n <= 17) return "o'smir";
  if (n <= 64) return "kattalar";
  return "keksa";
}
```

```ts
// classifyAge.test.ts
import { classifyAge } from "./classifyAge";

describe("classifyAge", () => {
  test.each([
    [12, "bola"],
    [13, "o'smir"],
    [17, "o'smir"],
    [18, "kattalar"],
    [64, "kattalar"],
    [65, "keksa"],
  ])("classifyAge(%i) === %s", (input, expected) => {
    expect(classifyAge(input)).toBe(expected);
  });
});
```

**Izoh:** `test.each` jadval beradi, har qator *alohida* test bo'lib ishlaydi — fail bo'lganda qaysi qator (qaysi yosh) yiqilganini aniq ko'rsatadi. Aynan *chegara qiymatlar* (12/13, 17/18, 64/65) tanlandi, chunki off-by-one xatolar shu yerda yashiringan bo'ladi. `%i` (integer) va `%s` (string) — test nomidagi formatlash belgilari. Bu yondashuv copy-paste'siz, o'qiladigan va edge case'larni bir joyda jamlaydi.

---

## 9. DI bilan test-friendly refactor

```ts
// ❌ AVVAL — bog'liqlik ichida yaratilgan, test qiyin
class SmsClient {
  send(phone: string, text: string) {
    /* real SMS API'ga so'rov */
  }
}

class NotificationServiceOld {
  notify(phone: string) {
    const client = new SmsClient(); // qattiq bog'liqlik
    client.send(phone, "Salom!");
  }
}
// Nega qiyin: NotificationServiceOld ni test qilish uchun SmsClient
// modulini mock qilish kerak (jest.mock), real SMS yuborilib ketmasligi
// uchun. Mock murakkab va kod bilan qattiq bog'langan.
```

```ts
// ✅ KEYIN — DI bilan, bog'liqlik tashqaridan uzatiladi
interface SmsSender {
  send(phone: string, text: string): void;
}

class NotificationService {
  constructor(private readonly sms: SmsSender) {}

  notify(phone: string) {
    this.sms.send(phone, "Salom!");
  }
}
```

```ts
// NotificationService.test.ts
import { NotificationService } from "./NotificationService";

it("SMS to'g'ri raqamga va matn bilan yuboriladi", () => {
  const sms = { send: jest.fn() }; // oddiy mock — modul mock kerak emas
  const service = new NotificationService(sms);

  service.notify("+998901234567");

  expect(sms.send).toHaveBeenCalledTimes(1);
  expect(sms.send).toHaveBeenCalledWith("+998901234567", "Salom!");
});
```

**Izoh:** Refaktordan oldin `SmsClient` *ichida* `new` qilingani uchun, test qilishda butun modulni `jest.mock('./SmsClient')` bilan almashtirish va real SMS yuborilmasligini ta'minlash kerak edi — murakkab va mo'rt. DI'dan keyin bog'liqlik konstruktor orqali *tashqaridan* keladi (`SmsSender` interfeysi), shuning uchun testda oddiy `{ send: jest.fn() }` mock'ni uzatamiz — modul mock'ning hech qanday murakkabligi yo'q. `toHaveBeenCalledWith` raqam va matn to'g'riligini tasdiqlaydi.

---

← [Testing bo'limiga qaytish](../../testing/README.md)
