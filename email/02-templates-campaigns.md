# Template va Kampaniyalar

Email jo'natishni o'rgandingiz — endi eng qiyin ikki qism: chiroyli, hamma joyda to'g'ri ko'rinadigan **template** yasash va odamlarga to'g'ri, qonuniy, samarali **kampaniya** yuborish. Bu bo'lim shu haqda.

HTML email — bu web dasturlashning eng g'alati, eng eski, eng "sekin o'zgargan" qismi. 2026-yilda ham Gmail, Outlook, Apple Mail bir xil HTML'ni turlicha render qiladi, va sizga 20 yil oldingi `<table>`-based layout yozishga to'g'ri keladi. Kampaniya tomonida esa texnika emas, balki qonun (CAN-SPAM, GDPR), metrika va odob-axloq muhim.

**💡 Tushuncha:** Email dizayni — bu "chiroyli qilish" emas, balki "har xil buzuq brauzerda ham buzilmaydigan qilish". Bu yerda mahorat — eski texnologiyani qasddan ishlatishda.

## Mundarija

- [Nega HTML email qiyin](#nega-html-email-qiyin)
- [Email HTML qoidalari](#email-html-qoidalari)
- [MJML](#mjml)
- [React Email](#react-email)
- [Boshqa vositalar: Maizzle, Handlebars, Liquid](#boshqa-vositalar)
- [Personalizatsiya va merge tag](#personalizatsiya-va-merge-tag)
- [Responsive email](#responsive-email)
- [Testing va preview](#testing-va-preview)
- [Kampaniya turlari: transactional, marketing, drip](#kampaniya-turlari)
- [Kampaniya jarayoni: list, segmentatsiya, subject](#kampaniya-jarayoni)
- [A/B testing](#ab-testing)
- [Metrikalar](#metrikalar)
- [Qonuniy talablar: CAN-SPAM, GDPR](#qonuniy-talablar)
- [Automation / drip misollari](#automation--drip-misollari)
- [Marketing platformalar taqqoslash](#marketing-platformalar-taqqoslash)
- [❓ Savol-javoblar](#savol-javoblar)
- [Masalalar](#masalalar)

---

## Nega HTML email qiyin

Web sahifa yozganda siz bitta narsaga tayanasiz: brauzerlar (Chrome, Firefox, Safari) modern CSS'ni deyarli bir xil qo'llab-quvvatlaydi. Email'da bunday narsa **yo'q**.

Email HTML'ni email client render qiladi, va ular juda ko'p va juda har xil:

- **Gmail** (web) — o'zining CSS filtri bor, `<style>` blokini qisman qoldiradi, lekin `<head>`'dan olib tashlaydigan holatlar bor. External `<link>` CSS umuman ishlamaydi.
- **Apple Mail** (macOS/iOS) — eng yaxshisi, WebKit ishlatadi, modern CSS'ni ko'p qo'llab-quvvatlaydi.
- **Outlook (desktop, Windows)** — **eng yomoni**. 2007-yildan beri HTML'ni **Microsoft Word** engine bilan render qiladi (`mso` deb ataladi). `float`, `position`, `background-image`, `flexbox`, `grid` — hech biri ishlamaydi. `padding`/`margin` ba'zi elementlarda buzuq.
- **Outlook.com / Outlook (yangi, web)** — desktop'dan farqli, boshqa engine, lekin baribir cheklangan.
- Yana: Yahoo Mail, ProtonMail, Samsung Mail, mobil ilovalar...

**💡 Tushuncha:** "Outlook eng yomoni" — bu email dasturchilar orasida mem emas, real muammo. Kampaniyaning ~5-30% ochilishi Outlook'da bo'lishi mumkin (B2B'da ayniqsa ko'p), shuning uchun uni tashlab bo'lmaydi.

Natijada: siz 2003-yildagidek yozasiz — `<table>` bilan layout, hamma CSS inline, modern CSS'siz.

**⚠️ Ehtiyot bo'l:** "Menda Chrome'da chiroyli ko'rindi" degani — hech narsani anglatmaydi. Email'ni ko'p clientda test qilmaguningizcha, u ishlaydi deb ayta olmaysiz.

---

## Email HTML qoidalari

Xavfsiz, hamma joyda ishlaydigan email uchun asosiy qoidalar:

**1. Table-based layout (`<div>` emas).** Butun struktura `<table>` ustiga quriladi. Outlook `<div>` bilan layoutni buzadi.

```html
<table role="presentation" width="100%" cellpadding="0" cellspacing="0" border="0">
  <tr>
    <td align="center">
      <!-- markaziy content shu yerda, 600px -->
      <table role="presentation" width="600" cellpadding="0" cellspacing="0" border="0">
        <tr>
          <td style="padding: 24px; font-family: Arial, sans-serif; font-size: 16px; color: #1a1a1a;">
            Salom, bu email matni.
          </td>
        </tr>
      </table>
    </td>
  </tr>
</table>
```

`role="presentation"` — screen reader'ga "bu jadval ma'lumot uchun emas, layout uchun" deb aytadi (accessibility).

**2. Inline CSS.** Har bir element `style="..."` bilan bezatiladi. `<style>` blok qisman ishlaydi (Gmail ba'zan olib tashlaydi), external CSS umuman ishlamaydi.

```html
<!-- Yaxshi: inline -->
<td style="color: #333; font-size: 16px;">Matn</td>

<!-- Ishonchsiz: <style> blok Gmail'da olib tashlanishi mumkin -->
```

**3. 600px kenglik.** An'anaviy standart. Desktop clientlarda content qismi ~600px bo'ladi, undan keng qilinsa ba'zi joyda kesiladi.

**4. Web-safe font.** `Arial`, `Helvetica`, `Georgia`, `Times New Roman`, `Verdana`, `Tahoma`. Custom font (Google Fonts) faqat ba'zi clientda ishlaydi — shuning uchun `font-family` da fallback bering:

```html
<td style="font-family: 'Inter', Arial, Helvetica, sans-serif;">
```

Agar `Inter` yuklanmasa, `Arial`ga tushadi.

**5. `<img>` — absolute URL, `alt`, kenglik.** Rasm CDN'da bo'lishi kerak, absolute `https://` URL bilan. Ko'p client rasmni default o'chirib qo'yadi, shuning uchun `alt` matn muhim.

```html
<img src="https://cdn.example.com/logo.png" alt="Company Logo"
     width="120" height="40" style="display: block; border: 0;">
```

`display: block` — rasm ostidagi bo'sh joyni (whitespace) yo'qotadi.

**6. Dark mode.** Ko'p client dark mode'da ranglarni avtomatik o'zgartiradi (invert). `<meta>` va `color-scheme` bilan boshqarish mumkin, lekin to'liq nazorat yo'q. Logotipni PNG (shaffof fon) qilib, oq/qora ikki holatda ham ko'rinadigan qiling.

```html
<meta name="color-scheme" content="light dark">
<meta name="supported-color-schemes" content="light dark">
```

**⚠️ Ehtiyot bo'l:** Rasm ustiga matn qo'ymang ("hero image with text"). Rasm o'chirilsa yoki yuklanmasa, matn ko'rinmaydi. Matnni har doim HTML matn sifatida yozing.

---

## MJML

**MJML** (Mailjet Markup Language) — responsive email uchun maxsus markup tili. Siz oddiy, o'qiladigan MJML yozasiz, u esa buzuq, table-based HTML'ga **compile** qilinadi.

Ya'ni: qo'lda `<table>` yozish o'rniga, MJML'ni yozib qo'yasiz, u sizga barcha Outlook hack'larini avtomatik generatsiya qiladi.

```html
<mjml>
  <mj-body background-color="#f4f4f4">
    <mj-section background-color="#ffffff" padding="24px">
      <mj-column>
        <mj-image width="120px" src="https://cdn.example.com/logo.png" alt="Logo" />
        <mj-text font-size="20px" font-weight="bold" color="#1a1a1a">
          Xush kelibsiz!
        </mj-text>
        <mj-text font-size="16px" color="#555555">
          Ro'yxatdan o'tganingiz uchun rahmat.
        </mj-text>
        <mj-button background-color="#2563eb" href="https://example.com/start">
          Boshlash
        </mj-button>
      </mj-column>
    </mj-section>
  </mj-body>
</mjml>
```

Compile qilish:

```bash
npm install -g mjml
mjml template.mjml -o output.html
```

Yoki dasturiy:

```js
import mjml2html from "mjml";

const { html, errors } = mjml2html(`
  <mjml><mj-body><mj-section><mj-column>
    <mj-text>Salom!</mj-text>
  </mj-column></mj-section></mj-body></mjml>
`);

console.log(html); // to'liq email-safe HTML
```

**💡 Tushuncha:** MJML'ning kuchi — `<mj-column>` avtomatik responsive bo'ladi. Mobil ekranda ustunlar bir-birining ostiga tushadi (stack) — buni qo'lda yozish og'riqli.

---

## React Email

**React Email** — 2026-yilda eng zamonaviy va tavsiya etiladigan yondashuv. Email'ni oddiy React komponent sifatida yozasiz, u email-safe HTML'ga render bo'ladi. TypeScript, komponent qayta ishlatish, propslar — barchasi bor.

```tsx
import {
  Html, Head, Body, Container, Section,
  Text, Button, Img, Hr,
} from "@react-email/components";

interface WelcomeEmailProps {
  name: string;
  loginUrl: string;
}

export default function WelcomeEmail({ name, loginUrl }: WelcomeEmailProps) {
  return (
    <Html lang="uz">
      <Head />
      <Body style={{ backgroundColor: "#f4f4f4", fontFamily: "Arial, sans-serif" }}>
        <Container style={{ maxWidth: "600px", backgroundColor: "#ffffff", padding: "24px" }}>
          <Img
            src="https://cdn.example.com/logo.png"
            alt="Logo" width="120" height="40"
          />
          <Section>
            <Text style={{ fontSize: "20px", fontWeight: "bold", color: "#1a1a1a" }}>
              Salom, {name}!
            </Text>
            <Text style={{ fontSize: "16px", color: "#555555" }}>
              Ro'yxatdan o'tganingiz uchun rahmat. Boshlash uchun tugmani bosing.
            </Text>
            <Button
              href={loginUrl}
              style={{
                backgroundColor: "#2563eb", color: "#ffffff",
                padding: "12px 24px", borderRadius: "6px", textDecoration: "none",
              }}
            >
              Hisobga kirish
            </Button>
          </Section>
          <Hr />
          <Text style={{ fontSize: "12px", color: "#999999" }}>
            Company Inc, Toshkent, O'zbekiston
          </Text>
        </Container>
      </Body>
    </Html>
  );
}
```

HTML'ga render qilish va jo'natish (masalan Resend bilan):

```tsx
import { render } from "@react-email/render";
import { Resend } from "resend";
import WelcomeEmail from "./emails/welcome";

const resend = new Resend(process.env.RESEND_API_KEY);

const html = await render(<WelcomeEmail name="Ramziddin" loginUrl="https://app.example.com/login" />);

await resend.emails.send({
  from: "Company <hello@example.com>",
  to: "user@example.com",
  subject: "Xush kelibsiz!",
  html,
});
```

React Email'ning yana bir plyusi — lokal preview server:

```bash
npm install react-email @react-email/components
npx react-email dev   # localhost:3000 da email'larni ko'rasiz
```

**💡 Tushuncha:** React Email `@react-email/components` ichida `<Button>`, `<Container>` kabi komponentlar allaqachon Outlook-safe HTML'ga o'giriladi. Siz React yozasiz, u eski `<table>` hack'larini o'zi qiladi. Resend jamoasi tomonidan ochiq manba sifatida ishlab chiqilgan.

---

## Boshqa vositalar

**Maizzle** — Tailwind CSS bilan email yozish. Utility-class'lar (`class="text-lg font-bold"`) yoziladi, build vaqtida CSS inline qilinadi (juice/purge). Tailwind biladigan jamoalar uchun qulay.

```html
<table class="w-full">
  <tr>
    <td class="px-6 py-4 text-base text-gray-800">
      Maizzle bilan Tailwind email
    </td>
  </tr>
</table>
```

**Handlebars / Liquid** — bu template *tili* (framework emas), dinamik content va shartlar uchun. Ko'p ESP (Mailchimp, Shopify) Liquid ishlatadi.

```html
<!-- Handlebars -->
<p>Salom, {{name}}!</p>
{{#if isPremium}}
  <p>Premium a'zoligingiz uchun rahmat.</p>
{{else}}
  <p>Premium'ga o'tishni ko'rib chiqing.</p>
{{/if}}

{{#each orders}}
  <li>{{this.product}} — {{this.price}} so'm</li>
{{/each}}
```

**Taqqoslash qisqacha:**

| Vosita | Yondashuv | Kimga |
|--------|-----------|-------|
| Qo'lda HTML | `<table>` + inline CSS | Faqat kichik, oddiy email |
| MJML | Maxsus markup → HTML | Dizaynerlar, tez responsive |
| React Email | React komponent | Modern JS/TS jamoalar (tavsiya) |
| Maizzle | Tailwind | Tailwind biladiganlar |
| Handlebars/Liquid | Dinamik data | Har qanday vosita ustida |

---

## Personalizatsiya va merge tag

**Merge tag** — email matni ichidagi placeholder, jo'natishda real qiymatga almashadi. Eng oddiysi `{{name}}`.

```
Subject: {{name}}, sizga maxsus taklif bor!
Body: Salom {{first_name}}, oxirgi tashrifingiz {{last_seen}} edi.
```

ESP'da har bir kontakt uchun `first_name`, `city`, `plan` kabi maydonlar (merge fields) saqlanadi. Jo'natishda ular to'ldiriladi.

**Dinamik content** — nafaqat so'z, balki butun blok segmentga qarab o'zgaradi:

```html
{{#if country == "UZ"}}
  <p>To'lov: Click, Payme orqali</p>
{{else}}
  <p>Payment via Stripe</p>
{{/if}}
```

**⚠️ Ehtiyot bo'l:** Har doim **fallback** bering. Agar `first_name` bo'sh bo'lsa, "Salom !" chiqadi — bu yomon. `{{first_name | default: "do'stim"}}` kabi default qo'ying. Va merge tag'ni subject line'da ishlatishdan oldin data toza ekaniga ishonch hosil qiling.

---

## Responsive email

Email 600px desktop uchun quriladi, lekin ochilishlarning yarmidan ko'pi **mobil**da. Shuning uchun responsive kerak.

Ikki yondashuv:

**1. Media query** (`<style>` blokida — Apple Mail, ba'zi clientlarda ishlaydi, Gmail'da cheklangan):

```html
<style>
  @media only screen and (max-width: 600px) {
    .container { width: 100% !important; }
    .stack { display: block !important; width: 100% !important; }
    .mobile-hide { display: none !important; }
  }
</style>
```

**2. Fluid / hybrid** (media query'siz ham ishlaydi) — `width: 100%` va `max-width` bilan, ustunlarni `align`/`inline-block` bilan stack qiladigan texnika. Media query ishlamaydigan clientlarda ham degradatsiya normal bo'ladi.

**💡 Tushuncha:** MJML va React Email responsive'ni o'zi hal qiladi — shuning uchun ular tavsiya etiladi. Qo'lda media query yozish faqat maxsus holatlarda kerak bo'ladi.

---

## Testing va preview

Email yasab bo'ldingiz — endi uni **hamma clientda** ko'rish kerak. Bu deploydan oldingi eng muhim qadam.

**1. Litmus / Email on Acid** — ushbu servislar sizning email'ingizni 90+ real client (Outlook 2016, Gmail, iPhone, Yahoo...) da screenshot qilib ko'rsatadi. Pullik, lekin professional kampaniya uchun deyarli majburiy.

**2. Test jo'natish (seed test).** Kampaniyani o'zingizga va jamoaga (Gmail, Outlook, iPhone hisoblariga) yuborib, real ochib ko'ring. Spam papkaga tushdimi ham tekshiring.

**3. Preview text / preheader.** Inbox'da subject'dan keyin ko'rinadigan qisqa matn. Uni maxsus qo'ymasangiz, email boshidagi tasodifiy matn ("View in browser...") chiqadi — bu open rate'ni tushiradi.

```html
<!-- Body eng boshida, ko'rinmas preheader -->
<div style="display:none; max-height:0; overflow:hidden; opacity:0;">
  50% chegirma faqat bugun — ichiga kiring va ko'ring
</div>
```

**⚠️ Ehtiyot bo'l:** Preheader'dan keyin bo'sh joy (`&nbsp;&zwnj;` takrorlash) qo'ying, aks holda inbox preview'ga email boshidagi keraksiz matn qo'shilib ketadi.

---

## Kampaniya turlari

Email kampaniyalari uch xil, va ular **texnik va qonuniy jihatdan** farq qiladi:

| Tur | Nima | Trigger | Misol |
|-----|------|---------|-------|
| **Transactional** | Bir foydalanuvchi harakatiga javob | Event (signup, purchase) | Parol reset, buyurtma cheki, OTP |
| **Marketing / broadcast** | Ko'pchilikka bir vaqtda | Qo'lda / rejalashtirilgan | Newsletter, aksiya e'loni |
| **Drip / automation** | Vaqt/harakat bo'yicha ketma-ket | Sequence trigger | Welcome seriyasi, onboarding |

**Muhim farq:**

- **Transactional** — rozilik (consent) shart emas, chunki foydalanuvchi o'zi kutmoqda. Lekin ichiga marketing tiqishtirmang (buzsa CAN-SPAM buziladi). Odatda alohida ESP (Postmark, Resend) orqali, yuqori deliverability bilan.
- **Marketing** — **rozilik majburiy**, unsubscribe link majburiy. Marketing ESP (Mailchimp, Brevo) orqali.
- **Drip** — marketing qoidalariga bo'ysunadi (rozilik + unsubscribe), lekin avtomatik, sequence sifatida ketadi.

**⚠️ Ehtiyot bo'l:** Transactional va marketing'ni bir domendan yubormang. Marketing spam complaint reputatsiyani buzsa, transactional email'laringiz (parol reset!) ham spam'ga tusha boshlaydi. Ko'pincha `send.domain.com` (marketing) va `mail.domain.com` (transactional) ajratiladi.

---

## Kampaniya jarayoni

Yaxshi kampaniya texnikadan emas, **jarayondan** boshlanadi:

**1. List building (ro'yxat yig'ish).**

- **Opt-in** — foydalanuvchi o'zi obuna bo'ladi (form to'ldiradi). Sotib olingan yoki scrape qilingan list — **taqiqlangan** (spam, jarima, reputatsiya o'limi).
- **Double opt-in** — obunadan keyin tasdiqlash email'i yuboriladi, foydalanuvchi link bosib tasdiqlaydi. Bu list sifatini oshiradi (real, faol manzillar) va bot/typo'ni filtrlaydi. GDPR uchun ham tavsiya etiladi.

**2. Segmentatsiya.** Butun ro'yxatga bir xil email yuborish — samarasiz. Auditoriyani bo'ling:

- Xatti-harakat (faol / faol emas, sotib olganlar)
- Demografiya (davlat, til)
- Lifecycle (yangi / eski mijoz)

Segmentatsiya open va click rate'ni sezilarli oshiradi.

**3. Personalizatsiya** (yuqorida ko'rdik) — merge tag, dinamik content.

**4. Subject line.** Bu **eng muhim** qism — open rate to'g'ridan-to'g'ri unga bog'liq. Odam faqat subject va sender'ni ko'rib ochish/ochmaslikni hal qiladi.

- Qisqa (~40-50 belgi, mobilda kesilmasin)
- Aniq, clickbait emas (ishonch)
- Emoji — ehtiyotkorlik bilan (ba'zan spam filtri yoqtirmaydi)
- Shoshilinchlik/qiziqish, lekin yolg'onsiz

**5. Send time.** Qachon yuborish ochilishga ta'sir qiladi. Universal qoida yo'q — o'z auditoriyangizni A/B test qilib toping. Ko'pincha ish kuni ertalab yaxshi, lekin bu segmentga bog'liq.

---

## A/B testing

**A/B test** — ikki variant yuborib, qaysi biri yaxshi ishlashini o'lchash. ESP odatda buni avtomatlashtiradi:

- Listning kichik qismiga (masalan 20%) ikki variantni yuboradi (10% A, 10% B)
- G'olibni aniqlaydi (yuqori open yoki click)
- Qolgan 80%'ga g'olib variantni yuboradi

Nimani test qilish mumkin:

- **Subject line** — eng ko'p test qilinadi (open rate'ga ta'sir)
- **Content** — CTA matni, rasm, tugma rangi (click rate'ga ta'sir)
- **Send time** — qachon yuborish
- **From name** — "Company" vs "Ali from Company"

**⚠️ Ehtiyot bo'l:** Bir vaqtda faqat **bitta** narsani o'zgartiring. Subject VA content'ni birga o'zgartirsangiz, qaysi biri natijaga sabab bo'lganini bilmaysiz. Va yetarli sample bo'lsin — 50 kishilik listda A/B test statistik ma'nosiz.

---

## Metrikalar

Kampaniya muvaffaqiyatini raqamlar ko'rsatadi. Asosiylari:

| Metrika | Nima o'lchaydi | Qanday yaxshilash |
|---------|----------------|-------------------|
| **Delivery rate** | Yetkazilgan / yuborilgan | List tozalash, deliverability (SPF/DKIM) |
| **Open rate** | Ochganlar / yetkazilgan | Yaxshi subject, sender name, send time |
| **Click-through rate (CTR)** | Bosganlar / yetkazilgan | Aniq CTA, tegishli content, dizayn |
| **Bounce rate** | Qaytgan / yuborilgan | List gigiyenasi, double opt-in |
| **Unsubscribe rate** | Obunani bekor qilganlar | Kamroq/tegishliroq email, kutilma to'g'ri |
| **Spam complaint rate** | "Spam" bosganlar | Faqat rozilik berganlarga, oson unsubscribe |
| **Conversion** | Maqsadli harakat (sotib olish) | To'liq voronka, landing page mosligi |

**Bounce ikki xil:**

- **Hard bounce** — manzil mavjud emas / xato. Darhol ro'yxatdan o'chiring.
- **Soft bounce** — vaqtinchalik (inbox to'la, server band). Bir necha marta urinib, keyin o'chiring.

**💡 Tushuncha:** Open rate 2021-yildan beri **ishonchsiz** metrikaga aylandi — Apple Mail Privacy Protection (MPP) rasmlarni oldindan yuklab, "ochildi" deb belgilaydi (aslida ochilmagan bo'lsa ham). Shuning uchun 2026-da **click rate va conversion** ancha ishonchli KPI hisoblanadi.

**⚠️ Ehtiyot bo'l:** Spam complaint rate 0.1%'dan oshsa — jiddiy muammo. Gmail/Yahoo bu chegaradan oshsa domeningizni blok qilishi mumkin. Har doim oson, real ishlaydigan unsubscribe bering.

---

## Qonuniy talablar

Bu **eng muhim** qism — buzilsa spam bloki, jarima yoki sud. Marketing email yuborayotgan bo'lsangiz, bularni bilishingiz **shart**.

**CAN-SPAM (AQSh):**

- Yolg'on "From" / subject taqiqlangan
- Reklama ekani ko'rinsin (agar shunday bo'lsa)
- **Jismoniy pochta manzili** (physical address) email ichida bo'lishi shart
- **Unsubscribe link** — aniq, ishlaydigan, 10 kun ichida bajariladigan
- Har bir buzilish uchun jarima (juda katta)

**GDPR (Yevropa Ittifoqi):**

- **Rozilik (consent)** — foydalanuvchi aniq (opt-in) rozilik bergan bo'lishi kerak. Oldindan belgilangan checkbox (pre-checked) — taqiqlangan.
- Ma'lumotni nima uchun yig'ishni tushuntirish
- Foydalanuvchi ma'lumotini ko'rish/o'chirishni so'rashi mumkin
- Buzilsa — jarima aylanmaning 4%'igacha

**Umumiy majburiy elementlar (har bir marketing email'da):**

1. Ishlaydigan **unsubscribe link** (footer'da)
2. Jo'natuvchining real **jismoniy manzili**
3. Aniq **sender identity** (kim yuboryapti — yolg'on emas)
4. Rozilik asosida yig'ilgan list

```html
<!-- Har bir marketing email footer'i -->
<td style="font-size: 12px; color: #999; text-align: center; padding: 24px;">
  Company Inc, Amir Temur ko'chasi 1, Toshkent, O'zbekiston<br>
  Bu email'ni ololmaslik uchun
  <a href="{{unsubscribe_url}}" style="color:#999;">obunani bekor qiling</a>.
</td>
```

**💡 Tushuncha:** Gmail/Yahoo 2024-yildan beri katta jo'natuvchilardan **one-click unsubscribe** (`List-Unsubscribe` header) talab qiladi. Ko'p ESP buni avtomatik qo'shadi, lekin o'z SMTP'ingizdan yuborsangiz — o'zingiz qo'shishingiz kerak.

**⚠️ Ehtiyot bo'l:** O'zbekistonda ham shaxsiy ma'lumotlar to'g'risidagi qonun bor, va sizning mijozlaringiz EI/AQSh'da bo'lsa, GDPR/CAN-SPAM baribir amal qiladi. "Bizga tegishli emas" deb o'ylamang.

---

## Automation / drip misollari

Drip campaign — trigger'ga qarab avtomatik ketadigan email sequence. Eng keng tarqalganlari:

**1. Welcome sequence** (obunadan keyin):

```
Kun 0:  Xush kelibsiz + tasdiqlash (double opt-in)
Kun 1:  Mahsulot bilan tanishtirish, asosiy foyda
Kun 3:  Foydali resurs / qo'llanma
Kun 7:  Ijtimoiy dalil (mijoz sharhlari) + taklif
```

**2. Abandoned cart** (savatga qo'shib, sotib olmagan):

```
1 soat: "Savatingizda mahsulot qoldi"
24 soat: Eslatma + mahsulot tafsiloti
72 soat: Chegirma taklifi (oxirgi turtki)
```

Abandoned cart — e-commerce'da eng foydali (yuqori ROI) automation'lardan biri.

**3. Onboarding** (yangi mijoz mahsulotni o'rganishi):

```
Ro'yxatdan o'tdi:    Birinchi qadam (setup)
Setup tugadi:        Keyingi feature
3 kun faol emas:     "Yordam kerakmi?" re-engagement
```

**💡 Tushuncha:** Drip'ning kuchi — u **harakatga** bog'lanadi, nafaqat vaqtga. "Setup tugadi" event'i keyingi email'ni ochadi. Customer.io va Loops kabi platformalar shu event-based automation'da kuchli.

---

## Marketing platformalar taqqoslash

2026-yildagi mashhur ESP / marketing platformalar:

| Platforma | Kuchli tomoni | Kimga |
|-----------|---------------|-------|
| **Mailchimp** | Eng mashhur, hamma narsa bor, template ko'p | Kichik biznes, newsletter |
| **Brevo** (eski Sendinblue) | Arzon, SMS + email, transactional ham | Byudjet, EI'dagi kompaniyalar |
| **Loops** | Modern, developer-friendly, SaaS uchun | Startaplar, product email |
| **Customer.io** | Kuchli event-based automation, segmentatsiya | Data-heavy, katta product |
| **ConvertKit** (Kit) | Creator/blogger'lar uchun, sodda | Kontent yaratuvchilar |

Yana: **Resend** (developer transactional + broadcast, React Email bilan), **Postmark** (faqat transactional, tez), **Klaviyo** (e-commerce marketing).

**ESP tanlash mezoni:**

- **Nima yuborasiz?** Faqat transactional → Postmark/Resend. Marketing → Mailchimp/Brevo. Ikkalasi → Resend/Brevo.
- **Automation kerakmi?** Murakkab event-based → Customer.io/Loops.
- **Byudjet?** Brevo arzon, Mailchimp qimmatlashadi.
- **Developer-mi yoki marketolog?** Loops/Resend developer'ga, Mailchimp/ConvertKit marketologga qulay.
- **Deliverability va support** — eng muhimlaridan.

**💡 Tushuncha:** Ko'p SaaS kompaniya ikki tool ishlatadi: transactional uchun **Resend/Postmark**, marketing uchun **Loops/Customer.io**. Bitta tool hammasini a'lo qilmaydi.

---

## Savol-javoblar

### ❓ Nega email'da `<div>` bilan layout qilib bo'lmaydi?

**✅ Javob:** Outlook (Windows desktop) HTML'ni Microsoft Word engine bilan render qiladi, va u `<div>` uchun modern CSS (`float`, `flexbox`, `position`) ni qo'llab-quvvatlamaydi. `<table>` esa Word'da ham ishonchli ishlaydi, chunki jadval juda eski, barqaror texnologiya. Shuning uchun email layout'i har doim nested `<table>` ustiga quriladi.

### ❓ Nega inline CSS ishlataman, `<style>` blok yaxshiroq-ku?

**✅ Javob:** Ko'p email client (ayniqsa Gmail ba'zi holatlarda) `<head>`dagi yoki `<style>` blokidagi CSS'ni olib tashlaydi yoki cheklaydi. Inline `style="..."` esa hamma joyda ishlaydi. Amalda: `<style>`ni media query (responsive) uchun ishlating, lekin asosiy bezakni inline qiling. MJML/React Email buni avtomatik inline qiladi.

### ❓ MJML va React Email'ning qaysi birini tanlayman?

**✅ Javob:** Agar jamoangiz JavaScript/TypeScript va React biladigan bo'lsa — **React Email** (komponent qayta ishlatish, TS, propslar, lokal preview). Agar dizaynerlar markup yozsa yoki React ishlatmasangiz — **MJML** sodda va tez. Ikkalasi ham responsive va Outlook-safe HTML generatsiya qiladi, shuning uchun qo'lda `<table>` yozishdan yaxshi.

### ❓ Rasmlar nega ko'p clientda ko'rinmaydi?

**✅ Javob:** Ko'p client (Outlook, Gmail default) xavfsizlik uchun tashqi rasmlarni avtomatik yuklamaydi — foydalanuvchi "Show images" bosishi kerak. Shuning uchun: (1) muhim matnni rasm ichiga qo'ymang, HTML matn qiling; (2) har bir `<img>`ga mazmunli `alt` bering; (3) rasm absolute `https://` URL'da (CDN) bo'lsin.

### ❓ Preheader (preview text) nima va nega kerak?

**✅ Javob:** Bu inbox'da subject'dan keyin ko'rinadigan qisqa matn. Uni maxsus belgilamasangiz, client email boshidagi tasodifiy matnni ("View in browser" kabi) ko'rsatadi, bu open rate'ni tushiradi. Body eng boshiga ko'rinmas `<div>` bilan tegishli preheader matnini qo'ying.

### ❓ Transactional email'ga marketing xabar qo'shsam bo'ladimi?

**✅ Javob:** Yo'q, xavfli. Transactional (parol reset, chek) rozilik talab qilmaydi, lekin ichiga reklama qo'shsangiz — CAN-SPAM buziladi va unsubscribe majburiy bo'ladi. Bundan tashqari, transactional email'ni marketing bilan aralashtirish deliverability'ni buzadi. Ularni alohida ushlang.

### ❓ Nega marketing va transactional'ni alohida domendan yuborish tavsiya etiladi?

**✅ Javob:** Marketing email'da spam complaint yuqori bo'ladi (odamlar "Spam" bosadi). Bu sender reputatsiyasini pasaytiradi. Agar transactional ham shu domendan chiqsa, sizning parol reset / chek email'laringiz ham spam'ga tusha boshlaydi — bu biznes uchun halokat. Subdomen ajratish (`mail.` vs `send.`) reputatsiyani izolyatsiya qiladi.

### ❓ Open rate pasayib ketdi, sabab nima bo'lishi mumkin?

**✅ Javob:** Bir necha sabab: (1) subject line zaif; (2) deliverability muammosi — email spam'ga tushmoqda (SPF/DKIM/DMARC tekshiring); (3) sender name notanish; (4) send time noto'g'ri. Yana e'tibor bering — Apple Mail Privacy Protection tufayli open rate 2021-dan beri to'liq ishonchli emas. Click rate'ni ham kuzating.

### ❓ Double opt-in shartmi? U ro'yxatni kichraytiradi-ku.

**✅ Javob:** Majburiy emas (ba'zi holatlarda GDPR tavsiya qiladi), lekin qattiq tavsiya etiladi. Ha, u ro'yxatni kichraytiradi — lekin faqat **soxta/typo/bot** manzillarni olib tashlaydi. Natijada real, faol obunachilar qoladi, bu open rate va deliverability'ni oshiradi. "Katta lekin o'lik" ro'yxatdan "kichik lekin faol" ro'yxat yaxshi.

### ❓ Unsubscribe link'ni yashirsam yoki qiyinlashtirsam bo'ladimi (odamlar chiqib ketmasin)?

**✅ Javob:** Yo'q, buni qilmang. Bu qonun buzilishi (CAN-SPAM/GDPR) va teskari samara beradi: unsubscribe topolmagan odam "Spam" bosadi, bu esa reputatsiyaga unsubscribe'dan ancha yomonroq zarar yetkazadi. Gmail/Yahoo endi one-click unsubscribe talab qiladi. Unsubscribe'ni oson qiling.

### ❓ A/B testda nechta narsani birga o'zgartiraman?

**✅ Javob:** Faqat **bitta**. Agar subject VA tugma rangini birga o'zgartirsangiz, qaysi biri natijaga sabab bo'lganini bilmaysiz — test ma'nosini yo'qotadi. Bitta o'zgaruvchi, yetarli sample (kichik listda A/B test statistik ishonchsiz), va g'olibni aniq metrika (open yoki click) bilan tanlang.

### ❓ Spam papkaga tushmaslik uchun eng muhim narsalar nima?

**✅ Javob:** (1) To'g'ri auth — SPF, DKIM, DMARC (01-bo'limda); (2) faqat rozilik bergan real list'ga yuborish; (3) oson unsubscribe (complaint kamayadi); (4) spam trigger so'zlardan qochish ("BEPUL!!!", ko'p "!!!", CAPS); (5) matn/rasm balansi (faqat bitta katta rasm — spam belgisi); (6) barqaror jo'natish hajmi (birdan minglab yubormang).

### ❓ Qaysi ESP'ni startapim uchun tanlayman?

**✅ Javob:** Bog'liq. Agar asosan transactional (SaaS product email) → **Resend** yoki **Postmark**. Agar marketing/newsletter → **Loops** (modern) yoki **Brevo** (arzon). Murakkab event-based automation → **Customer.io**. Ko'p startap ikkitasini ishlatadi: transactional uchun Resend, marketing uchun Loops. Deliverability, narx va developer tajribasiga qarab tanlang.

### ❓ Custom font (Google Fonts) email'da ishlatsam bo'ladimi?

**✅ Javob:** Qisman. Apple Mail va ba'zi client custom font'ni qo'llab-quvvatlaydi, lekin Outlook va Gmail ko'pincha yo'q. Shuning uchun `font-family`da har doim web-safe fallback bering (`'Inter', Arial, sans-serif`). Font yuklanmasa, Arial'ga tushadi va email baribir o'qiladigan bo'ladi. Brend uchun font muhim bo'lsa, logotip/sarlavhani rasm qiling (lekin asosiy matnni emas).

### ❓ Merge tag'da qiymat bo'lmasa nima bo'ladi?

**✅ Javob:** "Salom, !" kabi buzuq matn chiqadi — bu juda yomon taassurot. Har doim fallback/default bering: `{{first_name | default: "do'stim"}}`. Va merge tag'ni subject'da ishlatishdan oldin data toza ekaniga ishonch hosil qiling — subject'dagi buzuq merge tag open rate'ni to'g'ridan-to'g'ri pasaytiradi.

---

## Masalalar

> Yechimlar: [`solutions/email/02-templates-campaigns.md`](../solutions/email/02-templates-campaigns.md)

1. **Table-based welcome email.** Qo'lda (vosita ishlatmasdan) 600px kenglikdagi, markazlashtirilgan welcome email HTML yozing: logotip (`<img>` alt bilan), sarlavha, ikki abzas matn, CTA tugma va footer (jismoniy manzil + unsubscribe). Barcha CSS inline bo'lsin, `<table>` layout ishlatilsin.

2. **React Email komponenti.** `PasswordResetEmail` nomli React Email komponent yozing: props sifatida `userName` va `resetUrl` oladi. Ichida sarlavha, tushuntirish matni, "Parolni tiklash" tugmasi va "agar siz so'ramagan bo'lsangiz e'tibor bermang" ogohlantirishi bo'lsin.

3. **MJML → HTML.** Bitta section, ikki ustunli (chapda rasm, o'ngda matn+tugma) MJML template yozing. Mobil ekranda ustunlar stack bo'lishini tushuntiring. Keyin uni HTML'ga compile qilish buyrug'ini yozing.

4. **Merge tag va fallback.** Subject va body'da merge tag ishlatilgan template yozing (`first_name`, `city`, `plan`). Har bir tag uchun fallback bering va `plan == "premium"` bo'lsa boshqa xabar ko'rsatadigan shartli blok qo'shing (Handlebars yoki Liquid).

5. **Metrikani tahlil qiling.** Kampaniya natijasi: 10,000 yuborilgan, 9,600 yetkazilgan, 2,400 ochilgan, 240 bosgan, 15 unsubscribe, 12 spam complaint. Delivery rate, open rate, CTR, unsubscribe rate va spam complaint rate'ni hisoblang. Qaysi metrika muammoli va nima uchun — tahlil qiling.

6. **Welcome drip sequence dizayni.** SaaS mahsulot uchun 4 email'lik welcome sequence rejalashtiring: har birining trigger'i (vaqt yoki event), maqsadi, subject line'i va asosiy CTA'sini yozing. Kamida bittasi event-based (vaqtga emas, harakatga bog'liq) bo'lsin.

7. **Qonuniy audit.** Sizga marketing email berilgan: unsubscribe link yo'q, jismoniy manzil yo'q, "From: noreply@" ko'rinishida, list sotib olingan. CAN-SPAM va GDPR bo'yicha qaysi qoidalar buzilganini sanang va har birini qanday tuzatishni yozing.

8. **ESP tanlash keysi.** Uch stsenariy uchun ESP tanlang va sababini yozing: (a) faqat parol reset / chek yuboradigan SaaS; (b) haftalik newsletter yuboradigan blogger; (c) murakkab event-based onboarding + abandoned cart kerak bo'lgan e-commerce.

9. **Responsive muammosi.** Ikki ustunli email desktop'da yaxshi, lekin mobilda ustunlar siqilib o'qib bo'lmayapti. Media query bilan mobilda ustunlarni stack qiladigan (bir-birining ostiga) yechim yozing va nega ba'zi clientda media query ishlamasligini tushuntiring.

10. **Spam'dan qutulish.** Sizning kampaniyangiz Gmail'da doim "Promotions" yoki "Spam"ga tushyapti. Deliverability va content bo'yicha kamida 6 ta tekshirish/tuzatish ro'yxatini tuzing (auth, list, content, unsubscribe, hajm va boshqalar).

---

← [Email bo'limiga qaytish](./README.md)
