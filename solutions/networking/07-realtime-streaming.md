# Real-time va Streaming Protokollar — Masalalar Yechimi

Bu fayl [`networking/07-realtime-streaming.md`](../../networking/07-realtime-streaming.md) bo'limidagi masalalarning yechimlari.

---

## 1-masala — Polling tahlili

**Hisob:**
- Har foydalanuvchi 2 sekundda 1 so'rov = daqiqada 30 so'rov.
- 10 000 foydalanuvchi × 30 = **300 000 so'rov/daqiqa**.
- 95% bo'sh: `300 000 × 0.95 = 285 000` **behuda so'rov/daqiqa**.

**Yechim — SSE yoki WebSocket'ga o'tish.** Polling muammosi: ma'lumot bo'lmasa ham doimiy so'rov ketadi. Faqat **server → client** push yetarli bo'lsa (notification/feed) — **SSE**, chunki:
- bitta ochiq ulanish, takroriy so'rov yo'q;
- server ma'lumot bo'lganda **faqat o'shanda** yuboradi → behuda trafik nolga yaqin;
- HTTP ustida, avto-reconnect bilan.

Ikki tomonlama almashinuv kerak bo'lsa — **WebSocket**. Agar to'liq o'tish imkoni bo'lmasa, oraliq qadam: **long polling** (bo'sh so'rovlarni kamaytiradi).

**Asosiy g'oya:** behuda so'rovlar push modelida deyarli yo'qoladi, chunki ulanish faqat real voqea bo'lganda ma'lumot olib keladi.

---

## 2-masala — SSE yoki WebSocket?

| Holat | Tanlov | Sabab |
|-------|--------|-------|
| (a) birja narxlari tickeri | **SSE** | Faqat server → client oqim; client hech narsa yubormaydi. Avto-reconnect foydali. |
| (b) ikki kishilik shaxmat | **WebSocket** | Ikki tomonlama, past kechikishli (har ikki o'yinchi yuradi). |
| (c) jonli log paneli | **SSE** | Bir tomonlama matn oqimi; klassik SSE use-case. |
| (d) hamkorlikdagi chizish doskasi | **WebSocket** | Ko'p foydalanuvchi tez-tez ikki tomonlama event yuboradi; past kechikish kritik. |

**Qoida:** faqat server→client push → SSE; ikki tomonlama/binary/juda past latency → WebSocket.

---

## 3-masala — Handshake o'qish

Bu **WebSocket handshake** so'rovi (HTTP Upgrade orqali).

**Muhim header'lar:**
- `Upgrade: websocket` + `Connection: Upgrade` — protokolni HTTP'dan WebSocket'ga o'tkazishni so'raydi.
- `Sec-WebSocket-Key` — tasodifiy base64 nonce; server uni standart GUID bilan birlashtirib, SHA-1 + base64 qilib `Sec-WebSocket-Accept`'ni hisoblaydi (handshake'ni tasdiqlash uchun, kesh/proxy aldovlaridan himoya).
- `Sec-WebSocket-Version: 13` — protokol versiyasi.

**Server javobi:**
```text
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: <hisoblangan-qiymat>
```
`101` status kod orqali server protokolni o'zgartirishga rozi bo'ladi; shundan keyin bir xil TCP ustida WebSocket frame'lari oqadi.

---

## 4-masala — HOL blocking

**Nega HTTP/2 yetarli emas:** HTTP/2 multiplexing **application-layer** HOL blocking'ni yechadi, lekin transport TCP bo'lib qoladi. TCP **tartibni kafolatlaydi** — bitta paket yo'qolsa, undan keyin kelgan barcha baytlar (turli HTTP/2 stream'larga tegishli bo'lsa ham) yo'qolgan paket qayta uzatilguncha kernel'da kutib turadi. Bu **TCP-layer HOL blocking**. Yuqori packet loss'li mobil tarmoqlarda bu sezilarli sekinlikni keltiradi.

**Yechim — HTTP/3 (QUIC).** QUIC `UDP` ustida ishlaydi va stream'larni **mustaqil** qiladi: bitta paket yo'qolsa faqat o'sha stream kutadi, qolgan stream'lar davom etadi. Shuning uchun TCP HOL blocking yo'qoladi va packet loss'li tarmoqlarda HTTP/3 sezilarli yutuq beradi.

---

## 5-masala — gRPC dizayni

| Vazifa | Tur | Signature |
|--------|-----|-----------|
| (a) ID bo'yicha profil | unary | `rpc GetProfile (UserId) returns (Profile);` |
| (b) sensor oqimi → server | client streaming | `rpc PushSensor (stream Reading) returns (Ack);` |
| (c) jonli hisob kuzatish | server streaming | `rpc WatchScore (MatchId) returns (stream Score);` |
| (d) real-time chat | bidirectional | `rpc Chat (stream Message) returns (stream Message);` |

**Mantiq:** so'rov/javobning qaysi tomoni ko'p (stream) ekanini aniqlash kifoya — bitta↔bitta = unary, ko'p→bitta = client streaming, bitta→ko'p = server streaming, ko'p↔ko'p = bidirectional.

---

## 6-masala — Scaling arxitekturasi

```text
       A ──ws──► [Server-1] ──publish─┐
                                      ▼
                              ┌──────────────────┐
                              │  Pub/Sub broker  │  (Redis/Kafka/NATS)
                              │  channel: room   │
                              └──────────────────┘
                                      │ subscribe
       B ◄──ws── [Server-3] ◄─────────┘
```

**Oqim:**
1. A `Server-1`'ga ulangan, xabar yuboradi.
2. `Server-1` xabarni broker'ning tegishli channel'iga **publish** qiladi.
3. `Server-3` o'sha channel'ga **subscribe** bo'lgani uchun xabarni oladi.
4. `Server-3` xabarni o'zidagi B'ga WebSocket orqali yuboradi.

**Kerakli komponentlar:**
- **Load balancer** — `sticky session` (session affinity) yoqilgan, har client doim bir serverda qoladi.
- **Pub/Sub broker** — serverlararo fan-out (Redis Pub/Sub yetarli; ishonchli yetkazish/tarix kerak bo'lsa Kafka/NATS).
- **Presence/routing** (ixtiyoriy) — qaysi foydalanuvchi qaysi serverda ekanini bilish uchun.

---

## 7-masala — HTTP versiyalari

| | HTTP/1.1 | HTTP/2 | HTTP/3 |
|--|----------|--------|--------|
| Transport | TCP | TCP | QUIC (UDP) |
| Multiplexing | yo'q | ha | ha |
| App HOL blocking | bor | yo'q | yo'q |
| TCP HOL blocking | bor | bor | yo'q |
| Handshake | sekin (2-3 RTT) | sekin | tez (0/1-RTT) |

**Nega UDP mantiqiy:** UDP "xom" va ishonchsiz ko'rinadi, lekin aynan shu sababli **moslashuvchan**. TCP ishonchlilik/tartibni kernel'da qattiq belgilab qo'ygan — uni o'zgartirib bo'lmaydi. QUIC esa UDP'ning "bo'sh" sahifasidan boshlab, ishonchlilik, tartib, shifrlash va multiplexing'ni **user-space**'da, **stream darajasida mustaqil** qilib quradi. Shu sababli TCP HOL blocking'dan qutuladi, tezroq handshake va connection migration beradi. Ya'ni UDP — yakuniy yechim emas, balki ustiga yaxshiroq transport qurish uchun toza poydevor.

---

## 8-masala — WebRTC arxitekturasi

**Komponentlar va bosqichlar:**

1. **Signaling server** (odatda WebSocket) — peer'lar SDP offer/answer va ICE candidate'larni almashadi. Bu **ulanishni o'rnatish** bosqichida ishlaydi; media o'rnatilgach shart emas.
2. **STUN server** — har peer'ga uning tashqi (public) IP/port'ini aytadi, shunda peer'lar to'g'ridan-to'g'ri P2P ulanishga urinadi. **Ulanishni o'rnatish** bosqichida.
3. **TURN server** — agar NAT/firewall to'g'ridan-to'g'ri P2P ulanishga yo'l qo'ymasa, media'ni o'zi orqali **relay** qiladi.

```text
   A ◄═══ media (P2P) ═══► B      (ideal holat — STUN yetarli)

   A ──► [TURN relay] ──► B       (P2P imkonsiz — TURN orqali)

   A ── signaling (WS) ── Server ── signaling ── B   (faqat o'rnatishda)
```

**TURN qachon:** to'g'ridan-to'g'ri P2P ulanish o'rnatib bo'lmaganda (simmetrik NAT, qattiq firewall). U media'ni relay qilgani uchun server resursi va kechikish qimmat — shuning uchun **oxirgi chora** sifatida, faqat STUN ishlamaganda ishga tushadi.

---

## 9-masala — Reconnect strategiyasi

Bu **thundering herd** (reconnect storm) muammosi.

**Client tomonda:**
- **Exponential backoff + jitter**: qayta ulanishni 1s, 2s, 4s... oshirib, ustiga tasodifiy jitter qo'shish — shunda barcha client bir vaqtda emas, tarqoq urinади.
- Maksimal urinish/oraliq cheklash.

**Server tomonda:**
- **Connection draining**: deploy'da ulanishlarni bir vaqtda emas, **sekin-asta** uzish; client'larni asta boshqa instansga ko'chirish.
- **Rolling/blue-green deploy**: barcha serverni bir vaqtda qayta ishga tushirmaslik.
- **Rate limiting** qayta ulanish endpoint'ida.
- Load balancer darajasida qabul tezligini cheklash.

---

## 10-masala — Protokol tanlash bayonnomasi

**"Hammasi uchun WebSocket" yondashuvining kamchiliklari:**
- WebSocket **stateful** — har ulanish xotira/connection slot band qiladi; oddiy notification uchun ortiqcha.
- **Sticky session** va Pub/Sub fan-out kabi murakkab scaling infratuzilma talab qiladi.
- Avto-reconnect, heartbeat, ack/retry'ni **qo'lda** yozish kerak.
- Ba'zi proxy/firewall WebSocket'ni bloklashi mumkin.

**SSE / HTTP/2 SSE qachon afzal:**
- Faqat **server → client** push bo'lsa (notification, feed, dashboard) — SSE oddiyroq, HTTP ustida, brauzer avto-reconnect beradi.
- **HTTP/2 ustidagi SSE** — HTTP/1.1'dagi 6-ulanish limitini ham yechadi (multiplexing), shuning uchun ko'p SSE oqimini bitta ulanishda olib ketadi.

**Xulosa:** to'g'ri yondashuv — ehtiyojga qarab tanlash: bir tomonlama push uchun SSE, haqiqiy ikki tomonlama interaksiya uchun WebSocket. "Bitta bolg'a bilan hamma narsa mix" anti-pattern.

---

← [Networking bo'limiga qaytish](../../networking/README.md)
