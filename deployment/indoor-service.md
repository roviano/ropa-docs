# Indoor Service Robot Deployment — iMRP400 / iMRP500

> Applies to: iMRP400 (compact indoor), iMRP500 (general service)
> Required: Developer Kit or Integrator Package, DeploymentTool APK

---

## Overview

Deploy an autonomous mobile robot for **greeting, delivery, and concierge** tasks in indoor environments — hotels, hospitals, restaurants, office lobbies, retail stores.

The iMRP400 and iMRP500 share the same ROPA protocol (WebSocket port 9090, JSON format). One SDK, any scenario.

---

## Choosing Your Chassis

| Requirement | Recommended Model | Reason |
|-------------|-------------------|--------|
| Tight corridors, small lobby, <20 kg payload | **iMRP400** | 450 mm compact footprint, RGBD 3D vision for depth perception |
| Standard corridors, food/goods delivery, 50 kg payload | **iMRP500** | Differential drive, 12h endurance, ±5 cm positioning |
| Multi-floor delivery (across floors) | **iMRP600** → see [multi-floor deployment](multi-floor.md) | Autonomous elevator riding |

---

## Step-by-Step Deployment

### 1. Map the Environment

Use the DeploymentTool Android app to create a floor map.

| Step | Action | ROPA Equivalent |
|------|--------|-----------------|
| Connect to robot WiFi | SSID: `TY1251D-xxxxx`, password: `123456789` | — |
| Start mapping mode | DeploymentTool → New Map | `call_service /node_manager_control` (cmd=0) |
| Walk robot through area | Push or teleop via `/cmd_vel_mux/input/teleop` | — |
| Save map | DeploymentTool → Save Map | `call_service /node_manager_control` (cmd=3) |

For indoor spaces under 2,000 m², use **Standard** mapping mode. For larger lobbies or connected halls, use **Similar** mode (reuses previous map template).

### 2. Define Points of Interest (POI)

Create named destinations the robot will navigate between.

| Typical POI | Name Example | Purpose |
|-------------|-------------|---------|
| Front desk | `lobby_desk` | Greeting origin / return point |
| Table / room | `table_7`, `room_301` | Delivery destination |
| Kitchen / prep area | `kitchen_pickup` | Pickup origin for food delivery |
| Charging dock | `charge_dock` | Auto-recharge station |

Add via DeploymentTool or ROPA:
```json
{
  "op": "publish",
  "topic": "/insert_current_pose_marker",
  "msg": {
    "name": "lobby_desk",
    "behavior_code": 0,
    "time_out": 10,
    "rest_time": 5
  }
}
```

### 3. Edit Map for Safety

Use DeploymentTool map editing to add safety constraints:

- **Virtual walls** — block doorways the robot should not enter (staff-only areas, stairwells)
- **Deceleration zones** — near doorways, intersections, pedestrian areas
- **Dangerous zones** — near stairs, escalators, loading docks

> All map edits are stored on the robot and persist across sessions.

### 4. Set Navigation Parameters

| Parameter | Recommended Indoor Value | ROPA Service |
|-----------|--------------------------|--------------|
| Speed mode | **Balance medium** (cmd=4) | `call_service /velocity_control` |
| Travel mode | **Balanced** | DeploymentTool → Settings |
| Safety threshold | Default (0.3 m obstacle, 0.8 confidence) | `publish /laser_safety_range`, `/laser_safety_controller/setting_confidence_threshold` |

For crowded environments (hotel lobby during peak hours), use **Safety low speed** (cmd=0).

### 5. Set Up Charging

Each map supports **one charging pile**.

| Requirement | Detail |
|-------------|--------|
| Location | Wall-mounted, ≥1.5 m clearance |
| Beacon | IR auto-dock beacon facing robot approach path |
| POI | Add charging pile as a named point on the map |
| Auto-dock | Robot auto-docks when battery < threshold |

Monitor battery via ROPA:
```json
{
  "op": "subscribe",
  "topic": "/robot_status",
  "type": "robot_status_msg"
}
```
→ `battery` field (0–100), `charger` field (0=not charging, 1=charging).

---

## Application Patterns

### Pattern A: Greeting & Return

Robot waits at `lobby_desk` → navigates to guest → displays greeting → returns to `lobby_desk`.

```python
# ROPA sequence: navigate to guest POI, wait, return
import websocket
ws = websocket.create_connection("ws://10.42.0.1:9090")

# Navigate to guest
ws.send(json.dumps({
    "op": "call_service",
    "id": "nav_guest",
    "service": "/poi",
    "args": {"poi_name": "table_7", "poi_floor": "1", "poi_building": "Hotel"}
}))

# Monitor navigation status
ws.send(json.dumps({"op": "subscribe", "topic": "/navi_status", "type": "navi_status_msg"}))
# Wait for status 603 (success), then trigger greeting logic

# Return to lobby
ws.send(json.dumps({
    "op": "call_service",
    "id": "nav_return",
    "service": "/poi",
    "args": {"poi_name": "lobby_desk", "poi_floor": "1", "poi_building": "Hotel"}
}))
```

### Pattern B: Food / Document Delivery

Kitchen picks up order → robot navigates to `kitchen_pickup` → loaded → navigates to `table_7` → waits for unload → returns.

### Pattern C: Concierge Patrol (Multi-POI Cycle)

Robot visits multiple POI in a loop — lobby desk → front entrance → elevator hall → return.

Use **Cycle mode** in DeploymentTool (or implement via sequential `/poi` calls with ROPA).

---

## Monitoring & Safety

### Real-Time Monitoring Topics

| What to monitor | ROPA Topic | Key Field |
|-----------------|------------|-----------|
| Robot position | `/robot_pose` | `x`, `y`, `theta` |
| Navigation status | `/navi_status` | `status` (600–604) |
| Battery level | `/robot_status` | `battery`, `charger` |
| Obstacle proximity | `/obstacle_region` | `region` (bitmask) |
| Localization quality | `/localization_confidence` | `confidence` (0.0–1.0) |

### Emergency Stop

| Type | Trigger | ROPA |
|------|---------|------|
| Hardware E-stop | Physical button on chassis | Immediate motor cutoff |
| Software E-stop | App button or API call | `publish /soft_stop` (msg: `{data: true}`) |

> Use **hardware E-stop** for physical emergencies (person in path). Use **software E-stop** for temporary navigation pauses.

---

## Localization Recovery

If the robot is manually relocated (picked up and placed elsewhere):

1. Trigger **global localization** via DeploymentTool → robot scans environment and matches against entire map
2. Monitor confidence via `/localization_confidence` — wait until `confidence ≥ 0.8`
3. If confidence stays low, re-map or re-calibrate

---

## Next Steps

- **Integrate with your application**: See [ROS-bridgeable setup](../integration/ros-bridge.md) or [Android Mapping](../integration/android-mapping.md)
- **Custom hardware**: Add payload trays, screens, sensors — see [custom-hardware](../integration/custom-hardware.md)
- **Choose your package**: [Developer Kit](../kits/developer-kit.md) vs [Integrator Package](../kits/integrator-kit.md)

---

## Get the Hardware

👉 [Roviano on Alibaba.com](https://roviano.en.alibaba.com) — iMRP400 / iMRP500

**Contact:** engineering@roviano.com
