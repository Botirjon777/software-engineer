# Arxitektura Patternlar

Senior backend muhandisdan kutiladigan narsa — alohida endpoint yozish emas, balki **tizimni qanday bo'laklarga ajratish**, bo'laklar bir-biri bilan qanday gaplashishi va xato bo'lganda nima sodir bo'lishini loyihalashtirishdir. Bu bo'lim layered architecture'dan tortib event-driven sistemalar, saga, CQRS va observability'gacha bo'lgan asosiy patternlarni qamrab oladi.

## Mundarija

- [Layered Architecture (Controller / Service / Repository)](#layered-architecture)
- [Separation of Concerns](#separation-of-concerns)
- [Dependency Injection](#dependency-injection)
- [Monolith vs Microservices](#monolith-vs-microservices)
- [Service Communication (Sync vs Async)](#service-communication)
- [Message Queue va Event-Driven](#message-queue-va-event-driven)
- [At-least-once vs Exactly-once](#delivery-guarantees)
- [Idempotency](#idempotency)
- [Saga Pattern va Distributed Transactions](#saga-pattern)
- [CQRS va Event Sourcing](#cqrs-va-event-sourcing)
- [API Gateway va BFF](#api-gateway-va-bff)
- [Caching Layer](#caching-layer)
- [Twelve-Factor App](#twelve-factor-app)
- [Config Management](#config-management)
- [Error Handling, Logging, Observability](#observability)
- [Graceful Shutdown](#graceful-shutdown)
- [Intervyu Savollari (Q&A)](#intervyu-savollari)
- [Masalalar](#masalalar)

---

## Layered Architecture

**💡 Tushuncha:** Layered (qatlamli) arxitektura kodni mas'uliyatiga ko'ra qatlamlarga bo'ladi. Eng keng tarqalgan uch qatlam:

- **Controller (yoki Handler)** — HTTP so'rovni qabul qiladi, inputni validatsiya qiladi, service'ni chaqiradi, javobni qaytaradi. Biznes logikani BILMASLIGI kerak.
- **Service** — biznes logika shu yerda. Tranzaksiyalar, qoidalar, bir nechta repository'ni birlashtirish. HTTP'ni BILMASLIGI kerak (`req`/`res` bu yerga kirmaydi).
- **Repository** — ma'lumotlar bazasi bilan ishlash. SQL/ORM faqat shu yerda. Biznes qoidalarni BILMASLIGI kerak.

Bog'lanish yo'nalishi bir tomonlama: `Controller → Service → Repository`. Yuqori qatlam pastdagini chaqiradi, aksincha emas.

```ts
// repository — faqat data access
export class UserRepository {
  constructor(private db: Pool) {}

  async findById(id: string): Promise<User | null> {
    const { rows } = await this.db.query(
      "SELECT * FROM users WHERE id = $1",
      [id]
    );
    return rows[0] ?? null;
  }
}

// service — biznes logika
export class UserService {
  constructor(private users: UserRepository) {}

  async getProfile(id: string): Promise<UserProfile> {
    const user = await this.users.findById(id);
    if (!user) throw new NotFoundError("user not found");
    return toProfile(user); // biznes qoida: qaysi maydonlar ko'rinadi
  }
}

// controller — faqat HTTP
export async function getUserHandler(req: Request, res: Response) {
  const profile = await userService.getProfile(req.params.id);
  res.json(profile);
}
```

**⚠️ Ehtiyot bo'l:** Eng keng tarqalgan xato — "fat controller" yoki biznes logikani repository ichiga yashirish. Agar `if (user.role === 'admin')` kabi qoida controller'da yoki SQL'da turgan bo'lsa, u service'ga ko'chishi kerak. Aks holda testlash qiyinlashadi va logika takrorlanadi.

---

## Separation of Concerns

**💡 Tushuncha:** Separation of Concerns (SoC) — har bir modul faqat BITTA narsaga javobgar bo'lsin degan prinsip. Layered architecture buning bir ko'rinishi. SoC quyidagilarni beradi:

- **Testlash osonligi** — service'ni HTTP server ko'tarmasdan unit-test qilish mumkin.
- **O'zgartirish izolyatsiyasi** — Postgres'dan Mongo'ga o'tsangiz, faqat repository o'zgaradi.
- **Cognitive load kamayishi** — bir faylni o'qiyotganda bir vaqtning o'zida HTTP, biznes va SQL haqida o'ylash shart emas.

**⚠️ Ehtiyot bo'l:** SoC'ni haddan oshirib yuborish ham xato. Har bir kichik narsa uchun alohida qatlam yaratish (over-engineering) kodni o'qishni qiyinlashtiradi. Qatlam qo'shishdan oldin: "bu qatlam haqiqatan o'zgarish sababini izolyatsiya qiladimi?" deb so'rang.

---

## Dependency Injection

**💡 Tushuncha:** Dependency Injection (DI) — obyekt o'z bog'liqliklarini (dependencies) ichida `new` bilan YARATMAY, tashqaridan qabul qilishi. Bu **Dependency Inversion** prinsipini amalda qo'llash usuli.

```ts
// YOMON — service o'zi repository yaratadi, qattiq bog'langan
class OrderService {
  private repo = new OrderRepository(new Pool()); // test qilish qiyin
}

// YAXSHI — dependency tashqaridan keladi
class OrderService {
  constructor(private repo: OrderRepository) {}
}

// composition root — barcha bog'liqliklar bir joyda ulanadi
const db = new Pool(config.db);
const orderRepo = new OrderRepository(db);
const orderService = new OrderService(orderRepo);
```

DI'ning afzalliklari: testda real repository o'rniga **mock/fake** berish mumkin; bog'liqliklar interfeysga (abstraksiyaga) bog'lanadi, konkret klassga emas.

```ts
// interfeysga bog'lanish — repository implementatsiyasini almashtirish mumkin
interface UserRepo {
  findById(id: string): Promise<User | null>;
}

class UserService {
  constructor(private users: UserRepo) {}
}

// testda:
const fakeRepo: UserRepo = {
  findById: async () => ({ id: "1", name: "Test" } as User),
};
const service = new UserService(fakeRepo);
```

**⚠️ Ehtiyot bo'l:** DI container (InversifyJS, tsyringe, NestJS DI) qulay, lekin majburiy emas. Kichik loyihada qo'lda "composition root"da ulash (manual DI) ko'pincha yetarli va tushunarliroq. Container'ni "sehrli" deb hisoblamang — u faqat siz qo'lda yozadigan ulanishni avtomatlashtiradi.

---

## Monolith vs Microservices

**💡 Tushuncha:** **Monolith** — butun ilova bitta deploy qilinadigan birlik (bitta codebase, bitta jarayon). **Microservices** — ilova mustaqil deploy qilinadigan kichik servislarga bo'linadi, har biri o'z DB'siga ega va tarmoq orqali gaplashadi.

| | Monolith | Microservices |
|---|----------|---------------|
| Deploy | bitta birlik | har servis alohida |
| Scaling | butun ilova | servis darajasida |
| Tranzaksiya | oddiy (bitta DB) | murakkab (distributed) |
| Tarmoq | yo'q (in-process) | har chaqiruv tarmoq orqali |
| Operatsion murakkablik | past | yuqori |
| Jamoa | kichik jamoaga mos | ko'p mustaqil jamoa |
| Debugging | oddiy (bitta stack trace) | murakkab (tracing kerak) |

**⚠️ Ehtiyot bo'l:** "Microservices = zamonaviy = yaxshi" degan fikr noto'g'ri. Ko'pchilik tizimlar uchun **modular monolith** (yaxshi modullarga bo'lingan, lekin bitta deploy) optimal nuqta. Microservices'ga o'tish operatsion narx (tarmoq, monitoring, distributed tracing, eventual consistency) keltiradi.

**Qachon microservices'ga o'tish kerak:**
- Jamoalar bir-birini deploy'da bloklayotgan bo'lsa (mustaqil release kerak).
- Tizimning bir qismi boshqasidan butunlay boshqacha scaling talab qilsa (masalan, video transcoding vs. CRUD).
- Domenlar aniq chegaralangan bo'lsa (bounded contexts).

**Maslahat:** "Monolith first" — kichikdan boshlang, chegaralarni o'rganib, keyin kerakli joyni ajrating. Boshidan microservice qilish — chegaralarni hali bilmaganda noto'g'ri bo'lish riski.

---

## Service Communication

**💡 Tushuncha:** Servislar ikki xil usulda gaplashadi:

**Synchronous (so'rov-javob):** chaqiruvchi javobni kutib turadi.
- **REST/HTTP** — oddiy, universal, debugging oson, lekin verbose.
- **gRPC** — Protobuf orqali binary, tez, strongly-typed kontrakt, streaming. Servis-servis ichki aloqa uchun yaxshi; brauzer bilan to'g'ridan-to'g'ri ishlamaydi.

**Asynchronous (xabar orqali):** chaqiruvchi xabarni yuboradi va javobni kutmaydi (fire-and-forget yoki keyinroq event orqali).
- **Message queue / event bus** — Kafka, RabbitMQ, SQS.

```ts
// SYNC — REST chaqiruv, javobni kutadi
const res = await fetch("http://inventory/api/reserve", {
  method: "POST",
  body: JSON.stringify({ sku, qty }),
});
const { reserved } = await res.json();

// ASYNC — event chiqaradi, javobni kutmaydi
await producer.send("order.placed", { orderId, items });
// inventory service o'zi mustaqil ravishda bu eventni qabul qiladi
```

| | Sync (REST/gRPC) | Async (messaging) |
|---|------------------|-------------------|
| Bog'lanish (coupling) | qattiq (temporal) | bo'sh (decoupled) |
| Javob | darhol | keyinroq / yo'q |
| Failure ta'siri | callee yiqilsa caller ham bloklanadi | broker buffer qiladi |
| Murakkablik | past | yuqori (broker, ordering) |
| Mos holatlar | so'rov darhol javob talab qilsa | background ish, fan-out, decoupling |

**⚠️ Ehtiyot bo'l:** Sync chaqiruvlar zanjiri (A→B→C→D) **latency'ni qo'shadi** va **cascading failure** xavfini oshiradi — bitta servis sekinlashsa, butun zanjir sekinlashadi. Bunday joylarda timeout, retry va circuit breaker majburiy.

---

## Message Queue va Event-Driven

**💡 Tushuncha:** Message queue — **producer** xabar yuboradi, **broker** uni saqlaydi, **consumer** o'qib qayta ishlaydi. Bu producer va consumer'ni vaqt jihatidan ajratadi (decoupling) va load'ni tekislaydi (load leveling).

Ikki asosiy model:
- **Queue (point-to-point):** har xabarni bitta consumer oladi (ish taqsimlash). Masalan SQS, RabbitMQ work queue.
- **Pub/Sub (publish-subscribe):** har xabarni barcha obunachilar oladi (fan-out). Masalan Kafka topiclar, RabbitMQ fanout exchange.

**Brokerlar taqqoslamasi:**

| | Kafka | RabbitMQ | SQS |
|---|-------|----------|-----|
| Model | distributed log | message broker | managed queue |
| Saqlash | retention bilan (qayta o'qish mumkin) | iste'mol qilingach o'chadi | iste'mol qilingach o'chadi |
| Ordering | partition ichida | queue ichida | FIFO queue'da |
| Throughput | juda yuqori | o'rtacha-yuqori | yuqori (managed) |
| Use case | event streaming, log, analytics | task queue, routing | AWS'da oddiy queue |

```ts
// PRODUCER (Kafka misoli)
await producer.send({
  topic: "order.events",
  messages: [{ key: orderId, value: JSON.stringify({ type: "placed", orderId }) }],
});

// CONSUMER
await consumer.subscribe({ topic: "order.events" });
await consumer.run({
  eachMessage: async ({ message }) => {
    const event = JSON.parse(message.value!.toString());
    await handleOrderEvent(event); // qayta ishlash
    // offset commit qilish bu yerda yoki avtomatik
  },
});
```

**Event-driven architecture (EDA):** servislar bir-birini to'g'ridan chaqirmaydi, balki **eventlar** chiqaradi ("OrderPlaced", "PaymentReceived"). Boshqa servislar bu eventlarga reaksiya bildiradi. Bu bo'sh bog'lanish va kengaytirilishni beradi (yangi consumer qo'shish mavjud kodga tegmaydi).

---

## Delivery Guarantees

**💡 Tushuncha:** Distributed sistemada xabar yetkazib berish kafolatlari:

- **At-most-once:** xabar ko'pi bilan bir marta yetkaziladi (yo'qolishi mumkin). Tez, lekin ishonchsiz.
- **At-least-once:** xabar kamida bir marta yetkaziladi (TAKRORLANISHI mumkin). Eng keng tarqalgan; consumer **idempotent** bo'lishi shart.
- **Exactly-once:** xabar aniq bir marta. Nazariy jihatdan tarmoqda erishish qiyin; amalda "at-least-once + idempotent consumer" yoki Kafka transactions orqali taqlid qilinadi.

**⚠️ Ehtiyot bo'l:** Haqiqiy distributed sistemada "exactly-once delivery" amalda yo'q — yetkazib berishni network qaytarishi mumkin. To'g'ri yondashuv: **at-least-once delivery + idempotent processing = exactly-once effect** (natija). "Exactly-once'ni broker konfiguratsiyasi hal qiladi" deb ishonmang.

---

## Idempotency

**💡 Tushuncha:** Operatsiya **idempotent** bo'lsa — uni bir marta yoki ko'p marta bajarish bir xil natija beradi. Distributed sistemada retry va at-least-once delivery sababli bir xabar ikki marta kelishi mumkin, shuning uchun consumer/endpoint idempotent bo'lishi kerak.

```ts
// IDEMPOTENCY KEY orqali takroriy so'rovni aniqlash
async function processPayment(req: PaymentRequest) {
  const key = req.idempotencyKey; // mijoz beradi

  const existing = await db.query(
    "SELECT result FROM idempotency WHERE key = $1",
    [key]
  );
  if (existing.rows.length > 0) {
    return existing.rows[0].result; // allaqachon bajarilgan, qayta bajarilmaydi
  }

  const result = await charge(req); // haqiqiy operatsiya
  await db.query(
    "INSERT INTO idempotency (key, result) VALUES ($1, $2)",
    [key, result]
  );
  return result;
}
```

HTTP'da `GET`, `PUT`, `DELETE` tabiatan idempotent; `POST` emas — shuning uchun `POST` uchun ko'pincha `Idempotency-Key` header ishlatiladi (Stripe shunday qiladi).

**⚠️ Ehtiyot bo'l:** Idempotency keyni saqlash va haqiqiy operatsiyani **atomar** qilish kerak, aks holda race condition'da ikkala so'rov ham "yangi" deb hisoblanishi mumkin. Unique constraint yoki tranzaksiya bilan himoyalang.

---

## Saga Pattern

**💡 Tushuncha:** Microservices'da har servis o'z DB'siga ega, shuning uchun bitta ACID tranzaksiya bir nechta servisni qamrab ololmaydi. **Saga** — distributed tranzaksiyani lokal tranzaksiyalar ketma-ketligiga aylantiradi; biror qadam muvaffaqiyatsiz bo'lsa, **compensating** (qaytaruvchi) amallar ishga tushadi.

Ikki uslub:
- **Choreography:** har servis event chiqaradi, keyingisi unga reaksiya bildiradi. Markaziy boshqaruvchi yo'q. Oddiy, lekin oqimni kuzatish qiyin.
- **Orchestration:** markaziy **orchestrator** qadamlarni boshqaradi va xato bo'lsa compensation chaqiradi. Aniqroq, lekin markaziy nuqta.

```ts
// ORCHESTRATION misoli — order saga
async function createOrderSaga(order: Order) {
  const reserved = await inventory.reserve(order.items); // qadam 1
  try {
    const payment = await payments.charge(order); // qadam 2
    try {
      await shipping.schedule(order); // qadam 3
    } catch (e) {
      await payments.refund(payment); // compensation 2
      await inventory.release(reserved); // compensation 1
      throw e;
    }
  } catch (e) {
    await inventory.release(reserved); // compensation 1
    throw e;
  }
}
```

**⚠️ Ehtiyot bo'l:** Saga **eventual consistency** beradi, ACID emas — qadamlar orasida tizim vaqtincha nomuvofiq holatda bo'ladi (masalan, pul yechildi lekin yetkazib berish hali rejalashtirilmadi). UI va biznes mantiq buni hisobga olishi kerak. Compensation har doim ham mukammal "rollback" emas (email yuborilgan bo'lsa, uni qaytarib bo'lmaydi).

---

## CQRS va Event Sourcing

**💡 Tushuncha:** **CQRS** (Command Query Responsibility Segregation) — yozish (command) va o'qish (query) modellarini ajratadi. Yozish normallashtirilgan modelga boradi, o'qish esa optimallashtirilgan (denormalized) read modeldan keladi. Murakkab domenlarda va o'qish/yozish nisbati juda nomutanosib bo'lganda foydali.

**Event Sourcing** — joriy holatni saqlash o'rniga, holatni o'zgartirgan **barcha eventlarni** ketma-ket saqlaydi. Joriy holat eventlarni qayta o'ynatish (replay) orqali tiklanadi.

```ts
// EVENT SOURCING — holat emas, eventlar saqlanadi
const events = [
  { type: "AccountOpened", balance: 0 },
  { type: "Deposited", amount: 100 },
  { type: "Withdrawn", amount: 30 },
];

// joriy holat = eventlarni qayta o'ynatish
const balance = events.reduce((acc, e) => {
  if (e.type === "Deposited") return acc + e.amount!;
  if (e.type === "Withdrawn") return acc - e.amount!;
  return acc;
}, 0); // => 70
```

Afzalliklar: to'liq audit log, holatni istalgan vaqtga tiklash, debugging. Kamchiliklar: murakkablik, schema evolution muammosi, snapshot kerakligi.

**⚠️ Ehtiyot bo'l:** CQRS va event sourcing kuchli, lekin OG'IR patternlar. Oddiy CRUD ilovaga ularni kiritish — klassik over-engineering. Ularni faqat murakkab domen, audit talabi yoki haqiqiy o'qish/yozish nomutanosibligi bo'lganda qo'llang. CQRS event sourcing'siz ham mumkin.

---

## API Gateway va BFF

**💡 Tushuncha:** **API Gateway** — barcha tashqi so'rovlar uchun yagona kirish nuqtasi. U routing, authentication, rate limiting, request aggregation, SSL termination kabi cross-cutting vazifalarni bajaradi. Mijoz ichki servislar tuzilishini bilmaydi.

**BFF (Backend for Frontend)** — har xil mijoz turi (web, mobil, smart TV) uchun alohida moslangan backend qatlami. Mobil kam ma'lumot, web ko'proq ma'lumot oladi — har biriga moslangan API.

```
Mijoz → API Gateway → [auth, rate limit, routing] → ichki servislar

Web   → Web BFF   ┐
Mobil → Mobil BFF ┼→ ichki microservices
```

**⚠️ Ehtiyot bo'l:** API Gateway "god object" bo'lib qolmasin — unga biznes logika joylashtirmang, faqat cross-cutting concerns. Gateway single point of failure bo'lishi mumkin, shuning uchun u high-available bo'lishi kerak.

---

## Caching Layer

**💡 Tushuncha:** Caching layer — tez-tez so'raladigan ma'lumotni tez xotiraga (Redis, Memcached) saqlab, DB yukini kamaytiradi va latency'ni tushiradi. Eng keng tarqalgan pattern **cache-aside**: avval cache'ni tekshir, bo'lmasa DB'dan ol va cache'ga yoz.

```ts
async function getUser(id: string): Promise<User> {
  const cached = await redis.get(`user:${id}`);
  if (cached) return JSON.parse(cached); // cache hit

  const user = await userRepo.findById(id); // cache miss
  await redis.set(`user:${id}`, JSON.stringify(user), "EX", 300); // TTL 5 min
  return user;
}
```

**⚠️ Ehtiyot bo'l:** Cache invalidation — eng qiyin masalalardan biri. Ma'lumot o'zgarganda cache'ni yangilash yoki o'chirish unutilsa, **stale data** beriladi. TTL qisqa bo'lsa hit rate tushadi, uzun bo'lsa stale risk oshadi. (Caching strategiyalari haqida batafsil — System Design bo'limida.)

---

## Twelve-Factor App

**💡 Tushuncha:** Twelve-Factor App — SaaS ilovalarni qurish uchun 12 ta amaliy prinsip to'plami. Backend uchun eng muhimlari:

1. **Codebase** — bitta codebase, version control'da, ko'p deploy.
2. **Dependencies** — aniq deklaratsiya (package.json), izolyatsiya.
3. **Config** — config muhitda (environment variables), kodda emas.
4. **Backing services** — DB, cache, queue almashtiriladigan resurs sifatida.
5. **Build, release, run** — bosqichlarni qat'iy ajratish.
6. **Processes** — ilova **stateless**, holat backing service'da.
7. **Port binding** — ilova o'zi portga bind qiladi (self-contained).
8. **Concurrency** — gorizontal scaling (ko'proq process).
9. **Disposability** — tez start, **graceful shutdown**.
10. **Dev/prod parity** — muhitlar imkon qadar bir xil.
11. **Logs** — log'ni stdout'ga oqim sifatida yozish (faylga emas).
12. **Admin processes** — bir martalik vazifalar (migration) bir xil muhitda.

**⚠️ Ehtiyot bo'l:** Eng ko'p buziladigan qoida — **stateless processes** (6-omil). Agar siz session yoki yuklangan faylni lokal xotira/diskda saqlasangiz, gorizontal scaling buziladi (ikkinchi instance bu holatni bilmaydi). Holatni Redis/DB/S3'ga ko'chiring.

---

## Config Management

**💡 Tushuncha:** Konfiguratsiya muhitga ko'ra o'zgaradi (dev/staging/prod), kod esa o'zgarmaydi. Shuning uchun config kodda **hardcode** qilinmasligi kerak. Manbalar: environment variables, secret manager (Vault, AWS Secrets Manager), config service.

```ts
// config'ni bir joyda validatsiya bilan o'qish
import { z } from "zod";

const ConfigSchema = z.object({
  PORT: z.coerce.number().default(3000),
  DATABASE_URL: z.string().url(),
  REDIS_URL: z.string().url(),
  NODE_ENV: z.enum(["development", "production", "test"]),
});

// ilova start bo'lishida tekshir — noto'g'ri config = darhol crash (fail fast)
export const config = ConfigSchema.parse(process.env);
```

**⚠️ Ehtiyot bo'l:** Sirlarni (parol, API key) hech qachon kodga yoki git'ga qo'shmang. `.env` faylni `.gitignore`'ga qo'shing. Config'ni start vaqtida validatsiya qiling — noto'g'ri config bilan ishlamay, darhol yiqilish (fail fast) productionda yashirin xatodan yaxshiroq.

---

## Observability

**💡 Tushuncha:** Observability — tizim ichida nima bo'layotganini tashqaridan tushuna olish. Uch ustun:

- **Logs** — diskret hodisalar yozuvi. **Structured logging** (JSON) ishlating, matn emas, shunda qidirish/filtrlash oson.
- **Metrics** — vaqt bo'yicha agregatlangan sonlar (request count, latency p99, error rate, CPU). Dashboard va alert uchun.
- **Traces** — bitta so'rovning servislar bo'ylab to'liq yo'li. Distributed sistemada bottleneck topish uchun.

```ts
// STRUCTURED LOGGING — JSON, correlation ID bilan
logger.info({
  event: "order_created",
  orderId,
  userId,
  durationMs: 42,
  traceId: req.headers["x-trace-id"], // so'rovlarni bog'lash uchun
});
```

**Error handling prinsiplari:**
- Operatsion xato (404, validatsiya) vs programmer xato (bug) ni ajrating.
- Markaziy error handler ishlating, har joyda `try/catch` takrorlamang.
- Xatoni yutib yubormang (swallow) — log qiling yoki qayta tashlang.

```ts
// markaziy error handler (Express)
app.use((err: Error, req: Request, res: Response, next: NextFunction) => {
  logger.error({ event: "unhandled_error", message: err.message, stack: err.stack });
  if (err instanceof NotFoundError) return res.status(404).json({ error: err.message });
  res.status(500).json({ error: "internal error" }); // mijozga stack chiqarmang
});
```

**⚠️ Ehtiyot bo'l:** Xato xabarida ichki ma'lumot (stack trace, SQL, fayl yo'li) mijozga oqib chiqmasin — bu xavfsizlik teshigidir. Ichkariga to'liq log, tashqariga umumiy xabar.

---

## Graceful Shutdown

**💡 Tushuncha:** Graceful shutdown — ilova `SIGTERM`/`SIGINT` signalini olganda **darhol o'lmaslik**, balki: yangi so'rovlarni qabul qilishni to'xtatish, davom etayotgan so'rovlarni tugatish, ulanishlarni (DB, queue) yopish va shundan keyin chiqish. Bu deploy va scaling vaqtida so'rov yo'qotmaslik uchun muhim (12-factor disposability).

```ts
const server = app.listen(config.PORT);

async function shutdown(signal: string) {
  logger.info({ event: "shutdown_started", signal });

  server.close(async () => { // yangi connection qabul qilmaydi, mavjudini tugatadi
    await db.end();       // DB pool yopiladi
    await redis.quit();   // Redis ulanishi yopiladi
    await consumer.disconnect(); // queue consumer to'xtaydi
    logger.info({ event: "shutdown_complete" });
    process.exit(0);
  });

  // agar belgilangan vaqtda tugamasa — majburan chiq
  setTimeout(() => {
    logger.error({ event: "shutdown_forced" });
    process.exit(1);
  }, 10_000).unref();
}

process.on("SIGTERM", () => shutdown("SIGTERM"));
process.on("SIGINT", () => shutdown("SIGINT"));
```

**⚠️ Ehtiyot bo'l:** Kubernetes pod'ni o'chirishda avval `SIGTERM` yuboradi, keyin grace period o'tgach `SIGKILL`. Agar graceful shutdown grace period'dan uzoq cho'zilsa, baribir majburan o'ldiriladi. Shutdown'ni timeout bilan cheklang.

---

## Intervyu Savollari

### ❓ Service va Repository qatlamlari orasidagi farq nimada?

**✅ Javob:** **Repository** faqat ma'lumotlar bazasi bilan ishlaydi — CRUD, query'lar, ORM. U biznes qoidalarni bilmaydi. **Service** biznes logikani saqlaydi — qoidalar, tranzaksiyalar, bir nechta repository'ni birlashtirish. Service repository'ni ishlatadi, lekin SQL bilan to'g'ridan ishlamaydi. Sinov mezoni: agar `if user.role === 'admin'` kabi qoida repository'da bo'lsa — bu noto'g'ri, u service'ga tegishli.

### ❓ Dependency Injection nima beradi va u qachon kerak?

**✅ Javob:** DI obyektni o'z bog'liqliklarini tashqaridan qabul qilishga majbur qiladi, ichida `new` qilmasdan. Bu beradi: (1) testda real dependency o'rniga mock berish; (2) konkret klassga emas, interfeysga bog'lanish; (3) bog'liqliklarni bir joyda (composition root) ko'rish. DI container majburiy emas — kichik loyihada qo'lda ulash yetarli.

### ❓ Qachon monolith'dan microservices'ga o'tish kerak?

**✅ Javob:** Texnik sabab emas, **tashkiliy va scaling** sababga ko'ra: jamoalar bir-birini deploy'da bloklayotganda, tizimning bir qismi butunlay boshqacha scaling talab qilganda, domenlar aniq chegaralanganda. "Monolith first" — kichikdan boshlab chegaralarni o'rganib, keyin kerakli joyni ajrating. Boshidan microservice — chegaralarni bilmasdan noto'g'ri bo'lish riski va katta operatsion narx.

### ❓ Sync va async kommunikatsiya orasida qanday tanlaysiz?

**✅ Javob:** **Sync** (REST/gRPC) — so'rov darhol javob talab qilsa (foydalanuvchi profilini olish). **Async** (messaging) — background ish, fan-out, servislarni decouple qilish kerak bo'lsa (order placed → email, analytics, inventory). Sync zanjirlari latency va cascading failure xavfini oshiradi, async esa murakkablik (broker, ordering, eventual consistency) keltiradi.

### ❓ At-least-once va exactly-once delivery farqi nimada?

**✅ Javob:** **At-least-once** — xabar kamida bir marta yetadi, lekin takrorlanishi mumkin; consumer idempotent bo'lishi shart. **Exactly-once** — aniq bir marta, lekin haqiqiy tarmoqda yetkazib berish darajasida amalda erishish qiyin. To'g'ri yondashuv: at-least-once delivery + idempotent processing = exactly-once **effect** (natija).

### ❓ Idempotency nima va uni qanday ta'minlaysiz?

**✅ Javob:** Idempotent operatsiyani bir yoki ko'p marta bajarish bir xil natija beradi. Buni ta'minlash: (1) `Idempotency-Key` orqali takroriy so'rovni aniqlash va saqlangan natijani qaytarish; (2) unique constraint bilan ikkilanishni bloklash; (3) operatsiyani idempotent yozish (`SET status='paid'` o'rniga `balance += x` emas). Retry va at-least-once delivery sababli distributed sistemada majburiy.

### ❓ Saga pattern qanday ishlaydi va u nima uchun kerak?

**✅ Javob:** Microservices'da har servis o'z DB'siga ega, shuning uchun bir ACID tranzaksiya bir nechta servisni qamrolmaydi. Saga distributed tranzaksiyani lokal tranzaksiyalar ketma-ketligiga aylantiradi; qadam muvaffaqiyatsiz bo'lsa, **compensating** amallar ishga tushib avvalgilarni qaytaradi. Ikki uslub: choreography (eventlar orqali, markazsiz) va orchestration (markaziy boshqaruvchi). Saga eventual consistency beradi, ACID emas.

### ❓ CQRS va event sourcing'ni qachon ishlatish kerak?

**✅ Javob:** **CQRS** — o'qish va yozish modellarini ajratadi; o'qish/yozish nisbati juda nomutanosib yoki o'qish modeli juda murakkab bo'lganda. **Event sourcing** — holat o'rniga uni o'zgartirgan eventlarni saqlash; to'liq audit, time-travel debugging kerak bo'lganda. Ikkalasi ham og'ir pattern — oddiy CRUD'ga kiritish over-engineering. Faqat murakkab domen, audit talabi yoki haqiqiy ehtiyoj bo'lganda.

### ❓ API Gateway va BFF orasidagi farq nima?

**✅ Javob:** **API Gateway** — barcha mijozlar uchun yagona kirish nuqtasi; routing, auth, rate limiting kabi cross-cutting concerns'ni bajaradi. **BFF** — har mijoz turi (web, mobil) uchun alohida moslangan backend; mobil kam, web ko'p ma'lumot oladi. Gateway umumiy, BFF mijozga xos. Ikkalasi birga ham ishlatilishi mumkin.

### ❓ Stateless arxitektura nima uchun muhim?

**✅ Javob:** Stateless process holatni o'zida (lokal xotira/disk) saqlamaydi, balki backing service'da (Redis, DB, S3). Bu gorizontal scaling'ni mumkin qiladi — istalgan instance istalgan so'rovni qabul qila oladi, load balancer erkin taqsimlaydi, instance yiqilsa holat yo'qolmaydi. Session yoki yuklangan faylni lokal saqlash bu prinsipni buzadi va scaling'ni sindiradi (12-factor 6-omil).

### ❓ Graceful shutdown qanday amalga oshiriladi va nega kerak?

**✅ Javob:** `SIGTERM` olinganda: yangi so'rovlarni qabul qilishni to'xtatish (`server.close`), davom etayotganlarini tugatish, DB/Redis/queue ulanishlarini yopish, keyin chiqish. Timeout bilan cheklash kerak (masalan 10s), aks holda osilib qoladi. Bu deploy va auto-scaling vaqtida so'rov yo'qotmaslik uchun muhim — aks holda foydalanuvchi 502 oladi (12-factor disposability).

### ❓ Observability'ning uch ustuni nima va ular qachon ishlatiladi?

**✅ Javob:** **Logs** — diskret hodisalar (xato sodir bo'ldi, nima bo'ldi); structured JSON ishlating. **Metrics** — agregatlangan sonlar (latency p99, error rate); dashboard va alert uchun. **Traces** — bitta so'rovning servislar bo'ylab yo'li; distributed sistemada bottleneck topish uchun. Correlation/trace ID barcha uchtasini bir so'rov atrofida bog'laydi.

### ❓ Structured logging nima va nega oddiy matn loglardan yaxshi?

**✅ Javob:** Structured logging log'ni JSON kabi mashina o'qiy oladigan formatda yozadi (`{"event":"order_created","orderId":"123","durationMs":42}`). Bu log agregator'da (ELK, Loki, Datadog) maydonlar bo'yicha qidirish, filtrlash va agregatsiya qilishni oson qiladi. Oddiy matn log'da `orderId=123`ni ajratib olish uchun regex kerak; JSON'da bu tabiiy maydon.

### ❓ Cascading failure nima va undan qanday himoyalanasiz?

**✅ Javob:** Cascading failure — bitta servisning yiqilishi yoki sekinlashishi unga bog'liq servislarni ham yiqitishi (zanjir reaksiya). Sync chaqiruvlar zanjirida bitta sekin servis butun zanjirni bloklab, thread/connection'larni to'ldiradi. Himoya: **timeout** (cheksiz kutmaslik), **circuit breaker** (yiqilgan servisga chaqiruvni to'xtatish), **bulkhead** (resurslarni izolyatsiya), **retry + backoff** (ehtiyotkorlik bilan).

### ❓ Message queue tizimga qanday foyda beradi?

**✅ Javob:** Uch asosiy foyda: (1) **Decoupling** — producer va consumer bir-birini bilmaydi, mustaqil deploy/scale; (2) **Load leveling** — trafik tepasida xabarlar buferlanadi, consumer o'z tezligida ishlaydi (DB'ni o'ldirmaydi); (3) **Reliability** — consumer yiqilsa, xabar broker'da saqlanib turadi, qayta urinish mumkin. Bundan tashqari fan-out (pub/sub) bilan bir eventni ko'p consumer qabul qiladi.

---

## Masalalar

> Yechimlar: [solutions/backend/09-architecture-patterns.md](../solutions/backend/09-architecture-patterns.md)

1. **Layered refactor.** Quyidagi "fat controller"ni controller / service / repository qatlamlariga ajrating: u `req`'dan `userId` oladi, SQL bilan to'g'ridan DB'dan o'qiydi, `if (user.banned)` qoidasini tekshiradi va `res.json` qaytaradi. Har qatlamning mas'uliyatini ko'rsating.

2. **Idempotent endpoint.** `POST /payments` endpointini `Idempotency-Key` header orqali idempotent qiling. Takroriy so'rov kelganda saqlangan natijani qaytarsin, race condition'da ham ikki marta to'lov bo'lmasin. Saqlash va to'lovni atomar qilishni tushuntiring.

3. **Saga loyihalash.** "Buyurtma berish" jarayonini saga sifatida loyihalashtiring: inventory reserve → payment charge → shipping schedule. Har qadam uchun compensating amalni yozing va 2-qadam muvaffaqiyatsiz bo'lganda nima sodir bo'lishini ko'rsating.

4. **Sync vs async qaror.** Quyidagi 4 ssenariyni sync (REST/gRPC) yoki async (messaging) deb belgilang va sababini yozing: (a) foydalanuvchi profilini ko'rsatish, (b) buyurtmadan keyin email yuborish, (c) to'lovni amalga oshirish, (d) analytics eventini yozish.

5. **Graceful shutdown.** Express + Postgres pool + Kafka consumer'li ilova uchun to'liq graceful shutdown yozing. `SIGTERM`'da yangi so'rovni to'xtatib, mavjudini tugatib, barcha ulanishlarni yopsin va 10 soniyalik forced-exit timeout qo'ysin.

6. **Config validatsiya.** `zod` (yoki o'xshash) bilan ilova start'ida environment variable'larni validatsiya qiluvchi config moduli yozing: `PORT` (raqam, default 3000), `DATABASE_URL` (url, majburiy), `NODE_ENV` (enum). Noto'g'ri config'da fail-fast bo'lsin.

7. **Monolith vs microservices tahlili.** 5 kishilik startup CRUD SaaS qurmoqda. Microservices'ga o'tish kerakmi? Argumentlaringizni trade-off jadval bilan asoslang va alternativ (modular monolith) tavsiya qiling.

8. **Cache-aside + invalidation.** `getProduct(id)` uchun cache-aside implementatsiya yozing (Redis, TTL 5 min). Keyin `updateProduct(id)` chaqirilganda stale data bermaslik uchun cache'ni qanday invalidate qilishni ko'rsating.

---

← [Backend bo'limiga qaytish](./README.md)
