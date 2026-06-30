# Clean Code va SWE Amaliyoti

Toza kod (Clean Code) — bu shunchaki "chiroyli ko'rinadigan" kod emas. Bu boshqa odam (yoki 6 oydan keyingi o'zing) tez tushunadigan, oson o'zgartiriladigan va kam xato chiqadigan kod. Bu hujjat dasturchilar uchun intervyuga tayyorlanish va kundalik amaliyotni mustahkamlash uchun yozilgan. Bu yerda nega toza kod muhimligini, naming, funksiyalar, comment'lar, code smell'lar, refactoring texnikalari, error handling, code review va technical debt mavzularini chuqur ko'rib chiqamiz. Misollar TypeScript'da, lekin g'oyalar barcha tillarga tegishli.

**💡 Tushuncha:** Kod yozilishidan ko'ra ko'p marta o'qiladi. Robert Martin (Uncle Bob) ta'kidlaganidek, o'qish vaqti yozish vaqtidan deyarli 10 baravar ko'p. Shuning uchun "o'qiluvchanlik" (readability) — birinchi optimizatsiya maqsadi.

## Mundarija

- [Nega toza kod kerak](#nega-toza-kod-kerak)
- [Meaningful Naming](#meaningful-naming)
- [Funksiyalar](#funksiyalar)
- [Comment'lar](#commentlar)
- [DRY, KISS, YAGNI](#dry-kiss-yagni)
- [Code Smell'lar](#code-smelllar)
- [Refactoring Texnikalari](#refactoring-texnikalari)
- [Error Handling](#error-handling)
- [Pure Function va Immutability](#pure-function-va-immutability)
- [Boy Scout Rule](#boy-scout-rule)
- [Code Review](#code-review)
- [Technical Debt](#technical-debt)
- [Savol-Javoblar](#savol-javoblar)
- [Masalalar](#masalalar)

## Nega toza kod kerak

Kod bizning asosiy aloqa vositamiz — kompyuter bilan emas, balki **boshqa dasturchilar bilan**. Kompyuterga 0 va 1 yetarli; toza kod esa jamoaga kerak.

Toza kodning foydalari:

- **O'qish tezligi.** Yangi a'zo loyihaga qo'shilganda yoki bug qidirayotganda kodni tez tushunadi.
- **O'zgartirish narxi kam.** Yaxshi tuzilgan kodda yangi funksiya qo'shish yoki bug tuzatish xavfsiz va tez.
- **Kam xato.** Soddalik xato uchun joy qoldirmaydi. Murakkablik xatolarni yashiradi.
- **Onboarding tez.** Yangi dasturchi 1 hafta o'rniga 2 kunda samarali bo'la oladi.

**⚠️ Ehtiyot bo'l:** "Keyinroq tozalayman" — eng ko'p aldaydigan jumla. "Keyinroq" hech qachon kelmaydi. Toza kod hozir, kod yozayotgan paytda yoziladi.

## Meaningful Naming

Yaxshi nom — eng kuchli hujjatlash vositasi. Nom o'qiganda nima ekanligi, nima uchun kerakligi tushunarli bo'lishi kerak.

Qoidalar:

- **Maqsadni ochib beruvchi nom (intention-revealing).** `d` emas, `elapsedTimeInDays`.
- **Talaffuz qilinadigan nom.** `genymdhms` emas, `generationTimestamp`.
- **Qidirib topiladigan nom.** Bitta harfli o'zgaruvchilar faqat juda qisqa scope'da (masalan `for` loop indeksi `i`).
- **Sinf nomi — ot (noun):** `Customer`, `Account`. **Metod nomi — fe'l (verb):** `save()`, `deletePage()`.
- **Boolean nomlari savol kabi:** `isActive`, `hasPermission`, `canDelete`.
- **Bir tushunchaga bitta so'z.** `fetch`, `get`, `retrieve` aralashtirilmasin — bittasini tanla.

```ts
// ❌ Yomon
function getThem(theList: number[][]): number[][] {
  const list1: number[][] = [];
  for (const x of theList) {
    if (x[0] === 4) list1.push(x);
  }
  return list1;
}

// ✅ Yaxshi
const FLAGGED = 4;
function getFlaggedCells(gameBoard: Cell[]): Cell[] {
  return gameBoard.filter((cell) => cell.status === FLAGGED);
}
```

**💡 Tushuncha:** Magic number (`4`) o'rniga nomli konstanta (`FLAGGED`) qo'yish — naming va code smell tuzatishni bir vaqtda hal qiladi.

## Funksiyalar

Funksiyalar — kodning eng kichik tashkiliy birligi. Toza funksiya qoidalari:

- **Kichik bo'lsin.** Ideal — ekranga sig'sin (taxminan 20 qatorgacha).
- **Bitta ish qilsin (Single Responsibility).** Funksiya nomidagi "va" (`saveAndNotify`) ko'pincha ikkita funksiya kerakligini bildiradi.
- **Kam argument.** 0 (ideal), 1, 2 — yaxshi. 3 — ehtiyot bo'l. 4+ — deyarli har doim refactoring kerak.
- **Side-effect'siz bo'lishga intil.** Funksiya nomida aytilmagan o'zgarish qilmasin.
- **Bir abstraksiya darajasi.** Yuqori darajadagi logika va past darajadagi tafsilotlar bir funksiyada aralashmasin.

```ts
// ❌ Ko'p argument, side-effect yashiringan
function createUser(name: string, email: string, age: number, isAdmin: boolean) {
  // ... validatsiya, DB yozish, email yuborish — hammasi bir joyda
}

// ✅ Argumentlarni obyektga yig'ish + mas'uliyatni bo'lish
interface CreateUserInput {
  name: string;
  email: string;
  age: number;
  isAdmin: boolean;
}
function createUser(input: CreateUserInput): User {
  validateUser(input);
  const user = buildUser(input);
  return user;
}
```

**⚠️ Ehtiyot bo'l:** Boolean argument (flag argument) — funksiya kamida ikki ishni qilayotganini bildiradi: `render(true)` o'rniga `renderForPrint()` va `renderForScreen()`.

## Comment'lar

Eng yaxshi comment — yozish kerak bo'lmagan comment. Kod o'zi gapirsa, comment ortiqcha.

**Comment qachon kerak:**

- **Nega** shunday qilinganini tushuntirish (kod "qanday"ni, comment "nega"ni aytadi).
- Murakkab regex, algoritm yoki biznes qoidasining sababi.
- TODO/FIXME (vaqtinchalik, kuzatiladigan).
- Public API hujjati (JSDoc, docstring).
- Ogohlantirish: "Bu sekin, katta datada ishlatma".

**Comment qachon kerak emas:**

- Kod o'zi aytayotgan narsani takrorlash: `i++; // i ni oshir`.
- Eskirgan, kod bilan mos kelmaydigan comment (yolg'on comment).
- Commentlangan ("o'lik") kod — uni o'chirib tashla, git tarixda saqlanadi.

```ts
// ❌ Foydasiz comment
let d = 0; // elapsed time

// ✅ Kod o'zi gapiradi
let elapsedTimeInDays = 0;

// ✅ "Nega"ni tushuntiruvchi foydali comment
// To'lov provayderi 3 soniyadan keyin timeout beradi, shuning uchun retry 2s da.
const RETRY_DELAY_MS = 2000;
```

## DRY, KISS, YAGNI

Uchta asosiy prinsip:

- **DRY (Don't Repeat Yourself):** Har bir bilim (knowledge) tizimda bitta, aniq joyda bo'lsin. Takrorlangan kod — bug uchun ikki barobar joy.
- **KISS (Keep It Simple, Stupid):** Eng sodda yechimni tanla. Murakkablik — dushman.
- **YAGNI (You Aren't Gonna Need It):** "Kelajakda kerak bo'lar" deb funksiya qo'shma. Hozir kerak bo'lganini yoz.

**⚠️ Ehtiyot bo'l:** DRY'ni haddan oshirma. Ikki kod parchasi tasodifan o'xshash bo'lsa (lekin har xil sababga ega bo'lsa), ularni majburan birlashtirish noto'g'ri abstraksiya yaratadi. "Tasodifiy takror" va "haqiqiy takror"ni farqla. Bunga AHA (Avoid Hasty Abstractions) deyiladi.

## Code Smell'lar

Code smell — kod xato emas, lekin chuqurroq muammoga ishora qiluvchi belgi. Eng keng tarqalganlari:

- **Long Method:** Juda uzun funksiya. → Extract Method.
- **Large Class (God Object):** Hamma ishni qiladigan ulkan sinf. → Extract Class.
- **Duplicate Code:** Takrorlangan kod. → Extract Method/Function, DRY.
- **Magic Number/String:** Kontekstsiz literal qiymat. → Nomli konstanta.
- **Deep Nesting:** Chuqur `if`/`for` ichma-ichligi. → Guard clause, Early return.
- **Long Parameter List:** Ko'p argument. → Parameter Object.
- **Feature Envy:** Metod boshqa sinf ma'lumotini juda ko'p ishlatadi. → Move Method.
- **Primitive Obsession:** Hamma narsa `string`/`number`. → Domain tipi (`Money`, `Email`).

```ts
// ❌ Deep nesting + magic number
function getDiscount(user: User): number {
  if (user) {
    if (user.isActive) {
      if (user.yearsActive > 5) {
        return 0.2;
      }
    }
  }
  return 0;
}

// ✅ Guard clause + early return + nomli konstanta
const LOYAL_DISCOUNT = 0.2;
const LOYALTY_THRESHOLD_YEARS = 5;
function getDiscount(user: User): number {
  if (!user || !user.isActive) return 0;
  if (user.yearsActive <= LOYALTY_THRESHOLD_YEARS) return 0;
  return LOYAL_DISCOUNT;
}
```

## Refactoring Texnikalari

Refactoring — kodning **tashqi xatti-harakatini o'zgartirmasdan** ichki tuzilishini yaxshilash. Refactoring qilishdan oldin testlar bo'lishi shart.

Eng kerakli texnikalar:

- **Extract Method/Function:** Kod parchasini alohida nomli funksiyaga ajratish. Nom o'zi comment vazifasini bajaradi.
- **Guard Clause:** Funksiya boshida noto'g'ri holatlarni darrov qaytarib yuborish, asosiy logikani toza qoldirish.
- **Early Return:** `else` qatlamlarini kamaytirish uchun shartlar bajarilmasa darrov `return`.
- **Inline Variable / Introduce Variable:** Murakkab ifodani nomli o'zgaruvchiga ajratish.
- **Replace Magic Number with Constant.**
- **Extract Class:** Katta sinfni mantiqiy bo'laklarga bo'lish.

```ts
// ❌ Vergulli, nested
function pay(employee: Employee) {
  if (employee.isActive) {
    if (employee.hasBankAccount) {
      processPayment(employee);
    }
  }
}

// ✅ Guard clause bilan
function pay(employee: Employee) {
  if (!employee.isActive) return;
  if (!employee.hasBankAccount) return;
  processPayment(employee);
}
```

**💡 Tushuncha:** Refactoringni kichik qadamlar bilan qil va har qadamdan keyin testlarni ishlat. "Katta refactor" — eng xavfli refactor.

## Error Handling

Xatolarni qanday boshqarish — toza kodning muhim qismi. Ikki asosiy yondashuv: **exception** va **error code (qaytariladigan qiymat)**.

- **Exception (throw/try-catch):** Xato logikasini asosiy logikadan ajratadi. Kod toza qoladi.
- **Error code:** Har bir chaqiruvni tekshirishni talab qiladi, nested if'larga olib keladi. Ammo kontroldagi (kutilgan) holatlar uchun aniq bo'lishi mumkin (masalan `Result<T, E>` patterni).

Qoidalar:

- Exception'ni context bilan tashla: shunchaki `throw new Error()` emas, balki nima va nega.
- `null` qaytarish o'rniga exception tashla yoki bo'sh kolleksiya/Optional qaytar.
- `catch` qilib, hech narsa qilmaslik (swallow) — eng yomon amaliyot.

```ts
// ❌ Error code + null
function findUser(id: string): User | null {
  const user = db.query(id);
  if (!user) return null;
  return user;
}
// chaqiruvchi har safar null tekshirishi kerak...

// ✅ Exception bilan aniq xato
function findUser(id: string): User {
  const user = db.query(id);
  if (!user) {
    throw new UserNotFoundError(`User not found: ${id}`);
  }
  return user;
}
```

**⚠️ Ehtiyot bo'l:** Bo'sh `catch {}` bloki — xatoni yutib yuboradi va keyinchalik topib bo'lmaydigan bug yaratadi. Hech bo'lmaganda log qil yoki qayta tashla.

## Pure Function va Immutability

- **Pure function:** Bir xil input — har doim bir xil output, va hech qanday side-effect yo'q (global o'zgaruvchi, DB, I/O o'zgartirmaydi). Test qilish va tushunish oson.
- **Immutability:** Yaratilgandan keyin o'zgartirilmaydigan ma'lumot. Bug'larning katta qismi kutilmagan mutatsiyadan kelib chiqadi.

```ts
// ❌ Mutatsiya qiluvchi (impure)
function addItem(cart: Item[], item: Item): Item[] {
  cart.push(item); // tashqi massivni o'zgartiradi!
  return cart;
}

// ✅ Pure, immutable
function addItem(cart: readonly Item[], item: Item): Item[] {
  return [...cart, item]; // yangi massiv qaytaradi
}
```

**💡 Tushuncha:** Pure function va immutability — funksional dasturlashdan kelgan, lekin OOP'da ham qimmatli. Ular concurrency va testlashtirishni sezilarli osonlashtiradi.

## Boy Scout Rule

> "Leave the campground cleaner than you found it."

Boy Scout qoidasi: kod bilan ishlaganingda uni topganingdan **biroz toza** qoldir. Bitta nomni yaxshila, bitta funksiyani ajrat, bitta magic number'ni konstanta qil. Katta refactor talab qilinmaydi — kichik, doimiy yaxshilanishlar vaqt o'tishi bilan kodni sog'lom saqlaydi.

## Code Review

Code review — boshqa dasturchining o'zgarishini birlashtirishdan (merge) oldin ko'rib chiqish. Maqsadi: xatolarni topish, bilim almashish, jamoa standartini saqlash.

**Nimaga qarash kerak:**

- **To'g'rilik (correctness):** Kod kerakli ishni qiladimi, edge case'lar qoplanganmi?
- **O'qiluvchanlik:** Naming, tuzilish tushunarlimi?
- **Test:** Yangi logika test bilan qoplanganmi?
- **Xavfsizlik:** Input validatsiya, SQL injection, secrets ochiq qolmaganmi?
- **Dizayn:** O'zgarish loyiha arxitekturasiga mos keladimi, takror yo'qmi?

**Qanday feedback berish:**

- **Mehribon va aniq bo'l.** Odamga emas, kodga e'tibor: "bu funksiya..." (not "sen...").
- **Sababini tushuntir.** "Buni o'zgartir" emas, "bu null bo'lsa crash bo'ladi, shuning uchun...".
- **Muhimni mayda-chuydadan ajrat.** Nitpick'larni "nit:" deb belgila.
- **Maqtovni ham yoz.** Yaxshi yechimni ko'rsang, ayt.
- **Savol shaklida taklif qil.** "Bu yerda guard clause qanday bo'lardi?"

**💡 Tushuncha:** Code review — gatekeeping emas, hamkorlik. Maqsad — "men haqman"ni isbotlash emas, kodni birgalikda yaxshilash.

## Technical Debt

Technical debt (texnik qarz) — tezroq yetkazib berish uchun ataylab (yoki bilmasdan) tanlangan "qisqa yo'l"ning kelajakdagi narxi. Moliyaviy qarz kabi: foiz (interest) to'lanadi — har bir keyingi o'zgarish sekinroq bo'ladi.

Turlari:

- **Ataylab (deliberate):** "Deadline yaqin, hozir oddiy yechim, keyin tuzatamiz" — agar haqiqatan tuzatilsa, OK.
- **Bilmasdan (inadvertent):** Tajriba yetishmaganidan kelib chiqqan yomon dizayn.
- **Eskirgan (bit rot):** Kutubxonalar, talablar o'zgarib, kod eskiradi.

Texnik qarzni boshqarish: uni ko'rinadigan qil (backlog'ga yoz), foizini bahola, muntazam "qarz to'lash" (refactor) vaqtini ajrat.

**⚠️ Ehtiyot bo'l:** Hamma texnik qarz yomon emas. Bozorga tez chiqish uchun ongli qarz olish — strategik bo'lishi mumkin. Yomoni — qarzni e'tiborsiz qoldirish va foizini abadiy to'lash.

## Savol-Javoblar

### ❓ Clean Code nima va nega muhim?

**✅ Javob:** Clean Code — o'qish, tushunish va o'zgartirish oson bo'lgan kod. U muhim, chunki kod yozilishidan ko'p marta o'qiladi (taxminan 1:10 nisbat). Toza kod onboarding'ni tezlashtiradi, bug'larni kamaytiradi, o'zgarish narxini pasaytiradi va jamoa tezligini saqlaydi. Murakkab kod esa har bir keyingi o'zgarishni sekinlashtiradi.

### ❓ Yaxshi o'zgaruvchi nomining belgilari qanday?

**✅ Javob:** Maqsadni ochib beradi (intention-revealing), talaffuz va qidirib topiladigan, kontekstga mos. Sinf — ot, metod — fe'l, boolean — savol (`isActive`). Bir tushunchaga bitta so'z ishlatiladi (`get`/`fetch` aralashmasin). Bitta harfli nom faqat juda qisqa scope'da. Magic number o'rniga nomli konstanta.

### ❓ Yaxshi funksiya qanday bo'lishi kerak?

**✅ Javob:** Kichik (ideal 20 qatorgacha), bitta ish qiladi (SRP), kam argumentli (0-2 yaxshi), bir abstraksiya darajasida ishlaydi va side-effect'siz bo'lishga intiladi. Funksiya nomi uning nima qilishini aniq aytishi kerak. Boolean flag argument odatda funksiya ikki ishni qilayotganini bildiradi.

### ❓ Funksiyada nechta argument bo'lishi maqbul?

**✅ Javob:** 0 — ideal, 1-2 — yaxshi, 3 — ehtiyotkorlik talab qiladi, 4+ — deyarli har doim refactoring belgisi. Ko'p argumentni Parameter Object (bir obyektga yig'ish) bilan kamaytiriladi. Bu o'qiluvchanlikni va kengaytirilishni yaxshilaydi.

### ❓ Comment qachon yozish va qachon yozmaslik kerak?

**✅ Javob:** Comment "nega" qilinganini, murakkab biznes qoidasi yoki algoritm sababini, ogohlantirish va public API hujjatini tushuntirish uchun yoziladi. Kod o'zi aytayotgan narsani takrorlash, eskirgan comment va commentlangan o'lik kod — yozilmasligi kerak. Eng yaxshi comment — kerak bo'lmagan comment: kodni shunday yoz-ki, o'zi gapirsin.

### ❓ DRY, KISS, YAGNI nima?

**✅ Javob:** DRY — har bir bilim tizimda bitta joyda bo'lsin (takrorlama). KISS — eng sodda yechimni tanla. YAGNI — hozir kerak bo'lmagan funksiyani oldindan qo'shma. Bu prinsiplar murakkablikni va xatoni kamaytiradi. DRY'ni haddan oshirmaslik kerak: tasodifiy o'xshashlikni majburan birlashtirish noto'g'ri abstraksiya yaratadi (AHA prinsipi).

### ❓ Code smell nima? Misol keltiring.

**✅ Javob:** Code smell — kod xato emas, lekin chuqurroq muammoga ishora qiluvchi belgi. Misollar: Long Method, Large Class (God Object), Duplicate Code, Magic Number, Deep Nesting, Long Parameter List, Feature Envy, Primitive Obsession. Har biri tegishli refactoring texnikasi bilan tuzatiladi (masalan deep nesting → guard clause).

### ❓ Refactoring nima? Test bilan qanday bog'liq?

**✅ Javob:** Refactoring — tashqi xatti-harakatni o'zgartirmasdan ichki tuzilishni yaxshilash. Test'lar bo'lishi shart, chunki ular xatti-harakat o'zgarmaganini kafolatlaydi. Refactoring kichik qadamlarda, har qadamdan keyin testni ishlatib qilinadi. Test'siz refactoring — xavfli, chunki regressiyani sezmay qolish mumkin.

### ❓ Guard clause va early return nima foyda beradi?

**✅ Javob:** Ular noto'g'ri/chetki holatlarni funksiya boshida darrov qaytarib yuboradi, natijada chuqur `if/else` ichma-ichligini (deep nesting) kamaytiradi. Asosiy "baxtli yo'l" (happy path) kod toza va o'qiluvchan qoladi. Bu deep nesting code smell'ini hal qiladigan eng oddiy texnika.

### ❓ Exception va error code — qaysi biri yaxshi?

**✅ Javob:** Exception xato logikasini asosiy logikadan ajratadi, kodni toza qoldiradi va kutilmagan holatlar uchun mos. Error code (yoki `Result<T,E>` patterni) kutilgan, kontroldagi holatlar uchun aniqroq bo'lishi mumkin, lekin har chaqiruvni tekshirishni talab qiladi. Asosiy qoida: `null` qaytarib chalkashtirma, exception'ni context bilan tashla va hech qachon bo'sh `catch` bilan xatoni yutma.

### ❓ Pure function nima va nega foydali?

**✅ Javob:** Pure function — bir xil input har doim bir xil output beradigan va side-effect'siz funksiya. U test qilish oson (mock kerak emas), oldindan aytib bo'ladigan va concurrency uchun xavfsiz. Immutability bilan birga ular kutilmagan mutatsiyadan kelib chiqadigan bug'larni kamaytiradi.

### ❓ Boy Scout Rule nima?

**✅ Javob:** "Kodni topganingdan biroz toza qoldir." Kod bilan ishlaganda kichik yaxshilanishlar kirit: bitta nomni yaxshila, magic number'ni konstanta qil, kichik funksiyani ajrat. Bu katta refactor talab qilmaydi; doimiy kichik yaxshilanishlar kodning vaqt o'tishi bilan chirishini (rot) oldini oladi.

### ❓ Code review'da nimaga qarash kerak va qanday feedback beriladi?

**✅ Javob:** Qaraladigan narsalar: to'g'rilik va edge case'lar, o'qiluvchanlik, test qoplanishi, xavfsizlik, dizayn mosligi. Feedback mehribon va aniq bo'lishi, kodga (odamga emas) qaratilishi, sababini tushuntirishi kerak. Muhim muammolarni nitpick'lardan ajrat ("nit:"), savol shaklida taklif qil, yaxshi yechimni maqta. Code review — gatekeeping emas, hamkorlik.

### ❓ Technical debt nima va uni qanday boshqarasiz?

**✅ Javob:** Technical debt — tez yetkazish uchun tanlangan qisqa yo'lning kelajakdagi narxi; "foizi" har keyingi o'zgarishni sekinlashtiradi. Turlari: ataylab, bilmasdan va bit rot. Boshqarish: qarzni ko'rinadigan qil (backlog'ga yoz), foizini bahola, muntazam refactor vaqtini ajrat. Ataylab olingan strategik qarz yomon emas; yomoni — uni e'tiborsiz qoldirish.

### ❓ Single Responsibility Principle naming bilan qanday bog'liq?

**✅ Javob:** Agar funksiya yoki sinfga to'g'ri, qisqa nom topa olmasangiz (yoki nomda "And"/"Manager"/"Util" paydo bo'lsa), bu ko'pincha u bir nechta mas'uliyatga ega ekanini bildiradi. Yaxshi nom topishning qiyinligi — SRP buzilganining belgisi. Mas'uliyatni bo'lsangiz, nomlar ham tabiiy ravishda aniqlashadi.

### ❓ "Premature optimization" va clean code o'rtasida ziddiyat bormi?

**✅ Javob:** Yo'q. Avval toza, sodda va to'g'ri kod yoziladi (KISS). Optimizatsiya faqat o'lchangan (profiled) haqiqiy bottleneck uchun qilinadi. Premature optimization — KISS va YAGNI'ga zid, chunki u kodni murakkablashtiradi va ko'pincha keraksiz joyni tezlashtiradi. "Avval to'g'ri qil, keyin tez qil."

## Masalalar

> Yechimlar: [yechimlarni ko'rish](../solutions/oop-patterns/03-clean-code.md)

1. **Naming refactor.** Quyidagi funksiyani toza qiling: o'zgaruvchi va funksiya nomlarini mazmunli qiling, magic number'ni nomli konstantaga aylantiring.
   ```ts
   function p(u: any[]): number {
     let r = 0;
     for (const x of u) {
       if (x.t > 30) r += x.a * 0.9;
     }
     return r;
   }
   ```

2. **Guard clause.** Quyidagi deep nested funksiyani guard clause va early return bilan qayta yozing:
   ```ts
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
   ```

3. **Long parameter list.** Quyidagi funksiyani Parameter Object yordamida refactor qiling:
   ```ts
   function createOrder(productId: string, qty: number, userId: string,
     address: string, city: string, zip: string, isGift: boolean) { /* ... */ }
   ```

4. **DRY.** Quyidagi takrorlangan kodni Extract Function bilan birlashtiring (lekin AHA prinsipini hisobga oling — agar takror tasodifiy bo'lsa, izohlab bering):
   ```ts
   const fullName = user.firstName + " " + user.lastName;
   // boshqa joyda:
   const label = customer.firstName + " " + customer.lastName;
   ```

5. **Side-effect tozalash.** Quyidagi impure funksiyani pure va immutable qiling:
   ```ts
   let total = 0;
   function addToTotal(value: number): void {
     total += value;
   }
   ```

6. **Error handling.** `null` qaytaradigan quyidagi funksiyani exception bilan qayta yozing va chaqiruvchi kodga ta'sirini izohlang:
   ```ts
   function getConfig(key: string): string | null {
     return store[key] ?? null;
   }
   ```

7. **Code smell aniqlash.** Quyidagi sinfdagi kamida 3 ta code smell'ni nomlang va har biri uchun refactoring texnikasini taklif qiling:
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

8. **Code review feedback.** Sizga PR berildi: funksiya 80 qatorli, 5 argumentli, bo'sh `catch {}` bloki bor va testsiz. Mehribon, aniq va sababli 4 ta code review kommentariyasi yozing.

---

← [OOP bo'limiga qaytish](./README.md)
