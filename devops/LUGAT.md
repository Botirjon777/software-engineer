# DevOps — Lug'at

Bu lug'at DevOps bo'limida uchraydigan asosiy inglizcha texnik terminlarni sodda o'zbek tilida tushuntiradi. Termin ingliz tilida qoladi, izoh esa "bu nima degani"ni ochib beradi. Terminlar alfavit tartibida joylashtirilgan.

| Termin | Ma'nosi (o'zbekcha) |
|--------|---------------------|
| Artifact | Build jarayonidan chiqadigan tayyor natija (masalan, `.jar` fayl yoki Docker image). Uni saqlab, keyin deploy qilishda ishlatasan. |
| Auto-scaling | Yuk oshganda serverlar sonini avtomatik ko'paytirib, kamayganda kamaytirib turadigan mexanizm. Faqat kerak bo'lgancha resurs ishlatasan. |
| Bash | Linux'da eng ko'p ishlatiladigan buyruqlar tarjimoni (shell). Buyruq yozasan, u bajarib beradi. |
| Blue-green | Deploymentning bir usuli: ikkita bir xil muhit ("ko'k" va "yashil") bo'ladi, yangi versiyani birida sinab, tayyor bo'lgach trafikni o'sha tomonga o'tkazasan. |
| Branch | Git'da kodning alohida "shoxi" — asosiy koddan nusxa olib, unda mustaqil ishlaysan, boshqalarga xalaqit bermaysan. |
| Canary | Yangi versiyani avval foydalanuvchilarning kichik qismiga chiqarib sinash usuli. Muammo chiqsa, hammaga tarqalmaydi. |
| CD (Continuous Delivery/Deployment) | Testdan o'tgan kodni avtomatik tarzda tayyor holatga yoki to'g'ridan-to'g'ri serverga chiqarish jarayoni. |
| Cherry-pick | Boshqa branchdagi bitta aniq commitni olib, o'z branchingga ko'chirish. Butun branchni emas, faqat kerakli o'zgarishni olasan. |
| CI (Continuous Integration) | Har safar kod qo'shilganda uni avtomatik yig'ish va test qilish. Xatolarni erta topishga yordam beradi. |
| CI/CD | CI va CD birgalikda — koddan serverga qadar bo'lgan avtomatlashtirilgan yo'l (test, build, deploy). |
| CLI (Command Line Interface) | Sichqoncha o'rniga buyruqlarni matn ko'rinishida yozib ishlaydigan interfeys (terminal). |
| Cloud | Boshqa kompaniyaning serverlarini internet orqali ijaraga olib ishlatish (masalan, AWS, Google Cloud). O'z serveringni sotib olishing shart emas. |
| Commit | Git'da o'zgarishlarni saqlab, ularga "suratga olish" kabi qayd yaratish. Har bir commit tarixda qoladi. |
| ConfigMap | Kubernetes'da sozlamalar (config)ni koddan ajratib saqlaydigan obyekt. Kodni o'zgartirmasdan sozlamani almashtirasan. |
| Container | Ilovani va u ishlashi uchun kerak bo'lgan hamma narsani bitta yengil "quti"ga joylash. Har qanday joyda bir xil ishlaydi. |
| Cron | Belgilangan vaqtda avtomatik ishga tushadigan vazifalarni sozlash tizimi (masalan, har kuni soat 3:00'da backup olish). |
| chmod | Linux'da fayl yoki papkaga ruxsatlarni (o'qish/yozish/bajarish) o'zgartiradigan buyruq. |
| chown | Linux'da fayl yoki papkaning egasini (owner) o'zgartiradigan buyruq. |
| Deployment | Yozilgan kodni serverga chiqarib, foydalanuvchilar ishlata oladigan holatga keltirish. Kubernetes'da esa podlarni boshqaradigan obyekt. |
| Docker | Ilovalarni containerlarga joylashtirish va ishga tushirishning eng mashhur vositasi. |
| Docker Compose | Bir nechta containerni (masalan, ilova + baza) bitta fayl orqali birga sozlab, birgalikda ishga tushiradigan vosita. |
| Dockerfile | Docker image qanday yig'ilishini bosqichma-bosqich yozib qo'yadigan matnli fayl (retsept kabi). |
| Environment variable | Ilovaga tashqaridan beriladigan sozlama qiymati (masalan, parol yoki server manzili). Kodga yozib qo'yilmaydi. |
| Git | Kod tarixini saqlaydigan va bir nechta odam birga ishlashiga imkon beradigan version control tizimi. |
| HEAD | Git'da hozir qaysi commitda turganingni ko'rsatadigan "kursor". Odatda joriy branchning oxirgi commitiga ishora qiladi. |
| HPA (Horizontal Pod Autoscaler) | Kubernetes'da yukka qarab podlar sonini avtomatik ko'paytirib-kamaytiradigan mexanizm. |
| IaaS (Infrastructure as a Service) | Cloudda faqat "yalang'och" server, tarmoq va disk kabi asosiy resurslarni ijaraga olish. Qolganini o'zing sozlaysan. |
| IaC (Infrastructure as Code) | Serverlar va infratuzilmani qo'lda emas, kod (fayl) orqali yaratish va boshqarish. Takrorlash va kuzatish oson. |
| Image | Container ishga tushadigan tayyor "qolip" — ichida ilova va uning muhiti bor. Imagedan containerlar yaratiladi. |
| Ingress | Kubernetes'da tashqi trafikni klaster ichidagi kerakli servisga yo'naltiradigan "kirish darvozasi". |
| kubelet | Har bir Kubernetes node'ida ishlab, podlarning to'g'ri ishlashini ta'minlaydigan agent (dastur). |
| Kubernetes | Ko'plab containerlarni avtomatik boshqaradigan, ularni ishga tushiradigan va nazorat qiladigan orkestratsiya tizimi. |
| Layer | Docker imageni tashkil etuvchi qatlamlar. Har bir buyruq yangi qatlam yaratadi; o'zgarmagan qatlamlar qayta ishlatiladi (tez bo'ladi). |
| Merge | Ikki branchdagi o'zgarishlarni birlashtirish. Masalan, tayyor ishni asosiy branchga qo'shish. |
| Merge conflict | Ikki branch bir joyni har xil o'zgartirganda Git qaysi birini olishni bilmay qolgani. Uni qo'lda hal qilasan. |
| Orchestration | Ko'plab containerlarni avtomatik boshqarish jarayoni — ishga tushirish, tiklash, masshtablash (Kubernetes shu ishni qiladi). |
| Origin | Git'da masofadagi asosiy repository'ga berilgan standart nom (odatda GitHub'dagi nusxa). |
| PaaS (Platform as a Service) | Cloudda tayyor platforma olib, faqat kodni yuklaysan — server sozlash bilan shug'ullanmaysan (masalan, Heroku). |
| Permission | Fayl yoki papkaga kim nima qila olishini (o'qish/yozish/bajarish) belgilaydigan ruxsat. |
| Pipe | Bir buyruqning natijasini boshqasiga uzatuvchi belgi (`\|`). Buyruqlarni zanjir qilib bog'laysan. |
| Pipeline | CI/CD'da kod bosib o'tadigan bosqichlar ketma-ketligi (build → test → deploy). Avtomatik ishlaydi. |
| Pod | Kubernetes'da eng kichik ishga tushirish birligi — bir yoki bir nechta containerni o'z ichiga oladi. |
| Process | Hozir ishlab turgan dastur nusxasi. Har bir ishlab turgan dastur bir yoki bir nechta processdan iborat. |
| Pull request | Kodingdagi o'zgarishni asosiy branchga qo'shish uchun so'rov. Boshqalar ko'rib, tasdiqlaydi (code review). |
| Rebase | Branchdagi commitlarni boshqa branch ustiga "ko'chirib" tarixni tekislash. Merge'ga o'xshaydi, lekin tarix chiziqli chiqadi. |
| Redirection | Buyruq natijasini ekran o'rniga faylga yo'naltirish (`>`) yoki fayldan olib kelish (`<`). |
| Registry | Docker imagelar saqlanadigan va tarqatiladigan ombor (masalan, Docker Hub). |
| Remote | Git'da masofadagi (serverda turgan) repository nusxasi. Kodni u yerga yuborasan va undan olasan. |
| Repository | Loyihaning butun kodi va tarixi saqlanadigan Git ombori ("repo"). |
| Rollback | Deploy qilingan yangi versiyada muammo chiqsa, oldingi ishlaydigan versiyaga qaytish. |
| Rolling deployment | Yangi versiyani serverlarga bittalab, asta-sekin chiqarish. Xizmat to'xtamaydi, foydalanuvchi sezmaydi. |
| SaaS (Software as a Service) | Tayyor dasturni internet orqali ishlatish (masalan, Gmail). O'rnatish yoki server kerak emas. |
| Serverless | Serverni o'zing boshqarmaysan — faqat kod yozasan, u kerak bo'lganda avtomatik ishga tushadi va faqat ishlatilgani uchun to'laysan. |
| Service | Kubernetes'da podlar guruhiga barqaror manzil beradigan obyekt. Podlar almashsa ham manzil o'zgarmaydi. |
| Shell | Buyruqlaringni qabul qilib, operatsion tizimga bajartiradigan dastur (masalan, bash). |
| SSH (Secure Shell) | Masofadagi serverga xavfsiz (shifrlangan) tarzda ulanib, uni buyruqlar orqali boshqarish usuli. |
| Staging area | Git'da commitdan oldin o'zgarishlar yig'iladigan "tayyorlash zonasi". Nimani commit qilishni shu yerda tanlaysan. |
| Stash | Tayyor bo'lmagan o'zgarishlarni commit qilmasdan vaqtincha yashirib qo'yish. Keyin qaytarib olasan. |
| Terraform | IaC vositasi — serverlar va cloud resurslarini kod orqali yaratadi va boshqaradi. |
| VM (Virtual Machine) | Bitta fizik server ichida ishlaydigan "virtual kompyuter". Har birida alohida operatsion tizim bo'ladi. |
| Version control | Kod o'zgarishlarini tarixi bilan saqlab, kim nimani qachon o'zgartirganini kuzatuvchi tizim (masalan, Git). |
| Volume | Docker'da containerdan tashqarida saqlanadigan ma'lumot joyi. Container o'chsa ham ma'lumot yo'qolmaydi. |

← [DevOps bo'limiga qaytish](./README.md)
