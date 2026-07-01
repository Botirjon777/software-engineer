# DevOps, Data va AI Loyihalar

Dasturchi bo'lish faqat kod yozish emas — kodni *ishga tushirish, kuzatish, ma'lumot bilan boyitish va aqlli qilish* ham shu kasbning bir qismi. Bu fayl aynan shu uch yo'nalishga bag'ishlangan: **DevOps/automation** (kodni serverga chiqarish va boshqarish), **Data engineering** (ma'lumotni yig'ish, tozalash, oqim qilish) va **AI/LLM** (zamonaviy sun'iy intellekt loyihalari).

Nega bu uchtasi bir joyda? Chunki zamonaviy real loyihada ular bir-biriga bog'lanadi: AI ilovang ma'lumot (data) bilan oziqlanadi, o'sha ma'lumot pipeline orqali keladi, va butun tizim DevOps orqali serverda yashaydi. Bu loyihalarni qursang — portfolio'ng "faqat CRUD app yozgan" darajasidan ancha yuqoriga chiqadi.

Ayniqsa **AI/LLM** qismiga alohida e'tibor ber: RAG, embeddings, vector DB va agent'lar — bugungi kunda eng ko'p ish beruvchi va eng qiziq mavzular. Ularni oddiy tushuntirib, real loyiha sifatida quramiz.

---

# 1-QISM: DevOps va Automation

## 🟢 Beginner

### 🔹 CI/CD Pipeline (GitHub Actions)
- **🎯 Maqsad:** O'z loyihangga har push'da avtomatik test ishga tushirib, muvaffaqiyatli bo'lsa deploy qiladigan pipeline sozlash.
- **📚 Nimani o'rganasiz:** CI/CD tushunchasi, GitHub Actions workflow (YAML), avtomatlashtirilgan test/build/deploy bosqichlari, secret'lar bilan ishlash.
- **⚙️ Asosiy funksiyalar:** Push/PR'da workflow ishga tushishi; lint + test bosqichlari; build; muvaffaqiyatli bo'lsa deploy (masalan Vercel/Render'ga); yashil/qizil status badge.
- **🛠️ Tech tavsiya:** GitHub Actions (YAML), o'z mavjud loyihang (Node/Python).
- **🚀 Qo'shimcha:** Matrix build (bir nechta versiyada test); cache bilan tezlashtirish; deploy'dan oldin manual approval.

### 🔹 Dockerize a Full App + docker-compose
- **🎯 Maqsad:** To'liq ilovani (frontend + backend + database) Docker konteynerlarga joylab, bitta buyruq bilan ishga tushirish.
- **📚 Nimani o'rganasiz:** Docker image va container, Dockerfile yozish, `docker-compose` bilan ko'p servisni bog'lash, environment o'zgaruvchilari, volume va network.
- **⚙️ Asosiy funksiyalar:** Har servis uchun Dockerfile; `docker-compose.yml` bilan hamma servisni birga ishga tushirish; database volume; servislararo bog'lanish; `docker compose up` bilan ishga tushish.
- **🛠️ Tech tavsiya:** Docker, docker-compose. Backend (Node/Django), Postgres, frontend (React).
- **🚀 Qo'shimcha:** Multi-stage build (kichik image); `.env` bilan konfiguratsiya; health check; production/dev uchun alohida compose fayl.

## 🟡 Intermediate

### 🔹 Monitoring Dashboard (Prometheus + Grafana)
- **🎯 Maqsad:** Ilovang metrikalarini (so'rovlar soni, xatolar, javob vaqti) yig'ib, chiroyli dashboard'da ko'rsatish.
- **📚 Nimani o'rganasiz:** Observability tushunchasi, metrics yig'ish (Prometheus), vizualizatsiya (Grafana), alerting, time-series ma'lumot.
- **⚙️ Asosiy funksiyalar:** Ilovadan metrics chiqarish (`/metrics` endpoint); Prometheus scrape qilishi; Grafana dashboard qurish; asosiy panellar (RPS, latency, error rate); alert.
- **🛠️ Tech tavsiya:** Prometheus, Grafana, ilovada prometheus client kutubxona. Docker bilan ishga tushir.
- **🚀 Qo'shimcha:** Alertmanager bilan Telegram/email ogohlantirish; custom metrics; node exporter bilan server resursi kuzatish.

### 🔹 Auto-Deploy Bot
- **🎯 Maqsad:** Yangi versiya tayyor bo'lganda avtomatik build qilib, serverga deploy qiladigan va natijani xabar qiladigan bot.
- **📚 Nimani o'rganasiz:** Webhook, avtomatlashtirish, SSH orqali deploy, chat botlar bilan integratsiya (Telegram/Slack), rollback.
- **⚙️ Asosiy funksiyalar:** Git webhook'ni tinglash; serverga SSH bilan ulanib pull + restart; deploy natijasini (muvaffaqiyat/xato) Telegram'ga yuborish; oddiy rollback buyrug'i.
- **🛠️ Tech tavsiya:** Node.js/Python, SSH kutubxona, Telegram Bot API. Yoki GitHub Actions self-hosted runner.
- **🚀 Qo'shimcha:** Deploy tarixini saqlash; bir buyruq bilan oldingi versiyaga qaytish; zero-downtime deploy.

### 🔹 Infrastructure with Terraform
- **🎯 Maqsad:** Cloud serverlaringni qo'lda emas, kod (IaC) orqali yaratish va boshqarish.
- **📚 Nimani o'rganasiz:** Infrastructure as Code (IaC), Terraform HCL sintaksisi, cloud resurslar (server, network, DB), state boshqaruvi, reproducible infratuzilma.
- **⚙️ Asosiy funksiyalar:** Terraform bilan server/VM yaratish; network va firewall qoidalari; `terraform plan`/`apply`/`destroy`; o'zgaruvchilar va modullar.
- **🛠️ Tech tavsiya:** Terraform + istalgan cloud (AWS/DigitalOcean/Hetzner). Bepul tier bilan boshla.
- **🚀 Qo'shimcha:** Modullarga bo'lish; remote state (S3); bir nechta muhit (dev/prod); CI'da terraform ishga tushirish.

### 🔹 Log Aggregator
- **🎯 Maqsad:** Bir nechta servisdan loglarni bir joyga yig'ib, qidirish va tahlil qilish mumkin bo'lgan tizim.
- **📚 Nimani o'rganasiz:** Centralized logging, log parsing, structured logs (JSON), qidiruv indeksi, ELK/Loki stack.
- **⚙️ Asosiy funksiyalar:** Servislardan loglarni yig'ish; markaziy saqlash; sana/servis/level bo'yicha filtrlash; qidiruv; oddiy dashboard.
- **🛠️ Tech tavsiya:** Grafana Loki + Promtail (yengil), yoki ELK (Elasticsearch + Logstash + Kibana).
- **🚀 Qo'shimcha:** Xato spike'da alert; log'lardan metrics ajratish; saqlash muddati (retention) siyosati.

## 🔴 Advanced

### 🔹 Kubernetes'ga Deploy (Helm)
- **🎯 Maqsad:** Ilovangni Kubernetes klasterida ishga tushirib, Helm bilan boshqarish va masshtablash.
- **📚 Nimani o'rganasiz:** Container orchestration, Kubernetes obyektlari (Pod, Deployment, Service, Ingress), Helm chart, scaling, self-healing.
- **⚙️ Asosiy funksiyalar:** Ilovani Deployment sifatida chiqarish; Service va Ingress bilan tashqariga ochish; Helm chart yozish; replica'larni masshtablash; ConfigMap/Secret.
- **🛠️ Tech tavsiya:** Kubernetes (minikube/kind lokal, yoki managed), Helm, kubectl.
- **🚀 Qo'shimcha:** Horizontal Pod Autoscaler; rolling update; resource limit; Prometheus bilan monitoring integratsiya.

### 🔹 Blue-Green Deploy Pipeline
- **🎯 Maqsad:** Yangi versiyani ishlab turgan tizimni to'xtatmasdan, xavfsiz almashtirib deploy qiladigan pipeline.
- **📚 Nimani o'rganasiz:** Zero-downtime deployment, blue-green va canary strategiyalari, traffic switching, tez rollback.
- **⚙️ Asosiy funksiyalar:** Ikki bir xil muhit (blue/green); yangi versiyani green'ga deploy; test o'tsa trafikni green'ga o'tkazish; muammoda darhol blue'ga qaytish.
- **🛠️ Tech tavsiya:** Kubernetes yoki load balancer + IaC + CI/CD. Nginx/Traefik traffic switch uchun.
- **🚀 Qo'shimcha:** Canary (trafikning 5% yangi versiyaga); avtomatik rollback (metrics yomonlashsa); smoke test.

### 🔹 Self-Healing System
- **🎯 Maqsad:** Servis o'lsa yoki nosog'lom bo'lsa, avtomatik aniqlab, qayta ishga tushiradigan yoki almashtiradigan tizim.
- **📚 Nimani o'rganasiz:** Health check, avtomatik tiklanish (recovery), reliability, resilience, circuit breaker.
- **⚙️ Asosiy funksiyalar:** Doimiy health check; nosog'lom servisni aniqlash; avtomatik restart yoki almashtirish; ogohlantirish; hodisalar jurnali.
- **🛠️ Tech tavsiya:** Kubernetes (liveness/readiness probe o'zi qiladi) yoki o'z watchdog service (Go/Python).
- **🚀 Qo'shimcha:** Circuit breaker; avtomatik masshtablash yuk oshganda; chaos testing (atay servis o'ldirib sinash).

---

# 2-QISM: Data Engineering

## 🟢 Beginner

### 🔹 Web Scraper → CSV/DB
- **🎯 Maqsad:** Veb-sahifalardan ma'lumot yig'ib, CSV fayl yoki bazaga saqlaydigan scraper yozish.
- **📚 Nimani o'rganasiz:** Web scraping, HTML parsing, HTTP so'rovlar, ma'lumotni tuzilgan ko'rinishga keltirish, rate limiting va etika.
- **⚙️ Asosiy funksiyalar:** Sahifani yuklab olish; kerakli maydonlarni ajratish (narx, sarlavha, h.k.); CSV/DB'ga yozish; pagination (keyingi sahifa); xatolarni boshqarish.
- **🛠️ Tech tavsiya:** Python (requests + BeautifulSoup), yoki Node.js (cheerio/playwright).
- **🚀 Qo'shimcha:** JS bilan yuklanadigan sahifa uchun Playwright; jadval bo'yicha rejalashtirilgan ishga tushirish; dublikatlarni chetlab o'tish.

### 🔹 Data Cleaning Script
- **🎯 Maqsad:** Iflos (dirty) ma'lumotni — bo'sh, dublikat, noto'g'ri formatdagi — tozalab, tahlilga tayyor holatga keltirish.
- **📚 Nimani o'rganasiz:** Data cleaning/wrangling, missing value, normalization, deduplication, data quality tushunchasi.
- **⚙️ Asosiy funksiyalar:** Bo'sh qiymatlarni to'ldirish/tashlash; dublikatlarni o'chirish; format birxillashtirish (sana, telefon); outlier'larni aniqlash; toza natijani chiqarish.
- **🛠️ Tech tavsiya:** Python + pandas. Kaggle'dagi iflos datasetni ol.
- **🚀 Qo'shimcha:** Tozalash hisobotini yaratish (nechta qator tuzatildi); validatsiya qoidalari; qayta ishlatiladigan cleaning pipeline.

## 🟡 Intermediate

### 🔹 ETL Pipeline (Extract–Transform–Load)
- **🎯 Maqsad:** Ma'lumotni manbadan olib (extract), o'zgartirib (transform), maqsadli bazaga yuklaydigan (load) to'liq pipeline.
- **📚 Nimani o'rganasiz:** ETL tushunchasi, data pipeline arxitekturasi, transformation logikasi, scheduling, idempotentlik.
- **⚙️ Asosiy funksiyalar:** Manbadan (API/CSV/DB) ma'lumot olish; tozalash va boyitish (transform); data warehouse'ga yuklash; jadval bo'yicha ishga tushirish; xato boshqaruvi.
- **🛠️ Tech tavsiya:** Python + pandas, yoki Apache Airflow (orkestratsiya uchun). Maqsad: Postgres/BigQuery.
- **🚀 Qo'shimcha:** Airflow DAG bilan rejalashtirish; incremental load (faqat yangisi); data quality tekshiruvi bosqichi.

### 🔹 Real-Time Data Dashboard
- **🎯 Maqsad:** Ma'lumot kelib turishi bilan jonli yangilanadigan (real-time) vizual dashboard.
- **📚 Nimani o'rganasiz:** Real-time data, WebSocket/SSE, streaming vizualizatsiya, backend'dan frontend'ga jonli uzatish.
- **⚙️ Asosiy funksiyalar:** Ma'lumot manbasidan doimiy o'qish; WebSocket orqali frontend'ga yuborish; jonli grafik/hisoblagichlar; tarixiy va jonli ko'rinish.
- **🛠️ Tech tavsiya:** Node.js/Python backend (WebSocket), React + Chart.js/Recharts frontend.
- **🚀 Qo'шimcha:** Bir nechta manba; filtrlash; ma'lumotni bazaga ham yozish; ogohlantirish (threshold oshsa).

### 🔹 API'dan Data Yig'ib Analiz
- **🎯 Maqsad:** Ochiq API'lardan (ob-havo, valyuta, kripto, h.k.) muntazam ma'lumot yig'ib, tahlil qilib, xulosa chiqarish.
- **📚 Nimani o'rganasiz:** API integratsiya, rate limit, ma'lumotni to'plash va saqlash, tahliliy hisobot, vizualizatsiya.
- **⚙️ Asosiy funksiyalar:** API'dan jadval bo'yicha data olish; bazaga saqlash; trend/o'rtacha/anomaliya tahlili; grafik va hisobot yaratish.
- **🛠️ Tech tavsiya:** Python (requests + pandas + matplotlib), cron bilan rejalashtirish.
- **🚀 Qo'shimcha:** Bir nechta API'ni birlashtirish; avtomatik kunlik hisobot (email/Telegram); oddiy bashorat (forecast).

## 🔴 Advanced

### 🔹 Streaming Pipeline (Kafka)
- **🎯 Maqsad:** Uzluksiz kelayotgan hodisalar oqimini (event stream) real vaqtda qabul qilib, qayta ishlaydigan pipeline.
- **📚 Nimani o'rganasiz:** Event streaming, Apache Kafka (producer/consumer/topic), stream processing, katta hajmli real-time data.
- **⚙️ Asosiy funksiyalar:** Producer hodisalarni Kafka'ga yuborishi; consumer real vaqtda o'qib qayta ishlashi; topic va partition; qayta ishlangan natijani saqlash/ko'rsatish.
- **🛠️ Tech tavsiya:** Apache Kafka (Docker), Python/Java consumer. Faraz qilingan hodisa oqimi generatori bilan sina.
- **🚀 Qo'shimcha:** Stream processing (Kafka Streams/Faust); windowed aggregation (oxirgi 1 daqiqa); dead-letter queue; monitoring.

### 🔹 Recommendation System (oddiy)
- **🎯 Maqsad:** Foydalanuvchi xatti-harakatiga qarab tavsiya (film/mahsulot/maqola) beradigan tizim.
- **📚 Nimani o'rganasiz:** Recommendation algoritmlari, collaborative filtering, content-based filtering, o'xshashlik (similarity) o'lchash.
- **⚙️ Asosiy funksiyalar:** Foydalanuvchi–element o'zaro ta'sir ma'lumoti; o'xshashlik hisoblash; "Sizga yoqishi mumkin" tavsiyalari; oddiy baholash (precision/recall).
- **🛠️ Tech tavsiya:** Python (pandas, scikit-learn, surprise kutubxonasi). MovieLens dataset bilan boshla.
- **🚀 Qo'shimcha:** Hybrid (content + collaborative); real vaqt tavsiya API'si; cold-start muammosini yechish.

---

# 3-QISM: AI va LLM (Zamonaviy)

Bu qism eng qiziq va eng talab yuqori bo'lgan yo'nalish. Bir necha muhim atamani oldindan tushunib olaylik:

- **LLM (Large Language Model):** Claude, GPT kabi til modellari — matnni tushunib, matn generatsiya qiladi.
- **Embeddings:** Matnni raqamlar vektoriga aylantirish — bu orqali "ma'no bo'yicha o'xshashlik"ni matematik o'lchash mumkin bo'ladi.
- **Vector DB:** Embedding vektorlarni saqlab, "shu matnga eng o'xshashini top" so'rovini tez bajaradigan maxsus baza (Pinecone, Chroma, pgvector).
- **RAG (Retrieval-Augmented Generation):** LLM'ga javob berishdan oldin, o'z hujjatlaringdan tegishli qismni topib berish — shunda model faqat o'zi bilganidan emas, sening ma'lumotingdan javob beradi.
- **Agent:** LLM'ni "asboblar" (tool) bilan ta'minlab, u o'zi qaror qabul qilib, bir necha qadamda vazifani bajaradigan tizim.
- **MCP (Model Context Protocol):** Agentlarni tashqi asboblar va ma'lumot manbalariga ulaydigan ochiq standart.

> Eslatma: LLM API'lari bilan ishlaganda aniq model nomi, narx va imkoniyatlarni xotiradan taxmin qilmang — rasmiy hujjatga qarang. Bu loyihalarda Claude (Anthropic) API'sini asosiy tanlov sifatida ishlatish tavsiya etiladi.

## 🟢 Beginner

### 🔹 Chatbot with LLM API (Claude / OpenAI)
- **🎯 Maqsad:** LLM API'ga ulanib, foydalanuvchi bilan suhbatlashadigan chatbot qurish.
- **📚 Nimani o'rganasiz:** LLM API bilan ishlash, prompt yuborish, suhbat tarixini (context) saqlash, streaming javob, API key xavfsizligi.
- **⚙️ Asosiy funksiyalar:** Foydalanuvchi xabarini API'ga yuborish; javobni ko'rsatish; suhbat tarixini eslab qolish; system prompt bilan "shaxs" berish; oqib chiqadigan (streaming) javob.
- **🛠️ Tech tavsiya:** Claude API (Anthropic SDK) yoki OpenAI SDK. Frontend: React yoki oddiy CLI. API key'ni backend'da yashir.
- **🚀 Qo'шimcha:** Suhbatni bazaga saqlash; bir nechta "shaxs" (persona); token hisoblagich; markdown render.

### 🔹 Text Summarizer
- **🎯 Maqsad:** Uzun matn yoki maqolani LLM yordamida qisqa, mazmunli xulosaga aylantiruvchi ilova.
- **📚 Nimani o'rganasiz:** Prompt engineering, matnni bo'laklash (chunking), uzunlik cheklovlari, natija formatini boshqarish.
- **⚙️ Asosiy funksiyalar:** Matn/URL kiritish; xulosa uzunligini tanlash (qisqa/o'rta); asosiy fikrlar (bullet) chiqarish; til tanlash.
- **🛠️ Tech tavsiya:** Claude/OpenAI API + Python/Node. Web maqola uchun scraper qo'sh.
- **🚀 Qo'shimcha:** PDF yuklab xulosa qilish; juda uzun matnni bo'lib-bo'lib (map-reduce) qisqartirish; kalit so'zlar ajratish.

### 🔹 Prompt Playground
- **🎯 Maqsad:** Turli promptlarni sinab, natijalarni yonma-yon solishtiradigan interaktiv vosita.
- **📚 Nimani o'rganasiz:** Prompt engineering, parametrlar (temperature, max tokens) ta'siri, promptlarni taqqoslash, model xulq-atvori.
- **⚙️ Asosiy funksiyalar:** System/user prompt tahrirlash; parametrlarni sozlash; javobni ko'rish; ikki promptni yonma-yon solishtirish; promptlarni saqlash.
- **🛠️ Tech tavsiya:** Claude/OpenAI API, React frontend (yoki Streamlit — tez).
- **🚀 Qo'shimcha:** A/B taqqoslash; prompt shablon kutubxonasi; xarajat (token narxi) ko'rsatkichi; natijani baholash.

## 🟡 Intermediate

### 🔹 RAG System (o'z hujjatlaring ustidan savol-javob)
- **🎯 Maqsad:** O'z hujjatlaring (PDF, matn, wiki) ustidan savol berib, aniq javob oladigan tizim qurish — "sening ma'lumoting bilan gaplashadigan AI".
- **📚 Nimani o'rganasiz:** RAG'ning to'liq oqimi — embeddings, chunking, vector DB, retrieval, keyin LLM'ga context berib javob generatsiya. Eng muhim zamonaviy AI ko'nikma.
- **⚙️ Asosiy funksiyalar:** Hujjatlarni bo'laklash (chunk); har bo'lakni embedding'ga aylantirish; vector DB'ga saqlash; savolga eng mos bo'laklarni topish (retrieval); ularni LLM'ga context qilib berib javob olish; manba (source) ko'rsatish.
- **🛠️ Tech tavsiya:** Claude API + embedding model + vector DB (Chroma/pgvector/Pinecone). Python (LangChain/LlamaIndex ixtiyoriy) yoki toza SDK.
- **🚀 Qo'shimcha:** Manba iqtiboslari (citations); hybrid search (keyword + semantic); re-ranking; suhbat tarixini hisobga olish.

### 🔹 Semantic Search
- **🎯 Maqsad:** Kalit so'z emas, *ma'no* bo'yicha qidiradigan qidiruv tizimi ("arzon telefon" so'rovi "byudjet smartfon"ni ham topsin).
- **📚 Nimani o'rganasiz:** Embeddings, vector similarity (cosine), vector DB, semantik va keyword qidiruv farqi.
- **⚙️ Asosiy funksiyalar:** Kontentni embedding qilib indekslash; so'rovni embedding'ga aylantirish; eng o'xshash natijalarni chiqarish; relevantlik bo'yicha saralash.
- **🛠️ Tech tavsiya:** Embedding model + vector DB (Chroma/pgvector). Python yoki Node.
- **🚀 Qo'shimcha:** Filtrlar bilan birlashtirish; keyword + semantic hybrid; katta hajmni indekslash tezligini optimallash.

### 🔹 PDF / Document Q&A Bot
- **🎯 Maqsad:** PDF yoki hujjatni yuklab, uning mazmuni bo'yicha savol-javob qiladigan bot.
- **📚 Nimani o'rganasiz:** PDF matn ajratish, document chunking, RAG'ni amaliy qo'llash, context uzunligini boshqarish.
- **⚙️ Asosiy funksiyalar:** PDF yuklash va matnini ajratish; bo'laklash + embedding + saqlash; savolga hujjatdan javob; qaysi sahifadan olinganini ko'rsatish; ko'p hujjat.
- **🛠️ Tech tavsiya:** Claude API + PDF parser (pypdf) + vector DB. Frontend: Streamlit yoki React.
- **🚀 Qo'shimcha:** Skanerlangan PDF uchun OCR; jadval/rasm bilan ishlash; bir nechta hujjatni birga so'rash.

### 🔹 Semantic Search + AI Code Reviewer
- **🎯 Maqsad:** Pull request yoki kod o'zgarishini LLM yordamida ko'rib chiqib, muammo va yaxshilash takliflarini beradigan tizim.
- **📚 Nimani o'rganasiz:** Kodni LLM'ga context sifatida berish, diff tahlili, strukturaviy (JSON) javob olish, CI bilan integratsiya.
- **⚙️ Asosiy funksiyalar:** Diff/faylni olish; LLM'ga review promptida yuborish; xato, xavfsizlik, uslub bo'yicha izohlar; strukturaviy natija; PR'ga komment qo'yish (ixtiyoriy).
- **🛠️ Tech tavsiya:** Claude API + GitHub API/webhook. Python/Node. GitHub Actions bilan bog'la.
- **🚀 Qo'shimcha:** Repository konteksti (RAG bilan tegishli fayllar); jiddiylik darajasi; faqat o'zgargan qatorlarga izoh.

## 🔴 Advanced

### 🔹 AI Agent (tool-calling, MCP bilan)
- **🎯 Maqsad:** LLM'ni asboblar (qidiruv, hisoblash, API chaqirish) bilan ta'minlab, vazifani o'zi rejalab, bir necha qadamda bajaradigan agent qurish.
- **📚 Nimani o'rganasiz:** Tool calling (function calling), agent loop (o'ylash → asbob → natija → yana o'ylash), MCP protokoli, ko'p qadamli reasoning.
- **⚙️ Asosiy funksiyalar:** Asboblarni ta'riflash (schema); LLM asbob chaqirishni tanlaydi; asbobni bajarib natijani qaytarish; agent loop bilan maqsadga yetguncha davom etish; MCP orqali tashqi asboblarga ulanish.
- **🛠️ Tech tavsiya:** Claude API (tool use) + MCP. Python/Node SDK.
- **🚀 Qo'shimcha:** Xotira (memory); parallel tool calling; xavfsizlik (asbob ruxsatlari); reja-va-bajarish (plan-and-execute) arxitektura.

### 🔹 Multi-Agent System
- **🎯 Maqsad:** Bir necha ixtisoslashgan agent (masalan tadqiqotchi, yozuvchi, tekshiruvchi) o'zaro hamkorlikda murakkab vazifani bajaradigan tizim.
- **📚 Nimani o'rganasiz:** Multi-agent orkestratsiya, agentlar orasida rol taqsimlash, muloqot va koordinatsiya, orchestrator pattern.
- **⚙️ Asosiy funksiyalar:** Har agentga rol va asboblar; orchestrator vazifani bo'lib beradi; agentlar natijani bir-biriga uzatadi; yakuniy natijani yig'ish; nazorat (tekshiruvchi agent).
- **🛠️ Tech tavsiya:** Claude API + o'z orkestratsiya kodi (yoki agent framework). Python.
- **🚀 Qo'shimcha:** Parallel agentlar; umumiy xotira/knowledge base; xatoda qayta urinish; jarayonni vizual kuzatish.

### 🔹 Fine-tuning yoki Local LLM Deploy
- **🎯 Maqsad:** Ochiq LLM'ni o'z serveringda ishga tushirish yoki maxsus vazifaga moslashtirish (fine-tune).
- **📚 Nimani o'rganasiz:** Local model inference, quantization, fine-tuning (LoRA), GPU/CPU resurslari, model deploy va serving.
- **⚙️ Asosiy funksiyalar:** Ochiq modelni (Llama/Mistral) lokal ishga tushirish; API sifatida ochish; maxsus dataset bilan fine-tune (LoRA); natijani baholash; deploy.
- **🛠️ Tech tavsiya:** Ollama (tez lokal ishga tushirish), yoki Hugging Face + PEFT/LoRA fine-tuning uchun. GPU tavsiya etiladi.
- **🚀 Qo'shimcha:** Quantization bilan kichik xotirada ishlatish; o'z chatbot'ingga ulash; hosting API sifatida; sifatni tijoriy model bilan taqqoslash.

### 🔹 Coding Assistant
- **🎯 Maqsad:** Kod yozish, tushuntirish, refactor qilish va debug'da yordam beradigan AI yordamchi (Copilot/Cursor mini g'oyasi).
- **📚 Nimani o'rganasiz:** Kod konteksti bilan ishlash, RAG codebase ustida, tool-calling (fayl o'qish/yozish/test), agentic coding workflow.
- **⚙️ Asosiy funksiyalar:** Kod so'rash va generatsiya; mavjud codebase'ni RAG bilan tushunish; faylni o'qish/o'zgartirish asboblari; testni ishga tushirib natijaga qarab tuzatish; suhbat orqali ishlash.
- **🛠️ Tech tavsiya:** Claude API (tool use) + fayl tizimi asboblari + vector DB (codebase indeksi). Python/Node, CLI yoki VS Code kengaytma.
- **🚀 Qo'shimcha:** Ko'p faylni bir vaqtda tahrirlash; test-driven avtomatik tuzatish; git bilan integratsiya; MCP orqali kengaytirish.

← [Loyihalar bo'limiga qaytish](./README.md)
