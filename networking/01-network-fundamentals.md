# Tarmoq Asoslari

Internet qanday ishlaydi — brauzerga URL kiritilgandan to javob kelguncha bo'lgan butun yo'l, OSI va TCP/IP modellari, packet/frame/segment va qatlamlar bo'ylab oqim.

## Mundarija

- [Internet qanday ishlaydi](#internet-qanday-ishlaydi)
- [Client-server va Peer-to-peer](#client-server-va-peer-to-peer)
- [OSI 7-layer model](#osi-7-layer-model)
- [TCP/IP 4-layer model](#tcpip-4-layer-model)
- [Packet, Frame, Segment](#packet-frame-segment)
- [Encapsulation va De-encapsulation](#encapsulation-va-de-encapsulation)
- [MAC address va IP address](#mac-address-va-ip-address)
- [Switch, Router, Hub](#switch-router-hub)
- [LAN va WAN](#lan-va-wan)
- [Bandwidth va Protocol](#bandwidth-va-protocol)
- [HTTP so'rovining qatlamlar bo'ylab harakati](#http-sorovining-qatlamlar-boylab-harakati)
- [Intervyu savollari](#intervyu-savollari)
- [Masalalar](#masalalar)

---

## Internet qanday ishlaydi

**💡 Tushuncha:** Internet — bu bir-biriga ulangan millionlab tarmoqlardan iborat global tarmoq. Hech qanday "markaziy server" yo'q; ma'lumotlar `packet` (paket) shaklida marshrutlanib (routing) bir kompyuterdan ikkinchisiga yetib boradi.

Brauzerga `https://example.com` kiritganingizdan javob kelguncha bo'lgan umumiy yo'l:

```text
1. URL parse        -> brauzer URL ni qismlarga ajratadi (scheme, host, path)
2. DNS resolution   -> "example.com" -> IP address (masalan 93.184.216.34)
3. TCP connection   -> IP ga 3-way handshake orqali ulanish ochiladi
4. TLS handshake    -> HTTPS bo'lsa shifrlangan kanal o'rnatiladi
5. HTTP request     -> "GET / HTTP/1.1" so'rovi yuboriladi
6. Server processing-> server javobni tayyorlaydi (HTML, JSON, ...)
7. HTTP response    -> "200 OK" + body qaytadi
8. Rendering        -> brauzer HTML ni parse qilib sahifani chizadi
9. Connection close -> ulanish yopiladi yoki keep-alive bilan saqlanadi
```

Har bir qadam pastki qatlamlardagi protokollarga tayanadi: IP marshrutlash, Ethernet/Wi-Fi fizik uzatish, TCP ishonchli yetkazib berish.

---

## Client-server va Peer-to-peer

**💡 Tushuncha:** Bu ikkalasi — tarmoq arxitekturasining ikki asosiy modeli.

```text
CLIENT-SERVER:                      PEER-TO-PEER (P2P):

   [Client]   [Client]                [Peer] ---- [Peer]
       \        /                        |   \    /   |
        \      /                         |    \  /    |
       [ Server ]                        |     \/     |
       (markaziy)                      [Peer] ---- [Peer]
                                       (har biri ham client ham server)
```

| Xususiyat | Client-Server | Peer-to-Peer |
|-----------|---------------|--------------|
| Markaz | Bor (server) | Yo'q |
| Boshqaruv | Oson, markazlashgan | Murakkab, taqsimlangan |
| Masshtablash | Server cheklovi bor | Tabiiy masshtablanadi |
| Misol | Web (HTTP), email | BitTorrent, blockchain |
| Single point of failure | Bor | Yo'q |

---

## OSI 7-layer model

**💡 Tushuncha:** OSI (Open Systems Interconnection) — tarmoq aloqasini 7 qatlamga ajratadigan konseptual model. Amalda to'liq ishlatilmaydi, lekin o'rganish va muammoni lokalizatsiya qilish uchun standart "til".

```text
+-----+--------------+---------------------------------+-------------------+
| #   | Layer        | Vazifa                          | Misol             |
+-----+--------------+---------------------------------+-------------------+
| L7  | Application  | Ilova protokollari              | HTTP, FTP, DNS    |
| L6  | Presentation | Shifrlash, kodlash, siqish      | TLS, JPEG, UTF-8  |
| L5  | Session      | Sessiya boshqaruvi              | sessiya, RPC      |
| L4  | Transport    | Uchma-uch yetkazish, port       | TCP, UDP          |
| L3  | Network      | Marshrutlash, IP adres          | IP, ICMP, router  |
| L2  | Data Link    | Frame, MAC adres                | Ethernet, switch  |
| L1  | Physical     | Bit, kabel, signal              | kabel, radio, hub |
+-----+--------------+---------------------------------+-------------------+
```

Eslab qolish uchun (yuqoridan pastga): **A**ll **P**eople **S**eem **T**o **N**eed **D**ata **P**rocessing.

---

## TCP/IP 4-layer model

**💡 Tushuncha:** Internet aslida TCP/IP modeli ustida ishlaydi. U OSI ning soddalashtirilgan, amaliy versiyasi.

```text
TCP/IP (4 layer)        OSI (7 layer) bilan moslik
+------------------+    +----------------------------------+
| Application      | =  | Application + Presentation + Session (L5-L7) |
+------------------+    +----------------------------------+
| Transport        | =  | Transport (L4)                   |
+------------------+    +----------------------------------+
| Internet         | =  | Network (L3)                     |
+------------------+    +----------------------------------+
| Link (Network    | =  | Data Link + Physical (L1-L2)     |
| Access)          |    |                                  |
+------------------+    +----------------------------------+
```

**⚠️ Ehtiyot bo'l:** Intervyuda "OSI da nechta qatlam?" deb so'rashsa — 7. "Internet qaysi modelda ishlaydi?" deb so'rashsa — TCP/IP (4 qatlam). Ularni aralashtirib yubormang.

---

## Packet, Frame, Segment

**💡 Tushuncha:** Bir xil ma'lumot turli qatlamlarda har xil nom oladi. Bu nomlar Protocol Data Unit (PDU) deyiladi.

```text
+-----------+----------------------+------------------------+
| Layer     | PDU nomi             | Nima qo'shiladi        |
+-----------+----------------------+------------------------+
| Transport | Segment (TCP)        | port, seq, ack         |
|           | Datagram (UDP)       | port                   |
| Network   | Packet               | source/dest IP         |
| Data Link | Frame                | source/dest MAC, CRC   |
| Physical  | Bits                 | 0 va 1 (signal)        |
+-----------+----------------------+------------------------+
```

---

## Encapsulation va De-encapsulation

**💡 Tushuncha:** Encapsulation — ma'lumot pastki qatlamga tushganda har bir qatlam o'z header (sarlavha) sini qo'shadi. De-encapsulation — qabul qiluvchida teskari jarayon: har bir qatlam o'z headerini olib tashlaydi.

```text
YUBORUVCHI (encapsulation, yuqoridan pastga):

  Application:  [ Data ]
  Transport:    [ TCP header | Data ]                 <- Segment
  Network:      [ IP header | TCP header | Data ]     <- Packet
  Data Link:    [ Eth header | IP | TCP | Data | CRC ]<- Frame
  Physical:     0101110100101...                      <- Bits

QABUL QILUVCHI (de-encapsulation, pastdan yuqoriga):
  teskari yo'nalishda har bir header o'qiladi va olib tashlanadi.
```

---

## MAC address va IP address

**💡 Tushuncha:** Ikkalasi ham adres, lekin har xil qatlamda va har xil maqsadda.

| Xususiyat | MAC address | IP address |
|-----------|-------------|------------|
| Qatlam | L2 (Data Link) | L3 (Network) |
| Format | `00:1A:2B:3C:4D:5E` (48-bit) | `192.168.1.10` (IPv4) |
| Doimiylik | Qurilmaga "kuyilgan", o'zgarmaydi | Tarmoqqa qarab o'zgaradi |
| Ko'lam | Lokal segment ichida | Global (marshrutlanadi) |
| Beruvchi | Ishlab chiqaruvchi (NIC) | Tarmoq/DHCP |

**💡 Tushuncha:** Analogiya — IP address bu "uy manzili" (shaharma-shahar yo'l topiladi), MAC address bu "pasport raqami" (lokal segmentda aniq qurilmani aniqlaydi). IP va MAC ni bog'lash uchun **ARP** protokoli ishlatiladi.

---

## Switch, Router, Hub

**💡 Tushuncha:** Uchalasi ham qurilmalarni ulaydi, lekin har xil "aqlga" ega va har xil qatlamda ishlaydi.

```text
+--------+-------+----------------------------+----------------------------+
| Qurilma| Layer | Qaror nimaga asoslanadi    | Xususiyat                  |
+--------+-------+----------------------------+----------------------------+
| Hub    | L1    | Hech narsaga (broadcast)   | Hammaga uzatadi, "ahmoq"   |
| Switch | L2    | MAC address                | Aniq portga uzatadi        |
| Router | L3    | IP address                 | Tarmoqlararo marshrutlash  |
+--------+-------+----------------------------+----------------------------+
```

- **Hub:** kelgan signalni hamma portga ko'r-ko'rona takrorlaydi (collision ko'p). Bugun deyarli ishlatilmaydi.
- **Switch:** MAC jadvalini saqlaydi, frameni faqat kerakli portga yuboradi. LAN ichida ishlaydi.
- **Router:** turli tarmoqlarni (LAN<->WAN, internet) bog'laydi, IP bo'yicha marshrutlaydi.

---

## LAN va WAN

**💡 Tushuncha:** Tarmoqlar geografik ko'lamiga qarab tasniflanadi.

| Xususiyat | LAN (Local Area Network) | WAN (Wide Area Network) |
|-----------|--------------------------|-------------------------|
| Ko'lam | Bino, ofis, uy | Shahar, davlat, dunyo |
| Tezlik | Yuqori (1-10 Gbps) | Pastroq |
| Egasi | Bitta tashkilot | Ko'p provayder (ISP) |
| Misol | Ofis Wi-Fi | Internet |

---

## Bandwidth va Protocol

**💡 Tushuncha (Bandwidth):** Bandwidth — kanal orqali vaqt birligida o'tkazilishi mumkin bo'lgan maksimal ma'lumot hajmi (masalan 100 Mbps). Bu "quvur diametri"ga o'xshaydi. **Throughput** esa amalda o'tgan haqiqiy hajm (har doim bandwidthdan kichik yoki teng). **Latency** — bitta paketning yetib borish vaqti — bu boshqa o'lcham.

**💡 Tushuncha (Protocol):** Protocol — qurilmalar bir-birini tushunishi uchun kelishilgan qoidalar to'plami (format, tartib, javob qoidalari). HTTP, TCP, IP — bularning barchasi protokol.

---

## HTTP so'rovining qatlamlar bo'ylab harakati

Bitta `GET /index.html` so'rovi qatlamlar bo'ylab qanday harakatlanadi:

```text
BROWSER (client)                              SERVER

L7 Application: GET /index.html HTTP/1.1
       |  (encapsulation pastga)
L4 Transport:   TCP segment qo'shiladi
                (src port 54321, dst port 443, seq)
       |
L3 Network:     IP packet qo'shiladi
                (src IP -> dst IP 93.184.216.34)
       |
L2 Data Link:   Ethernet frame qo'shiladi
                (src MAC -> gateway MAC)
       |
L1 Physical:    bitlar kabel/Wi-Fi orqali ketadi
       |
       +----> [router] -> [router] -> ... -> server NIC
                                                 |
                                          de-encapsulation
                                          (pastdan yuqoriga)
                                                 |
                                          L7: HTTP so'rov o'qiladi
                                                 |
                                          javob teskari yo'l bilan
                                          qaytadi: 200 OK + HTML
```

---

## Intervyu savollari

### ❓ Brauzerga URL kiritganingizdan keyin nima sodir bo'ladi?

**✅ Javob:** Ketma-ketlik: (1) brauzer URL ni parse qiladi; (2) DNS orqali domen IP ga aylantiriladi (cache -> resolver -> root -> TLD -> authoritative); (3) IP ga TCP 3-way handshake bilan ulanish ochiladi; (4) HTTPS bo'lsa TLS handshake; (5) HTTP request yuboriladi; (6) server javobni qaytaradi; (7) brauzer HTML/CSS/JS ni parse qilib render qiladi va kerak bo'lsa qo'shimcha resurslar (rasm, skript) uchun yangi so'rovlar yuboradi. Bu savol deyarli har bir tizim suhbatida beriladi — qatlamlarni eslab qoling.

### ❓ OSI va TCP/IP modellari farqi nimada?

**✅ Javob:** OSI — 7 qatlamli konseptual o'quv modeli. TCP/IP — 4 qatlamli amaliy model bo'lib, internet aynan shunda ishlaydi. TCP/IP ning Application qatlami OSI ning yuqori 3 qatlamini (L5-L7), Link qatlami esa OSI ning pastki 2 qatlamini (L1-L2) birlashtiradi.

### ❓ MAC address va IP address farqi?

**✅ Javob:** MAC — L2 dagi fizik adres, qurilmaga kuyilgan, lokal segmentda ishlaydi. IP — L3 dagi mantiqiy adres, tarmoqqa qarab o'zgaradi, global marshrutlanadi. IP bilan "qaysi tarmoq/uy", MAC bilan "shu segmentda aniq qaysi qurilma" aniqlanadi. Bog'lash uchun ARP.

### ❓ Switch va router orasidagi farq nima?

**✅ Javob:** Switch — L2 qurilma, frameni MAC address bo'yicha bitta LAN ichida to'g'ri portga uzatadi. Router — L3 qurilma, IP address bo'yicha turli tarmoqlar orasida marshrutlaydi (masalan uy tarmog'idan internetga). Ko'p uy routerlarida ikkalasi (router + switch + Wi-Fi AP) bitta qutida bo'ladi.

### ❓ Encapsulation nima?

**✅ Javob:** Ma'lumot pastki qatlamlarga tushganda har bir qatlam o'z headerini (Transport: TCP header, Network: IP header, Data Link: Ethernet header + CRC) qo'shadi. Qabul qiluvchida teskari jarayon — de-encapsulation — har qatlam o'z headerini o'qib olib tashlaydi.

### ❓ Packet, frame va segment farqi?

**✅ Javob:** Bir xil ma'lumotning turli qatlamlardagi nomi. Transport qatlamida — segment (TCP) yoki datagram (UDP); Network qatlamida — packet; Data Link qatlamida — frame; Physical qatlamida — bits.

### ❓ Hub nima uchun zamonaviy tarmoqlarda ishlatilmaydi?

**✅ Javob:** Hub L1 da ishlaydi va kelgan signalni hamma portga ko'r-ko'rona takrorlaydi. Bu collision (to'qnashuv) va keraksiz trafikni ko'paytiradi, xavfsizlik ham past (hamma hammani "eshitadi"). Switch esa frameni faqat kerakli portga yuboradi, shuning uchun hubni almashtirdi.

### ❓ Client-server va P2P qachon ishlatiladi?

**✅ Javob:** Client-server — markazlashgan boshqaruv va izchillik kerak bo'lganda (web ilovalar, bank). P2P — markazsiz, masshtablanadigan va sensorga chidamli tizim kerak bo'lganda (fayl almashish, blockchain, ba'zi video qo'ng'iroqlar). P2P da single point of failure yo'q, lekin boshqaruv murakkab.

### ❓ Bandwidth va throughput farqi?

**✅ Javob:** Bandwidth — kanalning nazariy maksimal sig'imi (masalan 100 Mbps). Throughput — amalda erishilgan haqiqiy uzatish tezligi (har doim bandwidthdan kichik yoki teng, chunki overhead, congestion, latency ta'sir qiladi). Bandwidth "quvur diametri", throughput esa "haqiqatda oqayotgan suv".

### ❓ Protocol nima va nega kerak?

**✅ Javob:** Protocol — qurilmalar bir-birini tushunishi uchun kelishilgan qoidalar (xabar formati, tartibi, javob qoidalari). Umumiy "til" bo'lmasa, turli ishlab chiqaruvchilarning qurilmalari aloqa qila olmaydi. HTTP, TCP, IP, DNS — barchasi protokol.

### ❓ Transport qatlami nima vazifa bajaradi?

**✅ Javob:** Uchma-uch (end-to-end) yetkazib berishni ta'minlaydi va port raqamlari orqali bir xost ichidagi turli ilovalarni ajratadi. TCP — ishonchli, tartibli yetkazish; UDP — tez, ishonchsiz yetkazish. Bu qatlam segmentlash, oqim nazorati (TCP) bilan shug'ullanadi.

### ❓ LAN va WAN farqi?

**✅ Javob:** LAN — kichik geografik hudud (ofis, uy), yuqori tezlik, bitta tashkilot egaligida. WAN — keng hudud (shahar/davlat/dunyo), pastroq tezlik, ko'p provayderni o'z ichiga oladi. Internet — eng katta WAN.

### ❓ ARP nima ish qiladi?

**✅ Javob:** ARP (Address Resolution Protocol) — ma'lum IP address uchun mos MAC addressni topadi. Frame yuborishdan oldin qurilma "Bu IP kimda?" deb broadcast qiladi; egasi o'z MAC ini javob qiladi. Bu L3 (IP) va L2 (MAC) ni bog'laydigan ko'prik.

### ❓ Nima uchun qatlamli model foydali?

**✅ Javob:** Modullik beradi — har qatlam mustaqil rivojlanadi (masalan Wi-Fi yoki kabel L1-L2 da o'zgarsa, yuqori qatlamlar o'zgarmaydi). Muammoni lokalizatsiya qilish osonlashadi (qaysi qatlamda nosozlik bor?). Standartlashtirish turli ishlab chiqaruvchilar mahsulotlarining birga ishlashini ta'minlaydi.

### ❓ Bir HTTP so'rovida nechta TCP ulanish bo'lishi mumkin?

**✅ Javob:** HTTP/1.1 da keep-alive bilan bitta TCP ulanishida ketma-ket bir nechta so'rov yuborilishi mumkin, lekin brauzerlar parallellik uchun bir hostga bir nechta ulanish ochadi (odatda 6 tagacha). HTTP/2 da bitta ulanishda multiplexing orqali parallel so'rovlar bo'ladi. Bitta sahifa odatda o'nlab resurs uchun ko'p so'rov yuboradi.

---

## Masalalar

> Yechimlar: [../solutions/networking/01-network-fundamentals.md](../solutions/networking/01-network-fundamentals.md)

1. `https://news.example.com/article?id=5` URL ni qismlarga ajrating (scheme, host, subdomain, path, query) va har qaysi qism nima uchun kerakligini yozing.
2. OSI ning 7 qatlamini tartib bilan yozing va har biriga bittadan protokol/qurilma misol keltiring.
3. Quyidagi qurilmalarni mos qatlamga joylashtiring: hub, switch, router. Nima uchun shu qatlamda ekanini tushuntiring.
4. Encapsulation jarayonini `GET /` so'rovi misolida har bir qatlamda qanday header qo'shilishini ko'rsatib chizing (ASCII).
5. Bitta uy tarmog'ida 3 ta qurilma bor. Switch MAC jadvali bo'sh holatda birinchi frame kelganda nima qiladi? Keyingi framelarda-chi?
6. Bandwidth 50 Mbps bo'lgan kanalda 10 MB faylni yuklab olish nazariy minimal vaqtini hisoblang. Amalda nega ko'proq vaqt ketishi mumkin?
7. Kompaniya ofisida (LAN) ishchi internetdagi (WAN) serverga ulanmoqda. Paket qaysi qurilmalardan o'tib, har birida qaysi adres (MAC/IP) ishlatilishini diagrammada ko'rsating.
8. Client-server va P2P arxitekturani video qo'ng'iroq ilovasi uchun taqqoslang: har birining afzalligi va kamchiligini 3 tadan yozing.
9. Brauzerga URL kiritilgandan javob kelguncha bo'lgan 8 qadamni yozing va har bir qadamga DNS, TCP, TLS, HTTP dan qaysi biri tegishli ekanini belgilang.
10. Bir xil `192.168.1.0/24` tarmoqdagi ikkita qurilma o'zaro ma'lumot uzatganda router kerakmi yoki yo'qmi? Javobni MAC/IP va ARP orqali asoslang.

---

← [Networking bo'limiga qaytish](./README.md)
