# Copyright (C) 2019-2020 Intel Corporation
# SPDX-License-Identifier: Apache-2.0
FROM ubuntu:18.04 AS base

# hadolint ignore=DL3002
USER root
WORKDIR /

SHELL ["/bin/bash", "-xo", "pipefail", "-c"]

# hadolint ignore=DL3008
RUN apt-get update && \
    apt-get install -y --no-install-recommends curl ca-certificates && \
    rm -rf /var/lib/apt/lists/* && ln -snf /usr/share/zoneinfo/$(curl https://ipapi.co/timezone -k) /etc/localtime

# download source for pypi-kenlm LGPL package
WORKDIR /tmp
RUN curl -L https://files.pythonhosted.org/packages/7f/e6/1639d2de28c27632e3136015ecfd67774cca6f55146507baeaef06b113ba/pypi-kenlm-0.1.20190403.tar.gz --output pypi-kenlm.tar.gz


# get product from URL
ARG package_url="https://storage.openvinotoolkit.org/repositories/openvino/packages/2020.3.1/l_openvino_toolkit_runtime_ubuntu18_p_2020.3.341.tgz"
ARG TEMP_DIR=/tmp/openvino_installer

WORKDIR ${TEMP_DIR}
# hadolint ignore=DL3020
ADD ${package_url} ${TEMP_DIR}

# install product by copying archive content
ENV INTEL_OPENVINO_DIR /opt/intel/openvino

RUN tar -xzf "${TEMP_DIR}"/*.tgz && \
    OV_BUILD="$(find . -maxdepth 1 -type d -name "*openvino*" | grep -oP '(?<=_)\d+.\d+.\d+')" && \
    OV_YEAR="$(find . -maxdepth 1 -type d -name "*openvino*" | grep -oP '(?<=_)\d+')" && \
    OV_FOLDER="$(find . -maxdepth 1 -type d -name "*openvino*")" && \
    mkdir -p /opt/intel/openvino_"$OV_BUILD"/ && \
    cp -rf "$OV_FOLDER"/*  /opt/intel/openvino_"$OV_BUILD"/ && \
    rm -rf "${TEMP_DIR:?}"/"$OV_FOLDER" && \
    ln --symbolic /opt/intel/openvino_"$OV_BUILD"/ /opt/intel/openvino && \
    ln --symbolic /opt/intel/openvino_"$OV_BUILD"/ /opt/intel/openvino_"$OV_YEAR" && \
    rm -rf ${INTEL_OPENVINO_DIR}/deployment_tools/tools/workbench && rm -rf ${TEMP_DIR}


# for GPU
ARG GMMLIB="19.3.2"
ARG IGC_CORE="1.0.2597"
ARG IGC_OPENCL="1.0.2597"
ARG INTEL_OPENCL="19.41.14441"
ARG INTEL_OCLOC="19.41.14441"
ARG TEMP_DIR=/tmp/opencl

WORKDIR ${TEMP_DIR}
RUN curl -L "https://github.com/intel/compute-runtime/releases/download/${INTEL_OPENCL}/intel-gmmlib_${GMMLIB}_amd64.deb" --output "intel-gmmlib_${GMMLIB}_amd64.deb" && \
    curl -L "https://github.com/intel/compute-runtime/releases/download/${INTEL_OPENCL}/intel-igc-core_${IGC_CORE}_amd64.deb" --output "intel-igc-core_${IGC_CORE}_amd64.deb" && \
    curl -L "https://github.com/intel/compute-runtime/releases/download/${INTEL_OPENCL}/intel-igc-opencl_${IGC_OPENCL}_amd64.deb" --output "intel-igc-opencl_${IGC_OPENCL}_amd64.deb" && \
    curl -L "https://github.com/intel/compute-runtime/releases/download/${INTEL_OPENCL}/intel-opencl_${INTEL_OPENCL}_amd64.deb" --output "intel-opencl_${INTEL_OPENCL}_amd64.deb" && \
    curl -L "https://github.com/intel/compute-runtime/releases/download/${INTEL_OPENCL}/intel-ocloc_${INTEL_OCLOC}_amd64.deb" --output "intel-ocloc_${INTEL_OCLOC}_amd64.deb"

# for VPU
ARG BUILD_DEPENDENCIES="autoconf \
                        automake \
                        build-essential \
                        libtool \
                        unzip"

# hadolint ignore=DL3008
RUN apt-get update && \
    apt-get install -y --no-install-recommends ${BUILD_DEPENDENCIES} && \
    rm -rf /var/lib/apt/lists/*

WORKDIR /opt
RUN curl -L https://github.com/libusb/libusb/archive/v1.0.22.zip --output v1.0.22.zip && \
    unzip v1.0.22.zip && rm -rf v1.0.22.zip

WORKDIR /opt/libusb-1.0.22
RUN ./bootstrap.sh && \
    ./configure --disable-udev --enable-shared && \
    make -j4

# -----------------
FROM ubuntu:18.04 AS ov_base

LABEL Description="This is the runtime image for Intel(R) Distribution of OpenVINO(TM) toolkit on Ubuntu 18.04 LTS"
LABEL Vendor="Intel Corporation"

USER root
WORKDIR /

SHELL ["/bin/bash", "-xo", "pipefail", "-c"]

# Creating user openvino and adding it to groups "video" and "users" to use GPU and VPU
RUN useradd -ms /bin/bash -G video,users openvino && \
    chown openvino -R /home/openvino

RUN mkdir /opt/intel

ENV INTEL_OPENVINO_DIR /opt/intel/openvino

COPY --from=base /opt/intel /opt/intel


ARG LGPL_DEPS=libgtk-3-0


# hadolint ignore=DL3008
RUN sed -Ei 's/# deb-src /deb-src /' /etc/apt/sources.list && \
    apt-get update && \
    apt-get install -y --no-install-recommends dpkg-dev curl ${LGPL_DEPS} && \
    apt-get source ${LGPL_DEPS} && \
    rm -rf /var/lib/apt/lists/* && ln -snf /usr/share/zoneinfo/$(curl https://ipapi.co/timezone -k) /etc/localtime


# setup Python
ENV PYTHON_VER python3.6


# hadolint ignore=DL3008
RUN apt-get update && \
    apt-get install -y --no-install-recommends python3-pip python3-setuptools lib${PYTHON_VER} && \
    rm -rf /var/lib/apt/lists/*


RUN ${PYTHON_VER} -m pip install --upgrade pip

# runtime package
WORKDIR /tmp

RUN ${PYTHON_VER} -m pip install --no-cache-dir -r ${INTEL_OPENVINO_DIR}/python/${PYTHON_VER}/requirements.txt && \
    if [ -f ${INTEL_OPENVINO_DIR}/data_processing/dl_streamer/requirements.txt ]; then \
        ${PYTHON_VER} -m pip install --no-cache-dir -r ${INTEL_OPENVINO_DIR}/data_processing/dl_streamer/requirements.txt; \
    fi

# for CPU

# for GPU
ARG TEMP_DIR=/tmp/opencl

COPY --from=base ${TEMP_DIR} ${TEMP_DIR}

WORKDIR ${TEMP_DIR}
# hadolint ignore=DL3008
RUN apt-get update && \
    apt-get install -y --no-install-recommends ocl-icd-libopencl1 && \
    rm -rf /var/lib/apt/lists/* && \
    dpkg -i ${TEMP_DIR}/*.deb && \
    ldconfig && \
    rm -rf ${TEMP_DIR}

# for VPU
ARG DEPENDENCIES=udev

# hadolint ignore=DL3008
RUN apt-get update && \
    apt-get install -y --no-install-recommends ${DEPENDENCIES} && \
    apt-get source ${DEPENDENCIES} && \
    rm -rf /var/lib/apt/lists/*

COPY --from=base /opt/libusb-1.0.22 /opt/libusb-1.0.22

WORKDIR /opt/libusb-1.0.22/libusb
RUN /bin/mkdir -p '/usr/local/lib' && \
    /bin/bash ../libtool   --mode=install /usr/bin/install -c   libusb-1.0.la '/usr/local/lib' && \
    /bin/mkdir -p '/usr/local/include/libusb-1.0' && \
    /usr/bin/install -c -m 644 libusb.h '/usr/local/include/libusb-1.0' && \
    /bin/mkdir -p '/usr/local/lib/pkgconfig'

WORKDIR /opt/libusb-1.0.22/
RUN /usr/bin/install -c -m 644 libusb-1.0.pc '/usr/local/lib/pkgconfig' && \
    cp ${INTEL_OPENVINO_DIR}/deployment_tools/inference_engine/external/97-myriad-usbboot.rules /etc/udev/rules.d/ && \
    ldconfig

# for HDDL
WORKDIR /tmp
# hadolint ignore=DL3008
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        libboost-filesystem1.65-dev \
        libboost-thread1.65-dev \
        libjson-c3 libxxf86vm-dev && \
    rm -rf /var/lib/apt/lists/* && rm -rf /tmp/*


# Post-installation cleanup and setting up OpenVINO environment variables
RUN if [ -f "${INTEL_OPENVINO_DIR}"/bin/setupvars.sh ]; then \
        printf "\nexport TBB_DIR=\${INTEL_OPENVINO_DIR}/deployment_tools/inference_engine/external/tbb/cmake\n" >> ${INTEL_OPENVINO_DIR}/bin/setupvars.sh; \
        printf "\nsource \${INTEL_OPENVINO_DIR}/bin/setupvars.sh\n" >> /home/openvino/.bashrc; \
        printf "\nsource \${INTEL_OPENVINO_DIR}/bin/setupvars.sh\n" >> /root/.bashrc; \
    fi; \
    if [ -d "${INTEL_OPENVINO_DIR}"/opt/intel/mediasdk ]; then \
        printf "\nexport LIBVA_DRIVER_NAME=iHD \nexport LIBVA_DRIVERS_PATH=\${INTEL_OPENVINO_DIR}/opt/intel/mediasdk/lib64/ \nexport GST_VAAPI_ALL_DRIVERS=1 \nexport LIBRARY_PATH=\${INTEL_OPENVINO_DIR}/opt/intel/mediasdk/lib64/:\$LIBRARY_PATH \nexport LD_LIBRARY_PATH=\${INTEL_OPENVINO_DIR}/opt/intel/mediasdk/lib64/:\$LD_LIBRARY_PATH \n" >> /home/openvino/.bashrc; \
        printf "\nexport LIBVA_DRIVER_NAME=iHD \nexport LIBVA_DRIVERS_PATH=\${INTEL_OPENVINO_DIR}/opt/intel/mediasdk/lib64/ \nexport GST_VAAPI_ALL_DRIVERS=1 \nexport LIBRARY_PATH=\${INTEL_OPENVINO_DIR}/opt/intel/mediasdk/lib64/:\$LIBRARY_PATH \nexport LD_LIBRARY_PATH=\${INTEL_OPENVINO_DIR}/opt/intel/mediasdk/lib64/:\$LD_LIBRARY_PATH \n" >> /root/.bashrc; \
    fi;

RUN apt-get update && \
    apt-get autoremove -y dpkg-dev && \
    apt-get install -y --no-install-recommends ${LGPL_DEPS} && \
    rm -rf /var/lib/apt/lists/*

USER openvino
WORKDIR ${INTEL_OPENVINO_DIR}

CMD ["/bin/bash"]

# Setup custom layers below
