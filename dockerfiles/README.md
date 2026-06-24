

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

## x11docker

```shell
x11docker --gpu --init=systemd -- --cap-add SYS_RESOURCE -- x11docker/kde-plasma startplasma-wayland
```

```shell
x11docker --gpu --kwin x11docker/kde-plasma plasmashell
```

```shell
x11docker --wayland --dbus x11docker/kde-plasma konsole 
```