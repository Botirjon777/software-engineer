# Tarmoq Asoslari — Yechimlar

Quyida `01-network-fundamentals.md` masalalari uchun to'liq yechimlar.

---

### 1-masala — URL ni qismlarga ajratish

`https://news.example.com/article?id=5`

```text
https              -> scheme (protokol): qaysi protokol ishlatiladi (HTTPS = shifrlangan HTTP)
news               -> subdomain: hostning bo'lagi (masalan yangiliklar bo'limi)
example.com        -> domain: ro'yxatdan o'tgan asosiy nom
news.example.com   -> host (FQDN): DNS orqali IP ga aylantiriladigan to'liq nom
(default port 443) -> port: ko'rsatilmagani uchun HTTPS uchun 443 olinadi
/article           -> path: serverdagi resurs yo'li
?id=5              -> query string: serverga qo'shimcha parametr (id=5)
```

Har qism nima uchun kerak: scheme protokolni, host qaysi serverga borishni, port qaysi ilovaga ulanishni, path qaysi resursni, query esa resursning qaysi variantini (filtr/parametr) belgilaydi.

---

### 2-masala — OSI 7 qatlam + misollar

```text
L7 Application   -> HTTP, FTP, DNS, SMTP
L6 Presentation  -> TLS/SSL, JPEG, ASCII/UTF-8, siqish
L5 Session       -> sessiya boshqaruvi, RPC, NetBIOS
L4 Transport     -> TCP, UDP (port, segment)
L3 Network       -> IP, ICMP; qurilma: router
L2 Data Link     -> Ethernet, ARP; qurilma: switch
L1 Physical      -> kabel, radio signal; qurilma: hub
```

---

### 3-masala — qurilmalarni qatlamga joylashtirish

```text
Hub    -> L1 (Physical): faqat elektr signalni takrorlaydi, hech qanday adresga qaramaydi.
Switch -> L2 (Data Link): MAC address jadvalini saqlab, frameni to'g'ri portga yuboradi.
Router -> L3 (Network):  IP address bo'yicha turli tarmoqlar orasida marshrutlaydi.
```

Sabab: qurilma qaysi ma'lumotga (signal / MAC / IP) qarab qaror qabul qilsa, o'sha qatlamga tegishli bo'ladi.

---

### 4-masala — encapsulation diagrammasi

```text
L7 Application:  [ GET / HTTP/1.1 ]
                          |
L4 Transport:    [ TCP hdr (src 54321, dst 443, seq) | GET / HTTP/1.1 ]   <- Segment
                          |
L3 Network:      [ IP hdr (src IP -> dst IP) | TCP hdr | GET / ... ]       <- Packet
                          |
L2 Data Link:    [ Eth hdr (src MAC -> gw MAC) | IP | TCP | GET / | CRC ]  <- Frame
                          |
L1 Physical:     0101000111010101...                                       <- Bits
```

Har qatlam o'z headerini old tomonga (va L2 da CRC ni oxiriga) qo'shadi.

---

### 5-masala — switch MAC jadvali bo'sh holatda

**Birinchi frame (jadval bo'sh):** switch manba MAC ni o'sha port bilan jadvalga yozib oladi (learning). Manzil MAC noma'lum bo'lgani uchun frameni **barcha portlarga** (kelgan portdan tashqari) yuboradi — bu `flooding`.

**Keyingi framelar:** javob qaytganda switch ikkinchi qurilmaning MAC ini ham o'rganadi. Endi manzil MAC jadvalda bo'lgani uchun frame faqat **bitta to'g'ri portga** yuboriladi (forwarding). Vaqt o'tishi bilan jadval to'lib, flooding kamayadi.

---

### 6-masala — yuklab olish vaqti

Bandwidth = 50 Mbps = 50 / 8 = 6.25 MB/s.

```text
Fayl = 10 MB
Nazariy minimal vaqt = 10 MB / 6.25 MB/s = 1.6 soniya
```

**Amalda nega ko'proq:** TCP slow start boshida sekin yuboradi; protokol overhead (TCP/IP/Ethernet headerlar) foydali yukni kamaytiradi; latency va RTT; tarmoq congestion; server tezligi; DNS/TCP/TLS handshakelar qo'shimcha vaqt oladi. Shuning uchun real throughput har doim bandwidthdan past.

---

### 7-masala — LAN dan WAN serverga yo'l

```text
[Ishchi PC] --frame(src MAC=PC, dst MAC=Router)--> [Switch] --> [Router/Gateway]
   IP packet: src IP = PC IP, dst IP = server IP (o'zgarmaydi)

[Router] --NAT--> [ISP/WAN] --> ... --> [Server]
   Har "hop" da:  IP (manba/manzil)  ODATDA o'zgarmaydi (NAT bo'lmasa)
                  MAC esa HAR hop da yangilanadi (joriy qurilma -> keyingi qurilma)
```

Asosiy nuqta: **IP address uchma-uch (end-to-end)** saqlanadi (NAT bo'lsa manba IP almashadi), **MAC address esa har segmentda** keyingi qurilmaga moslab o'zgaradi. ARP har hop da keyingi MAC ni topadi.

---

### 8-masala — client-server vs P2P video qo'ng'iroq

**Client-server (markaziy media server):**
- (+) Boshqaruv oson, sifat/yozib olish markazda; (+) NAT/firewall ortidagi qurilmalar uchun ishonchli; (+) ko'p ishtirokchini server mikslaydi.
- (-) Server resurs/trafik nuqtasi; (-) qo'shimcha latency (hammasi serverdan o'tadi); (-) server qulasa hammasi to'xtaydi.

**P2P (to'g'ridan-to'g'ri):**
- (+) Past latency (to'g'ridan-to'g'ri yo'l); (+) server xarajati kam; (+) single point of failure yo'q.
- (-) NAT traversal murakkab (STUN/TURN kerak); (-) ko'p ishtirokchida har kim hammaga yuborishi og'ir; (-) boshqaruv/moderatsiya qiyin.

Amalda ko'p ilovalar gibrid: 1:1 da P2P, guruhda SFU/server.

---

### 9-masala — 8 qadam va protokollar

```text
1. URL parse          -> (brauzer ichki)
2. DNS resolution     -> DNS
3. TCP connection     -> TCP (3-way handshake)
4. TLS handshake      -> TLS (HTTPS bo'lsa)
5. HTTP request       -> HTTP
6. Server processing  -> (server-side)
7. HTTP response      -> HTTP
8. Render / close     -> (brauzer) + TCP (close yoki keep-alive)
```

---

### 10-masala — bir xil tarmoqda router kerakmi?

`192.168.1.0/24` ichidagi ikki qurilma **bitta subnetda**, shuning uchun **router kerak emas**.

Yo'l: yuboruvchi manzil IP `/24` maskasi bilan o'zi bilan bir tarmoqda ekanini aniqlaydi -> ARP orqali manzil IP uchun MAC ni so'raydi -> frameni to'g'ridan-to'g'ri o'sha MAC ga (switch orqali) yuboradi. Router (default gateway) faqat boshqa tarmoqqa (subnetdan tashqariga) chiqishda ishlatiladi.

---

← [Networking bo'limiga qaytish](../../networking/README.md)
