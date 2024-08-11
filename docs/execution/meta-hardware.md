# meta_hardware

`meta_hardware` package is a `ros2_control` compatible hardware interface that implements management of DJI and MI motors.

## List of hardware interfaces

| Motor Vendor | Interface Name             |
| ------------ | -------------------------- |
| DJI          | `MetaRobotDjiMotorNetwork` |
| MI           | `MetaRobotMiMotorNetwork`  |

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

```

## URDF configuration
To activate a motor network (a series of motors on a can network belonging to a single vendor), you simply need to create a `ros2_control` tag with motor information in your robot URDF.

### Motor network params
`can_network_name`
: The CAN interface name of the CAN network, usually `can0` or `can1`.

### Common motor params
`motor_model`
: The model name of the motor.

`mechanical_reduction`
: The mechanical reduction rate of the motor. This is only relevant to motors with internal gearboxes.

`offset`
: The offset of the motor zero position with respect to joint zero position. This value is always measured in the joint coordinate space.

### DJI motor params

`motor_id`
: The DJI motor ID, can be adjusted by DIP switches on the motor, as described in DJI documentation.

### MI motor params

`motor_id`
: The MI motor ID, adjustable via MI motor debugging software.

`Kp` `Kd`
: Parameters used in dynamic control mode. Kp stands for proportion (position) and Kd stands for differential (velocity).

### Example URDF configuration

A robot with a DJI GM6020 and MI CyberGear can be configured as follows.

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