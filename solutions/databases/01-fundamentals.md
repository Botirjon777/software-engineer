# 01 — Database asoslari: Masalalar yechimi

Bu fayl [`databases/01-fundamentals.md`](../../databases/01-fundamentals.md) dagi **Masalalar** bo'limining yechimlarini o'z ichiga oladi. Avval o'zingiz urinib ko'ring, keyin solishtiring.

---

## 1-masala. Schema dizayni (library)

```sql
CREATE TABLE authors (
    id          BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    full_name   TEXT NOT NULL,
    born_year   SMALLINT,
    country     TEXT
);

CREATE TABLE members (
    id          BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    full_name   TEXT NOT NULL,
    email       TEXT NOT NULL UNIQUE,
    joined_at   DATE NOT NULL DEFAULT CURRENT_DATE
);

CREATE TABLE books (
    id          BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    title       TEXT NOT NULL,
    author_id   BIGINT NOT NULL REFERENCES authors(id),
    isbn        TEXT UNIQUE,
    published   SMALLINT,
    copies      INT NOT NULL DEFAULT 1 CHECK (copies >= 0)
);
```

**Izoh:** Har bir jadval surrogate primary key (`id`) ishlatadi. `books.author_id` — foreign key. `isbn` va `email` noyob (UNIQUE). `NOT NULL` muhim maydonlarda. `copies` manfiy bo'lmasligi uchun CHECK.

---

## 2-masala. Constraint qo'llash

```sql
CREATE TABLE employees (
    id        BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    email     TEXT NOT NULL UNIQUE,
    salary    NUMERIC(12,2) NOT NULL CHECK (salary > 0),
    status    TEXT NOT NULL DEFAULT 'active',
    hired_at  DATE NOT NULL DEFAULT CURRENT_DATE
);
```

**Izoh:** `salary > 0` — CHECK constraint (musbat). `status` va `hired_at` da DEFAULT. `email` da UNIQUE + NOT NULL. Esda tuting: CHECK NULL'ni rad etmaydi, shuning uchun `salary` ga `NOT NULL` ham qo'shildi.

---

## 3-masala. NULL semantikasi

| Ifoda | Natija | Sabab |
|-------|--------|-------|
| `NULL = NULL` | `NULL` | NULL bilan taqqoslash har doim UNKNOWN |
| `NULL IS NULL` | `TRUE` | `IS NULL` — NULL'ni tekshirishning to'g'ri yo'li |
| `5 > NULL` | `NULL` | Noma'lum bilan taqqoslash → UNKNOWN |
| `NULL OR TRUE` | `TRUE` | `TRUE` har qanday narsani TRUE qiladi (short-circuit) |
| `NULL AND FALSE` | `FALSE` | `FALSE` har qanday narsani FALSE qiladi |
| `COUNT(null_column)` | NULL bo'lmagan qatorlar soni | `COUNT(col)` NULL'larni sanamaydi |

**Izoh:** `NULL OR TRUE` va `NULL AND FALSE` — three-valued logic'dagi maxsus holatlar: ikkinchi operand natijani aniqlay olsa, birinchisi NULL bo'lsa ham javob aniq bo'ladi.

---

## 4-masala. OLTP vs OLAP

| Stsenariy | Tur | Sabab |
|-----------|-----|-------|
| (a) Savatga mahsulot qo'shish | **OLTP** | Kichik, tez write operatsiyasi |
| (b) 3 yillik savdoni hududlar bo'yicha tahlil | **OLAP** | Katta tarixiy aggregation |
| (c) Bankomatdan pul yechish | **OLTP** | Bitta transaction, joriy holat |
| (d) Yillik daromad hisoboti | **OLAP** | Katta scan, jamlash |

---

## 5-masala. Data turini tanlash

| Ustun | Type | Sabab |
|-------|------|-------|
| Foydalanuvchi yoshi | `SMALLINT` (yoki `INT`) | Kichik butun son; yoki tug'ilgan sanani `DATE` saqlash yaxshiroq |
| Mahsulot narxi | `NUMERIC(12,2)` | Aniq o'nlik, rounding xatosiz |
| Telefon raqami | `TEXT` (yoki `VARCHAR`) | Raqam emas — `+`, `0` boshida, format; arifmetika kerak emas |
| Ro'yxatdan o'tgan vaqt | `TIMESTAMPTZ` | Timezone-aware vaqt |
| "Faolmi" bayrog'i | `BOOLEAN` | TRUE/FALSE |
| Foydalanuvchi profili (JSON) | `JSONB` | Indexlanadigan semi-structured |

**Izoh:** Telefon raqamini `INT`/`BIGINT` saqlamaslik kerak — boshidagi nol yo'qoladi, `+998` kabi format saqlanmaydi, arifmetika ma'nosizdir.

---

## 6-masala. Relational vs NoSQL

| Loyiha | Tanlov | Sabab |
|--------|--------|-------|
| (a) Bank tranzaksiyalari | **Relational** | ACID, kuchli consistency, integrity shart |
| (b) Real-time chat cache | **NoSQL (Redis)** | In-memory, ultra-tez key-value lookup |
| (c) Turli atributli katalog | **NoSQL (document)** yoki Postgres+JSONB | Moslashuvchan schema |
| (d) Do'stlik grafigi | **NoSQL (graph, Neo4j)** | Kuchli bog'langan data, ko'p qatlamli munosabat |

**Izoh:** (c) uchun PostgreSQL + JSONB ham yaxshi gibrid yechim — relational ishonchlilik + moslashuvchanlik.

---

## 7-masala. Foreign key xatti-harakati

Agar guruhda talabalar bo'lsa va siz guruhni o'chirmoqchi bo'lsangiz:

| Referential action | Oqibat |
|--------------------|--------|
| `ON DELETE RESTRICT` (yoki default) | O'chirish **bloklanadi**, xato beradi. Avval talabalarni ko'chirish/o'chirish kerak. |
| `ON DELETE CASCADE` | Guruh bilan birga **barcha talabalar ham o'chiriladi**. Xavfli — ehtiyot bo'ling. |
| `ON DELETE SET NULL` | Guruh o'chiriladi, talabalarning `group_id` qiymati **NULL** ga aylanadi (ustun NULL'ga ruxsat berishi shart). |

**Izoh:** Default xatti-harakat `NO ACTION` bo'lib, amalda `RESTRICT`ga o'xshaydi (faqat tekshirish vaqti farqli). Production'da ko'pincha `RESTRICT` yoki `SET NULL` xavfsizroq, `CASCADE` esa ataylab ishlatiladi.

---

## 8-masala. Query hayot sikli

`SELECT name FROM users WHERE age > 30 ORDER BY name`

1. **Parser:** Sintaksisni tekshiradi (`SELECT ... FROM ... WHERE ... ORDER BY` to'g'rimi). `users` jadvali, `name` va `age` ustunlari mavjudligini tekshiradi. Parse tree quradi.

2. **Planner/Optimizer:** Bir nechta variant ko'rib chiqadi:
   - `age` ustunida index bormi? Bor bo'lsa va kam qator `age > 30` bo'lsa — **index scan**.
   - Aks holda — **sequential scan** (butun jadvalni o'qish), keyin filter.
   - `ORDER BY name` uchun: `name` da index bo'lsa, qo'shimcha sort kerak emas; bo'lmasa — natijani **sort** qilish.
   - Statistika (qancha qator `age > 30`) asosida eng arzon rejani tanlaydi.

3. **Executor:** Tanlangan rejani bajaradi — page'larni o'qiydi, `age > 30` filterini qo'llaydi, `name` bo'yicha tartiblaydi, faqat `name` ustunini qaytaradi.

**Izoh:** Agar `users` katta va `age > 30` qatorlar kam bo'lsa, planner index scan'ni tanlaydi; agar ko'p qator mos kelsa, seq scan arzonroq bo'lishi mumkin.

---

← [Database bo'limiga qaytish](../../databases/README.md)
