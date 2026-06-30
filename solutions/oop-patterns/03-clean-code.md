# Clean Code va SWE Amaliyoti — Masalalar Yechimi

Bu fayl `03-clean-code.md` dagi mashqlarning to'liq yechimlari. Har bir yechim "yomon kod → yaxshi kod" ko'rinishida va izoh bilan berilgan.

## 1. Naming refactor

Maqsad: mazmunli nom + magic number'ni konstantaga aylantirish.

```ts
// ❌ Asl
function p(u: any[]): number {
  let r = 0;
  for (const x of u) {
    if (x.t > 30) r += x.a * 0.9;
  }
  return r;
}

// ✅ Yechim
interface Order {
  daysOpen: number;   // t
  amount: number;     // a
}

const STALE_ORDER_DAYS = 30;
const STALE_ORDER_DISCOUNT = 0.9;

function totalDiscountedAmountForStaleOrders(orders: Order[]): number {
  return orders
    .filter((order) => order.daysOpen > STALE_ORDER_DAYS)
    .reduce((sum, order) => sum + order.amount * STALE_ORDER_DISCOUNT, 0);
}
```

Izoh: `p`, `u`, `r`, `x.t`, `x.a` — hech narsa anglatmaydi. Nomlar maqsadni ochib berdi, `30` va `0.9` nomli konstantaga aylandi, `any` o'rniga aniq tip qo'yildi.

## 2. Guard clause

```ts
// ❌ Deep nesting
function canAccess(user: User): boolean {
  if (user) {
    if (user.isActive) {
      if (user.role === "admin" || user.role === "editor") {
        return true;
      }
    }
  }
  return false;
}

// ✅ Guard clause + early return
const ACCESS_ROLES = ["admin", "editor"];

function canAccess(user: User): boolean {
  if (!user) return false;
  if (!user.isActive) return false;
  return ACCESS_ROLES.includes(user.role);
}
```

Izoh: chetki holatlar boshida qaytarib yuborildi, asosiy shart bitta o'qiluvchan qatorga aylandi.

## 3. Long parameter list → Parameter Object

```ts
// ❌ 7 ta argument
function createOrder(productId: string, qty: number, userId: string,
  address: string, city: string, zip: string, isGift: boolean) { /* ... */ }

// ✅ Parameter Object + ichki guruhlash
interface ShippingAddress {
  street: string;
  city: string;
  zip: string;
}

interface CreateOrderInput {
  productId: string;
  quantity: number;
  userId: string;
  address: ShippingAddress;
  isGift: boolean;
}

function createOrder(input: CreateOrderInput): Order {
  // input.address.city ...
}
```

Izoh: bog'liq maydonlar (`address`, `city`, `zip`) tabiiy ravishda `ShippingAddress` obyektiga yig'ildi. Argumentlar tartibini eslab qolish kerak emas, kengaytirish oson.

## 4. DRY

```ts
// ✅ Extract Function
function formatFullName(person: { firstName: string; lastName: string }): string {
  return `${person.firstName} ${person.lastName}`;
}

const fullName = formatFullName(user);
const label = formatFullName(customer);
```

AHA izohi: Bu yerda takror **haqiqiy** — ikkalasi ham "to'liq ismni formatlash" degan bir xil bilimni ifodalaydi, shuning uchun birlashtirish to'g'ri. Ammo agar `user` ismi va `invoice` raqami tasodifan bir xil formatda bo'lsa-yu, ular har xil sababga ega bo'lsa, ularni birlashtirmaslik kerak edi — bu noto'g'ri abstraksiya bo'lardi.

## 5. Side-effect tozalash → Pure & Immutable

```ts
// ❌ Global mutatsiya, impure
let total = 0;
function addToTotal(value: number): void {
  total += value;
}

// ✅ Pure function
function add(current: number, value: number): number {
  return current + value;
}

// massiv uchun immutable variant:
function sum(values: readonly number[]): number {
  return values.reduce((acc, v) => acc + v, 0);
}
```

Izoh: global `total`ga bog'liqlik olib tashlandi. Endi funksiya bir xil input uchun bir xil natija beradi, test qilish oson (mock kerak emas) va concurrency uchun xavfsiz.

## 6. Error handling: null → exception

```ts
// ❌ null qaytaradi
function getConfig(key: string): string | null {
  return store[key] ?? null;
}

// ✅ Exception bilan
class ConfigKeyNotFoundError extends Error {
  constructor(key: string) {
    super(`Config key not found: ${key}`);
    this.name = "ConfigKeyNotFoundError";
  }
}

function getConfig(key: string): string {
  const value = store[key];
  if (value === undefined) {
    throw new ConfigKeyNotFoundError(key);
  }
  return value;
}
```

Chaqiruvchi kodga ta'siri: avval har bir chaqiruvchi `if (config === null)` tekshirishi kerak edi va unutilsa `null` kutilmagan joyda crash berardi. Endi qaytuvchi tip har doim `string` — chaqiruvchi qo'shimcha tekshiruvsiz ishlatadi, xato esa aniq, context bilan, markazda (masalan global error handler'da) ushlanadi.

Eslatma: agar kalitning yo'qligi **kutilgan, normal holat** bo'lsa (masalan ixtiyoriy sozlama), unda default qiymatli variant (`getConfig(key, fallback)`) yoki `Result` patterni ham to'g'ri tanlov bo'lishi mumkin. Exception kutilmagan holatlar uchun.

## 7. Code smell aniqlash

```ts
class UserManager {
  saveUser(u: any) { /* DB */ }
  sendEmail(u: any) { /* SMTP */ }
  generateReport(u: any) { /* PDF */ }
  validate(u: any) {
    if (u) { if (u.email) { if (u.email.includes("@")) { return true; } } }
    return false;
  }
}
```

Aniqlangan smell'lar (kamida 3 ta):

1. **Large Class / God Object (SRP buzilishi):** `UserManager` DB, email, PDF va validatsiya — to'rt xil mas'uliyatni birga bajaradi. → **Extract Class:** `UserRepository`, `EmailService`, `ReportGenerator`, `UserValidator`.
2. **Deep Nesting:** `validate` da uchta ichma-ich `if`. → **Guard clause:** `if (!u?.email) return false; return u.email.includes("@");`.
3. **Primitive Obsession / `any`:** Hamma joyda `any`. → Aniq `User` tipi va, masalan, `Email` value object.
4. **"Manager" nomi:** "Manager"/"Util" nomi ko'pincha aniq mas'uliyat yo'qligining belgisi. → Sinflarni mas'uliyatga ko'ra qayta nomlash.

Refactor namunasi:

```ts
class UserValidator {
  isValid(user: User): boolean {
    if (!user?.email) return false;
    return user.email.includes("@");
  }
}
class UserRepository { save(user: User): void { /* DB */ } }
class EmailService { send(user: User): void { /* SMTP */ } }
class ReportGenerator { generate(user: User): Report { /* PDF */ } }
```

## 8. Code review feedback

PR holati: 80 qatorli funksiya, 5 argument, bo'sh `catch {}`, testsiz. Namunaviy kommentariyalar:

1. **(Muhim — to'g'rilik) Bo'sh catch bloki.**
   > "Bu `catch {}` xatoni jimgina yutadi, keyinchalik bug'ni topib bo'lmay qoladi. Hech bo'lmaganda xatoni log qilsak yoki qayta tashlasak (`throw`) qanday bo'lardi? Foydalanuvchiga ham aniq xabar kerak bo'lishi mumkin."

2. **(Muhim — test) Test yo'qligi.**
   > "Bu logika muhim ko'rinadi, lekin test ko'rmadim. Kamida happy path va bitta-ikkita edge case (bo'sh input, xato holat) uchun test qo'shsak, kelajakdagi regressiyadan himoyalanamiz. Yordam kerak bo'lsa, ayt."

3. **(Dizayn) Uzun funksiya.**
   > "Funksiya ~80 qator va bir nechta ishni qilayotgandek. Mantiqiy bo'laklarni alohida nomli funksiyalarga (Extract Method) ajratsak, o'qish va test qilish osonlashadi. Masalan, validatsiya qismini `validateInput()` ga ajratish mumkinmi?"

4. **(nit — o'qiluvchanlik) Argumentlar soni.**
   > "nit: 5 argumentni eslab qolish qiyin va chaqiruvda tartibni adashtirish oson. Ularni bitta `input` obyektiga (Parameter Object) yig'sak yaxshiroq bo'lardi. Bu shoshilinch emas, lekin keyingi marta foydali."

Izoh: har bir komment kodga qaratilgan (odamga emas), sababini tushuntiradi, savol/taklif ohangida va muhimlik darajasi belgilangan ("nit:").

---

← [OOP bo'limiga qaytish](../../oop-patterns/README.md)
