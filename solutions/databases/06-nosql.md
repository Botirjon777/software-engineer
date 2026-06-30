# NoSQL — Masalalar yechimi

[← Mavzuga qaytish](../../databases/06-nosql.md)

---

## 1-masala — Senariy uchun NoSQL turini tanlash

- **a) Sessiyalarni tez saqlash/o'qish** → **Key-value (Redis).** Kalit bo'yicha O(1) lookup, TTL bilan avtomatik tugash; in-memory tezligi sessiyaga ideal.
- **b) "Umumiy do'stlar"** → **Graph (Neo4j).** Munosabatlar bo'yicha yurish (relationship traversal) graph'ning kuchli tomoni; SQL'da bu ko'p self-JOIN va sekin.
- **c) Sekundiga millionlab sensor o'lchovi** → **Wide-column (Cassandra).** Juda yuqori write throughput, linear scaling, vaqtli seriyalar uchun optimallangan.
- **d) Turli maydonli mahsulot katalogi** → **Document (MongoDB).** Egiluvchan schema har bir mahsulotning turli atributlarini bir document'da saqlashga imkon beradi.

---

## 2-masala — Post va comment modeli (MongoDB)

**Qaror: Referencing** (comment'larni alohida collection'da saqlash).

```js
// posts collection
{ "_id": "p1", "title": "...", "body": "...", "author": "u5" }

// comments collection
{ "_id": "c1", "post_id": "p1", "author": "u9", "text": "...", "created_at": ... }
```

**Asos:** Post'da minglab comment bo'lishi mumkin — bu cheksiz o'sadigan massiv. Agar comment'larni post ichiga embed qilsak:
- 16MB document limitiga yaqinlashamiz;
- har bir yangi comment butun katta document'ni qayta yozadi;
- pagination (comment'larni 20 tadan ko'rsatish) qiyinlashadi.

Referencing'da comment'lar mustaqil yoziladi va `db.comments.find({ post_id: "p1" }).limit(20)` bilan pagination qulay. `post_id` ustiga index qo'yiladi.

---

## 3-masala — CAP: kim xato, kim eski data qaytaradi

- **MongoDB (CP)** — network partition'da consistency'ni saqlaydi, shuning uchun ba'zi node'lar **xato/javobsiz** qoladi (availability'dan voz kechadi).
- **Cassandra (AP)** — availability'ni saqlaydi, shuning uchun javob beradi, lekin **eski (stale) data** qaytarishi mumkin (consistency'dan voz kechadi).

---

## 4-masala — Redis rate limiter

```
INCR rate:user:5
EXPIRE rate:user:5 60     # birinchi INCR'dan keyin oyna boshlanadi
```

Application logikasi:
```js
const count = await redis.incr(`rate:user:${id}`);
if (count === 1) await redis.expire(`rate:user:${id}`, 60); // yangi oyna
if (count > 100) throw new Error('Rate limit exceeded'); // daqiqada 100
```

`INCR` atomik, shuning uchun concurrent so'rovlarda ham to'g'ri sanaydi. `EXPIRE`ni faqat birinchi so'rovda (count === 1) qo'yamiz, aks holda oyna har safar uzayib ketadi.

---

## 5-masala — Redis leaderboard (Sorted Set)

```
# ballni o'rnatish/yangilash
ZADD leaderboard 1500 "ali"
ZADD leaderboard 2300 "vali"

# ballni oshirish
ZINCRBY leaderboard 50 "ali"

# top-10 (eng yuqori balldan past tomon), ball bilan
ZREVRANGE leaderboard 0 9 WITHSCORES

# bitta o'yinchining o'rni (0-indeksli)
ZREVRANK leaderboard "ali"
```

Sorted Set elementlarni score bo'yicha avtomatik tartiblab saqlaydi, shuning uchun ranking va range so'rovlar O(log n).

---

## 6-masala — SQL → MongoDB aggregation pipeline

```js
db.orders.aggregate([
  { $match: { status: "paid" } },                          // WHERE status='paid'
  { $group: { _id: "$customer_id",                          // GROUP BY customer_id
              total: { $sum: "$amount" } } },               // SUM(amount)
  { $sort:  { total: -1 } },                                // ORDER BY total DESC
  { $limit: 5 }                                             // LIMIT 5
]);
```

`$group` natijasida `_id` — guruhlash kaliti (`customer_id`), `total` — yig'indi.

---

## 7-masala — DynamoDB single-table design

```
PK         | SK            | atributlar
USER#5     | PROFILE       | name, email
USER#5     | ORDER#1001    | total, status, created_at
USER#5     | ORDER#1002    | total, status, created_at
```

**Query ishlashi:**
```
Query: PK = "USER#5"
→ "USER#5" partition'idagi BARCHA item'larni qaytaradi:
   profil + barcha order'lar, bitta so'rovda, JOINsiz.
```

Agar faqat order'lar kerak bo'lsa: `PK = "USER#5" AND begins_with(SK, "ORDER#")`. Sort key prefiks pattern'i bir partition ichida entity turlarini ajratadi va range query beradi.

---

## 8-masala — SQL vs NoSQL: bank tizimi

**Tanlov: SQL (relational, masalan PostgreSQL).**

**Asos:**
- Bank ko'chirmasi **ACID transaction** talab qiladi — pul bir hisobdan chiqib, ikkinchisiga kirishi atomik bo'lishi shart (ikkalasi yoki hech biri).
- **Strong consistency** majburiy — balansni hech qachon eski/noto'g'ri ko'rsatib bo'lmaydi (eventual consistency mos kelmaydi).
- Data tabiatan relational (hisoblar, tranzaksiyalar, mijozlar) va integrity constraint'lar (`CHECK balance >= 0`, foreign key) muhim.

NoSQL'ning eventual consistency va denormalizatsiya modeli bu yerda xavfli. (Eslatma: Cassandra/DynamoDB ham strong consistency rejimini taklif qiladi, lekin relational integrity va murakkab tranzaksiya uchun SQL tabiiyroq.)

---

## 9-masala — Denormalizatsiya muammosi va yechimi

**Muammo:** `customer.name` ko'plab order document'lariga nusxalangan. Foydalanuvchi ismini o'zgartirsa, eski order'lardagi `customer.name` eskirib qoladi — data nomuvofiqligi.

**Yechim 1 — sinxronlash (fan-out update):** ism o'zgarganda barcha bog'liq order'larni yangilash. Background job yoki event-driven (CDC/queue) orqali. Eksilik: ko'p document yangilash, vaqtinchalik nomuvofiqlik.

**Yechim 2 — snapshot sifatida qoldirish:** order'dagi ism — buyurtma vaqtidagi tarixiy nusxa deb qabul qilish; uni umuman yangilamaslik. Joriy ismni esa faqat `customers` collection'idan, kerak bo'lganda reference orqali olish.

Tanlov biznes talabiga bog'liq: order tarixiy yozuv bo'lsa — snapshot to'g'ri.

---

## 10-masala — "Schema-less = validatsiya kerak emas" nega xato

Schema-less DB schema'ni **majburlamaydi**, lekin data baribir application kodida ma'lum strukturaga ega bo'lishi shart — kod `user.email` maydonini kutadi. Bu "schema-on-read": schema yo'qolmaydi, u DB'dan application'ga ko'chadi.

Validatsiyasiz:
- Typo'li yoki yo'q maydonlar (`emial`, yo'q `email`) bilan buzuq document'lar to'planadi;
- Turli versiyali document'lar (eski/yangi) handle qilinmay xatolik beradi;
- Data integrity hech kim tomonidan ta'minlanmaydi.

Shuning uchun jiddiy loyihalarda validatsiya qo'llanadi: MongoDB `$jsonSchema` validator, yoki application-level (Mongoose, Zod). Schema-less — moslashuvchanlik, validatsiyadan voz kechish emas.

---

← [Database bo'limiga qaytish](../../databases/README.md)
