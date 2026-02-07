# We use a standard Ubuntu image as our base
FROM ubuntu:22.04

# Avoid prompts during package installation
ENV DEBIAN_FRONTEND=noninteractive

# 1. Install everything available via Ubuntu's official manager
RUN apt-get update && apt-get install -y \
    build-essential cmake git wget \
    python3 python3-pip python3-dev \
    # These install instantly as pre-compiled binaries:
    python3-numpy \
    python3-matplotlib \
    python3-scipy \
    python3-pyqt5 \
    python3-pil \
    python3-openpyxl \
    # System deps for preCICE/Chrono
    libboost-all-dev libgl1-mesa-dev libeigen3-dev \
    libpetsc-real-dev libopenmpi-dev libxml2-dev \
    && rm -rf /var/lib/apt/lists/*

# 2. Install only the niche stuff via Pip
COPY requirements.txt /tmp/requirements.txt
RUN pip3 install --no-cache-dir -r /tmp/requirements.txt

COPY requirements.txt /tmp/requirements.txt
# Using --no-cache-dir saves space in your Docker image
RUN pip3 install --no-cache-dir -r /tmp/requirements.txt

# 3. Install preCICE (Coupling Library) (v3.3.1 for Ubuntu 22.04 Jammy)
RUN apt-get update && apt-get install -y libxml2-dev \
    && ARCH=$(dpkg --print-architecture) \
    && if [ "$ARCH" = "amd64" ]; then \
        echo "AMD64 detected: Downloading pre-built .deb..." \
        && wget -q https://github.com/precice/precice/releases/download/v3.3.1/libprecice3_3.3.1_jammy.deb -O precice.deb \
        && apt-get install -y ./precice.deb \
        && rm precice.deb; \
    else \
        echo "ARM64 detected (e.g., Apple Silicon): Building preCICE v3.3.1 from source..." \
        && git clone --branch v3.3.1 --depth 1 https://github.com/precice/precice.git /opt/precice_source \
        && mkdir /opt/precice_source/build && cd /opt/precice_source/build \
        && cmake -DCMAKE_BUILD_TYPE=Release \
                 -DPRECICE_PythonActions=OFF \
                 -DCMAKE_INSTALL_PREFIX=/usr/local \
                 .. \
        && make -j$(nproc) install \
        && rm -rf /opt/precice_source; \
    fi

# 4. Install Project Chrono (Build from Source)
# This takes time! We clone it, make a build directory, and install it to /usr/local
# installing the 9.0.1 version
RUN git clone -b 9.0.1 --depth 1 https://github.com/projectchrono/chrono.git /opt/chrono_source \
    && mkdir /opt/chrono_source/build \
    && cd /opt/chrono_source/build \
    && cmake -DCMAKE_BUILD_TYPE=Release \
             -DENABLE_MODULE_IRRLICHT=OFF \
             -DENABLE_MODULE_POSTPROCESS=ON \
             .. \
    && make -j$(nproc) install \
    && rm -rf /opt/chrono_source

ENV CHRONO_DATA_DIR=/usr/local/share/chrono/data/


# did not try this yet, so don't know if mac will work

# 5. (OPTIONAL) OpenFOAM Section
# I have commented this out. To enable it later:
# 1. Remove the '#' from the lines below.
# 2. Run "Rebuild Container" in VS Code.
# 
# RUN wget -q -O - https://dl.openfoam.com/add-debian-repo.sh | bash \
#     && apt-get install -y openfoam2312-default \
#     && echo "source /usr/lib/openfoam/openfoam2312/etc/bashrc" >> ~/.bashrc

# Set the default user to root (simplifies permission issues on Windows/WSL)
USER root