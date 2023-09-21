# Ros2 Humble Build
This Tutorial is based of [ROS 2 Documentation: HumbleLogo](https://docs.ros.org/en/humble/Installation/Alternatives/Ubuntu-Development-Setup.html#try-some-examples)
| Name   	| Version  	|
|--------	|----------	|
| Debian 	| Bullseye 	|
| ROS2   	| Humble   	|


## System setup

### Set locale

Make sure you have a locale which supports UTF-8. If you are in a minimal environment (such as a docker container), the locale may be something minimal like POSIX. We test with the following settings. However, it should be fine if you’re using a different UTF-8 supported locale.

```shell
locale  # check for UTF-8

sudo apt update && sudo apt install locales
sudo locale-gen en_US en_US.UTF-8
sudo update-locale LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8
export LANG=en_US.UTF-8

locale  # verify settings
```

## Add the ROS 2 apt repository

You will need to add the ROS 2 apt repository to your system. First, make sure that the **Ubuntu Universe**(Raspberry os is based of Debian, so we don't have **Universe**) repository is enabled by checking the output of this command.

Now add the ROS 2 apt repository to your system. First authorize our GPG key with apt.

```shell
sudo apt update && sudo apt install curl gnupg lsb-release
sudo curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key -o /usr/share/keyrings/ros-archive-keyring.gpg
```

Then add the repository to your sources list.
```shell
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] http://packages.ros.org/ros2/ubuntu $(source /etc/os-release && echo $VERSION_CODENAME) main" | sudo tee /etc/apt/sources.list.d/ros2.list > /dev/null
```

### Install development tools and ROS tools

```shell
sudo apt update && sudo apt install -y \
  build-essential \
  cmake \
  git \
  python3-colcon-common-extensions \
  python3-flake8 \
  python3-flake8-docstrings \
  python3-pip \
  python3-pytest \
  python3-pytest-cov \
  python3-pytest-rerunfailures \
  python3-rosdep \
  python3-setuptools \
  python3-vcstool \
  wget

pip install \
  flake8-blind-except \
  flake8-builtins \
  flake8-class-newline \
  flake8-comprehensions \
  flake8-deprecated \
  flake8-import-order \
  flake8-quotes \
  pytest-repeat
```

## Get ROS 2 code

Create a workspace and clone all repos:

```shell
mkdir -p ~/ros2_humble/src
cd ~/ros2_humble
wget https://raw.githubusercontent.com/ros2/ros2/humble/ros2.repos
vcs import src < ros2.repos
```

## Install dependencies using rosdep (rosdep can not solve all dependices)

ROS 2 packages are built on frequently updated Ubuntu systems. It is always recommended that you ensure your system is up to date before installing new packages.

```shell
sudo apt upgrade
```

```shell
sudo rosdep init
rosdep update
rosdep install --from-paths src --ignore-src -y --skip-keys "fastcdr rti-connext-dds-6.0.1 urdfdom_headers ignition-cmake2 ignition-math6" --rosdistro humble --os=ubuntu:focal
```
rosdep can not install ignition-cmake2 and ignition-math6. So we have to build them by ourself. First we have to build camke2, after that build math6.


```shell
# cmake2
mkdir ~/temp
cd ~/temp
git clone https://github.com/gazebosim/gz-cmake.git
cd gz-cmake
mkdir build && cd build 
cmake ..
sudo make install
```
Don't follow the **Prepare for Installing** on the github of math6. After Pull from github just compile math6 directly.

```shell
# math6
cd ~/temp
git clone https://github.com/gazebosim/gz-math.git
sudo apt-get install libeigen3-dev
cd gz-math
mkdir build && cd build 
cmake ..
sudo make install
cd ~/temp
```

## Build the code in the workspace
If you have already installed ROS 2 another way (either via Debians or the binary distribution), make sure that you run the below commands in a fresh environment that does not have those other installations sourced. Also ensure that you do not have ``source /opt/ros/${ROS_DISTRO}/setup.bash`` in your .bashrc. You can make sure that ROS 2 is not sourced with the command ``printenv | grep -i ROS``. The output should be empty.

For Raspberry Pi you have to wait for about 12h. if you don't need rviz. You can create a file under /src/ros2/rviz with name ``COLCON_IGNORE`` to ignore it. It will save about 4-5h for you. 
```
cd ~/ros2_humble/
colcon build --symlink-install
```

Note: if you are having trouble compiling all examples and this is preventing you from completing a successful build, you can use ``COLCON_IGNORE`` in the same manner as [CATKIN_IGNORE](https://github.com/ros-infrastructure/rep/blob/master/rep-0128.rst) to ignore the subtree or remove the folder from the workspace. Take for instance: you would like to avoid installing the large OpenCV library. Well then simply run ``touch COLCON_IGNORE`` in the ``cam2image`` demo directory to leave it out of the build process.

## Environment setup

### Source the setup script

Set up your environment by sourcing the following file.

```shell
# Replace ".bash" with your shell if you're not using bash
# Possible values are: setup.bash, setup.sh, setup.zsh
. ~/ros2_humble/install/local_setup.bash
```

## Try some examples
In one terminal, source the setup file and then run a C++ ``talker``:
```shell
. ~/ros2_humble/install/local_setup.bash
ros2 run demo_nodes_cpp talker
```
In another terminal source the setup file and then run a Python ``listener``:
```shell
. ~/ros2_humble/install/local_setup.bash
ros2 run demo_nodes_py listener
```
You should see the ``talker`` saying that it’s ``Publishing`` messages and the ``listener`` saying ``I heard`` those messages. This verifies both the C++ and Python APIs are working properly. Hooray!













