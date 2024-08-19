# gimbal_controller

The `gimbal_controller` package is a `ros2_control` compatible (chainable) controller responsible for controlling gimbal motors.

The controller is a simple dual-loop PID. But the feedback comes from an IMU sensor rather than an encoder.

```{mermaid}
flowchart LR
    A[Position Reference] --> B[Position PID]
    B --> D[Velocity Command]
    D --> E[Velocity PID]
    E --> F[Effort Command]
    F --> H[Motor]

    H --> G[IMU]
    G --> |Position Feedback| B
    G --> |Velocity Feedback| E
```

## Controller interfaces

### Reference interfaces (in chained mode)

- `yaw/position`
- `pitch/position`

### Command interfaces

- `<yaw_gimbal_joint.name>/effort`
- `<pitch_gimbal_joint.name>/effort`

### State interfaces

No state interface is required as IMU feedback comes from ROS topic (`<imu_topic>`).

### Subscribers

- `<imu_topic>` [[sensor_msgs/msg/Imu](https://docs.ros2.org/foxy/api/sensor_msgs/msg/Imu.html)]

- `<controller_name>/reference` [behavior_interface/msg/Aim] (if not in chained mode)

### Publishers

- `<controller_name>/controller_state`

## Parameters

`yaw_gimbal_joint.name` (string)
: Specifies the name of yaw gimbal joint.

`yaw_gimbal_joint.enable` (bool)
: Specifies if the yaw gimbal joint should be enabled.

`pitch_gimbal_joint.name` (string)
: Specifies the name of pitch gimbal joint.

`pitch_gimbal_joint.enable` (bool)
: Specifies if the pitch gimbal joint should be enabled.

`imu_topic` (string)
: Specifies the topic name where gimbal IMU data is published.
