# Hosting va Infratuzilma

Har qanday tizim oxir-oqibat **biror joyda ishlab turadi** — server, konteyner yoki bulut (cloud). Senior muhandis uchun infratuzilma tanlovi arxitektura tanlovining davomi: qayerda hosting qilish, qanday scale qilish, qancha to'lash, va uzilish (failure) bo'lganda nima bo'lishi — bularning barchasi tizimning ishonchliligi, tezligi va narxini belgilaydi. "Kod yaxshi, lekin infra yomon" — tizim baribir yiqiladi.

Bu hujjat hosting va infratuzilmani senior nuqtai nazaridan O'zbek tilida, ingliz atamalari bilan yoritadi: hosting variantlari (shared → VPS → dedicated → cloud → serverless → PaaS) taqqoslash; cloud asoslari (compute, storage, database, networking, CDN, DNS, load balancer); container va orkestratsiya (Docker, Kubernetes); IaC (Terraform); multi-region va geo-distribution; availability zone va region; auto-scaling; managed vs self-hosted; cost optimizatsiya; provayderlar taqqosi (AWS/GCP/Azure/Cloudflare); edge computing; va infratuzilma monitoringi. Har qaror uchun trade-off ko'rsatiladi.

## Mundarija

- [Tushunchalar: infratuzilma nima uchun muhim](#tushunchalar-infratuzilma-nima-uchun-muhim)
- [Hosting variantlari taqqosi](#hosting-variantlari-taqqosi)
- [Cloud asoslari: compute](#cloud-asoslari-compute)
- [Cloud asoslari: storage](#cloud-asoslari-storage)
- [Cloud asoslari: database](#cloud-asoslari-database)
- [Cloud asoslari: networking](#cloud-asoslari-networking)
- [CDN, DNS, load balancer](#cdn-dns-load-balancer)
- [Containers va orkestratsiya](#containers-va-orkestratsiya)
- [IaC (Infrastructure as Code)](#iac-infrastructure-as-code)
- [Region, availability zone, multi-region](#region-availability-zone-multi-region)
- [Auto-scaling infra darajasida](#auto-scaling-infra-darajasida)
- [Managed vs self-hosted](#managed-vs-self-hosted)
- [Cost optimizatsiya](#cost-optimizatsiya)
- [Provayderlar taqqosi](#provayderlar-taqqosi)
- [Edge computing](#edge-computing)
- [Infratuzilma monitoringi](#infratuzilma-monitoringi)
- [❓ Savol-javoblar](#-savol-javoblar)
- [Masalalar](#masalalar)

---

## Tushunchalar: infratuzilma nima uchun muhim

**💡 Tushuncha:** Infratuzilma qarorlari **teskari qaytarish qiyin** (hard to reverse) va **narx**, **latency**, **availability**ni bevosita belgilaydi. Yaxshi arxitektura yomon infrada past ishlaydi; oddiy arxitektura yaxshi infrada mustahkam turadi.

**💡 Tushuncha:** Zamonaviy infra spektri **boshqaruv** va **moslashuvchanlik** o'rtasidagi murosaga asoslanadi. Bir uchida — o'zingiz boshqaradigan dedicated server (to'liq nazorat, katta mas'uliyat); ikkinchi uchida — serverless (nazorat yo'q, mas'uliyat provayderda). Senior muhandis "qancha boshqaruv **kerak**?" degan savolga javob beradi.

**💡 Tushuncha:** Infratuzilmada uchta doimiy trade-o'lchov: **narx** (cost), **ishonchlilik** (reliability/availability), **tezlik** (latency/performance). Ularni bir vaqtda maksimallashtirib bo'lmaydi — kontekst hal qiladi.

**⚠️ Ehtiyot bo'l:** "Cloud avtomatik arzon/tez/ishonchli" degan mif bor. Cloud noto'g'ri ishlatilsa (over-provisioning, mos kelmagan xizmatlar) an'anaviy serverdan **qimmatroq** bo'lishi mumkin. Cloud imkoniyat beradi, natijani sizning qarorlaringiz belgilaydi.

---

## Hosting variantlari taqqosi

**💡 Tushuncha:** Hosting variantlari boshqaruv/mas'uliyatning kamayib borishi bo'yicha spektr tashkil qiladi:

```text
Ko'proq nazorat, ko'proq mas'uliyat
◄──────────────────────────────────────────────────►
Dedicated  →  VPS  →  Cloud VM  →  Container  →  PaaS  →  Serverless
   │          │         │            │           │          │
 to'liq    virtual   boshqarilgan  orkestr.   platforma  faqat kod
 server    bo'lak    infratuzilma  qatlami    (Heroku)   (Lambda)
Kamroq nazorat, kamroq mas'uliyat
```

| Variant | Nima | Afzallik | Kamchilik | Qachon |
|---------|------|----------|-----------|--------|
| **Shared hosting** | Bir serverni ko'p mijoz bo'lishadi | Eng arzon, sodda | Cheklangan, "noisy neighbor" | Kichik sayt, blog |
| **VPS** | Virtual ajratilgan bo'lak | Arzon, root kirish | Cheklangan resurs, o'zing boqasan | Kichik ilova, side-project |
| **Dedicated** | Butun fizik server siznikida | To'liq nazorat, izolyatsiya | Qimmat, scale sekin | Yuqori/barqaror yuk, compliance |
| **Cloud VM (IaaS)** | Elastik virtual server (EC2) | Elastik, on-demand | O'zing OS/patch boqasan | Ko'p tizim uchun default |
| **PaaS** | Platforma boshqaradi (Heroku) | Tez deploy, infra yashirin | Qimmat scale'da, cheklangan | Startup, tez ishga tushirish |
| **Serverless (FaaS)** | Faqat funksiya (Lambda) | Scale-to-zero, pay-per-use | Cold start, lock-in | Notekis trafik, event ishlovi |

**⚠️ Ehtiyot bo'l:** Ko'p jamoa PaaS'da boshlab, scale/narx muammosiga uchraydi va IaaS/K8s'ga o'tadi. Boshida tezlik uchun PaaS oqilona, lekin **exit strategiyani** oldindan o'ylang.

---

## Cloud asoslari: compute

**💡 Tushuncha:** Cloud'da hisoblash (compute) uch shaklda keladi — abstraksiya darajasi ortib boradi:

```text
VM (virtual machine)  →  Container  →  Serverless (FaaS)
   EC2 nusxasi           ECS/EKS pod     Lambda funksiya
   OS boshqarasan        image deploy    faqat kod
   sekundlarda           millisekund     event-driven
   startup               startup         scale-to-zero
```

| Compute turi | Nima | Qachon |
|--------------|------|--------|
| **VM (EC2)** | To'liq virtual server, OS nazorati | Legacy, maxsus kernel, doimiy yuk, to'liq nazorat |
| **Container** | Izolyatsiyalangan, yengil, portativ | Microservices, CI/CD, portativlik, aralash yuk |
| **Serverless** | Boshqarilmaydigan funksiya | Event ishlovi, notekis trafik, glue kod |

**💡 Tushuncha:** VM'lar **instance family**larga bo'linadi: compute-optimized (CPU og'ir), memory-optimized (RAM og'ir), GPU (ML), storage-optimized (I/O og'ir). **Right-sizing** — ish yukiga mos oilani tanlash — cost va performance uchun kritik.

**⚠️ Ehtiyot bo'l:** Container ≠ VM. Container OS yadrosini host bilan bo'lishadi (yengil, tez), VM esa alohida OS (og'ir, izolyatsiya kuchli). Xavfsizlik izolyatsiyasi kritik bo'lsa, VM chegarasini eslang.

---

## Cloud asoslari: storage

**💡 Tushuncha:** Cloud'da uch xil storage bor, har biri boshqa muammo uchun. Ularni aralashtirib yuborish keng tarqalgan xato.

```text
┌──────────────┬──────────────┬──────────────┐
│ OBJECT       │ BLOCK        │ FILE         │
│ (S3)         │ (EBS)        │ (EFS/NFS)    │
├──────────────┼──────────────┼──────────────┤
│ Rasm, video, │ VM diski,    │ Umumiy       │
│ backup, log  │ DB storage   │ fayl tizimi  │
│ HTTP API     │ VM'ga ulanadi│ ko'p VM      │
│ ∞ scale      │ bitta VM     │ bo'lishadi   │
│ arzon        │ tez, past    │ POSIX        │
│              │ latency      │              │
└──────────────┴──────────────┴──────────────┘
```

| Storage | Nima | Misol foydalanish |
|---------|------|-------------------|
| **Object (S3)** | Kalit-qiymat obyekt do'koni, HTTP orqali | Rasm/video, statik sayt, backup, data lake |
| **Block (EBS)** | Xom disk bloki, VM'ga ulanadi | Ma'lumotlar bazasi diski, OS root |
| **File (EFS)** | Tarmoq fayl tizimi, ko'p VM bo'lishadi | Umumiy fayllar, legacy ilovalar |

**💡 Tushuncha:** Object storage **durability**si juda yuqori (S3 — 99.999999999%, "11 nine"). U mustahkamlikni bir necha AZ'ga replikatsiya orqali beradi. Storage class'lar (Standard → Infrequent → Glacier) — kamroq murojaat qilinadigan ma'lumot arzonroq saqlanadi.

---

## Cloud asoslari: database

**💡 Tushuncha:** Cloud'da DB'ni ikki yo'l bilan ishlatasiz: **managed** (provayder boqadi) yoki **self-hosted** (VM'da o'zingiz). Senior default — managed, agar maxsus sabab bo'lmasa.

| Managed DB | Turi | Qachon |
|------------|------|--------|
| **RDS / Aurora** | Managed SQL (Postgres, MySQL) | Relational, transaction, murakkab query |
| **DynamoDB** | Managed NoSQL key-value | Ulkan scale, oddiy access, predictable latency |
| **ElastiCache** | Managed Redis/Memcached | Cache, session, leaderboard |
| **OpenSearch** | Managed search | To'liq matnli qidiruv, log analitika |

**💡 Tushuncha:** Managed DB sizdan **operatsion yukni** oladi: backup, patch, replication, failover, monitoring. Aurora kabi cloud-native DB'lar storage'ni compute'dan ajratib, avtomatik scale va tez failover beradi.

**⚠️ Ehtiyot bo'l:** Managed DB qulay, lekin **lock-in** va **narx** keltiradi. DynamoDB provayderga chuqur bog'lanadi. Katta scale'da managed DB narxi self-hosted'dan sezilarli oshib ketishi mumkin — hisoblang.

---

## Cloud asoslari: networking

**💡 Tushuncha:** Cloud tarmog'i **VPC (Virtual Private Cloud)** — sizning izolyatsiyalangan virtual tarmog'ingiz. Ichida subnet'lar, routing va security qoidalari.

```text
┌─────────────────── VPC (10.0.0.0/16) ───────────────────┐
│                                                          │
│  ┌── Public Subnet ──┐      ┌── Private Subnet ──┐       │
│  │  Load Balancer    │      │  App servers       │       │
│  │  (internetga oq)  │─────►│  (internetga yopiq)│       │
│  └───────────────────┘      └─────────┬──────────┘       │
│         ▲                             │                  │
│    Internet Gateway            ┌──────▼──────┐           │
│                                │ DB (private)│           │
│                                └─────────────┘           │
│   Security Group: qaysi port/IP ruxsat etilgan           │
└──────────────────────────────────────────────────────────┘
```

| Tushuncha | Nima |
|-----------|------|
| **VPC** | Izolyatsiyalangan virtual tarmoq |
| **Subnet** | VPC ichidagi IP diapazon (public/private) |
| **Security Group** | Instance darajasidagi firewall (port/IP qoidalari) |
| **NACL** | Subnet darajasidagi firewall |
| **Internet/NAT Gateway** | Internetga chiqish/kirish nuqtasi |

**💡 Tushuncha:** **Public subnet** — internetga ochiq (LB, bastion). **Private subnet** — faqat ichkaridan kirish (app, DB). Bu **defense-in-depth**: DB'ni hech qachon to'g'ridan-to'g'ri internetga ochmang.

---

## CDN, DNS, load balancer

**💡 Tushuncha:** Uch tarmoq bloki har tizimda uchraydi:

**DNS** — domen nomini IP'ga tarjima qiladi va **birinchi marshrutlash** qatlami. Route 53 kabi DNS'lar geo-routing (foydalanuvchini eng yaqin region'ga), health-check based failover va weighted routing beradi.

**CDN** — statik (va ba'zan dinamik) kontentni foydalanuvchiga geografik yaqin **edge**'dan yetkazadi. Latency kamayadi, origin server yuki tushadi.

```text
Foydalanuvchi (Toshkent)
     │  DNS ──► eng yaqin edge
     ▼
┌──────────────┐   cache miss   ┌──────────┐
│ CDN Edge     │───────────────►│ Origin   │
│ (Frankfurt)  │◄───────────────│ (US)     │
└──────────────┘   cache fill   └──────────┘
   cache hit → tez qaytaradi (origin'ga bormaydi)
```

**Load balancer** — trafikni bir xil server nusxalari orasida taqsimlaydi:

| LB turi | Qatlam | Nima |
|---------|--------|------|
| **L4 (Network LB)** | TCP/UDP | Tez, mazmunga qaramaydi, ultra-yuqori throughput |
| **L7 (Application LB)** | HTTP | Path/header'ga qarab routing, TLS termination |

**💡 Tushuncha:** LB **health check** qiladi — sog'lom bo'lmagan nusxaga trafik yubormaydi. Bu **SPOF'ni yo'qotish** va **zero-downtime deploy** (rolling)ning asosi.

---

## Containers va orkestratsiya

**💡 Tushuncha:** **Docker** ilovani va uning barcha bog'liqliklarini bitta portativ **image**ga qadoqlaydi. "Mening mashinamda ishlaydi" muammosini yo'qotadi — image har joyda bir xil ishlaydi.

**💡 Tushuncha:** Bir necha container bir mashinada ishlasa boshqarish oson; **yuzlab container o'nlab mashinada** — bu yerda **orkestratsiya** (Kubernetes) kerak. K8s: joylashtirish (scheduling), scaling, self-healing (o'lgan container'ni qayta ishga tushirish), service discovery, rolling deploy.

```text
┌─────────────── Kubernetes Cluster ───────────────┐
│  Control Plane (scheduler, API server)            │
│  ┌────────── Node 1 ──────┐ ┌──── Node 2 ──────┐  │
│  │ ┌────┐ ┌────┐          │ │ ┌────┐ ┌────┐    │  │
│  │ │Pod │ │Pod │          │ │ │Pod │ │Pod │    │  │
│  │ └────┘ └────┘          │ │ └────┘ └────┘    │  │
│  └────────────────────────┘ └──────────────────┘  │
│  Service (stable IP) ──► Pod'lar orasida LB        │
│  Self-healing: pod o'lsa → yangisi tug'iladi       │
└───────────────────────────────────────────────────┘
```

| Nima uchun infra uchun | Sabab |
|------------------------|-------|
| **Portativlik** | Image har cloud/on-prem'da bir xil ishlaydi (lock-in kam) |
| **Zichlik (density)** | Bir mashinada ko'p container (VM'dan arzonroq) |
| **Self-healing** | O'lgan container avtomatik tiklanadi |
| **Deklarativ** | "Men 5 replica xohlayman" → K8s ta'minlaydi |

**⚠️ Ehtiyot bo'l:** Kubernetes kuchli, lekin **murakkab**. Kichik jamoa uchun u ko'pincha ortiqcha yuk — managed container (ECS Fargate, Cloud Run) yoki PaaS yetarli. K8s'ni "kerak bo'lganda" qo'shing.

---

## IaC (Infrastructure as Code)

**💡 Tushuncha:** **IaC** — infratuzilmani **kod bilan** (qo'lda konsolda bosib emas) tavsiflash. Terraform kabi vositalar bilan siz VM, tarmoq, DB'ni **deklarativ** yozasiz; vosita real holatni shu tavsifga keltiradi.

```text
# Terraform (deklarativ): "MEN NIMA XOHLAYMAN"
resource "aws_instance" "web" {
  ami           = "ami-123"
  instance_type = "t3.medium"
  count         = 3
}
# terraform apply → Terraform 3 ta nusxa yaratadi/saqlaydi
```

| Afzallik | Sabab |
|----------|-------|
| **Takrorlanuvchi** | Bir xil infra dev/staging/prod'da |
| **Versiyalangan** | Git'da, kod-review, tarix |
| **Deklarativ** | "Qanday" emas, "nima" — vosita farqni hisoblaydi |
| **Xatolarni kamaytiradi** | Qo'lda "click-ops" o'rniga qayta tiklanuvchi |

**💡 Tushuncha:** IaC **state** tushunchasiga tayanadi — Terraform real infra holatini state faylida saqlaydi va **desired** (kod) bilan **actual** (state) farqini (plan) hisoblab, faqat kerakli o'zgarishni qo'llaydi (idempotent).

**⚠️ Ehtiyot bo'l:** IaC va qo'lda o'zgarishlarni **aralashtirmang** — konsolda qo'lda o'zgartirish (drift) IaC state'ini rassoslashtiradi va keyingi apply kutilmagan natija beradi.

---

## Region, availability zone, multi-region

**💡 Tushuncha:** Cloud infratuzilmasi ierarxiyasi:

```text
┌─────────── REGION (us-east-1, geografik joy) ────────────┐
│   past latency ichida, alohida geografiya                 │
│  ┌── AZ-a ──┐  ┌── AZ-b ──┐  ┌── AZ-c ──┐                 │
│  │ alohida  │  │ alohida  │  │ alohida  │                 │
│  │ data     │  │ data     │  │ data     │                 │
│  │ center   │  │ center   │  │ center   │                 │
│  └──────────┘  └──────────┘  └──────────┘                 │
│   AZ'lar past latency tarmoq bilan bog'langan, lekin       │
│   fizik jihatdan izolyatsiyalangan (alohida quvvat/tarmoq) │
└───────────────────────────────────────────────────────────┘
```

**💡 Tushuncha:** **Availability Zone (AZ)** — region ichidagi izolyatsiyalangan data-markaz. Bir necha AZ'ga tarqatish (**multi-AZ**) bitta data-markaz uzilishidan himoya qiladi — bu **high availability**ning asosi.

**💡 Tushuncha:** **Multi-region** — bir necha geografik region'da deploy. Sabablar: (1) **latency** — foydalanuvchiga yaqin; (2) **disaster recovery** — butun region qulasa; (3) **data residency** — qonun ma'lumotni mamlakat ichida talab qiladi (GDPR).

| Strategiya | Availability | Murakkablik | Narx |
|------------|--------------|-------------|------|
| **Single AZ** | Past | Past | Past |
| **Multi-AZ** | Yuqori | O'rta | O'rta |
| **Multi-region active-passive** | Juda yuqori | Yuqori | Yuqori |
| **Multi-region active-active** | Eng yuqori | Eng yuqori | Eng yuqori |

**⚠️ Ehtiyot bo'l:** Multi-region **data consistency**ni juda qiyinlashtiradi — region'lar orasida ma'lumot replikatsiyasi latency va conflict keltiradi. Buni faqat haqiqatan kerak bo'lganda qiling; ko'p tizim uchun multi-AZ yetarli.

---

## Auto-scaling infra darajasida

**💡 Tushuncha:** **Auto-scaling** — yukga qarab compute nusxalari sonini avtomatik oshirish/kamaytirish. Infra darajasida bu Auto Scaling Group (VM) yoki HPA (Kubernetes pod) orqali amalga oshadi.

```text
Yuk (CPU %) ──► metrik ──► Auto-scaler qarori
   60% ─────► 3 nusxa
   85% ─────► scale UP → 6 nusxa
   20% ─────► scale DOWN → 2 nusxa
```

| Turi | Nima |
|------|------|
| **Horizontal (scale-out)** | Ko'proq nusxa qo'shish (afzal, cheksiz) |
| **Vertical (scale-up)** | Nusxani kattalashtirish (chegarali) |
| **Scheduled** | Ma'lum vaqtda oldindan scale (Black Friday) |
| **Predictive** | ML bilan yukni bashorat qilib oldindan scale |

**💡 Tushuncha:** Yaxshi auto-scaling uchun ilova **stateless** bo'lishi kerak — har nusxa bir xil, session tashqarida (Redis/DB) saqlanadi. Aks holda scale bo'lgan nusxa foydalanuvchi holatini bilmaydi.

**⚠️ Ehtiyot bo'l:** Auto-scaling **darrov emas** — yangi nusxa ishga tushishi vaqt oladi (warm-up). Kutilmagan portlash uchun buffer capacity yoki pre-scaling kerak. Yana DB ko'pincha auto-scale qilmaydi — u yangi bottleneck bo'ladi.

---

## Managed vs self-hosted

**💡 Tushuncha:** Har infra komponenti uchun tanlov: **managed** (provayder boqadi, qulay, qimmatroq, lock-in) yoki **self-hosted** (o'zingiz boqasiz, nazorat, arzon scale'da, operatsion yuk).

```text
                MANAGED                 SELF-HOSTED
Operatsion yuk:  provayderda            sizda
Tezlik:          tez ishga tushirish    sekin
Nazorat:         cheklangan             to'liq
Narx (kichik):   arzon                  qimmat (odam vaqti)
Narx (ulkan):    qimmat                 arzon
Lock-in:         yuqori                 past
```

| Vaziyat | Tavsiya |
|---------|---------|
| Kichik jamoa, tez ishga tushirish | **Managed** (RDS, ElastiCache) |
| Ulkan scale, DB xarajati katta | **Self-hosted** hisobga oling |
| Maxsus konfiguratsiya/versiya | **Self-hosted** |
| Compliance/nazorat talabi | **Vaziyatga qarab** |

**💡 Tushuncha:** Asosiy savol: "Bu komponent bizning **raqobat afzalligimizmi**?" Yo'q bo'lsa (masalan, DB boqish) — managed qiling, jamoa energiyasini mahsulotga yo'naltiring. Ha bo'lsa — nazoratni saqlang.

---

## Cost optimizatsiya

**💡 Tushuncha:** Cloud narxi oson **portlaydi** — elastiklik ehtiyotsizlikni ham osonlashtiradi. Senior muhandis cost'ni arxitektura o'lchovidek biladi.

| Texnika | Nima | Tejash |
|---------|------|--------|
| **Right-sizing** | Ish yukiga mos instance tanlash | Over-provisioning'ni yo'qotadi |
| **Reserved / Savings Plan** | 1-3 yil oldindan majburiyat | 30-70% arzon |
| **Spot instances** | Ortiqcha quvvat, uzilishi mumkin | 70-90% arzon (fault-tolerant ish uchun) |
| **Auto-scaling** | Kam yukda nusxa kamaytirish | Bo'sh quvvat uchun to'lamaslik |
| **Serverless** | Faqat ishlatilgan vaqt | Notekis trafik uchun juda arzon |
| **Storage tiering** | Kam ishlatilgan ma'lumot arzon class'ga | S3 Glacier 10× arzon |
| **Egress diqqati** | Data chiqishi (out) qimmat | CDN cache, region ichida saqlash |

**💡 Tushuncha:** **Serverless narx modeli** — invokatsiya soni × ishlash vaqti × RAM. Kam trafik = deyarli bepul; doimiy yuqori yuk = qimmat. Chegara nuqtasini hisoblang: ma'lum QPS'dan keyin reserved container arzonroq.

**⚠️ Ehtiyot bo'l:** Eng katta yashirin xarajat — **data egress** (cloud'dan tashqariga chiqadigan trafik) va **noto'g'ri arxitektura** (masalan, cross-AZ chatty traffic). Monitoring va cost alert'lar o'rnating; oyning oxirida "kutilmagan hisob" bo'lmasin.

---

## Provayderlar taqqosi

**💡 Tushuncha:** Uchta yirik cloud + edge yetakchi. Ular 80% funksiyada bir xil; farq — ekotizim, narx nuansi va kuchli tomonlar.

| Provayder | Kuchli tomoni | Tipik tanlov |
|-----------|---------------|--------------|
| **AWS** | Eng keng xizmat, yetuk, bozor yetakchisi | Default, keng ehtiyoj, katta ekotizim |
| **GCP** | Data/ML, Kubernetes (GKE), tarmoq | Data-heavy, ML, K8s-first jamoalar |
| **Azure** | Korporativ, Microsoft integratsiyasi | .NET, enterprise, hybrid cloud |
| **Cloudflare** | Edge, CDN, DDoS himoya, Workers | Edge compute, CDN, xavfsizlik qatlami |

**💡 Tushuncha:** **Multi-cloud** modaga aylandi, lekin ko'pincha xato — u murakkablikni ko'paytiradi va har provayderning kuchli xizmatlaridan foydalanishga xalaqit beradi. Aksariyat jamoa uchun bitta asosiy provayder + edge (Cloudflare) yetarli.

**⚠️ Ehtiyot bo'l:** Provayder tanlash **lock-in** darajasini belgilaydi. Managed maxsus xizmatlar (DynamoDB, BigQuery) kuchli, lekin ko'chirishni qiyinlashtiradi. Portativlik muhim bo'lsa — ochiq standartlar (K8s, Postgres) ni afzal ko'ring.

---

## Edge computing

**💡 Tushuncha:** **Edge computing** — hisoblashni foydalanuvchiga **geografik yaqin** (edge location'da) bajarish, markaziy region'ga bormasdan. CDN'ning "aqlli" davomi: nafaqat statik kontent, balki **kod** ham edge'da ishlaydi (Cloudflare Workers, Lambda@Edge).

```text
An'anaviy:  Foydalanuvchi ──── uzoq ──► Markaziy region (kod)
Edge:       Foydalanuvchi ─► yaqin edge (kod) ─┐
                                               └► kerak bo'lsa region
```

| Foyda | Misol |
|-------|-------|
| **Ultra-past latency** | Auth token tekshirish, A/B test, redirect |
| **Origin yukini kamaytirish** | Personalizatsiya edge'da |
| **Geo-mantiq** | Foydalanuvchi joyiga qarab kontent |

**⚠️ Ehtiyot bo'l:** Edge cheklangan muhit — kam CPU/xotira, qisqa ishlash vaqti, cheklangan runtime. Og'ir ish yoki ma'lumotlar bazasiga yaqin ishlash edge uchun emas. Edge — yengil, latency-sezgir mantiq uchun.

---

## Infratuzilma monitoringi

**💡 Tushuncha:** "Ko'ra olmagan narsangizni boshqara olmaysiz." Infra monitoringi uch ustunga tayanadi (observability):

```text
┌──────────┬──────────────┬──────────────┐
│ METRICS  │ LOGS         │ TRACES       │
├──────────┼──────────────┼──────────────┤
│ raqamlar │ voqealar     │ so'rov yo'li │
│ (CPU,    │ (matn        │ (service'lar │
│ RAM, QPS)│  yozuvlar)   │  bo'ylab)    │
│ trend    │ debug/audit  │ latency manba│
└──────────┴──────────────┴──────────────┘
   Prometheus   ELK/Loki     Jaeger/OTel
```

| Ustun | Nima uchun |
|-------|-----------|
| **Metrics** | Trend, alert, dashboard (CPU, xotira, QPS, error rate) |
| **Logs** | Nima bo'ldi — debug va audit |
| **Traces** | Distributed so'rov yo'li — bottleneck qayerda |

**💡 Tushuncha:** Yaxshi monitoring **proaktiv** — SLO/SLI belgilanadi (masalan, "99.9% so'rov < 200ms"), va **alert**lar buzilishdan oldin ishlaydi. Reaktiv monitoring (foydalanuvchi shikoyat qilgach bilish) — kech.

**⚠️ Ehtiyot bo'l:** **Alert fatigue** — juda ko'p alert jamoani "his qilmaydigan" qiladi. Faqat harakat talab qiladigan, foydalanuvchiga ta'sir qiladigan holatlarda alert bering. Qolganini dashboard'da kuzating.

---

## ❓ Savol-javoblar

### ❓ Cloud har doim on-prem'dan arzonmi?

**✅ Javob:** Yo'q. Cloud **elastik** va notekis/o'zgaruvchan yuk uchun arzon — bo'sh quvvat uchun to'lamaysiz, tez ishga tushasiz, capex yo'q. Lekin **doimiy, barqaror, yuqori yuk** uchun on-prem yoki dedicated ko'pincha arzonroq (ayniqsa data egress ko'p bo'lsa). Katta kompaniyalar (Dropbox kabi) ma'lum scale'da cloud'dan qisman on-prem'ga qaytishgan. Qaror — yuk profili va scale'ga bog'liq.

### ❓ Region va Availability Zone farqi nima?

**✅ Javob:** **Region** — alohida geografik joy (us-east-1, eu-west-1), o'z ichida bir necha AZ'ga ega. Region'lar orasidagi latency yuqori (o'nlab-yuzlab ms). **AZ** — region ichidagi izolyatsiyalangan data-markaz (alohida quvvat, sovutish, tarmoq), lekin AZ'lar orasida past-latency tarmoq bor. Multi-AZ = bitta data-markaz uzilishidan himoya; multi-region = butun geografiya uzilishi/latency/data-residency uchun.

### ❓ Nima uchun DB'ni public subnet'ga qo'ymaslik kerak?

**✅ Javob:** Xavfsizlik — **defense-in-depth**. DB internetga to'g'ridan-to'g'ri ochiq bo'lsa, u hujum yuzasi (attack surface) bo'ladi va bitta noto'g'ri konfiguratsiya (zaif parol, ochiq port) to'g'ridan-to'g'ri ma'lumot o'g'irlanishiga olib keladi. DB private subnet'da bo'lsa, unga faqat VPC ichidagi app server'lar kira oladi; hujumchi avval boshqa qatlamni buzishi kerak. Kirish faqat security group orqali app tier'ga cheklanadi.

### ❓ Docker container va VM'ning asosiy farqi nima?

**✅ Javob:** VM o'zining **to'liq OS**iga ega va hypervisor orqali fizik resursni virtualizatsiya qiladi — og'ir (GB'lar), sekin ishga tushadi (sekundlar), lekin kuchli izolyatsiya. Container **host OS yadrosini bo'lishadi** va faqat ilova + bog'liqliklarni qadoqlaydi — yengil (MB'lar), tez (millisekund), zich, lekin izolyatsiya zaifroq (bir yadro). Shuning uchun container tez CI/CD va microservices uchun, VM esa kuchli izolyatsiya yoki maxsus kernel uchun.

### ❓ Kubernetes har doim kerakmi?

**✅ Javob:** Yo'q, ko'pincha ortiqcha. K8s murakkab — u ko'p service, ko'p jamoa, murakkab deploy, portativlik talabi bo'lgan scale'da o'zini oqlaydi. Kichik jamoa yoki bir necha service uchun **managed container** (ECS Fargate, Google Cloud Run) yoki hatto PaaS (Heroku, Render) yetarli va operatsion yuk ancha kam. "Hamma K8s ishlatadi" — K8s'ga o'tish sababi emas; og'riq signali (ko'p service orkestratsiyasi kerakligi) sabab.

### ❓ IaC nima uchun qo'lda konfiguratsiyadan yaxshi?

**✅ Javob:** Qo'lda ("click-ops") konfiguratsiya **takrorlanmaydi, hujjatlanmaydi va xatoga moyil**. IaC (Terraform) infra'ni versiyalangan, review qilinadigan, takrorlanuvchi kodga aylantiradi: dev/staging/prod bir xil bo'ladi, o'zgarishlar Git tarixida, felokatdan tiklash bir buyruq (`terraform apply`). Deklarativ bo'lgani uchun siz "nima kerakligini" yozasiz, vosita "qanday"ini hisoblaydi. Qo'lda o'zgarish esa **drift** (rassoslik) keltiradi.

### ❓ Spot instance nima va qachon ishlatiladi?

**✅ Javob:** Spot instance — cloud provayderning ortiqcha, ishlatilmayotgan quvvati, juda arzon (70-90% chegirma), lekin provayder istalgan payt (odatda 2 daqiqa ogohlantirish bilan) uni **qaytarib olishi** mumkin. Shuning uchun u faqat **fault-tolerant, uzilishga chidamli** ishlar uchun: batch processing, CI/CD, ML training, stateless worker'lar. Kritik, uzluksiz service (masalan, asosiy DB yoki foydalanuvchi so'roviga xizmat qiluvchi API) uchun ishlatmang yoki on-demand bilan aralashtiring.

### ❓ CDN dinamik kontentda ham foydalimi?

**✅ Javob:** Ha, cheklangan darajada. CDN eng ko'p statik kontent (rasm, JS, CSS, video) uchun foydali. Lekin zamonaviy CDN'lar **dynamic content acceleration** ham beradi: optimallashtirilgan tarmoq yo'li (origin'ga tez marshrut), TLS termination edge'da, va edge compute (Workers) bilan personalizatsiya/cache. To'liq personallashtirilgan, cache qilinmaydigan javob uchun foyda kamroq — lekin edge'gacha bo'lgan tarmoq optimizatsiyasi baribir latency'ni kamaytiradi.

### ❓ Active-active va active-passive multi-region farqi nima?

**✅ Javob:** **Active-passive** — bitta asosiy region trafikni qabul qiladi, ikkinchisi kutib turadi (standby) va faqat asosiysi qulasa faollashadi. Sodda, arzonroq, lekin failover vaqti va ma'lumot yo'qolishi (RPO/RTO) bor. **Active-active** — ikkala region ham bir vaqtda trafik qabul qiladi (foydalanuvchini eng yaqiniga marshrutlab). Eng yuqori availability va past latency, lekin ma'lumot **ikki yo'nalishda** replikatsiya bo'lgani uchun conflict resolution va consistency juda murakkab. Ko'p tizim active-passive'dan boshlaydi.

### ❓ Serverless narxi qachon container'dan qimmat bo'ladi?

**✅ Javob:** Serverless invokatsiya soni × ishlash vaqti × ajratilgan xotira bo'yicha to'lanadi. **Kam yoki notekis trafik**da u deyarli bepul (scale-to-zero). Lekin **doimiy yuqori yuk**da har invokatsiya to'lovi yig'ilib, ma'lum QPS chegarasidan keyin doimiy ishlab turgan container (ECS/K8s) ancha arzonlashadi — chunki container'da siz quvvatni "ijaraga" olib, uni to'liq ishlatasiz. Uzoq ishlaydigan yoki og'ir CPU ishlari uchun ham serverless qimmat. Chegara nuqtasini yukingizga qarab hisoblang.

### ❓ Managed DB'ni qachon self-hosted'ga almashtirish kerak?

**✅ Javob:** Managed DB (RDS) qulaylik uchun default, lekin ma'lum scale'da uch sabab self-hosted'ga turtki bo'ladi: (1) **narx** — juda katta DB'da managed premium sezilarli oshadi; (2) **nazorat** — maxsus versiya, extension, tuning yoki konfiguratsiya kerak bo'lsa; (3) **lock-in** — provayderdan chiqish rejasi. Lekin self-hosted operatsion yuk (backup, patch, failover, monitoring) ni sizga qaytaradi — bu odam-vaqt xarajati. Faqat foyda operatsion narxdan oshsa o'ting.

### ❓ Edge computing va CDN bir xilmi?

**✅ Javob:** Yaqin, lekin farqli. **CDN** asosan statik kontentni edge'da **cache** qiladi va yetkazadi. **Edge computing** esa edge location'da **kod ishlatadi** — hisoblash, mantiq, personalizatsiya (Cloudflare Workers, Lambda@Edge). CDN — passiv cache; edge compute — aktiv hisoblash. Ko'p provayder ikkalasini birlashtiradi: CDN infratuzilmasi ustida edge compute quriladi. Edge compute — auth, redirect, A/B test kabi yengil, latency-sezgir mantiq uchun.

### ❓ Auto-scaling nima uchun stateless ilova talab qiladi?

**✅ Javob:** Auto-scaling istalgan payt yangi nusxa qo'shadi yoki mavjudini o'chiradi. Agar ilova **stateful** bo'lsa (foydalanuvchi session yoki ma'lumotni o'z xotirasida saqlasa), o'chirilgan nusxa bilan o'sha holat yo'qoladi, yangi nusxa esa foydalanuvchini "tanimaydi". Shuning uchun holat **tashqarida** (Redis session store, DB) saqlanishi kerak — shunda har nusxa bir xil (fungible) bo'ladi va istalganini o'chirib/qo'shsa bo'ladi. Bu "cattle, not pets" prinsipi.

### ❓ Data egress nima uchun cost muammosi?

**✅ Javob:** Cloud provayderlar ma'lumot **ichkariga** (ingress) kirishini odatda bepul, lekin **tashqariga** (egress) chiqishini — internetga yoki boshqa region/cloud'ga — pulli qiladi, va bu narx yig'ilib katta hisob keltiradi. Shuning uchun senior dizayn: (1) CDN bilan takroriy egress'ni kamaytirish (cache); (2) chatty cross-AZ/cross-region trafik'ni minimallashtirish; (3) ma'lumotni foydalanuvchi va compute bilan bir joyda (data locality) saqlash. Multi-cloud egress esa ayniqsa qimmat — bu multi-cloud'ning yashirin narxi.

### ❓ Infra monitoringida "uch ustun" nima va nima uchun uchalasi kerak?

**✅ Javob:** **Metrics** (raqamli o'lchovlar: CPU, QPS, error rate) — trend va alert uchun, "nimadir buzildi" ni tez ko'rsatadi, lekin **nima uchun**ini emas. **Logs** (matnli voqealar) — "aynan nima bo'ldi"ni debug va audit uchun beradi. **Traces** (distributed so'rov yo'li) — bir so'rov ko'p service bo'ylab qanday o'tganini va **qayerda sekinlashganini** ko'rsatadi. Uchalasi birga to'liq rasm beradi: metric muammoni sezadi, trace joyni topadi, log sababini ochadi. Bittasi yetishmasa, ko'r nuqta qoladi.

---

## Masalalar

> Yechimlar: [solutions/system-design/06-hosting-infrastructure.md](../solutions/system-design/06-hosting-infrastructure.md)

1. **Hosting tanlash.** Quyidagi 4 loyiha uchun eng mos hosting variantini (shared/VPS/dedicated/cloud VM/PaaS/serverless) tanlang va asoslang: (a) shaxsiy portfolio sayti; (b) tez o'sayotgan startup MVP; (c) notekis trafikli event ishlovchi (rasm resize); (d) barqaror yuqori yukli, compliance talab qiladigan bank tizimi.

2. **VPC dizayni.** Uch qatlamli veb-ilova (web → app → DB) uchun VPC tarmoq dizaynini chizing (ASCII): public/private subnet'lar, load balancer, security group qoidalari. DB nima uchun private subnet'da bo'lishini asoslang.

3. **Multi-region qaror.** Bir kompaniya global foydalanuvchiga xizmat qiladi va 99.99% availability xohlaydi. Multi-AZ, active-passive va active-active variantlarini availability, murakkablik, narx va consistency bo'yicha taqqoslang. Qaysi holatda qaysini tanlaysiz?

4. **Cost optimizatsiya.** Oylik cloud hisobi kutilmaganda 3× oshdi. Sababni topish va kamaytirish uchun ketma-ket 6 ta tekshirish/harakat ro'yxatini tuzing (right-sizing, reserved, spot, egress, storage tiering, monitoring).

5. **Container qarori.** Jamoa (8 dev, 5 service) K8s'ga o'tmoqchi. K8s, managed container (Fargate/Cloud Run) va PaaS variantlarini operatsion yuk, moslashuvchanlik va murakkablik bo'yicha taqqoslang va tavsiya bering.

6. **Auto-scaling loyihasi.** E-commerce sayt Black Friday'ga tayyorlanmoqda (10× yuk kutilmoqda). App tier va DB tier uchun scaling strategiyasini loyihalang. DB nima uchun app kabi oson scale qilmasligini va yechimlarni tushuntiring.

7. **IaC migratsiyasi.** Qo'lda ("click-ops") qurilgan infratuzilmani Terraform'ga ko'chirish rejasini tuzing. Drift, state boshqaruvi va bosqichma-bosqich import strategiyasini ko'rsating.

8. **Managed vs self-hosted.** Redis cache uchun managed (ElastiCache) va self-hosted (EC2'da o'zi) variantlarini kichik jamoa va ulkan scale kontekstlarida taqqoslang. Har kontekstda qaysini tanlaysiz va nima uchun?

---

← [System Design bo'limiga qaytish](./README.md)
