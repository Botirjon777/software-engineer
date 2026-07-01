# VPN

Ushbu qo'llanmada **VPN** (Virtual Private Network) — masofaviy va xavfsiz tarmoq ulanishining asosiy texnologiyasi batafsil yoritiladi. Siz VPN nima ekanligini, qanday ishlashini, protokollar farqini o'rganasiz va eng muhimi — 2026-yilgi eng zamonaviy usul **WireGuard** yordamida o'z VPN serveringizni VPS da qadama-qadam qurasiz. Barcha misollar Linux/Ubuntu asosida.

VPN sizga jamoat internetidan foydalanganda ham shifrlangan, xavfsiz "xususiy tarmoq" yaratib beradi: ochiq WiFi da himoya, ofis ichki resurslariga masofadan kirish, IP ni yashirish va geografik cheklovlarni chetlab o'tish.

## Mundarija

- [VPN nima va nega kerak](#vpn-nima-va-nega-kerak)
- [VPN qanday ishlaydi](#vpn-qanday-ishlaydi)
- [Use-case'lar](#use-caselar)
- [VPN protokollari taqqoslash](#vpn-protokollari-taqqoslash)
  - [WireGuard](#wireguard)
  - [OpenVPN](#openvpn)
  - [IPSec / IKEv2](#ipsec--ikev2)
  - [WireGuard vs OpenVPN jadval](#wireguard-vs-openvpn-jadval)
- [Split tunneling vs full tunneling](#split-tunneling-vs-full-tunneling)
- [O'z VPN serveringizni qurish (WireGuard)](#oz-vpn-serveringizni-qurish-wireguard)
  - [Tayyor yechim: Tailscale va Netbird](#tayyor-yechim-tailscale-va-netbird)
- [VPN vs proxy vs SSH tunnel](#vpn-vs-proxy-vs-ssh-tunnel)
- [Tijorat VPN vs o'z VPN](#tijorat-vpn-vs-oz-vpn)
- [DNS leak va kill switch](#dns-leak-va-kill-switch)
- [Xavfsizlik va maxfiylik](#xavfsizlik-va-maxfiylik)
- [Savol-javob (Q&A)](#savol-javob-qa)
- [Masalalar](#masalalar)

---

## VPN nima va nega kerak

**VPN** (Virtual Private Network — Virtual Xususiy Tarmoq) — bu jamoat tarmog'i (internet) ustidan shifrlangan "tunnel" yaratib, ikki nuqta yoki tarmoqni xuddi bitta xususiy lokal tarmoqdagidek bog'laydigan texnologiya.

Nega kerak:

1. **Xavfsizlik** — butun trafik shifrlanadi. Ochiq WiFi (kafe, aeroport) da hech kim trafigingizni o'qiy olmaydi.
2. **Ichki resurslarga kirish** — ofis serverlariga, ichki DB, admin panellarga uydan xavfsiz ulanish.
3. **IP yashirish** — real IP ingiz o'rniga VPN serverning IP si ko'rinadi.
4. **Geo-cheklovlarni chetlab o'tish** — boshqa mamlakatdagi kontentga kirish.
5. **Site-to-site** — ikki ofis tarmog'ini bitta xususiy tarmoqqa birlashtirish.

**💡 Tushuncha:** "Virtual" so'zi muhim — jismoniy alohida kabel tortmaysiz. Internetning o'zi ustidan **mantiqiy (logical)** xususiy tarmoq yaratasiz. Trafik shifrlangani uchun oradagi hech kim (internet provayder ham) ichini ko'ra olmaydi.

---

## VPN qanday ishlaydi

VPN uchta asosiy komponentga tayanadi: **tunnel**, **shifrlash (encryption)** va **virtual tarmoq interfeysi**.

```text
   Client                                              VPN Server
   ┌───────────────────┐                           ┌───────────────────┐
   │  Ilova            │                           │                   │
   │    │              │                           │                   │
   │    ▼              │    Internet (shifrlangan) │                   │
   │  Virtual iface ───┼═══════════════════════════┼──▶ Virtual iface  │
   │  (wg0 / tun0)     │      encrypted tunnel     │   (wg0)           │
   │  10.0.0.2         │                           │   10.0.0.1        │
   └───────────────────┘                           └─────────┬─────────┘
                                                             ▼
                                                    Internet / ichki tarmoq
```

Bosqichma-bosqich:

1. Client da **virtual tarmoq interfeysi** yaratiladi (masalan, `wg0` yoki `tun0`) va unga xususiy IP (masalan, `10.0.0.2`) beriladi.
2. Shu interfeysga yuborilgan trafik **shifrlanadi** (encryption) va internet orqali VPN serverga jo'natiladi.
3. Server trafikni **deshifrlaydi** va uni yakuniy manzilga (internetga yoki ichki tarmoqqa) yo'naltiradi.
4. Javob teskari yo'l bilan qaytadi.

**💡 Tushuncha:** Virtual interfeys — operatsion tizim uchun oddiy tarmoq kartasidek ko'rinadi, lekin uning "orqasida" shifrlash va tunnel bor. Shuning uchun barcha ilovalar hech qanday o'zgarishsiz VPN orqali ishlay oladi.

---

## Use-case'lar

- **Remote work (masofaviy ish):** uydan ofis tarmog'iga ulanib, ichki serverlar, DB, wiki ga xavfsiz kirish.
- **Ochiq WiFi da xavfsizlik:** kafe/aeroportda trafikni shifrlash, o'g'irlashning oldini olish.
- **Geo-restriction:** boshqa hududda mavjud xizmatga kirish (IP boshqa mamlakatda ko'rinadi).
- **Site-to-site:** ikki ofis yoki ofis va cloud (AWS/GCP) tarmoqlarini bitta xususiy tarmoqqa birlashtirish.
- **IP yashirish / maxfiylik:** real joylashuv va IP ni tashqi saytlardan yashirish.

**⚠️ Ehtiyot bo'l:** VPN sizni to'liq "anonim" qilmaydi. VPN provayder (yoki o'z serveringiz) trafikni ko'ra oladi. Maxfiylik uchun ishonchli/o'z serveringizdan foydalaning va HTTPS ni unutmang.

---

## VPN protokollari taqqoslash

Protokol — VPN qanday tunnel qurishi va shifrlashini belgilaydi. Asosiylari: WireGuard, OpenVPN, IPSec/IKEv2.

### WireGuard

2026-yilning **tavsiya etilgan** protokoli. Zamonaviy, juda tez, kodi kichik (~4000 qator, audit qilish oson), zamonaviy kriptografiyaga tayanadi (Curve25519, ChaCha20). Linux yadrosiga kiritilgan.

- **Afzalliklari:** yuqori tezlik, past kechikish (latency), oddiy config, kam batareya sarfi (mobil uchun yaxshi), tez ulanish/qayta ulanish.
- **Kamchiliklari:** default holda dinamik IP boshqaruvi cheklangan (lekin Tailscale/Netbird buni hal qiladi), ba'zi korporativ muhitlarda kamroq "moslashuvchan".

### OpenVPN

Yillar davomida sinovdan o'tgan, o'ta **ishonchli va moslashuvchan** protokol. TCP yoki UDP ustida ishlaydi, TLS ga tayanadi, deyarli barcha platformada mavjud.

- **Afzalliklari:** juda moslashuvchan, TCP/443 orqali firewalllarni chetlab o'ta oladi (HTTPS dek ko'rinadi), keng qo'llab-quvvatlash, yetuk ekotizim.
- **Kamchiliklari:** WireGuard dan sekinroq, config murakkabroq, kodi katta.

### IPSec / IKEv2

Ko'pincha korporativ va mobil qurilmalarda ishlatiladigan protokol to'plami. **IKEv2** ayniqsa mobil uchun yaxshi — tarmoq almashganda (WiFi ↔ 4G) ulanishni tez tiklaydi (MOBIKE).

- **Afzalliklari:** OS larga o'rnatilgan (native), mobil barqarorligi yaxshi, tez qayta ulanish.
- **Kamchiliklari:** sozlash murakkab, ba'zan NAT/firewall bilan muammo, konfiguratsiya nozik.

### WireGuard vs OpenVPN jadval

| Xususiyat | WireGuard | OpenVPN |
|---|---|---|
| Tezlik | Juda yuqori | O'rtacha |
| Kod hajmi | ~4000 qator | ~400000+ qator |
| Kriptografiya | Zamonaviy, qat'iy | Moslashuvchan (TLS) |
| Config murakkabligi | Oddiy | Murakkab |
| Firewall chetlash | UDP (kamroq moslashuvchan) | TCP/443 (yaxshi) |
| Ulanish tezligi | Bir zumda | Sekinroq |
| Mobil/batareya | A'lo | O'rtacha |
| Yetuklik | Yangi, tez tarqalgan | Uzoq sinovdan o'tgan |

**💡 Tushuncha:** Shubhangiz bo'lsa — **WireGuard** dan boshlang. U tez, oddiy va xavfsiz. Faqat qattiq cheklovli firewall (masalan, faqat TCP/443 ruxsat etilgan) muhitida OpenVPN (TCP rejimida) afzalroq bo'lishi mumkin.

---

## Split tunneling vs full tunneling

VPN yoqilganda qaysi trafik tunneldan o'tishini belgilaydi:

```text
Full tunneling:                    Split tunneling:
BUTUN trafik → VPN                 Faqat ofis trafigi → VPN
                                   Qolgani → to'g'ridan-to'g'ri internetga

  Client                            Client
    │                                 │
    ├── hamma narsa ──▶ VPN           ├── 10.0.0.0/8 ──▶ VPN (ichki)
                                      └── boshqasi ────▶ Internet
```

- **Full tunneling:** butun trafik VPN orqali. Maksimal xavfsizlik va IP yashirish, lekin sekinroq va serverga yuk ko'proq.
- **Split tunneling:** faqat kerakli trafik (masalan, ofis `10.0.0.0/8`) VPN dan o'tadi, qolgani to'g'ridan-to'g'ri. Tezroq, lekin faqat qisman himoya.

WireGuard da buni `AllowedIPs` belgilaydi:

```ini
# Full tunnel — hamma narsa VPN orqali
AllowedIPs = 0.0.0.0/0, ::/0

# Split tunnel — faqat ichki tarmoq VPN orqali
AllowedIPs = 10.0.0.0/24
```

**💡 Tushuncha:** WireGuard da `AllowedIPs` — bu ham "kimga yo'naltirish" (routing), ham "kimdan qabul qilish" (access control) ma'nosini bildiradi. Bu WireGuard ning eng muhim va ko'p chalkashtiriladigan tushunchasi.

---

## O'z VPN serveringizni qurish (WireGuard)

Endi eng amaliy qismi — VPS da o'z WireGuard VPN serveringizni qadama-qadam quramiz. Sizga oddiy Ubuntu VPS (public IP bilan) kerak.

**1-qadam. WireGuard o'rnatish (server va client ikkalasida):**

```bash
sudo apt update && sudo apt install -y wireguard
```

**2-qadam. Kalitlar generatsiyasi (server tomonida):**

```bash
# Server kalitlari
wg genkey | tee server_private.key | wg pubkey > server_public.key

# Client kalitlari
wg genkey | tee client_private.key | wg pubkey > client_public.key

cat server_private.key server_public.key client_private.key client_public.key
```

**💡 Tushuncha:** Har bir tomon (server va har bir client) o'z **private** kalitini yashirin saqlaydi va faqat **public** kalitini boshqa tomonga beradi. Bu asimmetrik kalit almashuvi — private kalit hech qachon tarmoqdan chiqmaydi.

**3-qadam. Server config (`/etc/wireguard/wg0.conf`):**

```ini
# /etc/wireguard/wg0.conf  (SERVER)
[Interface]
Address = 10.0.0.1/24
ListenPort = 51820
PrivateKey = <SERVER_PRIVATE_KEY>
# NAT — clientlar internetga chiqishi uchun (eth0 ni o'z interfeysingizga moslang)
PostUp = iptables -A FORWARD -i wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
PostDown = iptables -D FORWARD -i wg0 -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE

[Peer]
# Client
PublicKey = <CLIENT_PUBLIC_KEY>
AllowedIPs = 10.0.0.2/32
```

**4-qadam. IP forwarding ni yoqish (serverda):**

```bash
echo 'net.ipv4.ip_forward=1' | sudo tee /etc/sysctl.d/99-wireguard.conf
sudo sysctl -p /etc/sysctl.d/99-wireguard.conf
```

**5-qadam. Firewall — `51820/udp` ochish:**

```bash
sudo ufw allow 51820/udp
sudo ufw reload
```

**6-qadam. Client config (`/etc/wireguard/wg0.conf` client mashinada):**

```ini
# /etc/wireguard/wg0.conf  (CLIENT)
[Interface]
Address = 10.0.0.2/24
PrivateKey = <CLIENT_PRIVATE_KEY>
DNS = 1.1.1.1

[Peer]
PublicKey = <SERVER_PUBLIC_KEY>
Endpoint = <SERVER_PUBLIC_IP>:51820
# Full tunnel: 0.0.0.0/0  |  Split tunnel: faqat 10.0.0.0/24
AllowedIPs = 0.0.0.0/0
PersistentKeepalive = 25
```

**7-qadam. Tunnelni ishga tushirish (`wg-quick`):**

```bash
# Serverda
sudo wg-quick up wg0
sudo systemctl enable wg-quick@wg0   # yuklanishda avtomatik

# Clientda
sudo wg-quick up wg0

# Holatni tekshirish
sudo wg show
```

Test:

```bash
# Client da — IP endi serverniki bo'lishi kerak (full tunnel bo'lsa)
curl https://ifconfig.me
ping 10.0.0.1
```

**⚠️ Ehtiyot bo'l:** `PersistentKeepalive = 25` — NAT orqasidagi client uchun muhim. Usiz NAT jadvalining "eskirishi" sabab server clientga trafik yubora olmay qolishi mumkin. Server tomonida bu kerak emas.

### Tayyor yechim: Tailscale va Netbird

Config, kalit, NAT bilan ovora bo'lmaslik uchun **mesh VPN** yechimlari bor. Ular WireGuard ustiga qurilgan, lekin ulanish, kalit almashuv va NAT traversal ni avtomatlashtiradi.

**Tailscale** (eng oson):

```bash
# Har bir qurilmada
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up
# Brauzer orqali login qiling — barcha qurilmalaringiz avtomatik bir tarmoqda
```

**Netbird** (ochiq kodli, o'z-hosting mumkin):

```bash
curl -fsSL https://pkgs.netbird.io/install.sh | sh
sudo netbird up --setup-key <SETUP_KEY>
```

**💡 Tushuncha:** Tailscale/Netbird — **mesh** (to'r) VPN. Qurilmalar markaziy server orqali emas, imkon qadar **to'g'ridan-to'g'ri** (peer-to-peer) ulanadi. Bu tezroq va bitta VPS "chokepoint" bo'lmasligini ta'minlaydi. Boshlovchilar uchun Tailscale — eng oson yo'l.

---

## VPN vs proxy vs SSH tunnel

| Xususiyat | VPN | Proxy (HTTP/SOCKS) | SSH tunnel |
|---|---|---|---|
| Qamrov | Butun mashina (barcha ilova) | Odatda bitta ilova | Bitta port / SOCKS |
| Ish darajasi | Tarmoq (IP) | Ilova | Ilova / TCP |
| Shifrlash | Ha (to'liq) | Odatda yo'q (HTTP proxy) | Ha |
| UDP | Ha | Cheklangan | Yo'q |
| Sozlash | Server + client | Oddiy | Bir buyruq |
| Use-case | Doimiy xavfsiz tarmoq | Trafik yo'naltirish | Tez, bitta xizmat |

**💡 Tushuncha:** Oddiy proxy (ayniqsa HTTP proxy) trafikni shifrlamaydi — u faqat yo'naltiradi. SSH tunnel shifrlaydi, lekin asosan bitta xizmat/TCP uchun. VPN butun mashina trafigini shifrlab tunnel qiladi — eng keng qamrovli.

---

## Tijorat VPN vs o'z VPN

**Tijorat VPN** (NordVPN, ExpressVPN, ProtonVPN va h.k.):

- **Afzalliklari:** oson (ilova o'rnatib, tugma bosasiz), ko'p davlatda serverlar, tezkor sozlash, "no-logs" siyosati (odatda).
- **Kamchiliklari:** provayderga ishonish kerak, oylik to'lov, ba'zi saytlar VPN IP larni bloklaydi, maxfiylik provayderga bog'liq.

**O'z VPN** (o'z VPS + WireGuard yoki Tailscale):

- **Afzalliklari:** to'liq nazorat, arzon (VPS narxi), o'z ichki resurslaringizga kirish, hech kimga ishonish shart emas.
- **Kamchiliklari:** o'zingiz sozlaysiz/qo'llab-quvvatlaysiz, bitta joylashuv (IP), texnik bilim kerak.

**💡 Tushuncha:** Maqsad **ichki resurslarga kirish yoki ochiq WiFi da xavfsizlik** bo'lsa — o'z VPN (Tailscale/WireGuard) eng yaxshisi. Maqsad **ko'p davlatdagi geo-kontent** bo'lsa — tijorat VPN qulayroq.

---

## DNS leak va kill switch

**DNS leak** — VPN yoqilgan bo'lsa-da, DNS so'rovlari VPN dan tashqarida (masalan, provayder DNS iga) ketishi. Bu real joylashuvingizni oshkor qiladi.

```bash
# WireGuard client config da DNS ni belgilang
[Interface]
DNS = 1.1.1.1

# Tekshirish uchun:
# https://dnsleaktest.com ga kiring yoki:
resolvectl status
```

**Kill switch** — VPN ulanishi uzilsa, butun internet trafigini bloklaydigan mexanizm. Bu tunneldan tashqari "sizib chiqishning" oldini oladi.

```bash
# Oddiy kill switch (WireGuard config ichida, PostUp/PreDown bilan)
# yoki ufw orqali: faqat wg0 interfeysiga ruxsat, boshqa hammasini bloklash
```

**⚠️ Ehtiyot bo'l:** DNS leak — maxfiylikda eng ko'p uchraydigan xato. VPN yoqib ham DNS provayderdan o'tsa, "yashirin" bo'lganingiz yolg'on bo'ladi. Har doim `dnsleaktest.com` bilan tekshiring.

---

## Xavfsizlik va maxfiylik

1. **Kalitlarni himoyalang** — WireGuard private kalitlarni hech kimga bermang, `chmod 600` qiling.
2. **Faqat kerakli portni oching** — `51820/udp` (WireGuard) dan boshqasini yopiq tuting.
3. **Server yangilanib tursin** — `apt upgrade`, xavfsizlik yamoqlari.
4. **DNS leak va kill switch** ni sozlang.
5. **Kirishlarni cheklang** — `AllowedIPs` bilan har bir clientga aniq IP bering.
6. **Log lar** — o'z serveringizda ham keraksiz log yig'maslikka harakat qiling (maxfiylik).
7. **MFA/kalit almashuv** — Tailscale/Netbird da SSO va qurilma nazorati imkoniyatlaridan foydalaning.

```bash
chmod 600 /etc/wireguard/wg0.conf
```

**💡 Tushuncha:** VPN xavfsizligi ikki qismdan iborat: **tunnel shifrlangan bo'lishi** (protokol hal qiladi) va **serverga kimlar ulana olishi** (kalit/config nazorati). Ikkinchisini e'tiborsiz qoldirmang — kuchli shifr ham noto'g'ri sozlangan kirish ro'yxatini qutqarmaydi.

---

## Savol-javob (Q&A)

### ❓ WireGuard va OpenVPN dan qaysi birini tanlay?

**✅ Javob:** Aksariyat hollarda **WireGuard** — u tezroq, oddiyroq va xavfsiz. Faqat qattiq firewall (faqat TCP/443 ruxsat) yoki juda moslashuvchan korporativ konfiguratsiya kerak bo'lsa OpenVPN (TCP rejimida) afzal.

### ❓ WireGuard da `AllowedIPs` aslida nimani anglatadi?

**✅ Javob:** Ikki narsani: (1) qaysi manzillarga trafik shu peer orqali yo'naltiriladi (routing), (2) shu peerdan qaysi manzillardan kelgan trafik qabul qilinadi (access control). `0.0.0.0/0` = full tunnel, `10.0.0.0/24` = faqat ichki tarmoq (split tunnel).

### ❓ `PersistentKeepalive` nima uchun kerak?

**✅ Javob:** NAT orqasidagi client uchun. NAT jadvali bo'sh ulanishlarni eskirtiradi; `PersistentKeepalive = 25` har 25 soniyada kichik paket yuborib, "teshik"ni ochiq saqlaydi. Odatda faqat client tomonida yoziladi.

### ❓ Full tunnel va split tunnel orasidagi farq nima?

**✅ Javob:** Full tunnel (`AllowedIPs = 0.0.0.0/0`) — butun trafik VPN orqali (maksimal himoya, sekinroq). Split tunnel (`AllowedIPs = 10.0.0.0/24`) — faqat kerakli tarmoq VPN dan, qolgani to'g'ridan-to'g'ri (tezroq, qisman himoya).

### ❓ Tailscale WireGuard dan farq qiladimi?

**✅ Javob:** Tailscale — WireGuard **ustiga** qurilgan. WireGuard shifrlash/tunnel ni beradi, Tailscale esa kalit almashuv, qurilmalarni topish, NAT traversal va mesh routing ni avtomatlashtiradi. Ya'ni Tailscale — "oson qilingan WireGuard".

### ❓ O'z VPN serverim kerakmi yoki tijorat VPN yetarlimi?

**✅ Javob:** Ichki resurslarga kirish yoki ochiq WiFi da xavfsizlik uchun — o'z VPN (Tailscale/WireGuard) yaxshiroq va arzon. Ko'p davlatdagi geo-kontent kerak bo'lsa — tijorat VPN qulayroq.

### ❓ VPN meni to'liq anonim qiladimi?

**✅ Javob:** Yo'q. VPN provayderi (yoki o'z serveringiz) trafikni ko'ra oladi, saytlar sizni cookie/login orqali taniydi. VPN faqat IP ni yashiradi va trafikni shifrlaydi. To'liq anonimlik uchun VPN yetarli emas (Tor va boshqa choralar kerak).

### ❓ DNS leak nima va qanday oldini olaman?

**✅ Javob:** DNS leak — VPN yoqiq bo'lsa ham DNS so'rovlar VPN dan tashqarida ketishi. Client config da `DNS = 1.1.1.1` belgilang va `dnsleaktest.com` bilan tekshiring. Bu real joylashuvingiz oshkor bo'lishining oldini oladi.

### ❓ Kill switch nima?

**✅ Javob:** VPN uzilib qolsa, butun internet trafigini bloklaydigan mexanizm. Bu VPN uzilgan paytda trafik himoyasiz "sizib chiqishi"ning oldini oladi. Firewall (ufw/iptables) yoki VPN client sozlamalari orqali qilinadi.

### ❓ WireGuard qaysi port va protokoldan foydalanadi?

**✅ Javob:** Default `51820/UDP`. UDP bo'lgani uchun tez, lekin ba'zi qattiq firewalllar UDP ni bloklashi mumkin — bunday holda OpenVPN TCP/443 afzalroq bo'ladi.

### ❓ Site-to-site VPN nima?

**✅ Javob:** Ikki butun tarmoqni (masalan, ikki ofis yoki ofis va cloud) bitta xususiy tarmoqqa birlashtirish. Alohida qurilma emas, tarmoq shlyuzlari (gateway) o'zaro tunnel quradi va ikkala tarmoq bir-birini "ko'radi".

### ❓ VPN va SSH tunnel farqi nimada?

**✅ Javob:** SSH tunnel — asosan bitta port/xizmat, tez, TCP. VPN — butun mashina trafigini (TCP va UDP) tarmoq darajasida shifrlab tunnel qiladi. Doimiy, keng qamrovli xavfsizlik uchun VPN; tez, bir martalik xizmat uchun SSH tunnel.

### ❓ Serverimda IP forwarding yoqilmasa nima bo'ladi?

**✅ Javob:** Clientlar VPN ga ulanadi, lekin internetga (yoki boshqa tarmoqqa) chiqa olmaydi — server paketlarni uzatmaydi. `net.ipv4.ip_forward=1` va NAT (MASQUERADE) qoidasi shart.

### ❓ Bir nechta client qo'shsam nima o'zgaradi?

**✅ Javob:** Server config ga har bir client uchun alohida `[Peer]` bloki (uning public kaliti va noyob `AllowedIPs`, masalan `10.0.0.3/32`) qo'shasiz. Har bir client o'z private kaliti va noyob IP si bilan sozlanadi.

### ❓ WireGuard config o'zgartirsam, qayta ishga tushirish kerakmi?

**✅ Javob:** Ha, oddiy usul: `sudo wg-quick down wg0 && sudo wg-quick up wg0`. Yoki jonli o'zgartirish uchun `wg syncconf` dan foydalanish mumkin, lekin boshlovchilar uchun `wg-quick` qayta ishga tushirish osonroq.

---

## Masalalar

> Yechimlar: [solutions/remote-access/02-vpn.md](../solutions/remote-access/02-vpn.md)

1. **Tushunchalar:** VPN ning uchta asosiy komponentini (tunnel, shifrlash, virtual interfeys) o'z so'zlaringiz bilan, har biriga bittadan real misol bilan tushuntiring.

2. **Protokol tanlash:** Quyidagi uch stsenariyda qaysi protokolni (WireGuard / OpenVPN / IKEv2) tanlaysiz va nega: (a) mobil telefon, tez-tez WiFi↔4G almashadi; (b) faqat TCP/443 ochiq korporativ firewall; (c) uy uchun tez, oddiy shaxsiy VPN.

3. **Kalit generatsiyasi:** WireGuard uchun server va client kalitlarini generatsiya qiladigan buyruqlarni yozing va qaysi kalit qayerda saqlanishini tushuntiring.

4. **Server config:** `10.0.0.1/24` manzilli, `51820`-portda ishlaydigan va bitta clientni (`10.0.0.2`) qabul qiladigan to'liq WireGuard server config faylini yozing (NAT/MASQUERADE bilan).

5. **Client config — full va split:** Bir xil server uchun ikkita client config yozing: (a) full tunnel (`0.0.0.0/0`), (b) split tunnel (faqat `10.0.0.0/24`). Farqni izohlang.

6. **Ikkinchi client:** Mavjud serverga ikkinchi clientni (`10.0.0.3`) qo'shish uchun server config ga qanday `[Peer]` bloki qo'shilishini va yangi client configni yozing.

7. **Diagnostika:** Client WireGuard ga ulandi (`wg show` handshake ko'rsatyapti), lekin internetga chiqa olmayapti. 3 ta ehtimoliy sababni va tekshirish/tuzatish usulini yozing.

8. **DNS leak:** Client config da DNS ni to'g'ri sozlang va DNS leak yo'qligini qanday tekshirishni tushuntiring.

9. **Tailscale:** Uch qurilmani (laptop, telefon, VPS) bitta xususiy tarmoqqa Tailscale bilan birlashtirish qadamlarini yozing va u nega WireGuard config ga qaraganda oson ekanligini tushuntiring.

10. **Taqqoslash:** VPN, HTTP proxy va SSH tunnel ni "shifrlash", "qamrov" va "use-case" bo'yicha o'z so'zlaringiz bilan taqqoslovchi jadval tuzing.

---

← [Remote Access bo'limiga qaytish](./README.md)
