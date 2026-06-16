---
title: Container Basics 
description: Container Basics with Podman
---

Podman interacts with both SELinux and AppArmor to provide better container isolation, but the underlying interfaces are different.

Containers are Linux-based, and the different container engines and runtimes interact with the Linux kernel and libraries to
operate. Windows has also introduced support for native containers with an approach to isolation that\'s quite close to the
Linux namespaces concepts described previously.
However, only Windows-based images can run natively, and very few container engines support native execution.

For macOS: its architecture is not based on Linux but on a hybrid Mach/BSD kernel called **XNU** ("X is not Unix" For this reason, it does not offer the Linux kernel features necessary to run containers natively.
Apple\'s container
- Written in Swift and 
- optimized specifically for Apple silicon
- allows users to run Linux containers as lightweight virtual machines. 
- Consumes and produces OCI-compatible images
- Fully interoperable with standard registries
- Pull, build, and push images that function across any OCI-compatible ecosystem.

For both Windows and macOS, a virtualization layer that abstracts the Linux machine is necessary to run native Linux containers.

Podman offers remote client functions for Windows and macOS, enabling users to connect to a local or remote Linux box.

Windows users can also benefit from an alternative approach based on the **Windows Subsystem for Linux** (**WSL**) **2.0**,
a compatibility layer that runs a lightweight VM to expose Linux kernel interfaces along with Linux userspace binaries, thanks to Hyper-V virtualization support.

# Running your first container

`podman run` 

``` 
$ podman run <imageID>
```

 If the image is not present in the cache or we have not downloaded it before, Podman will pull the image for us from the respective container registry.
## Interactive and pseudo-tty
``` 
$ podman run -i -t fedora /bin/bash
Resolved "fedora" as an alias (/etc/containers/registries.conf.d/000-shortnames.conf)
Trying to pull registry.fedoraproject.org/fedora:latest...
Getting image source signatures
Copying blob dbe590460ba2 done   |
Copying config 5eea52ff5b done   |
Writing manifest to image destination
[root@a45628efe4b8 /]
```

The previous command prompted an interactive shell thanks to the two options that we can analyze, as follows:

`--tty, -t`
- Podman allocates a **pseudo-tty** (see `man pty`) and attaches it to the container's standard input.

`--interactive, -i`
- Podman keeps `stdin` open and ready to be attached to the previous pseudo-tty

When a container is created, the isolated processes inside it will run on a writable root filesystem, as a result of a layered overlay.

This allows any process to write files, but don\'t forget that they will last until the container is running, as containers are ephemeral by
default.

To exit this interactive shell, we can just press *Ctrl* + *D* or execute the `exit` command. By doing this,
the container will be terminated because the main running process we requested to execute (`/bin/bash`) will stop!

## Detaching from a running container

Once a container has been started, we can easily detach from it, even if we start it with an interactive `tty` attached:
`Ctrl + P, Ctrl + Q`. With this sequence, we will return to our shell prompt while the container will keep running.

To recover our detached container's `tty`, we must get the list of running containers:
``` 
$ podman ps
CONTAINER ID  IMAGE                           COMMAND               CREATED        STATUS            PORTS       NAMES
685a339917e7  registry.fedoraproject.org/httpd:latest  /usr/bin/run-http...  3 minutes ago  Up 3 minutes ago              clever_zhukovsky
```

To re-attach to the previous `tty`:
``` 
$ podman attach 685a339917e7
```

To start a container in *detached* mode: (`-d`)
``` 
$ podman run -d registry.fedoraproject.org/httpd
```


## Network port publishing

Podman, like any other container engine, attaches a virtual network to a container in a running state that has been isolated from
the original host network. If we want to easily reach our container or even expose it outside our host network, we need to instruct Podman to do port mapping.

The Podman `-p` option publishes a container\'s port to the host:
``` 
-p=ip:hostPort:containerPort/protocol
```

- Both `hostPort` and `containerPort` could be a range of ports
- If the host IP is not set or it is set to `0.0.0.0`, then the port will be bound to all the IP addresses of the host
- If set to \[`::`\] then we will bind to all IPv6 addresses. 
- Protocol is an optional field that could be useful to further restrict the access to TCP or UDP connections.

``` 
$ podman run -p 8080:8080 -d registry.fedoraproject.org/httpd
```

To ook at the port mapping:
``` 
$ podman port fc9d97642801 <- container id
8080/tcp -> 0.0.0.0:8080
```

Test if this port mapping with curl of visit in a browser:
``` 
$ curl -s 127.0.0.1:8080 | head
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  <!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.1//EN" "http://www.w3.org/TR/xhtml11/DTD/xhtml11.dtd">
4
6<html xmlns="http://www.w3.org/1999/xhtml" xml:lang="en" lang="en">
5    <head>
0         <title>Test Page for the Apache HTTP Server on Fedora</title>
          <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
          <style type="text/css">
1               /*<![CDATA[*/
0               body {
0                     background-color: #fff;
```

## Configuration and environment variables

Change the time zone of  running containers (`--tz`):
``` 
$ date
Wed Sep 25 09:57:25 AM UTC 2024

$ podman run --tz=Asia/Shanghai fedora date
Wed Sep 25 17:57:33 CST 2024
```

Change the DNS of our brand-new container with the `--dns` option:
``` 
$ podman run --dns=1.1.1.1 fedora cat /etc/resolv.conf

nameserver 1.1.1.1
```

Add a host to the `/etc/hosts` file to override a local internal address:
``` 
$ podman run --add-host=my.server.local:192.168.1.10 \
fedora  cat /etc/hosts
192.168.1.10    my.server.local
127.0.0.1    localhost localhost.localdomain localhost4 localhost4.localdomain4
1    localhost localhost.localdomain localhost6 localhost6.localdomain6
192.168.124.6    3497f23d8551 serene_shannon
```

Add an HTTP proxy to let our container use a proxy for HTTP requests. The default Podman behavior is to pass many
environment variables from the host, some of which are
- `http_proxy`
- `https_proxy`
- `ftp_proxy`
- `no_proxy`

Define custom environment variables that are passed to the container (`–env`):
``` 
$ podman run --env MYENV=podman fedora printenv
container=oci
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
MYENV=podman
HOME=/root
HOSTNAME=1db6008431c2
```

Adding and using environment variables with our containers is a best practice for passing configuration parameters to the application and influencing the service\'s behavior from the operating system host. We should leverage environment variables to configure a container at runtime.



