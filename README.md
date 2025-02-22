# Compliant Control and Application

  It contains the compliant control algorithms in robotic arm, and the chosen robot is **universal robot** which is popular collaborate robot in the world.

## Compile

```bash
mkdir catkin_ws/src && cd catkin_ws/src
git clone https://github.com/MingshanHe/Compliant-Control-and-Application.git
catkin build (or cd .. && catkin_make)
```
If there is come up with some errors during compile process, you might need to install some ros packages for support this project, I recommend you to run following command or search the compile error in Google or Baidu and so on.
```bash
rosdep install -i --from-path src --rosdistro noetic --ignore-src -r -y
```

If there is an error 'Unable to locate ...', try following commands.
```bash
echo "deb http://packages.ros.org/ros/ubuntu $(lsb_release -sc) main" | sudo tee /etc/apt/sources.list.d/ros-latest.list
sudo apt update
```
## Change the arm to Xarm6
1. Clone xarm repository [https://github.com/xArm-Developer/xarm_ros]
1. Copy stl files in `xarm_ros/xarm_description/meshes/xarm6/visual/` to `Compliant-Control-and-Application/robots/universal_robot/fmauch_universal_robot/ur_description/meshes/ur5e/visual/` adding `_xarm` to the end of file names.
1. Rename the values of `visual` and `collision` in `Compliant-Control-and-Application/robots/universal_robot/fmauch_universal_robot/ur_description/config/ur5e/visual_parameters.yaml` to the xarm's stl files.
1. Copy the values in `xarm_ros/xarm_description/urdf/xarm6.urdf.xacro` to `Compliant-Control-and-Application/robots/universal_robot/fmauch_universal_robot/ur_description/urdf/inc/ur_macro.xacro`
    1. Add the definitions such as `joint1_lower_limit` to `<xacro:macro name="ur_robot" params="`.
    1. Do not change the name of links and joints as defined in `ur_macro.xacro`.

## Check

using the following command to check the self-defined controller. Like:`cartesian_velocity_controller`

```bash
rospack plugins --attrib=plugin controller_interface
```

## Run

  **Notice**: In this repository, I have used ur5e robot and its `urdf` file need to be changed in different situation, like need or not a force/torque sensor in the end effector. Please check the `urdf` file seriously and run the algorithm, Thanks.

### 1. Admittance

```bash
roslaunch ur_gazebo ur5e_bringup.launch transmission_hw_interface:=hardware_interface/PositionJointInterface specified_controller:=cartesian_velocity_controller
```

```bash
roslaunch admittance Admittance.launch
```

After the robot has moved to the desired pose, run the following commands to generate fake wrench signal

```bash
roslaunch admittance Wrench_Fake.launch
```

### 2. Impedance

```bash
roslaunch ur_gazebo ur5e_bringup.launch transmission_hw_interface:=hardware_interface/EffortJointInterface specified_controller:=joint_torque_controller
```

```
roslaunch impedance Impedance.launch
```

### 3. Application (Hybrid Admittance Control)

```bash
roslaunch mir_gazebo mir_ur5e.launch tf_prefix:=robot1
```

```
roslaunch admittance HybridAdmittance.launch tf_prefix:=robot1
```

### 4. Hybrid Position Force Control

```bash
roslaunch ur_gazebo ur5e_bringup.launch transmission_hw_interface:=hardware_interface/PositionJointInterface specified_controller:=cartesian_velocity_controller environment:=polish
```

```bash
roslaunch hybrid_position_force_control hybrid_position_force_control.launch
```

  And then it needs to use the topic publish command in the terminal. It is recommended to move to the pose [-0.1, 0.3, 0.3, 0.707, -0.707, 0.0, 0.0]. 

```bash
rostopic pub /desired_carteisan_pose_cmd geometry_msgs/Pose "position: x: -0.10 y: 0.30 z: 0.30 orientation: x: 0.707 y: -0.707 z: 0.0 w: 0.0" 
```

### 5. Mobile Robot
```bash
roslaunch mir_gazebo mir.launch
```
There are some pre-settings that will be needed to run in this project. Three mode is provided to be tested.
1) Mobile Robot(mir robot)
2) Mobile Robot(mir robot) with a Robotic Arm(UR5E)
3) Mobile Robot(mir robot) with a Robotic Arm(UR5E) and a Gripper(Robotiq)

For change amoung them, this project have provided the setting params in the `mir.launch` file. And you also need to change the start controller for the robotic arm in `mir_gazebo_common.launch`
#### Mapping
`Gmapping:`
```bash
roslaunch mir_navigation hector_mapping.launch
```
`MoveBase Framework:`
```bash
roslaunch mir_navigation move_base.xml with_virtual_walls:=false
```
```bash
rviz -d $(rospack find mir_navigation)/rviz/navigation.rviz
```
#### Navigation
`AMCL Localization:`
```bash
roslaunch mir_navigation amcl.launch initial_pose_x:=10.0 initial_pose_y:=10.0
```
`MoveBase Framework:`
```bash
roslaunch mir_navigation start_planner.launch \
    map_file:=$(rospack find mir_gazebo)/maps/maze.yaml \
    virtual_walls_map_file:=$(rospack find mir_gazebo)/maps/maze_virtual_walls.yaml
```
```bash
rviz -d $(rospack find mir_navigation)/rviz/navigation.rviz
```

## Performance

### 1. Admittance

![1](Image/Admittance.gif)

### 2. Hybrid Admittance Control
![2](Image/Hybrid_Admittance.gif)

### 3. Hybrid Position Force Control

![3](Image/Hybrid_Position_Force_Control.gif)

