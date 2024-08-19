# omni_chassis_controller

The `omni_chassis_controller` package is a `ros2_control` compatible (chainable) controller responsible for controlling omnidirectional wheel (omni wheel and mecanum wheel) chassis.

## Controller interfaces

### Reference interfaces (in chained mode)

- `linear/x/velocity`
- `linear/y/velocity`
- `angular/z/velocity`

### Command interfaces

- `<omni_wheel_joints[i]>/velocity`

### State interfaces

No state interface for wheels is required as inverse kinematics is completely open-loop.

If the controller is not in `CHASSIS` mode, `<yaw_gimbal_joint>/position` is required.

### Subscribers

- `<controller_name>/reference` [[geometry_msgs/Twist](https://docs.ros2.org/galactic/api/geometry_msgs/msg/Twist.html)] (if not in chained mode)

### Publishers

- `<controller_name>/controller_state`

## Parameters

`omni_wheel_joints` (string_array)
: Specifies the name of omni wheel joints.

`omni_wheel_forward_angles` (double_array)
: Specifies the forward angle of each omni wheel (in degrees). [^nw-rb]

`omni_wheel_center_x` (double_array)
: Specifies the x-coordinate of the center of each omni wheel.  [^nw-rb]

`omni_wheel_center_y` (double_array)
: Specifies the y-coordinate of the center of each omni wheel.  [^nw-rb]

`omni_wheel_sliding_angle` (double_array)
: Specifies the sliding angle of each omni wheel (in degrees).  [^nw-rb]

`omni_wheel_radius` (double)
: Specifies the radius of each omni wheel.

`yaw_gimbal_joint` (string)
: Specifies the name of yaw gimbal joint.

`control_mode` (string)
: Specifies the control mode of the controller. Available options are `CHASSIS`, `GIMBAL` and `CHASSIS_FOLLOW_GIMBAL`.

  - `CHASSIS`: The reference twist is in the chassis frame.
  - `GIMBAL`: The reference twist is in the gimbal frame.
  - `CHASSIS_FOLLOW_GIMBAL`: On top of `GIMBAL` mode, the chassis will attempt to follow the gimbal.

`follow_pid_target` (double)
: Specifies the target of chassis follow PID controller.

`reference_timeout` (double)
: Specifies the timeout of reference.

[^nw-rb]: Consult this [video](https://modernrobotics.northwestern.edu/nu-gm-book-resource/13-2-omnidirectional-wheeled-mobile-robots-part-1-of-2/) from Northwestern University for more information on the parameters.
