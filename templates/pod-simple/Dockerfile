FROM nvidia/cuda:9.0-cudnn7-devel-ubuntu16.04
LABEL maintainer "LastName FirstName <firstname.lastname@epfl.ch>"

# Install some necessary tools.
RUN apt-get update && apt-get install -y \
    cmake \
    curl \
    htop \
    locales \
    python3 \
    python3-pip \
    sudo \
    unzip \
    vim \
    wget \
    zsh \
    libssl-dev \
    libffi-dev \
&& rm -rf /var/lib/apt/lists/*

# Configure environments.
RUN echo "en_US.UTF-8 UTF-8" > /etc/locale.gen && locale-gen

# Python packages
RUN pip3 install --upgrade \
    scipy \
    numpy \
    torch \
    torchvision

# Configure user and group
ENV SHELL=/bin/bash \
    NB_USER=<your-name> \
    NB_UID=<your-uid> \
    NB_GROUP=MLO-unit \
    NB_GID=11169

ENV HOME=/home/$NB_USER


RUN groupadd $NB_GROUP -g $NB_GID
RUN useradd -m -s /bin/bash -N -u $NB_UID -g $NB_GID $NB_USER && \
    echo "${NB_USER}:${NB_USER}" | chpasswd && \
    usermod -aG sudo,adm,root ${NB_USER}
RUN chown -R ${NB_USER}:${NB_GROUP} ${HOME}

# The user gets passwordless sudo
RUN echo "${NB_USER}   ALL = NOPASSWD: ALL" > /etc/sudoers
