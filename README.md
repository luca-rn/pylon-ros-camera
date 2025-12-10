# ROS2-Driver for Basler Cameras

An adapted version of the official pylon ROS2 driver for [Basler](http://www.baslerweb.com/) GigE Vision, Basler USB3 Vision and Basler blaze 3D cameras (Humble Hawksbill)

This driver provides many functionalities available through the Basler [pylon Camera Software Suite](https://www.baslerweb.com/en/products/software/basler-pylon-camera-software-suite/) C++ API.

## Installation (Notes from [_Interfacing Basler Cameras with ROS 2_](https://rjwilson.com/wp-content/uploads/Interfacing-Basler-Cameras-with-ROS-2-RJ-Wilson-Inc.pdf))
### Prerequisites (Included in Instructions)
- From [Ubuntu 22.04 Jammy Jellyfish](https://releases.ubuntu.com/jammy/)
- From [ROS2 Humble Hawksbill](https://docs.ros.org/en/humble/Installation/Ubuntu-Install-Debians.html). Your ROS2 environment must be [configured](https://docs.ros.org/en/humble/Tutorials/Beginner-CLI-Tools/Configuring-ROS2-Environment.html), your workspace [created](https://docs.ros.org/en/humble/Tutorials/Beginner-Client-Libraries/Creating-A-Workspace/Creating-A-Workspace.html), and colcon, used to build the packages, [installed](https://docs.ros.org/en/humble/Tutorials/Beginner-Client-Libraries/Colcon-Tutorial.html).
- [rosdep](https://docs.ros.org/en/humble/Tutorials/Intermediate/Rosdep.html). rosdep must be installed as a debian package (`sudo apt update && sudo apt install python3-rosdep2 && sudo rosdep init && rosdep update`).
- From [pylon Camera Software Suite](https://www2.baslerweb.com/en/downloads/software-downloads/) version 7.2 or newer. The latest APi libraries must be installed manually. Download and install the latest pylon Camera Software Suite Linux Debian Installer Package for your architecture. You may be experiencing some problems with the codemeter debian package installation. Just drop it for now and install only the pylon debian package in this case.
- From [pylon Supplementary Package for blaze](https://www2.baslerweb.com/en/downloads/software-downloads/) version 1.3 or newer (compatibility with the installed pylon Camera Software Suite needs to be ensured, please refer to the documentation). The latest APi libraries must be installed manually. Download and install the latest pylon Supplementary Package for blaze Linux Debian Installer Package for your architecture.
- [Git](https://git-scm.com/). Git must be installed as a debian package (`sudo apt update && sudo apt install git`).
- [xterm](https://invisible-island.net/xterm/). The xterm terminal emulator must be installed (refer to the *Know Issues* section below) as a debian package (`sudo apt update && sudo apt install xterm`).
### Pylon Installation
The [pylon-ros2-camera driver package](https://github.com/basler/pylon-ros-camera/tree/humble) requires that the library of pylon version 6.2 or newer is
installed. If you need to install a suitable pylon version, continue with the following steps. Otherwise, if you already have Pylon installed, continue with [Setting up the Driver in ROS2](#setting-up-the-pylon-camera-driver-in-ros-2).

1. Visit the [Basler software downloads](https://www.baslerweb.com/en/downloads/software/) page.\
2. Download the appropriate pylon version package for your OS. The install notes (downloadable) on the downloads page for each pylon version are useful for this step.\
  __For use with the supplementary package for Basler Blaze, it may be necessary to downloaded an outdated Pylon version. Check the requirements before installing.__\
  \
  For our system we downloaded pylon 7.3.0 with the .tar.gz for a Debian based ARM 64-bit system. This was used alongside the pylon Supplementary Package for blaze 1.7.3.\
  \
  If you're using a Debian-based Linux distribution (e.g., Ubuntu) you can choose one of the corresponding Debian packages provided with this pylon release. Alternatively, you can always use the tar.gz files, which will also work for Linux distributions not based on Debian.\
  \
  __If you downloaded a debian/.deb package:__\
  On many Debian-based Linux distributions, you can install the Debian package by double-clicking the file or with the command `sudo dpkg -i route\to\deb\install\pylon_X.X.X.XXXXX-deb0_arm64.deb`. Check the pylon root location environment variable and make sure it exists (using `echo $PYLON_ROOT`). If not, type the following `echo “export PYLON_ROOT=/opt/pylon” >> ~/.bashrc`. Check again with `echo $PYLON_ROOT`and the output should be `opt/pylon`.\
  \
  Alternatively, follow these steps:\
    a. Change to the directory that contains the pylon Debian package.\
    b. Install the Debian packages: `sudo apt-get install ./pylon_*.deb ./codemeter*.deb`\
  \
  During the installation, an environment variable required for pylon GenTL producers and a permission file for Basler USB cameras are installed automatically. For this to take effect, you need to log out and log in again toyour Linux system as well as unplug and replug all USB cameras.\
  \
  __If you downloaded a .tar.gz package:__\
  Details about installation and configuration are available from the included INSTALL and README files.

### Setting up the pylon camera driver in ROS 2
1. Clone this repository into your src folder
  ```bash
  cd ~/dev_ws/src/ && git clone https://github.com/luca-rn/pylon-ros-camera.git
  ```
2. (Not Tested) Clone any necessary additional packages. For example packages from ros-perception. Example given by basler is image_common.git.
  ```bash
  cd ~/dev_ws/src/pylon_ros2_camera && git clone –b humble https://github.com/rosperception/image_common.git image_common
  ```
3. Install mandatory dependencies
  ```bash
  cd ~/dev_ws && sudo rosdep install --frompaths src --ignore-src –r -y
  ```
4. Build the workspace using colcon build
  ```bash
  cd ~/dev_ws && colcon build
  ```
5. Permanent setup of environment settings
  ```bash
  echo “source ~/dev_ws/install/setup.bash” >> ~/.bashrc
  ```
  Then, open a new terminal or run
  ```bash
  source ~/.bashrc
  ```
6. Running the package
  ```bash
  ros2 launch pylon_ros2_camera_wrapper pylon_ros2_camera.launch.py
  ```
  This automatically uses the first camera model that is found by underlaying pylon API. If no camera can be found, it will create an error.

## Camera configuration
The Basler cameras must be configured with a suitable IP for connection with the ROS nodes. This can be easily achieved with the pylon IP configurator.
1) Connect the camera via ethernet, ensure it is receiving power
2) Open the pylon ip configurator
3) The connected camera should show up in the list of devices connected via ethernet. Select this device and give it an appropriate IP and name.\
  For Basler cameras, an IPV4 address `192.168.5.XX` is recommended, with the subnet mask `255.255.255.0` and ensure that the changes are saved.\
  For our Basler Ace2 camera used the ip `192.168.5.10` and the device name BaslerAce1. An address and name of the same format was used for our Blaze camera as well.
4) In terminal, use the command `ip addr` to check the your network connections. If an inet with an address  `192.168.5.XX` is listed under your ethernet connections, you may be able to connect to your camera in ROS immediately. Otherwise, you must add such an address manually.
5) Go to network settings on your device and click to edit a network connection. For our project, we created a new connection as we had to add ip addresses for multiple devices.
6) Add in IPV4 address `192.168.5.XX` (we used `192.168.5.2`) and the subnet mask `255.255.255.0`.
7) Ensure that the changes are persisted.\
\
This should be enough to enable that the camera is able to connect in ROS 2, it is recommended that you now test this in your ROS 2 workspace with the [camera launch command](#using-the-pylon-ros2-camera-wrapper).\
\
If the camera can still not be found, it is recommended to edit or create a new `.yaml` file for the camera configuration. this can be found in `/pylon_ros2_camera/pylon_ros2_camera_wrapper/config`. Set the device name to that which you assigned to your device in the .yaml file. If you are using an RGB-enabled camera, you may also want to set the image encoding to RGB-8 or another coloured encoding.\
\
In the pylon suite, it is possible to make a custom configuration for the camera settings. This can be used to set your desired configuration. In our experience, it was necessary to change the image encoding from greyscale to RGB in order to obtain a colour output from our basler ace camera. The use this configuration in ROS2, follow these steps:
1) Save the custom configuration in the pylon suite and remember the name.
2) Open the file, pylon_ros2_camera.launch.py in the launch directory of the pylon_ros2_camera_wrapper.
3) Find the function DeclareLaunchArgument.
4) Change the default_value parameter to the configuration that matches the name of your custom configuration (UserSet1, UserSet2 or UserSet3).
5) Save the file and use colcon to build your workspace again
```bash
colcon build --packages-select pylon_ros2_camera
```

## Using the pylon ros2 camera wrapper
For the basler ace:
```bash
ros2 launch pylon_ros2_camera_wrapper pylon_ros2_camera.launch.py
```
For the basler blaze:
```bash
ros2 launch pylon_ros2_camera_wrapper my_blaze.launch.py
```
Use `ros2 topic list` in another terminal to check if nodes run correctly and `rviz2` to view the camera streams.

# Changes from original Basler Repo
- Changes in launch and .yaml files in pylon_ros2_camera_wrapper for RBG encoding and device name
- Our own installation instructions in this readme file

For more extensive documentation, see the [original repository from Basler](https://github.com/basler/pylon-ros-camera.git)

