# Hash Tables va Sets — Yechimlar

Bu yerda [05-hash-tables.md](../../dsa/05-hash-tables.md) masalalarining to'liq yechimlari berilgan: ishlaydigan kod, complexity tahlili va qisqacha izoh.

## Mundarija

- [1. Two Sum](#1-two-sum)
- [2. Contains Duplicate](#2-contains-duplicate)
- [3. Valid Anagram](#3-valid-anagram)
- [4. Group Anagrams](#4-group-anagrams)
- [5. First Unique Character](#5-first-unique-character)
- [6. Intersection of Two Arrays](#6-intersection-of-two-arrays)
- [7. Subarray Sum Equals K](#7-subarray-sum-equals-k)
- [8. Top K Frequent Elements](#8-top-k-frequent-elements)
- [9. Longest Consecutive Sequence](#9-longest-consecutive-sequence)
- [10. Ransom Note](#10-ransom-note)
- [11. Four Sum II](#11-four-sum-ii)
- [12. LRU Cache](#12-lru-cache)

## 1. Two Sum

Komplement (`target - nums[i]`) ni Map'da qidiramiz. Massivni bir marta aylanamiz.

```js
function twoSum(nums, target) {
  const seen = new Map(); // value -> index
  for (let i = 0; i < nums.length; i++) {
    const need = target - nums[i];
    if (seen.has(need)) return [seen.get(need), i];
    seen.set(nums[i], i);
  }
  return [];
}
```

**Complexity:** Time `O(n)`, Space `O(n)`.

**Izoh:** Brute-force `O(n²)` ni hash qidiruv `O(1)` ga almashtirib `O(n)` ga tushiramiz. Avval qidirib, keyin yozamiz — shunda bir elementni o'zi bilan juftlashtirib qo'ymaymiz.

## 2. Contains Duplicate

Set yordamida har bir elementni avval ko'rganmiz yo'qligini tekshiramiz.

```js
function containsDuplicate(nums) {
  const seen = new Set();
  for (const x of nums) {
    if (seen.has(x)) return true;
    seen.add(x);
  }
  return false;
}
```

**Complexity:** Time `O(n)`, Space `O(n)`.

**Izoh:** Muqobil `new Set(nums).size !== nums.length` — qisqaroq, lekin to'liq massivni doim aylanadi; yuqoridagi versiya birinchi takrorni topishi bilan to'xtaydi.

## 3. Valid Anagram

Chastota hisoblagich: `s` belgilarini sanab, `t` bo'yicha kamaytiramiz.

```js
function isAnagram(s, t) {
  if (s.length !== t.length) return false;
  const count = new Map();
  for (const ch of s) count.set(ch, (count.get(ch) || 0) + 1);
  for (const ch of t) {
    if (!count.has(ch)) return false;
    const c = count.get(ch) - 1;
    if (c === 0) count.delete(ch);
    else count.set(ch, c);
  }
  return count.size === 0;
}
```

**Complexity:** Time `O(n)`, Space `O(1)` (alifbo cheklangan, ko'pi bilan 26 kalit) yoki `O(k)` umumiy holatda.

**Izoh:** Saralashga asoslangan yechim (`O(n log n)`) ham bor, lekin chastota hisoblash tezroq.

## 4. Group Anagrams

Saralangan harflarni kanonik kalit qilamiz.

```js
function groupAnagrams(words) {
  const groups = new Map();
  for (const w of words) {
    const key = [...w].sort().join("");
    if (!groups.has(key)) groups.set(key, []);
    groups.get(key).push(w);
  }
  return [...groups.values()];
}
```

**Complexity:** Time `O(n * k log k)` (`n` so'z, `k` o'rtacha uzunlik), Space `O(n * k)`.

**Izoh:** Kalitni 26 harf chastotasidan yasab (`count[0..25].join("#")`) `O(n * k)` ga tushirish mumkin — saralashdan tezroq.

## 5. First Unique Character

Avval chastotani sanaymiz, keyin birinchi `count === 1` ni topamiz.

```js
function firstUniqChar(s) {
  const count = new Map();
  for (const ch of s) count.set(ch, (count.get(ch) || 0) + 1);
  for (let i = 0; i < s.length; i++) {
    if (count.get(s[i]) === 1) return i;
  }
  return -1;
}
```

**Complexity:** Time `O(n)`, Space `O(1)` (26 harf) yoki `O(k)`.

**Izoh:** Ikki o'tish kerak — biri sanash, biri birinchi noyobni topish uchun. Tartibni saqlash uchun ikkinchi o'tishda asl string bo'ylab yuramiz.

## 6. Intersection of Two Arrays

Birinchi massivni Set qilamiz, ikkinchisini tekshiramiz, natijani ham Set bilan noyob qilamiz.

```js
function intersection(nums1, nums2) {
  const set1 = new Set(nums1);
  const result = new Set();
  for (const x of nums2) {
    if (set1.has(x)) result.add(x);
  }
  return [...result];
}
```

**Complexity:** Time `O(n + m)`, Space `O(n)`.

**Izoh:** Natija Set'i noyoblikni avtomatik ta'minlaydi — bir element ikki marta qo'shilmaydi.

## 7. Subarray Sum Equals K

Prefix sum + Map: har bir prefix uchun `prefix - k` nechta marta uchraganini sanaymiz.

```js
function subarraySum(nums, k) {
  const prefixCount = new Map([[0, 1]]); // bo'sh prefix bir marta
  let sum = 0, result = 0;
  for (const x of nums) {
    sum += x;
    result += prefixCount.get(sum - k) || 0;
    prefixCount.set(sum, (prefixCount.get(sum) || 0) + 1);
  }
  return result;
}
```

**Complexity:** Time `O(n)`, Space `O(n)`.

**Izoh:** `sum[i..j] = prefix[j] - prefix[i-1] = k` bo'lishini istaymiz, ya'ni `prefix[i-1] = prefix[j] - k`. Map shu qiymatdagi prefixlar sonini saqlaydi. `[[0,1]]` boshlang'ich holat — boshidan boshlanadigan subarray'larni hisobga olish uchun.

## 8. Top K Frequent Elements

Chastotani sanab, bucket sort bilan `O(n)` da tartiblaymiz.

```js
function topKFrequent(nums, k) {
  const count = new Map();
  for (const x of nums) count.set(x, (count.get(x) || 0) + 1);

  // bucket[freq] = shu chastotali elementlar
  const buckets = Array.from({ length: nums.length + 1 }, () => []);
  for (const [num, freq] of count) buckets[freq].push(num);

  const result = [];
  for (let f = buckets.length - 1; f >= 0 && result.length < k; f--) {
    for (const num of buckets[f]) {
      result.push(num);
      if (result.length === k) break;
    }
  }
  return result;
}
```

**Complexity:** Time `O(n)`, Space `O(n)`.

**Izoh:** Chastota ko'pi bilan `n` bo'lgani uchun indeks sifatida ishlatib bucket sort qilamiz — bu sortdan (`O(n log n)`) tezroq. Heap bilan ham `O(n log k)` da yechsa bo'ladi.

## 9. Longest Consecutive Sequence

Set'ga solib, faqat zanjir boshidan (`x-1` yo'q bo'lgan elementdan) hisoblaymiz.

```js
function longestConsecutive(nums) {
  const set = new Set(nums);
  let best = 0;
  for (const x of set) {
    if (set.has(x - 1)) continue; // bu zanjir boshi emas
    let length = 1;
    while (set.has(x + length)) length++;
    best = Math.max(best, length);
  }
  return best;
}
```

**Complexity:** Time `O(n)`, Space `O(n)`.

**Izoh:** Har bir zanjir faqat o'z boshidan bir marta sanaladi, shuning uchun ichki `while` jami `O(n)` ishlaydi — natija `O(n)`, saralashsiz (`O(n log n)` dan tez).

## 10. Ransom Note

`magazine` chastotasidan `ransomNote`'ni yasashga harf yetadimi tekshiramiz.

```js
function canConstruct(ransomNote, magazine) {
  const count = new Map();
  for (const ch of magazine) count.set(ch, (count.get(ch) || 0) + 1);
  for (const ch of ransomNote) {
    const c = count.get(ch) || 0;
    if (c === 0) return false;
    count.set(ch, c - 1);
  }
  return true;
}
```

**Complexity:** Time `O(n + m)`, Space `O(1)` (26 harf) yoki `O(k)`.

**Izoh:** Har bir harf bir marta ishlatilishi sababli chastotani kamaytirib boramiz; yetmasa darhol `false`.

## 11. Four Sum II

Birinchi ikki massiv yig'indilarini Map'da sanab, qolgan ikkitasidan komplement qidiramiz.

```js
function fourSumCount(a, b, c, d) {
  const sumAB = new Map();
  for (const x of a)
    for (const y of b)
      sumAB.set(x + y, (sumAB.get(x + y) || 0) + 1);

  let count = 0;
  for (const x of c)
    for (const y of d)
      count += sumAB.get(-(x + y)) || 0;

  return count;
}
```

**Complexity:** Time `O(n²)`, Space `O(n²)`.

**Izoh:** To'rt sikl (`O(n⁴)`) o'rniga ikkiga bo'lamiz: `a+b` yig'indilarini saqlab, `-(c+d)` ni qidiramiz. Bu klassik "meet in the middle" pattern'ining hash table versiyasi.

## 12. LRU Cache

JS'da `Map` insertion order'ni saqlagani uchun undan LRU tartibi sifatida foydalanamiz.

```js
class LRUCache {
  constructor(capacity) {
    this.capacity = capacity;
    this.map = new Map();
  }

  get(key) {
    if (!this.map.has(key)) return -1;
    const value = this.map.get(key);
    this.map.delete(key);     // eng yangi qilib
    this.map.set(key, value); // oxiriga qo'yamiz
    return value;
  }

  put(key, value) {
    if (this.map.has(key)) this.map.delete(key);
    else if (this.map.size >= this.capacity) {
      // eng eski (birinchi) kalitni o'chiramiz
      const oldest = this.map.keys().next().value;
      this.map.delete(oldest);
    }
    this.map.set(key, value);
  }
}
```

**Complexity:** `get` va `put` ikkalasi ham `O(1)`, Space `O(capacity)`.

**Izoh:** `Map.keys().next().value` insertion order'da eng birinchi (eng eski ishlatilgan) kalitni beradi. Har bir foydalanishda kalitni o'chirib qayta qo'shamiz — shunda u "eng yangi" pozitsiyaga o'tadi. Klassik yechim doubly linked list + hash map, lekin JS'da Map bu ishni soddalashtiradi.

---

← [DSA bo'limiga qaytish](../../dsa/README.md)
