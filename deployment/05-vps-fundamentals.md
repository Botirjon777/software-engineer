# VPS Asoslari

Managed platformalar (Vercel, Railway) oson, lekin ular sizga to'liq nazorat bermaydi va ba'zan qimmatga tushadi. **VPS** (Virtual Private Server) — bu sizning shaxsiy Linux serveringiz: unda istagan narsani o'rnatasiz, istaganingizcha sozlaysiz. Bu qo'llanmada VPS nima ekanini, uni qanday sotib olishni, SSH orqali ulanishni va serverni birinchi marta xavfsiz sozlashni (2026-yil, Ubuntu 24.04 asosida) qadama-qadam o'rganamiz.

## Mundarija

- [VPS nima](#vps-nima)
- [Shared hosting vs VPS vs Cloud](#shared-hosting-vs-vps-vs-cloud)
- [VPS qachon kerak](#vps-qachon-kerak)
- [Provayderlar (narx va tavsiya)](#provayderlar)
- [VPS yaratish](#vps-yaratish)
- [SSH nima va serverga ulanish](#ssh-nima-va-serverga-ulanish)
- [SSH key bilan parolsiz kirish](#ssh-key-bilan-parolsiz-kirish)
- [Boshlang'ich server sozlash](#boshlangich-server-sozlash)
- [Non-root sudo user yaratish](#non-root-sudo-user-yaratish)
- [SSH hardening (xavfsizlash)](#ssh-hardening)
- [Firewall (ufw) sozlash](#firewall-ufw-sozlash)
- [Asosiy Linux boshqaruv](#asosiy-linux-boshqaruv)
- [Swap qo'shish](#swap-qoshish)
- [Amaliy Q&A (troubleshooting)](#amaliy-qa)
- [Amaliy Checklist](#amaliy-checklist)

---

## VPS nima

**VPS** — bu bitta katta fizik serverni virtualizatsiya orqali bir nechta mustaqil "virtual" serverlarga bo'lib berilgan bo'lagi. Har bir VPS o'zining CPU, RAM, disk va IP-manz

iliga ega bo'ladi va o'zini alohida kompyuterdek tutadi. Siz unga `root` (to'liq huquqli) sifatida kirasiz va istagan dasturingizni (Node.js, Nginx, PostgreSQL, Docker) o'rnatasiz.

**💡 Tushuncha:** VPS — bu "internetda doim yoqilgan turgan Linux kompyuteringiz". Uni uydagi kompyuter kabi tasavvur qiling, lekin u data-markazda joylashgan, doim online, statik IP-manzilga ega va sizga faqat internet orqali (SSH bilan) ko'rinadi. Ekran ham, klaviatura ham yo'q — hamma narsa terminal orqali.

---

## Shared hosting vs VPS vs Cloud

| Xususiyat | Shared Hosting | VPS | Cloud (AWS/GCP) |
|-----------|----------------|-----|-----------------|
| **Nazorat** | Cheklangan (cPanel) | To'liq root | To'liq + xizmatlar |
| **root access** | Yo'q | Bor | Bor |
| **Narx** | Arzon ($2–5/oy) | O'rtacha ($4–20/oy) | O'zgaruvchan (murakkab) |
| **Resurs** | Boshqalar bilan bo'lishilgan | Ajratilgan (kafolatlangan) | Cheksiz kengayadi |
| **Mos** | Oddiy PHP/WordPress | Node/Python/full-stack | Katta, scale kerak loyihalar |
| **Murakkablik** | Oson | O'rtacha (Linux bilish kerak) | Yuqori |

**💡 Tushuncha:**
- **Shared hosting** — bitta serverda yuzlab foydalanuvchi bir vaqtda. Siz faqat fayllaringizni yuklaysiz, sozlash imkoni deyarli yo'q. WordPress/oddiy PHP uchun.
- **VPS** — sizga ajratilgan resurs va to'liq root. Node.js, custom Nginx, o'z database'ingiz. Bu qo'llanmaning asosiy mavzusi.
- **Cloud** — Amazon (AWS), Google (GCP). Cheksiz kengayadigan, lekin murakkab va boshqaruvi qiyin. Startup katta bo'lganda kerak bo'ladi; boshlovchiga hozircha shart emas.

---

## VPS qachon kerak

**VPS quyidagi hollarda kerak:**

1. **To'liq nazorat kerak** — o'z Nginx configingiz, custom port, cron job, system-level tool o'rnatish.
2. **Managed platforma cheklovlari** — Railway/Render bepul tarifi tugadi yoki qimmatlashdi.
3. **Bir nechta loyihani bitta serverda** — bitta VPS'da 5-10 ta kichik sayt joylashishi mumkin.
4. **Database + backend + fayllar birga** — hammasi bir joyda, network latency past.
5. **WebSocket, background worker, long-running process** — ba'zi managed platformalar buni cheklaydi.
6. **O'rganish** — DevOps, Linux, server boshqaruvini o'rganmoqchisiz.

**⚠️ Ehtiyot bo'l:** VPS'da hamma javobgarlik sizda — xavfsizlik (security), yangilanish (update), backup, monitoring. Managed platforma bularni siz uchun qiladi. Agar loyihangiz oddiy static sayt yoki kichik API bo'lsa, avval Vercel/Railway'ni sinab ko'ring — VPS ortiqcha yuk bo'lishi mumkin.

---

## Provayderlar

2026-yil holatida eng mashhur VPS provayderlar:

| Provayder | Boshlang'ich narx | Xususiyat | Tavsiya |
|-----------|-------------------|-----------|---------|
| **Hetzner** | ~€4/oy (2 vCPU, 4GB) | Eng arzon, kuchli, Germaniya/Finlyandiya | ⭐ Narx/sifat eng yaxshi |
| **DigitalOcean** | $6/oy (1 vCPU, 1GB) | Sodda UI, ajoyib docs, community | ⭐ Boshlovchi uchun eng oson |
| **Vultr** | $6/oy (1 vCPU, 1GB) | Ko'p region, tez | Yaxshi alternativa |
| **Linode (Akamai)** | $5/oy (1 vCPU, 1GB) | Barqaror, ishonchli | Yaxshi |
| **Contabo** | ~€5/oy (4 vCPU, 8GB) | Juda ko'p resurs arzonga | Resurs ko'p kerak bo'lsa |

**💡 Tushuncha:**
- **Hetzner** — pul tejamoqchi bo'lsangiz eng yaxshi tanlov. Xuddi shu pulga boshqalardan 2-4 baravar ko'p resurs beradi. Kamchiligi: data-markaz asosan Yevropada (O'zbekistonga latency ~80-120ms, bu ko'p loyiha uchun me'yorda).
- **DigitalOcean** — birinchi VPS'ingiz uchun ideal: interfeys sodda, hujjatlari (tutorials) juda sifatli, xatoga yo'l qo'ysangiz Google'dan tez yechim topasiz.
- **Contabo** — arzonga juda ko'p RAM/CPU, lekin support va tarmoq tezligi ba'zan sekinroq.

**Tavsiya (boshlovchi uchun):** Birinchi loyiha — **DigitalOcean** (o'rganish oson). Pul muhim bo'lsa yoki tajriba ortsa — **Hetzner**.

---

## VPS yaratish

Misol tariqasida DigitalOcean'da (boshqa provayderlarda ham deyarli bir xil):

1. **Ro'yxatdan o'ting** va to'lov usulini qo'shing (karta yoki PayPal).
2. **Create → Droplet** (DigitalOcean'da VPS "Droplet" deyiladi; Hetzner'da "Server", boshqalarda "Instance").
3. **Region tanlang** — foydalanuvchilaringizga eng yaqin joy. O'zbekiston uchun: **Frankfurt (Germaniya)** yoki **Amsterdam** yaxshi (latency past). Uzoq region (masalan, San Francisco) sekin bo'ladi.
4. **OS (image) tanlang** — **Ubuntu 24.04 LTS**. LTS = Long Term Support (uzoq muddat yangilanadi, barqaror). Boshlovchi uchun eng ko'p hujjat va yechim shu tizimga mavjud.
5. **Size (plan) tanlang** — boshlash uchun:
   - **Test/o'rganish**: 1 vCPU / 1GB RAM / 25GB SSD (~$6/oy).
   - **Kichik prod (Node app + kichik DB)**: 2 vCPU / 2-4GB RAM (~$12-18/oy). 1GB RAM'da `npm run build` xotira yetmay xato berishi mumkin (buni swap bilan hal qilamiz).
6. **Authentication** — iloji bo'lsa **SSH key** tanlang (pastda o'rgatamiz). Agar hozircha bilmasangiz, **Password** tanlang va keyin key'ga o'tasiz.
7. **Hostname** bering (masalan, `my-app-server`) va **Create** bosing.

Bir daqiqada server tayyor bo'ladi va sizga **IP-manzil** beriladi, masalan `164.92.10.50`. Bu manzil orqali ulanamiz.

**⚠️ Ehtiyot bo'l:** IP-manzil va (agar parol tanlagan bo'lsangiz) root parolni xavfsiz saqlang. Root parol email'ga kelishi mumkin — uni birinchi kirishdayoq o'zgartiring.

---

## SSH nima va serverga ulanish

**SSH** (Secure Shell) — bu shifrlangan (encrypted) tarmoq protokoli bo'lib, uzoqdagi serverga terminal orqali xavfsiz ulanish va buyruq berish imkonini beradi. Barcha ma'lumot shifrlanadi, shuning uchun parol yoki kod yo'lda o'g'irlanmaydi.

Serverga birinchi ulanish (root parol bilan):

```bash
ssh root@164.92.10.50
```

Birinchi marta ulanganda "fingerprint" haqida savol chiqadi — `yes` deb yozing. Keyin parol so'raydi (parol terganда ekranda hech narsa ko'rinmaydi — bu normal, shunchaki tering va Enter bosing).

**💡 Tushuncha:** `ssh root@IP` buyrug'i "root nomli foydalanuvchi sifatida shu IP'dagi serverga ula" degani. `root` — Linux'dagi eng yuqori huquqli foydalanuvchi (Windows'dagi Administrator kabi, lekin undan ham kuchliroq — hamma narsani o'chirib yubora oladi).

**Windows'da SSH:** Windows 10/11'da SSH allaqachon o'rnatilgan. **PowerShell** yoki **Terminal**ni oching va xuddi shu buyruqni yozing. Agar ishlamasa, WSL (Ubuntu) yoki Git Bash ishlating.

---

## SSH key bilan parolsiz kirish

Parol bilan kirish qulay emas va xavfsizroq usul bor — **SSH key** (kalit juftligi). Bu ikki fayldan iborat:
- **private key** (maxfiy kalit) — sizning kompyuteringizda qoladi, HECH KIMGA bermaysiz.
- **public key** (ochiq kalit) — serverga qo'yasiz.

Ular matematik jufт — private key'siz hech kim kira olmaydi.

**1-qadam. O'z kompyuteringizda key yarating** (server EMAS, o'z laptopingizda):

```bash
ssh-keygen -t ed25519 -C "sizning-email@example.com"
```

Savollarga Enter bosavering (standart joy `~/.ssh/id_ed25519`, passphrase ixtiyoriy). Natijada ikki fayl paydo bo'ladi:
- `~/.ssh/id_ed25519` — private (maxfiy)
- `~/.ssh/id_ed25519.pub` — public (ochiq)

**2-qadam. Public key'ni serverga qo'shing** (eng oson yo'l):

```bash
ssh-copy-id root@164.92.10.50
```

Windows'da `ssh-copy-id` bo'lmasa, qo'lda:

```bash
# 1. Public key matnini ko'rish (o'z kompyuteringizda):
cat ~/.ssh/id_ed25519.pub

# 2. Chiqqan matnni nusxalang, serverga kirib qo'shing:
ssh root@164.92.10.50
mkdir -p ~/.ssh && chmod 700 ~/.ssh
echo "PASTE-QILINGAN-PUBLIC-KEY-BU-YERGA" >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
exit
```

**3-qadam. Endi parolsiz kiring:**

```bash
ssh root@164.92.10.50
```

Parol so'ramasdan to'g'ridan-to'g'ri kiradi.

**💡 Tushuncha (nega key xavfsizroq):** Parolni bruteforce (millionlab urinish) bilan topish mumkin, ayniqsa oddiy parol bo'lsa. SSH key esa ~50 ta belgidan iborat murakkab kriptografik kalit — uni taxmin qilish amalda imkonsiz. Shuning uchun prod serverlarda faqat key ishlatiladi, parol butunlay o'chiriladi.

**~/.ssh/config bilan qulay ulanish:** Har safar IP yozmaslik uchun o'z kompyuteringizda `~/.ssh/config` faylini yarating:

```
Host myserver
    HostName 164.92.10.50
    User deploy
    Port 22
    IdentityFile ~/.ssh/id_ed25519
```

Endi shunchaki:

```bash
ssh myserver
```

**💡 Tushuncha:** `Host myserver` — bu siz o'ylab topgan qisqa nom (alias). IP, user, port va key'ni bir marta yozib qo'yasiz, keyin faqat `ssh myserver` deysiz. Bir nechta server bo'lsa juda qulay.

---

## Boshlang'ich server sozlash

Yangi server "yalang'och" — birinchi navbatda uni yangilash va xavfsizlash kerak.

**Tizimni yangilash** (root sifatida serverda):

```bash
apt update && apt upgrade -y
```

**💡 Tushuncha:** `apt update` — mavjud paketlar ro'yxatini yangilaydi (nima yangilanishi bor-yo'qligini biladi). `apt upgrade` — aslida yangilaydi. `-y` — hamma savolga avtomatik "ha" deydi. Buni har safar server sozlashda birinchi bo'lib bajaring — ko'p xavfsizlik teshiklari eski versiyalarda bo'ladi.

Vaqt zonasini sozlash (ixtiyoriy, loglar to'g'ri vaqt ko'rsatishi uchun):

```bash
timedatectl set-timezone Asia/Tashkent
```

---

## Non-root sudo user yaratish

Root bilan doimiy ishlash xavfli — bitta noto'g'ri buyruq butun serverni o'chirib yuborishi mumkin. To'g'ri yo'l: oddiy foydalanuvchi yaratib, kerak bo'lganda `sudo` bilan root huquqini olish.

```bash
# 1. Yangi user yaratish (parol va ma'lumot so'raydi):
adduser deploy

# 2. Unga sudo (admin) huquqini berish:
usermod -aG sudo deploy
```

Endi bu user'ga SSH key'ni ko'chiring, toki u ham parolsiz kira olsin:

```bash
# root sifatida turib:
rsync --archive --chown=deploy:deploy ~/.ssh /home/deploy
```

Boshqa terminalда yangi user bilan kirishni tekshiring (eski oynani yopmang!):

```bash
ssh deploy@164.92.10.50
sudo whoami   # "root" chiqishi kerak — demak sudo ishlayapti
```

**⚠️ Ehtiyot bo'l:** Yangi user bilan kirish VA `sudo` ishlashini TASDIQLAMASDAN root'ni o'chirmang yoki SSH sozlamalarini o'zgartirmang. Aks holda o'zingizni serverdan tashqarida qoldirasiz (lockout). Har doim eski (ishlayotgan) SSH oynani ochiq qoldirib, yangisida test qiling.

---

## SSH hardening

Endi SSH'ni xavfsizlash. Config faylni tahrirlang:

```bash
sudo nano /etc/ssh/sshd_config
```

Quyidagilarni toping va o'zgartiring (yo'q bo'lsa qo'shing):

```
# root bilan to'g'ridan-to'g'ri kirishni o'chirish:
PermitRootLogin no

# Parol bilan kirishni o'chirish (faqat SSH key):
PasswordAuthentication no

# (Ixtiyoriy) portni o'zgartirish — botlar 22-portni ko'p qidiradi:
Port 2222
```

Saqlang (Ctrl+O, Enter, Ctrl+X) va SSH'ni qayta ishga tushiring:

```bash
sudo systemctl restart ssh
```

**⚠️ Ehtiyot bo'l:** `PasswordAuthentication no` qilishdan OLDIN SSH key bilan kirish ishlayotganiga 100% ishonch hosil qiling. Aks holda serverga umuman kira olmay qolasiz. Portni o'zgartirsangiz (`Port 2222`), keyingi ulanishда `ssh -p 2222 deploy@IP` yozasiz va firewall'da shu portni ochishni unutmang.

**💡 Tushuncha:** Bu uchta o'zgartirish serverni hujumlarning aksariyatidan himoya qiladi: `PermitRootLogin no` — hujumchi root parolini urib ko'ra olmaydi; `PasswordAuthentication no` — bruteforce butunlay mumkin emas; port o'zgartirish — avtomatik botlar (22-portni skanerlaydigan) sizni topmaydi.

---

## Firewall (ufw) sozlash

**ufw** (Uncomplicated Firewall) — Ubuntu'da firewall'ni oson boshqarish vositasi. Faqat kerakli portlarni ochib, qolganini yopamiz.

```bash
# SSH portini ochish (MUHIM — birinchi shuni qiling, aks holda o'zingiz qamalasiz):
sudo ufw allow OpenSSH
# Agar portni 2222 ga o'zgartirgan bo'lsangiz:
# sudo ufw allow 2222/tcp

# Web portlarini ochish (HTTP va HTTPS):
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

# Firewall'ni yoqish:
sudo ufw enable

# Holatni ko'rish:
sudo ufw status
```

**💡 Tushuncha:** Portlar — bu serverning "eshiklari". 22 — SSH (ulanish), 80 — HTTP (web), 443 — HTTPS (shifrlangan web). Firewall bularni ochiq qoldiradi, qolgan barcha portlarni yopadi. Masalan, database (5432, 3306) portini TASHQARIGA ochmaymiz — u faqat server ichida ishlashi kerak (xavfsizlik uchun).

**⚠️ Ehtiyot bo'l:** `sudo ufw enable` bosishdan oldin SSH portini (`allow OpenSSH` yoki `allow 2222`) ochganingizga ishonch hosil qiling. Aks holda firewall SSH'ni ham yopadi va serverga kira olmaysiz.

---

## Asosiy Linux boshqaruv

Serverni boshqarish uchun asosiy buyruqlar:

**Fayl va papka:**

```bash
ls -lah          # Fayllar ro'yxati (o'lcham, huquqlar bilan)
cd /var/www      # Papkaga o'tish
pwd              # Hozir qayerdaman
mkdir myapp      # Papka yaratish
rm file.txt      # Fayl o'chirish
rm -rf papka     # Papkani hamma ichidagisi bilan o'chirish (EHTIYOT!)
cp a.txt b.txt   # Nusxalash
mv a.txt b.txt   # Ko'chirish / nomini o'zgartirish
nano file.txt    # Faylni tahrirlash (oddiy muharrir)
cat file.txt     # Fayl mazmunini ko'rish
```

**Process (jarayon) boshqaruvi:**

```bash
htop             # Interaktiv process/resurs monitori (chiroyli, q — chiqish)
top              # Oddiy versiya (htop bo'lmasa)
ps aux | grep node   # node process'larni topish
kill 12345       # Process'ni to'xtatish (PID bo'yicha)
kill -9 12345    # Majburan to'xtatish
```

`htop` bo'lmasa o'rnatish: `sudo apt install htop -y`

**Disk va xotira:**

```bash
df -h            # Disk band-bo'shligi (-h = o'qishga qulay format)
du -sh *         # Har papka qancha joy egallagan
free -h          # RAM (xotira) holati
```

**Xizmatlar (services) — systemctl:**

```bash
sudo systemctl status nginx     # Xizmat holati (ishlayaptimi?)
sudo systemctl start nginx      # Ishga tushirish
sudo systemctl stop nginx       # To'xtatish
sudo systemctl restart nginx    # Qayta ishga tushirish
sudo systemctl reload nginx     # Sozlamani qayta yuklash (uzilishsiz)
sudo systemctl enable nginx     # Reboot'da avtomatik ishga tushirish
sudo journalctl -u nginx -f     # Xizmat loglarini jonli kuzatish
```

**💡 Tushuncha:** `systemctl` — Linux'da fonда doim ishlaydigan dasturlarni (Nginx, PostgreSQL, PM2) boshqaradi. `enable` va `start` farqi: `start` — hozir ishga tushiradi; `enable` — server qayta yoqilganda avtomatik ishga tushishini ta'minlaydi. Ikkalasini ham qilish kerak.

---

## Swap qo'shish

**Swap** — bu diskda ajratilgan "zaxira xotira". RAM to'lib qolganда tizim ba'zi ma'lumotni diskga (swap'ga) ko'chiradi va serverning "o'lib qolishi"ning oldini oladi. 1-2GB RAM'li serverlarda `npm run build` yoki build jarayonlari xotira yetmay yiqilmasligi uchun juda foydali.

```bash
# 1. 2GB swap fayl yaratish:
sudo fallocate -l 2G /swapfile

# 2. Huquqlarni cheklash (xavfsizlik):
sudo chmod 600 /swapfile

# 3. Swap sifatida formatlash va yoqish:
sudo mkswap /swapfile
sudo swapon /swapfile

# 4. Reboot'dan keyin ham saqlanishi uchun:
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab

# Tekshirish:
free -h    # "Swap" qatorida 2.0Gi ko'rinishi kerak
```

**💡 Tushuncha:** Swap RAM'dan sekinroq (chunki disk RAM'dan sekin), lekin u "xavfsizlik yostig'i" — xotira tugab qolganда serverni butunlay yiqilishdan saqlaydi. Ayniqsa arzon (1GB RAM) planlarda swap qo'shish deyarli majburiy. Odatda RAM hajmiga teng yoki 2 baravar swap beriladi.

---

## Amaliy Q&A

### ❓ "Permission denied (publickey)" xatosi chiqyapti, kira olmayapman

**✅ Javob:** SSH key server tomonidan qabul qilinmayapti. Sabablar:
1. Public key `~/.ssh/authorized_keys`ga to'g'ri qo'shilmagan — `cat ~/.ssh/authorized_keys` bilan tekshiring (agar kira olsangiz).
2. Fayl huquqlari noto'g'ri — serverda: `chmod 700 ~/.ssh && chmod 600 ~/.ssh/authorized_keys`.
3. Noto'g'ri key ishlatilyapti — `ssh -i ~/.ssh/id_ed25519 deploy@IP` bilan aniq key ko'rsating.
4. Batafsil xatoni ko'rish: `ssh -v deploy@IP` (verbose rejim).

Agar butunlay kira olmasangiz — provayder panelidagi **Web Console / VNC** orqali kiring va sozlashni to'g'rilang.

### ❓ Root'ni o'chirib qo'ydim va endi serverga umuman kira olmayapman

**✅ Javob:** Deyarli har bir provayder **Web Console** (browser orqali server terminaliga to'g'ridan-to'g'ri kirish) beradi — DigitalOcean'da "Access → Launch Console", Hetzner'da "Console". Bu SSH'dan o'tmaydi, shuning uchun har doim ishlaydi. Undan kirib, `sudo nano /etc/ssh/sshd_config`da sozlamani tuzating va `sudo systemctl restart ssh` qiling. Kelajakda bunday holatdan qochish uchun: sozlamani o'zgartirishdan oldin YANGI oynada kirishni tekshiring, eskisini yopmang.

### ❓ `sudo: command not found` yoki user'da sudo huquqi yo'q

**✅ Javob:** User `sudo` guruhida emas. Root sifatida kiring (yoki Web Console orqali) va: `usermod -aG sudo deploy`. Keyin user'ni to'liq qayta login qiling (chiqib-kiring) — guruh o'zgarishi yangi sessiyada kuchga kiradi.

### ❓ `apt update` "Could not get lock" yoki xato beryapti

**✅ Javob:** Boshqa `apt` jarayoni ishlab turibdi (masalan, avtomatik yangilanish). Bir-ikki daqiqa kuting va qayta urinib ko'ring. Agar davom etsa: `sudo lsof /var/lib/dpkg/lock-frontend` bilan qaysi process band qilganini ko'ring, tugashini kuting. Oxirgi chora: serverni `sudo reboot` qiling.

### ❓ Server juda sekin ishlayapti, ba'zan javob bermayapti

**✅ Javob:** Ehtimol RAM yoki disk to'lgan. Tekshiring: `free -h` (RAM), `df -h` (disk). Agar RAM to'la bo'lsa — swap qo'shing (yuqoridagi bo'lim) yoki ortiqcha process'larni `htop`da toping va to'xtating. Disk to'la bo'lsa — eski log/build fayllarni tozalang: `sudo journalctl --vacuum-time=3d`, `sudo apt clean`.

### ❓ `ufw enable` qilgandan keyin SSH uzildi

**✅ Javob:** SSH portini ochishdan oldin firewall'ni yoqib yuborgansiz. **Web Console** orqali kiring va `sudo ufw allow OpenSSH` (yoki `sudo ufw allow 2222/tcp` agar port o'zgartirilgan bo'lsa) qiling. Kelajakда: har doim SSH portini ochib, keyingina `ufw enable` qiling.

### ❓ Portni o'zgartirdim, endi `ssh deploy@IP` ishlamayapti

**✅ Javob:** Yangi port bilan ulanish kerak: `ssh -p 2222 deploy@IP`. Doimiy qulaylik uchun `~/.ssh/config`da `Port 2222` yozib qo'ying. Shuningdek firewall'da yangi portni ochganingizni (`sudo ufw allow 2222/tcp`) va eski 22-port SSH configда haqiqatan o'zgarганини tekshiring.

### ❓ SSH sessiyani yopgach, ishga tushirgan dasturim ham to'xtab qoladi

**✅ Javob:** Bu normal — SSH terminalда ishga tushirilgan process siz chiqganda o'ladi. Yechim: dasturni **process manager** (PM2) yoki **systemd service** orqali fonда ishlating. Vaqtincha test uchun `nohup node app.js &` yoki `screen`/`tmux` ishlatish mumkin. To'liq yo'l — keyingi bo'limda (PM2).

### ❓ "Connection timed out" — ulanolmayapman

**✅ Javob:** Sabablar: (1) noto'g'ri IP; (2) server o'chiq — provayder panelидан tekshiring; (3) firewall SSH portini yopgan; (4) portni o'zgartирган, lekin eski port bilan urinyapsiz. IP va portni tekshiring: `ping IP` (server javob beryaptimi), `ssh -v -p PORT user@IP` (batafsil).

### ❓ Serverni yangilaganda "reboot required" yoki kernel yangilandi

**✅ Javob:** Ba'zi yangilanishlar (ayniqsa kernel) qayta ishga tushirishni talab qiladi. Xavfsiz vaqtda: `sudo reboot`. Server ~30 soniyada qayta yuklanadi, keyin SSH bilan qayta ulanasiz. `enable` qilingan xizmatlar (Nginx, PM2) avtomatik ishga tushadi.

### ❓ Har safar parol so'rayapti, key ishlamayapti (kirsam ham)

**✅ Javob:** Public key serverga qo'shilmagan yoki noto'g'ri userga qo'shilgan. `ssh-copy-id deploy@IP` bilan qayta qo'shing. Agar `~/.ssh` huquqlari noto'g'ri bo'lsa ham parol so'raydi — server tomonda: `chmod 700 ~/.ssh && chmod 600 ~/.ssh/authorized_keys`.

---

## Amaliy Checklist

- [ ] VPS provayder tanladim (DigitalOcean yoki Hetzner) va Ubuntu 24.04 LTS bilan server yaratdim
- [ ] To'g'ri region tanladim (foydalanuvchilarga yaqin — masalan Frankfurt)
- [ ] `ssh root@IP` bilan birinchi marta ulandim
- [ ] O'z kompyuterimda `ssh-keygen` bilan SSH key yaratdim
- [ ] Public key'ni serverga qo'shdim va parolsiz kirish ishlayapti
- [ ] `~/.ssh/config` bilan qulay alias (`ssh myserver`) sozladim
- [ ] `apt update && apt upgrade -y` bilan tizimni yangiladim
- [ ] Non-root sudo user (`deploy`) yaratdim va unga key ko'chirdim
- [ ] Yangi user + `sudo` ishlashini TASDIQLADIM (eski oynani yopmasdan)
- [ ] SSH hardening: `PermitRootLogin no`, `PasswordAuthentication no` qildim
- [ ] `ufw` bilan 22 (yoki 2222), 80, 443 portlarni ochdim va firewall'ni yoqdim
- [ ] Asosiy Linux buyruqlar bilan tanishdim (`htop`, `df`, `systemctl`)
- [ ] Swap qo'shdim (ayniqsa RAM 1-2GB bo'lsa)

---

← [Deployment bo'limiga qaytish](./README.md)
