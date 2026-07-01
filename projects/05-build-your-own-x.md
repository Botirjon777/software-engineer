# Build Your Own X — Past Daraja Loyihalar

Har kuni ishlatadigan asboblaringiz — JSON parser, HTTP server, Git, Docker, React — ko'pchilik dasturchi uchun "qora quti" (black box) bo'lib qoladi. Ishlaydi, lekin *qanday* ishlashini hech kim so'ramaydi. Aynan mana shu qora qutilarni ochib, ichini o'z qo'ling bilan qayta qurish — dasturlashni chuqur tushunishning eng kuchli yo'li.

Nega bunday loyihalar eng ko'p o'rgatadi? Chunki sen tutorialdagi qadamlarni ko'chirmaysan — sen *muammoni hal qilasan*. `{{ name }}` ni haqiqiy qiymatga aylantirish uchun stringni qanday parse qilishni o'ylab topishing kerak. HTTP so'rovni qabul qilish uchun TCP socket'dan kelgan xom baytlarni o'zing tushunishing kerak. Har bir "sehr" (magic) aslida oddiy algoritm ekanini ko'rganingda, kutubxonalardan qo'rqmay foydalanadigan, kerak bo'lsa o'zingnikini yozadigan dasturchiga aylanasan.

Bu ro'yxat [codecrafters](https://codecrafters.io) va mashhur ["Build Your Own X"](https://github.com/codecrafters-io/build-your-own-x) ruhida tuzilgan: kichik, lekin haqiqiy narsalarni noldan qurish. Har bir loyihaning yonida u qaysi tushunchani ochishini yozib qo'ydim — shunga qarab tanla.

## 🟢 Beginner

### 🔹 O'z JSON Parser'ing
- **🎯 Maqsad:** JSON string'ni o'qib, uni tilingdagi object/array'ga aylantiruvchi parser yozish (`JSON.parse` ning ichini qurish).
- **📚 Nimani o'rganasiz:** *Ochiladigan tushuncha* — parsing nima ekani. Tokenization (matnni bo'laklarga ajratish) va recursive descent parsing. Grammar (grammatika) tushunchasi.
- **⚙️ Asosiy funksiyalar:** String, number, boolean, null, array, nested object'larni qo'llab-quvvatlash; escape belgilar (`\n`, `\"`); noto'g'ri JSON'da aniq xato xabari (qatorni ko'rsatib).
- **🛠️ Tech tavsiya:** JavaScript / Python / Go — istalgan til. Tashqi kutubxonasiz, faqat string ustida ishla.
- **🚀 Qo'shimcha:** Teskarisini yoz (`JSON.stringify`); xatoda satr va ustun (line/column) raqamini ko'rsat; katta faylni stream qilib parse qil.

### 🔹 Template Engine (`{{ }}`)
- **🎯 Maqsad:** `Salom {{ name }}!` kabi template'ni data bilan to'ldirib, tayyor matn chiqaradigan engine (Handlebars/Mustache mini versiyasi).
- **📚 Nimani o'rganasiz:** *Ochiladigan tushuncha* — HTML frameworklar (view engine) ichida nima bo'layotgani. String interpolation, oddiy syntax parsing, context (o'zgaruvchilar) bilan ishlash.
- **⚙️ Asosiy funksiyalar:** `{{ variable }}` almashtirish; `{{#if}}` shart; `{{#each}}` loop (ro'yxat aylanish); nested object (`{{ user.name }}`).
- **🛠️ Tech tavsiya:** JS yoki Python. Regex bilan boshlab, keyin haqiqiy parser'ga o'sib bor.
- **🚀 Qo'shimcha:** XSS'dan himoya uchun auto-escape; partial'lar (template ichida template); oddiy filter (`{{ name | uppercase }}`).

### 🔹 Markdown → HTML Parser
- **🎯 Maqsad:** Markdown matnni (`# sarlavha`, `**qalin**`, ro'yxatlar) HTML'ga o'giradigan converter yozish.
- **📚 Nimani o'rganasiz:** *Ochiladigan tushuncha* — GitHub va bloglar Markdown'ni qanday render qilishi. Qatorlar bo'yicha (line-based) parsing va inline parsing farqi.
- **⚙️ Asosiy funksiyalar:** Sarlavhalar (`#`–`######`), **bold**/*italic*, linklar, rasm, code block, ordered/unordered ro'yxat, blockquote.
- **🛠️ Tech tavsiya:** Istalgan til. Avval blok elementlarni, keyin inline formatlashni ajratib ishla.
- **🚀 Qo'shimcha:** Code block'da syntax highlighting; jadval (table) qo'llab-quvvatlash; CommonMark spec'ga yaqinlashtirish.

### 🔹 Mini Test Framework (Jest-lite)
- **🎯 Maqsad:** `describe`, `it`, `expect` bilan test yozib, ularni ishga tushiruvchi kichik test framework qurish.
- **📚 Nimani o'rganasiz:** *Ochiladigan tushuncha* — Jest/Mocha ichida `expect(x).toBe(y)` qanday ishlashi. Assertion, test runner, closure va callback tashkil qilish.
- **⚙️ Asosiy funksiyalar:** `describe`/`it` bilan guruhlash; `expect().toBe()`, `.toEqual()`, `.toThrow()` matcher'lar; o'tgan/yiqilgan testlar hisobi; rangli natija (yashil/qizil).
- **🛠️ Tech tavsiya:** JavaScript (Node.js) — o'z ekotizimini takrorlash uchun ideal.
- **🚀 Qo'shimcha:** `beforeEach`/`afterEach` hooklar; async test qo'llab-quvvatlash; oddiy code coverage hisoblagich.

### 🔹 CLI Todo / Task Tool
- **🎯 Maqsad:** Terminaldan vazifa qo'shish, ko'rish, o'chirish va belgilash mumkin bo'lgan buyruq qatori (command-line) dasturi.
- **📚 Nimani o'rganasiz:** *Ochiladigan tushuncha* — `git`, `npm` kabi CLI'lar qanday tuzilishi. Argument parsing, fayl orqali data saqlash (persistence), foydalanuvchi tajribasi (UX) terminalda.
- **⚙️ Asosiy funksiyalar:** `add`, `list`, `done`, `remove` buyruqlari; JSON faylga saqlash; flag'lar (`--priority high`); rangli chiqish.
- **🛠️ Tech tavsiya:** Node.js (commander), Python (argparse/click), yoki Go (cobra).
- **🚀 Qo'shimcha:** Sanaga qarab saralash; due date va eslatma; oddiy tab-completion.

## 🟡 Intermediate

### 🔹 O'z HTTP Server'ing (raw TCP socket ustida)
- **🎯 Maqsad:** Express/Flask'siz, faqat TCP socket ustida ishlaydigan HTTP server yozib, brauzerga sahifa qaytarish.
- **📚 Nimani o'rganasiz:** *Ochiladigan tushuncha* — Express aslida nima ekani. HTTP protokoli xom matn (raw text) ekani, request/response formati, header'lar, status kodlar, socket bilan ishlash.
- **⚙️ Asosiy funksiyalar:** TCP socket'ni ochish va tinglash; HTTP so'rovni parse qilish (method, path, headers, body); response yasash; routing; static fayl berish.
- **🛠️ Tech tavsiya:** Node.js `net` moduli, Python `socket`, yoki Go `net` — HTTP kutubxonasidan foydalanmasdan.
- **🚀 Qo'shimcha:** Bir vaqtda ko'p ulanish (concurrency); keep-alive; oddiy middleware zanjiri; `POST` body va form parse.

### 🔹 Mini Redis (in-memory key-value + TCP protocol)
- **🎯 Maqsad:** `SET`, `GET`, `DEL` buyruqlarini TCP orqali qabul qiladigan in-memory ma'lumotlar ombori (Redis'ning kichik nusxasi).
- **📚 Nimani o'rganasiz:** *Ochiladigan tushuncha* — Redis qanday ishlashi. Wire protocol (RESP), in-memory data structure, TCP server, komanda parsing.
- **⚙️ Asosiy funksiyalar:** RESP protokolni parse qilish; `SET`/`GET`/`DEL`/`EXPIRE`; `redis-cli` bilan ulanib ishlash; TTL (vaqt tugashi).
- **🛠️ Tech tavsiya:** Go yoki Rust (concurrency uchun kuchli), yoki Node.js. codecrafters'da mashhur trek.
- **🚀 Qo'shimcha:** List/Hash data type; pub/sub; diskka saqlash (persistence); bir nechta klient bir vaqtda.

### 🔹 O'z Bundler / Module Resolver
- **🎯 Maqsad:** Bir nechta JS faylni `import`/`require` bog'lanishlariga qarab bitta faylga birlashtiruvchi bundler (Webpack/esbuild mini versiyasi).
- **📚 Nimani o'rganasiz:** *Ochiladigan tushuncha* — Webpack/Vite ichida nima bo'layotgani. Dependency graph qurish, module resolution, kodni birlashtirish (concatenation).
- **⚙️ Asosiy funksiyalar:** Entry fayldan boshlab `import`larni topish; dependency graph qurish; har modulni funksiyaga o'rash; bitta bundle chiqarish.
- **🛠️ Tech tavsiya:** Node.js. AST uchun `@babel/parser` yoki `acorn`.
- **🚀 Qo'shimcha:** Tree-shaking (ishlatilmagan kodni tashlash); source map; oddiy loader (masalan CSS import).

### 🔹 Regex Engine (oddiy)
- **🎯 Maqsad:** `a*b`, `a.c`, `(ab)+` kabi oddiy pattern'larni matnga moslashtiruvchi regex mexanizmini noldan yozish.
- **📚 Nimani o'rganasiz:** *Ochiladigan tushuncha* — regex "sehri" aslida avtomat (state machine) ekani. NFA/DFA, backtracking, pattern'ni ichki strukturaga aylantirish.
- **⚙️ Asosiy funksiyalar:** Literal belgilar; `.` (istalgan belgi); `*`, `+`, `?`; `(...)` guruhlash; `|` yoki; anchor (`^`, `$`).
- **🛠️ Tech tavsiya:** Istalgan til. Thompson NFA algoritmini o'rgan.
- **🚀 Qo'shimcha:** Character class (`[a-z]`); capture group va match natijasini qaytarish; NFA→DFA konversiya.

### 🔹 Shell / Command Interpreter
- **🎯 Maqsad:** Buyruq kiritib, tashqi dasturlarni ishga tushiradigan mini shell (bash'ning kichik nusxasi).
- **📚 Nimani o'rganasiz:** *Ochiladigan tushuncha* — terminal (bash/zsh) ichida nima bo'layotgani. Process yaratish (fork/exec), pipe, redirection, environment o'zgaruvchilari.
- **⚙️ Asosiy funksiyalar:** Buyruqni o'qish va bajarish; argumentlarni parse qilish; `cd`, `exit` kabi built-in'lar; pipe (`|`); input/output redirection (`>`, `<`).
- **🛠️ Tech tavsiya:** C (fork/exec asl tushuncha), Go, yoki Python (`subprocess`). codecrafters'da "Build your own shell" treki bor.
- **🚀 Qo'shimcha:** Background job (`&`); history; tab-completion; oddiy skript ishga tushirish.

### 🔹 Virtual DOM + Diff Algoritmi
- **🎯 Maqsad:** Sahifa holatini virtual daraxt (tree) sifatida saqlab, o'zgarishlarni topib, faqat kerakli DOM'ni yangilaydigan tizim.
- **📚 Nimani o'rganasiz:** *Ochiladigan tushuncha* — React/Vue nega tez ekani. Virtual DOM, diffing (reconciliation), minimal DOM operatsiyalari.
- **⚙️ Asosiy funksiyalar:** JS object bilan DOM'ni ifodalash; render funksiya; ikki daraxtni solishtirib farqni (diff) topish; faqat o'zgargan qismni real DOM'ga qo'llash (patch).
- **🛠️ Tech tavsiya:** Vanilla JavaScript (brauzer DOM API).
- **🚀 Qo'shimcha:** Key'lar bilan ro'yxatni samarali yangilash; event handling; oddiy component tushunchasi.

### 🔹 Promise Implementatsiyasi (A+ spec)
- **🎯 Maqsad:** JavaScript `Promise`'ni noldan, Promises/A+ spesifikatsiyasiga mos qilib yozish.
- **📚 Nimani o'rganasiz:** *Ochiladigan tushuncha* — `then`, `async/await` ichida nima bo'layotgani. Asynchronous kod, callback, microtask navbati, holat mashinasi (pending/fulfilled/rejected).
- **⚙️ Asosiy funksiyalar:** `resolve`/`reject`; `then` zanjiri; xato tarqalishi (error propagation); `then` ichida yana promise qaytarish.
- **🛠️ Tech tavsiya:** JavaScript. Rasmiy Promises/A+ test suite bilan tekshir.
- **🚀 Qo'shimcha:** `catch`, `finally`; `Promise.all`, `Promise.race`; `async/await` bilan qanday bog'lanishini tushun.

## 🔴 Advanced

### 🔹 O'z Dasturlash Tiling (lexer → parser → interpreter)
- **🎯 Maqsad:** Kichik, lekin ishlaydigan dasturlash tili yaratish: kod yozib, natija olish (o'zgaruvchi, funksiya, shart, loop).
- **📚 Nimani o'rganasiz:** *Ochiladigan tushuncha* — barcha tillar qanday ishlashi. Lexer (tokenizatsiya) → parser (AST) → interpreter (bajarish) zanjiri. Scope, closure, evaluation.
- **⚙️ Asosiy funksiyalar:** Lexer (kodni token'larga); parser (AST qurish); interpreter (AST'ni yurgizish); o'zgaruvchi, arifmetika, `if`, `while`, funksiya.
- **🛠️ Tech tavsiya:** Istalgan til. "Crafting Interpreters" (Robert Nystrom) — eng yaxshi qo'llanma.
- **🚀 Qo'shimcha:** Funksiya va closure; xato xabarlari qator raqami bilan; REPL (interaktiv rejim); tur tekshiruvi (type checking).

### 🔹 Mini Git (commit / branch / diff)
- **🎯 Maqsad:** `init`, `add`, `commit`, `log`, `branch` ishlaydigan versiya nazorati tizimi (Git ichki mexanizmi).
- **📚 Nimani o'rganasiz:** *Ochiladigan tushuncha* — Git aslida qanday ishlashi. Content-addressable storage, blob/tree/commit object'lar, SHA hash, DAG (yo'naltirilgan graf).
- **⚙️ Asosiy funksiyalar:** Fayl mazmunini hash qilib blob sifatida saqlash; tree va commit object; `.git` katalog strukturasi; `log`; oddiy `diff`.
- **🛠️ Tech tavsiya:** Go, Python yoki Rust. codecrafters "Build your own Git" treki mavjud.
- **🚀 Qo'shimcha:** `branch` va `merge`; `checkout`; staging area (index); remote'ga oddiy push.

### 🔹 Mini Database (B-tree index + persistence)
- **🎯 Maqsad:** Ma'lumotni diskka saqlab, tez qidiruvchi kichik ma'lumotlar bazasi (SQLite'ning kichik g'oyasi).
- **📚 Nimani o'rganasiz:** *Ochiladigan tushuncha* — database'lar nega tez ekani. B-tree index, disk sahifalari (pages), persistence, oddiy SQL parsing.
- **⚙️ Asosiy funksiyalar:** `INSERT`/`SELECT` buyruqlari; ma'lumotni faylga sahifalar bilan yozish; B-tree bilan indexlash; disk formatini boshqarish.
- **🛠️ Tech tavsiya:** C yoki Rust (past darajali xotira nazorati). "Let's Build a Simple Database" (cstack) qo'llanmasi.
- **🚀 Qo'shimcha:** Transaction va WAL (write-ahead log); ko'p ustunli jadval; oddiy query planner.

### 🔹 Mini Docker (namespaces / cgroups — Linux)
- **🎯 Maqsad:** Jarayonni izolyatsiya qilib, o'z fayl tizimi va resurs chegarasi bilan ishga tushiruvchi kontainer runtime (Docker ichki mexanizmi).
- **📚 Nimani o'rganasiz:** *Ochiladigan tushuncha* — konteyner "sehri" aslida Linux xususiyatlari ekani. Namespaces (izolyatsiya), cgroups (resurs cheklovi), chroot/pivot_root, jarayonlar.
- **⚙️ Asosiy funksiyalar:** Yangi PID/mount/network namespace yaratish; root fayl tizimini almashtirish; cgroups bilan CPU/xotira cheklash; jarayonni izolyatsiyada ishga tushirish.
- **🛠️ Tech tavsiya:** Go (Docker o'zi Go'da) yoki C — faqat Linux'da ishlaydi. "Building a container from scratch" (Liz Rice) ma'ruzasi.
- **🚀 Qo'shimcha:** Oddiy image (tar) qo'llab-quvvatlash; network bridge; volume mount; oddiy `run` CLI.

### 🔹 Mini React (fiber g'oyasi)
- **🎯 Maqsad:** Component, JSX-ga o'xshash render va holat (state) qo'llab-quvvatlaydigan mini UI kutubxona.
- **📚 Nimani o'rganasiz:** *Ochiladigan tushuncha* — React ichki arxitekturasi. Fiber (uzatiladigan ish birligi), reconciliation, hooks (`useState`), render/commit fazalari.
- **⚙️ Asosiy funksiyalar:** `createElement` va render; virtual daraxtni fiber'larga bo'lish; diffing; `useState` bilan qayta render; commit fazasida DOM yangilash.
- **🛠️ Tech tavsiya:** Vanilla JavaScript. "Build your own React" (Rodrigo Pombo — didact) qo'llanmasi.
- **🚀 Qo'shimcha:** `useEffect`; ish uzilishi (interruptible rendering); oddiy event delegation.

### 🔹 Mini Compiler (bytecode + VM)
- **🎯 Maqsad:** Kodni bytecode'ga kompilyatsiya qilib, uni virtual mashinada (VM) bajaradigan tizim (interpreter'dan bir qadam yuqori).
- **📚 Nimani o'rganasiz:** *Ochiladigan tushuncha* — Python/Java virtual mashinasi qanday ishlashi. Bytecode, stack-based VM, compilation vs interpretation, instruction set.
- **⚙️ Asosiy funksiyalar:** Lexer/parser → AST; AST'ni bytecode'ga kompilyatsiya; stack-based VM bajarish; arifmetika, o'zgaruvchi, funksiya chaqiruvi.
- **🛠️ Tech tavsiya:** Go yoki C. "Writing a Compiler in Go" / "Crafting Interpreters" (2-qism, clox).
- **🚀 Qo'shimcha:** Garbage collection; optimizatsiya; disassembler (bytecode'ni ko'rsatish); tezlikni interpreter bilan solishtirish.

### 🔹 Load Balancer (reverse proxy)
- **🎯 Maqsad:** Kelgan so'rovlarni bir nechta backend server orasida taqsimlaydigan reverse proxy / load balancer.
- **📚 Nimani o'rganasiz:** *Ochiladigan tushuncha* — Nginx/HAProxy qanday ishlashi. Reverse proxy, load balancing algoritmlari, health check, concurrency.
- **⚙️ Asosiy funksiyalar:** Bir nechta backend'ni ro'yxatga olish; so'rovni ularga yo'naltirish; round-robin taqsimlash; health check (o'lgan serverni chetlab o'tish).
- **🛠️ Tech tavsiya:** Go (concurrency va `net/http` kuchli) yoki Rust.
- **🚀 Qo'shimcha:** Turli algoritmlar (least-connections, weighted); sticky session; TLS termination; metrikalar.

← [Loyihalar bo'limiga qaytish](./README.md)
