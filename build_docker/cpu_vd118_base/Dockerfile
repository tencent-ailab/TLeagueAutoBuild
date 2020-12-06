# ubuntu 18.04, TF1.15.0 and ViZDoom 875ba34 (Apr 25, 2020, which should be 1.1.8 compatible)
FROM ubuntu:18.04

# basic settings
ARG python=3.6
ENV PYTHON_VERSION=${python}
# ENV VIZDOOM_VERSION=1.1.7
ENV TENSORFLOW_VERSION=1.15.0
ENV WORK_DIR /root/work

#SHELL ["/bin/bash", "-cu"]
WORKDIR ${WORK_DIR}

#
RUN apt-get update && apt-get install -y \
    apt-transport-https \
    apt-utils \
    ca-certificates

# use domestic sources for ubuntu
RUN mv /etc/apt/sources.list /etc/apt/sources.list.bak
COPY ./build_docker/sources.ubuntu1804.list.tencent /etc/apt/sources.list
#COPY ./build_docker/sources.ubuntu1804.list.aliyun /etc/apt/sources.list

# apt-get stuff
# borrowed from https://github.com/mwydmuch/ViZDoom/blob/master/docker/python3/Dockerfile
RUN apt-get update && apt-get install -y --allow-downgrades --allow-change-held-packages --no-install-recommends \
    cmake \
    make \
    libevent-dev \
    libncurses-dev \
    g++ \
    vim \
    unzip \
    git \
    curl \
    wget \
    iftop \
    ethtool \
    lsof \
    google-perftools \
    build-essential \
    bzip2 \
    libboost-all-dev \
    libbz2-dev \
    libfluidsynth-dev \
    libfreetype6-dev \
    libgme-dev \
    libgtk2.0-dev \
    libjpeg-dev \
    libopenal-dev \
    libpng-dev \
    libsdl2-dev \
    libwildmidi-dev \
    libzmq3-dev \
    nano \
    nasm \
    pkg-config \
    rsync \
    software-properties-common \
    sudo \
    tar \
    timidity \
    zlib1g-dev \
    python${PYTHON_VERSION} \
    python${PYTHON_VERSION}-dev \
    python${PYTHON_VERSION}-distutils
RUN mv /usr/bin/python /usr/bin/python.bak && \
    ln -s /usr/bin/python${PYTHON_VERSION} /usr/bin/python

# configure google-perftools
RUN echo "export LD_PRELOAD=/usr/lib/x86_64-linux-gnu/libtcmalloc.so.4" >> /root/.bashrc

# pip stuff
#RUN curl -O https://bootstrap.pypa.io/get-pip.py && \
#    python get-pip.py && \
#    rm get-pip.py
COPY ./build_docker/get-pip.py ./get-pip.py
RUN python get-pip.py && rm get-pip.py

# use domestic sources for pip
RUN mkdir -p /root/.pip
COPY ./build_docker/pip.conf.tencent /root/.pip/pip.conf

# python deps, with required version
RUN pip install virtualenv
RUN pip install numpy \
  gym==0.12.1 \
  keras \
  h5py \
  tensorflow==${TENSORFLOW_VERSION} \
  libtmux==0.8.2

# install vizdoom
# RUN pip install vizdoom==${VIZDOOM_VERSION}
RUN pip install git+https://github.com/mwydmuch/ViZDoom@875ba34554e9d7cfe2053a041faa68846819a86c
# other python deps used
RUN pip install portpicker \
    opencv-python

# --- binary deps ---
#RUN wget https://www.remi-coulom.fr/Bayesian-Elo/bayeselo.tar.bz2
#RUN tar -xvf bayeselo.tar.bz2 && cd BayesElo && make
COPY ./build_docker/BayesElo.zip .
RUN unzip BayesElo.zip && cd BayesElo && make && cp bayeselo /usr/local/bin
# use specific tmux version 2.1
# https://gist.github.com/japrescott/aa15cb024fe38ea36849f5f62c3314a3
COPY build_docker/tmux-2.1.tar.gz /tmp/
RUN cd /tmp && \
    tar xvzf tmux-2.1.tar.gz && \
    cd tmux-2.1/ && \
    ./configure && make && \
    make install && \
    cd .. && \
    rm -rf ./tmux-2.1*


### install ssh to ease the use of other container compilation
### system (e.g., tencent c.oa)
# Install OpenSSH
RUN apt-get install -y --no-install-recommends openssh-client openssh-server && \
    mkdir -p /var/run/sshd
# Allow OpenSSH to talk to containers without asking for confirmation
RUN cat /etc/ssh/ssh_config | grep -v StrictHostKeyChecking > /etc/ssh/ssh_config.new && \
    echo "    StrictHostKeyChecking no" >> /etc/ssh/ssh_config.new && \
    mv /etc/ssh/ssh_config.new /etc/ssh/ssh_config

# Avoid connection closed immediately after succesfully connected
RUN cat /etc/ssh/sshd_config | grep -v UsePAM > /etc/ssh/sshd_config.new && \
    echo "UsePAM no" >> /etc/ssh/sshd_config.new && \
    mv /etc/ssh/sshd_config.new /etc/ssh/sshd_config
# Allow passwordless ssh
RUN mkdir -p /root/.ssh && \
    cd /root/.ssh && \
    yes | ssh-keygen -N "" -f id_rsa && \
    chmod 600 id_rsa
RUN cd /root/.ssh && \
    cat id_rsa.pub >> authorized_keys && \
    echo "    StrictHostKeyChecking no" >> config
# large ssh alive time
RUN cp /etc/ssh/sshd_config /etc/ssh/sshd_config.new && \
    echo "ClientAliveInterval 60" >> /etc/ssh/sshd_config.new && \
    mv /etc/ssh/sshd_config.new /etc/ssh/sshd_config

# other patches.
RUN echo "Host *" > /root/.ssh/config && \
    echo "    ServerAliveInterval 60" >> /root/.ssh/config && \
    echo "    TCPKeepAlive yes" >> /root/.ssh/config

RUN echo "ulimit -s 1000000" >> /root/.bashrc
RUN echo "ulimit -n 60000" >> /root/.bashrc
