# SSH va Tunneling

Ushbu qo'llanmada SSH orqali xavfsiz masofaviy ulanish va **tunneling** (port forwarding) texnikalari batafsil yoritiladi. Maqsad — sizga masofaviy serverlar, ichki tarmoqlar (private network) va NAT/firewall orqasidagi mashinalar bilan ishlashning amaliy usullarini o'rgatish. Barcha misollar 2026-yilgi Linux/Ubuntu muhitida sinovdan o'tkazilgan buyruqlar bilan berilgan.

Tunneling — bu tarmoq trafigini boshqa bir protokol yoki shifrlangan kanal ichiga "o'rab" (encapsulate) yuborish. SSH tunneling esa shu trafikni SSH ulanishining shifrlangan kanali orqali o'tkazadi. Bu bilan siz ochiq bo'lmagan portlarga xavfsiz kirasiz, ichki xizmatlarni sinovdan o'tkazasiz va butun trafikni himoyalaysiz.

## Mundarija

- [SSH nima (qisqa takror)](#ssh-nima-qisqa-takror)
- [Tunneling nima va nega kerak](#tunneling-nima-va-nega-kerak)
- [Port forwarding uch turi](#port-forwarding-uch-turi)
  - [Local port forwarding (ssh -L)](#local-port-forwarding-ssh--l)
  - [Remote port forwarding (ssh -R)](#remote-port-forwarding-ssh--r)
  - [Dynamic port forwarding (ssh -D, SOCKS proxy)](#dynamic-port-forwarding-ssh--d-socks-proxy)
- [Jump host / Bastion host (ssh -J, ProxyJump)](#jump-host--bastion-host-ssh--j-proxyjump)
- [Reverse SSH tunnel (NAT orqasidagi mashina)](#reverse-ssh-tunnel-nat-orqasidagi-mashina)
- [SSH config (~/.ssh/config) tunnellar uchun](#ssh-config-sshconfig-tunnellar-uchun)
- [Autossh — tunnelni doim tirik saqlash](#autossh--tunnelni-doim-tirik-saqlash)
- [SSH tunnel vs VPN farqi](#ssh-tunnel-vs-vpn-farqi)
- [ngrok va Cloudflare Tunnel](#ngrok-va-cloudflare-tunnel)
- [SCP / SFTP / rsync orqali fayl uzatish](#scp--sftp--rsync-orqali-fayl-uzatish)
- [X11 forwarding (ssh -X)](#x11-forwarding-ssh--x)
- [Xavfsizlik](#xavfsizlik)
- [Savol-javob (Q&A)](#savol-javob-qa)
- [Masalalar](#masalalar)

---

## SSH nima (qisqa takror)

**SSH** (Secure Shell) — masofaviy mashinaga shifrlangan kanal orqali ulanib, buyruqlar bajarish uchun protokol. Standart port — `22`. Odatda parol yoki (ancha xavfsizroq) SSH kalitlar (public/private key) orqali autentifikatsiya qilinadi.

```bash
# Oddiy ulanish
ssh user@server.example.com

# Boshqa portga ulanish
ssh -p 2222 user@server.example.com

# Kalit fayl bilan ulanish
ssh -i ~/.ssh/id_ed25519 user@server.example.com
```

**💡 Tushuncha:** SSH nafaqat "masofadan terminal", balki **umumiy shifrlangan transport** hamdir. Aynan shu transport orqali biz istalgan TCP trafikni "tunnel" qilib o'tkaza olamiz — bu tunnelingning asosi.

SSH kalit yaratish (agar hali yo'q bo'lsa):

```bash
ssh-keygen -t ed25519 -C "sizning-emailingiz@example.com"
# Public kalitni serverga ko'chirish
ssh-copy-id user@server.example.com
```

---

## Tunneling nima va nega kerak

**Tunneling** — bir tarmoq ulanishini boshqa ulanish ichiga joylab yuborish. SSH tunnelingda odatiy (shifrlanmagan) TCP trafik SSH ning shifrlangan kanali ichidan o'tadi.

Nega kerak:

1. **Xavfsizlik** — shifrlanmagan protokol (masalan, eski DB ulanishi) SSH ichida shifrlangan holda o'tadi.
2. **Firewall/NAT ni chetlab o'tish** — faqat `22`-port ochiq bo'lsa ham, uning ichidan boshqa xizmatlarga kirasiz.
3. **Ichki resurslarga kirish** — jamoat internetiga chiqmaydigan ichki DB, admin panel, dashboardlarga.
4. **Test/debug** — lokal xizmatni vaqtincha tashqariga ochish (webhook, demo).

```text
   Shifrlanmagan trafik              Shifrlanmagan trafik
   ┌──────────┐   ╔═══════════════════════╗   ┌──────────┐
   │ Ilovangiz│──▶║  SSH shifrlangan kanal ║──▶│  Xizmat  │
   └──────────┘   ╚═══════════════════════╝   └──────────┘
    localhost         (Internet orqali)          server
```

**⚠️ Ehtiyot bo'l:** Tunnel — bu "sehrli devor" emas. Trafik faqat SSH client va SSH server orasida shifrlanadi. SSH serverdan yakuniy xizmatgacha bo'lgan qism (masalan, server ichidagi `localhost:5432`) shifrlanmasligi mumkin. Odatda bu ichki tarmoq bo'lgani uchun xavfsiz, lekin buni bilib turing.

---

## Port forwarding uch turi

SSH da uch xil port forwarding bor. Har birini alohida ko'rib chiqamiz. Farqni eslab qolishning oson yo'li:

```text
-L  (Local)   : lokal port  → masofaviy xizmatga   (ichkariga kirish)
-R  (Remote)  : masofaviy port → lokal xizmatga     (tashqariga ochish)
-D  (Dynamic) : lokal SOCKS proxy → istalgan joyga  (brauzer proxy)
```

**💡 Tushuncha:** `-L` va `-R` ni chalkashtirmaslik uchun: "harf trafik QAYSI TOMONDA boshlanishini bildiradi". `-L` — trafik **Lokal** mashinada boshlanadi. `-R` — trafik **Remote** (masofaviy) serverda boshlanadi.

### Local port forwarding (ssh -L)

**Maqsad:** masofaviy (yoki masofaviy server ko'ra oladigan) xizmatga o'zingizning lokal portingiz orqali kirish.

**Klassik use-case:** masofaviy serverda ishlayotgan, lekin tashqariga ochilmagan PostgreSQL (`localhost:5432`) ga o'z kompyuteringizdan ulanish.

```text
   Lokal mashina (siz)                       Masofaviy server
   ┌───────────────────┐                     ┌──────────────────┐
   │ psql localhost:5432│                     │                  │
   │        │           │  SSH (port 22)      │  PostgreSQL      │
   │        ▼           │═════════════════════▶  localhost:5432  │
   │  localhost:5432 ───┼─────────────────────┤                  │
   │  (tunnel kirishi)  │                     │                  │
   └───────────────────┘                     └──────────────────┘
```

Buyruq sintaksisi:

```bash
ssh -L [lokal_manzil:]lokal_port:maqsad_host:maqsad_port user@ssh_server
```

Real misol — masofaviy DB ga lokal `5432` orqali kirish:

```bash
# localhost:5432 (mahalliy) -> server ichidagi localhost:5432 (DB)
ssh -L 5432:localhost:5432 user@db-server.example.com

# Endi boshqa terminalda:
psql -h localhost -p 5432 -U dbuser mydb
```

**💡 Tushuncha:** `maqsad_host` — bu **SSH server nuqtai nazaridan** manzil. `localhost:5432` degani "SSH serverning o'zidagi 5432". Agar DB alohida mashinada bo'lsa (SSH server u bilan bir tarmoqda), `-L 5432:10.0.0.5:5432` deb yozasiz.

Faqat lokal ulanishlarga ruxsat berilishi uchun (default xavfsiz) `-N` (buyruq bajarmaslik) va `-f` (fonga o'tkazish) ni qo'shish qulay:

```bash
ssh -N -f -L 8080:localhost:80 user@server.example.com
# Endi brauzerda http://localhost:8080 -> serverdagi 80-port
```

### Remote port forwarding (ssh -R)

**Maqsad:** o'zingizning (yoki lokal tarmoqdagi) xizmatingizni masofaviy serverga/internetga ochish. Trafik masofaviy serverning portida boshlanib, sizga qaytib keladi.

**Klassik use-case:** NAT orqasidagi kompyuteringizda ishlayotgan veb-serverni umumiy VPS orqali hamkasbga ko'rsatish.

```text
   Lokal mashina (siz)                       Masofaviy server (VPS)
   ┌───────────────────┐                     ┌──────────────────┐
   │  Web app           │                     │  0.0.0.0:8080    │◀── Hamkasb
   │  localhost:3000 ◀──┼═════════════════════┤  (tunnel chiqishi)│    ulanadi
   │                    │  SSH (port 22)      │                  │
   └───────────────────┘                     └──────────────────┘
```

Buyruq sintaksisi:

```bash
ssh -R [masofaviy_manzil:]masofaviy_port:lokal_host:lokal_port user@ssh_server
```

Real misol:

```bash
# VPS dagi 8080 -> mening localhost:3000 ga yo'naltiriladi
ssh -R 8080:localhost:3000 user@vps.example.com

# Endi kimdir vps.example.com:8080 ga kirsa, sizning localhost:3000 ni ko'radi
```

**⚠️ Ehtiyot bo'l:** Default holatda `-R` tunnel faqat serverning `localhost` iga bog'lanadi (tashqaridan ko'rinmaydi). Uni butun internetga ochish uchun serverning `/etc/ssh/sshd_config` faylida `GatewayPorts yes` (yoki `clientspecified`) bo'lishi kerak, va buyruqda `-R 0.0.0.0:8080:...` deb yozasiz. Buni faqat ishonchli serverda va ehtiyotkorlik bilan yoqing.

### Dynamic port forwarding (ssh -D, SOCKS proxy)

**Maqsad:** SSH server orqali **istalgan** manzilga chiqadigan SOCKS proxy yaratish. Bitta tunnel — cheksiz maqsadlar. Ayniqsa brauzer trafigini SSH orqali o'tkazish uchun ideal.

**Klassik use-case:** butun brauzer trafigingizni masofaviy server orqali o'tkazib, ichki resurslarga kirish yoki o'z IP ingizni yashirish.

```text
   Lokal mashina (siz)                       Masofaviy server
   ┌───────────────────┐                     ┌──────────────────┐
   │  Brauzer           │                     │                  │
   │  SOCKS proxy:1080 ─┼═════════════════════▶  Internet /      │
   │  (istalgan manzil) │  SSH (port 22)      │  ichki tarmoq    │
   └───────────────────┘                     └──────────────────┘
```

Buyruq:

```bash
# Lokal 1080-portda SOCKS5 proxy ochiladi
ssh -N -D 1080 user@server.example.com
```

Endi brauzer yoki dasturni SOCKS5 proxy `localhost:1080` ga sozlang. Masalan `curl` bilan test:

```bash
curl --socks5-hostname localhost:1080 https://ifconfig.me
# Serverning IP manzilini qaytaradi, sizniki emas
```

**💡 Tushuncha:** `--socks5-hostname` (nafaqat `--socks5`) muhim — u DNS so'rovlarini ham tunnel ichida hal qiladi, ya'ni DNS leak bo'lmaydi va ichki domenlar to'g'ri ishlaydi.

---

## Jump host / Bastion host (ssh -J, ProxyJump)

**Bastion host** (yoki jump host) — ichki tarmoqqa yagona kirish nuqtasi bo'lgan oraliq server. Ichki mashinalar to'g'ridan-to'g'ri internetdan ochilmaydi; siz avval bastionga, keyin uning orqali ichkariga ulanasiz.

```text
   Siz ────▶ Bastion host ────▶ Ichki server
   (Internet)  (public IP)      (10.0.0.7, private)
```

Eng oson usul — `-J` (ProxyJump):

```bash
# Bir vaqtning o'zida bastion orqali ichki serverga
ssh -J user@bastion.example.com user@10.0.0.7
```

Bir nechta jump orqali (zanjir):

```bash
ssh -J user@bastion1,user@bastion2 user@10.0.0.7
```

**💡 Tushuncha:** `-J` ichida trafik bastionda **ochilmaydi** — SSH client bastion orqali tunnel qurib, yakuniy serverga uchi-uchigacha (end-to-end) shifrlangan alohida SSH sessiya ochadi. Ya'ni bastion parolingizni yoki sessiyangizni ko'ra olmaydi.

Config bilan yanada qulay (pastda `~/.ssh/config` bo'limiga qarang).

---

## Reverse SSH tunnel (NAT orqasidagi mashina)

Muammo: uydagi yoki ofisdagi kompyuteringiz NAT/firewall orqasida — unga tashqaridan to'g'ridan-to'g'ri ulanib bo'lmaydi (public IP yo'q). Yechim: mashina **o'zi** umumiy serverga (VPS) ulanib, teskari (reverse) tunnel ochadi.

```text
   NAT orqasidagi mashina           VPS (public IP)          Siz (istalgan joy)
   ┌─────────────────┐              ┌──────────────┐         ┌──────────┐
   │  SSH server:22  │◀═════════════┤ localhost:2222│◀────────┤  ssh -p  │
   │                 │  reverse     │              │         │  2222    │
   │  (o'zi ulanadi) │──────────────▶              │         └──────────┘
   └─────────────────┘              └──────────────┘
```

NAT orqasidagi mashinada bajariladi:

```bash
# VPS ning 2222-porti -> shu mashinaning localhost:22 (SSH)
ssh -N -R 2222:localhost:22 user@vps.example.com
```

Endi istalgan joydan VPS orqali o'sha mashinaga ulanish:

```bash
ssh -p 2222 localuser@vps.example.com
# yoki VPS ichidan: ssh -p 2222 localuser@localhost
```

**⚠️ Ehtiyot bo'l:** Reverse tunnel doimiy ishlashi uchun uni **autossh** (pastda) bilan boshqarish yoki `systemd` service qilib qo'yish kerak. Oddiy `ssh -R` ulanish uzilsa qaytadan tiklanmaydi.

---

## SSH config (~/.ssh/config) tunnellar uchun

Uzun buyruqlarni har safar terish o'rniga `~/.ssh/config` da profil yozing.

```ini
# ~/.ssh/config

# Oddiy server
Host web
    HostName web.example.com
    User deploy
    Port 22
    IdentityFile ~/.ssh/id_ed25519

# Bastion orqali ichki serverga (ProxyJump)
Host internal
    HostName 10.0.0.7
    User admin
    ProxyJump bastion

Host bastion
    HostName bastion.example.com
    User admin

# Doimiy local forward (DB tunnel avtomatik ochiladi)
Host db-tunnel
    HostName db-server.example.com
    User dbuser
    LocalForward 5432 localhost:5432

# Tunnelni tirik saqlash sozlamalari
Host *
    ServerAliveInterval 30
    ServerAliveCountMax 3
```

Endi shunchaki:

```bash
ssh internal        # bastion orqali avtomatik
ssh -N db-tunnel    # DB tunnel ochiladi
```

**💡 Tushuncha:** `ServerAliveInterval 30` — har 30 soniyada "tirik" signali yuboradi. Bu NAT/firewall tunnelni "bo'sh" deb yopib qo'yishining oldini oladi.

---

## Autossh — tunnelni doim tirik saqlash

`autossh` — SSH tunnel uzilsa, uni avtomatik qayta ochadigan vosita. Reverse tunnel va doimiy forward uchun juda foydali.

```bash
sudo apt update && sudo apt install -y autossh
```

Doimiy reverse tunnel:

```bash
autossh -M 0 -N -R 2222:localhost:22 user@vps.example.com
```

- `-M 0` — autossh ning o'z monitoring portini o'chiradi (SSH ning `ServerAliveInterval` iga tayanadi).
- `-N` — masofaviy buyruq bajarmaslik (faqat tunnel).

Uni `systemd` service qilib, mashina yoqilganda avtomatik ishga tushirish:

```ini
# /etc/systemd/system/reverse-tunnel.service
[Unit]
Description=Reverse SSH tunnel
After=network-online.target
Wants=network-online.target

[Service]
User=tunnel
Environment=AUTOSSH_GATETIME=0
ExecStart=/usr/bin/autossh -M 0 -N -o "ServerAliveInterval 30" -o "ServerAliveCountMax 3" -R 2222:localhost:22 user@vps.example.com
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl enable --now reverse-tunnel.service
```

**⚠️ Ehtiyot bo'l:** `systemd` bilan ishlaganda SSH kalit parolsiz (yoki `ssh-agent` orqali) bo'lishi kerak, aks holda service parol so'rab qotib qoladi. Kalitni faqat shu tunnel uchun cheklangan huquqlar bilan yarating.

---

## SSH tunnel vs VPN farqi

| Xususiyat | SSH tunnel | VPN |
|---|---|---|
| Qamrov | Bitta port / SOCKS (aniq trafik) | Butun mashina trafigi (odatda) |
| Sozlash | Tez, bir buyruq | Server + client config kerak |
| Ish darajasi | Ilova/TCP darajasi | Tarmoq (IP) darajasi |
| UDP qo'llab-quvvatlash | Yo'q (asosan TCP) | Ha |
| Use-case | Bitta xizmat, debug, admin | Doimiy xavfsiz tarmoq |

**💡 Tushuncha:** Tez, vaqtinchalik, bitta xizmatga ulanish kerak bo'lsa — SSH tunnel. Doimiy, butun tarmoq darajasidagi xavfsiz ulanish kerak bo'lsa — VPN (`02-vpn.md` ga qarang).

---

## ngrok va Cloudflare Tunnel

Lokal serveringizni internetga **tez** ochish uchun tayyor xizmatlar. Webhook test qilish, demo ko'rsatish uchun ideal — o'z VPS ingiz shart emas.

### ngrok

```bash
# O'rnatish (Ubuntu misolida, snap yoki apt orqali)
# Ro'yxatdan o'tib authtoken oling: https://ngrok.com
ngrok config add-authtoken <SIZNING_TOKEN>

# Lokal 3000-portni internetga ochish
ngrok http 3000
```

ngrok sizga `https://xxxx.ngrok-free.app` kabi umumiy URL beradi va uni lokal `3000` ga bog'laydi. Webhook (masalan, Stripe, Telegram bot) test qilishda foydali.

### Cloudflare Tunnel

```bash
# cloudflared o'rnatish
# Tez (nomsiz) tunnel:
cloudflared tunnel --url http://localhost:3000
```

Bu ham sizga `https://<random>.trycloudflare.com` URL beradi. Doimiy va domen bilan ishlash uchun `cloudflared tunnel login` va nomlangan tunnel sozlanadi.

**💡 Tushuncha:** ngrok/Cloudflare Tunnel — bu aslida **boshqarilgan reverse tunnel** xizmati. Lokal agent ular serveriga ulanadi va internet trafigini sizga qaytaradi. Shuning uchun NAT/firewall orqasida ham ishlaydi.

**⚠️ Ehtiyot bo'l:** Bepul ngrok URL i tasodifiy va ochiq — hech kim topmaydi deb o'ylamang. Muhim xizmatlarda autentifikatsiya qo'ying (`ngrok http --basic-auth ...`) yoki tez o'chiring.

---

## SCP / SFTP / rsync orqali fayl uzatish

SSH kanali orqali xavfsiz fayl uzatish.

```bash
# scp — oddiy nusxa ko'chirish
scp fayl.txt user@server:/home/user/          # yuklash
scp user@server:/var/log/app.log ./           # yuklab olish
scp -r ./papka user@server:/opt/app/          # katalog (recursive)

# sftp — interaktiv sessiya
sftp user@server
# sftp> put fayl.txt
# sftp> get remote.txt

# rsync — samarali (faqat o'zgargan qism), katta papkalar uchun eng yaxshi
rsync -avz --progress ./project/ user@server:/opt/project/
rsync -avz -e "ssh -p 2222" ./data/ user@server:/data/   # boshqa port
```

**💡 Tushuncha:** `rsync` katta yoki tez-tez yangilanadigan papkalar uchun eng yaxshisi — u faqat o'zgarishlarni yuboradi (delta transfer), shu bilan vaqt va trafikni tejaydi. `-a` (arxiv rejim) huquq va vaqt belgilarini saqlaydi, `-z` siqadi.

---

## X11 forwarding (ssh -X)

Masofaviy serverdagi grafik (GUI) ilovani o'z ekraningizda ko'rsatish.

```bash
ssh -X user@server.example.com
# Endi serverda:
xclock   # yoki firefox, gedit — o'z ekraningizda ochiladi
```

`-X` o'rniga `-Y` (trusted) ba'zi ilovalar uchun kerak bo'lishi mumkin, lekin u kamroq xavfsiz.

**⚠️ Ehtiyot bo'l:** X11 forwarding sekin va xavfsizlik jihatidan nozik. Faqat ishonchli serverlarda va zarur bo'lgandagina foydalaning. Serverda `X11Forwarding yes` yoqilgan bo'lishi kerak.

---

## Xavfsizlik

SSH va tunneling kuchli, lekin noto'g'ri sozlansa xavf tug'diradi:

1. **Parol emas, kalitlardan foydalaning.** `PasswordAuthentication no` qo'ying.
2. **root bilan to'g'ridan-to'g'ri kirmang.** `PermitRootLogin no`.
3. **Fail2ban** o'rnating — brute-force urinishlarini bloklaydi.
4. **Portni o'zgartirish** (`22` dan boshqasiga) shovqinni kamaytiradi, lekin haqiqiy himoya emas.
5. **`GatewayPorts` va `AllowTcpForwarding`** ni ehtiyotkorlik bilan yoqing — tunneling imkoniyati suiiste'mol qilinishi mumkin.
6. **Bastion host** — ichki tarmoqqa yagona, nazorat qilinadigan kirish nuqtasi qiling.
7. **Kalitlarni parol (passphrase) bilan** himoyalang va `ssh-agent` dan foydalaning.

```ini
# /etc/ssh/sshd_config (server tomonida) — tavsiya etilgan minimal
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
X11Forwarding no
AllowTcpForwarding yes   # tunneling kerak bo'lsa
```

**💡 Tushuncha:** Xavfsizlik — qatlamlar (defense in depth). Bitta chora yetarli emas: kalitlar + fail2ban + minimal huquqlar + bastion birga ishlaganda haqiqiy himoya hosil bo'ladi.

---

## Savol-javob (Q&A)

### ❓ `ssh -L` va `ssh -R` orasidagi asosiy farq nima?

**✅ Javob:** `-L` (Local) — trafik **sizning** mashinangizda boshlanib, SSH orqali masofaviy xizmatga boradi (masofaviy DB ga kirasiz). `-R` (Remote) — trafik **masofaviy serverda** boshlanib, sizning lokal xizmatingizga qaytadi (lokal xizmatni tashqariga ochasiz). Harf trafik qayerda boshlanishini bildiradi.

### ❓ Tunnelning ma'nosini eslab qolishning oson yo'li bormi?

**✅ Javob:** Ha. `-L` = "ichkariga tortish" (pull in), `-R` = "tashqariga surish" (push out), `-D` = "dinamik SOCKS proxy — istalgan joyga". Buyruqning birinchi soni doim **portni ochadigan tomondagi** port.

### ❓ Nega `ssh -N -f` ni tunnellarda ishlataman?

**✅ Javob:** `-N` — masofaviy shell/buyruq ochmaydi (faqat tunnel kerak). `-f` — SSH ni fonga (background) o'tkazadi, terminal band bo'lmaydi. Ikkalasi birga sof tunnel jarayoni beradi.

### ❓ `-D` (SOCKS proxy) ni brauzerda qanday ishlataman?

**✅ Javob:** `ssh -N -D 1080 user@server` ishga tushiring, so'ng brauzer proxy sozlamalarida SOCKS5 host `localhost`, port `1080` deb belgilang. Firefox da "Proxy DNS when using SOCKS v5" ni ham yoqing — DNS ham tunneldan o'tadi.

### ❓ Reverse tunnel nima uchun kerak?

**✅ Javob:** Mashinangiz NAT/firewall orqasida bo'lib, public IP i bo'lmaganda. Mashina o'zi umumiy serverga ulanib, teskari tunnel ochadi va shu orqali tashqaridan unga ulanish mumkin bo'ladi.

### ❓ Reverse tunnelim tez-tez uziladi. Nima qilay?

**✅ Javob:** `autossh -M 0 -N -R ...` dan foydalaning va `ServerAliveInterval 30`, `ServerAliveCountMax 3` qo'shing. Doimiylik uchun `systemd` service qilib, `Restart=always` bilan sozlang.

### ❓ `GatewayPorts` nima va qachon kerak?

**✅ Javob:** Default holda `-R` tunnel faqat serverning `localhost` iga bog'lanadi. Tunnelni serverning barcha interfeyslarida (tashqaridan ko'rinadigan) ochish uchun serverda `GatewayPorts yes` (yoki `clientspecified`) kerak. Ehtiyot bo'ling — bu xizmatni internetga ochadi.

### ❓ Bastion host orqali fayl ham uzata olamanmi?

**✅ Javob:** Ha. `scp -o ProxyJump=user@bastion fayl.txt user@10.0.0.7:/tmp/` yoki `~/.ssh/config` da `ProxyJump` sozlangan bo'lsa oddiy `scp fayl.txt internal:/tmp/`.

### ❓ ngrok va Cloudflare Tunnel orasida qaysi biri yaxshi?

**✅ Javob:** Tez, bir martalik test uchun ikkalasi ham yaxshi. Cloudflare Tunnel bepul, doimiy va o'z domeningiz bilan ishlash uchun kuchli. ngrok qulay interfeys, trafik inspektori (webhook debug) bilan ajralib turadi. Ehtiyoj bo'yicha tanlang.

### ❓ SSH tunnel VPN o'rnini bosa oladimi?

**✅ Javob:** Ba'zi hollarda ha (`-D` SOCKS proxy butun brauzer trafigini o'tkaza oladi), lekin to'liq emas. SSH asosan TCP bilan ishlaydi, UDP ni yo'q. Butun mashina/tarmoq darajasidagi doimiy xavfsizlik uchun VPN to'g'riroq.

### ❓ `ssh -J` va eski `ProxyCommand` orasidagi farq nima?

**✅ Javob:** `-J` (ProxyJump) — zamonaviy, oddiy, xavfsizroq usul (uchi-uchigacha shifrlash saqlanadi). `ProxyCommand netcat` — eski, qo'lda usul. 2026-yilda deyarli har doim `-J` yoki config dagi `ProxyJump` dan foydalaning.

### ❓ Tunnel orqali o'tayotgan trafik to'liq shifrlanganmi?

**✅ Javob:** SSH client va SSH server orasida — ha, to'liq shifrlangan. Ammo SSH serverdan yakuniy xizmatgacha (masalan, server ichidagi `localhost:5432` gacha) shifrlanmasligi mumkin. Bu odatda ishonchli ichki tarmoq bo'lgani uchun xavfsiz.

### ❓ Bir buyruqda bir nechta portni forward qila olamanmi?

**✅ Javob:** Ha. `-L` yoki `-R` ni bir necha marta yozing: `ssh -L 5432:localhost:5432 -L 8080:localhost:80 user@server`. Yoki `~/.ssh/config` da bir nechta `LocalForward` qatori qo'shing.

### ❓ `autossh` da `-M 0` nega ishlatiladi?

**✅ Javob:** `-M` autossh ning monitoring portini belgilaydi (uzilishni aniqlash uchun). `-M 0` uni o'chirib, o'rniga SSH ning ancha ishonchli `ServerAliveInterval` mexanizmiga tayanadi. 2026-yilda tavsiya etilgan usul aynan shu.

### ❓ X11 forwarding ishlamayapti, sababi nimada?

**✅ Javob:** Serverda `X11Forwarding yes` yoqilganini, lokal mashinada X server (Linux da odatda bor, Windows da VcXsrv/X410) ishga tushganini va `xauth` o'rnatilganini tekshiring. `ssh -X` o'rniga `ssh -Y` ni ham sinab ko'ring.

---

## Masalalar

> Yechimlar: [solutions/remote-access/01-ssh-tunneling.md](../solutions/remote-access/01-ssh-tunneling.md)

1. **Local forward:** Masofaviy serverda `127.0.0.1:6379` da ishlayotgan Redis bor. Uni o'z mashinangizdan `localhost:6380` orqali ochadigan `ssh -L` buyrug'ini yozing va `redis-cli` bilan ulanish buyrug'ini keltiring.

2. **Remote forward:** Lokal `localhost:8000` da ishlayotgan veb-serveringizni VPS (`vps.example.com`) ning `9000`-porti orqali internetga ochmoqchisiz. Kerakli `ssh -R` buyrug'ini va server tarafida qanday `sshd_config` sozlamasi zarurligini yozing.

3. **SOCKS proxy:** `ssh -D` yordamida SOCKS5 proxy oching va `curl` bilan proxy orqali chiqayotgan IP ni tekshiring. DNS leak bo'lmasligi uchun qaysi flag muhim?

4. **Jump host:** `bastion.corp.com` orqali ichki `10.0.5.20` serverga ulanishni (a) bir qatorli `-J` buyrug'i bilan, (b) `~/.ssh/config` profili bilan yozing.

5. **Reverse tunnel + autossh:** NAT orqasidagi mashinaning SSH portini VPS ning `2222`-porti orqali doimiy ochiq saqlaydigan `autossh` buyrug'ini va uni `systemd` service qiladigan minimal unit faylini yozing.

6. **SSH config:** Bitta `Host db` profilida quyidagilarni jamlang: HostName, User, IdentityFile, `LocalForward 5432 localhost:5432`, va tunnelni tirik saqlash sozlamalari.

7. **rsync:** Lokal `./backup/` papkasini masofaviy `2222`-portdagi serverning `/srv/backup/` iga faqat o'zgarishlarni yuboradigan, siqilgan va progress ko'rsatadigan `rsync` buyrug'ini yozing.

8. **ngrok/Cloudflare:** Lokal `3000`-portda ishlayotgan ilovani (a) ngrok, (b) Cloudflare Tunnel yordamida internetga ochadigan buyruqlarni yozing va ular qanday farq qilishini bir jumlada tushuntiring.

9. **Xavfsizlik:** Bir serverni tunneling uchun ishlatmoqchisiz, lekin foydalanuvchiga faqat port forwarding beradigan, shell bermaydigan cheklangan SSH kalit sozlamasini o'ylab toping (maslahat: `authorized_keys` dagi `no-pty`, `command=` opsiyalari).

10. **Diagnostika:** `ssh -L 5432:localhost:5432 user@server` ishlamayapti — `psql` "connection refused" beryapti. Muammoning 3 ta ehtimoliy sababini va tekshirish usulini yozing (maslahat: `-v` flag, server tarafda xizmat ishlayaptimi, `AllowTcpForwarding`).

---

← [Remote Access bo'limiga qaytish](./README.md)
