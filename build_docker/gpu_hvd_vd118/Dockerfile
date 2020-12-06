FROM ccr.ccs.tencentyun.com/sc2ai/tleague-gpu-hvd-vd118:vbase

# basic settings
ENV WORK_DIR /root/work
#SHELL ["/bin/bash", "-cu"]
WORKDIR ${WORK_DIR}

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

