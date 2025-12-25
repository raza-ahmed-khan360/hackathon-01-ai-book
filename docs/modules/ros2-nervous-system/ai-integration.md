---
sidebar_position: 3
title: "URDF & Python-ROS Integration"
---

# URDF & Python-ROS Integration

## Learning Objectives

By the end of this chapter, you will be able to:
- Use rclpy to connect Python-based AI agents to ROS 2 systems
- Implement communication patterns for AI-to-robot integration
- Bridge decision logic from AI agents to robot control systems
- Create service clients and action clients for AI-ROS integration
- Apply troubleshooting techniques for Python-ROS integration

## Introduction to Python-ROS Integration

Python is one of the most popular languages for AI development, making it a natural choice for connecting AI agents to ROS 2 systems. The `rclpy` library provides the Python client library for ROS 2, enabling Python-based AI agents to communicate with the ROS 2 ecosystem.

### Why Python for AI Integration?

- Rich ecosystem for machine learning and AI (TensorFlow, PyTorch, scikit-learn)
- Simpler prototyping and development
- Extensive libraries for data processing and analysis
- Strong community support for AI research

## Setting up rclpy

To use rclpy in your Python projects, you'll need to install the appropriate ROS 2 packages:

```bash
# Make sure ROS 2 is sourced
source /opt/ros/humble/setup.bash  # or your ROS 2 distribution

# In your Python environment
import rclpy
from rclpy.node import Node
```

## Connecting AI Agents to ROS 2

### Basic Node Structure

Here's the basic structure for an AI agent node:

```python
import rclpy
from rclpy.node import Node
import numpy as np  # Example: for AI processing

class AIAgentNode(Node):
    def __init__(self):
        super().__init__('ai_agent_node')

        # Initialize AI components
        self.initialize_ai_components()

        # Set up ROS 2 communication
        self.setup_ros_communication()

    def initialize_ai_components(self):
        # Initialize your AI models, algorithms, etc.
        self.get_logger().info('AI components initialized')

    def setup_ros_communication(self):
        # Set up publishers, subscribers, services, etc.
        pass

def main(args=None):
    rclpy.init(args=args)
    ai_agent = AIAgentNode()

    try:
        rclpy.spin(ai_agent)
    except KeyboardInterrupt:
        pass
    finally:
        ai_agent.destroy_node()
        rclpy.shutdown()

if __name__ == '__main__':
    main()
```

## Communication Patterns for AI Integration

### 1. Sensor Data Processing

AI agents often need to process sensor data to make decisions:

```python
from sensor_msgs.msg import LaserScan, Image
from std_msgs.msg import String

class PerceptionNode(Node):
    def __init__(self):
        super().__init__('perception_node')

        # Subscribe to sensor data
        self.scan_subscription = self.create_subscription(
            LaserScan,
            'scan',
            self.scan_callback,
            10)

        self.image_subscription = self.create_subscription(
            Image,
            'camera/image_raw',
            self.image_callback,
            10)

        # Publish processed data or decisions
        self.decision_publisher = self.create_publisher(
            String,
            'ai_decisions',
            10)

    def scan_callback(self, msg):
        # Process laser scan data with AI
        processed_data = self.ai_process_scan(msg.ranges)

        # Publish decision
        decision_msg = String()
        decision_msg.data = f"Obstacle detected at {min(msg.ranges)}m"
        self.decision_publisher.publish(decision_msg)

    def image_callback(self, msg):
        # Process image data with AI
        decision = self.ai_process_image(msg)
        # Publish decision
        pass

    def ai_process_scan(self, ranges):
        # Your AI processing logic here
        return "processed_data"

    def ai_process_image(self, image_msg):
        # Your AI processing logic here
        return "decision"
```

### 2. Control Command Execution

AI agents need to send control commands to the robot:

```python
from geometry_msgs.msg import Twist
from std_msgs.msg import Bool

class ControlNode(Node):
    def __init__(self):
        super().__init__('control_node')

        # Publisher for robot commands
        self.cmd_vel_publisher = self.create_publisher(
            Twist,
            'cmd_vel',
            10)

        # Subscriber for AI decisions
        self.decision_subscription = self.create_subscription(
            String,
            'ai_decisions',
            self.decision_callback,
            10)

    def decision_callback(self, msg):
        # Convert AI decision to robot command
        cmd = self.decision_to_command(msg.data)
        self.cmd_vel_publisher.publish(cmd)

    def decision_to_command(self, decision_str):
        cmd = Twist()
        if "forward" in decision_str:
            cmd.linear.x = 0.5
        elif "turn" in decision_str:
            cmd.angular.z = 0.5
        elif "stop" in decision_str:
            cmd.linear.x = 0.0
            cmd.angular.z = 0.0
        return cmd
```

## Service Clients for AI-ROS Integration

AI agents often need to request specific services from the ROS system:

```python
from example_interfaces.srv import Trigger
from std_msgs.msg import String

class AIAgentWithServices(Node):
    def __init__(self):
        super().__init__('ai_agent_with_services')

        # Create service client
        self.service_client = self.create_client(
            Trigger,
            'robot_reset')

        # Wait for service to be available
        while not self.service_client.wait_for_service(timeout_sec=1.0):
            self.get_logger().info('Service not available, waiting again...')

        # Example: Call service from AI logic
        self.call_reset_service()

    def call_reset_service(self):
        request = Trigger.Request()
        future = self.service_client.call_async(request)
        future.add_done_callback(self.service_response_callback)

    def service_response_callback(self, future):
        try:
            response = future.result()
            if response.success:
                self.get_logger().info('Service call successful')
            else:
                self.get_logger().info(f'Service call failed: {response.message}')
        except Exception as e:
            self.get_logger().error(f'Service call failed: {e}')
```

## Action Clients for Complex Tasks

For long-running tasks that require feedback, use action clients:

```python
import rclpy
from rclpy.action import ActionClient
from rclpy.node import Node
from nav2_msgs.action import NavigateToPose
from geometry_msgs.msg import PoseStamped

class NavigationAI(Node):
    def __init__(self):
        super().__init__('navigation_ai')

        # Create action client for navigation
        self._action_client = ActionClient(
            self,
            NavigateToPose,
            'navigate_to_pose')

    def send_goal(self, x, y, theta):
        # Wait for action server
        self._action_client.wait_for_server()

        # Create goal
        goal_msg = NavigateToPose.Goal()
        goal_msg.pose.header.frame_id = 'map'
        goal_msg.pose.pose.position.x = x
        goal_msg.pose.pose.position.y = y
        goal_msg.pose.pose.orientation.z = theta

        # Send goal
        self._send_goal_future = self._action_client.send_goal_async(
            goal_msg,
            feedback_callback=self.feedback_callback)

        self._send_goal_future.add_done_callback(self.goal_response_callback)

    def goal_response_callback(self, future):
        goal_handle = future.result()
        if not goal_handle.accepted:
            self.get_logger().info('Goal rejected')
            return

        self.get_logger().info('Goal accepted')
        self._get_result_future = goal_handle.get_result_async()
        self._get_result_future.add_done_callback(self.get_result_callback)

    def feedback_callback(self, feedback_msg):
        self.get_logger().info(f'Navigation feedback: {feedback_msg.feedback}')

    def get_result_callback(self, future):
        result = future.result().result
        self.get_logger().info(f'Navigation result: {result}')
```

## Connecting Decision Logic to Robot Control

### Example: AI-Based Obstacle Avoidance

Here's a complete example that connects AI decision-making to robot control:

```python
import rclpy
from rclpy.node import Node
from sensor_msgs.msg import LaserScan
from geometry_msgs.msg import Twist
import numpy as np

class AIObstacleAvoidance(Node):
    def __init__(self):
        super().__init__('ai_obstacle_avoidance')

        # Publisher for velocity commands
        self.cmd_vel_publisher = self.create_publisher(Twist, 'cmd_vel', 10)

        # Subscriber for laser scan data
        self.scan_subscription = self.create_subscription(
            LaserScan,
            'scan',
            self.scan_callback,
            10)

        # AI parameters
        self.safe_distance = 1.0  # meters
        self.linear_speed = 0.5   # m/s
        self.angular_speed = 0.5  # rad/s

        self.get_logger().info('AI Obstacle Avoidance Node Started')

    def scan_callback(self, msg):
        # Process laser scan with AI logic
        cmd = self.ai_decision_process(msg.ranges)

        # Publish command
        self.cmd_vel_publisher.publish(cmd)

    def ai_decision_process(self, ranges):
        cmd = Twist()

        # Convert ranges to numpy array for easier processing
        ranges_array = np.array(ranges)
        ranges_array = ranges_array[np.isfinite(ranges_array)]  # Remove inf values

        if len(ranges_array) == 0:
            # No valid readings, stop
            return cmd

        # Find minimum distance
        min_distance = np.min(ranges_array)

        # AI decision logic
        if min_distance < self.safe_distance:
            # Too close to obstacle, turn
            cmd.angular.z = self.angular_speed
            self.get_logger().info(f'Obstacle detected! Distance: {min_distance:.2f}m, turning')
        else:
            # Safe to move forward
            cmd.linear.x = self.linear_speed
            self.get_logger().info(f'Moving forward. Distance to nearest: {min_distance:.2f}m')

        return cmd

def main(args=None):
    rclpy.init(args=args)
    ai_node = AIObstacleAvoidance()

    try:
        rclpy.spin(ai_node)
    except KeyboardInterrupt:
        pass
    finally:
        ai_node.destroy_node()
        rclpy.shutdown()

if __name__ == '__main__':
    main()
```

## Troubleshooting Python-ROS Integration

### Common Issues and Solutions

1. **Import Errors**: Make sure ROS 2 environment is sourced and rclpy is installed
   ```bash
   source /opt/ros/humble/setup.bash
   ```

2. **Node Registration**: Ensure each node has a unique name in the ROS graph
   ```python
   node = rclpy.create_node('unique_node_name')
   ```

3. **Threading Issues**: Use appropriate threading for AI processing
   ```python
   import threading
   from rclpy.executors import MultiThreadedExecutor

   executor = MultiThreadedExecutor()
   executor.add_node(node)
   ```

4. **Memory Management**: Properly destroy nodes to prevent memory leaks
   ```python
   node.destroy_node()
   ```

## Exercises

1. **Implementation Exercise**: Create an AI agent that processes camera images to detect colors and publishes navigation commands based on detected objects.

2. **Integration Exercise**: Modify the obstacle avoidance example to use a machine learning model for more sophisticated decision making.

3. **Service Integration**: Create a service that allows external nodes to query the AI agent's current state and confidence level.

## Summary

This chapter covered how to integrate Python-based AI agents with ROS 2 systems using rclpy. We explored different communication patterns including topics for sensor data and control commands, services for specific requests, and actions for complex tasks. We also looked at practical examples of connecting AI decision logic to robot control systems.

---

**Previous Chapter**: [Nodes, Topics, Services](./nodes-topics-services.md)
**Next Chapter**: [Humanoid Modeling with URDF](./urdf-modeling.md)