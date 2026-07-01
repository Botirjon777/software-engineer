# CS Asoslari — Lug'at

Computer Science asoslari: operatsion tizimlar, jarayonlar, oqimlar (threads), parallellik va xotira boshqaruvi bo'yicha eng muhim texnik terminlar lug'ati. Terminlar ingliz tilida qoldirilgan, izohlar o'zbek tilida sodda tushuntirish tarzida berilgan.

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

← [CS Asoslari bo'limiga qaytish](./README.md)
