# Dynamic Programming — Yechimlar

Bu yerda [`dsa/11-dynamic-programming.md`](../../dsa/11-dynamic-programming.md) dagi masalalarning to'liq yechimlari: kod, complexity tahlili va izoh bilan.

---

## 1. Fibonacci Number (oson)

`n`-chi Fibonacci sonini `O(1)` xotira bilan hisoblang.

```js
function fib(n) {
  if (n <= 1) return n;
  let prev2 = 0, prev1 = 1; // dp[0], dp[1]
  for (let i = 2; i <= n; i++) {
    const cur = prev1 + prev2;
    prev2 = prev1;
    prev1 = cur;
  }
  return prev1;
}
// Time: O(n), Space: O(1)
```

**Izoh:** `dp[i] = dp[i-1] + dp[i-2]`. Faqat oxirgi ikki qiymat kerak, shuning uchun butun massiv o'rniga ikkita o'zgaruvchi yetadi (rolling array). Bu DP'ning eng oddiy namunasi.

---

## 2. Climbing Stairs (oson)

`n` zinaga 1 yoki 2 zina bosib necha xil usulda chiqasiz?

```js
function climbStairs(n) {
  if (n <= 2) return n;
  let prev2 = 1, prev1 = 2; // 1 va 2 zina uchun usullar
  for (let i = 3; i <= n; i++) {
    const cur = prev1 + prev2;
    prev2 = prev1;
    prev1 = cur;
  }
  return prev1;
}
// Time: O(n), Space: O(1)
```

**Izoh:** `i`-zinaga `i-1` (1 zina) yoki `i-2` (2 zina) dan kelasiz: `dp[i] = dp[i-1] + dp[i-2]`. Bu yashirin Fibonacci. Base case: `dp[1]=1`, `dp[2]=2`.

---

## 3. House Robber (oson)

Qo'shni uylarni o'g'irlamasdan maksimal pul.

```js
function rob(nums) {
  let prev2 = 0; // dp[i-2]
  let prev1 = 0; // dp[i-1]
  for (const num of nums) {
    const cur = Math.max(prev1, prev2 + num);
    prev2 = prev1;
    prev1 = cur;
  }
  return prev1;
}
// Time: O(n), Space: O(1)
```

**Izoh:** Har uyda tanlov — o'g'irlamaslik (`prev1`) yoki o'g'irlash (`prev2 + num`, qo'shnini o'tkazib). `dp[i] = max(dp[i-1], dp[i-2] + nums[i])`. Rolling array bilan `O(1)` xotira.

---

## 4. Min Cost Climbing Stairs (oson)

Har zinada narx bor; eng kam narxda yuqoriga chiqing.

```js
function minCostClimbingStairs(cost) {
  let prev2 = 0, prev1 = 0; // 0 va 1-zinaga yetib kelish narxi
  for (let i = 2; i <= cost.length; i++) {
    const cur = Math.min(prev1 + cost[i - 1], prev2 + cost[i - 2]);
    prev2 = prev1;
    prev1 = cur;
  }
  return prev1;
}
// Time: O(n), Space: O(1)
```

**Izoh:** `dp[i]` = `i`-zinaga yetib kelishning minimal narxi. `i`-zinaga `i-1` dan (narxi `cost[i-1]`) yoki `i-2` dan (narxi `cost[i-2]`) kelish mumkin. Yuqori (top) — `cost.length`-zina. Base case: `dp[0]=dp[1]=0` (boshlash bepul).

---

## 5. Unique Paths (o'rta)

`m × n` panjarada faqat o'ng/past yurib necha xil yo'l.

```js
function uniquePaths(m, n) {
  let row = new Array(n).fill(1); // birinchi qator — hammasi 1
  for (let i = 1; i < m; i++) {
    for (let j = 1; j < n; j++) {
      row[j] += row[j - 1]; // yuqoridagi (row[j]) + chapdagi (row[j-1])
    }
  }
  return row[n - 1];
}
// Time: O(m·n), Space: O(n)
```

**Izoh:** `dp[i][j] = dp[i-1][j] + dp[i][j-1]`. Faqat oldingi qator kerak, shuning uchun bitta qatorni joyida yangilaymiz: `row[j] += row[j-1]` da `row[j]` (yangilanishidan oldin) = yuqoridagi qiymat, `row[j-1]` = chapdagi yangi qiymat. `O(m·n)` xotiradan `O(n)` ga tushdik.

---

## 6. Coin Change (o'rta)

`amount` summa uchun minimal tangalar soni.

```js
function coinChange(coins, amount) {
  const dp = new Array(amount + 1).fill(Infinity);
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
// Time: O(amount · coins.length), Space: O(amount)
```

**Izoh:** `dp[a]` = `a` summa uchun minimal tangalar. Har `coin` uchun `dp[a] = min(dp[a], dp[a-coin] + 1)`. Greedy bu yerda XATO (`[1,3,4]`, `6` → greedy 3, to'g'risi 2). Imkonsiz holat `Infinity` bo'lib qoladi → `-1`.

---

## 7. Longest Common Subsequence (o'rta)

Ikki string'ning eng uzun umumiy subsequence uzunligi.

```ts
function longestCommonSubsequence(text1: string, text2: string): number {
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
    [prev, cur] = [cur, prev];
  }
  return prev[n];
}
// Time: O(m·n), Space: O(n)
```

**Izoh:** Harflar mos kelsa `dp[i][j] = dp[i-1][j-1] + 1`, aks holda `max(dp[i-1][j], dp[i][j-1])`. Faqat ikki qator kerak — rolling array bilan `O(n)` xotira. Almashtirgandan keyin javob `prev[n]` da.

---

## 8. Longest Increasing Subsequence (o'rta)

Eng uzun o'suvchi subsequence uzunligi. `O(n log n)` yechim.

```ts
function lengthOfLIS(nums: number[]): number {
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

**Izoh:** `tails[k]` = uzunligi `k+1` LIS'ning eng kichik mumkin oxirgi elementi. Har element uchun binary search bilan `num` dan kichik bo'lmagan birinchi o'rinni topib almashtiramiz; eng oxirida bo'lsa qo'shamiz. `tails` uzunligi = LIS uzunligi. `O(n²)` DP versiyasi ham bor (`dp[i] = max(dp[j]+1)`), lekin bu tezroq.

---

## 9. Word Break (o'rta)

String'ni lug'atdagi so'zlarga bo'lib bo'ladimi?

```ts
function wordBreak(s: string, wordDict: string[]): boolean {
  const words = new Set(wordDict);
  const dp: boolean[] = new Array(s.length + 1).fill(false);
  dp[0] = true; // bo'sh string — har doim bo'linadi
  for (let i = 1; i <= s.length; i++) {
    for (let j = 0; j < i; j++) {
      if (dp[j] && words.has(s.slice(j, i))) {
        dp[i] = true;
        break;
      }
    }
  }
  return dp[s.length];
}
// Time: O(n² · k), Space: O(n)  (k — slice/lookup narxi)
```

**Izoh:** `dp[i]` = `s[0..i-1]` lug'at so'zlariga bo'linadimi. Har `i` uchun bo'linish nuqtasi `j` ni izlaymiz: agar `dp[j]` true va `s[j..i)` lug'atda bo'lsa, `dp[i] = true`. Base case `dp[0] = true`. `Set` lookup `O(1)` (uzunlikka nisbatan amortizatsiya).

---

## 10. House Robber II (o'rta)

Uylar doira bo'ylab joylashgan (birinchi va oxirgi qo'shni).

```js
function rob(nums) {
  if (nums.length === 1) return nums[0];
  // Doira: birinchi va oxirgi birga bo'la olmaydi.
  // Ikki holatni ko'ramiz: [0..n-2] yoki [1..n-1]
  return Math.max(
    robLinear(nums, 0, nums.length - 2),
    robLinear(nums, 1, nums.length - 1)
  );
}

function robLinear(nums, start, end) {
  let prev2 = 0, prev1 = 0;
  for (let i = start; i <= end; i++) {
    const cur = Math.max(prev1, prev2 + nums[i]);
    prev2 = prev1;
    prev1 = cur;
  }
  return prev1;
}
// Time: O(n), Space: O(1)
```

**Izoh:** Doiraviy joylashuv tufayli birinchi va oxirgi uy bir vaqtda o'g'irlanmaydi. Shuning uchun masalani ikkita oddiy House Robber'ga ajratamiz: oxirgi uysiz (`0..n-2`) va birinchi uysiz (`1..n-1`). Ikkalasidan maksimum — javob.

---

## 11. Coin Change II (o'rta)

`amount` summani yig'ishning necha xil usuli bor (tartib muhim emas).

```js
function change(amount, coins) {
  const dp = new Array(amount + 1).fill(0);
  dp[0] = 1; // 0 summa — bitta usul (hech narsa olmaslik)
  for (const coin of coins) {       // tashqi — coins bo'yicha!
    for (let a = coin; a <= amount; a++) {
      dp[a] += dp[a - coin];
    }
  }
  return dp[amount];
}
// Time: O(amount · coins.length), Space: O(amount)
```

**Izoh:** `dp[a]` = `a` summani yig'ish usullari soni. Bu unbounded knapsack ko'rinishi. **Tsikl tartibi muhim:** tashqi tsikl `coins` bo'yicha bo'lishi shart — shunda har kombinatsiya bir marta sanaladi (tartib hisobga olinmaydi). Agar tsikllarni almashtirsak, permutatsiyalarni sanab, noto'g'ri javob beramiz.

---

## 12. Edit Distance (qiyin)

`word1` ni `word2` ga aylantirish uchun minimal insert/delete/replace soni.

```ts
function minDistance(word1: string, word2: string): number {
  const m = word1.length, n = word2.length;
  let prev: number[] = Array.from({ length: n + 1 }, (_, j) => j); // dp[0][j] = j
  for (let i = 1; i <= m; i++) {
    const cur: number[] = new Array(n + 1);
    cur[0] = i; // dp[i][0] = i
    for (let j = 1; j <= n; j++) {
      if (word1[i - 1] === word2[j - 1]) {
        cur[j] = prev[j - 1];
      } else {
        cur[j] = 1 + Math.min(
          prev[j],      // delete
          cur[j - 1],   // insert
          prev[j - 1]   // replace
        );
      }
    }
    prev = cur;
  }
  return prev[n];
}
// Time: O(m·n), Space: O(n)
```

**Izoh:** `dp[i][j]` = `word1[0..i-1]` ni `word2[0..j-1]` ga aylantirish narxi. Harf mos kelsa operatsiya kerak emas (`prev[j-1]`), aks holda uchta operatsiyaning eng arzonidan `1 + min(delete, insert, replace)`. Base case: bo'sh string narxi = harflar soni. Ikki qator bilan `O(n)` xotira.

---

## 13. 0/1 Knapsack (qiyin)

Sig'imi `W` ryukzakka har buyumni bir marta olib, maksimal qiymat.

```ts
function knapsack01(weights: number[], values: number[], W: number): number {
  const dp: number[] = new Array(W + 1).fill(0);
  for (let i = 0; i < weights.length; i++) {
    // w ni O'NGDAN CHAPGA — har buyum bir marta ishlatilishi uchun
    for (let w = W; w >= weights[i]; w--) {
      dp[w] = Math.max(dp[w], dp[w - weights[i]] + values[i]);
    }
  }
  return dp[W];
}
// Time: O(n·W), Space: O(W)
```

**Izoh:** `dp[w]` = sig'im `w` uchun maksimal qiymat. Har buyumda tanlov — olish yoki olmaslik. **Ichki tsikl o'ngdan chapga** yuradi: shunda `dp[w - weights[i]]` hali joriy buyumni o'z ichiga olmaydi (oldingi buyumlar holati), demak har buyum bir marta ishlatiladi. Agar chapdan o'ngga yursa — unbounded knapsack bo'lib qoladi. Bu nozik detal — intervyuda tushuntiring.

---

## 14. Longest Palindromic Subsequence (qiyin)

String'dagi eng uzun palindrom subsequence uzunligi.

```ts
function longestPalindromeSubseq(s: string): number {
  const n = s.length;
  // dp[i][j] = s[i..j] dagi eng uzun palindrom subsequence
  const dp: number[][] = Array.from({ length: n }, () => new Array(n).fill(0));
  for (let i = n - 1; i >= 0; i--) {
    dp[i][i] = 1; // bir harf — palindrom, uzunligi 1
    for (let j = i + 1; j < n; j++) {
      if (s[i] === s[j]) {
        dp[i][j] = dp[i + 1][j - 1] + 2; // ikki chetdagi harf mos keldi
      } else {
        dp[i][j] = Math.max(dp[i + 1][j], dp[i][j - 1]);
      }
    }
  }
  return dp[0][n - 1];
}
// Time: O(n²), Space: O(n²)
```

**Izoh:** `dp[i][j]` = `s[i..j]` oraliqdagi eng uzun palindrom subsequence. Agar chetlar mos kelsa `dp[i+1][j-1] + 2`, aks holda ichkariroq oraliqlardan maksimum. **To'ldirish tartibi muhim:** `i` ni teskari (pastdan), `j` ni `i` dan o'ngga — chunki `dp[i][j]` `dp[i+1][...]` ga (pastki qatorga) bog'liq. Bu aslida `s` va uning teskarisi orasidagi LCS bilan ham yechiladi.

---

← [DSA bo'limiga qaytish](../../dsa/README.md)
