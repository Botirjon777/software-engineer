# Arrays va Strings — Yechimlar

Har masala uchun to'liq ishlaydigan kod, time/space complexity va o'zbekcha izoh. Kod JavaScript/TypeScript'da.

---

## 1. Two Sum (saralanmagan massiv)

Yig'indisi `target` bo'lgan ikki indeksni hash map bilan toping.

```js
function twoSum(nums, target) {
  const seen = new Map(); // qiymat -> indeks
  for (let i = 0; i < nums.length; i++) {
    const need = target - nums[i];
    if (seen.has(need)) return [seen.get(need), i];
    seen.set(nums[i], i);
  }
  return [-1, -1];
}

// twoSum([2, 7, 11, 15], 9) => [0, 1]
// twoSum([3, 2, 4], 6)      => [1, 2]
```

**Complexity:** Time `O(n)`, Space `O(n)`.

**Izoh:** Brute force ikki tsikl bilan `O(n²)` bo'ladi. Hash map yondashuvida har elementni ko'rganda "menga kerakli juftlik" (`target - nums[i]`) ilgari ko'rilganmi deb tekshiramiz. Map'ga elementni **tekshirgandan keyin** qo'shamiz — shunda bitta elementni ikki marta ishlatib qo'ymaymiz. Bir o'tishda yechiladi.

---

## 2. Valid Palindrome

Faqat harf/raqamlarni hisobga olib, registr farqsiz palindromni two pointers bilan tekshiring.

```js
function isPalindrome(s) {
  const isAlnum = (c) => /[a-z0-9]/i.test(c);
  let l = 0, r = s.length - 1;
  while (l < r) {
    while (l < r && !isAlnum(s[l])) l++;   // chapdan keraksiz belgini o'tkazib yubor
    while (l < r && !isAlnum(s[r])) r--;   // o'ngdan keraksiz belgini o'tkazib yubor
    if (s[l].toLowerCase() !== s[r].toLowerCase()) return false;
    l++; r--;
  }
  return true;
}

// isPalindrome("A man, a plan, a canal: Panama") => true
// isPalindrome("race a car")                     => false
```

**Complexity:** Time `O(n)`, Space `O(1)`.

**Izoh:** Two pointers — chetdan markazga. Harf/raqam bo'lmagan belgilarni (bo'shliq, vergul) ikkala tomondan ham o'tkazib yuboramiz. Solishtirishda `toLowerCase()` bilan registrni e'tiborsiz qoldiramiz. Qo'shimcha string yaratmaganimiz uchun xotira `O(1)`.

---

## 3. Valid Anagram

Ikki string anagram ekanini frequency map bilan aniqlang.

```js
function isAnagram(a, b) {
  if (a.length !== b.length) return false;
  const count = new Map();
  for (const ch of a) count.set(ch, (count.get(ch) || 0) + 1);
  for (const ch of b) {
    const c = count.get(ch);
    if (!c) return false;        // yo'q yoki nolga tushgan -> anagram emas
    count.set(ch, c - 1);
  }
  return true;
}

// isAnagram("anagram", "nagaram") => true
// isAnagram("rat", "car")         => false
```

**Complexity:** Time `O(n)`, Space `O(k)` (k — alifbo hajmi, ko'pincha konstanta).

**Izoh:** Anagram — bir xil harflardan tuzilgan ikki string. Uzunliklar farq qilsa, darrov `false`. Birinchi string harflarini sanaymiz, ikkinchisida kamaytiramiz. Agar biror harf yo'q bo'lsa yoki hisobi nolga tushgan bo'lsa — anagram emas. Sortlash usuli ham bor (`O(n log n)`), lekin frequency map tezroq.

---

## 4. Reverse String

String'ni (array sifatida) in-place teskari aylantiring (`O(1)` space).

```js
function reverseString(s) {
  // s — belgilar massivi (LeetCode 344 shartiga ko'ra), in-place o'zgaradi
  let l = 0, r = s.length - 1;
  while (l < r) {
    [s[l], s[r]] = [s[r], s[l]]; // ikki uchni almashtir
    l++; r--;
  }
  return s;
}

// reverseString(['h','e','l','l','o']) => ['o','l','l','e','h']
```

**Complexity:** Time `O(n)`, Space `O(1)`.

**Izoh:** JS string'lari immutable bo'lgani uchun masala belgilar **massivini** beradi va uni joyida o'zgartirishni so'raydi. Two pointers chetdan markazga yurib, har qadamda chap va o'ng elementlarni almashtiradi. Qo'shimcha xotira yo'q.

---

## 5. Maximum Average Subarray

Hajmi `k` bo'lgan subarray'ning eng katta o'rtacha yig'indisini fixed sliding window bilan toping.

```js
function findMaxAverage(nums, k) {
  let windowSum = 0;
  for (let i = 0; i < k; i++) windowSum += nums[i]; // birinchi oyna
  let maxSum = windowSum;
  for (let i = k; i < nums.length; i++) {
    windowSum += nums[i] - nums[i - k]; // yangi kir, eski chiq
    maxSum = Math.max(maxSum, windowSum);
  }
  return maxSum / k;
}

// findMaxAverage([1, 12, -5, -6, 50, 3], 4) => 12.75
```

**Complexity:** Time `O(n)`, Space `O(1)`.

**Izoh:** Fixed sliding window. Har gal oynani noldan qayta sanash `O(n·k)` bo'ladi. O'rniga oynani siljitganda faqat **chiqib ketgan** elementni ayirib, **kirgan** elementni qo'shamiz — `O(1)` yangilanish. Maksimal yig'indini topib, oxirida `k` ga bo'lamiz (eng katta yig'indi = eng katta o'rtacha, chunki `k` doimiy).

---

## 6. Longest Substring Without Repeating Characters

Takrorlanmaydigan eng uzun substring uzunligini variable sliding window bilan toping.

```js
function lengthOfLongestSubstring(s) {
  const lastSeen = new Map(); // belgi -> oxirgi ko'rilgan indeks
  let left = 0, best = 0;
  for (let right = 0; right < s.length; right++) {
    const ch = s[right];
    if (lastSeen.has(ch) && lastSeen.get(ch) >= left) {
      left = lastSeen.get(ch) + 1; // takror — chap chegarani sakrat
    }
    lastSeen.set(ch, right);
    best = Math.max(best, right - left + 1);
  }
  return best;
}

// lengthOfLongestSubstring("abcabcbb") => 3 ("abc")
// lengthOfLongestSubstring("bbbbb")    => 1 ("b")
// lengthOfLongestSubstring("pwwkew")   => 3 ("wke")
```

**Complexity:** Time `O(n)`, Space `O(min(n, alphabet))`.

**Izoh:** Variable sliding window. O'ng pointer doim oldinga yuradi. Takror belgi uchrasa, chap chegarani o'sha belgining **oldingi pozitsiyasidan keyingi** joyga sakratamiz (`Set` bilan birma-bir qisqartirishdan tezroq). `lastSeen.get(ch) >= left` sharti chap chegaradan oldin qolib ketgan eski takrorlarni e'tiborsiz qoldiradi. Har qadamda oyna uzunligini yangilaymiz.

---

## 7. Move Zeroes

Barcha nollarni massiv oxiriga, qolgan tartibni saqlab, in-place suring.

```js
function moveZeroes(nums) {
  let insert = 0; // keyingi noldan-boshqa element joyi
  for (let i = 0; i < nums.length; i++) {
    if (nums[i] !== 0) {
      [nums[insert], nums[i]] = [nums[i], nums[insert]];
      insert++;
    }
  }
  return nums;
}

// moveZeroes([0, 1, 0, 3, 12]) => [1, 3, 12, 0, 0]
```

**Complexity:** Time `O(n)`, Space `O(1)`.

**Izoh:** Read/write pointer texnikasi. `i` (read) butun massivni kezadi; `insert` (write) faqat noldan boshqa element joylashtirilganda oldinga suriladi. Noldan boshqa elementni `insert` pozitsiyasiga almashtiramiz — shunda noldan boshqalar tartibi saqlanib, nollar avtomatik oxiriga to'planadi.

---

## 8. Product of Array Except Self

Har indeks uchun o'zidan boshqa hamma elementlar ko'paytmasini bo'lishsiz (`O(n)`) toping.

```js
function productExceptSelf(nums) {
  const n = nums.length;
  const res = new Array(n).fill(1);
  // 1-o'tish: res[i] = chapdagilar ko'paytmasi (prefix)
  let prefix = 1;
  for (let i = 0; i < n; i++) {
    res[i] = prefix;
    prefix *= nums[i];
  }
  // 2-o'tish: o'ngdagilar ko'paytmasini (suffix) ko'paytirib qo'shamiz
  let suffix = 1;
  for (let i = n - 1; i >= 0; i--) {
    res[i] *= suffix;
    suffix *= nums[i];
  }
  return res;
}

// productExceptSelf([1, 2, 3, 4])  => [24, 12, 8, 6]
// productExceptSelf([-1, 1, 0, -3, 3]) => [0, 0, 9, 0, 0]
```

**Complexity:** Time `O(n)`, Space `O(1)` (natija massividan tashqari).

**Izoh:** Bo'lish ishlatish taqiqlangan (nol bilan bo'lish muammosi ham bor). G'oya: har indeks natijasi = **chapdagi hammasi** ko'paytmasi × **o'ngdagi hammasi** ko'paytmasi. Birinchi o'tishda prefix (chap) ko'paytmalarini yozamiz, ikkinchi o'tishda suffix (o'ng) ko'paytmasini joyida ko'paytiramiz. Alohida prefix/suffix massivlari shart emas — bitta natija massivi yetarli.

---

## 9. Subarray Sum Equals K

Yig'indisi `k` ga teng subarray'lar sonini prefix sum + hash map bilan `O(n)` da sanang.

```js
function subarraySum(nums, k) {
  const seen = new Map([[0, 1]]); // prefix yig'indi -> necha marta uchragan
  let sum = 0, count = 0;
  for (const x of nums) {
    sum += x;
    if (seen.has(sum - k)) count += seen.get(sum - k);
    seen.set(sum, (seen.get(sum) || 0) + 1);
  }
  return count;
}

// subarraySum([1, 1, 1], 2)    => 2
// subarraySum([1, 2, 3], 3)    => 2  ([1,2] va [3])
```

**Complexity:** Time `O(n)`, Space `O(n)`.

**Izoh:** `[i..j]` subarray yig'indisi = `prefix[j] - prefix[i-1]`. Demak yig'indisi `k` bo'lishi uchun `prefix[j] - prefix[i-1] = k`, ya'ni `prefix[i-1] = sum - k`. Har bir prefix yig'indini map'da sanab boramiz; joriy `sum` da `sum - k` qiymati necha marta uchraganini qo'shamiz. `seen` ni `{0: 1}` bilan boshlash bosh qismdan boshlanadigan subarray'larni hisobga oladi.

---

## 10. Group Anagrams

String'lar massivini anagram guruhlariga ajrating.

```js
function groupAnagrams(strs) {
  const groups = new Map(); // kalit (saralangan harflar) -> guruh
  for (const s of strs) {
    const key = s.split('').sort().join(''); // anagramlar uchun bir xil kalit
    if (!groups.has(key)) groups.set(key, []);
    groups.get(key).push(s);
  }
  return [...groups.values()];
}

// groupAnagrams(["eat","tea","tan","ate","nat","bat"])
// => [["eat","tea","ate"], ["tan","nat"], ["bat"]]
```

**Complexity:** Time `O(n · k log k)` (n — so'zlar soni, k — eng uzun so'z), Space `O(n · k)`.

**Izoh:** Anagramlarni guruhlash uchun ularga **bir xil kalit** kerak. Harflarni saralab birlashtirsak (`"eat"` va `"tea"` ikkalasi ham `"aet"` bo'ladi), anagramlar bir kalitga tushadi. Map'da shu kalit bo'yicha yig'amiz. Muqobil: har so'z uchun 26 elementli harf-hisob kalitini ishlatish (`O(n·k)`, saralashsiz).

---

## 11. Minimum Window Substring

`t` ning barcha harflarini o'z ichiga olgan `s` ning eng qisqa substring'ini sliding window bilan toping.

```js
function minWindow(s, t) {
  if (t.length > s.length) return "";
  const need = new Map();
  for (const ch of t) need.set(ch, (need.get(ch) || 0) + 1);

  let required = need.size; // qondirilishi kerak bo'lgan unik harflar soni
  let formed = 0;           // hozir qondirilgan unik harflar soni
  const window = new Map();
  let left = 0, bestLen = Infinity, bestStart = 0;

  for (let right = 0; right < s.length; right++) {
    const ch = s[right];
    window.set(ch, (window.get(ch) || 0) + 1);
    if (need.has(ch) && window.get(ch) === need.get(ch)) formed++;

    while (formed === required) {        // joriy oyna to'liq — qisqartirishga urinamiz
      if (right - left + 1 < bestLen) {
        bestLen = right - left + 1;
        bestStart = left;
      }
      const lch = s[left];
      window.set(lch, window.get(lch) - 1);
      if (need.has(lch) && window.get(lch) < need.get(lch)) formed--;
      left++;
    }
  }
  return bestLen === Infinity ? "" : s.substring(bestStart, bestStart + bestLen);
}

// minWindow("ADOBECODEBANC", "ABC") => "BANC"
// minWindow("a", "a")               => "a"
// minWindow("a", "aa")              => ""
```

**Complexity:** Time `O(|s| + |t|)`, Space `O(|s| + |t|)`.

**Izoh:** Variable sliding window ikkita hisoblagich bilan: `need` — `t` dagi har harf nechta kerakligi, `window` — joriy oynada nechta bor. `formed` oynada qancha unik harf yetarli miqdorda yig'ilganini sanaydi. O'ng pointer oynani kengaytiradi; oyna `t` ni to'liq qoplaganda (`formed === required`) chap pointer bilan iloji boricha qisqartiramiz va eng qisqasini saqlaymiz. Har belgi ko'pi bilan ikki marta ko'riladi → chiziqli.

---

## 12. Trapping Rain Water

Balandliklar massivida tutilgan suv hajmini two pointers bilan `O(n)` time, `O(1)` space da hisoblang.

```js
function trap(height) {
  let l = 0, r = height.length - 1;
  let leftMax = 0, rightMax = 0, water = 0;
  while (l < r) {
    if (height[l] < height[r]) {
      // chap tomon pastroq -> chap ustun suvini chap maksimum aniqlaydi
      leftMax = Math.max(leftMax, height[l]);
      water += leftMax - height[l];
      l++;
    } else {
      rightMax = Math.max(rightMax, height[r]);
      water += rightMax - height[r];
      r--;
    }
  }
  return water;
}

// trap([0,1,0,2,1,0,1,3,2,1,2,1]) => 6
// trap([4,2,0,3,2,5])             => 9
```

**Complexity:** Time `O(n)`, Space `O(1)`.

**Izoh:** Har ustun ustida tutilgan suv = `min(chapdagi eng baland, o'ngdagi eng baland) - shu ustun balandligi`. Two pointers usulida har qadamda **pastroq** tomonni ishlaymiz: agar `height[l] < height[r]` bo'lsa, chap ustun uchun cheklovchi to'siq aniq `leftMax` bo'ladi (chunki o'ngda undan baland nimadir borligi kafolatlangan). Shu sababli prefix/suffix massivlarisiz, bitta o'tishda va `O(1)` xotirada yechiladi.

---

← [DSA bo'limiga qaytish](../../dsa/README.md)
