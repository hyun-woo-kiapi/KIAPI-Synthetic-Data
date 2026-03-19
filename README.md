KIAPI End-to-End Dataset

KIAPI End-to-End Dataset for Autonomous Systems
------------------------------GIF-------------------------
The KIAPI End-to-End Dataset is a large-scale, multi-modal dataset specifically curated to advance the frontiers of End-to-End (E2E) Autonomous Driving and Physical AI. Additionally, it serves as the official foundational dataset for the “A1 Autonomous Car Challenge”, a premier E2E autonomous driving competition. (https://autonomouscar.or.kr/)
Collected across diverse geographical regions and complex urban road environments, this dataset provides a comprehensive sensor suite designed to facilitate the direct mapping of raw sensor inputs to vehicle control and behavioral planning. Our primary goal is to empower the global research community by providing high-quality, real-world data tailored for pure E2E learning paradigms.
Key Highlights-----------------
=Comprehensive Sensor Suite: Includes 360-degree multi-camera coverage, high-resolution 360-degree LiDAR point clouds, and synchronized GNSS/IMU telemetry.
=Future-Ready Scalability: The current sensor configuration is designed with extensibility in mind, with plans to integrate additional sensor modalities in future releases. Furthermore, the inclusion of perception-level labeling data is currently under consideration to support diverse auxiliary research tasks in the future.
=Versatile Applications: While strictly optimized for E2E control and Physical AI systems, the dataset's rich raw sensor logs and structural flexibility allow it to be utilized for various foundational research efforts in autonomous systems.
=Open Research Initiative: Developed and released to accelerate innovation in the field. We encourage researchers to explore, benchmark, and build upon this data to drive the industry forward.
Note on Sample Release: > The current distribution consists of Sample Data intended for format familiarization and initial testing. A more refined and comprehensive Official Dataset is scheduled for release in the near future.

Version History
Version	Notes
v0.1	Sample data for test

License
TBU (This dataset’s owner is KIAPI) ???
BSD???
Apache License???
사사표기 명확히 해라
사용문의는 우현주임 윤형석 팀장한테 해라
국가과제다 

Intented Usage
=End-to-End Autonomous Driving (Primary Focus): Developing, training, and benchmarking models that map raw multi-modal sensor inputs directly to future vehicle trajectories. Instead of low-level control commands, the dataset provides high-fidelity recorded trajectory logs to serve as ground truth for motion planning. 
=Physical AI & Embodied Intelligence: Advancing the broader field of Physical AI. The comprehensive 360-degree spatial and temporal data can be leveraged to train generalized foundation models that understand, navigate, and interact with complex physical environments, offering high transferability to mobile robotics and autonomous machinery.
=Multi-Sensor Fusion & Alignment: Utilizing the rigorously synchronized multi-camera, 360-degree LiDAR, and GNSS/IMU data for advanced research in sensor calibration, spatial-temporal alignment, and robust state estimation in dynamic urban scenes.
=Open-Ended Foundational Research: Beyond specific driving tasks, researchers are encouraged to utilize the raw, unadulterated sensor logs for exploratory studies in multi-modal scene representation, predictive modeling, and spatial awareness.


Data Format
------------------------------Image---------------------------
The KIAPI End-to-End Dataset employs a highly structured, relational database approach (PostgreSQL) to manage extensive metadata, seamlessly linked with external storage for raw multi-modal sensor files. This architecture is designed to handle large-scale time-series data efficiently for E2E trajectory planning.
1. Data Hierarchy
To facilitate streamlined training and evaluation, the continuous driving data is hierarchically segmented:
=Session: Represents a continuous, unbroken recording of a driving run, linking the specific vehicle, driver, and scenario.
=Clip: The fundamental unit for training and evaluation. Sessions are sliced into fixed time segments (e.g., 20 seconds). Each clip contains information regarding specific driving events and references the compressed raw sensor data files.
=Frame: The granular synchronization points within a clip. A frame aligns the disparate multi-modal sensor data (LiDAR, Camera, Radar) to a single reference timestamp.
2. Ego State & Trajectory Ground Truth (ego_pos)
=For E2E motion planning, the ego_pos table serves as the primary ground truth. Derived from high-precision GNSS/IMU sensors, it provides comprehensive vehicle kinematics.
=Trajectory Tracking: Includes relative coordinates (rel_x, rel_y, rel_z) measured from the starting position of each clip, enabling precise local trajectory generation.
=Kinematics & Dynamics: Provides 3D velocities (vx, vy, vz), accelerations (ax, ay, az), quaternion-based orientation, and path curvature (curvature).
3. Sensor Data Management
To optimize storage and access, the dataset separates raw sensor files from their metadata:
=Raw Data: The actual high-resolution images and point clouds are stored externally. The paths to these compressed files (e.g., clip_index_CAMERA_FRONT.zip, clip_index_LIDAR_TOP.zip) are cataloged at the Clip level.
=Sensor Metadata: The camera, lidar, and radar tables store frame-level metadata, including exact timestamps, channel information, and an is_key_frame flag to indicate exact synchronization points across different sensors.
4. Contextual Metadata
Understanding the environment and setup is crucial for Physical AI generalization. The dataset provides rich context through auxiliary tables:
=Vehicle: Contains detailed sensor configurations, including extrinsic calibration parameters (TF) and intrinsic specs, stored in a flexible JSON format (sensor_config).
=Driver: Includes behavioral context, such as driving style (e.g., Aggressive, Normal, Defensive) and years of experience.
=Scenario: Categorizes the environmental context, including city names, road names, and road types (e.g., Highway, City road).

Developer Toolkit
TBU
About Python API development toolkit or pip package (???)
Tiny examples for usage (???)


