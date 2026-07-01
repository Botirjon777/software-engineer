# Kompyuter Arxitekturasi

Kompyuter qanday qilib elektr signallaridan foydalanib dasturlarni bajarishini va uning asosiy komponentlari qanday hamkorlik qilishini o'rganamiz.

## Mundarija

- [Kompyuterning asosiy komponentlari](#komponentlar)
- [von Neumann arxitekturasi](#von-neumann)
- [Harvard arxitekturasi](#harvard)
- [CPU-RAM-Storage o'zaro bog'lanishi](#bog-lanish)
- [Instruction cycle: fetch-decode-execute-writeback](#instruction-cycle)
- [Machine code va instruction](#machine-code)
- [Dastur RAM'ga yuklanib qanday bajariladi](#dastur-bajarilishi)
- [Abstraksiya qatlamlari](#abstraksiya)
- [Binary va nega](#binary)
- [Savol-javob](#savol-javob)
- [Masalalar](#masalalar)

<a name="komponentlar"></a>
## Kompyuterning asosiy komponentlari

Har qanday zamonaviy kompyuter bir nechta asosiy komponentdan tashkil topgan. Ularning har biri o'z vazifasini bajaradi, lekin birgalikda ishlaydi.

**💡 Tushuncha:** Kompyuterni oshxona deb tasavvur qiling. CPU — oshpaz (hisob-kitob qiladi), RAM — ish stoli (hozir ishlatilayotgan narsalar), storage — muzlatgich (uzoq muddat saqlash), motherboard — oshxonaning o'zi (hamma narsani bog'laydi).

| Komponent | To'liq nomi | Vazifasi |
|-----------|-------------|----------|
| **CPU** | Central Processing Unit | Barcha hisob-kitob va instruksiyalarni bajaradi (kompyuterning "miyasi") |
| **RAM** | Random Access Memory | Hozir ishlatilayotgan dastur va ma'lumotlarni vaqtincha saqlaydi (tez, lekin volatile) |
| **Storage** | SSD / HDD | Ma'lumotlarni uzoq muddat saqlaydi (sekin, lekin doimiy) |
| **Motherboard** | Ona plata | Barcha komponentlarni bir-biriga ulaydi, bus'lar orqali ma'lumot uzatadi |
| **GPU** | Graphics Processing Unit | Grafika va parallel hisob-kitoblarni bajaradi (minglab kichik core) |
| **PSU** | Power Supply Unit | Elektr tokini komponentlar uchun mos kuchlanishga aylantiradi |
| **I/O** | Input/Output | Tashqi dunyo bilan aloqa (klaviatura, monitor, tarmoq, disk) |

```text
                    ┌──────────────────────────────────┐
                    │           MOTHERBOARD            │
                    │                                  │
   ┌────────┐       │   ┌───────┐        ┌───────┐     │
   │  PSU   │──────►│   │  CPU  │◄──────►│  RAM  │     │
   │ (tok)  │       │   └───┬───┘        └───────┘     │
   └────────┘       │       │                          │
                    │       │            ┌───────┐     │
   ┌────────┐       │       ├───────────►│  GPU  │────►│──► Monitor
   │  I/O   │◄─────►│       │            └───────┘     │
   │ klav.  │       │       │                          │
   └────────┘       │       │            ┌─────────┐   │
                    │       └───────────►│ Storage │   │
                    │                    │ SSD/HDD │   │
                    └────────────────────┴─────────┴───┘
```

**⚠️ Ehtiyot bo'l:** RAM va storage'ni chalkashtirmang. RAM *volatile* — tok o'chsa hamma narsa yo'qoladi. Storage *non-volatile* — tok o'chsa ham saqlanadi. Shuning uchun ishingizni saqlamasangiz (storage'ga yozmasangiz) va tok o'chsa, u yo'qoladi.

<a name="von-neumann"></a>
## von Neumann arxitekturasi

1945-yilda matematik John von Neumann taklif qilgan model bugungi deyarli barcha kompyuterlar asosini tashkil qiladi.

Asosiy g'oya — **stored-program** konsepsiyasi: dastur (instruksiyalar) ham, ma'lumot (data) ham **bir xil xotirada** saqlanadi. Bu inqilobiy edi, chunki oldin dasturni o'zgartirish uchun simlarni fizik ravishda qayta ulash kerak edi.

**💡 Tushuncha:** Stored-program degani — kompyuterga aytadigan buyruqlaringiz ham xuddi oddiy raqamlar kabi xotirada yotadi. Kompyuter uchun "5 + 3 ni qo'sh" degan instruksiya ham, "5" va "3" raqamlari ham bir xil joyda — xotirada — turadigan raqamlardir.

```text
   von Neumann arxitekturasi (bo'linmagan xotira):

        ┌─────────────────────────────┐
        │           CPU               │
        │   ┌──────┐    ┌──────────┐  │
        │   │ ALU  │    │ Control  │  │
        │   └──────┘    │  Unit    │  │
        │               └──────────┘  │
        └──────────────┬──────────────┘
                       │  BITTA bus (yagona yo'l)
                       │
        ┌──────────────┴──────────────┐
        │          XOTIRA (RAM)        │
        │  ┌────────────┬───────────┐  │
        │  │ Instruksiya│  Ma'lumot │  │  ← ikkalasi ham
        │  │  (dastur)  │  (data)   │  │     bir xotirada
        │  └────────────┴───────────┘  │
        └─────────────────────────────┘
```

Bitta xotira va bitta bus'ning kamchiligi bor: CPU bir vaqtda yo instruksiyani, yo ma'lumotni o'qiy oladi, ikkalasini birdan emas. Bu **von Neumann bottleneck** deb ataladi.

<a name="harvard"></a>
## Harvard arxitekturasi

Harvard arxitekturasida instruksiya va ma'lumot **alohida xotiralarda** va **alohida bus'larda** saqlanadi. Bu CPU'ga bir vaqtning o'zida instruksiyani ham, ma'lumotni ham o'qish imkonini beradi.

```text
   Harvard arxitekturasi (alohida xotiralar):

                  ┌──────────┐
                  │   CPU    │
                  └──┬────┬──┘
       instruction   │    │   data bus
          bus        │    │
        ┌────────────┘    └────────────┐
   ┌────┴────────┐              ┌───────┴──────┐
   │ Instruksiya │              │   Ma'lumot   │
   │   xotirasi  │              │   xotirasi   │
   └─────────────┘              └──────────────┘
```

| Xususiyat | von Neumann | Harvard |
|-----------|-------------|---------|
| Xotira | Bitta (birlashgan) | Ikkita (alohida) |
| Bus | Bitta | Ikkita |
| Instruksiya + data bir vaqtda o'qish | Yo'q | Ha |
| Soddaligi / narxi | Sodda, arzon | Murakkab, qimmat |
| Qayerda | Umumiy PC/server | Mikrokontroller, DSP, CPU cache |

**💡 Tushuncha:** Amalda zamonaviy CPU'lar ikkalasini aralashtiradi: tashqi ko'rinishda von Neumann (bitta RAM), lekin ichkarida cache darajasida instruction va data alohida saqlanadi (masalan L1i va L1d cache) — bu "modified Harvard" deb ataladi.

<a name="bog-lanish"></a>
## CPU-RAM-Storage o'zaro bog'lanishi

Bu uch komponent tezlik va hajm bo'yicha ierarxiya (memory hierarchy) tashkil qiladi. Yuqoriroq — tezroq va qimmatroq, pastroq — sekinroq va arzonroq.

```text
   TEZ      ┌──────────────────┐   Hajm: KB       Narx: qimmat
    ▲       │   CPU Registers  │   Kechikish: ~0.3 ns
    │       ├──────────────────┤
    │       │   L1 / L2 / L3    │   Hajm: KB–MB
    │       │      Cache        │   Kechikish: ~1–40 ns
    │       ├──────────────────┤
    │       │       RAM         │   Hajm: GB
    │       │   (main memory)   │   Kechikish: ~100 ns
    │       ├──────────────────┤
    │       │   SSD Storage     │   Hajm: TB
    ▼       │   (yoki HDD)      │   Kechikish: ~100 µs (SSD)
   SEKIN    └──────────────────┘   Narx: arzon
```

Dastur ishlaganda ma'lumot pastdan yuqoriga ko'chadi: storage → RAM → cache → register. CPU faqat register va cache bilan tez ishlay oladi, shuning uchun kerakli ma'lumotni oldindan RAM'dan cache'ga olib kelish muhim.

**⚠️ Ehtiyot bo'l:** RAM'dan bitta so'zni o'qish CPU register'idan o'qishga qaraganda ~300 baravar sekinroq. Shuning uchun "cache miss" (kerakli ma'lumot cache'da yo'q) dastur tezligiga katta ta'sir qiladi.

<a name="instruction-cycle"></a>
## Instruction cycle: fetch-decode-execute-writeback

CPU har bir instruksiyani bir xil takrorlanuvchi tsikl orqali bajaradi. Bu tsikl sekundiga milliardlab marta takrorlanadi.

```text
   ┌──────────┐   ┌──────────┐   ┌──────────┐   ┌───────────┐
   │  FETCH   │──►│  DECODE  │──►│ EXECUTE  │──►│ WRITEBACK │──┐
   └──────────┘   └──────────┘   └──────────┘   └───────────┘  │
        ▲                                                       │
        └───────────────────────────────────────────────────────┘
             (keyingi instruksiya uchun qaytadan boshlanadi)
```

**1. Fetch (olib kelish):** Control Unit `PC` (Program Counter) register'i ko'rsatgan manzildan instruksiyani RAM'dan olib keladi va uni `IR` (Instruction Register)'ga joylaydi. Keyin PC keyingi instruksiyaga o'tadi.

**2. Decode (ochish):** Control Unit instruksiyaning binary kodini "tushunadi" — bu qanday amal (qo'shish? o'qish? sakrash?) va qaysi operand'lar (registerlar, xotira manzillari) ishlatiladi.

**3. Execute (bajarish):** ALU (Arithmetic Logic Unit) haqiqiy amalni bajaradi — masalan ikki sonni qo'shadi yoki taqqoslaydi.

**4. Writeback (yozib qo'yish):** Natija register'ga yoki xotiraga yoziladi.

**💡 Tushuncha:** Bu tsikl to'xtovsiz aylanadi — kompyuter yoqilgandan o'chguncha. Fetch → decode → execute → writeback → yana fetch... Har bir "aylanish" bir yoki bir necha clock cycle davom etadi.

<a name="machine-code"></a>
## Machine code va instruction

**Instruction (instruksiya)** — CPU bajara oladigan eng oddiy amal: "ikki sonni qo'sh", "xotiradan o'qi", "shu manzilga sakra". CPU'da faqat cheklangan sondagi instruksiya turlari bor — bu ISA (Instruction Set Architecture)'da belgilangan.

**Machine code** — bu instruksiyalarning binary (0 va 1) ko'rinishi. CPU faqat shu tilni "tushunadi".

```text
   Yozgan kodimiz (yuqori til):
       c = a + b

   Assembly (odam o'qiy oladigan machine code):
       LOAD  R1, a       ; a ni R1 register'ga o'qi
       LOAD  R2, b       ; b ni R2 register'ga o'qi
       ADD   R3, R1, R2  ; R1 + R2 = R3
       STORE c, R3       ; R3 ni c ga yoz

   Machine code (haqiqiy binary, CPU shuni ko'radi):
       10110000 00000001
       10110001 00000010
       00011010 00110011
       10100000 00000011
```

**💡 Tushuncha:** Odam machine code'ni to'g'ridan-to'g'ri yozmaydi. Biz Python, C, JavaScript kabi yuqori tillarda yozamiz, keyin compiler yoki interpreter uni machine code'ga aylantiradi.

<a name="dastur-bajarilishi"></a>
## Dastur RAM'ga yuklanib qanday bajariladi

Dasturni ikki marta bosishdan (yoki ishga tushirishdan) to natijagacha bo'lgan yo'lni ko'raylik.

```text
   1. Dastur storage'da (SSD) fayl sifatida yotadi
                    │
                    ▼
   2. OS dasturni RAM'ga yuklaydi (instruksiyalar + data)
                    │
                    ▼
   3. OS CPU'ning PC register'ini dasturning birinchi
      instruksiyasi manziliga o'rnatadi
                    │
                    ▼
   4. CPU instruction cycle'ni boshlaydi:
      fetch → decode → execute → writeback → fetch → ...
                    │
                    ▼
   5. Har bir instruksiya register/cache/RAM bilan ishlaydi
                    │
                    ▼
   6. Natija I/O orqali chiqadi (ekranga, faylga, tarmoqqa)
```

Masalan, `2 + 3` ni hisoblaydigan dastur:
1. Fayl SSD'da turadi.
2. OS uni RAM'ga yuklaydi.
3. CPU `LOAD 2`, `LOAD 3`, `ADD`, `STORE`, `PRINT` instruksiyalarini birma-bir bajaradi.
4. Natija `5` ekranga chiqadi.

**⚠️ Ehtiyot bo'l:** Dastur RAM'dan ishlaydi, storage'dan emas. Storage sekin bo'lgani uchun CPU to'g'ridan-to'g'ri undan instruksiya o'qimaydi — avval RAM'ga ko'chiriladi.

<a name="abstraksiya"></a>
## Abstraksiya qatlamlari

Kompyuter — bir necha abstraksiya qatlamining ustma-ust joylashuvi. Har bir qatlam pastdagisiga suyanadi, lekin uning ichki tafsilotlarini yashiradi.

```text
   ┌─────────────────────────────────────┐  ← eng yuqori (odamga yaqin)
   │  Dastur (Python, C, sizning kodingiz)│
   ├─────────────────────────────────────┤
   │  Til / Compiler / Interpreter        │
   ├─────────────────────────────────────┤
   │  Operatsion tizim (OS)               │
   ├─────────────────────────────────────┤
   │  ISA (Instruction Set Architecture)  │  ← software/hardware chegarasi
   ├─────────────────────────────────────┤
   │  CPU (mikroarxitektura)              │
   ├─────────────────────────────────────┤
   │  Logic gate (AND, OR, NOT)           │
   ├─────────────────────────────────────┤
   │  Transistor                          │
   └─────────────────────────────────────┘  ← eng past (fizikaga yaqin)
```

**💡 Tushuncha:** Abstraksiya tufayli Python yozayotganingizda transistorlar haqida o'ylash shart emas. Har bir qatlam pastdagi murakkablikni yashiradi. Bu — informatikaning eng kuchli g'oyalaridan biri.

<a name="binary"></a>
## Binary va nega

Kompyuterlar hamma narsani **binary** (0 va 1) bilan ifodalaydi. Nega o'nlik (decimal) emas?

Sabab — fizik. Transistor faqat ikki holatda bo'la oladi: tok bor (1) yoki tok yo'q (0). Ikki holatni ishonchli ajratish oson va xatoga chidamli. Agar 10 xil kuchlanish darajasi ishlatilsa, shovqin (noise) tufayli xatolik ko'p bo'lardi.

```text
   Transistor holatlari:
       Yuqori kuchlanish (~5V) → 1
       Past kuchlanish  (~0V)  → 0

   Decimal → Binary:
       0 → 0000        5 → 0101
       1 → 0001        8 → 1000
       2 → 0010       10 → 1010
       3 → 0011       15 → 1111
```

**💡 Tushuncha:** 1 bit = 1 ta ikkilik raqam (0 yoki 1). 8 bit = 1 bayt. 1 bayt bilan 2^8 = 256 xil qiymat ifodalanadi (0–255).

<a name="savol-javob"></a>
## Savol-javob

### ❓ CPU va RAM o'rtasidagi asosiy farq nima?

**✅ Javob:** CPU — hisob-kitob qiluvchi komponent (instruksiyalarni bajaradi), RAM — vaqtincha ma'lumot saqlaydigan xotira. CPU o'zi ma'lumot saqlamaydi (juda oz register'dan tashqari), RAM esa hisob-kitob qilmaydi. Ular birga ishlaydi: CPU RAM'dan instruksiya va data o'qiydi, natijani RAM'ga qaytaradi.

### ❓ Nega dastur storage'dan emas, RAM'dan ishlaydi?

**✅ Javob:** Storage (SSD/HDD) RAM'ga qaraganda yuzlab yoki minglab marta sekinroq. Agar CPU har bir instruksiyani storage'dan o'qisa, kompyuter juda sekin bo'lardi. Shuning uchun OS dasturni avval tez RAM'ga ko'chiradi, keyin CPU RAM'dan ishlaydi.

### ❓ Stored-program konsepsiyasi nima?

**✅ Javob:** von Neumann g'oyasi — dastur (instruksiyalar) ham xuddi ma'lumot kabi xotirada saqlanadi. Bu dasturni o'zgartirishni oson qildi: endi simlarni qayta ulash emas, faqat xotiradagi raqamlarni o'zgartirish yetarli.

### ❓ von Neumann bottleneck nima?

**✅ Javob:** von Neumann arxitekturasida instruksiya va data bitta bus orqali o'tadi. CPU bir vaqtda faqat bittasini o'qiy oladi, shuning uchun bus tezligi butun tizim tezligini cheklaydi — bu "bottleneck" (qisilish nuqtasi).

### ❓ von Neumann va Harvard arxitekturasi qaysi holatda ishlatiladi?

**✅ Javob:** von Neumann — umumiy maqsadli kompyuterlar (PC, server), sodda va arzon. Harvard — mikrokontroller, DSP va tezlik muhim joylarda, chunki instruksiya va datani bir vaqtda o'qish mumkin. Zamonaviy CPU'lar cache darajasida "modified Harvard" ishlatadi.

### ❓ Instruction cycle'ning to'rt bosqichi qanday?

**✅ Javob:** Fetch (instruksiyani RAM'dan olib kelish), Decode (uni ochish/tushunish), Execute (ALU'da bajarish), Writeback (natijani yozish). Bu tsikl to'xtovsiz takrorlanadi.

### ❓ Program Counter (PC) register nima qiladi?

**✅ Javob:** PC keyingi bajariladigan instruksiyaning xotiradagi manzilini saqlaydi. Har fetch'dan keyin PC keyingi instruksiyaga o'tadi (yoki sakrash instruksiyasi uni boshqa manzilga o'zgartiradi).

### ❓ Machine code va assembly farqi nima?

**✅ Javob:** Machine code — sof binary (0/1), CPU to'g'ridan-to'g'ri shuni bajaradi. Assembly — machine code'ning odam o'qiy oladigan matnli ko'rinishi (masalan `ADD R1, R2`). Assembler assembly'ni machine code'ga aylantiradi.

### ❓ Abstraksiya qatlamlari nega muhim?

**✅ Javob:** Har bir qatlam pastdagi murakkablikni yashiradi. Python dasturchisi transistor haqida o'ylamaydi, CPU dizayneri esa Python haqida o'ylamaydi. Bu murakkab tizimlarni boshqarish imkonini beradi.

### ❓ Nega kompyuterlar binary ishlatadi?

**✅ Javob:** Transistor faqat ikki ishonchli holatda bo'la oladi (tok bor/yo'q). Ikki holatni shovqindan ajratish oson va xatoga chidamli. 10 xil kuchlanish darajasi ishlatilsa, xatolik ko'p bo'lardi.

### ❓ Motherboard nima vazifa bajaradi?

**✅ Javob:** Motherboard barcha komponentlarni (CPU, RAM, GPU, storage, I/O) fizik va elektr jihatdan bir-biriga ulaydi. Bus'lar orqali ular o'rtasida ma'lumot uzatiladi.

### ❓ Memory hierarchy (xotira ierarxiyasi) nima?

**✅ Javob:** Xotiraning tezlik va hajm bo'yicha darajalari: register (eng tez, eng kichik) → cache → RAM → storage (eng sekin, eng katta). Yuqoriroq darajalar qimmatroq, shuning uchun kichik. Bu ierarxiya tezlik va narx o'rtasidagi muvozanatni ta'minlaydi.

### ❓ GPU va CPU farqi nima?

**✅ Javob:** CPU'da kam sonli kuchli core bor — ketma-ket (sequential) vazifalarga yaxshi. GPU'da minglab kichik core bor — parallel vazifalarga (grafika, matritsa hisob-kitobi, ML) juda yaxshi. Har biri boshqa turdagi ishga moslashgan.

### ❓ RAM volatile bo'lishi nimani anglatadi?

**✅ Javob:** Volatile — tok o'chsa ma'lumot yo'qoladi. RAM elektr toki bilan ma'lumotni ushlab turadi. Storage esa non-volatile — tok o'chsa ham saqlanadi. Shuning uchun ishingizni storage'ga saqlash muhim.

<a name="masalalar"></a>
## Masalalar

> Yechimlar: [solutions/hardware/01-computer-architecture.md](../solutions/hardware/01-computer-architecture.md)

1. Bir dastur `x = 10`, `y = 20`, `z = x + y` ni hisoblaydi. Ushbu dasturni bajarish uchun taxminan qancha instruksiya kerak bo'ladi va ularni ketma-ketligini assembly ko'rinishida yozing (LOAD/ADD/STORE).

2. von Neumann va Harvard arxitekturasining afzallik va kamchiliklarini jadval ko'rinishida taqqoslang. Har biri qaysi holatda afzalroq ekanini tushuntiring.

3. Quyidagi decimal sonlarni binary'ga aylantiring: 7, 12, 25, 64, 100. Har birida nechta bit kerakligini ko'rsating.

4. Instruction cycle'ning to'rt bosqichini `a + b` amali misolida qadama-qadam yozing: har bosqichda qaysi register va komponent ishtirok etadi?

5. Memory hierarchy'da register RAM'dan ~300 baravar tez deb faraz qilaylik. Agar bir amal register'dan 1 ns olsa, RAM'dan qancha vaqt oladi? Nega bu farq dastur tezligiga muhim?

6. Abstraksiya qatlamlarini transistordan dasturgacha tartib bilan sanab bering va har birini bir jumlada tushuntiring.

7. Nega kompyuter decimal (10 lik) emas, binary (2 lik) sistemani ishlatadi? Kamida ikkita fizik sabab keltiring.

8. 1 bayt bilan nechta xil qiymat ifodalanadi? 2 bayt bilan-chi? 4 bayt bilan? Formulani yozing.

← [Hardware bo'limiga qaytish](./README.md)
