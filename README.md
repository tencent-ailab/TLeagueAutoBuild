For `TLeague` + `Pommerman` docker building.


# How to use
## Manual Docker-build
This repo includes `Arena`, `TPolicies` and `TLeague` as git submodules.
Please git clone the repo recursively and cd to it:
```bash
git clone --recurse-submodules https://github.com/tencent-ailab/TLeagueAutoBuild.git ~/TLeagueAutoBuild
cd ~/TLeagueAutoBuild
```

First build the base images (one for GPU and the other for CPU):
```bash
docker build -f build_docker/gpu_hvd_base/Dockerfile -t tleague-gpu-hvd:vbase .
docker build -f build_docker/cpu_base/Dockerfile -t tleague:vbase .
```
and push it to your own (remote) docker registry.
Note, modify `gpu_hvd_base/Dockerfile` (and `cpu_base/Dockerfile`) if you don't need to use a mirror site for ubuntu and pip.

Then build the full image (one for GPU and the other for CPU):
```bash
docker build -f build_docker/gpu_hvd_pommerman/Dockerfile -t tleague-gpu-hvd-pommerman:latest .
docker build -f build_docker/cpu_pommerman/Dockerfile -t tleague-pommerman:latest .
```
which allows a fast re-building when only the `TLeague` (`Arena`, `TPolicies`) code gets updated. 
Note, modify `gpu_hvd_pommerman/Dockerfile` (and `cpu_pommerman/Dockerfile`) to use your own remote docker registry for the base image.
When building on local machine, 
simply change the line `FROM ccr.ccs.tencentyun.com/sc2ai/tleague-gpu-hvd:vbase` to `FROM tleague-gpu-hvd:vbase`.

## Auto Docker-build
The auto docker-build relies on [orange-ci](http://orange-ci.oa.com),
which is currently Tencent internal tool but can be conveniently replaced with public counterpart, e.g., Travis-CI.
Henceforth the corresponding scripts/configs can serve as an example for how to do the auto-build during the development cycle.

One should write what to build and where to push in the file `.orange-ci.yml` and get the web-hook well configured.
Then, any commit pushing triggers the building.
A commit can be:
* Update each submodule (i.e., updated `TLeague`, `Arena`, `TPolicies` ).
* Push an empty commit, e.g., `git commit --allow-empty -m "build the docker" && git push`