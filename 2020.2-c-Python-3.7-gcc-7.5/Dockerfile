FROM ubuntu:18.04


ENV DEBIAN_FRONTEND=noninteractive \
    CONDA_DIR=/opt/conda \
    LC_ALL=C.UTF-8 \
    LANG=C.UTF-8 \
    LANGUAGE=C.UTF-8

ENV DEPENDENCIES="autoconf \
                  automake \
                  build-essential \
                  bzip2 \
                  ca-certificates \
                  cmake \
                  cpio \
                  curl \
                  fonts-liberation \
                  git \
                  gnupg2 \
                  libdrm2 \
                  libglib2.0-0 \
                  lsb-release \
                  libgtk-3-0 \
                  libtool \
                  locales \
                  sudo \
                  udev \
                  unzip \
                  wget \
                  zip"

RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        ${DEPENDENCIES} && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

RUN echo "C.UTF-8 UTF-8" > /etc/locale.gen && \
    locale-gen

# Configure environment
ENV PATH=${CONDA_DIR}/bin:${PATH} \
    MINICONDA_VERSION=4.8.2 \
    MINICONDA_MD5=87e77f097f6ebb5127c77662dfc3165e \
    CONDA_VERSION=4.8.2

ARG PYTHON_VERSION=default

WORKDIR /tmp

RUN wget --quiet https://repo.continuum.io/miniconda/Miniconda3-py37_${MINICONDA_VERSION}-Linux-x86_64.sh && \
    echo "${MINICONDA_MD5} *Miniconda3-py37_${MINICONDA_VERSION}-Linux-x86_64.sh" | md5sum -c - && \
    /bin/bash Miniconda3-py37_${MINICONDA_VERSION}-Linux-x86_64.sh -f -b -p ${CONDA_DIR} && \
    rm Miniconda3-py37_${MINICONDA_VERSION}-Linux-x86_64.sh && \
    echo "conda ${CONDA_VERSION}" >> ${CONDA_DIR}/conda-meta/pinned && \
    conda config --system --prepend channels conda-forge && \
    conda config --system --set auto_update_conda false && \
    conda config --system --set show_channel_urls true && \
    conda config --system --set channel_priority strict && \
    if [ ! ${PYTHON_VERSION} = 'default' ]; then conda install --yes python=${PYTHON_VERSION}; fi && \
    conda list python | grep '^python ' | tr -s ' ' | cut -d '.' -f 1,2 | sed 's/$/.*/' >> ${CONDA_DIR}/conda-meta/pinned && \
    conda install --quiet --yes conda && \
    conda install --quiet --yes pip && \
    conda update --all --quiet --yes && \
    conda clean --all -f -y && \
    rm -rf /root/.cache/yarn

#Install OpenVINO and 3rd-party dependencies
ARG OPENVINO_VERSION=2020.2.120
ARG DOWNLOAD_LINK=http://registrationcenter-download.intel.com/akdlm/irc_nas/16612/l_openvino_toolkit_p_$OPENVINO_VERSION.tgz

ENV INSTALLDIR=/opt/intel/openvino

RUN curl -LOJ "${DOWNLOAD_LINK}" && \
    tar -xzf ./*.tgz && \
    cd l_openvino_toolkit_p_${OPENVINO_VERSION} && \
    sed -i 's/decline/accept/g' silent.cfg && \
    ./install.sh -s silent.cfg && \
    cd /tmp && \
    rm -rf /tmp/l_openvino* && \
    ${INSTALLDIR}/install_dependencies/install_openvino_dependencies.sh

# source $INSTALLDIR/bin/setupvars.sh
ENV PATH=$INSTALLDIR/deployment_tools/model_optimizer:$INSTALLDIR/data_processing/gstreamer/bin:$INSTALLDIR/data_processing/gstreamer/bin/gstreamer-1.0:$PATH \
    LD_LIBRARY_PATH=$INSTALLDIR/data_processing/dl_streamer/lib:$INSTALLDIR/data_processing/gstreamer/lib:$INSTALLDIR/opencv/lib:$INSTALLDIR/deployment_tools/ngraph/lib:$INSTALLDIR/deployment_tools/inference_engine/external/hddl/lib:$INSTALLDIR/deployment_tools/inference_engine/external/gna/lib:$INSTALLDIR/deployment_tools/inference_engine/external/mkltiny_lnx/lib:$INSTALLDIR/deployment_tools/inference_engine/external/tbb/lib:$INSTALLDIR/deployment_tools/inference_engine/lib/intel64 \
    GST_PLUGIN_PATH=$INSTALLDIR/data_processing/dl_streamer/lib:$INSTALLDIR/data_processing/gstreamer/lib/gstreamer-1.0 \
    INTEL_CVSDK_DIR=$INSTALLDIR \
    GST_VAAPI_ALL_DRIVERS=1 \
    MODELS_PATH=/root/intel/dl_streamer/models \
    OpenCV_DIR=$INSTALLDIR/opencv/cmake \
    InferenceEngine_DIR=$INSTALLDIR/deployment_tools/inference_engine/share \
    GI_TYPELIB_PATH=$INSTALLDIR/data_processing/gstreamer/lib/girepository-1.0 \
    LIBRARY_PATH=$INSTALLDIR/data_processing/dl_streamer/lib:$INSTALLDIR/data_processing/gstreamer/lib \
    LC_NUMERIC=C \
    ngraph_DIR=$INSTALLDIR/deployment_tools/ngraph/cmake \
    GST_SAMPLES_DIR=$INSTALLDIR/data_processing/dl_streamer/samples \
    PYTHONPATH=$INSTALLDIR/python/python3.6:$INSTALLDIR/python/python3:$INSTALLDIR/deployment_tools/open_model_zoo/tools/accuracy_checker:$INSTALLDIR/deployment_tools/model_optimizer:$INSTALLDIR/data_processing/dl_streamer/python:$INSTALLDIR/data_processing/gstreamer/lib/python3.6/site-packages \
    GST_PLUGIN_SCANNER=$INSTALLDIR/data_processing/gstreamer/bin/gstreamer-1.0/gst-plugin-scanner \
    INTEL_OPENVINO_DIR=$INSTALLDIR \
    PKG_CONFIG_PATH=$INSTALLDIR/data_processing/dl_streamer/lib/pkgconfig:$INSTALLDIR/data_processing/gstreamer/lib/pkgconfig \
    HDDL_INSTALL_DIR=$INSTALLDIR/deployment_tools/inference_engine/external/hddl \
    OPENVINO_DIR=$INSTALLDIR
