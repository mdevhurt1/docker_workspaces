# ----- Base: Nvidia -----
 FROM nvidia/cuda:13.0.0-cudnn-devel-ubuntu22.04 AS base
 
 # ----- Set environment variables -----
 ENV DEBIAN_FRONTEND=noninteractive
 ENV ROS_DISTRO=humble
 
 # ----- Temporarily disable NVIDIA/CUDA repo -----
 RUN set -eux; \
  echo "Disabling NVIDIA/CUDA apt sources..."; \
  find /etc/apt/sources.list.d/ -maxdepth 1 -type f \( -name 'cuda*.list' -o -name 'nvidia*.list' \) \
  -exec sed -i 's/^deb /# deb /' {} +; \
  apt-get update && echo "NVIDIA/CUDA sources disabled and apt updated."
 
 # ----- Install basic dependencies -----
 RUN set -eux; \
  echo "Installing basic dependencies..."; \
  apt-get update; \
  apt-get install -y --no-install-recommends \
    nano \
    git \
    cmake \
    clangd \
    curl \
    gnupg2 \
    clang-tidy \
    clang-format \
    cppcheck \
    lcov \
    gcovr \
    python3-pip \
    pre-commit \
    doxygen \
    libgl1-mesa-glx \
    libxkbcommon-x11-0 \
    x11-apps \
    lsb-release; \
  python3 -m pip install -U pre-commit identify; \
  git config --global credential.helper store; \
  rm -rf /var/lib/apt/lists/*; \
  echo "Basic dependencies installed."
 
 # ----- Install ROS2 -----
 RUN set -eux; \
  echo "Setting up locales..."; \
  apt update && apt install -y locales; \
  locale-gen en_US en_US.UTF-8 && \
  update-locale LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8; \
  export LANG=en_US.UTF-8; \
  echo "Locales set."; \
  echo "Installing software-properties-common and curl..."; \
  apt install -y software-properties-common; \
  add-apt-repository universe; \
  apt update && apt install -y curl; \
  echo "software-properties-common and curl installed."; \
  echo "Adding ROS 2 apt source..."; \
  export ROS_APT_SOURCE_VERSION=$(curl -s https://api.github.com/repos/ros-infrastructure/ros-apt-source/releases/latest | grep -F "tag_name" | awk -F\" '{print $4}') && \
  curl -L -o /tmp/ros2-apt-source.deb "https://github.com/ros-infrastructure/ros-apt-source/releases/download/${ROS_APT_SOURCE_VERSION}/ros2-apt-source_${ROS_APT_SOURCE_VERSION}.$(. /etc/os-release && echo $VERSION_CODENAME)_all.deb" && \
  dpkg -i /tmp/ros2-apt-source.deb; \
  apt update; \
  echo "ROS 2 apt source added."; \
  echo "Installing ros-dev-tools..."; \
  apt install -y ros-dev-tools && \
  echo "ros-dev-tools installed."; \
  echo "Upgrading system..."; \
  apt update && apt upgrade -y; \
  echo "System upgraded."; \
  echo "Installing ROS 2 desktop..."; \
  apt install -y ros-${ROS_DISTRO}-desktop; \
  echo "ROS 2 desktop installed."; \
  echo "Adding ROS 2 setup to bashrc..."; \
  echo "source /opt/ros/${ROS_DISTRO}/setup.bash" >> /etc/bash.bashrc; \
  echo "source ./install/setup.bash" >> /etc/bash.bashrc; \
  echo "source /usr/share/colcon_cd/function/colcon_cd.sh" >> ~/.bashrc; \
  echo "export _colcon_cd_root=/opt/ros/humble/" >> ~/.bashrc; \
  rm -rf /var/lib/apt/lists/*; \
  echo "ROS 2 setup complete."
 
 # ----- Install ROS2 Dependencies -----
 RUN set -eux; \
  echo "Installing ROS2 dependencies..."; \
  apt-get update && \
  apt-get install -y --no-install-recommends \
    python3-colcon-common-extensions; \
  rm -rf /var/lib/apt/lists/*; \
  echo "ROS2 dependencies installed."
 
 ARG USERNAME=vscode
 ARG USER_UID=1000
 ARG USER_GID=1000
 
 # Create user + ensure groups exist (idempotent), then grant passwordless sudo
 RUN set -eux; \
 # base primary group for the user
 groupadd -g "${USER_GID}" "${USERNAME}" || true; \
 # make sure these supplemental groups exist on this image
 for g in sudo video render dialout; do getent group "$g" >/dev/null || groupadd "$g"; done; \
 # create the user
 useradd -m -s /bin/bash -u "${USER_UID}" -g "${USER_GID}" "${USERNAME}" || true; \
 # add to supplemental groups
 usermod -aG sudo,video,render,dialout "${USERNAME}"; \
 # sudoers entry
 echo "${USERNAME} ALL=(ALL) NOPASSWD:ALL" > /etc/sudoers.d/99-${USERNAME}; \
 chmod 0440 /etc/sudoers.d/99-${USERNAME}
 
 USER ${USERNAME}