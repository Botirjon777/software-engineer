# Git va Version Control — Yechimlar

Bu yerda [`devops/01-git.md`](../../devops/01-git.md) masalalarining yechimlari, buyruqlar va qisqa izohlar bilan keltirilgan.

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
- [11-masala](#11-masala)
- [12-masala](#12-masala)

## 1-masala

```bash
mkdir my-project
cd my-project
git init
echo "# My Project" > README.md
git add README.md
git commit -m "Initial commit"
```

**Izoh:** `git init` `.git` katalogini yaratadi. `git add` faylni staging'ga oladi, `git commit` esa repository'ga yozadi.

## 2-masala

```bash
git switch -c feature/login   # yaratib, darrov o'tish
# ... fayl tahrir qilinadi ...
git add .
git commit -m "feat: login funksiyasi qo'shildi"
```

**Izoh:** `git switch -c` bitta buyruqda branch yaratib, unga o'tadi (`git checkout -b`ning yangi muqobili).

## 3-masala

```bash
git reset --soft HEAD~1
```

**Izoh:** `--soft` commit'ni bekor qiladi, lekin o'zgarishlar staging area'da qoladi. Endi tahrirlab, qaytadan commit qilishingiz mumkin. `--mixed` bo'lsa o'zgarishlar working dir'ga tushardi, `--hard` esa butunlay o'chirib yuborardi.

## 4-masala

```bash
git rebase -i HEAD~4
```

Ochilgan oynada birinchi commit'ni `pick` qoldirib, qolgan uchtasini `squash` (yoki `s`) ga o'zgartiring:

```text
pick   aaa Birinchi commit
squash bbb Ikkinchi
squash ccc Uchinchi
squash ddd To'rtinchi
```

**Izoh:** Saqlab chiqqach, Git yangi, yagona commit xabarini yozishni so'raydi. Natijada 4 ta commit bitta toza commit'ga birlashadi.

## 5-masala

```bash
git pull origin main
# Conflict yuzaga keldi:
git status                 # konfliktli fayllarni ko'rish
# Faylni qo'lda tahrirlab, <<<<<<<, =======, >>>>>>> markerlarini o'chiramiz
git add <hal-qilingan-fayl>
git commit                 # merge commit'ni yakunlash
```

**Izoh:** Agar `pull --rebase` ishlatilgan bo'lsa, `git commit` o'rniga `git rebase --continue` ishlatiladi. Butunlay bekor qilish uchun `git merge --abort`.

## 6-masala

```bash
git reflog
# Chiqishda yo'qolgan commit hash'ini topamiz, masalan:
# a1b2c3d HEAD@{1}: commit: oxirgi ish
git reset --hard a1b2c3d
# yoki:
git reset --hard HEAD@{1}
```

**Izoh:** `reflog` HEAD'ning barcha harakatlarini saqlaydi. `reset --hard` qilingan bo'lsa ham, commit'lar darrov o'chmaydi va reflog orqali topib qaytariladi.

## 7-masala

```bash
git switch main
git cherry-pick a1b2c3d
```

**Izoh:** `cherry-pick` faqat bitta aniq commit'ni joriy branch'ga ko'chiradi, butun branch'ni emas.

## 8-masala

```bash
git stash               # yarim ishni vaqtincha yashirish
git switch main
# ... bug tuzatiladi, commit qilinadi ...
git switch feature      # o'z branch'ingizga qaytish
git stash pop           # yashirilgan ishni qaytarish
```

**Izoh:** `stash` commit qilmasdan ishni vaqtincha saqlaydi. `pop` uni qaytaradi va stash'dan o'chiradi (`apply` bo'lsa qoldiradi).

## 9-masala

`.gitignore` fayli:

```text
node_modules/
.env
```

Agar `.env` allaqachon commit qilingan bo'lsa:

```bash
git rm --cached .env
git commit -m "chore: .env'ni kuzatuvdan chiqarish"
```

**Izoh:** `.gitignore` faqat untracked fayllarga ta'sir qiladi, shuning uchun allaqachon kuzatilayotgan faylni `git rm --cached` bilan indeksdan chiqarish kerak (lokal fayl o'chmaydi).

## 10-masala

```bash
git push -u origin feature
```

**Izoh:** `-u` (yoki `--set-upstream`) lokal branch'ni remote bilan bog'laydi. Keyingi safar oddiy `git push` va `git pull` yetadi.

## 11-masala

```bash
git tag -a v2.0.0 -m "2.0.0 релизи"
git push origin v2.0.0
```

**Izoh:** `-a` annotated tag yaratadi (muallif, sana, izoh saqlanadi). Tag'lar avtomatik push qilinmaydi — alohida yuborish kerak. Hammasini birdan yuborish uchun `git push --tags`.

## 12-masala

```bash
git switch feature
git rebase main
```

**Golden Rule eslatmasi:** Bu yondashuv faqat `feature` branch lokal bo'lsa yoki push qilingan bo'lsa-da, undan boshqa hech kim foydalanmasa to'g'ri. Umumiy, baham ko'rilgan branch'ni rebase qilish boshqalarning tarixini buzadi — bunday holatda `merge` ishlating.

← [DevOps bo'limiga qaytish](../../devops/README.md)
