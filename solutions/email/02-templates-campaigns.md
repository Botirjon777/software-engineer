# Template va Kampaniyalar — Yechimlar

Bu yerda [`email/02-templates-campaigns.md`](../../email/02-templates-campaigns.md) dagi masalalarning yechimlari. Avval o'zingiz urinib ko'ring, keyin taqqoslang.

---

## 1-masala: Table-based welcome email

```html
<!DOCTYPE html>
<html lang="uz">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta name="color-scheme" content="light dark">
  <title>Xush kelibsiz</title>
</head>
<body style="margin:0; padding:0; background-color:#f4f4f4;">
  <!-- ko'rinmas preheader -->
  <div style="display:none; max-height:0; overflow:hidden; opacity:0;">
    Company oilasiga xush kelibsiz — boshlash uchun bir necha qadam qoldi.
  </div>

  <!-- tashqi wrapper: markazlashtirish uchun -->
  <table role="presentation" width="100%" cellpadding="0" cellspacing="0" border="0" style="background-color:#f4f4f4;">
    <tr>
      <td align="center" style="padding:24px 12px;">

        <!-- 600px content -->
        <table role="presentation" width="600" cellpadding="0" cellspacing="0" border="0" style="max-width:600px; width:100%; background-color:#ffffff; border-radius:8px;">
          <!-- logo -->
          <tr>
            <td style="padding:24px 24px 0 24px;">
              <img src="https://cdn.example.com/logo.png" alt="Company Logo"
                   width="120" height="40" style="display:block; border:0;">
            </td>
          </tr>
          <!-- sarlavha -->
          <tr>
            <td style="padding:16px 24px 0 24px; font-family:Arial, Helvetica, sans-serif; font-size:22px; font-weight:bold; color:#1a1a1a;">
              Xush kelibsiz, do'stim!
            </td>
          </tr>
          <!-- matn -->
          <tr>
            <td style="padding:12px 24px 0 24px; font-family:Arial, Helvetica, sans-serif; font-size:16px; line-height:24px; color:#555555;">
              Ro'yxatdan o'tganingiz uchun rahmat. Sizni ko'rganimizdan xursandmiz.
            </td>
          </tr>
          <tr>
            <td style="padding:12px 24px 0 24px; font-family:Arial, Helvetica, sans-serif; font-size:16px; line-height:24px; color:#555555;">
              Boshlash uchun quyidagi tugmani bosing va hisobingizni sozlang.
            </td>
          </tr>
          <!-- CTA tugma -->
          <tr>
            <td style="padding:24px;">
              <table role="presentation" cellpadding="0" cellspacing="0" border="0">
                <tr>
                  <td align="center" bgcolor="#2563eb" style="border-radius:6px;">
                    <a href="https://app.example.com/start"
                       style="display:inline-block; padding:12px 28px; font-family:Arial, sans-serif; font-size:16px; color:#ffffff; text-decoration:none; font-weight:bold;">
                      Boshlash
                    </a>
                  </td>
                </tr>
              </table>
            </td>
          </tr>
          <!-- footer -->
          <tr>
            <td style="padding:16px 24px 24px 24px; font-family:Arial, sans-serif; font-size:12px; line-height:18px; color:#999999; text-align:center; border-top:1px solid #eeeeee;">
              Company Inc, Amir Temur ko'chasi 1, Toshkent, O'zbekiston<br>
              <a href="{{unsubscribe_url}}" style="color:#999999;">Obunani bekor qilish</a>
            </td>
          </tr>
        </table>

      </td>
    </tr>
  </table>
</body>
</html>
```

**Muhim nuqtalar:** ikki qavatli table (tashqi = markazlashtirish, ichki = 600px content). Tugma ham `<table>` ichida (Outlook'da `<a>` padding buziladi, `bgcolor` bilan katak fon beriladi). Footer'da jismoniy manzil va unsubscribe majburiy.

---

## 2-masala: React Email komponenti

```tsx
import {
  Html, Head, Body, Container, Section,
  Heading, Text, Button, Hr,
} from "@react-email/components";

interface PasswordResetEmailProps {
  userName: string;
  resetUrl: string;
}

export default function PasswordResetEmail({ userName, resetUrl }: PasswordResetEmailProps) {
  return (
    <Html lang="uz">
      <Head />
      <Body style={{ backgroundColor: "#f4f4f4", fontFamily: "Arial, Helvetica, sans-serif", margin: 0 }}>
        <Container style={{ maxWidth: "600px", backgroundColor: "#ffffff", padding: "24px", borderRadius: "8px" }}>
          <Heading style={{ fontSize: "22px", color: "#1a1a1a" }}>
            Parolni tiklash
          </Heading>

          <Text style={{ fontSize: "16px", lineHeight: "24px", color: "#555555" }}>
            Salom, {userName}. Hisobingiz uchun parolni tiklash so'rovi keldi.
            Yangi parol o'rnatish uchun quyidagi tugmani bosing.
          </Text>

          <Section style={{ margin: "24px 0" }}>
            <Button
              href={resetUrl}
              style={{
                backgroundColor: "#2563eb", color: "#ffffff",
                padding: "12px 28px", borderRadius: "6px",
                fontSize: "16px", fontWeight: "bold", textDecoration: "none",
              }}
            >
              Parolni tiklash
            </Button>
          </Section>

          <Hr style={{ borderColor: "#eeeeee" }} />

          <Text style={{ fontSize: "13px", color: "#999999" }}>
            Agar siz bu so'rovni yubormagan bo'lsangiz, bu email'ga e'tibor bermang —
            parolingiz o'zgarishsiz qoladi. Bu havola 60 daqiqadan keyin ishlamaydi.
          </Text>
        </Container>
      </Body>
    </Html>
  );
}
```

**Eslatma:** Bu **transactional** email — unsubscribe/jismoniy manzil shart emas (foydalanuvchi o'zi so'ragan). Xavfsizlik ogohlantirishi ("siz so'ramagan bo'lsangiz") va havola muddati (expiry) yaxshi amaliyot.

---

## 3-masala: MJML → HTML

```html
<mjml>
  <mj-body background-color="#f4f4f4">
    <mj-section background-color="#ffffff" padding="24px">
      <!-- chap ustun: rasm -->
      <mj-column width="40%">
        <mj-image src="https://cdn.example.com/product.png" alt="Mahsulot" />
      </mj-column>
      <!-- o'ng ustun: matn + tugma -->
      <mj-column width="60%">
        <mj-text font-size="20px" font-weight="bold" color="#1a1a1a">
          Yangi mahsulot
        </mj-text>
        <mj-text font-size="15px" color="#555555">
          Sizga yoqadigan yangilikni ko'ring.
        </mj-text>
        <mj-button background-color="#2563eb" href="https://example.com/product" align="left">
          Ko'rish
        </mj-button>
      </mj-column>
    </mj-section>
  </mj-body>
</mjml>
```

Compile:

```bash
npm install -g mjml
mjml two-column.mjml -o two-column.html
```

**Nega stack bo'ladi:** MJML `<mj-column>`ni desktop'da yonma-yon (`inline-block`), lekin ekran torayganda (mobil) avtomatik `width:100%` qilib bir-birining ostiga (stack) tushiradi. Buni MJML generatsiya qiladigan media query hal qiladi — qo'lda yozish shart emas. Natijada mobilda rasm tepada, matn+tugma ostida bo'ladi.

---

## 4-masala: Merge tag va fallback (Liquid)

```liquid
Subject: {{ first_name | default: "Do'stim" }}, {{ city | default: "shahringiz" }} uchun taklif!
```

```html
<p>Salom, {{ first_name | default: "do'stim" }}!</p>

<p>Sizning tarifingiz:
  {% if plan == "premium" %}
    <strong>Premium</strong> — barcha imkoniyatlar ochiq. Rahmat!
  {% else %}
    <strong>{{ plan | default: "Basic" }}</strong> — Premium'ga o'tib ko'proq imkoniyat oling.
  {% endif %}
</p>

<p>{{ city | default: "Shahringiz" }}dagi mijozlarimizga maxsus chegirma.</p>
```

**Muhim:** Har bir `{{ }}`da `| default:` bor — data bo'sh bo'lsa "Salom, !" chiqmaydi. Subject'dagi merge tag ayniqsa xavfli (open rate'ga to'g'ridan-to'g'ri ta'sir), shuning uchun u yerda ham fallback bor. `{% if %}` bilan `plan`ga qarab boshqa xabar.

---

## 5-masala: Metrikani tahlil qiling

Berilgan: 10,000 yuborilgan, 9,600 yetkazilgan, 2,400 ochilgan, 240 bosgan, 15 unsubscribe, 12 spam complaint.

**Hisoblash:**

| Metrika | Formula | Natija |
|---------|---------|--------|
| Delivery rate | 9,600 / 10,000 | **96%** |
| Bounce rate | 400 / 10,000 | **4%** |
| Open rate | 2,400 / 9,600 | **25%** |
| CTR | 240 / 9,600 | **2.5%** |
| Unsubscribe rate | 15 / 9,600 | **~0.16%** |
| Spam complaint rate | 12 / 9,600 | **~0.125%** |

**Tahlil:**

- **Delivery 96%** — normadan biroz past. Bounce 4% — ideal <2%. List gigiyenasi kerak (hard bounce'larni o'chirish, double opt-in). Bu deliverability muammosining belgisi bo'lishi mumkin.
- **Open 25%** — o'rtacha, yomon emas (lekin MPP tufayli aniq emas).
- **CTR 2.5%** — normal oraliqda.
- **Spam complaint ~0.125%** — **eng muammoli metrika**. 0.1% chegaradan yuqori. Gmail/Yahoo bu chegaradan oshsa jazolaydi. Sabab: list rozilik asosida emasmi, content kutilmagan yoki unsubscribe qiyin bo'lishi mumkin. Buni darhol tuzatish kerak (rozilik, oson unsubscribe, tegishli content).

**Xulosa:** Bounce va spam complaint — ikkalasi list sifati muammosiga ishora qiladi. Faol, rozilik bergan list'ga o'tish kerak.

---

## 6-masala: Welcome drip sequence

SaaS mahsulot uchun 4 email'lik welcome sequence:

| # | Trigger | Maqsad | Subject | Asosiy CTA |
|---|---------|--------|---------|------------|
| 1 | Ro'yxatdan o'tdi (event, darhol) | Xush kelibsiz + tasdiqlash | "Xush kelibsiz! Bir qadam qoldi" | "Emailni tasdiqlash" |
| 2 | Kun 1 (vaqt) | Asosiy foyda, birinchi setup | "3 daqiqada birinchi natija" | "Setup'ni boshlash" |
| 3 | **Setup tugadi (event)** | Keyingi feature'ga yo'naltirish | "Zo'r! Endi buni sinab ko'ring" | "Feature X'ni ochish" |
| 4 | 3 kun faol emas (event) | Re-engagement | "Yordam kerakmi?" | "Qo'llanmani ko'rish" |

**Muhim:** 3- va 4-email **event-based** — vaqtga emas, foydalanuvchi harakatiga (setup tugatdi / faol emas) bog'langan. Bu drip'ni tegishli va samarali qiladi. Faqat vaqt bo'yicha ketadigan sequence hammaga bir xil bo'lib, kamroq ishlaydi.

---

## 7-masala: Qonuniy audit

Berilgan email: unsubscribe yo'q, jismoniy manzil yo'q, "From: noreply@", list sotib olingan.

| Buzilish | Qaysi qonun | Tuzatish |
|----------|-------------|----------|
| **Unsubscribe link yo'q** | CAN-SPAM + GDPR | Har bir marketing email footer'iga ishlaydigan, oson unsubscribe link. `List-Unsubscribe` header ham qo'shing (one-click). |
| **Jismoniy manzil yo'q** | CAN-SPAM | Footer'ga jo'natuvchining real pochta manzilini qo'shing. |
| **From: noreply@** | Sender identity (yaxshi amaliyot; CAN-SPAM ruhi) | Real, javob berish mumkin bo'lgan manzil (`hello@`, `team@`). "noreply" ishonchni va deliverability'ni pasaytiradi. |
| **List sotib olingan** | GDPR (rozilik yo'q) + CAN-SPAM ruhi | Sotib olingan list'ni **umuman ishlatmang**. Faqat opt-in (rozilik bergan) list yig'ing. Bu spam complaint va domen blokining asosiy sababi. |

**Xulosa:** To'rttala buzilish ham jiddiy. Eng xavflisi — sotib olingan list, chunki u ham qonuniy (GDPR), ham texnik (reputatsiya o'limi) muammo. Bu email'ni hozircha yubormaslik va noldan rozilik asosida list qurish kerak.

---

## 8-masala: ESP tanlash keysi

**(a) Faqat parol reset / chek yuboradigan SaaS:**
→ **Postmark** yoki **Resend**. Sabab: bular transactional email uchun optimallashtirilgan — tez yetkazish, yuqori deliverability, oddiy API. Marketing feature (kampaniya, segmentatsiya) shart emas.

**(b) Haftalik newsletter yuboradigan blogger:**
→ **ConvertKit (Kit)** yoki **Loops**. Sabab: content yaratuvchilar uchun mo'ljallangan, sodda editor, obunachi boshqaruvi, oddiy automation. Mailchimp ham mumkin, lekin blogger uchun ortiqcha murakkab bo'lishi mumkin.

**(c) Event-based onboarding + abandoned cart kerak e-commerce:**
→ **Customer.io** yoki **Klaviyo**. Sabab: kuchli event-based automation, segmentatsiya, e-commerce integratsiya. Abandoned cart va onboarding aynan harakatga (event) bog'langan drip talab qiladi — bu ularning kuchli tomoni.

---

## 9-masala: Responsive muammosi (media query bilan stack)

```html
<style>
  @media only screen and (max-width: 600px) {
    .container { width: 100% !important; }
    .stack-column {
      display: block !important;
      width: 100% !important;
      max-width: 100% !important;
    }
  }
</style>

<table role="presentation" width="600" class="container" cellpadding="0" cellspacing="0" border="0">
  <tr>
    <!-- har bir ustun class="stack-column" -->
    <td class="stack-column" width="50%" style="padding:12px; vertical-align:top;">
      <img src="https://cdn.example.com/a.png" alt="Chap" width="100%" style="display:block;">
    </td>
    <td class="stack-column" width="50%" style="padding:12px; vertical-align:top; font-family:Arial,sans-serif; font-size:15px; color:#555;">
      O'ng ustun matni.
    </td>
  </tr>
</table>
```

Desktop'da ikki `<td>` yonma-yon (50% + 50%). Mobilda (`max-width:600px`) `.stack-column` `display:block; width:100%` bo'lib, ustunlar bir-birining ostiga tushadi.

**Nega ba'zi clientda ishlamaydi:** Media query `<style>` blokida yashaydi, lekin ba'zi client (ayniqsa ba'zi Gmail holatlari, Outlook desktop) `<style>` yoki media query'ni qo'llab-quvvatlamaydi/olib tashlaydi. Bunday clientda email desktop (600px) ko'rinishida qoladi — mobilda gorizontal scroll bo'lishi mumkin. Shuning uchun murakkab responsive uchun **fluid/hybrid** texnika yoki to'g'ridan-to'g'ri **MJML/React Email** tavsiya etiladi — ular media query ishlamasa ham degradatsiyani boshqaradi.

---

## 10-masala: Spam'dan qutulish checklist

Gmail "Promotions"/"Spam"dan qochish uchun:

1. **Authentication** — SPF, DKIM, DMARC to'g'ri sozlangan bo'lsin (01-bo'lim). Bularsiz Gmail/Yahoo 2024-dan beri katta jo'natuvchilarni bloklaydi.

2. **List sifati** — faqat opt-in (rozilik bergan) real manzillar. Sotib olingan/scrape qilingan list — darhol spam. Hard bounce'larni o'chiring, double opt-in ishlating.

3. **Oson unsubscribe** — footer'da link + `List-Unsubscribe` header (one-click). Topolmagan odam "Spam" bosadi, bu eng zararli signal.

4. **Content balansi** — faqat bitta katta rasm (matnsiz) spam belgisi. Matn/rasm nisbatini muvozanatli qiling. Spam trigger so'zlardan qoching: "BEPUL!!!", ko'p "!!!", CAPS, "100% kafolat".

5. **Barqaror hajm** — birdan 10 kishidan 50,000'ga sakramang. Domain warm-up qiling (asta-sekin oshiring). Keskin o'zgarish reputatsiyani buzadi.

6. **Engagement** — faol foydalanuvchilarga yuboring. Ochmaydigan, bosmaydiganlarni segmentga ajratib, kamroq yuboring yoki tozalang. Gmail engagement'ni signal sifatida ishlatadi.

7. **Sender reputatsiya** — Google Postmaster Tools bilan domen reputatsiyasini kuzating, spam complaint rate'ni <0.1% ushlang.

8. **(bonus) Preheader va tashqi ko'rinish** — professional, spam'ga o'xshamagan dizayn; real `from` manzil (`noreply` emas), aniq subject.

---

← [Email bo'limiga qaytish](../../email/README.md)
