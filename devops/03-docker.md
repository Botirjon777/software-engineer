# Docker va Containers

Docker — bu zamonaviy software engineeringning eng muhim asboblaridan biri. U dasturni va uning barcha bog'liqliklarini (dependencies) yagona, ko'chiriladigan (portable) paketga jamlaydi. Natijada dastur sizning laptopingizda ham, hamkasbingiznikida ham, production serverda ham bir xil ishlaydi. Bu hujjatda biz containerlar nima ekanini, Docker arxitekturasini, Dockerfile yozishni, image optimizatsiyasini va Docker Compose'ni o'zbek tilida, amaliy misollar bilan ko'rib chiqamiz.

## Mundarija

- [Container nima va nega kerak](#container-nima-va-nega-kerak)
- [VM bilan taqqoslash](#vm-bilan-taqqoslash)
- [Docker arxitekturasi](#docker-arxitekturasi)
- [Image vs Container](#image-vs-container)
- [Layer va Union Filesystem](#layer-va-union-filesystem)
- [Dockerfile asoslari](#dockerfile-asoslari)
- [To'liq Node.js misol](#toliq-nodejs-misol)
- [Image build va run](#image-build-va-run)
- [Layer caching va optimizatsiya](#layer-caching-va-optimizatsiya)
- [Multi-stage build](#multi-stage-build)
- [.dockerignore](#dockerignore)
- [Volume va Bind Mount](#volume-va-bind-mount)
- [Port mapping va Networking](#port-mapping-va-networking)
- [Docker Compose](#docker-compose)
- [Image hajmini kichraytirish](#image-hajmini-kichraytirish)
- [Best Practices](#best-practices)
- [Savol-javoblar](#savol-javoblar)
- [Masalalar](#masalalar)

---

## Container nima va nega kerak

**💡 Tushuncha:** Container — bu dasturni va uning ishlashi uchun kerakli hamma narsani (code, runtime, system tools, kutubxonalar, sozlamalar) bitta izolyatsiyalangan paketga o'rab beruvchi texnologiya. Container host operatsion tizimning kerneli ustida ishlaydi, lekin o'zining alohida fayl tizimi, process maydoni va tarmoq stekiga ega.

Eng mashhur muammoni eslang: **"Menda ishlayapti"** ("works on my machine"). Dasturchi o'z laptopida hamma narsa ishlayotganini ko'radi, lekin serverga deploy qilingach dastur ishlamay qoladi. Sabablari:

- Laptopda Node.js v20, serverda v16.
- Laptopda kerakli system library bor, serverda yo'q.
- Environment variable'lar boshqacha.
- Operatsion tizim farqi (macOS vs Linux).

Container bu muammoni hal qiladi: u dasturni **bir xil muhit** bilan birga o'raydi. Qayerda ishga tushirsangiz ham, ichidagi muhit aynan bir xil bo'ladi.

```text
+-----------------------------------------+
|         Application Code                |
+-----------------------------------------+
|   Dependencies (npm, libs, runtime)     |
+-----------------------------------------+
|   System Tools, Config, ENV             |
+-----------------------------------------+
|        => Bularning hammasi             |
|        bitta CONTAINER ichida           |
+-----------------------------------------+
```

**Nega container kerak:**

- **Consistency** — har joyda bir xil muhit.
- **Isolation** — bir container boshqasiga xalaqit bermaydi.
- **Portability** — bir marta build qilasiz, hamma joyda ishlatasiz.
- **Tezlik** — container soniyalarda ishga tushadi.
- **Resurs tejamkorligi** — VM'dan ancha yengil.

---

## VM bilan taqqoslash

Containerni tushunish uchun uni Virtual Machine (VM) bilan solishtirish kerak.

**Virtual Machine (VM):** Har bir VM o'zining to'liq operatsion tizimini (guest OS) yuklaydi. Ular orasida **Hypervisor** (masalan VMware, VirtualBox) turadi. Bu og'ir — har bir VM gigabaytlab joy egallaydi va ishga tushishi daqiqalar oladi.

**Container:** Containerlar host OS'ning **kernelini** ulashadi. Ular alohida OS yuklamaydi — faqat dastur va uning kutubxonalarini o'z ichiga oladi. Shuning uchun yengil (megabaytlar) va tez (soniyalar).

```text
   VIRTUAL MACHINES                      CONTAINERS
+------+ +------+ +------+         +------+ +------+ +------+
| App  | | App  | | App  |         | App  | | App  | | App  |
+------+ +------+ +------+         +------+ +------+ +------+
| Libs | | Libs | | Libs |         | Libs | | Libs | | Libs |
+------+ +------+ +------+         +------+-+------+-+------+
|Guest | |Guest | |Guest |         |                       |
|  OS  | |  OS  | |  OS  |         |   Docker Engine       |
+------+-+------+-+------+         +-----------------------+
|     Hypervisor        |         |       Host OS         |
+-----------------------+         +-----------------------+
|       Host OS         |         |      Hardware         |
+-----------------------+         +-----------------------+
|      Hardware         |
+-----------------------+
```

| Xususiyat | Virtual Machine | Container |
|-----------|-----------------|-----------|
| Hajm | GB (gigabaytlar) | MB (megabaytlar) |
| Ishga tushish | Daqiqalar | Soniyalar |
| OS | Har VM o'z OS'i | Host kernelni ulashadi |
| Izolyatsiya | Kuchli (to'liq OS) | Process darajasida |
| Resurs sarfi | Yuqori | Past |

**⚠️ Ehtiyot bo'l:** Containerlar host kernelni ulashgani uchun ular VM kabi to'liq izolyatsiya bermaydi. Yuqori xavfsizlik talab qilinadigan multi-tenant muhitlarda bu jihatni hisobga olish kerak.

---

## Docker arxitekturasi

Docker **client-server** arxitekturasida ishlaydi. Uchta asosiy komponent bor:

```text
+----------------+        +---------------------------+
|  Docker CLI    |        |     Docker DAEMON         |
|  (client)      |        |     (dockerd)             |
|                |  REST  |                           |
| docker build --+------->|  - Images boshqaradi      |
| docker run     |  API   |  - Containers boshqaradi  |
| docker pull    |        |  - Networks, Volumes      |
+----------------+        +-------------+-------------+
                                        |
                                        | pull / push
                                        v
                          +---------------------------+
                          |   Docker REGISTRY         |
                          |   (Docker Hub, ECR, ...)  |
                          |   Images saqlanadi        |
                          +---------------------------+
```

- **Docker Client (CLI):** Siz `docker` buyruqlarini yozadigan joy. U buyruqlarni Docker daemonga REST API orqali yuboradi.
- **Docker Daemon (dockerd):** Asosiy "miya". Imagelarni build qiladi, containerlarni ishga tushiradi, network va volumelarni boshqaradi.
- **Docker Registry:** Imagelar saqlanadigan ombor. Eng mashhuri — **Docker Hub**. Boshqalar: AWS ECR, Google GCR, GitHub Container Registry.

`docker run nginx` deganingizda quyidagilar sodir bo'ladi:

1. Client daemonga buyruqni yuboradi.
2. Daemon `nginx` imageni lokal qidiradi.
3. Topilmasa, registry'dan (Docker Hub) `pull` qiladi.
4. Image'dan container yaratadi va ishga tushiradi.

---

## Image vs Container

Bu eng ko'p chalkashtirib yuboriladigan tushuncha. Oddiy analogiya:

**💡 Tushuncha:** **Image** — bu "retsept" yoki "shablon" (class kabi). **Container** — bu shu retsept asosida tayyorlangan "taom" yoki ishlab turgan nusxa (object/instance kabi).

| Image | Container |
|-------|-----------|
| O'zgarmas (immutable) shablon | Image'ning ishlab turgan nusxasi |
| Diskda saqlanadi | Xotirada (RAM) ishlaydi |
| Class kabi | Object/instance kabi |
| `docker build` orqali yaratiladi | `docker run` orqali yaratiladi |
| Bittadan ko'p container yaratish mumkin | Har biri alohida hayot davriga ega |

```bash
# Bitta image'dan bir nechta container ishga tushirish mumkin
docker run -d --name web1 nginx
docker run -d --name web2 nginx
docker run -d --name web3 nginx
# Uchchalasi bir xil 'nginx' image'dan, lekin alohida containerlar
```

---

## Layer va Union Filesystem

Docker image'lar **layer**'lardan (qatlamlardan) tashkil topgan. Dockerfile'dagi har bir buyruq (`FROM`, `RUN`, `COPY`) yangi layer yaratadi.

**💡 Tushuncha:** Layerlar bir-birining ustiga "stack" qilinadi va **Union Filesystem** yordamida bitta yagona fayl tizimi sifatida ko'rinadi. Layerlar **read-only** (faqat o'qish uchun). Container ishga tushganda ustiga yupqa **read-write** layer (container layer) qo'shiladi.

```text
+------------------------------+  <- Container layer (read-write)
+------------------------------+
| Layer 4: CMD                 |  \
| Layer 3: COPY app code       |   |  Image layerlari
| Layer 2: RUN npm install     |   |  (read-only)
| Layer 1: FROM node:20-alpine |  /
+------------------------------+
```

**Layerlarning afzalligi — caching va ulashish:**

- Bir nechta image bir xil base layerni ulashishi mumkin (diskda bir marta saqlanadi).
- Build paytida o'zgarmagan layerlar qayta ishlatiladi (caching).
- Faqat o'zgargan layerlar qayta build/pull qilinadi.

---

## Dockerfile asoslari

`Dockerfile` — bu image qanday quriladigan ko'rsatmalar to'plami. Asosiy instruksiyalar:

| Instruksiya | Vazifasi |
|-------------|----------|
| `FROM` | Base image — qaysi image'dan boshlash |
| `WORKDIR` | Ishchi katalog o'rnatish |
| `COPY` | Fayllarni image ichiga nusxalash |
| `ADD` | COPY kabi, lekin URL va arxiv ham qo'llaydi |
| `RUN` | Build paytida buyruq bajarish (layer yaratadi) |
| `ENV` | Environment variable o'rnatish |
| `ARG` | Build paytidagi o'zgaruvchi |
| `EXPOSE` | Container qaysi portni tinglashini hujjatlash |
| `CMD` | Container ishga tushganda standart buyruq |
| `ENTRYPOINT` | Containerning asosiy bajariladigan buyrug'i |

### CMD vs ENTRYPOINT

Bu juda muhim farq:

- **`ENTRYPOINT`** — containerning "asosiy" buyrug'i. Odatda o'zgarmaydi.
- **`CMD`** — `ENTRYPOINT`'ga beriladigan standart argumentlar yoki standart buyruq. `docker run` paytida oson almashtiriladi.

```dockerfile
ENTRYPOINT ["node"]
CMD ["app.js"]
# `docker run myimg`        -> node app.js
# `docker run myimg test.js` -> node test.js  (CMD almashtirildi)
```

### ENV vs ARG

```dockerfile
ARG NODE_VERSION=20          # Faqat build paytida mavjud
FROM node:${NODE_VERSION}

ENV NODE_ENV=production      # Build paytida VA container ichida mavjud
```

**⚠️ Ehtiyot bo'l:** `ARG` faqat build vaqtida yashaydi va image ichida saqlanmaydi (faqat ko'rinib turishi mumkin). Hech qachon parol yoki maxfiy ma'lumotni `ARG` yoki `ENV` orqali bermang — ular image history'da qoladi.

---

## To'liq Node.js misol

Oddiy Express ilovasi uchun Dockerfile:

```dockerfile
# Base image: yengil alpine variantidan foydalanamiz
FROM node:20-alpine

# Ishchi katalogni o'rnatamiz
WORKDIR /app

# Avval faqat package fayllarini nusxalaymiz (caching uchun muhim!)
COPY package*.json ./

# Dependencylarni o'rnatamiz
RUN npm ci --only=production

# Endi qolgan kodni nusxalaymiz
COPY . .

# Ilova 3000-portni tinglaydi (hujjatlash maqsadida)
EXPOSE 3000

# Environment o'rnatamiz
ENV NODE_ENV=production

# Ilovani ishga tushiramiz
CMD ["node", "server.js"]
```

Mos `server.js`:

```text
const express = require('express');
const app = express();
app.get('/', (req, res) => res.send('Salom, Docker!'));
app.listen(3000, () => console.log('Server 3000-portda'));
```

---

## Image build va run

```bash
# Image build qilish (. = joriy katalogdagi Dockerfile)
docker build -t my-node-app:1.0 .

# Imagelarni ko'rish
docker images

# Containerni ishga tushirish (-d = background, -p = port mapping)
docker run -d -p 8080:3000 --name myapp my-node-app:1.0

# Ishlab turgan containerlarni ko'rish
docker ps

# Container loglarini ko'rish
docker logs myapp

# Container ichiga kirish
docker exec -it myapp sh

# Containerni to'xtatish va o'chirish
docker stop myapp
docker rm myapp
```

`-p 8080:3000` — host'ning 8080-porti container'ning 3000-portiga bog'lanadi. Endi `http://localhost:8080` orqali ilovaga kirasiz.

---

## Layer caching va optimizatsiya

Docker build paytida har bir layerni **cache**'laydi. Agar layer va undan oldingi hamma narsa o'zgarmagan bo'lsa, Docker cache'dan foydalanadi — bu build'ni ancha tezlashtiradi.

**Asosiy qoida:** Tez-tez o'zgaradigan narsalarni Dockerfile'ning **oxiriga**, kam o'zgaradiganlarni **boshiga** qo'ying.

**❌ Yomon (har o'zgarishda npm install qayta ishlaydi):**

```dockerfile
FROM node:20-alpine
WORKDIR /app
COPY . .                 # Kodning HAR o'zgarishi bu layerni buzadi
RUN npm ci               # => npm ci HAR safar qayta ishlaydi (sekin!)
CMD ["node", "server.js"]
```

**✅ Yaxshi (npm install faqat package.json o'zgarganda ishlaydi):**

```dockerfile
FROM node:20-alpine
WORKDIR /app
COPY package*.json ./    # Faqat package fayllari
RUN npm ci               # Cache'lanadi, kod o'zgarsa ham qayta ishlamaydi
COPY . .                 # Kod oxirida
CMD ["node", "server.js"]
```

**💡 Tushuncha:** `package.json` kamdan-kam o'zgaradi, kod esa tez-tez. Shuning uchun `npm ci` qatlamini kod nusxalashdan **oldin** qo'ysak, kod o'zgarganda ham `npm ci` cache'dan olinadi va build sekundlarda tugaydi.

---

## Multi-stage build

**Muammo:** Build uchun kerak bo'lgan asboblar (compiler, dev dependencies, build artifacts) final image'da kerak emas, lekin image hajmini oshiradi.

**Yechim — multi-stage build:** Bir Dockerfile ichida bir nechta `FROM` ishlatib, oraliq bosqichdan faqat kerakli natijani final image'ga nusxalaymiz.

```dockerfile
# 1-bosqich: BUILD (barcha dev asboblari bilan)
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build           # dist/ katalogini hosil qiladi

# 2-bosqich: PRODUCTION (faqat kerakli narsalar)
FROM node:20-alpine AS production
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
# Faqat build natijasini oldingi bosqichdan nusxalaymiz
COPY --from=builder /app/dist ./dist
EXPOSE 3000
CMD ["node", "dist/server.js"]
```

**Natija:** Final image'da dev dependencies, source code va build asboblari yo'q — faqat ishlab turgan ilova qoladi. Image hajmi bir necha barobar kichrayadi.

---

## .dockerignore

`.dockerignore` fayli `COPY . .` paytida nusxalanmasligi kerak bo'lgan fayllarni belgilaydi (`.gitignore` kabi).

```text
node_modules
npm-debug.log
.git
.gitignore
Dockerfile
.dockerignore
.env
*.md
dist
coverage
.vscode
```

**Nega muhim:**

- **Tezlik** — kamroq fayl nusxalanadi, build context kichrayadi.
- **Xavfsizlik** — `.env`, maxfiy fayllar image'ga tushmaydi.
- **To'g'rilik** — lokal `node_modules` (host platformaga moslangan) image'ga tushib, xato qilmaydi.

---

## Volume va Bind Mount

**Muammo:** Container o'chirilsa, uning ichidagi ma'lumotlar yo'qoladi (ephemeral). Ma'lumotlarni saqlash uchun **volume** kerak.

**💡 Tushuncha:** **Volume** — Docker tomonidan boshqariladigan, container hayotidan tashqari saqlanadigan ma'lumot ombori. **Bind mount** — host'dagi aniq katalogni container ichiga ulash.

```bash
# Named volume (Docker boshqaradi) — production'da tavsiya etiladi
docker volume create mydata
docker run -d -v mydata:/var/lib/mysql mysql:8

# Bind mount (host katalogi) — development'da qulay
docker run -d -v /home/user/app:/app my-node-app

# Development uchun: kod o'zgarishi container ichida darhol ko'rinadi
docker run -d -v $(pwd):/app -p 3000:3000 my-node-app
```

| | Volume | Bind Mount |
|---|--------|------------|
| Boshqaruv | Docker | Foydalanuvchi |
| Joylashuv | Docker boshqargan maydon | Host'dagi aniq yo'l |
| Tavsiya | Production (DB ma'lumotlari) | Development (live reload) |

**⚠️ Ehtiyot bo'l:** Bind mount bilan host yo'li xato bo'lsa, container kutilmagan ma'lumotni ko'rishi mumkin. Production DB uchun har doim named volume ishlating.

---

## Port mapping va Networking

Container o'zining izolyatsiyalangan tarmog'iga ega. Tashqaridan kirish uchun port'ni **map** qilish kerak.

```bash
# -p HOST_PORT:CONTAINER_PORT
docker run -p 8080:3000 my-node-app
# host:8080 -> container:3000
```

**Docker network turlari:**

- **bridge** (standart) — containerlar uchun izolyatsiyalangan ichki tarmoq.
- **host** — container host tarmog'ini to'g'ridan-to'g'ri ishlatadi.
- **none** — tarmoqsiz.

```bash
# Custom network yaratish (containerlar nom orqali bir-birini topadi)
docker network create myapp-net
docker run -d --name db --network myapp-net postgres
docker run -d --name api --network myapp-net my-node-app
# Endi 'api' container 'db' nomi orqali postgresga ulanadi
```

**💡 Tushuncha:** Bitta custom network ichidagi containerlar bir-birini **konteyner nomi** orqali topa oladi (DNS). Bu Docker Compose'ning asosiy mexanizmi.

---

## Docker Compose

**Muammo:** Real ilova bir nechta containerdan iborat — API, ma'lumotlar bazasi, cache (Redis), va h.k. Ularning har birini qo'lda `docker run` bilan ishga tushirish noqulay.

**Yechim — Docker Compose:** Barcha containerlar (services) bitta `docker-compose.yml` faylida ta'riflanadi va bitta buyruq bilan ishga tushiriladi.

```yaml
version: "3.9"

services:
  api:
    build: .
    ports:
      - "8080:3000"
    environment:
      - NODE_ENV=production
      - DATABASE_URL=postgres://user:pass@db:5432/mydb
    depends_on:
      - db
    networks:
      - app-net

  db:
    image: postgres:16-alpine
    environment:
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=pass
      - POSTGRES_DB=mydb
    volumes:
      - db-data:/var/lib/postgresql/data
    networks:
      - app-net

  redis:
    image: redis:7-alpine
    networks:
      - app-net

volumes:
  db-data:

networks:
  app-net:
```

```bash
# Hammasini ishga tushirish
docker compose up -d

# Loglarni kuzatish
docker compose logs -f

# Hammasini to'xtatish va o'chirish
docker compose down

# Volumelar bilan birga o'chirish
docker compose down -v
```

**💡 Tushuncha:** Compose ichida `api` container `db` xizmatiga **`db`** nomi orqali ulanadi (`DATABASE_URL=...@db:5432...`). Compose avtomatik network yaratadi va xizmat nomlarini DNS sifatida ishlatadi.

---

## Image hajmini kichraytirish

Kichik image — tezroq build, tezroq deploy, kamroq xavfsizlik yuzasi.

**1. Alpine base image ishlating:**

```dockerfile
FROM node:20            # ~1 GB
FROM node:20-alpine     # ~150 MB  <- yaxshiroq
```

**2. Multi-stage build** (yuqorida ko'rdik) — build asboblarini tashlab yuboradi.

**3. Layerlarni birlashtiring:**

```dockerfile
# ❌ Har RUN alohida layer
RUN apt-get update
RUN apt-get install -y curl
RUN rm -rf /var/lib/apt/lists/*

# ✅ Bitta layer, tozalash bilan
RUN apt-get update && \
    apt-get install -y curl && \
    rm -rf /var/lib/apt/lists/*
```

**4. `.dockerignore`** orqali keraksiz fayllarni tashlab yuboring.

**5. `--only=production`** bilan dev dependencylarni o'rnatmang.

---

## Best Practices

- **Non-root user ishlating** — xavfsizlik uchun container'ni root sifatida ishlatmang:

```dockerfile
FROM node:20-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
# Root bo'lmagan foydalanuvchi yaratamiz
USER node
CMD ["node", "server.js"]
```

- **Aniq versiya teglarini ishlating** — `node:20-alpine`, hech qachon `node:latest` emas.
- **`.dockerignore`** har doim qo'shing.
- **Layer tartibini optimallashtiring** — kam o'zgaradigani yuqorida.
- **Bir container = bitta vazifa** — bitta containerda nginx + mysql + app emas.
- **Multi-stage build** kichik production image uchun.
- **Maxfiy ma'lumotni image'ga qo'ymang** — runtime'da secret/env orqali bering.
- **HEALTHCHECK** qo'shing — container sog'lig'ini kuzatish uchun.

### Container vs Image lifecycle

```text
IMAGE lifecycle:                 CONTAINER lifecycle:
  build  -> image                  created -> running
  pull   -> image                  running -> paused/stopped
  push   -> registry               stopped -> running (restart)
  rmi    -> o'chirish              stopped -> removed (rm)
```

```bash
docker ps              # Ishlab turgan containerlar
docker ps -a           # Barcha (to'xtatilganlar ham)
docker images          # Barcha imagelar
docker rmi IMAGE       # Image o'chirish
docker system prune    # Ishlatilmayotgan resurslarni tozalash
```

---

## Savol-javoblar

### ❓ Container nima va u VM'dan qanday farq qiladi?

**✅ Javob:** Container — dastur va uning bog'liqliklarini izolyatsiyalangan paketga o'rovchi yengil texnologiya. Asosiy farq: VM o'zining to'liq guest OS'ini yuklaydi va hypervisor ustida ishlaydi (og'ir, GB hajm, daqiqalarda ishga tushadi), container esa host OS'ning kernelini ulashadi va faqat dastur + kutubxonalarni o'z ichiga oladi (yengil, MB hajm, soniyalarda ishga tushadi).

### ❓ "Works on my machine" muammosini Docker qanday hal qiladi?

**✅ Javob:** Docker dasturni aniq runtime, kutubxonalar, system tools va sozlamalar bilan birga bitta image'ga o'raydi. Bu image qayerda ishga tushirilsa ham, ichidagi muhit aynan bir xil bo'ladi. Shuning uchun "menda ishlayapti, serverda ishlamayapti" muammosi yo'qoladi — muhit har joyda bir xil.

### ❓ Image va Container o'rtasidagi farq nima?

**✅ Javob:** Image — o'zgarmas (immutable) shablon, "retsept" yoki class kabi; diskda saqlanadi va `docker build` orqali yaratiladi. Container — shu image'ning ishlab turgan nusxasi, object/instance kabi; `docker run` orqali yaratiladi. Bitta image'dan bir nechta container ishga tushirish mumkin.

### ❓ Docker arxitekturasining asosiy komponentlari qaysilar?

**✅ Javob:** Uchta: (1) **Docker Client (CLI)** — buyruqlar yoziladigan joy; (2) **Docker Daemon (dockerd)** — imagelar, containerlar, network va volumelarni boshqaradigan asosiy jarayon; (3) **Registry** — imagelar saqlanadigan ombor (Docker Hub, ECR, GCR). Client daemonga REST API orqali ulanadi.

### ❓ Docker layer nima va caching qanday ishlaydi?

**✅ Javob:** Dockerfile'dagi har bir instruksiya (`FROM`, `RUN`, `COPY`) yangi read-only layer yaratadi. Build paytida Docker har bir layerni cache'laydi. Agar layer va undan oldingi hamma narsa o'zgarmagan bo'lsa, Docker cache'dan foydalanadi — bu build'ni tezlashtiradi. Shuning uchun kam o'zgaradigan narsalar (npm install) Dockerfile boshida, tez o'zgaradiganlar (kod) oxirida bo'lishi kerak.

### ❓ CMD va ENTRYPOINT o'rtasidagi farq nima?

**✅ Javob:** `ENTRYPOINT` — containerning asosiy bajariladigan buyrug'i, odatda o'zgarmaydi. `CMD` — `ENTRYPOINT`'ga beriladigan standart argumentlar yoki standart buyruq, `docker run` paytida oson almashtiriladi. Masalan `ENTRYPOINT ["node"]` + `CMD ["app.js"]` bo'lsa, `docker run img test.js` natijasida `node test.js` ishlaydi.

### ❓ ENV va ARG o'rtasidagi farq nima?

**✅ Javob:** `ARG` faqat **build vaqtida** mavjud bo'ladi va container ichida yashamaydi. `ENV` esa build vaqtida ham, ishlab turgan container ichida ham mavjud. Ikkalasiga ham maxfiy ma'lumot bermaslik kerak — ular image history'da qoladi.

### ❓ Multi-stage build nima va nega kerak?

**✅ Javob:** Bu bitta Dockerfile ichida bir nechta `FROM` bosqichi ishlatish texnikasi. Build bosqichida barcha dev asboblar bilan dasturni compile qilamiz, keyin final bosqichga faqat tayyor natijani (`COPY --from=builder`) nusxalaymiz. Natijada final image'da build asboblari, dev dependencies va source code bo'lmaydi — image bir necha barobar kichrayadi.

### ❓ .dockerignore nima uchun kerak?

**✅ Javob:** U `COPY . .` paytida image'ga nusxalanmasligi kerak bo'lgan fayllarni belgilaydi (`.gitignore` kabi). Foydasi: build tezligini oshiradi (kamroq fayl), xavfsizlikni ta'minlaydi (`.env` image'ga tushmaydi), va host'dagi `node_modules` kabi platforma-bog'liq fayllarni image'ga tushishidan saqlaydi.

### ❓ Volume va bind mount o'rtasidagi farq nima?

**✅ Javob:** **Volume** — Docker tomonidan boshqariladigan, container hayotidan mustaqil saqlanadigan ma'lumot ombori; production'da (masalan DB ma'lumotlari uchun) tavsiya etiladi. **Bind mount** — host'dagi aniq katalogni container ichiga ulash; development'da live reload uchun qulay. Container o'chirilsa, volume'dagi ma'lumot saqlanib qoladi.

### ❓ Port mapping qanday ishlaydi?

**✅ Javob:** `docker run -p HOST_PORT:CONTAINER_PORT` orqali host'ning porti container'ning portiga bog'lanadi. Masalan `-p 8080:3000` — host'ning 8080-porti container'ning 3000-portiga yo'naltiriladi, shunda `http://localhost:8080` orqali ilovaga kirasiz. `EXPOSE` esa faqat hujjatlash maqsadida, port'ni avtomatik ochmaydi.

### ❓ Docker Compose nima uchun kerak?

**✅ Javob:** Real ilova bir nechta containerdan iborat (API, DB, Redis). Docker Compose ularning hammasini bitta `docker-compose.yml` faylida ta'riflaydi va `docker compose up` bitta buyruq bilan ishga tushiradi. Compose avtomatik network yaratadi, shunda xizmatlar bir-birini nomi orqali (DNS) topadi.

### ❓ Image hajmini qanday kichraytirish mumkin?

**✅ Javob:** (1) Alpine base image (`node:20-alpine`); (2) multi-stage build orqali build asboblarini tashlash; (3) `RUN` buyruqlarini `&&` bilan birlashtirib, bir layerda tozalash; (4) `.dockerignore` orqali keraksiz fayllarni tashlash; (5) `npm ci --only=production` bilan dev dependencylarni o'rnatmaslik.

### ❓ Nega container'ni root sifatida ishlatmaslik kerak?

**✅ Javob:** Xavfsizlik sababli. Agar container ichidagi dastur buzilsa va u root sifatida ishlayotgan bo'lsa, hujumchi ko'proq imkoniyatga ega bo'ladi (privilege escalation xavfi). Dockerfile'da `USER node` kabi root bo'lmagan foydalanuvchi yaratib, shu orqali ishga tushirish best practice hisoblanadi.

### ❓ Layer caching'ni optimizatsiya qilish uchun Dockerfile'ni qanday tartiblash kerak?

**✅ Javob:** Kam o'zgaradigan narsalarni yuqorida, tez-tez o'zgaradiganlarni pastda qo'yish kerak. Amalda: avval `COPY package*.json ./` va `RUN npm ci`, keyin `COPY . .`. Shunda kod o'zgarganda `npm ci` cache'dan olinadi va build tezlashadi. Agar `COPY . .` yuqorida bo'lsa, har bir kod o'zgarishida `npm ci` qayta ishlaydi.

---

## Masalalar

> Yechimlar: [`solutions/devops/03-docker.md`](../solutions/devops/03-docker.md)

1. **Birinchi Dockerfile.** Oddiy Node.js (yoki Python) "Hello World" web server uchun Dockerfile yozing. Image'ni build qiling, container ishga tushiring va brauzerda port mapping orqali tekshiring.

2. **Layer caching.** Berilgan "yomon" Dockerfile'ni (`COPY . .` dan keyin `RUN npm ci`) optimallashtiring. Kod o'zgartirib qayta build qiling va caching ishlayotganini `docker build` chiqishidan ko'rsating.

3. **Multi-stage build.** TypeScript yoki React loyihasi uchun multi-stage Dockerfile yozing. Build bosqichida `npm run build` qiling, final bosqichga faqat natijani nusxalang. `docker images` orqali oddiy va multi-stage image hajmlarini solishtiring.

4. **.dockerignore.** Loyihangizga `.dockerignore` qo'shing (`node_modules`, `.git`, `.env`). Build context hajmi qanchaga kamayganini kuzating.

5. **Volume bilan ma'lumot saqlash.** PostgreSQL containerini named volume bilan ishga tushiring. Bir nechta yozuv qo'shing, containerni o'chiring va qayta yarating — ma'lumot saqlanib qolganini tekshiring.

6. **Bind mount va live development.** Node.js ilovangizni `nodemon` bilan bind mount orqali ishga tushiring. Host'da kodni o'zgartiring va container ichida avtomatik qayta yuklanishini kuzating.

7. **Custom network.** Custom bridge network yarating. Ikkita container (API va Redis) shu network'da ishga tushiring va API'dan Redis'ga uning nomi orqali ulanishni sinab ko'ring.

8. **Docker Compose stack.** API + PostgreSQL + Redis'dan iborat `docker-compose.yml` yozing. `docker compose up` bilan ishga tushiring, `depends_on` va volume'larni qo'shing.

9. **Non-root user.** Mavjud Dockerfile'ga non-root `USER` qo'shing. Container ichiga kirib (`docker exec`), `whoami` orqali root emasligini tasdiqlang.

10. **Image hajmini kichraytirish.** Bitta ilova uchun ikkita Dockerfile yozing: bittasi `node:20`, ikkinchisi `node:20-alpine` + multi-stage bilan. Hajmlarni solishtiring va kamida 50% kichraytirishga erishing.

---

← [DevOps bo'limiga qaytish](./README.md)
