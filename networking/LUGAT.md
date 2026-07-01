# Networking — Lug'at

Bu lug'at kompyuter tarmoqlari (networking) sohasidagi asosiy ingliz terminlarni o'zbek tilida sodda tushuntiradi. Termin ingliz tilida qoladi, izoh esa "bu nima degani" tarzida beriladi. Terminlar alfavit tartibida joylashtirilgan.

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

← [Networking bo'limiga qaytish](./README.md)
