# Database Design

Database design — bu ma'lumotlarni qanday saqlash, tashkil qilish va bog'lash to'g'risidagi qarorlar majmuasi. Yaxshi dizayn ilovani yillar davomida tez, ishonchli va kengaytiriladigan qiladi; yomon dizayn esa har bir yangi feature uchun azob beradi. Bu hujjat requirements'dan ER model va schema'ga o'tish jarayonini, normalizatsiya va denormalizatsiyani, key strategiyalarini, partitioning, sharding, replication, CAP/PACELC va zero-downtime migration'ni senior darajada yoritadi.

## Mundarija

- [Dizayn jarayoni: requirements → ER → schema](#dizayn-jarayoni-requirements--er--schema)
- [ER modeling: entity, attribute, relationship](#er-modeling-entity-attribute-relationship)
- [Cardinality: 1:1, 1:N, M:N va junction table](#cardinality-11-1n-mn-va-junction-table)
- [Normalizatsiya: 1NF, 2NF, 3NF, BCNF](#normalizatsiya-1nf-2nf-3nf-bcnf)
- [Denormalizatsiya: qachon va nega](#denormalizatsiya-qachon-va-nega)
- [Primary key strategiyalari](#primary-key-strategiyalari)
- [Many-to-many modeling](#many-to-many-modeling)
- [Soft delete va audit column](#soft-delete-va-audit-column)
- [Data type tanlash](#data-type-tanlash)
- [Dizaynda indexing strategiyasi](#dizaynda-indexing-strategiyasi)
- [Partitioning](#partitioning)
- [Sharding](#sharding)
- [Replication](#replication)
- [CAP va PACELC theorem](#cap-va-pacelc-theorem)
- [Schema migration: zero-downtime](#schema-migration-zero-downtime)
- [Polyglot persistence](#polyglot-persistence)
- [Ishlangan misol: e-commerce schema](#ishlangan-misol-e-commerce-schema)
- [Savol-javoblar](#savol-javoblar)
- [Masalalar](#masalalar)

---

## Dizayn jarayoni: requirements → ER → schema

**💡 Tushuncha:** Database dizayni darhol `CREATE TABLE` yozishdan boshlanmaydi. U uchta bosqichdan o'tadi: **conceptual** (requirements'ni tushunish, entity'larni aniqlash) → **logical** (ER model, normalizatsiya, atributlar va bog'lanishlar) → **physical** (haqiqiy schema: data type'lar, index'lar, partitioning, aniq DBMS).

Bosqichlar:

| Bosqich | Nima qilinadi | Natija |
|---------|---------------|--------|
| Requirements gathering | Biznes qoidalari, query pattern'lar, hajm/o'sish bashorati | hujjat, foydalanuvchi stories |
| Conceptual design | Asosiy entity'lar va ular orasidagi bog'lanishlar | yuqori darajadagi ER diagramma |
| Logical design | Atributlar, key'lar, cardinality, normalizatsiya | normallashtirilgan relational model |
| Physical design | Data type, index, partition, denormalizatsiya | DBMS uchun real schema (DDL) |

**⚠️ Ehtiyot bo'l:** Eng katta xato — query pattern'larni so'ramasdan dizayn qilish. "Bu ma'lumotni qanday o'qiymiz?" degan savol "Bu ma'lumotni qanday saqlaymiz?" degan savoldan muhimroq. Read-heavy ilova bilan write-heavy ilova tubdan farqli schema talab qiladi.

---

## ER modeling: entity, attribute, relationship

**💡 Tushuncha:** ER (Entity-Relationship) model real dunyodagi narsalarni va ular orasidagi munosabatlarni tasvirlaydi.

- **Entity** — mustaqil mavjudlikka ega narsa (User, Order, Product). Jadvalga aylanadi.
- **Attribute** — entity'ning xususiyati (User.email, Order.total). Ustunga aylanadi.
- **Relationship** — entity'lar orasidagi bog'lanish (User *places* Order). Foreign key yoki junction table'ga aylanadi.

Atribut turlari:

| Tur | Tavsif | Misol |
|-----|--------|-------|
| Simple | bo'linmaydi | `age` |
| Composite | qism-qismlardan iborat | `full_name` → `first_name` + `last_name` |
| Derived | boshqasidan hisoblanadi, saqlanmaydi | `age` (birth_date'dan) |
| Multivalued | bir nechta qiymat | telefon raqamlar → alohida jadval |

**⚠️ Ehtiyot bo'l:** Multivalued attribute'ni bitta ustunda vergul bilan saqlash (`phones = "111,222,333"`) — bu 1NF buzilishi. Har doim alohida jadval (`user_phones`) ishlating.

---

## Cardinality: 1:1, 1:N, M:N va junction table

**💡 Tushuncha:** Cardinality bir entity'ning nechta boshqa entity bilan bog'lanishini bildiradi.

**1:1 (one-to-one)** — bir User'ning bitta Profile'i bor. Odatda bitta jadvalga birlashtiriladi; ammo ajratish foydali bo'lishi mumkin: kamdan-kam ishlatiladigan/og'ir ustunlar (large blob), xavfsizlik (sensitive ma'lumotni alohida), yoki optional 1:1.

```sql
CREATE TABLE users (
    id         BIGINT PRIMARY KEY,
    email      VARCHAR(255) NOT NULL UNIQUE
);

CREATE TABLE user_profiles (
    user_id    BIGINT PRIMARY KEY REFERENCES users(id),
    bio        TEXT,
    avatar_url VARCHAR(500)
);
```

**1:N (one-to-many)** — bir User'ning ko'p Order'i bor. FK "ko'p" tarafda turadi (`orders.user_id`).

```sql
CREATE TABLE orders (
    id      BIGINT PRIMARY KEY,
    user_id BIGINT NOT NULL REFERENCES users(id),
    total   NUMERIC(12,2) NOT NULL
);
```

**M:N (many-to-many)** — bir Student ko'p Course'ga yoziladi, bir Course'da ko'p Student bor. Relational model'da to'g'ridan-to'g'ri ifodalab bo'lmaydi — **junction table** (associative/bridge table) kerak.

```sql
CREATE TABLE enrollments (
    student_id BIGINT NOT NULL REFERENCES students(id),
    course_id  BIGINT NOT NULL REFERENCES courses(id),
    enrolled_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    grade      CHAR(1),
    PRIMARY KEY (student_id, course_id)
);
```

**⚠️ Ehtiyot bo'l:** Junction table ko'pincha o'z atributlariga ega bo'ladi (`enrolled_at`, `grade`). Bu uni shunchaki "texnik jadval" emas, balki mustaqil entity'ga aylantiradi — bu normal holat.

---

## Normalizatsiya: 1NF, 2NF, 3NF, BCNF

**💡 Tushuncha:** Normalizatsiya — ma'lumotni redundancy (takrorlanish) va anomaliyalardan (insert/update/delete anomaly) qutqarish uchun jadvallarni bo'lish jarayoni. Har bir normal forma oldingisini o'z ichiga oladi.

### 1NF (First Normal Form)

**Muammo:** atomic bo'lmagan qiymatlar — bir katakda bir nechta qiymat yoki takrorlanuvchi guruhlar.

```
-- BUZILGAN: phones ustunida bir nechta qiymat
| user_id | name  | phones            |
| 1       | Ali   | 111, 222, 333     |
```

**Yechim:** har bir katak atomik bo'lsin, takrorlanuvchi guruhni alohida jadvalga chiqaring.

```
users: | user_id | name |        user_phones: | user_id | phone |
       | 1       | Ali  |                     | 1       | 111   |
                                               | 1       | 222   |
```

### 2NF (Second Normal Form)

**Muammo:** 1NF + composite primary key bo'lganda, non-key atribut key'ning **bir qismiga** bog'liq (partial dependency).

```
-- PK = (order_id, product_id). product_name faqat product_id'ga bog'liq.
| order_id | product_id | product_name | quantity |
```
Bu yerda `product_name` har bir takrorlangan product uchun qayta saqlanadi → update anomaly.

**Yechim:** partial dependency'ni alohida jadvalga chiqaring.

```
order_items: | order_id | product_id | quantity |
products:    | product_id | product_name |
```

### 3NF (Third Normal Form)

**Muammo:** 2NF + transitive dependency — non-key atribut boshqa non-key atribut orqali key'ga bog'liq.

```
-- PK = employee_id. department_name department_id orqali bog'liq.
| employee_id | name | department_id | department_name |
```
`department_name` har bir xodim uchun takrorlanadi.

**Yechim:** transitive dependency'ni ajrating.

```
employees:   | employee_id | name | department_id |
departments: | department_id | department_name |
```

> **3NF qoidasi (xotirjam shakl):** "Har bir non-key atribut **key'ga, butun key'ga va faqat key'ga** bog'liq bo'lishi kerak" — *the key, the whole key, and nothing but the key*.

### BCNF (Boyce-Codd Normal Form)

**Muammo:** 3NF'ning kuchli versiyasi. Buziladi, qachonki determinant (bog'lovchi tomon) candidate key bo'lmasa. Odatda jadvalda bir nechta overlapping candidate key bo'lganda yuzaga keladi.

```
-- Qoida: har bir professor bitta subject o'qitadi; har bir subject'ni ko'p professor o'qitishi mumkin.
-- (student, subject) -> professor   VA   professor -> subject
| student | subject  | professor |
```
`professor → subject` da `professor` candidate key emas → BCNF buzilgan, anomaliya bor.

**Yechim:** har bir determinant candidate key bo'ladigan tarzda jadvalni bo'ling.

```
teaches:  | professor | subject |      (professor PK)
enrolled: | student | professor |
```

**Normal formalar xulosasi:**

| Forma | Bartaraf etadi |
|-------|----------------|
| 1NF | atomik bo'lmagan qiymatlar, takrorlanuvchi guruhlar |
| 2NF | partial dependency (composite key qismiga) |
| 3NF | transitive dependency (non-key → non-key) |
| BCNF | determinant candidate key bo'lmagan holatlar |

**⚠️ Ehtiyot bo'l:** Amaliyotda ko'pchilik OLTP ilovalar uchun **3NF yetarli**. BCNF kamdan-kam zarur, ko'pincha 3NF tabiiy ravishda BCNF'ni ham qanoatlantiradi.

---

## Denormalizatsiya: qachon va nega

**💡 Tushuncha:** Denormalizatsiya — ataylab redundancy qo'shish (normalizatsiyani qisman orqaga qaytarish), read performance'ni yaxshilash uchun. Bu trade-off: tezroq read, ammo murakkabroq write va consistency riski.

Qachon denormalizatsiya qilinadi:

- **Read >> Write**: bir necha join'lar har bir read'da takrorlanadi va bottleneck bo'ladi.
- **Aggregate'lar**: `order_count`, `total_spent` kabilarni har safar hisoblamasdan saqlash.
- **Reporting/analytics**: OLAP/data warehouse'da star schema ataylab denormallashtirilgan.
- **Hot path**: kritik query'ni join'siz qilish kerak bo'lganda.

Texnikalar:

| Texnika | Tavsif |
|---------|--------|
| Precomputed column | `users.order_count` ni order yaratilganda yangilash |
| Duplicated column | `order_items.product_name` ni products'dan ko'chirish (snapshot) |
| Materialized view | murakkab query natijasini fizik saqlash |
| Summary table | kunlik/oylik aggregat jadvallar |

**⚠️ Ehtiyot bo'l:** Denormalizatsiya consistency mas'uliyatini ilova/trigger zimmasiga yuklaydi. Agar `order_count` ni yangilashni unutib qo'ysangiz — ma'lumot noto'g'ri bo'ladi. Avval normallashtiring, keyin o'lchovga asoslanib (profiling) zarurat bo'lsa denormallashtiring. Premature denormalizatsiya — premature optimization'ning bir turi.

---

## Primary key strategiyalari

**💡 Tushuncha:** Primary key har bir row'ni noyob aniqlaydi. Tanlov ikki o'qda: natural vs surrogate, hamda surrogate uchun auto-increment vs UUID vs ULID.

**Natural vs Surrogate:**

- **Natural key** — biznesdagi mavjud noyob qiymat (email, ISBN, SSN). Kamchilik: o'zgarishi mumkin (email almashtirish), katta bo'lishi mumkin, har doim mavjud emas.
- **Surrogate key** — biznes ma'nosi yo'q sun'iy ID (auto-increment, UUID). Afzallik: barqaror, ixcham, biznes o'zgarishidan mustaqil. Ko'pchilik amaliyotda surrogate afzal ko'riladi.

**Surrogate variantlari trade-off:**

| Tur | Hajm | Tartiblangan? | Index locality | Distributed-safe | Kamchilik |
|-----|------|---------------|----------------|------------------|-----------|
| Auto-increment (BIGINT) | 8 bayt | ha | a'lo (ketma-ket) | yo'q (markaziy) | guessable, sharding'da to'qnashuv |
| UUIDv4 (random) | 16 bayt | yo'q | yomon (random insert → page split) | ha | katta, index bloat |
| UUIDv7 / ULID | 16 bayt | ha (time-ordered) | yaxshi | ha | yangiroq, qo'llab-quvvatlash kerak |

**💡 Tushuncha (index locality):** Auto-increment va ULID/UUIDv7 monotonik o'sadi, shuning uchun yangi row'lar B-tree'ning oxiriga ketma-ket qo'shiladi — page split kam, cache yaxshi. UUIDv4 random bo'lgani uchun har bir insert tasodifiy joyga tushadi — bu page split, fragmentation va sekin write keltirib chiqaradi (ayniqsa InnoDB'da clustered primary key bilan).

```sql
-- Auto-increment (PostgreSQL)
id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,

-- UUID
id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
```

**⚠️ Ehtiyot bo'l:** Distributed/sharded tizimda auto-increment markaziy nuqta yaratadi va ID to'qnashuvi muammosini keltiradi. Bunday holatda ULID yoki UUIDv7 (time-ordered, index-friendly va distributed-safe) eng yaxshi tanlov. Sof UUIDv4'ni katta hajmli, write-heavy clustered jadvalda primary key sifatida ishlatishdan ehtiyot bo'ling.

---

## Many-to-many modeling

**💡 Tushuncha:** M:N munosabati har doim junction table orqali modellashtiriladi (yuqorida ko'rdik). Senior darajada muhim nuanslar:

- **Composite PK vs surrogate PK**: junction table'da `PRIMARY KEY (a_id, b_id)` tabiiy va duplicate'ni oldini oladi. Ammo junction table'ning o'zi boshqa jadvalga FK bo'lib bog'lansa (masalan `enrollment_payments` → `enrollments`), surrogate `id` qulayroq.
- **Atributli munosabat**: `enrolled_at`, `role`, `quantity` kabi munosabat atributlari junction table'da yashaydi.
- **Index'lar**: ikkala yo'nalishda ham so'rov bo'lsa, har ikki FK uchun index kerak. `PRIMARY KEY (a_id, b_id)` faqat `a_id` bo'yicha qidirishni qoplaydi (leftmost-prefix); `b_id` bo'yicha qidirish uchun alohida index qo'shing.

```sql
CREATE TABLE enrollments (
    student_id BIGINT NOT NULL REFERENCES students(id),
    course_id  BIGINT NOT NULL REFERENCES courses(id),
    PRIMARY KEY (student_id, course_id)
);
-- course bo'yicha "bu kursda kim bor?" so'rovi uchun:
CREATE INDEX idx_enrollments_course ON enrollments(course_id);
```

---

## Soft delete va audit column

**💡 Tushuncha:** **Soft delete** — row'ni jismonan o'chirmasdan, `deleted_at` ustunini belgilash. Ma'lumotni tiklash, audit va referential butunlikni saqlash uchun.

```sql
ALTER TABLE users ADD COLUMN deleted_at TIMESTAMPTZ;
-- "tirik" foydalanuvchilar:
SELECT * FROM users WHERE deleted_at IS NULL;
-- partial unique index — faqat tirik row'lar uchun email unique:
CREATE UNIQUE INDEX uq_users_email_active
  ON users(email) WHERE deleted_at IS NULL;
```

**Audit column'lar** — har bir jadvalga standart kuzatuv ustunlari:

```sql
created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
created_by BIGINT REFERENCES users(id),
updated_by BIGINT REFERENCES users(id)
```

**⚠️ Ehtiyot bo'l:** Soft delete'ning xavfli tomonlari:
- Har bir query'ga `WHERE deleted_at IS NULL` qo'shishni unutmaslik kerak — aks holda o'chirilgan ma'lumot "tiriladi".
- Unique constraint'lar buziladi: o'chirilgan va yangi row bir xil email bilan to'qnashadi → partial unique index ishlating.
- Jadval o'sib boraveradi → eski soft-deleted row'larni arxivlash/o'chirish kerak bo'lishi mumkin.

To'liq audit kerak bo'lsa (kim, qachon, nimani o'zgartirdi), alohida `audit_log` / history jadval yoki temporal table ishlatish soft delete'dan kuchliroq yechim.

---

## Data type tanlash

**💡 Tushuncha:** To'g'ri data type — saqlash hajmi, performance va to'g'rilik (correctness) masalasi. Eng tor mos keladigan turni tanlang.

| Maqsad | To'g'ri tanlov | Xato |
|--------|----------------|------|
| Pul | `NUMERIC(12,2)` / `DECIMAL` | `FLOAT`/`DOUBLE` (yaxlitlash xatosi) |
| Vaqt | `TIMESTAMPTZ` (UTC) | `TIMESTAMP` timezone'siz, yoki `VARCHAR` |
| Boolean | `BOOLEAN` | `CHAR(1)`, `INT` |
| Enum/holat | `VARCHAR` + CHECK yoki native ENUM / lookup table | erkin `TEXT` |
| Identifikator | `BIGINT` / `UUID` | `INT` (overflow), `VARCHAR` |
| Ixtiyoriy strukturali | `JSONB` (Postgres) | normallashmagan TEXT blob |
| Qisqa matn | `VARCHAR(n)` mantiqiy limit bilan | har doim `TEXT` |

**⚠️ Ehtiyot bo'l:**
- **Pulni hech qachon `FLOAT`'da saqlamang** — `0.1 + 0.2 != 0.3`. Har doim `NUMERIC`/`DECIMAL`.
- Vaqtni har doim **UTC (`TIMESTAMPTZ`)** da saqlang, ko'rsatishda local timezone'ga aylantiring.
- `INT` (2.1 milliard limit) ID uchun xavfli — ko'p ilovalar buni hisobga olmay overflow'ga uchragan. `BIGINT` ishlating.
- `VARCHAR(255)` "default" deb o'ylamang — limitni biznes mantiqdan oling.

---

## Dizaynda indexing strategiyasi

**💡 Tushuncha:** Index'lar dizaynning ajralmas qismi — ularni schema bilan birga, query pattern'larga qarab rejalashtirish kerak.

Dizayn paytida index qoidalari:

- **Har bir foreign key'ga index** — join va `ON DELETE` tekshiruvlari uchun (ko'p DBMS buni avtomatik qilmaydi).
- **WHERE/ORDER BY/GROUP BY** ustunlariga index — eng tez-tez ishlatiladigan filtrlarga.
- **Composite index tartibi**: eng selective yoki eng tez-tez teng-qiymat (equality) bilan filtrlanadigan ustun birinchi (leftmost-prefix qoidasi).
- **Covering index**: query faqat index'dagi ustunlardan o'qisa, jadvalga murojaat shart emas.
- **Partial index**: faqat row'lar qism to'plamiga (`WHERE deleted_at IS NULL`).
- **Unique index** — biznes qoidasini (unique email) majburlash uchun.

**⚠️ Ehtiyot bo'l:** Index bepul emas — har bir write'ni sekinlashtiradi va joy egallaydi. Write-heavy jadvalda ortiqcha index qo'shmang. "Har ehtimolga qarshi" index qo'ymang; real query pattern'lar va `EXPLAIN` natijasiga asoslaning.

---

## Partitioning

**💡 Tushuncha:** Partitioning — bitta katta jadvalni bir DBMS instance ichida fizik bo'laklarga (partition) bo'lish. Mantiqan u bitta jadval bo'lib qoladi, ammo har bir partition alohida saqlanadi. Maqsad: katta jadvalda query (partition pruning), maintenance va eski ma'lumotni o'chirishni tezlashtirish.

Partitioning turlari:

| Tur | Partition key bo'yicha | Misol |
|-----|------------------------|-------|
| Range | diapazon | `created_at` oy bo'yicha (events_2026_01, ...) |
| Hash | hash(key) % N | `user_id` ni N partition'ga teng taqsimlash |
| List | aniq qiymatlar ro'yxati | `region IN ('EU')`, `('US')` |
| Geo | geografik | mintaqa bo'yicha |

```sql
-- PostgreSQL range partitioning misoli
CREATE TABLE events (
    id BIGINT, created_at TIMESTAMPTZ NOT NULL, payload JSONB
) PARTITION BY RANGE (created_at);

CREATE TABLE events_2026_01 PARTITION OF events
    FOR VALUES FROM ('2026-01-01') TO ('2026-02-01');
```

**Foydasi:**
- **Partition pruning**: `WHERE created_at >= '2026-01'` faqat tegishli partition'larni skanlaydi.
- **Tez o'chirish**: eski oyni o'chirish = `DROP TABLE events_2025_01` (DELETE'dan ancha tez).
- **Maintenance**: VACUUM/index rebuild partition bo'yicha.

**⚠️ Ehtiyot bo'l:** Partition key'ni noto'g'ri tanlash hammasini buzadi. Agar query'lar partition key bo'yicha filtrlamasa, pruning ishlamaydi va barcha partition'lar skanlanadi (sekinroq). Partition key — eng tez-tez filtr ustuni bo'lishi kerak.

---

## Sharding

**💡 Tushuncha:** Sharding — ma'lumotni **bir nechta alohida DBMS instance (node)** ga gorizontal taqsimlash. Partitioning bitta server ichidagi bo'linish bo'lsa, sharding serverlar orasidagi bo'linish — horizontal scaling'ning asosiy usuli.

**Shard key** — qaysi shard'ga borishni belgilovchi ustun (masalan `user_id`). Strategiyalar:

| Strategiya | Ishlash | Plus | Minus |
|-----------|---------|------|-------|
| Range | key diapazoni bo'yicha | range query oson | hotspot (ketma-ket key bir shard'ga) |
| Hash | hash(key) bo'yicha | teng taqsimlash | range query qiyin, resharding og'ir |
| Geo/Directory | lookup jadval orqali | moslashuvchan | lookup qatlam murakkabligi |

**⚠️ Ehtiyot bo'l:** Sharding eng kuchli, ammo eng murakkab vosita. U bilan birga kelgan og'riqlar:
- **Cross-shard query**: bir nechta shard'dan ma'lumot yig'ish (JOIN, aggregation) qiyin va sekin.
- **Cross-shard transaction**: distributed transaction (2PC) — sekin va murakkab.
- **Resharding**: shard sonini o'zgartirish ma'lumotni ko'chirishni talab qiladi (consistent hashing bu og'riqni kamaytiradi).
- **Hotspot**: yomon shard key (masalan `country` — barcha trafik bitta shard'ga).

Sharding'ni faqat oddiyroq usullar (read replica, vertical scaling, partitioning, cache) tugaganda qo'llang. Shard key tanlovi qaytarib bo'lmaydigan qaror.

---

## Replication

**💡 Tushuncha:** Replication — bir xil ma'lumotni bir nechta node'da saqlash. Maqsad: high availability (bir node o'lsa, boshqasi xizmat qiladi), read scaling (read'larni replica'larga taqsimlash), va geo-yaqinlik.

Topologiyalar:

| Topologiya | Tavsif | Plus | Minus |
|-----------|--------|------|-------|
| Leader-Follower (master-replica) | bitta yozuvchi leader, ko'p faqat-o'quvchi follower | oddiy, read scaling | replication lag, failover kerak |
| Multi-Leader | bir nechta yozuvchi node | geo-write, HA | write conflict, conflict resolution |
| Quorum (leaderless) | W + R > N qoidasi bilan o'qish/yozish | yuqori availability | tuning murakkab (Dynamo-style) |

**💡 Tushuncha (replication lag):** Leader-follower'da follower leader'dan biroz orqada qoladi. "Read your own writes" muammosi: foydalanuvchi yozadi (leader'ga), darhol o'qiydi (follower'dan) — hali yangilanmagan ma'lumotni ko'radi. Yechimlar: muhim read'larni leader'dan o'qish, yoki kechikishni nazorat qilish.

**Sync vs Async:**
- **Synchronous**: leader follower tasdiqlagunча kutadi — ma'lumot yo'qolmaydi, ammo sekinroq va follower o'lsa bloklanadi.
- **Asynchronous**: leader kutmaydi — tez, ammo leader crash bo'lsa, replicate bo'lmagan write'lar yo'qoladi.

**⚠️ Ehtiyot bo'l:** Read replica'lar consistency'ni avtomatik kafolatlamaydi. Eventual consistency'ga tayyor bo'lmagan funksiya (masalan, to'lovdan keyin balansni darhol ko'rsatish) replica'dan o'qisa, noto'g'ri ma'lumot chiqaradi.

---

## CAP va PACELC theorem

**💡 Tushuncha:** **CAP theorem** — distributed tizimda network **P**artition (tarmoq bo'linishi) yuz berganda, **C**onsistency va **A**vailability orasidan faqat bittasini tanlash mumkin.

- **C (Consistency)**: har bir read eng so'nggi yozuvni qaytaradi (yoki xato).
- **A (Availability)**: har bir so'rov javob oladi (eng so'nggi bo'lmasa ham).
- **P (Partition tolerance)**: tarmoq xabarlari yo'qolsa ham ishlaydi.

Tarmoq partition'i real tizimda muqarrar, shuning uchun P majburiy. Demak haqiqiy tanlov: **partition paytida CP yoki AP**.

| Tanlov | Mazmun | Misol |
|--------|--------|-------|
| CP | partition'da consistency saqlanadi, availability qurbon | MongoDB (default), HBase, ZooKeeper |
| AP | partition'da javob beraveradi, consistency qurbon | Cassandra, DynamoDB, Riak |

**💡 Tushuncha (PACELC):** CAP faqat partition holatini ko'radi. **PACELC** to'liqroq: *agar Partition bo'lsa (P), A yoki C orasidan tanla; Else (E), normal holatda Latency (L) yoki Consistency (C) orasidan tanla.* Ya'ni partition bo'lmaganda ham consistency va latency orasida trade-off bor.

- Dynamo/Cassandra: **PA/EL** (partition'da availability, normalda low latency).
- Spanner/Postgres (sync): **PC/EC** (har doim consistency afzal).

**⚠️ Ehtiyot bo'l:** "CAP'da 3 dan 2 tasini tanla" degan oddiy ta'rif chalg'ituvchi — P'ni "tanlamaslik" mumkin emas. To'g'ri savol: tizimingiz partition paytida consistent qoladimi yoki available qoladimi?

---

## Schema migration: zero-downtime

**💡 Tushuncha:** Schema migration — ishlayotgan production database'da schema'ni o'zgartirish. Maqsad: eski va yangi kod versiyasi bir vaqtda ishlayotganda ham buzilmasligi (zero-downtime).

**Xavfli operatsiyalar (lock/downtime keltiradi):**
- Ustunni `NOT NULL` default bilan to'la jadvalga qo'shish (eski DBMS'da to'liq rewrite).
- Ustunni o'chirish/qayta nomlash (eski kod hali ishlatayotgan bo'lishi mumkin).
- Type o'zgartirish, katta jadvalga index `CREATE INDEX` (concurrent bo'lmasa lock).

**Xavfsiz pattern — expand/contract (uch bosqichli):**

1. **Expand**: yangi ustun/jadvalni qo'shing (nullable, default'siz, backward-compatible). Eski kod ham, yangi kod ham ishlaydi.
2. **Migrate + dual-write**: kod yangi ustunga ham yozadi; eski ma'lumotni batch'lab backfill qiling.
3. **Contract**: barcha kod yangiga o'tgach, eski ustunni o'chiring.

```sql
-- Xavfsiz: nullable qo'shish, lock yo'q
ALTER TABLE users ADD COLUMN phone VARCHAR(20);
-- Postgres'da bloklamasdan index:
CREATE INDEX CONCURRENTLY idx_users_phone ON users(phone);
```

**⚠️ Ehtiyot bo'l:**
- Rename'ni hech qachon bitta qadamda qilmang — "add new + dual-write + drop old" qiling.
- Backfill'ni bitta katta UPDATE bilan emas, batch'lab qiling (lock va replication lag'ni oldini olish).
- Migration har doim **backward-compatible** bo'lishi kerak: deploy paytida eski kod hali ishlaydi.

---

## Polyglot persistence

**💡 Tushuncha:** Polyglot persistence — bitta tizimda har bir vazifa uchun eng mos ma'lumot saqlash texnologiyasini ishlatish. "One database fits all" o'rniga to'g'ri vositani tanlash.

| Ehtiyoj | Mos texnologiya |
|---------|-----------------|
| Transactional, relational ma'lumot | PostgreSQL / MySQL (RDBMS) |
| Cache, session, rate limit | Redis (in-memory KV) |
| Full-text search | Elasticsearch / OpenSearch |
| Graph (do'stlar, tavsiya) | Neo4j |
| Time-series (metrics) | TimescaleDB / InfluxDB |
| Hujjat (moslashuvchan schema) | MongoDB |
| Analytics / OLAP | ClickHouse / BigQuery |
| Blob/fayl | S3 / object storage |

**⚠️ Ehtiyot bo'l:** Har bir yangi datastore operatsion narx (monitoring, backup, ekspertiza, consistency murakkabligi) qo'shadi. Bir nechta store orasida ma'lumotni sinxronlash (dual-write, CDC, eventual consistency) qiyin masala. Bitta yaxshi tanlangan RDBMS (JSONB, full-text, range type bilan) ko'p ehtiyojni qoplaydi — polyglot'ga faqat real zarurat bo'lganda o'ting.

---

## Ishlangan misol: e-commerce schema

**💡 Tushuncha:** Requirements'dan to'liq schema'gacha. Talablar: foydalanuvchilar mahsulot ko'radi, savatga qo'shadi, buyurtma beradi; buyurtmada bir nechta mahsulot bo'ladi; mahsulot bir nechta kategoriyaga tegishli (M:N); narx vaqt o'tishi bilan o'zgaradi, lekin buyurtmadagi narx o'sha paytdagi narx (snapshot) bo'lishi kerak.

```sql
-- Foydalanuvchilar
CREATE TABLE users (
    id         BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    email      VARCHAR(255) NOT NULL,
    full_name  VARCHAR(200) NOT NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    deleted_at TIMESTAMPTZ
);
CREATE UNIQUE INDEX uq_users_email_active
    ON users(email) WHERE deleted_at IS NULL;

-- Mahsulotlar
CREATE TABLE products (
    id          BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    sku         VARCHAR(64) NOT NULL UNIQUE,   -- natural business key
    name        VARCHAR(300) NOT NULL,
    price       NUMERIC(12,2) NOT NULL CHECK (price >= 0),
    stock       INT NOT NULL DEFAULT 0 CHECK (stock >= 0),
    created_at  TIMESTAMPTZ NOT NULL DEFAULT now()
);

-- Kategoriyalar va M:N junction
CREATE TABLE categories (
    id   BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    name VARCHAR(120) NOT NULL UNIQUE
);
CREATE TABLE product_categories (
    product_id  BIGINT NOT NULL REFERENCES products(id),
    category_id BIGINT NOT NULL REFERENCES categories(id),
    PRIMARY KEY (product_id, category_id)
);
CREATE INDEX idx_pc_category ON product_categories(category_id);

-- Buyurtmalar (1:N user -> orders)
CREATE TABLE orders (
    id          BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    user_id     BIGINT NOT NULL REFERENCES users(id),
    status      VARCHAR(20) NOT NULL DEFAULT 'pending'
                CHECK (status IN ('pending','paid','shipped','cancelled')),
    total       NUMERIC(12,2) NOT NULL DEFAULT 0,
    created_at  TIMESTAMPTZ NOT NULL DEFAULT now()
);
CREATE INDEX idx_orders_user ON orders(user_id);
CREATE INDEX idx_orders_created ON orders(created_at);

-- Buyurtma elementlari (junction + atributli; narx snapshot)
CREATE TABLE order_items (
    order_id    BIGINT NOT NULL REFERENCES orders(id) ON DELETE CASCADE,
    product_id  BIGINT NOT NULL REFERENCES products(id),
    quantity    INT NOT NULL CHECK (quantity > 0),
    unit_price  NUMERIC(12,2) NOT NULL,   -- snapshot: o'sha paytdagi narx
    PRIMARY KEY (order_id, product_id)
);
```

Dizayn qarorlari:
- **`unit_price` snapshot**: `products.price` o'zgarsa ham, eski buyurtma narxi saqlanadi (3NF'dan ataylab chetga chiqish — bu derived emas, tarixiy fakt).
- **`sku` natural key + `id` surrogate PK**: ikkalasi ham bor; FK'lar barqaror `id`'ga bog'lanadi.
- **Soft delete** faqat `users`'da (audit kerak); `products` uchun `is_active` flag etarli bo'lardi.
- **Index'lar** FK'lar va tez-tez filtrlanadigan ustunlarga (`orders.created_at`).
- Hajm o'sganda `orders`/`order_items`'ni `created_at` bo'yicha **range partition** qilish mumkin; juda katta o'lchovda `user_id` bo'yicha **shard**.

---

## Savol-javoblar

### ❓ Conceptual, logical va physical design farqi nimada?

**✅ Javob:** **Conceptual** — biznes darajasidagi entity'lar va bog'lanishlar (DBMS'dan mustaqil). **Logical** — atributlar, key'lar, cardinality, normalizatsiya bilan to'liq relational model (hali ham aniq DBMS'siz). **Physical** — real DBMS uchun schema: data type'lar, index'lar, partitioning, denormalizatsiya. Avvalgisi nimani saqlash, oxirgisi qanday saqlash haqida.

### ❓ Normalizatsiya nima va u qanday anomaliyalarni bartaraf etadi?

**✅ Javob:** Normalizatsiya — redundancy'ni kamaytirish uchun jadvallarni dependency'larga ko'ra bo'lish. U uchta anomaliyani bartaraf etadi: **insert anomaly** (boshqa ma'lumotsiz row qo'sha olmaslik), **update anomaly** (takrorlangan qiymatni bir joyda yangilab, boshqasini unutish), **delete anomaly** (row o'chirilganda kerakli ma'lumot ham yo'qolishi). 1NF atomiklik, 2NF partial dependency, 3NF transitive dependency, BCNF determinant muammosini hal qiladi.

### ❓ 3NF'ni bir jumlada tushuntiring.

**✅ Javob:** "Har bir non-key atribut **key'ga, butun key'ga va faqat key'ga** bog'liq bo'lishi kerak." Ya'ni partial dependency (2NF) ham, transitive dependency (3NF) ham bo'lmasin.

### ❓ 3NF va BCNF farqi nimada? Qachon 3NF yetarli emas?

**✅ Javob:** 3NF non-key atributning transitive dependency'siga e'tibor beradi. BCNF kuchliroq: **har bir functional dependency'ning chap tomoni (determinant) candidate key bo'lishi** kerak. 3NF buzilmagan, ammo BCNF buzilgan holat — jadvalda bir nechta overlapping candidate key bo'lib, ulardan biri ikkinchisining qismini aniqlaganda yuzaga keladi (klassik student-subject-professor misoli). Amaliyotda kamdan-kam uchraydi.

### ❓ Qachon denormallashtirasiz?

**✅ Javob:** Faqat profiling/o'lchov real bottleneck ko'rsatganda. Tipik holatlar: read >> write, har bir read'da takrorlanuvchi qimmat join, tez-tez kerak bo'ladigan aggregate'lar (`order_count`), reporting/analytics. Trade-off — tezroq read, ammo consistency'ni ilova/trigger zimmasiga olish va redundancy. Qoida: avval normallashtir, keyin o'lchovga asoslanib denormallashtir.

### ❓ Surrogate va natural key — qaysi birini tanlaysiz?

**✅ Javob:** Ko'p hollarda **surrogate** (sun'iy ID), chunki u barqaror va biznes o'zgarishidan mustaqil — natural key'lar (email, telefon) o'zgarishi mumkin va ularni FK qilib bog'lash zanjirli yangilanishlarga olib keladi. Natural key'ni unique constraint sifatida saqlash mumkin (`sku UNIQUE`), lekin PK va FK sifatida surrogate ishlatiladi.

### ❓ UUID vs auto-increment vs ULID — index locality nuqtai nazaridan farqi?

**✅ Javob:** Auto-increment va ULID/UUIDv7 monotonik o'sadi, shuning uchun yangi row'lar B-tree'ning oxiriga ketma-ket joylashadi — page split kam, write tez, index ixcham. UUIDv4 random bo'lgani uchun har insert tasodifiy joyga tushadi → page split, fragmentation, sekin write va katta index. Distributed tizimda auto-increment markaziy bottleneck va to'qnashuv yaratadi, shuning uchun u yerda ULID/UUIDv7 eng yaxshi: distributed-safe va index-friendly.

### ❓ M:N munosabatni relational model'da qanday ifodalaysiz?

**✅ Javob:** To'g'ridan-to'g'ri ifodalab bo'lmaydi — **junction table** (bridge/associative) yaratiladi, unda har ikki tomonga FK bo'ladi va odatda composite primary key (`PRIMARY KEY (a_id, b_id)`). Junction table o'z atributlarini ham (sana, role, quantity) saqlashi mumkin. Ikkala yo'nalish bo'yicha so'rov kerak bo'lsa, ikkala FK'ga ham index qo'yiladi.

### ❓ Soft delete'ning xavflari nimada?

**✅ Javob:** (1) Har query'ga `WHERE deleted_at IS NULL` qo'shishni unutish o'chirilgan ma'lumotni "tiriltirib" qo'yadi. (2) Unique constraint'lar buziladi (o'chirilgan va yangi row bir xil email bilan) — partial unique index kerak. (3) Jadval o'sib boraveradi va performance pasayadi. To'liq audit kerak bo'lsa, alohida history/audit jadval kuchliroq.

### ❓ Pulni nima uchun FLOAT'da saqlamaslik kerak?

**✅ Javob:** FLOAT/DOUBLE binary floating-point bo'lgani uchun o'nlik kasrlarni aniq ifodalay olmaydi (`0.1 + 0.2 != 0.3`). Bu pul hisob-kitobida yaxlitlash xatolari beradi. Har doim `NUMERIC`/`DECIMAL` (fixed-point) ishlating — u aniq o'nlik arifmetikani kafolatlaydi.

### ❓ Partitioning va sharding farqi nimada?

**✅ Javob:** **Partitioning** — bitta DBMS instance ichida bitta jadvalni fizik bo'laklarga bo'lish (mantiqan bitta jadval qoladi). **Sharding** — ma'lumotni bir nechta alohida server/node'ga taqsimlash (horizontal scaling). Partitioning bitta mashina chegarasida query/maintenance'ni yaxshilaydi; sharding bir mashinaga sig'maydigan hajm va yuk uchun, ammo cross-shard query/transaction murakkabligini keltiradi.

### ❓ Shard key'ni qanday tanlaysiz va yomon tanlovning oqibati nima?

**✅ Javob:** Shard key teng taqsimlanish (hotspot'siz) va eng tez-tez query'lar bilan mos kelishi kerak (cross-shard query'ni minimallashtirish). Yomon tanlov: low-cardinality key (`country`) → barcha trafik bitta shard'ga (hotspot); query'lar shard key'ni ishlatmasligi → har query barcha shard'larga (scatter-gather). Shard key qaytarib bo'lmas qaror — resharding qimmat.

### ❓ CAP theorem'ni tushuntiring. "3 dan 2" ta'rifi nega chalg'ituvchi?

**✅ Javob:** Distributed tizimda network partition paytida Consistency va Availability'dan faqat bittasini tanlash mumkin. "3 dan 2 tasini tanla" noto'g'ri, chunki Partition tolerance'ni "tanlamaslik" mumkin emas — tarmoq bo'linishi real tizimda muqarrar. Demak haqiqiy tanlov faqat partition paytida CP (consistency'ni saqla, availability'ni qurbon qil) yoki AP (available qol, consistency'ni qurbon qil) orasida.

### ❓ PACELC CAP'dan nimasi bilan to'liqroq?

**✅ Javob:** CAP faqat partition holatini ko'radi. PACELC qo'shimcha: **Else** (partition yo'q, normal holat) da ham tizim Latency va Consistency orasida tanlov qiladi. Ya'ni har doim trade-off bor: agar Partition (PA/PC), Else Latency/Consistency (EL/EC). Masalan Cassandra PA/EL, Spanner PC/EC. Bu replica'lardan sinxron tasdiq kutish (consistency, sekinroq) yoki kutmaslik (latency past, eventual) tanlovini tushuntiradi.

### ❓ Zero-downtime schema migration'ni qanday qilasiz?

**✅ Javob:** **Expand/contract** pattern bilan: (1) Expand — yangi ustun/jadvalni backward-compatible (nullable, default'siz) qo'shing. (2) Migrate — kod yangiga ham yozadi (dual-write), eski ma'lumotni batch'lab backfill qiling. (3) Contract — barcha kod o'tgach, eski ustunni o'chiring. Rename'ni hech qachon bitta qadamda qilmang; index'ni `CONCURRENTLY` yarating; backfill'ni katta UPDATE emas, batch bilan qiling.

### ❓ Leader-follower replikatsiyada "read your own writes" muammosi nima?

**✅ Javob:** Async replikatsiyada follower leader'dan orqada qoladi (replication lag). Foydalanuvchi leader'ga yozib, darhol follower'dan o'qisa — o'z yozuvini ko'rmaydi. Yechimlar: foydalanuvchining yaqinda yozgan ma'lumotini leader'dan o'qish, sticky routing, yoki yozgandan keyin ma'lum vaqt leader'dan o'qish. Bu eventual consistency'ning amaliy ko'rinishi.

### ❓ Polyglot persistence nima va xavfi nimada?

**✅ Javob:** Har bir vazifa uchun eng mos datastore'ni ishlatish (RDBMS transactional, Redis cache, Elasticsearch search, Neo4j graph). Xavfi — har bir yangi store operatsion narx, ekspertiza va store'lar orasida ma'lumotni sinxronlash (dual-write, CDC, consistency) murakkabligi qo'shadi. Ko'pincha bitta yaxshi RDBMS (JSONB, full-text) yetarli; polyglot'ga faqat real zarurat bo'lganda o'ting.

---

## Masalalar

> Yechimlar: [solutions/databases/07-database-design.md](../solutions/databases/07-database-design.md)

1. **Normal forma aniqlash.** Quyidagi jadval qaysi normal formani buzadi va nega? Tuzating:
   `enrollments(student_id, course_id, student_name, course_title, grade)` — PK `(student_id, course_id)`.

2. **M:N dizayn.** Ijtimoiy tarmoq uchun "foydalanuvchilar bir-birini follow qiladi" (followers/following) munosabatini schema bilan modellashtiring. Self-referencing M:N'ni hisobga oling.

3. **Primary key tanlash.** Global, sharded write-heavy "events" jadvali uchun primary key strategiyasini tanlang va tanlovingizni index locality hamda distributed-safety nuqtai nazaridan asoslang.

4. **Soft delete + unique.** `users` jadvalida `email` unique bo'lishi, ammo soft delete (`deleted_at`) qo'llanishi kerak. O'chirilgan foydalanuvchi email'i qayta ishlatilishi mumkin. Constraint'ni qanday yozasiz?

5. **Denormalizatsiya qarori.** Bir blog ilovasida har bir post sahifasida muallifning post soni ko'rsatiladi va bu sahifa juda ko'p ochiladi. `JOIN ... COUNT` sekin. Qanday denormallashtirasiz va consistency'ni qanday saqlaysiz?

6. **Partition strategiyasi.** 2 milliard qatorli `logs(id, created_at, level, message)` jadvalida so'rovlarning aksariyati oxirgi 7 kunlik ma'lumotga tegishli va eski loglar 90 kundan keyin o'chiriladi. Partitioning strategiyasini va partition key'ni tanlang, asoslang.

7. **Schema migration rejasi.** Production'da `users.full_name` ustunini `first_name` va `last_name` ga ajratish kerak, downtime'siz. Bosqichma-bosqich (expand/contract) reja yozing.

8. **CAP qarori.** (a) Bank balansi va (b) ijtimoiy tarmoq "like" hisoblagichi uchun partition paytida CP yoki AP tanlaysiz? Har birini asoslang.

9. **E-commerce schema dizayni.** Restoran yetkazib berish ilovasi uchun schema dizayn qiling: foydalanuvchilar, restoranlar, menyu elementlari, buyurtmalar (bir nechta element, har birida miqdor), kuryer tayinlash. Cardinality, junction table, narx snapshot va kerakli index'larni ko'rsating.

10. **Replikatsiya tanlovi.** Global ilova uchun foydalanuvchilar Yevropa va Amerikadan yozadi, past latency kerak, ammo vaqti-vaqti bilan write conflict bo'lishi mumkin. Qaysi replikatsiya topologiyasini tanlaysiz va conflict'ni qanday hal qilasiz?

---

← [Database bo'limiga qaytish](./README.md)
