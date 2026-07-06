# Frequently Asked Questions

---

## Protocol & Architecture

### What is ROPA?

**ROPA** (Roviano Open Platform API) is the WebSocket+JSON protocol that gives you full data access to every iMRP chassis — sensors, motors, navigation commands, map data, and system status. It runs on port 9090, uses JSON format, and is full-duplex (you can both read data and send commands simultaneously).

- Full protocol reference: [docs/protocol.md](https://github.com/roviano/ropa-sdk/blob/main/docs/protocol.md)
- Code examples: [examples/](https://github.com/roviano/ropa-sdk/tree/main/examples)

### Does the robot run ROS internally?

**No.** The iMRP chassis runs ROPA — a standalone WebSocket protocol. It does not run ROS1 or ROS2 internally. You can bridge ROPA data into your ROS application using a lightweight bridge node. See [ROS-bridgeable setup](integration/ros-bridge.md).

### Is ROPA open-source?

**Yes.** The protocol specification, documentation, and code examples are all on GitHub under MIT license — no NDA, no paywall, no restricted access.

- [github.com/roviano/ropa-sdk](https://github.com/roviano/ropa-sdk) — protocol + examples
- [github.com/roviano/ropa-docs](https://github.com/roviano/ropa-docs) — deployment + integration guides

### What programming languages can I use?

**Any language with WebSocket support.** Python, JavaScript, C++, Java, Go, Rust — ROPA uses standard WebSocket (port 9090) and JSON, so any language that can open a WebSocket connection and parse JSON works.

We provide ready-to-run examples in Python and JavaScript.

### What are the 7 ROPA operation types?

| op | Direction | Purpose |
|----|-----------|---------|
| `advertise` | Client → Robot | Register as publisher |
| `publish` | Client → Robot | Send command/data |
| `unadvertise` | Client → Robot | Stop publishing |
| `subscribe` | Client → Robot | Request data stream |
| `unsubscribe` | Client → Robot | Stop data stream |
| `call_service` | Client → Robot | Request service execution |
| `service_response` | Robot → Client | Service result |

---

## Hardware & Models

### What's the difference between the four models?

| | iMRP400 | iMRP500 | iMRP600 | iMRP800 |
|---|---------|---------|---------|---------|
| **Payload** | 20 kg | 50 kg | 100 kg | 100 kg |
| **Speed** | 0.5 m/s | 0.8 m/s | 0.8 m/s | 1.2 m/s |
| **Mapping** | 10,000 m² | 10,000 m² | 30,000 m² | 10,000 m² |
| **Elevator** | — | — | ✅ Exclusive | — |
| **Best for** | Tight spaces, education | Warehouse, general service | Multi-floor, heavy payload | Fast transit, factory |

Full specs: [docs/models.md](https://github.com/roviano/ropa-sdk/blob/main/docs/models.md)

### Do all four models use the same protocol?

**Yes.** All four models run the same ROPA protocol on port 9090. One SDK, any chassis. Your application code works across the entire series without modification.

### What is "Autonomous Elevator Riding"?

An **iMRP600-exclusive** feature. The robot can call elevators, enter, ride to another floor, exit, and continue navigation — all autonomously, no human intervention. Requires algorithm ≥ V4.5.0 and compatible elevator controller. See [multi-floor deployment](deployment/multi-floor.md).

### What certifications do you have?

| What we certify | What we do NOT certify (yet) |
|-----------------|------------------------------|
| Battery: UN38.3 + MSDS compliant | Full machine CE / FCC |

CE/FCC certification for the complete machine is in progress. Contact engineering@roviano.com for timeline.

---

## Deployment & Setup

### How do I connect to the robot?

| Method | Details |
|--------|---------|
| **WiFi** | SSID: `TY1251D-xxxxx` or `TY1251C003-xxxx`, password: `123456789`, IP: `10.42.0.1` or `192.168.20.22` |
| **Ethernet** | Gateway: `192.168.20.1`, Robot IP: `192.168.20.22`, Subnet: `255.255.255.0` |
| **ROPA WebSocket** | `ws://<ROBOT_IP>:9090` |

### How do I map my workspace?

Use the DeploymentTool Android app or ROPA service calls. Three mapping modes: Standard (1,000–2,000 m²), Similar (2,000–3,000 m²), Large Scene (3,000–10,000 m²). See [Android mapping workflow](integration/android-mapping.md).

### Can I map without the Android app?

**Yes.** All mapping functions have ROPA API equivalents:
- `call_service /node_manager_control` (cmd=0 → start mapping, cmd=3 → save, cmd=4 → start navigation)
- Walk robot via `publish /cmd_vel_mux/input/teleop`
- See [Protocol Reference](https://github.com/roviano/ropa-sdk/blob/main/docs/protocol.md)

### How long does mapping take?

- Small office/lobby (1,000 m²): 10–20 minutes
- Warehouse floor (5,000 m²): 30–45 minutes
- Hospital wing (10,000 m²): 60–90 minutes

Time depends on walking speed and area complexity. Slower, more thorough walks produce higher-quality maps.

---

## Developer Kit vs Integrator

### What's the difference?

| | Developer Kit ($2,699) | Integrator Package ($2,899) |
|---|------------------------|-----------------------------|
| Chassis + ROPA SDK + docs | ✅ | ✅ |
| Python/JS code examples | ✅ | ✅ |
| Remote engineer support (72h) | ✅ | ✅ |
| **On-site deployment** | — | ✅ |
| **Priority support (48h SLA)** | — | ✅ |
| **Integration recipe cookbook** | — | ✅ |
| **Production readiness audit** | — | ✅ |

### Who should choose Developer Kit?

Software developers, researchers, and teams with engineering capability who want full protocol access to build their own application.

### Who should choose Integrator Package?

System integrators deploying for clients, enterprise IT teams with production SLAs, and anyone who needs on-site verification before go-live.

See: [Developer Kit](kits/developer-kit.md) / [Integrator Package](kits/integrator-kit.md)

---

## Integration

### Can I use ROS with iMRP?

**Yes — through a bridge.** ROPA is WebSocket+JSON. You write a bridge node that subscribes to ROPA topics and publishes into your ROS topic tree. No ROS installation on the robot required. See [ROS-bridgeable setup](integration/ros-bridge.md).

### Can I attach custom hardware?

**Yes.** The chassis provides mounting surfaces, payload capacity, and full data access via ROPA. You can attach sensors, screens, robotic arms, payload trays, or any third-party peripheral without modifying the robot firmware. See [custom hardware attachment](integration/custom-hardware.md).

### Can I control the robot from my own application?

**Yes.** All navigation, mapping, and control commands are available via ROPA service calls and publish topics. Your application can:
- Navigate to POI: `call_service /poi`
- Teleop control: `publish /cmd_vel_mux/input/teleop`
- Emergency stop: `publish /soft_stop`
- Speed control: `call_service /velocity_control`

---

## Troubleshooting

### Robot won't connect

- Reboot robot (long-press power button)
- Verify WiFi SSID visible on phone
- Try Ethernet connection if WiFi unreliable

### Map quality is poor

- Re-map with slower, more thorough walking path
- Use "Similar" mode to build on previous map
- Ensure LiDAR is not obstructed by payload or mounting

### Localization drift

- Run global localization via DeploymentTool
- Or point calibration at a known reference POI
- Monitor `/localization_confidence` — below 0.8 indicates poor localization

### Navigation fails repeatedly

- Check if map is outdated (furniture moved, new obstacles)
- Re-map or edit map (add/remove virtual walls and obstacle areas)
- Verify POI coordinates are accurate

### Battery drains fast

- Switch from high speed to balanced/low mode
- Reduce payload weight
- Shorten navigation routes (optimize POI sequence)

---

## Contact & Support

| Channel | Response | Scope |
|---------|----------|-------|
| engineering@roviano.com | 72h (Developer Kit) / 48h (Integrator) | All technical questions |
| [GitHub Issues](https://github.com/roviano/ropa-sdk/issues) | Community + team | Documentation, examples, protocol |
| [Roviano on Alibaba.com](https://roviano.en.alibaba.com) | — | Purchase, pricing, availability |

---

## Get the Hardware

👉 [Roviano on Alibaba.com](https://roviano.en.alibaba.com) — iMRP400 / 500 / 600 / 800

**Developer Kit** (iMRP500: $2,699 FOB) — for software builders
**Integrator Package** (iMRP500: $2,899 FOB) — for production deployments
