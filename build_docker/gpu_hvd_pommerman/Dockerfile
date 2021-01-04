# TLeague with TF GPU and pommerman game core
#
# IMPORTANT:
# Place TLeague, TPolicies, Arena, PySC2 Tencent Extension in the same dir
# Then build the docker in the parent dir.
#
# For example:
# /root/code/TLeagueAutoBuild:
#   TLeague/
#   TPolicies/
#   Arena/
#   PySC2/
#   TImitate/
#
# in the dir /root/code/TLeagueAutoBuild
# First build gpu_hvd_pommerman_base/Dockerfile
#
# Then build this , e.g.:
# docker build -f build_docker/gpu_hvd_pommerman/Dockerfile -t tleague-gpu-hvd-pommerman:latest .
#
# IMPORTANT:
# optionally use the domestic ubuntu/pip sources for your consideration!
#
FROM ccr.ccs.tencentyun.com/sc2ai/tleague-gpu-hvd:vbase

# basic settings
ENV TMUX_VERSION=2.1
ENV WORK_DIR /root/work
SHELL ["/bin/bash", "-cu"]
WORKDIR ${WORK_DIR}

# install playground
RUN git clone https://github.com/MultiAgentLearning/playground.git
RUN cd ${WORK_DIR}/playground && git checkout 4ba44f601157ab04aa0291bed7c2de4bde9e8f6b
RUN sed -i 's/reraise(/raise ImportError(str(error) +/g;s/suffix="/". /g;/from gym.utils import reraise/d;/except pyglet.canvas.xlib.NoSuchDisplayException as error:/d;/print("Import error NSDE! You will not be able to render --> %s" % error)/d' ${WORK_DIR}/playground/pommerman/graphics.py
RUN cd ${WORK_DIR}/playground && pip install -e .

# install TLeague
RUN mkdir -p ${WORK_DIR}/TLeague
COPY ./TLeague/tleague ./TLeague/tleague
COPY ./TLeague/setup.py ./TLeague/setup.py
RUN cd ${WORK_DIR}/TLeague && pip install -e .

# install TPolicies
RUN mkdir -p ${WORK_DIR}/TPolicies
COPY ./TPolicies/tpolicies ./TPolicies/tpolicies
COPY ./TPolicies/setup.py ./TPolicies/setup.py
RUN cd ${WORK_DIR}/TPolicies && pip install -e .

# install Arena
RUN mkdir -p ${WORK_DIR}/Arena
COPY ./Arena/arena ./Arena/arena
COPY ./Arena/setup.py ./Arena/setup.py
RUN cd ${WORK_DIR}/Arena && pip install -e .