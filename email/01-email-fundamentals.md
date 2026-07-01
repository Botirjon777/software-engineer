# Email Asoslari va Jo'natish

Email 1970-yillardan beri mavjud, lekin 2026-yilda ham u eng ishonchli va universal aloqa kanali bo'lib qolmoqda: parolni tiklash, buyurtma tasdig'i, hisob-faktura, xabarnomalar — bularning barchasi email orqali boradi. Muammo shundaki, email jo'natish "SMTP server ochib xat yubordim" degan darajada oddiy emas. Bugungi kunda internetning katta qismi spam bilan kurashadi, va agar siz email'ni noto'g'ri jo'natsangiz, u user'ning inbox'iga umuman yetib bormaydi — Spam papkasida qoladi yoki butunlay bloklanadi.

Bu hujjat email qanday ishlashini (protokollar, serverlar), nega o'z serveringizdan to'g'ridan-to'g'ri jo'natmaslik kerakligini, qaysi email service'ni tanlashni va eng muhimi — **deliverability** (email'ning haqiqatan yetib borishi) ni chuqur yoritadi. Misollar Node.js'da (Nodemailer va Resend SDK).

**💡 Tushuncha:** Email jo'natishda eng qiyin qism kod yozish emas — kod bir necha qator. Eng qiyin qism email'ning **spam'ga tushmasligini** ta'minlash. Shuning uchun bu hujjatning yarmi deliverability haqida.

## Mundarija

- [Email qanday ishlaydi](#email-qanday-ishlaydi)
- [Email yo'li: jo'natuvchidan qabul qiluvchigacha](#email-yoli-jonatuvchidan-qabul-qiluvchigacha)
- [Nega o'z serveringizdan to'g'ridan-to'g'ri jo'natmaslik kerak](#nega-oz-serveringizdan-togridan-togri-jonatmaslik-kerak)
- [Email service'lar: batafsil taqqos](#email-servicelar-batafsil-taqqos)
- [Transactional vs Marketing email](#transactional-vs-marketing-email)
- [SMTP vs API orqali jo'natish](#smtp-vs-api-orqali-jonatish)
- [Node.js'da jo'natish](#nodejsda-jonatish)
- [Deliverability: eng muhim qism](#deliverability-eng-muhim-qism)
- [SPF, DKIM, DMARC](#spf-dkim-dmarc)
- [Sender reputation va IP warm-up](#sender-reputation-va-ip-warm-up)
- [Bounce, complaint va list hygiene](#bounce-complaint-va-list-hygiene)
- [Domain sozlash](#domain-sozlash)
- [Testing](#testing)
- [Savol-Javob](#savol-javob)
- [Masalalar](#masalalar)

---

## Email qanday ishlaydi

Email tizimi bir necha komponentdan tashkil topgan. Ularning nomlarini bilish muhim, chunki intervyularda va dokumentatsiyalarda tez-tez uchraydi.

**MUA (Mail User Agent)** — bu siz email o'qiydigan/yozadigan dastur: Gmail veb-interfeysi, Outlook, Apple Mail, yoki mobil ilova. User email bilan ishlaydigan har qanday client.

**MTA (Mail Transfer Agent)** — email'ni serverdan serverga uzatuvchi. Bu "pochta tashuvchisi". Masalan Postfix, Exim, yoki email service'larning (SendGrid, SES) ichki serverlari. MTA'lar bir-biri bilan **SMTP** protokoli orqali gaplashadi.

**MDA (Mail Delivery Agent)** — email'ni qabul qiluvchining pochta qutisiga (mailbox) yakuniy joylaydigan komponent. MTA email'ni kerakli serverga yetkazgach, MDA uni to'g'ri user papkasiga tashlaydi.

**💡 Tushuncha:** Oddiy analogiya: MUA — bu siz (xat yozuvchi), MTA — pochta tashuvchi mashinalar va bo'limlar, MDA — sizning pochta qutingizga xatni tashlaydigan pochtalyon.

### Protokollar

Email uchta asosiy protokoldan foydalanadi. Ularni aralashtirib yubormaslik kerak:

**SMTP (Simple Mail Transfer Protocol)** — email **jo'natish** uchun. MUA'dan MTA'ga va MTA'dan MTA'ga xat uzatiladi. SMTP faqat jo'natish uchun — o'qish uchun emas.

SMTP portlari (bu intervyu savoli sifatida tez-tez so'raladi):

```text
Port 25   — MTA-dan-MTA-ga (server-to-server relay). ISP'lar buni ko'pincha bloklaydi.
Port 587  — Submission port. MUA/ilova-dan MTA-ga jo'natish uchun ZAMONAVIY standart.
            STARTTLS bilan shifrlash. ENG KO'P TAVSIYA ETILADI.
Port 465  — SMTPS (implicit TLS). Boshida deprecated qilingan, keyin qayta tiklangan.
            Ulanish boshidanoq TLS bilan.
```

**⚠️ Ehtiyot bo'l:** Port 25'ni ilovangizdan email jo'natish uchun ISHLATMANG. U server-to-server relay uchun mo'ljallangan va aksariyat cloud provayderlar (AWS, GCP, DigitalOcean) uni tashqi trafik uchun bloklaydi. Ilovadan jo'natish uchun **587** (yoki 465) ishlating.

**IMAP (Internet Message Access Protocol)** — email **o'qish** uchun. Email server'da qoladi, client sinxronizatsiya qiladi. Bir nechta qurilmadan bir xil inbox'ni ko'rish mumkin (telefon + noutbuk). Bugungi standart.

**POP3 (Post Office Protocol v3)** — email o'qishning eski usuli. Email'ni serverdan **yuklab oladi va o'chiradi** (odatda). Bitta qurilma uchun. Hozir kam ishlatiladi.

**💡 Tushuncha:** Backend dasturchi sifatida siz asosan **SMTP jo'natish** bilan ishlaysiz. IMAP/POP3 ko'proq email client yozganda kerak bo'ladi. Transactional email jo'natishda IMAP/POP3 deyarli hech qachon kerak bo'lmaydi.

---

## Email yo'li: jo'natuvchidan qabul qiluvchigacha

Bir email `ali@example.com` dan `vali@gmail.com` ga qanday boradi? Mana to'liq yo'l:

```text
┌──────────────┐
│  Ali (MUA)   │  1. Ali xat yozadi, "Yuborish" bosadi
│  yoki ilova  │
└──────┬───────┘
       │ SMTP (port 587, TLS)
       ▼
┌──────────────────┐
│  Jo'natuvchi MTA │  2. example.com pochta serveri (yoki email service)
│  (example.com)   │     xatni qabul qiladi
└──────┬───────────┘
       │ DNS: gmail.com uchun MX record'ni so'raydi
       ▼
┌──────────────────┐
│  DNS server      │  3. "gmail.com xatlarini qaysi serverga yuboray?"
│  MX lookup       │     → gmail-smtp-in.l.google.com
└──────┬───────────┘
       │ SMTP (port 25, server-to-server)
       ▼
┌──────────────────┐
│  Qabul qiluvchi  │  4. Gmail serveri xatni qabul qiladi.
│  MTA (Gmail)     │     SPF/DKIM/DMARC tekshiradi, spam filtri ishlaydi
└──────┬───────────┘
       │ MDA
       ▼
┌──────────────────┐
│  Vali mailbox    │  5. Xat Inbox (yoki Spam) papkasiga tushadi
│  (MDA joylaydi)  │
└──────┬───────────┘
       │ IMAP/POP3
       ▼
┌──────────────┐
│  Vali (MUA)  │  6. Vali telefonida/noutbukida xatni o'qiydi
└──────────────┘
```

Bu yerda 4-qadam kritik: **qabul qiluvchi MTA (Gmail) sizning email'ingizni tekshiradi**. SPF, DKIM, DMARC to'g'ri sozlanmagan bo'lsa yoki IP reputatsiyangiz yomon bo'lsa — email Spam'ga tushadi yoki rad etiladi. Deliverability haqidagi bo'limda buni chuqur ko'ramiz.

---

## Nega o'z serveringizdan to'g'ridan-to'g'ri jo'natmaslik kerak

Ko'p yangi dasturchilar "VPS oldim, Postfix o'rnataman, o'zim email jo'nataman, tekin!" deb o'ylaydi. Bu deyarli har doim yomon g'oya. Sabablari:

**1. Port 25 bloklangan.** Aksariyat cloud provayderlar (AWS EC2, GCP, DigitalOcean, Hetzner) yangi serverlar uchun tashqi 25-portni bloklaydi — aynan spam'ni oldini olish uchun. Blokni ochish uchun murojaat qilsangiz ham, ko'pincha rad etadi.

**2. IP reputatsiya nolga teng.** Yangi server IP'sining email jo'natish tarixi yo'q. Gmail/Outlook uchun siz "notanish"siz. Notanish IP'dan kelgan email darrov shubha ostida — ko'pincha to'g'ri Spam'ga. Reputatsiya orttirish (warm-up) oylar davom etadi va juda nozik ish.

**3. Blacklist'lar.** IP'ingiz allaqachon qora ro'yxatda (Spamhaus, SORBS) bo'lishi mumkin — chunki oldingi egasi spam yuborgan. Buni tekshirish va tozalash og'riqli.

**4. Deliverability infratuzilma.** SPF, DKIM, DMARC, reverse DNS/PTR, feedback loop'lar, bounce handling — bularning barchasini o'zingiz sozlashingiz va boshqarishingiz kerak. Bu to'liq vaqtli ish.

**5. Monitoring va yetkazib berish tezligi.** Gmail bugun qoidalarni o'zgartiradi, Outlook ertaga. Email service'lar bu bilan doim shug'ullanadi, siz esa yo'q.

**⚠️ Ehtiyot bo'l:** "O'zim SMTP server ochaman" — 2026-yilda bu deyarli har doim xato. Katta kompaniyalar (jiddiy resurs bilan) o'z infratuzilmasini quradi, lekin startap yoki oddiy loyiha uchun **email service ishlatish** yagona oqilona yo'l. Vaqtingizni saqlaydi va email'laringiz yetib boradi.

Yechim: **email service provider** — SPF/DKIM/reputatsiya/bounce'larni siz uchun boshqaradigan tashqi xizmat.

---

## Email service'lar: batafsil taqqos

Email service'lar ikki katta guruhga bo'linadi: **transactional** (bitta user'ga, hodisaga bog'liq: welcome, parol tiklash, buyurtma) va **marketing/campaign** (ko'p user'ga, newsletter, aksiya). Avval transactional'larni ko'ramiz.

### Transactional email service'lar

| Service | Afzalligi | Narx (2026, taxminan) | Qachon ishlatish |
|---------|-----------|----------------------|------------------|
| **Resend** | Zamonaviy, dev-friendly, React Email bilan zo'r integratsiya, chiroyli API va DX | Bepul: ~3,000/oy. Pro: ~$20/oy 50k email | Yangi loyiha, React/Next.js stack, tez boshlash |
| **Postmark** | Deliverability **eng yaxshi**, tez yetkazish, transactional'ga ixtisoslashgan | ~$15/oy 10k email | Deliverability kritik bo'lganda (parol, hisob-faktura) |
| **Amazon SES** | **Eng arzon**, cheksiz scale, AWS ekotizim | ~$0.10 / 1,000 email | Katta hajm, AWS'da o'tirgan, narx muhim |
| **SendGrid** | Yetuk, keng imkoniyatlar, katta hajmga chidamli | Bepul: ~100/kun. Keyin ~$20/oy dan | Enterprise, aralash (transactional + marketing) |
| **Mailgun** | Developer-friendly, kuchli API, email validatsiya | ~$15/oy dan | API-first, email validatsiya kerak bo'lganda |

**Resend** — 2023-yilda chiqib, tez ommalashdi. Zamonaviy DX, React Email bilan native ishlaydi (JSX'da email yozasiz). Yangi loyihalar uchun eng ko'p tavsiya etiladigan tanlov. Kichik va o'rta hajm uchun ideal.

**Postmark** — agar sizga deliverability muhim bo'lsa (masalan, parol tiklash email'i albatta yetib borishi kerak), Postmark klassik tanlov. Ular faqat transactional'ga e'tibor beradi va marketing spam'ni umuman qabul qilmaydi — shu sabab ularning IP'lari toza va reputatsiyasi zo'r.

**Amazon SES** — juda arzon va cheksiz scale. Lekin DX yomonroq, ko'proq sozlash kerak, va boshida sizni "sandbox" rejimida ushlab turadi (faqat verified email'larga jo'nata olasiz). Katta hajm va narx muhim bo'lganda eng yaxshi tanlov.

**SendGrid** (Twilio egalik qiladi) — yetuk, ishonchli, ham transactional ham marketing qiladi. Enterprise'da mashhur. Interfeysi biroz og'irroq.

**Mailgun** — API-first, developer'lar uchun qulay, email validatsiya (real email ekanini tekshirish) xususiyati kuchli.

**💡 Tushuncha:** 2026-yilda yangi loyiha uchun oddiy tavsiya: **Resend** bilan boshlang (DX zo'r, tez). Hajm o'sib narx muammo bo'lsa yoki deliverability juda kritik bo'lsa — **SES** (arzon) yoki **Postmark** (deliverability) ga o'ting.

### Marketing/campaign service'lar

Bular newsletter, aksiya, drip-kampaniyalar uchun. Ularda vizual editor, list segmentatsiya, A/B test, avtomatlashtirish bor:

- **Mailchimp** — eng mashhur, oson, kichik biznes uchun. Qimmatroq bo'lib boradi.
- **Brevo** (avvalgi Sendinblue) — email + SMS, arzon, o'rta biznes uchun yaxshi.
- **Loops** — zamonaviy, SaaS loyihalar uchun, dev-friendly marketing avtomatlashtirish.

Bu service'lar va kampaniya boshqaruvi keyingi mavzuda ([Template va Kampaniyalar](./02-templates-campaigns.md)) batafsil.

---

## Transactional vs Marketing email

Bu farqni tushunish muhim, chunki ular boshqacha yondashuvni talab qiladi:

| Xususiyat | Transactional | Marketing |
|-----------|---------------|-----------|
| Kimga | Bitta user'ga (1:1) | Ko'p user'ga (1:N) |
| Sabab | User harakati (ro'yxatdan o'tdi, buyurtma berdi) | Siz qaror qildingiz (aksiya, newsletter) |
| Misol | Welcome, parol tiklash, chek, xabarnoma | Chegirma, yangilik, oylik xat |
| Kutilma | User **kutmoqda** — tez yetishi kerak | User kutmayotgan bo'lishi mumkin |
| Unsubscribe | Odatda shart emas (lekin tavsiya) | **Majburiy** (qonun talabi) |
| Open rate | Yuqori (~50-80%) | Past (~20-30%) |
| IP/domain | Alohida saqlash tavsiya | Alohida saqlash tavsiya |

**⚠️ Ehtiyot bo'l:** Transactional va marketing email'ni **bir xil domen/IP'dan aralashtirib yubormang**. Agar marketing email'laringiz spam report olsa, bu reputatsiyani buzadi va parol tiklash kabi kritik transactional email'laringiz ham spam'ga tusha boshlaydi. Ko'p kompaniyalar subdomen ajratadi: `mail.example.com` (marketing) va `notify.example.com` (transactional).

---

## SMTP vs API orqali jo'natish

Email service'ga email'ni ikki yo'l bilan topshirish mumkin: **SMTP** yoki **HTTP API**.

**SMTP orqali:** Service sizga SMTP host, port, username, password beradi. Siz Nodemailer kabi kutubxona bilan ulanaib jo'natasiz.

- Afzalligi: **universal standart**. Har qanday til/framework SMTP bilan ishlay oladi. Service almashtirmoqchi bo'lsangiz, faqat credential o'zgartirasiz — kod deyarli o'zgarmaydi. Legacy tizimlar bilan mos.
- Kamchiligi: sekinroq (bir necha round-trip), ulanishni boshqarish kerak, xatoliklar ba'zan noaniq.

**API orqali:** Service HTTP REST API beradi (masalan Resend SDK, SendGrid API). Siz JSON POST qilasiz.

- Afzalligi: **tezroq va ishonchliroq**, boy javob (message ID, status), webhook'lar bilan yaxshi integratsiya (bounce/open/click tracking), zamonaviy DX.
- Kamchiligi: service'ga bog'lanib qolasiz (vendor lock-in) — almashtirish uchun kod o'zgartirish kerak.

**💡 Tushuncha:** Qachon qaysi biri? **Yangi loyihada API ishlating** (Resend SDK, SendGrid SDK) — tezroq, DX yaxshiroq, tracking oson. **SMTP ishlating** agar: eski tizimingiz allaqachon SMTP'da; ko'p service o'rtasida oson almashishni istaysiz; yoki tilingizda yaxshi SDK yo'q.

---

## Node.js'da jo'natish

### Nodemailer (SMTP orqali)

Nodemailer — Node.js'da eng mashhur email kutubxonasi. SMTP orqali jo'natadi.

```bash
npm install nodemailer
```

```js
import nodemailer from "nodemailer";

// Transporter — SMTP ulanish konfiguratsiyasi
const transporter = nodemailer.createTransport({
  host: "smtp.resend.com", // service bergan SMTP host
  port: 587,               // submission port (STARTTLS)
  secure: false,           // 587 uchun false (STARTTLS), 465 uchun true
  auth: {
    user: "resend",
    pass: process.env.RESEND_API_KEY, // credential'ni HECH QACHON kodga yozmang
  },
});

async function sendWelcomeEmail(to) {
  const info = await transporter.sendMail({
    from: '"MyApp" <welcome@myapp.uz>', // verified domendan bo'lishi shart
    to,
    subject: "MyApp'ga xush kelibsiz!",
    text: "Salom! Ro'yxatdan o'tganingiz uchun rahmat.", // plain-text versiya
    html: "<h1>Xush kelibsiz!</h1><p>Ro'yxatdan o'tganingiz uchun rahmat.</p>",
  });

  console.log("Yuborildi:", info.messageId);
  return info;
}

await sendWelcomeEmail("user@example.com");
```

**⚠️ Ehtiyot bo'l:** Har doim `text` (plain-text) versiyasini ham qo'shing, faqat `html` emas. Spam filtrlari plain-text yo'q email'larni shubhali deb biladi, va ba'zi client'lar HTML'ni ko'rsata olmaydi. `from` manzili **verified domen**dan bo'lishi shart.

### Resend SDK (API orqali)

```bash
npm install resend
```

```ts
import { Resend } from "resend";

const resend = new Resend(process.env.RESEND_API_KEY);

async function sendWelcomeEmail(to: string) {
  const { data, error } = await resend.emails.send({
    from: "MyApp <welcome@myapp.uz>", // verified domen
    to,
    subject: "MyApp'ga xush kelibsiz!",
    html: "<h1>Xush kelibsiz!</h1><p>Rahmat!</p>",
    text: "Xush kelibsiz! Rahmat!",
  });

  if (error) {
    console.error("Xato:", error);
    throw new Error(error.message);
  }

  console.log("Yuborildi, ID:", data.id);
  return data;
}
```

API yondashuvida ko'rib turganingizdek — javob boy (`data.id`, `error` obyekti), xatolarni boshqarish oson. Resend React Email bilan ham ishlaydi (keyingi mavzuda).

**💡 Tushuncha:** Real ilovada email jo'natishni **fon vazifasiga (background job / queue)** o'tkazing (masalan BullMQ, yoki service'ning o'z queue'si). Foydalanuvchi so'rovini email jo'natilgunча kutdirmang — email jo'natish sekin va ba'zan xato beradi. HTTP javobni darrov qaytaring, email'ni orqada jo'nating.

---

## Deliverability: eng muhim qism

Deliverability — email'ning haqiqatan **inbox'ga** yetib borishi (Spam emas, rad etilmagan). Bu email jo'natishning eng qiyin va eng muhim qismi. Kod ishlayapti, "yuborildi" chiqyapti — lekin user email'ni ko'rmayapti? Deliverability muammosi.

Qabul qiluvchi server (Gmail, Outlook) email'ni qabul qilishdan oldin bir necha narsani tekshiradi:

1. **Autentifikatsiya** — bu email haqiqatan shu domendan kelganmi? (SPF, DKIM, DMARC)
2. **Reputatsiya** — jo'natuvchi IP/domen ishonchlimi?
3. **Kontent** — spam'ga o'xshaydimi? (so'zlar, linklar, HTML/text nisbati)
4. **Engagement** — oldingi email'laringizni user'lar ochadimi, spam deb belgilaydimi?

Keling, har birini ko'ramiz.

---

## SPF, DKIM, DMARC

Bu uchtasi email autentifikatsiyasining ustunlari. Ular DNS record'lar orqali sozlanadi. Email service (Resend/SES) sizga aniq qiymatlarni beradi, siz domeningizning DNS'iga qo'shasiz.

### SPF (Sender Policy Framework)

SPF — "qaysi serverlar mening domenim nomidan email jo'nata oladi?" degan ro'yxat. DNS'da **TXT record** sifatida saqlanadi.

Qabul qiluvchi server email kelganda: "Bu email `myapp.uz`dan kelibdi. `myapp.uz`ning SPF record'iga qarayman — bu jo'natuvchi IP ruxsat etilganmi?" Agar yo'q bo'lsa — soxta bo'lishi mumkin, shubhali.

```text
Type:  TXT
Name:  myapp.uz  (yoki @)
Value: v=spf1 include:_spf.resend.com include:amazonses.com -all
```

- `v=spf1` — SPF versiyasi
- `include:_spf.resend.com` — Resend serverlariga jo'natishga ruxsat
- `-all` — ro'yxatda yo'q boshqa hamma narsani **rad et** (hard fail). `~all` — soft fail (belgilaydi lekin qabul qiladi)

**⚠️ Ehtiyot bo'l:** Bitta domenga **faqat bitta** SPF record bo'lishi mumkin. Bir nechta service ishlatsangiz, hammasini bitta record'ga `include` qiling. Ikkita alohida `v=spf1` record — xato va SPF butunlay ishlamaydi.

### DKIM (DomainKeys Identified Mail)

DKIM — email'ga **kriptografik imzo** qo'yadi. Bu email yo'lda o'zgartirilmaganini va haqiqatan shu domendan kelganini isbotlaydi. Soxtalashtirishni (spoofing) oldini oladi.

Ishlash tartibi: jo'natuvchi server email'ni **maxfiy kalit** bilan imzolaydi (header'ga `DKIM-Signature` qo'shadi). **Ochiq kalit** DNS'da TXT record sifatida turadi. Qabul qiluvchi server ochiq kalit bilan imzoni tekshiradi — mos kelsa, email haqiqiy.

```text
Type:  TXT
Name:  resend._domainkey.myapp.uz
Value: p=MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQC... (uzun ochiq kalit)
```

`resend._domainkey` qismidagi `resend` — bu **selector** (service o'zi belgilaydi). Bir domen bir nechta DKIM kalitiga ega bo'lishi mumkin (har xil service uchun), selector ular orasidan tanlaydi.

**💡 Tushuncha:** SPF "qaysi server ruxsat etilgan" ni tekshiradi (IP asosida), DKIM esa "email o'zgartirilmagan va haqiqiy" ligini kriptografik isbotlaydi. Ikkalasi birga ishlaydi — biri boshqasini to'ldiradi.

### DMARC (Domain-based Message Authentication, Reporting & Conformance)

DMARC — SPF va DKIM ustidan **siyosat** o'rnatadi: "Agar SPF yoki DKIM muvaffaqiyatsiz bo'lsa, email bilan nima qilay?" Va bu haqda hisobot (report) yuboradi.

```text
Type:  TXT
Name:  _dmarc.myapp.uz
Value: v=DMARC1; p=quarantine; rua=mailto:dmarc@myapp.uz; adkim=s; aspf=s
```

- `p=` — siyosat: `none` (hech narsa, faqat kuzat), `quarantine` (Spam'ga tashla), `reject` (butunlay rad et)
- `rua=mailto:...` — agregat hisobotlarni shu manzilga yubor
- `adkim=s; aspf=s` — qat'iy (strict) moslik talab qilinadi

**💡 Tushuncha:** DMARC'ni bosqichma-bosqich joriy qiling: avval `p=none` (kuzatuv, hisobot yig'ing, hech narsani buzmaydi), keyin `p=quarantine`, oxirida `p=reject`. Darrov `reject` qo'ymang — noto'g'ri sozlangan bo'lsa, o'z legitim email'laringizni bloklaysiz.

**Nima uchun uchalasi kerak?** 2024-yildan Gmail va Yahoo katta jo'natuvchilar uchun SPF + DKIM + DMARC'ni **majburiy** qildi. 2026-yilda bu barcha jiddiy jo'natuvchi uchun standart. Bularsiz email'ingiz to'g'ri Spam'ga tushadi.

### Reverse DNS / PTR

**PTR record** — IP manzilni domen nomiga qaytaradi (forward DNS'ning teskarisi). Qabul qiluvchi server: "Bu email `1.2.3.4` IP'dan keldi. `1.2.3.4`ning PTR'iga qaray — u haqiqiy domenga ishora qiladimi?" PTR yo'q yoki mos kelmasa — shubhali.

**💡 Tushuncha:** Email service ishlatsangiz, PTR'ni **service o'zi boshqaradi** — sizga tashvish yo'q. Bu faqat o'z serveringizdan jo'natganda muammo bo'ladi (yana bir sabab: service ishlating).

---

## Sender reputation va IP warm-up

**Sender reputation** — Gmail/Outlook sizning IP'ingiz va domeningizga bergan "ishonch bahosi". Yaxshi reputatsiya = inbox. Yomon = Spam yoki bloklash. Reputatsiyaga ta'sir qiladi: spam report'lar soni, bounce nisbati, engagement (ochish/bosish), jo'natish hajmining barqarorligi.

**IP warm-up** — yangi IP'dan darrov katta hajmda email jo'natsangiz, provayderlar buni spam deb biladi. Shuning uchun hajmni **asta-sekin oshirish** kerak: 1-kun 50 email, 2-kun 100, keyin 200... bir necha hafta davomida. Bu IP'ni "isitadi".

**Dedicated vs Shared IP:**

- **Shared IP** — bir nechta mijoz bir IP'dan jo'natadi (service tomonidan boshqariladi). Afzalligi: reputatsiya allaqachon tayyor, warm-up shart emas, kichik hajm uchun ideal. Kamchiligi: boshqa mijoz spam yuborsa, sizga ta'sir qilishi mumkin.
- **Dedicated IP** — faqat sizga tegishli IP. Afzalligi: to'liq nazorat, katta hajm uchun barqaror. Kamchiligi: qimmat, warm-up kerak, va **doimiy yuqori hajm** talab qiladi (aks holda reputatsiya "sovib" ketadi).

**💡 Tushuncha:** Oyiga ~100k email'dan kam jo'natsangiz — **shared IP** ishlating (default). Dedicated IP faqat oyiga million+ email va barqaror hajm bo'lganda mantiqiy.

---

## Bounce, complaint va list hygiene

**Bounce** — email yetkazib bo'lmadi, qaytib keldi. Ikki xil:

- **Hard bounce** — doimiy xato: email manzil mavjud emas, domen yo'q. Bu manzilga **hech qachon qayta jo'natmang** — darrov ro'yxatdan olib tashlang. Hard bounce nisbati yuqori bo'lsa, reputatsiya keskin tushadi.
- **Soft bounce** — vaqtinchalik xato: mailbox to'la, server band, xabar juda katta. Bir necha marta qayta urinish mumkin, lekin doim soft bounce bersa — olib tashlang.

**Complaint / spam report** — user email'ingizni "Spam" deb belgiladi. Bu eng yomon signal. Complaint nisbati **0.1% dan oshmasligi** kerak (1000 email'dan 1 tadan ko'p emas). Oshsa — provayderlar sizni bloklashi mumkin.

**List hygiene** — email ro'yxatini toza saqlash:

- Hard bounce va complaint bergan manzillarni **darrov olib tashlang**
- Uzoq vaqt (6+ oy) hech narsa ochmagan user'larni tozalang yoki qayta-jalb kampaniyasi qiling
- Email manzillarni **sotib olmang** yoki scrape qilmang — bu spam trap'larga urilib reputatsiyani o'ldiradi
- Ro'yxatga qo'shishda **double opt-in** ishlating (user emailni tasdiqlaydi)

**Unsubscribe** — obunani bekor qilish oson bo'lishi kerak. Marketing email'da **majburiy** (qonun: CAN-SPAM, GDPR). 2024-yildan Gmail/Yahoo **one-click unsubscribe** (`List-Unsubscribe` header) talab qiladi.

```text
List-Unsubscribe: <https://myapp.uz/unsubscribe?token=abc123>
List-Unsubscribe-Post: List-Unsubscribe=One-Click
```

**⚠️ Ehtiyot bo'l:** Unsubscribe'ni qiyinlashtirmang (kirish talab qilish, ko'p qadam). User topa olmasa, o'rniga "Spam" bosadi — bu unsubscribe'dan **ancha** yomonroq reputatsiyaga.

### Spam filtrlari nimaga qaraydi

Qabul qiluvchi spam filtri bir necha omilni ballaydi:

- **Autentifikatsiya** — SPF/DKIM/DMARC bormi va to'g'rimi (eng muhim)
- **Reputatsiya** — IP/domen tarixi, oldingi spam report'lar
- **Kontent** — spam so'zlar ("BEPUL!!!", "HOZIROQ BOSING"), ko'p bosh harf, undov belgilar
- **Link'lar** — shubhali/qisqartirilgan URL'lar, link matni bilan haqiqiy URL mos kelmasligi
- **HTML/text nisbati** — faqat bitta katta rasm va matn yo'q = shubhali. HTML va plain-text ikkalasi bo'lsin
- **Engagement** — user'lar ochadimi, javob beradimi, arxivlaydimi (ijobiy) yoki o'chiradimi/spam bosadimi (salbiy)

**💡 Tushuncha:** Deliverability'ni yaxshilashning eng samarali yo'li — **faqat xohlagan odamlarga, foydali email yuborish**. Engagement yuqori bo'lsa, provayderlar sizni sevadi. Texnik sozlash (SPF/DKIM) shart, lekin engagement uzoq muddatda hal qiluvchi.

---

## Domain sozlash

Email jo'natish uchun o'z domeningizni service'da **verify** qilish kerak:

1. Service'da domen qo'shasiz (masalan `myapp.uz`)
2. Service sizga DNS record'lar beradi: SPF (TXT), DKIM (TXT), ba'zan MX/DMARC
3. Bu record'larni domeningizning DNS panelida (Cloudflare, Namecheap...) qo'shasiz
4. Service verify tugmasini bosasiz — DNS'ni tekshiradi

**From address:** `from` manzili **verified domendan** bo'lishi shart. `noreply@myapp.uz` yoki `hello@myapp.uz`. `gmail.com` yoki `yahoo.com` dan **jo'natmang** — bu ularning DMARC siyosatini buzadi va email rad etiladi.

**Subdomen strategiyasi:** transactional va marketing'ni ajrating:
- `notify.myapp.uz` yoki `mail.myapp.uz` — transactional
- `news.myapp.uz` — marketing

**💡 Tushuncha:** `noreply@` manzilidan foydalanish keng tarqalgan lekin ideal emas — user javob bera olmaydi. Iloji bo'lsa haqiqiy, kuzatiladigan manzil ishlating (`support@`). Bu engagement va ishonchni oshiradi.

---

## Testing

Email jo'natishni ishlab chiqarishga (production) chiqarishdan oldin sinash shart:

- **[mail-tester.com](https://www.mail-tester.com)** — beradigan manzilga email jo'natasiz, u sizga **spam score** (10 dan) va nima yaxshilash kerakligini ko'rsatadi: SPF/DKIM/DMARC holati, kontent muammolari, blacklist'lar. Eng oddiy va foydali test.
- **Service'ning o'z tools'i** — Resend/SendGrid dashboard'ida deliverability, bounce, open/click statistikasi bor.
- **Gmail/Outlook'ga haqiqiy test** — turli provayderlarga jo'nating, Inbox'ga tushdimi yoki Spam'ga qarang. "Show original" bilan SPF/DKIM/DMARC `pass` ligini tekshiring.
- **Dev muhitda** — [Mailtrap](https://mailtrap.io) yoki [Ethereal](https://ethereal.email) kabi "sandbox" SMTP ishlating — email haqiqatan yuborilmaydi, faqat ushlab qoladi. Bu development'da xavfsiz.

**💡 Tushuncha:** mail-tester.com'da **10/10 ball** olishga harakat qiling productionга chiqishdan oldin. Har bir kamchilikni u aniq ko'rsatadi. Bu 5 daqiqalik test ko'p muammolardan qutqaradi.

---

## Savol-Javob

### ❓ SMTP, IMAP va POP3 o'rtasidagi farq nima?

**✅ Javob:** **SMTP** — email **jo'natish** protokoli (MUA→MTA, MTA→MTA). **IMAP** va **POP3** — email **o'qish** protokollari. IMAP email'ni serverda saqlaydi va bir nechta qurilma sinxronlanadi (zamonaviy standart). POP3 email'ni yuklab olib serverdan o'chiradi (eski, bitta qurilma uchun). Backend dasturchi sifatida siz asosan SMTP jo'natish bilan ishlaysiz.

### ❓ Port 25, 587 va 465 o'rtasidagi farq nima? Qaysi birini ishlatay?

**✅ Javob:** **25** — server-to-server relay uchun, ISP'lar bloklaydi, ilovadan ishlatmang. **587** — submission port, STARTTLS bilan, ilovadan jo'natish uchun zamonaviy standart, **shuni ishlating**. **465** — implicit TLS (SMTPS), boshidan shifrlangan ulanish, u ham yaxshi. Amalda 587 (yoki 465) — ilovangizdan email jo'natish uchun.

### ❓ Nega o'z serverimdan to'g'ridan-to'g'ri email jo'natmasligim kerak?

**✅ Javob:** Bir necha sabab: (1) cloud provayderlar port 25'ni bloklaydi; (2) yangi IP'ning reputatsiyasi nol — email darrov Spam'ga tushadi; (3) IP allaqachon blacklist'da bo'lishi mumkin; (4) SPF/DKIM/DMARC/PTR/bounce infratuzilmasini o'zingiz boshqarishingiz kerak — bu to'liq vaqtli ish. Yechim: **email service** (Resend, SES, Postmark) ishlating — ular bularni siz uchun boshqaradi.

### ❓ Transactional va marketing email o'rtasidagi farq nima?

**✅ Javob:** **Transactional** — bitta user'ga, uning harakatiga javoban (welcome, parol tiklash, buyurtma cheki). User kutmoqda, tez yetishi kerak. **Marketing** — ko'p user'ga, siz tashabbuskor (newsletter, aksiya). Unsubscribe majburiy. Ularni **alohida domen/subdomenda** saqlash tavsiya — marketing spam report'lari transactional deliverability'ni buzmasligi uchun.

### ❓ Qaysi email service'ni tanlayin?

**✅ Javob:** Yangi loyiha, tez boshlash, React stack — **Resend**. Deliverability juda kritik (parol, hisob-faktura) — **Postmark**. Katta hajm, arzon, AWS'da — **Amazon SES**. Enterprise, aralash — **SendGrid**. API-first, email validatsiya — **Mailgun**. Ko'pchilik uchun 2026-da Resend'dan boshlash oqilona.

### ❓ SMTP va API orqali jo'natish — qaysi biri yaxshi?

**✅ Javob:** **API** (Resend SDK, SendGrid API) — tezroq, ishonchliroq, boy javob va tracking, zamonaviy DX. Yangi loyihada shuni ishlating. **SMTP** — universal standart, har qanday til bilan ishlaydi, service almashish oson, legacy mos. Eski tizim yoki service moslashuvchanligi kerak bo'lsa SMTP.

### ❓ SPF nima va nima uchun kerak?

**✅ Javob:** SPF (Sender Policy Framework) — domeningiz nomidan email jo'nata oladigan serverlarning ro'yxati (DNS TXT record). Qabul qiluvchi server email kelgan IP shu ro'yxatdami tekshiradi. Bu boshqa birov sizning domeningiz nomidan soxta email jo'natishini qiyinlashtiradi. Bitta domenga faqat bitta SPF record — barcha service'larni `include` bilan qo'shing.

### ❓ DKIM nima uchun kerak, SPF yetarli emasmi?

**✅ Javob:** DKIM email'ga **kriptografik imzo** qo'yadi (maxfiy kalit bilan), ochiq kalit DNS'da turadi. Bu email yo'lda o'zgartirilmaganini va haqiqiy manbadan kelganini isbotlaydi. SPF faqat **IP** ni tekshiradi (qaysi server ruxsat etilgan), DKIM esa **kontentning butunligini** kriptografik isbotlaydi. Ular birga ishlaydi — biri IP, biri imzo. DMARC ikkalasidan foydalanadi.

### ❓ DMARC nima qiladi?

**✅ Javob:** DMARC — SPF va DKIM ustidan **siyosat**: agar autentifikatsiya muvaffaqiyatsiz bo'lsa nima qilish (`none` — hech narsa, `quarantine` — Spam'ga, `reject` — rad et) va hisobot yuborish. Bosqichma-bosqich joriy qiling: `p=none` (kuzatuv) → `p=quarantine` → `p=reject`. 2024-yildan Gmail/Yahoo katta jo'natuvchilar uchun DMARC'ni majburiy qildi.

### ❓ Email nega Spam'ga tushadi?

**✅ Javob:** Asosiy sabablar: (1) SPF/DKIM/DMARC yo'q yoki noto'g'ri; (2) IP/domen reputatsiyasi past; (3) spam'ga o'xshash kontent (bosh harflar, "BEPUL!!!", shubhali linklar); (4) plain-text yo'q, faqat rasm; (5) yuqori bounce yoki spam report nisbati; (6) past engagement. Yechim: autentifikatsiyani to'g'ri sozlang, toza ro'yxatga foydali email yuboring, mail-tester.com bilan test qiling.

### ❓ Hard bounce va soft bounce farqi nima?

**✅ Javob:** **Hard bounce** — doimiy xato (manzil mavjud emas, domen yo'q). Bu manzilga qayta jo'natmang, darrov ro'yxatdan olib tashlang. **Soft bounce** — vaqtinchalik (mailbox to'la, server band). Bir necha marta qayta urinish mumkin. Yuqori bounce nisbati reputatsiyani buzadi, shuning uchun bounce'larni doim qayta ishlang va list hygiene qiling.

### ❓ IP warm-up nima va qachon kerak?

**✅ Javob:** IP warm-up — yangi IP'dan email hajmini asta-sekin oshirish (kun sayin ko'proq), toki provayderlar reputatsiya qursin. Yangi IP'dan darrov katta hajm = spam signali. Warm-up **dedicated IP** olganda kerak. **Shared IP** (default, ko'p loyiha uchun) da reputatsiya tayyor — warm-up shart emas.

### ❓ Dedicated IP va shared IP — qaysi biri?

**✅ Javob:** **Shared IP** — service boshqaradigan umumiy IP, reputatsiya tayyor, warm-up shart emas, kichik-o'rta hajm uchun ideal (default). **Dedicated IP** — faqat sizniki, to'liq nazorat, lekin qimmat, warm-up kerak va doimiy yuqori hajm talab qiladi. Oyiga million+ barqaror email bo'lmasa — shared IP ishlating.

### ❓ From address'da gmail.com ishlatsam bo'ladimi?

**✅ Javob:** Yo'q. `from` manzili **o'zingizning verified domeningizdan** bo'lishi shart (`hello@myapp.uz`). `gmail.com`/`yahoo.com` dan jo'natsangiz, ularning DMARC siyosatini buzasiz — email rad etiladi yoki Spam'ga tushadi. O'z domeningizni service'da verify qiling va shundan jo'nating.

### ❓ Email jo'natishni qanday test qilaman?

**✅ Javob:** **mail-tester.com** — email jo'natib, 10 balli spam score va aniq yaxshilash tavsiyalarini olasiz (SPF/DKIM/DMARC, kontent, blacklist). Turli provayderlarga (Gmail, Outlook) haqiqiy test yuboring, Inbox/Spam'ni tekshiring, "Show original" bilan autentifikatsiya `pass` ligini ko'ring. Development'da Mailtrap/Ethereal sandbox ishlating (haqiqatan yuborilmaydi).

### ❓ MUA, MTA, MDA nima?

**✅ Javob:** **MUA** (Mail User Agent) — email client (Gmail, Outlook), user ishlaydigan dastur. **MTA** (Mail Transfer Agent) — email'ni serverdan serverga uzatuvchi (Postfix, service serverlari), SMTP bilan gaplashadi. **MDA** (Mail Delivery Agent) — email'ni qabul qiluvchining mailbox'iga yakuniy joylaydigan komponent. Yo'l: MUA → MTA → (DNS MX) → MTA → MDA → MUA.

---

## Masalalar

> Yechimlar: [`solutions/email/01-email-fundamentals.md`](../solutions/email/01-email-fundamentals.md)

1. **SMTP transporter.** Nodemailer bilan bir email service (Resend yoki Mailtrap) uchun SMTP transporter yozing. Port 587, STARTTLS ishlating. Credential'larni environment o'zgaruvchilaridan oling (kodga yozmang). Ulanishni `transporter.verify()` bilan tekshiring.

2. **Welcome email funksiyasi.** `sendWelcomeEmail(to, name)` funksiyasi yozing: HTML va plain-text ikkala versiyani ham qo'shsin, `from` verified domendan bo'lsin, xatolarni to'g'ri boshqarsin (try/catch, log). Nodemailer yoki Resend SDK — o'zingiz tanlang.

3. **SPF record yozing.** `myapp.uz` domeni Resend va Amazon SES orqali email jo'natadi. To'g'ri **bitta** SPF TXT record'ini yozing. Nega ikkita alohida SPF record noto'g'ri ekanligini izohlang.

4. **DMARC bosqichma-bosqich.** Yangi domen uchun DMARC joriy qilish rejasini yozing: qaysi `p=` qiymatidan boshlaysiz, qanday kuzatasiz, qachon keyingi bosqichga o'tasiz. Har bosqich uchun DNS record misolini bering.

5. **Bounce handler.** Webhook orqali kelgan bounce hodisasini qayta ishlovchi funksiya yozing (pseudo-kod yoki Node.js). Hard va soft bounce'ni ajrating: hard bounce'da manzilni "yuborilmaydi" deb belgilang, soft bounce'da retry hisoblagichini oshiring (3 martadan keyin o'chiring).

6. **Spam score debug.** Sizga aytishdi: "welcome email'imiz Gmail'da Spam'ga tushyapti". mail-tester.com 4/10 ball berdi. Muammoni topish va tuzatishning bosqichma-bosqich diagnostika rejasini yozing (kamida 6 tekshiruv nuqtasi).

7. **Service tanlash.** Uchta stsenariy uchun email service tanlang va sababini yozing: (a) Next.js SaaS MVP, oyiga ~2,000 email; (b) e-commerce, oyiga ~5 million email, AWS'da; (c) fintech ilova, parol tiklash email'i har doim yetishi shart.

8. **Transactional queue.** Nega welcome email'ni to'g'ridan-to'g'ri HTTP so'rov ichida jo'natish yomon g'oya ekanini izohlang. Uni background queue (BullMQ yoki shunga o'xshash) orqali jo'natish arxitekturasini chizib bering (pseudo-kod yoki diagramma).

---

← [Email bo'limiga qaytish](./README.md)
