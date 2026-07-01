# Distributed Systems

Taqsimlangan tizim — bir nechta mustaqil kompyuter bir maqsad uchun tarmoq orqali ishlaganda, foydalanuvchiga esa **bitta tizim** kabi ko'rinadi. Bu miqyos va ishonchlilik beradi, lekin tubdan qiyin: tarmoq ishonchsiz, kechikish bor, va bir qism boshqasidan xabarsiz o'lishi mumkin (partial failure). Bu hujjat distributed systems asoslarini O'zbek tilida, ingliz atamalari bilan senior darajada ko'rib chiqadi.

Biz quyidagilarni yoritamiz: nega distributed qiyin ("fallacies of distributed computing"), CAP theorem chuqur (CP vs AP), PACELC, consistency modellari (strong, eventual, causal, read-your-writes), **consensus** (Paxos, Raft — leader election va log replication), quorum (N/W/R), replication turlari (leader-follower, multi-leader, leaderless/Dynamo), conflict resolution (LWW, vector clocks, CRDT), partitioning/sharding (range/hash/consistent hashing), distributed transactions (2PC va saga), idempotency va exactly-once muammosi, distributed clocks (Lamport, vector clock), gossip protocol va split-brain. Amaliy misollar bilan.

## Mundarija

- [Distributed system nima va nega qiyin](#distributed-system-nima-va-nega-qiyin)
- [Fallacies of distributed computing](#fallacies-of-distributed-computing)
- [CAP theorem](#cap-theorem)
- [PACELC](#pacelc)
- [Consistency modellari](#consistency-modellari)
- [Consensus: nega kerak](#consensus-nega-kerak)
- [Raft algoritmi](#raft-algoritmi)
- [Quorum (N/W/R)](#quorum-nwr)
- [Replication turlari](#replication-turlari)
- [Conflict resolution](#conflict-resolution)
- [Partitioning va sharding](#partitioning-va-sharding)
- [Distributed transactions: 2PC va Saga](#distributed-transactions-2pc-va-saga)
- [Idempotency va exactly-once](#idempotency-va-exactly-once)
- [Distributed clocks](#distributed-clocks)
- [Gossip protocol va split-brain](#gossip-protocol-va-split-brain)
- [❓ Savol-javoblar](#-savol-javoblar)
- [Masalalar](#masalalar)

---

## Distributed system nima va nega qiyin

**💡 Tushuncha:** Distributed system — tarmoq orqali xabar almashib, umumiy maqsadga ishlaydigan mustaqil node'lar to'plami. Foydalanuvchi buni yaxlit tizim deb ko'radi. Misol: Google Search, kubernetes klasteri, Cassandra DB.

**💡 Tushuncha:** Nega qiyin? Uch fundamental muammo:

```text
1. NETWORK UNRELIABLE — paket yo'qoladi, kechikadi, qayta keladi.
   So'rov yetdimi? Javob yo'qoldimi? — BILIB BO'LMAYDI.

2. LATENCY — mahalliy chaqiruv ns, tarmoq chaqiruvi ms (10^6x sekin).
   Xotiraga murojaat ≠ boshqa mashinaga murojaat.

3. PARTIAL FAILURE — bir node o'ladi, boshqalari ishlaydi.
   Monolithda: hamma ishlaydi YOKI hamma o'ladi. Distributed'da: oraliq.
```

**⚠️ Ehtiyot bo'l:** Partial failure — eng zarali. Node A, B'ga so'rov yubordi va javob olmadi. B o'ldimi? Sekinlashdimi? Javob yo'ldami? A buni **hech qachon aniq bilmaydi** — faqat timeout bilan taxmin qiladi. Barcha distributed muammolar shu noaniqlikdan kelib chiqadi.

---

## Fallacies of distributed computing

**💡 Tushuncha:** L. Peter Deutsch va boshqalar (Sun) sanab bergan 8 ta yolg'on taxmin — yangi muhandislar shularga ishonib xato qiladi:

```text
1. Tarmoq ishonchli (reliable)         → YO'Q, paket yo'qoladi
2. Latency nol                          → YO'Q, ms'lar bor
3. Bandwidth cheksiz                    → YO'Q, chegara bor
4. Tarmoq xavfsiz                       → YO'Q, hujum bor
5. Topologiya o'zgarmaydi               → YO'Q, node'lar kelib-ketadi
6. Bitta administrator bor              → YO'Q, ko'p jamoa
7. Transport narxi nol                  → YO'Q, serializatsiya/tarmoq qiymati
8. Tarmoq bir jinsli (homogeneous)      → YO'Q, turli HW/SW
```

**💡 Tushuncha:** Har bir yolg'on real dizaynda muammo keltiradi. "Tarmoq ishonchli" deb retry qo'ymaslik → ma'lumot yo'qoladi. "Latency nol" deb ko'p mayda chaqiruv → tizim sekinlashadi (chatty API). Distributed dizaynda har doim shu 8 tani hisobga ol.

---

## CAP theorem

**💡 Tushuncha:** CAP theorem (Eric Brewer) — distributed tizimda **uchtadan faqat ikkitasini** bir vaqtda kafolatlash mumkin:

| Harf | Nomi | Ma'nosi |
|------|------|---------|
| **C** | Consistency | Har o'qish eng oxirgi yozishni ko'radi (yoki xato) |
| **A** | Availability | Har so'rov (xatosiz) javob oladi |
| **P** | Partition tolerance | Tarmoq bo'linsa ham ishlaydi |

**💡 Tushuncha:** Aslida tanlov **C vs A** (P majburiy). Chunki tarmoq partition (bo'linish) distributed tizimda **muqarrar** — uni tanlab bo'lmaydi. Partition sodir bo'lganda:
- **CP tanlaysan**: consistency saqlab, ba'zi so'rovlarni rad qilasan (available emas).
- **AP tanlaysan**: javob berasan, lekin ma'lumot eskirgan (stale) bo'lishi mumkin.

```text
   Node A  ╳╳╳ tarmoq uzildi ╳╳╳  Node B
   (yozildi: x=5)                  (eski: x=3)

   Client B'ga o'qish so'rovi yubordi:
   ┌─────────────────────────────────────┐
   │ CP: "bilmayman, A bilan bog'lanolmadim" → xato (A sacrifice)
   │ AP: "x=3" (eski, lekin javob bor)         → stale (C sacrifice)
   └─────────────────────────────────────┘
```

| Tanlov | Misol tizimlar | Qachon |
|--------|----------------|--------|
| **CP** | HBase, MongoDB (default), Zookeeper, etcd | Bank, inventar, to'g'rilik kritik |
| **AP** | Cassandra, DynamoDB, Riak, CouchDB | Ijtimoiy feed, savat, doim javob kerak |

**⚠️ Ehtiyot bo'l:** Partition **yo'q** paytda CAP hech narsa demaydi — tizim ham C, ham A bo'la oladi. CAP faqat partition paytidagi tanlovni tavsiflaydi. Shuning uchun PACELC to'liqroq.

---

## PACELC

**💡 Tushuncha:** CAP faqat partition holatini ko'radi. PACELC (Daniel Abadi) to'ldiradi: partition **bo'lmaganda ham** tanlov bor — latency vs consistency.

```text
PAC-ELC:
  agar Partition (P) bo'lsa  → Availability yoki Consistency? (A/C)
  Else (E) — normal holatda  → Latency yoki Consistency?      (L/C)
```

| Tizim | PACELC | O'qilishi |
|-------|--------|-----------|
| Cassandra, Dynamo | PA/EL | Partition'da A, normalda L (tezlik uchun consistency'ni qurbon) |
| MongoDB | PA/EC | Partition'da A, normalda C |
| HBase, BigTable | PC/EC | Doim consistency (partition'da ham, normalda ham) |
| PostgreSQL (single) | PC/EC | Doim consistency |

**💡 Tushuncha:** PACELC eslatadi: tanlov faqat falokatda emas. **Har kuni**, har so'rovda consistency (kuchli mos, lekin sekinroq — koordinatsiya kerak) yoki latency (tez, lekin ehtimol eskirgan) o'rtasida tanlaysan.

---

## Consistency modellari

**💡 Tushuncha:** "Consistency" bitta narsa emas — bu spektr. Kuchli (qat'iy, sekin) dan zaif (bo'sh, tez) gacha.

```text
KUCHLI ◄─────────────────────────────────────► ZAIF
Linearizable  Sequential  Causal  Read-your-writes  Eventual
(qat'iy)                                            (bo'sh, tez)
```

| Model | Kafolat | Misol |
|-------|---------|-------|
| **Strong (linearizable)** | Har o'qish eng oxirgi yozishni ko'radi, real vaqt tartibida | Bank balansi |
| **Eventual** | Yozishlar to'xtasa, oxir-oqibat hamma bir xil qiymatga keladi | DNS, ijtimoiy like |
| **Causal** | Sababiy bog'liq hodisalar tartibda ko'rinadi (sabab → oqibat) | Chat: savoldan oldin javob ko'rinmasin |
| **Read-your-writes** | O'z yozganingni darrov o'qiy olasan | Profilni tahrirlash: o'zgarish darrov ko'rinsin |

**💡 Tushuncha:**
- **Strong consistency** koordinatsiya (consensus/quorum) talab qiladi — sekin, lekin sodda mental model.
- **Eventual consistency** — replica'lar vaqtincha farq qiladi, keyin "yaqinlashadi" (converge). Tez va available, lekin dasturchi vaqtinchalik nomuvofiqlikni boshqarishi kerak.
- **Causal** — o'rta yo'l: sababiy tartibni saqlaydi, lekin bog'liqmas hodisalarni erkin qoldiradi.
- **Read-your-writes** — foydalanuvchi tajribasi uchun muhim (sticky session yoki o'z yozuvni cache'lash bilan).

**⚠️ Ehtiyot bo'l:** "Eventual consistency" — "oxir-oqibat" qancha vaqt? Odatda ms, lekin partition'da soniyalar/daqiqalar bo'lishi mumkin. Foydalanuvchi eski ma'lumot ko'rishga tayyor bo'lishi kerak (masalan like soni bir oz orqada).

---

## Consensus: nega kerak

**💡 Tushuncha:** Consensus — bir nechta node bir qiymat (yoki qaror) bo'yicha **kelishishi**, ba'zilari o'lgan yoki sekin bo'lsa ham. Bu distributed tizimning yuragi: leader tanlash, tartibli log, atomik commit — hammasi consensus'ga tayanadi.

**Nega qiyin:** Node'lar o'ladi, xabar yo'qoladi, kechikadi. Barchasi bir vaqtda "ha" deyishini kafolatlash — nontrivial. FLP theorem: to'liq asinxron tizimda bitta node o'lsa ham, consensus'ni kafolatlab bo'lmaydi (amalda timeout bilan yon aylanadi).

**💡 Tushuncha: Paxos** (Leslie Lamport) — birinchi isbotlangan consensus algoritmi. Rollar: proposer (taklif qiladi), acceptor (ovoz beradi), learner (natijani biladi). Ikki fazali: prepare/promise, keyin accept/accepted. **Kuchli, lekin tushunish va to'g'ri amalga oshirish juda qiyin** — shuning uchun Raft yaratildi.

---

## Raft algoritmi

**💡 Tushuncha:** Raft — Paxos bilan bir xil kuchga ega, lekin **tushunarli** bo'lish uchun dizayn qilingan consensus algoritmi. Uni ikki qismga bo'ladi: **leader election** (leader tanlash) va **log replication** (jurnalni ko'chirish). etcd, Consul, TiKV Raft ishlatadi.

Har node uch holatdan birida:

```text
   timeout (leader'dan xabar yo'q)
  ┌─────────────────────────────────┐
  ▼                                 │
┌──────────┐  ovoz to'pladi  ┌──────────┐
│ CANDIDATE│ ──────────────► │  LEADER  │
└──────────┘                 └──────────┘
  ▲     │ boshqa leader topildi   │ heartbeat
  │     ▼                         ▼
  │  ┌──────────┐  ◄──────────────┘
  └──│ FOLLOWER │  (leader'ga bo'ysunadi)
     └──────────┘
```

### Leader election

**💡 Tushuncha:** Vaqt **term** (davr) larga bo'linadi. Har term'da ko'pi bilan bitta leader.

```text
1. Follower leader'dan heartbeat kutadi.
2. Timeout (tasodifiy 150-300ms) → Candidate bo'ladi, term++.
3. O'ziga ovoz beradi va boshqalardan ovoz so'raydi (RequestVote).
4. Majority (N/2+1) ovoz → LEADER bo'ladi.
5. Leader muntazam heartbeat yuboradi → boshqalar follower qoladi.
```

Tasodifiy timeout ikki node bir vaqtda candidate bo'lib "split vote" bo'lishini kamaytiradi.

### Log replication

**💡 Tushuncha:** Leader barcha yozishlarni qabul qiladi, log'ga qo'yadi va follower'larga ko'chiradi (AppendEntries).

```text
1. Client → Leader: "x=5 yoz"
2. Leader o'z log'iga qo'yadi (uncommitted)
3. Follower'larga AppendEntries yuboradi
4. Majority (N/2+1) tasdiqladi → "committed"
5. Leader state machine'ga qo'llaydi, client'ga OK
6. Keyingi heartbeat'da follower'lar ham commit qiladi
```

**💡 Tushuncha:** Majority (quorum) sharti tufayli, hatto leader o'lsa ham, yangi leader har doim eng yangi commit qilingan log'ga ega bo'ladi (unda eng ko'p ovoz bor). Shu tarzda ma'lumot yo'qolmaydi. 5 node'da 3 kelishsa yetadi — 2 tasi o'lsa ham ishlaydi.

**⚠️ Ehtiyot bo'l:** Raft toq (odd) sonli node bilan ishlatiladi (3, 5, 7). 5 node → 2 nosozlikka chidaydi. Juft son foydasiz: 4 node ham 2 nosozlikka chidaydi (majority=3), lekin ko'proq mashina sarflaydi.

---

## Quorum (N/W/R)

**💡 Tushuncha:** Quorum — ma'lum bir amal uchun **kelishuvchi node'larning minimal soni**. Leaderless replikatsiyada (Dynamo) o'qish/yozish quorumi orqali consistency sozlanadi.

```text
N = replika soni
W = yozish tasdig'i uchun kerakli node soni
R = o'qish uchun so'raladigan node soni

Agar W + R > N  → strong consistency (yozish/o'qish to'plamlari kesishadi)
```

```text
Misol: N=3, W=2, R=2  →  W+R=4 > 3 ✓
   yozish 2 node'ga borsa, o'qish 2 node'dan so'ralsa,
   kamida 1 ta umumiy node bor → eng yangi qiymat topiladi.

N=3, W=1, R=1  →  W+R=2 < 3 ✗  → tez, lekin eventual (stale mumkin)
```

| Sozlash | Xususiyat |
|---------|-----------|
| W=N, R=1 | Yozish sekin, o'qish tez (read-heavy) |
| W=1, R=N | Yozish tez, o'qish sekin (write-heavy) |
| W=R=(N/2+1) | Balanslangan, strong consistency |

**💡 Tushuncha:** W va R'ni oshirish consistency'ni oshiradi, lekin latency va availability'ni pasaytiradi. Bu CAP tanlovining amaliy "sozlagichi" — bitta DB'da so'rov bo'yicha o'zgartirish mumkin.

---

## Replication turlari

**💡 Tushuncha:** Replication — ma'lumotni bir nechta node'da saqlash. Uch asosiy arxitektura:

### Leader-follower (single-leader)

```text
        yozish
Client ────────► ┌────────┐  replikatsiya  ┌──────────┐
                 │ LEADER │ ──────────────► │ FOLLOWER │
                 └────────┘                 └──────────┘
                     ▲                          │ o'qish
   o'qish ◄──────────┘◄─────────────────────────┘
```

**💡 Tushuncha:** Barcha yozishlar bitta leader'ga, u follower'larga ko'chiradi. O'qish istalgan node'dan. Sodda, konfliktsiz. Kamchilik: leader — yozish uchun SPOF (failover kerak), va async replikatsiyada follower orqada qolishi mumkin (replication lag → read-your-writes buziladi).

- **Sync replikatsiya**: leader follower tasdig'ini kutadi (durable, sekin).
- **Async**: kutmaydi (tez, lekin leader o'lsa oxirgi yozish yo'qolishi mumkin).

### Multi-leader

**💡 Tushuncha:** Bir nechta node yozishni qabul qiladi (masalan har region'da bitta leader). Geo-taqsimot, offline ishlash uchun yaxshi. Muammo: **konflikt** — ikki leader bir yozuvni bir vaqtda o'zgartirsa, kelishtirish kerak (conflict resolution).

### Leaderless (Dynamo uslubi)

**💡 Tushuncha:** Leader yo'q — client (yoki koordinator) yozishni bir nechta replica'ga to'g'ridan-to'g'ri yuboradi, o'qishda quorum yig'adi. Cassandra, DynamoDB, Riak shunday. Yuqori availability (AP). Konsistensiya quorum (N/W/R) bilan boshqariladi. Orqada qolgan node'lar **read repair** va **anti-entropy** (gossip) bilan tuzatiladi.

| Tur | Yozish | Konflikt | Misol |
|-----|--------|----------|-------|
| Single-leader | Bitta node | Yo'q | PostgreSQL, MySQL replikatsiya |
| Multi-leader | Bir necha node | Bor | CouchDB, multi-region setup |
| Leaderless | Istalgan node | Bor | Cassandra, DynamoDB |

---

## Conflict resolution

**💡 Tushuncha:** Multi-leader va leaderless'da ikki nusxa bir vaqtda o'zgarishi mumkin — konfliktni hal qilish kerak.

### Last-Write-Wins (LWW)

**💡 Tushuncha:** Har yozuvga timestamp qo'yiladi, konfliktda **eng kech** yozuv g'olib. Oddiy, lekin **ma'lumot yo'qotadi** (yo'qotilgan yozuv abadiy ketadi) va soatlar farq qilsa (clock skew) noto'g'ri g'olib tanlaydi.

### Vector clocks

**💡 Tushuncha:** Har node uchun hisoblagich saqlab, ikki versiya **sababiy bog'liqmi** yoki **haqiqiy konfliktmi** aniqlaydi. Konflikt bo'lsa — ikkalasini ham saqlab, ilovaga (yoki foydalanuvchiga) hal qilishni topshiradi ("sibling" versiyalar).

```text
Node A: [A:2, B:1]   Node B: [A:1, B:2]
  Hech biri ikkinchisining "ajdodi" emas → KONFLIKT (concurrent)
  → ikkalasini saqla, merge qil (masalan savatlar birlashadi)
```

### CRDT (Conflict-free Replicated Data Types)

**💡 Tushuncha:** CRDT — shunday ma'lumot strukturasi, uni **koordinatsiyasiz** yangilash mumkin va replica'lar avtomatik, ziddiyatsiz **yaqinlashadi** (converge). G'oya: operatsiyalar kommutativ (tartibi muhim emas) va idempotent bo'lsin.

```text
Misollar:
  G-Counter  → faqat oshadigan hisoblagich (har node o'z qismini oshiradi, yig'indi)
  OR-Set     → element qo'shish/o'chirishni unique tag bilan (konfliktsiz merge)
  LWW-Register → LWW registr

Ishlatiladi: Redis CRDT, Riak, kollaborativ tahrir (Automerge, Yjs)
```

**💡 Tushuncha:** CRDT konfliktni **oldindan** hal qiladi (matematik struktura tufayli har doim converge bo'ladi), LWW/vector clocks esa **keyin** hal qiladi. CRDT — offline-first va real-time collab (Google Docs uslubi) uchun ideal, lekin har ma'lumot turi CRDT bo'la olmaydi.

---

## Partitioning va sharding

**💡 Tushuncha:** Partitioning (sharding) — katta ma'lumotni bo'laklarga (shard) bo'lib, turli node'larga tarqatish. Sabab: bitta node barcha ma'lumot/yukni ko'tara olmaydi. Maqsad: yuk **teng** taqsimlansin, "hot spot" bo'lmasin.

### Range partitioning

```text
user_id 1     - 1M   → Shard A
user_id 1M+1  - 2M   → Shard B
```
Diapazon so'rovlar oson (`WHERE id BETWEEN`), lekin **hot spot** xavfi (yangi user'lar hammasi oxirgi shard'ga).

### Hash partitioning

```text
shard = hash(user_id) % N
```
Teng taqsimlaydi, lekin diapazon so'rov qiyin. Va **N o'zgarsa** (node qo'shish) deyarli hamma kalit ko'chadi (`% N` o'zgardi) — bu resharding falokati.

### Consistent hashing

**💡 Tushuncha:** Hash halqasi (ring) — node va kalitlar bir halqaga joylashadi. Kalit soat yo'nalishida eng yaqin node'ga tegishli. Node qo'shilsa/o'chsa — faqat **qo'shni** kalitlar ko'chadi (`K/N` ta, hammasi emas).

```text
        0/360°
    node C   node A
       ╲     ╱
        ╲   ╱   kalit → soat yo'nalishida
   ─────( ring )─────   birinchi node'ga
        ╱   ╲
       ╱     ╲
    node B  (virtual nodes bilan tekislanadi)
```

**💡 Tushuncha:** Virtual nodes (vnodes) — har fizik node halqada ko'p nuqtada turadi, shunda taqsimot tekisroq va node o'chganda yuk qolganlarga teng tarqaladi. Cassandra, DynamoDB, memcached client'lari shundan foydalanadi.

| Usul | Diapazon so'rov | Yuk taqsimoti | Resharding narxi |
|------|-----------------|---------------|------------------|
| Range | Oson | Hot spot xavfi | O'rta |
| Hash (%N) | Qiyin | Teng | Yuqori (hammasi ko'chadi) |
| Consistent hash | Qiyin | Teng (vnodes bilan) | Past (K/N ko'chadi) |

---

## Distributed transactions: 2PC va Saga

**💡 Tushuncha:** Bir tranzaksiya bir necha xizmat/DB'ni o'z ichiga olsa (masalan: pul yechish + inventar kamaytirish), atomiklik (hammasi yoki hech biri) muammoga aylanadi.

### Two-Phase Commit (2PC)

**💡 Tushuncha:** Koordinator ikki fazada boshqaradi:

```text
FAZA 1 (prepare):  Coordinator → hammaga "tayyormisan?"
                   har biri: "ha, tayyor" (lock oladi) yoki "yo'q"
FAZA 2 (commit):   hamma "ha" desa → "commit!"; birortasi "yo'q" desa → "abort!"
```

```text
   Coordinator
    │  prepare
    ├──────────► Service A → "ready" ✓
    ├──────────► Service B → "ready" ✓
    │  commit
    ├──────────► A → commit
    └──────────► B → commit
```

**⚠️ Ehtiyot bo'l:** 2PC muammosi — **bloklovchi**. Faza 1'da resurslar lock qilinadi. Agar **koordinator** faza 2'dan oldin o'lsa, participant'lar lock bilan "osilib" qoladi (kimdir commit yoki abort deyishini kutib). Bu availability'ni buzadi va sekin — shuning uchun mikroservislarda kamdan-kam ishlatiladi.

### Saga pattern

**💡 Tushuncha:** Saga — uzun tranzaksiyani kichik **local tranzaksiyalar** ketma-ketligiga bo'ladi, har biri o'z servisida. Birortasi muvaffaqiyatsiz bo'lsa — oldingilarni **compensating** (qaytaruvchi) tranzaksiyalar bilan bekor qiladi. Lock yo'q, atomiklik o'rniga **eventual consistency**.

```text
Buyurtma sagasi:
  1. Order yaratildi ──► 2. To'lov olindi ──► 3. Inventar band ──► 4. Yetkazish
                                                    │ FAIL!
                          ◄── compensate: pul qaytar ◄── order bekor
```

Ikki uslub:
- **Choreography**: har servis event chiqaradi, keyingisi eshitadi (markazsiz, bo'shroq bog'liq).
- **Orchestration**: markaziy orkestrator qadamlarni boshqaradi (aniqroq, lekin markaz bor).

| | 2PC | Saga |
|-|-----|------|
| Atomiklik | Kuchli (ACID) | Eventual (compensation) |
| Lock | Bor (bloklovchi) | Yo'q |
| Availability | Past | Yuqori |
| Murakkablik | Koordinator | Compensation logikasi |
| Ishlatish | Bir necha DB, kam servis | Mikroservislar |

---

## Idempotency va exactly-once

**💡 Tushuncha:** Distributed tizimda tarmoq noaniqligi tufayli xabar yetdimi bilinmaydi → qayta yuborish kerak → xabar **ikki marta** yetishi mumkin. "Exactly-once delivery" (aynan bir marta yetkazish) — deyarli **imkonsiz** tarmoq darajasida.

```text
DELIVERY kafolatlari:
  at-most-once   → ko'pi bilan 1 marta (yo'qolishi mumkin, dublikat yo'q)
  at-least-once  → kamida 1 marta (dublikat mumkin) ← eng keng tarqalgan
  exactly-once   → aynan 1 marta (idealda, amalda qiyin)
```

**💡 Tushuncha:** Amaliy yechim — **exactly-once processing** (yetkazish emas): at-least-once delivery + **idempotent consumer**. Ya'ni xabar ikki marta yetsa ham, ishlov idempotent bo'lgani uchun natija bir marta bo'ladi.

```text
Idempotent consumer:
  har xabarda unique ID
  consumer: "bu ID'ni ko'rdimmi?"
    ha → tashla (skip)
    yo'q → ishlov ber, ID'ni saqla (dedup table)
```

**💡 Tushuncha:** Kafka "exactly-once semantics" ni idempotent producer + transactional write bilan beradi — lekin bu Kafka ichida. Tashqi ta'sir (email yuborish, pul yechish) uchun har doim idempotency key ishlat.

---

## Distributed clocks

**💡 Tushuncha:** Distributed tizimda "hozir soat necha?" — muammoli. Har mashinaning soati biroz farq qiladi (clock skew), NTP bilan sinxronlansa ham. Shuning uchun "kim oldin bo'ldi?" ni fizik vaqt bilan aniqlash ishonchsiz. Yechim — **logical clocks**.

### Lamport clock

**💡 Tushuncha:** Har node bitta hisoblagich (counter) saqlaydi:
```text
1. Har lokal hodisada: counter++
2. Xabar yuborishda: counter'ni xabar bilan yubor
3. Xabar olishda: counter = max(local, received) + 1
```
Bu **sababiy tartib** beradi: agar A → B (A B'ga sabab) bo'lsa, `L(A) < L(B)`. Lekin teskarisi shart emas: `L(A) < L(B)` bo'lsa ham, ular bog'liqmas (concurrent) bo'lishi mumkin.

### Vector clock

**💡 Tushuncha:** Har node **barcha** node'lar uchun hisoblagich saqlaydi (vektor). Bu ikki hodisa **sababiy bog'liqmi** yoki **concurrent** ekanligini **aniq** ayta oladi (Lamport ayta olmaydi).

```text
3 node: vektor [A, B, C]
  A'da hodisa → [1,0,0]
  A → B xabar → B: max har element + o'ziniki → [1,1,0]
  
Taqqoslash:
  [2,1,0] va [1,2,0] → hech biri ≤ ikkinchisi → CONCURRENT
  [1,0,0] va [2,1,0] → birinchi ≤ ikkinchi   → 1-si 2-dan OLDIN
```

**💡 Tushuncha:** Vector clocks konflikt aniqlashda (Dynamo, Riak) ishlatiladi. Kamchilik: node soni bilan vektor kattalashadi. Amaliy tizimlar hybrid logical clocks (HLC — fizik + logik) ham ishlatadi (masalan CockroachDB).

---

## Gossip protocol va split-brain

### Gossip protocol

**💡 Tushuncha:** Gossip (epidemic) protocol — har node vaqti-vaqti bilan **tasodifiy bir necha node** bilan holat almashadi, xuddi mish-mish tarqagandek. Ma'lumot eksponensial tez butun klasterga tarqaladi, markaziy koordinatorsiz.

```text
Round 1: A → B'ga aytadi
Round 2: A → C, B → D
Round 3: har biri yana ikkitasiga...
  → log(N) roundda hamma biladi
```

**💡 Tushuncha:** Ishlatilishi: membership (kim tirik/o'lgan — Cassandra, Consul), anti-entropy (replica farqlarini tuzatish), failure detection. Afzalligi: markazsiz, chidamli, miqyoslanadi. Kamchiligi: tez emas (eventual), bir oz ortiqcha trafik.

### Split-brain

**⚠️ Ehtiyot bo'l:** Split-brain — tarmoq bo'linganda, klasterning **ikki qismi ham** o'zini "asosiy" (leader) deb hisoblab, alohida ishlashni boshlaydi. Natija: ikkita leader, ziddiyatli yozishlar, ma'lumot buzilishi.

```text
   ╳ tarmoq bo'lindi ╳
 [A][B]  ╳╳╳  [C][D][E]
  ↓            ↓
 "men leader"  "men leader"   → ikkovi yozadi → KONFLIKT
```

**💡 Tushuncha:** Oldini olish — **quorum (majority)** talab qilish. Faqat node'larning yarmidan ko'pi (N/2+1) bilan bog'lana olgan tomon ishlaydi; ozchilik tomon o'zini "read-only" yoki o'chiradi (fencing). Shuning uchun toq sonli node kerak — bir tomon aniq majority oladi. Fencing token (monoton o'suvchi raqam) eski leader'ning yozishini rad etadi.

---

## ❓ Savol-javoblar

### ❓ Nega distributed tizim monolithdan tubdan qiyin?

**✅ Javob:** Uch fundamental sabab: (1) tarmoq ishonchsiz — so'rov yetdimi/javob yo'qoldimi bilib bo'lmaydi; (2) latency — tarmoq chaqiruvi lokaldan million marta sekin; (3) partial failure — bir node o'ladi, boshqalari ishlaydi (monolithda hamma ishlaydi yoki hamma o'ladi). Eng zarali partial failure: node javob bermasa, o'ldimi yoki sekinmi — hech qachon aniq bilmaysan, faqat timeout bilan taxmin qilasan.

### ❓ CAP theorem'ni tushuntiring — "ikkitasini tanla" nima ma'noda?

**✅ Javob:** Consistency, Availability, Partition tolerance — uchtadan ikkitasini kafolatlaysan. Lekin partition (P) distributed tizimda muqarrar, uni tanlab bo'lmaydi. Shuning uchun aslida tanlov C vs A: partition sodir bo'lganda consistency saqlash uchun so'rovni rad qilasan (CP) yoki javob berish uchun stale ma'lumotga rozi bo'lasan (AP). Partition yo'q paytda tizim ham C, ham A bo'la oladi.

### ❓ PACELC CAP'dan qanday farq qiladi?

**✅ Javob:** CAP faqat partition holatini ko'radi. PACELC qo'shadi: partition **bo'lmaganda ham** (Else) tanlov bor — latency vs consistency. Ya'ni normal ishlashda ham strong consistency (koordinatsiya kerak, sekin) yoki past latency (ehtimol stale) o'rtasida tanlaysan. Masalan Cassandra = PA/EL (doim tezlikni afzal), HBase = PC/EC (doim consistency).

### ❓ Eventual va strong consistency farqi, qachon qaysi biri?

**✅ Javob:** Strong (linearizable) — har o'qish eng oxirgi yozishni ko'radi, real vaqt tartibida; koordinatsiya kerak, sekin, lekin sodda. Ishlatish: bank balansi, inventar. Eventual — replica'lar vaqtincha farq qiladi, keyin yaqinlashadi; tez, available, lekin dasturchi nomuvofiqlikni boshqaradi. Ishlatish: ijtimoiy like, DNS, feed. Orada causal (sababiy tartib) va read-your-writes (o'z yozganingni ko'rish) bor.

### ❓ Consensus nima uchun kerak va nega qiyin?

**✅ Javob:** Consensus — node'lar bir qiymat/qaror bo'yicha kelishishi (ba'zilari o'lsa ham). Kerak: leader tanlash, tartibli replikatsiya, atomik commit. Qiyin: node'lar o'ladi, xabar yo'qoladi/kechikadi, va FLP theorem bo'yicha to'liq asinxron tizimda bitta nosozlikda ham kafolatlab bo'lmaydi (amalda timeout bilan yon aylaniladi). Paxos birinchi yechim, lekin murakkab; Raft tushunarli muqobil.

### ❓ Raft'da leader election qanday ishlaydi?

**✅ Javob:** Vaqt term'larga bo'linadi. Follower leader'dan heartbeat kutadi; tasodifiy timeout (150-300ms) tugasa Candidate bo'ladi, term'ni oshiradi, o'ziga ovoz beradi va boshqalardan RequestVote so'raydi. Majority (N/2+1) ovoz olsa Leader bo'ladi va heartbeat yuboradi. Tasodifiy timeout ikki node bir vaqtda candidate bo'lishi (split vote) ehtimolini kamaytiradi. Odatda toq sonli node ishlatiladi.

### ❓ Raft log replication ma'lumotni qanday yo'qotmaydi?

**✅ Javob:** Leader yozishni log'ga qo'yadi (uncommitted), follower'larga AppendEntries yuboradi. Majority tasdiqlagandan **keyingina** "committed" deb belgilaydi va client'ga OK qaytaradi. Majority quorum tufayli, leader o'lsa ham yangi leader eng yangi commit qilingan log'ga ega bo'ladi (majority'da mavjud). Shu tarzda commit qilingan ma'lumot yo'qolmaydi.

### ❓ Quorum (N/W/R) qanday consistency beradi?

**✅ Javob:** N=replika, W=yozish tasdig'i node soni, R=o'qish node soni. Agar `W+R > N` bo'lsa, yozish va o'qish to'plamlari kamida bitta node'da kesishadi → o'qish eng yangi yozuvni topadi (strong). Masalan N=3, W=2, R=2 → 4>3 ✓. W va R'ni oshirish consistency'ni oshiradi, lekin latency/availability'ni pasaytiradi — bu CAP'ning amaliy sozlagichi.

### ❓ Single-leader, multi-leader, leaderless replikatsiya farqi?

**✅ Javob:** Single-leader: bitta node yozishni oladi, follower'larga ko'chiradi — sodda, konfliktsiz, lekin leader SPOF va replication lag muammosi. Multi-leader: bir necha node yozadi (masalan har region) — geo va offline uchun yaxshi, lekin konflikt bor. Leaderless (Dynamo): client bir necha replica'ga to'g'ridan yozadi, o'qishda quorum yig'adi — yuqori availability (AP), konsistensiya N/W/R bilan sozlanadi (Cassandra, DynamoDB).

### ❓ LWW, vector clocks va CRDT konfliktni qanday hal qiladi?

**✅ Javob:** LWW (Last-Write-Wins): timestamp bo'yicha eng kech g'olib — oddiy, lekin ma'lumot yo'qotadi va clock skew'ga sezgir. Vector clocks: ikki versiya sababiy bog'liqmi yoki haqiqiy konfliktmi aniqlaydi; konfliktda ikkalasini saqlab ilovaga merge topshiradi. CRDT: matematik struktura (kommutativ, idempotent operatsiyalar) tufayli koordinatsiyasiz avtomatik converge bo'ladi — offline-first va real-time collab uchun ideal, lekin har ma'lumot turi CRDT bo'la olmaydi.

### ❓ Consistent hashing nima muammoni hal qiladi?

**✅ Javob:** Oddiy `hash % N` sharding'da node soni o'zgarsa (qo'shilsa/o'chsa), deyarli barcha kalitlar ko'chadi (resharding falokati). Consistent hashing halqa (ring) ishlatadi: node va kalit halqaga joylashadi, kalit soat yo'nalishida eng yaqin node'ga tegishli. Node o'zgarsa faqat qo'shni kalitlar (K/N ta) ko'chadi. Virtual nodes bilan taqsimot tekislanadi. Cassandra, DynamoDB, memcached ishlatadi.

### ❓ 2PC va Saga farqi, mikroservislarda qaysi biri?

**✅ Javob:** 2PC: koordinator prepare (lock oladi) → commit/abort. Kuchli atomiklik, lekin bloklovchi (lock) va koordinator o'lsa participant'lar osilib qoladi — availability past, mikroservislarda kam ishlatiladi. Saga: uzun tranzaksiyani local tranzaksiyalar ketma-ketligiga bo'ladi, xato bo'lsa compensating tranzaksiyalar bilan bekor qiladi — lock yo'q, eventual consistency, yuqori availability. Mikroservislarda odatda Saga (choreography yoki orchestration).

### ❓ Exactly-once delivery mumkinmi?

**✅ Javob:** Tarmoq darajasida deyarli imkonsiz — xabar yetdimi bilinmagani uchun retry kerak, retry esa dublikat keltiradi. Amaliy yechim: at-least-once delivery + idempotent consumer = exactly-once **processing**. Ya'ni xabar ikki marta yetsa ham, unique ID bo'yicha dedup qilib, natija bir marta bo'ladi. Tashqi ta'sirlar (pul, email) uchun idempotency key ishlatiladi. Kafka "exactly-once" ni idempotent producer + transactional write bilan beradi (Kafka ichida).

### ❓ Lamport va vector clock farqi?

**✅ Javob:** Lamport clock — bitta hisoblagich, sababiy tartib beradi: A→B bo'lsa L(A)<L(B). Lekin teskarisi shart emas — L(A)<L(B) bo'lsa ular concurrent bo'lishi mumkin (Lamport ayta olmaydi). Vector clock — har node uchun alohida hisoblagich (vektor), ikki hodisa sababiy bog'liqmi yoki concurrent'mi **aniq** ayta oladi. Konflikt aniqlashda (Dynamo, Riak) shu kerak. Kamchiligi: node soni bilan vektor kattalashadi.

### ❓ Split-brain nima va qanday oldini olinadi?

**✅ Javob:** Tarmoq bo'linganda klasterning ikki qismi ham o'zini leader deb, alohida yozishni boshlaydi → ikkita leader, ziddiyatli yozish, ma'lumot buzilishi. Oldini olish: quorum (majority) talab qilish — faqat yarmidan ko'pi (N/2+1) bilan bog'lana olgan tomon ishlaydi, ozchilik o'zini read-only qiladi yoki o'chadi (fencing). Toq sonli node kerak, bir tomon aniq majority olishi uchun. Fencing token eski leader yozishini rad etadi.

---

## Masalalar

> Yechimlar: [`solutions/system-design/04-distributed-systems.md`](../solutions/system-design/04-distributed-systems.md)

1. **CP yoki AP tanlang.** Quyidagi tizimlar uchun CP yoki AP tanlang va asoslang: (a) aviabilet bron tizimi, (b) Instagram feed, (c) bank pul o'tkazmasi, (d) DNS, (e) e-commerce savati. Har biri uchun partition paytida qanday xatti-harakat bo'lishini yozing.

2. **Quorum sozlash.** N=5 replika bilan Cassandra klasterida: (a) strong consistency uchun W va R qanday? (b) write-heavy yuk uchun-chi? (c) read-heavy uchun-chi? Har birida availability va latency ta'sirini tushuntiring.

3. **Raft simulyatsiyasi.** 5 node'li Raft klasterida leader o'ldi. Qadamma-qadam nima bo'lishini yozing: qanday yangi leader tanlanadi, term qanday o'zgaradi, va nima uchun eski committed ma'lumot yo'qolmaydi. 3 node bir vaqtda o'lsa nima bo'ladi?

4. **Consistent hashing tahlili.** 4 ta cache node'dan biri o'ldi. (a) Oddiy `hash%N` da nechta kalit ko'chadi? (b) Consistent hashing'da-chi? (c) Virtual nodes nima muammoni hal qiladi? Halqa diagrammasini chizing va taxminiy raqamlar bering.

5. **Saga dizayni.** Sayohat bron tizimi uchun saga dizayn qiling: reys + mehmonxona + avtomobil bron. Har qadam va uning compensating tranzaksiyasini yozing. Mehmonxona bron muvaffaqiyatsiz bo'lsa nima bo'ladi? Choreography yoki orchestration tanlang va asoslang.

6. **Konflikt hal qilish.** Ikki region'da bir foydalanuvchi savatini bir vaqtda o'zgartirdi (biri item qo'shdi, biri o'chirdi). (a) LWW qanday hal qiladi va nima yo'qoladi? (b) Vector clocks-chi? (c) CRDT (OR-Set) qanday ishlatiladi? Har birining natijasini ko'rsating.

7. **Exactly-once.** To'lov xabarlarini ishlaydigan consumer dizayn qiling. At-least-once queue'dan pul ikki marta yechilmasligini kafolatlang: idempotency key qayerda saqlanadi, dedup qanday ishlaydi, va race condition (bir xabar bir vaqtda ikki consumer'da) qanday hal qilinadi?

8. **Vector clock hisobi.** 3 node (A, B, C) uchun quyidagi hodisalar ketma-ketligida vector clock qiymatlarini hisoblang va qaysi juftliklar concurrent ekanligini aniqlang: A yozadi, A→B yuboradi, B yozadi, C mustaqil yozadi, B→C yuboradi.

9. **Split-brain oldini olish.** 6 node'li klaster (3+3) ikki data-markazga bo'lingan. Tarmoq DC'lar orasida uzildi. (a) Nega 6 (juft) son muammo? (b) Split-brain'ni qanday oldini olasiz? (c) Bitta DC to'liq o'lsa, qolgani ishlashi uchun nima kerak (witness/tiebreaker)?

10. **Consistency model tanlash.** Kollaborativ hujjat tahrirlash ilovasi (Google Docs uslubi) uchun qaysi consistency model va replikatsiya strategiyasini tanlaysiz? Nega strong consistency mos kelmaydi? CRDT qanday yordam beradi? Offline tahrir qanday merge qilinadi?

---

← [System Design bo'limiga qaytish](./README.md)
