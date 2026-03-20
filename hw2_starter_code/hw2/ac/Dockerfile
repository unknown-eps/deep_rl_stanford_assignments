FROM ubuntu:20.04

# Avoid interactive prompts during apt installations
ENV DEBIAN_FRONTEND=noninteractive

# Install system dependencies
RUN apt-get update && apt-get install -y \
    unzip \
    curl \
    wget \
    git \
    libglew-dev \
    patchelf \
    libegl1-mesa \
    libgl1-mesa-glx \
    libopengl0 \
    libosmesa6-dev \
    libglib2.0-0 \
    build-essential \
    sudo \
    && rm -rf /var/lib/apt/lists/*

# Create ubuntu user to match the script paths
RUN useradd -m -s /bin/bash ubuntu && \
    echo "ubuntu ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers
USER ubuntu
WORKDIR /home/ubuntu

# Install MuJoCo
RUN mkdir -p /home/ubuntu/.mujoco && \
    cd /home/ubuntu/.mujoco && \
    wget https://mujoco.org/download/mujoco210-linux-x86_64.tar.gz && \
    tar -xvf mujoco210-linux-x86_64.tar.gz && \
    rm mujoco210-linux-x86_64.tar.gz

# Set LD_LIBRARY_PATH
ENV LD_LIBRARY_PATH=/home/ubuntu/.mujoco/mujoco210/bin:/usr/lib/nvidia

# Install micromamba
RUN curl -L micro.mamba.pm/install.sh | bash
# Micromamba usually installs to ~/.local/bin by default in this script
ENV PATH="/home/ubuntu/.local/bin:$PATH"

# Copy ONLY the conda_env.yml so we can pre-build the environment
COPY --chown=ubuntu:ubuntu conda_env.yml /tmp/conda_env.yml

# Pre-build the AC environment
RUN /bin/bash -c "source ~/.bashrc && \
    micromamba config set channel_priority flexible && \
    micromamba env create --file=/tmp/conda_env.yml && \
    micromamba run -n AC pip install metaworld@git+https://github.com/Farama-Foundation/Metaworld.git@a98086ababc81560772e27e7f63fe5d120c4cc50 && \
    micromamba run -n AC pip install 'cython<3'"

# Make the micromamba environment activate automatically on shell start
RUN echo "micromamba activate AC" >> /home/ubuntu/.bashrc

# The workspace will be automatically mounted by VS Code to /workspace
WORKDIR /workspace

