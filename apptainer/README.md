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
    * [Log in to Private Registry](#log-in-to-private-registry)
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
- [Multistage Apptainer](https://github.com/apptainer/apptainer/blob/main/examples/multistage/Apptainer)

```shell
export TARGET="rv_rocky"

mkdir -p ./OpenStudioLandscapes-ASWF-OpenRV-BuildBox-CY2024-${TARGET}

docker save \
    openstudiolandscapes-aswf-openrv-rocky9-cy2024-${TARGET}:latest \
    --output ./OpenStudioLandscapes-ASWF-OpenRV-BuildBox-CY2024-${TARGET}/openstudiolandscapes-aswf-openrv-rocky9-cy2024-${TARGET}.latest.tar

apptainer build \
    ./OpenStudioLandscapes-ASWF-OpenRV-BuildBox-CY2024-${TARGET}/openstudiolandscapes-aswf-openrv-rocky9-cy2024-${TARGET}.sif \
    docker-archive://./OpenStudioLandscapes-ASWF-OpenRV-BuildBox-CY2024-${TARGET}/openstudiolandscapes-aswf-openrv-rocky9-cy2024-${TARGET}.latest.tar
#     \
#    && chown "$(stat -c %u /work)":"$(stat -c %g /work)" /work/<output_name>.sif

# apptainer --version
# apptainer version 1.5.0
# singularity --version
# apptainer version 1.5.0
# -> https://apptainer.org/docs/user/main/singularity_compatibility.html#singularity-command-symlink

# https://stackoverflow.com/a/60316979
# --compat ?
# apptainer cache clean
# /tmp might not be large enough for large images.
# https://apptainer.org/docs/user/main/build_env.html#temporary-folders
# APPTAINER_TMPDIR needs to be absolute
# APPTAINER_CACHEDIR needs to be absolute?
APPTAINER_TMPDIR=/home/michael/git/repos/OpenStudioLandscapes-ASWF-OpenRV/apptainer/tmp \
APPTAINER_CACHEDIR=/home/michael/git/repos/OpenStudioLandscapes-ASWF-OpenRV/apptainer/cache \
singularity build \
    ./OpenStudioLandscapes-ASWF-OpenRV-BuildBox-CY2024-${TARGET}/openstudiolandscapes-aswf-openrv-rocky9-cy2024-${TARGET}.sif \
    docker-daemon://openstudiolandscapes-aswf-openrv-rocky9-cy2024-${TARGET}:latest

# i.e.
# apptainer exec \
#     --nv \
#     --bind /run/user/$UID \
#     ./OpenStudioLandscapes-ASWF-OpenRV-BuildBox-CY2024-${TARGET}/openstudiolandscapes-aswf-openrv-rocky9-cy2024-${TARGET}.sif \
#     /opt/rv/bin/rv \
#     -rthreads $(nproc) \
#     -workItemThreads $(nproc) \
#     -diff \
#     '/home/user/Downloads/Big_Buck_Bunny_1080_10s_5MB_AV1.mp4' \
#     '/home/user/Downloads/Big_Buck_Bunny_1080_10s_5MB_h264.mp4'

# The user in the Docker image has to be root?
apptainer -d exec \
    ./OpenStudioLandscapes-ASWF-OpenRV-BuildBox-CY2024-${TARGET}/openstudiolandscapes-aswf-openrv-rocky9-cy2024-${TARGET}.sif \
    /opt/rv/bin/rv
# /opt/rv/bin/rv.bin
# Version 4.0.0 (RELEASE), built on Jun 24 2026 at 07:57:53 (HEAD=4517984).
# Copyright Contributors to the Open RV Project
# INFO: File logger path: /home/michael/.local/share/ASWF/OpenRV/
# Segmentation fault         (core dumped) apptainer exec --nv ./OpenStudioLandscapes-ASWF-OpenRV-BuildBox-CY2024-${TARGET}/openstudiolandscapes-aswf-openrv-rocky9-cy2024-${TARGET}.sif /opt/rv/bin/rv
```

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

### Log in to Private Registry

```shell
apptainer registry \
    login \
    -u <registry-user> \
    -p <registry-password> \
    docker://registry.openstudiolandscapes.lan:5000
```

```
# cat /home/michael/.docker/config.json
{
        "auths": {
                "https://index.docker.io/v1/": {
                        "auth": "bWljaGltdXNzYXRvOmRja3JfcGF0X21EbklXYXpqcThZN21tZm5sNE1BeG5HSDZYOA=="
                },
                "https://index.docker.io/v1/access-token": {
                        "auth": "bWljaGltdXNzYXRvOmV5SmhiR2NpT2lKU1V6STFOaUlzSW5SNWNDSTZJa3BYVkNJc0ltdHBaQ0k2SW5oWWEzQkNkRE55VjNNeVJ5MTFZamxzY0VwbmNTSjkuZXlKb2RIUndjem92TDJoMVlpNWtiMk5yWlhJdVkyOXRJanA3SW1WdFlXbHNJam9pYldsamFHbHRkWE56WVhSdlFHZHRZV2xzTG1OdmJTSXNJbk5sYzNOcGIyNWZhV1FpT2lKbU9EUmpOell4WmkwMU0yUTBMVFExWXpRdFltVTNNaTFsTnpjME5EazRaVGN5TW1JaUxDSnpiM1Z5WTJVaU9pSmhkWFJvTUNJc0luVnpaWEp1WVcxbElqb2liV2xqYUdsdGRYTnpZWFJ2SWl3aWRYVnBaQ0k2SWpJMk1UTTJORGcyTFRCaU1XSXROR0V6TnkxaE9XTm1MVFppTTJWaFkyTm1aREEwWWlKOUxDSnBjM01pT2lKb2RIUndjem92TDJ4dloybHVMbVJ2WTJ0bGNpNWpiMjB2SWl3aWMzVmlJam9pWVhWMGFEQjhNall4TXpZME9EWXRNR0l4WWkwMFlUTTNMV0U1WTJZdE5tSXpaV0ZqWTJaa01EUmlJaXdpWVhWa0lqcGJJbWgwZEhCek9pOHZhSFZpTG1SdlkydGxjaTVqYjIwaUxDSm9kSFJ3Y3pvdkwyUnZZMnRsY2kxd2NtOWtMblZ6TG1GMWRHZ3dMbU52YlM5MWMyVnlhVzVtYnlKZExDSnBZWFFpT2pFM09ERXhPREV5TnpFc0ltVjRjQ0k2TVRjNE1URTRNakUzTVN3aWMyTnZjR1VpT2lKdmNHVnVhV1FnYjJabWJHbHVaVjloWTJObGMzTWlMQ0poZW5BaU9pSk1OSFl3Wkcxc1RrSndXVlZxUjBkaFlqQkRNa3AwWjFSbldISXhVWG8wWkNKOS52TE5aR2NVd2p4QnMxSkRRSXFYRzYyY2QweW01Slg5azNXT1cycEllejRBVkFhNDh0UGEwNERzVk8wN1E0Z3A5YllXdFRONzBPV2NTeUkxdF8xWjRCbS1nUGV2b090ZTJWSElXSmU5bzNyRjVQZ0dTTC1iY3NLMHBid2dUX05VLWZ1a1lJSzRJaW9sWGtCMHhZa25QeTk3SDVoUFRiMGhDMXdzTS1QVy1pTnQyeHFFUkxNTjlZaDE5bFYtZWt3dW9nR0M4djFFSXVabWUwQUstbWFmVDVYN0xEek50WU5NUWlvN2Q5bzBaTFF1dzlpaWZVTzRGSHpGR0NNbFVvTF8tTS03ejEwWWt6ZkJDZG5yeFJVZTZDVERRMkdwTDBIUUlSTGdJM2c5RklYYjhtZmpCdTFnbEZzMDRTc1BxaG5pU181aGxIWGlrS1dCYnNZU0xsbmM3MUE="
                },
                "https://index.docker.io/v1/refresh-token": {
                        "auth": "bWljaGltdXNzYXRvOnYxLk1mYnlBUEo1a3dSaktUb1NSY3B1TkJFR2NrbTE2Zm5ucnc0M282cEZSVHVlZ18tN2VNOXkyWjB3TkZfQjJtQmZ5SWY0QlE2aHlVc2M1SzJkWjdjZWdWYy4uTDR2MGRtbE5CcFlVakdHYWIwQzJKdGdUZ1hyMVF6NGQ="
                },
                "registry-1.docker.io": {
                        "auth": "bWljaGltdXNzYXRvQGdtYWlsLmNvbTo3RjtCZSY6JjYzSGkzfSY="
                },
                "registry.openstudiolandscapes.lan:5000": {
                        "auth": "cmVnaXN0cnktdXNlcjpyZWdpc3RyeS1wYXNzd29yZA=="
                }
        }
}
```

```yaml
# cat /home/michael/.apptainer/remote.yamll
Active: DefaultRemote
Remotes:
    DefaultRemote:
        URI: cloud.apptainer.org
        System: true
        Exclusive: false
Credentials:
    - URI: docker://registry.openstudiolandscapes.lan:5000
      Insecure: false
```

### Apptainer

```shell
export APPTAINERFILE="OpenRV.def"
export TIMESTAMP=$(date +"%Y-%m-%d_%H-%M-%S")

export APPTAINER_TMPDIR=/home/michael/git/repos/OpenStudioLandscapes-ASWF-OpenRV/apptainer/tmp
export APPTAINER_CACHEDIR=/home/michael/git/repos/OpenStudioLandscapes-ASWF-OpenRV/apptainer/cache

/usr/bin/time -f 'Commandline Args: %C\nElapsed Time: %E\nPeak Memory: %M\nExit Code: %x' \
    apptainer \
    --config ./conf/apptainer.conf \
    --debug \
    build \
    --ignore-fakeroot-command \
    --build-arg-file .env \
    --warn-unused-build-args \
    ./.sif/OpenRV_${TIMESTAMP}.sif \
    ./${APPTAINERFILE} \
    &> >(tee -a ./.logs/${APPTAINERFILE}.${TIMESTAMP}.log)

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
$ apptainer --debug exec --nv OpenRV_2026-06-26_15-42-48.sif /rv/build/OpenRV/_install/bin/rv
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
