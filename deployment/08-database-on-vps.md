# VPS'da Ma'lumotlar Bazasi

Har bir jiddiy ilova ma'lumotlar bazasiga (database) muhtoj — foydalanuvchilar, buyurtmalar, postlar hammasi shu yerda saqlanadi. Bu qo'llanmada VPS'da ma'lumotlar bazasini **yaratish, sozlash, xavfsizlash va zaxiralashni** o'rganamiz: PostgreSQL va MySQL o'rnatishdan tortib, backup, restore, remote xavfsiz ulanish va monitoringgacha. Ubuntu 24.04 uchun real buyruqlar bilan.

## Mundarija

- [Managed DB vs Self-hosted (VPS)](#managed-db-vs-self-hosted-vps)
- [PostgreSQL o'rnatish](#postgresql-ornatish)
- [Database va user yaratish (PostgreSQL)](#database-va-user-yaratish-postgresql)
- [MySQL/MariaDB alternativi](#mysqlmariadb-alternativi)
- [Xavfsizlik: localhost va tashqi ulanish](#xavfsizlik-localhost-va-tashqi-ulanish)
- [App'ni DB'ga ulash](#appni-dbga-ulash)
- [Backup: zaxira va restore](#backup-zaxira-va-restore)
- [Migration'lar production'da](#migrationlar-productionda)
- [DB monitoring asoslari](#db-monitoring-asoslari)
- [Remote'dan xavfsiz ulanish (SSH tunnel)](#remotedan-xavfsiz-ulanish-ssh-tunnel)
- [Amaliy Q&A](#amaliy-qa)
- [Amaliy Checklist](#amaliy-checklist)

---

## Managed DB vs Self-hosted (VPS)

Ma'lumotlar bazasini joylashtirishning ikki asosiy yo'li bor: **managed** (boshqa kompaniya boshqaradi) va **self-hosted** (o'zingiz VPS'da o'rnatasiz).

| Xususiyat | Managed DB (RDS, Supabase, Neon, Railway) | Self-hosted (VPS'da o'zingiz) |
|-----------|-------------------------------------------|-------------------------------|
| O'rnatish | Bir necha klik | Qo'lda o'rnatasiz |
| Backup | Avtomatik | O'zingiz sozlaysiz |
| Yangilanish/xavfsizlik | Ular hal qiladi | Sizning zimmangizda |
| Narx | Qimmatroq | Arzon (VPS ichida) |
| Nazorat | Cheklangan | To'liq |
| Masshtablash | Oson (bir tugma) | Qo'lda, murakkab |

**💡 Tushuncha:** **Managed DB** — Amazon (RDS), Supabase, Neon, Railway kabi kompaniyalar sizning ma'lumotlar bazangizni o'z serverlarida boshqaradi. Siz faqat connection string olasiz, backup/xavfsizlik/yangilanish ular zimmasida. **Self-hosted** — DB'ni o'z VPS'ingizga o'rnatasiz, hamma narsa sizning javobgarligingizda.

**Tavsiya:**
- **Boshlovchi** yoki mahsulotni tez ishga tushirmoqchi bo'lsangiz → **managed** (Supabase, Neon bepul rejalari bor). Backup va xavfsizlik haqida bosh qotirmaysiz.
- **Byudjet cheklangan**, DB'ni o'rganmoqchi, yoki o'zbek hostingda VPS olgan bo'lsangiz → **self-hosted** (bu qo'llanmaning asosiy mavzusi).

**⚠️ Ehtiyot bo'l:** Self-hosted arzon ko'rinadi, lekin backup, xavfsizlik, monitoring — hammasi sizning ishingiz. Agar backup sozlamasangiz va disk buzilsa, hamma ma'lumot yo'qoladi. Managed DB narxi ana shu xotirjamlik uchun.

---

## PostgreSQL o'rnatish

PostgreSQL — kuchli, ochiq kodli, zamonaviy loyihalar uchun eng ko'p tavsiya etiladigan DB. Ubuntu 24.04'ga o'rnatamiz.

### Qadam 1: O'rnatish

```bash
sudo apt update
sudo apt install postgresql postgresql-contrib -y
```

`postgresql-contrib` — qo'shimcha foydali extension'lar (masalan `uuid-ossp`).

### Qadam 2: Ishlayotganini tekshirish

```bash
sudo systemctl status postgresql
sudo systemctl enable postgresql   # boot'da avtomatik ishga tushishi uchun
```

### Qadam 3: Versiyani tekshirish

```bash
psql --version
# psql (PostgreSQL) 16.x
```

**💡 Tushuncha:** O'rnatilgach, PostgreSQL avtomatik `postgres` nomli tizim useri va DB superuser yaratadi. Barcha admin ishlar shu user orqali qilinadi.

---

## Database va user yaratish (PostgreSQL)

### Qadam 1: psql'ga kirish

```bash
sudo -u postgres psql
```

**💡 Tushuncha:** `sudo -u postgres` — `postgres` tizim useri nomidan buyruq bajaradi. `psql` — PostgreSQL'ning interaktiv terminali (SQL yozadigan joy). Kirgach `postgres=#` ko'rinadigan prompt paydo bo'ladi.

### Qadam 2: Database yaratish

```sql
CREATE DATABASE myapp;
```

### Qadam 3: User yaratish va parol qo'yish

```sql
CREATE USER myapp_user WITH PASSWORD 'JudaKuchliParol123!';
```

### Qadam 4: Privilegiyalar berish

```sql
GRANT ALL PRIVILEGES ON DATABASE myapp TO myapp_user;
```

PostgreSQL 15+ da schema'ga ham alohida ruxsat kerak. Avval to'g'ri database'ga o'ting:

```sql
\c myapp
GRANT ALL ON SCHEMA public TO myapp_user;
```

**⚠️ Ehtiyot bo'l:** PostgreSQL 15 dan boshlab yangi user default `public` schema'ga jadval yarata olmaydi — bu ko'p uchraydigan xato ("permission denied for schema public"). Yuqoridagi `GRANT ALL ON SCHEMA public` qatorini unutmang.

### Qadam 5: Chiqish

```sql
\q
```

### Qadam 6: Yangi user bilan ulanishni tekshirish

```bash
psql -h localhost -U myapp_user -d myapp
```

Parolni so'raydi. Kirsangiz — hammasi to'g'ri.

**Foydali psql buyruqlari:**

```sql
\l          -- barcha database'lar ro'yxati
\du         -- barcha user'lar
\dt         -- joriy DB'dagi jadvallar
\c dbname   -- boshqa DB'ga o'tish
\q          -- chiqish
```

---

## MySQL/MariaDB alternativi

Agar loyihangiz MySQL talab qilsa (ko'p PHP/Laravel loyihalar, WordPress), MySQL yoki uning ochiq varianti MariaDB o'rnatiladi.

### MySQL o'rnatish

```bash
sudo apt install mysql-server -y
sudo systemctl enable --now mysql
```

### Xavfsizlashtirish

```bash
sudo mysql_secure_installation
```

**💡 Tushuncha:** `mysql_secure_installation` — MySQL'ni xavfsizlashtiruvchi skript. U: root parol qo'yadi, anonim user'larni o'chiradi, remote root loginni bloklaydi, test database'ni o'chiradi. Barcha savollarga "Y" (ha) deb javob bering.

### Database va user yaratish (MySQL)

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

**💡 Tushuncha:** `'myapp_user'@'localhost'` — bu user faqat **localhost'dan** ulana oladi. `@'%'` bo'lsa har joydan — bu xavfli, ehtiyot bo'ling (pastda batafsil). `FLUSH PRIVILEGES` — o'zgarishlarni darhol kuchga kiritadi.

**MariaDB** — MySQL'ning to'liq mos ochiq varianti. `sudo apt install mariadb-server` bilan o'rnatiladi, buyruqlar bir xil.

---

## Xavfsizlik: localhost va tashqi ulanish

Bu bo'lim **eng muhimi**. Noto'g'ri sozlangan DB — internetdagi eng ko'p hack qilinadigan narsalardan biri.

### Nega DB'ni tashqariga ochmaslik kerak

**💡 Tushuncha:** Default holatda PostgreSQL va MySQL faqat **localhost** (`127.0.0.1`) ni tinglaydi — ya'ni faqat **o'sha serverdagi** ilovalar ulana oladi. Bu xavfsiz, chunki app va DB bir serverda bo'lsa, tashqi internet DB'ga umuman yeta olmaydi.

**⚠️ Ehtiyot bo'l:** DB'ni tashqi internetga ochish (`0.0.0.0` tinglatish) — hujumchilarga eshik ochadi. Botlar internetni doim skanerlaydi, ochiq DB portini (5432, 3306) topib, zaif parolni brute-force qiladi. Real hodisalar: minglab ochiq MongoDB/PostgreSQL bazalari o'g'irlangan yoki ransomware bilan shifrlangan. **Iloji boricha DB'ni localhost'da qoldiring.**

### PostgreSQL'ni faqat localhost'da tinglatish (default)

Sozlamani tekshiring:

```bash
sudo nano /etc/postgresql/16/main/postgresql.conf
```

```
listen_addresses = 'localhost'
```

Bu default va shunday qolishi kerak (agar remote kerak bo'lmasa).

Kim ulanishi mumkinligi `pg_hba.conf`da boshqariladi:

```bash
sudo nano /etc/postgresql/16/main/pg_hba.conf
```

```
# TYPE  DATABASE  USER        ADDRESS         METHOD
local   all       all                         peer
host    all       all         127.0.0.1/32    scram-sha-256
```

**💡 Tushuncha:** `pg_hba.conf` (Host-Based Authentication) — kim, qaysi DB'ga, qayerdan, qanday usulda ulana olishini belgilaydi. `scram-sha-256` — zamonaviy xavfsiz parol usuli.

### MySQL'ni localhost'da tinglatish (default)

```bash
sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf
```

```
bind-address = 127.0.0.1
```

Bu default. `0.0.0.0`ga o'zgartirmang, agar remote zarur bo'lmasa.

### Agar remote ulanish HAQIQATAN kerak bo'lsa

Ba'zan app va DB alohida serverlarda bo'ladi. U holda **uch qatlamli himoya** shart:

1. **Kuchli parol** (uzun, tasodifiy).
2. **Firewall** — faqat ma'lum IP'ga port ochish:

```bash
sudo ufw allow from 123.45.67.89 to any port 5432
```

Bu faqat `123.45.67.89` (sizning app serveringiz IP'si) ulana oladi, boshqa hech kim.

3. **SSL/TLS** — ulanishni shifrlash (parol tarmoqda ochiq ketmasligi uchun).

**⚠️ Ehtiyot bo'l:** DB'ni butun internetga ochishdan ko'ra, **SSH tunnel** ishlatish ancha xavfsiz (pastdagi bo'limga qarang). Remote ulanishga hech qachon `bind-address = 0.0.0.0` + `ufw allow 5432` (hammaga) qo'ymang.

---

## App'ni DB'ga ulash

App va DB bir VPS'da bo'lganda, ulanish **`localhost`** orqali. Bu eng tez va xavfsiz variant.

### Laravel (.env)

```
DB_CONNECTION=pgsql
DB_HOST=127.0.0.1
DB_PORT=5432
DB_DATABASE=myapp
DB_USERNAME=myapp_user
DB_PASSWORD=JudaKuchliParol123!
```

MySQL uchun `DB_CONNECTION=mysql`, `DB_PORT=3306`.

### Node.js (connection string)

```
DATABASE_URL="postgresql://myapp_user:JudaKuchliParol123!@localhost:5432/myapp"
```

**💡 Tushuncha:** **Connection string** — DB ulanish ma'lumotlarini bitta qatorga jamlash. Format: `protokol://user:parol@host:port/dbname`. `localhost` yoki `127.0.0.1` — DB shu serverdaligini bildiradi.

**⚠️ Ehtiyot bo'l:** Parolda maxsus belgilar (`@`, `:`, `/`) bo'lsa, connection string'ni buzadi. Ularni URL-encode qiling (masalan `@` → `%40`) yoki parolda bunday belgilarni ishlatmang.

---

## Backup: zaxira va restore

Backup — DB'ning eng muhim vazifasi. Disk buzilishi, xato buyruq, hack — hammasidan zaxira saqlaydi.

### PostgreSQL backup (pg_dump)

```bash
pg_dump -U myapp_user -h localhost myapp > myapp_backup.sql
```

Siqilgan (kichikroq) backup:

```bash
pg_dump -U myapp_user -h localhost myapp | gzip > myapp_backup.sql.gz
```

### PostgreSQL restore

```bash
psql -U myapp_user -h localhost myapp < myapp_backup.sql
```

Siqilgan backup'dan:

```bash
gunzip < myapp_backup.sql.gz | psql -U myapp_user -h localhost myapp
```

### MySQL backup (mysqldump)

```bash
mysqldump -u myapp_user -p myapp > myapp_backup.sql
```

### MySQL restore

```bash
mysql -u myapp_user -p myapp < myapp_backup.sql
```

### Avtomatik backup (cron)

Qo'lda backup unutiladi. Uni avtomatlashtiramiz. Backup skripti yozing:

```bash
sudo nano /usr/local/bin/db-backup.sh
```

```bash
#!/bin/bash
BACKUP_DIR="/var/backups/postgres"
DATE=$(date +%Y-%m-%d_%H-%M)
mkdir -p "$BACKUP_DIR"

pg_dump -U myapp_user -h localhost myapp | gzip > "$BACKUP_DIR/myapp_$DATE.sql.gz"

# 7 kundan eski backuplarni o'chirish
find "$BACKUP_DIR" -name "*.sql.gz" -mtime +7 -delete
```

Ishga tushirish huquqi berib, cron'ga qo'shing:

```bash
sudo chmod +x /usr/local/bin/db-backup.sh
sudo crontab -e
```

Har kuni tunda soat 03:00'da backup:

```
0 3 * * * /usr/local/bin/db-backup.sh
```

**💡 Tushuncha:** `0 3 * * *` — cron formati: (daqiqa=0, soat=3, har kun, har oy, har hafta kuni). Ya'ni har kuni 03:00'da ishlaydi. `find ... -mtime +7 -delete` — 7 kundan eski fayllarni o'chiradi, disk to'lib ketmasligi uchun.

**⚠️ Ehtiyot bo'l:** Backup'ni **faqat o'sha VPS'da** saqlamang! Server butunlay buzilsa, backup ham yo'qoladi. Backuplarni tashqi joyga ko'chiring — masalan `rclone` bilan Cloud storage'ga (S3, Google Drive), yoki boshqa serverga `scp` orqali. Backup — faqat u boshqa joyda saqlansa, backup.

### Restore'ni sinab ko'ring

**⚠️ Ehtiyot bo'l:** Sinalmagan backup — backup emas! Vaqti-vaqti bilan test database'ga restore qilib, backup haqiqatan ishlashini tekshiring. Ko'p kompaniyalar backup bor deb o'ylagan, lekin kerak bo'lganda buzuq ekanini bilib qolgan.

---

## Migration'lar production'da

Migration — DB strukturasini (jadvallar, ustunlar) kod orqali o'zgartirish. Production'da ehtiyotkorlik talab qiladi.

### Laravel

```bash
php artisan migrate --force
```

`--force` — production'da tasdiqsiz ishga tushiradi (deploy skript uchun).

### Migration'dan oldin backup

**⚠️ Ehtiyot bo'l:** Production'da migration ishga tushirishdan **oldin doim backup** oling! Migration jadval o'chirsa yoki ustun o'zgartirsa, xato bo'lsa ma'lumot yo'qoladi:

```bash
pg_dump -U myapp_user -h localhost myapp | gzip > pre_migrate_$(date +%F).sql.gz
php artisan migrate --force
```

**💡 Tushuncha:** Migration'lar **oldinga** (up) va **orqaga** (down/rollback) bo'ladi. Xato bo'lsa `php artisan migrate:rollback` bilan qaytarasiz. Lekin ma'lumot o'chiruvchi migration'ni (masalan ustun o'chirish) rollback tiklay olmaydi — shuning uchun backup birlamchi himoya.

**⚠️ Ehtiyot bo'l:** Katta jadvalda (millionlab qator) migration jadvalni "lock" qilib, saytni sekinlashtirishi yoki to'xtatishi mumkin. Katta o'zgarishlarni kam trafikli vaqtda (tunda) qiling.

---

## DB monitoring asoslari

DB sog'lig'ini kuzatish — muammoni oldindan sezish uchun.

### Faol ulanishlar (PostgreSQL)

```sql
SELECT count(*) FROM pg_stat_activity;
SELECT pid, usename, state, query FROM pg_stat_activity WHERE state = 'active';
```

### Database hajmi

```sql
SELECT pg_size_pretty(pg_database_size('myapp'));
```

### Sekin so'rovlar

`postgresql.conf`da sekin so'rovlarni log qilish:

```
log_min_duration_statement = 1000   # 1 soniyadan uzun so'rovlarni logla
```

### Disk va xotira (server darajasi)

```bash
df -h              # disk to'lganini tekshirish
free -h            # xotira holati
sudo systemctl status postgresql
```

**⚠️ Ehtiyot bo'l:** DB'ning eng ko'p uchraydigan production muammosi — **disk to'lib ketishi** (log, backup, DB o'sishidan). `df -h` bilan muntazam tekshiring. Disk 100% to'lsa, DB yozishni to'xtatadi va sayt ishlamay qoladi.

**💡 Tushuncha:** Jiddiy loyihalar uchun `pg_stat_statements` extension (so'rov statistikasi) yoki tashqi monitoring (Grafana + Prometheus, yoki Netdata) sozlanadi. Boshlash uchun yuqoridagi buyruqlar yetarli.

---

## Remote'dan xavfsiz ulanish (SSH tunnel)

Ba'zan localhost'dagi DB'ga o'z kompyuteringizdan (masalan DBeaver, TablePlus, pgAdmin bilan) ulanish kerak — lekin DB'ni internetga ochmasdan. Yechim: **SSH tunnel**.

**💡 Tushuncha:** **SSH tunnel** — SSH ulanishi orqali DB portini xavfsiz "quvur" bilan o'z kompyuteringizga olib keladi. DB internetga ochilmaydi (localhost'da qoladi), lekin siz SSH orqali unga yeta olasiz. Bu remote ulanishning eng xavfsiz usuli.

### Terminal orqali tunnel yaratish

O'z kompyuteringizdan:

```bash
ssh -L 5433:localhost:5432 deploy@server_ip
```

**💡 Tushuncha:** Bu buyruq: "Mening kompyuterimning `5433` portini, server orqali serverning `localhost:5432` (PostgreSQL) portiga ulab ber". Endi o'z kompyuteringizda `localhost:5433`ga ulansangiz, aslida serverdagi DB'ga ulanasiz.

Tunnel ochiq turganda, DB client'ida:

```
Host:     localhost
Port:     5433
Database: myapp
User:     myapp_user
Password: JudaKuchliParol123!
```

MySQL uchun `5432` o'rniga `3306`, mahalliy port sifatida `3307`.

### GUI client'larda (TablePlus, DBeaver)

Bu dasturlarda "SSH tunnel" yorlig'i bor — u yerda SSH ma'lumotlarini (server IP, SSH user, kalit) kiritasiz, ular tunnel'ni avtomatik yaratadi. Buyruq yozish shart emas.

**⚠️ Ehtiyot bo'l:** SSH tunnel DB'ni internetga ochishdan **ancha xavfsiz**. Agar biror maslahat "DB portni tashqariga och" desa, avval SSH tunnel yetarli emasligiga ishonch hosil qiling — 99% holatda tunnel yetarli.

---

## Amaliy Q&A

### ❓ Boshlovchi sifatida managed DB (Supabase) yoki VPS'da o'zim o'rnatishimni tanlaymanmi?

**✅ Javob:** Boshlovchi uchun **managed DB tavsiya etiladi** (Supabase, Neon bepul rejalari bor). Backup, xavfsizlik, yangilanish — hammasi ular zimmasida, siz kodga e'tibor berasiz. VPS'da o'zingiz o'rnatishni DB'ni chuqurroq o'rganmoqchi bo'lsangiz yoki byudjet juda cheklangan bo'lsa tanlang. Ammo o'rganish uchun VPS'da o'rnatib ko'rish juda foydali — bu qo'llanma shuning uchun.

### ❓ DB va app bir VPS'da bo'lishi kerakmi, yoki alohida?

**✅ Javob:** Kichik/o'rta loyihalar uchun **bitta VPS** yetarli va oson — app va DB `localhost` orqali gaplashadi (tez va xavfsiz). Katta yuk yoki masshtablash kerak bo'lganda alohida DB serverga ajratasiz. Boshlash uchun bitta VPS'da qoldiring.

### ❓ `postgres` useri parolini so'ramayapti, bu xavfli emasmi?

**✅ Javob:** `sudo -u postgres psql` parol so'ramaydi, chunki u `peer` autentifikatsiya ishlatadi — ya'ni Linux'da `postgres` useri bo'lsangiz, DB'ga ham kirasiz. Bu localhost'da xavfsiz, chunki `sudo` huquqi allaqachon kerak. Tashqi ulanishlar uchun esa parol (`scram-sha-256`) talab qilinadi.

### ❓ "permission denied for schema public" xatosi nima?

**✅ Javob:** PostgreSQL 15+ da yangi user default `public` schema'ga jadval yarata olmaydi. Yechim: superuser bilan kirib, `\c myapp` (database'ga o'ting), so'ng `GRANT ALL ON SCHEMA public TO myapp_user;` bajaring. Bu Laravel migration'i "permission denied" berayotganda eng ko'p uchraydigan sabab.

### ❓ Backup'ni qanchalik tez-tez olishim kerak?

**✅ Javob:** Loyiha muhimligiga qarab. Aktiv sayt uchun kuniga kamida bir marta (cron bilan). Muhim moliyaviy/biznes ma'lumotlar uchun har bir necha soatda. **Eng muhimi:** backup'ni **boshqa joyda** saqlang (Cloud storage, boshqa server) — bir VPS'da saqlangan backup server buzilsa foydasiz.

### ❓ MySQL yoki PostgreSQL — qaysi birini tanlaymanmi?

**✅ Javob:** Ikkovi ham a'lo. **PostgreSQL** — zamonaviy, ilg'or funksiyalar (JSON, full-text search, murakkab so'rovlar) uchun tavsiya etiladi. **MySQL** — WordPress, ba'zi PHP loyihalar talab qilsa, yoki sizga tanish bo'lsa. Yangi loyiha uchun ko'pchilik PostgreSQL'ni tanlaydi, lekin ikkovi ham to'g'ri tanlov.

### ❓ DB parolini `.env`da ochiq saqlash xavfsizmi?

**✅ Javob:** `.env` fayl **Git'ga tushmasa** (`.gitignore`da bo'lsa) va server fayl huquqlari to'g'ri bo'lsa (`chmod 600`, faqat egasi o'qiy oladi) — bu odatiy amaliyot. Muhim: `.env`ni GitHub'ga push qilmang va public papkaga (`public/`) qo'ymang. Kattaroq tizimlarda "secrets manager" (Vault, AWS Secrets Manager) ishlatiladi.

### ❓ DB portini (5432/3306) firewall'da ochishim kerakmi?

**✅ Javob:** **Yo'q**, agar app bir VPS'da bo'lsa. DB localhost orqali ulanadi, port tashqariga ochilmasligi kerak. Faqat DB alohida serverda bo'lsa va remote kerak bo'lsa — u holda **faqat app serveri IP'siga** ochasiz (`ufw allow from APP_IP to any port 5432`), hammaga emas. Remote GUI ulanish uchun esa firewall ochish o'rniga SSH tunnel ishlating.

### ❓ Restore qilmoqchiman, lekin database allaqachon mavjud — nima qilaman?

**✅ Javob:** Odatda avval bo'sh database yaratasiz yoki mavjudini tozalaysiz. Ehtiyot uchun avval joriy holatning backup'ini oling, so'ng: yangi database yaratib (`CREATE DATABASE`), unga restore qiling. Mavjud DB ustiga restore qilsangiz, `pg_dump`ning `--clean` opsiyasi eski jadvallarni o'chirib qayta yaratadi — lekin bu ma'lumotni yo'qotadi, ehtiyot bo'ling.

### ❓ SSH tunnel har safar qo'lda ochish zerikarli, avtomatlashtira olamanmi?

**✅ Javob:** Ha. TablePlus/DBeaver kabi GUI client'larda "SSH tunnel" sozlamasi bor — bir marta sozlaysiz, ular avtomatik ochadi. Terminal uchun `~/.ssh/config`da alias yaratib, `LocalForward 5433 localhost:5432` qatorini qo'shasiz — keyin faqat `ssh myserver` yetarli.

### ❓ Disk to'lib ketsa DB'ga nima bo'ladi?

**✅ Javob:** Disk 100% to'lsa DB yangi ma'lumot yoza olmaydi va xato beradi ("could not write to file"), sayt ishlamay qoladi. Buning oldini olish: `df -h` bilan muntazam tekshiring, eski backup/log fayllarni tozalang (backup skriptidagi `find -mtime +7 -delete` shuning uchun), diskni oshiring. Bu production'dagi eng ko'p uchraydigan avariyalardan biri.

### ❓ PostgreSQL'ni yangilaganimda (major version) ma'lumotlarim yo'qoladimi?

**✅ Javob:** Major yangilanish (masalan 15 → 16) avtomatik emas — `pg_upgrade` yoki dump/restore kerak. **Doim avval to'liq backup oling.** Minor yangilanish (16.1 → 16.2) `apt upgrade` bilan xavfsiz, ma'lumotga tegmaydi. Production'da major yangilanishni ehtiyotkorlik bilan, avval test muhitda sinab ko'ring.

---

## Amaliy Checklist

- [ ] Managed vs self-hosted tanlandi (boshlovchiga managed tavsiya)
- [ ] PostgreSQL yoki MySQL o'rnatildi
- [ ] `mysql_secure_installation` bajarildi (MySQL bo'lsa)
- [ ] Database yaratildi (`CREATE DATABASE`)
- [ ] User yaratildi va kuchli parol qo'yildi (`CREATE USER ... PASSWORD`)
- [ ] Privilegiyalar berildi (`GRANT`, PG 15+ da schema ham)
- [ ] DB faqat `localhost` tinglayotgani tasdiqlandi (`bind-address`/`listen_addresses`)
- [ ] DB porti (5432/3306) tashqariga OCHIQ EMASligi tekshirildi
- [ ] App `.env`/connection string `localhost` orqali ulandi
- [ ] `.env` Git'da emas, fayl huquqi to'g'ri
- [ ] Qo'lda backup (`pg_dump`/`mysqldump`) sinab ko'rildi
- [ ] Avtomatik backup (cron) sozlandi
- [ ] Backup tashqi joyga (Cloud/boshqa server) ko'chirilyapti
- [ ] Restore sinab ko'rildi (ishlaydigan backup ekani tasdiqlandi)
- [ ] Migration'dan oldin backup olish odat qilingan
- [ ] Disk holati (`df -h`) muntazam kuzatilyapti
- [ ] Remote kerak bo'lsa — SSH tunnel ishlatilyapti (internetga ochish emas)

---

← [Deployment bo'limiga qaytish](./README.md)
