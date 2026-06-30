# Recursion va Backtracking

Recursion (rekursiya) — masalani **o'ziga o'xshash kichikroq qism-masalaga** bo'lib yechish usuli, backtracking esa rekursiyaning eng kuchli qo'llanilishi: barcha mumkin variantlarni tizimli ko'rib chiqib, yaroqsizlarini "orqaga qaytib" (backtrack) tashlash. Bu ikkisi suhbatlarda alohida ahamiyatga ega — tree, graph, DP, permutation/combination masalalarining hammasi shu fikrlash uslubiga tayanadi. Bu hujjat base case'dan tortib, call stack, recursion tree, choose-explore-unchoose shabloni va klassik backtracking masalalarining (permutations, subsets, N-Queens, sudoku) to'liq mantig'igacha boradi.

## Mundarija

- [Recursion nima (base case va recursive case)](#recursion-nima-base-case-va-recursive-case)
- [Call stack va stack overflow](#call-stack-va-stack-overflow)
- [Recursion vs Iteration](#recursion-vs-iteration)
- [Recursive thinking — masalani bo'lish](#recursive-thinking--masalani-bolish)
- [Recursion tree va complexity](#recursion-tree-va-complexity)
- [Tail recursion](#tail-recursion)
- [Backtracking nima (choose-explore-unchoose)](#backtracking-nima-choose-explore-unchoose)
- [Backtracking template](#backtracking-template)
- [Subsets](#subsets)
- [Permutations](#permutations)
- [Combinations](#combinations)
- [N-Queens](#n-queens)
- [Generate Parentheses](#generate-parentheses)
- [Word Search](#word-search)
- [Sudoku Solver](#sudoku-solver)
- [Intervyu Q&A](#intervyu-qa)
- [Masalalar](#masalalar)

---

## Recursion nima (base case va recursive case)

**💡 Tushuncha:** Rekursiya — funksiyaning o'zini chaqirishi. Har rekursiv funksiyada ikki qism bo'lishi shart:
- **Base case** — rekursiya to'xtaydigan eng oddiy holat (chaqiriqsiz javob).
- **Recursive case** — masalani kichikroq qilib, o'zini qayta chaqirish.

```js
function factorial(n) {
  if (n <= 1) return 1;           // base case
  return n * factorial(n - 1);    // recursive case
}
factorial(4); // 4 * 3 * 2 * 1 = 24
```

**⚠️ Ehtiyot bo'l:** Base case bo'lmasa yoki rekursiya unga **yaqinlashmasa** — cheksiz rekursiya va stack overflow bo'ladi. Har rekursiv chaqiriq muammoni base case tomon "kichraytirishi" kerak (`n - 1`, massivning yarmi va h.k.).

---

## Call stack va stack overflow

**💡 Tushuncha:** Har funksiya chaqirilganda JS uning ma'lumotini (argumentlar, lokal o'zgaruvchilar, qaytish manzili) **call stack** ga "frame" sifatida qo'yadi. Rekursiv chaqiriq esa stack'ga frame ustiga frame qo'yadi. Base case'ga yetganda stack teskari tartibda "bo'shaydi" (unwind).

```js
// factorial(3) ning stack holati:
// factorial(3) → factorial(2) → factorial(1)=1
//   ↑ qaytadi: 1*2=2 ← 2*3=6 ← natija
```

**⚠️ Ehtiyot bo'l:** Stack chegarali (V8'da ~10⁴–10⁵ frame). Juda chuqur rekursiya `RangeError: Maximum call stack size exceeded` beradi. Chuqur rekursiya kerak bo'lsa (masalan 10⁶ element) — iteratsiya yoki **aniq stack** (explicit stack with array) ishlatib, call stack'ni o'z stack'ingiz bilan almashtiring.

```js
// Chuqur rekursiyani manual stack bilan qayta yozish
function sumIterative(n) {
  let sum = 0;
  while (n > 0) { sum += n; n--; }   // stack overflow yo'q
  return sum;
}
```

---

## Recursion vs Iteration

**💡 Tushuncha:** Har rekursiyani tsikl bilan yozish mumkin va aksincha. Tanlov o'qilishi va masala tabiatiga bog'liq.

| Mezon | Recursion | Iteration |
|---|---|---|
| Xotira | O(chuqurlik) stack | O(1) (odatda) |
| Tezlik | sekinroq (call overhead) | tezroq |
| O'qilishi | tree/graph/divide masalalarda **toza** | linear masalalarda toza |
| Stack overflow | xavf bor | yo'q |

**✅ Qachon rekursiya:** tree traversal, divide-and-conquer (merge sort), backtracking, graph DFS — bularda rekursiya tabiiy va o'qilishi oson.

**✅ Qachon iteratsiya:** oddiy linear hisob-kitob, chuqurlik katta bo'lishi mumkin bo'lgan holatlar, performance kritik joylar.

**⚠️ Ehtiyot bo'l:** "Rekursiya har doim toza" — noto'g'ri. Oddiy tsikl bilan yechiladigan narsani rekursiya bilan yozish faqat overhead va overflow xavfini qo'shadi. Rekursiya muammoning tuzilishi rekursiv bo'lganda foydali.

---

## Recursive thinking — masalani bo'lish

**💡 Tushuncha:** Rekursiv yechimda 3 savol berasiz:
1. **Base case** nima? (eng kichik, javob ma'lum holat)
2. Masalani qanday **kichikroq** qism-masalaga bo'lasiz?
3. Qism-masala yechimidan **butun yechimni** qanday yig'asiz?

Misol — massiv yig'indisi:

```js
function sum(arr, i = 0) {
  if (i === arr.length) return 0;          // 1) base case
  return arr[i] + sum(arr, i + 1);         // 2,3) bittasi + qolgani
}
```

**💡 Tushuncha:** "Leap of faith" — rekursiv chaqiriq **to'g'ri ishlaydi deb ishonib**, faqat hozirgi qadamni to'g'ri yozasiz. `sum(arr, i+1)` qolgan elementlar yig'indisini beradi deb ishonasiz, siz faqat `arr[i]` ni qo'shasiz. Bu rekursiv fikrlashning kaliti — ichkariga "tushib" ketmaslik.

---

## Recursion tree va complexity

**💡 Tushuncha:** Rekursiya complexity'sini hisoblash uchun **recursion tree** chizamiz: har tugun bitta chaqiriq, har darajada nechta chaqiriq va har biri qancha ish bajarishini sanaymiz.

```js
// Fibonacci — har chaqiriq 2 ta chaqiriq tug'diradi
function fib(n) {
  if (n < 2) return n;
  return fib(n - 1) + fib(n - 2);
}
// Recursion tree: 2^n ga yaqin tugun → O(2^n) vaqt!
// fib(5) → fib(4)+fib(3) → ... bir xil qiymatlar qayta hisoblanadi
```

Complexity formulasi: **(tugunlar soni) × (har tugun ishi)**.
- Branching factor `b`, chuqurlik `d` → O(bᵈ) tugun.
- Fibonacci: b=2, d=n → O(2ⁿ).
- Merge sort: har darajada O(n) ish, log n daraja → O(n log n).

**⚠️ Ehtiyot bo'l:** Naive Fibonacci O(2ⁿ) — `fib(50)` deyarli abadiy ishlaydi, chunki bir qiymat ko'p marta qayta hisoblanadi (overlapping subproblems). **Memoization** bilan O(n) ga tushadi — bu Dynamic Programming'ga ko'prik.

```js
function fibMemo(n, memo = {}) {
  if (n < 2) return n;
  if (memo[n] !== undefined) return memo[n];
  return memo[n] = fibMemo(n - 1, memo) + fibMemo(n - 2, memo);
}
// O(n) vaqt, O(n) xotira
```

---

## Tail recursion

**💡 Tushuncha:** **Tail recursion** — rekursiv chaqiriq funksiyaning **eng oxirgi** amali bo'lganda (qaytgandan keyin hech narsa qilinmaydi). Bunday holatda engine teoretik jihatdan stack frame'ni qayta ishlatishi (tail-call optimization, TCO) va overflow'ni oldini olishi mumkin.

```js
// ❌ Tail emas: qaytgandan keyin "n *" qiladi
function fact(n) { return n <= 1 ? 1 : n * fact(n - 1); }

// ✅ Tail: chaqiriq oxirgi amal, akkumulyator bilan
function factTail(n, acc = 1) {
  if (n <= 1) return acc;
  return factTail(n - 1, n * acc);  // oxirgi amal — sof chaqiriq
}
```

**⚠️ Ehtiyot bo'l:** TCO ES2015 spesifikatsiyasida bor, lekin **V8 (Node, Chrome) uni qo'llab-quvvatlamaydi** — faqat Safari/JavaScriptCore'da. Demak JS'da tail recursion'ga overflow'dan himoya sifatida tayanmang. Chuqur rekursiya kerak bo'lsa iteratsiyaga aylantiring.

---

## Backtracking nima (choose-explore-unchoose)

**💡 Tushuncha:** Backtracking — barcha mumkin variantlarni qurish jarayonida, hozirgi tanlov yaroqsiz bo'lsa **orqaga qaytib** (tanlovni bekor qilib) boshqasini sinash. U "qaror daraxti"ni (decision tree) DFS bilan kezadi, lekin foydasiz shoxlarni **kesib** (pruning) tashlaydi.

Uch qadamli shablon:
1. **Choose** — bir variant tanlash (yo'lga qo'shish).
2. **Explore** — shu tanlov bilan rekursiv davom etish.
3. **Unchoose** — tanlovni bekor qilish (yo'ldan olib tashlash), keyingisini sinash uchun.

```js
// Konseptual ko'rinish
function backtrack(path, choices) {
  if (isSolution(path)) { result.push([...path]); return; }
  for (const choice of choices) {
    if (!isValid(choice, path)) continue;  // pruning
    path.push(choice);                     // CHOOSE
    backtrack(path, nextChoices(choice));  // EXPLORE
    path.pop();                            // UNCHOOSE (backtrack)
  }
}
```

**⚠️ Ehtiyot bo'l:** `result.push([...path])` da **nusxa** (`[...path]`) shart! `path` ni to'g'ridan-to'g'ri push qilsangiz, keyin `path.pop()` uni o'zgartiradi va natijadagi barcha massivlar buziladi (reference muammosi).

---

## Backtracking template

**💡 Tushuncha:** Aksariyat backtracking masalalari bitta umumiy skeletga to'g'ri keladi. Faqat 3 narsa o'zgaradi: **qachon yechim tayyor**, **qaysi variantlar bor**, **qaysi variant yaroqli (pruning)**.

```js
function solve(input) {
  const result = [];
  function backtrack(path, start /* yoki used, state */) {
    // 1) To'xtash sharti — yechim tayyor
    if (/* yechim sharti */) {
      result.push([...path]);
      return;                          // ko'pincha return qilamiz
    }
    // 2) Variantlarni ko'rib chiqish
    for (let i = start; i < input.length; i++) {
      // 3) Pruning — yaroqsiz variantni o'tkazib yuborish
      if (/* yaroqsiz */) continue;
      path.push(input[i]);             // CHOOSE
      backtrack(path, i + 1);          // EXPLORE
      path.pop();                      // UNCHOOSE
    }
  }
  backtrack([], 0);
  return result;
}
```

**💡 Tushuncha:** `start` parametri — kombinatsiyalarda **takrorlanishni** oldini oladi (har element faqat o'zidan keyingilar bilan birikadi). Permutatsiyalarda esa `start` o'rniga `used[]` massivi ishlatiladi (har element istalgan pozitsiyaga kelishi mumkin).

---

## Subsets

**💡 Tushuncha:** Massivning barcha qism-to'plamlarini (power set) topish. Har element uchun ikki tanlov: **olish** yoki **olmaslik** — 2ⁿ ta qism-to'plam.

```js
function subsets(nums) {
  const result = [];
  function backtrack(start, path) {
    result.push([...path]);            // har holat — yaroqli qism-to'plam
    for (let i = start; i < nums.length; i++) {
      path.push(nums[i]);              // CHOOSE
      backtrack(i + 1, path);          // EXPLORE (i+1 → takror yo'q)
      path.pop();                      // UNCHOOSE
    }
  }
  backtrack(0, []);
  return result;
}
// subsets([1,2,3]) → [[],[1],[1,2],[1,2,3],[1,3],[2],[2,3],[3]]
// Vaqt: O(n · 2^n). Xotira: O(n) (recursion depth).
```

**⚠️ Ehtiyot bo'l:** Subsets'da yechim **har tugunda** push qilinadi (alohida base case yo'q) — har oraliq holat ham yaroqli qism-to'plam. Permutation/combination'dan farqi shu.

---

## Permutations

**💡 Tushuncha:** Elementlarning barcha tartiblanishlari (n! ta). Bu yerda **tartib muhim**, har element istalgan pozitsiyaga kelishi mumkin — shuning uchun `start` emas, `used[]` massivi bilan ishlatilganini kuzatamiz.

```js
function permute(nums) {
  const result = [];
  const used = new Array(nums.length).fill(false);
  function backtrack(path) {
    if (path.length === nums.length) {  // base case: to'liq
      result.push([...path]);
      return;
    }
    for (let i = 0; i < nums.length; i++) {
      if (used[i]) continue;            // allaqachon ishlatilgan
      used[i] = true; path.push(nums[i]);     // CHOOSE
      backtrack(path);                        // EXPLORE
      path.pop(); used[i] = false;            // UNCHOOSE
    }
  }
  backtrack([]);
  return result;
}
// Vaqt: O(n · n!). Xotira: O(n).
```

**⚠️ Ehtiyot bo'l:** Dublikatli massivda (`[1,1,2]`) takror permutatsiyalar chiqmasligi uchun avval `sort()` qiling, keyin "agar oldingi bir xil element ishlatilmagan bo'lsa, bu elementni o'tkaz" shartini qo'shing (`if (i > 0 && nums[i] === nums[i-1] && !used[i-1]) continue`).

---

## Combinations

**💡 Tushuncha:** n ta elementdan k tasini tanlash — bu yerda **tartib muhim emas** (`[1,2]` va `[2,1]` bir xil). `start` parametri bilan har element faqat o'zidan keyingilar bilan birikadi.

```js
function combine(n, k) {
  const result = [];
  function backtrack(start, path) {
    if (path.length === k) {            // base case: k ta tanlandi
      result.push([...path]);
      return;
    }
    for (let i = start; i <= n; i++) {
      path.push(i);                     // CHOOSE
      backtrack(i + 1, path);           // EXPLORE (i+1 → takror yo'q)
      path.pop();                       // UNCHOOSE
    }
  }
  backtrack(1, []);
  return result;
}
// combine(4, 2) → [[1,2],[1,3],[1,4],[2,3],[2,4],[3,4]]
// Vaqt: O(k · C(n,k)).
```

**💡 Tushuncha:** Pruning bilan tezlashtirish mumkin: agar qolgan elementlar k ni to'ldirishga yetmasa, tsikldan chiqish — `for (let i = start; i <= n - (k - path.length) + 1; i++)`.

---

## N-Queens

**💡 Tushuncha:** n×n shaxmat taxtasiga n ta vazirni bir-biriga hujum qilmaydigan qilib joylash. Har qatorga bittadan vazir qo'yamiz; har ustun va diagonal band-bandligini kuzatamiz. Yaroqsiz joyni darhol kesib tashlaymiz (pruning).

```js
function solveNQueens(n) {
  const result = [];
  const cols = new Set(), diag1 = new Set(), diag2 = new Set();
  const board = Array.from({ length: n }, () => new Array(n).fill('.'));
  function backtrack(row) {
    if (row === n) {                    // base: hamma qator to'ldi
      result.push(board.map(r => r.join('')));
      return;
    }
    for (let col = 0; col < n; col++) {
      // diagonal kalitlari: row-col va row+col unikal
      if (cols.has(col) || diag1.has(row - col) || diag2.has(row + col))
        continue;                        // pruning — hujum ostida
      cols.add(col); diag1.add(row - col); diag2.add(row + col);
      board[row][col] = 'Q';            // CHOOSE
      backtrack(row + 1);               // EXPLORE
      cols.delete(col); diag1.delete(row - col); diag2.delete(row + col);
      board[row][col] = '.';            // UNCHOOSE
    }
  }
  backtrack(0);
  return result;
}
// Vaqt: ~O(n!) (pruning bilan ancha kam).
```

**💡 Tushuncha:** Diagonal aniqlash hiylasi: bitta `↘` diagonalda `row - col` doimiy, bitta `↙` diagonalda `row + col` doimiy. Set'lar bilan O(1) da tekshirib, butun qator/ustun aylanishidan qutulamiz.

---

## Generate Parentheses

**💡 Tushuncha:** n juft qavsdan barcha **to'g'ri** (balanced) kombinatsiyalarni yaratish. Pruning sharti: ochuvchi qavslar soni n dan oshmasin, yopuvchi qavslar ochuvchilardan oshmasin.

```js
function generateParenthesis(n) {
  const result = [];
  function backtrack(path, open, close) {
    if (path.length === 2 * n) {        // base: to'liq
      result.push(path);
      return;
    }
    if (open < n) backtrack(path + '(', open + 1, close);   // '(' qo'sh
    if (close < open) backtrack(path + ')', open, close + 1); // ')' qo'sh
  }
  backtrack('', 0, 0);
  return result;
}
// generateParenthesis(2) → ["(())","()()"]
// Vaqt: O(4^n / √n) (Catalan number).
```

**⚠️ Ehtiyot bo'l:** Bu yerda string immutable bo'lgani uchun `path + '('` yangi string yaratadi — qo'lda `pop()` (unchoose) kerak emas. Massiv ishlatsangiz `push`/`pop` qilishingiz kerak edi.

---

## Word Search

**💡 Tushuncha:** 2D harf gridida berilgan so'zni qo'shni kataklar (4 yo'nalish) orqali topish mumkinmi? Har katakdan DFS boshlab, mos kelsa qo'shnilarga o'tamiz, qaytishda katakni "tashrif buyurilgan" belgisidan tozalaymiz (backtrack).

```js
function exist(board, word) {
  const rows = board.length, cols = board[0].length;
  function dfs(r, c, i) {
    if (i === word.length) return true;          // butun so'z topildi
    if (r < 0 || c < 0 || r >= rows || c >= cols ||
        board[r][c] !== word[i]) return false;   // chegara/mos emas
    const tmp = board[r][c];
    board[r][c] = '#';                           // CHOOSE (visited belgisi)
    const found = dfs(r+1, c, i+1) || dfs(r-1, c, i+1) ||
                  dfs(r, c+1, i+1) || dfs(r, c-1, i+1);
    board[r][c] = tmp;                           // UNCHOOSE (tiklash)
    return found;
  }
  for (let r = 0; r < rows; r++)
    for (let c = 0; c < cols; c++)
      if (dfs(r, c, 0)) return true;
  return false;
}
// Vaqt: O(m·n·4^L), L = so'z uzunligi.
```

**⚠️ Ehtiyot bo'l:** Katakni `'#'` bilan belgilab, qaytishda asl harfni **tiklash** shart — aks holda bir katak ikki marta ishlatilishi yoki keyingi qidiruv buzilishi mumkin. Bu backtracking'ning "unchoose" qadami.

---

## Sudoku Solver

**💡 Tushuncha:** Bo'sh kataklarni tartib bilan to'ldiramiz. Har katakka 1–9 raqamlarini sinab ko'ramiz; agar raqam qator/ustun/3×3 blokda takrorlanmasa — qo'yamiz va davom etamiz. Agar boshi berk ko'chaga kirsa, orqaga qaytib boshqa raqam sinaymiz.

```js
function solveSudoku(board) {
  function isValid(r, c, ch) {
    for (let i = 0; i < 9; i++) {
      if (board[r][i] === ch) return false;       // qator
      if (board[i][c] === ch) return false;       // ustun
      const br = 3 * Math.floor(r/3) + Math.floor(i/3);
      const bc = 3 * Math.floor(c/3) + (i % 3);
      if (board[br][bc] === ch) return false;      // 3x3 blok
    }
    return true;
  }
  function backtrack() {
    for (let r = 0; r < 9; r++)
      for (let c = 0; c < 9; c++)
        if (board[r][c] === '.') {
          for (let d = 1; d <= 9; d++) {
            const ch = String(d);
            if (isValid(r, c, ch)) {
              board[r][c] = ch;                    // CHOOSE
              if (backtrack()) return true;        // EXPLORE
              board[r][c] = '.';                   // UNCHOOSE
            }
          }
          return false;                            // hech biri ishlamadi
        }
    return true;                                   // bo'sh katak yo'q — yechildi
  }
  backtrack();
  return board;
}
```

**💡 Tushuncha:** Bu yerda backtracking **bitta** yechim topishi bilan to'xtaydi (`if (backtrack()) return true`). Permutations/subsets'da esa hamma yechimni yig'amiz va `return` qilmay davom etamiz. Bu ikki rejimni farqlash muhim.

---

## Intervyu Q&A

### ❓ Rekursiyada base case nima va nega zarur?

**✅ Javob:** Base case — rekursiya to'xtaydigan, javob to'g'ridan-to'g'ri ma'lum bo'lgan eng oddiy holat (masalan `factorial`da `n <= 1`). U zarur, chunki base case bo'lmasa yoki rekursiya unga yaqinlashmasa, funksiya o'zini cheksiz chaqirib stack overflow beradi. Har rekursiv chaqiriq muammoni base case tomon kichraytirishi shart.

### ❓ Call stack nima va stack overflow qachon yuz beradi?

**✅ Javob:** Call stack — chaqirilgan funksiyalarning frame'larini (argumentlar, lokal o'zgaruvchilar, qaytish manzili) saqlovchi xotira tuzilmasi. Har rekursiv chaqiriq yangi frame qo'yadi. Stack chegarali (V8'da ~10⁴–10⁵ frame), shuning uchun juda chuqur yoki cheksiz rekursiya `Maximum call stack size exceeded` xatosini beradi. Yechim — iteratsiya yoki o'z (explicit) stack'ingiz.

### ❓ Rekursiya va iteratsiyani qachon tanlaysiz?

**✅ Javob:** Rekursiya — muammo tuzilishi rekursiv bo'lganda (tree traversal, divide-and-conquer, backtracking, graph DFS) — kodni tabiiy va o'qilishi oson qiladi. Iteratsiya — oddiy linear hisob, chuqurlik katta bo'lishi mumkin bo'lgan yoki performance kritik holatlarda — O(1) xotira va overflow xavfsizligini beradi. Har rekursiyani iteratsiyaga aylantirish mumkin, lekin har doim arzimaydi.

### ❓ "Recursive thinking" ni qanday tushuntirasiz?

**✅ Javob:** Masalani uch savol bilan bo'laman: base case nima, masalani qanday kichikroq qism-masalaga bo'laman, qism yechimidan butunni qanday yig'aman. "Leap of faith" — rekursiv chaqiriq to'g'ri ishlaydi deb ishonib, faqat hozirgi qadamni to'g'ri yozaman. Masalan `sum(arr, i+1)` qolganlar yig'indisini beradi deb ishonib, faqat `arr[i]` ni qo'shaman — ichkariga tushib ketmayman.

### ❓ Naive Fibonacci nega O(2ⁿ) va qanday tezlashtirasiz?

**✅ Javob:** Naive `fib(n) = fib(n-1) + fib(n-2)` har chaqiriqda 2 ta chaqiriq tug'diradi, recursion tree ~2ⁿ tugunli. Bir xil qiymatlar (overlapping subproblems) ko'p marta qayta hisoblanadi. **Memoization** (hisoblangan natijalarni cache'lash) bilan har qiymat bir marta hisoblanadi → O(n) vaqt, O(n) xotira. Bu Dynamic Programming'ga o'tish nuqtasi.

### ❓ Recursion tree bilan complexity'ni qanday hisoblaysiz?

**✅ Javob:** Recursion tree chizib, (tugunlar soni) × (har tugun bajaradigan ish) ni hisoblayman. Branching factor b va chuqurlik d bo'lsa O(bᵈ) tugun. Fibonacci: b=2, d=n → O(2ⁿ). Merge sort: har darajada O(n) ish, log n daraja → O(n log n). Xotira esa eng chuqur yo'l (rekursiya chuqurligi) bilan o'lchanadi.

### ❓ Tail recursion nima va JS'da unga tayanish mumkinmi?

**✅ Javob:** Tail recursion — rekursiv chaqiriq funksiyaning eng oxirgi amali (qaytgandan keyin hech narsa qilinmaydi), akkumulyator parametri bilan yoziladi. Teoretik jihatdan engine stack frame'ni qayta ishlatib (TCO) overflow'ni oldini olishi mumkin. Lekin **V8 (Node/Chrome) TCO'ni qo'llab-quvvatlamaydi** — faqat Safari. Demak JS'da tail recursion'ga overflow himoyasi sifatida tayanmang; chuqur rekursiyani iteratsiyaga aylantiring.

### ❓ Backtracking nima va u DFS'dan qanday farq qiladi?

**✅ Javob:** Backtracking — barcha mumkin variantlarni qaror daraxti bo'ylab qurib, yaroqsiz bo'lsa orqaga qaytib boshqasini sinash. U aslida qaror daraxtidagi **DFS**, lekin ikki qo'shimcha bilan: (1) **pruning** — foydasiz shoxlarni darhol kesish, (2) **state'ni tiklash** (unchoose) — orqaga qaytishda qilingan o'zgarishni bekor qilish. Oddiy DFS graph kezadi, backtracking esa yechimlar fazosini quradi.

### ❓ "choose-explore-unchoose" shablonini tushuntiring.

**✅ Javob:** Uch qadam: **choose** — bir variant tanlab yo'lga (path) qo'shaman; **explore** — shu tanlov bilan rekursiv davom etaman; **unchoose** — tanlovni yo'ldan olib tashlab (`path.pop()`), keyingi variant uchun state'ni tozalayman. Unchoose eng muhimi — usiz oldingi tanlov keyingi shoxni buzadi. Natijani saqlaganda `[...path]` nusxasini olaman, aks holda keyingi pop natijani buzadi.

### ❓ Subsets, permutations, combinations backtracking'da nimasi bilan farq qiladi?

**✅ Javob:** **Subsets** — yechim har tugunda push qilinadi (har oraliq holat yaroqli), `start` bilan takror oldini olinadi, 2ⁿ ta. **Combinations** — k ta element tanlangach push, tartib muhim emas, `start` ishlatiladi (i+1). **Permutations** — to'liq uzunlikda push, tartib muhim, `start` emas `used[]` ishlatiladi (har element istalgan pozitsiyada). Asosiy farq — `start` (tartib muhimmas) vs `used[]` (tartib muhim).

### ❓ Natijaga `path` ni push qilganda nega nusxa olasiz?

**✅ Javob:** `path` — bitta o'zgaruvchan (mutable) massiv, butun rekursiya davomida `push`/`pop` bilan o'zgarib turadi. Agar `result.push(path)` qilsam, hamma element bir xil reference'ga ishora qiladi va keyingi `pop`'lar ularni buzadi. `result.push([...path])` esa o'sha lahzadagi holatning mustaqil nusxasini saqlaydi — bu juda tez-tez uchraydigan xato.

### ❓ N-Queens'da diagonal hujumni qanday tez tekshirasiz?

**✅ Javob:** Har `↘` diagonalda `row - col` doimiy, har `↙` diagonalda `row + col` doimiy. Shu ikki qiymatni ikki `Set`da (plus ustunlar uchun bitta Set) saqlab, O(1) da hujum bor-yo'qligini tekshiraman — butun qator/ustun/diagonalni aylanishga hojat qolmaydi. Har qatorga bittadan vazir qo'yib, `row+1` ga o'taman, yaroqsiz pozitsiyani darhol kesaman (pruning).

### ❓ Word Search'da nega katakni belgilab, keyin tiklaysiz?

**✅ Javob:** DFS so'z harflarini qo'shni kataklarda izlaydi. Bir katak bir yo'lda ikki marta ishlatilmasligi uchun tashrif buyurganda uni `'#'` bilan belgilayman (choose). Lekin DFS qaytib boshqa yo'lni sinaganda o'sha katak yana ishlatilishi mumkin — shuning uchun qaytishda asl harfni **tiklayman** (unchoose). Tiklamasam grid asta-sekin buzilib, keyingi qidiruvlar noto'g'ri bo'ladi.

### ❓ Backtracking qachon to'xtaydi — hamma yechimni yig'ganda yoki bittasini topganda?

**✅ Javob:** Masalaga bog'liq. **Hamma** yechim kerak bo'lsa (subsets, permutations, N-Queens hammasi), yechim topilganda `result`'ga qo'shib, `return`'siz davom etib qolgan shoxlarni ham kezaman. **Bitta** yechim yetarli bo'lsa (sudoku, word search "mumkinmi"), topilishi bilan `return true` qaytarib butun rekursiyani to'xtataman. Bu ikki rejimni ajratish kodning to'g'riligi uchun muhim.

### ❓ Backtracking'ning vaqt complexity'sini qanday baholaysiz?

**✅ Javob:** Yechimlar fazosi hajmiga qarab: permutations O(n·n!), subsets O(n·2ⁿ), combinations O(k·C(n,k)). Umumiy qoida — (yechimlar/tugunlar soni) × (har yechimni qurish/nusxalash xarajati). Pruning amalda buni ancha kamaytiradi, lekin worst case baholashda barcha shoxlar hisobga olinadi. Backtracking eksponensial bo'lgani uchun faqat kichik kirishlarda (n ≤ ~15–20) amaliy.

---

## Masalalar

> Yechimlar: [solutions/dsa/10-recursion-backtracking.md](../solutions/dsa/10-recursion-backtracking.md)

1. **Power funksiya.** `pow(x, n)` ni rekursiv yozing, O(log n) bo'lsin (fast exponentiation: `x^n = (x^(n/2))^2`). Manfiy `n` ni ham qo'llab-quvvatlang.

2. **String teskari aylantirish.** Berilgan stringni rekursiya bilan teskari aylantiring (tsiklsiz). Base case va recursive case'ni aniq ko'rsating.

3. **Subsets.** Berilgan unikal sonlar massivining barcha qism-to'plamlarini (power set) backtracking bilan qaytaring. Complexity'ni izohlang.

4. **Subsets II (dublikatli).** Dublikatli massivning takror qism-to'plamsiz power set'ini qaytaring. Sort + skip-duplicate mantig'ini qo'llang.

5. **Permutations.** Unikal sonlar massivining barcha permutatsiyalarini `used[]` bilan generatsiya qiling. Complexity'ni izohlang.

6. **Combinations.** `1..n` dan `k` ta sonni tanlashning barcha kombinatsiyalarini qaytaring. Pruning bilan optimallashtiring.

7. **Combination Sum.** Berilgan `candidates` (har biri cheksiz ishlatilishi mumkin) dan yig'indisi `target` ga teng bo'lgan barcha kombinatsiyalarni toping.

8. **Generate Parentheses.** `n` juft qavsdan barcha to'g'ri (balanced) kombinatsiyalarni generatsiya qiling. open/close hisoblagichi bilan pruning qiling.

9. **N-Queens.** n×n taxtaga n ta vazirni hujumsiz joylashning barcha yechimlarini qaytaring. Diagonal tekshiruvni Set bilan O(1) qiling.

10. **Word Search.** Harf gridida berilgan so'z qo'shni kataklar orqali mavjudligini DFS + backtracking bilan aniqlang. Visited belgilash va tiklashni to'g'ri qiling.

11. **Letter Combinations of a Phone Number.** Telefon raqamidagi (2–9) raqamlarga mos harf kombinatsiyalarining barchasini generatsiya qiling.

12. **Palindrome Partitioning.** Stringni shunday bo'laklarga ajratingki, har bo'lak palindrom bo'lsin — barcha bunday bo'linishlarni qaytaring.

← [DSA bo'limiga qaytish](./README.md)
