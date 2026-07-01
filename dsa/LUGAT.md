# DSA — Lug'at

Data Structures va Algoritmlar bo'yicha eng ko'p uchraydigan texnik terminlar lug'ati. Terminlar ingliz tilida qoldirilgan (chunki intervyularda va dokumentatsiyada shunday ishlatiladi), izohlar esa o'zbek tilida — "bu nima degani" tushuntirish tarzida berilgan.

| Termin | Ma'nosi (o'zbekcha) |
|--------|---------------------|
| Adjacency list | Graf saqlashning usuli: har bir vertex uchun unga bog'langan qo'shnilar ro'yxati saqlanadi. Kam bog'lamli (sparse) graflar uchun tejamli. |
| Adjacency matrix | Graf saqlashning usuli: N×N o'lchamli jadval, katakcha ikki vertex o'rtasida bog'lam bor-yo'qligini bildiradi. Tez tekshiradi, lekin ko'p joy egallaydi. |
| Algorithm | Muammoni yechish uchun aniq, qadamma-qadam ko'rsatmalar to'plami. Masalan, ro'yxatni saralash tartibi. |
| Amortized | O'rtacha xarajat: ba'zi amallar qimmat bo'lsa-da, ko'p amallar bo'yicha o'rtacha olganda arzon chiqishi. Masalan, dynamic array'ga qo'shish. |
| Array | Bir xil turdagi elementlarni ketma-ket, indeks bo'yicha saqlaydigan tuzilma. Elementga indeks orqali darhol (O(1)) kirish mumkin. |
| Asymptotic | Kirish hajmi (n) juda kattalashganda algoritm qanday o'sishini tavsiflash usuli. Kichik detallar tashlab, asosiy o'sish tendensiyasiga qaraladi. |
| AVL tree | O'zini avtomatik muvozanatlab turadigan binary search tree. Balandligi doim log(n) atrofida bo'lib, qidiruvni tez saqlaydi. |
| Backtracking | Yechim variantlarini birma-bir sinab ko'rish; agar yo'l boshi berk chiqsa, orqaga qaytib boshqa variantni sinash. Masalan, labirintdan chiqish. |
| Base case | Rekursiyani to'xtatadigan eng oddiy holat. U bo'lmasa rekursiya cheksiz davom etib, xatoga olib keladi. |
| BFS (Breadth-First Search) | Grafni "qavatma-qavat" aylanish: avval eng yaqin qo'shnilar, keyin ularning qo'shnilari. Eng qisqa yo'lni topishda ishlatiladi. |
| Big-O | Algoritm qancha vaqt yoki xotira talab qilishini eng yomon holatda ko'rsatadigan belgi. Masalan, O(n) — kirish hajmiga to'g'ri proporsional. |
| Binary search | Saralangan ro'yxatda elementni topish usuli: har safar ro'yxatning yarmini tashlab yuborish. Juda tez — O(log n). |
| Binary tree | Har bir tugun ko'pi bilan ikkita bolaga (chap va o'ng) ega bo'lgan daraxt tuzilmasi. |
| BST (Binary Search Tree) | Binary tree turi: chapdagi qiymatlar kichik, o'ngdagilari katta bo'lib joylashadi. Bu qidiruv, qo'shish, o'chirishni tezlashtiradi. |
| Bucket | Hash table'da bir xil hash qiymatiga tushgan elementlarni saqlash uchun ajratilgan "chelak" (yacheyka). |
| Chaining | Hash table'da collision'ni hal qilish usuli: bir katakka tushgan elementlarni ro'yxat (zanjir) ko'rinishida ulash. |
| Circular buffer | O'lchami qat'iy bo'lib, oxiriga yetgach yana boshiga qaytadigan navbat. Doimiy oqim ma'lumotlari uchun qulay. |
| Collision | Ikki xil kalit hash funksiyasidan bir xil natija (bir xil katak) olganda yuzaga keladigan to'qnashuv. |
| Complete tree | Barcha darajalari to'liq to'ldirilgan (oxirgi darajadan tashqari, u chapdan boshlab to'ladi) daraxt. Heap shu tuzilishga ega. |
| Connected component | Grafda bir-biriga yo'l orqali bog'langan vertexlar guruhi. Bir komponent ichidan tashqariga chiqib bo'lmaydi. |
| Cycle | Grafda boshlang'ich vertexga qaytib keladigan yopiq yo'l. Cycle borligi ba'zi algoritmlarda muhim. |
| Data structure | Ma'lumotlarni tartibli saqlash va ular ustida samarali ishlash uchun tuzilma. Masalan, array, stack, tree. |
| Deque | Ikki tomonli navbat (double-ended queue): elementni ikkala uchidan ham qo'shish va olib tashlash mumkin. |
| DFS (Depth-First Search) | Grafni "chuqurlikka" aylanish: bir yo'ldan oxirigacha borib, keyin orqaga qaytish. Rekursiya yoki stack bilan bajariladi. |
| Dijkstra | Grafda bir vertexdan boshqalarga eng qisqa (og'irlikli) yo'lni topadigan algoritm. Manfiy og'irliklar bilan ishlamaydi. |
| Directed graph | Bog'lamlar bir tomonlama bo'lgan graf (yo'nalishli). Masalan, "A dan B ga" yo'l, lekin teskarisi bo'lmasligi mumkin. |
| Divide and conquer | Muammoni kichik qismlarga bo'lib, har birini alohida yechib, keyin natijalarni birlashtirish usuli. Masalan, merge sort. |
| Doubly linked list | Har bir tugun ham keyingi, ham oldingi tugunga ko'rsatkichga ega bo'lgan bog'langan ro'yxat. Ikki tomonga yurish mumkin. |
| Dynamic array | O'zi avtomatik kattalashadigan array (masalan JS'dagi Array). To'lgach, kattaroq joy ajratib, elementlar ko'chiriladi. |
| Dynamic programming | Muammoni takrorlanuvchi kichik masalalarga bo'lib, ularning javobini eslab qolib, qayta hisoblamaslik usuli. |
| Edge | Grafda ikki vertexni bog'lovchi bog'lam (chiziq). Og'irlikli yoki oddiy bo'lishi mumkin. |
| Eviction | To'lgan tuzilmadan (masalan cache yoki heap) eski/keraksiz elementni chiqarib tashlash. |
| Fast & slow pointers | Ro'yxat bo'ylab ikki ko'rsatkichni turli tezlikda yuritish usuli. Sikl bor-yo'qligini yoki o'rtani topishda ishlatiladi. |
| Fibonacci | Har bir son o'zidan oldingi ikkitasining yig'indisiga teng ketma-ketlik (0,1,1,2,3,5...). Rekursiya va DP misoli sifatida mashhur. |
| FIFO | "First In, First Out" — birinchi kirgan birinchi chiqadi. Navbat (queue) shu tamoyilda ishlaydi. |
| Graph | Vertexlar (nuqtalar) va ularni bog'lovchi edge'lardan iborat tuzilma. Tarmoq, xarita kabi bog'lanishlarni modellashtiradi. |
| Greedy | Har qadamda o'sha ondagi eng yaxshi variantni tanlash strategiyasi, umumiy natija haqida o'ylamay. Ba'zan optimal, ba'zan yo'q. |
| Hash function | Kalitni (masalan matnni) qat'iy o'lchamli son (indeks)ga aylantiruvchi funksiya. Hash table asosini tashkil qiladi. |
| Hash table | Kalit-qiymat juftliklarini saqlab, kalit orqali deyarli darhol (O(1)) topib beradigan tuzilma. JS'da Map/Object. |
| Heap | To'liq binary tree ko'rinishidagi tuzilma bo'lib, ildizda doim eng katta (yoki eng kichik) element turadi. Priority queue asosi. |
| Heapify | Massivni heap qoidalariga muvofiq qayta tartiblash jarayoni. |
| In-place | Qo'shimcha katta xotira ishlatmay, mavjud tuzilmaning o'zida o'zgartirish kiritish. Xotira tejaydi. |
| Inorder | Binary tree'ni aylanish tartibi: chap → tugun → o'ng. BST'da bu saralangan tartibda qiymat beradi. |
| Iteration | Tsikl (loop) yordamida amalni takror-takror bajarish. Rekursiyaga muqobil yondashuv. |
| Leaf node | Daraxtda bolasi yo'q tugun (eng chekka, oxirgi tugun). |
| LIFO | "Last In, First Out" — oxirgi kirgan birinchi chiqadi. Stack shu tamoyilda ishlaydi. |
| Linked list | Har bir element (tugun) qiymat va keyingi tugunga ko'rsatkichdan iborat ketma-ket bog'langan ro'yxat. |
| Load factor | Hash table'ning qancha to'lganini ko'rsatuvchi nisbat (elementlar soni / kataklar soni). Yuqori bo'lsa, collision ko'payadi. |
| Memoization | Rekursiv hisoblash natijalarini eslab qolib (kesh), bir xil hisobni qayta bajarmaslik. "Yuqoridan pastga" DP. |
| Merge sort | Massivni yarmiga bo'lib, har birini saralab, keyin birlashtirib saralaydigan barqaror algoritm. Har doim O(n log n). |
| Min-heap / Max-heap | Heap turi: min-heap'da ildiz eng kichik, max-heap'da eng katta element bo'ladi. |
| Monotonic stack | Elementlari doim o'suvchi yoki kamayuvchi tartibda turadigan stack. "Keyingi kattaroq element" kabi masalalarda ishlatiladi. |
| Node | Tuzilmaning bitta "yacheykasi" — qiymat va boshqa tugunlarga ko'rsatkichlarni saqlaydi (list, tree, graph'da). |
| NP-hard | Ma'lum bo'lgan tez (polinomial) yechimi yo'q, juda murakkab masalalar sinfi. Faqat taxminiy yechimlar amaliy bo'ladi. |
| Postorder | Binary tree'ni aylanish tartibi: chap → o'ng → tugun. Daraxtni o'chirishda qulay. |
| Preorder | Binary tree'ni aylanish tartibi: tugun → chap → o'ng. Daraxt nusxasini olishda ishlatiladi. |
| Prefix sum | Har bir indeksgacha bo'lgan elementlar yig'indisini oldindan hisoblab qo'yish. Oraliq yig'indini tez topadi. |
| Priority queue | Elementlar kelish tartibida emas, muhimlik (prioritet) bo'yicha chiqadigan navbat. Odatda heap asosida quriladi. |
| Queue | FIFO tamoyilida ishlaydigan navbat: elementlar orqadan qo'shilib, oldindan olinadi. |
| Quick sort | Bitta "tayanch" (pivot) element atrofida massivni bo'lib saralaydigan tez algoritm. O'rtacha O(n log n). |
| Recursion | Funksiyaning o'zini-o'zi chaqirishi orqali muammoni yechish. Har chaqiruvda masala kichrayadi, base case'da to'xtaydi. |
| Red-black tree | O'zini muvozanatlab turadigan BST turi. Har tugunga "rang" belgilanadi va qoidalar orqali balandlik nazorat qilinadi. |
| Rotation | Daraxtni muvozanatlash uchun tugunlarni qayta joylashtirish amali (chapga/o'ngga aylantirish). |
| Set | Takrorlanmaydigan elementlar to'plami. Element bor-yo'qligini tez tekshiradi. JS'da Set. |
| Sibling | Daraxtda bitta ota-ona tugunga tegishli tugunlar (aka-uka tugunlar). |
| Singly linked list | Har bir tugun faqat keyingi tugunga ko'rsatuvchi bir tomonlama bog'langan ro'yxat. |
| Sliding window | Massiv yoki matn bo'ylab qat'iy yoki o'zgaruvchan "oyna"ni siljitib, oraliqlarni tekshirish usuli. Takroriy hisobni kamaytiradi. |
| Sorting | Elementlarni ma'lum tartibga (o'sish yoki kamayish) keltirish jarayoni. |
| Space complexity | Algoritm bajarilishi uchun qancha qo'shimcha xotira kerakligini kirish hajmiga bog'liq ravishda ifodalash. |
| Stable sort | Teng qiymatli elementlarning asl tartibini saqlab qoladigan saralash. Masalan, merge sort barqaror. |
| Stack | LIFO tamoyilida ishlaydigan tuzilma: element ustidan qo'shilib, ustidan olinadi. Funksiya chaqiruvlari shu tarzda saqlanadi. |
| Stack overflow | Rekursiya juda chuqurlashib, funksiya chaqiruvlari uchun ajratilgan xotira to'lib qolganda yuzaga keladigan xato. |
| String | Belgilar (harflar) ketma-ketligidan iborat matn ma'lumot turi. |
| Subtree | Daraxtning bir tugunidan boshlanuvchi, uning barcha avlodlarini o'z ichiga olgan qismi. |
| Tabulation | DP'ning "pastdan yuqoriga" usuli: kichik masalalardan boshlab jadvalni to'ldirib, katta javobga yetish. |
| Time complexity | Algoritm bajarilishi uchun qancha vaqt (amal) kerakligini kirish hajmiga bog'liq ravishda ifodalash. |
| Topological sort | Yo'nalishli graf (DAG)dagi vertexlarni bog'liqlik tartibida ketma-ket joylashtirish. Masalan, vazifalar navbati. |
| Traversal | Tuzilmaning (tree yoki graph) barcha elementlaridan ma'lum tartibda o'tib chiqish jarayoni. |
| Tree | Ildizdan boshlanib, tarmoqlanib ketuvchi ierarxik tuzilma. Sikllarsiz bog'langan graf turi. |
| Trie | Matnlarni belgi-belgi bo'yicha saqlaydigan maxsus daraxt. Prefiks (boshlanma) bo'yicha tez qidiruv beradi. Autocomplete misoli. |
| Two pointers | Massiv yoki matn ustida ikkita ko'rsatkichni harakatlantirib ishlash usuli. Ko'p masalalarni O(n)'ga tushiradi. |
| Undirected graph | Bog'lamlari ikki tomonlama bo'lgan graf: "A–B" bog'lami "B–A" ni ham anglatadi. |
| Union-Find | Elementlar qaysi guruhga tegishliligini tez aniqlaydigan tuzilma (Disjoint Set). Graf komponentlari va sikl aniqlashda ishlatiladi. |
| Vertex | Grafning bitta nuqtasi (tugun). Odam, shahar yoki obyektni bildirishi mumkin. |
| Weighted graph | Har bir edge'ga son (og'irlik/narx) biriktirilgan graf. Masalan, shaharlar orasidagi masofa. |

← [DSA bo'limiga qaytish](./README.md)
