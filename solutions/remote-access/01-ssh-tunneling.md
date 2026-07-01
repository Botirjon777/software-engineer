# SSH va Tunneling — Yechimlar

Bu fayl [`remote-access/01-ssh-tunneling.md`](../../remote-access/01-ssh-tunneling.md) dagi masalalar yechimlarini o'z ichiga oladi. Avval o'zingiz yechishga harakat qiling, keyin solishtiring.

## Mundarija

- [1-masala: Local forward (Redis)](#1-masala-local-forward-redis)
- [2-masala: Remote forward](#2-masala-remote-forward)
- [3-masala: SOCKS proxy](#3-masala-socks-proxy)
- [4-masala: Jump host](#4-masala-jump-host)
- [5-masala: Reverse tunnel + autossh](#5-masala-reverse-tunnel--autossh)
- [6-masala: SSH config](#6-masala-ssh-config)
- [7-masala: rsync](#7-masala-rsync)
- [8-masala: ngrok / Cloudflare](#8-masala-ngrok--cloudflare)
- [9-masala: Cheklangan kalit](#9-masala-cheklangan-kalit)
- [10-masala: Diagnostika](#10-masala-diagnostika)

---

## 1-masala: Local forward (Redis)

```bash
# Lokal 6380 -> serverdagi 127.0.0.1:6379 (Redis)
ssh -N -L 6380:localhost:6379 user@server.example.com

# Boshqa terminalda:
redis-cli -h localhost -p 6380 ping   # PONG qaytishi kerak
```

**Izoh:** Lokal `6380` ni tanladik, chunki lokal Redis (agar bo'lsa) `6379` ni band qilgan bo'lishi mumkin. `localhost:6379` — server nuqtai nazaridan Redis manzili.

---

## 2-masala: Remote forward

```bash
# VPS ning 9000-porti -> mening localhost:8000
ssh -N -R 9000:localhost:8000 user@vps.example.com

# Tunnelni tashqaridan ko'rinadigan qilish uchun 0.0.0.0 ga bog'lash:
ssh -N -R 0.0.0.0:9000:localhost:8000 user@vps.example.com
```

Server tarafida `/etc/ssh/sshd_config` da:

```ini
GatewayPorts clientspecified
```

So'ng `sudo systemctl restart ssh`. `clientspecified` — client `0.0.0.0` deb so'raganidagina tashqariga ochadi (xavfsizroq). Firewallda `9000/tcp` ham ochilishi kerak.

---

## 3-masala: SOCKS proxy

```bash
ssh -N -D 1080 user@server.example.com

# Boshqa terminalda tekshirish:
curl --socks5-hostname localhost:1080 https://ifconfig.me
```

**Muhim flag:** `--socks5-hostname` (nafaqat `--socks5`) — DNS so'rovlarini ham tunnel ichida hal qiladi, shu bilan **DNS leak** bo'lmaydi. Brauzerda esa "Proxy DNS when using SOCKS v5" ni yoqish kerak.

---

## 4-masala: Jump host

(a) Bir qatorli buyruq:

```bash
ssh -J user@bastion.corp.com user@10.0.5.20
```

(b) `~/.ssh/config` bilan:

```ini
Host bastion
    HostName bastion.corp.com
    User user

Host internal
    HostName 10.0.5.20
    User user
    ProxyJump bastion
```

So'ng shunchaki: `ssh internal`.

---

## 5-masala: Reverse tunnel + autossh

autossh buyrug'i:

```bash
autossh -M 0 -N -o "ServerAliveInterval 30" -o "ServerAliveCountMax 3" \
    -R 2222:localhost:22 user@vps.example.com
```

`systemd` unit fayli:

```ini
# /etc/systemd/system/reverse-tunnel.service
[Unit]
Description=Reverse SSH tunnel to VPS
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
sudo systemctl daemon-reload
sudo systemctl enable --now reverse-tunnel.service
```

Kalit parolsiz (yoki ssh-agent orqali) bo'lishi shart.

---

## 6-masala: SSH config

```ini
# ~/.ssh/config
Host db
    HostName db-server.example.com
    User dbuser
    IdentityFile ~/.ssh/id_ed25519
    LocalForward 5432 localhost:5432
    ServerAliveInterval 30
    ServerAliveCountMax 3
```

Ishlatish: `ssh -N db` (tunnel ochiladi), so'ng `psql -h localhost -p 5432 ...`.

---

## 7-masala: rsync

```bash
rsync -avz --progress -e "ssh -p 2222" ./backup/ user@server.example.com:/srv/backup/
```

- `-a` — arxiv rejim (huquq, vaqt saqlanadi)
- `-v` — batafsil
- `-z` — siqish
- `--progress` — progress ko'rsatish
- `-e "ssh -p 2222"` — boshqa portdagi SSH
- Manba oxiridagi `/` — "papka ichidagini ko'chir" (papkaning o'zini emas) degani.

---

## 8-masala: ngrok / Cloudflare

(a) ngrok:

```bash
ngrok http 3000
```

(b) Cloudflare Tunnel:

```bash
cloudflared tunnel --url http://localhost:3000
```

**Farq:** Ikkalasi ham lokal `3000` ni internetga ochadi va HTTPS URL beradi. ngrok trafik inspektori (webhook debug) bilan qulay; Cloudflare bepul, doimiy va o'z domeningiz bilan ishlashda kuchli. Ikkalasi ham boshqariladigan reverse tunnel bo'lgani uchun NAT orqasida ishlaydi.

---

## 9-masala: Cheklangan kalit

Serverdagi `~/.ssh/authorized_keys` da kalit oldiga cheklovlar qo'yiladi:

```text
command="echo 'faqat forwarding'",no-pty,no-X11-forwarding,permitopen="localhost:5432" ssh-ed25519 AAAA... foydalanuvchi@host
```

- `command="..."` — bu kalit bilan kirgan sessiya shell o'rniga shu buyruqni bajaradi (interaktiv shell yo'q).
- `no-pty` — terminal (PTY) berilmaydi.
- `no-X11-forwarding` — X11 o'chirilgan.
- `permitopen="localhost:5432"` — faqat shu manzilga forwarding ruxsat.

Client `ssh -N -L 5432:localhost:5432 user@server` bilan faqat tunnel ochadi, shell ololmaydi.

---

## 10-masala: Diagnostika

`ssh -L 5432:localhost:5432 user@server` da "connection refused" sabablari:

1. **Serverda DB ishlamayapti yoki boshqa manzilda tinglaydi.**
   Tekshirish: serverga kirib `ss -tlnp | grep 5432`. Agar `127.0.0.1:5432` da bo'lsa `localhost:5432` to'g'ri; agar boshqa IP da bo'lsa `-L 5432:<o'sha_ip>:5432` yozing.

2. **`AllowTcpForwarding no` server sshd_config da.**
   Tekshirish: `sudo grep AllowTcpForwarding /etc/ssh/sshd_config`. Agar `no` bo'lsa, `yes` qilib `systemctl restart ssh`.

3. **Lokal port band yoki noto'g'ri ulanish manzili.**
   Tekshirish: `ssh -v -L ...` bilan verbose log ko'ring; lokalda `psql -h localhost` (127.0.0.1) ni ishlating, `-h /var/run` (socket) emas. Lokal `5432` band bo'lsa boshqa port (masalan `5433`) tanlang.

Umumiy diagnostika: `ssh -v` (yoki `-vvv`) tunnel qurilish jarayonini batafsil ko'rsatadi.

---

← [Remote Access bo'limiga qaytish](../../remote-access/README.md)
