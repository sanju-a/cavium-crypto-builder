# cavium-crypto-builder

Docker build based compilation and cavium nitrox module for crypto accelration (n5pf.ko) generation for RockyLinux kernel version - 4.18. This project also supports the multi-arch builds and works for x86 and arm.

The Dockerfile has support for building the following components for linux kernel:

    1.  CAVIUM_CRYPTO_DEV_NITROX_CNN55XX

Here are the commands required to build:


For ARM64
------------------
$DOCKER_BUILDKIT=1 docker buildx build --progress=plain --platform=linux/arm64 --build-arg="ROCKY_LINUX_REL=9.2" --build-arg="KERNEL_VERSION=5.14" --build-arg="BUILDPLATFORM=linux/arm64" --build-arg="TARGETPLATFORM=linux/arm64" --file Dockerfile -t zreis-cavium-crypto-builder:5.14 .

For X86_64 / AMD64
-------------------
$docker buildx build --platform=linux/amd64 --build-arg "KERNEL_VERSION=4.18" --build-arg "BUILDPLATFORM=linux/amd64" --build-arg "TARGETPLATFORM=linux/amd64" --file dockerbuild/Dockerfile -t zreis-cavium-crypto-builder:4.18 .

In order to copy the generated driver module, you can either run the container from the built image or use the following command to get the kernel module for cavium nitrox (n5pf.ko) copied to /tmp on the host using the --output argument to docker.


For ARM64
-----------------
$DOCKER_BUILDKIT=1 docker buildx build --progress=plain --platform=linux/arm64 --build-arg="ROCKY_LINUX_REL=9.2" --build-arg="KERNEL_VERSION=5.14" --build-arg="BUILDPLATFORM=linux/arm64" --build-arg="TARGETPLATFORM=linux/arm64" --file Dockerfile -t zreis-cavium-crypto-builder:5.14 --output /tmp .

For x86_64/AMD64
-----------------
$DOCKER_BUILDKIT=1 docker buildx build --platform=linux/amd64 --build-arg "KERNEL_VERSION=4.18" --build-arg "BUILDPLATFORM=linux/amd64" --build-arg "TARGETPLATFORM=linux/amd64" --file Dockerfile -t zreis-cavium-crypto-builder:4.18 --output /tmp .

