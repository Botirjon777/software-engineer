# Cloud Asoslari — Yechimlar

Bu fayl "Cloud Asoslari" mavzusidagi masalalar uchun to'liq yechimlarni o'z ichiga oladi.

## 1. Startap uchun CapEx vs OpEx va minimal xarajat strategiyasi

**Nega cloud (OpEx)?** Cheklangan byudjetli startap uchun CapEx (oldindan server xarid qilish) o'limga teng — bir necha ming dollar apparatga ketadi, biznes hali daromad keltirmaydi va server eskirsa qoladi. Cloud OpEx modeli oldindan katta to'lovni talab qilmaydi: faqat har oy ishlatgan resursing uchun to'laysan. Bu pulni mahsulot va marketingga sarflash imkonini beradi, biznes o'sgan sari xarajat ham asta o'sadi.

**Dastlabki oylarda xarajatni minimal saqlash strategiyasi:**

- **Free tier'dan foydalan:** AWS/GCP/Azure'ning bepul qatlamlari (masalan kichik VM, S3'ning birinchi GB'lari) bilan boshla.
- **Right-sizing:** Eng kichik instance turini (masalan `t3.micro`) tanla, kerak bo'lsa keyin kattalashtirasan.
- **Managed/serverless tanla:** Kam trafikli startap uchun serverless (Lambda) "ishlamasa to'lamaysan" tamoyili bilan arzon bo'ladi.
- **Auto-scaling pastdan:** Minimal serverlardan boshlab, faqat trafik kelganda kengay.
- **Budget alert qo'y:** Byudjet ogohlantirishini (masalan oyiga $50) o'rnat — kutilmagan hisobdan himoyalanasan.
- **Test resurslarini o'chir:** Sinov uchun ko'tarilgan serverlarni ishdan keyin o'chir.

**Izoh:** Startap mantig'i — "kam boshlab, talabga qarab o'sish". Cloud aynan shuni qulay qiladi.

## 2. E-commerce arxitekturasi (AWS xizmatlari bilan)

| Komponent | Vazifa | AWS xizmati |
|-----------|--------|-------------|
| Foydalanuvchi rasmlari | Mahsulot/profil rasmlari saqlash | **S3** (object storage) |
| Database | Buyurtma, foydalanuvchi, mahsulot ma'lumotlari | **RDS** (PostgreSQL/MySQL) |
| Statik kontent (rasm, JS, CSS) | Foydalanuvchiga tez yetkazish | **CloudFront** (CDN, S3 ustida) |
| Trafik taqsimlash | So'rovlarni serverlar orasida bo'lish | **ELB / ALB** (Load Balancer) |
| Ilova serverlari | Backend kodi ishlaydi | **EC2** (yoki ECS/EKS konteynerlar) |
| Tarmoq izolyatsiyasi | Xavfsiz ichki tarmoq | **VPC** |

**Oqim:** Foydalanuvchi -> CloudFront (statik kontent va rasmlar S3'dan, edge'dan tez) -> dinamik so'rovlar Load Balancer'ga -> bir nechta EC2 ilova serveri -> RDS database. Rasmlar to'g'ridan-to'g'ri S3'da, CloudFront ularni kesh qiladi.

```text
Foydalanuvchi
   │
   ├──(statik: rasm/JS/CSS)──> CloudFront (CDN) ──> S3
   │
   └──(dinamik so'rov)──> Load Balancer
                              ├──> EC2 (ilova) ──┐
                              ├──> EC2 (ilova) ──┼──> RDS (database)
                              └──> EC2 (ilova) ──┘
```

**Izoh:** Rasmlarni database'da emas, S3'da saqlash arzon va miqyoslanadigan; CDN sahifa tezligini oshiradi; Load Balancer va bir nechta EC2 high availability beradi.

## 3. VM vs Serverless tanlovi

**(a) Doimiy yuqori trafik bilan ishlaydigan API — VM (yoki konteyner/ECS).**
Doimiy va katta yuk bo'lganda, har so'rovga to'lov qiladigan serverless qimmatga tushadi. Doimo ishlaydigan VM (yoki Reserved Instance) bunday holatda arzonroq va barqaror latency beradi (cold start yo'q). Auto-scaling guruhi yukка moslashadi.

**(b) Kuniga bir necha marta ishlaydigan hisobot generatori — Serverless (Lambda).**
Bu vazifa kamdan-kam, qisqa muddat ishlaydi. VM'ni doimo yoqib turish — bo'sh resurs uchun to'lov. Serverless faqat funksiya ishga tushganda (millisekundlar) to'laydi, qolgan vaqt 0 dollar. Cold start bu yerda muammo emas, chunki foydalanuvchi real-time javob kutmaydi.

**Izoh:** Umumiy qoida — doimiy/og'ir yuk = VM/konteyner; siyrak/hodisaga asoslangan yuk = serverless. Asosiy mezon — "resurs qancha vaqt bo'sh turadi?" va "har so'rov narxi VM narxidan arzonmi?".

## 4. Bitta AZ'dan multi-AZ high availability'ga o'tish

**Muammo:** Bitta AZ'da ishlash — agar shu ma'lumotlar markazi ishdan chiqsa, butun ilova o'chadi (single point of failure).

**Yechim:** Ilovani kamida ikki AZ'ga taqsimlash va ular oldiga Load Balancer qo'yish. Database'ni ham Multi-AZ rejimida (RDS Multi-AZ — bitta AZ'da primary, boshqasida standby replica) ishga tushirish. Auto-scaling guruhini bir nechta AZ'ga sozlash.

```text
Region: eu-central-1
                Load Balancer (multi-AZ)
                ┌──────────┴──────────┐
                ▼                     ▼
        ┌──────────────┐      ┌──────────────┐
        │     AZ-a     │      │     AZ-b     │
        │  EC2 (ilova) │      │  EC2 (ilova) │
        │  RDS primary │◄────►│  RDS standby │
        └──────────────┘      └──────────────┘

Bir AZ ishdan chiqsa, Load Balancer trafikni
sog'lom AZ'ga yo'naltiradi, RDS standby'ga failover qiladi.
```

**Izoh:** HA'ning asosi — har bir qatlamda (ilova va database) zaxira (redundancy) yaratish va ularni turli AZ'larga joylash. Load Balancer sog'lom node'larni avtomatik tanlaydi.

## 5. Auto-scaling siyosatini tasvirlash

**Siyosat (so'z bilan):**
- **Minimal:** har doim kamida 2 server ishlasin (high availability uchun — bittasi o'lsa ham bittasi qoladi).
- **Maksimal:** eng ko'pi 10 server (xarajat shifti / cheklov).
- **Scale out (kengaytirish):** o'rtacha CPU 70%'dan oshsa, yangi server qo'sh.
- **Scale in (kamaytirish):** o'rtacha CPU 30%'dan pasaysa, server o'chir (lekin minimal 2 dan pastga tushma).

Buzilib-buzilib qo'shib-o'chirmaslik uchun "cooldown" (masalan har scaling amalidan keyin 5 daqiqa kutish) qo'yiladi.

**Kuzatiladigan metrikalar:**
- O'rtacha CPU foizi (asosiy mezon — guruh bo'yicha o'rtacha).
- So'rovlar soni / sekund yoki Load Balancer'dagi navbat uzunligi (CPU yetarli signal bermasa).
- Javob vaqti (latency) — degradatsiyani sezish uchun.
- Sog'lom server soni (health check).

**Izoh:** CPU yagona to'g'ri metrika emas — I/O-bound ilovalarda so'rovlar soni yoki latency'ga qarab scaling to'g'riroq bo'lishi mumkin.

## 6. Kutilmagan 3 baravar yuqori hisob — sababini topish va oldini olish

**Sababini topish qadamlari:**
1. **Cost Explorer / billing dashboard'ni och** — xarajatni xizmat, region va teg bo'yicha ajratib ko'r. Qaysi xizmat o'sganini aniqla.
2. **Tekshir:** ishdan keyin o'chirilmagan katta server, ishlatilmagan storage/snapshot, "yetim" resurslar (egasiz Elastic IP, eski backup), kutilmagan data transfer (chiqish trafigi qimmat).
3. **Loglarni tekshir** — DDoS yoki noto'g'ri kod cheksiz so'rov/data transfer yaratayotgan bo'lishi mumkin.
4. **Yangi resurslarni ko'r** — kimdir test uchun katta instance yoki noto'g'ri konfiguratsiya qo'shganmi.

**Kelajakda oldini olish:**
- **Budget alert** o'rnat (masalan, byudjetning 50%, 80%, 100%'ida ogohlantirish).
- **Resurslarni teg bilan belgila** (loyiha/muhit/egasi) — xarajat manbasini tez topish uchun.
- **Right-sizing** muntazam tekshiruvi — keraksiz katta resurslarni kichraytir.
- **Auto-stop/scheduler** — dev/test resurslarini kechasi avtomatik o'chir.
- **IAM cheklovlari** — kim katta resurs ochishini cheklab qo'y.

**Izoh:** Eng ko'p "kutilmagan hisob" manbasi — unutilgan/ishlatilmagan resurslar va kuzatuvsizlik. Tagging + budget alert kombinatsiyasi shuni hal qiladi.

## 7. IaaS/PaaS/SaaS — mahsulot misollari va javobgarlik jadvali

**3 tadan mahsulot misoli:**
- **IaaS:** AWS EC2, GCP Compute Engine, Azure Virtual Machines.
- **PaaS:** Heroku, AWS Elastic Beanstalk, Google App Engine.
- **SaaS:** Gmail, Slack, Salesforce.

**Qaysi qatlamni kim boshqaradi:**

| Qatlam | IaaS | PaaS | SaaS |
|--------|------|------|------|
| Application (ilova) | Sen | Sen | Provayder |
| Data (ma'lumot) | Sen | Sen | Provayder |
| Runtime | Sen | Provayder | Provayder |
| OS | Sen | Provayder | Provayder |
| Virtualizatsiya | Provayder | Provayder | Provayder |
| Server / Tarmoq | Provayder | Provayder | Provayder |

**Izoh:** IaaS'dan SaaS'ga qarab sening mas'uliyating (va nazorating) kamayadi, provayderniki ortadi. Tanlov "qancha nazorat kerak vs qancha tez deploy kerak" muvozanatiga bog'liq.

## 8. Terraform bilan EC2 + S3 yaratishda IaC afzalliklari

Terraform misoli:

```hcl
resource "aws_instance" "web" {
  ami           = "ami-0abcdef1234567890"
  instance_type = "t3.micro"
  tags = { Name = "web-server" }
}

resource "aws_s3_bucket" "assets" {
  bucket = "my-app-assets-bucket"
}
```

**Namoyon bo'ladigan IaC afzalliklari:**
- **Takrorlanuvchanlik:** `terraform apply` ni dev, staging, production'da ishlatib, aynan bir xil EC2 + S3 ni yaratasan — qo'lda har safar bir xil bosish mumkin emas.
- **Versiyalash:** `.tf` fayl Git'da saqlanadi; kim, qachon, nimani o'zgartirganini ko'rasan; kerak bo'lsa orqaga qaytarasan.
- **Hujjatlashtirish:** Kodning o'zi infratuzilma tavsifi — qanday resurslar borligi bir qarashda ko'rinadi.
- **Xavfsiz o'zgarish:** `terraform plan` o'zgarishni amalga oshirishdan oldin ko'rsatadi.
- **Bog'liqlikni boshqarish:** EC2 va S3 o'rtasidagi aloqani (masalan IAM ruxsati) kodda aniq bog'laysan.

**Qo'lda yaratish bilan taqqos:**

| Jihat | Qo'lda (konsol) | Terraform (IaC) |
|-------|-----------------|------------------|
| Takrorlash | Har safar qo'lda, xatoga moyil | Bir buyruq, aynan bir xil |
| Tarix | Yo'q | Git'da to'liq |
| Drift (chetlanish) | Sezilmaydi | `plan` aniqlaydi |
| Tozalash | Qo'lda, unutish oson | `terraform destroy` |

**Izoh:** Qo'lda yaratish 1-2 resurs uchun tez ko'rinadi, lekin o'nlab resurs va bir nechta muhit bo'lganda IaC vaqt va xatolarni keskin kamaytiradi.

## 9. Maxfiy ma'lumot o'z serverida, web global — qaysi deployment modeli?

**Tavsiya: Hybrid cloud.**

**Sabab:** Talab ikki qarama-qarshi ehtiyojni o'z ichiga oladi:
- Maxfiy moliyaviy ma'lumot **o'z serverida** (private cloud / on-premise) qolishi shart — qonun/regulyatsiya yoki xavfsizlik talabi.
- Web-ilova **global va tez** bo'lishi kerak — buni public cloud (global region'lar, CDN, auto-scaling) eng yaxshi beradi.

Hybrid model ikkalasini birlashtiradi: maxfiy ma'lumotlar bazasi private'da, web qatlami va statik kontent public cloud'da (CDN bilan) ishlaydi; ikkalasi xavfsiz tarmoq (VPN / private link) orqali bog'lanadi.

```text
   Global foydalanuvchilar
            │
   Public Cloud (web + CDN, global, auto-scale)
            │  (xavfsiz VPN / private link)
            ▼
   Private Cloud / On-prem
   (maxfiy moliyaviy ma'lumot — o'z serverida)
```

**Izoh:** Public cloud — global tezlik; private cloud — maxfiylik nazorati. Hybrid ikkala talabni ham qondiradi. Faqat public yoki faqat private bularning birini buzar edi.

## 10. Serverless cold start muammosini hal qilish variantlari

**Cold start nima:** Funksiya uzoq ishlatilmasa, keyingi so'rovda muhit (konteyner) noldan tayyorlanadi va birinchi javob sekin bo'ladi (yuz ms — bir necha soniya).

**Kamaytirish variantlari:**
1. **Provisioned concurrency / "warm" tutish:** Provayder ma'lum sondagi nusxani doimo issiq holatda saqlaydi (AWS Lambda Provisioned Concurrency). Cold start deyarli yo'qoladi, lekin qo'shimcha to'lov bor.
2. **Periodik "ping" (keep-warm):** Funksiyani har bir necha daqiqada chaqirib, sovib qolishiga yo'l qo'ymaslik (oddiy, lekin to'liq hal qilmaydi).
3. **Runtime/paketni yengillashtirish:** Kichikroq deploy paketi, kamroq dependency, tezroq ishga tushadigan til (masalan og'ir JVM o'rniga Node.js/Go) tanlash.
4. **Init kodini kamaytirish:** Og'ir ulanishlarni (DB pool) global doirada bir marta yarat, har chaqiruvda emas.

**Boshqa yondashuvga o'tish:**
5. **Doimiy konteyner/VM:** Agar trafik doimiy va latency kritik bo'lsa, serverless o'rniga doimo ishlaydigan konteyner (ECS/Fargate, Kubernetes) yoki kichik VM ishlatish — cold start umuman bo'lmaydi.

**Izoh:** Tanlov trafik xarakteriga bog'liq. Latency-sezgir va trafik barqaror bo'lsa, doimiy compute (yoki provisioned concurrency) to'g'ri; trafik siyrak bo'lsa, keep-warm yoki shunchaki cold start'ga chidash arzonroq bo'lishi mumkin.

---

← [DevOps bo'limiga qaytish](../../devops/README.md)
