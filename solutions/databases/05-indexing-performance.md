# Indexing va Performance — Masalalar yechimi

[← Mavzuga qaytish](../../databases/05-indexing-performance.md)

---

## 1-masala — Composite index va ustun tartibi

```sql
CREATE INDEX idx_orders_cust_status ON orders (customer_id, status);
```

**Tushuntirish:** Ikkala ustun ham tenglik (`=`) bilan filterlanadi, shuning uchun har ikkisi ham indexga kiradi. Tartibni selektivlik belgilaydi: `customer_id` `status`dan ko'ra ko'proq xil qiymatga ega (yuqori cardinality), shuning uchun u oldinda turgani index'ni samaraliroq qiladi. `status` faqat 3 xil qiymatga ega, shuning uchun u ikkinchi o'rinda mantiqiy.

Agar ko'pincha faqat `customer_id` bo'yicha ham qidirilsa — bu tartib leftmost-prefix tufayli o'sha query'ni ham qoplaydi.

---

## 2-masala — Leftmost-prefix tahlili

Index: `(customer_id, created_at)`.

- **a) `WHERE customer_id = 5`** → ✅ **Ishlatadi.** `customer_id` — index'ning leftmost ustuni.
- **b) `WHERE created_at > '2026-01-01'`** → ❌ **Ishlatmaydi** (yoki to'liq emas). `created_at` index'ning ikkinchi ustuni; birinchi ustun (`customer_id`) shartda yo'q, leftmost-prefix buziladi → Seq Scan.
- **c) `WHERE customer_id = 5 AND created_at > '2026-01-01'`** → ✅ **To'liq ishlatadi.** `customer_id` bo'yicha index seek, keyin shu `customer_id` ichida `created_at` range bo'yicha skanlash — composite index uchun ideal pattern.

---

## 3-masala — Functional index / query qayta yozish

**Variant 1 — query qayta yozish (agar email allaqachon lowercase saqlansa):**
```sql
SELECT * FROM customers WHERE email = 'a@b.com';
CREATE INDEX idx_customers_email ON customers (email);
```

**Variant 2 — functional (expression) index (case-insensitive saqlash kerak bo'lsa):**
```sql
CREATE INDEX idx_customers_lower_email ON customers (LOWER(email));
-- endi bu query index'dan foydalanadi:
SELECT * FROM customers WHERE LOWER(email) = 'a@b.com';
```

Asosiy g'oya: index ifodasi (`LOWER(email)`) query'dagi ifoda bilan **aynan** mos kelishi kerak.

---

## 4-masala — Covering index

```sql
CREATE INDEX idx_orders_cust_cover ON orders (customer_id, total);
```

Yoki PostgreSQL `INCLUDE` bilan (`total`ni faqat payload sifatida):
```sql
CREATE INDEX idx_orders_cust_cover ON orders (customer_id) INCLUDE (total);
```

`SELECT id, total FROM orders WHERE customer_id = 5` — `id` PK bo'lgani uchun secondary index'da allaqachon mavjud, `total` esa indexda saqlanadi. Natijada Index-Only Scan bo'ladi, heap'ga teginish yo'qoladi.

---

## 5-masala — Partial index

```sql
CREATE INDEX idx_orders_pending ON orders (created_at)
WHERE status = 'pending';
```

**Nega afzal:** Agar `pending` buyurtmalar jami qatorlarning kichik qismini (masalan 2-5%) tashkil qilsa, partial index to'liq indexdan ancha kichik bo'ladi. Kichik index → kamroq disk, tezroq lookup, arzonroq maintenance. Bundan tashqari, `WHERE status='pending'` query'lari aynan shu index'dan foydalanadi.

---

## 6-masala — N+1 ni JOIN bilan hal qilish

```js
const rows = await q(`
  SELECT c.id   AS customer_id,
         c.email,
         o.id   AS order_id,
         o.total
  FROM customers c
  LEFT JOIN orders o ON o.customer_id = c.id
  ORDER BY c.id
`);

// natijani application'da customer bo'yicha guruhlash
const customers = {};
for (const r of rows) {
  if (!customers[r.customer_id])
    customers[r.customer_id] = { id: r.customer_id, email: r.email, orders: [] };
  if (r.order_id)
    customers[r.customer_id].orders.push({ id: r.order_id, total: r.total });
}
```

N+1 query (1 + N) o'rniga bitta query. Muqobil: 2 ta query (customers, keyin `WHERE customer_id IN (...)`) — agar JOIN natijasi juda keng bo'lsa.

---

## 7-masala — Keyset (cursor) pagination

```sql
-- birinchi sahifa
SELECT * FROM orders
ORDER BY created_at DESC, id DESC
LIMIT 20;

-- keyingi sahifa: oxirgi qatorning (created_at, id) qiymatidan
SELECT * FROM orders
WHERE (created_at, id) < ('2026-06-01 10:00:00', 4821)
ORDER BY created_at DESC, id DESC
LIMIT 20;

-- yordamchi index
CREATE INDEX idx_orders_created_id ON orders (created_at DESC, id DESC);
```

`id`ni tiebreaker sifatida qo'shish bir xil `created_at` bo'lgan qatorlarda barqaror tartibni ta'minlaydi. Bu O(log n) index seek bo'lib, OFFSET 50000 dagi 50,020 qator skanlanishini yo'qotadi.

---

## 8-masala — Seq Scan'ni diagnostika qilish

`EXPLAIN ANALYZE`'da `Seq Scan on orders` ko'rsangiz, tekshiring:

1. **Mos index bormi?** `WHERE` ustunida index yo'q bo'lishi mumkin → yarating.
2. **Ustunga funksiya/cast qo'llanilganmi?** (`LOWER(...)`, `YEAR(...)`, implicit cast) — index'ni o'chiradi.
3. **Estimated vs actual rows** — agar juda farq qilsa, statistika eskirgan → `ANALYZE orders`.
4. **Query selektivmi?** Agar query qatorlarning katta qismini (masalan 80%) qaytarsa, planner ataylab Seq Scan tanlaydi — bu to'g'ri qaror, muammo emas.

---

## 9-masala — Index foydasiz holat (low selectivity)

**Index yordam bermaydi.** Sabab: query 50 million qatorning **90%**ini qaytaradi. Index orqali borib, keyin har bir mos qator uchun heap'ga qaytish (random I/O) 45 million marta — bu butun jadvalni ketma-ket o'qishdan (sequential I/O) ancha qimmat.

Planner buni tushunib, ataylab **Seq Scan** tanlaydi. Index faqat query **kam** qator qaytarganda (yuqori selektivlik, odatda < 5-10%) foydali.

**Agar baribir tezlatish kerak bo'lsa:** muammoni boshqacha modellashtirish kerak — masalan `total > 100` qatorlarni alohida partition'ga ajratish, yoki agregat natijani oldindan hisoblab denormalizatsiya qilish. Oddiy B-tree index bu yerda yechim emas.

---

← [Database bo'limiga qaytish](../../databases/README.md)
