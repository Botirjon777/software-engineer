# Kubernetes

Bu hujjat Kubernetes (qisqacha K8s) — konteynerlarni orkestratsiya qilish tizimini o'zbek dasturchilari uchun tushuntiradi. Docker bilan ilovani konteynerga joylashtirishni bilsang, keyingi savol shu konteynerlarni production'da minglab nusxada qanday boshqarishdir. Aynan shu yerda Kubernetes kerak bo'ladi.

**💡 Tushuncha:** Orkestratsiya — bu ko'plab konteynerlarni avtomatik ravishda joylashtirish, miqyoslash, sog'lig'ini tekshirish va yangilashni boshqaradigan tizim. "Dirijyor" orkestrni boshqargani kabi, Kubernetes konteynerlarni boshqaradi.

## Mundarija

- [Orkestratsiya nima va nega kerak](#orkestratsiya-nima-va-nega-kerak)
- [Kubernetes arxitekturasi](#kubernetes-arxitekturasi)
- [Pod — eng kichik birlik](#pod--eng-kichik-birlik)
- [ReplicaSet](#replicaset)
- [Deployment](#deployment)
- [Service](#service)
- [Ingress](#ingress)
- [ConfigMap va Secret](#configmap-va-secret)
- [Namespace](#namespace)
- [Volume va PersistentVolume](#volume-va-persistentvolume)
- [kubectl asosiy buyruqlar](#kubectl-asosiy-buyruqlar)
- [Declarative YAML misol](#declarative-yaml-misol)
- [Self-healing va health checks](#self-healing-va-health-checks)
- [Horizontal Pod Autoscaler](#horizontal-pod-autoscaler)
- [Label va Selector](#label-va-selector)
- [Qachon Kubernetes kerak emas](#qachon-kubernetes-kerak-emas)
- [Savol-javoblar](#savol-javoblar)
- [Masalalar](#masalalar)

## Orkestratsiya nima va nega kerak

Docker bilan sen bitta ilovani konteynerga joylashtirib, bitta serverda ishga tushira olasan. Lekin production'da quyidagi savollar paydo bo'ladi:

- Trafik oshsa, konteynerni 10 nusxada qanday ishlataman? (**scaling**)
- Konteyner ishdan chiqsa, uni kim qayta ishga tushiradi? (**self-healing**)
- Yangi versiyaga to'xtatmasdan qanday o'taman? (**rolling update**)
- 50 ta server o'rtasida konteynerlarni qanday taqsimlayman? (**scheduling**)

Docker o'zi bularni hal qila olmaydi. Kubernetes — aynan shu muammolarni yechadigan orkestratsiya tizimi. U konteynerlarni avtomatik joylashtiradi, sog'lig'ini kuzatadi, miqyoslaydi va yangilaydi.

**💡 Tushuncha:** Docker — bitta konteyner haqida. Kubernetes — minglab konteyner, ko'plab serverlar va ularning hamkorligi haqida.

## Kubernetes arxitekturasi

Kubernetes klasteri ikki qismdan iborat: **Control Plane** (boshqaruv) va **Worker Node**'lar (ish bajaruvchilar).

```text
┌──────────────────────── CONTROL PLANE ────────────────────────┐
│                                                                │
│  ┌────────────┐  ┌────────┐  ┌───────────┐  ┌──────────────┐   │
│  │ API Server │  │  etcd  │  │ Scheduler │  │  Controller  │   │
│  │ (kirish    │  │ (holat │  │ (Pod'larni│  │   Manager    │   │
│  │  nuqtasi)  │  │  bazasi)│ │  joylash) │  │ (nazorat)    │   │
│  └────────────┘  └────────┘  └───────────┘  └──────────────┘   │
└────────────────────────────┬───────────────────────────────────┘
                             │
        ┌────────────────────┼────────────────────┐
        ▼                    ▼                     ▼
┌───────────────┐   ┌───────────────┐    ┌───────────────┐
│  WORKER NODE  │   │  WORKER NODE  │    │  WORKER NODE  │
│  ┌─────────┐  │   │  ┌─────────┐  │    │  ┌─────────┐  │
│  │ kubelet │  │   │  │ kubelet │  │    │  │ kubelet │  │
│  │kube-proxy│ │   │  │kube-proxy│ │    │  │kube-proxy│ │
│  │ runtime │  │   │  │ runtime │  │    │  │ runtime │  │
│  │ [Pods]  │  │   │  │ [Pods]  │  │    │  │ [Pods]  │  │
│  └─────────┘  │   │  └─────────┘  │    │  └─────────┘  │
└───────────────┘   └───────────────┘    └───────────────┘
```

**Control Plane komponentlari:**

- **API Server:** Klasterning yagona kirish nuqtasi. `kubectl` va boshqa komponentlar shu bilan gaplashadi.
- **etcd:** Klasterning butun holatini saqlaydigan kalit-qiymat bazasi. "Klaster xotirasi".
- **Scheduler:** Yangi Pod qaysi node'da ishlashini hal qiladi (resurs, qoidalarga qarab).
- **Controller Manager:** Joriy holatni kerakli holatga moslashtiradi (masalan, Pod o'lsa yangisini yaratadi).

**Worker Node komponentlari:**

- **kubelet:** Node'dagi agent. Pod'larning ishlashini ta'minlaydi va API Server bilan bog'lanadi.
- **kube-proxy:** Tarmoq qoidalarini boshqaradi, Service'lar ishlashini ta'minlaydi.
- **Container Runtime:** Konteynerlarni haqiqatda ishga tushiradi (containerd, CRI-O).

## Pod — eng kichik birlik

**Pod** — Kubernetes'dagi eng kichik joylashtirish birligi. Pod bir yoki bir nechta konteynerni o'z ichiga oladi, ular bir xil tarmoq va storage'ni bo'lishadi.

**💡 Tushuncha:** Odatda bir Pod = bir asosiy konteyner. Lekin ba'zan yordamchi "sidecar" konteyner ham qo'shiladi (masalan log yig'uvchi). Pod konteynerlari `localhost` orqali bir-biriga ulanadi.

**⚠️ Ehtiyot bo'l:** Pod'lar "vaqtinchalik" (ephemeral). Ular o'lishi va yangisi yaratilishi mumkin, har safar yangi IP oladi. Shuning uchun Pod'ga to'g'ridan-to'g'ri murojaat qilma — Service ishlat.

## ReplicaSet

**ReplicaSet** — belgilangan sondagi bir xil Pod nusxalarini ("replica") doimo ishlab turishini ta'minlaydi. Agar Pod o'lsa, ReplicaSet yangisini yaratadi. Sen 3 ta replica so'rasang, ReplicaSet doimo 3 tasini saqlaydi.

Amalda sen ReplicaSet'ni to'g'ridan-to'g'ri ishlatmaysan — uni Deployment boshqaradi.

## Deployment

**Deployment** — eng ko'p ishlatiladigan obyekt. U ReplicaSet ustida ishlaydi va quyidagilarni beradi:

- **Rolling update:** Yangi versiyani Pod'larni bittalab almashtirib, to'xtatmasdan joriy qilish.
- **Rollback:** Yangilanish muvaffaqiyatsiz bo'lsa, oldingi versiyaga qaytish.
- **Scaling:** Replica sonini oson o'zgartirish.

```text
Rolling Update jarayoni:
v1 v1 v1   →   v2 v1 v1   →   v2 v2 v1   →   v2 v2 v2
(eski)         (1 ta yangi)    (2 ta yangi)    (hammasi yangi)
```

**💡 Tushuncha:** Rolling update tufayli foydalanuvchilar uzilishni sezmaydi — har doim ishlayotgan Pod'lar mavjud bo'ladi.

## Service

Pod'lar o'zgaruvchan IP'ga ega bo'lgani uchun, ularga barqaror manzil kerak. **Service** — bir guruh Pod'ga doimiy IP va DNS nomi beradi va trafikni ular o'rtasida taqsimlaydi.

Service turlari:

- **ClusterIP:** Faqat klaster ichida ko'rinadi (standart). Mikroservislar o'zaro shu bilan gaplashadi.
- **NodePort:** Node'ning aniq portida xizmatni tashqariga ochadi. Oddiy test uchun.
- **LoadBalancer:** Cloud provayder load balancer'ini yaratib, tashqi IP beradi. Production'da tashqi kirish uchun.

```text
              Service (barqaror IP)
                     │
        ┌────────────┼────────────┐
        ▼            ▼            ▼
     Pod (IP1)   Pod (IP2)    Pod (IP3)
     o'zgaruvchan  o'zgaruvchan  o'zgaruvchan
```

## Ingress

**Ingress** — HTTP/HTTPS trafikni domen va URL yo'liga qarab turli Service'larga yo'naltiruvchi qatlam. Bitta tashqi IP orqali ko'plab xizmatni boshqarish imkonini beradi.

```text
                    ┌─────────────┐
  example.com/api → │             │ → api-service
                    │   Ingress   │
  example.com/web → │             │ → web-service
                    └─────────────┘
```

**💡 Tushuncha:** Ingress'siz har bir xizmat uchun alohida LoadBalancer kerak bo'lardi (qimmat). Ingress bitta kirish nuqtasi bilan ko'plab xizmatni boshqaradi va SSL'ni ham hal qiladi.

## ConfigMap va Secret

Ilovaning konfiguratsiyasini koddan ajratish kerak.

- **ConfigMap:** Maxfiy bo'lmagan konfiguratsiya (URL'lar, sozlamalar, muhit o'zgaruvchilari).
- **Secret:** Maxfiy ma'lumot (parol, API kalit, token). Base64'da kodlanadi.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  LOG_LEVEL: "info"
  API_URL: "https://api.example.com"
```

**⚠️ Ehtiyot bo'l:** Secret base64'da kodlanadi, lekin bu shifrlash EMAS. Production'da Secret'larni Git'ga yuborma va shifrlashni (encryption at rest, external secret manager) yoq.

## Namespace

**Namespace** — bitta klaster ichidagi resurslarni mantiqiy guruhlarga ajratish. Masalan `dev`, `staging`, `production` namespace'lari. Har biri o'z resurslari, kvotalari va kirish huquqlariga ega bo'lishi mumkin.

```text
Klaster
├── namespace: dev         (test resurslari)
├── namespace: staging     (sinov muhiti)
└── namespace: production  (haqiqiy ishlab chiqarish)
```

## Volume va PersistentVolume

Pod o'lsa, uning ichidagi ma'lumot yo'qoladi. Doimiy ma'lumot uchun:

- **Volume:** Pod hayoti davomida saqlanadigan storage. Pod o'lsa yo'qolishi mumkin.
- **PersistentVolume (PV):** Klaster darajasidagi doimiy storage resursi (disk).
- **PersistentVolumeClaim (PVC):** Pod tomonidan PV'ga "so'rov". Pod PVC orqali doimiy diskka ulanadi.

**💡 Tushuncha:** Database kabi ma'lumot saqlaydigan ilovalar PersistentVolume ishlatishi shart, aks holda Pod qayta ishga tushganda ma'lumot yo'qoladi.

## kubectl asosiy buyruqlar

`kubectl` — Kubernetes bilan ishlash uchun asosiy buyruq qatori vositasi.

```bash
# Resurslarni ko'rish
kubectl get pods                    # Pod'lar ro'yxati
kubectl get services                # Service'lar ro'yxati
kubectl get deployments             # Deployment'lar

# Batafsil ma'lumot
kubectl describe pod my-pod         # Pod tafsilotlari
kubectl logs my-pod                 # Pod loglari

# YAML qo'llash va o'chirish
kubectl apply -f deployment.yaml    # YAML'ni qo'llash
kubectl delete -f deployment.yaml   # O'chirish

# Scaling va debug
kubectl scale deployment my-app --replicas=5
kubectl exec -it my-pod -- /bin/sh  # Pod ichiga kirish
```

## Declarative YAML misol

Quyida nginx ilovasi uchun Deployment va Service misoli. Declarative yondashuvda sen "kerakli holatni" yozasan, Kubernetes uni amalga oshiradi.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
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
    app: nginx
  ports:
    - port: 80
      targetPort: 80
```

**💡 Tushuncha:** `selector` va `labels` orqali Service qaysi Pod'larga trafik yuborishini biladi. Bu yerda `app: nginx` tegli barcha Pod'lar Service'ga ulanadi.

## Self-healing va health checks

Kubernetes Pod'lar sog'lig'ini doimiy kuzatadi va muammoli Pod'larni avtomatik tuzatadi (self-healing). Buning uchun ikki turdagi probe ishlatiladi:

- **Liveness probe:** "Pod tirikmi?" Agar muvaffaqiyatsiz bo'lsa, Kubernetes Pod'ni qayta ishga tushiradi.
- **Readiness probe:** "Pod trafik qabul qilishga tayyormi?" Tayyor bo'lmaguncha Service unga trafik yubormaydi.

```yaml
containers:
  - name: app
    image: my-app:1.0
    livenessProbe:
      httpGet:
        path: /healthz
        port: 8080
      initialDelaySeconds: 10
      periodSeconds: 5
    readinessProbe:
      httpGet:
        path: /ready
        port: 8080
      initialDelaySeconds: 5
      periodSeconds: 5
```

**⚠️ Ehtiyot bo'l:** Liveness va readiness'ni adashtirma. Agar database'ga ulanmagan ilovaga liveness probe qo'ysang, u doimiy qayta ishga tushib turadi. Bunday holatda readiness probe to'g'riroq.

## Horizontal Pod Autoscaler

**HPA (Horizontal Pod Autoscaler)** — yukка (masalan CPU foiziga) qarab Pod sonini avtomatik o'zgartiradi.

```bash
# CPU 50%'da ushlab, 2 dan 10 gacha miqyoslash
kubectl autoscale deployment my-app --cpu-percent=50 --min=2 --max=10
```

CPU 50%'dan oshsa HPA Pod qo'shadi, pasaysa kamaytiradi. Bu trafik o'zgarishlariga avtomatik moslashishni ta'minlaydi.

## Label va Selector

**Label** — resurslarga biriktiriladigan kalit-qiymat teglari (masalan `app: nginx`, `env: prod`). **Selector** — shu teglar bo'yicha resurslarni tanlash mexanizmi.

```text
Pod'lar:                          Selector: app=web
┌──────────────┐
│ app=web      │ ◄── tanlanadi
│ app=web      │ ◄── tanlanadi
│ app=database │     tanlanmaydi
└──────────────┘
```

Service, Deployment, ReplicaSet — hammasi label/selector orqali bir-biri bilan bog'lanadi. Bu Kubernetes'ning markaziy mexanizmi.

## Qachon Kubernetes kerak emas

Kubernetes kuchli, lekin murakkab. Quyidagi holatlarda u ortiqcha bo'lishi mumkin:

- **Kichik loyiha:** Bitta yoki ikkita konteyner bo'lsa — oddiy VM yoki PaaS (Heroku, Render) yetarli.
- **Kichik jamoa:** K8s'ni boshqarish uchun maxsus bilim va vaqt kerak.
- **Oddiy statik sayt:** CDN yoki static hosting yetarli.
- **Serverless mos kelsa:** Ba'zi vazifalar uchun Lambda/Cloud Functions soddaroq.

**💡 Tushuncha:** "Boshqalar ishlatyapti" degani sening loyihangga ham kerak degani emas. Kubernetes haqiqiy ehtiyoj (ko'p mikroservis, yuqori miqyoslash, murakkab deploy) bo'lgandagina o'zini oqlaydi.

## Savol-javoblar

### ❓ Orkestratsiya nima va nega Docker o'zi yetarli emas?

**✅ Javob:** Orkestratsiya — ko'plab konteynerlarni avtomatik joylashtirish, miqyoslash, sog'lig'ini tekshirish va yangilashni boshqarish. Docker bitta konteynerni bitta serverda ishga tushiradi, lekin scaling (ko'p nusxa), self-healing (o'lgan konteynerni qayta yaratish), rolling update va ko'p server o'rtasida taqsimlashni o'zi qila olmaydi. Kubernetes aynan shularni hal qiladi.

### ❓ Kubernetes arxitekturasini tushuntir — Control Plane va Worker Node nima?

**✅ Javob:** Control Plane klasterni boshqaradi: API Server (kirish nuqtasi), etcd (holat bazasi), Scheduler (Pod'larni joylash), Controller Manager (kerakli holatni saqlash). Worker Node'lar haqiqiy ishni bajaradi: kubelet (node agenti), kube-proxy (tarmoq), container runtime (konteynerlarni ishlatadi). Control Plane "miya", Worker Node'lar "qo'l".

### ❓ etcd nima va u nima uchun muhim?

**✅ Javob:** etcd — Kubernetes klasterining butun holatini saqlaydigan kalit-qiymat bazasi. Qaysi Pod'lar mavjud, ularning konfiguratsiyasi, Secret'lar — hammasi shu yerda. Bu "klaster xotirasi". etcd yo'qolsa, klaster o'z holatini bilmaydi, shuning uchun u backup qilinishi va yuqori ishonchli bo'lishi shart.

### ❓ Pod nima va nega u eng kichik birlik?

**✅ Javob:** Pod — Kubernetes'dagi eng kichik joylashtirish birligi, bir yoki bir nechta konteynerni o'rab oladi, ular bir xil tarmoq va storage'ni bo'lishadi. Kubernetes konteynerni emas, Pod'ni boshqaradi. Pod'lar vaqtinchalik — o'lishi va yangi IP bilan qayta yaratilishi mumkin, shuning uchun ularga to'g'ridan-to'g'ri murojaat qilinmaydi.

### ❓ ReplicaSet va Deployment o'rtasidagi farq nima?

**✅ Javob:** ReplicaSet belgilangan sondagi Pod nusxalarini doimo saqlaydi (Pod o'lsa yangisini yaratadi). Deployment esa ReplicaSet ustida ishlaydi va qo'shimcha imkoniyatlar beradi: rolling update (to'xtatmasdan yangilash), rollback (oldingi versiyaga qaytish), oson scaling. Amalda sen ReplicaSet'ni to'g'ridan-to'g'ri emas, Deployment orqali ishlatasan.

### ❓ Service nima va uning turlarini ayt.

**✅ Javob:** Service bir guruh Pod'ga barqaror IP va DNS nomi beradi, trafikni ular o'rtasida taqsimlaydi (Pod IP'lari o'zgaruvchan bo'lgani uchun). Turlari: ClusterIP (faqat klaster ichida, standart), NodePort (node portida tashqariga ochadi), LoadBalancer (cloud load balancer bilan tashqi IP beradi). ClusterIP mikroservislar uchun, LoadBalancer tashqi kirish uchun.

### ❓ Ingress nima va u Service'dan nimasi bilan farq qiladi?

**✅ Javob:** Service Pod'larga trafik taqsimlaydi (4-qatlam, IP/port). Ingress esa HTTP/HTTPS trafikni domen va URL yo'liga qarab turli Service'larga yo'naltiradi (7-qatlam). Ingress bitta tashqi IP orqali ko'plab xizmatni boshqaradi va SSL'ni hal qiladi, har bir xizmatga alohida LoadBalancer kerak bo'lmaydi.

### ❓ ConfigMap va Secret o'rtasidagi farq nima?

**✅ Javob:** Ikkalasi ham konfiguratsiyani koddan ajratadi. ConfigMap maxfiy bo'lmagan ma'lumot uchun (URL, sozlamalar, muhit o'zgaruvchilari). Secret maxfiy ma'lumot uchun (parol, API kalit, token) va base64'da kodlanadi. Muhim: base64 shifrlash emas, shuning uchun Secret'lar uchun qo'shimcha himoya (encryption at rest, external secret manager) kerak.

### ❓ Namespace nima uchun kerak?

**✅ Javob:** Namespace bitta klaster ichidagi resurslarni mantiqiy guruhlarga ajratadi, masalan `dev`, `staging`, `production`. Har biri o'z resurslari, resurs kvotalari va kirish huquqlariga ega bo'lishi mumkin. Bu turli muhitlar yoki jamoalarni bir klasterda izolyatsiya qilish imkonini beradi.

### ❓ Pod ma'lumotni qanday doimiy saqlaydi?

**✅ Javob:** Pod o'lsa uning ichidagi ma'lumot yo'qoladi. Doimiy saqlash uchun PersistentVolume (PV) — klaster darajasidagi doimiy disk — va PersistentVolumeClaim (PVC) — Pod'ning PV'ga so'rovi ishlatiladi. Database kabi ilovalar PVC orqali doimiy diskka ulanishi shart, aks holda qayta ishga tushganda ma'lumot yo'qoladi.

### ❓ Self-healing qanday ishlaydi?

**✅ Javob:** Kubernetes Pod'lar sog'lig'ini doimiy kuzatadi. Pod o'lsa yoki liveness probe muvaffaqiyatsiz bo'lsa, uni avtomatik qayta ishga tushiradi. Agar butun node ishdan chiqsa, Pod'lar boshqa node'larga ko'chiriladi. ReplicaSet doimo kerakli replica sonini saqlaydi. Bularning hammasi inson aralashuvisiz amalga oshadi.

### ❓ Liveness va readiness probe o'rtasidagi farq nima?

**✅ Javob:** Liveness probe "Pod tirikmi?" deb tekshiradi — muvaffaqiyatsiz bo'lsa Kubernetes Pod'ni qayta ishga tushiradi. Readiness probe "Pod trafik qabul qilishga tayyormi?" deb tekshiradi — tayyor bo'lmaguncha Service unga trafik yubormaydi. Masalan, ilova ishga tushyapti lekin hali database'ga ulanmagan bo'lsa, readiness false bo'ladi.

### ❓ Rolling update qanday ishlaydi va rollback nima?

**✅ Javob:** Rolling update yangi versiyani Pod'larni bittalab almashtirib joriy qiladi — har doim ishlayotgan Pod'lar mavjud bo'ladi, foydalanuvchilar uzilishni sezmaydi. Agar yangi versiya muammoli bo'lsa, rollback bilan oldingi ishlaydigan versiyaga qaytariladi. Bu Deployment'ning asosiy afzalligi.

### ❓ Horizontal Pod Autoscaler (HPA) nima?

**✅ Javob:** HPA yukка (masalan CPU foiziga) qarab Pod sonini avtomatik o'zgartiradi. Masalan CPU 50%'dan oshsa Pod qo'shiladi, pasaysa kamaytiriladi, belgilangan min va max chegarasida. Bu trafik o'zgarishlariga avtomatik moslashish va resurslarni tejashni ta'minlaydi.

### ❓ Label va Selector qanday ishlaydi?

**✅ Javob:** Label — resurslarga biriktiriladigan kalit-qiymat teglari (`app: nginx`). Selector — shu teglar bo'yicha resurslarni tanlash mexanizmi. Service `selector: app=nginx` orqali shu teglga ega Pod'larni topadi va ularga trafik yuboradi. Bu Kubernetes'da obyektlar (Service, Deployment, ReplicaSet) bir-biri bilan bog'lanadigan markaziy mexanizm.

### ❓ Qachon Kubernetes ishlatmaslik kerak?

**✅ Javob:** Kubernetes murakkab, shuning uchun kichik loyihalar (1-2 konteyner), kichik jamoalar (boshqarish uchun resurs yo'q), oddiy statik saytlar yoki serverless mos keladigan vazifalar uchun ortiqcha. Bunday holatlarda oddiy VM, PaaS (Heroku, Render) yoki serverless soddaroq va arzonroq. K8s haqiqiy ehtiyoj (ko'p mikroservis, yuqori miqyoslash) bo'lgandagina o'zini oqlaydi.

## Masalalar

> Yechimlar: [yechimlar fayli](../solutions/devops/06-kubernetes.md)

1. Bir mikroservis ilovangda 3 ta replica ishlamoqda. Bittasi to'satdan o'ldi. Kubernetes qanday qadamlar bilan reaksiya qiladi? Qaysi komponentlar ishtirok etishini tushuntir.

2. Nginx ilovasi uchun 4 replica'li Deployment va uni tashqariga ochadigan LoadBalancer Service yozadigan YAML tuz. Label va selector to'g'ri bog'langaniga e'tibor ber.

3. Sening ilovang ishga tushganda 30 soniya database migratsiyasini bajaradi va shu vaqtda so'rovlarni qabul qila olmaydi. Qaysi probe'ni qanday sozlaysan, nega?

4. CPU yukiga qarab API'ni 2 dan 8 Pod'gacha avtomatik miqyoslamoqchisan, CPU maqsadi 60%. Buni qanday sozlaysan va HPA qanday ishlashini tushuntir.

5. Bir klasterda `dev`, `staging` va `production` muhitlarini bir-biridan ajratish kerak. Qaysi Kubernetes mexanizmidan foydalanasan va u qanday foyda beradi?

6. PostgreSQL database'ni Kubernetes'da ishga tushirmoqchisan. Pod qayta ishga tushganda ma'lumot yo'qolmasligi uchun qaysi obyektlardan foydalanasan? Arxitekturani tushuntir.

7. Yangi versiyani joriy qilding, lekin u xatolik bilan ishlamayapti. Foydalanuvchilarga ta'sir qilmasdan tezda oldingi holatga qaytish uchun nima qilasan?

8. Ikkita xizmating bor: `api.example.com/users` va `api.example.com/orders`. Ularni bitta tashqi IP orqali boshqarish uchun qaysi obyektni qanday sozlaysan? Diagramma chiz.

9. Kichik startap senga "biz ham Kubernetes ishlatamiz" deb murojaat qildi, lekin ularda atigi bitta oddiy web-ilova bor. Sen qanday maslahat berasan va nega?

10. Database paroli va API URL'ni ilovaga qanday uzatasan? Maxfiy va maxfiy bo'lmagan ma'lumotni qanday ajratasan? Tegishli obyektlarni ayt va xavfsizlik nuqtai nazaridan e'tibor ber.

← [DevOps bo'limiga qaytish](./README.md)
