# CI/CD

CI/CD — zamonaviy dasturlash jamoasining yuragi. Kodni qo'lda, qadam-baqadam serverga yuklash o'rniga, butun jarayonni avtomatlashtirish: test qilish, build qilish va deploy qilish. Bu bo'limda CI nima, CD ning ikki ma'nosi, pipeline bosqichlari, GitHub Actions misoli, deployment strategiyalari, secrets va DORA metrikalarini amaliy va intervyuga tayyorlovchi tarzda ko'rib chiqamiz.

**💡 Tushuncha:** CI/CD ning asosiy g'oyasi — "kichik o'zgarishlar, tez-tez, avtomatik". Kodni har kuni o'nlab marta xavfsiz yetkazib berish; har bir o'zgarishni avtomatik tekshirib, qo'lда xato qilish imkonini kamaytirish.

## Mundarija

- [CI nima](#ci-nima)
- [CD: Delivery vs Deployment](#cd-delivery-vs-deployment)
- [Nega CI/CD kerak](#nega-cicd-kerak)
- [Pipeline bosqichlari](#pipeline-bosqichlari)
- [Pipeline as code](#pipeline-as-code)
- [GitHub Actions to'liq misol](#github-actions-toliq-misol)
- [Artifact va caching](#artifact-va-caching)
- [Environment'lar](#environmentlar)
- [Deployment strategiyalari](#deployment-strategiyalari)
- [Rollback](#rollback)
- [Secrets management](#secrets-management)
- [Popular toollar](#popular-toollar)
- [DORA metrikalari](#dora-metrikalari)
- [Savol-javoblar](#savol-javoblar)
- [Masalalar](#masalalar)

## CI nima

CI (Continuous Integration / Uzluksiz integratsiya) — har bir dasturchi o'z kodini umumiy branch'ga tez-tez (kuniga bir necha marta) qo'shib, har bir qo'shilishda avtomatik test va tekshiruvlarni ishga tushirish amaliyoti.

Maqsad: integratsiya muammolarini erta topish. Agar har kim haftalab alohida ishlasa, oxirida kodni birlashtirganda katta konfliktlar va xatolar chiqadi ("integration hell"). CI buni oldini oladi — kichik, tez-tez integratsiya bilan.

CI tipik oqimi:

1. Dasturchi `push` qiladi yoki PR ochadi.
2. CI tizimi avtomatik ishga tushadi: kodni oladi, dependency o'rnatadi, lint va test qiladi.
3. Natija (yashil/qizil) PR'da ko'rinadi. Test o'tmasa, kod merge qilinmaydi.

**💡 Tushuncha:** CI ning asosiy qiymati — **tez fikr-mulohaza (feedback)**. Dasturchi xatoni kod yozgandan keyin bir necha daqiqada biladi, bir hafta keyin emas. Bu tuzatishni arzon va oson qiladi.

## CD: Delivery vs Deployment

CD ikki xil ma'noga ega va intervyuda farqi tez-tez so'raladi:

| | Continuous **Delivery** | Continuous **Deployment** |
|---|---|---|
| Build & test | avtomatik | avtomatik |
| Production'ga chiqarish | **qo'lda tasdiqlash** (tugma bosish) | **to'liq avtomatik** |
| Nazorat | inson oxirgi qarorni beradi | hech qanday qo'lda qadam yo'q |

- **Continuous Delivery:** har bir o'zgarish production'ga chiqarishga **tayyor** holatda turadi, lekin chiqarishni odam tasdiqlaydi (masalan, "Deploy" tugmasi).
- **Continuous Deployment:** test'lardan o'tgan har bir o'zgarish **avtomatik** production'ga chiqadi, hech kim tugma bosmaydi.

```text
CI  →  Build  →  Test  →  [Delivery: qo'lda tasdiq]  →  Production
CI  →  Build  →  Test  →  [Deployment: avtomatik]    →  Production
```

**💡 Tushuncha:** Ikkalasi ham "CD" deb qisqartiriladi — shuning uchun chalkashlik. Farqi faqat bitta nuqtada: production'ga chiqishdan oldin **inson tasdig'i bormi?** Bor bo'lsa — Delivery, yo'q bo'lsa — Deployment.

## Nega CI/CD kerak

- **Tezlik:** kod soatlar ichida (kunlar emas) foydalanuvchiga yetadi.
- **Ishonchlilik:** har bir o'zgarish avtomatik test qilinadi, qo'lda xato kamayadi.
- **Tez tuzatish:** xato chiqsa, kichik o'zgarishni tez orqaga qaytarish oson.
- **Takrorlanuvchanlik:** deploy jarayoni har safar bir xil, "menda ishlayapti" muammosi yo'qoladi.
- **Dasturchi vaqti:** qo'lda deploy o'rniga, jamoa kod yozishga e'tibor beradi.

**⚠️ Ehtiyot bo'l:** CI/CD sehrli tayoqcha emas. Agar test'lar yomon (yoki yo'q) bo'lsa, avtomatik pipeline shunchaki xatoni tezroq production'ga yetkazadi. CI/CD ning qiymati sifatli test qoplamiga bog'liq.

## Pipeline bosqichlari

Tipik CI/CD pipeline ketma-ket bosqichlardan iborat. Har biri o'tsa, keyingisi boshlanadi:

```text
checkout → install → lint → test → build → deploy
```

| Bosqich | Vazifasi |
|---------|----------|
| **checkout** | repozitoriyadan kodni olib kelish |
| **install** | dependency'larni o'rnatish (`npm ci`, `pip install`) |
| **lint** | kod uslubi va statik tahlil (`eslint`, `flake8`) |
| **test** | unit/integration test'larni ishga tushirish |
| **build** | bajariluvchi artefakt yasash (binary, Docker image) |
| **deploy** | artefaktni environment'ga chiqarish |

**💡 Tushuncha:** Bosqichlarni "tez va arzonlari oldin" tartibida joylash yaxshi amaliyot. `lint` sekundlar oladi, `test` daqiqalar — shuning uchun lint avval ishlasa, kod yomon formatlangan bo'lsa darrov to'xtaydi va sekin test'larni ishlatishga vaqt ketmaydi (fail fast).

## Pipeline as code

Zamonaviy CI/CD'da pipeline grafik interfeysda emas, balki **kod sifatida** — odatda YAML faylida — repozitoriya ichida saqlanadi.

Afzalliklari:

- pipeline o'zgarishlari git'da versiyalanadi va PR'da ko'rib chiqiladi;
- har bir branch o'z pipeline versiyasiga ega bo'lishi mumkin;
- yangi loyihaga ko'chirish — faylni nusxalash kifoya.

```yaml
# Soddalashtirilgan misol
name: CI
on: [push]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - run: npm ci
      - run: npm test
```

**💡 Tushuncha:** "Pipeline as code" — Infrastructure as Code falsafasining bir qismi. Konfiguratsiya kod bilan birga yashasa, u o'z-o'zini hujjatlaydi va takrorlanuvchan bo'ladi.

## GitHub Actions to'liq misol

GitHub Actions tushunchalari: **workflow** (`.github/workflows/*.yml` fayli) → **jobs** (parallel/ketma-ket ishlar) → **steps** (job ichidagi qadamlar) → **triggers** (`on`, qachon ishga tushishi).

```yaml
name: CI/CD Pipeline

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Lint
        run: npm run lint

      - name: Run tests
        run: npm test

  build-and-deploy:
    needs: test            # faqat test job'i o'tsa ishlaydi
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Build Docker image
        run: docker build -t myapp:${{ github.sha }} .

      - name: Deploy to production
        env:
          DEPLOY_KEY: ${{ secrets.DEPLOY_KEY }}
        run: ./deploy.sh
```

**💡 Tushuncha:** `needs: test` — `build-and-deploy` job'i `test` o'tmasa ishlamaydi (ketma-ketlik). `if:` — shartli ishga tushish (faqat `main` branch'da deploy). `${{ github.sha }}` — commit hash'i, har bir image'ni noyob teglash uchun.

**⚠️ Ehtiyot bo'l:** `uses: actions/checkout@v4` da versiyani (`@v4`) qotirib qo'y. `@main` yoki versiyasiz ishlatsang, action egasi kodni o'zgartirса, pipeline'ing kutilmaganda buziladi yoki xavfsizlik xavfi tug'iladi.

## Artifact va caching

**Artifact** — pipeline natijasida hosil bo'lgan fayl (binary, Docker image, test hisoboti). Job'lar orasida yoki keyinroq ishlatish uchun saqlanadi.

```yaml
      - name: Upload test report
        uses: actions/upload-artifact@v4
        with:
          name: coverage-report
          path: coverage/
```

**Caching** — har safar qaytadan yuklamaslik uchun dependency'larni saqlash. Bu pipeline'ni sezilarli tezlashtiradi.

```yaml
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'       # node_modules cache'lanadi
```

**💡 Tushuncha:** Artifact vs cache farqi: **artifact** — pipeline **natijasi** (build mahsuloti, sizga kerakli chiqish); **cache** — pipeline'ni **tezlashtirish** uchun oraliq saqlash (dependency'lar). Cache yo'qolsa muammo emas — qaytadan yaratiladi; artifact esa kerakli mahsulot.

## Environment'lar

Kod bir nechta muhitdan o'tib production'ga yetadi:

| Environment | Maqsad |
|-------------|--------|
| **dev** | dasturchining tezkor sinov maydoni |
| **staging** | production'ga o'xshash, oxirgi sinov muhiti |
| **prod** | haqiqiy foydalanuvchilar ishlatadigan muhit |

Har bir muhitda alohida konfiguratsiya (database, API kalit, domen) bo'ladi.

```yaml
jobs:
  deploy-staging:
    environment: staging
    runs-on: ubuntu-latest
    steps:
      - run: ./deploy.sh staging
```

**💡 Tushuncha:** `staging` muhiti production'ga iloji boricha o'xshash bo'lishi kerak (bir xil OS, bir xil baza versiyasi). Aks holda "staging'da ishlayapti, prod'da yo'q" muammosi paydo bo'ladi. Maqsad — prod'ga chiqishdan oldin xatolarni xavfsiz muhitda tutish.

## Deployment strategiyalari

Yangi versiyani production'ga chiqarishning bir necha usuli bor:

- **Rolling deployment:** server'larni ketma-ket, kichik guruhlarga bo'lib yangilash. Eski va yangi versiya bir muddat birga ishlaydi. Downtime yo'q, lekin orqaga qaytarish sekinroq.
- **Blue-green:** ikkita bir xil muhit (blue = eski, green = yangi). Yangi versiyani green'ga to'liq joylab, sinab ko'rib, keyin trafikni bir zumda green'ga o'tkazasiz. Rollback — trafikni blue'ga qaytarish, bir zumda.
- **Canary:** yangi versiyani avval foydalanuvchilarning kichik qismiga (masalan 5%) chiqarasiz. Hammasi yaxshi bo'lsa, asta-sekin 100% ga oshirasiz. Xato bo'lsa, faqat kichik qism ta'sirlanadi.
- **Feature flags:** kod deploy qilinadi, lekin yangi funksiya **bayroq** (sozlama) bilan yoqib/o'chiriladi. Deploy va "release"ni ajratadi — funksiyani kod chiqarmasdan yoqish/o'chirish mumkin.

```text
Rolling:    [v1][v1][v1] → [v2][v1][v1] → [v2][v2][v1] → [v2][v2][v2]
Blue-green: blue(v1) live → green(v2) tayyor → trafik green'ga → blue zaxira
Canary:     95% v1 / 5% v2 → 75/25 → 50/50 → 0/100
```

**💡 Tushuncha:** Blue-green — eng tez rollback (faqat trafikni qaytarasiz), lekin ikki barobar resurs kerak. Canary — eng xavfsiz, chunki xato kam odamga ta'sir qiladi, lekin sozlash murakkabroq. Feature flag — deploy'ni release'dan ajratib, marketing/biznesga nazorat beradi.

## Rollback

Rollback — yangi versiyada jiddiy xato chiqsa, oldingi ishonchli versiyaga qaytish.

```bash
# Docker tag orqali oldingi versiyaga qaytish
kubectl set image deployment/myapp myapp=myapp:v1.2.0
kubectl rollout undo deployment/myapp     # oldingi versiyaga
```

**💡 Tushuncha:** Yaxshi rollback strategiyasi — har bir deploy'ni o'zgarmas (immutable) versiyalangan artefakt qilish (masalan `myapp:v1.2.0`). Shunda "qaytarish" — shunchaki eski tegni ishga tushirish. Database o'zgarishlari (migration) rollback'ni murakkablashtiradi — ularni orqaga moslashuvchan (backward-compatible) qilish kerak.

**⚠️ Ehtiyot bo'l:** Database migration'larni rollback qilish xavfli — ustun o'chirilgan bo'lsa, ma'lumot yo'qolgan bo'lishi mumkin. Shuning uchun migration'larni "additive" (qo'shimcha) qilib, eski kod ham, yangi kod ham ishlaydigan tarzda yozish kerak (expand-contract pattern).

## Secrets management

Parol, API kalit, token kabi maxfiy ma'lumotlar **hech qachon** kodda yoki YAML'da ochiq yozilmaydi.

```yaml
      - name: Deploy
        env:
          API_KEY: ${{ secrets.API_KEY }}
          DB_PASSWORD: ${{ secrets.DB_PASSWORD }}
        run: ./deploy.sh
```

Secret'lar maxsus joyda saqlanadi: GitHub Secrets, HashiCorp Vault, AWS Secrets Manager, GitLab CI Variables.

**⚠️ Ehtiyot bo'l:** Secret'ni hech qachon log'ga chiqarma (`echo $API_KEY`). Aksariyat CI tizimlari secret qiymatlarini log'da avtomatik yashiradi (`***`), lekin shifrlash/kodlash bilan o'ralgan bo'lsa fosh bo'lishi mumkin. Shuningdek, `.env` faylni hech qachon git'ga commit qilma.

## Popular toollar

| Tool | Xususiyati |
|------|-----------|
| **GitHub Actions** | GitHub'ga integratsiyalangan, YAML, marketplace action'lar |
| **GitLab CI** | GitLab'ga o'rnatilgan, `.gitlab-ci.yml`, kuchli built-in registry |
| **Jenkins** | eng eski, o'z-o'zi hostlanadi, plagin ekotizimi keng, Groovy `Jenkinsfile` |

**💡 Tushuncha:** GitHub Actions va GitLab CI — bulutда, sozlash oson, repozitoriyaga bog'langan. Jenkins — moslashuvchan va kuchli, lekin o'zingiz server'da o'rnatib, qo'llab-quvvatlashingiz kerak. Yangi loyihalar ko'pincha GitHub Actions/GitLab CI'ni tanlaydi, eski korporativ tizimlarda Jenkins ko'p uchraydi.

## DORA metrikalari

DORA (DevOps Research and Assessment) — jamoaning deployment samaradorligini o'lchaydigan 4 ta asosiy metrika:

| Metrika | Nima o'lchaydi |
|---------|----------------|
| **Deployment Frequency** | qanchalik tez-tez deploy qilinadi (kuniga? haftasiga?) |
| **Lead Time for Changes** | kod commit'dan production'gacha qancha vaqt ketadi |
| **Change Failure Rate** | deploy'larning necha foizi xatoga olib keladi |
| **Time to Restore Service** | xatodan keyin tiklanish qancha vaqt oladi (MTTR) |

**💡 Tushuncha:** Birinchi ikkitasi — **tezlik** (qanchalik tez yetkazasiz), keyingi ikkitasi — **barqarorlik** (qanchalik ishonchli). Yuqori samarali jamoalar ikkalasida ham yaxshi — tez VA barqaror. "Tez deploy qilsak sifat tushadi" — afsona; aksincha, kichik tez-tez deploy'lar xavfsizroq.

## Savol-javoblar

### ❓ CI nima va u qanday muammoni hal qiladi?

**✅ Javob:** Continuous Integration — dasturchilar kodni umumiy branch'ga tez-tez qo'shib, har safar avtomatik test/lint ishlatadi. U "integration hell" — uzoq alohida ishlab, oxirida birlashtirganda chiqadigan katta konflikt va xatolarni hal qiladi. Tez feedback berib, xatoni erta va arzon topishga yordam beradi.

### ❓ Continuous Delivery va Continuous Deployment farqi nima?

**✅ Javob:** Ikkalasida ham build/test avtomatik. Farqi production'ga chiqishdа: **Delivery** — har o'zgarish chiqarishga tayyor turadi, lekin odam tugma bosib tasdiqlaydi; **Deployment** — test'dan o'tgan har o'zgarish to'liq avtomatik, hech qanday qo'lda qadam siz production'ga chiqadi.

### ❓ Tipik pipeline qanday bosqichlardan iborat?

**✅ Javob:** `checkout` (kod olish) → `install` (dependency) → `lint` (uslub/statik tahlil) → `test` (test'lar) → `build` (artefakt yasash) → `deploy` (chiqarish). Tez va arzon bosqichlar oldin qo'yiladi (fail fast).

### ❓ "Pipeline as code" nima va nega yaxshi?

**✅ Javob:** Pipeline'ni grafik interfeysda emas, repozitoriya ichida YAML fayl sifatida saqlash. Yaxshi, chunki o'zgarishlar git'da versiyalanadi, PR'da ko'rib chiqiladi, har bir branch o'z pipeline'iga ega bo'la oladi va boshqa loyihaga oson ko'chiriladi.

### ❓ GitHub Actions'da workflow, job va step nima?

**✅ Javob:** **Workflow** — `.github/workflows/` dagi YAML fayl, butun avtomatlashtirish. **Job** — workflow ichidagi mustaqil ish (standart holatda parallel ishlaydi, `needs` bilan ketma-ket). **Step** — job ichidagi alohida qadam (buyruq yoki action). Trigger (`on`) qachon ishga tushishini belgilaydi.

### ❓ Artifact va cache farqi nima?

**✅ Javob:** **Artifact** — pipeline natijasi/mahsuloti (build chiqishi, test hisoboti), saqlanadi va kerak bo'lganda ishlatiladi. **Cache** — pipeline'ni tezlashtirish uchun oraliq saqlash (dependency'lar). Cache yo'qolsa muammo emas (qayta yaratiladi), artifact esa kerakli chiqish.

### ❓ Caching nima uchun kerak?

**✅ Javob:** Har pipeline ishga tushganda dependency'larni (`node_modules`, `pip` paketlar) noldan yuklash sekin. Cache ularni saqlab, keyingi ishlarda qayta ishlatadi — pipeline'ni daqiqalarga tezlashtiradi. Faqat dependency fayllari (`package-lock.json`) o'zgarganda yangilanadi.

### ❓ dev, staging, prod muhitlari nima uchun kerak?

**✅ Javob:** Kodni bosqichma-bosqich sinash uchun. `dev` — tezkor ishlanma; `staging` — production'ga o'xshash oxirgi sinov; `prod` — haqiqiy foydalanuvchilar. Maqsad — xatolarni production'ga yetishidan oldin xavfsiz muhitda tutish.

### ❓ Blue-green va canary deployment farqi?

**✅ Javob:** **Blue-green** — ikki to'liq muhit; yangi versiyani green'ga joylab, trafikni bir zumda o'tkazasiz (tez rollback, lekin ikki barobar resurs). **Canary** — yangi versiyani avval kichik qism foydalanuvchiga (5%) chiqarib, asta oshirasiz (eng xavfsiz, xato kam odamga ta'sir qiladi).

### ❓ Feature flag nima va nima uchun ishlatiladi?

**✅ Javob:** Feature flag — kodni deploy qilib, lekin yangi funksiyani sozlama (bayroq) orqali yoqib/o'chirish. Bu **deploy**'ni **release**'dan ajratadi: kod production'da, lekin funksiya o'chiq turishi mumkin, keyin xohlagan vaqt kod chiqarmasdan yoqiladi. A/B test va xavfsiz tarqatish uchun ham foydali.

### ❓ Rollback nima va uni qanday osonlashtirasiz?

**✅ Javob:** Rollback — yangi versiyada xato chiqsa, oldingi ishonchli versiyaga qaytish. Osonlashtirish uchun: har deploy'ni versiyalangan o'zgarmas artefakt qilish (`myapp:v1.2.0`), shunda qaytarish — eski tegni ishga tushirish. Database migration'larni backward-compatible qilish ham muhim.

### ❓ Database migration rollback nega xavfli?

**✅ Javob:** Agar migration ustunni o'chirgan yoki tipini o'zgartirgan bo'lsa, orqaga qaytarganda ma'lumot yo'qolishi mumkin. Yechim — expand-contract pattern: avval qo'shimcha (additive) o'zgarish qilib, eski va yangi kod ikkalasi ham ishlaydigan holatni saqlash, keyin eski qismni alohida bosqichda olib tashlash.

### ❓ Secrets'ni qanday boshqarasiz?

**✅ Javob:** Parol, kalit, token kodda yoki YAML'da ochiq yozilmaydi. Maxsus secret store ishlatiladi: GitHub Secrets, Vault, AWS Secrets Manager. Pipeline'da ular o'zgaruvchi sifatida (`${{ secrets.API_KEY }}`) uzatiladi. Secret'ni log'ga chiqarmaslik va `.env`ni git'ga commit qilmaslik shart.

### ❓ GitHub Actions va Jenkins'ni qanday solishtirasiz?

**✅ Javob:** **GitHub Actions** — bulutda, GitHub'ga integratsiyalangan, sozlash oson, infratuzilmani boshqarish shart emas. **Jenkins** — o'z-o'zini hostlaydi, juda moslashuvchan va keng plagin ekotizimiga ega, lekin server'ni o'zingiz o'rnatib qo'llab-quvvatlashingiz kerak. Yangi loyihalar ko'pincha Actions/GitLab CI'ni, korporativ eski tizimlar Jenkins'ni ishlatadi.

### ❓ DORA metrikalari nima va nimani o'lchaydi?

**✅ Javob:** Jamoaning DevOps samaradorligini o'lchaydigan 4 metrika: Deployment Frequency va Lead Time for Changes — **tezlik**; Change Failure Rate va Time to Restore Service — **barqarorlik**. Yuqori samarali jamoa ikkalasida ham yaxshi: tez VA ishonchli deploy qiladi.

## Masalalar

> Yechimlar: [yechimlar fayli](../../solutions/devops/04-ci-cd.md)

1. GitHub Actions uchun shunday workflow yozing: `main` branch'ga `push` yoki PR ochilganda ishga tushsin, kodni olib, Node.js 20 o'rnatib, `npm ci` va `npm test` bajarsin.

2. Yuqoridagi workflow'ga shunday ikkinchi job qo'shing: u faqat `test` job'i muvaffaqiyatli o'tgandan keyin va faqat `main` branch'da ishlasin, hamda Docker image build qilsin.

3. Continuous Delivery va Continuous Deployment farqini bitta jumlada tushuntiring va qaysi nuqtada ajralishini ko'rsating.

4. Bir jamoa har bir deploy'da butun server parkini bir vaqtda yangilab, ba'zan to'liq downtime'ga uchrayapti. Qaysi deployment strategiyalarini tavsiya qilasiz va nega? Kamida ikkitasini ayting.

5. Workflow'da `coverage/` katalogidagi test hisobotini artifact sifatida saqlab qo'yadigan step yozing.

6. Pipeline har safar `npm install`'ni noldan bajarib, juda sekin ishlayapti. GitHub Actions'da buni tezlashtiradigan caching'ni qanday qo'shasiz? YAML qismini yozing.

7. `DEPLOY_TOKEN` nomli maxfiy kalitni deploy step'ida xavfsiz ishlatish kerak. Uni qanday uzatasiz va nimaga e'tibor berasiz?

8. Jamoa yangi funksiyani kodga qo'shdi, lekin uni faqat ichki test foydalanuvchilarga ko'rsatmoqchi, hozircha hammaga emas. Qaysi yondashuvni tavsiya qilasiz va u deploy'dan nimasi bilan farq qiladi?

9. Production'ga chiqqan yangi versiyada jiddiy bug topildi. Rollback'ni tez va xavfsiz qilish uchun deploy jarayonini qanday tashkil qilgan bo'lishingiz kerak edi? Tushuntiring.

10. Sizdan jamoa samaradorligini o'lchaydigan 4 ta DORA metrikasini nomlab, ularning qaysilari "tezlik", qaysilari "barqarorlik"ka tegishli ekanini guruhlash so'raldi.

← [DevOps bo'limiga qaytish](./README.md)
