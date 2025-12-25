---
sidebar_position: 2
title: "Nodes, Topics, Services"
---

# Nodes, Topics, Services in ROS 2

## Learning Objectives

By the end of this chapter, you will be able to:
- Implement nodes that publish and subscribe to topics
- Create and use services for request-response communication
- Understand Quality of Service (QoS) settings and their impact
- Design appropriate communication patterns for robot systems

## Nodes in Detail

A node is an executable that uses ROS 2 client libraries to communicate with other nodes. Nodes can publish or subscribe to Topics, provide or use Services, and send or receive Actions.

### Creating a Node

In Python, using rclpy (the Python ROS 2 client library), you create a node by inheriting from the Node class:

```python
import rclpy
from rclpy.node import Node

class MyNode(Node):
    def __init__(self):
        super().__init__('my_node_name')
        # Node initialization code here
```

### Node Parameters

Nodes can accept parameters that can be configured at runtime:

```python
self.declare_parameter('param_name', 'default_value')
param_value = self.get_parameter('param_name').value
```

## Topics and Publishers/Subscribers

Topics provide a way for nodes to send and receive data asynchronously. The communication follows a publish/subscribe model.

### Publishers

A publisher sends messages to a topic:

```python
from std_msgs.msg import String

def __init__(self):
    super().__init__('publisher_node')
    self.publisher = self.create_publisher(String, 'topic_name', 10)
    self.timer = self.create_timer(0.5, self.timer_callback)

def timer_callback(self):
    msg = String()
    msg.data = 'Hello World'
    self.publisher.publish(msg)
```

### Subscribers

A subscriber receives messages from a topic:

```python
def __init__(self):
    super().__init__('subscriber_node')
    self.subscription = self.create_subscription(
        String,
        'topic_name',
        self.listener_callback,
        10)
    self.subscription  # prevent unused variable warning

def listener_callback(self, msg):
    self.get_logger().info('I heard: "%s"' % msg.data)
```

## Services

Services provide synchronous request-response communication between nodes.

### Service Servers

A service server provides a service:

```python
from example_interfaces.srv import AddTwoInts

def __init__(self):
    super().__init__('service_server')
    self.srv = self.create_service(AddTwoInts, 'add_two_ints', self.add_two_ints_callback)

def add_two_ints_callback(self, request, response):
    response.sum = request.a + request.b
    self.get_logger().info('Incoming request\na: %d b: %d' % (request.a, request.b))
    return response
```

### Service Clients

A service client uses a service:

```python
def __init__(self):
    super().__init__('service_client')
    self.cli = self.create_client(AddTwoInts, 'add_two_ints')
    while not self.cli.wait_for_service(timeout_sec=1.0):
        self.get_logger().info('Service not available, waiting again...')
    self.req = AddTwoInts.Request()

def send_request(self, a, b):
    self.req.a = a
    self.req.b = b
    self.future = self.cli.call_async(self.req)
```

## Quality of Service (QoS)

QoS settings allow you to configure how messages are delivered based on your application's requirements:

```python
from rclpy.qos import QoSProfile, ReliabilityPolicy, DurabilityPolicy

# Configure QoS for reliable delivery
qos_profile = QoSProfile(
    depth=10,
    reliability=ReliabilityPolicy.RELIABLE,
    durability=DurabilityPolicy.VOLATILE
)
```

## Communication Patterns

### Publisher-Subscriber (Topics)
- Use when you need one-to-many communication
- Good for sensor data, status updates
- Asynchronous communication
- Decouples publisher from subscribers

### Service-Client (Services)
- Use when you need synchronous request-response
- Good for actions that return a result immediately
- Blocking call until response is received

### Action-Client-Server (Actions)
- Use for long-running tasks with feedback
- Good for navigation, manipulation tasks
- Supports goal preemption

## Practical Examples

### Sensor Node Example

A sensor node that publishes data to a topic:

```python
import rclpy
from rclpy.node import Node
from sensor_msgs.msg import LaserScan

class SensorNode(Node):
    def __init__(self):
        super().__init__('sensor_node')
        self.publisher = self.create_publisher(LaserScan, 'scan', 10)

        # Simulate sensor data
        self.timer = self.create_timer(0.1, self.publish_scan_data)

    def publish_scan_data(self):
        msg = LaserScan()
        # Fill in laser scan data
        msg.ranges = [1.0, 1.5, 2.0, 2.5]  # Example ranges
        self.publisher.publish(msg)

def main(args=None):
    rclpy.init(args=args)
    sensor_node = SensorNode()
    rclpy.spin(sensor_node)
    sensor_node.destroy_node()
    rclpy.shutdown()

if __name__ == '__main__':
    main()
```

### Controller Node Example

A controller node that subscribes to sensor data and publishes commands:

```python
import rclpy
from rclpy.node import Node
from sensor_msgs.msg import LaserScan
from geometry_msgs.msg import Twist

class ControllerNode(Node):
    def __init__(self):
        super().__init__('controller_node')
        self.subscription = self.create_subscription(
            LaserScan,
            'scan',
            self.scan_callback,
            10)
        self.publisher = self.create_publisher(Twist, 'cmd_vel', 10)

    def scan_callback(self, msg):
        # Simple obstacle avoidance
        min_distance = min(msg.ranges) if msg.ranges else float('inf')

        cmd = Twist()
        if min_distance > 1.0:  # No obstacle nearby
            cmd.linear.x = 0.5  # Move forward
        else:
            cmd.angular.z = 0.5  # Turn to avoid obstacle

        self.publisher.publish(cmd)

def main(args=None):
    rclpy.init(args=args)
    controller_node = ControllerNode()
    rclpy.spin(controller_node)
    controller_node.destroy_node()
    rclpy.shutdown()

if __name__ == '__main__':
    main()
```

## Exercises

1. **Implementation Exercise**: Create a simple publisher that publishes temperature data every second and a subscriber that prints the received temperature values.

2. **Design Exercise**: For a humanoid robot walking application, identify what would be appropriate topics, services, and actions for:
   - Joint position control
   - Walking gait planning
   - Obstacle detection
   - Emergency stop

## Summary

This chapter covered the core communication mechanisms in ROS 2: nodes, topics, and services. We explored how to implement publishers and subscribers for asynchronous communication, and service clients and servers for synchronous communication. We also looked at practical examples of how these concepts apply to robot systems.

## Visual Aids

*Note: Diagrams and visual aids would be included in the final published version to illustrate:*
- *Node architecture and lifecycle*
- *Publisher-subscriber communication flow*
- *Service request-response interaction*
- *Quality of Service configuration options*

## Further Reading

- [ROS 2 Topics and Services](https://docs.ros.org/en/humble/Concepts/About-Topics-And-Services.html)
- [ROS 2 Quality of Service](https://docs.ros.org/en/humble/Concepts/About-Quality-of-Service-Settings.html)
- [ROS 2 Client Libraries](https://docs.ros.org/en/humble/How-To-Guides/Using-RCL-CPP.html)

---

**Previous Chapter**: [ROS 2 Basics](./fundamentals.md)
**Next Chapter**: [URDF & Python-ROS Integration](./urdf-modeling.md)