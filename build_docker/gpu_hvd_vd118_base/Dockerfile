# TF-1.15.0, Horovod 0.16.14 and ViZDoom 875ba34 (Apr 25, 2020, which should be 1.1.8 compatible)
# for TLeague with TF GPU, borrowed and modified from Horovod
FROM nvidia/cuda:10.0-devel-ubuntu18.04

# basic settings
ENV HOROVOD_VERSION=0.18.2
ENV CUDNN_VERSION=7.6.5.32-1+cuda10.0
ENV NCCL_VERSION=2.4.7-1+cuda10.0
ARG python=3.6
ENV PYTHON_VERSION=${python}
# ENV VIZDOOM_VERSION=1.1.7
ENV WORK_DIR /root/work

# Set default shell to /bin/bash
SHELL ["/bin/bash", "-cu"]
WORKDIR ${WORK_DIR}


# before using domestic sources
#RUN apt-get update && apt-get install -y \
#    apt-transport-https \
#    apt-utils
# use domestic sources for nvidia stuff
RUN mv /etc/apt/sources.list.d/cuda.list /etc/apt/sources.list.d/cuda.list.bak && \
    mv /etc/apt/sources.list.d/nvidia-ml.list /etc/apt/sources.list.d/nvidia-ml.list.bak
COPY ./build_docker/cuda.list.tencent /etc/apt/sources.list.d/cuda.list
COPY ./build_docker/nvidia-ml.list.tencent /etc/apt/sources.list.d/nvidia-ml.list
# use domestic sources for ubuntu
RUN mv /etc/apt/sources.list /etc/apt/sources.list.bak
COPY ./build_docker/sources.ubuntu1804.list.tencent /etc/apt/sources.list
#COPY ./build_docker/sources.ubuntu1804.list.aliyun /etc/apt/sources.list

# apt-get stuff
RUN apt-get update && apt-get install -y --allow-downgrades --allow-change-held-packages --no-install-recommends \
        build-essential \
        cmake \
        make \
        libevent-dev \
        libncurses-dev \
        g++-4.8 \
        git \
        curl \
        vim \
        wget \
        zip \
        unzip \
        ca-certificates \
        libcudnn7=${CUDNN_VERSION} \
        libnccl2=${NCCL_VERSION} \
        libnccl-dev=${NCCL_VERSION} \
        libjpeg-dev \
        libpng-dev \
        google-perftools \
        bzip2 \
        libboost-all-dev \
        libbz2-dev \
        libfluidsynth-dev \
        libfreetype6-dev \
        libgme-dev \
        libgtk2.0-dev \
        libopenal-dev \
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
        librdmacm1 \
        libibverbs1 \
        ibverbs-providers
RUN if [[ "${PYTHON_VERSION}" == "3.6" ]]; then \
        apt-get install -y python${PYTHON_VERSION}-distutils; \
    fi
RUN mv /usr/bin/python /usr/bin/python.bak && \
    ln -s /usr/bin/python${PYTHON_VERSION} /usr/bin/python

# configure google-perftools
RUN echo "export LD_PRELOAD=/usr/lib/x86_64-linux-gnu/libtcmalloc.so.4" >> /root/.bashrc

# pip from web
#RUN curl -O https://bootstrap.pypa.io/get-pip.py && \
#    python get-pip.py && \
#    rm get-pip.py
# pip from local cached
COPY build_docker/get-pip.py ./get-pip.py
RUN python get-pip.py && rm get-pip.py

# pip has been ready, use domestic sources for pip
RUN mkdir -p /root/.pip
COPY ./build_docker/pip.conf.tencent /root/.pip/pip.conf

# python deps, with required version
RUN pip install future typing
RUN pip install numpy \
        gym==0.12.1 \
        keras \
        h5py \
        libtmux==0.8.2
# install tensorflow from binary, with memory-leak bug fixed
#COPY ./build_docker/tensorflow-1.15.0-fix-cp36-cp36m-linux_x86_64.whl ./tensorflow-1.15.0-cp36-cp36m-linux_x86_64.whl
#RUN pip install ./tensorflow-1.15.0-cp36-cp36m-linux_x86_64.whl && rm ./tensorflow-1.15.0-cp36-cp36m-linux_x86_64.whl
# or you simply install the public tf
RUN pip install tensorflow==1.15.0

# install vizdoom
# RUN pip install vizdoom==${VIZDOOM_VERSION}
RUN pip install git+https://github.com/mwydmuch/ViZDoom@875ba34554e9d7cfe2053a041faa68846819a86c
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


# --- default OS settings ---
RUN echo "ulimit -s 1000000" >> /root/.bashrc
RUN echo "ulimit -n 60000" >> /root/.bashrc

# --- for horovod stuff, borrowd from official horovod repo ---
# Install Open MPI
RUN mkdir /tmp/openmpi && \
    cd /tmp/openmpi && \
    wget https://www.open-mpi.org/software/ompi/v4.0/downloads/openmpi-4.0.0.tar.gz && \
    tar zxf openmpi-4.0.0.tar.gz && \
    cd openmpi-4.0.0 && \
    ./configure --enable-orterun-prefix-by-default && \
    make -j $(nproc) all && \
    make install && \
    ldconfig && \
    rm -rf /tmp/openmpi
# Install Horovod, temporarily using CUDA stubs.
# NOTE the HOROVOD_WITHOUT_XXX flags, which are pitfalls
RUN ldconfig /usr/local/cuda/targets/x86_64-linux/lib/stubs && \
    HOROVOD_GPU_ALLREDUCE=NCCL HOROVOD_WITH_TENSORFLOW=1 HOROVOD_WITHOUT_PYTORCH=1 HOROVOD_WITHOUT_MXNET=1 \
         pip install --no-cache-dir horovod==${HOROVOD_VERSION} && \
    ldconfig
# Install OpenSSH for MPI to communicate between containers
RUN apt-get install -y --no-install-recommends openssh-client openssh-server && \
    mkdir -p /var/run/sshd
# Allow OpenSSH to talk to containers without asking for confirmation
RUN cat /etc/ssh/ssh_config | grep -v StrictHostKeyChecking > /etc/ssh/ssh_config.new && \
    echo "    StrictHostKeyChecking no" >> /etc/ssh/ssh_config.new && \
    mv /etc/ssh/ssh_config.new /etc/ssh/ssh_config

# --- our own recipes ---
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
