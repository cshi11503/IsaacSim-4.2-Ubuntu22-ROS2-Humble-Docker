# ============================================================
# Isaac Sim 4.2.0 + ROS 2 Humble
#
# Intended for:
# NVIDIA "Getting Started with Isaac Sim" tutorials
#
# Host:
#   Hyperion / Ubuntu 24.04
#
# Container:
#   NVIDIA Isaac Sim 4.2.0
#   Ubuntu 22.04-era environment
#   ROS 2 Humble
#
# User:
#   UID:GID = 1234:1234
# ============================================================

FROM nvcr.io/nvidia/isaac-sim:4.2.0


# ------------------------------------------------------------
# Build arguments
# ------------------------------------------------------------

ARG USERNAME=casey
ARG USER_UID=1234
ARG USER_GID=1234


# ------------------------------------------------------------
# Environment
# ------------------------------------------------------------

ENV DEBIAN_FRONTEND=noninteractive

ENV ROS_DISTRO=humble
ENV RMW_IMPLEMENTATION=rmw_fastrtps_cpp

ENV ISAAC_SIM_PATH=/isaac-sim

ENV LANG=en_US.UTF-8
ENV LC_ALL=en_US.UTF-8


# ------------------------------------------------------------
# Basic system tools
# ------------------------------------------------------------

USER root
RUN chmod a+x \
    /isaac-sim/runheadless.native.sh \
    /isaac-sim/runheadless.sh \
    /isaac-sim/isaac-sim.sh \
    2>/dev/null || true

RUN apt-get update && apt-get install -y --no-install-recommends \
    locales \
    curl \
    wget \
    git \
    gnupg2 \
    lsb-release \
    software-properties-common \
    build-essential \
    ca-certificates \
    && locale-gen en_US.UTF-8 \
    && rm -rf /var/lib/apt/lists/*


# ------------------------------------------------------------
# Add ROS 2 apt repository
# ------------------------------------------------------------

RUN curl -sSL \
    https://raw.githubusercontent.com/ros/rosdistro/master/ros.key \
    -o /usr/share/keyrings/ros-archive-keyring.gpg \
    && echo \
    "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] http://packages.ros.org/ros2/ubuntu $(lsb_release -cs) main" \
    > /etc/apt/sources.list.d/ros2.list


# ------------------------------------------------------------
# ROS 2 Humble + tutorial dependencies
# ------------------------------------------------------------

RUN apt-get update && apt-get install -y --no-install-recommends \
    ros-humble-desktop \
    ros-humble-vision-msgs \
    ros-humble-ackermann-msgs \
    python3-rosdep \
    python3-rosinstall \
    python3-rosinstall-generator \
    python3-wstool \
    python3-colcon-common-extensions \
    && rm -rf /var/lib/apt/lists/*


# ------------------------------------------------------------
# rosdep
# ------------------------------------------------------------

RUN rosdep init 2>/dev/null || true


# ------------------------------------------------------------
# Create non-root user matching Hyperion UID/GID
# ------------------------------------------------------------

RUN if ! getent group ${USER_GID} >/dev/null; then \
        groupadd --gid ${USER_GID} ${USERNAME}; \
    fi \
    && EXISTING_GROUP=$(getent group ${USER_GID} | cut -d: -f1) \
    && if ! getent passwd ${USER_UID} >/dev/null; then \
        useradd \
            --uid ${USER_UID} \
            --gid ${EXISTING_GROUP} \
            --create-home \
            --shell /bin/bash \
            ${USERNAME}; \
    fi


# ------------------------------------------------------------
# Determine/create writable user directories
# ------------------------------------------------------------
RUN mkdir -p \
    /workspace \
    /home/${USERNAME} \
    /home/${USERNAME}/.cache/ov \
    /home/${USERNAME}/.cache/pip \
    /home/${USERNAME}/.cache/nvidia/GLCache \
    /home/${USERNAME}/.nv/ComputeCache \
    /home/${USERNAME}/.nvidia-omniverse/logs \
    /home/${USERNAME}/.local/share/ov/data \
    /home/${USERNAME}/Documents \
    && chown -R ${USER_UID}:${USER_GID} /home/${USERNAME} \
    && chown -R ${USER_UID}:${USER_GID} /workspace \
    && chmod 775 /workspace


# ------------------------------------------------------------
# Shell configuration
# ------------------------------------------------------------

RUN echo "source /opt/ros/humble/setup.bash" \
        >> /home/${USERNAME}/.bashrc \
    && echo "export ROS_DISTRO=humble" \
        >> /home/${USERNAME}/.bashrc \
    && echo "export RMW_IMPLEMENTATION=rmw_fastrtps_cpp" \
        >> /home/${USERNAME}/.bashrc \
    && echo "export ISAAC_SIM_PATH=/isaac-sim" \
        >> /home/${USERNAME}/.bashrc \
    && chown ${USER_UID}:${USER_GID} \
        /home/${USERNAME}/.bashrc


# ------------------------------------------------------------
# Runtime environment
# ------------------------------------------------------------

ENV HOME=/home/${USERNAME}


# ------------------------------------------------------------
# Switch to non-root user
# ------------------------------------------------------------

USER ${USER_UID}:${USER_GID}


# ------------------------------------------------------------
# Default working directory
# ------------------------------------------------------------

WORKDIR /workspace


# ------------------------------------------------------------
# Keep container available for interactive use
# ------------------------------------------------------------

CMD ["/bin/bash"]