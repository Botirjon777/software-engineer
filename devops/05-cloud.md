# Cloud Asoslari

Bu hujjat cloud computing (bulutli hisoblash) asoslarini o'zbek dasturchilari uchun tushuntiradi. Bugungi kunda deyarli har bir loyiha qaysidir cloud provayderda (AWS, GCP yoki Azure) ishlaydi. DevOps muhandisi sifatida sen cloud xizmatlari qanday ishlashini, qaysi xizmatni qachon tanlashni va xarajatlarni qanday boshqarishni bilishing shart.

**💡 Tushuncha:** Cloud computing — bu o'z serveringni xarid qilib, ofisingda saqlash o'rniga, internet orqali boshqa kompaniyaning serverlarini ijaraga olib ishlatish modeli. Sen faqat ishlatgan resursing uchun to'laysan.

## Mundarija

- [Cloud computing nima va nega kerak](#cloud-computing-nima-va-nega-kerak)
- [CapEx vs OpEx](#capex-vs-opex)
- [Deployment modellari](#deployment-modellari)
- [Xizmat modellari: IaaS vs PaaS vs SaaS](#xizmat-modellari-iaas-vs-paas-vs-saas)
- [Compute xizmatlari](#compute-xizmatlari)
- [Storage xizmatlari](#storage-xizmatlari)
- [Database xizmatlari](#database-xizmatlari)
- [Networking xizmatlari](#networking-xizmatlari)
- [Serverless va FaaS](#serverless-va-faas)
- [Regions va Availability Zones](#regions-va-availability-zones)
- [Auto-scaling](#auto-scaling)
- [Infrastructure as Code (IaC)](#infrastructure-as-code-iac)
- [Cloud xarajatlarini boshqarish](#cloud-xarajatlarini-boshqarish)
- [Asosiy provayderlar taqqosi](#asosiy-provayderlar-taqqosi)
- [Monitoring va Logging](#monitoring-va-logging)
- [Savol-javoblar](#savol-javoblar)
- [Masalalar](#masalalar)

## Cloud computing nima va nega kerak

Ilgari kompaniyalar dastur ishga tushirish uchun o'zlari fizik server xarid qilishi, uni ma'lumotlar markazida (data center) joylashtirishi, sovutish, elektr, xavfsizlik va texnik xizmatni o'zlari ta'minlashi kerak edi. Bu juda qimmat va sekin jarayon — yangi server kerak bo'lsa, uni xarid qilib o'rnatishga haftalar ketardi.

Cloud bu muammoni hal qiladi. Sen brauzer yoki buyruq orqali bir necha daqiqada virtual server yaratasan va ishlatasan. Kerak bo'lmasa — o'chirasan va to'lashni to'xtatasan.

Cloud'ning asosiy afzalliklari:

- **Elastiklik (elasticity):** Yuk oshganda resurslarni avtomatik ko'paytirish, kamayganda kamaytirish.
- **Managed services:** Database, queue, cache kabi murakkab tizimlarni provayder o'zi boshqaradi — sen faqat ishlatasan.
- **Global yetib borish:** Bir necha tugma bilan dunyoning boshqa qit'asida server ishga tushirish.
- **Pay-as-you-go:** Faqat ishlatgan resursing uchun to'lash.

**💡 Tushuncha:** "Managed service" — provayder serverning operatsion tizimi, yangilanishlar, backup va xavfsizligini o'zi boshqaradigan xizmat. Masalan, managed database'da sen DB versiyasini yangilash bilan shug'ullanmaysan.

## CapEx vs OpEx

Bu cloud'ga o'tishning asosiy moliyaviy sababi.

```text
CapEx (Capital Expenditure)        OpEx (Operational Expenditure)
─────────────────────────────      ─────────────────────────────
Katta dastlabki xarajat            Doimiy kichik to'lovlar
Server xarid qilish                Server ijaraga olish
Oldindan to'lash                   Ishlatgancha to'lash
Resurs band turadi                 Kerak bo'lganda ishlatish
Eskirish riski bor                 Provayder yangilaydi
```

An'anaviy model **CapEx** — sen oldindan katta pul to'lab server xarid qilasan. Cloud model **OpEx** — sen har oy ishlatgan miqdoringga qarab to'laysan. Startaplar uchun bu juda muhim: dastlab kam pul bilan boshlab, biznes o'sgan sari xarajatni oshirish mumkin.

## Deployment modellari

Cloud'ni qayerda va kim uchun joylashtirishga qarab uch modelga bo'linadi:

- **Public cloud:** AWS, GCP, Azure kabi provayderlar barcha mijozlarga ochiq taklif qiladigan infratuzilma. Eng ko'p ishlatiladi.
- **Private cloud:** Faqat bitta tashkilot uchun ajratilgan infratuzilma. Banklar va davlat tashkilotlari xavfsizlik talablari uchun ishlatadi.
- **Hybrid cloud:** Public va private cloud'ni birlashtirib ishlatish. Maxfiy ma'lumot private'da, qolgan yuk public'da.

```text
┌──────────────┐   ┌──────────────┐   ┌──────────────────────┐
│ Public Cloud │   │ Private Cloud│   │   Hybrid Cloud       │
│ (AWS/GCP)    │   │ (o'z DC)     │   │ Public + Private bog' │
│ Ochiq        │   │ Yopiq        │   │ Moslashuvchan        │
└──────────────┘   └──────────────┘   └──────────────────────┘
```

## Xizmat modellari: IaaS vs PaaS vs SaaS

Cloud xizmatlari javobgarlik chegarasiga qarab uch turga bo'linadi. Eng muhim farq — nimani sen boshqarasan, nimani provayder boshqaradi.

```text
                 IaaS          PaaS          SaaS
              ──────────    ──────────    ──────────
Application   SEN          SEN           Provayder
Data          SEN          SEN           Provayder
Runtime       SEN          Provayder     Provayder
OS            SEN          Provayder     Provayder
Virtualizatsiya Provayder  Provayder     Provayder
Server/Tarmoq Provayder    Provayder     Provayder
```

- **IaaS (Infrastructure as a Service):** Provayder virtual server, tarmoq, storage beradi. Sen OS, runtime, ilovani o'zing boshqarasan. Misol: AWS EC2, GCP Compute Engine. Eng ko'p nazorat, lekin eng ko'p mas'uliyat.
- **PaaS (Platform as a Service):** Provayder OS va runtime'ni ham boshqaradi. Sen faqat kodingni joylashtirasan. Misol: Heroku, AWS Elastic Beanstalk, Google App Engine. Tezroq deploy, kamroq nazorat.
- **SaaS (Software as a Service):** To'liq tayyor dastur. Sen faqat foydalanasan. Misol: Gmail, Slack, Salesforce.

**💡 Tushuncha:** Oson eslab qolish uchun pizza analogiyasi: IaaS — sen pizza pishirasan, lekin oshxona ijaraga olingan. PaaS — pizza tayyor, sen faqat qo'shimchalarni qo'shasan. SaaS — pizza yetkazib beriladi, sen faqat yeysan.

## Compute xizmatlari

Compute — bu kod ishlaydigan "hisoblash quvvati". Uch asosiy turi bor:

**Virtual Machine (VM / EC2):** To'liq virtual server. Sen OS tanlaysan, kerakli dasturlarni o'rnatasan. AWS'da EC2, GCP'da Compute Engine deyiladi.

```bash
# AWS CLI orqali EC2 instance ishga tushirish (misol)
aws ec2 run-instances \
  --image-id ami-0abcdef1234567890 \
  --instance-type t3.micro \
  --key-name my-key
```

**Container:** Ilovani izolyatsiya qilingan, yengil "konteyner"da ishlatish. Docker bilan paketlash, ECS/EKS bilan boshqarish. VM'dan yengilroq va tezroq.

**Serverless (Lambda):** Sen serverni umuman ko'rmaysan. Faqat funksiya yozasan, u kerak bo'lganda ishga tushadi va to'xtaydi. AWS'da Lambda, GCP'da Cloud Functions.

## Storage xizmatlari

Cloud'da uch xil saqlash turi mavjud:

- **Object storage (S3):** Fayllarni (rasm, video, backup) saqlash uchun. Cheksiz hajm, arzon, lekin "fayl tizimi" emas. AWS S3, GCP Cloud Storage. URL orqali murojaat qilinadi.
- **Block storage:** VM'ga ulanadigan disk. Operatsion tizim va database shu yerda ishlaydi. AWS EBS.
- **File storage:** Bir nechta server uchun umumiy fayl tizimi (NFS). AWS EFS.

```text
Object Storage (S3)    Block Storage (EBS)    File Storage (EFS)
──────────────────     ───────────────────    ──────────────────
Fayllar, backup        VM diski               Umumiy fayl tizimi
HTTP API orqali        Bitta VM'ga ulanadi    Ko'p VM ulashadi
Cheksiz, arzon         Tez, doimiy            Tarmoq orqali
```

## Database xizmatlari

Cloud'da database'ni o'zing o'rnatish o'rniga managed xizmatdan foydalanish odatiy:

- **Managed relational (RDS):** PostgreSQL, MySQL kabi SQL database'lar. Backup, replikatsiya, yangilanish avtomatik. AWS RDS, GCP Cloud SQL.
- **Managed NoSQL (DynamoDB):** Kalit-qiymat yoki hujjat database. Juda yuqori miqyoslanish. AWS DynamoDB, GCP Firestore.

**💡 Tushuncha:** Managed database'da provayder backup, patch va failover'ni boshqaradi. Sen o'zing EC2'ga PostgreSQL o'rnatsang, bularning hammasini o'zing qilishing kerak bo'ladi.

## Networking xizmatlari

- **VPC (Virtual Private Cloud):** Cloud ichidagi o'zingning izolyatsiya qilingan virtual tarmog'ing. Subnet'lar, IP diapazonlari shu yerda.
- **Load Balancer:** Kelayotgan trafikni bir nechta server o'rtasida taqsimlaydi. Bitta server o'lsa, trafik qolganlariga yo'naltiriladi.
- **CDN (Content Delivery Network):** Statik kontentni (rasm, JS, CSS) foydalanuvchiga eng yaqin serverdan yetkazadi. AWS CloudFront. Sahifa tezligini oshiradi.

```text
                  ┌─────────────────┐
   Foydalanuvchi →│  Load Balancer  │
                  └────────┬────────┘
              ┌────────────┼────────────┐
              ▼            ▼            ▼
          Server 1     Server 2     Server 3
```

## Serverless va FaaS

**FaaS (Function as a Service)** — serverless'ning asosiy shakli. Sen kichik funksiya yozasan, u biror hodisa (HTTP so'rov, fayl yuklanishi, timer) sodir bo'lganda ishga tushadi.

Afzalliklari:
- Server boshqarish kerak emas.
- Faqat ishlagan vaqt uchun to'lov (millisekundlar).
- Avtomatik miqyoslanadi (0 dan minglab nusxagacha).

Kamchiliklari:
- **Cold start:** Funksiya uzoq vaqt ishlamasa, birinchi so'rovda ishga tushishi sekin bo'ladi (bir necha yuz ms).
- Uzoq davom etadigan vazifalar uchun mos emas (vaqt limiti bor).
- Debug qilish va lokal test qiyinroq.

**⚠️ Ehtiyot bo'l:** Serverless "har doim arzon" degani emas. Juda yuqori va doimiy trafik bo'lsa, har so'rovga to'lov VM'dan qimmatroqqa tushishi mumkin. Hisob-kitob qil.

## Regions va Availability Zones

Cloud provayderlar dunyo bo'ylab ma'lumotlar markazlariga ega.

- **Region:** Geografik hudud (masalan, `eu-central-1` — Frankfurt). Foydalanuvchilaringga yaqin region tanlash kechikishni (latency) kamaytiradi.
- **Availability Zone (AZ):** Bir region ichidagi alohida, izolyatsiya qilingan ma'lumotlar markazi. Bir AZ ishdan chiqsa, boshqasi ishlashda davom etadi.

```text
Region: eu-central-1
┌─────────────────────────────────────┐
│  ┌────────┐  ┌────────┐  ┌────────┐  │
│  │  AZ-a  │  │  AZ-b  │  │  AZ-c  │  │
│  │ DC #1  │  │ DC #2  │  │ DC #3  │  │
│  └────────┘  └────────┘  └────────┘  │
└─────────────────────────────────────┘
```

**💡 Tushuncha:** Yuqori ishonchlilik (high availability) uchun ilovangni kamida ikkita AZ'ga joylashtir. Shunda bitta ma'lumotlar markazi yonib ketsa ham, xizmat ishlashda davom etadi.

## Auto-scaling

Auto-scaling — yukка qarab server sonini avtomatik oshirish yoki kamaytirish. Masalan, CPU 70%'dan oshsa yangi server qo'shish, 30%'dan pasaysa o'chirish.

- **Horizontal scaling (scale out):** Server sonini ko'paytirish. Cloud uchun afzal.
- **Vertical scaling (scale up):** Bitta serverni kuchaytirish (ko'proq CPU/RAM). Cheklangan.

```text
Yuk past:    [Server]
Yuk o'rta:   [Server] [Server]
Yuk yuqori:  [Server] [Server] [Server] [Server]
```

## Infrastructure as Code (IaC)

**IaC** — infratuzilmani (server, tarmoq, database) qo'lda emas, balki kod orqali yaratish va boshqarish. Eng mashhur vosita — **Terraform**.

Nega kerak:
- **Takrorlanuvchi:** Bir xil infratuzilmani har safar aynan bir xil yaratish.
- **Versiyalash:** Git'da saqlash, o'zgarishlarni ko'rish.
- **Hujjatlashtirilgan:** Kodning o'zi infratuzilma tavsifi.

```text
# Terraform misoli (HCL tili)
resource "aws_instance" "web" {
  ami           = "ami-0abcdef1234567890"
  instance_type = "t3.micro"
  tags = {
    Name = "web-server"
  }
}
```

**💡 Tushuncha:** Terraform "declarative" — sen "nima bo'lishini" yozasan, "qanday qilishni" emas. Terraform joriy holatni ko'rib, kerakli o'zgarishlarni o'zi qiladi.

## Cloud xarajatlarini boshqarish

Cloud arzon ko'rinadi, lekin nazoratsiz qoldirilsa, hisob tez o'sadi. Asosiy usullar:

- **Right-sizing:** Kerak bo'lganidan kattaroq server olmaslik.
- **Reserved instances / Savings plans:** Uzoq muddatga oldindan to'lash — chegirma beradi.
- **Spot instances:** Provayderning bo'sh resurslarini juda arzon ishlatish (lekin istalgan vaqtda to'xtatilishi mumkin).
- **Auto-scaling:** Tunda yoki kam trafikda serverlarni kamaytirish.
- **Tagging va budgeting:** Resurslarni teglab, xarajatni kuzatib borish, byudjet ogohlantirishlari o'rnatish.

**⚠️ Ehtiyot bo'l:** Test uchun ishga tushirilgan katta server yoki ishlatilmagan storage'ni o'chirishni unutma. Eng ko'p "kutilmagan hisob" shundan kelib chiqadi.

## Asosiy provayderlar taqqosi

```text
Xizmat          AWS              GCP                Azure
──────────      ─────────        ─────────          ─────────
VM              EC2              Compute Engine     Virtual Machines
Serverless      Lambda           Cloud Functions    Azure Functions
Object Storage  S3               Cloud Storage      Blob Storage
SQL DB          RDS              Cloud SQL          Azure SQL
NoSQL           DynamoDB         Firestore          Cosmos DB
Kubernetes      EKS              GKE                AKS
Monitoring      CloudWatch       Cloud Monitoring   Azure Monitor
```

- **AWS:** Eng katta, eng ko'p xizmat, eng katta jamoa va hujjatlar.
- **GCP:** Kubernetes va ma'lumotlar/AI sohasida kuchli.
- **Azure:** Microsoft ekotizimi (Windows, .NET, enterprise) bilan yaxshi integratsiya.

## Savol-javoblar

### ❓ Cloud computing nima va u an'anaviy server'dan nimasi bilan farq qiladi?

**✅ Javob:** Cloud computing — bu hisoblash resurslarini (server, storage, database) internet orqali ijaraga olib, ishlatgancha to'lash modeli. An'anaviy modelda sen fizik serverni xarid qilib, ma'lumotlar markazida o'zing boshqarasan. Cloud'da provayder fizik infratuzilmani boshqaradi, sen esa bir necha daqiqada virtual resurslarni yaratasan va kerak bo'lmasa o'chirasan.

### ❓ CapEx va OpEx o'rtasidagi farq nima?

**✅ Javob:** CapEx (Capital Expenditure) — katta dastlabki kapital xarajat, masalan server xarid qilish. OpEx (Operational Expenditure) — doimiy operatsion to'lovlar, masalan serverni oyma-oy ijaraga olish. Cloud OpEx modelini taklif qiladi: oldindan katta pul to'lamasdan, faqat ishlatgan resurs uchun to'laysan. Bu startaplar uchun ayniqsa foydali.

### ❓ IaaS, PaaS va SaaS o'rtasidagi farqni misol bilan tushuntir.

**✅ Javob:** IaaS'da provayder server/tarmoqni beradi, sen OS, runtime va ilovani boshqarasan (AWS EC2). PaaS'da provayder OS va runtime'ni ham boshqaradi, sen faqat kodni joylashtirasan (Heroku). SaaS'da hamma narsa tayyor, sen faqat foydalanasan (Gmail). IaaS'dan SaaS'ga qarab sening mas'uliyating kamayadi, lekin nazorating ham kamayadi.

### ❓ Public, private va hybrid cloud o'rtasidagi farq nima?

**✅ Javob:** Public cloud barcha mijozlarga ochiq umumiy infratuzilma (AWS, GCP). Private cloud faqat bitta tashkilot uchun ajratilgan (banklar, davlat tashkilotlari xavfsizlik uchun ishlatadi). Hybrid cloud ikkalasini birlashtiradi — maxfiy ma'lumot private'da, qolgan yuk public'da ishlaydi.

### ❓ Object storage va block storage o'rtasidagi farq nima?

**✅ Javob:** Object storage (S3) fayllarni HTTP API orqali saqlaydi, cheksiz hajm, arzon, lekin oddiy fayl tizimi emas — backup, rasm, video uchun ideal. Block storage (EBS) esa VM'ga ulanadigan disk, operatsion tizim va database shu yerda ishlaydi, tez va doimiy. Object storage URL orqali, block storage disk sifatida ishlaydi.

### ❓ Serverless (FaaS) nima va uning cold start muammosi nimada?

**✅ Javob:** Serverless — sen serverni ko'rmasdan faqat funksiya yozadigan model (AWS Lambda). Funksiya biror hodisa sodir bo'lganda ishga tushadi va tugaydi, faqat ishlagan millisekundlar uchun to'laysan. Cold start — funksiya uzoq ishlatilmaganda, birinchi so'rovda muhitni tayyorlash uchun qo'shimcha kechikish (yuz millisekundlardan bir necha soniyagacha) yuzaga kelishi.

### ❓ Region va Availability Zone o'rtasidagi farq nima?

**✅ Javob:** Region — geografik hudud (masalan Frankfurt, `eu-central-1`). Availability Zone (AZ) — bir region ichidagi alohida, izolyatsiya qilingan ma'lumotlar markazi. Bir AZ ishdan chiqsa, boshqasi ishlashda davom etadi. Yuqori ishonchlilik uchun ilovani kamida ikkita AZ'ga joylashtirish kerak.

### ❓ Horizontal va vertical scaling farqi nima?

**✅ Javob:** Horizontal scaling (scale out) — server sonini ko'paytirish (1 ta o'rniga 4 ta). Vertical scaling (scale up) — bitta serverni kuchaytirish (ko'proq CPU/RAM). Cloud'da horizontal scaling afzal, chunki cheksiz miqyoslanadi va bitta server o'lsa ham xizmat ishlaydi. Vertical scaling esa apparat cheklovlariga bog'liq.

### ❓ Auto-scaling qanday ishlaydi?

**✅ Javob:** Auto-scaling oldindan belgilangan metrikaga (masalan CPU yuki) qarab server sonini avtomatik o'zgartiradi. Masalan CPU 70%'dan oshsa yangi server qo'shiladi, 30%'dan pasaysa o'chiriladi. Bu yukка mos resurs ta'minlaydi va kam trafikda xarajatni kamaytiradi.

### ❓ Infrastructure as Code (IaC) nima va nega kerak?

**✅ Javob:** IaC — infratuzilmani qo'lda emas, kod orqali yaratish va boshqarish (Terraform). Kerak, chunki: takrorlanuvchi (bir xil infratuzilmani aynan qayta yaratish), versiyalanadigan (Git'da saqlash, o'zgarishlarni ko'rish) va o'zini hujjatlashtiradigan. Qo'lda sozlash xatolarga olib keladi va takrorlash qiyin.

### ❓ Terraform declarative degani nima?

**✅ Javob:** Declarative — sen "nima bo'lishini" tasvirlaysan, "qanday qilishni" emas. Terraform joriy holatni (state) ko'radi va kerakli yakuniy holatga yetish uchun qaysi o'zgarishlarni qilish kerakligini o'zi hisoblaydi. Sen "menga 3 ta server kerak" deysan, Terraform 1 ta bor bo'lsa 2 tasini qo'shadi.

### ❓ Load balancer nima ish qiladi?

**✅ Javob:** Load balancer kelayotgan trafikni bir nechta server o'rtasida taqsimlaydi. Bu yukni teng bo'lib, hech bir server haddan tashqari yuklanmasligini ta'minlaydi. Agar bitta server ishdan chiqsa, load balancer trafikni avtomatik ravishda sog'lom serverlarga yo'naltiradi — bu high availability'ni ta'minlaydi.

### ❓ CDN nima va u nima uchun ishlatiladi?

**✅ Javob:** CDN (Content Delivery Network) statik kontentni (rasm, JS, CSS, video) foydalanuvchiga geografik jihatdan eng yaqin serverdan (edge) yetkazadi. Bu sahifa yuklanish tezligini oshiradi va asosiy serverга yukni kamaytiradi. AWS'da CloudFront. Masalan, Toshkentdagi foydalanuvchi kontentni Amerikadagi serverdan emas, yaqin edge'dan oladi.

### ❓ Managed database'ning afzalligi nima?

**✅ Javob:** Managed database (AWS RDS) provayder tomonidan backup, replikatsiya, versiya yangilash va failover avtomatik boshqariladi. Sen o'zing EC2'ga PostgreSQL o'rnatsang, bularning hammasini o'zing qilishing kerak — backup sozlash, yangilash, monitoring. Managed xizmat vaqtni tejaydi va inson xatosini kamaytiradi.

### ❓ Cloud xarajatlarini qanday kamaytirish mumkin?

**✅ Javob:** Asosiy usullar: right-sizing (kerakli o'lchamdagi serverni tanlash), reserved instances yoki savings plans (uzoq muddatga oldindan to'lab chegirma olish), spot instances (arzon bo'sh resurslar), auto-scaling (kam trafikda serverlarni kamaytirish), tagging va byudjet ogohlantirishlari. Ishlatilmagan resurslarni o'chirish ham muhim.

### ❓ AWS, GCP va Azure o'rtasida qanday farq bor?

**✅ Javob:** AWS eng katta, eng ko'p xizmat va eng katta jamoaga ega. GCP Kubernetes va ma'lumotlar/AI sohasida kuchli (Kubernetes'ni o'zi yaratgan). Azure Microsoft ekotizimi (Windows, .NET, enterprise) bilan eng yaxshi integratsiyalashgan. Tanlov ko'pincha mavjud texnologiyalar, narx va jamoaning tajribasiga bog'liq.

## Masalalar

> Yechimlar: [yechimlar fayli](../solutions/devops/05-cloud.md)

1. Sen yangi startap ochyapsan va dastlab byudjeting juda cheklangan. CapEx vs OpEx nuqtai nazaridan cloud'ni tanlashing afzalliklarini va dastlabki oylarda xarajatni minimal saqlash strategiyangni tushuntir.

2. Bir e-commerce loyihasi uchun arxitektura tuz: foydalanuvchi rasmlari qayerda saqlanadi, database qaysi xizmat, statik kontent qanday yetkaziladi, trafik qanday taqsimlanadi? Har bir komponent uchun AWS xizmatini ko'rsat.

3. Loyihangda ikkita variant bor: (a) doimiy yuqori trafik bilan ishlaydigan API, (b) kuniga bir necha marta ishlaydigan hisobot generatori. Har biri uchun VM va serverless'dan qaysi birini tanlaysan va nega?

4. Sening ilovang faqat bitta Availability Zone'da ishlamoqda. High availability'ni ta'minlash uchun arxitekturani qanday o'zgartirasan? Diagramma chiz.

5. CPU yukiga qarab auto-scaling sozlash kerak: minimal 2 server, maksimal 10 server, CPU 70%'da kengaytirish, 30%'da kamaytirish. Bu siyosatni so'z bilan tasvirla va qaysi metrikalarni kuzatish kerakligini ayt.

6. Bir oy oxirida cloud hisobing kutilganidan 3 baravar ko'p chiqdi. Sababini topish va kelajakda oldini olish uchun qaysi qadamlarni qo'yasan?

7. IaaS, PaaS va SaaS'ga 3 tadan haqiqiy mahsulot misoli yoz va har biri uchun "qaysi qatlamlarni kim boshqaradi"ni jadval ko'rinishida ko'rsat.

8. Terraform yordamida bitta EC2 instance va unga ulangan S3 bucket yaratish kerak bo'lsa, IaC'ning qaysi afzalliklari namoyon bo'ladi? Qo'lda yaratish bilan taqqosla.

9. Bir kompaniya maxfiy moliyaviy ma'lumotni o'z serverida saqlashi shart, lekin web-ilovasini global miqyosda tez ishlatishi kerak. Qaysi deployment modelini tavsiya qilasan va nega?

10. Serverless funksiyangda cold start muammosi foydalanuvchilarni bezovta qilmoqda. Bu muammoni kamaytirish yoki butunlay boshqa yondashuvga o'tish bo'yicha variantlaringni tushuntir.

← [DevOps bo'limiga qaytish](./README.md)
