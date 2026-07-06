# ROPA Docs — Deployment & Integration Guides

Production-ready deployment recipes and integration tutorials for the **iMRP series** mobile robot chassis.

Built on [ROPA Open Platform API](https://github.com/roviano/ropa-sdk) — the WebSocket+JSON protocol that gives you **full data access** to every sensor, every motor, every navigation command. No black box.

---

## Quick Deployment Paths

| Scenario | Recommended Model | Guide |
|----------|------------------|-------|
| Indoor service robot (delivery / greeting / concierge) | iMRP400 / iMRP500 | [`deployment/indoor-service.md`](deployment/indoor-service.md) |
| Warehouse material handling | iMRP500 / iMRP600 | [`deployment/warehouse.md`](deployment/warehouse.md) |
| Multi-floor building logistics (hospital / hotel / office) | iMRP600 (Autonomous Elevator) | [`deployment/multi-floor.md`](deployment/multi-floor.md) |
| Factory rapid transit & cleanroom logistics | iMRP800 (1.2 m/s Wide Base) | [`deployment/warehouse.md`](deployment/warehouse.md) |

---

## Integration Recipes

- **ROS-bridgeable setup**: [`integration/ros-bridge.md`](integration/ros-bridge.md) — connect ROPA to your ROS stack without rewriting your application
- **Android Mapping Workflow**: [`integration/android-mapping.md`](integration/android-mapping.md) — map your workspace from an Android phone
- **Custom hardware attach**: [`integration/custom-hardware.md`](integration/custom-hardware.md) — sensors, robotic arms, payload trays, third-party peripherals

---

## Integrator Package

The **Integrator Package** (from $2,899 FOB) includes everything in the Developer Kit **plus**:

| What Developer Kit includes | What Integrator adds |
|-----------------------------|----------------------|
| Chassis (any iMRP model) | **On-site deployment assistance** |
| ROPA SDK + full documentation | **Priority engineer support** (48h response SLA) |
| Python / JS / Android examples | **Integration recipe cookbook** |
| 1-on-1 engineer support (remote) | **Production readiness audit** |

**Why pay $200 more?** If you're an integrator deploying into a live production environment, the Integrator Package saves weeks of trial-and-error. Our engineers arrive on-site, verify your deployment, and hand you a production-ready system.

👉 [Roviano on Alibaba.com](https://roviano.en.alibaba.com) — contact us for Integrator Package pricing and availability

---

## Developer Kit vs Integrator — Which one?

| You are a... | Choose | Reason |
|--------------|--------|--------|
| Software developer building your own AMR app | **Developer Kit** ($2,699) | You have the engineering team; you need the SDK and docs |
| System integrator deploying for a client | **Integrator Package** ($2,899) | You need on-site verification + priority support + recipes |
| Research lab / university | **Developer Kit** | You want full protocol access for research |
| Enterprise IT deploying in-hospital / in-hotel | **Integrator Package** | Production uptime SLA requires priority support |

---

## Model Selection Guide

| Your requirement | Recommended model | Key spec |
|-----------------|-------------------|----------|
| Small spaces, <20 kg payload | iMRP400 | 450 mm compact, RGBD 3D vision |
| General warehouse, 50 kg payload | iMRP500 | 12h endurance, differential drive |
| Multi-floor, heavy payload, elevator | iMRP600 | Autonomous elevator riding, dual-LiDAR |
| Wide corridors, fast transit | iMRP800 | 1.2 m/s, 800 mm wide base |

> All four models run the **same ROPA protocol** — one SDK, any chassis. See [Model Specifications](https://github.com/roviano/ropa-sdk/blob/main/docs/models.md).

---

## Related

- **[ROPA SDK Repository](https://github.com/roviano/ropa-sdk)** — Python/JS examples + protocol reference + model specs
- **[Protocol Reference](https://github.com/roviano/ropa-sdk/blob/main/docs/protocol.md)** — complete op types, topics, services, status codes

---

## License

MIT — use commercially, modify freely, no restrictions.
