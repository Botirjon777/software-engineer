# Intervyu Jarayoni va Muzokara

Texnik bilim — bu jangning faqat yarmi. Ko'p kuchli dasturchilar intervyudan o'tolmaydi, chunki ular **jarayonni** tushunmaydi: qaysi bosqichda nima kutiladi, coding intervyuda qanday "think-aloud" qilish kerak, rekruter bilan qanday gaplashish, va eng muhimi — **maosh muzokarasini** qanday olib borish. Bu hujjatda intervyu jarayonining boshidan oxirigacha (recruiter screen → offer) har bir bosqichni, kommunikatsiya texnikalarini, CV/LinkedIn maslahatlarini va maosh muzokarasini O'zbek hamda xalqaro (remote) kontekstda ko'rib chiqamiz.

Maqsad — sizni nafaqat "to'g'ri javob beradigan", balki **birga ishlash yoqimli** nomzodga aylantirish. Intervyuer ko'pincha o'zidan so'raydi: "Men bu odam bilan har kuni ishlay olamanmi?" Sizning vazifangiz — bu savolga "ha" deb javob berishga yordam berish.

## Mundarija

- [Tushunchalar: Intervyu pipeline](#tushunchalar-intervyu-pipeline)
- [Intervyu bosqichlari](#intervyu-bosqichlari)
- [Coding intervyuga yondashuv](#coding-intervyuga-yondashuv)
- [Kommunikatsiya va think-aloud](#kommunikatsiya-va-think-aloud)
- [Intervyuerga savol berish](#intervyuerga-savol-berish)
- [CV/Rezyume va LinkedIn](#cvrezyume-va-linkedin)
- [Take-home va remote intervyu](#take-home-va-remote-intervyu)
- [Rad javobdan keyin](#rad-javobdan-keyin)
- [Maosh muzokarasi](#maosh-muzokarasi)
- [Offer'ni baholash](#offerni-baholash)
- [❓ Savol-javoblar](#-savol-javoblar)
- [Masalalar](#masalalar)

---

## Tushunchalar: Intervyu pipeline

**💡 Tushuncha:** Intervyu — bu bitta voqea emas, balki **pipeline** (voronka). Har bosqichda nomzodlarning bir qismi tushib qoladi. Sizning vazifangiz — har bosqichni alohida o'yin deb qarash va o'sha bosqich nimani tekshirayotganini tushunish.

```
100 ariza
   │
   ▼  Recruiter / CV screen   (~10-20 qoladi)
   ▼  Technical / phone screen (~5-10 qoladi)
   ▼  Coding interview         (~3-5 qoladi)
   ▼  System design            (senior uchun)
   ▼  Behavioral / culture fit
   ▼  Offer + muzokara         (1-2 nomzod)
```

**💡 Tushuncha:** Har bosqich **boshqa narsani** baholaydi. Recruiter — sizning umumiy mosligingizni va maosh kutilmangizni. Technical screen — siz "haqiqiy dasturchimi"sizmi degan filtr. Coding — muammoni yechish jarayoni. System design — kattaroq miqyosda fikrlash. Behavioral — jamoaga mosligingiz. Buni bilsangiz, har bosqichga **to'g'ri** tayyorlanasiz.

**⚠️ Ehtiyot bo'l:** Eng keng tarqalgan xato — har bosqichni "savolga to'g'ri javob berish" deb qarash. Aslida intervyuer **siz qanday fikrlayotganingizni** ko'rmoqchi. To'g'ri javob, lekin jim o'tirib yozsangiz — yomon ball. Noto'g'ri javob, lekin yaxshi reasoning bilan — ko'pincha o'tasiz.

---

## Intervyu bosqichlari

### 1. Recruiter screen (15–30 daqiqa)

Odatda telefon yoki video qo'ng'iroq. Texnik emas. Tekshiriladi:

- Tajribangiz vakansiyaga mos keladimi.
- Ish haqi kutilmangiz (real budjetga mos keladimi).
- Ish vaqti / timezone (remote uchun muhim).
- Vizа / relokatsiya (xalqaro uchun).

**⚠️ Ehtiyot bo'l:** Recruiter "kutgan maoshingiz qancha?" deb so'rasa — bu birinchi tuzoq. Bu haqda pastdagi [maosh muzokarasi](#maosh-muzokarasi) bo'limida batafsil.

### 2. Technical / phone screen (45–60 daqiqa)

Yengilroq texnik filtr. Bir-ikkita o'rta darajadagi coding masala (CoderPad/HackerRank), yoki tilingiz bo'yicha savollar. Maqsad — vaqtni behuda sarflamaslik uchun "haqiqiy dasturchi"ni filtrlash.

### 3. Coding interview (45–60 daqiqa, 1–4 round)

Asosiy texnik bosqich. DSA (arrays, strings, hash maps, trees, graphs, DP). Quyida alohida bo'limda batafsil.

### 4. System design (senior/middle+ uchun)

Katta tizimni nol'dan loyihalash: "TinyURL", "Instagram feed", "rate limiter". Trade-off'lar, scalability. (System Design bo'limiga qarang.)

### 5. Behavioral / culture fit

STAR metodi bo'yicha javoblar. Konflikt, muvaffaqiyatsizlik, liderlik. (`01-behavioral-star.md` ga qarang.)

### 6. Offer + muzokara

Hammasi yaxshi o'tsa — offer. Bu yerda haqiqiy o'yin boshlanadi.

**💡 Tushuncha:** Bosqichlar tartibi va soni kompaniyaga qarab o'zgaradi. Startaplar 2-3 bosqich, FAANG 5-7 bosqich. Hammasini bir kunda o'tkazadigan "onsite loop" ham bo'ladi.

---

## Coding intervyuga yondashuv

**💡 Tushuncha:** Coding intervyuda **jarayon — javobdan muhimroq**. Quyidagi 6 qadamli freymvorkni yodlab oling va HAR masalada ishlatish:

```
1. CLARIFY  → savolni tushun, savol ber
2. EXAMPLE  → misol(lar)ni qo'lda ishlab chiq
3. APPROACH → yondashuvni ayt (brute force → optimal)
4. CODE     → think-aloud bilan yoz
5. TEST     → kodni qo'lda "yugurtir", edge case
6. ANALYZE  → time/space complexity
```

### 1-qadam: Savolni tushun va clarify qil

Masala berilishi bilan **darhol kod yozmang**. Avval savol bering:

- Input formati qanaqa? (sorted? unique? hajmi?)
- Bo'sh input bo'lishi mumkinmi? Manfiy sonlar?
- Bir nechta to'g'ri javob bo'lsa, qaysi birini qaytaray?
- Vaqt/xotira cheklovi bormi?

**⚠️ Ehtiyot bo'l:** Savol bermay kod yozish — eng keng tarqalgan xato. Intervyuer ataylab ba'zi detallarni aytmaydi va siz so'rashingizni kutadi. So'ramasangiz — "real ishda ham requirement'ni clarify qilmaydi" degan signal.

### 2-qadam: Misol ishlab chiq

Inputga bitta konkret misol oling va kutilgan outputni qo'lda hisoblang. Bu sizning ham, intervyuerning ham masala bir xil tushunganini tasdiqlaydi.

### 3-qadam: Yondashuv (brute force → optimal)

Avval **brute force**ni ayting (hatto O(n²) bo'lsa ham): "Eng oddiy yo'l — barcha juftlarni tekshirish, bu O(n²)." Keyin: "Buni hash map bilan O(n) ga tushirsa bo'ladi." Intervyuer sizning **fikrlash yo'lingizni** ko'radi.

**💡 Tushuncha:** Hech qachon optimal yechimni darhol "tashlamang", agar topа olsangiz ham. Brute force'dan boshlash sizning sistematik fikrlayotganingizni ko'rsatadi. Keyin optimallashtirasiz.

### 4-qadam: Kod yozish (think-aloud)

Yozayotganda nima qilayotganingizni gapirib turing: "Endi hash mapni init qilaman, har element uchun complement bor-yo'qligini tekshiraman..."

### 5-qadam: Test va edge case

Kod yozib bo'lgach, uni o'zingiz "yugurtiring". Edge case'lar:

- Bo'sh massiv / null
- Bitta element
- Takrorlanuvchi elementlar
- Manfiy sonlar / overflow
- Juda katta input

**⚠️ Ehtiyot bo'l:** "Bo'ldi, ishladi" deb to'xtab qolmang. O'zingiz test qilmasangiz, intervyuer bug topadi va bu yomon. Bug'ni o'zingiz topish — kuchli signal.

### 6-qadam: Complexity tahlili

Doim time va space complexity ayting: "Bu O(n) time, O(n) space, chunki hash mapda n ta element saqlanadi."

---

## Kommunikatsiya va think-aloud

**💡 Tushuncha:** **Think-aloud** (ovoz chiqarib fikrlash) — coding intervyuning eng muhim ko'nikmasi. Intervyuer sizning miyangizni o'qiy olmaydi; faqat aytganingiz va yozganingizni ko'radi. Jim o'tirib to'g'ri yechim yozsangiz ham, "bu odam bilan ishlash qiyin" degan taassurot qoldirasiz.

Yaxshi think-aloud:

- "Men bu yerda ikkita yondashuv ko'rayapman..."
- "Hmm, bu edge case'da muammo bo'lishi mumkin, bir daqiqa..."
- "Bu yechim ishlaydi-yu, lekin O(n²). Optimallashtirsa bo'ladimi deb o'ylayapman."

**⚠️ Ehtiyot bo'l:** Tiqilib qolsangiz (stuck) — jim qolmang. "Hozir biroz tiqilib qoldim, qanday yondashsam ekan deb o'ylayapman" deyish — jimlikdan yaxshiroq. Intervyuer ko'pincha hint beradi, lekin faqat siz gaplashsangiz.

**💡 Tushuncha:** Hint berilsa — **eshiting va qabul qiling**. "O'zim qila olaman" deb hintni rad etish — qaysarlik signali. Real ishda hamkasb maslahat bersa qabul qilasiz; intervyu ham shunday.

---

## Intervyuerga savol berish

**💡 Tushuncha:** Intervyu oxirida deyarli har doim "Savollaringiz bormi?" deb so'rashadi. **"Yo'q"** deb javob bermang — bu qiziqish yo'qligini bildiradi. 2-3 ta yaxshi savol tayyorlab boring.

Yaxshi savollar:

- "Bu jamoada bir kun qanday o'tadi?"
- "Hozirgi eng katta texnik challenge nima?"
- "Code review va deploy jarayoni qanday?"
- "Mentorlik / o'sish imkoniyatlari qanday?"
- "Muvaffaqiyatni qanday o'lchaysiz bu rolda?"

**⚠️ Ehtiyot bo'l:** Birinchi suhbatda "Ta'til necha kun?", "Qachon oshiriladi?" kabi faqat-imtiyoz savollarini ko'p bermang. Ular muhim, lekin keyingi bosqichlarga / offer fazasiga qoldiring.

---

## CV/Rezyume va LinkedIn

**💡 Tushuncha:** CV — sizning birinchi (va ko'pincha yagona 6 soniyalik) taassurotingiz. Rekruter o'rtacha 6-7 soniyada CV'ni ko'zdan kechiradi. Demak u **skanerlash uchun** optimallashtirilishi kerak.

CV qoidalari:

- **1 sahifa** (5+ yil tajribasi bo'lsa 2 sahifa).
- **Impact-driven**: "X qildim" emas, "X qilib Y natijaga erishdim". Misol: "Optimized API latency by 40% by introducing Redis caching."
- **Raqamlar**: foiz, vaqt, foydalanuvchi soni. "Improved performance" — bo'sh; "reduced p99 latency from 800ms to 200ms" — kuchli.
- **Action verbs**: Built, Designed, Led, Optimized, Migrated.
- **Texnologiyalar**: aniq stack (React, Node.js, PostgreSQL), "familiar with" emas.
- **ATS-friendly**: oddiy format, jadval/grafika/rasmsiz. Ko'p kompaniyalar ATS (Applicant Tracking System) ishlatadi va murakkab format'ni o'qiy olmaydi.

**⚠️ Ehtiyot bo'l:** CV'da yolg'on yozmang. "Expert in Kubernetes" deb yozsangiz, intervyuda chuqur so'raladi. Bilmagan narsa — qizil bayroq.

### LinkedIn

- Professional foto, aniq sarlavha ("Backend Engineer | Node.js, Go").
- "Open to work" belgisini yoqing (remote ish qidirsangiz).
- Loyihalaringizni va GitHub'ni bog'lang.
- Faol bo'ling: tegishli postlarga izoh, o'z loyihalaringizni ulashing.

**💡 Tushuncha:** O'zbekistondagi dasturchilar uchun **xalqaro remote** ishlarning asosiy kanali — LinkedIn. Inglizcha to'liq profil + GitHub portfolio + ochiq "open to work" — bu kombinatsiya rekruterlar sizni topishiga yordam beradi.

---

## Take-home va remote intervyu

### Take-home assignment

Ba'zi kompaniyalar uy vazifasi beradi (kichik loyiha, 2-8 soat).

- **Talablarni o'qing** va aniq bajaring (over-engineer qilmang).
- **README yozing**: qanday ishga tushirish, qaror sabablari.
- **Test yozing** — kichik bo'lsa ham.
- **Toza commit'lar** — git history ham ko'riladi.
- Deadline'ga rioya qiling.

**⚠️ Ehtiyot bo'l:** Take-home'ni "shunchaki ishlashi kifoya" deb qarang emas. Kod sifati, struktura, nomlash, testlar — hammasi baholanadi. Bu sizning real kodingizning namunasi.

### Remote intervyu maslahatlari

- **Texnikani oldindan tekshiring**: kamera, mikrofon, internet, Zoom/Meet.
- **Fon va yorug'lik**: toza, jim, yaxshi yoritilgan joy.
- **Backup reja**: internet uzilsa — telefon hotspot tayyor tursin.
- **CoderPad/screen share**: muhitni oldindan sinab ko'ring.
- **Timezone**: vaqtni aniq tasdiqlang (UTC bilan), kechikmang.
- **Ko'z kontakti**: kameraga qarang, ekranga emas.

**💡 Tushuncha:** Remote intervyuda **kommunikatsiya yanada muhim**, chunki tana tili kam ko'rinadi. Aniqroq gapiring, pauzalar qiling, "menimcha eshitildi, davom etamanmi?" deb tasdiqlang.

---

## Rad javobdan keyin

**💡 Tushuncha:** Rad (rejection) — jarayonning normal qismi. Hatto eng kuchli dasturchilar ham ko'p rad oladi. Muhimi — undan **o'rganish**.

- **Feedback so'rang**: "Kelajak uchun nimani yaxshilashim mumkin?" Ba'zi rekruterlar aniq feedback beradi.
- **Shaxsiy qabul qilmang**: ko'pincha "fit" masalasi yoki boshqa nomzod kuchliroq edi.
- **Retrospektiva**: qaysi savolda qiynaldim? Nimani takrorlashim kerak?
- **Reapply**: ko'p kompaniyalar 6-12 oydan keyin qayta arizani qabul qiladi.

**⚠️ Ehtiyot bo'l:** Rad javobiga g'azab bilan javob yozmang. Industriya kichik — bugungi rekruter ertaga boshqa kompaniyada bo'lishi mumkin. Professional va minnatdor javob qoldiring.

---

## Maosh muzokarasi

**💡 Tushuncha:** Maosh muzokarasi — bu intervyuning eng yuqori ROI'li (qaytim) qismi. 30 daqiqalik suhbat yillik daromadingizni minglab dollarga oshirishi mumkin. Lekin ko'p dasturchilar muzokara qilishdan qo'rqadi va birinchi taklifni qabul qiladi.

### Qoida 1: Bozorni o'rganing

Muzokaradan oldin **real bozor narxini** biling:

- Levels.fyi, Glassdoor, Payscale — xalqaro maoshlar.
- LinkedIn Salary, mahalliy hamjamiyatlar — O'zbekiston/remote.
- Rol, daraja, tajriba, joylashuv (yoki remote) bo'yicha diapazon.

**💡 Tushuncha:** "Bozor narxi" — bu bitta raqam emas, **diapazon** (range). Masalan "$3000–4500/oy remote middle". O'zingizning kuchli tomonlaringizni hisobga olib, diapazonning yuqori qismini maqsad qiling.

### Qoida 2: Raqamni birinchi aytmang

Recruiter "Qancha maosh kutyapsiz?" deb so'rasa, imkon qadar **birinchi raqamni aytishdan qoching**:

- "Avval rol va mas'uliyatlarni yaxshiroq tushunsam, keyin gaplashsak. Sizning bu rol uchun budjetingiz qanaqa?"
- "Bozor narxiga mos, adolatli taklifga ochiqman. Sizning diapazoningiz qancha?"

Agar majburlasalar — **diapazon** ayting, bitta raqam emas, va o'zingiz xohlagan summani diapazonning **pastki** chegarasi qiling.

**⚠️ Ehtiyot bo'l:** Juda past raqam aytsangiz — shu summa "anchor" bo'lib qoladi va undan yuqoriga chiqish qiyin. Juda yuqori aytsangiz — voronkadan tushib qolasiz. Shuning uchun avval **ular** raqam aytishini afzal ko'ring.

### Qoida 3: Butun paketni ko'ring

Maosh — paketning bir qismi. **Total compensation** quyidagilardan iborat:

```
Total Comp = Base Salary
           + Bonus (yillik/performance)
           + Equity / Stock (RSU, options)
           + Signing bonus (bir martalik)
           + Imtiyozlar (sug'urta, ta'lim, ta'til, remote stipend)
```

**💡 Tushuncha:** Base salary'ni oshirish qiyin bo'lsa, boshqa komponentlarda muzokara qiling: signing bonus, qo'shimcha ta'til kunlari, ish jihozlari budjeti, o'rganish budjeti. Equity'ni esa ehtiyotkorlik bilan baholang — startap optsionlari ko'pincha "qog'ozdagi pul".

### Qoida 4: Counter-offer qiling

Birinchi taklif deyarli har doim **muzokara qilinadigan**. Qabul qilishga shoshilmang:

1. **Minnatdorchilik bildiring**: "Taklif uchun rahmat, juda qiziqyapman."
2. **Vaqt so'rang**: "O'ylab ko'rishga bir necha kun bera olasizmi?"
3. **Counter qiling**: bozor ma'lumotiga asoslanib. "Bozorni o'rgandim va mening tajribam $X diapazonga to'g'ri keladi. Base'ni $X ga ko'tara olamizmi?"
4. **Sabab bering**: tajriba, boshqa offer, maxsus ko'nikma.

**💡 Tushuncha:** Agar boshqa offer'ingiz bo'lsa — bu eng kuchli leverage. "Boshqa kompaniyadan $X taklif oldim, lekin sizning jamoangizni afzal ko'raman" — bu kuchli, lekin halol bo'lishi shart (yolg'on offer — xavfli o'yin).

**⚠️ Ehtiyot bo'l:** Counter-offer'ni hurmat bilan qiling, ultimatum bilan emas. "$X bo'lmasa, kelmaymen" deyish — agressiv. "$X ga yetkaza olsak, darhol imzolayman" — kollaborativ. Bitta marta counter qiling, doimiy savdolashmang.

### O'zbek / remote konteksti

- O'zbekistondan xalqaro remote ishda — maosh ko'pincha **USD**da va yashash narxiga nisbatan yuqori. Ortiqcha pasaytirmang ("men O'zbekistonda yashayman, kam ham yetadi" — bu xato).
- Contractor (B2B) vs full-time: contractor ko'proq oladi-yu, soliq/sug'urta o'zingizning zimmangizda. Buni hisobga oling.
- To'lov usuli (Payoneer, Wise, bank) va valyuta xavfini muhokama qiling.

---

## Offer'ni baholash

**💡 Tushuncha:** Eng yuqori maosh — har doim ham eng yaxshi offer emas. Offer'ni ko'p o'lchamda baholang:

| Omil | Savol |
|------|-------|
| Total comp | Base + bonus + equity barchasi qancha? |
| O'sish | Mentorlik, karyera yo'li, o'rganish bormi? |
| Jamoa/menejer | Bular bilan ishlash yoqimlimi? |
| Texnologiya | Zamonaviy stack, qiziqarli muammolar? |
| Work-life | Ish vaqti, remote, ta'til, burnout xavfi? |
| Barqarorlik | Kompaniya moliyaviy holati, funding? |
| Timezone | Remote bo'lsa — qanchalik mos? |

**⚠️ Ehtiyot bo'l:** Faqat raqamga qarab qaror qabul qilmang. Yomon menejer yoki toksik jamoa — eng yuqori maoshni ham behuda qiladi. Aksincha — kuchli mentorlik bilan biroz kamroq maosh, karyerangiz uchun yaxshiroq investitsiya bo'lishi mumkin.

---

## ❓ Savol-javoblar

### ❓ Coding intervyuda masala berilishi bilan nima qilish kerak?

**✅ Javob:** Darhol kod yozmaslik kerak. Avval savolni **clarify** qilish: input formati, cheklovlar, edge case'lar haqida savol berish. Keyin konkret **misol** ishlab chiqib, masalani to'g'ri tushunganingizni tasdiqlash. Faqat shundan keyin yondashuv aytib (brute force → optimal), kod yozishga o'tish kerak.

### ❓ Think-aloud nima va nega muhim?

**✅ Javob:** Think-aloud — fikrlash jarayonini ovoz chiqarib aytish. Intervyuer sizning miyangizni o'qiy olmaydi, faqat aytgan va yozganingizni ko'radi. Jim o'tirib to'g'ri yechim yozsangiz ham, "bu odam bilan kollaboratsiya qiyin" degan taassurot qoladi. Jarayonni ko'rsatish — to'g'ri javobdan ko'ra muhimroq, chunki real ishda ham siz jamoa bilan fikr almashasiz.

### ❓ Nega brute force'dan boshlash kerak, agar optimal yechimni bilsam ham?

**✅ Javob:** Brute force'dan boshlash sistematik fikrlashni ko'rsatadi va intervyuerga sizning yondashuvingizni kuzatish imkonini beradi. Bundan tashqari, ishlaydigan (sodda) yechim — umuman yechim yo'qligidan yaxshiroq. Agar optimallashtirishga vaqt yetmasa, hech bo'lmaganda brute force qoladi. Keyin "buni O(n) ga tushirsa bo'ladi" deb optimallashtirasiz.

### ❓ Recruiter "qancha maosh kutyapsiz?" deb so'rasa nima deyish kerak?

**✅ Javob:** Imkon qadar birinchi raqamni aytishdan qoching. "Avval rolni yaxshiroq tushunsam, keyin gaplashsak; sizning bu pozitsiya uchun budjetingiz qancha?" deb savolni qaytaring. Agar majbur qilsalar — bitta raqam emas, **diapazon** ayting va xohlagan summangizni diapazonning pastki chegarasi qiling. Birinchi raqam ko'pincha "anchor" bo'lib qoladi.

### ❓ Total compensation nima va nega faqat base salary'ga qaramaslik kerak?

**✅ Javob:** Total comp = base salary + bonus + equity/stock + signing bonus + imtiyozlar. Base'ni oshirish qiyin bo'lsa, boshqa komponentlarda (signing bonus, ta'til, jihoz budjeti) muzokara qilish mumkin. Faqat base'ga qarash to'liq rasmni bermaydi — masalan, biroz kam base, lekin yaxshi equity yoki bonus bilan kompaniya umumiy ko'proq berishi mumkin.

### ❓ Birinchi offer'ni darhol qabul qilish kerakmi?

**✅ Javob:** Yo'q. Birinchi taklif deyarli har doim muzokara qilinadigan. To'g'ri ketma-ketlik: (1) minnatdorchilik bildirish, (2) o'ylash uchun bir necha kun so'rash, (3) bozor ma'lumotiga asoslanib counter-offer qilish, (4) sabab keltirish. Shoshilib qabul qilish — pul qoldirib ketish demakdir. Ko'p kompaniyalar muzokarani kutadi va birinchi taklifni atayin pastroq beradi.

### ❓ Counter-offer'ni qanday qilish kerak — agressiv emasmi?

**✅ Javob:** Counter-offer normal va kutiladigan narsa. Muhimi — uni **kollaborativ** qilish, ultimatum emas. "$X bo'lmasa kelmayman" o'rniga "$X ga yetkaza olsak, darhol imzolayman" deyish. Bozor ma'lumoti yoki boshqa offer bilan asoslang. Bir marta counter qiling, doimiy savdolashmang. Hurmat bilan qilingan counter — professionallik signali, qizil bayroq emas.

### ❓ Intervyuer "savollaringiz bormi?" desa, nima so'rash kerak?

**✅ Javob:** Hech qachon "yo'q" demang — bu qiziqish yo'qligini bildiradi. 2-3 ta tayyor savol bering: jamoada bir kun qanday o'tadi, hozirgi eng katta texnik challenge, code review/deploy jarayoni, o'sish imkoniyatlari. Birinchi suhbatda faqat imtiyozlar (ta'til, oshirish) haqida ko'p so'ramang — ularni offer fazasiga qoldiring.

### ❓ Take-home assignment'ga qanday yondashish kerak?

**✅ Javob:** Talablarni aniq o'qib, ortiqcha murakkablashtirmasdan (over-engineer qilmasdan) bajaring. README yozing (ishga tushirish + qaror sabablari), kichik bo'lsa ham test qo'shing, toza commit'lar qiling, deadline'ga rioya qiling. Bu sizning real kodingiz namunasi — sifat, struktura, nomlash, testlar hammasi baholanadi.

### ❓ Remote intervyuga nima bilan tayyorlanish kerak?

**✅ Javob:** Texnikani oldindan tekshiring (kamera, mikrofon, internet, platform). Toza/jim fon va yaxshi yorug'lik tayyorlang. Backup reja qiling (internet uzilsa telefon hotspot). CoderPad/screen share muhitini sinab ko'ring. Timezone'ni UTC bilan aniq tasdiqlang. Kameraga qarang (ekranga emas) va remote'da kommunikatsiyani yanada aniqroq qiling, chunki tana tili kam ko'rinadi.

### ❓ Stuck (tiqilib) qolsam intervyuda nima qilay?

**✅ Javob:** Jim qolmaslik kerak. "Hozir biroz tiqilib qoldim, qanday yondashsam deb o'ylayapman" deb holatni e'lon qiling. Intervyuer ko'pincha hint beradi — uni eshiting va qabul qiling, "o'zim qila olaman" deb rad etmang. Muammoni kichikroq qismlarga bo'ling, oddiyroq versiyasini ko'rib chiqing, yoki misol ustida qo'lda ishlang. Stuck bo'lish normal; muhimi — undan qanday chiqishingiz.

### ❓ CV'ni qanday yaxshi qilish mumkin?

**✅ Javob:** 1 sahifa, impact-driven bo'lsin: "X qildim" emas, "X qilib Y natijaga erishdim" (raqamlar bilan: "reduced p99 latency from 800ms to 200ms"). Action verb'lar (Built, Led, Optimized), aniq texnologiyalar, ATS-friendly oddiy format (jadval/grafikasiz). Yolg'on yozmang — CV'dagi har bir narsa intervyuda so'ralishi mumkin.

### ❓ Rad javobidan keyin nima qilish kerak?

**✅ Javob:** Feedback so'rang ("nimani yaxshilashim mumkin?"), shaxsiy qabul qilmang (ko'pincha "fit" yoki kuchliroq nomzod masalasi), retrospektiva qiling (qaysi savolda qiynaldim?), va 6-12 oydan keyin qayta ariza topshirish mumkinligini unutmang. G'azab bilan javob yozmang — industriya kichik, professional munosabat saqlang.

### ❓ Equity / stock optsionlarini qanday baholash kerak?

**✅ Javob:** Ehtiyotkorlik bilan. Yirik public kompaniyada RSU — real qiymat. Erta bosqich startapda optsion — ko'pincha "qog'ozdagi pul" bo'lib, kompaniya muvaffaqiyatiga bog'liq. Vesting jadvali (odatda 4 yil, 1 yil cliff), strike price, kompaniya bahosi (valuation) va dilution xavfini so'rang. Optsionni "bonus" deb qarang, kafolatlangan daromad emas.

### ❓ Offer'ni faqat maosh bo'yicha baholash to'g'rimi?

**✅ Javob:** Yo'q. Offer'ni ko'p o'lchamda baholang: total comp, o'sish/mentorlik, jamoa va menejer sifati, texnologiya stack'i, work-life balance, kompaniya barqarorligi, remote bo'lsa timezone mosligi. Toksik jamoa yoki yomon menejer — eng yuqori maoshni ham behuda qiladi. Kuchli mentorlik bilan biroz kamroq maosh — uzoq muddatda yaxshiroq investitsiya bo'lishi mumkin.

---

## Masalalar

> Yechimlar: [solutions/behavioral/02-interview-process.md](../solutions/behavioral/02-interview-process.md)

1. **Pipeline xaritasi.** O'zingiz orzu qilgan kompaniyani tanlang va uning intervyu jarayonini bosqichma-bosqich (recruiter screen → offer) yozib chiqing. Har bosqich nimani tekshirishini va siz qanday tayyorlanishingizni belgilang.

2. **Coding freymvork mashqi.** Bitta o'rta darajadagi LeetCode masalasini tanlang va 6 qadamli freymvork (clarify → example → approach → code → test → analyze) bo'yicha yozma ravishda yeching. Har qadamda nima aytishingizni matnda ko'rsating.

3. **Clarifying savollar.** "Massivdan ikkita son yig'indisi target'ga teng bo'lganini toping" masalasi uchun intervyuerga beradigan kamida 5 ta clarifying savol yozing.

4. **Think-aloud skripti.** "Reverse a linked list" masalasini yechayotganda aytadigan think-aloud monologingizni to'liq yozib chiqing (kamida 8 jumla).

5. **Maosh muzokarasi role-play.** Recruiter "$2500/oy taklif qilamiz" dedi, lekin bozor narxi $3500. Counter-offer dialogini (siz va recruiter) yozing — minnatdorchilik, asoslash va counter raqamni qo'shing.

6. **Offer taqqoslash jadvali.** Ikkita farazi offer tuzing (biri yuqori base, ikkinchisi yaxshi equity + mentorlik) va yuqoridagi 7 omil bo'yicha jadvalda taqqoslab, qaysi birini va nega tanlashingizni asoslang.

7. **CV qayta yozish.** "Worked on the backend and improved performance" jumlasini impact-driven, raqamli, action-verb bilan 3 xil variantda qayta yozing.

8. **Recruiter screen savollari.** Recruiter screen'da sizga beriladigan 5 tipik savolni va ularga tayyor javoblaringizni yozing (jumladan maosh kutilmasi savoliga qanday javob berishingizni).

---

← [Behavioral bo'limiga qaytish](./README.md)
