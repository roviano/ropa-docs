# Warehouse Material Handling Deployment — iMRP500 / iMRP600 / iMRP800

> Applies to: iMRP500 (50 kg general), iMRP600 (100 kg heavy + elevator), iMRP800 (fast transit)
> Required: Developer Kit or Integrator Package, DeploymentTool APK

---

## Overview

Deploy autonomous mobile robots for **pick-and-place, inventory transport, and inter-zone logistics** in warehouse environments.

All three models share the same ROPA protocol (WebSocket port 9090, JSON format). One SDK, any chassis — from 50 kg shelf-to-shelf delivery to 100 kg pallet transport across floors.

---

## Choosing Your Chassis

| Requirement | Recommended Model | Key Advantage |
|-------------|-------------------|---------------|
| Shelf-to-shelf, <50 kg, single floor | **iMRP500** | ±5 cm precision, 12h endurance, differential drive |
| Heavy pallet, 100 kg, multi-floor | **iMRP600** | Dual-LiDAR, autonomous elevator riding, 30,000 m² mapping |
| Fast point-to-point in wide corridors | **iMRP800** | 1.2 m/s speed, 800 mm stable wide base |
| Cleanroom / factory transit | **iMRP800** | Fast + stable under load |

---

## Step-by-Step Deployment

### 1. Map the Warehouse

| Warehouse Size | Recommended Mapping Mode |
|----------------|-------------------------|
| 1,000–2,000 m² (small warehouse) | **Standard** |
| 2,000–3,000 m² (mid-size) | **Similar** (uses previous map template) |
| 3,000–10,000 m² (large facility) | **Large Scene** |

For iMRP600 multi-floor warehouses: map each floor separately, then link via elevator configuration (see [multi-floor deployment](multi-floor.md)).

### 2. Define Warehouse POI

| POI Category | Name Examples | Purpose |
|-------------|-------------|---------|
| Pick station | `pick_zone_A`, `rack_12` | Source for outbound picking |
| Pack station | `pack_station_1` | Destination for order consolidation |
| Staging area | `staging_outbound`, `staging_inbound` | Inter-zone transfer buffer |
| Charging dock | `charge_dock_1` | Auto-recharge (one per map) |
| Elevator point (iMRP600) | `elevator_L1`, `elevator_L2` | Cross-floor connection points |

### 3. Set Navigation Parameters

| Parameter | Recommended Warehouse Value | ROPA Service |
|-----------|----------------------------|--------------|
| Speed mode | **Balance high** (cmd=5) for open areas | `call_service /velocity_control` |
| Speed mode | **Safety low** (cmd=0) for crowded aisles | Same |
| Travel mode | **Efficient** for long straight corridors | DeploymentTool Settings |
| Obstacle threshold | 0.3 m (default) for standard aisles | `publish /laser_safety_range` |

> **Tip:** Use ROPA `/velocity_control` to dynamically switch speed modes based on zone. Fast in open transit corridors, slow in active picking aisles.

### 4. Configure Multi-POI Routes

For warehouse patrol / cyclic transport:

- **Single pass** — visit each station once (order fulfillment route)
- **Round-trip** — pick → pack → return to start (single order delivery)
- **Cycle** — continuous patrol between staging areas (inventory shuttle)

Implement via sequential ROPA `/poi` service calls:

```python
# Multi-point warehouse delivery cycle
destinations = ["pick_zone_A", "pack_station_1", "staging_outbound"]

for poi in destinations:
    ws.send(json.dumps({
        "op": "call_service",
        "id": f"nav_{poi}",
        "service": "/poi",
        "args": {"poi_name": poi, "poi_floor": "1", "poi_building": "Warehouse"}
    }))
    # Wait for navi_status == 603 (success) before next destination
```

### 5. Map Safety Zones

| Zone Type | Where to Apply | ROPA Effect |
|-----------|---------------|-------------|
| **Deceleration zone** | Rack intersections, dock doors, pedestrian aisles | Robot slows automatically |
| **Dangerous zone** | Loading dock edges, forklift operating areas | Maximum caution navigation |
| **Virtual wall** | Cold storage doors (robot should not enter), staff-only zones | Robot never crosses |
| **Strong light zone** | Near warehouse skylights, dock doors with sunlight | Sensor parameter adjustment |

### 6. Charging Setup

For 24/7 warehouse operation, plan charging strategically:

- Place charging dock at **least-used corridor** to avoid blocking traffic
- Robot auto-docks when `battery < threshold` (monitor via `/robot_status`)
- 12h endurance on iMRP500/600 means **one charge cycle covers a full shift**
- For continuous operation, schedule dock returns during low-demand periods

---

## Application Patterns

### Pattern A: Shelf-to-Shelf Inventory Shuttle (iMRP500)

Continuous cycle between `pick_zone_A` → `pack_station_1` → `staging_outbound` → return.

- 50 kg payload covers most shelf bins and tote boxes
- ±5 cm positioning accuracy ensures precise dock alignment at pick stations

### Pattern B: Cross-Floor Pallet Transport (iMRP600)

Move heavy loads between warehouse floors:

- Autonomous elevator riding — no manual intervention
- Dual-LiDAR for reliable navigation in varying floor environments
- 100 kg rated payload for pallet-sized loads

See detailed setup: [multi-floor deployment](multi-floor.md)

### Pattern C: Factory Rapid Transit (iMRP800)

Fast, stable transport across wide factory corridors:

- 1.2 m/s transit speed — 3× faster than standard AMR in open areas
- 800 mm wide base — stable under 100 kg load, no tipping at speed
- Ideal for cleanroom logistics where rapid point-to-point matters

---

## Monitoring Dashboard

Build a warehouse operations dashboard using ROPA subscribe topics:

| Metric | ROPA Topic | Field | How to Use |
|--------|------------|-------|------------|
| Robot location | `/robot_pose` | `x`, `y`, `theta` | Track fleet positions on warehouse map |
| Delivery status | `/navi_status` | `status` | 603=delivered, 604=failed, 601=in transit |
| Battery level | `/robot_status` | `battery` | Schedule charging, prevent mid-task dock |
| Obstacle detection | `/obstacle_region` | `region` | Monitor aisle congestion |
| Localization quality | `/localization_confidence` | `confidence` | Alert if robot loses position reference |
| System health | `/notification` | `name`, `level` | LiDAR errors, drive anomalies |

---

## Path Distance Calculation

Use ROPA `/calculate_distance` service to plan optimal routes:

```json
{
  "op": "call_service",
  "id": "route_planning",
  "service": "/calculate_distance",
  "args": {
    "start_x": 1.0,
    "start_y": 2.0,
    "start_floor": "1",
    "goal_x": 15.0,
    "goal_y": 8.0,
    "goal_floor": "1"
  }
}
```

Response: `values.distance` in meters. Use this to:
- Compare route lengths between alternative POI sequences
- Estimate transit time (distance / speed mode)
- Validate that navigation goals are reachable (result=0 success, 150 unreachable)

---

## Troubleshooting

| Issue | Cause | Fix |
|-------|-------|-----|
| Robot drifts off path | Accumulated odometry error | Run global localization via DeploymentTool |
| Navigation fails at intersection | Map outdated (aisle reconfigured) | Re-map or edit virtual walls |
| Battery drains fast | High speed + heavy payload + long route | Switch to balanced/low speed, optimize route |
| Localization confidence low | Featureless corridor (long blank walls) | Add distinctive POI markers, increase confidence threshold |
| Robot stops unexpectedly | Obstacle threshold triggered | Check `/obstacle_region` — adjust safety range |

---

## Next Steps

- **Multi-floor setup**: [Cross-floor deployment guide](multi-floor.md) for iMRP600 elevator riding
- **Integrate with WMS**: See [ROS-bridgeable setup](../integration/ros-bridge.md) for warehouse management system integration
- **Custom payload hardware**: [Custom hardware attachment](../integration/custom-hardware.md) for shelf bins, conveyor attachments
- **Choose your package**: [Developer Kit](../kits/developer-kit.md) vs [Integrator Package](../kits/integrator-kit.md)

---

## Get the Hardware

👉 [Roviano on Alibaba.com](https://roviano.en.alibaba.com) — iMRP500 / iMRP600 / iMRP800

**Contact:** engineering@roviano.com
