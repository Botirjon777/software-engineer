# Behavioral Savollar va STAR Metodi — Masalalar Yechimi

Bu fayl `01-behavioral-star.md` mashqlari uchun to'liq namunaviy javoblar. Javoblar o'zbek dasturchisi kontekstida yozilgan — sizniki emas, **namuna**. O'z haqiqiy tajribangiz bilan almashtiring, lekin tuzilish va ohangni saqlang.

**Eslatma:** Bu javoblar shablon. Eng yaxshi javob — sizning haqiqiy tajribangiz. Raqamlarni va tafsilotlarni o'zingiznikiga moslashtiring.

## 1. "O'zingiz haqingizda" — professional hikoya

> "Men 4 yillik backend dasturchiman, asosan Node.js, TypeScript va PostgreSQL bilan ishlayman. Oxirgi 2 yilda Toshkentdagi fintech kompaniyada to'lov tizimlari ustida ishladim — u yerda kunlik 50 mingdan ortiq tranzaksiyani qayta ishlaydigan API'larni qurish va optimallashtirishda qatnashdim. Men ayniqsa tizim ishonchliligi va performance optimizatsiyasini yaxshi ko'raman — sekin endpoint'ni topib, uni bir necha barobar tezlashtirish menga zavq beradi. Sizning kompaniyangiz scale'da ishlaydigan mahsulot qurayotgani uchun, bu lavozim mening tajribam va qiziqishimga juda mos keladi."

Nima yaxshi: 90 soniyaga sig'adi, ishga oid, kuchli tom + lavozimga bog'lanish bilan tugaydi.

## 2. Texnik qiyinchilik (STAR)

> **Situation:** "O'tgan yil fintech kompaniyada ishlaganimda, oyning oxirgi kunlari hisobot generatsiya qiluvchi servis production'da ko'p marta ishdan chiqardi (out of memory). Bu vaqtda mijozlar oylik hisobotini ola olmasdi va support'ga shikoyatlar yog'ilardi."
>
> **Task:** "Backend dasturchi sifatida menga bu servisni barqaror ishlaydigan qilish topshirildi, chunki bu mijozlarning ishonchiga bevosita ta'sir qilardi."
>
> **Action:** "Men avval xotira profilini oldim va servis butun bir oylik ma'lumotni (millionlab qator) bir vaqtda RAM'ga yuklayotganini topdim. Buni hal qilish uchun men hisobotni stream/batch usulida — har safar 10 ming qator bilan — qayta ishlashga o'zgartirdim. Keyin og'ir SQL aggregatsiyalarni database tomoniga ko'chirdim va natijani kesh qildim. O'zgarishni staging'da real hajmdagi ma'lumot bilan stress-test qilib, team lead bilan code review qildim, so'ng canary deploy qildim."
>
> **Result:** "Natijada xotira sarfi 4 GB dan 400 MB ga tushdi, servis oxirgi kunlarda ham crash bo'lmay ishladi va hisobot generatsiya vaqti 8 daqiqadan 90 soniyaga qisqardi. Men shu tajribadan keyin jamoaga 'katta ma'lumot bilan ishlaganda har doim stream qil' degan amaliyotni hujjatlashtirdim."

## 3. Konflikt (STAR)

> **Situation:** "Bir loyihada men va yana bir dasturchi yangi modul uchun arxitektura tanlashda kelisha olmadik. Men oddiy monolit ichida qoldirishni, u esa alohida microservice qilishni xohlardi. Bahsimiz biroz cho'zilib, sprint sekinlasha boshladi."
>
> **Task:** "Mening maqsadim — bizni to'g'ri texnik qarorga olib kelish va munosabatni buzmasdan kelishuvga erishish edi."
>
> **Action:** "Men uni qahva ustida alohida chaqirib, avval uning nuqtai nazarini diqqat bilan tingladim — u kelajakdagi scale'dan xavotirda edi. Keyin men o'z dalilimni ko'rsatdim: hozircha trafik kam, microservice qo'shimcha murakkablik (deploy, monitoring) keltiradi. Biz kelishib, kichik prototip qildik va modulni keyinchalik oson ajratish mumkin bo'ladigan tarzda monolit ichida toza chegara (interface) bilan yozdik."
>
> **Result:** "Bu yechim ikkalamizni ham qanoatlantirdi: hozir oddiy, kelajakda ajratish oson. Modul muddatida chiqdi. Bundan keyin biz katta qarorlarni bahsdan oldin qisqa 'design doc' yozib, jamoa bilan muhokama qiladigan bo'ldik — bu keyingi konfliktlarni kamaytirdi."

## 4. Muvaffaqiyatsizlik (STAR)

> **Situation:** "Junior bo'lganimda, juma kuni kech soatda kichik 'bir qatorlik' tuzatishni to'g'ridan-to'g'ri production'ga, testsiz deploy qildim, chunki 'bu juda oddiy' deb o'yladim."
>
> **Task:** "Maqsad oddiy bug'ni tezda tuzatish edi, lekin men jarayonni e'tiborsiz qoldirdim."
>
> **Action:** "O'zgarishim kutilmagan tarzda boshqa kod yo'lini buzdi va to'lov sahifasi ishlamay qoldi. Buni 20 daqiqadan keyin monitoring orqali sezdim. Men darrov o'zgarishni rollback qildim, team lead'ga o'zim aytib, muammoni tan oldim va keyin to'g'ri tuzatishni test bilan birga qayta yubordim."
>
> **Result:** "Sahifa qisqa vaqt ishlamadi, lekin men muammoni yashirmasdan tezda hal qildim. Eng muhimi — bu menga 'hech qachon testsiz va review'siz production'ga deploy qilmaslik'ni o'rgatdi. Men keyin jamoaga CI'da majburiy test bosqichini va production deploy uchun review talab qilishni taklif qildim, va bu o'rnatildi. Shundan keyin bunday xato takrorlanmadi."

## 5. Kuchsiz tomon

> **Situation/Task:** "Mening haqiqiy kuchsiz tomonim — ilgari vazifani boshqalarga topshirishni (delegation) qiyinlardim. Hamma ishni o'zim qilsam 'ishonchliroq' deb hisoblardim, natijada o'zim ortiqcha yuklanardim va jamoa o'sishi sekinlashardi."
>
> **Action:** "Buni anglaganimdan keyin, ataylab kichik, xavfi past vazifalarni jamoa a'zolariga bera boshladim. Ishonch hosil qilish uchun nazoratni code review orqali o'rnatdim — natijani tekshiraman, lekin jarayonga aralashmayman. Yangi a'zolarga aniq kontekst va kutilgan natijani yozib berishni o'rgandim."
>
> **Result:** "Natijada men muhimroq ishlarga vaqt topdim, jamoa a'zolari o'sdi va men endi delegation'ni kuchli tomonga aylantirdim. Hali ham bunga e'tibor beraman, lekin ancha yaxshilandim."

Izoh: haqiqiy zaiflik, soxta emas; va eng muhimi — uni yaxshilash uchun konkret harakat ko'rsatilgan.

## 6. Deadline (STAR)

> **Situation:** "Bir sprintda menga yangi to'lov provayderini integratsiya qilish topshirilgan edi va deadline 1 hafta edi. Ishni boshlaganimda provayderning hujjati to'liqsiz va sandbox'i beqaror ekanini ko'rdim — bu kutilganidan ancha murakkab edi."
>
> **Task:** "Mening vazifam integratsiyani o'z vaqtida yetkazish, agar imkonsiz bo'lsa, buni to'g'ri boshqarish edi."
>
> **Action:** "Deadline'ni kutib o'tirmasdan, 2-kuniyoq risk borligini manager'ga aytdim. Birgalikda scope'ni qayta ko'rib chiqdik: men avval asosiy to'lov oqimini (happy path) ishga tushirib, refund va edge case'larni keyingi sprintga qoldirishni taklif qildim. Provayder support'i bilan ham bevosita bog'lanib, sandbox muammosini tezlashtirishni so'radim."
>
> **Result:** "Asosiy integratsiya o'z vaqtida, ishlaydigan holatda chiqdi; qolgan 20% keyingi sprintga rejalashtirilgan tarzda ko'chdi. Hech kim kutilmaganda qolmadi, chunki men erta ogohlantirgandim. Bundan keyin men noma'lum tashqi tizimlar bilan ishlaganda estimatsiyaga buffer qo'shishni o'rgandim."

## 7. Ishontirish (STAR)

> **Situation:** "Jamoamiz yangi loyiha uchun ma'lumotlar bazasini tanlayotgan edik. Ko'pchilik 'hozir moda' bo'lgani uchun NoSQL'ni xohlardi, lekin men bizning ma'lumotimiz juda relyatsion (tranzaksiyalar, hisobotlar) ekanini ko'rardim."
>
> **Task:** "Men jamoani to'g'ri tanlovga — bu holatda PostgreSQL'ga — bosim o'tkazmasdan, dalil bilan ishontirishim kerak edi."
>
> **Action:** "Men hissiyot bilan bahslashish o'rniga kichik tadqiqot qildim: bizning real query namunalarimizni ikkala variantda prototip qilib, benchmark oldim. Natijalarni qisqa hujjatda — performance, ACID kafolat, jamoaning tajribasi bo'yicha solishtirib ko'rsatdim. Yig'ilishda men avval NoSQL tarafdorlarining xavotirini (kelajakdagi scale) tingladim va ularga PostgreSQL'da bu qanday hal qilinishini ko'rsatdim."
>
> **Result:** "Ma'lumotga asoslangan taqdimotdan keyin jamoa PostgreSQL'ni tanladi. Loyiha barqaror ishladi va keyinchalik hisobot talablari ko'paygach, relyatsion model bizni ko'p ishdan qutqardi. Men shundan o'rgandimki, eng yaxshi ishontirish — bosim emas, dalil va boshqalarning xavotirini tinglash."

## 8. Feedback (STAR)

> **Situation:** "Bir code review'da team lead'im mening yozgan funksiyam 'juda aqlli' (clever) — ya'ni qisqa, lekin tushunish qiyin ekanini aytdi va uni soddalashtirishni so'radi."
>
> **Task:** "Mening oldimda tanlov bor edi: mudofaaga o'tib o'zimni oqlash yoki feedback'ni jiddiy qabul qilish."
>
> **Action:** "Men avval biroz xafa bo'ldim — kodimdan faxrlanardim. Lekin bir kun o'tib, uning nuqtai nazaridan qaytib qaradim: agar men o'zim 6 oydan keyin bu kodni tushunmasam, demak u haq. Funksiyani bir nechta aniq nomli kichik qismga ajratdim, 'aqlli' bir qatorni o'qiluvchan bir nechta qatorga aylantirdim va uni qayta yubordim."
>
> **Result:** "Yangi versiya review'dan tez o'tdi va keyinroq boshqa dasturchi shu kodni oson o'zgartira oldi. Men shundan 'kod aqlli emas, oddiy bo'lishi kerak' degan prinsipni o'zimga oldim. Endi men ham feedback'ni shaxsiy hujum emas, o'sish imkoniyati sifatida qabul qilaman."

---

← [Behavioral bo'limiga qaytish](../../behavioral/README.md)
