# VPN — Yechimlar

Bu fayl [`remote-access/02-vpn.md`](../../remote-access/02-vpn.md) dagi masalalar yechimlarini o'z ichiga oladi. Avval o'zingiz yechishga harakat qiling, keyin solishtiring.

## Mundarija

- [1-masala: Tushunchalar](#1-masala-tushunchalar)
- [2-masala: Protokol tanlash](#2-masala-protokol-tanlash)
- [3-masala: Kalit generatsiyasi](#3-masala-kalit-generatsiyasi)
- [4-masala: Server config](#4-masala-server-config)
- [5-masala: Client config — full va split](#5-masala-client-config--full-va-split)
- [6-masala: Ikkinchi client](#6-masala-ikkinchi-client)
- [7-masala: Diagnostika](#7-masala-diagnostika)
- [8-masala: DNS leak](#8-masala-dns-leak)
- [9-masala: Tailscale](#9-masala-tailscale)
- [10-masala: Taqqoslash](#10-masala-taqqoslash)

---

## 1-masala: Tushunchalar

- **Tunnel** — jamoat interneti ustidan qurilgan mantiqiy xususiy kanal. Misol: uydagi laptop VPS ga tunnel qurib, ofis tarmog'iga xuddi ofisda o'tirgandek ulanadi.
- **Shifrlash (encryption)** — tunnel ichidagi barcha ma'lumot o'qib bo'lmaydigan holga keltiriladi. Misol: kafe WiFi da parolingiz shifrlanib o'tadi, hech kim ko'ra olmaydi.
- **Virtual interfeys** — OS uchun oddiy tarmoq kartasidek ko'rinadigan, lekin orqasida tunnel/shifrlash bo'lgan interfeys (`wg0`, `tun0`). Misol: `wg0` ga `10.0.0.2` IP beriladi va barcha ilova hech o'zgarishsiz VPN dan foydalanadi.

---

## 2-masala: Protokol tanlash

- (a) **Mobil, WiFi↔4G tez almashadi → IKEv2.** MOBIKE tufayli tarmoq almashganda ulanishni tez tiklaydi. (WireGuard ham yaxshi ishlaydi, lekin IKEv2 mobil uchun native va barqaror.)
- (b) **Faqat TCP/443 ochiq firewall → OpenVPN (TCP/443).** HTTPS trafikdek ko'rinib, firewalldan o'tadi. WireGuard UDP bo'lgani uchun bloklanishi mumkin.
- (c) **Uy uchun tez, oddiy → WireGuard.** Eng tez, eng oddiy config, past kechikish.

---

## 3-masala: Kalit generatsiyasi

```bash
# Server
wg genkey | tee server_private.key | wg pubkey > server_public.key

# Client
wg genkey | tee client_private.key | wg pubkey > client_public.key
```

**Qayerda saqlanadi:**
- `server_private.key` — faqat **serverda**, hech kimga berilmaydi (`chmod 600`).
- `server_public.key` — **clientga** beriladi (client config dagi `[Peer] PublicKey`).
- `client_private.key` — faqat **clientda**, hech kimga berilmaydi.
- `client_public.key` — **serverga** beriladi (server config dagi `[Peer] PublicKey`).

Private kalit hech qachon tarmoqdan chiqmaydi.

---

## 4-masala: Server config

```ini
# /etc/wireguard/wg0.conf  (SERVER)
[Interface]
Address = 10.0.0.1/24
ListenPort = 51820
PrivateKey = <SERVER_PRIVATE_KEY>
PostUp = iptables -A FORWARD -i wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
PostDown = iptables -D FORWARD -i wg0 -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE

[Peer]
PublicKey = <CLIENT_PUBLIC_KEY>
AllowedIPs = 10.0.0.2/32
```

Qo'shimcha:

```bash
echo 'net.ipv4.ip_forward=1' | sudo tee /etc/sysctl.d/99-wireguard.conf
sudo sysctl -p /etc/sysctl.d/99-wireguard.conf
sudo ufw allow 51820/udp
sudo wg-quick up wg0
```

`eth0` ni serveringizning haqiqiy tashqi interfeysiga moslang (`ip a` bilan tekshiring).

---

## 5-masala: Client config — full va split

(a) Full tunnel:

```ini
# /etc/wireguard/wg0.conf  (CLIENT — FULL)
[Interface]
Address = 10.0.0.2/24
PrivateKey = <CLIENT_PRIVATE_KEY>
DNS = 1.1.1.1

[Peer]
PublicKey = <SERVER_PUBLIC_KEY>
Endpoint = <SERVER_PUBLIC_IP>:51820
AllowedIPs = 0.0.0.0/0
PersistentKeepalive = 25
```

(b) Split tunnel — faqat `AllowedIPs` o'zgaradi:

```ini
AllowedIPs = 10.0.0.0/24
```

**Farq:** Full tunnelda (`0.0.0.0/0`) butun trafik VPN dan o'tadi — IP yashiriladi, maksimal himoya, lekin sekinroq. Split tunnelda (`10.0.0.0/24`) faqat ichki tarmoq VPN dan o'tadi, qolgan internet to'g'ridan-to'g'ri — tezroq, lekin faqat qisman himoya va IP yashirilmaydi.

---

## 6-masala: Ikkinchi client

Server config ga yangi `[Peer]` bloki qo'shiladi:

```ini
[Peer]
PublicKey = <CLIENT2_PUBLIC_KEY>
AllowedIPs = 10.0.0.3/32
```

So'ng `sudo wg-quick down wg0 && sudo wg-quick up wg0`.

Ikkinchi client config:

```ini
# CLIENT 2
[Interface]
Address = 10.0.0.3/24
PrivateKey = <CLIENT2_PRIVATE_KEY>
DNS = 1.1.1.1

[Peer]
PublicKey = <SERVER_PUBLIC_KEY>
Endpoint = <SERVER_PUBLIC_IP>:51820
AllowedIPs = 0.0.0.0/0
PersistentKeepalive = 25
```

Har bir client noyob IP (`10.0.0.3`) va o'z kalitlariga ega bo'lishi shart.

---

## 7-masala: Diagnostika

Handshake bor, lekin internetga chiqa olmayapti:

1. **IP forwarding o'chirilgan.**
   Tekshirish: `sysctl net.ipv4.ip_forward` → `1` bo'lishi kerak. Yo'q bo'lsa yoqing va `sysctl -p`.

2. **NAT (MASQUERADE) qoidasi yo'q yoki noto'g'ri interfeys.**
   Tekshirish: `sudo iptables -t nat -L POSTROUTING -v`. `eth0` o'rniga haqiqiy tashqi interfeys (`ip a` bilan aniqlang) bo'lishi kerak.

3. **Client da DNS ishlamayapti.**
   Tekshirish: `ping 1.1.1.1` ishlasa-yu `ping google.com` ishlamasa — DNS muammosi. Client config ga `DNS = 1.1.1.1` qo'shing va tunnelni qayta ishga tushiring.

---

## 8-masala: DNS leak

Client config da:

```ini
[Interface]
DNS = 1.1.1.1
```

Tekshirish:

```bash
resolvectl status        # wg0 interfeysida DNS 1.1.1.1 ekanini ko'ring
# yoki brauzerda https://dnsleaktest.com ga kiring
```

Agar test faqat VPN serverning (yoki 1.1.1.1) DNS ini ko'rsatsa — leak yo'q. Agar internet provayderingizning DNS i ko'rinsa — leak bor, DNS sozlamasini tuzating.

---

## 9-masala: Tailscale

```bash
# Har uch qurilmada (laptop, VPS):
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up
# Telefonda: App Store/Play Market dan Tailscale ilovasini o'rnating.

# Barchasida bir xil akkaunt bilan login qiling.
tailscale status   # barcha qurilmalar va ularning IP lari ko'rinadi
```

**Nega osonroq:** WireGuard config da har bir juftlik uchun kalit almashuv, `AllowedIPs`, `Endpoint` ni qo'lda yozish kerak (N qurilma uchun murakkablik tez oshadi). Tailscale kalit almashuv, qurilmalarni topish, NAT traversal va routing ni avtomatlashtiradi — siz faqat login qilasiz.

---

## 10-masala: Taqqoslash

| Xususiyat | VPN | HTTP proxy | SSH tunnel |
|---|---|---|---|
| Shifrlash | Ha, to'liq (tarmoq darajasi) | Odatda yo'q (faqat yo'naltiradi) | Ha (SSH kanali) |
| Qamrov | Butun mashina (barcha ilova, TCP+UDP) | Odatda bitta ilova (brauzer) | Bitta port / SOCKS (asosan TCP) |
| Use-case | Doimiy xavfsiz tarmoq, ichki resurslar | Trafikni yo'naltirish, oddiy filtr | Tez, bir martalik xizmatga ulanish, admin |

**Qisqacha:** HTTP proxy shifrlamaydi va tor qamrovli. SSH tunnel shifrlaydi, lekin asosan bitta xizmat/TCP. VPN — butun mashina trafigini shifrlab, tarmoq darajasida tunnel qiladi (eng keng qamrovli).

---

← [Remote Access bo'limiga qaytish](../../remote-access/README.md)
