# Multi-Floor Building Logistics — iMRP600 Autonomous Elevator Riding

> Applies to: iMRP600 (exclusive elevator capability)
> Required: Integrator Package recommended, algorithm ≥ V4.5.0, compatible elevator controller
> Prerequisites: Read [indoor-service deployment](indoor-service.md) and [warehouse deployment](warehouse.md) for single-floor fundamentals

---

## Overview

The iMRP600 is the **only iMRP model with autonomous elevator riding** — the robot calls, enters, rides, and exits elevators without human intervention. This enables true multi-floor logistics in hospitals, hotels, office towers, and multi-story warehouses.

> **iMRP600 exclusive feature.** iMRP400, iMRP500, and iMRP800 do not support autonomous elevator riding.

---

## Prerequisites

| Requirement | Detail |
|-------------|--------|
| Algorithm version | ≥ V4.5.0 (check via `call_service /robot_info` → `software_version`) |
| Elevator controller | Serial / CAN / TCP interface compatible with building elevator system |
| Floor maps | Each floor must be mapped separately before linking |
| Hardware setup | Elevator controller wiring must be completed by building facilities team |

---

## Step-by-Step Setup

### 1. Map Each Floor

Map every floor the robot will operate on, using DeploymentTool:

| Floor | Action | ROPA |
|-------|--------|------|
| Floor 1 | Standard mapping mode → save as "Floor1" in "Building1" | `call_service /node_manager_control` (cmd=0, then cmd=3) |
| Floor 2 | Switch map → map floor 2 → save as "Floor2" in "Building1" | Same sequence |
| Floor N | Repeat for each floor | Same |

> **Important:** Map each floor from scratch. The robot needs independent floor maps to localize correctly on each level.

### 2. Define Elevator Connection Points

On **each floor map**, create a POI at the elevator door:

| Floor | POI Name | Location |
|-------|----------|----------|
| Floor 1 | `elevator_L1_door` | Directly outside elevator door on floor 1 |
| Floor 2 | `elevator_L2_door` | Directly outside elevator door on floor 2 |
| Floor N | `elevator_LN_door` | Directly outside elevator door on floor N |

These POI are the cross-floor navigation waypoints. When the robot receives a navigation command to a POI on another floor, it will:
1. Navigate to the elevator POI on the current floor
2. Call the elevator
3. Enter when doors open
4. Ride to target floor
5. Exit when doors open on target floor
6. Navigate to destination POI on target floor

### 3. Configure Elevator Parameters

In DeploymentTool → **Settings** → **Elevator**:

| Parameter | Description | Example |
|-----------|-------------|---------|
| Elevator ID | Unique identifier per elevator | `elevator_A` |
| Floor mapping | Map elevator floor numbers to robot floor maps | `1 → Floor1, 2 → Floor2` |
| Door wait time | Seconds robot waits for doors to open | `15` |
| Communication protocol | Interface to elevator controller | Serial / CAN / TCP |

> Elevator controller wiring and protocol configuration typically require collaboration with building facilities management. **Integrator Package** includes on-site assistance for this step.

### 4. Test Single-Floor First

Before attempting cross-floor navigation:

1. Verify the robot navigates correctly on each floor independently
2. Verify elevator POI positions are accurate (robot stops precisely at elevator door)
3. Verify elevator controller communication (test elevator call from robot position)

### 5. Execute Cross-Floor Navigation

```json
{
  "op": "call_service",
  "id": "cross_floor_delivery",
  "service": "/poi",
  "args": {
    "poi_name": "room_301",
    "poi_floor": "3",
    "poi_building": "Hospital"
  }
}
```

The robot automatically handles the entire cross-floor sequence. Monitor via `/navi_status`:

| Status Code | Stage |
|-------------|-------|
| 639 | Cross-floor navigation waiting (initializing) |
| 640 | Feasibility detection (verifying route) |
| 641 | Navigate to elevator exterior (on current floor) |
| 642 | Wait for elevator at designated floor |
| 643 | Entering elevator |
| 644 | Switching map (loading target floor map) |
| 645 | Leaving elevator (on target floor) |
| 646 | Navigate to target point |
| 647 | Cross-floor navigation complete ✅ |

### 6. Monitor Cross-Floor Status

```python
# Subscribe to cross-floor navigation progress
ws.send(json.dumps({
    "op": "subscribe",
    "topic": "/navi_status",
    "type": "navi_status_msg"
}))

# Also monitor robot status for floor tracking
ws.send(json.dumps({
    "op": "subscribe",
    "topic": "/robot_status",
    "type": "robot_status_msg"
}))
# → "building" and "floor" fields update as robot transitions
```

---

## Error Codes & Troubleshooting

| Code | Meaning | Fix |
|------|---------|-----|
| 630 | Input error | Check POI name, floor, building parameters |
| 631 | Target floor invalid | Verify floor exists in robot's building config |
| 632 | Target marker invalid | Verify POI exists on target floor map |
| 633 | Marker retrieval error | Re-sync POI list via `/marker_operation/get_markers` |
| 634 | Elevator point invalid | Verify elevator POI on both source and target floors |
| 635 | Cross-floor navigation failed | Check elevator controller connection, retry |
| 636 | Cross-floor navigation cancelled | Normal if `/move_base/cancel` was sent |
| 637 | Elevator reservation failed | Elevator controller not responding — check wiring |

---

## Application Patterns

### Pattern A: Hospital Multi-Floor Delivery

- Floor 1: Pharmacy → elevator → Floor 3: Ward delivery
- Robot carries medication trays (≤50 kg recommended payload)
- Returns to pharmacy via reverse elevator ride

### Pattern B: Hotel Room Service Across Floors

- Floor 1: Kitchen → elevator → Floor 5: Guest room
- Robot carries food trays, returns to kitchen
- Concierge POI at elevator hall on each floor for staging

### Pattern C: Multi-Story Warehouse

- Floor 1: Receiving dock → elevator → Floor 2: Storage rack zone
- Heavy inventory transport (≤100 kg rated, ≤50 kg recommended for safety)
- Continuous shuttle cycle between floors

---

## Safety Considerations

| Concern | Mitigation |
|---------|------------|
| Robot trapped in elevator | Set `time_out` parameter on elevator POI — robot cancels if doors don't open |
| Elevator door closes on robot | Configure door wait time with safety margin (recommend 15+ seconds) |
| Localization failure on new floor | Monitor `/localization_confidence` — if <0.8 after map switch, robot pauses navigation |
| Emergency in elevator | Hardware E-stop button accessible — immediate motor cutoff |

> **Recommendation:** For production multi-floor deployments, choose the **Integrator Package** — our engineers handle elevator controller integration and on-site verification.

---

## ROPA API Summary for Multi-Floor

| Function | ROPA Call | Notes |
|----------|----------|-------|
| Navigate to cross-floor POI | `call_service /poi` | Include `poi_floor` and `poi_building` |
| Get all POI markers | `call_service /marker_operation/get_markers` | Verify elevator POI exist |
| Check robot floor | `subscribe /robot_status` | `building` + `floor` fields |
| Monitor cross-floor progress | `subscribe /navi_status` | Codes 630–647 |
| Cancel cross-floor navigation | `publish /move_base/cancel` | Safe cancellation at any stage |
| Emergency stop | `publish /soft_stop` | Immediate halt |

---

## Next Steps

- **Single-floor fundamentals**: [indoor-service deployment](indoor-service.md)
- **Warehouse logistics**: [warehouse deployment](warehouse.md)
- **Integrate with building management**: [ROS-bridgeable setup](../integration/ros-bridge.md)
- **Choose your package**: [Developer Kit](../kits/developer-kit.md) vs [Integrator Package](../kits/integrator-kit.md)

---

## Get the Hardware

👉 [Roviano on Alibaba.com](https://roviano.en.alibaba.com) — iMRP600 (Autonomous Elevator Riding)

**Contact:** engineering@roviano.com
