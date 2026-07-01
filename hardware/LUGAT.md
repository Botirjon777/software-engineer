# Hardware — Lug'at

Bu lug'at kompyuter apparat qismi (hardware) sohasidagi asosiy ingliz terminlarni o'zbek tilida sodda tushuntiradi. Termin ingliz tilida qoladi, izoh esa "bu nima degani" tarzida beriladi. Terminlar alfavit tartibida joylashtirilgan.

| Termin | Ma'nosi (o'zbekcha) |
|--------|---------------------|
| address bus | Xotiradаги qaysi manzилга murojaat qilishни tashuvchi simlar to'plami. CPU shu orqali kerakли joyni ko'rsatadi. |
| ALU | "Arithmetic Logic Unit" — CPU ичидаги arifmetik va mantiqiy amallarни bajaruvchi qism. Qo'shish, taqqoslash kabi ishlarни qiladi. |
| Amdahl's law | Dasturни parallellashtirishдан olinadigan tezlanishning nazariy chegarasi. Ketma-ket bajariladigan qism umumiy foydани cheklaydi. |
| branch prediction | CPU keyingi qaysi buyruq bajarilishini oldindan taxmin qilishi. To'g'ri taxmin qilsa, kutmasdan ishlashда davom etadi. |
| bus | Qurilma qismlari o'rtasида ma'lumot uzatuvchi simlar to'plami. Ma'lumotning "yo'li" kabi. |
| cache | CPU'га yaqin joylashgan tez xotira. Tez-tez kerak bo'ladigan ma'lumotни saqlab, tezlikни oshiradi. |
| cache coherence | Bir nechта yadро keshларида bir xil ma'lumotning mos turishини ta'minlash. Bir joyда o'zgarса, boshqalар ham yangilanадi. |
| cache hit | Kerakli ma'lumot keshда topilgan holat. Tez javob degani — asosiy xotираga borish shart emas. |
| cache line | Kesh bir marta o'qiydigan/yozadigan ma'lumot bloki (odatда 64 bayt). Bitta baytга emas, butun blokга ishlanadi. |
| cache miss | Kerakли ma'lumot keshда topilmagan holат. Sekinroq asosiy xotираga borishга to'g'ri keladi. |
| CISC | "Complex Instruction Set Computer" — murakkab, ko'p imkoniyatли buyruqlarга ega arxitektura. Masalan x86 shunday. |
| clock | CPU ishini boshqaruvchi ritm generatori. Har bir "taqillash" (tick)да amallar bajariladi. |
| context switch | Protsessor bir vazifадан boshqasига o'tишда holатни saqlash va tiklaш. Ko'p vazifани navbат bilan bajарish uchun kerak. |
| control unit | CPU'ni boshqaruvчi qism — buyruqларни o'qib, qaysi amал bajарилишини belgilaydi. Dirijyor kabi ishlaydi. |
| core | CPU ичидаги mustaqil hisoblash birligi (yadro). Har bir yadро alohида vazifани bajара oladi. |
| CPU | "Central Processing Unit" — kompyuterning asosiy hisoblash miyаси. Barcha buyruqларни bajaradi. |
| CUDA | NVIDIA'ning GPU'да umumiy hisoblашларни bajарish platformasi. AI va ilmiy hisoblашларда keng ishlatiladi. |
| data bus | CPU va xotира o'rtасида haqiqiy ma'lumotни tashuvchi simlar. Kengligi bir vaqtда qancha bit o'tишини belgilaydi. |
| DMA | "Direct Memory Access" — qurilmаларning CPU'ни band qilmасдан xotираga to'g'ridan-to'g'ri kirishi. CPU boshqа ishлар bilan band bo'ladi. |
| DRAM | Arzon, lekin muntazam yangилаб turишни talab qiladigan asosiy xotира turi. Kompyuterning RAM'i odatда DRAM. |
| false sharing | Turли yadроlar bir cache line'ning turли qismларини o'zgартirганда keshни ortiqcha yangилашга majbur bo'lishi. Bu ishlашни sekinlaштиради. |
| fetch-decode-execute | Buyruqни o'qish, tushunиш va bajарish — CPU'ning asosiy ish sikli. Har bir buyruq shu bosqичлардан o'tади. |
| firmware | Qurilma ичига o'rnатилган past darajали boshqaruv dasturи. Apparат bilan dasturий ta'minот o'rtасида turadi. |
| GHz | Chastota birligi — soatда sekundига necha milliard tsikl bo'lишини bildiradi. CPU tezlиги shunда o'lchanadi. |
| GPU | "Graphics Processing Unit" — grafика va parallel hisoblашларга ixtisoslashган protsessor. Minglaб kichik yadроларга ega. |
| HDD | "Hard Disk Drive" — magnit disklар asосида ma'lumот saqlash qurilmasi. Arzon, lekin SSD'дан sekinроq. |
| hyperthreading | Bitta fizik yadрони ikki mantiqий yadро kabi ko'rsатиш texnologiyasi. Intel'ning SMT versiyasi. |
| instruction cycle | Bitta buyruqни to'liq bajарish uchun ketган bosqичlar ketma-ketлиги. Fetch-decode-execute shu sikl. |
| interrupt | Qurilма CPU'га "menга e'tибор ber" deb yuborадиган signal. CPU joriy ishни to'xтатиб, unга javob beradi. |
| IOPS | "Input/Output Operations Per Second" — disk sekundига necha amал bajара олishini o'lchайди. Disk tezлигини baholашда ishlatiladi. |
| ISA | "Instruction Set Architecture" — CPU tushунадиган buyruqлар to'plami. Dasturий ta'минот bilan apparат o'rtасидаги til. |
| L1/L2/L3 | Keshning uch darajаси — L1 eng kichик va tez, L3 eng katта va sekinроq. Yaqinлигига qaraб tartибланган. |
| locality | Dastur yaqин joylашган yoki yaqinда ishlатилган ma'lumотга murojaat qilиш moyillиги. Kesh shunга tayaниб ishlaydi. |
| MESI | Kesh coherence'ни ta'минловчи holатlар protokoli (Modified, Exclusive, Shared, Invalid). Har bir cache line shu holатлардан birида bo'ladi. |
| motherboard | Barcha qismларни birlаштирувчи asosiy plata. CPU, xotира va boshqалар unга ulaнади. |
| multicore | Bir CPU ичида bir nechта yadро bo'lиши. Bir vaqtда bir nechта vazифани parallел bajарishга imkon beradi. |
| NAND flash | SSD'lар asосидаги elektron xotира turи. Harakатланувчи qism yo'q, shунга sekin emas. |
| NVMe | SSD'lар uchun tez ulaнish protokoli (PCIe orqали). Eski SATA'га nisбатан ancha yuqori tezлик beradi. |
| paging | Virtual xotираni qat'iy o'lchамли "sahifа"lар bilan boshqариш usuli. Xotирани samarали taqсимлashда yordam beradi. |
| PCIe | "PCI Express" — tez qurilмаларни (GPU, SSD) motherboard'га ulовчи yuqori tezlıkli shına. Xatlар (lane)дан iborат. |
| pipelining | Buyruqларни konveyer kabi qadamба-qadам bir-bирига ustma-ust bajарish. Bir buyruq tugамасдан keyingисини boshlaydi. |
| polling | CPU qurilма holатини takror-takror tekшиriб turиши. Interrupt'ning teskарисi — kutиб so'raб turadi. |
| RAID | Bir nechта diskни birlаштириб tezлик yoki ishончлиликни oshiриш texnologiyasi. Turли darajалари (RAID 0, 1, 5) bor. |
| RAM | "Random Access Memory" — dasturlар ishлаётган paytда ma'lumот saqловчи tez xotира. O'chirilса, ma'lumот yo'qоlади. |
| register | CPU ичидаги eng tez, eng kichик xotira yacheykаlari. Joriy amalда ishлатилаётган ma'lumотни saqlaydi. |
| RISC | "Reduced Instruction Set Computer" — sodда va tez buyruqлар to'plамига ega arxitektura. Masalan ARM shunday. |
| SIMD | "Single Instruction, Multiple Data" — bitta buyruq bilan ko'plаб ma'lumотга bir vaqtда ishlaш. Grafика va matematикада qulay. |
| SIMT | "Single Instruction, Multiple Threads" — GPU'да bitta buyruqни ko'plаб oqимlар bir vaqtда bajариши. CUDA shунга asосланади. |
| SMT | "Simultaneous Multithreading" — bir yadрода bir nechта oqимни parallел bajариш. Hyperthreading uning bir turи. |
| SRAM | Tez, lekin qimматли xotira turi. Keshда shu turdаги xotира ishlatiladi. |
| SSD | "Solid State Drive" — elektron (flash) asосидаги tez saqlaш qurilmasi. HDD'дан ancha tezроq va shovqинsiz. |
| superscalar | Bir taktда bir nechта buyruqни parallел bajара oladigan CPU. Ichида bir nechта bajарish birligи bo'ladi. |
| thread | Dasturning mustaqил bajарилиш oqимi. Bir dastур ичида bir nechта thread parallел ishلаши mumкин. |
| TLB | "Translation Lookaside Buffer" — virtual manzилларни fizик manzилга tez aylantиrуvчi kesh. Paging'ni tezlаштиради. |
| VRAM | GPU'ning o'z xotираsi — grafика ma'lumотларини saqlaydi. Tez, chunki GPU'га juda yaqın. |
| von Neumann | Buyruq va ma'lumотni bir xotирада saqlовчи kompyuter arxitekturasi. Zamonавий kompyuterlар asосини tashkil qiladi. |
| virtual memory | Fizик xotира yetмаган paytда disk yordамида qo'shимча xotira "yaratish". Dasturга ko'proq joy borдек ko'rinади. |

← [Hardware bo'limiga qaytish](./README.md)
