# VPS'da Node + Nginx + PM2

Serverni sozlab bo'ldingiz (05-bo'lim). Endi eng qiziq qismi — **Node.js loyihangizni VPS'ga to'liq deploy qilish**. Bu qo'llanmada Node'ni o'rnatishdan tortib, PM2 bilan app'ni doim ishlatish, Nginx orqali reverse proxy qilish, domen ulash va bepul SSL (HTTPS) o'rnatishgacha — hammasini real buyruqlar bilan qadama-qadam qilamiz (2026-yil, Ubuntu 24.04).

## Mundarija

- [Umumiy arxitektura](#umumiy-arxitektura)
- [1. Node.js o'rnatish (nvm bilan)](#1-nodejs-ornatish-nvm-bilan)
- [2. Loyihani serverga olib kelish](#2-loyihani-serverga-olib-kelish)
- [3. PM2 — process manager](#3-pm2--process-manager)
- [PM2 cluster mode](#pm2-cluster-mode)
- [4. Nginx reverse proxy](#4-nginx-reverse-proxy)
- [5. Domain + SSL (certbot)](#5-domain--ssl-certbot)
- [6. Deploy'ni yangilash (zero-downtime)](#6-deployni-yangilash-zero-downtime)
- [Statik fayllarni Nginx orqali berish](#statik-fayllarni-nginx-orqali-berish)
- [Xatolarni topish (loglar)](#xatolarni-topish-loglar)
- [Amaliy Q&A (troubleshooting)](#amaliy-qa)
- [Amaliy Checklist](#amaliy-checklist)

---

## Umumiy arxitektura

Deploy qilingan tizim shunday ishlaydi:

```
Foydalanuvchi  →  Domain (misol.uz)  →  Server IP  →  Nginx (80/443)  →  Node app (localhost:3000)
                                                          ↑
                                                    SSL (Let's Encrypt)
                                                          
                                          PM2 → Node app'ni doim ishlatib turadi
```

**💡 Tushuncha:** Foydalanuvchi domenga kiradi → so'rov serverga keladi → **Nginx** uni qabul qiladi (80/443-portда) va **Node app'ga** (masalan localhost:3000) uzatadi. **PM2** esa Node app'ni orqada doim ishlab turishini, crash bo'lsa qayta ishga tushishini ta'minlaydi. **Certbot** HTTPS'ni bepul beradi. Har birini alohida sozlaymiz.

---

## 1. Node.js o'rnatish (nvm bilan)

Node'ni to'g'ridan-to'g'ri `apt` bilan o'rnatish mumkin, lekin **nvm** (Node Version Manager) yaxshiroq — versiyani oson almashtirasiz va sudo shart emas.

`deploy` user sifatida kiring va nvm o'rnating:

```bash
# nvm o'rnatish skripti:
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.1/install.sh | bash

# Terminalni qayta yuklash (yoki chiqib-kirish):
source ~/.bashrc

# Node LTS (barqaror) versiyasini o'rnatish:
nvm install --lts

# Tekshirish:
node -v    # masalan v22.x.x
npm -v
```

**💡 Tushuncha:** `nvm install --lts` eng so'nggi LTS (Long Term Support) Node versiyasini o'rnatadi. LTS — production uchun tavsiya etilgan barqaror versiya. Agar loyihangiz aniq versiya talab qilsa: `nvm install 20` kabi yozing.

**⚠️ Ehtiyot bo'l:** nvm faqat o'zi o'rnatilgan user uchun ishlaydi. Agar `deploy` bilan o'rnatgan bo'lsangiz, Node ham faqat `deploy`da bo'ladi. PM2 startup sozlashда bu muhim bo'ladi — o'sha user bilan ishlang.

---

## 2. Loyihani serverga olib kelish

Eng keng tarqalgan yo'l — **git clone** (GitHub/GitLab'dan).

```bash
# Loyihalar uchun papka:
mkdir -p ~/apps && cd ~/apps

# Repozitoriyani klonlash:
git clone https://github.com/username/my-app.git
cd my-app

# Kutubxonalarni o'rnatish (faqat production paketlar):
npm ci --omit=dev
# yoki oddiy:
npm install

# Agar build kerak bo'lsa (TypeScript, bundler):
npm run build
```

**Environment (.env) sozlash** — maxfiy ma'lumotlar (DB parol, API kalitlar):

```bash
nano .env
```

Ichiga yozing (misol):

```
NODE_ENV=production
PORT=3000
DATABASE_URL=postgres://user:pass@localhost:5432/mydb
JWT_SECRET=juda-maxfiy-uzun-satr
```

**⚠️ Ehtiyot bo'l:** `.env` faylni HECH QACHON git'ga qo'shmang (`.gitignore`da bo'lsin). U faqat serverda qo'lda yaratiladi. Aks holda maxfiy kalitlar GitHub'ga chiqib ketadi — bu jiddiy xavfsizlik xatosi.

**💡 Tushuncha (private repo):** Agar repo maxfiy (private) bo'lsa, klonlash uchun **deploy key** yoki **personal access token** kerak. Eng oson: GitHub'da repo → Settings → Deploy keys → serverdagi `~/.ssh/id_ed25519.pub`ni qo'shing, keyin `git clone git@github.com:username/my-app.git` (SSH orqali). Kelajakda CI/CD (GitHub Actions) bilan avtomatlashtirish mumkin, lekin boshlash uchun qo'lda git clone yetarli.

App to'g'ri ishlashini tekshiring:

```bash
node index.js   # yoki npm start
# Boshqa terminalда: curl http://localhost:3000
# Ishlagach Ctrl+C bilan to'xtating — endi PM2 bilan doimiy ishlatamiz
```

---

## 3. PM2 — process manager

**PM2** — bu Node.js uchun **process manager**: app'ni fonда doim ishlatib turadi, crash bo'lsa avtomatik qayta ishga tushiradi, server reboot bo'lsa ham avtomatik yoqadi va loglarni boshqaradi.

**Nega kerak?** `node index.js`ni terminalда ishga tushirsangiz, SSH'dan chiqishingiz bilan app o'ladi. Va agar app crash qilsa, hech kim uni qayta ishga tushirmaydi. PM2 bu ikki muammoni hal qiladi.

PM2'ni global o'rnating va app'ni ishga tushiring:

```bash
# Global o'rnatish:
npm install -g pm2

# App'ni ishga tushirish (nom berib):
pm2 start index.js --name my-app

# Agar npm start orqali bo'lsa:
pm2 start npm --name my-app -- start
```

**Reboot'da avtomatik ishga tushirish** (juda muhim):

```bash
# 1. startup skriptini yaratish (chiqqan buyruqni nusxalab bajaring):
pm2 startup
# Chiqadi: "sudo env PATH=... pm2 startup systemd -u deploy --hp /home/deploy"
# ← shu buyruqni nusxalab, sudo bilan bajaring

# 2. Joriy process ro'yxatini saqlash:
pm2 save
```

**💡 Tushuncha:** `pm2 startup` — server qayta yoqilganда PM2 avtomatik ishga tushishini sozlaydi (systemd service yaratadi). `pm2 save` — hozir ishlab turган app'lar ro'yxatini saqlaydi, toki reboot'dan keyin aynan o'shalar tiklansин. Ikkalasini ham qilmasangiz, server reboot bo'lganда app o'chib qoladi.

**Kundalik PM2 buyruqlari:**

```bash
pm2 list              # Barcha app'lar holati (online/stopped)
pm2 restart my-app    # Qayta ishga tushirish
pm2 stop my-app       # To'xtatish
pm2 delete my-app     # PM2'dan olib tashlash
pm2 reload my-app     # Uzilishsiz qayta yuklash (zero-downtime)
```

**Log kuzatish:**

```bash
pm2 logs              # Barcha app loglari (jonli)
pm2 logs my-app       # Faqat bitta app logi
pm2 logs --lines 100  # Oxirgi 100 qatorni ko'rish
pm2 monit             # Interaktiv monitor (CPU, RAM, loglar birga)
pm2 flush             # Eski loglarni tozalash
```

**💡 Tushuncha:** `pm2 logs` — app nima yozayotganini (console.log, xatolar) jonli ko'rsatadi. Deploy'dan keyin birinchi navbatda shuni ochib, app to'g'ri ishga tushganini tekshiring. `pm2 monit` esa CPU/RAM ishlatilishini ham ko'rsatadi — resurs muammosini topishда juda foydali.

### PM2 cluster mode

Server bir nechta CPU yadrosiga ega bo'lsa, **cluster mode** bilan app'ning bir nechta nusxasini ishga tushirib, barcha yadrolardan foydalanish mumkin:

```bash
# Barcha CPU yadrolarини ishlatish:
pm2 start index.js --name my-app -i max

# yoki aniq son (masalan 4 ta):
pm2 start index.js --name my-app -i 4
```

**💡 Tushuncha:** Oddiy Node bitta yadroда ishlaydi (single-threaded). Cluster mode app'ning bir nechta nusxasини ishga tushiradi va yuk (so'rovlar)ni ular orasида taqsimlaydi — bu ko'proq foydalanuvchini ko'tarish imkonини beradi. `-i max` = mavjud barcha yadrolar.

**⚠️ Ehtiyot bo'l:** Cluster mode faqat **stateless** app'lar uchun to'g'ri ishlaydi. Agar app xotirasида (in-memory) sessiya yoki holat saqlasa, har bir nusxada u alohida bo'ladi va muammo chiqadi. Sessiyani Redis yoki DB'da saqlang. WebSocket ham sticky-session talab qiladi.

---

## 4. Nginx reverse proxy

**Nginx** — kuchli web-server. Bu yerda uni **reverse proxy** sifatida ishlatamiz: tashqi so'rovlarni (80/443-port) qabul qilib, ichki Node app'ga (localhost:3000) uzatadi.

**Nega kerak?**
1. Node app'ni to'g'ridan-to'g'ri 80/443-portда ochish xavfli va sudo talab qiladi.
2. Nginx SSL (HTTPS), gzip siqish, statik fayl berish, load balancing kabi ishlarni Node'dan yaxshiroq bajaradi.
3. Bitta serverда bir nechta app'ni (domen bo'yicha) ajratish mumkin.

Nginx o'rnatish:

```bash
sudo apt update
sudo apt install nginx -y

# Ishga tushirish va enable:
sudo systemctl enable --now nginx

# Brauzerда server IP'ni ochib tekshiring — "Welcome to nginx" chiqishi kerak
```

**Server block (config)** yarating. Har bir domen/app uchun alohida fayl:

```bash
sudo nano /etc/nginx/sites-available/my-app
```

Ichiga (domeningizni yozing; hozircha domen yo'q bo'lsa IP ham bo'ladi):

```nginx
server {
    listen 80;
    server_name misol.uz www.misol.uz;

    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
    }
}
```

**💡 Tushuncha:** `proxy_pass http://localhost:3000` — bu qatorда butun sehr: Nginx'ga kelgan so'rovni Node app'ga (3000-port) uzat. `proxy_set_header` qatorlar esa foydalanuvchi haqidagi ma'lumotni (real IP, protokol, WebSocket upgrade) Node app'ga uzatadi — busiz app hamma so'rovни "localhost'dan kelgan" deb o'ylaydi.

Config'ni yoqing va tekshiring:

```bash
# sites-enabled'ga symbolic link (yoqish):
sudo ln -s /etc/nginx/sites-available/my-app /etc/nginx/sites-enabled/

# Standart demo config'ni o'chirish (ixtiyoriy, konflikt bo'lmasligi uchun):
sudo rm /etc/nginx/sites-enabled/default

# Config sintaksisини tekshirish (MUHIM):
sudo nginx -t

# Xato yo'q bo'lsa, qayta yuklash (uzilishsiz):
sudo systemctl reload nginx
```

**💡 Tushuncha:** `sudo nginx -t` — config faylда xato bor-yo'qligini tekshiradi va HECH NARSANI o'zgartirmaydi. Doim `reload`dan OLDIN buni bajaring — noto'g'ri config bilan `reload` qilsangiz Nginx umuman ishga tushmay qolishi mumkin. `reload` — configни qayta o'qiydi, lekin mavjud ulanishlarni uzmaydi (`restart`dan yaxshiroq).

Endi brauzerда `http://misol.uz` (yoki IP) ochilsa, Node app'ingiz ko'rinishi kerak.

---

## 5. Domain + SSL (certbot)

### Domenni serverga ulash

Domeningiz DNS sozlamalarида (Namecheap, Cloudflare, GoDaddy — qayerdan olган bo'lsangiz) **A record** qo'shing:

| Type | Host/Name | Value (IP) |
|------|-----------|------------|
| A | @ | 164.92.10.50 |
| A | www | 164.92.10.50 |

**💡 Tushuncha:** **A record** — domenni server IP-manziliga bog'laydi. `@` — asosiy domen (misol.uz), `www` — www.misol.uz uchun. DNS o'zgarishlari tarqalishi (propagation) uchun bir necha daqiqadan bir necha soatgacha vaqt ketishi mumkin. Tekshirish: `dig misol.uz +short` yoki `nslookup misol.uz` — server IP'ingiz chiqishi kerak.

### Bepul SSL (Let's Encrypt) — certbot bilan

**Certbot** — Let's Encrypt'dan bepul SSL sertifikat oladi va Nginx config'ini avtomatik HTTPS'ga sozlaydi.

```bash
# Certbot o'rnatish:
sudo apt install certbot python3-certbot-nginx -y

# Sertifikat olish va Nginx'ni avtomatik sozlash:
sudo certbot --nginx -d misol.uz -d www.misol.uz
```

Certbot email so'raydi, shartlarга rozilik so'raydi va HTTP → HTTPS redirect qo'shishни taklif qiladi (2/Redirect tanlang). Tugagach, `https://misol.uz` yashil qulf bilan ishlaydi.

**💡 Tushuncha:** Certbot avtomatik ravishда: (1) domeningizni tekshiradi (haqiqatan sizniki ekanini), (2) SSL sertifikat oladi, (3) Nginx config'iga `listen 443 ssl` va sertifikat yo'llarini qo'shadi, (4) HTTP'ni HTTPS'ga yo'naltiradi. Qo'lда hech narsa sozlash shart emas.

**Avtomatik yangilanish:** Let's Encrypt sertifikatlari 90 kun amal qiladi. Certbot o'rnatilganда avtomatik yangilanish (systemd timer) ham sozlanadi. Tekshirish:

```bash
# Yangilanish sinovi (haqiqiy yangilamaydi, faqat tekshiradi):
sudo certbot renew --dry-run
```

Agar `dry-run` muvaffaqiyatli o'tsa, sertifikat 90 kun tugashiga yaqin avtomatik yangilanadi — hech narsa qilishingiz shart emas.

**⚠️ Ehtiyot bo'l:** Certbot ishlashi uchun 80-port ochiq bo'lishi kerak (firewall'da `ufw allow 80/tcp`) va domen A record haqiqatan serveringizga ishora qilishi kerak. Agar DNS hali tarqalmagan bo'lsa, certbot "challenge failed" xatosi beradi — DNS tarqalguncha kuting.

---

## 6. Deploy'ni yangilash (zero-downtime)

Kodni o'zgartirgach, serverga yangi versiyани chiqarish jarayoni:

```bash
cd ~/apps/my-app

# 1. Yangi kodni olish:
git pull origin main

# 2. Yangi kutubxonalar (agar package.json o'zgargan bo'lsa):
npm ci --omit=dev

# 3. Qayta build (kerak bo'lsa):
npm run build

# 4. Uzilishsiz qayta yuklash (zero-downtime):
pm2 reload my-app
```

**💡 Tushuncha (reload vs restart):** `pm2 restart` — app'ни to'xtatib, qaytadan ishga tushiradi (bir necha soniya "downtime" bo'ladi). `pm2 reload` — cluster mode'да eski nusxalar yangi so'rovни qabul qilib turган holda birma-bir yangilanadi, natijada foydalanuvchi hech qanday uzilishni sezmaydi (zero-downtime). Prodда doim `reload` ishlating.

Bu qadamlarни bitta skriptга (`deploy.sh`) yig'ib qo'yish qulay:

```bash
#!/bin/bash
set -e   # birinchi xatoда to'xtash
cd ~/apps/my-app
git pull origin main
npm ci --omit=dev
npm run build
pm2 reload my-app
echo "Deploy tugadi!"
```

Ishga tushirish: `chmod +x deploy.sh && ./deploy.sh`

---

## Statik fayllarni Nginx orqali berish

Agar loyihangizда statik fayllar (rasm, CSS, JS, yuklamalar) bo'lsa, ularни Node emas, to'g'ridan-to'g'ri **Nginx** bersin — bu ancha tezroq va Node'ga yuk tushmaydi.

Nginx config'iga qo'shing:

```nginx
server {
    listen 80;
    server_name misol.uz;

    # Statik fayllar — to'g'ridan-to'g'ri Nginx beradi:
    location /static/ {
        alias /home/deploy/apps/my-app/public/;
        expires 30d;
        access_log off;
    }

    # Qolган hammasi — Node app'ga:
    location / {
        proxy_pass http://localhost:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

**💡 Tushuncha:** `expires 30d` — brauzerga statik fayllarни 30 kun keshlashni aytadi (qayta-qayta yuklab olmaydi, sayt tezlashadi). Nginx statik fayllarни Node'дан o'nlab marta tezroq beradi, chunki bu uning asosiy vazifasi. Frontend build (React `dist/`) ham shu tarzda beriladi.

---

## Xatolarni topish (loglar)

Biror narsa ishlamasa, birinchi navbatда loglarga qarang:

**PM2 (Node app) loglari:**

```bash
pm2 logs my-app --lines 50    # App xatolari (crash, exception)
pm2 monit                     # Jonli CPU/RAM/log
```

**Nginx loglari:**

```bash
sudo tail -f /var/log/nginx/error.log    # Nginx xatolari (jonli)
sudo tail -f /var/log/nginx/access.log   # Kelган so'rovlar
```

**💡 Tushuncha:** Muammoni topish tartibi: (1) `pm2 logs` — Node app o'zi crash qilyaptimi yoki xato beryaptimi? (2) `sudo nginx -t` — Nginx config to'g'rimi? (3) Nginx `error.log` — proxy ishlayaptimi, Node'ga yeta olyaptimi? Ko'p muammo "502 Bad Gateway" — bu odatда Node app ishlamayapti degани (PM2'ni tekshiring).

---

## Amaliy Q&A

### ❓ "502 Bad Gateway" xatosi chiqyapti

**✅ Javob:** Nginx Node app'ga yeta olmayapti. Sabablar: (1) Node app ishlamayapti — `pm2 list` bilan tekshiring (`online` bo'lishi kerak); (2) app boshqa portда ishlayapti — `.env`даги `PORT` va Nginx `proxy_pass`даги port bir xilми? (3) app crash qildi — `pm2 logs my-app` bilan xatoni ko'ring. Odатда `pm2 restart my-app` yordam beradi, lekin avval log'даги asl sababни toping.

### ❓ Domen ochilmayapti, "This site can't be reached"

**✅ Javob:** (1) DNS A record to'g'ri sozlanganмi va tarqalдими? — `dig misol.uz +short` server IP'ingizни ko'rsatishi kerak. (2) Firewall 80/443-portни ochганми? — `sudo ufw status`. (3) Nginx ishlayaptими? — `sudo systemctl status nginx`. DNS yangi bo'lsa, tarqalishга bир necha soat ketishи mumkin.

### ❓ `pm2 startup` qildim, lekin reboot'дан keyin app ishга tushmади

**✅ Javob:** Ikki sabab: (1) `pm2 startup`дан keyin `pm2 save` qilmagansiz — save qilinмаган app'lar tiklanмaydi. (2) `pm2 startup` chiqарган `sudo env PATH=...` buyruғини bajarмagansiz — uni to'liq nusxalаб bajaring. Qayta: `pm2 start ... && pm2 save && pm2 startup` (chiqган buyruqни sudo bilan).

### ❓ `nginx -t` "failed" beryapti

**✅ Javob:** Config'да sintaksis xatosi bor. Xato xabari qайси faylда, nechanchi qatorда ekanини aniq ko'rsatadi — o'shани o'qing. Ko'p uchraydigan xato: qatorlarда `;` (nuqtali vergul) tushiб qolган yoki `{ }` qavslar mos kelmаган. Tuzatib, qayта `sudo nginx -t`, keyin `sudo systemctl reload nginx`.

### ❓ Certbot "challenge failed" yoki sertifikat olмaяpti

**✅ Javob:** (1) Domen A record haqiqатан serveringizга ishora qilyaptими? — `dig misol.uz +short`. (2) 80-port ochiqми? — Let's Encrypt uni tekshirish uchun ishlatadi. (3) DNS hali tarqалmagan bo'lsa kuting. (4) Nginx `server_name`даги domen certbot'даги `-d` domenи bilan bir xilми? Bir necha marta muvaffaqiyатsiz urinсangiz Let's Encrypt vaqтинча rate-limit qo'yishи mumkin — bir soat kuting.

### ❓ `npm run build` "JavaScript heap out of memory" beryapti

**✅ Javob:** RAM yetмаяpti (odатда 1GB serverларда). Yechim: (1) Swap qo'shing (05-bo'limда ko'rsатилган); (2) build'ни mahalliy kompyuterда qiling va faqat natijани (`dist/`) serverга yuklang; (3) build vaqтида RAM limitини oshiring: `NODE_OPTIONS="--max-old-space-size=1024" npm run build`.

### ❓ SSH'dan chiqсам app to'xтаб qoladi

**✅ Javob:** App'ни PM2 orqali ishga tushirмаgansiz (to'g'ridan-to'g'ri `node index.js` qilгансиз). PM2 fonда ishlатади: `pm2 start index.js --name my-app && pm2 save`. Endi SSH'дан chiqсangiz ham app ishлайверади.

### ❓ WebSocket (Socket.io) ishlаmayapti Nginx orqali

**✅ Javob:** Nginx config'да WebSocket upgrade header'lари bo'lishi shart: `proxy_set_header Upgrade $http_upgrade;` va `proxy_set_header Connection 'upgrade';` (yuqоридаги config'да bor). Bularsiz WebSocket ulanмайди. Config'ни tekshirib, `sudo nginx -t && sudo systemctl reload nginx` qiling.

### ❓ Git pull "permission denied" yoki "authentication failed"

**✅ Javob:** Private repo uchun autentifikatsiya kerak. Yechim: serverда SSH deploy key sozlаng — `ssh-keygen` bilan key yarating, `~/.ssh/id_ed25519.pub`ни GitHub repo → Settings → Deploy keys'ga qo'shing, keyin `git clone git@github.com:...` (HTTPS emas, SSH URL). Yoki HTTPS uchun personal access token ishlating.

### ❓ Deploy'дан keyin eski versiya ko'rinyapti (o'zgarish ko'rinмaяpti)

**✅ Javob:** (1) `pm2 reload my-app` qildingizми? — app qayта yuklanмаса eski kod ishlайверади. (2) Build qilишни unutдingizми? (`npm run build`). (3) Brauzer keshi — Ctrl+Shift+R bilan hard refresh qiling. (4) Nginx statik keshi — `expires` qo'yган fayллар brauzerда keshланган bo'lishi mumkin.

### ❓ App ishlаyди, lekin foydalanuvchi real IP'sini emas, `127.0.0.1` ni ko'ryapman

**✅ Javob:** Node app Nginx orqасидан kelган so'rovларни "localhost'дан kelган" deб ko'ryapti. Nginx config'га `proxy_set_header X-Real-IP $remote_addr;` va `X-Forwarded-For` qo'shинг (yuqоридаги config'да bor). App tomonида `req.headers['x-forwarded-for']`ни o'qing yoki `app.set('trust proxy', true)` (Express) qiling.

### ❓ Bitta serverда bir nechta app (domen) qanday joylаshtiraman?

**✅ Javob:** Har app'ни boshqa portда ishlатing (3000, 3001, ...) va har biriга alohida Nginx server block yarating (har xil `server_name` va `proxy_pass`). PM2'да har biriга alohida `--name` bering. Har bir domenга alohida certbot SSL oling. Shunday qilib bitta VPS o'nlаб saytни ko'tара oladi.

---

## Amaliy Checklist

- [ ] nvm bilan Node.js LTS o'rnatдим (`node -v` ishlаyди)
- [ ] Loyihани `git clone` bilan serverга olдим
- [ ] `npm ci` / `npm run build` bajардим va `.env` sozладим (git'га qo'shмаган holda)
- [ ] App'ни `node index.js` bilan bир marta test qilдим (localhost:3000 ishлayди)
- [ ] PM2 o'rnатдим va `pm2 start ... --name`bilan ishga tushiрдим
- [ ] `pm2 startup` + `pm2 save` qildim (reboot'да avtomatik ishга tushади)
- [ ] `pm2 logs` / `pm2 monit` bilan log kuzатишни bilaman
- [ ] (Ixtiyoriy) Cluster mode (`-i max`) — agar stateless app bo'lsa
- [ ] Nginx o'rnатдим va reverse proxy server block yarат dim
- [ ] `sudo nginx -t` xato berмади va `reload` qildim — sayt IP orqali ochilди
- [ ] Domen A record'ни server IP'ga sozладим (@ va www)
- [ ] `certbot --nginx` bilan bepul SSL oldim — HTTPS ishlаyди
- [ ] `certbot renew --dry-run` muvaffaqiyатли (avtomatik yangilaнади)
- [ ] Deploy skriptи (git pull → build → `pm2 reload`) tayyor — zero-downtime
- [ ] Statik fayлларни Nginx orqali beryapman (kerak bo'lsa)
- [ ] Loglarни (`pm2 logs`, Nginx `error.log`) qayerдан ko'rishни bilaman

---

← [Deployment bo'limiga qaytish](./README.md)
