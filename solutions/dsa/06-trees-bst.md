# Trees va Binary Search Trees — Yechimlar

Har masala uchun to'liq ishlaydigan kod, time/space complexity va o'zbekcha izoh. Kod JavaScript'da. Hamma yechim quyidagi tugun strukturasidan foydalanadi:

```js
class TreeNode {
  constructor(val, left = null, right = null) {
    this.val = val;
    this.left = left;
    this.right = right;
  }
}
```

---

## 1. Maximum Depth of Binary Tree

Daraxtning maksimal chuqurligini qaytaring. (LeetCode 104)

```js
function maxDepth(root) {
  if (!root) return 0;
  return 1 + Math.max(maxDepth(root.left), maxDepth(root.right));
}

// maxDepth(build([3,9,20,null,null,15,7])) => 3
```

**Complexity:** Time `O(n)`, Space `O(h)` (h — balandlik, rekursiya stack'i).

**Izoh:** Klassik rekursiv "pastdan yuqoriga" hisob. Base case: bo'sh tugun chuqurligi `0`. Har tugun uchun chuqurlik = chap va o'ng subtree chuqurliklarining kattasi `+ 1` (joriy tugun). Rekursiya call stack'i bir vaqtda ko'pi bilan `h` ta freym saqlaydi, shuning uchun xotira `O(h)`.

---

## 2. Invert Binary Tree

Daraxtni oynaviy aks ettiring (chap/o'ng almashtirib). (LeetCode 226)

```js
function invertTree(root) {
  if (!root) return null;
  [root.left, root.right] = [invertTree(root.right), invertTree(root.left)];
  return root;
}

// invertTree(build([4,2,7,1,3,6,9])) => [4,7,2,9,6,3,1]
```

**Complexity:** Time `O(n)`, Space `O(h)`.

**Izoh:** Har tugunda chap va o'ng subtree'larni almashtiramiz, so'ng har biriga rekursiv tushib aynan shuni qilamiz. Destructuring (`[a, b] = [invert(right), invert(left)]`) ikkala tomonni ham rekursiv invert qilib, keyin almashtiradi — tartib muhim, aks holda bitta tomonni ikki marta ishlab qo'yish mumkin.

---

## 3. Same Tree

Ikki daraxt strukturasi va qiymatlari bir xilmi aniqlang. (LeetCode 100)

```js
function isSameTree(p, q) {
  if (!p && !q) return true;          // ikkalasi ham bo'sh
  if (!p || !q || p.val !== q.val) return false; // biri bo'sh yoki qiymat farq
  return isSameTree(p.left, q.left) && isSameTree(p.right, q.right);
}

// isSameTree(build([1,2,3]), build([1,2,3])) => true
// isSameTree(build([1,2]), build([1,null,2])) => false
```

**Complexity:** Time `O(n)`, Space `O(h)`.

**Izoh:** Ikki daraxtni bir vaqtda kezamiz. Base case'lar: ikkalasi `null` bo'lsa — bir xil; biri `null` yoki qiymatlar farq qilsa — har xil. Aks holda chap subtree'lar ham, o'ng subtree'lar ham bir xil bo'lishi shart (`&&` bilan). Struktura va qiymat — ikkalasi tekshiriladi.

---

## 4. Symmetric Tree

Daraxt o'z markaziga nisbatan simmetrikmi? (LeetCode 101)

```js
function isSymmetric(root) {
  if (!root) return true;
  function mirror(a, b) {
    if (!a && !b) return true;
    if (!a || !b || a.val !== b.val) return false;
    // tashqi juftlik (a.left vs b.right) va ichki juftlik (a.right vs b.left)
    return mirror(a.left, b.right) && mirror(a.right, b.left);
  }
  return mirror(root.left, root.right);
}

// isSymmetric(build([1,2,2,3,4,4,3])) => true
// isSymmetric(build([1,2,2,null,3,null,3])) => false
```

**Complexity:** Time `O(n)`, Space `O(h)`.

**Izoh:** "Same tree" ning oynaviy varianti. Ikki tugun bir-birining oynaviy aksi bo'lishi uchun qiymatlari teng bo'lib, **tashqi** juftliklar (`a.left` ↔ `b.right`) va **ichki** juftliklar (`a.right` ↔ `b.left`) o'zaro mos kelishi kerak. Root'ning chap va o'ng subtree'larini shu `mirror` funksiyasi bilan solishtiramiz.

---

## 5. Binary Tree Level Order Traversal

Har darajani alohida massiv qilib qaytaring. (LeetCode 102)

```js
function levelOrder(root) {
  if (!root) return [];
  const result = [];
  const queue = [root];
  while (queue.length) {
    const levelSize = queue.length;
    const level = [];
    for (let i = 0; i < levelSize; i++) {
      const node = queue.shift();
      level.push(node.val);
      if (node.left) queue.push(node.left);
      if (node.right) queue.push(node.right);
    }
    result.push(level);
  }
  return result;
}

// levelOrder(build([3,9,20,null,null,15,7])) => [[3], [9,20], [15,7]]
```

**Complexity:** Time `O(n)`, Space `O(w)` (w — eng keng daraja).

**Izoh:** BFS queue bilan. Asosiy hiyla — har iteratsiya boshida `queue.length` ni `levelSize` ga saqlash: bu joriy darajadagi tugunlar soni. Faqat shuncha tugunni ishlab, ularning farzandlarini queue'ga qo'shamiz. Shunda har daraja alohida massivga ajraladi. (Katta ishlab chiqarishda `shift()` `O(n)` bo'lgani uchun index-pointer queue afzal, lekin tushuncha uchun bu yetarli.)

---

## 6. Validate Binary Search Tree

Berilgan daraxt to'g'ri BST ekanini tekshiring. (LeetCode 98)

```js
function isValidBST(root, min = -Infinity, max = Infinity) {
  if (!root) return true;
  if (root.val <= min || root.val >= max) return false;
  return isValidBST(root.left, min, root.val) &&
         isValidBST(root.right, root.val, max);
}

// isValidBST(build([2,1,3]))     => true
// isValidBST(build([5,1,4,null,null,3,6])) => false
```

**Complexity:** Time `O(n)`, Space `O(h)`.

**Izoh:** BST xossasi **global** — chap subtree'dagi *barcha* qiymat tugundan kichik, o'ngdagi *barchasi* katta. Faqat bevosita farzandni solishtirish yetmaydi. Shuning uchun har tugun uchun ruxsat etilgan `(min, max)` oraliqni rekursiv uzatamiz: chapga tushganda `max` ni joriy qiymatga, o'ngga tushganda `min` ni joriy qiymatga toraytiramiz. Tugun shu oraliqdan chiqsa — BST emas.

---

## 7. Lowest Common Ancestor of a BST

Ikki tugunning eng past umumiy ajdodini toping. (LeetCode 235)

```js
function lowestCommonAncestor(root, p, q) {
  let node = root;
  while (node) {
    if (p.val < node.val && q.val < node.val) node = node.left;   // ikkalasi chapda
    else if (p.val > node.val && q.val > node.val) node = node.right; // ikkalasi o'ngda
    else return node; // yo'llar shu yerda ajraladi -> LCA
  }
  return null;
}

// p=2, q=8 bo'lsa: lowestCommonAncestor(build([6,2,8,0,4,7,9]), p, q) => 6
// p=2, q=4 bo'lsa: => 2
```

**Complexity:** Time `O(h)`, Space `O(1)`.

**Izoh:** BST tartibidan foydalanamiz. Ikkala tugun ham joriy tugundan kichik bo'lsa — LCA chap subtree'da; ikkalasi katta bo'lsa — o'ngda. Birinchi marta yo'llar ajralganda (biri kichik, biri katta yoki biri tugunga teng) — shu tugun LCA. Oddiy binary tree'dagi `O(n)` yondashuvdan farqli, BST'da `O(h)` va qo'shimcha xotirasiz.

---

## 8. Diameter of Binary Tree

Daraxtning diametrini (eng uzun yo'l) toping. (LeetCode 543)

```js
function diameterOfBinaryTree(root) {
  let best = 0;
  function depth(node) {
    if (!node) return 0;
    const l = depth(node.left);
    const r = depth(node.right);
    best = Math.max(best, l + r); // shu tugundan o'tuvchi yo'l (qirralar soni)
    return 1 + Math.max(l, r);
  }
  depth(root);
  return best;
}

// diameterOfBinaryTree(build([1,2,3,4,5])) => 3  (yo'l: 4-2-1-3 yoki 5-2-1-3)
```

**Complexity:** Time `O(n)`, Space `O(h)`.

**Izoh:** Diametr — istalgan ikki tugun orasidagi eng uzun yo'l (qirralarda). Har tugun uchun "shu tugundan o'tuvchi eng uzun yo'l" = chap balandlik + o'ng balandlik. Depth funksiyasini rekursiv hisoblayotganda har tugunda `l + r` ni global `best` bilan yangilaymiz. Bu bitta o'tishda `O(n)` — har tugun uchun balandlikni qayta hisoblamaymiz.

---

## 9. Balanced Binary Tree

Daraxt height-balanced ekanini aniqlang. (LeetCode 110)

```js
function isBalanced(root) {
  // -1 = balanssiz signal; aks holda balandlik qaytadi
  function check(node) {
    if (!node) return 0;
    const l = check(node.left);
    if (l === -1) return -1;
    const r = check(node.right);
    if (r === -1) return -1;
    if (Math.abs(l - r) > 1) return -1; // shu tugunda balans buzildi
    return 1 + Math.max(l, r);
  }
  return check(root) !== -1;
}

// isBalanced(build([3,9,20,null,null,15,7])) => true
// isBalanced(build([1,2,2,3,3,null,null,4,4])) => false
```

**Complexity:** Time `O(n)`, Space `O(h)`.

**Izoh:** Sodda yondashuv har tugunda balandlikni alohida hisoblaydi — `O(n²)`. Bu yerda balandlikni hisoblash bilan birga balansni ham tekshiramiz: `-1` qiymatini "balanssiz" signali sifatida yuqoriga uzatamiz va topilgan zahoti rekursiyani to'xtatamiz. Shu sababli bitta `O(n)` o'tishda yechiladi. Balanced — har tugunda chap/o'ng balandlik farqi `<= 1`.

---

## 10. Kth Smallest Element in a BST

BST'dagi `k`-eng kichik elementni qaytaring. (LeetCode 230)

```js
function kthSmallest(root, k) {
  const stack = [];
  let curr = root;
  while (curr || stack.length) {
    while (curr) {        // eng chapga tushib boramiz
      stack.push(curr);
      curr = curr.left;
    }
    curr = stack.pop();
    if (--k === 0) return curr.val; // k-chi inorder element
    curr = curr.right;
  }
  return -1;
}

// kthSmallest(build([3,1,4,null,2]), 1) => 1
// kthSmallest(build([5,3,6,2,4,null,null,1]), 3) => 3
```

**Complexity:** Time `O(h + k)`, Space `O(h)`.

**Izoh:** BST'da **inorder** (chap → tugun → o'ng) traversal qiymatlarni o'sish tartibida beradi. Demak `k`-eng kichik element = inorder'dagi `k`-chi tashrif. Iterative inorder (stack bilan) ishlatamiz va har tugunni ko'rganda `k` ni kamaytiramiz; `0` ga yetganda javob shu tugun. Butun daraxtni kezish shart emas — faqat birinchi `k` ta elementgacha.

---

## 11. Convert Sorted Array to BST

Saralangan massivdan balanced BST yarating. (LeetCode 108)

```js
function sortedArrayToBST(nums) {
  function build(lo, hi) {
    if (lo > hi) return null;
    const mid = (lo + hi) >> 1;     // o'rta element — root
    const node = new TreeNode(nums[mid]);
    node.left = build(lo, mid - 1);  // chap yarim
    node.right = build(mid + 1, hi); // o'ng yarim
    return node;
  }
  return build(0, nums.length - 1);
}

// sortedArrayToBST([-10,-3,0,5,9]) => balanced BST, root = 0
```

**Complexity:** Time `O(n)`, Space `O(log n)` (rekursiya stack'i; chiqish daraxtidan tashqari).

**Izoh:** Balanced BST hosil qilish uchun har oraliqning **o'rta** elementini root qilamiz — shunda chap va o'ng subtree'lar deyarli teng o'lchamda bo'ladi. Massiv saralangan bo'lgani uchun o'rta element BST root sharti (chap kichik, o'ng katta) ni avtomatik qondiradi. Rekursiv ravishda chap yarmidan chap subtree, o'ng yarmidan o'ng subtree quramiz — natija balandligi `O(log n)`.

---

## 12. Binary Tree Right Side View

Daraxtga o'ngdan qaraganda ko'rinadigan tugunlarni qaytaring. (LeetCode 199)

```js
function rightSideView(root) {
  if (!root) return [];
  const result = [];
  const queue = [root];
  while (queue.length) {
    const levelSize = queue.length;
    for (let i = 0; i < levelSize; i++) {
      const node = queue.shift();
      if (i === levelSize - 1) result.push(node.val); // darajadagi oxirgi (eng o'ng)
      if (node.left) queue.push(node.left);
      if (node.right) queue.push(node.right);
    }
  }
  return result;
}

// rightSideView(build([1,2,3,null,5,null,4])) => [1, 3, 4]
```

**Complexity:** Time `O(n)`, Space `O(w)`.

**Izoh:** O'ngdan qaraganda har darajadan faqat **eng o'ngdagi** tugun ko'rinadi. Level-order (BFS) qilamiz va har darajaning **oxirgi** tugunini (`i === levelSize - 1`) natijaga qo'shamiz. `levelSize` darajalarni ajratish imkonini beradi. (Muqobil: DFS bilan o'ngdan boshlab har yangi chuqurlikdagi birinchi tugunni olish ham mumkin.)

---

← [DSA bo'limiga qaytish](../../dsa/README.md)
