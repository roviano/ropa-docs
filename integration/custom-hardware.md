# Custom Hardware Attachment — Sensors, Arms, Payload Trays, Third-Party Peripherals

> Applies to: iMRP400 / iMRP500 / iMRP600 / iMRP800
> Philosophy: **White-Box Robot Integration** — you see every sensor signal, you control every motor, you attach whatever you need

---

## Overview

The iMRP chassis is designed as a **platform, not a product**. It exposes every sensor reading, every motor command, and every navigation status through the ROPA protocol (WebSocket port 9090, JSON format). This means you can:

- Attach custom payload structures (trays, bins, screens)
- Mount additional sensors (cameras, environmental sensors, RFID readers)
- Integrate robotic arms or manipulation devices
- Build third-party peripheral integrations ( barcode scanners, label printers)

> **Non-Invasive Integration**: The chassis provides mounting surfaces, power, and data access — you don't need to modify internal firmware or drill into the frame.

---

## Mounting & Physical Integration

### Payload Capacity Reference

| Model | Rated Payload | Recommended Payload | Mounting Surface |
|-------|--------------|--------------------|-----------------|
| iMRP400 | 20 kg | ≤15 kg | 450×430 mm top plate |
| iMRP500 | 50 kg | ≤40 kg | D500 mm top plate |
| iMRP600 | 100 kg (rated) | ≤50 kg (recommended) | 600×450 mm top plate |
| iMRP800 | 100 kg (rated) | ≤50 kg (recommended) | 800×550 mm top plate |

> "Recommended payload" accounts for dynamic loads during navigation (acceleration, deceleration, turns). Static payload can reach rated capacity.

### Attachment Principles

| Principle | Why |
|-----------|-----|
| **Secure to top plate** | The chassis top surface is the designated mounting area — no drilling into side panels |
| **Balance the load** | Center-heavy mounting prevents tip-over during turns and stops |
| **Stay within footprint** | Payload should not extend beyond chassis dimensions in tight indoor spaces |
| **Protect sensors** | Do not obstruct LiDAR (360° scan), RGBD cameras, or ultrasonic sensors |
| **Consider cable routing** | Route cables along chassis edges, avoid wheels and rotating parts |

---

## Power Access

| Source | Voltage | Use Case |
|--------|---------|----------|
| Chassis battery (24V or 29.4V depending on model) | Main power | Robot motors, navigation compute |
| External power tap | Contact us for wiring details | Custom peripherals (screens, arms, sensors) |

> Power budget: Ensure total peripheral power draw does not exceed available capacity. Heavy power peripherals may reduce battery endurance.

---

## Data Integration via ROPA

### Reading Chassis Sensor Data for Your Hardware

Your custom hardware can read chassis data through ROPA WebSocket:

| Data You Need | ROPA Topic | Use Case |
|---------------|------------|----------|
| Robot position | `/robot_pose` | Arm targeting, screen positioning, scanner alignment |
| LiDAR scan | `/laser_data` | Custom obstacle logic, environmental monitoring |
| Robot status | `/robot_status` | Trigger hardware actions on battery/charging state |
| Navigation status | `/navi_status` | Trigger payload action at delivery point (status 603 = arrived) |
| Localization confidence | `/localization_confidence` | Pause arm operations if robot position uncertain |

### Sending Commands from Your Hardware

Your custom hardware controller can also send ROPA commands:

| Command You Need | ROPA Call | Use Case |
|-----------------|-----------|----------|
| Navigate to destination | `call_service /poi` | Arm-loaded → send to delivery point |
| Cancel navigation | `publish /move_base/cancel` | Emergency abort |
| Emergency stop | `publish /soft_stop` | Safety trigger from peripheral sensor |
| Speed control | `call_service /velocity_control` | Slow down when arm is extended |
| Teleop control | `publish /cmd_vel_mux/input/teleop` | Manual positioning for precise arm operation |

---

## Integration Patterns

### Pattern A: Payload Tray + Delivery (iMRP500)

```
┌─────────────┐
│  Payload Tray │ ← Custom top-mounted tray for goods delivery
│  (max 40 kg) │
├─────────────┤
│  iMRP500     │ ← Chassis navigates to POI
│  Chassis     │ ← ROPA /poi command
└─────────────┘
```

- Mount delivery tray on top plate
- ROPA monitors `/navi_status` → when status=603 (arrived), trigger tray indicator (LED or screen)
- Customer retrieves item → robot returns to pickup point

### Pattern B: Robotic Arm + Warehouse Pick (iMRP600)

```
┌─────────────┐
│  Robotic Arm │ ← 3rd-party arm (e.g., small DOF manipulation arm)
│  Controller   │ ← Separate compute, communicates with ROPA bridge
├─────────────┤
│  iMRP600     │ ← Chassis navigates to pick location
│  Chassis     │ ← Arm controller reads /robot_pose for positioning
└─────────────┘
```

- Chassis navigates to rack position via `/poi`
- Arm controller subscribes to `/robot_pose` → aligns arm to exact position
- Arm picks item → chassis navigates to pack station
- Dual-LiDAR on iMRP600 ensures reliable positioning near racks

### Pattern C: Environmental Monitoring Sensors (iMRP400)

- Mount temperature / humidity / air quality sensors on compact chassis
- Chassis patrols facility in Cycle mode (continuous POI loop)
- Custom controller reads sensor data + `/robot_pose` → creates spatial environmental map
- iMRP400's compact 450 mm footprint fits tight monitoring routes

### Pattern D: Interactive Screen + Concierge (iMRP500)

- Mount touchscreen on top plate
- ROPA `/navi_status` triggers screen content changes (welcome at lobby, directions at elevator)
- `/robot_pose` enables location-aware content delivery
- Touchscreen input can trigger `/poi` navigation commands

---

## Safety Considerations for Custom Hardware

| Concern | Mitigation |
|---------|------------|
| Payload tip-over | Keep center of gravity low and centered; stay within recommended payload |
| Sensor obstruction | Ensure LiDAR 360° view is unobstructed; RGBD cameras have clear forward view |
| Cable entanglement | Route cables along chassis edges, away from wheels and rotating parts |
| Emergency stop access | Hardware E-stop button must remain accessible at all times |
| Power overload | Calculate total peripheral power draw; ensure battery capacity covers full shift |
| Navigation interference | Extended payload may trigger obstacle detection — adjust `/laser_safety_range` threshold |

---

## Developer Kit vs Integrator Package for Custom Hardware

| You are a... | Choose | Reason |
|--------------|--------|--------|
| Developer building your own payload + software | **Developer Kit** ($2,699) | Full ROPA protocol access + docs + code examples |
| Integrator deploying custom hardware for a client | **Integrator Package** ($2,899) | On-site verification + priority support for physical integration |

> For complex hardware integrations (robotic arms, custom sensor arrays), the Integrator Package's on-site deployment verification is strongly recommended.

---

## Next Steps

- **Protocol details**: [ROPA Protocol Reference](https://github.com/roviano/ropa-sdk/blob/main/docs/protocol.md)
- **Model specs**: [Hardware Specifications](https://github.com/roviano/ropa-sdk/blob/main/docs/models.md)
- **ROS integration**: [ROS-bridgeable setup](ros-bridge.md) for streaming sensor data to your custom controller
- **Deployment guides**: [Indoor service](../deployment/indoor-service.md) / [Warehouse](../deployment/warehouse.md) / [Multi-floor](../deployment/multi-floor.md)
- **Choose your package**: [Developer Kit](../kits/developer-kit.md) vs [Integrator Package](../kits/integrator-kit.md)

---

## Get the Hardware

👉 [Roviano on Alibaba.com](https://roviano.en.alibaba.com) — iMRP400 / 500 / 600 / 800

**Contact:** engineering@roviano.com
