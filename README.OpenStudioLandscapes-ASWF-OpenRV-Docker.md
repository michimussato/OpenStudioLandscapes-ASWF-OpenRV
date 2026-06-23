[![ Logo OpenStudioLandscapes ](https://github.com/michimussato/OpenStudioLandscapes/raw/main/media/images/logo128.png)](https://github.com/michimussato/OpenStudioLandscapes)

---

<!-- TOC -->
* [Build OpenRV with Docker](#build-openrv-with-docker)
  * [Overview - Decision Tree](#overview---decision-tree)
  * [Step 1 - Build Docker Image](#step-1---build-docker-image)
    * [Dockerfile.Linux-Rocky9-CY2024](#dockerfilelinux-rocky9-cy2024)
  * [Step 2 - Run the container and enter](#step-2---run-the-container-and-enter)
    * [CY2024](#cy2024)
  * [Step 3 - Build OpenRV](#step-3---build-openrv)
  * [Step 4 - Extract Stage Folder to local Machine](#step-4---extract-stage-folder-to-local-machine)
    * [Stage](#stage)
    * [Installable Package](#installable-package)
    * [Build Logs](#build-logs)
<!-- TOC -->

---

# Build OpenRV with Docker

## Overview - Decision Tree

Simplified decision tree:

```mermaid
---
config:
  flowchart:
    htmlLabels: false
---
flowchart TB
    start(("Start"))
    clone("`git clone --recursive https://github.com/michimussato/OpenStudioLandscapes-ASWF-OpenRV.git`")
    cd("`cd OpenRV`")
    export("`export RV_VFX_PLATFORM='CY2024'`")
    source("`source rvcmds.sh`")
    rvcfg("`rvcfg`")
    rvbuild("`rvbuild`")
    rvclean("`rvclean`")
    fail(("`fail`"))
    rverrsummary("`rverrsummary`")
    rverrors("`rverrors`")
    
    success(("Success"))
    
    start --> clone
    clone --> cd
    cd --> export
    export --> source
    
    %%source ----> rvbootstrap
    source --> rvsetup
    
    rvsetup --> rvcfg
    rvcfg --> rvbuild
    
    rvbuild ------> success
    
    rvbuild <-..-> fail
    
    fail -.-> rverrors
    fail -.-> rverrsummary
    
    fail --> rvclean
    rvclean --> rvsetup
```

## Step 1 - Build Docker Image

[Building with Docker](https://aswf-openrv.readthedocs.io/en/latest/build_system/config_linux_rocky89.html#building-with-docker-optional)

```shell
git clone --recursive https://github.com/michimussato/OpenStudioLandscapes-ASWF-OpenRV.git
cd OpenStudioLandscapes-ASWF-OpenRV
```

> [!TIP]
> 
> The next step nees a lot of disk space: (~35 GiB)!

### Dockerfile.Linux-Rocky9-CY2024

#### Create Builder

> [!TIP]
> 
> Prevent Docker Log Clipping for this build
> - i.e. [output clipped, log limit 2MiB reached]

`buildx`
- https://github.com/docker/buildx/issues/484#issuecomment-3299100959
- https://oneuptime.com/blog/post/2026-02-08-how-to-use-docker-buildx-commands-for-advanced-builds/view
- https://docs.docker.com/build/builders/drivers/


```shell
docker buildx \
    create \
    --name openrv_builder \
    --driver-opt env.BUILDKIT_STEP_LOG_MAX_SPEED=-1 \
    --driver-opt env.BUILDKIT_STEP_LOG_MAX_SIZE=-1
#    `--driver-opt default-load=true` OR `--output type=docker` in build command
```

> [!IMPORTANT]
> 
> Don't remove custom builder if cache is needed
> - https://docs.docker.com/build/cache/backends/local/
> `docker buildx rm openrv_builder`

#### Build to Target

```mermaid
---
config:
  flowchart:
    htmlLabels: false
---
flowchart TB
    
    rocky9(("rockylinux/rockylinux:9"))
    
    subgraph "Build System"
        direction TB
        base_os(("base_os"))
        build_system(("build_system"))
        
        subgraph "Tags"
            direction TB
            
            build_system_image["openstudiolandscapes-aswf-openrv-rocky9-cy2024-build_system:latest"]
        end
        
    end
    
    subgraph "Deployment System"
        direction TB
        rv_base(("rv_base"))
    
        stage_on_base_os(("stage_on_base_os"))
        installable_on_base_os(("installable_on_base_os"))
        stage_on_rocky(("stage_on_rocky"))
        installable_on_rocky(("installable_on_rocky"))
        
    end

    rocky9 ----> base_os
    base_os --> build_system
    build_system -.-> Tags
    rocky9 ----> rv_base
    
    rv_base ----> stage_on_base_os
    rv_base ----> installable_on_base_os
    rv_base ----> stage_on_rocky
    rv_base ----> installable_on_rocky
    
    build_system_image ~~~~~~~~~ stage_on_base_os
    build_system_image -. "COPY --from /home/rv/OpenRV/_build" .-> stage_on_base_os
    build_system_image ~~~~~~~~~ installable_on_base_os
    build_system_image -. "COPY --from /home/rv/OpenRV/_install" .-> installable_on_base_os
    build_system_image ~~~~~~~~~ stage_on_rocky
    build_system_image -. "COPY --from /home/rv/OpenRV/_build" .-> stage_on_rocky
    build_system_image ~~~~~~~~~ installable_on_rocky
    build_system_image -. "COPY --from/home/rv/OpenRV/_install" .-> installable_on_rocky
```

##### `build_system`

> [!TIP]
> 
> This takes a lot of time, uses 
> up a lot of space (~35GiB) and 
> caching does not seem to work for 
> the `Qt` installation layers.
> For testing, it makes a lot of sense
> to break this step out and create 
> an image from it so that we can
> re-use it for subsequent steps.

Build `build_system`

```shell
# stage
export TARGET="build_system"

# use `time`?
time docker build \
    --builder openrv_builder \
    --target ${TARGET} \
    --output type=docker \
    --progress plain \
    --shm-size=32g \
    --tag openstudiolandscapes-aswf-openrv-rocky9-cy2024-${TARGET}:$(date "+%Y-%m-%d_%H-%M-%S") \
    --tag openstudiolandscapes-aswf-openrv-rocky9-cy2024-${TARGET}:latest \
    --file dockerfiles/Dockerfile.Linux-Rocky9-CY2024 \
    ./dockerfiles  # --no-cache
# Disable cache: --no-cache

NTFY_RSNAPSHOT_TOKEN=tk_amlipjwa7eb3rpxd00rsdshz5vyh5 \
    && /usr/bin/curl \
        -H "X-Title: OpenRV" \
        -H "Authorization: Bearer ${NTFY_RSNAPSHOT_TOKEN}" \
        -d "Build to target ${TARGET} finished." https://ntfy.pangolin.openstudiolandscapes.cloud-ip.cc/builds
```

> [!TIP]
> 
> To enter `build_system` interactively:
> ```shell
> export TARGET="build_system"
> 
> docker run \
>     --shm-size=32g \
>     --rm \
>     --interactive \
>     --tty \
>     --name OpenStudioLandscapes-ASWF-OpenRV-BuildBox-CY2024-${TARGET} \
>     openstudiolandscapes-aswf-openrv-rocky9-cy2024-${TARGET}:latest /bin/bash
> ```
> 
> To create a `tar` archive from the resulting build:
> ```shell
> source /etc/os-release
> mkdir -p ./tarballs
> pushd ./tarballs || exit 1
> 
> tar -C /home/rv/OpenRV/_install \
>     --create \
>     --verbose \
>     --file - . \
>     | xz \
>     --verbose \
>     --threads=0 \
>     -9 \
>     --stdout - \
>     > OpenRV-$(/home/rv/OpenRV/_install/bin/rv -version)-${ROCKY_SUPPORT_PRODUCT}-$(uname --hardware-platform).tar.xz
> 
> popd || exit 1
> ```
> 
> To copy the resulting `tar` archive to the Docker host:
> ```shell
> export TARGET="build_system"
> 
> mkdir -p ./OpenStudioLandscapes-ASWF-OpenRV-BuildBox-CY2024-${TARGET}
> docker cp \
>     OpenStudioLandscapes-ASWF-OpenRV-BuildBox-CY2024-${TARGET}:/home/rv/OpenRV/tarballs/. \
>     ./OpenStudioLandscapes-ASWF-OpenRV-BuildBox-CY2024-${TARGET}
> ```








##### `installable_on_rocky`

```shell
# stage
export TARGET="installable_on_rocky"

# use `time`?
docker build \
    --builder openrv_builder \
    --target ${TARGET} \
    --output type=docker \
    --progress plain \
    --shm-size=32g \
    --tag openstudiolandscapes-aswf-openrv-rocky9-cy2024-${TARGET}:$(date "+%Y-%m-%d_%H-%M-%S") \
    --tag openstudiolandscapes-aswf-openrv-rocky9-cy2024-${TARGET}:latest \
    --file dockerfiles/Dockerfile.Linux-Rocky9-CY2024 \
    ./dockerfiles  # --no-cache
# Disable cache: --no-cache

NTFY_RSNAPSHOT_TOKEN=tk_amlipjwa7eb3rpxd00rsdshz5vyh5 \
    && /usr/bin/curl \
        -H "X-Title: OpenRV " \
        -H "Authorization: Bearer ${NTFY_RSNAPSHOT_TOKEN}" \
        -d "Build to target ${TARGET} finished." https://ntfy.pangolin.openstudiolandscapes.cloud-ip.cc/builds
```

## Step 2 - Run the container and enter

### CY2024

```shell
docker run \
    --shm-size=32g \
    --rm \
    --interactive \
    --tty \
    --name OpenStudioLandscapes-ASWF-OpenRV-BuildBox-CY2024 \
    openstudiolandscapes-aswf-openrv-rocky9-cy2024-${TARGET}:latest /bin/bash
```

## Step 3 - Build OpenRV

[Building Open RV](https://aswf-openrv.readthedocs.io/en/latest/build_system/config_common_build.html)

```shell
git clone --recursive https://github.com/AcademySoftwareFoundation/OpenRV.git
cd OpenRV
# Set RV_VFX_PLATFORM to 2024 so that we won't be confronted with an interactive
# questionnaire:
export RV_VFX_PLATFORM="CY2024"
source rvcmds.sh

# rvbootstrap is an alias
# alias rvbootstrap='rvsetup && rvmk'
# Hence, if the docs already suggest to run rvmk if rvbootstrap fails (which
# does leave "mixed feelings"), maybe it's just better to **not** use rvbootstrap
# in the first place as it is obviously considered wonky.
#
# Simpler approach:
rvsetup && rvcfg && rvbuild || rvbuild

# Non Free Codecs
# alias rvcfg='rvhomedir && rvenv && cmake -B ${RV_BUILD_DIR} -G "${CMAKE_GENERATOR}" ${RV_TOOLCHAIN} ${CMAKE_WIN_ARCH} -DCMAKE_BUILD_TYPE=${RV_BUILD_TYPE} -DRV_DEPS_QT_LOCATION=${QT_HOME} -DRV_VFX_PLATFORM=${RV_VFX_PLATFORM} -DRV_DEPS_WIN_PERL_ROOT=${WIN_PERL}'
# FFMPEG
# rvcfg -DRV_FFMPEG_NON_FREE_DECODERS_TO_ENABLE="aac;hevc" -DRV_FFMPEG_NON_FREE_ENCODERS_TO_ENABLE="aac"
# ProRes
# https://aswf-openrv.readthedocs.io/en/latest/build_system/config_common_build.html#apple-prores

# Creating Installation Package
# https://aswf-openrv.readthedocs.io/en/latest/build_system/config_common_build.html#creating-the-installation-package
# alias rvinst='rvenv && cmake --install ${RV_BUILD_DIR} --prefix ${RV_INST_DIR} --config ${RV_BUILD_TYPE}'
# rel (.venv) [rv@c865f3be91dd OpenRV]$ echo ${RV_BUILD_DIR}
# /home/rv/OpenRV/_build
# rel (.venv) [rv@c865f3be91dd OpenRV]$ echo ${RV_INST_DIR}
# /home/rv/OpenRV/_install
# rel (.venv) [rv@c865f3be91dd OpenRV]$ echo ${RV_BUILD_TYPE}
# Release
# cmake --install _build --prefix _install
cmake --install ${RV_BUILD_DIR} --prefix ${RV_INST_DIR} --config ${RV_BUILD_TYPE}
```

## Step 4 - Extract Stage Folder to local Machine

### Stage

```shell
mkdir -p ./OpenStudioLandscapes-ASWF-OpenRV-BuildBox-CY2024/stage

# Container id is the same as the one used in the step above
docker cp \
    OpenStudioLandscapes-ASWF-OpenRV-BuildBox-CY2024:/home/rv/OpenRV/_build/stage/. \
    ./OpenStudioLandscapes-ASWF-OpenRV-BuildBox-CY2024/stage
```

### Installable Package

```shell
mkdir -p ./OpenStudioLandscapes-ASWF-OpenRV-BuildBox-CY2024/install

# Container id is the same as the one used in the step above
docker cp \
    OpenStudioLandscapes-ASWF-OpenRV-BuildBox-CY2024:/home/rv/OpenRV/_install/. \
    ./OpenStudioLandscapes-ASWF-OpenRV-BuildBox-CY2024/install
```

### Build Logs

```shell
mkdir -p ./OpenStudioLandscapes-ASWF-OpenRV-BuildBox-CY2024/logs

docker cp \
    OpenStudioLandscapes-ASWF-OpenRV-BuildBox-CY2024:/home/rv/OpenRV/_build/error_summary.txt \
    ./OpenStudioLandscapes-ASWF-OpenRV-BuildBox-CY2024/logs || echo "No error summary log found. Build may have succeeded or not run yet."

docker cp \
    OpenStudioLandscapes-ASWF-OpenRV-BuildBox-CY2024:/home/rv/OpenRV/_build/build_errors.log \
    ./OpenStudioLandscapes-ASWF-OpenRV-BuildBox-CY2024/logs || echo "No build error log found. Build may have succeeded or not run yet."
```
