

---

# GUI Applications in Docker Container

- https://unix.stackexchange.com/a/359244
- https://github.com/mviereck/x11docker
  - https://github.com/mviereck/dockerfile-x11docker-kde-plasma
- https://hub.docker.com/r/xorilog/telegram

## docker run

```shell
mkdir -p ~/telegram
docker run --rm -it --name telegram \
    --hostname=$(hostname) \
    -e DISPLAY=unix$DISPLAY \
    -e WAYLAND_DISPLAY=$WAYLAND_DISPLAY \
    -e PULSE_SERVER=unix:$XDG_RUNTIME_DIR/pulse/native \
    -v /tmp/.X11-unix:/tmp/.X11-unix \
    -v "/home/$(whoami)/.Xauthority:/home/user/.Xauthority" \
    -v $XDG_RUNTIME_DIR/pulse:$XDG_RUNTIME_DIR/pulse \
    -v /etc/localtime:/etc/localtime:ro \
    -v $XDG_RUNTIME_DIR/$WAYLAND_DISPLAY:/tmp/$WAYLAND_DISPLAY \
    -v ~/telegram:/home/user/.local/share/TelegramDesktop/ \
    xorilog/telegram
```

```shell
docker run --rm -it --name vlc \
    --hostname=$(hostname) \
    -e DISPLAY=unix$DISPLAY \
    -e WAYLAND_DISPLAY=$WAYLAND_DISPLAY \
    -e PULSE_SERVER=unix:$XDG_RUNTIME_DIR/pulse/native \
    -v /tmp/.X11-unix:/tmp/.X11-unix \
    -v "/home/$(whoami)/.Xauthority:/home/vlc/.Xauthority" \
    -v $XDG_RUNTIME_DIR/pulse:$XDG_RUNTIME_DIR/pulse \
    -v /etc/localtime:/etc/localtime:ro \
    -v $XDG_RUNTIME_DIR/$WAYLAND_DISPLAY:/tmp/$WAYLAND_DISPLAY \
    -v ~/Downloads/:/home/vlc/Downloads \
    jess/vlc
```

```shell
# https://github.com/linuxserver/docker-baseimage-selkies#gpu-configuration
docker run --rm -it \
    --runtime=nvidia \
    --name rv \
    --hostname=$(hostname) \
    --cap-add=SYS_ADMIN \
    --user 1000:1000 \
    -e DISPLAY=unix$DISPLAY \
    -e WAYLAND_DISPLAY=$WAYLAND_DISPLAY \
    -e PULSE_SERVER=unix:$XDG_RUNTIME_DIR/pulse/native \
    -e NVIDIA_VISIBLE_DEVICES=all \
    -e NVIDIA_DRIVER_CAPABILITIES=all \
    -v $XDG_RUNTIME_DIR/pulse:$XDG_RUNTIME_DIR/pulse \
    -v /etc/localtime:/etc/localtime:ro \
    -v $XDG_RUNTIME_DIR/$WAYLAND_DISPLAY:/tmp/$WAYLAND_DISPLAY \
    --volume="/dev/dri:/dev/dri:ro" \
    --volume="${HOME}/.local/share/ASWF/OpenRV:/home/rv/.local/share/ASWF/OpenRV:rw" \
    --entrypoint /opt/rv/bin/rv \
    openstudiolandscapes-aswf-openrv-rocky9-cy2024-rv_rocky:latest "/home/michael/Downloads/Full Caddy Server Docker Course HTTPS Reverse Proxy Deployments.mp4"
```

## x11docker

```shell
pacman -Syy x11docker
```

```shell
QT_QPA_PLATFORM=xcb
# x11docker --pulseaudio --share=$HOME/Videos jess/vlc
x11docker --gpu --pipewire --share=$HOME/Videos --share=$HOME/Downloads jess/vlc
```

```shell
x11docker --gpu --init=systemd -- --cap-add SYS_RESOURCE -- x11docker/kde-plasma startplasma-wayland
```

```shell
x11docker --gpu --kwin x11docker/kde-plasma plasmashell
```

```shell
x11docker --wayland --dbus x11docker/kde-plasma konsole 
```