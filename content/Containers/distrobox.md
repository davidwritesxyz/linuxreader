---
title: Distrobox Basics
summary: Getting started with Distrobox
---

Github: https://github.com/89luca89/distrobox


Run native containers that have access to the host file system
Creates an app launcher icon for the container

Launch gui apps from other distros natively. Even without spinning up the container first

Make separate shortcuts for separate distros in the terminal

Make a profile in the terminal for the container:

`distrobox-enter --name ubuntu-20

Make sure to check the box "run a custom command instead of my shell"

Then set a custom shortcut in your OS to that terminal profile
`gnome-terminal --profile=ubuntu`

Find a list of containers supported here: https://distrobox.it/compatibility/#containers-distros
## How to Install

`curl -s https://raw.githubusercontent.com/89luca89/distrobox/main/install | sudo sh`

or 

`sudo dnf -y install distrobox`

You can find a list of container distros here: https://github.com/89luca89/distrobox/blob/main/docs/compatibility.md#host-distros

## Commands

### Create a container
`distrobox create --name aur --image archlinux` or
`distrobox-create --name archlinux --image docker.io/library/archlinux` or
`distrobox create -n ubuntu -i ubuntu:20.04`

### Additional Options

--nvidia
	- add your host nvidia drivers

--additional-packages {package-name}
	- install additional packages

### Enter the container

`distrobox enter aur`

### Add app from another distro to your app launcher/ application menu

`distrobox-export --app app-name 

### Create a GUI app 

`distrobox-export --app {appname}`

Create a temporary container
distrobox-ephemeral --image archlinux

Create a container for testing server distros
`distrobox create --image almalinux:latest`

Create an arm container
```
distrobox create -i aarch64/fedora -n fedora-arm64
distrobox enter fedora-arm64
```

Add distro to your display manager to boot into the distribution

https://cloudyday.tech.blog/2022/05/14/distrobox-is-awesome/
https://github.com/89luca89/distrobox/blob/main/docs/posts/run_latest_gnome_kde_on_distrobox.md


## Setting up a fedora distrobox in ublue

To create a Fedora 38 Distrobox, run:

`distrobox create --image registry.fedoraproject.org/fedora-toolbox:38 --name fedora`

Enter the box:
`distrobox enter fedora`

To create an Ubuntu 22.04 Distrobox, run:

`distrobox create --image docker.io/library/ubuntu:22.04 --name ubuntu`

To create an Arch Linux Distrobox, run:

`distrobox create --image docker.io/library/archlinux:latest --name arch`

To create an Debian Linux Distrobox, run:

`distrobox create --image docker.io/library/debian:latest --name debian`


[List of tested container images](https://distrobox.privatedns.org/compatibility/#containers-distros).

When choosing an image for your container, consider which package manager and repos you want to use (e.g. `apt` vs. `pacman`), and pick one that you’re most comfortable with!
## Exporting programs from Distrobox

You can export a GUI program (for example, `mpv`) by running the following from **inside the Distrobox**:

`distrobox-export --app mpv`

You can also export a binary or CLI program (for example, `vim`) by running the following from **inside the Distrobox**. You need to provide an export path and know where the original binary exists in the Distrobox’s filesystem. An easy way to find out is running `which vim` (replace `vim` with the name of the binary you want to export). The export command will create a shell script that runs the specified binary from inside the Distrobox.

`distrobox-export --bin /usr/bin/vim --export-path ~/.local/bin`

[Read more in the Distrobox documentation about `distrobox-export`](https://distrobox.privatedns.org/usage/distrobox-export/)

## Integrating VSCode with Distrobox

There are two ways to integrate VSCode with a Distrobox.  
The easiest way is to just install it inside your Distrobox and `distrobox-export` it. The integrated terminal and all addons will then be run inside that single Distrobox. The other way is with [the Dev Containers extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers).  
Both ways are detailed inside [the official Distrobox tutorial for integrating with VSCode](https://distrobox.privatedns.org/posts/integrate_vscode_distrobox/)

## Using the host’s `xdg-open` inside Distrobox

Some GUI programs use `xdg-open` to open URLs, but it doesn’t work when running your browser on the host.  
You can fix this by adding the following shell script to your `~/.local/bin/` and giving it executable permissions.

`#!/bin/bash if [ ! -e /run/.containerenv ] && [ ! -e /.dockerenv ]; then # if not inside a container     /usr/bin/xdg-open "$@" # run xdg-open normally else     distrobox-host-exec /usr/bin/xdg-open "$@" # run xdg-open on the host fi`

If you have `xdg-open` installed inside the Distrobox, you need to make sure that `~/.local/bin/` is in your `$PATH` before `/usr/bin/`.

## Immutable distros

Apps do not get root access to your o not get root access to your system.

## Cloud native client

/usr is read only parts of the disk
Able to roll back after updates


