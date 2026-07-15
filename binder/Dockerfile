# Dockerfile of the base image: https://github.com/IntEL4CoRo/jupyter-ros2/blob/main/Dockerfile
FROM intel4coro/jupyter-ros2:jazzy-py3.12

USER root
ENV PATH=$HOME/.local/bin:$PATH

EXPOSE 1883

ENV REPO_DIR=/app

# Copy content to the image
COPY . ${REPO_DIR}/
RUN git config --global --add safe.directory ${REPO_DIR}

# Set JupyterLab default directory
WORKDIR ${REPO_DIR}

# Set VSCODE default directory
ENV CODE_WORKING_DIRECTORY=${REPO_DIR}

#============= Install extra software packages =============#
# Set ROS distribution to use
ENV ROS_DISTRO='jazzy'
RUN echo 'export ROS_DISTRO=$ROS_DISTRO' >> ~/.bashrc && echo 'source /opt/ros/$ROS_DISTRO/setup.bash' >> ~/.bashrc

# Install Requirements
RUN apt-get update && apt-get install apt-utils -y
RUN apt-get install -y \
    nano \
    python3.12 \
    python3.12-venv \
    python3-pip \
    python3-sklearn \
    python3-matplotlib \
    python3-numpy \
    openjdk-21-jdk \
    ros-"$ROS_DISTRO"-desktop \
    ros-dev-tools \
    ros-"$ROS_DISTRO"-cartographer \
    ros-"$ROS_DISTRO"-cartographer-ros \
    ros-"$ROS_DISTRO"-mqtt-client \
    ros-"$ROS_DISTRO"-nav2-bringup \
    ros-"$ROS_DISTRO"-nav2-lifecycle-manager \
    ros-"$ROS_DISTRO"-navigation2 \
    ros-"$ROS_DISTRO"-ros-gz \
    ros-"$ROS_DISTRO"-slam-toolbox \
    ros-"$ROS_DISTRO"-tf-transformations \
    ros-"$ROS_DISTRO"-tf2 \
    ros-"$ROS_DISTRO"-turtlebot3 \
    ros-"$ROS_DISTRO"-turtlebot3-msgs \
    ros-"$ROS_DISTRO"-turtlebot3-simulations \
    ros-"$ROS_DISTRO"-crazyflie \
    ros-"$ROS_DISTRO"-crazyflie-interfaces \
    ros-"$ROS_DISTRO"-xacro \
    python3-colcon-common-extensions \
    mosquitto \
    freeglut3-dev \
    libgsl-dev

# Change the default shell to Bash
SHELL [ "/bin/bash" , "-c" ]

# Enable mosquitto
RUN systemctl enable mosquitto.service

# Install sklearn (binder container doesn't seem to work with apt version)
RUN pip install scikit-learn

# Create Python venv
RUN python3 -m venv --system-site-packages "$REPO_DIR"/rumros/webapp/venv

# Clone repository
RUN git clone --recursive https://github.com/sebastiangoetz/RuMROS.git "$REPO_DIR"/rumros \
    && cd "$REPO_DIR"/rumros \
    && git pull

# Install ARGoS
RUN cd "$REPO_DIR"/rumros/dependencies \
    && apt-get install -y ./argos3_simulator-3.0.0-x86_64-beta59.deb

# Link libglut for ARGoS (ARGoS compiled for Ubuntu 22.04, where this path is different)
RUN cd /usr/lib/x86_64-linux-gnu \
    && ln -s libglut.so.3.12.0 libglut.so.3

# Install Turtlebot plugin for ARGoS
RUN cd "$REPO_DIR" \
    && git clone https://github.com/ilpincy/argos3-turtlebot3.git \
    && cd argos3-turtlebot3 \
    && mkdir build \
    && cd build \
    && cmake -DCMAKE_BUILD_TYPE=Release ../src -DCMAKE_INSTALL_PREFIX=/usr \
    && make \
    && make install \
    && ln -s /usr/lib/argos3 /usr/local/lib/argos3

# Install Crazyflie plugin for ARGoS
RUN cd "$REPO_DIR" \
    && git clone https://gitlab.com/uniluxembourg/snt/pcog/adars/crazyflie.git \
    && cd crazyflie \
    && mkdir build \
    && cd build \
    && cmake -DCMAKE_BUILD_TYPE=Release ../src -DCMAKE_INSTALL_PREFIX=/usr \
    && make \
    && make install

# Build ROS workspace
RUN cd "$REPO_DIR"/rumros/ros/rumros_swarm_ws \
    && source /opt/ros/"$ROS_DISTRO"/setup.bash \
    && colcon build

# Setup webapp
RUN source "$REPO_DIR"/rumros/webapp/venv/bin/activate \
    && pip install -r "$REPO_DIR"/rumros/webapp/requirements.txt

# The entrypoint of the docker image
COPY binder/entrypoint.sh /entrypoint.sh
ENTRYPOINT ["/entrypoint.sh"]

