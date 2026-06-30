# React Asoslari — Yechimlar

Bu fayl [05-react-fundamentals.md](../../frontend/05-react-fundamentals.md) dagi "Masalalar" bo'limining yechimlari. Har bir yechim kod va o'zbekcha izoh bilan.

---

## 1. Counter va step

```tsx
import { useState } from "react";

function Counter({ step }: { step: number }) {
  const [count, setCount] = useState(0);
  // functional update: oldingi qiymatga ishonchli tarzda step qo'shadi
  return (
    <button onClick={() => setCount((prev) => prev + step)}>
      Hisob: {count}
    </button>
  );
}
```

**Izoh:** `setCount((prev) => prev + step)` — functional update. Bu eski qiymatga emas, React beradigan eng so'nggi qiymatga (`prev`) tayanadi. Agar bir nechta yangilanish ketma-ket bo'lsa ham, har biri to'g'ri ishlaydi. `setCount(count + step)` yozsangiz, bir hodisada bir nechta yangilanish bo'lganda stale qiymat muammosi yuzaga kelishi mumkin.

---

## 2. Controlled form

```tsx
import { useState } from "react";

function SignupForm() {
  const [name, setName] = useState("");
  const [email, setEmail] = useState("");

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault(); // sahifa qayta yuklanmasligi uchun
    console.log({ name, email });
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        value={name}
        onChange={(e) => setName(e.target.value)}
        placeholder="Ism"
      />
      <input
        value={email}
        onChange={(e) => setEmail(e.target.value)}
        placeholder="Email"
      />
      <button type="submit">Yuborish</button>
    </form>
  );
}
```

**Izoh:** Har bir input controlled — qiymati state'da (`value`), o'zgarishi `onChange` orqali state'ga yoziladi. Single source of truth state'da. `e.preventDefault()` form'ning standart submit (sahifani reload qilish) xatti-harakatini to'xtatadi.

---

## 3. Key bug'ni tuzating

Muammoli kod (index key bilan):

```tsx
// ❌ NOTO'G'RI: o'chirilganda input qiymatlari aralashadi
{items.map((item, index) => (
  <li key={index}>
    <input defaultValue={item.text} />
  </li>
))}
```

Tuzatilgan:

```tsx
// ✅ TO'G'RI: barqaror, ma'lumotga bog'liq id
{items.map((item) => (
  <li key={item.id}>
    <input defaultValue={item.text} />
  </li>
))}
```

**Izoh:** Index — element'ning pozitsiyasi, identifikatori emas. O'rtadagi element o'chirilsa, qolganlarning index'i siljiydi va React noto'g'ri DOM (input)'ni qayta ishlatadi — natijada uncontrolled input'lardagi qiymatlar boshqa qatorga "ko'chib" qoladi. `item.id` esa element bilan birga harakatlanadi, shuning uchun React har bir input'ni to'g'ri element'ga bog'lab turadi.

---

## 4. Lifting state up

```tsx
import { useState } from "react";

function TemperatureInput({
  label,
  value,
  onChange,
}: {
  label: string;
  value: string;
  onChange: (v: string) => void;
}) {
  return (
    <label>
      {label}:{" "}
      <input value={value} onChange={(e) => onChange(e.target.value)} />
    </label>
  );
}

function Calculator() {
  const [celsius, setCelsius] = useState("");

  const toF = (c: string) => (c === "" ? "" : String((+c * 9) / 5 + 32));
  const toC = (f: string) => (f === "" ? "" : String(((+f - 32) * 5) / 9));

  return (
    <div>
      <TemperatureInput label="Celsius" value={celsius} onChange={setCelsius} />
      <TemperatureInput
        label="Fahrenheit"
        value={toF(celsius)}
        onChange={(f) => setCelsius(toC(f))}
      />
    </div>
  );
}
```

**Izoh:** State (`celsius`) umumiy parent `Calculator`'ga ko'tarilgan — single source of truth. Ikkala input ham shu state'dan kelib chiqadi: Fahrenheit input qiymati `celsius`'dan hisoblanadi, o'zgartirilsa esa qaytadan `celsius`'ga yoziladi. Shuning uchun biri o'zgarganda ikkinchisi avtomatik yangilanadi.

---

## 5. Modal portal

```tsx
import { createPortal } from "react-dom";

function Modal({
  isOpen,
  onClose,
  children,
}: {
  isOpen: boolean;
  onClose: () => void;
  children: React.ReactNode;
}) {
  if (!isOpen) return null;

  return createPortal(
    <div className="modal-overlay" onClick={onClose}>
      <div className="modal-box" onClick={(e) => e.stopPropagation()}>
        {children}
        <button onClick={onClose}>Yopish</button>
      </div>
    </div>,
    document.body
  );
}
```

**Izoh:** `createPortal(jsx, document.body)` modal'ni DOM'ning yuqori qismiga render qiladi — bu `z-index`/`overflow` muammolaridan qochiradi. `isOpen` `false` bo'lsa `null` qaytarib, hech narsa render qilinmaydi. Overlay'ga bosilsa yopiladi; `stopPropagation` esa modal ichidagi bosish overlay'gacha "ko'tarilib", uni yopib qo'yishini to'sadi.

---

## 6. Conditional rendering tuzog'i

Muammo:

```tsx
// ❌ items bo'sh bo'lsa ekranda "0" ko'rinadi
{items.length && <List items={items} />}
```

`items.length === 0` bo'lsa, `0 && ...` natijasi `0` bo'ladi — va `0` renderlanadigan qiymat, shuning uchun ekranga chiqadi.

Tuzatilgan variantlar:

```tsx
// ✅ 1-variant: aniq boolean shartdan foydalanish
{items.length > 0 && <List items={items} />}

// ✅ 2-variant: ternary
{items.length ? <List items={items} /> : null}
```

**Izoh:** `&&` operatori chap tomon falsy bo'lsa o'sha falsy qiymatni qaytaradi. `0`, `NaN`, `""` falsy, lekin `0` va `NaN` ekranda ko'rinadi (`""`, `null`, `undefined`, `false` ko'rinmaydi). Shu sababli son ustida `&&` ishlatishdan oldin uni aniq boolean'ga aylantirish kerak.

---

## 7. Composition

```tsx
function Card({
  title,
  children,
}: {
  title: string;
  children: React.ReactNode;
}) {
  return (
    <div className="card">
      <h3 className="card-title">{title}</h3>
      <div className="card-body">{children}</div>
    </div>
  );
}

function App() {
  return (
    <>
      <Card title="Matn">
        <p>Oddiy paragraf.</p>
      </Card>

      <Card title="Ro'yxat">
        <ul>
          <li>Birinchi</li>
          <li>Ikkinchi</li>
        </ul>
      </Card>

      <Card title="Tugma">
        <button>Bosing</button>
      </Card>
    </>
  );
}
```

**Izoh:** `Card` o'z ichidagi kontentni bilmaydi — u faqat `title`'ni va `children` orqali kelgan har qanday narsani render qiladi. Bu composition: bir "qobiq" component'ni turli kontent bilan qayta ishlatamiz, inheritance'siz.

---

## 8. List render (faqat active)

```tsx
type User = { id: number; name: string; active: boolean };

function UserList({ users }: { users: User[] }) {
  return (
    <ul>
      {users
        .filter((u) => u.active)
        .map((u) => (
          <li key={u.id}>{u.name}</li>
        ))}
    </ul>
  );
}
```

**Izoh:** Avval `.filter()` bilan faqat `active` foydalanuvchilar tanlanadi, keyin `.map()` ularni `<li>`'ga aylantiradi. `key={u.id}` — barqaror, noyob identifikator (index emas). `.filter().map()` zanjiri keng tarqalgan va o'qilishi oson naqsh.

---

← [Frontend bo'limiga qaytish](../../frontend/README.md)
