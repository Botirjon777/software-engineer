# Behavioral Savollar va STAR Metodi

Texnik intervyu kodni biladiganingni tekshiradi. Behavioral (xulq-atvor) intervyu esa boshqa narsani tekshiradi: jamoada qanday ishlaysan, muammoga qanday yondashasan, xatodan o'rganasanmi, konfliktni qanday hal qilasan. Bu hujjat behavioral intervyu nima ekanligini, nega o'tkazilishini, STAR metodini batafsil va keng tarqalgan savollarga STAR bilan namunaviy javob shablonlarini o'z ichiga oladi. Maqsad — o'zbek dasturchisi sifatida bu savollarga ishonchli va aniq javob berishga tayyorlash.

**💡 Tushuncha:** Behavioral intervyuning asosiy g'oyasi: "O'tmishdagi xatti-harakat kelajakdagi xatti-harakatni eng yaxshi bashorat qiladi" (past behavior predicts future behavior). Shuning uchun savollar "nima qilgan bo'larding?" emas, ko'pincha "qachon... shunday qilgansan?" shaklida — haqiqiy o'tmish tajribangni so'raydi.

## Mundarija

- [Behavioral intervyu nima va nega](#behavioral-intervyu-nima-va-nega)
- [STAR metodi](#star-metodi)
- [STAR namunaviy javob](#star-namunaviy-javob)
- [Keng tarqalgan savollar va yondashuv](#keng-tarqalgan-savollar-va-yondashuv)
- [Umumiy maslahatlar](#umumiy-maslahatlar)
- [Tez-tez uchraydigan xatolar](#tez-tez-uchraydigan-xatolar)
- [Savol-Javoblar](#savol-javoblar)
- [Masalalar](#masalalar)

## Behavioral intervyu nima va nega

Behavioral intervyu — nomzodning o'tmishdagi haqiqiy ish vaziyatlaridagi xatti-harakatini so'rab, uning ko'nikmalarini (soft skills) baholaydigan suhbat turi. U "Bu vaziyatda nima qilarding?" (gipotetik) emas, "Bunday vaziyat bo'lganida sen aslida nima qilgansan?" (real) deb so'raydi.

Nega kompaniyalar buni o'tkazadi:

- **Bashorat:** O'tmish xatti-harakat kelajakni eng ishonchli bashorat qiladi. Gipotetik javob "to'g'ri javob"ni aytib qo'yish oson; real tajriba esa haqiqatni ko'rsatadi.
- **Soft skills:** Texnik test jamoada ishlash, kommunikatsiya, mas'uliyat, konflikt hal qilish, o'rganish qobiliyatini ko'rsatmaydi. Bu fazilatlar uzoq muddatda kod yozishdan kam muhim emas.
- **Madaniyat mosligi (culture fit):** Nomzod kompaniya qadriyatlariga, jamoa ish uslubiga mos keladimi.

**⚠️ Ehtiyot bo'l:** "Men har doim hammasini to'g'ri qilaman", "Hech qachon konflikt bo'lmagan" kabi javoblar zaiflik belgisi — ular o'z-o'zini tahlil qila olmaslikni ko'rsatadi. Intervyuer rostgo'ylik va o'sishni qidiradi, mukammallikni emas.

## STAR metodi

STAR — behavioral savolga tuzilgan, to'liq va aniq javob berish uchun ramka. To'rt qism:

- **S — Situation (Vaziyat):** Kontekst. Qayerda, qachon, qanday loyiha, qanday muammo edi. Qisqa va aniq.
- **T — Task (Vazifa):** Senga yuklangan mas'uliyat yoki maqsad nima edi. Aynan SEN nima qilishing kerak edi.
- **A — Action (Harakat):** Sen aynan nima qilding. Bu — javobning yuragi va eng uzun qismi. "Biz" emas, "men" deb gapir.
- **R — Result (Natija):** Nima bilan tugadi. Imkon qadar raqam bilan o'lcha. Va nima o'rganding.

```
S — Vaziyat (15%):   "E-commerce loyihada checkout sahifasi sekin edi..."
T — Vazifa (10%):    "Mening vazifam yuklanish vaqtini 2 baravar kamaytirish edi..."
A — Harakat (60%):   "Men profiling qildim, N+1 query topdim, indeks qo'shdim..."
R — Natija (15%):    "Yuklanish 4s dan 1.5s ga tushdi, konversiya 12% oshdi..."
```

**💡 Tushuncha:** Action qismi javobning 60% ini tashkil qilishi kerak. Ko'p nomzodlar Situation'da uzoq qoladi va Action'ni qisqartiradi — bu eng keng tarqalgan xato. Intervyuer aynan SENING harakatlaringni bilishni xohlaydi.

## STAR namunaviy javob

**Savol:** "Texnik qiyin muammoni hal qilgan holatingizni ayting."

> **Situation:** "O'tgan yil bir fintech startup'da to'lov tizimi ustida ishlayotgandim. Production'da har kuni soat 18:00 atrofida API javob vaqti keskin oshib ketardi va ba'zi to'lovlar timeout bo'lardi."
>
> **Task:** "Backend dasturchi sifatida bu muammoni topib, bartaraf etish menga topshirildi, chunki bu bevosita foydalanuvchilar to'lovlariga ta'sir qilardi."
>
> **Action:** "Men avval monitoring (Grafana) loglarini tahlilladim va sekinlik aniq bir endpoint'da ekanini ko'rdim. Keyin shu kod yo'lini profiling qildim va har bir to'lov uchun ortiqcha database query (N+1 problem) ketayotganini topdim. Men query'larni batch'ga birlashtirdim, kerakli ustunga indeks qo'shdim va eng ko'p so'raladigan ma'lumotni Redis'da kesh qildim. O'zgarishni avval staging'da yuklash testidan o'tkazdim, keyin team lead bilan code review qilib, bosqichma-bosqich (canary) deploy qildim."
>
> **Result:** "Natijada peak vaqtidagi javob vaqti 3.2 soniyadan 600 millisekundga tushdi, timeout xatolari deyarli nolga keldi. Men bu tajribadan keyin jamoaga N+1 query'larni avtomatik aniqlaydigan lint qoidasini ham qo'shishni taklif qildim, shunda muammo kelajakda takrorlanmaydi."

E'tibor ber: aniq raqamlar (3.2s → 600ms), "men" tilida, va oxirida tizimli yaxshilanish (faqat tuzatish emas, oldini olish).

## Keng tarqalgan savollar va yondashuv

### "O'zingiz haqingizda gapiring"

Bu — intervyu boshlanishi, STAR emas. 60-90 soniyalik "professional hikoya": hozir kimsan → tajriba/kuchli tomon → nega shu lavozim. Shaxsiy hayot tafsilotlariga ketma; ishga oid bo'l.

> Shablon: "Men {N} yillik backend dasturchiman, asosan Node.js va PostgreSQL bilan ishlayman. Oxirgi loyihamda {qisqacha yutuq}. Men ayniqsa {kuchli tomon} ni yaxshi ko'raman. Shuning uchun bu lavozim menga qiziq, chunki {bog'lanish}."

### "Eng qiyin loyiha"

Texnik yoki tashkiliy qiyinchilikni tanla. STAR bilan: qiyinchilik nima edi (S/T), uni qanday yengding (A), natija va o'rganganing (R). "Qiyin"likni "imkonsiz"ga aylantirib, faqat shikoyat qilma.

### "Jamoadagi konflikt"

Diqqat — konfliktni qanday **hal qilganing**da, kim aybdor ekanida emas. Hamkasbni yomonlama. Empatiya, tinglash va kompromiss ko'rsat.

> Shablon (A qismi): "Men uni alohida chaqirib, uning nuqtai nazarini tingladim, keyin o'z dalillarimni ma'lumot (benchmark) bilan ko'rsatdim. Biz ikkalamizning g'oyalarimizdan eng yaxshisini olib, kichik prototip qildik va natijaga ko'ra qaror qabul qildik."

### "Muvaffaqiyatsizlik / xato"

Bu savol rostgo'ylik va o'rganishni tekshiradi. **Haqiqiy** xatoni ayt (lekin loyihani halokatga olib kelmagan). Eng muhimi R: undan nima o'rganding va keyin nimani o'zgartirding.

> Shablon (R qismi): "Bu mendan production'ga test'siz deploy qilmaslikni o'rgatdi. Shundan keyin men CI'ga majburiy test bosqichini qo'shdim va boshqa hech qachon bu xato takrorlanmadi."

### "Kuchli va kuchsiz tomon"

Kuchli tomon — lavozimga mos va misol bilan. Kuchsiz tomon — **haqiqiy**, lekin uni yaxshilash uchun nima qilayotganingni ko'rsat. "Men juda perfeksionistman" kabi soxta zaiflikdan qoch.

> Shablon (kuchsiz): "Ilgari boshqalarga vazifa topshirishni (delegation) qiyinlardim, hammasini o'zim qilmoqchi bo'lardim. Buni anglab, kichik vazifalarni ataylab jamoaga bera boshladim va code review orqali ishonch hosil qildim. Endi bu ancha yaxshi."

### "Nega bu kompaniya"

Kompaniya haqida tadqiqot qilganingni ko'rsat: mahsulot, missiya, texnologiya. "Pul yaxshi" emas, balki o'sish, ta'sir, qiziqish haqida gapir.

### "Deadline o'tkazib yuborgan holatingiz"

Mas'uliyatni qabul qil, ammo qanday boshqarganingni ko'rsat: erta ogohlantirish, prioritetlash, kommunikatsiya.

> Shablon: "Vazifa kutilganidan murakkabroq chiqdi. Men deadline'dan oldinroq buni manager'ga aytdim, scope'ni MVP ga qisqartirishni taklif qildim. Asosiy qism o'z vaqtida chiqdi, qolgani keyingi sprint'ga ko'chdi. Bundan keyin estimatsiyaga buffer qo'shishni o'rgandim."

### "Boshqalarni ishontirgan holatingiz"

Texnik qaror (masalan kutubxona tanlash, arxitektura) bo'yicha jamoani ishontirgan vaziyat. Dalil, ma'lumot va tinglash orqali, bosim orqali emas.

### "Feedback qabul qilish"

Tanqidiy feedback'ni qanday qabul qilishing — yetuklik belgisi. Mudofaaga o'tmaganing, o'rganib o'zgartirganingni ko'rsat.

> Shablon: "Code review'da team lead mening kodim juda murakkab ekanini aytdi. Men avval biroz xafa bo'ldim, lekin uning nuqtai nazaridan qaytib qaradim va u haq ekanini tushundim. Kodni soddalashtirdim va shundan keyin 'oddiylik'ni o'zimga prinsip qildim."

## Umumiy maslahatlar

- **Aniq misol keltir.** Umumiy gap ("Men har doim tirishaman") emas, konkret bitta hodisa.
- **"Men" deb gapir, "biz" emas.** Jamoaviy ishni hurmat qil, lekin SENING hissangni aniq ko'rsat. Intervyuer seni yollaydi, jamoani emas.
- **Raqam bilan natija ber.** "Tezroq bo'ldi" emas, "30% tez bo'ldi", "5 mingdan 50 ming foydalanuvchiga chidadi".
- **Salbiy holatdan o'rganganni ko'rsat.** Xato/muvaffaqiyatsizlik savolida — eng muhimi "nima o'rgandim va nimani o'zgartirdim".
- **Oldindan tayyorlan.** 5-6 ta kuchli hikoyani oldindan tayyorla; ular ko'p savolga moslashadi.
- **Qisqa va tuzilgan bo'l.** STAR ramkasi javobni 2-3 daqiqada ushlab turadi. Cheksiz gapirma.

**⚠️ Ehtiyot bo'l:** Yolg'on hikoya to'qima. Tajribali intervyuer follow-up savollar ("aniq qanday indeks qo'shding?", "jamoa qanday reaksiya berdi?") bilan to'qimani fosh qiladi. Haqiqiy, biroz oddiy hikoya — soxta "ajoyib" hikoyadan yaxshiroq.

## Tez-tez uchraydigan xatolar

- Situation'da uzoq qolib, Action'ni qisqartirish.
- "Biz" deb gapirib, shaxsiy hissani yo'qotish.
- Hamkasbni yoki sobiq kompaniyani yomonlash.
- Natijani raqamsiz, noaniq qoldirish.
- Soxta zaiflik ("men juda mehnatsevarman") aytish.
- Savolga javob bermay, mavzudan chetga ketish.

## Savol-Javoblar

### ❓ Behavioral intervyu nima va u nega o'tkaziladi?

**✅ Javob:** Behavioral intervyu — nomzodning o'tmishdagi real ish vaziyatlaridagi xatti-harakatini so'rab, uning soft skill'larini (jamoada ishlash, kommunikatsiya, mas'uliyat, o'rganish) baholaydi. U o'tkaziladi, chunki o'tmish xatti-harakat kelajakni eng ishonchli bashorat qiladi va texnik test ko'rsata olmaydigan fazilatlarni hamda madaniyat mosligini ochib beradi.

### ❓ STAR metodi nima?

**✅ Javob:** STAR — behavioral savolga tuzilgan javob berish ramkasi: Situation (vaziyat/kontekst), Task (sening vazifang), Action (sen aynan nima qilding — eng uzun qism), Result (natija, imkon qadar raqam bilan + o'rganganing). U javobni to'liq, fokuslangan va eslab qolinadigan qiladi.

### ❓ STAR'da qaysi qism eng muhim va nega?

**✅ Javob:** Action (taxminan 60%). Chunki intervyuer aynan SENING qanday harakat qilganingni — qanday qaror qabul qilganing, qanday hal qilganingni bilishni xohlaydi. Ko'p nomzodlar Situation'da uzoq qoladi va Action'ni qisqartiradi — bu eng keng tarqalgan xato.

### ❓ Nega "biz" emas "men" deb gapirish kerak?

**✅ Javob:** Intervyuer jamoani emas, seni yollamoqchi. "Biz qildik" deyilsa, sening shaxsiy hissang noaniq qoladi. Jamoaviy ishni hurmat qilish kerak, lekin SEN aynan nima qilganingni "men" tilida aniq ko'rsatish kerak.

### ❓ "Muvaffaqiyatsizlik haqida gapiring" savoliga qanday javob berasiz?

**✅ Javob:** Haqiqiy, lekin halokatli bo'lmagan xatoni tanla. STAR bilan ayt va eng ko'p e'tiborni Result'ga ber: undan nima o'rganding va keyin nimani o'zgartirding. Maqsad — rostgo'ylik va o'sishni ko'rsatish. "Hech qachon xato qilmaganman" — eng yomon javob.

### ❓ Kuchsiz tomon haqida so'rashganda nima deysiz?

**✅ Javob:** Haqiqiy, lekin lavozim uchun halokatli bo'lmagan zaiflikni ayt va uni yaxshilash uchun nima qilayotganingni ko'rsat. Soxta zaiflikdan ("men juda perfeksionistman") qoch — bu intervyuerga sun'iy ko'rinadi va o'z-o'zini tahlil qila olmaslikni bildiradi.

### ❓ Jamoadagi konflikt savolida nimaga e'tibor berish kerak?

**✅ Javob:** Konfliktni qanday HAL qilganingga, kim aybdor ekanini ko'rsatishga emas. Hamkasbni yomonlama. Empatiya, tinglash, dalil bilan ishontirish va kompromiss qobiliyatini ko'rsat. Natija — ikkala tomon uchun ham yaxshi bo'lgan yechim.

### ❓ "O'zingiz haqingizda gapiring" — STAR ishlatiladimi?

**✅ Javob:** Yo'q. Bu — qisqa (60-90 soniya) professional hikoya: hozir kimsan, tajriba va kuchli toming, nega shu lavozim. Shaxsiy hayot tafsilotlariga ketmaslik kerak; ishga oid, lavozimga bog'langan bo'lishi kerak.

### ❓ "Nega bu kompaniya?" savoliga qanday tayyorlanasiz?

**✅ Javob:** Kompaniya haqida oldindan tadqiqot qil: mahsulot, missiya, texnologiya stack'i, yaqindagi yangiliklar. Javobni o'z qiziqishing va o'sishing bilan bog'la. "Maosh yaxshi" emas, balki ta'sir ko'rsatish, o'rganish va mahsulotga qiziqish haqida gapir.

### ❓ Natijani raqam bilan ko'rsatish nega muhim?

**✅ Javob:** Raqam hikoyani aniq va ishonarli qiladi. "Tezroq bo'ldi" — noaniq; "javob vaqti 3s dan 600ms ga tushdi" — ishonchli va o'lchanadigan. Raqam sening ta'siringni isbotlaydi va javobni esda qoladigan qiladi.

### ❓ Deadline o'tkazib yuborgan holat haqida qanday gapirasiz?

**✅ Javob:** Mas'uliyatni qabul qil, ammo qanday boshqarganingni ko'rsat: muammoni erta sezish, manager'ni vaqtida ogohlantirish, scope'ni prioritetlash (MVP), shaffof kommunikatsiya. Result'da bundan keyin estimatsiyani qanday yaxshilaganingni ayt.

### ❓ Feedback qabul qilish savoli nimani tekshiradi?

**✅ Javob:** Yetuklik va o'sishga ochiqlikni. Tanqidiy feedback'ga mudofaaga o'tmasdan, tinglab, undan o'rganib, xatti-harakatni o'zgartirganingni ko'rsat. Bu — yaxshi jamoa a'zosi belgisi.

### ❓ Intervyuga necha hikoya tayyorlash kerak?

**✅ Javob:** 5-6 ta kuchli, har xil hikoya yetarli: bittasi texnik qiyinchilik, bittasi konflikt, bittasi muvaffaqiyatsizlik/o'rganish, bittasi liderlik/ishontirish, bittasi katta yutuq. Bir hikoyani bir nechta savolga moslashtirib aytish mumkin.

### ❓ Behavioral savolga javob qancha davom etishi kerak?

**✅ Javob:** Taxminan 2-3 daqiqa. STAR ramkasi javobni tuzilgan va vaqt jihatdan nazoratda ushlaydi. Juda qisqa javob (1 jumla) yetarli kontekst bermaydi; cheksiz gapirish esa fokusni yo'qotadi.

### ❓ Tajribang kam bo'lsa (junior), behavioral savolga nima deysiz?

**✅ Javob:** Ish tajribasi bo'lmasa ham, universitet loyihalari, jamoaviy hackathon, open-source hissa, o'qish jarayonidagi vaziyatlardan misol keltir. STAR baribir ishlaydi. Muhimi — qiyinchilik, harakat va o'rganganni ko'rsatish; loyiha hajmi emas.

## Masalalar

> Yechimlar: [yechimlarni ko'rish](../solutions/behavioral/01-behavioral-star.md)

1. **O'zingiz haqingizda.** O'zingiz uchun 60-90 soniyalik "professional hikoya" yozing (hozir kimsiz → tajriba/kuchli tom → nega bu lavozim).

2. **Texnik qiyinchilik (STAR).** "Eng qiyin texnik muammoni hal qilgan holatingiz"ga to'liq STAR javob yozing. Action 60% bo'lsin, Result raqam bilan tugasin.

3. **Konflikt (STAR).** "Jamoadagi a'zo bilan kelishmovchilik"ka STAR javob yozing. Hamkasbni yomonlamasdan, qanday hal qilganingizni ko'rsating.

4. **Muvaffaqiyatsizlik (STAR).** "Katta xato qilgan holatingiz"ga STAR javob yozing. Result qismida nima o'rganganingiz va keyin nimani o'zgartirganingizni aniq yozing.

5. **Kuchsiz tomon.** Haqiqiy bitta kuchsiz tomoningizni va uni yaxshilash uchun qilayotgan ishingizni STAR ohangida yozing. Soxta zaiflikdan qoching.

6. **Deadline.** "Deadline'ni o'tkazib yuborgan yoki o'tkazib yuborish xavfi bo'lgan holat"ga STAR javob yozing. Erta kommunikatsiya va prioritetlashni ko'rsating.

7. **Ishontirish.** "Texnik qaror bo'yicha jamoani ishontirgan holatingiz"ga STAR javob yozing. Dalil va ma'lumot bilan, bosim bilan emas.

8. **Feedback.** "Tanqidiy feedback olgan va undan o'rgangan holatingiz"ga STAR javob yozing.

---

← [Behavioral bo'limiga qaytish](./README.md)
