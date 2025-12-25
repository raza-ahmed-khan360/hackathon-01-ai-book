---
sidebar_position: 1
title: "Physics Simulation in Gazebo"
---

# Physics Simulation in Gazebo

## Learning Objectives

By the end of this chapter, you will be able to:
- Understand the role of Gazebo as a physics simulation environment for humanoid robotics
- Configure Gazebo for realistic physics simulation of humanoid robots
- Integrate Gazebo with ROS 2 for sensor simulation and control
- Set up proper physics parameters for humanoid robot simulation
- Validate simulation results against real-world robot behavior

## Introduction to Gazebo for Humanoid Robotics

Gazebo is a 3D simulation environment that provides realistic physics simulation, high-quality graphics, and convenient programmatic interfaces. For humanoid robotics, Gazebo serves as a crucial component in the development pipeline, allowing developers to test algorithms, validate control strategies, and perform experiments in a safe, repeatable environment before deploying to real robots.

### Why Gazebo for Humanoid Robotics?

- **Physics Accuracy**: Realistic simulation of joint dynamics, contact forces, and environmental interactions
- **Sensor Simulation**: Accurate modeling of cameras, IMUs, force/torque sensors, and other robot sensors
- **Environment Modeling**: Creation of complex environments with realistic lighting and physics
- **Cost-Effective**: Reduces the need for expensive hardware testing and prototyping
- **Safety**: Allows testing of risky behaviors without physical robot damage
- **Repeatability**: Enables controlled experiments with consistent conditions

## Gazebo Architecture and Components

### Core Components

1. **Gazebo Server**: Runs the physics simulation and handles all simulation state
2. **Gazebo Client**: Provides the graphical user interface for visualization
3. **Physics Engine**: Underlying engine that performs physics calculations (ODE, Bullet, Simbody)
4. **Sensor Models**: Simulates various robot sensors with realistic noise models
5. **Plugins**: Extensible architecture for custom simulation behaviors

### Physics Engines in Gazebo

Gazebo supports multiple physics engines, each with different characteristics:

- **ODE (Open Dynamics Engine)**: Default engine, good balance of speed and accuracy
- **Bullet**: Good for contact-rich scenarios and complex collisions
- **Simbody**: High-fidelity simulation for complex articulated systems

## Setting Up Gazebo for Humanoid Robots

### Installation and Dependencies

```bash
# Install Gazebo (example for Ubuntu with ROS 2 Humble)
sudo apt update
sudo apt install ros-humble-gazebo-ros-pkgs
sudo apt install ros-humble-gazebo-ros-control
sudo apt install ros-humble-gazebo-ros2-control
```

### Basic Gazebo Launch Configuration

```xml
<!-- Example launch file for Gazebo simulation -->
<launch>
  <!-- Start Gazebo server -->
  <include file="$(find gazebo_ros)/launch/empty_world.launch.py">
    <arg name="world" value="$(find my_robot_description)/worlds/my_world.world"/>
    <arg name="paused" value="false"/>
    <arg name="use_sim_time" value="true"/>
  </include>

  <!-- Spawn robot model -->
  <node name="spawn_urdf" pkg="gazebo_ros" type="spawn_entity.py"
        args="-entity my_robot -topic robot_description -x 0 -y 0 -z 1.0"/>
</launch>
```

## Physics Configuration for Humanoid Robots

### Gravity and World Settings

For humanoid robots, proper gravity settings are crucial for realistic simulation:

```xml
<!-- World file configuration -->
<sdf version="1.7">
  <world name="default">
    <physics type="ode">
      <max_step_size>0.001</max_step_size>
      <real_time_factor>1.0</real_time_factor>
      <real_time_update_rate>1000.0</real_time_update_rate>
      <gravity>0 0 -9.8</gravity>
    </physics>

    <include>
      <uri>model://ground_plane</uri>
    </include>

    <include>
      <uri>model://sun</uri>
    </include>
  </world>
</sdf>
```

### Joint Dynamics Configuration

Proper joint dynamics are essential for humanoid robot simulation:

```xml
<!-- Joint configuration for realistic humanoid simulation -->
<joint name="hip_joint" type="revolute">
  <parent>torso</parent>
  <child>thigh</child>
  <axis>
    <xyz>0 0 1</xyz>
    <limit effort="100" velocity="1.0" lower="-1.57" upper="1.57"/>
    <dynamics damping="1.0" friction="0.1"/>
  </axis>
</joint>
```

### Contact and Friction Models

For humanoid robots that interact with the environment, contact modeling is critical:

```xml
<!-- Link with proper collision properties -->
<link name="foot">
  <collision>
    <geometry>
      <box size="0.15 0.1 0.05"/>
    </geometry>
    <surface>
      <friction>
        <ode>
          <mu>1.0</mu>
          <mu2>1.0</mu2>
          <fdir1>0 0 1</fdir1>
          <slip1>0.0</slip1>
          <slip2>0.0</slip2>
        </ode>
      </friction>
      <bounce>
        <restitution_coefficient>0.01</restitution_coefficient>
        <threshold>100000</threshold>
      </bounce>
      <contact>
        <ode>
          <soft_cfm>0</soft_cfm>
          <soft_erp>0.2</soft_erp>
          <kp>1e+12</kp>
          <kd>1e+09</kd>
          <max_vel>100.0</max_vel>
          <min_depth>0.001</min_depth>
        </ode>
      </contact>
    </surface>
  </collision>
</link>
```

## Gazebo-ROS 2 Integration

### Gazebo Plugins for ROS 2

Gazebo integrates with ROS 2 through plugins that enable communication between the simulation and ROS 2 nodes:

```xml
<!-- Example of ROS 2 control plugin in URDF -->
<gazebo>
  <plugin name="gazebo_ros_control" filename="libgazebo_ros_control.so">
    <robotNamespace>/my_robot</robotNamespace>
    <robotSimType>gazebo_ros_control/DefaultRobotHWSim</robotSimType>
  </plugin>
</gazebo>
```

### Sensor Integration

Sensors in Gazebo can be configured to publish directly to ROS 2 topics:

```xml
<!-- Camera sensor configuration -->
<gazebo reference="camera_link">
  <sensor type="camera" name="camera1">
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
      <topic_name>image_raw</topic_name>
    </plugin>
  </sensor>
</gazebo>
```

## Control Integration with ROS 2

### Joint State Publisher

```yaml
# Controller configuration for Gazebo simulation
controller_manager:
  ros__parameters:
    update_rate: 100
    use_sim_time: true

joint_state_broadcaster:
  type: joint_state_broadcaster/JointStateBroadcaster

position_controller:
  type: position_controllers/JointGroupPositionController
  joints:
    - joint1
    - joint2
    - joint3
```

### Control Strategies for Humanoid Robots

Humanoid robots require sophisticated control strategies in simulation:

1. **PD Controllers**: For basic joint position control
2. **Impedance Control**: For compliant behavior and safe interaction
3. **Whole-Body Control**: For coordinated multi-joint movements
4. **Balance Control**: For maintaining stability during locomotion

## Simulation Best Practices

### Model Validation

Before using a model in simulation, validate it:

```bash
# Check URDF for errors
check_urdf /path/to/robot.urdf

# Visualize in RViz
ros2 run rviz2 rviz2

# Test in Gazebo
gazebo --verbose /path/to/world.world
```

### Performance Optimization

For complex humanoid robot simulations:

1. **Reduce Physics Update Rate**: Balance accuracy with performance
2. **Simplify Collision Models**: Use simpler shapes for collision than visual
3. **Limit Sensor Updates**: Adjust sensor update rates based on requirements
4. **Use Appropriate Damping**: Prevent simulation instability

### Realism vs. Performance Trade-offs

- **High Fidelity**: More accurate but slower simulation
- **Fast Simulation**: Suitable for algorithm development but less realistic
- **Mixed Approach**: Use high fidelity for critical components, simplified for others

## Troubleshooting Common Issues

### Simulation Instability

- **Symptoms**: Robot joints oscillating wildly, parts flying apart
- **Solutions**:
  - Reduce physics step size
  - Increase damping values
  - Check mass and inertia properties

### Sensor Noise and Accuracy

- **Symptoms**: Sensor readings too clean or too noisy
- **Solutions**:
  - Adjust sensor noise parameters
  - Verify sensor placement and orientation

### Control Issues

- **Symptoms**: Robot not responding to commands as expected
- **Solutions**:
  - Verify controller configuration
  - Check joint limits and dynamics
  - Ensure proper timing between nodes

## Exercises

1. **Simulation Setup Exercise**: Create a simple humanoid model in Gazebo and configure basic physics properties. Test the simulation stability with different physics parameters.

2. **Sensor Integration Exercise**: Add camera and IMU sensors to your humanoid model and verify that they publish data to ROS 2 topics.

3. **Control Integration Exercise**: Implement a simple joint position controller and test it in simulation.

## Summary

This chapter covered the fundamentals of physics simulation in Gazebo for humanoid robotics. We explored the architecture of Gazebo, how to configure physics parameters for realistic humanoid simulation, and how to integrate Gazebo with ROS 2. Proper simulation setup is crucial for developing and testing humanoid robot algorithms before deployment to real hardware.

## Further Reading

- [Gazebo Tutorials](http://gazebosim.org/tutorials)
- [ROS 2 with Gazebo](https://github.com/ros-simulation/gazebo_ros_pkgs)
- [Physics Simulation Best Practices](https://classic.gazebosim.org/tutorials?tut=physics)

---

**Next Chapter**: [Digital Twins & Unity](./unity-digital-twins.md)