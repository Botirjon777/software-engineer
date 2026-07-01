# Fullstack Loyihalar

To'liq mahsulot qurish — bu dasturchining eng qadrli ko'nikmasi. Bu yerda **frontend + backend + database + deploy** — hammasini birlashtiradigan loyihalar. Ya'ni foydalanuvchi ko'radigan UI, uni orqada quvvatlaydigan API, ma'lumot saqlaydigan DB va oxirida — internetda jonli ishlaydigan **haqiqiy link**.

Har bir loyihada e'tibor beradigan narsalar: **auth** (kim kirgan), **state management** (frontend holati), **API dizayni** (REST/GraphQL), **DB sxemasi** (relations, indexing) va **deploy** (CI/CD, environment variables, monitoring). Portfolioga qo'yiladigan loyiha — bu shunchaki kod emas, balki **jonli, ishlatiladigan mahsulot**.

> Maslahat: har loyihani albatta **deploy** qiling. Vercel/Netlify (frontend), Railway/Fly.io/Render (backend), Neon/Supabase/PlanetScale (DB). Portfolioda "jonli demo" linki bo'lishi 10 ta lokal loyihadan qimmatroq.

## 🟢 Beginner

### 🔹 Link-in-Bio (Linktree klon)
- **🎯 Maqsad:** Instagram/Telegram bio'siga qo'yiladigan bitta sahifa — barcha havolalaringiz (portfolio, GitHub, ijtimoiy tarmoqlar) chiroyli tarzda bir joyda. Real qiymati bor: har bir dasturchiga o'ziga kerak.
- **📚 Nimani o'rganasiz:** CRUD asoslari, user auth (register/login), form validation, responsive design, deploy pipeline.
- **⚙️ Asosiy funksiyalar:** Ro'yxatdan o'tish → dashboard'da havola qo'shish/o'chirish/tartiblash → public profil sahifasi (`/username`) → click statistikasi (nechta bosildi).
- **🛠️ Tech tavsiya:** Next.js + Prisma + PostgreSQL (yoki SQLite boshlash uchun), NextAuth, Tailwind, Vercel deploy.
- **🚀 Qo'shimcha:** Custom domain ulash, tema tanlash (rang/shrift), drag-and-drop bilan tartiblash, QR-kod generatsiya.

### 🔹 Expense Tracker with Charts (Xarajatlar nazorati)
- **🎯 Maqsad:** Pul kirim-chiqimini kuzatadigan va **grafiklar** bilan ko'rsatadigan ilova. Har oyda qayerga qancha ketganini ko'rasiz — real hayotda ishlatiladigan narsa.
- **📚 Nimani o'rganasiz:** Data aggregation (kategoriya bo'yicha yig'ish), chart kutubxonalari, date handling, filtering/sorting, protected routes.
- **⚙️ Asosiy funksiyalar:** Tranzaksiya qo'shish (summa, kategoriya, sana) → oylik/haftalik ko'rinish → pie/bar chart bilan tahlil → kategoriya bo'yicha filter → CSV eksport.
- **🛠️ Tech tavsiya:** React + Recharts/Chart.js, Node/Express yoki Next API routes, PostgreSQL, JWT auth.
- **🚀 Qo'shimcha:** Budjet limiti va ogohlantirish (limitdan oshsa), takrorlanuvchi to'lovlar (subscription), multi-valyuta konvertatsiya.

### 🔹 Habit Tracker (Odat kuzatuvchi)
- **🎯 Maqsad:** Kunlik odatlarni (sport, kitob o'qish, suv ichish) belgilash va **streak** (ketma-ket kunlar) ko'rish. GitHub'ning "contribution graph"iga o'xshash heatmap bilan motivatsiya.
- **📚 Nimani o'rganasiz:** Time-series data, calendar/heatmap UI, streak logikasi, timezone handling, daily reset cron.
- **⚙️ Asosiy funksiyalar:** Odat yaratish → har kuni "bajarildi" belgilash → streak hisoblash → yillik heatmap → statistika (necha % bajardim).
- **🛠️ Tech tavsiya:** SvelteKit yoki Next.js, Drizzle/Prisma, PostgreSQL, Tailwind.
- **🚀 Qo'shimcha:** Push notification (eslatma), do'stlar bilan solishtirish, mobil PWA versiya.

## 🟡 Intermediate

### 🔹 Real-time Collaborative Notes (Notion-lite)
- **🎯 Maqsad:** Bir nechta odam **bir vaqtda** bitta hujjatni tahrirlaydigan ilova — kimdir yozsa boshqalar darhol ko'radi. Google Docs/Notion tajribasini o'zing qurish.
- **📚 Nimani o'rganasiz:** WebSocket, real-time sync, operational transform yoki CRDT asoslari, optimistic UI, conflict resolution, rich-text editor.
- **⚙️ Asosiy funksiyalar:** Hujjat yaratish/ulashish → real-time birgalikda tahrir → kursor pozitsiyalarini ko'rsatish → nested pages (papka tuzilishi) → markdown/rich-text.
- **🛠️ Tech tavsiya:** Next.js + TipTap/Slate editor, Yjs (CRDT) + WebSocket, PostgreSQL, Redis (presence).
- **🚀 Qo'shimcha:** Offline rejim (keyin sync), versiya tarixi (history), izohlar (comments), @mention.

### 🔹 Issue Tracker (Jira-lite)
- **🎯 Maqsad:** Jamoa uchun vazifa/bug kuzatuvchi — Kanban board bilan. Ustunlar bo'ylab kartochkalarni sudrab tashlash, assignee, label, priority.
- **📚 Nimani o'rganasiz:** Drag-and-drop, kompleks relational DB (project→board→column→card→user), role-based access, activity log, real-time updates.
- **⚙️ Asosiy funksiyalar:** Loyiha yaratish → jamoa a'zolarini taklif qilish → issue yaratish (title, description, assignee, label) → Kanban board (drag between columns) → filter/qidiruv.
- **🛠️ Tech tavsiya:** React + dnd-kit, tRPC yoki REST, Prisma + PostgreSQL, WebSocket real-time uchun.
- **🚀 Qo'shimcha:** Sprint/milestone, markdown comments, GitHub commit'lar bilan bog'lash, burndown chart.

### 🔹 Booking System (Shifokor/Stol/Xona band qilish)
- **🎯 Maqsad:** Vaqt oynasini band qilish tizimi — masalan shifokorga navbat yoki restoran stoli. Asosiy qiyinchilik: **ikki kishi bir vaqtni band qila olmasligi** (concurrency!).
- **📚 Nimani o'rganasiz:** Vaqt slotlari logikasi, double-booking oldini olish (DB transaction/locking), calendar UI, availability calculation, email confirmation.
- **⚙️ Asosiy funksiyalar:** Provayder o'z bo'sh vaqtlarini belgilaydi → mijoz bo'sh slotni ko'radi va band qiladi → tasdiqlash email → bekor qilish/qayta rejalash → provayder dashboard.
- **🛠️ Tech tavsiya:** Next.js, Prisma + PostgreSQL (transaction bilan atomic booking), Resend/Nodemailer email, FullCalendar.
- **🚀 Qo'shimcha:** To'lov integratsiyasi (Stripe/Payme), SMS eslatma, timezone qo'llab-quvvatlash, waitlist.

### 🔹 URL Monitoring & Uptime Tracker (Alerts bilan)
- **🎯 Maqsad:** Saytlaringiz **ishlayaptimi** — buni kuzatib turadigan tizim. Sayt yiqilsa darhol email/Telegram xabar keladi. UptimeRobot klonining o'zi.
- **📚 Nimani o'rganasiz:** Background jobs/cron, scheduled HTTP checks, alerting, time-series metrics, status page, retry logic.
- **⚙️ Asosiy funksiyalar:** Monitor qo'shish (URL, interval) → har X daqiqada ping → javob vaqti/status kod yozish → yiqilsa alert yuborish → public status page (uptime %).
- **🛠️ Tech tavsiya:** Node/Express + BullMQ (Redis queue) yoki node-cron, PostgreSQL/TimescaleDB, Next.js dashboard, Fly.io deploy.
- **🚀 Qo'shimcha:** SSL sertifikat muddati tekshirish, response time grafigi, ko'p region'dan tekshirish, incident tarixi.

### 🔹 Recipe Manager with Search (Retsept boshqaruvchi)
- **🎯 Maqsad:** Retseptlarni saqlash, qidirish va rejalashtirish. Ingredientlar bo'yicha qidirish ("uyimda tuxum va un bor — nima pishirsam bo'ladi?").
- **📚 Nimani o'rganasiz:** Full-text search, image upload/storage, tag/filter tizimi, many-to-many relations (retsept↔ingredient), pagination.
- **⚙️ Asosiy funksiyalar:** Retsept qo'shish (rasm, ingredientlar, qadamlar) → nom/ingredient/tag bo'yicha qidiruv → sevimlilar → haftalik menyu rejasi → xarid ro'yxati generatsiya.
- **🛠️ Tech tavsiya:** Next.js, PostgreSQL full-text search (yoki Meilisearch), Cloudinary/S3 rasm uchun, Prisma.
- **🚀 Qo'shimcha:** AI bilan retsept tavsiya, kaloriya hisoblash, jamoat retseptlari (ulashish), reyting/izoh.

### 🔹 Forum / Q&A Platform (Reddit/StackOverflow-lite)
- **🎯 Maqsad:** Savol-javob yoki muhokama platformasi — post yozish, ovoz berish (upvote/downvote), komment tashlash. Community qurish tajribasi.
- **📚 Nimani o'rganasiz:** Nested comments (tree structure), voting va ranking algoritmi, reputation tizimi, moderation, feed generation.
- **⚙️ Asosiy funksiyalar:** Post/savol yaratish → upvote/downvote → nested javoblar → "eng yaxshi javob" belgilash → tag bo'yicha bo'limlar → user profil/reputation.
- **🛠️ Tech tavsiya:** Next.js, Prisma + PostgreSQL (recursive query yoki materialized path), Redis (hot ranking cache).
- **🚀 Qo'shimcha:** Hot/Top/New saralash algoritmlari (Reddit ranking), real-time yangi post, markdown + code highlight, notification.

## 🔴 Advanced

### 🔹 Real-time Multiplayer Game (Chess/Tic-tac-toe — WebSocket)
- **🎯 Maqsad:** Ikki o'yinchi internet orqali real vaqtda o'ynaydigan o'yin. Server holatni saqlaydi, harakatlarni tekshiradi (cheat oldini olish), ikkala tomonni sinxron ushlab turadi.
- **📚 Nimani o'rganasiz:** WebSocket bilan real-time bidirectional aloqa, server-authoritative state, matchmaking, game state machine, reconnection handling, latency compensation.
- **⚙️ Asosiy funksiyalar:** O'yin xonasi yaratish/qo'shilish → real-time harakatlar → server tomonda qoidalarni validatsiya → g'olibni aniqlash → reyting/leaderboard → tomoshabin rejimi.
- **🛠️ Tech tavsiya:** React + Socket.IO/native WS, Node backend (state in-memory + Redis), PostgreSQL (o'yin tarixi), Fly.io.
- **🚀 Qo'shimcha:** Matchmaking (skill bo'yicha juftlash), ELO reyting, o'yin qaytadan ko'rish (replay), turnir rejimi, mobil.

### 🔹 Video Call App (WebRTC)
- **🎯 Maqsad:** Brauzerda video qo'ng'iroq — Zoom/Meet klon. Peer-to-peer video/audio oqimi, ekran ulashish. Chuqur texnik loyiha.
- **📚 Nimani o'rganasiz:** WebRTC (peer connection, ICE, STUN/TURN), signaling server, media streams, SFU vs mesh arxitektura, NAT traversal.
- **⚙️ Asosiy funksiyalar:** Xona yaratish/qo'shilish → video/audio oqim → mute/camera off → ekran ulashish → matnli chat → 1:1 va group qo'ng'iroq.
- **🛠️ Tech tavsiya:** React, WebRTC API, signaling uchun Socket.IO, TURN server (coturn), mediasoup (SFU group uchun).
- **🚀 Qo'shimcha:** Yozib olish (recording), virtual fon (background blur), qo'l ko'tarish/reaction, breakout rooms, sifat adaptatsiyasi.

### 🔹 E-Learning Platform (Video + Progress)
- **🎯 Maqsad:** Onlayn kurs platformasi — video darslar, progress kuzatish, quiz, sertifikat. Udemy/Coursera arxitekturasini kichraytirilgan holda qurish.
- **📚 Nimani o'rganasiz:** Video streaming/hosting, progress tracking, payment integration, content access control, quiz engine, HLS adaptive streaming.
- **⚙️ Asosiy funksiyalar:** Kurs yaratish (bo'lim/dars) → video yuklash va stream → o'quvchi progress'i → quiz/test → tugatgach sertifikat → kurs sotib olish.
- **🛠️ Tech tavsiya:** Next.js, Mux/Cloudflare Stream (video), PostgreSQL, Stripe (to'lov), S3, Prisma.
- **🚀 Qo'shimcha:** Live vebinar, o'qituvchi dashboard (daromad analitikasi), muhokama forumi, AI subtitr, drip content (bosqichma-bosqich ochilish).

### 🔹 SaaS Boilerplate (Auth + Billing + Dashboard)
- **🎯 Maqsad:** Har qanday SaaS mahsuloti uchun **tayyor asos** — ro'yxatdan o'tish, obuna to'lovi, jamoa boshqaruvi, dashboard. Bir marta qursangiz, keyin har g'oyada qayta ishlatasiz. Juda qimmatli portfolio.
- **📚 Nimani o'rganasiz:** Multi-tenancy, subscription billing (Stripe webhook'lar), team/organization model, RBAC, transactional email, feature flags.
- **⚙️ Asosiy funksiyalar:** Auth (email + OAuth) → organization/team → rol boshqaruvi → Stripe obuna (trial, upgrade, cancel) → billing portal → admin dashboard.
- **🛠️ Tech tavsiya:** Next.js, Prisma + PostgreSQL, Stripe, NextAuth/Clerk, Resend, Vercel.
- **🚀 Qo'shimcha:** Usage-based billing (metering), audit log, feature flags, referral tizimi, ko'p tilli (i18n).

### 🔹 Analytics Platform (Event Tracking)
- **🎯 Maqsad:** Sayt/ilovadagi foydalanuvchi harakatlarini yig'ib tahlil qiladigan tizim — Google Analytics/PostHog klon. Katta hajmdagi event'larni qabul qilish va real-time dashboard.
- **📚 Nimani o'rganasiz:** High-throughput ingestion, event pipeline, time-series aggregation, batching/buffering, funnel analysis, data modeling for analytics.
- **⚙️ Asosiy funksiyalar:** JS snippet (event yuboradigan) → event ingestion API → real-time yig'ish → dashboard (page views, unique users, funnel) → filtrlash → export.
- **🛠️ Tech tavsiya:** Node ingestion API, ClickHouse/TimescaleDB (analytics DB), Kafka/Redis Stream (buffer), Next.js dashboard.
- **🚀 Qo'shimcha:** Funnel/retention/cohort tahlil, session recording, A/B test, alerting, real-time live view.

### 🔹 Collaborative Code Editor (Birgalikda kod yozish)
- **🎯 Maqsad:** VS Code Live Share yoki Replit tarzida — bir nechta odam bir vaqtda kod yozadi, ijro etadi. Real-time sync + kodni **sandboxda ishga tushirish**.
- **📚 Nimani o'rganasiz:** CRDT/OT (text sync), Monaco editor integratsiya, container-based code execution (sandbox xavfsizligi), syntax highlighting, WebSocket at scale.
- **⚙️ Asosiy funksiyalar:** Xona yaratish → real-time birgalikda kod tahrir → kursor/selection ulashish → kodni ishga tushirish (output ko'rsatish) → fayl daraxti → chat.
- **🛠️ Tech tavsiya:** React + Monaco, Yjs (CRDT), WebSocket, Docker sandbox (yoki Firecracker/WASM), PostgreSQL.
- **🚀 Qo'shimcha:** Ko'p tilli runtime, terminal ulashish, Git integratsiya, live preview (web loyihalar uchun), voice chat.

← [Loyihalar bo'limiga qaytish](./README.md)
