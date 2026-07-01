# OOP, Patterns va Clean Code — Lug'at

Obyektga yo'naltirilgan dasturlash, dizayn shablonlari va toza kod bo'yicha inglizcha terminlar va ularning sodda o'zbekcha izohi.

| Termin | Ma'nosi (o'zbekcha) |
|--------|---------------------|
| Abstraction | Murakkab tafsilotlarni yashirib, faqat kerakli, sodda ko'rinishni tashqariga ochish tamoyili. |
| Adapter | Bir-biriga mos kelmaydigan ikki interfeysni ulaydigan shablon — xuddi elektr rozetka o'tkazgichi kabi. |
| Anti-pattern | Ko'pincha ishlatiladigan, lekin yomon oqibatga olib keladigan yechim — "shunday qilmang" namunasi. |
| Boy Scout Rule | "Kodni topganingdan ko'ra tozaroq qoldir" — har tegishda kichik yaxshilash kiritish tamoyili. |
| Builder | Murakkab obyektni bosqichma-bosqich, qadam-baqadam qurishga imkon beruvchi yaratuvchi shablon. |
| Chain of Responsibility | So'rovni bir nechta ishlovchi orasidan zanjir bo'ylab o'tkazish — kim uddalasa, o'sha bajaradi. |
| Class | Obyektlarni yasash uchun "chizma" — qanday maydon va metodlar bo'lishini belgilaydi. |
| Code Smell | Kodda "nimadir noto'g'ri" degan ishora beruvchi belgilar (masalan juda uzun funksiya) — xato emas, lekin ogohlantirish. |
| Coupling | Modullarning bir-biriga qanchalik bog'liqligi — kam (loose) bog'liqlik yaxshi, ko'p (tight) bog'liqlik yomon. |
| Cohesion | Bir modul ichidagi qismlar bir maqsadga qanchalik jipslashganini bildiradi — yuqori cohesion yaxshi. |
| Command | So'rov yoki amalni obyektga o'rab, uni saqlash, navbatga qo'yish yoki bekor qilishga imkon beruvchi shablon. |
| Composition | Obyektni meros olmasdan, boshqa obyektlarni ichiga jamlab qurish — "meros o'rniga birlashma" tamoyili. |
| Decorator | Obyektni o'zgartirmay, unga yangi xatti-harakat qo'shadigan shablon — "o'rab" imkoniyat qo'shadi. |
| Dependency Injection | Modulga kerakli bog'liqliklarni ichida yaratmay, tashqaridan berish — moslashuvchanlik va testni osonlashtiradi. |
| Design Pattern | Tez-tez uchraydigan muammoga tayyor, sinovdan o'tgan umumiy yechim namunasi. |
| DIP (Dependency Inversion) | SOLID'ning "D"si: yuqori darajali modul quyi modulga emas, ikkalasi ham abstraksiyaga bog'liq bo'lishi kerak. |
| DRY (Don't Repeat Yourself) | Bir xil kodni takrorlamang — mantiqni bir joyda saqlab, kerakli joylardan foydalaning. |
| Encapsulation | Obyekt ichki ma'lumotini yashirib, unga faqat belgilangan metodlar orqali kirishga ruxsat berish. |
| Facade | Murakkab tizim ustiga oddiy, yagona interfeys qo'yib, foydalanishni soddalashtiradigan shablon. |
| Factory | Obyektni to'g'ridan-to'g'ri new bilan yaratmay, uni yasab beradigan maxsus metod yoki klass orqali olish shabloni. |
| Inheritance | Bir klass boshqa klassdan maydon va metodlarni meros olib, ustiga o'zini qo'shishi. |
| Interface | Klass qanday metodlarni ta'minlashi kerakligini belgilaydigan shartnoma — amalga oshirishni o'zi bermaydi. |
| ISP (Interface Segregation) | SOLID'ning "I"si: katta, umumiy interfeys o'rniga kichik, maxsus interfeyslar yaxshiroq — keraksizga majburlamang. |
| Iterator | To'plam ichki tuzilishini ochmasdan, uning elementlarini birma-bir aylanib chiqishga imkon beruvchi shablon. |
| KISS (Keep It Simple, Stupid) | Yechimni imkon qadar sodda tuting — keraksiz murakkablikdan qoching. |
| Liskov Substitution | SOLID'ning "L"si: bola klass ota klass o'rnida ishlatilganda, dastur buzilmasligi kerak. |
| LSP (Liskov Substitution Principle) | Liskov Substitution'ning to'liq nomi — subklass superklassni to'liq va xatosiz almashtira olishi kerak. |
| Mediator | Obyektlar bir-biri bilan to'g'ridan-to'g'ri emas, oraliq "vositachi" orqali muloqot qilishini ta'minlaydigan shablon. |
| Memento | Obyektning holatini saqlab qo'yib, keyin o'sha holatga qaytarishga (undo) imkon beruvchi shablon. |
| Object | Klass asosida yaratilgan aniq nusxa — o'z ma'lumoti (holati) va xatti-harakatiga ega. |
| Observer | Bir obyekt o'zgarganda, unga "obuna bo'lgan" boshqa obyektlarga avtomatik xabar beradigan shablon. |
| OCP (Open/Closed Principle) | SOLID'ning "O"si: kod kengaytirishga ochiq, lekin o'zgartirishga yopiq bo'lishi kerak. |
| OOP (Object-Oriented Programming) | Dasturni bir-biri bilan aloqada bo'lgan obyektlar to'plami sifatida qurish uslubi. |
| Polymorphism | Bir xil metod turli obyektlarda turlicha ishlashi — "bitta interfeys, ko'p shakl". |
| Proxy | Asl obyekt oldida turadigan o'rinbosar — kirishni nazorat qiladi, keshlaydi yoki kechiktirib yuklaydi. |
| Pure Function | Bir xil kirishga doim bir xil natija qaytaradigan va tashqi holatga ta'sir qilmaydigan funksiya. |
| Refactoring | Kodning tashqi xatti-harakatini o'zgartirmay, ichki tuzilishini yaxshilash jarayoni. |
| Immutability | Yaratilgandan keyin o'zgartirilmaydigan obyekt — o'zgarish kerak bo'lsa, yangi nusxa yasaladi. |
| Singleton | Klassdan butun dasturda faqat bitta nusxa bo'lishini kafolatlaydigan shablon (masalan sozlamalar obyekti). |
| SOLID | Toza obyektli dizaynning beshta asosiy tamoyili: SRP, OCP, LSP, ISP, DIP. |
| SRP (Single Responsibility) | SOLID'ning "S"si: har bir klass yoki modulning faqat bitta o'zgarish sababi (bitta mas'uliyati) bo'lishi kerak. |
| State | Obyekt ichki holatiga qarab xatti-harakatini o'zgartiradigan shablon — holat almashsa, xulqi ham almashadi. |
| Strategy | Bir vazifani bajarishning bir nechta usulini almashtiriladigan alohida obyektlarga ajratib qo'yadigan shablon. |
| Technical Debt | Tez yechim uchun sifatni qurbon qilish natijasida to'planadigan "qarz" — keyin uni tuzatish qimmatga tushadi. |
| Template Method | Algoritmning umumiy skeletini otada belgilab, ayrim qadamlarini bola klasslarga to'ldirtiradigan shablon. |
| Tight Coupling | Modullar bir-biriga juda qattiq bog'langan holat — birini o'zgartirsang, boshqasi ham buziladi (yomon holat). |
| Loose Coupling | Modullar bir-biriga kam bog'langan holat — birini o'zgartirish boshqasiga ta'sir qilmaydi (yaxshi holat). |
| Visitor | Obyektlar strukturasini o'zgartirmasdan, ularga yangi amallar qo'shishga imkon beruvchi shablon. |
| YAGNI (You Aren't Gonna Need It) | "Kerak bo'lmaydigan narsani oldindan yozma" — faqat hozir zarur funksiyani qo'sh. |

← [OOP, Patterns va Clean Code bo'limiga qaytish](./README.md)
