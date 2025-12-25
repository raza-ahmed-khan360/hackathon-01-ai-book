---
sidebar_position: 2
title: "Isaac ROS & VSLAM"
---

# Isaac ROS & Visual Simultaneous Localization and Mapping

## Learning Objectives

By the end of this chapter, you will be able to:
- Understand the integration between Isaac ROS and VSLAM systems
- Implement visual SLAM algorithms for humanoid robot navigation
- Configure Isaac ROS packages for visual perception tasks
- Integrate camera sensors with SLAM algorithms in simulation
- Evaluate SLAM performance and optimize parameters for humanoid applications
- Deploy VSLAM systems on real humanoid robots using Isaac ROS

## Introduction to Isaac ROS and VSLAM

Isaac ROS is NVIDIA's collection of hardware-accelerated ROS 2 packages that leverage GPU computing to enable high-performance robotics applications. When combined with Visual Simultaneous Localization and Mapping (VSLAM), Isaac ROS enables humanoid robots to understand and navigate their environment using visual sensors like cameras.

### What is VSLAM?

Visual SLAM (Simultaneous Localization and Mapping) is a technology that allows robots to build a map of an unknown environment while simultaneously tracking their position within that map using visual input from cameras. For humanoid robots, VSLAM is crucial for autonomous navigation, obstacle avoidance, and spatial understanding.

### Isaac ROS Advantages for VSLAM

Isaac ROS provides several advantages for VSLAM applications:

1. **GPU Acceleration**: Leverage NVIDIA GPUs for real-time visual processing
2. **Optimized Algorithms**: Hardware-accelerated computer vision and SLAM algorithms
3. **ROS 2 Integration**: Seamless integration with the ROS 2 ecosystem
4. **Modular Architecture**: Flexible, reusable components for different robot platforms
5. **Simulation Integration**: Easy transition from simulation to real hardware

## Isaac ROS Architecture

### Core Components

Isaac ROS consists of several key components:

1. **Isaac ROS Common**: Shared utilities and data structures
2. **Isaac ROS Image Pipeline**: Optimized image processing components
3. **Isaac ROS VSLAM**: Visual SLAM algorithms and utilities
4. **Isaac ROS Navigation**: Navigation stack components
5. **Isaac ROS Manipulation**: Manipulation-specific components

### Image Pipeline Components

```python
# Example Isaac ROS image pipeline
import rclpy
from rclpy.node import Node
from sensor_msgs.msg import Image
from cv_bridge import CvBridge
import cv2
import numpy as np

class IsaacROSImageProcessor(Node):
    def __init__(self):
        super().__init__('isaac_ros_image_processor')

        # Create subscribers for camera images
        self.image_sub = self.create_subscription(
            Image,
            '/camera/image_raw',
            self.image_callback,
            10
        )

        # Create publishers for processed images
        self.processed_pub = self.create_publisher(
            Image,
            '/camera/image_processed',
            10
        )

        self.bridge = CvBridge()

    def image_callback(self, msg):
        # Convert ROS image to OpenCV format
        cv_image = self.bridge.imgmsg_to_cv2(msg, desired_encoding='bgr8')

        # Apply image processing (e.g., distortion correction, feature extraction)
        processed_image = self.process_image(cv_image)

        # Convert back to ROS image format
        processed_msg = self.bridge.cv2_to_imgmsg(processed_image, encoding='bgr8')

        # Publish processed image
        self.processed_pub.publish(processed_msg)

    def process_image(self, image):
        # Apply GPU-accelerated image processing
        # This could include:
        # - Camera distortion correction
        # - Feature detection
        # - Image enhancement
        # - Preprocessing for VSLAM

        # Example: Apply basic preprocessing
        gray = cv2.cvtColor(image, cv2.COLOR_BGR2GRAY)

        # Apply Gaussian blur to reduce noise
        blurred = cv2.GaussianBlur(gray, (5, 5), 0)

        return cv2.cvtColor(blurred, cv2.COLOR_GRAY2BGR)
```

## Visual SLAM Fundamentals

### SLAM Problem Definition

The SLAM problem can be defined as estimating the robot's trajectory and building a map of the environment simultaneously:

```
State estimation: x_t = f(x_{t-1}, u_t, z_t)
```

Where:
- `x_t`: Robot pose and map at time t
- `u_t`: Control inputs
- `z_t`: Sensor observations

### VSLAM Pipeline

The typical VSLAM pipeline consists of:

1. **Feature Detection and Extraction**: Identify key points in images
2. **Feature Matching**: Match features between consecutive frames
3. **Pose Estimation**: Estimate camera motion between frames
4. **Map Building**: Create and maintain a map of landmarks
5. **Loop Closure**: Detect when the robot revisits a location
6. **Optimization**: Optimize the map and trajectory

### Feature Detection with Isaac ROS

```python
import rclpy
from rclpy.node import Node
from sensor_msgs.msg import Image
from vision_msgs.msg import Detection2DArray, Point2D
from cv_bridge import CvBridge
import cv2
import numpy as np

class IsaacROSFeatureDetector(Node):
    def __init__(self):
        super().__init__('isaac_ros_feature_detector')

        self.image_sub = self.create_subscription(
            Image,
            '/camera/image_raw',
            self.image_callback,
            10
        )

        self.features_pub = self.create_publisher(
            Detection2DArray,
            '/camera/features',
            10
        )

        self.bridge = CvBridge()

        # Initialize feature detector (using GPU-accelerated methods when possible)
        self.detector = cv2.SIFT_create()

    def image_callback(self, msg):
        # Convert ROS image to OpenCV format
        cv_image = self.bridge.imgmsg_to_cv2(msg, desired_encoding='mono8')

        # Detect features
        keypoints, descriptors = self.detector.detectAndCompute(cv_image, None)

        # Create feature message
        features_msg = self.create_feature_message(keypoints, msg.header)

        # Publish features
        self.features_pub.publish(features_msg)

    def create_feature_message(self, keypoints, header):
        features_msg = Detection2DArray()
        features_msg.header = header

        for kp in keypoints:
            feature = Detection2D()
            feature.bbox.center.x = kp.pt[0]
            feature.bbox.center.y = kp.pt[1]
            feature.bbox.size_x = 10.0  # Placeholder size
            feature.bbox.size_y = 10.0

            features_msg.detections.append(feature)

        return features_msg
```

## Isaac ROS VSLAM Implementation

### Setting up Isaac ROS VSLAM

```bash
# Install Isaac ROS VSLAM packages
sudo apt update
sudo apt install ros-humble-isaac-ros-visual-slam
sudo apt install ros-humble-isaac-ros-gxf
sudo apt install ros-humble-isaac-ros-common
```

### Launch Configuration

```xml
<!-- Example launch file for Isaac ROS VSLAM -->
<launch>
  <!-- Launch camera driver -->
  <node pkg="camera_ros" exec="camera_node" name="camera_node">
    <param name="camera_info_url" value="package://my_robot/config/camera.yaml"/>
  </node>

  <!-- Launch Isaac ROS VSLAM -->
  <node pkg="isaac_ros_visual_slam" exec="visual_slam_node" name="visual_slam_node">
    <param name="enable_rectified_pose" value="true"/>
    <param name="rectified_base_frame" value="camera_link"/>
    <param name="enable_fisheye_distortion" value="false"/>
    <param name="map_frame" value="map"/>
    <param name="odom_frame" value="odom"/>
    <param name="base_frame" value="base_link"/>
    <param name="publish_odom_tf" value="true"/>
    <param name="publish_map_tf" value="true"/>
  </node>

  <!-- Launch visualization -->
  <node pkg="rviz2" exec="rviz2" name="rviz" args="-d $(find my_robot)/config/vslam.rviz"/>
</launch>
```

### VSLAM Node Implementation

```python
import rclpy
from rclpy.node import Node
from sensor_msgs.msg import Image, CameraInfo
from geometry_msgs.msg import PoseStamped, TransformStamped
from nav_msgs.msg import Odometry
from tf2_ros import TransformBroadcaster
import numpy as np
import cv2
from cv_bridge import CvBridge

class IsaacROSVisualSLAMNode(Node):
    def __init__(self):
        super().__init__('isaac_ros_vslam_node')

        # Initialize parameters
        self.declare_parameter('enable_rectified_pose', True)
        self.declare_parameter('rectified_base_frame', 'camera_link')
        self.declare_parameter('map_frame', 'map')
        self.declare_parameter('odom_frame', 'odom')
        self.declare_parameter('base_frame', 'base_link')

        self.enable_rectified_pose = self.get_parameter('enable_rectified_pose').value
        self.rectified_base_frame = self.get_parameter('rectified_base_frame').value
        self.map_frame = self.get_parameter('map_frame').value
        self.odom_frame = self.get_parameter('odom_frame').value
        self.base_frame = self.get_parameter('base_frame').value

        # Create subscribers
        self.left_image_sub = self.create_subscription(
            Image,
            '/camera/left/image_rect',
            self.left_image_callback,
            10
        )

        self.right_image_sub = self.create_subscription(
            Image,
            '/camera/right/image_rect',
            self.right_image_callback,
            10
        )

        self.camera_info_sub = self.create_subscription(
            CameraInfo,
            '/camera/camera_info',
            self.camera_info_callback,
            10
        )

        # Create publishers
        self.odom_pub = self.create_publisher(Odometry, '/visual_slam/odometry', 10)
        self.pose_pub = self.create_publisher(PoseStamped, '/visual_slam/pose', 10)

        # Initialize TF broadcaster
        self.tf_broadcaster = TransformBroadcaster(self)

        # Initialize VSLAM components
        self.bridge = CvBridge()
        self.vslam_system = self.initialize_vslam_system()

        # Camera parameters
        self.camera_matrix = None
        self.distortion_coeffs = None

        # Robot state
        self.current_pose = np.eye(4)  # 4x4 transformation matrix
        self.previous_features = None

    def initialize_vslam_system(self):
        """Initialize the VSLAM system"""
        # This would typically initialize a GPU-accelerated VSLAM algorithm
        # For example, using Isaac ROS's VisualSLAM components
        return {
            'keypoints': [],
            'descriptors': [],
            'map_points': [],
            'poses': []
        }

    def camera_info_callback(self, msg):
        """Process camera calibration information"""
        self.camera_matrix = np.array(msg.k).reshape(3, 3)
        self.distortion_coeffs = np.array(msg.d)

    def left_image_callback(self, msg):
        """Process left camera image for stereo VSLAM"""
        cv_image = self.bridge.imgmsg_to_cv2(msg, desired_encoding='mono8')

        # Process frame with VSLAM
        self.process_vslam_frame(cv_image, msg.header.stamp)

    def right_image_callback(self, msg):
        """Process right camera image for stereo VSLAM"""
        # For stereo VSLAM, process the right image as well
        cv_image = self.bridge.imgmsg_to_cv2(msg, desired_encoding='mono8')

        # Store for stereo processing
        self.right_image = cv_image

    def process_vslam_frame(self, image, timestamp):
        """Process a single frame through the VSLAM pipeline"""
        # 1. Feature detection and extraction
        keypoints, descriptors = self.extract_features(image)

        # 2. Feature matching with previous frame
        if self.previous_features is not None:
            matches = self.match_features(
                self.previous_features['keypoints'],
                keypoints,
                self.previous_features['descriptors'],
                descriptors
            )

            # 3. Estimate motion between frames
            if len(matches) >= 10:  # Minimum matches required
                transformation = self.estimate_motion(matches)

                # 4. Update robot pose
                self.update_pose(transformation)

                # 5. Publish odometry
                self.publish_odometry(timestamp)

                # 6. Broadcast TF transform
                self.broadcast_transform(timestamp)

        # Store current features for next iteration
        self.previous_features = {
            'keypoints': keypoints,
            'descriptors': descriptors
        }

    def extract_features(self, image):
        """Extract features from image using GPU-accelerated methods"""
        # Using ORB for feature extraction (can be replaced with more advanced methods)
        orb = cv2.ORB_create(nfeatures=2000)
        keypoints, descriptors = orb.detectAndCompute(image, None)

        if descriptors is not None:
            # Convert keypoints to array format
            kp_array = np.array([kp.pt for kp in keypoints])
            return kp_array, descriptors
        else:
            return np.array([]), np.array([])

    def match_features(self, prev_kp, curr_kp, prev_desc, curr_desc):
        """Match features between previous and current frames"""
        if len(prev_desc) == 0 or len(curr_desc) == 0:
            return []

        # Use FLANN matcher for efficient matching
        index_params = dict(algorithm=6, table_number=6, key_size=12, multi_probe_level=1)
        search_params = dict(checks=50)
        flann = cv2.FlannBasedMatcher(index_params, search_params)

        matches = flann.knnMatch(prev_desc, curr_desc, k=2)

        # Apply Lowe's ratio test
        good_matches = []
        for m, n in matches:
            if m.distance < 0.7 * n.distance:
                good_matches.append(m)

        return good_matches

    def estimate_motion(self, matches):
        """Estimate motion between frames using matched features"""
        if len(matches) < 10:
            return np.eye(4)  # Return identity if not enough matches

        # Get matched points
        prev_pts = np.float32([self.previous_features['keypoints'][m.queryIdx] for m in matches]).reshape(-1, 1, 2)
        curr_pts = np.float32([self.current_keypoints[m.trainIdx] for m in matches]).reshape(-1, 1, 2)

        # Estimate essential matrix
        E, mask = cv2.findEssentialMat(
            curr_pts, prev_pts,
            self.camera_matrix,
            threshold=1,
            prob=0.999
        )

        if E is not None:
            # Decompose essential matrix to get rotation and translation
            _, R, t, mask = cv2.recoverPose(E, curr_pts, prev_pts, self.camera_matrix)

            # Create transformation matrix
            transformation = np.eye(4)
            transformation[0:3, 0:3] = R
            transformation[0:3, 3] = t.flatten()

            return transformation

        return np.eye(4)

    def update_pose(self, transformation):
        """Update the current robot pose based on estimated motion"""
        self.current_pose = self.current_pose @ np.linalg.inv(transformation)

    def publish_odometry(self, timestamp):
        """Publish odometry information"""
        odom_msg = Odometry()
        odom_msg.header.stamp = timestamp
        odom_msg.header.frame_id = self.odom_frame
        odom_msg.child_frame_id = self.base_frame

        # Set pose from transformation matrix
        odom_msg.pose.pose.position.x = self.current_pose[0, 3]
        odom_msg.pose.pose.position.y = self.current_pose[1, 3]
        odom_msg.pose.pose.position.z = self.current_pose[2, 3]

        # Convert rotation matrix to quaternion
        quat = self.rotation_matrix_to_quaternion(self.current_pose[0:3, 0:3])
        odom_msg.pose.pose.orientation.x = quat[0]
        odom_msg.pose.pose.orientation.y = quat[1]
        odom_msg.pose.pose.orientation.z = quat[2]
        odom_msg.pose.pose.orientation.w = quat[3]

        # Set velocity (approximate from pose differences)
        # This is a simplified approach; in practice, you'd compute from pose differences

        self.odom_pub.publish(odom_msg)

        # Also publish pose stamped
        pose_msg = PoseStamped()
        pose_msg.header = odom_msg.header
        pose_msg.pose = odom_msg.pose.pose
        self.pose_pub.publish(pose_msg)

    def broadcast_transform(self, timestamp):
        """Broadcast TF transform"""
        t = TransformStamped()
        t.header.stamp = timestamp
        t.header.frame_id = self.odom_frame
        t.child_frame_id = self.base_frame

        t.transform.translation.x = self.current_pose[0, 3]
        t.transform.translation.y = self.current_pose[1, 3]
        t.transform.translation.z = self.current_pose[2, 3]

        quat = self.rotation_matrix_to_quaternion(self.current_pose[0:3, 0:3])
        t.transform.rotation.x = quat[0]
        t.transform.rotation.y = quat[1]
        t.transform.rotation.z = quat[2]
        t.transform.rotation.w = quat[3]

        self.tf_broadcaster.sendTransform(t)

    def rotation_matrix_to_quaternion(self, R):
        """Convert rotation matrix to quaternion"""
        # Using the algorithm from:
        # https://www.euclideanspace.com/maths/geometry/rotations/conversions/matrixToQuaternion/
        trace = np.trace(R)

        if trace > 0:
            s = np.sqrt(trace + 1.0) * 2  # S=4*qw
            qw = 0.25 * s
            qx = (R[2, 1] - R[1, 2]) / s
            qy = (R[0, 2] - R[2, 0]) / s
            qz = (R[1, 0] - R[0, 1]) / s
        elif (R[0, 0] > R[1, 1]) and (R[0, 0] > R[2, 2]):
            s = np.sqrt(1.0 + R[0, 0] - R[1, 1] - R[2, 2]) * 2  # S=4*qx
            qw = (R[2, 1] - R[1, 2]) / s
            qx = 0.25 * s
            qy = (R[0, 1] + R[1, 0]) / s
            qz = (R[0, 2] + R[2, 0]) / s
        elif R[1, 1] > R[2, 2]:
            s = np.sqrt(1.0 + R[1, 1] - R[0, 0] - R[2, 2]) * 2  # S=4*qy
            qw = (R[0, 2] - R[2, 0]) / s
            qx = (R[0, 1] + R[1, 0]) / s
            qy = 0.25 * s
            qz = (R[1, 2] + R[2, 1]) / s
        else:
            s = np.sqrt(1.0 + R[2, 2] - R[0, 0] - R[1, 1]) * 2  # S=4*qz
            qw = (R[1, 0] - R[0, 1]) / s
            qx = (R[0, 2] + R[2, 0]) / s
            qy = (R[1, 2] + R[2, 1]) / s
            qz = 0.25 * s

        return np.array([qx, qy, qz, qw])

def main(args=None):
    rclpy.init(args=args)
    vslam_node = IsaacROSVisualSLAMNode()

    try:
        rclpy.spin(vslam_node)
    except KeyboardInterrupt:
        pass
    finally:
        vslam_node.destroy_node()
        rclpy.shutdown()

if __name__ == '__main__':
    main()
```

## GPU-Accelerated VSLAM with Isaac ROS

### Leveraging CUDA for VSLAM

Isaac ROS leverages CUDA for GPU acceleration in VSLAM operations:

```python
import numpy as np
import cv2
from numba import cuda
import math

class GPUSLAMProcessor:
    def __init__(self):
        # Check for GPU availability
        self.gpu_available = cuda.is_available()
        if self.gpu_available:
            print(f"GPU available: {cuda.list_devices()}")
        else:
            print("GPU not available, falling back to CPU")

    def gpu_feature_matching(self, descriptors1, descriptors2):
        """Perform feature matching on GPU using CUDA"""
        if not self.gpu_available:
            # Fall back to CPU matching
            return self.cpu_feature_matching(descriptors1, descriptors2)

        # Transfer descriptors to GPU
        d_desc1 = cuda.to_device(descriptors1.astype(np.float32))
        d_desc2 = cuda.to_device(descriptors2.astype(np.float32))

        # Allocate result arrays on GPU
        h_matches = np.zeros((len(descriptors1), 2), dtype=np.int32)
        d_matches = cuda.to_device(h_matches)

        # Configure kernel launch parameters
        threads_per_block = 256
        blocks_per_grid = math.ceil(len(descriptors1) / threads_per_block)

        # Launch CUDA kernel for feature matching
        self.gpu_feature_matching_kernel[blocks_per_grid, threads_per_block](
            d_desc1, d_desc2, d_matches, len(descriptors1), len(descriptors2)
        )

        # Copy results back to host
        h_matches = d_matches.copy_to_host()

        return h_matches

    @cuda.jit
    def gpu_feature_matching_kernel(self, desc1, desc2, matches, n1, n2):
        """CUDA kernel for feature matching"""
        idx = cuda.grid(1)

        if idx < n1:
            best_match = -1
            best_distance = float('inf')
            second_best = float('inf')

            # Find best and second best matches for descriptor idx
            for i in range(n2):
                distance = 0.0
                # Compute descriptor distance (using sum of squared differences)
                for j in range(desc1.shape[1]):
                    diff = desc1[idx, j] - desc2[i, j]
                    distance += diff * diff

                if distance < best_distance:
                    second_best = best_distance
                    best_distance = distance
                    best_match = i
                elif distance < second_best:
                    second_best = distance

            # Apply Lowe's ratio test
            if best_distance < 0.7 * second_best:
                matches[idx, 0] = idx
                matches[idx, 1] = best_match
            else:
                matches[idx, 0] = -1
                matches[idx, 1] = -1
```

### Optimized Image Processing Pipeline

```python
import rclpy
from rclpy.node import Node
from sensor_msgs.msg import Image
from image_transport import ImageTransport
from cv_bridge import CvBridge
import numpy as np
import cv2
from cuda import cudart
import pycuda.driver as cuda
import pycuda.autoinit

class OptimizedVSLAMImageProcessor(Node):
    def __init__(self):
        super().__init__('optimized_vslam_image_processor')

        # Create subscribers for stereo images
        self.left_sub = self.create_subscription(Image, '/camera/left/image_raw', self.left_callback, 10)
        self.right_sub = self.create_subscription(Image, '/camera/right/image_raw', self.right_callback, 10)

        # Create publishers for processed images
        self.left_processed_pub = self.create_publisher(Image, '/camera/left/image_processed', 10)
        self.right_processed_pub = self.create_publisher(Image, '/camera/right/image_processed', 10)

        self.bridge = CvBridge()

        # Initialize CUDA context
        self.cuda_context = cudart.cudaFree(0)[0]  # Initialize CUDA

        # GPU memory buffers
        self.gpu_buffers_initialized = False
        self.left_gpu_buffer = None
        self.right_gpu_buffer = None

    def initialize_gpu_buffers(self, height, width):
        """Initialize GPU memory buffers for image processing"""
        if not self.gpu_buffers_initialized:
            # Allocate GPU memory for images
            image_size = height * width * 3  # Assuming RGB
            self.left_gpu_buffer = cuda.mem_alloc(image_size)
            self.right_gpu_buffer = cuda.mem_alloc(image_size)
            self.gpu_buffers_initialized = True

    def left_callback(self, msg):
        """Process left camera image with GPU acceleration"""
        cv_image = self.bridge.imgmsg_to_cv2(msg, desired_encoding='bgr8')

        if not self.gpu_buffers_initialized:
            self.initialize_gpu_buffers(cv_image.shape[0], cv_image.shape[1])

        # Transfer image to GPU
        cuda.memcpy_htod(self.left_gpu_buffer, cv_image)

        # Process image on GPU (e.g., distortion correction, feature extraction)
        processed_image = self.gpu_process_image(self.left_gpu_buffer, cv_image.shape)

        # Convert back to ROS message
        processed_msg = self.bridge.cv2_to_imgmsg(processed_image, encoding='bgr8')
        processed_msg.header = msg.header

        # Publish processed image
        self.left_processed_pub.publish(processed_msg)

    def right_callback(self, msg):
        """Process right camera image with GPU acceleration"""
        cv_image = self.bridge.imgmsg_to_cv2(msg, desired_encoding='bgr8')

        if not self.gpu_buffers_initialized:
            self.initialize_gpu_buffers(cv_image.shape[0], cv_image.shape[1])

        # Transfer image to GPU
        cuda.memcpy_htod(self.right_gpu_buffer, cv_image)

        # Process image on GPU
        processed_image = self.gpu_process_image(self.right_gpu_buffer, cv_image.shape)

        # Convert back to ROS message
        processed_msg = self.bridge.cv2_to_imgmsg(processed_image, encoding='bgr8')
        processed_msg.header = msg.header

        # Publish processed image
        self.right_processed_pub.publish(processed_msg)

    def gpu_process_image(self, gpu_buffer, image_shape):
        """Process image using GPU acceleration"""
        # This would typically call a CUDA kernel for image processing
        # For example: lens distortion correction, image rectification, etc.

        # For demonstration, we'll just return the original image
        # In practice, this would involve calling optimized CUDA kernels
        height, width, channels = image_shape
        processed_image = np.empty((height, width, channels), dtype=np.uint8)
        cuda.memcpy_dtoh(processed_image, gpu_buffer)

        return processed_image
```

## VSLAM for Humanoid Robots

### Humanoid-Specific VSLAM Considerations

Humanoid robots present unique challenges for VSLAM:

1. **Dynamic Motion**: Humanoid robots have complex, dynamic motion patterns
2. **Height Variation**: The robot's height changes during walking/running
3. **Occlusions**: Robot's own body parts can occlude the camera view
4. **Sensor Placement**: Cameras may be placed on moving parts (head)

### Multi-Camera VSLAM for Humanoid Robots

```python
import rclpy
from rclpy.node import Node
from sensor_msgs.msg import Image
from geometry_msgs.msg import PoseStamped
from tf2_ros import Buffer, TransformListener
import numpy as np

class HumanoidMultiCameraVSLAM(Node):
    def __init__(self):
        super().__init__('humanoid_multi_camera_vslam')

        # Create subscribers for multiple cameras
        self.head_camera_sub = self.create_subscription(
            Image, '/head_camera/image_rect', self.head_camera_callback, 10
        )
        self.chest_camera_sub = self.create_subscription(
            Image, '/chest_camera/image_rect', self.chest_camera_callback, 10
        )
        self.eye_camera_sub = self.create_subscription(
            Image, '/eye_camera/image_rect', self.eye_camera_callback, 10
        )

        # Publisher for fused pose
        self.fused_pose_pub = self.create_publisher(PoseStamped, '/humanoid_vslam/pose', 10)

        # TF buffer for camera poses
        self.tf_buffer = Buffer()
        self.tf_listener = TransformListener(self.tf_buffer, self)

        # Initialize VSLAM for each camera
        self.head_vslam = self.initialize_vslam_system()
        self.chest_vslam = self.initialize_vslam_system()
        self.eye_vslam = self.initialize_vslam_system()

        # Camera poses relative to base
        self.camera_poses = {}

        # Timer for fusing poses
        self.fuse_timer = self.create_timer(0.033, self.fuse_poses)  # 30 Hz

    def initialize_vslam_system(self):
        """Initialize a VSLAM system for a camera"""
        return {
            'keypoints': [],
            'descriptors': [],
            'local_map': [],
            'camera_pose': np.eye(4)  # Camera pose relative to world
        }

    def head_camera_callback(self, msg):
        """Process head camera image"""
        self.process_camera_vslam(msg, 'head_camera', self.head_vslam)

    def chest_camera_callback(self, msg):
        """Process chest camera image"""
        self.process_camera_vslam(msg, 'chest_camera', self.chest_vslam)

    def eye_camera_callback(self, msg):
        """Process eye camera image"""
        self.process_camera_vslam(msg, 'eye_camera', self.eye_vslam)

    def process_camera_vslam(self, msg, camera_name, vslam_system):
        """Process image for a specific camera's VSLAM"""
        # Get camera pose relative to robot base
        try:
            transform = self.tf_buffer.lookup_transform(
                'base_link', camera_name, msg.header.stamp, timeout=rclpy.duration.Duration(seconds=0.1)
            )
            camera_pose = self.transform_to_matrix(transform)
            self.camera_poses[camera_name] = camera_pose
        except Exception as e:
            self.get_logger().warn(f'Could not get transform for {camera_name}: {e}')
            return

        # Process VSLAM for this camera
        # (Implementation would be similar to single-camera VSLAM)
        # Extract features, match, estimate pose, etc.

    def transform_to_matrix(self, transform):
        """Convert TF transform to 4x4 transformation matrix"""
        t = transform.transform.translation
        r = transform.transform.rotation

        # Create transformation matrix
        matrix = np.eye(4)

        # Translation
        matrix[0, 3] = t.x
        matrix[1, 3] = t.y
        matrix[2, 3] = t.z

        # Rotation (quaternion to rotation matrix)
        qx, qy, qz, qw = r.x, r.y, r.z, r.w
        matrix[0, 0] = 1 - 2*qy*qy - 2*qz*qz
        matrix[0, 1] = 2*qx*qy - 2*qz*qw
        matrix[0, 2] = 2*qx*qz + 2*qy*qw
        matrix[1, 0] = 2*qx*qy + 2*qz*qw
        matrix[1, 1] = 1 - 2*qx*qx - 2*qz*qz
        matrix[1, 2] = 2*qy*qz - 2*qx*qw
        matrix[2, 0] = 2*qx*qz - 2*qy*qw
        matrix[2, 1] = 2*qy*qz + 2*qx*qw
        matrix[2, 2] = 1 - 2*qx*qx - 2*qy*qy

        return matrix

    def fuse_poses(self):
        """Fuse poses from multiple cameras"""
        # Get current poses from all cameras
        camera_poses = []
        weights = []

        for camera_name, vslam_system in [
            ('head_camera', self.head_vslam),
            ('chest_camera', self.chest_vslam),
            ('eye_camera', self.eye_vslam)
        ]:
            if camera_name in self.camera_poses:
                # Transform camera pose to world frame
                world_pose = vslam_system['camera_pose'] @ self.camera_poses[camera_name]
                camera_poses.append(world_pose)

                # Assign weight based on camera quality/tracking confidence
                weight = self.estimate_camera_weight(camera_name, vslam_system)
                weights.append(weight)

        if camera_poses:
            # Weighted average of poses
            fused_pose = self.weighted_pose_average(camera_poses, weights)

            # Publish fused pose
            pose_msg = self.pose_matrix_to_msg(fused_pose, self.get_clock().now().to_msg())
            self.fused_pose_pub.publish(pose_msg)

    def estimate_camera_weight(self, camera_name, vslam_system):
        """Estimate weight for a camera based on tracking quality"""
        # Weight based on number of tracked features, etc.
        num_features = len(vslam_system['keypoints'])
        if num_features > 50:
            return 1.0
        elif num_features > 20:
            return 0.7
        elif num_features > 10:
            return 0.3
        else:
            return 0.1

    def weighted_pose_average(self, poses, weights):
        """Compute weighted average of poses"""
        # Normalize weights
        total_weight = sum(weights)
        if total_weight == 0:
            return np.eye(4)

        weights = [w/total_weight for w in weights]

        # Average translation
        avg_translation = np.zeros(3)
        for i, pose in enumerate(poses):
            avg_translation += weights[i] * pose[0:3, 3]

        # Average rotation (using quaternion averaging)
        quats = [self.rotation_matrix_to_quaternion(pose[0:3, 0:3]) for pose in poses]
        avg_quat = self.average_quaternions(quats, weights)
        avg_rotation = self.quaternion_to_rotation_matrix(avg_quat)

        # Combine into transformation matrix
        result = np.eye(4)
        result[0:3, 3] = avg_translation
        result[0:3, 0:3] = avg_rotation

        return result

    def pose_matrix_to_msg(self, pose_matrix, stamp):
        """Convert pose matrix to ROS PoseStamped message"""
        pose_msg = PoseStamped()
        pose_msg.header.stamp = stamp
        pose_msg.header.frame_id = 'map'

        pose_msg.pose.position.x = pose_matrix[0, 3]
        pose_msg.pose.position.y = pose_matrix[1, 3]
        pose_msg.pose.position.z = pose_matrix[2, 3]

        quat = self.rotation_matrix_to_quaternion(pose_matrix[0:3, 0:3])
        pose_msg.pose.orientation.x = quat[0]
        pose_msg.pose.orientation.y = quat[1]
        pose_msg.pose.orientation.z = quat[2]
        pose_msg.pose.orientation.w = quat[3]

        return pose_msg
```

## Performance Optimization and Evaluation

### VSLAM Performance Metrics

```python
import numpy as np
import matplotlib.pyplot as plt
from scipy.spatial.transform import Rotation as R

class VSLAMPerformanceEvaluator:
    def __init__(self):
        self.poses = []
        self.gt_poses = []
        self.timestamps = []
        self.processing_times = []

    def add_estimated_pose(self, pose_matrix, timestamp):
        """Add estimated pose from VSLAM"""
        self.poses.append(pose_matrix)
        self.timestamps.append(timestamp)

    def add_ground_truth_pose(self, pose_matrix, timestamp):
        """Add ground truth pose for evaluation"""
        self.gt_poses.append(pose_matrix)

    def add_processing_time(self, time_ms):
        """Add processing time for performance evaluation"""
        self.processing_times.append(time_ms)

    def calculate_metrics(self):
        """Calculate VSLAM performance metrics"""
        if len(self.poses) != len(self.gt_poses):
            print("Warning: Different number of estimated and ground truth poses")
            return {}

        # Calculate trajectory error
        position_errors = []
        rotation_errors = []

        for est, gt in zip(self.poses, self.gt_poses):
            # Position error
            pos_err = np.linalg.norm(est[0:3, 3] - gt[0:3, 3])
            position_errors.append(pos_err)

            # Rotation error
            rot_err_matrix = est[0:3, 0:3] @ gt[0:3, 0:3].T
            rot_err_angle = R.from_matrix(rot_err_matrix).magnitude()
            rotation_errors.append(rot_err_angle)

        # Calculate metrics
        metrics = {
            'rmse_position': np.sqrt(np.mean(np.square(position_errors))),
            'mean_position_error': np.mean(position_errors),
            'max_position_error': np.max(position_errors),
            'rmse_rotation': np.sqrt(np.mean(np.square(rotation_errors))),
            'mean_rotation_error': np.mean(rotation_errors),
            'max_rotation_error': np.max(rotation_errors),
            'mean_processing_time': np.mean(self.processing_times),
            'std_processing_time': np.std(self.processing_times),
            'max_processing_time': np.max(self.processing_times)
        }

        return metrics

    def plot_results(self):
        """Plot VSLAM performance results"""
        if not self.poses or not self.gt_poses:
            return

        # Extract trajectory points
        est_x = [pose[0, 3] for pose in self.poses]
        est_y = [pose[1, 3] for pose in self.poses]
        gt_x = [pose[0, 3] for pose in self.gt_poses]
        gt_y = [pose[1, 3] for pose in self.gt_poses]

        fig, axes = plt.subplots(2, 2, figsize=(12, 10))

        # Plot trajectory
        axes[0, 0].plot(gt_x, gt_y, 'g-', label='Ground Truth', linewidth=2)
        axes[0, 0].plot(est_x, est_y, 'r-', label='Estimated', linewidth=2)
        axes[0, 0].set_title('Trajectory Comparison')
        axes[0, 0].set_xlabel('X (m)')
        axes[0, 0].set_ylabel('Y (m)')
        axes[0, 0].legend()
        axes[0, 0].grid(True)

        # Plot position errors over time
        position_errors = [np.linalg.norm(est[0:3, 3] - gt[0:3, 3])
                          for est, gt in zip(self.poses, self.gt_poses)]
        axes[0, 1].plot(position_errors, 'b-')
        axes[0, 1].set_title('Position Error Over Time')
        axes[0, 1].set_xlabel('Frame')
        axes[0, 1].set_ylabel('Position Error (m)')
        axes[0, 1].grid(True)

        # Plot processing times
        axes[1, 0].plot(self.processing_times, 'b-')
        axes[1, 0].set_title('Processing Time')
        axes[1, 0].set_xlabel('Frame')
        axes[1, 0].set_ylabel('Time (ms)')
        axes[1, 0].grid(True)

        # Plot histogram of position errors
        axes[1, 1].hist(position_errors, bins=50, density=True)
        axes[1, 1].set_title('Position Error Distribution')
        axes[1, 1].set_xlabel('Position Error (m)')
        axes[1, 1].set_ylabel('Density')
        axes[1, 1].grid(True)

        plt.tight_layout()
        plt.show()
```

## Troubleshooting VSLAM Systems

### Common Issues and Solutions

1. **Tracking Loss**: When VSLAM loses track of features
   - Solution: Use wider FOV cameras, improve lighting, add more distinctive features to environment

2. **Drift**: Accumulated errors over time
   - Solution: Implement loop closure, use sensor fusion with IMU, optimize bundle adjustment

3. **Computational Overhead**: High processing requirements
   - Solution: Use GPU acceleration, optimize feature extraction, reduce processing frequency

4. **Scale Ambiguity**: Inability to determine absolute scale from monocular cameras
   - Solution: Use stereo cameras, incorporate IMU data, add scale constraints

### Debugging Tools

```python
class VSLAMDebugger:
    def __init__(self, node):
        self.node = node
        self.debug_publisher = node.create_publisher(Image, '/vslam/debug_image', 10)
        self.bridge = CvBridge()

    def visualize_features(self, image, keypoints, matches=None):
        """Visualize detected features on image"""
        # Draw keypoints
        vis_image = cv2.drawKeypoints(image, [cv2.KeyPoint(float(kp[0]), float(kp[1]), 10)
                                            for kp in keypoints], None, color=(0, 255, 0))

        if matches:
            # Draw matches if provided
            # This would require converting the match format appropriately
            pass

        # Publish debug image
        debug_msg = self.bridge.cv2_to_imgmsg(vis_image, encoding='bgr8')
        self.debug_publisher.publish(debug_msg)

    def log_tracking_status(self, num_features, tracking_confidence):
        """Log tracking status for debugging"""
        if num_features < 50:
            self.node.get_logger().warn(f'Low number of features: {num_features}')
        if tracking_confidence < 0.5:
            self.node.get_logger().warn(f'Low tracking confidence: {tracking_confidence}')
```

## Best Practices for Isaac ROS VSLAM

### 1. Hardware Considerations

- Use cameras with appropriate specifications for your application
- Ensure sufficient GPU memory and compute capability
- Consider power consumption for humanoid robot platforms

### 2. Parameter Tuning

- Adjust feature detection parameters based on environment
- Tune tracking thresholds based on robot speed and environment complexity
- Optimize processing frequency based on computational constraints

### 3. Integration Strategies

- Implement proper sensor calibration procedures
- Use appropriate coordinate frame conventions
- Ensure robust TF tree setup

## Exercises

1. **VSLAM Implementation Exercise**: Implement a basic VSLAM system using Isaac ROS components that can process stereo camera data and estimate robot trajectory.

2. **GPU Acceleration Exercise**: Modify the VSLAM implementation to leverage GPU acceleration for feature matching and pose estimation.

3. **Multi-Camera Fusion Exercise**: Create a VSLAM system that fuses data from multiple cameras on a humanoid robot to improve localization accuracy.

## Summary

This chapter covered Isaac ROS and Visual SLAM for humanoid robotics. We explored the architecture of Isaac ROS, the fundamentals of VSLAM, GPU-accelerated processing techniques, and humanoid-specific considerations. Isaac ROS provides powerful tools for implementing high-performance VSLAM systems that can enable autonomous navigation for humanoid robots.

## Further Reading

- [Isaac ROS Documentation](https://nvidia-isaac-ros.github.io/repositories_and_packages/isaac_ros_visual_slam/index.html)
- [Visual SLAM: Why, How, and Where Next?](https://arxiv.org/abs/2107.02960)
- [GPU-Accelerated Computer Vision with CUDA](https://developer.nvidia.com/blog/gpu-accelerated-computer-vision-cuda/)
- [ROS 2 Navigation: From ROS 1 to ROS 2](https://navigation.ros.org/)

---

**Previous Chapter**: [Isaac Sim & Synthetic Data](./isaac-sim.md)
**Next Chapter**: [Navigation with Nav2](./nav2-navigation.md)