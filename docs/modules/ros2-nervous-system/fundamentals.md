---
sidebar_position: 1
title: "ROS 2 Basics"
---

# ROS 2 Fundamentals for Physical AI

## Learning Objectives

By the end of this chapter, you will be able to:
- Understand the role of ROS 2 as a middleware nervous system for humanoid robots
- Identify and explain the purpose of nodes, topics, services, and actions in ROS 2 architecture
- Recognize communication patterns in ROS 2 systems
- Appreciate how ROS 2 enables embodied intelligence in robotics

## Introduction to ROS 2

Robot Operating System 2 (ROS 2) is not an actual operating system but rather a flexible framework for writing robot software. It is a collection of tools, libraries, and conventions that aim to simplify the task of creating complex and robust robot behavior across a heterogeneous system of different computers.

### The Nervous System Analogy

Think of ROS 2 as the nervous system of a robot. Just as the nervous system in biological organisms transmits signals between different parts of the body, ROS 2 enables communication between different software components of a robot. This middleware architecture allows for distributed computation, where different parts of the robot's "brain" can run on different processors or computers while still communicating effectively.

## Core Concepts

### Nodes

A node is a process that performs computation in the ROS 2 system. It represents a component of the robotic nervous system. In ROS 2, a software system is built as a collection of nodes that communicate with each other.

**Key characteristics of nodes:**
- Each node runs a specific task or set of tasks
- Nodes can be written in different programming languages (C++, Python, etc.)
- Nodes can be distributed across multiple machines
- Nodes are managed by a ROS 2 daemon

### Topics and Message Passing

Topics are communication channels for data streams between nodes, enabling the flow of information in the robotic nervous system. Communication via topics follows a publish/subscribe model where nodes can publish data to a topic or subscribe to data from a topic.

**Key characteristics of topics:**
- One-to-many communication (one publisher, multiple subscribers)
- Asynchronous communication
- Data is published in a specific message type
- Communication is anonymous (publishers don't know who subscribes)

### Services

Services provide a request-response communication pattern for direct interaction between nodes. This is synchronous communication where a client sends a request and waits for a response from a server.

**Key characteristics of services:**
- One-to-one communication
- Synchronous communication
- Request-response pattern
- Request and response have specific types

### Actions

Actions are goal-oriented communication patterns for long-running tasks with feedback. They are similar to services but are designed for tasks that take a significant amount of time to complete.

**Key characteristics of actions:**
- Goal → Feedback → Result pattern
- Support for preempting/canceling goals
- Provide continuous feedback during execution
- Suitable for long-running operations

## ROS 2 System Architecture

### DDS (Data Distribution Service)

ROS 2 uses DDS as its underlying communication middleware. DDS provides the infrastructure for the publish/subscribe communication model and handles the discovery of nodes and topics across the network.

### RMW (ROS Middleware) Abstraction

ROS 2 provides an abstraction layer over different DDS implementations, allowing users to switch between different middleware implementations without changing their application code.

### Lifecycle Management

ROS 2 includes lifecycle management capabilities that allow for more sophisticated node management, including initialization, configuration, and cleanup phases.

## Practical Example: Publisher-Subscriber Pattern

Let's examine a simple publisher-subscriber example to understand how nodes communicate via topics:

```python
# Publisher example
import rclpy
from rclpy.node import Node
from std_msgs.msg import String

class MinimalPublisher(Node):
    def __init__(self):
        super().__init__('minimal_publisher')
        self.publisher_ = self.create_publisher(String, 'topic', 10)
        timer_period = 0.5  # seconds
        self.timer = self.create_timer(timer_period, self.timer_callback)
        self.i = 0

    def timer_callback(self):
        msg = String()
        msg.data = 'Hello World: %d' % self.i
        self.publisher_.publish(msg)
        self.get_logger().info('Publishing: "%s"' % msg.data)
        self.i += 1

def main(args=None):
    rclpy.init(args=args)
    minimal_publisher = MinimalPublisher()
    rclpy.spin(minimal_publisher)
    minimal_publisher.destroy_node()
    rclpy.shutdown()

if __name__ == '__main__':
    main()
```

## Exercises

1. **Conceptual Understanding**: Explain in your own words the difference between topics, services, and actions. When would you use each one?

2. **Real-World Application**: Identify potential nodes, topics, and services in a humanoid robot system. For example, consider a walking controller node, a sensor data processing node, and a high-level planning node.

## Summary

This chapter introduced the fundamental concepts of ROS 2, including nodes, topics, services, and actions. We explored how ROS 2 serves as the middleware nervous system for humanoid robots, enabling distributed computation and communication between different software components. Understanding these concepts is crucial for working with Physical AI and humanoid robotics systems.

## Visual Aids

*Note: Diagrams and visual aids would be included in the final published version to illustrate:*
- *The relationship between nodes, topics, services, and actions*
- *The publisher-subscriber communication pattern*
- *The request-response service pattern*
- *The goal-feedback-result action pattern*

## Further Reading

- [Official ROS 2 Documentation](https://docs.ros.org/en/humble/)
- [ROS 2 Design Overview](https://design.ros2.org/)
- [ROS 2 Tutorials](https://docs.ros.org/en/humble/Tutorials.html)

---

**Next Chapter**: [Nodes, Topics, Services](./nodes-topics-services.md)