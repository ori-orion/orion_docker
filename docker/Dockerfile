FROM ros:noetic-perception

ENV ROS_WS_DIR="/orion_ws"

WORKDIR ${ROS_WS_DIR}

SHELL ["/bin/bash", "-c"]

ENV DEBIAN_FRONTEND=noninteractive

# Useful non-ROS tools
RUN apt-get update \
 && apt-get install -y \
    git \
    net-tools \
    wget \
 && rm -rf /var/lib/apt/lists/*

# Useful ROS tools
RUN apt-get update \
 && apt-get install -y \
    python3-catkin-tools \
    python3-osrf-pycommon \
    python3-vcstool \
 && rm -rf /var/lib/apt/lists/*

# Taken from HSR wiki page
RUN sh -c 'echo "deb [arch=amd64] https://hsr-user:jD3k4G2e@packages.hsr.io/ros/ubuntu `lsb_release -cs` main" > /etc/apt/sources.list.d/tmc.list' \
 && sh -c 'echo "deb [arch=amd64] https://hsr-user:jD3k4G2e@packages.hsr.io/tmc/ubuntu `lsb_release -cs` multiverse main" >> /etc/apt/sources.list.d/tmc.list' \
 && sh -c 'echo "deb http://packages.osrfoundation.org/gazebo/ubuntu-stable `lsb_release -cs` main" > /etc/apt/sources.list.d/gazebo-stable.list' \
 && wget https://hsr-user:jD3k4G2e@packages.hsr.io/tmc.key -O - | apt-key add - \
 && wget https://raw.githubusercontent.com/ros/rosdistro/master/ros.asc -O - | apt-key add - \
 && wget https://packages.osrfoundation.org/gazebo.key -O - | apt-key add - \
 && sh -c 'mkdir -p /etc/apt/auth.conf.d' \
 && sh -c '/bin/echo -e "machine packages.hsr.io\nlogin hsr-user\npassword jD3k4G2e" >/etc/apt/auth.conf.d/auth.conf' \
 && apt-get update \
 && apt-get install -y \
    ros-${ROS_DISTRO}-tmc-desktop-full \
 && rm -rf /var/lib/apt/lists/*
RUN apt-get update \
 && apt-get install -y \
    ros-${ROS_DISTRO}-hsrb-description \
    ros-${ROS_DISTRO}-tmc-control-msgs \
 && rm -rf /var/lib/apt/lists/*

# install nano for editing within docker if needed
RUN apt-get update 
RUN apt-get install nano -y

# add in the lines in basrc to set network interface, set alias (not sure about this line, figure out with tobit)
# RUN echo "export ROS_HOME=~/.ros" >> ~/.bashrc
#  && echo "alias sim_mode='export ROS_MASTER_URI=http://localhost:11311 export PS1="\[\033[44;1;37m\]<local>\[\033[0m\]\w$ "'" >> ~/.bashrc
#  && echo "alias hsrb_mode='export ROS_MASTER_URI=http://hsrb:11311 export PS1="\[\033[41;1;37m\]<hsrb>\[\033[0m\]\w$ "'" >> ~/.bashrc

# install orion packages: tmux and openni2-camera package
RUN apt-get update \
 && apt-get install -y \
    python3-pip \
    tmux \
    openni2-utils \
    portaudio19-dev \
    python3-pyaudio \
    python3-sphinx \
 && rm -rf /var/lib/apt/lists/*

RUN pip install numpy --upgrade
RUN pip install numba --upgrade
RUN pip install requests --upgrade

# Speech Recognition dependencies
RUN pip install spacy vosk sounddevice nltk SpeechRecognition python-Levenshtein noisereduce
RUN pip install pvporcupine 
RUN pip install pvrecorder

# Semantic mapping dependencies
RUN apt-get update
RUN apt-get install mongodb-server-core -y
RUN pip install pymongo

# Manipulation dependencies
RUN pip install imutils
RUN apt install ros-noetic-octomap
RUN apt install ros-noetic-octomap-ros
RUN apt install ros-noetic-rviz-visual-tools -y

# Navigation dependencies
RUN pip install transform3d

# Perception dependencies
RUN pip install einops
RUN pip install seaborn

# install pytorch (torch = 1.13.1, torchvision = 0.14.1, torchaudio = 0.13.1, cuda = 11.6)
RUN pip install torch==1.13.1+cu116 torchvision==0.14.1+cu116 torchaudio==0.13.1 --extra-index-url https://download.pytorch.org/whl/cu116

# install caffe (needed for gpd)
RUN sudo cp /etc/apt/sources.list /etc/apt/sources.list~
RUN sudo sed -Ei 's/^# deb-src /deb-src /' /etc/apt/sources.list
RUN sudo apt-get update
RUN apt build-dep caffe-cpu -y
RUN apt install libatlas-base-dev -y

ENV DEBIAN_FRONTEND=dialog

COPY "/.vscode" "${ROS_WS_DIR}/.vscode"
RUN sed -i -E "s/(ROS_DISTRO=\\$\{1:-\").*(\"\})/\1${ROS_DISTRO}\2/" ${ROS_WS_DIR}/.vscode/ros_source.bash \
 && echo "alias rsource='source ${ROS_WS_DIR}/.vscode/ros_source.bash'" >> ~/.bash_aliases \
 && echo "alias rbuild='(cd ${ROS_WS_DIR} && catkin build)'" >> ~/.bash_aliases \
 && echo "alias rclean='(cd ${ROS_WS_DIR} && catkin clean -y)'" >> ~/.bash_aliases \
 && echo "source ${ROS_WS_DIR}/.vscode/ros_source.bash" >> ~/.bashrc

