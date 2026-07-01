# PHP Deploy

PHP hali ham dunyodagi eng ko'p tarqalgan backend tillaridan biri — WordPress, Laravel, Symfony, va son-sanoqsiz sayt PHP'da yozilgan. Bu qo'llanmada PHP loyihangizni qayerga va qanday deploy qilishni o'rganamiz: eng oddiy **shared hosting** (cPanel + FTP) dan tortib, professional **VPS + LEMP stack** va **Laravel** ilovasini production'ga to'liq chiqarishgacha. Har bir qadam Ubuntu 24.04 uchun real buyruqlar bilan.

## Mundarija

- [PHP'ni qayerga deploy qilamiz](#phpni-qayerga-deploy-qilamiz)
- [(a) Shared Hosting (cPanel + FTP)](#a-shared-hosting-cpanel--ftp)
- [(b) VPS'da PHP: LEMP Stack](#b-vpsda-php-lemp-stack)
- [PHP-FPM nima va Nginx bilan qanday ishlaydi](#php-fpm-nima-va-nginx-bilan-qanday-ishlaydi)
- [Oddiy PHP saytni Nginx bilan serve qilish](#oddiy-php-saytni-nginx-bilan-serve-qilish)
- [(c) Laravel Deploy](#c-laravel-deploy)
- [Domain va SSL (certbot)](#domain-va-ssl-certbot)
- [Keng uchraydigan xatolar](#keng-uchraydigan-xatolar)
- [Amaliy Q&A](#amaliy-qa)
- [Amaliy Checklist](#amaliy-checklist)

---

## PHP'ni qayerga deploy qilamiz

PHP loyihani deploy qilishning uchta asosiy yo'li bor. Qay birini tanlash loyihangiz turiga va byudjetingizga bog'liq:

| Yo'l | Kimga mos | Narx | Nazorat |
|------|-----------|------|---------|
| **Shared hosting (cPanel)** | Oddiy PHP saytlar, WordPress, kichik biznes | Arzon (~20-50 ming so'm/oy) | Kam |
| **VPS + LEMP** | Laravel, katta loyihalar, o'z ustuvorliklaringiz | O'rta (~50 ming so'm/oydan) | To'liq |
| **Managed PaaS** (Laravel Forge, Ploi) | Laravel + qulaylik | Qimmatroq | O'rta-yuqori |

**💡 Tushuncha:** "Deploy" — bu kodni o'z kompyuteringizdan (localhost) internetga chiqarilgan serverga ko'chirish va u yerda ishlatish. Localhost'da `php artisan serve` ishlashi — bu deploy emas, faqat siz ko'rasiz. Deploy — hamma ko'radigan holatga keltirish.

---

## (a) Shared Hosting (cPanel + FTP)

Shared hosting — bitta katta server bir nechta mijoz o'rtasida "bo'lib" ishlatiladigan eng arzon variant. Odatda **cPanel** boshqaruv paneli beriladi va fayllarni **FTP** yoki cPanel File Manager orqali yuklaysiz.

**Qachon mos:**
- Oddiy PHP sayt (kontakt forma, landing, kichik CMS)
- WordPress, Joomla, OpenCart kabi tayyor tizimlar
- Composer/queue/CLI kerak bo'lmaydigan loyihalar
- Server administrasiyasini bilmaysiz va o'rganishga vaqtingiz yo'q

**Qachon MOS EMAS:**
- Laravel queue worker, supervisor, cron kerak bo'lsa (ba'zi shared hosting'da cheklangan)
- SSH to'liq kirish, root huquqi kerak bo'lsa
- Katta trafik, o'z PHP versiyangizni boshqarish kerak bo'lsa

**⚠️ Ehtiyot bo'l:** O'zbek hosting provayderlari (masalan **ahost.uz**, **uzhost**, **hosting.uz**, **ums.uz**) shared hosting va VPS taklif qiladi. To'lov so'm'da, texnik yordam o'zbek/rus tilida — bu boshlovchi uchun qulay. Lekin arzon shared hosting'da ko'pincha PHP versiyasi eski (7.x) yoki cheklovlar bo'ladi. Sotib olishdan oldin **PHP 8.2+** borligini va **SSH** kirish bor-yo'qligini so'rang.

### FTP orqali fayl yuklash

Localhost'dagi tayyor PHP fayllarni serverga ko'chiramiz. FileZilla (bepul FTP client) ishlatamiz:

1. cPanel'da yangi FTP account yarating yoki asosiy loginni oling.
2. FileZilla'da ulanish ma'lumotlarini kiriting:

```
Host:     ftp.sizningdomen.uz
Username: sizning_ftp_useringiz
Password: ftp_parolingiz
Port:     21
```

3. Serverdagi `public_html/` papkasiga fayllaringizni sudrab tashlang. **Aynan `public_html` ichi** — bu web root, ya'ni brauzer ko'radigan joy.

```
public_html/
├── index.php
├── config.php
├── assets/
│   ├── style.css
│   └── app.js
└── includes/
```

**💡 Tushuncha:** `public_html` — Apache'ning "DocumentRoot" papkasi. Bu papkadagi `index.php` faylini brauzer `https://sizningdomen.uz/` ochilganda avtomatik ishga tushiradi.

### cPanel'da database yaratish

cPanel → **MySQL Databases** bo'limi:

1. **Create New Database**: masalan `mysite_db` (cPanel prefiks qo'shadi: `user_mysite_db`).
2. **Add New User**: user va kuchli parol.
3. **Add User To Database** → barcha privilegiyalarni bering.

PHP kodingizda ulanish:

```php
<?php
$host = 'localhost';        // shared hosting'da odatda localhost
$db   = 'user_mysite_db';   // to'liq nom (prefiks bilan)
$user = 'user_dbuser';
$pass = 'kuchli_parol';

$pdo = new PDO("mysql:host=$host;dbname=$db;charset=utf8mb4", $user, $pass);
$pdo->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
```

Shu bilan oddiy PHP sayt tayyor. Endi jiddiyroq loyihalar uchun VPS'ga o'tamiz.

---

## (b) VPS'da PHP: LEMP Stack

VPS (Virtual Private Server) — o'zingizga tegishli mustaqil Linux serveri. To'liq root kirish, xohlagan dasturni o'rnatasiz. PHP uchun standart to'plam — **LEMP stack**.

**💡 Tushuncha:** **LEMP** = **L**inux + **E**(N)ginx + **M**ySQL + **P**HP-FPM. "E" — Nginx'ning "Engine-X" talaffuzidan. LAMP'dan farqi: LAMP'da Apache, LEMP'da Nginx ishlatiladi.

### Apache vs Nginx + PHP-FPM

| Xususiyat | Apache (mod_php) | Nginx + PHP-FPM |
|-----------|------------------|-----------------|
| Statik fayl (css/js/rasm) | Sekinroq | Juda tez |
| Ko'p ulanish (concurrency) | Ko'p xotira sarflaydi | Kam xotira, samarali |
| `.htaccess` | Bor | Yo'q (config'da yoziladi) |
| PHP ishlashi | Har request'da process | Alohida FPM pool |

Zamonaviy production uchun **Nginx + PHP-FPM** tavsiya etiladi: tezroq, kam resurs, katta trafikka chidamli. Biz shuni o'rnatamiz.

### Qadam 1: Serverni yangilash

Avval VPS'ga SSH orqali kiring (`ssh root@server_ip` yoki oddiy `deploy` user bilan), so'ng:

```bash
sudo apt update && sudo apt upgrade -y
```

### Qadam 2: Nginx o'rnatish

```bash
sudo apt install nginx -y
sudo systemctl enable --now nginx
sudo systemctl status nginx
```

Brauzerda `http://server_ip` ochsangiz "Welcome to nginx!" ko'rinishi kerak.

**⚠️ Ehtiyot bo'l:** Agar sahifa ochilmasa, firewall'ni tekshiring:

```bash
sudo ufw allow 'Nginx Full'
sudo ufw allow OpenSSH
sudo ufw enable
sudo ufw status
```

### Qadam 3: PHP-FPM o'rnatish

Ubuntu 24.04 default repository'da PHP 8.3 bor. Laravel va zamonaviy loyihalar uchun yetarli:

```bash
sudo apt install php8.3-fpm php8.3-cli php8.3-mysql php8.3-mbstring \
  php8.3-xml php8.3-curl php8.3-zip php8.3-bcmath php8.3-gd -y
```

FPM'ni tekshiring:

```bash
sudo systemctl status php8.3-fpm
php -v
```

**💡 Tushuncha:** Yuqoridagi extension'lar (`mbstring`, `xml`, `curl`, `zip`, `bcmath`, `gd`) — Laravel va aksariyat PHP framework'lar talab qiladigan standart to'plam. `mysql` — MySQL bilan ishlash uchun; PostgreSQL ishlatsangiz `php8.3-pgsql`.

### Qadam 4: MySQL o'rnatish

```bash
sudo apt install mysql-server -y
sudo systemctl enable --now mysql
sudo mysql_secure_installation
```

`mysql_secure_installation` root parol qo'yish, anonim user o'chirish kabi savollarni beradi — hammasiga "Y" deb javob bering.

Database yaratish:

```bash
sudo mysql
```

```sql
CREATE DATABASE myapp CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER 'myapp_user'@'localhost' IDENTIFIED BY 'JudaKuchliParol123!';
GRANT ALL PRIVILEGES ON myapp.* TO 'myapp_user'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

**⚠️ Ehtiyot bo'l:** DB'ni to'liqroq boshqarish (backup, remote ulanish, xavfsizlik) alohida qo'llanmada — [VPS'da Ma'lumotlar Bazasi](./08-database-on-vps.md).

### Qadam 5: Composer o'rnatish

Composer — PHP'ning paket menejeri (npm'ning PHP'dagi ekvivalenti). Laravel va boshqa kutubxonalarni o'rnatish uchun kerak:

```bash
cd /tmp
curl -sS https://getcomposer.org/installer -o composer-setup.php
sudo php composer-setup.php --install-dir=/usr/local/bin --filename=composer
composer --version
```

---

## PHP-FPM nima va Nginx bilan qanday ishlaydi

**💡 Tushuncha:** **PHP-FPM** (FastCGI Process Manager) — PHP kodni ishlatuvchi alohida dastur. Nginx PHP'ni o'zi ishlata olmaydi (Apache'dagi `mod_php`dan farqli); u faqat statik fayllarni beradi. `.php` fayl kerak bo'lganda, Nginx uni **PHP-FPM**ga uzatadi, FPM ishlatib natijani qaytaradi.

Jarayon quyidagicha:

```
Brauzer  →  Nginx  →  (agar .php bo'lsa)  →  PHP-FPM  →  PHP kod ishlaydi  →  natija
                ↓
         (agar .css/.js/.png)  →  Nginx to'g'ridan-to'g'ri qaytaradi
```

Nginx va PHP-FPM o'zaro **socket** orqali gaplashadi. Ubuntu'da bu:

```
/run/php/php8.3-fpm.sock
```

Nginx config'da `fastcgi_pass` shu socket'ni ko'rsatadi:

```nginx
fastcgi_pass unix:/run/php/php8.3-fpm.sock;
```

**💡 Tushuncha:** `fastcgi_pass` — Nginx'ga "PHP kodni mana shu joyga yubor" degan buyruq. `unix:...sock` — mahalliy socket fayl (tarmoq portidan tezroq). Ba'zan `127.0.0.1:9000` (TCP port) ham ishlatiladi.

Socket manzilini tekshirish:

```bash
cat /etc/php/8.3/fpm/pool.d/www.conf | grep "listen ="
# listen = /run/php/php8.3-fpm.sock
```

---

## Oddiy PHP saytni Nginx bilan serve qilish

Endi oddiy PHP saytni Nginx orqali ishlataylik.

### Qadam 1: Sayt papkasini yaratish

```bash
sudo mkdir -p /var/www/mysite
sudo chown -R $USER:$USER /var/www/mysite
```

Test fayl:

```bash
echo "<?php phpinfo();" | sudo tee /var/www/mysite/index.php
```

### Qadam 2: Nginx server block yaratish

```bash
sudo nano /etc/nginx/sites-available/mysite
```

```nginx
server {
    listen 80;
    server_name sizningdomen.uz www.sizningdomen.uz;

    root /var/www/mysite;
    index index.php index.html;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/run/php/php8.3-fpm.sock;
    }

    location ~ /\.ht {
        deny all;
    }
}
```

**💡 Tushuncha:** Har bir bo'lakning ma'nosi:
- `root` — sayt fayllari qayerda joylashgan.
- `index` — papka ochilganda qaysi fayl birinchi ishlaydi.
- `location /` — har qanday URL uchun avval fayl bor-yo'qligini tekshiradi, bo'lmasa `index.php`ga yo'naltiradi (routing uchun muhim).
- `location ~ \.php$` — `.php` bilan tugagan har qanday so'rovni PHP-FPM'ga uzatadi.
- `location ~ /\.ht` — `.htaccess` kabi maxfiy fayllarni tashqariga bermaydi.

### Qadam 3: Yoqish va tekshirish

```bash
sudo ln -s /etc/nginx/sites-available/mysite /etc/nginx/sites-enabled/
sudo nginx -t          # config sintaksisini tekshirish
sudo systemctl reload nginx
```

**💡 Tushuncha:** Nginx'da saytlar `sites-available/`da yoziladi, so'ng `sites-enabled/`ga **symlink** (ln -s) orqali "yoqiladi". Bu saytni o'chirmasdan vaqtincha o'chirib qo'yish imkonini beradi (symlink'ni o'chirasiz, fayl qoladi).

Brauzerda `http://sizningdomen.uz` ochsangiz PHP info sahifasi ko'rinadi. Tayyor bo'lgach, `index.php`ni haqiqiy kodingiz bilan almashtiring.

---

## (c) Laravel Deploy

Endi eng ko'p uchraydigan holat — **Laravel** ilovasini production'ga to'liq deploy qilamiz.

### Qadam 1: Kodni serverga olib kelish

Eng yaxshi yo'l — Git orqali:

```bash
cd /var/www
sudo git clone https://github.com/username/loyiha.git myapp
sudo chown -R $USER:$USER /var/www/myapp
cd /var/www/myapp
```

### Qadam 2: Dependency'larni o'rnatish

```bash
composer install --optimize-autoloader --no-dev
```

**💡 Tushuncha:** `--no-dev` — test/development uchun kerakli paketlarni o'rnatmaydi (production'da kerak emas). `--optimize-autoloader` — autoloader'ni tezlashtiradi. Bu ikkovi production deploy'da **doim** ishlatiladi.

### Qadam 3: .env sozlash

```bash
cp .env.example .env
php artisan key:generate
nano .env
```

`.env` faylida muhim sozlamalar:

```
APP_NAME=MyApp
APP_ENV=production
APP_DEBUG=false
APP_URL=https://sizningdomen.uz

DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=myapp
DB_USERNAME=myapp_user
DB_PASSWORD=JudaKuchliParol123!
```

**⚠️ Ehtiyot bo'l:** Production'da `APP_DEBUG=false` **majburiy**! Agar `true` qolsa, xato yuz berganda maxfiy ma'lumotlar (DB parol, path'lar) brauzerda ko'rinadi — bu jiddiy xavfsizlik teshigi.

### Qadam 4: Migration ishga tushirish

```bash
php artisan migrate --force
```

**💡 Tushuncha:** `--force` — production muhitda `artisan migrate` xavfsizlik uchun tasdiq so'raydi; `--force` "ha, ishonchim komil" degani. Deploy skriptlarida shart.

### Qadam 5: Fayl permission'lari (www-data)

Laravel `storage/` va `bootstrap/cache/` papkalariga yozadi (log, cache, session). Nginx/PHP-FPM `www-data` user nomidan ishlaydi, shuning uchun bu papkalar `www-data`ga tegishli bo'lishi kerak:

```bash
sudo chown -R www-data:www-data /var/www/myapp/storage /var/www/myapp/bootstrap/cache
sudo chmod -R 775 /var/www/myapp/storage /var/www/myapp/bootstrap/cache
```

**⚠️ Ehtiyot bo'l:** Laravel'dagi eng ko'p uchraydigan xato — permission. "Failed to open stream: Permission denied" yoki oq/500 sahifa ko'rsangiz, birinchi shu ikki papkaning egasini tekshiring.

### Qadam 6: Config va route cache

```bash
php artisan config:cache
php artisan route:cache
php artisan view:cache
```

**💡 Tushuncha:** `config:cache` barcha config fayllarni bitta tez o'qiladigan faylga jamlaydi — production'da tezlikni oshiradi. **⚠️ Muhim:** har safar `.env`ni o'zgartirsangiz, `php artisan config:cache`ni qayta ishga tushiring, aks holda eski qiymatlar ishlatiladi. `config:clear` bilan tozalash mumkin.

### Qadam 7: Storage symlink

```bash
php artisan storage:link
```

Bu `public/storage`ni `storage/app/public`ga bog'laydi — yuklangan fayllar (rasm va h.k.) brauzerdan ko'rinishi uchun.

### Qadam 8: Nginx config (Laravel uchun)

Laravel'ning muhim jihati — **web root `public/` papkasi** bo'lishi kerak, loyihaning asosiy papkasi emas:

```bash
sudo nano /etc/nginx/sites-available/myapp
```

```nginx
server {
    listen 80;
    server_name sizningdomen.uz www.sizningdomen.uz;
    root /var/www/myapp/public;

    add_header X-Frame-Options "SAMEORIGIN";
    add_header X-Content-Type-Options "nosniff";

    index index.php;
    charset utf-8;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location = /favicon.ico { access_log off; log_not_found off; }
    location = /robots.txt  { access_log off; log_not_found off; }

    error_page 404 /index.php;

    location ~ \.php$ {
        fastcgi_pass unix:/run/php/php8.3-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        include fastcgi_params;
    }

    location ~ /\.(?!well-known).* {
        deny all;
    }
}
```

**⚠️ Ehtiyot bo'l:** `root` **`/var/www/myapp/public`** bo'lishi shart — `/public` qo'shmasangiz, hamma `.env`, kod fayllaringiz internetdan ochiq bo'lib qoladi! Bu jiddiy xavfsizlik xatosi.

```bash
sudo ln -s /etc/nginx/sites-available/myapp /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

### Qadam 9: Queue worker + Supervisor

Agar Laravel'da queue (navbat — email yuborish, og'ir vazifalar) ishlatsangiz, worker'ni doim ishlab turishi kerak. Buning uchun **Supervisor** ishlatiladi:

```bash
sudo apt install supervisor -y
sudo nano /etc/supervisor/conf.d/myapp-worker.conf
```

```ini
[program:myapp-worker]
process_name=%(program_name)s_%(process_num)02d
command=php /var/www/myapp/artisan queue:work --sleep=3 --tries=3 --max-time=3600
autostart=true
autorestart=true
stopasgroup=true
killasgroup=true
user=www-data
numprocs=2
redirect_stderr=true
stdout_logfile=/var/www/myapp/storage/logs/worker.log
stopwaitsecs=3600
```

```bash
sudo supervisorctl reread
sudo supervisorctl update
sudo supervisorctl start myapp-worker:*
sudo supervisorctl status
```

**💡 Tushuncha:** **Supervisor** — Linux jarayonlarini doim ishlatib turadigan dastur. Worker o'chib qolsa (crash, xato), avtomatik qayta ishga tushiradi. `numprocs=2` — 2 ta parallel worker. Node.js dunyosidagi **PM2**ning PHP'dagi ekvivalenti. (PM2'ni ham ishlatish mumkin, lekin PHP CLI jarayonlari uchun Supervisor standart hisoblanadi.)

**⚠️ Ehtiyot bo'l:** Kod yangilanganda worker'lar eski kodni xotirada ushlab turadi. Har deploy'dan keyin ularni qayta ishga tushiring:

```bash
php artisan queue:restart
```

### PHP versiya boshqaruvi

Agar loyihalaringiz turli PHP versiyalarini talab qilsa (biri 8.1, biri 8.3), bir nechta versiyani o'rnatib, almashtirish mumkin:

```bash
sudo add-apt-repository ppa:ondrej/php -y
sudo apt update
sudo apt install php8.1-fpm php8.1-cli -y

# CLI default versiyani almashtirish
sudo update-alternatives --config php
```

Har bir sayt uchun Nginx config'da mos socket'ni ko'rsatasiz:

```nginx
fastcgi_pass unix:/run/php/php8.1-fpm.sock;   # yoki 8.3
```

---

## Domain va SSL (certbot)

Sayt ishlagach, HTTPS (yashil qulf) qo'shamiz. Bepul Let's Encrypt sertifikati **certbot** orqali:

### Qadam 1: DNS sozlash

Domen provayderingizda **A record** yarating: domeningizni VPS IP manziliga yo'naltiring.

```
A    @      123.45.67.89
A    www    123.45.67.89
```

DNS tarqalganini tekshiring:

```bash
dig +short sizningdomen.uz
```

### Qadam 2: Certbot o'rnatish va sertifikat olish

```bash
sudo apt install certbot python3-certbot-nginx -y
sudo certbot --nginx -d sizningdomen.uz -d www.sizningdomen.uz
```

Certbot avtomatik ravishda:
- Sertifikat oladi
- Nginx config'ni HTTPS uchun sozlaydi
- HTTP → HTTPS redirect qo'shadi

### Qadam 3: Avtomatik yangilanishni tekshirish

Let's Encrypt sertifikati 90 kun amal qiladi. Certbot avtomatik yangilaydi:

```bash
sudo certbot renew --dry-run
```

**💡 Tushuncha:** `--dry-run` — haqiqiy yangilashsiz, jarayon ishlayotganini "sinov" qiladi. Muvaffaqiyatli bo'lsa, avtomatik yangilanish sozlangan.

---

## Keng uchraydigan xatolar

### 500 Internal Server Error

Eng noaniq xato. Log'ni ko'ring:

```bash
tail -50 /var/www/myapp/storage/logs/laravel.log
tail -50 /var/log/nginx/error.log
```

Ko'p sabablari: permission, noto'g'ri `.env`, migration ishlamagani, `APP_KEY` yo'qligi.

### Permission denied / oq sahifa

```bash
sudo chown -R www-data:www-data /var/www/myapp/storage /var/www/myapp/bootstrap/cache
sudo chmod -R 775 /var/www/myapp/storage /var/www/myapp/bootstrap/cache
```

### .env o'zgarishi ishlamayapti

Config cache eski qiymatni ushlab turgan:

```bash
php artisan config:clear
php artisan config:cache
```

### "502 Bad Gateway"

Nginx PHP-FPM socket'ini topa olmayapti. FPM ishlayotganini va socket manzili to'g'riligini tekshiring:

```bash
sudo systemctl status php8.3-fpm
sudo systemctl restart php8.3-fpm
```

---

## Amaliy Q&A

### ❓ Shared hosting'da Laravel deploy qila olamanmi?

**✅ Javob:** Ba'zilarida ha, lekin cheklovlar bilan. Muammolar: SSH ko'pincha yo'q yoki cheklangan (composer install qiyinlashadi), queue worker/supervisor ishlamaydi, cron cheklangan bo'lishi mumkin. Oddiy Laravel sayt ishlaydi, lekin queue va scheduled task kerak bo'lsa VPS oling. Web root'ni `public_html`ga Laravel'ning `public/` papkasiga yo'naltirishni ham sozlashingiz kerak bo'ladi.

### ❓ `www-data` nima va nega hamma narsa unga tegishli bo'lishi kerak?

**✅ Javob:** `www-data` — Ubuntu'da Nginx va PHP-FPM ishlaydigan tizim useri. Sayt request'ni qayta ishlaganda, PHP kod `www-data` huquqi bilan bajariladi. Shuning uchun Laravel yozadigan papkalar (`storage`, `bootstrap/cache`) `www-data`ga tegishli bo'lishi kerak — aks holda yoza olmaydi va xato beradi. Butun loyihani `www-data`ga bermang; faqat yoziladigan papkalarni.

### ❓ `chmod 777` qo'ysam hamma muammo hal bo'ladimi?

**✅ Javob:** Yo'q, buni **hech qachon qilmang!** `777` — har kimga o'qish, yozish, ishga tushirish huquqi beradi, ya'ni server buzilsa hujumchi fayllaringizni o'zgartira oladi. To'g'ri yo'l: to'g'ri **egalik** (`chown www-data`) + `775` yoki `755`. `777` — vaqtinchalik ham xavfli.

### ❓ Apache'ni to'liq tark etib, faqat Nginx ishlatsam bo'ladimi?

**✅ Javob:** Ha, va tavsiya etiladi. Nginx + PHP-FPM zamonaviy standart. Agar `.htaccess`ga bog'liq eski loyiha bo'lsa (ba'zi WordPress plaginlari), qoidalarni Nginx `location` bloklariga o'girish kerak bo'ladi. Yangi loyihalar uchun to'g'ridan-to'g'ri Nginx bilan boshlang.

### ❓ `composer install` server'da xotira yetmasligi xatosini berayapti?

**✅ Javob:** Kichik VPS'da (512MB-1GB RAM) `composer install` xotira tugatishi mumkin. Yechimlar: (1) swap fayl qo'shing:

```bash
sudo fallocate -l 2G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
```

(2) Yoki `COMPOSER_MEMORY_LIMIT=-1 composer install` bilan ishlating. (3) Yoki localhost'da `composer install` qilib, `vendor/` papkasini serverga yuklang.

### ❓ Har deploy'da qaysi buyruqlarni ketma-ket ishlatishim kerak?

**✅ Javob:** Laravel uchun standart deploy ketma-ketligi:

```bash
cd /var/www/myapp
git pull origin main
composer install --optimize-autoloader --no-dev
php artisan migrate --force
php artisan config:cache
php artisan route:cache
php artisan view:cache
php artisan queue:restart
```

Buni bitta `deploy.sh` skriptga solib, `bash deploy.sh` bilan chaqiring.

### ❓ Nginx config'ni o'zgartirdim, lekin o'zgarish ko'rinmayapti?

**✅ Javob:** Config'ni yozgach **doim** ikki qadam qiling:

```bash
sudo nginx -t              # sintaksisni tekshir
sudo systemctl reload nginx  # qayta yukla
```

`nginx -t` xatoni ko'rsatadi va reload qilmaydi — bu sizni buzilgan config bilan saytni o'chirib qo'yishdan saqlaydi. Faqat `reload` (restart emas) — ulanishlarni uzmasdan qayta o'qiydi.

### ❓ Laravel `.env` fayl Git'da bo'lishi kerakmi?

**✅ Javob:** **Yo'q, hech qachon!** `.env`da DB parol, API kalit kabi maxfiy ma'lumotlar bor. `.gitignore`da bo'lishi shart (Laravel'da default bor). Server'da `.env`ni qo'lda yoki xavfsiz yo'l bilan yaratasiz. Faqat `.env.example` (parolsiz namuna) Git'da bo'ladi.

### ❓ Sayt "This site can't be reached" beryapti, garchi Nginx ishlayotgan bo'lsa?

**✅ Javob:** Tekshiring: (1) DNS to'g'ri IP'ga yo'naltirilganmi (`dig sizningdomen.uz`), (2) firewall port 80/443'ni ochganmi (`sudo ufw status`), (3) `server_name` config'da domeningizga mos kelyaptimi. Ko'pincha DNS hali tarqalmagan bo'ladi — bir necha soat kutish kerak.

### ❓ PHP versiyasini qanday tekshiraman va sayt qaysi versiyada ishlayotganini bilaman?

**✅ Javob:** CLI versiyasi: `php -v`. Lekin sayt (FPM) boshqa versiyada bo'lishi mumkin! Nginx config'da qaysi socket (`php8.3-fpm.sock`) ko'rsatilganiga qarang, yoki `<?php phpinfo();` sahifa ochib brauzerda ko'ring. FPM va CLI versiyalari ba'zan farq qiladi.

### ❓ Queue worker ishlayotganini qanday bilaman?

**✅ Javob:** `sudo supervisorctl status` — barcha worker holatini ko'rsatadi (RUNNING bo'lishi kerak). Test uchun bir job dispatch qilib, `storage/logs/worker.log`ni kuzating. RUNNING emas bo'lsa, log'da xatoni ko'ring — ko'pincha permission yoki `.env` muammosi.

---

## Amaliy Checklist

- [ ] Loyiha turi aniqlandi: oddiy PHP → shared hosting, Laravel → VPS
- [ ] VPS'da server yangilandi (`apt update && upgrade`)
- [ ] Nginx o'rnatildi va firewall (UFW) sozlandi
- [ ] PHP 8.3-FPM va kerakli extension'lar o'rnatildi
- [ ] MySQL o'rnatildi, `mysql_secure_installation` bajarildi
- [ ] Database va user yaratildi (`CREATE DATABASE`, `GRANT`)
- [ ] Composer o'rnatildi
- [ ] Kod Git orqali serverga olindi
- [ ] `composer install --no-dev --optimize-autoloader` bajarildi
- [ ] `.env` sozlandi, `APP_DEBUG=false`, `APP_KEY` yaratildi
- [ ] `php artisan migrate --force` bajarildi
- [ ] `storage/` va `bootstrap/cache/` egaligi `www-data`ga berildi (775)
- [ ] `config:cache`, `route:cache`, `view:cache` bajarildi
- [ ] Nginx config `root`i `public/` papkasiga yo'naltirildi
- [ ] Queue kerak bo'lsa Supervisor sozlandi
- [ ] DNS A record VPS IP'ga yo'naltirildi
- [ ] Certbot bilan SSL (HTTPS) o'rnatildi
- [ ] Deploy skripti (`deploy.sh`) tayyorlandi

---

← [Deployment bo'limiga qaytish](./README.md)
