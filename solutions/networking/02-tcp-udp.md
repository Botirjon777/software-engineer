# TCP va UDP — Yechimlar

Quyida `02-tcp-udp.md` masalalari uchun to'liq yechimlar.

---

### 1-masala — 3-way handshake (seq=100, seq=300)

```text
   CLIENT (seq=100)                         SERVER (seq=300)
     |                                         |
     |  1. SYN  seq=100                        |
     | --------------------------------------> |
     |                                         |
     |  2. SYN-ACK  seq=300, ack=101           |
     | <-------------------------------------- |
     |                                         |
     |  3. ACK  seq=101, ack=301               |
     | --------------------------------------> |
     |        === ESTABLISHED ===              |
```

Izoh: SYN bayrog'i 1 ta "fantom bayt" sanaladi, shuning uchun ack = qabul qilingan seq + 1. Client 100 ni yubordi -> server ack=101. Server 300 ni yubordi -> client ack=301.

---

### 2-masala — TCP yoki UDP tanlash

```text
(a) Bank o'tkazmasi          -> TCP  (to'liqlik va tartib hayotiy muhim)
(b) Jonli futbol translyatsiyasi -> UDP (past latency > yo'qotish; kadr yo'qolsa o'tib ketadi)
(c) DNS so'rovi              -> UDP  (kichik, tez; katta javobda TCP ga o'tadi)
(d) Fayl yuklash             -> TCP  (har bayt to'g'ri yetishi shart)
(e) Multiplayer shooter      -> UDP  (eng so'nggi holat muhim, eski paket keraksiz)
```

---

### 3-masala — ack qiymatlari

```text
Client seq=5000, data 3000 bayt -> baytlar 5000..7999 ni egallaydi
Server javob: ack = 8000   (8000 dan keyingisini kutaman)

Client yana 1500 bayt: seq=8000, baytlar 8000..9499
Server javob: ack = 9500
```

---

### 4-masala — fast retransmit ssenariysi

```text
Client yuboradi:               Server javobi:
  seg1 (seq=1)    ----------->  ack=1001   (seg1 olindi, keyingini kutaman)
  seg2 (seq=1001) ----X         (yo'qoldi)
  seg3 (seq=2001) ----------->  ack=1001   (duplicate #1 - hali 1001 kutyapman)
  seg4 (seq=3001) ----------->  ack=1001   (duplicate #2)
  seg5 (seq=4001) ----------->  ack=1001   (duplicate #3)

3 ta duplicate ACK kelgach:
  seg2 (seq=1001) ----------->  (FAST RETRANSMIT, timeoutni kutmadi)
                                ack=5001   (endi hammasi to'liq)
```

Izoh: 3 ta duplicate ACK "oradan bittasi yo'qolgan, lekin keyingilar yetib kelmoqda" signali. Shuning uchun RTO ni kutmasdan darhol qayta yuboriladi.

---

### 5-masala — cwnd grafigi va AIMD

```text
cwnd (MSS)
  |
16|                              .          /
  |                            /   \       /
 8|  ssthresh ........... /-----\----/---- (multiplicative
  |                      /  conges- \/      decrease: 16 -> 8)
 4|                     / (avoidance, +1/RTT)
  |     slow start     /
 2|     (x2/RTT)      /
 1|____/_____________/_______________________________ vaqt
       ^             ^                  ^
   start         ssthresh ga      paket yo'qoldi:
                 yetdi (chiziqli)  cwnd yarmiga, ssthresh yangilanadi
```

Fazalar: (1) slow start — eksponensial, ssthresh gacha; (2) congestion avoidance — chiziqli (+1 MSS/RTT); (3) paket yo'qolishi — AIMD bilan cwnd keskin kamayadi (multiplicative decrease), ssthresh yangi qiymatga o'rnatiladi, keyin yana o'sish boshlanadi.

---

### 6-masala — HTTP/2 HOL va HTTP/3 yechimi

```text
HTTP/2 (bitta TCP ulanish, ko'p stream multiplekslangan):
  TCP oqimi: [s1][s2 X yo'qoldi][s1][s3][s2]...
  TCP TARTIBNI kafolatlaydi -> s2 ning yo'qolgan paketi kelmaguncha
  s1 va s3 ning KEYINGI baytlari ham ilovaga BERILMAYDI.
  => bitta paket yo'qolishi HAMMA streamni bloklaydi (TCP-level HOL).

HTTP/3 (QUIC, UDP ustida):
  Har stream MUSTAQIL. QUIC stream-darajada tartib saqlaydi.
  s2 ning paketi yo'qolsa, faqat s2 kutadi; s1 va s3 davom etadi.
  => stream-lar bir-birini bloklamaydi.
```

Sabab: TCP butun ulanishni bitta tartibli oqim deb biladi; QUIC esa streamlarni alohida tartiblaydi, shuning uchun bir streamdagi yo'qotish boshqalarni to'xtatmaydi.

---

### 7-masala — qatorni teskari aylantiradigan TCP server

```js
const net = require('net');

const server = net.createServer((socket) => {
  let buffer = '';

  socket.on('data', (chunk) => {
    buffer += chunk.toString();
    let idx;
    // '\n' bo'yicha to'liq qatorlarni ajratamiz (framing)
    while ((idx = buffer.indexOf('\n')) !== -1) {
      const line = buffer.slice(0, idx);
      buffer = buffer.slice(idx + 1);
      const reversed = line.split('').reverse().join('');
      socket.write(reversed + '\n');
    }
  });
});

server.listen(4000, () => console.log('Reverse server 4000-portda'));
```

**Message boundary muammosi:** TCP stream bo'lgani uchun bitta `data` event to'liq qator bermasligi yoki bir nechta qatorni birga berishi mumkin. Yechim — buferda to'plab, `\n` ajratuvchisi bo'yicha to'liq qatorlarni kesib olish (line framing). Binar protokolda esa odatda uzunlik prefiksi ishlatiladi.

---

### 8-masala — UDP ping (RTT o'lchash)

```js
const dgram = require('dgram');

// SERVER
const server = dgram.createSocket('udp4');
server.on('message', (msg, rinfo) => {
  // kelgan timestampni o'zgartirmasdan qaytaramiz
  server.send(msg, rinfo.port, rinfo.address);
});
server.bind(5000, () => console.log('UDP ping server 5000-portda'));

// CLIENT
const client = dgram.createSocket('udp4');
const sentAt = Date.now();
client.send(String(sentAt), 5000, '127.0.0.1');

client.on('message', (msg) => {
  const rtt = Date.now() - Number(msg.toString());
  console.log('RTT =', rtt, 'ms');
  client.close();
});

// timeout (UDP ishonchsiz - javob kelmasligi mumkin)
setTimeout(() => {
  console.log('Javob kelmadi (timeout)');
  client.close();
}, 2000).unref();
```

Izoh: client yuborayotgan timestampni server o'zgartirmasdan qaytaradi; client `now - sent` ni hisoblab RTT topadi. UDP ishonchsiz bo'lgani uchun timeout qo'shildi.

---

### 9-masala — zero-window va window probe

Qabul qiluvchi buferi to'lsa, `window=0` e'lon qiladi va yuboruvchi yangi ma'lumot yuborishni **to'xtatadi**.

Yuboruvchi window yana ochilganini bilish uchun **window probe** (persist timer) ishlatadi: vaqti-vaqti bilan kichik (1 bayt) probe segment yuboradi va qabul qiluvchining yangi window e'lonini kutadi. Shu orqali, agar window ochilgani haqidagi xabar yo'qolgan bo'lsa ham, ulanish "muzlab" qolmaydi (deadlock oldini olish).

---

### 10-masala — bir portda 10 000 ulanish

Ha, mumkin. TCP ulanish bitta port bilan emas, balki **5-lik (5-tuple)** bilan unikal aniqlanadi:

```text
(protokol, manba IP, manba port, manzil IP, manzil port)
```

Server tomonda manzil IP va manzil port (443) bir xil bo'lsa ham, har bir client **boshqa manba IP va/yoki manba port** bilan keladi. Shuning uchun 5-lik har ulanishda farq qiladi va bitta server porti minglab-millionlab parallel ulanishga xizmat qila oladi. Amaliy chegara port emas, balki xotira, fayl deskriptorlari (ulimit) va CPU.

---

← [Networking bo'limiga qaytish](../../networking/README.md)
