# Encoding va Hashing — Lug'at

Bu lug'at kodlash (encoding), shifrlash va xeshlash sohasidagi asosiy ingliz terminlarni o'zbek tilida sodda tushuntiradi. Termin ingliz tilida qoladi, izoh esa "bu nima degani" tarzida beriladi. Terminlar alfavit tartibida joylashtirilgan.

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

← [Encoding bo'limiga qaytish](./README.md)
