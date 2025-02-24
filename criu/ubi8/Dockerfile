# (C) Copyright IBM Corporation 2023, 2023
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#      https://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#

################################################################################
# Requires FTP3 credentials for ftp3.linux.ibm.com
# Set credentials here or at build time using --build-arg option,
# e.g. docker build --build-arg FTP3_USER=<user name> --build-arg FTP3_PASSWORD=<password> -f Dockerfile .
################################################################################

FROM registry.access.redhat.com/ubi8/ubi:latest

# Must have FTP3 account
# Set your FTP3 credentials here or pass them at build time
ARG FTP3_USER="YOUR FTP3 USER NAME"
ARG FTP3_PASSWORD="YOUR PASSWORD"

ARG CRIU_REPO=https://github.com/checkpoint-restore/criu.git
ARG CRIU_BRANCH=criu-dev
# Last stable commit SHA
ARG CRIU_SHA=f7bcd1c8c32bd04df72dfac71ba8da3dbd36b245

# Cherry pick fix to restoring SO_BUF_LOCK (from developer fork)
ARG CRIU_FIX_REPO=https://github.com/ymanton/criu.git
ARG CRIU_FIX_SHA=d46233dc7775539c275710e18733ed9b5aceb153

# First compile criu required for checkpoint
# CRIU dependencies
RUN dnf install -y \
    iptables-libs \
    jansson \
    libibverbs \
    libcap \
    libmnl \
    libnet \
    libnftnl \
    libpcap \
    nftables \
    protobuf-c

# Download script to install CRIU build dependencies
RUN cd /tmp \
    && curl -k -u ${FTP3_USER}:${FTP3_PASSWORD} -o ibm-yum.sh ftp://ftp3.linux.ibm.com/redhat/ibm-yum.sh \
    && chmod a+x ibm-yum.sh

# CRIU build dependencies

RUN set -eux; \
    export FTP3USER=${FTP3_USER} \
    && export FTP3PASS=${FTP3_PASSWORD} \
    && /tmp/ibm-yum.sh install -y \
        glibc-locale-source \
        kernel-devel \
        make \
        automake \
        gcc \
        gcc-c++ \
        glibc \
        vim \
        git \
        asciidoc \
        gnutls-devel \
        libcap-devel \
        libnet-devel \
        libnl3-devel \
        nftables \
        nftables-devel \
        pkgconfig \
        protobuf-c \
        protobuf-c-devel \
        protobuf-devel \
        python3 \
        python3-protobuf \
        xmlto \
    && unset FTP3PASS \
    && unset FTP3USER \
    && cd ../ \
    && rm -f ibm-yum.sh

RUN cd /tmp; \
    git clone -b ${CRIU_BRANCH} ${CRIU_REPO}; \
    mv criu criu-build; \
    cd criu-build; \
    git config --global user.email "build@example.com"; \
    git config --global user.name "build"; \
    if [ "x${CRIU_SHA}" == x ] ; then \
        CRIU_SHA=$(git rev-parse HEAD); \
    fi; \
    git reset --hard ${CRIU_SHA}; \
    if [ "x${CRIU_FIX_SHA}" != x ] ; then \
        git remote add dev ${CRIU_FIX_REPO}; \
        git fetch dev; \
        git cherry-pick ${CRIU_FIX_SHA}; \
    fi; \
    make; \
    make DESTDIR=/tmp/criu install-lib install-criu; \
    cd ../; \
    tar -czf criu.tar.gz criu; \
    sha256sum criu.tar.gz > criu.tar.gz.sha256.txt; \
    rm -fr criu-build;
