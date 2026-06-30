# Recursion va Backtracking — Yechimlar

Bu bo'lim `10-recursion-backtracking.md` dagi 12 ta masalaning to'liq ishlaydigan JS yechimlarini, complexity tahlilini va o'zbekcha izohlarni o'z ichiga oladi.

## 1. Power funksiya

`pow(x, n)` ni rekursiv, O(log n) da yozamiz (fast exponentiation). Manfiy `n` ni ham qo'llab-quvvatlaymiz.

```js
function myPow(x, n) {
  if (n < 0) return 1 / myPow(x, -n); // manfiy daraja
  if (n === 0) return 1;              // base case
  const half = myPow(x, Math.floor(n / 2));
  if (n % 2 === 0) return half * half;
  return half * half * x;             // toq daraja
}

console.log(myPow(2, 10)); // 1024
console.log(myPow(2, -2)); // 0.25
console.log(myPow(3, 0));  // 1
```

- **Time:** O(log n) — har qadamda daraja yarmiga kamayadi.
- **Space:** O(log n) — rekursiya chuqurligi.

**Izoh:** `x^n = (x^(n/2))²` xususiyatidan foydalanamiz — har chaqiriqda darajani ikkiga bo'lib, O(n) o'rniga O(log n) ga erishamiz. `half` ni bir marta hisoblab qayta ishlatish muhim (ikki marta `myPow` chaqirsak O(n) bo'lib qoladi).

---

## 2. String teskari aylantirish

Stringni rekursiya bilan teskari aylantiramiz (tsiklsiz).

```js
function reverseString(s) {
  if (s.length <= 1) return s;                  // base case
  return reverseString(s.slice(1)) + s[0];      // qolgani teskari + birinchi
}

console.log(reverseString('hello')); // "olleh"
console.log(reverseString('a'));     // "a"
console.log(reverseString(''));      // ""
```

- **Time:** O(n²) — har chaqiriqda `slice` O(n), n ta chaqiriq.
- **Space:** O(n) rekursiya stack + slice'lar.

**Izoh:** Base case — bo'sh yoki bitta belgili string (o'zi teskari). Recursive case — birinchi belgini olib tashlab, qolganini teskari aylantirib, birinchi belgini **oxiriga** qo'shamiz. (Optimal in-place variant ikki pointer bilan O(n) bo'lardi, lekin masala rekursiyani so'raydi.)

---

## 3. Subsets

Unikal sonlar massivining barcha qism-to'plamlarini (power set) backtracking bilan qaytaramiz.

```js
function subsets(nums) {
  const result = [];
  function backtrack(start, path) {
    result.push([...path]);              // har holat yaroqli qism-to'plam
    for (let i = start; i < nums.length; i++) {
      path.push(nums[i]);                // CHOOSE
      backtrack(i + 1, path);            // EXPLORE (i+1 → takror yo'q)
      path.pop();                        // UNCHOOSE
    }
  }
  backtrack(0, []);
  return result;
}

console.log(subsets([1, 2, 3]));
// [[],[1],[1,2],[1,2,3],[1,3],[2],[2,3],[3]]
```

- **Time:** O(n · 2ⁿ) — 2ⁿ qism-to'plam, har birini nusxalash O(n).
- **Space:** O(n) — rekursiya chuqurligi (natijani hisobga olmaganda).

**Izoh:** Subsets'da yechim **har tugunda** push qilinadi (alohida base case yo'q), chunki har oraliq holat ham yaroqli qism-to'plam. `start` parametri har elementni faqat o'zidan keyingilar bilan birikishini ta'minlaydi.

---

## 4. Subsets II (dublikatli)

Dublikatli massivning takror qism-to'plamsiz power set'ini qaytaramiz. Sort + skip-duplicate mantig'i.

```js
function subsetsWithDup(nums) {
  nums.sort((a, b) => a - b);            // dublikatlar yonma-yon turishi uchun
  const result = [];
  function backtrack(start, path) {
    result.push([...path]);
    for (let i = start; i < nums.length; i++) {
      if (i > start && nums[i] === nums[i - 1]) continue; // takrorni o'tkaz
      path.push(nums[i]);                // CHOOSE
      backtrack(i + 1, path);            // EXPLORE
      path.pop();                        // UNCHOOSE
    }
  }
  backtrack(0, []);
  return result;
}

console.log(subsetsWithDup([1, 2, 2]));
// [[],[1],[1,2],[1,2,2],[2],[2,2]]
```

- **Time:** O(n · 2ⁿ).
- **Space:** O(n).

**Izoh:** Avval `sort()` qilamiz, keyin `i > start && nums[i] === nums[i-1]` sharti bilan **bir xil darajada** takrorlangan elementni o'tkazib yuboramiz. `i > start` muhim — bir xil elementni boshqa darajalarda ishlatishga ruxsat beramiz (`[2,2]` to'g'ri chiqishi uchun), faqat bir darajada takrorni bloklaymiz.

---

## 5. Permutations

Unikal sonlar massivining barcha permutatsiyalarini `used[]` bilan generatsiya qilamiz.

```js
function permute(nums) {
  const result = [];
  const used = new Array(nums.length).fill(false);
  function backtrack(path) {
    if (path.length === nums.length) {   // base case: to'liq
      result.push([...path]);
      return;
    }
    for (let i = 0; i < nums.length; i++) {
      if (used[i]) continue;             // allaqachon ishlatilgan
      used[i] = true; path.push(nums[i]);     // CHOOSE
      backtrack(path);                        // EXPLORE
      path.pop(); used[i] = false;            // UNCHOOSE
    }
  }
  backtrack([]);
  return result;
}

console.log(permute([1, 2, 3]));
// [[1,2,3],[1,3,2],[2,1,3],[2,3,1],[3,1,2],[3,2,1]]
```

- **Time:** O(n · n!) — n! permutatsiya, har birini nusxalash O(n).
- **Space:** O(n) — rekursiya chuqurligi va `used`.

**Izoh:** Permutatsiyalarda **tartib muhim**, har element istalgan pozitsiyaga kelishi mumkin — shuning uchun `start` emas, `used[]` massivi bilan qaysi elementlar ishlatilganini kuzatamiz.

---

## 6. Combinations

`1..n` dan `k` ta sonni tanlashning barcha kombinatsiyalarini qaytaramiz. Pruning bilan optimallashtiramiz.

```js
function combine(n, k) {
  const result = [];
  function backtrack(start, path) {
    if (path.length === k) {             // base case: k ta tanlandi
      result.push([...path]);
      return;
    }
    // Pruning: qolgan elementlar k ni to'ldirishga yetmasa, to'xta
    const need = k - path.length;
    for (let i = start; i <= n - need + 1; i++) {
      path.push(i);                      // CHOOSE
      backtrack(i + 1, path);            // EXPLORE
      path.pop();                        // UNCHOOSE
    }
  }
  backtrack(1, []);
  return result;
}

console.log(combine(4, 2));
// [[1,2],[1,3],[1,4],[2,3],[2,4],[3,4]]
```

- **Time:** O(k · C(n, k)).
- **Space:** O(k) — rekursiya chuqurligi.

**Izoh:** Tartib muhim emas (`[1,2]` = `[2,1]`), shuning uchun `start` bilan har element faqat o'zidan keyingilar bilan birikadi. Pruning: `i <= n - need + 1` — agar qolgan elementlar `k` ni to'ldirishga yetmasa, o'sha shoxni umuman boshlamaymiz.

---

## 7. Combination Sum

`candidates` (har biri cheksiz ishlatilishi mumkin) dan yig'indisi `target` ga teng barcha kombinatsiyalarni topamiz.

```js
function combinationSum(candidates, target) {
  const result = [];
  function backtrack(start, path, remain) {
    if (remain === 0) {                  // aniq target topildi
      result.push([...path]);
      return;
    }
    if (remain < 0) return;              // oshib ketdi → pruning
    for (let i = start; i < candidates.length; i++) {
      path.push(candidates[i]);                       // CHOOSE
      backtrack(i, path, remain - candidates[i]);     // i (o'zini qayta) — takror ruxsat
      path.pop();                                     // UNCHOOSE
    }
  }
  backtrack(0, [], target);
  return result;
}

console.log(combinationSum([2, 3, 6, 7], 7));
// [[2,2,3],[7]]
```

- **Time:** O(2^target) atrofida (target va candidates'ga bog'liq eksponensial).
- **Space:** O(target / min) — rekursiya chuqurligi.

**Izoh:** Bir element cheksiz ishlatilishi mumkin bo'lgani uchun rekursiv chaqiriqda `i + 1` emas, **`i`** beramiz (o'sha elementni qayta tanlash mumkin). `start` esa tartibni belgilab, takror kombinatsiyalarni (`[2,3]` va `[3,2]`) oldini oladi.

---

## 8. Generate Parentheses

`n` juft qavsdan barcha to'g'ri (balanced) kombinatsiyalarni generatsiya qilamiz. open/close hisoblagichi bilan pruning.

```js
function generateParenthesis(n) {
  const result = [];
  function backtrack(path, open, close) {
    if (path.length === 2 * n) {         // base: to'liq
      result.push(path);
      return;
    }
    if (open < n) backtrack(path + '(', open + 1, close);     // '(' qo'sh
    if (close < open) backtrack(path + ')', open, close + 1);  // ')' qo'sh
  }
  backtrack('', 0, 0);
  return result;
}

console.log(generateParenthesis(3));
// ["((()))","(()())","(())()","()(())","()()()"]
```

- **Time:** O(4ⁿ / √n) — Catalan soni.
- **Space:** O(n) — rekursiya chuqurligi.

**Izoh:** Pruning ikki shart bilan: ochuvchi qavslar `n` dan oshmasin (`open < n`), yopuvchilar ochuvchilardan oshmasin (`close < open`). String immutable bo'lgani uchun `path + '('` yangi string yaratadi — qo'lda `pop()` (unchoose) kerak emas.

---

## 9. N-Queens

n×n taxtaga n ta vazirni hujumsiz joylashning barcha yechimlarini qaytaramiz. Diagonal tekshiruvni Set bilan O(1) qilamiz.

```js
function solveNQueens(n) {
  const result = [];
  const cols = new Set(), diag1 = new Set(), diag2 = new Set();
  const board = Array.from({ length: n }, () => new Array(n).fill('.'));

  function backtrack(row) {
    if (row === n) {                     // base: hamma qator to'ldi
      result.push(board.map(r => r.join('')));
      return;
    }
    for (let col = 0; col < n; col++) {
      if (cols.has(col) || diag1.has(row - col) || diag2.has(row + col))
        continue;                        // pruning — hujum ostida
      cols.add(col); diag1.add(row - col); diag2.add(row + col);
      board[row][col] = 'Q';             // CHOOSE
      backtrack(row + 1);                // EXPLORE
      cols.delete(col); diag1.delete(row - col); diag2.delete(row + col);
      board[row][col] = '.';             // UNCHOOSE
    }
  }

  backtrack(0);
  return result;
}

console.log(solveNQueens(4).length); // 2
console.log(solveNQueens(4));
// [[".Q..","...Q","Q...","..Q."], ["..Q.","Q...","...Q",".Q.."]]
```

- **Time:** ~O(n!) — pruning bilan amalda ancha kam.
- **Space:** O(n²) — board, plus O(n) Set'lar.

**Izoh:** Har qatorga bittadan vazir qo'yamiz. Diagonal hiylasi: bitta `↘` diagonalda `row - col` doimiy, bitta `↙` diagonalda `row + col` doimiy. Uch Set bilan O(1) da hujumni tekshiramiz.

---

## 10. Word Search

Harf gridida berilgan so'z qo'shni kataklar orqali mavjudligini DFS + backtracking bilan aniqlaymiz.

```js
function exist(board, word) {
  const rows = board.length, cols = board[0].length;

  function dfs(r, c, i) {
    if (i === word.length) return true;          // butun so'z topildi
    if (r < 0 || c < 0 || r >= rows || c >= cols ||
        board[r][c] !== word[i]) return false;   // chegara yoki mos emas

    const tmp = board[r][c];
    board[r][c] = '#';                           // CHOOSE (visited belgisi)
    const found = dfs(r + 1, c, i + 1) || dfs(r - 1, c, i + 1) ||
                  dfs(r, c + 1, i + 1) || dfs(r, c - 1, i + 1);
    board[r][c] = tmp;                           // UNCHOOSE (tiklash)
    return found;
  }

  for (let r = 0; r < rows; r++)
    for (let c = 0; c < cols; c++)
      if (dfs(r, c, 0)) return true;
  return false;
}

const board = [['A','B','C','E'],['S','F','C','S'],['A','D','E','E']];
console.log(exist(board, 'ABCCED')); // true
console.log(exist(board, 'ABCB'));   // false
```

- **Time:** O(m · n · 4^L) — L = so'z uzunligi.
- **Space:** O(L) — rekursiya chuqurligi.

**Izoh:** Katakni `'#'` bilan belgilab, bir yo'lda ikki marta ishlatilishini oldini olamiz. Qaytishda asl harfni **tiklash** shart (unchoose) — aks holda boshqa yo'llar uchun grid buziladi.

---

## 11. Letter Combinations of a Phone Number

Telefon raqamidagi (2–9) raqamlarga mos harf kombinatsiyalarining barchasini generatsiya qilamiz.

```js
function letterCombinations(digits) {
  if (!digits || digits.length === 0) return [];
  const map = {
    '2': 'abc', '3': 'def', '4': 'ghi', '5': 'jkl',
    '6': 'mno', '7': 'pqrs', '8': 'tuv', '9': 'wxyz',
  };
  const result = [];
  function backtrack(index, path) {
    if (index === digits.length) {       // base: barcha raqamlar ishlandi
      result.push(path);
      return;
    }
    const letters = map[digits[index]];
    for (const ch of letters) {
      backtrack(index + 1, path + ch);   // CHOOSE + EXPLORE (string immutable)
    }
  }
  backtrack(0, '');
  return result;
}

console.log(letterCombinations('23'));
// ["ad","ae","af","bd","be","bf","cd","ce","cf"]
```

- **Time:** O(4ⁿ · n) — n = raqamlar soni (har raqam ≤ 4 harf).
- **Space:** O(n) — rekursiya chuqurligi.

**Izoh:** Har raqam uchun mos harflar bo'yicha aylanib, `index + 1` ga o'tamiz. String immutable bo'lgani uchun `path + ch` yangi string yaratadi — `pop()` kerak emas. Bo'sh kirish uchun bo'sh array qaytaramiz (muhim edge case).

---

## 12. Palindrome Partitioning

Stringni shunday bo'laklarga ajratamizki, har bo'lak palindrom bo'lsin — barcha bunday bo'linishlarni qaytaramiz.

```js
function partition(s) {
  const result = [];

  function isPalindrome(str, lo, hi) {
    while (lo < hi) {
      if (str[lo] !== str[hi]) return false;
      lo++; hi--;
    }
    return true;
  }

  function backtrack(start, path) {
    if (start === s.length) {            // butun string bo'laklandi
      result.push([...path]);
      return;
    }
    for (let end = start; end < s.length; end++) {
      if (isPalindrome(s, start, end)) { // faqat palindrom bo'lakni olamiz
        path.push(s.slice(start, end + 1)); // CHOOSE
        backtrack(end + 1, path);           // EXPLORE
        path.pop();                         // UNCHOOSE
      }
    }
  }

  backtrack(0, []);
  return result;
}

console.log(partition('aab'));
// [["a","a","b"],["aa","b"]]
```

- **Time:** O(n · 2ⁿ) — 2ⁿ bo'linish, har palindrom tekshiruvi O(n).
- **Space:** O(n) — rekursiya chuqurligi.

**Izoh:** Har pozitsiyadan boshlab, mumkin bo'lgan har bo'lakni (`start..end`) tekshiramiz; agar palindrom bo'lsa, uni olib qolganini rekursiv bo'laklaymiz. Pruning — faqat palindrom bo'laklarni o'rganamiz, qolgan shoxlarni darhol kesamiz.

---

← [DSA bo'limiga qaytish](../../dsa/README.md)
