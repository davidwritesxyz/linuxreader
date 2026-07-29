---
title: "Installing Podman"
summary: "Installing Podman on different systems"
---

### Installing Podman on Fedora, CentOS Stream, Rocky Linux

```
# dnf install -y podman
```

- Installs Podman
- configures the environment with config files (covered in the next section).
- installs `systemd`units to provide additional features such as REST API services or container auto-updates.

### Installing Podman on RHEL

On RHEL 8, the Podman package is available as a single package and also under a dedicated module called `container-tools` 

``` {.programlisting .snippet-con}
# yum module enable -y container-tools:rhel8
# yum module install -y container-tools:rhel8
```

Installs, along with Podman and required libraries, other useful tools that will be covered later in
this book:
-   **Skopeo**, a tool for managing OCI images and registries
-   **Buildah**, a specialized tool for building custom OCI images from Dockerfiles and from scratch
-   CRIU, a utility to implement checkpoint/restore functionality for Linux
-   Udica, a tool for generating SELinux security profiles for containers

If not interested in the full module content, users can install the Podman package only:

``` {.programlisting .snippet-con}
# yum install -y podman
```

On RHEL9/10, there is no `container-tools`module. Instead, users will be able to install a `container-tools` meta-package that brings the same extra tools as the module:
```
# dnf install -y container-tools
```

If not interested in full meta-package:
```
# dnf install -y podman
```

### (Not) Installing Podman on Fedora CoreOS and Fedora Silverblue

Podman is already installed on both distributions and is a crucial tool for running containerized
workloads.

The **Fedora** **CoreOS** and **Fedora** **Silverblue** distributions are immutable, atomic operating systems aimed to be used on server/cloud and desktop environments, respectively.

Fedora CoreOS https://getfedora.org/en/coreos is the upstream of Red Hat CoreOS, the operating system used to run Red
Hat OpenShift and the base OS of **OpenShift** **Kubernetes** **Distribution** (**OKD**), the community-based Kubernetes distribution used as the upstream of Red Hat OpenShift.

Fedora Silverblue
- Desktop-focused immutable operating system that aims to provide a stable and comfortable desktop user experience, especially for working with containers.



### Installing Podman on Debian and Raspberry Pi OS
``` {.programlisting .snippet-con}
# apt-get -y install podman
```

### Installing Podman on Ubuntu
``` {.programlisting .snippet-con}
# apt-get -y update
# apt-get -y install podman
```

### Installing Podman on openSUSE
``` {.programlisting .snippet-con}
# zypper install podman
```

### Installing Podman on Gentoo
``` {.programlisting .snippet-con}
# emerge app-emulation/podman
```

The `emerge` utility will download and automatically build the Podman sources on the system.

### Installing Podman on Arch Linux
``` {.programlisting .snippet-con}
# pacman -S podman
```

By default, Podman\'s installation on Arch Linux does not permit rootless containers. To enable them, follow the official Arch
wiki instructions: https://wiki.archlinux.org/title/Podman#Rootless_Podman

### Installing Podman on macOS

Apple users develop and run Linux containers can install and use Podman as a remote client, while the containers are executed on a remote Linux box. The Linux machine can also be a VM that's executed on macOS and directly managed by Podman.

The Podman project offers a macOS installer that can be downloaded from Podman.io

Alternatively, even though this is not recommended by the Podman developer team, users can install Podman using the Homebrew package manager by running the following command from the Terminal:
```
$ brew install podman
```

To initialize the VM running the Linux box, run the following commands:
```
$ podman machine init
$ podman machine start
```

In the preceding example, the Podman Machine service is initialized and started.The service downloads and configures a dedicated Linux virtual machine that runs the containers seamlessly. You can even initialize and start it with this command:
```
$ podman machine init --now
```

Alternatively, users can create and connect to an external Linux host by using Podman as a remote client.

Another valid approach on macOS to creating fast, lightweight VMs for development use is Vagrant. When the Vagrant machine is created, users can manually or automatically provision additional software, such as Podman, and start using the customized instance using the remote client.

### Installing Podman on Windows

Running Podman on Windows Subsystem or Linux (also known as WSLv2) is a simple and convenient alternative to execute
containers on Windows. The guest distribution runs on a Podman machine that can be automatically created with the
`podman machine init` command. This solution requires a recent release of Windows 10 or Windows 11 to support WSLv2 and a system capable of supporting virtualization (used internally by WSLv2).

As an alternative, it's even possible to use Hyper-V as documented on the Podman documentation page.

As a first step, download the latest release of the installer (named `podman-x.y.z-setup.exe` from the GitHub releases page
https://github.com/containers/podman/releases/ and launch the setup to complete the installation process.

When the setup is complete, open a terminal and launch the following command:
```
PS C:\Users\User> podman machine init
```

If WSLv2 is not already installed on the system, the command will also take care of it and produce a prompt to initialize the automated
installation of the Podman machine, which includes the installation of the necessary WSLv2 components, reboot(s), and the import of the Podman machine. Obviously, users can choose to skip the automated installation of WSLv2 and choose to install it manually. If WSLv2 is already installed, the command will simply download a minimal Fedora distribution and import it into WSL.

When the import into WSLv2 is complete, start the Podman machine:
```
PS C:\Users\User> podman machine start
```

The system is now ready to run containers in the Podman machine executed in WSLv2.

Alternatively, to run Podman as a remote client only, we have a Podman command to manage the system connections:
`podman system connection add`, which automates the edit in Podman's configuration. The manual, not recommended, way is to download and install the latest release from the GitHub releases page https://github.com/containers/podman/releases/, Extract the `.zip` archive in a suitable location and edit the TOML-encoded `containers.conf` file to configure a remote URI for the Linux machine or pass additional options.

The following code snippet shows an example configuration:

```
[engine]
remote_uri= " ssh://root@10.10.1.9:22/run/podman/podman.sock"
```

The remote Linux machine exposes Podman on a UNIX socket managed by a systemd

### Building Podman from source

Building an application from source has many advantages: users can inspect and customize code before building, cross-compile for different architectures, or selectively build only a subset of binaries. It is also a great learning opportunity to get into the project's structure and understand its evolution. Last but not least, building from source lets the users get the latest development versions with cool new features, bugs included.

The following steps assume that the building machine is a Fedora distribution.

Install the necessary build dependencies needed to compile Podman:
```
# git clone https://github.com/containers/podman/ && \
  cd podman
# dnf install -y builddep rpm/podman.spec
```

The `dnf builddep` command will install all the necessary build dependencies declared in the `rpm/podman.spec` file. It will take a while to install all the packages and their cascading dependencies.

Before starting the build, install the following runtime dependencies:
```
$ sudo dnf -y install \
  conmon \
  containers-common \
  crun \
  netavark \
  nftables \
```

Remember that the RPM format is associated with Fedora/CentOS/RHEL distributions and managed by the `dnf` and `yum` package managers.

Change to the project directory and start the build:
```
$ make BUILDTAGS="selinux seccomp" PREFIX=/usr
$ sudo make install PREFIX=/usr
```

The first `make` command compiles the source code, applying specific build tags that enable **SELinux** and **seccomp** support, even though in the latest release, the build process should auto-detect the build tags based on the installed libraries. The next `sudo``make install` command installs the packages locally under the `/usr/bin` path.

The build process will take a few minutes to complete. To test the successful installation of the packages, simply run the following command:
```
$ podman version
Client:       Podman Engine
Version:      5.3.0-dev
API Version:  5.3.0-dev
Go Version:   go1.22.7
Git Commit:   af4b061f5383938a38910d81a0f637c478fc838f
Built:        Tue Sep 24 18:11:45 2024
OS/Arch:      linux/amd64
```

To create a binary release similar to the `.tar.gz` archive, which is available on the GitHub release page, run the following command:
```
$ make podman-release
```

Building a different version is very easy -- just switch to the tag of the target release using the `git` command. For example, to build v5.3.0, use the following command:
```
$ git checkout v5.3.0
```
