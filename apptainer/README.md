<!-- TOC -->
* [Apptainer](#apptainer)
  * [Install](#install)
    * [Arch](#arch)
      * [Secondary Deps](#secondary-deps)
    * [Debian](#debian)
      * [Ubuntu](#ubuntu)
    * [Red Hat](#red-hat)
* [OpenRV](#openrv)
  * [Resources](#resources)
    * [Alternate Approaches](#alternate-approaches)
  * [Build](#build)
    * [Apptainer](#apptainer-1)
    * [From Docker Image](#from-docker-image)
  * [Non Free Codecs](#non-free-codecs)
  * [Enable Non Free Codecs](#enable-non-free-codecs)
    * [AAC](#aac)
    * [HEVC](#hevc)
  * [Run](#run)
    * [Issues](#issues)
      * [`GLIBC_2.38' not found](#glibc_238-not-found)
<!-- TOC -->

---

# Apptainer

Resources:
- Convert existing Docker images to Apptainer images:
  - from Docker Hub:
    - [How to convert a Docker container to an Apptainer container](https://rcs.ucalgary.ca/How_to_convert_a_Docker_container_to_an_Apptainer_container)
  - local images:
    - [Using Containers in Supercomputing Environment](https://yetulaxman.github.io/ContainersHPC2024/hands-on/local_docker_to_apptainer.html)
    - [ucla-oarc-cs/docker-to-apptainer](https://github.com/ucla-oarc-cs/docker-to-apptainer)

## Install

Install
- `apptainer`
- GNU `time` (vs. bash built-in time)

### Arch

https://www.baeldung.com/linux/process-peak-memory-usage

```shell
sudo pacman -Syy apptainer time
```

#### Secondary Deps

```shell
sudo pacman -Syy gocryptfs
```

### Debian

```shell
sudo apt-get install apptainer time
```

```shell
sudo apt update
sudo apt install -y wget
cd /tmp
wget https://github.com/apptainer/apptainer/releases/download/v1.4.1/apptainer_1.4.1_amd64.deb
sudo apt install -y ./apptainer_1.4.1_amd64.deb
```

#### Ubuntu

```shell
sudo apt-get update
sudo apt-get install -y software-properties-common

sudo add-apt-repository -y ppa:apptainer/ppa
sudo apt-get update
sudo apt-get install -y apptainer
```

### Red Hat

```shell
sudo dnf install apptainer time
```

# OpenRV

## Resources

- https://github.com/AcademySoftwareFoundation/OpenRV
- https://deepwiki.com/AcademySoftwareFoundation/OpenRV/2.2-building-openrv
- https://aswf-openrv.readthedocs.io/en/latest/

### Alternate Approaches

- https://github.com/col-one/OpenRV-www
- https://github.com/rubikstriangle/OpenRV-Rocky9-Docker

## Build

### Apptainer

```shell
export APPTAINERFILE="OpenRV.def"
export TIMESTAMP=$(date +"%Y-%m-%d_%H-%M-%S")

/usr/bin/time -f 'Commandline Args: %C\nElapsed Time: %E\nPeak Memory: %M\nExit Code: %x' \
    apptainer \
    --config ./apptainer/conf/apptainer.conf \
    --verbose \
    build \
    --ignore-fakeroot-command \
    --build-arg-file .env \
    --warn-unused-build-args \
    ./apptainer/.sif/OpenRV_${TIMESTAMP}.sif \
    ./apptainer/${APPTAINERFILE} \
    > >(tee -a ./apptainer/.logs/${APPTAINERFILE}.${TIMESTAMP}.STDOUT.log) \
    2> >(tee -a ./apptainer/.logs/${APPTAINERFILE}.${TIMESTAMP}.STDERR.log >&2)

unset APPTAINERFILE TIMESTAMP
```

### From Docker Image

```shell
docker save OpenStudioLandscapes-ASWF-OpenRV-BuildBox-CY2024 -o OpenStudioLandscapes-ASWF-OpenRV-BuildBox-CY2024.tar
ls -l  trimmomatic_image.tar  # to see the local copy of image tarball
```

## Non Free Codecs

Decoders:
- [NON_FREE_DECODERS_TO_DISABLE](https://github.com/AcademySoftwareFoundation/OpenRV/blob/c9932773d3a8953ec28d19a591f2fd892179da09/cmake/dependencies/ffmpeg.cmake#L277)
Encoders:
- [NON_FREE_ENCODERS_TO_DISABLE](https://github.com/AcademySoftwareFoundation/OpenRV/blob/c9932773d3a8953ec28d19a591f2fd892179da09/cmake/dependencies/ffmpeg.cmake#L308)

Refs: 
- https://github.com/AcademySoftwareFoundation/OpenRV/blob/c9932773d3a8953ec28d19a591f2fd892179da09/src/lib/image/mio_ffmpeg/init.cpp#L39

## Enable Non Free Codecs

https://github.com/AcademySoftwareFoundation/OpenRV/tree/main?tab=readme-ov-file#how-to-enable-non-free-ffmpeg-codecs

### AAC

```
rvcfg -DRV_FFMPEG_NON_FREE_DECODERS_TO_ENABLE="aac" -DRV_FFMPEG_NON_FREE_ENCODERS_TO_ENABLE="aac"
```

### HEVC

```
rvcfg -DRV_FFMPEG_NON_FREE_DECODERS_TO_ENABLE="hevc"
```

## Run

```
apptainer exec --nv --bind /run/user/$UID,/Volumes,/run/media/$USER OpenRV.sif /opt/OpenRV/bin/rv
# i.e. apptainer exec --nv --bind /run/user/$UID OpenRV.sif /opt/OpenRV/bin/rv -rthreads $(nproc) -workItemThreads $(nproc) -diff '/home/user/Downloads/Big_Buck_Bunny_1080_10s_5MB_AV1.mp4' '/home/user/Downloads/Big_Buck_Bunny_1080_10s_5MB_h264.mp4'
```

### Issues

#### `GLIBC_2.38' not found

- [x] Manjaro

```
$ apptainer exec --nv OpenRV.sif  /build/OpenRV/_install/bin/rv
/build/OpenRV/_install/bin/rv.bin: /lib64/libc.so.6: version `GLIBC_2.38' not found (required by /.singularity.d/libs/libGLX.so.0)
/build/OpenRV/_install/bin/rv.bin: /lib64/libc.so.6: version `GLIBC_2.38' not found (required by /.singularity.d/libs/libGLdispatch.so.0)
```

Reference: https://www.tecmint.com/install-multiple-glibc-libraries-linux/

```shell
wget https://ftp.gnu.org/gnu/glibc/glibc-2.38.tar.xz

tar -xvf glibc-2.38.tar.xz

cd glibc-2.38
mkdir build
cd build
../configure --prefix=/usr/local/glibc-2.38
make -j$(nproc)
sudo make install

export LD_LIBRARY_PATH=/usr/local/glibc-2.38/lib:$LD_LIBRARY_PATH

/usr/local/glibc-2.38/lib/ld-linux-x86-64.so.2 --version

LD_PRELOAD=/usr/local/glibc-2.38/lib/ld-linux-x86-64.so.2 ./your_application
```
