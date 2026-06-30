# Dynamic Programming

Dynamic Programming (DP) — intervyularning eng qo'rqinchli mavzusi. Aslida DP — bu sehrli algoritm emas, balki **fikrlash usuli**: katta masalani kichik, takrorlanuvchi qism-masalalarga bo'lib, ularning javoblarini bir marta hisoblab, qayta-qayta ishlatish. Bu bo'limda DP'ni noldan, sabr bilan, har bir qadamni ko'rsatib quramiz. Shoshilmang — DP tushunish uchun ko'p mashq kerak, lekin asosiy g'oya juda sodda.

## Mundarija

- [DP nima va qachon ishlatiladi](#dp-nima-va-qachon-ishlatiladi)
- [Ikki shart: optimal substructure va overlapping subproblems](#ikki-shart-optimal-substructure-va-overlapping-subproblems)
- [DP vs Divide-and-Conquer vs Greedy](#dp-vs-divide-and-conquer-vs-greedy)
- [Memoization (top-down) vs Tabulation (bottom-up)](#memoization-top-down-vs-tabulation-bottom-up)
- [DP masalasiga qanday yondashish](#dp-masalasiga-qanday-yondashish)
- [1D DP](#1d-dp)
- [2D DP](#2d-dp)
- [Knapsack (0/1 va Unbounded)](#knapsack-01-va-unbounded)
- [Coin Change](#coin-change)
- [Longest Increasing Subsequence](#longest-increasing-subsequence)
- [Space Optimization (Rolling Array)](#space-optimization-rolling-array)
- [Savol-javoblar](#savol-javoblar)
- [Masalalar](#masalalar)

---

## DP nima va qachon ishlatiladi

Tasavvur qiling, sizdan `fibonacci(5)` ni hisoblash so'raladi. Naive rekursiya quyidagicha:

```js
function fib(n) {
  if (n <= 1) return n;
  return fib(n - 1) + fib(n - 2);
}
```

Bu kod ishlaydi, lekin `fib(5)` hisoblanganda `fib(3)` **uch marta**, `fib(2)` **besh marta** qayta hisoblanadi. Daraxtni chizsak, bir xil tugunlar takror-takror paydo bo'ladi. Bu — `O(2ⁿ)` murakkablik, ya'ni `fib(50)` deyarli abadiy ishlaydi.

DP shu yerda yordamga keladi: agar bir qism-masalani allaqachon hal qilgan bo'lsak, javobni **saqlab qo'yamiz** va keyingi safar qayta hisoblamasdan, tayyor javobni olamiz.

**💡 Tushuncha:** DP — bu "esda saqlash bilan rekursiya" yoki "jadval to'ldirib borish". Asosiy g'oya: *bir qism-masalani ikki marta hisoblama*. Shuning hisobiga eksponensial `O(2ⁿ)` murakkablikni odatda `O(n)` yoki `O(n²)` ga tushiramiz.

DP qachon kerak bo'ladi? Odatda quyidagi so'zlar masalada uchrasa, DP haqida o'ylash kerak:

- "eng katta / eng kichik / optimal" (maximum, minimum, longest, shortest)
- "necha xil usulda" (number of ways, count)
- "mumkinmi" (can you reach / is it possible)
- masalada **tanlovlar** bor: har qadamda bir nechta variantdan birini tanlaysiz, va tanlovlar kelajakka ta'sir qiladi.

---

## Ikki shart: optimal substructure va overlapping subproblems

Masala DP bilan yechiladigan bo'lishi uchun **ikkita shart** bajarilishi kerak.

### 1. Optimal substructure (optimal qism-tuzilma)

Katta masalaning optimal yechimi — kichik qism-masalalarning optimal yechimlaridan tuziladi.

Misol: `A` dan `C` gacha eng qisqa yo'l `B` orqali o'tsa, u holda `A→C` ning qismi bo'lgan `A→B` ham eng qisqa yo'l bo'lishi shart. Demak katta yechim kichik yechimlardan "yig'iladi".

### 2. Overlapping subproblems (qism-masalalar takrorlanishi)

Bir xil qism-masala hisoblash davomida bir necha bor uchraydi. `fib(5)` da `fib(3)` qayta-qayta hisoblanadi — bu overlapping.

**💡 Tushuncha:** Ikkala shart birga bo'lsa — DP. Faqat optimal substructure bo'lsa-yu, overlapping bo'lmasa (har qism-masala bir marta hisoblansa), bu **divide-and-conquer** (masalan, merge sort). Faqat lokal optimal tanlov global optimalga olib kelsa — bu **greedy**.

```
fib(5)
├── fib(4)
│   ├── fib(3)   ← takrorlanadi
│   │   ├── fib(2)
│   │   └── fib(1)
│   └── fib(2)   ← takrorlanadi
└── fib(3)       ← takrorlanadi
    ├── fib(2)   ← takrorlanadi
    └── fib(1)
```

---

## DP vs Divide-and-Conquer vs Greedy

Bu uchtasini chalkashtirish oson. Farqni aniq tushunib oling.

| Xususiyat | Divide & Conquer | Dynamic Programming | Greedy |
|-----------|------------------|---------------------|--------|
| Qism-masalalar takrorlanadimi? | Yo'q (mustaqil) | Ha (overlapping) | — |
| Javoblarni saqlaydimi? | Yo'q | Ha (memo/jadval) | Yo'q |
| Yondashuv | bo'l → yech → birlashtir | barcha variantlarni ko'rib, eng yaxshisini saqla | har qadamda lokal eng yaxshi tanlov |
| Optimal kafolati | — | Har doim global optimal | Faqat ba'zi masalalarda |
| Misol | Merge Sort, Quick Sort | Knapsack, LCS, Coin Change | Dijkstra, Huffman, interval scheduling |

**💡 Tushuncha:** Greedy tezroq (`O(n log n)` odatda) lekin doim to'g'ri javob bermaydi. DP barcha variantlarni ko'rib chiqadi, shuning uchun sekinroq lekin ishonchli. Coin Change masalasida greedy ba'zi tangalar to'plamida xato javob beradi, DP esa har doim to'g'ri.

**⚠️ Ehtiyot bo'l:** "Eng katta sonni har safar olaman" kabi greedy yondashuv DP masalasini yechmaydi. Greedy ishlashiga ishonch hosil qilmasangiz — DP ishlatish xavfsizroq.

---

## Memoization (top-down) vs Tabulation (bottom-up)

DP ni amalga oshirishning ikki uslubi bor. Ikkalasi ham bir xil natija beradi, lekin yo'nalishi har xil.

### Memoization (top-down) — rekursiya + cache

Naive rekursiyani saqlaymiz va javoblarni `memo` (cache) da saqlaymiz. "Yuqoridan pastga": katta masaladan boshlab, kichiklariga tushamiz.

```js
function fib(n, memo = {}) {
  if (n <= 1) return n;
  if (n in memo) return memo[n]; // tayyor javob bormi?
  memo[n] = fib(n - 1, memo) + fib(n - 2, memo);
  return memo[n];
}
// Time: O(n), Space: O(n) memo + O(n) call stack
```

**Afzalligi:** asl rekursiyaga yaqin, yozish oson. Faqat kerakli qism-masalalar hisoblanadi.
**Kamchiligi:** chuqur rekursiyada stack overflow xavfi bor.

### Tabulation (bottom-up) — jadval to'ldirish

"Pastdan yuqoriga": eng kichik holatdan boshlab, jadvalni ketma-ket to'ldiramiz. Rekursiya yo'q, faqat tsikl.

```js
function fib(n) {
  if (n <= 1) return n;
  const dp = new Array(n + 1);
  dp[0] = 0;
  dp[1] = 1;
  for (let i = 2; i <= n; i++) {
    dp[i] = dp[i - 1] + dp[i - 2]; // oldingilardan quramiz
  }
  return dp[n];
}
// Time: O(n), Space: O(n)
```

**Afzalligi:** stack overflow yo'q, ko'pincha tezroq, space optimizatsiya qulay.
**Kamchiligi:** to'ldirish tartibini o'ylab topish kerak, ba'zan barcha holatlar hisoblanadi (keraksizlari ham).

**💡 Tushuncha:** Intervyuda istalganini ishlatishingiz mumkin. Top-down — agar rekurrentlikni topgan bo'lsangiz, tez yozish uchun. Bottom-up — space optimizatsiya qilmoqchi bo'lsangiz. Ko'p tajribali muhandislar avval top-down yozadi, keyin kerak bo'lsa bottom-up'ga aylantiradi.

---

## DP masalasiga qanday yondashish

DP masalasini ko'rganda panika qilmang. Quyidagi **5 qadamli retsept** ni ishlating:

### 1-qadam: State (holat) ni aniqlang

`dp[i]` (yoki `dp[i][j]`) nimani anglatadi? Bu eng muhim qadam. Masalan:
- "`dp[i]` = `i`-uydan boshlab o'g'irlash mumkin bo'lgan maksimal pul"
- "`dp[i][j]` = `s1` ning birinchi `i` harfi va `s2` ning birinchi `j` harfi uchun LCS uzunligi"

### 2-qadam: Recurrence / transition (o'tish) ni yozing

`dp[i]` ni oldingi holatlardan qanday hisoblaysiz? Bu yerda "tanlov" paydo bo'ladi:

```
dp[i] = max(dp[i - 1], dp[i - 2] + nums[i])  // house robber
```

### 3-qadam: Base case (boshlang'ich holat) ni aniqlang

Eng kichik holat(lar)ning javobi. Masalan `dp[0] = 0`, `dp[1] = nums[0]`.

### 4-qadam: Hisoblash tartibini aniqlang

Bottom-up uchun: `dp[i]` `dp[i-1]` ga bog'liq bo'lsa, chapdan o'ngga yuramiz. 2D'da ko'pincha yuqori-chapdan pastki-o'ngga.

### 5-qadam: Javob qayerda?

Ko'pincha `dp[n]` yoki `dp[n][m]`, lekin ba'zan butun jadvaldagi maksimum (LIS kabi).

**💡 Tushuncha:** State'ni to'g'ri aniqlash — jangning 80%. Agar `dp[i]` nima ekanligini bir jumlada aniq ayta olsangiz, recurrence o'zi kelib chiqadi.

---

## 1D DP

Eng oddiy DP turi: holat bitta indeks bilan ifodalanadi.

### Climbing Stairs

`n` zinapoyaga ko'tarilyapsiz. Har safar 1 yoki 2 zina bosishingiz mumkin. Necha xil usulda yuqoriga chiqasiz?

**State:** `dp[i]` = `i`-zinaga chiqishning usullari soni.
**Recurrence:** `i`-zinaga 1 zina (`i-1` dan) yoki 2 zina (`i-2` dan) bosib kelasiz. Demak `dp[i] = dp[i-1] + dp[i-2]`.
**Base case:** `dp[0] = 1` (joyda turish — bitta usul), `dp[1] = 1`.

```js
function climbStairs(n) {
  if (n <= 2) return n;
  const dp = new Array(n + 1);
  dp[0] = 1;
  dp[1] = 1;
  for (let i = 2; i <= n; i++) {
    dp[i] = dp[i - 1] + dp[i - 2];
  }
  return dp[n];
}
// Time: O(n), Space: O(n)
```

E'tibor bering — bu aynan Fibonacci! Ko'p DP masalalari yashirin Fibonacci.

### House Robber

`nums` — uylar bo'yicha pul miqdori. **Qo'shni** uylarni o'g'irlay olmaysiz (signalizatsiya yonadi). Maksimal qancha pul o'g'irlash mumkin?

**State:** `dp[i]` = `0..i` uylar orasida o'g'irlash mumkin bo'lgan maksimal pul.
**Recurrence:** Har uyda **tanlov** bor: yo o'g'irlaysiz (u holda `i-1` ni o'tkazib yuborasiz), yo yo'q.
```
dp[i] = max(dp[i - 1],              // i-uyni o'g'irlamaslik
            dp[i - 2] + nums[i])    // i-uyni o'g'irlash
```
**Base case:** `dp[0] = nums[0]`, `dp[1] = max(nums[0], nums[1])`.

```js
function rob(nums) {
  const n = nums.length;
  if (n === 0) return 0;
  if (n === 1) return nums[0];
  const dp = new Array(n);
  dp[0] = nums[0];
  dp[1] = Math.max(nums[0], nums[1]);
  for (let i = 2; i < n; i++) {
    dp[i] = Math.max(dp[i - 1], dp[i - 2] + nums[i]);
  }
  return dp[n - 1];
}
// Time: O(n), Space: O(n)
```

**💡 Tushuncha:** House Robber — DP'ning klassik "tanlov" namunasi. "Olaman yoki olmayman" mantiqi knapsack'gacha boradigan g'oyaning urug'i.

---

## 2D DP

State ikkita indeks bilan ifodalanadi — odatda ikkita ketma-ketlik yoki grid (panjara) bilan ishlaganda.

### Grid Paths (Unique Paths)

`m × n` panjarada chap-yuqori burchakdan o'ng-pastki burchakka borish kerak. Faqat **pastga** yoki **o'ngga** yurish mumkin. Necha xil yo'l bor?

**State:** `dp[i][j]` = `(0,0)` dan `(i,j)` ga borish usullari soni.
**Recurrence:** `(i,j)` ga faqat yuqoridan (`i-1,j`) yoki chapdan (`i,j-1`) kelish mumkin:
```
dp[i][j] = dp[i - 1][j] + dp[i][j - 1]
```
**Base case:** birinchi qator va birinchi ustun — hammasi `1` (faqat bir yo'l bor).

```js
function uniquePaths(m, n) {
  const dp = Array.from({ length: m }, () => new Array(n).fill(1));
  for (let i = 1; i < m; i++) {
    for (let j = 1; j < n; j++) {
      dp[i][j] = dp[i - 1][j] + dp[i][j - 1];
    }
  }
  return dp[m - 1][n - 1];
}
// Time: O(m·n), Space: O(m·n)
```

### Longest Common Subsequence (LCS)

Ikki string `text1`, `text2` berilgan. Eng uzun umumiy **subsequence** (ketma-ket bo'lmasa ham, tartibni saqlagan) uzunligini toping. Masalan `"abcde"` va `"ace"` → `"ace"` → 3.

**State:** `dp[i][j]` = `text1[0..i-1]` va `text2[0..j-1]` uchun LCS uzunligi.
**Recurrence:**
```
agar text1[i-1] === text2[j-1]:  dp[i][j] = dp[i-1][j-1] + 1   // harf mos keldi
aks holda:                        dp[i][j] = max(dp[i-1][j], dp[i][j-1])
```
**Base case:** `dp[0][*] = 0`, `dp[*][0] = 0` (bo'sh string bilan LCS = 0).

```ts
function longestCommonSubsequence(text1: string, text2: string): number {
  const m = text1.length, n = text2.length;
  const dp: number[][] = Array.from({ length: m + 1 }, () =>
    new Array(n + 1).fill(0)
  );
  for (let i = 1; i <= m; i++) {
    for (let j = 1; j <= n; j++) {
      if (text1[i - 1] === text2[j - 1]) {
        dp[i][j] = dp[i - 1][j - 1] + 1;
      } else {
        dp[i][j] = Math.max(dp[i - 1][j], dp[i][j - 1]);
      }
    }
  }
  return dp[m][n];
}
// Time: O(m·n), Space: O(m·n)
```

**💡 Tushuncha:** 2D DP'da `dp` jadvalining o'lchami `(m+1) × (n+1)` qilinadi — bo'sh string holatini ham hisobga olish uchun. Bu base case'ni soddalashtiradi.

### Edit Distance (Levenshtein)

`word1` ni `word2` ga aylantirish uchun minimal nechta operatsiya kerak? Operatsiyalar: **insert**, **delete**, **replace** (har biri 1 narx).

**State:** `dp[i][j]` = `word1[0..i-1]` ni `word2[0..j-1]` ga aylantirish narxi.
**Recurrence:**
```
agar word1[i-1] === word2[j-1]:  dp[i][j] = dp[i-1][j-1]   // o'zgartirish kerak emas
aks holda: dp[i][j] = 1 + min(
    dp[i-1][j],     // delete (word1'dan harf o'chir)
    dp[i][j-1],     // insert (word2'dan harf qo'sh)
    dp[i-1][j-1]    // replace (harfni almashtir)
)
```
**Base case:** `dp[i][0] = i` (hamma harfni o'chirish), `dp[0][j] = j` (hamma harfni qo'shish).

```ts
function minDistance(word1: string, word2: string): number {
  const m = word1.length, n = word2.length;
  const dp: number[][] = Array.from({ length: m + 1 }, () =>
    new Array(n + 1).fill(0)
  );
  for (let i = 0; i <= m; i++) dp[i][0] = i;
  for (let j = 0; j <= n; j++) dp[0][j] = j;

  for (let i = 1; i <= m; i++) {
    for (let j = 1; j <= n; j++) {
      if (word1[i - 1] === word2[j - 1]) {
        dp[i][j] = dp[i - 1][j - 1];
      } else {
        dp[i][j] = 1 + Math.min(
          dp[i - 1][j],     // delete
          dp[i][j - 1],     // insert
          dp[i - 1][j - 1]  // replace
        );
      }
    }
  }
  return dp[m][n];
}
// Time: O(m·n), Space: O(m·n)
```

**⚠️ Ehtiyot bo'l:** Edit Distance'da base case'ni unutmang. `dp[0][j] = j` va `dp[i][0] = i` qo'yilmasa, bo'sh string holatlari noto'g'ri bo'ladi va butun jadval buziladi.

---

## Knapsack (0/1 va Unbounded)

Knapsack (ryukzak) — DP'ning "kitobchasidagi" eng mashhur masalasi. Sig'imi `W` bo'lgan ryukzakka buyumlar joylaymiz. Har buyumning `weight` (og'irligi) va `value` (qiymati) bor. Maksimal umumiy qiymatni toping.

### 0/1 Knapsack

Har buyumni **faqat bir marta** olish mumkin (yo olasiz, yo yo'q — "0 yoki 1").

**State:** `dp[i][w]` = birinchi `i` buyum va sig'im `w` uchun maksimal qiymat.
**Recurrence:** har buyumda tanlov — olish yoki olmaslik:
```
agar weight[i-1] > w:  dp[i][w] = dp[i-1][w]   // sig'maydi, olmaymiz
aks holda: dp[i][w] = max(
    dp[i-1][w],                              // olmaymiz
    dp[i-1][w - weight[i-1]] + value[i-1]    // olamiz
)
```

```ts
function knapsack01(weights: number[], values: number[], W: number): number {
  const n = weights.length;
  const dp: number[][] = Array.from({ length: n + 1 }, () =>
    new Array(W + 1).fill(0)
  );
  for (let i = 1; i <= n; i++) {
    for (let w = 0; w <= W; w++) {
      dp[i][w] = dp[i - 1][w]; // olmaslik
      if (weights[i - 1] <= w) {
        dp[i][w] = Math.max(
          dp[i][w],
          dp[i - 1][w - weights[i - 1]] + values[i - 1] // olish
        );
      }
    }
  }
  return dp[n][W];
}
// Time: O(n·W), Space: O(n·W)
```

### Unbounded Knapsack

Har buyumni **cheksiz marta** olish mumkin. Yagona farq — buyumni olganimizda `dp[i-1]` emas, `dp[i]` (o'sha buyumni yana olishimiz mumkin) dan foydalanamiz.

```ts
function knapsackUnbounded(weights: number[], values: number[], W: number): number {
  const n = weights.length;
  const dp: number[] = new Array(W + 1).fill(0);
  for (let i = 0; i < n; i++) {
    for (let w = weights[i]; w <= W; w++) {
      // w chapdan o'ngga yuradi — bir buyumni qayta ishlatish uchun
      dp[w] = Math.max(dp[w], dp[w - weights[i]] + values[i]);
    }
  }
  return dp[W];
}
// Time: O(n·W), Space: O(W)
```

**💡 Tushuncha:** 0/1 va unbounded knapsack farqi — ichki tsikl yo'nalishida (1D versiyada). 0/1'da `w` ni **o'ngdan chapga** (har buyum bir marta), unbounded'da **chapdan o'ngga** (qayta ishlatish) yuriladi. Bu nozik, lekin juda muhim detal.

**⚠️ Ehtiyot bo'l:** `dp[i][w]` da `W` butun son bo'lishi shart — knapsack DP "pseudo-polynomial", ya'ni murakkablik `W` qiymatiga bog'liq, uning bit-uzunligiga emas. Juda katta `W` da bu sekin bo'ladi.

---

## Coin Change

Tangalar to'plami `coins` va summa `amount` berilgan. Bu summani yig'ish uchun **minimal tangalar soni**ni toping (har tanga cheksiz). Imkonsiz bo'lsa `-1` qaytaring.

Bu — unbounded knapsack'ning bir ko'rinishi.

**State:** `dp[a]` = `a` summani yig'ish uchun minimal tangalar soni.
**Recurrence:**
```
dp[a] = min(dp[a - coin] + 1)  // har coin uchun
```
**Base case:** `dp[0] = 0` (0 summa uchun 0 tanga). Qolganlari `Infinity` (hali topilmagan).

```ts
function coinChange(coins: number[], amount: number): number {
  const dp: number[] = new Array(amount + 1).fill(Infinity);
  dp[0] = 0;
  for (let a = 1; a <= amount; a++) {
    for (const coin of coins) {
      if (coin <= a) {
        dp[a] = Math.min(dp[a], dp[a - coin] + 1);
      }
    }
  }
  return dp[amount] === Infinity ? -1 : dp[amount];
}
// Time: O(amount·coins.length), Space: O(amount)
```

**⚠️ Ehtiyot bo'l:** Greedy ("har safar eng katta tanga") bu yerda XATO! `coins = [1, 3, 4]`, `amount = 6` uchun greedy `4+1+1 = 3` tanga deydi, lekin to'g'ri javob `3+3 = 2` tanga. Coin Change DP' siz to'g'ri yechilmaydi.

**💡 Tushuncha:** "Necha xil usulda summani yig'ish mumkin" (Coin Change II) — bu boshqa masala. U yerda tartib muhim emas, shuning uchun tsikllar tartibini almashtirishingiz kerak: tashqi tsikl coins bo'yicha, ichki — amount bo'yicha. Tsikl tartibi javobni o'zgartiradi!

---

## Longest Increasing Subsequence

`nums` massivida eng uzun **o'suvchi** subsequence uzunligini toping. Masalan `[10, 9, 2, 5, 3, 7, 101, 18]` → `[2, 3, 7, 101]` → 4.

### O(n²) yechim

**State:** `dp[i]` = `i`-element bilan tugaydigan eng uzun o'suvchi subsequence uzunligi.
**Recurrence:** `i` dan oldingi har bir `j` ni ko'rib, agar `nums[j] < nums[i]` bo'lsa, `dp[i]` ni yangilaymiz:
```
dp[i] = max(dp[j] + 1) har j < i uchun, agar nums[j] < nums[i]
```
**Base case:** har bir `dp[i] = 1` (har element o'zi 1 uzunlikdagi subsequence).

```ts
function lengthOfLIS(nums: number[]): number {
  const n = nums.length;
  if (n === 0) return 0;
  const dp: number[] = new Array(n).fill(1);
  let max = 1;
  for (let i = 1; i < n; i++) {
    for (let j = 0; j < i; j++) {
      if (nums[j] < nums[i]) {
        dp[i] = Math.max(dp[i], dp[j] + 1);
      }
    }
    max = Math.max(max, dp[i]);
  }
  return max;
}
// Time: O(n²), Space: O(n)
```

### O(n log n) yechim (binary search bilan)

`tails` massivini saqlaymiz: `tails[k]` = uzunligi `k+1` bo'lgan o'suvchi subsequence'ning eng kichik mumkin oxirgi elementi. Har element uchun binary search bilan o'rnini topamiz.

```ts
function lengthOfLISFast(nums: number[]): number {
  const tails: number[] = [];
  for (const num of nums) {
    let lo = 0, hi = tails.length;
    while (lo < hi) {
      const mid = (lo + hi) >> 1;
      if (tails[mid] < num) lo = mid + 1;
      else hi = mid;
    }
    if (lo === tails.length) tails.push(num);
    else tails[lo] = num;
  }
  return tails.length;
}
// Time: O(n log n), Space: O(n)
```

**💡 Tushuncha:** `tails` massivi LIS'ning o'zi EMAS — uning uzunligi to'g'ri, lekin elementlari haqiqiy subsequence bo'lmasligi mumkin. Bu trik faqat **uzunlikni** topadi. Intervyuda `O(n log n)` ni bilsangiz katta plus.

---

## Space Optimization (Rolling Array)

Ko'pgina DP'larda `dp[i]` faqat oxirgi bir-ikki holatga bog'liq. Demak butun massivni saqlash shart emas — faqat kerakli oxirgi qiymatlarni saqlaymiz. Bu **rolling array** texnikasi.

### 1D misol: Fibonacci/Climbing Stairs

`dp[i]` faqat `dp[i-1]` va `dp[i-2]` ga bog'liq. Demak ikkita o'zgaruvchi yetadi:

```js
function climbStairs(n) {
  if (n <= 2) return n;
  let prev2 = 1, prev1 = 2; // dp[1], dp[2]
  for (let i = 3; i <= n; i++) {
    const cur = prev1 + prev2;
    prev2 = prev1;
    prev1 = cur;
  }
  return prev1;
}
// Time: O(n), Space: O(1)  ← O(n) dan O(1) ga!
```

### 2D misol: LCS space optimizatsiyasi

LCS'da `dp[i][j]` faqat oldingi qator (`dp[i-1]`) va joriy qatorga bog'liq. Demak butun `(m+1)×(n+1)` jadval o'rniga **ikkita qator** yetadi:

```ts
function lcsOptimized(text1: string, text2: string): number {
  const m = text1.length, n = text2.length;
  let prev = new Array(n + 1).fill(0);
  let cur = new Array(n + 1).fill(0);
  for (let i = 1; i <= m; i++) {
    for (let j = 1; j <= n; j++) {
      if (text1[i - 1] === text2[j - 1]) {
        cur[j] = prev[j - 1] + 1;
      } else {
        cur[j] = Math.max(prev[j], cur[j - 1]);
      }
    }
    [prev, cur] = [cur, prev]; // qatorlarni almashtirib, rolling
  }
  return prev[n];
}
// Time: O(m·n), Space: O(n)  ← O(m·n) dan O(n) ga!
```

**💡 Tushuncha:** Rolling array murakkablikni o'zgartirmaydi (vaqt baribir bir xil), lekin xotirani sezilarli kamaytiradi. Intervyuda avval to'liq jadval bilan yeching (tushunarli), keyin "men buni `O(n)` xotiraga optimizatsiya qila olaman" deb qo'shsangiz — bu yetuklik belgisi.

**⚠️ Ehtiyot bo'l:** Space optimizatsiya qilganda jadvalni qayta tiklash (path reconstruction) imkonsiz bo'lib qoladi. Agar masalada nafaqat javob, balki *qaysi elementlar* ekanligi so'ralsa — to'liq jadvalni saqlash kerak.

---

## Savol-javoblar

### ❓ Dynamic Programming nima va u rekursiyadan qanday farq qiladi?

**✅ Javob:** DP — bu katta masalani takrorlanuvchi qism-masalalarga bo'lib, ularning javoblarini bir marta hisoblab saqlaydigan optimizatsiya texnikasi. Oddiy rekursiya bir xil qism-masalani qayta-qayta hisoblaydi (`fib` da `O(2ⁿ)`). DP esa javoblarni `memo` yoki jadvalda saqlab, qayta hisoblashni oldini oladi va murakkablikni odatda `O(n)` yoki `O(n²)` ga tushiradi. DP — rekursiyaning o'zi emas, balki uning ustiga qurilgan "esda saqlash" qatlami.

### ❓ Masala DP bilan yechiladimi yoki yo'qligini qanday bilasiz?

**✅ Javob:** Ikki shartni tekshiraman: (1) **optimal substructure** — katta yechim kichik yechimlardan tuziladimi, (2) **overlapping subproblems** — bir xil qism-masala qayta-qayta uchraydimi. Shuningdek masalada "maksimal/minimal", "necha xil usulda", "mumkinmi" so'zlari bo'lsa va har qadamda **tanlov** bo'lsa — bu DP signali. Agar qism-masalalar takrorlanmasa — bu divide-and-conquer.

### ❓ Memoization va tabulation orasidagi farq nima?

**✅ Javob:** Ikkalasi ham DP, lekin yo'nalishi har xil. **Memoization (top-down)** — rekursiya + cache, katta masaladan kichiklariga "tushadi", faqat kerakli qism-masalalarni hisoblaydi, lekin stack overflow xavfi bor. **Tabulation (bottom-up)** — tsikl bilan jadvalni eng kichikdan to'ldiradi, stack ishlatmaydi, space optimizatsiya qulay, lekin to'ldirish tartibini o'ylash kerak. Natija bir xil, tanlov vaziyatga bog'liq.

### ❓ DP, divide-and-conquer va greedy orasidagi farqni tushuntiring.

**✅ Javob:** **Divide-and-conquer** masalani mustaqil (takrorlanmaydigan) qismlarga bo'ladi va birlashtiradi (merge sort). **DP** takrorlanuvchi qismlarni saqlab, barcha variantlarni ko'rib chiqib global optimalni topadi (knapsack). **Greedy** har qadamda lokal eng yaxshi tanlovni qiladi — tezroq, lekin faqat ba'zi masalalarda to'g'ri ishlaydi (Dijkstra ha, Coin Change yo'q). Asosiy farq: DP hamma variantni ko'radi, greedy bittasini tanlaydi.

### ❓ DP masalasiga qanday yondashasiz? Qadamlarni ayting.

**✅ Javob:** Besh qadam: (1) **State** ni aniqlayman — `dp[i]` aniq nimani anglatadi, (2) **recurrence/transition** yozaman — `dp[i]` ni oldingi holatlardan qanday hisoblash, (3) **base case** ni belgilayman, (4) **hisoblash tartibi**ni aniqlayman (bottom-up uchun), (5) **javob qayerda** ekanligini topaman (`dp[n]` yoki maksimum). State'ni to'g'ri aniqlash — eng muhim qadam, qolgani undan kelib chiqadi.

### ❓ DP yechimning vaqt va xotira murakkabligini qanday hisoblaysiz?

**✅ Javob:** **Vaqt** = (state'lar soni) × (har state'ni hisoblash narxi). Masalan LCS'da `m·n` ta state, har biri `O(1)` — demak `O(m·n)`. **Xotira** = jadval o'lchami, lekin rolling array bilan kamaytirilishi mumkin. LIS `O(n²)` da `n` ta state, har biri ichki `O(n)` tsikl — `O(n²)`. State'lar sonini sanash murakkablikni topishning eng oson yo'li.

### ❓ House Robber masalasida recurrence'ni tushuntiring.

**✅ Javob:** `dp[i]` = `0..i` uylar orasida maksimal pul. Har uyda tanlov: (1) `i`-uyni o'g'irlamayman → `dp[i-1]`, yoki (2) o'g'irlayman → `dp[i-2] + nums[i]` (qo'shni `i-1` ni o'tkazib yuboraman). Demak `dp[i] = max(dp[i-1], dp[i-2] + nums[i])`. Base case: `dp[0] = nums[0]`, `dp[1] = max(nums[0], nums[1])`. Bu klassik "olaman/olmayman" tanlov namunasi.

### ❓ 0/1 knapsack va unbounded knapsack farqi nimada?

**✅ Javob:** **0/1**'da har buyumni faqat bir marta olish mumkin — buyumni olganda `dp[i-1][...]` (oldingi buyumlar holati) ishlatiladi. **Unbounded**'da cheksiz marta olish mumkin — buyumni olganda `dp[i][...]` (o'sha buyumni yana ishlatish mumkin) ishlatiladi. 1D versiyada farq tsikl yo'nalishida: 0/1 sig'im bo'yicha o'ngdan chapga, unbounded chapdan o'ngga yuriladi.

### ❓ Coin Change masalasida nega greedy ishlamaydi?

**✅ Javob:** Greedy har safar eng katta tangani oladi, lekin bu global optimalni kafolatlamaydi. Misol: `coins = [1, 3, 4]`, `amount = 6`. Greedy `4 + 1 + 1 = 3` tanga deydi, lekin to'g'ri javob `3 + 3 = 2` tanga. DP barcha kombinatsiyalarni ko'rib chiqadi: `dp[a] = min(dp[a - coin] + 1)`, shuning uchun har doim minimal tangalar sonini topadi.

### ❓ Top-down yechimda stack overflow'ni qanday oldini olasiz?

**✅ Javob:** Eng ishonchli yo'l — bottom-up (tabulation) ga o'tish, u rekursiyani umuman ishlatmaydi. Agar top-down qolsa, holatlar sonini kamaytirish yoki ba'zi tillarda rekursiya chuqurligi limitini oshirish mumkin. JS'da chuqur rekursiya (masalan `n = 100000`) stack overflow beradi, shuning uchun katta `n` da bottom-up afzalroq.

### ❓ DP jadvalini to'ldirish tartibi nega muhim?

**✅ Javob:** `dp[i]` ni hisoblashda kerakli oldingi holatlar **allaqachon hisoblangan** bo'lishi shart. Agar `dp[i]` `dp[i-1]` ga bog'liq bo'lsa, chapdan o'ngga yuramiz. 2D'da `dp[i][j]` yuqori va chap kataklarga bog'liq bo'lsa, yuqori-chapdan pastki-o'ngga yuramiz. Tartib noto'g'ri bo'lsa, hali hisoblanmagan (yoki noto'g'ri) qiymatlarni ishlatamiz — javob xato chiqadi. Coin Change II'da tartib hatto javob mazmunini o'zgartiradi.

### ❓ Rolling array (space optimization) qanday ishlaydi va qachon qo'llaysiz?

**✅ Javob:** Agar `dp[i]` faqat oxirgi bir necha holatga bog'liq bo'lsa (masalan `dp[i-1]`, `dp[i-2]`), butun massivni saqlash shart emas — faqat o'sha bir necha qiymatni o'zgaruvchilarda saqlaymiz. Fibonacci'da `O(n)` → `O(1)`, 2D DP'da to'liq jadval o'rniga ikki qator → `O(n)`. Vaqt murakkabligi o'zgarmaydi, faqat xotira kamayadi. Faqat path reconstruction kerak bo'lmasa qo'llanadi, chunki optimizatsiyadan keyin yo'lni tiklab bo'lmaydi.

### ❓ LIS'da O(n²) va O(n log n) yechimlar qanday farq qiladi?

**✅ Javob:** `O(n²)` yechim klassik DP: `dp[i]` = `i` bilan tugaydigan LIS uzunligi, har `i` uchun barcha oldingi `j` larni ko'radi. `O(n log n)` yechim `tails` massivini saqlaydi (`tails[k]` = uzunligi `k+1` LIS'ning eng kichik oxirgi elementi) va har element o'rnini binary search bilan topadi. Tezroq, lekin `tails` haqiqiy subsequence emas — faqat to'g'ri **uzunlik**ni beradi. Agar subsequence'ning o'zi kerak bo'lsa, qo'shimcha ish kerak.

### ❓ Edit Distance recurrence'sidagi uchta tanlov nimani anglatadi?

**✅ Javob:** Harflar mos kelmasa, uchta operatsiyadan birini tanlaymiz: `dp[i-1][j]` — **delete** (`word1` dan harf o'chirish), `dp[i][j-1]` — **insert** (`word2` dan harf qo'shish), `dp[i-1][j-1]` — **replace** (harfni almashtirish). Eng arzonini olib, ustiga 1 qo'shamiz: `1 + min(...)`. Harflar mos kelsa, operatsiya kerak emas: `dp[i][j] = dp[i-1][j-1]`. Base case: bo'sh stringga aylantirish narxi = harflar soni.

### ❓ 2D DP jadvalining o'lchami nega ko'pincha (m+1)×(n+1) bo'ladi?

**✅ Javob:** Qo'shimcha qator va ustun **bo'sh string / nol holat** uchun base case'ni saqlaydi. Masalan LCS'da `dp[0][j] = 0` (bo'sh string bilan LCS = 0), Edit Distance'da `dp[i][0] = i`. Bu indekslashni soddalashtiradi: `dp[i][j]` `i`-chi va `j`-chi harflargacha bo'lgan holatni anglatadi, indeks `1` dan boshlanadi, `0` esa "hech narsa" holatini. Aks holda base case'larni alohida tsiklda boshqarish kerak bo'lardi.

---

## Masalalar

> Yechimlar: [`solutions/dsa/11-dynamic-programming.md`](../solutions/dsa/11-dynamic-programming.md)

1. **Fibonacci Number** (oson) — `n`-chi Fibonacci sonini DP bilan hisoblang. `O(1)` xotira bilan yeching.
2. **Climbing Stairs** (oson) — `n` zinaga 1 yoki 2 zina bosib necha xil usulda chiqasiz?
3. **House Robber** (oson) — qo'shni uylarni o'g'irlamasdan maksimal pul to'plang.
4. **Min Cost Climbing Stairs** (oson) — har zinada narx bor; eng kam narxda yuqoriga chiqing (0 yoki 1-zinadan boshlash mumkin).
5. **Unique Paths** (o'rta) — `m × n` panjarada faqat o'ng/past yurib, necha xil yo'l bilan oxirgi katakka borasiz?
6. **Coin Change** (o'rta) — `amount` summani yig'ish uchun minimal tangalar soni (imkonsiz bo'lsa `-1`).
7. **Longest Common Subsequence** (o'rta) — ikki string'ning eng uzun umumiy subsequence uzunligi.
8. **Longest Increasing Subsequence** (o'rta) — massivdagi eng uzun o'suvchi subsequence uzunligi.
9. **Word Break** (o'rta) — string'ni lug'atdagi so'zlarga bo'lib bo'ladimi?
10. **House Robber II** (o'rta) — uylar **doira** bo'ylab joylashgan (birinchi va oxirgi qo'shni); maksimal pul.
11. **Coin Change II** (o'rta) — `amount` summani yig'ishning necha xil usuli bor (tartib muhim emas)?
12. **Edit Distance** (qiyin) — bir string'ni boshqasiga aylantirish uchun minimal insert/delete/replace soni.
13. **0/1 Knapsack** (qiyin) — sig'imi `W` ryukzakka har buyumni bir marta olib, maksimal qiymat to'plang.
14. **Longest Palindromic Subsequence** (qiyin) — string'dagi eng uzun palindrom subsequence uzunligi.

---

← [DSA bo'limiga qaytish](./README.md)
