# Database — Lug'at

Bu lug'at ma'lumotlar bazalari sohasidagi asosiy texnik terminlarni o'z ichiga oladi. Har bir termin ingliz tilida qoldirilgan, izoh esa "bu nima degani" tarzida sodda o'zbek tilida berilgan. Terminlar alfavit tartibida joylashtirilgan.

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

← [Database bo'limiga qaytish](./README.md)
