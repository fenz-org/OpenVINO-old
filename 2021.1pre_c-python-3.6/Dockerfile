ARG BASE_IMAGE=provarepro/openvino:2021.1pre_cpu_deps
FROM ${BASE_IMAGE}

USER root

SHELL ["/bin/bash", "-xo", "pipefail", "-c"]

RUN cd /openvino && \
    mkdir build && cd build && \
    cmake \
        -DCMAKE_BUILD_TYPE=Release \
        -DENABLE_PYTHON=ON \
        -DPYTHON_EXECUTABLE=/usr/bin/python${PYTHON_VERSION} \
        -DPYTHON_LIBRARY=/usr/lib/x86_64-linux-gnu/libpython${PYTHON_VERSION}m.so \
        -DPYTHON_INCLUDE_DIR=/usr/include/python${PYTHON_VERSION} \
        -DENABLE_OPENCV=OFF \
        -DENABLE_VPU=OFF \
        -DENABLE_CLDNN=OFF \
        -DENABLE_GNA=OFF \
        -DENABLE_TESTS=OFF \
        -DTHREADING=OMP \
        **-DNGRAPH_ONNX_IMPORT_ENABLE=OFF \
        -DNGRAPH_DEPRECATED_ENABLE=FALSE** \
        .. && \
    make --jobs=$(nproc --all)
