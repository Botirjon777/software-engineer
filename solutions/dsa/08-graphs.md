# Graphs — Yechimlar

Bu bo'lim `08-graphs.md` dagi 12 ta masalaning to'liq ishlaydigan JS yechimlarini, complexity tahlilini va o'zbekcha izohlarni o'z ichiga oladi.

## 1. Number of Islands

2D grid'da `'1'` (quruqlik) bloklarini sanaymiz. Har bir ko'rilmagan `'1'`dan DFS boshlab, butun orolni "cho'ktiramiz" (suvga aylantiramiz).

```js
function numIslands(grid) {
  if (!grid || grid.length === 0) return 0;
  const rows = grid.length, cols = grid[0].length;
  let count = 0;

  function dfs(r, c) {
    if (r < 0 || c < 0 || r >= rows || c >= cols || grid[r][c] !== '1') return;
    grid[r][c] = '0'; // visited deb belgilaymiz
    dfs(r + 1, c);
    dfs(r - 1, c);
    dfs(r, c + 1);
    dfs(r, c - 1);
  }

  for (let r = 0; r < rows; r++) {
    for (let c = 0; c < cols; c++) {
      if (grid[r][c] === '1') {
        count++;
        dfs(r, c);
      }
    }
  }
  return count;
}

// Misol:
const grid = [
  ['1','1','0','0'],
  ['1','0','0','1'],
  ['0','0','1','1'],
];
console.log(numIslands(grid)); // 2
```

- **Time:** O(R·C) — har katak bir marta ko'riladi.
- **Space:** O(R·C) — worst case rekursiya chuqurligi (butun grid bitta orol).

**Izoh:** Grid'ni o'zgartirmaslik kerak bo'lsa, alohida `visited` Set ishlatamiz. DFS o'rniga BFS (queue) ham ishlaydi — chuqur grid'larda stack overflow'dan himoya qiladi.

---

## 2. Clone Graph

Grafning chuqur nusxasini (deep copy) qaytaramiz. Hash-map orqali asl node → nusxa moslamasini saqlab, sikl va takroriy node'larni boshqaramiz.

```js
// Node tuzilishi: { val: number, neighbors: Node[] }
function cloneGraph(node) {
  if (!node) return null;
  const map = new Map(); // asl node → nusxa

  function dfs(n) {
    if (map.has(n)) return map.get(n); // allaqachon nusxalangan
    const copy = { val: n.val, neighbors: [] };
    map.set(n, copy); // rekursiyadan OLDIN saqlash (sikl uchun muhim!)
    for (const nb of n.neighbors) {
      copy.neighbors.push(dfs(nb));
    }
    return copy;
  }

  return dfs(node);
}
```

- **Time:** O(V + E) — har vertex va edge bir marta.
- **Space:** O(V) — map va rekursiya stack.

**Izoh:** `map.set(n, copy)` ni qo'shnilarni nusxalashdan **oldin** chaqirish shart. Aks holda siklda (A → B → A) cheksiz rekursiya bo'ladi, chunki A hali map'da bo'lmaydi.

---

## 3. Course Schedule

`numCourses` va prerequisite juftliklar berilgan; barcha kurslarni tugatish mumkinmi (ya'ni grafda sikl yo'qmi)? Kahn's algoritmi (BFS, in-degree) bilan yechamiz.

```js
function canFinish(numCourses, prerequisites) {
  const adj = Array.from({ length: numCourses }, () => []);
  const inDegree = new Array(numCourses).fill(0);

  // [a, b] → b kursni a dan oldin olish kerak: b → a edge
  for (const [a, b] of prerequisites) {
    adj[b].push(a);
    inDegree[a]++;
  }

  const queue = [];
  for (let i = 0; i < numCourses; i++) {
    if (inDegree[i] === 0) queue.push(i);
  }

  let completed = 0;
  while (queue.length) {
    const node = queue.shift();
    completed++;
    for (const nb of adj[node]) {
      if (--inDegree[nb] === 0) queue.push(nb);
    }
  }

  return completed === numCourses; // hammasi tugadi → sikl yo'q
}

console.log(canFinish(2, [[1, 0]]));        // true
console.log(canFinish(2, [[1, 0], [0, 1]])); // false (sikl)
```

- **Time:** O(V + E).
- **Space:** O(V + E).

**Izoh:** Agar sikl bo'lsa, sikldagi kurslarning in-degree'si hech qachon 0 bo'lmaydi, queue'ga tushmaydi va `completed < numCourses` bo'ladi.

---

## 4. Course Schedule II

Kurslarni tugatish tartibini (topological order) qaytaramiz; mumkin bo'lmasa bo'sh array.

```js
function findOrder(numCourses, prerequisites) {
  const adj = Array.from({ length: numCourses }, () => []);
  const inDegree = new Array(numCourses).fill(0);

  for (const [a, b] of prerequisites) {
    adj[b].push(a);
    inDegree[a]++;
  }

  const queue = [];
  for (let i = 0; i < numCourses; i++) {
    if (inDegree[i] === 0) queue.push(i);
  }

  const order = [];
  while (queue.length) {
    const node = queue.shift();
    order.push(node);
    for (const nb of adj[node]) {
      if (--inDegree[nb] === 0) queue.push(nb);
    }
  }

  return order.length === numCourses ? order : [];
}

console.log(findOrder(4, [[1, 0], [2, 0], [3, 1], [3, 2]])); // [0,1,2,3] (yoki [0,2,1,3])
console.log(findOrder(2, [[0, 1], [1, 0]]));                 // [] (sikl)
```

- **Time:** O(V + E).
- **Space:** O(V + E).

**Izoh:** #3 dan farqi — faqat `true/false` emas, balki haqiqiy tartibni yig'amiz. Sikl bo'lsa `order.length < numCourses`, shuning uchun bo'sh array qaytaramiz.

---

## 5. Word Ladder

`beginWord`dan `endWord`gacha eng qisqa transformatsiya zanjiri uzunligini topamiz. Har bir so'z — vertex, bir harf farqli so'zlar — qo'shnilar. Eng qisqa yo'l uchun BFS.

```js
function ladderLength(beginWord, endWord, wordList) {
  const words = new Set(wordList);
  if (!words.has(endWord)) return 0;

  let queue = [beginWord];
  let steps = 1;

  while (queue.length) {
    const next = [];
    for (const word of queue) {
      if (word === endWord) return steps;
      for (let i = 0; i < word.length; i++) {
        for (let c = 97; c <= 122; c++) { // 'a'..'z'
          const cand = word.slice(0, i) + String.fromCharCode(c) + word.slice(i + 1);
          if (words.has(cand)) {
            words.delete(cand); // takror tashrifni oldini olish
            next.push(cand);
          }
        }
      }
    }
    queue = next;
    steps++;
  }
  return 0;
}

console.log(ladderLength('hit', 'cog', ['hot','dot','dog','lot','log','cog'])); // 5
```

- **Time:** O(N · L² · 26) — N so'z, L so'z uzunligi (slice O(L)).
- **Space:** O(N · L).

**Izoh:** BFS level-by-level aylangani uchun `endWord` topilgan birinchi qadam — eng qisqa masofa. `words.delete(cand)` visited belgilash vazifasini bajaradi.

---

## 6. Number of Connected Components

`n` vertex va edge'lar berilgan; bog'langan komponentlar sonini topamiz. DFS yoki Union-Find bilan. Bu yerda Union-Find ishlatamiz.

```js
function countComponents(n, edges) {
  const parent = Array.from({ length: n }, (_, i) => i);
  const rank = new Array(n).fill(0);
  let count = n; // har vertex alohida komponent

  function find(x) {
    while (parent[x] !== x) {
      parent[x] = parent[parent[x]]; // path compression
      x = parent[x];
    }
    return x;
  }

  function union(x, y) {
    const rx = find(x), ry = find(y);
    if (rx === ry) return;
    if (rank[rx] < rank[ry]) parent[rx] = ry;
    else if (rank[rx] > rank[ry]) parent[ry] = rx;
    else { parent[ry] = rx; rank[rx]++; }
    count--; // ikki komponent birlashdi
  }

  for (const [a, b] of edges) union(a, b);
  return count;
}

console.log(countComponents(5, [[0, 1], [1, 2], [3, 4]])); // 2
```

- **Time:** O(V + E·α(V)) — α inverse Ackermann, deyarli doimiy.
- **Space:** O(V).

**Izoh:** Har bir edge ikki vertexni birlashtiradi; har muvaffaqiyatli `union` komponentlar sonini 1 ga kamaytiradi. DFS variantida ham xuddi shu natija O(V + E) da chiqadi.

---

## 7. Graph Valid Tree

`n` vertex va edge'lar yaroqli daraxt hosil qiladimi? Daraxt sharti: **(1)** aniq `n-1` ta edge, **(2)** sikl yo'q, **(3)** to'liq bog'langan. Union-Find ideal.

```js
function validTree(n, edges) {
  // Daraxtda aniq n-1 ta edge bo'lishi shart
  if (edges.length !== n - 1) return false;

  const parent = Array.from({ length: n }, (_, i) => i);

  function find(x) {
    while (parent[x] !== x) {
      parent[x] = parent[parent[x]];
      x = parent[x];
    }
    return x;
  }

  for (const [a, b] of edges) {
    const ra = find(a), rb = find(b);
    if (ra === rb) return false; // allaqachon bog'langan → sikl
    parent[ra] = rb;
  }

  return true; // n-1 edge + sikl yo'q → avtomatik bog'langan
}

console.log(validTree(5, [[0,1],[0,2],[0,3],[1,4]])); // true
console.log(validTree(5, [[0,1],[1,2],[2,3],[1,3],[1,4]])); // false (sikl)
```

- **Time:** O(V + E·α(V)).
- **Space:** O(V).

**Izoh:** Asosiy hiyla — agar edge'lar soni `n-1` bo'lsa va sikl bo'lmasa, graf avtomatik bog'langan (connected) bo'ladi. Shuning uchun alohida bog'langanlik tekshiruvi kerak emas.

---

## 8. Rotting Oranges

Grid'da chirigan apelsinlar (`2`) har daqiqa qo'shni yangi apelsinlarni (`1`) chiritadi. Hamma chirishi uchun minimal vaqtni multi-source BFS bilan topamiz.

```js
function orangesRotting(grid) {
  const rows = grid.length, cols = grid[0].length;
  let queue = [];
  let fresh = 0;

  for (let r = 0; r < rows; r++) {
    for (let c = 0; c < cols; c++) {
      if (grid[r][c] === 2) queue.push([r, c]); // boshlang'ich manbalar
      else if (grid[r][c] === 1) fresh++;
    }
  }

  if (fresh === 0) return 0;

  let minutes = 0;
  const dirs = [[1, 0], [-1, 0], [0, 1], [0, -1]];

  while (queue.length && fresh > 0) {
    const next = [];
    for (const [r, c] of queue) {
      for (const [dr, dc] of dirs) {
        const nr = r + dr, nc = c + dc;
        if (nr >= 0 && nc >= 0 && nr < rows && nc < cols && grid[nr][nc] === 1) {
          grid[nr][nc] = 2; // chiriydi
          fresh--;
          next.push([nr, nc]);
        }
      }
    }
    queue = next;
    minutes++;
  }

  return fresh === 0 ? minutes : -1; // qolgan fresh bo'lsa imkonsiz
}

console.log(orangesRotting([[2,1,1],[1,1,0],[0,1,1]])); // 4
console.log(orangesRotting([[2,1,1],[0,1,1],[1,0,1]])); // -1
```

- **Time:** O(R·C).
- **Space:** O(R·C).

**Izoh:** Multi-source BFS — barcha chirigan apelsinlar bir vaqtda boshlanadi (hammasi boshlang'ich queue'da). Har BFS qatlami = 1 daqiqa. Oxirida `fresh > 0` bo'lsa, ba'zi apelsinlar yetib bo'lmaydigan joyda — `-1`.

---

## 9. Pacific Atlantic Water Flow

Suv balandlikdan past/teng katakka oqadi. Ikkala okeanga ham oqib o'ta oladigan kataklarni topamiz. Okeanlardan **teskari** (reverse) DFS qilamiz.

```js
function pacificAtlantic(heights) {
  const rows = heights.length, cols = heights[0].length;
  const pacific = Array.from({ length: rows }, () => new Array(cols).fill(false));
  const atlantic = Array.from({ length: rows }, () => new Array(cols).fill(false));
  const dirs = [[1, 0], [-1, 0], [0, 1], [0, -1]];

  function dfs(r, c, visited, prevHeight) {
    if (r < 0 || c < 0 || r >= rows || c >= cols) return;
    if (visited[r][c]) return;
    if (heights[r][c] < prevHeight) return; // teskari: faqat balandga ko'tarila olamiz
    visited[r][c] = true;
    for (const [dr, dc] of dirs) dfs(r + dr, c + dc, visited, heights[r][c]);
  }

  // Pacific: yuqori va chap chekka; Atlantic: pastki va o'ng chekka
  for (let r = 0; r < rows; r++) {
    dfs(r, 0, pacific, -Infinity);
    dfs(r, cols - 1, atlantic, -Infinity);
  }
  for (let c = 0; c < cols; c++) {
    dfs(0, c, pacific, -Infinity);
    dfs(rows - 1, c, atlantic, -Infinity);
  }

  const result = [];
  for (let r = 0; r < rows; r++) {
    for (let c = 0; c < cols; c++) {
      if (pacific[r][c] && atlantic[r][c]) result.push([r, c]);
    }
  }
  return result;
}

console.log(pacificAtlantic([
  [1,2,2,3,5],
  [3,2,3,4,4],
  [2,4,5,3,1],
  [6,7,1,4,5],
  [5,1,1,2,4],
])); // [[0,4],[1,3],[1,4],[2,2],[3,0],[3,1],[4,0]]
```

- **Time:** O(R·C) — har katak har okean uchun bir marta.
- **Space:** O(R·C).

**Izoh:** To'g'ridan-to'g'ri har katakdan okean tomon oqishni tekshirish O((R·C)²) bo'lardi. Buning o'rniga okean chetidan **teskari** (suv tepaga ketadigan tomonga) DFS qilib, qaysi kataklar har okeanga yeta olishini belgilaymiz, keyin kesishmani olamiz.

---

## 10. Network Delay Time

`k` node'dan signal yuborilganda barcha node'larga yetishi uchun vaqtni Dijkstra bilan topamiz.

```js
function networkDelayTime(times, n, k) {
  const adj = Array.from({ length: n + 1 }, () => []); // 1-indexed
  for (const [u, v, w] of times) adj[u].push([v, w]);

  const dist = new Array(n + 1).fill(Infinity);
  dist[k] = 0;

  // Oddiy massiv-asosli min izlash (kichik n uchun yetarli)
  const pq = [[0, k]]; // [masofa, node]
  while (pq.length) {
    pq.sort((a, b) => a[0] - b[0]); // min-heap simulyatsiyasi
    const [d, node] = pq.shift();
    if (d > dist[node]) continue;
    for (const [nb, w] of adj[node]) {
      if (d + w < dist[nb]) {
        dist[nb] = d + w;
        pq.push([dist[nb], nb]);
      }
    }
  }

  let max = 0;
  for (let i = 1; i <= n; i++) {
    if (dist[i] === Infinity) return -1; // yetib bo'lmaydigan node
    max = Math.max(max, dist[i]);
  }
  return max;
}

console.log(networkDelayTime([[2,1,1],[2,3,1],[3,4,1]], 4, 2)); // 2
```

- **Time:** O(E log E) (haqiqiy heap bilan), bu sodda variantda sort tufayli biroz yuqori.
- **Space:** O(V + E).

**Izoh:** Javob — barcha node'larga minimal masofalarning **eng kattasi** (oxirgi yetib boradigan node). Agar biror node yetib bo'lmas bo'lsa (`Infinity`), signal hamma joyga yetmaydi → `-1`. Production'da `pq.sort` o'rniga to'g'ri binary heap ishlatish kerak.

---

## 11. Cheapest Flights Within K Stops

Ko'pi bilan `k` to'xtash bilan eng arzon parvoz narxini topamiz. Bellman-Ford ning cheklangan versiyasi (k+1 ta relaksatsiya).

```js
function findCheapestPrice(n, flights, src, dst, k) {
  let dist = new Array(n).fill(Infinity);
  dist[src] = 0;

  // Ko'pi bilan k+1 ta qirra orqali (k to'xtash)
  for (let i = 0; i <= k; i++) {
    const temp = [...dist]; // shu raunddagi yangilanishlarni ajratamiz
    for (const [u, v, w] of flights) {
      if (dist[u] !== Infinity && dist[u] + w < temp[v]) {
        temp[v] = dist[u] + w;
      }
    }
    dist = temp;
  }

  return dist[dst] === Infinity ? -1 : dist[dst];
}

console.log(findCheapestPrice(4, [[0,1,100],[1,2,100],[2,0,100],[1,3,600],[2,3,200]], 0, 3, 1)); // 700
```

- **Time:** O(k · E).
- **Space:** O(V).

**Izoh:** Eng muhim nuqta — har raundda `temp = [...dist]` nusxasi ishlatiladi. Busiz bir raundda bir nechta qirra ketma-ket relaks bo'lib, "k to'xtash" cheklovi buziladi. `temp` har bosqichda aniq bitta qirra qo'shilishini kafolatlaydi.

---

## 12. Redundant Connection

Daraxtga qo'shilgan ortiqcha edge'ni topamiz (sikl hosil qiladigan oxirgi edge). Union-Find bilan.

```js
function findRedundantConnection(edges) {
  const n = edges.length;
  const parent = Array.from({ length: n + 1 }, (_, i) => i); // 1-indexed

  function find(x) {
    while (parent[x] !== x) {
      parent[x] = parent[parent[x]];
      x = parent[x];
    }
    return x;
  }

  for (const [a, b] of edges) {
    const ra = find(a), rb = find(b);
    if (ra === rb) return [a, b]; // ikki uch allaqachon bog'langan → ortiqcha
    parent[ra] = rb;
  }
  return [];
}

console.log(findRedundantConnection([[1,2],[1,3],[2,3]])); // [2,3]
console.log(findRedundantConnection([[1,2],[2,3],[3,4],[1,4],[1,5]])); // [1,4]
```

- **Time:** O(E·α(V)).
- **Space:** O(V).

**Izoh:** Edge'larni tartib bilan qo'shib boramiz. Qaysi edge ikki allaqachon bir to'plamdagi uchni bog'lasa — o'sha sikl hosil qiladi va ortiqcha hisoblanadi. Masala bir nechta javob bo'lsa oxirgisini so'raydi, biz edge'larni tartib bilan kezgani uchun avtomatik to'g'ri javob chiqadi.

---

← [DSA bo'limiga qaytish](../../dsa/README.md)
