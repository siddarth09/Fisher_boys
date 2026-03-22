# AIC MuJoCo Scene

Converted from Gazebo SDF using `sdf2mjcf`, then split and augmented.

## Directory Structure

```
aic_mujoco_scene/
├── scene.xml           # Top-level file - load this one
├── aic_robot.xml       # UR5e + gripper (actuators, sensors, equality constraints)
├── aic_world.xml       # Environment (enclosure, floor, task board, cable)
├── meshes/             # All OBJ/STL mesh files (copied from sdf2mjcf output)
└── README.md
```

## Usage

### Quick test
```bash
python3 -m mujoco.viewer --mjcf scene.xml
```

### In Python
```python
import mujoco
model = mujoco.MjModel.from_xml_path('scene.xml')
data = mujoco.MjData(model)

# UR5e joints (indices 0-5 in actuator array)
data.ctrl[0] = 0.0   # shoulder_pan
data.ctrl[1] = -1.57  # shoulder_lift
data.ctrl[2] = 1.57   # elbow
data.ctrl[3] = -1.57  # wrist_1
data.ctrl[4] = -1.57  # wrist_2
data.ctrl[5] = 0.0    # wrist_3
data.ctrl[6] = 0.01   # gripper (0=open, 0.025=closed)

mujoco.mj_step(model, data)
```

## Actuators

| Index | Name | Joint | Type | Range |
|-------|------|-------|------|-------|
| 0 | act_shoulder_pan_joint | shoulder_pan_joint | position | [-6.28, 6.28] |
| 1 | act_shoulder_lift_joint | shoulder_lift_joint | position | [-6.28, 6.28] |
| 2 | act_elbow_joint | elbow_joint | position | [-3.14, 3.14] |
| 3 | act_wrist_1_joint | wrist_1_joint | position | [-6.28, 6.28] |
| 4 | act_wrist_2_joint | wrist_2_joint | position | [-6.28, 6.28] |
| 5 | act_wrist_3_joint | wrist_3_joint | position | [-6.28, 6.28] |
| 6 | act_gripper | gripper left_finger_joint | position | [0, 0.025] |

## Sensors

- `force_AtiForceTorqueSensor` / `torque_AtiForceTorqueSensor` - Axia80 F/T sensor
- `sensor_<joint>_pos` / `sensor_<joint>_vel` - Joint position/velocity for all 6 UR5e joints

## Equality Constraints

- `gripper_mimic` - Right finger mirrors left finger position

## Contact Exclusions

- `tabletop` <-> `shoulder_link` (robot base mount)
- `gripper_fingers` left <-> right (allow gripping without self-collision)
- `task_board` <-> `sc_port_0` (board-mounted components)
- `task_board` <-> `nic_card_mount_0` (board-mounted components)

## Notes

- Mesh files must be in the `meshes/` subdirectory
- Cable uses `freejoint` + ball joints for flexible body simulation
- Camera sensors (center/left/right) are rudimentary conversions from Gazebo
- For RL training, you can randomize task_board pose in `aic_world.xml`