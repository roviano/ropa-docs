# Android Mapping Workflow — Map Your Workspace from a Phone

> Applies to: iMRP400 / iMRP500 / iMRP600 / iMRP800
> Required: DeploymentTool APK (provided with Developer Kit or Integrator Package)
> Algorithm ≥ V4.5.0 required for advanced mapping features

---

## Overview

The **DeploymentTool Android app** lets you create, edit, and manage floor maps directly from your phone — no laptop, no command line, no ROS installation required. This is the fastest path from "empty room" to "robot navigating autonomously."

> **Zero-Drill Philosophy**: No drilling into walls, no permanent infrastructure changes. The robot maps the environment as-is and navigates within it. Move furniture? Just re-map.

---

## Quick Start: Map Your Space in Minutes

### 1. Connect

| Step | Action |
|------|--------|
| Power on robot | Long-press power button (~3s) until indicator glows |
| Connect phone WiFi | SSID: `TY1251D-xxxxx` or `TY1251C003-xxxx`, password: `123456789` |
| Open DeploymentTool | Auto-detects robot at `10.42.0.1` (WiFi) or `192.168.20.22` (Ethernet) |
| Verify connection | Robot status appears in app header |

### 2. Choose Mapping Mode

| Mode | Best For | Coverage |
|------|----------|----------|
| **Standard** | Offices, small warehouses, hotel lobbies | 1,000–2,000 m² |
| **Similar** | Re-mapping after minor changes, extending existing map | 2,000–3,000 m² |
| **Large Scene** | Full warehouse floors, hospital wings, large facilities | 3,000–10,000 m² |

### 3. Map

1. Tap **New Map** → select mapping mode
2. Walk the robot through every area it needs to navigate
   - Push the robot manually (easiest)
   - Or teleop via ROPA: `publish /cmd_vel_mux/input/teleop` (forward, turn)
3. The robot's LiDAR + RGBD + ultrasonic sensors continuously scan the environment
4. Watch the map build in real-time on your phone screen
5. When coverage is complete, tap **Save Map**

### 4. Add Points of Interest (POI)

| Action | Steps |
|--------|-------|
| Add POI | Position robot at destination → tap **Add Point** → name it (e.g., "Kitchen", "Dock A") |
| Navigate to POI | Select POI → tap **Go** → robot navigates autonomously |
| Multi-POI route | Select multiple POIs → choose **Single pass / Round-trip / Cycle** mode |

### 5. Edit Map for Safety

| Edit Type | Purpose | How |
|-----------|---------|-----|
| **Virtual wall** | Block doorways/zones robot should not cross | Draw line on map — robot never crosses |
| **Deceleration zone** | Robot slows near intersections, doorways | Tap zone → set type |
| **Dangerous zone** | Maximum caution near stairs, docks | Tap zone → set type |
| **Strong light zone** | Adjust sensors near windows/skylights | Tap zone → set type |
| **Obstacle area** | Permanent obstacle (furniture) | Paint red area on map |

---

## Mapping by Scenario

### Small Office / Retail (<2,000 m²)

- Mode: **Standard**
- Time: 10–20 minutes to walk robot through all rooms and corridors
- POI: Reception desk, meeting rooms, kitchen, charging dock
- Edits: Virtual walls at staff-only doors, deceleration zones at intersections

### Warehouse Floor (2,000–10,000 m²)

- Mode: **Large Scene** (or **Similar** if extending existing map)
- Time: 30–60 minutes for full coverage
- POI: Pick zones, pack stations, staging areas, charging dock
- Edits: Virtual walls at cold storage, dangerous zones at dock edges, strong light zones near skylights

### Hospital Wing / Hotel (Multi-Floor, iMRP600)

- Map each floor separately in **Standard** or **Similar** mode
- Add elevator POI on each floor (see [multi-floor deployment](../deployment/multi-floor.md))
- Link floors via elevator configuration in DeploymentTool Settings

---

## ROPA API Equivalents

Every DeploymentTool action has a ROPA API equivalent — useful for automation or integration:

| DeploymentTool Feature | ROPA Equivalent | See |
|------------------------|-----------------|------|
| New map | `call_service /node_manager_control` (cmd=0) | [Protocol Reference](https://github.com/roviano/ropa-sdk/blob/main/docs/protocol.md) |
| Save map | `call_service /node_manager_control` (cmd=3) | Same |
| Start navigation | `call_service /node_manager_control` (cmd=4) | Same |
| Add POI | `publish /insert_current_pose_marker` | Same |
| Navigate to POI | `call_service /poi` | Same |
| Virtual walls / areas | `call_service /layered_map_cmd` | Same |
| Speed settings | `call_service /velocity_control` | Same |
| Software E-stop | `publish /soft_stop` | Same |
| Robot pose | `subscribe /robot_pose` | Same |
| Battery / status | `subscribe /robot_status` | Same |

> Full field guide: [DeploymentTool Android App Guide](https://github.com/roviano/ropa-sdk/blob/main/docs/android-deployment.md)

---

## Localization & Calibration

### After Manual Relocation

If you pick up the robot and place it somewhere new:

1. DeploymentTool → **Global Localization** → robot scans and matches against entire map
2. Monitor progress: `/localization_confidence` topic → wait until confidence ≥ 0.8
3. If confidence stays low → **Regional Localization** (if approximate position known)

### Fine-Tuning Position

| Method | When to Use | Steps |
|--------|------------|-------|
| **Point calibration** | Robot at known reference point | DeployTool → Calibrate → select reference POI |
| **Manual calibration** | Robot drifted over time | DeployTool → Manual Calibration → adjust X/Y/heading offset |

---

## Tips for Better Maps

| Tip | Why |
|-----|-----|
| Walk slowly during mapping | Slower path = denser scan points = higher map quality |
| Cover all areas robot will visit | Missing areas = unexplored zones = robot won't plan paths through them |
| Map during quiet hours | Moving people/objects during mapping create "ghost obstacles" |
| Re-map after major changes | Furniture rearrangement, new walls, removed obstacles — re-map or edit map |
| Use "Similar" mode for updates | Reuses previous map as template, much faster than full re-map |
| Save map immediately after completion | Unsaved maps are lost on power cycle |

---

## Next Steps

- **Full DeploymentTool guide**: [Android Deployment Field Guide](https://github.com/roviano/ropa-sdk/blob/main/docs/android-deployment.md)
- **Protocol details**: [ROPA Protocol Reference](https://github.com/roviano/ropa-sdk/blob/main/docs/protocol.md)
- **ROS integration**: [ROS-bridgeable setup](ros-bridge.md) for streaming map data into your ROS application
- **Choose your package**: [Developer Kit](../kits/developer-kit.md) vs [Integrator Package](../kits/integrator-kit.md)

---

## Get the Hardware

👉 [Roviano on Alibaba.com](https://roviano.en.alibaba.com) — iMRP400 / 500 / 600 / 800

**Contact:** engineering@roviano.com
