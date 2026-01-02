# Vehicle Odometry ROS demo (Monza Circuit)

## Overview
This project is a **ROS (Robot Operating System)** demo developed for Vehicle Odometry and Sector-Timing use cases. It implements odometry estimation and race sector analysis for an autonomous vehicle driving on the Monza circuit.

The system processes raw sensor data (GPS and vehicle telemetry) to:
1.  Compute Dead Reckoning odometry using the Ackermann steering model.
2.  Compute Absolute odometry by converting GPS coordinates to a local Cartesian ENU frame.
3.  Calculate lap sector times and mean speeds by detecting when the vehicle crosses specific geofenced lines on the track.

## Project Structure
The package contains three main nodes managed by a single launch file:

* **`odometer`**: Estimates pose x, y, theta using wheel speed and steering angle (Ackermann bicycle model).
* **`gps_odometer`**: Converts Latitude/Longitude/Altitude to a local metric frame (ENU) relative to a reference point.
* **`sector_times`**: Monitors vehicle position to detect sector transitions and publishes timing/speed stats.

## Dependencies
* **ROS** (Tested on Noetic/Melodic)
* **C++ 11/14**
* **Standard ROS msgs**: `geometry_msgs`, `nav_msgs`, `sensor_msgs`, `tf`

## Installation

1.  **Clone the repository** into your catkin workspace `src` folder:
    ```bash
    cd ~/catkin_ws/src
    git clone https://github.com/cienzman/vehicle-odometry-ROS-demo.git
    ```

2.  **Build the package**:
    ```bash
    cd ~/catkin_ws
    catkin_make
    source devel/setup.bash
    ```

## Usage

### 1. Play the Data
You need the provided ROS bag file containing the vehicle data. Run it in a separate terminal with the `--clock` flag:
    ```rosbag play --clock project.bag
    ```

### 2. Launch the Project
The entire system (nodes + RViz visualization) is started via a single launch file:
    ```roslaunch first_project launch.launch```

This command will:
* Start the `odometer` node.
* Start the `gps_odometer` node.
* Start the `sector_times` node.
* Open **RViz** with the pre-configured `config.rviz` to visualize the paths (Blue = Dead Reckoning, Red = GPS).

## Nodes & Parameters

### 1. Odometer Node (`odometer.cpp`)
Calculates the vehicle's position using a kinematic bicycle model.
* **Subscribes to:** `/speedsteer` (`geometry_msgs/PointStamped`)
    * `y`: speed (km/h)
    * `x`: steering angle (degrees)
* **Publishes:** `/odom` (`nav_msgs/Odometry`)
* **Parameters:**
    * `steer`: Steering ratio factor (Default: ~32.0)
    * `steering_bias`: Offset to correct steering drift.
    * `initial_theta`: Initial heading of the car on the track.

### 2. GPS Odometer Node (`gps_odometer.cpp`)
Converts WGS84 coordinates to Cartesian ECEF and then to a local ENU (East-North-Up) frame.
* **Subscribes to:** `/swiftnav/front/gps_pose` (`sensor_msgs/NavSatFix`)
* **Publishes:** `/gps_odom` (`nav_msgs/Odometry`)
* **Parameters (Reference Point):**
    * `lat_r`, `lon_r`, `alt_r`: Coordinates of the map origin.
    * *Note: If parameters are not provided, the node uses the first valid GPS reading as the origin.*

### 3. Sector Times Node (`sector_times.cpp`)
Detects sector crossings using geometric intersection algorithms.
* **Subscribes to:** `/speedsteer` and `/swiftnav/front/gps_pose`
* **Publishes:** `/sector_times` (Custom Message)
* **Custom Message Structure (`sector_times.msg`):**
    * `int32 current_sector`: ID of the entered sector (1, 2, or 3).
    * `float32 current_sector_time`: Duration spent in the previous sector.
    * `float32 current_sector_mean_speed`: Average speed in the previous sector.

## Visual Results
The launch file opens RViz. You should see two paths tracing the Monza circuit:
* **Blue Path (Odom):** The path estimated purely by wheel encoders and steering. It may drift over time.
* **Red Path (GPS):** The absolute position of the vehicle.





