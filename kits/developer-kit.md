# Developer Kit — Full Protocol Access for Software Builders

> Applies to: All iMRP models (iMRP400 / iMRP500 / iMRP600 / iMRP800)
> FOB Pricing: iMRP500 Developer Kit = $2,699 | Other models: contact for pricing

---

## What's in the Developer Kit

| Item | Description |
|------|-------------|
| **iMRP Chassis** | Your chosen model (400/500/600/800), fully assembled, battery charged |
| **ROPA SDK** | Full protocol documentation, Python & JavaScript examples |
| **DeploymentTool APK** | Android app for mapping, POI management, calibration, settings |
| **Protocol Reference** | Complete ROPA WebSocket+JSON specification (7 op types, 10+ subscribe topics, 8 publish topics, 12 service calls) |
| **Model Specifications** | Verified hardware specs for all four chassis models |
| **1-on-1 Engineer Support** | Remote support via email/video call (response within 72 hours) |

### Documentation You Get

All documentation is open-source on GitHub — no NDA, no paywall:

| Repo | Content | Link |
|------|---------|------|
| **ropa-sdk** | Protocol reference, model specs, deployment guide, code examples | [github.com/roviano/ropa-sdk](https://github.com/roviano/ropa-sdk) |
| **ropa-docs** | Deployment recipes, integration tutorials, kit comparison | [github.com/roviano/ropa-docs](https://github.com/roviano/ropa-docs) |

---

## Why Developer Kit

You are a **software developer or researcher** building your own AMR application. You need:

- **Full data access** — every sensor, every motor, every navigation command via ROPA
- **No black box** — the protocol is fully documented, all topics and services are specified
- **Code examples** — Python and JavaScript ready-to-run, covering connection, pose subscription, mapping, navigation
- **Freedom to choose your stack** — ROPA is language-agnostic WebSocket+JSON; integrate with ROS, custom Python, JavaScript, or any WebSocket-capable framework

### Developer Kit vs Writing Your Own Protocol

| Option | Effort | Access | Risk |
|--------|--------|--------|------|
| **Developer Kit (ROPA)** | Plug into existing, documented protocol | Full sensor/motor/nav access | Zero — protocol is stable and tested |
| Reverse-engineer closed protocol | Weeks/months of effort | Partial — undocumented gaps | High — firmware updates may break your integration |

---

## Quick Start: From Box to Moving Robot

### 1. Unbox & Power On

- Long-press power button (~3s) until indicator glows
- Robot broadcasts WiFi: SSID `TY1251D-xxxxx`, password `123456789`

### 2. Connect via ROPA

```python
import websocket, json

ws = websocket.create_connection("ws://10.42.0.1:9090")

# Subscribe to robot position
ws.send(json.dumps({
    "op": "subscribe",
    "topic": "/robot_pose",
    "type": "geometry_msgs/Pose2D",
    "throttle_rate": 100
}))

result = ws.recv()
print(json.loads(result)["msg"])  # {"x": 1.23, "y": 4.56, "theta": 0.78}
```

### 3. Map Your Space

Use DeploymentTool APK or ROPA:

```python
# Start mapping
ws.send(json.dumps({
    "op": "call_service",
    "id": "start_mapping",
    "service": "/node_manager_control",
    "args": {"cmd": 0}
}))

# Walk robot through area...

# Save map
ws.send(json.dumps({
    "op": "call_service",
    "id": "save_map",
    "service": "/node_manager_control",
    "args": {"cmd": 3}
}))
```

### 4. Navigate

```python
ws.send(json.dumps({
    "op": "call_service",
    "id": "nav_to_point",
    "service": "/poi",
    "args": {"poi_name": "destination_A", "poi_floor": "1", "poi_building": "MyBuilding"}
}))
```

---

## What You Can Build

| Project | How with Developer Kit |
|---------|----------------------|
| **Custom AMR application** | Full ROPA access → your own planner, your own UI, your own business logic |
| **ROS integration** | Bridge ROPA data into your ROS stack → see [ros-bridge guide](../integration/ros-bridge.md) |
| **Warehouse WMS integration** | ROPA `/robot_pose` + `/navi_status` → feed into your warehouse management system |
| **Fleet coordinator** | Multiple chassis → one coordinator → `/poi` commands per robot |
| **Research prototype** | Full sensor stream (LiDAR, RGBD, ultrasonic, IMU) for algorithm development |
| **Interactive kiosk robot** | Custom screen + ROPA pose → location-aware content delivery |

---

## Specifications at a Glance

| | iMRP400 | iMRP500 | iMRP600 | iMRP800 |
|---|---------|---------|---------|---------|
| **Payload** | 20 kg | 50 kg | 100 kg | 100 kg |
| **Max Speed** | 0.5 m/s | 0.8 m/s | 0.8 m/s | 1.2 m/s |
| **Endurance** | 12 h | 12 h | 12 h | 10 h |
| **Mapping Area** | 10,000 m² | 10,000 m² | 30,000 m² | 10,000 m² |
| **Positioning** | ±10 cm | ±5 cm | ±10 cm | ±10 cm |
| **Elevator** | — | — | ✅ | — |
| **Developer Kit FOB** | Contact us | **$2,699** | Contact us | Contact us |

> Full specs: [Model Specifications](https://github.com/roviano/ropa-sdk/blob/main/docs/models.md)

---

## Support

| Channel | Response Time | Coverage |
|---------|--------------|---------|
| engineering@roviano.com | Within 72 hours | Protocol questions, integration guidance, bug reports |
| GitHub Issues | Community + team | Documentation corrections, feature requests |
| Video call | Scheduled per request | Complex integration troubleshooting |

---

## Developer Kit → Integrator Upgrade

If you start with the Developer Kit and later need on-site deployment verification or priority support, you can upgrade to the Integrator Package. Contact engineering@roviano.com for upgrade pricing.

---

## Get the Developer Kit

👉 [Roviano on Alibaba.com](https://roviano.en.alibaba.com) — Developer Kit (iMRP500: $2,699 FOB)

**Contact:** engineering@roviano.com

---

## See Also

- **[Integrator Package](integrator-kit.md)** — Developer Kit + on-site deployment + priority support
- **[Deployment guides](../deployment/)** — Indoor service, warehouse, multi-floor scenarios
- **[Integration guides](../integration/)** — ROS bridge, Android mapping, custom hardware
- **[ROPA SDK](https://github.com/roviano/ropa-sdk)** — Protocol reference + code examples
