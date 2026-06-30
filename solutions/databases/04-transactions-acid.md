# Transactions & ACID — Masalalar yechimi

Bu yerda [`databases/04-transactions-acid.md`](../../databases/04-transactions-acid.md) bo'limidagi masalalarning yechimlari. Avval o'zingiz urinib ko'ring, keyin solishtiring. Yechimlar **PostgreSQL** uchun.

Eslatma — barcha yechimlar quyidagi jadvalga tayanadi:

```sql
CREATE TABLE accounts (
    id      INT PRIMARY KEY,
    owner   TEXT,
    balance NUMERIC CHECK (balance >= 0),
    version INT DEFAULT 0
);
INSERT INTO accounts(id, owner, balance) VALUES
    (1, 'Alice', 1000), (2, 'Bob', 500);
```

---

## 1. Pul o'tkazish (atomik)

```sql
BEGIN;
    UPDATE accounts SET balance = balance - 300 WHERE id = 1;
    UPDATE accounts SET balance = balance + 300 WHERE id = 2;
COMMIT;
```

**Izoh:** `CHECK (balance >= 0)` tufayli Alice balansi yetmasa, birinchi `UPDATE` xato beradi va transaction abort bo'ladi — bu holatda `COMMIT` o'rniga `ROLLBACK` ishlaydi, hech narsa o'zgarmaydi. Atomicity shu yerda: yo ikkala `UPDATE` birga saqlanadi, yo hech biri. Application tomonida xatoni tutib `ROLLBACK` chaqirish kerak.

---

## 2. Savepoint

```sql
BEGIN;
    UPDATE accounts SET balance = balance + 100 WHERE id = 1;   -- 1000 -> 1100

    SAVEPOINT sp1;
    UPDATE accounts SET balance = balance - 5000 WHERE id = 1;  -- CHECK buziladi, xato
    ROLLBACK TO SAVEPOINT sp1;   -- faqat shu xato UPDATE bekor, +100 saqlanadi

COMMIT;   -- yakuniy balans: 1100
```

**Izoh:** Xato yuz bergach, butun transaction "abort" holatiga o'tadi va keyingi buyruqlar rad etiladi. `ROLLBACK TO SAVEPOINT sp1` transaction'ni yana ishchi holatga qaytaradi, savepoint'dan keyingi o'zgarishlarni bekor qilib, undan oldingilarni (ya'ni +100 ni) saqlaydi.

---

## 3. Lost update va uni to'g'rilash

**Muammo (read-then-write):**

```
T1: SELECT balance FROM accounts WHERE id=1;   -- 1000 o'qidi
T2: SELECT balance FROM accounts WHERE id=1;   -- 1000 o'qidi
T1: UPDATE accounts SET balance = 1000 + 50;  COMMIT;   -- 1050
T2: UPDATE accounts SET balance = 1000 + 30;  COMMIT;   -- 1030  (T1 ning +50 yo'qoldi!)
```

**To'g'rilangan (atomik UPDATE):**

```sql
-- Har bir transaction o'qimasdan, to'g'ridan-to'g'ri o'zgartiradi
UPDATE accounts SET balance = balance + 50 WHERE id = 1;
UPDATE accounts SET balance = balance + 30 WHERE id = 1;
```

**Izoh:** `balance = balance + N` qiymatni o'qish-yozishni bitta atomik operatsiyaga aylantiradi. Ikkinchi `UPDATE` row lock'ni kutadi, keyin **yangilangan** qiymat ustiga qo'shadi — natija to'g'ri (1080), hech narsa yo'qolmaydi.

---

## 4. Pessimistic lock (FOR UPDATE)

```sql
BEGIN;
    SELECT balance FROM accounts WHERE id = 1 FOR UPDATE;   -- exclusive row lock
    -- application: hisob-kitob qiladi (masalan yangi balans = balance - 200)
    UPDATE accounts SET balance = balance - 200 WHERE id = 1;
COMMIT;   -- lock ozod bo'ladi
```

**Izoh:** `FOR UPDATE` qatorga exclusive lock qo'yadi. Boshqa transaction xuddi shu qatorni `FOR UPDATE` qilmoqchi bo'lsa, bu transaction `COMMIT`/`ROLLBACK` qilguncha **kutadi**. Shu sababli ikki transaction bir vaqtda o'qib-o'zgartira olmaydi — lost update bo'lmaydi (ular ketma-ket bajariladi).

---

## 5. Optimistic lock (version)

```sql
-- 1-qadam: o'qish (version ham olinadi)
SELECT balance, version FROM accounts WHERE id = 1;   -- masalan balance=1000, version=0

-- 2-qadam: yangilash, faqat version o'zgarmagan bo'lsa
UPDATE accounts
SET balance = 1000 - 200,
    version = version + 1
WHERE id = 1 AND version = 0;
```

**Izoh:** Agar oraliqda boshqa kim balansni yangilagan bo'lsa, `version` 0 emas (1 bo'lib qoladi), shuning uchun `WHERE`ga mos qator yo'q — `UPDATE` **0 qatorga** ta'sir qiladi. Bu konflikt belgisi: application yangilanish bo'lmaganini aniqlab, ma'lumotni qayta o'qib **retry** qiladi. Hech qanday lock ushlanmaydi, deadlock bo'lmaydi.

---

## 6. Isolation jadvali

| Isolation level | Dirty read | Non-repeatable read | Phantom read |
|-----------------|:----------:|:-------------------:|:------------:|
| READ UNCOMMITTED | mumkin | mumkin | mumkin |
| READ COMMITTED | yo'q | mumkin | mumkin |
| REPEATABLE READ | yo'q | yo'q | mumkin (standartda) |
| SERIALIZABLE | yo'q | yo'q | yo'q |

**PostgreSQL farqi:**
- `READ UNCOMMITTED` Postgres'da aslida `READ COMMITTED` kabi ishlaydi — dirty read **hech qachon bo'lmaydi**.
- Postgres'ning `REPEATABLE READ`i (snapshot isolation) phantom read'ni **ham** oldini oladi (standart talab qilmasa-da).
- `SERIALIZABLE` esa "write skew" anomaliyasini ham oldini oladi va kerak bo'lsa serialization xatosi berib retry talab qiladi.
- Default daraja — `READ COMMITTED`.

---

## 7. Deadlock va uni tuzatish

**Deadlock yaratuvchi ssenariy (teskari tartib):**

```
T1: BEGIN;
T1: UPDATE accounts SET balance = balance - 10 WHERE id = 1;   -- row 1 lock

T2: BEGIN;
T2: UPDATE accounts SET balance = balance - 10 WHERE id = 2;   -- row 2 lock

T1: UPDATE accounts SET balance = balance + 10 WHERE id = 2;   -- row 2 ni kutadi (T2 ushlagan)
T2: UPDATE accounts SET balance = balance + 10 WHERE id = 1;   -- row 1 ni kutadi (T1 ushlagan)
-- => DEADLOCK; Postgres birini rollback qiladi
```

**Tuzatilgan (lock'ni doim id tartibida olish):**

```
-- Ikkala transaction ham AVVAL kichik id ni, KEYIN katta id ni qulflaydi
T1: UPDATE accounts SET balance = balance - 10 WHERE id = 1;
T1: UPDATE accounts SET balance = balance + 10 WHERE id = 2;
T2: UPDATE accounts SET balance = balance + 10 WHERE id = 1;
T2: UPDATE accounts SET balance = balance - 10 WHERE id = 2;
```

**Izoh:** Lock'larni har doim bir xil tartibda (masalan, doim kichik `id`dan kattasiga) olsak, ikki transaction bir-birini "halqa" qilib kutmaydi — deadlock yo'qoladi. Bitta transaction ikkinchisini kutadi, keyin davom etadi.

---

## 8. Non-repeatable read

**READ COMMITTED da sodir bo'ladi:**

```
T1: BEGIN; SET TRANSACTION ISOLATION LEVEL READ COMMITTED;
T1: SELECT balance FROM accounts WHERE id = 1;   -- 1000

T2: BEGIN;
T2: UPDATE accounts SET balance = 2000 WHERE id = 1;
T2: COMMIT;

T1: SELECT balance FROM accounts WHERE id = 1;   -- 2000  ← bir transaction ichida o'zgardi!
T1: COMMIT;
```

**REPEATABLE READ uni yo'qotadi:**

```
T1: BEGIN; SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;
T1: SELECT balance FROM accounts WHERE id = 1;   -- 1000

T2: UPDATE accounts SET balance = 2000 WHERE id = 1;  COMMIT;

T1: SELECT balance FROM accounts WHERE id = 1;   -- HALI HAM 1000
T1: COMMIT;
```

**Izoh:** `READ COMMITTED` da har bir statement eng so'nggi commit qilingan ma'lumotni ko'radi, shuning uchun ikki `SELECT` orasida qiymat o'zgaradi. `REPEATABLE READ` da transaction boshida olingan **snapshot** butun transaction davomida saqlanadi — T2 commit qilsa ham, T1 doim 1000 ni ko'radi. Postgres buni MVCC snapshot orqali amalga oshiradi.

---

← [Database bo'limiga qaytish](../../databases/README.md)
