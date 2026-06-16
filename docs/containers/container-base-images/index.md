# Container Universal Base Images

Choosing the right container base image is an important task in the container journey: a container base image is the underlying operating system layer that our system\'s service, application, or code will rely on. Due to this, we should choose one that fits our best practices concerning security and updates.

In this chapter, we\'re going to cover the following main topics:

- The Open Container Initiative image format
- Where do container images come from?
- Trusted container image sources
- Introducing Universal Base Image

# The Open Container Initiative image format 

These contributors developed the Runtime Specification (runtime-spec) and the Image Specification (image-spec) for describing how the API and the architecture for new container engines should be created in the future.

After a few months of work, the OCI team released its first implementation of a container runtime that adhered to their specifications; the project was named **runc**.

The specification defines an OCI container image that consists of the following:

- **Manifest**: This contains the metadata of the contents and dependencies of the image. This also includes the ability to identify one or more filesystem archives that will be unpacked to get the final runnable filesystem.
- **Image** **index (optional)**: This represents a list of manifests and descriptors that can provide different implementations of the image, depending on the target platform.
- **Set of** **filesystem** **layers**: The actual set of layers that should be merged to build the final container filesystem.
- **Configuration**: This contains all the information that\'s required by the container runtime engine to effectively run the application, such as arguments and environment variables.

We will not deep dive into every element of the OCI Image Specification as it is out of scope, but the Image Manifest deserves a closer look.

## OCI Image Manifest 

The Image Manifest defines a set of layers and the configuration for a single container image that is built for a specific architecture and an operating system.

Let\'s explore the details of the OCI Image Manifest by looking at the following example:

```
 {
 "schemaVersion": 2, "config": { "mediaType": "application/vnd.oci.image.config.v1+json", "size": 7023, "digest": "sha256:b5b2b2c507a0944348e0303114d8d93aaaa081732b86451d9bce1f432a537bc7" }, "layers": [ { "mediaType": "application/vnd.oci.image.layer.v1.tar+gzip", "size": 32654, "digest": "sha256:9834876dcfb05cb167a5c24953eba58c4ac89b1adf57f28f2f9d09af107ee8f0" } ], "annotations": { "com.example.key1": "value1", "com.example.key2": "value2" } }
```

Here, we are using the following keywords:

- `schemaVersion`: A property that must be set to a value of `2`. This ensures backward compatibility with Docker.
- `config`: A property that references an image\'s configuration through a digest:
- `mediaType`: This property defines the actual configuration format (just one currently)
- `layers`: This property provides an array of descriptor objects:
- `mediaType`: In this case, this descriptor should be one of the media types that are allowed for the layer\'s descriptors
- `annotations`: This property defines additional metadata for the Image Manifest.

To summarize, the main goal of the specification is to make interoperable tools for building, transporting, and preparing a container image to be run.

The Image Manifest Specification has three main goals:

- To enable hashing for the image\'s configuration, thereby generating a unique ID
- To allow multi-architecture images due to its high-level manifest (image index) that references platform-specific versions of the Image Manifest
- To be able to easily translate the container image into the OCI Runtime Specification

Now, let\'s learn where these container images come from.

# Where do container images come from? 

We can choose between the following cloud container registries:

- **Docker** **Hub**: This is the hosted registry solution by Docker Inc. This registry also hosts official repositories and security-verified images for some popular open source projects.
- **Quay.io**: This is the hosted registry solution that was born under the CoreOS company, though it is now part of Red Hat. It offers private and public repositories, automated scanning for security purposes, image builds, and integration with popular Git public repositories.
- **Linux** **distribution** **registries**: Popular Linux distributions are typically community-baked, such as Fedora Linux, or enterprise-based, such as **Red Hat Enterprise Linux** (**RHEL**). They usually offer public container registries, though these are often only available for projects or packages that have already been provided as system packages. These registries are not available to end users and they are fed by the Linux distributions\' maintainers.
- **Public** **cloud** **registries**: Amazon, Google, Microsoft, and other public cloud providers offer public and private container registries for their customers.




# Introducing the Universal Base Image 

When working on enterprise environments, many users and companies adopt RHEL as the operating system of choice to execute workloads reliably and securely. RHEL-based container images are available too, and they take advantage of the same package versioning as the operating system release. All the security updates that are released for RHEL are immediately applied to OCI images, making them wealthy, secure images to build production-grade applications with.

Unfortunately, RHEL images are not publicly available without a Red Hat subscription. Users who have activated a valid subscription can use them freely on their RHEL systems and build custom images on top of them, but they are not freely redistributable without breaking the Red Hat enterprise agreement.

So, why worry? There are plenty of commonly used images that can replace them. This is true, but when it comes to reliability and security, many companies choose to stick to an enterprise-grade solution, and this is no exception for containers.

For these reasons, and to address the redistribution limitations of RHEL images, Red Hat created **UBI**. UBI images are freely redistributable; can be used to build containerized applications, middleware, and utilities; and are constantly maintained and upgraded by Red Hat.

UBI images are based on the currently supported versions of RHEL. At the time of writing, UBI 8, UBI 9, and UBI 10 images are available, based on RHEL 8, RHEL 9, and RHEL 10, respectively. In general, UBI images can be considered a subset of the RHEL operating system.

All UBI images are available on the public Red Hat registry ([[registry.access.redhat.com]{.url}](https://registry.access.redhat.com){style="text-decoration: none;"}) and Docker Hub ([[docker.io]{.url}](https://docker.io){style="text-decoration: none;"}).

There are currently four different flavors of UBI images, each one specialized for a particular use case:

- **Standard**: This is the standard UBI image. It has the most features and package availability.
- **Minimal**: This is a stripped-down version of the standard image with minimalistic package management.
- **Micro**: This is a UBI version with the smallest footprint possible, without a package manager.
- **Init**: This is a UBI image that includes the `systemd` init system so that you can manage the execution of multiple services in a single container.

All of these are *free to use and redistribute* inside custom images. Unlike standard RHEL images, UBI is freely redistributable, allowing developers to build applications on a trusted foundation and share them anywhere, including public registries, without requiring a Red Hat subscription. In addition to that, UBI bridges the gap between development and production; you can build on a laptop running Fedora or a CI/CD pipeline in the cloud and deploy to a production RHEL or OpenShift environment with zero compatibility concerns.

Let\'s describe each in detail, starting with the UBI Standard image.

## The UBI Standard image 

The UBI Standard image is the most complete UBI image version and the closest one to standard RHEL images. It includes the **yum** package manager, which is available in RHEL, and can be customized by installing the packages that are available in its dedicated software repositories, that is, `ubi-8-baseos` and `ubi-8-appstream`.

The following example shows a Dockerfile/Containerfile that uses a standard UBI 8 image to build a minimal `httpd` server:

``` 
FROM registry.access.redhat.com/ubi8

# Update image and install httpd RUN yum update -y && yum install -y httpd && yum clean all

# Expose the default httpd port 80 EXPOSE 80

# Run the httpd CMD ["/usr/sbin/httpd", "-DFOREGROUND"] 
```

The UBI Standard image was designed for generic applications and packages that are available on RHEL and already includes a curated list of basic system tools (including `curl`, `tar`, `vi`, `sed`, and `gzip`) and OpenSSL libraries while still retaining a small size (around 230 MiB): fewer packages mean more lightweight images and a smaller attack surface.

In *[Chapter 6](#Chapter_6.xhtml#h1_163){.chapref}*, *Meet* *Buildah* *-- Building Containers from Scratch*, we learned how to build a new container image starting from a Containerfile or a Dockerfile, so you can leverage Buildah to test the previous example.

If the UBI Standard image is still considered too big, UBI Minimal can be a good fit.

## The UBI Minimal image 

The UBI Minimal image is a stripped-down version of UBI Standard and was designed for self-consistent applications and their runtimes (Python, Ruby, Node.js, and so on). For this reason, it\'s smaller in size, has a small selection of packages, and doesn\'t include the yum package manager; this has been replaced with a minimal tool called `microdnf`. The UBI Minimal image is smaller than UBI Standard and is roughly half its size.

The following example shows a Dockerfile/Containerfile using a UBI 8 minimal image to build a proof-of-concept Python web server:

```
# Based on the UBI8 Minimal image

FROM registry.access.redhat.com/ubi8-minimal

# Upgrade and install Python 3 RUN microdnf upgrade && microdnf install python3

# Copy source code COPY entrypoint.sh http_server.py /

# Expose the default httpd port 80 EXPOSE 8080

# Configure the container entrypoint ENTRYPOINT ["/entrypoint.sh"]

# Run the httpd CMD ["/usr/bin/python3", "-u", "/http_server.py"] 
```

By looking at the source code of the Python web server that\'s been executed by the container, we can see that the web server handler prints a *Hello World!* string when an HTTP GET request is received. The server also manages signal termination using the Python `signal` module, allowing the container to be stopped gracefully:

```
#!/usr/bin/python3 import http.server import socketserver import logging import sys import signal from http import HTTPStatus

port = 8080 message = b'Hello World!\n' logging.basicConfig( stream = sys.stdout, level = logging.INFO )

def signal_handler(signum, frame): sys.exit(0)

class Handler(http.server.SimpleHTTPRequestHandler): def do_GET(self): self.send_response(HTTPStatus.OK) self.end_headers() self.wfile.write(message)

if __name__ == "__main__": signal.signal(signal.SIGTERM, signal_handler) signal.signal(signal.SIGINT, signal_handler) try: httpd = socketserver.TCPServer(('', port), Handler) logging.info("Serving on port %s", port) httpd.serve_forever() except SystemExit: httpd.shutdown() httpd.server_close() 
```

Finally, the Python executable is called by a minimal entry point script:

```
#!/bin/bash set -e exec $@ 
```

The script launches the command that\'s passed by the array in the `CMD` instruction. Also, notice the `-u` option that\'s passed to the Python executable in the command array. This enables unbuffered output and has the container print access logs in real time.

Let\'s try to build and run the container to see what happens:

``` 
$ buildah build -t python_httpd .

$ podman run -p 8080:8080 python_httpd INFO:root:Serving on port 8080 
```

With that, our minimal Python `http` server is ready to operate and serve a lot of barely useful but warming *Hello World!* responses. You can find all these examples in the GitHub repository of the book.

UBI Minimal works best for these kinds of use cases. However, an even smaller image may be necessary. This is the perfect use case for the UBI Micro image.

## The UBI Micro image 

The UBI Micro image is the latest arrival to the UBI family. Its basic idea is to provide a distroless image and a stripped-down package manager, without all the unnecessary packages, to provide a very small image that could also offer a minimal attack surface. Reducing the attack surface is required to achieve secure, minimal images that are more complex to exploit. In addition to that, the small size also minimizes the amount of data that must be downloaded when using the image for the first time or after an update.

The UBI 8 Micro image is great in multi-stage builds, where the first stage creates the finished artifact(s) and the second stage copies them inside the final image. The following example shows a basic multi-stage Dockerfile/Containerfile where a minimal Golang application is being built inside a UBI Standard container while the final artifact is copied inside a UBI Micro image:

```
# Builder image FROM registry.access.redhat.com/ubi8-minimal AS builder

# Install Golang packages RUN microdnf upgrade && \ microdnf install golang && \ microdnf clean all

# Copy files for build COPY go.mod /go/src/hello-world/ COPY main.go /go/src/hello-world/

# Set the working directory WORKDIR /go/src/hello-world

# Download dependencies RUN go get -d -v ./...

# Install the package RUN go build -v ./...

# Runtime image FROM registry.access.redhat.com/ubi8/ubi-micro:latest COPY --from=builder /go/src/hello-world/hello-world /

EXPOSE 8080

CMD ["/hello-world"] 
```

The build\'s output results in an image that\'s approximately 45 MB in size.

The UBI Micro image has no built-in package manager, but it is still possible to install additional packages using Buildah native commands. This works effectively on an RHEL system, where all Red Hat GPG certificates are installed.

The following example shows a build script that can be executed on RHEL 8. Its purpose is to install additional Python packages using the host\'s `yum` package manager, on top of a UBI Micro image:

```
#!/bin/bash

set -euo pipefail

if [ $UID -ne 0 ]; then echo "This script must be run as root" exit 1 fi

container=$(buildah from registry.access.redhat.com/ubi8/ubi-micro) mount=$(buildah mount $container)

yum install -y \ --installroot $mount \ --setopt install_weak_deps=false \ --nodocs \ --noplugins \ --releasever 8 \ python3

yum clean all --installroot $mount

buildah umount $container buildah commit $container micro_python3 
```

Notice that the `yum install` command is executed by passing the `--installroot $mount` option, which tells the installer to use the working container mount point as the temporary root to install the packages.

UBI Minimal and UBI Micro images are great for implementing microservices architectures where we need to orchestrate multiple containers together, with each running a specific microservice.

Now, let\'s look at the UBI Init image, which allows us to coordinate the execution of multiple services inside a container.

## The UBI Init image 

A common pattern in container development is to create highly specialized images with a single component running inside them.

To implement multi-tier applications, such as those with a frontend, middleware, and a backend, the best practice is to create and orchestrate multiple containers, each one running a specific component. The goal is to have minimal and very specialized containers, each one running its own service/process while following the **Keep It Simple, Stupid** (**KISS**) philosophy, which has been carried out in UNIX systems since their inception.

Despite being great for most use cases, this approach does not always suit some special scenarios where many processes need to be orchestrated together. An example is when we need to share all the container namespaces across processes, or when we just want a single, *uber* image.

Container images are normally created without an init system, and the process that\'s executed inside the container (invoked by the `CMD` instruction) usually gets **PID** **1**.

For this reason, Red Hat introduced the UBI Init image, which runs a minimal **systemd** init process inside the container, allowing multiple systemd units that are governed by the systemd process with a PID of `1` to be executed.

The UBI Init image is slightly smaller than the Standard image but has more packages available than the Minimal image.

The default CMD is set to `/sbin/init`, which corresponds to the systemd process. systemd ignores the `SIGTERM` and `SIGKILL` signals, which are used by Podman to stop running containers. For this reason, the image is configured to send `SIGRTMIN+3` signals for termination by passing the `STOPSIGNAL SIGRTMIN+3` instruction inside the image Dockerfile.

The following example shows a Dockerfile/Containerfile that installs the `httpd` package and configures a `systemd` unit to run the `httpd` service:

```
FROM registry.access.redhat.com/ubi8/ubi-init

RUN yum -y install httpd && \ yum clean all && \ systemctl enable httpd

RUN echo "Successful Web Server Test" > /var/www/html/index.html

RUN mkdir /etc/systemd/system/httpd.service.d/ && \ echo -e '[Service]\nRestart=always' > /etc/systemd/system/httpd.service.d/httpd.conf

EXPOSE 80 CMD [ "/sbin/init" ] 
```

Notice the `RUN` instruction, where we create the `/etc/systemd/system/httpd.service.d/` folder and the systemd unit file. This minimal example could be replaced with a copy of pre-edited unit files, which is particularly useful when multiple services must be created.

We can build and run the image and inspect the behavior of the `init` system inside the container using the `ps` command:

``` 
$ buildah build -t init_httpd . $ podman run -d --name httpd_init -p 8080:80 init_httpd $ podman exec -ti httpd_init /bin/bash [root@b4fb727f1907 /]# ps aux USER PID %CPU %MEM VSZ RSS TTY STAT START TIME COMMAND root 1 0.1 0.0 89844 9404 ? Ss 10:30 0:00 /sbin/init root 10 0.0 0.0 95552 10636 ? Ss 10:30 0:00 /usr/lib/systemd/systemd-journald root 20 0.1 0.0 258068 10700 ? Ss 10:30 0:00 /usr/sbin/httpd -DFOREGROUND dbus 21 0.0 0.0 54056 4856 ? Ss 10:30 0:00 /usr/bin/dbus-daemon --system --address=systemd: --nofork --nopidfile --systemd-activation --syslog-only apache 23 0.0 0.0 260652 7884 ? S 10:30 0:00 /usr/sbin/httpd -DFOREGROUND apache 24 0.0 0.0 2760308 9512 ? Sl 10:30 0:00 /usr/sbin/httpd -DFOREGROUND apache 25 0.0 0.0 2563636 9748 ? Sl 10:30 0:00 /usr/sbin/httpd -DFOREGROUND apache 26 0.0 0.0 2563636 9516 ? Sl 10:30 0:00 /usr/sbin/httpd -DFOREGROUND root 238 0.0 0.0 19240 3564 pts/0 Ss 10:30 0:00 /bin/bash root 247 0.0 0.0 51864 3728 pts/0 R+ 10:30 0:00 ps aux 
```

Note that the `/sbin/init` process is executed with a PID of `1` and that it spawns the `httpd` processes. The container also executed `dbus-daemon`, which is used by systemd to expose its API, along with `systemd-journald` to handle logs.

Following this approach, we can add multiple services that are supposed to work together in the same container and have them orchestrated by systemd.

So far, we have looked at the four currently available UBI images and demonstrated how they can be used to create custom applications. Many public Red Hat images are based on UBI. Let\'s take a look.

## Other UBI-based images 

Red Hat uses UBI images to produce many pre-built specialized images, especially for runtimes. They are usually expected not to have redistribution limitations.

This allows runtime images to be created for languages, runtimes, and frameworks such as Python, Quarkus, Golang, Perl, PDP, .NET, Node.js, Ruby, and OpenJDK.

UBI is also used as the base image for the **Source-to-Image** (**S2I**) framework, which is used to build applications natively in OpenShift without the use of Dockerfiles. With S2I, it is possible to assemble images from user-defined custom scripts and, obviously, application source code.

Last but not least, Red Hat\'s supported container releases of Buildah, Podman, and Skopeo are packaged using UBI 8 images.

Moving beyond Red Hat\'s offering, other vendors use UBI images to release their images too -- Intel, IBM, Isovalent, Cisco, Aqua Security, and many others adopt UBI as the base for their official images on the Red Hat Marketplace.

# Further reading 

To learn more about the topics that were covered in this chapter, take a look at the following resources:

- \[1\] MITRE ATT&CK® matrix: https://attack.mitre.org/matrices/enterprise/containers/
- \[2\] Things you should know about the Kubernetes threat matrix: https://cloud.redhat.com/blog/2021-kubernetes-threat-matrix-updates-things-you-should-know
- \[3\] How to manage Linux container registries: https://www.redhat.com/sysadmin/manage-container-registries
- \[4\] (Re)Introducing the Red Hat Universal Base Image: https://www.redhat.com/en/blog/introducing-red-hat-universal-base-image
- \[5\] Introduction to Red Hat\'s UBI Micro: https://www.redhat.com/en/blog/int