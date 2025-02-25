# ============================================================================ #
# Copyright (c) 2024 NVIDIA Corporation & Affiliates.                          #
# All rights reserved.                                                         #
#                                                                              #
# This source code and the accompanying materials are made available under     #
# the terms of the Apache License 2.0 which accompanies this distribution.     #
# ============================================================================ #


# To build the release image run the following from the top-level of the repo (must clone the cudaqx-private repo)
#
# docker build -t ghcr.io/nvidia/cudaqx-private:latest-$(uname -m) . -f docker/release/Dockerfile 
#
# Test the image with 
# 
# scripts/validation/container/validate_container.sh 

FROM ghcr.io/nvidia/private/cuda-quantum:cu12-0.9.0 as build 

ARG GITHUB_UNAME
ARG GITHUB_API_TOKEN
ARG DEBIAN_FRONTEND=noninteractive
USER root

RUN apt-get update && apt-get install -y g++ gfortran libblas-dev ninja-build \
    && python3 -m pip install cmake --user

ADD . .

RUN mkdir build && cd build \
    && ~/.local/bin/cmake .. -G Ninja -DCMAKE_INSTALL_PREFIX=/opt/nvidia/cudaq \
          -DCUDAQ_DIR=/opt/nvidia/cudaq/lib/cmake/cudaq \
          -DCUDAQX_ENABLE_LIBS="all" \
          -DCUDAQX_INCLUDE_TESTS=ON \
          -DCUDAQX_BINDINGS_PYTHON=ON \
          -DCUDAQX_INSTALL_PYTHON=ON \
          -DCMAKE_BUILD_TYPE=Release \
    && ninja install && pwd

FROM ghcr.io/nvidia/private/cuda-quantum:cu12-0.9.0 

USER root

COPY --from=build "/opt/nvidia/cudaq/" "/opt/nvidia/cudaq"
COPY --from=build "/home/cudaq/docs/sphinx/examples" "/home/cudaq/cudaqx-examples"
COPY --from=build "/home/cudaq/libs/qec/python/tests" "/opt/nvidia/cudaq/cudaqx_pytests/qec"
COPY --from=build "/home/cudaq/libs/solvers/python/tests" "/opt/nvidia/cudaq/cudaqx_pytests/solvers"

RUN chown -R cudaq /home/cudaq && chgrp -R cudaq /home/cudaq
RUN apt-get update && apt-get install -y g++ gfortran 

USER cudaq

RUN python3 -m pip install --user mpi4py \
             pyscf fastapi uvicorn cupy-cuda12x nvidia-cublas-cu12 networkx 

WORKDIR /home/cudaq
ENTRYPOINT ["bash", "-l"]
