# Docker va Containers — Yechimlar

Bu fayl [`devops/03-docker.md`](../../devops/03-docker.md) dagi masalalar yechimlarini o'z ichiga oladi. Har bir yechimda to'liq kod, izoh va tekshirish buyruqlari berilgan.

## Mundarija

- [1-masala: Birinchi Dockerfile](#1-masala-birinchi-dockerfile)
- [2-masala: Layer caching](#2-masala-layer-caching)
- [3-masala: Multi-stage build](#3-masala-multi-stage-build)
- [4-masala: .dockerignore](#4-masala-dockerignore)
- [5-masala: Volume bilan ma'lumot saqlash](#5-masala-volume-bilan-malumot-saqlash)
- [6-masala: Bind mount va live development](#6-masala-bind-mount-va-live-development)
- [7-masala: Custom network](#7-masala-custom-network)
- [8-masala: Docker Compose stack](#8-masala-docker-compose-stack)
- [9-masala: Non-root user](#9-masala-non-root-user)
- [10-masala: Image hajmini kichraytirish](#10-masala-image-hajmini-kichraytirish)

---

## 1-masala: Birinchi Dockerfile

`server.js`:

```text
const express = require('express');
const app = express();
app.get('/', (req, res) => res.send('Hello World, Docker!'));
app.listen(3000, () => console.log('Server 3000-portda ishlamoqda'));
```

`Dockerfile`:

```dockerfile
FROM node:20-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 3000
CMD ["node", "server.js"]
```

Build va run:

```bash
docker build -t hello-docker:1.0 .
docker run -d -p 8080:3000 --name hello hello-docker:1.0
# Brauzerda http://localhost:8080 oching
docker logs hello
```

**Izoh:** `-p 8080:3000` host'ning 8080-portini container'ning 3000-portiga bog'laydi. `package*.json` ni alohida nusxalash caching uchun (2-masalaga qarang).

---

## 2-masala: Layer caching

**Yomon variant (har o'zgarishda npm qayta ishlaydi):**

```dockerfile
FROM node:20-alpine
WORKDIR /app
COPY . .
RUN npm install        # Kod o'zgarsa, bu qayta ishlaydi
CMD ["node", "server.js"]
```

**Optimallashtirilgan variant:**

```dockerfile
FROM node:20-alpine
WORKDIR /app
COPY package*.json ./   # Faqat package fayllari
RUN npm install         # Kod o'zgarsa ham cache'lanadi
COPY . .                # Kod oxirida
CMD ["node", "server.js"]
```

Tekshirish:

```bash
docker build -t app:v1 .
# server.js da matnni o'zgartiring, qayta build qiling
docker build -t app:v2 .
# Chiqishda 'RUN npm install' qatorida 'CACHED' so'zini ko'rasiz
```

**Izoh:** `package.json` o'zgarmasa, `RUN npm install` layeri cache'dan olinadi. Build chiqishida `=> CACHED [3/5] RUN npm install` ko'rinadi va build sekundlarda tugaydi.

---

## 3-masala: Multi-stage build

`Dockerfile` (TypeScript/React misol):

```dockerfile
# 1-bosqich: build
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build          # dist/ yoki build/ hosil qiladi

# 2-bosqich: production
FROM node:20-alpine AS production
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY --from=builder /app/dist ./dist
EXPOSE 3000
CMD ["node", "dist/server.js"]
```

Solishtirish:

```bash
docker build -t app:full -f Dockerfile.full .       # multi-stage'siz
docker build -t app:multi .                          # multi-stage bilan
docker images | grep app
# app:multi sezilarli kichikroq bo'ladi
```

**Izoh:** `builder` bosqichida barcha dev dependencies va build asboblari bor, lekin `production` bosqichga faqat `dist/` nusxalanadi. Final image'da source code va dev asboblar yo'q.

---

## 4-masala: .dockerignore

`.dockerignore`:

```text
node_modules
.git
.gitignore
.env
*.md
dist
coverage
npm-debug.log
.vscode
Dockerfile
.dockerignore
```

Tekshirish:

```bash
# .dockerignore'siz build context hajmini ko'ring
docker build -t app:test .
# Chiqishda "transferring context: XX MB" ko'rinadi
# .dockerignore qo'shilgach context ancha kichrayadi
```

**Izoh:** `node_modules` (eng katta), `.git` va `.env` tashlab yuborilgani uchun build context megabaytlardan kilobaytlarga tushishi mumkin. Bu build tezligini va xavfsizlikni oshiradi.

---

## 5-masala: Volume bilan ma'lumot saqlash

```bash
# Named volume yaratish
docker volume create pgdata

# PostgreSQL'ni volume bilan ishga tushirish
docker run -d --name pg \
  -e POSTGRES_PASSWORD=secret \
  -v pgdata:/var/lib/postgresql/data \
  -p 5432:5432 postgres:16-alpine

# Ma'lumot qo'shish
docker exec -it pg psql -U postgres -c \
  "CREATE TABLE users(id int); INSERT INTO users VALUES(1),(2);"

# Containerni o'chirib, qayta yaratish
docker rm -f pg
docker run -d --name pg \
  -e POSTGRES_PASSWORD=secret \
  -v pgdata:/var/lib/postgresql/data \
  -p 5432:5432 postgres:16-alpine

# Ma'lumot saqlanib qolganini tekshirish
docker exec -it pg psql -U postgres -c "SELECT * FROM users;"
# Natija: 1, 2 — ma'lumot saqlanib qolgan!
```

**Izoh:** Container o'chirilganda ham `pgdata` volume saqlanadi, shuning uchun yangi container eski ma'lumotni ko'radi.

---

## 6-masala: Bind mount va live development

`Dockerfile.dev`:

```dockerfile
FROM node:20-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
CMD ["npx", "nodemon", "server.js"]
```

Ishga tushirish:

```bash
docker build -t app:dev -f Dockerfile.dev .
docker run -d --name dev-app \
  -v $(pwd):/app \
  -v /app/node_modules \
  -p 3000:3000 app:dev

# Host'da server.js ni o'zgartiring -> nodemon avtomatik qayta yuklaydi
docker logs -f dev-app
```

**Izoh:** `-v $(pwd):/app` host kodini container ichiga ulaydi. `-v /app/node_modules` — anonymous volume bilan container'ning `node_modules`'ini host'nikidan himoyalaydi (host'da node_modules bo'lmasligi yoki boshqa platformaniki bo'lishi mumkin).

---

## 7-masala: Custom network

```bash
# Custom bridge network yaratish
docker network create app-net

# Redis'ni network'da ishga tushirish
docker run -d --name redis --network app-net redis:7-alpine

# API'ni shu network'da ishga tushirish
docker run -d --name api --network app-net -p 3000:3000 my-api

# API ichidan Redis'ga nomi orqali ulanishni tekshirish
docker exec -it api sh -c "ping -c 2 redis"
# 'redis' nomi DNS orqali Redis container IP'siga aylanadi
```

API kodida ulanish: `redis://redis:6379` (host nomi sifatida container nomi `redis`).

**Izoh:** Bitta custom network ichidagi containerlar bir-birini **nomi** orqali topadi. Default `bridge` network'da bu ishlamaydi — custom network kerak.

---

## 8-masala: Docker Compose stack

`docker-compose.yml`:

```yaml
version: "3.9"

services:
  api:
    build: .
    ports:
      - "8080:3000"
    environment:
      - DATABASE_URL=postgres://user:pass@db:5432/mydb
      - REDIS_URL=redis://redis:6379
    depends_on:
      - db
      - redis

  db:
    image: postgres:16-alpine
    environment:
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=pass
      - POSTGRES_DB=mydb
    volumes:
      - db-data:/var/lib/postgresql/data

  redis:
    image: redis:7-alpine

volumes:
  db-data:
```

```bash
docker compose up -d
docker compose ps
docker compose logs -f api
docker compose down        # to'xtatish
docker compose down -v     # volume bilan o'chirish
```

**Izoh:** Compose avtomatik network yaratadi, shuning uchun `api` xizmat `db` va `redis` nomlari orqali ulanadi. `depends_on` ishga tushish tartibini belgilaydi (lekin DB tayyorligini kafolatlamaydi — healthcheck kerak bo'lishi mumkin).

---

## 9-masala: Non-root user

`Dockerfile`:

```dockerfile
FROM node:20-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
# node:alpine image'da 'node' foydalanuvchi oldindan mavjud
USER node
EXPOSE 3000
CMD ["node", "server.js"]
```

Tekshirish:

```bash
docker build -t app:secure .
docker run -d --name secure-app app:secure
docker exec -it secure-app whoami
# Natija: node  (root emas!)
```

**Izoh:** `node:alpine` image'da `node` nomli root bo'lmagan foydalanuvchi tayyor. Agar custom foydalanuvchi kerak bo'lsa: `RUN addgroup -S app && adduser -S app -G app` keyin `USER app`.

---

## 10-masala: Image hajmini kichraytirish

**Katta variant** (`Dockerfile.full`):

```dockerfile
FROM node:20
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build
CMD ["node", "dist/server.js"]
```

**Kichik variant** (`Dockerfile`, alpine + multi-stage):

```dockerfile
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM node:20-alpine AS production
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY --from=builder /app/dist ./dist
USER node
CMD ["node", "dist/server.js"]
```

Solishtirish:

```bash
docker build -t app:full -f Dockerfile.full .
docker build -t app:slim .
docker images | grep app
# app:full  ~1.1 GB
# app:slim  ~180 MB   => 80%+ kichrayish
```

**Izoh:** Kichraytirish uchta usul bilan amalga oshdi: (1) `node:20` o'rniga `node:20-alpine` (~1GB -> ~150MB base); (2) multi-stage build dev dependencies va source code'ni tashladi; (3) `--only=production` faqat production dependencies'ni o'rnatdi.

---

← [DevOps bo'limiga qaytish](../../devops/README.md)
