# SQL Advanced — Masalalar yechimi

Bu yerda [`databases/03-sql-advanced.md`](../../databases/03-sql-advanced.md) bo'limidagi masalalarning yechimlari. Avval o'zingiz urinib ko'ring, keyin solishtiring. Yechimlar **PostgreSQL** uchun.

---

## 1. Ikkinchi eng katta maosh

`DENSE_RANK` bir nechta odam eng katta maoshda bo'lsa ham "ikkinchi"ni to'g'ri topadi.

```sql
SELECT DISTINCT salary
FROM (
    SELECT salary, DENSE_RANK() OVER (ORDER BY salary DESC) AS rnk
    FROM employees
) t
WHERE rnk = 2;
```

**Izoh:** Agar eng katta maosh bir nechta xodimda bo'lsa, ular `rnk = 1` bo'ladi; `rnk = 2` esa undan keyingi farqli maoshni beradi. "N-chi eng katta" uchun `WHERE rnk = N`.

---

## 2. Har bo'limdan top-2

```sql
SELECT name, department_id, salary, rn
FROM (
    SELECT name, department_id, salary,
           ROW_NUMBER() OVER (PARTITION BY department_id ORDER BY salary DESC) AS rn
    FROM employees
) t
WHERE rn <= 2
ORDER BY department_id, rn;
```

**Izoh:** Window function'ni `WHERE`da ishlatib bo'lmaydi, shuning uchun subquery'ga o'rab tashqarida filtrlaymiz. Teng maoshlarni ham qo'shmoqchi bo'lsangiz, `ROW_NUMBER` o'rniga `RANK` ishlating.

---

## 3. O'rtachadan yuqori (ikki xil yechim)

**a) Correlated subquery:**

```sql
SELECT e.name, e.salary
FROM employees e
WHERE e.salary > (
    SELECT AVG(salary)
    FROM employees e2
    WHERE e2.department_id = e.department_id
);
```

**b) Window function (tezroq, bir martalik skan):**

```sql
SELECT name, salary
FROM (
    SELECT name, salary,
           AVG(salary) OVER (PARTITION BY department_id) AS dept_avg
    FROM employees
) t
WHERE salary > dept_avg;
```

**Izoh:** Window varianti har bir qator uchun subquery'ni qaytadan ishlatmaydi, shuning uchun katta jadvalda samaraliroq.

---

## 4. Duplikatlar

```sql
SELECT name, COUNT(*) AS cnt
FROM employees
GROUP BY name
HAVING COUNT(*) > 1;
```

**Izoh:** Bir nechta ustun bo'yicha takrorni topish kerak bo'lsa, ularning hammasini `GROUP BY`ga qo'shing (masalan `GROUP BY name, department_id`).

---

## 5. Running total

```sql
SELECT name, hired_at, salary,
       SUM(salary) OVER (ORDER BY hired_at
                         ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS running_total
FROM employees
ORDER BY hired_at;
```

**Izoh:** Frame'ni aniq `ROWS BETWEEN ...` bilan yozish — teng `hired_at` qiymatlarida `RANGE` default'idan farqli (to'g'riroq) natija beradi.

---

## 6. Maosh ulushi

```sql
SELECT name, department_id, salary,
       ROUND(100.0 * salary / SUM(salary) OVER (PARTITION BY department_id), 2) AS pct_of_dept
FROM employees
ORDER BY department_id, pct_of_dept DESC;
```

**Izoh:** `SUM(salary) OVER (PARTITION BY department_id)` har bir qatorga o'z bo'limining umumiy maosh fondini beradi; uni qatorga bo'lib foiz olamiz.

---

## 7. Org chart (recursive CTE)

```sql
WITH RECURSIVE org_chart AS (
    SELECT id, name, manager_id, 1 AS level
    FROM employees
    WHERE id = 1

    UNION ALL

    SELECT e.id, e.name, e.manager_id, oc.level + 1
    FROM employees e
    JOIN org_chart oc ON e.manager_id = oc.id
)
SELECT id, name, manager_id, level
FROM org_chart
ORDER BY level, id;
```

**Izoh:** Anchor — Alice (id=1, level 1). Recursive qism har qadamda bir pog'ona pastdagi xodimlarni qo'shadi va `level`ni oshiradi.

---

## 8. Pivot

```sql
SELECT department_id,
       COUNT(*) FILTER (WHERE salary >= 100000)               AS high,
       COUNT(*) FILTER (WHERE salary BETWEEN 80000 AND 99999) AS medium,
       COUNT(*) FILTER (WHERE salary < 80000)                 AS low
FROM employees
GROUP BY department_id
ORDER BY department_id;
```

`CASE` bilan portable varianti:

```sql
SELECT department_id,
       SUM(CASE WHEN salary >= 100000 THEN 1 ELSE 0 END)               AS high,
       SUM(CASE WHEN salary BETWEEN 80000 AND 99999 THEN 1 ELSE 0 END) AS medium,
       SUM(CASE WHEN salary < 80000 THEN 1 ELSE 0 END)                 AS low
FROM employees
GROUP BY department_id
ORDER BY department_id;
```

---

## 9. Gaps and islands (login streak)

```sql
WITH grouped AS (
    SELECT user_id, login_date,
           login_date - (ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY login_date))::INT AS grp
    FROM logins
),
streaks AS (
    SELECT user_id, grp,
           MIN(login_date) AS streak_start,
           MAX(login_date) AS streak_end,
           COUNT(*)        AS streak_len
    FROM grouped
    GROUP BY user_id, grp
)
SELECT DISTINCT ON (user_id)
       user_id, streak_start, streak_end, streak_len
FROM streaks
ORDER BY user_id, streak_len DESC, streak_start;
```

**Izoh:** Ketma-ket kunlar uchun `login_date − ROW_NUMBER()` o'zgarmas qoladi (chunki ikkalasi ham har qadamda 1 ga ortadi) — shu o'zgarmas qiymat "orol"ni belgilaydi. `DISTINCT ON (user_id)` har foydalanuvchi uchun eng uzun streak'ni oladi.

---

## 10. Set operation (INTERSECT va EXCEPT)

```sql
-- Kesishma: Engineering'da VA maoshi > 90000 bo'lganlar
SELECT name FROM employees WHERE department_id = 1
INTERSECT
SELECT name FROM employees WHERE salary > 90000;

-- Ayirma: Engineering'da bor, lekin maoshi > 90000 emaslar
SELECT name FROM employees WHERE department_id = 1
EXCEPT
SELECT name FROM employees WHERE salary > 90000;
```

**Izoh:** `INTERSECT` va `EXCEPT` ikkalasi ham duplikatlarni avtomatik olib tashlaydi. Ustunlar soni va tiplari ikki so'rovda mos bo'lishi shart.

---

← [Database bo'limiga qaytish](../../databases/README.md)
