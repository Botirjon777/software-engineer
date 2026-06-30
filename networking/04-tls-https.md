# TLS va HTTPS

HTTP ochiq matnda ishlaydi — har kim trafikni o'qishi va o'zgartirishi mumkin. HTTPS bu muammoni TLS (Transport Layer Security) yordamida hal qiladi: shifrlash, butunlik va serverning haqiqiyligini ta'minlaydi. Bu mavzu mid/senior networking va security intervyularining markazida — handshake, sertifikatlar va MITM himoyasi tez-tez so'raladi.

## Mundarija

- [Nega HTTPS kerak](#nega-https-kerak)
- [Symmetric vs asymmetric encryption](#symmetric-vs-asymmetric-encryption)
- [Public va private key](#public-va-private-key)
- [TLS handshake qadama-qadam](#tls-handshake-qadama-qadam)
- [TLS 1.2 vs TLS 1.3](#tls-12-vs-tls-13)
- [Digital certificate va CA](#digital-certificate-va-ca)
- [Chain of trust](#chain-of-trust)
- [Self-signed certificate](#self-signed-certificate)
- [HTTPS HTTP ustida qanday ishlaydi](#https-http-ustida-qanday-ishlaydi)
- [HSTS](#hsts)
- [SNI](#sni)
- [mTLS (mutual TLS)](#mtls-mutual-tls)
- [Certificate'ni tekshirish](#certificateni-tekshirish)
- [MITM hujumi va TLS himoyasi](#mitm-hujumi-va-tls-himoyasi)
- [Intervyu savollari (Q&A)](#intervyu-savollari-qa)
- [Masalalar](#masalalar)

---

## Nega HTTPS kerak

**💡 Tushuncha:** HTTPS (HTTP Secure) — HTTP'ni TLS ustida ishlatish. U uchta asosiy kafolat beradi:

| Kafolat | Ma'nosi | TLS qanday ta'minlaydi |
|---------|---------|------------------------|
| **Confidentiality** (maxfiylik) | Trafikni hech kim o'qiy olmaydi | Shifrlash (encryption) |
| **Integrity** (butunlik) | Trafik yo'lda o'zgartirilmagan | MAC / AEAD (autentifikatsiyalangan shifrlash) |
| **Authentication** (haqiqiylik) | Siz haqiqatan o'sha server bilan gaplashyapsiz | Sertifikat + CA imzosi |

Misol: ochiq Wi-Fi'da HTTP orqali login qilsangiz, parolingiz ochiq matnda uchadi — yonidagi odam uni ko'rishi mumkin. HTTPS bilan trafik shifrlangan, hatto Wi-Fi'ni eshitayotgan kishi ham faqat tushunarsiz baytlarni ko'radi.

**⚠️ Ehtiyot bo'l:** HTTPS faqat **transport**ni himoya qiladi (client–server orasidagi yo'lni). U serverdagi zaifliklarni (SQL injection, XSS) yoki ilovangizdagi xatolarni hal qilmaydi. "HTTPS bor, demak xavfsiz" — noto'g'ri xulosa.

---

## Symmetric vs asymmetric encryption

**💡 Tushuncha:** TLS ikkala turdagi shifrlashni birlashtiradi. Har birining o'z roli bor.

**Symmetric encryption** — bir xil kalit ham shifrlaydi, ham deshifrlaydi (masalan AES).

```text
xabar ──[kalit K bilan shifrlash]──▶ shifr ──[kalit K bilan ochish]──▶ xabar
```

- ✅ Tez (katta ma'lumot uchun ideal).
- ❌ Muammo: ikki tomon bir xil kalitni qanday xavfsiz almashadi? (key distribution).

**Asymmetric encryption** — juft kalit: public key (hamma ko'radi) va private key (faqat ega). Biri bilan shifrlangan narsa faqat ikkinchisi bilan ochiladi (masalan RSA, ECDH).

```text
public key bilan shifrlangan  ──▶  faqat private key bilan ochiladi
private key bilan imzolangan  ──▶  public key bilan tekshiriladi
```

- ✅ Key distribution muammosini hal qiladi (public key'ni ochiq berish mumkin).
- ❌ Sekin (katta ma'lumot uchun og'ir).

**💡 Tushuncha:** TLS aqlli kombinatsiya qiladi — **asymmetric** orqali xavfsiz symmetric kalitni kelishadi (handshake), keyin butun ma'lumotni tez **symmetric** bilan shifrlaydi. Ikki dunyoning eng yaxshisi.

---

## Public va private key

**💡 Tushuncha:** Asymmetric kriptografiyada matematik bog'langan ikkita kalit bo'ladi:

- **Public key** — hamma bilan baham ko'rsa bo'ladi (sertifikat ichida tarqatiladi).
- **Private key** — hech qachon, hech kimga berilmaydi (serverda maxfiy saqlanadi).

Ikki asosiy operatsiya:

```text
1. SHIFRLASH (confidentiality):
   kimdir public key bilan shifrlaydi  →  faqat private key egasi ochadi

2. IMZOLASH (authentication):
   private key egasi imzolaydi  →  hamma public key bilan tekshiradi
   (faqat haqiqiy egada private key bor → imzo haqiqiyligini isbotlaydi)
```

**⚠️ Ehtiyot bo'l:** Private key sizib chiqsa (leak), butun xavfsizlik buziladi — boshqa odam serveringizni "siz" deb ko'rsata oladi. Shuning uchun private key fayllar `600` ruxsat bilan, HSM yoki secret-manager'da saqlanadi va git'ga hech qachon commit qilinmaydi.

---

## TLS handshake qadama-qadam

**💡 Tushuncha:** TLS handshake — client va server shifrlash kalitini kelishadigan va serverni autentifikatsiya qiladigan boshlang'ich muloqot. Bu TLS 1.2 uchun soddalashtirilgan ko'rinishi:

```text
   CLIENT                                              SERVER
     │                                                   │
     │ ──── (1) ClientHello ──────────────────────────▶ │
     │   "salom, men TLS 1.2/1.3, shu cipher'lar,        │
     │    random R_c, qo'llab-quvvatlangan kalitlar"     │
     │                                                   │
     │ ◀─── (2) ServerHello ──────────────────────────── │
     │   "tanlangan cipher, random R_s"                  │
     │                                                   │
     │ ◀─── (3) Certificate ───────────────────────────  │
     │   "mana mening sertifikatim (public key + CA imzo)"│
     │                                                   │
     │ ◀─── (4) ServerHelloDone ──────────────────────   │
     │                                                   │
     │   [client sertifikatni tekshiradi: CA, muddat,    │
     │    domen mosligi...]                              │
     │                                                   │
     │ ──── (5) Key exchange ──────────────────────────▶ │
     │   "pre-master secret (server public key bilan      │
     │    shifrlangan) yoki ECDHE parametrlari"          │
     │                                                   │
     │   [ikkala tomon ham R_c, R_s, pre-master'dan      │
     │    bir xil SESSION KEY (symmetric) hosil qiladi]  │
     │                                                   │
     │ ──── (6) Finished (shifrlangan) ───────────────▶  │
     │ ◀─── (7) Finished (shifrlangan) ────────────────  │
     │                                                   │
     │ ════ endi butun trafik symmetric kalit bilan ════ │
     │ ════ shifrlangan (application data) ═════════════ │
```

Qadamlar:

1. **ClientHello** — client qo'llab-quvvatlanadigan TLS versiyalari, cipher suite'lar va tasodifiy son (`R_c`) yuboradi.
2. **ServerHello** — server eng yaxshi umumiy cipher'ni tanlaydi va o'z tasodifiy sonini (`R_s`) yuboradi.
3. **Certificate** — server o'z sertifikatini (public key + CA imzosi) yuboradi.
4. **ServerHelloDone** — server tomonidan boshlang'ich xabarlar tugadi.
5. **Key exchange** — pre-master secret kelishiladi (RSA yoki, hozir afzal, ECDHE orqali). Ikkala tomon ham bir xil **session key** (symmetric) ni mustaqil hisoblab oladi.
6. **Finished (client)** — client endi shifrlangan tasdiq xabari yuboradi.
7. **Finished (server)** — server ham shifrlangan tasdiq yuboradi. Handshake tugadi; application data symmetric kalit bilan oqadi.

**⚠️ Ehtiyot bo'l:** Session key (symmetric kalit) hech qachon tarmoq orqali to'g'ridan-to'g'ri yuborilmaydi — ikkala tomon uni mahalliy hisoblaydi. ECDHE'da hatto private key sizib chiqsa ham o'tgan sessiyalarni deshifrlab bo'lmaydi (**forward secrecy**).

---

## TLS 1.2 vs TLS 1.3

**💡 Tushuncha:** TLS 1.3 (2018) — eski, sekin va zaif elementlarni olib tashlab, handshake'ni tezlashtirgan zamonaviy versiya.

| Xususiyat | TLS 1.2 | TLS 1.3 |
|-----------|---------|---------|
| Handshake RTT | 2-RTT | **1-RTT** (qayta ulanishda **0-RTT**) |
| Key exchange | RSA yoki ECDHE | faqat (EC)DHE — har doim forward secrecy |
| Eski/zaif cipher'lar | bor (RC4, CBC...) | olib tashlangan |
| Sertifikat | ochiq yuboriladi | handshake'da shifrlanadi |
| Tezlik | sekinroq | tezroq |

```text
TLS 1.2 (2-RTT):              TLS 1.3 (1-RTT):
 ClientHello ───▶             ClientHello + key share ───▶
        ◀─── ServerHello            ◀─── ServerHello + key
        ◀─── Certificate                 + Certificate
 KeyExchange ───▶                        + Finished
        ◀─── Finished           Finished + data ───▶
 (2 marta borib-kelish)        (1 marta borib-kelish)
```

**💡 Tushuncha:** **0-RTT** (zero round-trip) — avval ulangan serverga qayta ulanishda client birinchi xabar bilanoq application data yuborishi mumkin. Juda tez, lekin **replay attack** xavfi bor — shuning uchun faqat idempotent (takror xavfsiz) so'rovlar uchun ishlatiladi.

**⚠️ Ehtiyot bo'l:** "TLS 1.3 tezroq" deganda nimaga e'tibor berishni biling: bir kam round-trip, ayniqsa mobil va uzoq masofada sezilarli farq. Hozir TLS 1.0/1.1 deprecated — production'da kamida TLS 1.2, afzal 1.3 ishlatiladi. SSL (TLS oldingi nomi) butunlay eskirgan.

---

## Digital certificate va CA

**💡 Tushuncha:** Digital certificate — server public key'ini va u qaysi domenга tegishliligini bog'lovchi, **Certificate Authority (CA)** tomonidan imzolangan elektron hujjat. CA — hamma ishonadigan uchinchi tomon (masalan Let's Encrypt, DigiCert).

Sertifikatda nima bor:

```text
Subject:        CN=example.com         (kimga tegishli)
Issuer:         CN=Let's Encrypt R3    (kim imzolagan — CA)
Validity:       2026-01-01 → 2026-04-01 (amal muddati)
Public Key:     (server public key)
SAN:            example.com, www.example.com  (qaysi domenlar)
Signature:      (CA private key bilan imzo)
```

Nega kerak: shunchaki public key olsangiz, u haqiqatan `example.com` ga tegishlimi yoki hujumchining kaliti emasligini bilmaysiz. CA imzosi "men bu public key haqiqatan `example.com` egasiniki ekanligini tekshirdim" deydi.

**⚠️ Ehtiyot bo'l:** Sertifikat **domen**ga bog'langan, IP'ga emas. Sertifikatdagi domen (CN/SAN) siz ulanayotgan host'ga mos kelmasa, brauzer xato beradi (`ERR_CERT_COMMON_NAME_INVALID`).

---

## Chain of trust

**💡 Tushuncha:** Brauzer har bir CA'ni to'g'ridan-to'g'ri bilmaydi — u faqat bir nechta **root CA**'larni ishonchli deb biladi (OS/brauzerga oldindan o'rnatilgan trust store). Qolgan sertifikatlar zanjir (chain) orqali root'ga bog'lanadi.

```text
┌────────────────────┐
│   Root CA          │  ◀── brauzer trust store'da (oldindan ishoniladi)
│   (self-signed)    │
└─────────┬──────────┘
          │ imzolaydi
          ▼
┌────────────────────┐
│ Intermediate CA    │  ◀── server yuboradi
└─────────┬──────────┘
          │ imzolaydi
          ▼
┌────────────────────┐
│ Leaf (server) cert │  ◀── example.com sertifikati
│   example.com      │
└────────────────────┘

Brauzer tekshiradi:  leaf → intermediate → root
har bir imzo yuqoridagi CA public key bilan to'g'rimi?
Root trust store'dami?  →  ishonchli ✅
```

**💡 Tushuncha:** Server odatda leaf + intermediate sertifikatlarni yuboradi; root'ni brauzer o'zida bor deb topadi. Agar intermediate yuborilmasa, ba'zi brauzerlarda "incomplete chain" xatosi chiqadi.

**⚠️ Ehtiyot bo'l:** Zanjirning istalgan bo'g'ini yaroqsiz bo'lsa (muddat o'tgan, imzo noto'g'ri, root ishonchli emas) — butun zanjir ishonchsiz bo'ladi. Ko'p production muammolari aynan tushib qolgan yoki muddati o'tgan intermediate sertifikat tufayli bo'ladi.

---

## Self-signed certificate

**💡 Tushuncha:** Self-signed certificate — CA tomonidan emas, o'zining private key'i bilan imzolangan sertifikat. Issuer = Subject (o'zi o'zini imzolagan).

```bash
# self-signed sertifikat yaratish
openssl req -x509 -newkey rsa:2048 -keyout key.pem -out cert.pem -days 365 -nodes
```

- ✅ Bepul, tez, internet kerak emas — **dev/test** va ichki tarmoq uchun yaxshi.
- ❌ Brauzer ishonmaydi (CA imzosi yo'q) — "Your connection is not private" ogohlantirishi chiqadi.

**⚠️ Ehtiyot bo'l:** Self-signed sertifikatni production'da public sayt uchun ishlatmang — foydalanuvchilar ogohlantirishni ko'radi va MITM hujumini self-signed'dan ajrata olmaydi. Production uchun bepul **Let's Encrypt** ishlatiladi. Self-signed faqat siz trust store'ga qo'lda qo'shgan ichki tizimlarda o'rinli.

---

## HTTPS HTTP ustida qanday ishlaydi

**💡 Tushuncha:** HTTPS — yangi protokol emas. Bu shunchaki **HTTP + TLS qatlami**. TLS transport (TCP) ustida shifrlangan tunnel quradi, HTTP esa o'zgarmagan holda o'sha tunnel ichida ishlaydi.

```text
┌─────────────────────────────┐
│         HTTP                 │  ← so'rov/javob (GET /, 200 OK) — o'zgarmagan
├─────────────────────────────┤
│         TLS                  │  ← shifrlash + autentifikatsiya (HTTPS shu yerda)
├─────────────────────────────┤
│         TCP                  │  ← ishonchli ulanish
├─────────────────────────────┤
│         IP                   │  ← marshrutlash
└─────────────────────────────┘
```

Ulanish ketma-ketligi:

```text
1. TCP handshake     →  IP:443 ga ulanish (SYN/SYN-ACK/ACK)
2. TLS handshake     →  shifrlangan kanal + server autentifikatsiyasi
3. HTTP request      →  GET / HTTP/1.1  (TLS ichida shifrlangan)
4. HTTP response     →  200 OK + body   (TLS ichida shifrlangan)
```

**💡 Tushuncha:** Shuning uchun HTTPS HTTP'dan bir oz sekinroq boshlanadi (TLS handshake qo'shimcha vaqt) — lekin TLS 1.3, session resumption va keep-alive bu farqni deyarli yo'qqa chiqaradi.

**⚠️ Ehtiyot bo'l:** TLS request body, headerlar va URL path'ni shifrlaydi, **lekin** domen nomi (SNI orqali) va IP ochiq qoladi — tarmoqni kuzatuvchi siz qaysi saytga ulanayotganingizni (lekin nima yuborayotganingizni emas) ko'rishi mumkin.

---

## HSTS

**💡 Tushuncha:** HSTS (HTTP Strict Transport Security) — server brauzerga "menga faqat HTTPS orqali ulan, hech qachon HTTP emas" deb aytuvchi header.

```text
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
```

- `max-age` — necha soniya bu qoidani eslab qolish (bir yil = 31536000).
- `includeSubDomains` — barcha subdomenlarga ham qo'llash.
- `preload` — brauzerlarning ichki HSTS ro'yxatiga qo'shilish (birinchi tashrifdayoq HTTPS).

**💡 Tushuncha:** HSTS **SSL stripping** hujumidan himoya qiladi: hujumchi sizning birinchi HTTP so'rovingizni ushlab, HTTPS'ga o'tkazmasdan trafikni ochiq holatda olib qolishga uringanda. HSTS bilan brauzer HTTP'ga umuman urinmaydi.

**⚠️ Ehtiyot bo'l:** HSTS'ni `preload` bilan qo'shish — kuchli majburiyat: agar keyin HTTPS'ni o'chirsangiz, sayt brauzerlarda ochilmay qoladi (ro'yxatdan chiqarish sekin). Avval `max-age` ni qisqa qilib sinab ko'ring.

---

## SNI

**💡 Tushuncha:** SNI (Server Name Indication) — ClientHello'da client qaysi domenга ulanayotganini ochiq aytadigan TLS kengaytmasi. Bitta IP'da bir nechta HTTPS sayt joylashganda kerak.

```text
Bir server IP: 93.184.216.34, lekin 3 ta sayt:
   a.com, b.com, c.com   (har birida o'z sertifikati)

Client ClientHello'da SNI: "b.com" deydi
   →  server qaysi sertifikatni yuborishni biladi (b.com niki)
```

Muammo: SNI'siz server qaysi sertifikatni berishni bilmasdi — chunki HTTP `Host` header sertifikat allaqachon yuborilgandan keyin keladi.

**⚠️ Ehtiyot bo'l:** Klassik SNI **ochiq matnda** yuboriladi — tarmoqni kuzatuvchi siz qaysi domenга ulanayotganingizni ko'radi (kontent shifrlangan bo'lsa ham). Buni hal qilish uchun **ESNI/ECH** (Encrypted Client Hello) ishlab chiqilmoqda.

---

## mTLS (mutual TLS)

**💡 Tushuncha:** Oddiy TLS'da faqat **server** o'zini sertifikat bilan tasdiqlaydi, client esa anonim (parol/token bilan keyinroq autentifikatsiya qilinadi). **mTLS** (mutual TLS) da **ikkala tomon** ham sertifikat bilan bir-birini tasdiqlaydi.

```text
Oddiy TLS:                      mTLS:
 server → cert → client          server → cert → client
 (client anonim)                 client → cert → server  ← qo'shimcha
                                 (ikkala tomon tasdiqlanadi)
```

Qayerda ishlatiladi:

- **Microservice'lar orasida** — service A faqat sertifikatli service B'ni qabul qiladi (zero-trust tarmoq).
- **API'lar** — partnyor tizimlar mTLS bilan o'zini tasdiqlaydi (parol o'rniga).
- **Service mesh** (Istio, Linkerd) avtomatik mTLS o'rnatadi.

**⚠️ Ehtiyot bo'l:** mTLS kuchli, lekin client sertifikatlarini boshqarish (tarqatish, rotatsiya, bekor qilish) qo'shimcha murakkablik keltiradi. Public veb-saytlar uchun odatda haddan ortiq — u ichki/B2B tizimlar uchun mos.

---

## Certificate'ni tekshirish

**💡 Tushuncha:** Brauzer (yoki client) handshake paytida sertifikatni avtomatik tekshiradi. Asosiy tekshiruvlar:

1. **Imzo** — sertifikat ishonchli CA tomonidan imzolanganmi (chain of trust)?
2. **Muddat** — `Not Before` / `Not After` oralig'idami (muddati o'tmaganmi)?
3. **Domen mosligi** — sertifikatdagi CN/SAN siz ulanayotgan host'ga mosmi?
4. **Bekor qilinmaganmi** — CRL yoki OCSP orqali sertifikat bekor qilinmaganmi (revoked)?

Qo'lda tekshirish:

```bash
# serverning sertifikatini ko'rish
openssl s_client -connect example.com:443 -servername example.com

# sertifikat tafsilotlari (muddat, issuer, SAN)
echo | openssl s_client -connect example.com:443 2>/dev/null \
  | openssl x509 -noout -text

# faqat amal muddati
echo | openssl s_client -connect example.com:443 2>/dev/null \
  | openssl x509 -noout -dates

# lokal sertifikat faylni ko'rish
openssl x509 -in cert.pem -noout -subject -issuer -dates
```

**⚠️ Ehtiyot bo'l:** Eng ko'p uchraydigan production muammosi — **muddati o'tgan sertifikat**. Avtomatik yangilash (Let's Encrypt + certbot/acme) va monitoring/alert (muddat tugashidan 30 kun oldin) sozlash shart. Sertifikatni qo'lda yangilashga tayanmang.

---

## MITM hujumi va TLS himoyasi

**💡 Tushuncha:** MITM (Man-In-The-Middle) — hujumchi client va server orasiga "o'rtaga" kirib, trafikni o'qiydi yoki o'zgartiradi.

```text
HTTPS'siz (zaif):
   client ──▶ [HUJUMCHI o'qiydi/o'zgartiradi] ──▶ server
              (ochiq matn, hammasi ko'rinadi)

HTTPS bilan:
   client ──▶ [HUJUMCHI faqat shifrlangan baytlarni ko'radi] ──▶ server
```

TLS qanday himoya qiladi:

- **Confidentiality** — hujumchi shifrlangan trafikni ko'radi, lekin session key'siz o'qiy olmaydi.
- **Integrity** — trafikni o'zgartirsa, MAC/AEAD tekshiruvi buziladi va ulanish uziladi.
- **Authentication** — hujumchi serverni "taqlid qilish" uchun haqiqiy sertifikat kerak. Lekin u `example.com` uchun CA imzolagan sertifikatni ololmaydi (CA domen egaligini tekshiradi). Soxta sertifikat bersa, brauzer CA imzosi yo'qligidan xato beradi.

Misol hujum stsenariysi: hujumchi soxta sertifikat bilan o'zini server qilib ko'rsatmoqchi → brauzer sertifikat ishonchli CA'dan emasligini ko'rib **ogohlantiradi** → foydalanuvchi to'xtaydi. Aynan shu sabab self-signed sertifikatda chiqadigan "not private" ogohlantirishini e'tiborsiz qoldirmaslik kerak.

**⚠️ Ehtiyot bo'l:** TLS himoyasi foydalanuvchi sertifikat ogohlantirishini **bosib o'tmasligiga** bog'liq. Shuningdek, hujumchi qurilmaga o'z root CA'sini o'rnatib qo'ya olsa (korporativ proxy, malware), MITM mumkin bo'ladi — shuning uchun trust store'ni boshqarish muhim. Yuqori xavfsizlik uchun mobil ilovalarda **certificate pinning** (faqat ma'lum sertifikatga ishonish) ishlatiladi.

---

## Intervyu savollari (Q&A)

### ❓ HTTPS qanday uchta kafolat beradi?

**✅ Javob:** Confidentiality (shifrlash — trafikni hech kim o'qiy olmaydi), integrity (MAC/AEAD — trafik yo'lda o'zgartirilmagan) va authentication (sertifikat + CA imzosi — siz haqiqatan o'sha server bilan gaplashyapsiz). HTTPS = HTTP + TLS.

### ❓ Symmetric va asymmetric encryption farqi nima, TLS qaysisini ishlatadi?

**✅ Javob:** Symmetric — bir kalit shifrlaydi va ochadi, tez, lekin key distribution muammosi bor. Asymmetric — public/private juft kalit, key distribution'ni hal qiladi, lekin sekin. TLS ikkalasini birlashtiradi: asymmetric bilan symmetric session key'ni xavfsiz kelishadi, keyin butun ma'lumotni tez symmetric bilan shifrlaydi.

### ❓ Public va private key nima uchun?

**✅ Javob:** Public key hamma bilan baham ko'riladi (sertifikatda), private key faqat egada maxfiy qoladi. Public bilan shifrlangan narsa faqat private bilan ochiladi (confidentiality); private bilan imzolangan narsa public bilan tekshiriladi (authentication). Private key sizsa, butun xavfsizlik buziladi.

### ❓ TLS handshake bosqichlarini tushuntiring.

**✅ Javob:** (1) ClientHello — versiya, cipher'lar, random; (2) ServerHello — tanlangan cipher, random; (3) Certificate — server public key + CA imzo; (4) client sertifikatni tekshiradi; (5) key exchange — pre-master secret kelishiladi, ikkala tomon bir xil session key hisoblaydi; (6,7) Finished — shifrlangan tasdiq. So'ng application data symmetric kalit bilan oqadi.

### ❓ TLS 1.3 nimasi bilan 1.2'dan yaxshiroq?

**✅ Javob:** 1-RTT handshake (1.2 da 2-RTT), qayta ulanishda 0-RTT; faqat (EC)DHE — har doim forward secrecy; eski/zaif cipher'lar olib tashlangan; sertifikat shifrlangan holda yuboriladi. Natija: tezroq va xavfsizroq.

### ❓ 0-RTT nima va xavfi nimada?

**✅ Javob:** 0-RTT (TLS 1.3) — avval ulangan serverga qayta ulanishda client birinchi xabar bilanoq application data yuboradi (round-trip kutmasdan). Xavfi — replay attack (xabarni takror yuborish). Shuning uchun faqat idempotent so'rovlar uchun ishlatiladi.

### ❓ Certificate Authority (CA) nima va nega kerak?

**✅ Javob:** CA — hamma ishonadigan uchinchi tomon, server public key'ini va u qaysi domenга tegishliligini tekshirib, sertifikatni imzolaydi. Kerakligi: shunchaki public key haqiqatan o'sha domenga tegishlimi yoki hujumchinikimi bilib bo'lmaydi — CA imzosi buni kafolatlaydi.

### ❓ Chain of trust qanday ishlaydi?

**✅ Javob:** Brauzer faqat oldindan o'rnatilgan root CA'larga ishonadi. Server leaf sertifikatini intermediate CA imzolaydi, intermediate'ni root imzolaydi. Brauzer zanjirni leaf → intermediate → root tartibida tekshiradi; root trust store'da bo'lsa, ishonchli deb topadi. Bir bo'g'in yaroqsiz bo'lsa, butun zanjir ishonchsiz.

### ❓ Self-signed certificate nima va qachon ishlatiladi?

**✅ Javob:** O'zini o'zi imzolagan (CA emas) sertifikat. Bepul va tez — dev/test va ichki tizimlar uchun yaxshi. Lekin brauzer ishonmaydi ("not private" ogohlantirishi). Public production saytda ishlatilmaydi — Let's Encrypt afzal.

### ❓ HTTPS HTTP ustida qanday ishlaydi?

**✅ Javob:** HTTPS — alohida protokol emas, HTTP + TLS qatlami. TCP ustida TLS shifrlangan tunnel quradi, HTTP o'zgarmagan holda o'sha tunnel ichida ishlaydi. Ketma-ketlik: TCP handshake → TLS handshake → shifrlangan HTTP request/response.

### ❓ HSTS nima va qaysi hujumdan himoya qiladi?

**✅ Javob:** `Strict-Transport-Security` header — brauzerga "menga faqat HTTPS orqali ulan" deydi. SSL stripping hujumidan himoya qiladi: brauzer HTTP'ga umuman urinmaydi, shuning uchun hujumchi birinchi HTTP so'rovni ushlab ochiq holatga tushira olmaydi. `preload` bilan birinchi tashrifdayoq HTTPS majburlanadi.

### ❓ SNI nima uchun kerak?

**✅ Javob:** Bitta IP'da bir nechta HTTPS sayt bo'lganda, server qaysi sertifikatni yuborishni bilishi uchun. Client ClientHello'da SNI orqali domen nomini aytadi (HTTP `Host` header sertifikatdan keyin kelgani uchun yetmaydi). Klassik SNI ochiq matnda — ECH bu muammoni hal qiladi.

### ❓ mTLS oddiy TLS'dan nima bilan farq qiladi?

**✅ Javob:** Oddiy TLS'da faqat server sertifikat bilan tasdiqlanadi, client anonim. mTLS'da ikkala tomon ham sertifikat ko'rsatadi — server clientni ham tekshiradi. Microservice'lar, B2B API'lar va service mesh'da (zero-trust) ishlatiladi. Public saytlar uchun odatda ortiqcha.

### ❓ Brauzer sertifikatni qanday tekshiradi?

**✅ Javob:** (1) CA imzosi to'g'ri va ishonchli zanjirdan kelganmi; (2) muddati o'tmaganmi; (3) sertifikatdagi domen (CN/SAN) ulanilayotgan host'ga mosmi; (4) bekor qilinmaganmi (OCSP/CRL). Biror tekshiruv buzilsa — xato/ogohlantirish.

### ❓ MITM hujumini TLS qanday to'xtatadi?

**✅ Javob:** Hujumchi shifrlangan trafikni o'qiy olmaydi (confidentiality), o'zgartirsa MAC/AEAD buziladi (integrity), va serverni taqlid qilish uchun CA imzolagan haqiqiy sertifikatni ololmaydi (authentication) — soxta sertifikat bersa brauzer ogohlantiradi. Foydalanuvchi ogohlantirishni bosib o'tmasa, hujum to'xtaydi.

### ❓ Forward secrecy nima?

**✅ Javob:** Har sessiya uchun vaqtinchalik (ephemeral) kalit ishlatish (ECDHE) — natijada serverning private key'i kelajakda sizib chiqsa ham, **o'tgan** yozib olingan sessiyalarni deshifrlab bo'lmaydi. TLS 1.3 buni majburiy qiladi.

---

## Masalalar

> Yechimlar: [solutions/networking/04-tls-https.md](../solutions/networking/04-tls-https.md)

1. **Kafolatni moslashtirish.** Quyidagi har bir holatda HTTPS'ning qaysi kafolati (confidentiality / integrity / authentication) ishlamoqda: (a) parol shifrlangan holda uzatildi; (b) brauzer soxta saytni aniqladi; (c) o'rtadagi proxy javob HTML'ni o'zgartirib yubora olmadi.

2. **Encryption tanlash.** Nega TLS butun ma'lumotni asymmetric encryption bilan shifrlamaydi? Symmetric va asymmetric har biri TLS'da qaysi bosqichda ishlatiladi — tushuntiring.

3. **Handshake tartibi.** Quyidagi TLS xabarlarni to'g'ri tartibga keltiring: `Finished`, `ClientHello`, `Certificate`, `ServerHello`, `Key exchange`. Har birida kim (client/server) yuborayotganini belgilang.

4. **1.2 yoki 1.3?** Mobil ilovangiz uzoq masofadagi serverga tez-tez qisqa so'rov yuboradi va latency muhim. TLS 1.2 yoki 1.3 ni tanlaysizmi va nega? 0-RTT'ni shu API uchun yoqasizmi?

5. **Sertifikat xatosi debug.** Foydalanuvchi `https://shop.example.com` da "NET::ERR_CERT_COMMON_NAME_INVALID" xatosini ko'ryapti. Sabab nima bo'lishi mumkin va qanday tekshirasiz/tuzatasiz?

6. **Chain muammosi.** Saytingiz brauzerda ishlaydi, lekin ba'zi mobil clientlarda "untrusted certificate" beradi. Eng ehtimoliy sabab nima va qanday topasiz?

7. **mTLS yoki yo'q?** Quyidagilardan qaysilariga mTLS mos, qaysilariga ortiqcha: (a) public blog; (b) ikki ichki microservice orasidagi aloqa; (c) bank partnyori bilan B2B API; (d) oddiy login formali veb-sayt. Sabab bilan.

8. **MITM stsenariysi.** Hujumchi ochiq Wi-Fi'da o'tirib, foydalanuvchi `https://bank.com` ga ulanganida o'zini server qilib ko'rsatmoqchi. Hujum qaysi bosqichda va nega muvaffaqiyatsiz bo'ladi? Hujumchi qanday qilib (noto'g'ri yo'l bilan) muvaffaqiyatga erishishi mumkin?

9. **openssl bilan tekshirish.** `example.com:443` sertifikatining (a) amal muddatini va (b) issuer'ini ko'rsatadigan `openssl` buyruqlarini yozing.

10. **HSTS dilemma.** Jamoangiz `Strict-Transport-Security: max-age=63072000; includeSubDomains; preload` qo'shdi. Bir hafta o'tib bitta subdomen (`legacy.example.com`) HTTPS'ni qo'llab-quvvatlamasligi ma'lum bo'ldi. Qanday muammo yuzaga keladi va buni oldindan qanday oldini olish kerak edi?

---

← [Networking bo'limiga qaytish](./README.md)
