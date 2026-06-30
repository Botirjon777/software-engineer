# Transactions & ACID

Bu bo'lim ma'lumotlar bazasidagi eng muhim tushunchalardan birini — **transaction** va uning kafolatlari **ACID**ni yoritadi. Bir nechta foydalanuvchi bir vaqtda bir ma'lumotni o'zgartirsa nima bo'ladi? Pul o'tkazish yarmida server o'chsa-chi? Mana shu savollarga javob shu yerda: transaction asoslari, `BEGIN/COMMIT/ROLLBACK`, `SAVEPOINT`, concurrency muammolari, isolation level'lar, locking, optimistic vs pessimistic, MVCC va ACID vs BASE. Misollar **PostgreSQL** uchun.

## Mundarija

- [Transaction nima](#transaction-nima)
- [ACID](#acid)
- [BEGIN, COMMIT, ROLLBACK](#begin-commit-rollback)
- [SAVEPOINT](#savepoint)
- [Concurrency muammolari](#concurrency-muammolari)
- [Isolation level'lar](#isolation-levellar)
- [Locking](#locking)
- [Deadlock](#deadlock)
- [Optimistic vs Pessimistic concurrency](#optimistic-vs-pessimistic)
- [MVCC — Postgres qanday qiladi](#mvcc)
- [ACID vs BASE](#acid-vs-base)
- [Savol-javoblar](#savol-javoblar)
- [Masalalar](#masalalar)

---

## Transaction nima

**💡 Tushuncha:** Transaction — bu bir nechta operatsiyani **bitta bo'linmas ish** sifatida bajaradigan birlik. Yo hammasi muvaffaqiyatli bajariladi, yo hech biri (qisman emas). Klassik misol — pul o'tkazish: A hisobdan minus, B hisobga plus. Agar minus bajarilib, plus bajarilmasa, pul "yo'qoladi". Transaction shuni oldini oladi.

```sql
BEGIN;
    UPDATE accounts SET balance = balance - 100 WHERE id = 1;
    UPDATE accounts SET balance = balance + 100 WHERE id = 2;
COMMIT;
```

Agar ikkinchi `UPDATE` xato bersa, `ROLLBACK` orqali birinchisi ham bekor qilinadi — pul o'z joyida qoladi.

---

## ACID

**💡 Tushuncha:** ACID — transaction'ning to'rtta kafolati: **A**tomicity, **C**onsistency, **I**solation, **D**urability.

### Atomicity (bo'linmaslik)

Transaction "hammasi yoki hech narsa". Ichidagi operatsiyalarning bir qismi bajarilib, qolgani bajarilmasligi mumkin emas. Xato bo'lsa — to'liq `ROLLBACK`. Yuqoridagi pul o'tkazishda minus va plus birga bajariladi yoki birga bekor qilinadi.

### Consistency (izchillik)

Transaction ma'lumotlar bazasini bir **to'g'ri (valid) holatdan boshqa to'g'ri holatga** o'tkazadi. Barcha cheklovlar (constraint, foreign key, unique, check, trigger) saqlanib qoladi. Masalan, balans manfiy bo'lmasligi `CHECK (balance >= 0)` bilan kafolatlanadi — transaction buni buzsa, rad etiladi.

### Isolation (izolyatsiya)

Bir vaqtda ishlayotgan transaction'lar bir-biriga **xalaqit bermaydi**. Garchi ular parallel ishlasa-da, natija go'yo ketma-ket bajarilgandek bo'lishi kerak (eng kuchli darajada — serializable). Izolyatsiya darajasi sozlanadi (pastda).

### Durability (chidamlilik)

`COMMIT` bo'lgan ma'lumot **doimiy saqlanadi** — server o'chib qaytsa, crash bo'lsa ham yo'qolmaydi. Buni ko'pincha **WAL** (Write-Ahead Log) ta'minlaydi: o'zgarish avval log'ga (disk'ga) yoziladi, keyin asosiy fayllarga.

| Harf | Nomi | Kafolat |
|------|------|---------|
| A | Atomicity | Hammasi yoki hech narsa |
| C | Consistency | Bazadan doim valid holatda chiqadi |
| I | Isolation | Parallel transaction'lar xalaqit bermaydi |
| D | Durability | Commit bo'lgani yo'qolmaydi |

---

## BEGIN, COMMIT, ROLLBACK

**💡 Tushuncha:** Transaction `BEGIN` (yoki `START TRANSACTION`) bilan boshlanadi, `COMMIT` bilan tasdiqlanadi (saqlanadi), `ROLLBACK` bilan bekor qilinadi.

```sql
BEGIN;
    UPDATE accounts SET balance = balance - 100 WHERE id = 1;
    -- biror tekshiruv yoki xato yuz berdi
    ROLLBACK;   -- yuqoridagi UPDATE bekor qilinadi
```

```sql
BEGIN;
    INSERT INTO orders(user_id, total) VALUES (5, 200);
    UPDATE inventory SET qty = qty - 1 WHERE product_id = 10;
COMMIT;   -- ikkala o'zgarish birga saqlanadi
```

**💡 Tushuncha:** Ko'p mijozlarda (psql, drivers) default holatda **autocommit** yoqilgan — har bir alohida statement o'zi transaction bo'lib darhol commit bo'ladi. Bir nechta statement'ni bitta transaction qilish uchun ularni `BEGIN ... COMMIT` ichiga olish kerak.

**⚠️ Ehtiyot bo'l:** Transaction'ni ochiq qoldirib (`COMMIT`/`ROLLBACK` qilmasdan) ulanishni band qilish — lock'larni ushlab turadi va boshqa transaction'larni bloklaydi. Transaction'larni iloji boricha **qisqa** tuting.

---

## SAVEPOINT

**💡 Tushuncha:** `SAVEPOINT` — transaction ichidagi "belgi", unga qaytib transaction'ning **bir qismini** bekor qilish mumkin (butunini emas). Bu qisman rollback'ni beradi.

```sql
BEGIN;
    INSERT INTO accounts(id, balance) VALUES (10, 500);

    SAVEPOINT sp1;
    UPDATE accounts SET balance = balance - 1000 WHERE id = 10;  -- xato: manfiy bo'ladi
    ROLLBACK TO SAVEPOINT sp1;   -- faqat shu UPDATE bekor, INSERT qoladi

    UPDATE accounts SET balance = balance - 100 WHERE id = 10;   -- to'g'ri operatsiya
COMMIT;
```

`RELEASE SAVEPOINT sp1` — savepoint'ni o'chiradi (kerak bo'lmay qolsa).

**⚠️ Ehtiyot bo'l:** PostgreSQL'da transaction ichida xato yuz bersa, butun transaction "abort" holatiga o'tadi va keyingi barcha buyruqlar rad etiladi. Savepoint shu yerda asqotadi: xatodan oldin `SAVEPOINT` qo'yib, xato bo'lsa `ROLLBACK TO` qilib davom etish mumkin.

---

## Concurrency muammolari

**💡 Tushuncha:** Bir vaqtda bir nechta transaction bir ma'lumotga tegsa, izolyatsiya yetarli bo'lmasa, quyidagi anomaliyalar kelib chiqadi.

### Dirty read (iflos o'qish)

Transaction boshqa transaction'ning hali **commit qilinmagan** o'zgarishini o'qiydi. Agar u boshqa transaction `ROLLBACK` qilsa, o'qilgan ma'lumot umuman mavjud bo'lmagan bo'lib chiqadi.

```
T1: UPDATE balance = 0  (commit emas)
T2: SELECT balance  → 0 ni o'qidi   ← dirty read
T1: ROLLBACK            → aslida 0 bo'lmagan!
```

### Non-repeatable read (takrorlanmas o'qish)

Bir transaction ichida bir qatorni ikki marta o'qiganda, oraliqda boshqa transaction uni o'zgartirib commit qilsa, **ikki xil qiymat** chiqadi.

```
T1: SELECT balance  → 100
T2: UPDATE balance = 200; COMMIT
T1: SELECT balance  → 200   ← bir transaction ichida o'zgardi
```

### Phantom read (sharpana o'qish)

Bir transaction ichida bir xil shart bilan **qatorlar to'plamini** ikki marta o'qiganda, oraliqda boshqa transaction yangi qator qo'shsa (yoki o'chirsa), ikkinchi o'qishda "sharpana" qatorlar paydo bo'ladi.

```
T1: SELECT COUNT(*) WHERE dept=1  → 3
T2: INSERT ... dept=1; COMMIT
T1: SELECT COUNT(*) WHERE dept=1  → 4   ← yangi (phantom) qator
```

### Lost update (yo'qolgan yangilanish)

Ikki transaction bir qatorni o'qib, o'zgartirib, yozsa — biri ikkinchisining o'zgarishini **ustidan bosib o'chiradi**.

```
T1: read balance=100
T2: read balance=100
T1: write 100+50 = 150; COMMIT
T2: write 100+30 = 130; COMMIT   ← T1 ning +50 si yo'qoldi!
```

**⚠️ Ehtiyot bo'l:** Lost update'ni oldini olishning eng oddiy yo'li — o'qib-keyin-yozish o'rniga atomik yangilash: `UPDATE accounts SET balance = balance + 50 WHERE id = 1`. Yoki `SELECT ... FOR UPDATE` (pessimistic) yoki version ustuni (optimistic) ishlatish.

---

## Isolation level'lar

**💡 Tushuncha:** Isolation level transaction'lar bir-biridan qanchalik izolyatsiya qilinishini belgilaydi. Yuqori daraja ko'proq anomaliyani oldini oladi, lekin parallellik (performance) kamayadi. SQL standartida 4 daraja bor.

```sql
BEGIN;
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;
-- ...
COMMIT;
```

### Qaysi daraja qaysi muammoni oldini oladi

| Isolation level | Dirty read | Non-repeatable read | Phantom read |
|-----------------|:----------:|:-------------------:|:------------:|
| READ UNCOMMITTED | mumkin | mumkin | mumkin |
| READ COMMITTED | oldini oladi | mumkin | mumkin |
| REPEATABLE READ | oldini oladi | oldini oladi | mumkin* |
| SERIALIZABLE | oldini oladi | oldini oladi | oldini oladi |

**💡 Tushuncha:** PostgreSQL'ning o'ziga xosligi:
- `READ UNCOMMITTED` so'ralsa ham, Postgres uni `READ COMMITTED` kabi ishlatadi — Postgres'da dirty read **umuman bo'lmaydi** (MVCC tufayli).
- Postgres'ning `REPEATABLE READ`i (snapshot isolation) phantom read'ni ham **oldini oladi** (jadvalda * shuning uchun). Lekin u "write skew" anomaliyasini oldini olmaydi — bu uchun `SERIALIZABLE` kerak.
- Default isolation — `READ COMMITTED`.

**⚠️ Ehtiyot bo'l:** Yuqori isolation darajasi (`SERIALIZABLE`) serialization xatosi (`could not serialize access`) berishi mumkin — application bunday holatda transaction'ni **qayta urinishga (retry)** tayyor bo'lishi kerak.

---

## Locking

**💡 Tushuncha:** Lock (qulf) — bir transaction ma'lumotga tegayotganda boshqalarni vaqtincha to'sib turuvchi mexanizm. Lock'larning ikki o'lchovi bor: **rejimi** (shared/exclusive) va **doirasi** (row/table).

### Shared (S) vs Exclusive (X) lock

- **Shared (read) lock** — bir nechta transaction bir vaqtda o'qishi mumkin (S va S mos keladi), lekin yozishni to'sadi.
- **Exclusive (write) lock** — faqat bitta transaction; boshqa hech kim o'qiy ham, yoza ham olmaydi (X hech narsa bilan mos kelmaydi).

| | S so'ramoqchi | X so'ramoqchi |
|---|:---:|:---:|
| **S ushlagan** | mos (✓) | bloklanadi |
| **X ushlagan** | bloklanadi | bloklanadi |

### Row vs Table lock

- **Row-level lock** — faqat ma'lum qatorlarni qulflaydi, boshqa qatorlar erkin. Parallellik yuqori. Postgres ko'pincha shu darajada ishlaydi.
- **Table-level lock** — butun jadvalni qulflaydi. `ALTER TABLE`, `TRUNCATE` kabi DDL operatsiyalar table lock oladi. Parallellik past.

```sql
-- Aniq row lock olish: qatorlarni o'qib, yozish uchun qulflash
BEGIN;
SELECT * FROM accounts WHERE id = 1 FOR UPDATE;   -- exclusive row lock
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
COMMIT;
```

`FOR UPDATE` — exclusive row lock (boshqa yozuvchilarni va `FOR UPDATE`larni to'sadi). `FOR SHARE` — shared row lock (o'qishga ruxsat, yozishni to'sadi).

**💡 Tushuncha:** Postgres'da **oddiy `SELECT` o'qishni bloklamaydi va o'qish yozishni bloklamaydi** (MVCC tufayli). Lock'lar asosan yozuvchilar orasida (yoki aniq `FOR UPDATE` so'ralganda) muhim bo'ladi.

---

## Deadlock

**💡 Tushuncha:** Deadlock — ikki (yoki ko'p) transaction bir-birining lock'ini kutib, hech qaysi davom eta olmay qolgan holat. T1 A'ni qulflagan, B'ni kutyapti; T2 B'ni qulflagan, A'ni kutyapti — abadiy kutish.

```
T1: LOCK row A ... endi B kerak (kutadi)
T2: LOCK row B ... endi A kerak (kutadi)
→ ikkalasi ham kutadi → deadlock
```

PostgreSQL deadlock'ni avtomatik **aniqlaydi** va birini "qurbon" qilib `ROLLBACK` qiladi (`deadlock detected` xatosi), boshqasi davom etadi.

**⚠️ Ehtiyot bo'l:** Deadlock'ni oldini olish:
- **Lock'larni doim bir xil tartibda oling** (masalan, har doim avval kichik `id`li qatorni). Bu eng samarali usul.
- Transaction'larni **qisqa** tuting.
- Kerakli lock'larni iloji boricha **birdaniga** oling, bo'lak-bo'lak emas.
- Deadlock bo'lganda application transaction'ni **retry** qilsin.

---

## Optimistic vs Pessimistic

**💡 Tushuncha:** Bu — concurrency'ni boshqarishning ikki falsafasi.

### Pessimistic concurrency

"Konflikt bo'ladi deb taxmin qilamiz" — ma'lumotni **oldindan qulflaymiz** (`SELECT ... FOR UPDATE`), boshqalar kutadi. Konflikt ko'p bo'lganda yaxshi, lekin lock'lar parallellikni kamaytiradi va deadlock xavfi bor.

```sql
BEGIN;
SELECT balance FROM accounts WHERE id = 1 FOR UPDATE;  -- qulfladik
-- hisob-kitob...
UPDATE accounts SET balance = ... WHERE id = 1;
COMMIT;
```

### Optimistic concurrency

"Konflikt kam bo'ladi deb taxmin qilamiz" — qulflamaymiz, lekin yozishdan oldin ma'lumot o'zgarmaganini **version yoki timestamp** orqali tekshiramiz. O'zgargan bo'lsa — rad etib, retry qilamiz.

```sql
-- accounts jadvalida version ustuni bor
UPDATE accounts
SET balance = 150, version = version + 1
WHERE id = 1 AND version = 7;   -- biz o'qiganda version 7 edi

-- Agar 0 qator yangilangan bo'lsa → kimdir o'zgartirgan → retry
```

| | Pessimistic | Optimistic |
|---|---|---|
| Yondashuv | Oldindan qulflash | Yozishda tekshirish |
| Konflikt ko'p bo'lsa | yaxshi | ko'p retry, sekin |
| Konflikt kam bo'lsa | ortiqcha lock | yaxshi |
| Deadlock | mumkin | yo'q |
| Tipik foydalanish | bank, inventar | web app, REST API |

---

## MVCC

**💡 Tushuncha:** MVCC (Multi-Version Concurrency Control) — PostgreSQL (va ko'p zamonaviy DB)ning izolyatsiya usuli. G'oya: qatorni o'zgartirganda eskisini o'chirmasdan, **yangi versiyasini** yaratadi. Har transaction o'z "snapshot"iga (vaqt kesimiga) mos versiyani ko'radi.

Natija: **o'qish yozishni bloklamaydi, yozish o'qishni bloklamaydi.** Har kim o'z snapshot'idagi izchil ma'lumotni ko'radi.

Postgres buni qator ichidagi yashirin ustunlar bilan amalga oshiradi:
- `xmin` — qatorni yaratgan transaction id.
- `xmax` — qatorni o'chirgan/yangilagan transaction id.

Transaction o'z snapshot'iga ko'ra qaysi versiya "ko'rinadi"ganini hisoblaydi.

**⚠️ Ehtiyot bo'l:** MVCC'ning narxi — eski versiyalar ("dead tuples") jadvalda qoladi va joy egallaydi (bloat). Ularni tozalash uchun Postgres'da **VACUUM** (odatda avtomatik `autovacuum`) ishlaydi. Yangilanishlar ko'p jadvallarda autovacuum sozlamalari muhim.

---

## ACID vs BASE

**💡 Tushuncha:** ACID — relational (SQL) bazalarning kuchli izchillik modeli. **BASE** — ko'p NoSQL/distributed tizimlarning yumshoqroq modeli: **B**asically **A**vailable, **S**oft state, **E**ventually consistent. Bu CAP teoremasi bilan bog'liq: tarmoq bo'linishi (partition) bo'lganda izchillik va mavjudlik orasida tanlov qilinadi.

| | ACID | BASE |
|---|---|---|
| Izchillik | Kuchli (strong), darhol | Pirovardida (eventual) |
| Mavjudlik | Ba'zan izchillik uchun qurbon | Yuqori prioritet |
| Misol tizimlar | PostgreSQL, MySQL, Oracle | Cassandra, DynamoDB, ko'p NoSQL |
| Mos keladi | Bank, to'lov, buyurtma | Feed, log, katalog, analitika |

**💡 Tushuncha:** "Eventually consistent" — yozilgan ma'lumot barcha nusxalarga **biroz vaqt o'tib** yetadi; oraliqda turli node'lar turli qiymat ko'rishi mumkin. Bu mavjudlik va masshtablash uchun to'lanadigan narx. Pul masalasida ACID, ijtimoiy tarmoq feed'ida BASE odatda to'g'ri tanlov.

---

## Savol-javoblar

### ❓ Transaction nima va nima uchun kerak?

**✅ Javob:** Transaction — bir nechta operatsiyani bitta bo'linmas birlik sifatida bajaradigan mexanizm: yo hammasi muvaffaqiyatli, yo hech biri. U ma'lumot izchilligini saqlaydi (masalan, pul o'tkazishda minus va plus birga). ACID kafolatlari shu transaction orqali ta'minlanadi.

### ❓ ACID nimani anglatadi?

**✅ Javob:** **Atomicity** — hammasi yoki hech narsa; **Consistency** — baza doim valid holatda qoladi (constraint'lar buzilmaydi); **Isolation** — parallel transaction'lar bir-biriga xalaqit bermaydi; **Durability** — commit qilingan ma'lumot crash bo'lsa ham yo'qolmaydi (odatda WAL orqali).

### ❓ COMMIT va ROLLBACK farqi?

**✅ Javob:** `COMMIT` transaction ichidagi barcha o'zgarishlarni doimiy saqlaydi va boshqalarga ko'rinadigan qiladi. `ROLLBACK` esa transaction boshlanganidan beri qilingan barcha o'zgarishlarni bekor qiladi, go'yo hech narsa bo'lmagandek. Xato yuz berganda yoki shart bajarilmaganda `ROLLBACK` ishlatiladi.

### ❓ SAVEPOINT nima uchun kerak?

**✅ Javob:** Savepoint transaction ichida belgi qo'yadi va `ROLLBACK TO SAVEPOINT` orqali transaction'ning **bir qismini** bekor qilishga imkon beradi — butun transaction'ni emas. Murakkab, ko'p qadamli transaction'larda bir qadam xato bersa, undan oldingi ishlarni saqlab qolib davom etish uchun foydali.

### ❓ Dirty read, non-repeatable read va phantom read farqi?

**✅ Javob:** **Dirty read** — boshqa transaction'ning commit qilinmagan (keyin bekor bo'lishi mumkin) o'zgarishini o'qish. **Non-repeatable read** — bir transaction ichida bir **qatorni** ikki marta o'qiganda, oraliqdagi commit tufayli qiymat o'zgarishi. **Phantom read** — bir shart bilan **qatorlar to'plamini** ikki marta o'qiganda yangi qatorlar paydo bo'lishi (yoki yo'qolishi).

### ❓ Lost update nima va qanday oldini olinadi?

**✅ Javob:** Ikki transaction bir qatorni o'qib, o'zgartirib yozganda, biri ikkinchisining o'zgarishini ustidan yozib o'chirishi. Oldini olish: (1) atomik yangilash `SET x = x + 1` (o'qib-yozish o'rniga); (2) pessimistic — `SELECT ... FOR UPDATE`; (3) optimistic — version ustuni bilan `WHERE version = ?`; (4) yuqoriroq isolation (`REPEATABLE READ`/`SERIALIZABLE`).

### ❓ Isolation level'lar qaysilar va qaysi muammoni oldini oladi?

**✅ Javob:** To'rtta: `READ UNCOMMITTED` (hech narsani oldini olmaydi), `READ COMMITTED` (dirty read'ni), `REPEATABLE READ` (dirty + non-repeatable read'ni), `SERIALIZABLE` (hammasini, shu jumladan phantom). Daraja qancha yuqori bo'lsa, anomaliya kam, lekin parallellik (performance) pasayadi. Postgres'da default — `READ COMMITTED`.

### ❓ PostgreSQL'da isolation level'lar standartdan qanday farq qiladi?

**✅ Javob:** Postgres'da `READ UNCOMMITTED` aslida `READ COMMITTED` kabi ishlaydi — dirty read umuman bo'lmaydi (MVCC). Postgres'ning `REPEATABLE READ`i snapshot isolation bo'lib, phantom read'ni ham oldini oladi (standart talab qilmasa ham). Faqat `SERIALIZABLE` "write skew"ga qarshi to'liq kafolat beradi.

### ❓ Shared va exclusive lock farqi?

**✅ Javob:** **Shared (S, read) lock** — bir nechta transaction bir vaqtda ushlashi mumkin (o'qish uchun), lekin yozishni to'sadi. **Exclusive (X, write) lock** — faqat bitta transaction ushlaydi, boshqa hech kim (na o'qish, na yozish) tega olmaydi. S va S mos keladi; X esa hech narsa bilan mos kelmaydi.

### ❓ Row-level va table-level lock farqi?

**✅ Javob:** Row-level lock faqat ma'lum qatorlarni qulflaydi, boshqa qatorlar erkin qoladi — parallellik yuqori. Table-level lock butun jadvalni qulflaydi (masalan `ALTER TABLE`, `TRUNCATE`) — parallellik past. Postgres odatda eng mayda (row) darajada qulflaydi.

### ❓ Deadlock nima va qanday oldini olinadi?

**✅ Javob:** Deadlock — ikki transaction bir-birining lock'ini kutib qotib qolishi (T1 A'ni ushlab B'ni kutadi, T2 B'ni ushlab A'ni kutadi). Postgres uni avtomatik aniqlab, birini rollback qiladi. Oldini olish: lock'larni **doim bir xil tartibda** olish, transaction'larni qisqa tutish, kerakli lock'larni birdaniga olish va application'da retry mantiqi.

### ❓ Optimistic va pessimistic concurrency farqi?

**✅ Javob:** **Pessimistic** — konflikt bo'ladi deb ma'lumotni oldindan qulflaydi (`FOR UPDATE`); konflikt ko'p bo'lganda yaxshi, lekin lock va deadlock xavfi. **Optimistic** — qulflamaydi, yozishda version/timestamp orqali o'zgarmaganini tekshiradi, o'zgargan bo'lsa retry; konflikt kam bo'lganda yaxshi (ko'p web ilovalar shuni ishlatadi).

### ❓ MVCC nima va u qanday ishlaydi?

**✅ Javob:** MVCC (Multi-Version Concurrency Control) — qatorni o'zgartirganda eskisini o'chirmay, yangi versiyasini yaratadi. Har transaction o'z snapshot'iga mos versiyani ko'radi. Natijada o'qish yozishni, yozish o'qishni bloklamaydi. Postgres buni `xmin`/`xmax` yashirin ustunlar bilan amalga oshiradi; eski versiyalarni `VACUUM` tozalaydi.

### ❓ MVCC'ning kamchiligi bormi?

**✅ Javob:** Ha — eski qator versiyalari ("dead tuples") darhol o'chmaydi va jadval/index "shishadi" (bloat), disk va xotira egallaydi. Ularni `VACUUM` (odatda avtomatik `autovacuum`) tozalaydi. Yangilanishlar ko'p jadvallarda autovacuum to'g'ri sozlanmasa, bloat va sekinlashuv muammosi bo'ladi.

### ❓ ACID va BASE farqi? Qachon qaysi biri?

**✅ Javob:** ACID — kuchli izchillik (darhol consistency), relational DB'larda; pul, to'lov, buyurtma kabi to'g'rilik muhim bo'lgan joylarda. BASE (Basically Available, Soft state, Eventually consistent) — yumshoqroq, ko'p NoSQL/distributed tizimlarda; mavjudlik va masshtablash birinchi o'rinda bo'lib, vaqtinchalik nomuvofiqlik qabul qilinadigan joylarda (feed, katalog, log, analitika).

### ❓ Durability qanday ta'minlanadi?

**✅ Javob:** Asosan **WAL** (Write-Ahead Log) orqali: har qanday o'zgarish avval ketma-ket log'ga (disk'ga) yoziladi, keyingina asosiy ma'lumot fayllariga qo'llanadi. Crash bo'lsa, baza qaytib WAL'dan commit qilingan o'zgarishlarni tiklaydi. Shu sabab `COMMIT` qaytgan ma'lumot yo'qolmaydi.

---

## Masalalar

> Yechimlar: [solutions/databases/04-transactions-acid.md](../solutions/databases/04-transactions-acid.md)

Quyidagi jadvaldan foydalaning:

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

1. **Pul o'tkazish.** Alice (id=1) dan Bob (id=2) ga 300 o'tkazadigan transaction yozing. Ikkala `UPDATE` atomik bajarilsin; balans manfiy bo'lsa o'tkazish bekor bo'lsin.

2. **Savepoint.** Bitta transaction ichida: avval Alice balansiga 100 qo'shing, keyin savepoint qo'yib, Alice'dan 5000 yechishga urinib ko'ring (xato beradi), uni savepoint'ga rollback qiling va transaction'ni muvaffaqiyatli yakunlang.

3. **Lost update.** Ikki konkurent transaction Alice balansini o'qib, oshirmoqchi bo'lsa lost update qanday yuz berishini tushuntiring va uni atomik `UPDATE ... SET balance = balance + N` bilan to'g'rilang.

4. **Pessimistic lock.** `SELECT ... FOR UPDATE` ishlatib, Alice qatorini qulflagan, keyin yangilaydigan xavfsiz transaction yozing. Nima uchun bu lost update'ni oldini olishini ayting.

5. **Optimistic lock.** `version` ustunidan foydalanib optimistic concurrency bilan balansni yangilang. Yangilanish 0 qatorga ta'sir qilsa nima qilish kerakligini izohlang.

6. **Isolation jadvali.** READ UNCOMMITTED, READ COMMITTED, REPEATABLE READ, SERIALIZABLE darajalarining qaysi biri dirty/non-repeatable/phantom read'ni oldini olishini jadval ko'rinishida to'ldiring va PostgreSQL'da farqini yozing.

7. **Deadlock.** Ikki transaction ikki qatorni teskari tartibda qulflab deadlock yaratadigan ssenariy yozing (T1 va T2 buyruqlar ketma-ketligi), so'ng uni lock tartibini bir xillashtirish bilan tuzating.

8. **Non-repeatable read.** READ COMMITTED da non-repeatable read sodir bo'ladigan ikki transaction ketma-ketligini ko'rsating va REPEATABLE READ uni qanday yo'qotishini tushuntiring.

---

← [Database bo'limiga qaytish](./README.md)
