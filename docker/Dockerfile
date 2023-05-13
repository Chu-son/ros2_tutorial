FROM ubuntu:22.04

# Timezone, Launguage設定
RUN apt update \
  && apt install -y --no-install-recommends \
     locales \
     software-properties-common tzdata \
  && locale-gen en_US en_US.UTF-8 \
  && update-locale LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8 \
  && add-apt-repository universe

ENV LANG en_US.UTF-8
ENV TZ=Asia/Tokyo
ENV DEBIAN_FRONTEND=noninteractive
ARG ROS_VERSION=humble

# Install ROS2
RUN apt update \
  && apt install -y --no-install-recommends \
     curl gnupg2 lsb-release python3-pip vim wget build-essential ca-certificates git

RUN curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key -o /usr/share/keyrings/ros-archive-keyring.gpg \
  && echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] http://packages.ros.org/ros2/ubuntu $(lsb_release -cs) main" | tee /etc/apt/sources.list.d/ros2.list > /dev/null

RUN apt-get update \
  && apt-get upgrade -y \
  && apt-get install -y --no-install-recommends \
     ros-$ROS_VERSION-desktop \
  && rm -rf /var/lib/apt/lists/*

RUN bash /opt/ros/$ROS_VERSION/setup.sh 

### NVIDIA Container Toolkit
RUN distribution=$(. /etc/os-release;echo $ID$VERSION_ID) \
  && curl -s -L https://nvidia.github.io/libnvidia-container/gpgkey | apt-key add - \
  && curl -s -L https://nvidia.github.io/libnvidia-container/$distribution/libnvidia-container.list | tee /etc/apt/sources.list.d/nvidia-container-toolkit.list

RUN apt-get update \
  && apt-get install -y --no-install-recommends nvidia-container-toolkit

### Install Packages
RUN apt update \
  && apt install -y --no-install-recommends \
      python3-argcomplete python3-colcon-common-extensions python3-rosdep ament-cmake

### Install custom packages
RUN apt update \
  && apt install -y --no-install-recommends \
      ros-${ROS_VERSION}-gazebo-* ros-${ROS_VERSION}-turtlebot3 ros-${ROS_VERSION}-turtlebot3-msgs

WORKDIR /root/ros2_ws/src
RUN git clone -b ${ROS_VERSION}-devel https://github.com/ROBOTIS-GIT/turtlebot3_simulations.git \
  && git clone -b ${ROS_VERSION} https://github.com/ros2/common_interfaces.git

WORKDIR /root/ros2_ws
RUN rosdep init \
  && rosdep update \
  && rosdep install -y --from-paths src --ignore-src --rosdistro $ROS_VERSION

RUN . /opt/ros/${ROS_VERSION}/setup.sh && colcon build --symlink-install

RUN echo "source /opt/ros/$ROS_VERSION/setup.bash" >> ~/.bashrc
RUN echo "source /root/ros2_ws/install/setup.bash" >> ~/.bashrc

RUN echo 'export GAZEBO_MODEL_PATH=$GAZEBO_MODEL_PATH:~/ros2_ws/src/turtlebot3/turtlebot3_simulations/turtlebot3_gazebo/models' >> ~/.bashrc
RUN echo 'export TURTLEBOT3_MODEL=burger' >> ~/.bashrc