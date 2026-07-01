# DSA Masalalar — LeetCode va NeetCode

Bu yo'l xaritasi algoritmlar va ma'lumotlar tuzilmalarini (DSA — Data Structures & Algorithms) tizimli ravishda mashq qilish uchun tuzilgan. Barcha masalalar **pattern** (naqsh) bo'yicha guruhlangan va har biriga **LeetCode** havolasi berilgan. Tuzilma **NeetCode 150** ro'yxatiga mos keladi.

## Qanday mashq qilish kerak

DSA'da muvaffaqiyat — bu ko'p masala ishlash emas, balki **pattern'larni tushunish**dir. Quyidagi tamoyillarga amal qiling:

- **Pattern bo'yicha o'rganing.** Masalalarni tasodifiy tanlamang. Bitta mavzuni (masalan, *Two Pointers*) to'liq tugatib, keyingisiga o'ting. Shunda miyangiz naqshni tanib olishga o'rganadi.
- **Kuniga 2–3 masala.** Har kuni oz-ozdan, lekin muntazam ishlash bir kunda 20 ta masalani ishlab, keyin bir hafta dam olishdan yaxshiroq. Muntazamlik (consistency) — eng muhim omil.
- **Brute force → Optimize.** Avval eng oddiy (brute force) yechimni yozing, ishlaganiga ishonch hosil qiling, so'ng vaqt va xotira murakkabligini (*time & space complexity*) yaxshilashga harakat qiling. Optimallashtirishni O(n²) dan O(n) ga tushirish kabi bosqichlarda o'ylang.
- **Takrorlash (spaced repetition).** Ishlagan masalangizni 1 kun, 1 hafta va 1 oydan keyin qayta ishlang. Agar yechimni eslay olmasangiz, demak siz uni haqiqatan ham tushunmagansiz.
- **Vaqt qo'ying, keyin qarang.** Har bir masalaga 20–40 daqiqa vaqt ajrating. Agar yecha olmasangiz, yechimni (editorial yoki NeetCode video) ko'ring, tushuning va **keyin o'zingiz qaytadan yozing**.
- **O'zingiz kod yozing.** Yechimni ko'chirib olmang. Faqat kodni qo'lda, boshdan yozganingizdagina u xotirangizda qoladi.

## NeetCode 150 va Blind 75 haqida

- **Blind 75** — bu Facebook'da ishlagan muhandis tomonidan tuzilgan, suhbatlarga (*coding interview*) tayyorgarlik uchun eng muhim 75 ta masala ro'yxati. Vaqtingiz kam bo'lsa, aynan shundan boshlang.
- **NeetCode 150** — Blind 75'ning kengaytirilgan versiyasi. 150 ta masala har bir pattern'ni yaxshiroq qamrab oladi. Bu roadmap aynan shu ro'yxat tuzilishiga mos keladi.

**Tavsiya:** [NeetCode.io roadmap](https://neetcode.io/roadmap) sahifasidan foydalaning — u masalalar orasidagi bog'liqlik va o'rganish tartibini vizual ko'rsatib beradi. Har bir masala uchun bepul video tushuntirish mavjud. Mashq qilish uchun esa [NeetCode practice](https://neetcode.io/practice) sahifasida progress'ingizni kuzatib borishingiz mumkin.

## LeetCode'da qanday mashq qilish

1. [LeetCode.com](https://leetcode.com)'da bepul akkaunt oching.
2. Masalani o'qing, misollarni (*examples*) va cheklovlarni (*constraints*) diqqat bilan tahlil qiling.
3. Yechimni o'zingiz yozing, "Run" bilan test qiling, so'ng "Submit" qiling.
4. Submit qilgandan keyin **"Solutions"** va **"Discuss"** bo'limlarini ko'rib, boshqalarning yondashuvlarini o'rganing.
5. O'z tilingizni (Python, Java, C++, JavaScript) tanlang va faqat unda ishlang — sintaksisni yodlab olish uchun.

---

## Arrays & Hashing

**Pattern:** Massivlar (*arrays*) va hash jadvallar (*hash map / set*) — barcha DSA'ning asosi. Ko'p masalalarda O(n²) brute force yechimni hash map yordamida O(n) gacha tezlashtirish mumkin. Asosiy g'oya: qidiruvni (*lookup*) O(1) ga aylantirish uchun elementlarni hash strukturada saqlash.

| Masala | Daraja | LeetCode |
|--------|--------|----------|
| Two Sum | Easy | https://leetcode.com/problems/two-sum/ |
| Contains Duplicate | Easy | https://leetcode.com/problems/contains-duplicate/ |
| Valid Anagram | Easy | https://leetcode.com/problems/valid-anagram/ |
| Group Anagrams | Medium | https://leetcode.com/problems/group-anagrams/ |
| Top K Frequent Elements | Medium | https://leetcode.com/problems/top-k-frequent-elements/ |
| Product of Array Except Self | Medium | https://leetcode.com/problems/product-of-array-except-self/ |
| Valid Sudoku | Medium | https://leetcode.com/problems/valid-sudoku/ |
| Longest Consecutive Sequence | Medium | https://leetcode.com/problems/longest-consecutive-sequence/ |

## Two Pointers

**Pattern:** Ikki ko'rsatkich (*two pointers*) — massiv yoki string bo'ylab bir vaqtning o'zida ikkita indeks bilan harakatlanish. Odatda biri boshdan, biri oxirdan yuriladi (yoki sekin/tez ko'rsatkichlar). Saralangan (*sorted*) massivlarda O(n²) o'rniga O(n) yechim beradi.

| Masala | Daraja | LeetCode |
|--------|--------|----------|
| Valid Palindrome | Easy | https://leetcode.com/problems/valid-palindrome/ |
| Two Sum II - Input Array Is Sorted | Medium | https://leetcode.com/problems/two-sum-ii-input-array-is-sorted/ |
| 3Sum | Medium | https://leetcode.com/problems/3sum/ |
| Container With Most Water | Medium | https://leetcode.com/problems/container-with-most-water/ |
| Trapping Rain Water | Hard | https://leetcode.com/problems/trapping-rain-water/ |

## Sliding Window

**Pattern:** Suriladigan oyna (*sliding window*) — massiv yoki string ustida uzunligi o'zgaruvchan (yoki qat'iy) "oyna"ni surib borish. Oynani kengaytirish/qisqartirish orqali subarray/substring masalalarini O(n) da yechadi. "Eng uzun/qisqa ... " degan savollar ko'pincha shu pattern bilan yechiladi.

| Masala | Daraja | LeetCode |
|--------|--------|----------|
| Best Time to Buy and Sell Stock | Easy | https://leetcode.com/problems/best-time-to-buy-and-sell-stock/ |
| Longest Substring Without Repeating Characters | Medium | https://leetcode.com/problems/longest-substring-without-repeating-characters/ |
| Longest Repeating Character Replacement | Medium | https://leetcode.com/problems/longest-repeating-character-replacement/ |
| Permutation in String | Medium | https://leetcode.com/problems/permutation-in-string/ |
| Minimum Window Substring | Hard | https://leetcode.com/problems/minimum-window-substring/ |

## Stack

**Pattern:** Stek (*stack*) — LIFO (Last In, First Out) prinsipida ishlaydigan tuzilma. Qavslarni tekshirish, monotonic stack (o'sib/kamayib boruvchi), ifodalarni hisoblash kabi masalalarda asosiy vosita.

| Masala | Daraja | LeetCode |
|--------|--------|----------|
| Valid Parentheses | Easy | https://leetcode.com/problems/valid-parentheses/ |
| Min Stack | Medium | https://leetcode.com/problems/min-stack/ |
| Evaluate Reverse Polish Notation | Medium | https://leetcode.com/problems/evaluate-reverse-polish-notation/ |
| Generate Parentheses | Medium | https://leetcode.com/problems/generate-parentheses/ |
| Daily Temperatures | Medium | https://leetcode.com/problems/daily-temperatures/ |
| Car Fleet | Medium | https://leetcode.com/problems/car-fleet/ |
| Largest Rectangle in Histogram | Hard | https://leetcode.com/problems/largest-rectangle-in-histogram/ |

## Binary Search

**Pattern:** Ikkilik qidiruv (*binary search*) — saralangan ma'lumotda qidiruv maydonini har qadamda ikkiga bo'lish orqali O(log n) da javob topish. Faqat massivda emas, balki "javob oralig'i" bo'yicha ham qidirish mumkin (masalan, Koko Eating Bananas).

| Masala | Daraja | LeetCode |
|--------|--------|----------|
| Binary Search | Easy | https://leetcode.com/problems/binary-search/ |
| Search a 2D Matrix | Medium | https://leetcode.com/problems/search-a-2d-matrix/ |
| Koko Eating Bananas | Medium | https://leetcode.com/problems/koko-eating-bananas/ |
| Find Minimum in Rotated Sorted Array | Medium | https://leetcode.com/problems/find-minimum-in-rotated-sorted-array/ |
| Search in Rotated Sorted Array | Medium | https://leetcode.com/problems/search-in-rotated-sorted-array/ |
| Time Based Key-Value Store | Medium | https://leetcode.com/problems/time-based-key-value-store/ |
| Median of Two Sorted Arrays | Hard | https://leetcode.com/problems/median-of-two-sorted-arrays/ |

## Linked List

**Pattern:** Bog'langan ro'yxat (*linked list*) — tugunlar (*nodes*) ko'rsatkichlar orqali bog'langan tuzilma. Fast & slow pointers (tez/sekin ko'rsatkich), ro'yxatni teskari qilish (*reverse*), sikl (*cycle*) aniqlash asosiy texnikalar. Dummy node ko'p masalada kodni soddalashtiradi.

| Masala | Daraja | LeetCode |
|--------|--------|----------|
| Reverse Linked List | Easy | https://leetcode.com/problems/reverse-linked-list/ |
| Merge Two Sorted Lists | Easy | https://leetcode.com/problems/merge-two-sorted-lists/ |
| Linked List Cycle | Easy | https://leetcode.com/problems/linked-list-cycle/ |
| Reorder List | Medium | https://leetcode.com/problems/reorder-list/ |
| Remove Nth Node From End of List | Medium | https://leetcode.com/problems/remove-nth-node-from-end-of-list/ |
| Copy List with Random Pointer | Medium | https://leetcode.com/problems/copy-list-with-random-pointer/ |
| Add Two Numbers | Medium | https://leetcode.com/problems/add-two-numbers/ |
| Find the Duplicate Number | Medium | https://leetcode.com/problems/find-the-duplicate-number/ |
| LRU Cache | Medium | https://leetcode.com/problems/lru-cache/ |
| Merge k Sorted Lists | Hard | https://leetcode.com/problems/merge-k-sorted-lists/ |
| Reverse Nodes in k-Group | Hard | https://leetcode.com/problems/reverse-nodes-in-k-group/ |

## Trees

**Pattern:** Daraxtlar (*trees*) — rekursiya (*recursion*) va DFS/BFS (chuqurlik/kenglik bo'yicha aylanish) bilan ishlanadi. Binary tree, BST (binary search tree), level-order traversal asosiy mavzular. Ko'p masala "har bir tugun uchun bir xil ishni takrorlash" g'oyasiga asoslanadi.

| Masala | Daraja | LeetCode |
|--------|--------|----------|
| Invert Binary Tree | Easy | https://leetcode.com/problems/invert-binary-tree/ |
| Maximum Depth of Binary Tree | Easy | https://leetcode.com/problems/maximum-depth-of-binary-tree/ |
| Diameter of Binary Tree | Easy | https://leetcode.com/problems/diameter-of-binary-tree/ |
| Balanced Binary Tree | Easy | https://leetcode.com/problems/balanced-binary-tree/ |
| Same Tree | Easy | https://leetcode.com/problems/same-tree/ |
| Subtree of Another Tree | Easy | https://leetcode.com/problems/subtree-of-another-tree/ |
| Lowest Common Ancestor of a Binary Search Tree | Medium | https://leetcode.com/problems/lowest-common-ancestor-of-a-binary-search-tree/ |
| Binary Tree Level Order Traversal | Medium | https://leetcode.com/problems/binary-tree-level-order-traversal/ |
| Binary Tree Right Side View | Medium | https://leetcode.com/problems/binary-tree-right-side-view/ |
| Validate Binary Search Tree | Medium | https://leetcode.com/problems/validate-binary-search-tree/ |
| Kth Smallest Element in a BST | Medium | https://leetcode.com/problems/kth-smallest-element-in-a-bst/ |
| Construct Binary Tree from Preorder and Inorder Traversal | Medium | https://leetcode.com/problems/construct-binary-tree-from-preorder-and-inorder-traversal/ |
| Binary Tree Maximum Path Sum | Hard | https://leetcode.com/problems/binary-tree-maximum-path-sum/ |
| Serialize and Deserialize Binary Tree | Hard | https://leetcode.com/problems/serialize-and-deserialize-binary-tree/ |

## Tries

**Pattern:** Trie (prefix daraxti) — string'larni belgi-belgi bo'yicha daraxt ko'rinishida saqlaydigan tuzilma. Prefiks bo'yicha qidiruv, autocomplete, so'zlar lug'ati kabi masalalarda samarali.

| Masala | Daraja | LeetCode |
|--------|--------|----------|
| Implement Trie (Prefix Tree) | Medium | https://leetcode.com/problems/implement-trie-prefix-tree/ |
| Design Add and Search Words Data Structure | Medium | https://leetcode.com/problems/design-add-and-search-words-data-structure/ |
| Word Search II | Hard | https://leetcode.com/problems/word-search-ii/ |

## Heap / Priority Queue

**Pattern:** Uyum (*heap*) yoki ustuvor navbat (*priority queue*) — eng katta/kichik elementni O(log n) da olishga imkon beradi. "K-th eng katta", "top K", "median" kabi masalalarda ideal. Ko'pincha ikkita heap (min + max) birga ishlatiladi.

| Masala | Daraja | LeetCode |
|--------|--------|----------|
| Kth Largest Element in a Stream | Easy | https://leetcode.com/problems/kth-largest-element-in-a-stream/ |
| Last Stone Weight | Easy | https://leetcode.com/problems/last-stone-weight/ |
| K Closest Points to Origin | Medium | https://leetcode.com/problems/k-closest-points-to-origin/ |
| Kth Largest Element in an Array | Medium | https://leetcode.com/problems/kth-largest-element-in-an-array/ |
| Task Scheduler | Medium | https://leetcode.com/problems/task-scheduler/ |
| Design Twitter | Medium | https://leetcode.com/problems/design-twitter/ |
| Find Median from Data Stream | Hard | https://leetcode.com/problems/find-median-from-data-stream/ |

## Backtracking

**Pattern:** Orqaga qaytish (*backtracking*) — barcha mumkin bo'lgan variantlarni rekursiv ravishda sinab ko'rish va noto'g'ri yo'lni "orqaga qaytarib" tashlab yuborish. Subsets, permutations, combinations kabi masalalarda qo'llaniladi. Daraxt shaklidagi qaror maydonini (*decision tree*) tasavvur qiling.

| Masala | Daraja | LeetCode |
|--------|--------|----------|
| Subsets | Medium | https://leetcode.com/problems/subsets/ |
| Combination Sum | Medium | https://leetcode.com/problems/combination-sum/ |
| Permutations | Medium | https://leetcode.com/problems/permutations/ |
| Subsets II | Medium | https://leetcode.com/problems/subsets-ii/ |
| Combination Sum II | Medium | https://leetcode.com/problems/combination-sum-ii/ |
| Word Search | Medium | https://leetcode.com/problems/word-search/ |
| Palindrome Partitioning | Medium | https://leetcode.com/problems/palindrome-partitioning/ |
| Letter Combinations of a Phone Number | Medium | https://leetcode.com/problems/letter-combinations-of-a-phone-number/ |
| N-Queens | Hard | https://leetcode.com/problems/n-queens/ |

## Graphs

**Pattern:** Graflar (*graphs*) — tugunlar va qirralar (*edges*) tarmog'i. DFS, BFS, Union-Find, topologik saralash (*topological sort*) asosiy algoritmlar. Matritsa (grid) ko'pincha yashirin graf sifatida ishlatiladi (masalan, orollar sonini topish).

| Masala | Daraja | LeetCode |
|--------|--------|----------|
| Number of Islands | Medium | https://leetcode.com/problems/number-of-islands/ |
| Clone Graph | Medium | https://leetcode.com/problems/clone-graph/ |
| Max Area of Island | Medium | https://leetcode.com/problems/max-area-of-island/ |
| Pacific Atlantic Water Flow | Medium | https://leetcode.com/problems/pacific-atlantic-water-flow/ |
| Surrounded Regions | Medium | https://leetcode.com/problems/surrounded-regions/ |
| Rotting Oranges | Medium | https://leetcode.com/problems/rotting-oranges/ |
| Course Schedule | Medium | https://leetcode.com/problems/course-schedule/ |
| Course Schedule II | Medium | https://leetcode.com/problems/course-schedule-ii/ |
| Redundant Connection | Medium | https://leetcode.com/problems/redundant-connection/ |
| Number of Connected Components in an Undirected Graph | Medium | https://leetcode.com/problems/number-of-connected-components-in-an-undirected-graph/ |
| Graph Valid Tree | Medium | https://leetcode.com/problems/graph-valid-tree/ |
| Word Ladder | Hard | https://leetcode.com/problems/word-ladder/ |

## Dynamic Programming 1D

**Pattern:** Dinamik dasturlash (*dynamic programming, DP*) — katta masalani kichik qism-masalalarga (*subproblems*) bo'lib, natijalarni saqlash (*memoization*) orqali qayta hisoblashdan qochish. 1D DP'da odatda bitta massiv (`dp[]`) yetarli. "Nechta yo'l bor", "maksimal/minimal qiymat" savollarida qo'llaniladi.

| Masala | Daraja | LeetCode |
|--------|--------|----------|
| Climbing Stairs | Easy | https://leetcode.com/problems/climbing-stairs/ |
| Min Cost Climbing Stairs | Easy | https://leetcode.com/problems/min-cost-climbing-stairs/ |
| House Robber | Medium | https://leetcode.com/problems/house-robber/ |
| House Robber II | Medium | https://leetcode.com/problems/house-robber-ii/ |
| Longest Palindromic Substring | Medium | https://leetcode.com/problems/longest-palindromic-substring/ |
| Palindromic Substrings | Medium | https://leetcode.com/problems/palindromic-substrings/ |
| Decode Ways | Medium | https://leetcode.com/problems/decode-ways/ |
| Coin Change | Medium | https://leetcode.com/problems/coin-change/ |
| Maximum Product Subarray | Medium | https://leetcode.com/problems/maximum-product-subarray/ |
| Word Break | Medium | https://leetcode.com/problems/word-break/ |
| Longest Increasing Subsequence | Medium | https://leetcode.com/problems/longest-increasing-subsequence/ |
| Partition Equal Subset Sum | Medium | https://leetcode.com/problems/partition-equal-subset-sum/ |

## Dynamic Programming 2D

**Pattern:** 2D DP'da qism-masalalar ikkita o'zgaruvchiga bog'liq bo'ladi, shuning uchun `dp[i][j]` matritsa ishlatiladi. Ikki string'ni solishtirish (LCS, edit distance) yoki grid bo'yicha yo'llar sonini topishda qo'llaniladi.

| Masala | Daraja | LeetCode |
|--------|--------|----------|
| Unique Paths | Medium | https://leetcode.com/problems/unique-paths/ |
| Longest Common Subsequence | Medium | https://leetcode.com/problems/longest-common-subsequence/ |
| Best Time to Buy and Sell Stock with Cooldown | Medium | https://leetcode.com/problems/best-time-to-buy-and-sell-stock-with-cooldown/ |
| Coin Change II | Medium | https://leetcode.com/problems/coin-change-ii/ |
| Target Sum | Medium | https://leetcode.com/problems/target-sum/ |
| Edit Distance | Medium | https://leetcode.com/problems/edit-distance/ |

## Greedy

**Pattern:** Ochko'z algoritm (*greedy*) — har qadamda o'sha lahzada eng yaxshi ko'ringan tanlovni qilish. Har doim ham to'g'ri ishlamaydi, lekin to'g'ri ishlaganda DP'dan sodda va tez bo'ladi. Isbotlash (*proof of correctness*) muhim.

| Masala | Daraja | LeetCode |
|--------|--------|----------|
| Maximum Subarray | Medium | https://leetcode.com/problems/maximum-subarray/ |
| Jump Game | Medium | https://leetcode.com/problems/jump-game/ |
| Jump Game II | Medium | https://leetcode.com/problems/jump-game-ii/ |
| Gas Station | Medium | https://leetcode.com/problems/gas-station/ |
| Hand of Straights | Medium | https://leetcode.com/problems/hand-of-straights/ |

## Intervals

**Pattern:** Oraliqlar (*intervals*) — `[start, end]` ko'rinishidagi juftliklar ustida ishlash. Deyarli har doim avval `start` (yoki `end`) bo'yicha saralanadi (*sort*), so'ng ustma-ust tushish (*overlap*) tekshiriladi. Meeting rooms, merge intervals klassik misollar.

| Masala | Daraja | LeetCode |
|--------|--------|----------|
| Insert Interval | Medium | https://leetcode.com/problems/insert-interval/ |
| Merge Intervals | Medium | https://leetcode.com/problems/merge-intervals/ |
| Non-overlapping Intervals | Medium | https://leetcode.com/problems/non-overlapping-intervals/ |
| Meeting Rooms | Easy | https://leetcode.com/problems/meeting-rooms/ |
| Meeting Rooms II | Medium | https://leetcode.com/problems/meeting-rooms-ii/ |

## Bit Manipulation

**Pattern:** Bit amallari (*bit manipulation*) — sonlarni ikkilik (binary) ko'rinishda AND (`&`), OR (`|`), XOR (`^`), shift (`<<`, `>>`) amallari bilan ishlash. XOR ayniqsa foydali: `a ^ a = 0` xususiyati takrorlangan elementlarni topishda ishlatiladi.

| Masala | Daraja | LeetCode |
|--------|--------|----------|
| Single Number | Easy | https://leetcode.com/problems/single-number/ |
| Number of 1 Bits | Easy | https://leetcode.com/problems/number-of-1-bits/ |
| Counting Bits | Easy | https://leetcode.com/problems/counting-bits/ |
| Reverse Bits | Easy | https://leetcode.com/problems/reverse-bits/ |
| Missing Number | Easy | https://leetcode.com/problems/missing-number/ |
| Sum of Two Integers | Medium | https://leetcode.com/problems/sum-of-two-integers/ |

---

## Mashq qilish rejasi

Quyida taxminiy **8 haftalik** reja keltirilgan. Bu tempda kuniga 2–3 masala ishlab, taxminan 2 oyda butun NeetCode 150'ni yakunlaysiz. Agar vaqtingiz ko'p bo'lsa tezlashtiring, kam bo'lsa sekinlashtiring — muhimi to'xtamaslik.

| Hafta | Mavzular |
|-------|----------|
| 1-hafta | Arrays & Hashing, Two Pointers |
| 2-hafta | Sliding Window, Stack |
| 3-hafta | Binary Search, Linked List |
| 4-hafta | Trees, Tries |
| 5-hafta | Heap / Priority Queue, Backtracking |
| 6-hafta | Graphs |
| 7-hafta | Dynamic Programming 1D, Dynamic Programming 2D |
| 8-hafta | Greedy, Intervals, Bit Manipulation |

**Kunlik tartib:**

- Har kuni **2–3 masala** ishlang (masalan, 2 ta yangi + 1 ta eski takror).
- Bitta mavzuni **to'liq tugatib**, keyingisiga o'ting. Yarim tashlab ketmang.
- Har bir masalada **avval o'zingiz** yechishga urinib ko'ring (20–40 daqiqa), keyingina yechim/videoni ko'ring.
- Har hafta oxirida o'sha haftadagi 2–3 ta eng qiyin masalani **qayta ishlang**.

**Umumiy maslahatlar:**

- Vaqt/xotira murakkabligini (*Big-O*) har bir yechim uchun aytib bera olishga o'rganing.
- Yechimni yodlamang — **pattern**ni tushuning. Bir marta *Two Pointers*'ni tushunsangiz, o'nlab yangi masalani yecha olasiz.
- Suhbatga tayyorlanayotgan bo'lsangiz, yechimingizni ovoz chiqarib tushuntirib ko'ring (*talk through your solution*).

**Foydali resurslar:**

- **[NeetCode.io](https://neetcode.io/roadmap)** — roadmap va har bir masalaga bepul video tushuntirish.
- **[NeetCode Practice](https://neetcode.io/practice)** — progress'ingizni kuzatib borish uchun.
- **[LeetCode.com](https://leetcode.com)** — masalalarni yechish va boshqalarning yechimlarini o'rganish platformasi.
- **[LeetCode Explore](https://leetcode.com/explore/)** — mavzular bo'yicha tuzilgan bepul o'quv kartalari.

---

← [Loyihalar bo'limiga qaytish](./README.md)
