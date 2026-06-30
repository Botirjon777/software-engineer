# IP, DNS, Port va Socket

Internetda har bir mashina qanday topiladi, ulanish qaysi ilovaga yo'naltiriladi va `google.com` kabi nom qanday qilib raqamli manzilga aylanadi — bularning hammasi IP, port, socket va DNS tushunchalari bilan bog'liq. Bu mavzu deyarli har bir networking intervyusining poydevori: junior'dan "DNS nima?" so'ralsa, mid/senior'dan butun "URL kiritilgandan sahifa ochilgungacha" zanjirini tushuntirib berish talab qilinadi.

## Mundarija

- [IP address: IPv4 vs IPv6](#ip-address-ipv4-vs-ipv6)
- [Subnet va CIDR notation](#subnet-va-cidr-notation)
- [Private vs public IP](#private-vs-public-ip)
- [NAT qanday ishlaydi](#nat-qanday-ishlaydi)
- [Port nima va well-known portlar](#port-nima-va-well-known-portlar)
- [Socket nima](#socket-nima)
- [DNS nima va nega kerak](#dns-nima-va-nega-kerak)
- [DNS record turlari](#dns-record-turlari)
- [DNS resolution jarayoni](#dns-resolution-jarayoni)
- [DNS caching va TTL](#dns-caching-va-ttl)
- [URL kiritilgandan sahifa ochilgungacha](#url-kiritilgandan-sahifa-ochilgungacha)
- [dig va nslookup misollari](#dig-va-nslookup-misollari)
- [Intervyu savollari (Q&A)](#intervyu-savollari-qa)
- [Masalalar](#masalalar)

---

## IP address: IPv4 vs IPv6

**💡 Tushuncha:** IP address (Internet Protocol address) — tarmoqdagi har bir qurilmaning unikal manzili. Pochta yuborish uchun uy manzili kerak bo'lgani kabi, paket (packet) yuborish uchun ham manba (source) va manzil (destination) IP kerak.

**IPv4** — 32 bitlik manzil, 4 ta oktet (har biri 0–255) sifatida yoziladi:

```text
192.168.1.10
└─┬─┘ └─┬─┘
oktet  oktet   (har biri 8 bit = 1 bayt, jami 32 bit)
```

- Jami manzillar soni: 2³² ≈ 4.3 milliard. Bu internet o'sishi bilan **tugab qoldi** (IPv4 exhaustion) — shu sababli NAT va IPv6 paydo bo'lgan.

**IPv6** — 128 bitlik manzil, 8 ta guruh hexadecimal sifatida, `:` bilan ajratiladi:

```text
2001:0db8:85a3:0000:0000:8a2e:0370:7334
```

- Qisqartirish qoidalari: yetakchi nollarni (`0db8` → `db8`) tashlash mumkin; ketma-ket nol guruhlarni bir marta `::` bilan almashtirish mumkin:

```text
2001:db8:85a3::8a2e:370:7334
```

- Jami manzillar: 2¹²⁸ ≈ 3.4×10³⁸ — amalda cheksiz.

| Xususiyat | IPv4 | IPv6 |
|-----------|------|------|
| Bit | 32 | 128 |
| Yozilishi | decimal, nuqta | hexadecimal, ikki nuqta |
| Manzil soni | ~4.3 mlrd | ~3.4×10³⁸ |
| NAT | ko'p ishlatiladi | odatda kerak emas |
| Header | murakkabroq | soddalashtirilgan |
| Misol | `8.8.8.8` | `2001:4860:4860::8888` |

**⚠️ Ehtiyot bo'l:** IPv4 va IPv6 bir-biriga to'g'ridan-to'g'ri mos kelmaydi (incompatible). Ko'p tizimlar **dual-stack** ishlaydi — bir vaqtda IPv4 ham, IPv6 ham qo'llab-quvvatlanadi.

---

## Subnet va CIDR notation

**💡 Tushuncha:** Har bir IP manzil ikki qismdan iborat: **network qismi** (qaysi tarmoq) va **host qismi** (o'sha tarmoqdagi qaysi qurilma). Subnet — IP fazoni mantiqiy tarmoqchalarga bo'lish.

**CIDR notation** (Classless Inter-Domain Routing) — `IP/prefix` ko'rinishida network qismining uzunligini bildiradi. Masalan `/24` — birinchi 24 bit network, qolgan 8 bit host:

```text
192.168.1.0/24
┌──────────────────┬────────┐
│  network (24 bit)│host(8) │
│  192.168.1       │  .0    │
└──────────────────┴────────┘

Diapazon:  192.168.1.0  →  192.168.1.255
Host soni: 2^(32-24) = 256 (shundan 254 ta foydalanish mumkin)
```

- `.0` — network address (tarmoqning o'zi), `.255` — broadcast address. Shuning uchun 256 − 2 = 254 ta foydalaniladigan host.

Boshqa keng tarqalgan prefikslar:

| CIDR | Subnet mask | Host soni | Foydalaniladigan |
|------|-------------|-----------|------------------|
| `/8` | 255.0.0.0 | 16 777 216 | 16 777 214 |
| `/16` | 255.255.0.0 | 65 536 | 65 534 |
| `/24` | 255.255.255.0 | 256 | 254 |
| `/30` | 255.255.255.252 | 4 | 2 |
| `/32` | 255.255.255.255 | 1 | 1 (bitta host) |

**💡 Tushuncha:** Prefiks qancha katta bo'lsa (`/24` → `/30`), tarmoq shuncha kichik. `/32` — aniq bitta IP (masalan firewall qoidasida bitta manzilga ruxsat berish uchun).

**⚠️ Ehtiyot bo'l:** Ikki IP bir subnetdami yoki yo'qmi aniqlash uchun ularning network qismi bir xilligini tekshiriladi. `192.168.1.10/24` va `192.168.2.10/24` — turli subnetlarda (3-oktet farq qiladi), shuning uchun ular o'zaro to'g'ridan-to'g'ri emas, router orqali gaplashadi.

---

## Private vs public IP

**💡 Tushuncha:** **Public IP** — butun internetda unikal, dunyoning istalgan joyidan ko'rinadi. **Private IP** — faqat lokal tarmoq ichida (uy, ofis) ishlaydi, internetga to'g'ridan-to'g'ri chiqmaydi.

RFC 1918 bo'yicha private diapazonlar (IPv4):

```text
10.0.0.0/8        →  10.0.0.0    – 10.255.255.255
172.16.0.0/12     →  172.16.0.0  – 172.31.255.255
192.168.0.0/16    →  192.168.0.0 – 192.168.255.255
```

- Uy router'ingiz odatda `192.168.0.0/16` dan manzil beradi (masalan `192.168.1.x`).
- Bu private manzillar internetda **takrorlanadi** — millionlab uyda `192.168.1.1` bor, lekin ular bir-birini ko'rmaydi, chunki har biri o'z lokal tarmog'ida.

**⚠️ Ehtiyot bo'l:** Private IP'li server internetdan to'g'ridan-to'g'ri ochib bo'lmaydi — uni public qilish uchun port forwarding, NAT yoki reverse proxy kerak. Intervyu savoli: "Nega `192.168.1.5` da turgan API'mga do'stim ulanolmayapti?" — chunki bu private manzil, faqat sizning lokal tarmog'ingizda ishlaydi.

Yana bir maxsus diapazon — **loopback**: `127.0.0.0/8` (eng ko'p `127.0.0.1`, ya'ni `localhost`) — qurilmaning o'ziga ishora qiladi.

---

## NAT qanday ishlaydi

**💡 Tushuncha:** NAT (Network Address Translation) — bir nechta private IP'li qurilmalarni bitta public IP orqali internetga ulashga imkon beradi. Bu IPv4 manzillari tugaganligi muammosini hal qiladi.

Misol: uyda 3 ta qurilma bor, lekin internet-provayder sizga bitta public IP bergan (`203.0.113.5`).

```text
Lokal tarmoq (private)                 Internet (public)
┌───────────────────────┐
│ Laptop 192.168.1.10    │──┐
│ Telefon 192.168.1.11   │──┤   ┌────────┐
│ TV     192.168.1.12    │──┼──▶│ ROUTER │──▶  203.0.113.5  ──▶ google.com
└───────────────────────┘  │   │  (NAT) │
                           │   └────────┘
   source:port            translation jadvali
   192.168.1.10:51514  →  203.0.113.5:40001
   192.168.1.11:33012  →  203.0.113.5:40002
```

Qadamlar:

1. Laptop (`192.168.1.10:51514`) `google.com` ga so'rov yuboradi.
2. Router so'rovni ushlaydi, source manzilni o'z public IP'siga (`203.0.113.5:40001`) almashtiradi va **translation jadvaliga** yozadi.
3. Google javobi `203.0.113.5:40001` ga keladi.
4. Router jadvaldan qaraydi: `40001` → `192.168.1.10:51514`, javobni laptopga qaytaradi.

Bu aniqrog'i **PAT** (Port Address Translation) yoki **NAPT** deyiladi — port raqami orqali qaysi qurilma ekanligi ajratiladi.

**⚠️ Ehtiyot bo'l:** NAT orqasidagi qurilmaga tashqaridan ulanish qiyin (NAT traversal muammosi) — shuning uchun P2P, video-qo'ng'iroq tizimlari STUN/TURN serverlardan foydalanadi. Shuningdek, server log'larida ko'p foydalanuvchi bitta IP'dan ko'rinishi mumkin (hammasi bir NAT orqasida).

---

## Port nima va well-known portlar

**💡 Tushuncha:** IP manzil mashinani topadi, ammo bitta mashinada o'nlab ilova ishlashi mumkin (web server, SSH, DB...). **Port** — qaysi ilovaga so'rovni yetkazishni bildiradigan raqam (0–65535). IP — bino manzili, port — kvartira raqami.

Port diapazonlari:

| Diapazon | Nomi | Misol |
|----------|------|-------|
| 0–1023 | well-known (tizim) | HTTP 80, HTTPS 443 |
| 1024–49151 | registered | PostgreSQL 5432, MySQL 3306 |
| 49152–65535 | ephemeral (dinamik) | client tomonidan vaqtincha tanlanadi |

Eng muhim well-known portlar:

```text
20/21  FTP        80   HTTP      443  HTTPS
22     SSH        110  POP3      587  SMTP (submission)
25     SMTP       143  IMAP      993  IMAPS
53     DNS        3306 MySQL     5432 PostgreSQL
6379   Redis      27017 MongoDB  8080 HTTP (alt)
```

**💡 Tushuncha:** `https://example.com` aslida `https://example.com:443` — brauzer HTTPS uchun 443 portni standart sifatida o'zi qo'shadi. `http://...` → 80.

**⚠️ Ehtiyot bo'l:** 1024 dan past portlarni ochish (bind qilish) Linux'da odatda root/administrator huquqini talab qiladi. Shuning uchun ko'p ilovalar dev rejimda 3000, 8080 kabi yuqori portlarda ishlaydi.

---

## Socket nima

**💡 Tushuncha:** Socket — bitta aniq ulanishning so'nggi nuqtasi (endpoint), ya'ni **IP + port** juftligi. Ikki socket (client va server) o'rtasida ulanish o'rnatiladi.

Bitta TCP ulanish to'rtlik (4-tuple) bilan unikal aniqlanadi:

```text
(source IP, source port, destination IP, destination port)

192.168.1.10 : 51514   ───────▶   142.250.74.78 : 443
   client socket                     server socket
```

- Shuning uchun bitta server (`:443`) minglab client bilan bir vaqtda ulanishi mumkin — har bir ulanish source IP/port bilan farqlanadi.
- Bitta brauzer ham bitta serverga bir nechta ulanish ochishi mumkin — har biri har xil source port oladi.

Socket turlari:

- **Stream socket (SOCK_STREAM)** — TCP ustida, ishonchli, tartibli.
- **Datagram socket (SOCK_DGRAM)** — UDP ustida, tez, ishonchsiz.

**⚠️ Ehtiyot bo'l:** "Socket" so'zi kontekstga qarab ikki ma'noda ishlatiladi: (1) OS API sifatida (`socket()`, `bind()`, `listen()`, `accept()`), (2) WebSocket protokoli (alohida narsa). Intervyu savolida qaysi biri nazarda tutilganini aniqlashtiring.

---

## DNS nima va nega kerak

**💡 Tushuncha:** DNS (Domain Name System) — odamga tushunarli domen nomlarini (`google.com`) mashinaga kerak bo'lgan IP manzilga (`142.250.74.78`) o'giruvchi tarmoqning "telefon kitobi".

Nega kerak:

- Odam `142.250.74.78` ni emas, `google.com` ni eslab qoladi.
- Serverning IP'si o'zgarsa, domen o'zgarmaydi — faqat DNS yozuvi yangilanadi.
- Bitta domen bir nechta IP'ga (load balancing) yoki geografik joylashuvga qarab turli IP'ga ishora qilishi mumkin.

```text
Foydalanuvchi:  "google.com ga kirmoqchiman"
                        │
                        ▼
                 ┌──────────┐
                 │   DNS    │  google.com  →  142.250.74.78
                 └──────────┘
                        │
                        ▼
       Brauzer 142.250.74.78 ga ulanadi
```

**⚠️ Ehtiyot bo'l:** DNS o'zi sahifani ochmaydi — u faqat "qaysi IP" savoliga javob beradi. Undan keyin TCP, TLS, HTTP bosqichlari bo'ladi.

---

## DNS record turlari

**💡 Tushuncha:** DNS server bir nechta turdagi yozuvlar (records) saqlaydi. Har bir tur ma'lum savolga javob beradi.

| Record | Vazifasi | Misol |
|--------|----------|-------|
| **A** | domen → IPv4 | `example.com → 93.184.216.34` |
| **AAAA** | domen → IPv6 | `example.com → 2606:2800:220:1:248:1893:25c8:1946` |
| **CNAME** | domen → boshqa domen (alias) | `www.example.com → example.com` |
| **MX** | pochta serveri | `example.com → mail.example.com (priority 10)` |
| **TXT** | ixtiyoriy matn (verifikatsiya, SPF) | `"v=spf1 include:_spf.google.com ~all"` |
| **NS** | qaysi nameserver mas'ul | `example.com → ns1.example.com` |

Qo'shimcha tez-tez uchraydiganlar: **PTR** (reverse DNS: IP → domen), **SOA** (zona haqida asosiy ma'lumot), **SRV** (servis joylashuvi).

**⚠️ Ehtiyot bo'l:** CNAME zone apex'da (ya'ni `example.com` o'zida, subdomain emas) ishlatib bo'lmaydi — buning uchun `ALIAS`/`ANAME` (provayderga xos) yoki to'g'ridan-to'g'ri A record kerak. TXT yozuvlari domen egaligini tasdiqlash (Google, SSL CA) va email autentifikatsiyasi (SPF, DKIM, DMARC) uchun keng ishlatiladi.

---

## DNS resolution jarayoni

**💡 Tushuncha:** Domen nomidan IP topish jarayoni iyerarxik. Hech qaysi bitta server butun internetni bilmaydi — javob bosqichma-bosqich, yuqoridan pastga topiladi.

`www.example.com` ni qidirish (cache bo'sh deb faraz qilaylik):

```text
                         ┌─────────────────────┐
  Brauzer ──(1)──▶       │ Recursive Resolver  │  (ISP yoki 8.8.8.8)
  www.example.com?       │   (caching qiladi)  │
                         └──────────┬──────────┘
                                    │
              (2) ".com kim?"       ▼
                         ┌─────────────────────┐
                         │   Root nameserver   │  → "TLD .com ni so'ra: a.gtld-servers.net"
                         └──────────┬──────────┘
                                    │
              (3) "example.com kim?"▼
                         ┌─────────────────────┐
                         │   TLD nameserver    │  → "authoritative: ns1.example.com"
                         │      (.com)         │
                         └──────────┬──────────┘
                                    │
              (4) "www.example.com?"▼
                         ┌─────────────────────┐
                         │ Authoritative NS    │  → "A record: 93.184.216.34"
                         │  (example.com)      │
                         └──────────┬──────────┘
                                    │
   (5) 93.184.216.34  ◀─────────────┘
       resolver brauzerga qaytaradi (va cache'ga yozadi)
```

Qadamlar:

1. Brauzer **recursive resolver** ga so'rov yuboradi (odatda ISP yoki `8.8.8.8`/`1.1.1.1`). Avval o'z keshini, OS keshini, brauzer keshini tekshiradi.
2. Keshda yo'q bo'lsa, resolver **root nameserver** dan so'raydi. Root `.com` uchun TLD serverni ko'rsatadi.
3. Resolver **TLD nameserver** (`.com`) dan so'raydi. U `example.com` uchun authoritative serverni ko'rsatadi.
4. Resolver **authoritative nameserver** dan so'raydi. U haqiqiy A record (IP) ni qaytaradi.
5. Resolver IP'ni brauzerga qaytaradi va TTL muddatigacha keshlaydi.

**💡 Tushuncha:** "Recursive" — resolver butun og'irlikni o'z zimmasiga oladi, brauzerga faqat yakuniy javobni beradi. Root/TLD/authoritative serverlar esa "iterative" (referral) javob beradi — "men bilmayman, anavi serverdan so'ra".

**⚠️ Ehtiyot bo'l:** Real hayotda bu 4 bosqichning ko'pi keshdan keladi — root va TLP javoblari uzoq vaqt keshlanadi. Shuning uchun ko'pchilik so'rovlar bir-ikki millisekundda hal bo'ladi.

---

## DNS caching va TTL

**💡 Tushuncha:** Har bir DNS yozuvida **TTL** (Time To Live) — soniyalarda muddat — bo'ladi. U yozuv qancha vaqt keshlanishini bildiradi. TTL tugagach, kesh "eskiradi" va qayta so'rov qilinadi.

Kesh bir necha qatlamda bo'ladi:

```text
Brauzer kesh  →  OS kesh (stub resolver)  →  Router kesh  →  Recursive resolver kesh
   (eng yaqin, eng tez)                                          (eng uzoq)
```

TTL tanlash savdo-sotiq (trade-off):

| TTL | Afzallik | Kamchilik |
|-----|----------|-----------|
| Past (60s) | tez o'zgartirish (migratsiya, failover) | DNS serverga ko'p yuk, sekinroq |
| Yuqori (24s) | kam so'rov, tez (keshdan) | o'zgarish sekin tarqaladi |

**⚠️ Ehtiyot bo'l:** Serverni boshqa IP'ga ko'chirayotganda, **oldindan** TTL'ni pasaytiring (masalan 24 soatdan 5 daqiqaga), o'tkazishdan keyin yana ko'taring. Aks holda eski IP keshda uzoq qoladi va ba'zi foydalanuvchilar eski serverga tushaversadi ("DNS propagation" kechikishi). `nslookup`/`dig` ba'zan keshlangan natija beradi — `dig` da "ANSWER SECTION" dagi TTL har so'rovda kamayib borishini ko'rasiz.

---

## URL kiritilgandan sahifa ochilgungacha

**💡 Tushuncha:** Bu eng mashhur intervyu savoli ("What happens when you type a URL?"). To'liq zanjir: DNS → TCP → TLS → HTTP → render. Quyida `https://example.com` uchun bosqichma-bosqich.

```text
1. URL parsing
   brauzer https://example.com ni qismlarga ajratadi:
   scheme=https, host=example.com, port=443 (default), path=/

2. DNS resolution
   example.com → 93.184.216.34
   (brauzer → OS → resolver → root → TLD → authoritative; keshdan kelishi mumkin)

3. TCP handshake (3-way)
   client ──SYN────────▶ server
   client ◀──SYN-ACK──── server
   client ──ACK────────▶ server     (ulanish o'rnatildi, :443 ga)

4. TLS handshake (HTTPS bo'lgani uchun)
   ClientHello / ServerHello / certificate / key exchange / finished
   → shifrlangan kanal o'rnatiladi, server identifikatsiyasi tekshiriladi

5. HTTP request
   GET / HTTP/1.1
   Host: example.com
   (TLS ichida shifrlangan holda yuboriladi)

6. Server javobi
   HTTP/1.1 200 OK
   Content-Type: text/html
   <html>...</html>

7. Render
   brauzer HTML'ni parse qiladi → DOM quradi →
   CSS, JS, rasm kabi qo'shimcha resurslar uchun (2–6) qadamlar takrorlanadi →
   sahifa ekranga chiqadi
```

Bosqichlar batafsil:

1. **URL parsing** — brauzer scheme, host, port, path'ni ajratadi. HSTS ro'yxatini tekshiradi (http → https ga majburiy o'tkazish mumkin).
2. **DNS** — yuqorida tavsiflangan resolution. Natija: IP manzil.
3. **TCP** — 3-way handshake bilan `IP:443` ga ishonchli ulanish o'rnatiladi.
4. **TLS** — HTTPS bo'lsa, handshake orqali shifrlash kaliti kelishiladi va sertifikat tekshiriladi (keyingi mavzu — TLS va HTTPS).
5. **HTTP request** — `GET /` so'rovi (HTTP/2 yoki HTTP/3 bo'lishi mumkin) yuboriladi.
6. **Response** — server status, headerlar va HTML body qaytaradi.
7. **Render** — DOM/CSSOM qurish, qo'shimcha resurslar (CSS, JS, rasm) uchun DNS+TCP+TLS+HTTP takrorlanadi, sahifa chiziladi.

**⚠️ Ehtiyot bo'l:** Intervyuda "qancha chuqur tushuntiray?" deb so'rang. Junior'da: DNS → ulanish → so'rov → javob → render yetadi. Senior'da har bosqichni (TCP slow start, TLS 1.3 1-RTT, HTTP/2 multiplexing, browser critical rendering path) chuqurroq ochish kutiladi.

---

## dig va nslookup misollari

**💡 Tushuncha:** `dig` (Linux/macOS) va `nslookup` (har joyda) — DNS so'rovlarini qo'lda yuborish va resolution'ni tekshirish vositalari.

A record so'rash:

```bash
# dig — batafsil chiqish
dig example.com A

# nslookup — soddaroq
nslookup example.com
```

`dig` chiqishidagi muhim qism:

```text
;; ANSWER SECTION:
example.com.    3600    IN    A    93.184.216.34
                 ▲      ▲     ▲    ▲
                TTL   class  type IP
```

Foydali variantlar:

```bash
dig example.com AAAA          # IPv6 (AAAA) record
dig example.com MX            # pochta serverlari
dig example.com NS            # nameserverlar
dig example.com TXT           # TXT yozuvlar (SPF, verifikatsiya)
dig +short example.com        # faqat javob (qisqa)
dig @8.8.8.8 example.com      # aniq resolver (Google DNS) dan so'rash
dig +trace example.com        # root → TLD → authoritative butun zanjirni ko'rsatadi
dig -x 93.184.216.34          # reverse DNS (PTR): IP → domen
```

`nslookup` da record turini ko'rsatish:

```bash
nslookup -type=MX example.com
nslookup -type=AAAA example.com
nslookup example.com 1.1.1.1     # Cloudflare resolver dan so'rash
```

**⚠️ Ehtiyot bo'l:** `dig +trace` haqiqiy iterativ resolution'ni ko'rsatadi (keshni chetlab o'tib), shuning uchun DNS muammosini debug qilishda eng foydali. `nslookup` natijasi keshlangan bo'lishi mumkin — "Non-authoritative answer" degani javob authoritative serverdan emas, resolver keshidan kelganini bildiradi.

---

## Intervyu savollari (Q&A)

### ❓ IPv4 va IPv6 farqi nima va nega IPv6 kerak bo'ldi?

**✅ Javob:** IPv4 — 32 bit (~4.3 mlrd manzil), IPv6 — 128 bit (~3.4×10³⁸ manzil). IPv4 manzillari internet o'sishi bilan tugab qoldi (IPv4 exhaustion). IPv6 deyarli cheksiz manzil, soddalashtirilgan header va NAT'ga ehtiyojni kamaytirish bilan bu muammoni hal qiladi. Hozir ko'p tizimlar dual-stack (ikkalasini ham qo'llab-quvvatlovchi) ishlaydi.

### ❓ CIDR `/24` nimani anglatadi?

**✅ Javob:** Birinchi 24 bit network qismi, qolgan 8 bit host qismi degani. Bu `192.168.1.0/24` uchun `192.168.1.0`–`192.168.1.255` diapazonini (256 manzil, shundan 254 ta foydalaniladigan) beradi. Prefiks qancha katta bo'lsa, tarmoq shuncha kichik.

### ❓ Private va public IP farqi nima?

**✅ Javob:** Public IP butun internetda unikal va ko'rinadi. Private IP (`10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`) faqat lokal tarmoq ichida ishlaydi, internetda takrorlanadi va to'g'ridan-to'g'ri chiqmaydi. Private qurilmalar internetga NAT orqali chiqadi.

### ❓ NAT qanday ishlaydi?

**✅ Javob:** NAT bir nechta private IP'li qurilmani bitta public IP orqali internetga ulaydi. Router chiquvchi paketning source IP/port'ini o'z public IP'siga almashtiradi va translation jadvaliga yozadi; javob kelganda jadvaldan asl qurilmani topib qaytaradi. Port orqali ajratilgani uchun aniqrog'i PAT/NAPT deyiladi.

### ❓ Port nima va socket nima — farqi?

**✅ Javob:** Port — mashinadagi ma'lum ilovani bildiruvchi raqam (0–65535). Socket — ulanishning so'nggi nuqtasi, ya'ni IP + port juftligi. IP mashinani, port ilovani topadi; socket esa aniq endpoint. TCP ulanish (source IP, source port, dest IP, dest port) to'rtligi bilan unikal aniqlanadi.

### ❓ HTTP va HTTPS qaysi portda ishlaydi?

**✅ Javob:** HTTP — 80, HTTPS — 443. Bular well-known portlar (0–1023). Brauzer scheme'ga qarab bu portlarni avtomatik qo'shadi (`https://x.com` = `https://x.com:443`).

### ❓ DNS nima va nega kerak?

**✅ Javob:** DNS domen nomlarini (`google.com`) IP manzilga (`142.250.74.78`) o'giruvchi tarmoq "telefon kitobi". Kerakligi: odam IP'ni emas, nomni eslaydi; serverning IP'si o'zgarsa domen o'zgarmaydi; bitta domen bir nechta IP yoki geografik joyga ishora qilishi mumkin.

### ❓ A, AAAA, CNAME, MX, TXT, NS record'lari nima uchun?

**✅ Javob:** A — domen → IPv4; AAAA — domen → IPv6; CNAME — domen → boshqa domen (alias); MX — pochta serveri; TXT — ixtiyoriy matn (SPF/verifikatsiya); NS — qaysi nameserver mas'ul. CNAME zone apex'da ishlatilmaydi.

### ❓ DNS resolution jarayonini bosqichma-bosqich tushuntiring.

**✅ Javob:** Brauzer recursive resolver'ga so'raydi (avval keshlar tekshiriladi). Keshda yo'q bo'lsa: resolver root nameserver'dan TLD serverni, TLD'dan (`.com`) authoritative serverni, authoritative'dan haqiqiy A record (IP) ni oladi. So'ng IP'ni brauzerga qaytaradi va TTL muddatiga keshlaydi. Resolver "recursive", root/TLD/authoritative "iterative" (referral) javob beradi.

### ❓ TTL nima va qachon DNS migratsiyada uni o'zgartirasiz?

**✅ Javob:** TTL — DNS yozuvi qancha vaqt keshlanishini (soniyalarda) bildiradi. Serverni boshqa IP'ga ko'chirishdan oldin TTL'ni pasaytirib qo'yiladi (masalan 5 daqiqa), shunda eski IP keshda uzoq qolmaydi va o'zgarish tez tarqaladi; ko'chirishdan keyin yana ko'tariladi.

### ❓ "URL kiritilgandan sahifa ochilgungacha" nima bo'ladi?

**✅ Javob:** (1) URL parsing va HSTS tekshirish; (2) DNS resolution (domen → IP); (3) TCP 3-way handshake; (4) HTTPS bo'lsa TLS handshake (kalit kelishuvi + sertifikat tekshiruvi); (5) HTTP request (`GET /`); (6) server javobi (status + HTML); (7) brauzer DOM/CSSOM quradi, qo'shimcha resurslar (CSS/JS/rasm) uchun bosqichlar takrorlanadi, sahifa render qilinadi.

### ❓ Recursive resolver va authoritative nameserver farqi nima?

**✅ Javob:** Recursive resolver (masalan `8.8.8.8`) butun qidiruvni o'z zimmasiga oladi, keshlaydi va brauzerga yakuniy javob beradi. Authoritative nameserver esa ma'lum domen uchun haqiqiy yozuvlarni (A, MX...) saqlovchi va to'g'ridan-to'g'ri javob beradigan manba. Resolver authoritative'dan javob oladi.

### ❓ Bitta server qanday qilib minglab clientga bir vaqtda xizmat qiladi, hammasi bir portda (`:443`)?

**✅ Javob:** Har bir ulanish (source IP, source port, dest IP, dest port) to'rtligi bilan unikal. Server porti bir xil (`:443`) bo'lsa ham, har bir client boshqa source IP yoki source port bilan keladi, shuning uchun OS ularni alohida socket sifatida ajratadi.

### ❓ `dig +trace` nima qiladi va qachon ishlatasiz?

**✅ Javob:** `dig +trace` resolution'ni keshni chetlab o'tib, root'dan boshlab TLD va authoritative serverlargacha butun zanjirni bosqichma-bosqich bajaradi va ko'rsatadi. DNS muammosini (qaysi bosqichda noto'g'ri javob kelyapti) debug qilishda eng foydali.

### ❓ Loopback address nima?

**✅ Javob:** `127.0.0.0/8` diapazoni (eng ko'p `127.0.0.1` = `localhost`) — qurilmaning o'ziga ishora qiladi. Tarmoqdan chiqmasdan, ilovaning o'z mashinasidagi boshqa ilova bilan ulanish uchun ishlatiladi.

### ❓ Subnet mask `255.255.255.0` qaysi CIDR'ga teng?

**✅ Javob:** `/24`. Mask ikkilik sanoqda `11111111.11111111.11111111.00000000` — 24 ta bir (network), 8 ta nol (host). 256 manzil, 254 foydalaniladigan host.

---

## Masalalar

> Yechimlar: [solutions/networking/03-ip-dns-sockets.md](../solutions/networking/03-ip-dns-sockets.md)

1. **CIDR hisoblash.** `10.20.30.0/26` tarmog'i uchun: (a) nechta foydalaniladigan host bor? (b) diapazon qanday? (c) subnet mask nimaga teng? (d) `10.20.30.70` shu tarmoqda bormi?

2. **Bir subnetdami?** Quyidagi juftliklar bir subnetdami (mask `/24` deb)? (a) `192.168.5.10` va `192.168.5.200`; (b) `192.168.5.10` va `192.168.6.10`; (c) `10.0.0.1` va `10.0.1.1`.

3. **Private yoki public?** Quyidagilarni private yoki public deb tasniflang: `8.8.8.8`, `172.16.5.4`, `192.168.0.1`, `172.32.0.1`, `127.0.0.1`, `203.0.113.9`.

4. **NAT debug.** Uy tarmog'ingizda 3 ta qurilma bir public IP orqali internetga chiqadi. Do'stingiz sizning laptopda ishlayotgan API'ga (`192.168.1.10:3000`) ulanolmayapti. Sabablarini va yechimlarni (kamida 2 ta) tushuntiring.

5. **Record turini tanlash.** Har bir holatga mos DNS record turini yozing: (a) `shop.example.com` ni `93.184.216.34` ga yo'naltirish; (b) `www` ni asosiy domenga alias qilish; (c) email'ni `mx.mailhost.com` orqali olish; (d) Google domen egaligini tasdiqlash; (e) IPv6 manzil; (f) qaysi nameserver mas'ulligini ko'rsatish.

6. **Resolution zanjiri.** `blog.startup.uz` so'ralganda kesh bo'sh bo'lsa, qaysi serverlar qanday tartibda so'raladi? Har bir bosqichda kim nima javob beradi — yozib chiqing.

7. **TTL strategiyasi.** Production serveringizni yangi data-markazga (yangi IP) ko'chirmoqchisiz. DNS TTL'ni qachon va qanday o'zgartirasiz, downtime'ni minimallashtirish uchun? Qadamlarni yozing.

8. **dig buyrug'i yozing.** Quyidagilarni topadigan `dig` buyruqlarini yozing: (a) `example.com` MX yozuvlari; (b) faqat IP (qisqa chiqish); (c) `1.1.1.1` resolver orqali AAAA; (d) `93.184.216.34` uchun reverse DNS; (e) butun resolution zanjiri.

9. **URL ssenariysi.** Hamkasbingiz "DNS ishladi, lekin sahifa baribir ochilmadi" deydi. URL → sahifa zanjirining qaysi keyingi bosqichlarida muammo bo'lishi mumkin? Kamida 3 ta ehtimolni va ularni qanday tekshirishni yozing.

10. **Socket to'rtligi.** Bitta client (`192.168.1.10`) bitta serverga (`93.184.216.34:443`) 3 ta parallel ulanish ochdi. Bu 3 ulanish bir-biridan nima bilan farqlanadi? 4-tuple'larni misol bilan yozing.

---

← [Networking bo'limiga qaytish](./README.md)
