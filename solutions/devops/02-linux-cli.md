# Linux va Command Line — Yechimlar

Bu yerda [`devops/02-linux-cli.md`](../../devops/02-linux-cli.md) masalalarining yechimlari, buyruqlar va qisqa izohlar bilan keltirilgan.

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

```bash
grep -rni "error" /var/log/*.log
```

**Izoh:** `-r` rekursiv, `-n` qator raqami, `-i` katta-kichik harfga befarq. `*.log` faqat `.log` fayllar bilan cheklaydi.

## 2-masala

```bash
mkdir -p loyiha/src/utils
touch loyiha/src/utils/helper.js
```

**Izoh:** `-p` (parents) ichma-ich kataloglarni birdan yaratadi va mavjud bo'lsa xato bermaydi. `touch` bo'sh fayl hosil qiladi.

## 3-masala

```bash
chmod 755 script.sh
```

**Izoh:** `7=rwx` (owner), `5=r-x` (group), `5=r-x` (others). `r=4, w=2, x=1`, yig'indi: `4+2+1=7`, `4+0+1=5`.

## 4-masala

```bash
ps aux | grep nginx        # PID'ni topish
kill <PID>                 # tartibli to'xtatish (SIGTERM)
kill -9 <PID>              # javob bermasa, majburan (SIGKILL)
# yoki bir buyruqda nom bo'yicha:
pkill nginx
pkill -9 nginx
```

**Izoh:** Avval `SIGTERM` (oddiy `kill`) bilan tartibli yopishga imkon ber. Faqat osilib qolsa `-9` (`SIGKILL`) ishlat — u tozalanishga imkon bermaydi.

## 5-masala

```bash
tail -f app.log | grep -E "WARN|ERROR"
```

**Izoh:** `tail -f` faylga qo'shilayotgan yangi qatorlarni real vaqtda uzatadi, `grep -E` (kengaytirilgan regex) `WARN` yoki `ERROR` borlarini filtrlaydi.

## 6-masala

```bash
awk '{print $1}' access.log | sort | uniq -c | sort -rn | head -5
```

**Izoh:** `awk` IP'larni ajratadi → `sort` bir xillarni yonma-yon qiladi → `uniq -c` har birini sanaydi → `sort -rn` songa ko'ra teskari saralaydi → `head -5` eng yuqori 5 tasini oladi.

## 7-masala

```bash
nohup myapp > app.log 2>&1 &
```

**Izoh:** `nohup` — terminal yopilsa ham ishlashda davom etadi. `> app.log` stdout'ni faylga, `2>&1` stderr'ni ham o'sha joyga yo'naltiradi, `&` esa fonda ishga tushiradi.

## 8-masala

```bash
chown -R deploy:web /opt/app
```

**Izoh:** `-R` rekursiv (katalog ichidagi hamma fayl/katalog), `deploy:web` — egasi `deploy`, guruhi `web`.

## 9-masala

```bash
tar -czf backup-2026.tar.gz config/
tar -tzf backup-2026.tar.gz          # ichini ochmasdan ko'rish
```

**Izoh:** `c` create, `z` gzip, `f` fayl nomi. Ko'rish uchun `t` (list) bayrog'i — arxivni diskka ochmaydi.

## 10-masala

```bash
export PATH="$PATH:/opt/tools/bin"
```

**Izoh:** `$PATH` mavjud qiymatni saqlaydi, `:` bilan yangi katalog qo'shiladi. `export` o'zgarishni bola process'larga uzatadi. Doimiy bo'lishi uchun shu qatorni `~/.bashrc` ga yozish kerak.

← [DevOps bo'limiga qaytish](../../devops/README.md)
