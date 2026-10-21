> **What this is:** the 29-step handout for my live workshop *Mastering ROS 2 — Build an Edge-Avoiding Robot from Scratch* (12 July 2025): workspace, URDF, sensors, Gazebo, ros2_control and the avoidance node, written so a beginner can follow it alone.
>
> Recording: [YouTube (2.5 h)](https://youtu.be/luZq317FUrE) · Finished code: [MechaMind-Labs/edge-avoiding-robot](https://github.com/MechaMind-Labs/edge-avoiding-robot) · TurtleSim basics: [Readme_Turtle.md](Readme_Turtle.md)
>
> Playlist: [Workshops and Live Labs](https://www.youtube.com/playlist?list=PLdMuvhY3uTKtMOVVnfKRV4qswTEHeXTy2)

# 🚀 ROS 2 Project Starter Template

#### A clean, well-structured ROS 2 simulation project from scratch, ideal for workshops, personal learning, or prototyping. This walks you through every step — workspace setup, URDF creation, Gazebo simulation, ROS 2 Control, and more.
---

## 🛠️ Build Your Own ROS2 Robot from Scratch – Workshop Starter Guide

Welcome to the **"Mastering ROS2 – Build an Edge-Avoiding Robot from Scratch"** workshop!
This guide will walk you through setting up your ROS2 package and building your first robot from **scratch**, step-by-step.

---

## 📁 1. Create Your Workspace

Open your terminal and run the following:

```bash
cd ~
mkdir -p bot_ws/src
cd bot_ws/src
```

---

## 🧱 2. Create a New ROS2 Package

```bash
ros2 pkg create bot_description --build-type ament_cmake
```

This creates the package `bot_description` with `ament_cmake` build system.

---

## 📁 3. Add Essential Folders

Navigate to the package and add necessary directories:

```bash
cd bot_description
mkdir launch models rviz urdf worlds
```

---

## 💻 4. Open in VS Code

Return to the `src` directory and open the workspace in VS Code:

```bash
cd ~/bot_ws/src
code .
```

---

## ⚙️ 5. Create the Base URDF (bot\_base.xacro)

Go to the `urdf` folder in VS Code and create a file named `bot_base.xacro`. Add the following step by step:

### Step 1: XML declaration and xacro root

```xml
<?xml version="1.0" ?>
<robot name="bot" xmlns:xacro="http://www.ros.org/wiki/xacro"></robot>
```

### Step 2: Add `base_footprint` link

```xml
<link name="base_footprint"/>
```

### Step 3: Add `base_link`

```xml
<link name="base_link">
    <inertial>
        <origin xyz="0 0 0" rpy="0 0 0"/>
        <mass value="5.0"/>
        <inertia ixx="0.1" ixy="0.0" ixz="0.0" iyy="0.1" iyz="0.0" izz="0.1"/>
    </inertial>
    <visual>
        <origin xyz="0.0 0.0 0.0" rpy="0.0 0.0 0.0"/>
        <geometry>
            <box size="0.7 0.5 0.2"/>
        </geometry>
        <material name="red">
            <color rgba="1.0 0.0 0.0 1.0"/>
        </material>
    </visual>
    <collision>
        <origin xyz="0.0 0.0 0.0" rpy="0.0 0.0 0.0"/>
        <geometry>
            <box size="0.7 0.5 0.2"/>
        </geometry>
    </collision>
</link>
```

### Step 4: Add a fixed joint

```xml
<joint name="base_joint" type="fixed">
    <origin xyz="0.0 0.0 0.15" rpy="0.0 0.0 0.0"/>
    <parent link="base_footprint"/>
    <child link="base_link"/>
</joint>
```

---

## 🧩 6. Create the Main URDF (bot.urdf.xacro)

In the same `urdf` folder, create a file called `bot.urdf.xacro` and add:

```xml
<?xml version="1.0" ?>
<robot name="bot" xmlns:xacro="http://www.ros.org/wiki/xacro">
    <xacro:include filename="$(find bot_description)/urdf/bot_base.xacro" />
</robot>
```

---

## 🚀 7. Create the Launch File

Go to the `launch` folder and create `display.launch.py`:

```python
import os
from ament_index_python.packages import get_package_share_directory

from launch import LaunchDescription
from launch.actions import DeclareLaunchArgument
from launch.substitutions import Command, LaunchConfiguration

from launch_ros.actions import Node
from launch_ros.parameter_descriptions import ParameterValue


def generate_launch_description():
    bot_description_dir = get_package_share_directory("bot_description")

    model_arg = DeclareLaunchArgument(name="model", default_value=os.path.join(
                                        bot_description_dir, "urdf", "bot.urdf.xacro"
                                        ),
                                      description="Absolute path to robot urdf file")
    
    robot_description = ParameterValue(Command([
            "xacro ",
            LaunchConfiguration("model"),
            " is_sim:=True",
        ]),
        value_type=str
    )
    
    robot_state_publisher_node = Node(
        package="robot_state_publisher",
        executable="robot_state_publisher",
        parameters=[{"robot_description": robot_description,
                     "use_sim_time": True}]
    )

    joint_state_publisher_gui_node = Node(
        package="joint_state_publisher_gui",
        executable="joint_state_publisher_gui"
    )

    rviz_node = Node(
        package="rviz2",
        executable="rviz2",
        name="rviz2",
        output="screen",
        arguments=["-d", os.path.join(bot_description_dir, "rviz", "display.rviz")],
    )

    return LaunchDescription([
        model_arg,
        joint_state_publisher_gui_node,
        robot_state_publisher_node,
        rviz_node
    ])
```

---

## 🎯 RViz Setup
## 8.A. Create RViz Configuration File

File: rviz/display.rviz

```bash
Panels:
  - Class: rviz_common/Displays
    Help Height: 0
    Name: Displays
    Property Tree Widget:
      Expanded:
        - /RobotModel1/Description Topic1
      Splitter Ratio: 0.5833333134651184
    Tree Height: 685
  - Class: rviz_common/Selection
    Name: Selection
  - Class: rviz_common/Tool Properties
    Expanded:
      - /Publish Point1
    Name: Tool Properties
    Splitter Ratio: 0.5886790156364441
  - Class: rviz_common/Views
    Expanded:
      - /Current View1
    Name: Views
    Splitter Ratio: 0.5
Visualization Manager:
  Class: ""
  Displays:
    - Alpha: 1
      Autocompute Intensity Bounds: true
      Autocompute Value Bounds:
        Max Value: 10
        Min Value: -10
        Value: true
      Axis: Z
      Channel Name: intensity
      Class: rviz_default_plugins/LaserScan
      Color: 255; 255; 255
      Color Transformer: Intensity
      Decay Time: 0
      Enabled: true
      Invert Rainbow: false
      Max Color: 255; 255; 255
      Max Intensity: 0
      Min Color: 0; 0; 0
      Min Intensity: 0
      Name: LaserScan
      Position Transformer: XYZ
      Selectable: true
      Size (Pixels): 3
      Size (m): 0.029999999329447746
      Style: Flat Squares
      Topic:
        Depth: 5
        Durability Policy: Volatile
        Filter size: 10
        History Policy: Keep Last
        Reliability Policy: Reliable
        Value: /scan
      Use Fixed Frame: true
      Use rainbow: true
      Value: true
    - Alpha: 0.5
      Cell Size: 1
      Class: rviz_default_plugins/Grid
      Color: 160; 160; 164
      Enabled: true
      Line Style:
        Line Width: 0.029999999329447746
        Value: Lines
      Name: Grid
      Normal Cell Count: 0
      Offset:
        X: 0
        Y: 0
        Z: 0
      Plane: XY
      Plane Cell Count: 10
      Reference Frame: <Fixed Frame>
      Value: true
    - Alpha: 1
      Class: rviz_default_plugins/RobotModel
      Collision Enabled: false
      Description File: ""
      Description Source: Topic
      Description Topic:
        Depth: 5
        Durability Policy: Volatile
        History Policy: Keep Last
        Reliability Policy: Reliable
        Value: /robot_description
      Enabled: true
      Links:
        All Links Enabled: true
        Expand Joint Details: false
        Expand Link Details: false
        Expand Tree: false
        Link Tree Style: Links in Alphabetic Order
        back_castor_link:
          Alpha: 1
          Show Axes: false
          Show Trail: false
          Value: true
        base_footprint:
          Alpha: 1
          Show Axes: false
          Show Trail: false
        base_link:
          Alpha: 1
          Show Axes: false
          Show Trail: false
          Value: true
        front_castor_link:
          Alpha: 1
          Show Axes: false
          Show Trail: false
          Value: true
        laser_link:
          Alpha: 1
          Show Axes: false
          Show Trail: false
          Value: true
        left_wheel_link:
          Alpha: 1
          Show Axes: false
          Show Trail: false
          Value: true
        right_wheel_link:
          Alpha: 1
          Show Axes: false
          Show Trail: false
          Value: true
      Mass Properties:
        Inertia: false
        Mass: false
      Name: RobotModel
      TF Prefix: ""
      Update Interval: 0
      Value: true
      Visual Enabled: true
    - Class: rviz_default_plugins/TF
      Enabled: true
      Frame Timeout: 15
      Frames:
        All Enabled: true
        back_castor_link:
          Value: true
        base_footprint:
          Value: true
        base_link:
          Value: true
        front_castor_link:
          Value: true
        laser_link:
          Value: true
        left_wheel_link:
          Value: true
        odom:
          Value: true
        right_wheel_link:
          Value: true
      Marker Scale: 1
      Name: TF
      Show Arrows: true
      Show Axes: true
      Show Names: false
      Tree:
        base_footprint:
          base_link:
            back_castor_link:
              {}
            front_castor_link:
              {}
            laser_link:
              {}
            left_wheel_link:
              {}
            right_wheel_link:
              {}
        odom:
          {}
      Update Interval: 0
      Value: true
  Enabled: true
  Global Options:
    Background Color: 48; 48; 48
    Fixed Frame: base_footprint
    Frame Rate: 30
  Name: root
  Tools:
    - Class: rviz_default_plugins/MoveCamera
    - Class: rviz_default_plugins/Select
    - Class: rviz_default_plugins/FocusCamera
    - Class: rviz_default_plugins/Measure
      Line color: 128; 128; 0
    - Class: rviz_default_plugins/SetInitialPose
      Covariance x: 0.25
      Covariance y: 0.25
      Covariance yaw: 0.06853891909122467
      Topic:
        Depth: 5
        Durability Policy: Volatile
        History Policy: Keep Last
        Reliability Policy: Reliable
        Value: /initialpose
    - Class: rviz_default_plugins/PublishPoint
      Single click: true
      Topic:
        Depth: 5
        Durability Policy: Volatile
        History Policy: Keep Last
        Reliability Policy: Reliable
        Value: /clicked_point
    - Class: nav2_rviz_plugins/GoalTool
  Transformation:
    Current:
      Class: rviz_default_plugins/TF
  Value: true
  Views:
    Current:
      Class: rviz_default_plugins/Orbit
      Distance: 3.625094413757324
      Enable Stereo Rendering:
        Stereo Eye Separation: 0.05999999865889549
        Stereo Focal Distance: 1
        Swap Stereo Eyes: false
        Value: false
      Focal Point:
        X: -0.07663324475288391
        Y: 0.07754191756248474
        Z: -0.014682665467262268
      Focal Shape Fixed Size: true
      Focal Shape Size: 0.05000000074505806
      Invert Z Axis: false
      Name: Current View
      Near Clip Distance: 0.009999999776482582
      Pitch: 0.6752017736434937
      Target Frame: <Fixed Frame>
      Value: Orbit (rviz_default_plugins)
      Yaw: 2.0423831939697266
    Saved: ~
Window Geometry:
  Displays:
    collapsed: false
  Height: 836
  Hide Left Dock: false
  Hide Right Dock: true
  QMainWindow State: 000000ff00000000fd00000004000000000000018b000002eafc020000000cfb0000001200530065006c0065006300740069006f006e00000001e10000009b0000005c00fffffffb0000001e0054006f006f006c002000500072006f007000650072007400690065007302000001ed000001df00000185000000a3fb000000120056006900650077007300200054006f006f02000001df000002110000018500000122fb000000200054006f006f006c002000500072006f0070006500720074006900650073003203000002880000011d000002210000017afb000000100044006900730070006c006100790073010000003d000002ea000000c900fffffffb0000002000730065006c0065006300740069006f006e00200062007500660066006500720200000138000000aa0000023a00000294fb00000014005700690064006500530074006500720065006f02000000e6000000d2000003ee0000030bfb0000000c004b0069006e0065006300740200000186000001060000030c00000261fb00000018004e0061007600690067006100740069006f006e0020003201000001ee000001390000000000000000fb0000001e005200650061006c00730065006e0073006500430061006d006500720061000000028f000000980000000000000000fb0000000c00430061006d0065007200610000000267000000c00000000000000000fb0000000c00430061006d0065007200610100000267000000c00000000000000000000000010000010f000002eafc0200000003fb0000001e0054006f006f006c002000500072006f00700065007200740069006500730100000041000000780000000000000000fb0000000a00560069006500770073000000003d000002ea000000a400fffffffb0000001200530065006c0065006300740069006f006e010000025a000000b200000000000000000000000200000490000000a9fc0100000001fb0000000a00560069006500770073030000004e00000080000002e10000019700000003000004420000003efc0100000002fb0000000800540069006d00650100000000000004420000000000000000fb0000000800540069006d00650100000000000004500000000000000000000004af000002ea00000004000000040000000800000008fc0000000100000002000000010000000a0054006f006f006c00730100000000ffffffff0000000000000000
  Selection:
    collapsed: false
  Tool Properties:
    collapsed: false
  Views:
    collapsed: true
  Width: 1600
  X: 642
  Y: 102

```

---

Absolutely! Here's how you can write the next step (📦 **Step 22**) for your GitHub `README.md`, showing how to set up the `CMakeLists.txt` and `package.xml` for the `bot_description` package — with clear explanations and emojis:

---

## 📦 8.B.: Setup `CMakeLists.txt` and `package.xml` for `bot_description`

This step sets up the core metadata and build configuration files for the `bot_description` package.

---

### 🧱 `CMakeLists.txt`

Create or update your `CMakeLists.txt` inside the `bot_description` folder with the following content:

```cmake
cmake_minimum_required(VERSION 3.8)
project(bot_description)

if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic)
endif()

# find dependencies
find_package(ament_cmake REQUIRED)

# Install important directories like URDF, launch, rviz, etc.
install(
  DIRECTORY launch models urdf rviz worlds
  DESTINATION share/${PROJECT_NAME}
)

# Optional testing support
if(BUILD_TESTING)
  find_package(ament_lint_auto REQUIRED)
  set(ament_cmake_copyright_FOUND TRUE)
  set(ament_cmake_cpplint_FOUND TRUE)
  ament_lint_auto_find_test_dependencies()
endif()

ament_package()
```

✅ **What this does**:

* Sets the CMake version and project name.
* Enables strict compile warnings.
* Finds the `ament_cmake` build system.
* Installs your `launch`, `urdf`, `rviz`, `models`, and `worlds` folders.
* Optionally sets up testing and linting tools.

---

### 📄 `package.xml`

Add the following `package.xml` file in the same `bot_description` folder:

```xml
<?xml version="1.0"?>
<?xml-model href="http://download.ros.org/schema/package_format3.xsd" schematypens="http://www.w3.org/2001/XMLSchema"?>
<package format="3">
  <name>bot_description</name>
  <version>0.0.0</version>
  <description>TODO: Package description</description>
  <maintainer email="kutkarsh706@gmail.com">utk</maintainer>
  <license>TODO: License declaration</license>

  <buildtool_depend>ament_cmake</buildtool_depend>

  <!-- Runtime dependencies -->
  <exec_depend>robot_state_publisher</exec_depend>
  <exec_depend>urdf</exec_depend>
  <exec_depend>joint_state_publisher_gui</exec_depend>
  <exec_depend>rviz2</exec_depend>
  <exec_depend>xacro</exec_depend>
  <exec_depend>ros2launch</exec_depend>
  <exec_depend>ros_gz_sim</exec_depend>

  <!-- Test dependencies -->
  <test_depend>ament_lint_auto</test_depend>
  <test_depend>ament_lint_common</test_depend>

  <export>
    <build_type>ament_cmake</build_type>
  </export>
</package>
```

🧾 **What this includes**:

* Metadata like package name, version, maintainer, and license.
* Declares **runtime dependencies** like `xacro`, `robot_state_publisher`, `rviz2`, and `ros_gz_sim`.
* Adds **testing tools** under test dependencies.
* Uses `ament_cmake` as the build tool.

---


## 🛠️ 9. Build the Workspace

Go to your workspace root and build the package:

```bash
cd ~/bot_ws
colcon build
```

---

## 🌐 10. Source and Launch

After a successful build:

```bash
source ~/.bashrc
ros2 launch bot_description display.launch.py
```

Your robot should now appear in **RViz** with the correct URDF structure!

---

Sure! Here's a **cleaner rewrite of Steps 11 to 13**, with each component (left/right wheels, front/rear castors, sensors) broken into **separate code snippets** for clarity in your `README.md`.

---

## **Step 11: Add Left and Right Wheels**

### 📄 Edit `bot_base.xacro`

Open the file:

```bash
cd ~/bot_ws/src/bot_description/urdf
nano bot_base.xacro
```

### ➕ Add Right Wheel

```xml
<!-- Right Wheel -->
<link name="right_wheel_link">
  <inertial>
    <origin xyz="0 0 0" rpy="0 0 0"/>
    <mass value="1.0"/>
    <inertia ixx="0.01" ixy="0.0" ixz="0.0" iyy="0.01" iyz="0.0" izz="0.01"/>
  </inertial>
  <visual>
    <origin xyz="0.0 0.0 0.0" rpy="0.0 0.0 0.0"/>
    <geometry>
      <cylinder radius="0.15" length="0.05"/>
    </geometry>
    <material name="blue">
      <color rgba="0.0 0.0 1.0 1.0"/>
    </material>
  </visual>
  <collision>
    <origin xyz="0.0 0.0 0.0" rpy="0.0 0.0 0.0"/>
    <geometry>
      <cylinder radius="0.15" length="0.05"/>
    </geometry>
  </collision>
</link>

<joint name="right_wheel_joint" type="continuous">
  <origin xyz="0.0 -0.275 0.0" rpy="1.57 0.0 0.0"/>
  <parent link="base_link"/>
  <child link="right_wheel_link"/>
  <axis xyz="0.0 0.0 -1.0"/>
</joint>
```

### ➕ Add Left Wheel

```xml
<!-- Left Wheel -->
<link name="left_wheel_link">
  <inertial>
    <origin xyz="0 0 0" rpy="0 0 0"/>
    <mass value="1.0"/>
    <inertia ixx="0.01" ixy="0.0" ixz="0.0" iyy="0.01" iyz="0.0" izz="0.01"/>
  </inertial>
  <visual>
    <origin xyz="0.0 0.0 0.0" rpy="0.0 0.0 0.0"/>
    <geometry>
      <cylinder radius="0.15" length="0.05"/>
    </geometry>
    <material name="blue">
      <color rgba="0.0 0.0 1.0 1.0"/>
    </material>
  </visual>
  <collision>
    <origin xyz="0.0 0.0 0.0" rpy="0.0 0.0 0.0"/>
    <geometry>
      <cylinder radius="0.15" length="0.05"/>
    </geometry>
  </collision>
</link>

<joint name="left_wheel_joint" type="continuous">
  <origin xyz="0.0 0.275 0.0" rpy="1.57 0.0 0.0"/>
  <parent link="base_link"/>
  <child link="left_wheel_link"/>
  <axis xyz="0.0 0.0 -1.0"/>
</joint>
```

### 🔧 Build and Launch RViz

```bash
cd ~/bot_ws
colcon build --symlink-install
source install/setup.bash
ros2 launch bot_description display.launch.py
```

✅ Check if **both wheels** are correctly attached in RViz.

---

## **Step 12: Add Castor Wheels**

### ➕ Add Front Castor

```xml
<!-- Front Castor -->
<link name="front_castor_link">
  <inertial>
    <origin xyz="0 0 0" rpy="0 0 0"/>
    <mass value="0.5"/>
    <inertia ixx="0.005" ixy="0.0" ixz="0.0" iyy="0.005" iyz="0.0" izz="0.005"/>
  </inertial>
  <visual>
    <origin xyz="0.0 0.0 0.0" rpy="0.0 0.0 0.0"/>
    <geometry>
      <sphere radius="0.045"/>
    </geometry>
    <material name="black">
      <color rgba="0.0 0.0 0.0 1.0"/>
    </material>
  </visual>
  <collision>
    <origin xyz="0.0 0.0 0.0" rpy="0.0 0.0 0.0"/>
    <geometry>
      <sphere radius="0.045"/>
    </geometry>
  </collision>
</link>

<joint name="front_castor_joint" type="fixed">
  <origin xyz="0.25 0.0 -0.1" rpy="0.0 0.0 0.0"/>
  <parent link="base_link"/>
  <child link="front_castor_link"/>
</joint>
```

### ➕ Add Rear Castor

```xml
<!-- Rear Castor -->
<link name="back_castor_link">
  <inertial>
    <origin xyz="0 0 0" rpy="0 0 0"/>
    <mass value="0.5"/>
    <inertia ixx="0.005" ixy="0.0" ixz="0.0" iyy="0.005" iyz="0.0" izz="0.005"/>
  </inertial>
  <visual>
    <origin xyz="0.0 0.0 0.0" rpy="0.0 0.0 0.0"/>
    <geometry>
      <sphere radius="0.045"/>
    </geometry>
    <material name="black">
      <color rgba="0.0 0.0 0.0 1.0"/>
    </material>
  </visual>
  <collision>
    <origin xyz="0.0 0.0 0.0" rpy="0.0 0.0 0.0"/>
    <geometry>
      <sphere radius="0.045"/>
    </geometry>
  </collision>
</link>

<joint name="back_castor_joint" type="fixed">
  <origin xyz="-0.25 0.0 -0.1" rpy="0.0 0.0 0.0"/>
  <parent link="base_link"/>
  <child link="back_castor_link"/>
</joint>
```

### 🔧 Rebuild and Launch RViz

```bash
colcon build --symlink-install
source install/setup.bash
ros2 launch bot_description display.launch.py
```

✅ Check if **both castors** are added and positioned correctly.

---

## **Step 13: Add Sensor Using a Separate Xacro File**

### 🆕 Create `bot_sensors.xacro`

```bash
cd ~/bot_ws/src/bot_description/urdf
nano bot_sensors.xacro
```

### 🧩 Add the Sensor Link and Joint

```xml
<?xml version="1.0" ?>
<robot name="bot" xmlns:xacro="http://www.ros.org/wiki/xacro">

  <link name="laser_link">
    <inertial>
      <origin xyz="0.0 0.0 0.0" rpy="0.0 0.0 0.0"/>
      <mass value="0.3"/>
      <inertia ixx="0.0002" ixy="0.0" ixz="0.0" iyy="0.0004" iyz="0.0" izz="0.0005"/>
    </inertial>
    <visual>
      <origin xyz="0.0 0.0 0.0" rpy="0.0 0.0 0.0"/>
      <geometry>
        <box size="0.05 0.2 0.1"/>
      </geometry>
      <material name="green">
        <color rgba="0.0 1.0 0.0 1.0"/>
      </material>
    </visual>
    <collision>
      <origin xyz="0.0 0.0 0.0" rpy="0.0 0.0 0.0"/>
      <geometry>
        <box size="0.05 0.2 0.1"/>
      </geometry>
    </collision>
  </link>

  <joint name="laser_joint" type="fixed">
    <origin xyz="0.35 0.0 0.0" rpy="0.0 0.0 0.0"/>
    <parent link="base_link"/>
    <child link="laser_link"/>
  </joint>

</robot>
```

### 🧩 Include in `bot.urdf.xacro`

Open `bot.urdf.xacro`:

```bash
nano bot.urdf.xacro
```

Add this line **at the top**, before `<robot>` begins:

```xml
<xacro:include filename="$(find bot_description)/urdf/bot_sensors.xacro" />
```

### ✅ Why Use Xacro?

Using `.xacro` provides the following benefits:

* ✅ Reuse macros like functions to avoid repeating the same XML blocks.
* ✅ Parameterize sizes, colors, and positions using variables.
* ✅ Split URDF into smaller, manageable pieces.
* ✅ Easier maintenance and debugging.

---

### 🔁 Final Build and Visual Test

```bash
cd ~/bot_ws
colcon build --symlink-install
source install/setup.bash
ros2 launch bot_description display.launch.py
```

✅ You should now see the full robot in RViz with:

* Base link
* Left and right wheels
* Front and back castors
* A front-mounted laser sensor

---

---

## 🚀 Step 14: Create the `gazebo.xacro` File

📁 Navigate to the `urdf` folder and create a new file called **`gazebo.xacro`**:

```xml
<?xml version="1.0" ?>
<robot name="bot" xmlns:xacro="http://www.ros.org/wiki/xacro">
</robot>
```

---

## ⚙️ Step 15: Add Gazebo Properties for Right Wheel

🔧 Inside the `<robot>` tag, add Gazebo friction and contact properties for the **right wheel**:

```xml
<!-- bot Wheels -->
<gazebo reference="right_wheel_link">
    <mu1>1000000000000000.0</mu1>
    <mu2>1000000000000000.0</mu2>
    <kp>1000000000000.0</kp>
    <kd>10.0</kd>
    <minDepth>0.001</minDepth>
    <maxVel>0.1</maxVel>
    <fdir1>1 0 0</fdir1>
</gazebo>
```

### 📝 Explanation:

* 🔩 **mu1, mu2** – Friction coefficients (very high to prevent slipping)
* 🧷 **kp** – Spring stiffness (high for rigid contact)
* 🛠️ **kd** – Damping coefficient
* 🔎 **minDepth** – Minimum depth before contact forces apply
* ⚡ **maxVel** – Max contact velocity for stability
* 🔁 **fdir1** – Friction direction (wheel rolling axis)

---

## ⚙️ Step 16: Add Gazebo Properties for Left Wheel

🛞 Add the same block for the **left wheel**:

```xml
<gazebo reference="left_wheel_link">
    <mu1>1000000000000000.0</mu1>
    <mu2>1000000000000000.0</mu2>
    <kp>1000000000000.0</kp>
    <kd>10.0</kd>
    <minDepth>0.001</minDepth>
    <maxVel>0.1</maxVel>
    <fdir1>1 0 0</fdir1>
</gazebo>
```

---

## 🛞 Step 17: Add Properties for Castor Wheels

⚙️ Castor wheels aren’t powered, so they use lower friction and different damping:

```xml
<!-- Caster Wheels -->
<gazebo reference="front_castor_link">
    <mu1>0.1</mu1>
    <mu2>0.1</mu2>
    <kp>1000000.0</kp>
    <kd>100.0</kd>
    <minDepth>0.001</minDepth>
    <maxVel>1.0</maxVel>
</gazebo>

<gazebo reference="back_castor_link">
    <mu1>0.1</mu1>
    <mu2>0.1</mu2>
    <kp>1000000.0</kp>
    <kd>100.0</kd>
    <minDepth>0.001</minDepth>
    <maxVel>1.0</maxVel>
</gazebo>
```

---

## 🛰️ Step 18: Add 2D Laser Sensor Plugin

📡 Add a simulated GPU-based laser scanner (LiDAR):

```xml
<!-- bot Sensors -->
<!-- 2D Laser Sensor -->
<gazebo reference="laser_link">
    <sensor name="lidar" type="gpu_lidar">
        <pose>0 0 0 1.57 0 0</pose>
        <always_on>true</always_on>
        <visualize>true</visualize>
        <topic>scan</topic>
        <update_rate>5</update_rate>
        <gz_frame_id>laser_link</gz_frame_id> 
        <lidar>
            <scan>
                <horizontal>
                    <samples>10</samples>
                    <resolution>1.00000</resolution>
                    <min_angle>-1.000000</min_angle>
                    <max_angle>0.0</max_angle>
                </horizontal>
            </scan>
            <range>
                <min>0.05</min>
                <max>4.0</max>
                <resolution>0.02</resolution>
            </range>
            <noise>
                <type>gaussian</type>
                <mean>0.0</mean>
                <stddev>0.01</stddev>
            </noise>
        </lidar>
    </sensor>
</gazebo>
```

---

## 🧩 Step 19: Include `gazebo.xacro` in Your Main URDF

Go to `bot.urdf.xacro` and include the gazebo file:

```xml
<xacro:include filename="$(find bot_description)/urdf/gazebo.xacro"/>
```

📌 Place it below the other includes like `bot_base.xacro` and `bot_sensors.xacro`.

---
---

## 🚀 Step 20: Launch Your Bot in Gazebo – `gazebo.launch.py`

🔧 Go to the `launch/` folder in your `bot_description` package and **create a launch file** named `gazebo.launch.py`.

Paste the following code inside:

```python
import os
from os import pathsep
from pathlib import Path
from ament_index_python.packages import get_package_share_directory

from launch import LaunchDescription
from launch.actions import DeclareLaunchArgument, IncludeLaunchDescription, SetEnvironmentVariable
from launch.substitutions import Command, LaunchConfiguration, PathJoinSubstitution, PythonExpression
from launch.launch_description_sources import PythonLaunchDescriptionSource

from launch_ros.actions import Node
from launch_ros.parameter_descriptions import ParameterValue

def generate_launch_description():
    bot_description = get_package_share_directory("bot_description")

    model_arg = DeclareLaunchArgument(
        name="model", default_value=os.path.join(
                bot_description, "urdf", "bot.urdf.xacro"
            ),
        description="Absolute path to robot urdf file"
    )

    world_name_arg = DeclareLaunchArgument(name="world_name", default_value="empty_table")

    world_path = PathJoinSubstitution([
            bot_description,
            "worlds",
            PythonExpression(expression=["'", LaunchConfiguration("world_name"), "'", " + '.world'"])
        ]
    )

    model_path = str(Path(bot_description).parent.resolve())
    model_path += pathsep + os.path.join(get_package_share_directory("bot_description"), 'models')

    gazebo_resource_path = SetEnvironmentVariable(
        "GZ_SIM_RESOURCE_PATH",
        model_path
    )

    robot_description = ParameterValue(Command([
            "xacro ",
            LaunchConfiguration("model"),
            " is_sim:=True",
        ]),
        value_type=str
    )

    robot_state_publisher_node = Node(
        package="robot_state_publisher",
        executable="robot_state_publisher",
        parameters=[{"robot_description": robot_description,
                     "use_sim_time": True}]
    )

    gazebo = IncludeLaunchDescription(
                PythonLaunchDescriptionSource([os.path.join(
                    get_package_share_directory("ros_gz_sim"), "launch"), "/gz_sim.launch.py"]),
                launch_arguments={
                    "gz_args": PythonExpression(["'", world_path, " -v 4 -r'"])
                }.items()
             )

    gz_spawn_entity = Node(
        package="ros_gz_sim",
        executable="create",
        output="screen",
        arguments=[
            "-topic", "robot_description",
            "-name", "bot",
            "-x", "0.0",  
            "-y", "0.0",  
            "-z", "2.0",  
            "-R", "0.0", 
            "-P", "0.0",
            "-Y", "0.0",
        ],
    )

    gz_ros2_bridge = Node(
        package="ros_gz_bridge",
        executable="parameter_bridge",
        arguments=[
            "/clock@rosgraph_msgs/msg/Clock[gz.msgs.Clock",
            "/scan@sensor_msgs/msg/LaserScan[gz.msgs.LaserScan",
        ],
    )

    return LaunchDescription([
        model_arg,
        world_name_arg,
        gazebo_resource_path,
        robot_state_publisher_node,
        gazebo,
        gz_spawn_entity,
        gz_ros2_bridge,
    ])
```

✅ This launch file:

* Launches the **Gazebo simulator**
* Loads your **robot description**
* Spawns the robot into the world
* Sets up **sensor bridges** for `/scan` and `/clock`

---

## 🧱 Step 21: Add Table Model to `models/` Folder

📁 Go to the `models/` folder inside the `bot_description` package and **paste the table model folder** there.

Your structure should look like this:

```
bot_description/
├── models/
│   └── table/       # 👈 Make sure this folder contains model.sdf and model.config
```

✅ Gazebo will load this model when the appropriate world file references it.

---

## 🌍 Step 22: Create an Empty World – `empty.world`

📂 Go to the `worlds/` folder in the same package and **create a file** named `empty.world`.

Paste the following contents inside:

```xml
<?xml version="1.0" ?>
<sdf version="1.7">
    <world name="empty_world">
        <!-- Directional Light -->
        <light type="directional" name="sun">
            <cast_shadows>true</cast_shadows>
            <pose>0 0 10 0 0 0</pose>
            <diffuse>0.8 0.8 0.8 1</diffuse>
            <specular>0.2 0.2 0.2 1</specular>
            <attenuation>
                <range>1000</range>
                <constant>0.9</constant>
                <linear>0.01</linear>
                <quadratic>0.001</quadratic>
            </attenuation>
            <direction>-0.5 0.1 -0.9</direction>
        </light>

        <!-- Ground Plane -->
        <model name="ground_plane">
            <static>true</static>
            <link name="link">
                <collision name="collision">
                    <geometry>
                        <plane>
                            <normal>0 0 1</normal>
                        </plane>
                    </geometry>
                </collision>
                <visual name="visual">
                    <geometry>
                        <plane>
                            <normal>0 0 1</normal>
                            <size>100 100</size>
                        </plane>
                    </geometry>
                    <material>
                        <ambient>0.8 0.8 0.8 1</ambient>
                        <diffuse>0.8 0.8 0.8 1</diffuse>
                        <specular>0.8 0.8 0.8 1</specular>
                    </material>
                </visual>
            </link>
        </model>
    </world>
</sdf>
```

✅ This world:

* Adds a **ground plane**
* Adds **sunlight**
* Keeps the world clean and minimal for testing your robot

---


---

## 🌍 Step 23: Create an Empty World with a Table– `empty_table.world`

📂 Go to the `worlds/` folder in the same package and **create a file** named `empty_table.world`.

Paste the following contents inside:

```xml
<sdf version='1.9'>
  <world name='empty_world'>
    <gravity>0 0 -9.8</gravity>
    <magnetic_field>6e-06 2.3e-05 -4.2e-05</magnetic_field>
    <atmosphere type='adiabatic'/>
    <physics type='ode'>
      <max_step_size>0.001</max_step_size>
      <real_time_factor>1</real_time_factor>
      <real_time_update_rate>1000</real_time_update_rate>
    </physics>
    <scene>
      <ambient>0.4 0.4 0.4 1</ambient>
      <background>0.7 0.7 0.7 1</background>
      <shadows>true</shadows>
    </scene>
    <model name='ground_plane'>
      <static>true</static>
      <link name='link'>
        <collision name='collision'>
          <geometry>
            <plane>
              <normal>0 0 1</normal>
              <size>1 1</size>
            </plane>
          </geometry>
          <surface>
            <friction>
              <ode/>
            </friction>
            <bounce/>
            <contact/>
          </surface>
        </collision>
        <visual name='visual'>
          <geometry>
            <plane>
              <normal>0 0 1</normal>
              <size>100 100</size>
            </plane>
          </geometry>
          <material>
            <ambient>0.8 0.8 0.8 1</ambient>
            <diffuse>0.8 0.8 0.8 1</diffuse>
            <specular>0.8 0.8 0.8 1</specular>
          </material>
        </visual>
        <pose>0 0 0 0 -0 0</pose>
        <inertial>
          <pose>0 0 0 0 -0 0</pose>
          <mass>1</mass>
          <inertia>
            <ixx>1</ixx>
            <ixy>0</ixy>
            <ixz>0</ixz>
            <iyy>1</iyy>
            <iyz>0</iyz>
            <izz>1</izz>
          </inertia>
        </inertial>
        <enable_wind>false</enable_wind>
      </link>
      <pose>0 0 0 0 -0 0</pose>
      <self_collide>false</self_collide>
    </model>
    <model name='CoffeeTable_1'>
      <static>true</static>
      <model name='aws_robomaker_residential_CoffeeTable_01'>
        <link name='link'>
          <inertial>
            <mass>21.699999999999999</mass>
            <inertia>
              <ixx>0.81374999999999975</ixx>
              <ixy>0</ixy>
              <ixz>0</ixz>
              <iyy>2.76675</iyy>
              <iyz>0</iyz>
              <izz>3.254</izz>
            </inertia>
          </inertial>
          <collision name='collision'>
            <geometry>
              <mesh>
                <uri>model://aws_robomaker_residential_CoffeeTable_01/meshes/aws_CoffeeTable_01_collision.DAE</uri>
                <scale>1 1 1</scale>
              </mesh>
            </geometry>
            <surface>
              <contact>
                <ode>
                  <max_vel>0.01</max_vel>
                </ode>
              </contact>
            </surface>
          </collision>
          <visual name='visual'>
            <geometry>
              <mesh>
                <uri>model://aws_robomaker_residential_CoffeeTable_01/meshes/aws_CoffeeTable_01_visual.DAE</uri>
              </mesh>
            </geometry>
            <meta>
              <layer>1</layer>
            </meta>
          </visual>
        </link>
        <static>true</static>
      </model>
      <pose>0 0 0.041983 0 -0 0</pose>
      <self_collide>false</self_collide>
    </model>
    <model name='CoffeeTable_2'>
      <static>true</static>
      <model name='aws_robomaker_residential_CoffeeTable_01'>
        <link name='link'>
          <inertial>
            <mass>21.699999999999999</mass>
            <inertia>
              <ixx>0.81374999999999975</ixx>
              <ixy>0</ixy>
              <ixz>0</ixz>
              <iyy>2.76675</iyy>
              <iyz>0</iyz>
              <izz>3.254</izz>
            </inertia>
          </inertial>
          <collision name='collision'>
            <geometry>
              <mesh>
                <uri>model://aws_robomaker_residential_CoffeeTable_01/meshes/aws_CoffeeTable_01_collision.DAE</uri>
                <scale>1 1 1</scale>
              </mesh>
            </geometry>
            <surface>
              <contact>
                <ode>
                  <max_vel>0.01</max_vel>
                </ode>
              </contact>
            </surface>
          </collision>
          <visual name='visual'>
            <geometry>
              <mesh>
                <uri>model://aws_robomaker_residential_CoffeeTable_01/meshes/aws_CoffeeTable_01_visual.DAE</uri>
              </mesh>
            </geometry>
            <meta>
              <layer>1</layer>
            </meta>
          </visual>
        </link>
        <static>true</static>
      </model>
      <pose>1 0 0.041983 0 -0 0</pose>
      <self_collide>false</self_collide>
    </model>
    <model name='CoffeeTable_3'>
      <static>true</static>
      <model name='aws_robomaker_residential_CoffeeTable_01'>
        <link name='link'>
          <inertial>
            <mass>21.699999999999999</mass>
            <inertia>
              <ixx>0.81374999999999975</ixx>
              <ixy>0</ixy>
              <ixz>0</ixz>
              <iyy>2.76675</iyy>
              <iyz>0</iyz>
              <izz>3.254</izz>
            </inertia>
          </inertial>
          <collision name='collision'>
            <geometry>
              <mesh>
                <uri>model://aws_robomaker_residential_CoffeeTable_01/meshes/aws_CoffeeTable_01_collision.DAE</uri>
                <scale>1 1 1</scale>
              </mesh>
            </geometry>
            <surface>
              <contact>
                <ode>
                  <max_vel>0.01</max_vel>
                </ode>
              </contact>
            </surface>
          </collision>
          <visual name='visual'>
            <geometry>
              <mesh>
                <uri>model://aws_robomaker_residential_CoffeeTable_01/meshes/aws_CoffeeTable_01_visual.DAE</uri>
              </mesh>
            </geometry>
            <meta>
              <layer>1</layer>
            </meta>
          </visual>
        </link>
        <static>true</static>
      </model>
      <pose>1.01186 1.27053 0.041983 0 -0 0</pose>
      <self_collide>false</self_collide>
    </model>
    <model name='CoffeeTable_4'>
      <static>true</static>
      <model name='aws_robomaker_residential_CoffeeTable_01'>
        <link name='link'>
          <inertial>
            <mass>21.699999999999999</mass>
            <inertia>
              <ixx>0.81374999999999975</ixx>
              <ixy>0</ixy>
              <ixz>0</ixz>
              <iyy>2.76675</iyy>
              <iyz>0</iyz>
              <izz>3.254</izz>
            </inertia>
          </inertial>
          <collision name='collision'>
            <geometry>
              <mesh>
                <uri>model://aws_robomaker_residential_CoffeeTable_01/meshes/aws_CoffeeTable_01_collision.DAE</uri>
                <scale>1 1 1</scale>
              </mesh>
            </geometry>
            <surface>
              <contact>
                <ode>
                  <max_vel>0.01</max_vel>
                </ode>
              </contact>
            </surface>
          </collision>
          <visual name='visual'>
            <geometry>
              <mesh>
                <uri>model://aws_robomaker_residential_CoffeeTable_01/meshes/aws_CoffeeTable_01_visual.DAE</uri>
              </mesh>
            </geometry>
            <meta>
              <layer>1</layer>
            </meta>
          </visual>
        </link>
        <static>true</static>
      </model>
      <pose>0 0.652446 0.041983 0 -0 0</pose>
      <self_collide>false</self_collide>
    </model>
    <model name='CoffeeTable_5'>
      <static>true</static>
      <model name='aws_robomaker_residential_CoffeeTable_01'>
        <link name='link'>
          <inertial>
            <mass>21.699999999999999</mass>
            <inertia>
              <ixx>0.81374999999999975</ixx>
              <ixy>0</ixy>
              <ixz>0</ixz>
              <iyy>2.76675</iyy>
              <iyz>0</iyz>
              <izz>3.254</izz>
            </inertia>
          </inertial>
          <collision name='collision'>
            <geometry>
              <mesh>
                <uri>model://aws_robomaker_residential_CoffeeTable_01/meshes/aws_CoffeeTable_01_collision.DAE</uri>
                <scale>1 1 1</scale>
              </mesh>
            </geometry>
            <surface>
              <contact>
                <ode>
                  <max_vel>0.01</max_vel>
                </ode>
              </contact>
            </surface>
          </collision>
          <visual name='visual'>
            <geometry>
              <mesh>
                <uri>model://aws_robomaker_residential_CoffeeTable_01/meshes/aws_CoffeeTable_01_visual.DAE</uri>
              </mesh>
            </geometry>
            <meta>
              <layer>1</layer>
            </meta>
          </visual>
        </link>
        <static>true</static>
      </model>
      <pose>1 0.631228 0.041983 0 -0 0</pose>
      <self_collide>false</self_collide>
    </model>
    <model name='CoffeeTable_6'>
      <static>true</static>
      <model name='aws_robomaker_residential_CoffeeTable_01'>
        <link name='link'>
          <inertial>
            <mass>21.699999999999999</mass>
            <inertia>
              <ixx>0.81374999999999975</ixx>
              <ixy>0</ixy>
              <ixz>0</ixz>
              <iyy>2.76675</iyy>
              <iyz>0</iyz>
              <izz>3.254</izz>
            </inertia>
          </inertial>
          <collision name='collision'>
            <geometry>
              <mesh>
                <uri>model://aws_robomaker_residential_CoffeeTable_01/meshes/aws_CoffeeTable_01_collision.DAE</uri>
                <scale>1 1 1</scale>
              </mesh>
            </geometry>
            <surface>
              <contact>
                <ode>
                  <max_vel>0.01</max_vel>
                </ode>
              </contact>
            </surface>
          </collision>
          <visual name='visual'>
            <geometry>
              <mesh>
                <uri>model://aws_robomaker_residential_CoffeeTable_01/meshes/aws_CoffeeTable_01_visual.DAE</uri>
              </mesh>
            </geometry>
            <meta>
              <layer>1</layer>
            </meta>
          </visual>
        </link>
        <static>true</static>
      </model>
      <pose>0.00463 1.28609 0.041983 0 -0 0</pose>
      <self_collide>false</self_collide>
    </model>
    <model name='CoffeeTable_1_1'>
      <static>true</static>
      <model name='aws_robomaker_residential_CoffeeTable_01'>
        <link name='link'>
          <inertial>
            <mass>21.699999999999999</mass>
            <inertia>
              <ixx>0.81374999999999975</ixx>
              <ixy>0</ixy>
              <ixz>0</ixz>
              <iyy>2.76675</iyy>
              <iyz>0</iyz>
              <izz>3.254</izz>
            </inertia>
          </inertial>
          <collision name='collision'>
            <geometry>
              <mesh>
                <uri>model://aws_robomaker_residential_CoffeeTable_01/meshes/aws_CoffeeTable_01_collision.DAE</uri>
                <scale>1 1 1</scale>
              </mesh>
            </geometry>
            <surface>
              <contact>
                <ode>
                  <max_vel>0.01</max_vel>
                </ode>
              </contact>
            </surface>
          </collision>
          <visual name='visual'>
            <geometry>
              <mesh>
                <uri>model://aws_robomaker_residential_CoffeeTable_01/meshes/aws_CoffeeTable_01_visual.DAE</uri>
              </mesh>
            </geometry>
            <meta>
              <layer>1</layer>
            </meta>
          </visual>
        </link>
        <static>true</static>
      </model>
      <pose>-0.000582 -0.654619 0.041983 0 -0 0</pose>
      <self_collide>false</self_collide>
    </model>
    <model name='CoffeeTable_1_2'>
      <static>true</static>
      <model name='aws_robomaker_residential_CoffeeTable_01'>
        <link name='link'>
          <inertial>
            <mass>21.699999999999999</mass>
            <inertia>
              <ixx>0.81374999999999975</ixx>
              <ixy>0</ixy>
              <ixz>0</ixz>
              <iyy>2.76675</iyy>
              <iyz>0</iyz>
              <izz>3.254</izz>
            </inertia>
          </inertial>
          <collision name='collision'>
            <geometry>
              <mesh>
                <uri>model://aws_robomaker_residential_CoffeeTable_01/meshes/aws_CoffeeTable_01_collision.DAE</uri>
                <scale>1 1 1</scale>
              </mesh>
            </geometry>
            <surface>
              <contact>
                <ode>
                  <max_vel>0.01</max_vel>
                </ode>
              </contact>
            </surface>
          </collision>
          <visual name='visual'>
            <geometry>
              <mesh>
                <uri>model://aws_robomaker_residential_CoffeeTable_01/meshes/aws_CoffeeTable_01_visual.DAE</uri>
              </mesh>
            </geometry>
            <meta>
              <layer>1</layer>
            </meta>
          </visual>
        </link>
        <static>true</static>
      </model>
      <pose>0.99619 -0.630908 0.041983 0 -0 0</pose>
      <self_collide>false</self_collide>
    </model>
    <model name='CoffeeTable_1_3'>
      <static>true</static>
      <model name='aws_robomaker_residential_CoffeeTable_01'>
        <link name='link'>
          <inertial>
            <mass>21.699999999999999</mass>
            <inertia>
              <ixx>0.81374999999999975</ixx>
              <ixy>0</ixy>
              <ixz>0</ixz>
              <iyy>2.76675</iyy>
              <iyz>0</iyz>
              <izz>3.254</izz>
            </inertia>
          </inertial>
          <collision name='collision'>
            <geometry>
              <mesh>
                <uri>model://aws_robomaker_residential_CoffeeTable_01/meshes/aws_CoffeeTable_01_collision.DAE</uri>
                <scale>1 1 1</scale>
              </mesh>
            </geometry>
            <surface>
              <contact>
                <ode>
                  <max_vel>0.01</max_vel>
                </ode>
              </contact>
            </surface>
          </collision>
          <visual name='visual'>
            <geometry>
              <mesh>
                <uri>model://aws_robomaker_residential_CoffeeTable_01/meshes/aws_CoffeeTable_01_visual.DAE</uri>
              </mesh>
            </geometry>
            <meta>
              <layer>1</layer>
            </meta>
          </visual>
        </link>
        <static>true</static>
      </model>
      <pose>1.00337 -1.27806 0.041983 0 -0 0</pose>
      <self_collide>false</self_collide>
    </model>
    <model name='CoffeeTable_1_4'>
      <static>true</static>
      <model name='aws_robomaker_residential_CoffeeTable_01'>
        <link name='link'>
          <inertial>
            <mass>21.699999999999999</mass>
            <inertia>
              <ixx>0.81374999999999975</ixx>
              <ixy>0</ixy>
              <ixz>0</ixz>
              <iyy>2.76675</iyy>
              <iyz>0</iyz>
              <izz>3.254</izz>
            </inertia>
          </inertial>
          <collision name='collision'>
            <geometry>
              <mesh>
                <uri>model://aws_robomaker_residential_CoffeeTable_01/meshes/aws_CoffeeTable_01_collision.DAE</uri>
                <scale>1 1 1</scale>
              </mesh>
            </geometry>
            <surface>
              <contact>
                <ode>
                  <max_vel>0.01</max_vel>
                </ode>
              </contact>
            </surface>
          </collision>
          <visual name='visual'>
            <geometry>
              <mesh>
                <uri>model://aws_robomaker_residential_CoffeeTable_01/meshes/aws_CoffeeTable_01_visual.DAE</uri>
              </mesh>
            </geometry>
            <meta>
              <layer>1</layer>
            </meta>
          </visual>
        </link>
        <static>true</static>
      </model>
      <pose>0.004101 -1.29441 0.041983 0 -0 0</pose>
      <self_collide>false</self_collide>
    </model>
    <model name='CoffeeTable_1_5'>
      <static>true</static>
      <model name='aws_robomaker_residential_CoffeeTable_01'>
        <link name='link'>
          <inertial>
            <mass>21.699999999999999</mass>
            <inertia>
              <ixx>0.81374999999999975</ixx>
              <ixy>0</ixy>
              <ixz>0</ixz>
              <iyy>2.76675</iyy>
              <iyz>0</iyz>
              <izz>3.254</izz>
            </inertia>
          </inertial>
          <collision name='collision'>
            <geometry>
              <mesh>
                <uri>model://aws_robomaker_residential_CoffeeTable_01/meshes/aws_CoffeeTable_01_collision.DAE</uri>
                <scale>1 1 1</scale>
              </mesh>
            </geometry>
            <surface>
              <contact>
                <ode>
                  <max_vel>0.01</max_vel>
                </ode>
              </contact>
            </surface>
          </collision>
          <visual name='visual'>
            <geometry>
              <mesh>
                <uri>model://aws_robomaker_residential_CoffeeTable_01/meshes/aws_CoffeeTable_01_visual.DAE</uri>
              </mesh>
            </geometry>
            <meta>
              <layer>1</layer>
            </meta>
          </visual>
        </link>
        <static>true</static>
      </model>
      <pose>2.22092 1.26113 0.041983 0 -0 0</pose>
      <self_collide>false</self_collide>
    </model>
    <model name='CoffeeTable_1_6'>
      <static>true</static>
      <model name='aws_robomaker_residential_CoffeeTable_01'>
        <link name='link'>
          <inertial>
            <mass>21.699999999999999</mass>
            <inertia>
              <ixx>0.81374999999999975</ixx>
              <ixy>0</ixy>
              <ixz>0</ixz>
              <iyy>2.76675</iyy>
              <iyz>0</iyz>
              <izz>3.254</izz>
            </inertia>
          </inertial>
          <collision name='collision'>
            <geometry>
              <mesh>
                <uri>model://aws_robomaker_residential_CoffeeTable_01/meshes/aws_CoffeeTable_01_collision.DAE</uri>
                <scale>1 1 1</scale>
              </mesh>
            </geometry>
            <surface>
              <contact>
                <ode>
                  <max_vel>0.01</max_vel>
                </ode>
              </contact>
            </surface>
          </collision>
          <visual name='visual'>
            <geometry>
              <mesh>
                <uri>model://aws_robomaker_residential_CoffeeTable_01/meshes/aws_CoffeeTable_01_visual.DAE</uri>
              </mesh>
            </geometry>
            <meta>
              <layer>1</layer>
            </meta>
          </visual>
        </link>
        <static>true</static>
      </model>
      <pose>2.21551 0.611993 0.041983 0 -0 0</pose>
      <self_collide>false</self_collide>
    </model>
    <model name='CoffeeTable_1_7'>
      <static>true</static>
      <model name='aws_robomaker_residential_CoffeeTable_01'>
        <link name='link'>
          <inertial>
            <mass>21.699999999999999</mass>
            <inertia>
              <ixx>0.81374999999999975</ixx>
              <ixy>0</ixy>
              <ixz>0</ixz>
              <iyy>2.76675</iyy>
              <iyz>0</iyz>
              <izz>3.254</izz>
            </inertia>
          </inertial>
          <collision name='collision'>
            <geometry>
              <mesh>
                <uri>model://aws_robomaker_residential_CoffeeTable_01/meshes/aws_CoffeeTable_01_collision.DAE</uri>
                <scale>1 1 1</scale>
              </mesh>
            </geometry>
            <surface>
              <contact>
                <ode>
                  <max_vel>0.01</max_vel>
                </ode>
              </contact>
            </surface>
          </collision>
          <visual name='visual'>
            <geometry>
              <mesh>
                <uri>model://aws_robomaker_residential_CoffeeTable_01/meshes/aws_CoffeeTable_01_visual.DAE</uri>
              </mesh>
            </geometry>
            <meta>
              <layer>1</layer>
            </meta>
          </visual>
        </link>
        <static>true</static>
      </model>
      <pose>2.20432 -0.031436 0.041983 0 -0 0</pose>
      <self_collide>false</self_collide>
    </model>
    <model name='CoffeeTable_1_8'>
      <static>true</static>
      <model name='aws_robomaker_residential_CoffeeTable_01'>
        <link name='link'>
          <inertial>
            <mass>21.699999999999999</mass>
            <inertia>
              <ixx>0.81374999999999975</ixx>
              <ixy>0</ixy>
              <ixz>0</ixz>
              <iyy>2.76675</iyy>
              <iyz>0</iyz>
              <izz>3.254</izz>
            </inertia>
          </inertial>
          <collision name='collision'>
            <geometry>
              <mesh>
                <uri>model://aws_robomaker_residential_CoffeeTable_01/meshes/aws_CoffeeTable_01_collision.DAE</uri>
                <scale>1 1 1</scale>
              </mesh>
            </geometry>
            <surface>
              <contact>
                <ode>
                  <max_vel>0.01</max_vel>
                </ode>
              </contact>
            </surface>
          </collision>
          <visual name='visual'>
            <geometry>
              <mesh>
                <uri>model://aws_robomaker_residential_CoffeeTable_01/meshes/aws_CoffeeTable_01_visual.DAE</uri>
              </mesh>
            </geometry>
            <meta>
              <layer>1</layer>
            </meta>
          </visual>
        </link>
        <static>true</static>
      </model>
      <pose>2.22269 -0.638146 0.041983 0 -0 0</pose>
      <self_collide>false</self_collide>
    </model>
    <model name='CoffeeTable_1_9'>
      <static>true</static>
      <model name='aws_robomaker_residential_CoffeeTable_01'>
        <link name='link'>
          <inertial>
            <mass>21.699999999999999</mass>
            <inertia>
              <ixx>0.81374999999999975</ixx>
              <ixy>0</ixy>
              <ixz>0</ixz>
              <iyy>2.76675</iyy>
              <iyz>0</iyz>
              <izz>3.254</izz>
            </inertia>
          </inertial>
          <collision name='collision'>
            <geometry>
              <mesh>
                <uri>model://aws_robomaker_residential_CoffeeTable_01/meshes/aws_CoffeeTable_01_collision.DAE</uri>
                <scale>1 1 1</scale>
              </mesh>
            </geometry>
            <surface>
              <contact>
                <ode>
                  <max_vel>0.01</max_vel>
                </ode>
              </contact>
            </surface>
          </collision>
          <visual name='visual'>
            <geometry>
              <mesh>
                <uri>model://aws_robomaker_residential_CoffeeTable_01/meshes/aws_CoffeeTable_01_visual.DAE</uri>
              </mesh>
            </geometry>
            <meta>
              <layer>1</layer>
            </meta>
          </visual>
        </link>
        <static>true</static>
      </model>
      <pose>2.21105 -1.28236 0.041983 0 -0 0</pose>
      <self_collide>false</self_collide>
    </model>
    <light name='sun' type='directional'>
      <pose>0 0 10 0 -0 0</pose>
      <cast_shadows>true</cast_shadows>
      <intensity>1</intensity>
      <direction>-0.5 0.1 -0.9</direction>
      <diffuse>0.8 0.8 0.8 1</diffuse>
      <specular>0.2 0.2 0.2 1</specular>
      <attenuation>
        <range>1000</range>
        <linear>0.01</linear>
        <constant>0.90000000000000002</constant>
        <quadratic>0.001</quadratic>
      </attenuation>
      <spot>
        <inner_angle>0</inner_angle>
        <outer_angle>0</outer_angle>
        <falloff>0</falloff>
      </spot>
    </light>
  </world>
</sdf>
```

---

---

## 🏗️ Step 24: Build & Launch the Simulation

### 🔨 1. Build your workspace

Make sure you are in the root of your ROS 2 workspace (e.g., `~/bot_ws`) and then run:

```bash
colcon build --packages-select bot_description
```

After building, **source your workspace**:

```bash
source install/setup.bash
```

### 🚀 2. Launch the robot in Gazebo with the table

Run the following launch command:

```bash
ros2 launch bot_description gazebo.launch.py
```

✅ What happens now:

* 🌍 The **`empty.world`** is loaded into Gazebo.
* 🪑 The **table model** from the `models/` folder is auto-loaded.
* 🤖 The **robot** spawns **on top of the table** using the `z=2.0` value set in the `gazebo.launch.py` file.
* 📡 The `/scan` and `/clock` topics are bridged for Gazebo–ROS 2 communication.

---

🎉 You should now see your robot standing proudly on the table inside Gazebo!

---
Here's how you can document **🧭 Step 25** in your `README.md` to **add controllers to your robot using `ros2_control`** and integrate them in the simulation using `ign_ros2_control`.

---

## ⚙️ Step 25: Add Controllers using `ros2_control` 🛠️

### 🧾 1. Create `ros2_control.xacro` in `urdf/` folder

Inside your `bot_description/urdf/` folder, create a file named:

```
ros2_control.xacro
```

Paste the following contents into it:

```xml
<?xml version="1.0"?>
<robot name="bot" xmlns:xacro="http://www.ros.org/wiki/xacro">

  <xacro:arg name="is_sim" default="true"/>

  <ros2_control name="RobotSystem" type="system">

    <xacro:property name="PI" value="3.14159265359" />

    <hardware>
      <plugin>ign_ros2_control/IgnitionSystem</plugin>
    </hardware>

    <joint name="right_wheel_joint">
      <command_interface name="velocity">
        <param name="min">-4.5</param>
        <param name="max">4.5</param>
      </command_interface>
      <state_interface name="position" />
      <state_interface name="velocity" />
    </joint>

    <joint name="left_wheel_joint">
      <command_interface name="velocity">
        <param name="min">-4.5</param>
        <param name="max">4.5</param>
      </command_interface>
      <state_interface name="position" />
      <state_interface name="velocity" />
    </joint>

  </ros2_control>

</robot>
```

🧩 This file defines the **hardware interface** using the `IgnitionSystem` plugin and adds **velocity control** for both drive joints.

---

### 🧬 2. Add the control plugin in `gazebo.xacro`

Open your `bot_description/urdf/gazebo.xacro` and add the following inside the `<robot>` tag (but outside of any `<link>` or `<gazebo reference>` tags):

```xml
<!-- ROS 2 Control -->
<gazebo>
    <plugin filename="ign_ros2_control-system" name="ign_ros2_control::IgnitionROS2ControlPlugin">
        <parameters>$(find bot_controller)/config/bot_controllers.yaml</parameters>
    </plugin>

    <plugin filename="ignition-gazebo-sensors-system" name="ignition::gazebo::systems::Sensors">
        <render_engine>ogre2</render_engine>
    </plugin>
</gazebo>
```

📌 This plugin:

* Loads your **ROS 2 controllers config** from `bot_controller/config/bot_controllers.yaml`.
* Enables **sensor support** in Ignition Gazebo.

## 🧩 Include `ros2_control.xacro` in Your Main URDF

Go to `bot.urdf.xacro` and include the gazebo file:

```xml
<xacro:include filename="$(find bot_description)/urdf/ros2_control.xacro"/>
```

📌 Place it below the other includes like `bot_base.xacro`, `bot_sensors.xacro` and `gazebo.xacro`

---

➡️ In the next step, we’ll define the actual controllers in a YAML config and create a new `bot_controller` package.

---

---

## 🧭 Step 26: Create and Configure the `bot_controller` Package 🎮🤖

### 🛠️ 1. Create the `bot_controller` package

In your ROS 2 workspace (`bot_ws/src`), run the following command to create a new package:

```bash
ros2 pkg create bot_controller --build-type ament_cmake
```

---

### 📁 2. Create necessary folders

Navigate to the new package and create two directories:

```bash
cd bot_controller
mkdir config launch
```

---

### 📝 3. Add controller config YAML

Inside the `config` folder, create a file named:

```
bot_controllers.yaml
```

Paste the following contents:

```yaml
controller_manager:
  ros__parameters:
    update_rate: 100

    wheel_controller:
      type: diff_drive_controller/DiffDriveController

    joint_state_broadcaster:
      type: joint_state_broadcaster/JointStateBroadcaster

wheel_controller:
  ros__parameters:
    type: diff_drive_controller/DiffDriveController

    use_stamped_vel: false
    left_wheel_names  : ['left_wheel_joint']
    right_wheel_names : ['right_wheel_joint']

    publish_rate: 50.0
    wheel_separation : 0.55
    wheel_radius : 0.15

    wheel_separation_multiplier: 1.0
    left_wheel_radius_multiplier: 1.0
    right_wheel_radius_multiplier: 1.0

    cmd_vel_timeout: 0.5
    base_frame_id: base_footprint
    publish_limited_velocity: true
    publish_wheel_data: true
    enable_odom_tf: true

    linear:
      x:
        has_velocity_limits    : true
        max_velocity           : 1.0
        min_velocity           : -1.0
        has_acceleration_limits: true
        max_acceleration       : 1.0
        min_acceleration       : -1.0
        has_jerk_limits        : false
        max_jerk               : 0.5

    angular:
      z:
        has_velocity_limits    : true
        max_velocity           : 2.5
        min_velocity           : -2.5
        has_acceleration_limits: true
        max_acceleration       : 2.5
        min_acceleration       : -2.5
        has_jerk_limits        : false
        max_jerk               : 2.24
```

---

### 🚀 4. Create a launch file

In the `launch/` folder, create:

```
controller.launch.py
```

Paste the following code:

```python
from launch import LaunchDescription
from launch_ros.actions import Node

def generate_launch_description():

    joint_state_broadcaster_spawner = Node(
        package="controller_manager",
        executable="spawner",
        arguments=[
            "joint_state_broadcaster",
            "--controller-manager",
            "/controller_manager",
        ],
    )

    wheel_controller_spawner = Node(
        package="controller_manager",
        executable="spawner",
        arguments=[
            "wheel_controller", 
            "--controller-manager", 
            "/controller_manager"
        ],
    )

    return LaunchDescription([
        joint_state_broadcaster_spawner,
        wheel_controller_spawner,
    ])
```

---

### ⚙️ 5. Configure `CMakeLists.txt`

Update your `CMakeLists.txt` as follows:

```cmake
cmake_minimum_required(VERSION 3.5)
project(bot_controller)

find_package(ament_cmake REQUIRED)
find_package(rclcpp REQUIRED)
find_package(rclpy REQUIRED)
find_package(geometry_msgs REQUIRED)
find_package(std_msgs REQUIRED)
find_package(sensor_msgs REQUIRED)
find_package(nav_msgs REQUIRED)
find_package(tf2_ros REQUIRED)
find_package(tf2 REQUIRED)
find_package(Eigen3 REQUIRED)
find_package(trajectory_msgs REQUIRED)
find_package(twist_mux_msgs REQUIRED)
find_package(rclcpp_action REQUIRED)

include_directories(include)
include_directories(${EIGEN3_INCLUDE_DIR})

install(
  DIRECTORY include
  DESTINATION include
)

install(
  DIRECTORY launch config
  DESTINATION share/${PROJECT_NAME}
)

ament_package()
```

---

### 📦 6. Update `package.xml`

Make sure your `package.xml` looks like this:

```xml
<?xml version="1.0"?>
<package format="3">
  <name>bot_controller</name>
  <version>0.0.0</version>
  <description>bot Controller Package</description>
  <maintainer email="kutkarsh706@gmail.com">Utkarsh</maintainer>
  <license>Apache 2.0</license>
  <author email="kutkarsh706@gmail.com">utk</author>

  <buildtool_depend>ament_cmake</buildtool_depend>
  <buildtool_depend>ament_cmake_python</buildtool_depend>

  <depend>rclcpp</depend>
  <depend>rclpy</depend>
  <depend>geometry_msgs</depend>
  <depend>std_msgs</depend>
  <depend>sensor_msgs</depend>
  <depend>nav_msgs</depend>
  <depend>tf2_ros</depend>
  <depend>tf2</depend>
  <depend>eigen</depend>
  <depend>trajectory_msgs</depend>
  <depend>twist_mux_msgs</depend>
  <depend>rclcpp_action</depend>
  
  <exec_depend>tf_transformations</exec_depend>
  <exec_depend>ros2launch</exec_depend>
  <exec_depend>robot_state_publisher</exec_depend>
  <exec_depend>xacro</exec_depend>
  <exec_depend>controller_manager</exec_depend>
  <exec_depend>ros2_controllers</exec_depend>
  <exec_depend>ros2_control</exec_depend>
  <exec_depend>joint_state_publisher_gui</exec_depend>

  <test_depend>ament_lint_auto</test_depend>
  <test_depend>ament_lint_common</test_depend>

  <export>
    <build_type>ament_cmake</build_type>
  </export>
</package>
```

---

✅ Once all files are in place, you're ready for the next step: **building the workspace** and **launching the robot with controllers**.

---

---

## 🕹️ Step 27: Teleoperate the Robot in Gazebo using Keyboard 🧭🧑‍💻

After setting up your simulation and controllers, let’s now move to teleoperating the robot using keyboard commands via `teleop_twist_keyboard`.

---

### ✅ 1. Launch the robot in Gazebo

In **Terminal 1**, launch the Gazebo simulation with your robot:

```bash
ros2 launch bot_description gazebo.launch.py
```

---

### ✅ 2. Launch the controllers

In **Terminal 2**, spawn the controller manager and robot controllers:

```bash
ros2 launch bot_controller controller.launch.py
```

---

### ✅ 3. Install keyboard teleop tool (if not already installed)

In **Terminal 3**, run:

```bash
sudo apt install ros-humble-teleop-twist-keyboard
```

Replace `${ROS_DISTRO}` with your ROS 2 distribution name (e.g., `humble`, `iron`, etc.).

---

### ✅ 4. Run the teleop node

Still in **Terminal 3**, run the teleop node and remap to your robot's velocity topic:

```bash
ros2 run teleop_twist_keyboard teleop_twist_keyboard \
  --ros-args -p stamped:=false -r cmd_vel:=wheel_controller/cmd_vel_unstamped
```

---

### 🕹️ 5. Control your robot using your keyboard!

Once the teleop node is running, use the following keys to control the robot:

```
Moving around:
   u    i    o
   j    k    l
   m    ,    .

i: move forward  
k: stop  
,: move backward  
j/l: turn left/right

CTRL-C to quit
```

---

✅ You should now be able to see your robot moving inside Gazebo based on your keyboard input!

---

---

## 🚀 Step 28: Create `bot_bringup` Package to Launch Entire Stack

To streamline launching your simulation, controllers, and optionally any custom scripts or RViz config, we’ll create a centralized **launch package**.

---

### ✅ 1. Create the bringup package

```bash
cd ~/bot_ws/src
ros2 pkg create bot_bringup --build-type ament_cmake
```

---

### ✅ 2. Create the launch folder

```bash
cd bot_bringup
mkdir launch
```

---

### ✅ 3. Create `simulated_robot.launch.py`

Inside the `launch/` folder, create the file `simulated_robot.launch.py`:

```python
import os
from launch import LaunchDescription
from launch.actions import IncludeLaunchDescription, DeclareLaunchArgument
from launch.substitutions import LaunchConfiguration
from launch_ros.actions import Node
from ament_index_python.packages import get_package_share_directory
from launch.launch_description_sources import PythonLaunchDescriptionSource

def generate_launch_description():
    world_name = LaunchConfiguration("world_name")

    world_name_arg = DeclareLaunchArgument(
        "world_name",
        default_value="empty_table"
    )

    gazebo = IncludeLaunchDescription(
        PythonLaunchDescriptionSource(os.path.join(
            get_package_share_directory("bot_description"),
            "launch",
            "gazebo.launch.py"
        )),
        launch_arguments={
            "world_name": world_name
        }.items()
    )

    controller_only = IncludeLaunchDescription(
        PythonLaunchDescriptionSource(os.path.join(
            get_package_share_directory("bot_controller"),
            "launch",
            "controller.launch.py"
        ))
    )

    rviz = Node(
        package="rviz2",
        executable="rviz2",
        arguments=["-d", os.path.join(
            get_package_share_directory("bot_description"),
            "rviz",
            "display.rviz"
        )],
        output="screen",
        parameters=[{"use_sim_time": True}]
    )

    edge_avoidance_node = Node(
        package="bot_script",
        executable="edge_detection",
        name="edge_avoidance_node",
        output="screen",
        parameters=[{"use_sim_time": True}]
    )

    return LaunchDescription([
        world_name_arg,
        gazebo,
        controller_only,
        # edge_avoidance_node,
        rviz,
    ])
```

Uncomment lines as needed to launch custom logic or visualization.

---

### ✅ 4. Update `CMakeLists.txt`

```cmake
cmake_minimum_required(VERSION 3.8)
project(bot_bringup)

if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic)
endif()

find_package(ament_cmake REQUIRED)

install(
  DIRECTORY launch
  DESTINATION share/${PROJECT_NAME}
)

ament_package()
```

---

### ✅ 5. Update `package.xml`

```xml
<?xml version="1.0"?>
<package format="3">
  <name>bot_bringup</name>
  <version>0.0.0</version>
  <description>Launch everything for bot simulation</description>
  <maintainer email="kutkarsh706@gmail.com">utk</maintainer>
  <license>Apache-2.0</license>

  <buildtool_depend>ament_cmake</buildtool_depend>

  <exec_depend>rviz2</exec_depend>
  <exec_depend>bot_controller</exec_depend>
  <exec_depend>bot_description</exec_depend>
  <exec_depend>bot_script</exec_depend>

  <export>
    <build_type>ament_cmake</build_type>
  </export>
</package>
```

---

### ✅ 6. Build and source the workspace

```bash
cd ~/bot_ws
colcon build --symlink-install
source install/setup.bash
```

---

### ✅ 7. Launch the full system from one terminal 🎯

```bash
ros2 launch bot_bringup simulated_robot.launch.py
```

You now have one command that:

* Launches your robot in the Gazebo world
* Starts the controllers
* Optionally starts RViz and your custom script

---
---

## 🤖 Step 29: Create `bot_script` Package (for Custom Logic like Edge Detection)

We'll now create a Python-based ROS 2 package to run robot logic nodes — such as edge detection, obstacle avoidance, or navigation scripts.

---

### ✅ 1. Create the package

```bash
cd ~/bot_ws/src
ros2 pkg create bot_script --build-type ament_python
```

---

### ✅ 2. Create the edge detection script

Go into the Python module folder and add the script:

```bash
cd bot_script/bot_script
touch edge_detection.py __init__.py
```

Make both files executable:

```bash
chmod +x edge_detection.py __init__.py
```

> You'll add logic inside `edge_detection.py` later.

---

### ✅ 3. Edit `package.xml`

Update the contents of `bot_script/package.xml`:

```xml
<?xml version="1.0"?>
<package format="3">
  <name>bot_script</name>
  <version>0.0.0</version>
  <description>Edge detection and behavior scripts for the bot</description>
  <maintainer email="kutkarsh706@gmail.com">utk</maintainer>
  <license>Apache-2.0</license>

  <depend>rclpy</depend>
  <depend>geometry_msgs</depend>
  <depend>sensor_msgs</depend>
  <depend>gazebo_msgs</depend>

  <test_depend>ament_copyright</test_depend>
  <test_depend>ament_flake8</test_depend>
  <test_depend>ament_pep257</test_depend>
  <test_depend>python3-pytest</test_depend>

  <export>
    <build_type>ament_python</build_type>
  </export>
</package>
```

---

### ✅ 4. Edit `setup.py`

Replace `bot_script/setup.py` with:

```python
from setuptools import find_packages, setup

package_name = 'bot_script'

setup(
    name=package_name,
    version='0.0.0',
    packages=find_packages(exclude=['test']),
    data_files=[
        ('share/ament_index/resource_index/packages',
            ['resource/' + package_name]),
        ('share/' + package_name, ['package.xml']),
    ],
    install_requires=['setuptools'],
    zip_safe=True,
    maintainer='utk',
    maintainer_email='kutkarsh706@gmail.com',
    description='Edge detection and bot behavior logic',
    license='Apache-2.0',
    tests_require=['pytest'],
    entry_points={
        'console_scripts': [
            'edge_detection = bot_script.edge_detection:main',
        ],
    },
)
```

---

### ✅ 5. Final folder structure (recap)

```bash
bot_ws/src/bot_script/
├── bot_script/
│   ├── __init__.py
│   └── edge_detection.py
├── package.xml
└── setup.py
```

---


✅ You're now ready to implement the logic inside `edge_detection.py` in the next step!

---

## ✅ Step 30: Write Final Edge Detection Node and Run Complete Project

---

### 🧠 1. Final `edge_detection.py` Node

Paste this into `bot_ws/src/bot_script/bot_script/edge_detection.py`:

```python
#!/usr/bin/env python3
import rclpy
from rclpy.node import Node
from sensor_msgs.msg import LaserScan
from geometry_msgs.msg import Twist
from time import sleep

class EdgeAvoidanceBot(Node):
    def __init__(self):
        super().__init__('edge_avoidance_bot')
        
        self.publisher = self.create_publisher(Twist, '/wheel_controller/cmd_vel_unstamped', 10)
        self.subscription = self.create_subscription(LaserScan, '/scan', self.lidar_callback, 10)
        self.timer = self.create_timer(0.1, self.control_loop)
        self.cmd_vel = Twist()

        self.up = 0.0
        self.down = 0.0

        self.flag = False

    def lidar_callback(self, msg):
        # Extract ranges
        ranges = msg.ranges
        valid_ranges = [r if r > 0.05 and r < 4.0 else 4.0 for r in ranges]

        self.up = valid_ranges[6]
        self.down = valid_ranges[2]
        #print(len(valid_ranges))

        # Log the regions
        self.get_logger().info(f"Distances - U: {self.up:.2f}, - D: {self.down:.2f}")

    def control_loop(self):
        # Default forward motion
        self.cmd_vel.linear.x = 0.3
        self.cmd_vel.angular.z = 0.0

        #Edge Detection
        if self.down > 0.50:
            self.flag = True #Edge Detected

        if self.flag == True and self.up > 0.60:
            self.cmd_vel.linear.x = -0.4
            self.cmd_vel.angular.z = -1.65
            self.get_logger().info("Edge ahead! Reversing and turning.")
        else:
            self.flag = False



        # Send command
        self.publisher.publish(self.cmd_vel)

def main(args=None):
    rclpy.init(args=args)
    bot = EdgeAvoidanceBot()
    rclpy.spin(bot)
    rclpy.shutdown()

if __name__ == '__main__':
    main()
```

---

### ✅ 2. Rebuild the Workspace

After saving the script:

```bash
cd ~/bot_ws
colcon build --symlink-install
source install/setup.bash
```

---

### 🚀 3. Launch the Full Project

#### Terminal 1: Launch Simulation + Controllers via `bot_bringup`

```bash
ros2 launch bot_bringup simulated_robot.launch.py
```

#### Terminal 2: Run Edge Detection Logic

```bash
ros2 run bot_script edge_detection
```

✅ Your robot will now **automatically stop near edges/cliffs** detected using `/scan` data, and otherwise drive forward. You can integrate additional logic later (like rotating on edge detection, etc.).

---

### 🎉 Project Completed in 30 Steps!

You now have a:

* ✅ Fully simulated Gazebo robot on a table
* ✅ Integrated ROS 2 Control with diff drive
* ✅ Keyboard teleop support
* ✅ Custom edge detection script reacting to lidar
* ✅ One-command launch via bringup file

---

### Congratulations on completing this build! 🥳

---























