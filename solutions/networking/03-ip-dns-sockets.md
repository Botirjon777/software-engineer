# IP, DNS, Port va Socket — Yechimlar

Bu fayl [`networking/03-ip-dns-sockets.md`](../../networking/03-ip-dns-sockets.md) dagi "Masalalar" bo'limining yechimlarini o'z ichiga oladi.

## Mundarija

- [1. CIDR hisoblash](#1-cidr-hisoblash)
- [2. Bir subnetdami?](#2-bir-subnetdami)
- [3. Private yoki public?](#3-private-yoki-public)
- [4. NAT debug](#4-nat-debug)
- [5. Record turini tanlash](#5-record-turini-tanlash)
- [6. Resolution zanjiri](#6-resolution-zanjiri)
- [7. TTL strategiyasi](#7-ttl-strategiyasi)
- [8. dig buyrug'i yozish](#8-dig-buyrugi-yozish)
- [9. URL ssenariysi](#9-url-ssenariysi)
- [10. Socket to'rtligi](#10-socket-tortligi)

---

## 1. CIDR hisoblash

`10.20.30.0/26` — birinchi 26 bit network, qolgan 6 bit host.

- **(a) Foydalaniladigan host:** 2^(32−26) = 2⁶ = 64 manzil, shundan network va broadcast ayrilib **62 ta** foydalaniladigan host.
- **(b) Diapazon:** `10.20.30.0` – `10.20.30.63`. Network address = `10.20.30.0`, broadcast = `10.20.30.63`, foydalaniladigan: `.1`–`.62`.
- **(c) Subnet mask:** `/26` = `11111111.11111111.11111111.11000000` = **`255.255.255.192`**.
- **(d) `10.20.30.70` shu tarmoqdami?** Yo'q. `.70` > `.63`, ya'ni keyingi blokda (`10.20.30.64/26`, diapazon `.64`–`.127`).

**Izoh:** Block hajmi 64 bo'lgani uchun `/26` bloklari `.0`, `.64`, `.128`, `.192` dan boshlanadi.

---

## 2. Bir subnetdami?

`/24` da network qismi birinchi 3 oktet — ular bir xil bo'lsa, bir subnetda.

- **(a) `192.168.5.10` va `192.168.5.200`** — ✅ bir subnetda (`192.168.5` mos).
- **(b) `192.168.5.10` va `192.168.6.10`** — ❌ turli subnet (3-oktet `5` ≠ `6`).
- **(c) `10.0.0.1` va `10.0.1.1`** — ❌ turli subnet (3-oktet `0` ≠ `1`).

**Izoh:** Agar mask `/16` bo'lganida (c) bir subnetda bo'lardi (`10.0` mos). Subnet aniqlash har doim prefiks uzunligiga bog'liq.

---

## 3. Private yoki public?

| IP | Tasnif | Sabab |
|----|--------|-------|
| `8.8.8.8` | **public** | Google DNS, internetda ko'rinadi |
| `172.16.5.4` | **private** | `172.16.0.0/12` ichida |
| `192.168.0.1` | **private** | `192.168.0.0/16` ichida |
| `172.32.0.1` | **public** | `172.32` > `172.31`, private diapazondan tashqarida |
| `127.0.0.1` | **maxsus (loopback)** | `127.0.0.0/8`, public ham private ham emas |
| `203.0.113.9` | **public** (lekin documentation-reserved) | rasmiy public diapazon |

**Izoh:** Private `172` diapazoni faqat `172.16.0.0` – `172.31.255.255`. `172.32.x.x` allaqachon undan tashqarida. `203.0.113.0/24` — hujjatlar uchun ajratilgan, lekin formati bo'yicha public.

---

## 4. NAT debug

**Muammo:** `192.168.1.10:3000` — bu **private IP**, faqat sizning lokal tarmog'ingizda ko'rinadi. Do'stingiz internetdan unga to'g'ridan-to'g'ri ulanolmaydi, chunki paket sizning private manzilingizga marshrutlanmaydi va router NAT orqasida turibsiz.

**Yechimlar (kamida 2 ta):**

1. **Port forwarding** — router'da `203.0.113.5:3000` → `192.168.1.10:3000` qoidasini sozlash. Endi do'stingiz sizning public IP'ingizga ulanadi.
2. **Tunnel servis** — `ngrok`, `cloudflared` kabi vosita lokal portni public URL bilan ochib beradi (NAT'ni chetlab o'tadi):
   ```bash
   ngrok http 3000
   ```
3. **Cloud deploy** — API'ni public IP'li serverga (VPS, cloud) joylash.
4. **VPN** — ikkalangiz bir VPN tarmog'ida bo'lsangiz, private IP ko'rinadi.

**Qo'shimcha tekshiruv:** firewall 3000-portni bloklamaganini, server `0.0.0.0:3000` (faqat `127.0.0.1` emas) ga bind qilganini ham tekshiring.

---

## 5. Record turini tanlash

| Holat | Record |
|-------|--------|
| (a) `shop.example.com` → `93.184.216.34` | **A** |
| (b) `www` ni asosiy domenga alias | **CNAME** |
| (c) email'ni `mx.mailhost.com` orqali olish | **MX** |
| (d) Google domen egaligini tasdiqlash | **TXT** |
| (e) IPv6 manzil | **AAAA** |
| (f) qaysi nameserver mas'ul | **NS** |

**Izoh:** (b) da agar `www` apex domen bo'lsa (subdomain emas), CNAME ishlamaydi — A record yoki provayder ALIAS kerak. (c) da MX yozuvi prioritet bilan keladi (`10 mx.mailhost.com`).

---

## 6. Resolution zanjiri

`blog.startup.uz` (kesh bo'sh):

```text
1. Brauzer → Recursive resolver:  "blog.startup.uz IP'si?"
   resolver keshini tekshiradi → yo'q

2. Resolver → Root nameserver:  ".uz kim?"
   Root javob:  ".uz TLD serverlardan so'ra: <ns>.uz"

3. Resolver → .uz TLD nameserver:  "startup.uz kim?"
   TLD javob:  "authoritative: ns1.startup.uz"

4. Resolver → Authoritative NS (startup.uz):  "blog.startup.uz A?"
   Authoritative javob:  "A 185.x.x.x"

5. Resolver → Brauzer:  "185.x.x.x"  (va TTL gacha keshlaydi)
```

**Izoh:** Resolver "recursive" (butun ishni qiladi), root/TLD/authoritative "iterative" (referral — "men bilmayman, anavidan so'ra") javob beradi. Real hayotda root va TLD javoblari ko'pincha keshdan keladi.

---

## 7. TTL strategiyasi

Downtime'ni minimallashtirish uchun bosqichlar:

```text
1. Ko'chirishdan ~24–48 soat OLDIN:
   eski A record'ning TTL'ini pasaytiring (masalan 86400 → 300 soniya).
   → Endi keshlar tez yangilanadi.

2. Eski TTL muddati o'tishini kuting (eski yuqori TTL tarqalib bo'lishini).

3. Yangi serverni tayyorlang, ikkala server ham vaqtincha ishlasin
   (data sinxron yoki read-only oyna).

4. DNS A record'ni yangi IP'ga o'zgartiring.
   → Past TTL (300s) tufayli o'zgarish ~5 daqiqada tarqaladi.

5. Trafik to'liq yangi serverga o'tganini monitoring qiling
   (eski serverda kelayotgan so'rovlar nolga tushishini kuting).

6. Hammasi barqaror bo'lgach, TTL'ni qaytadan ko'taring (300 → 3600/86400).
```

**Izoh:** Eng katta xato — TTL'ni o'zgartirishdan oldin emas, ko'chirish payti pasaytirish. Bunda eski yuqori TTL tufayli ba'zi foydalanuvchilar eski IP'da soatlab qolib ketadi.

---

## 8. dig buyrug'i yozish

```bash
# (a) MX yozuvlari
dig example.com MX

# (b) faqat IP (qisqa)
dig +short example.com

# (c) 1.1.1.1 resolver orqali AAAA
dig @1.1.1.1 example.com AAAA

# (d) reverse DNS (PTR)
dig -x 93.184.216.34

# (e) butun resolution zanjiri
dig +trace example.com
```

**Izoh:** `+short` faqat javob qiymatini chiqaradi; `@<resolver>` aniq DNS serverni tanlaydi; `-x` PTR so'rovini avtomatik teskari formatda yuboradi; `+trace` keshni chetlab root'dan boshlaydi.

---

## 9. URL ssenariysi

DNS ishlagan (IP topilgan), lekin sahifa ochilmadi — keyingi bosqichlarda muammo:

1. **TCP ulanish o'rnatilmayapti** — server o'chiq, port yopiq yoki firewall bloklayapti.
   - Tekshirish: `telnet <ip> 443` yoki `nc -zv <ip> 443` — ulanish ochilyaptimi?
2. **TLS handshake muvaffaqiyatsiz** — sertifikat muddati o'tgan, noto'g'ri domen yoki ishonilmagan CA.
   - Tekshirish: `openssl s_client -connect <host>:443` — sertifikat xatosini ko'rsatadi; brauzerda "NET::ERR_CERT..." xatosi.
3. **HTTP darajasidagi xato** — server `5xx` qaytaryapti yoki cheksiz redirect.
   - Tekshirish: `curl -v https://<host>` — status kod va headerlarni ko'rish.
4. **Noto'g'ri IP'ga ulandi** — DNS eski/noto'g'ri yozuv qaytargan (stale kesh).
   - Tekshirish: `dig +short <host>` natijasi haqiqiy server IP'siga mosmi?

**Izoh:** Tartib bilan tekshirish foydali: DNS → TCP → TLS → HTTP. Qaysi bosqichda to'xtaganini topib, muammoni lokalizatsiya qilish kerak.

---

## 10. Socket to'rtligi

3 parallel ulanish bir xil source IP, dest IP va dest port'ga ega — ular faqat **source port** bilan farqlanadi (OS har bir yangi ulanishga boshqa ephemeral port beradi):

```text
Ulanish 1:  192.168.1.10 : 51514  →  93.184.216.34 : 443
Ulanish 2:  192.168.1.10 : 51515  →  93.184.216.34 : 443
Ulanish 3:  192.168.1.10 : 51520  →  93.184.216.34 : 443
                          ▲ faqat shu (source port) farq qiladi
```

**Izoh:** 4-tuple = (source IP, source port, dest IP, dest port). Uchovida 3 ta element bir xil, faqat source port turlicha — shu yetadi unikallik uchun. Shuning uchun ham bitta mashina bitta serverga ko'p ulanish ocha oladi (ephemeral portlar tugab qolmaguncha).

---

← [Networking bo'limiga qaytish](../../networking/README.md)
