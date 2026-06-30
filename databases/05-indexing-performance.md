# Indexing va Performance

Database katta jadvalda kerakli qatorni millisekundlarda topishi yoki sekundlab kutdirishi mumkin — farqni `index` va to'g'ri yozilgan `query` belgilaydi. Bu bo'limda index ichki tuzilishi, turlari, qachon ishlamasligi, query plan o'qish, N+1 muammosi va pagination performance ko'rib chiqiladi.

## Mundarija

- [Index nima va trade-off](#index-nima-va-trade-off)
- [B-tree index qanday ishlaydi](#b-tree-index-qanday-ishlaydi)
- [Clustered vs non-clustered index](#clustered-vs-non-clustered-index)
- [Primary vs secondary index](#primary-vs-secondary-index)
- [Composite index va leftmost-prefix qoidasi](#composite-index-va-leftmost-prefix-qoidasi)
- [Covering index](#covering-index)
- [Unique va partial index](#unique-va-partial-index)
- [Hash, full-text va GIN/GiST index](#hash-full-text-va-gingist-index)
- [Index qachon ishlatilmaydi](#index-qachon-ishlatilmaydi)
- [Query plan o'qish (EXPLAIN / EXPLAIN ANALYZE)](#query-plan-oqish-explain--explain-analyze)
- [N+1 query muammosi](#n1-query-muammosi)
- [Query optimizatsiya](#query-optimizatsiya)
- [Denormalizatsiya](#denormalizatsiya)
- [Connection pooling](#connection-pooling)
- [Pagination performance: offset vs keyset](#pagination-performance-offset-vs-keyset)
- [Sekin query sabablari](#sekin-query-sabablari)
- [Savol-javoblar (Q&A)](#savol-javoblar-qa)
- [Masalalar](#masalalar)

---

## Index nima va trade-off

**💡 Tushuncha:** Index — bu jadvaldagi qatorlarni butun jadvalni o'qimasdan tez topish uchun yaratilgan qo'shimcha data struktura. Aynan kitob oxiridagi alfavitli ko'rsatkichga o'xshaydi: "B-tree" so'zini topish uchun har bir sahifani o'qimaysiz, ko'rsatkichdan kerakli sahifa raqamiga sakraysiz.

Indexsiz `WHERE email = 'a@b.com'` so'rovi **sequential scan** (har bir qatorni o'qish, O(n)) talab qiladi. `email` ustunida B-tree index bilan qidiruv O(log n) bo'ladi.

Lekin index bepul emas — bu **trade-off**:

| Jihat | Index qo'shilganda |
|-------|--------------------|
| Read tezligi | **Tezroq** — point lookup, range, sort uchun |
| Write tezligi | **Sekinroq** — har bir `INSERT`/`UPDATE`/`DELETE` indexni ham yangilashi kerak |
| Storage | **Ko'proq** — index alohida disk strukturasi |
| Maintenance | Index bloat/fragment bo'lishi, vaqti-vaqti rebuild talab qilishi mumkin |

**⚠️ Ehtiyot bo'l:** "Har bir ustunga index qo'yaman" — bu xato. Write-heavy jadvalda ortiqcha indexlar INSERT/UPDATE'ni sekinlashtiradi va diskni yeydi. Index faqat real query'lar bo'yicha qo'yiladi.

---

## B-tree index qanday ishlaydi

**💡 Tushuncha:** Aksariyat databaselarda standart index — **B-tree** (aniqrog'i B+tree). Bu balanslangan daraxt: barcha leaf node'lar bir xil chuqurlikda, va leaf node'lar bir-biriga linked list orqali bog'langan.

```
                 [ 50 | 100 ]
                /     |      \
         [10|30]   [60|80]   [120|150]
         /  |  \   ...        ...
      leaf leaf leaf  →  →  →  (linked list, sorted)
```

Nega aynan B-tree:

- **Balanslangan** — daraxt chuqurligi har doim O(log n), shuning uchun istalgan qator uchun lookup taxminan bir xil vaqt oladi.
- **Keng (wide), past (shallow)** — har bir node ko'p kalit saqlaydi, shuning uchun millionlab qator atigi 3-4 daraja chuqurlikda joylashadi. Bu disk I/O'ni minimallaydi.
- **Range va sort'ni qo'llab-quvvatlaydi** — leaf'lar tartiblangan va linked. `WHERE age BETWEEN 20 AND 30` yoki `ORDER BY age` index orqali samarali bajariladi.

**❓ Nega hash index emas, balki B-tree default?**

**✅ Javob:** Hash index faqat aniq tenglik (`=`) uchun ishlaydi va O(1) beradi, lekin range (`<`, `>`, `BETWEEN`), `ORDER BY`, va prefix qidiruvni umuman qo'llab-quvvatlamaydi. B-tree esa equality, range, sort, prefix — hammasini bir strukturada beradi. Shu universal qulayligi uchun u default.

---

## Clustered vs non-clustered index

**💡 Tushuncha:** Bu jadvalda qatorlar **jismonan** qanday saqlanishini belgilaydi.

| | Clustered index | Non-clustered (secondary) index |
|---|------------------|---------------------------------|
| Ta'rif | Jadval qatorlari index tartibida diskda saqlanadi | Alohida struktura, qatorga pointer saqlaydi |
| Soni | Jadvalda faqat **1 ta** (chunki qatorlar bir xil tartibda) | Ko'p bo'lishi mumkin |
| Leaf node | Butun qatorni o'zida saqlaydi | Index kalit + qatorga reference (PK yoki row pointer) |
| Misol | InnoDB (MySQL): PK = clustered | Qo'shimcha `CREATE INDEX` indexlari |

**⚠️ Ehtiyot bo'l:** MySQL InnoDB'da har bir secondary index leaf'i **primary key**ni saqlaydi, qator pointerini emas. Demak secondary index orqali qidiruv ba'zan ikki bosqichli bo'ladi: secondary index → PK → clustered index (buni "bookmark lookup" deyiladi). Shuning uchun InnoDB'da juda katta PK (masalan UUID string) barcha secondary indexlarni shishiradi.

PostgreSQL'da esa jadval **heap** sifatida saqlanadi (clustered emas) va barcha indexlar non-clustered. `CLUSTER` komandasi bilan bir martalik fizik tartiblash mumkin, lekin u avtomatik saqlanmaydi.

---

## Primary vs secondary index

**💡 Tushuncha:**

- **Primary index** — primary key ustida avtomatik yaratiladi. Unique va NOT NULL. InnoDB'da bu clustered index.
- **Secondary index** — boshqa ustun(lar)da qo'lda yaratilgan index (`CREATE INDEX`). Qidiruv, JOIN, filter, sort'ni tezlashtirish uchun.

```sql
-- primary (avtomatik, PRIMARY KEY orqali)
CREATE TABLE users (
    id    BIGINT PRIMARY KEY,
    email TEXT,
    city  TEXT
);

-- secondary
CREATE INDEX idx_users_email ON users (email);
CREATE INDEX idx_users_city  ON users (city);
```

---

## Composite index va leftmost-prefix qoidasi

**💡 Tushuncha:** Composite (multi-column) index bir nechta ustun ustida yaratiladi. Ularning **tartibi** juda muhim.

```sql
CREATE INDEX idx_orders_cust_date ON orders (customer_id, created_at);
```

Bu index qatorlarni avval `customer_id` bo'yicha, keyin har bir `customer_id` ichida `created_at` bo'yicha tartiblaydi — telefon kitobi familiya, keyin ism bo'yicha tartiblangani kabi.

**Leftmost-prefix qoidasi:** Index faqat ustunlarning chap tomondan ketma-ket prefiksi bo'yicha ishlaydi. Yuqoridagi index quyidagilarni qo'llaydi:

| Query | Index ishlaydimi? |
|-------|-------------------|
| `WHERE customer_id = 5` | ✅ Ha (leftmost) |
| `WHERE customer_id = 5 AND created_at > '2026-01-01'` | ✅ Ha (to'liq prefiks) |
| `WHERE created_at > '2026-01-01'` | ❌ Yo'q (`customer_id` o'tkazib yuborilgan) |

**❓ Composite index'da ustun tartibini qanday tanlash kerak?**

**✅ Javob:** Umumiy qoida: (1) equality (`=`) bilan ishlatiladigan ustun(lar) oldinda, range (`>`, `<`, `BETWEEN`) ustun keyinda; (2) eng tez-tez yolg'iz filterlanadigan ustun chapda. `WHERE a = ? AND b > ?` uchun `(a, b)` to'g'ri, `(b, a)` yomon.

**⚠️ Ehtiyot bo'l:** `(a, b)` indexi `(b)` bo'yicha qidiruvga yordam bermaydi, lekin `(a)` bo'yicha alohida indexga ehtiyojni yo'qotadi. Demak ortiqcha bitta-ustunli indexlardan saqlaning.

---

## Covering index

**💡 Tushuncha:** Covering index — query'ga kerak bo'lgan **barcha** ustunlarni o'zida saqlaydigan index. Database javobni faqat indexdan oladi, asosiy jadvalga (heap/clustered'ga) umuman murojaat qilmaydi. Buni "index-only scan" deyiladi.

```sql
-- query
SELECT customer_id, created_at FROM orders WHERE customer_id = 5;

-- bu index query'ni to'liq "qoplaydi"
CREATE INDEX idx_orders_cover ON orders (customer_id, created_at);
```

PostgreSQL'da `INCLUDE` bilan filterlanmaydigan, lekin SELECT'da kerak ustunlarni indexga "yopishtirib" qo'yish mumkin:

```sql
CREATE INDEX idx_orders_cover
ON orders (customer_id) INCLUDE (created_at, total);
```

**✅ Foydasi:** Bookmark lookup (index → jadval) yo'qoladi → disk I/O kamayadi → query sezilarli tezlashadi.

---

## Unique va partial index

**Unique index** — qiymatlar takrorlanmasligini ta'minlaydi va ayni paytda qidiruvni tezlashtiradi.

```sql
CREATE UNIQUE INDEX idx_users_email_uniq ON users (email);
```

**Partial index** — jadvalning faqat bir qismini (predikatga mos qatorlarni) indexlaydi. Indexni kichik va tez saqlaydi.

```sql
-- faqat aktiv buyurtmalarni indexlaymiz
CREATE INDEX idx_orders_pending
ON orders (created_at)
WHERE status = 'pending';
```

**✅ Javob:** Agar query'larning 99% `status = 'pending'` bilan ishlasa va `pending` qatorlar jami 2% bo'lsa, partial index to'liq indexdan ancha kichik, tez va arzon bo'ladi.

---

## Hash, full-text va GIN/GiST index

| Index turi | Qachon | Cheklov |
|------------|--------|---------|
| **Hash** | Faqat `=` tenglik, O(1) lookup | Range/sort/prefix yo'q |
| **Full-text** (`tsvector` / FULLTEXT) | Matn ichida so'z qidirish: "find documents containing 'database'" | `LIKE` o'rniga, til-aware (stemming) |
| **GIN** (Generalized Inverted Index) | Ko'p qiymatli ustunlar: `array`, `jsonb`, full-text | Sekin build/update, lekin tez qidiruv |
| **GiST** | Geometriya, range type, nearest-neighbor, full-text | Egiluvchan, lekin GIN'dan biroz sekinroq qidiruv |

```sql
-- jsonb ustunida GIN
CREATE INDEX idx_doc_data ON documents USING GIN (data);

-- full-text
CREATE INDEX idx_articles_fts ON articles USING GIN (to_tsvector('english', body));
SELECT * FROM articles WHERE to_tsvector('english', body) @@ to_tsquery('database');
```

---

## Index qachon ishlatilmaydi

**⚠️ Ehtiyot bo'l:** Index mavjud bo'lsa ham, query uni ishlatmasligi mumkin. Eng keng tarqalgan sabablar:

**1. Ustunga funksiya/operatsiya qo'llanilsa:**

```sql
-- ❌ index ishlamaydi (created_at ustiga funksiya)
WHERE YEAR(created_at) = 2026
-- ✅ index ishlaydi (range)
WHERE created_at >= '2026-01-01' AND created_at < '2027-01-01'
```

Yechim: yo query'ni qayta yozish, yoki **functional/expression index** yaratish:
```sql
CREATE INDEX idx_users_lower_email ON users (LOWER(email));
```

**2. Leading wildcard `LIKE`:**

```sql
WHERE name LIKE '%john%'   -- ❌ B-tree ishlamaydi (prefiks noma'lum)
WHERE name LIKE 'john%'    -- ✅ ishlaydi (prefiks aniq)
```

**3. Low cardinality (kam xil qiymat):** `gender`, `is_active` (boolean) kabi 2-3 xil qiymatli ustunda index foydasiz — planner qatorlarning yarmiga teginishni ko'rib, baribir seq scan tanlaydi.

**4. Implicit type cast:** `WHERE phone = 998901234567` (ustun `VARCHAR`, qiymat son) — cast index'ni buzadi.

**5. `OR` shartlar** ba'zan har bir tarmoq alohida indexlanmagan bo'lsa seq scan'ga olib keladi.

---

## Query plan o'qish (EXPLAIN / EXPLAIN ANALYZE)

**💡 Tushuncha:** `EXPLAIN` — database query'ni qanday bajarishni **rejalashtirayotganini** ko'rsatadi (haqiqatda ishga tushirmaydi). `EXPLAIN ANALYZE` — query'ni **haqiqatan ishga tushiradi** va real vaqt/qatorlarni qaytaradi.

```sql
EXPLAIN ANALYZE
SELECT * FROM orders WHERE customer_id = 5;
```

Asosiy node turlari (eng yomondan yaxshigacha emas, kontekstga bog'liq):

| Node | Ma'nosi |
|------|---------|
| **Seq Scan** | Butun jadval o'qiladi — kichik jadvalda OK, kattada signal |
| **Index Scan** | Index orqali topib, jadvaldan qator olinadi |
| **Index Only Scan** | Javob faqat indexdan (covering index) — eng tez |
| **Bitmap Heap/Index Scan** | Ko'p qator mos kelganda index + heap kombinatsiyasi |
| **Nested Loop / Hash Join / Merge Join** | JOIN bajarish strategiyalari |

Nimaga e'tibor berish kerak:

- **Estimated vs actual rows** — agar planner 10 qator deb o'ylab, aslida 1,000,000 chiqsa → statistika eskirgan, `ANALYZE` ishga tushiring.
- **Seq Scan katta jadvalda** + `WHERE` filter → index yetishmayotgan bo'lishi mumkin.
- Eng katta `cost` / `actual time` bo'lgan node — bottleneck.

**❓ "cost=0.00..431.00" nimani anglatadi?**

**✅ Javob:** Birinchi son — birinchi qatorni qaytarishgacha bo'lgan startup cost, ikkinchisi — barcha qatorlarni qaytarishgacha total cost. Bu absolut vaqt emas, planner'ning ichki nisbiy o'lchov birligi; planlarni solishtirish uchun ishlatiladi.

---

## N+1 query muammosi

**💡 Tushuncha:** N+1 — ORM'lar (Hibernate, ActiveRecord, Sequelize, Django ORM) bilan eng keng tarqalgan performance bug. 1 ta query bilan ro'yxat olinadi, keyin har bir element uchun yana 1 ta query yuboriladi → jami N+1 query.

```js
// ❌ N+1: 1 ta query authorlar uchun + N ta query har bir author kitoblari uchun
const authors = await db.query('SELECT * FROM authors');         // 1
for (const a of authors) {
  a.books = await db.query('SELECT * FROM books WHERE author_id = $1', [a.id]); // N
}
```

**Yechimlar:**

**1. JOIN (eager loading bitta query bilan):**
```sql
SELECT a.*, b.* FROM authors a
LEFT JOIN books b ON b.author_id = a.id;
```

**2. `IN` bilan batch (2 ta query):**
```sql
SELECT * FROM authors;
SELECT * FROM books WHERE author_id IN (1, 2, 3, ...);
```

**3. ORM eager loading:** `include` (Sequelize), `joinedload`/`selectinload` (SQLAlchemy), `prefetch_related` (Django), `JOIN FETCH` (JPA).

**4. DataLoader** (GraphQL) — bir tick ichidagi so'rovlarni batch qiladi.

**⚠️ Ehtiyot bo'l:** N+1 lokal kichik DB'da sezilmaydi, lekin production'da (network latency × N) sahifani sekundlab sekinlashtiradi. Har doim ORM'ning generatsiya qilayotgan query'larini log qiling.

---

## Query optimizatsiya

**💡 Tushuncha:** Tartib bo'yicha asosiy texnikalar:

- **`SELECT *` o'rniga kerakli ustunlar** — kamroq data, covering index imkoniyati.
- **`WHERE` filterni indexlanadigan qilish** — ustunga funksiya qo'ymaslik.
- **Kerakli indexlar** — query plan'ga qarab.
- **JOIN tartibini va shartlarni tekshirish** — indexlangan ustunlar bo'yicha JOIN.
- **`LIMIT` qo'shish** — barcha qatorlar kerak bo'lmasa.
- **`EXISTS` vs `IN`** — katta subquery'da ko'pincha `EXISTS` tezroq.
- **Aggregat/og'ir hisoblarni materialized view'ga** o'tkazish.
- **Statistikani yangilab turish** (`ANALYZE`).

```sql
-- ❌
SELECT * FROM orders WHERE customer_id IN (SELECT id FROM customers WHERE country = 'UZ');
-- ✅ ko'pincha tezroq
SELECT o.* FROM orders o
WHERE EXISTS (SELECT 1 FROM customers c WHERE c.id = o.customer_id AND c.country = 'UZ');
```

---

## Denormalizatsiya

**💡 Tushuncha:** Denormalizatsiya — JOIN'larni kamaytirish va read'ni tezlashtirish uchun ataylab takroriy data qo'shish (normalizatsiyaning teskarisi).

Misol: har bir `order`ga `customer_name`ni nusxalab qo'yish, har safar `customers` bilan JOIN qilmaslik uchun. Yoki `posts` jadvalida `comment_count` ustunini saqlash (har safar `COUNT` qilmaslik uchun).

| Plus | Minus |
|------|-------|
| Read tezroq (kam JOIN) | Write murakkab (bir nechta joyni yangilash) |
| Aggregat oldindan hisoblangan | Data anomaliya/nomuvofiqlik xavfi |
| Report query'lar sodda | Ko'proq storage |

**⚠️ Ehtiyot bo'l:** Denormalizatsiya — oxirgi chora, premature emas. Avval indexing va query optimizatsiyani sinab ko'ring. Denormalizatsiya qilganda nusxalangan datani sinxron saqlash mas'uliyati sizga o'tadi (trigger, application logic yoki periodik job orqali).

---

## Connection pooling

**💡 Tushuncha:** Har bir yangi DB connection ochish qimmat (TCP handshake, autentifikatsiya, backend process). Connection pool — oldindan ochilgan connectionlar to'plamini saqlaydi va so'rovlarga qayta ishlatib beradi.

```
App  →  [Pool: 20 ta ochiq connection]  →  Database
         (so'rov keldi → bo'sh conn ber → ish tugadi → qaytar)
```

- **PgBouncer / pgpool** (PostgreSQL), **HikariCP** (Java), driver-level pool'lar (Node `pg`, Python `asyncpg`).
- Pool hajmi muhim: juda kichik → so'rovlar navbatda kutadi; juda katta → DB CPU/memory tugaydi. Odatda `pool_size ≈ ((core_count * 2) + effective_spindles)` atrofida boshlanadi.

**⚠️ Ehtiyot bo'l:** Serverless (Lambda) muhitda har bir instance o'z poolini ochsa, DB connection limiti tez tugaydi. Bu yerda tashqi pooler (RDS Proxy, PgBouncer) shart.

---

## Pagination performance: offset vs keyset

**💡 Tushuncha:** `LIMIT ... OFFSET ...` oddiy, lekin chuqur sahifalarda sekin.

```sql
-- OFFSET pagination
SELECT * FROM posts ORDER BY created_at DESC LIMIT 20 OFFSET 100000;
```

**⚠️ Ehtiyot bo'l:** `OFFSET 100000` — database 100,020 qatorni o'qib, birinchi 100,000 tasini **tashlab yuboradi**. Offset qanchalik chuqur bo'lsa, shunchalik sekin. Bundan tashqari, sahifalar orasida yangi qator qo'shilsa, elementlar siljiydi (dublikat/o'tkazib yuborish).

**Keyset (cursor) pagination** — oxirgi ko'rilgan qator qiymatidan davom etadi:

```sql
-- birinchi sahifa
SELECT * FROM posts ORDER BY created_at DESC, id DESC LIMIT 20;
-- keyingi sahifa: oxirgi elementning (created_at, id) qiymatidan
SELECT * FROM posts
WHERE (created_at, id) < ('2026-06-01 10:00', 5012)
ORDER BY created_at DESC, id DESC
LIMIT 20;
```

| | OFFSET | Keyset/Cursor |
|---|--------|---------------|
| Tezligi | Chuqurlikda sekin (O(offset)) | Doimo tez (index seek, O(log n)) |
| "Sahifaga sakrash" | Mumkin (page 500) | Faqat keyingi/oldingi |
| Ma'lumot o'zgarsa | Siljish/dublikat | Barqaror |
| Infinite scroll | Yomon | Ideal |

---

## Sekin query sabablari

**Checklist** — query sekin bo'lsa, tartib bo'yicha tekshiring:

1. **Index yo'q** yoki ishlamayapti (`EXPLAIN`'da Seq Scan).
2. **Ustunga funksiya/cast** — index buzilgan.
3. **N+1** — ko'p mayda query application'dan.
4. **`SELECT *`** — keraksiz ustunlar, covering index yo'qoladi.
5. **Eskirgan statistika** — planner noto'g'ri plan tanlaydi (`ANALYZE`).
6. **Katta OFFSET pagination**.
7. **Kartezian/noto'g'ri JOIN** — qatorlar ko'payib ketgan.
8. **Lock kutish** — boshqa transaction qatorni bloklab turibdi.
9. **Index bloat / fragmentation** — `REINDEX`/`VACUUM` kerak.
10. **Memory yetmasligi** — sort/hash diskka tushgan (`work_mem` kam).

---

## Savol-javoblar (Q&A)

### ❓ Index nima va u qanday trade-off keltiradi?

**✅ Javob:** Index — qatorlarni butun jadvalni skanlamasdan tez topish uchun yordamchi data struktura. Read'ni tezlashtiradi, lekin write'ni sekinlashtiradi (har bir DML index'ni ham yangilaydi) va qo'shimcha storage talab qiladi.

### ❓ Nega aksariyat databaselarda default index — B-tree?

**✅ Javob:** B-tree balanslangan, past chuqurlikli daraxt bo'lib, equality, range (`<`, `>`, `BETWEEN`), `ORDER BY` va prefix qidiruvni — barchasini bitta strukturada qo'llab-quvvatlaydi. Hash faqat tenglik bilan ishlaydi, shuning uchun universal emas.

### ❓ Clustered va non-clustered index farqi nima?

**✅ Javob:** Clustered index jadval qatorlarini diskda jismonan index tartibida saqlaydi (jadvalda faqat 1 ta bo'ladi, masalan InnoDB PK). Non-clustered index alohida struktura bo'lib, qatorga reference saqlaydi va jadvalda ko'p bo'lishi mumkin.

### ❓ Leftmost-prefix qoidasi nima?

**✅ Javob:** Composite index `(a, b, c)` faqat ustunlarning chap prefiksi bo'yicha ishlaydi: `(a)`, `(a, b)`, `(a, b, c)`. `(b)` yoki `(b, c)` bo'yicha yolg'iz qidiruvda bu index yordam bermaydi.

### ❓ Composite index'da ustun tartibini qanday tanlaysiz?

**✅ Javob:** Equality (`=`) bilan filterlanadigan ustunlar oldinda, range bilan filterlanadigan ustunlar keyinda. Eng selektiv va eng tez-tez yolg'iz ishlatiladigan ustun chapda.

### ❓ Covering index nima va u nimani tezlashtiradi?

**✅ Javob:** Query'ga kerakli barcha ustunlarni o'zida saqlaydigan index. Database javobni faqat indexdan oladi (index-only scan), asosiy jadvalga teginmaydi — bu bookmark lookup'ni va disk I/O'ni yo'qotadi.

### ❓ Partial index qachon foydali?

**✅ Javob:** Query'lar jadvalning kichik, aniq qismini (masalan `status = 'pending'`) doimo filterlasa. Partial index faqat o'sha qatorlarni indexlaydi → kichik, tez, kam storage.

### ❓ Index borligiga qaramay nega query uni ishlatmasligi mumkin?

**✅ Javob:** Ustunga funksiya/cast qo'llanilsa (`YEAR(date)`), `LIKE '%...'` leading wildcard, low cardinality ustun, implicit type cast yoki planner statistikasi eskirgan bo'lsa.

### ❓ EXPLAIN va EXPLAIN ANALYZE farqi nima?

**✅ Javob:** `EXPLAIN` faqat rejalashtirilgan planni (estimate) ko'rsatadi, query'ni ishga tushirmaydi. `EXPLAIN ANALYZE` query'ni haqiqatan bajaradi va real vaqt hamda haqiqiy qator sonlarini qaytaradi.

### ❓ Seq Scan har doim yomonmi?

**✅ Javob:** Yo'q. Kichik jadvalda yoki query qatorlarning katta qismini qaytarganda Seq Scan index scan'dan tezroq bo'lishi mumkin. U faqat katta jadvalda selektiv `WHERE` bilan ishlaganda muammo signali.

### ❓ N+1 query muammosi nima va qanday hal qilinadi?

**✅ Javob:** Ro'yxat uchun 1 query, keyin har bir element uchun yana 1 query (jami N+1) yuborilishi. Yechim: JOIN bilan eager loading, `IN` bilan batch query, ORM eager loading (`prefetch`/`include`/`JOIN FETCH`) yoki DataLoader.

### ❓ OFFSET pagination nega chuqur sahifalarda sekin?

**✅ Javob:** `OFFSET N` database'ni N qatorni o'qib, tashlab yuborishga majbur qiladi. N qancha katta bo'lsa, shuncha sekin. Keyset (cursor) pagination oxirgi qator qiymatidan `WHERE`+index seek bilan davom etadi — doimo tez.

### ❓ Connection pooling nima uchun kerak?

**✅ Javob:** Yangi DB connection ochish qimmat (handshake, auth, backend process). Pool oldindan ochilgan connectionlarni qayta ishlatadi — latency va DB yukini kamaytiradi.

### ❓ Denormalizatsiya qachon o'zini oqlaydi?

**✅ Javob:** Read-heavy tizimda, JOIN/aggregat ko'p va sekin bo'lganda, indexing va query optimizatsiya yetmaganda. Bu data nomuvofiqlik xavfini va write murakkabligini oshiradi, shuning uchun oxirgi chora.

### ❓ GIN index qachon ishlatiladi?

**✅ Javob:** Bir qatorda ko'p qiymat bo'lgan ustunlar uchun: `jsonb`, `array`, full-text (`tsvector`). U qidiruv tez, lekin build/update sekinroq.

### ❓ Query sekin bo'lsa, qayerdan boshlaysiz?

**✅ Javob:** `EXPLAIN ANALYZE` bilan plan o'qish: Seq Scan bormi, estimated vs actual rows mosmi, eng katta cost qaysi node'da. Keyin index, query qayta yozish, statistikani yangilash yoki N+1'ni hal qilish.

---

## Masalalar

> Yechimlar: [solutions/databases/05-indexing-performance.md](../solutions/databases/05-indexing-performance.md)

Quyidagi sxemadan foydalaning:

```sql
CREATE TABLE customers (
    id      BIGINT PRIMARY KEY,
    email   TEXT,
    country TEXT,
    created_at TIMESTAMP
);

CREATE TABLE orders (
    id          BIGINT PRIMARY KEY,
    customer_id BIGINT REFERENCES customers(id),
    status      TEXT,          -- 'pending' | 'paid' | 'cancelled'
    total       NUMERIC(10,2),
    created_at  TIMESTAMP
);
```

1. `SELECT * FROM orders WHERE customer_id = 5 AND status = 'paid'` query'sini tezlashtirish uchun composite index yozing va ustun tartibini tushuntiring.

2. `(customer_id, created_at)` indexi mavjud. Quyidagi uchta query'dan qaysilari indexdan foydalanadi, qaysilari yo'q va nega?
   - a) `WHERE customer_id = 5`
   - b) `WHERE created_at > '2026-01-01'`
   - c) `WHERE customer_id = 5 AND created_at > '2026-01-01'`

3. Quyidagi query'ni indexdan foydalanadigan qilib qayta yozing:
   `SELECT * FROM customers WHERE LOWER(email) = 'a@b.com'`. Ikki xil yechim taklif qiling (query qayta yozish va functional index).

4. `SELECT id, total FROM orders WHERE customer_id = 5` uchun covering index (index-only scan) yarating.

5. Faqat `status = 'pending'` buyurtmalar ustida partial index yarating va u nega to'liq indexdan afzal ekanligini yozing.

6. Quyidagi N+1 muammoli psevdokodni bitta JOIN query'siga aylantiring:
   ```js
   const customers = await q('SELECT * FROM customers');
   for (const c of customers)
     c.orders = await q('SELECT * FROM orders WHERE customer_id = $1', [c.id]);
   ```

7. `SELECT * FROM orders ORDER BY created_at DESC LIMIT 20 OFFSET 50000` query'sini keyset (cursor) pagination'ga aylantiring. Index ham taklif qiling.

8. `EXPLAIN ANALYZE` natijasida `Seq Scan on orders` ko'rsangiz, muammoni aniqlash uchun qaysi 4 narsani tekshirasiz?

9. `WHERE total > 100` query'si juda sekin. `orders` jadvalida 50 million qator bor va ularning 90% `total > 100`. Index yordam beradimi? Javobingizni asoslang.

---

← [Database bo'limiga qaytish](./README.md)
