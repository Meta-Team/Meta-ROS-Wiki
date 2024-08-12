# meta_hardware

`meta_hardware` package is a `ros2_control` compatible hardware interface that implements management of DJI and MI motors.

## List of hardware interfaces

| Motor Vendor | Interface Name               |
| ------------ | ---------------------------- |
| DJI          | `MetaRobotDjiMotorNetwork`   |
| MI           | `MetaRobotMiMotorNetwork`    |
| BRITER       | `MetaRobotBrtEncoderNetwork` |

## Supported motors

```{eval-rst}
+--------------+-------------+----------+
| Motor vendor | Motor Model | Protocol |
+--------------+-------------+----------+
| DJI          | GM6020      | CAN      |
|              +-------------+----------+
|              | M3508       | CAN      |
|              +-------------+----------+
|              | M2006       | CAN      |
+--------------+-------------+----------+
| MI           | CyberGear   | CAN      |
+--------------+-------------+----------+
| BRITER       | BRT38       | CAN      |
+--------------+-------------+----------+

```

## URDF configuration

To activate a motor network (a series of motors on a can network belonging to a single vendor), you simply need to create a `ros2_control` tag with motor information in your robot URDF.

### Motor network params

`can_network_name`
: The CAN interface name of the CAN network, usually `can0` or `can1`.

### Common motor params

`motor_model`
: The model name of the motor.

`mechanical_reduction` `offset`
: The mechanical reduction rate of the motor and the offset of the motor zero position with respect to joint zero position.

```{math}
  \theta_{\text{joint}} =
  \frac{\theta_{\text{motor}}}{\text{mechanical_reduction}} + \text{offset}
```

### DJI motor params

`motor_id`
: The DJI motor ID, can be adjusted by DIP switches on the motor, as described in DJI documentation.

### MI motor params

`motor_id`
: The MI motor ID, adjustable via MI motor debugging software.

`control_mode`
: The operating mode of the motor. Can be `dynamic`, `position`, or `velocity`.

  - Dynamic control mode is a flexible control mode that allows three inputs: `pos_dst`, `vel_dst`, and `t_ff`, the actual torque is computed by th following formula:

    ```{math}
    \tau = K_{\text{p}} \cdot (\theta_{\text{dst}} - \theta) + K_{\text{d}} \cdot (\omega_{\text{dst}} - \omega) + \tau_{\text{ff}}
    ```

    `MetaRobotMiMotorNetwork` accepts partial inputs to reduce user's complexity:

    - If only `position` interface is present, `pos_dst = position`, `vel_dst = 0`, `t_ff = 0`.
    - If only `velocity` interface is present, `pos_dst = 0`, `vel_dst = velocity`, `t_ff = 0`.
    - If only `effort` interface is present, `pos_dst = 0`, `vel_dst = 0`, `t_ff = effort`.
    - If both `position` and `effort` interfaces are present, `pos_dst = position`, `vel_dst = 0`, `t_ff = effort`.
    - If both `velocity` and `effort` interfaces are present, `pos_dst = 0`, `vel_dst = velocity`, `t_ff = effort`.
    - If all interfaces are present, `pos_dst = position`, `vel_dst = velocity`, `t_ff = effort`.
    - The combination of `position` and `velocity` interfaces is not considered valid, as this generally doesn't generate a stable system.

  - Position control mode is an internal dual-loop PI controller that controls the motor's position.
  - Velocity control mode is an internal PI controller that controls the motor's velocity.

  Please refer to MI motor's documentation for more information on control modes.

`Kp` `Kd`
: Parameters used in dynamic control mode. Refer to MI motor's documentation for more information.

`spd_kp` `spd_ki`
: Parameters used in velocity and position control mode. Refer to MI motor's documentation for more information.

`pos_kp` `pos_ki`
: Parameters used in position control mode. Refer to MI motor's documentation for more information.

### Example URDF configuration

A robot with a DJI GM6020 and MI CyberGear (in dynamic control mode with a target position) can be configured as follows.

```xml
<ros2_control name="motor_control_dji" type="system">
    <hardware>
        <plugin>meta_hardware/MetaRobotDjiMotorNetwork</plugin>
        <param name="can_network_name">can0</param>
    </hardware>
    <joint name="motor1_joint">
        <command_interface name="effort" />
        <state_interface name="position" />
        <state_interface name="velocity" />
        <state_interface name="effort" />
        <param name="motor_model">GM6020</param>
        <param name="motor_id">1</param>
        <param name="mechanical_reduction">1.0</param>
        <param name="offset">0.0</param>
    </joint>
</ros2_control>

<ros2_control name="motor_control_mi" type="system">
    <hardware>
        <plugin>meta_hardware/MetaRobotMiMotorNetwork</plugin>
        <param name="can_network_name">can0</param>
    </hardware>
    <joint name="motor2_joint">
        <command_interface name="position" />
        <state_interface name="position" />
        <state_interface name="velocity" />
        <state_interface name="effort" />
        <param name="motor_model">CyberGear</param>
        <param name="motor_id">1</param>
        <param name="mechanical_reduction">1.0</param>
        <param name="offset">0.0</param>
        <param name="control_mode">dynamic</param>
        <param name="Kp">30.0</param>
        <param name="Kd">1.0</param>
    </joint>
</ros2_control>
```
