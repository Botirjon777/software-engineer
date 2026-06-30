# Intervyu Jarayoni va Muzokara — Masalalar Yechimi (CHUQUR instruksiya)

Bu fayl [`behavioral/02-interview-process.md`](../../behavioral/02-interview-process.md) dagi "Masalalar" bo'limidagi mashqlarning namunaviy yechimlarini o'z ichiga oladi. Yechimlar — yagona to'g'ri javob emas, balki **namuna va yondashuv**; o'z tajribangiz va kontekstingizga moslang.

## Mundarija

- [1. Pipeline xaritasi](#1-pipeline-xaritasi)
- [2. Coding freymvork mashqi](#2-coding-freymvork-mashqi)
- [3. Clarifying savollar](#3-clarifying-savollar)
- [4. Think-aloud skripti](#4-think-aloud-skripti)
- [5. Maosh muzokarasi role-play](#5-maosh-muzokarasi-role-play)
- [6. Offer taqqoslash jadvali](#6-offer-taqqoslash-jadvali)
- [7. CV qayta yozish](#7-cv-qayta-yozish)
- [8. Recruiter screen savollari](#8-recruiter-screen-savollari)

---

## 1. Pipeline xaritasi

**💡 Tushuncha:** Misol uchun o'rta hajmli xalqaro remote startapni olamiz.

| Bosqich | Nimani tekshiradi | Tayyorgarlik |
|---------|-------------------|--------------|
| Recruiter screen (30 daq) | Umumiy moslik, maosh kutilmasi, timezone | Kompaniyani o'rgan, maosh diapazonini bil, "o'zim haqimda" 2 daqiqalik pitch tayyorla |
| Technical screen (1 soat) | "Haqiqiy dasturchimi" filtri, 1 ta easy/medium coding | LeetCode easy/medium, til bo'yicha asoslar |
| Coding round (1 soat) | DSA, muammo yechish jarayoni | 6 qadamli freymvork, think-aloud mashqi |
| System design (1 soat) | Miqyosda fikrlash, trade-off | TinyURL/rate limiter kabi klassik dizaynlar |
| Behavioral (45 daq) | Jamoaga moslik, kommunikatsiya | STAR metodi, 5-6 tayyor hikoya |
| Offer + muzokara | — | Bozor narxi, counter-offer skripti |

**⚠️ Ehtiyot bo'l:** Har kompaniyaning jarayoni har xil — buni rekruterdan oldindan so'rab oling: "Jarayon necha bosqichdan iborat va har biri qancha vaqt oladi?"

---

## 2. Coding freymvork mashqi

**Masala:** "Two Sum" — massivdan ikkita indeks toping, ularning yig'indisi `target`ga teng.

```
1. CLARIFY
   - Massiv sorted'mi? (Yo'q, deylik)
   - Bir nechta javob bo'lishi mumkinmi? (Yo'q, aniq bitta)
   - Bir element ikki marta ishlatiladimi? (Yo'q)
   - Manfiy sonlar bormi? (Ha, bo'lishi mumkin)

2. EXAMPLE
   nums = [2, 7, 11, 15], target = 9
   2 + 7 = 9  →  javob: [0, 1]

3. APPROACH
   - Brute force: barcha juftlarni tekshirish → O(n²) time, O(1) space.
   - Optimal: hash map'da har element uchun complement (target - num)
     bor-yo'qligini tekshirish → O(n) time, O(n) space.

4. CODE (think-aloud bilan)
```

```js
function twoSum(nums, target) {
  const seen = new Map(); // qiymat -> indeks
  for (let i = 0; i < nums.length; i++) {
    const complement = target - nums[i];
    if (seen.has(complement)) {
      return [seen.get(complement), i];
    }
    seen.set(nums[i], i);
  }
  return []; // topilmadi
}
```

```
5. TEST
   - [2,7,11,15], 9 → [0,1] ✓
   - [3,3], 6 → [0,1] ✓ (takror qiymat)
   - [1,2], 10 → [] (topilmadi)
   - [] → [] (bo'sh)

6. ANALYZE
   Time: O(n) — bitta o'tish.
   Space: O(n) — hash map.
```

---

## 3. Clarifying savollar

"Two Sum" masalasi uchun namunaviy savollar:

1. Massiv saralangan (sorted) bo'ladimi yoki tartibsizmi?
2. Aniq bitta yechim borligiga kafolat bormi, yoki nol/bir nechta bo'lishi mumkinmi?
3. Bir xil elementni ikki marta ishlatishga ruxsatmi (masalan indeks bir xil)?
4. Massivda takrorlanuvchi qiymatlar bo'lishi mumkinmi?
5. Sonlar manfiy yoki nol bo'lishi mumkinmi? Qiymatlar diapazoni qanaqa (overflow xavfi)?
6. Indekslarni qaytaraymi yoki qiymatlarni? Tartib muhimmi?
7. Massiv hajmi qanchalik katta bo'lishi mumkin (performance cheklovi)?

---

## 4. Think-aloud skripti

"Reverse a linked list" uchun namunaviy monolog:

> "Mayli, bizda singly linked list bor va uni teskari aylantirishimiz kerak. Avval bir misol ko'ray: 1 → 2 → 3 → null, natija 3 → 2 → 1 → null bo'lishi kerak.
>
> Ikki yondashuv ko'rayapman: iterativ va rekursiv. Iterativni afzal ko'raman, chunki O(1) space ishlatadi, rekursiyada esa stack O(n) bo'ladi.
>
> Iterativ uchun uchta pointer kerak: `prev`, `curr`, va `next`. `prev`ni null qilib boshlayman, `curr`ni head qilaman.
>
> Har qadamda: avval `next`ni saqlab qo'yaman (`curr.next`), keyin `curr.next`ni `prev`ga yo'naltiraman — mana shu yerda link teskari bo'ladi. Keyin `prev`ni `curr`ga, `curr`ni `next`ga suraman.
>
> Loop `curr` null bo'lganda tugaydi, va yangi head — `prev` bo'ladi.
>
> Edge case'larni o'ylab ko'ray: bo'sh ro'yxat (head null) — loop umuman ishlamaydi, prev null qaytadi, to'g'ri. Bitta element — bir martagina aylanadi, o'zi qaytadi, to'g'ri.
>
> Endi kodni yozaman va keyin qo'lda yugurtirib ko'raman."

```js
function reverseList(head) {
  let prev = null;
  let curr = head;
  while (curr !== null) {
    const next = curr.next; // saqlab qol
    curr.next = prev;       // link'ni teskari qil
    prev = curr;            // prev'ni surib qo'y
    curr = next;            // curr'ni surib qo'y
  }
  return prev; // yangi head
}
```

---

## 5. Maosh muzokarasi role-play

**Recruiter:** "Biz sizga $2500/oy taklif qilmoqchimiz. Nima deysiz?"

**Siz:** "Taklif uchun katta rahmat, jamoa va loyihangizdan juda hayajondaman. Rolni qabul qilishga jiddiy qaraganim uchun ochiq gaplashmoqchiman."

**Siz:** "Men bu pozitsiya uchun bozor narxini o'rgandim — Levels.fyi va shu darajadagi remote middle backend rollar uchun diapazon $3200–3800/oy atrofida. Mening N yillik tajribam, [aniq ko'nikma, masalan distributed systems] bo'yicha kuchli tomonlarimni hisobga olib, base'ni $3500 ga ko'tara olamizmi?"

**Recruiter:** "Bu bizning diapazondan biroz yuqori. $2800 qila olamiz."

**Siz:** "Buni qadrlayman. $3500 mening maqsadim edi, lekin moslashishga tayyorman. Agar base'ni $3200 ga yetkazsangiz va signing bonus yoki qo'shimcha o'rganish budjeti haqida gaplashsak, men bu taklifni darhol imzolashga tayyorman."

**💡 Tushuncha:** Diqqat qiling — (1) minnatdorchilik, (2) bozor ma'lumotiga asoslash, (3) konkret raqam, (4) kollaborativ ton ("imzolashga tayyorman"), (5) base bo'lmasa boshqa komponentlar.

---

## 6. Offer taqqoslash jadvali

**Offer A** (yirik kompaniya): base $4000, kichik bonus, struktura aniq, lekin legacy stack, mentorlik kam.
**Offer B** (startap): base $3200, yaxshi equity, kuchli mentor, zamonaviy stack, lekin risk yuqori.

| Omil | Offer A | Offer B |
|------|---------|---------|
| Total comp | Yuqori (kafolatli) | O'rta (equity riskli) |
| O'sish/mentorlik | Past | Yuqori |
| Jamoa/menejer | Noma'lum | Kuchli mentor |
| Texnologiya | Legacy | Zamonaviy |
| Work-life | Barqaror | O'zgaruvchan |
| Barqarorlik | Yuqori | Past (funding xavfi) |

**Namunaviy qaror:** "Agar karyeramning erta bosqichidaman va o'rganishga ustuvorlik bersam — Offer B'ni tanlardim, chunki kuchli mentor va zamonaviy stack uzoq muddatda ko'proq qiymat beradi, qisqa muddatli $800 farqdan muhimroq. Agar moliyaviy barqarorlik ustuvor bo'lsa (masalan oilaviy majburiyat) — Offer A xavfsizroq."

**⚠️ Ehtiyot bo'l:** "To'g'ri" javob yo'q — muhimi, qaroringizni **asoslay olishingiz**.

---

## 7. CV qayta yozish

**Asl (zaif):** "Worked on the backend and improved performance."

Qayta yozish variantlari:

1. "Reduced API p99 latency from 850ms to 180ms by introducing Redis caching and query optimization, improving checkout conversion by 12%."
2. "Redesigned the order-processing service in Node.js, cutting average response time by 65% and supporting 3x traffic growth without new infrastructure."
3. "Led migration of monolithic backend to microservices, decreasing deployment time from 40 min to 6 min and reducing production incidents by 30%."

**💡 Tushuncha:** Har bir variantda — action verb (Reduced, Redesigned, Led) + aniq raqam + biznes/texnik natija.

---

## 8. Recruiter screen savollari

| Savol | Namunaviy javob (yondashuv) |
|-------|------------------------------|
| "O'zingiz haqingizda gapiring" | 2 daqiqalik pitch: hozirgi rol → asosiy tajriba → nega bu vakansiya qiziq. |
| "Nega ish o'zgartiryapsiz?" | Pozitiv: o'sish/yangi challenge izlayman (eski ishni yomonlamang). |
| "Maosh kutilmangiz qancha?" | "Avval rolni tushunsam... sizning budjetingiz qancha?" yoki diapazon (bozorga asoslangan). |
| "Qachon boshlay olasiz?" | Aniq: notice period + tayyorgarlik (masalan 2-4 hafta). |
| "Bizning kompaniya haqida nima bilasiz?" | Oldindan tadqiqot: mahsulot, missiya, so'nggi yangiliklar 1-2 jumla. |

**⚠️ Ehtiyot bo'l:** Maosh savoliga aniq raqam berishga shoshilmang — bu birinchi screen'da "anchor" bo'lib qoladi.

---

← [Behavioral bo'limiga qaytish](../../behavioral/README.md)
