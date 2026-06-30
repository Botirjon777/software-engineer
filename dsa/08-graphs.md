# Graphs

Graph (graf) — bu vertex'lar (tugunlar) va ularni bog'lovchi edge'lar (qirralar) to'plamidan iborat ma'lumotlar strukturasi. Ijtimoiy tarmoqlar, yo'l xaritalari, bog'liqliklar (dependency) — bularning hammasi graf. Bu hujjatda graf terminologiyasi, ifodalash usullari, traversal (BFS/DFS), cycle detection, topological sort, Dijkstra va Union-Find pattern'larini ko'rib chiqamiz.

## Mundarija

- [Terminologiya](#terminologiya)
- [Ifodalash: adjacency list vs matrix](#ifodalash-adjacency-list-vs-matrix)
- [BFS — kenglik bo'yicha](#bfs--kenglik-boyicha)
- [DFS — chuqurlik bo'yicha](#dfs--chuqurlik-boyicha)
- [Cycle detection](#cycle-detection)
- [Connected components](#connected-components)
- [Topological sort](#topological-sort)
- [Dijkstra](#dijkstra)
- [Union-Find](#union-find)
- [Pattern'lar](#patternlar)
- [Savol-javoblar](#savol-javoblar)
- [Masalalar](#masalalar)

---

## Terminologiya

**💡 Tushuncha:** Graf bilan ishlashda quyidagi atamalarni bilish shart:

- **Vertex (node):** grafdagi tugun.
- **Edge:** ikki vertexni bog'lovchi qirra.
- **Directed (yo'naltirilgan):** edge'lar yo'nalishga ega (A → B, lekin B → A emas). Masalan, Twitter follow.
- **Undirected (yo'naltirilmagan):** edge ikki tomonlama (A — B). Masalan, Facebook do'stlik.
- **Weighted (vaznli):** har bir edge'ning vazni (cost, masofa) bor.
- **Cyclic (siklli):** grafda sikl (boshlanish nuqtasiga qaytadigan yo'l) bor.
- **Acyclic (siklsiz):** sikl yo'q. **DAG** (Directed Acyclic Graph) — yo'naltirilgan siklsiz graf, topological sort uchun asos.
- **Degree:** vertexga ulangan edge'lar soni (directed grafda in-degree va out-degree).

---

## Ifodalash: adjacency list vs matrix

**💡 Tushuncha:** Grafni ikki asosiy usulda saqlash mumkin:

### Adjacency List (qo'shnilar ro'yxati)

Har bir vertex uchun uning qo'shnilari ro'yxati.

```js
// 0 → [1, 2], 1 → [2], 2 → [0, 3], 3 → []
const graph = {
  0: [1, 2],
  1: [2],
  2: [0, 3],
  3: []
};
// yoki Map orqali:
const adj = new Map();
adj.set(0, [1, 2]);
```

### Adjacency Matrix (qo'shnilar matritsasi)

`V × V` o'lchovli matritsa; `matrix[i][j] = 1` agar i va j orasida edge bo'lsa.

```js
const matrix = [
  [0, 1, 1, 0],
  [0, 0, 1, 0],
  [1, 0, 0, 1],
  [0, 0, 0, 0]
];
```

### Trade-off

| Mezon | Adjacency List | Adjacency Matrix |
|-------|---------------|------------------|
| Xotira | O(V + E) | O(V²) |
| Edge bor-yo'qligini tekshirish | O(degree) | O(1) |
| Qo'shnilarni aylanish | O(degree) | O(V) |
| Mos keladi | Siyrak (sparse) graf | Zich (dense) graf |

**⚠️ Ehtiyot bo'l:** Aksariyat real graflar **sparse** (siyrak), shuning uchun adjacency list ko'pincha afzal. Matrix faqat zich graflar yoki edge tekshiruvi tez-tez kerak bo'lganda yaxshi.

---

## BFS — kenglik bo'yicha

**💡 Tushuncha:** BFS (Breadth-First Search) — grafni "qatlam-qatlam" (level by level) aylanadi. **Queue** ishlatiladi. Vaznsiz grafda **eng qisqa yo'l** (shortest path) topishda ideal.

```js
function bfs(graph, start) {
  const visited = new Set([start]);
  const queue = [start];
  const order = [];

  while (queue.length > 0) {
    const node = queue.shift();
    order.push(node);
    for (const neighbor of graph[node]) {
      if (!visited.has(neighbor)) {
        visited.add(neighbor);   // queue'ga qo'shganda belgilash (muhim!)
        queue.push(neighbor);
      }
    }
  }
  return order;
}
// Time: O(V + E), Space: O(V)
```

**⚠️ Ehtiyot bo'l:** `visited`ga node'ni queue'ga **qo'shgan** paytda belgilang, pop qilgan paytda emas. Aks holda bir node bir necha marta queue'ga tushib, takror qayta ishlanadi.

Vaznsiz grafda eng qisqa masofa (level) ham BFS bilan topiladi:

```js
function shortestPath(graph, start, target) {
  const visited = new Set([start]);
  let queue = [start], dist = 0;
  while (queue.length) {
    const next = [];
    for (const node of queue) {
      if (node === target) return dist;
      for (const nb of graph[node]) {
        if (!visited.has(nb)) { visited.add(nb); next.push(nb); }
      }
    }
    queue = next; dist++;
  }
  return -1;
}
```

---

## DFS — chuqurlik bo'yicha

**💡 Tushuncha:** DFS (Depth-First Search) — bir yo'nalish bo'yicha imkon qadar chuqur ketadi, so'ng orqaga qaytadi (backtrack). **Stack** yoki **recursion** orqali amalga oshiriladi.

### Rekursiv DFS

```js
function dfs(graph, start, visited = new Set()) {
  visited.add(start);
  const order = [start];
  for (const neighbor of graph[start]) {
    if (!visited.has(neighbor)) {
      order.push(...dfs(graph, neighbor, visited));
    }
  }
  return order;
}
// Time: O(V + E), Space: O(V) (recursion stack)
```

### Iterativ DFS (stack bilan)

```js
function dfsIterative(graph, start) {
  const visited = new Set();
  const stack = [start];
  const order = [];
  while (stack.length > 0) {
    const node = stack.pop();
    if (visited.has(node)) continue;
    visited.add(node);
    order.push(node);
    for (const neighbor of graph[node]) {
      if (!visited.has(neighbor)) stack.push(neighbor);
    }
  }
  return order;
}
// Time: O(V + E), Space: O(V)
```

---

## Cycle detection

**💡 Tushuncha:** Sikl (cycle) borligini aniqlash directed va undirected graflarda farq qiladi.

### Undirected grafda

DFS paytida, agar qo'shni allaqachon visited bo'lsa **va u parent bo'lmasa** — sikl bor.

```js
function hasCycleUndirected(graph, n) {
  const visited = new Set();
  function dfs(node, parent) {
    visited.add(node);
    for (const nb of graph[node]) {
      if (!visited.has(nb)) {
        if (dfs(nb, node)) return true;
      } else if (nb !== parent) {
        return true; // parent'dan boshqa visited node → sikl
      }
    }
    return false;
  }
  for (let i = 0; i < n; i++) {
    if (!visited.has(i) && dfs(i, -1)) return true;
  }
  return false;
}
```

### Directed grafda (uchta rang / state)

`0 = unvisited`, `1 = visiting (rekursiya stack'da)`, `2 = done`. Agar `visiting` holatdagi node'ga qaytsak — sikl.

```js
function hasCycleDirected(graph, n) {
  const state = new Array(n).fill(0);
  function dfs(node) {
    state[node] = 1; // visiting
    for (const nb of graph[node]) {
      if (state[nb] === 1) return true;       // orqaga edge → sikl
      if (state[nb] === 0 && dfs(nb)) return true;
    }
    state[node] = 2; // done
    return false;
  }
  for (let i = 0; i < n; i++) {
    if (state[i] === 0 && dfs(i)) return true;
  }
  return false;
}
```

---

## Connected components

**💡 Tushuncha:** Connected component — graf ichida bir-biriga bog'langan vertex'lar guruhi. Har bir ko'rilmagan node'dan DFS/BFS boshlab, komponentlar sonini sanaymiz.

```js
function countComponents(n, edges) {
  const adj = Array.from({ length: n }, () => []);
  for (const [a, b] of edges) { adj[a].push(b); adj[b].push(a); }

  const visited = new Set();
  let count = 0;
  function dfs(node) {
    visited.add(node);
    for (const nb of adj[node]) if (!visited.has(nb)) dfs(nb);
  }
  for (let i = 0; i < n; i++) {
    if (!visited.has(i)) { count++; dfs(i); }
  }
  return count;
}
// Time: O(V + E), Space: O(V)
```

---

## Topological sort

**💡 Tushuncha:** Topological sort — DAG'dagi vertex'larni shunday chiziqli tartiblashki, har bir edge `u → v` uchun `u` doim `v`dan oldin keladi. Bog'liqliklarni (dependency) hal qilishda ishlatiladi (build order, course schedule). Faqat **DAG** uchun mavjud.

### Kahn's algoritmi (BFS, in-degree)

```js
function topoSortKahn(n, edges) {
  const adj = Array.from({ length: n }, () => []);
  const inDegree = new Array(n).fill(0);
  for (const [u, v] of edges) { adj[u].push(v); inDegree[v]++; }

  const queue = [];
  for (let i = 0; i < n; i++) if (inDegree[i] === 0) queue.push(i);

  const order = [];
  while (queue.length) {
    const node = queue.shift();
    order.push(node);
    for (const nb of adj[node]) {
      if (--inDegree[nb] === 0) queue.push(nb);
    }
  }
  // Agar order.length < n bo'lsa, grafda sikl bor (topo sort mumkin emas)
  return order.length === n ? order : [];
}
// Time: O(V + E), Space: O(V)
```

### DFS-based topo sort

```js
function topoSortDFS(n, edges) {
  const adj = Array.from({ length: n }, () => []);
  for (const [u, v] of edges) adj[u].push(v);

  const visited = new Array(n).fill(0);
  const stack = [];
  function dfs(node) {
    visited[node] = 1;
    for (const nb of adj[node]) if (!visited[nb]) dfs(nb);
    stack.push(node); // tugagandan keyin qo'shamiz
  }
  for (let i = 0; i < n; i++) if (!visited[i]) dfs(i);
  return stack.reverse(); // teskari post-order
}
```

**⚠️ Ehtiyot bo'l:** Kahn's algoritmi sikl borligini avtomatik aniqlaydi (agar `order.length < n` bo'lsa). DFS-versiyada esa sikl tekshiruvini alohida qo'shish kerak.

---

## Dijkstra

**💡 Tushuncha:** Dijkstra — vaznli (**manfiy vaznsiz**) grafda bitta manbadan boshqa barcha vertex'largacha eng qisqa yo'lni topadi. Min-heap (priority queue) bilan har qadamda eng kichik masofali vertex tanlanadi.

```js
function dijkstra(n, edges, start) {
  const adj = Array.from({ length: n }, () => []);
  for (const [u, v, w] of edges) {
    adj[u].push([v, w]);
    adj[v].push([u, w]); // undirected bo'lsa
  }

  const dist = new Array(n).fill(Infinity);
  dist[start] = 0;
  // MinHeap: [distance, node]
  const pq = new MinHeap((a, b) => a[0] - b[0]);
  pq.push([0, start]);

  while (!pq.isEmpty()) {
    const [d, node] = pq.pop();
    if (d > dist[node]) continue; // eskirgan yozuv
    for (const [nb, w] of adj[node]) {
      if (dist[node] + w < dist[nb]) {
        dist[nb] = dist[node] + w;
        pq.push([dist[nb], nb]);
      }
    }
  }
  return dist;
}
// Time: O((V + E) log V), Space: O(V)
```

**⚠️ Ehtiyot bo'l:** Dijkstra **manfiy vaznli** edge'larda noto'g'ri ishlaydi. Manfiy vazn bo'lsa **Bellman-Ford** (O(VE)) ishlating. `if (d > dist[node]) continue` qatori "lazy deletion" — eskirgan heap yozuvlarini o'tkazib yuboradi.

---

## Union-Find

**💡 Tushuncha:** Union-Find (Disjoint Set Union, DSU) — elementlarni to'plamlarga ajratish va "bu ikki element bir to'plamdami?" savoliga deyarli O(1) javob beruvchi struktura. Cycle detection (undirected) va connected components'da juda samarali.

```js
class UnionFind {
  constructor(n) {
    this.parent = Array.from({ length: n }, (_, i) => i);
    this.rank = new Array(n).fill(0);
    this.count = n; // alohida to'plamlar soni
  }
  find(x) {
    while (this.parent[x] !== x) {
      this.parent[x] = this.parent[this.parent[x]]; // path compression
      x = this.parent[x];
    }
    return x;
  }
  union(x, y) {
    const rx = this.find(x), ry = this.find(y);
    if (rx === ry) return false; // allaqachon bir to'plamda → sikl
    // union by rank
    if (this.rank[rx] < this.rank[ry]) this.parent[rx] = ry;
    else if (this.rank[rx] > this.rank[ry]) this.parent[ry] = rx;
    else { this.parent[ry] = rx; this.rank[rx]++; }
    this.count--;
    return true;
  }
}
// find/union: deyarli O(1) (amortizatsiyalangan, α(n) — inverse Ackermann)
```

---

## Pattern'lar

### Number of Islands

**💡 Tushuncha:** 2D grid'da `'1'` (quruqlik) bloklarini sanaymiz. Har bir ko'rilmagan `'1'`dan DFS/BFS boshlab, butun orolni "cho'ktirib" (belgilab) chiqamiz.

```js
function numIslands(grid) {
  const rows = grid.length, cols = grid[0].length;
  let count = 0;
  function dfs(r, c) {
    if (r < 0 || c < 0 || r >= rows || c >= cols || grid[r][c] !== '1') return;
    grid[r][c] = '0'; // visited
    dfs(r+1, c); dfs(r-1, c); dfs(r, c+1); dfs(r, c-1);
  }
  for (let r = 0; r < rows; r++)
    for (let c = 0; c < cols; c++)
      if (grid[r][c] === '1') { count++; dfs(r, c); }
  return count;
}
// Time: O(R*C), Space: O(R*C)
```

### Course Schedule (cycle detection on DAG)

**💡 Tushuncha:** Kurslarni topological sort orqali tartiblash mumkinmi? Agar sikl bo'lsa — yo'q. Kahn's algoritmi bilan yechamiz.

```js
function canFinish(numCourses, prerequisites) {
  const order = topoSortKahn(numCourses, prerequisites.map(([a, b]) => [b, a]));
  return order.length === numCourses;
}
```

### Clone Graph

**💡 Tushuncha:** Grafni chuqur nusxalash (deep copy). Hash-map orqali asl node → nusxa moslamasini saqlab, DFS/BFS bilan aylanamiz.

```js
function cloneGraph(node) {
  if (!node) return null;
  const map = new Map();
  function dfs(n) {
    if (map.has(n)) return map.get(n);
    const copy = { val: n.val, neighbors: [] };
    map.set(n, copy);
    for (const nb of n.neighbors) copy.neighbors.push(dfs(nb));
    return copy;
  }
  return dfs(node);
}
// Time: O(V + E), Space: O(V)
```

### Word Ladder

**💡 Tushuncha:** Har bir so'z — vertex; bir harf bilan farq qiladigan so'zlar — qo'shnilar. Eng qisqa transformatsiya zanjiri — BFS bilan.

```js
function ladderLength(beginWord, endWord, wordList) {
  const words = new Set(wordList);
  if (!words.has(endWord)) return 0;
  let queue = [beginWord], steps = 1;
  while (queue.length) {
    const next = [];
    for (const word of queue) {
      if (word === endWord) return steps;
      for (let i = 0; i < word.length; i++) {
        for (let c = 97; c <= 122; c++) {
          const cand = word.slice(0, i) + String.fromCharCode(c) + word.slice(i + 1);
          if (words.has(cand)) { words.delete(cand); next.push(cand); }
        }
      }
    }
    queue = next; steps++;
  }
  return 0;
}
// Time: O(N * L^2 * 26), Space: O(N * L)
```

---

## Savol-javoblar

### ❓ BFS va DFS qachon qaysi birini tanlash kerak?

**✅ Javob:** Vaznsiz grafda **eng qisqa yo'l** kerak bo'lsa — BFS (level-by-level). "Bor-yo'qligini" tekshirish, butun grafni aylanish, backtracking yoki topo sort kerak bo'lsa — DFS ko'pincha qulayroq va kam xotira (chuqurlik bo'yicha). Ikkalasi ham O(V + E).

### ❓ Adjacency list va matrix orasidagi asosiy trade-off nima?

**✅ Javob:** List O(V + E) xotira ishlatadi va siyrak graflar uchun ideal, lekin "i–j edge bormi?" tekshiruvi O(degree). Matrix O(V²) xotira, lekin edge tekshiruvi O(1). Real graflar ko'pincha siyrak, shuning uchun list afzal.

### ❓ BFS'da visited'ni qachon belgilash kerak — push paytida emas, pop paytida?

**✅ Javob:** **Push (enqueue) paytida** belgilash kerak. Agar pop paytida belgilasangiz, bitta node bir nechta marta queue'ga tushib, takror ishlanadi va O(V + E) buziladi (eksponensial holatlar ham bo'lishi mumkin).

### ❓ Directed va undirected grafda cycle detection nega farq qiladi?

**✅ Javob:** Undirected'da har bir edge ikki tomonlama, shuning uchun "parent"ga qaytishni siklga sanamaymiz — faqat parent'dan boshqa visited node sikl. Directed'da yo'nalish bor, shuning uchun uchta holat (unvisited/visiting/done) bilan "orqaga edge"ni (rekursiya stack'idagi node) aniqlaymiz.

### ❓ Topological sort qachon mumkin emas?

**✅ Javob:** Faqat **DAG** (Directed Acyclic Graph) uchun mumkin. Agar grafda sikl bo'lsa, topo sort yo'q. Kahn's algoritmida buni `order.length < n` orqali aniqlaymiz — qolgan node'lar sikl ichida bo'lib, in-degree'lari hech qachon 0 bo'lmaydi.

### ❓ Kahn's algoritmi va DFS-based topo sort orasida qaysi biri yaxshi?

**✅ Javob:** Ikkalasi ham O(V + E). Kahn's (BFS, in-degree) sikl tekshiruvini tabiiy beradi va iterativ (stack overflow yo'q). DFS-version rekursiv, chuqur graflarda stack overflow xavfi bor, lekin ba'zan sodda. Amalda Kahn's ko'proq afzal ko'riladi.

### ❓ Dijkstra manfiy vaznli edge'larda nega ishlamaydi?

**✅ Javob:** Dijkstra "bir marta finallashtirilgan vertex masofasi o'zgarmaydi" deb taxmin qiladi (greedy). Manfiy vazn bu taxminni buzadi — keyinroq topilgan yo'l qisqaroq bo'lishi mumkin. Bunday holatda **Bellman-Ford** (O(VE)) yoki manfiy sikl bo'lsa — uni aniqlash kerak.

### ❓ Dijkstra'da `if (d > dist[node]) continue` qatori nima qiladi?

**✅ Javob:** Bu "lazy deletion". Heap'da bir node uchun eskirgan (kattaroq masofali) yozuvlar qolishi mumkin, chunki heap'dan ixtiyoriy elementni o'chirib bo'lmaydi. Pop qilganda agar bu yozuv joriy eng yaxshi masofadan katta bo'lsa — uni o'tkazib yuboramiz.

### ❓ Union-Find'da path compression va union by rank nima beradi?

**✅ Javob:** Path compression — `find` paytida node'larni to'g'ridan-to'g'ri ildizga ulab, daraxtni yassilaydi. Union by rank — kichikroq daraxtni kattaroqqa ulab, balandlikni cheklaydi. Ikkalasi birga `find`/`union`ni deyarli O(1) (amortizatsiyalangan α(n) — inverse Ackermann) qiladi.

### ❓ Cycle detection uchun Union-Find qachon DFS'dan yaxshi?

**✅ Javob:** **Undirected** grafda, ayniqsa edge'lar bittalab kelganda (streaming/online), Union-Find ideal — har bir edge uchun `union` qilamiz, agar ikki uch allaqachon bir to'plamda bo'lsa — sikl. Directed grafda esa Union-Find to'g'ridan-to'g'ri ishlamaydi, DFS (uchta state) kerak.

### ❓ Number of Islands'da grid'ni o'zgartirmasdan qanday yechamiz?

**✅ Javob:** Alohida `visited` 2D array (yoki Set) ishlatamiz va ko'rilgan kataklarni o'sha yerda belgilaymiz. Grid'ni o'zgartirish (in-place `'0'` qilish) xotirani tejaydi, lekin kirish ma'lumotini buzadi — agar buzilmasligi kerak bo'lsa, `visited` ishlatish to'g'ri.

### ❓ Word Ladder'ni nega BFS bilan yechamiz, DFS bilan emas?

**✅ Javob:** Eng **qisqa** transformatsiya zanjiri kerak. BFS level-by-level aylanib, target topilgan birinchi level — eng qisqa masofa. DFS eng qisqani kafolatlamaydi (avval uzun yo'lni topishi mumkin). Bidirectional BFS yanada tezroq.

### ❓ Grafda eng qisqa yo'l: BFS, Dijkstra, Bellman-Ford — qaysi biri?

**✅ Javob:** Vaznsiz (yoki teng vaznli) — **BFS** O(V + E). Manfiy vaznsiz vaznli — **Dijkstra** O((V+E) log V). Manfiy vaznli — **Bellman-Ford** O(VE). Barcha juftlik (all-pairs) — **Floyd-Warshall** O(V³).

### ❓ Clone Graph'da nega hash-map kerak?

**✅ Javob:** Sikl va takroriy node'larni boshqarish uchun. Map asl node → nusxa moslamasini saqlaydi; bir node'ni ikkinchi marta uchratsak, yangi nusxa yaratmasdan mavjud nusxani qaytaramiz. Busiz cheksiz rekursiya (siklda) yoki dublikat node'lar paydo bo'ladi.

### ❓ Disconnected graf bo'lsa, traversal'da nimaga e'tibor berish kerak?

**✅ Javob:** Bitta start node'dan BFS/DFS faqat **bir** komponentni aylanadi. Butun grafni qamrash uchun barcha node'lar bo'ylab tashqi sikl qo'yib, har bir ko'rilmagan node'dan traversal boshlash kerak (connected components yondashuvi).

---

## Masalalar

> Yechimlar: [08-graphs yechimlari](../solutions/dsa/08-graphs.md)

1. **Number of Islands** — 2D grid'da `'1'` (quruqlik) va `'0'` (suv) berilgan; orollar sonini toping.
2. **Clone Graph** — Bog'langan undirected grafning chuqur nusxasini (deep copy) qaytaring.
3. **Course Schedule** — `numCourses` va prerequisite juftliklar berilgan; barcha kurslarni tugatish mumkinmi (sikl yo'qmi)?
4. **Course Schedule II** — Kurslarni tugatish tartibini (topological order) qaytaring; mumkin bo'lmasa bo'sh array.
5. **Word Ladder** — `beginWord`dan `endWord`gacha eng qisqa transformatsiya zanjiri uzunligini toping.
6. **Number of Connected Components in an Undirected Graph** — `n` vertex va edge'lar berilgan; bog'langan komponentlar sonini toping.
7. **Graph Valid Tree** — `n` vertex va edge'lar berilgan; ular yaroqli daraxt (tree) hosil qiladimi?
8. **Rotting Oranges** — Grid'da chirigan apelsinlar har daqiqa qo'shnilarini chiritadi; barcha apelsinlar chirishi uchun minimal vaqt (multi-source BFS).
9. **Pacific Atlantic Water Flow** — Suv ikkala okeanga ham oqib o'ta oladigan kataklarni toping.
10. **Network Delay Time** — `k` node'dan signal yuborilganda barcha node'larga yetishi uchun vaqt (Dijkstra).
11. **Cheapest Flights Within K Stops** — Ko'pi bilan `k` to'xtash bilan eng arzon parvoz narxini toping.
12. **Redundant Connection** — Daraxtga qo'shilgan ortiqcha edge'ni toping (Union-Find).

---

← [DSA bo'limiga qaytish](./README.md)
