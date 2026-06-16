---
title: Podman Links
---

podman desktop tutorial
https://www.youtube.com/watch?v=YXfA5O5Mr18&t=216s

Connecting to remote machine
```Just a note that as of very recently, Podman Desktop v1.1.12 does support administering remote podman containers. I have it working on my mac.

It's still experimental though, and buried in the menus (Settings -> Preferences -> Podman Extension -> Remote (a toggle button). But the GUI seemed to throw js errors until after I had podman remote properly setup on the cli as follows, so only turn it on after the cli works!

First, you will need to have your ssh keys already setup correctly to ssh in passwordlessly, and I have read that the ed25519 ssh key algorithm is preferred (YMMV). There are many generate and ssh-copy-id tutorials on the internet.

But then you still have to add the remote connection at the command line, and not in the GUI. E.g.

`podman --remote system connection add <<NICKNAMEYOURREMOTEMACHINEHERE>> --identity ~/.ssh/id_ed25519 ssh://<<YOURREMOTEUSERNAMEHERE>>@<<YOURIPHERE>>/run/user/<<YOURUIDHERE>>/podman/podman.sock`

You should test it at the command-line with something like:

podman -c REMOTEMACHINENICKNAME ps

Once I sorted through the various connection errors (I initially had the wrong uid & had to upgrade my remote podman to version 4) and got it to work at the cli, I turned it on in the GUI and it worked.

I can say the Podman Desktop GUI makes it easier to admin remote podman containers. But there are a few issues I can already see with it, such as it does not not yet visually distinguish between a remote and a local container. Again, it is very early stage...
```

ctop to view metrics
So, to work with podman there are some quick steps to set the socket enabled and be transparent with docker-ctop:

1. Install podman with dnf
2. when up, enable both podman and the socket:

```shell
systemctl enable podman
systemctl enable --now podman.socket
```

3. with both services enabled, now confirm where the socket path is set for podman:

```shell
podman info --format '/run/podman/podman.sock'
## on rocky9 is /run/podman/podman.sock
```

4. copy the prompted path, and add to your .bashrc (or shell in use)

```shell
## Set the DOCKER_HOST environment variable to your Podman socket location. 
## Be sure to add the unix:// scheme to the path retrieved previously:
## export DOCKER_HOST=unix://<your_podman_socket_location>
## note to not remove the double slashes
export DOCKER_HOST=unix:///run/podman/podman.sock
```

and that should do it 👯‍♀