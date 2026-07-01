# Frontend — Lug'at

Brauzerda ishlaydigan texnologiyalar bo'yicha eng ko'p uchraydigan inglizcha terminlar va ularning sodda o'zbekcha izohi.

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

← [Frontend bo'limiga qaytish](./README.md)
