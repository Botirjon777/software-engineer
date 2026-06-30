# Testing Asoslari

Bu fayl test yozishning nima uchun kerakligini (regression, refactoring ishonchi, dokumentatsiya, dizayn), test turlarini va test pyramid'ni, AAA pattern'ni, TDD/BDD sikllarini, test double'larni (dummy, stub, spy, mock, fake), mocking qachon o'rinli ekanini, code coverage'ning haqiqiy ma'nosini va yaxshi test xossalarini (FIRST) boshidan tushuntiradi. Maqsad: intervyuda "nega test yozamiz?" degan oddiy savolga ham, "stub va mock farqi nima?" degan nozik savolga ham ishonch bilan javob bera olish. Misollar JS/TS (Jest/Vitest).

## Mundarija

- [Tushuncha: test nima va nega yozamiz](#tushuncha-test-nima-va-nega-yozamiz)
- [Test pyramid: unit, integration, e2e](#test-pyramid-unit-integration-e2e)
- [Unit test va AAA pattern](#unit-test-va-aaa-pattern)
- [Integration test](#integration-test)
- [End-to-end (e2e) test](#end-to-end-e2e-test)
- [TDD: red-green-refactor](#tdd-red-green-refactor)
- [BDD qisqacha](#bdd-qisqacha)
- [Test double'lar: dummy, stub, spy, mock, fake](#test-doublelar-dummy-stub-spy-mock-fake)
- [Mocking: qachon va qachon emas](#mocking-qachon-va-qachon-emas)
- [Code coverage va 100% xayoli](#code-coverage-va-100-xayoli)
- [Nimani test qilish kerak va nimani emas](#nimani-test-qilish-kerak-va-nimani-emas)
- [FIRST: yaxshi test xossalari](#first-yaxshi-test-xossalari)
- [Flaky test](#flaky-test)
- [Intervyu savollari (Q&A)](#intervyu-savollari-qa)
- [Masalalar](#masalalar)

---

## Tushuncha: test nima va nega yozamiz

**💡 Tushuncha:** **Test** — bu kodingiz *kutilgandek* ishlashini avtomatik tekshiruvchi yana bir kod. Siz funksiyaga ma'lum kirish (input) berasiz, natijani (output) kutilgan qiymat bilan solishtirasiz. Tekshiruv mos kelsa test "yashil" (pass), kelmasa "qizil" (fail) bo'ladi. Asosiy g'oya: tekshiruvni odam qo'lda emas, kompyuter har safar takrorlab beradi.

Nega test yozamiz — to'rtta asosiy sabab:

- **Regression himoyasi.** Yangi kod yozganda eski ishlayotgan narsani buzib qo'yish — eng ko'p uchraydigan xato. Testlar shu buzilishni siz prodga chiqarmasdan oldin tutadi.
- **Refactoring ishonchi.** Kodni qayta yozish (yaxshilash) qo'rqinchli, chunki "buzib qo'ymadimmi?" degan savol turadi. Yaxshi test to'plami "xulq-atvor o'zgarmadi" deb kafolat beradi — siz ichki tuzilmani bemalol o'zgartirasiz.
- **Dokumentatsiya.** Yaxshi yozilgan test funksiyaning *qanday ishlatilishi kerakligini* ko'rsatadi. Test nomi (`it('throws if amount is negative')`) — eng to'g'ri va eskirmaydigan hujjat.
- **Dizayn bosimi.** Test yozish qiyin bo'lgan kod odatda yomon dizaynli kod (juda ko'p bog'liqlik, yashirin holat). Test yozish jarayoni sizni toza, bo'lakli (modular) dizaynga undaydi.

```js
// Tekshirilayotgan kod (production code)
function add(a, b) {
  return a + b;
}

// Test
test('add() ikki sonni qo\'shadi', () => {
  expect(add(2, 3)).toBe(5);
});
```

**⚠️ Ehtiyot bo'l:** Test — "xatosiz kod" kafolati emas. Test faqat *siz o'ylab topgan* holatlarni tekshiradi. O'ylab topmagan holat (edge case) baribir buzilishi mumkin. Test ishonchni oshiradi, lekin 100% to'g'rilikni isbotlamaydi.

---

## Test pyramid: unit, integration, e2e

**💡 Tushuncha:** **Test pyramid** — testlarni qaysi nisbatda yozish kerakligini ko'rsatuvchi metafora. Pastda — ko'p va arzon **unit** testlar, o'rtada — kamroq **integration**, eng tepada — eng kam va eng qimmat **e2e** testlar. Pastdan tepaga: testlar *kamayadi*, *sekinlashadi*, *qimmatlashadi*, lekin *ishonch* (haqiqiy tizimga yaqinlik) *ortadi*.

```text
              /\
             /e2e\          kam, sekin, qimmat, eng ishonchli
            /------\
           /  inte- \
          / gration  \      o'rtacha
         /------------\
        /     unit     \    ko'p, tez, arzon, izolyatsiyalangan
       /----------------\
```

- **Unit** — bitta funksiya/klassni izolyatsiyada (bog'liqliklarsiz) tekshiradi. Millisekundlar ichida ishlaydi.
- **Integration** — bir nechta qism birga ishlaganini tekshiradi (masalan, kod + haqiqiy database).
- **E2e** — butun tizimni foydalanuvchi nuqtai nazaridan, brauzer orqali tekshiradi.

Asosiy **trade-off**: tezlik vs ishonch. Unit test tez, lekin "qismlar bir-biriga to'g'ri ulanganmi?"ni bilmaydi. E2e test ishonchli ("haqiqatan ishlaydi"), lekin sekin va mo'rt (flaky). Pyramid shakli — bu ikkisi orasidagi muvozanat.

**⚠️ Ehtiyot bo'l:** Teskari shakl — **"ice cream cone"** (muzqaymoq koni) — anti-pattern: ko'p e2e, kam unit. Bunda test to'plami sekin ishlaydi, tez-tez asossiz qiziradi (flaky) va xato qayerdaligini aniqlash qiyin bo'ladi.

---

## Unit test va AAA pattern

**💡 Tushuncha:** **Unit test** — kodning eng kichik mantiqiy bo'lagini (odatda funksiya yoki metod) *boshqa hammasidan ajratib*, faqat shuning o'zini tekshiradi. Tashqi bog'liqliklar (DB, network, vaqt) mavjud bo'lsa, ular test double bilan almashtiriladi. Shu sababli unit test tez va deterministik bo'ladi.

Yaxshi unit test **AAA pattern** bo'yicha tuziladi:

- **Arrange** — tayyorgarlik: ma'lumot, obyekt, mock'larni sozlaysiz.
- **Act** — harakat: tekshirilayotgan funksiyani chaqirasiz.
- **Assert** — tasdiq: natijani kutilgan bilan solishtirasiz.

```js
test('discount 20% ni to\'g\'ri qo\'llaydi', () => {
  // Arrange
  const price = 100;
  const percent = 20;

  // Act
  const result = applyDiscount(price, percent);

  // Assert
  expect(result).toBe(80);
});
```

Bu uch bosqichni vizual ajratish testni o'qishni osonlashtiradi: nima berildi, nima qilindi, nima kutilyapti. Bir testda faqat bitta mantiqiy narsani tekshirishga harakat qiling — shunda fail bo'lganda sabab darrov ma'lum bo'ladi.

**⚠️ Ehtiyot bo'l:** Bir testda ko'p `Act` bo'lsa (bir necha funksiyani ketma-ket chaqirish va oradagi natijalarni tekshirish), bu odatda testni bo'lish kerakligining belgisi. Bunday test fail bo'lsa, aynan qaysi qadam buzilganini topish qiyin.

---

## Integration test

**💡 Tushuncha:** **Integration test** — bir nechta birlik (unit) *birga* to'g'ri ishlashini tekshiradi. Unit test "mening funksiyam to'g'ri hisoblaydimi?" desa, integration test "mening kodim haqiqiy database bilan to'g'ri gaplashadimi?", "service A va service B to'g'ri ulanganmi?" deb so'raydi.

Unit test ko'pincha bog'liqlikni *mock* qiladi; integration test esa, aksincha, *haqiqiy* (yoki haqiqiyga yaqin) bog'liqlikni ishlatadi — masalan, test uchun ko'tarilgan haqiqiy Postgres yoki in-memory DB.

```ts
// Integration test: repository + haqiqiy (test) database
describe('UserRepository', () => {
  let db: Database;

  beforeAll(async () => {
    db = await connectTestDb(); // haqiqiy test DB
  });

  afterAll(() => db.close());

  it('user yaratadi va o\'qiydi', async () => {
    const repo = new UserRepository(db);
    const created = await repo.create({ name: 'Ali' });

    const found = await repo.findById(created.id);

    expect(found?.name).toBe('Ali');
  });
});
```

Integration test ko'p xatolarni ushlaydi: noto'g'ri SQL, sxema mosligi, serializatsiya muammolari — bularni unit test (DB mock qilingani uchun) ko'rmaydi.

**⚠️ Ehtiyot bo'l:** Integration test holat (state) qoldiradi — bir test yaratgan ma'lumot keyingisiga ta'sir qilishi mumkin. Har testdan oldin/keyin DB'ni tozalang (`beforeEach` da reset), aks holda testlar bir-biriga bog'liq va flaky bo'ladi.

---

## End-to-end (e2e) test

**💡 Tushuncha:** **E2e (end-to-end) test** butun tizimni foydalanuvchi nuqtai nazaridan tekshiradi: haqiqiy brauzer ochiladi, tugmalar bosiladi, formalar to'ldiriladi va natija ekrandan o'qiladi — xuddi haqiqiy odam kabi. U frontend + backend + DB + network — hammasini *birga* sinaydi.

```ts
// Playwright e2e misoli
test('foydalanuvchi tizimga kiradi', async ({ page }) => {
  await page.goto('https://app.example.uz/login');
  await page.fill('input[name="email"]', 'ali@example.uz');
  await page.fill('input[name="password"]', 'secret123');
  await page.click('button[type="submit"]');

  await expect(page.getByText('Xush kelibsiz, Ali')).toBeVisible();
});
```

E2e test eng yuqori ishonch beradi ("foydalanuvchi haqiqatan tizimga kira oladi"), lekin: sekin (brauzer, network), qimmat (yozish va saqlash), va eng flaky (timing, tarmoq, animatsiya). Shuning uchun ularni faqat eng muhim "happy path" stsenariylar uchun yozing.

**⚠️ Ehtiyot bo'l:** Hamma narsani e2e bilan tekshirishga urinish — keng tarqalgan xato. Validatsiya qoidalarining har bir varianti (10 ta xato xabari) unit testda tekshirilishi kerak, e2e da emas — aks holda test to'plami soatlab ishlaydi.

---

## TDD: red-green-refactor

**💡 Tushuncha:** **TDD (Test-Driven Development)** — testni koddan *oldin* yozish uslubi. Sikl uch qadamdan iborat: **Red** (avval fail bo'ladigan test yoz), **Green** (testni o'tkazadigan eng oddiy kodni yoz), **Refactor** (testlar yashil turganda kodni toza qil).

```text
   ┌─────────────────────────────────────────┐
   │                                         ▼
RED (test yoz,  →  GREEN (minimal kod,  →  REFACTOR (tozalash,
fail bo'lsin)       pass bo'lsin)            test yashil qolsin)
   ▲                                         │
   └─────────────────────────────────────────┘
```

Misol — `isEven` funksiyasini TDD bilan:

```js
// 1. RED — funksiya hali yo'q, test fail
test('2 juft son', () => {
  expect(isEven(2)).toBe(true);
});

// 2. GREEN — eng oddiy kod
function isEven(n) {
  return n % 2 === 0;
}

// 3. REFACTOR — bu yerda tozalash shart emas, kod allaqachon toza
```

TDD foydasi: har bir qator kod test bilan qoplanadi; dizayn "ishlatish nuqtai nazaridan" boshlanadi (avval API'ni o'ylaysiz); ortiqcha kod yozilmaydi (faqat testni o'tkazish uchun yetarli).

**⚠️ Ehtiyot bo'l:** TDD — qoida emas, vosita. Murakkab algoritmda foydali, oddiy CRUD'da ortiqcha bo'lishi mumkin. "Test koddan oldin" — maqsad emas, balki yaxshi dizayn va qamrov uchun usul.

---

## BDD qisqacha

**💡 Tushuncha:** **BDD (Behavior-Driven Development)** — TDD'ning kengaytmasi bo'lib, e'tiborni *texnik tekshiruv*dan *xulq-atvor (behavior)* ga ko'chiradi. Test nomlari biznes tili bilan, "given-when-then" tarzida yoziladi, shunda ularni dasturchi bo'lmagan odam ham o'qiy oladi.

```text
Given (berilgan)  — savatda 100$ lik mahsulot bor
When  (qachonki)  — foydalanuvchi 20% promo-kod kiritsa
Then  (u holda)   — yakuniy summa 80$ bo'ladi
```

Jest/Vitest'da `describe`/`it` bloklari aslida BDD uslubidan kelib chiqqan — `it('should ...')` "u ... qilishi kerak" degan jumla hosil qiladi. Cucumber/Gherkin esa Given-When-Then'ni alohida `.feature` faylida yozadi.

**⚠️ Ehtiyot bo'l:** BDD asboblari (Cucumber, Gherkin) qatlam qo'shadi va kichik jamoada ortiqcha bo'lishi mumkin. Ko'pincha oddiygina `describe`/`it` ni yaxshi nomlar bilan yozishning o'zi BDD ruhini beradi.

---

## Test double'lar: dummy, stub, spy, mock, fake

**💡 Tushuncha:** **Test double** — haqiqiy bog'liqlik o'rniga testda ishlatiladigan "dublyor" obyekt (kino dublyori kabi). Beshta asosiy tur bor va ular tez-tez chalkashtiriladi. Farqni bilish intervyuda kuchli signal.

| Tur | Nima qiladi | Misol |
|-----|-------------|-------|
| **Dummy** | Hech nima qilmaydi, faqat parametr to'ldirish uchun uzatiladi | `new Service(null, dummyLogger)` |
| **Stub** | Oldindan tayyorlangan javob qaytaradi | `getUser` har doim `{id:1}` qaytarsin |
| **Spy** | Haqiqiy ishni bajaradi, lekin chaqiruvlarni yozib boradi | "`save` necha marta chaqirildi?" |
| **Mock** | Oldindan *kutilgan chaqiruvlar* belgilanadi, ular tekshiriladi | "`send` aynan shu argument bilan chaqirilishi shart" |
| **Fake** | Soddalashtirilgan, lekin ishlaydigan implementatsiya | in-memory database |

```js
// STUB — javobni belgilab beradi
const userStub = { getName: () => 'Ali' };

// SPY — chaqiruvni kuzatadi, lekin xatti-harakatni o'zgartirmaydi
const logger = { log: jest.fn() };
doWork(logger);
expect(logger.log).toHaveBeenCalled();

// MOCK — kutilgan o'zaro ta'sirni tasdiqlaydi
const mailer = { send: jest.fn() };
notifyUser(mailer, 'ali@example.uz');
expect(mailer.send).toHaveBeenCalledWith('ali@example.uz', expect.any(String));

// FAKE — ishlaydigan, lekin soddalashtirilgan
class FakeUserRepo {
  #data = new Map();
  save(u) { this.#data.set(u.id, u); }
  findById(id) { return this.#data.get(id); }
}
```

Asosiy farq — **state verification vs behavior verification**: stub/fake "natija to'g'rimi?"ni tekshiradi (state), mock "to'g'ri chaqiruv bo'ldimi?"ni tekshiradi (behavior).

**⚠️ Ehtiyot bo'l:** "Mock" so'zi amalda hamma turlarga nisbatan ishlatiladi (Jest `jest.fn()` ham mock, ham spy bo'la oladi). Intervyuda aniq farqni bilsangiz ham, kundalik nutqda hamma "mock" deydi — bu normal, lekin farqni tushunib turing.

---

## Mocking: qachon va qachon emas

**💡 Tushuncha:** **Mocking** — haqiqiy bog'liqlikni soxta (boshqariladigan) versiya bilan almashtirish. Maqsad: testni tez, deterministik va izolyatsiyalangan qilish. Lekin mock'ni ko'p ishlatish testni haqiqatdan uzoqlashtiradi, shuning uchun *qachon* mock qilishni bilish muhim.

**Mock qilish o'rinli bo'lgan holatlar:**

- **Tashqi bog'liqlik** — network so'rovi (`fetch`), tashqi API. Testda haqiqiy so'rov yubormaysiz: sekin, ishonchsiz, pulga tushadi.
- **Vaqt** — `Date.now()`, `setTimeout`. Testning bugungi sanaga bog'liq bo'lishi flaky'lik manbai. Vaqtni "muzlatish" kerak.
- **Tasodif** — `Math.random()`. Deterministik bo'lishi uchun stub qilinadi.
- **Yon ta'sirlar** — email yuborish, to'lov olish. Testda haqiqiy email yubormaysiz.

```js
// Vaqtni mock qilish (Jest)
jest.useFakeTimers();
jest.setSystemTime(new Date('2026-01-01'));

expect(getYear()).toBe(2026);

jest.useRealTimers();
```

**Mock qilmaslik kerak bo'lgan holatlar:**

- O'z kodingizning oddiy, sof (pure) funksiyalari — ularni to'g'ridan-to'g'ri ishlating.
- Mock'ni *implementatsiya detallariga* bog'lab qo'yish — ichki metod necha marta chaqirilganini tekshirish kodni o'zgartirsangiz darrov sinadi (brittle test).

**⚠️ Ehtiyot bo'l:** **Over-mocking** — hamma narsani mock qilish — eng katta tuzoq. Agar testingiz faqat mock'lar bilan ishlasa, u "mening mock'larim mening mock'larimga to'g'ri keladimi?"ni tekshiradi, haqiqiy kodni emas. Mock'ni faqat chegarada (boundary) ishlatib, ichkarini haqiqiy qoldiring.

---

## Code coverage va 100% xayoli

**💡 Tushuncha:** **Code coverage** — testlar ishga tushganda kodning qancha qismi *bajarilganini* o'lchaydigan metrika (foizda). Asosiy turlari: **line coverage** (qaysi qatorlar bajarildi) va **branch coverage** (har bir `if`/`else` shoxining ikkala tomoni ham sinaldimi). Branch coverage line'dan kuchliroq.

```js
function classify(n) {
  if (n > 0) return 'positive';   // shox 1
  else return 'non-positive';     // shox 2
}

// Bu test 100% LINE coverage beradi, lekin faqat 50% BRANCH:
test('positive', () => {
  expect(classify(5)).toBe('positive'); // "else" shoxi sinalmadi!
});
```

Yuqoridagi misol asosiy fikrni ko'rsatadi: **coverage to'g'rilikni isbotlamaydi**. 100% coverage — bu "har bir qator *bajarildi*" degani, "har bir qator *to'g'ri ishlaydi*" degani emas. `expect` bo'lmagan test ham coverage'ni oshiradi, lekin hech narsani tekshirmaydi.

100% coverage talab qilish ko'pincha zararli: dasturchilar ma'nosiz testlar yozib, raqamni "majburlay" boshlaydi. Ko'p loyihalarda 70–85% maqbul — qolgan qism (oddiy getter, konfiguratsiya, log) test qilishga arzimaydi.

**⚠️ Ehtiyot bo'l:** Coverage — *yo'naltiruvchi* metrika, *maqsad* emas. "Qaysi muhim qism qoplanmagan?"ni topish uchun foydali, lekin "100% bo'ldi, demak kod xatosiz" — yolg'on xulosa. Goodhart qonuni: metrika maqsadga aylansa, u yaxshi metrika bo'lishdan to'xtaydi.

---

## Nimani test qilish kerak va nimani emas

**💡 Tushuncha:** Hamma narsani test qilish — ham vaqt, ham keyinchalik qiyinchilik. To'g'ri tanlov: **xulq-atvor (behavior) ni test qiling, implementatsiyani emas.** Ya'ni "funksiya nima qaytaradi" muhim, "u buni qanday hisoblaydi" emas.

**Test qilishga arzigan:**

- Biznes logika, hisob-kitoblar, qoidalar (discount, soliq, validatsiya).
- Edge case'lar (bo'sh massiv, manfiy son, `null`, juda katta qiymat).
- Avval xato chiqqan joylar (har topilgan bug uchun avval uni ushlovchi test yozing — "regression test").
- Murakkab shartlar va kombinatsiyalar.

**Test qilishga arzimaydigan (yoki kam):**

- Til/framework'ning o'zini (massiv `.map` ishlashini tekshirish — ortiqcha).
- Oddiy getter/setter va trivial kod.
- Tez-tez o'zgaradigan implementatsiya detallari (ichki o'zgaruvchi nomi, private metod chaqiruvi soni).
- Tashqi kutubxonaning ichki ishi (uni o'zlari sinaydi).

```js
// YOMON — implementatsiyaga bog'langan test
expect(cart._internalItems.length).toBe(2); // private holat

// YAXSHI — xulq-atvorni tekshiruvchi test
expect(cart.getTotal()).toBe(150);
```

**⚠️ Ehtiyot bo'l:** Implementatsiyani test qilish testni "mo'rt" qiladi: refactoring qilganingizda (xulq o'zgarmasa ham) test sinadi. Bu testning asosiy foydasini — refactoring ishonchini — yo'qqa chiqaradi.

---

## FIRST: yaxshi test xossalari

**💡 Tushuncha:** **FIRST** — yaxshi unit testning beshta xossasini eslatuvchi akronim. Bu xossalar testni ishonchli va foydali qiladi.

- **Fast (tez)** — testlar millisekundlarda ishlashi kerak, shunda ularni tez-tez (har saqlashda) ishga tushirasiz. Sekin test — kam ishlatiladigan test.
- **Independent (mustaqil)** — har bir test boshqasidan ajralgan; tartibga bog'liq bo'lmasligi kerak. Test A test B yaratgan holatga tayanmasligi shart.
- **Repeatable (takrorlanuvchi)** — qaysi muhitda, necha marta ishga tushirsangiz ham bir xil natija. Vaqt, tarmoq, tasodifga bog'liq bo'lmasligi kerak.
- **Self-validating (o'zini-tasdiqlovchi)** — test o'zi "pass/fail" deb javob beradi; natijani odam qo'lda tekshirib o'tirmaydi (console.log'ga qarab emas).
- **Timely (o'z vaqtida)** — test kod bilan birga (ko'pincha undan oldin, TDD'da) yoziladi, "keyinroq" emas — "keyinroq" amalda "hech qachon" degani.

```js
// "Independent" buzilgan misol — testlar global holatga bog'liq
let counter = 0;
test('a', () => { counter++; expect(counter).toBe(1); });
test('b', () => { counter++; expect(counter).toBe(2); }); // tartibga bog'liq!

// To'g'risi — har test o'z holatini yaratadi (beforeEach yoki lokal)
```

**⚠️ Ehtiyot bo'l:** "Independent"ni buzadigan eng keng tarqalgan sabab — testlar orasida bo'lishiladigan o'zgaruvchan (mutable) holat. Har test uchun `beforeEach` da yangi obyekt yarating, holatni reset qiling.

---

## Flaky test

**💡 Tushuncha:** **Flaky test** — kod o'zgarmasa ham *ba'zan o'tadigan, ba'zan yiqiladigan* test. U eng zararli test turi, chunki ishonchni yo'qotadi: jamoa "yana o'sha flaky test" deb fail'larga e'tibor bermay qo'yadi va haqiqiy bug ham e'tibordan chetda qoladi.

Flaky'likning asosiy sabablari:

- **Vaqtga bog'liqlik** — `Date.now()`, `setTimeout` bilan yarish (race). Vaqtni mock qiling.
- **Tartib bog'liqligi** — testlar bo'lishilgan holatga tayanadi. Izolyatsiya qiling.
- **Async timing** — promise tugashini kutmasdan assert qilish, qattiq `sleep(100)` ishlatish. To'g'ri `await` va "kutish" (waitFor) ishlating, qattiq vaqt emas.
- **Tashqi resurs** — haqiqiy network, real vaqt DB. Mock yoki test-konteyner ishlating.
- **Tasodif** — `Math.random()` ni seed qiling yoki stub qiling.

```js
// FLAKY — qattiq sleep, timing'ga umid
test('flaky', async () => {
  startJob();
  await sleep(100);            // ba'zan yetarli emas!
  expect(jobDone()).toBe(true);
});

// BARQAROR — natija paydo bo'lishini kutamiz
test('barqaror', async () => {
  startJob();
  await waitFor(() => jobDone() === true);
  expect(jobDone()).toBe(true);
});
```

**⚠️ Ehtiyot bo'l:** Flaky testni "shunchaki qayta ishga tushirib" e'tiborsiz qoldirish — qarz to'plash. Flaky test topilsa, uni darhol tuzating yoki vaqtincha o'chiring (skip), lekin "ko'rinmasin" deb retry'ga tashlamang — bu muammoni yashiradi, hal qilmaydi.

---

## Intervyu savollari (Q&A)

### ❓ Nega umuman test yozamiz, qo'lda tekshirsa bo'lmaydimi?

**✅ Javob:** Qo'lda tekshiruv bir martalik va takrorlanmaydi; kod o'sgani sayin har o'zgarishdan keyin hammasini qayta sinash imkonsiz bo'ladi. Avtomatik testlar regressiyani ushlaydi, refactoringga ishonch beradi, dokumentatsiya bo'lib xizmat qiladi va yaxshi dizaynga undaydi. Asosiysi — ular sekundlarda takrorlanadi.

### ❓ Test pyramid nima va nega aynan shu shaklda?

**✅ Javob:** Ko'p tez/arzon unit, kamroq integration, eng kam sekin/qimmat e2e. Shakl tezlik vs ishonch trade-off'ini muvozanatlaydi: pastda tezlik, tepada haqiqiyga yaqinlik. Teskari ("ice cream cone") shakl — ko'p e2e — sekin, flaky va xatoni topish qiyin bo'lgan to'plamga olib keladi.

### ❓ AAA pattern nima?

**✅ Javob:** Arrange-Act-Assert — testni uch bosqichga ajratish: tayyorlash (ma'lumot, mock), harakat (funksiyani chaqirish), tasdiq (natijani solishtirish). Bu test o'qishni osonlashtiradi va bir testda bir narsani tekshirishga undaydi.

### ❓ Unit va integration test farqi nimada?

**✅ Javob:** Unit bitta bo'lakni izolyatsiyada (bog'liqliklarni mock qilib) sinaydi — tez va deterministik. Integration bir nechta qism birga (ko'pincha haqiqiy DB bilan) ishlashini tekshiradi — sekinroq, lekin ulanish/SQL/sxema kabi xatolarni ushlaydi.

### ❓ E2e test qachon kerak va uning kamchiliklari nima?

**✅ Javob:** E2e butun tizimni foydalanuvchi nuqtai nazaridan tekshiradi (brauzer orqali) va eng yuqori ishonch beradi. Kamchiliklari: sekin, qimmat, eng flaky. Shuning uchun faqat eng muhim "happy path" stsenariylar uchun yoziladi, har bir edge case uchun emas.

### ❓ TDD siklini tushuntiring.

**✅ Javob:** Red-Green-Refactor: avval fail bo'ladigan test yoz (red), uni o'tkazadigan minimal kod yoz (green), keyin testlar yashil turganda kodni tozala (refactor). Foydasi: to'liq qamrov, ishlatishga yo'naltirilgan dizayn, ortiqcha kod yozilmasligi.

### ❓ Stub va mock farqi nima?

**✅ Javob:** Stub — oldindan tayyor javob qaytaradi va biz *natijani* (state) tekshiramiz. Mock — kutilgan *chaqiruvlarni* (qaysi argument bilan, necha marta) belgilaymiz va o'zaro ta'sirni (behavior) tekshiramiz. Stub "nima qaytdi", mock "qanday chaqirildi"ga e'tibor beradi.

### ❓ Spy va mock farqi nima?

**✅ Javob:** Spy haqiqiy xatti-harakatni saqlab, ustidan chaqiruvlarni *yozib boradi* (keyin tekshirish uchun). Mock esa oldindan kutilgan chaqiruvlarni *belgilab qo'yadi* va ularni majburlaydi. Spy "kuzatuvchi", mock "talab qo'yuvchi". Amalda Jest'ning `jest.fn()` ikkala rolni ham bajara oladi.

### ❓ Qachon mock qilish kerak?

**✅ Javob:** Tashqi va boshqarib bo'lmaydigan bog'liqliklarda: network/API, vaqt (`Date.now`), tasodif (`Math.random`), yon ta'sirli amallar (email, to'lov). Maqsad — testni tez, deterministik qilish. O'z sof funksiyalaringizni va ichki implementatsiyani mock qilmaslik kerak.

### ❓ Over-mocking nima va nega yomon?

**✅ Javob:** Hamma narsani mock qilish. Bunda test haqiqiy kod o'rniga mock'lar bilan ishlaydi va aslida "mock'lar mock'larga mos keladimi"ni tekshiradi. Natijada test o'tadi, lekin haqiqiy tizim buzilishi mumkin. Mock'ni faqat chegarada ishlating.

### ❓ 100% code coverage kafolatmi?

**✅ Javob:** Yo'q. Coverage faqat qator/shox *bajarilganini* o'lchaydi, *to'g'ri ishlaganini* emas. `expect`siz test ham coverage'ni oshiradi. Branch coverage line'dan kuchliroq, lekin baribir to'g'rilik isboti emas. Coverage yo'naltiruvchi metrika, maqsad emas.

### ❓ FIRST printsiplari nima?

**✅ Javob:** Fast (tez), Independent (mustaqil), Repeatable (takrorlanuvchi), Self-validating (o'zini-tasdiqlovchi), Timely (o'z vaqtida). Bu xossalar testni ishonchli va tez-tez ishlatiladigan qiladi.

### ❓ Flaky test nima va u bilan nima qilish kerak?

**✅ Javob:** Kod o'zgarmasa ham ba'zan o'tib, ba'zan yiqiladigan test. Sabablari: vaqt/timing, tartib bog'liqligi, async race, tashqi resurs, tasodif. Yechim — sabab manbasini izolyatsiya qilish (vaqtni mock, holatni reset, to'g'ri await). Uni retry'ga tashlab yashirmaslik kerak.

### ❓ Nimani test qilmaslik kerak?

**✅ Javob:** Til/framework'ning o'zini, trivial getter/setterlarni, tez o'zgaradigan implementatsiya detallarini va tashqi kutubxonalarning ichki ishini. E'tiborni biznes logika, edge case va avval bug chiqqan joylarga qarating.

### ❓ Test "implementatsiyaga bog'langan" deganda nima nazarda tutiladi?

**✅ Javob:** Test ko'rinadigan xulq o'rniga ichki tafsilotlarga (private holat, metod chaqiruvlari soni) tayansa. Bunday test refactoring qilinganda — xulq o'zgarmasa ham — sinadi, va shu bilan testning asosiy foydasini yo'qotadi. Xulq-atvorni test qiling.

### ❓ BDD nima va TDD'dan farqi?

**✅ Javob:** BDD — TDD'ning xulq-atvorga yo'naltirilgan ko'rinishi: testlar biznes tilida, Given-When-Then tarzida yoziladi, shunda dasturchi bo'lmaganlar ham o'qiy oladi. TDD texnik tekshiruvga, BDD esa kutilgan xatti-harakatga urg'u beradi. `describe`/`it('should...')` aslida BDD uslubidan kelgan.

---

## Masalalar

> Yechimlar: [solutions/testing/01-testing-fundamentals.md](../solutions/testing/01-testing-fundamentals.md)

1. **AAA bilan test yozish.** `applyDiscount(price, percent)` funksiyasi uchun (foiz 0–100 oralig'ida bo'lishi kerak, aks holda xato tashlasin) AAA pattern'iga aniq amal qilib uchta test yozing: normal holat, chegara (0% va 100%), va noto'g'ri foiz.

2. **Test double turini tanlash.** Quyidagi har bir holat uchun qaysi test double (dummy/stub/spy/mock/fake) mosligini va nega ekanini yozing, so'ng Jest bilan misol kodini keltiring: (a) `getExchangeRate()` har doim `12500` qaytarsin, (b) `logger.log` chaqirilganini tekshirish, (c) email yuborilgani va aynan qaysi manzilga ekanini tasdiqlash, (d) in-memory user repository.

3. **TDD bilan FizzBuzz.** Red-Green-Refactor siklidan foydalanib `fizzbuzz(n)` funksiyasini yozing (3 ga bo'linsa "Fizz", 5 ga "Buzz", ikkalasiga "FizzBuzz", aks holda sonning o'zi). Har bir qadamda avval test, keyin kod tartibini ko'rsating.

4. **Branch coverage tutqun.** Berilgan funksiya 100% line coverage'ga ega bo'lsa-yu, baribir bug o'tib ketadigan holatni ko'rsating. So'ng branch coverage'ni 100% qiladigan testlarni qo'shing va qaysi shox avval qoplanmaganini izohlang.

5. **Flaky testni tuzatish.** Quyidagi flaky test berilgan: `setTimeout(() => done(), 100)` dan keyin natija tekshiriladi. Nima uchun u flaky ekanini tushuntiring va uni barqaror qiladigan ikki xil yondashuvni (fake timers va waitFor) yozing.

6. **FIRST buzilishini topish.** Berilgan uchta test global `cache` o'zgaruvchisini bo'lishadi va shu sababli tartibga bog'liq. Qaysi FIRST printsipi buzilganini aniqlang va testlarni mustaqil qiladigan tarzda qayta yozing.

7. **Mocking chegarasi.** `createUser` funksiyasi: parolni hash qiladi (sof funksiya), DB'ga yozadi va xush kelibsiz email yuboradi. Qaysi qismni mock qilasiz, qaysi qismni haqiqiy qoldirasiz va nega? Test kodini yozing.

8. **Nimani test qilmaslik.** Berilgan klassda `getId()`, `setId()` (trivial), `calculateTax()` (biznes logika) va `_buildQuery()` (private) bor. Qaysilarini test qilasiz, qaysilarini yo'q va nega? Faqat arzigan qismlar uchun test yozing.

← [Testing bo'limiga qaytish](./README.md)
