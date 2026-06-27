

Download VMWWare Workstation Pro from
- https://support.broadcom.com/group/ecx/downloads
- https://www.vmware.com/products/desktop-hypervisor/workstation-and-fusion
- https://support.broadcom.com/group/ecx/productfiles?subFamily=VMware%20Workstation%20Pro&displayGroup=VMware%20Workstation%20Pro%2026H1%20%20for%20Linux&release=26H1&os=&servicePk=543222&language=EN&freeDownloads=true
- https://techdocs.broadcom.com/us/en/vmware-cis/desktop-hypervisors/workstation-pro/26H1/using-vmware-workstation-pro.html

```shell
sudo chmod a+x /home/michael/Downloads/VMware-Workstation-Full-26H1-25388281.x86_64.bundle
# scripts: /etc/init.d
```

```shell
vmware
```

Ubuntu

Install Apptainer
- https://apptainer.org/docs/admin/main/installation.html#install-ubuntu-packages
```shell
sudo apt update
sudo apt install -y software-properties-common

sudo add-apt-repository -y ppa:apptainer/ppa
sudo apt update
sudo apt install -y apptainer

# Test
# apptainer exec docker://alpine cat /etc/alpine-release
```
-> `segmentation fault`

Rocky

```shell
ssh user@192.168.178.88
dnf install -y htop

su root
cd

# https://aswf-openrv.readthedocs.io/en/latest/build_system/config_linux_rocky89.html



dnf upgrade -y

dnf config-manager --set-enabled crb devel

sudo dnf config-manager --set-enabled crb devel
dnf install -y perl-CPAN
# cpan FindBin
PERL_MM_USE_DEFAULT=1 cpan FindBin

dnf install -y wget git epel-release
dnf groupinstall -y "Development Tools"

dnf install -y alsa-lib-devel autoconf automake avahi-compat-libdns_sd-devel bison bzip2-devel cmake-gui curl-devel flex gcc gcc-c++ git libXcomposite libXi-devel libaio-devel libffi-devel nasm ncurses-devel nss libtool libxkbcommon libXcomposite libXdamage libXrandr libXtst libXcursor mesa-compat-libOSMesa mesa-compat-libOSMesa-devel meson openssl-devel patch pulseaudio-libs pulseaudio-libs-glib2 ocl-icd ocl-icd-devel opencl-headers qt5-qtbase-devel readline-devel sqlite-devel systemd-devel tcl-devel tcsh tk-devel yasm zip zlib-devel wget patchelf pcsc-lite libxkbfile perl-IPC-Cmd
dnf install -y libX11-devel libXext-devel libXrender-devel libXrandr-devel libXcursor-devel libXi-devel libXxf86vm-devel libxkbcommon-devel
dnf install -y xz-devel mesa-libGLU mesa-libGLU-devel

dnf clean all

dnf install -y ccache
ccache --max-size=10G
# ccache --show-stats
# ccache --clear

dnf config-manager --set-disabled devel

# Install Pyenv
echo 'export PIP_ROOT_USER_ACTION=ignore' >> /etc/profile
echo 'export PYENV_ROOT="/opt/pyenv"' >> /etc/profile
echo 'export PATH="${PYENV_ROOT}/shims:${PYENV_ROOT}/bin:${PATH}"' >> /etc/profile
exec -l $SHELL
# export PYENV_ROOT="/opt/pyenv"
# export PATH="${PYENV_ROOT}/shims:${PYENV_ROOT}/bin:${PATH}"
mkdir -p "${PYENV_ROOT}"
# export PIP_ROOT_USER_ACTION=ignore
git clone http://github.com/pyenv/pyenv.git ${PYENV_ROOT}
echo 'eval "$(pyenv init -)"' >> /etc/profile
exec -l $SHELL
pyenv install 3.11.8
pyenv global 3.11.8
pip install --upgrade pip

# Install CMake
echo 'export CMAKE_VERSION="3.31.6"' >> /etc/profile
echo 'export CMAKE_BASE="/opt/cmake"' >> /etc/profile
echo 'export PATH="${CMAKE_BASE}/bin:${PATH}"' >> /etc/profile
exec -l $SHELL
mkdir -p ${CMAKE_BASE}
curl \
    -L https://github.com/Kitware/CMake/releases/download/v${CMAKE_VERSION}/cmake-${CMAKE_VERSION}.tar.gz \
    -o cmake-${CMAKE_VERSION}.tar.gz
tar -xzvf cmake-${CMAKE_VERSION}.tar.gz
pushd cmake-${CMAKE_VERSION}
# ./configure --prefix=${CMAKE_BASE}
./bootstrap --parallel=$(nproc --all) --prefix=${CMAKE_BASE}
make -j $(nproc --all)
make install
popd
rm -rf cmake-${CMAKE_VERSION}*

# Install Rust
echo 'export RUSTUP_HOME="/opt/rust"' >> /etc/profile
echo 'export CARGO_HOME="/opt/rust"' >> /etc/profile
echo 'export PATH="${RUSTUP_HOME}/bin:${CARGO_HOME}/bin:${PATH}"' >> /etc/profile
exec -l $SHELL
mkdir -p ${RUSTUP_HOME}
mkdir -p ${CARGO_HOME}
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- -y --no-modify-path

# Install Ninja
echo 'export NINJA_HOME="/opt/ninja"' >> /etc/profile
echo 'export NINJA_VERSION="1.12.1"' >> /etc/profile
echo 'export PATH="${NINJA_HOME}:${PATH}"' >> /etc/profile
exec -l $SHELL
curl \
    -L https://github.com/ninja-build/ninja/releases/download/v${NINJA_VERSION}/ninja-linux.zip \
    -o ninja-linux.zip
unzip ninja-linux.zip -d "${NINJA_HOME}"
rm -f ninja-linux.zip

# Install Qt
echo 'export QT_BASE="/opt/Qt"' >> /etc/profile
echo 'export QT_VERSION="6.5.3"' >> /etc/profile
echo 'export QT_MODULES="debug_info qt3d qt5compat qtcharts qtconnectivity qtdatavis3d qtgrpc qthttpserver qtimageformats qtlanguageserver qtlocation qtlottie qtmultimedia qtnetworkauth qtpdf qtpositioning qtquick3d qtquick3dphysics qtquickeffectmaker qtquicktimeline qtremoteobjects qtscxml qtsensors qtserialbus qtserialport qtshadertools qtspeech qtvirtualkeyboard qtwaylandcompositor qtwebchannel qtwebengine qtwebsockets qtwebview"' >> /etc/profile
echo 'export QT_ARCHIVES="icu qtbase qtdeclarative qtsvg qttools qttranslations qtwayland"' >> /etc/profile
# echo 'export QT_QPA_PLATFORM="offscreen"' >> /etc/profile
echo 'export QT_HOME="${QT_BASE}/${QT_VERSION}/gcc_64"' >> /etc/profile
# echo 'export PATH="${NINJA_HOME}:${PATH}"' >> /etc/profile
exec -l $SHELL
python -m pip install aqtinstall
python -m aqt install-qt linux desktop ${QT_VERSION} gcc_64 -O ${QT_BASE} \
    -m ${QT_MODULES} \
    --archives ${QT_ARCHIVES}

# https://aswf-openrv.readthedocs.io/en/latest/build_system/config_common_build.html

# echo 'export WORKDIR="/rv/build"' >> /etc/profile
# RV_HOME has to be the root of the Git repository
# However, files like
# - /home/michael/git/repos/OpenStudioLandscapes-ASWF-OpenRV/src/bin/apps/rvpkg/rvpkg.wrapper
# refer to RV_HOME and build their paths to executable:
# FAILED: stage/app/plugins/Packages/rvinstall /rv/build/OpenRV/_build/stage/app/plugins/Packages/rvinstall 
# cd /rv/build/OpenRV/_build/stage/app/bin && /rv/build/OpenRV/_build/RV_DEPS_PYTHON3/install/bin/python3 /rv/build/OpenRV/src/build/install_all_rvpkg.py --rvpkg /rv/build/OpenRV/_build/stage/app/bin/rvpkg --destination /rv/build/OpenRV/_build/stage/app/plugins --source /rv/build/OpenRV/_build/stage/packages
# /rv/build/OpenRV/bin/python: Command not found.
# /rv/build/OpenRV/bin/rvpkg.bin: Command not found.
# Traceback (most recent call last):
#   File "/rv/build/OpenRV/src/build/install_all_rvpkg.py", line 41, in <module>
#     install_rvpkg_packages(**vars(parser.parse_args()))
#   File "/rv/build/OpenRV/src/build/install_all_rvpkg.py", line 26, in install_rvpkg_packages
#     subprocess.run(command).check_returncode()
#   File "/rv/build/OpenRV/_build/RV_DEPS_PYTHON3/install/lib/python3.11/subprocess.py", line 502, in check_returncode
#     raise CalledProcessError(self.returncode, self.args, self.stdout,
# subprocess.CalledProcessError: Command '[PosixPath('/rv/build/OpenRV/_build/stage/app/bin/rvpkg'), '-force', '-install', '-add', PosixPath('/rv/build/OpenRV/_build/stage/app/plugins'), PosixPath('/rv/build/OpenRV/_build/stage/packages/additional_nodes-1.2.rvpkg'), PosixPath('/rv/build/OpenRV/_build/stage/packages/annot_tools-0.1.rvpkg'), PosixPath('/rv/build/OpenRV/_build/stage/packages/annotate-1.21.rvpkg'), PosixPath('/rv/build/OpenRV/_build/stage/packages/channel_select-1.3.rvpkg'), PosixPath('/rv/build/OpenRV/_build/stage/packages/collapse_missing_frames-1.2.rvpkg'), PosixPath('/rv/build/OpenRV/_build/stage/packages/custom_lut_menu_mode-2.9.rvpkg'), PosixPath('/rv/build/OpenRV/_build/stage/packages/custom_mattes-2.3.rvpkg'), PosixPath('/rv/build/OpenRV/_build/stage/packages/data_display_indicators-1.2.rvpkg'), PosixPath('/rv/build/OpenRV/_build/stage/packages/doc_browser-1.4.rvpkg'), PosixPath('/rv/build/OpenRV/_build/stage/packages/export_cuts-1.5.rvpkg'), PosixPath('/rv/build/OpenRV/_build/stage/packages/lat_long_viewer-1.2.rvpkg'), PosixPath('/rv/build/OpenRV/_build/stage/packages/layer_select-1.7.rvpkg'), PosixPath('/rv/build/OpenRV/_build/stage/packages/maya_tools-1.6.rvpkg'), PosixPath('/rv/build/OpenRV/_build/stage/packages/media_library_demo-1.0.rvpkg'), PosixPath('/rv/build/OpenRV/_build/stage/packages/missing_frame_bling-1.7.rvpkg'), PosixPath('/rv/build/OpenRV/_build/stage/packages/multiple_source_media_rep-1.3.rvpkg'), PosixPath('/rv/build/OpenRV/_build/stage/packages/node_graph_viz-0.1.rvpkg'), PosixPath('/rv/build/OpenRV/_build/stage/packages/ocio_source_setup-2.5.rvpkg'), PosixPath('/rv/build/OpenRV/_build/stage/packages/openrv_help_menu-1.0.rvpkg'), PosixPath('/rv/build/OpenRV/_build/stage/packages/os_dependent_path_conversion_mode-1.7.rvpkg'), PosixPath('/rv/build/OpenRV/_build/stage/packages/otio_reader-1.2.rvpkg'), PosixPath('/rv/build/OpenRV/_build/stage/packages/pyhello-1.2.rvpkg'), PosixPath('/rv/build/OpenRV/_build/stage/packages/pymystuff-1.2.rvpkg'), PosixPath('/rv/build/OpenRV/_build/stage/packages/pyside_example-1.2.rvpkg'), PosixPath('/rv/build/OpenRV/_build/stage/packages/rvio_basic_scripts-1.2.rvpkg'), PosixPath('/rv/build/OpenRV/_build/stage/packages/rvnuke-1.12.rvpkg'), PosixPath('/rv/build/OpenRV/_build/stage/packages/scrub_offset-1.6.rvpkg'), PosixPath('/rv/build/OpenRV/_build/stage/packages/sequence_from_file-1.4.rvpkg'), PosixPath('/rv/build/OpenRV/_build/stage/packages/session_manager-1.10.rvpkg'), PosixPath('/rv/build/OpenRV/_build/stage/packages/source_setup-4.3.rvpkg'), PosixPath('/rv/build/OpenRV/_build/stage/packages/stereo_autoload-1.6.rvpkg'), PosixPath('/rv/build/OpenRV/_build/stage/packages/stereo_disassembly-1.3.rvpkg'), PosixPath('/rv/build/OpenRV/_build/stage/packages/sync-1.5.rvpkg'), PosixPath('/rv/build/OpenRV/_build/stage/packages/webview2-0.3.rvpkg'), PosixPath('/rv/build/OpenRV/_build/stage/packages/window_title-1.6.rvpkg')]' returned non-zero exit status 1.
# ninja: build stopped: cannot make progress due to previous errors.
# Let's call it RV_REPO
echo 'export RV_REPO="/rv/build/OpenRV"' >> /etc/profile
echo 'export RV_VFX_PLATFORM="CY2024"' >> /etc/profile
echo 'export RV_BUILD_TYPE="Debug"' >> /etc/profile
# echo 'export RV_BUILD_TYPE="Release"' >> /etc/profile
exec -l $SHELL
mkdir -p ${RV_REPO}
pushd ${RV_REPO}

git clone --recursive https://github.com/AcademySoftwareFoundation/OpenRV.git .
# pushd OpenRV
git config blame.ignoreRevsFile .git-blame-ignore-revs
popd
# source rvcmds.sh
# https://aswf-openrv.readthedocs.io/en/latest/build_system/config_common_build.html#first-time-build-only-rvbootstrap
# - rvbootstrap
#   - rvsetup
#   - rvmk

# create venv
# rvsetup
# rvenv
#   __rv_env_shell
#     python3 -m venv .venv
#     source ".venv/bin/activate"
#   __rv_install_precommit_hooks
#     pre-commit install
# cmd="SETUPTOOLS_USE_DISTUTILS=${SETUPTOOLS_USE_DISTUTILS} python3 -m pip install --upgrade -r ${RV_REPO}/requirements.txt"
# eval ${cmd}
# CMake parameters:
# RV_BUILD_PARALLELISM is 4
# RV_REPO is /rv/build/OpenRV
# RV_BUILD_DIR is /rv/build/OpenRV/_build
# RV_INST_DIR is /rv/build/OpenRV/_install
# CMAKE_GENERATOR is Ninja
# QT_HOME is /opt/Qt/6.5.3/gcc_64
# rvmk
# rvcfg
#   rvenv
#   cmd="cmake -B ${RV_BUILD_DIR} -G '${CMAKE_GENERATOR}' ${RV_TOOLCHAIN} ${CMAKE_WIN_ARCH} -DCMAKE_BUILD_TYPE=${RV_BUILD_TYPE} -DRV_DEPS_QT_LOCATION=${QT_HOME} -DRV_VFX_PLATFORM=${RV_VFX_PLATFORM} -DRV_DEPS_WIN_PERL_ROOT=${WIN_PERL} ${*}"
#   eval ${cmd}
# rvbuild
#   rvenv
#   rvbuildt main_executable
#     rvenv
#     __rv_build_with_errors ${*}


# Isolated Steps
# https://linuxvox.com/blog/after-changing-etc-profile-what-do-i-have-to-do-to-reset-my-shell/
# Environment Setup
exec -l $SHELL
pushd ${RV_REPO}
source rvcmds.sh

# rvbootstrap
# tail -f /rv/build/OpenRV/_build/build_errors.log

python3 -m venv .venv
source .venv/bin/activate
pip install --upgrade pip
SETUPTOOLS_USE_DISTUTILS=${SETUPTOOLS_USE_DISTUTILS} python3 -m pip install --upgrade -r ${RV_REPO}/requirements.txt
pre-commit install

time cmake -B ${RV_BUILD_DIR} -G "${CMAKE_GENERATOR}" ${RV_TOOLCHAIN} ${CMAKE_WIN_ARCH} -DCMAKE_BUILD_TYPE=${RV_BUILD_TYPE} -DRV_DEPS_QT_LOCATION=${QT_HOME} -DRV_VFX_PLATFORM=${RV_VFX_PLATFORM} -DRV_DEPS_WIN_PERL_ROOT=${WIN_PERL} 2>&1 | tee "cmake.log"
deactivate
popd

# export NINJA_STATUS="[%f/%t %p] "
# ninja_flags="-v"
# ninja_flags="-k 0"
# ninja_flags="-j 0"

# Build Dependencies
# Logs: tail -f ${RV_REPO}/_build/build_dependencies.log
# rvbuildt dependencies
# = __rv_build_with_errors dependencies
exec -l $SHELL
pushd ${RV_REPO}
source rvcmds.sh
source .venv/bin/activate
set -xo pipefail
TARGET="dependencies" && time cmake --build ${RV_BUILD_DIR} --config ${RV_BUILD_TYPE} -v --parallel=${RV_BUILD_PARALLELISM} --target ${TARGET} -- --verbose ${ninja_flags} 2>&1 | tee "${RV_BUILD_DIR}/build_${TARGET}.log"
set +xo pipefail
deactivate
popd

# Build OpenRV
# Option: [ ] export CMAKE_GENERATOR="Unix Makefiles"  # Uncomment to not use Ninja
# Logs: tail -f ${RV_REPO}/_build/build_main_executable.log
# rvbuildt main_executable
# = __rv_build_with_errors main_executable
exec -l $SHELL
pushd ${RV_REPO}
source rvcmds.sh
source .venv/bin/activate
set -xo pipefail
TARGET="main_executable" && time cmake --build ${RV_BUILD_DIR} --config ${RV_BUILD_TYPE} -v --parallel=${RV_BUILD_PARALLELISM} --target ${TARGET} -- --verbose ${ninja_flags} 2>&1 | tee "${RV_BUILD_DIR}/build_${TARGET}.log"
set +xo pipefail
deactivate
popd

# Install
exec -l $SHELL
pushd ${RV_REPO}
source rvcmds.sh
source .venv/bin/activate
time cmake --install ${RV_BUILD_DIR} --prefix ${RV_INST_DIR} --config ${RV_BUILD_TYPE} 2>&1 | tee "${RV_BUILD_DIR}/install_errors.log"
deactivate
popd
```

```shell
tcsh

setenv RV_HOME "/rv/build/OpenRV/_install"
setenv LD_LIBRARY_PATH "$RV_HOME/lib"

$RV_HOME/bin/rv.bin
```

Segmentation fault:
```shell
coredumpctl -r -n 10
coredumpctl gdb <PID>
dnf --enablerepo='*debug*' install -y libxkbcommon-debuginfo-1.0.3-4.el9.x86_64 libglvnd-glx-debuginfo-1.3.4-1.el9.x86_64 mesa-libGLU-debuginfo-9.0.1-6.el9.x86_64 freetype-debuginfo-2.10.4-10.el9_5.x86_64 libX11-debuginfo-1.8.12-1.el9.x86_64 libglvnd-opengl-debuginfo-1.3.4-1.el9.x86_64 libstdc++-debuginfo-11.5.0-14.el9.x86_64 glibc-debuginfo-2.34-270.el9_8.x86_64 libgcc-debuginfo-11.5.0-14.el9.x86_64 nss-debuginfo-3.112.0-8.el9.x86_64 nss-util-debuginfo-3.112.0-8.el9.x86_64 nspr-debuginfo-4.36.0-8.el9.x86_64 libXcomposite-debuginfo-0.4.5-7.el9.x86_64 libXdamage-debuginfo-1.1.5-7.el9.x86_64 libXext-debuginfo-1.3.4-8.el9.x86_64 libXfixes-debuginfo-5.0.3-16.el9.x86_64 libXrender-debuginfo-0.9.10-16.el9.x86_64 libXrandr-debuginfo-1.5.2-8.el9.x86_64 libXtst-debuginfo-1.2.3-16.el9.x86_64 fontconfig-debuginfo-2.14.0-2.el9_1.x86_64 expat-debuginfo-2.5.0-6.el9_8.1.x86_64 libdrm-debuginfo-2.4.128-1.el9.x86_64 libxcb-debuginfo-1.13.1-9.el9.x86_64 libXi-debuginfo-1.7.10-8.el9.x86_64 alsa-lib-debuginfo-1.2.15.3-1.el9.x86_64 dbus-libs-debuginfo-1.12.20-8.el9.x86_64 libxshmfence-debuginfo-1.3-10.el9.x86_64 libxkbfile-debuginfo-1.1.0-8.el9.x86_64 pulseaudio-libs-debuginfo-15.0-3.el9.x86_64 krb5-libs-debuginfo-1.21.1-10.el9_8.x86_64 libglvnd-egl-debuginfo-1.3.4-1.el9.x86_64 glib2-debuginfo-2.68.4-19.el9_8.1.x86_64 libglvnd-debuginfo-1.3.4-1.el9.x86_64 libaio-debuginfo-0.3.111-13.el9.x86_64 bzip2-libs-debuginfo-1.0.8-11.el9.x86_64 libpng-debuginfo-1.6.37-15.el9_8.2.x86_64 harfbuzz-debuginfo-2.7.4-10.el9.x86_64 libbrotli-debuginfo-1.0.9-9.el9_7.x86_64 libxml2-debuginfo-2.9.13-14.el9_8.1.x86_64 libXau-debuginfo-1.0.9-8.el9.x86_64 systemd-libs-debuginfo-252-67.el9_8.4.rocky.0.1.x86_64 libcom_err-debuginfo-1.46.5-8.el9.x86_64 keyutils-libs-debuginfo-1.6.3-1.el9.x86_64 pcre-debuginfo-8.44-4.el9.x86_64 graphite2-debuginfo-1.3.14-9.el9.x86_64 xz-libs-debuginfo-5.2.5-8.el9_0.x86_64 libcap-debuginfo-2.48-10.el9_7.1.x86_64 libgcrypt-debuginfo-1.10.0-11.el9.x86_64 libzstd-debuginfo-1.5.5-1.el9.x86_64 lz4-libs-debuginfo-1.9.3-5.el9.x86_64 libsndfile-debuginfo-1.0.31-9.el9_8.1.x86_64 libasyncns-debuginfo-0.8-22.el9.x86_64 libselinux-debuginfo-3.6-3.el9.x86_64 libgpg-error-debuginfo-1.42-5.el9.x86_64 gsm-debuginfo-1.0.19-6.el9.x86_64 flac-libs-debuginfo-1.3.3-10.el9_2.1.x86_64 libvorbis-debuginfo-1.3.7-5.el9.x86_64 opus-debuginfo-1.3.1-10.el9.x86_64 libogg-debuginfo-1.3.4-6.el9.x86_64 pcre2-debuginfo-10.40-6.el9.x86_64 libwayland-client-debuginfo-1.21.0-1.el9.x86_64 libwayland-cursor-debuginfo-1.21.0-1.el9.x86_64 libffi-debuginfo-3.4.2-8.el9.x86_64 gtk3-debuginfo-3.24.31-8.el9.x86_64 pango-debuginfo-1.48.7-3.el9.x86_64 atk-debuginfo-2.36.0-5.el9.x86_64 cairo-gobject-debuginfo-1.17.4-7.el9.x86_64 cairo-debuginfo-1.17.4-7.el9.x86_64 gdk-pixbuf2-debuginfo-2.42.6-6.el9_7.1.x86_64 fribidi-debuginfo-1.0.10-6.el9.2.x86_64 libepoxy-debuginfo-1.5.5-4.el9.x86_64 at-spi2-atk-debuginfo-2.38.0-4.el9.x86_64 libtracker-sparql-debuginfo-3.1.2-3.el9_1.x86_64 libwayland-egl-debuginfo-1.21.0-1.el9.x86_64 libXcursor-debuginfo-1.2.0-7.el9.x86_64 libXinerama-debuginfo-1.1.4-10.el9.x86_64 libthai-debuginfo-0.1.28-8.el9.x86_64 pixman-debuginfo-0.40.0-6.el9_3.x86_64 libmount-debuginfo-2.37.4-25.el9.x86_64 at-spi2-core-debuginfo-2.40.3-1.el9.x86_64 libstemmer-debuginfo-0-18.585svn.el9.x86_64 json-glib-debuginfo-1.6.6-1.el9.x86_64 libicu-debuginfo-67.1-10.el9_6.x86_64 sqlite-libs-debuginfo-3.34.1-10.el9_8.x86_64 libdatrie-debuginfo-0.2.13-4.el9.x86_64 libblkid-debuginfo-2.37.4-25.el9.x86_64 dconf-debuginfo-0.40.0-6.el9.x86_64 gvfs-client-debuginfo-1.48.1-8.el9.x86_64 mesa-libEGL-debuginfo-25.2.7-4.el9.x86_64 mesa-dri-drivers-debuginfo-25.2.7-4.el9.x86_64 mesa-libgbm-debuginfo-25.2.7-4.el9.x86_64 libX11-xcb-debuginfo-1.8.12-1.el9.x86_64 llvm-libs-debuginfo-21.1.8-2.el9.x86_64 spirv-tools-libs-debuginfo-2025.4-1.el9.x86_64 elfutils-libelf-debuginfo-0.194-1.el9.rocky.0.1.x86_64 libedit-debuginfo-3.1-39.20210216cvs.el9.x86_64 ncurses-libs-debuginfo-6.2-12.20210508.el9.x86_64

```

X11?

```shell
dnf install -y xorg-x11-server-Xorg xorg-x11-xauth xorg-x11-utils
vi /etc/gdm/custom.conf
# uncomment WaylandEnable=false
# QT_QPA_PLATFORM=xcb /rv/build/OpenRV/_install_debug/bin/rv
# Available platform plugins are: eglfs, linuxfb, minimal, minimalegl, offscreen, vkkhrdisplay, vnc, wayland-egl, wayland, xcb.
dnf install xcb-util-cursor
QT_QPA_PLATFORM=xcb /rv/build/OpenRV/_install_debug/bin/rv
# => success
```