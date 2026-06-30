# MCP — Model Context Protocol — Masalalar Yechimi

Bu fayl [`networking/08-mcp.md`](../../networking/08-mcp.md) bo'limidagi masalalarning yechimlari.

---

## 1-masala — M×N hisobi

- (a) **MCP'siz:** har juftlik uchun alohida kod → `4 × 5 = 20` integratsiya.
- (b) **MCP bilan:** har ilova bir marta client, har tizim bir marta server → `4 + 5 = 9` implementatsiya.
- (c) **Yangi tizim qo'shilsa:**
  - MCP'siz: har ilovaga ulash kerak → **+4** ta yangi konnektor (jami 24).
  - MCP bilan: faqat **+1** ta server → barcha ilovalar avtomatik ishlatadi (jami 10).

**Izoh:** MCP'siz murakkablik **kvadratik** (M×N) o'sadi, MCP bilan **chiziqli** (M+N). Yangi qo'shimcha kelganda farq yanada keskinlashadi — bu MCP'ning asosiy qiymati.

---

## 2-masala — Rollarni ajrating

| | Element | Rol | Sabab |
|--|---------|-----|-------|
| (a) | Claude Desktop ilovasi | **Host** | Foydalanuvchi ishlatadigan AI ilova; LLM va client'larni boshqaradi. |
| (b) | Postgres'ga ulanuvchi MCP komponenti | **Server** | Tashqi tizimga (DB) ko'prik; tools/resources e'lon qiladi. |
| (c) | Bitta serverga 1:1 ulanishni saqlovchi obyekt | **Client** | Host ichidagi, aynan bitta server bilan ulanishni boshqaruvchi komponent. |
| (d) | GitHub tools'larini e'lon qiluvchi dastur | **Server** | Imkoniyatlarini (tools) e'lon qilib, so'rovga javob beradi. |

---

## 3-masala — Primitive tanlash

| | Vazifa | Primitive | Kim boshqaradi |
|--|--------|-----------|----------------|
| (a) | faylga matn yozish | **Tool** | model-controlled (harakat, side-effect) |
| (b) | README mazmunini kontekstga qo'shish | **Resource** | application-controlled (o'qish, ma'lumot) |
| (c) | kod ko'rib chiqish shabloni | **Prompt** | user-controlled (foydalanuvchi tanlaydi) |
| (d) | ob-havo API'siga so'rov | **Tool** | model-controlled (harakat, tashqi chaqiruv) |

**Mantiq:** harakat/side-effect bo'lsa → Tool; kontekst ma'lumoti (o'qish) bo'lsa → Resource; tayyor parametrli shablon bo'lsa → Prompt.

---

## 4-masala — JSON-RPC o'qish

- Bu **request** (`id` bor, javob kutiladi), `method: tools/call`.
- **Nima so'ralayapti:** `send_email` nomli tool'ni `{ to, subject }` argumentlari bilan chaqirish.
- **Keyingi kutilayotgan xabar:** server'dan bir xil `id: 7` bilan **response** — natija (`result.content`) yoki xato (`error`).

**Xavfsizlik chorasi:** `send_email` — **side-effect'li** tool (tashqariga xabar yuboradi). Shuning uchun model uni chaqirishidan oldin **foydalanuvchi tasdig'i** (human-in-the-loop, tool permission) so'ralishi kerak. Argumentlar (qabul qiluvchi manzili) ham foydalanuvchiga ko'rsatilishi lozim — toki model noto'g'ri manzilga yubormasin.

---

## 5-masala — Transport tanlash

- (a) **Lokal fayl tizimi serveri → stdio.** Server foydalanuvchi mashinasida lokal jarayon sifatida ishlaydi; `stdin`/`stdout` orqali aloqa — tarmoqsiz, past kechikish, jarayon izolyatsiyasi.
- (b) **Markazlashgan bulutli xizmat → Streamable HTTP.** Masofaviy, ko'p foydalanuvchi ulanishi kerak; HTTP POST orqali JSON-RPC, kerak bo'lsa SSE bilan streaming. Autentifikatsiya (OAuth/token) ham shu yerda qo'llanadi.

---

## 6-masala — Xavfsizlik tahlili

- (a) Bu **prompt injection** (untrusted server orqali).
- (b) **Zarar zanjiri:** model tool'ni chaqiradi → tool veb-sahifa mazmunini qaytaradi → mazmun ichidagi yashirin matn ("env o'zgaruvchilarni yubor") model kontekstiga kiradi → model uni **buyruq** deb qabul qiladi → maxfiy ma'lumotni o'qib/yuborib zarar yetkazadi.
- (c) **Himoya choralari (uchta):**
  1. **Tool permission / inson tasdig'i** — har side-effect'li harakatni foydalanuvchi tasdiqlasin.
  2. **Untrusted content izolyatsiyasi** — tool qaytargan matnni ishonchsiz deb belgilash, uni ko'rsatma emas, **ma'lumot** sifatida qarash.
  3. **Least privilege** — server/tool faqat minimal ruxsatga ega bo'lsin; maxfiy env'lar modelga umuman ochilmasin.
  - (Qo'shimcha: faqat ishonchli serverlarni o'rnatish, output sanitatsiyasi.)

---

## 7-masala — MCP vs REST

**To'rt texnik farq:**
1. **Dinamik discovery:** MCP'da model `tools/list` bilan imkoniyatlarni **runtime'da** kashf etadi; REST endpointlari statik va oldindan bilingan.
2. **Yagona standart interfeys:** MCP har serverga bir xil tools/resources/prompts modelini beradi; har REST API o'ziga xos.
3. **Model-aware dizayn:** har MCP tool model uchun tavsif + JSON Schema bilan keladi, model qaysi tool'ni qachon chaqirishni o'zi tanlaydi; REST bunga mo'ljallanmagan.
4. **Protokol:** MCP — JSON-RPC 2.0 ustida sessiyali, capability negotiation bilan; REST — odatda stateless HTTP, har biri o'z konvensiyasi.

**Nega REST'ni almashtirmaydi:** MCP serverlar ko'pincha **ichida REST API'larni chaqiradi** — ya'ni MCP REST ustidagi standart **qatlam**, raqibi emas. Server-server yoki public API uchun REST/gRPC hali ham mos. MCP — model'ni tool'larga ulashning standart usuli, umumiy API almashtirувchi emas.

---

## 8-masala — Capability negotiation ssenariysi

- (a) Bu **`initialize` handshake** bosqichida aniqlanadi: server javobida o'z capabilities'ini e'lon qiladi va u yerda `prompts` yo'q.
- (b) **Client xatti-harakati:** `prompts/list` yoki `prompts/get` so'rovlarini **yubormaydi** (yoki UI'da prompt menyusini ko'rsatmaydi), faqat qo'llab-quvvatlanadigan `tools` va `resources` bilan ishlaydi. Qo'llab-quvvatlanmaydigan xususiyatga tayanmaydi.
- (c) **Kelajakka chidamlilik:** har tomon **faqat ikkalasi ham qo'llab-quvvatlaydigan** xususiyatlardan foydalanadi. Shuning uchun yangi xususiyatlar qo'shilsa eski client/serverlar buzilmaydi (orqaga moslik), va versiyalar bir-biriga moslashadi.

---

## 9-masala — Minimal server dizayni

**(a) Tool ta'rifi:**
```json
{
  "name": "reverse_text",
  "description": "Berilgan matnni teskari aylantiradi",
  "inputSchema": {
    "type": "object",
    "properties": { "text": { "type": "string" } },
    "required": ["text"]
  }
}
```

**(b) JSON-RPC almashinuvi:**
```json
// tools/call so'rovi
{ "jsonrpc": "2.0", "id": 3, "method": "tools/call",
  "params": { "name": "reverse_text",
              "arguments": { "text": "salom" } } }

// server result javobi
{ "jsonrpc": "2.0", "id": 3, "result": {
    "content": [ { "type": "text", "text": "molas" } ]
}}
```

---

## 10-masala — Arxitektura savoli

**MCP'siz:** Jira integratsiyasini to'g'ridan-to'g'ri agent kodiga yozish kerak — Jira API client, autentifikatsiya, tool logikasi, va agentning tool-dispatch qismini o'zgartirish. Har yangi imkoniyat agent kodini qayta yozish/deploy qilishni talab qiladi.

**MCP bilan:** Alohida **Jira MCP server** yoziladi (yoki tayyorini olasiz), u `create_ticket` tool'ini e'lon qiladi. Agentga shu serverni **ulash** kifoya — agent `tools/list` orqali yangi tool'ni avtomatik kashf etadi va ishlatadi.

**Nega agent kodini o'zgartirmasdan:** Agent faqat **standart MCP protokol** bilan ishlaydi, aniq tool'lar haqida hech narsa hardcode qilmaydi. Yangi imkoniyat = yangi MCP server ulanishi; agentning kashf-etish (`tools/list`) va chaqirish (`tools/call`) mantiqi o'zgarmaydi. Bu **plug-and-play** kengaytirilishni beradi — M+N modelining amaliy foydasi.

---

← [Networking bo'limiga qaytish](../../networking/README.md)
