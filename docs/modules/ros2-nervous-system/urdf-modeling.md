---
sidebar_position: 4
title: "Humanoid Modeling with URDF"
---

# Humanoid Modeling with URDF

## Learning Objectives

By the end of this chapter, you will be able to:
- Create proper URDF models with links, joints, and kinematic properties
- Understand the purpose of URDF and its structure
- Include sensors and actuators in URDF models for deployment readiness
- Visualize and validate URDF models in ROS 2 tools
- Apply best practices for humanoid robot modeling

## Introduction to URDF

URDF (Unified Robot Description Format) is an XML-based format used in ROS to describe robot models. It defines the physical and visual properties of a robot, including its links, joints, and kinematic relationships. URDF is crucial for simulation, visualization, and understanding robot structure in ROS-based systems.

### Why URDF is Important

- **Simulation**: URDF models are used in Gazebo and other simulators
- **Visualization**: RViz uses URDF to visualize robot models
- **Kinematics**: URDF defines the kinematic chain for forward and inverse kinematics
- **Control**: Controllers use URDF information for robot control
- **Collision Detection**: URDF provides collision geometry for planning

## URDF Structure

### Basic URDF File Structure

```xml
<?xml version="1.0"?>
<robot name="my_robot" xmlns:xacro="http://www.ros.org/wiki/xacro">
  <!-- Links -->
  <link name="base_link">
    <visual>
      <geometry>
        <cylinder length="0.6" radius="0.2"/>
      </geometry>
    </visual>
    <collision>
      <geometry>
        <cylinder length="0.6" radius="0.2"/>
      </geometry>
    </collision>
    <inertial>
      <mass value="10"/>
      <inertia ixx="1.0" ixy="0.0" ixz="0.0" iyy="1.0" iyz="0.0" izz="1.0"/>
    </inertial>
  </link>

  <!-- Joints -->
  <joint name="base_to_wheel" type="continuous">
    <parent link="base_link"/>
    <child link="wheel_link"/>
    <origin xyz="0 0.5 0" rpy="0 0 0"/>
  </joint>

  <link name="wheel_link">
    <visual>
      <geometry>
        <cylinder length="0.1" radius="0.3"/>
      </geometry>
    </visual>
  </link>
</robot>
```

## Links

A link represents a rigid body in the robot. Each link has visual, collision, and inertial properties.

### Link Components

1. **Visual**: Defines how the link looks in visualization
2. **Collision**: Defines collision geometry for physics simulation
3. **Inertial**: Defines mass and inertial properties for physics simulation

### Visual Properties

```xml
<visual>
  <origin xyz="0 0 0" rpy="0 0 0"/>
  <geometry>
    <box size="1 1 1"/>
    <!-- or -->
    <cylinder radius="0.5" length="1.0"/>
    <!-- or -->
    <sphere radius="0.5"/>
    <!-- or -->
    <mesh filename="package://my_robot/meshes/link.stl"/>
  </geometry>
  <material name="blue">
    <color rgba="0 0 1 1"/>
  </material>
</visual>
```

### Collision Properties

```xml
<collision>
  <origin xyz="0 0 0" rpy="0 0 0"/>
  <geometry>
    <box size="1 1 1"/>
    <!-- Use simpler geometry for collision than visual when possible -->
  </geometry>
</collision>
```

### Inertial Properties

```xml
<inertial>
  <origin xyz="0 0 0" rpy="0 0 0"/>
  <mass value="1.0"/>
  <inertia ixx="0.4" ixy="0.0" ixz="0.0" iyy="0.4" iyz="0.0" izz="0.4"/>
</inertial>
```

## Joints

Joints connect links and define how they can move relative to each other.

### Joint Types

1. **Fixed**: No movement (0 DOF)
2. **Revolute**: Rotational movement around an axis (1 DOF)
3. **Continuous**: Continuous rotational movement (1 DOF)
4. **Prismatic**: Linear movement along an axis (1 DOF)
5. **Planar**: Movement in a plane (2 DOF)
6. **Floating**: Movement in 3D space (3 DOF)

### Joint Definition

```xml
<joint name="joint_name" type="revolute">
  <parent link="parent_link_name"/>
  <child link="child_link_name"/>
  <origin xyz="0 0 0" rpy="0 0 0"/>
  <axis xyz="0 0 1"/>
  <limit lower="-1.57" upper="1.57" effort="100" velocity="1"/>
  <dynamics damping="0.1" friction="0.0"/>
</joint>
```

## Kinematics in URDF

### Forward Kinematics

URDF defines the kinematic chain from which forward kinematics can be computed. The relationship between joint angles and end-effector position is determined by the URDF structure.

### Denavit-Hartenberg Parameters

While not explicitly defined in URDF, the joint transformations effectively implement DH parameters.

## Sensors in URDF

Sensors can be defined in URDF to specify where they are mounted on the robot:

```xml
<link name="camera_link">
  <visual>
    <geometry>
      <box size="0.05 0.1 0.05"/>
    </geometry>
  </visual>
</link>

<joint name="camera_joint" type="fixed">
  <parent link="head_link"/>
  <child link="camera_link"/>
  <origin xyz="0.05 0 0.1" rpy="0 0 0"/>
</joint>

<gazebo reference="camera_link">
  <sensor type="camera" name="camera1">
    <visualize>true</visualize>
    <update_rate>30.0</update_rate>
    <camera name="head_camera">
      <horizontal_fov>1.3962634</horizontal_fov>
      <image>
        <width>800</width>
        <height>800</height>
        <format>R8G8B8</format>
      </image>
      <clip>
        <near>0.02</near>
        <far>300</far>
      </clip>
    </camera>
    <plugin name="camera_controller" filename="libgazebo_ros_camera.so">
      <frame_name>camera_link</frame_name>
    </plugin>
  </sensor>
</gazebo>
```

## Actuators in URDF

Actuators are typically not defined directly in URDF but are referenced through joint properties and control interfaces:

```xml
<joint name="motor_joint" type="revolute">
  <parent link="upper_link"/>
  <child link="lower_link"/>
  <limit effort="100" velocity="1"/>
  <dynamics damping="1" friction="1"/>
</joint>

<!-- Controller interface -->
<xacro:macro name="transmission_block" params="joint_name">
  <transmission name="tran1">
    <type>transmission_interface/SimpleTransmission</type>
    <joint name="${joint_name}">
      <hardwareInterface>hardware_interface/PositionJointInterface</hardwareInterface>
    </joint>
    <actuator name="motor1">
      <hardwareInterface>hardware_interface/PositionJointInterface</hardwareInterface>
      <mechanicalReduction>1</mechanicalReduction>
    </actuator>
  </transmission>
</xacro:macro>
```

## Humanoid Robot Modeling

### Humanoid Kinematic Structure

Humanoid robots typically have a kinematic structure similar to humans:

```
base_link (torso)
├── head_link
├── left_upper_arm
│   └── left_lower_arm
│       └── left_hand
├── right_upper_arm
│   └── right_lower_arm
│       └── right_hand
├── left_upper_leg
│   └── left_lower_leg
│       └── left_foot
└── right_upper_leg
    └── right_lower_leg
        └── right_foot
```

### Example Humanoid URDF

```xml
<?xml version="1.0"?>
<robot name="simple_humanoid" xmlns:xacro="http://www.ros.org/wiki/xacro">
  <!-- Torso -->
  <link name="torso">
    <visual>
      <geometry>
        <box size="0.3 0.2 0.5"/>
      </geometry>
    </visual>
    <collision>
      <geometry>
        <box size="0.3 0.2 0.5"/>
      </geometry>
    </collision>
    <inertial>
      <mass value="10"/>
      <inertia ixx="1.0" ixy="0.0" ixz="0.0" iyy="1.0" iyz="0.0" izz="1.0"/>
    </inertial>
  </link>

  <!-- Head -->
  <joint name="neck_joint" type="revolute">
    <parent link="torso"/>
    <child link="head"/>
    <origin xyz="0 0 0.3" rpy="0 0 0"/>
    <axis xyz="0 1 0"/>
    <limit lower="-0.5" upper="0.5" effort="10" velocity="1"/>
  </joint>

  <link name="head">
    <visual>
      <geometry>
        <sphere radius="0.1"/>
      </geometry>
    </visual>
    <collision>
      <geometry>
        <sphere radius="0.1"/>
      </geometry>
    </collision>
    <inertial>
      <mass value="2"/>
      <inertia ixx="0.04" ixy="0.0" ixz="0.0" iyy="0.04" iyz="0.0" izz="0.04"/>
    </inertial>
  </link>

  <!-- Left Arm -->
  <joint name="left_shoulder_joint" type="revolute">
    <parent link="torso"/>
    <child link="left_upper_arm"/>
    <origin xyz="0.2 0 0.1" rpy="0 0 0"/>
    <axis xyz="0 1 0"/>
    <limit lower="-1.57" upper="1.57" effort="10" velocity="1"/>
  </joint>

  <link name="left_upper_arm">
    <visual>
      <geometry>
        <cylinder length="0.3" radius="0.05"/>
      </geometry>
    </visual>
    <collision>
      <geometry>
        <cylinder length="0.3" radius="0.05"/>
      </geometry>
    </collision>
    <inertial>
      <mass value="1"/>
      <inertia ixx="0.01" ixy="0.0" ixz="0.0" iyy="0.01" iyz="0.0" izz="0.01"/>
    </inertial>
  </link>

  <joint name="left_elbow_joint" type="revolute">
    <parent link="left_upper_arm"/>
    <child link="left_lower_arm"/>
    <origin xyz="0 0 -0.3" rpy="0 0 0"/>
    <axis xyz="0 1 0"/>
    <limit lower="-1.57" upper="1.57" effort="10" velocity="1"/>
  </joint>

  <link name="left_lower_arm">
    <visual>
      <geometry>
        <cylinder length="0.3" radius="0.05"/>
      </geometry>
    </visual>
    <collision>
      <geometry>
        <cylinder length="0.3" radius="0.05"/>
      </geometry>
    </collision>
    <inertial>
      <mass value="0.5"/>
      <inertia ixx="0.005" ixy="0.0" ixz="0.0" iyy="0.005" iyz="0.0" izz="0.005"/>
    </inertial>
  </link>
</robot>
```

## Deployment Readiness Guidelines

### Best Practices

1. **Realistic Mass Properties**: Use accurate mass and inertial values
2. **Simplified Collision Geometry**: Use simpler shapes for collision than visual
3. **Consistent Units**: Use consistent units throughout (typically meters, kilograms)
4. **Proper Joint Limits**: Set realistic joint limits based on physical constraints
5. **Material Properties**: Include appropriate material definitions

### Validation Steps

1. **Check for Errors**: Use `check_urdf` command
   ```bash
   check_urdf /path/to/robot.urdf
   ```

2. **Visualize**: Load in RViz to check visual appearance
3. **Test Kinematics**: Verify forward and inverse kinematics work
4. **Collision Check**: Ensure no unintended collisions in default pose

## Tools for URDF Development

### Command Line Tools

- `check_urdf`: Validates URDF file syntax
- `urdf_to_graphiz`: Creates visual graph of robot structure
- `joint_state_publisher`: Publishes joint states for visualization

### Visualization

- **RViz**: Real-time visualization of robot model
- **Gazebo**: Physics simulation with URDF models
- **Blender**: For creating and editing mesh files

## Exercises

1. **Modeling Exercise**: Create a simple 2-link manipulator URDF model with proper kinematics.

2. **Integration Exercise**: Load your URDF model in RViz and verify the joint relationships work correctly.

3. **Design Exercise**: Design a simple humanoid model with at least 6 joints (torso, head, 2 arms) and include proper inertial properties.

## Summary

This chapter covered the fundamentals of URDF modeling for humanoid robots. We explored the structure of URDF files, including links, joints, and their properties. We looked at how to model humanoid kinematics, include sensors and actuators, and ensure deployment readiness. Understanding URDF is crucial for creating robots that can be properly simulated, visualized, and controlled in ROS-based systems.

## Further Reading

- [URDF Tutorials](http://wiki.ros.org/urdf/Tutorials)
- [URDF Specification](http://wiki.ros.org/urdf/XML)
- [Xacro Tutorials](http://wiki.ros.org/xacro)

---

**Previous Chapter**: [URDF & Python-ROS Integration](./ai-integration.md)