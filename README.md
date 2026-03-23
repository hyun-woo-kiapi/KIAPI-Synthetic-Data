# KIAPI End-to-End Dataset

## KIAPI End-to-End Dataset for Autonomous Systems

![Driving_Sample_Image](assets/dataset_setup.jpg) 

The KIAPI End-to-End Dataset is a large-scale, multi-modal dataset specifically curated to advance the frontiers of End-to-End (E2E) Autonomous Driving and Physical AI. 


![AI_Challenge_Image](assets/A1_challenge.png)

Additionally, it serves as the official foundational dataset for the “2026 A1 Autonomous Car Challenge”, a premier E2E autonomous driving competition. (https://autonomouscar.or.kr/)

Collected across diverse geographical regions and complex urban road environments, this dataset provides a comprehensive sensor suite designed to facilitate the direct mapping of raw sensor inputs to vehicle control and behavioral planning. Our primary goal is to empower the global research community by providing high-quality, real-world data tailored for pure E2E learning paradigms.

### Key Highlights
* **Comprehensive Sensor Suite:** Includes 360-degree multi-camera coverage, high-resolution 360-degree LiDAR point clouds, and synchronized GNSS/IMU telemetry.
* **Future-Ready Scalability:** The current sensor configuration is designed with extensibility in mind, with plans to integrate additional sensor modalities in future releases. Furthermore, the inclusion of perception-level labeling data is currently under consideration to support diverse auxiliary research tasks in the future.
* **Versatile Applications:** While strictly optimized for E2E control and Physical AI systems, the dataset's rich raw sensor logs and structural flexibility allow it to be utilized for various foundational research efforts in autonomous systems.
* **Open Research Initiative:** Developed and released to accelerate innovation in the field. We encourage researchers to explore, benchmark, and build upon this data to drive the industry forward.

> **Note on Sample Release:** > The current distribution consists of Sample Data intended for format familiarization and initial testing. A more refined and comprehensive Official Dataset is scheduled for release in the near future.

---

### Version History

| Version | Notes |
| :--- | :--- |
| **v0.1** | Sample data for test |

---

## Intended Usage

* **End-to-End Autonomous Driving (Primary Focus):** Developing, training, and benchmarking models that map raw multi-modal sensor inputs directly to future vehicle trajectories. Instead of low-level control commands, the dataset provides high-fidelity recorded trajectory logs to serve as ground truth for motion planning.
* **Physical AI & Embodied Intelligence:** Advancing the broader field of Physical AI. The comprehensive 360-degree spatial and temporal data can be leveraged to train generalized foundation models that understand, navigate, and interact with complex physical environments, offering high transferability to mobile robotics and autonomous machinery.
* **Multi-Sensor Fusion & Alignment:** Utilizing the rigorously synchronized multi-camera, 360-degree LiDAR, and GNSS/IMU data for advanced research in sensor calibration, spatial-temporal alignment, and robust state estimation in dynamic urban scenes.
* **Open-Ended Foundational Research:** Beyond specific driving tasks, researchers are encouraged to utilize the raw, unadulterated sensor logs for exploratory studies in multi-modal scene representation, predictive modeling, and spatial awareness.

---

## Data Format

![Data_Architecture_Image](assets/dataset_architecture.jpg)

The KIAPI End-to-End Dataset employs a highly structured, relational database approach (PostgreSQL) to manage extensive metadata, seamlessly linked with external storage for raw multi-modal sensor files. This architecture is designed to handle large-scale time-series data efficiently for E2E trajectory planning.

### 1. Data Hierarchy
To facilitate streamlined training and evaluation, the continuous driving data is hierarchically segmented:

* **Session:** Represents a continuous, unbroken recording of a driving run.
  
  | Column Name | Description |
  | :--- | :--- |
  | `session_index` | Unique index of the session. |
  | `session_start_timestamp` | Timestamp when the session recording started. |
  | `session_finish_timestamp` | Timestamp when the session recording ended. |
  | `head_clip_index` | Index of the first clip within the session. |
  | `tail_clip_index` | Index of the last clip within the session. |
  | `note` | Additional details about the session. |

* **Clip:** The fundamental unit for training and evaluation. Sessions are sliced into fixed time segments (e.g., 20 seconds). Each clip contains information regarding specific driving events and references the compressed raw sensor data files.
  
  | Column Name | Description |
  | :--- | :--- |
  | `clip_index` | Unique index of the clip. |
  | `session_index` | Index of the session to which this clip belongs. |
  | `event` | Indicates the presence of a specific driving event within the clip. |
  | `clip_start_timestamp` | Start timestamp of the clip. |
  | `clip_finish_timestamp` | End timestamp of the clip. |
  | `tail_frame_index` | Index of the last frame within this clip. |
  | `lidar_compressed_filename` | Path and filename of the compressed LiDAR data (e.g., `clip_0000000/lidar_pcd/lidar.parquet`). |
  | `camera_compressed_filename` | Path and filename of the compressed Camera data (e.g., `clip_0000000/camera_video/camera.zip`). |
  | `radar_compressed_filename` | Path and filename of the compressed Radar data. |

* **Frame:** The granular reference point within a clip. Rather than relying on exact hardware-level synchronization, a frame is constructed based on a baseline timestamp. It aligns the disparate multi-modal sensor data (LiDAR, Camera, Radar) by strictly grouping the most recent data points acquired prior to this baseline, ensuring no future information is referenced.
  
  | Column Name | Description |
  | :--- | :--- |
  | `frame_index` | Unique internal index for the frame. |
  | `clip_index` | Index of the clip to which this frame belongs. |
  | `current_index` | The sequential index of the frame within its specific clip. |
  | `time_offset` | Delta time from the starting frame of the clip to this current frame. |

### 2. Ego State & Trajectory Ground Truth (`ego_pos`)
* For E2E motion planning, the `ego_pos` table serves as the primary ground truth. Derived from high-precision GNSS/IMU sensors, it provides comprehensive vehicle kinematics.
* **Trajectory Tracking:** Includes relative coordinates (`rel_x`, `rel_y`, `rel_z`) measured from the starting position of each clip, enabling precise local trajectory generation.
* **Kinematics & Dynamics:** Provides 3D velocities (`vx`, `vy`, `vz`), accelerations (`ax`, `ay`, `az`), quaternion-based orientation, and path curvature (`curvature`).

| Column Name | Description |
| :--- | :--- |
| `table_index` | Unique internal index. |
| `frame_index` | Index of the frame to which this data belongs. |
| `ego_pos_timestamp` | Acquisition timestamp of this specific data point. |
| `time_diff` | Delta time between the frame and this data point. |
| `rel_x`, `rel_y`, `rel_z` | Relative coordinates measured from the starting position of the clip's first frame. |
| `qw`, `qx`, `qy`, `qz` | Quaternion-based orientation values. |
| `vx`, `vy`, `vz` | 3D velocity vectors. |
| `ax`, `ay`, `az` | 3D acceleration vectors. |
| `curvature` | Path curvature data. |
| `is_key_frame` | Indicates if this data was selected as the closest preceding match to the frame's baseline timestamp. |

### 3. Sensor Data Management
To optimize storage and access, the dataset separates raw sensor files from their metadata:
* **Raw Data:** The actual high-resolution images and point clouds are stored externally. The paths to these compressed files (e.g., `clip_0000000/camera_video/camera.zip`, `clip_0000000/lidar_pcd/lidar.parquet`) are cataloged at the Clip level.
* **Sensor Metadata:** The `camera`, `lidar` tables store frame-level metadata, including exact acquisition timestamps, channel information, and an is_key_frame flag. This flag indicates which specific sensor data point was selected as the closest preceding match to the frame's baseline timestamp.

| Column Name | Description |
| :--- | :--- |
| `table_index` | Unique internal index. |
| `frame_index` | Index of the frame to which this data belongs. |
| `[sensor]_timestamp` | Acquisition timestamp of this specific sensor data. |
| `time_diff` | Delta time between the frame and this data point. |
| `channel` | The specific sensor channel (e.g., TOP, FRONT, FRONT_LEFT). |
| `sensor_config_index` | Internal index for sensor configuration. |
| `is_key_frame` | Indicates if this data was selected as the closest preceding match to the frame's baseline timestamp. |

<!-- ### 4. Contextual Metadata
Understanding the environment and setup is crucial for Physical AI generalization. The dataset provides rich context through auxiliary tables:
* **Vehicle:** Contains detailed sensor configurations, including extrinsic calibration parameters (TF) and intrinsic specs, stored in a flexible JSON format (`sensor_config`).
* **Driver:** Includes behavioral context, such as driving style (e.g., Aggressive, Normal, Defensive) and years of experience.
* **Scenario:** Categorizes the environmental context, including city names, road names, and road types (e.g., Highway, City road). -->

---

## Developer Toolkit

*(To Be Updated)*
* **Python API Toolkit & pip package:** Tools for easily querying the database and loading synchronized sensor data.
* **Quick Start Examples:** Simple code snippets demonstrating how to integrate the dataset into a PyTorch DataLoader.

---

## License & Acknowledgement

### License
The copyright and ownership of this dataset belong to the **Korea Intelligent Automotive Parts Promotion Institute (KIAPI)**.
*(Note: Specific open-source license terms such as Apache 2.0 or BSD will be updated shortly.)*

### Acknowledgement
This dataset was developed as part of a **National Research Project** funded by the South Korean government. 

If you use this dataset in your research, academic papers, or any published work, you **must explicitly include the following acknowledgment** to recognize the funding source and KIAPI's contribution. Please copy and paste the appropriate language version into your publication:

**[Korean]**
> 이 데이터셋은 2025년도 산업통상부 및 한국산업기술기획평가원(KEIT) 연구비 지원에 의한 연구로 생성된 데이터셋임 (RS-2025-25453109)

**[English]**
> This work was supported by the Industrial Strategic Technology Development Program (Project No. RS-2025-25453109, Development of advanced synthetic data collection and simulation technology for autonomous driving) funded by the Ministry of Trade, Industry & Resources (MOTIR, Korea)

### Contact
For usage inquiries, data access, or technical support, please contact the project coordinators:
* **HyeongSeok Yun** (Team Manager/Strategic Planning Division) - `[gudtjr0124@kiapi.or.kr]`
* **Hyun Woo** (Associate Researcher/Strategic Planning Division) - `[hyunwoo@kiapi.or.kr]`
