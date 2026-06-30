# 02 — SQL asoslari: Masalalar yechimi

Bu fayl [`databases/02-sql-basics.md`](../../databases/02-sql-basics.md) dagi **Masalalar** bo'limining yechimlarini o'z ichiga oladi. Avval o'zingiz urinib ko'ring, keyin solishtiring.

Jadvallar: `employees(id, name, dept_id, salary, hired_at)`, `departments(id, name)`.

---

## 1-masala. SELECT + WHERE + ORDER

```sql
SELECT name, salary
FROM employees
WHERE salary > 5000
ORDER BY salary DESC;
```

**Izoh:** `WHERE` qatorlarni filter qiladi, `ORDER BY ... DESC` kamayish tartibida tartiblaydi.

---

## 2-masala. DISTINCT + IN

```sql
SELECT DISTINCT dept_id
FROM employees
WHERE dept_id IN (1, 2);
```

**Izoh:** `IN (1, 2)` — id 1 yoki 2; `DISTINCT` takrorlanishlarni olib tashlaydi.

---

## 3-masala. Aggregate

```sql
SELECT
    COUNT(*)      AS total_employees,
    SUM(salary)   AS total_salary,
    AVG(salary)   AS avg_salary,
    MAX(salary)   AS max_salary
FROM employees;
```

**Izoh:** `GROUP BY`siz aggregate butun jadvalni bitta guruh deb hisoblaydi. `AVG` va `SUM` NULL maoshlarni e'tiborsiz qoldiradi.

---

## 4-masala. GROUP BY + HAVING

```sql
SELECT dept_id, COUNT(*) AS cnt
FROM employees
GROUP BY dept_id
HAVING COUNT(*) > 1;
```

**Izoh:** Avval guruhlaymiz, keyin `HAVING` bilan guruhlarni (aggregate natijasiga ko'ra) filter qilamiz. `WHERE COUNT(*) > 1` bu yerda xato bo'lardi.

---

## 5-masala. WHERE vs HAVING

```sql
SELECT dept_id, AVG(salary) AS avg_salary
FROM employees
WHERE hired_at >= '2020-01-01'   -- qatorlarni filter (guruhlashdan oldin)
GROUP BY dept_id
HAVING AVG(salary) > 5000;        -- guruhlarni filter (guruhlashdan keyin)
```

**Izoh:** `WHERE` alohida qatorlarni (ishga olingan sana bo'yicha) kesadi, `HAVING` esa hisoblangan o'rtacha maoshga ko'ra guruhlarni filter qiladi. Bu ikkalasini birga ishlatishning klassik misoli.

---

## 6-masala. LEFT JOIN

```sql
SELECT e.name, d.name AS dept_name
FROM employees e
LEFT JOIN departments d ON e.dept_id = d.id;
```

**Izoh:** `LEFT JOIN` barcha xodimlarni saqlaydi. `dept_id IS NULL` bo'lgan xodim (masalan Olim) ham chiqadi, uning `dept_name` ustuni NULL bo'ladi. INNER JOIN bo'lganda u tushib qolardi.

---

## 7-masala. INNER JOIN + filter

```sql
SELECT e.name, e.salary
FROM employees e
INNER JOIN departments d ON e.dept_id = d.id
WHERE d.name = 'Sales';
```

**Izoh:** Bo'lim nomi `departments` jadvalida bo'lgani uchun JOIN kerak. `d.name = 'Sales'` filteri INNER JOIN bilan to'g'ri ishlaydi.

---

## 8-masala. NULL bilan ishlash

```sql
SELECT name, COALESCE(salary, 0) AS salary
FROM employees;
```

**Izoh:** `COALESCE(salary, 0)` — `salary` NULL bo'lsa 0, aks holda asl qiymat. Ustun alias `salary` deb nomlandi.

---

## 9-masala. Pagination

```sql
SELECT *
FROM employees
ORDER BY id
LIMIT 2 OFFSET 2;
```

**Izoh:** `OFFSET 2` birinchi 2 qatorni o'tkazadi, `LIMIT 2` keyingi 2 tasini oladi — ya'ni 3- va 4-qatorlar (2-sahifa). `ORDER BY`siz natija beqaror bo'lardi.

---

## 10-masala. SELF JOIN

```sql
SELECT e.name AS employee, m.name AS manager
FROM employees e
LEFT JOIN employees m ON e.manager_id = m.id;
```

**Izoh:** Jadval o'zi bilan ikki alias (`e` — xodim, `m` — manager) orqali bog'lanadi. `LEFT JOIN` ishlatildi, shunda manageri yo'q xodimlar (`manager_id IS NULL`, masalan eng yuqori rahbar) ham chiqadi, ularning `manager` ustuni NULL bo'ladi. Agar INNER JOIN ishlatilsa, manageri yo'qlar tushib qolardi.

---

← [Database bo'limiga qaytish](../../databases/README.md)
