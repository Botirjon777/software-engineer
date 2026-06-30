# Hash Tables va Sets

Hash table — bu ma'lumotni `key -> value` ko'rinishida saqlaydigan va o'rtacha **O(1)** vaqtda topish, qo'shish hamda o'chirish imkonini beradigan eng muhim data structure'lardan biri. DSA intervyularida masalalarning katta qismi aynan hash table yordamida "tezlashtiriladi": `O(n²)` yechimni `O(n)` ga aylantirish odatda hash table orqali bo'ladi.

Bu hujjatda hash table ichki mexanizmidan tortib (hash function, bucket, collision), JS'dagi `Map`, `Object`, `Set` farqlarigacha va eng ko'p uchraydigan pattern'largacha ko'rib chiqamiz.

> Eslatma: bu yerda gap **DSA hash table** haqida — ya'ni xotirada ma'lumot saqlash strukturasi. Bu **cryptographic hash** (SHA-256, parol hashlash va h.k.) bilan bog'liq emas.

## Mundarija

- [Hash table nima va qanday ishlaydi](#hash-table-nima-va-qanday-ishlaydi)
- [Hash function va bucket](#hash-function-va-bucket)
- [Collision va uni hal qilish](#collision-va-uni-hal-qilish)
- [Load factor va resize](#load-factor-va-resize)
- [JS'da Map vs Object vs Set](#jsda-map-vs-object-vs-set)
- [Complexity: average vs worst](#complexity-average-vs-worst)
- [Pattern'lar](#patternlar)
- [Savol-javoblar](#savol-javoblar)
- [Masalalar](#masalalar)

## Hash table nima va qanday ishlaydi

**💡 Tushuncha:** Hash table — bu kalitni (`key`) son indeksga aylantirib, qiymatni (`value`) massivning shu indeksiga joylashtiradigan struktura. Kalitni indeksga aylantiradigan funksiya — `hash function` deb ataladi.

Asosiy g'oya: oddiy massivda elementni indeks bo'yicha o'qish `O(1)`. Agar biz istalgan kalitni (string, son) tezda indeksga aylantira olsak, demak istalgan kalit bo'yicha qiymatni `O(1)` da topa olamiz.

```
key "ali"  --hash()-->  3   -->  buckets[3] = "ali" qiymati
key "vali" --hash()-->  7   -->  buckets[7] = "vali" qiymati
```

3 ta asosiy operatsiya:

- `set(key, value)` — kalit bo'yicha qiymat yozish
- `get(key)` — kalit bo'yicha qiymat o'qish
- `delete(key)` — kalitni o'chirish

Uchalasi ham o'rtacha **O(1)**.

## Hash function va bucket

**💡 Tushuncha:** `bucket` — bu ichki massivning bitta katakchasi. Hash function kalitni shu massiv indeksiga (bucket'ga) map qiladi.

Yaxshi hash function quyidagi xossalarga ega bo'lishi kerak:

- **Deterministik** — bir xil kalit har doim bir xil natija beradi.
- **Tez** — hisoblanishi `O(1)` (kalit uzunligidan tashqari).
- **Uniform** — natijalar bucket'lar bo'ylab teng taqsimlanadi (to'planib qolmaydi).

Soddalashtirilgan string hash function (faqat tushunish uchun, production emas):

```js
function hash(key, size) {
  let total = 0;
  for (let i = 0; i < key.length; i++) {
    total = (total * 31 + key.charCodeAt(i)) % size;
  }
  return total; // 0 dan size-1 gacha bucket indeksi
}
// Complexity: O(k), bu yerda k = kalit uzunligi
```

`31` — toq tub son bo'lib, taqsimotni yaxshilaydi (Java'da `String.hashCode` ham 31 ishlatadi).

## Collision va uni hal qilish

**💡 Tushuncha:** `collision` (to'qnashuv) — bu ikki xil kalit bir xil bucket indeksiga tushishi. Bu muqarrar (kalitlar cheksiz, bucket'lar cheklangan), shuning uchun har bir hash table collision'ni qandaydir hal qilishi shart.

Ikki asosiy strategiya bor:

### 1. Chaining (zanjirlash)

Har bir bucket'da bitta qiymat emas, balki **ro'yxat** (linked list yoki massiv) saqlanadi. Collision bo'lganda yangi element shu ro'yxatga qo'shiladi.

```
buckets[3] -> [ ["ali", 25], ["bek", 30] ]   // ikkalasi 3-bucketga tushdi
```

`get` paytida bucket ichidagi ro'yxat bo'ylab to'g'ri kalitni qidiramiz.

### 2. Open addressing (ochiq adreslash)

Bucket band bo'lsa, keyingi bo'sh bucket'ga o'tib joylashtiramiz (`linear probing`, `quadratic probing` va h.k.). Bunda har bir bucket'da ko'pi bilan bitta element bo'ladi.

```
hash("ali") = 3, band -> 4 ni tekshir -> bo'sh -> shu yerga yoz
```

**⚠️ Ehtiyot bo'l:** Collision'lar ko'paysa, bucket ichidagi ro'yxat uzayadi yoki probing uzoq davom etadi — natijada operatsiya `O(1)` dan `O(n)` ga yaqinlashadi. Aynan shu sabab worst-case `O(n)`.

## Load factor va resize

**💡 Tushuncha:** `load factor` = `elementlar soni / bucket'lar soni`. U hash table qanchalik "to'lganini" ko'rsatadi.

Load factor oshgan sari collision'lar ham ortadi. Shuning uchun u ma'lum chegaradan (odatda `0.75`) oshganda hash table **resize** qiladi:

1. Bucket'lar massivi 2 baravar kattalashtiriladi.
2. Barcha mavjud elementlar yangi (kattaroq) massivga qaytadan hash qilinadi (`rehash`).

```
load factor 0.75 ga yetdi
-> buckets: 16 dan 32 ga
-> barcha elementlarni qayta joylashtir
```

**⚠️ Ehtiyot bo'l:** Resize — bu `O(n)` operatsiya, chunki hamma element qayta hash qilinadi. Lekin u kamdan-kam sodir bo'lgani uchun **amortized** (taqsimlangan) qiymatda har bir `set` baribir `O(1)` bo'lib qoladi.

## JS'da Map vs Object vs Set

JavaScript'da hash table'ning 3 ta asosiy ko'rinishi bor. Intervyuda "qaysi birini tanlaysan?" degan savol tez uchraydi.

### Object `{}`

```js
const obj = {};
obj["ali"] = 25;
console.log(obj["ali"]); // 25
```

- Kalitlar faqat `string` yoki `symbol` (sonli kalit ham string'ga aylanadi).
- Prototype zanjiri bor — `obj["toString"]` kabi meros bo'lib o'tgan kalitlar bilan to'qnashuv xavfi.
- JSON bilan tabiiy ishlaydi.

### Map

```js
const map = new Map();
map.set("ali", 25);
map.set(42, "son kalit");
map.set(objKey, "obyekt kalit"); // istalgan tip kalit bo'la oladi
console.log(map.get("ali")); // 25
console.log(map.size); // 3
```

- Kalit **istalgan tip** bo'la oladi (object, function, NaN ham).
- Insertion order saqlanadi.
- `.size` xossasi bor, tez iteratsiya qilinadi.
- Tez-tez qo'shish/o'chirish uchun optimallashtirilgan.

### Set

```js
const set = new Set();
set.add(1);
set.add(1); // takror — e'tiborga olinmaydi
set.add(2);
console.log(set.has(1)); // true
console.log(set.size); // 2
```

- Faqat **noyob qiymatlar** to'plami (value -> value, key yo'q).
- Deduplication va "ko'rdimmi?" tekshiruvi uchun ideal.

### Qachon qaysi?

| Vaziyat | Tanlov |
|---|---|
| Kalit-qiymat, kalit string/oddiy, JSON kerak | `Object` |
| Kalit istalgan tip, tez add/delete, order kerak, `size` kerak | `Map` |
| Faqat noyoblik / "bormi?" tekshiruvi | `Set` |

**💡 Tushuncha:** Intervyu masalalarining aksariyatida `Map` yoki `Set` afzal — chunki ular toza, `prototype` muammosi yo'q va `size` bilan ishlash qulay.

## Complexity: average vs worst

| Operatsiya | Average | Worst |
|---|---|---|
| `get` / `has` | O(1) | O(n) |
| `set` / `add` | O(1) | O(n) |
| `delete` | O(1) | O(n) |
| Iteratsiya | O(n) | O(n) |

- **Average O(1):** yaxshi hash function va past load factor sharoitida.
- **Worst O(n):** barcha kalitlar bitta bucket'ga tushib qolsa (yomon hash yoki adversarial input).

Xotira: `O(n)` — saqlanayotgan elementlar soniga proporsional.

## Pattern'lar

### 1. Frequency counter (chastota hisoblagich)

Har bir elementning nechta marta uchraganini sanash:

```js
function frequency(arr) {
  const map = new Map();
  for (const x of arr) {
    map.set(x, (map.get(x) || 0) + 1);
  }
  return map;
}
// Time: O(n), Space: O(n)
```

### 2. Two Sum (komplement qidirish)

Ko'rgan elementlarni saqlab, har biri uchun "kerakli juftlik bormi?" deb tekshiramiz:

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
// Time: O(n), Space: O(n)
```

### 3. Group anagrams (kalit yasash)

Saralangan harflarni kalit sifatida ishlatib guruhlaymiz:

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
// Time: O(n * k log k), Space: O(n * k)
```

### 4. Subarray sum (prefix sum + map)

Yig'indisi `k` ga teng subarray'lar sonini topish:

```js
function subarraySum(nums, k) {
  const prefixCount = new Map([[0, 1]]);
  let sum = 0, result = 0;
  for (const x of nums) {
    sum += x;
    result += prefixCount.get(sum - k) || 0;
    prefixCount.set(sum, (prefixCount.get(sum) || 0) + 1);
  }
  return result;
}
// Time: O(n), Space: O(n)
```

### 5. Deduplication (takrorlarni olib tashlash)

```js
function dedupe(arr) {
  return [...new Set(arr)];
}
// Time: O(n), Space: O(n)
```

### 6. Caching / memoization

Hisoblangan natijalarni `Map`'da saqlab, qayta hisoblamaslik:

```js
function memoize(fn) {
  const cache = new Map();
  return function (n) {
    if (cache.has(n)) return cache.get(n);
    const result = fn(n);
    cache.set(n, result);
    return result;
  };
}
// Lookup: O(1) average
```

## Savol-javoblar

### ❓ Hash table operatsiyalari nega average O(1), worst O(n)?

**✅ Javob:** Average holatda yaxshi hash function elementlarni bucket'lar bo'ylab teng taqsimlaydi, shuning uchun har bir bucket'da juda kam element bo'ladi — topish deyarli to'g'ridan-to'g'ri indeks bo'yicha. Worst holatda esa barcha kalitlar bitta bucket'ga tushadi (collision), va biz `n` ta elementli ro'yxat bo'ylab qidirishga majbur bo'lamiz — bu `O(n)`.

### ❓ Collision nima va nega muqarrar?

**✅ Javob:** Collision — ikki xil kalit bir xil bucket indeksiga map bo'lishi. U muqarrar, chunki mumkin bo'lgan kalitlar soni cheksiz (yoki juda katta), bucket'lar soni esa cheklangan. Pigeonhole printsipi bo'yicha kalitlar bucket'lardan ko'p bo'lsa, hech bo'lmaganda ikkitasi bir bucket'ga tushadi. Shuning uchun chaining yoki open addressing kerak.

### ❓ Chaining va open addressing farqi nimada?

**✅ Javob:** Chaining'da har bir bucket bir ro'yxat saqlaydi, collision bo'lsa element shu ro'yxatga qo'shiladi — bucket'lar soni cheklamaydi. Open addressing'da esa har bir bucket ko'pi bilan bitta element saqlaydi; band bo'lsa boshqa bo'sh bucket qidiriladi (probing). Chaining xotira ko'proq ishlatadi (pointer'lar), lekin to'lganda yaxshiroq ishlaydi; open addressing cache-friendly, lekin load factor yuqori bo'lsa yomonlashadi.

### ❓ Load factor nima va u nima uchun muhim?

**✅ Javob:** Load factor = `elementlar / bucket'lar`. U hash table to'lganlik darajasini ko'rsatadi. Yuqori bo'lsa collision'lar ko'payadi va operatsiyalar sekinlashadi. Shuning uchun u chegaradan (ko'pincha 0.75) oshganda hash table resize qiladi va load factor'ni pasaytiradi.

### ❓ Resize qanday ishlaydi va uning narxi qancha?

**✅ Javob:** Resize'da bucket massivi (odatda 2 baravar) kattalashtiriladi va barcha mavjud elementlar yangi massivga qaytadan hash qilinadi (rehash). Bitta resize `O(n)`, lekin u kamdan-kam bo'lgani uchun amortized hisobda har bir `set` baribir `O(1)`.

### ❓ JS'da Map va Object orasidagi farq nima?

**✅ Javob:** Asosiy farqlar: (1) Map kaliti istalgan tip bo'la oladi, Object kaliti faqat string/symbol; (2) Map insertion order'ni kafolatlaydi; (3) Map'da `.size` bor, Object'da `Object.keys().length` kerak; (4) Object'da prototype zanjiri tufayli meros bo'lib o'tgan kalitlar bilan to'qnashuv xavfi bor; (5) tez-tez add/delete uchun Map optimallashtirilgan.

### ❓ Qachon Object emas, Map ishlatish kerak?

**✅ Javob:** Kalit string'dan boshqa tip bo'lganda (object, son), insertion order kerak bo'lganda, tez-tez qo'shish/o'chirish bo'lganda, dinamik (foydalanuvchidan keladigan) kalitlar ishlatilganda — Map afzal. Object esa qat'iy struktura, JSON serializatsiya yoki oddiy konfiguratsiya uchun yaxshi.

### ❓ Set qachon kerak bo'ladi?

**✅ Javob:** Set faqat noyob qiymatlar muhim bo'lganda — masalan takrorlarni olib tashlash (deduplication), yoki "bu elementni avval ko'rganmidim?" degan tezkor `has` tekshiruvi kerak bo'lganda. Set value -> value xaritalashdir, value/key ajratilmaydi.

### ❓ Frequency counter pattern qanday ishlaydi?

**✅ Javob:** Massivni bir marta aylanib, har bir element uchun Map'da sanog'ini oshiramiz: `map.set(x, (map.get(x) || 0) + 1)`. Natijada har bir elementning chastotasi `O(n)` da hisoblanadi. Bu pattern ko'plab masalalarda (anagram tekshirish, ko'pchilik element, takror topish) asos bo'ladi.

### ❓ Two Sum'ni hash table bilan nega O(n) da yechsa bo'ladi?

**✅ Javob:** Har bir element uchun kerakli juftlik `target - nums[i]` ni Map'dan `O(1)` da qidiramiz. Massivni bir marta aylanamiz, shuning uchun jami `O(n)`. Brute-force ikki sikl `O(n²)` bo'lardi — hash table aynan ikkinchi siklni `O(1)` qidiruvga almashtiradi.

### ❓ Group anagrams'da kalit qanday yasaladi?

**✅ Javob:** Anagram so'zlarning harflari bir xil, faqat tartibi farq qiladi. Shuning uchun har bir so'z harflarini saralab (`[...w].sort().join("")`) kanonik kalit yasaymiz — anagramlar bir xil kalitga ega bo'ladi va bir guruhga tushadi. Muqobil: 26 ta harf chastotasidan kalit yasash (`O(k)`).

### ❓ Subarray sum equals k masalasi nega prefix sum + map talab qiladi?

**✅ Javob:** `sum[i..j] = prefix[j] - prefix[i-1]`. Bizga `prefix[j] - prefix[i-1] = k`, ya'ni `prefix[i-1] = prefix[j] - k`. Har bir `j` da shu qiymatdagi prefix nechta bo'lganini Map'dan `O(1)` qidiramiz. Bu `O(n²)` brute-force'ni `O(n)` ga aylantiradi.

### ❓ Hash table'ni xotira jihatidan qachon yaxshilab o'ylash kerak?

**✅ Javob:** Hash table `O(n)` qo'shimcha xotira ishlatadi. Agar input juda katta bo'lsa va biz faqat oddiy qidiruv qilsak, ba'zan saralash + ikki ko'rsatkich (`O(1)` qo'shimcha xotira) afzalroq bo'ladi. Shuningdek, kalitlar cheklangan diapazonda bo'lsa (masalan 0–25 harflar), Map o'rniga oddiy massiv ishlatish cache-friendly va tezroq.

### ❓ Map'da object kalit ishlatishda qanday tuzoq bor?

**✅ Javob:** Map object'ni **reference** (havola) bo'yicha solishtiradi, qiymat bo'yicha emas. Ya'ni `{a:1}` va boshqa `{a:1}` — ikki xil kalit. Agar mantiqiy tenglik kerak bo'lsa, object'ni avval string'ga (masalan `JSON.stringify`) aylantirib kalit yasash kerak.

### ❓ NaN'ni Map/Set kaliti sifatida ishlatsa bo'ladimi?

**✅ Javob:** Ha. Oddiy `NaN === NaN` `false` bo'lsa-da, Map va Set ichida `SameValueZero` solishtirish ishlatiladi, shuning uchun `NaN` to'g'ri kalit/qiymat bo'la oladi va bitta `NaN` boshqasi bilan tenglashtiriladi.

## Masalalar

> Yechimlar: [solutions/dsa/05-hash-tables.md](../solutions/dsa/05-hash-tables.md)

1. **Two Sum** — massiv va `target` berilgan. Yig'indisi `target` ga teng ikki elementning indekslarini qaytaring. (LeetCode 1)

2. **Contains Duplicate** — massivda biror element ikki marta uchrasa `true`, aks holda `false` qaytaring. (LeetCode 217)

3. **Valid Anagram** — `s` va `t` string berilgan. `t` `s`'ning anagrami ekanini aniqlang. (LeetCode 242)

4. **Group Anagrams** — string'lar massivini anagram guruhlariga ajrating. (LeetCode 49)

5. **First Unique Character** — string'da birinchi takrorlanmaydigan belgi indeksini qaytaring, bo'lmasa `-1`. (LeetCode 387)

6. **Intersection of Two Arrays** — ikki massivning kesishmasini (noyob elementlar) qaytaring. (LeetCode 349)

7. **Subarray Sum Equals K** — yig'indisi `k` ga teng continuous subarray'lar sonini toping. (LeetCode 560)

8. **Top K Frequent Elements** — eng ko'p uchraydigan `k` ta elementni qaytaring. (LeetCode 347)

9. **Longest Consecutive Sequence** — saralanmagan massivda eng uzun ketma-ket sonlar zanjiri uzunligini `O(n)` da toping. (LeetCode 128)

10. **Ransom Note** — `magazine` harflaridan `ransomNote`'ni yasash mumkinmi (har harf bir marta)? (LeetCode 383)

11. **Four Sum II** — to'rt massivdan har biridan bittadan element olib yig'indisi `0` bo'ladigan to'rtliklar sonini toping. (LeetCode 454)

12. **LRU Cache** — `get` va `put` `O(1)` bo'ladigan, sig'imi cheklangan cache yarating (Map + tartib). (LeetCode 146)

---

← [DSA bo'limiga qaytish](./README.md)
