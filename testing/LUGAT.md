# Testing — Lug'at

Dasturiy ta'minotni test qilishda uchraydigan inglizcha terminlar va ularning sodda o'zbekcha izohi.

| Termin | Ma'nosi (o'zbekcha) |
|--------|---------------------|
| AAA Pattern | Testni uch qismga bo'lish tartibi: Arrange (tayyorla), Act (harakat qil), Assert (natijani tekshir). |
| Acceptance Test | Tizim biznes talablariga javob berishini tasdiqlaydigan test — "buyurtmachi qabul qiladimi?" degan savolga javob. |
| Arrange | AAA namunasining birinchi bosqichi — testga kerakli ma'lumot va holatni tayyorlash. |
| Assertion | Test ichida "natija kutilganidek bo'lishi kerak" degan tekshiruv — noto'g'ri bo'lsa, test yiqiladi. |
| BDD (Behavior-Driven Development) | Testni foydalanuvchi xatti-harakati tilida (masalan "given/when/then") yozish uslubi — biznes bilan umumiy til. |
| Black Box Testing | Ichki kodni bilmay, faqat kirish va chiqishga qarab tekshirish — tizimga "qora quti" sifatida qarash. |
| Branch Coverage | Koddagi barcha shart tarmoqlari (if'ning ham rost, ham yolg'on yo'llari) test qilinganini o'lchaydigan qamrov turi. |
| Contract Test | Ikki servis o'rtasidagi "kelishuv" (so'rov/javob shakli) buzilmaganini tekshiradigan test. |
| Coverage | Kodning qancha qismi testlar bilan qamralganini foizda ko'rsatadigan o'lchov (masalan qatorlar, tarmoqlar). |
| Cypress | Brauzerda ishlaydigan e2e va integratsiya testlari uchun mashhur test vositasi — real UI bilan ishlaydi. |
| DI (Dependency Injection) | Modulga kerakli bog'liqliklarni tashqaridan berish — test paytida ularni soxta (mock) versiyalarga almashtirishni osonlashtiradi. |
| Dummy | Faqat o'rin to'ldirish uchun uzatiladigan, lekin ishlatilmaydigan soxta obyekt (test double turi). |
| E2E (End-to-End) | Butun tizimni foydalanuvchi ko'zi bilan boshdan-oxir sinovdan o'tkazadigan test (masalan login qilishdan buyurtmagacha). |
| Edge Case | Kutilgan chegaradagi noodatiy holat (masalan bo'sh ro'yxat, 0 yoki juda katta son) — ko'pincha xatolar shu yerdan chiqadi. |
| Expectation | Test kutayotgan natija — expect(...) orqali yoziladi va matcher bilan solishtiriladi. |
| Fake | Haqiqiy ishlaydigan, lekin soddalashtirilgan almashtiruvchi (masalan xotiradagi soxta ma'lumotlar bazasi). |
| Fixture | Test uchun oldindan tayyorlangan ma'lumot yoki holat — testlar shu barqaror boshlang'ich nuqtadan foydalanadi. |
| FIRST | Yaxshi unit test tamoyillari: Fast, Independent, Repeatable, Self-validating, Timely. |
| Flaky Test | Kod o'zgarmasa ham goh o'tadigan, goh yiqiladigan beqaror test — ishonchsizligi sababli katta muammo. |
| Given-When-Then | BDD'da test stsenariysini yozish shakli: berilgan holat, sodir bo'lgan harakat, kutilgan natija. |
| Happy Path | Hamma narsa to'g'ri va kutilganidek ketadigan asosiy stsenariy — xatolar yo'q "baxtli yo'l". |
| Headless | Brauzerni ko'rinadigan oynasiz, fon rejimida ishga tushirish — CI'da e2e testlarni tez ishlatish uchun. |
| Integration Test | Bir nechta modul yoki komponent birga to'g'ri ishlashini tekshiradigan test — ular orasidagi bog'lanishni sinaydi. |
| Jest | JavaScript uchun keng tarqalgan test freymvorki — assertion, mock va coverage'ni bir joyda beradi. |
| Matcher | Assertion ichida qiymatni qanday solishtirishni belgilaydigan funksiya (masalan toBe, toEqual, toContain). |
| Mock | Haqiqiy obyekt o'rniga qo'yiladigan soxta obyekt — u qanday chaqirilganini yozib boradi va tekshirishga imkon beradi. |
| Mutation Testing | Kodga atayin kichik xatolar kiritib, testlar ularni ushlaydimi yo'qmi tekshirish — testlar sifatini o'lchaydi. |
| Playwright | Bir nechta brauzerda ishonchli e2e testlar yozish uchun zamonaviy Microsoft vositasi. |
| React Testing Library | React komponentlarini foydalanuvchi ko'radigan tarzda (ichki tafsilotlarga emas, xatti-harakatga qarab) test qiladigan kutubxona. |
| Red-Green-Refactor | TDD sikli: avval yiqiladigan test (qizil), keyin uni o'tkazadigan kod (yashil), so'ng kodni tozalash (refactor). |
| Regression | Ilgari ishlagan funksiyaning yangi o'zgarish tufayli buzilishi — regression testlar buni ushlab qoladi. |
| Sandbox | Testni real muhitga ta'sir qilmaydigan izolyatsiyalangan "qum qutisi"da ishlatish. |
| Setup | Har bir test yoki test to'plamidan oldin ishga tushadigan tayyorgarlik kodi (masalan beforeEach). |
| Smoke Test | Tizim umuman ishga tushadimi va asosiy funksiyalar tirikmi — tez, yuzaki "tutun bormi?" tekshiruvi. |
| Snapshot | Komponent yoki natijaning "surati"ni saqlab, keyingi ishlashda o'sha suratga o'zgarganini tekshiradigan test turi. |
| Spy | Funksiyani o'rab olib, u qancha marta va qanday argument bilan chaqirilganini kuzatadigan, lekin asl ishini buzmaydigan vosita. |
| Stub | Chaqirilganda oldindan belgilangan tayyor javob qaytaradigan soxta funksiya — real logikani ishlatmaydi. |
| Teardown | Test tugagach ishlab, tozalash ishlarini bajaradigan kod (masalan afterEach) — holatni tiklaydi. |
| TDD (Test-Driven Development) | Avval testni yozib, keyin uni o'tkazadigan kodni yozish uslubi (Red → Green → Refactor sikli). |
| Test Case | Bitta aniq holatni tekshiradigan mustaqil test — kirish, harakat va kutilgan natijadan iborat. |
| Test Double | Real obyekt o'rniga test uchun qo'yiladigan har qanday almashtiruvchining umumiy nomi (mock, stub, spy, fake, dummy). |
| Test Pyramid | Testlarni nisbatini ko'rsatuvchi model: ko'p unit, o'rtacha integration, kam e2e test bo'lishi tavsiya etiladi. |
| Test Runner | Testlarni topib, ishga tushirib, natijasini ko'rsatadigan dastur (masalan Jest, Vitest'ning yadrosi). |
| Test Suite | O'zaro bog'liq testlar guruhi — odatda bitta modul yoki xususiyatga tegishli (describe bloki). |
| Unit Test | Kodning eng kichik mustaqil bo'lagini (masalan bitta funksiyani) alohida sinovdan o'tkazadigan test. |
| Vitest | Vite ekotizimi uchun tez va zamonaviy test freymvorki — Jest'ga o'xshash, lekin tezroq va sozlash oson. |
| White Box Testing | Ichki kod strukturasini bilib turib, uning har bir yo'lini maqsadli tekshirish (black box'ning teskarisi). |

← [Testing bo'limiga qaytish](./README.md)
