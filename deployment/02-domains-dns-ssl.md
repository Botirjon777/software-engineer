# Domain va DNS

Loyihangiz Vercel'da yoki VPS'da ishlab turibdi, lekin uning manzili `my-project-abc123.vercel.app` yoki `185.203.11.42` ko'rinishida — buni odam eslay olmaydi. **Domain** shu muammoni hal qiladi: `example.uz` deb yozasiz, mijoz brauzeriga kiritadi va sizning saytingiz ochiladi. Bu fayl domenni qayerdan sotib olish, DNS record'larni sozlash, domenni platforma yoki VPS'ga ulash va HTTPS (SSL) yoqishni qadama-qadam ko'rsatadi. Barcha buyruqlar real — o'z domeningizda takrorlab ko'ring.

## Mundarija

- [Domain nima](#domain-nima)
- [Domenni qayerdan sotib olish (registrar)](#domenni-qayerdan-sotib-olish-registrar)
- [TLD tanlash (.com / .uz / .dev)](#tld-tanlash)
- [DNS nima va record turlari](#dns-nima-va-record-turlari)
- [Nameserver vs DNS record](#nameserver-vs-dns-record)
- [Domenni Vercel/Netlify'ga ulash](#domenni-vercelnetlifyga-ulash)
- [Domenni VPS IP'siga ulash](#domenni-vps-ipsiga-ulash)
- [DNS propagation va tekshirish](#dns-propagation-va-tekshirish)
- [SSL / HTTPS](#ssl--https)
- [www vs non-www va redirect](#www-vs-non-www-va-redirect)
- [Subdomain sozlash](#subdomain-sozlash)
- [Cloudflare: DNS + proxy + CDN](#cloudflare-dns--proxy--cdn)
- [Savol-Javob (Troubleshooting)](#savol-javob-troubleshooting)
- [Amaliy Checklist](#amaliy-checklist)

---

## Domain nima

Domain — bu internetdagi manzil (masalan `example.uz`). Aslida internet IP-manzillar (`185.203.11.42`) orqali ishlaydi, lekin raqamlarni eslash qiyin. Domain — bu raqamli IP ustidagi "chiroyli nom", telefon kitobidagi ismga o'xshaydi.

**💡 Tushuncha:** Domain — bu sizning **mulkingiz emas**, balki **ijaraga olingan** nom. Har yili to'lov qilib turasiz. To'lamasangiz, domen ozod bo'ladi va boshqa odam olib ketishi mumkin. Shuning uchun auto-renew (avtomatik uzaytirish) yoqib qo'ying.

Domen tarkibi:

```text
        api . example . uz
        │       │        │
   subdomain  name      TLD (Top-Level Domain)
```

- **TLD** — `.uz`, `.com`, `.dev` — eng o'ngdagi qism.
- **name** — `example` — siz tanlaydigan asosiy nom.
- **subdomain** — `api`, `www`, `blog` — asosiy domen oldidagi qism (bepul, cheksiz yaratasiz).

---

## Domenni qayerdan sotib olish (registrar)

Domenni **registrar** deb ataladigan kompaniyadan sotib olasiz. Ular ICANN oldida ro'yxatdan o'tgan va domen sotish huquqiga ega. Asosiy registrar'lar:

| Registrar | Narx (.com/yil) | Tavsiya |
|-----------|-----------------|---------|
| **Cloudflare** | ~$9-10 (at-cost, ustama yo'q) | ⭐ Eng arzon, uzaytirishda narx oshmaydi, DNS + CDN bepul birga keladi |
| **Porkbun** | ~$9-11 | Arzon, WHOIS privacy bepul, toza interfeys |
| **Namecheap** | ~$10-13 (1-yil chegirmali, keyin qimmatroq) | Ommabop, qulay panel, WHOIS privacy bepul |
| **GoDaddy** | ~$12 (1-yil arzon, uzaytirishda ~$20+) | Katta, lekin qimmat va upsell (qo'shimcha sotish) ko'p — tavsiya etmaymiz |

**💡 Tushuncha:** GoDaddy birinchi yil arzon narx bilan jalb qiladi, lekin **uzaytirish (renewal)** narxi ancha qimmat. Har doim **renewal narxini** tekshiring, birinchi yil narxini emas.

**Tavsiya (2026):**
- Xalqaro `.com` uchun → **Cloudflare** yoki **Porkbun** (arzon, halol narx).
- O'zbek `.uz` domeni uchun → mahalliy registrar (quyida).

### .uz domenini qayerdan olish

`.uz` — bu O'zbekistonning milliy domeni, uni CCTLD.UZ (`cctld.uz`) boshqaradi. Xalqaro registrar'lar odatda `.uz` sotmaydi. `.uz` ni mahalliy akkreditatsiyalangan provayderlardan olasiz (masalan `ahost.uz`, `uzinfocom` va boshqalar). `.uz` uchun ko'pincha O'zbekiston rezidenti bo'lish yoki mahalliy hujjat talab qilinishi mumkin — provayder shartlarini o'qing.

---

## TLD tanlash

TLD (Top-Level Domain) — domen oxiridagi qism. Qaysi birini tanlash loyihaga bog'liq:

| TLD | Qachon ishlatiladi | Eslatma |
|-----|--------------------|---------| 
| **.com** | Universal, biznes, startup, xalqaro | Eng ishonchli, mijoz avtomatik `.com` deb yozadi. Birinchi tanlov |
| **.uz** | O'zbekiston bozori, mahalliy biznes | Mahalliy auditoriya uchun ishonch, lekin talab/hujjat bo'lishi mumkin |
| **.dev** | Dasturchi portfolio, texnik loyiha | Google boshqaradi, **majburiy HTTPS** (HSTS preload) |
| **.io / .app / .co** | Startup, texnologiya | `.com` band bo'lsa alternativa, lekin qimmatroq |

**💡 Tushuncha:** `.dev` va `.app` TLD'lari HSTS preload ro'yxatida — ya'ni brauzer bu domenlarga **faqat HTTPS orqali** kiradi. Agar SSL sozlamasangiz, sayt umuman ochilmaydi. Bu xavfsizlik uchun yaxshi, lekin SSL ni birinchi kundan sozlashingiz shart.

**⚠️ Ehtiyot bo'l:** Juda arzon TLD'lar (`.xyz`, `.online`, `.top` birinchi yil $1) spam bilan bog'liq bo'lgani uchun Gmail/Outlook ba'zan bu domenlardan kelgan xatlarni spam'ga tashlaydi. Jiddiy biznes uchun `.com` yoki `.uz` xavfsizroq.

---

## DNS nima va record turlari

**DNS** (Domain Name System) — bu internetning "telefon kitobi". Odam `example.uz` deb yozadi, DNS uni IP-manzilga (`185.203.11.42`) tarjima qiladi, brauzer o'sha IP'ga boradi.

**💡 Tushuncha:** DNS record'lar — bu domen uchun "yo'l ko'rsatkichlari". Siz registrar yoki Cloudflare panelida record'larni kiritasiz, ular butun internet bo'ylab tarqaladi. Har bir record turi boshqa vazifani bajaradi.

Deploy'da eng ko'p kerak bo'ladigan record turlari:

### A record — domenni IP'ga bog'laydi (IPv4)

```text
Type: A
Name: @        (@ = asosiy domen, example.uz)
Value: 185.203.11.42   (VPS yoki server IPv4 manzili)
TTL: Auto (yoki 3600)
```

**Qachon kerak:** Domenni VPS'ga ulaganingizda (server IP'siga to'g'ridan-to'g'ri). Bu eng ko'p ishlatiladigan record.

### AAAA record — domenni IPv6'ga bog'laydi

```text
Type: AAAA
Name: @
Value: 2a01:4f8:c17:b8f::1   (IPv6 manzil)
```

**Qachon kerak:** Serveringiz IPv6 qo'llab-quvvatlasa. Majburiy emas, lekin zamonaviy serverlarda bo'lgani yaxshi.

### CNAME record — domenni boshqa domenga bog'laydi (alias)

```text
Type: CNAME
Name: www       (www.example.uz)
Value: cname.vercel-dns.com   (yoki example.netlify.app)
```

**Qachon kerak:** Vercel/Netlify kabi platformalarga ulaganda. Platforma sizga IP bermaydi (IP o'zgarib turadi), balki CNAME manzil beradi.

**⚠️ Ehtiyot bo'l:** Asosiy domen (`@`) uchun CNAME ishlatib bo'lmaydi (klassik DNS qoidasi). Faqat subdomain (`www`, `api`) uchun CNAME ishlatasiz. Asosiy domen uchun A record ishlating (yoki Cloudflare'ning "CNAME flattening" xususiyati).

### TXT record — matn ma'lumot (tekshiruv, email)

```text
Type: TXT
Name: @
Value: "v=spf1 include:_spf.google.com ~all"
```

**Qachon kerak:**
- Domen egaligini tasdiqlash (Google Search Console, Vercel domain verification).
- Email sozlash: SPF, DKIM, DMARC (xatlaringiz spam'ga tushmasligi uchun).

### MX record — email serverini ko'rsatadi

```text
Type: MX
Name: @
Value: aspmx.l.google.com
Priority: 1
```

**Qachon kerak:** Domenga email ulaganingizda (masalan `info@example.uz` Google Workspace orqali). Faqat email uchun, sayt uchun kerak emas.

### NS record — nameserverlarni ko'rsatadi

```text
Type: NS
Name: @
Value: kim.ns.cloudflare.com
```

**Qachon kerak:** DNS boshqaruvini boshqa provayderga (masalan Cloudflare) o'tkazganingizda. Odatda registrar panelida sozlanadi (record emas, "nameservers" bo'limida).

---

## Nameserver vs DNS record

Bu ikkisi ko'p chalkashtiriladi — farqni tushunib oling:

**💡 Tushuncha:**
- **Nameserver (NS)** — domen uchun **qaysi kompaniya** DNS record'larni boshqarishini aytadi. Bu "manzilingiz qaysi pochta bo'limiga tegishli" degan savol.
- **DNS record (A/CNAME/TXT...)** — o'sha nameserverlar ichidagi **aniq ko'rsatkichlar**. Bu "pochta bo'limi ichidagi aniq yashash manzilingiz".

Ikki xil ish qilish yo'li bor:

**Variant 1 — Registrar'ning o'z DNS'idan foydalanish.** Namecheap'dan domen oldingiz, DNS record'larni ham Namecheap panelida kiritasiz. Nameserverlar Namecheap'niki bo'lib qoladi.

**Variant 2 — DNS'ni Cloudflare'ga o'tkazish.** Registrar panelida nameserverlarni Cloudflare'nikiga o'zgartirasiz:

```text
Nameservers:
  kim.ns.cloudflare.com
  walt.ns.cloudflare.com
```

Shundan keyin barcha DNS record'larni **Cloudflare panelida** kiritasiz (registrar'da emas). Bu tavsiya etiladi — Cloudflare DNS tez, bepul CDN va himoya beradi.

**⚠️ Ehtiyot bo'l:** Nameserverni o'zgartirsangiz, registrar'dagi eski DNS record'lar **ishlamay qoladi**. Barcha record'larni yangi nameserverda (Cloudflare) qaytadan kiritishingiz kerak. Aks holda sayt vaqtincha o'chib qolishi mumkin.

---

## Domenni Vercel/Netlify'ga ulash

Frontend'ni Vercel yoki Netlify'ga deploy qildingiz (01-mavzu). Endi custom domen ulaymiz.

### Vercel'ga custom domain qo'shish

1. Vercel dashboard → loyihangiz → **Settings → Domains**.
2. `example.uz` deb yozing → **Add**.
3. Vercel sizga qo'shish kerak bo'lgan DNS record'larni ko'rsatadi. Ikki yo'l:

**Yo'l A — CNAME (subdomain uchun, masalan www):**

```text
Type: CNAME
Name: www
Value: cname.vercel-dns.com
```

**Yo'l B — A record (asosiy domen uchun):**

```text
Type: A
Name: @
Value: 76.76.21.21
```

4. Bu record'larni registrar yoki Cloudflare panelida qo'shing.
5. Vercel avtomatik tekshiradi va SSL sertifikatini o'zi beradi (bir necha daqiqada).

### Netlify'ga custom domain qo'shish

1. Netlify dashboard → **Site settings → Domain management → Add custom domain**.
2. Domenni yozing.
3. Netlify ikki variant beradi:
   - **CNAME** (tavsiya, subdomain): `Name: www`, `Value: your-site.netlify.app`
   - Yoki **Netlify DNS** (nameserverni Netlify'ga o'tkazish).
4. SSL avtomatik yoqiladi (Let's Encrypt).

**💡 Tushuncha:** Platformalar (Vercel/Netlify) sizga to'g'ridan-to'g'ri IP bermaydi, chunki ular yuzlab serverlarda ishlaydi va IP o'zgarib turadi. Shuning uchun **CNAME** ishlatishni afzal ko'radi — u nomga (domainga) bog'lanadi, IP o'zgarsa ham ishlayveradi.

---

## Domenni VPS IP'siga ulash

VPS oldingiz (05-mavzu) va uning IP manzili bor: `185.203.11.42`. Domenni shu IP'ga ulash — eng oddiy:

1. Registrar yoki Cloudflare DNS paneliga kiring.
2. Ikkita A record qo'shing:

```text
Type: A
Name: @
Value: 185.203.11.42
TTL: Auto

Type: A
Name: www
Value: 185.203.11.42
TTL: Auto
```

3. Saqlang va propagation'ni kuting (quyida).

Endi `example.uz` va `www.example.uz` sizning VPS'ingizga yo'naltiriladi. VPS ichida Nginx bu so'rovni qabul qilib, saytni ko'rsatadi (06-mavzu).

**💡 Tushuncha:** A record faqat domenni IP'ga yo'naltiradi. VPS ichida server (Nginx) ishlab turmasa, sayt baribir ochilmaydi ("connection refused"). DNS — bu yo'l ko'rsatkich, server — bu manzildagi bino. Ikkalasi ham kerak.

---

## DNS propagation va tekshirish

DNS record'larni o'zgartirgandan keyin, o'zgarish **darhol** butun dunyoga tarqalmaydi. Bu jarayon **propagation** deb ataladi.

**💡 Tushuncha:** Propagation — DNS o'zgarishining dunyodagi barcha serverlarga yetib borishi. Odatda **5 daqiqadan 2 soatgacha**, ba'zan (nameserver o'zgarsa) **24-48 soatgacha** vaqt oladi. Bu TTL (Time To Live) qiymatiga bog'liq.

Tekshirish yo'llari:

### dig (Linux/Mac — eng aniq)

```bash
dig example.uz +short
# Natija: 185.203.11.42  ← A record IP'si

dig www.example.uz CNAME +short
# CNAME record'ni ko'rsatadi
```

### nslookup (Windows/hamma joyda)

```bash
nslookup example.uz
# Server javobida IP'ni ko'rsatadi
```

### whatsmydns.net (dunyo bo'ylab)

Brauzerda `whatsmydns.net` ochib, domeningizni yozing → dunyoning turli davlatlarida DNS qanday ko'rinayotganini xaritada ko'rsatadi. Barcha joyda yashil ✓ bo'lsa, propagation tugagan.

**⚠️ Ehtiyot bo'l:** Brauzeringiz eski DNS'ni cache'da saqlab qolishi mumkin. Sayt sizga ochilmasa ham, boshqa qurilma/mobil internetda ochilishi mumkin. DNS cache'ni tozalash:

```bash
# Windows
ipconfig /flushdns

# Mac
sudo dscacheutil -flushcache
```

---

## SSL / HTTPS

**SSL/TLS** — bu sayt va foydalanuvchi o'rtasidagi aloqani shifrlaydigan texnologiya. HTTPS = HTTP + SSL. Manzil satrida qulf 🔒 belgisi ko'rinadi.

### Nega SSL kerak

- **Xavfsizlik:** parol, karta ma'lumotlari shifrlanadi (aks holda ochiq ketadi).
- **Brauzer talabi:** HTTPS'siz saytni Chrome "Not Secure" (Xavfsiz emas) deb belgilaydi — mijoz qo'rqib chiqib ketadi.
- **SEO:** Google HTTPS saytlarni yuqoriroq ko'taradi.
- **Majburiy:** `.dev`, `.app` domenlari HTTPS'siz umuman ochilmaydi.

### Let's Encrypt — bepul SSL

**Let's Encrypt** — bepul, avtomatik SSL sertifikat beruvchi tashkilot. Ilgari SSL yiliga $50-100 turardi, endi bepul. Deyarli barcha zamonaviy deploy Let's Encrypt ishlatadi.

### Platformalarda avtomatik SSL

Vercel, Netlify, Railway, Render — SSL'ni **avtomatik** sozlaydi. Custom domain qo'shsangiz, ular Let's Encrypt sertifikatini o'zlari oladi va yangilaydi. Sizga hech narsa qilish kerak emas.

### VPS'da SSL (certbot)

VPS'da SSL'ni **o'zingiz** sozlaysiz. Buning uchun **certbot** (Let's Encrypt rasmiy vositasi) ishlatiladi:

```bash
sudo certbot --nginx -d example.uz -d www.example.uz
```

Bu buyruq SSL sertifikat oladi, Nginx'ga o'rnatadi va HTTP→HTTPS redirect qo'shadi. To'liq jarayon **06-mavzuda** (VPS'da Nginx + SSL).

**💡 Tushuncha:** Let's Encrypt sertifikati **90 kun** amal qiladi, keyin yangilanishi kerak. Certbot buni avtomatik qiladi (systemd timer orqali). Platformalarda esa umuman o'ylab o'tirmaysiz.

---

## www vs non-www va redirect

Sayt ikki manzilda ochilishi mumkin:
- `https://example.uz` (non-www)
- `https://www.example.uz` (www)

**⚠️ Ehtiyot bo'l:** Ikkalasi ham alohida ishlashi SEO uchun **yomon** — Google ularni ikki xil sayt deb hisoblab, reytingni ikkiga bo'lib yuboradi. Bittasini **asosiy** qilib tanlang va ikkinchisini unga **redirect** qiling.

Qaysi birini tanlash — o'zingizga bog'liq (2026'da ko'pchilik non-www `example.uz` ni afzal ko'radi, qisqaroq). Muhimi — bittada to'xtash.

**Platformalarda:** Vercel/Netlify panelida "Redirect www to non-www" (yoki teskarisi) tugmasi bor — bir marta bosasiz.

**VPS'da (Nginx):** redirect'ni Nginx config'da qo'shasiz:

```text
server {
    server_name www.example.uz;
    return 301 https://example.uz$request_uri;
}
```

Ikkala manzil uchun ham DNS record (A yoki CNAME) bo'lishi shart, aks holda redirect ishlamaydi.

---

## Subdomain sozlash

Subdomain — asosiy domen oldidagi qism: `api.example.uz`, `blog.example.uz`, `admin.example.uz`. Ular **bepul** va cheksiz yaratasiz.

Masalan backend API'ni `api.example.uz` da ishlatmoqchisiz:

**Agar API VPS'da bo'lsa (A record):**

```text
Type: A
Name: api
Value: 185.203.11.42
```

**Agar API Railway/Render'da bo'lsa (CNAME):**

```text
Type: CNAME
Name: api
Value: your-api.up.railway.app
```

Endi `api.example.uz` sizning backendingizga yo'naltiriladi. Bu keng tarqalgan amaliyot: frontend `example.uz`, backend `api.example.uz`. To'liq API deploy — **04-mavzuda**.

**💡 Tushuncha:** Subdomain uchun har doim SSL alohida sozlanadi. Platformada avtomatik, VPS'da certbot buyrug'iga `-d api.example.uz` qo'shasiz.

---

## Cloudflare: DNS + proxy + CDN

**Cloudflare** — bu DNS + xavfsizlik + tezlik platformasi. Bepul rejasi juda kuchli. Nima uchun tavsiya etiladi:

- **Bepul DNS** — tez va ishonchli DNS boshqaruvi.
- **CDN (Content Delivery Network)** — saytingiz fayllarini dunyo bo'ylab kesh qiladi, mijozga eng yaqin serverdan yetkazadi → sayt tezroq ochiladi.
- **DDoS himoya** — hujumlardan bepul himoya.
- **Bepul SSL** — Cloudflare o'zi HTTPS beradi.
- **Proxy** — real server IP'ingizni yashiradi (xavfsizlik).

### Cloudflare'ga o'tish (qisqacha)

1. `cloudflare.com` da account oching, domeningizni qo'shing.
2. Cloudflare mavjud DNS record'larni o'qiydi.
3. Registrar panelida nameserverlarni Cloudflare'nikiga o'zgartiring.
4. DNS record qo'shganda **"Proxy status"** ustuni bor:
   - 🟠 **Proxied (to'q sariq bulut)** — trafik Cloudflare orqali o'tadi (CDN, himoya, IP yashirin).
   - ⚪ **DNS only (kulrang bulut)** — Cloudflare faqat DNS, to'g'ridan-to'g'ri serverga.

**⚠️ Ehtiyot bo'l:** VPS'da certbot bilan SSL olayotganda, o'sha record'ni vaqtincha **"DNS only"** ga o'ting. Proxy yoqiq bo'lsa, certbot Let's Encrypt tekshiruvidan o'tolmaydi. SSL o'rnatib bo'lgach, proxy'ni qayta yoqing.

**💡 Tushuncha:** Cloudflare "Proxied" rejimida ishlatsangiz, `dig example.uz` sizga Cloudflare IP'sini ko'rsatadi (real VPS IP emas) — bu normal, IP yashirilgani uchun.

---

## Savol-Javob (Troubleshooting)

### ❓ Domenni sozladim, lekin sayt ochilmayapti — nima qilay?

**✅ Javob:** Ketma-ket tekshiring:
1. DNS to'g'ri sozlanganmi? `dig example.uz +short` — to'g'ri IP chiqyaptimi?
2. Propagation tugaganmi? `whatsmydns.net` da tekshiring.
3. Server ishlayaptimi? VPS'da Nginx/backend ishlab turibdimi (`curl localhost`)?
4. Brauzer cache — `ipconfig /flushdns` yoki boshqa qurilmada oching.
Ko'p hollarda muammo — shunchaki propagation tugamagani, biroz kuting.

### ❓ "DNS_PROBE_FINISHED_NXDOMAIN" xatosi chiqyapti?

**✅ Javob:** Bu — DNS domenni topa olmayapti degani. Sabablari:
- Record umuman qo'shilmagan yoki noto'g'ri saqlangan.
- Nameserverni o'zgartirgansiz, lekin yangi nameserverda record yo'q.
- Propagation hali tugamagan (yangi domen — 1-2 soat kuting).
Registrar/Cloudflare panelida A record borligini qayta tekshiring.

### ❓ CNAME record qo'shmoqchiman, lekin "@" (asosiy domen)ga qo' shmayapti?

**✅ Javob:** To'g'ri — klassik DNS qoidasi bo'yicha asosiy domen (`@`) uchun CNAME ishlatib bo'lmaydi. Yechim: asosiy domen uchun **A record** ishlating (platforma bergan IP bilan), yoki **Cloudflare** ishlating — u "CNAME flattening" orqali `@` ga ham CNAME qo'yishga ruxsat beradi.

### ❓ SSL o'rnatildi, lekin brauzer "Not Secure" deyapti?

**✅ Javob:** Odatda **mixed content** muammosi: sahifa HTTPS'da, lekin ichida `http://` bilan yuklangan rasm/skript bor. Brauzer console'da (F12) qaysi resurs `http://` orqali kelayotganini ko'ring va uni `https://` ga o'zgartiring. Yoki HTTP→HTTPS redirect sozlanmagan bo'lishi mumkin.

### ❓ `certbot` "Timeout / challenge failed" xatosini beryapti?

**✅ Javob:** Let's Encrypt domeningizga kirib tekshira olmayapti. Sabablari:
- DNS hali propagate bo'lmagan (A record IP'ni to'g'ri ko'rsatyaptimi tekshiring).
- **Cloudflare proxy (to'q sariq bulut) yoqiq** — vaqtincha "DNS only" ga o'ting.
- 80-port (HTTP) firewall'da yopiq — `sudo ufw allow 80` qiling.

### ❓ Nameserverni Cloudflare'ga o'zgartirdim, sayt o'chib qoldi?

**✅ Javob:** Nameserver o'zgarsa, eski DNS record'lar ishlamay qoladi. Cloudflare panelida barcha kerakli record'larni (A, CNAME, MX) **qaytadan** kiritganingizga ishonch hosil qiling. Nameserver o'zgarishi 24 soatgacha vaqt olishi mumkin.

### ❓ `.uz` domenini xalqaro registrar'dan (Namecheap) olsam bo'ladimi?

**✅ Javob:** Odatda yo'q — `.uz` ni CCTLD.UZ boshqaradi va uni faqat akkreditatsiyalangan mahalliy provayderlar sotadi. `.uz` uchun mahalliy registrar (masalan `ahost.uz`) dan foydalaning. Xalqaro `.com`/`.dev` uchun esa Cloudflare/Porkbun qulay.

### ❓ Domen qimmatga uzaydi (renewal) — normalmi?

**✅ Javob:** Ba'zi registrar'larda (ayniqsa GoDaddy) birinchi yil arzon, keyingi yillar qimmat. Bu ataylab qilingan marketing. Yechim: domenni arzon renewal beradigan registrar'ga (Cloudflare, Porkbun) **transfer** qiling. Transfer odatda 1 yil uzaytirishni ham qo'shadi.

### ❓ www bilan ochiladi, lekin www'siz ochilmaydi (yoki teskarisi)?

**✅ Javob:** Ikkala variant uchun ham DNS record bo'lishi kerak. `example.uz` uchun A record, `www` uchun A yoki CNAME record qo'shing. Keyin platformada yoki Nginx'da bittasidan ikkinchisiga redirect sozlang.

### ❓ DNS o'zgarishimni tezlashtirish mumkinmi?

**✅ Javob:** To'g'ridan-to'g'ri emas, lekin **TTL** ni pasaytirish yordam beradi. O'zgarishdan **oldin** TTL'ni kichik qiling (masalan 300 = 5 daqiqa). Keyingi o'zgarishlar tezroq tarqaladi. Yangi domenlar odatda tez propagate bo'ladi.

### ❓ Subdomain (api.example.uz) SSL bermayapti?

**✅ Javob:** Har bir subdomain uchun SSL alohida kerak. Platformada subdomenni loyihaga qo'shsangiz avtomatik bo'ladi. VPS'da certbot buyrug'iga subdomenni qo'shing: `sudo certbot --nginx -d api.example.uz`. Wildcard sertifikat (`*.example.uz`) ham mumkin, lekin DNS challenge talab qiladi.

---

## Amaliy Checklist

- [ ] Domen sotib olindi (Cloudflare/Porkbun/Namecheap yoki `.uz` uchun mahalliy)
- [ ] Auto-renew (avtomatik uzaytirish) yoqilgan
- [ ] TLD to'g'ri tanlangan (.com/.uz/.dev — loyihaga mos)
- [ ] DNS boshqaruvi tanlangan (registrar yoki Cloudflare nameserver)
- [ ] A record (yoki CNAME) asosiy domen uchun qo'shildi
- [ ] www uchun record qo'shildi
- [ ] Platformaga (Vercel/Netlify) yoki VPS IP'siga ulandi
- [ ] `dig` / `nslookup` bilan DNS tekshirildi
- [ ] `whatsmydns.net` da propagation yashil ✓
- [ ] SSL/HTTPS ishlayapti (🔒 qulf ko'rinadi, `.dev` bo'lsa majburiy)
- [ ] www ↔ non-www redirect sozlandi
- [ ] Kerak bo'lsa subdomain (api.) qo'shildi va SSL oldi
- [ ] Cloudflare CDN/proxy (ixtiyoriy) yoqildi

---

← [Deployment bo'limiga qaytish](./README.md)
