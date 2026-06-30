# NoSQL

Relational database hamma muammoni yechmaydi. Katta hajm, egiluvchan schema va horizontal scaling kerak bo'lganda NoSQL paydo bo'ldi. Bu bo'limda NoSQL'ning 4 asosiy turi, CAP theorem, eventual consistency, sharding, hamda MongoDB, Redis va DynamoDB amaliyoti ko'rib chiqiladi.

## Mundarija

- [NoSQL nima va nega paydo bo'ldi](#nosql-nima-va-nega-paydo-boldi)
- [4 asosiy NoSQL turi](#4-asosiy-nosql-turi)
- [Document store (MongoDB)](#document-store-mongodb)
- [Key-value store (Redis, DynamoDB)](#key-value-store-redis-dynamodb)
- [Wide-column store (Cassandra, HBase)](#wide-column-store-cassandra-hbase)
- [Graph database (Neo4j)](#graph-database-neo4j)
- [CAP theorem NoSQL'da](#cap-theorem-nosqlda)
- [Eventual consistency va BASE](#eventual-consistency-va-base)
- [Horizontal scaling va sharding](#horizontal-scaling-va-sharding)
- [Schema-less: pros va cons](#schema-less-pros-va-cons)
- [Denormalizatsiya va data duplication](#denormalizatsiya-va-data-duplication)
- [SQL vs NoSQL qaror freymvorki](#sql-vs-nosql-qaror-freymvorki)
- [MongoDB chuqurroq](#mongodb-chuqurroq)
- [Redis data strukturalari va use-case'lar](#redis-data-strukturalari-va-use-caselar)
- [DynamoDB: partition/sort key va single-table design](#dynamodb-partitionsort-key-va-single-table-design)
- [Savol-javoblar (Q&A)](#savol-javoblar-qa)
- [Masalalar](#masalalar)

---

## NoSQL nima va nega paydo bo'ldi

**💡 Tushuncha:** NoSQL ("Not Only SQL") — relational bo'lmagan databaselar oilasi. Ular qat'iy jadval/schema va `JOIN`'larsiz, ko'pincha egiluvchan strukturada data saqlaydi.

Nega paydo bo'ldi (2000-yillar, web-scale davri):

- **Hajm (Volume):** Google, Amazon, Facebook'da data bitta serverga sig'maydigan darajada o'sdi → **horizontal scaling** kerak bo'ldi. Relational DB asosan vertical (kuchliroq server) scale qiladi.
- **Tezlik (Velocity):** Yuqori write throughput (millionlab event/sekund).
- **Xilma-xillik (Variety):** Yarim-strukturalangan data (JSON, log, sensor) — qat'iy schema noqulay.
- **Egiluvchanlik:** Tez o'zgaruvchan product'da har safar `ALTER TABLE` qilmasdan schema'ni rivojlantirish.

**⚠️ Ehtiyot bo'l:** NoSQL "SQL'dan yaxshiroq" degani emas. U boshqa trade-off'lar to'plami: ko'pincha scalability va egiluvchanlik uchun strict consistency va boy query (JOIN, ad-hoc) qurbon qilinadi.

---

## 4 asosiy NoSQL turi

| Tur | Misol | Data modeli | Asosiy use-case |
|-----|-------|-------------|------------------|
| **Document** | MongoDB, CouchDB | JSON-ga o'xshash document'lar | Kataloglar, profil, CMS, egiluvchan data |
| **Key-value** | Redis, DynamoDB | Kalit → qiymat | Cache, session, feature flag |
| **Wide-column** | Cassandra, HBase | Row key + ustun oilalari | Vaqtli seriyalar, log, ulkan write hajmi |
| **Graph** | Neo4j, ArangoDB | Node + edge (munosabatlar) | Ijtimoiy tarmoq, tavsiya, fraud detection |

---

## Document store (MongoDB)

**💡 Tushuncha:** Data **document** ko'rinishida saqlanadi (BSON — JSON'ning binar formati). Document'lar **collection**'larda guruhlanadi (jadval o'xshashi). Har bir document o'z strukturasiga ega bo'lishi mumkin.

```json
// "users" collection ichidagi bitta document
{
  "_id": "u1",
  "name": "Ramziddin",
  "email": "r@example.com",
  "address": { "city": "Tashkent", "zip": "100000" },
  "roles": ["admin", "editor"]
}
```

Querying:
```js
db.users.find({ "address.city": "Tashkent" });
db.users.find({ roles: "admin" });
db.users.updateOne({ _id: "u1" }, { $set: { name: "Ramz" } });
```

**Embedding vs Referencing** — bog'liq datani qanday saqlash:

| | Embedding (ichiga joylash) | Referencing (id orqali bog'lash) |
|---|----------------------------|----------------------------------|
| Ko'rinish | `post` ichida `comments: [...]` | `comments`da `post_id` saqlash |
| Read | 1 ta query — tez | JOIN/`$lookup` kerak |
| Yangilash | Butun document yangilanadi | Mustaqil yangilanadi |
| Qachon | "Birga o'qiladigan", cheklangan o'lcham (1:few) | Ko'p/o'sib boruvchi (1:many, many:many), mustaqil ishlatiladigan |

**✅ Javob:** Qoida: *"data birga ishlatilsa — embed; mustaqil yoki cheksiz o'ssa — reference."* MongoDB document'ning hajmi 16MB bilan cheklangan, shuning uchun cheksiz o'sadigan massivni embed qilmang.

---

## Key-value store (Redis, DynamoDB)

**💡 Tushuncha:** Eng sodda model: kalit → qiymat. Lookup O(1), juda tez. Qiymat ichini (odatda) query qilmaysiz — faqat kalit bo'yicha olasiz/qo'yasiz.

```
SET session:abc123  "{userId: 5, ...}"   EX 3600
GET session:abc123
```

Asosiy use-case — **caching**: og'ir DB query yoki hisob natijasini key-value'da saqlab, keyingi safar darrov qaytarish.

```
1. cache'ga qara (GET product:42)
2. bor bo'lsa → qaytar (cache hit)
3. yo'q bo'lsa → DB'dan ol, cache'ga yoz (SET ... EX 300), qaytar (cache miss)
```

Bu **cache-aside** pattern. Redis — in-memory (juda tez), DynamoDB — managed, disk-backed va massiv scale.

---

## Wide-column store (Cassandra, HBase)

**💡 Tushuncha:** Data row key bo'yicha indekslangan; har bir row turli ustunlarga ega bo'lishi mumkin, ustunlar "column family"larga guruhlanadi. Tashqaridan jadvalga o'xshaydi, lekin ichida — distributed, sparse, yozishga optimallangan struktura.

- **Cassandra** — masterless (peer-to-peer), juda yuqori write throughput, linear scaling, AP tizim. Vaqtli seriyalar, IoT, log, event store uchun ideal.
- Query **oldindan ma'lum access pattern bo'yicha** modellanishi shart (ad-hoc JOIN yo'q). Jadval query atrofida quriladi, data atrofida emas.

**⚠️ Ehtiyot bo'l:** Cassandra'da partition key tanlash kritik. Yomon tanlangan key "hot partition" (bitta node'ga ortiqcha yuk) yoki ulkan partition yaratadi.

---

## Graph database (Neo4j)

**💡 Tushuncha:** Data **node** (entity) va **edge** (relationship) ko'rinishida. Munosabatlar birinchi darajali fuqaro — ular ham xususiyatlarga ega bo'lishi mumkin.

```cypher
// Neo4j Cypher
(:Person {name:'Ali'})-[:FRIEND]->(:Person {name:'Vali'})

// "Ali do'stlarining do'stlarini top"
MATCH (a:Person {name:'Ali'})-[:FRIEND]->()-[:FRIEND]->(fof)
RETURN DISTINCT fof.name;
```

**❓ Qachon graph DB relational'dan yutadi?**

**✅ Javob:** Chuqur, ko'p bosqichli munosabatlar bo'yicha tez-tez yurish kerak bo'lganda. SQL'da "do'stlarning do'stlarining do'stlari" har bosqich uchun yana bir self-JOIN talab qiladi va eksponensial sekinlashadi. Graph DB'da bu shunchaki edge bo'yicha yurish — chuqurlikdan deyarli mustaqil. Misollar: ijtimoiy tarmoq, tavsiya tizimi, fraud detection, knowledge graph, marshrut.

---

## CAP theorem NoSQL'da

**💡 Tushuncha:** Distributed tizimda network bo'linishi (Partition) bo'lganda **Consistency** va **Availability**dan faqat bittasini tanlashingiz mumkin (ikkalasini emas).

- **CP** (Consistency + Partition tolerance): bo'linishda ba'zi node'lar javob bermaydi (xato qaytaradi), lekin hech qachon eski/noto'g'ri data bermaydi. Misol: MongoDB (default), HBase.
- **AP** (Availability + Partition tolerance): bo'linishda hamma node javob beradi, lekin data vaqtincha nomuvofiq (eski) bo'lishi mumkin. Misol: Cassandra, DynamoDB, Riak.

**⚠️ Ehtiyot bo'l:** Network partition real tizimda muqarrar, shuning uchun P'dan voz kechib bo'lmaydi. Demak amalda tanlov har doim **CP vs AP** orasida. "CA" faqat partition bo'lmaydigan (real bo'lmagan) holatda mavjud.

---

## Eventual consistency va BASE

**💡 Tushuncha:** ACID (relational) ning NoSQL muqobili — **BASE**:

- **B**asically **A**vailable — tizim doimo javob beradi.
- **S**oft state — holat tashqi kiritmasdan ham vaqt o'tishi bilan o'zgarishi mumkin.
- **E**ventual consistency — yangilanish hamma replikaga *oxir-oqibat* yetadi; bir muddat node'lar turli qiymat ko'rsatishi mumkin.

Misol: Amazon'da do'kon mavjudligini yangiladingiz; ba'zi region foydalanuvchilari bir necha soniya eski qiymatni ko'radi, keyin barchasi muvofiqlashadi.

**❓ Eventual consistency qachon maqbul?**

**✅ Javob:** Like soni, view counter, ijtimoiy feed, do'kon mavjudligi kabi — bir necha soniya nomuvofiqlik zarar qilmaydigan joylarda. Bank balansi, inventar rezervatsiyasi kabi joyda esa strong consistency (CP yoki ACID) kerak.

---

## Horizontal scaling va sharding

**💡 Tushuncha:**

- **Vertical scaling** — bitta serverni kuchaytirish (CPU/RAM). Cheklangan va qimmat.
- **Horizontal scaling** — ko'p serverga tarqatish. NoSQL bunga tug'ma moslashgan.

**Sharding** — datani shard key bo'yicha bir nechta node'ga ("shard") bo'lib joylash:

```
shard key = user_id
user_id % 4 → shard 0..3
```

Sharding strategiyalari:

| Strategiya | Qanday | Plus/Minus |
|------------|--------|-------------|
| **Hash-based** | `hash(key) % N` | Teng taqsimlanish; range query qiyin |
| **Range-based** | kalit oralig'i (A-M, N-Z) | Range query oson; hot shard xavfi |
| **Geo/directory** | mintaqa/lookup jadval | Egiluvchan; qo'shimcha boshqaruv |

**⚠️ Ehtiyot bo'l:** Yomon shard key "hot partition" (bir shard'ga nomutanosib yuk) yaratadi. Misol: `country`ni shard key qilsangiz, eng yirik mamlakat shard'i tiqilib qoladi. Yuqori cardinality va teng taqsimlanadigan key tanlang.

---

## Schema-less: pros va cons

**💡 Tushuncha:** Ko'p NoSQL DB schema'ni majburlamaydi — har bir document/row turlicha maydonlarga ega bo'lishi mumkin.

| Pros | Cons |
|------|------|
| Tez prototip, migratsiyasiz o'zgarish | "Schema baribir bor" — endi u application kodida |
| Yarim-strukturalangan/heterogen data | Nomuvofiq data (typo, yo'q maydon) xavfi |
| Har document mustaqil rivojlanadi | Validatsiya/integrity application zimmasida |
| Tez feature qo'shish | Eski va yangi document'larni birga handle qilish kerak |

**✅ Javob:** "Schema-less" aslida "schema-on-read": schema yo'qolmaydi, u DB'dan application'ga ko'chadi. Shuning uchun jiddiy loyihalarda baribir validatsiya (MongoDB `$jsonSchema`, application-level) qo'llanadi.

---

## Denormalizatsiya va data duplication

**💡 Tushuncha:** NoSQL'da JOIN qimmat yoki yo'q, shuning uchun datani **ataylab takrorlash** norma. Bitta read'da kerak bo'ladigan hamma narsa bitta document/itemda saqlanadi.

```json
// order document — customer ma'lumotini nusxalab saqlaymiz
{
  "_id": "o1",
  "customer": { "id": "c5", "name": "Ali", "city": "Tashkent" },
  "items": [ { "sku": "A1", "price": 10 } ],
  "total": 10
}
```

**Trade-off:** Read bitta query — tez. Lekin `customer.name` o'zgarsa, uni saqlangan barcha order'larda yangilash kerak (yoki eski snapshot sifatida qoldirish). NoSQL modellashtirish — read pattern atrofida quriladi: *"qaysi query'lar bo'ladi?"* degan savoldan boshlanadi.

---

## SQL vs NoSQL qaror freymvorki

**SQL (relational) tanlang, agar:**

- Data tabiatan strukturalangan va munosabatlarga boy (kuchli relational).
- Tranzaksiyalar va strong consistency muhim (moliya, buyurtma, inventar).
- Ad-hoc query, murakkab JOIN, reporting kerak.
- Schema barqaror.

**NoSQL tanlang, agar:**

- Massiv hajm va horizontal scaling kerak.
- Schema egiluvchan/o'zgaruvchan yoki yarim-strukturalangan.
- Access pattern oldindan ma'lum va sodda (kalit bo'yicha lookup).
- Yuqori write throughput yoki past-latency cache kerak.

**⚠️ Ehtiyot bo'l:** Ko'p real tizim **polyglot persistence** ishlatadi: asosiy data PostgreSQL, cache/session Redis, qidiruv Elasticsearch, hodisalar Cassandra. "Bitta DB hammaga" — kamdan-kam to'g'ri javob.

---

## MongoDB chuqurroq

**Document & collection:** yuqorida ko'rildi. `_id` har bir document uchun unique (default `ObjectId`).

**Index:**
```js
db.users.createIndex({ email: 1 }, { unique: true });   // 1 = ASC
db.orders.createIndex({ customer_id: 1, created_at: -1 }); // composite
```
MongoDB ham B-tree index ishlatadi va leftmost-prefix qoidasi shu yerda ham amal qiladi.

**Aggregation pipeline:** datani bosqichma-bosqich qayta ishlash (SQL'ning `GROUP BY`/`JOIN` muqobili):
```js
db.orders.aggregate([
  { $match:  { status: "paid" } },                  // WHERE
  { $group:  { _id: "$customer_id",
               total: { $sum: "$amount" } } },      // GROUP BY + SUM
  { $sort:   { total: -1 } },                       // ORDER BY
  { $limit:  10 }
]);
```

**Transaction:** MongoDB 4.0+ multi-document ACID transaction'ni qo'llab-quvvatlaydi (replica set kerak):
```js
const session = client.startSession();
session.startTransaction();
try {
  await accounts.updateOne({ _id: "a" }, { $inc: { bal: -100 } }, { session });
  await accounts.updateOne({ _id: "b" }, { $inc: { bal:  100 } }, { session });
  await session.commitTransaction();
} catch (e) {
  await session.abortTransaction();
}
```

**⚠️ Ehtiyot bo'l:** MongoDB transaction qimmat. To'g'ri document modeli (bog'liq datani birga embed qilish) ko'p hollarda transaction zaruriyatini yo'qotadi — bitta document yangilanishi atomik.

---

## Redis data strukturalari va use-case'lar

**💡 Tushuncha:** Redis shunchaki key-value emas — boy data strukturalar serveri (in-memory).

| Struktura | Misol komanda | Use-case |
|-----------|---------------|----------|
| **String** | `SET k v EX 300` | Cache, counter (`INCR`) |
| **Hash** | `HSET user:1 name Ali` | Obyekt/session field'lari |
| **List** | `LPUSH q job1` / `BRPOP` | Navbat (queue), oxirgi N element |
| **Set** | `SADD tags red` | Unikal to'plam, a'zolik tekshirish |
| **Sorted Set (ZSet)** | `ZADD board 100 ali` | **Leaderboard**, ranking, vaqtli oyna |
| **Stream** | `XADD events * ...` | Event log, pub/sub o'rnida durable |

Asosiy use-case'lar:

- **Cache** — eng keng tarqalgan, `EX` (TTL) bilan.
- **Session store** — stateless serverlar uchun markazlashgan session.
- **Rate limiting** — `INCR` + `EXPIRE`: bir oynada nechta so'rov.
  ```
  INCR rate:user:5
  EXPIRE rate:user:5 60   # 60s oynada limit
  ```
- **Pub/Sub** — real-time xabar tarqatish (chat, notification).
- **Leaderboard** — Sorted Set: `ZADD`, `ZREVRANGE board 0 9 WITHSCORES`.
- **Distributed lock** — `SET lock NX EX 10` (faqat mavjud bo'lmasa o'rnatish).

**⚠️ Ehtiyot bo'l:** Redis in-memory — RAM cheklovi bor va default'da to'liq durable emas. Muhim datani faqat Redis'da saqlamang; persistence (RDB/AOF) sozlamalarini bilib ishlating.

---

## DynamoDB: partition/sort key va single-table design

**💡 Tushuncha:** DynamoDB — AWS'ning managed key-value/document DB'si. Har bir item **primary key** orqali topiladi:

- **Partition key (PK)** — item qaysi fizik partition'ga tushishini belgilaydi (hash). Yolg'iz ishlatilsa — unique bo'lishi kerak.
- **Sort key (SK)** — ixtiyoriy; bir xil PK ichida item'larni tartiblaydi va range query imkonini beradi (`begins_with`, `between`).

```
PK = "USER#5",  SK = "PROFILE"      → user profili
PK = "USER#5",  SK = "ORDER#2026-01" → o'sha userning buyurtmasi
```

**Single-table design:** DynamoDB'da ko'pincha barcha entity'lar (user, order, product) **bitta jadvalda** PK/SK pattern'lari bilan saqlanadi. Maqsad — eng ko'p access pattern'ni bitta `Query` bilan, JOINsiz bajarish.

```
PK            | SK              | atributlar
USER#5        | PROFILE         | name, email
USER#5        | ORDER#1001      | total, status
ORDER#1001    | ITEM#A1         | sku, qty
```
`Query(PK = "USER#5")` → bitta foydalanuvchi va uning barcha order'larini bitta so'rovda qaytaradi.

**⚠️ Ehtiyot bo'l:** Single-table design access pattern'larni **oldindan** to'liq bilishni talab qiladi. Yangi, kutilmagan query paydo bo'lsa, ko'pincha Global Secondary Index (GSI) qo'shish kerak. Bu kuchli, lekin moslashuvchanligi past — model qiyin.

---

## Savol-javoblar (Q&A)

### ❓ NoSQL nima va nega paydo bo'ldi?

**✅ Javob:** Relational bo'lmagan databaselar oilasi. Web-scale data hajmi, yuqori write tezligi, yarim-strukturalangan data va horizontal scaling ehtiyoji tufayli paydo bo'ldi — relational DB asosan vertical scale qiladi.

### ❓ NoSQL'ning 4 asosiy turi va misollari?

**✅ Javob:** Document (MongoDB), key-value (Redis, DynamoDB), wide-column (Cassandra, HBase), graph (Neo4j). Har biri boshqa access pattern uchun optimallangan.

### ❓ MongoDB'da embedding va referencing qachon ishlatiladi?

**✅ Javob:** Embedding — data birga o'qiladigan va cheklangan o'lchamli bo'lsa (1:few). Referencing — data mustaqil yoki cheksiz o'sadigan bo'lsa (1:many, many:many). 16MB document limiti embedding chegarasini belgilaydi.

### ❓ Qachon graph database relational'dan afzal?

**✅ Javob:** Chuqur, ko'p bosqichli munosabatlar bo'yicha yurish kerak bo'lganda (do'stlarning do'stlari, tavsiya, fraud). SQL'da bu ko'p self-JOIN va eksponensial sekinlik, graph'da — oddiy edge traversal.

### ❓ CAP theorem nima va NoSQL'da qanday qo'llanadi?

**✅ Javob:** Network partition'da Consistency va Availability'dan faqat birini tanlash mumkin. MongoDB/HBase — CP (consistency), Cassandra/DynamoDB — AP (availability). Partition muqarrar, shuning uchun tanlov CP vs AP.

### ❓ BASE nima va ACID'dan farqi?

**✅ Javob:** BASE = Basically Available, Soft state, Eventual consistency. ACID strong consistency va atomiclik beradi; BASE availability va scaling uchun vaqtinchalik nomuvofiqlikka yo'l qo'yadi.

### ❓ Eventual consistency qachon maqbul, qachon emas?

**✅ Javob:** Maqbul: like soni, view counter, feed, mavjudlik — qisqa nomuvofiqlik zarar qilmaydi. Emas: bank balansi, inventar rezervatsiyasi — strong consistency shart.

### ❓ Sharding nima va shard key qanday tanlanadi?

**✅ Javob:** Datani shard key bo'yicha bir nechta node'ga bo'lib joylash. Key yuqori cardinality va teng taqsimlanadigan bo'lishi kerak; aks holda "hot partition" yuzaga keladi.

### ❓ Schema-less'ning afzallik va kamchiligi?

**✅ Javob:** Afzallik — egiluvchanlik, migratsiyasiz o'zgarish, heterogen data. Kamchilik — schema application kodiga ko'chadi, validatsiya/integrity mas'uliyati siz zimmangizda, nomuvofiq data xavfi.

### ❓ NoSQL'da denormalizatsiya nega norma?

**✅ Javob:** JOIN qimmat yoki yo'q, shuning uchun bitta read'da kerak bo'ladigan hamma narsa bitta document/itemga nusxalab saqlanadi. Buning evaziga write murakkablashadi va data takrorlanadi.

### ❓ SQL yoki NoSQL — qanday tanlaysiz?

**✅ Javob:** SQL — strukturalangan, relational, tranzaksion data va ad-hoc query uchun. NoSQL — massiv scale, egiluvchan schema, oldindan ma'lum sodda access pattern, yuqori throughput uchun. Ko'pincha aralash (polyglot persistence).

### ❓ Redis qaysi use-case'larda ishlatiladi?

**✅ Javob:** Cache (TTL bilan), session store, rate limiting (`INCR`+`EXPIRE`), pub/sub, leaderboard (Sorted Set), distributed lock. In-memory bo'lgani uchun juda tez.

### ❓ MongoDB aggregation pipeline nima?

**✅ Javob:** Datani bosqichma-bosqich (`$match`, `$group`, `$sort`, `$limit`, `$lookup`) qayta ishlash mexanizmi — SQL'ning `WHERE`/`GROUP BY`/`ORDER BY`/`JOIN` muqobili.

### ❓ MongoDB transaction'ni qo'llab-quvvatlaydimi?

**✅ Javob:** Ha, 4.0+ dan multi-document ACID transaction (replica set kerak). Lekin u qimmat; to'g'ri document modeli ko'p hollarda transaction zaruriyatini yo'qotadi (bitta document yangilanishi atomik).

### ❓ DynamoDB'da partition key va sort key nima?

**✅ Javob:** Partition key item qaysi partition'ga tushishini (hash) belgilaydi; sort key bir xil PK ichida item'larni tartiblaydi va range query beradi. Birgalikda composite primary key tashkil qiladi.

### ❓ Single-table design nima?

**✅ Javob:** DynamoDB'da barcha entity'larni bitta jadvalda PK/SK pattern'lari bilan saqlash — eng ko'p access pattern'ni bitta `Query` bilan, JOINsiz bajarish uchun. Access pattern'larni oldindan bilishni talab qiladi.

---

## Masalalar

> Yechimlar: [solutions/databases/06-nosql.md](../solutions/databases/06-nosql.md)

1. Quyidagi har bir senariy uchun eng mos NoSQL turini (document / key-value / wide-column / graph) tanlang va bir jumlada asoslang:
   - a) Foydalanuvchi sessiyalarini tez saqlash/o'qish.
   - b) Ijtimoiy tarmoqda "umumiy do'stlar" hisoblash.
   - c) IoT sensorlaridan sekundiga millionlab o'lchov yozish.
   - d) Turli maydonli mahsulot katalogi (CMS).

2. E-commerce blog uchun `post` va `comment`ni MongoDB'da modellashtiring. Embedding yoki referencing? Qaroringizni asoslang (post'da minglab comment bo'lishi mumkin).

3. Bitta jumlada ayting: Cassandra (AP) va MongoDB default (CP) — CAP nuqtai nazaridan qaysi biri network partition'da xato qaytaradi, qaysi biri eski data qaytaradi?

4. Redis bilan "bir foydalanuvchi daqiqada eng ko'pi bilan 100 so'rov" rate limiter'ini komanda(lar) bilan yozing.

5. Redis Sorted Set bilan o'yin leaderboard'i tuzing: ballni yangilash va top-10 ni olish komandalarini yozing.

6. Quyidagi SQL query'ni MongoDB aggregation pipeline'ga aylantiring:
   ```sql
   SELECT customer_id, SUM(amount) AS total
   FROM orders WHERE status = 'paid'
   GROUP BY customer_id ORDER BY total DESC LIMIT 5;
   ```

7. DynamoDB single-table design'da bitta `User` va uning bir nechta `Order`larini saqlash uchun PK/SK pattern'ini taklif qiling va `Query` qanday ishlashini tushuntiring.

8. Quyidagi loyiha uchun SQL yoki NoSQL tanlang va asoslang: bank ko'chirma tizimi — balans, tranzaksiya, qat'iy consistency.

9. NoSQL document'da `customer.name` denormalizatsiya qilib saqlangan. Foydalanuvchi ismini o'zgartirsa, qanday muammo yuzaga keladi va uni hal qilishning ikki yo'lini ayting.

10. "Schema-less degani validatsiya kerak emas" — bu fikr nega xato? Tushuntiring.

---

← [Database bo'limiga qaytish](./README.md)
