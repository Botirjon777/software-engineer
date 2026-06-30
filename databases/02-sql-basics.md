# SQL asoslari (SQL Basics)

Kundalik SQL to'plami: jadval ta'riflash, ma'lumotni o'zgartirish va `SELECT` query'larni filter, group hamda JOIN bilan yozish. Bu intervyularda eng ko'p amaliy savol beriladigan bo'lim.

## Mundarija

- [DDL vs DML vs DCL vs TCL](#ddl-vs-dml-vs-dcl-vs-tcl)
- [Namuna schema](#namuna-schema)
- [CREATE / ALTER / DROP TABLE](#create--alter--drop-table)
- [INSERT / UPDATE / DELETE](#insert--update--delete)
- [SELECT asoslari](#select-asoslari)
- [Operatorlar](#operatorlar)
- [Aggregate funksiyalar](#aggregate-funksiyalar)
- [GROUP BY va HAVING](#group-by-va-having)
- [JOIN turlari](#join-turlari)
- [Alias](#alias)
- [NULL bilan ishlash (COALESCE, NULLIF)](#null-bilan-ishlash-coalesce-nullif)
- [SELECT mantiqiy bajarilish tartibi](#select-mantiqiy-bajarilish-tartibi)
- [Savol-javoblar](#savol-javoblar)
- [Masalalar](#masalalar)

---

## DDL vs DML vs DCL vs TCL

**💡 Tushuncha:** SQL statement'lari to'rt kategoriyaga bo'linadi.

| Kategoriya | To'liq nomi | Maqsad | Statement'lar |
|------------|-------------|--------|---------------|
| **DDL** | Data Definition Language | Struktura belgilash | `CREATE`, `ALTER`, `DROP`, `TRUNCATE` |
| **DML** | Data Manipulation Language | Ma'lumot bilan ishlash | `SELECT`, `INSERT`, `UPDATE`, `DELETE` |
| **DCL** | Data Control Language | Ruxsat boshqarish | `GRANT`, `REVOKE` |
| **TCL** | Transaction Control Language | Transaction boshqarish | `COMMIT`, `ROLLBACK`, `SAVEPOINT` |

**⚠️ Ehtiyot bo'l:** `TRUNCATE` — DDL (jadvalni tezda bo'shatadi, odatda rollback qilib bo'lmaydi yoki cheklangan), `DELETE` esa DML (qator-qator, WHERE bilan, rollback mumkin). Ko'pchilik bularni adashtiradi.

---

## Namuna schema

Quyidagi misollar shu jadvallarga asoslanadi:

```sql
CREATE TABLE departments (
    id    INT PRIMARY KEY,
    name  TEXT NOT NULL
);

CREATE TABLE employees (
    id       INT PRIMARY KEY,
    name     TEXT NOT NULL,
    dept_id  INT REFERENCES departments(id),
    salary   NUMERIC(10,2),
    hired_at DATE
);
```

`employees`:

| id | name  | dept_id | salary | hired_at   |
|----|-------|---------|--------|------------|
| 1  | Ali   | 1       | 5000   | 2021-03-01 |
| 2  | Vali  | 1       | 6000   | 2020-07-15 |
| 3  | Hasan | 2       | 4500   | 2022-01-10 |
| 4  | Olim  | NULL    | 7000   | 2019-05-20 |

---

## CREATE / ALTER / DROP TABLE

```sql
-- Yaratish
CREATE TABLE projects (
    id    INT PRIMARY KEY,
    title TEXT NOT NULL
);

-- O'zgartirish
ALTER TABLE projects ADD COLUMN budget NUMERIC(12,2);
ALTER TABLE projects ALTER COLUMN title SET NOT NULL;
ALTER TABLE projects RENAME COLUMN title TO name;
ALTER TABLE projects DROP COLUMN budget;

-- O'chirish
DROP TABLE projects;            -- butun jadvalni o'chiradi
DROP TABLE IF EXISTS projects;  -- mavjud bo'lsa o'chiradi (xato bermaydi)
```

**⚠️ Ehtiyot bo'l:** `DROP TABLE` struktura va data'ni butunlay yo'q qiladi. `TRUNCATE TABLE` faqat ma'lumotni o'chiradi, struktura qoladi. `DELETE` esa qatorlarni (WHERE bilan tanlab) o'chiradi.

---

## INSERT / UPDATE / DELETE

```sql
-- INSERT
INSERT INTO employees (id, name, dept_id, salary, hired_at)
VALUES (5, 'Sardor', 2, 5500, '2023-02-01');

-- Bir nechta qator
INSERT INTO employees (id, name, dept_id, salary)
VALUES (6, 'Aziz', 1, 4800),
       (7, 'Bobur', 2, 5200);

-- UPDATE (WHERE'ni unutmang!)
UPDATE employees SET salary = salary * 1.10 WHERE dept_id = 1;

-- DELETE
DELETE FROM employees WHERE id = 7;

-- PostgreSQL: RETURNING bilan natijani qaytarish
INSERT INTO employees (id, name) VALUES (8, 'Jasur') RETURNING id, name;
```

**⚠️ Ehtiyot bo'l:** `UPDATE`/`DELETE`'da `WHERE`'ni unutsangiz — **butun jadval** o'zgaradi yoki o'chiriladi. Bu eng mashhur xato. Avval `SELECT` bilan tekshiring.

---

## SELECT asoslari

```sql
-- Hamma ustun
SELECT * FROM employees;

-- Tanlangan ustunlar
SELECT name, salary FROM employees;

-- WHERE — filter
SELECT name FROM employees WHERE salary > 5000;

-- ORDER BY — tartiblash (ASC default, DESC kamayish)
SELECT name, salary FROM employees ORDER BY salary DESC;

-- LIMIT / OFFSET — sahifalash (pagination)
SELECT name FROM employees ORDER BY id LIMIT 2 OFFSET 2;  -- 3- va 4-qator

-- DISTINCT — takrorlanmaydigan qiymatlar
SELECT DISTINCT dept_id FROM employees;
```

**⚠️ Ehtiyot bo'l:** `LIMIT` `ORDER BY`siz beqaror (non-deterministic) natija beradi — qatorlar tartibi kafolatlanmaydi. Pagination uchun har doim `ORDER BY` bilan.

---

## Operatorlar

```sql
-- Comparison: =, <>, !=, <, >, <=, >=
SELECT * FROM employees WHERE salary >= 5000;

-- LIKE — pattern matching (% har qanday, _ bitta belgi)
SELECT * FROM employees WHERE name LIKE 'A%';     -- A bilan boshlanadi
SELECT * FROM employees WHERE name LIKE '%n';     -- n bilan tugaydi
SELECT * FROM employees WHERE name LIKE '_li';    -- 3 harf, oxiri "li"

-- ILIKE — case-insensitive (PostgreSQL)
SELECT * FROM employees WHERE name ILIKE 'ali';

-- IN — ro'yxatdan biri
SELECT * FROM employees WHERE dept_id IN (1, 2);

-- BETWEEN — oraliq (chegaralar kiritiladi)
SELECT * FROM employees WHERE salary BETWEEN 4000 AND 6000;

-- IS NULL / IS NOT NULL
SELECT * FROM employees WHERE dept_id IS NULL;
```

**⚠️ Ehtiyot bo'l:** `BETWEEN a AND b` — bu **inclusive** (a va b ham kiradi). `salary NOT IN (1, NULL)` ichida NULL bo'lsa, hech qator qaytmaydi (NULL semantikasi).

---

## Aggregate funksiyalar

**💡 Tushuncha:** Aggregate funksiyalar ko'p qatorni bitta qiymatga jamlaydi.

```sql
SELECT COUNT(*)      FROM employees;              -- barcha qatorlar
SELECT COUNT(dept_id) FROM employees;             -- NULL bo'lmaganlar
SELECT SUM(salary)   FROM employees;              -- yig'indi
SELECT AVG(salary)   FROM employees;              -- o'rtacha
SELECT MIN(salary), MAX(salary) FROM employees;   -- eng kichik/katta
```

| Funksiya | Vazifa | NULL'ga munosabat |
|----------|--------|--------------------|
| `COUNT(*)` | Barcha qatorlar | NULL'ni ham sanaydi |
| `COUNT(col)` | NULL bo'lmagan qatorlar | NULL'ni e'tiborsiz |
| `SUM` / `AVG` | Yig'indi / o'rtacha | NULL'ni e'tiborsiz |
| `MIN` / `MAX` | Eng kichik / katta | NULL'ni e'tiborsiz |

**⚠️ Ehtiyot bo'l:** `AVG(salary)` NULL salary'larni hisobga olmaydi — bu `SUM/COUNT(*)`dan farq qilishi mumkin. Agar NULL'ni 0 deb hisoblamoqchi bo'lsangiz, `AVG(COALESCE(salary, 0))`.

---

## GROUP BY va HAVING

**💡 Tushuncha:** `GROUP BY` qatorlarni guruhlarga bo'ladi, aggregate har bir guruh uchun hisoblanadi. `HAVING` esa **guruhlarni** filter qiladi (aggregate natijasiga ko'ra).

```sql
-- Har bo'lim bo'yicha o'rtacha maosh
SELECT dept_id, AVG(salary) AS avg_salary
FROM employees
GROUP BY dept_id;

-- HAVING — guruhni filter qilish
SELECT dept_id, COUNT(*) AS cnt
FROM employees
GROUP BY dept_id
HAVING COUNT(*) > 1;
```

**WHERE vs HAVING:**

| | `WHERE` | `HAVING` |
|---|---------|----------|
| Nimani filter qiladi | Alohida **qatorlar** | **Guruhlar** (GROUP BY natijasi) |
| Qachon ishlaydi | Guruhlashdan **oldin** | Guruhlashdan **keyin** |
| Aggregate ishlatish | **Mumkin emas** | Mumkin (`HAVING COUNT(*) > 1`) |

```sql
-- To'g'ri: WHERE oldin (qator), HAVING keyin (guruh)
SELECT dept_id, AVG(salary) AS avg_sal
FROM employees
WHERE hired_at >= '2020-01-01'   -- qatorlarni filter
GROUP BY dept_id
HAVING AVG(salary) > 5000;       -- guruhlarni filter
```

**⚠️ Ehtiyot bo'l:** `WHERE`'da `AVG()`, `COUNT()` kabi aggregate ishlatib bo'lmaydi — bu xato. Aggregate sharti uchun `HAVING` kerak.

---

## JOIN turlari

**💡 Tushuncha:** JOIN ikki (yoki undan ko'p) jadvalni umumiy ustun (odatda PK-FK) bo'yicha birlashtiradi.

Tasavvur uchun ikki to'plam (A — chap, B — o'ng):

```
   A          B
 ┌────┐    ┌────┐
 │ 1  │    │ 2  │
 │ 2  │    │ 3  │
 │ 4  │    │ 5  │
 └────┘    └────┘

INNER JOIN  → faqat moslar:           {2, 3 mavjud bo'lganlar} → 2
LEFT JOIN   → A ning hammasi + mos B:  1,2,4 (1 va 4 da B=NULL)
RIGHT JOIN  → B ning hammasi + mos A:  2,3,5 (3 va 5 da A=NULL)
FULL JOIN   → A va B ning hammasi:     1,2,3,4,5
CROSS JOIN  → har bir A x har bir B:   dekart ko'paytmasi (3x3=9)
```

```sql
-- INNER JOIN — faqat ikkala jadvalda mos bor qatorlar
SELECT e.name, d.name AS dept
FROM employees e
INNER JOIN departments d ON e.dept_id = d.id;

-- LEFT JOIN — barcha employees, mos dept bo'lmasa NULL
SELECT e.name, d.name AS dept
FROM employees e
LEFT JOIN departments d ON e.dept_id = d.id;
-- Olim (dept_id=NULL) ham chiqadi, dept ustuni NULL bo'ladi

-- RIGHT JOIN — barcha departments, employees bo'lmasa NULL
SELECT e.name, d.name AS dept
FROM employees e
RIGHT JOIN departments d ON e.dept_id = d.id;

-- FULL OUTER JOIN — ikkala tomonning hammasi
SELECT e.name, d.name AS dept
FROM employees e
FULL OUTER JOIN departments d ON e.dept_id = d.id;

-- CROSS JOIN — dekart ko'paytmasi (har bir kombinatsiya)
SELECT e.name, d.name FROM employees e CROSS JOIN departments d;

-- SELF JOIN — jadvalning o'zi bilan (masalan, manager iyerarxiyasi)
SELECT e.name AS xodim, m.name AS manager
FROM employees e
JOIN employees m ON e.manager_id = m.id;
```

| JOIN turi | Natija |
|-----------|--------|
| `INNER JOIN` | Faqat ikkala tomonda mos bor qatorlar |
| `LEFT JOIN` | Chap jadvalning hammasi + mos o'ng (yo'q bo'lsa NULL) |
| `RIGHT JOIN` | O'ng jadvalning hammasi + mos chap (yo'q bo'lsa NULL) |
| `FULL OUTER JOIN` | Ikkala jadvalning hammasi |
| `CROSS JOIN` | Dekart ko'paytmasi (barcha kombinatsiya) |
| `SELF JOIN` | Jadval o'zi bilan (iyerarxiya, taqqoslash) |

**⚠️ Ehtiyot bo'l:** `LEFT JOIN`'da o'ng jadvaldan shart qo'ymoqchi bo'lsangiz, uni `ON`'da yozing, `WHERE`'da emas. `WHERE d.name = 'X'` LEFT JOIN'ni amalda INNER JOIN'ga aylantiradi (NULL qatorlarni kesib tashlaydi).

---

## Alias

**💡 Tushuncha:** Alias — jadval yoki ustunga vaqtinchalik nom berish. Kodni qisqartiradi va o'qishni osonlashtiradi.

```sql
-- Ustun alias (AS ixtiyoriy)
SELECT salary * 12 AS yearly_salary FROM employees;

-- Jadval alias
SELECT e.name, d.name
FROM employees AS e
JOIN departments AS d ON e.dept_id = d.id;
```

**⚠️ Ehtiyot bo'l:** Ustun alias'ini `WHERE`'da ishlatib bo'lmaydi (mantiqiy tartib — SELECT WHERE'dan keyin bajariladi), lekin `ORDER BY`'da mumkin. PostgreSQL `GROUP BY`'da alias'ga ruxsat beradi (standart emas).

---

## NULL bilan ishlash (COALESCE, NULLIF)

```sql
-- COALESCE — birinchi NULL bo'lmagan qiymatni qaytaradi
SELECT name, COALESCE(salary, 0) AS salary FROM employees;
-- salary NULL bo'lsa, 0 ko'rsatadi

-- COALESCE bir nechta argument bilan
SELECT COALESCE(nickname, full_name, 'Anonim') FROM users;

-- NULLIF — ikki qiymat teng bo'lsa NULL qaytaradi
SELECT NULLIF(value, 0);   -- value=0 bo'lsa NULL, aks holda value
-- Foydali: nolga bo'lishni oldini olish
SELECT total / NULLIF(count, 0) FROM stats;  -- count=0 bo'lsa NULL (xato emas)
```

| Funksiya | Vazifa |
|----------|--------|
| `COALESCE(a, b, c)` | Birinchi NULL bo'lmagan qiymat |
| `NULLIF(a, b)` | `a = b` bo'lsa NULL, aks holda `a` |

**💡 Tushuncha:** `NULLIF(count, 0)` — division by zero xatosini oldini olishning klassik usuli, chunki `x / NULL = NULL` (xato emas).

---

## SELECT mantiqiy bajarilish tartibi

**💡 Tushuncha:** SQL yozilish tartibi va **mantiqiy bajarilish** tartibi har xil. Buni bilish ko'p xatolarni tushuntiradi.

```
Yozilish tartibi:        Mantiqiy bajarilish tartibi:
SELECT                   1. FROM     (+ JOIN)
FROM                     2. WHERE
WHERE                    3. GROUP BY
GROUP BY                 4. HAVING
HAVING                   5. SELECT   (alias shu yerda paydo bo'ladi)
ORDER BY                 6. ORDER BY
LIMIT                    7. LIMIT / OFFSET
```

Shuning uchun:

- `WHERE`'da `SELECT` alias'ini ishlatib bo'lmaydi (WHERE oldin bajariladi).
- `ORDER BY`'da alias ishlatish mumkin (u SELECT'dan keyin).
- `WHERE`'da aggregate ishlatib bo'lmaydi (u GROUP BY'dan oldin); aggregate sharti — `HAVING`.

```sql
SELECT dept_id, AVG(salary) AS avg_sal   -- 5
FROM employees                            -- 1
WHERE hired_at >= '2020-01-01'            -- 2
GROUP BY dept_id                          -- 3
HAVING AVG(salary) > 5000                 -- 4
ORDER BY avg_sal DESC                     -- 6  (alias ishlaydi)
LIMIT 5;                                   -- 7
```

---

## Savol-javoblar

### ❓ DDL, DML, DCL, TCL farqini ayting.

**✅ Javob:** DDL strukturani belgilaydi (`CREATE`, `ALTER`, `DROP`). DML ma'lumot bilan ishlaydi (`SELECT`, `INSERT`, `UPDATE`, `DELETE`). DCL ruxsatni boshqaradi (`GRANT`, `REVOKE`). TCL transaction'ni boshqaradi (`COMMIT`, `ROLLBACK`, `SAVEPOINT`).

### ❓ `DELETE`, `TRUNCATE` va `DROP` farqi nima?

**✅ Javob:** `DELETE` — DML, qatorlarni (WHERE bilan tanlab) o'chiradi, rollback mumkin, sekinroq. `TRUNCATE` — DDL, butun jadvalni tezda bo'shatadi, WHERE yo'q, odatda rollback cheklangan, identity reset bo'ladi. `DROP` — DDL, jadval strukturasini ham, datasini ham butunlay o'chiradi.

### ❓ `WHERE` va `HAVING` farqi nima?

**✅ Javob:** `WHERE` alohida qatorlarni guruhlashdan **oldin** filter qiladi va aggregate ishlatib bo'lmaydi. `HAVING` guruhlarni `GROUP BY`'dan **keyin** filter qiladi va aggregate (`COUNT`, `SUM`) ishlatish mumkin. Tartib: WHERE → GROUP BY → HAVING.

### ❓ `COUNT(*)` va `COUNT(column)` farqi?

**✅ Javob:** `COUNT(*)` barcha qatorlarni (NULL bo'lsa ham) sanaydi. `COUNT(column)` faqat o'sha ustunda NULL bo'lmagan qatorlarni sanaydi. Agar ustunda NULL bo'lsa, ikkala natija farq qiladi.

### ❓ INNER JOIN va LEFT JOIN farqi nima?

**✅ Javob:** INNER JOIN faqat ikkala jadvalda mos topilgan qatorlarni qaytaradi. LEFT JOIN chap jadvalning barcha qatorlarini qaytaradi; o'ng jadvalda mos bo'lmasa, o'ng ustunlar NULL bo'ladi.

### ❓ CROSS JOIN nima va qachon ishlatiladi?

**✅ Javob:** CROSS JOIN — ikki jadvalning dekart ko'paytmasi (har bir chap qator har bir o'ng qator bilan). Natija M×N qator. Masalan, barcha o'lcham va rang kombinatsiyalarini generatsiya qilish, yoki sana jadvali (calendar) yaratish uchun.

### ❓ SELF JOIN nima?

**✅ Javob:** Jadvalni o'zi bilan JOIN qilish, ikki alias bilan. Iyerarxik ma'lumot (xodim → manager), bir jadval ichidagi qatorlarni taqqoslash kabi holatlarda ishlatiladi. Masalan: `employees e JOIN employees m ON e.manager_id = m.id`.

### ❓ `LIKE` operatorida `%` va `_` nimani anglatadi?

**✅ Javob:** `%` — nol yoki ko'p belgi; `_` — aniq bitta belgi. `'A%'` — A bilan boshlanadi; `'%n'` — n bilan tugaydi; `'_li'` — uch belgili, oxiri "li". Case-insensitive uchun PostgreSQL'da `ILIKE`.

### ❓ `IN` va `BETWEEN` operatorlari nima qiladi?

**✅ Javob:** `IN (a, b, c)` qiymat ro'yxatdan biriga teng bo'lsa TRUE. `BETWEEN a AND b` — `>= a AND <= b` (inclusive, chegaralar kiradi). `IN` subquery bilan ham ishlatilishi mumkin.

### ❓ `COALESCE` va `NULLIF` nima qiladi?

**✅ Javob:** `COALESCE(a, b, ...)` argumentlar ichidagi birinchi NULL bo'lmaganini qaytaradi — NULL'ni default qiymat bilan almashtirishda ishlatiladi. `NULLIF(a, b)` agar `a = b` bo'lsa NULL, aks holda `a` qaytaradi — masalan, nolga bo'lishni oldini olish (`x / NULLIF(y, 0)`).

### ❓ `LIMIT`ni `ORDER BY`siz ishlatish nega xavfli?

**✅ Javob:** SQL'da qatorlar tartibi kafolatlanmaydi. `ORDER BY`siz `LIMIT` har safar boshqa qatorlarni qaytarishi mumkin (beqaror natija). Pagination uchun har doim aniq, noyob `ORDER BY` ishlating.

### ❓ SELECT mantiqiy bajarilish tartibini ayting.

**✅ Javob:** FROM (+JOIN) → WHERE → GROUP BY → HAVING → SELECT → ORDER BY → LIMIT/OFFSET. Shuning uchun WHERE'da SELECT alias'i ishlamaydi (WHERE oldin), lekin ORDER BY'da ishlaydi (SELECT'dan keyin).

### ❓ Nega `WHERE`'da `AVG(salary) > 5000` xato beradi?

**✅ Javob:** Chunki aggregate funksiyalar guruhlashdan keyin hisoblanadi, `WHERE` esa guruhlashdan oldin alohida qatorlar ustida ishlaydi. Aggregate sharti uchun `HAVING` kerak.

### ❓ `DISTINCT` nima qiladi va u GROUP BY'dan farqi?

**✅ Javob:** `DISTINCT` natijadagi takrorlanadigan qatorlarni olib tashlaydi. `SELECT DISTINCT dept_id` — noyob bo'lim id'lari. `GROUP BY` esa guruhlaydi va aggregate hisoblash imkonini beradi. Agar aggregate kerak bo'lmasa, DISTINCT odatda yetarli va o'qilishi osonroq.

### ❓ `LEFT JOIN`da o'ng jadval shartini `WHERE`ga qo'yish nega muammo?

**✅ Javob:** LEFT JOIN NULL qatorlar (mos topilmaganlar) qaytaradi. `WHERE d.col = 'X'` shu NULL qatorlarni kesib tashlaydi (NULL = 'X' → UNKNOWN), natijada LEFT JOIN amalda INNER JOIN'ga aylanadi. Shartni `ON`'ga qo'yish kerak.

---

## Masalalar

> Yechimlar: [02-sql-basics yechimlari](../solutions/databases/02-sql-basics.md)

`employees(id, name, dept_id, salary, hired_at)` va `departments(id, name)` jadvallaridan foydalaning.

1. **SELECT + WHERE + ORDER.** Maoshi 5000 dan yuqori xodimlarni maosh bo'yicha kamayish tartibida (eng yuqori birinchi) tanlang, faqat `name` va `salary` ustunlarini.

2. **DISTINCT + IN.** `dept_id` qiymati 1 yoki 2 bo'lgan xodimlarning noyob bo'lim id'larini chiqaring.

3. **Aggregate.** Butun kompaniya bo'yicha: xodimlar soni, umumiy maosh fondi (SUM), o'rtacha maosh va eng yuqori maoshni bitta query'da chiqaring.

4. **GROUP BY + HAVING.** Har bir bo'lim bo'yicha xodimlar sonini chiqaring, lekin faqat 1 dan ortiq xodimi bor bo'limlarni ko'rsating.

5. **WHERE vs HAVING.** 2020-yil 1-yanvardan keyin ishga olingan xodimlar bo'yicha, har bo'limning o'rtacha maoshini hisoblang va faqat o'rtacha maoshi 5000 dan yuqori bo'limlarni chiqaring.

6. **LEFT JOIN.** Barcha xodimlarni va ularning bo'lim nomini chiqaring. Bo'limi bo'lmagan xodim (`dept_id IS NULL`) ham chiqsin, uning bo'lim nomi NULL bo'lsin.

7. **INNER JOIN + filter.** "Sales" bo'limidagi barcha xodimlarning ismini va maoshini chiqaring (bo'lim nomini `departments` jadvalidan oling).

8. **NULL bilan ishlash.** Barcha xodimlar ro'yxatini chiqaring, lekin `salary` NULL bo'lsa, uning o'rniga 0 ko'rsating (ustunni `salary` deb nomlang).

9. **Pagination.** Xodimlarni `id` bo'yicha tartiblab, har sahifada 2 tadan, **2-sahifani** (ya'ni 3- va 4-qatorlarni) chiqaring.

10. **SELF JOIN (kengaytirilgan).** `employees` jadvaliga `manager_id INT` ustuni qo'shildi deb faraz qiling. Har bir xodim va uning manageri ismini bitta natijada chiqaring (manageri yo'qlar ham chiqsin).

---

← [Database bo'limiga qaytish](./README.md)
