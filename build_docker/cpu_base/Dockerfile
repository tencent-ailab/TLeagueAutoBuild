# ubuntu 18.04, TF1.15.0 and game core free (for any env/game).
#
# IMPORTANT:
# Then build the docker in the parent dir.
#
# For example:
# In /root/code/SC2RLAutoBuild run the build cmd, e.g.:
# docker build -f build_docker/cpu_base/Dockerfile -t tleague:vbase .
#
# IMPORTANT:
# optionally use the domestic ubuntu/pip sources for your consideration!
#
FROM ubuntu:18.04

# basic settings
ARG python=3.6
ENV PYTHON_VERSION=${python}
ENV TENSORFLOW_VERSION=1.15.0
ENV WORK_DIR /root/work
WORKDIR ${WORK_DIR}

# apt-get stuff (pre)
RUN apt-get update && apt-get install -y \
    apt-transport-https \
    apt-utils \
    ca-certificates

# apt-get stuff
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
    python${PYTHON_VERSION} \
    python${PYTHON_VERSION}-dev \
    python${PYTHON_VERSION}-distutils
RUN ln -s /usr/bin/python${PYTHON_VERSION} /usr/bin/python

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
  tensorflow==${TENSORFLOW_VERSION} \
  gym==0.12.1 \
  libtmux==0.8.2

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