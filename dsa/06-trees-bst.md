# Trees va Binary Search Trees

Tree (daraxt) — bu ierarxik (siyrak, tarmoqlangan) ma'lumotlarni ifodalovchi data structure. Fayl tizimlari, DOM, ma'lumotlar bazasi indekslari, JSON — barchasi tree. DSA intervyularida tree masalalari rekursiya, traversal va BST xossalarini tushunishni sinaydi.

Bu hujjatda tree terminologiyasi, binary tree va binary search tree (BST), barcha traversal'lar (recursive va iterative), BST operatsiyalari hamda eng ko'p uchraydigan pattern'larni ko'rib chiqamiz.

Misollarda quyidagi tugun strukturasidan foydalanamiz:

```js
class TreeNode {
  constructor(val, left = null, right = null) {
    this.val = val;
    this.left = left;
    this.right = right;
  }
}
```

## Mundarija

- [Tree terminologiyasi](#tree-terminologiyasi)
- [Binary tree](#binary-tree)
- [Binary search tree (BST)](#binary-search-tree-bst)
- [Traversal'lar](#traversallar)
- [BST operatsiyalari](#bst-operatsiyalari)
- [Balanced vs unbalanced](#balanced-vs-unbalanced)
- [Pattern'lar](#patternlar)
- [Savol-javoblar](#savol-javoblar)
- [Masalalar](#masalalar)

## Tree terminologiyasi

**💡 Tushuncha:** Tree — bu tugunlar (`node`) to'plami bo'lib, ular qirralar (`edge`) orqali bog'langan va sikl (loop) hosil qilmaydi. Har bir tugun bir nechta "farzand" (`child`) ga ega bo'lishi mumkin.

Asosiy atamalar:

- **Root** — eng yuqoridagi tugun (ota-onasi yo'q).
- **Node** — daraxtning har bir elementi (qiymat + farzandlarga havola).
- **Leaf** — farzandi yo'q tugun (uchidagi tugun).
- **Parent / Child** — bir tugun bilan uning to'g'ridan-to'g'ri pastdagi tuguni o'rtasidagi munosabat.
- **Subtree** — istalgan tugun va undan pastdagi barcha tugunlar tashkil etgan daraxt.
- **Height** — tugundan eng uzoq leaf'gacha bo'lgan qirralar soni. Daraxt height'i = root height'i.
- **Depth** — root'dan shu tugungacha bo'lgan qirralar soni.

```
        A          depth 0   (root)
       / \
      B   C        depth 1
     / \
    D   E          depth 2   (D, E — leaf)

height(A) = 2,  depth(E) = 2
```

**⚠️ Ehtiyot bo'l:** Height va depth tez chalkashtiriladi. **Depth** — yuqoridan pastga (root'dan), **height** — pastdan yuqoriga (leaf'dan) o'lchanadi.

## Binary tree

**💡 Tushuncha:** Binary tree — har bir tugun **ko'pi bilan 2 ta** farzandga ega bo'lgan tree: `left` va `right`.

Maxsus turlari:

- **Full** — har bir tugunda 0 yoki 2 ta farzand bor.
- **Complete** — oxirgi darajadan tashqari hamma daraja to'la, oxirgisi chapdan to'ladi (heap shunday).
- **Perfect** — barcha leaf'lar bir xil darajada, hamma ichki tugunda 2 ta farzand.
- **Balanced** — chap va o'ng subtree balandliklari farqi har tugunda cheklangan (`<= 1`).

`n` tugunli balanced binary tree balandligi `O(log n)`, unbalanced (zanjirga aylangan) holatda `O(n)`.

## Binary search tree (BST)

**💡 Tushuncha:** BST — bu tartiblangan binary tree. Har bir tugun uchun **chap subtree'dagi barcha qiymatlar tugundan kichik**, **o'ng subtree'dagi barchasi katta**.

```
        8
       / \
      3   10
     / \    \
    1   6    14
```

Bu xossa tufayli qidiruv har qadamda yarmiga qisqaradi — balanced BST'da `search`, `insert`, `delete` `O(log n)`.

**⚠️ Ehtiyot bo'l:** BST xossasi **lokal** emas, **global** — chap subtree'dagi *barcha* tugun (faqat bevosita farzand emas) root'dan kichik bo'lishi shart. Validatsiyada ko'pchilik shu yerda xato qiladi.

## Traversal'lar

Tree'ni aylanib chiqishning (har tugunni bir marta ko'rish) asosiy usullari.

### DFS (Depth-First Search) — recursive

```js
// Inorder: chap -> tugun -> o'ng   (BST'da o'sish tartibida beradi)
function inorder(node, result = []) {
  if (!node) return result;
  inorder(node.left, result);
  result.push(node.val);
  inorder(node.right, result);
  return result;
}

// Preorder: tugun -> chap -> o'ng   (daraxtni nusxalash/serializatsiya)
function preorder(node, result = []) {
  if (!node) return result;
  result.push(node.val);
  preorder(node.left, result);
  preorder(node.right, result);
  return result;
}

// Postorder: chap -> o'ng -> tugun   (daraxtni o'chirish, hisoblash)
function postorder(node, result = []) {
  if (!node) return result;
  postorder(node.left, result);
  postorder(node.right, result);
  result.push(node.val);
  return result;
}
// Hammasi: Time O(n), Space O(h) — h = balandlik (stack)
```

### DFS — iterative (stack bilan)

```js
function inorderIterative(root) {
  const result = [];
  const stack = [];
  let curr = root;
  while (curr || stack.length) {
    while (curr) {        // eng chapga tushib boramiz
      stack.push(curr);
      curr = curr.left;
    }
    curr = stack.pop();   // tugunni ol
    result.push(curr.val);
    curr = curr.right;    // o'ngga o't
  }
  return result;
}
// Time O(n), Space O(h)
```

### BFS (Breadth-First Search) — level-order, queue bilan

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
// Time O(n), Space O(w) — w = eng keng daraja
```

**💡 Tushuncha:** `levelSize` ni har iteratsiya boshida olib, bitta darajani alohida guruhlaymiz. Bu "har daraja alohida massiv" natijani beradi.

## BST operatsiyalari

### Search

```js
function search(root, target) {
  let node = root;
  while (node) {
    if (target === node.val) return node;
    node = target < node.val ? node.left : node.right;
  }
  return null;
}
// Time O(h): balanced O(log n), worst O(n)
```

### Insert

```js
function insert(root, val) {
  if (!root) return new TreeNode(val);
  if (val < root.val) root.left = insert(root.left, val);
  else if (val > root.val) root.right = insert(root.right, val);
  return root;
}
// Time O(h)
```

### Delete

O'chirishda 3 holat: leaf, bitta farzand, ikkita farzand.

```js
function deleteNode(root, key) {
  if (!root) return null;
  if (key < root.val) {
    root.left = deleteNode(root.left, key);
  } else if (key > root.val) {
    root.right = deleteNode(root.right, key);
  } else {
    // topildi
    if (!root.left) return root.right;   // 0 yoki o'ng farzand
    if (!root.right) return root.left;    // faqat chap farzand
    // 2 farzand: o'ng subtree'ning eng kichigini (inorder successor) topamiz
    let succ = root.right;
    while (succ.left) succ = succ.left;
    root.val = succ.val;
    root.right = deleteNode(root.right, succ.val);
  }
  return root;
}
// Time O(h)
```

**⚠️ Ehtiyot bo'l:** Ikki farzandli tugunni o'chirganda uni **inorder successor** (o'ng subtree'ning minimumi) yoki **inorder predecessor** (chap subtree maksimumi) bilan almashtirish kerak — shunda BST xossasi buzilmaydi.

## Balanced vs unbalanced

**💡 Tushuncha:** BST tartiblangan ma'lumot ketma-ket qo'shilsa (`1,2,3,4,5`), u zanjirga (linked list) aylanadi va balandligi `O(n)` bo'lib qoladi — barcha operatsiyalar `O(n)` ga aylanadi.

```
1
 \
  2
   \
    3      // unbalanced — aslida linked list
```

Buni oldini olish uchun **self-balancing** daraxtlar bor:

- **AVL tree** — har tugunda chap/o'ng balandlik farqi `<= 1`. Qat'iy balanslangan, qidiruv juda tez; insert/delete'da rotatsiyalar ko'proq.
- **Red-Black tree** — yumshoqroq balans (rang qoidalari orqali). Insert/delete amaliy jihatdan tezroq. JS engine'lari, C++ `std::map`, Java `TreeMap` shularga asoslanadi.

Ikkalasi ham barcha operatsiyani `O(log n)` da kafolatlaydi. Intervyuda odatda ulardan amalda implementatsiya so'ralmaydi, lekin "balanced bo'lmasa nima bo'ladi va qanday hal qilinadi" degan tushunchani bilish kerak.

## Pattern'lar

### 1. Max depth (balandlik)

```js
function maxDepth(root) {
  if (!root) return 0;
  return 1 + Math.max(maxDepth(root.left), maxDepth(root.right));
}
// Time O(n), Space O(h)
```

### 2. Validate BST (min/max chegara bilan)

```js
function isValidBST(node, min = -Infinity, max = Infinity) {
  if (!node) return true;
  if (node.val <= min || node.val >= max) return false;
  return isValidBST(node.left, min, node.val) &&
         isValidBST(node.right, node.val, max);
}
// Time O(n), Space O(h)
```

### 3. Lowest Common Ancestor (BST'da)

```js
function lca(root, p, q) {
  let node = root;
  while (node) {
    if (p < node.val && q < node.val) node = node.left;
    else if (p > node.val && q > node.val) node = node.right;
    else return node; // yo'llar shu yerda ajraladi
  }
  return null;
}
// Time O(h), Space O(1)
```

### 4. Invert tree (oynaviy aks ettirish)

```js
function invertTree(root) {
  if (!root) return null;
  [root.left, root.right] = [invertTree(root.right), invertTree(root.left)];
  return root;
}
// Time O(n), Space O(h)
```

### 5. Diameter (eng uzun yo'l)

```js
function diameter(root) {
  let best = 0;
  function depth(node) {
    if (!node) return 0;
    const l = depth(node.left);
    const r = depth(node.right);
    best = Math.max(best, l + r); // shu tugundan o'tuvchi yo'l
    return 1 + Math.max(l, r);
  }
  depth(root);
  return best;
}
// Time O(n), Space O(h)
```

## Savol-javoblar

### ❓ Height va depth orasidagi farq nima?

**✅ Javob:** Depth — root'dan shu tugungacha bo'lgan qirralar soni (yuqoridan pastga). Height — shu tugundan eng uzoq leaf'gacha bo'lgan qirralar soni (pastdan yuqoriga). Root'ning depth'i har doim 0, leaf'larning height'i 0. Butun daraxt height'i = root height'i.

### ❓ Binary tree bilan BST farqi nimada?

**✅ Javob:** Binary tree — har tugunda ko'pi bilan 2 farzand bo'lgan har qanday daraxt, qiymatlar tartibi yo'q. BST — qo'shimcha tartib xossasiga ega binary tree: chap subtree'dagi hamma qiymat tugundan kichik, o'ng subtree'dagi hamma qiymat katta. Shu tartib tufayli BST `O(log n)` qidiruv beradi.

### ❓ Inorder, preorder, postorder qachon qaysi biri kerak?

**✅ Javob:** Inorder (chap-tugun-o'ng) — BST'da qiymatlarni o'sish tartibida beradi, saralangan ma'lumot kerak bo'lganda ishlatiladi. Preorder (tugun-chap-o'ng) — daraxtni nusxalash yoki serializatsiya qilishda (root birinchi kerak). Postorder (chap-o'ng-tugun) — daraxtni o'chirish yoki pastdan yuqoriga hisoblash (masalan subtree yig'indisi) kerak bo'lganda.

### ❓ DFS va BFS ni qachon tanlaysiz?

**✅ Javob:** DFS (recursive yoki stack) — yo'l, balandlik, subtree masalalari uchun, kam xotira (`O(h)`). BFS (queue, level-order) — daraja bo'yicha ishlash, eng yaqin tugunni topish, level-order natija kerak bo'lganda. Keng daraxtda BFS ko'p xotira (`O(w)`), chuqur daraxtda DFS stack overflow xavfi bor.

### ❓ Iterative inorder traversal qanday ishlaydi?

**✅ Javob:** Stack ishlatamiz: avval eng chapga tushib, yo'ldagi har tugunni stack'ga qo'yamiz. Keyin stack'dan tugun olib, uning qiymatini yozamiz va uning o'ng farzandiga o'tamiz. Bu rekursiyaning stack'ini qo'lda boshqarish — natija recursive inorder bilan bir xil, lekin chuqur daraxtda stack overflow'dan himoyalaydi.

### ❓ Level-order traversal'da har darajani qanday ajratasiz?

**✅ Javob:** Queue bilan BFS qilamiz, lekin har iteratsiya boshida `queue.length` ni `levelSize` ga saqlaymiz — bu joriy darajadagi tugunlar soni. Faqat shuncha tugunni ishlab, ularning farzandlarini queue'ga qo'shamiz. Shunda har darajani alohida massivga ajratamiz.

### ❓ BST'da qidiruv nega O(log n)?

**✅ Javob:** Har qadamda joriy tugun bilan solishtirib, faqat bir tomonga (chap yoki o'ng) tushamiz — qolgan yarim subtree butunlay tashlanadi. Balanced BST'da har qadam qidiruv maydonini yarmiga qisqartiradi, shuning uchun `O(log n)`. Lekin daraxt unbalanced bo'lsa (zanjir), bu `O(n)` ga aylanadi.

### ❓ BST'dan tugun o'chirishning eng murakkab holati qaysi va qanday hal qilinadi?

**✅ Javob:** Ikkita farzandi bo'lgan tugunni o'chirish. Uni shunchaki olib tashlasak BST buziladi. Yechim: inorder successor (o'ng subtree'ning eng kichik tuguni) yoki inorder predecessor (chap subtree'ning eng katta tuguni) qiymatini shu tugunga ko'chiramiz, keyin o'sha successor/predecessor'ni o'chiramiz. Shunda tartib saqlanadi.

### ❓ Validate BST'ni nega oddiy "chap < tugun < o'ng" tekshiruvi bilan qilib bo'lmaydi?

**✅ Javob:** Chunki BST xossasi global. Faqat bevosita farzandni solishtirsak, masalan o'ng subtree ichida root'dan kichik tugun bo'lib qolishi mumkin va biz buni ko'rmaymiz. To'g'ri yechim — har tugun uchun ruxsat etilgan `(min, max)` oraliqni rekursiv uzatish va har tugun shu oraliqda ekanini tekshirish.

### ❓ BST'da Lowest Common Ancestor'ni nega oddiy binary tree'dagidan tezroq topish mumkin?

**✅ Javob:** BST tartibidan foydalanamiz: agar `p` va `q` ikkalasi ham joriy tugundan kichik bo'lsa — chapga, ikkalasi katta bo'lsa — o'ngga tushamiz. Birinchi marta yo'llar ajralgan tugun (biri kichik, biri katta yoki biri tugunga teng) — bu LCA. Bu `O(h)`, qo'shimcha xotirasiz. Oddiy binary tree'da esa ikkala tugunni izlash uchun butun daraxtni `O(n)` aylanish kerak.

### ❓ Diameter masalasida nega depth funksiyasi ichida hisoblaymiz?

**✅ Javob:** Diametr — istalgan ikki tugun orasidagi eng uzun yo'l. Har tugun uchun "shu tugundan o'tuvchi eng uzun yo'l" = chap balandlik + o'ng balandlik. Biz depth funksiyasini rekursiv hisoblayotganimizda har tugunda `l + r` ni global maksimum bilan yangilaymiz. Bu bir o'tishda `O(n)` da ishlaydi — har tugun uchun alohida balandlik hisoblamasdan.

### ❓ Balanced bo'lmagan BST nega xavfli va qanday hal qilinadi?

**✅ Javob:** Tartiblangan ma'lumot ketma-ket qo'shilsa BST zanjirga (linked list) aylanadi, balandligi `O(n)` bo'ladi va barcha operatsiyalar `O(n)` ga sekinlashadi — ya'ni BST afzalligi yo'qoladi. Hal: AVL yoki Red-Black tree kabi self-balancing daraxtlar rotatsiyalar orqali balandlikni `O(log n)` da ushlab turadi.

### ❓ AVL va Red-Black tree farqi nimada?

**✅ Javob:** AVL qat'iyroq balanslangan (balandlik farqi har tugunda `<= 1`), shuning uchun qidiruv tezroq, lekin insert/delete'da rotatsiyalar ko'proq. Red-Black yumshoqroq balanslangan (rang qoidalari), insert/delete amaliy jihatdan tezroq — shuning uchun ko'p kutubxonalar (Java `TreeMap`, C++ `std::map`) uni ishlatadi. Ikkalasi ham operatsiyalarni `O(log n)` kafolatlaydi.

### ❓ Recursive traversal'ning xotira murakkabligi nima?

**✅ Javob:** `O(h)`, bu yerda `h` — daraxt balandligi, chunki rekursiya call stack'i bir vaqtda ko'pi bilan `h` ta freym saqlaydi. Balanced daraxtda `O(log n)`, unbalanced (zanjir) daraxtda `O(n)`. BFS'da esa xotira `O(w)` — eng keng daraja kengligiga bog'liq.

### ❓ Tree masalalarini yechishda umumiy yondashuv qanday?

**✅ Javob:** Ko'p tree masalasi rekursiv: (1) base case'ni aniqlash (odatda `if (!node) return ...`); (2) chap va o'ng subtree natijalarini rekursiv olish; (3) ularni joriy tugun bilan birlashtirish. "Bu masalada har tugun o'z farzandlaridan qanday natija kutadi?" deb o'ylash kalit. Ba'zan global o'zgaruvchi (diameter kabi) yoki min/max chegara (validate BST kabi) uzatish kerak bo'ladi.

## Masalalar

> Yechimlar: [solutions/dsa/06-trees-bst.md](../solutions/dsa/06-trees-bst.md)

1. **Maximum Depth of Binary Tree** — daraxtning maksimal chuqurligini qaytaring. (LeetCode 104)

2. **Invert Binary Tree** — daraxtni oynaviy aks ettiring (chap/o'ng almashtirib). (LeetCode 226)

3. **Same Tree** — ikki daraxt strukturasi va qiymatlari bir xilmi aniqlang. (LeetCode 100)

4. **Symmetric Tree** — daraxt o'z markaziga nisbatan simmetrikmi? (LeetCode 101)

5. **Binary Tree Level Order Traversal** — har darajani alohida massiv qilib qaytaring. (LeetCode 102)

6. **Validate Binary Search Tree** — berilgan daraxt to'g'ri BST ekanini tekshiring. (LeetCode 98)

7. **Lowest Common Ancestor of a BST** — ikki tugunning eng past umumiy ajdodini toping. (LeetCode 235)

8. **Diameter of Binary Tree** — daraxtning diametrini (eng uzun yo'l) toping. (LeetCode 543)

9. **Balanced Binary Tree** — daraxt height-balanced ekanini aniqlang. (LeetCode 110)

10. **Kth Smallest Element in a BST** — BST'dagi `k`-eng kichik elementni qaytaring. (LeetCode 230)

11. **Convert Sorted Array to BST** — saralangan massivdan balanced BST yarating. (LeetCode 108)

12. **Binary Tree Right Side View** — daraxtga o'ngdan qaraganda ko'rinadigan tugunlarni qaytaring. (LeetCode 199)

---

← [DSA bo'limiga qaytish](./README.md)
