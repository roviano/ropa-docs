# ROS-bridgeable Setup — Connect ROPA to Your ROS Stack

> Applies to: iMRP400 / iMRP500 / iMRP600 / iMRP800
> No ROS2 installation required on the robot — ROPA is a standalone WebSocket+JSON protocol
> You bridge ROPA data into your existing ROS application on your own host machine

---

## What "ROS-bridgeable" Means

The iMRP chassis runs **ROPA** — a WebSocket protocol on port 9090 using JSON messages. It does not run ROS internally. "ROS-bridgeable" means **you can stream every sensor, pose, and navigation data point from ROPA into your ROS application** through a lightweight bridge node, without modifying the robot firmware or installing ROS on the chassis.

| Myth | Reality |
|------|---------|
| "You need to install ROS2 on the robot" | **False.** ROPA runs on the robot. ROS runs on your host. The bridge connects them. |
| "You must rewrite your application in ROPA" | **False.** Write a thin bridge node once, then use standard ROS tools. |
| "ROPA replaces ROS" | **False.** ROPA gives you **raw data access** — LiDAR, pose, status, maps. You bring your own planner. |

---

## Architecture

```
┌─────────────┐     WebSocket      ┌──────────────────────┐
│  iMRP Chassis│  ←── port 9090 ──→ │  Your Host Machine   │
│  (ROPA Server)│     JSON messages  │  ┌─────────────────┐ │
│              │                     │  │ ropa_bridge_node │ │
│  LiDAR       │                     │  │ (Python/JS)      │ │
│  RGBD        │                     │  ├─────────────────┤ │
│  Ultrasonic  │                     │  │ ROS Topics       │ │
│  IMU         │                     │  │ /robot_pose      │ │
│  Motors      │                     │  │ /laser_scan      │ │
│  Navigation  │                     │  │ /robot_status    │ │
└─────────────┘                     │  │ /map             │ │
                                    │  │ ...              │ │
                                    │  └─────────────────┘ │
                                    │  Your ROS Application │
                                    └──────────────────────┘
```

The bridge node:
1. Opens WebSocket to `ws://<ROBOT_IP>:9090`
2. Subscribes to ROPA topics (`/robot_pose`, `/laser_data`, `/robot_status`, etc.)
3. Converts ROPA JSON messages to ROS message types
4. Publishes into your ROS topic tree

---

## Quick Start Bridge Node (Python)

### Dependencies

- Python 3.7+
- `websocket-client` package (`pip install websocket-client`)
- Your ROS environment (ROS1 or ROS2 — the bridge works with either)

### Bridge Implementation

```python
#!/usr/bin/env python3
"""
ropa_bridge.py — Stream ROPA data into ROS topics.

ROPA → ROS topic mapping:
  /robot_pose     → geometry_msgs/Pose2D
  /laser_data     → sensor_msgs/LaserScan
  /robot_status   → std_msgs/String (JSON-encoded)
  /navi_status    → std_msgs/Int32
  /map            → nav_msgs/OccupancyGrid
"""

import json
import websocket
import math

# ROS1 example — for ROS2, use rclpy with equivalent message types
import rospy
from geometry_msgs.msg import Pose2D
from sensor_msgs.msg import LaserScan
from nav_msgs.msg import OccupancyGrid
from std_msgs.msg import String, Int32

ROBOT_IP = "10.42.0.1"  # Default WiFi IP
ROPA_PORT = 9090

# ROS publishers
pub_pose = rospy.Publisher("/robot_pose", Pose2D, queue_size=10)
pub_laser = rospy.Publisher("/laser_scan", LaserScan, queue_size=10)
pub_status = rospy.Publisher("/robot_status_ros", String, queue_size=10)
pub_navi = rospy.Publisher("/navi_status", Int32, queue_size=10)

def on_message(ws, message):
    data = json.loads(message)

    if data.get("op") != "publish":
        # Handle service_response, etc.
        if data.get("op") == "service_response":
            rospy.loginfo(f"Service response: {data.get('service')} → result={data.get('result')}")
        return

    topic = data.get("topic")
    msg = data.get("msg", {})

    if topic == "/robot_pose":
        p = Pose2D()
        p.x = msg.get("x", 0.0)
        p.y = msg.get("y", 0.0)
        p.theta = msg.get("theta", 0.0)
        pub_pose.publish(p)

    elif topic == "/laser_data":
        scan = LaserScan()
        scan.header.stamp = rospy.Time.now()
        scan.angle_min = -math.pi
        scan.angle_max = math.pi
        scan.angle_increment = 2 * math.pi / len(msg.get("px", []))
        scan.range_min = 0.1
        scan.range_max = 40.0  # TOF LiDAR 40m range
        px = msg.get("px", [])
        py = msg.get("py", [])
        scan.ranges = [math.sqrt(x**2 + y**2) for x, y in zip(px, py)]
        pub_laser.publish(scan)

    elif topic == "/robot_status":
        pub_status.publish(json.dumps(msg))

    elif topic == "/navi_status":
        pub_navi.publish(Int32(data=msg.get("status", 0)))

def on_error(ws, error):
    rospy.logerr(f"ROPA WebSocket error: {error}")

def on_close(ws, close_status, close_msg):
    rospy.logwarn(f"ROPA WebSocket closed: {close_status} {close_msg}")

def on_open(ws):
    rospy.loginfo("Connected to ROPA. Subscribing to topics...")
    # Subscribe to all key topics
    for topic in ["/robot_pose", "/laser_data", "/robot_status",
                  "/navi_status", "/obstacle_region",
                  "/localization_confidence", "/notification"]:
        ws.send(json.dumps({
            "op": "subscribe",
            "topic": topic,
            "type": "auto",
            "throttle_rate": 100
        }))

if __name__ == "__main__":
    rospy.init_node("ropa_bridge")
    ws = websocket.WebSocketApp(
        f"ws://{ROBOT_IP}:{ROPA_PORT}",
        on_open=on_open,
        on_message=on_message,
        on_error=on_error,
        on_close=on_close
    )
    ws.run_forever()
```

---

## ROPA → ROS Topic Mapping

| ROPA Topic | ROS Message Type | Key Fields | Notes |
|------------|-----------------|------------|-------|
| `/robot_pose` | `geometry_msgs/Pose2D` | x, y, theta | Direct 1:1 mapping |
| `/laser_data` | `sensor_msgs/LaserScan` | ranges[] | Convert px/py to range via sqrt(x²+y²) |
| `/robot_status` | JSON on `std_msgs/String` | battery, charger, estop | Full status object |
| `/navi_status` | `std_msgs/Int32` | status code | 600–604 range |
| `/map` | `nav_msgs/OccupancyGrid` | data, width, height, resolution | Base64 decode PNG → occupancy array |
| `/obstacle_region` | `std_msgs/Int32` | region bitmask | 0=clear, 2=front, etc. |
| `/localization_confidence` | `std_msgs/Float32` | confidence | 0.0–1.0 |
| `/global_path` | `nav_msgs/Path` | poses[] | Planned navigation path |

---

## Sending Commands Back to ROPA

Your ROS application can also **send navigation and control commands** through the bridge:

```python
# Navigate to a POI from your ROS application
ws.send(json.dumps({
    "op": "call_service",
    "id": "ros_nav_command",
    "service": "/poi",
    "args": {"poi_name": "dock_A", "poi_floor": "1", "poi_building": "Building1"}
}))

# Teleop control from ROS cmd_vel
def cmd_vel_callback(msg):
    ws.send(json.dumps({
        "op": "publish",
        "topic": "/cmd_vel_mux/input/teleop",
        "msg": {
            "linear": {"x": msg.linear.x, "y": 0, "z": 0},
            "angular": {"x": 0, "y": 0, "z": msg.angular.z}
        }
    }))
```

---

## What You Can Do With This Bridge

| Application | How |
|-------------|-----|
| Use your own planner | Subscribe `/robot_pose` + `/laser_data` → plan paths → send via `/poi` or `/cmd_vel_mux/input/teleop` |
| Build monitoring dashboards | Subscribe all status topics → aggregate in your ROS visualization tools |
| Integrate with WMS / BMS | Bridge robot status into your warehouse/building management system via ROS middleware |
| Research & experimentation | Full sensor access (LiDAR, IMU, RGBD, ultrasonic) for algorithm development |
| Fleet management | Bridge multiple robots → central ROS coordinator → `/poi` commands per robot |

---

## Important Notes

1. **ROPA is not ROS.** The robot does not run ROS internally. The bridge is your integration layer.
2. **No robot firmware modification.** ROPA is the stable, documented API. Your bridge adapts it to your environment.
3. **One protocol, all models.** iMRP400/500/600/800 all use the same ROPA on port 9090. Your bridge works universally.
4. **Full data access.** Every sensor, every motor, every navigation command is exposed via ROPA — no black-box gaps.

---

## Next Steps

- **Protocol details**: [ROPA Protocol Reference](https://github.com/roviano/ropa-sdk/blob/main/docs/protocol.md)
- **Code examples**: [Python/JS examples in ropa-sdk](https://github.com/roviano/ropa-sdk/tree/main/examples)
- **Map your workspace**: [Android Mapping workflow](android-mapping.md)
- **Choose your package**: [Developer Kit](../kits/developer-kit.md) vs [Integrator Package](../kits/integrator-kit.md)

---

## Get the Hardware

👉 [Roviano on Alibaba.com](https://roviano.en.alibaba.com) — iMRP400 / 500 / 600 / 800

**Contact:** engineering@roviano.com
