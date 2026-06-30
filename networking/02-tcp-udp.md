# TCP va UDP

Transport qatlamining ikki asosiy protokoli: ishonchli va tartibli TCP hamda tez va yengil UDP — ularning ichki mexanizmlari va qachon qaysi birini tanlash.

## Mundarija

- [TCP nima](#tcp-nima)
- [3-way handshake](#3-way-handshake)
- [4-way termination](#4-way-termination)
- [Sequence va Acknowledgment number](#sequence-va-acknowledgment-number)
- [Reliability: retransmission va timeout](#reliability-retransmission-va-timeout)
- [Flow control: sliding window](#flow-control-sliding-window)
- [Congestion control](#congestion-control)
- [Head-of-line blocking](#head-of-line-blocking)
- [TCP segment struktura](#tcp-segment-struktura)
- [UDP nima](#udp-nima)
- [Port tushunchasi](#port-tushunchasi)
- [TCP vs UDP taqqoslash](#tcp-vs-udp-taqqoslash)
- [Node misollari](#node-misollari)
- [Intervyu savollari](#intervyu-savollari)
- [Masalalar](#masalalar)

---

## TCP nima

**💡 Tushuncha:** TCP (Transmission Control Protocol) — `connection-oriented`, `reliable` (ishonchli) va `ordered` (tartibli) transport protokoli. Ya'ni: ma'lumot uzatishdan oldin ulanish o'rnatiladi, har bir bayt yetib borishi kafolatlanadi va qabul qiluvchiga yuborilgan tartibda yetkaziladi.

TCP ning asosiy kafolatlari:
- **Reliability:** yo'qolgan paketlar qayta yuboriladi (retransmission).
- **Ordering:** paketlar tartib raqami (sequence number) bilan tartiblanadi.
- **Error detection:** checksum orqali buzilgan ma'lumot aniqlanadi.
- **Flow control:** qabul qiluvchini bosib ketmaslik (sliding window).
- **Congestion control:** tarmoqni bosib ketmaslik (slow start, AIMD).

---

## 3-way handshake

**💡 Tushuncha:** TCP ulanishi uchma-uch ochilishidan oldin uch bosqichli "qo'l berib ko'rishish" amalga oshiriladi — bu seq raqamlarni sinxronlash uchun.

```text
   CLIENT                                   SERVER
     |                                         |
     |  1. SYN  (seq=x)                        |
     | --------------------------------------> |
     |                                         |
     |  2. SYN-ACK  (seq=y, ack=x+1)           |
     | <-------------------------------------- |
     |                                         |
     |  3. ACK  (seq=x+1, ack=y+1)             |
     | --------------------------------------> |
     |                                         |
     |  === ulanish o'rnatildi (ESTABLISHED) === |
```

1. **SYN:** client tasodifiy boshlang'ich seq raqami `x` ni yuboradi.
2. **SYN-ACK:** server o'z seq raqami `y` ni yuboradi va clientning `x+1` ni tasdiqlaydi.
3. **ACK:** client serverning `y+1` ni tasdiqlaydi.

**⚠️ Ehtiyot bo'l:** Nima uchun 2 emas, aynan 3 qadam? Chunki ikkala tomon ham bir-birining seq raqamini bilishi va tasdiqlashi kerak. 2 qadam bilan faqat bir tomon tasdiqlangan bo'lardi.

---

## 4-way termination

**💡 Tushuncha:** Ulanishni yopish 4 qadamda bo'ladi, chunki TCP `full-duplex` — har ikki yo'nalish alohida yopiladi.

```text
   CLIENT                                   SERVER
     |                                         |
     |  1. FIN  (seq=u)                        |
     | --------------------------------------> |
     |                                         |
     |  2. ACK  (ack=u+1)                      |
     | <-------------------------------------- |
     |        (server hali yuborishi mumkin)   |
     |                                         |
     |  3. FIN  (seq=v)                        |
     | <-------------------------------------- |
     |                                         |
     |  4. ACK  (ack=v+1)                      |
     | --------------------------------------> |
     |                                         |
     | client TIME_WAIT holatida biroz kutadi  |
```

**💡 Tushuncha (TIME_WAIT):** Yopishdan keyin client `2*MSL` (Maximum Segment Lifetime) davomida kutadi — kechikkan paketlar yangi ulanishni buzmasligi uchun.

---

## Sequence va Acknowledgment number

**💡 Tushuncha:** Bu ikki raqam TCP ning ishonchliligi va tartibining yuragi.

- **Sequence number:** yuborilayotgan ma'lumotning birinchi baytining raqami. Har bayt sanaladi.
- **Acknowledgment number:** "men shu raqamgacha bo'lgan hamma narsani oldim, endi shu raqamdagini kutyapman" degani.

```text
Client yuboradi:  seq=1000, data 500 bayt
Server javob:     ack=1500   (1000..1499 olindi, 1500 ni kutaman)
```

---

## Reliability: retransmission va timeout

**💡 Tushuncha:** TCP har yuborilgan segment uchun ACK kutadi. ACK belgilangan vaqtda kelmasa, segment qayta yuboriladi.

- **RTO (Retransmission Timeout):** ACK kutish vaqti. RTT (Round-Trip Time) o'lchovlari asosida dinamik hisoblanadi.
- **Fast retransmit:** agar 3 ta takroriy ACK (duplicate ACK) kelsa, timeoutni kutmasdan darhol qayta yuboradi.

```text
Client: seq=1 ---X (yo'qoldi)
Client: seq=2 ----> Server: ack=1 (men hali 1 ni kutyapman)
Client: seq=3 ----> Server: ack=1 (duplicate)
Client: seq=4 ----> Server: ack=1 (duplicate -> 3-marta)
Client: seq=1 ----> (fast retransmit, timeoutni kutmadi)
```

---

## Flow control: sliding window

**💡 Tushuncha:** Flow control — tez yuboruvchi sekin qabul qiluvchini bosib ketmasligini ta'minlaydi. Buni `receive window` (rwnd) boshqaradi: qabul qiluvchi "men hozir yana shuncha bayt qabul qila olaman" deb e'lon qiladi.

```text
Yuborilgan       Yuborilgan      Yuborilishi      Hali
va ACK olingan   ACK kutilmoqda  mumkin           mumkin emas
[####|##########|##########|.........................]
                <----- window (rwnd) ----->
```

Window suriladi: ACK kelgani sayin chap chegara o'ngga suriladi va yangi baytlar yuborishga ruxsat beriladi. Agar qabul qiluvchining buferi to'lsa, u `window=0` e'lon qilib yuboruvchini to'xtatadi.

---

## Congestion control

**💡 Tushuncha:** Flow control qabul qiluvchini himoya qiladi; congestion control esa butun **tarmoqni** bosib ketishdan himoya qiladi. Buning uchun yuboruvchi alohida `congestion window` (cwnd) saqlaydi.

**Slow start:** cwnd kichikdan boshlanadi va har RTT da eksponensial (2 barobar) o'sadi, `ssthresh` (threshold) ga yetguncha.

**Congestion avoidance:** ssthresh dan keyin o'sish chiziqli bo'ladi (har RTT da +1 MSS) — ehtiyotkorona.

**AIMD (Additive Increase, Multiplicative Decrease):** congestion (paket yo'qolishi) sezilganda cwnd keskin (masalan yarmiga) kamaytiriladi, keyin yana sekin chiziqli o'sadi.

```text
cwnd
  |                           /\          /
  |        slow start        /  \  AIMD  /
  |        (exponential)    /    \      /  (additive
  |                        /      \    /    increase)
  |    ssthresh ---------/--------\--/
  |                    /  conges- \/ (multiplicative
  |                   /   tion     decrease - yarmiga)
  |__________________/______________________________ vaqt
```

Amaldagi yuborish tezligi: `effektiv window = min(cwnd, rwnd)`.

---

## Head-of-line blocking

**💡 Tushuncha:** TCP tartibni kafolatlaganligi sababli, oldinroq yuborilgan bitta paket yo'qolsa, undan keyingi yetib kelgan paketlar ham uni kutib turishga majbur — ular bufferda "qamalib" qoladi. Bu **head-of-line (HOL) blocking**.

```text
Kelgan: [1] [X yo'qoldi] [3] [4] [5]
                  ^
            2 kelmaguncha 3,4,5 ilovaga berilmaydi
```

**⚠️ Ehtiyot bo'l:** HTTP/2 da bitta TCP ustida ko'p stream multiplekslangani uchun bitta paket yo'qolsa hamma streamlar bloklanadi (TCP-darajadagi HOL). HTTP/3 buni UDP ustidagi QUIC bilan hal qiladi.

---

## TCP segment struktura

```text
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-------------------------------+-------------------------------+
|          Source Port          |       Destination Port        |
+-------------------------------+-------------------------------+
|                        Sequence Number                        |
+---------------------------------------------------------------+
|                     Acknowledgment Number                     |
+-------+-----------+-----------+-------------------------------+
| Data  | Reserved  |U A P R S F|         Window Size           |
| Offset|           |R C S S Y I|                               |
|       |           |G K H T N N|                               |
+-------+-----------+-----------+-------------------------------+
|           Checksum            |        Urgent Pointer         |
+-------------------------------+-------------------------------+
|                    Options (ixtiyoriy)                        |
+---------------------------------------------------------------+
|                            Data                               |
+---------------------------------------------------------------+
```

Asosiy maydonlar: portlar, sequence/acknowledgment number, control flags (SYN, ACK, FIN, RST, PSH, URG), window size (flow control), checksum.

---

## UDP nima

**💡 Tushuncha:** UDP (User Datagram Protocol) — `connectionless`, `unreliable`, `fast`. Handshake yo'q, ACK yo'q, tartib kafolati yo'q, congestion control yo'q. Faqat "yubor va unut" (fire-and-forget).

UDP header atigi 8 bayt (TCP 20+ bayt), shuning uchun overhead juda kichik:

```text
+-------------------+-------------------+
|   Source Port     | Destination Port  |
+-------------------+-------------------+
|     Length        |    Checksum       |
+-------------------+-------------------+
|              Data ...                 |
+---------------------------------------+
```

**UDP use-case'lari:**
- **DNS:** kichik so'rov-javob, tezlik muhim (TCP handshake ortiqcha).
- **Video streaming:** bitta kadr yo'qolsa, uni kutgandan ko'ra keyingisini ko'rsatish yaxshiroq.
- **Online gaming:** eng so'nggi holat muhim, eski paketni qayta yuborish ma'nosiz.
- **VoIP (ovozli qo'ng'iroq):** kechikish (latency) yo'qotishdan yomonroq; kichik uzilish chidasa bo'ladi.

---

## Port tushunchasi

**💡 Tushuncha:** Port (0-65535) — bitta IP address ortidagi turli ilovalarni ajratadigan raqam. IP "uy manzili" bo'lsa, port "xonadon raqami". Ulanish to'rtlik bilan aniqlanadi: `(src IP, src port, dst IP, dst port)` + protokol.

- **Well-known portlar (0-1023):** HTTP 80, HTTPS 443, DNS 53, SSH 22.
- **Registered (1024-49151)** va **Ephemeral (49152-65535):** client tomon vaqtinchalik portlari.

---

## TCP vs UDP taqqoslash

| Xususiyat | TCP | UDP |
|-----------|-----|-----|
| Ulanish | Connection-oriented | Connectionless |
| Ishonchlilik | Reliable (ACK, retransmit) | Unreliable |
| Tartib | Ordered | Tartib yo'q |
| Tezlik | Sekinroq (overhead) | Tez |
| Header hajmi | 20+ bayt | 8 bayt |
| Flow/Congestion control | Bor | Yo'q |
| Handshake | Bor (3-way) | Yo'q |
| Use-case | Web, email, fayl | DNS, video, gaming, VoIP |

**Qachon TCP:** ma'lumotning to'liq va tartibli yetib borishi muhim bo'lganda (HTTP, fayl yuklash, bank).

**Qachon UDP:** tezlik va past latency yo'qotishdan muhimroq bo'lganda (jonli video, o'yin, DNS).

---

## Node misollari

**TCP server (`net` moduli):**

```js
const net = require('net');

const server = net.createServer((socket) => {
  console.log('Client ulandi:', socket.remoteAddress);

  socket.on('data', (data) => {
    console.log('Kelgan:', data.toString());
    socket.write('Echo: ' + data); // qaytarib yuboramiz
  });

  socket.on('end', () => console.log('Client uzildi'));
});

server.listen(4000, () => console.log('TCP server 4000-portda'));
```

**UDP (`dgram` moduli):**

```js
const dgram = require('dgram');

const server = dgram.createSocket('udp4');

server.on('message', (msg, rinfo) => {
  console.log(`UDP xabar ${rinfo.address}:${rinfo.port} dan: ${msg}`);
  server.send('Javob: ' + msg, rinfo.port, rinfo.address);
});

server.bind(5000, () => console.log('UDP server 5000-portda'));

// Client tomon:
const client = dgram.createSocket('udp4');
client.send('Salom UDP', 5000, '127.0.0.1');
```

**⚠️ Ehtiyot bo'l:** TCP `socket.write()` da yozilgan ma'lumot "stream" — bitta `write` bir nechta `data` event sifatida kelishi yoki bir nechta `write` birga kelishi mumkin (TCP "message boundary" saqlamaydi). UDP esa har bir datagram alohida xabar sifatida keladi.

---

## Intervyu savollari

### ❓ TCP va UDP ning asosiy farqi nima?

**✅ Javob:** TCP connection-oriented, reliable, ordered — handshake, ACK, retransmission, flow/congestion control bilan. UDP connectionless, unreliable, fast — handshake va kafolatlar yo'q, header atigi 8 bayt. TCP to'liqlik kerak bo'lganda, UDP esa past latency va tezlik muhim bo'lganda ishlatiladi.

### ❓ 3-way handshake nima uchun aynan 3 qadam?

**✅ Javob:** Har ikki tomon bir-birining boshlang'ich sequence number ini bilishi va tasdiqlashi kerak. Client SYN yuboradi (o'z seq ini), server SYN-ACK bilan o'z seq ini yuborib clientnikini tasdiqlaydi, client ACK bilan serverning seq ini tasdiqlaydi. 2 qadam bilan bir tomon tasdiqlanmagan qolardi.

### ❓ Nima uchun ulanish ochilishi 3, yopilishi 4 qadam?

**✅ Javob:** Ochilishda server SYN va ACK ni bitta segmentda (SYN-ACK) birlashtira oladi. Yopilishda esa TCP full-duplex bo'lgani uchun har yo'nalish alohida yopiladi: bir tomon FIN yuborgach, ikkinchi tomon hali ma'lumot yuborishi mumkin, shuning uchun ACK va o'z FIN i alohida keladi.

### ❓ Sequence va acknowledgment number nima vazifa bajaradi?

**✅ Javob:** Sequence number — yuborilayotgan baytlarni raqamlaydi, tartibni va yo'qotishni aniqlash imkonini beradi. Acknowledgment number — "shu raqamgacha hammasini oldim, keyingisini kutyapman" degani; bu retransmission va tartibni boshqaradi.

### ❓ Flow control va congestion control farqi?

**✅ Javob:** Flow control — qabul qiluvchining buferini bosib ketmaslik (receive window, rwnd). Congestion control — butun tarmoqni bosib ketmaslik (congestion window, cwnd; slow start, AIMD). Effektiv yuborish tezligi `min(rwnd, cwnd)`.

### ❓ Slow start qanday ishlaydi?

**✅ Javob:** Ulanish boshida cwnd kichik (masalan 1-10 MSS) bo'ladi va har RTT da eksponensial 2 barobar o'sadi, ssthresh ga yetguncha. Bu tarmoq sig'imini ehtiyotkorlik bilan "paypaslab" topish uchun. ssthresh dan keyin congestion avoidance (chiziqli o'sish) boshlanadi.

### ❓ AIMD nima?

**✅ Javob:** Additive Increase Multiplicative Decrease — congestion avoidance siyosati. Hammasi yaxshi bo'lsa cwnd har RTT da chiziqli (+1 MSS) o'sadi; paket yo'qolishi (congestion) sezilsa cwnd keskin (masalan yarmiga) kamaytiriladi. Bu tarmoqning adolatli va barqaror ishlashini ta'minlaydi.

### ❓ Head-of-line blocking nima?

**✅ Javob:** TCP tartibni kafolatlaganligi uchun, oldinroq yuborilgan bitta paket yo'qolsa, undan keyin yetib kelgan paketlar bufferda kutib turishga majbur — ilovaga berilmaydi. HTTP/2 da bitta TCP ulanishida ko'p stream bo'lgani uchun bir paket yo'qolishi hammasini bloklaydi; HTTP/3 buni QUIC (UDP ustida) bilan hal qiladi.

### ❓ Retransmission qachon ishga tushadi?

**✅ Javob:** Ikki holatda: (1) RTO (timeout) tugaganda ACK kelmasa; (2) fast retransmit — 3 ta duplicate ACK kelganda timeoutni kutmasdan darhol qayta yuboriladi. RTO RTT o'lchovlari asosida dinamik hisoblanadi.

### ❓ Nima uchun DNS odatda UDP ishlatadi?

**✅ Javob:** DNS so'rovlari kichik (bitta datagramga sig'adi) va tezlik muhim. TCP handshake qo'shimcha RTT qo'shadi — bu DNS uchun ortiqcha. Lekin javob katta bo'lsa (yoki zone transfer) DNS TCP ga o'tadi.

### ❓ Video streaming nega ko'pincha UDP/QUIC ishlatadi?

**✅ Javob:** Jonli oqimda kechikish (latency) yo'qotishdan yomonroq. Bitta kadr yo'qolsa, uni qayta yuborib kutgandan ko'ra, keyingi kadrni ko'rsatish yaxshiroq. TCP retransmission va HOL blocking jonli oqimda "buffering" va kechikish keltiradi.

### ❓ Port nima va ulanish qanday aniqlanadi?

**✅ Javob:** Port (0-65535) — bitta IP ortidagi turli ilovalarni ajratadi. Bitta TCP ulanish 5 ta narsa bilan unikal aniqlanadi: protokol, manba IP, manba port, manzil IP, manzil port. Shuning uchun bitta server porti (masalan 443) bir vaqtda minglab clientga xizmat qila oladi.

### ❓ TCP "stream" bo'lishi nimani anglatadi?

**✅ Javob:** TCP ma'lumotni baytlar oqimi sifatida ko'radi, message chegarasini saqlamaydi. Bitta `write` qabul tomonda bir nechta `data` event sifatida bo'linishi yoki bir nechta `write` birga kelishi mumkin. Shuning uchun ilova darajasida o'z framing (uzunlik prefiksi yoki ajratuvchi) kerak. UDP esa har datagramni alohida xabar sifatida saqlaydi.

### ❓ TIME_WAIT holati nima uchun kerak?

**✅ Javob:** Ulanish yopilgach, faol yopuvchi tomon `2*MSL` davomida TIME_WAIT da kutadi. Bu kechikkan/takroriy paketlar shu port juftligida ochiladigan yangi ulanishga aralashib ketmasligi va oxirgi ACK yo'qolsa qayta yuborilishi mumkin bo'lishi uchun.

### ❓ UDP da ishonchlilik kerak bo'lsa nima qilinadi?

**✅ Javob:** Ishonchlilik ilova darajasida quriladi — masalan QUIC (HTTP/3 ostida) UDP ustida o'z sequence, ACK, retransmission va congestion control ini amalga oshiradi, lekin TCP ning HOL blocking muammosisiz. Ya'ni UDP "bo'sh asos" bo'lib, ustiga kerakli kafolatlar maxsus quriladi.

---

## Masalalar

> Yechimlar: [../solutions/networking/02-tcp-udp.md](../solutions/networking/02-tcp-udp.md)

1. 3-way handshake ni `seq=100` (client) va `seq=300` (server) qiymatlari bilan to'liq diagrammada chizing va har bir segmentdagi seq/ack qiymatlarini yozing.
2. Quyidagi ssenariy uchun TCP yoki UDP tanlang va sababini yozing: (a) bank o'tkazmasi, (b) jonli futbol translyatsiyasi, (c) DNS so'rovi, (d) fayl yuklash, (e) ko'p o'yinchili shooter.
3. Client `seq=5000` dan boshlab 3000 bayt yubordi. Server qaysi `ack` qiymatini qaytaradi? Keyin client yana 1500 bayt yuborsa-chi?
4. Fast retransmit ssenariysini chizing: 5 ta segment yuborilib, 2-chisi yo'qolgan holatda server qanday ACK lar yuboradi va client qachon qayta yuboradi.
5. Slow start va congestion avoidance fazalarini cwnd vaqt grafigida chizing, ssthresh nuqtasini belgilang va paket yo'qolganda nima bo'lishini (AIMD) ko'rsating.
6. HTTP/2 da head-of-line blocking nima uchun yuzaga keladi va HTTP/3 (QUIC) buni qanday hal qilishini tushuntiring (diagramma bilan).
7. `net` moduli yordamida shunday TCP server yozing-ki, u kelgan har bir qatorni teskari aylantirib (reverse) qaytarsin. Message boundary muammosini qanday hal qilasiz?
8. `dgram` yordamida UDP "ping" client-server yozing: client xabar yuboradi, server timestamp bilan javob qaytaradi, client RTT ni hisoblaydi.
9. Receive window 0 ga tushganda yuboruvchi nima qiladi? Window yana ochilganini qanday biladi? (zero-window va window probe haqida yozing.)
10. Bitta serverda 443-portda 10 000 ta bir vaqtdagi ulanish bo'lishi mumkinmi? Ulanish qanday unikal aniqlanishini 4-lik/5-lik orqali asoslang.

---

← [Networking bo'limiga qaytish](./README.md)
