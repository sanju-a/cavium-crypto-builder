
### ================================== ###
###   STAGE 1 CREATE BUILD IMAGE       ###
### ================================== ###

ARG ROCKY_LINUX_REL=8.8
ARG BUILDPLATFORM
ARG TARGETPLATFORM

FROM --platform=$BUILDPLATFORM rockylinux:${ROCKY_LINUX_REL} as zreis-builder-base

ENV TARGETPLATFORM=${TARGETPLATFORM:-linux/amd64} \
    BUILDPLATFORM=${BUILDPLATFORM:-linux/amd64}

RUN printf "Building for TARGETPLATFORM=${TARGETPLATFORM} on BUILDPLATFORM=${BUILDPLATFORM}\n" \
    && printf "With uname -s : " && uname -s \
    && printf "and  uname -m : " && uname -m

### ================================================== ###
###   STAGE 2 COMPILE kernel crypto drivers for cavium ###
### ================================================== ###

FROM zreis-builder-base as zreis-cavium-crypto-compiler

LABEL maintainer="Sanju Abraham <sabraham@zscaler.com> and <asanju@gmail.com>"

ARG KERNEL_VERSION=4.18

RUN dnf update -y && \
    dnf install -y wget git automake libtool autoconf gnutls epel-release make \
                   gcc gcc-c++ gettext gnutls-devel  gettext-devel kernel-headers-$(uname -r) \
                   kernel-devel-$(uname -r) kernel-modules-$(uname -r) \
                   openssl-devel bison flex ncurses-devel elfutils-libelf-devel bc diffutils \
                   groupinstall 'Development Tools' openssl-devel python3

RUN OS_VER=$( grep -E '^(VERSION)=' /etc/os-release ) && \
    if [[ $OS_VER == *"Green Obsidian"* ]]; then \
       dnf config-manager --set-enabled powertools; \
    elif [[ $OS_VER == *"Blue Onyx"* ]]; then \
       dnf config-manager --set-enabled crb; \
    else \
       echo "Unsupported OS Version $OS_VER"; \
    fi

# This step is safe to have for both rockylinu-8 and 9
RUN /usr/bin/crb enable

RUN dnf -y install dwarves

# setting github params for cloning the linux kernel
RUN echo -n "Downloading linux kernel ${KERNEL_VERSION} source code: " && \
    git init && git config --global http.postBuffer 524288000 && \
    git config --global pack.window 1 && \
    mkdir -p /tmp/crypto && cd /tmp/crypto && \  
    git clone -b v${KERNEL_VERSION} https://github.com/torvalds/linux && \
    echo "Successfully downloaded the linux kernel source code for ${KERNEL_VERSION}"

# Compile cavium nitrox driver in Linux kernel 
RUN    ls -l /tmp/crypto/ && cd /tmp/crypto/linux && git checkout v${KERNEL_VERSION} && \
       OLDIFS="$IFS" && IFS='-' && KERNEL_TOKENS=$(uname -r) && \
       echo -e "\n ===========================\n  Configure Linux Kernel \n ============================\n" && \
       make defconfig && \
       echo -e "\n\n ===========================\n Configured Linux Kernel \n =========================== \n\n" && \
       echo -e "\n\n ===========================\n Configure Cavium Nitrox driver \n =========================== \n\n" && \
       sed -i '/# CONFIG_MODVERSIONS is not set/a CONFIG_MODVERSIONS=y' .config && \
       sed -i '/# CONFIG_CRYPTO_DEV_NITROX_CNN55XX is not set/a CONFIG_CRYPTO_DEV_NITROX=m' .config && \
       sed -i '/CONFIG_CRYPTO_DEV_NITROX=m/a CONFIG_CRYPTO_DEV_NITROX_CNN55XX=m' .config && \
       sed -i '/# CONFIG_THUNDER_NIC_VF is not set/a CONFIG_THUNDER_NIC_VF=m' .config && && \
       sed -i '/# CONFIG_THUNDER_NIC_PF is not set/a CONFIG_THUNDER_NIC_PF=m' .config && \
       sed -i '/CONFIG_NET_VENDOR_CAVIUM=y is not set/a CONFIG_NET_VENDOR_CAVIUM=m' .config && \
       sed -i '/CONFIG_ETHERNET=y is not set/a CONFIG_ETHERNET=m' .config && \
       echo -e "\n\n ===========================\n Configured Cavium Nitrox driver \n =========================== \n\n" && \
       echo -e "\n\n ===========================\n Compile base vmlinux and modules_prepare \n =========================== \n\n" && \
       make EXTRAVERSION=-${KERNEL_TOKENS[1]} vmlinux && \
       make EXTRAVERSION=-${KERNEL_TOKENS[1]} modules_prepare && \
       echo -e "\n\n ===========================\n Compiled base vmlinux and modules_prepare \n =========================== \n\n" && \
       echo -e "\n\n ===========================\n Compile Cavium Nitrox driver \n =========================== \n\n" && \
       make EXTRAVERSION=-${KERNEL_TOKENS[1]} M=drivers/crypto/cavium/nitrox modules && \
       echo -e "\n\n ===========================\n Compiled Cavium Nitrox driver \n =========================== \n\n" && \
       echo -e "\n\n ===========================\n Compile Cavium Thunder driver \n =========================== \n\n" && \
       make EXTRAVERSION=-${KERNEL_TOKENS[1]} M=drivers/net/ethernet/cavium/thunder modules && \
       echo -e "\n\n ===========================\n Compiled Cavium Thunder driver \n =========================== \n\n" && \
       echo -e "\n\n ===========================\n List Cavium Nitrox driver \n =========================== \n\n" && \
       ls -ltr drivers/crypto/cavium/nitrox/n5pf.ko && \
       echo -e "\n\n ===========================\n List Cavium Thunder drivers \n =========================== \n\n" && \
       ls -ltr drivers/net/ethernet/cavium/thunder/*.ko && \
       echo -e "\n\n ===========================\n Successfully built Cavium Nitrox and Thunder drivers \n =========================== \n\n"


### ==================================================== ###
###   STAGE 6 COPY cavium nitrox kernel module to host   ###
### ==================================================== ###
 
FROM scratch AS export-stage

ARG CAVIUM_CRYPTO_NITROX_MODULE_PATH="/tmp/crypto/linux/drivers/crypto/cavium/nitrox/"
ARG CAVIUM_CRYPTO_THUNDER_MODULE_PATH="/tmp/crypto/linux/drivers/net/ethernet/cavium/thunder/"

COPY --from=zreis-cavium-crypto-compiler ${CAVIUM_CRYPTO_MODULE_PATH}/n5pf.ko .
COPY --from=zreis-cavium-crypto-compiler ${CAVIUM_CRYPTO_THUNDER_MODULE_PATH}/nicpf.ko .
COPY --from=zreis-cavium-crypto-compiler ${CAVIUM_CRYPTO_THUNDER_MODULE_PATH}/nicvf.ko .
COPY --from=zreis-cavium-crypto-compiler ${CAVIUM_CRYPTO_THUNDER_MODULE_PATH}/thunder_xcv.ko .
COPY --from=zreis-cavium-crypto-compiler ${CAVIUM_CRYPTO_THUNDER_MODULE_PATH}/thunder_bgx.ko .