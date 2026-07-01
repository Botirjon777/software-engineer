# System Design — Intervyuga Tayyorgarlik

Katta, masshtablanadigan tizimlarni loyihalash: distributed systems asoslari, building block'lar va klassik case study'lar.

> Eslatma: frontend-specific dizayn uchun [frontend/09](../frontend/09-frontend-system-design.md), backend-specific uchun [backend/10](../backend/10-backend-system-design.md) ga ham qarang. Bu bo'lim umumiy distributed systems va to'liq case study'larga bag'ishlangan.

## Mundarija

| # | Mavzu | Daraja |
|---|-------|--------|
| 01 | [System Design Asoslari](./01-fundamentals.md) | ⭐⭐⭐ Murakkab |
| 02 | [Case Study'lar (Twitter, Uber, Chat...)](./02-case-studies.md) | ⭐⭐⭐⭐ Senior |
| 03 | [Scalability va Reliability](./03-scalability-reliability.md) | ⭐⭐⭐ Murakkab |
| 04 | [Distributed Systems](./04-distributed-systems.md) | ⭐⭐⭐⭐ Senior |
| 05 | [Arxitektura Blueprint va Freymvorklar](./05-architecture-blueprints.md) | ⭐⭐⭐⭐ Senior |
| 06 | [Hosting va Infratuzilma](./06-hosting-infrastructure.md) | ⭐⭐⭐ Murakkab |
| 07 | [Production Tizim Misollari](./07-production-examples.md) | ⭐⭐⭐⭐ Senior |

## Intervyuda nimaga e'tibor beriladi

- **Yondashuv**: requirements → capacity estimation → API → data model → high-level → deep dive → bottleneck.
- **Building block'lar**: load balancer, cache, queue, DB sharding/replication, CDN, consistency.
- **Trade-off'lar**: CAP, consistency vs availability, latency vs durability — har qaror asoslanishi kerak.

> Yechimlar: [`solutions/system-design/`](../solutions/system-design/)

← [Bosh sahifaga qaytish](../README.md)
