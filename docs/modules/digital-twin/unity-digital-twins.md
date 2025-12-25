---
sidebar_position: 2
title: "Digital Twins & Unity"
---

# Digital Twins & Unity for Humanoid Robotics

## Learning Objectives

By the end of this chapter, you will be able to:
- Understand the concept of digital twins in humanoid robotics
- Set up Unity as a digital twin environment for humanoid robots
- Integrate Unity with ROS 2 for bidirectional communication
- Create realistic humanoid robot models in Unity
- Implement sensor simulation and visualization in Unity
- Deploy Unity-based digital twins for robot development

## Introduction to Digital Twins in Robotics

A digital twin is a virtual representation of a physical system that enables real-time monitoring, simulation, and analysis. In humanoid robotics, digital twins serve as powerful tools for testing algorithms, validating control strategies, and performing virtual experiments before deployment to physical robots.

### Benefits of Digital Twins for Humanoid Robotics

- **Risk Reduction**: Test dangerous or complex behaviors in simulation first
- **Cost Efficiency**: Reduce the need for expensive hardware prototypes
- **Development Acceleration**: Parallel development of software and hardware
- **Data Generation**: Create large datasets for AI training
- **Validation**: Verify algorithms before real-world deployment
- **Optimization**: Fine-tune parameters in the virtual environment

## Unity as a Digital Twin Platform

Unity is a powerful 3D development platform that offers several advantages for creating digital twins of humanoid robots:

### Key Features of Unity for Robotics

1. **High-Quality Graphics**: Realistic rendering capabilities
2. **Physics Engine**: Built-in physics simulation with PhysX
3. **Cross-Platform Deployment**: Deploy to multiple platforms
4. **Asset Store**: Access to robotics-specific tools and assets
5. **Scripting**: C# scripting for custom behaviors
6. **Real-time Performance**: Optimized for real-time applications

### Unity Robotics Ecosystem

Unity provides several tools specifically for robotics:

- **Unity Robotics Hub**: Centralized access to robotics tools
- **ROS# (ROS Sharp)**: ROS communication for Unity
- **Unity ML-Agents**: Reinforcement learning framework
- **Unity Perception**: Synthetic data generation tools

## Setting Up Unity for Robotics

### Installing Unity Robotics Tools

```bash
# Install Unity Hub and Unity Editor
# Download from https://unity.com/

# Install ROS# (ROS Sharp) package
# Available on Unity Asset Store or GitHub
```

### Unity-ROS Bridge Setup

The Unity-ROS bridge enables communication between Unity and ROS 2:

```csharp
// Example Unity C# script for ROS communication
using UnityEngine;
using ROS2;

public class UnityRobotController : MonoBehaviour
{
    private ROS2UnityComponent ros2Unity;
    private ROS2Socket ros2Socket;
    private Publisher<std_msgs.msg.String> publisher;
    private Subscriber<std_msgs.msg.String> subscriber;

    void Start()
    {
        ros2Unity = GetComponent<ROS2UnityComponent>();
        ros2Unity.ROS2UnitySettings.DomainId = 0;
        ros2Unity.Init();

        ros2Socket = ros2Unity.Ros2Socket;

        // Initialize publisher and subscriber
        publisher = ros2Socket.advertise<std_msgs.msg.String>("/unity_data");
        subscriber = ros2Socket.subscribe<std_msgs.msg.String>("/robot_commands",
            ProcessRobotCommands);
    }

    void ProcessRobotCommands(std_msgs.msg.String msg)
    {
        // Handle incoming ROS messages
        Debug.Log("Received command: " + msg.Data);
    }

    void Update()
    {
        // Send data to ROS
        var msg = new std_msgs.msg.String();
        msg.Data = "Unity sensor data";
        publisher.Publish(msg);
    }
}
```

## Creating Humanoid Robot Models in Unity

### Importing Robot Models

Unity supports various 3D model formats that can be used for robot models:

```csharp
// Example of loading robot model in Unity
public class RobotModelLoader : MonoBehaviour
{
    public string robotModelPath;
    public GameObject robotModel;

    void Start()
    {
        // Load robot model from file
        robotModel = Resources.Load<GameObject>(robotModelPath);
        Instantiate(robotModel, transform.position, transform.rotation);
    }
}
```

### Joint Configuration for Humanoid Robots

Proper joint configuration is essential for realistic humanoid robot simulation:

```csharp
// Example of configuring humanoid joints in Unity
using UnityEngine;

public class HumanoidJointController : MonoBehaviour
{
    [System.Serializable]
    public class JointConfiguration
    {
        public string jointName;
        public Transform jointTransform;
        public float minAngle;
        public float maxAngle;
        public float speed;
    }

    public JointConfiguration[] joints;

    public void SetJointAngle(string jointName, float angle)
    {
        foreach (var joint in joints)
        {
            if (joint.jointName == jointName)
            {
                // Clamp angle to joint limits
                float clampedAngle = Mathf.Clamp(angle, joint.minAngle, joint.maxAngle);

                // Apply rotation to joint
                joint.jointTransform.localRotation =
                    Quaternion.Euler(0, 0, clampedAngle);
                break;
            }
        }
    }
}
```

## Unity-ROS Integration Patterns

### Sensor Simulation in Unity

Unity can simulate various robot sensors with realistic physics:

```csharp
// Example of camera sensor simulation in Unity
using UnityEngine;
using ROS2;
using sensor_msgs.msg;

public class UnityCameraSensor : MonoBehaviour
{
    public Camera sensorCamera;
    private ROS2Socket ros2Socket;
    private Publisher<sensor_msgs.msg.Image> imagePublisher;

    void Start()
    {
        ros2Socket = GetComponent<ROS2UnityComponent>().Ros2Socket;
        imagePublisher = ros2Socket.advertise<sensor_msgs.msg.Image>("/unity_camera/image_raw");
    }

    void Update()
    {
        // Capture image from Unity camera
        Texture2D imageTexture = CaptureCameraImage(sensorCamera);

        // Convert to ROS image format
        Image rosImage = ConvertToROSImage(imageTexture);

        // Publish to ROS topic
        imagePublisher.Publish(rosImage);
    }

    Texture2D CaptureCameraImage(Camera cam)
    {
        // Render texture setup
        RenderTexture renderTexture = cam.targetTexture;
        Texture2D image = new Texture2D(renderTexture.width, renderTexture.height,
            TextureFormat.RGB24, false);

        // Read pixels from render texture
        RenderTexture.active = renderTexture;
        image.ReadPixels(new Rect(0, 0, renderTexture.width, renderTexture.height), 0, 0);
        image.Apply();

        return image;
    }
}
```

### Control Interface Integration

```csharp
// Example of control interface for Unity robot
using UnityEngine;
using ROS2;
using geometry_msgs.msg;

public class UnityRobotController : MonoBehaviour
{
    public Transform robotBase;
    private ROS2Socket ros2Socket;
    private Subscriber<geometry_msgs.msg.Twist> cmdVelSubscriber;

    void Start()
    {
        ros2Socket = GetComponent<ROS2UnityComponent>().Ros2Socket;
        cmdVelSubscriber = ros2Socket.subscribe<geometry_msgs.msg.Twist>(
            "/cmd_vel", ProcessVelocityCommand);
    }

    void ProcessVelocityCommand(geometry_msgs.msg.Twist msg)
    {
        // Apply linear and angular velocities to robot
        Vector3 linearVelocity = new Vector3(
            (float)msg.linear.x,
            (float)msg.linear.y,
            (float)msg.linear.z
        );

        Vector3 angularVelocity = new Vector3(
            (float)msg.angular.x,
            (float)msg.angular.y,
            (float)msg.angular.z
        );

        // Update robot position based on velocities
        UpdateRobotMotion(linearVelocity, angularVelocity);
    }

    void UpdateRobotMotion(Vector3 linearVel, Vector3 angularVel)
    {
        // Apply motion to robot
        robotBase.Translate(linearVel * Time.deltaTime);
        robotBase.Rotate(angularVel * Time.deltaTime);
    }
}
```

## Advanced Unity Features for Digital Twins

### Physics Simulation in Unity

Unity's PhysX engine provides realistic physics simulation:

```csharp
// Example of physics-based robot simulation
using UnityEngine;

public class PhysicsRobotSimulator : MonoBehaviour
{
    public Rigidbody[] robotLinks;
    public ConfigurableJoint[] robotJoints;

    void FixedUpdate()
    {
        // Apply physics-based control
        ApplyPhysicsControl();
    }

    void ApplyPhysicsControl()
    {
        // Example: Apply torques to joints based on control signals
        for (int i = 0; i < robotJoints.Length; i++)
        {
            // Calculate target position based on control input
            float targetPosition = CalculateTargetPosition(i);

            // Apply target position to joint
            robotJoints[i].targetPosition = targetPosition;
        }
    }

    float CalculateTargetPosition(int jointIndex)
    {
        // Implement control algorithm
        return 0.0f; // Placeholder
    }
}
```

### Synthetic Data Generation

Unity's Perception package enables synthetic data generation for AI training:

```csharp
// Example of synthetic data generation setup
using UnityEngine;
using Unity.Perception.GroundTruth;

public class SyntheticDataGenerator : MonoBehaviour
{
    public GameObject robotModel;
    public Camera sensorCamera;
    public bool generateSemanticSegmentation = true;
    public bool generateDepth = true;

    void Start()
    {
        if (generateSemanticSegmentation)
        {
            // Add semantic segmentation capability
            var semanticSensor = sensorCamera.gameObject.AddComponent<SemanticSegmentationSensorComponent>();
        }

        if (generateDepth)
        {
            // Add depth sensor capability
            var depthSensor = sensorCamera.gameObject.AddComponent<DepthSensorComponent>();
        }
    }

    // Function to generate multiple scenarios
    public void GenerateScenario()
    {
        // Randomize environment
        RandomizeEnvironment();

        // Capture synthetic data
        CaptureSyntheticData();
    }

    void RandomizeEnvironment()
    {
        // Randomize lighting, textures, etc.
    }

    void CaptureSyntheticData()
    {
        // Capture images and annotations
    }
}
```

## Unity-ROS Bridge Best Practices

### Network Configuration

Proper network setup is crucial for reliable Unity-ROS communication:

```csharp
// Example of network configuration for Unity-ROS bridge
using UnityEngine;
using ROS2;

public class ROSNetworkConfig : MonoBehaviour
{
    public string rosMasterUri = "http://localhost:11311";
    public int domainId = 0;
    public bool useInsecureConnection = true;

    private ROS2UnityComponent ros2Unity;

    void Start()
    {
        ros2Unity = GetComponent<ROS2UnityComponent>();

        // Configure ROS settings
        ros2Unity.ROS2UnitySettings.DomainId = domainId;
        ros2Unity.ROS2UnitySettings.UseInsecureConnection = useInsecureConnection;

        ros2Unity.Init();
    }
}
```

### Performance Optimization

For real-time simulation, performance optimization is essential:

1. **LOD (Level of Detail)**: Use simplified models when far from camera
2. **Occlusion Culling**: Don't render objects not visible to camera
3. **Texture Compression**: Use appropriate texture formats
4. **Physics Optimization**: Limit physics calculations to necessary elements
5. **Script Optimization**: Optimize update loops and calculations

## Troubleshooting Unity-ROS Integration

### Common Issues and Solutions

1. **Connection Problems**:
   - Verify ROS master is running
   - Check network connectivity
   - Ensure correct domain ID is used

2. **Performance Issues**:
   - Reduce scene complexity
   - Optimize physics calculations
   - Use appropriate quality settings

3. **Synchronization Problems**:
   - Ensure consistent time sources
   - Implement proper message buffering
   - Handle message delays appropriately

### Debugging Strategies

```csharp
// Example of debugging Unity-ROS communication
using UnityEngine;
using ROS2;

public class UnityROSDiagnostics : MonoBehaviour
{
    private ROS2UnityComponent ros2Unity;
    private bool isConnected = false;

    void Start()
    {
        ros2Unity = GetComponent<ROS2UnityComponent>();
        ros2Unity.OnConnected += OnROSConnected;
        ros2Unity.OnDisconnected += OnROSDisconnected;
    }

    void OnROSConnected()
    {
        isConnected = true;
        Debug.Log("Connected to ROS");
    }

    void OnROSDisconnected()
    {
        isConnected = false;
        Debug.LogWarning("Disconnected from ROS");
    }

    void Update()
    {
        if (!isConnected)
        {
            Debug.LogError("Not connected to ROS - check connection settings");
        }
    }
}
```

## Exercises

1. **Unity Setup Exercise**: Install Unity and set up the ROS bridge. Create a simple robot model and establish communication with ROS 2.

2. **Digital Twin Exercise**: Create a digital twin of a simple humanoid robot in Unity with basic joint control and sensor simulation.

3. **Integration Exercise**: Implement a Unity-ROS bridge that allows controlling a virtual humanoid robot from ROS 2 nodes.

## Summary

This chapter covered the use of Unity as a digital twin platform for humanoid robotics. We explored how to set up Unity for robotics applications, create realistic robot models, integrate Unity with ROS 2, and implement sensor simulation. Digital twins provide a powerful environment for testing and validating humanoid robot algorithms before deployment to physical hardware.

## Further Reading

- [Unity Robotics Hub Documentation](https://github.com/Unity-Technologies/Unity-Robotics-Hub)
- [ROS# (ROS Sharp) GitHub](https://github.com/siemens/ros-sharp)
- [Unity ML-Agents Toolkit](https://github.com/Unity-Technologies/ml-agents)
- [Unity Perception Package](https://docs.unity3d.com/Packages/com.unity.perception@latest)

---

**Previous Chapter**: [Physics Simulation in Gazebo](./gazebo-physics.md)
**Next Chapter**: [Sensor Simulation](./sensor-simulation.md)