# Log, Monitoring va Security

Serverni deploy qilib qo'yish — bu ishning faqat yarmi. Asl ish undan keyin boshlanadi: server ishlab turibdimi, xatolar chiqyaptimi, kimdir sizni buzishga urinyaptimi, disk to'lib qolmadimi? Bularning barchasini bilish uchun sizga uchta narsa kerak — **loglar** (nima bo'layotganini ko'rish), **monitoring** (holatni doimiy kuzatish) va **security** (himoya). Bu bo'lim aynan shu haqda: mavhum nazariya emas, balki real Ubuntu 24.04 serverda ishlaydigan amaliy qadamlar.

## Mundarija

- [Loglar (log kuzatish)](#loglar-log-kuzatish)
  - [Loglar nima va nega muhim](#loglar-nima-va-nega-muhim)
  - [Tizim loglari: journalctl](#tizim-loglari-journalctl)
  - [Nginx loglari](#nginx-loglari)
  - [PM2 loglari va log rotation](#pm2-loglari-va-log-rotation)
  - [Application logging best practice](#application-logging-best-practice)
  - [Disk to'lib ketmasligi: logrotate](#disk-tolib-ketmasligi-logrotate)
- [Monitoring](#monitoring)
  - [Serverni real-time kuzatish](#serverni-real-time-kuzatish)
  - [Uptime monitoring (UptimeRobot)](#uptime-monitoring-uptimerobot)
  - [Resource monitoring (Netdata / Prometheus + Grafana)](#resource-monitoring-netdata--prometheus--grafana)
  - [Xatolarni kuzatish (Sentry)](#xatolarni-kuzatish-sentry)
  - [Health check](#health-check)
- [Security (server xavfsizligi)](#security-server-xavfsizligi)
  - [SSH hardening](#ssh-hardening)
  - [Firewall: ufw](#firewall-ufw)
  - [fail2ban: brute-force himoyasi](#fail2ban-brute-force-himoyasi)
  - [Tizimni yangilab turish](#tizimni-yangilab-turish)
  - [Secret'larni to'g'ri saqlash](#secretlarni-togri-saqlash)
  - [Nginx rate limiting va security header'lar](#nginx-rate-limiting-va-security-headerlar)
  - [Backup strategiyasi (3-2-1)](#backup-strategiyasi-3-2-1)
  - [Buzilgan serverni aniqlash belgilari](#buzilgan-serverni-aniqlash-belgilari)
- [Amaliy Q&A](#amaliy-qa)
- [Amaliy Checklist](#amaliy-checklist)

---

## Loglar (log kuzatish)

### Loglar nima va nega muhim

**💡 Tushuncha:** Log — bu serverda va dasturingizda sodir bo'lgan hodisalarning yozib borilgan tarixi. Har bir so'rov, har bir xato, har bir qayta ishga tushish — bularning hammasi qayerdadir yozib qoladi. Loglar bo'lmasa, siz "sayt ishlamayapti" degan xabarni olganingizda, ko'zi bog'langan holda sabab qidirasiz.

Loglar sizga quyidagilarni beradi:

- **Xatoni topish:** "500 error qaytyapti" — nega? Log aytadi.
- **Xavfsizlikni tekshirish:** kimdir SSH'ga 10000 marta parol urinib ko'rdimi? Log aytadi.
- **Performance:** qaysi so'rov sekin ishlayapti? Log aytadi.
- **Audit:** kim, qachon, nima qildi? Log aytadi.

Ubuntu'da loglar asosan uch joyda bo'ladi:

1. **Tizim loglari** — `journalctl` (systemd) orqali.
2. **Xizmat loglari** — Nginx, PostgreSQL va h.k. o'z fayllarida (`/var/log/...`).
3. **Application loglari** — sizning dasturingiz yozadigan loglar (PM2, stdout, fayl).

### Tizim loglari: journalctl

Ubuntu 24.04'da systemd barcha tizim va xizmat loglarini bir joyda saqlaydi. Ularni `journalctl` bilan o'qiysiz:

```bash
# Barcha loglar (eng oxirgisi tepada emas, oxirida)
journalctl

# Faqat oxirgi qatorlarni ko'rish va jonli (real-time) kuzatish
journalctl -f

# Muayyan xizmatning loglari (masalan nginx)
journalctl -u nginx

# Muayyan xizmatni JONLI kuzatish (eng ko'p ishlatiladigan buyruq)
journalctl -u nginx -f

# Oxirgi 100 qator
journalctl -u nginx -n 100

# Bugungi loglar
journalctl -u nginx --since today

# Ma'lum vaqt oralig'i
journalctl -u nginx --since "2026-06-30 09:00" --until "2026-06-30 12:00"

# Faqat xatolar (priority: err va yuqori)
journalctl -p err -b

# Loglar qancha joy egallayapti
journalctl --disk-usage
```

**💡 Tushuncha:** `-u` — "unit" (xizmat) degani, `-f` — "follow" (jonli kuzatish, `tail -f` kabi). `journalctl -u <service> -f` — bu sizning eng ko'p ishlatadigan buyrug'ingiz bo'ladi. Xizmat qayta ishga tushmayaptimi? Shu buyruqni ishga tushiring va nega yiqilayotganini o'z ko'zingiz bilan ko'ring.

**⚠️ Ehtiyot bo'l:** `journalctl` loglari ham disk egallaydi. Vaqt o'tishi bilan gigabaytlab bo'lib ketishi mumkin. Cheklovni sozlang:

```bash
# Loglarni 500MB bilan cheklash
sudo journalctl --vacuum-size=500M

# Yoki 2 haftadan eski loglarni o'chirish
sudo journalctl --vacuum-time=2weeks
```

Doimiy cheklov uchun `/etc/systemd/journald.conf` faylida `SystemMaxUse=500M` ni sozlang.

### Nginx loglari

Nginx ikkita asosiy log fayl yozadi:

- **`/var/log/nginx/access.log`** — har bir kelgan so'rov (kim, qachon, qaysi URL, qanday status kod).
- **`/var/log/nginx/error.log`** — Nginx darajasidagi xatolar (config xatosi, upstream yiqilishi, 502/504).

```bash
# Kelgan so'rovlarni jonli kuzatish
sudo tail -f /var/log/nginx/access.log

# Xatolarni jonli kuzatish (sayt ishlamayotganda BIRINCHI shu yerga qarang)
sudo tail -f /var/log/nginx/error.log

# Oxirgi 50 qator xato
sudo tail -n 50 /var/log/nginx/error.log

# access.log'dan 500 statuslarni ajratib olish
sudo grep " 500 " /var/log/nginx/access.log

# Eng ko'p so'rov yuborgan IP'larni topish (top 10)
sudo awk '{print $1}' /var/log/nginx/access.log | sort | uniq -c | sort -rn | head
```

**💡 Tushuncha:** `access.log` format odatda bunday ko'rinadi:

```
203.0.113.45 - - [30/Jun/2026:10:15:32 +0500] "GET /api/users HTTP/1.1" 200 1543 "-" "Mozilla/5.0..."
```

Bu yerda: IP manzil, vaqt, so'rov turi va yo'li, **status kod (200)**, javob hajmi (1543 bayt), va User-Agent. Status kodga qarab tez baho berasiz: 2xx — yaxshi, 4xx — client xatosi, 5xx — server xatosi.

**⚠️ Ehtiyot bo'l:** Agar `access.log`da bitta IP dan minglab so'rov ko'rsangiz yoki g'alati URL'lar (`/wp-admin`, `/.env`, `/phpmyadmin`) bo'lsa — bu bot yoki hujum. Bu IP'ni `fail2ban` yoki `ufw` bilan bloklang.

### PM2 loglari va log rotation

Agar Node.js ilovangizni **PM2** bilan ishlatayotgan bo'lsangiz, PM2 stdout va stderr'ni avtomatik yozib boradi:

```bash
# Barcha ilovalarning jonli loglari
pm2 logs

# Faqat bitta ilova logi (nom yoki id bo'yicha)
pm2 logs my-app

# Oxirgi 200 qator
pm2 logs my-app --lines 200

# Faqat xatolar oqimi
pm2 logs my-app --err

# Log fayllar qayerda yotibdi (pathlarni ko'rsatadi)
pm2 info my-app
```

PM2 loglari odatda `~/.pm2/logs/` ichida saqlanadi (`my-app-out.log` va `my-app-error.log`).

**⚠️ Ehtiyot bo'l:** PM2 loglari o'z-o'zidan rotate bo'lmaydi — vaqt o'tishi bilan gigabaytlarga o'sib, diskni to'ldiradi. Buni oldini olish uchun **`pm2-logrotate`** modulini o'rnating:

```bash
# pm2-logrotate modulini o'rnatish
pm2 install pm2-logrotate

# Har bir log fayl maksimal 10MB bo'lsin
pm2 set pm2-logrotate:max_size 10M

# Faqat oxirgi 30 ta fayl saqlansin (qolganlari o'chib boradi)
pm2 set pm2-logrotate:retain 30

# Har kuni yarim tunda aylantirish (rotate)
pm2 set pm2-logrotate:rotateInterval '0 0 * * *'

# Eski loglarni siqish (gzip)
pm2 set pm2-logrotate:compress true
```

**💡 Tushuncha:** "Log rotation" — bu logni cheksiz bitta faylga yozmasdan, ma'lum hajm/vaqtda yangi faylga o'tish va eskilarini o'chirish/siqish. Bu diskni himoya qiladi.

### Application logging best practice

**💡 Tushuncha:** `console.log("bu yerga yetdi")` — debug uchun yaxshi, lekin production uchun yaramaydi. Production'da sizga **strukturaviy (structured) loglar** kerak: har bir log yozuvi mashina o'qiy oladigan formatda (odatda JSON), log level bilan.

Log level'lar (muhimlik darajasi):

- **`error`** — dastur ishlamay qoldi, darhol e'tibor kerak.
- **`warn`** — muammo bor, lekin ishlashda davom etyapti.
- **`info`** — muhim hodisa (foydalanuvchi kirdi, buyurtma yaratildi).
- **`debug`** — batafsil ma'lumot, faqat rivojlantirishda.

Node.js uchun `pino` yoki `winston` kutubxonalari eng mashhur. Pino misoli:

```javascript
// logger.js
const pino = require('pino');

const logger = pino({
  level: process.env.LOG_LEVEL || 'info',
});

module.exports = logger;
```

```javascript
// Foydalanish
const logger = require('./logger');

logger.info({ userId: 42, action: 'login' }, 'User logged in');
logger.error({ err, orderId: 100 }, 'Failed to process order');
```

Bu quyidagicha strukturaviy JSON log beradi:

```json
{"level":30,"time":1719736800000,"userId":42,"action":"login","msg":"User logged in"}
```

**💡 Tushuncha:** Structured log'ning kuchi — uni keyinchalik qidirish va filtrlash oson. `msg` bo'yicha grep qilasiz, `userId` bo'yicha filtrlaysiz. Oddiy matn logda bu qiyin.

**⚠️ Ehtiyot bo'l:** Logga **hech qachon** parol, token, credit card, yoki boshqa maxfiy ma'lumotni yozmang. `logger.info(req.body)` deb butun so'rov tanasini loglash — parolni logga chiqarib yuborishning eng keng tarqalgan usuli.

### Disk to'lib ketmasligi: logrotate

**💡 Tushuncha:** Ubuntu'da `logrotate` allaqachan o'rnatilgan va `/var/log/` ichidagi ko'p loglarni (Nginx ham) avtomatik aylantiradi. U har kuni `cron` orqali ishga tushadi.

Nginx uchun sozlama fayli allaqachon mavjud:

```bash
cat /etc/logrotate.d/nginx
```

O'zingizning application logingiz uchun (masalan `/var/www/myapp/logs/`) yangi qoida yozishingiz mumkin:

```bash
sudo nano /etc/logrotate.d/myapp
```

```
/var/www/myapp/logs/*.log {
    daily
    rotate 14
    compress
    delaycompress
    missingok
    notifempty
    create 0640 www-data www-data
}
```

Bu: har kuni aylantirish, 14 kunlik tarix saqlash, eskilarini gzip qilish. Sinab ko'rish:

```bash
# Config'ni tekshirish (aslida bajarmasdan)
sudo logrotate -d /etc/logrotate.d/myapp

# Majburan ishga tushirish
sudo logrotate -f /etc/logrotate.d/myapp
```

**⚠️ Ehtiyot bo'l:** Disk to'lib ketishi — production'ni yiqitadigan eng keng tarqalgan sabablardan biri. Muntazam `df -h` bilan tekshiring. Disk 100% bo'lsa, dastur ham, DB ham yozolmay qoladi va butun sayt yiqiladi.

---

## Monitoring

### Serverni real-time kuzatish

Serverning hozirgi holatini ko'rish uchun asosiy buyruqlar:

```bash
# Interaktiv process/CPU/RAM monitori (htop yoqimliroq, top ham bor)
htop

# Disk band bo'lishi (-h = odam o'qiy oladigan format)
df -h

# RAM holati (megabaytlarda)
free -m

# Server qancha vaqtdan beri ishlayapti va o'rtacha yuklama (load average)
uptime

# Eng ko'p RAM yeyayotgan 10 ta process
ps aux --sort=-%mem | head

# Eng ko'p CPU yeyayotgan 10 ta process
ps aux --sort=-%cpu | head

# Qaysi papka ko'p joy egallayapti (joriy papkada)
du -sh * | sort -rh | head
```

**💡 Tushuncha:** `uptime` chiqishidagi "load average: 0.50, 0.75, 0.60" — bu 1, 5 va 15 daqiqalik o'rtacha yuklama. **Muhim qoida:** agar bu son sizning CPU yadrolaringiz sonidan (`nproc` bilan bilasiz) doimiy yuqori bo'lsa — server bo'g'ilib qolgan (overloaded).

`htop` bo'lmasa o'rnating: `sudo apt install htop`.

**⚠️ Ehtiyot bo'l:** Bu buyruqlar faqat **hozirgi** holatni ko'rsatadi. Siz uxlab yotganingizda server yiqilsa, buni ko'rmaysiz. Shuning uchun avtomatik, doimiy monitoring va **alert** kerak.

### Uptime monitoring (UptimeRobot)

**💡 Tushuncha:** Uptime monitoring — bu tashqi bir xizmat sizning saytingizni har bir necha daqiqada "chaqirib", javob berayotganini tekshirib turadi. Sayt yiqilsa — sizga darhol xabar (email, Telegram, SMS) yuboradi. Bu eng oddiy va eng muhim monitoring turi.

**UptimeRobot** (bepul tarifi yetarli) sozlash qadamlar:

1. [uptimerobot.com](https://uptimerobot.com) da ro'yxatdan o'ting.
2. **Add New Monitor** → Monitor Type: **HTTP(s)**.
3. URL: `https://sizning-saytingiz.uz` (yoki health check endpoint: `https://sizning-saytingiz.uz/health`).
4. Monitoring Interval: 5 daqiqa (bepul tarifda).
5. **Alert Contacts:** email qo'shing, yoki Telegram integratsiyasini ulang (eng qulay).

**💡 Tushuncha:** Health check endpoint'ga (`/health`) ulash bosh sahifaga ulashdan yaxshiroq — chunki `/health` DB ulanishini ham tekshirishi mumkin. Pastda "Health check" bo'limiga qarang.

### Resource monitoring (Netdata / Prometheus + Grafana)

Uptime monitoring "sayt tirikmi?" degan savolga javob beradi. Resource monitoring esa "server o'zini qanday his qilyapti?" (CPU, RAM, disk, tarmoq trendlari) degan savolga tarix bilan javob beradi.

**Netdata** — eng oson variant, bir buyruqda o'rnatiladi va chiroyli real-time dashboard beradi:

```bash
# Netdata'ni o'rnatish (rasmiy kickstart skript)
wget -O /tmp/netdata-kickstart.sh https://get.netdata.cloud/kickstart.sh
sudo sh /tmp/netdata-kickstart.sh
```

O'rnatilgach `http://server-ip:19999` orqali dashboard ochiladi.

**⚠️ Ehtiyot bo'l:** Netdata porti (19999) ni internetga ochiq qoldirmang! `ufw` bilan faqat o'zingizning IP'ingizga ruxsat bering, yoki SSH tunnel orqali kiring:

```bash
# O'z kompyuteringizdan SSH tunnel (localhost:19999 -> server)
ssh -L 19999:localhost:19999 user@server-ip
```

**Prometheus + Grafana** — bir nechta serverni, murakkab metrikalarni va professional alerting'ni boshqarish uchun sanoat standarti. Ammo o'rnatish va sozlash ancha mehnat talab qiladi. Bitta kichik VPS uchun bu ortiqcha — UptimeRobot + Netdata yetarli. Loyihangiz o'sib, bir nechta serverga chiqqanda Prometheus + Grafana'ga o'ting.

### Xatolarni kuzatish (Sentry)

**💡 Tushuncha:** Loglarni har kuni qo'lda o'qib o'tirmaysiz. **Sentry** — dasturingizdagi xatolar (exception, crash) sodir bo'lganda ularni avtomatik yig'ib, guruhlab, sizga xabar beradigan xizmat. Xato qaysi qatorda, qanday stack trace bilan, qaysi foydalanuvchida bo'lganini ko'rsatadi.

Node.js uchun ulash juda oson:

```bash
npm install @sentry/node
```

```javascript
// Ilovaning eng boshida (boshqa importlardan oldin)
const Sentry = require('@sentry/node');

Sentry.init({
  dsn: process.env.SENTRY_DSN, // Sentry loyihangizdan olasiz
  environment: process.env.NODE_ENV || 'production',
  tracesSampleRate: 0.1,
});
```

Endi qayta ushlanmagan har qanday xato avtomatik Sentry'ga boradi va sizga email/Telegram xabar keladi. Bepul tarifi kichik loyiha uchun yetarli.

**💡 Tushuncha:** DSN — bu maxfiy kalit emas, lekin uni ham `.env` da saqlash yaxshi amaliyot. Front-end va back-end uchun alohida loyiha (project) oching.

### Health check

**💡 Tushuncha:** Health check — bu dasturingiz sog'lig'ini tekshiradigan maxsus endpoint (`/health` yoki `/healthz`). Uni monitoring xizmatlari (UptimeRobot), load balancer'lar va deploy skriptlar chaqiradi.

Oddiy health check (Express misoli):

```javascript
app.get('/health', (req, res) => {
  res.status(200).json({ status: 'ok', uptime: process.uptime() });
});
```

DB ulanishini ham tekshiradigan chuqurroq versiya:

```javascript
app.get('/health', async (req, res) => {
  try {
    await db.query('SELECT 1'); // DB tirikligini tekshirish
    res.status(200).json({ status: 'ok' });
  } catch (err) {
    res.status(503).json({ status: 'error', message: 'DB unreachable' });
  }
});
```

**💡 Tushuncha:** Yaxshi health check faqat "server javob beryapti" demasdan, **muhim bog'liqliklar** (DB, cache, tashqi API) ishlayotganini ham tekshiradi. Shunda DB yiqilsa, health check ham 503 qaytaradi va monitoring darhol xabar beradi.

---

## Security (server xavfsizligi)

**💡 Tushuncha:** Internetga ochiq har qanday server bir necha daqiqa ichida avtomatik botlar tomonidan skaner qilina boshlaydi. Ular SSH parolini topishga, ochiq portlarni qidirishga, zaif joylarni izlashga urinadi. "Meni kim ham buzardi" deb o'ylamang — hujum shaxsiy emas, avtomatik va doimiy. Quyidagilar — minimal himoya.

### SSH hardening

SSH — serverga kirish eshigi. Uni mustahkamlash birinchi vazifa.

**1-qadam: SSH key yaratish** (agar hali bo'lmasa, o'z kompyuteringizda):

```bash
# O'z LOKAL kompyuteringizda (serverda emas!)
ssh-keygen -t ed25519 -C "ramziddin@example.com"

# Public kalitni serverga ko'chirish
ssh-copy-id user@server-ip
```

**2-qadam: SSH konfiguratsiyasini mustahkamlash:**

```bash
sudo nano /etc/ssh/sshd_config
```

Quyidagilarni sozlang:

```
# Parol bilan kirishni butunlay o'chirish (faqat key)
PasswordAuthentication no

# root sifatida to'g'ridan-to'g'ri kirishni taqiqlash
PermitRootLogin no

# Standart 22 portni o'zgartirish (avtomatik skanerlarni chalg'itadi)
Port 2222

# Bo'sh parolni taqiqlash
PermitEmptyPasswords no
```

O'zgarishni qo'llash:

```bash
sudo systemctl restart ssh
```

**⚠️ Ehtiyot bo'l:** `PasswordAuthentication no` qilishdan **oldin** SSH key bilan kira olishingizni tekshiring! Aks holda o'zingizni serverdan qulflab qo'yasiz. Yangi terminalda test qiling, joriy sessiyani yopmang. Port o'zgartirsangiz, `ufw`da yangi portni ochishni unutmang.

**💡 Tushuncha:** Port o'zgartirish "haqiqiy" xavfsizlik emas (security through obscurity), lekin avtomatik botlar 99% ni 22-portga urinadi, shuning uchun log shovqinini keskin kamaytiradi. `fail2ban` bilan birga ishlatilganda foydali.

### Firewall: ufw

**💡 Tushuncha:** Firewall — qaysi portlarga tashqaridan ulanishga ruxsat berilishini boshqaradi. Qoida oddiy: **hamma narsani yoping, faqat kerakligini oching.** Ubuntu'da `ufw` (Uncomplicated Firewall) eng oson vosita.

```bash
# Standart qoida: kiruvchi hammasini bloklash, chiquvchiga ruxsat
sudo ufw default deny incoming
sudo ufw default allow outgoing

# SSH'ni ochish (portingizga qarab! standart 22 yoki o'zgartirilgan 2222)
sudo ufw allow 2222/tcp

# HTTP va HTTPS
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

# Firewall'ni yoqish
sudo ufw enable

# Holatni ko'rish
sudo ufw status verbose
```

**⚠️ Ehtiyot bo'l:** `ufw enable` qilishdan oldin SSH portini **albatta** oching! Bo'lmasa ulanish uzilib, serverga kira olmay qolasiz (agar bu VPS bo'lsa, provayder konsoli orqali tiklashga to'g'ri keladi).

**💡 Tushuncha:** Ma'lum bir portni faqat bitta IP'ga ochish mumkin (masalan Netdata yoki DB uchun):

```bash
# 5432 (PostgreSQL) portini faqat sizning IP'ingizga ochish
sudo ufw allow from 203.0.113.10 to any port 5432
```

### fail2ban: brute-force himoyasi

**💡 Tushuncha:** `fail2ban` loglarni kuzatib turadi va agar bir IP takror-takror muvaffaqiyatsiz kirishga urinsa (masalan SSH parolni 5 marta xato terse), o'sha IP'ni firewall darajasida avtomatik bloklaydi. Bu brute-force hujumlariga qarshi eng samarali va oddiy himoya.

**O'rnatish:**

```bash
sudo apt update
sudo apt install fail2ban -y
```

**Sozlash:** asl faylni tahrirlamang, `jail.local` yarating (yangilanishlar uni o'chirib yubormaydi):

```bash
sudo nano /etc/fail2ban/jail.local
```

```ini
[DEFAULT]
# Blok muddati: 1 soat
bantime = 1h
# Necha daqiqa ichida urinishlar hisoblanadi
findtime = 10m
# Necha marta xato urinishdan keyin bloklash
maxretry = 5
# O'z IP'ingizni hech qachon bloklamaslik (o'z IP'ingizni yozing)
ignoreip = 127.0.0.1/8 203.0.113.10

[sshd]
enabled = true
port = 2222
```

**Ishga tushirish va tekshirish:**

```bash
# Xizmatni yoqish va ishga tushirish
sudo systemctl enable fail2ban
sudo systemctl restart fail2ban

# Umumiy holat
sudo fail2ban-client status

# SSH "jail" holati (nechta IP bloklangan)
sudo fail2ban-client status sshd

# Bir IP'ni qo'lda blokdan chiqarish
sudo fail2ban-client set sshd unbanip 203.0.113.45
```

**💡 Tushuncha:** `fail2ban` faqat SSH uchun emas — Nginx uchun ham "jail" sozlash mumkin (masalan 404/403 ni ko'p urayotgan botlarni bloklash). `nginx-http-auth`, `nginx-limit-req` kabi tayyor filtrlar bor.

### Tizimni yangilab turish

**💡 Tushuncha:** Eskirgan dasturlarda ma'lum (ochiq e'lon qilingan) xavfsizlik teshiklari bo'ladi, va botlar aynan shularni qidiradi. Xavfsizlik yangilanishlarini o'z vaqtida qo'llash — eng arzon va eng ta'sirli himoya.

Qo'lda yangilash:

```bash
sudo apt update && sudo apt upgrade -y
```

**Avtomatik xavfsizlik yangilanishlari** (`unattended-upgrades`) — buni albatta yoqing:

```bash
# O'rnatish
sudo apt install unattended-upgrades -y

# Yoqish (interaktiv sozlama ochiladi, "Yes" tanlang)
sudo dpkg-reconfigure -plow unattended-upgrades
```

Sozlamalarni tekshirish/tahrirlash:

```bash
sudo nano /etc/apt/apt.conf.d/50unattended-upgrades
```

**⚠️ Ehtiyot bo'l:** Ba'zi yangilanishlar server qayta yuklashni (reboot) talab qiladi. `unattended-upgrades` avtomatik reboot qilishini xohlasangiz, sozlamada `Automatic-Reboot "true"` va vaqtini (masalan tunda `03:00`) belgilang — lekin buni ehtiyotkorlik bilan qiling, kutilmagan reboot production'ni uzishi mumkin.

### Secret'larni to'g'ri saqlash

**💡 Tushuncha:** Secret'lar — bu parollar, API kalitlar, DB parollari, JWT secret'lar, token'lar. Ularni kodning ichiga yozish yoki `git`ga qo'yish — eng katta va eng keng tarqalgan xavfsizlik xatosi.

**Amaliy qoidalar:**

1. **Barcha secret'lar `.env` faylida bo'lsin**, kod ichida emas:

```bash
# .env fayli (serverda, git'da EMAS)
DB_PASSWORD=super-secret-parol
JWT_SECRET=uzoq-tasodifiy-string
SENTRY_DSN=https://...
```

2. **`.env` ni `.gitignore` ga qo'shing** — hech qachon commit qilmang:

```bash
echo ".env" >> .gitignore
```

3. **Fayl ruxsatlarini cheklang** — faqat egasi o'qiy olsin:

```bash
chmod 600 .env
```

4. Repo'da `.env.example` (bo'sh qiymatlar bilan) saqlang, real qiymatlarni emas.

**⚠️ Ehtiyot bo'l:** Agar secret'ni bir marta git'ga commit qilib yuborgan bo'lsangiz, uni oddiygina o'chirish YETARLI EMAS — u git tarixida qoladi. U secret'ni **buzilgan** deb hisoblang va **darhol yangisiga almashtiring** (rotate). GitHub public repo bo'lsa, botlar bir necha daqiqada topib ishlatib ketadi.

### Nginx rate limiting va security header'lar

**💡 Tushuncha:** Rate limiting — bir foydalanuvchi/IP dan keladigan so'rovlar sonini cheklaydi. Bu brute-force, spam va DoS hujumlaridan himoya qiladi (masalan login sahifasiga soniyasiga 100 marta urishni to'xtatadi).

Nginx'da rate limiting (`/etc/nginx/nginx.conf` ning `http` blokida zona e'lon qilinadi):

```nginx
# http { } bloki ichida
limit_req_zone $binary_remote_addr zone=mylimit:10m rate=10r/s;
```

```nginx
# server { } yoki location bloki ichida qo'llash
location /api/ {
    limit_req zone=mylimit burst=20 nodelay;
    proxy_pass http://localhost:3000;
}

# Login endpoint uchun qattiqroq cheklov
location /api/login {
    limit_req zone=mylimit burst=5;
    proxy_pass http://localhost:3000;
}
```

**Security header'lar** — brauzerga qo'shimcha himoya qoidalarini bildiradi. `server` blokiga qo'shing:

```nginx
# Sahifani iframe ichida ochishni taqiqlash (clickjacking himoyasi)
add_header X-Frame-Options "SAMEORIGIN" always;

# Brauzerni MIME-type taxmin qilishdan to'xtatish
add_header X-Content-Type-Options "nosniff" always;

# HTTPS'ni majburiy qilish (faqat HTTPS ishlaganda yoqing!)
add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;

# Referrer siyosati
add_header Referrer-Policy "strict-origin-when-cross-origin" always;
```

Har o'zgarishdan keyin config'ni tekshirib qayta yuklang:

```bash
# Config sintaksisini tekshirish
sudo nginx -t

# Xatosiz bo'lsa qayta yuklash
sudo systemctl reload nginx
```

**💡 Tushuncha:** HTTPS/SSL — bugun majburiy. Let's Encrypt bilan bepul sertifikat oling (Certbot: `sudo certbot --nginx`). HTTP (80) so'rovlarni HTTPS (443) ga redirect qiling. `Strict-Transport-Security` header'ini faqat HTTPS to'liq ishlaganda yoqing.

### Backup strategiyasi (3-2-1)

**💡 Tushuncha:** Backup bo'lmasa, bir kuni yo'qotasiz — disk buziladi, DB o'chib ketadi, yoki ransomware urib ketadi. Sanoat standarti — **3-2-1 qoidasi:**

- **3** nusxa ma'lumot (asl + 2 backup),
- **2** xil turdagi joyda (masalan serverning diski + tashqi bulut),
- **1** nusxa boshqa fizik joyda (offsite — boshqa serverda yoki S3/bulutda).

Amaliy misol: PostgreSQL DB'ni har kuni backup qilib, S3/bulutga yuborish:

```bash
#!/bin/bash
# /usr/local/bin/db-backup.sh
set -e

DATE=$(date +%Y-%m-%d_%H-%M)
BACKUP_DIR="/var/backups/db"
mkdir -p "$BACKUP_DIR"

# DB'ni dump qilish va siqish
pg_dump -U myuser mydb | gzip > "$BACKUP_DIR/mydb_$DATE.sql.gz"

# 7 kundan eski local backuplarni o'chirish
find "$BACKUP_DIR" -name "*.sql.gz" -mtime +7 -delete

# Bulutga (offsite) yuklash — masalan S3
# aws s3 cp "$BACKUP_DIR/mydb_$DATE.sql.gz" s3://my-backups/db/
```

`cron` orqali har kuni ishga tushirish:

```bash
# crontab tahrirlash
crontab -e
```

```
# Har kuni ertalab 03:00 da DB backup
0 3 * * * /usr/local/bin/db-backup.sh >> /var/log/db-backup.log 2>&1
```

**⚠️ Ehtiyot bo'l:** Backup'ni **hech qachon** faqat o'sha serverning o'zida saqlamang — server yiqilsa, backup ham yo'qoladi. Va eng muhimi: **backup'ni tiklab (restore) sinab ko'ring!** Tiklanmaydigan backup — backup emas. Chorakda bir marta test restore qiling.

### Buzilgan serverni aniqlash belgilari

**💡 Tushuncha:** Server buzilgan (compromised) bo'lishi mumkinligining belgilarini bilib qo'ying. Quyidagilardan biri ko'rinsa — darhol tekshiring:

- **G'ayritabiiy yuqori CPU/tarmoq** — server kimningdir cryptocurrency mining yoki DDoS uchun ishlatilayotgan bo'lishi mumkin (`htop`, `iftop`).
- **Notanish process'lar** — `ps aux` da g'alati nomli, tushunarsiz process'lar.
- **Notanish cron job'lar** — `crontab -l` va `/etc/cron.*` da siz qo'ymagan vazifalar.
- **`auth.log`da muvaffaqiyatli kirishlar** — notanish IP yoki vaqtda:

```bash
# Muvaffaqiyatli SSH kirishlarni ko'rish
sudo grep "Accepted" /var/log/auth.log

# Muvaffaqiyatsiz urinishlar (brute-force belgisi)
sudo grep "Failed password" /var/log/auth.log | tail -50

# Hozir kim tizimga kirgan
who

# Oxirgi kirishlar tarixi
last -20
```

- **O'zgargan tizim fayllari** yoki notanish yangi foydalanuvchilar (`cat /etc/passwd`).
- **Chiquvchi (outbound) g'alati trafik** — serveringiz o'zingiz bilmagan joylarga ulanishga urinyapti.

**⚠️ Ehtiyot bo'l:** Agar server buzilganiga jiddiy shubha bo'lsa, uni "tozalashga" urinmang — buzg'unchi backdoor qoldirgan bo'lishi mumkin va uni to'liq topa olmaysiz. Eng ishonchli yo'l: muhim ma'lumotni (backup'dan) ko'chirib olib, **serverni noldan qayta o'rnatish** va barcha parol/kalitlarni almashtirish.

---

## Amaliy Q&A

### ❓ Sayt "ishlamayapti" degan xabar keldi. Birinchi nima qilaman?

**✅ Javob:** Ketma-ket tekshiring:

```bash
# 1. Ilova process tirikmi?
pm2 status

# 2. Nginx tirikmi?
sudo systemctl status nginx

# 3. Nginx error logda nima bor?
sudo tail -n 50 /var/log/nginx/error.log

# 4. Ilova logida nima bor?
pm2 logs my-app --lines 50

# 5. Disk to'lib qolmaganmi?
df -h
```

90% hollarda sabab shu 5 buyruqda ko'rinadi: process yiqilgan, disk to'lgan, yoki config xatosi.

### ❓ Disk 100% to'ldi. Nima qilaman?

**✅ Javob:** Avval nima joy egallaganini toping:

```bash
# Eng katta papkalarni topish (root'dan boshlab)
sudo du -sh /* 2>/dev/null | sort -rh | head

# Ko'pincha aybdor — loglar
sudo du -sh /var/log/* | sort -rh | head
```

Odatiy sabablar va yechim: journal loglarni tozalash (`journalctl --vacuum-size=200M`), eski PM2 loglarni o'chirish (va `pm2-logrotate` o'rnatish), eski backup fayllar, Docker image'lari (`docker system prune`).

### ❓ `pm2 logs` juda ko'p eski ma'lumot ko'rsatyapti va disk to'lyapti. Nima qilaman?

**✅ Javob:** `pm2-logrotate` o'rnating (yuqoridagi bo'limga qarang) va joriy loglarni tozalang:

```bash
pm2 install pm2-logrotate
pm2 set pm2-logrotate:max_size 10M
pm2 flush   # barcha joriy loglarni tozalash
```

### ❓ SSH'ga kira olmay qoldim (parol o'chirilganidan keyin). Nima qilaman?

**✅ Javob:** Bu SSH key noto'g'ri sozlangani belgisi. Yechim:

1. VPS provayderining **web console** (VNC/serial console) orqali kiring — bu SSH'ni chetlab o'tadi.
2. Kirgach `/etc/ssh/sshd_config` da vaqtincha `PasswordAuthentication yes` qiling, `sudo systemctl restart ssh`.
3. SSH key'ni to'g'ri qo'shing (`~/.ssh/authorized_keys` faylini tekshiring, ruxsatlar `chmod 600`).
4. Key ishlaganini tasdiqlab, keyin parolni yana o'chiring.

**⚠️ Ehtiyot bo'l:** Shuning uchun ham hech qachon joriy SSH sessiyani yopmasdan, yangi terminalda test qilish kerak.

### ❓ Serverimga kimdir hujum qilyaptimi, qanday bilaman?

**✅ Javob:** SSH urinishlarini va Nginx trafigini tekshiring:

```bash
# Muvaffaqiyatsiz SSH urinishlar (ko'p bo'lsa — brute-force)
sudo grep "Failed password" /var/log/auth.log | wc -l

# fail2ban nechta IP'ni bloklagan
sudo fail2ban-client status sshd

# Nginx'da eng ko'p so'rov yuborgan IP'lar
sudo awk '{print $1}' /var/log/nginx/access.log | sort | uniq -c | sort -rn | head
```

Kunlik minglab "Failed password" — normal (botlar doim urinadi), agar `fail2ban` ishlab tursa. Xavotir — muvaffaqiyatli notanish kirish bo'lsa.

### ❓ `.env` faylimni git'ga qo'yib yuboribman. Nima qilaman?

**✅ Javob:** Faylni o'chirish yetarli emas — u tarixda qoladi. To'g'ri qadamlar:

1. **Darhol barcha secret'larni almashtiring (rotate):** DB parolini, API kalitlarini, JWT secret'ni yangilang. Eski qiymatlarni buzilgan deb hisoblang.
2. `.env` ni `.gitignore` ga qo'shing.
3. Tarixdan tozalash uchun `git filter-repo` yoki BFG Repo-Cleaner ishlating (ammo bu ikkilamchi — asosiysi rotate).

### ❓ Qaysi portlar ochiqligini qanday tekshiraman?

**✅ Javob:**

```bash
# ufw qoidalari
sudo ufw status verbose

# Serverda haqiqatan tinglayotgan portlar
sudo ss -tulpn
```

`ss -tulpn` chiqishida `0.0.0.0:5432` ni ko'rsangiz — DB butun internetga ochiq, bu xavfli. Uni faqat `localhost` (`127.0.0.1`) da tinglashga sozlang.

### ❓ Ma'lumotlar bazamni tashqaridan qanday himoyalayman?

**✅ Javob:** Ikki qavat:

1. **DB'ni faqat localhost'da tinglashga majburlang.** PostgreSQL'da `postgresql.conf`:
```
listen_addresses = 'localhost'
```
2. **Firewall bilan DB portini yoping** (yoki faqat kerakli IP'ga oching):
```bash
sudo ufw deny 5432/tcp
```

Ilovangiz o'sha serverda bo'lsa, DB'ni umuman tashqariga ochish shart emas — `localhost` orqali ulanadi.

### ❓ Health check endpoint qaytmaganda UptimeRobot bir necha marta yolg'on alert beryapti. Nega?

**✅ Javob:** Odatda sabab — health check juda "og'ir" (masalan har chaqiruvda katta DB so'rovi bajaradi) yoki timeout past. Yechim: health check'ni yengil qiling (`SELECT 1` yetarli), UptimeRobot'da monitor interval va "confirmation" sozlamasini oshiring (bitta xatoda emas, 2-3 ketma-ket xatoda alert bersin).

### ❓ Loglarni bir necha serverdan bir joyda ko'rmoqchiman. Qanday?

**✅ Javob:** Bir necha server bo'lganda markazlashgan logging kerak. Commercial: **Better Stack (Logtail)**, **Datadog**, **Grafana Loki**. Kichik loyiha uchun Loki + Grafana (o'z serveringizda) yaxshi va arzon. Bitta server uchun esa bu ortiqcha — `journalctl` va PM2 yetarli.

### ❓ `unattended-upgrades` yoqilganini qanday tekshiraman?

**✅ Javob:**

```bash
# Xizmat holati
systemctl status unattended-upgrades

# Nima yangilanganini ko'rish (log)
cat /var/log/unattended-upgrades/unattended-upgrades.log

# Quruq ishga tushirish (nima qilardi, ko'rsatadi)
sudo unattended-upgrades --dry-run --debug
```

### ❓ Server load average juda yuqori. Nima qilaman?

**✅ Javob:** Avval yadrolar sonini bilib oling (`nproc`), keyin load average bilan solishtiring:

```bash
uptime          # load average'ni ko'rsatadi
nproc           # CPU yadrolari soni
htop            # qaysi process yuklayotganini ko'rish
ps aux --sort=-%cpu | head   # eng ko'p CPU yeyayotganlar
```

Load average yadro sonidan doimiy yuqori bo'lsa — yo kod optimizatsiya kerak, yo serverni kattalashtirish (upgrade), yoki kimdir noqonuniy foydalanayapti (buzilganlik belgisi).

### ❓ SSL sertifikatim muddati tugadi va sayt "not secure" ko'rsatyapti. Nima qilaman?

**✅ Javob:** Let's Encrypt sertifikatlari 90 kunlik. Certbot odatda avtomatik yangilaydi — buni tekshiring:

```bash
# Sertifikatlarni qo'lda yangilash
sudo certbot renew

# Avtomatik yangilanish timer'i ishlayaptimi
sudo systemctl status certbot.timer

# Yangilashni sinash (aslida yangilamasdan)
sudo certbot renew --dry-run
```

Kelajakda takrorlanmasligi uchun `certbot.timer` yoqilganiga ishonch hosil qiling — u kuniga ikki marta tekshiradi va kerak bo'lsa avtomatik yangilaydi.

---

## Amaliy Checklist

**Loglar:**

- [ ] `journalctl -u <service> -f` bilan xizmat loglarini o'qiy olaman.
- [ ] Nginx `access.log` va `error.log` ni `tail -f` bilan kuzataman.
- [ ] `pm2-logrotate` o'rnatilgan, PM2 loglari cheklangan.
- [ ] Application'da structured logging (pino/winston) va to'g'ri log level'lar.
- [ ] Logga hech qachon parol/token yozilmaydi.
- [ ] `logrotate` sozlangan, loglar diskni to'ldirmaydi.

**Monitoring:**

- [ ] `htop`, `df -h`, `free -m`, `uptime` bilan holatni tez tekshira olaman.
- [ ] UptimeRobot (yoki shunga o'xshash) alert bilan sozlangan.
- [ ] `/health` endpoint bor (DB'ni ham tekshiradi).
- [ ] Resource monitoring (Netdata) o'rnatilgan, porti himoyalangan.
- [ ] Sentry (yoki shunga o'xshash) xato kuzatuvi ulangan.

**Security:**

- [ ] SSH: key-only, `PermitRootLogin no`, port o'zgartirilgan.
- [ ] `ufw` yoqilgan, faqat kerakli portlar ochiq (SSH, 80, 443).
- [ ] `fail2ban` o'rnatilgan va ishlayapti.
- [ ] `unattended-upgrades` yoqilgan.
- [ ] Ilova non-root user ostida ishlaydi (least privilege).
- [ ] Barcha secret'lar `.env` da, git'ga qo'yilmagan, `chmod 600`.
- [ ] HTTPS/SSL majburiy, HTTP → HTTPS redirect.
- [ ] DB faqat `localhost` da tinglaydi, tashqariga ochiq emas.
- [ ] Nginx rate limiting va security header'lar sozlangan.
- [ ] Backup 3-2-1 qoidasi bo'yicha, offsite, test restore qilingan.
- [ ] Buzilganlik belgilarini (`auth.log`, notanish process/cron) tekshira olaman.

---

← [Deployment bo'limiga qaytish](./README.md)
