# Arxitektura Blueprint va Freymvorklar

Senior darajadagi System Design'ning yuragi — bu **har qanday tizimni noldan loyihalash** qobiliyati. Yaxshi muhandis har safar noldan o'ylab topmaydi; uning boshida **takrorlanuvchi freymvork** va **isbotlangan arxitektura blueprint'lar** palitrasi bor. Xuddi arxitektor uy quriishdan oldin tayyor sxemalarni (blueprint) biladigan kabi, tizim dizayneri ham Monolith, Microservices, Event-driven, CQRS, Hexagonal kabi uslublarni yoddan biladi va **kontekstga qarab** tanlaydi.

Bu hujjat quyidagilarni O'zbek tilida, ingliz atamalari bilan yoritadi: har qanday tizimni loyihalash uchun qadama-qadam reusable freymvork (checklist); arxitektura uslublari (blueprints) — har biri diagramma, afzallik/kamchilik va qachon ishlatilishi bilan; har tizimda ishlatiladigan building block'lar palitrasi; modern design pattern'lar; va "qaysi blueprint qaysi muammoga" degan qaror matritsasi. Maqsad — sizga har qanday "X ni dizayn qiling" savoliga tizimli, senior darajada javob bera oladigan qurol berish.

## Mundarija

- [Tushunchalar: blueprint nima uchun kerak](#tushunchalar-blueprint-nima-uchun-kerak)
- [Universal loyihalash freymvorki (checklist)](#universal-loyihalash-freymvorki-checklist)
- [Monolith](#monolith)
- [Modular monolith](#modular-monolith)
- [Microservices](#microservices)
- [Event-driven architecture](#event-driven-architecture)
- [CQRS](#cqrs)
- [Event Sourcing](#event-sourcing)
- [Hexagonal / Clean / Onion architecture](#hexagonal--clean--onion-architecture)
- [Serverless](#serverless)
- [SOA, Layered, Pipe-and-filter, BFF](#soa-layered-pipe-and-filter-bff)
- [Strangler Fig (migration)](#strangler-fig-migration)
- [Building block'lar palitrasi](#building-blocklar-palitrasi)
- [Qaysi blueprint qaysi muammoga](#qaysi-blueprint-qaysi-muammoga)
- [Monolith'dan microservices'ga qachon o'tish](#monolithdan-microservicesga-qachon-otish)
- [❓ Savol-javoblar](#-savol-javoblar)
- [Masalalar](#masalalar)

---

## Tushunchalar: blueprint nima uchun kerak

**💡 Tushuncha:** Arxitektura **blueprint** — bu ko'p marta isbotlangan tizim tuzilishi shabloni. Uni tanlaganingizda, siz o'nlab yillik tajribani (best practices, ma'lum trade-off'lar, tayyor tooling) meros qilib olasiz. Blueprint'ni bilish — g'ildirakni qayta ixtiro qilmaslik demakdir.

**💡 Tushuncha:** Bitta "eng yaxshi" arxitektura yo'q. Har blueprint'ning **konteksti** bor: jamoa hajmi, scale, domen murakkabligi, deployment tezligi talabi. Senior muhandis blueprint'ni requirement'dan **kelib chiqib** tanlaydi, moda uchun emas.

**💡 Tushuncha:** Arxitektura ikki o'lchamda o'ylanadi: (1) **runtime tuzilishi** — komponentlar qanday joylashadi va gaplashadi (Monolith vs Microservices vs Serverless); (2) **kod tuzilishi** — modullar ichki bog'lanishi (Layered vs Hexagonal vs Clean). Bu ikkisi **ortogonal**: modular monolith Hexagonal bo'lishi mumkin, microservice esa oddiy layered bo'lishi mumkin.

**⚠️ Ehtiyot bo'l:** Eng keng tarqalgan senior xatosi — **premature distribution**. Kichik jamoa darhol 20 ta microservice quradi va operatsion do'zaxga tushadi. Qoida: murakkablikni **kerak bo'lganda** qo'shing, requirement talab qilganda, "kelajakda kerak bo'lishi mumkin" degani uchun emas.

---

## Universal loyihalash freymvorki (checklist)

**💡 Tushuncha:** Quyidagi 8 qadamli freymvork — har qanday "X ni dizayn qiling" savoliga ishlaydigan **reusable checklist**. Uni yoddan bilib oling; intervyuda ham, real ishda ham shu tartibda yuring.

```text
┌────────────────────────────────────────────────────────────┐
│  UNIVERSAL SYSTEM DESIGN FREYMVORK                          │
├────────────────────────────────────────────────────────────┤
│ 1. REQUIREMENTS   → functional + non-functional (NFR)      │
│ 2. CONSTRAINTS    → scale: QPS, storage, users, latency SLA│
│ 3. HIGH-LEVEL     → asosiy komponentlar diagrammasi        │
│ 4. DATA MODEL     → schema, SQL/NoSQL, access pattern'lar  │
│ 5. API DESIGN     → endpoint'lar / event'lar kontrakti     │
│ 6. DEEP DIVE      → 1-2 kritik komponentni chuqur          │
│ 7. TRADE-OFF      → SPOF, bottleneck, CAP, cost            │
│ 8. EVOLVE         → tizim kelajakda qanday o'sadi          │
└────────────────────────────────────────────────────────────┘
```

**1. Requirements.** Functional (tizim NIMA qiladi) va non-functional (QANDAY: scalability, latency, availability, consistency, durability, security). Avval **scope'ni toraytiring** — intervyuer bilan nimaga e'tibor qaratishni kelishing.

**2. Constraints / scale.** Back-of-envelope: kunlik faol foydalanuvchi (DAU), o'qish/yozish nisbati (read:write ratio), QPS (peak = o'rtacha × 2–3), storage o'sishi, bandwidth. Bu raqamlar arxitektura tanlovini **majburlaydi**.

**3. High-level components.** Kutubxona, client, API gateway, service'lar, DB, cache, queue — quti va o'qlar (box-and-arrow) diagrammasi.

**4. Data model.** Asosiy entity'lar, munosabatlar, access pattern (kim nima o'qiydi/yozadi). SQL vs NoSQL qaroriga shu bosqichda kelasiz.

**5. API design.** REST/gRPC endpoint'lari yoki event kontraktlari. Idempotency, pagination, versioning.

**6. Deep dive.** Intervyuer qiziqqan yoki eng murakkab qismni (masalan, feed generatsiyasi, matching, ranking) chuqurlashtiring.

**7. Trade-off.** Single Point of Failure (SPOF) qayerda? Bottleneck qayerda? CAP bo'yicha nimani qurbon qildingiz? Cost qancha?

**8. Evolve.** 10× o'sganda nima sinadi? Qaysi komponent birinchi bo'lib ajratiladi (extract)?

**⚠️ Ehtiyot bo'l:** Qadamlarni tartibida yuring, lekin **qattiq** emas — intervyuer signaliga qarab chuqurlashtiring. Checklist yo'lboshchi, kishan emas.

---

## Monolith

**💡 Tushuncha:** **Monolith** — butun ilova bitta deployable birlik (bitta process, bitta codebase, bitta DB). Barcha modullar bir xotira maydonida, funksiya chaqiruvlari orqali gaplashadi.

```text
┌─────────────────────────────────┐
│           MONOLITH              │
│  ┌────────┐ ┌────────┐ ┌──────┐ │
│  │  Auth  │ │ Orders │ │ Ship │ │   → bitta process
│  └────────┘ └────────┘ └──────┘ │
│         funksiya chaqiruvi       │
└──────────────┬──────────────────┘
               │
          ┌────▼────┐
          │   DB    │
          └─────────┘
```

| Afzallik | Kamchilik |
|----------|-----------|
| Sodda deploy, debug, test | Katta hajmda build/deploy sekinlashadi |
| Transaction oson (bitta DB) | Bitta modul crash butun tizimni yiqitadi |
| Latency past (in-process call) | Jamoa kattalashsa, kod chalkashadi |
| Boshlash tezligi eng yuqori | Bitta til/stack'ga bog'lanish |

**✅ Qachon:** Yangi startup, MVP, kichik jamoa (<15 dev), domen hali noaniq. **Deyarli har doim monolith'dan boshlang.**

---

## Modular monolith

**💡 Tushuncha:** **Modular monolith** — bitta deployable birlik, lekin ichida qattiq chegaralangan modullar (bounded context'lar). Modullar faqat ochiq interfeys orqali gaplashadi, bir-birining DB jadvaliga to'g'ridan-to'g'ri tegmaydi.

```text
┌──────────────────────────────────────────┐
│         MODULAR MONOLITH                  │
│  ┌─────────┐   API   ┌─────────┐          │
│  │ Orders  │◄───────►│Shipping │          │
│  │ [schema]│         │[schema] │          │
│  └─────────┘         └─────────┘          │
│   har modul o'z schema/interfeysi bilan   │
└──────────────────────────────────────────┘
```

| Afzallik | Kamchilik |
|----------|-----------|
| Monolith soddaligi + toza chegaralar | Hali ham bitta deploy |
| Microservices'ga o'tish oson (extract) | Chegara intizomini talab qiladi |
| Bitta transaction saqlanadi | Til/runtime hali yagona |

**✅ Qachon:** Domen aniqlanyapti, jamoa o'syapti, lekin distributed murakkablikka hali tayyor emassiz. **Aksariyat kompaniyalar uchun eng yaxshi "default".**

---

## Microservices

**💡 Tushuncha:** **Microservices** — tizim mustaqil deploy qilinadigan mayda servicelarga bo'linadi. Har biri o'z DB'siga ega (**database-per-service**), tarmoq orqali (REST/gRPC/queue) gaplashadi, mustaqil scale va deploy qilinadi.

```text
   ┌──────────────┐
   │ API Gateway  │
   └──┬────┬───┬──┘
   ┌──▼─┐┌─▼──┐┌▼────┐
   │Auth││Ordr││Ship │   → har biri alohida process/deploy
   └─┬──┘└─┬──┘└──┬──┘
   ┌─▼─┐  ┌▼─┐   ┌▼─┐
   │DB │  │DB│   │DB│    → database-per-service
   └───┘  └──┘   └──┘
```

| Afzallik | Kamchilik |
|----------|-----------|
| Mustaqil deploy va scale | Distributed murakkablik (tarmoq, latency) |
| Xatolik izolyatsiyasi | Transaction murakkab (saga kerak) |
| Jamoalar mustaqil ishlaydi | Operatsion overhead (monitoring, tracing) |
| Polyglot (har xil til/DB) | Data consistency eventual bo'ladi |

**✅ Qachon:** Katta jamoa (ko'p team), turli scale profillar, mustaqil release tezligi kerak. Aniq bounded context'lar mavjud.

**⚠️ Ehtiyot bo'l:** Microservices — **tashkiliy** yechim, texnik emas. Conway qonuni: arxitektura jamoa tuzilishini aks ettiradi. Bitta jamoa uchun 10 ta service — anti-pattern.

---

## Event-driven architecture

**💡 Tushuncha:** **Event-driven architecture (EDA)** — komponentlar bir-birini to'g'ridan-to'g'ri chaqirmaydi, balki **event** (voqea) e'lon qiladi va boshqalar unga obuna bo'ladi (**pub/sub**). Aloqa **asinxron** va **decoupled** (bir-biriga bog'lanmagan).

```text
                ┌──────────────┐
   publish      │  EVENT BUS   │   subscribe
  ┌────────────►│ (Kafka/SNS)  │◄────────────┐
  │             └──────┬───────┘             │
┌─┴──────┐            │            ┌─────────┴─┐
│ Order  │  "OrderPlaced"          │ Shipping  │
│Service │            ├───────────►│ Service   │
└────────┘            │            └───────────┘
                      ├───────────►┌───────────┐
                      │            │ Analytics │
                      │            └───────────┘
                      └───────────►┌───────────┐
                                   │  Email    │
                                   └───────────┘
```

| Afzallik | Kamchilik |
|----------|-----------|
| Yuqori decoupling | Oqim (flow) kuzatish qiyin |
| Yangi consumer qo'shish oson | Eventual consistency |
| Yuk cho'qqilarini yutadi (buffer) | Debugging murakkab (async) |
| Scalable, resilient | Event schema evolutsiyasi qiyin |

**✅ Qachon:** Ko'p consumer bitta voqeaga reaksiya qilishi kerak; komponentlarni ajratmoqchisiz; yuk portlashlarini silliqlash kerak (order processing, notifications, IoT).

**💡 Tushuncha:** Ikki uslub: **event notification** (yengil "nimadir bo'ldi" xabari) va **event-carried state transfer** (event ichida to'liq ma'lumot, consumer DB'ga bormasdan ishlaydi).

---

## CQRS

**💡 Tushuncha:** **CQRS (Command Query Responsibility Segregation)** — yozish (command) va o'qish (query) yo'llarini **ajratish**. Yozish modeli normalizatsiya qilingan va biznes qoidalariga, o'qish modeli esa denormalizatsiya qilingan va tez o'qishga optimallashtirilgan.

```text
        ┌──────────┐              ┌──────────┐
Write──► │ COMMAND  │──►Write DB──►│  sync    │
         │ model    │  (SQL,      │ (event/  │
         └──────────┘   normal)   │  CDC)    │
                                  └────┬─────┘
                                       ▼
        ┌──────────┐              ┌──────────┐
Read ◄── │  QUERY   │◄──Read DB◄───│ Read DB  │
         │ model    │  (denorm,   │(Elastic, │
         └──────────┘   fast)     │ Redis)   │
```

| Afzallik | Kamchilik |
|----------|-----------|
| O'qish va yozishni alohida scale | Ikki model — murakkablik |
| Har model o'z DB/storage'iga | Eventual consistency (read lag) |
| Murakkab query'lar optimizatsiyasi | Sync mexanizmi kerak |

**✅ Qachon:** O'qish:yozish nisbati juda nomutanosib (masalan, 100:1); murakkab reporting; turli o'qish shakllari kerak. Ko'pincha Event Sourcing bilan birga.

**⚠️ Ehtiyot bo'l:** CQRS'ni har CRUD ilovaga tiqmang. U murakkab domen va yuqori scale uchun. Oddiy tizimda ortiqcha yuk.

---

## Event Sourcing

**💡 Tushuncha:** **Event Sourcing** — joriy holatni (current state) saqlash o'rniga, **barcha o'zgarishlarni event ketma-ketligi** sifatida saqlash. Joriy holat event'larni qayta o'ynash (replay) orqali qayta tiklanadi. Bank ledger (defter) kabi: balansni emas, har tranzaksiyani yozasiz.

```text
Event log (append-only, immutable):
  ┌──────────────┐┌──────────────┐┌──────────────┐
  │AccountOpened ││ Deposited    ││ Withdrew     │
  │ balance:0    ││ +100         ││ -30          │
  └──────────────┘└──────────────┘└──────────────┘
        replay ──────────────────────► state: 70
```

| Afzallik | Kamchilik |
|----------|-----------|
| To'liq audit trail (tabiiy) | O'qish uchun replay/snapshot kerak |
| Vaqt bo'yicha "orqaga qaytish" | Katta o'rganish egri chizig'i |
| Event'lar CQRS read model'ini quradi | Event schema o'zgarishi qiyin |
| Xatolarni qayta hisoblash mumkin | Storage ko'p o'sadi |

**✅ Qachon:** Audit majburiy (moliya, sog'liq); "nima uchun holat shunday?" savoliga javob kerak; temporal query'lar (o'tmishdagi holat).

**💡 Tushuncha:** Performance uchun **snapshot** ishlatiladi — vaqti-vaqti bilan joriy holatni saqlab, replay'ni qisqartirish.

---

## Hexagonal / Clean / Onion architecture

**💡 Tushuncha:** Bu uch nom — bir g'oyaning variantlari: **biznes mantiqni (domain/core) infratuzilmadan (DB, web, tashqi API) ajratish**. Yadro tashqi dunyoni bilmaydi; bog'lanish **ports & adapters** orqali. Bog'liqlik doim **ichkariga** yo'naladi (Dependency Rule).

```text
                 ┌───────────────────────────┐
                 │      ADAPTERS (tashqi)     │
                 │  ┌─────────────────────┐   │
                 │  │      PORTS          │   │
                 │  │  ┌───────────────┐  │   │
   HTTP  ───────►│──┤► │  DOMAIN CORE  │  ◄┼───┼──► DB
   REST          │  │  │ (biznes mantiq)│  │   │   (Postgres)
                 │  │  └───────────────┘  │   │
   CLI  ────────►│──┤►  use-case'lar      ◄┼───┼──► Kafka
                 │  └─────────────────────┘   │
                 └───────────────────────────┘
    Bog'liqlik doim ICHKARIGA (Dependency Rule)
```

**Ports** — yadro belgilagan interfeyslar (masalan `OrderRepository`). **Adapters** — ularning konkret implementatsiyasi (`PostgresOrderRepository`). DB'ni almashtirsangiz, faqat adapter o'zgaradi, yadro tegilmaydi.

| Afzallik | Kamchilik |
|----------|-----------|
| Biznes mantiq testlash oson (mock adapter) | Ko'proq boilerplate/interfeys |
| Infratuzilma almashtiriladi | Kichik ilovada ortiqcha |
| Domenga e'tibor markazda | O'rganish kerak |

**✅ Qachon:** Murakkab biznes mantiq; uzoq umr ko'radigan tizim; texnologiya almashishi ehtimoli. Bu **kod tuzilishi** blueprint'i — monolith yoki microservice ichida ishlatiladi.

---

## Serverless

**💡 Tushuncha:** **Serverless (FaaS)** — siz server boshqarmaysiz; kodni funksiya sifatida yozasiz, provayder (AWS Lambda) uni **hodisaga javoban** ishga tushiradi, avtomatik scale qiladi, va faqat **ishlatilgan vaqt** uchun to'laysiz. Trafik yo'q — narx nol.

```text
  HTTP event ──►┌─────────┐
                │ Lambda  │──► DynamoDB
  Queue msg ───►│ funksiya│──► S3
                └─────────┘
  Kerak bo'lganda ishlaydi, keyin so'nadi (scale-to-zero)
```

| Afzallik | Kamchilik |
|----------|-----------|
| Server boshqaruvi yo'q | Cold start latency |
| Avtomatik scale (0 dan ∞) | Vendor lock-in |
| Pay-per-use (arzon, kam trafik) | Uzoq/og'ir ish uchun qimmat |
| Tez ishga tushirish | Debug/local test qiyin |

**✅ Qachon:** Notekis/kam trafik, event ishlovi, cron ishlar, glue kod, tez prototip. **⚠️ Ehtiyot bo'l:** Doimiy yuqori yuk yoki uzoq ishlarga serverless qimmatga tushadi — bu yerda container arzonroq.

---

## SOA, Layered, Pipe-and-filter, BFF

**💡 Tushuncha:** **SOA (Service-Oriented Architecture)** — microservices'ning "kattaroq" ajdodi: yirikroq service'lar, ko'pincha markaziy ESB (Enterprise Service Bus) orqali. Microservices — SOA'ning yengil, decentralized versiyasi.

**💡 Tushuncha:** **Layered (n-tier)** — klassik qatlamlar: Presentation → Business → Data Access → DB. Har qatlam faqat pastdagiga tayanadi. Sodda, tushunarli, lekin qatlamlar orasidan "sizib o'tish" (leaky) va performance narxi bor.

```text
Presentation  →  Business Logic  →  Data Access  →  DB
   (UI/API)         (services)        (repository)
```

**💡 Tushuncha:** **Pipe-and-filter** — ma'lumot filtrlar (qayta ishlash bosqichlari) zanjiridan pipe'lar orqali oqadi. ETL, video kodlash, log ishlovi uchun ideal. Har filter mustaqil, qayta ishlatiladi.

```text
  Input ─►[Filter A]─►[Filter B]─►[Filter C]─► Output
```

**💡 Tushuncha:** **Backend-for-Frontend (BFF)** — har client turi (web, iOS, Android) uchun alohida backend qatlam. U bir nechta downstream service'ni chaqirib, aynan shu client'ga mos javob tayyorlaydi. Client kodini soddalashtiradi, over-fetching'ni kamaytiradi.

```text
  Web  ──► [Web BFF]  ──┐
  iOS  ──► [iOS BFF]  ──┼──► Orders, Users, Catalog services
  And  ──► [And BFF]  ──┘
```

**✅ Qachon (BFF):** Turli client'lar juda turlicha ma'lumot talab qilsa; API gateway'dan aqlliroq per-client mantiq kerak bo'lsa.

---

## Strangler Fig (migration)

**💡 Tushuncha:** **Strangler Fig** — eski (legacy) tizimni birdan almashtirmasdan, **asta-sekin** bo'g'ib almashtirish pattern'i. Nomi tropik "bo'g'uvchi anjir"dan: u daraxtni o'rab, asta-sekin o'rnini egallaydi. Yangi tizim eski funksiyalarni birma-bir "yutadi".

```text
Bosqich 1:            Bosqich 2:            Bosqich 3:
┌─────────┐          ┌─────────┐           ┌─────────┐
│ Facade/ │          │ Facade/ │           │ Facade/ │
│ Proxy   │          │ Proxy   │           │ Proxy   │
└──┬───┬──┘          └──┬───┬──┘           └────┬────┘
   │   │                │   │                   │
┌──▼┐ ┌▼──────┐      ┌──▼┐ ┌▼──────┐        ┌───▼────┐
│New│ │Legacy │      │New│ │Legacy │        │  New   │
│10%│ │ 90%   │      │60%│ │ 40%   │        │ 100%   │
└───┘ └───────┘      └───┘ └───────┘        └────────┘
```

| Afzallik | Kamchilik |
|----------|-----------|
| Past risk (bosqichma-bosqich) | Uzoq davom etadi |
| Ishlab turgan tizim buzilmaydi | Facade/routing qatlami kerak |
| Rollback oson | Ikki tizim parallel (murakkablik) |

**✅ Qachon:** Katta legacy monolith'ni microservices'ga yoki yangi platformaga ko'chirish. **Big-bang rewrite'dan doim xavfsizroq.**

---

## Building block'lar palitrasi

**💡 Tushuncha:** Blueprint'lardan qat'i nazar, har tizim shu **standart qurilish bloklari**dan yig'iladi. Ularni Lego kabi biling — qaysi biri qaysi muammoni hal qilishini.

```text
Client
  │
┌─▼───────────┐    ┌──────────┐
│    CDN      │    │   DNS    │  → statik kontent, marshrutlash
└─┬───────────┘    └──────────┘
┌─▼───────────┐
│Load Balancer│  → trafikni serverlarga taqsimlash (SPOF'ni yo'qotish)
└─┬───────────┘
┌─▼───────────┐
│ API Gateway │  → auth, rate-limit, routing, aggregation
└─┬───────────┘
┌─▼──────┐ ┌────────┐ ┌──────────┐
│Service │ │ Cache  │ │  Search  │
│        │ │(Redis) │ │(Elastic) │
└─┬──────┘ └────────┘ └──────────┘
┌─▼──────┐ ┌────────────┐ ┌──────────────┐
│  DB    │ │Message Queue│ │Object Storage│
│SQL/NoSQL│ │(Kafka/SQS)  │ │   (S3)       │
└────────┘ └──────┬─────┘ └──────────────┘
                  ▼
            ┌──────────┐
            │ Worker / │  → og'ir ishlarni async bajarish
            │ Job queue│
            └──────────┘
```

| Block | Muammo hal qiladi |
|-------|-------------------|
| **API Gateway** | Yagona kirish nuqtasi: auth, rate-limit, routing |
| **Load Balancer** | Trafik taqsimoti, SPOF'ni yo'qotish, health check |
| **Cache** (Redis) | Latency past, DB yukini kamaytirish |
| **Message Queue** (Kafka/SQS) | Decoupling, buffering, async ishlov |
| **DB SQL** | Kuchli consistency, transaction, murakkab query |
| **DB NoSQL** | Yuqori scale, moslashuvchan schema, denormalizatsiya |
| **Search index** (Elastic) | To'liq matnli qidiruv, ranking, facet |
| **Object storage** (S3) | Katta binar fayllar (rasm, video) arzon saqlash |
| **CDN** | Statik kontentni foydalanuvchiga yaqin yetkazish |
| **Worker / job queue** | Og'ir/uzoq ishlarni fon rejimida bajarish |

**⚠️ Ehtiyot bo'l:** Har block'ni "chunki hamma ishlatadi" deb qo'shmang. Har biri **operatsion narx** (monitoring, failure mode). Faqat requirement talab qilsa qo'shing.

---

## Qaysi blueprint qaysi muammoga

**💡 Tushuncha:** Quyidagi jadval — tez qaror matritsasi. Muammodan blueprint'ga.

| Muammo / talab | Tavsiya etilgan blueprint |
|----------------|---------------------------|
| Yangi startup, MVP, tez ishga tushirish | **Monolith** |
| O'sib borayotgan domen, toza chegara kerak | **Modular monolith** |
| Katta jamoa, mustaqil deploy | **Microservices** |
| Ko'p consumer bir voqeaga reaksiya | **Event-driven (pub/sub)** |
| O'qish:yozish keskin nomutanosib | **CQRS** |
| To'liq audit, temporal query | **Event Sourcing** |
| Murakkab biznes mantiq, uzoq umr | **Hexagonal / Clean** |
| Notekis/kam trafik, event ishlovi | **Serverless** |
| Turli client'lar turli data | **BFF** |
| Legacy'dan ko'chish | **Strangler Fig** |
| ETL, ma'lumot oqimi ishlovi | **Pipe-and-filter** |

**💡 Tushuncha:** Blueprint'lar **birga** ishlatiladi. Real tizim: Microservices (runtime) + har service Hexagonal (kod) + EDA (aloqa) + CQRS (og'ir o'qish service'da) + BFF (client qatlam). Bu kombinatsiya — norma.

---

## Monolith'dan microservices'ga qachon o'tish

**💡 Tushuncha:** O'tish qaroriga **texnik sabab** emas, **og'riq signallari** turtki bo'ladi. Signal yo'q bo'lsa, monolith'da qoling.

```text
O'TISH SIGNALLARI (og'riq):
  ✓ Deploy qo'rquvi — bitta o'zgarish butun tizimni riskga soladi
  ✓ Jamoalar bir-birini bloklayapti (merge do'zaxi)
  ✓ Modullar juda turli scale profilga ega
  ✓ Turli qismlar turli texnologiya talab qiladi
  ✓ Build/test vaqti chidab bo'lmas darajada

HALI O'TMANG, agar:
  ✗ Jamoa < 15 dev
  ✗ Domen chegaralari hali noaniq
  ✗ Operatsion tayyorlik yo'q (CI/CD, monitoring, tracing)
```

**💡 Tushuncha:** To'g'ri yo'l: **Monolith → Modular monolith → tanlab extract**. Avval monolith ichida toza modul chegaralarini quring; keyin eng og'riqli modulni (masalan, notifications yoki payments) birinchi bo'lib service'ga ajrating. Strangler Fig pattern'idan foydalaning.

**⚠️ Ehtiyot bo'l:** "Microservices bizni tez qiladi" — noto'g'ri. Microservices sizni **mustaqil** qiladi, lekin **sekinroq** har birlik uchun (tarmoq, versioning, distributed debugging). Foyda tashkiliy miqyosda paydo bo'ladi.

---

## ❓ Savol-javoblar

### ❓ Monolith har doim yomonmi?

**✅ Javob:** Yo'q — bu keng tarqalgan noto'g'ri tushuncha. Monolith sodda, tez, latency past va aksariyat kompaniyalar uchun **to'g'ri tanlov**. "Monolith yomon" degan mif microservices modasidan kelib chiqqan. Aslida yomon **ball of mud** (chalkash, chegarasiz monolith) yomon, monolith'ning o'zi emas. Yaxshi tuzilgan modular monolith yillar davomida a'lo ishlaydi.

### ❓ Microservices va SOA farqi nima?

**✅ Javob:** Ikkalasi ham service'larga bo'lish g'oyasiga asoslanadi, lekin: SOA yirikroq service'lar va markaziy ESB (Enterprise Service Bus) ishlatadi, ko'pincha korporativ standartlarga bog'lanadi. Microservices — mayda, mustaqil, **decentralized** (har service o'z DB va deploy'i), yengil aloqa (REST/gRPC/queue), ESB'siz. Microservices — SOA'dan olingan saboqlarning zamonaviy, yengil varianti.

### ❓ Event-driven arxitekturada eng katta muammo nima?

**✅ Javob:** **Kuzatuvchanlik (observability)** va **debugging**. Sinxron chaqiruvda oqim ko'rinadi; event-driven'da esa "bu event kim tomonidan, qachon, nima uchun ishlandi?" degan savolga javob berish qiyin. Yechim: distributed tracing (correlation ID har event'da), event schema registry, va dead-letter queue (ishlanmagan event'lar uchun). Yana bir muammo — eventual consistency: consumer'lar turli vaqtda yangilanadi.

### ❓ CQRS'da read va write DB qanday sinxronlanadi?

**✅ Javob:** Uch asosiy usul: (1) **event/message** — write tarafi event chiqaradi, read tarafi obuna bo'lib o'z DB'sini yangilaydi; (2) **CDC (Change Data Capture)** — write DB'ning transaction log'ini o'qib, o'zgarishlarni read DB'ga ko'chirish (Debezium); (3) **materialized view** — DB darajasida. Barchasida read DB ozgina orqada qoladi (**eventual consistency**), shuning uchun "yozdim, darhol o'qidim" ssenariysida read lag'ni hisobga oling.

### ❓ Event Sourcing va oddiy audit log farqi nima?

**✅ Javob:** Audit log — **qo'shimcha** yozuv; joriy holat baribir alohida saqlanadi va "haqiqat manbai" (source of truth) bo'ladi. Event Sourcing'da esa **event'lar o'zi source of truth** — joriy holat ular ustidan hisoblanadi (derived). Ya'ni audit log'da state va log ikkilanadi (rasoslashmasligi mumkin), Event Sourcing'da esa faqat event'lar bor va state doim ulardan kelib chiqadi.

### ❓ Hexagonal arxitektura microservices'ni almashtiradimi?

**✅ Javob:** Yo'q — ular **turli o'lchamlarda**. Microservices — runtime/deployment tuzilishi (nechta process, qanday joylashgan). Hexagonal — bitta service ichidagi **kod tuzilishi** (biznes mantiq vs infratuzilma ajratimi). Siz Hexagonal monolith ham, Hexagonal microservice ham qurishingiz mumkin. Ular bir-birini almashtirmaydi, birga ishlatiladi.

### ❓ Serverless doimo arzonmi?

**✅ Javob:** Yo'q. Serverless **kam yoki notekis trafik**da arzon (scale-to-zero, pay-per-use). Lekin doimiy yuqori yukda, uzoq ishlaydigan jarayonlarda yoki og'ir CPU ishlarida u **qimmatlashadi** — har invokatsiya va ishlash vaqti uchun to'lov container'dan oshib ketadi. Ma'lum bir yuk chegarasidan keyin reserved container'lar (ECS/K8s) arzonroq. Yana cold start latency va vendor lock-in narxi bor.

### ❓ API Gateway va Load Balancer farqi nima?

**✅ Javob:** **Load Balancer** — L4/L7 darajada trafikni bir xil server nusxalari orasida taqsimlaydi (health check, session affinity). U "aqlsiz" — mazmunga qaramaydi. **API Gateway** — L7 aqlli qatlam: autentifikatsiya, rate limiting, request routing (qaysi service'ga), response aggregation, protokol tarjimasi. Ko'pincha ikkalasi ketma-ket: LB gateway nusxalariga trafik tarqatadi, gateway esa backend service'larga aqlli marshrutlaydi.

### ❓ BFF va API Gateway bir xilmi?

**✅ Javob:** Yaqin, lekin farqli. API Gateway — **umumiy** kirish nuqtasi, barcha client'lar uchun bir xil. BFF — **har client turi uchun alohida** backend, aynan shu client ehtiyojiga moslangan (web BFF, mobile BFF). BFF client-specific aggregation va transformatsiya qiladi. Ba'zan BFF gateway ustida quriladi. Agar client'lar juda turlicha bo'lsa — BFF; bir xil bo'lsa — oddiy gateway yetadi.

### ❓ Modular monolith bilan microservices'ning haqiqiy farqi nima, agar ikkovida ham chegara bo'lsa?

**✅ Javob:** Asosiy farq — **deployment va aloqa**. Modular monolith'da modullar bir process'da, in-process funksiya chaqiruvi orqali (tez, ishonchli, bitta transaction) gaplashadi va **birga** deploy bo'ladi. Microservices'da esa tarmoq orqali (sekin, ishonchsiz, distributed transaction) va **mustaqil** deploy bo'ladi. Modular monolith sizga toza chegaralarni **distributed narxsiz** beradi — shuning uchun u ajratishdan oldingi ideal bosqich.

### ❓ Strangler Fig qachon ishlamaydi?

**✅ Javob:** Agar eski tizimda **toza chegara ajratib bo'lmasa** (masalan, hamma narsa bitta ulkan shared DB'ga chirmashib ketgan bo'lsa), Strangler qiyinlashadi — chunki funksiyalarni birma-bir ajratish uchun ma'lumotni ham ajratish kerak. Bunday holda avval anti-corruption layer va DB dekomozitsiyasi kerak. Yana, agar tizim tez orada butunlay o'ladigan bo'lsa, bosqichma-bosqich migratsiyaga sarmoya isrof bo'lishi mumkin.

### ❓ Qaysi DB tanlashni arxitektura qanday belgilaydi?

**✅ Javob:** Access pattern (kirish shakli) hal qiladi. Kuchli consistency, transaction, murakkab join kerak bo'lsa — **SQL** (Postgres). Ulkan scale, oddiy key-based access, moslashuvchan schema kerak bo'lsa — **NoSQL** (DynamoDB/Cassandra). To'liq matnli qidiruv — **search index** (Elastic). Grafik munosabatlar — **graph DB**. Vaqt seriyasi — **time-series DB**. Microservices'da har service o'z DB turini tanlaydi (polyglot persistence) — bu database-per-service prinsipining afzalligi.

### ❓ "Ports and adapters"da port qayerda joylashadi — ichkaridami tashqaridami?

**✅ Javob:** **Port yadroga (domain) tegishli** — u yadro belgilaydigan interfeys (masalan `PaymentGateway` interfeysi). **Adapter esa tashqarida** — portning konkret implementatsiyasi (`StripePaymentAdapter`). Bu Dependency Inversion prinsipining amali: yadro abstraksiyaga (port) tayanadi, konkret texnologiyaga emas. Shuning uchun Stripe'dan boshqasiga o'tsangiz, faqat adapter yoziladi, yadro tegilmaydi.

### ❓ Bir tizimda nechta arxitektura uslubini aralashtirsa bo'ladi?

**✅ Javob:** Kerakligicha — ular bir-birini to'ldiradi. Tipik senior tizim: **Microservices** (umumiy tuzilish) + har service ichida **Hexagonal** (kod) + service'lar orasida **Event-driven** (aloqa) + og'ir o'qish service'da **CQRS** + mobil/web uchun **BFF** + migratsiya paytida **Strangler Fig**. Muhimi — har uslubni **asosli sabab** bilan qo'shish, "trend" uchun emas. Ortiqcha uslub aralashtirish (accidental complexity) ham anti-pattern.

### ❓ Intervyuda arxitektura tanlaganimni qanday asoslashim kerak?

**✅ Javob:** Har doim **requirement → constraint → tanlov → trade-off** zanjirini ovoz chiqarib bayon qiling. Masalan: "O'qish:yozish 100:1 va query'lar murakkab, shuning uchun CQRS bilan read model'ni Elastic'ga ajrataman; narxi — eventual consistency, lekin bu tizimda 1-2 soniya read lag qabul qilinadi." Intervyuer **javobingizni** emas, **fikrlash jarayoningizni** baholaydi. Muqobillarni ham eslang: "Monolith ham yetardi, lekin jamoa 40 kishi, shuning uchun..."

---

## Masalalar

> Yechimlar: [solutions/system-design/05-architecture-blueprints.md](../solutions/system-design/05-architecture-blueprints.md)

1. **Universal freymvork qo'llash.** "URL qisqartirgich (bit.ly)" tizimini 8 qadamli freymvork bo'yicha to'liq loyihalang: requirements, scale estimation (100M URL/oy), high-level, data model, API, deep dive (redirect yo'li), trade-off, evolve. Har qadam uchun 2-3 jumla yozing.

2. **Blueprint tanlash.** Quyidagi 4 ssenariy uchun eng mos arxitektura blueprint'ni tanlang va **asoslang**: (a) 3 kishilik startup MVP; (b) 50 muhandisli e-commerce platforma; (c) IoT sensorlardan real-time ma'lumot yig'ish; (d) bankdagi tranzaksiya tizimi (to'liq audit majburiy).

3. **CQRS + Event Sourcing.** Buyurtma boshqaruv tizimi uchun CQRS va Event Sourcing'ni birga qo'llash diagrammasini chizing (ASCII). Command tomoni, event store, read model sinxronizatsiyasi va bitta misol event ketma-ketligini ko'rsating.

4. **Migratsiya rejasi.** 8 yillik legacy monolith (bitta ulkan PostgreSQL, 40 ta jadval, hammasi bog'langan) ni microservices'ga ko'chirish uchun Strangler Fig asosida bosqichma-bosqich reja tuzing. Birinchi ajratiladigan service qaysi bo'lishi kerak va nima uchun?

5. **Building block xaritasi.** "Video streaming platforma (YouTube kabi)" uchun qaysi building block'lar kerak bo'lishini ro'yxatlang va har biri qaysi muammoni hal qilishini bir jumlada yozing. Kamida 7 ta block.

6. **Trade-off tahlili.** Bitta jamoa (10 dev) "biz zamonaviy bo'lishimiz kerak" deb 15 ta microservice qurmoqchi. Ularga qarshi 5 ta texnik argument va tavsiya etilgan muqobil arxitekturani asoslang.

7. **Hexagonal dizayn.** "To'lov ishlovi" service'ini Hexagonal arxitekturada loyihalang: domain core, portlar (interfeyslar) ro'yxati, va adapterlar (Stripe, PostgreSQL, Kafka) ni ko'rsating. Nega bu tuzilish to'lov provayderini almashtirishni osonlashtiradi?

8. **Kombinatsiya dizayni.** "Ovqat yetkazish ilovasi (Uber Eats kabi)" uchun kamida 4 ta arxitektura blueprint'ni birga ishlatadigan yuqori darajali dizayn taklif qiling. Har blueprint qayerda va nima uchun ishlatilganini asoslang.

---

← [System Design bo'limiga qaytish](./README.md)
