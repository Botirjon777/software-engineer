# Remote Desktop (RDP, VNC, X2Go) — Masalalar Yechimi

Bu fayl [`remote-access/03-remote-desktop.md`](../../remote-access/03-remote-desktop.md) dagi mashqlarning yechimlarini o'z ichiga oladi. Avval o'zingiz yechishga harakat qiling, keyin bu yerga qarang.

---

## 1-masala: RDP yoqish va ulanish

**Server (Windows Pro) tomonda RDP yoqish:**

PowerShell orqali (administrator huquqi bilan):

```powershell
# RDP ni yoqish
Set-ItemProperty -Path 'HKLM:\System\CurrentControlSet\Control\Terminal Server' -Name "fDenyTSConnections" -Value 0
# Firewall'da ruxsat
Enable-NetFirewallRule -DisplayGroup "Remote Desktop"
```

Yoki GUI: `Settings → System → Remote Desktop → Enable Remote Desktop = ON`.

**Ulanish (client tomonda):**

```powershell
# Windows'dan
mstsc /v:192.168.1.50:3389
```

```bash
# Linux'dan
xfreerdp /v:192.168.1.50 /u:Administrator /p:'parol' /size:1440x900
```

- **Ulanish manzili:** `192.168.1.50:3389`
- **Port:** 3389 (RDP standart porti, TCP)
- **Foydalanuvchi:** Windows account nomi (masalan `Administrator`)

---

## 2-masala: Xavfsiz RDP tunnel

Client mashinasida SSH tunnel ochamiz:

```bash
# Lokal 13389 portini masofadagi server'ning 3389 portiga bog'lash
ssh -L 13389:localhost:3389 user@server.example.com
```

So'ng RDP client'da ulanish manzili sifatida **`localhost:13389`** kiritiladi:

```powershell
mstsc /v:localhost:13389
```

```bash
xfreerdp /v:localhost:13389 /u:Administrator /p:'parol'
```

**Tushuntirish:** RDP trafigi endi `localhost:13389` → SSH shifrlangan tunnel → `server:3389` yo'nalishida o'tadi. 3389 porti internetga umuman ochilmaydi — server'ga faqat SSH (22) orqali kiriladi.

```text
[RDP client] → localhost:13389 → [SSH tunnel, shifrlangan] → server:3389 → [RDP server]
```

---

## 3-masala: xRDP o'rnatish

```bash
sudo apt update
sudo apt install xrdp -y
sudo systemctl enable --now xrdp
sudo systemctl status xrdp        # ishlayotganini tekshirish
sudo ufw allow 3389/tcp           # firewall (lokal tarmoq uchun)
```

Windows'dan: `mstsc /v:<ubuntu-ip>:3389`.

**Qora ekran chiqsa bartaraf etish qadamlar:**

1. Yengil desktop o'rnatish:
   ```bash
   sudo apt install xfce4 xfce4-goodies -y
   echo "startxfce4" > ~/.xsession
   ```
2. Wayland o'chirish (agar GDM ishlatilsa):
   ```bash
   # /etc/gdm3/custom.conf ichida:
   # WaylandEnable=false
   sudo systemctl restart gdm3
   ```
3. xRDP'ni qayta ishga tushirish:
   ```bash
   sudo systemctl restart xrdp
   ```
4. Bir vaqtda mahalliy ekranga login qilmaslik (bitta foydalanuvchi ikki sessiya ochsa konflikt bo'ladi).

---

## 4-masala: VNC server + SSH tunnel

**Server tomonda** (faqat localhost'ga bog'langan):

```bash
sudo apt install tigervnc-standalone-server -y
vncpasswd                                        # parol o'rnatish
vncserver :1 -geometry 1440x900 -localhost yes   # faqat localhost, port 5901
```

**Client tomonda** SSH tunnel:

```bash
ssh -L 5901:localhost:5901 user@server.example.com
```

So'ng VNC client'da ulanish manzili:

```bash
xvncviewer localhost:5901
```

- **Port:** 5901 (5900 + display raqami 1).
- `-localhost yes` tufayli VNC faqat server ichida tinglaydi; tashqaridan faqat SSH tunnel orqali kiriladi.

---

## 5-masala: VNC port hisoblash

Formula: **port = 5900 + display raqami**.

**(a)** Display `:3` → `5900 + 3` = **5903**.

**(b)** Boshqa displaylar:

| Display | Port |
|---------|------|
| `:0` | 5900 |
| `:1` | 5901 |
| `:5` | 5905 |

**(c)** Nega odatda `:1` dan boshlanadi: Display `:0` ko'pincha **mashinaning haqiqiy (fizik) ekraniga** (mahalliy monitor) tegishli bo'ladi. Masofaviy VNC sessiyasi yangi virtual display kerak qilgani uchun konfliktdan qochib `:1` (5901) dan boshlanadi.

---

## 6-masala: X2Go sozlash

**Server tomonda:**

```bash
sudo apt update
sudo apt install x2goserver x2goserver-xsession -y
sudo apt install xfce4 xfce4-goodies -y
sudo apt install openssh-server -y
sudo systemctl enable --now ssh
```

**Client tomonda:**

```bash
sudo apt install x2goclient -y
```

Client'da yangi session yaratiladi:

```text
Host         : server.example.com
Login        : user
SSH port     : 22
Session type : XFCE
```

**Nega alohida SSH tunnel kerak emas:** X2Go dizayni bo'yicha **SSH ustida** ishlaydi — u port 22 orqali shifrlangan ulanish o'rnatib, grafik ma'lumotni shu tunnel ichida uzatadi. Xavfsizlik protokolga o'rnatilgan (built-in), shuning uchun qo'lda `ssh -L` qilish shart emas.

---

## 7-masala: Protokol tanlash

| Holat | Tanlov | Sabab |
|-------|--------|-------|
| (a) Windows Server boshqaruvi | **RDP** | Windows'ga o'rnatilgan, samarali, native. |
| (b) Sekin mobil internetdan Linux GUI | **X2Go** (yoki NoMachine) | NX texnologiyasi past tezlikda ham ravon, SSH ustida xavfsiz. |
| (c) Brauzer orqali, o'rnatmasdan | **Apache Guacamole** | Clientless — faqat brauzer kerak. |
| (d) macOS'dan Windows'ga tez | **NoMachine** (yoki RDP) | Cross-platform, tez; RDP ham Microsoft Remote Desktop client orqali ishlaydi. |

---

## 8-masala: Xavfsizlik auditi

Ochiq 3389 + oddiy parol holatidagi 3 ta xavf va tuzatish:

1. **3389 porti to'g'ridan-to'g'ri internetda** → botlar skanerlab, brute-force va BlueKeep kabi zaifliklarga hujum qiladi.
   - Tuzatish: portni yopib, SSH tunnel/VPN orqasiga yashirish.
     ```bash
     sudo ufw deny 3389/tcp
     ssh -L 13389:localhost:3389 user@server   # keyin localhost:13389 ga ulanish
     ```

2. **Oddiy (zaif) parol** → brute-force bilan tez ochiladi.
   - Tuzatish: kuchli parol + account lockout / fail2ban:
     ```bash
     sudo apt install fail2ban -y
     # jail sozlab, ketma-ket xato loginlarda IP'ni bloklash
     ```

3. **IP cheklovi yo'q** → dunyoning istalgan joyidan urinish mumkin.
   - Tuzatish: firewall'da faqat kerakli IP'ga ruxsat:
     ```bash
     sudo ufw allow from 203.0.113.10 to any port 3389
     ```

Qo'shimcha: RDP/OS yangilanishlarini o'rnatish (BlueKeep patch), imkon bo'lsa Network Level Authentication (NLA) yoqish.

---

## 9-masala: Resource redirection

Linux'dan Windows'ga papka + clipboard + tovush bilan ulanish:

```bash
xfreerdp /v:192.168.1.50 /u:User /p:'parol' \
  /drive:downloads,/home/user/Downloads \
  /clipboard \
  /sound \
  /size:1440x900
```

- `/drive:downloads,/home/user/Downloads` — mahalliy papkani masofaviy sessiyada "downloads" nomli disk sifatida ko'rsatadi.
- `/clipboard` — nusxa-joylash (copy-paste) ikki tomonlama ishlaydi.
- `/sound` — masofaviy tovushni mahalliy karnaydan chiqaradi.

---

## 10-masala: Guacamole arxitekturasi

```text
                        ┌──────────────────────┐
   Foydalanuvchi        │   Guacamole server   │
   ┌───────────┐        │  (guacd + web app)   │
   │  Brauzer  │──HTTPS─▶│  - autentifikatsiya  │
   │  (HTML5)  │        │  - 2FA, audit log    │
   └───────────┘        └───────┬──────────────┘
                                │  (ichki tarmoq)
              ┌─────────────────┼─────────────────┐
              │ RDP             │ VNC             │ SSH
         ┌────▼────┐       ┌────▼────┐       ┌────▼────┐
         │ Windows │       │ Linux   │       │ Server  │
         │  :3389  │       │  :5901  │       │   :22   │
         └─────────┘       └─────────┘       └─────────┘
              (faqat ichki tarmoqda, internetga OCHILMAGAN)
```

**Nega ichki portlar internetga ochilmaydi:** Faqat Guacamole server internetga qaraydi (HTTPS orqali). Maqsad mashinalarning RDP/VNC/SSH portlari faqat ichki tarmoqda, Guacamole server ular bilan gaplashadi. Foydalanuvchi hech qachon 3389/5901/22 ga to'g'ridan-to'g'ri tegmaydi — hamma trafik bitta nazorat qilinadigan, shifrlangan, autentifikatsiyalangan "eshik" (bastion) orqali o'tadi. Bu hujum yuzasini (attack surface) minimallashtiradi.

---

← [Remote Access bo'limiga qaytish](../../remote-access/README.md)
