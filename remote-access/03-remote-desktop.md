# Remote Desktop (RDP, VNC, X2Go)

Serverga SSH orqali ulanganda siz faqat **terminal** (matnli buyruq qatori) olasiz. Lekin ba'zan sizga masofadagi mashinaning **to'liq grafik ish stoli** kerak bo'ladi — sichqoncha, oynalar, brauzer, GUI dasturlar. Ana shu yerda **Remote Desktop** texnologiyalari ishga tushadi: RDP, VNC, X2Go va NoMachine.

Bu hujjatda ular qanday ishlashini, farqlarini, qadama-qadam sozlashni va — eng muhimi — **xavfsizligini** o'rganamiz. Chunki noto'g'ri ochilgan RDP porti dunyodagi eng ko'p buziladigan (hacked) narsalardan biridir.

## Mundarija

- [Remote Desktop nima va SSH bilan farqi](#remote-desktop-nima-va-ssh-bilan-farqi)
- [Qanday ishlaydi (ekran uzatish)](#qanday-ishlaydi-ekran-uzatish)
- [RDP — Windows'ning yechimi](#rdp--windowsning-yechimi)
- [VNC — platformadan mustaqil](#vnc--platformadan-mustaqil)
- [X2Go — Linux uchun tez yechim](#x2go--linux-uchun-tez-yechim)
- [NoMachine — tez va oson alternativa](#nomachine--tez-va-oson-alternativa)
- [Taqqoslash jadvali](#taqqoslash-jadvali)
- [Umumiy xavfsizlik qoidalari](#umumiy-xavfsizlik-qoidalari)
- [Browser-based: Apache Guacamole](#browser-based-apache-guacamole)
- [Savol-javob (Q&A)](#savol-javob-qa)
- [Masalalar](#masalalar)

---

## Remote Desktop nima va SSH bilan farqi

**Remote Desktop** — bu masofadagi kompyuterning grafik ekranini (desktop) o'z ekraningizda ko'rish va boshqarish texnologiyasi. Siz sichqoncha va klaviatura harakatlarini masofadagi mashinaga yuborasiz, u esa o'z ekranining rasmini sizga qaytaradi.

**💡 Tushuncha:** SSH = **CLI** (Command Line Interface, matnli terminal). Remote Desktop = **GUI** (Graphical User Interface, grafik ish stoli). SSH — "telefon orqali gaplashish", Remote Desktop — "video qo'ng'iroq".

```text
   SSH (terminal)                    Remote Desktop (GUI)
 ┌────────────────┐               ┌────────────────────────┐
 │ $ ls -la       │               │  [Firefox] [Files]     │
 │ $ sudo apt ... │               │   ___________________  │
 │ $ vim conf     │               │  |  brauzer oynasi   | │
 │ _              │               │  |___________________| │
 └────────────────┘               └────────────────────────┘
   Yengil, tez                      Og'ir, ko'p trafik
   Faqat matn                       To'liq grafik desktop
```

**Qachon GUI (Remote Desktop) kerak:**

- Grafik dastur bilan ishlash kerak (dizayn dasturi, IDE'ning GUI versiyasi, brauzer).
- Windows serverni boshqarish (ko'p Windows ishlari GUI'siz qilinmaydi).
- Foydalanuvchiga masofadan yordam berish (uning ekranini ko'rish).
- GUI o'rnatish/test qilish talab qiladigan ishlar.

**Qachon SSH yetarli (GUI shart emas):**

- Server sozlash, konfiguratsiya faylini tahrirlash.
- Dastur deploy qilish, log ko'rish, servis restart.
- Deyarli barcha DevOps/backend ishlari.

**⚠️ Ehtiyot bo'l:** GUI ancha ko'p tarmoq trafigi va resurs talab qiladi. Agar ishni SSH orqali qila olsangiz — SSH'ni tanlang. GUI'ni faqat haqiqatan kerak bo'lganda ishlating.

---

## Qanday ishlaydi (ekran uzatish)

Barcha remote desktop protokollari ikki narsani qiladi:

1. **Ekranni uzatish** (server → client): masofadagi mashina ekranini rasm sifatida yuboradi.
2. **Input yuborish** (client → server): sizning sichqoncha/klaviatura harakatlaringiz masofadagi mashinaga boradi.

Lekin ekranni **qanday** uzatish — protokollar orasidagi asosiy farq:

```text
┌──────────────────────────────────────────────────────────────┐
│  USUL 1: Bitmap / Pixel uzatish (VNC — RFB protokoli)         │
│  Ekran = piksellar to'plami. O'zgargan qismlarni rasm         │
│  sifatida yuboradi. Sodda, lekin ko'p trafik, sekinroq.       │
├──────────────────────────────────────────────────────────────┤
│  USUL 2: Grafik komandalar (RDP, X2Go/NX)                     │
│  "Bu joyga to'rtburchak chiz", "bu matnni yoz" kabi           │
│  buyruqlar yuboradi. Kamroq trafik, tezroq, aqlliroq.         │
└──────────────────────────────────────────────────────────────┘
```

**💡 Tushuncha:** VNC "surat yuboradi" — har o'zgarishda ekran bo'lagini rasmga oladi. RDP va NX (X2Go asosi) "chizish buyrug'ini yuboradi" — bu ancha kam ma'lumot va sekin internetda ham tezroq ishlaydi. Shu sabab RDP va X2Go odatda VNC'dan tezroq.

Bundan tashqari zamonaviy protokollar **siqish** (compression), **kesh** (caching — o'zgarmagan qismni qayta yubormaslik) va **shifrlash** (encryption) qo'llaydi.

---

## RDP — Windows'ning yechimi

**RDP (Remote Desktop Protocol)** — Microsoft yaratgan, Windows'ga o'rnatilgan protokol. **Port: 3389** (TCP). U grafik komandalar asosida ishlaydi, shuning uchun juda **efficient** (samarali) va Windows ekaniga optimallashtirilgan.

### RDP ni Windows'da yoqish

Server tomonda (masofadan ulanmoqchi bo'lgan Windows mashinada):

```text
Settings → System → Remote Desktop → "Enable Remote Desktop" = ON
```

Yoki PowerShell orqali (administrator huquqi bilan):

```powershell
# RDP ni yoqish (registry)
Set-ItemProperty -Path 'HKLM:\System\CurrentControlSet\Control\Terminal Server' -Name "fDenyTSConnections" -Value 0

# Firewall'da RDP'ga ruxsat
Enable-NetFirewallRule -DisplayGroup "Remote Desktop"
```

**⚠️ Ehtiyot bo'l:** RDP faqat **Windows Pro / Enterprise / Server** versiyalarida server sifatida ishlaydi. **Windows Home** RDP serverini qo'llab-quvvatlamaydi (faqat client sifatida ulanadi).

### Windows'dan Windows'ga ulanish (mstsc)

Windows'da tayyor client bor — **`mstsc`** (Microsoft Terminal Services Client):

```text
Win+R → "mstsc" → Enter
```

Yoki to'g'ridan-to'g'ri:

```powershell
mstsc /v:192.168.1.50:3389
```

So'ng foydalanuvchi nomi va parolni kiritasiz — masofadagi Windows ish stoli ochiladi.

### Linux'dan Windows RDP'ga ulanish

Linux'da eng ko'p ishlatiladigan clientlar:

**1. Remmina** (GUI, qulay):

```bash
# Ubuntu/Debian
sudo apt install remmina remmina-plugin-rdp

# Ishga tushirish
remmina
# So'ng: New connection → Protocol: RDP → Server: IP → Ulanish
```

**2. FreeRDP (`xfreerdp`)** — buyruq qatoridan:

```bash
sudo apt install freerdp2-x11

# Ulanish
xfreerdp /v:192.168.1.50 /u:Administrator /p:'parol' /size:1440x900

# Tovush, clipboard va papka ulashish bilan
xfreerdp /v:192.168.1.50 /u:User /dynamic-resolution /clipboard /sound
```

**💡 Tushuncha:** `xfreerdp` sertifikat haqida ogohlantirsa, birinchi ulanishda `/cert:ignore` (yoki `/cert-ignore`) qo'shishingiz mumkin — lekin bu faqat ishonchli lokal tarmoqda.

### Linux'da RDP server: xRDP

Linux mashinaga **RDP orqali** ulanmoqchi bo'lsangiz (masalan Windows'dan Ubuntu'ga), **xRDP** o'rnatasiz:

```bash
sudo apt update
sudo apt install xrdp -y

# Servisni yoqish
sudo systemctl enable --now xrdp

# Holatini tekshirish
sudo systemctl status xrdp

# Firewall (agar ufw ishlatilsa)
sudo ufw allow 3389/tcp
```

Endi Windows'dan `mstsc` orqali Ubuntu IP'siga 3389 portida ulanasiz.

**⚠️ Ehtiyot bo'l:** xRDP ko'pincha desktop environment bilan konflikt qiladi (ayniqsa GNOME + Wayland). Agar qora ekran chiqsa, `/etc/gdm3/custom.conf` da `WaylandEnable=false` qilib, XFCE kabi yengil desktopni sinab ko'ring.

### RDP xavfsizligi — ENG MUHIM

**⚠️ Ehtiyot bo'l:** RDP portini (3389) **hech qachon** to'g'ridan-to'g'ri internetga ochmang! Internetdagi botlar doim 3389 portini skanerlab, parol tanlashga (brute-force) urinadi. RDP — ransomware hujumlarining №1 kirish nuqtasi.

To'g'ri yechim — RDP'ni **SSH tunnel** yoki **VPN** orqasiga yashirish:

```bash
# SSH tunnel: lokal 13389 portini masofadagi 3389 ga bog'lash
ssh -L 13389:localhost:3389 user@server.example.com

# Endi RDP client'da ulanish manzili:
#   localhost:13389
# (trafik SSH orqali shifrlangan holda o'tadi)
```

```text
[RDP client] → localhost:13389 → [SSH shifrlangan tunnel] → server:3389 → [RDP server]
```

---

## VNC — platformadan mustaqil

**VNC (Virtual Network Computing)** — **RFB (Remote Framebuffer)** protokoli asosidagi ochiq standart. Uning kuchi — **cross-platform**: har qanday OS'da server va client bor. Kamchiligi — u pixel/bitmap uzatadi, shuning uchun **sekinroq** va RDP/X2Go'dan ko'proq trafik yeydi.

Mashhur implementatsiyalar:

- **TigerVNC** — zamonaviy, tez, Linux'da ko'p ishlatiladi.
- **TightVNC** — Windows/Linux, siqishga urg'u.
- **RealVNC** — tijoriy, oson, cross-platform.

### VNC server o'rnatish (Linux, TigerVNC)

```bash
# O'rnatish
sudo apt install tigervnc-standalone-server tigervnc-common -y

# VNC parol o'rnatish
vncpasswd

# VNC serverni :1 displayda ishga tushirish (port = 5900 + 1 = 5901)
vncserver :1 -geometry 1440x900 -depth 24

# Ro'yxatni ko'rish
vncserver -list

# To'xtatish
vncserver -kill :1
```

**💡 Tushuncha:** VNC portlari `5900 + display raqami` formatida. Display `:1` → port **5901**, `:2` → **5902** va hokazo.

Qaysi desktop environment ishga tushishini `~/.vnc/xstartup` faylida sozlaysiz:

```bash
#!/bin/sh
unset SESSION_MANAGER
unset DBUS_SESSION_BUS_ADDRESS
startxfce4 &   # yengil XFCE desktop
```

### VNC client

- Linux: **Remmina** (VNC plugin bilan), **TigerVNC viewer** (`xvncviewer`).
- Windows: **TightVNC Viewer**, **RealVNC Viewer**.

```bash
# TigerVNC viewer bilan ulanish
xvncviewer 192.168.1.50:5901
```

### VNC ni SSH tunnel orqali xavfsizlash

**⚠️ Ehtiyot bo'l:** Klassik VNC trafigi **shifrlanmaydi** (yoki juda zaif shifrlaydi). Parollaringiz va ekraningiz ochiq ko'rinishi mumkin. Har doim **SSH tunnel** orqali ishlating:

```bash
# 5901 portini SSH tunnel orqali mahalliy 5901 ga bog'lash
ssh -L 5901:localhost:5901 user@server.example.com

# Endi VNC client'da ulanish manzili:
#   localhost:5901
```

Server tomonda VNC'ni faqat localhost'ga bog'lang (internetga ochilmasin):

```bash
vncserver :1 -localhost yes
```

---

## X2Go — Linux uchun tez yechim

**X2Go** — Linux desktopga masofadan ulanish uchun eng yaxshi yechimlardan biri. U **NX texnologiyasi** asosida qurilgan bo'lib, grafik komandalarni aqlli siqadi va keshlaydi. Natijada **sekin tarmoqda ham (masalan mobil internet) juda ravon** ishlaydi.

**💡 Tushuncha:** X2Go'ning eng katta ustunligi — u **SSH ustida** ishlaydi. Ya'ni siz alohida tunnel ochishingiz shart emas: X2Go allaqachon SSH orqali (port 22) shifrlangan holda ulanadi. Bu ham xavfsiz, ham qulay.

### X2Go server o'rnatish (Linux)

```bash
# Ubuntu/Debian
sudo apt update
sudo apt install x2goserver x2goserver-xsession -y

# Yengil desktop kerak (X2Go og'ir GNOME bilan yaxshi ishlamaydi)
sudo apt install xfce4 xfce4-goodies -y
```

X2Go server SSH'dan foydalanadi, shuning uchun mashinada SSH server ishlab turishi kerak:

```bash
sudo apt install openssh-server -y
sudo systemctl enable --now ssh
```

**⚠️ Ehtiyot bo'l:** X2Go GNOME va KDE'ning yangi versiyalari bilan muammoli. **XFCE**, **MATE** yoki **LXDE** kabi yengil desktoplar tavsiya etiladi.

### X2Go client o'rnatish va ulanish

Client (o'z kompyuteringizda):

```bash
# Linux
sudo apt install x2goclient -y

# Windows / macOS uchun x2go.org saytidan installer yuklab olinadi
```

Client'da yangi session yaratasiz:

```text
Session name : Ish serveri
Host         : server.example.com
Login        : user
SSH port     : 22
Session type : XFCE   (yoki MATE/LXDE)
```

So'ng "OK" → session'ni bosib ulanadi. SSH parol (yoki key) so'raladi, keyin masofadagi Linux desktop ochiladi.

### Session turlari

X2Go bir nechta session turini qo'llab-quvvatlaydi:

- **Full desktop** — to'liq desktop (XFCE, MATE, LXDE, KDE).
- **Single application** — faqat bitta dastur (masalan faqat Firefox) masofadan.
- **Shared desktop** — mavjud lokal sessiyaga qo'shilish (yordam ko'rsatish uchun).

**💡 Tushuncha:** X2Go sessiyalarni **uzib-ulash** (suspend/resume) qila oladi. Ulanishni uzsangiz ham dasturlaringiz serverda ochiq qoladi — keyin qaytib ulanganda o'sha joydan davom etasiz. Bu SSH'dagi `tmux` kabi, lekin GUI uchun.

---

## NoMachine — tez va oson alternativa

**NoMachine** — NX texnologiyasining asl mualliflari yaratgan tijoriy (shaxsiy foydalanish bepul) mahsulot. **Juda tez**, **cross-platform** (Windows, macOS, Linux) va o'rnatishi juda oson.

```bash
# nomachine.com saytidan .deb / .rpm / .exe / .dmg yuklab olinadi
sudo dpkg -i nomachine_*.deb
```

Ustunliklari:

- Server ham, client ham bir paketda — o'rnatib qo'yganingizda mashina avtomatik "ulanishga tayyor" bo'ladi.
- 4K, ko'p monitor, tovush, video oqim — hammasi ravon.
- Ulanish shifrlangan (o'z NX protokoli orqali).

**💡 Tushuncha:** Agar sizga "ishlaydigan, tez, minimal sozlash" GUI remote kerak bo'lsa va Linux ↔ Windows ↔ macOS aralashsa — NoMachine ko'pincha eng oson tanlov. X2Go — bepul, ochiq kodli muqobil.

---

## Taqqoslash jadvali

| Xususiyat | RDP | VNC | X2Go | NoMachine |
|-----------|-----|-----|------|-----------|
| **Protokol** | RDP (komandalar) | RFB (pixel/bitmap) | NX (komandalar) | NX (komandalar) |
| **Tezlik** | Tez | Sekin | Juda tez | Juda tez |
| **Sekin internetda** | Yaxshi | Yomon | A'lo | A'lo |
| **Platforma** | Windows (asosan) | Barcha | Linux server | Barcha |
| **Shifrlash** | Bor (TLS) | Yo'q (SSH kerak) | Bor (SSH ustida) | Bor (NX) |
| **Sozlash qulayligi** | Oson (Windows) | O'rta | O'rta | Juda oson |
| **Narx** | Bepul (Windows'da) | Bepul | Bepul (ochiq) | Bepul (shaxsiy) |
| **Session resume** | Bor | Cheklangan | Bor (a'lo) | Bor |
| **Qachon ishlatish** | Windows'ga ulanish | Universal, sodda | Linux GUI, sekin tarmoq | Universal, tez, oson |

**Qisqacha tanlov:**

- Windows serverga → **RDP**.
- Linux serverga, tez va SSH ustida → **X2Go**.
- Har qanday platforma, minimal sozlash, maksimal tezlik → **NoMachine**.
- Eng universal, sodda, lekin sekin → **VNC** (albatta SSH tunnel bilan).

---

## Umumiy xavfsizlik qoidalari

**⚠️ Ehtiyot bo'l:** Remote desktop portlari internetdagi eng ko'p hujumga uchraydigan nishonlardir. Quyidagi qoidalarni HAR DOIM bajaring:

1. **Hech qachon to'g'ridan-to'g'ri internetga ochmang.** RDP (3389), VNC (5900+) portlarini ochiq qoldirmang. `nmap`/Shodan botlari ularni bir necha daqiqada topadi.

2. **VPN yoki SSH tunnel orqasiga yashiring.** Avval VPN'ga (yoki SSH'ga) ulaning, keyin ichki tarmoqda RDP/VNC ishlating.

   ```bash
   # RDP uchun SSH tunnel namunasi
   ssh -L 13389:localhost:3389 user@server
   # keyin RDP client → localhost:13389
   ```

3. **Kuchli parol + 2FA.** Zaif parol brute-force'da bir necha soatda ochiladi. Iloji bo'lsa key-asosli autentifikatsiya.

4. **Standart portni o'zgartiring** (security-through-obscurity — asosiy himoya emas, lekin avtomatik skanerlarni kamaytiradi).

5. **Firewall bilan cheklang.** Faqat kerakli IP manzillardan (`allow from`) ruxsat bering.

   ```bash
   # Faqat bitta IP dan RDP ga ruxsat (ufw)
   sudo ufw allow from 203.0.113.10 to any port 3389
   ```

6. **Account lockout / fail2ban.** Bir necha marta xato parol → IP'ni bloklash.

7. **Yangilanib turing.** RDP/VNC serverlaringizni doim update qiling (BlueKeep kabi jiddiy RDP zaifliklari bo'lgan).

---

## Browser-based: Apache Guacamole

**Apache Guacamole** — brauzer orqali ishlaydigan **clientless** remote desktop shlyuzi (gateway). "Clientless" degani — foydalanuvchining mashinasiga hech narsa o'rnatilmaydi, faqat brauzer kerak.

```text
[Foydalanuvchi brauzeri] ──HTTPS──> [Guacamole server] ──RDP/VNC/SSH──> [Maqsad mashinalar]
```

Guacamole o'zi RDP, VNC va SSH protokollarini "tarjima" qiladi va HTML5 orqali brauzerga ekranni chiqaradi. Ustunliklari:

- Foydalanuvchi hech narsa o'rnatmaydi (faqat brauzer).
- Markazlashgan kirish nuqtasi — bitta joyda barcha ulanishlarni boshqarasiz.
- 2FA, audit log, markaziy autentifikatsiya qo'shish oson.

**💡 Tushuncha:** Guacamole ko'pincha kompaniyalarda "bitta xavfsiz eshik" (bastion) sifatida ishlatiladi: xodimlar brauzer orqali Guacamole'ga kiradi, u esa ichki serverlarga RDP/VNC/SSH qiladi. Ichki portlar internetga umuman ochilmaydi.

---

## Savol-javob (Q&A)

### ❓ Remote Desktop bilan SSH orasidagi asosiy farq nima?

**✅ Javob:** SSH sizga **matnli terminal** (CLI) beradi — buyruqlar yozasiz. Remote Desktop sizga **grafik ish stoli** (GUI) beradi — sichqoncha, oynalar, dasturlar. SSH yengil va tez, ko'p server ishlari uchun yetarli. Remote Desktop og'irroq, lekin grafik dasturlar yoki Windows GUI kerak bo'lganda zarur.

### ❓ VNC nega RDP va X2Go'dan sekinroq?

**✅ Javob:** VNC **pixel/bitmap** uzatadi — ekranning o'zgargan qismlarini rasm sifatida yuboradi. RDP va X2Go (NX) esa **grafik komandalar** yuboradi ("bu joyga to'rtburchak chiz"), bu ancha kam ma'lumot. Kam ma'lumot = tez uzatish, ayniqsa sekin internetda.

### ❓ RDP portini (3389) to'g'ridan-to'g'ri internetga ochsam nima bo'ladi?

**✅ Javob:** Bir necha daqiqada avtomatik botlar uni topib, parol tanlashga (brute-force) va ma'lum zaifliklarga (masalan BlueKeep) hujum qila boshlaydi. RDP — ransomware hujumlarining eng ko'p kirish nuqtasi. HAR DOIM SSH tunnel yoki VPN orqasiga yashiring.

### ❓ Windows Home versiyasida RDP server ishlaydimi?

**✅ Javob:** Yo'q. Windows Home faqat RDP **client** sifatida ishlaydi (boshqa mashinaga ulanadi), lekin o'ziga RDP server sifatida ulanishga ruxsat bermaydi. RDP server uchun Windows **Pro, Enterprise** yoki **Server** kerak. Muqobil: Windows Home'da VNC yoki NoMachine ishlating.

### ❓ Linux mashinaga RDP orqali ulanish mumkinmi?

**✅ Javob:** Ha. Linux'ga **xRDP** o'rnatasiz — u Linux'da RDP server ochadi. Keyin Windows'dan `mstsc` orqali Linux IP'siga ulanasiz. Lekin xRDP ba'zi desktoplar bilan (GNOME/Wayland) muammoli, XFCE tavsiya etiladi.

### ❓ VNC trafigi shifrlanadimi?

**✅ Javob:** Klassik VNC (RFB) trafigi **shifrlanmaydi** yoki juda zaif shifrlanadi — parol va ekran ochiq ko'rinishi mumkin. Shuning uchun VNC'ni HAR DOIM **SSH tunnel** ichida ishlating: `ssh -L 5901:localhost:5901 user@server`, keyin `localhost:5901` ga ulaning.

### ❓ X2Go nega alohida tunnel talab qilmaydi?

**✅ Javob:** Chunki X2Go allaqachon **SSH ustida** ishlaydi — u port 22 (SSH) orqali shifrlangan holda ulanadi. Ya'ni xavfsizlik "beriladigan" emas, "o'rnatilgan" (built-in). Bu X2Go'ni ham qulay, ham xavfsiz qiladi.

### ❓ X2Go va NoMachine orasida qaysi birini tanlashim kerak?

**✅ Javob:** Ikkalasi ham NX texnologiyasiga asoslangan va tez. **X2Go** — bepul, ochiq kodli, Linux serverlar uchun ajoyib. **NoMachine** — o'rnatishi juda oson, cross-platform (Windows/macOS ham server bo'la oladi), tijoriy lekin shaxsiy foydalanish bepul. Faqat Linux server bo'lsa X2Go; aralash platforma va maksimal soddalik kerak bo'lsa NoMachine.

### ❓ VNC port raqamlarini qanday hisoblayman?

**✅ Javob:** Formula: **5900 + display raqami**. Display `:0` → port 5900, `:1` → 5901, `:2` → 5902. Ko'pincha `:0` lokal ekranga tegishli bo'lgani uchun masofaviy sessiya `:1` (5901) dan boshlanadi.

### ❓ RDP orqali ulanganda mahalliy papkalarimni ko'ra olamanmi?

**✅ Javob:** Ha. RDP **resource redirection** qo'llab-quvvatlaydi — mahalliy disk, printer, clipboard va tovushni masofaviy sessiyaga ulash mumkin. FreeRDP'da: `xfreerdp /v:IP /drive:home,/home/user /clipboard /sound`. Windows client'da esa "Show Options → Local Resources" bo'limidan sozlaysiz.

### ❓ Masofaviy sessiyani uzsam, ochiq dasturlarim qoladimi?

**✅ Javob:** Bu protokolga bog'liq. **X2Go va NoMachine** sessiyani suspend/resume qiladi — uzsangiz ham dasturlar serverda ochiq qoladi, qaytganda davom etadi. **RDP** ham odatda sessiyani saqlaydi (disconnect ≠ logoff). Klassik **VNC** esa server jarayoni yashab tursa dasturlar qoladi. Terminal ishlar uchun `tmux`/`screen` shu vazifani bajaradi.

### ❓ Qora ekran (black screen) chiqsa nima qilaman?

**✅ Javob:** Bu ko'pincha desktop environment muammosi. Yechimlar: (1) yengil desktop o'rnating (XFCE/MATE/LXDE); (2) Wayland o'rniga Xorg ishlating (`/etc/gdm3/custom.conf` da `WaylandEnable=false`); (3) VNC'da `~/.vnc/xstartup` faylida to'g'ri desktopni ko'rsating (`startxfce4 &`); (4) xRDP'da bir vaqtda mahalliy va RDP sessiyani ochib qo'ymang.

### ❓ Guacamole "clientless" degani nimani anglatadi?

**✅ Javob:** Foydalanuvchi mashinasiga hech qanday remote desktop dasturi o'rnatilmaydi — faqat **brauzer** yetarli. Guacamole server RDP/VNC/SSH'ni HTML5'ga tarjima qiladi va brauzerda ko'rsatadi. Bu markazlashgan, o'rnatishsiz kirish nuqtasi beradi.

### ❓ Bir nechta odam bir vaqtda bitta Linux mashinaga GUI orqali ulana oladimi?

**✅ Javob:** Ha, agar har biriga alohida sessiya berilsa. X2Go va xRDP har bir foydalanuvchiga alohida display/sessiya ochadi (masalan VNC `:1`, `:2`, `:3`). Bir xil sessiyani birga ko'rish uchun esa "shared desktop" (VNC'da view-only yoki X2Go shared) rejimi ishlatiladi — bu ko'proq yordam ko'rsatish uchun.

### ❓ Mobil internetdan (sekin, uzilib turadigan) qaysi protokol yaxshi?

**✅ Javob:** **X2Go** yoki **NoMachine** (ikkalasi NX asosida). Ular grafik komandalarni aqlli siqadi va keshlaydi, shuning uchun past tezlikli va yuqori latentlikli tarmoqda ham ravon ishlaydi. VNC bunday sharoitda deyarli ishlab bo'lmaydigan darajada sekinlashadi.

---

## Masalalar

> Yechimlar: [`solutions/remote-access/03-remote-desktop.md`](../solutions/remote-access/03-remote-desktop.md)

1. **RDP yoqish va ulanish.** Bitta Windows Pro mashinada RDP'ni yoqing (Settings yoki PowerShell orqali). Boshqa mashinadan (`mstsc` yoki Linux'dan `xfreerdp`) unga ulaning. Ulanish manzili, foydalanuvchi va portni yozib bering.

2. **Xavfsiz RDP tunnel.** RDP portini internetga ochmasdan, SSH tunnel orqali masofadagi Windows'ga ulaning. Kerakli `ssh -L ...` buyrug'ini yozing va RDP client qaysi manzilga (`localhost:port`) ulanishini tushuntiring.

3. **xRDP o'rnatish.** Ubuntu mashinaga xRDP o'rnating va yoqing. Windows'dan unga ulaning. Agar qora ekran chiqsa, uni bartaraf etish uchun qanday qadamlar qo'yasiz? Buyruqlar bilan yozing.

4. **VNC server + SSH tunnel.** TigerVNC serverni `:1` displayda, faqat localhost'ga bog'langan holda ishga tushiring. So'ng SSH tunnel orqali unga ulanish uchun kerakli buyruqlarni (server va client tomonda) yozing. Port raqamini ko'rsating.

5. **VNC port hisoblash.** VNC server `:3` displayda ishlayapti. (a) U qaysi TCP portda tinglaydi? (b) `:0`, `:1`, `:5` displaylar uchun portlarni ham yozing. (c) Nega odatda `:0` emas, `:1` dan boshlanadi?

6. **X2Go sozlash.** Bir Linux serverga X2Go server va XFCE desktop o'rnating. Boshqa mashinadan X2Go client orqali unga ulaning. X2Go nega alohida SSH tunnel talab qilmasligini tushuntiring.

7. **Protokol tanlash.** Quyidagi holatlar uchun eng mos protokolni tanlang va sababini yozing: (a) Windows Server'ni boshqarish; (b) sekin mobil internetdan Linux GUI'ga ulanish; (c) foydalanuvchiga hech narsa o'rnatmasdan brauzer orqali kirish; (d) macOS'dan Windows'ga tez ulanish.

8. **Xavfsizlik auditi.** Bir jamoa RDP'ni 3389 portida to'g'ridan-to'g'ri internetga ochib qo'ygan, oddiy parol bilan. Bu yerdagi 3 ta xavfni sanang va har biri uchun tuzatish taklif qiling (aniq buyruq/sozlama bilan).

9. **Resource redirection.** Linux'dan `xfreerdp` orqali Windows'ga ulanib, mahalliy `/home/user/Downloads` papkasini, clipboard'ni va tovushni masofaviy sessiyaga ulang. To'liq buyruqni yozing.

10. **Guacamole arxitekturasi.** Apache Guacamole'ni "bastion" sifatida ishlatadigan sxemani ASCII diagramma bilan chizing: foydalanuvchi brauzeri, Guacamole server, va ichki RDP/VNC/SSH maqsad mashinalar. Ichki portlar nega internetga ochilmasligini tushuntiring.

---

← [Remote Access bo'limiga qaytish](./README.md)
