# System Design — Lug'at

Katta miqyosli tizimlarni loyihalash (System Design) bo'yicha eng ko'p uchraydigan texnik terminlar lug'ati: masshtablash, keshlash, ma'lumotlar bazasini bo'lish, ishonchlilik va samaradorlik tushunchalari. Terminlar ingliz tilida qoldirilgan, izohlar o'zbek tilida sodda tushuntirish tarzida berilgan.

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

← [System Design bo'limiga qaytish](./README.md)
