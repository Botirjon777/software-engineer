# Distributed Systems — Masalalar Yechimi (CHUQUR instruksiya)

Bu fayl [`system-design/04-distributed-systems.md`](../../system-design/04-distributed-systems.md) dagi "Masalalar" bo'limining namunaviy yechimlarini o'z ichiga oladi. System design'da yagona to'g'ri javob yo'q — quyidagilar **namunaviy yondashuv**; muhimi, qaroringizni trade-off bilan asoslay olishingiz.

## Mundarija

- [1. CP yoki AP tanlang](#1-cp-yoki-ap-tanlang)
- [2. Quorum sozlash](#2-quorum-sozlash)
- [3. Raft simulyatsiyasi](#3-raft-simulyatsiyasi)
- [4. Consistent hashing tahlili](#4-consistent-hashing-tahlili)
- [5. Saga dizayni](#5-saga-dizayni)
- [6. Konflikt hal qilish](#6-konflikt-hal-qilish)
- [7. Exactly-once](#7-exactly-once)
- [8. Vector clock hisobi](#8-vector-clock-hisobi)
- [9. Split-brain oldini olish](#9-split-brain-oldini-olish)
- [10. Consistency model tanlash](#10-consistency-model-tanlash)

---

## 1. CP yoki AP tanlang

| Tizim | Tanlov | Partition paytida xatti-harakat |
|-------|--------|----------------------------------|
| (a) Aviabilet bron | **CP** | Ikki marta sotmaslik uchun consistency ustun. Partition'da bron rad etiladi ("keyinroq urinib ko'ring") — noto'g'ri joy sotgandan yaxshi. |
| (b) Instagram feed | **AP** | Doim javob berish muhim. Partition'da biroz eski feed ko'rsatiladi (stale) — bo'sh ekrandan yaxshi. |
| (c) Bank o'tkazma | **CP** | Balans to'g'riligi mutlaq kritik. Partition'da tranzaksiya bloklanadi/rad etiladi — ikki marta yechishdan yaxshi. |
| (d) DNS | **AP** | Availability hamma narsadan muhim. Eventual consistency — o'zgarish tarqalishi vaqt oladi, lekin doim javob bor. |
| (e) E-commerce savati | **AP** | Savat doim ishlashi kerak (Dynamo shu uchun yaratilgan). Konflikt bo'lsa savatlarni merge qil (item yo'qotmaslik uchun). |

**💡 Tushuncha:** Umumiy qoida — **pul/inventar/to'g'rilik** → CP; **foydalanuvchi tajribasi/doim javob** → AP. Lekin bir tizim ichida turli qismlar turlicha bo'lishi mumkin (savat AP, checkout CP).

---

## 2. Quorum sozlash

```text
N = 5 replika

(a) Strong consistency:  W + R > N → W=3, R=3 (3+3=6 > 5) ✓
    yoki W=4,R=2 / W=2,R=4 — kesishish kafolatlangan

(b) Write-heavy:  yozishni tez qil → W kichik
    W=1, R=5  (yozish 1 node, o'qish hammadan)
    W+R=6>5 → hali strong, lekin o'qish sekin va past availability o'qishda

(c) Read-heavy:  o'qishni tez qil → R kichik
    W=5, R=1  (o'qish 1 node, yozish hammaga)
    W+R=6>5 → strong, lekin yozish sekin va past availability yozishda
```

| Sozlash | Consistency | Yozish | O'qish | Availability |
|---------|-------------|--------|--------|--------------|
| W=3,R=3 | Strong | O'rta | O'rta | Balanslangan |
| W=1,R=5 | Strong | Tez | Sekin | Yozishda yuqori, o'qishda past |
| W=5,R=1 | Strong | Sekin | Tez | O'qishda yuqori, yozishda past |
| W=1,R=1 | Eventual | Tez | Tez | Eng yuqori (stale mumkin) |

**💡 Tushuncha:** W yoki R'ni oshirsang consistency oshadi, lekin o'sha amalning latency va availability'i pasayadi. `W+R>N` strong consistency beradi, lekin `W+R≤N` (masalan W=1,R=1) eventual va eng tez.

---

## 3. Raft simulyatsiyasi

```text
5 node (term=4, N1 leader edi). N1 o'ldi.

1. Follower'lar N1 heartbeat'ini kutadi → timeout (tasodifiy 150-300ms).
2. Birinchi timeout bo'lgan (masalan N2) Candidate bo'ladi: term=5, o'ziga ovoz.
3. N2 → RequestVote(term=5) yuboradi N3,N4,N5'ga.
4. Ular term=5 > o'z termi va N2 log'i yangi bo'lsa → ovoz beradi.
5. N2 majority (3 ovoz: o'zi + 2) oldi → LEADER (term=5).
6. N2 heartbeat yuboradi → N3,N4,N5 follower qoladi.

Nima uchun ma'lumot yo'qolmaydi:
  Eski committed yozuv majority'da (kamida 3 node) bor edi.
  Raft "election restriction": faqat eng yangi log'ga ega node leader bo'la oladi.
  Demak yangi leader (N2) o'sha committed yozuvga ega — yo'qolmaydi.
```

3 node bir vaqtda o'lsa (5 dan):
```text
Qolgani 2 node → majority (3) yig'a olmaydi → leader tanlanmaydi.
Klaster YOZISHNI TO'XTATADI (unavailable), lekin ma'lumot buzilmaydi (CP).
```

**💡 Tushuncha:** 5 node → 2 nosozlikka chidaydi (3 majority saqlanadi). 3 nosozlikda majority yo'q — Raft ataylab to'xtaydi (split-brain o'rniga unavailability tanlaydi).

---

## 4. Consistent hashing tahlili

```text
4 cache node, 1 tasi o'ldi (K = umumiy kalitlar soni)

(a) hash % N:  N 4→3 o'zgardi → modul o'zgardi → deyarli HAMMA kalit ko'chadi
    Taxminan: ~75% (3/4) kalit boshqa node'ga tushadi (falokat!)

(b) Consistent hashing:  faqat o'lgan node'ning kalitlari ko'chadi
    O'lgan node ~1/4 kalitni saqlar edi → faqat ~25% (K/N) ko'chadi
    Va ular halqada keyingi node'ga o'tadi.

(c) Virtual nodes: bitta fizik node halqada ko'p nuqtada
    → o'lgan node yuki BITTA qo'shniga emas, HAMMA qolganlarga
      teng tarqaladi (hot spot yo'q) + taqsimot dastlab tekisroq.
```

```text
Halqa (V-nodes'siz, soddalashtirilgan):
        0°
    D       A
     ╲     ╱
      ( ring )   kalit → soat yo'nalishida birinchi node
     ╱     ╲
    C       B

  B o'ldi → B'ning kalitlari → C'ga (keyingi). A,D o'zgarmaydi.
  V-nodes bilan: B ning yuki A,C,D ga bo'linadi.
```

**💡 Tushuncha:** Consistent hashing resharding narxini `~75%` dan `~25%` ga (`K/N`) tushiradi. Virtual nodes hot spot'ni yo'qotadi va teng taqsimlaydi.

---

## 5. Saga dizayni

```text
Sayohat bron sagasi (orchestration):

Qadam                     Compensating (bekor qiluvchi)
──────────────────────────────────────────────────────
1. Reys bron              → Reys bekor (refund)
2. Mehmonxona bron        → Mehmonxona bekor
3. Avtomobil bron         → Avtomobil bekor

Mehmonxona (qadam 2) FAIL bo'lsa:
  → 1-qadamning compensating'i: Reys bekor + refund
  → saga to'xtaydi, foydalanuvchiga "bron amalga oshmadi" (hech narsa qolmaydi)
```

```text
Orkestrator:
  Reys OK ──► Mehmonxona FAIL
                  │
                  ▼
             compensate: Reys bekor
                  │
                  ▼
             saga aborted (toza holat)
```

Uslub tanlovi: **Orchestration** — chunki bu ko'p qadamli, aniq tartibli biznes-jarayon; markaziy orkestrator holat va compensation'ni oson boshqaradi va kuzatiladi (observability). Choreography (event-driven) ko'p servisda tarqalib, kim nima qilayotganini kuzatish qiyinlashadi.

**💡 Tushuncha:** Saga eventual consistency beradi — bir lahzada "reys bron, mehmonxona yo'q" oraliq holat bo'lishi mumkin, keyin compensation tozalaydi. Har qadam va compensation **idempotent** bo'lishi shart (retry xavfsiz).

---

## 6. Konflikt hal qilish

```text
Boshlang'ich savat: {A, B}
  Region 1: C qo'shdi → {A, B, C}
  Region 2: B o'chirdi → {A}
```

```text
(a) LWW (timestamp bo'yicha):
    kechroq yozilgan g'olib → biri butunlay yutadi
    Agar Region1 kechroq → {A,B,C} (B o'chirilishi YO'QOLDI)
    Agar Region2 kechroq → {A} (C qo'shilishi YO'QOLDI)
    → HAR HOLDA ma'lumot yo'qoladi. Yomon.

(b) Vector clocks:
    ikki versiya concurrent deb aniqlanadi → ikkalasi saqlanadi (siblings)
    ilova/foydalanuvchi merge qiladi. Xavfsiz, lekin merge logikasi kerak.

(c) CRDT (OR-Set):
    har element unique tag bilan qo'shiladi/o'chiriladi.
    C qo'shildi (tag), B o'chirildi (o'z tag bilan)
    → avtomatik, ziddiyatsiz merge → {A, C}
    (qo'shish ham, o'chirish ham saqlanadi, konfliktsiz)
```

| Usul | Natija | Ma'lumot yo'qoladi? |
|------|--------|---------------------|
| LWW | {A,B,C} yoki {A} | Ha (biri) |
| Vector clocks | Siblings → merge kerak | Yo'q (lekin qo'lda) |
| CRDT (OR-Set) | {A, C} | Yo'q (avtomatik) |

**💡 Tushuncha:** Savat kabi "item yo'qotmaslik" muhim joyda LWW yomon (Amazon Dynamo aynan shu sabab vector clocks/merge ishlatgan). CRDT eng qulay — avtomatik va to'g'ri merge.

---

## 7. Exactly-once

```text
At-least-once queue'dan pulni ikki marta yechmaslik:

1. Har xabarda unique idempotency key (masalan payment_id).
2. Consumer DB'da "processed_payments" jadval:
     INSERT payment_id (UNIQUE constraint)
       muvaffaqiyat → yangi, ishlov ber
       duplicate key xato → allaqachon ishlov berilgan, SKIP
3. Ishlov + INSERT bitta DB tranzaksiyada (atomik):
     BEGIN
       INSERT processed(payment_id)   -- dedup
       UPDATE balance ...             -- asosiy ish
     COMMIT
   → ikkalasi birga bo'ladi yoki hech biri.
```

Race condition (bir xabar bir vaqtda ikki consumer'da):
- **UNIQUE constraint** DB darajasida himoya qiladi — ikkinchi INSERT fail bo'ladi, o'sha consumer skip qiladi.
- Yoki: xabarga **lock/lease** (queue visibility timeout) — bittasi ishlaganda ikkinchisi ko'rmaydi.
- Tashqi ta'sir (masalan real pul yechish) bo'lsa: idempotency key'ni **tashqi provider**ga ham yubor (u ham dedup qilsin).

```text
Consumer 1        Consumer 2
   │ xabar (dup)     │ xabar (dup)
   ▼                 ▼
INSERT id ✓       INSERT id ✗ (unique violation)
UPDATE balance    SKIP (allaqachon)
COMMIT
→ pul BIR marta yechildi
```

**💡 Tushuncha:** "Exactly-once delivery" imkonsiz, lekin at-least-once + idempotent consumer (DB unique constraint bilan) = exactly-once **processing**.

---

## 8. Vector clock hisobi

```text
3 node [A, B, C], boshlang'ich [0,0,0]

1. A yozadi:       A = [1,0,0]
2. A → B yuboradi: B = max([0,0,0],[1,0,0]) + B++ = [1,1,0]
3. B yozadi:       B = [1,2,0]
4. C mustaqil yozadi: C = [0,0,1]
5. B → C yuboradi: C = max([0,0,1],[1,2,0]) + C++ = [1,2,2]

Yakuniy:  A=[1,0,0]   B=[1,2,0]   C=[1,2,2]
```

Concurrent juftliklarni aniqlash (hech biri ikkinchisining ≤ emas):
```text
A=[1,0,0] vs C=[0,0,1]:
  A: 1>0 (1-o'rin), lekin C: 0<... 3-o'rinda C kattaroq (0<1)
  → hech biri ≤ ikkinchisi → CONCURRENT ✓

C=[0,0,1] (yozilgan payt, 4-qadam) B=[1,2,0] bilan:
  C'ning 3-elementi (1) > B'ning (0), B'ning 1,2-elementi > C
  → CONCURRENT (B → C birlashguncha edi)

A=[1,0,0] vs B=[1,2,0]:  A ≤ B (har element) → A B'dan OLDIN (sababiy)
B=[1,2,0] vs C=[1,2,2]:  B ≤ C → B C'dan OLDIN (5-qadam merge)
```

**💡 Tushuncha:** Vector clock element-ma-element taqqoslanadi. Biri ikkinchisidan hamma elementda ≤ bo'lsa — sababiy oldinda. Aks holda (aralash) — concurrent (haqiqiy konflikt).

---

## 9. Split-brain oldini olish

```text
6 node = DC1(3) + DC2(3). DC'lar orasida tarmoq uzildi.
```

(a) **Nega 6 (juft) muammo:** Bo'linganda har tomon aynan 3 node — hech biri majority (N/2+1 = 4) emas. Ikki tomon ham "men majority emasman" deb to'xtaydi → **butun klaster o'ladi** (yoki noto'g'ri sozlansa, ikkovi ham "men leader" deb split-brain).

(b) **Oldini olish:** Quorum (majority = 4) talab qilish — faqat 4+ node ko'rgan tomon ishlaydi. Juft 3+3 da hech tomonda 4 yo'q, shuning uchun **toq son kerak** (5 yoki 7).

(c) **Bitta DC o'lsa qolgan ishlashi uchun:**
```text
Yechim: 5 node → DC1(2) + DC2(2) + WITNESS(1) uchinchi joyda
  DC1 o'lsa: DC2(2) + witness(1) = 3 = majority → ishlaydi ✓
  DC2 o'lsa: DC1(2) + witness(1) = 3 = majority → ishlaydi ✓
  Tarmoq DC1|DC2 uzilsa: witness qaysi tomonda bo'lsa, o'sha majority oladi.
```

**💡 Tushuncha:** Toq son + uchinchi zonada witness/tiebreaker — split-brain'ni oldini oladi va bitta zona to'liq yo'qolsa ham qolgani ishlaydi. Fencing token esa eski leader'ning kech kelgan yozishini rad etadi.

---

## 10. Consistency model tanlash

```text
Kollaborativ hujjat (Google Docs uslubi):
  - Ko'p foydalanuvchi bir vaqtda tahrir qiladi
  - Offline tahrir → keyin sinxron
  - Real-time, past latency talab
```

**Nega strong consistency mos kelmaydi:**
- Har tahrirda global koordinatsiya (lock/consensus) kerak bo'lardi → juda sekin, real-time buziladi.
- Offline tahrirda koordinatsiya imkonsiz (tarmoq yo'q).
- Foydalanuvchi lock kutib o'tira olmaydi (yozishda kechikish qabul qilinmaydi).

**Tanlov:** Eventual consistency + **CRDT** (yoki OT — Operational Transformation).

```text
CRDT qanday yordam beradi:
  - Har belgi/operatsiya unique ID va tartib bilan
  - Ikki foydalanuvchi bir vaqtda yozsa → operatsiyalar kommutativ
    → har replica bir xil natijaga converge bo'ladi (koordinatsiyasiz)
  - Konflikt "avtomatik" hal bo'ladi (matematik struktura tufayli)

Offline merge:
  - Offline'da lokal operatsiyalar to'planadi (o'z CRDT holatida)
  - Ulanish tiklanganda operatsiyalar almashadi
  - CRDT xususiyati: tartibsiz kelsa ham, hammasi bir xil holatga keladi
  - Ma'lumot yo'qolmaydi (LWW'dan farqli), ikkovining tahriri saqlanadi
```

**💡 Tushuncha:** Google Docs OT ishlatadi (tarixiy), zamonaviy collab (Automerge, Yjs) CRDT ishlatadi. Ikkalasi ham eventual consistency + avtomatik konflikt hal qilish g'oyasiga asoslanadi — real-time va offline-first uchun strong consistency'dan afzal.

---

← [System Design bo'limiga qaytish](../../system-design/README.md)
