---
sidebar_position: 3
title: "Sensor Simulation"
---

# Sensor Simulation for Humanoid Robotics

## Learning Objectives

By the end of this chapter, you will be able to:
- Understand the importance of sensor simulation in humanoid robotics
- Implement realistic sensor models for cameras, IMUs, force/torque sensors, and LIDAR
- Configure sensor noise and accuracy parameters for realistic simulation
- Integrate simulated sensors with ROS 2 message formats
- Validate sensor simulation against real-world sensor data
- Apply sensor fusion techniques in simulation environments

## Introduction to Sensor Simulation

Sensor simulation is a critical component of humanoid robotics development, enabling developers to test perception algorithms, sensor fusion techniques, and control strategies before deploying to physical robots. In simulation, we can create ideal conditions, introduce controlled noise, and generate ground truth data that would be difficult or impossible to obtain with real sensors.

### Why Sensor Simulation Matters

- **Algorithm Development**: Test perception and control algorithms without physical hardware
- **Safety**: Validate dangerous behaviors in simulation first
- **Cost Reduction**: Reduce the need for expensive sensor hardware during development
- **Ground Truth Data**: Access perfect sensor readings for training AI models
- **Controlled Testing**: Create reproducible test scenarios with known conditions
- **Edge Case Testing**: Simulate rare or dangerous scenarios safely

## Types of Sensors in Humanoid Robots

Humanoid robots typically use multiple sensor types to perceive their environment and their own state:

### Vision Sensors

Vision sensors are crucial for humanoid robots to perceive their environment:

```xml
<!-- Example camera sensor configuration in URDF/SDF -->
<gazebo reference="camera_link">
  <sensor type="camera" name="head_camera">
    <update_rate>30.0</update_rate>
    <camera name="head_camera">
      <horizontal_fov>1.3962634</horizontal_fov>
      <image>
        <width>640</width>
        <height>480</height>
        <format>R8G8B8</format>
      </image>
      <clip>
        <near>0.02</near>
        <far>300</far>
      </clip>
      <noise>
        <type>gaussian</type>
        <mean>0.0</mean>
        <stddev>0.007</stddev>
      </noise>
    </camera>
    <plugin name="camera_controller" filename="libgazebo_ros_camera.so">
      <frame_name>camera_link</frame_name>
      <topic_name>image_raw</topic_name>
      <hack_baseline>0.07</hack_baseline>
      <distortion_k1>0.0</distortion_k1>
      <distortion_k2>0.0</distortion_k2>
      <distortion_k3>0.0</distortion_k3>
      <distortion_t1>0.0</distortion_t1>
      <distortion_t2>0.0</distortion_t2>
    </plugin>
  </sensor>
</gazebo>
```

### IMU Sensors

IMU (Inertial Measurement Unit) sensors provide orientation and acceleration data:

```xml
<!-- Example IMU sensor configuration -->
<gazebo reference="imu_link">
  <sensor type="imu" name="imu_sensor">
    <always_on>true</always_on>
    <update_rate>100</update_rate>
    <imu>
      <angular_velocity>
        <x>
          <noise type="gaussian">
            <mean>0.0</mean>
            <stddev>2e-4</stddev>
            <bias_mean>0.0000075</bias_mean>
            <bias_stddev>0.0000008</bias_stddev>
          </noise>
        </x>
        <y>
          <noise type="gaussian">
            <mean>0.0</mean>
            <stddev>2e-4</stddev>
            <bias_mean>0.0000075</bias_mean>
            <bias_stddev>0.0000008</bias_stddev>
          </noise>
        </y>
        <z>
          <noise type="gaussian">
            <mean>0.0</mean>
            <stddev>2e-4</stddev>
            <bias_mean>0.0000075</bias_mean>
            <bias_stddev>0.0000008</bias_stddev>
          </noise>
        </z>
      </angular_velocity>
      <linear_acceleration>
        <x>
          <noise type="gaussian">
            <mean>0.0</mean>
            <stddev>1.7e-2</stddev>
            <bias_mean>0.1</bias_mean>
            <bias_stddev>0.001</bias_stddev>
          </noise>
        </x>
        <y>
          <noise type="gaussian">
            <mean>0.0</mean>
            <stddev>1.7e-2</stddev>
            <bias_mean>0.1</bias_mean>
            <bias_stddev>0.001</bias_stddev>
          </noise>
        </y>
        <z>
          <noise type="gaussian">
            <mean>0.0</mean>
            <stddev>1.7e-2</stddev>
            <bias_mean>0.1</bias_mean>
            <bias_stddev>0.001</bias_stddev>
          </noise>
        </z>
      </linear_acceleration>
    </imu>
    <plugin name="imu_plugin" filename="libgazebo_ros_imu.so">
      <topicName>imu/data</topicName>
      <bodyName>imu_link</bodyName>
      <serviceName>imu/service</serviceName>
      <gaussianNoise>0.0</gaussianNoise>
      <updateRate>100.0</updateRate>
    </plugin>
  </sensor>
</gazebo>
```

### Force/Torque Sensors

Force/torque sensors are essential for humanoid robots to interact safely with their environment:

```xml
<!-- Example force/torque sensor configuration -->
<gazebo>
  <plugin name="ft_sensor" filename="libgazebo_ros_ft_sensor.so">
    <update_rate>100</update_rate>
    <topic_name>wrench</topic_name>
    <joint_name>left_foot_joint</joint_name>
  </plugin>
</gazebo>
```

### LIDAR Sensors

LIDAR sensors provide 3D environmental mapping:

```xml
<!-- Example LIDAR sensor configuration -->
<gazebo reference="lidar_link">
  <sensor type="ray" name="laser_sensor">
    <pose>0 0 0 0 0 0</pose>
    <visualize>true</visualize>
    <update_rate>10</update_rate>
    <ray>
      <scan>
        <horizontal>
          <samples>720</samples>
          <resolution>1</resolution>
          <min_angle>-1.570796</min_angle>
          <max_angle>1.570796</max_angle>
        </horizontal>
      </scan>
      <range>
        <min>0.10</min>
        <max>30.0</max>
        <resolution>0.01</resolution>
      </range>
      <noise>
        <type>gaussian</type>
        <mean>0.0</mean>
        <stddev>0.01</stddev>
      </noise>
    </ray>
    <plugin name="laser_plugin" filename="libgazebo_ros_laser.so">
      <topicName>scan</topicName>
      <frameName>lidar_link</frameName>
    </plugin>
  </sensor>
</gazebo>
```

## Sensor Noise Modeling

Realistic sensor noise is crucial for effective simulation:

### Camera Noise Models

```python
import numpy as np
import cv2

def add_camera_noise(image, noise_type='gaussian', noise_params=None):
    """
    Add realistic noise to camera images
    """
    if noise_type == 'gaussian':
        # Add Gaussian noise
        mean = noise_params.get('mean', 0)
        std = noise_params.get('std', 0.01)
        noise = np.random.normal(mean, std, image.shape).astype(np.float32)
        noisy_image = image.astype(np.float32) + noise
        return np.clip(noisy_image, 0, 255).astype(np.uint8)

    elif noise_type == 'poisson':
        # Add Poisson noise (photon noise)
        vals = len(np.unique(image))
        vals = 2 ** np.ceil(np.log2(vals))
        noisy_image = np.random.poisson(image * vals) / float(vals)
        return np.clip(noisy_image, 0, 255).astype(np.uint8)

    return image
```

### IMU Noise Models

IMU sensors have complex noise characteristics including bias, drift, and random walk:

```python
import numpy as np

class IMUNoiseModel:
    def __init__(self, gyro_noise_density=1.7e-4, gyro_random_walk=1.93e-5,
                 accel_noise_density=0.0017, accel_random_walk=2.44e-5):
        self.gyro_noise_density = gyro_noise_density
        self.gyro_random_walk = gyro_random_walk
        self.accel_noise_density = accel_noise_density
        self.accel_random_walk = accel_random_walk

        # Initialize bias states
        self.gyro_bias = np.zeros(3)
        self.accel_bias = np.zeros(3)

    def update_bias(self, dt):
        """Update bias states using random walk model"""
        # Update gyroscope bias
        self.gyro_bias += np.random.normal(0, self.gyro_random_walk, 3) * np.sqrt(dt)

        # Update accelerometer bias
        self.accel_bias += np.random.normal(0, self.accel_random_walk, 3) * np.sqrt(dt)

    def add_noise(self, true_angular_velocity, true_linear_acceleration, dt):
        """Add realistic noise to IMU measurements"""
        # Update bias
        self.update_bias(dt)

        # Add noise to gyroscope measurement
        gyro_noise = np.random.normal(0, self.gyro_noise_density, 3) / np.sqrt(dt)
        noisy_angular_velocity = true_angular_velocity + self.gyro_bias + gyro_noise

        # Add noise to accelerometer measurement
        accel_noise = np.random.normal(0, self.accel_noise_density, 3) / np.sqrt(dt)
        noisy_linear_acceleration = true_linear_acceleration + self.accel_bias + accel_noise

        return noisy_angular_velocity, noisy_linear_acceleration
```

## Sensor Fusion in Simulation

Sensor fusion combines data from multiple sensors to improve perception accuracy:

### Kalman Filter Example

```python
import numpy as np

class SensorFusionKF:
    def __init__(self):
        # State: [position, velocity]
        self.state_dim = 6  # 3 for position, 3 for velocity
        self.obs_dim = 6    # 3 from IMU (accel), 3 from position sensor

        # Initialize state vector [px, py, pz, vx, vy, vz]
        self.x = np.zeros(self.state_dim)

        # Initialize covariance matrix
        self.P = np.eye(self.state_dim) * 1000

        # Process noise
        self.Q = np.eye(self.state_dim) * 0.1

        # Measurement noise
        self.R = np.eye(self.obs_dim) * 0.5

        # Observation matrix
        self.H = np.zeros((self.obs_dim, self.state_dim))
        self.H[0:3, 0:3] = np.eye(3)  # Position measurements
        self.H[3:6, 3:6] = np.eye(3)  # Velocity measurements (from IMU integration)

    def predict(self, dt):
        """Prediction step"""
        # State transition matrix (constant velocity model)
        F = np.eye(self.state_dim)
        F[0:3, 3:6] = dt * np.eye(3)  # Position changes with velocity

        # Predict state
        self.x = F @ self.x

        # Predict covariance
        self.P = F @ self.P @ F.T + self.Q

    def update(self, z):
        """Update step with measurement z"""
        # Calculate innovation
        y = z - self.H @ self.x

        # Calculate innovation covariance
        S = self.H @ self.P @ self.H.T + self.R

        # Calculate Kalman gain
        K = self.P @ self.H.T @ np.linalg.inv(S)

        # Update state
        self.x = self.x + K @ y

        # Update covariance
        I = np.eye(self.state_dim)
        self.P = (I - K @ self.H) @ self.P
```

## ROS 2 Sensor Message Types

Understanding ROS 2 message types is essential for sensor simulation:

### Common Sensor Message Types

1. **sensor_msgs/Image**: Camera images
2. **sensor_msgs/Imu**: IMU data
3. **sensor_msgs/LaserScan**: LIDAR data
4. **geometry_msgs/WrenchStamped**: Force/torque data
5. **sensor_msgs/JointState**: Joint positions, velocities, efforts

### Example Sensor Publisher

```python
import rclpy
from rclpy.node import Node
from sensor_msgs.msg import Imu, JointState
from geometry_msgs.msg import WrenchStamped
import numpy as np

class SimulatedSensorPublisher(Node):
    def __init__(self):
        super().__init__('simulated_sensor_publisher')

        # Publishers for different sensor types
        self.imu_publisher = self.create_publisher(Imu, 'imu/data', 10)
        self.joint_state_publisher = self.create_publisher(JointState, 'joint_states', 10)
        self.wrench_publisher = self.create_publisher(WrenchStamped, 'ft_sensor', 10)

        # Timer for sensor updates
        self.timer = self.create_timer(0.01, self.publish_sensor_data)  # 100 Hz

        # Initialize IMU noise model
        self.imu_noise_model = IMUNoiseModel()

    def publish_sensor_data(self):
        # Publish IMU data
        imu_msg = self.create_imu_message()
        self.imu_publisher.publish(imu_msg)

        # Publish joint states
        joint_msg = self.create_joint_state_message()
        self.joint_state_publisher.publish(joint_msg)

        # Publish force/torque data
        wrench_msg = self.create_wrench_message()
        self.wrench_publisher.publish(wrench_msg)

    def create_imu_message(self):
        msg = Imu()

        # Simulate IMU readings with noise
        true_angular_vel = np.array([0.1, 0.05, 0.02])  # Example values
        true_linear_acc = np.array([0.0, 0.0, 9.81])    # Gravity

        noisy_angular_vel, noisy_linear_acc = self.imu_noise_model.add_noise(
            true_angular_vel, true_linear_acc, 0.01)  # dt = 0.01s

        msg.angular_velocity.x = noisy_angular_vel[0]
        msg.angular_velocity.y = noisy_angular_vel[1]
        msg.angular_velocity.z = noisy_angular_vel[2]

        msg.linear_acceleration.x = noisy_linear_acc[0]
        msg.linear_acceleration.y = noisy_linear_acc[1]
        msg.linear_acceleration.z = noisy_linear_acc[2]

        return msg

    def create_joint_state_message(self):
        msg = JointState()
        msg.name = ['joint1', 'joint2', 'joint3']
        msg.position = [0.1, 0.2, 0.3]  # Example positions
        msg.velocity = [0.01, 0.02, 0.03]  # Example velocities
        msg.effort = [0.5, 0.6, 0.7]  # Example efforts

        return msg

    def create_wrench_message(self):
        msg = WrenchStamped()
        msg.wrench.force.x = 10.0  # Example force values
        msg.wrench.force.y = 5.0
        msg.wrench.force.z = 15.0
        msg.wrench.torque.x = 1.0  # Example torque values
        msg.wrench.torque.y = 0.5
        msg.wrench.torque.z = 2.0

        return msg

def main(args=None):
    rclpy.init(args=args)
    sensor_publisher = SimulatedSensorPublisher()

    try:
        rclpy.spin(sensor_publisher)
    except KeyboardInterrupt:
        pass
    finally:
        sensor_publisher.destroy_node()
        rclpy.shutdown()

if __name__ == '__main__':
    main()
```

## Validation and Calibration

### Sensor Validation Techniques

1. **Ground Truth Comparison**: Compare simulated sensor output with known values
2. **Statistical Analysis**: Verify noise characteristics match expected models
3. **Cross-Sensor Validation**: Check consistency between different sensor types
4. **Real vs. Sim Comparison**: Compare with real sensor data when available

### Example Validation Script

```python
import numpy as np
import matplotlib.pyplot as plt

def validate_imu_noise(data, expected_std, sample_rate=100):
    """
    Validate IMU noise characteristics
    """
    # Calculate actual standard deviation
    actual_std = np.std(data)

    # Calculate bias (mean offset)
    bias = np.mean(data)

    # Perform statistical tests
    print(f"Expected std: {expected_std}")
    print(f"Actual std: {actual_std}")
    print(f"Bias: {bias}")

    # Check if within acceptable range (e.g., 10% tolerance)
    tolerance = 0.1 * expected_std
    if abs(actual_std - expected_std) < tolerance:
        print("✓ Noise level is within acceptable range")
    else:
        print("✗ Noise level is outside acceptable range")

    # Plot histogram to check normality
    plt.figure(figsize=(10, 6))
    plt.hist(data, bins=50, density=True, alpha=0.7)
    plt.title('IMU Noise Distribution')
    plt.xlabel('Noise Value')
    plt.ylabel('Probability Density')
    plt.grid(True)
    plt.show()
```

## Advanced Sensor Simulation Techniques

### Dynamic Sensor Simulation

For humanoid robots, sensors may have dynamic properties that change during operation:

```python
class DynamicSensorSimulator:
    def __init__(self):
        self.temperature = 25.0  # degrees Celsius
        self.age = 0.0  # in simulation hours

    def update_sensor_characteristics(self, dt):
        """Update sensor characteristics based on environmental factors"""
        # Temperature drift
        self.temperature += np.random.normal(0, 0.1) * dt

        # Aging effects
        self.age += dt / 3600.0  # Convert to hours

        # Update noise parameters based on temperature and age
        temp_coefficient = 0.001  # per degree C
        age_coefficient = 0.0001  # per hour

        temp_effect = temp_coefficient * abs(self.temperature - 25.0)
        age_effect = age_coefficient * self.age

        # Return updated noise parameters
        return {
            'noise_multiplier': 1.0 + temp_effect + age_effect,
            'bias_drift': temp_effect * 0.001
        }
```

### Multi-Sensor Coordination

Humanoid robots often have coordinated sensor systems:

```python
class MultiSensorCoordinator:
    def __init__(self):
        self.sensors = {
            'camera': {'rate': 30, 'last_update': 0},
            'lidar': {'rate': 10, 'last_update': 0},
            'imu': {'rate': 100, 'last_update': 0},
            'ft_sensor': {'rate': 200, 'last_update': 0}
        }

    def should_update(self, sensor_name, current_time):
        """Check if sensor should update based on its rate"""
        sensor = self.sensors[sensor_name]
        time_since_last = current_time - sensor['last_update']
        update_interval = 1.0 / sensor['rate']

        return time_since_last >= update_interval

    def update_sensor(self, sensor_name, current_time):
        """Update sensor and record time"""
        if self.should_update(sensor_name, current_time):
            self.sensors[sensor_name]['last_update'] = current_time
            return True
        return False
```

## Best Practices for Sensor Simulation

### Realism vs. Performance Trade-offs

1. **Fidelity Requirements**: Match simulation fidelity to application needs
2. **Computational Cost**: Balance accuracy with real-time performance
3. **Validation**: Regularly validate simulation against real-world data
4. **Modularity**: Design sensors to be easily configurable and replaceable

### Sensor Simulation Guidelines

1. **Start Simple**: Begin with basic sensor models, add complexity as needed
2. **Validate Early**: Test sensor models against known benchmarks
3. **Document Assumptions**: Clearly document sensor model limitations
4. **Modular Design**: Keep sensor models separate and reusable
5. **Configurable Parameters**: Allow easy adjustment of noise and accuracy parameters

## Troubleshooting Sensor Simulation

### Common Issues and Solutions

1. **Drifting Sensors**: Implement proper bias modeling and correction
2. **Excessive Noise**: Verify noise parameters match real sensor specifications
3. **Timing Issues**: Ensure proper synchronization between sensor updates
4. **Coordinate Frame Errors**: Verify all sensors use correct reference frames

### Debugging Tools

```python
class SensorDebugger:
    def __init__(self):
        self.data_buffer = {}
        self.stats = {}

    def log_sensor_data(self, sensor_name, data):
        """Log sensor data for debugging"""
        if sensor_name not in self.data_buffer:
            self.data_buffer[sensor_name] = []
            self.stats[sensor_name] = {'count': 0, 'sum': 0, 'sum_sq': 0}

        self.data_buffer[sensor_name].append(data)
        self.update_stats(sensor_name, data)

    def update_stats(self, sensor_name, data):
        """Update running statistics"""
        stats = self.stats[sensor_name]
        stats['count'] += 1
        stats['sum'] += data
        stats['sum_sq'] += data * data

    def get_statistics(self, sensor_name):
        """Get current statistics for sensor"""
        stats = self.stats[sensor_name]
        if stats['count'] == 0:
            return None

        mean = stats['sum'] / stats['count']
        variance = (stats['sum_sq'] / stats['count']) - (mean * mean)
        std_dev = np.sqrt(variance)

        return {
            'mean': mean,
            'std_dev': std_dev,
            'count': stats['count']
        }
```

## Exercises

1. **Sensor Implementation Exercise**: Create a simulated camera sensor node that publishes realistic camera data with appropriate noise models.

2. **Fusion Exercise**: Implement a simple sensor fusion algorithm that combines IMU and position sensor data using a Kalman filter.

3. **Validation Exercise**: Develop a validation script that compares simulated sensor data with theoretical models and reports any discrepancies.

## Summary

This chapter covered sensor simulation for humanoid robotics, including various sensor types, noise modeling, sensor fusion techniques, and validation approaches. Realistic sensor simulation is crucial for developing robust perception and control algorithms that can transfer from simulation to real-world humanoid robots. Proper modeling of sensor characteristics, including noise and bias, is essential for effective simulation-based development.

## Further Reading

- [ROS 2 Sensor Message Types](https://docs.ros.org/en/humble/Concepts/About-ROS-Interfaces.html)
- [Gazebo Sensor Documentation](http://gazebosim.org/tutorials?tut=ros_gzplugins#Sensor-plugins)
- [Sensor Fusion Techniques](https://en.wikipedia.org/wiki/Sensor_fusion)
- [Kalman Filter Tutorials](https://www.bzarg.com/p/how-a-kalman-filter-works-in-pictures/)

---

**Previous Chapter**: [Digital Twins & Unity](./unity-digital-twins.md)
**Next Chapter**: [Isaac Sim & Synthetic Data](../ai-robot-brain/isaac-sim.md)