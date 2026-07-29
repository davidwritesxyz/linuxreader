---
title: Podman Socket
summary: Activating the Podman Daemon
---

## Optional -- enable socket-based services

Users may need to interact with the Libpod APIs exposed by Podman, and in addition to that, Podman also has Docker-compatible APIs
that can be used to transition, especially useful when migrating from a Docker-based environment.

Podman can expose its APIs using a UNIX socket (default behavior) or a TCP socket. The latter option is less secure because it
makes Podman accessible from the outside world, but it is necessary in some cases, such as when it should be accessed by a  Podman client on a Windows or macOS workstation. Before doing this, please also evaluate the chances of mediating the service through SSH access and protocol.

The following command exposes the Podman APIs on a UNIX socket:
```
$ sudo podman system service --time=0 \
  unix:///run/podman/podman.sock
```

After running this command, users can connect to the API service.

Having to run this command on a terminal window is not a handy approach. Instead, the best approach is to use a **systemd** **socket** (see `man systemd.socket`).

Socket units in `systemd` are special kinds of service activators: when a request reaches the pre-defined endpoint of the socket, systemd immediately spawns the homonymous service.

When Podman is installed, the `podman.socket` and `podman.service` unit files are created. `podman.socket` has the following content:
```
# /usr/lib/systemd/system/podman.socket
[Unit]
Description=Podman API Socket
Documentation=man:podman-system-service(1)
[Socket]
ListenStream=%t/podman/podman.sock
SocketMode=0660
[Install]
WantedBy=sockets.target
```

The `ListenStream` key holds the relative path of the socket, which is expanded to
`/run/podman/podman.sock`

The complete `podman.service` Systemd unit file has the following content:
```
# /usr/lib/systemd/system/podman.service
[Unit]
Description=Podman API Service
Requires=podman.socket
After=podman.socket
Documentation=man:podman-system-service(1)
StartLimitIntervalSec=0
[Service]
Type=exec
KillMode=process
Environment=LOGGING="--log-level=info"
ExecStart=/usr/bin/podman $LOGGING system service
[Install]
WantedBy=multi-user.target
```

The `ExecStart` field indicates the command to be launched by the service, which is the same `podman system service` command we showed previously. The `Requires` field indicates that the `podman.service` unit needs `podman.socket` to be activated.

So, what happens when we enable and start the `podman.socket` unit? `systemd` handles the socket and waits for a connection to the socket endpoint. When this event happens, it immediately starts the `podman.service` unit.

After a period of inactivity, the service is stopped again.

To enable and start the socket unit, run the following command:
```
# systemctl enable --now podman.socket
```

We can test the results with a simple `curl` command:
```
# curl --unix-socket /run/podman/podman.sock \ 
  http://d/v3.0.0/libpod/info
```

The printed output will be a JSON payload that contains the container engine configuration.

What happened when we hit the URL? Under the hood, the service unit was immediately started and triggered by the socket when the connection was established. Some of you may have noticed a slight delay (in the order of 1/10th of a second) the very first time the
command was executed.

After 5 seconds of inactivity, `podman.service` deactivates again. This is due to the default behavior of the `podman system service` command, which runs for 5 seconds only by default unless the `–time` option is passed to provide a different timeout (a
value of 0 means forever).