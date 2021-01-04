# TLeague with TF CPU and pommerman game core
#
# IMPORTANT:
# First, build cpu_pommerman_base/Dockerfile
# then, place TLeague, TPolicies, Arena, PySC2 Tencent Extension in the same dir
# Then build the docker in the parent dir.
#
# For example:
# /root/code/TLeagueAutoBuild:
#   TLeague/
#   TPolicies/
#   Arena/
#   PySC2/
# Then in /root/code/TLeagueAutoBuild run the build cmd, e.g.:
# docker build -f build_docker/cpu_pommerman/Dockerfile -t tleague-pommerman:latest .
#
# IMPORTANT:
# optionally use the domestic ubuntu/pip sources for your consideration!
#
FROM ccr.ccs.tencentyun.com/sc2ai/tleague:vbase

# basic settings
ENV WORK_DIR /root/work
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