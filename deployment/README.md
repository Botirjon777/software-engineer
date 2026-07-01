# Deployment — Loyihani Ishga Tushirish (Amaliy Qo'llanma)

Kod yozish yarim ish — uni **internetga chiqarish** (deploy qilish) ikkinchi yarmi. Bu bo'lim to'liq amaliy: static saytdan tortib VPS'da to'liq backend + ma'lumotlar bazasi + domain + SSL gacha, real buyruqlar bilan.

## Mundarija

| # | Mavzu | Nima o'rganasiz |
|---|-------|-----------------|
| 01 | [Frontend Deploy (Static, React, Next.js)](./01-frontend-deploy.md) | HTML/CSS, Vite/React, Next.js — Vercel/Netlify/Pages |
| 02 | [Domain va DNS](./02-domains-dns-ssl.md) | Domen sotib olish, DNS record, ulash, SSL/HTTPS |
| 03 | [SEO'ni To'g'rilash](./03-seo.md) | Meta, Open Graph, sitemap, robots, Core Web Vitals |
| 04 | [Backend/API Deploy va CORS](./04-backend-deploy.md) | Railway/Render/Fly.io, env, CORS tekshirish |
| 05 | [VPS Asoslari](./05-vps-fundamentals.md) | VPS olish, SSH ulanish, boshqarish, security |
| 06 | [VPS'da Node + Nginx + PM2](./06-vps-nodejs-nginx-pm2.md) | Deploy, reverse proxy, PM2, SSL, log |
| 07 | [PHP Deploy](./07-php-deploy.md) | Shared hosting vs VPS, LEMP, Laravel |
| 08 | [VPS'da Ma'lumotlar Bazasi](./08-database-on-vps.md) | PostgreSQL/MySQL o'rnatish, xavfsizlash, backup |
| 09 | [Log, Monitoring va Security](./09-logs-monitoring-security.md) | journalctl, PM2 log, firewall, fail2ban |

## Umumiy tavsiya

- **Boshlovchi uchun**: static/React/Next → **Vercel yoki Netlify** (bepul, oson, SSL avtomatik).
- **Backend uchun tez yo'l**: **Railway / Render / Fly.io** (managed, VPS shart emas).
- **To'liq nazorat kerak bo'lsa**: **VPS** (DigitalOcean / Hetzner / Contabo) + Nginx + PM2.
- **Har doim**: HTTPS (Let's Encrypt), environment variable'larni maxfiy saqlash, backup.

> Bu amaliy qo'llanma — buyruqlarni o'z loyihangizda takrorlab ko'ring.

← [Bosh sahifaga qaytish](../README.md)
