# AEGIS
Autonomous Electronic Guardian Interception System

# 1. Project Overview

AEGIS (**A**utonomous **E**lectronic **G**uardian **I**ntelligence **S**ystem) is a portable, wearable drone-detection and tracking system designed to provide real-time awareness of nearby unmanned aerial vehicles (UAVs).

The goal is to develop a low-cost, head-mountable device capable of detecting, identifying, and continuously tracking drones using computer vision and inertial sensor data.

Unlike stationary drone-detection systems, AEGIS is designed to remain mobile and compensate for the wearer's head movement, allowing it to maintain visual tracking of a drone as the user moves.

### Primary Objectives

- Detect drones using a live camera feed
- Identify and track drones across consecutive frames
- Estimate drone position, direction, velocity, and trajectory
- Compensate for head movement using inertial measurement data
- Automatically orient a motorized pan/tilt camera assembly toward detected drones
- Eventually integrate multiple sensing modalities to improve detection reliability
- Operate as a self-contained, portable system using a Raspberry Pi 5

Initial development focus: Build a reliable passive drone-detection and tracking prototype before expanding into more advanced capabilities.

# 2. Current Hardware

AEGIS is built around a Raspberry Pi 5, which serves as the central processing and control unit. All software, sensor communication, computer vision, and hardware control will run directly on the Pi.

### Hardware Architecture

The Raspberry Pi 5 acts as the central controller.

- Camera: Connects through the CSI camera interface and continuously captures image frames.
- IMU: Connects through I2C and provides orientation and motion measurements.
- Gimbal: Uses two servos to control horizontal and vertical camera movement.
- Power: The power bank supplies the system. Servo power is distributed separately from the Pi's GPIO power circuitry, with a shared ground.
- AI Acceleration: A Raspberry Pi AI HAT+ 2 may be integrated later to accelerate supported machine-learning inference workloads.

# 3. Software Stack

The software architecture uses ROS 2 to organize AEGIS into modular, independently testable components.

### Operating System and Middleware

- Ubuntu 24.04 64-bit
- ROS 2 Kaiju
- Python 3
- C++ for performance-critical components when necessary

### Computer Vision and Machine Learning

| Technology | Purpose                                                |
| ---------- | ------------------------------------------------------ |
| OpenCV     | Camera capture, image processing, and computer vision. |
| PyTorch    | Model development and training.                        |
| HailoRT    | AI inference using the future AI HAT+ 2.               |
| NumPy      | Numerical computations.                                |
| ByteTrack  | Multi-object tracking.                                 |
| FilterPy   | Optional custom Kalman filtering.                      |

### Robotics and Sensor Integration

| Technology       | Purpose                                   |
| ---------------- | ----------------------------------------- |
| ROS 2 `rclpy`    | Python-based ROS nodes                    |
| TF2              | Coordinate transformations                |
| `sensor_msgs`    | Sensor and IMU communication              |
| `geometry_msgs`  | Position, orientation, and velocity data  |
| Adafruit BNO08x  | BNO085 communication                      |
| Adafruit PCA9685 | Servo PWM control                         |

### Development Tools

- VS Code — primary development environment
- Git and GitHub — version control
- RViz2 — visualization of sensor data and coordinate frames
- rosbag2 — recording and replaying ROS data
- rqt — ROS debugging and visualization
- pytest — automated testing
- Jupyter — experimentation and data analysis

# 4. ROS 2 Architecture

AEGIS will use ROS 2 to separate its functionality into independent packages. Each package will handle a specific subsystem and communicate with other packages through ROS topics, services, and actions.

### Proposed Workspace

```
aegis_ws/
└── src/
    ├── aegis_camera/
    │   └── Camera capture and image publishing
    │
    ├── aegis_detection/
    │   └── Drone detection and AI inference
    │
    ├── aegis_tracking/
    │   └── Object tracking and motion estimation
    │
    ├── aegis_imu/
    │   └── BNO085 data acquisition
    │
    ├── aegis_fusion/
    │   └── Sensor fusion and coordinate transformations
    │
    ├── aegis_prediction/
    │   └── Drone trajectory prediction
    │
    ├── aegis_gimbal/
    │   └── Pan/tilt servo control
    │
    └── aegis_bringup/
        └── Launch files and system configuration
```

### Package Responsibilities

`aegis_camera`

- Initialize and manage the CSI camera
- Capture frames using an appropriate Raspberry Pi camera interface and OpenCV
- Publish timestamped image messages
- Manage camera configuration and frame rate

`aegis_detection`

- Receive camera frames
- Run the drone detection model
- Identify potential drone bounding boxes and confidence scores
- Publish detection results

`aegis_tracking`

- Associate detections across consecutive frames
- Maintain persistent tracking IDs
- Estimate image-space position and movement
- Handle temporary detection loss and reacquisition

`aegis_imu`

- Communicate with the BNO085
- Read orientation and motion measurements
- Publish timestamped IMU data

`aegis_fusion`

- Combine visual tracking and inertial measurements
- Account for camera and head movement
- Transform measurements between coordinate frames

`aegis_prediction`

- Estimate drone movement and velocity
- Predict short-term future positions
- Provide target information for camera control

`aegis_gimbal`

- Control the pan and tilt servos
- Receive target positions
- Calculate required camera adjustments
- Implement feedback-based position control where necessary

`aegis_bringup`

- Provide ROS launch files
- Configure parameters
- Start and coordinate system components
- Manage system-wide configuration

### Intended Data Flow

1. The camera node captures live video
2. The detection node identifies potential drones
3. The tracking node associates detections across frames
4. The IMU node publishes orientation and motion data
5. The fusion node combines visual and inertial information
6. The prediction node estimates drone movement
7. The gimbal node receives tracking targets and adjusts the camera

### Coordinate Frames

TF2 should be used to maintain consistent transformations between the system's physical components.

Proposed coordinate frames:

```
head_link
└── gimbal_base
    └── camera_link
        └── camera_optical_frame

imu_link
```

The exact relationships will depend on sensor placement and gimbal feedback.

The system must distinguish between head orientation and camera orientation. Camera movement caused by the gimbal should not be confused with movement caused by the wearer.

# 6. Development Roadmap

Development should proceed incrementally, with each subsystem tested independently before integration.

### Phase 1 — Foundation and Environment Setup ( <span style="color:#00FF00">CURRENT</span> )

- Set up the Raspberry Pi 5.
- Install the operating system and ROS 2.
- Configure the development environment.
- Create the GitHub repository and ROS 2 workspace.
- Verify connectivity with the camera, IMU, and servos.
- Establish the initial ROS 2 package structure.

Milestone: A working Raspberry Pi environment with ROS 2 and connected hardware.

### Phase 2 — Camera and Computer Vision

- Implement camera capture using OpenCV.
- Publish camera frames through ROS 2.
- Develop or adapt a drone detection model.
- Test detection using a suitable test drone.
- Measure detection accuracy, latency, and effective detection range.
- Evaluate performance under different lighting conditions and backgrounds.

Milestone: A live camera feed with reliable drone detection.

### Phase 3 — Object Tracking

- Implement multi-frame object tracking using ByteTrack.
- Associate detections with persistent tracking IDs.
- Estimate image-space position and movement.
- Handle temporary occlusions and missed detections.
- Measure tracking stability and latency.

Milestone: The system can consistently track a drone across consecutive frames.

### Phase 4 — IMU Integration

- Integrate the BNO085 through I2C.
- Publish orientation and motion data through ROS 2.
- Establish coordinate frames using TF2.
- Account for head movement when estimating drone movement.
- Test tracking while the camera or wearer moves.

Milestone: Visual tracking accounts for changes in camera orientation.

### Phase 5 — Gimbal Control

- Integrate the pan/tilt mechanism.
- Implement servo control through the PCA9685.
- Implement appropriate feedback-based positioning for the continuous-rotation pan servo.
- Control the MG90S tilt servo using commanded positions.
- Use tracking coordinates to calculate camera adjustments.
- Tune control response to reduce oscillation and tracking delay.

Milestone: The camera automatically follows a detected drone.

### Phase 6 — Sensor Fusion and Prediction

- Combine visual tracking and inertial measurements.
- Improve estimates of drone position and movement.
- Implement motion estimation and trajectory prediction.
- Evaluate performance under changing environmental conditions.
- Improve tracking when the drone temporarily leaves the camera's field of view.

Milestone: AEGIS can estimate and predict drone movement using integrated sensor information.

### Phase 7 — AI Acceleration and Optimization

- Integrate the Raspberry Pi AI HAT+ 2.
- Convert compatible models to Hailo's inference format.
- Implement HailoRT inference.
- Benchmark CPU versus accelerator performance.
- Optimize memory usage, frame rate, and inference latency.
- Improve power efficiency.

Milestone: Real-time detection and tracking with hardware-accelerated inference.

### Phase 8 — Mechanical Design and Wearable Prototype

- Design the head-mountable enclosure in CAD.
- Integrate the camera and gimbal assembly.
- Determine optimal placement for the Raspberry Pi, battery, and IMU.
- Design cable routing and power distribution.
- Manufacture and assemble the enclosure.
- Evaluate weight distribution, comfort, and stability.

Milestone: A functional wearable AEGIS prototype.

### Phase 9 — System Integration and Field Testing

- Integrate all ROS 2 packages.
- Create a unified system launch configuration.
- Conduct controlled outdoor testing.
- Evaluate detection range, accuracy, tracking stability, and latency.
- Measure battery life and operating temperature.
- Identify hardware and software bottlenecks.
- Document test results and iterate on the design.

Milestone: A fully integrated prototype with documented performance.
