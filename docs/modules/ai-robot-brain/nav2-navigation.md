---
sidebar_position: 3
title: "Navigation with Nav2"
---

# Navigation with Nav2 for Humanoid Robotics

## Learning Objectives

By the end of this chapter, you will be able to:
- Understand the Nav2 navigation stack architecture and components
- Configure Nav2 for humanoid robot navigation applications
- Implement path planning algorithms suitable for humanoid locomotion
- Integrate Nav2 with perception systems for obstacle avoidance
- Set up navigation parameters for dynamic humanoid environments
- Deploy Nav2 navigation systems on humanoid robots with Isaac ROS

## Introduction to Navigation 2 (Nav2)

Navigation 2 (Nav2) is the next-generation navigation stack for ROS 2, designed to provide reliable and robust navigation capabilities for mobile robots. For humanoid robots, Nav2 provides essential capabilities for autonomous navigation, path planning, and obstacle avoidance while considering the unique kinematic and dynamic constraints of bipedal locomotion.

### Nav2 Architecture Overview

Nav2 is built as a collection of modular, configurable components that work together to provide navigation capabilities:

1. **Navigation Server**: Coordinates all navigation components
2. **Planner Server**: Global and local path planning
3. **Controller Server**: Path following and motion control
4. **Recovery Server**: Recovery behaviors for navigation failures
5. **Lifecycle Manager**: Manages component lifecycle states

### Key Features of Nav2

- **Modular Design**: Components can be replaced with custom implementations
- **Behavior Trees**: Flexible behavior composition for complex navigation tasks
- **Plugin Architecture**: Extensible with custom algorithms and behaviors
- **Real-time Performance**: Optimized for real-time navigation applications
- **Simulation Ready**: Works seamlessly with Gazebo and Isaac Sim

## Nav2 Components and Configuration

### Navigation Server

The Navigation Server orchestrates the navigation system:

```yaml
# Example Nav2 configuration file (nav2_params.yaml)
amcl:
  ros__parameters:
    use_sim_time: True
    alpha1: 0.2
    alpha2: 0.2
    alpha3: 0.2
    alpha4: 0.2
    alpha5: 0.2
    base_frame_id: "base_footprint"
    beam_skip_distance: 0.5
    beam_skip_error_threshold: 0.9
    beam_skip_threshold: 0.3
    do_beamskip: false
    global_frame_id: "map"
    lambda_short: 0.1
    laser_likelihood_max_dist: 2.0
    laser_max_range: 100.0
    laser_min_range: -1.0
    laser_model_type: "likelihood_field"
    max_beams: 60
    max_particles: 2000
    min_particles: 500
    odom_frame_id: "odom"
    pf_err: 0.05
    pf_z: 0.99
    recovery_alpha_fast: 0.0
    recovery_alpha_slow: 0.0
    resample_interval: 1
    robot_model_type: "nav2_amcl::DifferentialMotionModel"
    save_pose_rate: 0.5
    sigma_hit: 0.2
    tf_broadcast: true
    transform_tolerance: 1.0
    update_min_a: 0.2
    update_min_d: 0.25
    z_hit: 0.5
    z_max: 0.05
    z_rand: 0.5
    z_short: 0.05
    scan_topic: scan

bt_navigator:
  ros__parameters:
    use_sim_time: True
    global_frame: map
    robot_base_frame: base_link
    odom_topic: /odom
    bt_loop_duration: 10
    default_server_timeout: 20
    enable_groot_monitoring: True
    groot_zmq_publisher_port: 1666
    groot_zmq_server_port: 1667
    navigate_through_poses: False
    navigate_to_pose: True
    behavior_tree:
      main_tree_file: "navigate_w_replanning_and_recovery.xml"

controller_server:
  ros__parameters:
    use_sim_time: True
    controller_frequency: 20.0
    min_x_velocity_threshold: 0.001
    min_y_velocity_threshold: 0.001
    min_theta_velocity_threshold: 0.001
    progress_checker_plugin: "progress_checker"
    goal_checker_plugin: "goal_checker"
    controller_plugins: ["FollowPath"]

    # Progress checker parameters
    progress_checker:
      plugin: "nav2_controller::SimpleProgressChecker"
      required_movement_radius: 0.5
      movement_time_allowance: 10.0

    # Goal checker parameters
    goal_checker:
      plugin: "nav2_controller::SimpleGoalChecker"
      xy_goal_tolerance: 0.25
      yaw_goal_tolerance: 0.25
      stateful: True

    # Controller parameters
    FollowPath:
      plugin: "nav2_rotation_shim_controller::RotationShimController"
      progress_checker_plugin: "progress_checker"
      goal_checker_plugin: "goal_checker"
      in_collision_distance: 0.05
      in_cone_angle: 1.0471975512
      forward_sampling_distance: 0.5
      min_forward_sampling_distance: 0.1
      rotation_shim:
        plugin: "nav2_controller::SimplePurePersuit"
        lookahead_dist: 0.6
        rotate_to_heading_angular_dist: 0.785
        use_velocity_scaled_lookahead_dist: false
        min_approach_linear_velocity: 0.05
        approach_velocity_scaling_dist: 0.5
        lookahead_time: 1.5
```

### Global Planner Configuration

```yaml
planner_server:
  ros__parameters:
    use_sim_time: True
    planner_plugins: ["GridBased"]
    GridBased:
      plugin: "nav2_navfn_planner::NavfnPlanner"
      tolerance: 0.5
      use_astar: false
      allow_unknown: true
```

### Local Planner Configuration

```yaml
# Local costmap configuration
local_costmap:
  local_costmap:
    ros__parameters:
      update_frequency: 5.0
      publish_frequency: 2.0
      global_frame: odom
      robot_base_frame: base_link
      use_sim_time: True
      rolling_window: true
      width: 3
      height: 3
      resolution: 0.05
      robot_radius: 0.22
      plugins: ["voxel_layer", "inflation_layer"]
      inflation_layer:
        plugin: "nav2_costmap_2d::InflationLayer"
        cost_scaling_factor: 3.0
        inflation_radius: 0.55
      voxel_layer:
        plugin: "nav2_costmap_2d::VoxelLayer"
        enabled: True
        publish_voxel_map: True
        origin_z: 0.0
        z_resolution: 0.05
        z_voxels: 16
        max_obstacle_height: 2.0
        mark_threshold: 0
        observation_sources: scan
        scan:
          topic: /scan
          max_obstacle_height: 2.0
          clearing: True
          marking: True
          data_type: "LaserScan"
          raytrace_max_range: 3.0
          raytrace_min_range: 0.0
          obstacle_max_range: 2.5
          obstacle_min_range: 0.0
```

## Nav2 Behavior Trees

### Behavior Tree Fundamentals

Nav2 uses behavior trees for flexible navigation task composition:

```xml
<!-- Example behavior tree: navigate_w_replanning_and_recovery.xml -->
<root main_tree_to_execute="MainTree">
  <BehaviorTree ID="MainTree">
    <RecoveryNode number_of_retries="6" name="NavigateRecovery">
      <PipelineSequence name="NavigateWithReplanning">
        <RateController hz="1.0">
          <ComputePathToPose goal="{goal}" path="{path}" planner_id="GridBased"/>
        </RateController>
        <RecoveryNode number_of_retries="1" name="FollowPathRecovery">
          <FollowPath path="{path}" controller_id="FollowPath"/>
          <ReactiveRecovery recovery_behavior_id="spin" name="Spin"/>
        </RecoveryNode>
      </PipelineSequence>
      <ReactiveRecovery recovery_behavior_id="backup" name="Backup"/>
    </RecoveryNode>
  </BehaviorTree>

  <BehaviorTree ID="Spin">
    <Spin spin_dist="1.57"/>
  </BehaviorTree>

  <BehaviorTree ID="Backup">
    <Backup backup_dist="0.15" backup_speed="0.025"/>
  </BehaviorTree>
</root>
```

### Custom Behavior Trees for Humanoid Navigation

```xml
<!-- Humanoid-specific behavior tree with additional safety checks -->
<root main_tree_to_execute="HumanoidMainTree">
  <BehaviorTree ID="HumanoidMainTree">
    <SequenceStar name="HumanoidNavigateSequence">
      <!-- Check if humanoid is in stable state -->
      <CheckBalanceState name="CheckBalance"/>

      <!-- Compute path to goal -->
      <RateController hz="0.5">
        <ComputePathToPose goal="{goal}" path="{path}" planner_id="GridBased"/>
      </RateController>

      <!-- Follow path with humanoid-specific controller -->
      <RecoveryNode number_of_retries="3" name="HumanoidFollowPathRecovery">
        <FollowPath path="{path}" controller_id="HumanoidPathFollower"/>

        <!-- Recovery behaviors -->
        <ReactiveRecovery recovery_behavior_id="humanoid_spin" name="HumanoidSpinRecovery"/>
      </RecoveryNode>

      <!-- Additional safety checks -->
      <CheckObstacleClearance min_distance="0.5" name="CheckObstacleClearance"/>
    </SequenceStar>
  </BehaviorTree>

  <BehaviorTree ID="HumanoidSpin">
    <Spin spin_dist="0.785" linear_x_slew_rate="0.2" angular_z_slew_rate="0.5"/>
  </BehaviorTree>
</root>
```

## Humanoid-Specific Navigation Considerations

### Kinematic Constraints

Humanoid robots have unique kinematic constraints that must be considered in navigation:

```python
import rclpy
from rclpy.node import Node
from nav2_msgs.action import NavigateToPose
from geometry_msgs.msg import PoseStamped, Twist
from std_msgs.msg import Float32
import math

class HumanoidNavigationConstraints(Node):
    def __init__(self):
        super().__init__('humanoid_navigation_constraints')

        # Humanoid-specific parameters
        self.max_step_length = 0.3  # Maximum step length for humanoid
        self.min_turn_radius = 0.5  # Minimum turn radius
        self.max_linear_velocity = 0.4  # Max walking speed
        self.max_angular_velocity = 0.5  # Max turning speed
        self.max_acceleration = 0.2     # Max acceleration
        self.balance_margin = 0.1       # Safety margin for balance

        # Publisher for constrained velocity commands
        self.cmd_vel_pub = self.create_publisher(Twist, '/humanoid/cmd_vel', 10)

        # Subscriber for raw velocity commands from Nav2
        self.nav_vel_sub = self.create_subscription(
            Twist, '/nav_vel_raw', self.nav_vel_callback, 10
        )

    def nav_vel_callback(self, msg):
        """Process velocity commands from Nav2 and apply humanoid constraints"""
        constrained_vel = Twist()

        # Apply linear velocity constraints
        linear_speed = math.sqrt(msg.linear.x**2 + msg.linear.y**2)
        if linear_speed > self.max_linear_velocity:
            scale = self.max_linear_velocity / linear_speed
            constrained_vel.linear.x = msg.linear.x * scale
            constrained_vel.linear.y = msg.linear.y * scale
        else:
            constrained_vel.linear.x = msg.linear.x
            constrained_vel.linear.y = msg.linear.y

        # Apply angular velocity constraints
        if abs(msg.angular.z) > self.max_angular_velocity:
            constrained_vel.angular.z = math.copysign(
                self.max_angular_velocity, msg.angular.z
            )
        else:
            constrained_vel.angular.z = msg.angular.z

        # Apply acceleration constraints
        current_vel = self.get_current_velocity()
        constrained_vel = self.apply_acceleration_constraints(
            current_vel, constrained_vel
        )

        # Publish constrained velocity
        self.cmd_vel_pub.publish(constrained_vel)

    def apply_acceleration_constraints(self, current_vel, target_vel):
        """Apply acceleration limits to velocity commands"""
        max_delta_linear = self.max_acceleration * 0.1  # 0.1s time step
        max_delta_angular = self.max_acceleration * 0.1

        # Calculate velocity differences
        delta_linear = math.sqrt(
            (target_vel.linear.x - current_vel.linear.x)**2 +
            (target_vel.linear.y - current_vel.linear.y)**2
        )
        delta_angular = abs(target_vel.angular.z - current_vel.angular.z)

        # Apply acceleration limits
        if delta_linear > max_delta_linear:
            scale = max_delta_linear / delta_linear
            target_vel.linear.x = current_vel.linear.x + (target_vel.linear.x - current_vel.linear.x) * scale
            target_vel.linear.y = current_vel.linear.y + (target_vel.linear.y - current_vel.linear.y) * scale

        if delta_angular > max_delta_angular:
            target_vel.angular.z = current_vel.angular.z + math.copysign(
                max_delta_angular, target_vel.angular.z - current_vel.angular.z
            )

        return target_vel

    def get_current_velocity(self):
        """Get current robot velocity (from odometry)"""
        # Implementation would get current velocity from odometry
        return Twist()  # Placeholder
```

### Balance-Aware Navigation

```python
import numpy as np
from geometry_msgs.msg import Vector3
from sensor_msgs.msg import Imu

class BalanceAwareNavigation(Node):
    def __init__(self):
        super().__init__('balance_aware_navigation')

        # Subscribe to IMU for balance information
        self.imu_sub = self.create_subscription(
            Imu, '/imu/data', self.imu_callback, 10
        )

        # Balance state variables
        self.roll = 0.0
        self.pitch = 0.0
        self.balance_threshold = 0.3  # Radians

        # Navigation state
        self.navigation_enabled = True

    def imu_callback(self, msg):
        """Process IMU data to monitor balance"""
        # Convert quaternion to roll/pitch
        quat = msg.orientation
        self.roll, self.pitch, yaw = self.quaternion_to_euler(
            quat.x, quat.y, quat.z, quat.w
        )

        # Check if robot is balanced
        balance_ok = (abs(self.roll) < self.balance_threshold and
                      abs(self.pitch) < self.balance_threshold)

        if not balance_ok and self.navigation_enabled:
            self.get_logger().warn('Balance threshold exceeded, pausing navigation')
            self.pause_navigation()

    def quaternion_to_euler(self, x, y, z, w):
        """Convert quaternion to Euler angles"""
        # Roll (x-axis rotation)
        sinr_cosp = 2 * (w * x + y * z)
        cosr_cosp = 1 - 2 * (x * x + y * y)
        roll = math.atan2(sinr_cosp, cosr_cosp)

        # Pitch (y-axis rotation)
        sinp = 2 * (w * y - z * x)
        if abs(sinp) >= 1:
            pitch = math.copysign(math.pi / 2, sinp)
        else:
            pitch = math.asin(sinp)

        # Yaw (z-axis rotation)
        siny_cosp = 2 * (w * z + x * y)
        cosy_cosp = 1 - 2 * (y * y + z * z)
        yaw = math.atan2(siny_cosp, cosy_cosp)

        return roll, pitch, yaw

    def pause_navigation(self):
        """Pause navigation when balance is compromised"""
        self.navigation_enabled = False
        # Stop robot
        stop_cmd = Twist()
        self.cmd_vel_pub.publish(stop_cmd)

    def resume_navigation(self):
        """Resume navigation when balance is restored"""
        if abs(self.roll) < self.balance_threshold * 0.8 and \
           abs(self.pitch) < self.balance_threshold * 0.8:
            self.navigation_enabled = True
            self.get_logger().info('Balance restored, resuming navigation')
```

## Nav2 Integration with Isaac ROS

### Isaac ROS Navigation Components

```python
import rclpy
from rclpy.node import Node
from sensor_msgs.msg import LaserScan, Image
from nav_msgs.msg import OccupancyGrid, Odometry
from geometry_msgs.msg import PoseStamped, Twist
from tf2_ros import TransformBroadcaster
import numpy as np
import cv2
from cv_bridge import CvBridge

class IsaacROSNav2Integrator(Node):
    def __init__(self):
        super().__init__('isaac_ros_nav2_integrator')

        # Initialize Isaac ROS specific components
        self.bridge = CvBridge()

        # Subscribe to Isaac ROS perception outputs
        self.depth_sub = self.create_subscription(
            Image, '/depth/image_rect_raw', self.depth_callback, 10
        )
        self.lidar_sub = self.create_subscription(
            LaserScan, '/scan', self.lidar_callback, 10
        )
        self.odom_sub = self.create_subscription(
            Odometry, '/odom', self.odom_callback, 10
        )

        # Publishers for Nav2
        self.costmap_pub = self.create_publisher(
            OccupancyGrid, '/global_costmap/costmap', 10
        )
        self.local_costmap_pub = self.create_publisher(
            OccupancyGrid, '/local_costmap/costmap', 10
        )

        # Isaac ROS perception data
        self.depth_image = None
        self.lidar_data = None
        self.robot_pose = None

        # Costmap parameters
        self.costmap_resolution = 0.05
        self.costmap_width = 200  # cells
        self.costmap_height = 200  # cells

    def depth_callback(self, msg):
        """Process depth image from Isaac ROS"""
        try:
            self.depth_image = self.bridge.imgmsg_to_cv2(msg, desired_encoding='32FC1')
        except Exception as e:
            self.get_logger().error(f'Error converting depth image: {e}')

    def lidar_callback(self, msg):
        """Process LIDAR data from Isaac ROS"""
        self.lidar_data = msg

    def odom_callback(self, msg):
        """Process odometry from Isaac ROS"""
        self.robot_pose = msg.pose.pose

    def create_costmap_from_depth(self):
        """Create costmap from depth image"""
        if self.depth_image is None:
            return None

        # Process depth image to create obstacle map
        # This is a simplified example - in practice, you'd use more sophisticated processing
        height, width = self.depth_image.shape
        obstacle_map = np.zeros((height, width), dtype=np.int8)

        # Threshold depth values to detect obstacles
        obstacle_threshold = 2.0  # meters
        obstacle_map[self.depth_image < obstacle_threshold] = 100  # occupied
        obstacle_map[self.depth_image > 10.0] = 0  # free space

        # Create OccupancyGrid message
        costmap_msg = OccupancyGrid()
        costmap_msg.header.stamp = self.get_clock().now().to_msg()
        costmap_msg.header.frame_id = 'map'
        costmap_msg.info.resolution = self.costmap_resolution
        costmap_msg.info.width = width
        costmap_msg.info.height = height
        costmap_msg.info.origin.position.x = -width * self.costmap_resolution / 2
        costmap_msg.info.origin.position.y = -height * self.costmap_resolution / 2

        # Flatten the 2D array to 1D for the message
        costmap_msg.data = obstacle_map.flatten().tolist()

        return costmap_msg

    def integrate_with_nav2(self):
        """Integrate Isaac ROS perception with Nav2"""
        # Create costmaps from perception data
        global_costmap = self.create_costmap_from_depth()

        if global_costmap is not None:
            # Publish to Nav2
            self.costmap_pub.publish(global_costmap)

        # Also create local costmap if needed
        if self.lidar_data is not None:
            local_costmap = self.create_local_costmap_from_lidar()
            if local_costmap is not None:
                self.local_costmap_pub.publish(local_costmap)

    def create_local_costmap_from_lidar(self):
        """Create local costmap from LIDAR data"""
        if self.lidar_data is None or self.robot_pose is None:
            return None

        # Create costmap based on LIDAR data
        width = int(6.0 / self.costmap_resolution)  # 6m x 6m area
        height = width
        local_costmap = np.zeros((height, width), dtype=np.int8)

        # Convert LIDAR ranges to obstacle positions in costmap
        angle_increment = self.lidar_data.angle_increment
        current_angle = self.lidar_data.angle_min

        robot_x = int(width / 2)
        robot_y = int(height / 2)

        for range_val in self.lidar_data.ranges:
            if not (np.isfinite(range_val) and
                    self.lidar_data.range_min < range_val < self.lidar_data.range_max):
                current_angle += angle_increment
                continue

            # Calculate position in costmap
            x = robot_x + int(range_val * math.cos(current_angle) / self.costmap_resolution)
            y = robot_y + int(range_val * math.sin(current_angle) / self.costmap_resolution)

            # Mark as occupied if within bounds
            if 0 <= x < width and 0 <= y < height:
                local_costmap[y, x] = 100

            current_angle += angle_increment

        # Create OccupancyGrid message
        costmap_msg = OccupancyGrid()
        costmap_msg.header.stamp = self.lidar_data.header.stamp
        costmap_msg.header.frame_id = 'odom'
        costmap_msg.info.resolution = self.costmap_resolution
        costmap_msg.info.width = width
        costmap_msg.info.height = height
        costmap_msg.info.origin.position.x = -3.0  # Center of 6m x 6m area
        costmap_msg.info.origin.position.y = -3.0
        costmap_msg.info.origin.position.z = 0.0
        costmap_msg.info.origin.orientation.w = 1.0

        costmap_msg.data = local_costmap.flatten().tolist()

        return costmap_msg
```

## Path Planning Algorithms for Humanoid Robots

### Humanoid Path Planner Implementation

```python
import numpy as np
from scipy.spatial.distance import cdist
import heapq

class HumanoidPathPlanner:
    def __init__(self, costmap, resolution=0.05):
        self.costmap = costmap
        self.resolution = resolution
        self.height, self.width = costmap.shape

    def plan_path(self, start, goal, robot_radius=0.3):
        """
        Plan path for humanoid robot considering step constraints

        Args:
            start: (x, y) start position in map coordinates
            goal: (x, y) goal position in map coordinates
            robot_radius: Robot radius for collision checking
        """
        # Convert to grid coordinates
        start_grid = (int(start[1] / self.resolution), int(start[0] / self.resolution))
        goal_grid = (int(goal[1] / self.resolution), int(goal[0] / self.resolution))

        # Check if start and goal are valid
        if not self.is_valid_cell(start_grid[1], start_grid[0]) or \
           not self.is_valid_cell(goal_grid[1], goal_grid[0]):
            return None

        # Run A* path planning with humanoid constraints
        path = self.a_star_humonoid(start_grid, goal_grid, robot_radius)

        if path is None:
            return None

        # Convert grid path back to world coordinates
        world_path = []
        for grid_x, grid_y in path:
            world_x = grid_x * self.resolution
            world_y = grid_y * self.resolution
            world_path.append((world_x, world_y))

        return world_path

    def a_star_humonoid(self, start, goal, robot_radius):
        """A* algorithm with humanoid step constraints"""
        # Define possible moves (limited step size for humanoid)
        max_step_cells = int(0.3 / self.resolution)  # 30cm max step

        moves = []
        for dx in range(-max_step_cells, max_step_cells + 1):
            for dy in range(-max_step_cells, max_step_cells + 1):
                if dx == 0 and dy == 0:
                    continue
                if dx*dx + dy*dy <= max_step_cells*max_step_cells:  # Circular constraint
                    moves.append((dx, dy))

        # A* algorithm
        open_set = [(0, start)]
        came_from = {}
        g_score = {start: 0}
        f_score = {start: self.heuristic(start, goal)}

        while open_set:
            current = heapq.heappop(open_set)[1]

            if current == goal:
                # Reconstruct path
                path = [current]
                while current in came_from:
                    current = came_from[current]
                    path.append(current)
                path.reverse()
                return path

            for dx, dy in moves:
                neighbor = (current[0] + dx, current[1] + dy)

                # Check bounds and collision
                if not self.is_valid_cell(neighbor[1], neighbor[0]):
                    continue

                # Calculate step cost (considering terrain and step size)
                step_cost = self.calculate_step_cost(current, neighbor, robot_radius)
                if step_cost == float('inf'):
                    continue

                tentative_g_score = g_score[current] + step_cost

                if neighbor not in g_score or tentative_g_score < g_score[neighbor]:
                    came_from[neighbor] = current
                    g_score[neighbor] = tentative_g_score
                    f_score[neighbor] = g_score[neighbor] + self.heuristic(neighbor, goal)
                    heapq.heappush(open_set, (f_score[neighbor], neighbor))

        return None  # No path found

    def calculate_step_cost(self, current, neighbor, robot_radius):
        """Calculate cost of stepping from current to neighbor"""
        # Get the cells along the path between current and neighbor
        cells = self.get_line_cells(current, neighbor)

        # Check collision for each cell
        for x, y in cells:
            if not self.is_valid_cell(x, y):
                return float('inf')

        # Calculate geometric distance cost
        dist = np.sqrt((neighbor[0] - current[0])**2 + (neighbor[1] - current[1])**2)

        # Add cost based on terrain difficulty (from costmap)
        terrain_cost = 0
        for x, y in cells:
            terrain_cost += self.costmap[y, x] / 100.0  # Normalize costmap value

        return dist + terrain_cost

    def get_line_cells(self, start, end):
        """Get all cells along a line from start to end using Bresenham's algorithm"""
        cells = []
        x0, y0 = start
        x1, y1 = end

        dx = abs(x1 - x0)
        dy = abs(y1 - y0)
        sx = 1 if x0 < x1 else -1
        sy = 1 if y0 < y1 else -1
        err = dx - dy

        x, y = x0, y0
        while True:
            cells.append((x, y))
            if x == x1 and y == y1:
                break
            e2 = 2 * err
            if e2 > -dy:
                err -= dy
                x += sx
            if e2 < dx:
                err += dx
                y += sy

        return cells

    def is_valid_cell(self, x, y):
        """Check if a cell is valid (not out of bounds or occupied)"""
        if x < 0 or x >= self.width or y < 0 or y >= self.height:
            return False
        # Consider cell occupied if cost is above threshold
        return self.costmap[y, x] < 50  # 50 is threshold for "occupied"

    def heuristic(self, a, b):
        """Heuristic function for A* (Euclidean distance)"""
        return np.sqrt((a[0] - b[0])**2 + (a[1] - b[1])**2)
```

## Navigation Recovery Behaviors

### Custom Recovery Behaviors for Humanoid Robots

```python
import rclpy
from rclpy.action import ActionServer
from rclpy.node import Node
from nav2_msgs.action import Recover
from geometry_msgs.msg import Twist
from std_msgs.msg import Bool
import time

class HumanoidRecoveryBehaviors(Node):
    def __init__(self):
        super().__init__('humanoid_recovery_behaviors')

        # Recovery action server
        self.recovery_server = ActionServer(
            self,
            Recover,
            'humanoid_recovery',
            self.execute_recovery
        )

        # Publisher for velocity commands
        self.cmd_vel_pub = self.create_publisher(Twist, '/cmd_vel', 10)

        # Balance monitoring
        self.balance_ok = True
        self.balance_sub = self.create_subscription(
            Bool, '/balance_ok', self.balance_callback, 10
        )

    def balance_callback(self, msg):
        """Update balance status"""
        self.balance_ok = msg.data

    def execute_recovery(self, goal_handle):
        """Execute recovery behavior based on type"""
        recovery_type = goal_handle.request.behavior.type
        feedback = Recover.Feedback()
        result = Recover.Result()

        if not self.balance_ok:
            self.get_logger().info('Robot not balanced, cannot execute recovery')
            result.status = Recover.Result.FAILED
            goal_handle.succeed()
            return result

        if recovery_type == 'HUMANOID_SPIN':
            success = self.execute_spin_recovery()
        elif recovery_type == 'HUMANOID_STOMP':
            success = self.execute_stomp_recovery()
        elif recovery_type == 'HUMANOID_SIDESTEP':
            success = self.execute_sidestep_recovery()
        else:
            self.get_logger().warn(f'Unknown recovery type: {recovery_type}')
            success = False

        if success:
            result.status = Recover.Result.SUCCEEDED
        else:
            result.status = Recover.Result.FAILED

        goal_handle.succeed()
        return result

    def execute_spin_recovery(self):
        """Execute spin recovery (careful turning)"""
        self.get_logger().info('Executing spin recovery')

        # Slow, careful spin to clear local minima
        cmd = Twist()
        cmd.angular.z = 0.2  # Slow angular velocity

        start_time = time.time()
        duration = 5.0  # 5 seconds

        while time.time() - start_time < duration:
            if not self.balance_ok:
                self.get_logger().warn('Balance lost during spin recovery')
                cmd = Twist()  # Stop
                self.cmd_vel_pub.publish(cmd)
                return False

            self.cmd_vel_pub.publish(cmd)
            time.sleep(0.1)

        # Stop
        cmd = Twist()
        self.cmd_vel_pub.publish(cmd)
        return True

    def execute_stomp_recovery(self):
        """Execute stomp recovery (small forward/backward movements)"""
        self.get_logger().info('Executing stomp recovery')

        # Small forward movement
        cmd = Twist()
        cmd.linear.x = 0.1
        self.cmd_vel_pub.publish(cmd)
        time.sleep(1.0)

        # Small backward movement
        cmd.linear.x = -0.1
        self.cmd_vel_pub.publish(cmd)
        time.sleep(1.0)

        # Stop
        cmd = Twist()
        self.cmd_vel_pub.publish(cmd)
        return True

    def execute_sidestep_recovery(self):
        """Execute sidestep recovery (lateral movement)"""
        self.get_logger().info('Executing sidestep recovery')

        # Small lateral movement
        cmd = Twist()
        cmd.linear.y = 0.1  # Move right
        self.cmd_vel_pub.publish(cmd)
        time.sleep(1.0)

        cmd.linear.y = -0.1  # Move left
        self.cmd_vel_pub.publish(cmd)
        time.sleep(1.0)

        # Stop
        cmd = Twist()
        self.cmd_vel_pub.publish(cmd)
        return True
```

## Performance Optimization and Evaluation

### Navigation Performance Metrics

```python
import numpy as np
import matplotlib.pyplot as plt
from scipy.spatial.distance import euclidean

class NavigationPerformanceEvaluator:
    def __init__(self):
        self.path_executed = []
        self.path_planned = []
        self.execution_times = []
        self.success_count = 0
        self.failure_count = 0
        self.total_goals = 0

    def add_planned_path(self, path):
        """Add planned path for evaluation"""
        self.path_planned = path

    def add_executed_position(self, position):
        """Add executed position during navigation"""
        self.path_executed.append(position)

    def add_execution_time(self, time):
        """Add navigation execution time"""
        self.execution_times.append(time)

    def record_goal_result(self, success):
        """Record whether goal was reached successfully"""
        self.total_goals += 1
        if success:
            self.success_count += 1
        else:
            self.failure_count += 1

    def calculate_metrics(self):
        """Calculate navigation performance metrics"""
        if not self.path_planned or not self.path_executed:
            return {}

        # Path efficiency (actual path length vs optimal path length)
        planned_length = self.calculate_path_length(self.path_planned)
        executed_length = self.calculate_path_length(self.path_executed)

        # Deviation from planned path
        deviation = self.calculate_path_deviation(self.path_planned, self.path_executed)

        # Success rate
        success_rate = self.success_count / max(1, self.total_goals) if self.total_goals > 0 else 0

        # Average execution time
        avg_time = np.mean(self.execution_times) if self.execution_times else 0
        std_time = np.std(self.execution_times) if self.execution_times else 0

        metrics = {
            'path_efficiency': planned_length / max(executed_length, 0.001),  # Higher is better
            'path_deviation': deviation,
            'success_rate': success_rate,
            'average_execution_time': avg_time,
            'std_execution_time': std_time,
            'planned_path_length': planned_length,
            'executed_path_length': executed_length,
            'total_goals': self.total_goals,
            'success_count': self.success_count,
            'failure_count': self.failure_count
        }

        return metrics

    def calculate_path_length(self, path):
        """Calculate total length of a path"""
        if len(path) < 2:
            return 0.0

        length = 0.0
        for i in range(1, len(path)):
            length += euclidean(path[i-1], path[i])

        return length

    def calculate_path_deviation(self, planned_path, executed_path):
        """Calculate average deviation between planned and executed paths"""
        if not planned_path or not executed_path:
            return float('inf')

        # Find closest points and calculate average distance
        total_distance = 0.0
        count = 0

        for exec_pos in executed_path:
            min_dist = float('inf')
            for plan_pos in planned_path:
                dist = euclidean(exec_pos, plan_pos)
                if dist < min_dist:
                    min_dist = dist
            total_distance += min_dist
            count += 1

        return total_distance / max(count, 1)

    def plot_navigation_performance(self):
        """Plot navigation performance metrics"""
        metrics = self.calculate_metrics()

        if not metrics:
            print("No metrics available to plot")
            return

        fig, axes = plt.subplots(2, 2, figsize=(12, 10))

        # Plot success rate
        axes[0, 0].bar(['Success', 'Failure'],
                       [metrics['success_count'], metrics['failure_count']],
                       color=['green', 'red'])
        axes[0, 0].set_title('Navigation Success/Failure')
        axes[0, 0].set_ylabel('Number of Attempts')

        # Plot path efficiency
        axes[0, 1].bar(['Planned', 'Executed'],
                       [metrics['planned_path_length'], metrics['executed_path_length']],
                       color=['blue', 'orange'])
        axes[0, 1].set_title('Path Length Comparison')
        axes[0, 1].set_ylabel('Distance (m)')

        # Plot execution times
        if self.execution_times:
            axes[1, 0].hist(self.execution_times, bins=20, color='lightblue', edgecolor='black')
            axes[1, 0].set_title('Execution Time Distribution')
            axes[1, 0].set_xlabel('Time (s)')
            axes[1, 0].set_ylabel('Frequency')

        # Plot path deviation
        axes[1, 1].text(0.5, 0.5, f"Average Deviation: {metrics['path_deviation']:.2f}m",
                        horizontalalignment='center', verticalalignment='center',
                        transform=axes[1, 1].transAxes, fontsize=14)
        axes[1, 1].set_title('Path Deviation')
        axes[1, 1].set_xlim(0, 1)
        axes[1, 1].set_ylim(0, 1)

        plt.tight_layout()
        plt.show()

        # Print summary
        print(f"Navigation Performance Summary:")
        print(f"  Success Rate: {metrics['success_rate']:.2%}")
        print(f"  Path Efficiency: {metrics['path_efficiency']:.2f}")
        print(f"  Average Execution Time: {metrics['average_execution_time']:.2f}s")
        print(f"  Average Path Deviation: {metrics['path_deviation']:.2f}m")
```

## Troubleshooting Navigation Systems

### Common Navigation Issues and Solutions

1. **Local Minima**: Robot gets stuck in local minima
   - Solution: Implement better global planners, add random walks, improve costmap inflation

2. **Oscillation**: Robot oscillates between positions
   - Solution: Adjust controller parameters, implement hysteresis, improve path smoothing

3. **Collision**: Robot collides with obstacles
   - Solution: Increase costmap inflation, improve sensor coverage, adjust robot footprint

4. **Timeout**: Navigation takes too long
   - Solution: Optimize path planning, adjust frequency of updates, simplify behavior trees

### Navigation Debugging Tools

```python
class NavigationDebugger:
    def __init__(self, node):
        self.node = node
        self.debug_publisher = node.create_publisher(
            MarkerArray, '/nav_debug_markers', 10
        )
        self.marker_id = 0

    def visualize_path(self, path, color=(1.0, 0.0, 0.0), scale=0.1):
        """Visualize path in RViz"""
        marker = Marker()
        marker.header.frame_id = "map"
        marker.header.stamp = self.node.get_clock().now().to_msg()
        marker.ns = "navigation_path"
        marker.id = self.get_next_id()
        marker.type = Marker.LINE_STRIP
        marker.action = Marker.ADD
        marker.scale.x = scale
        marker.color.r = color[0]
        marker.color.g = color[1]
        marker.color.b = color[2]
        marker.color.a = 1.0

        for point in path:
            p = Point()
            p.x = point[0]
            p.y = point[1]
            p.z = 0.0
            marker.points.append(p)

        marker_array = MarkerArray()
        marker_array.markers.append(marker)
        self.debug_publisher.publish(marker_array)

    def visualize_costmap(self, costmap, resolution, origin):
        """Visualize costmap as markers"""
        # This would create markers for each cell in the costmap
        # Implementation would depend on the specific visualization needs

    def get_next_id(self):
        """Get next marker ID"""
        current_id = self.marker_id
        self.marker_id += 1
        return current_id

    def log_navigation_state(self, state):
        """Log navigation state for debugging"""
        self.node.get_logger().info(f'Navigation state: {state}')
```

## Best Practices for Humanoid Navigation

### 1. Safety First

- Always implement emergency stop capabilities
- Monitor robot balance and stability continuously
- Set appropriate velocity and acceleration limits
- Implement proper collision avoidance

### 2. Parameter Tuning

- Start with conservative parameters and gradually increase
- Test in simulation before deploying on hardware
- Monitor performance metrics and adjust accordingly
- Consider environmental conditions in parameter selection

### 3. Integration Strategies

- Ensure proper sensor calibration and synchronization
- Use appropriate coordinate frame conventions
- Implement robust TF tree management
- Plan for graceful degradation when sensors fail

## Exercises

1. **Navigation Setup Exercise**: Configure Nav2 for a humanoid robot simulation with appropriate parameters for bipedal locomotion.

2. **Path Planning Exercise**: Implement a custom path planner that considers humanoid step constraints and balance requirements.

3. **Integration Exercise**: Create a navigation system that integrates Isaac ROS perception with Nav2 for humanoid robot navigation.

## Summary

This chapter covered navigation with Nav2 for humanoid robotics, including the Nav2 architecture, configuration, behavior trees, humanoid-specific considerations, and integration with Isaac ROS. We explored path planning algorithms suitable for humanoid robots, recovery behaviors, and performance evaluation techniques. Proper navigation setup is crucial for autonomous humanoid robot operation in real-world environments.

## Further Reading

- [Nav2 Documentation](https://navigation.ros.org/)
- [Behavior Trees in Robotics](https://arxiv.org/abs/1709.00084)
- [Humanoid Robot Navigation](https://ieeexplore.ieee.org/document/9109381)
- [ROS 2 Navigation Tutorials](https://navigation.ros.org/tutorials/)

---

**Previous Chapter**: [Isaac ROS & VSLAM](./isaac-ros-vslam.md)
**Next Chapter**: [Voice-to-Action](../vla/voice-to-action.md)