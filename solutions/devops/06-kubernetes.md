# Kubernetes — Yechimlar

Bu fayl "Kubernetes" mavzusidagi masalalar uchun to'liq yechimlarni o'z ichiga oladi.

## 1. 3 replica'dan biri o'ldi — Kubernetes reaksiyasi

Deployment 3 replica so'ragan, biri o'ldi. Kubernetes self-healing'ni quyidagi qadamlar bilan bajaradi:

1. **kubelet** (o'lgan Pod joylashgan node'da) Pod'ning ishdan chiqqanini sezadi va bu holatni **API Server**'ga xabar qiladi.
2. API Server klaster holatini **etcd**'da yangilaydi (endi 2 ta tirik Pod).
3. **Controller Manager** ichidagi ReplicaSet kontrolleri "kerakli holat = 3, joriy holat = 2" nomuvofiqligini sezadi.
4. ReplicaSet yangi Pod yaratish uchun API Server'ga so'rov yuboradi.
5. **Scheduler** yangi Pod qaysi node'da ishlashini hal qiladi (resurs va qoidalarga qarab).
6. Tanlangan node'dagi **kubelet** Pod'ni ishga tushiradi (container runtime orqali).
7. Pod tayyor (readiness probe o'tgan) bo'lgach, **Service** yana unga trafik yo'naltira boshlaydi.

**Ishtirok etuvchi komponentlar:** kubelet, API Server, etcd, Controller Manager (ReplicaSet controller), Scheduler, container runtime, Service.

**Izoh:** Asosi — "reconciliation loop": Kubernetes doimo joriy holatni kerakli holatga (3 replica) moslashtiradi. Bu inson aralashuvisiz avtomatik sodir bo'ladi.

## 2. Nginx uchun 4 replica Deployment + LoadBalancer Service YAML

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 4
  selector:
    matchLabels:
      app: nginx          # <- bu Pod'larni tanlaydi
  template:
    metadata:
      labels:
        app: nginx        # <- Pod'larga shu teg beriladi (selector bilan mos)
    spec:
      containers:
        - name: nginx
          image: nginx:1.25
          ports:
            - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: LoadBalancer
  selector:
    app: nginx            # <- Service shu tegli Pod'larga trafik yuboradi
  ports:
    - port: 80            # Service tashqi porti
      targetPort: 80      # Pod konteyner porti
```

**Label/selector bog'lanishi (muhim):**
- Deployment'ning `spec.selector.matchLabels` (`app: nginx`) Pod template'idagi `metadata.labels` (`app: nginx`) bilan **aynan mos** bo'lishi shart, aks holda Deployment xato beradi.
- Service'ning `selector` (`app: nginx`) ham shu tegga ishora qiladi — shu orqali Service 4 ta Pod'ni topib, ular orasida trafikni taqsimlaydi.

**Izoh:** `app: nginx` tegi uch joyda mos kelishi kerak (Deployment selector, Pod label, Service selector) — bu Kubernetes'da obyektlarni bog'laydigan markaziy mexanizm.

## 3. 30 soniya migratsiya bajaruvchi ilova uchun probe sozlash

**Muammo:** Ilova ishga tushganda 30 soniya database migratsiyasini bajaradi va shu vaqtda so'rovlarni qabul qila olmaydi.

**Yechim: Readiness probe** (liveness emas) ishlat.
- **Readiness probe** "Pod trafik qabul qilishga tayyormi?" deb tekshiradi. Migratsiya tugamaguncha probe false qaytaradi, shuning uchun Service bu Pod'ga trafik yubormaydi. Migratsiya tugab, ilova tayyor bo'lgach, probe true bo'ladi va trafik kela boshlaydi.

```yaml
readinessProbe:
  httpGet:
    path: /ready
    port: 8080
  initialDelaySeconds: 5    # birinchi tekshiruvni biroz kechiktir
  periodSeconds: 5          # har 5 soniyada tekshir
  failureThreshold: 8       # ~40s gacha tayyor bo'lishini kut (30s migratsiya + zaxira)
```

**Nega liveness emas?** Agar 30 soniyalik migratsiyaga liveness probe qo'ysang va `initialDelaySeconds` yetarli bo'lmasa, Kubernetes Pod'ni "o'lgan" deb hisoblab, migratsiya tugamasidan qayta ishga tushiradi — bu cheksiz restart loop'ga olib keladi. Liveness'ni qo'ymoqchi bo'lsang, `initialDelaySeconds` ni migratsiyadan kattaroq (masalan 35-40s) qilib, faqat haqiqiy osilib qolishni tekshirish uchun ishlat.

**Izoh:** Boshlang'ich sekin ishga tushish = readiness probe ishi. "Ilova osilib qoldimi" = liveness probe ishi. Ikkalasini adashtirmaslik kerak.

## 4. CPU'ga qarab 2-8 Pod avtomatik miqyoslash (HPA)

**Sozlash (buyruq bilan):**

```bash
kubectl autoscale deployment my-api --cpu-percent=60 --min=2 --max=8
```

Yoki YAML (declarative) ko'rinishida:

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: my-api-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-api
  minReplicas: 2
  maxReplicas: 8
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 60
```

**HPA qanday ishlaydi:**
1. HPA controller muntazam (odatda har 15 soniyada) Pod'larning o'rtacha CPU foizini metrics-server orqali o'qiydi.
2. O'rtacha CPU 60%'dan oshsa, kerakli Pod sonini hisoblaydi va Deployment'ning replica sonini oshiradi (8 gacha).
3. CPU 60%'dan ancha pasaysa, replica sonini kamaytiradi (2 dan past emas).
4. Tez-tez tebranib qolmaslik uchun "stabilization window" (masalan kamaytirishda kechikish) qo'llaniladi.

**Muhim shart:** HPA ishlashi uchun Pod'larda CPU `requests` belgilangan bo'lishi va klasterda **metrics-server** o'rnatilgan bo'lishi kerak (foiz `requests`ga nisbatan hisoblanadi).

**Izoh:** HPA Pod sonini (horizontal) o'zgartiradi; bitta Pod'ni kuchaytirmaydi. Bu trafikка avtomatik moslashish va resurslarni tejashni beradi.

## 5. dev/staging/production'ni ajratish

**Mexanizm: Namespace.**

Bitta klaster ichida `dev`, `staging`, `production` nomli uch Namespace yaratasan:

```bash
kubectl create namespace dev
kubectl create namespace staging
kubectl create namespace production
```

**Qanday foyda beradi:**
- **Izolyatsiya:** Har bir muhitning resurslari (Pod, Service, ConfigMap) alohida ajratilgan; nomlar to'qnashmaydi (har namespace'da `web-service` bo'lishi mumkin).
- **Resurs kvotalari:** Har namespace'ga CPU/xotira limiti (ResourceQuota) qo'yib, dev production resursini "yeb qo'ymasligini" ta'minlash.
- **Kirish nazorati (RBAC):** Kim qaysi namespace'ga kirishini cheklash — masalan, ishlab chiquvchilarga faqat `dev`, faqat ishonchli pipeline'ga `production`.
- **Tartib:** Resurslarni mantiqiy guruhlash, monitoring va tozalashni osonlashtirish.

**Izoh:** Bitta klaster + Namespace'lar — arzon va oddiy yechim. Yuqori xavfsizlik talab qilinsa, production uchun butunlay alohida klaster ham ishlatiladi, lekin ko'p holatda Namespace yetarli.

## 6. Kubernetes'da PostgreSQL — ma'lumotni yo'qotmaslik

**Muammo:** Pod o'lsa yoki qayta ishga tushsa, uning ichidagi ma'lumot yo'qoladi (Pod ephemeral). Database uchun bu qabul qilib bo'lmas.

**Foydalaniladigan obyektlar:**
- **PersistentVolume (PV):** Klaster darajasidagi doimiy disk (cloud'da masalan AWS EBS, GCP PD). Pod hayotidan mustaqil.
- **PersistentVolumeClaim (PVC):** Pod'ning PV'ga "menga shuncha doimiy disk kerak" so'rovi.
- **StatefulSet:** Database kabi holatli (stateful) ilovalar uchun Deployment o'rniga ishlatiladi — barqaror nom, barqaror storage, tartibli ishga tushish beradi.

**Arxitektura:**

```text
StatefulSet (postgres)
   │
   ▼
  Pod (postgres) ──ulanadi──> PVC ──bog'lanadi──> PV (doimiy disk, EBS)
                                                   │
                          Pod o'lsa/ko'chsa ham disk saqlanadi,
                          yangi Pod o'sha PVC/PV ga qayta ulanadi.
```

Pod konteyneri PostgreSQL ma'lumot katalogini (`/var/lib/postgresql/data`) PVC orqali doimiy diskка mount qiladi. Pod qayta ishga tushganda yoki boshqa node'ga ko'chganda, o'sha PV qayta ulanadi va ma'lumot saqlanib qoladi.

**Izoh:** Stateless ilovalar uchun Deployment, stateful (database) uchun StatefulSet + PVC/PV. Qo'shimcha himoya uchun muntazam backup ham kerak — PV o'zi backup o'rnini bosmaydi.

## 7. Xato versiyani tezda orqaga qaytarish (rollback)

Yangi versiya xato ishlayapti — Deployment'ning rollback imkoniyatidan foydalanasan:

```bash
# Oldingi ishlaydigan versiyaga darhol qaytish
kubectl rollout undo deployment/my-app

# Yoki muayyan reviziyaga qaytish
kubectl rollout history deployment/my-app      # tarixni ko'r
kubectl rollout undo deployment/my-app --to-revision=3

# Holatni kuzatish
kubectl rollout status deployment/my-app
```

**Nima sodir bo'ladi:** Deployment oldingi ReplicaSet'ni saqlab qo'ygan. `rollout undo` rolling update'ni teskari yo'nalishda bajaradi — eski (ishlaydigan) versiyali Pod'larni asta tiklab, yangi (xato) Pod'larni o'chiradi. Har doim ishlaydigan Pod'lar mavjud bo'lgani uchun foydalanuvchilar uzilishni sezmaydi.

**Izoh:** Bu Deployment'ning kalit afzalligi. Rollback tez va xavfsiz bo'lishi uchun har deploy alohida revizion sifatida saqlanadi. Kelajakda probe'lar (readiness) yomon versiyani umuman trafikка chiqarmasligi uchun ham foydali.

## 8. `/users` va `/orders` ni bitta tashqi IP orqali — Ingress

**Obyekt: Ingress.** Bitta tashqi IP orqali URL yo'liga qarab turli Service'larga yo'naltiradi.

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: api-ingress
spec:
  rules:
    - host: api.example.com
      http:
        paths:
          - path: /users
            pathType: Prefix
            backend:
              service:
                name: users-service
                port:
                  number: 80
          - path: /orders
            pathType: Prefix
            backend:
              service:
                name: orders-service
                port:
                  number: 80
```

**Diagramma:**

```text
                              ┌─────────────────┐
 api.example.com/users  ────► │                 │ ──► users-service ──► [users Pod'lar]
                              │     Ingress     │
 api.example.com/orders ────► │  (bitta IP)     │ ──► orders-service ──► [orders Pod'lar]
                              └─────────────────┘
```

**Izoh:** Ingress'siz har xizmatga alohida LoadBalancer (alohida tashqi IP, qimmat) kerak bo'lardi. Ingress bitta kirish nuqtasi bilan ko'plab Service'ni boshqaradi va SSL/TLS'ni ham markazlashtiradi. Ishlashi uchun klasterda Ingress Controller (masalan nginx-ingress) o'rnatilgan bo'lishi kerak.

## 9. Bitta web-ilovasi bor startap'ga Kubernetes maslahati

**Maslahat: Kubernetes ishlatma (hozircha kerak emas).**

**Sabab:** Ularda atigi bitta oddiy web-ilova bor. Kubernetes'ning kuchi — ko'p mikroservis, yuqori miqyoslash va murakkab deploy'ni boshqarish. Bitta ilova uchun K8s ortiqcha murakkablik (overkill):
- **Operatsion yuk:** Klaster, node, networking, monitoring'ni boshqarish maxsus bilim va vaqt talab qiladi.
- **Xarajat:** Control plane va minimal node'lar doimo pul yeydi — bitta ilova uchun isrof.
- **Sekinlik:** Kichik jamoa K8s o'rganishga vaqt sarflab, mahsulotni rivojlantirishdan orqada qoladi.

**Tavsiya etiladigan soddaroq variantlar:**
- **PaaS:** Heroku, Render, Railway, Fly.io — kodni push qilasan, qolganini platforma qiladi.
- **Bitta VM + Docker:** Oddiy va arzon.
- **Serverless / konteyner xizmati:** AWS App Runner, Google Cloud Run — konteynerni boqmasdan ishga tushiradi, avtomatik miqyoslaydi.

**Qachon K8s'ga o'tish kerak:** ilova bir nechta mikroservisga bo'linganda, yuqori va o'zgaruvchan trafik paydo bo'lganda, murakkab deploy/scaling ehtiyoji tug'ilganda.

**Izoh:** "Boshqalar ishlatyapti" — yetarli sabab emas. To'g'ri texnologiya = ehtiyojga mos texnologiya. Kichik startap uchun soddalik tezlik va arzonlik beradi.

## 10. Database paroli va API URL'ni ilovaga uzatish

**Maxfiy va maxfiy bo'lmagan ma'lumotni ajratish:**
- **API URL** (maxfiy emas) -> **ConfigMap**.
- **Database paroli** (maxfiy) -> **Secret**.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  API_URL: "https://api.example.com"
---
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
type: Opaque
data:
  DB_PASSWORD: c3VwZXJzZWNyZXQ=   # base64 ("supersecret")
```

Pod'da ularni muhit o'zgaruvchisi sifatida ulash:

```yaml
spec:
  containers:
    - name: app
      image: my-app:1.0
      env:
        - name: API_URL
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: API_URL
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: app-secret
              key: DB_PASSWORD
```

**Xavfsizlik nuqtai nazaridan e'tibor:**
- **base64 shifrlash EMAS** — Secret faqat kodlangan, oson dekodlanadi. Buni "yashirin" deb hisoblama.
- **Secret'larni Git'ga yuborma** — parolni kodга/repoga qo'yma.
- **Encryption at rest** yoq — etcd'da Secret'lar shifrlangan saqlanishini ta'minla (provayder yoki KMS bilan).
- **External secret manager** ishlat (AWS Secrets Manager, HashiCorp Vault, External Secrets Operator) — parollarni K8s'dan tashqarida, xavfsiz boshqarish.
- **RBAC** — kim Secret'larni o'qiy olishini cheklab qo'y.

**Izoh:** Asosiy qoida — maxfiy = Secret, maxfiy emas = ConfigMap; lekin Secret'ning o'zi yetarli himoya emas, shuning uchun encryption at rest + external secret manager + RBAC bilan kuchaytiriladi.

---

← [DevOps bo'limiga qaytish](../../devops/README.md)
