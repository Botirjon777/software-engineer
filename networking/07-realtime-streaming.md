# Real-time va Streaming Protokollar

Klassik web `HTTP request-response` modeli ustiga qurilgan: client so'rov yuboradi, server javob qaytaradi, ulanish yopiladi. Bu model ko'p holatlar uchun yetarli, lekin **chat, notifications, live dashboard, online o'yin, birja narxlari** kabi server o'zi mustaqil ravishda client'ga ma'lumot yuborishi kerak bo'lgan holatlarda ojiz qoladi. Bu bo'limda real-time kommunikatsiyaning evolyutsiyasini — `short polling`'dan `WebSocket`, `HTTP/3` va `WebRTC`'gacha — va ularni qanday miqyoslashtirishni (`scaling`) ko'rib chiqamiz.

## Mundarija

- [Real-time muammosi: HTTP request-response cheklovi](#real-time-muammosi-http-request-response-cheklovi)
- [Short polling](#short-polling)
- [Long polling](#long-polling)
- [Server-Sent Events (SSE)](#server-sent-events-sse)
- [WebSocket](#websocket)
- [WebSocket vs SSE vs Polling taqqoslash](#websocket-vs-sse-vs-polling-taqqoslash)
- [HTTP/1.1 vs HTTP/2 vs HTTP/3](#http11-vs-http2-vs-http3)
- [gRPC va streaming turlari](#grpc-va-streaming-turlari)
- [WebRTC qisqacha](#webrtc-qisqacha)
- [Real-time scaling muammolari](#real-time-scaling-muammolari)
- [Savol-javoblar](#savol-javoblar)
- [Masalalar](#masalalar)

---

## Real-time muammosi: HTTP request-response cheklovi

**💡 Tushuncha:** Klassik `HTTP/1.1`'da kommunikatsiyani **faqat client boshlaydi**. Server o'zidan-o'zi "Hey, yangi xabar keldi!" deb client'ga ulanishni ocha olmaydi. Server faqat client yuborgan so'rovga **javob** qaytara oladi.

Tasavvur qiling: chat ilovasi yozyapsiz. Foydalanuvchi A xabar yubordi, foydalanuvchi B uni darhol ko'rishi kerak. Lekin B'ning brauzeri serverga "men uchun yangi xabar bormi?" deb so'ramaguncha, server B'ga hech narsa yetkaza olmaydi.

```text
Klassik HTTP (faqat client boshlaydi):

  Client                        Server
    │   GET /messages            │
    │ ─────────────────────────► │
    │   200 OK (bo'sh)           │
    │ ◄───────────────────────── │
    │                            │  ← yangi xabar keldi, lekin
    │                            │     server xabar bera olmaydi!
    │                            │
    │   GET /messages (yana)     │
    │ ─────────────────────────► │
```

Real-time muammosini yechishning bir necha yo'li bor va ular tarixiy ravishda quyidagi tartibda rivojlangan:

```text
Short polling  →  Long polling  →  SSE  →  WebSocket
   (sodda)         (kechikish↓)   (1 yo'l)  (2 yo'l, full-duplex)
```

**⚠️ Ehtiyot bo'l:** "Real-time" so'zi web kontekstida odatda **soft real-time** (past kechikish, lekin kafolatsiz) degani. Bu o'zini-o'zi boshqaradigan avtomobil yoki kardiostimulyatordagi **hard real-time** (qat'iy deadline kafolati) emas.

---

## Short polling

**💡 Tushuncha:** `Short polling` — client serverga **belgilangan interval bilan** (masalan, har 3 sekundda) takror-takror so'rov yuboradi: "yangilik bormi?". Server bor bo'lsa beradi, bo'lmasa bo'sh javob qaytaradi.

```js
// Short polling — eng sodda usul
setInterval(async () => {
  const res = await fetch('/api/messages?since=' + lastId);
  const data = await res.json();
  if (data.length) render(data);
}, 3000); // har 3 sekundda
```

```text
Client                Server
  │  GET /messages     │
  │ ─────────────────► │  bo'sh
  │ ◄───────────────── │
  │     (3s kutish)    │
  │  GET /messages     │
  │ ─────────────────► │  bo'sh
  │ ◄───────────────── │
  │     (3s kutish)    │
  │  GET /messages     │
  │ ─────────────────► │  [yangi xabar!]
  │ ◄───────────────── │
```

**Kamchiliklari:**
- **Latency**: o'rtacha kechikish interval'ning yarmi (3s'da ~1.5s).
- **Behuda trafik**: aksariyat so'rovlar bo'sh javob bilan qaytadi — server va tarmoq resurslari isrof bo'ladi.
- Intervalni qisqartirsa kechikish kamayadi, lekin yuk oshadi — kelishuv (trade-off).

**Qachon mos:** ma'lumot kamdan-kam yangilanadigan, real-time'lik kritik bo'lmagan holatlar (masalan, har 30 sekundda statistikani yangilash).

---

## Long polling

**💡 Tushuncha:** `Long polling`'da client so'rov yuboradi, lekin server **darhol javob qaytarmaydi** — ma'lumot tayyor bo'lguncha (yoki timeout bo'lguncha) ulanishni **ochiq ushlab turadi**. Ma'lumot kelishi bilan javob qaytaradi, client esa darhol yangi so'rov yuboradi.

```text
Client                       Server
  │  GET /messages           │
  │ ───────────────────────► │  (ushlab turadi...)
  │                          │  ← yangi xabar keldi
  │ ◄─────────────────────── │  javob darhol
  │  GET /messages (qayta)   │
  │ ───────────────────────► │  (yana ushlab turadi...)
```

```js
// Long polling — client tomoni
async function poll() {
  try {
    const res = await fetch('/api/messages/longpoll');
    const data = await res.json();
    render(data);
  } catch (e) {
    await sleep(1000); // xatoda biroz kutib qayta urinish
  }
  poll(); // darhol qayta ulanish
}
poll();
```

**Afzalligi:** short polling'ga nisbatan kechikish ancha kam (ma'lumot kelishi bilan yetkaziladi) va bo'sh so'rovlar kamroq.

**⚠️ Ehtiyot bo'l:** Long polling serverda har bir ochiq so'rov uchun resurs (thread/connection) band qiladi. Klassik bloklovchi serverlarda (`thread-per-request`) bu tez tugab qoladi. `Node.js`, `Go` kabi asinxron serverlar buni yaxshiroq ko'taradi. Bu emulyatsiya — haqiqiy push emas, "yashirin polling".

---

## Server-Sent Events (SSE)

**💡 Tushuncha:** `SSE` — server'dan client'ga **bir tomonlama (unidirectional)** uzluksiz oqim. Bitta `HTTP` ulanishi ochiq qoladi va server o'zi xohlagan paytda matn xabarlarini "stream" qiladi. Brauzerda `EventSource` API orqali ishlatiladi.

```text
        SSE — bir tomonlama (server → client)

  Client                          Server
    │  GET /events                 │
    │  Accept: text/event-stream   │
    │ ───────────────────────────► │
    │ ◄─── data: xabar 1 ───────── │
    │ ◄─── data: xabar 2 ───────── │  (ulanish ochiq qoladi)
    │ ◄─── data: xabar 3 ───────── │
```

```js
// SSE — client tomoni (brauzer)
const es = new EventSource('/api/stream');

es.onmessage = (e) => {
  console.log('Yangi:', e.data);
};

es.addEventListener('price', (e) => { // nomlangan event
  updatePrice(JSON.parse(e.data));
});

es.onerror = () => console.log('Ulanish uzildi, brauzer avto-qayta ulanadi');
```

```js
// SSE — server tomoni (Node.js / Express)
app.get('/api/stream', (req, res) => {
  res.set({
    'Content-Type': 'text/event-stream',
    'Cache-Control': 'no-cache',
    'Connection': 'keep-alive',
  });

  const timer = setInterval(() => {
    res.write(`data: ${JSON.stringify({ time: Date.now() })}\n\n`);
  }, 1000);

  req.on('close', () => clearInterval(timer)); // tozalash
});
```

SSE wire-format'i juda sodda — oddiy matn:

```text
event: price
data: {"symbol":"BTC","price":67000}

data: bu default (message) event

id: 42
data: id orqali oxirgi event eslab qolinadi
```

**Afzalliklari:**
- Oddiy `HTTP` ustida ishlaydi — proxy/firewall bilan muammosiz.
- Brauzer **avtomatik qayta ulanadi** (`Last-Event-ID` header orqali qoldirilgan joydan davom etadi).
- WebSocket'dan yengilroq, server tomoni sodda.

**Cheklovlari:**
- **Faqat server → client** (bir tomonlama). Client → server uchun oddiy HTTP so'rov ishlatiladi.
- Faqat **UTF-8 matn** (binary emas).
- `HTTP/1.1`'da brauzer har bir domen uchun ulanishlar sonini cheklaydi (~6) — `HTTP/2`'da bu muammo yo'q.

**Qachon ishlatish:** server → client push yetarli bo'lganda — live feed, notifications, log streaming, narx tickerlari, AI javoblarini token-token oqimlash (LLM streaming).

---

## WebSocket

**💡 Tushuncha:** `WebSocket` — bitta `TCP` ulanishi ustida **to'liq ikki tomonlama (full-duplex)** kommunikatsiya. Client va server bir vaqtning o'zida, mustaqil ravishda bir-biriga xabar yubora oladi. Ulanish bir marta o'rnatiladi va ochiq qoladi.

### Handshake — HTTP Upgrade orqali

WebSocket ulanishi oddiy `HTTP` so'rovi sifatida boshlanadi, keyin protokol "yangilanadi" (upgrade):

```text
Client → Server (HTTP Upgrade so'rovi):

  GET /chat HTTP/1.1
  Host: example.com
  Upgrade: websocket
  Connection: Upgrade
  Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==
  Sec-WebSocket-Version: 13

Server → Client (rozilik):

  HTTP/1.1 101 Switching Protocols
  Upgrade: websocket
  Connection: Upgrade
  Sec-WebSocket-Accept: s3pPLMBiTxaQ9kYGzzhZRbK+xOo=

  ── shundan keyin TCP ustida WebSocket frame'lari oqadi ──
```

`101 Switching Protocols`'dan keyin endi HTTP emas, balki binary WebSocket frame'lari (matn yoki binary) ikki tomonga oqadi.

### ws va wss

- `ws://` — shifrlanmagan (HTTP'ga o'xshash).
- `wss://` — `TLS` ustida shifrlangan (HTTPS'ga o'xshash). **Production'da har doim `wss://` ishlating.**

### Misol kod

```js
// WebSocket — client tomoni (brauzer)
const ws = new WebSocket('wss://example.com/chat');

ws.onopen  = () => ws.send(JSON.stringify({ type: 'join', room: 'uz' }));
ws.onmessage = (e) => console.log('Keldi:', e.data);
ws.onclose = () => console.log('Ulanish yopildi');
ws.onerror = (e) => console.error('Xato:', e);
```

```js
// WebSocket — server tomoni (Node.js, ws kutubxonasi)
import { WebSocketServer } from 'ws';

const wss = new WebSocketServer({ port: 8080 });

wss.on('connection', (socket) => {
  socket.on('message', (data) => {
    // har bir client'ga broadcast
    wss.clients.forEach((c) => {
      if (c.readyState === c.OPEN) c.send(data.toString());
    });
  });
  socket.send('Xush kelibsiz!');
});
```

**Qachon ishlatish:** ikki tomonlama, past kechikishli, tez-tez almashinuv kerak bo'lganda — chat, multiplayer o'yin, kollaborativ tahrirlash (Google Docs), live trading, signaling.

**⚠️ Ehtiyot bo'l:** WebSocket **stateful** — har bir ochiq ulanish serverda xotira va connection slot band qiladi. WebSocket o'zida avtomatik qayta ulanish, ack/retry yoki heartbeat'ni bermaydi — bularni **siz qo'lda** (yoki `Socket.IO` kabi kutubxona bilan) qo'shasiz. `ping/pong` frame'lari bilan o'lik ulanishlarni aniqlash kerak.

---

## WebSocket vs SSE vs Polling taqqoslash

| Xususiyat | Short polling | Long polling | SSE | WebSocket |
|-----------|---------------|--------------|-----|-----------|
| Yo'nalish | client→server | client→server | **server→client** | **ikki tomonlama** |
| Ulanish | qisqa, takroriy | uzun, ushlab turadi | bitta, ochiq | bitta, ochiq |
| Protokol | HTTP | HTTP | HTTP | TCP (HTTP upgrade) |
| Binary | yo'q (JSON) | yo'q | yo'q (faqat matn) | **ha** |
| Avto-reconnect | — | qo'lda | **brauzer o'zi** | qo'lda |
| Latency | yuqori | o'rta | past | **eng past** |
| Server yuki | yuqori (bo'sh so'rov) | o'rta | past | past (lekin stateful) |
| Murakkablik | eng oddiy | oddiy | oddiy | murakkabroq |
| Proxy/firewall | muammosiz | muammosiz | muammosiz | ba'zan muammo |

**Tanlash qoidasi (rule of thumb):**
- Faqat server → client push kerakmi? → **SSE** (oddiyroq).
- Ikki tomonlama, tez-tez almashinuv kerakmi? → **WebSocket**.
- Real-time kritik emasmi, kamdan-kam yangilanadimi? → **Polling**.

---

## HTTP/1.1 vs HTTP/2 vs HTTP/3

**💡 Tushuncha:** HTTP'ning har bir versiyasi avvalgisining performance muammolarini hal qiladi. Bu real-time'ga to'g'ridan-to'g'ri aloqador, chunki SSE/WebSocket ham HTTP transport'i ustida ishlaydi.

### HTTP/1.1 — head-of-line blocking

Bitta TCP ulanishida so'rovlar **ketma-ket** (navbatma-navbat) qayta ishlanadi. Birinchi so'rov sekin bo'lsa, ortidagilar kutadi — bu **head-of-line (HOL) blocking**. Brauzer buni domen uchun 6 ta parallel ulanish ochib aldab o'tadi.

```text
HTTP/1.1 (bir ulanish, ketma-ket):
  │ req1 │ wait... │ req2 │ wait... │ req3 │
  └─ req2 req1 tugashini kutadi (HOL blocking)
```

### HTTP/2 — multiplexing, binary framing, server push

- **Multiplexing**: bitta TCP ulanishida ko'p so'rovlar **parallel** (interleaved) oqadi — application-layer HOL blocking yo'qoladi.
- **Binary framing**: matn o'rniga binary frame'lar — samaraliroq parsing.
- **Header compression (HPACK)**: takroriy headerlar siqiladi.
- **Server push**: server so'ralmagan resurslarni oldindan yuborishi mumkin (amalda kam ishlatilgani uchun deprecate qilindi).

```text
HTTP/2 (bir ulanish, multiplexed):
  │ req1 req2 req3 │  ← hammasi parallel, bir TCP ustida
```

**⚠️ Ehtiyot bo'l:** HTTP/2 **application-layer** HOL blocking'ni yechadi, lekin **TCP-layer** HOL blocking qoladi: bitta TCP paket yo'qolsa, butun ulanishdagi barcha stream'lar kutadi (chunki TCP tartibni kafolatlaydi). Bu muammoni HTTP/3 yechadi.

### HTTP/3 — QUIC, UDP ustida, 0-RTT

`HTTP/3` `TCP` o'rniga **QUIC** transport'ini ishlatadi — bu **UDP** ustiga qurilgan, lekin ishonchlilik, tartib va shifrlashni o'zi ta'minlaydi.

- **TCP HOL blocking yo'q**: QUIC'da har bir stream mustaqil — bitta paket yo'qolsa faqat o'sha stream kutadi, qolganlari davom etadi.
- **0-RTT / 1-RTT ulanish**: QUIC `TLS 1.3`'ni transport'ga qo'shgan — handshake va shifrlash birga, ulanish ancha tez. Avval ulangan serverga **0-RTT** bilan darhol ma'lumot yuborish mumkin.
- **Connection migration**: ulanish IP'ga emas, `Connection ID`'ga bog'langan — Wi-Fi'dan mobil tarmoqqa o'tganda ulanish uzilmaydi.

```text
TCP+TLS handshake:        QUIC handshake:
  TCP SYN/ACK (1-RTT)       QUIC + TLS birga
  TLS handshake (1-2 RTT)   (1-RTT yoki 0-RTT)
  → 2-3 RTT jami            → 1 RTT yoki 0 RTT
```

| | HTTP/1.1 | HTTP/2 | HTTP/3 |
|--|----------|--------|--------|
| Transport | TCP | TCP | **QUIC (UDP)** |
| Multiplexing | yo'q | ha | ha |
| App HOL blocking | bor | yo'q | yo'q |
| TCP HOL blocking | bor | **bor** | **yo'q** |
| Format | matn | binary | binary |
| Handshake | sekin | sekin | **tez (0/1-RTT)** |

---

## gRPC va streaming turlari

**💡 Tushuncha:** `gRPC` — Google'ning yuqori unumdor RPC framework'i. `HTTP/2` ustida ishlaydi, `Protocol Buffers` (protobuf) binary serializatsiyasidan foydalanadi va to'rt xil chaqiruv turini qo'llab-quvvatlaydi.

```text
1) Unary               — 1 so'rov  → 1 javob (oddiy RPC)
   Client ──req──► Server
   Client ◄─resp── Server

2) Server streaming    — 1 so'rov  → N javob oqimi
   Client ──req──► Server
   Client ◄─r1─r2─r3── Server

3) Client streaming    — N so'rov oqimi → 1 javob
   Client ──r1─r2─r3──► Server
   Client ◄────resp──── Server

4) Bidirectional       — N so'rov ⇄ N javob (mustaqil oqimlar)
   Client ⇄ ⇄ ⇄ Server
```

```text
// .proto — streaming turlarini e'lon qilish
service ChatService {
  rpc GetUser (UserReq) returns (User);                    // unary
  rpc ListUsers (Query) returns (stream User);             // server streaming
  rpc UploadLogs (stream LogEntry) returns (Summary);      // client streaming
  rpc Chat (stream Message) returns (stream Message);      // bidirectional
}
```

- **Unary**: klassik so'rov-javob — REST'ga eng yaqin.
- **Server streaming**: server bir nechta javobni oqim qilib qaytaradi (masalan, katta natijalar ro'yxati, live yangilanishlar).
- **Client streaming**: client ko'p ma'lumot oqimini yuboradi, server bittada javob beradi (masalan, fayl yuklash, telemetriya).
- **Bidirectional streaming**: ikki tomon ham mustaqil oqim yuboradi (chat, real-time o'yin) — `HTTP/2` multiplexing buni mumkin qiladi.

**⚠️ Ehtiyot bo'l:** Brauzerlar to'g'ridan-to'g'ri gRPC'ni (ayniqsa client/bidirectional streaming'ni) qo'llab-quvvatlamaydi — `gRPC-Web` proxy yoki `Connect` protokoli kerak bo'ladi. gRPC asosan **server-server** (microservices) kommunikatsiyasida porlaydi.

---

## WebRTC qisqacha

**💡 Tushuncha:** `WebRTC` (Web Real-Time Communication) — brauzerlar va ilovalar o'rtasida **peer-to-peer (P2P)** audio, video va data uzatish texnologiyasi. Asosiy maqsadi — server orqali o'tmasdan, **to'g'ridan-to'g'ri** ikki client o'rtasida past kechikishli media oqimi.

```text
        WebRTC P2P (server faqat tanishtiruvchi)

   Peer A ◄═══════ media (P2P, UDP/SRTP) ═══════► Peer B
      │                                              │
      └──── signaling (SDP, ICE) ──► Server ◄────────┘
                  (faqat ulanishni o'rnatish uchun)
```

- **P2P**: media to'g'ridan-to'g'ri peer'lar o'rtasida — past kechikish, server yuki kam.
- **Signaling**: peer'lar bir-birini topishi uchun (SDP offer/answer, ICE candidate'lar) **alohida** kanal kerak — buni WebSocket yoki SSE bilan qilasiz. WebRTC signaling'ni o'zi belgilamaydi.
- **NAT traversal**: peer'lar NAT/firewall ortida bo'lgani uchun `STUN` (tashqi IP'ni aniqlash) va `TURN` (to'g'ridan-to'g'ri ulanish bo'lmasa, relay) serverlari kerak.
- **Transport**: media uchun `UDP`/`SRTP` (kechikish > ishonchlilik), data uchun `DataChannel`.

**Qachon:** video qo'ng'iroq, audio konferensiya, ekran ulashish, P2P fayl uzatish, bulutli o'yin.

---

## Real-time scaling muammolari

**💡 Tushuncha:** Klassik REST API stateless — istalgan serverga so'rov yuborsa bo'ladi. Lekin WebSocket/SSE/long-poll **uzun yashovchi, stateful ulanishlar** — bu horizontal scaling'ni qiyinlashtiradi.

### Muammo 1 — Connection state va horizontal scaling

Har bir client aniq **bitta** server'ga ulanadi (ulanish o'sha serverda "yashaydi"). Foydalanuvchi A — `Server-1`'ga, B — `Server-2`'ga ulangan bo'lsa, A yuborgan xabar `Server-2`'dagi B'ga qanday yetadi?

```text
   A ──ws──► [Server-1]      [Server-2] ◄──ws── B
                  │               │
                  └─── A → B xabarini qanday yetkazish? ───┘
```

### Yechim — Pub/Sub bilan fan-out

Serverlar o'rtasida **message broker** (`Redis Pub/Sub`, `Kafka`, `NATS`) qo'yiladi. Har bir server xabarni broker'ga e'lon qiladi (publish), boshqa serverlar obuna (subscribe) bo'lgani uchun oladi va o'z client'lariga uzatadi (fan-out).

```text
   A ──► [Server-1] ──publish──► ┌─────────────┐
                                 │ Redis Pub/Sub│
   B ◄── [Server-2] ◄─subscribe──└─────────────┘
```

### Muammo 2 — Sticky session

WebSocket handshake'dan keyingi barcha frame'lar **bitta** TCP ulanishi orqali o'tadi. Load balancer client'ni har safar boshqa serverga yuborsa, ulanish buziladi. Shuning uchun **sticky session** (session affinity) kerak — bir client doim bir serverga yo'naltiriladi.

```text
   Load Balancer
        │  ← cookie/IP hash bo'yicha
        ├──► Server-1  (A doim shu yerga)
        └──► Server-2  (B doim shu yerga)
```

**⚠️ Ehtiyot bo'l:** Boshqa scaling muammolari:
- **Connection limit**: har bir server cheklangan ochiq ulanish (file descriptor, xotira) ko'tara oladi — minglab/millionlab ulanish uchun maxsus tuning kerak.
- **Graceful shutdown / deploy**: serverni qayta ishga tushirsangiz barcha ulanishlar uziladi — client'lar bir vaqtda qayta ulanishga urinib **thundering herd** yaratadi. Sekin-asta drain qilish kerak.
- **Backpressure**: client ma'lumotni qabul qilishdan sekinroq bo'lsa, server buferi to'lib ketadi.

---

## Savol-javoblar

### ❓ HTTP request-response modeli real-time uchun nega yetarli emas?

**✅ Javob:** Chunki kommunikatsiyani **faqat client boshlay oladi** — server o'zidan ulanish ochib client'ga ma'lumot "push" qila olmaydi. Yangi voqea (xabar, narx o'zgarishi) server tomonda sodir bo'lsa-da, client so'ramaguncha server uni yetkaza olmaydi. Bu cheklov polling, SSE, WebSocket kabi yechimlarni keltirib chiqargan.

### ❓ Short polling va long polling farqi nimada?

**✅ Javob:** **Short polling**'da server so'rovga darhol javob qaytaradi (ma'lumot bo'lmasa ham bo'sh) va client interval bilan takror so'raydi. **Long polling**'da server ma'lumot tayyor bo'lguncha (yoki timeout) javobni **ushlab turadi**, ma'lumot kelishi bilan qaytaradi. Long polling kechikishi kamroq va bo'sh so'rovlar kamroq, lekin server resurslarini uzoqroq band qiladi.

### ❓ SSE qanday ishlaydi va qachon ishlatiladi?

**✅ Javob:** SSE bitta `HTTP` ulanishini ochiq saqlab, server'dan client'ga **bir tomonlama** matn oqimini yuboradi (`Content-Type: text/event-stream`). Brauzerda `EventSource` API ishlatiladi va u **avtomatik qayta ulanadi**. Faqat server → client push yetarli bo'lganda ideal: notifications, live feed, log streaming, LLM javoblarini token-token oqimlash.

### ❓ WebSocket handshake qanday kechadi?

**✅ Javob:** WebSocket ulanishi oddiy `HTTP GET` so'rovi sifatida boshlanadi, lekin `Upgrade: websocket` va `Connection: Upgrade` header'lari bilan. Server roziligini `101 Switching Protocols` bilan bildiradi va `Sec-WebSocket-Accept` qaytaradi. Shundan keyin bir xil TCP ulanishi ustida HTTP emas, balki **WebSocket frame'lari** ikki tomonga oqa boshlaydi. Bu mavjud HTTP/HTTPS portlari (80/443) orqali ishlash imkonini beradi.

### ❓ SSE va WebSocket orasidan qaysi birini tanlaysiz?

**✅ Javob:** Agar faqat **server → client** push kerak bo'lsa (yangiliklar, dashboard, AI streaming) — **SSE**, chunki u oddiyroq, HTTP ustida ishlaydi va avto-reconnect bor. Agar **ikki tomonlama**, past kechikishli, tez-tez almashinuv kerak bo'lsa (chat, o'yin, kollaboratsiya) — **WebSocket**. Binary ma'lumot kerak bo'lsa ham WebSocket.

### ❓ HTTP/2 multiplexing nima va u nimani hal qiladi?

**✅ Javob:** Multiplexing — bitta TCP ulanishida bir nechta so'rov/javobni **parallel** (interleaved frame'lar bilan) yuborish. Bu HTTP/1.1'dagi **application-layer head-of-line blocking**'ni yechadi: endi sekin so'rov ortidagilarni bloklamaydi va brauzer 6 ta alohida ulanish ochishi shart emas.

### ❓ HTTP/2 HOL blocking'ni to'liq yechganmi?

**✅ Javob:** Yo'q. HTTP/2 **application-layer** HOL blocking'ni yechadi, lekin **TCP-layer** HOL blocking qoladi. TCP tartibni kafolatlagani uchun, bitta paket yo'qolsa — undan keyingi barcha ma'lumot (turli stream'lar bo'lsa ham) qayta uzatilguncha kutadi. Buni **HTTP/3 (QUIC)** yechadi, chunki QUIC'da stream'lar mustaqil.

### ❓ QUIC nima va u TCP'dan qanday afzal?

**✅ Javob:** `QUIC` — `UDP` ustiga qurilgan transport protokol bo'lib, ishonchlilik, tartib va shifrlashni (`TLS 1.3`) o'zi ta'minlaydi. TCP'dan afzalliklari: (1) **stream-level mustaqillik** — TCP HOL blocking yo'q; (2) **tez handshake** — TLS transport'ga qo'shilgani uchun 1-RTT yoki qayta ulanishda 0-RTT; (3) **connection migration** — ulanish IP emas, `Connection ID`'ga bog'langan, tarmoq o'zgarsa uzilmaydi. HTTP/3 aynan QUIC ustida ishlaydi.

### ❓ gRPC'dagi to'rt streaming turini tushuntiring.

**✅ Javob:** (1) **Unary** — 1 so'rov, 1 javob (oddiy RPC); (2) **Server streaming** — 1 so'rov, javoblar oqimi (live yangilanishlar); (3) **Client streaming** — so'rovlar oqimi, 1 javob (fayl/telemetriya yuklash); (4) **Bidirectional streaming** — ikki tomon mustaqil oqim yuboradi (chat, o'yin). Bularni `HTTP/2` multiplexing mumkin qiladi.

### ❓ WebRTC nima uchun signaling server'ga muhtoj, agar u P2P bo'lsa?

**✅ Javob:** Media WebRTC'da **P2P** (to'g'ridan-to'g'ri peer'lar o'rtasida) oqsa-da, peer'lar avval bir-birini **topishi va kelishishi** kerak — bir-birining IP/port'i, kodek'lari, shifrlash kalitlari haqida ma'lumot almashish (SDP offer/answer, ICE candidate'lar). Bu almashinuv uchun alohida **signaling** kanali (odatda WebSocket) kerak. Ulanish o'rnatilgach, media to'g'ridan-to'g'ri oqadi, signaling endi shart emas.

### ❓ STUN va TURN nima farqi bor?

**✅ Javob:** Ikkisi ham NAT traversal uchun. **STUN** server peer'ga uning tashqi (public) IP/port'ini aytadi, shunda peer'lar to'g'ridan-to'g'ri P2P ulanishga urinadi — yengil va arzon. Agar NAT/firewall to'g'ridan-to'g'ri ulanishga yo'l qo'ymasa, **TURN** server media'ni o'zi orqali **relay** qiladi (proxy bo'lib) — ishonchli, lekin server resursi va kechikish qimmat. Amalda STUN avval sinaladi, ishlamasa TURN'ga tushadi.

### ❓ WebSocket ilovasini horizontal scaling qilishda asosiy muammo nima?

**✅ Javob:** Ulanishlar **stateful** va aniq bitta serverga bog'langan. Foydalanuvchilar turli serverlarga ulangani uchun, bir serverdagi xabar boshqa serverdagi foydalanuvchiga to'g'ridan-to'g'ri yeta olmaydi. Yechim — serverlar o'rtasida **Pub/Sub broker** (Redis, Kafka, NATS) qo'yib, xabarlarni publish/subscribe orqali barcha serverlarga **fan-out** qilish.

### ❓ Sticky session nima va WebSocket'da nega kerak?

**✅ Javob:** Sticky session (session affinity) — load balancer bir client'ni doim **bir xil** backend serverga yo'naltirishi. WebSocket'da handshake va undan keyingi barcha frame'lar bitta uzluksiz TCP ulanishi orqali o'tadi; agar LB client'ni boshqa serverga yuborsa, ulanish o'sha serverda yo'q va buziladi. Shuning uchun cookie yoki IP-hash bo'yicha sticky routing kerak.

### ❓ "Thundering herd" muammosi real-time'da qachon yuzaga keladi?

**✅ Javob:** Server qayta ishga tushganda (deploy, crash) barcha ochiq ulanishlar bir vaqtda uziladi va minglab client **bir vaqtda** qayta ulanishga urinadi — bu serverga keskin yuk (spike) beradi va uni yana yiqitishi mumkin. Yechim: client'larda **jitter bilan exponential backoff** (tasodifiy kechikish), serverda **connection draining** (sekin-asta o'chirish) va qayta ulanish so'rovlarini cheklash.

### ❓ Long polling serverda qaysi resurs muammosini keltiradi?

**✅ Javob:** Har bir ochiq long-poll so'rovi server ulanish (va bloklovchi serverda thread) band qiladi, chunki javob ma'lumot kelguncha ushlab turiladi. Ko'p bir vaqtli foydalanuvchida bu **connection/thread pool** tugab qolishiga olib keladi. Shuning uchun long polling asinxron, event-driven serverlarda (`Node.js`, `Go`, `nginx`) ancha yaxshi miqyoslanadi.

---

## Masalalar

> Yechimlar: [solutions/networking/07-realtime-streaming.md](../solutions/networking/07-realtime-streaming.md)

1. **Polling tahlili.** Ilovangiz har 2 sekundda short polling qiladi, 10 000 aktiv foydalanuvchi bor va so'rovlarning 95% bo'sh javob qaytaradi. Bir daqiqada qancha behuda so'rov ketadi? Bu yukni kamaytirish uchun qaysi yondashuvga o'tasiz va nega?

2. **SSE yoki WebSocket?** Quyidagi har bir holatda qaysi protokol mosligini va sababini yozing: (a) birja narxlari tickeri; (b) ikki kishilik shaxmat o'yini; (c) serverdagi log'larni jonli ko'rsatuvchi admin panel; (d) hamkorlikdagi rasm chizish doskasi.

3. **Handshake o'qish.** Quyidagi so'rovni ko'rib, bu qanday ulanish ekanini, qaysi header'lar muhimligini va server qanday status kod bilan javob berishini tushuntiring:
```text
GET /live HTTP/1.1
Host: api.example.com
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: x3JJHMbDL1EzLkh9GBhXDw==
Sec-WebSocket-Version: 13
```

4. **HOL blocking.** HTTP/2'ga o'tdingiz, lekin tarmoqda paket yo'qolishi (packet loss) yuqori bo'lgan mobil foydalanuvchilar hali ham sekinlikdan shikoyat qilmoqda. Nega HTTP/2 bu holatda yetarli emas va qaysi yechim yordam beradi? Sababini transport darajasida tushuntiring.

5. **gRPC dizayni.** Quyidagi har bir vazifa uchun mos gRPC streaming turini tanlang va `.proto` da signature yozing: (a) foydalanuvchi ID bo'yicha bitta profil olish; (b) qurilmadan sensor ma'lumotlarini uzluksiz serverga yuborish; (c) jonli sport o'yini hisobini kuzatish; (d) ikki foydalanuvchi o'rtasidagi real-time chat.

6. **Scaling arxitekturasi.** Sizda 3 ta WebSocket server orqasida load balancer bor. Foydalanuvchi A `Server-1`'da, B `Server-3`'da. A yuborgan xabarni B real-time olishi kerak. To'liq oqimni (load balancer sozlamasi + serverlararo mexanizm) diagramma bilan tasvirlang va qaysi komponentlar kerakligini sanang.

7. **HTTP versiyalari.** HTTP/1.1, HTTP/2 va HTTP/3 ni transport, multiplexing, HOL blocking va handshake tezligi bo'yicha jadval qilib taqqoslang. Keyin: nega HTTP/3 UDP ustida qurilgani paradoksal tuyulsa-da, aslida mantiqiy ekanini tushuntiring.

8. **WebRTC arxitekturasi.** Ikki brauzer o'rtasida video qo'ng'iroq ilovasini loyihalashtiryapsiz. Qaysi komponentlar kerak (signaling, STUN, TURN) va ularning har biri qaysi bosqichda ishlaydi? TURN qachon va nega ishga tushadi?

9. **Reconnect strategiyasi.** WebSocket serverlaringizni deploy qilganda 50 000 client bir vaqtda uziladi va darhol qayta ulanishga urinib serverlarni yiqitadi. Bu muammoni nima deyiladi va client hamda server tomonda qanday choralar bilan oldini olasiz?

10. **Protokol tanlash bayonnomasi.** Bir jamoa "barcha real-time ehtiyojlar uchun WebSocket ishlatamiz" deb qaror qildi, jumladan oddiy server→client notification'lar uchun ham. Bu yondashuvning kamchiliklarini ayting va qaysi holatlarda SSE yoki HTTP/2 SSE afzalroq bo'lishini asoslang.

---

← [Networking bo'limiga qaytish](./README.md)
