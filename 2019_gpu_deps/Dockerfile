ARG BASE_IMAGE=provarepro/openvino:2019_cpu_deps
FROM ${BASE_IMAGE}

#Install Intel Graphics Compute Runtime for OpenCL Driver package 19.04.12237.
RUN mkdir /neo && cd /neo && \
    wget https://github.com/intel/compute-runtime/releases/download/19.04.12237/intel-gmmlib_18.4.1_amd64.deb && \
    wget https://github.com/intel/compute-runtime/releases/download/19.04.12237/intel-igc-core_18.50.1270_amd64.deb && \
    wget https://github.com/intel/compute-runtime/releases/download/19.04.12237/intel-igc-opencl_18.50.1270_amd64.deb && \
    wget https://github.com/intel/compute-runtime/releases/download/19.04.12237/intel-opencl_19.04.12237_amd64.deb && \
    wget https://github.com/intel/compute-runtime/releases/download/19.04.12237/intel-ocloc_19.04.12237_amd64.deb && \
    sudo dpkg -i *.deb && \
    cd / && rm -rf /neo

