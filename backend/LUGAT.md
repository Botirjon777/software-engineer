# Backend — Lug'at

Bu lug'at backend dasturlash sohasidagi asosiy texnik terminlarni o'z ichiga oladi. Har bir termin ingliz tilida qoldirilgan, izoh esa "bu nima degani" tarzida sodda o'zbek tilida berilgan. Terminlar alfavit tartibida joylashtirilgan.

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

← [Backend bo'limiga qaytish](./README.md)
