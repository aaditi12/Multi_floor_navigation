# Multi_floor_navigation
This is the full ROS 2 Humble multi-floor nav workspace (Gazebo + single Nav2 stack). Everything needed is present in `src/` — the empty `{worlds,urdf,maps,...}` literal folders and the duplicate nested `multifloor_navigation_ws/multifloor_navigation_ws/` copy are just leftover packaging junk (empty/older), safe to ignore.

**Setup (one-time):**
```bash
sudo apt install \
  ros-humble-navigation2 \
  ros-humble-nav2-bringup \
  ros-humble-gazebo-ros-pkgs \
  ros-humble-robot-state-publisher \
  ros-humble-xacro
```

**Build:**
```bash
cd multifloor_navigation_ws
source /opt/ros/humble/setup.bash
colcon build --symlink-install
source install/setup.bash
```

**Run — full sim (Gazebo GUI + Nav2 + all nodes):**
```bash
ros2 launch multifloor_bringup bringup.launch.py
```

**Run headless (no GUI, faster):**
```bash
ros2 launch multifloor_bringup bringup.launch.py headless:=true
```

**Send a cross-floor mission (Floor 1 → Floor 2):**
```bash
ros2 topic pub --once /mission_goal_floor std_msgs/msg/String \
  "data: 'floor:2,x:20.0,y:5.0'"
```

**Send a same-floor mission:**
```bash
ros2 topic pub --once /mission_goal_floor std_msgs/msg/String \
  "data: 'floor:1,x:8.0,y:7.0'"
```

**Monitor:**
```bash
ros2 topic echo /mission_status
ros2 topic echo /current_floor_info
ros2 topic echo /elevator_state
```

**Fix AMCL if localization drifts:**
```bash
ros2 topic pub --once /initialpose geometry_msgs/msg/PoseWithCovarianceStamped \
  "{header: {frame_id: map}, pose: {pose: {position: {x: 2.0, y: 5.0}}, \
   covariance: [0.25,0,0,0,0,0, 0,0.25,0,0,0,0, 0,0,0,0,0,0, \
                0,0,0,0,0,0, 0,0,0,0,0,0, 0,0,0,0,0,0.07]}}"
```

**Regenerate maps if you edit the world:**
```bash
cd src/multifloor_gazebo/maps
python3 generate_maps.py .
```

One thing worth flagging before you run this on `pi-node1`: Gazebo Classic + Nav2 together is heavy for a Pi — if it's the target, `headless:=true` is basically mandatory, and you may still want to run Gazebo/RViz on a separate machine with `ROS_DOMAIN_ID` matched.
