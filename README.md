## Articubot by Articulated Robotics (Humble Version)

I'm also using [his GitHub template](https://github.com/joshnewans/my_bot).

---

### Both ws
- Install Arduino IDE, ROS2 Humble (via Debian, Desktop version)
- Add sourcing to shell startup script: echo "source /opt/ros/humble/setup.bash" >> ~/.bashrc
~~~
git clone https://github.com/joshnewans/serial_motor_demo.git
~~~
~~~
sudo apt install git python3-colcon-common-extensions ros-humble-xacro ros-humble-image-transport-plugins ros-humble-rqt-image-view ros-humble-ros2-control ros-humble-ros2-controllers 
~~~

### dev_ws
~~~
sudo snap install --classic code
sudo apt install ros-humble-joint-state-publisher-gui ros-humble-gazebo-ros2-control joystick jstest-gtk evtest ros-humble-navigation2 ros-humble-nav2-bringup ros-humble-twist-mux

~/dev_ws/src$ git clone -b humble https://github.com/ros-controls/gazebo_ros2_control

cp /opt/ros/humble/share/nav2_bringup/launch/navigation_launch.py src/articubot_humble/launch/navigation_launch.py
cp /opt/ros/humble/share/nav2_bringup/params/nav2_params.yaml src/articubot_humble/config/nav2_params.yaml
~~~ 
### robot_ws
- Install: git, openssh-server, Arduino IDE
- Power off the pi before connect lidar to it
- Before power off the pi, shut the ssh connection: sudo shutdown -h now
- [Set up LD06 small lidar](https://www.youtube.com/watch?v=OJWAsV6-0GE)
~~~
sudo apt install libraspberrypi-bin v4l-utils ros-humble-v4l2-camera python3-serial

git clone https://github.com/joshnewans/diffdrive_arduino.git
git clone https://github.com/joshnewans/serial.git
~~~

---

## Simulation

Run Gazebo:
~~~
ros2 launch articubot_humble launch_sim.launch.py world:=src/articubot_humble/worlds/obstacles.world
~~~

Run Rviz:
~~~
ros2 run rviz2 rviz2 -d src/articubot_humble/config/main.rviz --ros-args -p use_sim_time:=true
~~~

Run teleop:
~~~
ros2 run teleop_twist_keyboard teleop_twist_keyboard --ros-args -r /cmd_vel:=/diff_cont/cmd_vel_unstamped
~~~

Run slam_toolbox:
~~~
ros2 launch slam_toolbox online_async_launch.py params_file:=./src/articubot_humble/config/mapper_params_online_async.yaml use_sim_time:=true
~~~

Run SLAM: 
~~~
ros2 launch articubot_humble online_async_launch.py use_sim_time:=true
~~~

Run Nav2: 
~~~
ros2 launch articubot_humble navigation_launch.py use_sim_time:=true
~~~

Run Localization with amcl:
~~~
ros2 launch nav2_bringup localization_launch.py map:=./y_map_save.yaml use_sim_time:=true

#Rerun Gazebo (same cmd) and Rviz with:
# Fixed frame: map (if don't have in the dropdown then just type in yourself)
# Map --> Topic --> Reliability Policy: Reliable, Durability Policy: Transient Local

ros2 run rviz2 rviz2 -d src/articubot_humble/config/localization.rviz --ros-args -p use_sim_time:=true

# Then choose 2D Pose Estimate to give the AGV position

~~~

---
## Real life ver

### Camera node
~~~
#robot_ws:
ros2 launch camera.launch.py

#dev_ws:
ros2 run rqt_image_view rqt_image_view
~~~
### Lidar node
~~~
#robot_ws:
ros2 launch rplidar.launch.py

#dev_ws:
rviz2
~~~
### Motor node
~~~
#robot_ws:
ros2 run serial_motor_demo driver --ros-args -p serial_port:=/dev/... -p baud_rate:= ... -p loop_rate:= ... -p encoder_cpr:= ...

#dev_ws:
ros2 run serial_motor_demo gui 
~~~
