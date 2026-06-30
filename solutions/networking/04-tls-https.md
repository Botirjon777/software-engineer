# TLS va HTTPS — Yechimlar

Bu fayl [`networking/04-tls-https.md`](../../networking/04-tls-https.md) dagi "Masalalar" bo'limining yechimlarini o'z ichiga oladi.

## Mundarija

- [1. Kafolatni moslashtirish](#1-kafolatni-moslashtirish)
- [2. Encryption tanlash](#2-encryption-tanlash)
- [3. Handshake tartibi](#3-handshake-tartibi)
- [4. 1.2 yoki 1.3?](#4-12-yoki-13)
- [5. Sertifikat xatosi debug](#5-sertifikat-xatosi-debug)
- [6. Chain muammosi](#6-chain-muammosi)
- [7. mTLS yoki yo'q?](#7-mtls-yoki-yoq)
- [8. MITM stsenariysi](#8-mitm-stsenariysi)
- [9. openssl bilan tekshirish](#9-openssl-bilan-tekshirish)
- [10. HSTS dilemma](#10-hsts-dilemma)

---

## 1. Kafolatni moslashtirish

| Holat | Kafolat |
|-------|---------|
| (a) parol shifrlangan holda uzatildi | **Confidentiality** |
| (b) brauzer soxta saytni aniqladi | **Authentication** |
| (c) proxy javob HTML'ni o'zgartira olmadi | **Integrity** |

**Izoh:** (a) shifrlash trafikni o'qishdan saqlaydi; (b) sertifikat + CA imzosi serverning haqiqiyligini tasdiqlaydi, soxtasini fosh qiladi; (c) MAC/AEAD har baytni tekshiradi — o'zgartirilsa ulanish uziladi.

---

## 2. Encryption tanlash

**Nega butun ma'lumot asymmetric bilan emas:** asymmetric encryption juda **sekin** va resurs talab qiladi — har bir paketni shunday shifrlash ilovani og'irlashtirar va katta yuk uchun amalda mumkin emas edi.

**TLS'da har birining roli:**

```text
ASYMMETRIC (handshake bosqichi):
   server public key / ECDHE orqali symmetric SESSION KEY xavfsiz kelishiladi.
   Maqsad: key distribution muammosini hal qilish + server autentifikatsiyasi.

SYMMETRIC (application data bosqichi):
   handshake'dan keyin butun trafik tez symmetric kalit (AES) bilan shifrlanadi.
   Maqsad: katta hajmli ma'lumotni tez shifrlash.
```

**Izoh:** Bu "asymmetric bilan kalitni kelishib, symmetric bilan gaplashish" yondashuvi — TLS'ning markaziy g'oyasi. Sekin operatsiya faqat bir marta (handshake), tez operatsiya doimiy (data).

---

## 3. Handshake tartibi

```text
1. ClientHello      (client → server)   versiya, cipher'lar, random
2. ServerHello      (server → client)   tanlangan cipher, random
3. Certificate      (server → client)   public key + CA imzo
4. Key exchange     (client → server)   pre-master / ECDHE parametrlari
5. Finished         (har ikki tomon)    shifrlangan tasdiq
```

**Izoh:** ClientHello har doim birinchi. Sertifikatni server beradi (3). Key exchange'dan keyin ikkala tomon session key'ni hisoblaydi, so'ng Finished xabarlari (avval client, keyin server) almashinadi va handshake yakunlanadi.

---

## 4. 1.2 yoki 1.3?

**Tanlov: TLS 1.3.** Sabablari:

- **1-RTT handshake** — uzoq masofada (yuqori RTT) bitta kam borib-kelish sezilarli latency tejaydi.
- Qayta ulanishda **session resumption / 0-RTT** — keyingi so'rovlar yana tezroq.
- Har doim forward secrecy va kuchli cipher'lar — xavfsizroq.

**0-RTT'ni yoqishmi?** **Ehtiyotkorlik bilan.** Faqat agar so'rovlar **idempotent** (masalan `GET`) bo'lsa. 0-RTT replay attack'ga ochiq — agar so'rov holatni o'zgartirsa (to'lov, buyurtma yaratish), takror yuborilishi zararli bo'ladi. Shuning uchun yozuv (write) operatsiyalarini 0-RTT'siz, oddiy 1-RTT yo'l bilan yuboring.

**Izoh:** "Latency muhim" deganda 1.3 deyarli har doim to'g'ri tanlov. 0-RTT esa qo'shimcha tezlik beradi, lekin xavfsizlik shartini buzmaslik kerak.

---

## 5. Sertifikat xatosi debug

**`ERR_CERT_COMMON_NAME_INVALID` ma'nosi:** sertifikatdagi domen (CN/SAN) siz ulanayotgan host'ga (`shop.example.com`) mos kelmaydi.

**Ehtimoliy sabablar:**

- Sertifikat `example.com` uchun olingan, lekin `shop.example.com` (subdomen) SAN'da yo'q.
- `www.example.com` uchun sertifikat, `shop.` ga ishlatilyapti.
- Wildcard `*.example.com` kerak edi, lekin oddiy sertifikat qo'yilgan.

**Tekshirish:**

```bash
echo | openssl s_client -connect shop.example.com:443 -servername shop.example.com 2>/dev/null \
  | openssl x509 -noout -subject -ext subjectAltName
```

SAN ro'yxatida `shop.example.com` bor-yo'qligini ko'ring.

**Tuzatish:** `shop.example.com` ni o'z ichiga olgan sertifikat oling — yoki SAN'ga qo'shing, yoki `*.example.com` wildcard sertifikat ishlating, yoki shu host uchun alohida sertifikat.

---

## 6. Chain muammosi

**Eng ehtimoliy sabab:** server **intermediate sertifikatni yubormayapti** (incomplete chain). Desktop brauzerlar ko'pincha tushib qolgan intermediate'ni o'zlari to'ldira oladi (AIA fetching/keshlangan), lekin ko'p mobil/eski clientlar buni qila olmaydi — shuning uchun ularda "untrusted" chiqadi.

**Qanday topasiz:**

```bash
# yuborilgan zanjirni ko'rish
openssl s_client -connect example.com:443 -showcerts
```

Chiqishda faqat leaf (`example.com`) bo'lib, intermediate (`... CA ...`) yo'qligini ko'rsangiz — muammo shu. Yoki onlayn SSL test (masalan SSL Labs) "chain issues / incomplete" deydi.

**Tuzatish:** serverda to'liq zanjir (leaf + intermediate) o'rnatilgan `fullchain.pem` ni ishlating (Let's Encrypt buni beradi), faqat leaf `cert.pem` emas.

---

## 7. mTLS yoki yo'q?

| Holat | mTLS? | Sabab |
|-------|-------|-------|
| (a) public blog | ❌ ortiqcha | foydalanuvchilar anonim, client sertifikat boshqarib bo'lmaydi |
| (b) ikki ichki microservice | ✅ mos | zero-trust, xizmatlar bir-birini tasdiqlaydi |
| (c) bank B2B API | ✅ mos | kuchli o'zaro autentifikatsiya, parol o'rniga sertifikat |
| (d) oddiy login formali sayt | ❌ ortiqcha | oddiy TLS + parol/token yetadi |

**Izoh:** mTLS qiymati — clientni ham kriptografik tasdiqlash, bu ichki/B2B kontekstda mantiqiy. Public va keng auditoriyali tizimlarda client sertifikatlarini tarqatish/boshqarish amalda imkonsiz, shuning uchun ortiqcha.

---

## 8. MITM stsenariysi

**Qaysi bosqichda muvaffaqiyatsiz:** TLS handshake'ning **sertifikat tekshirish** bosqichida.

```text
client ──ClientHello──▶ HUJUMCHI
client ◀──Certificate── HUJUMCHI   ← bu yerda muammo:
   hujumchida bank.com uchun ISHONCHLI CA imzolagan sertifikat YO'Q.
   → brauzer authentication'da yiqiladi → ogohlantirish → foydalanuvchi to'xtaydi.
```

**Nega:** hujumchi `bank.com` uchun haqiqiy CA imzolagan sertifikatni ololmaydi — CA imzolashdan oldin domen egaligini tekshiradi (DNS/HTTP challenge), hujumchi esa domen egasi emas. Soxta yoki self-signed sertifikat bersa, brauzer "not private" ogohlantiradi.

**Hujumchi qanday (noto'g'ri yo'l bilan) muvaffaqiyatga erishishi mumkin:**

- Foydalanuvchi ogohlantirishni **bosib o'tsa** ("Proceed anyway").
- Hujumchi qurilmaga o'z **root CA**'sini o'rnata olsa (malware, korporativ MDM) — endi soxta sertifikat "ishonchli" bo'lib ko'rinadi.
- HTTP'dan boshlansa va HSTS bo'lmasa — SSL stripping bilan TLS'ni umuman o'rnatmaslik.

**Himoya:** HSTS (preload), certificate pinning (mobil ilovalarda), foydalanuvchilarni ogohlantirishni e'tiborsiz qoldirmaslikka o'rgatish.

---

## 9. openssl bilan tekshirish

```bash
# (a) amal muddati (Not Before / Not After)
echo | openssl s_client -connect example.com:443 -servername example.com 2>/dev/null \
  | openssl x509 -noout -dates

# (b) issuer (kim imzolagan — CA)
echo | openssl s_client -connect example.com:443 -servername example.com 2>/dev/null \
  | openssl x509 -noout -issuer
```

**Izoh:** `-dates` `notBefore=...` va `notAfter=...` ni beradi; `-issuer` CA nomini beradi. Ikkalasini birga olish uchun `-subject -issuer -dates` ni bitta `x509` chaqiruvida qo'shsa bo'ladi. `-servername` SNI'ni to'g'ri yuboradi (bir IP'da ko'p sayt bo'lsa muhim).

---

## 10. HSTS dilemma

**Yuzaga keladigan muammo:** `max-age=63072000` (2 yil) tufayli foydalanuvchilar brauzeri `example.com` va barcha subdomenlar (`includeSubDomains`) uchun **faqat HTTPS** ni majburlaydi. `legacy.example.com` HTTPS'ni qo'llab-quvvatlamagani uchun, bu subdomen brauzerlarda **umuman ochilmaydi** — va HSTS keshi 2 yil davom etadi (foydalanuvchi qo'lda tozalamasa). `preload` bilan esa muammo yanada og'ir: subdomen brauzerlarning ichki ro'yxatiga kirib qoladi, ro'yxatdan chiqarish oylar olishi mumkin.

**Qanday oldini olish kerak edi:**

1. HSTS qo'shishdan oldin **barcha** subdomenlar HTTPS'da ishlashini tekshirish (`includeSubDomains` hammasini qamrab oladi).
2. Avval **qisqa `max-age`** (masalan 300s yoki bir necha kun) bilan sinab ko'rish, hammasi joyidaligiga ishonch hosil qilib, keyin oshirish.
3. `preload`'ni eng oxirida, to'liq tekshirgandan keyin qo'shish (u qaytarib bo'lmaydigan majburiyat).

**Tezkor yechim hozir:** `legacy.example.com` ni HTTPS'ga o'tkazish (sertifikat qo'yish) — chunki HSTS keshini foydalanuvchilardan tozalab bo'lmaydi. `includeSubDomains`/`preload`'ni olib tashlash kech, eski keshlarga ta'sir qilmaydi.

---

← [Networking bo'limiga qaytish](../../networking/README.md)
