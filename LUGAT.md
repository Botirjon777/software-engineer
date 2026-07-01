# 📖 Umumiy Lug'at — Texnik Terminlar

Butun repo bo'ylab ishlatilgan ingliz tilidagi texnik terminlarning o'zbekcha ma'nolari. Bu **to'g'ridan-to'g'ri tarjima emas** — har bir termin "nima degani, nima ma'noda" tarzida sodda tushuntirilgan.

> Har bir bo'limning o'z lug'ati ham bor (`<bo'lim>/LUGAT.md`). Bu fayl hammasini bitta joyga jamlagan.

## Mundarija

1. [Frontend](#frontend)
2. [Backend](#backend)
3. [Database](#database)
4. [Networking](#networking)
5. [Encoding va Hashing](#encoding-va-hashing)
6. [DSA](#dsa)
7. [CS Asoslari](#cs-asoslari)
8. [DevOps](#devops)
9. [Testing](#testing)
10. [OOP, Patterns va Clean Code](#oop-patterns-va-clean-code)
11. [System Design](#system-design)
12. [Behavioral](#behavioral)
13. [Hardware](#hardware)
14. [Loyihalar](#loyihalar)
15. [Deployment](#deployment)
16. [Email](#email)

---

## Frontend

| Termin | Ma'nosi (o'zbekcha) |
|--------|---------------------|
| App Router | Next.js'ning yangi marshrutlash tizimi — papka strukturasi orqali sahifalar yasaladi va server komponentlarini standart qilib qo'yadi. |
| Async/Await | Promise'lar bilan ishlashning chiroyli usuli — kod xuddi ketma-ket yozilganday ko'rinadi, aslida esa kutish asinxron bo'ladi. |
| Bundle | Barcha JavaScript fayllaringni birlashtirib, brauzer yuklaydigan bitta (yoki bir nechta) katta faylga jamlash natijasi. |
| Bundler | Kodni bir joyga yig'ib, optimallashtirib chiqadigan vosita (masalan Webpack, Vite) — importlarni bir bundle'ga bog'laydi. |
| Cascade | CSS'da bir nechta qoida bitta elementga tegsa, qaysi biri g'olib chiqishini belgilaydigan tartib qoidalari. |
| Callback | Boshqa funksiyaga argument qilib beriladigan funksiya — ish tugagach yoki hodisa yuz berganda chaqiriladi. |
| CLS (Cumulative Layout Shift) | Sahifa yuklanayotganda elementlar "sakrab" joyini o'zgartirishini o'lchaydigan Core Web Vitals ko'rsatkichi — kam bo'lgani yaxshi. |
| Closure | Funksiya o'zi yaratilgan scope'dagi o'zgaruvchilarni "eslab qoladi" — funksiya tashqarida chaqirilsa ham o'sha o'zgaruvchilarga kira oladi. |
| Code Splitting | Butun kodni bir vaqtda yuklamay, faqat kerakli qismini yuklash uchun bo'laklarga ajratish. |
| Coercion | JavaScript'ning bir turdagi qiymatni avtomatik boshqa turga o'girishi (masalan son bilan matnni qo'shganda). |
| Component | React'da UI'ning qayta ishlatiladigan mustaqil bo'lagi — o'z markup va logikasiga ega funksiya yoki klass. |
| Context | React'da ma'lumotni har bir komponentga props orqali uzatmay, "yuqoridan pastga" to'g'ridan-to'g'ri yetkazish mexanizmi. |
| Controlled Component | Qiymati React state tomonidan boshqariladigan forma elementi — har bir o'zgarish state orqali o'tadi. |
| Core Web Vitals | Google'ning sahifa tajribasini o'lchaydigan asosiy ko'rsatkichlari (LCP, CLS, INP) — SEO va tezlikка ta'sir qiladi. |
| CSR (Client-Side Rendering) | Sahifa brauzerda JavaScript yordamida chiziladi — server bo'sh HTML yuboradi, qolganini brauzer to'ldiradi. |
| Currying | Ko'p argumentli funksiyani, har biri bitta argument oladigan zanjirli funksiyalarga aylantirish usuli. |
| Debounce | Tez-tez takrorlanadigan hodisani (masalan yozish) — foydalanuvchi to'xtaganidan keyingina bir marta ishga tushirish texnikasi. |
| Destructuring | Massiv yoki obyektdan qiymatlarni bitta qatorda alohida o'zgaruvchilarga ajratib olish sintaksisi. |
| DOM (Document Object Model) | HTML sahifasining daraxt shaklidagi obyekt ko'rinishi — JavaScript shu daraxt orqali sahifani o'zgartiradi. |
| Enum | Belgilangan doimiy qiymatlar to'plamiga nom beruvchi TypeScript turi (masalan Status.Active, Status.Done). |
| Event Delegation | Har bir bola elementga alohida emas, balki bitta ota elementga hodisa qo'yib, hodisa "ko'tarilishi"dan foydalanish. |
| Event Loop | JavaScript'ning bir yo'nalishli bo'lsa ham asinxron ishlarni navbatga qo'yib bajaradigan mexanizmi. |
| Flexbox | Elementlarni bir o'lchamli (qator yoki ustun) bo'ylab moslashuvchan joylashtiradigan CSS tartiblash tizimi. |
| Generic | TypeScript'da turni oldindan belgilamay, ishlatilganda aniqlanadigan "shablon tur" (masalan Array<T>). |
| Grid | Elementlarni ikki o'lchamli (qator va ustun) to'r shaklida joylashtiradigan CSS tartiblash tizimi. |
| Higher-Order Function | Boshqa funksiyani argument sifatida qabul qiladigan yoki funksiya qaytaradigan funksiya (masalan map, filter). |
| Hoisting | O'zgaruvchi va funksiya e'lonlarini JavaScript "yuqoriga ko'tarib" oldindan tanidek qilishi — lekin qiymat keyin beriladi. |
| Hook | React'da funksional komponentga state va boshqa imkoniyatlarni ulaydigan maxsus funksiya (useState, useEffect...). |
| HTML | Sahifaning skeleti — sarlavha, matn, tugma kabi kontent va uning strukturasini belgilaydigan belgilash tili. |
| Hydration | Serverda tayyor chizilgan HTML'ni brauzerda "jonlantirish" — JavaScript'ni ulab, interaktiv qilish jarayoni. |
| Immutability | Ma'lumotni o'zgartirmaslik — o'zgartirish kerak bo'lsa, eskisini o'zgartirmay, yangi nusxa yasash tamoyili. |
| INP (Interaction to Next Paint) | Foydalanuvchi bosgandan keyin sahifa qanchalik tez javob berishini o'lchaydigan Core Web Vitals ko'rsatkichi. |
| Interface | TypeScript'da obyekt qanday ko'rinishda bo'lishi (qaysi maydonlar va turlar) kerakligini tavsiflovchi shartnoma. |
| ISR (Incremental Static Regeneration) | Statik sahifalarni butunlay qayta qurmasdan, vaqti-vaqti bilan fonda yangilab turadigan Next.js usuli. |
| JavaScript | Brauzerda ishlaydigan, sahifani interaktiv qiladigan asosiy dasturlash tili. |
| JSX | React'da HTML kabi ko'rinadigan, lekin JavaScript ichida yoziladigan sintaksis — komponent markup'ini tavsiflaydi. |
| Lazy Loading | Kontentni yoki kodni faqat kerak bo'lganda (masalan ekranga yaqinlashganda) yuklash texnikasi. |
| LCP (Largest Contentful Paint) | Sahifadagi eng katta ko'rinadigan element qachon chizilishini o'lchaydi — sahifa "yuklandi" hissini beradigan asosiy vaqt. |
| Media Query | Ekran o'lchami yoki qurilma turiga qarab CSS qoidalarini shartli qo'llaydigan mexanizm (responsive dizayn asosi). |
| Memoization | Funksiya natijasini eslab qolib, xuddi shu kirish bilan yana chaqirilsa, qayta hisoblamay saqlangan natijani qaytarish. |
| Microtask | Event loop'da oddiy vazifalardan oldin bajariladigan yuqori ustuvorlikdagi kichik vazifa (masalan Promise then'i). |
| Middleware | So'rov sahifaga yetib borishidan oldin ishlaydigan oraliq kod — yo'naltirish, autentifikatsiya kabi ishlarni bajaradi. |
| Narrowing | TypeScript'da tekshiruvlar orqali kengroq turni aniqroq turga "toraytirish" (masalan union'dan bitta turga). |
| Next.js | React ustiga qurilgan freymvork — SSR, marshrutlash, optimallashtirish kabi imkoniyatlarni tayyor beradi. |
| Box Model | Har bir HTML elementi content, padding, border va margin qatlamlaridan iborat "quti" ekanini tushuntiruvchi model. |
| Portal | React'da komponentni DOM daraxtining boshqa joyiga (masalan modal uchun body oxiriga) chizish imkonini beruvchi mexanizm. |
| Promise | Kelajakda tugaydigan asinxron amalning natijasini ifodalovchi obyekt — muvaffaqiyat yoki xatolik holatiga ega. |
| Props | Ota komponentdan bola komponentga uzatiladigan ma'lumotlar — komponentni sozlash uchun "kirish parametrlari". |
| Prototype | JavaScript obyektlari xususiyat va metodlarni meros oladigan zanjir — obyekt topolmasa, prototipidan qidiradi. |
| Pseudo-class | Elementning maxsus holatiga (masalan :hover, :focus) CSS qo'llash imkonini beruvchi selektor qismi. |
| Reconciliation | React virtual DOM'dagi eski va yangi holatni solishtirib, real DOM'da faqat farqni yangilash jarayoni. |
| Reducer | Joriy state va harakatni (action) qabul qilib, yangi state qaytaradigan sof funksiya (useReducer, Redux asosi). |
| Reflow | Brauzer element o'lchami yoki joylashuvi o'zgargani sababli sahifa tartibini qaytadan hisoblab chiqishi — qimmat amal. |
| Repaint | Element ko'rinishi (rang, soya) o'zgarganda, tartibni o'zgartirmay faqat qayta bo'yash — reflow'dan arzonroq. |
| Re-render | React komponentini state yoki props o'zgargani uchun qaytadan chizilishi (funksiyaning qayta chaqirilishi). |
| Ref | React'da DOM elementiga yoki qiymatni re-render'siz saqlashga to'g'ridan-to'g'ri murojaat qilish vositasi (useRef). |
| RSC (React Server Components) | Faqat serverda ishlaydigan React komponentlari — brauzerga JavaScript yubormay, tayyor natija beradi. |
| Scope | O'zgaruvchi qayerda ko'rinadigan va ishlatiladiganini belgilaydigan "hudud" (global, funksiya, blok scope). |
| Semantic HTML | Teglarni ma'nosiga qarab ishlatish (header, nav, article) — ekran o'quvchilar va SEO uchun tushunarli struktura. |
| Specificity | Bir nechta CSS selektori bir elementga tegganda, qaysi biri kuchliroq ekanini hisoblaydigan ustuvorlik og'irligi. |
| Spread | Massiv yoki obyekt elementlarini "yoyib" boshqa joyga ko'chirish sintaksisi (...) — nusxalash va birlashtirishда qulay. |
| SSG (Static Site Generation) | Sahifalarni build vaqtida oldindan HTML qilib tayyorlab qo'yish — juda tez, lekin kontent statik. |
| SSR (Server-Side Rendering) | Sahifani serverda tayyor HTML qilib chizib, brauzerga yuborish — tez ko'rinadi va SEO uchun yaxshi. |
| Stacking Context | Elementlarning bir-birining ustidagi "qatlam"lari qanday tartibda joylashishini belgilaydigan mustaqil kontekst. |
| State | Komponentning vaqt bilan o'zgaradigan ichki ma'lumoti — o'zgarganda React komponentni qayta chizadi. |
| Suspense | React'da kontent (masalan lazy komponent yoki ma'lumot) yuklanguncha zaxira UI ("Yuklanmoqda...") ko'rsatish mexanizmi. |
| This | Funksiya qanday chaqirilganiga qarab o'zgaradigan maxsus kalit so'z — joriy kontekst obyektiga ishora qiladi. |
| Throttle | Tez-tez takrorlanadigan hodisani ma'lum vaqt oralig'ida faqat bir marta ishga tushirishga cheklaydigan texnika. |
| Tree Shaking | Bundle'dan ishlatilmagan ("o'lik") kodni avtomatik olib tashlab, hajmni kichraytirish jarayoni. |
| Type | TypeScript'da qiymat qanday shaklda bo'lishini bildiruvchi tavsif — kompilyatorga xatolarni oldindan topishga yordam beradi. |
| Type Guard | Kod ichida qiymat qaysi turga tegishliligini tekshirib, turni aniqlashtiradigan shart (masalan typeof, instanceof). |
| Union | TypeScript'da qiymat bir nechta turdan birortasi bo'lishi mumkinligini bildiruvchi tur (masalan string | number). |
| Utility Type | TypeScript'ning mavjud turdan yangi tur yasaydigan tayyor yordamchilari (Partial, Pick, Omit, Readonly...). |
| Viewport | Brauzerning sahifa ko'rinadigan oynasi — responsive dizayn va o'lchamlar shu ko'rinish maydoniga nisbatan hisoblanadi. |
| Virtual DOM | React saqlaydigan real DOM'ning yengil nusxasi — o'zgarishlarni avval shu yerda hisoblab, keyin real DOM'ni yangilaydi. |
| Z-index | Elementlarning bir-biriga nisbatan qaysi biri yuqorida turishini (qatlam tartibini) belgilaydigan CSS xususiyati. |

[⬆ Mundarijaga qaytish](#mundarija)

---

## Backend

| Termin | Ma'nosi (o'zbekcha) |
|--------|---------------------|
| API gateway | Barcha so'rovlar avval kiradigan yagona kirish nuqtasi. U so'rovni to'g'ri xizmatga yo'naltiradi, autentifikatsiya va rate limiting kabi ishlarni bir joyda bajaradi. |
| async/await | Asinxron kodni xuddi oddiy ketma-ket kod kabi yozish imkonini beruvchi sintaksis. `await` natija kelguncha kutadi, lekin dasturni bloklamaydi. |
| authentication | Foydalanuvchi kimligini tekshirish jarayoni, ya'ni "sen aytgan odammisan?". Odatda login va parol orqali amalga oshiriladi. |
| authorization | Foydalanuvchi nima qilishga ruxsati borligini tekshirish, ya'ni "senga bu ishga ruxsat bormi?". Autentifikatsiyadan keyin keladi. |
| backpressure | Ma'lumot ishlab chiqaruvchi qabul qiluvchidan tezroq ishlaganda yuzaga keladigan holat. Stream oqimni sekinlashtirib, xotira to'lib ketishining oldini oladi. |
| blocking | Bir amal tugamaguncha keyingi kodni ishga tushirmay, dasturni to'xtatib turadigan operatsiya. Node.js'da bundan qochish tavsiya etiladi. |
| buffer | Xotirada vaqtincha saqlanadigan xom baytlar to'plami. Fayl yoki tarmoq ma'lumotini o'qishda ishlatiladi. |
| caching | Tez-tez kerak bo'ladigan ma'lumotni tezkor joyda saqlab, keyingi so'rovda qayta hisoblamaslik usuli. Tezlikni sezilarli oshiradi. |
| callback | Boshqa funksiyaga argument sifatida uzatiladigan va biror ish tugagach chaqiriladigan funksiya. Asinxron kodning eng eski usuli. |
| client | Serverdan xizmat yoki ma'lumot so'rab murojaat qiluvchi tomon, masalan brauzer yoki mobil ilova. |
| cookie | Server tomonidan brauzerda saqlanadigan kichik ma'lumot bo'lagi. Har so'rovda avtomatik yuboriladi, ko'pincha sessiyani saqlash uchun ishlatiladi. |
| CORS | Bir domendagi sahifa boshqa domendagi API'ga murojaat qila olishini boshqaruvchi brauzer xavfsizlik qoidasi. Ruxsatni server sarlavhalar orqali beradi. |
| dependency injection | Obyekt o'ziga kerakli qismlarni o'zi yaratmasdan, tashqaridan tayyor holda qabul qilib olish usuli. Kodni test qilish va almashtirishni osonlashtiradi. |
| endpoint | API'ning ma'lum bir manzili (URL), unga so'rov yuborilib biror amal bajariladi. Masalan `/users/1`. |
| event loop | Node.js'ning yuragi bo'lgan mexanizm. U navbatdagi vazifalarni doimiy tekshirib, bittalab bajarib boradi va asinxronlikni ta'minlaydi. |
| EventEmitter | Node.js'da hodisalarni e'lon qilish va ularni tinglash imkonini beruvchi asosiy sinf. "on" bilan tinglanadi, "emit" bilan chaqiriladi. |
| graceful shutdown | Serverni to'satdan o'chirmasdan, joriy so'rovlarni tugatib, ulanishlarni yopib, so'ng xotirjam to'xtatish. |
| GraphQL | Mijoz o'ziga aynan kerak bo'lgan ma'lumatni bitta so'rovda so'rab olishga imkon beruvchi so'rov tili. Ortiqcha ma'lumot olishning oldini oladi. |
| gRPC | Xizmatlar orasida tez va samarali aloqa uchun ishlatiladigan zamonaviy protokol. Protobuf formatidan foydalanadi. |
| hashing | Ma'lumotni (masalan parolni) qaytarib bo'lmaydigan tarzda o'zgacha kodga aylantirish. Parollarni asl holida saqlamaslik uchun ishlatiladi. |
| HTTP method | So'rovning maqsadini bildiruvchi so'z: GET (olish), POST (yaratish), PUT/PATCH (yangilash), DELETE (o'chirish). |
| idempotency | Bitta so'rovni bir necha marta yuborsang ham natija o'zgarmaydigan xususiyat. Masalan DELETE bir necha bor chaqirilsa ham obyekt bir marta o'chadi. |
| JWT | Foydalanuvchi haqidagi ma'lumotni ichida saqlaydigan imzolangan token. Server har safar bazaga murojaat qilmasdan uni tekshira oladi. |
| libuv | Node.js'ga asinxronlik va tizim bilan ishlashni beruvchi C kutubxonasi. Event loop va thread pool aynan shu yerda. |
| load balancing | Kelayotgan so'rovlarni bir necha server orasida taqsimlab, birortasi ortiqcha yuklanib qolishining oldini olish. |
| macrotask | Event loop'da katta navbatdagi vazifa, masalan setTimeout yoki I/O. Microtask'lardan keyin bajariladi. |
| message queue | Xabarlarni vaqtincha saqlab, ularni keyinroq navbat bilan qayta ishlashga uzatuvchi tizim. Xizmatlarni bir-biridan ajratadi. |
| microservices | Ilovani mustaqil, kichik va alohida ishlaydigan xizmatlarga bo'lib qurish uslubi. Har biri o'z vazifasiga javob beradi. |
| microtask | Event loop'da eng tez bajariladigan kichik vazifa, masalan hal bo'lgan promise. Macrotask'lardan oldin ishlaydi. |
| middleware | So'rov serverga yetib borgunga qadar oraliqda ishlaydigan funksiya. Log yozish, tekshirish yoki ma'lumot qo'shishda ishlatiladi. |
| monolith | Butun ilova bitta yagona katta kod bazasi sifatida qurilgan arxitektura. Microservices'ning aksi. |
| Node.js | JavaScript'ni brauzerdan tashqarida, serverda ishlatish imkonini beruvchi platforma. V8 dvigateli asosida qurilgan. |
| non-blocking | Amal tugashini kutmasdan keyingi kodni davom ettiradigan operatsiya. Node.js aynan shu tamoyilga asoslanadi. |
| OAuth | Foydalanuvchi parolini bermasdan, boshqa xizmat orqali (masalan Google) tizimga kirish imkonini beruvchi standart. |
| ORM | Ma'lumotlar bazasidagi jadvallar bilan kod obyektlari sifatida ishlash imkonini beruvchi vosita. SQL yozishni kamaytiradi. |
| pipe | Bir stream chiqishini boshqasining kirishiga ulash usuli. Ma'lumot avtomatik oqib o'tadi, backpressure ham o'zi boshqariladi. |
| preflight | Brauzer asosiy so'rovdan oldin yuboradigan tekshiruv so'rovi (OPTIONS). Server ruxsat berish-bermasligini oldindan aniqlaydi. |
| promise | Kelajakda tayyor bo'ladigan natijani ifodalovchi obyekt. "resolve" bilan muvaffaqiyat, "reject" bilan xatolikni bildiradi. |
| pub/sub | Xabar yuboruvchilar (publisher) va tinglovchilar (subscriber) bir-birini bilmasdan xabar almashadigan model. Kanal orqali bog'lanadi. |
| rate limiting | Bir foydalanuvchi ma'lum vaqt ichida yubora oladigan so'rovlar sonini cheklash. Suiiste'mol va yuklanishdan himoya qiladi. |
| REST | HTTP metodlari va URL'lardan foydalanib API qurishning keng tarqalgan uslubi. Oddiy va bir xil qoidalarga asoslanadi. |
| runtime | Kod ishlab turgan muhit, ya'ni dasturni bajaruvchi tizim. Masalan Node.js — JavaScript uchun runtime. |
| salt | Parolni hashlashdan oldin qo'shiladigan tasodifiy qiymat. Bir xil parollar ham har xil hash olishini ta'minlaydi. |
| server | Mijoz so'rovlarini qabul qilib, ularga javob qaytaruvchi dastur yoki kompyuter. Backendning asosiy qismi. |
| session | Foydalanuvchining tizimga kirgan holatini serverda saqlab turish usuli. Har so'rovda kim ekanligini eslab qoladi. |
| SSE | Server mijozga bir tomonlama uzluksiz yangilanishlar yuboradigan texnologiya (Server-Sent Events). WebSocket'dan soddaroq. |
| statelessness | Server har so'rovni oldingilardan mustaqil ravishda, hech narsa eslab qolmasdan qayta ishlashi. REST'ning asosiy tamoyili. |
| status code | Serverning javob holatini bildiruvchi raqam: 200 (muvaffaqiyat), 404 (topilmadi), 500 (server xatosi). |
| stream | Ma'lumotni to'liq kutmasdan, bo'laklab uzatib borish usuli. Katta fayllar bilan xotirani tejab ishlaydi. |
| thread pool | Og'ir vazifalarni parallel bajarish uchun tayyor turgan bir nechta ish oqimlari to'plami. libuv boshqaradi. |
| V8 | Google yaratgan JavaScript dvigateli. Kodni tez mashina koduga aylantiradi, Node.js va Chrome'da ishlatiladi. |
| WebSocket | Mijoz va server o'rtasida ikki tomonlama, doimiy ochiq aloqa kanali. Chat va real vaqt ilovalari uchun ideal. |

[⬆ Mundarijaga qaytish](#mundarija)

---

## Database

| Termin | Ma'nosi (o'zbekcha) |
|--------|---------------------|
| ACID | Tranzaksiyalar ishonchli bo'lishini kafolatlovchi to'rtta xususiyat: Atomicity, Consistency, Isolation, Durability. |
| aggregate | Bir nechta qatordan yagona natija chiqaruvchi hisoblash, masalan yig'indi (SUM), o'rtacha (AVG) yoki son (COUNT). |
| atomicity | Tranzaksiya yo to'liq bajariladi, yo umuman bajarilmaydi — yarim holatda qolmaydi. ACID'ning "A" harfi. |
| BASE | NoSQL bazalar uchun yumshoqroq yondashuv: doim mavjud, moslashuvchan holat, oxir-oqibat izchillik. ACID'ga muqobil. |
| B-tree | Indekslarda ishlatiladigan muvozanatli daraxt tuzilmasi. Ma'lumotni tez topish va tartiblab saqlash imkonini beradi. |
| CAP theorem | Taqsimlangan tizimda izchillik, mavjudlik va bo'linishga chidamlilikning uchtasini bir vaqtda to'liq ta'minlab bo'lmasligi haqidagi qoida. |
| caching | Tez-tez so'raladigan ma'lumotni tezkor xotirada saqlab, bazaga qayta murojaatni kamaytirish. |
| clustered index | Jadvaldagi qatorlarni jismonan indeks tartibida saqlaydigan indeks. Bir jadvalda faqat bitta bo'lishi mumkin. |
| column | Jadvaldagi ustun, ya'ni ma'lumotning ma'lum bir maydoni, masalan "ism" yoki "yosh". |
| connection pooling | Bazaga ulanishlarni har safar yangidan ochmasdan, tayyor turganlarini qayta ishlatish usuli. Tezlikni oshiradi. |
| covering index | So'rovga kerak bo'lgan barcha ustunlarni o'zida saqlaydigan indeks. Jadvalga umuman murojaat qilmasdan javob beradi. |
| CTE | So'rov ichida vaqtinchalik nomlangan natija to'plami (WITH ... AS). Murakkab so'rovlarni o'qishni osonlashtiradi. |
| DBMS | Ma'lumotlar bazasini boshqaruvchi tizim, masalan PostgreSQL yoki MySQL. Ma'lumotni saqlash va so'rashni ta'minlaydi. |
| DDL | Baza tuzilmasini belgilovchi buyruqlar to'plami: CREATE, ALTER, DROP. Jadval va sxemalarni yaratadi. |
| deadlock | Ikki tranzaksiya bir-birining resursini kutib, ikkalasi ham hech qachon davom eta olmay qotib qolishi. |
| denormalization | Tezlik uchun ataylab ma'lumotni takrorlab saqlash. Normalizatsiyaning aksi, JOIN'larni kamaytiradi. |
| dirty read | Bir tranzaksiya boshqa tranzaksiya hali saqlamagan (tasdiqlanmagan) ma'lumotni o'qib olishi. Xato holat. |
| DML | Ma'lumotning o'zi bilan ishlovchi buyruqlar: SELECT, INSERT, UPDATE, DELETE. |
| document | NoSQL'da JSON kabi ko'rinishdagi yagona yozuv. Ichida ichma-ich maydonlar bo'lishi mumkin. |
| eventual consistency | Ma'lumot hamma nusxalarga darhol emas, biroz vaqt o'tib bir xil bo'lishi. Taqsimlangan tizimlarda keng qo'llaniladi. |
| foreign key | Bir jadvaldagi ustunni boshqa jadvalning asosiy kalitiga bog'laydigan maydon. Jadvallar orasidagi aloqani ta'minlaydi. |
| graph | Ma'lumotni tugunlar va ular orasidagi bog'lanishlar tarzida saqlaydigan NoSQL turi. Ijtimoiy tarmoqlar uchun qulay. |
| GROUP BY | Qatorlarni ma'lum ustun qiymati bo'yicha guruhlab, har guruh uchun hisob chiqarish usuli. |
| index | So'rovni tezlashtirish uchun tuzilgan qidiruv tuzilmasi, kitobning ko'rsatkichi kabi. Topishni tezlashtiradi. |
| isolation | Bir vaqtda ishlayotgan tranzaksiyalar bir-biriga xalaqit bermasligini ta'minlash. ACID'ning "I" harfi. |
| isolation level | Tranzaksiyalar bir-birini qanchalik "ko'rishi" mumkinligini belgilovchi daraja, masalan READ COMMITTED. |
| JOIN | Ikki yoki undan ortiq jadvalni bog'lanish orqali birlashtirib, birga so'rash. INNER, LEFT, RIGHT, FULL turlari bor. |
| key-value | Ma'lumotni oddiy "kalit — qiymat" juftliklari sifatida saqlaydigan eng sodda NoSQL turi. Masalan Redis. |
| locking | Bir tranzaksiya ma'lumotni o'zgartirayotganda boshqalarni vaqtincha to'xtatib turish mexanizmi. |
| MVCC | Har o'zgarish uchun ma'lumotning yangi nusxasini saqlab, o'qish va yozishni bir-biriga to'sqinliksiz olib borish usuli. |
| N+1 query | Bitta so'rov o'rniga har element uchun alohida qo'shimcha so'rov yuborilishidan kelib chiqadigan samarasizlik muammosi. |
| NoSQL | Jadval-satr shakliga bog'lanmagan, moslashuvchan bazalar oilasi. Katta hajm va tez o'sish uchun qulay. |
| normalization | Ma'lumot takrorlanishini kamaytirish uchun uni bir necha jadvalga bo'lib tashkil qilish. 1NF, 2NF, 3NF bosqichlari bor. |
| OLAP | Katta hajmdagi ma'lumotni tahlil va hisobot uchun ishlatishga mo'ljallangan tizim turi. Analitika uchun. |
| OLTP | Kundalik ko'plab kichik operatsiyalarni (yozish, yangilash) tez bajaruvchi tizim turi. Odatiy ilovalar uchun. |
| partitioning | Katta jadvalni mantiqiy bo'laklarga bo'lib saqlash, masalan yil bo'yicha. So'rov va boshqaruvni yengillashtiradi. |
| phantom read | Bir tranzaksiya davomida bir xil shart bilan qayta so'rasa, yangi qatorlar "paydo bo'lib qolishi". |
| primary key | Jadvaldagi har bir qatorni yagona qilib ajratib turadigan ustun. Takrorlanmas va bo'sh bo'lmaydi. |
| query | Bazadan ma'lumot so'rash yoki o'zgartirish uchun yoziladigan buyruq, odatda SQL tilida. |
| relational | Ma'lumotni jadvallar va ular orasidagi bog'lanishlar ko'rinishida saqlaydigan baza turi. Klassik SQL bazalar. |
| replication | Ma'lumotni bir necha serverga nusxalab saqlash. Ishonchlilik va o'qish tezligini oshiradi. |
| row | Jadvaldagi bitta yozuv, ya'ni bir qator ma'lumot. Masalan bitta foydalanuvchi haqidagi barcha maydonlar. |
| schema | Bazaning tuzilmasi: qanday jadvallar, ustunlar va turlar borligi haqidagi ta'rif. |
| sharding | Ma'lumotni bir necha alohida serverga (shard'larga) bo'lib tarqatish. Juda katta hajmni ko'tarishga yordam beradi. |
| subquery | Boshqa so'rov ichiga joylashtirilgan yordamchi so'rov. Ko'pincha filtr yoki hisob uchun ishlatiladi. |
| table | Ma'lumot satr va ustunlar shaklida saqlanadigan tuzilma. Relational bazaning asosiy birligi. |
| transaction | Bir yoki bir nechta amalni yagona bo'linmas birlik sifatida bajarish. Yo hammasi, yo hech qaysi. |
| view | Bitta yoki bir nechta jadvalga asoslangan saqlangan so'rov, virtual jadval kabi ishlaydi. |
| wide-column | Ma'lumotni ustunlar oilalari bo'yicha saqlaydigan NoSQL turi, masalan Cassandra. Juda katta hajmga chidamli. |
| window function | Qatorlarni guruhlamasdan, ular ustidan yugurib hisob chiqaruvchi funksiya, masalan reyting yoki yig'indi. |

[⬆ Mundarijaga qaytish](#mundarija)

---

## Networking

| Termin | Ma'nosi (o'zbekcha) |
|--------|---------------------|
| A record | DNS'da domen nomini IPv4 manzilга bog'laydigan yozuv. Masalan `example.com` degan nom qaysi IP'ga to'g'ri kelishini ko'rsatadi. |
| ACK | "Acknowledgement" — qabul qildim degan tasdiq signali. Qabul qiluvchi ma'lumot yetib kelganини jo'natuvchiga bildiradi. |
| API gateway | Ko'plab ichki xizmatlar oldida turuvchi yagona kirish nuqtasi. Barcha so'rovlar shu yerdan o'tib, tegishli xizmatga yo'naltiriladi. |
| bandwidth | Kanaldan bir vaqtda o'tkaza oladigan ma'lumotning maksimal hajmi. Yo'lning kengligi kabi — qancha keng bo'lsa, shuncha ko'p ma'lumot sig'adi. |
| CA | "Certificate Authority" — sertifikatlarни imzolab, ularга ishonch beruvchi tashkilot. Brauzer sayt haqiqiyligini shu CA orqali tekshiradi. |
| CDN | "Content Delivery Network" — kontentni dunyo bo'ylab serverlarга tarqatib, foydalanuvchiga eng yaqinidan yetkazuvchi tarmoq. Sayt tezroq ochilishiga yordam beradi. |
| certificate | Saytning haqiqiyligini isbotlovchi raqamli hujjat. Ichida saytning ochiq kaliti va CA imzosi bo'ladi. |
| CIDR | IP manzillar diapazonini qisqacha yozish usuli, masalan `192.168.0.0/24`. Slashdan keyingi son tarmoq qismining uzunligini bildiradi. |
| CNAME | Bir domen nomini boshqa domen nomiga bog'lovchi DNS yozuvi. Ya'ni "bu nom aslida ana u nomga qarasin" degani. |
| congestion control | Tarmoq tiqilib qolmasligi uchun jo'natish tezligini boshqarish. TCP tarmoqning holatiga qarab tezlikni oshirib-kamaytiradi. |
| encapsulation | Ma'lumotni har bir tarmoq qatlamida o'z sarlavhasi (header) bilan "o'rab" berish jarayoni. Har qatlam o'ziga kerakли ma'lumotni qo'shadi. |
| flow control | Jo'natuvchi qabul qiluvchini ko'p ma'lumot bilan bosib tashlamasligini ta'minlash. Qabul qiluvchi "sekinroq jo'nat" deya olishi mumkin. |
| forward proxy | Foydalanuvchi nomidan tashqi serverlarга so'rov yuboruvchi vositachi. Odatda foydalanuvchi tarafda turadi va uni yashiradi. |
| frame | Ma'lumotning eng past (Data Link) qatlamdagi paketi. MAC manzillar shu frame ichida bo'ladi. |
| gRPC | Google'ning yuqori samarali RPC (masofaviy chaqiruv) protokoli. HTTP/2 va Protobuf'dan foydalanib, xizmatlar o'rtasida tez aloqa qiladi. |
| handshake | Ikki tomon aloqa boshlashdan oldin kelishib olish jarayoni. Masalan TLS handshake'da shifrlash kalitlari almashiladi. |
| head-of-line blocking | Navbatdagi birinchi paket kechiksa, orqasidagilar ham kutib turishi. Bitta tiqilib qolgan element hammani ushlab qoladi. |
| HSTS | "HTTP Strict Transport Security" — brauzerga saytni faqat HTTPS orqali ochishni majbur qiluvchi qoida. HTTP'ga tushib qolishning oldini oladi. |
| HTTP/2 | HTTP protokolining tezlashtirilgan versiyasi. Bitta ulanish orqali bir nechta so'rovni parallel jo'natadi (multiplexing). |
| HTTP/3 | HTTP'ning eng yangi versiyasi bo'lib, QUIC (UDP asosidagi) protokoli ustida ishlaydi. Ulanishni tezroq o'rnatadi va kechikishlarni kamaytiradi. |
| HTTPS | Shifrlangan (TLS bilan himoyalangan) HTTP. Sayt bilan brauzer o'rtasidagi ma'lumot begonalar uchun o'qib bo'lmas holatga keladi. |
| IP address | Tarmoqdagi qurilmaning manzili — internetdagi "uy manzili" kabi. Har bir qurilma shu manzil orqali topiladi. |
| IPv4 | IP manzilning eski versiyasi, 32 bit, masalan `192.168.1.1`. Manzillar soni cheklangan bo'lgani uchun tugab bormoqda. |
| IPv6 | IP manzilning yangi versiyasi, 128 bit, juda katta manzillar zaxirasi bilan. Masalan `2001:0db8::1` ko'rinishida yoziladi. |
| JSON-RPC | JSON formatidan foydalanib masofaviy funksiya chaqiruvchi oddiy protokol. So'rov va javob JSON obyekti sifatida jo'natiladi. |
| L4 | Transport qatlami (Layer 4) — TCP/UDP darajasi. Bu darajada balanslash IP va portga qarab bajariladi. |
| L7 | Ilova qatlami (Layer 7) — HTTP darajasi. Bu darajada balanslash URL, sarlavha kabi ilova ma'lumotlarига qarab qilinadi. |
| latency | Ma'lumot jo'natilgandan qabul qilingunga qadar o'tgan kechikish vaqti. Kam bo'lsa yaxshi — tez javob degani. |
| load balancer | So'rovlarni bir nechta serverга teng taqsimlovchi qurilma yoki dastur. Bir serverга yuk to'planib qolmasligini ta'minlaydi. |
| MAC address | Tarmoq kartasiga zavodda berilgan noyob qurilma manzili. Mahalliy tarmoq ichida qurilmani aniqlash uchun ishlatiladi. |
| MCP | "Model Context Protocol" — AI modellarni tashqi vosita va ma'lumot manbalarига ulash uchun standart protokol. Model bilan xizmatlar o'rtasida ko'prik vazifasini bajaradi. |
| mTLS | "Mutual TLS" — ikkala tomon ham bir-birini sertifikat orqali tekshiradigan TLS. Faqat server emas, mijoz ham o'zini tasdiqlaydi. |
| multiplexing | Bitta ulanish orqali bir nechta oqimni aralashtirmasdan jo'natish. Har bir oqim o'z identifikatori bilan ajratiladi. |
| NAT | "Network Address Translation" — ichki tarmoq manzillarini bitta tashqi IP orqali internetга chiqarish. Uy routeriдa keng qo'llaniladi. |
| OSI model | Tarmoq aloqasini 7 qatlamга bo'lib tushuntiruvchi nazariy model. Har qatlam o'z vazifasini bajaradi (fizik, tarmoq, ilova va h.k.). |
| packet | Tarmoq (Network) qatlamidagi ma'lumot bo'lagi. Ichida jo'natuvchi va qabul qiluvchining IP manzili bo'ladi. |
| port | Bitta qurilmadagi turli xizmatlarни ajratuvchi raqam. Masalan HTTP 80-portда, HTTPS 443-portда ishlaydi. |
| proxy | Mijoz va server o'rtasida turuvchi vositachi. So'rovlarni qabul qilib, yo'naltiradi yoki filtrlaydi. |
| QUIC | UDP ustida qurilgan yangi transport protokoli. HTTP/3 asosini tashkil qiladi va ulanishni tez o'rnatadi. |
| resolver | DNS so'rovlarини bajarib, domen nomiни IP manzilга aylantiruvchi xizmat. Odatda internet provayderi taqdim etadi. |
| reverse proxy | Serverlar oldida turib, kelayotgan so'rovlarни qabul qilib taqsimlovchi vositachi. Mijozга bitta server ko'rinadi, aslida orqada ko'plari bo'lishi mumkin. |
| round robin | Yuklarni serverlar bo'ylab navbat bilan ketma-ket taqsimlash usuli. Har bir yangi so'rov navbatdagi serverга yuboriladi. |
| RTT | "Round-Trip Time" — signal borib qaytishга ketgan to'liq vaqt. Ping natijasida ko'rsatiladigan vaqt shu. |
| segment | Transport (TCP) qatlamidagi ma'lumot bo'lagi. Ichida port raqamlari va tartib raqamlari bo'ladi. |
| sliding window | Tasdiq kutmasdan jo'natish mumkin bo'lgan ma'lumot miqdorini boshqaruvchi mexanizm. "Oyna" kattaligi tarmoq holatiga qarab o'zgaradi. |
| socket | IP manzil va port birlashmasi — aloqa nuqtasi. Ikki dastur o'rtasidagi ulanishning uchi. |
| SSE | "Server-Sent Events" — server mijozга bir tomonlama uzluksiz yangilanish yuboradigan texnologiya. Yangiликlar oqimini kuzatish uchun qulay. |
| SSL | TLS'ning eski nomi va salafi — ma'lumotni shifrlash protokoli. Hozir eskirgan, o'rniga TLS ishlatiladi. |
| subnet | Katta tarmoqni kichik bo'laklarга ajratish. Manzillashtirish va boshqaruvni osonlashtiradi. |
| SYN | TCP ulanishini boshlash uchun jo'natiladigan birinchi signal. Three-way handshake shu SYN bilan boshlanadi. |
| tail latency | So'rovlarning eng sekin ozchiligiga ketgan kechikish (masalan eng yomon 1%). O'rtacha yaxshi bo'lsa ham, bu qism foydalanuvchini bezovta qilishi mumkin. |
| TCP | Ishonchli ulanishli transport protokoli. Ma'lumotni tartib bilan va yo'qotmasdan yetkazishni kafolatlaydi. |
| TCP/IP | Internetning asosini tashkil qiluvchi protokollar to'plami. TCP ishonchli yetkazishni, IP manzillashtirishни ta'minlaydi. |
| three-way handshake | TCP ulanishini o'rnatishning uch bosqichli jarayoni: SYN → SYN-ACK → ACK. Shundan so'ng ma'lumot almashinuvi boshlanadi. |
| throughput | Vaqt birligida haqiqatda o'tkazilgan ma'lumot miqdori. Ko'p bo'lsa yaxshi — tizim samaradorligini ko'rsatadi. |
| TLS | "Transport Layer Security" — ma'lumotni shifrlab himoyalovchi protokol. HTTPS aynan TLS ustida ishlaydi. |
| TTL | "Time To Live" — ma'lumot yoki yozuvning yashash muddati. DNS'da yozuv qancha vaqt keshda saqlanishini bildiradi. |
| UDP | Ulanishsiz, tez, lekin ishonchsiz transport protokoli. Yo'qotishlarga chidamli — video va o'yinlarда qo'llaniladi. |
| WebSocket | Brauzer va server o'rtasida ikki tomonlama doimiy ulanish yaratuvchi protokol. Chat va real vaqt ilovalari uchun qulay. |
| DNS | "Domain Name System" — domen nomlarini IP manzilга aylantiruvchi tizim. Internetning "telefon kitobi" kabi. |
| bit | Ma'lumotning eng kichik birligi — 0 yoki 1. Tarmoqda tezlik ko'pincha bit/soniyada o'lchanadi. |

[⬆ Mundarijaga qaytish](#mundarija)

---

## Encoding va Hashing

| Termin | Ma'nosi (o'zbekcha) |
|--------|---------------------|
| argon2 | Zamonaviy va xavfsiz parol xeshlash algoritmi. Xotira va vaqt talab qiladi, shuning uchun buzish qiyin. |
| ASCII | Harflar va belgilarни raqamlarга bog'lovchi eski standart. 128 ta belgini o'z ichiga oladi (ingliz harflari, raqamlar). |
| avalanche effect | Kirishда bitta bit o'zgarsa ham, xesh natijasi butunlay o'zgarishi. Yaxshi xesh funksiyasining muhim xususiyati. |
| Base64 | Ikkilik (binary) ma'lumotni matn belgilarига aylantiruvchi kodlash usuli. Rasm yoki fayllarni matn ичida jo'natishга yordam beradi. |
| Base64URL | Base64'ning URL'да ishlatишга moslashtirilgan varianti. `+` va `/` belgilari `-` va `_` bilan almashtiriladi. |
| bcrypt | Parollarni xavfsiz xeshlash uchun mo'ljallangan algoritm. Ataylab sekin ishlaydi, buzishni qiyinlashtiradi. |
| binary | Ikkilik sanoq sistemasi — faqat 0 va 1 dan iborat. Kompyuterlar barcha ma'lumotni shu shaklda saqlaydi. |
| bit | Ma'lumotning eng kichik birligi — 0 yoki 1 qiymatidan biri. Sakkiz bit bir baytни tashkil qiladi. |
| bitwise | Sonlar ustidа bit darajasida bajariladigan amallar (AND, OR, XOR). Har bir bit alohida qayta ishlanadi. |
| BOM | "Byte Order Mark" — fayl boshidagi maxsus belgi. Matn qaysi kodlash va bayt tartibida ekanини bildiradi. |
| byte | 8 bitdan iborat ma'lumot birligi. Odatda bitta belgini yoki kichik sonni saqlaydi. |
| Base64 decoding | Base64 matnini asl ikkilik ma'lumotга qaytarish jarayoni. Kodlashning teskarisi. |
| character encoding | Belgilarни raqamli kodlar bilan ifodalash tizimi. Masalan UTF-8 yoki ASCII shunday tizimlardir. |
| checksum | Ma'lumot to'g'riligini tekshirish uchun hisoblanadigan qisqa qiymat. Jo'natishda va qabulda solishtirilib, xato bor-yo'qligi bilinadi. |
| code point | Unicode'da har bir belgiga berilgan noyob raqam. Masalan `A` harfining code point'i U+0041. |
| collision | Ikki xil kirish uchun bir xil xesh natijasi chiqishi. Xavfsiz xesh funksiyalarда bu deyarli imkonsiz bo'lishi kerak. |
| compression | Ma'lumot hajmini kichraytirish jarayoni. Takrorlanishlarни olib tashlab, joyni tejaydi. |
| CRC | "Cyclic Redundancy Check" — ma'lumot buzilmaganини tekshiruvchi tez algoritm. Tarmoq va fayllarда xatolarни aniqlaydi. |
| cryptographic hash | Xavfsizlik uchun mo'ljallangan xesh funksiyasi. Teskarisiga qaytarib bo'lmaydi va kolliziyalarга chidamli. |
| decimal | O'nlik sanoq sistemasi — 0 dan 9 gacha raqamlar. Kundalik hayotda ishlatadigan tizimimiz. |
| decoding | Kodlangan ma'lumotни asl holiга qaytarish. Encoding'ning teskari jarayoni. |
| digest | Xesh funksiyasi chiqargan natija — ma'lumotning "barmoq izi". Odatda qat'iy uzunlikdagi qator bo'ladi. |
| encoding | Ma'lumotни bir ko'rinishdan boshqasига aylantirish. Masalan matnни baytlarга o'girish. |
| encryption | Ma'lumotни kalit yordamida o'qib bo'lmas holatга keltirish. Faqat kalitга ega bo'lgan uni ochib o'qiy oladi. |
| endianness | Baytlarning xotirada saqlanish tartibi (katta yoki kichik uchi oldinда). Big-endian va little-endian turlari bor. |
| hash | Ma'lumotни qat'iy uzunlikdagi qisqa qiymatга aylantirish natijasi. Har xil kirish har xil xesh beradi. |
| hash function | Har qanday ma'lumotни qat'iy o'lchamli xeshга o'giruvchi funksiya. Bir xil kirish doim bir xil natija beradi. |
| hex encoding | Ma'lumotни o'n oltilik (hexadecimal) belgilar bilan ifodalash. Har bir bayt ikki hex belgi bilan yoziladi. |
| hexadecimal | O'n oltilik sanoq sistemasi — 0-9 va A-F. Baytlarни ixcham ko'rsatish uchun qulay. |
| HMAC | Xesh va maxfiy kalit birgalikda ishlatiladigan xabar tasdiqlash usuli. Xabar o'zgармaganини va haqiqiyligiни tekshiradi. |
| MD5 | Eski xesh funksiyasi, endi xavfsiz emas. Kolliziyalar topilgani sabab shifrlashда ishlatilmaydi. |
| mojibake | Noto'g'ri kodlash sabab matnning buzilib, tushunarsiz belgilar bo'lib ko'rinishi. Masalan UTF-8 matnни ASCII deb o'qishда yuz beradi. |
| octal | Sakkizlik sanoq sistemasi — 0 dan 7 gacha raqamlar. Ba'zan fayl ruxsatlarини (Unix) yozishда ishlatiladi. |
| one-way function | Faqat bir tomonга hisoblash oson, teskarisига qaytarib bo'lmaydigan funksiya. Xesh funksiyalar shunday ishlaydi. |
| salt | Parolга qo'shiladigan tasodifiy qo'shimcha ma'lumot. Bir xil parollar turli xeshга ega bo'lishi uchun ishlatiladi. |
| SHA-256 | Keng qo'llaniladigan xavfsiz xesh funksiyasi, 256 bitли natija beradi. Bugungi kunda ishonchli hisoblanadi. |
| two's complement | Manfiy sonlarни ikkilik shaklда ifodalash usuli. Kompyuterlar butun sonlarни shu tizim orqali saqlaydi. |
| Unicode | Dunyodagi barcha yozuvlar belgilarни qamrab oluvchi standart. Har bir belgiга noyob code point beradi. |
| URL encoding | URL'да maxsus belgilarни `%` va hex kod bilan almashtirish. Masalan bo'sh joy `%20` bo'ladi. |
| UTF-8 | Unicode belgilarни o'zgaruvchan uzunlikdagi baytlar bilan kodlash usuli. Internetда eng ko'p ishlatiladigan kodlash. |
| UTF-16 | Unicode belgilarни 16 bitли birliklar bilan kodlash. Ba'zi belgilar ikki birlik egallaydi. |
| checksum verification | Qabul qilingan ma'lumot checksum'iни qayta hisoblab, mos kelishини tekshirish. Buzilishни aniqlash usuli. |
| bitmask | Kerakли bitlarни ajratib olish uchun ishlatiladigan bit shabloni. AND amali bilan aniq bitlarни "yoqish/o'chirish". |
| padding | Ma'lumotни kerakли o'lchamга yetkazish uchun qo'shiladigan to'ldiruvchi. Base64'да `=` belgisi shu vazifани bajaradi. |
| plaintext | Shifrlanmagan, ochiq o'qiladigan asl matn. Encryption'дан oldingi holat. |
| ciphertext | Shifrlangan, o'qib bo'lmas holatдаги matn. Encryption natijasi. |
| key | Shifrlash yoki ochishда ishlatiladigan maxfiy qiymat. Uchsiz shifrlangan ma'lumotни o'qib bo'lmaydi. |
| rainbow table | Oldindan hisoblangan xesh-parol juftliklari jadvali. Salt'siz parollarни tez buzishда ishlatiladi. |

[⬆ Mundarijaga qaytish](#mundarija)

---

## DSA

| Termin | Ma'nosi (o'zbekcha) |
|--------|---------------------|
| Adjacency list | Graf saqlashning usuli: har bir vertex uchun unga bog'langan qo'shnilar ro'yxati saqlanadi. Kam bog'lamli (sparse) graflar uchun tejamli. |
| Adjacency matrix | Graf saqlashning usuli: N×N o'lchamli jadval, katakcha ikki vertex o'rtasida bog'lam bor-yo'qligini bildiradi. Tez tekshiradi, lekin ko'p joy egallaydi. |
| Algorithm | Muammoni yechish uchun aniq, qadamma-qadam ko'rsatmalar to'plami. Masalan, ro'yxatni saralash tartibi. |
| Amortized | O'rtacha xarajat: ba'zi amallar qimmat bo'lsa-da, ko'p amallar bo'yicha o'rtacha olganda arzon chiqishi. Masalan, dynamic array'ga qo'shish. |
| Array | Bir xil turdagi elementlarni ketma-ket, indeks bo'yicha saqlaydigan tuzilma. Elementga indeks orqali darhol (O(1)) kirish mumkin. |
| Asymptotic | Kirish hajmi (n) juda kattalashganda algoritm qanday o'sishini tavsiflash usuli. Kichik detallar tashlab, asosiy o'sish tendensiyasiga qaraladi. |
| AVL tree | O'zini avtomatik muvozanatlab turadigan binary search tree. Balandligi doim log(n) atrofida bo'lib, qidiruvni tez saqlaydi. |
| Backtracking | Yechim variantlarini birma-bir sinab ko'rish; agar yo'l boshi berk chiqsa, orqaga qaytib boshqa variantni sinash. Masalan, labirintdan chiqish. |
| Base case | Rekursiyani to'xtatadigan eng oddiy holat. U bo'lmasa rekursiya cheksiz davom etib, xatoga olib keladi. |
| BFS (Breadth-First Search) | Grafni "qavatma-qavat" aylanish: avval eng yaqin qo'shnilar, keyin ularning qo'shnilari. Eng qisqa yo'lni topishda ishlatiladi. |
| Big-O | Algoritm qancha vaqt yoki xotira talab qilishini eng yomon holatda ko'rsatadigan belgi. Masalan, O(n) — kirish hajmiga to'g'ri proporsional. |
| Binary search | Saralangan ro'yxatda elementni topish usuli: har safar ro'yxatning yarmini tashlab yuborish. Juda tez — O(log n). |
| Binary tree | Har bir tugun ko'pi bilan ikkita bolaga (chap va o'ng) ega bo'lgan daraxt tuzilmasi. |
| BST (Binary Search Tree) | Binary tree turi: chapdagi qiymatlar kichik, o'ngdagilari katta bo'lib joylashadi. Bu qidiruv, qo'shish, o'chirishni tezlashtiradi. |
| Bucket | Hash table'da bir xil hash qiymatiga tushgan elementlarni saqlash uchun ajratilgan "chelak" (yacheyka). |
| Chaining | Hash table'da collision'ni hal qilish usuli: bir katakka tushgan elementlarni ro'yxat (zanjir) ko'rinishida ulash. |
| Circular buffer | O'lchami qat'iy bo'lib, oxiriga yetgach yana boshiga qaytadigan navbat. Doimiy oqim ma'lumotlari uchun qulay. |
| Collision | Ikki xil kalit hash funksiyasidan bir xil natija (bir xil katak) olganda yuzaga keladigan to'qnashuv. |
| Complete tree | Barcha darajalari to'liq to'ldirilgan (oxirgi darajadan tashqari, u chapdan boshlab to'ladi) daraxt. Heap shu tuzilishga ega. |
| Connected component | Grafda bir-biriga yo'l orqali bog'langan vertexlar guruhi. Bir komponent ichidan tashqariga chiqib bo'lmaydi. |
| Cycle | Grafda boshlang'ich vertexga qaytib keladigan yopiq yo'l. Cycle borligi ba'zi algoritmlarda muhim. |
| Data structure | Ma'lumotlarni tartibli saqlash va ular ustida samarali ishlash uchun tuzilma. Masalan, array, stack, tree. |
| Deque | Ikki tomonli navbat (double-ended queue): elementni ikkala uchidan ham qo'shish va olib tashlash mumkin. |
| DFS (Depth-First Search) | Grafni "chuqurlikka" aylanish: bir yo'ldan oxirigacha borib, keyin orqaga qaytish. Rekursiya yoki stack bilan bajariladi. |
| Dijkstra | Grafda bir vertexdan boshqalarga eng qisqa (og'irlikli) yo'lni topadigan algoritm. Manfiy og'irliklar bilan ishlamaydi. |
| Directed graph | Bog'lamlar bir tomonlama bo'lgan graf (yo'nalishli). Masalan, "A dan B ga" yo'l, lekin teskarisi bo'lmasligi mumkin. |
| Divide and conquer | Muammoni kichik qismlarga bo'lib, har birini alohida yechib, keyin natijalarni birlashtirish usuli. Masalan, merge sort. |
| Doubly linked list | Har bir tugun ham keyingi, ham oldingi tugunga ko'rsatkichga ega bo'lgan bog'langan ro'yxat. Ikki tomonga yurish mumkin. |
| Dynamic array | O'zi avtomatik kattalashadigan array (masalan JS'dagi Array). To'lgach, kattaroq joy ajratib, elementlar ko'chiriladi. |
| Dynamic programming | Muammoni takrorlanuvchi kichik masalalarga bo'lib, ularning javobini eslab qolib, qayta hisoblamaslik usuli. |
| Edge | Grafda ikki vertexni bog'lovchi bog'lam (chiziq). Og'irlikli yoki oddiy bo'lishi mumkin. |
| Eviction | To'lgan tuzilmadan (masalan cache yoki heap) eski/keraksiz elementni chiqarib tashlash. |
| Fast & slow pointers | Ro'yxat bo'ylab ikki ko'rsatkichni turli tezlikda yuritish usuli. Sikl bor-yo'qligini yoki o'rtani topishda ishlatiladi. |
| Fibonacci | Har bir son o'zidan oldingi ikkitasining yig'indisiga teng ketma-ketlik (0,1,1,2,3,5...). Rekursiya va DP misoli sifatida mashhur. |
| FIFO | "First In, First Out" — birinchi kirgan birinchi chiqadi. Navbat (queue) shu tamoyilda ishlaydi. |
| Graph | Vertexlar (nuqtalar) va ularni bog'lovchi edge'lardan iborat tuzilma. Tarmoq, xarita kabi bog'lanishlarni modellashtiradi. |
| Greedy | Har qadamda o'sha ondagi eng yaxshi variantni tanlash strategiyasi, umumiy natija haqida o'ylamay. Ba'zan optimal, ba'zan yo'q. |
| Hash function | Kalitni (masalan matnni) qat'iy o'lchamli son (indeks)ga aylantiruvchi funksiya. Hash table asosini tashkil qiladi. |
| Hash table | Kalit-qiymat juftliklarini saqlab, kalit orqali deyarli darhol (O(1)) topib beradigan tuzilma. JS'da Map/Object. |
| Heap | To'liq binary tree ko'rinishidagi tuzilma bo'lib, ildizda doim eng katta (yoki eng kichik) element turadi. Priority queue asosi. |
| Heapify | Massivni heap qoidalariga muvofiq qayta tartiblash jarayoni. |
| In-place | Qo'shimcha katta xotira ishlatmay, mavjud tuzilmaning o'zida o'zgartirish kiritish. Xotira tejaydi. |
| Inorder | Binary tree'ni aylanish tartibi: chap → tugun → o'ng. BST'da bu saralangan tartibda qiymat beradi. |
| Iteration | Tsikl (loop) yordamida amalni takror-takror bajarish. Rekursiyaga muqobil yondashuv. |
| Leaf node | Daraxtda bolasi yo'q tugun (eng chekka, oxirgi tugun). |
| LIFO | "Last In, First Out" — oxirgi kirgan birinchi chiqadi. Stack shu tamoyilda ishlaydi. |
| Linked list | Har bir element (tugun) qiymat va keyingi tugunga ko'rsatkichdan iborat ketma-ket bog'langan ro'yxat. |
| Load factor | Hash table'ning qancha to'lganini ko'rsatuvchi nisbat (elementlar soni / kataklar soni). Yuqori bo'lsa, collision ko'payadi. |
| Memoization | Rekursiv hisoblash natijalarini eslab qolib (kesh), bir xil hisobni qayta bajarmaslik. "Yuqoridan pastga" DP. |
| Merge sort | Massivni yarmiga bo'lib, har birini saralab, keyin birlashtirib saralaydigan barqaror algoritm. Har doim O(n log n). |
| Min-heap / Max-heap | Heap turi: min-heap'da ildiz eng kichik, max-heap'da eng katta element bo'ladi. |
| Monotonic stack | Elementlari doim o'suvchi yoki kamayuvchi tartibda turadigan stack. "Keyingi kattaroq element" kabi masalalarda ishlatiladi. |
| Node | Tuzilmaning bitta "yacheykasi" — qiymat va boshqa tugunlarga ko'rsatkichlarni saqlaydi (list, tree, graph'da). |
| NP-hard | Ma'lum bo'lgan tez (polinomial) yechimi yo'q, juda murakkab masalalar sinfi. Faqat taxminiy yechimlar amaliy bo'ladi. |
| Postorder | Binary tree'ni aylanish tartibi: chap → o'ng → tugun. Daraxtni o'chirishda qulay. |
| Preorder | Binary tree'ni aylanish tartibi: tugun → chap → o'ng. Daraxt nusxasini olishda ishlatiladi. |
| Prefix sum | Har bir indeksgacha bo'lgan elementlar yig'indisini oldindan hisoblab qo'yish. Oraliq yig'indini tez topadi. |
| Priority queue | Elementlar kelish tartibida emas, muhimlik (prioritet) bo'yicha chiqadigan navbat. Odatda heap asosida quriladi. |
| Queue | FIFO tamoyilida ishlaydigan navbat: elementlar orqadan qo'shilib, oldindan olinadi. |
| Quick sort | Bitta "tayanch" (pivot) element atrofida massivni bo'lib saralaydigan tez algoritm. O'rtacha O(n log n). |
| Recursion | Funksiyaning o'zini-o'zi chaqirishi orqali muammoni yechish. Har chaqiruvda masala kichrayadi, base case'da to'xtaydi. |
| Red-black tree | O'zini muvozanatlab turadigan BST turi. Har tugunga "rang" belgilanadi va qoidalar orqali balandlik nazorat qilinadi. |
| Rotation | Daraxtni muvozanatlash uchun tugunlarni qayta joylashtirish amali (chapga/o'ngga aylantirish). |
| Set | Takrorlanmaydigan elementlar to'plami. Element bor-yo'qligini tez tekshiradi. JS'da Set. |
| Sibling | Daraxtda bitta ota-ona tugunga tegishli tugunlar (aka-uka tugunlar). |
| Singly linked list | Har bir tugun faqat keyingi tugunga ko'rsatuvchi bir tomonlama bog'langan ro'yxat. |
| Sliding window | Massiv yoki matn bo'ylab qat'iy yoki o'zgaruvchan "oyna"ni siljitib, oraliqlarni tekshirish usuli. Takroriy hisobni kamaytiradi. |
| Sorting | Elementlarni ma'lum tartibga (o'sish yoki kamayish) keltirish jarayoni. |
| Space complexity | Algoritm bajarilishi uchun qancha qo'shimcha xotira kerakligini kirish hajmiga bog'liq ravishda ifodalash. |
| Stable sort | Teng qiymatli elementlarning asl tartibini saqlab qoladigan saralash. Masalan, merge sort barqaror. |
| Stack | LIFO tamoyilida ishlaydigan tuzilma: element ustidan qo'shilib, ustidan olinadi. Funksiya chaqiruvlari shu tarzda saqlanadi. |
| Stack overflow | Rekursiya juda chuqurlashib, funksiya chaqiruvlari uchun ajratilgan xotira to'lib qolganda yuzaga keladigan xato. |
| String | Belgilar (harflar) ketma-ketligidan iborat matn ma'lumot turi. |
| Subtree | Daraxtning bir tugunidan boshlanuvchi, uning barcha avlodlarini o'z ichiga olgan qismi. |
| Tabulation | DP'ning "pastdan yuqoriga" usuli: kichik masalalardan boshlab jadvalni to'ldirib, katta javobga yetish. |
| Time complexity | Algoritm bajarilishi uchun qancha vaqt (amal) kerakligini kirish hajmiga bog'liq ravishda ifodalash. |
| Topological sort | Yo'nalishli graf (DAG)dagi vertexlarni bog'liqlik tartibida ketma-ket joylashtirish. Masalan, vazifalar navbati. |
| Traversal | Tuzilmaning (tree yoki graph) barcha elementlaridan ma'lum tartibda o'tib chiqish jarayoni. |
| Tree | Ildizdan boshlanib, tarmoqlanib ketuvchi ierarxik tuzilma. Sikllarsiz bog'langan graf turi. |
| Trie | Matnlarni belgi-belgi bo'yicha saqlaydigan maxsus daraxt. Prefiks (boshlanma) bo'yicha tez qidiruv beradi. Autocomplete misoli. |
| Two pointers | Massiv yoki matn ustida ikkita ko'rsatkichni harakatlantirib ishlash usuli. Ko'p masalalarni O(n)'ga tushiradi. |
| Undirected graph | Bog'lamlari ikki tomonlama bo'lgan graf: "A–B" bog'lami "B–A" ni ham anglatadi. |
| Union-Find | Elementlar qaysi guruhga tegishliligini tez aniqlaydigan tuzilma (Disjoint Set). Graf komponentlari va sikl aniqlashda ishlatiladi. |
| Vertex | Grafning bitta nuqtasi (tugun). Odam, shahar yoki obyektni bildirishi mumkin. |
| Weighted graph | Har bir edge'ga son (og'irlik/narx) biriktirilgan graf. Masalan, shaharlar orasidagi masofa. |

[⬆ Mundarijaga qaytish](#mundarija)

---

## CS Asoslari

| Termin | Ma'nosi (o'zbekcha) |
|--------|---------------------|
| Atomic operation | Yarmida to'xtatib bo'lmaydigan, "bir butun" bajariladigan amal. O'rtada boshqa oqim aralasha olmaydi. |
| Address space | Jarayonga ajratilgan xotira manzillari to'plami. Har bir jarayon o'z alohida makonini "ko'radi". |
| Availability (uptime) | Tizimning ishlab turgan, foydalanuvchiga xizmat ko'rsatayotgan vaqti ulushi. |
| Blocking | Oqimning biror resurs yoki natijani kutib, boshqa ish qilmay to'xtab turishi. |
| Buffer | Ma'lumotlarni bir joydan ikkinchisiga o'tkazishda vaqtincha saqlab turadigan xotira sohasi. |
| Cache (CPU) | Protsessorga yaqin joylashgan, tez-tez ishlatiladigan ma'lumotlarni saqlaydigan o'ta tez xotira. Asosiy xotiradan tezroq. |
| Concurrency | Bir nechta vazifani "bir vaqtda boshlab, aralashtirib" bajarish. Ular navbatma-navbat almashib ishlashi ham mumkin. |
| Context switch | Protsessorning bir jarayon/oqimdan boshqasiga o'tishi. Joriy holat saqlanib, yangisi yuklanadi — bu vaqt talab qiladi. |
| Copy-on-write | Xotirani nusxalashni kechiktirish usuli: haqiqiy nusxa faqat o'zgartirish paytida olinadi. Tejamli. |
| CPU scheduling | Protsessor vaqtini qaysi jarayonga, qachon berishni hal qiladigan mexanizm. |
| Critical section | Kodning bir vaqtda faqat bitta oqim kirishi kerak bo'lgan qismi (umumiy resurs ishlatiladigan joy). |
| Daemon | Fonda, foydalanuvchi ko'zdan yashirin ishlaydigan tizim jarayoni. Masalan, print xizmati. |
| Deadlock | Ikki (yoki undan ko'p) oqim bir-birining resursini kutib, ikkalasi ham hech qachon davom eta olmay qotib qolishi. |
| Fork | Mavjud jarayondan yangi (bola) jarayon yaratish amali. Bola ota nusxasidan boshlanadi. |
| Fragmentation | Xotira bo'sh, lekin kichik-kichik parchalarga bo'linib ketgani sababli katta blok ajratib bo'lmasligi. |
| Garbage collection | Endi ishlatilmayotgan xotirani avtomatik topib, bo'shatib beradigan tizim. Dasturchi qo'lda o'chirmaydi. |
| Generational GC | Garbage collection'ning turi: obyektlarni "yoshi" bo'yicha guruhlab, yosh obyektlarni tez-tez tekshirish. Samaraliroq. |
| Green thread | Operatsion tizim emas, dasturiy muhit (runtime) boshqaradigan yengil oqim. |
| Heap (xotira) | Dastur ishlash paytida dinamik ravishda xotira ajratiladigan soha. Katta va uzoq yashaydigan obyektlar shu yerda. |
| Interrupt | Qurilma yoki dasturning protsessorga "menga e'tibor ber" degan signali. Joriy ish to'xtatilib, unga javob beriladi. |
| I/O bound | Ishlashi asosan disk yoki tarmoq kabi kirish/chiqishni kutishga bog'liq vazifa (protsessorga emas). |
| Kernel | Operatsion tizimning yadrosi: apparat va dasturlar o'rtasida vositachi, eng yuqori huquqlarga ega qism. |
| Kernel space | Xotiraning kernel ishlaydigan himoyalangan qismi. Oddiy dasturlar bu yerga to'g'ridan-to'g'ri kira olmaydi. |
| Livelock | Oqimlar qotib qolmaydi, lekin doim bir-biriga yo'l berib, hech qanday haqiqiy ish bajarmay aylanib qolishi. |
| Lock | Umumiy resursni bir vaqtda faqat bitta oqim ishlatishini ta'minlaydigan "qulf" mexanizmi. |
| Mark-and-sweep | Garbage collection algoritmi: avval ishlatilayotgan obyektlarni belgilash (mark), keyin belgilanmaganlarini o'chirish (sweep). |
| Memory leak | Ishlatilmay qolgan xotira bo'shatilmasdan to'planib borishi. Vaqt o'tib dastur xotirani tugatib qo'yadi. |
| Mutex | "Mutual exclusion" qulfi: bir vaqtda faqat bitta oqim kritik sohaga kirishiga ruxsat beradi. |
| Mutual exclusion | Umumiy resursga bir vaqtda faqat bitta oqim kirishi tamoyili. Poyga holatining oldini oladi. |
| Non-blocking | Oqim kutib qotib qolmasdan, resurs bo'sh bo'lmasa ham darhol qaytib ketishi. |
| Operating system | Kompyuter apparati va dasturlar o'rtasidagi asosiy boshqaruvchi dastur (Windows, Linux, macOS). |
| Page | Virtual xotira bo'linadigan qat'iy o'lchamli blok. Xotira sahifalar birligida boshqariladi. |
| Page fault | Kerakli sahifa asosiy xotirada bo'lmay, uni diskdan yuklash zarurati tug'ilgan holat. |
| Paging | Virtual xotirani sahifalarga bo'lib, kerak bo'lganini disk va RAM o'rtasida ko'chirib turadigan usul. |
| Parallelism | Bir nechta vazifani rostdan ham bir vaqtning o'zida (ko'p yadroda) bajarish. |
| Pointer | Xotiradagi qiymatning manzilini saqlaydigan o'zgaruvchi. Qiymatning o'ziga emas, "joyiga" ishora qiladi. |
| Preemptive | Rejalashtirish turi: tizim jarayonni o'zi majburan to'xtatib, boshqasiga vaqt bera oladi. |
| Priority | Jarayon yoki oqimga berilgan muhimlik darajasi. Yuqorilariga protsessor avval beriladi. |
| Process | Ishlab turgan dasturning bir nusxasi. O'z alohida xotirasi va resurslariga ega. |
| Producer-consumer | Bir oqim ma'lumot ishlab chiqarib navbatga qo'yadi, boshqasi undan olib qayta ishlaydigan naqsh. |
| Race condition | Natija oqimlarning tasodifiy bajarilish tartibiga bog'liq bo'lib qolgan xatolik. Kutilmagan buglar keltiradi. |
| Reference | Obyektga bevosita murojaat qiladigan bog'lanish (ko'rsatkichga o'xshash, lekin xavfsizroq). |
| Reference counting | Garbage collection usuli: obyektga nechta havola borligini sanab turish. Nol bo'lsa, obyekt o'chiriladi. |
| Register | Protsessor ichidagi eng tez, juda kichik xotira yacheykalari. Joriy hisob-kitob ma'lumotlari shu yerda. |
| Scheduling | Qaysi jarayon/oqim qachon va qancha protsessor vaqti olishini hal qilish jarayoni. |
| Segmentation fault | Dastur ruxsati bo'lmagan xotira manziliga kirmoqchi bo'lganda yuz beradigan halokat. |
| Semaphore | Cheklangan resursga bir vaqtda nechta oqim kira olishini sanoq bilan boshqaradigan mexanizm. |
| Spinlock | Qulf bo'sh bo'lishini kutib, oqim to'xtamasdan doimiy tekshirib turadigan qulf turi. Qisqa kutishlar uchun. |
| Stack (xotira) | Funksiya chaqiruvlari va lokal o'zgaruvchilar saqlanadigan tez, tartibli xotira sohasi. LIFO tamoyilida ishlaydi. |
| Stack frame | Har bir funksiya chaqiruvi uchun stack'da ajratiladigan blok: argumentlar, lokal o'zgaruvchilar va qaytish manzili. |
| Stack overflow | Stack xotirasi to'lib ketishi (odatda haddan tashqari chuqur rekursiyadan). Dastur ishdan chiqadi. |
| Starvation | Bir oqim doim boshqalarga o'rin berib, o'ziga hech qachon resurs tegmay ochlikda qolishi. |
| Swapping | Xotira yetishmaganda, jarayonlarni butunlay RAM va disk o'rtasida ko'chirib turish usuli. |
| System call | Dasturning kerneldan xizmat so'rashi (masalan fayl ochish, xotira olish). User va kernel o'rtasidagi "eshik". |
| Thrashing | Tizim asl ishdan ko'ra ko'proq sahifalarni disk-RAM o'rtasida ko'chirish bilan band bo'lib, sekinlashib qolishi. |
| Thread | Jarayon ichidagi mustaqil bajarilish yo'nalishi. Bitta jarayonda bir nechta oqim umumiy xotirani baham ko'radi. |
| Thread pool | Oldindan tayyorlab qo'yilgan oqimlar to'plami. Har safar yangi oqim yaratmay, tayyorlarini qayta ishlatish. |
| Thread-safe | Bir vaqtda bir nechta oqim ishlatsa ham to'g'ri ishlaydigan, xatoga olib kelmaydigan kod. |
| Time slice (quantum) | Rejalashtiruvchi har bir jarayonga bir marta beradigan qisqa protsessor vaqti bo'lagi. |
| TLB (Translation Lookaside Buffer) | Virtual manzillarni fizik manzilga tez o'girish uchun ishlatiladigan maxsus kesh. |
| User space | Xotiraning oddiy dasturlar ishlaydigan qismi. Cheklangan huquqlarga ega, kernelga to'g'ridan-to'g'ri kira olmaydi. |
| Virtual memory | Har bir dasturga o'zining uzluksiz, katta xotirasi bordek ko'rsatuvchi tizim. Aslida RAM va disk birga ishlatiladi. |
| Volatile | Qiymati kutilmaganda o'zgarishi mumkinligini bildiruvchi belgi. Kompilyator uni keshlab qo'ymaydi. |
| Yield | Oqimning protsessorni ixtiyoriy ravishda boshqa oqimga vaqtincha berib turishi. |
| Zombie process | Ishini tugatgan, lekin ota jarayoni hali natijasini olib "yig'ishtirmagan" o'lik jarayon. |

[⬆ Mundarijaga qaytish](#mundarija)

---

## DevOps

| Termin | Ma'nosi (o'zbekcha) |
|--------|---------------------|
| Artifact | Build jarayonidan chiqadigan tayyor natija (masalan, `.jar` fayl yoki Docker image). Uni saqlab, keyin deploy qilishda ishlatasan. |
| Auto-scaling | Yuk oshganda serverlar sonini avtomatik ko'paytirib, kamayganda kamaytirib turadigan mexanizm. Faqat kerak bo'lgancha resurs ishlatasan. |
| Bash | Linux'da eng ko'p ishlatiladigan buyruqlar tarjimoni (shell). Buyruq yozasan, u bajarib beradi. |
| Blue-green | Deploymentning bir usuli: ikkita bir xil muhit ("ko'k" va "yashil") bo'ladi, yangi versiyani birida sinab, tayyor bo'lgach trafikni o'sha tomonga o'tkazasan. |
| Branch | Git'da kodning alohida "shoxi" — asosiy koddan nusxa olib, unda mustaqil ishlaysan, boshqalarga xalaqit bermaysan. |
| Canary | Yangi versiyani avval foydalanuvchilarning kichik qismiga chiqarib sinash usuli. Muammo chiqsa, hammaga tarqalmaydi. |
| CD (Continuous Delivery/Deployment) | Testdan o'tgan kodni avtomatik tarzda tayyor holatga yoki to'g'ridan-to'g'ri serverga chiqarish jarayoni. |
| Cherry-pick | Boshqa branchdagi bitta aniq commitni olib, o'z branchingga ko'chirish. Butun branchni emas, faqat kerakli o'zgarishni olasan. |
| CI (Continuous Integration) | Har safar kod qo'shilganda uni avtomatik yig'ish va test qilish. Xatolarni erta topishga yordam beradi. |
| CI/CD | CI va CD birgalikda — koddan serverga qadar bo'lgan avtomatlashtirilgan yo'l (test, build, deploy). |
| CLI (Command Line Interface) | Sichqoncha o'rniga buyruqlarni matn ko'rinishida yozib ishlaydigan interfeys (terminal). |
| Cloud | Boshqa kompaniyaning serverlarini internet orqali ijaraga olib ishlatish (masalan, AWS, Google Cloud). O'z serveringni sotib olishing shart emas. |
| Commit | Git'da o'zgarishlarni saqlab, ularga "suratga olish" kabi qayd yaratish. Har bir commit tarixda qoladi. |
| ConfigMap | Kubernetes'da sozlamalar (config)ni koddan ajratib saqlaydigan obyekt. Kodni o'zgartirmasdan sozlamani almashtirasan. |
| Container | Ilovani va u ishlashi uchun kerak bo'lgan hamma narsani bitta yengil "quti"ga joylash. Har qanday joyda bir xil ishlaydi. |
| Cron | Belgilangan vaqtda avtomatik ishga tushadigan vazifalarni sozlash tizimi (masalan, har kuni soat 3:00'da backup olish). |
| chmod | Linux'da fayl yoki papkaga ruxsatlarni (o'qish/yozish/bajarish) o'zgartiradigan buyruq. |
| chown | Linux'da fayl yoki papkaning egasini (owner) o'zgartiradigan buyruq. |
| Deployment | Yozilgan kodni serverga chiqarib, foydalanuvchilar ishlata oladigan holatga keltirish. Kubernetes'da esa podlarni boshqaradigan obyekt. |
| Docker | Ilovalarni containerlarga joylashtirish va ishga tushirishning eng mashhur vositasi. |
| Docker Compose | Bir nechta containerni (masalan, ilova + baza) bitta fayl orqali birga sozlab, birgalikda ishga tushiradigan vosita. |
| Dockerfile | Docker image qanday yig'ilishini bosqichma-bosqich yozib qo'yadigan matnli fayl (retsept kabi). |
| Environment variable | Ilovaga tashqaridan beriladigan sozlama qiymati (masalan, parol yoki server manzili). Kodga yozib qo'yilmaydi. |
| Git | Kod tarixini saqlaydigan va bir nechta odam birga ishlashiga imkon beradigan version control tizimi. |
| HEAD | Git'da hozir qaysi commitda turganingni ko'rsatadigan "kursor". Odatda joriy branchning oxirgi commitiga ishora qiladi. |
| HPA (Horizontal Pod Autoscaler) | Kubernetes'da yukka qarab podlar sonini avtomatik ko'paytirib-kamaytiradigan mexanizm. |
| IaaS (Infrastructure as a Service) | Cloudda faqat "yalang'och" server, tarmoq va disk kabi asosiy resurslarni ijaraga olish. Qolganini o'zing sozlaysan. |
| IaC (Infrastructure as Code) | Serverlar va infratuzilmani qo'lda emas, kod (fayl) orqali yaratish va boshqarish. Takrorlash va kuzatish oson. |
| Image | Container ishga tushadigan tayyor "qolip" — ichida ilova va uning muhiti bor. Imagedan containerlar yaratiladi. |
| Ingress | Kubernetes'da tashqi trafikni klaster ichidagi kerakli servisga yo'naltiradigan "kirish darvozasi". |
| kubelet | Har bir Kubernetes node'ida ishlab, podlarning to'g'ri ishlashini ta'minlaydigan agent (dastur). |
| Kubernetes | Ko'plab containerlarni avtomatik boshqaradigan, ularni ishga tushiradigan va nazorat qiladigan orkestratsiya tizimi. |
| Layer | Docker imageni tashkil etuvchi qatlamlar. Har bir buyruq yangi qatlam yaratadi; o'zgarmagan qatlamlar qayta ishlatiladi (tez bo'ladi). |
| Merge | Ikki branchdagi o'zgarishlarni birlashtirish. Masalan, tayyor ishni asosiy branchga qo'shish. |
| Merge conflict | Ikki branch bir joyni har xil o'zgartirganda Git qaysi birini olishni bilmay qolgani. Uni qo'lda hal qilasan. |
| Orchestration | Ko'plab containerlarni avtomatik boshqarish jarayoni — ishga tushirish, tiklash, masshtablash (Kubernetes shu ishni qiladi). |
| Origin | Git'da masofadagi asosiy repository'ga berilgan standart nom (odatda GitHub'dagi nusxa). |
| PaaS (Platform as a Service) | Cloudda tayyor platforma olib, faqat kodni yuklaysan — server sozlash bilan shug'ullanmaysan (masalan, Heroku). |
| Permission | Fayl yoki papkaga kim nima qila olishini (o'qish/yozish/bajarish) belgilaydigan ruxsat. |
| Pipe | Bir buyruqning natijasini boshqasiga uzatuvchi belgi (`\|`). Buyruqlarni zanjir qilib bog'laysan. |
| Pipeline | CI/CD'da kod bosib o'tadigan bosqichlar ketma-ketligi (build → test → deploy). Avtomatik ishlaydi. |
| Pod | Kubernetes'da eng kichik ishga tushirish birligi — bir yoki bir nechta containerni o'z ichiga oladi. |
| Process | Hozir ishlab turgan dastur nusxasi. Har bir ishlab turgan dastur bir yoki bir nechta processdan iborat. |
| Pull request | Kodingdagi o'zgarishni asosiy branchga qo'shish uchun so'rov. Boshqalar ko'rib, tasdiqlaydi (code review). |
| Rebase | Branchdagi commitlarni boshqa branch ustiga "ko'chirib" tarixni tekislash. Merge'ga o'xshaydi, lekin tarix chiziqli chiqadi. |
| Redirection | Buyruq natijasini ekran o'rniga faylga yo'naltirish (`>`) yoki fayldan olib kelish (`<`). |
| Registry | Docker imagelar saqlanadigan va tarqatiladigan ombor (masalan, Docker Hub). |
| Remote | Git'da masofadagi (serverda turgan) repository nusxasi. Kodni u yerga yuborasan va undan olasan. |
| Repository | Loyihaning butun kodi va tarixi saqlanadigan Git ombori ("repo"). |
| Rollback | Deploy qilingan yangi versiyada muammo chiqsa, oldingi ishlaydigan versiyaga qaytish. |
| Rolling deployment | Yangi versiyani serverlarga bittalab, asta-sekin chiqarish. Xizmat to'xtamaydi, foydalanuvchi sezmaydi. |
| SaaS (Software as a Service) | Tayyor dasturni internet orqali ishlatish (masalan, Gmail). O'rnatish yoki server kerak emas. |
| Serverless | Serverni o'zing boshqarmaysan — faqat kod yozasan, u kerak bo'lganda avtomatik ishga tushadi va faqat ishlatilgani uchun to'laysan. |
| Service | Kubernetes'da podlar guruhiga barqaror manzil beradigan obyekt. Podlar almashsa ham manzil o'zgarmaydi. |
| Shell | Buyruqlaringni qabul qilib, operatsion tizimga bajartiradigan dastur (masalan, bash). |
| SSH (Secure Shell) | Masofadagi serverga xavfsiz (shifrlangan) tarzda ulanib, uni buyruqlar orqali boshqarish usuli. |
| Staging area | Git'da commitdan oldin o'zgarishlar yig'iladigan "tayyorlash zonasi". Nimani commit qilishni shu yerda tanlaysan. |
| Stash | Tayyor bo'lmagan o'zgarishlarni commit qilmasdan vaqtincha yashirib qo'yish. Keyin qaytarib olasan. |
| Terraform | IaC vositasi — serverlar va cloud resurslarini kod orqali yaratadi va boshqaradi. |
| VM (Virtual Machine) | Bitta fizik server ichida ishlaydigan "virtual kompyuter". Har birida alohida operatsion tizim bo'ladi. |
| Version control | Kod o'zgarishlarini tarixi bilan saqlab, kim nimani qachon o'zgartirganini kuzatuvchi tizim (masalan, Git). |
| Volume | Docker'da containerdan tashqarida saqlanadigan ma'lumot joyi. Container o'chsa ham ma'lumot yo'qolmaydi. |

[⬆ Mundarijaga qaytish](#mundarija)

---

## Testing

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

[⬆ Mundarijaga qaytish](#mundarija)

---

## OOP, Patterns va Clean Code

| Termin | Ma'nosi (o'zbekcha) |
|--------|---------------------|
| Abstraction | Murakkab tafsilotlarni yashirib, faqat kerakli, sodda ko'rinishni tashqariga ochish tamoyili. |
| Adapter | Bir-biriga mos kelmaydigan ikki interfeysni ulaydigan shablon — xuddi elektr rozetka o'tkazgichi kabi. |
| Anti-pattern | Ko'pincha ishlatiladigan, lekin yomon oqibatga olib keladigan yechim — "shunday qilmang" namunasi. |
| Boy Scout Rule | "Kodni topganingdan ko'ra tozaroq qoldir" — har tegishda kichik yaxshilash kiritish tamoyili. |
| Builder | Murakkab obyektni bosqichma-bosqich, qadam-baqadam qurishga imkon beruvchi yaratuvchi shablon. |
| Chain of Responsibility | So'rovni bir nechta ishlovchi orasidan zanjir bo'ylab o'tkazish — kim uddalasa, o'sha bajaradi. |
| Class | Obyektlarni yasash uchun "chizma" — qanday maydon va metodlar bo'lishini belgilaydi. |
| Code Smell | Kodda "nimadir noto'g'ri" degan ishora beruvchi belgilar (masalan juda uzun funksiya) — xato emas, lekin ogohlantirish. |
| Coupling | Modullarning bir-biriga qanchalik bog'liqligi — kam (loose) bog'liqlik yaxshi, ko'p (tight) bog'liqlik yomon. |
| Cohesion | Bir modul ichidagi qismlar bir maqsadga qanchalik jipslashganini bildiradi — yuqori cohesion yaxshi. |
| Command | So'rov yoki amalni obyektga o'rab, uni saqlash, navbatga qo'yish yoki bekor qilishga imkon beruvchi shablon. |
| Composition | Obyektni meros olmasdan, boshqa obyektlarni ichiga jamlab qurish — "meros o'rniga birlashma" tamoyili. |
| Decorator | Obyektni o'zgartirmay, unga yangi xatti-harakat qo'shadigan shablon — "o'rab" imkoniyat qo'shadi. |
| Dependency Injection | Modulga kerakli bog'liqliklarni ichida yaratmay, tashqaridan berish — moslashuvchanlik va testni osonlashtiradi. |
| Design Pattern | Tez-tez uchraydigan muammoga tayyor, sinovdan o'tgan umumiy yechim namunasi. |
| DIP (Dependency Inversion) | SOLID'ning "D"si: yuqori darajali modul quyi modulga emas, ikkalasi ham abstraksiyaga bog'liq bo'lishi kerak. |
| DRY (Don't Repeat Yourself) | Bir xil kodni takrorlamang — mantiqni bir joyda saqlab, kerakli joylardan foydalaning. |
| Encapsulation | Obyekt ichki ma'lumotini yashirib, unga faqat belgilangan metodlar orqali kirishga ruxsat berish. |
| Facade | Murakkab tizim ustiga oddiy, yagona interfeys qo'yib, foydalanishni soddalashtiradigan shablon. |
| Factory | Obyektni to'g'ridan-to'g'ri new bilan yaratmay, uni yasab beradigan maxsus metod yoki klass orqali olish shabloni. |
| Inheritance | Bir klass boshqa klassdan maydon va metodlarni meros olib, ustiga o'zini qo'shishi. |
| Interface | Klass qanday metodlarni ta'minlashi kerakligini belgilaydigan shartnoma — amalga oshirishni o'zi bermaydi. |
| ISP (Interface Segregation) | SOLID'ning "I"si: katta, umumiy interfeys o'rniga kichik, maxsus interfeyslar yaxshiroq — keraksizga majburlamang. |
| Iterator | To'plam ichki tuzilishini ochmasdan, uning elementlarini birma-bir aylanib chiqishga imkon beruvchi shablon. |
| KISS (Keep It Simple, Stupid) | Yechimni imkon qadar sodda tuting — keraksiz murakkablikdan qoching. |
| Liskov Substitution | SOLID'ning "L"si: bola klass ota klass o'rnida ishlatilganda, dastur buzilmasligi kerak. |
| LSP (Liskov Substitution Principle) | Liskov Substitution'ning to'liq nomi — subklass superklassni to'liq va xatosiz almashtira olishi kerak. |
| Mediator | Obyektlar bir-biri bilan to'g'ridan-to'g'ri emas, oraliq "vositachi" orqali muloqot qilishini ta'minlaydigan shablon. |
| Memento | Obyektning holatini saqlab qo'yib, keyin o'sha holatga qaytarishga (undo) imkon beruvchi shablon. |
| Object | Klass asosida yaratilgan aniq nusxa — o'z ma'lumoti (holati) va xatti-harakatiga ega. |
| Observer | Bir obyekt o'zgarganda, unga "obuna bo'lgan" boshqa obyektlarga avtomatik xabar beradigan shablon. |
| OCP (Open/Closed Principle) | SOLID'ning "O"si: kod kengaytirishga ochiq, lekin o'zgartirishga yopiq bo'lishi kerak. |
| OOP (Object-Oriented Programming) | Dasturni bir-biri bilan aloqada bo'lgan obyektlar to'plami sifatida qurish uslubi. |
| Polymorphism | Bir xil metod turli obyektlarda turlicha ishlashi — "bitta interfeys, ko'p shakl". |
| Proxy | Asl obyekt oldida turadigan o'rinbosar — kirishni nazorat qiladi, keshlaydi yoki kechiktirib yuklaydi. |
| Pure Function | Bir xil kirishga doim bir xil natija qaytaradigan va tashqi holatga ta'sir qilmaydigan funksiya. |
| Refactoring | Kodning tashqi xatti-harakatini o'zgartirmay, ichki tuzilishini yaxshilash jarayoni. |
| Immutability | Yaratilgandan keyin o'zgartirilmaydigan obyekt — o'zgarish kerak bo'lsa, yangi nusxa yasaladi. |
| Singleton | Klassdan butun dasturda faqat bitta nusxa bo'lishini kafolatlaydigan shablon (masalan sozlamalar obyekti). |
| SOLID | Toza obyektli dizaynning beshta asosiy tamoyili: SRP, OCP, LSP, ISP, DIP. |
| SRP (Single Responsibility) | SOLID'ning "S"si: har bir klass yoki modulning faqat bitta o'zgarish sababi (bitta mas'uliyati) bo'lishi kerak. |
| State | Obyekt ichki holatiga qarab xatti-harakatini o'zgartiradigan shablon — holat almashsa, xulqi ham almashadi. |
| Strategy | Bir vazifani bajarishning bir nechta usulini almashtiriladigan alohida obyektlarga ajratib qo'yadigan shablon. |
| Technical Debt | Tez yechim uchun sifatni qurbon qilish natijasida to'planadigan "qarz" — keyin uni tuzatish qimmatga tushadi. |
| Template Method | Algoritmning umumiy skeletini otada belgilab, ayrim qadamlarini bola klasslarga to'ldirtiradigan shablon. |
| Tight Coupling | Modullar bir-biriga juda qattiq bog'langan holat — birini o'zgartirsang, boshqasi ham buziladi (yomon holat). |
| Loose Coupling | Modullar bir-biriga kam bog'langan holat — birini o'zgartirish boshqasiga ta'sir qilmaydi (yaxshi holat). |
| Visitor | Obyektlar strukturasini o'zgartirmasdan, ularga yangi amallar qo'shishga imkon beruvchi shablon. |
| YAGNI (You Aren't Gonna Need It) | "Kerak bo'lmaydigan narsani oldindan yozma" — faqat hozir zarur funksiyani qo'sh. |

[⬆ Mundarijaga qaytish](#mundarija)

---

## System Design

| Termin | Ma'nosi (o'zbekcha) |
|--------|---------------------|
| Availability | Tizimning ishlab turgan va so'rovlarga javob berayotgan vaqti ulushi (masalan, 99.9%). |
| Back-of-envelope | Tez, taxminiy hisob-kitob: tizim qancha yuk, xotira, server talab qilishini "qo'pol" chamalash. |
| Backpressure | Tizim yukni ko'tara olmay qolganda, oqim tezligini oldingi bosqichga qaytarib sekinlatish mexanizmi. |
| Bloom filter | Element to'plamda "yo'q" ekanini aniq, "bor"ligini taxminan aytadigan tejamli tuzilma. Keraksiz qidiruvlarni kamaytiradi. |
| Bulkhead | Tizimni bo'linmalarga ajratish naqshi: bir qismning ishdan chiqishi qolganlariga tarqalmaydi (kema bo'limlaridek). |
| CAP theorem | Taqsimlangan tizim bir vaqtda Consistency, Availability va Partition tolerance'ning faqat ikkitasini to'liq bera oladi degan tamoyil. |
| Cache | Tez-tez so'raladigan ma'lumotni asosiy manbadan tezroq joyda saqlab, javobni tezlashtiruvchi qatlam. |
| Cache-aside | Keshlash naqshi: dastur avval keshni tekshiradi, topmasa bazadan olib, keyin keshga yozib qo'yadi. |
| Capacity estimation | Tizim qancha foydalanuvchi, so'rov, xotira va serverni ko'tarishi kerakligini oldindan hisoblash. |
| CDN | Kontentni foydalanuvchiga yaqin serverlarda tarqatib saqlaydigan tarmoq. Rasm, video kabi fayllarni tez yetkazadi. |
| Circuit breaker | Ishlamayotgan xizmatga so'rov yuborishni vaqtincha to'xtatuvchi "himoya avtomati". Zanjirli buzilishning oldini oladi. |
| Consistency | Barcha foydalanuvchilar bir vaqtda bir xil, eng so'nggi ma'lumotni ko'rishi holati. |
| Consistent hashing | Serverlarga ma'lumotni taqsimlash usuli: server qo'shilsa/o'chsa, ma'lumotning ozgina qismigina ko'chadi. |
| Data partitioning | Katta ma'lumotni bir nechta bo'lakka bo'lib, turli joylarda saqlash. Sharding'ga o'xshash keng tushuncha. |
| Decoupling | Tizim qismlarini bir-biridan mustaqil qilish. Biri o'zgarsa yoki buzilsa, boshqalar ta'sirlanmaydi. |
| Eventual consistency | Ma'lumot bir zumda emas, lekin bir oz vaqtdan keyin barcha nusxalarda bir xil bo'lib turadigan holat. |
| Eviction | Kesh to'lganda joy bo'shatish uchun eski/kam ishlatilgan yozuvni chiqarib tashlash. |
| Failover | Asosiy server ishdan chiqsa, avtomatik ravishda zaxira serverga o'tish. |
| Fan-out | Bitta hodisa yoki so'rovni ko'plab qabul qiluvchilarga tarqatish (masalan, post'ni barcha obunachilarga). |
| Forward proxy | Foydalanuvchi nomidan tashqi serverlarga so'rov yuboruvchi vositachi server. |
| Gossip protocol | Serverlar bir-biriga ma'lumotni "mish-mish" tarzida tarqatib, holatni sinxronlash usuli. |
| Heartbeat | Server "men tirikman" degan davriy signal yuborishi. Kelmasa, u ishdan chiqqan deb hisoblanadi. |
| Horizontal scaling | Yukni ko'tarish uchun serverlar sonini ko'paytirish ("kengaytirish"). |
| Hot key | Boshqalarga qaraganda haddan tashqari ko'p so'raladigan kalit yoki ma'lumot. Bir serverni ortiqcha yuklaydi. |
| Idempotency | Bir amalni bir necha marta bajarilishi bir marta bajarilgani bilan bir xil natija berishi. Takroriy so'rovlarni xavfsiz qiladi. |
| Latency | So'rov yuborilgandan javob kelguncha o'tgan vaqt (kechikish). Kam bo'lgani yaxshi. |
| Leaky bucket | Rate limiting algoritmi: so'rovlar "chelakka" tushib, undan doimiy tezlikda "oqib" ishlanadi. Oqimni tekislaydi. |
| Load balancer | Kelayotgan so'rovlarni bir nechta server o'rtasida teng taqsimlab yuboradigan qurilma/dastur. |
| L4 load balancing | Transport darajasida (IP va port bo'yicha) yukni taqsimlash. Tez, lekin so'rov mazmunini ko'rmaydi. |
| L7 load balancing | Ilova darajasida (URL, header, cookie bo'yicha) yukni taqsimlash. Aqlliroq, lekin sekinroq. |
| LRU | "Least Recently Used" — keshdan eng uzoq vaqt ishlatilmagan yozuvni birinchi bo'lib chiqarib tashlash siyosati. |
| Message queue | Xizmatlar o'rtasida xabarlarni vaqtincha saqlab, navbat bilan yetkazadigan oraliq. Ularni bir-biridan ajratadi. |
| Microservices | Tizimni kichik, mustaqil xizmatlarga bo'lib qurish uslubi. Har biri alohida ishlab chiqiladi va joylashtiriladi. |
| Partition tolerance | Serverlar orasidagi aloqa uzilsa ham tizim ishlashda davom eta olishi qobiliyati. |
| PACELC | CAP'ning kengaytmasi: bo'linish (P) bo'lsa A yoki C, aks holda (E) Latency yoki Consistency o'rtasida tanlov borligini aytadi. |
| Pub/sub | "Publish/Subscribe" — noshirlar xabar chiqaradi, obunachilar oladi. Ular bir-birini bilmaydi, faqat mavzu orqali bog'lanadi. |
| p50 / p95 / p99 | Kechikishning foizli o'lchovi. p99 — so'rovlarning 99% shu vaqtdan tez, faqat 1% sekin bajarilganini bildiradi. |
| QPS | "Queries Per Second" — tizim bir soniyada nechta so'rovni qayta ishlashi. Yuk o'lchovi. |
| Quorum | Taqsimlangan tizimda amalni tasdiqlash uchun kerak bo'lgan minimal nusxalar soni (ko'pchilik). |
| Rate limiting | Bir foydalanuvchi/mijoz ma'lum vaqtda yubora oladigan so'rovlar sonini cheklash. Suiiste'molni oldini oladi. |
| Read replica | Ma'lumotlar bazasining faqat o'qish uchun ishlatiladigan nusxasi. Asosiy bazadan o'qish yukini oladi. |
| Redundancy | Muhim komponentlarni zaxira nusxalash. Biri ishlamay qolsa, boshqasi o'rnini bosadi. |
| Replication | Ma'lumotni bir nechta serverda nusxalab saqlash. Ishonchlilik va o'qish tezligini oshiradi. |
| Reverse proxy | Serverlar oldida turib, tashqi so'rovlarni qabul qilib, ularni ichki serverlarga yo'naltiruvchi vositachi. |
| Scalability | Tizimning yuk ortishi bilan qo'shimcha resurs qo'shib, ishlashni saqlab qola olish qobiliyati. |
| Sharding | Ma'lumotlar bazasini bo'laklarga (shard'larga) bo'lib, turli serverlarda saqlash. Har biri ma'lumotning bir qismini olib yuradi. |
| Single point of failure | Ishdan chiqishi butun tizimni to'xtatadigan yagona zaif nuqta. Undan qochish kerak. |
| SLA | "Service Level Agreement" — xizmat sifati bo'yicha mijoz bilan rasmiy kelishuv (masalan, 99.9% ishlashi kafolati). |
| SLO | "Service Level Objective" — jamoa o'z oldiga qo'ygan ichki maqsadli sifat ko'rsatkichi. |
| Stateless | Server har bir so'rovni oldingilardan mustaqil qayta ishlashi (holatni eslamaydi). Osongina masshtablanadi. |
| Strong consistency | Yozilgan ma'lumot darhol barcha o'quvchilar uchun ko'rinadigan qat'iy izchillik. |
| Thundering herd | Ko'p so'rov bir vaqtda bir manba (masalan yangi bo'shagan kesh)ga yopirilib, tizimni yuklab qo'yishi. |
| Throughput | Tizim ma'lum vaqtda qancha ish (so'rov, ma'lumot) bajara olishi. O'tkazuvchanlik. |
| Token bucket | Rate limiting algoritmi: "chelak"ka doimiy tezlikda token to'planadi, har so'rov bitta token sarflaydi. Portlashli yukka moslashuvchan. |
| TTL | "Time To Live" — ma'lumot (masalan keshdagi yozuv) yaroqli bo'lib turadigan muddat. Tugagach o'chiriladi. |
| Vertical scaling | Bitta serverning quvvatini (CPU, RAM) oshirib yukni ko'tarish ("balandlashtirish"). |
| Write-back | Keshlash naqshi: ma'lumot avval faqat keshga yoziladi, bazaga keyinroq ko'chiriladi. Tez, lekin yo'qotish xavfi bor. |
| Write-through | Keshlash naqshi: ma'lumot bir vaqtda ham keshga, ham bazaga yoziladi. Ishonchli, lekin sekinroq. |

[⬆ Mundarijaga qaytish](#mundarija)

---

## Behavioral

| Termin | Ma'nosi (o'zbekcha) |
|--------|---------------------|
| Action | STAR usulida "A" — vaziyatni hal qilish uchun aynan sen qilgan harakatlar. |
| Base salary | Yiliga oladigan asosiy maoshing, bonus va boshqa qo'shimchalarsiz. |
| Behavioral interview | O'tmishdagi ish tajribalaring va xatti-harakating haqidagi savollar orqali seni baholaydigan intervyu. |
| Bonus | Maoshdan tashqari, natijaga qarab beriladigan qo'shimcha pul mukofoti. |
| Coding interview | Real vaqtda kod yozib, algoritm masalasini yechishing so'raladigan intervyu bosqichi. |
| Counter-offer | Kelgan taklifga javoban sen (yoki hozirgi ishoping) qo'yadigan qarshi taklif (odatda ko'proq maosh). |
| Culture fit | Nomzod kompaniyaning qadriyatlari va ish uslubiga qanchalik mos kelishini baholash. |
| Equity | Kompaniyaning bir qismiga egalik (aksiya/ulush). Ish haqining bir qismi shu ko'rinishda bo'lishi mumkin. |
| Feedback | Ishing yoki xatti-haraking haqida beriladigan fikr-mulohaza (nima yaxshi, nimani yaxshilash kerak). |
| Mentorship | Tajribaliroq odam senga yo'l ko'rsatib, o'sishingga yordam berishi. |
| Notice period | Ishdan ketishdan oldin oldindan ogohlantirish muddati (masalan, 2 hafta oldin xabar berish). |
| Offer | Kompaniyaning senga ishga qabul qilish taklifi (maosh va shartlar bilan). |
| Onboarding | Yangi ishga kirganingda seni jarayonlar, jamoa va vositalar bilan tanishtirish davri. |
| Ownership | Ishga o'z narsangdek mas'uliyat bilan yondashish — muammoni "meniki emas" demasdan hal qilish. |
| Proactive | Kutmasdan, o'zi tashabbus ko'rsatib ish qiladigan yondashuv (muammo chiqishidan oldin oldini olish). |
| Recruiter screen | Rekruter bilan dastlabki suhbat — asosiy moslikni va qiziqishingni tekshiradi. |
| Result | STAR usulida "R" — harakating natijasida erishilgan aniq natija (imkon bo'lsa raqamlar bilan). |
| Retrospective | Loyiha yoki sprintdan so'ng "nima yaxshi ketdi, nimani yaxshilaymiz" deb tahlil qiladigan jamoa uchrashuvi. |
| RSU (Restricted Stock Unit) | Vaqt o'tishi bilan (masalan, bir necha yilda) senga beriladigan kompaniya aksiyalari. |
| Salary negotiation | Maosh va shartlar haqida kompaniya bilan kelishuvga erishish uchun muzokara qilish. |
| Situation | STAR usulida "S" — voqea sodir bo'lgan vaziyat yoki kontekst. |
| Soft skills | Muloqot, jamoada ishlash, muammo yechish kabi texnik bo'lmagan shaxsiy ko'nikmalar. |
| Stakeholder | Loyiha natijasidan manfaatdor odam (masalan, mijoz, menejer, jamoa a'zosi). |
| STAR method | Behavioral savollarga tizimli javob berish usuli: Situation → Task → Action → Result. |
| System design interview | Katta tizimni (masalan, Instagram'ni) qanday qurishni loyihalash so'raladigan intervyu. |
| Take-home assignment | Uyga berib yuboriladigan amaliy vazifa — belgilangan vaqtda yechib qaytarasan. |
| Task | STAR usulida "T" — o'sha vaziyatda oldingda turgan vazifa yoki maqsad. |
| Technical screen | Texnik bilimingni tekshiradigan dastlabki (ko'pincha onlayn) intervyu bosqichi. |
| Think-aloud | Masalani yechayotganda o'z fikrlash jarayonini ovoz chiqarib aytib borish. Intervyuchi mantiqingni ko'radi. |
| Total compensation | Maosh, bonus, aksiya va boshqa imtiyozlarni qo'shgan holdagi umumiy yillik daromad. |
| Whiteboard | Doskada (yoki onlayn taxtada) kod yoki tizim sxemasini chizib tushuntirish. |

[⬆ Mundarijaga qaytish](#mundarija)

---

## Hardware

| Termin | Ma'nosi (o'zbekcha) |
|--------|---------------------|
| address bus | Xotiradаги qaysi manzилга murojaat qilishни tashuvchi simlar to'plami. CPU shu orqali kerakли joyni ko'rsatadi. |
| ALU | "Arithmetic Logic Unit" — CPU ичидаги arifmetik va mantiqiy amallarни bajaruvchi qism. Qo'shish, taqqoslash kabi ishlarни qiladi. |
| Amdahl's law | Dasturни parallellashtirishдан olinadigan tezlanishning nazariy chegarasi. Ketma-ket bajariladigan qism umumiy foydани cheklaydi. |
| branch prediction | CPU keyingi qaysi buyruq bajarilishini oldindan taxmin qilishi. To'g'ri taxmin qilsa, kutmasdan ishlashда davom etadi. |
| bus | Qurilma qismlari o'rtasида ma'lumot uzatuvchi simlar to'plami. Ma'lumotning "yo'li" kabi. |
| cache | CPU'га yaqin joylashgan tez xotira. Tez-tez kerak bo'ladigan ma'lumotни saqlab, tezlikни oshiradi. |
| cache coherence | Bir nechта yadро keshларида bir xil ma'lumotning mos turishини ta'minlash. Bir joyда o'zgarса, boshqalар ham yangilanадi. |
| cache hit | Kerakli ma'lumot keshда topilgan holat. Tez javob degani — asosiy xotираga borish shart emas. |
| cache line | Kesh bir marta o'qiydigan/yozadigan ma'lumot bloki (odatда 64 bayt). Bitta baytга emas, butun blokга ishlanadi. |
| cache miss | Kerakли ma'lumot keshда topilmagan holат. Sekinroq asosiy xotираga borishга to'g'ri keladi. |
| CISC | "Complex Instruction Set Computer" — murakkab, ko'p imkoniyatли buyruqlarга ega arxitektura. Masalan x86 shunday. |
| clock | CPU ishini boshqaruvchi ritm generatori. Har bir "taqillash" (tick)да amallar bajariladi. |
| context switch | Protsessor bir vazifадан boshqasига o'tишда holатни saqlash va tiklaш. Ko'p vazifани navbат bilan bajарish uchun kerak. |
| control unit | CPU'ni boshqaruvчi qism — buyruqларни o'qib, qaysi amал bajарилишини belgilaydi. Dirijyor kabi ishlaydi. |
| core | CPU ичидаги mustaqil hisoblash birligi (yadro). Har bir yadро alohида vazifани bajара oladi. |
| CPU | "Central Processing Unit" — kompyuterning asosiy hisoblash miyаси. Barcha buyruqларни bajaradi. |
| CUDA | NVIDIA'ning GPU'да umumiy hisoblашларни bajарish platformasi. AI va ilmiy hisoblашларда keng ishlatiladi. |
| data bus | CPU va xotира o'rtасида haqiqiy ma'lumotни tashuvchi simlar. Kengligi bir vaqtда qancha bit o'tишини belgilaydi. |
| DMA | "Direct Memory Access" — qurilmаларning CPU'ни band qilmасдан xotираga to'g'ridan-to'g'ri kirishi. CPU boshqа ishлар bilan band bo'ladi. |
| DRAM | Arzon, lekin muntazam yangилаб turишни talab qiladigan asosiy xotира turi. Kompyuterning RAM'i odatда DRAM. |
| false sharing | Turли yadроlar bir cache line'ning turли qismларини o'zgартirганда keshни ortiqcha yangилашга majbur bo'lishi. Bu ishlашни sekinlaштиради. |
| fetch-decode-execute | Buyruqни o'qish, tushunиш va bajарish — CPU'ning asosiy ish sikli. Har bir buyruq shu bosqичлардан o'tади. |
| firmware | Qurilma ичига o'rnатилган past darajали boshqaruv dasturи. Apparат bilan dasturий ta'minот o'rtасида turadi. |
| GHz | Chastota birligi — soatда sekundига necha milliard tsikl bo'lишини bildiradi. CPU tezlиги shunда o'lchanadi. |
| GPU | "Graphics Processing Unit" — grafика va parallel hisoblашларга ixtisoslashган protsessor. Minglaб kichik yadроларга ega. |
| HDD | "Hard Disk Drive" — magnit disklар asосида ma'lumот saqlash qurilmasi. Arzon, lekin SSD'дан sekinроq. |
| hyperthreading | Bitta fizik yadрони ikki mantiqий yadро kabi ko'rsатиш texnologiyasi. Intel'ning SMT versiyasi. |
| instruction cycle | Bitta buyruqни to'liq bajарish uchun ketган bosqичlar ketma-ketлиги. Fetch-decode-execute shu sikl. |
| interrupt | Qurilма CPU'га "menга e'tибор ber" deb yuborадиган signal. CPU joriy ishни to'xтатиб, unга javob beradi. |
| IOPS | "Input/Output Operations Per Second" — disk sekundига necha amал bajара олishini o'lchайди. Disk tezлигини baholашда ishlatiladi. |
| ISA | "Instruction Set Architecture" — CPU tushунадиган buyruqлар to'plami. Dasturий ta'минот bilan apparат o'rtасидаги til. |
| L1/L2/L3 | Keshning uch darajаси — L1 eng kichик va tez, L3 eng katта va sekinроq. Yaqinлигига qaraб tartибланган. |
| locality | Dastur yaqин joylашган yoki yaqinда ishlатилган ma'lumотга murojaat qilиш moyillиги. Kesh shunга tayaниб ishlaydi. |
| MESI | Kesh coherence'ни ta'минловчи holатlар protokoli (Modified, Exclusive, Shared, Invalid). Har bir cache line shu holатлардан birида bo'ladi. |
| motherboard | Barcha qismларни birlаштирувчи asosiy plata. CPU, xotира va boshqалар unга ulaнади. |
| multicore | Bir CPU ичида bir nechта yadро bo'lиши. Bir vaqtда bir nechта vazифани parallел bajарishга imkon beradi. |
| NAND flash | SSD'lар asосидаги elektron xotира turи. Harakатланувчи qism yo'q, shунга sekin emas. |
| NVMe | SSD'lар uchun tez ulaнish protokoli (PCIe orqали). Eski SATA'га nisбатан ancha yuqori tezлик beradi. |
| paging | Virtual xotираni qat'iy o'lchамли "sahifа"lар bilan boshqариш usuli. Xotирани samarали taqсимлashда yordam beradi. |
| PCIe | "PCI Express" — tez qurilмаларни (GPU, SSD) motherboard'га ulовчи yuqori tezlıkli shına. Xatlар (lane)дан iborат. |
| pipelining | Buyruqларни konveyer kabi qadamба-qadам bir-bирига ustma-ust bajарish. Bir buyruq tugамасдан keyingисини boshlaydi. |
| polling | CPU qurilма holатини takror-takror tekшиriб turиши. Interrupt'ning teskарисi — kutиб so'raб turadi. |
| RAID | Bir nechта diskни birlаштириб tezлик yoki ishончлиликни oshiриш texnologiyasi. Turли darajалари (RAID 0, 1, 5) bor. |
| RAM | "Random Access Memory" — dasturlар ishлаётган paytда ma'lumот saqловчи tez xotира. O'chirilса, ma'lumот yo'qоlади. |
| register | CPU ичидаги eng tez, eng kichик xotira yacheykаlari. Joriy amalда ishлатилаётган ma'lumотни saqlaydi. |
| RISC | "Reduced Instruction Set Computer" — sodда va tez buyruqлар to'plамига ega arxitektura. Masalan ARM shunday. |
| SIMD | "Single Instruction, Multiple Data" — bitta buyruq bilan ko'plаб ma'lumотга bir vaqtда ishlaш. Grafика va matematикада qulay. |
| SIMT | "Single Instruction, Multiple Threads" — GPU'да bitta buyruqни ko'plаб oqимlар bir vaqtда bajариши. CUDA shунга asосланади. |
| SMT | "Simultaneous Multithreading" — bir yadрода bir nechта oqимни parallел bajариш. Hyperthreading uning bir turи. |
| SRAM | Tez, lekin qimматли xotira turi. Keshда shu turdаги xotира ishlatiladi. |
| SSD | "Solid State Drive" — elektron (flash) asосидаги tez saqlaш qurilmasi. HDD'дан ancha tezроq va shovqинsiz. |
| superscalar | Bir taktда bir nechта buyruqни parallел bajара oladigan CPU. Ichида bir nechта bajарish birligи bo'ladi. |
| thread | Dasturning mustaqил bajарилиш oqимi. Bir dastур ичида bir nechта thread parallел ishلаши mumкин. |
| TLB | "Translation Lookaside Buffer" — virtual manzилларни fizик manzилга tez aylantиrуvчi kesh. Paging'ni tezlаштиради. |
| VRAM | GPU'ning o'z xotираsi — grafика ma'lumотларини saqlaydi. Tez, chunki GPU'га juda yaqın. |
| von Neumann | Buyruq va ma'lumотni bir xotирада saqlовчи kompyuter arxitekturasi. Zamonавий kompyuterlар asосини tashkil qiladi. |
| virtual memory | Fizик xotира yetмаган paytда disk yordамида qo'shимча xotira "yaratish". Dasturга ko'proq joy borдек ko'rinади. |

[⬆ Mundarijaga qaytish](#mundarija)

---

## Loyihalar

| Termin | Ma'nosi (o'zbekcha) |
|--------|---------------------|
| Agent | O'zi qaror qabul qilib, vositalarni ishlatib maqsadga erishadigan LLM asosidagi dastur. |
| Backpressure | Ma'lumot ishlab chiqarish uni qayta ishlashdan tezroq bo'lganda, tizimni ortiqcha yukdan himoya qiladigan mexanizm ("sekinlashtir" signali). |
| Blind 75 | Intervyuga tayyorlanish uchun eng muhim 75 ta LeetCode masalasi ro'yxati. |
| Boilerplate | Har safar qayta-qayta yoziladigan standart, o'zgarmaydigan kod (loyiha boshlash uchun tayyor asos). |
| Concurrency | Bir nechta vazifani navbatma-navbat, bir-biriga to'sqinlik qilmasdan olib borish (bir vaqtda bajarilayotgandek ko'rinadi). |
| CRDT (Conflict-free Replicated Data Type) | Bir nechta foydalanuvchi bir vaqtda tahrirlaganda ma'lumot avtomatik, konfliktsiz birlashadigan ma'lumot turi (real-time hamkorlik uchun). |
| CRUD | Ma'lumot bilan ishlashning 4 asosiy amali: Create (yaratish), Read (o'qish), Update (yangilash), Delete (o'chirish). |
| Data pipeline | Ma'lumotni bir joydan olib, qayta ishlab, boshqa joyga uzatadigan avtomatik oqim. |
| Drag & drop | Elementni sichqoncha bilan "ushlab olib, boshqa joyga tashlash" imkoniyati. |
| Embeddings | Matn yoki rasmni ma'no jihatidan raqamlar to'plamiga (vektorga) aylantirish. O'xshash narsalar yaqin joylashadi. |
| ETL (Extract, Transform, Load) | Ma'lumotni olish (Extract), qayta ishlash (Transform) va bazaga yuklash (Load) jarayoni. |
| Feature | Loyihaning bitta alohida funksiyasi yoki imkoniyati (masalan, "login" yoki "izlash"). |
| Fine-tuning | Tayyor LLM'ni o'z ma'lumotlaring bilan qo'shimcha o'qitib, aniq vazifaga moslashtirish. |
| LeetCode | Algoritm va data structure masalalarini yechib, intervyuga mashq qiladigan mashhur platforma. |
| LLM (Large Language Model) | Katta hajmdagi matnda o'qitilgan, matn tushunadigan va yaratadigan sun'iy intellekt modeli (masalan, Claude). |
| Load testing | Tizimga ko'p yuk (ko'p foydalanuvchi) berib, u qancha ko'tara olishini sinash. |
| MVP (Minimum Viable Product) | Ishlaydigan eng sodda versiya — asosiy g'oyani sinash uchun faqat zarur funksiyalar bilan. |
| NeetCode | LeetCode masalalarini tartibli yo'l xaritasi va video yechimlar bilan o'rgatadigan resurs. |
| Parallelism | Bir nechta vazifani rostdan ham bir vaqtning o'zida (ko'p yadroda) bajarish. |
| Portfolio | O'z loyihalaringni ko'rsatadigan to'plam — ish beruvchiga qobiliyatingni namoyish qiladi. |
| Prototype | Loyihaning dastlabki, sinov namunasi. G'oyani tez ko'rsatish uchun qilinadi, hali to'liq emas. |
| Race condition | Ikki jarayon bir resursga bir vaqtda kirgani sababli, natija tasodifiy va noto'g'ri chiqishi. |
| RAG (Retrieval-Augmented Generation) | LLM javob berishdan oldin tashqi manbadan kerakli ma'lumot topib, shunga asoslanib javob berishi. |
| Real-time | Ma'lumot deyarli darhol, kutmasdan yangilanadigan tizim (masalan, chat yoki jonli reyting). |
| Stretch goal | Asosiy vazifa bajarilgach, qo'shimcha qilinadigan "erishsa yaxshi bo'ladigan" maqsad. |
| Thread pool | Oldindan tayyorlab qo'yilgan ishchi thread'lar to'plami. Har safar yangi thread yaratmasdan, tayyorlaridan foydalaniladi. |
| Tool calling | LLM'ning o'zi tashqi funksiya yoki vositani (masalan, kalkulyator, qidiruv) chaqirib ishlata olishi. |
| Vector database | Embeddinglarni (vektorlarni) saqlab, ma'no bo'yicha o'xshash natijalarni tez topadigan maxsus ma'lumotlar bazasi. |
| Virtualization | Bitta fizik kompyuterda bir nechta virtual muhit (VM) yaratish texnologiyasi. |
| WebRTC | Brauzerlar o'rtasida to'g'ridan-to'g'ri video, audio va ma'lumot uzatishga imkon beradigan texnologiya. |
| Web scraper | Sayt sahifalaridan ma'lumotni avtomatik yig'ib oladigan dastur. |
| Worker thread | Asosiy oqimni bloklamasdan, og'ir hisob-kitobni yon tomonda bajaradigan alohida ishchi thread. |

[⬆ Mundarijaga qaytish](#mundarija)

---

## Deployment

| Termin | Ma'nosi (o'zbekcha) |
|--------|---------------------|
| A record | DNS yozuvi bo'lib, domen nomini aniq IP manzilga bog'laydi. Brauzer domen orqali serverni topa oladi. |
| Backup | Ma'lumotlarning zaxira nusxasi. Biror narsa buzilsa yoki o'chsa, shundan tiklaysan. |
| Base salary | Yiliga oladigan asosiy maoshing (bonus va boshqa qo'shimchalarsiz). |
| Build | Kodni ishga tayyor holatga keltirish jarayoni (masalan, siqib, birlashtirib tayyor fayllarga aylantirish). |
| Canonical | Bir xil sahifaning bir nechta manzili bo'lsa, qaysi biri "asosiy" ekanini qidiruv tizimlariga bildiruvchi meta belgi. |
| Certbot | Let's Encrypt'dan bepul SSL sertifikat olib, serverga avtomatik o'rnatadigan vosita. |
| CNAME | DNS yozuvi bo'lib, bir domen nomini boshqa domen nomiga bog'laydi (taxallus kabi). |
| Core Web Vitals | Google'ning sahifa tezligi va foydalanuvchi qulayligini o'lchaydigan asosiy ko'rsatkichlari. SEO'ga ta'sir qiladi. |
| CORS (Cross-Origin Resource Sharing) | Bir sayt boshqa domendagi API'ga so'rov yubora olishini boshqaradigan brauzer xavfsizlik qoidasi. |
| cPanel | Shared hostingda sayt, baza va emaillarni sichqoncha bilan boshqarish uchun grafik panel. |
| Canary | Yangi versiyani avval kichik guruh foydalanuvchilarga chiqarib sinash usuli. |
| DNS (Domain Name System) | Domen nomlarini (masalan, google.com) IP manzillarga tarjima qiladigan "internetning telefon kitobi". |
| DNS propagation | DNS o'zgarishlari butun dunyo serverlariga tarqalishi uchun ketadigan vaqt (ba'zan bir necha soatgacha). |
| Deploy | Yozilgan kodni serverga chiqarib, internetda ishlaydigan holatga keltirish. |
| Domain | Saytning internetdagi nomli manzili (masalan, example.com). |
| Fail2ban | Login urinishlarini kuzatib, ko'p marta xato kiritgan (hujum qilayotgan) IP'larni avtomatik bloklaydigan xavfsizlik vositasi. |
| Firewall | Serverga kiruvchi va chiquvchi tarmoq trafigini nazorat qilib, kerak bo'lmaganini bloklaydigan "himoya devori". |
| FTP (File Transfer Protocol) | Fayllarni kompyuter va server o'rtasida uzatish uchun ishlatiladigan eski protokol. |
| GitHub Pages | GitHub'ning bepul static sayt hosting xizmati. Repozitoriydan to'g'ridan-to'g'ri sayt chiqaradi. |
| Hosting | Sayting fayllarini saqlab, internetga ochib turadigan xizmat (server ijarasi). |
| HTTPS | HTTP'ning xavfsiz, shifrlangan versiyasi. Ma'lumot server bilan brauzer o'rtasida himoyalangan holda yuriladi. |
| Journalctl | Linux'da (systemd) tizim va servislar loglarini ko'rish uchun buyruq. |
| JSON-LD | Sahifa haqidagi tuzilgan ma'lumotni (structured data) JSON ko'rinishida yozish usuli. Qidiruv tizimlari yaxshiroq tushunadi. |
| LAMP | Web serverning klassik to'plami: Linux + Apache + MySQL + PHP. |
| LEMP | Web serverning to'plami: Linux + Nginx ("E" — engine-x) + MySQL + PHP. |
| Let's Encrypt | Bepul SSL sertifikat beradigan tashkilot. Sayting HTTPS'ga o'tishi mumkin bo'ladi. |
| Logrotate | Log fayllar juda kattalashib ketmasligi uchun ularni avtomatik bo'lib, eskilarini o'chirib turadigan vosita. |
| Meta tag | HTML'da sahifa haqidagi ma'lumotni (sarlavha, tavsif) qidiruv tizimlari va brauzerlarga bildiruvchi teg. |
| Monitoring | Server va ilovaning holatini (ishlayaptimi, yuki qancha) doimiy kuzatib turish. |
| Nameserver | Domenning DNS yozuvlarini saqlaydigan va so'rovlarga javob beradigan server. |
| Netlify | Frontend saytlarni oson deploy qiladigan mashhur hosting platformasi. |
| Nginx | Tez ishlaydigan web server va reverse proxy. Ko'p saytlarni yuritadi va trafikni taqsimlaydi. |
| Non-root user | Serverda to'liq huquqli bosh foydalanuvchi (root) emas, cheklangan huquqli oddiy foydalanuvchi. Xavfsizlik uchun ishlatiladi. |
| Open Graph | Sahifa ijtimoiy tarmoqda ulashilganda qanday ko'rinishini (rasm, sarlavha) belgilaydigan meta teglar. |
| PaaS (Platform as a Service) | Tayyor platforma — faqat kodni yuklaysan, server sozlash kerak emas (masalan, Heroku, Render). |
| pg_dump | PostgreSQL bazasining zaxira nusxasini (backup) yaratadigan buyruq. |
| PHP-FPM | PHP kodini tez ishga tushirib beradigan menejer. Nginx bilan birga ishlatiladi. |
| PM2 | Node.js ilovalarni doimiy ishlab turishini ta'minlaydigan process manager (server o'chsa qayta yoqadi). |
| Preflight | CORS'da brauzer asosiy so'rovdan oldin yuboradigan "ruxsatmi?" degan tekshiruv so'rovi (OPTIONS). |
| Process manager | Ilovani doimiy ishlab turishini nazorat qiladigan, yiqilsa qayta ishga tushiradigan vosita (masalan, PM2). |
| proxy_pass | Nginx'da kelgan so'rovni orqadagi ilovaga (masalan, Node.js portiga) uzatadigan sozlama. |
| Registrar | Domen nomini sotib olish va ro'yxatdan o'tkazish xizmatini ko'rsatuvchi kompaniya (masalan, Namecheap). |
| Reverse proxy | Foydalanuvchi so'rovini qabul qilib, orqadagi haqiqiy serverga uzatadigan oraliq server (masalan, Nginx). |
| robots.txt | Qidiruv botlariga saytning qaysi qismini indekslash mumkin-mumkin emasligini aytadigan fayl. |
| SEO (Search Engine Optimization) | Saytni qidiruv tizimlarida yuqoriroq chiqishi uchun optimallashtirish. |
| Shared hosting | Bitta serverni ko'p sayt bilan bo'lishib ishlatadigan arzon hosting turi. |
| Sitemap | Saytning barcha sahifalari ro'yxati bo'lgan fayl. Qidiruv tizimlariga sahifalarni topishga yordam beradi. |
| SPA (Single Page Application) | Bitta HTML sahifada ishlaydigan, kontent JavaScript orqali almashadigan sayt. Sahifa qayta yuklanmaydi. |
| SSH (Secure Shell) | Masofadagi serverga xavfsiz ulanib, buyruqlar orqali boshqarish protokoli. |
| SSL | Server bilan brauzer o'rtasidagi aloqani shifrlaydigan texnologiya. HTTPS shu asosida ishlaydi. |
| Static site | Oldindan tayyorlangan HTML/CSS/JS fayllardan iborat, serverda dinamik hisob-kitob talab qilmaydigan sayt. |
| Structured data | Sahifa mazmunini qidiruv tizimlari tushunadigan aniq formatda belgilash (masalan, mahsulot narxi, reyting). |
| sudo | Linux'da bir buyruqni administrator (root) huquqi bilan bajarishga imkon beruvchi buyruq. |
| TLD (Top-Level Domain) | Domenning eng oxirgi qismi (.com, .uz, .org). |
| TLS | SSL'ning zamonaviy, xavfsizroq versiyasi. Aloqani shifrlaydi (odatda "SSL" deb aytilsa ham, TLS ishlatiladi). |
| ufw (Uncomplicated Firewall) | Linux'da firewall'ni oson sozlash uchun sodda buyruq vositasi. |
| Uptime | Server yoki saytning uzluksiz ishlab turgan vaqti (foizda o'lchanadi, masalan 99.9%). |
| Vercel | Frontend (ayniqsa Next.js) saytlarni tez deploy qiladigan mashhur hosting platformasi. |
| VPS (Virtual Private Server) | Katta serverdan ajratilgan, faqat senga tegishli virtual server. To'liq nazorat beradi. |

[⬆ Mundarijaga qaytish](#mundarija)

---

## Email

| Termin | Ma'nosi (o'zbekcha) |
|--------|---------------------|
| A/B testing | Xatning ikki xil variantini turli guruhlarga yuborib, qaysi biri yaxshiroq natija berishini solishtirish usuli. |
| blacklist | Spam yuboruvchi deb topilgan IP yoki domenlar ro'yxati. Unga tushsangiz, xatlaringiz bloklanadi. |
| bounce (hard/soft) | Xat yetkazilmay qaytishi. Hard — doimiy xato (manzil yo'q), soft — vaqtinchalik (pochta to'lgan). |
| CAN-SPAM | AQShdagi email marketing qonuni. Bekor qilish havolasi va haqiqiy manzil ko'rsatishni majburiy qiladi. |
| click-through rate (CTR) | Xatni ochganlar orasidan ichidagi havolani bosganlar ulushi. Xat samaradorligini o'lchaydi. |
| complaint | Qabul qiluvchi xatingizni "spam" deb belgilashi. Ko'p bo'lsa, obro'ingizga jiddiy zarar yetadi. |
| conversion rate | Xatni olganlardan qanchasi kerakli maqsadli amalni (sotib olish, ro'yxatdan o'tish) bajargani ulushi. |
| dedicated IP | Faqat sizga tegishli, boshqalar bilan bo'lishilmagan yuborish IP manzili. Obro'ingizni o'zingiz nazorat qilasiz. |
| deliverability | Xatlaringiz spam papkasiga emas, asosiy "Inbox" ga tushib borish qobiliyati. Email marketingning asosi. |
| DKIM | Xatga qo'yiladigan raqamli imzo. U xat yo'lda o'zgartirilmaganini va haqiqiy jo'natuvchidan ekanini isbotlaydi. |
| DMARC | SPF va DKIM natijalariga qarab, tekshiruvdan o'tmagan xatlar bilan nima qilishni belgilovchi qoida. |
| double opt-in | Obunachi email kiritgach, tasdiqlash havolasi yuborilib, u bosgandagina ro'yxatga qo'shiladigan usul. Sifatli baza uchun. |
| drip campaign | Oldindan tayyorlangan xatlarni belgilangan vaqt oralig'ida avtomatik ketma-ket yuborish. Masalan xush kelibsiz seriyasi. |
| ESP | Ommaviy email yuborishni ta'minlovchi xizmat provayderi, masalan Mailchimp yoki SendGrid. |
| GDPR | Yevropa Ittifoqining shaxsiy ma'lumotlarni himoya qilish qonuni. Rozilik olmasdan email yuborishni taqiqlaydi. |
| IMAP | Xatlarni serverda saqlab, bir necha qurilmadan kirib o'qish imkonini beruvchi protokol. Xat serverda qoladi. |
| inline CSS | Uslublarni to'g'ridan-to'g'ri HTML elementi ichiga yozish. Email mijozlari alohida CSS'ni ko'pincha qo'llab-quvvatlamagani uchun zarur. |
| IP warm-up | Yangi IP manzildan asta-sekin, kam miqdordan boshlab yuborishni oshirib borish. Pochta xizmatlari ishonchini qozonadi. |
| list hygiene | Manzillar ro'yxatini muntazam tozalab, ishlamaydigan va faol bo'lmagan manzillarni olib tashlash. |
| marketing email | Reklama, aksiya yoki yangiliklarni ko'plab obunachilarga bir vaqtda yuboriladigan xat turi. |
| MDA | Xatni oxirgi bosqichda qabul qiluvchining pochta qutisiga joylashtiruvchi dastur (Mail Delivery Agent). |
| merge tag | Xat ichiga qo'yiladigan o'zgaruvchan joy, masalan {{ism}}. Har bir qabul qiluvchi uchun avtomatik to'ldiriladi. |
| MJML | Responsiv (moslashuvchan) email HTML'ini oson yozish uchun maxsus belgilash tili. Oddiy kodni murakkab HTML'ga aylantiradi. |
| MTA | Xatni bir serverdan boshqasiga uzatib yuboruvchi dastur (Mail Transfer Agent). Pochta "kuryeri" kabi. |
| MUA | Foydalanuvchi xat yozadigan va o'qiydigan dastur (Mail User Agent), masalan Gmail yoki Outlook. |
| Nodemailer | Node.js'da email yuborish uchun keng ishlatiladigan kutubxona. SMTP orqali oson xat jo'natadi. |
| open rate | Yuborilgan xatlar orasidan qanchasi ochib o'qilgani ulushi. Sarlavha jozibadorligini ko'rsatadi. |
| personalization | Har bir qabul qiluvchiga uning ismi yoki qiziqishiga moslab xatni shaxsiylashtirish. Natijani yaxshilaydi. |
| POP3 | Xatlarni serverdan qurilmaga yuklab olib, serverdan o'chiradigan eski protokol. IMAP'ning oddiyroq muqobili. |
| Postmark | Asosan tranzaksion xatlarni tez va ishonchli yetkazishga ixtisoslashgan email xizmati. |
| preheader | Xat sarlavhasidan keyin qutida ko'rinadigan qisqa oldindan ko'rish matni. Ochilish darajasiga ta'sir qiladi. |
| React Email | React komponentlari yordamida chiroyli email shablonlarini yozish imkonini beruvchi zamonaviy vosita. |
| Resend | Dasturchilar uchun mo'ljallangan zamonaviy email yuborish xizmati. Sodda API bilan mashhur. |
| reverse DNS | IP manzildan uning domen nomini aniqlash. Pochta serverlari jo'natuvchi haqiqiyligini shu orqali tekshiradi. |
| segmentation | Obunachilarni umumiy belgilari (qiziqish, hudud) bo'yicha guruhlarga ajratish. Har guruhga moslroq xat yuboriladi. |
| sender reputation | Pochta xizmatlari sizning yuboruvchi sifatingizga qo'ygan "ishonch bahosi". Deliverability'ga bevosita ta'sir qiladi. |
| SES | Amazon'ning arzon va katta hajmli email yuborish xizmati (Simple Email Service). |
| shared IP | Bir nechta yuboruvchi birgalikda ishlatadigan IP manzil. Arzon, lekin boshqalar obro'iga bog'liq bo'lasiz. |
| SMTP | Xatlarni yuborish va serverlar orasida uzatishning asosiy standart protokoli. Elektron pochtaning poydevori. |
| spam filter | Kelayotgan xatlarni tekshirib, keraksiz yoki xavfli deb topilganlarini ajratib qo'yadigan tizim. |
| SPF | Qaysi serverlar sizning domeningiz nomidan xat yuborishga haqli ekanini belgilovchi yozuv. Soxtalashtirishga qarshi. |
| transactional email | Foydalanuvchi amaliga javoban yuboriladigan alohida xat, masalan parolni tiklash yoki buyurtma tasdig'i. |
| unsubscribe | Obunachi email olishni to'xtatishi uchun havola. Qonun bo'yicha har marketing xatda bo'lishi shart. |

[⬆ Mundarijaga qaytish](#mundarija)

---

← [Bosh sahifaga qaytish](./README.md)
