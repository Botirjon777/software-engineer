# Arxitektura Patternlar — Masalalar Yechimi

Bu fayl [`backend/09-architecture-patterns.md`](../../backend/09-architecture-patterns.md) bo'limidagi "Masalalar" qismining yechimlari.

---

## 1. Layered refactor

**Maqsad:** "fat controller"ni uch qatlamga ajratish.

```ts
// === REPOSITORY — faqat data access ===
class UserRepository {
  constructor(private db: Pool) {}

  async findById(id: string): Promise<UserRow | null> {
    const { rows } = await this.db.query(
      "SELECT id, name, banned FROM users WHERE id = $1",
      [id]
    );
    return rows[0] ?? null; // SQL bu yerdan tashqariga chiqmaydi
  }
}

// === SERVICE — biznes logika ===
class UserService {
  constructor(private users: UserRepository) {}

  async getProfile(id: string): Promise<UserProfile> {
    const user = await this.users.findById(id);
    if (!user) throw new NotFoundError("user not found");
    if (user.banned) throw new ForbiddenError("user is banned"); // biznes qoida
    return { id: user.id, name: user.name };
  }
}

// === CONTROLLER — faqat HTTP ===
async function getUserHandler(req: Request, res: Response) {
  const profile = await userService.getProfile(req.params.id);
  res.json(profile);
  // req/res faqat shu yerda; SQL va `banned` qoidasi bu yerda emas
}
```

**Mas'uliyatlar:**
- **Controller** — `req`'dan inputni olish, service chaqirish, javob qaytarish. HTTP'dan boshqa hech narsa.
- **Service** — `banned` qoidasi, `NotFoundError`/`ForbiddenError` qarorlari, qaysi maydonlar ko'rinishi.
- **Repository** — SQL va DB ulanishi. Boshqa qatlamlar SQL ko'rmaydi.

---

## 2. Idempotent endpoint

```ts
async function createPayment(req: Request, res: Response) {
  const key = req.header("Idempotency-Key");
  if (!key) return res.status(400).json({ error: "Idempotency-Key required" });

  const result = await db.transaction(async (tx) => {
    // unique constraint key ustida — race condition'da ikkinchisi xato beradi
    const inserted = await tx.query(
      `INSERT INTO idempotency (key, status) VALUES ($1, 'in_progress')
       ON CONFLICT (key) DO NOTHING RETURNING key`,
      [key]
    );

    if (inserted.rowCount === 0) {
      // key allaqachon mavjud — saqlangan natijani qaytar
      const existing = await tx.query(
        "SELECT result FROM idempotency WHERE key = $1",
        [key]
      );
      return existing.rows[0].result;
    }

    const charge = await chargeProvider(req.body); // haqiqiy to'lov
    await tx.query(
      "UPDATE idempotency SET status = 'done', result = $2 WHERE key = $1",
      [key, charge]
    );
    return charge;
  });

  res.json(result);
}
```

**Atomarlik:** `INSERT ... ON CONFLICT DO NOTHING` + unique constraint ikki bir vaqtdagi so'rovdan faqat bittasini "yangi" deb hisoblaydi. Ikkinchisi `rowCount === 0` oladi va saqlangan natijani kutadi/qaytaradi. To'lov va saqlash bir tranzaksiyada — yarim holat qolmaydi.

---

## 3. Saga loyihalash (orchestration)

```ts
async function placeOrderSaga(order: Order) {
  // qadam 1
  const reservation = await inventory.reserve(order.items);

  let payment;
  try {
    payment = await payments.charge(order); // qadam 2
  } catch (e) {
    await inventory.release(reservation); // compensation 1
    throw new OrderFailedError("payment failed", { cause: e });
  }

  try {
    await shipping.schedule(order); // qadam 3
  } catch (e) {
    await payments.refund(payment); // compensation 2
    await inventory.release(reservation); // compensation 1
    throw new OrderFailedError("shipping failed", { cause: e });
  }

  return { orderId: order.id, status: "confirmed" };
}
```

**2-qadam (payment) muvaffaqiyatsiz bo'lganda:** to'lov amalga oshmadi, shuning uchun refund kerak emas. Faqat **compensation 1** ishlaydi — `inventory.release` zaxiralangan mahsulotni qaytaradi. Foydalanuvchiga xato qaytadi. Tizim eventual ravishda izchil holatga keladi (rezerv qaytarildi, hech narsa "osilib" qolmadi).

---

## 4. Sync vs async qaror

| Ssenariy | Tanlov | Sabab |
|----------|--------|-------|
| (a) Profilni ko'rsatish | **Sync (REST/gRPC)** | Foydalanuvchi javobni darhol kutadi; so'rov-javob tabiati. |
| (b) Buyurtmadan keyin email | **Async (messaging)** | Email darhol kerak emas; email servisi sekin/yiqilsa buyurtmani bloklamasligi kerak. |
| (c) To'lovni amalga oshirish | **Sync** | Natija (muvaffaqiyat/xato) darhol kerak — foydalanuvchiga ko'rsatiladi va keyingi qadam shunga bog'liq. |
| (d) Analytics event | **Async** | Fire-and-forget; yo'qolsa ham asosiy oqimga ta'sir qilmaydi, fan-out uchun ideal. |

Umumiy qoida: javob **darhol kerak** bo'lsa sync; background/decoupling/fan-out bo'lsa async.

---

## 5. Graceful shutdown

```ts
const server = app.listen(config.PORT);
let shuttingDown = false;

async function shutdown(signal: string) {
  if (shuttingDown) return;
  shuttingDown = true;
  logger.info({ event: "shutdown_started", signal });

  const forced = setTimeout(() => {
    logger.error({ event: "shutdown_forced" });
    process.exit(1);
  }, 10_000);
  forced.unref();

  server.close(async () => {
    try {
      await consumer.disconnect(); // yangi xabar olishni to'xtat
      await pool.end();            // Postgres pool
      logger.info({ event: "shutdown_complete" });
      clearTimeout(forced);
      process.exit(0);
    } catch (e) {
      logger.error({ event: "shutdown_error", error: (e as Error).message });
      process.exit(1);
    }
  });
}

process.on("SIGTERM", () => shutdown("SIGTERM"));
process.on("SIGINT", () => shutdown("SIGINT"));
```

`shuttingDown` flag takroriy signal'dan himoya qiladi. `server.close` mavjud so'rovlarni tugatib, yangisini qabul qilmaydi. Forced timeout `unref` bilan — u event loop'ni ushlab qolmaydi.

---

## 6. Config validatsiya

```ts
import { z } from "zod";

const ConfigSchema = z.object({
  PORT: z.coerce.number().int().positive().default(3000),
  DATABASE_URL: z.string().url(),
  NODE_ENV: z.enum(["development", "production", "test"]).default("development"),
});

function loadConfig() {
  const parsed = ConfigSchema.safeParse(process.env);
  if (!parsed.success) {
    // fail fast — noto'g'ri config bilan ishlamaymiz
    console.error("Invalid config:", parsed.error.flatten().fieldErrors);
    process.exit(1);
  }
  return parsed.data;
}

export const config = loadConfig();
```

`z.coerce.number()` env string'ni raqamga aylantiradi. `safeParse` + `process.exit(1)` — noto'g'ri config'da ilova start bo'lmaydi (fail fast), yashirin xato productionga chiqmaydi.

---

## 7. Monolith vs microservices tahlili

**Tavsiya: modular monolith. Microservices'ga o'tmang.**

| Mezon | 5 kishilik startup uchun |
|-------|---------------------------|
| Jamoa | Kichik — mustaqil deploy ehtiyoji yo'q |
| Operatsion narx | Microservices ko'p infra/monitoring/tracing talab qiladi — 5 kishi tortmaydi |
| Tranzaksiya | CRUD'da oddiy ACID yetarli; distributed tranzaksiya keraksiz murakkablik |
| Scaling | Hali scale muammosi yo'q; vertical/horizontal monolith yetadi |
| Tezlik | Bitta codebase'da feature tezroq chiqadi |

**Argument:** Microservices'ning haqiqiy foydasi (mustaqil jamoa deploylari, qism-darajali scaling) bu bosqichda yo'q, lekin narxi (tarmoq latency, eventual consistency, distributed debugging, ko'proq DevOps) darhol keladi. To'g'ri yo'l — **modular monolith**: kodni aniq modullarga (bounded context) bo'ling, lekin bitta deploy qiling. Kelajakda chegaralar oydinlashganda kerakli modulni ajratish oson bo'ladi.

---

## 8. Cache-aside + invalidation

```ts
const TTL = 300; // 5 daqiqa

async function getProduct(id: string): Promise<Product> {
  const key = `product:${id}`;
  const cached = await redis.get(key);
  if (cached) return JSON.parse(cached); // cache hit

  const product = await productRepo.findById(id); // cache miss
  if (product) await redis.set(key, JSON.stringify(product), "EX", TTL);
  return product;
}

async function updateProduct(id: string, data: Partial<Product>) {
  await productRepo.update(id, data); // avval DB
  await redis.del(`product:${id}`);   // keyin cache invalidate (write-through emas, delete)
}
```

**Invalidation strategiyasi:** `updateProduct`'da **avval DB yangilanadi, keyin cache O'CHIRILADI** (yangilanmaydi). O'chirish (`del`) yangilashdan (`set`) xavfsizroq — keyingi `getProduct` DB'dan yangi qiymatni o'qib qayta cache qiladi. Agar DB yangilashdan oldin cache'ni o'chirsangiz, oradagi `get` eski qiymatni qayta yozib qo'yishi (race) mumkin, shuning uchun tartib muhim: **DB → keyin cache del**.

---

← [Backend bo'limiga qaytish](../../backend/README.md)
