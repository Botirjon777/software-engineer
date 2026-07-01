# Concurrency va Thread Loyihalar

Concurrency (bir vaqtda ko'p ish) va parallelism (bir paytda haqiqiy parallel bajarish) — nazariyada o'qib tushunish qiyin. Ularni **his qilish** uchun kod yozish kerak: race condition qanday paydo bo'lishini o'z ko'zingiz bilan ko'rish, thread pool nega kerakligini sezish, backpressure'ni boshdan kechirish.

Bu bo'limdagi loyihalar aynan shu maqsadda — har biri bitta aniq konsepsiyani **amaliyotda** o'rgatadi:
- **Race condition** — ikki thread bir resursga bir vaqtda tegsa, ma'lumot buziladi. Buni ko'rmaguningizcha ishonmaysiz.
- **Thread pool / worker pool** — cheksiz thread ochish mumkin emas; cheklangan "ishchilar" qatlamiga vazifa taqsimlash.
- **Backpressure** — producer consumer'dan tez ishlab chiqarsa, tizim to'lib ketmasligi uchun tormozlash.
- **Parallelism va speedup** — CPU-bound ishni yadrolar bo'ylab bo'lish, va Amdahl qonuni (hamma narsa parallel bo'lmaydi).
- **Synchronization** — mutex, lock, atomic operatsiyalar bilan tartibni saqlash.

Til va vositalar bo'yicha eslatma:
- **Node.js:** `worker_threads` (CPU-bound ish uchun real thread), `cluster` (ko'p process, HTTP scaling uchun), `SharedArrayBuffer` + `Atomics` (thread'lar orasida xotira ulashish). Eslab qoling: Node'ning asosiy event loop'i single-threaded, shuning uchun CPU-bound ish uchun `worker_threads` shart.
- **Go:** `goroutine` + `channel` — concurrency uchun eng qulay model (CSP). Race detektor: `go run -race`.
- **Python:** `threading` (I/O-bound uchun, GIL sababli CPU-bound emas), `multiprocessing` (real parallel CPU uchun), `asyncio` (single-thread concurrency).
- **Java/Rust:** haqiqiy thread'lar; Rust'da `Arc<Mutex<T>>` va compiler data race'ni **kompilyatsiya vaqtida** ushlaydi.

> Maslahat: har loyihada **avval sekin, ketma-ket (sequential) versiya** yozing, keyin uni parallel qiling va **benchmark** oling. Farqni raqamlarda ko'rish — eng yaxshi dars.

## 🟢 Beginner

### 🔹 Parallel File Hasher
- **🎯 Maqsad:** Papkadagi ko'plab faylning hash'ini (SHA-256) hisoblash — lekin fayllarni bittalab emas, **bir nechta worker'da parallel**. Katta papkada sequential vs parallel farqini o'lchaysiz.
- **📚 Nimani o'rganasiz:** `worker_threads` bilan CPU-bound ishni parallel qilish, ishni worker'larga taqsimlash (work distribution), natijalarni yig'ish, thread soni vs tezlik.
- **⚙️ Asosiy funksiyalar:** Papka bo'ylab yurish → fayllarni N ta worker'ga taqsimlash → har worker hash hisoblaydi → natijalar main thread'da yig'iladi → sequential bilan vaqt solishtirish.
- **🛠️ Tech tavsiya:** Node.js `worker_threads`, `crypto` moduli, `os.cpus().length` (optimal worker soni). Muqobil: Go goroutine + `sync.WaitGroup`.
- **🚀 Qo'shimcha:** Progress bar, worker sonini o'zgartirib benchmark grafigi, duplicate fayllarni topish (bir xil hash).
- **Konsepsiya:** parallelism va CPU-bound ishni thread'larga bo'lish — asosiy tushuncha.

### 🔹 Concurrent Web Scraper (Concurrency Limit bilan)
- **🎯 Maqsad:** N ta URL'dan ma'lumot yig'ish, lekin **bir vaqtda hammasini emas** — masalan 5 tadan. Aks holda serverni yiqitasiz yoki bloklanasiz. "Concurrency limit" tushunchasini amalda o'rganasiz.
- **📚 Nimani o'rganasiz:** Concurrency (I/O-bound) vs parallelism farqi, concurrency limiting (semaphore mantiq), Promise pool, error handling ko'p vazifada.
- **⚙️ Asosiy funksiyalar:** URL ro'yxati → bir vaqtda maksimal K ta so'rov → biri tugasa navbatdagisi boshlanadi → natija/xatolarni yig'ish → jami statistika.
- **🛠️ Tech tavsiya:** Node.js `fetch` + `Promise` (bu I/O-bound, thread shart emas!), keyin o'zingiz limit yozing. Muqobil: Python `asyncio` + `Semaphore`.
- **🚀 Qo'shimcha:** Retry + exponential backoff, `robots.txt` hurmat, rate limiting, natijani DB'ga saqlash.
- **Konsepsiya:** concurrency limiting va I/O-bound concurrency (thread'siz ham concurrency bo'lishini tushunish).

### 🔹 Parallel Image Resizer
- **🎯 Maqsad:** Yuzlab rasmni turli o'lchamlarga o'zgartirish — CPU-bound og'ir ish. Uni worker'lar bo'ylab taqsimlab, ko'p yadroni ishlatasiz.
- **📚 Nimani o'rganasiz:** CPU-bound ishni worker pool'ga taqsimlash, main thread'ni bloklamaslik, worker'ga data uzatish (transferable objects), yadrolarni to'liq ishlatish.
- **⚙️ Asosiy funksiyalar:** Rasmlar papkasi → har rasm bir worker'da resize → bir nechta o'lcham (thumbnail, medium, large) → natijani saqlash → progress.
- **🛠️ Tech tavsiya:** Node.js `worker_threads` + `sharp`, yoki Python `multiprocessing.Pool` + Pillow.
- **🚀 Qo'shimcha:** Watch rejim (yangi rasm qo'shilsa avtomatik), format konvertatsiya (webp/avif), watermark qo'shish.
- **Konsepsiya:** CPU-bound ishni parallelizatsiya va event loop'ni bloklashdan qutulish.

## 🟡 Intermediate

### 🔹 O'z Thread Pool / Worker Pool implementatsiyangiz
- **🎯 Maqsad:** Tayyor kutubxonaga tayanmasdan, **o'zingizning worker pool'ingizni** yozish. N ta worker'ni oldindan ochib qo'yasiz, vazifalarni navbatga qo'yasiz, bo'sh worker vazifani oladi. Bu — kutubxonalar ichida nima borligini tushunish.
- **📚 Nimani o'rganasiz:** Worker lifecycle (ochish/qayta ishlatish/yopish), task queue, idle worker boshqaruvi, worker'ni qayta ishlatish (yangidan ochish qimmat), graceful shutdown.
- **⚙️ Asosiy funksiyalar:** `pool.submit(task)` → bo'sh worker'ga yuborish → hamma band bo'lsa navbatga → worker tugatsa keyingi vazifani olish → `pool.shutdown()`.
- **🛠️ Tech tavsiya:** Node.js `worker_threads` + o'z queue, Promise-based API. Muqobil: Go — channel'lar bilan worker pool (klassik pattern).
- **🚀 Qo'shimcha:** Dynamic sizing (yuk oshsa worker qo'shish), priority queue, timeout bilan task cancel, health check.
- **Konsepsiya:** thread pool arxitekturasi va resurs boshqaruvi — cheksiz thread ochmaslik nega muhimligini tushunish.

### 🔹 Job Queue with Concurrency Control (o'z `p-limit`ingiz)
- **🎯 Maqsad:** Mashhur `p-limit`/`p-queue` kutubxonalarini **o'zingiz yozing**. "Bir vaqtda maksimal 3 ta job ishlasin" degan qoidani ta'minlaydigan mexanizm.
- **📚 Nimani o'rganasiz:** Semaphore konsepsiyasi, task queueing, concurrency counter, Promise resolution boshqaruvi, FIFO tartib.
- **⚙️ Asosiy funksiyalar:** `limit(concurrency)` → job qo'shish → aktiv job soni limitdan oshmaydi → biri tugasa navbatdagi boshlanadi → hammasini kutish (`await all`).
- **🛠️ Tech tavsiya:** Sof JavaScript/TypeScript (Promise + counter + queue). DB-backed versiya uchun BullMQ'ni o'rganib solishtiring.
- **🚀 Qo'shimcha:** Priority, retry, delay/scheduling, pause/resume, progress event.
- **Konsepsiya:** semaphore va concurrency control — cheklangan resursga kirishni boshqarish.

### 🔹 Producer-Consumer Buffer (Bounded, Backpressure bilan)
- **🎯 Maqsad:** Producer ma'lumot ishlab chiqaradi, consumer iste'mol qiladi — orada **cheklangan buffer**. Buffer to'lsa producer to'xtaydi (backpressure). Bu — stream'lar va queue'lar ortidagi asosiy pattern.
- **📚 Nimani o'rganasiz:** Bounded buffer, backpressure (to'lganda tormozlash), blocking vs non-blocking, producer/consumer sinxronizatsiyasi, buffer to'lsa/bo'shalsa signal berish.
- **⚙️ Asosiy funksiyalar:** Buffer (maks N element) → producer push (to'lsa kutadi) → consumer pull (bo'sh bo'lsa kutadi) → tez producer + sekin consumer ssenariysi → backpressure ko'rsatish.
- **🛠️ Tech tavsiya:** Node.js — `SharedArrayBuffer` + `Atomics.wait/notify` (real blocking!), yoki async queue. Muqobil: Go buffered channel (`make(chan T, N)`) — tabiiy backpressure.
- **🚀 Qo'shimcha:** Ko'p producer/consumer, metrika (buffer to'ldirilishi grafigi), drop policy (to'lsa eskini tashlash).
- **Konsepsiya:** backpressure va bounded buffer — real oqim tizimlarining yuragi.

### 🔹 Rate Limiter (Token Bucket, Concurrent)
- **🎯 Maqsad:** "Sekundiga maksimal 100 so'rov" qoidasini ta'minlaydigan rate limiter — bir nechta thread/so'rov bir vaqtda kelganda ham to'g'ri ishlashi kerak (concurrent-safe).
- **📚 Nimani o'rganasiz:** Token bucket / leaky bucket algoritmi, atomic operatsiyalar (counter'ni xavfsiz yangilash), concurrent access, race condition token hisoblashda.
- **⚙️ Asosiy funksiyalar:** Bucket (token to'ldiriladi vaqt bilan) → so'rov kelsa token oladi → token yo'q bo'lsa rad/kutish → concurrent so'rovlarda ham limit buzilmaydi.
- **🛠️ Tech tavsiya:** Node.js `Atomics` + `SharedArrayBuffer` (yoki distributed uchun Redis + Lua script). Muqobil: Go `sync/atomic` yoki `golang.org/x/time/rate`.
- **🚀 Qo'shimcha:** Distributed rate limiter (Redis), sliding window algoritmi, per-user limit, burst qo'llab-quvvatlash.
- **Konsepsiya:** atomic operatsiyalar va concurrent-safe hisoblash — race condition'siz counter boshqarish.

### 🔹 Parallel Merge Sort (Worker'lar bilan)
- **🎯 Maqsad:** Klassik merge sort'ni parallel qilish — massivni bo'laklarga bo'lib, har bo'lakni alohida worker'da saralab, keyin birlashtirish. Divide-and-conquer'ning tabiiy parallelizmi.
- **📚 Nimani o'rganasiz:** Parallel divide-and-conquer, ishni rekursiv taqsimlash, worker'lar orasida data uzatish narxi, qachon parallel foyda beradi (kichik massivda sequential tezroq).
- **⚙️ Asosiy funksiyalar:** Massivni bo'lish → har bo'lakni worker'da sort → parallel natijalarni merge → threshold (kichik bo'lsa sequential) → benchmark.
- **🛠️ Tech tavsiya:** Node.js `worker_threads` + `SharedArrayBuffer` (nusxa ko'chirmaslik uchun). Muqobil: Go goroutine + channel.
- **🚀 Qo'shimcha:** Turli thread sonida speedup grafigi, quicksort bilan solishtirish, tashqi sort (fayl juda katta bo'lsa).
- **Konsepsiya:** parallel divide-and-conquer va data transfer overhead — parallellik doim tez emasligini tushunish.

### 🔹 Map-Reduce Mini Framework
- **🎯 Maqsad:** Katta ma'lumotni parallel qayta ishlashning MapReduce modelini kichraytirilgan holda qurish. "Map" bosqichi parallel, "Reduce" natijalarni yig'adi. Word count klassik misol.
- **📚 Nimani o'rganasiz:** Map (parallel transform) va reduce (aggregation) fazalari, ishni chunk'larga bo'lish, natijalarni birlashtirish, partitioning.
- **⚙️ Asosiy funksiyalar:** Kirish ma'lumot → chunk'larga bo'lish → har chunk'ga `map` (parallel worker'da) → `reduce` bilan yig'ish → misol: katta fayllarda word/URL count.
- **🛠️ Tech tavsiya:** Node.js `worker_threads` pool, generic `map`/`reduce` API. Muqobil: Python `multiprocessing.Pool.map`.
- **🚀 Qo'shimcha:** Shuffle bosqichi (kalit bo'yicha guruhlash), ko'p bosqichli pipeline, streaming input (fayl xotiraga sig'masa).
- **Konsepsiya:** parallel data processing modeli va map/reduce fazalar — katta data tizimlari (Spark/Hadoop) ortidagi g'oya.

## 🔴 Advanced

### 🔹 Multi-threaded HTTP Load Tester (mini-`wrk`)
- **🎯 Maqsad:** Serverga yuk berib, uni sinaydigan asbob — `wrk`/`k6` klon. Bir vaqtda minglab so'rov yuborib, RPS (requests/sec), latency percentile (p50/p99) o'lchash.
- **📚 Nimani o'rganasiz:** Ko'p worker/connection boshqarish, concurrent HTTP so'rovlar, latency o'lchash va aggregation (histogram), coordinated omission muammosi, resurs cheklovi.
- **⚙️ Asosiy funksiyalar:** `loadtest --connections=100 --duration=30s URL` → N ta parallel connection → so'rov yuborish sikli → latency yig'ish → hisobot (RPS, p50/p90/p99, error rate).
- **🛠️ Tech tavsiya:** Node.js `worker_threads` (har worker bir necha connection) + `http` moduli. Muqobil: Go goroutine (bu ish uchun ideal).
- **🚀 Qo'shimcha:** Ramp-up (yukni asta oshirish), turli scenario (POST + body), real-time live grafik, HdrHistogram bilan aniq percentile.
- **Konsepsiya:** parallelism yuk generatsiyada, concurrent connection boshqarish va aniq latency o'lchash.

### 🔹 Concurrent In-Memory Key-Value Store (Locking bilan)
- **🎯 Maqsad:** Redis'ning kichik versiyasi — xotirada key-value saqlash, lekin **bir nechta thread bir vaqtda** get/set qilishi kerak. Bu yerda locking va race condition'ni jiddiy o'rganasiz.
- **📚 Nimani o'rganasiz:** Locking strategiyalari (global lock vs sharded/striped lock), read-write lock, race condition ma'lumot buzilishi, lock contention va uni kamaytirish, atomic operatsiyalar.
- **⚙️ Asosiy funksiyalar:** `get`/`set`/`delete` → concurrent kirishni lock bilan himoya → TTL/expiry → concurrent testda ma'lumot buzilmasligi → global lock vs sharded lock benchmark.
- **🛠️ Tech tavsiya:** Node.js `worker_threads` + `SharedArrayBuffer` + `Atomics` (lock uchun). Muqobil: Go `sync.RWMutex` + sharded map, yoki Rust `RwLock`.
- **🚀 Qo'shimcha:** LRU eviction, persistence (AOF/snapshot), pub/sub, TCP protokol (haqiqiy client ulanadi), lock-free struktura.
- **Konsepsiya:** synchronization va locking — bir resursga concurrent kirish, lock contention bilan performance trade-off.

### 🔹 Parallel Matrix Multiplication + Benchmark (Amdahl)
- **🎯 Maqsad:** Katta matritsalarni ko'paytirishni parallel qilish va **thread soni ↔ speedup** munosabatini o'lchash. Amdahl qonunini o'z ko'zingiz bilan ko'rasiz: 2x thread ≠ 2x tezlik.
- **📚 Nimani o'rganasiz:** CPU-bound ishni yadrolarga bo'lish, Amdahl qonuni (serial qism speedup'ni cheklaydi), cache locality, false sharing, optimal thread soni topish.
- **⚙️ Asosiy funksiyalar:** Matritsani qatorlar bo'yicha worker'larga bo'lish → parallel ko'paytirish → natija yig'ish → 1,2,4,8... thread'da vaqt o'lchash → speedup grafigi.
- **🛠️ Tech tavsiya:** Node.js `worker_threads` + `SharedArrayBuffer` (matritsa ulashish). Muqobil: Go goroutine, yoki Python `multiprocessing` (NumPy bilan solishtirish).
- **🚀 Qo'shimcha:** Block/tiled multiplication (cache uchun), false sharing effektini ko'rsatish, Amdahl vs Gustafson tahlili, GPU bilan solishtirish.
- **Konsepsiya:** parallelism chegaralari va Amdahl qonuni — parallellik cheksiz tezlik bermasligini raqamlarda isbotlash.

### 🔹 Web Crawler at Scale (Queue + Workers + Dedup)
- **🎯 Maqsad:** Butun saytni (yoki bir necha saytni) chuqur kezib chiqadigan crawler — minglab sahifa. Worker'lar URL queue'dan olib ishlaydi, topilgan yangi linklarni qaytadan queue'ga qo'shadi, ko'rilgan URL'larni takrorlamaydi (dedup).
- **📚 Nimani o'rganasiz:** Work queue + worker pool birgalikda, concurrent dedup (Set'ga xavfsiz kirish — race condition!), politeness (bir domenga limit), graceful shutdown, backpressure queue to'lsa.
- **⚙️ Asosiy funksiyalar:** Seed URL → worker'lar queue'dan oladi → sahifani yuklab link ajratadi → yangi URL'larni dedup qilib queue'ga → domen bo'yicha rate limit → to'plangan ma'lumot.
- **🛠️ Tech tavsiya:** Node.js `worker_threads` + shared queue (yoki Redis queue), `cheerio`. Muqobil: Go goroutine + channel + `sync.Map` dedup.
- **🚀 Qo'shimcha:** Persistent queue (qayta boshlash uchun), depth limit, `robots.txt`, distributed (bir necha mashina), crawl statistikasi dashboard.
- **Konsepsiya:** queue + worker koordinatsiyasi, concurrent dedup va race condition (bir vaqtda ko'rilgan URL'ni ikki worker qo'shmasligi).

### 🔹 Race Condition Demonstrator (Bug → Mutex bilan tuzatish)
- **🎯 Maqsad:** Race condition'ni **ataylab yaratib**, keyin uni **hal qilish**. Masalan bir necha thread umumiy hisoblagichni oshiradi — natija noto'g'ri chiqadi. Keyin lock/atomic qo'shib tuzatasiz. Bu — concurrency'ning eng muhim darsi.
- **📚 Nimani o'rganasiz:** Race condition qanday paydo bo'lishi (non-atomic read-modify-write), critical section, mutex/lock, atomic operatsiya, lost update muammosi, race detektor vositalar.
- **⚙️ Asosiy funksiyalar:** Umumiy counter → N ta worker har biri +1000 → natija kutilgandan kam chiqadi (bug!) → mutex/`Atomics` qo'shib to'g'irlash → oldin/keyin natijani ko'rsatish → bank hisobi transfer misoli (deadlock ham).
- **🛠️ Tech tavsiya:** Node.js `worker_threads` + `SharedArrayBuffer` (`Atomics.add` bilan tuzatish). Muqobil: Go `go run -race` (detektor!), Rust (compiler'ning o'zi to'sadi).
- **🚀 Qo'shimcha:** Deadlock yaratib ko'rsatish (ikki lock, teskari tartib) va oldini olish, livelock, ABA muammosi, turli sinxronizatsiya primitivlarini solishtirish.
- **Konsepsiya:** race condition, critical section va synchronization — barcha concurrency muammolarining ildizini o'z ko'zingiz bilan ko'rish va tuzatish.

← [Loyihalar bo'limiga qaytish](./README.md)
