# Email Asoslari va Jo'natish — Yechimlar

Bu fayl [`email/01-email-fundamentals.md`](../../email/01-email-fundamentals.md) dagi masalalar yechimlari. Yechimlar bitta to'g'ri javob emas — yondashuv va mulohazani ko'rsatadi. O'zingiznikini avval yozib, keyin solishtiring.

---

## 1. SMTP transporter

Nodemailer bilan port 587 (STARTTLS), credential'lar environment'dan:

```js
import nodemailer from "nodemailer";

const transporter = nodemailer.createTransport({
  host: process.env.SMTP_HOST, // masalan "smtp.resend.com"
  port: Number(process.env.SMTP_PORT) || 587,
  secure: false, // 587 uchun false — STARTTLS orqali shifrlanadi
  auth: {
    user: process.env.SMTP_USER, // Resend uchun "resend"
    pass: process.env.SMTP_PASS, // API key / parol
  },
});

// Ulanishni ishga tushirishda tekshirish
try {
  await transporter.verify();
  console.log("SMTP ulanish tayyor");
} catch (err) {
  console.error("SMTP ulanish xatosi:", err.message);
  process.exit(1); // konfiguratsiya buzuq bo'lsa ishga tushmasin
}

export default transporter;
```

**Izoh:** `secure: false` + port 587 = STARTTLS (ulanish oddiy boshlanib, keyin TLS'ga o'tadi). Port 465 uchun `secure: true` bo'lardi. Credential'lar hech qachon kodda emas — `.env` faylda va `.gitignore`da. `verify()` ishga tushishda muammoni erta aniqlaydi.

---

## 2. Welcome email funksiyasi

Resend SDK versiyasi (HTML + plain-text, xato boshqaruvi):

```ts
import { Resend } from "resend";

const resend = new Resend(process.env.RESEND_API_KEY);
const FROM = "MyApp <welcome@myapp.uz>"; // verified domen

export async function sendWelcomeEmail(to: string, name: string) {
  const html = `
    <div style="font-family: sans-serif; max-width: 560px;">
      <h1>Xush kelibsiz, ${escapeHtml(name)}!</h1>
      <p>MyApp'ga ro'yxatdan o'tganingiz uchun rahmat.</p>
      <a href="https://myapp.uz/start">Boshlash</a>
    </div>`;

  const text = `Xush kelibsiz, ${name}!\n\nMyApp'ga ro'yxatdan o'tganingiz uchun rahmat.\nBoshlash: https://myapp.uz/start`;

  try {
    const { data, error } = await resend.emails.send({
      from: FROM,
      to,
      subject: "MyApp'ga xush kelibsiz!",
      html,
      text,
    });
    if (error) throw new Error(error.message);
    console.log(`Welcome email yuborildi -> ${to}, id=${data?.id}`);
    return data;
  } catch (err) {
    console.error(`Welcome email xatosi -> ${to}:`, (err as Error).message);
    throw err; // chaqiruvchi qaror qilsin (retry/queue)
  }
}

function escapeHtml(s: string) {
  return s.replace(/&/g, "&amp;").replace(/</g, "&lt;").replace(/>/g, "&gt;");
}
```

**Izoh:** Ikkala versiya (HTML + plain-text) spam filtri va HTML ko'rmaydigan client'lar uchun. `from` verified domendan. User kiritgan `name` HTML'ga qo'yilganda `escapeHtml` bilan tozalanadi (XSS/injection oldini olish). Xato yuqoriga uzatiladi — funksiya o'zi retry qilmaydi.

---

## 3. SPF record

`myapp.uz` Resend va SES orqali jo'natadi — **bitta** birlashtirilgan record:

```text
Type:  TXT
Name:  @  (yoki myapp.uz)
Value: v=spf1 include:_spf.resend.com include:amazonses.com -all
```

**Nega ikkita alohida record noto'g'ri:** SPF spetsifikatsiyasi bo'yicha bitta domenga **faqat bitta** `v=spf1` bilan boshlanadigan TXT record bo'lishi mumkin. Agar ikkita bo'lsa:

```text
v=spf1 include:_spf.resend.com -all      ❌
v=spf1 include:amazonses.com -all        ❌
```

qabul qiluvchi server `PermError` (permanent error) qaytaradi va SPF **butunlay ishlamaydi** — email autentifikatsiyasi buziladi. To'g'ri yo'l — barcha service'larni bitta record ichida `include` qilish. `-all` (hard fail) ro'yxatda yo'q hamma narsani rad etadi.

**Qo'shimcha:** SPF'da 10 tadan ortiq DNS lookup bo'lmasligi kerak (`include`lar sanaladi) — aks holda ham `PermError`.

---

## 4. DMARC bosqichma-bosqich

**Bosqich 1 — Kuzatuv (`p=none`), 2-4 hafta:**

```text
Name:  _dmarc.myapp.uz
Value: v=DMARC1; p=none; rua=mailto:dmarc@myapp.uz; fo=1
```

Hech narsani bloklamaydi. Faqat agregat hisobot yig'adi — kim sizning domeningiz nomidan jo'natyapti, SPF/DKIM `pass` bo'lyaptimi. Barcha legitim manbalar (Resend, SES, ehtimol boshqa tool'lar) to'g'ri autentifikatsiyadan o'tayotganini tekshirasiz.

**Bosqich 2 — Qisman himoya (`p=quarantine`), 2-4 hafta:**

```text
Name:  _dmarc.myapp.uz
Value: v=DMARC1; p=quarantine; pct=25; rua=mailto:dmarc@myapp.uz
```

`pct=25` — trafikning faqat 25%'iga qo'llash (asta-sekin). Muvaffaqiyatsiz email Spam'ga tushadi. Hisobotni kuzatib, o'z email'laringiz noto'g'ri tushmayotganiga ishonch hosil qilib, `pct`'ni 50 → 100 ga oshirasiz.

**Bosqich 3 — To'liq himoya (`p=reject`):**

```text
Name:  _dmarc.myapp.uz
Value: v=DMARC1; p=reject; rua=mailto:dmarc@myapp.uz; adkim=s; aspf=s
```

Autentifikatsiyadan o'tmagan email butunlay rad etiladi — soxta email'lar (spoofing) bloklanadi.

**Izoh:** Darrov `p=reject` qo'yish xato — noto'g'ri sozlangan legitim manba (masalan yangi qo'shilgan tool) email'lari bloklanib, real muammo tug'diradi. `none → quarantine → reject` va `pct` bilan bosqichma-bosqich o'tish — xavfsiz yo'l.

---

## 5. Bounce handler

Service webhook (Resend/SES) POST qiladi, uni qayta ishlaymiz:

```js
// POST /webhooks/email  (webhook imzosini avval tekshiring!)
async function handleBounceWebhook(event) {
  const { type, email } = normalize(event); // service formatini birlashtirish

  if (type === "hard_bounce" || type === "complaint") {
    // Doimiy xato yoki spam report -> boshqa yubormaymiz
    await db.suppression.upsert({
      email,
      reason: type,
      suppressedAt: new Date(),
    });
    console.log(`Suppress qilindi (${type}): ${email}`);
    return;
  }

  if (type === "soft_bounce") {
    const rec = await db.softBounce.increment(email); // retry hisoblagich
    if (rec.count >= 3) {
      // 3 marta ketma-ket soft bounce -> hard sifatida qaraymiz
      await db.suppression.upsert({ email, reason: "soft_bounce_max" });
      console.log(`Soft bounce limiti -> suppress: ${email}`);
    } else {
      console.log(`Soft bounce #${rec.count}: ${email} (keyinroq retry)`);
    }
    return;
  }
}
```

**Izoh:** Hard bounce va complaint — darrov **suppression list**ga (qora ro'yxat) qo'shiladi, boshqa hech qachon yubormaymiz. Har email jo'natishdan oldin bu ro'yxatni tekshirish shart. Soft bounce — vaqtinchalik, retry hisoblagichi bilan; 3 martadan keyin suppress. Webhook imzosini (signature) tekshirish muhim — aks holda soxta bounce yuborish mumkin.

---

## 6. Spam score debug (4/10)

Bosqichma-bosqich diagnostika:

1. **Autentifikatsiya tekshirish.** Gmail'da "Show original" ochib SPF/DKIM/DMARC `PASS` ekanini ko'ring. mail-tester natijasida qaysi biri fail bo'lgani ko'rsatiladi — bu ko'pincha eng katta muammo.
2. **DNS record'lar.** SPF (bitta record, to'g'ri `include`), DKIM (selector to'g'ri, service verify qilgan), DMARC (kamida `p=none`) mavjudligini tekshiring. `dig TXT myapp.uz` va service dashboard'da "verified" holatini ko'ring.
3. **From domen.** `from` verified domendanmi? `gmail.com`/generik domen emasmi? Domen verify to'liq o'tganmi?
4. **Kontent.** Spam so'zlar ("BEPUL", "HOZIROQ"), ortiqcha bosh harf/undov belgi, faqat bitta katta rasm bormi? HTML **va** plain-text ikkalasi bormi (faqat HTML — minus ball)?
5. **Link'lar.** Shubhali/qisqartirilgan URL, link matni bilan haqiqiy manzil mos kelmasligi, HTTP (HTTPS emas) linklar bormi?
6. **Reputatsiya/blacklist.** mail-tester IP/domen blacklist'da emasligini ko'rsatadi. Yangi domen bo'lsa reputatsiya hali qurilmagan — hajmni asta oshirish kerak.
7. **List-Unsubscribe header.** Marketing bo'lsa `List-Unsubscribe` (one-click) bormi — 2024+ da Gmail buni talab qiladi.

**Odatiy sabab:** DKIM verify o'tmagan yoki DMARC yo'q + faqat HTML (plain-text yo'q). Bularni tuzatgach ball odatda 9-10 ga ko'tariladi.

---

## 7. Service tanlash

**(a) Next.js SaaS MVP, ~2,000 email/oy → Resend.**
Kichik hajm, React/Next stack. Resend'ning DX'i zo'r, React Email bilan native ishlaydi, bepul tier (~3k/oy) MVP'ga yetadi. Tez boshlash muhim. Keyinroq o'sganda migratsiya oson.

**(b) E-commerce, ~5M email/oy, AWS'da → Amazon SES.**
Katta hajmda narx hal qiluvchi — SES ~$0.10/1000 email, ya'ni 5M ≈ $500/oy (raqobatchilardan sezilarli arzon). AWS'da o'tirgani uchun IAM/SDK integratsiya tabiiy. DX yomonroq, lekin bu hajmda narx afzalligi qoplaydi. Dedicated IP + warm-up mantiqiy.

**(c) Fintech, parol tiklash har doim yetishi shart → Postmark.**
Deliverability kritik. Postmark faqat transactional'ga ixtisoslashgan, marketing spam qabul qilmaydi — IP'lari toza, reputatsiyasi eng yaxshi, yetkazish tez. Bunday use-case uchun klassik tanlov. (SES ham ishlaydi lekin Postmark'ning transactional fokusi bu yerda ustun.)

---

## 8. Transactional queue

**Nega HTTP so'rov ichida jo'natish yomon:**

- Email jo'natish **sekin** (100ms-2s) — user javob kutib qoladi, sahifa muzlaydi.
- Email service **vaqtincha ishlamasligi** mumkin — bu holda user'ning ro'yxatdan o'tishi **butunlay** fail bo'ladi, garchi asosiy amal (user yaratildi) bajarilgan bo'lsa ham.
- **Retry** qilib bo'lmaydi — xato bo'lsa email yo'qoladi.

**Yechim — background queue:**

```text
┌─────────────┐   1. User ro'yxatdan o'tadi
│  HTTP        │   2. DB'ga user yoziladi
│  handler     │   3. Queue'ga job qo'shiladi (tez, ~ms)
│              │   4. Javob DARROV qaytariladi (201 Created)
└──────┬───────┘
       │ enqueue
       ▼
┌─────────────┐
│  Queue       │   (BullMQ / Redis)
│  (jobs)      │
└──────┬───────┘
       │ worker oladi
       ▼
┌─────────────┐   5. Worker email jo'natadi
│  Worker      │   6. Xato bo'lsa -> avtomatik retry (exponential backoff)
│  (email)     │   7. N marta fail -> dead-letter queue + alert
└─────────────┘
```

```js
// HTTP handler — tez, bloklanmaydi
await db.user.create({ email, name });
await emailQueue.add("welcome", { to: email, name });
return res.status(201).json({ ok: true }); // darrov javob

// Worker — alohida process
emailQueue.process("welcome", async (job) => {
  await sendWelcomeEmail(job.data.to, job.data.name);
  // xato bo'lsa throw -> BullMQ avtomatik retry qiladi
});
```

**Izoh:** User so'rovi email'ga bog'liq emas — HTTP javob darrov qaytadi. Email fon'da jo'natiladi, xato bo'lsa avtomatik retry (exponential backoff), N marta fail bo'lsa dead-letter'ga o'tib alert beriladi. Bu ishonchli va tez.

---

← [Email bo'limiga qaytish](../../email/README.md)
