# cavium-crypto-builder

Docker build based compilation and cavium nitrox module for crypto accelration (n5pf.ko) generation for RockyLinux kernel version - 4.18. This project also supports the multi-arch builds and works for x86 and arm.

The Dockerfile has support for building the following components for linux kernel:

    1.  CAVIUM_CRYPTO_DEV_NITROX_CNN55XX

Here are the commands required to build:


For X86_64 / AMD64
-------------------
$docker buildx build --platform=linux/amd64 --build-arg "KERNEL_VERSION=4.18" --build-arg "BUILDPLATFORM=linux/amd64" --build-arg "TARGETPLATFORM=linux/amd64" --file dockerbuild/Dockerfile -t zreis-cavium-crypto-builder:4.18 .

In order to copy the generated driver module, you can either run the container from the built image or use the following command to get the kernel module for cavium nitrox (n5pf.ko) copied to /tmp on the host using the --output argument to docker.


For x86_64/AMD64
-----------------
$DOCKER_BUILDKIT=1 docker buildx build --platform=linux/amd64 --build-arg "KERNEL_VERSION=4.18" --build-arg "BUILDPLATFORM=linux/amd64" --build-arg "TARGETPLATFORM=linux/amd64" --file Dockerfile -t zreis-cavium-crypto-builder --output /tmp .

