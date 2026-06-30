# SQL Advanced

Bu bo'limda SQL'ning kuchli, "junior"ni "senior"dan ajratuvchi vositalari ko'rib chiqiladi: subquery'lar, CTE va recursive CTE, window functions, set operations, `CASE`, view va materialized view, stored procedure / function / trigger asoslari, pivot va eng mashhur intervyu query patternlari (ikkinchi eng katta maosh, duplikatlarni topish, har guruhdan top N, gaps and islands, running total). Barcha misollar **PostgreSQL** (standart SQL) uchun.

## Mundarija

- [Namuna sxema (sample schema)](#namuna-sxema)
- [Subquery (ichki so'rovlar)](#subquery)
- [CTE — WITH](#cte--with)
- [Recursive CTE](#recursive-cte)
- [Window functions](#window-functions)
- [Set operations](#set-operations)
- [CASE expression](#case-expression)
- [View va Materialized View](#view-va-materialized-view)
- [Stored procedure, function, trigger (kirish)](#stored-procedure-function-trigger)
- [Pivot (qatorlarni ustunga aylantirish)](#pivot)
- [Klassik intervyu query patternlari](#klassik-intervyu-query-patternlari)
- [Savol-javoblar](#savol-javoblar)
- [Masalalar](#masalalar)

---

## Namuna sxema

Quyidagi misollarning ko'pi shu jadvallarga tayanadi.

```sql
CREATE TABLE departments (
    id   INT PRIMARY KEY,
    name TEXT
);

CREATE TABLE employees (
    id            INT PRIMARY KEY,
    name          TEXT,
    department_id INT REFERENCES departments(id),
    salary        NUMERIC(10,2),
    manager_id    INT REFERENCES employees(id),
    hired_at      DATE
);

INSERT INTO departments VALUES
    (1, 'Engineering'), (2, 'Sales'), (3, 'HR');

INSERT INTO employees VALUES
    (1, 'Alice', 1, 120000, NULL, '2019-01-10'),
    (2, 'Bob',   1,  95000, 1,    '2020-03-01'),
    (3, 'Carol', 1,  95000, 1,    '2021-07-15'),
    (4, 'Dave',  2,  80000, 1,    '2020-05-20'),
    (5, 'Eve',   2,  70000, 4,    '2022-02-01'),
    (6, 'Frank', 2,  70000, 4,    '2023-09-12'),
    (7, 'Grace', 3,  60000, 1,    '2021-11-30');
```

---

## Subquery

**💡 Tushuncha:** Subquery — bu boshqa so'rov ichiga joylashtirilgan so'rov. To'rtta asosiy ko'rinishi bor: **scalar**, **IN/EXISTS bilan**, **correlated** va **derived table** (FROM ichidagi subquery).

### Scalar subquery — bitta qiymat qaytaradi

Bitta qator va bitta ustun (ya'ni bitta qiymat) qaytaradi. Uni oddiy qiymat kabi ishlatish mumkin — `SELECT` ro'yxatida ham, `WHERE` da ham.

```sql
-- Har bir xodim maoshini kompaniya o'rtacha maoshi bilan birga ko'rsatish
SELECT name,
       salary,
       (SELECT AVG(salary) FROM employees) AS company_avg
FROM employees;
```

**⚠️ Ehtiyot bo'l:** Scalar subquery birdan ortiq qator qaytarsa, xatolik beradi. Bitta qiymat qaytishini kafolatlash uchun `LIMIT 1` yoki aggregate (`MAX`, `AVG`) ishlating.

### IN bilan subquery

```sql
-- 2 kishidan ko'p xodimi bo'lgan bo'limlardagi xodimlar
SELECT name
FROM employees
WHERE department_id IN (
    SELECT department_id
    FROM employees
    GROUP BY department_id
    HAVING COUNT(*) > 2
);
```

### EXISTS bilan subquery

`EXISTS` — subquery hech bo'lmaganda bitta qator qaytarsa `TRUE` bo'ladi. U **qiymatni emas, mavjudlikni** tekshiradi, shuning uchun ko'pincha `IN`dan tezroq, ayniqsa katta jadvallarda.

```sql
-- Kamida bitta qo'l ostidagi xodimi bor menejerlar
SELECT m.name
FROM employees m
WHERE EXISTS (
    SELECT 1
    FROM employees e
    WHERE e.manager_id = m.id
);
```

**💡 Tushuncha:** `IN` ro'yxatdagi qiymatlarni solishtiradi; `EXISTS` esa qator borligini tekshiradi. `NULL` qiymatlar ishtirok etganda farq muhim: `NOT IN` ro'yxatda `NULL` bo'lsa kutilmagan natija (hech narsa qaytmaslik) beradi, `NOT EXISTS` esa bunday muammodan xoli.

### Correlated subquery — tashqi so'rovga bog'liq

Correlated subquery tashqi so'rovning har bir qatori uchun qayta ishlaydi, chunki u tashqi qatorga murojaat qiladi (`e.department_id`).

```sql
-- Har bir xodim, o'z bo'limidagi o'rtacha maoshdan ko'p oladiganlar
SELECT e.name, e.salary
FROM employees e
WHERE e.salary > (
    SELECT AVG(salary)
    FROM employees e2
    WHERE e2.department_id = e.department_id   -- tashqi qatorga bog'lanish
);
```

**⚠️ Ehtiyot bo'l:** Correlated subquery har bir qator uchun qaytadan ishlaydi — bu sekin bo'lishi mumkin. Ko'pincha uni `JOIN` yoki window function bilan almashtirish tezroq.

### Derived table — FROM ichidagi subquery

```sql
-- Har bir bo'lim bo'yicha o'rtacha maoshni avval hisoblab, keyin filtrlash
SELECT dept_stats.department_id, dept_stats.avg_salary
FROM (
    SELECT department_id, AVG(salary) AS avg_salary
    FROM employees
    GROUP BY department_id
) AS dept_stats           -- derived table'ga albatta alias kerak
WHERE dept_stats.avg_salary > 80000;
```

**⚠️ Ehtiyot bo'l:** PostgreSQL'da derived table'ga **alias** majburiy (`AS dept_stats`), aks holda syntax xatosi.

---

## CTE — WITH

**💡 Tushuncha:** CTE (Common Table Expression) — `WITH` orqali e'lon qilinadigan, so'rov ichida bir marta nomlanadigan vaqtinchalik natija to'plami. U murakkab so'rovni o'qiladigan, bosqichma-bosqich bo'laklarga ajratadi.

```sql
WITH dept_avg AS (
    SELECT department_id, AVG(salary) AS avg_salary
    FROM employees
    GROUP BY department_id
)
SELECT e.name, e.salary, d.avg_salary
FROM employees e
JOIN dept_avg d ON e.department_id = d.department_id
WHERE e.salary > d.avg_salary;
```

Bir nechta CTE'ni vergul bilan zanjirlash mumkin:

```sql
WITH dept_avg AS (
    SELECT department_id, AVG(salary) AS avg_salary
    FROM employees
    GROUP BY department_id
),
big_depts AS (
    SELECT department_id FROM dept_avg WHERE avg_salary > 80000
)
SELECT * FROM employees
WHERE department_id IN (SELECT department_id FROM big_depts);
```

**💡 Tushuncha:** CTE vs subquery — natija bir xil, lekin CTE o'qish uchun qulayroq va bir necha joyda qayta ishlatish mumkin. PostgreSQL 12'dan boshlab CTE'lar odatda **inline** qilinadi (optimizator ichiga "qo'shib yuboradi"); eski versiyalarda CTE "optimization fence" bo'lardi (alohida materializatsiya). Kerak bo'lsa `WITH ... AS MATERIALIZED` yoki `NOT MATERIALIZED` bilan boshqarish mumkin.

---

## Recursive CTE

**💡 Tushuncha:** Recursive CTE o'zini-o'ziga murojaat qiladi va ierarxiya (xodim → menejer), graf yoki ketma-ketliklarni qayta ishlash uchun ishlatiladi. Ikki qismdan iborat: **anchor** (boshlang'ich qator) va **recursive** qism (`UNION ALL` orqali takrorlanadigan qadam).

```sql
-- Alice (id=1) ostidagi butun tashkiliy daraxt
WITH RECURSIVE org_chart AS (
    -- anchor: boshlang'ich nuqta
    SELECT id, name, manager_id, 1 AS level
    FROM employees
    WHERE id = 1

    UNION ALL

    -- recursive: oldingi qadam natijasiga ulanish
    SELECT e.id, e.name, e.manager_id, oc.level + 1
    FROM employees e
    JOIN org_chart oc ON e.manager_id = oc.id
)
SELECT * FROM org_chart ORDER BY level, id;
```

Sonlar ketma-ketligini generatsiya qilish:

```sql
WITH RECURSIVE nums AS (
    SELECT 1 AS n
    UNION ALL
    SELECT n + 1 FROM nums WHERE n < 10
)
SELECT n FROM nums;
```

**⚠️ Ehtiyot bo'l:** Recursive qismda to'xtash sharti (`WHERE n < 10`) bo'lmasa — cheksiz loop. Graf'da sikl (cycle) bo'lsa ham cheksiz aylanadi; bunday holatda yurilgan yo'lni array'da saqlab, takrorni tekshiring (PostgreSQL'da `CYCLE` clause bor).

---

## Window functions

**💡 Tushuncha:** Window function har bir qator uchun, unga "bog'liq qatorlar oynasi" (window) bo'yicha hisob qiladi — lekin `GROUP BY`dan farqli ravishda **qatorlarni birlashtirib yubormaydi**. Ya'ni har bir qator saqlanib qoladi, yoniga qo'shimcha hisoblangan ustun qo'shiladi.

Sintaksis: `funksiya() OVER (PARTITION BY ... ORDER BY ...)`.

### ROW_NUMBER, RANK, DENSE_RANK

```sql
SELECT name, department_id, salary,
       ROW_NUMBER() OVER (PARTITION BY department_id ORDER BY salary DESC) AS row_num,
       RANK()       OVER (PARTITION BY department_id ORDER BY salary DESC) AS rank,
       DENSE_RANK() OVER (PARTITION BY department_id ORDER BY salary DESC) AS dense_rank
FROM employees;
```

Farqi (teng qiymatlar bo'lganda):

| Funksiya | Teng qiymatlarga | Keyingi raqam |
|----------|------------------|---------------|
| `ROW_NUMBER` | har biriga har xil (1,2,3...) | ketma-ket |
| `RANK` | bir xil (1,1,3) | **sakraydi** (3 ga) |
| `DENSE_RANK` | bir xil (1,1,2) | **sakramaydi** |

### LAG va LEAD — oldingi/keyingi qatorga qarash

```sql
-- Har bir xodim maoshi va undan oldin yollangan xodim maoshi
SELECT name, hired_at, salary,
       LAG(salary)  OVER (ORDER BY hired_at) AS prev_salary,
       LEAD(salary) OVER (ORDER BY hired_at) AS next_salary
FROM employees;
```

`LAG(col, n, default)` — n qator oldinga; topilmasa `default`.

### Running total — SUM() OVER (...)

```sql
-- Yollanish tartibida maoshlarning yig'indisi (kumulyativ)
SELECT name, hired_at, salary,
       SUM(salary) OVER (ORDER BY hired_at
                         ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS running_total
FROM employees;
```

**💡 Tushuncha:** `ORDER BY` bo'lgan window'da default frame `RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW` bo'ladi — bu running total beradi. Frame'ni aniq yozish (`ROWS BETWEEN ...`) odatda xavfsizroq, chunki teng `ORDER BY` qiymatlarida `RANGE` bilan `ROWS` natijasi farq qiladi.

### PARTITION BY — guruhlarga bo'lish

`PARTITION BY` window'ni guruhlarga bo'ladi; har guruh ichida hisob qaytadan boshlanadi.

```sql
-- Har bir xodim maoshining o'z bo'limidagi ulushi (%)
SELECT name, department_id, salary,
       ROUND(100.0 * salary / SUM(salary) OVER (PARTITION BY department_id), 1) AS pct_of_dept
FROM employees;
```

**⚠️ Ehtiyot bo'l:** Window function'ni `WHERE`da ishlatib bo'lmaydi (chunki u `WHERE`dan keyin hisoblanadi). Window natijasi bo'yicha filtrlash uchun uni CTE/subquery'ga o'rab, tashqarida `WHERE` qo'ying. Masalan: `WHERE row_num = 1`.

---

## Set operations

**💡 Tushuncha:** Set operation ikki so'rov natijasini to'plam sifatida birlashtiradi. Shart: ikkala so'rovda **ustunlar soni va tiplari mos** bo'lishi kerak.

```sql
-- UNION — birlashtirish + duplikatlarni olib tashlash (sekinroq)
SELECT name FROM employees WHERE department_id = 1
UNION
SELECT name FROM employees WHERE salary > 90000;

-- UNION ALL — birlashtirish, duplikatlarni qoldirib (tezroq)
SELECT name FROM employees WHERE department_id = 1
UNION ALL
SELECT name FROM employees WHERE salary > 90000;

-- INTERSECT — ikkalasida ham bor qatorlar
SELECT name FROM employees WHERE department_id = 1
INTERSECT
SELECT name FROM employees WHERE salary > 90000;

-- EXCEPT — birinchisida bor, ikkinchisida yo'q (MINUS Oracle'da)
SELECT name FROM employees WHERE department_id = 1
EXCEPT
SELECT name FROM employees WHERE salary > 90000;
```

| Operatsiya | Natija | Duplikat |
|------------|--------|----------|
| `UNION` | A va B birlashmasi | olib tashlaydi (distinct) |
| `UNION ALL` | A va B birlashmasi | qoldiradi |
| `INTERSECT` | A va B kesishmasi | olib tashlaydi |
| `EXCEPT` | A − B ayirmasi | olib tashlaydi |

**⚠️ Ehtiyot bo'l:** Agar duplikat bo'lmasligini bilsangiz yoki muhim bo'lmasa, `UNION ALL` ishlating — `UNION` qo'shimcha sort/dedup qiladi va sekinroq.

---

## CASE expression

**💡 Tushuncha:** `CASE` — SQL'ning if/else'i. Ikki ko'rinishi bor: **searched** (har bir shart alohida) va **simple** (bitta ifodani qiymatlarga solishtirish).

```sql
-- Searched CASE
SELECT name, salary,
       CASE
           WHEN salary >= 100000 THEN 'High'
           WHEN salary >= 80000  THEN 'Medium'
           ELSE 'Low'
       END AS salary_band
FROM employees;
```

`CASE`ni aggregate ichida ishlatish — "conditional aggregation" (pivot uchun ham asosiy usul):

```sql
SELECT department_id,
       COUNT(*) FILTER (WHERE salary > 90000) AS high_earners,  -- Postgres FILTER
       SUM(CASE WHEN salary > 90000 THEN 1 ELSE 0 END) AS high_earners_classic
FROM employees
GROUP BY department_id;
```

**💡 Tushuncha:** PostgreSQL'da `COUNT(*) FILTER (WHERE ...)` — `SUM(CASE ...)`ning toza, o'qiladigan muqobili. Lekin `CASE` standart SQL bo'lgani uchun hamma joyda ishlaydi.

---

## View va Materialized View

**💡 Tushuncha:** **View** — saqlangan so'rov, "virtual jadval". Unda ma'lumot saqlanmaydi; har murojaatda asosiy so'rov qayta bajariladi. **Materialized view** — natijani fizik saqlaydi (disk'da), shuning uchun o'qish tez, lekin ma'lumot eskirishi (stale) mumkin.

```sql
-- Oddiy view
CREATE VIEW high_earners AS
SELECT name, department_id, salary
FROM employees
WHERE salary > 90000;

SELECT * FROM high_earners;   -- har safar asosiy so'rov ishlaydi
```

```sql
-- Materialized view
CREATE MATERIALIZED VIEW dept_salary_summary AS
SELECT department_id, COUNT(*) AS cnt, AVG(salary) AS avg_salary
FROM employees
GROUP BY department_id;

-- Ma'lumot o'zgargach, qo'lda yangilash kerak:
REFRESH MATERIALIZED VIEW dept_salary_summary;
-- O'qishni bloklamasdan yangilash (unique index kerak):
REFRESH MATERIALIZED VIEW CONCURRENTLY dept_salary_summary;
```

| Xususiyat | View | Materialized View |
|-----------|------|-------------------|
| Ma'lumot saqlash | Yo'q (virtual) | Ha (fizik) |
| O'qish tezligi | Asosiy so'rovga teng | Tez |
| Yangilik (freshness) | Doim aktual | Refresh'gacha eski |
| Yangilash | Avtomatik | `REFRESH` qo'lda/cron |

**⚠️ Ehtiyot bo'l:** Materialized view'ni ko'p o'qiladigan, kam o'zgaradigan og'ir hisob-kitoblar (dashboard, report) uchun ishlating. Tez-tez o'zgaradigan ma'lumotda `REFRESH` xarajati foydadan ko'p bo'lishi mumkin.

---

## Stored procedure, function, trigger

**💡 Tushuncha:** Bular ma'lumotlar bazasi ichida saqlanadigan kodlardir. **Function** qiymat qaytaradi va so'rov ichida ishlatiladi. **Procedure** (PostgreSQL 11+) transaction boshqarishi mumkin (`COMMIT`/`ROLLBACK`), `CALL` bilan chaqiriladi. **Trigger** — biror event (`INSERT`/`UPDATE`/`DELETE`) sodir bo'lganda avtomatik ishga tushadigan kod.

### Function

```sql
CREATE OR REPLACE FUNCTION dept_employee_count(dept INT)
RETURNS INT AS $$
    SELECT COUNT(*)::INT FROM employees WHERE department_id = dept;
$$ LANGUAGE sql;

SELECT dept_employee_count(1);
```

PL/pgSQL bilan mantiqli function:

```sql
CREATE OR REPLACE FUNCTION give_raise(emp_id INT, pct NUMERIC)
RETURNS NUMERIC AS $$
DECLARE
    new_salary NUMERIC;
BEGIN
    UPDATE employees
    SET salary = salary * (1 + pct / 100)
    WHERE id = emp_id
    RETURNING salary INTO new_salary;
    RETURN new_salary;
END;
$$ LANGUAGE plpgsql;
```

### Procedure

```sql
CREATE PROCEDURE archive_old_employees(cutoff DATE)
LANGUAGE plpgsql AS $$
BEGIN
    DELETE FROM employees WHERE hired_at < cutoff;
    COMMIT;   -- procedure transaction boshqara oladi
END;
$$;

CALL archive_old_employees('2018-01-01');
```

### Trigger

```sql
-- Audit jadvali
CREATE TABLE salary_audit (
    emp_id     INT,
    old_salary NUMERIC,
    new_salary NUMERIC,
    changed_at TIMESTAMP DEFAULT now()
);

-- Trigger function
CREATE OR REPLACE FUNCTION log_salary_change()
RETURNS TRIGGER AS $$
BEGIN
    IF NEW.salary <> OLD.salary THEN
        INSERT INTO salary_audit(emp_id, old_salary, new_salary)
        VALUES (OLD.id, OLD.salary, NEW.salary);
    END IF;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- Trigger'ni jadvalga ulash
CREATE TRIGGER trg_salary_change
AFTER UPDATE ON employees
FOR EACH ROW
EXECUTE FUNCTION log_salary_change();
```

**⚠️ Ehtiyot bo'l:** Trigger'lar "yashirin" mantiq — ular debug qilishni qiyinlashtiradi, chunki kodda ko'rinmaydi. Biznes-mantiqni application qatlamida saqlash ko'pincha tushunarliroq. Trigger'larni audit, denormalizatsiyani sinxronlash kabi cheklangan vazifalarga ishlating.

---

## Pivot

**💡 Tushuncha:** Pivot — qatorlarni ustunga aylantirish (long → wide format). PostgreSQL'da standart `PIVOT` operatori yo'q; uni **conditional aggregation** (`CASE` yoki `FILTER`) bilan qilamiz.

```sql
-- Har bir bo'lim uchun maosh darajalari bo'yicha sanoq
SELECT department_id,
       COUNT(*) FILTER (WHERE salary >= 100000) AS high,
       COUNT(*) FILTER (WHERE salary BETWEEN 80000 AND 99999) AS medium,
       COUNT(*) FILTER (WHERE salary < 80000) AS low
FROM employees
GROUP BY department_id
ORDER BY department_id;
```

`CASE` bilan ekvivalenti:

```sql
SELECT department_id,
       SUM(CASE WHEN salary >= 100000 THEN 1 ELSE 0 END) AS high,
       SUM(CASE WHEN salary < 80000 THEN 1 ELSE 0 END)   AS low
FROM employees
GROUP BY department_id;
```

**💡 Tushuncha:** PostgreSQL'da `tablefunc` extension'idagi `crosstab()` funksiyasi ham bor, lekin u qo'shimcha o'rnatish va aniq tip e'lon talab qiladi. Intervyuda conditional aggregation'ni ko'rsatish yetarli va ko'chma (portable) yechim.

---

## Klassik intervyu query patternlari

Bular intervyularda eng ko'p so'raladigan namunalar. Har birining **g'oyasini** tushunib oling — savol shakli o'zgarsa ham yondashuvi bir xil qoladi.

### 1. Ikkinchi eng katta maosh

```sql
-- a) Window function bilan (tavsiya etiladi, takrorlarni boshqaradi)
SELECT DISTINCT salary
FROM (
    SELECT salary, DENSE_RANK() OVER (ORDER BY salary DESC) AS rnk
    FROM employees
) t
WHERE rnk = 2;

-- b) OFFSET bilan (lekin teng maoshlarni hisobga olmaydi)
SELECT DISTINCT salary FROM employees ORDER BY salary DESC OFFSET 1 LIMIT 1;
```

**💡 Tushuncha:** `DENSE_RANK` ishlatish — bir nechta odam bir xil eng katta maoshda bo'lsa ham "ikkinchi"ni to'g'ri topadi. "N-chi eng katta" uchun `WHERE rnk = N`.

### 2. Duplikatlarni topish

```sql
-- Bir xil ism + bo'limga ega takror yozuvlar
SELECT name, department_id, COUNT(*) AS cnt
FROM employees
GROUP BY name, department_id
HAVING COUNT(*) > 1;
```

### 3. Har guruhdan top N

```sql
-- Har bir bo'limdan eng yuqori maoshli 2 xodim
SELECT *
FROM (
    SELECT name, department_id, salary,
           ROW_NUMBER() OVER (PARTITION BY department_id ORDER BY salary DESC) AS rn
    FROM employees
) t
WHERE rn <= 2;
```

### 4. Running total (kumulyativ yig'indi)

```sql
SELECT hired_at, salary,
       SUM(salary) OVER (ORDER BY hired_at
                         ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS running_total
FROM employees;
```

### 5. Gaps and Islands

**💡 Tushuncha:** "Gaps and islands" — ketma-ket guruhlarni (islands) va bo'shliqlarni (gaps) topish. Klassik usul: qator raqami bilan qiymat farqini hisoblash. Agar qiymatlar ketma-ket bo'lsa, `value − ROW_NUMBER()` o'zgarmas qoladi va shu o'zgarmas qiymat "orol"ni belgilaydi.

```sql
-- Misol jadval: login kunlari
CREATE TABLE logins (user_id INT, login_date DATE);
INSERT INTO logins VALUES
    (1,'2024-01-01'),(1,'2024-01-02'),(1,'2024-01-03'),
    (1,'2024-01-05'),(1,'2024-01-06');

-- Ketma-ket kunlardan iborat "orollar"
WITH grouped AS (
    SELECT user_id, login_date,
           login_date - (ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY login_date))::INT AS grp
    FROM logins
)
SELECT user_id,
       MIN(login_date) AS streak_start,
       MAX(login_date) AS streak_end,
       COUNT(*)        AS streak_len
FROM grouped
GROUP BY user_id, grp
ORDER BY streak_start;
```

---

## Savol-javoblar

### ❓ Subquery va JOIN orasidagi farq nima? Qaysi biri tezroq?

**✅ Javob:** JOIN ikki jadval qatorlarini gorizontal birlashtiradi va natijada ikkala jadvaldan ustunlar bo'ladi. Subquery — bir so'rov natijasini boshqasida ishlatish; ko'pincha filtrlash yoki bitta qiymat olish uchun. Tezlik bo'yicha — zamonaviy optimizatorlar ko'p hollarda ikkalasini bir xil planga aylantiradi, shuning uchun universal javob yo'q. Lekin **correlated subquery** har qator uchun qaytadan ishlashi mumkin va u yerda JOIN odatda tezroq. Eng to'g'ri yo'l — `EXPLAIN ANALYZE` bilan tekshirish.

### ❓ IN va EXISTS qachon farq qiladi?

**✅ Javob:** Kichik, statik ro'yxat uchun `IN` qulay. Katta yoki correlated subquery uchun `EXISTS` ko'pincha tezroq, chunki u birinchi mos qatorni topishi bilan to'xtaydi. Eng muhim farq — `NULL`: agar subquery natijasida `NULL` bo'lsa, `NOT IN` **hech qanday qator qaytarmaydi** (3-valued logic tufayli), `NOT EXISTS` esa to'g'ri ishlaydi. Shu sabab `NOT EXISTS` xavfsizroq tanlov.

### ❓ CTE va subquery orasida qaysi birini tanlash kerak?

**✅ Javob:** Natija bir xil bo'lishi mumkin — bu asosan o'qilishi masalasi. CTE murakkab so'rovni bosqichlarga ajratadi va bir necha joyda qayta ishlatishga imkon beradi; recursive ish faqat CTE bilan mumkin. Subquery oddiy, bir martalik holatlarda ixcham. PostgreSQL 12+ da CTE'lar inline qilingani uchun ilgarigi "CTE sekinroq" deyish endi to'g'ri emas.

### ❓ Window function bilan GROUP BY orasidagi asosiy farq nima?

**✅ Javob:** `GROUP BY` qatorlarni guruhga **birlashtiradi** — natijada har guruhga bitta qator qoladi. Window function esa har bir qatorni **saqlab qoladi**, faqat yoniga hisoblangan ustun qo'shadi. Shuning uchun "har bir xodimni va uning bo'limidagi o'rtacha maoshni bir qatorda" ko'rsatish — window function ishi.

### ❓ ROW_NUMBER, RANK va DENSE_RANK farqi?

**✅ Javob:** Uchalasi tartib raqami beradi, farq teng qiymatlarda: `ROW_NUMBER` hatto teng qatorlarga ham har xil raqam beradi (1,2,3); `RANK` teng qatorlarga bir xil raqam beradi va keyingisini sakratadi (1,1,3); `DENSE_RANK` teng qatorlarga bir xil beradi, lekin sakramaydi (1,1,2). "N-chi eng katta"ni topishda odatda `DENSE_RANK` to'g'ri keladi.

### ❓ Window function'ni WHERE'da nega ishlatib bo'lmaydi?

**✅ Javob:** SQL'ning mantiqiy bajarilish tartibida `WHERE` window function'dan **oldin** ishlaydi, shuning uchun u paytda window natijasi hali yo'q. Yechim — so'rovni CTE yoki subquery'ga o'rab, window natijasi bo'yicha tashqarida `WHERE` qo'yish (masalan `WHERE rn = 1`).

### ❓ UNION va UNION ALL farqi va qaysi biri tezroq?

**✅ Javob:** `UNION` natijalardan duplikatlarni olib tashlaydi — buning uchun sort/hash bilan dedup qiladi, ya'ni qo'shimcha ish. `UNION ALL` esa shunchaki ikkala natijani qo'shib yuboradi, duplikat tekshirmaydi — shuning uchun **tezroq**. Agar duplikat bo'lmasligini bilsangiz yoki muhim bo'lmasa, doim `UNION ALL` ishlating.

### ❓ View qanday foyda beradi? U tezlikni oshiradimi?

**✅ Javob:** View — saqlangan so'rov; u (1) murakkab so'rovni soddalashtirib qayta ishlatishga, (2) ruxsatni cheklashga (faqat ma'lum ustunlarni ko'rsatish), (3) abstraktsiya berishga yordam beradi. Lekin oddiy view **tezlikni oshirmaydi** — har murojaatda asosiy so'rov qaytadan ishlaydi. Tezlik kerak bo'lsa, materialized view ishlating.

### ❓ Materialized view qachon kerak va uning kamchiligi nima?

**✅ Javob:** Og'ir, kam o'zgaradigan, ko'p o'qiladigan hisob-kitoblar (dashboard, hisobot, aggregatsiya) uchun. Natija fizik saqlangani uchun o'qish tez. Kamchiligi — ma'lumot **eskirishi** (stale) mumkin; aktual qilish uchun `REFRESH MATERIALIZED VIEW` kerak, bu esa xarajat. Tez-tez o'zgaradigan ma'lumotga mos kelmaydi.

### ❓ Trigger nima va uni qachon ishlatmaslik kerak?

**✅ Javob:** Trigger — jadvalda `INSERT`/`UPDATE`/`DELETE` sodir bo'lganda avtomatik ishga tushadigan kod. Audit log yozish, denormalizatsiyani sinxronlash, validatsiya uchun foydali. Lekin u "yashirin mantiq" — kodni o'qiganda ko'rinmaydi, debug qiyinlashadi, ko'rinmas zanjirli ta'sirlar bo'lishi mumkin. Murakkab biznes-mantiqni application qatlamida saqlash odatda tushunarliroq.

### ❓ Correlated subquery nima va nega sekin bo'lishi mumkin?

**✅ Javob:** Correlated subquery — tashqi so'rovning ustuniga murojaat qiladigan ichki so'rov. U tashqi so'rovning **har bir qatori uchun qaytadan** bajariladi, shuning uchun N ta qatorda N marta ishlashi mumkin (O(N×M)). Ko'pincha uni `JOIN` yoki window function bilan almashtirib, bir martalik skanga aylantirish kerak.

### ❓ Recursive CTE qanday ishlaydi?

**✅ Javob:** U ikki qismdan iborat: **anchor** so'rov boshlang'ich qatorlarni beradi; **recursive** qism (`UNION ALL` orqali) oldingi qadam natijasiga murojaat qilib, yangi qatorlar qo'shadi — yangi qator qo'shilmaguncha takrorlanadi. Ierarxiya (org chart), graf yurish, ketma-ketlik generatsiya qilish uchun ishlatiladi. To'xtash sharti bo'lmasa cheksiz loop bo'ladi.

### ❓ "Har guruhdan top N" qatorni qanday olasiz?

**✅ Javob:** `ROW_NUMBER() OVER (PARTITION BY guruh ORDER BY mezon DESC)` bilan har guruh ichida raqamlash, keyin tashqi so'rovda `WHERE rn <= N`. Teng qiymatlarni ham qo'shish kerak bo'lsa, `ROW_NUMBER` o'rniga `RANK`/`DENSE_RANK` ishlatish mumkin.

### ❓ Running total'ni qanday hisoblaysiz?

**✅ Javob:** `SUM(col) OVER (ORDER BY tartib ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW)`. `ORDER BY` window'ni qanday tartibda yig'ishni belgilaydi; frame `UNBOUNDED PRECEDING` dan `CURRENT ROW` gacha — bu kumulyativ yig'indi. `PARTITION BY` qo'shsangiz, har guruh uchun alohida running total.

### ❓ Gaps and islands masalasi qanday yechiladi?

**✅ Javob:** Asosiy hiyla — qiymatdan qator raqamini ayirish. Ketma-ket qiymatlar uchun `value − ROW_NUMBER()` o'zgarmas qoladi va shu o'zgarmas "orol"ni belgilaydi. Keyin shu farq bo'yicha `GROUP BY` qilib, har orolning boshi/oxiri/uzunligini topamiz. Bo'shliqlar uchun esa `LAG` bilan ketma-ket qatorlar orasidagi farqni tekshirish mumkin.

### ❓ EXCEPT va NOT IN/NOT EXISTS orasidagi farq?

**✅ Javob:** `EXCEPT` ikki **so'rov natijasi** o'rtasida to'plam ayirmasini topadi va duplikatlarni olib tashlaydi. `NOT IN`/`NOT EXISTS` esa qator darajasida filtrlaydi. `EXCEPT` butun qatorni solishtiradi; `NOT EXISTS` esa correlated shartni tekshiradi. `NULL` xavfsizligi bo'yicha `EXCEPT` va `NOT EXISTS` `NOT IN`dan ishonchliroq.

---

## Masalalar

> Yechimlar: [solutions/databases/03-sql-advanced.md](../solutions/databases/03-sql-advanced.md)

Quyidagi `employees` va `departments` (yuqoridagi namuna sxema) jadvallaridan foydalaning.

1. **Ikkinchi eng katta maosh.** `employees` jadvalidan ikkinchi eng katta maoshni `DENSE_RANK` yordamida toping. Bir nechta odam eng katta maoshda bo'lsa ham to'g'ri ishlasin.

2. **Har bo'limdan top-2.** Har bir bo'limdagi eng yuqori maoshli 2 xodimni chiqaring (ism, bo'lim, maosh va bo'lim ichidagi tartib raqami bilan).

3. **O'rtachadan yuqori.** Har bir xodimni faqat o'z bo'limining o'rtacha maoshidan ko'p oladigan bo'lsa chiqaring. Correlated subquery va window function — ikki xil yechim yozing.

4. **Duplikatlar.** Faraz qiling `employees` jadvalida bir xil `name` bilan bir nechta yozuv bo'lishi mumkin. Takrorlangan ismlarni va ularning sonini toping.

5. **Running total.** Xodimlarni `hired_at` bo'yicha tartiblab, maoshlarning kumulyativ yig'indisini (running total) hisoblang.

6. **Maosh ulushi.** Har bir xodim maoshi o'z bo'limidagi umumiy maosh fondining necha foizini tashkil qilishini ko'rsating (`PARTITION BY` window bilan).

7. **Org chart.** Recursive CTE yordamida Alice (id=1) ostidagi butun ierarxiyani level (chuqurlik) bilan birga chiqaring.

8. **Pivot.** Har bir bo'lim uchun maosh darajalari (`high` ≥ 100000, `medium` 80000–99999, `low` < 80000) bo'yicha xodimlar sonini ustunlarga ajratib (pivot) ko'rsating.

9. **Gaps and islands.** `logins(user_id, login_date)` jadvalida har bir foydalanuvchining ketma-ket kunlardan iborat eng uzun login "streak"ini (boshi, oxiri, uzunligi) toping.

10. **Set operation.** Engineering bo'limidagi xodimlar ismlari bilan maoshi 90000 dan yuqori xodimlar ismlari kesishmasini (`INTERSECT`) va ayirmasini (`EXCEPT`) chiqaring.

---

← [Database bo'limiga qaytish](./README.md)
