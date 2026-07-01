# Software Engineering — Intervyuga Tayyorgarlik (O'zbek tilida)

Dasturchilar uchun intervyuga tayyorgarlik bo'yicha to'liq bilimlar bazasi — **JavaScript va TypeScript** asosida. Har bir mavzu **kerakli tushunchalar + intervyu savollari + javoblar** ko'rinishida yozilgan, ishlaydigan kod misollari bilan.

> Maqsad: **Frontend**, **Backend** va **Database** bo'yicha asoslardan tortib system design darajasigacha tayyorlash.

> Tushuntirishlar o'zbek tilida; kod, kalit so'zlar va standart texnik atamalar (closure, event loop, JOIN va h.k.) ingliz tilida — chunki real intervyularda aynan inglizcha atamalar ishlatiladi.

<p align="left">
  <img src="https://img.shields.io/badge/til-O'zbek-1EB53A?style=flat-square" alt="til" />
  <img src="https://img.shields.io/badge/litsenziya-MIT-blue?style=flat-square" alt="litsenziya" />
  <img src="https://img.shields.io/badge/PR-welcome-brightgreen?style=flat-square" alt="PR welcome" />
  <img src="https://img.shields.io/badge/bo'limlar-16-orange?style=flat-square" alt="bo'limlar" />
</p>

---

## Bu repo nima? (About)

Bu — **o'zbek tilidagi ochiq (open source) dasturchilar bilimlar bazasi va intervyuga tayyorgarlik qo'llanmasi**. Maqsad — o'zbek yoshlariga sifatli, tizimli va bepul ta'lim resursini yetkazish, chunki bunday chuqur materiallar ko'pincha faqat ingliz tilida bo'ladi.

**Kimga mo'ljallangan:**
- Ishga kirishga tayyorlanayotgan junior/mid dasturchilar.
- O'z bilimini tizimlashtirmoqchi bo'lganlar.
- Frontend, Backend, Database, DevOps yoki System Design'ni chuqur o'rganmoqchi bo'lganlar.

**Nima bor:** 16 ta bo'lim — nazariya + intervyu savol-javoblari + yechimsiz masalalar (alohida yechimlar bilan) + amaliy loyiha g'oyalari + deployment qo'llanmalari + ~875 terminlik lug'at.

**Ochiq manba (open source):** bu repo hammaga bepul va ochiq. Uni o'qishingiz, ko'chirishingiz, tarqatishingiz va yaxshilashingiz mumkin — [MIT litsenziyasi](./LICENSE) asosida. Xato topsangiz yoki qo'shimcha qilmoqchi bo'lsangiz — [hissa qo'shing](#hissa-qoshish-contributing)!

---

## Bu repodan qanday foydalanish kerak

- Har bir bo'limning o'z papkasi va `README.md` mundarijasi bor.
- Fayllar o'rganish tartibida raqamlangan.
- Hujjat ichidagi belgilar:
  - **💡 Tushuncha** — bilishingiz shart bo'lgan asosiy g'oya.
  - **❓ Intervyu savoli** — intervyuda so'raladigan ko'rinishda.
  - **✅ Javob** — moslashtirib aytsa bo'ladigan namunaviy javob.
  - **⚠️ Ehtiyot bo'l** — keng tarqalgan xatolar va qo'shimcha savollar.
  - **🧪 Kod** — JS/TS misollari.
- Har bir mavzuning oxirida **## Masalalar** bo'limi bor — yechimlarsiz. Yechimlar alohida [`solutions/`](./solutions/) papkasida joylashgan, shu strukturani takrorlaydi.

---

## 📖 Lug'at (Terminlar)

Ingliz tilidagi texnik terminlarning o'zbekcha ma'nolari — to'g'ridan-to'g'ri tarjima emas, **tushunish uchun izoh** bilan. Har bir bo'limning o'z lug'ati bor (`<bo'lim>/LUGAT.md`), hammasi birlashtirilgan umumiy lug'at esa: **[LUGAT.md](./LUGAT.md)**.

## Dastur (Curriculum)

### 1. [Frontend](./frontend/README.md)
Brauzerda ishlaydigan hamma narsa.

| # | Mavzu | Nimani o'rganasiz |
|---|-------|--------------------|
| 01 | [HTML](./frontend/01-html.md) | Semantika, formalar, accessibility, DOM |
| 02 | [CSS](./frontend/02-css.md) | Box model, flexbox, grid, specificity, responsive |
| 03 | [JavaScript](./frontend/03-javascript.md) | Tiplar, scope, closure, prototype, `this`, async |
| 04 | [TypeScript](./frontend/04-typescript.md) | Tiplar, generics, narrowing, utility types |
| 05 | [React Asoslari](./frontend/05-react-fundamentals.md) | JSX, komponentlar, props, state, rendering |
| 06 | [React Hooks](./frontend/06-react-hooks.md) | Barcha hooklar, qoidalar, custom hooks |
| 07 | [React Performance](./frontend/07-react-performance.md) | Memoization, re-render, profiling, code splitting |
| 08 | [React Arxitektura](./frontend/08-react-architecture.md) | State management, struktura, patternlar |
| 09 | [Frontend System Design](./frontend/09-frontend-system-design.md) | Katta frontend ilovalarni loyihalash |
| 10 | [Next.js](./frontend/10-nextjs.md) | CSR/SSR/SSG/ISR, App Router, React'dan afzalliklari |

### 2. [Backend](./backend/README.md)
Serverda ishlaydigan qism — Node.js orqali.

| # | Mavzu | Nimani o'rganasiz |
|---|-------|--------------------|
| 01 | [Backend nima](./backend/01-what-is-backend.md) | Client/server, so'rov hayot sikli |
| 02 | [Node.js Asoslari](./backend/02-nodejs-fundamentals.md) | Runtime, modullar, V8, libuv |
| 03 | [Event Loop](./backend/03-event-loop.md) | Fazalar, microtask, timerlar, starvation |
| 04 | [Events & EventEmitter](./backend/04-events-eventemitter.md) | Event-driven model |
| 05 | [Streams & Buffers](./backend/05-streams.md) | Readable/writable, backpressure, piping |
| 06 | [Async Patternlar](./backend/06-async-patterns.md) | Callbacks → Promises → async/await |
| 07 | [HTTP & REST API](./backend/07-http-rest-apis.md) | HTTP, REST, status kodlar, dizayn |
| 08 | [Auth & Xavfsizlik](./backend/08-auth-security.md) | Sessions, JWT, OWASP, hashing |
| 09 | [Arxitektura Patternlar](./backend/09-architecture-patterns.md) | Layers, monolith vs microservices, queue |
| 10 | [Backend System Design](./backend/10-backend-system-design.md) | Scaling, caching, load balancing |

### 3. [Database](./databases/README.md)
Ma'lumotlarni saqlash — relational va NoSQL.

| # | Mavzu | Nimani o'rganasiz |
|---|-------|--------------------|
| 01 | [Database Asoslari](./databases/01-fundamentals.md) | Nega DB, data modellar, OLTP vs OLAP |
| 02 | [SQL Asoslari](./databases/02-sql-basics.md) | CRUD, join, aggregatsiya, grouping |
| 03 | [SQL Advanced](./databases/03-sql-advanced.md) | Subquery, window functions, CTE |
| 04 | [Transactions & ACID](./databases/04-transactions-acid.md) | Isolation levels, locking, MVCC |
| 05 | [Indexing & Performance](./databases/05-indexing-performance.md) | B-tree, query plan, optimizatsiya |
| 06 | [NoSQL](./databases/06-nosql.md) | Document, key-value, column, graph |
| 07 | [Database Design](./databases/07-database-design.md) | Normalizatsiya, modeling, CAP, sharding |

### 4. [Networking](./networking/README.md)
Internet qanday ishlaydi — protokollar, infratuzilma va performance.

| # | Mavzu | Nimani o'rganasiz |
|---|-------|--------------------|
| 01 | [Tarmoq Asoslari](./networking/01-network-fundamentals.md) | OSI/TCP-IP model, packet, encapsulation |
| 02 | [TCP va UDP](./networking/02-tcp-udp.md) | Handshake, reliability, congestion, UDP |
| 03 | [IP, DNS, Port, Socket](./networking/03-ip-dns-sockets.md) | IP/subnet/NAT, DNS resolution, socket |
| 04 | [TLS va HTTPS](./networking/04-tls-https.md) | Handshake, sertifikat, shifrlash |
| 05 | [Proxy va Load Balancing](./networking/05-proxy-load-balancing.md) | Forward/reverse proxy, LB, CDN, gateway |
| 06 | [Latency va Performance](./networking/06-latency-performance.md) | Latency vs throughput, optimizatsiya, tail latency |
| 07 | [Real-time Protokollar](./networking/07-realtime-streaming.md) | WebSocket, SSE, gRPC, HTTP/2, QUIC |
| 08 | [MCP](./networking/08-mcp.md) | Model Context Protocol, AI agent integratsiyasi |

### 5. [Encoding va Hashing](./encoding/README.md)
Ma'lumotni ifodalash va o'zgartirish — kodlash, sanoq sistemalari, hashing.

| # | Mavzu | Nimani o'rganasiz |
|---|-------|--------------------|
| 01 | [Character Encoding](./encoding/01-character-encoding.md) | ASCII, Unicode, UTF-8/16, code point, byte vs char |
| 02 | [Sanoq Sistemalari](./encoding/02-number-systems.md) | Binary, hex, octal, bitwise, two's complement |
| 03 | [Data Encoding](./encoding/03-data-encoding.md) | Base64, hex, URL encoding, encoding vs encryption |
| 04 | [Hashing](./encoding/04-hashing.md) | Hash funksiyalar, SHA, HMAC, password hashing, collision |

### 6. [DSA — Data Structures va Algoritmlar](./dsa/README.md)
Texnik intervyuning yadrosi.

| # | Mavzu | Nimani o'rganasiz |
|---|-------|--------------------|
| 01 | [Complexity va Big-O](./dsa/01-complexity-big-o.md) | Time/space complexity, Big-O/Ω/Θ |
| 02 | [Arrays va Strings](./dsa/02-arrays-strings.md) | Two pointers, sliding window |
| 03 | [Linked Lists](./dsa/03-linked-lists.md) | Singly/doubly, fast & slow pointers |
| 04 | [Stacks va Queues](./dsa/04-stacks-queues.md) | Stack, queue, deque, monotonic stack |
| 05 | [Hash Tables va Sets](./dsa/05-hash-tables.md) | Hashing, collision, Map/Set |
| 06 | [Trees va BST](./dsa/06-trees-bst.md) | Traversal, BST, balance |
| 07 | [Heaps](./dsa/07-heaps.md) | Priority queue, top-K |
| 08 | [Graphs](./dsa/08-graphs.md) | BFS, DFS, Dijkstra, topological sort |
| 09 | [Sorting va Searching](./dsa/09-sorting-searching.md) | Merge/quick sort, binary search |
| 10 | [Recursion va Backtracking](./dsa/10-recursion-backtracking.md) | Recursion, backtracking |
| 11 | [Dynamic Programming](./dsa/11-dynamic-programming.md) | Memoization, tabulation |

### 7. [CS Asoslari](./cs-fundamentals/README.md)
Operatsion tizimlar, concurrency, memory.

| # | Mavzu | Nimani o'rganasiz |
|---|-------|--------------------|
| 01 | [Operatsion Tizimlar](./cs-fundamentals/01-operating-systems.md) | Process/thread, scheduling, virtual memory |
| 02 | [Concurrency](./cs-fundamentals/02-concurrency.md) | Race condition, mutex, deadlock, async modellar |
| 03 | [Memory Management](./cs-fundamentals/03-memory-management.md) | Stack vs heap, GC, memory leak |

### 8. [DevOps va Asboblar](./devops/README.md)
Git, Linux, Docker, CI/CD, Cloud.

| # | Mavzu | Nimani o'rganasiz |
|---|-------|--------------------|
| 01 | [Git](./devops/01-git.md) | Branching, merge/rebase, workflow |
| 02 | [Linux va CLI](./devops/02-linux-cli.md) | Buyruqlar, fayl tizimi, process, permissions |
| 03 | [Docker](./devops/03-docker.md) | Image, container, Dockerfile, compose |
| 04 | [CI/CD](./devops/04-ci-cd.md) | Pipeline, automation, deployment |
| 05 | [Cloud](./devops/05-cloud.md) | IaaS/PaaS/SaaS, serverless, scaling |
| 06 | [Kubernetes](./devops/06-kubernetes.md) | Pod, deployment, service, orkestratsiya |

### 9. [Testing](./testing/README.md)
Kodni ishonchli qilish.

| # | Mavzu | Nimani o'rganasiz |
|---|-------|--------------------|
| 01 | [Testing Asoslari](./testing/01-testing-fundamentals.md) | Test pyramid, TDD, mocking |
| 02 | [Amaliyotda Testing](./testing/02-testing-in-practice.md) | Jest/Vitest, RTL, Playwright |

### 10. [OOP, Design Patterns va Clean Code](./oop-patterns/README.md)
Obyektga yo'naltirilgan dizayn va toza kod.

| # | Mavzu | Nimani o'rganasiz |
|---|-------|--------------------|
| 01 | [OOP va SOLID](./oop-patterns/01-oop-solid.md) | Encapsulation/inheritance/polymorphism, SOLID |
| 02 | [Design Patterns](./oop-patterns/02-design-patterns.md) | Creational/structural/behavioral GoF patternlar |
| 03 | [Clean Code](./oop-patterns/03-clean-code.md) | DRY/KISS/YAGNI, refactoring, code smell |

### 11. [System Design](./system-design/README.md)
Distributed systems va case study'lar.

| # | Mavzu | Nimani o'rganasiz |
|---|-------|--------------------|
| 01 | [Asoslar](./system-design/01-fundamentals.md) | Estimation, building blocks, consistency |
| 02 | [Case Study'lar](./system-design/02-case-studies.md) | Twitter, Uber, chat, news feed loyihalash |
| 03 | [Scalability va Reliability](./system-design/03-scalability-reliability.md) | Fault tolerance, HA, resilience patterns |
| 04 | [Distributed Systems](./system-design/04-distributed-systems.md) | Consensus, replication, consistency, CRDT |
| 05 | [Arxitektura Blueprint/Freymvork](./system-design/05-architecture-blueprints.md) | Har qanday tizimni noldan loyihalash |
| 06 | [Hosting va Infratuzilma](./system-design/06-hosting-infrastructure.md) | Cloud, compute/storage, CDN, IaC, multi-region |
| 07 | [Production Misollari](./system-design/07-production-examples.md) | Netflix, Uber, WhatsApp real arxitekturasi |

### 12. [Behavioral va HR Intervyu](./behavioral/README.md)
Intervyuni o'tish ko'nikmalari.

| # | Mavzu | Nimani o'rganasiz |
|---|-------|--------------------|
| 01 | [Behavioral Savollar (STAR)](./behavioral/01-behavioral-star.md) | STAR metodi, keng savollar |
| 02 | [Intervyu Jarayoni](./behavioral/02-interview-process.md) | Yondashuv, kommunikatsiya, muzokara |

### 13. [Hardware — Kompyuter Apparati](./hardware/README.md)
CPU, GPU, xotira va komponentlar qanday ishlaydi va muloqot qiladi.

| # | Mavzu | Nimani o'rganasiz |
|---|-------|--------------------|
| 01 | [Kompyuter Arxitekturasi](./hardware/01-computer-architecture.md) | Von Neumann, fetch-decode-execute |
| 02 | [CPU](./hardware/02-cpu.md) | Core, register, ALU, cache, pipelining |
| 03 | [Xotira Ierarxiyasi](./hardware/03-memory-hierarchy.md) | Register→cache→RAM→disk, locality |
| 04 | [Thread'lar va Multicore](./hardware/04-threads-multicore.md) | Core vs thread, hyperthreading, SMT |
| 05 | [GPU](./hardware/05-gpu.md) | SIMD/SIMT, CUDA, CPU vs GPU |
| 06 | [Bus'lar va Muloqot](./hardware/06-buses-communication.md) | Bus, PCIe, DMA, interrupt, I/O |
| 07 | [Storage](./hardware/07-storage-devices.md) | HDD, SSD, NVMe qanday ishlaydi |

### 14. [Loyihalar (Projects)](./projects/README.md)
Amaliyot orqali o'rganish — daraja bo'yicha qiziq va maqsadli loyihalar.

| # | Yo'nalish | Tavsif |
|---|-----------|--------|
| 01 | [Frontend](./projects/01-frontend-projects.md) | UI, brauzer API, state, performance |
| 02 | [Backend](./projects/02-backend-projects.md) | API, DB, auth, real-time |
| 03 | [Fullstack](./projects/03-fullstack-projects.md) | To'liq mahsulotlar |
| 04 | [Concurrency & Threads](./projects/04-concurrency-projects.md) | Thread, parallel ishlash konsepsiyasi |
| 05 | [Build Your Own X](./projects/05-build-your-own-x.md) | O'z shell/server/DB'ingiz — past daraja |
| 06 | [DevOps, Data va AI](./projects/06-devops-data-ai-projects.md) | Automation, data, LLM/AI |
| 07 | [DSA Masalalar](./projects/07-dsa-problems.md) | LeetCode/NeetCode linklari, pattern bo'yicha |

### 15. [Deployment](./deployment/README.md)
Loyihani internetga chiqarish — amaliy qo'llanma.

| # | Mavzu | Nima o'rganasiz |
|---|-------|-----------------|
| 01 | [Frontend Deploy](./deployment/01-frontend-deploy.md) | Static/React/Next → Vercel/Netlify/Pages |
| 02 | [Domain va DNS](./deployment/02-domains-dns-ssl.md) | Domen olish, DNS, ulash, SSL |
| 03 | [SEO](./deployment/03-seo.md) | Meta, OG, sitemap, robots, Core Web Vitals |
| 04 | [Backend/API Deploy + CORS](./deployment/04-backend-deploy.md) | Railway/Render/Fly.io, CORS |
| 05 | [VPS Asoslari](./deployment/05-vps-fundamentals.md) | SSH, boshqarish, security |
| 06 | [VPS: Node + Nginx + PM2](./deployment/06-vps-nodejs-nginx-pm2.md) | Reverse proxy, PM2, SSL, log |
| 07 | [PHP Deploy](./deployment/07-php-deploy.md) | Shared hosting, LEMP, Laravel |
| 08 | [VPS'da Ma'lumotlar Bazasi](./deployment/08-database-on-vps.md) | Postgres/MySQL o'rnatish, backup |
| 09 | [Log, Monitoring, Security](./deployment/09-logs-monitoring-security.md) | journalctl, PM2 log, firewall, fail2ban |

### 16. [Email](./email/README.md)
Email jo'natish, template va kampaniyalar.

| # | Mavzu | Nima o'rganasiz |
|---|-------|-----------------|
| 01 | [Email Asoslari va Jo'natish](./email/01-email-fundamentals.md) | SMTP, service'lar, SPF/DKIM/DMARC, deliverability |
| 02 | [Template va Kampaniyalar](./email/02-templates-campaigns.md) | HTML email, MJML/React Email, kampaniya, metrikalar |

### 17. [Remote Access va Tunneling](./remote-access/README.md)
Serverga masofadan xavfsiz ulanish va tunneling.

| # | Mavzu | Nima o'rganasiz |
|---|-------|-----------------|
| 01 | [SSH va Tunneling](./remote-access/01-ssh-tunneling.md) | Port forwarding, SOCKS, jump host, reverse tunnel, ngrok |
| 02 | [VPN](./remote-access/02-vpn.md) | WireGuard/OpenVPN/IPSec, split tunneling, o'z VPN |
| 03 | [Remote Desktop](./remote-access/03-remote-desktop.md) | RDP, VNC, X2Go, NoMachine |

---

## Tavsiya etilgan o'rganish tartibi

1. **Avval asoslar**: JS → TS → HTML → CSS.
2. **Yo'nalish tanlang**: Frontend (React) yoki Backend (Node) — yoki ikkalasi.
3. **Database parallel**: SQL asoslarini erta, advanced + design keyinroq.
4. **Networking parallel**: backend bilan birga TCP/DNS/proxy/latency.
5. **System design oxirida**: hammasini bog'laydi.

---

## Masalalar va yechimlar

Har bir mavzu oxirida **Masalalar** bo'limi bor (yechimlarsiz). To'liq yechimlar [`solutions/`](./solutions/) papkasida — avval o'zingiz yechib ko'ring, keyin yechimlardan tekshiring.

---

## Hissa qo'shish (Contributing)

Bu ochiq loyiha — **hissangiz juda qadrli!** Qanday yordam berishingiz mumkin:

- 🐛 **Xato topsangiz** (texnik xato, terish xatosi, noto'g'ri havola) — Issue oching yoki Pull Request yuboring.
- ✍️ **Mavzu qo'shsangiz** yoki mavjudini yaxshilasangiz — bir xil formatga rioya qiling (💡 Tushuncha / ❓ Savol / ✅ Javob / ⚠️ Ehtiyot bo'l + Masalalar).
- 🌐 **Tarjima/til**: tushuntirish o'zbekcha, kod va texnik atamalar inglizcha.
- ⭐ **Foydali bo'lsa** — repoga **star** bering va do'stlaringizga ulashing.

Qanday qilish:
1. Reponi **fork** qiling.
2. Yangi **branch** oching (`git checkout -b mavzu/qoshimcha`).
3. O'zgartirishingizni **commit** qiling.
4. **Pull Request** yuboring — nima va nega o'zgartirganingizni yozing.

## Litsenziya (License)

Bu loyiha [MIT litsenziyasi](./LICENSE) asosida tarqatiladi — bepul foydalanishingiz, ko'chirishingiz, o'zgartirishingiz va tarqatishingiz mumkin. Faqat asl mualliflikni saqlab qoling.

---

_Bu — yashayotgan hujjat. O'zbek dasturchilari uchun ❤️ bilan yaratilgan. Tuzatish va qo'shimchalar uchun ochiq._
