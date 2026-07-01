# Hosting va Infratuzilma — Yechimlar

Bu fayl [system-design/06-hosting-infrastructure.md](../../system-design/06-hosting-infrastructure.md) masalalarining namunaviy yechimlarini beradi. Infratuzilmada yagona to'g'ri javob yo'q — muhimi trade-off (narx vs availability vs latency) zanjirini asoslay olish.

## Mundarija

- [1-masala: Hosting tanlash](#1-masala-hosting-tanlash)
- [2-masala: VPC dizayni](#2-masala-vpc-dizayni)
- [3-masala: Multi-region qaror](#3-masala-multi-region-qaror)
- [4-masala: Cost optimizatsiya](#4-masala-cost-optimizatsiya)
- [5-masala: Container qarori](#5-masala-container-qarori)
- [6-masala: Auto-scaling loyihasi](#6-masala-auto-scaling-loyihasi)
- [7-masala: IaC migratsiyasi](#7-masala-iac-migratsiyasi)
- [8-masala: Managed vs self-hosted](#8-masala-managed-vs-self-hosted)

---

## 1-masala: Hosting tanlash

**(a) Portfolio sayti → Shared hosting yoki statik (Netlify/S3+CDN).** Deyarli statik, kam trafik, byudjet muhim. Statik hosting + CDN eng arzon va tez. Server umuman kerak emas.

**(b) Startup MVP → PaaS (Heroku/Render) yoki cloud VM.** Tez ishga tushirish, kam operatsion yuk kritik; infra bilan emas, mahsulot bilan shug'ullaning. PaaS boshlash uchun, keyin scale/narx muammosida cloud VM/container'ga o'tish rejasi bilan.

**(c) Notekis rasm resize → Serverless (Lambda).** Trafik notekis, ish qisqa va event-driven (upload → resize). Scale-to-zero bilan bo'sh vaqtda bepul; portlashda avtomatik scale. Klassik serverless ssenariy.

**(d) Barqaror yuqori yuk + compliance bank → Dedicated yoki cloud VM (reserved).** Yuk barqaror va yuqori (serverless qimmat bo'ladi), compliance nazorat va izolyatsiya talab qiladi. Reserved instance bilan cost past, nazorat to'liq. Ba'zan on-prem/dedicated data-residency uchun.

---

## 2-masala: VPC dizayni

```text
┌──────────────────── VPC (10.0.0.0/16) ────────────────────┐
│                                                            │
│   Internet Gateway                                         │
│         │                                                  │
│  ┌──────▼─────── Public Subnet (10.0.1.0/24) ──────┐       │
│  │   Application Load Balancer (L7)                 │       │
│  │   SG: allow 443 from 0.0.0.0/0                   │       │
│  └──────┬──────────────────────────────────────────┘       │
│         │                                                  │
│  ┌──────▼─────── Private Subnet (10.0.2.0/24) ─────┐       │
│  │   App servers (auto-scaling group)              │       │
│  │   SG: allow 8080 from LB SG only                │       │
│  └──────┬──────────────────────────────────────────┘       │
│         │                                                  │
│  ┌──────▼─────── Private Subnet (10.0.3.0/24) ─────┐       │
│  │   Database (RDS, multi-AZ)                       │       │
│  │   SG: allow 5432 from App SG only                │       │
│  └──────────────────────────────────────────────────┘       │
│   NAT Gateway → app'ga chiqish (patch/update) uchun         │
└────────────────────────────────────────────────────────────┘
```

**DB private subnet'da:** DB internetga to'g'ridan-to'g'ri ochiq bo'lsa, u eng qimmatli aktiv (ma'lumot) uchun hujum yuzasi. Private subnet + security group DB'ga faqat app tier'dan kirishga ruxsat beradi. Hujumchi avval LB, keyin app qatlamini buzishi kerak — defense-in-depth. Har SG faqat oldingi qatlamdan trafik qabul qiladi (least privilege).

---

## 3-masala: Multi-region qaror

| Variant | Availability | Murakkablik | Narx | Consistency |
|---------|--------------|-------------|------|-------------|
| **Multi-AZ** | ~99.95% | O'rta | O'rta | Kuchli (oson) |
| **Active-passive** | ~99.99% | Yuqori | Yuqori | Kuchli, failover lag (RTO/RPO) |
| **Active-active** | 99.99%+ | Eng yuqori | Eng yuqori | Eventual / conflict resolution |

**Qaror:**
- **99.9% yetarli, region ichi** → Multi-AZ. Ko'p tizim uchun bu to'g'ri va arzon default.
- **99.99% + disaster recovery kerak, lekin write bitta joyda** → Active-passive. Standby region asosiysi qulaganda faollashadi.
- **Global past latency + eng yuqori availability** → Active-active. Lekin faqat consistency murakkabligiga (bidirectional replication, conflict resolution) tayyor bo'lsangiz.

**Tavsiya:** Multi-AZ'dan boshlang; haqiqiy talab (SLA, data residency, global latency) bo'lsagina multi-region'ga o'ting. Multi-region — consistency narxi juda katta.

---

## 4-masala: Cost optimizatsiya (3× oshgan hisob)

1. **Cost Explorer / billing breakdown** — qaysi xizmat oshgan? (compute? egress? storage?) Avval sababni aniqlang, ko'r-ko'rona kesmang.
2. **Egress tekshirish** — kutilmagan data chiqishi (yangi feature, cross-region chatty traffic, cache miss)? CDN cache-hit ratio'ni oshiring.
3. **Right-sizing** — over-provisioned instance'lar? CPU/RAM utilizatsiyasini ko'rib, kichikroq oilaga o'tkazing.
4. **Auto-scaling / bo'sh resurs** — kam yukda scale-down ishlayaptimi? Ishlatilmayotgan (idle) nusxa, ulanmagan EBS, eski snapshot'larni o'chiring.
5. **Reserved / Savings Plan** — barqaror baseline yuk uchun on-demand'dan reserved'ga o'ting (30-70% tejash).
6. **Storage tiering** — kam murojaat qilinadigan ma'lumotni S3 Infrequent/Glacier'ga; log retention siyosatini qisqartiring. So'ngida **cost alert va budget** o'rnating.

---

## 5-masala: Container qarori (8 dev, 5 service)

| Variant | Operatsion yuk | Moslashuvchanlik | Murakkablik |
|---------|----------------|------------------|-------------|
| **Self-managed K8s** | Yuqori (cluster boqish) | Eng yuqori | Yuqori |
| **Managed container (Fargate/Cloud Run)** | Past | O'rta-yuqori | Past-o'rta |
| **PaaS (Heroku/Render)** | Eng past | Cheklangan | Eng past |

**Tahlil:** 8 dev, 5 service — bu K8s'ni oqlash uchun kichik. Self-managed K8s butun bir jamoa vaqtini cluster boqishga sarflaydi (upgrade, networking, security).

**Tavsiya: Managed container (Fargate yoki Cloud Run).** Container portativligi va deklarativ deploy'ni beradi, lekin cluster boqish yukisiz. Scale-to-zero (Cloud Run) ham bonus. K8s'ga faqat service soni va murakkablik sezilarli oshganda o'ting. PaaS ham variant, lekin 5 service va o'sish rejasi bilan managed container muvozanatliroq.

---

## 6-masala: Auto-scaling loyihasi (Black Friday, 10×)

**App tier (oson scale):**
- Stateless app → horizontal auto-scaling group (CPU/QPS metrikasiga).
- **Scheduled/pre-scaling** — Black Friday oldidan baseline'ni oldindan oshiring (auto-scale warm-up kechikadi).
- Buffer capacity — kutilmagan portlash uchun ortiqcha zaxira.
- LB health check + rolling deploy.

**DB tier (qiyin scale):**
DB app kabi "nusxa qo'shib" scale bo'lmaydi, chunki yozish (write) bitta primary'ga boradi va ma'lumot holat (state) saqlaydi. Yechimlar:
- **Read replica'lar** — o'qish yukini tarqatish (read-heavy Black Friday'da).
- **Cache (Redis)** — DB'ga bormasdan mashhur ma'lumotni bermoq (eng katta yordam).
- **Connection pooling** — ulanish portlashini boshqarish.
- **Vertikal scale** primary'ni oldindan kattalashtirish (rejalashtirilgan).
- Og'ir yozishni **queue** orqali silliqlash (async).

**Xulosa:** App'ni auto-scale qiling, lekin **DB'ni oldindan** tayyorlang — u yangi bottleneck bo'lishi aniq.

---

## 7-masala: IaC migratsiyasi (click-ops → Terraform)

**Bosqich 1 — Inventarizatsiya.** Mavjud qo'lda qurilgan resurslarni ro'yxatlang (VM, VPC, DB, SG). Muhimlik va bog'liqlik bo'yicha guruhlang.

**Bosqich 2 — State strategiyasi.** Terraform state'ni **remote backend**da (S3 + DynamoDB lock) saqlang, lokal emas — jamoa va lock uchun.

**Bosqich 3 — Bosqichma-bosqich import.** Butun infra'ni birdan emas: (a) Terraform kodini yozing (mavjudga mos); (b) `terraform import` bilan real resursni state'ga bog'lang; (c) `terraform plan` **nol o'zgarish** ko'rsatishini tekshiring (kod real holatga to'g'ri kelishi). Kam-riskli resursdan (masalan, SG, S3) boshlab, kritiklarga (DB) o'ting.

**Bosqich 4 — Drift nazorati.** Import'dan keyin **qo'lda o'zgarishlarni to'xtating** (siyosat/IAM bilan) — aks holda state drift qaytadi. CI'da `terraform plan` bilan drift'ni aniqlang.

**Bosqich 5 — Modullashtirish.** Takrorlanuvchi qismlarni modulga chiqaring; dev/staging/prod uchun bir kod, turli o'zgaruvchi.

---

## 8-masala: Managed vs self-hosted (Redis)

| Kontekst | ElastiCache (managed) | Self-hosted EC2 |
|----------|-----------------------|-----------------|
| **Kichik jamoa** | ✅ Tavsiya | ❌ operatsion yuk katta |
| **Ulkan scale** | Qimmat bo'lishi mumkin | ✅ hisobga arziydi |

**Kichik jamoa → ElastiCache (managed).** Sabab: Redis boqish (failover, patch, backup, monitoring, cluster) — bu jamoaning raqobat afzalligi emas. Managed operatsion yukni oladi, jamoa mahsulotga fokuslanadi. Qo'shimcha narx odam-vaqt tejashi bilan oqlanadi.

**Ulkan scale → self-hosted hisobga oling.** Sabab: (1) juda katta cache klasterda managed premium sezilarli oshadi; (2) maxsus tuning, versiya yoki konfiguratsiya kerak bo'lishi mumkin; (3) lock-in kamayadi. Lekin bu faqat sizda ishonchli SRE/platforma jamoasi bo'lsa mantiqiy — operatsion yuk (24/7 failover boqish) qaytadan sizga o'tadi.

**Asosiy prinsip:** "Bu bizning farqlovchi qadriyatimizmi?" Yo'q bo'lsa — managed; ha yoki narx/nazorat oqlasagina — self-hosted.

---

← [System Design bo'limiga qaytish](../../system-design/README.md)
