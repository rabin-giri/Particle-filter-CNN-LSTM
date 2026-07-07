# Particle Filter Enhanced by CNN-LSTM For Indoor Robot Localization

[![ROS2](https://img.shields.io/badge/ROS2-Supported-blue.svg)](https://docs.ros.org/en/foxy/index.html)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

## Overview
This repository contains the official open-source ROS2 workspace for the research framework: **"Particle Filter Enhanced by CNN-LSTM For Localization of Autonomous Robot in Indoor Environment"**. 

Particle filters play a crucial role in indoor localization where absolute position sensors like GPS are unavailable. However, traditional setups face critical failure modes including poor particle initialization, particle degeneracy, and an inability to recover autonomously from the "kidnapped robot" problem. This project presents a novel hybrid approach that integrates Adaptive Monte Carlo Localization (AMCL) with a deep time-series regression network (CNN-LSTM) to resolve these foundational limits.

By tracking AMCL's state through the particle cloud variance and weights, the CNN-LSTM node dynamically recognizes localization degradation. It automatically infers global coordinates from stream sequences and re-injects localized positions into the filter via the `/initialpose` topic, guaranteeing robust tracking without human intervention.

## Key Features
* [**Hybrid Localization Engine:** Fuses Recursive Bayesian Estimation from AMCL with a sequential vision processing node.
* [**Autonomous Kidnapped Robot Recovery:** Continually monitors tracking stability; if particle weights drop or dispersion escalates, the CNN-LSTM re-initializes the system state.
* **Resource Efficiency Optimization:** Re-initialization allows AMCL to bypass high-density starting particle configurations. The filter safely initializes with just 533 particles instead of 2000, lowering computational strain.
* **Universal Regression Formulation:** Modeled as a continuous regression workflow rather than environment-specific grid classification, supporting universal deployment structures.

---

## Workspace Structure
```text
.
└── Particle-filter-CNN-LSTM
    ├── cnn_lstm_localization     # ROS2 Deep Learning localization package
    │   ├── cnn_lstm_localization
    │   │   ├── main.py           # Production hardware execution script
    │   │   └── main_sim.py       # Gazebo simulation handling script
    │   ├── package.xml
    │   ├── setup.cfg
    │   └── setup.py
    ├── data_collect              # Automation nodes for training dataset synthesis
    │   ├── data.csv              # Synchronization records for tracking images
    │   ├── data_pos.csv          # Stored pose coordinates
    │   ├── main.py               # Dataset automation recorder script
    │   └── package.xml
    ├── diffdrive_arduino         # ros2_control C++ hardware interface
    │   ├── CMakeLists.txt
    │   ├── bringup/launch        # Launch profiles for structural hardware bringup
    │   ├── description/urdf      # Xacro configs mapping out hardware transforms
    │   └── hardware              # Serial communications mapping directly to Arduino
    ├── my_bot / nav_bot          # Robot definition & simulation setup packages
    │   ├── description           # Sensor configurations (LIDAR, Camera, Odometry)
    │   ├── launch                # Simulation activation and system orchestrators
    │   └── worlds                # Nine-room Gazebo simulation testing environments
    ├── my_joy                    # Joystick driver mapping for manual exploration
    └── maps                      # 2D occupancy grids built via SLAM toolbox
```

---

## Network Architecture & Methodology
1. **Feature Extraction:** Standard 224x224x3 color image sequences [cite: 133, 154] [cite_start]are routed through a pre-trained **VGG16** network's fully connected (`fc1`) layer to derive a dense 4096-dimensional output vector[cite: 170].
2. **Temporal Processing:** A sliding queue window of size 15 handles the historical steps, feeding stacked **LSTM layers** (structured with 64, 128, and 64 hidden units)[cite: 171].
3. *Continuous Regression Output:** Predicts 3 continuous degrees of freedom representing the precise state ($x$, $y$, and $yaw$).

The network executes tracking monitoring using linear activations, optimizing via the Adam optimizer against a Mean Squared Error (MSE) loss standard.

---

## Performance Metrics
Experimental hardware and simulation data confirms substantial tracking accuracy gains:
* **CNN-LSTM Accuracy:** Reaches a validation accuracy of 91.17% (MSE loss: 0.0362) in simulation , scaling up to 94.45% accuracy (MSE loss: 0.0113) when evaluated on physical hardware due to real-world geometric visual richness.
* **Resource Savings:** Incorporating the neural node drops the average running active particle baseline across tracking loops from 1279 down to 966.
* **Error Mitigation:** Effectively caps total cumulative error build-up over runtime during environments containing dynamic obstacle adjustments compared to standalone AMCL.

---

## Getting Started

### Prerequisites
* **OS:** Ubuntu 24.04 or 26.04 LTS
* **Middleware:** ROS2 (Foxy or Humble setup desktop complete distribution)
**Simulation:** Gazebo Physics Engine 
* **Python Dependencies:**
  ```bash
  pip install tensorflow keras opencv-python pandas
  ```

### Installation & Compilation
1. Create a workspace and clone this project repository inside the source folder:
   ```bash
   mkdir -p ~/hybrid_nav_ws/src
   cd ~/hybrid_nav_ws/src
   git clone [https://github.com/your-username/Particle-filter-CNN-LSTM.git](https://github.com/your-username/Particle-filter-CNN-LSTM.git)
   ```
2. Navigate back to the root workspace directory and install required system references via `rosdep`:
   ```bash
   cd ~/hybrid_nav_ws
   rosdep install --from-paths src --ignore-src -r -y
   ```
3. Compile the workspace packages using `colcon`:
   ```bash
   colcon build --symlink-install
   ```

---

## Execution Guide

### 1. Launching Simulation Testing
To open up the interconnected 9-room test environment inside Gazebo  alongside structural state publishers:
```bash
source install/setup.bash
ros2 launch nav_bot launch_sim_launch.py world:=house.world
```

### 2. Autonomous Dataset Collection
To automatically build structural training logs pairing system cameras with safe ground-truth poses from AMCL:
```bash
source install/setup.bash
ros2 run data_collect main.py
```
*Note: Use the joystick package `ros2 launch my_joy joy_launch.py` to manually pilot your robot into unmapped zones to ensure full visual sequence diversity.*

### 3. Deploying Physical Hardware Interfaces
To initialize low-level runtime control loop nodes when moving from simulation over onto physical differential robots:
```bash
source install/setup.bash
ros2 launch diffdrive_arduino diffbot.launch.py
```

### 4. Running the Main Hybrid Localization Loop
To enable the background supervisor node that tracks particle stability metrics and feeds re-initialization coordinates to AMCL:
```bash
source install/setup.bash
ros2 run cnn_lstm_localization main.py
```

---

## Citation
If this hybrid architecture or dataset generation logic assists your academic research, please cite our underlying project work:

```bibtex
@article{giri2024particle,
  title={Particle Filter Enhanced by CNN-LSTM For Localization of Autonomous Robot in Indoor Environment},
  author={Giri, Rabin and Sah, Anand Kumar and Satyal, Sanjivan},
  journal={Department of Electronics and Computer, Pulchowk Campus},
  year={2024}
}
```
