# Database asoslari (Fundamentals)

Database (ma'lumotlar bazasi) — zamonaviy dasturlarning yuragi. Deyarli har bir backend intervyusi database savollaridan boshlanadi. Ushbu hujjatda database nima ekani, nega oddiy fayllardan ustun ekani, relational model, schema, key'lar, constraint'lar va boshqa fundamental tushunchalarni ko'rib chiqamiz.

## Mundarija

- [Database nima va nega kerak](#database-nima-va-nega-kerak)
- [DBMS nima](#dbms-nima)
- [Relational vs Non-relational](#relational-vs-non-relational)
- [Relational model](#relational-model)
- [Schema](#schema)
- [Primary key va Foreign key](#primary-key-va-foreign-key)
- [OLTP vs OLAP](#oltp-vs-olap)
- [Structured, semi-structured, unstructured](#structured-semi-structured-unstructured)
- [SQL roli](#sql-roli)
- [Mashhur database'lar](#mashhur-databaselar)
- [Query qanday bajariladi](#query-qanday-bajariladi)
- [Data type'lar](#data-typelar)
- [NULL semantikasi](#null-semantikasi)
- [Constraint'lar](#constraintlar)
- [Database vs Data Warehouse](#database-vs-data-warehouse)
- [Savol-javoblar](#savol-javoblar)
- [Masalalar](#masalalar)

---

## Database nima va nega kerak

**💡 Tushuncha:** Database — bu tartibli, oson kirish (access) va boshqarish mumkin bo'lgan, doimiy (persistent) saqlanadigan ma'lumotlar to'plami. Odatda u **DBMS** (Database Management System) orqali boshqariladi.

Ma'lumotlarni oddiy fayllarda (masalan, `.txt`, `.csv`, JSON) ham saqlash mumkin. Kichik dasturlar ko'pincha shunday qiladi. Lekin tizim o'sgani sayin fayllar quyidagi muammolarga duch keladi:

| Muammo (fayllarda) | Database qanday hal qiladi |
|--------|----------------------------|
| Concurrent access (bir vaqtda ko'p yozuvchi) faylni buzadi | Locking, MVCC, transactions |
| Qidiruv — kodda butun faylni skanerlash kerak | Index, query optimizer, deklarativ SQL |
| Integrity — yomon ma'lumotni hech narsa to'xtatmaydi | Constraint, type, foreign key |
| Atomicity — yozuv o'rtasida crash data buzadi | ACID transactions, write-ahead logging |
| Scale — 50 GB fayl ishlatib bo'lmaydi | Buffer pool, B-tree, partitioning |
| Security — fayl ruxsatlari qo'pol | Jadval/qator/rol darajasidagi access control |

**⚠️ Ehtiyot bo'l:** "Database ishlat" degani har doim ham *relational* database emas. Config fayli yoki 200 qatorli jadval uchun SQLite yoki hatto JSON to'g'ri tanlov bo'lishi mumkin. Qaror access pattern, concurrency va integrity ehtiyojiga bog'liq — obro'ga emas.

---

## DBMS nima

**💡 Tushuncha:** DBMS — bu "dvigatel". U storage, xotira (buffer/page cache), concurrency, recovery, security va query processing'ni boshqaradi. Misollar: PostgreSQL, MySQL, Oracle, SQL Server, SQLite, MongoDB.

DBMS odatda quyidagilarni ta'minlaydi:

- **Data model** — ma'lumot qanday tuzilgan (relational, document, key-value, graph).
- **Query language** — SQL yoki API.
- **Transaction management** — ACID kafolati.
- **Concurrency control** — lock yoki MVCC, ko'p klient bir vaqtda ishlashi uchun.
- **Recovery** — logging orqali crash'dan omon qolish.
- **Storage management** — page, index, caching.

**✅ Farqi:** **database** — bu *ma'lumot*; **DBMS** — bu *dasturiy ta'minot*. Odamlar "database" so'zini ikkalasiga ham nisbatan ishlatadi.

---

## Relational vs Non-relational

**💡 Tushuncha:** Relational database'lar ma'lumotni qat'iy schema'li **jadvallarda** saqlaydi va SQL ishlatadi. Non-relational ("NoSQL") boshqa modellardan foydalanadi.

| Tur | Model | Misollar | Qachon yaxshi |
|-----|-------|----------|---------------|
| **Relational (SQL)** | Table/row/column, key orqali bog'lanish | PostgreSQL, MySQL, SQLite | Structured data, murakkab query, JOIN, kuchli consistency |
| **Document** | JSON-like hujjatlar | MongoDB, CouchDB | Moslashuvchan schema, nested data |
| **Key-Value** | Oddiy key → value | Redis, DynamoDB | Cache, session, juda tez lookup |
| **Wide-column** | Dinamik ustunli qatorlar | Cassandra, HBase | Katta write scale, time-series |
| **Graph** | Node + edge | Neo4j | Kuchli bog'langan data (social, fraud) |

**⚠️ Ehtiyot bo'l:** "NoSQL relational'dan yaxshi" degan gap noto'g'ri. Chegara ham xira: PostgreSQL `JSONB` orqali document'ni qo'llab-quvvatlaydi, ko'p NoSQL tizimlar esa SQL-ga o'xshash til taklif qiladi.

---

## Relational model

**💡 Tushuncha:** E. F. Codd (1970) taklif qilgan relational model ma'lumotni **relation** (table) ko'rinishida ifodalaydi. U set theory va predicate logic'ga asoslanadi.

Atamalar (formal → kundalik):

| Formal atama | Kundalik atama |
|--------------|----------------|
| Relation | Table (jadval) |
| Tuple | Row / record (qator) |
| Attribute | Column / field (ustun) |
| Domain | Ustunning data type'i |
| Cardinality | Qatorlar soni |
| Degree | Ustunlar soni |

```sql
-- "employees" relation, degree = 4
CREATE TABLE employees (
    id          INTEGER PRIMARY KEY,
    name        TEXT NOT NULL,
    department  TEXT,
    salary      NUMERIC(10, 2)
);
```

Har bir qator — tuple; qatorlar to'plami — relation. Muhim nuqta: **haqiqiy relation'da qatorlar tartibsiz (unordered) va noyob** — shuning uchun tartiblash kerak bo'lsa `ORDER BY` ishlatiladi.

---

## Schema

**💡 Tushuncha:** "Schema" so'zi ikki ma'noda ishlatiladi:

1. **Logical schema** — ma'lumot *strukturasi*: qaysi jadvallar, ularning ustunlari, type'lari, bog'lanishlari. Bu blueprint.
2. **Namespace schema** (PostgreSQL/Oracle) — obyektlarni (jadval, view, function) guruhlovchi konteyner. Masalan, `public.employees` va `hr.employees`.

```sql
CREATE SCHEMA hr;
CREATE TABLE hr.employees ( id INT PRIMARY KEY, name TEXT );

SELECT * FROM hr.employees;   -- to'liq (qualified) nom
```

Namespace schema obyektlarni tartibga solish, guruh bo'yicha ruxsat berish va nom to'qnashuvini oldini olishga yordam beradi (masalan, multi-tenant ilovada har bir tenant'ga alohida schema).

---

## Primary key va Foreign key

**💡 Tushuncha:** **Primary key (PK)** — har bir qatorni noyob aniqlaydi. U **UNIQUE** va **NOT NULL** bo'lishi shart, jadvalda eng ko'pi bilan bitta bo'ladi (bir nechta ustundan iborat bo'lsa — *composite key*).

**Foreign key (FK)** — bir jadvaldagi ustun bo'lib, boshqa jadvalning primary key'iga ishora qiladi va **referential integrity**ni ta'minlaydi — mavjud bo'lmagan qatorga ishora qila olmaysiz.

```sql
CREATE TABLE users (
    id    BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    email TEXT UNIQUE NOT NULL
);

CREATE TABLE orders (
    id      BIGINT PRIMARY KEY,
    user_id BIGINT NOT NULL REFERENCES users(id)
);

-- user 999 yo'q bo'lsa, bu xato beradi:
INSERT INTO orders (id, user_id) VALUES (1, 999);  -- ERROR: FK violation
```

**Referential action**'lar parent qator o'chirilganda/yangilanganda nima bo'lishini boshqaradi:

```sql
user_id BIGINT REFERENCES users(id)
    ON DELETE CASCADE      -- child qatorlarni ham o'chirish
    ON DELETE SET NULL     -- ishorani NULL qilish
    ON DELETE RESTRICT     -- o'chirishni bloklash
```

- **Natural key** — biznes ma'nosiga ega key (email, passport raqami).
- **Surrogate key** — sun'iy key (auto-increment `id`, UUID). Odatda afzal, chunki biznes qiymatlar o'zgaradi, `id` esa hech qachon o'zgarmasligi kerak.

**⚠️ Ehtiyot bo'l:** PostgreSQL'da foreign key ustun avtomatik index'lanmaydi (faqat *referenced* PK index'langan). Index'lanmagan FK parent'dagi `DELETE`/`UPDATE`'ni sekinlashtiradi. FK ustunlarini index qiling.

---

## OLTP vs OLAP

**💡 Tushuncha:** Bu ikki butunlay boshqacha workload.

| | **OLTP** | **OLAP** |
|---|----------|----------|
| To'liq nomi | Online Transaction Processing | Online Analytical Processing |
| Maqsad | Biznesni yuritish (kundalik) | Biznesni tahlil qilish (BI) |
| Operatsiyalar | Ko'p kichik read/write | Kam, katta murakkab read |
| Misol | "Order joylashtirish" | "Hududlar bo'yicha kvartal daromadi" |
| Qatorlar | Har query'da bir nechta | Millionlab, jamlangan |
| Schema | Normalized | Denormalized (star/snowflake) |
| Storage | Row-oriented | Ko'pincha column-oriented |
| Tizim | PostgreSQL, MySQL | Snowflake, BigQuery, ClickHouse |

**⚠️ Ehtiyot bo'l:** Og'ir OLAP query'larni production OLTP database'ga yo'naltirsangiz, uni "yiqitib" qo'yishingiz mumkin. Odatiy yondashuv — analitika uchun ma'lumotni alohida warehouse'ga ETL/replicate qilish.

---

## Structured, semi-structured, unstructured

**💡 Tushuncha:** Ma'lumot schema'sining qat'iyligiga ko'ra tasniflanadi.

- **Structured** — qat'iy table schema'ga to'g'ri keladi; har bir qatorda bir xil ustunlar. Misol: `users` jadvali. SQL bilan oson.
- **Semi-structured** — teg/kalitlar bor, lekin shakli moslashuvchan; qat'iy schema yo'q. Misol: JSON, XML. `JSONB` operatorlari bilan query qilinadi.
- **Unstructured** — oldindan belgilangan model yo'q. Misol: rasm, video, audio, erkin matn, PDF. Odatda object storage (S3) da, metadata esa database'da saqlanadi.

```sql
-- Semi-structured ma'lumotni relational DB ichida JSONB bilan
CREATE TABLE events ( id BIGINT PRIMARY KEY, payload JSONB );
SELECT payload->>'user_id' FROM events WHERE payload->>'type' = 'click';
```

---

## SQL roli

**💡 Tushuncha:** SQL (Structured Query Language) — **deklarativ** til: siz natijada *NIMA* kerakligini aytasiz, *QANDAY* topishni optimizer hal qiladi.

SQL pastki tillarga bo'linadi:

| Pastki til | Maqsad | Statement'lar |
|------------|--------|---------------|
| **DDL** | Strukturani belgilash | `CREATE`, `ALTER`, `DROP`, `TRUNCATE` |
| **DML** | Ma'lumot bilan ishlash | `SELECT`, `INSERT`, `UPDATE`, `DELETE` |
| **DCL** | Ruxsatni boshqarish | `GRANT`, `REVOKE` |
| **TCL** | Transaction boshqarish | `COMMIT`, `ROLLBACK`, `SAVEPOINT` |

SQL — ANSI/ISO standarti, lekin har bir vendor o'z dialektini qo'shadi (PostgreSQL'da `RETURNING`, MySQL'da boshqa `LIMIT` sintaksisi va h.k.).

---

## Mashhur database'lar

| Database | Tur | Nima bilan mashhur |
|----------|-----|---------------------|
| **PostgreSQL** | Relational | Standartga mos, kengaytiriladigan, boy type'lar (JSONB, array), kuchli to'g'rilik. OSS default tanlov. |
| **MySQL / MariaDB** | Relational | Web stack'larda keng tarqalgan, tez read, katta ekotizim |
| **SQLite** | Relational (embedded) | Serversiz, bitta fayl, zero-config; mobil/desktop/test |
| **Oracle / SQL Server** | Relational | Enterprise, kommersiyaviy, yetuk tooling |
| **MongoDB** | Document | Moslashuvchan schema, JSON hujjatlar |
| **Redis** | Key-value | In-memory cache, queue, pub/sub |
| **Cassandra** | Wide-column | Katta write throughput, horizontal scale |

**⚠️ Ehtiyot bo'l:** SQLite "yengil/o'yinchoq SQL" emas. U to'liq SQL engine — faqat serversiz, in-process ishlaydi. Milliardlab qurilmalarda ishlaydi. Cheklovi — bir vaqtda faqat bitta yozuvchi (writer), funksiyalar emas.

---

## Query qanday bajariladi

**💡 Tushuncha:** Siz query yuborganingizda, DBMS ma'lumotga tegmasdan oldin uni pipeline orqali o'tkazadi.

```
SQL matn
   │
   ▼
┌──────────┐   Tokenize + sintaksis tekshirish; parse tree quradi.
│  PARSER  │   Jadval/ustun mavjudligini tekshiradi (semantic).
└──────────┘
   │
   ▼
┌──────────────┐  Rewrite (view ochish), keyin OPTIMIZER ko'plab
│  PLANNER /   │  bajarilish rejalarini ko'rib chiqadi (index scan
│  OPTIMIZER   │  vs seq scan, JOIN tartibi/metodi) va statistika
└──────────────┘  asosida eng arzonini tanlaydi.
   │
   ▼
┌──────────┐   Tanlangan rejani bajaradi: page o'qiydi, filter,
│ EXECUTOR │   JOIN, aggregate qiladi, qatorlarni qaytaradi.
└──────────┘
   │
   ▼
 Result set
```

1. **Parser** — sintaksisni va obyektlar mavjudligini tekshiradi; parse tree quradi.
2. **Rewriter** — qoidalarni qo'llaydi (view'larni ochadi).
3. **Planner/Optimizer** — nomzod rejalar yaratadi va statistika (qator soni, qiymat taqsimoti) asosida cost hisoblab, eng arzonini tanlaydi. Shuning uchun **statistika muhim**: eski statistika yomon reja beradi.
4. **Executor** — rejani fizik bajaradi va natijani qaytaradi.

```sql
EXPLAIN SELECT * FROM employees WHERE department = 'Sales';
EXPLAIN ANALYZE SELECT * FROM employees WHERE department = 'Sales';
```

**⚠️ Ehtiyot bo'l:** SQL deklarativ bo'lgani uchun bir xil natija qaytaradigan ikki query butunlay boshqacha tezlikda ishlashi mumkin. `EXPLAIN` natijasini o'qish (seq scan vs index scan) — asosiy ko'nikma.

---

## Data type'lar

**💡 Tushuncha:** Har bir ustun type'ga (domain) ega. To'g'ri type tanlash joyni tejaydi, yomon datani oldini oladi va query'ni tezlashtiradi.

| Kategoriya | PostgreSQL type'lar | Izoh |
|------------|---------------------|------|
| Integer | `SMALLINT`, `INTEGER`, `BIGINT` | 2 / 4 / 8 bayt |
| Auto-increment | `SERIAL`, `GENERATED ... AS IDENTITY` | Identity — SQL-standart usul |
| Exact decimal | `NUMERIC(p, s)` / `DECIMAL` | Pul uchun — rounding xatosi yo'q |
| Floating point | `REAL`, `DOUBLE PRECISION` | Tez, taxminiy — **pul uchun emas** |
| Text | `VARCHAR(n)`, `CHAR(n)`, `TEXT` | Postgres'da `TEXT` afzal, perf jarima yo'q |
| Boolean | `BOOLEAN` | `TRUE`/`FALSE`/`NULL` |
| Date/time | `DATE`, `TIMESTAMP`, `TIMESTAMPTZ` | `TIMESTAMPTZ` (timezone-aware) afzal |
| JSON | `JSON`, `JSONB` | `JSONB` binary, indexlanadi — afzal |
| UUID | `UUID` | 128-bit identifikator |

**⚠️ Ehtiyot bo'l:** Pul uchun `NUMERIC` ishlating. `FLOAT`/`DOUBLE` binary floating point bo'lib, `0.1 + 0.2 != 0.3`, balanslar vaqt o'tib "siljiydi".

---

## NULL semantikasi

**💡 Tushuncha:** `NULL` — *noma'lum* yoki *yo'q* degani — **nol emas** va **bo'sh satr emas**. U **uch qiymatli mantiq** (three-valued logic) ishlatadi: `TRUE`, `FALSE`, `UNKNOWN`.

```sql
SELECT NULL = NULL;       -- NULL (unknown), TRUE emas!
SELECT NULL = 5;          -- NULL
SELECT NULL IS NULL;      -- TRUE  (IS NULL / IS NOT NULL ishlat)

SELECT 5 + NULL;          -- NULL  (NULL bilan arifmetika → NULL)
SELECT 'a' || NULL;       -- NULL  (konkatenatsiya ham)
```

Asosiy xatti-harakatlar:

- `WHERE` faqat predikat **TRUE** bo'lgan qatorlarni qaytaradi (UNKNOWN emas), shuning uchun `WHERE col = NULL` hech narsa qaytarmaydi — `IS NULL` ishlat.
- Ko'p aggregate'lar **NULL'ni e'tiborsiz qoldiradi**: `AVG(col)` NULL qatorlarni o'tkazadi; `COUNT(col)` NULL'ni sanamaydi, ammo `COUNT(*)` hammasini sanaydi.
- `UNIQUE` bir nechta NULL'ga ruxsat beradi (har NULL boshqasidan farqli).
- `GROUP BY` barcha NULL'ni **bitta** guruh deb hisoblaydi.

**⚠️ Ehtiyot bo'l:** `NOT IN (subquery)` ichida bitta NULL bo'lsa ham *hech qator qaytarmaydi*, chunki `x NOT IN (1, NULL)` → UNKNOWN. `NOT EXISTS` afzal.

---

## Constraint'lar

**💡 Tushuncha:** Constraint — DBMS data'ga majburlovchi qoidalar, shunda yaroqsiz qatorlar umuman mavjud bo'la olmaydi. Ular to'g'rilikni ilova kodidan database'ga ko'chiradi — yagona haqiqat manbai.

```sql
CREATE TABLE products (
    id        SERIAL PRIMARY KEY,
    sku       TEXT UNIQUE NOT NULL,             -- UNIQUE + NOT NULL
    name      TEXT NOT NULL,                    -- NOT NULL
    price     NUMERIC(10,2) NOT NULL DEFAULT 0, -- DEFAULT
    discount  NUMERIC(4,2) DEFAULT 0,
    CHECK (price >= 0),                         -- CHECK
    CHECK (discount BETWEEN 0 AND 1)
);
```

| Constraint | Kafolat |
|------------|---------|
| `NOT NULL` | Ustunda har doim qiymat bor |
| `UNIQUE` | Ikki qator bir xil qiymatga ega emas (NULL ruxsat) |
| `PRIMARY KEY` | UNIQUE **va** NOT NULL; qator identifikatsiyasi |
| `FOREIGN KEY` | Qiymat referenced jadvalda mavjud |
| `CHECK` | Maxsus boolean shart bajariladi |
| `DEFAULT` | Qiymat berilmasa ishlatiladi |

**⚠️ Ehtiyot bo'l:** `CHECK` shart `TRUE` *yoki* `UNKNOWN` (NULL) bo'lganda o'tadi. `CHECK (price >= 0)` NULL price'ni **rad etmaydi** — `NOT NULL` ham kerak.

---

## Database vs Data Warehouse

**💡 Tushuncha:** Ikkalasi ham data saqlaydi, lekin qarama-qarshi ish uchun qurilgan.

| | **Operational DB (OLTP)** | **Data Warehouse (OLAP)** |
|---|---------------------------|---------------------------|
| Optimallashtirilgan | Transaction, joriy holat | Analitika, tarixiy trend |
| Data | Jonli, tez-tez yangilanadi | Davriy yuklanadi (ETL), append-mostly |
| Schema | Normalized | Star/snowflake, denormalized |
| Query | Qisqa point lookup & write | Katta scan & aggregation |
| Vaqt | Hozir | Oylar/yillar tarixi |
| Misol | PostgreSQL, MySQL | Snowflake, BigQuery, Redshift |

**Data lake** yana bir qadam oldinga: xom, unstructured/semi-structured data arzon (object storage) saqlanadi va schema o'qish vaqtida qo'llaniladi ("schema-on-read"), warehouse esa "schema-on-write".

---

## Savol-javoblar

### ❓ Nega fayl o'qish/yozish o'rniga database ishlatamiz?

**✅ Javob:** Database concurrency control, indexlangan query, constraint orqali integrity, ACID transaction, crash'dan recovery va aniq security'ni beradi — bularning hammasini xom fayllar ustida o'zingiz (yomon) qaytadan yozishingiz kerak bo'lardi. Fayllar kichik, single-writer, append-only yoki config holatlarida yaxshi.

### ❓ Database va DBMS farqi nima?

**✅ Javob:** Database — bu haqiqiy ma'lumotlar to'plami; DBMS — uni boshqaradigan dasturiy ta'minot (storage, query, transaction, concurrency). Kundalik nutqda "database" ikkalasiga ham nisbatan ishlatiladi.

### ❓ Relational model nima?

**✅ Javob:** Codd (1970) modeli bo'lib, ma'lumotni relation (table) — tuple (row) va attribute (column) ko'rinishida ifodalaydi, set theory'ga asoslanadi. Jadvallar orasidagi bog'lanish pointer emas, balki umumiy key qiymatlari orqali. Qatorlar tartibsiz va noyob.

### ❓ Primary key va unique key farqi nima?

**✅ Javob:** Primary key — `UNIQUE` + `NOT NULL`, har qatorni aniqlaydi, jadvalda eng ko'pi bitta. Unique key faqat noyoblikni majburlaydi, NULL'ga ruxsat beradi (Postgres'da ko'p NULL), jadvalda bir nechta bo'lishi mumkin.

### ❓ Natural key va surrogate key — qaysi birini ishlatish kerak?

**✅ Javob:** Natural key biznes ma'nosiga ega (email, passport); surrogate key sun'iy (auto-increment id, UUID). Primary key uchun surrogate afzal, chunki biznes qiymatlar o'zgaradi yoki qayta ishlatiladi, bu referential integrity'ni buzadi. Natural qiymatni `UNIQUE` bilan saqlash mumkin.

### ❓ Foreign key nimani majburlaydi va referential action'lar nima?

**✅ Javob:** U referential integrity'ni majburlaydi — child qator faqat mavjud parent qatorga ishora qila oladi. Referential action (`ON DELETE/UPDATE`) parent o'zgarganda xatti-harakatni belgilaydi: `CASCADE`, `SET NULL`, `SET DEFAULT`, `RESTRICT`, `NO ACTION`.

### ❓ Nega `WHERE col = NULL` ishlamaydi?

**✅ Javob:** NULL "noma'lum" degani, shuning uchun `col = NULL` → `UNKNOWN`, `TRUE` emas, va `WHERE` faqat TRUE bo'lgan qatorlarni qaytaradi. `IS NULL` / `IS NOT NULL` ishlating.

### ❓ NULL 0 yoki bo'sh satrdan qanday farq qiladi?

**✅ Javob:** 0 va '' — *ma'lum* qiymatlar. NULL — qiymatning *yo'qligi*. NULL bilan arifmetika va taqqoslash NULL beradi, ko'p aggregate'lar NULL'ni o'tkazadi, 0 yoki '' esa hisobga olinadi.

### ❓ `COUNT(*)`, `COUNT(col)` va `COUNT(DISTINCT col)` farqi?

**✅ Javob:** `COUNT(*)` barcha qatorlarni sanaydi. `COUNT(col)` faqat `col` NULL bo'lmagan qatorlarni. `COUNT(DISTINCT col)` noyob, NULL bo'lmagan qiymatlarni sanaydi.

### ❓ OLTP va OLAP?

**✅ Javob:** OLTP ko'p kichik tez transactional read/write'ni normalized data ustida bajaradi (biznesni yuritish). OLAP kam katta analitik aggregation'ni denormalized data ustida (biznesni tahlil qilish). Schema, storage va tizimlar har xil.

### ❓ Structured, semi-structured va unstructured data?

**✅ Javob:** Structured qat'iy schema'ga to'g'ri keladi (relational table). Semi-structured moslashuvchan, o'zini ta'riflovchi struktura (JSON/XML). Unstructured model'siz (rasm, video, erkin matn). Zamonaviy relational DB semi-structured'ni JSONB orqali boshqaradi.

### ❓ Nega SQL "deklarativ" deyiladi?

**✅ Javob:** Siz qanday natija kerakligini aytasiz, uni hisoblashning qadam-baqadam protsedurasini emas. Optimizer *qanday* qilishni hal qiladi (qaysi index, JOIN tartibi, algoritm). Imperativ kodda esa loop'larni o'zingiz yozasiz.

### ❓ Query qanday bajarilishini tushuntiring.

**✅ Javob:** Parser (sintaksis + semantic → parse tree) → rewriter (view ochish) → planner/optimizer (nomzod rejalar, statistika bilan cost, eng arzonini tanlash) → executor (rejani bajaradi, qatorlarni qaytaradi). `EXPLAIN ANALYZE` tanlangan reja va real vaqtni ko'rsatadi.

### ❓ Nega pul uchun `FLOAT` o'rniga `NUMERIC`?

**✅ Javob:** `FLOAT`/`DOUBLE` binary floating point bo'lib, ko'p o'nliklarni aniq ifodalay olmaydi (`0.1 + 0.2 != 0.3`), bu rounding "siljishi"ga olib keladi. `NUMERIC(p,s)` aniq o'nlik qiymatni saqlaydi.

### ❓ Database, data warehouse va data lake farqi?

**✅ Javob:** Operational database = OLTP, joriy holat, normalized. Warehouse = OLAP, tarixiy, denormalized, schema-on-write. Lake = xom ko'p formatli data arzon saqlanadi, schema-on-read.

### ❓ Qachon NoSQL'ni relational'dan ustun tanlaysiz?

**✅ Javob:** Schema juda o'zgaruvchan bo'lsa, ekstremal horizontal write scale kerak bo'lsa, access pattern oddiy key lookup yoki document fetch bo'lsa, yoki data tabiatan graph bo'lsa. Relational murakkab query, JOIN va kuchli consistency'da yutadi. Ko'pincha ikkalasi birga ishlatiladi (polyglot persistence).

---

## Masalalar

> Yechimlar: [01-fundamentals yechimlari](../solutions/databases/01-fundamentals.md)

1. **Schema dizayni.** Kutubxona (library) tizimi uchun `books`, `authors`, `members` jadvallarini ta'riflang. Har birida mos data type, primary key va kamida bitta `NOT NULL` constraint bo'lsin. `books` jadvalida `authors`'ga foreign key bo'lsin.

2. **Constraint qo'llash.** `employees` jadvalini yarating: `id` (PK), `email` (unique, not null), `salary` (musbat bo'lishi shart — CHECK), `status` (default `'active'`), `hired_at` (default — joriy sana).

3. **NULL semantikasi.** Quyidagi har bir ifoda natijasini (TRUE / FALSE / NULL) yozing va sababini tushuntiring: `NULL = NULL`, `NULL IS NULL`, `5 > NULL`, `NULL OR TRUE`, `NULL AND FALSE`, `COUNT(null_column)`.

4. **OLTP vs OLAP.** Quyidagilarni OLTP yoki OLAP'ga ajrating: (a) foydalanuvchi savatga mahsulot qo'shadi; (b) marketing oxirgi 3 yil savdosini hududlar bo'yicha tahlil qiladi; (c) bankomatdan pul yechish; (d) yillik daromad hisoboti.

5. **Data turini tanlash.** Quyidagi ustunlar uchun eng mos data type'ni tanlang va asoslang: foydalanuvchi yoshi, mahsulot narxi, telefon raqami, ro'yxatdan o'tgan vaqt, "faolmi" bayrog'i (flag), foydalanuvchi profili (moslashuvchan JSON).

6. **Relational vs NoSQL.** Quyidagilar uchun relational yoki NoSQL tanlang va sababini yozing: (a) bank tranzaksiyalari; (b) real-time chat cache'i; (c) turli atributli e-commerce katalog; (d) ijtimoiy tarmoq do'stlik grafigi.

7. **Foreign key xatti-harakati.** `groups` va `students` bog'langan. Guruhni o'chirmoqchisiz va unda talabalar bor. `ON DELETE CASCADE`, `ON DELETE SET NULL` va `ON DELETE RESTRICT` har birining oqibatini tushuntiring.

8. **Query hayot sikli.** `SELECT name FROM users WHERE age > 30 ORDER BY name` so'rovi uchun parser, planner va executor bosqichlarini o'z so'zlaringiz bilan tasvirlang. Planner qanday qaror qabul qilishi mumkin?

---

← [Database bo'limiga qaytish](./README.md)
