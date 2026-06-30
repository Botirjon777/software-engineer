# CI/CD — Yechimlar

Bu yerda [`devops/04-ci-cd.md`](../../devops/04-ci-cd.md) masalalarining yechimlari, kod va qisqa izohlar bilan keltirilgan.

## Mundarija

- [1-masala](#1-masala)
- [2-masala](#2-masala)
- [3-masala](#3-masala)
- [4-masala](#4-masala)
- [5-masala](#5-masala)
- [6-masala](#6-masala)
- [7-masala](#7-masala)
- [8-masala](#8-masala)
- [9-masala](#9-masala)
- [10-masala](#10-masala)

## 1-masala

```yaml
name: CI
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
      - run: npm ci
      - run: npm test
```

**Izoh:** `on` triggerlar — push va PR. `actions/checkout` kodni oladi, `setup-node` Node.js o'rnatadi. `npm ci` — lockfile asosida toza, takrorlanuvchan o'rnatish (CI uchun `npm install`dan afzal).

## 2-masala

```yaml
  build:
    needs: test
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Build Docker image
        run: docker build -t myapp:${{ github.sha }} .
```

**Izoh:** `needs: test` — `test` o'tmasa ishlamaydi (ketma-ketlik). `if: github.ref == 'refs/heads/main'` — faqat `main`'da deploy/build qilinadi, PR'larda emas.

## 3-masala

**Javob:** Ikkalasida ham build va test avtomatik. Farqi production'ga chiqish nuqtasida: **Continuous Delivery** — chiqarishni odam qo'lda tasdiqlaydi (tugma bosadi); **Continuous Deployment** — test'dan o'tgan o'zgarish to'liq avtomatik, hech qanday qo'lda qadam siz chiqadi.

**Izoh:** Yagona ajratuvchi savol: "production'ga chiqishdan oldin inson tasdig'i bormi?" Bor — Delivery, yo'q — Deployment.

## 4-masala

**Javob:** **Rolling deployment** — server'larni ketma-ket, kichik guruhlarga bo'lib yangilab, downtime'ni yo'qotadi. **Blue-green** — yangi versiyani alohida muhitga to'liq joylab, trafikni bir zumda o'tkazadi, downtime nol va rollback tez. **Canary** — yangi versiyani avval kichik qism foydalanuvchiga chiqarib, xavfni kamaytiradi.

**Izoh:** Asosiy muammo — "hammasini birvarakayiga" yangilash. Yechim — bosqichma-bosqich (rolling/canary) yoki ikkita parallel muhit (blue-green) orqali downtime'siz o'tish.

## 5-masala

```yaml
      - name: Upload coverage report
        uses: actions/upload-artifact@v4
        with:
          name: coverage-report
          path: coverage/
```

**Izoh:** `upload-artifact` action natija fayllarini saqlaydi. `name` — artifact nomi, `path` — saqlanadigan katalog/fayl. Keyinroq yuklab olish yoki boshqa job'da ishlatish mumkin.

## 6-masala

```yaml
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
```

**Izoh:** `setup-node` ning built-in `cache: 'npm'` opsiyasi `node_modules`'ni `package-lock.json` asosida cache'laydi. Lockfile o'zgarmasa, dependency'lar qaytadan yuklanmaydi — pipeline tezlashadi.

## 7-masala

```yaml
      - name: Deploy
        env:
          DEPLOY_TOKEN: ${{ secrets.DEPLOY_TOKEN }}
        run: ./deploy.sh
```

**Izoh:** Token GitHub Secrets'da saqlanadi va `${{ secrets.DEPLOY_TOKEN }}` orqali env o'zgaruvchi sifatida uzatiladi. **E'tibor:** uni hech qachon `echo` bilan log'ga chiqarmaslik va kodga/YAML'ga ochiq yozmaslik kerak.

## 8-masala

**Javob:** **Feature flag** (feature toggle) ishlatish kerak. Kod hammaga deploy qilinadi, lekin yangi funksiya bayroq orqali faqat ichki test foydalanuvchilar uchun yoqiladi.

**Izoh:** Feature flag **deploy**'ni **release**'dan ajratadi — kod production'da bo'lsa-da, funksiya tanlangan foydalanuvchilarga ko'rinadi. Oddiy deploy esa kodni chiqarishi bilan funksiyani hammaga ochib yuboradi.

## 9-masala

**Javob:** Har bir deploy'ni versiyalangan, o'zgarmas (immutable) artefakt sifatida chiqarish kerak edi (masalan `myapp:v1.2.0`). Shunda rollback — shunchaki oldingi tegni qayta ishga tushirish:

```bash
kubectl rollout undo deployment/myapp
# yoki aniq versiyaga:
kubectl set image deployment/myapp myapp=myapp:v1.2.0
```

**Izoh:** Blue-green strategiyasida rollback yanada tez — trafikni eski (blue) muhitga qaytarish kifoya. Database migration'lar backward-compatible bo'lishi ham shart, aks holda rollback ma'lumotni buzishi mumkin.

## 10-masala

**Javob:**

- **Tezlik:** Deployment Frequency (deploy chastotasi), Lead Time for Changes (commit'dan prod'gacha vaqt).
- **Barqarorlik:** Change Failure Rate (xato bilan tugagan deploy ulushi), Time to Restore Service / MTTR (tiklanish vaqti).

**Izoh:** Yuqori samarali jamoalar to'rttasida ham yaxshi ko'rsatkichga ega — ya'ni tez VA ishonchli deploy qiladi. Tezlik va barqarorlik bir-biriga zid emas.

← [DevOps bo'limiga qaytish](../../devops/README.md)
