# Linux va Command Line

Server dunyosi Linux ustida quriladi. Deyarli har bir DevOps vazifasi — deployment, monitoring, debugging — terminal orqali, command line'da bajariladi. Bu bo'limda shell nima ekanidan boshlab, fayl tizimi, navigatsiya, permissions, process'lar, I/O redirection, text processing va tarmoq buyruqlarigacha amaliy va intervyuga tayyorlovchi tarzda ko'rib chiqamiz.

**💡 Tushuncha:** Command line'ni shunchaki "buyruqlarni yodlash" deb qabul qilma. Asosiy model — kichik buyruqlarni `pipe` orqali bir-biriga ulab, kuchli zanjir yasash (Unix falsafasi: "do one thing well"). Buni tushunsang, har qanday vazifani yecha olasan.

## Mundarija

- [Shell nima](#shell-nima)
- [Fayl tizimi ierarxiyasi](#fayl-tizimi-ierarxiyasi)
- [Navigatsiya](#navigatsiya)
- [Fayl operatsiyalari](#fayl-operatsiyalari)
- [Qidirish: find, grep, locate](#qidirish-find-grep-locate)
- [Permissions](#permissions)
- [Process boshqaruvi](#process-boshqaruvi)
- [I/O redirection va pipe](#io-redirection-va-pipe)
- [Environment variables](#environment-variables)
- [Text processing](#text-processing)
- [Package manager](#package-manager)
- [Tarmoq buyruqlari](#tarmoq-buyruqlari)
- [Arxivlash](#arxivlash)
- [SSH asoslari](#ssh-asoslari)
- [Savol-javoblar](#savol-javoblar)
- [Masalalar](#masalalar)

## Shell nima

Shell — bu siz yozgan matnli buyruqlarni qabul qilib, operatsion tizim yadrosiga (kernel) yetkazuvchi dastur. Ya'ni inson bilan kernel o'rtasidagi tarjimon.

Eng keng tarqalgan shell — **bash** (Bourne Again SHell). Boshqalari ham bor: `sh`, `zsh`, `fish`. DevOzda asosan `bash` ishlatiladi, chunki u deyarli hamma Linux serverda mavjud.

```bash
echo $SHELL        # joriy shell qaysi
cat /etc/shells    # tizimda mavjud shell'lar ro'yxati
which bash         # bash bajariluvchi fayli qayerda
```

**💡 Tushuncha:** "Terminal", "console", "shell" ko'pincha bir ma'noda ishlatiladi, lekin aniq farqi bor. **Terminal** — oynacha (interfeys), **shell** — uning ichidagi buyruqlarni izohlovchi dastur. Terminal — chevaqa, shell — uning ichidagi tarjimon.

## Fayl tizimi ierarxiyasi

Linux'da hamma narsa bitta daraxt ko'rinishidagi tuzilmaga joylanadi. Eng yuqori nuqta — root (`/`). Windows'dagi `C:`, `D:` kabi alohida disklar yo'q — hammasi `/` ostida.

| Katalog | Vazifasi |
|---------|----------|
| `/` | root — butun fayl tizimining ildizi |
| `/home` | foydalanuvchilarning shaxsiy kataloglari (`/home/ali`) |
| `/etc` | tizim va dasturlar konfiguratsiya fayllari |
| `/var` | o'zgaruvchi ma'lumotlar: log'lar (`/var/log`), cache |
| `/bin`, `/usr/bin` | bajariluvchi dasturlar (buyruqlar: `ls`, `cp`) |
| `/tmp` | vaqtinchalik fayllar (reboot'da tozalanadi) |
| `/root` | root foydalanuvchining home katalogi |
| `/opt` | qo'shimcha (uchinchi tomon) dasturlar |
| `/dev` | qurilmalar (disk, terminal) fayl ko'rinishida |

```bash
ls /            # root katalogdagi narsalarni ko'rish
cat /etc/os-release   # tizim qaysi Linux distributivi
```

**⚠️ Ehtiyot bo'l:** `/etc` va `/var` ostidagi fayllarni o'zgartirishda ehtiyot bo'l — bu tizim ishlashiga ta'sir qiladi. Har doim avval nusxa olib qo'y: `cp /etc/nginx/nginx.conf /etc/nginx/nginx.conf.bak`.

## Navigatsiya

Fayl tizimi ichida harakatlanish uchun uchta asosiy buyruq:

```bash
pwd                # qayerdaman? (print working directory)
ls                 # joriy katalogdagi fayllar
ls -l              # batafsil (permissions, egasi, hajmi, sana)
ls -la             # yashirin (.) fayllar bilan
ls -lh             # hajmni o'qiladigan ko'rinishda (KB, MB)
cd /var/log        # absolyut yo'l bilan o'tish
cd ..              # bir pog'ona yuqoriga
cd ~               # home katalogga (yoki shunchaki cd)
cd -               # oldingi katalogga qaytish
```

**💡 Tushuncha:** Yo'l (path) ikki xil: **absolyut** — `/` dan boshlanadi (`/home/ali/docs`), **relativ** — joriy katalogdan (`docs/file.txt`, `../config`). `.` — joriy katalog, `..` — yuqoridagi katalog, `~` — home.

## Fayl operatsiyalari

```bash
mkdir loyiha            # katalog yaratish
mkdir -p a/b/c          # ichma-ich kataloglarni birdan yaratish
touch file.txt          # bo'sh fayl yaratish (yoki sanasini yangilash)
cp manba.txt nusxa.txt  # nusxa olish
cp -r src/ dest/        # katalogni rekursiv nusxalash
mv eski.txt yangi.txt   # ko'chirish yoki nomini o'zgartirish
rm file.txt             # fayl o'chirish
rm -r katalog/          # katalogni rekursiv o'chirish
rm -rf katalog/         # majburiy, so'ramasdan o'chirish
```

Fayl mazmunini ko'rish:

```bash
cat file.txt            # butun faylni chiqarish
less file.txt           # sahifalab ko'rish (q bilan chiqish)
head -n 20 file.txt     # birinchi 20 qator
tail -n 20 file.txt     # oxirgi 20 qator
tail -f /var/log/app.log  # real vaqtda yangi qatorlarni kuzatish
```

**⚠️ Ehtiyot bo'l:** `rm -rf` — eng xavfli buyruq. `rm -rf /` butun tizimni o'chiradi. Yo'lni har doim tekshir, ayniqsa o'zgaruvchi bilan ishlaganda: `rm -rf "$DIR/"` — agar `$DIR` bo'sh bo'lsa, falokat. Linux'da "o'chirilganni qaytarish" (recycle bin) yo'q.

**💡 Tushuncha:** `tail -f` — DevOps'ning eng sevimli buyrug'i. Log faylni real vaqtda kuzatib, dastur nima qilayotganini ko'rib turish uchun.

## Qidirish: find, grep, locate

`find` — fayllarni nom, hajm, sana, tur bo'yicha qidiradi:

```bash
find . -name "*.log"              # joriy katalogda barcha .log fayllar
find /var -type f -size +100M     # 100MB dan katta fayllar
find . -mtime -7                  # oxirgi 7 kunda o'zgargan fayllar
find . -name "*.tmp" -delete      # topib o'chirish
find /home -type d -name "node_modules"  # faqat kataloglar
```

`grep` — fayllar ichidagi matnni qidiradi:

```bash
grep "error" app.log              # "error" so'zi bor qatorlar
grep -i "error" app.log           # katta-kichik harfga e'tibor bermay
grep -r "TODO" ./src              # rekursiv, butun katalog ichida
grep -n "error" app.log           # qator raqami bilan
grep -v "debug" app.log           # "debug" YO'Q qatorlar (inkor)
grep -c "error" app.log           # nechta qator topilganini sanash
```

`locate` — oldindan tuzilgan indeks orqali juda tez qidiradi (lekin indeks eskirgan bo'lishi mumkin):

```bash
locate nginx.conf      # tez, lekin updatedb yangi bo'lishi kerak
sudo updatedb          # indeksni yangilash
```

**💡 Tushuncha:** `find` — sekin, lekin real vaqtda diskni o'qiydi va aniq natija beradi. `locate` — tez, lekin indeksga tayanadi. Intervyuda farqini so'rashlari mumkin.

## Permissions

Linux'da har bir faylda uchta toifa uchun ruxsatlar bor: **owner** (egasi), **group** (guruh), **others** (qolganlar). Har biri uchun uchta huquq: `r` (read/o'qish), `w` (write/yozish), `x` (execute/bajarish).

```text
-rwxr-xr--   1 ali devs  4096 Jun 30 10:00 script.sh
 │└┬┘└┬┘└┬┘
 │ │  │  └── others: r-- (faqat o'qish)
 │ │  └───── group:  r-x (o'qish + bajarish)
 │ └──────── owner:  rwx (hammasi)
 └────────── fayl turi (- fayl, d katalog, l link)
```

`chmod` — ruxsatlarni o'zgartirish (octal yoki simvolik):

```bash
chmod 755 script.sh    # rwxr-xr-x (octal)
chmod 644 file.txt     # rw-r--r--
chmod +x script.sh     # hammaga execute qo'shish (simvolik)
chmod u+w,o-r file     # owner'ga write, others'dan read olib tashlash
```

Octal hisobi: `r=4`, `w=2`, `x=1`. Yig'indi:

| Octal | Ruxsat | Ma'no |
|-------|--------|-------|
| 7 | rwx | hammasi |
| 6 | rw- | o'qish + yozish |
| 5 | r-x | o'qish + bajarish |
| 4 | r-- | faqat o'qish |
| 0 | --- | hech narsa |

`chown` — egasini va guruhini o'zgartirish:

```bash
chown ali file.txt          # egasini ali ga
chown ali:devs file.txt     # egasini ali, guruhini devs ga
chown -R ali:devs /app      # rekursiv, butun katalog
```

**⚠️ Ehtiyot bo'l:** `chmod 777` — "hamma narsaga ruxsat" — xavfli odat. Har kim faylni o'zgartira/bajara oladi. Production'da hech qachon `777` ishlatma; kerakli minimal ruxsatni ber (masalan, `644` fayl, `755` katalog/skript).

## Process boshqaruvi

Har bir ishlayotgan dastur — **process**, har birida unikal **PID** (Process ID) bor.

```bash
ps aux                 # barcha process'lar (egasi, PID, CPU, RAM)
ps aux | grep nginx    # faqat nginx process'lari
top                    # real vaqtda interaktiv monitor
htop                   # top'ning chiroyliroq versiyasi (alohida o'rnatiladi)
kill 1234              # PID 1234 ga to'xtash signali (SIGTERM)
kill -9 1234           # majburan o'ldirish (SIGKILL)
killall nginx          # nom bo'yicha barcha process'larni o'ldirish
```

Fonda (background) ishlatish:

```bash
./long-script.sh &     # fonda ishga tushirish
jobs                   # joriy shell'dagi fon ishlari
fg %1                  # 1-ishni oldingi planga olib kelish
bg %1                  # to'xtatilgan ishni fonda davom ettirish
nohup ./script.sh &    # terminal yopilsa ham ishlashda davom etadi
```

**💡 Tushuncha:** `kill` aslida "o'ldirmaydi" — u **signal** yuboradi. `SIGTERM` (15, oddiy `kill`) — "iltimos, tartibli yopil" deydi; dastur tozalanib chiqishga ulguradi. `SIGKILL` (9, `kill -9`) — "darrov o'l" deydi; dasturga tanlov qoldirmaydi. Avval `SIGTERM` sina, ishlamasa `-9` ishlat.

**⚠️ Ehtiyot bo'l:** `&` bilan ishga tushirilgan process terminal yopilganda o'ladi. Server'da uzoq ishlashi kerak bo'lsa, `nohup`, `tmux`, yoki yaxshisi `systemd` service ishlat.

## I/O redirection va pipe

Har bir process'da uchta standart oqim bor: **stdin** (0, kirish), **stdout** (1, chiqish), **stderr** (2, xato).

```bash
echo "salom" > file.txt        # stdout'ni faylga yozish (ustiga yozadi)
echo "yana" >> file.txt        # faylga qo'shish (oxiriga)
sort < input.txt               # fayldan stdin sifatida o'qish
command 2> error.log           # faqat xatolarni faylga
command > out.log 2>&1         # stdout VA stderr'ni bitta faylga
command > /dev/null 2>&1       # hamma chiqishni o'chirib tashlash
```

`|` (pipe) — bir buyruq chiqishini boshqasiga kirish sifatida uzatadi:

```bash
ps aux | grep nginx                    # process ro'yxatini filtrlash
cat app.log | grep error | wc -l       # xatolar sonini sanash
ls -la | sort -k5 -n                   # hajmi bo'yicha saralash
history | grep git | tail -20          # oxirgi git buyruqlari
```

**💡 Tushuncha:** `2>&1` ni "stderr'ni stdout boradigan joyga yubor" deb o'qi. Tartib muhim: `> out.log 2>&1` to'g'ri ishlaydi, lekin `2>&1 > out.log` ishlamaydi — chunki `2>&1` yozilganda stdout hali terminalga ketardi. `/dev/null` — "qora tuynuk", unga yozilgan hamma narsa yo'qoladi.

## Environment variables

Environment variable — shell va dasturlar foydalanadigan nom-qiymat juftliklari.

```bash
echo $HOME             # home katalog
echo $PATH             # buyruqlar qidiriladigan kataloglar
echo $USER             # joriy foydalanuvchi
MY_VAR="qiymat"        # faqat joriy shell uchun
export MY_VAR="qiymat" # bola process'larga ham uzatiladi
env                    # barcha environment o'zgaruvchilar
unset MY_VAR           # o'chirish
```

`PATH` — eng muhim o'zgaruvchi. Siz buyruq yozganingizda, shell uni `PATH` ichidagi kataloglardan qidiradi (`:` bilan ajratilgan):

```bash
echo $PATH
# /usr/local/bin:/usr/bin:/bin:/usr/sbin
export PATH="$PATH:/opt/myapp/bin"   # PATH'ga yangi katalog qo'shish
```

**💡 Tushuncha:** Doimiy (har safar terminal ochilganda) o'zgaruvchi kerak bo'lsa, uni `~/.bashrc` yoki `~/.profile` fayliga yoz. `export` qilingan o'zgaruvchi bola process'larga ham uzatiladi; `export`siz oddiy o'zgaruvchi faqat joriy shell'da qoladi.

## Text processing

DevOps'da log va konfiguratsiya fayllarini qayta ishlash uchun kuchli buyruqlar:

```bash
grep "ERROR" app.log                   # filtrlash (yuqorida ko'rdik)
cut -d':' -f1 /etc/passwd              # ':' ajratuvchi, 1-ustun (foydalanuvchilar)
sort names.txt                         # alfavit bo'yicha saralash
sort -n -r nums.txt                    # raqamli, teskari saralash
uniq                                   # ketma-ket takrorlarni o'chirish
sort file | uniq -c                    # har birini sanab chiqish
wc -l file.txt                         # qatorlar sonini sanash
```

`sed` — matnni qator-qator o'zgartirish (stream editor):

```bash
sed 's/eski/yangi/g' file.txt          # almashtirish (g = barcha)
sed -i 's/foo/bar/g' file.txt          # faylning o'zida (-i = in-place)
sed -n '10,20p' file.txt               # 10-20 qatorlarni chiqarish
```

`awk` — ustunlar bilan ishlovchi kuchli til:

```bash
awk '{print $1}' access.log            # birinchi ustun (IP)
awk '{print $1, $4}' file              # 1 va 4-ustunlar
awk -F':' '{print $1}' /etc/passwd     # ajratuvchini ':' qilib
awk '$3 > 100 {print $1}' data.txt     # shart bilan filtrlash
```

Amaliy misol — log'da eng ko'p uchragan IP'larni topish:

```bash
awk '{print $1}' access.log | sort | uniq -c | sort -rn | head -10
```

**💡 Tushuncha:** `cut` — oddiy, qat'iy ustunlar uchun. `awk` — murakkab, shartli mantiq kerak bo'lganda. `sed` — almashtirish va qator tahriri uchun. Bularni `pipe` bilan birlashtirib, deyarli har qanday matn vazifasini hal qilasan.

## Package manager

Dasturlarni o'rnatish/yangilash uchun. Distributivga qarab farq qiladi:

**Debian/Ubuntu — `apt`:**

```bash
sudo apt update                # paket ro'yxatini yangilash
sudo apt upgrade               # o'rnatilgan paketlarni yangilash
sudo apt install nginx         # o'rnatish
sudo apt remove nginx          # o'chirish
apt search docker              # qidirish
```

**RHEL/CentOS/Fedora — `yum` yoki `dnf`:**

```bash
sudo yum update
sudo yum install nginx
sudo yum remove nginx
```

**⚠️ Ehtiyot bo'l:** `apt update` va `apt upgrade` har xil. `update` — faqat mavjud paketlar ro'yxatini yangilaydi (hech narsa o'rnatmaydi); `upgrade` — haqiqatan paketlarni yangilaydi. Avval `update`, keyin `upgrade`.

## Tarmoq buyruqlari

```bash
curl https://api.example.com           # HTTP so'rov yuborish
curl -I https://example.com            # faqat header'lar
curl -X POST -d '{"a":1}' url          # POST so'rov, ma'lumot bilan
curl -o file.zip https://.../file.zip  # faylga saqlash
wget https://example.com/file.zip      # fayl yuklab olish
ping google.com                        # ulanish bor-yo'qligini tekshirish
ss -tulpn                              # ochiq portlar va ularni egallagan process'lar
ss -t                                  # faol TCP ulanishlar
```

**💡 Tushuncha:** `curl` vs `wget`: `curl` — moslashuvchan, API bilan ishlash (POST, header, JSON) uchun; standart holatda natijani ekranga chiqaradi. `wget` — fayl yuklab olishga (recursive download) qulay; standart holatda diskka saqlaydi. `ss` — eski `netstat`ning zamonaviy o'rnini bosadi.

## Arxivlash

`tar` — fayllarni bitta arxivga yig'ish (va gzip bilan siqish):

```bash
tar -czf arxiv.tar.gz katalog/    # yaratish (c) + gzip (z) + fayl (f)
tar -xzf arxiv.tar.gz             # ochish (x)
tar -tzf arxiv.tar.gz             # ichini ko'rish (t), ochmasdan
tar -xzf arxiv.tar.gz -C /opt     # ma'lum katalogga ochish
gzip file.txt                     # faylni siqish (file.txt.gz)
gunzip file.txt.gz                # ochish
```

**💡 Tushuncha:** Bayroqlarni eslab qolish: **c**reate, e**x**tract, **t**list (ko'rish), **z** = gzip siqish, **f** = fayl nomi, **v** = verbose (jarayonni ko'rsatish). `czf` — yaratish, `xzf` — ochish.

## SSH asoslari

SSH (Secure Shell) — masofadagi serverga shifrlangan ulanish.

```bash
ssh ali@192.168.1.10                   # serverga ulanish
ssh -p 2222 ali@server.com             # boshqa port bilan
ssh-keygen -t ed25519                  # kalit juftligini yaratish
ssh-copy-id ali@server.com             # public kalitni serverga yuborish
scp file.txt ali@server:/tmp/          # fayl ko'chirish (server'ga)
scp ali@server:/tmp/file.txt .         # server'dan olib kelish
```

**💡 Tushuncha:** SSH key juftligi: **private key** (`~/.ssh/id_ed25519`) — sizda qoladi, hech kimga bermaysiz; **public key** (`.pub`) — serverga joylanadi. Server public bilan tekshiradi, siz private bilan isbotlaysiz. Parol o'rniga kalit — xavfsizroq va qulayroq.

**⚠️ Ehtiyot bo'l:** Private key'ni hech qachon nusxalab, git'ga commit qilib yoki birovga yuborma. Uning fayl ruxsati `600` (faqat egasi o'qiy oladi) bo'lishi shart, aks holda SSH uni rad etadi.

## Savol-javoblar

### ❓ Shell va terminal o'rtasidagi farq nima?

**✅ Javob:** Terminal — bu siz matn yozadigan oyna (interfeys/emulator). Shell — uning ichida ishlaydigan, buyruqlaringizni izohlab kernelga uzatadigan dastur (masalan, `bash`). Terminal — idish, shell — ichidagi mexanizm.

### ❓ Absolyut va relativ yo'l farqi?

**✅ Javob:** Absolyut yo'l `/` (root)dan boshlanadi va har doim bir xil joyni ko'rsatadi: `/home/ali/file.txt`. Relativ yo'l joriy katalogga nisbatan: `file.txt`, `../config`, `./script.sh`. Skriptlarda absolyut yo'l ishonchliroq, chunki joriy katalogga bog'liq emas.

### ❓ `chmod 755` nimani anglatadi?

**✅ Javob:** `7=rwx` (owner: o'qish, yozish, bajarish), `5=r-x` (group: o'qish, bajarish), `5=r-x` (others: o'qish, bajarish). Ya'ni egasi to'liq nazoratga ega, qolganlar faqat o'qiy va bajara oladi, lekin o'zgartira olmaydi. Skript va kataloglar uchun odatiy.

### ❓ `kill` va `kill -9` farqi nima?

**✅ Javob:** Oddiy `kill` — `SIGTERM` (15) signalini yuboradi: "tartibli yopil" deydi, dastur tozalanib (fayllar yopish, vaqtinchalik ma'lumot saqlash) chiqadi. `kill -9` — `SIGKILL` (9): kernel process'ni darrov o'ldiradi, tozalanishga imkon bermaydi. Avval `kill`, faqat osilib qolsa `-9`.

### ❓ `>` va `>>` o'rtasidagi farq?

**✅ Javob:** `>` — faylni ustiga yozadi (mavjud mazmun o'chadi). `>>` — fayl oxiriga qo'shadi (eski mazmun saqlanadi). Log'ga yozayotganda har doim `>>` ishlat, aks holda har safar faylni o'chirib yuborasan.

### ❓ `2>&1` nimani bildiradi?

**✅ Javob:** "stderr (2) ni stdout (1) hozir ketayotgan joyga yo'naltir" degani. `command > log.txt 2>&1` — oddiy chiqish ham, xatolar ham bitta `log.txt` ga yoziladi. Tartib muhim: redirect avval `stdout`ni faylga burib, keyin `stderr`ni unga ulaydi.

### ❓ `/dev/null` nima?

**✅ Javob:** Bu "qora tuynuk" — unga yozilgan hamma narsa yo'qoladi. Keraksiz chiqishni o'chirish uchun ishlatiladi: `command > /dev/null 2>&1` — buyruq ishlaydi, lekin hech narsa ko'rsatmaydi (na chiqish, na xato).

### ❓ `find` va `locate` qaysi biri tezroq, qaysi biri aniqroq?

**✅ Javob:** `locate` tezroq — chunki oldindan tuzilgan indeksdan qidiradi, diskni real vaqtda kovlamaydi. Lekin indeks eskirgan bo'lishi mumkin (yangi fayllarni topmasligi mumkin). `find` sekinroq, lekin diskni real o'qiganidan har doim aniq va joriy natija beradi.

### ❓ `grep -v` nima qiladi?

**✅ Javob:** `-v` — inkor (invert). Naqshga **MOS KELMAYDIGAN** qatorlarni chiqaradi. Masalan, `grep -v "debug" app.log` — `debug` so'zi bo'lmagan barcha qatorlarni qaytaradi. Shovqinli log'larni tozalashda foydali.

### ❓ `export` qilingan va oddiy o'zgaruvchi farqi?

**✅ Javob:** Oddiy `VAR=qiymat` — faqat joriy shell'da yashaydi, undan ishga tushgan dasturlar (bola process'lar) uni ko'rmaydi. `export VAR=qiymat` — o'zgaruvchini bola process'larga ham uzatadi. Dastur konfiguratsiyasini env orqali bersangiz, `export` kerak.

### ❓ `PATH` o'zgaruvchisi nima vazifa bajaradi?

**✅ Javob:** `PATH` — buyruq yozganingizda shell qidiradigan kataloglar ro'yxati (`:` bilan ajratilgan). `ls` yozsangiz, shell `PATH` ichidagi har bir katalogdan `ls` faylini qidiradi. Yangi dastur "command not found" deyilsa, ko'pincha uning katalogi `PATH`da yo'q.

### ❓ `apt update` va `apt upgrade` farqi?

**✅ Javob:** `apt update` — faqat mavjud paketlar ro'yxati va versiyalarini yangilaydi (metama'lumot), hech narsa o'rnatmaydi. `apt upgrade` — haqiqatan o'rnatilgan paketlarni yangi versiyaga ko'taradi. To'g'ri tartib: avval `update`, keyin `upgrade`.

### ❓ `curl` va `wget` qachon qaysisini ishlatasiz?

**✅ Javob:** `curl` — API bilan ishlash uchun (POST, header, JSON yuborish), natijani standart holatda ekranga chiqaradi. `wget` — fayllarni (ayniqsa rekursiv) yuklab olish uchun qulay, standart holatda diskka saqlaydi. Skriptlarda API chaqiruv — `curl`, fayl yuklash — `wget`.

### ❓ SSH'da public va private key qanday ishlaydi?

**✅ Javob:** Asimmetrik kriptografiya: **private key** sizda maxfiy qoladi, **public key** serverга joylanadi. Ulanganda server tasodifiy ma'lumotni public bilan shifrlaydi, faqat mos private uni ocha oladi — shu orqali kimligingiz isbotlanadi. Parolsiz, xavfsizroq autentifikatsiya.

### ❓ `tar -czf` va `tar -xzf` bayroqlari nimani anglatadi?

**✅ Javob:** `c` = create (yaratish), `x` = extract (ochish), `z` = gzip bilan siqish, `f` = keyingi argument fayl nomi. Demak `czf` — gzip arxiv yaratish, `xzf` — gzip arxivni ochish. Eslab qolish uchun: "create zip file" / "extract zip file".

## Masalalar

> Yechimlar: [yechimlar fayli](../../solutions/devops/02-linux-cli.md)

1. `/var/log` katalogidagi barcha `.log` fayllar ichida "ERROR" so'zini, qator raqami bilan, katta-kichik harfga e'tibor bermay qidiradigan buyruqni yozing.

2. Joriy katalogda `loyiha/src/utils` kabi ichma-ich kataloglar tuzilmasini bitta buyruq bilan yarating, so'ng `utils` ichida bo'sh `helper.js` fayli hosil qiling.

3. `script.sh` faylini egasi to'liq (o'qish/yozish/bajarish), guruh va boshqalar faqat o'qish va bajarish huquqiga ega bo'ladigan qilib sozlang. Octal qiymatni ishlating.

4. `nginx` nomli process'ni avval tartibli to'xtatishga urinib ko'ring, agar javob bermasa majburan o'ldiring. PID'ni topish va o'ldirish buyruqlarini ketma-ket yozing.

5. `app.log` faylini real vaqtda kuzatib turadigan, lekin faqat "WARN" yoki "ERROR" so'zlari bor qatorlarni ko'rsatadigan buyruqni yozing.

6. `access.log` faylida birinchi ustun — IP manzil. Eng ko'p so'rov yuborgan 5 ta IP'ni, har biri yonida so'rovlar soni bilan chiqaradigan buyruqlar zanjirini yozing.

7. `myapp` dasturini fonda, terminal yopilsa ham ishlashda davom etadigan qilib ishga tushiring, chiqishini `app.log` ga, xatolarini ham o'sha faylga yo'naltiring.

8. `/opt/app` katalogini va uning butun ichidagi fayllarni `deploy` foydalanuvchisi va `web` guruhiga rekursiv tarzda o'tkazing.

9. `config/` katalogini `gzip` bilan siqib, `backup-2026.tar.gz` nomli arxiv yarating. So'ng arxiv ichidagi fayllarni ochmasdan ko'rib chiqing.

10. `PATH` o'zgaruvchisiga `/opt/tools/bin` katalogini, mavjud qiymatni yo'qotmasdan qo'shing va bu o'zgarish ishga tushgan dasturlarga ham uzatilishini ta'minlang.

← [DevOps bo'limiga qaytish](./README.md)
