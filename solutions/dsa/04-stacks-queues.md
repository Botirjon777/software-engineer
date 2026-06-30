# Stacks va Queues — Yechimlar

Har masala uchun to'liq ishlaydigan kod, time/space complexity va o'zbekcha izoh. Kod JavaScript/TypeScript'da.

---

## 1. Valid Parentheses

`()[]{}` qavslari to'g'ri ochilib-yopilganini stack bilan tekshiring. (LeetCode 20)

```js
function isValid(s) {
  const stack = [];
  const pairs = { ')': '(', ']': '[', '}': '{' };
  for (const ch of s) {
    if (ch === '(' || ch === '[' || ch === '{') {
      stack.push(ch);              // ochuvchi — stack'ga
    } else {
      if (stack.pop() !== pairs[ch]) return false; // mos ochuvchi emas
    }
  }
  return stack.length === 0;        // barcha ochilganlar yopilganmi?
}

// isValid("()[]{}") => true
// isValid("(]")     => false
// isValid("([)]")   => false
```

**Complexity:** Time `O(n)`, Space `O(n)`.

**Izoh:** Eng oxirgi ochilgan qavs eng birinchi yopilishi kerak — bu aniq LIFO, demak stack. Ochuvchi qavslarni stack'ga qo'yamiz; yopuvchi kelganda stack tepasidagi qavs mos ochuvchi bo'lishi shart. Bo'sh stack'dan `pop()` `undefined` qaytaradi va tekshiruvdan o'tmaydi. Oxirida stack bo'sh bo'lsa — barcha qavs juftlangan.

---

## 2. Min Stack

`push`, `pop`, `top` va `getMin` operatsiyalarining barchasi O(1) bo'lgan stack quring. (LeetCode 155)

```js
class MinStack {
  constructor() {
    this.stack = [];
    this.minStack = []; // har bosqichdagi minimum
  }
  push(val) {
    this.stack.push(val);
    const min = this.minStack.length
      ? Math.min(val, this.minStack[this.minStack.length - 1])
      : val;
    this.minStack.push(min);
  }
  pop() {
    this.minStack.pop();
    return this.stack.pop();
  }
  top() { return this.stack[this.stack.length - 1]; }
  getMin() { return this.minStack[this.minStack.length - 1]; }
}

// const ms = new MinStack();
// ms.push(-2); ms.push(0); ms.push(-3);
// ms.getMin(); // -3
// ms.pop(); ms.top();   // 0
// ms.getMin();          // -2
```

**Complexity:** har operatsiya `O(1)`, Space `O(n)`.

**Izoh:** Asosiy stack bilan parallel `minStack` yuritamiz. Har push'da o'sha paytdagi minimumni (`min(yangi, oldingi min)`) saqlaymiz. `pop` da ikkala stack sinxron qisqaradi, shuning uchun `minStack` tepasi doim joriy minimum. Bu yondashuv minimumni qayta hisoblamasdan `getMin` ni `O(1)` qiladi.

---

## 3. Implement Queue using Stacks

Faqat ikki stack ishlatib queue quring. Amortized O(1). (LeetCode 232)

```js
class MyQueue {
  constructor() {
    this.inStack = [];
    this.outStack = [];
  }
  push(val) { this.inStack.push(val); }       // O(1)
  _transfer() {
    if (this.outStack.length === 0) {
      while (this.inStack.length) this.outStack.push(this.inStack.pop());
    }
  }
  pop() {  this._transfer(); return this.outStack.pop(); }   // amortized O(1)
  peek() { this._transfer(); return this.outStack[this.outStack.length - 1]; }
  empty() { return this.inStack.length === 0 && this.outStack.length === 0; }
}

// const q = new MyQueue();
// q.push(1); q.push(2);
// q.peek(); // 1
// q.pop();  // 1
// q.empty();// false
```

**Complexity:** `push` `O(1)`, `pop`/`peek` amortized `O(1)`, Space `O(n)`.

**Izoh:** `inStack`ga enqueue qilamiz, `outStack`dan dequeue qilamiz. `outStack` bo'shaganda `inStack`ni unga "ag'daramiz" — tartib teskarilanib FIFO hosil bo'ladi. Har element ko'pi bilan bir marta ko'chiriladi (in → out), shuning uchun amortized `O(1)`. `outStack` bo'sh bo'lmaguncha qayta ag'darmaymiz — bu muhim.

---

## 4. Implement Stack using Queues

Queue(lar) ishlatib stack quring. Push yoki pop'dan birini O(n) qilishingiz mumkin. (LeetCode 225)

```js
class MyStack {
  constructor() {
    this.queue = []; // oddiy array, lekin faqat push/shift (FIFO) ishlatamiz
  }
  push(val) {
    this.queue.push(val);
    // yangi elementni boshga olib chiqamiz: qolganlarini aylantirib orqasiga
    for (let i = 0; i < this.queue.length - 1; i++) {
      this.queue.push(this.queue.shift());
    }
  }
  pop() { return this.queue.shift(); } // O(1) — eng yangi element oldinda
  top() { return this.queue[0]; }
  empty() { return this.queue.length === 0; }
}

// const st = new MyStack();
// st.push(1); st.push(2);
// st.top(); // 2
// st.pop(); // 2
// st.empty();// false
```

**Complexity:** `push` `O(n)`, `pop`/`top` `O(1)`, Space `O(n)`.

**Izoh:** Bitta queue bilan. `push`da yangi elementni qo'shamiz, so'ng undan oldingi barcha elementlarni navbat orqasiga aylantirib o'tkazamiz — natijada eng yangi element doim **oldinda** (queue boshida) turadi, ya'ni LIFO simulyatsiya qilinadi. Shu sababli `pop`/`top` `O(1)`, evaziga `push` `O(n)`. Trade-off: kerak bo'lsa aksincha (`push` O(1), `pop` O(n)) ham qilsa bo'ladi.

---

## 5. Evaluate Reverse Polish Notation

RPN (postfix) ifodani stack bilan hisoblang. (LeetCode 150)

```js
function evalRPN(tokens) {
  const stack = [];
  const ops = {
    '+': (a, b) => a + b,
    '-': (a, b) => a - b,
    '*': (a, b) => a * b,
    '/': (a, b) => Math.trunc(a / b), // nolga tomon yaxlitlash
  };
  for (const t of tokens) {
    if (t in ops) {
      const b = stack.pop();
      const a = stack.pop();
      stack.push(ops[t](a, b)); // tartib muhim: a OP b
    } else {
      stack.push(Number(t));
    }
  }
  return stack.pop();
}

// evalRPN(["2","1","+","3","*"]) => 9   ((2+1)*3)
// evalRPN(["4","13","5","/","+"]) => 6  (4 + 13/5)
```

**Complexity:** Time `O(n)`, Space `O(n)`.

**Izoh:** Token'larni ketma-ket o'qiymiz: son bo'lsa stack'ga push; operator bo'lsa ikki operandni pop qilamiz. **Tartib muhim** — `b` avval pop bo'ladi (chunki u keyin push qilingan), shuning uchun `a OP b`. `-` va `/` da tartibni almashtirsangiz natija noto'g'ri chiqadi. Bo'lishda `Math.trunc` nolga tomon yaxlitlaydi (manfiy sonlar uchun `Math.floor` dan farqi bor).

---

## 6. Next Greater Element I

Har element uchun o'ngdagi birinchi kattaroq elementni monotonic stack bilan toping. (LeetCode 496)

```js
function nextGreaterElement(nums1, nums2) {
  const nextGreater = new Map(); // qiymat -> o'ngdagi birinchi kattaroq
  const stack = [];              // kamayuvchi tartibdagi "kutayotgan" qiymatlar
  for (const num of nums2) {
    while (stack.length && num > stack[stack.length - 1]) {
      nextGreater.set(stack.pop(), num);
    }
    stack.push(num);
  }
  // stackda qolganlar uchun next greater yo'q (-1)
  return nums1.map((n) => nextGreater.get(n) ?? -1);
}

// nextGreaterElement([4,1,2], [1,3,4,2]) => [-1, 3, -1]
// nextGreaterElement([2,4], [1,2,3,4])   => [3, -1]
```

**Complexity:** Time `O(n + m)`, Space `O(n)`.

**Izoh:** `nums2` bo'ylab monotonic (kamayuvchi) stack yuritamiz. Joriy element stack tepasidagidan katta bo'lsa — demak stack tepasidagi elementlar uchun "next greater" topildi, ularni pop qilib map'ga yozamiz. `nums2` qiymatlari unik bo'lgani uchun qiymat-kalit ishlaydi. Oxirida `nums1` ning har elementi uchun map'dan o'qiymiz; topilmasa `-1`.

---

## 7. Daily Temperatures

Har kun uchun necha kundan keyin harorat ko'tarilishini monotonic stack bilan hisoblang. (LeetCode 739)

```js
function dailyTemperatures(temps) {
  const res = new Array(temps.length).fill(0);
  const stack = []; // indekslarni saqlaydi (haroratlari kamayuvchi)
  for (let i = 0; i < temps.length; i++) {
    while (stack.length && temps[i] > temps[stack[stack.length - 1]]) {
      const prev = stack.pop();
      res[prev] = i - prev; // necha kun kutgani
    }
    stack.push(i);
  }
  return res;
}

// dailyTemperatures([73,74,75,71,69,72,76,73])
// => [1, 1, 4, 2, 1, 1, 0, 0]
```

**Complexity:** Time `O(n)`, Space `O(n)`.

**Izoh:** "Next greater" ning indeks versiyasi. Stack'da **indekslarni** saqlaymiz (haroratlari kamayuvchi tartibda). Joriy kun harorati stack tepasidagi kun haroratidan yuqori bo'lsa — o'sha kun uchun javob topildi: `i - prev` (necha kun o'tgani). Har indeks bir marta push va bir marta pop bo'lgani uchun `O(n)`.

---

## 8. Largest Rectangle in Histogram

Histogrammada eng katta to'rtburchak yuzasini monotonic stack bilan toping. (LeetCode 84)

```js
function largestRectangleArea(heights) {
  const stack = []; // o'suvchi balandlikdagi indekslar
  let maxArea = 0;
  // oxiriga 0 qo'shamiz — stackni majburan bo'shatish uchun
  const h = [...heights, 0];
  for (let i = 0; i < h.length; i++) {
    while (stack.length && h[i] < h[stack[stack.length - 1]]) {
      const height = h[stack.pop()];
      // kenglik: chap chegara stack tepasi (yoki agar bo'sh bo'lsa, 0 dan)
      const width = stack.length ? i - stack[stack.length - 1] - 1 : i;
      maxArea = Math.max(maxArea, height * width);
    }
    stack.push(i);
  }
  return maxArea;
}

// largestRectangleArea([2,1,5,6,2,3]) => 10  (5 va 6 ustunlar: 5*2)
// largestRectangleArea([2,4])         => 4
```

**Complexity:** Time `O(n)`, Space `O(n)`.

**Izoh:** Har ustun uchun "shu ustun balandligini saqlab qancha kenglikda cho'zilish mumkin" degan savolga javob qidiramiz. Monotonic o'suvchi stack indekslarni saqlaydi. Joriy ustun stack tepasidagidan past bo'lsa, tepa ustun uchun o'ng chegara topildi (joriy `i`), chap chegara esa stack'dagi keyingi element. Kenglik = `i - chap_chegara - 1`. Oxiriga `0` qo'shish stack'da qolgan hamma ustunni hisoblashga majbur qiladi.

---

## 9. Sliding Window Maximum

`k` o'lchamli oynani siljitib har oynadagi maksimumni deque bilan O(n)da toping. (LeetCode 239)

```js
function maxSlidingWindow(nums, k) {
  const res = [];
  const deque = []; // indekslar, qiymatlari kamayuvchi tartibda
  for (let i = 0; i < nums.length; i++) {
    // oynadan chiqib ketgan indeksni oldidan olib tashla
    if (deque.length && deque[0] <= i - k) deque.shift();
    // joriy elementdan kichik orqadagilarni olib tashla (ular endi kerak emas)
    while (deque.length && nums[deque[deque.length - 1]] < nums[i]) {
      deque.pop();
    }
    deque.push(i);
    // oyna to'liq shakllanganda maksimumni (deque oldi) yoz
    if (i >= k - 1) res.push(nums[deque[0]]);
  }
  return res;
}

// maxSlidingWindow([1,3,-1,-3,5,3,6,7], 3) => [3, 3, 5, 5, 6, 7]
// maxSlidingWindow([1], 1)                 => [1]
```

**Complexity:** Time `O(n)`, Space `O(k)`.

**Izoh:** Monotonic deque'da **indekslarni** saqlaymiz, ularning qiymatlari kamayuvchi tartibda. Deque oldida (front) doim joriy oyna maksimumi turadi. Har qadamda: (1) oynadan chiqib ketgan indeksni oldidan olib tashlaymiz; (2) joriy elementdan kichik orqadagilarni olib tashlaymiz (ular hech qachon maksimum bo'la olmaydi). Har indeks bir marta kiritilib bir marta chiqarilgani uchun `O(n)`.

---

## 10. Basic Calculator II

`+ - * /` va bo'sh joylardan iborat ifodani stack bilan hisoblang (qavssiz). (LeetCode 227)

```js
function calculate(s) {
  const stack = [];
  let num = 0;
  let prevOp = '+'; // oldingi operator (boshlanishida +)
  for (let i = 0; i < s.length; i++) {
    const ch = s[i];
    if (ch >= '0' && ch <= '9') {
      num = num * 10 + (ch.charCodeAt(0) - 48);
    }
    // operator yoki oxirgi belgi bo'lsa, oldingi operatorni qo'llaymiz
    if ((ch !== ' ' && !(ch >= '0' && ch <= '9')) || i === s.length - 1) {
      if (prevOp === '+') stack.push(num);
      else if (prevOp === '-') stack.push(-num);
      else if (prevOp === '*') stack.push(stack.pop() * num);
      else if (prevOp === '/') stack.push(Math.trunc(stack.pop() / num));
      prevOp = ch;
      num = 0;
    }
  }
  return stack.reduce((a, b) => a + b, 0);
}

// calculate("3+2*2")     => 7
// calculate(" 3/2 ")     => 1
// calculate(" 3+5 / 2 ") => 5
```

**Complexity:** Time `O(n)`, Space `O(n)`.

**Izoh:** Qavssiz ifodada `*` va `/` ustuvor. Hiyla: har sonni **oldingi** operator bilan qayta ishlaymiz. `+`/`-` da sonni (ehtimol manfiy) stack'ga qo'yamiz — ularning hisobini keyinga qoldiramiz. `*`/`/` da darrov stack tepasi bilan amal bajaramiz (ustuvorlik shu yerda hal bo'ladi). Oxirida stack'dagi hamma sonni qo'shsak — to'g'ri natija. Oxirgi belgini ham operator kabi ishlash uchun `i === s.length - 1` shartini qo'shamiz.

---

## 11. Decode String

`3[a2[c]]` kabi kodlangan satrni stack bilan dekodlang. (LeetCode 394)

```js
function decodeString(s) {
  const countStack = [];  // takrorlash sonlari
  const strStack = [];     // '[' dan oldingi qurilgan matn
  let curr = '';
  let num = 0;
  for (const ch of s) {
    if (ch >= '0' && ch <= '9') {
      num = num * 10 + (ch.charCodeAt(0) - 48);
    } else if (ch === '[') {
      countStack.push(num);   // ichki blok necha marta takrorlanishi
      strStack.push(curr);    // hozirgacha qurilganni saqlab tur
      num = 0;
      curr = '';
    } else if (ch === ']') {
      const repeat = countStack.pop();
      const prev = strStack.pop();
      curr = prev + curr.repeat(repeat); // ichki blokni takrorlab biriktiramiz
    } else {
      curr += ch; // oddiy harf
    }
  }
  return curr;
}

// decodeString("3[a]2[bc]")   => "aaabcbc"
// decodeString("3[a2[c]]")    => "accaccacc"
// decodeString("2[abc]3[cd]ef") => "abcabccdcdcdef"
```

**Complexity:** Time `O(maxK · n)` (n — chiqish uzunligi), Space `O(n)`.

**Izoh:** Ikki stack: biri takrorlash sonlari, ikkinchisi `[` ochilishidan oldingi matn. `[` ko'rsak joriy holatni stack'ga saqlab, ichki blok uchun toza holatdan boshlaymiz. `]` ko'rsak ichki blokni mos songa takrorlab, oldingi (tashqi) matnga biriktiramiz. Ko'p qavatli (`3[a2[c]]`) ichki-tashqi takrorlash stack tabiati bilan o'zicha hal bo'ladi.

---

## 12. Remove Duplicate Letters

Har harf bir marta qoladigan, leksikografik eng kichik natijani monotonic stack bilan toping. (LeetCode 316)

```js
function removeDuplicateLetters(s) {
  const lastIndex = {}; // har harfning oxirgi pozitsiyasi
  for (let i = 0; i < s.length; i++) lastIndex[s[i]] = i;

  const stack = [];
  const inStack = new Set();
  for (let i = 0; i < s.length; i++) {
    const ch = s[i];
    if (inStack.has(ch)) continue; // bu harf allaqachon natijada
    // stack tepasi joriy harfdan katta va keyin yana uchrasa — uni olib tashla
    while (
      stack.length &&
      stack[stack.length - 1] > ch &&
      lastIndex[stack[stack.length - 1]] > i
    ) {
      inStack.delete(stack.pop());
    }
    stack.push(ch);
    inStack.add(ch);
  }
  return stack.join('');
}

// removeDuplicateLetters("bcabc")    => "abc"
// removeDuplicateLetters("cbacdcbc") => "acdb"
```

**Complexity:** Time `O(n)`, Space `O(1)` (alifbo cheklangan, ko'pi bilan 26).

**Izoh:** Maqsad — har harf bir marta, natija leksikografik eng kichik. Monotonic (o'suvchi) stack yuritamiz. Yangi harf stack tepasidan kichik bo'lsa **va** tepadagi harf keyinroq yana uchrasa (`lastIndex` bilan tekshiramiz), tepadagini olib tashlaymiz — chunki uni keyin qaytarib qo'sha olamiz va hozir kichikroq harfni oldinga chiqarish foydali. `inStack` to'plami harf allaqachon natijada ekanini `O(1)` da tekshiradi.

---

← [DSA bo'limiga qaytish](../../dsa/README.md)
