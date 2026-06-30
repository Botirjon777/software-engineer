# Database Design — Masalalar Yechimlari

Bu hujjat [07-database-design.md](../../databases/07-database-design.md) dagi masalalarning batafsil yechimlari.

---

## 1. Normal forma aniqlash

**Jadval:** `enrollments(student_id, course_id, student_name, course_title, grade)`, PK `(student_id, course_id)`.

**Tahlil:** PK composite. Dependency'lar:
- `student_id → student_name` (faqat PK qismiga bog'liq — **partial dependency**)
- `course_id → course_title` (faqat PK qismiga bog'liq — **partial dependency**)
- `(student_id, course_id) → grade` (to'liq PK'ga bog'liq — to'g'ri)

**Buzilgan forma: 2NF.** `student_name` va `course_title` composite key'ning faqat bir qismiga bog'liq. Bu update anomaliyaga olib keladi (talaba ismini o'zgartirsa, uning barcha enrollment qatorlarini yangilash kerak).

**Yechim — partial dependency'larni alohida jadvalga chiqarish:**

```sql
CREATE TABLE students (
    student_id   BIGINT PRIMARY KEY,
    student_name VARCHAR(200) NOT NULL
);
CREATE TABLE courses (
    course_id    BIGINT PRIMARY KEY,
    course_title VARCHAR(300) NOT NULL
);
CREATE TABLE enrollments (
    student_id BIGINT NOT NULL REFERENCES students(student_id),
    course_id  BIGINT NOT NULL REFERENCES courses(course_id),
    grade      CHAR(1),
    PRIMARY KEY (student_id, course_id)
);
```

Endi har bir non-key atribut to'liq key'ga bog'liq — 2NF (va 3NF) qanoatlantiriladi.

---

## 2. M:N dizayn — followers/following

Self-referencing M:N: bir `users` jadvali o'zi bilan ko'p-ko'pga bog'lanadi. Junction table ikkala FK ham `users`'ga ishora qiladi.

```sql
CREATE TABLE users (
    id       BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    username VARCHAR(50) NOT NULL UNIQUE
);

CREATE TABLE follows (
    follower_id  BIGINT NOT NULL REFERENCES users(id),  -- kim follow qildi
    followee_id  BIGINT NOT NULL REFERENCES users(id),  -- kimni follow qildi
    created_at   TIMESTAMPTZ NOT NULL DEFAULT now(),
    PRIMARY KEY (follower_id, followee_id),
    CHECK (follower_id <> followee_id)   -- o'zini follow qila olmaydi
);

-- "Men kimlarni follow qilaman" (following): follower_id bo'yicha
-- PK (follower_id, followee_id) bu so'rovni qoplaydi (leftmost-prefix).

-- "Meni kimlar follow qiladi" (followers): followee_id bo'yicha
CREATE INDEX idx_follows_followee ON follows(followee_id);
```

**Dizayn nuqtalari:**
- Composite PK `(follower_id, followee_id)` duplicate follow'ni avtomatik oldini oladi.
- `CHECK (follower_id <> followee_id)` o'zini follow qilishni taqiqlaydi.
- Ikkala yo'nalish (following va followers) so'rovlari uchun ikkala ustunga ham index kerak: PK birinchisini, qo'shimcha index ikkinchisini qoplaydi.
- Katta o'lchovda `follower_count`/`following_count` ni denormallashtirish (precomputed) mumkin.

---

## 3. Primary key tanlash — sharded events

**Tanlov: ULID (yoki UUIDv7).**

**Asoslash:**
- **Distributed-safe**: events bir nechta shard/node'da generatsiya qilinadi. Auto-increment markaziy ketma-ketlik talab qiladi → bottleneck va to'qnashuv. ULID har bir node'da mustaqil, to'qnashuvsiz generatsiya qilinadi.
- **Index locality**: ULID time-ordered (boshida timestamp). Shuning uchun yangi event'lar B-tree leaf'ining oxiriga ketma-ket joylashadi → page split kam, write tez, index ixcham. Sof UUIDv4 random bo'lib, har insert tasodifiy joyga tushadi → fragmentation va sekin write (ayniqsa write-heavy jadvalda).
- **Hajm**: 128-bit (16 bayt) — BIGINT'dan katta, lekin time-orderedlik foydasi buni qoplaydi.

```sql
-- ULID matn sifatida (26 belgi) yoki 16-baytli binary
CREATE TABLE events (
    id         CHAR(26) PRIMARY KEY,   -- ULID
    type       VARCHAR(50) NOT NULL,
    payload    JSONB,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

**Nega auto-increment emas:** sharded muhitda markaziy ID generatori scaling'ni cheklaydi va single point bo'ladi. **Nega UUIDv4 emas:** write-heavy clustered jadvalda random PK index locality'ni buzadi. ULID/UUIDv7 ikkala muammoni ham hal qiladi.

---

## 4. Soft delete + unique email

Oddiy `UNIQUE(email)` ishlamaydi — u o'chirilgan qatorlarni ham hisobga oladi, shuning uchun o'chirilgan foydalanuvchi email'ini qayta ishlatib bo'lmaydi. Yechim — **partial unique index** (faqat tirik qatorlar uchun).

```sql
ALTER TABLE users ADD COLUMN deleted_at TIMESTAMPTZ;

-- Faqat deleted_at IS NULL bo'lgan (tirik) qatorlar orasida email unique
CREATE UNIQUE INDEX uq_users_email_active
    ON users(email)
    WHERE deleted_at IS NULL;
```

Endi:
- Ikki tirik foydalanuvchi bir xil email'ga ega bo'la olmaydi.
- O'chirilgan foydalanuvchi (deleted_at to'ldirilgan) email'ini yangi foydalanuvchi qayta ishlatishi mumkin.

**Partial index'siz DBMS (masalan eski MySQL) uchun muqobil:** unique constraint'ga `deleted_at` ni ham qo'shish, lekin NULL o'rniga sentinel qiymat (masalan 0 yoki o'chirish vaqti):
```sql
-- deleted_at default 0; unique (email, deleted_at)
-- tirik: deleted_at = 0; o'chirilganda = o'chirish timestamp'i
UNIQUE (email, deleted_at)
```

---

## 5. Denormalizatsiya — author post count

**Muammo:** `SELECT COUNT(*) FROM posts WHERE author_id = ?` har post sahifasida ishlaydi, sahifa hot path.

**Yechim — precomputed counter ustuni:**

```sql
ALTER TABLE users ADD COLUMN post_count INT NOT NULL DEFAULT 0;
```

**Consistency'ni saqlash variantlari (eng ishonchlidan):**

1. **Trigger (DBMS ichida, eng ishonchli):**
```sql
CREATE FUNCTION bump_post_count() RETURNS trigger AS $$
BEGIN
  IF TG_OP = 'INSERT' THEN
    UPDATE users SET post_count = post_count + 1 WHERE id = NEW.author_id;
  ELSIF TG_OP = 'DELETE' THEN
    UPDATE users SET post_count = post_count - 1 WHERE id = OLD.author_id;
  END IF;
  RETURN NULL;
END; $$ LANGUAGE plpgsql;

CREATE TRIGGER trg_post_count
  AFTER INSERT OR DELETE ON posts
  FOR EACH ROW EXECUTE FUNCTION bump_post_count();
```
Trigger yangilanish bilan bir transaction'da bo'lgani uchun atomik va consistency kafolatlangan.

2. **Application-level**: post yaratish/o'chirish bilan bir transaction ichida counter'ni yangilash.

3. **Periodik reconciliation**: drift bo'lishi mumkin (counter yo'qotsa) deb, kunlik batch job real COUNT bilan `post_count` ni qayta hisoblab to'g'rilaydi.

**Trade-off:** read endi join'siz va tez (`SELECT post_count FROM users`); write biroz qimmatlashadi (qo'shimcha UPDATE) va consistency mas'uliyati qo'shiladi. Hot read path uchun bu o'rinli.

---

## 6. Partition strategiyasi — logs

**Tanlov: `created_at` bo'yicha RANGE partitioning (kunlik yoki haftalik partition).**

**Asoslash:**
- So'rovlarning aksariyati oxirgi 7 kunga tegishli → vaqt bo'yicha range partition **partition pruning** beradi: faqat tegishli kunlik partition'lar skanlanadi, qolgani tegilmaydi.
- 90 kundan keyin o'chirish → eski partition'ni `DROP TABLE` qilish (sekin `DELETE`'dan emas). Bu lock'siz va deyarli darhol, jadval bloat'ini ham oldini oladi.

```sql
CREATE TABLE logs (
    id         BIGINT,
    created_at TIMESTAMPTZ NOT NULL,
    level      VARCHAR(10),
    message    TEXT
) PARTITION BY RANGE (created_at);

CREATE TABLE logs_2026_06_30 PARTITION OF logs
    FOR VALUES FROM ('2026-06-30') TO ('2026-07-01');
-- ... har kun uchun yangi partition (avtomatik: pg_partman yoki cron)

-- 90 kundan eski partition'ni o'chirish:
DROP TABLE logs_2026_03_31;
```

**Nega hash emas:** hash partition vaqt-asosli pruning bermaydi va eski ma'lumotni butun partition sifatida o'chirishga imkon bermaydi (eski/yangi har partition'ga aralashadi). Range key esa retention va recency query pattern'iga ideal mos keladi.

**Eslatma:** Postgres'da partitioned jadvalda unique/PK partition key'ni o'z ichiga olishi kerak, shuning uchun PK `(id, created_at)` bo'ladi.

---

## 7. Schema migration — full_name → first_name + last_name

**Expand/contract, downtime'siz, bosqichma-bosqich:**

**Bosqich 1 — Expand (yangi ustunlar qo'shish):**
```sql
ALTER TABLE users ADD COLUMN first_name VARCHAR(100);
ALTER TABLE users ADD COLUMN last_name  VARCHAR(100);
```
Nullable, default'siz — lock minimal, eski kod buzilmaydi.

**Bosqich 2 — Dual-write (kod deploy):**
- Ilova kodini yangilang: yangi/yangilangan user uchun `first_name`, `last_name` **ham** to'ldiriladi (`full_name` ham eski compatibility uchun saqlanadi).
- Eski kod hali `full_name`'dan o'qiydi — backward-compatible.

**Bosqich 3 — Backfill (batch):**
```sql
-- Katta UPDATE emas, batch'lab (lock va replication lag oldini olish):
UPDATE users
SET first_name = split_part(full_name, ' ', 1),
    last_name  = substr(full_name, length(split_part(full_name,' ',1)) + 2)
WHERE id BETWEEN :start AND :end          -- batch oynasi
  AND first_name IS NULL;
```

**Bosqich 4 — O'qishni ko'chirish:**
- Barcha read kodini `first_name`/`last_name`'dan o'qiydigan qilib deploy qiling.
- `full_name` endi hech kim ishlatmaydi.

**Bosqich 5 — Contract (eski ustunni o'chirish):**
```sql
ALTER TABLE users DROP COLUMN full_name;
```

**Muhim qoidalar:** har bosqich alohida deploy; rename'ni bitta qadamda qilmaslik; backfill batch'lab; `NOT NULL` constraint'ni faqat backfill tugagach qo'shish.

---

## 8. CAP qarori — bank vs like counter

**(a) Bank balansi — CP (Consistency).**

Pul hisob-kitobida noto'g'ri yoki eskirgan balans qabul qilinmaydi: ikki marta yechish (double-spend) yoki manfiy balans katastrofa. Partition paytida tizim available bo'lmaslikni (xato qaytarish, operatsiyani rad etish) tanlaydi — consistency'ni hech qachon qurbon qilmaydi. Foydalanuvchi "hozir ishlamayapti" xabarini olishi, noto'g'ri balansdan ko'ra yaxshiroq.

**(b) Like counter — AP (Availability).**

Like soni biroz eskirgan yoki taxminiy bo'lishi mutlaqo qabul qilinadi. Asosiysi — tizim har doim javob bersin (like bosish ishlasin). Partition paytida har node o'z hisobini qabul qiladi, keyin eventual consistency bilan birlashtiriladi. Bir necha soniya/daqiqa noaniqlik foydalanuvchiga zarar qilmaydi, "xizmat ishlamayapti" esa qiladi.

**Umumiy tamoyil:** correctness kritik (pul, inventar, identity) → CP. Availability va UX kritik, eventual consistency yetarli (like, view count, feed) → AP.

---

## 9. E-commerce schema — restoran yetkazib berish

```sql
-- Foydalanuvchilar
CREATE TABLE users (
    id         BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    phone      VARCHAR(20) NOT NULL UNIQUE,
    name       VARCHAR(200) NOT NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

-- Kuryerlar
CREATE TABLE couriers (
    id         BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    name       VARCHAR(200) NOT NULL,
    is_active  BOOLEAN NOT NULL DEFAULT true
);

-- Restoranlar (1:N restaurant -> menu_items)
CREATE TABLE restaurants (
    id      BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    name    VARCHAR(300) NOT NULL,
    address VARCHAR(500) NOT NULL
);

-- Menyu elementlari
CREATE TABLE menu_items (
    id            BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    restaurant_id BIGINT NOT NULL REFERENCES restaurants(id),
    name          VARCHAR(300) NOT NULL,
    price         NUMERIC(10,2) NOT NULL CHECK (price >= 0),
    is_available  BOOLEAN NOT NULL DEFAULT true
);
CREATE INDEX idx_menu_restaurant ON menu_items(restaurant_id);

-- Buyurtmalar (1:N user -> orders; kuryer tayinlash)
CREATE TABLE orders (
    id            BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    user_id       BIGINT NOT NULL REFERENCES users(id),
    restaurant_id BIGINT NOT NULL REFERENCES restaurants(id),
    courier_id    BIGINT REFERENCES couriers(id),   -- NULL: hali tayinlanmagan
    status        VARCHAR(20) NOT NULL DEFAULT 'placed'
                  CHECK (status IN ('placed','accepted','cooking',
                                    'delivering','delivered','cancelled')),
    total         NUMERIC(10,2) NOT NULL DEFAULT 0,
    created_at    TIMESTAMPTZ NOT NULL DEFAULT now()
);
CREATE INDEX idx_orders_user    ON orders(user_id);
CREATE INDEX idx_orders_courier ON orders(courier_id);
CREATE INDEX idx_orders_created ON orders(created_at);

-- Buyurtma elementlari (junction + miqdor + narx snapshot)
CREATE TABLE order_items (
    order_id     BIGINT NOT NULL REFERENCES orders(id) ON DELETE CASCADE,
    menu_item_id BIGINT NOT NULL REFERENCES menu_items(id),
    quantity     INT NOT NULL CHECK (quantity > 0),
    unit_price   NUMERIC(10,2) NOT NULL,   -- snapshot: buyurtma paytidagi narx
    PRIMARY KEY (order_id, menu_item_id)
);
```

**Dizayn qarorlari:**
- **Cardinality:** `users 1:N orders`, `restaurants 1:N menu_items`, `restaurants 1:N orders`, `couriers 1:N orders`, `orders M:N menu_items` (junction `order_items`).
- **Kuryer tayinlash:** `orders.courier_id` nullable FK — buyurtma berilganda NULL, keyin tayinlanadi. (Bir kuryer ko'p buyurtma → 1:N.)
- **Narx snapshot:** `order_items.unit_price` buyurtma paytidagi narxni saqlaydi; `menu_items.price` keyin o'zgarsa ham tarixiy buyurtma to'g'ri qoladi.
- **Index'lar:** barcha FK'larga (`menu_items.restaurant_id`, `orders.user_id/courier_id`) va tez-tez filtrlanadigan `orders.created_at`.
- **`is_available`/`is_active`** flag'lar — soft delete o'rniga oddiy holat (menyu/kuryer uchun audit shart emas).

---

## 10. Replikatsiya tanlovi — global, multi-region write

**Tanlov: Multi-Leader (multi-master) replikatsiya.**

**Asoslash:**
- Foydalanuvchilar ikki regiondan (EU, US) **yozadi** va past latency kerak → har region o'z mahalliy leader'iga yozsin (uzoq markaziy leader'ga sayohatsiz). Bu leader-follower (bitta leader) bilan mumkin emas, chunki uzoq regiondan write yuqori latency'ga uchraydi.
- Multi-leader har regionda mustaqil yozishga imkon beradi; o'zgarishlar regionlar orasida asinxron tarqaladi.

**Write conflict'ni hal qilish:**
- Ikki region bir xil yozuvni bir vaqtda o'zgartirsa, conflict yuzaga keladi. Strategiyalar:
  - **LWW (Last-Write-Wins)**: timestamp bo'yicha eng oxirgisi yutadi — oddiy, ammo ma'lumot yo'qotishi mumkin.
  - **CRDT** (conflict-free replicated data types): counter, set kabilar uchun matematik birlashtirish — yo'qotishsiz.
  - **Application-level merge**: domen mantiqiga ko'ra qo'lda birlashtirish (masalan savatlarni qo'shish).
  - **Sharding by region**: har user ma'lumotini bitta "home" regionga biriktirib, write'larni shu yerga yo'naltirish → conflict'ni butunlay oldini olish (eng toza yechim, agar mumkin bo'lsa).

**Trade-off (PACELC):** bu PA/EL tanlov — partition'da available, normalda past latency, evaziga eventual consistency va conflict resolution murakkabligi. Agar kuchli consistency majburiy bo'lsa (masalan to'lov), o'sha jadvallarni alohida single-leader / quorum bilan ushlash kerak.

---

← [Database bo'limiga qaytish](../../databases/README.md)
