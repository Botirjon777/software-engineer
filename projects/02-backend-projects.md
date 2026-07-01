# Backend Loyihalar

Real muammolarni yechadigan, production-darajadagi API va distributed system tushunchalariga urg'u bergan Node.js/TypeScript loyihalari.

## 🟢 Beginner

### 🔹 URL Shortener (Analytics bilan)
- **🎯 Maqsad:** Uzun URL'ni qisqa link'ga aylantiradigan (bit.ly kabi) servis — har bir bosishni sanaydi va statistika beradi. Klassik, lekin real foydali.
- **📚 Nimani o'rganasiz:** REST API dizayni, short-code generatsiya (base62), redirect (3xx), oddiy analytics yozish, DB indekslar.
- **⚙️ Asosiy funksiyalar:** Link yaratish (POST) va redirect (GET); noyob short-code; click hisoblash; kunlik statistika endpoint.
- **🛠️ Tech tavsiya:** Node.js + TypeScript, Express/Fastify, PostgreSQL yoki SQLite, Prisma.
- **🚀 Qo'shimcha:** Custom alias, QR kod generatsiya, link muddati (expiry), UTM parametrlar tahlili.

### 🔹 File Upload Service (Image Resize)
- **🎯 Maqsad:** Rasm yuklaydigan va avtomatik bir nechta o'lchamga (thumbnail, medium) resize qiladigan servis. Har qanday ilovada kerak bo'ladigan infratuzilma.
- **📚 Nimani o'rganasiz:** Multipart upload, stream'lar, image processing, storage abstraktsiyasi (local/S3), MIME validatsiya.
- **⚙️ Asosiy funksiyalar:** Rasm yuklash (multipart); bir nechta o'lchamga resize; noyob URL qaytarish; fayl turi/hajmi validatsiyasi.
- **🛠️ Tech tavsiya:** Node.js + TypeScript, Fastify + `@fastify/multipart`, `sharp`, S3/MinIO.
- **🚀 Qo'shimcha:** WebP/AVIF konvertatsiya, on-the-fly resize (URL param), virus scan, CDN integratsiya, direct-to-S3 presigned URL.

### 🔹 REST API for a Real Domain (Kutubxona / Kino)
- **🎯 Maqsad:** To'liq CRUD API bir real domen uchun — masalan kutubxona (kitob/muallif/o'quvchi/qarz) yoki kino katalogi. Backend asoslarini mustahkamlaydi.
- **📚 Nimani o'rganasiz:** REST resurs modellashtirish, relational schema, validatsiya, pagination/filtering, xatoliklarni to'g'ri qaytarish.
- **⚙️ Asosiy funksiyalar:** CRUD bir nechta resurs uchun; relationlar (bir-nechta/ko'p-ko'p); qidiruv va pagination; input validatsiya.
- **🛠️ Tech tavsiya:** Node.js + TypeScript, Fastify, PostgreSQL, Prisma/Drizzle, `zod`.
- **🚀 Qo'shimcha:** OpenAPI/Swagger hujjat, role-based access, soft delete, audit log, seed data.

### 🔹 Webhook Receiver
- **🎯 Maqsad:** Tashqi servislardan (GitHub, Stripe, Telegram) webhook qabul qiladigan, tekshiradigan va qayta ishlaydigan endpoint. Integratsiyalar uchun asosiy pattern.
- **📚 Nimani o'rganasiz:** HMAC signature verification, idempotency, raw body parsing, retry/queue, event logging.
- **⚙️ Asosiy funksiyalar:** Webhook qabul qilish; imzoni tekshirish; idempotent qayta ishlash; event log va replay.
- **🛠️ Tech tavsiya:** Node.js + TypeScript, Express (raw body), Redis idempotency uchun.
- **🚀 Qo'shimcha:** Dead-letter queue, webhook'ni boshqa endpoint'ga forward, dashboard, exponential backoff retry.

### 🔹 Markdown-to-PDF API
- **🎯 Maqsad:** Markdown matn qabul qilib, chiroyli PDF qaytaradigan API. Hisobot, invoice, sertifikat generatsiyasi uchun real foydali.
- **📚 Nimani o'rganasiz:** Markdown→HTML→PDF pipeline, headless browser (Puppeteer), template rendering, stream orqali fayl qaytarish.
- **⚙️ Asosiy funksiyalar:** Markdown qabul qilish (POST); HTML template'ga solish; PDF generatsiya; download qaytarish.
- **🛠️ Tech tavsiya:** Node.js + TypeScript, `markdown-it`, Puppeteer yoki `@react-pdf`, Fastify.
- **🚀 Qo'shimcha:** Custom template/CSS, invoice generatori, header/footer va sahifa raqami, PDF'larni cache qilish.

### 🔹 Short Poll / Voting API
- **🎯 Maqsad:** So'rovnoma yaratish va ovoz berish API'si — natijalar real vaqtda hisoblanadi. Oddiy, lekin concurrency muammolarini o'rgatadi.
- **📚 Nimani o'rganasiz:** Atomic update (race condition), unique ovoz cheklovi (IP/token), aggregatsiya so'rovlari, caching.
- **⚙️ Asosiy funksiyalar:** So'rovnoma yaratish; ovoz berish (bir marta); natija endpoint; muddat cheklash.
- **🛠️ Tech tavsiya:** Node.js + TypeScript, PostgreSQL (atomic `UPDATE`), Redis rate-limit uchun.
- **🚀 Qo'shimcha:** Real-time natija (SSE/WebSocket), fraud aniqlash, multi-choice, natija grafigi.

## 🟡 Intermediate

### 🔹 Rate-Limited Public API (API Key + Usage Tracking)
- **🎯 Maqsad:** Tashqi developerlar API key bilan foydalanadigan, so'rovlar soni cheklangan va hisoblanadigan public API. SaaS backend'ning asosi.
- **📚 Nimani o'rganasiz:** API key generatsiya/hashing, rate limiting algoritmlari (token bucket/sliding window), usage metering, tier'lar.
- **⚙️ Asosiy funksiyalar:** API key yaratish/bekor qilish; per-key rate limit; usage tracking va statistika; tier'lar (free/pro).
- **🛠️ Tech tavsiya:** Node.js + TypeScript, Fastify, Redis (rate limit + counter), PostgreSQL.
- **🚀 Qo'shimcha:** Billing integratsiya (Stripe), developer dashboard, quota alertlar, per-endpoint limit.

### 🔹 Real-Time Chat Backend (WebSocket)
- **🎯 Maqsad:** Xonalar, presence, typing indikatori bilan chat backend. Real-time tizim muhandisligi.
- **📚 Nimani o'rganasiz:** WebSocket lifecycle, room/channel model, presence tracking, message persistence, horizontal scaling (Redis pub/sub).
- **⚙️ Asosiy funksiyalar:** Xona yaratish/qo'shilish; real-time xabar; presence va typing; xabar tarixi.
- **🛠️ Tech tavsiya:** Node.js + TypeScript, `ws`/`socket.io`, Redis pub/sub, PostgreSQL.
- **🚀 Qo'shimcha:** O'qildi belgisi, fayl yuborish, ko'p instansiyaga scale (Redis adapter), push notification offline uchun.

### 🔹 Job / Task Queue with Workers
- **🎯 Maqsad:** Og'ir vazifalarni (email yuborish, rasm ishlash) fon rejimida bajaradigan queue tizimi. Har qanday jiddiy backend'da bor.
- **📚 Nimani o'rganasiz:** Producer/consumer pattern, queue mexanikasi, retry va backoff, concurrency boshqaruvi, dead-letter queue.
- **⚙️ Asosiy funksiyalar:** Job qo'shish; worker'lar tomonidan bajarish; retry failure'da; job holati kuzatish.
- **🛠️ Tech tavsiya:** Node.js + TypeScript, BullMQ + Redis, alohida worker process.
- **🚀 Qo'shimcha:** Kechiktirilgan/rejalashtirilgan job, priority queue, Bull Board dashboard, rate-limited worker.

### 🔹 Notification Service (Email / SMS / Push)
- **🎯 Maqsad:** Bir necha kanal orqali (email, SMS, push) bildirishnoma yuboradigan markaziy servis. Provider-agnostik dizayn mashqi.
- **📚 Nimani o'rganasiz:** Adapter/strategy pattern, template tizimi, queue bilan async yuborish, provider failover, delivery tracking.
- **⚙️ Asosiy funksiyalar:** Bir nechta kanal; template va o'zgaruvchilar; queue orqali yuborish; delivery holati.
- **🛠️ Tech tavsiya:** Node.js + TypeScript, BullMQ, Nodemailer, Twilio, FCM/`web-push`.
- **🚀 Qo'shimcha:** Foydalanuvchi preferens (opt-out), digest (guruhlash), A/B test, provider fallback, retry.

### 🔹 Authentication Service (JWT + Refresh + OAuth)
- **🎯 Maqsad:** To'liq auth servisi — ro'yxatdan o'tish, login, refresh token rotatsiyasi, OAuth (Google/GitHub). Xavfsizlikning yuragi.
- **📚 Nimani o'rganasiz:** Parol hashing (argon2/bcrypt), JWT access/refresh, token rotation, OAuth2 flow, session xavfsizligi.
- **⚙️ Asosiy funksiyalar:** Register/login; access + refresh token; OAuth login; logout va token bekor qilish.
- **🛠️ Tech tavsiya:** Node.js + TypeScript, Fastify, `argon2`, `jose`, Redis (refresh store).
- **🚀 Qo'shimcha:** 2FA (TOTP), email verifikatsiya, parolni tiklash, device/session boshqaruvi, rate-limit brute-force'ga.

### 🔹 Full-Text Search API
- **🎯 Maqsad:** Hujjatlar bo'yicha tez qidiruv API'si — relevance ranking, filtr, autocomplete bilan. Qidiruv muhandisligi asoslari.
- **📚 Nimani o'rganasiz:** Inverted index tushunchasi, tokenizatsiya/stemming, relevance (BM25/TF-IDF), faceted search, indexing pipeline.
- **⚙️ Asosiy funksiyalar:** Hujjat indekslash; full-text qidiruv relevance bilan; filtr/facet; autocomplete/suggest.
- **🛠️ Tech tavsiya:** Node.js + TypeScript, Elasticsearch/Meilisearch (yoki PostgreSQL `tsvector`, yoki o'z inverted index).
- **🚀 Qo'shimcha:** Typo tolerance (fuzzy), synonym, natijalarni highlight, ko'p tilli qidiruv (o'zbekcha), search analytics.

### 🔹 E-Commerce Backend (Cart / Order / Payment Webhook)
- **🎯 Maqsad:** Onlayn do'kon backend'i — savat, buyurtma, to'lov (Stripe webhook), zaxira boshqaruvi. Real biznes logikasi.
- **📚 Nimani o'rganasiz:** Cart va order state machine, transaction, stock reservation (race condition), payment webhook, idempotency.
- **⚙️ Asosiy funksiyalar:** Mahsulot katalogi; savat; buyurtma yaratish (transaction); to'lov webhook bilan holat yangilash.
- **🛠️ Tech tavsiya:** Node.js + TypeScript, Fastify, PostgreSQL (transaction), Prisma, Stripe.
- **🚀 Qo'shimcha:** Kupon/chegirma, inventar reservation TTL bilan, refund oqimi, buyurtma tarixi va status webhook.

## 🔴 Advanced

### 🔹 URL Shortener at Scale (Caching + Analytics)
- **🎯 Maqsad:** Beginner URL shortener'ni millionlab so'rovga chidaydigan qilib qayta loyihalash — cache, read replica, real-time analytics pipeline bilan. Scalability mashqi.
- **📚 Nimani o'rganasiz:** Multi-layer caching (Redis + in-memory), cache invalidation, high-throughput yozuv (batching), distributed ID generatsiya, read/write ajratish.
- **⚙️ Asosiy funksiyalar:** Cache'li tez redirect; async analytics ingest (queue); aggregatsiya (kunlik/soatlik); DB indeks/sharding strategiyasi.
- **🛠️ Tech tavsiya:** Node.js + TypeScript, Redis, Kafka/queue, PostgreSQL/ClickHouse analytics uchun.
- **🚀 Qo'shimcha:** Geo-analytics, bot filtrlash, Snowflake-style ID, rate-limit, load test (k6) natijalari.

### 🔹 Video Transcoding Pipeline
- **🎯 Maqsad:** Yuklangan videoni turli sifat/formatga (240p–1080p, HLS) avtomatik konvertatsiya qiladigan pipeline. YouTube backend'ining kichik versiyasi.
- **📚 Nimani o'rganasiz:** FFmpeg, uzoq davom etadigan job'lar, progress tracking, storage/CDN, HLS/DASH segmentatsiya, worker orkestratsiya.
- **⚙️ Asosiy funksiyalar:** Video yuklash; queue orqali transcode (bir nechta bitrate); HLS manifest generatsiya; holat/progress API.
- **🛠️ Tech tavsiya:** Node.js + TypeScript, `fluent-ffmpeg`, BullMQ, S3/MinIO, CDN.
- **🚀 Qo'shimcha:** Adaptive bitrate streaming, thumbnail/sprite generatsiya, webhook tugagach, avtomatik subtitr (Whisper).

### 🔹 Distributed Job Scheduler (Cron)
- **🎯 Maqsad:** Ko'p instansiyada ishlaydigan, aynan bir marta bajarilishini kafolatlaydigan taqsimlangan cron scheduler. Distributed systems'ning klassik muammosi.
- **📚 Nimani o'rganasiz:** Distributed locking, leader election, "exactly-once" bajarish, clock skew, missed-run recovery, persistence.
- **⚙️ Asosiy funksiyalar:** Cron/interval job ro'yxatga olish; taqsimlangan lock bilan bir marta bajarish; failure recovery; bajarilish tarixi.
- **🛠️ Tech tavsiya:** Node.js + TypeScript, Redis (Redlock) yoki PostgreSQL advisory lock, `cron-parser`.
- **🚀 Qo'shimcha:** Web UI job boshqaruvi, timezone qo'llab-quvvatlash, retry policy, kechikish alertlari, horizontal scaling test.

### 🔹 API Gateway (Proxy + Rate Limit + Auth)
- **🎯 Maqsad:** Bir nechta mikroservis oldida turadigan gateway — routing, auth, rate limit, logging'ni markazlashtiradi. Mikroservis arxitekturasining kirish nuqtasi.
- **📚 Nimani o'rganasiz:** Reverse proxy, request routing, o'rtada auth/rate-limit, circuit breaker, request/response transform, observability.
- **⚙️ Asosiy funksiyalar:** Path-based routing servislarga; markaziy auth; per-route rate limit; markaziy logging/metrics.
- **🛠️ Tech tavsiya:** Node.js + TypeScript, `http-proxy`, Redis, `pino` logging, OpenTelemetry.
- **🚀 Qo'shimcha:** Circuit breaker, retry/timeout, response caching, canary/A-B routing, config'ni hot-reload.

### 🔹 Event-Driven Order System (Message Queue)
- **🎯 Maqsad:** Buyurtma oqimini event'lar bilan boshqaradigan tizim — order, payment, inventory, shipping alohida servis, message queue orqali bog'langan. Saga pattern mashqi.
- **📚 Nimani o'rganasiz:** Event-driven arxitektura, saga/orchestration, eventual consistency, idempotent consumer, outbox pattern, compensation.
- **⚙️ Asosiy funksiyalar:** Order event publish; servislar event'ga reaksiya; saga bilan tranzaksiya koordinatsiyasi; failure'da compensation.
- **🛠️ Tech tavsiya:** Node.js + TypeScript, RabbitMQ/Kafka/NATS, PostgreSQL (outbox), Docker Compose.
- **🚀 Qo'shimcha:** Event sourcing, dead-letter handling, distributed tracing, event replay, schema registry.

### 🔹 O'zining Mini-ORM yoki Query Builder
- **🎯 Maqsad:** Prisma/Knex kabi type-safe query builder yoki ORM'ni noldan yozish. Abstraktsiya va TypeScript type sehrini chuqur o'rganish.
- **📚 Nimani o'rganasiz:** SQL generatsiya, fluent/chainable API dizayni, ilg'or TypeScript generics/type inference, connection pooling, migration.
- **⚙️ Asosiy funksiyalar:** Chainable query builder (`select().where()`); parametrlangan xavfsiz SQL; model→jadval mapping; oddiy relation.
- **🛠️ Tech tavsiya:** TypeScript, `pg` driver, ilg'or generic type'lar.
- **🚀 Qo'shimcha:** Migration tizimi, transaction API, lazy/eager relation loading, query log, bir nechta dialekt (PG/MySQL/SQLite).

### 🔹 Multi-Tenant SaaS Backend
- **🎯 Maqsad:** Bir kod bazasi ko'p mijoz (tenant)ga xizmat qiladigan, ma'lumotlari izolyatsiya qilingan SaaS backend. Real SaaS arxitektura muammosi.
- **📚 Nimani o'rganasiz:** Tenant izolyatsiya strategiyalari (row-level / schema / DB), tenant-aware auth, per-tenant config, xavfsizlik chegaralari.
- **⚙️ Asosiy funksiyalar:** Tenant ro'yxatdan o'tishi; har so'rovda tenant kontekst; ma'lumot izolyatsiyasi (RLS); per-tenant sozlama.
- **🛠️ Tech tavsiya:** Node.js + TypeScript, PostgreSQL Row-Level Security, Fastify, Redis.
- **🚀 Qo'shimcha:** Per-tenant rate limit/billing, custom domen, tenant migration, usage analytics, admin panel.

← [Loyihalar bo'limiga qaytish](./README.md)
