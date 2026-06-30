# Git va Version Control

Git — bugungi kunda dasturchining eng asosiy ish quroli. Kod ustida jamoaviy ishlash, o'zgarishlar tarixini saqlash va xatolarni orqaga qaytarish — barchasi Git orqali. Bu bo'limda version control nazariyasi, Git ichki tushunchalari va kundalik buyruqlardan tortib `rebase`, `reflog`, `cherry-pick` kabi murakkab mavzulargacha ko'rib chiqamiz.

**💡 Tushuncha:** Git'ni shunchaki "buyruqlar to'plami" deb yodlash o'rniga, uning ichida nima sodir bo'layotganini (commit graph, HEAD, indeks) tushunsang, har qanday vaziyatdan chiqib keta olasan.

## Mundarija

- [Version control nima](#version-control-nima)
- [Git va distributed VCS](#git-va-distributed-vcs)
- [Asosiy tushunchalar](#asosiy-tushunchalar)
- [Asosiy buyruqlar](#asosiy-buyruqlar)
- [Branching](#branching)
- [Merge vs Rebase](#merge-vs-rebase)
- [Merge conflict](#merge-conflict)
- [Remote bilan ishlash](#remote-bilan-ishlash)
- [Git workflow'lar](#git-workflowlar)
- [.gitignore](#gitignore)
- [O'zgarishlarni bekor qilish (undo)](#ozgarishlarni-bekor-qilish-undo)
- [Stash, cherry-pick, tag](#stash-cherry-pick-tag)
- [Interactive rebase va squash](#interactive-rebase-va-squash)
- [Detached HEAD va reflog](#detached-head-va-reflog)
- [Pull request jarayoni](#pull-request-jarayoni)
- [Savol-javoblar](#savol-javoblar)
- [Masalalar](#masalalar)

## Version control nima

Version control (VCS — Version Control System) — bu fayllaringizdagi o'zgarishlarni vaqt o'tishi bilan kuzatib boruvchi tizim. Har bir o'zgarishni "snapshot" sifatida saqlaydi, shuning uchun:

- istalgan oldingi holatga qaytishingiz mumkin;
- kim, qachon, nima uchun o'zgartirganini bilasiz;
- bir nechta odam bir loyihada parallel ishlay oladi.

VCS'siz odamlar `loyiha_final.zip`, `loyiha_final_v2.zip`, `loyiha_haqiqiy_final.zip` kabi fayllar yaratadi — bu xaos. Git esa bularning hammasini bitta tartibli tarix bilan almashtiradi.

**💡 Tushuncha:** Centralized VCS (masalan, SVN) bitta markaziy serverga bog'liq — server o'chsa, ishlay olmaysiz. Git distributed (taqsimlangan), ya'ni har bir dasturchida loyihaning to'liq nusxasi va to'liq tarixi bor.

## Git va distributed VCS

Git — Linus Torvalds tomonidan 2005-yilda Linux yadrosi uchun yaratilgan distributed version control system (DVCS).

Distributed bo'lishining afzalliklari:

- **Offline ishlash:** commit, branch, history ko'rish — hammasi internetsiz, lokal ishlaydi.
- **Tezlik:** ko'p amallar lokal disk bilan ishlagani uchun juda tez.
- **Ishonchlilik:** har bir clone — to'liq backup. Markaziy server yo'qolsa ham, har bir nusxada butun tarix bor.

```bash
git --version
git config --global user.name "Ali Valiyev"
git config --global user.email "ali@example.com"
```

**⚠️ Ehtiyot bo'l:** `user.name` va `user.email` har bir commit ichiga yoziladi. Ish va shaxsiy hisoblarni aralashtirmaslik uchun bularni to'g'ri sozlab oling.

## Asosiy tushunchalar

Git'ning eng muhim modeli — uchta "joy" (area):

| Joy | Nomi | Vazifasi |
|-----|------|----------|
| Working Directory | Ish katalogi | Hozir tahrirlayotgan haqiqiy fayllaringiz |
| Staging Area | Index | Keyingi commit'ga kiritmoqchi bo'lgan o'zgarishlar to'plami |
| Repository | `.git` katalogi | Commit qilingan barcha tarix saqlanadigan joy |

Ish oqimi: fayl tahrir qilasan (working directory) → `git add` bilan staging'ga qo'shasan → `git commit` bilan repository'ga yozasan.

Boshqa muhim atamalar:

- **Commit** — o'zgarishlar snapshot'i. Har biri unikal SHA-1 hash (masalan, `a1b2c3d`) bilan belgilanadi va ota commit(lar)ga ishora qiladi.
- **HEAD** — hozir qayerda turganingizni ko'rsatuvchi pointer. Odatda joriy branch'ning oxirgi commit'iga ishora qiladi.

**💡 Tushuncha:** Commit — bu "diff" emas, balki butun loyihaning o'sha ondagi to'liq holati (snapshot). Git tejamkorlik uchun ichki tomondan o'zgarmagan fayllarga qayta ishora qiladi.

## Asosiy buyruqlar

```bash
git init                 # joriy katalogda yangi repository yaratish
git clone <url>          # mavjud repozitoriyni nusxalash
git status               # working dir va staging holatini ko'rish
git add <fayl>           # faylni staging'ga qo'shish
git add .                # barcha o'zgarishlarni staging'ga qo'shish
git commit -m "xabar"    # staging'dagini repository'ga yozish
git log                  # commit tarixini ko'rish
git log --oneline --graph --all   # ixcham, grafik ko'rinishda
git diff                 # working dir va staging orasidagi farq
git diff --staged        # staging va oxirgi commit orasidagi farq
git show <commit>        # ma'lum commit tafsilotlari
```

**💡 Tushuncha:** `git status` — eng ko'p ishlatiladigan buyruq. Adashganingda doim shu bilan boshla: u qaysi holatdaligingni va keyingi qadam uchun maslahatlarni ko'rsatadi.

## Branching

Branch (tarmoq) — bu commit'ga ishora qiluvchi yengil, ko'chuvchi pointer. Yangi branch yaratish deyarli "bepul" amaldir.

```bash
git branch                    # mavjud branch'lar ro'yxati
git branch feature-login      # yangi branch yaratish
git checkout feature-login    # branch'ga o'tish (eski usul)
git switch feature-login      # branch'ga o'tish (yangi, aniqroq usul)
git switch -c feature-login   # yaratib, darrov o'tish
git branch -d feature-login   # branch'ni o'chirish (merge qilingan bo'lsa)
git branch -D feature-login   # majburan o'chirish
```

**💡 Tushuncha:** `git switch` va `git restore` yangiroq buyruqlar bo'lib, eski `git checkout`ning ikki xil vazifasini (branch almashtirish va fayl tiklash) ikkita aniq buyruqqa ajratdi. Yangi loyihalarda shularni ishlatish tavsiya etiladi.

## Merge vs Rebase

Ikkala buyruq ham bir branch'dagi o'zgarishlarni boshqasiga olib kelish uchun, lekin tarixni har xil shakllantiradi.

**Merge:** ikki branch tarixini birlashtirib, yangi "merge commit" yaratadi. Tarix qanday bo'lgan bo'lsa, shundayligicha qoladi (haqqoniy, lekin ba'zan chigal).

```bash
git switch main
git merge feature-login
```

**Rebase:** sizning commit'laringizni boshqa branch uchiga "ko'chirib", chiziqli (linear) tarix yasaydi.

```bash
git switch feature-login
git rebase main
```

**Qachon qaysi:**

- Lokal, hali push qilinmagan branch'ni tozalash uchun → `rebase`.
- Umumiy `main`'ga feature'ni qo'shish va aniq tarix kerak bo'lsa → `merge`.

**⚠️ Ehtiyot bo'l — Golden Rule of Rebasing:** Boshqalar bilan baham ko'rilgan (push qilingan, umumiy) branch'ni HECH QACHON `rebase` qilmang. Rebase commit'larni qayta yozadi (yangi hash'lar), bu esa boshqa odamlarning tarixini buzadi.

## Merge conflict

Conflict — ikki branch bir faylning bir qatorini har xil o'zgartirsa yuzaga keladi. Git o'zi hal qila olmaydi va sizdan qaror kutadi.

Conflict bo'lgan fayl ichida:

```text
<<<<<<< HEAD
joriy branch'dagi kod
=======
kelayotgan branch'dagi kod
>>>>>>> feature-login
```

Hal qilish qadamlari:

```bash
# 1. Konfliktli fayllarni ko'rish
git status
# 2. Faylni qo'lda tahrirlab, kerakli kodni qoldirish (markerlarni o'chirib)
# 3. Hal qilingan faylni belgilash
git add <fayl>
# 4. Davom ettirish
git commit          # merge uchun
git rebase --continue   # rebase uchun
# Yoki butunlay bekor qilish:
git merge --abort
git rebase --abort
```

**💡 Tushuncha:** `git config merge.conflictstyle diff3` qo'shsangiz, conflict markerlarida "umumiy ota" (base) versiyasi ham ko'rinadi — bu nima o'zgarganini tushunishni osonlashtiradi.

## Remote bilan ishlash

Remote — repozitoriyaning internetdagi (GitHub, GitLab) nusxasi. Standart remote nomi — `origin`.

```bash
git remote -v                       # remote'larni ko'rish
git remote add origin <url>         # remote qo'shish
git push origin main                # lokal commit'larni remote'ga yuborish
git push -u origin main             # upstream'ni eslab qolib yuborish
git pull origin main                # remote'dan olib, merge qilish (fetch + merge)
git fetch origin                    # remote o'zgarishlarini olish (merge qilmasdan)
```

**💡 Tushuncha:** `fetch` — xavfsiz: ma'lumotni yuklab oladi, lekin sizning ishingizga tegmaydi. `pull` esa `fetch` + `merge` (yoki `pull --rebase` bilan `fetch` + `rebase`).

- **origin** — odatda sizning fork'ingiz yoki asosiy repo.
- **upstream** — fork qilingan loyihalarda asl (original) repozitoriya uchun qo'yiladigan nom.

**⚠️ Ehtiyot bo'l:** `git push --force` remote tarixni qayta yozadi va jamoadoshlaringizning ishini yo'qotishi mumkin. Zarur bo'lsa `--force-with-lease` ishlating — u kimdir siz bilmagan holda push qilgan bo'lsa, to'xtaydi.

## Git workflow'lar

| Workflow | Mohiyati | Qachon |
|----------|----------|--------|
| **Git Flow** | `main`, `develop`, `feature/*`, `release/*`, `hotfix/*` — ko'p uzoq yashaydigan branch'lar | Rejali, versiyalangan релизли mahsulotlar |
| **GitHub Flow** | Bitta `main` + qisqa umrli feature branch'lar + PR | Tez-tez deploy qilinadigan veb-loyihalar |
| **Trunk-based** | Hamma to'g'ridan-to'g'ri `main`'ga (yoki juda qisqa branch'lar) tez-tez qo'shadi | CI/CD yetuk, feature flag'lar bor jamoalar |

**💡 Tushuncha:** Zamonaviy jamoalarda trunk-based development va GitHub Flow ko'proq qo'llaniladi, chunki ular tez yetkazib berish (continuous delivery) bilan yaxshi mos keladi. Git Flow esa murakkabroq va sekinroq релиз sikllariga mos.

## .gitignore

`.gitignore` fayli Git kuzatmasligi kerak bo'lgan fayllarni belgilaydi: build artifaktlar, log fayllar, sirlar, IDE sozlamalari.

```text
# Dependencies
node_modules/
vendor/

# Build chiqishi
dist/
build/
*.class

# Muhit va sirlar
.env
*.key

# IDE
.idea/
.vscode/

# Loglar
*.log
```

**⚠️ Ehtiyot bo'l:** `.gitignore` faqat hali kuzatilmagan (untracked) fayllarga ta'sir qiladi. Agar fayl allaqachon commit qilingan bo'lsa, uni avval `git rm --cached <fayl>` bilan kuzatuvdan chiqarish kerak.

## O'zgarishlarni bekor qilish (undo)

Bu Git'dagi eng chalkash mavzu. Asosiy buyruqlarni farqi:

```bash
# reset — HEAD'ni boshqa commit'ga ko'chiradi
git reset --soft HEAD~1    # commit'ni bekor qiladi, o'zgarishlar STAGING'da qoladi
git reset --mixed HEAD~1   # (default) commit'ni bekor qiladi, o'zgarishlar WORKING DIR'da qoladi
git reset --hard HEAD~1    # commit VA o'zgarishlarni butunlay o'chiradi (XAVFLI)

# revert — yangi commit yaratib, eski commit ta'sirini bekor qiladi
git revert <commit>        # tarixni saqlaydi, umumiy branch uchun xavfsiz

# restore — fayllarni tiklash
git restore <fayl>          # working dir'dagi o'zgarishni bekor qilish
git restore --staged <fayl> # staging'dan chiqarish (commit'ga teginmaydi)
```

| Buyruq | Tarixni o'zgartiradimi | Qachon |
|--------|------------------------|--------|
| `reset` | Ha (commit'larni olib tashlaydi) | Lokal, push qilinmagan o'zgarishlar |
| `revert` | Yo'q (yangi commit qo'shadi) | Umumiy, push qilingan commit'ni bekor qilish |
| `restore` | Yo'q | Faqat fayl mazmunini tiklash |

**⚠️ Ehtiyot bo'l:** `git reset --hard` commit qilinmagan ishni qaytarib bo'lmaydigan tarzda o'chiradi. Ishlatishdan oldin `git status` bilan nima yo'qolishini tekshiring.

## Stash, cherry-pick, tag

**Stash** — yarim tugagan ishni vaqtincha "javonga" qo'yib turish:

```bash
git stash             # joriy o'zgarishlarni yashirish
git stash list        # yashirilganlar ro'yxati
git stash pop         # oxirgisini qaytarish va o'chirish
git stash apply       # qaytarish, lekin stash'da qoldirish
```

**Cherry-pick** — boshqa branch'dagi bitta aniq commit'ni joriy branch'ga olib kelish:

```bash
git cherry-pick <commit-hash>
```

**Tag** — muhim nuqtalarni (релизларни) belgilash:

```bash
git tag v1.0.0                          # yengil tag
git tag -a v1.0.0 -m "Birinchi релиз"   # izohli (annotated) tag
git push origin v1.0.0                  # tag'ni remote'ga yuborish
```

**💡 Tushuncha:** Stash juda foydali: shoshilinch bug tuzatish kerak bo'lganda, hali tayyor bo'lmagan ishni commit qilmasdan vaqtincha yig'ishtirib qo'yish imkonini beradi.

## Interactive rebase va squash

Interactive rebase (`rebase -i`) — commit tarixini tahrirlash uchun kuchli vosita: commit'larni birlashtirish (squash), nomini o'zgartirish (reword), tartibini almashtirish yoki o'chirish.

```bash
git rebase -i HEAD~3    # oxirgi 3 commit'ni tahrirlash
```

Ochilgan oynada har bir commit oldida buyruq turadi:

```text
pick   a1b2c3d Birinchi commit
squash e4f5g6h Ikkinchi commit   # oldingisiga qo'shib yuboradi
reword i7j8k9l Uchinchi commit   # xabarini o'zgartiradi
```

**Squash** — bir nechta mayda commit'ni bitta toza commit'ga birlashtirish. PR yuborishdan oldin "wip", "fix typo" kabi commit'larni yig'ishtirish uchun ideal.

**⚠️ Ehtiyot bo'l:** `rebase -i` ham tarixni qayta yozadi. Faqat lokal yoki sizdan boshqa hech kim ishlatmaydigan branch'larda qo'llang.

## Detached HEAD va reflog

**Detached HEAD** — HEAD branch'ga emas, to'g'ridan-to'g'ri commit'ga ishora qilgan holat. Bu odatda `git checkout <commit-hash>` qilganda yuzaga keladi.

```bash
git checkout a1b2c3d   # detached HEAD holatiga tushasiz
```

Bu holatda yangi commit qilsangiz va branch yaratmasangiz, branch almashtirgach o'sha commit'lar "yo'qoladi" (aslida reflog'da qoladi). Ishni saqlash uchun:

```bash
git switch -c yangi-branch   # joriy holatdan yangi branch yarating
```

**Reflog** — HEAD qayerlarga harakatlangani jurnali. "Yo'qolgan" commit'larni topishning hayotni saqlovchi vositasi:

```bash
git reflog                    # HEAD harakatlari tarixi
git reset --hard HEAD@{2}     # 2 qadam oldingi holatga qaytish
```

**💡 Tushuncha:** Hatto `reset --hard` qilib ishni "yo'qotgan" bo'lsang ham, panika qilma — `git reflog` deyarli har doim uni qaytarib topishga yordam beradi (commit'lar darrov o'chmaydi).

## Pull request jarayoni

Pull Request (PR) yoki Merge Request (MR) — o'zgarishlaringizni asosiy branch'ga qo'shishdan oldin ko'rib chiqishga (code review) yuborish mexanizmi.

Tipik jarayon:

```bash
# 1. main'dan yangi branch ochish
git switch -c feature/yangi-tugma
# 2. Ishlash va commit qilish
git add .
git commit -m "feat: yangi tugma qo'shildi"
# 3. Remote'ga push qilish
git push -u origin feature/yangi-tugma
# 4. GitHub/GitLab'da PR ochish (web interfeys orqali)
# 5. Review, muhokama, o'zgarishlar
# 6. Approve bo'lgach, main'ga merge qilinadi
```

**💡 Tushuncha:** Yaxshi PR — kichik, bitta vazifaga qaratilgan, aniq tavsifga ega bo'ladi. 1000 qatorlik PR'ni hech kim sifatli review qila olmaydi.

## Savol-javoblar

### ❓ Git va GitHub orasidagi farq nima?

**✅ Javob:** Git — bu lokal ishlaydigan version control tizimi (dastur). GitHub — bu Git repozitoriyalarini hostlaydigan veb-platforma (xizmat), unga qo'shimcha PR, issue, CI/CD kabi imkoniyatlar qo'shilgan. Git'siz ham ishlash mumkin, lekin GitHub Git ustiga qurilgan.

### ❓ `git fetch` va `git pull` farqi nimada?

**✅ Javob:** `git fetch` remote'dan o'zgarishlarni faqat yuklab oladi, lokal branch'ingizga tegmaydi — xavfsiz. `git pull` esa `fetch` + `merge` (yoki `--rebase` bilan `fetch` + `rebase`), ya'ni darrov sizning branch'ingizga qo'shib yuboradi. Avval `fetch` qilib, nima kelganini ko'rib, keyin merge qilish ehtiyotkorroq yondashuv.

### ❓ Staging area (index) nima uchun kerak?

**✅ Javob:** Staging area keyingi commit'ga aynan nimani kiritishni tanlash imkonini beradi. Masalan, bitta faylda 5 ta o'zgarish qilgan bo'lsangiz, ulardan faqat ikkitasini bitta commit'ga, qolganini boshqa commit'ga ajratishingiz mumkin (`git add -p`). Bu toza, mantiqiy commit'lar yasashga yordam beradi.

### ❓ `merge` va `rebase` qachon ishlatiladi?

**✅ Javob:** `merge` — tarixni qanday bo'lsa shunday saqlaydi va umumiy branch'larni birlashtirishda xavfsiz. `rebase` — chiziqli, toza tarix yasaydi va lokal branch'ni `main` ustiga yangilashda yaxshi. Asosiy qoida: push qilingan, umumiy branch'ni hech qachon rebase qilmang.

### ❓ `reset --soft`, `--mixed`, `--hard` farqi?

**✅ Javob:** Uchalasi ham HEAD'ni ko'chiradi, farq — o'zgarishlar bilan nima bo'lishida: `--soft` ularni staging'da qoldiradi, `--mixed` (default) working directory'da qoldiradi (staging'dan chiqaradi), `--hard` esa butunlay o'chiradi. `--hard` eng xavfli.

### ❓ `git revert` va `git reset` farqi?

**✅ Javob:** `reset` tarixni orqaga suradi va commit'larni olib tashlaydi — lokal uchun. `revert` esa eski commit ta'sirini bekor qiluvchi YANGI commit yaratadi, tarixni saqlaydi — umumiy/push qilingan commit'larni bekor qilish uchun xavfsiz.

### ❓ Merge conflict qanday hal qilinadi?

**✅ Javob:** Git konfliktli faylga `<<<<<<<`, `=======`, `>>>>>>>` markerlarini qo'yadi. Faylni qo'lda tahrirlab, kerakli kodni qoldirib, markerlarni o'chirasiz. So'ng `git add <fayl>` bilan hal qilinganini belgilab, `git commit` (yoki `git rebase --continue`) bilan davom etasiz. Yoki `git merge --abort` bilan butunlay bekor qilasiz.

### ❓ `git stash` nima uchun kerak?

**✅ Javob:** Yarim tugallangan ishni commit qilmasdan vaqtincha saqlash uchun. Masalan, feature ustida ishlayotganda shoshilinch bug tuzatish kerak bo'lsa, `git stash` bilan ishni yashirib, branch almashtirib, bug'ni tuzatib, qaytib kelib `git stash pop` qilasiz.

### ❓ `origin` va `upstream` nima?

**✅ Javob:** Bular remote nomlari. `origin` — odatda sizning shaxsiy repozitoriyangiz (yoki klonlangan asosiy repo). `upstream` — fork qilgan bo'lsangiz, asl (original) repozitoriya uchun an'anaviy nom. `upstream`'dan fetch qilib, o'z fork'ingizni asl loyiha bilan yangilab turasiz.

### ❓ Detached HEAD nima va undan qanday chiqiladi?

**✅ Javob:** HEAD branch o'rniga to'g'ridan-to'g'ri commit'ga ishora qilgan holat (`git checkout <hash>` qilganda yuzaga keladi). Bu yerda commit qilsangiz, branch yaratmasangiz, ishingiz "osilib" qoladi. Chiqish uchun: `git switch -c yangi-branch` (ishni saqlash) yoki `git switch main` (tashlab ketish).

### ❓ `reflog` qanday yordam beradi?

**✅ Javob:** `reflog` HEAD'ning barcha harakatlarini yozib boradi. Agar xato `reset --hard` qilib commit'larni "yo'qotsangiz", `git reflog` orqali ularning hash'ini topib, `git reset --hard HEAD@{n}` bilan qaytarib olishingiz mumkin. Bu Git'ning "xavfsizlik to'ri".

### ❓ `cherry-pick` qachon ishlatiladi?

**✅ Javob:** Boshqa branch'dagi butun branch'ni emas, balki faqat bitta aniq commit'ni joriy branch'ga olib kelmoqchi bo'lganda. Masalan, bug tuzatuvchi commit `develop`'da bor, uni `main`'ga ham shoshilinch ko'chirish kerak — `git cherry-pick <hash>`.

### ❓ Annotated va lightweight tag farqi?

**✅ Javob:** Lightweight tag (`git tag v1.0`) — shunchaki commit'ga nom. Annotated tag (`git tag -a v1.0 -m "..."`) esa to'liq obyekt: muallif, sana, izoh va imzoni saqlaydi. Релизлар uchun annotated tag tavsiya etiladi.

### ❓ Squash nima va nima uchun kerak?

**✅ Javob:** Squash — bir nechta commit'ni bittaga birlashtirish. PR yuborishdan oldin "wip", "fix", "oops" kabi mayda commit'larni bitta toza, mazmunli commit'ga yig'ish uchun ishlatiladi. `git rebase -i` orqali yoki PR merge qilishda "Squash and merge" tugmasi bilan.

### ❓ `.gitignore`'dagi fayl baribir kuzatilyapti — nega?

**✅ Javob:** `.gitignore` faqat hali kuzatilmagan (untracked) fayllarga ta'sir qiladi. Agar fayl allaqachon commit qilingan bo'lsa, `.gitignore`'ga qo'shilsa ham kuzatilaveradi. Avval `git rm --cached <fayl>` bilan indeksdan chiqarib, keyin commit qilish kerak.

### ❓ `git switch` va `git checkout` farqi?

**✅ Javob:** Eski `git checkout` ikkita ishni qilardi: branch almashtirish va fayl tiklash — bu chalkash edi. Git 2.23'dan beri bular ikki aniq buyruqqa ajratildi: `git switch` (branch almashtirish/yaratish) va `git restore` (fayl tiklash). Funksionallik bir xil, lekin yangilari aniqroq.

## Masalalar

> Yechimlar: [yechimlar fayli](../../solutions/devops/01-git.md)

1. Yangi `my-project` katalogida Git repozitoriya yarating, bitta `README.md` fayl yarating va uni "Initial commit" xabari bilan commit qiling. Kerakli buyruqlarni ketma-ket yozing.

2. `feature/login` nomli yangi branch yarating, unga o'ting, bir o'zgarish qiling va commit qiling — bularning hammasini bajaruvchi buyruqlarni yozing.

3. Hozir `main` branch'dasiz. Oxirgi commit'ni butunlay o'chirmasdan, undagi o'zgarishlarni qayta tahrirlash uchun staging'ga qaytarmoqchisiz. Qaysi buyruqni ishlatasiz?

4. Siz `feature` branch'ida 4 ta mayda commit qildingiz. PR yuborishdan oldin ularni bitta toza commit'ga birlashtirmoqchisiz. Qaysi buyruq va qanday ishlatasiz?

5. `git pull` qilganingizda merge conflict yuzaga keldi. Konfliktni hal qilib, jarayonni yakunlash uchun bajariladigan qadamlarni ketma-ket yozing.

6. Siz xato qilib `git reset --hard HEAD~3` bajardingiz va 3 ta commit'ni yo'qotdingiz. Ularni qanday qaytarib olasiz? Buyruqlarni yozing.

7. `develop` branch'idagi `a1b2c3d` commit'ini `main` branch'ga olib kelmoqchisiz, lekin butun `develop`'ni emas. Buyruqni yozing.

8. Hozir yarim tugagan ish bor, lekin shoshilinch `main`'da bug tuzatish kerak. Ishingizni yo'qotmasdan branch almashtirish va keyin qaytib kelish uchun buyruqlar ketma-ketligini yozing.

9. Loyihangizda `node_modules/` va `.env` fayllari kuzatilmasligi kerak. Tegishli `.gitignore` faylini yozing. Agar `.env` allaqachon commit qilingan bo'lsa, uni kuzatuvdan chiqarish buyrug'ini ham qo'shing.

10. Lokal `feature` branch'ingizni remote `origin`'ga birinchi marta push qilib, upstream'ni o'rnatmoqchisiz. Buyruqni yozing.

11. `v2.0.0` annotated tag yarating va uni remote'ga yuboring. Buyruqlarni yozing.

12. `git log --oneline`'da ikkita commit ketma-ket: avval `feature` keyin `main`'dagi o'zgarishlar. Lokal `feature` branch'ingizni `main` ustiga rebase qilib, chiziqli tarix yasamoqchisiz. Buyruqlarni yozing va golden rule'ni eslatib o'ting.

← [DevOps bo'limiga qaytish](./README.md)
