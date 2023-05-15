## SETTINGS ------------------------------------------------------------------
ARG TIMEZONE=UTC
ARG UBUNTU_VERSION=20.04


## BUILD IMAGE ---------------------------------------------------------------
FROM ubuntu:${UBUNTU_VERSION}
ARG TIMEZONE

# Run system updates
RUN apt update && apt -y upgrade

# Set time zone
RUN ln -sn /usr/share/zoneinfo/${TIMEZONE} /etc/localtime \
    && echo ${TIMEZONE} > /etc/timezone \
    && apt -y install tzdata

# Set locale
RUN apt -y install locales \
    && locale-gen en_US.UTF-8 \
    && update-locale LANG=en_US.UTF-8 LC_MESSAGES=POSIX

# Install system packages
RUN DEBIAN_FRONTEND=noninteractive apt -y install \
        curl \
        dos2unix \
        git \
        graphviz \
        htop \
        libenchant-2-dev \
        lshw \
        nano \
        openssl \
        python3-pip \
        python3-tk \
        python3-venv \
        texlive-full \
        tree \
        vim \
        wget \
        zip

# Set working directory
WORKDIR /root

# Set default command when running container
CMD ["/bin/bash"]
